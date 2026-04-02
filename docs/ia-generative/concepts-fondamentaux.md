# Concepts fondamentaux de l'IA générative

Ce guide couvre les notions essentielles pour comprendre et utiliser efficacement les outils d'IA générative, que ce soit via une interface comme ChatGPT, un modèle local avec Ollama, ou une API.

---

## Les modèles de langage (LLM)

Un **LLM** (Large Language Model) est un modèle entraîné sur d'immenses quantités de texte pour prédire le mot (ou token) suivant. Il ne "comprend" pas au sens humain — il calcule la suite la plus probable.

### Familles de modèles courants

| Éditeur | Modèles | Accès |
|---|---|---|
| Anthropic | Claude (Haiku, Sonnet, Opus) | API / claude.ai |
| OpenAI | GPT-4o, o1, o3 | API / ChatGPT |
| Google | Gemini Flash, Pro, Ultra | API / Gemini |
| Meta | Llama 3.x | Open source / local |
| Mistral | Mistral, Mixtral | API / open source |
| Microsoft | Phi-3, Phi-4 | Open source / local |

Les modèles **open source** (Llama, Mistral, Phi…) peuvent tourner localement via Ollama. Les modèles propriétaires (Claude, GPT…) nécessitent une API payante.

---

## Tokens — l'unité de base

Les LLM ne lisent pas des mots mais des **tokens** : fragments de texte découpés selon un algorithme. Un token ≈ ¾ d'un mot en anglais, un peu moins en français.

```
"Bonjour tout le monde"  →  ["Bon", "jour", " tout", " le", " monde"]  = 5 tokens
```

### Pourquoi c'est important

- **Coût API** : facturé au token (entrée + sortie)
- **Limite de contexte** : la fenêtre de contexte est exprimée en tokens
- **Vitesse** : plus de tokens = génération plus lente

### Estimations pratiques

| Contenu | Tokens approximatifs |
|---|---|
| 1 page A4 de texte | ~500 tokens |
| 1 fichier de code (100 lignes) | ~300–600 tokens |
| Ce document entier | ~1 500 tokens |

---

## La fenêtre de contexte (Context Window)

La fenêtre de contexte est la **quantité totale de texte** qu'un modèle peut traiter en une seule fois : votre historique de conversation + vos fichiers + sa réponse.

```
[System prompt] + [Historique] + [Message actuel] + [Réponse] ≤ Context window
```

### Tailles typiques (2025)

| Modèle | Context window |
|---|---|
| Claude Sonnet/Opus | 200 000 tokens (~150 000 mots) |
| GPT-4o | 128 000 tokens |
| Llama 3.3 70B | 128 000 tokens |
| Mistral 7B | 32 000 tokens |

⚠️ **Important** : le modèle n'a **pas de mémoire entre les sessions**. Chaque nouvelle conversation repart de zéro. Ce qui ressemble à de la mémoire (ChatGPT, Claude) est une injection automatique de résumés dans le contexte.

---

## Les rôles dans un échange

Un échange avec un LLM est structuré en trois types de messages :

| Rôle | Description | Exemple |
|---|---|---|
| `system` | Instructions permanentes données au modèle | "Tu es un assistant expert en Python. Réponds toujours en français." |
| `user` | Message de l'utilisateur | "Comment lire un fichier CSV ?" |
| `assistant` | Réponse générée par le modèle | "Voici comment lire un CSV avec pandas…" |

Le **system prompt** est la base du comportement du modèle. C'est là que l'on définit son rôle, ses contraintes, son ton.

---

## Température et paramètres de génération

Ces paramètres contrôlent le comportement du modèle lors de la génération.

### Température (`temperature`)

Contrôle la **créativité vs précision** des réponses.

| Valeur | Comportement | Usage |
|---|---|---|
| `0.0` | Déterministe, toujours la même réponse | Code, données, extraction |
| `0.3–0.7` | Équilibre (défaut courant) | Rédaction, analyse |
| `1.0+` | Créatif, varié, imprévisible | Brainstorming, fiction |

### Autres paramètres courants

| Paramètre | Rôle |
|---|---|
| `max_tokens` | Longueur maximale de la réponse |
| `top_p` | Filtrage par probabilité cumulée (alternative à temperature) |
| `stop` | Séquences de texte qui stoppent la génération |

