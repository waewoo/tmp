# Mission — Refonte professionnelle du skill de reporting COE Dev/DevOps

## 1. Rôle et objectif

Agis comme un architecte DevOps/DevEx senior, expert en pilotage de la maturité logicielle, reporting opérationnel, data visualisation et conception de skills pour agents IA.

Ta mission est d'**auditer, améliorer et faire évoluer le skill existant** qui analyse les repositories GitLab des squads accompagnées par le COE Dev/DevOps.

Ce skill produit actuellement plusieurs rapports Markdown. Je veux obtenir un dispositif de reporting professionnel, fiable, synthétique et orienté décision.

Il doit permettre de connaître :

1. Le niveau de maturité actuel des squads.
2. Les médailles obtenues, par domaine et par critère.
3. Les activités réellement réalisées chaque semaine.
4. Les objectifs de maturité fixés au début de l'incrément.
5. Les écarts entre les objectifs et les résultats.
6. La progression historique, semaine par semaine.
7. Les franchissements de médailles et leur date de première observation.
8. Les résultats consolidés en fin d'incrément.

**IMPORTANT : il existe déjà un skill fonctionnel, des règles métier, des métriques et des rapports. Il faut les adapter et les améliorer, pas reconstruire aveuglément un système parallèle.**

## 2. Audit obligatoire de l'existant

Avant toute modification :

- Examiner intégralement SKILL.md et les fichiers associés.
- Identifier les règles métier et les critères de maturité existants.
- Comprendre les médailles, leurs seuils et leurs conditions d'obtention.
- Identifier les scripts, modèles de données, configurations et templates disponibles.
- Examiner les rapports Markdown actuellement générés.
- Comprendre l'organisation des squads, produits et repositories.
- Vérifier les mécanismes de collecte GitLab et les éventuelles intégrations externes.
- Identifier les fonctionnalités déjà opérationnelles, à préserver.
- Identifier les défauts de fond, de forme, de calcul et de maintenabilité.

Réutiliser les mécanismes fiables déjà présents.

Ne pas modifier silencieusement les définitions des médailles, les règles de consolidation ou les critères d'évaluation.

## 3. Organisation et domaines métier

### Hiérarchie

Le modèle métier est :

COE → Squad → Produit → Repository GitLab → Critère évalué.

Une squad peut avoir plusieurs produits.

Un produit peut avoir plusieurs repositories.

La structure des rapports, l'agrégation des métriques et la gestion des objectifs doivent respecter cette hiérarchie.

### Domaines de maturité

Organiser le reporting autour de trois domaines principaux :

**Qualité**
- Ruff
- MyPy
- SonarQube
- Autres contrôles existants appartenant à ce domaine

**Fiabilité**
- Tests unitaires
- Tests end-to-end
- Autres contrôles existants appartenant à ce domaine

**Sécurité**
- Fortify
- Autres contrôles de sécurité existants

Cette répartition constitue le référentiel fonctionnel souhaité.

Vérifier son adéquation avec les règles du skill actuel. Conserver les critères et seuils existants, sauf arbitrage explicite.

Les contrôles doivent être extensibles et configurables.

### Médailles

Les niveaux de médaille doivent pouvoir être suivis à la granularité réellement supportée par le référentiel :

- Critère (Ruff, MyPy, Fortify, etc.).
- Repository si pertinent.
- Produit selon les règles de consolidation existantes.
- Squad selon les règles de consolidation existantes.
- Domaine si une règle d'agrégation existe.

Ne jamais inventer une médaille globale ou une moyenne de médailles sans règle métier explicite.

## 4. Objectifs des squads au début de chaque incrément

**Nouvelle exigence majeure.**

Au début de chaque incrément, les squads définissent les objectifs de maturité qu'elles souhaitent atteindre.

Exemples :
- Squad Alpha : atteindre Bronze sur Ruff.
- Squad Alpha : atteindre Argent sur Fortify.
- Squad Beta : atteindre Argent sur MyPy.
- Squad Gamma : atteindre Bronze sur les tests end-to-end.

