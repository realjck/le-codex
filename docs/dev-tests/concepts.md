# Concepts de tests

Les tests automatisés vérifient que le code se comporte comme attendu, maintenant et après chaque modification. Ils sont l'un des investissements les plus rentables en développement logiciel.

---

## Pourquoi tester ?

| Sans tests | Avec tests |
|---|---|
| Peur de modifier le code existant | Confiance pour refactorer |
| Bugs découverts en production | Bugs détectés en développement |
| Régression silencieuse | Régression détectée immédiatement |
| Documentation obsolète | Tests = documentation vivante |
| Déploiements risqués | Déploiements continus possibles |

Les tests ne garantissent pas l'absence de bugs — ils garantissent que les comportements testés fonctionnent. Un code non testé est un code dont on ne sait pas s'il fonctionne.

---

## Les types de tests

### Tests unitaires

Testent une **unité isolée** de code (fonction, méthode, classe) sans ses dépendances externes.

```
[Fonction] ← [Test unitaire]
```

- **Vitesse** : très rapides (millisecondes)
- **Isolation** : les dépendances sont remplacées par des mocks/stubs
- **Quantité** : la majorité des tests d'un projet
- **Exemples** : tester qu'une fonction de calcul retourne le bon résultat, qu'un validateur rejette les entrées invalides

### Tests d'intégration

Testent l'**interaction entre plusieurs composants** : un service et sa base de données, deux modules qui collaborent, une API et son ORM.

```
[Module A] + [Module B] ← [Test d'intégration]
```

- **Vitesse** : plus lents (requêtes réelles, I/O)
- **Isolation** : partielle — on peut mocker certaines parties mais pas toutes
- **Quantité** : moins nombreux que les tests unitaires
- **Exemples** : tester qu'un appel à l'API retourne bien les données de la base, qu'un service d'email envoie vraiment un message

### Tests end-to-end (e2e)

Testent **l'application entière** du point de vue de l'utilisateur, via un navigateur ou un client HTTP.

```
[Navigateur/Client] → [Frontend] → [API] → [BDD] ← [Test e2e]
```

- **Vitesse** : lents (plusieurs secondes par test)
- **Isolation** : aucune — l'environnement doit être complet
- **Quantité** : peu nombreux, couvrent les parcours critiques
- **Exemples** : tester qu'un utilisateur peut s'inscrire, se connecter et passer une commande

### Autres types

| Type | Description | Déclenchement |
|---|---|---|
| **Smoke tests** | Vérifications minimales que l'app démarre et répond | Après chaque déploiement |
| **Tests de régression** | Vérifient qu'un bug corrigé ne revient pas | À chaque PR |
| **Tests de performance** | Mesurent les temps de réponse sous charge | Périodiquement |
| **Tests de snapshot** | Comparent le rendu actuel à une référence | À chaque PR (UI) |
| **Tests de mutation** | Vérifient la qualité des tests eux-mêmes | Audit qualité |

---

## La pyramide des tests

La pyramide définit l'**équilibre recommandé** entre les types de tests :

```
        /\
       /e2e\          ← peu nombreux, lents, coûteux
      /──────\
     /intégra-\       ← nombre modéré
    /  tion    \
   /────────────\
  /   unitaires  \    ← nombreux, rapides, bon marché
 /────────────────\
```

### Règles d'équilibre

- **Tests unitaires** : 70-80% — rapides, fiables, faciles à maintenir
- **Tests d'intégration** : 15-20% — vérifient les contrats entre composants
- **Tests e2e** : 5-10% — couvrent uniquement les parcours critiques

### L'anti-pattern : le cône de glace

```
  /────────────────\
 /      e2e         \   ← trop nombreux → lents, fragiles, coûteux
/──────────────────────\
\    intégration       /
 \────────────────────/
  \    unitaires      /
   \────────────────/    ← trop peu → peu de valeur
```

Une suite de tests trop orientée e2e est lente, fragile (un changement UI casse tout) et difficile à déboguer.

---

## TDD — Test Driven Development

Le TDD inverse l'ordre naturel : **on écrit le test avant le code**.

### Le cycle Red / Green / Refactor

```
    ┌─────────────────────────────────────┐
    │                                     │
    ▼                                     │
  RED                                     │
  Écrire un test qui échoue               │
  (le code n'existe pas encore)           │
    │                                     │
    ▼                                     │
  GREEN                                   │
  Écrire le minimum de code               │
  pour faire passer le test               │
    │                                     │
    ▼                                     │
  REFACTOR ────────────────────────────────┘
  Améliorer le code sans
  changer son comportement
```

### Exemple de workflow TDD

```
1. Je dois implémenter une fonction calculateTax(price, rate)

2. RED — j'écris d'abord :
   test('calcule la TVA à 20%', () => {
     expect(calculateTax(100, 0.20)).toBe(20)
   })
   → Le test échoue car calculateTax n'existe pas

3. GREEN — j'écris le minimum :
   function calculateTax(price, rate) {
     return price * rate
   }
   → Le test passe

4. REFACTOR — j'améliore si nécessaire :
   function calculateTax(price, rate) {
     return Math.round(price * rate * 100) / 100  // arrondi à 2 décimales
   }
   → Le test passe toujours
```

