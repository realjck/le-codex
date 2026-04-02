# Prompt Engineering

Le prompt engineering consiste à formuler les instructions envoyées à un LLM pour obtenir des résultats précis, fiables et reproductibles. C'est la compétence la plus rentable à développer : elle s'applique à tous les modèles et tous les outils.

---

## Anatomie d'un bon prompt

Un prompt peut contenir jusqu'à quatre éléments :

```
[Rôle]       Tu es un développeur senior spécialisé en Python.
[Contexte]   Je travaille sur une API REST avec FastAPI.
[Tâche]      Écris une fonction qui valide un email.
[Contrainte] Utilise uniquement la bibliothèque standard. Ajoute des docstrings.
```

Tous ne sont pas toujours nécessaires — mais plus la tâche est complexe, plus les préciser améliore le résultat.

---

## Zero-shot vs Few-shot

### Zero-shot — aucun exemple

On demande directement, sans montrer d'exemple.

```
Classifie ce commentaire comme positif, négatif ou neutre :
"La livraison était rapide mais l'emballage abîmé."
```

Suffisant pour les tâches simples ou bien connues du modèle.

### Few-shot — avec exemples

On montre des exemples du résultat attendu avant de poser la vraie question.

```
Classifie chaque commentaire comme positif, négatif ou neutre.

Commentaire : "Excellent produit, je recommande !"
Classification : positif

Commentaire : "Délai de livraison beaucoup trop long."
Classification : négatif

Commentaire : "Le produit est conforme à la description."
Classification : neutre

Commentaire : "La livraison était rapide mais l'emballage abîmé."
Classification :
```

Le few-shot est particulièrement utile pour :
- Formater des sorties précises
- Imposer un ton ou un style
- Guider des tâches de classification

---

## Chain-of-Thought (CoT)

Demander au modèle de **raisonner étape par étape** avant de donner une réponse. Améliore significativement les résultats sur les tâches logiques, mathématiques ou multi-étapes.

### Sans CoT

```
Un train part à 9h et roule à 120 km/h. Il doit parcourir 300 km.
À quelle heure arrive-t-il ?
```

### Avec CoT

```
Un train part à 9h et roule à 120 km/h. Il doit parcourir 300 km.
Réfléchis étape par étape puis donne l'heure d'arrivée.
```

### CoT forcé avec déclencheur

Ajouter simplement `Réfléchis étape par étape.` ou `Think step by step.` à la fin d'un prompt déclenche automatiquement le raisonnement chaîné.

---

## Rôles et personnalité (Role Prompting)

Donner un rôle au modèle améliore la qualité et le ton des réponses.

```
Tu es un architecte logiciel senior avec 15 ans d'expérience en systèmes distribués.
Revois cette architecture et identifie les points de défaillance potentiels.
```

```
Tu es un professeur pédagogue qui explique des concepts complexes
à des débutants sans jargon technique.
Explique ce qu'est une API REST.
```

Le rôle influence :
- Le niveau de détail
- Le vocabulaire utilisé
- Le point de vue adopté (critique, bienveillant, exhaustif…)

---

## Structurer les sorties

### Forcer un format JSON

```
Extrais les informations de ce texte et retourne-les en JSON.
N'inclus rien d'autre que le JSON dans ta réponse.

Format attendu :
{
  "nom": "...",
  "email": "...",
  "entreprise": "..."
}

Texte : "Bonjour, je m'appelle Marie Dupont, développeuse chez Acme Corp.
Mon email est marie@acme.fr"
```

### Forcer un format Markdown structuré

```
Génère un rapport d'analyse avec exactement cette structure :

## Résumé
(2-3 phrases)

## Points positifs
(liste à puces)

## Points d'amélioration
(liste à puces)

## Recommandation
(1 paragraphe)
```

### Limiter la longueur

```
Explique Docker en 3 phrases maximum, sans jargon.
```

```
Donne-moi 5 idées de noms pour une app de gestion de tâches.
Une idée par ligne, sans explication.
```

---

## Prompts pour le code

### Générer du code

```
Écris une fonction Python qui :
- Prend en entrée une liste de dictionnaires avec les clés "nom" et "score"
- Retourne les 3 meilleurs scores triés par ordre décroissant
- Gère le cas où la liste est vide

Ajoute des type hints et une docstring.
```

### Déboguer

```
Ce code Python lève une KeyError. Identifie le problème et propose une correction.
Explique pourquoi l'erreur se produit.

```python
def get_user_name(users, user_id):
    return users[user_id]["name"]

users = {1: {"username": "alice"}}
print(get_user_name(users, 1))
```
```

### Refactoriser

```
Refactorise ce code pour améliorer la lisibilité.
Garde exactement le même comportement.
Explique chaque changement et pourquoi il améliore le code.
```

### Revue de code