Il faut pouvoir transmettre ces objectifs au skill sans modifier son code.

### Fichier YAML standardisé

Étudier et proposer un fichier de configuration YAML simple, lisible, versionnable et maintenable.

Il doit permettre de définir :

- L'identifiant de l'incrément.
- Son nom éventuel.
- Ses dates de début et de fin.
- Ses sprints et leurs dates, si disponibles.
- Les squads concernées.
- Les objectifs de chaque squad.
- Le domaine concerné : qualité, fiabilité ou sécurité.
- Le critère concerné : Ruff, MyPy, Fortify, etc.
- La médaille cible.
- Le périmètre de l'objectif : squad entière, produit ou repositories spécifiques lorsque nécessaire.
- Une échéance optionnelle si elle diffère de la fin d'incrément.
- Un identifiant stable permettant de suivre l'objectif dans le temps.

Les objectifs doivent respecter les niveaux réellement définis par le skill.

**Ne définis pas immédiatement un nouveau format si un mécanisme équivalent existe déjà.**

Inspecte les configurations existantes, puis propose le schéma YAML le plus simple et le plus compatible possible.

Prévoir :
- Un exemple documenté.
- Une validation du schéma.
- Des messages d'erreur compréhensibles.
- Une gestion explicite des objectifs absents.
- La détection des critères et médailles inconnus.
- La détection des squads ou produits non référencés.
- La prise en charge de plusieurs objectifs par squad.
- Une conservation de la référence initiale et des objectifs validés pour chaque incrément.

Éviter les duplications de configuration.

Un changement d'objectif en cours d'incrément doit être visible et traçable. Il ne doit pas effacer silencieusement l'engagement initial.

### Suivi des objectifs

Pour chaque objectif, déterminer :

1. Médaille au début de l'incrément.
2. Médaille cible.
3. Médaille actuellement observée.
4. Activités pertinentes détectées.
5. Évolution depuis le début de l'incrément.
6. Atteinte ou non de la cible.
7. Date de première observation du niveau cible.
8. Écart restant si la cible n'est pas atteinte.

Distinguer les statuts :
- Atteint.
- Non atteint.
- Non évaluable.
- Non applicable, si justifié.

Le suivi des activités doit rester indépendant du statut d'atteinte de la médaille.

Une squad peut avoir réalisé des travaux pertinents sans atteindre sa cible.

Une squad peut avoir atteint sa cible sans activité détectée pendant la dernière semaine.

## 5. Architecture des rapports

Concevoir un ensemble cohérent de rapports Markdown, avec trois niveaux complémentaires.

### A — Dashboard global COE

Public : managers, responsables COE, gouvernance.

Objectif : comprendre l'état et la trajectoire des squads en moins de deux minutes.

Présenter :
- Périmètre et couverture de collecte.
- Situation générale des domaines Qualité, Fiabilité et Sécurité.
- Médailles actuelles des squads, selon les règles existantes.
- Objectifs principaux de l'incrément.
- Avancement vers les objectifs.
- Activité de la dernière semaine.
- Nouvelles médailles obtenues ou nouvellement observées.
- Régressions confirmées.
- Évolutions significatives.
- Points de vigilance et éventuels arbitrages.

Le dashboard doit être court et immédiatement exploitable.

Éviter de répéter les informations détaillées déjà accessibles dans les fiches squads.

### B — Suivi hebdomadaire et trajectoire d'incrément

Public : COE, PO, Tech Leads, participants aux synchronisations.

Ce document est le support principal de la réunion hebdomadaire.

**Matrice d'activité**

Présenter les squads en lignes et les critères en colonnes, regroupés logiquement par domaine :

- Qualité : Ruff, MyPy, SonarQube.
- Fiabilité : TU, E2E.
- Sécurité : Fortify et critères existants.

Utiliser des statuts courts et homogènes :
- Activité intégrée et confirmée.
- Travaux identifiés mais non intégrés.
- Aucun changement détecté.
- Non applicable.
- Données indisponibles.

Chaque activité affirmée doit reposer sur des preuves GitLab.

**Chronologie**

