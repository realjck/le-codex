# n8n

n8n est une plateforme d'automatisation de workflows open-source (fair-code), auto-hébergeable. Elle permet de connecter des applications, d'automatiser des tâches répétitives et de construire des agents IA — le tout via une interface visuelle, avec la possibilité d'écrire du code quand nécessaire.

> **Alternatives comparables :** Zapier, Make (ex-Integromat) — mais n8n se distingue par l'auto-hébergement, la flexibilité du code, et les intégrations IA natives.

---

## Installation

### Via npm (usage local)

```bash
npm install -g n8n
n8n start
# Interface disponible sur http://localhost:5678
```

### Via Docker (recommandé)

```bash
docker run -it --rm \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

### Via Docker Compose (production)

```yaml
services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=mon-domaine.com
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://mon-domaine.com/
      - N8N_ENCRYPTION_KEY=ma-cle-secrete   # obligatoire en production
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

### AI Starter Kit (n8n + Ollama + Qdrant)

n8n propose un kit Docker Compose tout-en-un pour les workflows IA locaux :

```bash
git clone https://github.com/n8n-io/self-hosted-ai-starter-kit.git
cd self-hosted-ai-starter-kit
docker compose up
```

Inclut : n8n + Ollama (LLM local) + Qdrant (base vectorielle) + PostgreSQL.

---

## Interface et concepts clés

### Vocabulaire

| Terme | Description |
|---|---|
| **Workflow** | Séquence de nodes connectés qui automatise une tâche |
| **Node** | Brique unitaire : effectue une action ou récupère des données |
| **Trigger** | Node de départ qui déclenche le workflow (webhook, schedule, event…) |
| **Connection** | Lien entre deux nodes qui transmet les données |
| **Execution** | Une instance d'exécution d'un workflow |
| **Item** | Une unité de données qui circule entre les nodes (objet JSON) |
| **Credential** | Identifiants stockés de façon sécurisée pour accéder à des services |

### Structure d'un workflow

```
[Trigger] → [Node 1] → [Node 2] → [Node 3]
                ↓
           [Branch A] → [Node 4]
```

