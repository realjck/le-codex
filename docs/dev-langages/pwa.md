# PWA

Une **Progressive Web App** est une application web offrant la possibilité d'être installée sur l'appareil à l'instar d'applications mobile classiques. En s'appuyant sur les standards du web, les PWA offrent une expérience utilisateur proche de celle des applications natives.

le Service Worker est un script JavaScript qui s'exécute en arrière-plan et étend les capacités des navigateurs web. Il permet d'implémenter des fonctionnalités pour les PWA, telles que la mise en cache des ressources, les notifications push, la mise à jour en arrière-plan et l'exécution de tâches hors connexion.

## Etape 1 : Création du Service Worker

L'outil workbox-cli permet de générer directement le service-worker.

```Shell
npm install workbox-cli --save-dev

npx workbox wizard
```

Ceci lance l’outil permettant de choisir quels éléments sont mis en cache par l’application. Le Wizard vous guidera pour les différentes options.

Finir par générer le Service Worker `sw.js`  en faisant :

```Shell
npx workbox generateSW workbox-config.js
```

⚠️ **NB: ** : Il est nécessaire de régénérer sw.js pour tout nouveau build.  

L'appeler dans la page **index.html** de cette manière :

```HTML
<script>
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('./sw.js');
}
</script>
```

## Etape 2 : Déclaration du manifest.json

Créer le fichier `manifest.json` à la racine du projet

Pour configurer votre manifest.json pour une application web progressive (PWA), vous devez inclure les éléments suivants :

- Deux icône 512x512 png et svg avec l'attribut "purpose" à "maskable".
- Une autre icône 192x192 png avec l’attribut “purpose” à “any”
- Une valeur "display" de "standalone" ou "minimal-ui".

Voici un exemple de manifest.json pour une PWA :

```JSON
{
  "short_name": "devJCK",
  "name": "devJCK",
  "icons": [
    {
      "src": "./assets/images/html/favicon.svg",
      "type": "image/svg+xml",
      "sizes": "512x512",
      "purpose": "maskable"
    },
    {
      "src": "./assets/images/html/favicon.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "./assets/images/html/favicon_512.png",
      "type": "image/png",
      "sizes": "512x512",
      "purpose": "maskable"
    }
  ],
  "theme_color": "\#f1c40f",
  "background_color": "\#f1c40f",
  "display": "minimal-ui",
  "scope": "/",
  "start_url": "./index.html"
}
```

Relier le manifest dans la balise head de index.html :

```HTML
<!-- Progressive App -->
  <link rel="manifest" href="./manifest.json">
```

## Développement avec PWA : bonnes pratiques

### Eviter les problèmes de cache

L’IDE et le browser enregistrent la PWA en cache, ce qui peut empêcher le développement en mode refresh rapide.

Pour développer il est utile de travailler sur une branche de développement

Il faut alors remplacer le fichier `sw.js` par le code ci-dessous :

```JavaScript
self.registration.unregister();
```

Ajoutez sw.js au le `.gitignore`

### Rajouter dans `package.json` un script pour reconstruire le Service Worker :

```JSON
{
  "name": "my-app",
  "dependencies": {
    "workbox-cli": "^7.0.0"
  },
  "scripts": {
    "pwa": "npx workbox generateSW workbox-config.js"
  }
}
```

Pouvant être ainsi exécuté avec `npm run pwa`

## Vider le cache / rafraichir une application

### Sur Chrome :

- Outils développeur → Application
- rubrique Storage / Cache Storage : clic droit → Refresh Caches

### Sur mobile :

- Désinstallez l’application, puis réinstallez-là

ou bien :

- Clic sur icone → paramètres → Vider le cache

