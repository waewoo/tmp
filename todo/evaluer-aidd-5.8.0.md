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
- [ ] Vérifier la compatibilité des versions des plugins et le statut de la release globale ; identifier les limites Kilo encore ouvertes, notamment #914 (PR #992 fermée sans fusion).
- [ ] Produire une synthèse avec résultats de tests, limitations, recommandations et actions d'intégration éventuelles.

## Détails techniques

### Contrats déterministes et hooks (#1149)

Examiner le moteur de validation des artefacts AIDD : frontmatter, descriptions, structure Markdown, sections et ordre, noms, taille, liens, templates, manifests et configurations des hooks. Vérifier sur la version réellement installée les contrats applicables, le contrôle en fin de tour, les diagnostics fichier/ligne/règle, les limites des corrections et la réutilisation des contrôles en CI. Comparer les contrôles d'artefacts AIDD aux règles d'architecture de Cartulary et aux vérifications de code AST-grep ; ne pas supposer que le moteur valide les règles métier ou le code des squads.

### Installation et cycle de vie (#1142)

```bash
aidd setup --global --ide kilo --plugins all
aidd doctor --global
aidd sync --global
# Nettoyage uniquement sur un environnement de test autorisé :
aidd clean --global --force
```

Tester dans un HOME isolé et vérifier les emplacements réellement utilisés (notamment `~/.config/kilo/`), la coexistence des skills/agents globaux et projet, les conflits de versions, la préservation des fichiers utilisateur et la réversibilité. Comparer avec l'installation pilotée par Makefile/submodule, sans présumer que l'installation globale remplace la stratégie de versionnement par produit.

### Modèles par rôle (#1119)

```bash
aidd setup --ide opencode --provider mistral
# Retour au modèle choisi par l'IDE :
aidd setup --ide opencode --provider none
```

Exemples pour OpenCode, à ne pas transposer tels quels à Kilo Code ou à un fournisseur personnalisé. Examiner `agent-models.json` et la configuration machine, les niveaux smart (reviewer), standard (session principale) et light (explorer, executor, validator). Vérifier les identifiants de modèles et le comportement de la session principale de Kilo, qui reste pilotée par l'IDE d'après la PR. Mesurer la portée inter-projets et le retour arrière dans un environnement isolé.

### Review GitLab et QA (#1128, #1201)

Tester `aidd-vcs:06-answer-comments` avec des discussions de test : états done, answer, user-call et todo, commentaire récapitulatif unique, preuves, décision utilisateur explicite et absence de modification de code par le skill de réponse. Pour QA, contrôler les rapports et enregistrements hors Git, leur publication en commentaire de MR, leur conservation en cas d'échec et la distinction entre verdict QA et succès de publication. Les PR d'origine signalent que les scénarios réels de review et d'upload n'avaient pas été exercés : ne pas considérer ces fonctionnalités comme qualifiées avant essai.

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
