# Playwright

Playwright est un framework de tests end-to-end (e2e) développé par Microsoft. Il permet d'automatiser Chromium, Firefox et WebKit avec une seule API, et intègre nativement l'auto-attente, les assertions web-first et l'isolation complète entre les tests.

> **Playwright vs Cypress** : Playwright supporte nativement multi-navigateurs, multi-onglets, iframes et requêtes API. Il est plus rapide en CI et ne nécessite pas de serveur dédié.

---

## Installation et configuration

```bash
npm init playwright@latest
# → installe Playwright, crée playwright.config.ts et un exemple de test
```

Installation manuelle :

```bash
npm install --save-dev @playwright/test
npx playwright install          # installe les navigateurs (Chromium, Firefox, WebKit)
npx playwright install chromium # installer un seul navigateur
```

### playwright.config.ts

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30_000,              // timeout par test (ms)
  retries: process.env.CI ? 2 : 0,  // retry automatique en CI
  workers: process.env.CI ? 1 : undefined,

  reporter: [
    ['html'],                   // rapport HTML dans playwright-report/
    ['list'],                   // sortie console
  ],

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',    // enregistre une trace en cas d'échec
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox',  use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit',   use: { ...devices['Desktop Safari'] } },
    { name: 'mobile',   use: { ...devices['iPhone 13'] } },
  ],

  // Lancer le serveur de dev avant les tests
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Commandes utiles

```bash
npx playwright test                    # lancer tous les tests
npx playwright test mon-test.spec.ts   # un fichier spécifique
npx playwright test --headed           # avec le navigateur visible
npx playwright test --debug            # mode debug (pause sur chaque étape)
npx playwright test --ui               # interface graphique interactive
npx playwright show-report             # ouvrir le rapport HTML
npx playwright codegen https://...     # générer des tests en enregistrant des actions
```

---

## Premier test e2e

```ts
// tests/exemple.spec.ts
import { test, expect } from '@playwright/test';

test('la page d\'accueil affiche le titre', async ({ page }) => {
  await page.goto('/');                                    // navigue vers baseURL + /
  await expect(page).toHaveTitle(/Mon Application/);
});

test('l\'utilisateur peut se connecter', async ({ page }) => {
  await page.goto('/connexion');
  await page.getByLabel('Email').fill('alice@exemple.com');
  await page.getByLabel('Mot de passe').fill('secret');
  await page.getByRole('button', { name: 'Se connecter' }).click();
  await expect(page).toHaveURL('/tableau-de-bord');
  await expect(page.getByText('Bienvenue, Alice')).toBeVisible();
});
```

### Grouper les tests

```ts
import { test, expect } from '@playwright/test';

test.describe('Authentification', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/connexion');
  });

  test('connexion avec identifiants valides', async ({ page }) => { ... });
  test('affiche une erreur si le mot de passe est incorrect', async ({ page }) => { ... });

  test.skip('à implémenter', async ({ page }) => { ... });
  test.only('seul ce test s\'exécute', async ({ page }) => { ... });
});
```

---

## Locators et sélecteurs

Les **locators** sont la façon recommandée de cibler des éléments. Ils intègrent l'auto-attente et les retry automatiques.

### Locators recommandés (par priorité)

```ts
// Par rôle ARIA (le plus robuste)
page.getByRole('button', { name: 'Envoyer' })
page.getByRole('heading', { name: 'Connexion' })
page.getByRole('link', { name: /accueil/i })
page.getByRole('textbox', { name: 'Email' })
page.getByRole('checkbox', { name: 'Se souvenir de moi' })

// Par label (champs de formulaire)
page.getByLabel('Email')
page.getByLabel('Mot de passe')

// Par placeholder
page.getByPlaceholder('Rechercher...')

// Par texte
page.getByText('Bienvenue')
page.getByText('Bienvenue', { exact: true })

// Par alt text (images)
page.getByAltText('Logo de l\'entreprise')

// Par test id (en dernier recours)
page.getByTestId('submit-btn')   // data-testid="submit-btn"
```

### Sélecteurs CSS et XPath (si nécessaire)

```ts
page.locator('.btn-primary')
page.locator('#mon-id')
page.locator('input[type="email"]')
page.locator('xpath=//button[@type="submit"]')
```

### Filtres et chaînage

```ts
// Filtrer par texte
page.getByRole('listitem').filter({ hasText: 'Alice' })

// Filtrer par contenu enfant
page.getByRole('listitem').filter({ has: page.getByRole('button') })

// Locator dans un locator
page.getByRole('article').getByRole('button', { name: 'Lire la suite' })

// Premier, dernier, nième
page.getByRole('listitem').first()
page.getByRole('listitem').last()
page.getByRole('listitem').nth(2)   // index 0-based
```

