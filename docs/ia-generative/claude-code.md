# Claude Code — Protocoles & méthodes avancées

Claude Code est le CLI officiel d'Anthropic. Il dépasse le simple chat : il lit le code, exécute des commandes, gère Git, et peut opérer de manière autonome via un écosystème de protocoles extensibles.

---

## CLAUDE.md — Fichier d'instructions persistant

Fichier Markdown lu automatiquement à chaque session. Sert à donner du contexte permanent à Claude.

| Emplacement | Portée |
|---|---|
| `~/.claude/CLAUDE.md` | Global (tous les projets) |
| `./CLAUDE.md` | Projet courant |
| `./src/CLAUDE.md` | Sous-dossier spécifique |

```markdown
# Mon projet

## Stack
- Backend : Python / FastAPI
- Frontend : React + TypeScript

## Conventions
- Tests obligatoires pour chaque nouvelle fonction
- Ne jamais commiter sur main directement
```

Les fichiers `CLAUDE.md` imbriqués sont tous chargés : global → racine projet → sous-dossiers.

---

## Commandes slash intégrées

Disponibles dans la session interactive en tapant `/`.

### Navigation & session

| Commande | Description |
|---|---|
| `/help` | Affiche l'aide |
| `/context` | Affiche l'utilisation du contexte : tokens utilisés/disponibles par catégorie (system prompt, outils, MCP, agents, mémoire, skills, messages), outils MCP chargés, agents et skills actifs |
| `/compact` | Compresse l'historique avec un résumé |
| `/clear` | Efface l'historique de conversation |
| `/status` | Statut du compte et du système |
| `/cost` | Affiche la consommation de tokens |
| `/exit` ou `/quit` | Quitte Claude Code |

### Configuration

| Commande | Description |
|---|---|
| `/config` | Voir et modifier les paramètres |
| `/model` | Changer de modèle IA |
| `/fast` | Activer/désactiver le mode rapide (Opus 4.6) |
| `/vim` | Activer/désactiver le mode Vim |
| `/memory` | Éditer les fichiers mémoire |
| `/permissions` | Gérer les permissions des outils |

### Projet & outils

| Commande | Description |
|---|---|
| `/init` | Initialise un fichier `CLAUDE.md` dans le projet |
| `/plugin` | Gérer les plugins |
| `/mcp` | Gérer les serveurs MCP |
| `/doctor` | Diagnostiquer l'installation Claude Code |
| `/review` | Demander une revue de code |
| `/pr_comments` | Voir les commentaires d'une PR GitHub |
| `/terminal-setup` | Configurer l'intégration terminal |
| `/release-notes` | Voir les notes de version |

### Raccourci pratique en session

```bash
! git status     # Exécute une commande shell directement dans la session
```

---

## Flags CLI — Mode non-interactif

Utile pour l'automatisation, les scripts CI/CD et les pipelines.

```bash
# Mode one-shot (réponse puis fermeture)
claude -p "Explique la fonction auth() dans auth.py"

# Limiter les outils autorisés
claude -p "Corrige le bug dans auth.py" --allowedTools "Read,Edit,Bash"

# Sortie en JSON (pour parsing programmatique)
claude -p "Liste les fonctions principales" --output-format json

# Spécifier un modèle
claude -p "Analyse ce code" --model claude-opus-4-6

# Interdire certains outils
claude --disallowedTools "Bash,Write"
```

