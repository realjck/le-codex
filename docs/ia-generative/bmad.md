# La méthode BMAD

> **B**uild **M**ore **A**rchitect **D**reams

La **méthode BMAD** (BMAD-METHOD) est un framework open-source de développement logiciel piloté par l'IA (AI-Driven Development). Elle a pour objectif de structurer l'utilisation d'assistants IA (comme Claude Code, Cursor ou Codex CLI) en remplaçant les prompts chaotiques par des **flux de travail rigoureux**, une **gestion de contexte stricte** et des **agents spécialisés**.

Idéale pour un développeur freelance cherchant à multiplier sa force de frappe, elle permet de passer de l'idéation à l'implémentation agentique avec une précision chirurgicale, tout en documentant le processus.

[https://github.com/bmad-code-org/BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD)

---

## 1. Le Concept des "Named Agents" (Agents Nommés)

BMAD refuse le paradigme du "prompt vide" ou du chatbot générique. Le framework introduit la notion d'**Agents Nommés**, qui apportent une continuité de persona et des capacités spécifiques :
- Chaque agent possède une identité, un ton, des principes et des compétences propres.
- Les agents sont personnalisables via des fichiers de configuration (`customize.toml`), ce qui relève de l'approche *Agent as Code*.
- Il est possible de déclencher le **Party Mode** (`bmad-party-mode`) pour mettre toute l'équipe virtuelle dans la même "pièce" (le même terminal) et les faire débattre sur une décision complexe.

### L'Équipe par Défaut
* 🧠 **Mary (Analyste) :** Experte en brainstorming, recherche et cadrage de produit.
* 📋 **John (Product Manager) :** Rédige les spécifications, le PRD et découpe le travail.
* 🏗️ **Winston (Architecte) :** Prend les décisions techniques, crée les ADRs (Architecture Decision Records).
* 💻 **Amelia (Développeuse) :** Écrit le code, effectue les revues, écrit les tests.
* 🎨 **Sally (UX Designer) :** Pense l'expérience utilisateur et les interfaces.
* ✍️ **Paige (Rédactrice Technique) :** Produit la documentation, les diagrammes et les guides.

---

## 2. Installation et Prérequis

BMAD s'intègre directement dans votre projet local. Il n'a pas besoin de serveurs externes, il utilise simplement l'outil IA de votre choix dans votre terminal.

**Prérequis :**
* Node.js v20.12+ (nécessaire pour l'installateur).
* Git.
* Un outil CLI d'IA (ex: **Claude Code** est fortement recommandé pour un usage fluide en terminal) ou un IDE comme Cursor.

**Commande d'installation (à lancer à la racine de votre projet) :**
```bash
npx bmad-method install

```

*Lors de l'installation, vous pourrez sélectionner les modules à activer : BMM (le coeur Agile), BMB (BMad Builder pour créer vos agents), CIS (Creative Intelligence Suite), etc.*

> **Le réflexe de survie :** Une fois installé, si vous êtes perdu, invoquez la commande `bmad-help` dans votre IDE. Ce guide intelligent analyse l'état de votre projet et vous dit exactement quoi faire ensuite.

---

## 3. Le Workflow en 4 Phases

Le framework impose que **le contexte et la documentation pilotent le code**. Le travail est divisé en 4 phases strictes. Chaque phase génère des fichiers Markdown qui deviennent la source de vérité pour la phase suivante.

*Règle d'or : Ouvrez un "Fresh Chat" (une nouvelle session) pour chaque nouveau workflow afin d'éviter la saturation du contexte.*

### Phase 1 : Analyse (Optionnelle)

Destinée à l'idéation et à la validation des concepts.

* `bmad-brainstorming` : Séance guidée avec Mary pour générer des idées.
* `bmad-product-brief` : Crée le document de fondation (`product-brief.md`).

### Phase 2 : Planification (Requise)

Traduction de l'idée en spécifications claires.

* `bmad-prd` : Workflow de création du *Product Requirements Document*. Génère les fichiers `prd.md`, `addendum.md` et `decision-log.md`.
* `bmad-ux` : Si le projet a une interface, Sally générera le `ux-spec.md`.

### Phase 3 : Solution & Architecture (Cruciale)

C'est ici que l'approche "Architect Dreams" prend tout son sens. Aucune ligne de code n'est encore écrite.

* `bmad-create-architecture` : Winston décide de la stack technologique (par exemple, choix entre Next.js avec Tailwind ou un backend Python/FastAPI) et rédige `architecture.md`.
* `bmad-create-epics-and-stories` : Le travail est découpé en tâches implémentables (Epics et Stories) en se basant sur les contraintes architecturales.
* `bmad-check-implementation-readiness` : Vérification finale pour s'assurer qu'il n'y a pas d'incohérences avant de coder.

### Phase 4 : Implémentation (Le Code)

Amelia prend le relais pour coder fonctionnalité par fonctionnalité.

* `bmad-sprint-planning` : Initialise le suivi (`sprint-status.yaml`).
* `bmad-create-story` : Prépare le contexte pour la prochaine story.
* `bmad-dev-story` : Implémentation de la story (écriture du code et des tests).
* `bmad-code-review` : Revue de code automatisée par l'agent.

---

## 4. Piste Accélérée : Le "Quick Flow"

Pour des projets très simples, des scripts d'automatisation (ex: workflows n8n) ou des preuves de concept, les phases 1 à 3 peuvent sembler trop lourdes.
BMAD propose la commande :

```bash
bmad-quick-dev

```

Cette piste unique gère l'intention, le plan, l'implémentation et la revue en un seul flux continu, générant directement les specs courtes et le code.

---

## 5. Bonnes Pratiques & Gestion du Contexte

* **`project-context.md` :** Créez (ou générez avec `bmad-generate-project-context`) ce fichier dans le dossier `_bmad-output/`. Il est vital ! C'est ici que vous imposez vos préférences de développeur (ex: "Utiliser TypeScript strict, préférer les composants fonctionnels React, utiliser des docstrings Python claires"). Tous les agents liront ce fichier avant de travailler.
* **Ne pas fusionner les rôles :** Laissez l'Architecte faire l'architecture et la Développeuse coder. Ce cloisonnement garantit que le code respecte le design, évitant ainsi le piège des IA qui codent à l'aveugle.
* **Le code est éphémère, la spec est reine :** Si vous devez changer un comportement, ne demandez pas à l'agent de "juste modifier le code". Demandez-lui d'abord de mettre à jour le PRD ou l'architecture, puis de répercuter cela dans le code.