Présenter l'évolution semaine par semaine :
- Travaux réalisés.
- Évolution des critères.
- Changements de médailles.
- Objectifs atteints.
- Régressions.
- Points bloquants.

**Suivi de l'incrément**

Afficher les objectifs et leur avancement.

Distinguer :
- Travaux attendus.
- Travaux réalisés.
- Amélioration des métriques.
- Médaille obtenue.
- Objectif atteint.

Une activité ne prouve pas automatiquement une amélioration.

Un changement technique ne prouve pas automatiquement l'obtention d'une médaille.

Une absence de commits détectés ne prouve pas l'absence totale de travail.

### C — Fiches détaillées par squad

Public : PO, Tech Leads, développeurs.

Chaque fiche doit permettre de comprendre :

- État actuel de la squad.
- Produits et repositories concernés.
- Médailles par domaine et par critère.
- Objectifs de l'incrément.
- Progression hebdomadaire.
- Réalisations observées.
- Écarts aux cibles.
- Résultats détaillés des contrôles.
- Liens directs vers les preuves.
- Actions prioritaires factuelles.

La navigation doit respecter :

Squad → Produit → Repository → Critère → Preuve.

Ne pas générer inutilement un fichier séparé pour chaque repository si un tableau dans la fiche squad suffit.

## 6. Historisation et événements de maturité

Le reporting doit conserver les résultats des évaluations successives.

Étudier les mécanismes historiques déjà présents avant d'en ajouter de nouveaux.

Prévoir, si nécessaire, des snapshots structurés persistants contenant :

- Date et période d'évaluation.
- Incrément et sprint.
- Périmètre analysé.
- Résultats des critères.
- Médailles observées.
- Version des règles.
- Objectifs de référence.
- Sources des données.
- Erreurs et données indisponibles.

L'historique doit permettre de reconstruire :
- L'état initial d'un incrément.
- La progression hebdomadaire.
- Les franchissements de médailles.
- Les régressions.
- La réalisation des objectifs.
- Le bilan final.

**Précision historique obligatoire :**

Distinguer la date de première observation d'une médaille de sa date exacte d'obtention, lorsqu'elle n'est pas démontrable.

Ne pas reconstruire fictivement un historique manquant.

Une évolution provoquée par un changement de référentiel ou de périmètre doit être distinguée d'une réelle amélioration technique.

Les snapshots doivent être reproductibles, cohérents et protégés contre les réécritures involontaires.

## 7. Règles d'analyse GitLab

Les rapports doivent analyser les sources réellement accessibles :

- Commits et diffs.
- Merge requests ouvertes ou fusionnées.
- Configurations des outils.
- Fichiers de tests.
- Pipelines et jobs.
- Résultats et métriques accessibles.
- Intégrations SonarQube, Fortify ou autres, si elles sont effectivement disponibles.

Ne jamais prétendre disposer de métriques externes qui ne sont pas accessibles.

Séparer :
- Conformité actuelle.
- Activité sur la période.
- Évolution des résultats.
- Atteinte des objectifs.

Toute activité confirmée doit être reliée à une preuve.

Gérer correctement les branches, périodes de référence, dates de fusion et changements de périmètre.

Ne pas interpréter une pipeline verte comme une amélioration automatique de qualité.

Ne pas confondre zéro, donnée manquante, contrôle non applicable et collecte en échec.

## 8. Agrégation des données

Respecter les niveaux COE, Squad, Produit et Repository.

Réutiliser les méthodes de consolidation existantes lorsqu'elles sont définies et cohérentes.

Ne jamais faire de moyenne arithmétique arbitraire de médailles ordinales.

Ne pas masquer les écarts importants entre produits derrière une moyenne globale.

Les comparaisons temporelles doivent utiliser des périodes, règles et périmètres compatibles.

Si une agrégation existante pose un problème métier important, la documenter et demander mon arbitrage avant d'en modifier le comportement.

## 9. Exigences éditoriales et visuelles

Je souhaite un rendu comparable à un véritable dashboard professionnel de pilotage opérationnel.

Les rapports Markdown doivent être :

