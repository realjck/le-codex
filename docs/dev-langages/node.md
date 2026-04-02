# Node.js

Node.js est un environnement d'exécution JavaScript côté serveur, basé sur le moteur V8 de Chrome. Il permet d'exécuter du JavaScript en dehors du navigateur et est livré avec **npm** (Node Package Manager), le gestionnaire de packages le plus utilisé dans l'écosystème JS.

## Gestion des versions — nvm / nvm-windows

`nvm` (Node Version Manager) permet d'installer et de basculer entre plusieurs versions de Node.js sur une même machine.

- **Linux / macOS** : [github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)
- **Windows** : [github.com/coreybutler/nvm-windows](https://github.com/coreybutler/nvm-windows)

### Installer une version de Node

```bash
# nvm (Linux/macOS)
nvm install 20
nvm install --lts       # dernière version LTS

# nvm-windows
nvm install 20.0.0      # la version doit être complète (major.minor.patch)
nvm install lts
```

### Utiliser une version

```bash
nvm use 20              # Linux/macOS (le major suffit)
nvm use 20.0.0          # Windows (version complète requise)
```

### Lister les versions disponibles

```bash
nvm ls                  # versions installées
nvm ls-remote           # versions disponibles en ligne (Linux/macOS)
nvm list available      # versions disponibles en ligne (Windows)
```

### Définir une version par défaut

```bash
nvm alias default 20    # Linux/macOS
nvm use 20.0.0          # Windows : la version active au démarrage du shell
```

### Version active

```bash
node --version
nvm current             # Linux/macOS
```

---

## npm — Commandes essentielles

### Initialiser un projet

```bash
npm init        # assistant interactif
npm init -y     # accepte tous les défauts, crée package.json directement
```

### Installer des dépendances

```bash
npm install                   # installe toutes les dépendances du package.json
npm install express           # ajoute une dépendance de production
npm install jest --save-dev   # ajoute une dépendance de développement uniquement
npm install -g typescript     # installation globale (disponible partout sur le système)

# Raccourcis
npm i express
npm ci                        # installation propre depuis package-lock.json (CI/CD)
```

### Désinstaller

```bash
npm uninstall express
npm uninstall jest --save-dev
```

### Mettre à jour

```bash
npm update                    # met à jour toutes les dépendances (selon semver)
npm update express            # met à jour un package spécifique
npm outdated                  # liste les packages avec une version plus récente disponible
```

### Exécuter des scripts

```bash
npm run dev         # exécute le script "dev" défini dans package.json
npm run build
npm test            # raccourci pour npm run test
npm start           # raccourci pour npm run start
```

### npx — exécuter sans installer

```bash
npx create-react-app mon-app   # exécute un package sans l'installer globalement
npx tsc --init                 # utile pour les outils CLI ponctuels
```

### Lister les packages installés

```bash
npm list                  # arbre complet
npm list --depth=0        # uniquement les dépendances directes
npm list -g --depth=0     # packages installés globalement
```

---

## npm — Usage avancé

### Audit de sécurité

```bash
npm audit               # liste les vulnérabilités connues
npm audit fix           # corrige automatiquement les vulnérabilités non-breaking
npm audit fix --force   # ⚠️ peut inclure des mises à jour majeures (breaking changes)
```

### Workspaces (mono-repo)

Les workspaces permettent de gérer plusieurs packages dans un seul dépôt.

Configuration dans le `package.json` racine :

```json
{
  "name": "mon-monorepo",
  "workspaces": [
    "packages/*"
  ]
}
```

Commandes ciblant un workspace spécifique :

```bash
npm install --workspace=packages/api
npm run build --workspace=packages/ui
npm run test --workspaces         # exécute dans tous les workspaces
```

### Publication d'un package

```bash
npm login                         # connexion à npmjs.com
npm version patch                 # incrémente la version (patch / minor / major)
npm publish                       # publie sur le registry public
npm publish --access=restricted   # publication privée (compte payant)
```

### package.json vs package-lock.json

| | `package.json` | `package-lock.json` |
|---|---|---|
| **Rôle** | Déclare les dépendances et les scripts | Verrouille les versions exactes installées |
| **Édité par** | Le développeur | npm automatiquement |
| **À commiter** | Oui | Oui (garantit la reproductibilité) |
| **`npm ci` utilise** | Non | Oui (installation stricte) |

---

## package.json

Exemple annoté d'un `package.json` complet :

```json
{
  "name": "mon-projet",
  "version": "1.0.0",
  "description": "Description courte du projet",
  "type": "module",              // "commonjs" (défaut) ou "module" (ESM)
  "main": "index.js",           // point d'entrée du package
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "build": "tsc",
    "test": "jest"
  },
  "engines": {
    "node": ">=18.0.0"           // version minimale de Node requise
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "jest": "~29.5.0",
    "nodemon": "^3.0.0"
  }
}
```

### Versioning sémantique (semver)

```
MAJOR.MINOR.PATCH    →    1.4.2
```

| Préfixe | Signification | Exemple |
|---|---|---|
| `^1.4.2` | Accepte les mises à jour **mineures et patches** (1.x.x) | `^1.4.2` → accepte `1.5.0`, refuse `2.0.0` |
| `~1.4.2` | Accepte uniquement les **patches** (1.4.x) | `~1.4.2` → accepte `1.4.9`, refuse `1.5.0` |
| `1.4.2` | Version **exacte** uniquement | aucune mise à jour automatique |
| `*` | **Toute** version | déconseillé en production |

---

## Modules : CommonJS vs ESM

Node.js supporte deux systèmes de modules. Le champ `"type"` dans `package.json` détermine le format par défaut des fichiers `.js`.

### CommonJS (format historique, défaut)

```js
// importer
const express = require('express');
const { readFile } = require('fs');

// exporter
module.exports = maFonction;
module.exports = { fonctionA, fonctionB };
```

### ESM — ECMAScript Modules (format moderne)

```js
// importer
import express from 'express';
import { readFile } from 'fs/promises';

// exporter
export default maFonction;
export { fonctionA, fonctionB };
```

### Choisir son format

```json
// package.json
{
  "type": "module"      // tous les .js sont traités comme ESM
}
// sans ce champ : CommonJS par défaut
```

### Forcer le format fichier par fichier

| Extension | Format forcé |
|---|---|
| `.cjs` | CommonJS (même si `"type": "module"`) |
| `.mjs` | ESM (même si `"type": "commonjs"`) |

### Interopérabilité

```js
// Importer du CommonJS depuis de l'ESM : fonctionne
import monModule from './module.cjs';

// Importer de l'ESM depuis du CommonJS : ⚠️ require() ne supporte pas import/export
// Utiliser import() dynamique à la place :
const monModule = await import('./module.mjs');
```

> **Recommandation** : préférer ESM pour les nouveaux projets. CommonJS reste nécessaire pour les packages publiés qui doivent supporter d'anciens environnements.
