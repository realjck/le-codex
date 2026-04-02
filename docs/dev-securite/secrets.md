# Gestion des secrets

Un secret est toute valeur qui donne accès à une ressource protégée : clé API, mot de passe de base de données, token JWT, certificat, clé de chiffrement. Une mauvaise gestion des secrets est l'une des principales causes de compromission d'application.

---

## Principes fondamentaux

```
✅ Jamais dans le code source (même privé)
✅ Jamais dans git — un secret commité est un secret compromis, même supprimé ensuite
✅ Un secret différent par environnement (dev ≠ staging ≠ prod)
✅ Principe du moindre privilège — chaque service n'accède qu'aux secrets dont il a besoin
✅ Rotation régulière — surtout après un départ d'équipe ou un incident
✅ Tracer les accès — qui a lu quel secret, quand
```

```
❌ API_KEY=sk-1234 dans le code
❌ Secrets dans les URLs (GET /api?token=xxx → apparaît dans les logs)
❌ Secrets dans les messages d'erreur ou les logs applicatifs
❌ Partage de secrets par email, Slack ou chat
❌ Même secret pour dev et prod
```

---

## Dev local — dotenv

### Installation

```bash
npm install dotenv
```

### Structure des fichiers

```
projet/
├── .env              # secrets locaux — JAMAIS commité
├── .env.example      # template sans valeurs — commité dans git
├── .env.test         # variables pour les tests (pas de vraies clés)
└── .gitignore        # doit contenir .env
```

```ini
# .env.example — template partagé avec l'équipe
DATABASE_URL=postgres://user:password@localhost:5432/mydb
JWT_SECRET=
STRIPE_SECRET_KEY=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

```ini
# .env — valeurs réelles (local uniquement, non commité)
DATABASE_URL=postgres://alice:monmotdepasse@localhost:5432/mydb
JWT_SECRET=un-secret-fort-de-32-caracteres-minimum
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxx
```

### Chargement dans Node.js

```js
// ✅ Charger dotenv au plus tôt dans le point d'entrée
require('dotenv').config();

// Ou avec ES modules
import 'dotenv/config';
```

```js
// Valider les variables requises au démarrage
function validerConfig() {
  const requis = ['DATABASE_URL', 'JWT_SECRET', 'STRIPE_SECRET_KEY'];
  const manquants = requis.filter(key => !process.env[key]);

  if (manquants.length > 0) {
    throw new Error(`Variables d'environnement manquantes : ${manquants.join(', ')}`);
  }
}

validerConfig(); // fail fast — l'app ne démarre pas si un secret manque
```

```js
// Plusieurs fichiers .env selon l'environnement
require('dotenv').config({ path: ['.env.local', '.env'] });
// .env.local prend la priorité sur .env
```

### Pièges courants

```js
// ❌ Exposer tous les secrets dans les logs
console.log(process.env);

// ❌ Exposer les secrets dans les messages d'erreur
res.json({ erreur: err.message, config: process.env.DATABASE_URL });

// ✅ Ne logger que ce qui est nécessaire
console.log('Connexion DB établie sur', new URL(process.env.DATABASE_URL).hostname);
```

```bash
# .gitignore — toujours inclure
.env
.env.local
.env.*.local
*.key
*.pem
```

### Multilignes (certificats, clés RSA)

```ini
# .env — clé privée sur plusieurs lignes
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA...
-----END RSA PRIVATE KEY-----"

# Ou avec \n explicite
PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----\nMIIEow...\n-----END RSA PRIVATE KEY-----"
```

---

## Docker et Docker Compose

### Variables d'environnement dans Docker Compose

```yaml
# ✅ Option 1 : référencer les variables de l'hôte (sans valeur = hérité du shell ou du .env)
services:
  api:
    image: mon-api:latest
    environment:
      - NODE_ENV=production
      - DATABASE_URL          # hérite de l'environnement hôte
      - JWT_SECRET            # hérite de l'environnement hôte