- Sobres et élégants.
- Structurés et lisibles.
- Visuellement homogènes.
- Courts sans perdre les informations essentielles.
- Précis et factuels.
- Orientés progression, risques et décisions.
- Exploitables aussi bien par le management que par les équipes techniques.

Privilégier :
- Des KPI pertinents.
- Des tableaux compacts.
- Des titres courts.
- Une terminologie stable.
- Des liens vers les détails.
- Des chronologies faciles à lire.
- Des faits significatifs plutôt que des commentaires génériques.

Éviter :
- Les introductions et conclusions de remplissage.
- Les commentaires artificiellement positifs.
- Les recommandations génériques.
- Les emojis décoratifs excessifs.
- Les tableaux trop larges.
- Les répétitions.
- Les paragraphes qui reformulent simplement les données.
- Les graphiques complexes non compatibles avec le lecteur Markdown cible.

Chaque information doit répondre à une question utile :

« Où en sommes-nous ? »

« Qu'avons-nous réellement fait ? »

« Quel objectif poursuivons-nous ? »

« Quand avons-nous obtenu une médaille ? »

« Progressons-nous vers les objectifs de l'incrément ? »

« Quelles décisions ou actions sont nécessaires ? »

Si une information n'apporte aucune valeur à ces questions, supprime-la.

## 10. Architecture technique, collecte GitLab et maintenabilité

### Principe fondamental : privilégier les traitements déterministes et scriptés

Le skill utilise GitLab MCP et, lorsque nécessaire, l'API REST GitLab pour collecter les informations sur les squads, les produits et leurs repositories.

Il dispose déjà de scripts de collecte et de traitement.

**Règle impérative : dès qu'un traitement est déterministe, répétitif et automatisable, privilégier un script versionné et réutilisable plutôt qu'une succession d'appels MCP ou REST exécutés directement par l'agent.**

Cette règle s'applique notamment :
- À la collecte des projets, repositories, branches, commits et merge requests.
- À l'analyse des changements de fichiers.
- À l'extraction des indicateurs de qualité, sécurité et fiabilité.
- À la récupération des résultats des pipelines et jobs.
- À la détection des activités par domaine.
- Aux calculs de conformité et de médailles.
- À la comparaison des périodes et des incréments.
- Au suivi des objectifs YAML.
- À la génération des snapshots historiques.
- À la préparation des données des rapports Markdown.

### Priorité à la réutilisation de l'existant

Avant de développer une nouvelle fonctionnalité :

1. Rechercher si un script existant réalise déjà tout ou partie du traitement.
2. Réutiliser ce script lorsqu'il répond au besoin.
3. L'étendre proprement lorsqu'une fonctionnalité lui manque.
4. Créer un nouveau script seulement si la responsabilité est réellement distincte ou si la réutilisation rendrait l'architecture moins maintenable.
5. Éviter toute duplication de collecte, de logique métier ou de calcul.

Ne pas multiplier les scripts inutilement.

Ne pas reconstruire un système déjà opérationnel.

### Gestion des informations manquantes

Lorsqu'une nouvelle métrique ou information est nécessaire :

1. Vérifier si elle est déjà collectée par le skill.
2. Identifier sa source réelle dans GitLab ou les intégrations disponibles.
3. Examiner si le MCP GitLab expose cette information.
4. Vérifier si une API REST GitLab ou une autre interface autorisée permet de la récupérer.
5. Adapter prioritairement les scripts de collecte existants.
6. Si nécessaire, créer un composant de collecte réutilisable.
7. Normaliser son résultat dans le modèle de données commun.
8. Ajouter les tests et la documentation nécessaires.
9. Réutiliser ensuite ce mécanisme lors des générations suivantes.

**Ne pas contourner l'absence d'une information dans le modèle de collecte par une série d'appels improvisés à chaque exécution du rapport.**

Les appels directs de l'agent restent possibles pour l'exploration, le diagnostic, la validation ponctuelle ou lorsqu'une intégration scriptable n'est pas techniquement disponible.

### Choix entre MCP et REST

