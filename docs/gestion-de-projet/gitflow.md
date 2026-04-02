# Gitflow

Gitflow est un modèle de branching Git défini par Vincent Driessen. Il organise le développement autour de branches à rôles fixes, adapté aux projets avec des cycles de release planifiés.

---

## Vue d'ensemble

```
main       ──●────────────────────────────●──────────────●──→
              │ tag v1.0                   │ tag v1.1      │ tag v1.1.1
              │                           │               │
develop    ──●──────●──────●──────●───────●───────●───────●──→
              │     │      │      │               │
feature/   ──●─────●       │      │               │
  login                    │      │               │
feature/            ───────●      │               │
  paiement                        │               │
release/                    ──────●               │
  1.1                                             │
hotfix/                                    ───────●
  1.1.1
```

### Rôle de chaque branche

| Branche | Durée | Rôle |
|---|---|---|
| `main` | Permanente | Code en production — chaque commit est une version livrée, taguée |
| `develop` | Permanente | Intégration continue — base de toutes les features |
| `feature/*` | Temporaire | Développement d'une fonctionnalité isolée |
| `release/*` | Temporaire | Stabilisation avant livraison (bugfixes, version bump) |
| `hotfix/*` | Temporaire | Correction urgente en production |

### Règles fondamentales

```
✅ On ne commite jamais directement sur main
✅ On ne commite jamais directement sur develop (sauf petits projets solo)
✅ Les features partent toujours de develop
✅ Les releases et hotfixes mergent dans main ET develop
✅ Chaque merge dans main reçoit un tag de version
```

---

## Initialisation

```bash
# Initialiser gitflow dans un dépôt existant
git flow init

# → Questions interactives (appuyer sur Entrée pour les valeurs par défaut)
# Branch name for production releases: [main]
# Branch name for "next release" development: [develop]
# Feature branch prefix: [feature/]
# Release branch prefix: [release/]
# Hotfix branch prefix: [hotfix/]
# Version tag prefix: [v]
```

```bash
# Sans git flow CLI — initialisation manuelle
git checkout -b develop main
git push -u origin develop
```

### Convention de nommage

```
feature/nom-de-la-feature     feature/authentification-oauth
release/version               release/2.3.0
hotfix/description-du-bug     hotfix/correction-login-csrf
```

---

## Feature

Une feature est une fonctionnalité développée en isolation depuis `develop`.

```
develop  ──●──────────────────●──→
            │                  │
feature/    ●──●──●──●─────────●
  auth
```

### Avec git flow CLI

```bash
# Créer une feature
git flow feature start authentification-oauth
# → crée feature/authentification-oauth depuis develop

# Travailler normalement
git add .
git commit -m "feat(auth): ajouter le flow OAuth2 Google"

# Terminer la feature (merge dans develop, supprime la branche)
git flow feature finish authentification-oauth
# → merge dans develop + supprime feature/authentification-oauth

# Partager la feature (travail en équipe)
git flow feature publish authentification-oauth
git flow feature pull origin authentification-oauth
```

### Équivalent git vanilla

```bash
# Créer
git checkout develop
git checkout -b feature/authentification-oauth

# Merger
git checkout develop
git merge --no-ff feature/authentification-oauth   # --no-ff : préserve l'historique
git branch -d feature/authentification-oauth
git push origin develop
```

> `--no-ff` (no fast-forward) crée toujours un commit de merge, ce qui préserve le contexte de la feature dans l'historique.

---

## Release

La branche release sert à stabiliser le code avant livraison : correction de bugs mineurs, mise à jour du numéro de version, documentation. Aucune nouvelle fonctionnalité.

```
develop  ──●──────────────────────────●──→
            │                          │
release/    ●──●(bugfix)──●(version)───●
  2.3.0                               │
main     ────────────────────────────●──→
                                     tag v2.3.0
```

### Avec git flow CLI

```bash
# Créer la release
git flow release start 2.3.0
# → crée release/2.3.0 depuis develop

# Corrections de dernière minute, version bump
git commit -m "chore(release): bump version 2.3.0"
git commit -m "fix(api): corriger la pagination des résultats"

# Terminer la release
git flow release finish 2.3.0
# → merge dans main (avec tag v2.3.0) ET dans develop
# → supprime release/2.3.0

# Pousser tout
git push origin main develop --tags
```

### Équivalent git vanilla

```bash
# Créer
git checkout develop
git checkout -b release/2.3.0

# Finaliser
git checkout main
git merge --no-ff release/2.3.0
git tag -a v2.3.0 -m "Version 2.3.0"

git checkout develop
git merge --no-ff release/2.3.0

git branch -d release/2.3.0
git push origin main develop --tags
```

