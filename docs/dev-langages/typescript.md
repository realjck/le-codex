# TypeScript

TypeScript est un langage de programmation à typage statique qui se compile en JavaScript. Cela signifie que vous devez définir les types de vos variables, fonctions et autres éléments de code, ce qui permet au compilateur de détecter les erreurs potentielles dès le stade de l'écriture du code.

TypeScript est un sur-ensemble de JavaScript, ce qui signifie que tout code JavaScript valide est également du code TypeScript valide. Cela signifie que vous pouvez utiliser TypeScript pour écrire du code JavaScript plus robuste et plus maintenable.

### Exemple d’interface :

```TypeScript
interface Article {
  title: string
  id: number
}

const article: Article = {
  title: "Hayes",
  id: 0,
}
```

### Définition d’une propriété optionnelle :

```TypeScript
interface Article {
  title: string
  id?: number
}

const article: Article = {
  title: "Hayes",
}
```

## Type Vs Interface

La différence entre `interface` et `type` est un peu subtile, surtout depuis les dernières versions qui font converger leurs fonctionnalités. Néanmoins, les types sont un peu plus flexibles puisqu'ils peuvent être composés via des unions ou des intersections

```TypeScript
type Article = {
  title: string
  id?: number
}
```

### les types peuvent être combinés via **une intersection**  `&`:

```TypeScript
type Person = {
  name: string
  age: number
}

type Employee = Person & {
  id: number
  department: string
}
// Employee: { name: string; age: number; id: number; department: string }
```

### **l'union** permet de **conditionner** un type en énumérant plusieurs valeurs possibles :  
  
`type Color = "red" | "blue" | "green"`

```TypeScript
type Filter = {
  name: string
  email: string
  firstname: string
  lastname: string
}

type SortOptions = {
  sortBy: "name" | "email"
  sortOrder: "asc" | "desc"
}

function findUser(filter: Filter, sort?: SortOptions) {
  // ...
}
```

l'union est souvent utilisée pour signaler qu'une valeur peut être soit `null`, soit d'un type donné. Par exemple :

```TypeScript
type Filter = {
  id: string | number
  name: string
  email: string
  firstname: string | null // Ici firstname peut être SOIT une string, SOIT null
  lastname: string | null // Idem pour lastname
}
```

### L'opérateur `keyof` :

Il permet d'extraire les clés (ou propriétés) d'un type sous forme d'une union de chaînes de caractères. Ici, `keyof Filter` revient donc à écrire `'id' | 'name' | 'email' | 'firstname' | 'lastname'`.

```TypeScript
type SortOptions = {
  sortBy: keyof Filter
  sortOrder: "asc" | "desc"
}
```

Il existe un opérateur similaire à `keyof`, et il s'agit de `key` :

```TypeScript
type User = {
  [key: string]: any
}
```

Cette notation permet de préciser qu'il peut y avoir une clé de **n'importe quel nom** _(à condition que ce soit une chaîne de caractères)_ sur l'objet `User`. Le type `any` permet quant à lui d'indiquer que la valeur associée à cette clé peut être de **n'importe quel type**

## **Les tableaux**

```TypeScript
// tableau d'un seul et unique élément en string
type oneStringArray = [string]
// tableau de string sans limitation sur la taille
type stringArray = string[]
// ... qui peut également s'écrire ainsi :
type stringArray = Array<string>
```

```TypeScript
type streamFormat = [string, number, string]
// tableau de format "stream" contenant une chaîne de caractères,
// un nombre et une autre chaîne de caractères
const test: streamFormat = ["Start", 123, "End"] //on appelle cela un tuple !
```

## **Les fonctions**

```TypeScript
interface CalculatorInterface {
  add: (x: number, y: number) => number
  subtract: (x: number, y: number) => number
}

class Calculator implements CalculatorInterface {
  add(x: number, y: number) {
    return x + y
  }

  subtract(x: number, y: number) {
    return x - y
  }
}
```

Il est également possible de typer les signatures de fonction de manière indépendante :

```TypeScript
function greet(name: string): void {
  console.log(`Hello, ${name}!`)
}
//Dans cet exemple, la fonction greet prend un argument name de type string 
// et ne retourne rien (:void)
```

## Les génériques

Voici un exemple de fonction générique qui prend un tableau de n'importe quel type et renvoie le premier élément de ce tableau :

```TypeScript
function getFirstElement<T>(arr: T[]): T | undefined {
  if (arr.length > 0) {
    return arr[0]
  }
  return undefined
}
```

Dans cet exemple, le type générique `T` est utilisé pour représenter le type d'élément contenu dans le tableau. La fonction `getFirstElement` prend un tableau de type `T[]` en argument et renvoie le premier élément de ce tableau _(qui est donc de type_ `_T_`_)_ si le tableau n'est pas vide, `undefined` sinon _(d'où le retour_ `_T | undefined_`_)_.

On peut donc utiliser cette fonction avec différents types de tableaux :

```TypeScript
const numbers = [1, 2, 3, 4]
const firstNumber = getFirstElement(numbers) // renvoie 1

const strings = ["foo", "bar", "baz"]
const firstString = getFirstElement(strings) // renvoie "foo"
```

Par exemple, `Promise<unknown>` désigne simplement une promesse qui résoudra sur une valeur de type `unknown`. Si la promesse se résout avec un `number`, vous l'aurez devinez, il suffit de l'écrire ainsi : `Promise<number>`.

### Le type Unknown :

On peut affecter absolument ce qu'on veut à une variable de type `unknown` :  
  

```TypeScript
let value: unknown

value = undefined
value = null
value = 123
value = true
value = "Hello"
value = []
value = {}
value = () => {}
```

Le plus souvent, on va régler le linter de nos projets pour interdire l'usage de `any` implicites, afin justement d'éviter la confusion entre `unknown` et `any` : soit on affirme ne pas savoir (`unknown`), soit on affirme accepter _“n'importe quoi”_ (`any`).