Privilégier la réutilisation des intégrations et conventions existantes.

Utiliser le MCP lorsqu'il fournit la fonctionnalité nécessaire et que son mode d'accès est adapté.

Utiliser l'API REST lorsque la fonctionnalité est absente du MCP, incomplète ou inadaptée à la collecte demandée.

Ne pas supposer qu'un outil MCP disponible à l'agent est automatiquement utilisable depuis un script.

Si l'accès au MCP doit rester orchestré par l'agent, isoler ces appels dans une couche de collecte clairement définie et conserver des traitements déterministes pour la normalisation, les calculs et le rendu.

Ne pas imposer de nouvelle infrastructure MCP lorsque des appels REST existants et fiables répondent déjà au besoin.

### Qualité des scripts

Les scripts doivent être :
- Déterministes pour des entrées et un état des sources identiques.
- Réutilisables indépendamment de la rédaction des rapports.
- Paramétrables par période, squad, produit, repository et incrément.
- Testables sans interroger systématiquement GitLab.
- Idempotents lorsque leur fonctionnement implique des écritures.
- Robustes aux erreurs réseau, aux données manquantes et aux réponses partielles.
- Compatibles avec les conventions du dépôt existant.
- Documentés de manière concise.

Prendre en charge les mécanismes nécessaires :
- Pagination des API.
- Gestion raisonnable des limitations de débit.
- Retry contrôlé sur les erreurs transitoires.
- Gestion des délais d'attente.
- Contrôle des permissions.
- Journalisation technique exploitable.
- Évitement des collectes redondantes.
- Validation des données collectées.

Privilégier les accès en lecture seule à GitLab.

Ne jamais exposer de tokens, secrets ou informations sensibles dans les logs, snapshots ou rapports.

Ne jamais modifier les repositories des squads pendant une opération de reporting.

### Séparation des responsabilités

Privilégier l'architecture suivante, en l'adaptant à l'existant :

1. **Collecte** : scripts et adaptateurs MCP / REST pour récupérer les données.
2. **Normalisation** : transformation vers un modèle de données commun.
3. **Évaluation** : application des règles Qualité, Fiabilité et Sécurité et calcul des médailles.
4. **Objectifs** : lecture et validation des objectifs YAML de l'incrément.
5. **Historisation** : snapshots et détection des évolutions.
6. **Consolidation** : agrégation COE → Squad → Produit → Repository.
7. **Restitution** : génération déterministe des tableaux et rapports Markdown.
8. **Synthèse IA** : mise en avant des faits significatifs, uniquement sur la base des résultats vérifiés.

Les scripts doivent produire des données structurées et stables.

La génération Markdown doit s'appuyer sur des templates cohérents et réutilisables.

L'agent IA ne doit pas recalculer mentalement les métriques déjà calculables par un script.

### Maîtrise des coûts et des performances

Éviter les appels réseau répétés pour récupérer les mêmes informations.

Privilégier la mutualisation et le cache lorsqu'ils sont pertinents, sans compromettre la fraîcheur des données.

Les collectes doivent pouvoir être limitées aux périodes et périmètres nécessaires.

Les règles de fraîcheur et les éventuelles données mises en cache doivent être explicites.

Une indisponibilité de GitLab ou d'une source ne doit jamais conduire à présenter des données anciennes comme des mesures actuelles.

### Évolutivité et validation

Toute nouvelle métrique doit idéalement être ajoutée à la chaîne de collecte existante, puis devenir disponible automatiquement pour les futures exécutions.

Ajouter les tests appropriés sur :
- La collecte.
- La normalisation.
- Les calculs.
- Les agrégations.
- Les changements de médailles.
- Les objectifs.
- La génération des rapports.
- Les erreurs et absences de données.

Conserver la compatibilité avec les scripts et configurations existants.

Si une nouvelle exigence nécessite une transformation technique importante ou contredit les choix d'architecture actuels, expliquer la contradiction, proposer les options et demander mon arbitrage avant de modifier le fonctionnement concerné.