---

## Hotfix

Un hotfix corrige un bug critique directement en production. Il part de `main` et doit être répercuté dans `develop`.

```
main     ──●─────────────────────────●──→
            │ tag v2.3.0              │ tag v2.3.1
            │                        │
hotfix/     ●──●(fix)─────────────●──●
  2.3.1                            │
develop  ──────────────────────────●──→
```

### Avec git flow CLI

```bash
# Créer le hotfix depuis main
git flow hotfix start 2.3.1
# → crée hotfix/2.3.1 depuis main

# Corriger le bug
git commit -m "fix(auth): corriger la faille CSRF sur le formulaire de login"

# Terminer le hotfix
git flow hotfix finish 2.3.1
# → merge dans main (avec tag v2.3.1) ET dans develop
# → supprime hotfix/2.3.1

git push origin main develop --tags
```

### Équivalent git vanilla

```bash
# Créer
git checkout main
git checkout -b hotfix/2.3.1

# Corriger
git commit -m "fix(auth): corriger la faille CSRF sur le formulaire de login"

# Merger dans main
git checkout main
git merge --no-ff hotfix/2.3.1
git tag -a v2.3.1 -m "Hotfix 2.3.1"

# Répercuter dans develop
git checkout develop
git merge --no-ff hotfix/2.3.1

git branch -d hotfix/2.3.1
git push origin main develop --tags
```

---

## Conventional Commits

Les Conventional Commits standardisent les messages de commit pour rendre l'historique lisible et automatiser le versioning.

### Format

```
type(scope): description courte

[corps optionnel — explication détaillée]

[footer optionnel — références issues, breaking changes]
```

```
feat(auth): ajouter la connexion via Google OAuth
^    ^       ^
│    │       └─ description en minuscules, impératif, sans point final
│    └────────── scope : module ou composant concerné (optionnel)
└─────────────── type : nature du changement
```

### Types

| Type | Usage | Impact version |
|---|---|---|
| `feat` | Nouvelle fonctionnalité | `MINOR` (1.x.0) |
| `fix` | Correction de bug | `PATCH` (1.0.x) |
| `docs` | Documentation uniquement | — |
| `style` | Formatage, espaces (sans changement logique) | — |
| `refactor` | Refactoring sans nouvelle feature ni bug fix | — |
| `test` | Ajout ou correction de tests | — |
| `chore` | Tâches de maintenance (dépendances, config, CI) | — |
| `perf` | Amélioration de performance | `PATCH` |
| `build` | Système de build, scripts | — |
| `ci` | Configuration CI/CD | — |

### Exemples concrets

```bash
# Feature
git commit -m "feat(panier): ajouter le calcul automatique des frais de port"

# Bug fix
git commit -m "fix(auth): corriger l'expiration du token après déconnexion"

# Avec scope et corps
git commit -m "refactor(api): extraire la logique de pagination dans un helper

Déplacé depuis UserController et ProductController vers utils/pagination.js
pour éviter la duplication."

# Chore
git commit -m "chore(deps): mettre à jour express 4.18 → 4.19"

# CI/CD
git commit -m "ci(github-actions): ajouter le job de tests e2e Playwright"

# Documentation
git commit -m "docs(readme): ajouter les instructions d'installation Docker"
```

### Intégration avec Gitflow

```bash
# Sur une branche feature — commits granulaires
git commit -m "feat(facturation): ajouter le formulaire de saisie IBAN"
git commit -m "feat(facturation): intégrer la validation IBAN côté serveur"
git commit -m "test(facturation): ajouter les tests du validateur IBAN"

# Sur une branche release — uniquement fix et chore
git commit -m "fix(facturation): corriger l'affichage du symbole € sur Safari"
git commit -m "chore(release): bump version 2.4.0"

# Sur une branche hotfix — toujours fix
git commit -m "fix(securite): corriger l'injection SQL dans la recherche produits"
```

---

## Récapitulatif des commandes

| Action | git flow | git vanilla |
|---|---|---|
| Initialiser | `git flow init` | `git checkout -b develop main` |
| Créer feature | `git flow feature start nom` | `git checkout -b feature/nom develop` |
| Terminer feature | `git flow feature finish nom` | merge `--no-ff` dans develop |
| Créer release | `git flow release start x.x.x` | `git checkout -b release/x.x.x develop` |
| Terminer release | `git flow release finish x.x.x` | merge `--no-ff` dans main + develop + tag |
| Créer hotfix | `git flow hotfix start x.x.x` | `git checkout -b hotfix/x.x.x main` |
| Terminer hotfix | `git flow hotfix finish x.x.x` | merge `--no-ff` dans main + develop + tag |
