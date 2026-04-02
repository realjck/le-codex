# CI/CD

L'intégration continue (CI) et le déploiement continu (CD) sont des pratiques DevOps qui automatisent la vérification et la livraison du code. La CI garantit que chaque modification est automatiquement testée ; le CD automatise le déploiement vers les environnements cibles.

## Concepts clés

- **Pipeline** : séquence d'étapes (stages) qui s'exécutent à chaque push ou pull request.
- **Job** : unité de travail dans un stage (ex. lancer les tests unitaires).
- **Artifact** : fichier produit par un job et transmis aux suivants (ex. un build compilé).
- **Runner / Agent** : machine (physique ou virtuelle) qui exécute les jobs.
- **Environnement** : cible de déploiement — `dev`, `staging`, `production`.

## GitHub Actions

GitHub Actions est la solution CI/CD native de GitHub. Les workflows sont définis dans des fichiers YAML placés dans `.github/workflows/`.

### Structure d'un workflow

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Récupérer le code
        uses: actions/checkout@v4

      - name: Configurer Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Installer les dépendances
        run: npm ci

      - name: Lancer les tests
        run: npm test

      - name: Build de production
        run: npm run build
```

### Variables d'environnement et Secrets

Les secrets sont définis dans **Settings → Secrets and variables → Actions** du dépôt GitHub.

```yaml
steps:
  - name: Déployer
    env:
      API_KEY: ${{ secrets.API_KEY }}
      NODE_ENV: production
    run: npm run deploy
```

### Workflow CI + CD complet (Node.js → VPS)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  # ── 1. Tests ──────────────────────────────────────
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm test

  # ── 2. Build ──────────────────────────────────────
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - name: Sauvegarder le build
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  # ── 3. Déploiement ────────────────────────────────
  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Récupérer le build
        uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - name: Déployer par SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/mon-app
            rm -rf dist/
      - uses: appleboy/scp-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          source: "dist/"
          target: "/var/www/mon-app/"
```

### Workflow pour une image Docker

```yaml
name: Build & Push Docker

on:
  push:
    branches: [main]

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Se connecter à Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build et push de l'image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            monuser/mon-app:latest
            monuser/mon-app:${{ github.sha }}
```

### Déclencheurs courants

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'src/**'          # uniquement si des fichiers src/ sont modifiés

  pull_request:
    types: [opened, synchronize, reopened]

  schedule:
    - cron: '0 6 * * 1'  # tous les lundis à 6h UTC

  workflow_dispatch:       # déclenchement manuel depuis l'interface GitHub
```

### Matrice de tests (plusieurs versions)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci && npm test
```

## GitLab CI

Sur GitLab, le pipeline est défini dans `.gitlab-ci.yml` à la racine du projet.

### Structure de base

```yaml
stages:
  - install
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "20"

cache:
  paths:
    - node_modules/

install:
  stage: install
  image: node:20-alpine
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/

test:
  stage: test
  image: node:20-alpine
  script:
    - npm test

build:
  stage: build
  image: node:20-alpine
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

deploy-production:
  stage: deploy
  image: alpine
  environment:
    name: production
    url: https://mon-site.fr
  only:
    - main
  script:
    - apk add --no-cache openssh-client rsync
    - rsync -avz dist/ $SERVER_USER@$SERVER_HOST:/var/www/mon-app/
```

### Variables dans GitLab

Les variables sont définies dans **Settings → CI/CD → Variables**.

```yaml
deploy:
  script:
    - echo "Déploiement vers $CI_ENVIRONMENT_NAME"
    - echo $DEPLOY_TOKEN | docker login -u $DEPLOY_USER --password-stdin registry.gitlab.com
```

### Variables prédéfinies utiles

| Variable | Description |
|---|---|
| `$CI_COMMIT_SHA` | Hash complet du commit |
| `$CI_COMMIT_BRANCH` | Nom de la branche |
| `$CI_ENVIRONMENT_NAME` | Nom de l'environnement |
| `$CI_PROJECT_NAME` | Nom du projet |
| `$CI_PIPELINE_ID` | ID du pipeline en cours |

## Bonnes pratiques

**Fail fast** : placer les étapes les plus rapides (lint, tests unitaires) en premier pour échouer tôt sans consommer inutilement des ressources.

**Cache les dépendances** : `node_modules`, `vendor/`, `.m2/` — le cache entre les runs réduit considérablement les temps d'exécution.

**Protéger les branches** : configurer des règles de protection sur `main` pour exiger que le pipeline CI passe avant tout merge.

**Environnements distincts** : ne jamais déployer directement en production depuis une branche de feature. Utiliser `staging` comme étape intermédiaire.

**Versionner les actions** : toujours épingler les actions GitHub à une version précise (`@v4`) plutôt qu'à `@latest` pour éviter les régressions silencieuses.

## Conclusion

Pour aller plus loin :
[GitHub Actions — Documentation officielle](https://docs.github.com/fr/actions) · [GitLab CI — Documentation officielle](https://docs.gitlab.com/ee/ci/)