# ✅ Option 2 : fichier .env séparé (non commité)
services:
  api:
    image: mon-api:latest
    env_file:
      - .env.production       # fichier local non versionné
```

```dockerfile
# ❌ Ne jamais COPY le fichier .env dans l'image
COPY . .          # inclut .env si pas dans .dockerignore !
COPY .env .       # encore pire — explicitement dans l'image

# ✅ .dockerignore
.env
.env.*
*.key
*.pem
```

```ini
# .dockerignore
.env
.env.*
.git
*.key
*.pem
node_modules
```

### Docker secrets (mode Swarm / production)

Docker secrets monte les valeurs dans des fichiers à `/run/secrets/` — jamais dans les variables d'environnement du processus.

```yaml
# docker-compose.yml (Swarm mode)
services:
  api:
    image: mon-api:latest
    secrets:
      - db_password
      - jwt_secret

secrets:
  db_password:
    external: true    # créé via : docker secret create db_password ./db_password.txt
  jwt_secret:
    external: true
```

```js
// Lire un Docker secret depuis Node.js
const fs = require('fs');

function lireSecret(nom) {
  try {
    return fs.readFileSync(`/run/secrets/${nom}`, 'utf8').trim();
  } catch {
    return process.env[nom.toUpperCase()]; // fallback pour le dev local
  }
}

const dbPassword = lireSecret('db_password');
```

---

## CI/CD — GitHub Actions

### Configurer les secrets

Les secrets sont configurés dans **Settings → Secrets and variables → Actions** du dépôt ou de l'organisation.

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production   # secrets spécifiques à l'environnement

    steps:
      - uses: actions/checkout@v4

      - name: Build et déploiement
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
          STRIPE_SECRET_KEY: ${{ secrets.STRIPE_SECRET_KEY }}
        run: npm run deploy
```

### Masquage automatique

GitHub masque automatiquement les secrets dans les logs (`***`). Mais attention :

```yaml
# ❌ Ne jamais afficher explicitement un secret dans un step
- name: Debug
  run: echo "JWT_SECRET=${{ secrets.JWT_SECRET }}"  # masqué mais mauvaise pratique

# ❌ Les secrets ne sont pas disponibles dans les PRs de forks (sécurité intentionnelle)
on:
  pull_request:
    # secrets.* = vide pour les PRs de forks
```

### Environnements (staging vs prod)

```yaml
jobs:
  deploy-staging:
    environment: staging      # utilise les secrets de l'environnement "staging"
    steps:
      - run: npm run deploy:staging

  deploy-prod:
    environment: production   # utilise les secrets de l'environnement "production"
    needs: deploy-staging
    steps:
      - run: npm run deploy:prod
```

### Passer des secrets à Docker dans la CI

```yaml
- name: Build Docker image
  run: |
    docker build \
      --build-arg NODE_ENV=production \
      -t mon-app:latest .
  # ✅ Pas de --build-arg avec les secrets — ils resteraient dans les layers de l'image

- name: Deploy
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
  run: docker run -e DATABASE_URL mon-app:latest
```

---

## Production — secrets managers

Pour les équipes et les applications en production, un secrets manager centralise la gestion, l'audit et la rotation.

### Comparatif

| | HashiCorp Vault | AWS Secrets Manager | Doppler |
|---|---|---|---|
| **Type** | Self-hosted ou cloud | Cloud AWS | SaaS |
| **Rotation auto** | Oui (avec plugins) | Oui (intégration AWS) | Oui |
| **Audit des accès** | Oui | Oui (CloudTrail) | Oui |
| **Intégration CI/CD** | Vault Agent, plugins | Actions AWS | CLI/SDK natif |
| **Complexité** | Élevée | Moyenne | Faible |
| **Idéal pour** | Infrastructure complexe, multi-cloud | Stack AWS | Startups, équipes dev |