**Règle finale : l'IA doit améliorer et enrichir les outils de collecte existants plutôt que répéter indéfiniment des opérations déterministes à travers des appels directs. Elle orchestre la chaîne de traitement ; les scripts assurent la collecte répétable, les calculs et la production structurée des données.**

## 11. Gestion des contradictions et arbitrages

**Règle essentielle : ne prends pas seul de décisions qui changeraient significativement le fonctionnement métier existant.**

Si tu identifies une contradiction forte entre cette demande et le fonctionnement actuel, signale-la avant d'implémenter la modification concernée.

Exemples :
- Modèle de médailles existant incompatible avec la granularité demandée.
- Règles de consolidation incompatibles.
- Organisation des produits différente du modèle supposé.
- Méthode historique existante incompatible avec les snapshots proposés.
- Objectifs déjà gérés dans un autre format.
- Changement susceptible de casser une fonctionnalité actuellement utilisée.
- Ambiguïté importante sur l'évaluation d'un objectif.

Dans ces cas :

1. Explique brièvement le fonctionnement actuel constaté.
2. Explique la contradiction avec la nouvelle demande.
3. Présente les options possibles.
4. Indique ta recommandation et ses impacts.
5. Pose-moi la question précise permettant de décider.

**Ne modifie pas les règles métier concernées avant ma réponse.**

En revanche, ne bloque pas inutilement l'ensemble du travail pour des détails mineurs. Utilise ton jugement sur les décisions techniques non structurantes et documente-les.

Privilégie systématiquement l'évolution incrémentale et rétrocompatible.

## 12. Implémentation et validation

Procède dans cet ordre :

1. Auditer le skill et ses rapports.
2. Identifier les fonctionnalités à conserver.
3. Présenter brièvement l'architecture cible et les écarts.
4. Remonter les contradictions fortes nécessitant un arbitrage.
5. Définir le schéma YAML des objectifs en tenant compte de l'existant.
6. Implémenter les changements validés.
7. Adapter les templates Markdown.
8. Intégrer le suivi historique et les franchissements de médailles.
9. Générer des rapports représentatifs à partir des données disponibles.
10. Tester et vérifier l'absence de régression.

Tester notamment :
- Une squad avec plusieurs produits.
- Un produit avec plusieurs repositories.
- Plusieurs objectifs pour une même squad.
- Un objectif différent selon le produit.
- Un objectif Bronze Ruff et Argent Fortify.
- Un objectif atteint en cours d'incrément.
- Un objectif non atteint malgré une activité confirmée.
- Une médaille atteinte sans changement pendant la semaine.
- Une régression.
- Un changement de périmètre.
- Une modification d'objectif en cours d'incrément.
- Un changement de référentiel de maturité.
- Un nouvel incrément.
- L'absence d'historique.
- Les données GitLab manquantes.
- La compatibilité avec les configurations existantes.

Ne supprime pas d'anciens fichiers ou mécanismes sans vérifier les références et les usages.

## 13. Critères d'acceptation

La refonte est réussie si :

- Un manager comprend l'état global des squads en moins de deux minutes.
- Je peux animer mes points hebdomadaires directement depuis le reporting.
- Les domaines Qualité, Fiabilité et Sécurité sont clairement identifiables.
- Les objectifs des squads sont configurables via YAML.
- Les objectifs sont suivis pendant tout l'incrément.
- Les médailles et leurs évolutions sont visibles.
- Les franchissements de niveau sont historisés.
- Les travaux réalisés sont distingués des résultats obtenus.
- Les fiches permettent de descendre jusqu'au repository et à la preuve.
- Les rapports sont professionnels, concis et cohérents.
- Les calculs sont fiables et reproductibles.
- Les fonctionnalités existantes sont préservées.
- Les contradictions métier importantes sont soumises à mon arbitrage.

**Principe directeur : construire un véritable outil de pilotage de la maturité DevOps, avec le minimum de bruit et le maximum d'informations exploitables.**

Ne te limite pas à proposer des recommandations : audite, adapte, implémente et valide les améliorations compatibles avec l'existant. Suspends uniquement les changements qui nécessitent un arbitrage métier explicite.