### Avantages

- Force à réfléchir à l'interface avant l'implémentation
- Garantit que tout le code est couvert par des tests
- Facilite le refactoring en toute confiance
- Documentation automatique du comportement attendu

### Quand l'utiliser

✅ Logique métier complexe, algorithmes, validations  
✅ Code qui sera modifié fréquemment  
✅ APIs publiques et contrats d'interface  
❌ Prototypes et explorations rapides  
❌ Code d'infrastructure / configuration  
❌ Interfaces utilisateur (préférer les tests e2e)

---

## BDD — Behaviour Driven Development

Le BDD étend le TDD en formulant les tests en **langage naturel**, compréhensible par tous (développeurs, PO, QA, clients).

### La structure Given / When / Then

```
Given  [contexte initial — l'état du système]
When   [action réalisée — l'événement déclencheur]
Then   [résultat attendu — le comportement observable]
```

### Exemple

```gherkin
# Langage Gherkin (utilisé avec Cucumber, Behat…)
Feature: Authentification utilisateur

  Scenario: Connexion avec des identifiants valides
    Given un utilisateur avec l'email "alice@exemple.com" et le mot de passe "secret"
    When il soumet le formulaire de connexion
    Then il est redirigé vers le tableau de bord
    And un token JWT est retourné

  Scenario: Connexion avec un mauvais mot de passe
    Given un utilisateur avec l'email "alice@exemple.com"
    When il soumet le mot de passe "mauvais"
    Then il reçoit une erreur "Identifiants invalides"
    And aucun token n'est retourné
```

### BDD sans Gherkin

On peut appliquer la structure Given/When/Then sans outil dédié, directement dans les noms de tests :

```javascript
// Jest / Vitest
describe('Authentification', () => {
  describe('quand les identifiants sont valides', () => {
    it('redirige vers le tableau de bord', () => { ... })
    it('retourne un token JWT', () => { ... })
  })

  describe('quand le mot de passe est incorrect', () => {
    it('retourne une erreur 401', () => { ... })
    it('ne retourne pas de token', () => { ... })
  })
})
```

### TDD vs BDD

| | TDD | BDD |
|---|---|---|
| **Focus** | Comment le code fonctionne | Ce que le système fait |
| **Langage** | Technique | Naturel / métier |
| **Public** | Développeurs | Développeurs + PO + QA |
| **Granularité** | Unitaire | Comportement fonctionnel |
| **Outils** | Jest, Vitest, pytest… | Cucumber, Behat, Jest… |
| **Complémentaires ?** | Oui — BDD s'appuie sur TDD |  |

---

## Bonnes pratiques

### Écrire de bons tests

**F.I.R.S.T.**

| Lettre | Principe | Description |
|---|---|---|
| **F** | Fast | Un test doit s'exécuter en millisecondes |
| **I** | Independent | Les tests ne doivent pas dépendre les uns des autres |
| **R** | Repeatable | Même résultat à chaque exécution, quel que soit l'environnement |
| **S** | Self-validating | Le test dit lui-même s'il passe ou échoue (pas de vérification manuelle) |
| **T** | Timely | Écrit au bon moment (idéalement avant ou avec le code) |

### Nommage des tests

Un bon nom de test décrit **le comportement attendu**, pas l'implémentation :

```
❌ test('fonction login')
❌ test('test 1')
✅ test('retourne une erreur si le mot de passe est vide')
✅ test('envoie un email de confirmation après inscription')
```

### Ce qu'il faut tester

```
✅ La logique métier (calculs, règles, validations)
✅ Les cas limites (valeurs nulles, vides, limites max/min)
✅ Les cas d'erreur (que se passe-t-il quand ça échoue ?)
✅ Les contrats d'interface (ce que retourne une fonction)

❌ Les détails d'implémentation (comment c'est fait en interne)
❌ Le code de librairies tierces
❌ Les getters/setters triviaux
```

### Coverage : indicateur, pas objectif

La couverture de code mesure le pourcentage de code exécuté par les tests. Un taux élevé est rassurant, mais :

- 100% de coverage ≠ absence de bugs
- Un test mal écrit peut couvrir sans tester
- Viser 80% est souvent plus pragmatique que 100%

> Un bon test qui échoue quand le comportement change vaut mieux que 10 tests qui passent toujours.

### Mocks, stubs et spies

| Terme | Rôle | Exemple |
|---|---|---|
| **Mock** | Remplace une dépendance et vérifie les appels | Vérifier qu'un service email a bien été appelé |
| **Stub** | Remplace une dépendance et retourne une valeur fixe | Simuler une réponse d'API |
| **Spy** | Observe un vrai objet sans le remplacer | Vérifier qu'une méthode a été appelée N fois |
| **Fake** | Implémentation simplifiée (ex. base en mémoire) | Base de données in-memory pour les tests |
