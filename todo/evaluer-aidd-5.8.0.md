# Évaluer AIDD 5.8.0 pour le COE Dev/DevOps

- Statut : à faire
- Priorité : haute
- Créée le : 2026-11-27
- Échéance : à définir

## Objectif

Évaluer les nouvelles fonctionnalités AIDD publiées le 26 novembre 2026 et déterminer leur intérêt pour industrialiser les workflows de développement assisté par IA, notamment avec Kilo Code et GitLab.

## Travaux à réaliser

- [ ] **Contrats déterministes (#1149)** : identifier les artefacts contrôlés, tester les hooks de validation et leur comportement en cas de non-conformité ; évaluer la complémentarité avec Cartulary et AST-grep.
- [ ] **Installation globale (#1142, disponible depuis CLI 5.7.0)** : tester `aidd setup --global --ide kilo --plugins all`, `aidd doctor --global` et `aidd sync --global` ; vérifier la découverte des skills et agents dans un clone et un worktree, la coexistence avec la configuration projet, la mise à jour et le nettoyage ; comparer avec une installation pilotée par Makefile/submodule et proposer une stratégie de versionnement.
- [ ] **Modèles par rôle (#1119)** : examiner `agent-models.json`, tester l'affectation des modèles aux agents et déterminer les possibilités d'adaptation à des fournisseurs personnalisés ; vérifier les limites propres à Kilo Code, notamment le modèle de la session principale.
- [ ] **Review GitLab (#1128)** : tester le skill `06-answer-comments` sur une MR de démonstration ; vérifier les réponses, preuves, décisions en attente, absence de doublons et résolution des discussions.
- [ ] **Preuves QA (#1201)** : tester la publication des preuves dans une MR GitLab, la conservation locale en cas d'échec et l'absence de fichiers vidéo dans l'historique Git.
- [ ] **Refine et générateurs** : tester les nouveaux rapports de challenge/brainstorm, la conservation des rounds, et les générateurs de skills, rules, agents, commandes et hooks allégés ; vérifier les cibles natives Kilo réellement prises en charge.
- [ ] Vérifier la compatibilité des versions des plugins et le statut de la release globale ; identifier les limites Kilo encore ouvertes, notamment #914 (PR #992 fermée sans fusion).
- [ ] Produire une synthèse avec résultats de tests, limitations, recommandations et actions d'intégration éventuelles.

## Détails techniques

### Contrats déterministes et hooks (#1149)

Examiner le moteur de validation des artefacts AIDD : frontmatter, descriptions, structure Markdown, sections et ordre, noms, taille, liens, templates, manifests et configurations des hooks. Vérifier sur la version réellement installée les contrats applicables, le contrôle en fin de tour, les diagnostics fichier/ligne/règle, les limites des corrections et la réutilisation des contrôles en CI. Comparer les contrôles d'artefacts AIDD aux règles d'architecture de Cartulary et aux vérifications de code AST-grep ; ne pas supposer que le moteur valide les règles métier ou le code des squads.

### Fonctionnement concret des contrats et hooks

Source vérifiée : [spécification artifact-hook, tranche 2a](https://github.com/ai-driven-dev/framework/blob/2a3a53ce801c3f96d8da8e9de75247d8e4f1c4d1/aidd_docs/tasks/2026_11/2026_11_20_artifact-hook/spec.md), [PR #1317](https://github.com/ai-driven-dev/framework/pull/1317). Distinguer cette tranche du périmètre des autres contrats livrés par les plugins.

- Moteur déterministe, sans LLM, réseau ou dépendance externe Node pour les contrôles décrits ; un contrat par type d'artefact, avec périmètre déclaré dans le contrat. Les copies générées nécessaires au hook résident dans le répertoire `hooks/` de chaque plugin ; pas de dépendance entre plugins pour ces copies.
- En fin de tour, le hook examine les artefacts écrits durant le tour et renvoie les diagnostics de type fichier:ligne/règle/correction. Un tour conforme ou sans artefact concerné est silencieux ; une seule réponse par tour, dans le délai du host. Le hook ne bloque pas directement le tool call ni le tour. Distinguer ce retour au LLM d'un contrôle CI bloquant.
- La spécification cible Claude Code, Codex, Copilot, Cursor, OpenCode et Kilo ; Antigravity est exclu faute d'événement de fin de tour. Tester réellement chaque host retenu, notamment Kilo, plutôt que déduire le support d'un test de fixture.
- Périmètre initial de la tranche 2a : DEV (plans, phases, run), PM (user stories), Context (skills des hosts, agents/commandes au format Claude, règles au format neutre). Les formes spécifiques non comprises par un contrat sont ignorées ; notamment TOML, MDC, instructions.md, agent.md et prompt.md dans cette spécification. Des contrats supplémentaires existent dans les releases : inventorier la couverture exacte de chaque plugin et ne pas supposer que tout contrôle de contrat est activé dans le hook utilisateur.
- Limite : une modification humaine simultanée à un tour dans lequel l'agent lance une commande shell peut être assimilée à une écriture du modèle. Tester les modifications manuelles hors tour, les artefacts multi-fichiers, les formats ignorés, les fichiers hors périmètre, la gestion d'un contrat cassé et les retours répétés.
- Contrôler dans le dépôt source les scripts de validation réellement présents sous `scripts/check/`, les cibles Makefile et Lefthook/CI ; relever les commandes exactes. Ne pas traiter les exemples de l'issue #1149 comme des commandes finales garanties.

Matrice de test minimale : artefact valide (silence), artefact invalide (diagnostic précis), correction puis nouveau contrôle, artefact non couvert (pas de fausse assurance), fichier écrit manuellement hors tour, même contenu contrôlé deux fois (sortie identique), contrôle CI sur un artefact invalide, installation à plat et globale selon les surfaces disponibles. Prévoir un exemple AIDD et un exemple de règle AST-grep sur code pour distinguer les deux périmètres.

### Installation et cycle de vie (#1142)

```bash
aidd setup --global --ide kilo --plugins all
aidd doctor --global
aidd sync --global
# Nettoyage uniquement sur un environnement de test autorisé :
aidd clean --global --force
```

Tester dans un HOME isolé et vérifier les emplacements réellement utilisés (notamment `~/.config/kilo/`), la coexistence des skills/agents globaux et projet, les conflits de versions, la préservation des fichiers utilisateur et la réversibilité. Comparer avec l'installation pilotée par Makefile/submodule, sans présumer que l'installation globale remplace la stratégie de versionnement par produit.

#### Points d'architecture à vérifier

- `--global` correspond au scope utilisateur ; `setup`, `sync`, `doctor` et `clean` le prennent en charge. La PR #1142 décrit un manifeste machine et un mécanisme de mise à jour des fichiers appartenant à AIDD.
- OpenCode et Kilo : copie de skills et agents vers `~/.config/opencode/` ou `~/.config/kilo/`. Claude Code, Codex, Copilot : activation native des plugins au scope utilisateur selon le host ; les agents Codex sont copiés à part dans `~/.codex/agents/`. Cursor : plugins utilisateur sous `~/.cursor/plugins/local/`. Vérifier ces chemins dans l'environnement réel, particulièrement sous Windows et en présence de variables de configuration.
- Dans le dépôt du framework, `make check-hosts` vérifie les surfaces réellement chargées ; `make reload` s'appuie sur le setup global. Ne pas supposer que ces cibles existent dans un dépôt produit.
- Comparer trois scénarios : installation par projet, installation globale seule, installation globale pilotée par une version de framework fournie par le submodule COE. Vérifier la reproductibilité, les mises à niveau, les versions simultanées, le rollback, les fichiers non détenus par AIDD, la découverte en session et les clones/worktrees. Examiner la stratégie de coexistence avec les règles et la mémoire propres au projet.
- Prévoir un check Makefile sans effet de bord qui détecte une installation absente ou incompatible et explique la commande de remise en conformité. Ne pas imposer une mise à jour globale silencieuse à tous les projets.

### Modèles par rôle (#1119)

```bash
aidd setup --ide opencode --provider mistral
# Retour au modèle choisi par l'IDE :
aidd setup --ide opencode --provider none
```

Exemples pour OpenCode, à ne pas transposer tels quels à Kilo Code ou à un fournisseur personnalisé. Examiner `agent-models.json` et la configuration machine, les niveaux smart (reviewer), standard (session principale) et light (explorer, executor, validator). Vérifier les identifiants de modèles et le comportement de la session principale de Kilo, qui reste pilotée par l'IDE d'après la PR. Mesurer la portée inter-projets et le retour arrière dans un environnement isolé.

#### Mapping livré dans CLI 5.8.0

Source : [agent-models.json au tag cli-v5.8.0](https://github.com/ai-driven-dev/framework/blob/cli-v5.8.0/cli/src/contexts/tools/domain/formats/models/agent-models.json). Identifiants exacts du catalogue livré, pas une garantie de disponibilité sur une API particulière.

| Tier et rôle | OpenAI | Anthropic | Mistral |
|---|---|---|---|
| smart — reviewer | gpt-6.1-sol / high | claude-opus-5-5 / high | mistral-large-4 / high |
| standard — session principale | gpt-6.1-terra / medium | claude-sonnet-5-5 / medium | mistral-large-4 / medium |
| light — explorer, executor, validator | gpt-6-luna / medium | claude-haiku-5-5 / medium | mistral-small-latest / high |

Le catalogue déclare les providers OpenAI, Anthropic et Mistral pour OpenCode et Kilo, Anthropic pour Claude et Cursor, OpenAI pour Codex, OpenAI/Anthropic pour Copilot. La PR #1119 indique une configuration provider par IDE au niveau machine (`~/.config/aidd/agent-models.json`) et un état de restauration de la session principale (`~/.config/aidd/main-model.json`). Claude et OpenCode disposent d'une écriture de modèle principal dans les réglages utilisateur ; Kilo, Codex, Copilot et Cursor conservent le modèle choisi dans l'IDE pour la session principale. Les agents déjà installés dans d'autres projets peuvent nécessiter un nouveau setup. Une invocation ne sélectionne qu'un provider ; des combinaisons de hosts incompatibles peuvent être refusées.

Tester la sélection et la restauration dans un HOME isolé, le modèle réellement utilisé par chaque agent, le comportement en cas d'échec de setup et l'impact inter-projets. Identifier les éventuels mécanismes documentés de personnalisation du catalogue ; ne pas supposer qu'un provider interne arbitraire est supporté ni modifier un fichier installé sans stratégie de pérennité. Vérifier aussi les limites de budget, latence, qualité et confidentialité des modèles retenus.

### Review GitLab et QA (#1128, #1201)

Tester `aidd-vcs:06-answer-comments` avec des discussions de test : états done, answer, user-call et todo, commentaire récapitulatif unique, preuves, décision utilisateur explicite et absence de modification de code par le skill de réponse. Pour QA, contrôler les rapports et enregistrements hors Git, leur publication en commentaire de MR, leur conservation en cas d'échec et la distinction entre verdict QA et succès de publication. Les PR d'origine signalent que les scénarios réels de review et d'upload n'avaient pas été exercés : ne pas considérer ces fonctionnalités comme qualifiées avant essai.

### Refine, générateurs et compatibilité Kilo

- [Refine 3.3.0](https://github.com/ai-driven-dev/framework/releases/tag/aidd-refine-v3.3.0) : le challenge abandonne le score de confiance, conserve chaque round dans le dossier de tâche et contrôle son rapport par contrat ; les analyses ciblent notamment les chemins d'échec, l'état et les affirmations. Le brief de brainstorm persisté est également validé. Vérifier les boucles bornées, les décisions utilisateur, la cohérence du rapport et la conservation des rounds.
- [Context 2.12.0](https://github.com/ai-driven-dev/framework/releases/tag/aidd-context-v2.12.0) : refonte des générateurs selon un format de skill plus léger (rules #1226, agents #1233, commandes #1243, hooks #1248, learn #1271, génération/validation de skills #1092, routage context-generate #1256). Mesurer sur un scénario identique le contexte chargé et vérifier la qualité, les références et l'idempotence des artefacts produits ; aucune économie de tokens chiffrée n'est présumée.
- [Issue #914](https://github.com/ai-driven-dev/framework/issues/914) et [PR #992](https://github.com/ai-driven-dev/framework/pull/992) : la PR n'a pas été fusionnée. Tester séparément la découverte des skills (`.kilo/skills/`, `.agents/skills/` selon la configuration), les règles (instructions dans `.kilo/kilo.jsonc` ou fichier de configuration réellement propriétaire), les agents (`.kilo/agents/`), les commandes (`.kilo/commands/`), la mémoire (`AGENTS.md`) et les hooks/plugins (`.kilo/plugin/` ou `.kilo/plugins/` selon la version documentée). Ce sont des surfaces Kilo à vérifier et non des garanties de génération par AIDD. Détecter l'absence de doublons et de modifications destructrices des fichiers utilisateur.
- [Dev 2.9.0](https://github.com/ai-driven-dev/framework/releases/tag/aidd-dev-v2.9.0) : contrats des fichiers review, assert et audit. [PM 2.7.0](https://github.com/ai-driven-dev/framework/releases/tag/aidd-pm-v2.7.0) : contrôles des structures et templates, y compris les niveaux de titres. Vérifier les documents réellement générés et la compatibilité avec les conventions du dépôt.

### Versions publiées le 26 novembre 2026

| Composant | Version |
|---|---|
| CLI | 5.8.0 |
| Context | 2.12.0 |
| Dev | 2.9.0 |
| PM | 2.7.0 |
| QA | 1.2.0 |
| Refine | 3.3.0 |
| VCS | 2.6.0 |
| UI (alpha) | 0.4.0-alpha.0 |
| Orchestrator (inchangée) | 2.5.0 |
| Telemetry (inchangée) | 0.5.0 |

La release globale v5.15.0 n'était pas disponible lors de la vérification du 27 novembre 2026 ; vérifier à nouveau avant installation. L'installation globale est disponible depuis CLI 5.7.0. La PR #992 a été fermée sans fusion, l'issue #914 reste ouverte à cette date : distinguer support CLI et couverture des générateurs natifs Kilo. Verrouiller les versions testées, utiliser les sources autorisées et ne publier aucune donnée interne dans ce dépôt public.

## Critères d'acceptation

- [ ] Chaque fonctionnalité est qualifiée : supportée, partiellement supportée ou non supportée, avec preuve reproductible ou limitation explicite.
- [ ] Une recommandation est formulée sur l'installation globale versus projet, la gestion des versions et le comportement dans les worktrees.
- [ ] Les contrôles déterministes et leur complémentarité avec les outils de gouvernance sont documentés.
- [ ] Les workflows GitLab sont testés sur un dépôt de démonstration, ou les blocages sont documentés.
- [ ] Le périmètre réel des contrats par type d'artefact/host, les cas silencieux et les limites du hook sont documentés avec preuves.
- [ ] Les mappings de modèles, leur portée globale, leurs possibilités de personnalisation et le rollback ont été vérifiés ou déclarés bloqués.
- [ ] Les générateurs Kilo et les évolutions Refine sont qualifiés séparément de l'installation CLI.
- [ ] Une décision d'adoption progressive et une liste d'actions de suivi sont proposées.

## Livrables attendus

- Matrice de compatibilité par fonctionnalité et par agent, versions et commandes reproductibles.
- Rapport de tests avec preuves, résultats PASS / FAIL / PARTIAL / BLOCKED / NOT RUN, limites et risques.
- Proposition de stratégie d'installation et de versionnement (globale/projet, Makefile, submodule, clones et worktrees).
- Recommandations pour les contrôles déterministes, la configuration des modèles et l'intégration GitLab, avec plan d'adoption et actions de suivi.

## Instructions d'exécution

Lire les règles du dépôt interne autorisé avant tout test. Réaliser les expérimentations dans un environnement isolé et une MR de démonstration, sans modifier la configuration partagée ou publier de données internes dans ce dépôt public. Valider les options de commande sur la version installée. Distinguer les résultats observés des hypothèses et documenter les blocages ; ne pas attribuer PASS à un test non réalisé. Proposer les changements d'industrialisation pour revue avant déploiement.

## Références publiques

- [CLI 5.8.0](https://github.com/ai-driven-dev/framework/releases/tag/cli-v5.8.0)
- [Releases AIDD](https://github.com/ai-driven-dev/framework/releases)
- [#1149 — Validation déterministe](https://github.com/ai-driven-dev/framework/issues/1149)
- [#1142 — Installation globale](https://github.com/ai-driven-dev/framework/pull/1142)
- [#1119 — Modèles par rôle](https://github.com/ai-driven-dev/framework/pull/1119)
- [#1128 — Réponses aux reviews](https://github.com/ai-driven-dev/framework/pull/1128)
- [#1201 — Publication des preuves QA](https://github.com/ai-driven-dev/framework/pull/1201)
- [#914 — Génération native Kilo](https://github.com/ai-driven-dev/framework/issues/914)
- [Spécification des hooks de validation, tranche 2a](https://github.com/ai-driven-dev/framework/blob/2a3a53ce801c3f96d8da8e9de75247d8e4f1c4d1/aidd_docs/tasks/2026_11/2026_11_20_artifact-hook/spec.md)
- [Catalogue des modèles livré](https://github.com/ai-driven-dev/framework/blob/cli-v5.8.0/cli/src/contexts/tools/domain/formats/models/agent-models.json)
- [PR #992 — Kilo-native generation, fermée sans fusion](https://github.com/ai-driven-dev/framework/pull/992)
- [Refine 3.3.0](https://github.com/ai-driven-dev/framework/releases/tag/aidd-refine-v3.3.0)
- [Context 2.12.0](https://github.com/ai-driven-dev/framework/releases/tag/aidd-context-v2.12.0)
