# YAML

YAML (YAML Ain't Markup Language) est un format de sérialisation de données lisible par l'humain. Il est omniprésent dans les fichiers de configuration : Docker Compose, CI/CD, Kubernetes, MkDocs, n8n, Ansible…

Règles fondamentales :
- **L'indentation est significative** — utiliser des espaces, jamais des tabulations
- **Sensible à la casse** — `True` ≠ `true`
- Les commentaires commencent par `#`

---

## Syntaxe de base

```yaml
# Ceci est un commentaire

# Paire clé-valeur (mapping)
nom: Alice
age: 30
actif: true

# Séparateur de document (optionnel)
---
autre_document: valeur
```

Un fichier YAML peut contenir plusieurs documents séparés par `---`.

---

## Types de données

```yaml
# Chaînes de caractères
simple: bonjour
avec_espaces: bonjour le monde        # pas besoin de guillemets
guillemets_simples: 'texte : avec : deux-points'
guillemets_doubles: "texte avec\nnewline"

# Nombres
entier: 42
négatif: -7
flottant: 3.14
scientifique: 1.5e10
octal: 0o755
hexadécimal: 0xFF

# Booléens (attention aux variantes reconnues)
vrai_1: true
vrai_2: yes      # ⚠️ reconnu comme booléen en YAML 1.1
faux_1: false
faux_2: no       # ⚠️ idem

# Valeur nulle
vide_1: null
vide_2: ~
vide_3:          # clé sans valeur = null

# Dates (ISO 8601)
date: 2024-01-15
datetime: 2024-01-15T10:30:00Z
```

---

## Structures

### Listes (séquences)

```yaml
# Syntaxe bloc (recommandée)
fruits:
  - pomme
  - banane
  - cerise

# Syntaxe inline (flow)
fruits: [pomme, banane, cerise]

# Liste de mappings
utilisateurs:
  - nom: Alice
    age: 30
  - nom: Bob
    age: 25
```

### Objets (mappings)

```yaml
# Syntaxe bloc
adresse:
  rue: 12 rue des Lilas
  ville: Lyon
  code_postal: 69001

# Syntaxe inline (flow)
adresse: {rue: 12 rue des Lilas, ville: Lyon}
```

### Imbrication

```yaml
application:
  nom: mon-app
  version: "1.0.0"
  serveur:
    host: localhost
    port: 8080
  base_de_données:
    host: db
    port: 5432
    identifiants:
      utilisateur: admin
      mot_de_passe: secret
  tags:
    - web
    - api
    - production
```

---

## Syntaxe avancée

### Chaînes multilignes

```yaml
# Bloc littéral | — conserve les sauts de ligne
description: |
  Première ligne.
  Deuxième ligne.
  Troisième ligne.
# Résultat : "Première ligne.\nDeuxième ligne.\nTroisième ligne.\n"

# Bloc plié > — remplace les sauts de ligne par des espaces
résumé: >
  Ceci est une longue
  phrase sur plusieurs
  lignes dans le fichier.
# Résultat : "Ceci est une longue phrase sur plusieurs lignes dans le fichier.\n"

# Modificateurs de fin de bloc
texte_sans_fin: |−    # supprime le saut de ligne final
texte_avec_fins: |+   # conserve tous les sauts de ligne finaux
```

### Ancres et alias (DRY)

Les ancres (`&`) permettent de définir un bloc réutilisable, les alias (`*`) de le référencer.

```yaml
# Définir une ancre
defaults: &defaults
  restart: always
  logging:
    driver: json-file

# Réutiliser avec fusion (<<)
service_web:
  <<: *defaults
  image: nginx:alpine
  ports:
    - "80:80"

service_api:
  <<: *defaults
  image: mon-api:latest
  ports:
    - "3000:3000"
```

### Types explicites (tags)

Forcer l'interprétation d'une valeur :

```yaml
port_texte: !!str 8080          # forcer en chaîne
valeur_entiere: !!int "42"      # forcer en entier
flag: !!bool "yes"              # forcer en booléen
données: !!null ""              # forcer en null
```

---

## Pièges courants

### Tabulations interdites
```yaml
# ❌ Erreur : tabulation utilisée pour l'indentation
service:
	image: nginx    # tab → erreur de parsing

# ✅ Correct : espaces uniquement
service:
  image: nginx
```

### Chaînes qui ressemblent à d'autres types
```yaml
# ❌ Ces valeurs sont interprétées comme booléens en YAML 1.1
pays: NO        # → false !
reponse: YES    # → true !
statut: on      # → true !

# ✅ Mettre entre guillemets pour forcer la chaîne
pays: "NO"
reponse: "YES"
```

### Deux-points dans une valeur
```yaml
# ❌ Le deux-points est interprété comme séparateur clé-valeur
url: http://exemple.com    # peut provoquer une erreur selon le parser

# ✅ Mettre entre guillemets
url: "http://exemple.com"
```

### Nombres qui commencent par zéro
```yaml
# ❌ Interprété comme octal en YAML 1.1
mode: 0755      # → 493 en décimal !

# ✅ Forcer en chaîne
mode: "0755"
```

### Indentation incohérente
```yaml
# ❌ Indentation incohérente (mélange de 2 et 4 espaces)
parent:
  enfant1: valeur
    enfant2: valeur   # 4 espaces → interprété comme sous-élément de enfant1

# ✅ Indentation cohérente (2 espaces partout)
parent:
  enfant1: valeur
  enfant2: valeur
```

### Valeur vide vs null
```yaml
clé:          # null (pas de valeur)
clé: ""       # chaîne vide (différent de null !)
clé: []       # liste vide
clé: {}       # objet vide
```

---

## Exemples contextualisés

### Docker Compose

```yaml
# docker-compose.yml
services:
  web:
    image: nginx:alpine
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    environment:
      - NGINX_HOST=mon-domaine.com

  api:
    build:
      context: .
      dockerfile: Dockerfile
    restart: always
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: production
      DATABASE_URL: postgres://user:password@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15-alpine
    restart: always
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

  deploy:
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /app
            git pull
            npm ci --production
            pm2 restart app
```

### MkDocs

```yaml
# mkdocs.yml
site_name: Mon Projet
site_url: https://docs.mon-projet.com

theme:
  name: material
  language: fr
  palette:
    - scheme: default
      primary: indigo
  features:
    - navigation.tabs
    - search.highlight

nav:
  - Accueil: index.md
  - Guide:
    - Installation: guide/installation.md
    - Configuration: guide/configuration.md
  - Référence API: api.md

plugins:
  - search
  - mermaid2

extra:
  version:
    provider: mike
```
