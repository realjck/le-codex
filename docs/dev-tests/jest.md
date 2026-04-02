# Jest

Jest est le framework de test JavaScript le plus utilisé. Il intègre un runner, un système d'assertions, des mocks et la mesure de couverture. Il fonctionne avec JavaScript, TypeScript, React, Node.js et la plupart des stacks JS modernes.

---

## Installation et configuration

### Projet JavaScript

```bash
npm install --save-dev jest
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### Projet TypeScript

```bash
npm install --save-dev jest @types/jest ts-jest
```

```js
// jest.config.js
/** @type {import('jest').Config} */
const config = {
  preset: 'ts-jest',
  testEnvironment: 'node',
};

module.exports = config;
```

### Projet React (Create React App / Vite)

```bash
npm install --save-dev jest @types/jest ts-jest \
  jest-environment-jsdom \
  @testing-library/react \
  @testing-library/jest-dom \
  @testing-library/user-event
```

```js
// jest.config.js
/** @type {import('jest').Config} */
const config = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',                          // simule le DOM du navigateur
  setupFilesAfterFramework: ['<rootDir>/jest.setup.ts'],
};

module.exports = config;
```

```ts
// jest.setup.ts
import '@testing-library/jest-dom';   // ajoute les matchers DOM (toBeInTheDocument, etc.)
```

### Convention de nommage des fichiers

Jest détecte automatiquement les fichiers de test selon ces patterns :

```
src/
  utils/
    calcul.ts
    calcul.test.ts      ✅ détecté
    calcul.spec.ts      ✅ détecté
  __tests__/
    calcul.ts           ✅ détecté (dossier __tests__)
