# HTML5

Ce code source fournit une base pour le développement de page web. Il est conçu pour être un point de départ rapide et pratique. (Copié de [github.com/h5bp/html5-boilerplate](https://github.com/h5bp/html5-boilerplate))


Structure de projet type :

```
my-app/
|-- css/
|   |-- styles.css
|-- img/
|-- js/
|   |-- libs/
|   |-- app.js
|-- index.html
|-- 404.html
|-- site.webmanifest
|-- robots.txt
```

index.html :

```html
<!doctype html>
<html lang="fr">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title></title>
  <meta name="description" content="">
  <link rel="stylesheet" href="css/style.css">
  <script src="js/app.js" defer></script>

  <!-- Balises OpenGraph (OG) pour le partage : -->
  <meta property="og:title" content="">
  <meta property="og:type" content=""><!-- Type de contenu : website, article, etc. -->
  <meta property="og:url" content="">
  <meta property="og:image" content=""><!-- Lien de l'image dimensions 1200x630px -->
  <meta property="og:image:alt" content="">

  <link rel="icon" href="/favicon.ico" sizes="any">
  <link rel="icon" href="/icon.svg" type="image/svg+xml">
  <link rel="apple-touch-icon" href="icon.png">

  <link rel="manifest" href="site.webmanifest">
  <meta name="theme-color" content="#fafafa">
</head>

<body>
	
  <p>Hello, world!</p>
  
</body>

</html>
```

site.webmanifest :

```json
{
  "short_name": "",
  "name": "",
  "icons": [{
    "src": "icon.png",
    "type": "image/png",
    "sizes": "192x192"
  }],
  "start_url": "/?utm_source=homescreen",
  "background_color": "#fafafa",
  "theme_color": "#fafafa"
}
```

robots.txt :

```
# https://www.robotstxt.org/

# Allow crawling of all content
User-agent: *
Disallow:
```

404.html :

```html
<!doctype html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>Page Not Found</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    * {
      line-height: 1.2;
      margin: 0;
    }

    html {
      color: #888;
      display: table;
      font-family: sans-serif;
      height: 100%;
      text-align: center;
      width: 100%;
    }

    body {
      display: table-cell;
      vertical-align: middle;
      margin: 2em auto;
    }

    h1 {
      color: #555;
      font-size: 2em;
      font-weight: 400;
    }

    p {
      margin: 0 auto;
      width: 280px;
    }

    @media only screen and (max-width: 280px) {

      body,
      p {
        width: 95%;
      }

      h1 {
        font-size: 1.5em;
        margin: 0 0 0.3em;
      }

    }
  </style>
</head>

<body>
  <h1>Page Not Found</h1>
  <p>Sorry, but the page you were trying to view does not exist.</p>
</body>

</html>
<!-- IE needs 512+ bytes: https://docs.microsoft.com/archive/blogs/ieinternals/friendly-http-error-pages -->
```