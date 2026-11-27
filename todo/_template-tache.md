# Template — Tâche technique ou handoff pour un agent IA

> Ce fichier est le modèle de référence pour créer ou mettre à jour une tâche dans `waewoo/tmp/todo/`. Ne pas le traiter comme une tâche à exécuter. Lire les consignes ci-dessous, puis copier la structure et remplacer tous les placeholders.

## Consignes de rédaction et de gestion

1. **Nom du fichier** : utiliser un verbe d'action à l'infinitif suivi de l'objet, en français, sans accents, en minuscules et kebab-case : `evaluer-aidd-5.8.0.md`, `mettre-en-place-presentations-html-confluence.md`. Une version peut conserver ses points. Pas de préfixe handoff, ticket ou todo redondant. Ce fichier `_template-tache.md` est l'exception.
2. **Avant création** : parcourir le dossier `todo/` et lire les tâches proches ; compléter l'existante si elle couvre le même objectif. Ne pas écraser une tâche différente. Vérifier les liens publics et les versions si l'information peut évoluer.
3. **Structure commune** : garder les métadonnées et les sept sections du modèle ci-dessous, dans cet ordre. Utiliser des sous-sections pour les détails longs. Rédiger en français, avec identifiants techniques dans leur langue originale.
4. **Autonomie technique** : conserver tous les éléments utiles déjà fournis par le demandeur ou issus d'une analyse précédente. Ne pas réduire un handoff à une simple checklist. Décrire le pourquoi, le fonctionnement, les dépendances, les chemins de fichiers, les commandes et options, les snippets de configuration, les interfaces, les contraintes, les alternatives et leurs compromis, les erreurs connues et les limites. Inclure les liens directs vers les docs officielles, PR, issues, releases et fichiers versionnés. Distinguer ce qui est confirmé, proposé, à vérifier ou non supporté, avec date de vérification si pertinente. Ne pas inventer de version, d'option, de résultat de test ou de capacité. Marquer les exemples non exécutés comme tels.
5. **Travaux** : utiliser une checklist Markdown `- [ ]`, avec verbes d'action, ordre logique et dépendances. Ne cocher que les actions réalisées avec preuve. Ajouter un sous-plan si le travail nécessite plusieurs phases.
6. **Acceptation et preuves** : critères concrets, testables et reliés aux actions ; décrire préconditions, cas nominal, erreurs, réexécution/idempotence, sécurité et rollback lorsque pertinents. Indiquer la preuve attendue. Statuts de validation : PASS, FAIL, PARTIAL, BLOCKED, NOT RUN. Ne jamais déclarer PASS sans exécution ou preuve réelle ; documenter les blocages et le moyen de les lever.
7. **Livrables et exécution** : préciser les fichiers, tests, rapports, décisions et résultats attendus, ainsi que l'ordre des étapes. Pour une IA d'entreprise, demander de lire les règles du dépôt (AGENTS.md, workflow AIDD, conventions CI si applicables), de travailler dans un périmètre autorisé, de réaliser les actions locales possibles et de proposer les déploiements pour revue. Ne pas autoriser implicitement les actions destructives ou publications externes.
8. **Confidentialité** : le dépôt tmp est public. Aucun secret, identifiant privé, URL interne, donnée d'entreprise, capture sensible, nom de client ou configuration privée. Employer des exemples génériques et des placeholders explicites ; utiliser des secrets approuvés uniquement dans le périmètre interne. Ne jamais contourner TLS, permissions ou CSP.
9. **Métadonnées** : statut initial « à faire », priorité et échéance « à définir » si elles ne sont pas connues, date ISO YYYY-MM-DD. Ne pas inventer une échéance ni modifier le statut sans preuve.
10. **Avant commit** : vérifier les sept sections, les checklists, l'exactitude des références et la conservation des détails techniques. Pour une révision, comparer avec le fichier initial ; ne supprimer une information que si elle est obsolète et expliquer pourquoi. Pousser uniquement les changements demandés dans le dépôt/branche autorisés, puis donner le lien et signaler les hypothèses, limites ou blocages.

---

## Structure à copier

Copier à partir du titre ci-dessous jusqu'à la fin, sans les consignes ci-dessus. Remplacer tous les placeholders ; indiquer « non applicable » avec justification si nécessaire.

# [Verbe d'action + objectif concret]

- Statut : à faire
- Priorité : à définir
- Créée le : YYYY-MM-DD
- Échéance : à définir

## Objectif

[Contexte, problème à résoudre, résultat attendu, périmètre et hors périmètre.]

## Travaux à réaliser

- [ ] [Action 1 : investigation, prérequis, sources.]
- [ ] [Action 2 : mise en œuvre ou POC.]
- [ ] [Action 3 : vérifications, tests, restitution.]

## Détails techniques

### Contexte et prérequis

[Architecture, environnement, dépendances, versions, compatibilité, contraintes et hypothèses.]

### Mise en œuvre

[Procédure ordonnée, commandes exactes ou exemples à valider, chemins, fichiers, extraits de code/configuration, contrats d'interface. Expliquer le comportement attendu et les choix techniques.]

### Limites, risques et alternatives

[Support réel, points à confirmer, sécurité, échecs possibles, stratégie de mise à jour/retour arrière.]

### Scénarios de test

[Cas nominaux, négatifs, limites, répétition, preuves et critères de blocage. Adapter au sujet.]

## Critères d'acceptation

- [ ] **AC01 — [Résultat observable]**. Preuve : [test ou artefact attendu].
- [ ] **AC02 — [Résultat observable]**. Preuve : [test ou artefact attendu].

## Livrables attendus

- [Document, code, tests, rapport, preuve, décision d'architecture ou plan d'adoption.]

## Instructions d'exécution

[Ordre de réalisation et règles pour l'agent. Exécuter ce qui est autorisé, valider les versions et options, préserver l'existant, reporter les résultats réels, hypothèses et blocages. Ne pas publier de données internes dans ce dépôt public.]

## Références publiques

- [Documentation officielle ou lien direct vers une version, une PR, une issue ou un fichier de référence.]