### Doppler — exemple d'intégration Node.js

```bash
# Installation du CLI
npm install -g @dopplerhq/cli

# Authentification et liaison au projet
doppler login
doppler setup   # sélectionner projet + environnement

# Lancer l'app avec les secrets injectés
doppler run -- node server.js
```

```yaml
# GitHub Actions avec Doppler
- name: Fetch secrets from Doppler
  uses: dopplerhq/secrets-fetch-action@v1
  id: doppler
  with:
    doppler-token: ${{ secrets.DOPPLER_TOKEN }}
    inject-env-vars: true

- name: Deploy
  run: npm run deploy
  # Les secrets Doppler sont disponibles comme variables d'environnement
```

```js
// En production : les secrets sont injectés par Doppler comme variables d'env
// Le code reste identique — pas de SDK Doppler requis dans l'app
const dbUrl = process.env.DATABASE_URL;
```

### AWS Secrets Manager — exemple Node.js

```bash
npm install @aws-sdk/client-secrets-manager
```

```js
const { SecretsManagerClient, GetSecretValueCommand } = require('@aws-sdk/client-secrets-manager');

const client = new SecretsManagerClient({ region: 'eu-west-1' });

async function obtenirSecret(nom) {
  const commande = new GetSecretValueCommand({ SecretId: nom });
  const reponse = await client.send(commande);
  return JSON.parse(reponse.SecretString);
}

// Au démarrage de l'app (mettre en cache — ne pas appeler à chaque requête)
const secrets = await obtenirSecret('mon-app/production');
const { DB_PASSWORD, JWT_SECRET } = secrets;
```

---

## Détecter les fuites

### Outils de détection

```bash
# trufflehog — scan de l'historique git pour détecter des secrets
npx trufflehog git file://. --only-verified

# gitleaks — alternative légère
brew install gitleaks
gitleaks detect --source . --verbose

# detect-secrets (Python) — intégrable en pre-commit
pip install detect-secrets
detect-secrets scan > .secrets.baseline
```

### Pre-commit hook

```bash
# Installation de pre-commit
pip install pre-commit
```

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

```bash
pre-commit install   # active le hook
# → gitleaks s'exécute automatiquement avant chaque commit
```

### Que faire si un secret est exposé

```
1. Révoquer et régénérer immédiatement
   → Ne pas attendre — considérer le secret comme compromis dès la détection

2. Vérifier les accès
   → Consulter les logs : le secret a-t-il été utilisé par quelqu'un d'autre ?

3. Purger l'historique git
   → git filter-repo (remplace BFG Repo-Cleaner)
   → Forcer le push de toutes les branches concernées

4. Notifier
   → L'équipe, les parties prenantes, et si nécessaire les utilisateurs impactés

5. Post-mortem
   → Comment le secret s'est retrouvé là ? Ajouter une règle pour l'éviter
```

```bash
# Purger un secret de l'historique git avec git filter-repo
pip install git-filter-repo

git filter-repo --replace-text <(echo 'sk_live_XXXXXXXX==>***REMOVED***')

# Forcer le push (après coordination avec l'équipe)
git push origin --force --all
git push origin --force --tags
```

---

## Checklist

| # | Vérification |
|---|---|
| 1 | `.env` dans `.gitignore` et `.dockerignore` |
| 2 | `.env.example` commité avec les clés (sans valeurs) |
| 3 | Validation des variables requises au démarrage de l'app |
| 4 | Secrets différents par environnement (dev / staging / prod) |
| 5 | Secrets GitHub Actions configurés par environnement |
| 6 | Aucun secret dans les logs, erreurs ou URLs |
| 7 | Aucun `COPY .env` dans les Dockerfiles |
| 8 | Pre-commit hook de détection (gitleaks) activé |
| 9 | Secrets manager utilisé en production (Doppler, Vault, AWS SM…) |
| 10 | Procédure de rotation documentée et testée |
