# User Story — Fiabiliser l’installation et la disponibilité d’AIDD dans les projets et les worktrees Kilo Code

## Informations Jira

- **Type :** User Story
- **Composant :** Outillage transverse COE / Développement agentique
- **Priorité proposée :** Haute
- **Labels proposés :** aidd, kilo-code, developer-experience, makefile, worktree
- **Estimation :** à affiner après analyse de l’existant.

## User story

**En tant que** développeur d’une squad utilisant l’outillage COE,

**Je veux** que l’installation d’AIDD soit contrôlée, facilement réparable et automatiquement préparée lors de la création d’un worktree Kilo Code,

**Afin de** disposer du framework, de ses skills, agents et configurations attendus dans chaque espace de travail, sans devoir mémoriser une procédure spécifique ni versionner les fichiers générés dans le dépôt du produit.

## Contexte et problématique

Le COE distribue son outillage sous forme de submodule Git dans les dépôts produits. Une variable Makefile portée par le submodule fixe la version d’AIDD supportée. L’installation récupère la distribution depuis le gestionnaire d’artefacts et prépare localement l’environnement Kilo Code, notamment le répertoire `.kilo/`.

Les fichiers AIDD générés sont actuellement ignorés par Git, afin de ne pas mélanger le cycle de vie du framework avec celui du code produit.

Deux situations posent problème :

- Un développeur clone ou reclone un dépôt : les fichiers générés ne sont pas présents. Il doit penser à lancer l’installation.
- Un développeur crée un worktree dans Kilo Code : les fichiers ignorés ne sont pas automatiquement présents dans le nouveau worktree. Les skills peuvent être absents au démarrage de la session.

Une mise à jour du submodule COE peut également changer la version AIDD attendue sans que l’installation locale ait encore été synchronisée.

## Décisions et principes d’architecture

- Conserver la version AIDD définie par le submodule COE comme source de vérité. La révision du submodule représente un ensemble de composants compatibles : AIDD, configuration Kilo, règles et outillage transverse. Aucune mise à jour implicite vers la dernière version disponible.
- Continuer à récupérer les artefacts AIDD via le mécanisme de distribution existant. Ne pas embarquer les skills natifs AIDD dans le dépôt COE ni dans les dépôts produits.
- Conserver une installation propre à chaque clone ou worktree, sans installer globalement AIDD dans les répertoires de skills du HOME.
- Privilégier la réutilisation des cibles Makefile existantes, sans multiplier les commandes ni dupliquer les diagnostics.
- Ne pas introduire de nouveau mécanisme de cache. Le package est suffisamment léger ; télécharger la distribution attendue lorsqu’une installation est nécessaire. Ne pas supprimer un éventuel mécanisme existant au titre de cette US.
- Préserver les skills, règles, agents et configurations spécifiques au produit ainsi que les autres fonctionnalités de l’outillage COE.
- Aucune installation ou mise à jour automatique lors d’un simple `make help`. Aucun impact sur les pipelines CI existants.

## Existant à réutiliser

| Commande | Rôle affiché dans l’aide actuelle |
| --- | --- |
| `make help` | Aide et contrôle existant des submodules |
| `make run-setup` | Initialisation Python (binaire, configuration, environnement et dépendances) |
| `make aidd-full-setup` | Installation complète Kilo + AIDD |
| `make aidd-prereqs` | Prérequis de l’installation |
| `make aidd-setup` | Installation du framework |
| `make aidd-config-kilo` | Configuration Kilo |
| `make aidd-check` | Vérification de la structure `.kilo/` |
| `make aidd-status` | Affichage de la version et de l’état existant |
| `make aidd-doctor` | Diagnostic de santé |
| `make aidd-update` | Resynchronisation du framework depuis le gestionnaire d’artefacts |
| `make aidd-agents` | Affichage des modèles par défaut des agents |
| `make aidd-tree` / `make aidd-show-config` | Diagnostic de l’arborescence et de la configuration |
| `make gitignore-sync` | Synchronisation du bloc gitignore géré par le COE |

Les implémentations réelles de ces cibles doivent être examinées avant modification. Ne pas supposer qu’elles sont toutes idempotentes ou qu’elles ne réalisent aucun accès réseau.

## Périmètre fonctionnel

### 1. Diagnostic dans make help

Étendre la section de contrôle existante pour signaler l’état d’AIDD dans le clone ou worktree courant. Réutiliser les vérifications de `aidd-check`, `aidd-status` ou `aidd-doctor`, en mode local et synthétique si nécessaire.

Distinguer au minimum : submodule indisponible (version attendue indéterminable), installation absente, installation partielle/invalide, version différente de celle attendue, installation conforme.