```

---

## Structure d'un test

```ts
// describe : regroupe des tests liés
describe('calculateTax', () => {

  // it / test : un test unitaire (équivalents)
  it('calcule la TVA à 20%', () => {
    expect(calculateTax(100, 0.20)).toBe(20);
  });

  test('retourne 0 si le prix est 0', () => {
    expect(calculateTax(0, 0.20)).toBe(0);
  });

  // Groupes imbriqués
  describe('cas limites', () => {
    it('gère les nombres négatifs', () => {
      expect(calculateTax(-100, 0.20)).toBe(-20);
    });
  });
});
```

### Ignorer ou isoler des tests

```ts
test.skip('ce test est ignoré', () => { ... });
test.only('seul ce test s'exécute dans le fichier', () => { ... });

describe.skip('tout ce groupe est ignoré', () => { ... });
describe.only('seul ce groupe s'exécute', () => { ... });
```

---

## Matchers (assertions)

### Égalité

```ts
expect(value).toBe(42);              // égalité stricte (===)
expect(value).toEqual({ a: 1 });     // égalité profonde (objets, tableaux)
expect(value).not.toBe(42);          // négation
expect(value).toStrictEqual({});     // comme toEqual + vérifie les undefined
```

### Valeurs

```ts
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();
expect(value).toBeNaN();
```

### Nombres

```ts
expect(value).toBeGreaterThan(3);
expect(value).toBeGreaterThanOrEqual(3);
expect(value).toBeLessThan(10);
expect(value).toBeLessThanOrEqual(10);
expect(value).toBeCloseTo(0.3, 5);   // flottants (évite les erreurs de précision)
```

### Chaînes

```ts
expect(str).toMatch(/regex/);
expect(str).toMatch('sous-chaîne');
expect(str).toContain('mot');
expect(str).toHaveLength(5);
```

### Tableaux et objets

```ts
expect(arr).toContain('valeur');
expect(arr).toHaveLength(3);
expect(arr).toEqual(expect.arrayContaining(['a', 'b']));  // contient au moins ces éléments

expect(obj).toHaveProperty('clé');
expect(obj).toHaveProperty('clé', 'valeur');
expect(obj).toMatchObject({ a: 1 });  // l'objet contient au moins ces propriétés
```

### Erreurs

```ts
expect(() => maFonction()).toThrow();
expect(() => maFonction()).toThrow('message d\'erreur');
expect(() => maFonction()).toThrow(TypeError);
```

---

## Lifecycle hooks

```ts
beforeAll(() => {
  // exécuté une fois avant tous les tests du bloc describe
});

afterAll(() => {
  // exécuté une fois après tous les tests du bloc describe
});

beforeEach(() => {
  // exécuté avant chaque test
});

afterEach(() => {
  // exécuté après chaque test
  jest.clearAllMocks();   // bonne pratique : réinitialiser les mocks
});
```

### Portée

Les hooks s'appliquent au bloc `describe` dans lequel ils sont définis. Un hook au niveau racine s'applique à tout le fichier.

```ts
describe('groupe A', () => {
  beforeEach(() => { /* s'applique uniquement aux tests de groupe A */ });

  it('test 1', () => { ... });
  it('test 2', () => { ... });
});
```

---

## Mocks et spies

### jest.fn() — fonction fictive

```ts
const mockFn = jest.fn();
mockFn('arg1', 'arg2');

expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(1);
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenLastCalledWith('arg1', 'arg2');
```

### Valeur de retour

```ts
const mockFn = jest.fn();
mockFn.mockReturnValue(42);              // retourne toujours 42
mockFn.mockReturnValueOnce(100);         // retourne 100 une seule fois, puis la valeur par défaut
mockFn.mockResolvedValue({ data: [] });  // retourne une Promise résolue
mockFn.mockRejectedValue(new Error());   // retourne une Promise rejetée

// Implémentation personnalisée
mockFn.mockImplementation((x) => x * 2);
```

### jest.spyOn() — espionner une méthode existante

```ts
const spy = jest.spyOn(console, 'log');
console.log('test');

expect(spy).toHaveBeenCalledWith('test');
spy.mockRestore();   // restaure l'implémentation originale
```

```ts
// Remplacer temporairement une implémentation
jest.spyOn(Date, 'now').mockReturnValue(1_700_000_000_000);
```

### jest.mock() — mocker un module entier

```ts
// Mocker un module automatiquement
jest.mock('./monModule');

// Mocker avec une implémentation personnalisée
jest.mock('./api', () => ({
  fetchUser: jest.fn().mockResolvedValue({ id: 1, name: 'Alice' }),
}));

// Dans le test
import { fetchUser } from './api';
const result = await fetchUser(1);
expect(fetchUser).toHaveBeenCalledWith(1);
```

### Réinitialisation des mocks

```ts
jest.clearAllMocks();    // remet les appels et instances à zéro (garde l'implémentation)
jest.resetAllMocks();    // clearAllMocks + supprime les implémentations mockées
jest.restoreAllMocks();  // resetAllMocks + restaure les spies (jest.spyOn)
```

---

## Tests asynchrones

### async / await

```ts
it('récupère un utilisateur', async () => {
  const user = await fetchUser(1);
  expect(user.name).toBe('Alice');
});
```

### Rejets de Promise

```ts
it('lance une erreur si l\'id est invalide', async () => {
  await expect(fetchUser(-1)).rejects.toThrow('ID invalide');
});
```

### Timers simulés

```ts
jest.useFakeTimers();

it('appelle le callback après 1 seconde', () => {
  const callback = jest.fn();
  setTimeout(callback, 1000);

  expect(callback).not.toHaveBeenCalled();
  jest.advanceTimersByTime(1000);
  expect(callback).toHaveBeenCalledTimes(1);
});

afterEach(() => {
  jest.useRealTimers();   // toujours restaurer après
});
```

---

## Coverage

```bash
jest --coverage
```

Jest génère un rapport dans `coverage/` avec plusieurs formats (texte, HTML, LCOV).

```js
// jest.config.js — seuils minimaux
const config = {
  collectCoverageFrom: ['src/**/*.{ts,tsx}', '!src/**/*.d.ts'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

Les 4 métriques :

| Métrique | Description |
|---|---|
| **Statements** | % d'instructions exécutées |
| **Branches** | % de branches (if/else, ternaire) testées |
| **Functions** | % de fonctions appelées |
| **Lines** | % de lignes exécutées |

---

## React Testing Library

React Testing Library (RTL) encourage à tester le **comportement** visible par l'utilisateur plutôt que les détails d'implémentation.

> Principe clé : interroger le DOM comme un utilisateur le ferait (par le texte, le rôle, le label) — pas par les noms de classes ou l'état interne.

### Render et screen

```tsx
import { render, screen } from '@testing-library/react';
import MonComposant from './MonComposant';

it('affiche le titre', () => {
  render(<MonComposant titre="Bonjour" />);
  expect(screen.getByText('Bonjour')).toBeInTheDocument();
});
```

### Priorité des requêtes

Utiliser les requêtes dans cet ordre (du plus au moins accessible) :

| Requête | Usage recommandé |
|---|---|
| `getByRole` | Boutons, liens, inputs, headings… |
| `getByLabelText` | Champs de formulaire avec label |
| `getByPlaceholderText` | Champs avec placeholder |
| `getByText` | Éléments de contenu textuel |
| `getByDisplayValue` | Valeur courante d'un input/select |
| `getByAltText` | Images avec alt |
| `getByTitle` | Éléments avec attribut title |
| `getByTestId` | En dernier recours (`data-testid`) |

### Variantes de requêtes

```ts
// getBy* — lance une erreur si l'élément est absent (élément censé exister)
screen.getByRole('button', { name: /envoyer/i });

// queryBy* — retourne null si absent (tester l'absence)
expect(screen.queryByText('Erreur')).not.toBeInTheDocument();

// findBy* — attend que l'élément apparaisse (éléments asynchrones)
const message = await screen.findByText('Chargement terminé');
```

### Matchers jest-dom

```ts
expect(element).toBeInTheDocument();
expect(element).toBeVisible();
expect(element).toBeDisabled();
expect(element).toBeEnabled();
expect(element).toHaveValue('alice@exemple.com');
expect(element).toHaveTextContent('Bonjour');
expect(element).toHaveAttribute('href', '/accueil');
expect(element).toHaveClass('btn-primary');
expect(element).toBeChecked();          // checkbox / radio
expect(element).toHaveFocus();
```

### Interactions utilisateur (userEvent)

Préférer `userEvent` à `fireEvent` — il simule les vrais événements navigateur (keydown, keyup, focus…).

```tsx
import userEvent from '@testing-library/user-event';

it('soumet le formulaire', async () => {
  const user = userEvent.setup();
  render(<Formulaire onSubmit={jest.fn()} />);

  await user.type(screen.getByLabelText('Email'), 'alice@exemple.com');
  await user.click(screen.getByRole('button', { name: /envoyer/i }));

  expect(screen.getByText('Message envoyé')).toBeInTheDocument();
});
```

### Tester des composants asynchrones

```tsx
it('affiche les données après chargement', async () => {
  render(<ListeUtilisateurs />);

  // Pendant le chargement
  expect(screen.getByText('Chargement...')).toBeInTheDocument();

  // Attendre que les données apparaissent
  const alice = await screen.findByText('Alice');
  expect(alice).toBeInTheDocument();
  expect(screen.queryByText('Chargement...')).not.toBeInTheDocument();
});
```

### Exemple complet

```tsx
// Composant
function Compteur() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Valeur : {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Incrémenter</button>
      <button onClick={() => setCount(0)} disabled={count === 0}>Réinitialiser</button>
    </div>
  );
}

// Test
describe('Compteur', () => {
  it('affiche la valeur initiale à 0', () => {
    render(<Compteur />);
    expect(screen.getByText('Valeur : 0')).toBeInTheDocument();
  });

  it('incrémente la valeur au clic', async () => {
    const user = userEvent.setup();
    render(<Compteur />);

    await user.click(screen.getByRole('button', { name: /incrémenter/i }));
    expect(screen.getByText('Valeur : 1')).toBeInTheDocument();
  });

  it('désactive le bouton réinitialiser quand la valeur est 0', () => {
    render(<Compteur />);
    expect(screen.getByRole('button', { name: /réinitialiser/i })).toBeDisabled();
  });
});
```