Chaque node reçoit des **items** (tableaux d'objets JSON), les transforme, et les transmet au node suivant.

### Modes d'exécution

- **Manuel** — déclenché depuis l'interface pour tester
- **Actif** — le workflow tourne en production, déclenché par un trigger réel
- **Webhook** — déclenché par un appel HTTP externe

---

## Nodes essentiels

### Nodes de données

| Node | Usage |
|---|---|
| **HTTP Request** | Appeler n'importe quelle API REST |
| **Code** | Exécuter du JavaScript ou Python personnalisé |
| **Set** | Créer ou modifier des champs dans les items |
| **Edit Fields** | Interface visuelle pour transformer les données |
| **Function / Function Item** | Transformer les données avec du JS (legacy) |

### Nodes de flux

| Node | Usage |
|---|---|
| **If** | Condition binaire — sépare le flux en deux branches |
| **Switch** | Condition multiple — plusieurs branches selon une valeur |
| **Merge** | Fusionner plusieurs flux en un |
| **Loop Over Items** | Itérer sur chaque item individuellement |
| **Wait** | Mettre le workflow en pause (durée fixe ou événement) |
| **Stop and Error** | Interrompre le workflow avec une erreur personnalisée |

### Nodes utilitaires

| Node | Usage |
|---|---|
| **Schedule Trigger** | Déclenchement récurrent (cron) |
| **Webhook** | Recevoir des requêtes HTTP entrantes |
| **Email (SMTP/IMAP)** | Envoyer ou lire des emails |
| **Slack / Discord** | Envoyer des messages |
| **Google Sheets** | Lire/écrire des données dans un tableur |
| **Postgres / MySQL** | Requêtes SQL directes |

---

## Triggers et webhooks

### Schedule Trigger (cron)

```
Champs : Seconds / Minutes / Hours / Day of Month / Month / Day of Week
Exemple : "0 9 * * 1-5"  →  tous les jours ouvrés à 9h
```

### Webhook

Le node **Webhook** expose une URL unique pour recevoir des données HTTP :

```
URL de test    : http://localhost:5678/webhook-test/<id>
URL production : http://localhost:5678/webhook/<id>
```

Méthodes supportées : `GET`, `POST`, `PUT`, `DELETE`, `PATCH`.

Options utiles :
- **Respond** : `Immediately` (répond 200 tout de suite) ou `Using Respond to Webhook node` (contrôle la réponse)
- **Authentication** : Header Auth, Basic Auth, ou JWT

### Respond to Webhook

Pour renvoyer une réponse personnalisée à l'appelant :

```json
// Dans le node "Respond to Webhook"
{
  "status": 200,
  "body": {
    "message": "{{ $json.result }}"
  }
}
```

---

## Expressions et code

### Syntaxe des expressions

Les expressions utilisent `{{ }}` et permettent d'accéder dynamiquement aux données :

```javascript
// Données du node précédent
{{ $json.email }}
{{ $json.user.name }}

// Données d'un node spécifique
{{ $('HTTP Request').item.json.status }}

// Métadonnées du workflow
{{ $workflow.name }}
{{ $execution.id }}

// Variables d'environnement
{{ $env.API_KEY }}

// Opérations
{{ $json.price * 1.1 }}
{{ $json.email.toLowerCase() }}
{{ $json.status === 'active' ? 'Oui' : 'Non' }}
{{ new Date().toISOString() }}
```

### Variables built-in

| Variable | Description |
|---|---|
| `$json` | Données JSON de l'item courant |
| `$binary` | Données binaires de l'item courant |
| `$('NomDuNode').item.json` | Données d'un node nommé |
| `$items('NomDuNode')` | Tous les items d'un node |
| `$workflow` | Métadonnées du workflow (`name`, `id`) |
| `$execution` | Métadonnées d'exécution (`id`, `mode`) |
| `$env` | Variables d'environnement |
| `$now` | Date/heure actuelle (objet Luxon) |
| `$today` | Date du jour (objet Luxon) |

### Node Code (JavaScript)

```javascript
// Traiter tous les items
for (const item of $input.all()) {
  item.json.fullName = `${item.json.firstName} ${item.json.lastName}`;
  item.json.timestamp = new Date().toISOString();
}

return $input.all();
```

```javascript
// Créer de nouveaux items
const results = [];

for (const item of $input.all()) {
  if (item.json.active) {
    results.push({
      json: {
        id: item.json.id,
        name: item.json.name,
      }
    });
  }
}

return results;
```

### Node Code (Python)

```python
results = []

for item in _input.all():
    if item.json.get('active'):
        results.append({
            'json': {
                'id': item.json['id'],
                'name': item.json['name'],
            }
        })

return results
```

---

## Credentials

Les credentials stockent de façon chiffrée les clés API, tokens et identifiants nécessaires aux nodes.

### Créer des credentials

`Settings → Credentials → Add Credential` ou directement depuis un node lors de sa configuration.

### Types courants

| Type | Usage |
|---|---|
| **API Key (Header Auth)** | Clé dans l'en-tête `Authorization` ou `X-API-Key` |
| **Basic Auth** | Username + Password |
| **OAuth2** | Flux OAuth (Google, GitHub, Slack…) |
| **Bearer Token** | Token JWT ou Bearer |

### Utiliser en expression

```javascript
// Référencer dynamiquement un credential via une expression (cas avancé)
{{ $json.userCredentials }}
```

### Variables d'environnement comme credentials

Pour éviter de stocker des secrets dans n8n, utiliser des variables d'environnement et les appeler via `{{ $env.MA_CLE }}`.

---

## Intégrations IA

### Connexion à un LLM

n8n intègre nativement LangChain. Les nodes IA se trouvent dans la catégorie **AI** de l'éditeur.

**Nodes LLM disponibles :**
- `OpenAI Chat Model`
- `Anthropic Chat Model` (Claude)
- `Ollama Chat Model` (local)
- `Google Gemini Chat Model`
- `Azure OpenAI`

**Exemple de chaîne IA basique :**

```
[Chat Trigger] → [AI Agent] → [Respond to Webhook]
                     ↑
              [OpenAI Chat Model]
```

### AI Agent

Le node **AI Agent** orchestre un LLM avec des outils. Il peut :
- Appeler des outils (HTTP, code, recherche web…)
- Maintenir une mémoire de conversation
- Suivre une chaîne de raisonnement (ReAct)

```
[AI Agent]
  ├── LLM : Anthropic Chat Model (Claude)
  ├── Memory : Window Buffer Memory
  └── Tools : [HTTP Request Tool] [Code Tool] [Calculator]
```

### Mémoire de conversation

| Node mémoire | Description |
|---|---|
| **Window Buffer Memory** | Conserve les N derniers messages |
| **Postgres Chat Memory** | Persiste la mémoire en base de données |
| **Redis Chat Memory** | Persiste la mémoire dans Redis |

### Intégration MCP

n8n peut agir comme **serveur MCP** (exposer ses workflows à des agents IA externes) ou comme **client MCP** (consommer des outils MCP).

**n8n comme serveur MCP :**

```
[MCP Server Trigger] → [mes nodes n8n] → [réponse]
```

Accessible sur `http://localhost:5678/mcp-server/http` avec un token Bearer.

**n8n comme client MCP :**

Le node **MCP Client Tool** permet à un AI Agent n8n d'appeler des outils exposés par un serveur MCP externe.

```
[AI Agent]
  └── Tools : [MCP Client Tool → serveur MCP externe]
```

### Exemple : workflow IA avec webhook

```
[Webhook POST /chat]
    ↓
[AI Agent]
  ├── LLM : Claude (Anthropic Chat Model)
  ├── Memory : Window Buffer Memory
  └── Tools : [HTTP Request Tool]
    ↓
[Respond to Webhook → réponse JSON]
```