---

## Types de modèles selon la tâche

Les LLM texte ne font pas tout. Il existe des modèles spécialisés :

| Type | Usage | Exemples |
|---|---|---|
| **LLM (texte)** | Conversation, code, analyse, rédaction | Claude, GPT-4o, Llama |
| **Embedding** | Transformer du texte en vecteurs numériques (pour la recherche sémantique) | text-embedding-3, nomic-embed |
| **Image generation** | Créer des images à partir d'un texte | DALL-E 3, Stable Diffusion, Flux |
| **Vision** | Analyser des images | Claude, GPT-4o, LLaVA |
| **Speech-to-text** | Transcrire l'audio | Whisper |
| **Text-to-speech** | Synthèse vocale | ElevenLabs, OpenAI TTS |

---

## Inférence locale vs API cloud

| | Local (Ollama, LM Studio) | Cloud (Claude, GPT) |
|---|---|---|
| **Coût** | Gratuit (hors matériel) | Payant au token |
| **Confidentialité** | Données ne quittent pas la machine | Données envoyées au serveur |
| **Performance** | Limitée par votre GPU | Très haute |
| **Qualité** | Modèles open source (légèrement inférieurs) | Meilleurs modèles du marché |
| **Disponibilité** | Hors ligne possible | Nécessite Internet |

### Règle pratique

- Données sensibles ou usage intensif → **local**
- Meilleure qualité ou tâche ponctuelle → **cloud**

---

## RAG — Retrieval-Augmented Generation

Le RAG permet à un LLM de **répondre en s'appuyant sur vos propres documents**, sans fine-tuning.

```
Question utilisateur
        ↓
Recherche dans la base de documents (via embeddings)
        ↓
Passages pertinents injectés dans le contexte
        ↓
LLM génère une réponse basée sur ces passages
```

C'est ce qui permet à un chatbot de "connaître" votre documentation interne, vos PDF, vos bases de données — sans rien envoyer à l'entraînement du modèle.

---

## Fine-tuning vs Prompt Engineering vs RAG

Trois approches pour adapter un LLM à vos besoins :

| Approche | Principe | Quand l'utiliser |
|---|---|---|
| **Prompt Engineering** | Écrire de meilleurs prompts | Toujours — c'est la base |
| **RAG** | Injecter vos données dans le contexte | Quand le LLM doit connaître vos documents |
| **Fine-tuning** | Ré-entraîner le modèle sur vos données | Quand vous voulez changer le style/comportement profondément |

Pour 90% des cas, le **prompt engineering** suffit. Le RAG couvre la plupart des 10% restants. Le fine-tuning est rare et coûteux.

---

## Hallucinations

Un LLM peut **inventer des faits** avec une totale assurance. Ce n'est pas un bug : c'est inhérent au fonctionnement (le modèle prédit ce qui est plausible, pas ce qui est vrai).

### Comment les réduire

- Demander au modèle de citer ses sources
- Lui fournir les documents de référence (RAG)
- Baisser la température pour les tâches factuelles
- Lui demander explicitement : *"Si tu n'es pas sûr, dis-le"*
- Toujours vérifier les informations critiques (dates, chiffres, URLs)

---

## Glossaire rapide

| Terme | Définition |
|---|---|
| **LLM** | Large Language Model — modèle de langage de grande taille |
| **Token** | Unité de traitement du texte (~¾ de mot) |
| **Context window** | Quantité max de texte traitée en une fois |
| **System prompt** | Instructions permanentes définissant le comportement du modèle |
| **Température** | Paramètre de créativité (0 = précis, 1+ = créatif) |
| **Embedding** | Représentation vectorielle d'un texte pour la recherche sémantique |
| **RAG** | Retrieval-Augmented Generation — enrichir les réponses avec vos documents |
| **Fine-tuning** | Ré-entraînement du modèle sur des données spécifiques |
| **Inférence** | Le fait de faire tourner un modèle pour obtenir une réponse |
| **Hallucination** | Fait inventé présenté avec assurance par le LLM |
| **Prompt** | Le texte envoyé au modèle pour obtenir une réponse |
| **Open source** | Modèle dont les poids sont publics et utilisables localement |