Afficher la version attendue et la version effectivement installée si elles sont déterminables. Ne pas déduire la version installée uniquement de la variable du Makefile. Proposer une commande de correction adaptée (installation complète, synchronisation ou diagnostic détaillé). Le contrôle doit être rapide, non bloquant, sans réseau et sans écriture. Les messages d’aide restent cohérents avec la langue et le format existants.

Exemple indicatif :

```text
Environment checks:
  [OK]   COE submodule initialized
  [WARN] AIDD not installed in this workspace
         Run: make aidd-full-setup
```

Le diagnostic statique ne doit pas affirmer que les skills sont effectivement chargés dans une session Kilo : cette disponibilité sera vérifiée séparément par un test runtime.

### 2. Onboarding et nouveaux clones

Documenter la commande d’installation complète dans le parcours de démarrage développeur. Étudier l’intégration dans une étape de setup développeur existante ; ne pas ajouter AIDD aux étapes Python ou CI par défaut sans vérifier leur périmètre et leur compatibilité. Si `run-setup` est utilisé en CI ou pour Python uniquement, conserver une étape AIDD explicite dans l’onboarding.

Un clone Git standard ne déclenche pas de script post-clone distribué par le dépôt : le parcours repose sur l’initialisation documentée et le contrôle dans `make help`.

### 3. Préparation automatique des worktrees Kilo

Fournir un script de setup worktree versionné, distribué via le mécanisme d’intégration COE existant (template ou script géré). Sur Linux/macOS, utiliser le mécanisme natif `.kilo/setup-script` ou `.kilo/setup-script.sh`. Si Windows natif est supporté par le COE, prévoir la variante adaptée.

Le script cible le worktree courant (variable `WORKTREE_PATH`), initialise le submodule COE à la révision référencée par ce worktree si nécessaire, puis appelle les cibles existantes suffisantes pour installer et configurer AIDD localement. Ne pas réinstaller systématiquement Kilo ou ses prérequis globaux s’ils sont déjà présents. Terminer par une vérification de l’installation et remonter clairement les erreurs. Ne pas modifier le clone principal, les autres worktrees ou leurs révisions.

Le bootstrap et la migration des projets existants doivent permettre de disposer du script avant la création d’un nouveau worktree, sans écraser un éventuel script spécifique au produit. Documenter sa relance et le rattrapage manuel pour les worktrees déjà créés ou créés hors de Kilo.

