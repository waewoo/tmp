# Mettre en place les présentations HTML interactives et la publication Confluence as Code

- Statut : à faire
- Priorité : à définir
- Créée le : 2026-11-27
- Échéance : à définir

## Objectif

> Document de mission à transmettre intégralement à un agent de développement interne (Kilo Code, OpenCode ou équivalent).
> Version : 2026-11-27. Ce document est volontairement générique : aucune donnée, URL, identifiant ni configuration privée ne doit être ajouté au dépôt public qui l'héberge.

Tu es un ingénieur Dev/DevOps senior, spécialiste des Agent Skills, du design web, de l'automatisation documentaire et de Confluence. Mets en place et teste une chaîne reproductible permettant de transformer un contenu Markdown ou une présentation existante en une présentation HTML de qualité graphique professionnelle, puis de la publier dans une page Confluence via la macro HTML.

Travaille dans un dépôt de travail interne autorisé, sur une branche dédiée, en respectant les règles et le workflow de contribution existants (AGENTS.md, AIDD, règles d'architecture, conventions Makefile et GitLab CI, le cas échéant). Ne modifie pas les skills tiers pour les adapter si un skill d'orchestration ou un wrapper suffit. Ne te limite pas à proposer des commandes : exécute les étapes autorisées, produis le code et les tests, puis rapporte les preuves réelles. En cas de blocage réseau, de permission ou de configuration, poursuis les étapes locales et indique exactement ce qui manque.

### Contexte et contraintes

- La macro HTML est autorisée sur le Confluence cible (information fournie par le demandeur). Le type/version, la clé de macro, l'exécution JS, la CSP, les éventuels iframes/sandbox et le mode d'authentification restent à confirmer.
- GitLab Pages n'est pas disponible dans l'environnement cible. La consultation doit pouvoir se faire dans Confluence sans hébergement public.
- Approche Skills over MCP : privilégier des Agent Skills et CLI exécutables aussi dans une CI, sans serveur MCP supplémentaire.
- Le dépôt Git interne est la source de vérité ; Confluence est une cible de publication.
- Le POC doit être autonome, puis réutilisable par plusieurs équipes via leur socle de tooling partagé (par exemple submodule). L'intégration transverse et l'activation en production seront soumises à validation.
- Conserver Slidev lorsqu'il est déjà utilisé. Ne pas imposer une migration ; comparer les deux rendus si une présentation de référence est disponible.

## Travaux à réaliser

- [ ] **Phase A — Setup et vérification** : inventorier l'environnement, installer les quatre skills et les CLI approuvés, vérifier leur découverte et l'idempotence.
- [ ] **Phase B — Génération** : créer un deck de démonstration de 6 à 8 slides et les deux rendus depuis une source commune (HTML autonome et fragment Confluence).
- [ ] **Phase C — Validation** : exécuter les tests Playwright, contrôler l'isolation, inspecter les captures et générer les preuves.
- [ ] **Phase D — Publication** : confirmer les prérequis, effectuer un dry-run, tester CSS/JS sur une page autorisée, vérifier le rendu lecteur, la republication et le rollback.
- [ ] **Phase E — Industrialisation** : proposer des scripts réutilisables, un skill d'orchestration léger et une intégration Makefile/CI GitLab, sans déploiement généralisé non approuvé.
- [ ] Restituer les résultats et les limites, avec preuves réelles, recommandations et actions de suivi.

## Détails techniques

### Outillage, versions et contraintes d'installation

| Composant | Responsabilité | Source officielle |
|---|---|---|
| Frontend Design | Direction artistique, composition, typographie | https://github.com/anthropics/skills/tree/main/skills/frontend-design |
| Frontend Slides | Génération d'un deck HTML, navigation, animations | https://github.com/zarazhangrui/frontend-slides |
| Playwright CLI + skill | Contrôle visuel, interactions, captures | https://github.com/microsoft/playwright-cli |
| Playwright | Tests automatisés reproductibles et export PDF si nécessaire | https://playwright.dev/ |
| Confpub CLI + skill | Publication Markdown/macro HTML, pièces jointes, mises à jour | https://github.com/ThomasRohde/confpub-cli ; https://pypi.org/project/confpub-cli/ |
| CLI skills | Installation ciblée des Agent Skills | https://github.com/vercel-labs/skills |
| Kilo Code | Découverte et chargement des skills | https://github.com/Kilo-Org/kilocode/blob/main/packages/kilo-docs/pages/customize/skills.md |
| Confluence | Vérification de la macro et de l'API selon la version | https://confluence.atlassian.com/doc/html-macro-38273085.html |

Les commandes données ci-dessous sont des exemples issus de la documentation publique, pas des commandes déjà testées dans l'environnement interne. Vérifier les versions installées, les options exactes, la disponibilité des sources et les licences. Ne jamais utiliser un téléchargement non validé, un CDN public, un script de déploiement tiers ou une télémétrie non autorisée. Utiliser les miroirs internes et les versions approuvées, verrouillées par version ou commit. Confpub documente le bloc ::: html, la sélection de clé de macro et l'import des scripts/styles référencés ; cela ne prouve pas que le rendu est accepté par le serveur cible.

### Phase A — Inventaire, installation et vérification des skills

1. Lire les règles du dépôt et identifier le système d'installation existant, les versions Node/Python/agent, les registries approuvés, les restrictions proxy/TLS et les outils de test existants. Examiner les licences et les dépendances avant installation.
2. Installer seulement les quatre skills nécessaires et leurs fichiers auxiliaires, dans le périmètre projet. Ne pas installer tout un catalogue ni créer de répertoire .claude si l'agent cible ne l'utilise pas.
3. Exemples pour Kilo (résoudre les versions en amont et remplacer les références flottantes par les références approuvées) :

```bash
npx skills add zarazhangrui/frontend-slides --list
npx skills add anthropics/skills --list
npx skills add zarazhangrui/frontend-slides --skill frontend-slides -a kilo
npx skills add anthropics/skills --skill frontend-design -a kilo
# Installer Playwright CLI via le registry approuvé, à version fixée.
# Paquet : @playwright/cli
playwright-cli --help
playwright-cli install --help
playwright-cli install --skills
# Installer confpub-cli dans un venv ou un environnement isolé.
confpub --help
confpub guide
confpub skill install --help
```

Le CLI skills peut installer les skills Kilo dans .agents/skills selon sa version. Vérifier les chemins réellement supportés par Kilo, notamment .agents/skills et .kilo/skills, sans créer de doublons. Si un installateur génère un format spécifique à un autre agent, récupérer le skill officiel et ses ressources dans le chemin supporté, ou créer un adaptateur minimal documenté. Vérifier le frontmatter name/description, les références relatives, les symlinks et la découverte effective dans une session de l'agent. Pour Playwright, installer un navigateur compatible depuis les sources autorisées, sans désactiver son sandbox. Pour Confpub, installer le skill via l'option officiellement prise en charge ou le copier avec ses références dans le répertoire de skills de l'agent ; vérifier sa présence et sa lecture effective.

4. Fournir une procédure reproductible de setup et de check, idempotente et compatible avec les clones/worktrees. Réutiliser les mécanismes de setup du dépôt plutôt que d'imposer un nouveau framework. Enregistrer versions/commits, provenance, licences, checks et dépendances. Ne pas supprimer de skills existants.

### Phase B — Génération d'une présentation de démonstration

Utiliser des données fictives non sensibles au départ. Si le demandeur fournit une présentation interne de référence, reprendre son contenu à l'identique pour comparer la qualité graphique, sans l'exporter vers un service externe.

Créer 6 à 8 slides en français, avec au moins :
- une couverture et un message clé ;
- une architecture en couches ou un flux de delivery en SVG ;
- une roadmap en phases ;
- un tableau comparatif ou une matrice ;
- un graphique avec chiffres explicitement fictifs ;
- une conclusion et les actions suivantes.

Invoquer Frontend Design pour la direction artistique et Frontend Slides pour l'implémentation. Respecter les ressources du skill mais ne pas exécuter ses fonctions de déploiement public. Préférer une identité cohérente, contraste lisible, typographie locale, variété des mises en page, espacements réguliers, peu de texte par slide, schémas réellement informatifs. Utiliser des SVG et CSS locaux. Pas de décoration gratuite. Les contenus doivent rester modifiables dans Git.

#### Deux rendus issus de la même source

1. Un deck HTML autonome consultable localement, sans internet (fichier unique si faisable). Navigation précédent/suivant, indicateur de progression, clavier lorsque le focus est dans le deck, accessibilité et mode impression. Prévoir un affichage acceptable sans animations.
2. Un fragment HTML compatible avec la macro Confluence. Ne pas coller le document complet avec doctype/html/head/body dans la macro. Générer les deux formats depuis la même source, sans duplication manuelle.

Le fragment doit fonctionner dans le conteneur Confluence (dimension responsive, pas de dépendance à 100vh ou à la totalité de la fenêtre). CSS entièrement préfixé par un identifiant unique du deck ; pas de sélecteurs globaux html/body/:root/* non préfixés, pas de reset global. JavaScript encapsulé dans un module ou une fermeture, initialisation idempotente, pas de variable globale, pas de gestion globale des raccourcis clavier, pas d'accès à l'interface hôte, aux cookies ou aux autres pages. En cas de plusieurs decks sur une page, chaque instance doit rester indépendante. Utiliser des API DOM sûres (textContent, escaping des entrées), aucune exécution de code fourni par les données, aucun eval ni code dynamique.

Aucune requête externe, police distante, iframe externe, CDN, tracking, upload ou analytics. Ne jamais contourner la CSP ou les règles Confluence. Si le JS inline est interdit, étudier un script approuvé en pièce jointe si la politique le permet ; sinon prévoir un rendu statique fidèle. Fournir HTML, PDF et captures des slides si l'export est disponible.

### Phase C — Validation locale et visuelle

Utiliser le skill Playwright pour explorer les rendus et des tests Playwright scriptés pour répéter les contrôles sans LLM en CI. Tester au moins :
- toutes les slides en 1920x1080 et 1280x720, puis une largeur représentative de la zone de contenu Confluence et un petit écran ;
- navigation clavier et boutons, première/dernière slide, progression, redimensionnement, focus, accessibilité, mode impression ;
- absence d'erreurs console, ressources introuvables, requêtes réseau externes, chevauchements, texte tronqué, scroll horizontal involontaire ;
- isolation du fragment dans une page hôte factice contenant ses propres styles, boutons et deux instances du deck ;
- rendu avec animations désactivées et, si prévu, JS désactivé.

Capturer chaque slide, inspecter réellement les captures, corriger les défauts et relancer. Ne pas confondre succès du test DOM et qualité graphique. Si le moteur ne permet pas de visionner les captures, le déclarer et ne pas prétendre avoir fait une revue visuelle. Générer un rapport PASS/FAIL/BLOCKED, les captures et éventuellement le PDF depuis le rendu imprimable, vérifier le nombre de pages et l'absence de slides coupées.

### Phase D — Publication Confluence

#### D1. Vérification des prérequis

Identifier le type/version de Confluence (Cloud ou Data Center), le context path, la clé et la représentation de la macro HTML, la politique CSP, les permissions de l'utilisateur technique et le mode d'authentification autorisé. Confpub prend en charge les deux types de serveur mais la macro peut différer. Examiner une macro fonctionnelle dans une page de test autorisée. Confirmer l'espace et le parent de test, la visibilité et les droits des lecteurs. Si ces paramètres ou une autorisation d'écriture manquent, demander uniquement ce qui est indispensable avant de publier, sans bloquer la préparation locale.

Configurer l'authentification via le mécanisme de secrets approuvé (variables masquées/protégées ou coffre-fort), selon la documentation de la version de confpub installée. Ne jamais demander de coller un secret dans le chat, ne jamais stocker de jeton dans Git, ne pas afficher les en-têtes ou les valeurs des variables. Ne pas désactiver la validation TLS ; utiliser le certificat d'entreprise si nécessaire. Commencer par une requête de lecture à privilèges minimaux.

#### D2. Smoke test minimal puis publication du deck

Générer une page Markdown avec un bloc macro issu du fragment (exemple volontairement minimal, non sensible) :

```markdown
# POC présentation HTML

::: html
<style>
#demo-deck { padding: 1rem; border: 1px solid #53677e; }
#demo-deck .demo-title { font-size: 1.4rem; }
</style>
<div id="demo-deck">
  <h2 class="demo-title">Test de rendu</h2>
  <button type="button" class="demo-button">Tester</button>
  <p class="demo-result" aria-live="polite">Prêt</p>
</div>
<script>
(() => {
  const root = document.getElementById('demo-deck');
  if (!root || root.dataset.ready) return;
  root.dataset.ready = 'true';
  root.querySelector('.demo-button').addEventListener('click', () => {
    root.querySelector('.demo-result').textContent = 'Interaction OK';
  });
})();
</script>
:::
```

Le smoke test emploie un identifiant unique pour une seule instance. Le deck final doit gérer plusieurs instances et ne doit pas dépendre de l'ordre de chargement des macros.

Exemples documentés à valider avec --help avant exécution (les valeurs d'espace/parent sont des placeholders) :

```bash
confpub page publish presentation.md --space "<SPACE_TEST>" --parent "<PARENT_TEST>" --dry-run
# Seulement après validation du plan et de l'autorisation de publier :
confpub page publish presentation.md --space "<SPACE_TEST>" --parent "<PARENT_TEST>"
```

Pour Data Center, la clé est généralement html ; pour Cloud, elle varie selon l'app. Confpub permet un override de clé avec --html-macro-name sur les versions documentées. Ne pas présumer du format de stockage Cloud/Forge. Vérifier réellement le support du dry-run (aucune mutation) et contrôler le plan, la destination et les pièces jointes.

Si le smoke test CSS/JS fonctionne, publier le deck complet dans une page de POC dédiée. Si le CSS est accepté mais pas le JS, tester seulement les mécanismes autorisés ; utiliser des slides statiques en PNG/SVG si aucune exécution n'est possible. Pour les assets séparés, vérifier les scripts/styles joints et leurs URL réécrites par Confpub ; gérer explicitement les autres assets (images, fonts, URLs CSS), vérifier les erreurs et traiter toute ressource manquante comme un échec. Fournir si autorisé le HTML autonome et le PDF en pièces jointes.

#### D3. Vérification post-publication et cycle de vie

Ouvrir la page dans une session navigateur authentifiée autorisée, avec un compte lecteur représentatif. Vérifier CSS, JS, navigation, rendu à la largeur réelle, liens, pièces jointes, absence de conflits avec l'interface, erreurs console et réseau. Ne pas capturer de cookies ni exporter les traces authentifiées hors du périmètre interne. Une réponse API 200 ne valide pas le rendu.

Republier une seconde fois après une modification contrôlée : la même page doit être mise à jour, pas dupliquée. Tester un rerun sans changement : aucun doublon de page ou de pièce jointe, et idéalement aucune nouvelle version inutile (si l'outil ne l'évite pas, documenter et ajouter un contrôle de hash). Utiliser un mapping stable cible/page ID et le lockfile approprié ; ne pas dépendre uniquement du titre. Refuser les conflits et les modifications manuelles inattendues au lieu de les écraser. Prévoir une restauration de la dernière version validée, sans effacer l'historique, et tester le rollback dans le périmètre POC. Ne pas modifier d'autres pages ni permissions.

### Phase E — Industrialisation réutilisable

Après validation du POC, proposer l'intégration à l'outillage transverse (pas de déploiement généralisé sans accord). Réutiliser les conventions du dépôt cible. Exemple d'organisation, à adapter :

```text
docs/presentations/demo/
  content.md
  deck.html             # sortie générée
  confluence.md         # sortie générée
  assets/
tools/presentations/
  setup
  check
  build
  validate
  publish
tests/presentations/
  fixtures/
  presentation.spec.*
.agents/skills/
  confluence-html-presentation/SKILL.md
  # skills tiers selon mécanisme de distribution existant
docs/runbooks/presentations.md
```

Séparer sources et artefacts générés selon la convention du dépôt. Conserver les versions des outils dans un manifeste et les outputs de tests en artefacts CI internes. Les métadonnées de cible privées et états de publication restent dans le périmètre interne.

Créer un skill interne léger confluence-html-presentation si nécessaire : rôle d'orchestrateur, pas de duplication des quatre skills. Son SKILL.md doit expliquer quand l'utiliser, quels skills lire dans quel ordre, les deux sorties attendues, les commandes de validation, les contraintes macro/CSS/JS, les prérequis de publication, le dry-run, la preuve de lecture post-publication et la gestion des échecs. Il ne doit jamais publier automatiquement une page hors cible autorisée ni contourner les règles de contribution AIDD. Le rendre utilisable indépendamment de la génération par LLM.

Proposer des cibles Makefile adaptées (noms indicatifs) :
- presentation-setup : installation des dépendances autorisées.
- presentation-check : validation environnement et découverte skills.
- presentation-build : construction déterministe depuis les sources.
- presentation-validate : tests + captures + export.
- presentation-publish-dry-run : préparation de la publication.
- presentation-publish : publication explicite après validation.

CI GitLab : stages build, validate, publish-plan puis publish manuel/protégé sur branche autorisée. Pas de LLM requis pour publier un HTML déjà construit et validé. Utiliser des dépendances verrouillées, des variables de secrets protégées, des runners approuvés, des artefacts internes et une sérialisation des publications par cible pour éviter les courses. Aucun publish depuis une branche non approuvée ou une MR externe. Bloquer la publication si tests/contrôles réseau/validation des destinations échouent. Pour une CI sans accès navigateur authentifié, prévoir un contrôle de rendu autorisé après publication et un statut distinct ; ne pas considérer la validation complète comme acquise. Le déploiement du template mutualisé est une étape de revue séparée.

## Critères d'acceptation

- [ ] **AC01 — Les quatre skills sont découverts et utilisables, setup réexécutable**. Preuve : Versions, chemins, test réel de chargement.
- [ ] **AC02 — Un deck de 6–8 slides est produit depuis une source versionnée**. Preuve : HTML autonome et fragment macro générés.
- [ ] **AC03 — Le rendu est lisible, cohérent et professionnel**. Preuve : Captures de chaque slide et revue visuelle réelle.
- [ ] **AC04 — Navigation, tailles, accessibilité et impression fonctionnent**. Preuve : Tests Playwright et exports vérifiés.
- [ ] **AC05 — Le deck n'a pas d'appel externe et n'altère pas la page hôte**. Preuve : Test réseau, console, double instance, isolation CSS/JS.
- [ ] **AC06 — Le dry-run cible uniquement l'espace/page de test autorisé**. Preuve : Plan expurgé, preuve d'absence de mutation.
- [ ] **AC07 — Le deck est visible dans Confluence et son interactivité fonctionne**. Preuve : URL interne et preuves de test lecteur ; si JS bloqué, statut PARTIAL/BLOCKED et fallback.
- [ ] **AC08 — La republication est sûre et ne duplique pas la page**. Preuve : ID stable, test de modification et rerun, gestion conflits.
- [ ] **AC09 — Aucune fuite de secrets ou de données ; droits minimaux**. Preuve : Revue sécurité et vérifications automatisées.
- [ ] **AC10 — Les étapes sont réutilisables sans MCP et sans LLM pour la publication**. Preuve : Scripts, documentation, tests, exemple CI.
- [ ] **AC11 — Un retour arrière est documenté et vérifié sur POC**. Preuve : Version restaurée et test de rendu.
- [ ] **AC12 — L'adoption transverse est préparée mais pas imposée**. Preuve : Skill d'orchestration si utile, proposition d'intégration et limites.

Ne jamais attribuer PASS à un critère non testé. Utiliser PASS / FAIL / PARTIAL / BLOCKED / NOT RUN et indiquer les commandes, les résultats et les preuves, sans données confidentielles. Une publication statique de secours n'est pas équivalente à une présentation interactive.

## Livrables attendus

- Code, tests, sources et exemples dans une branche du dépôt interne approprié.
- Deck HTML autonome, fragment compatible Confluence, PDF si possible, captures et rapport de revue.
- Page Confluence de POC si accès et autorisations disponibles, avec preuve de rendu.
- Procédure de setup, build, contrôle, publication, mise à jour et rollback.
- Skill d'orchestration léger et proposition de template CI/submodule si pertinents.
- Tableau d'acceptation complet, versions et décisions d'architecture, limites et recommandations. Comparaison HTML natif vs Slidev sur le même contenu si la source est fournie.
- Synthèse finale : ce qui fonctionne réellement, ce qui a été partiellement réalisé, ce qui reste à faire, risques et proposition de suite. Aucun secret ni URL privée dans le dépôt public d'origine.

## Instructions d'exécution

Commence par lire ce handoff intégralement, analyser le dépôt de travail interne et établir un plan conforme aux règles existantes. Exécute ensuite les phases A à C avec un contenu fictif. Prépare la phase D et ne publie qu'après avoir identifié la cible de test autorisée et vérifié le dry-run. Réalise les livrables d'industrialisation en branche dédiée et présente le résultat pour revue. Si une étape est impossible, ne l'invente pas : fournis les preuves disponibles et la marche à suivre exacte. Ne publie aucune donnée interne vers Internet, même si un skill propose un service de partage public.

## Références publiques

Les liens officiels des huit composants figurent dans le tableau « Outillage, versions et contraintes d'installation » ci-dessus. Vérifier la documentation de la version effectivement installée.
