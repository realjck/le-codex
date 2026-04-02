# React

Cette fiche rassemble les notions essentielles de React, organisées par thèmes.  
Elle couvre les bases (JSX, composants, state/props), les principaux hooks, les bonnes pratiques d’architecture, l’écosystème courant (React Router, gestion d’état, requêtes API), ainsi que des techniques d’optimisation.  

## 🚀 Bases

- **JSX** : C’est une syntaxe proche du HTML qui est transformée en `React.createElement()` par Babel.  
  Exemple : `<h1>Hello</h1>` devient `React.createElement("h1", null, "Hello")`.  

- **Composants** : On peut créer des composants soit en fonction (modernes) soit en classe (anciens).  
  Exemple fonction : `function Button() { return <button>OK</button>; }`.  

- **Props / State** : Les **props** sont passées depuis le parent, le **state** est interne au composant (via `useState`).  
  Exemple : `<User name="Alice" />` (prop) et `const [count, setCount] = useState(0)` (state).  

- **Rendu conditionnel** : On affiche différemment selon une condition avec `? :` ou `&&`.  
  Exemple : `{isLoggedIn ? <p>Bienvenue</p> : <p>Connectez-vous</p>}`.  

- **Listes et keys** : Chaque élément d’une liste doit avoir une **key unique et stable** pour aider React au rendu.  
  Exemple : `{users.map(u => <li key={u.id}>{u.name}</li>)}`.  

---

## 🎣 Hooks principaux
- **useState** : Sert à gérer un état local qui déclenche un re-render quand il change.  
  Exemple : `const [count, setCount] = useState(0);`.  

- **useEffect** : Permet de gérer des effets de bord (API, timers), avec dépendances et nettoyage.  
  Exemple :  
```js
  useEffect(() => {
    const timer = setInterval(() => console.log("tick"), 1000);
    return () => clearInterval(timer);
  }, []);
```  

- **useRef** : Stocke une valeur persistante sans re-render et donne accès au DOM.  
  Exemple : `const inputRef = useRef(); <input ref={inputRef} />`.  

- **useContext** : Sert à partager un état global sans props drilling.  
  Exemple : 
```js
  const ThemeContext = createContext("light");
  const App = () => <ThemeContext.Provider value="dark"><Child /></ThemeContext.Provider>;
```

- **useReducer** : Alternative à `useState` pour un état complexe.  
  Exemple : `const [state, dispatch] = useReducer(reducer, initialState);`.  
- **useMemo** : Mémorise un calcul coûteux.  
  Exemple : `const value = useMemo(() => expensiveCalc(data), [data]);`.  
- **useCallback** : Mémorise une fonction pour éviter de la recréer.  
  Exemple : `const handleClick = useCallback(() => doSomething(), []);`.  

---

## 📐 Organisation & bonnes pratiques
- **Lifting state up** : Remonter l’état dans le parent pour le partager entre enfants.  
  Exemple : un `App` gère `count` et le passe à deux composants enfants.  

- **Composants contrôlés vs non contrôlés** : Les contrôlés utilisent `value` + `onChange`, les non contrôlés s’appuient sur le DOM (`ref`).  
  Exemple contrôlé : `<input value={text} onChange={e => setText(e.target.value)} />`.  

- **Éviter le props drilling** : Utiliser `Context`, Redux ou Zustand pour passer des données globales.  
  Exemple : partager un thème avec `useContext` au lieu de propager via 10 composants.  

- **Découper les composants** : Favorise la réutilisabilité et la lisibilité.  
  Exemple : au lieu d’un énorme formulaire, créer `<InputField />`, `<SubmitButton />`, etc.  

---

## 🌍 Écosystème React
- **React Router** : Sert à gérer la navigation entre pages.  
  Exemple : `<Route path="/about" element={<About />} />`.  

- **Redux / Zustand / Recoil** : Gèrent des états complexes et partagés à grande échelle.  
  Exemple Redux : `dispatch({ type: "ADD_TODO", payload: todo })`.  

- **Fetch API, Axios, React Query** : Pour récupérer et gérer les données distantes.  
  Exemple : `useEffect(() => { fetch("/api").then(r => r.json()).then(setData); }, []);`.  

---

## ⚡ Performance & optimisation
- **Virtual DOM et réconciliation** : React compare deux arbres virtuels pour n’actualiser que les changements.  
  Exemple : si un seul `<li>` change, React ne réaffiche pas toute la liste.  

- **React.memo** : Empêche un re-render si les props n’ont pas changé.  
  Exemple : `export default React.memo(MyComponent);`.  

- **useMemo / useCallback** : Optimisent les calculs et fonctions évitant leur recréation inutile.  
  Exemple : `const sorted = useMemo(() => data.sort(), [data]);`.  

- **Lazy loading** : Charger un composant uniquement quand nécessaire avec `React.lazy` + `Suspense`.  
  Exemple :  
```js
  const About = React.lazy(() => import("./About"));
  <Suspense fallback={<p>Loading...</p>}><About /></Suspense>
```  

---

## ❓ Questions classiques
- 👉 **Différence entre state et props ?**  
  Props = données externes immuables, State = données internes modifiables.  

- 👉 **Comment fonctionne le Virtual DOM ?**  
  C’est une copie en mémoire du DOM réel, comparée à chaque rendu pour appliquer uniquement les changements nécessaires.  

- 👉 **À quoi sert useEffect et comment éviter les boucles infinies ?**  
  Il gère les effets de bord ; on évite les boucles en mettant correctement le tableau de dépendances.  

- 👉 **Différence composant contrôlé / non contrôlé ?**  
  Contrôlé = React gère la valeur (`value`), Non contrôlé = le DOM gère la valeur (`ref`).  

- 👉 **Comment optimiser les performances React ?**  
  Avec `React.memo`, `useMemo`, `useCallback`, lazy loading, et éviter les re-rendus inutiles.  

- 👉 **Qu’est-ce que le lifting state up ?**  
  Partager un état entre plusieurs composants en le remontant au parent commun.  

- 👉 **Avantages/inconvénients Redux vs Context ?**  
  Redux : puissant pour de gros projets (middlewares, devtools) mais plus complexe ; Context : simple pour des cas légers mais moins adapté aux gros états.  
