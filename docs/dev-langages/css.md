# CSS

CSS (Cascading Style Sheets) est le langage de mise en forme des pages web. Il permet de contrôler l'apparence des éléments HTML : couleurs, typographie, espacements, disposition et animations. Cette fiche couvre les concepts fondamentaux et les techniques les plus utilisées.

## Sélecteurs

Les sélecteurs permettent de cibler les éléments HTML à styliser.

```css
/* Élément */
p { color: gray; }

/* Classe */
.card { border-radius: 8px; }

/* Identifiant */
#header { background: #333; }

/* Descendant */
.nav a { text-decoration: none; }

/* Enfant direct */
ul > li { list-style: none; }

/* Attribut */
input[type="text"] { border: 1px solid #ccc; }

/* Combinaison */
h1, h2, h3 { font-family: sans-serif; }
```

## Box Model

Chaque élément HTML est une boîte composée de quatre couches : `content`, `padding`, `border` et `margin`. Par défaut, la largeur définie ne comprend pas le padding et la bordure — `box-sizing: border-box` corrige ce comportement.

```css
* {
  box-sizing: border-box; /* recommandé globalement */
}

.box {
  width: 300px;
  padding: 16px;
  border: 2px solid #ddd;
  margin: 8px auto;
}
```

## Pseudo-classes et Pseudo-éléments

Les pseudo-classes ciblent des états dynamiques, les pseudo-éléments créent des sous-parties d'un élément.

```css
/* Pseudo-classes */
a:hover { color: blue; }
input:focus { outline: 2px solid royalblue; }
li:nth-child(odd) { background: #f5f5f5; }
p:not(.intro) { font-size: 0.9rem; }

/* Pseudo-éléments */
h2::before {
  content: "# ";
  color: #aaa;
}

p::first-line {
  font-weight: bold;
}
```

## Variables CSS

Les propriétés personnalisées (custom properties) permettent de centraliser les valeurs réutilisables comme les couleurs ou les tailles.

```css
:root {
  --color-primary: #3b82f6;
  --color-text: #1f2937;
  --border-radius: 8px;
  --spacing-md: 1rem;
}

.button {
  background-color: var(--color-primary);
  color: white;
  border-radius: var(--border-radius);
  padding: var(--spacing-md);
}
```

## Flexbox

Flexbox est le système de mise en page unidimensionnel (ligne ou colonne) de CSS. Il s'applique sur le conteneur parent.

```css
.container {
  display: flex;
  flex-direction: row;        /* row | column */
  justify-content: center;    /* axe principal : flex-start | flex-end | center | space-between | space-around */
  align-items: center;        /* axe secondaire : flex-start | flex-end | center | stretch */
  flex-wrap: wrap;            /* les enfants passent à la ligne si nécessaire */
  gap: 16px;
}

/* Sur les enfants */
.item {
  flex: 1;          /* grandit pour occuper l'espace disponible */
  flex-shrink: 0;   /* ne rétrécit pas */
  align-self: flex-start; /* override de align-items pour cet élément */
}
```

## Grid

CSS Grid est le système de mise en page bidimensionnel (lignes ET colonnes).

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 3 colonnes égales */
  grid-template-rows: auto;
  gap: 24px;
}

/* Positionnement d'un élément sur la grille */
.featured {
  grid-column: 1 / 3;   /* s'étend de la colonne 1 à 3 */
  grid-row: 1 / 2;
}

/* Grille responsive sans media query */
.auto-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 16px;
}
```

## Positionnement

```css
/* static : par défaut, dans le flux normal */
/* relative : décalé par rapport à sa position normale */
.tooltip-parent {
  position: relative;
}

/* absolute : retiré du flux, positionné par rapport au parent "relative" le plus proche */
.tooltip {
  position: absolute;
  top: 100%;
  left: 0;
}

/* fixed : positionné par rapport à la fenêtre, reste en place au scroll */
.navbar {
  position: fixed;
  top: 0;
  width: 100%;
}

/* sticky : reste dans le flux jusqu'à atteindre un seuil, puis se fixe */
.section-header {
  position: sticky;
  top: 64px;
}
```

## Media Queries

Les media queries permettent d'adapter le style selon la taille de l'écran (approche Mobile First recommandée).

```css
/* Mobile first : styles de base pour petits écrans */
.container {
  padding: 16px;
  flex-direction: column;
}

/* Tablette et plus */
@media (min-width: 768px) {
  .container {
    padding: 32px;
    flex-direction: row;
  }
}

/* Desktop */
@media (min-width: 1200px) {
  .container {
    max-width: 1140px;
    margin: 0 auto;
  }
}

/* Préférence de thème sombre */
@media (prefers-color-scheme: dark) {
  body {
    background: #111;
    color: #f5f5f5;
  }
}
```

## Transitions et Animations

### Transitions

Les transitions s'appliquent lors du changement d'un état (ex. `:hover`).

```css
.button {
  background-color: #3b82f6;
  transition: background-color 0.2s ease, transform 0.2s ease;
}

.button:hover {
  background-color: #2563eb;
  transform: translateY(-2px);
}
```

### Animations

Les animations permettent des séquences plus complexes via `@keyframes`.

```css
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.card {
  animation: fadeIn 0.4s ease forwards;
}
```

## Typographie

```css
body {
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 16px;       /* base */
  line-height: 1.6;
  color: #1f2937;
}

h1 { font-size: clamp(1.5rem, 4vw, 2.5rem); } /* taille fluide */
h2 { font-size: 1.75rem; font-weight: 700; }

/* Tronquer le texte sur une ligne */
.truncate {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Limiter à N lignes */
.clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

## Conclusion

Pour aller plus loin, consulter la référence complète sur [MDN Web Docs](https://developer.mozilla.org/fr/docs/Web/CSS) et l'outil interactif [CSS Tricks](https://css-tricks.com/).