```bash
# Exemple batch sur plusieurs fichiers
for file in $(cat files.txt); do
  claude -p "Migre $file de React vers Vue. Réponds OK ou FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

---


## MCP — Model Context Protocol

Protocole standard d'Anthropic pour connecter des outils et services externes à Claude. Un serveur MCP expose des **outils**, des **ressources** et des **prompts** que Claude peut utiliser.

### Configuration

```json
// ~/.claude/mcp.json (global) ou .mcp.json (projet)
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_xxxx"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/chemin/autorisé"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_URL": "postgresql://user:pass@localhost/db"
      }
    }
  }
}
```

### Commandes de gestion

```bash
claude mcp list                          # Lister les serveurs configurés
claude mcp add monserveur -- npx -y @org/mon-serveur-mcp
claude mcp remove monserveur             # Supprimer un serveur
claude mcp get monserveur                # Détails d'un serveur
```

### Permissions MCP dans settings.json

```json
{
  "enableAllProjectMcpServers": true,
  "enabledMcpjsonServers": ["github", "memory"],
  "disabledMcpjsonServers": ["filesystem"]
}
```

### Types de transport

| Type | Usage |
|---|---|
| `stdio` | Processus local (le plus courant) |
| `sse` | Server-Sent Events (serveur HTTP distant) |

---

## Skills — Instructions métier réutilisables

Les skills sont des fichiers Markdown qui encapsulent un processus ou une méthode de travail. Claude les suit comme une procédure.

### Structure d'un skill

```markdown
---
name: mon-skill
description: Décrit quand ce skill doit être utilisé
---

## Instructions

1. Lire les fichiers concernés avant toute modification
2. Écrire les tests avant le code (TDD)
3. Vérifier que les tests passent avant de conclure
```

### Emplacement

```
~/.claude/skills/           ← Skills globaux
.claude/skills/             ← Skills du projet
~/.claude/plugins/<nom>/skills/  ← Skills d'un plugin
```

### Invocation

Les skills s'invoquent via la commande `/nom-du-skill` ou en mentionnant leur nom dans la conversation. Claude Code charge automatiquement les skills disponibles au démarrage de session.

---

## Plugins — Bundles extensibles

Un plugin regroupe skills, commandes, hooks et serveurs MCP dans un package réutilisable et distribuable.

### Structure d'un plugin

```
mon-plugin/
├── plugin.json          ← Manifeste
├── commands/            ← Commandes slash personnalisées
├── skills/              ← Skills inclus
├── agents/              ← Agents spécialisés
├── hooks/               ← Hooks lifecycle
│   └── hooks.json
└── servers/             ← Serveurs MCP embarqués
    └── .mcp.json
