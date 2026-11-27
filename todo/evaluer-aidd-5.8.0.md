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

## Critères d'acceptation

- [ ] Chaque fonctionnalité est qualifiée : supportée, partiellement supportée ou non supportée, avec preuve reproductible ou limitation explicite.
- [ ] Une recommandation est formulée sur l'installation globale versus projet, la gestion des versions et le comportement dans les worktrees.
- [ ] Les contrôles déterministes et leur complémentarité avec les outils de gouvernance sont documentés.
- [ ] Les workflows GitLab sont testés sur un dépôt de démonstration, ou les blocages sont documentés.
- [ ] Une décision d'adoption progressive et une liste d'actions de suivi sont proposées.

## Références publiques

- [CLI 5.8.0](https://github.com/ai-driven-dev/framework/releases/tag/cli-v5.8.0)
- [Releases AIDD](https://github.com/ai-driven-dev/framework/releases)
- [#1149 — Validation déterministe](https://github.com/ai-driven-dev/framework/issues/1149)
- [#1142 — Installation globale](https://github.com/ai-driven-dev/framework/pull/1142)
- [#1119 — Modèles par rôle](https://github.com/ai-driven-dev/framework/pull/1119)
- [#1128 — Réponses aux reviews](https://github.com/ai-driven-dev/framework/pull/1128)
- [#1201 — Publication des preuves QA](https://github.com/ai-driven-dev/framework/pull/1201)
- [#914 — Génération native Kilo](https://github.com/ai-driven-dev/framework/issues/914)