Référence publique : [Kilo Code – Agent Manager Workflows](https://kilo.ai/docs/automate/agent-manager-workflows).

### 4. Gestion Git et mises à jour

Adapter `gitignore-sync` afin d’ignorer les artefacts générés par AIDD et les fichiers locaux Kilo, tout en permettant le suivi du script de bootstrap et des fichiers spécifiques au produit qui doivent être partagés.

Éviter une exclusion globale de `.kilo/` empêchant les exceptions ; vérifier les règles avec `git check-ignore` et les fichiers effectivement suivis avec Git. Définir la frontière entre fichiers générés et fichiers personnalisés d’après l’installation réelle. Ne jamais versionner de secrets ou de configurations personnelles.

Garantir une installation et une synchronisation répétables : si la version attendue et les fichiers gérés sont déjà conformes, ne pas effectuer de réinstallation inutile. En cas de changement de version du COE, utiliser la version attendue par le submodule courant, y compris en cas de retour à une version antérieure. Préserver les fichiers non gérés par l’installateur.

## Critères d’acceptation (tests Given / When / Then)

| ID | Étant donné (Given) | Quand (When) | Alors (Then) |
| --- | --- | --- | --- |
| AC01 | Un clone neuf avec submodule initialisé et sans AIDD | Le développeur lance `make help` | L’absence d’AIDD est affichée avec `make aidd-full-setup` ; l’aide se termine normalement sans installer ni télécharger quoi que ce soit. |
| AC02 | Un submodule absent ou non initialisé | Le développeur lance `make help` | Le diagnostic existant du submodule est conservé ; la version attendue est signalée comme indéterminable et aucune fausse conformité AIDD n’est affichée. |
| AC03 | Un clone sans AIDD et les prérequis disponibles | Le développeur lance `make aidd-full-setup` | La version définie par le COE est installée localement ; les skills, agents, hooks/plugins et configurations requis sont présents selon la distribution attendue ; le diagnostic réussit. |
| AC04 | Une installation conforme | Le développeur lance `make help` puis `make aidd-setup` | L’aide indique la conformité et les versions ; l’installation répétée ne télécharge ni ne réécrit inutilement les fichiers conformes. |
| AC05 | Une installation dont un fichier obligatoire manque ou est invalide | Le développeur lance `make help` et le diagnostic détaillé | L’état est signalé comme incomplet/invalide avec une procédure de réparation ; après réparation le diagnostic passe. |
| AC06 | Une version AIDD installée différente de la version fixée par la révision COE courante | Le développeur lance `make help`, puis `make aidd-update` | La différence de versions est visible ; la synchronisation installe la version exacte attendue, sans passer arbitrairement à une version plus récente. |
| AC07 | Un dépôt initialisé avec le script de setup partagé | Un développeur crée un nouveau worktree via Kilo Agent Manager | Le submodule et AIDD sont préparés dans ce worktree sans intervention manuelle ; le diagnostic réussit avant utilisation de l’agent. |
| AC08 | Le worktree créé lors d’AC07 | Le développeur ouvre une nouvelle session Kilo dans ce worktree | Les skills AIDD attendus sont détectables via le mécanisme runtime de Kilo ; les skills prévus pour une invocation utilisateur sont accessibles et un skill AIDD représentatif est exécuté avec succès. La simple présence des fichiers ne suffit pas. |
| AC09 | Deux worktrees utilisant des révisions COE qui fixent des versions AIDD différentes | Chacun est initialisé ou synchronisé | Chaque worktree utilise sa propre version attendue ; aucune modification de l’installation du clone principal ni de l’autre worktree. |
| AC10 | Des skills/règles/configurations spécifiques au produit et éventuellement un script de setup personnalisé | L’installation, la mise à jour et `gitignore-sync` sont exécutés | Les fichiers spécifiques sont préservés ; les fichiers générés restent ignorés ; le script de setup partagé et les fichiers spécifiques prévus restent suivables ; aucun secret n’est ajouté. |
| AC11 | Le gestionnaire d’artefacts est indisponible ou l’installation échoue | Une installation nécessaire est lancée, y compris dans un nouveau worktree | Une erreur exploitable est remontée ; aucune installation incomplète n’est marquée conforme ; une commande de relance/réparation est documentée. |
| AC12 | Un worktree créé avec Git hors de Kilo, ou créé avant le déploiement du script | Le développeur lance `make help`, puis la commande de correction indiquée | Le diagnostic identifie l’absence ou l’incohérence et l’installation manuelle rend AIDD utilisable. |
| AC13 | Un pipeline CI ou une commande de setup Python existante | Les tests de non-régression et le parcours CI sont exécutés | Aucun téléchargement ou installation AIDD non sollicité n’est introduit dans les workflows existants. |
| AC14 | Un projet utilisant l’outillage COE mais non configuré pour AIDD, si cette situation est supportée | Le développeur lance `make help` ou le setup habituel | AIDD n’est pas activé de force ; la politique d’activation existante est respectée. |

## Plan de réalisation proposé

1. Auditer les cibles Makefile, la provenance de la version, les fichiers générés, le périmètre du setup existant et le bloc gitignore.
2. Mutualiser les contrôles locaux d’installation et de version, puis intégrer leur synthèse dans `make help`.
3. Implémenter ou compléter la synchronisation idempotente et la gestion des fichiers générés, avec protection des personnalisations.
4. Distribuer le script de setup Kilo et intégrer sa migration via les mécanismes COE existants.
5. Tester sur un clone neuf, un reclone, un worktree Kilo, deux versions différentes et des cas d’échec ; réaliser au moins un test runtime réel de découverte et d’invocation d’un skill.
6. Mettre à jour la documentation d’onboarding, les commandes de diagnostic et la procédure de mise à jour du submodule.

## Definition of Done

- [ ] Critères d’acceptation applicables validés et preuves de test disponibles.
- [ ] Tests automatisés des contrôles Makefile, des versions, des exclusions Git, de l’idempotence et des erreurs, adaptés à l’architecture existante.
- [ ] Test d’intégration réel dans Kilo Agent Manager, avec création de worktree et validation runtime des skills.
- [ ] Documentation développeur mise à jour : clone/reclone, worktree, mise à jour COE, dépannage.
- [ ] Aucune distribution native AIDD ajoutée aux dépôts produits ou COE ; aucun secret versionné.
- [ ] Aucun changement de stratégie de cache, de distribution ou de gouvernance des versions.
- [ ] Non-régression des usages existants et revue de code par le COE.

## Hors périmètre

- Nouveau système de cache ou optimisation des téléchargements.
- Installation globale des skills AIDD dans le HOME.
- Gestion de canaux stable/expérimental indépendants du submodule COE.
- Refonte du framework AIDD ou de son système de distribution.
- Automatisation universelle des clones Git ou des worktrees créés hors de Kilo.
- Support de nouveaux assistants IA ou de plateformes qui ne sont pas déjà supportées.

## Points à confirmer pendant la réalisation

- Comportement exact des cibles existantes et emplacement de la métadonnée de version installée.
- Liste des artefacts obligatoires selon la version et les personnalisations du COE.
- Mécanisme de distribution/migration du script et coexistence avec les scripts produits.
- Plateformes réellement supportées, disponibilité de Make et des accès au gestionnaire d’artefacts dans l’environnement du setup Kilo.
- Respect du délai d’exécution du setup Kilo dans l’environnement réel.