```

### Manifeste plugin.json

```json
{
  "name": "mon-plugin",
  "version": "1.0.0",
  "description": "Description du plugin",
  "commands": ["./commands/"],
  "agents": ["./agents/"],
  "hooks": "./hooks/hooks.json",
  "mcpServers": "./.mcp.json"
}
```

### Utiliser `${CLAUDE_PLUGIN_ROOT}` pour les chemins portables

```json
{
  "mcpServers": {
    "mon-serveur": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/mon-serveur",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  }
}
```

### Commandes de gestion

```bash
claude plugin install github:org/mon-plugin
claude plugin list
claude plugin remove mon-plugin
```

---

## Hooks — Automatisation lifecycle

Les hooks exécutent des scripts ou des prompts automatiquement en réponse à des événements Claude Code.

### Événements disponibles

| Événement | Déclenchement |
|---|---|
| `SessionStart` | Au démarrage d'une session |
| `UserPromptSubmit` | Avant chaque message utilisateur |
| `PreToolUse` | Avant l'exécution d'un outil |
| `PostToolUse` | Après l'exécution d'un outil |
| `Stop` | Quand Claude termine sa réponse |

### Configuration dans settings.json

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/scan-secrets.sh",
            "timeout": 30
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "Cette commande bash est-elle sûre pour la production ? Vérifie les opérations destructives. Réponds 'approve' ou 'deny' avec une explication.",
            "timeout": 15
          }
        ]
      }
    ],
    "SessionStart": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/check-env.sh",
            "timeout": 20
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/hooks/notify.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

### Types de hooks

| Type | Description |
|---|---|
| `command` | Exécute un script shell. Si exit code ≠ 0, bloque l'action. |
| `prompt` | Soumet un prompt à Claude pour approbation dynamique. |

---

## Loops — Boucles agentiques

Les loops permettent de faire tourner Claude en continu sur une tâche répétitive ou en vagues d'agents parallèles.

### Loop périodique (skill `/loop`)

Exécute une commande ou un prompt à intervalle régulier.

```bash
/loop 5m /review          # Relance /review toutes les 5 minutes
/loop 10m "vérifie les logs d'erreur et résume"
```

### Loop autonomous avec agents parallèles

Pattern pour générer des itérations multiples en parallèle via des sous-agents :

```markdown
PHASE 1 : Lire la spécification source
PHASE 2 : Détecter le numéro de la dernière itération dans output_dir/
PHASE 3 : Planifier N directions créatives différentes
PHASE 4 : Déployer N sous-agents en parallèle (Task tool), chacun avec :
  - La spec complète
  - Son numéro d'itération unique
  - Sa direction créative assignée
PHASE 5 (mode infini) : Relancer en vagues de 3-5 jusqu'à saturation du contexte
```

### Loop CI/CD avec `continuous-claude`

```bash
# Boucle avec revue de code à chaque itération
continuous-claude \
  --prompt "Ajoute la feature d'authentification" \
  --max-runs 10 \
  --review-prompt "Lance npm test && npm run lint, corrige les échecs"

# Agents parallèles sur worktrees séparés
continuous-claude --prompt "Ajoute des tests" --max-runs 5 --worktree tests-worker &
continuous-claude --prompt "Refactore le code" --max-runs 5 --worktree refactor-worker &
wait
```

---

## Méthodes avancées

### Agent Teams (expérimental)

Permet plusieurs fenêtres de contexte indépendantes en parallèle.

```bash
# Dans ~/.claude/settings.json ou variable d'environnement
CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### Sous-agents avec outils et MCP spécifiques

```yaml
# .claude/agents/browser-tester.md
---
name: browser-tester
description: Teste les features dans un vrai navigateur via Playwright
mcpServers:
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
---

Teste l'interface en conditions réelles avec Playwright.
```

### Désactiver des sous-agents spécifiques

```json
{
  "permissions": {
    "deny": ["Agent(Explore)", "Agent(mon-agent-dangereux)"]
  }
}
```

```bash
claude --disallowedTools "Agent(Explore)"
```

### Git Worktrees pour l'isolation

Permet de travailler sur plusieurs branches simultanément sans conflits.

```bash
# Créer un worktree isolé
git worktree add ../feature-auth feature/auth

# Claude Code opère dans son propre worktree
cd ../feature-auth && claude
```

### Raccourcis clavier personnalisés

```json
// ~/.claude/keybindings.json
{
  "$schema": "https://www.schemastore.org/claude-code-keybindings.json",
  "bindings": [
    {
      "context": "Chat",
      "bindings": {
        "ctrl+e": "chat:externalEditor",
        "ctrl+s": null
      }
    }
  ]
}
```

### Commandes slash personnalisées

Créer un fichier `.md` dans `~/.claude/commands/` ou `.claude/commands/` pour définir une commande slash.

```markdown
<!-- ~/.claude/commands/deploy.md -->
Vérifie que tous les tests passent avec `npm test`, puis crée un commit
de déploiement et pousse sur la branche `release`.
```

```bash
/deploy    # Invoque la commande custom
```

Les arguments sont accessibles via `$ARGUMENTS` dans le fichier de commande.

---

## Résumé des emplacements clés

| Fichier / Dossier | Rôle |
|---|---|
| `~/.claude/CLAUDE.md` | Instructions globales persistantes |
| `~/.claude/settings.json` | Configuration globale (hooks, permissions, MCP) |
| `~/.claude/mcp.json` | Serveurs MCP globaux |
| `~/.claude/commands/` | Commandes slash globales |
| `~/.claude/skills/` | Skills globaux |
| `~/.claude/keybindings.json` | Raccourcis clavier |
| `.claude/CLAUDE.md` | Instructions du projet |
| `.mcp.json` | Serveurs MCP du projet |
| `.claude/commands/` | Commandes slash du projet |
| `.claude/agents/` | Sous-agents du projet |