---

## Assertions

Playwright utilise des **assertions web-first** : elles attendent automatiquement que la condition soit remplie (retry jusqu'au timeout).

### Page

```ts
await expect(page).toHaveURL('/tableau-de-bord');
await expect(page).toHaveURL(/dashboard/);
await expect(page).toHaveTitle('Mon Application');
```

### Éléments

```ts
await expect(locator).toBeVisible();
await expect(locator).toBeHidden();
await expect(locator).toBeEnabled();
await expect(locator).toBeDisabled();
await expect(locator).toBeChecked();
await expect(locator).toBeFocused();
await expect(locator).toBeEmpty();             // champ vide

await expect(locator).toHaveText('Bonjour');
await expect(locator).toContainText('Bon');
await expect(locator).toHaveValue('alice@exemple.com');
await expect(locator).toHaveAttribute('href', '/accueil');
await expect(locator).toHaveClass('btn-primary');
await expect(locator).toHaveCount(3);          // nombre d'éléments correspondants
```

### Assertions génériques (sans auto-attente)

```ts
expect(valeur).toBe(42);
expect(valeur).toEqual({ a: 1 });
expect(valeur).toBeTruthy();
expect(tableau).toHaveLength(3);
```

---

## Interactions

### Navigation

```ts
await page.goto('https://exemple.com');
await page.goto('/chemin');              // relatif à baseURL
await page.goBack();
await page.goForward();
await page.reload();
```

### Clics et formulaires

```ts
await locator.click();
await locator.dblclick();
await locator.click({ button: 'right' });    // clic droit
await locator.click({ modifiers: ['Shift'] }); // Shift+clic

await locator.fill('texte');             // remplace la valeur entière
await locator.type('texte');             // simule la frappe touche par touche
await locator.clear();                   // vide le champ
await locator.press('Enter');
await locator.press('Control+A');
await locator.selectOption('option1');           // <select>
await locator.selectOption({ label: 'Option 1' });
await locator.check();                   // checkbox / radio
await locator.uncheck();
```

### Upload et drag-and-drop

```ts
await page.getByLabel('Fichier').setInputFiles('chemin/vers/fichier.pdf');
await page.getByLabel('Fichier').setInputFiles([]);   // vider

await locator.dragTo(page.locator('#destination'));
```

### Hover et scroll

```ts
await locator.hover();
await locator.scrollIntoViewIfNeeded();
await page.mouse.wheel(0, 300);          // scroll de 300px vers le bas
```

### Clavier et souris

```ts
await page.keyboard.press('Escape');
await page.keyboard.type('Bonjour');
await page.mouse.move(100, 200);
await page.mouse.click(100, 200);
```

---

## Async et réseau

### Attendre des événements

```ts
// Attendre une navigation
await page.waitForURL('/tableau-de-bord');

// Attendre qu'un élément soit visible
await page.waitForSelector('.chargement', { state: 'hidden' });

// Attendre une réponse réseau
const [response] = await Promise.all([
  page.waitForResponse('**/api/utilisateurs'),
  page.getByRole('button', { name: 'Charger' }).click(),
]);
expect(response.status()).toBe(200);
```

### Intercepter et mocker le réseau

```ts
// Mocker une réponse API
await page.route('**/api/utilisateurs', async route => {
  await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify([{ id: 1, nom: 'Alice' }]),
  });
});

// Modifier une réponse existante
await page.route('**/api/produits', async route => {
  const response = await route.fetch();
  const json = await response.json();
  json.push({ id: 99, nom: 'Produit test' });
  await route.fulfill({ response, json });
});

// Bloquer des ressources (accélérer les tests)
await page.route('**/*.{png,jpg,gif,css}', route => route.abort());

// Intercepter et modifier les headers
await page.route('**/api/**', async route => {
  await route.continue({
    headers: {
      ...route.request().headers(),
      'Authorization': 'Bearer token-de-test',
    },
  });
});
```

---

## Screenshots, vidéos et traces

### Screenshots

```ts
await page.screenshot({ path: 'screenshot.png' });
await page.screenshot({ path: 'screenshot.png', fullPage: true });
await locator.screenshot({ path: 'element.png' });

// Assertion visuelle (snapshot testing)
await expect(page).toHaveScreenshot('page-accueil.png');
await expect(locator).toHaveScreenshot('bouton.png');
```

### Traces

La trace enregistre chaque action, screenshot et requête réseau pour le débogage.

```ts
// Dans playwright.config.ts
use: {
  trace: 'on-first-retry',        // enregistre en cas d'échec
  // autres options : 'on', 'off', 'retain-on-failure'
}
```

```bash
npx playwright show-trace trace.zip   # ouvrir une trace
```

### Ouvrir le viewer de traces

Après un test échoué, la trace est disponible dans le rapport HTML (`npx playwright show-report`).

---

## API testing

Playwright peut tester des APIs REST directement, sans navigateur.

```ts
import { test, expect } from '@playwright/test';

test('GET /api/utilisateurs retourne une liste', async ({ request }) => {
  const response = await request.get('/api/utilisateurs');

  expect(response.status()).toBe(200);
  expect(response.ok()).toBeTruthy();

  const body = await response.json();
  expect(body).toHaveLength(3);
  expect(body[0]).toMatchObject({ id: expect.any(Number), nom: expect.any(String) });
});

test('POST /api/utilisateurs crée un utilisateur', async ({ request }) => {
  const response = await request.post('/api/utilisateurs', {
    data: { nom: 'Alice', email: 'alice@exemple.com' },
    headers: { 'Authorization': 'Bearer mon-token' },
  });

  expect(response.status()).toBe(201);
  const created = await response.json();
  expect(created.nom).toBe('Alice');
});

test('DELETE /api/utilisateurs/:id supprime un utilisateur', async ({ request }) => {
  const response = await request.delete('/api/utilisateurs/1');
  expect(response.status()).toBe(204);
});
```

### Combiner API et UI dans un même test

```ts
test('l\'utilisateur créé via API apparaît dans l\'interface', async ({ page, request }) => {
  // Créer via API (plus rapide que via l'UI)
  await request.post('/api/utilisateurs', {
    data: { nom: 'Bob', email: 'bob@exemple.com' },
  });

  // Vérifier dans l'UI
  await page.goto('/utilisateurs');
  await expect(page.getByText('Bob')).toBeVisible();
});
```

---

## Page Object Model

Le POM encapsule les sélecteurs et les interactions d'une page dans une classe réutilisable.

```ts
// pages/ConnexionPage.ts
import { type Page, type Locator, expect } from '@playwright/test';

export class ConnexionPage {
  readonly page: Page;
  readonly champEmail: Locator;
  readonly champMotDePasse: Locator;
  readonly boutonConnexion: Locator;
  readonly messageErreur: Locator;

  constructor(page: Page) {
    this.page = page;
    this.champEmail = page.getByLabel('Email');
    this.champMotDePasse = page.getByLabel('Mot de passe');
    this.boutonConnexion = page.getByRole('button', { name: 'Se connecter' });
    this.messageErreur = page.getByRole('alert');
  }

  async goto() {
    await this.page.goto('/connexion');
  }

  async seConnecter(email: string, motDePasse: string) {
    await this.champEmail.fill(email);
    await this.champMotDePasse.fill(motDePasse);
    await this.boutonConnexion.click();
  }
}
```

```ts
// tests/connexion.spec.ts
import { test, expect } from '@playwright/test';
import { ConnexionPage } from '../pages/ConnexionPage';

test.describe('Connexion', () => {
  test('connexion réussie', async ({ page }) => {
    const connexionPage = new ConnexionPage(page);
    await connexionPage.goto();
    await connexionPage.seConnecter('alice@exemple.com', 'secret');
    await expect(page).toHaveURL('/tableau-de-bord');
  });

  test('erreur si mot de passe incorrect', async ({ page }) => {
    const connexionPage = new ConnexionPage(page);
    await connexionPage.goto();
    await connexionPage.seConnecter('alice@exemple.com', 'mauvais');
    await expect(connexionPage.messageErreur).toContainText('Identifiants invalides');
  });
});
```

---

## Configuration avancée et CI

### Fixtures personnalisées

```ts
// fixtures.ts
import { test as base } from '@playwright/test';
import { ConnexionPage } from './pages/ConnexionPage';

type Fixtures = {
  utilisateurConnecte: ConnexionPage;
};

export const test = base.extend<Fixtures>({
  utilisateurConnecte: async ({ page }, use) => {
    const connexionPage = new ConnexionPage(page);
    await connexionPage.goto();
    await connexionPage.seConnecter('alice@exemple.com', 'secret');
    await use(connexionPage);
  },
});

export { expect } from '@playwright/test';
```

```ts
// tests/tableau-de-bord.spec.ts
import { test, expect } from '../fixtures';

test('affiche le tableau de bord', async ({ utilisateurConnecte, page }) => {
  await expect(page).toHaveURL('/tableau-de-bord');
});
```

### GitHub Actions

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run Playwright tests
        run: npx playwright test

      - name: Upload report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```
