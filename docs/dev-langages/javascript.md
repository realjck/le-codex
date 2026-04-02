# JavaScript

Cet article passe en revue certaines des fonctionnalités les plus utiles et pratiques du langage.

## Déstructuration

La déstructuration en JavaScript est un moyen pratique d'extraire plusieurs propriétés d'un objet ou d'un tableau en une seule instruction. Elle peut considérablement simplifier votre code et le rendre plus lisible. Par exemple :

```js
let [a, b] = [1, 2];
console.log(a); // affiche : 1
console.log(b); // affiche : 2
```

La même chose peut être faite avec des objets :

```js
let {name, age} = {name: "John", age: 22};
console.log(name); // affiche : John
console.log(age); // affiche : 22
```

## Chaînage Optionnel

Le chaînage optionnel est une addition récente à JavaScript (ES2020), qui permet d'accéder à des propriétés d'objets profondément imbriquées sans avoir à vérifier si chaque référence dans la chaîne est valide. Il aide à écrire un code plus propre et plus sûr. Voici comment cela fonctionne :

```js
let user = {}; // un objet vide
console.log(user?.address?.street); // affiche : undefined, au lieu de lancer une erreur
```

## Opérateurs de Décomposition/Rest

L'opérateur de décomposition (`...`) en JavaScript est utilisé pour décomposer les éléments d'un tableau ou d'un objet. L'opérateur rest, également (`...`), est utilisé pour collecter le reste des éléments dans un tableau. Ces deux opérateurs contribuent à un code plus court et plus propre :

```js
let arr1 = [1, 2, 3];
let arr2 = [...arr1, 4, 5]; // affiche : [1, 2, 3, 4, 5]
```

Pour l'opérateur rest :

```js
let [first, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // affiche : 1
console.log(rest); // affiche : [2, 3, 4, 5]
```

## Évaluation en Court-circuit

L'évaluation en court-circuit en JavaScript implique des opérateurs logiques (`&&` et `||`) pour créer un code plus court. Elle peut être utilisée pour définir des valeurs par défaut, valider des objets avant de les utiliser, et plus encore :

```js
let name = user && user.name;
let port = serverPort || 3000;
```

## Littéraux de Modèle

Les littéraux de modèle fournissent un moyen facile d'interpoler des variables et des expressions dans des chaînes de caractères. Ils sont entourés de caractères d'accent grave (`) au lieu de guillemets :

```js
let name = "John";
console.log(`Hello, ${name}!`); // affiche : Hello, John!
```

## Fonctions Fléchées

Les fonctions fléchées offrent une syntaxe plus concise pour écrire des expressions de fonction. Elles sont idéales pour les fonctions non-méthodes et elles n'ont pas leur propre `this`, `arguments`, `super`, ou `new.target` :

```js
let hello = () => "Hello World";
console.log(hello()); // affiche : Hello World
```

## Opérateur de Fusion Null

L'opérateur de fusion null (`??`) est un opérateur logique qui retourne son opérande de droite lorsque son opérande de gauche est `null` ou `undefined`, et sinon retourne son opérande de gauche.

```js
let name = null ?? "default name";
console.log(name); // affiche : default name
```

## Opérateur Ternaire

L'opérateur ternaire est un moyen plus rapide d'écrire des instructions `if-else` simples. La syntaxe est : `condition ? valeur_si_vrai : valeur_si_faux` :

```js
let age = 15;
let type = age >= 18 ? "Adult" : "Minor";
console.log(type); // affiche : Minor
```

## Méthodes de Tableau

En JavaScript, les tableaux (ou arrays) sont des structures de données couramment utilisées, et le langage propose un grand nombre de méthodes pour les manipuler. Voici quelques-unes des plus courantes.

### forEach()

La méthode `forEach()` permet d'exécuter une fonction donnée sur chaque élément du tableau.

```JavaScript
const array1 = ['a', 'b', 'c'];

array1.forEach(element => console.log(element));

// Expected output: "a"
// Expected output: "b"
// Expected output: "c"
```

### map()

La méthode `map` parcourt chaque élément d'un tableau et renvoie un nouveau tableau contenant les résultats de l'appel de la fonction de rappel sur chaque élément. Il le fait sans muter le tableau d'origine.

```JavaScript
const users = [
  { name: 'John', age: 34 },
  { name: 'Amy', age: 20 },
  { name: 'camperCat', age: 10 }
];

const names = users.map(user => user.name);
```

### filter()

`filter` appelle une fonction sur chaque élément d'un tableau et renvoie un nouveau tableau contenant uniquement les éléments pour lesquels cette fonction renvoie une valeur vraie.

```JavaScript
const users = [
  { name: 'John', age: 34 },
  { name: 'Amy', age: 20 },
  { name: 'camperCat', age: 10 }
];

const usersUnder30 = users.filter(user => user.age < 30);
```

### slice()

La méthode `slice` renvoie une copie de certains éléments d'un tableau. Il peut prendre deux arguments, le premier donne l'index de l'endroit où commencer la tranche, le second est l'index de l'endroit où terminer la tranche (et il n'est pas inclusif).

```JavaScript
const arr = ["Cat", "Dog", "Tiger", "Zebra"];
const newArray = arr.slice(1, 3);
// newArray : ["Dog", "Tiger"]
```

### splice()

La méthode `splice` prend des arguments pour l'index indiquant où commencer à supprimer les éléments, puis le nombre d'éléments à supprimer. Si le deuxième argument n’est pas fourni, la valeur par défaut consiste à supprimer les éléments jusqu’à la fin. 

⚠️ Attention, la méthode `splice` mute le tableau d'origine sur lequel elle est appelée.

```JavaScript
const cities = ["Chicago", "Delhi", "Islamabad", "London", "Berlin"];
cities.splice(3, 1);
// cities : ["Chicago", "Delhi", "Islamabad", "Berlin"]
```

### reduce()

La méthode `reduce` parcourt chaque élément d'un tableau et renvoie une valeur unique (c'est-à-dire une chaîne, un nombre, un objet, un tableau). Ceci est réalisé via une fonction de rappel qui est appelée à chaque itération.

```JavaScript
const users = [
  { name: 'John', age: 34 },
  { name: 'Amy', age: 20 },
  { name: 'camperCat', age: 10 }
];

const sumOfAges = users.reduce((sum, user) => sum + user.age, 0);
```


## Async/Await

Async/Await rend le code asynchrone plus semblable au code synchrone. Ce sucre syntaxique au-dessus des promesses rend le code asynchrone plus facile à comprendre et à écrire :

```js
async function fetchUser() {
  let response = await fetch('https://api.github.com/users');
  let data = await response.json();
  console.log(data);
}
fetchUser();
```