```
Fais une revue de code de la fonction suivante.
Identifie : les bugs potentiels, les problèmes de performance,
les violations de bonnes pratiques Python.
Sois direct et concis.
```

---

## Techniques avancées

### Décomposition de tâche

Pour les tâches complexes, décomposer explicitement en sous-tâches.

```
Pour créer cette feature, procède dans cet ordre :
1. D'abord, lis les fichiers concernés et décris l'architecture existante
2. Ensuite, propose un plan de modification
3. Attends ma validation avant d'écrire le moindre code
4. Implémente en commençant par les tests
```

### Self-consistency (auto-vérification)

Demander au modèle de vérifier son propre travail.

```
Écris une fonction qui calcule le PGCD de deux nombres.
Ensuite, teste mentalement ta fonction avec les cas : (12, 8), (7, 0), (0, 0).
Corrige si nécessaire.
```

### Contrôle de la certitude

```
Réponds à la question suivante. Si tu n'es pas certain,
dis explicitement "Je ne suis pas sûr" plutôt que d'inventer.

Quelle est la date de sortie de Python 3.13 ?
```

### Persona contradictoire (Devil's Advocate)

```
Je vais te présenter mon architecture technique.
Joue le rôle d'un développeur sceptique et identifie tous les problèmes potentiels.
Sois critique, pas complaisant.
```

### Reformulation pour clarifier

Si le modèle donne une réponse trop vague :

```
Ta réponse est trop générale. Donne-moi un exemple concret avec du vrai code.
```

```
C'est trop long. Résume en 5 points essentiels.
```

```
Tu n'as pas répondu à ma question. Ma question était : [reformuler]
```

---

## Itérer un prompt

Un prompt rarement parfait du premier coup. Voici le cycle à suivre :

```
1. Prompt initial simple
        ↓
2. Analyser ce qui manque ou cloche dans la réponse
        ↓
3. Ajouter une contrainte ou précision ciblée
        ↓
4. Répéter jusqu'au résultat souhaité
        ↓
5. Consolider le prompt final dans un fichier réutilisable
```

### Exemple d'itération

**V1 — trop vague :**
```
Écris une description de produit pour des écouteurs.
```

**V2 — avec contexte :**
```
Écris une description de produit pour des écouteurs sans fil haut de gamme.
Cible : audiophiles de 30-45 ans.
```

**V3 — avec format :**
```
Écris une description de produit pour des écouteurs sans fil haut de gamme.
Cible : audiophiles de 30-45 ans.
Format : accroche (1 phrase) + 3 arguments clés (liste) + call-to-action (1 phrase).
Ton : expert mais accessible. 120 mots maximum.
```

---

## Anti-patterns à éviter

| Ce qu'on écrit | Problème | Ce qu'il faut écrire |
|---|---|---|
| "Écris un bon code" | Trop vague | "Écris une fonction Python avec type hints, docstring et gestion des erreurs" |
| "Fais quelque chose sur les tests" | Ambigu | "Écris les tests unitaires de la fonction `calculate_tax()`" |
| "Explique tout" | Pas de scope | "Explique en 5 points comment fonctionne l'authentification JWT" |
| "Et si tu pouvais…" | Trop conditionnel | Formuler comme une instruction directe |
| Question ouverte + tâche complexe | Résultat dilué | Séparer en plusieurs prompts distincts |

---

## Prompts système utiles (réutilisables)

### Assistant développeur

```
Tu es un développeur senior full-stack.
- Réponds toujours en français
- Fournis du code fonctionnel et testé
- Signale les cas limites et erreurs potentielles
- Si la question est ambiguë, demande une clarification avant de coder
- Préfère la lisibilité à la concision
```

### Relecteur de code strict

```
Tu es un relecteur de code exigeant.
Analyse le code fourni et liste :
1. Les bugs (critiques en premier)
2. Les problèmes de sécurité
3. Les problèmes de performance
4. Les violations de conventions
Sois direct. Ne complimente pas. Ne propose des améliorations cosmétiques que si tout le reste est correct.
```

### Traducteur technique

```
Tu es un traducteur technique expert.
Traduis le texte de l'anglais vers le français.
Conserve les termes techniques en anglais quand leur traduction française n'est pas d'usage courant.
Ne traduis pas les noms de variables, fonctions ou fichiers.
```

---

## Récapitulatif des techniques

| Technique | Quand l'utiliser | Gain |
|---|---|---|
| **Zero-shot** | Tâches simples et communes | Rapidité |
| **Few-shot** | Format précis, ton spécifique | Cohérence |
| **Chain-of-Thought** | Logique, maths, multi-étapes | Précision |
| **Role prompting** | Adapter ton et expertise | Qualité |
| **Décomposition** | Tâches complexes | Contrôle |
| **Auto-vérification** | Code, calculs | Fiabilité |
| **Contrôle certitude** | Questions factuelles | Moins d'hallucinations |
| **Itération** | Tout prompt important | Amélioration continue |
