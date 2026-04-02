# Kanban

Kanban est une méthode de gestion du flux de travail issue du système de production Toyota (années 1950). Adaptée au développement logiciel par David Anderson dans les années 2000, elle repose sur la visualisation du travail, la limitation des tâches en cours et l'amélioration continue du flux.

Contrairement à Scrum, Kanban n'impose pas d'itérations fixes ni de rôles définis : c'est une méthode évolutive qui s'adapte aux processus existants.

---

## Principes fondamentaux

Kanban repose sur **4 principes de base** et **6 pratiques fondamentales**.

### Les 4 principes

| Principe | Description |
|---|---|
| **Commencer avec ce qui existe** | Ne pas tout réorganiser — appliquer Kanban sur le processus en place |
| **Accepter le changement évolutif** | Progresser par petites améliorations continues, pas de transformation radicale |
| **Respecter les rôles existants** | Ne pas imposer de nouveaux titres ou responsabilités au départ |
| **Encourager le leadership à tous niveaux** | L'amélioration du flux est l'affaire de toute l'équipe |

### Les 6 pratiques fondamentales

1. **Visualiser le flux de travail** — rendre visible l'état de chaque tâche
2. **Limiter le travail en cours (WIP)** — ne pas démarrer plus que ce qu'on peut finir
3. **Gérer le flux** — surveiller et optimiser la progression des tâches
4. **Rendre les règles explicites** — formaliser les critères de passage entre étapes
5. **Mettre en place des boucles de feedback** — réunions régulières pour inspecter le flux
6. **Améliorer de façon collaborative** — identifier les blocages et expérimenter des solutions

---

## Le tableau Kanban

Le tableau Kanban est l'outil central : il rend visible l'ensemble du flux de travail.

### Structure type

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│   Backlog    │  En cours    │   En revue   │   Terminé    │
│              │   (WIP: 3)   │   (WIP: 2)   │              │
├──────────────┼──────────────┼──────────────┼──────────────┤
│  [ Tâche A ] │  [ Tâche C ] │  [ Tâche E ] │  [ Tâche F ] │
│  [ Tâche B ] │  [ Tâche D ] │              │  [ Tâche G ] │
│  [ Tâche H ] │  [ Tâche I ] │              │              │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

### Les colonnes

Chaque colonne représente une étape du processus. Les colonnes courantes :

| Colonne | Description |
|---|---|
| **Backlog** | Tâches à faire, non encore démarrées |
| **À faire (prêt)** | Tâches priorisées, prêtes à être prises |
| **En cours** | Tâches activement travaillées |
| **En revue / Test** | Tâches terminées en attente de validation |
| **Terminé** | Tâches livrées |

> Les colonnes sont personnalisables selon le processus de l'équipe. Une équipe dev peut avoir : `Backlog → Spécification → Développement → Review → Test → Déployé`.

### Les cartes

Chaque tâche est représentée par une carte contenant :
- Titre et description courte
- Responsable
- Date d'entrée dans la colonne
- Priorité ou classe de service (optionnel)

---

## Limites WIP

Le **WIP (Work In Progress)** est le nombre maximum de tâches autorisées simultanément dans une colonne. C'est le mécanisme le plus puissant de Kanban.

### Pourquoi limiter le WIP ?

- Réduit le multitâche et les changements de contexte
- Révèle les goulots d'étranglement (une colonne qui se remplit)
- Accélère le débit global (loi de Little : `Cycle Time = WIP / Throughput`)
- Force à terminer avant de commencer

### Exemple

```
En cours (WIP: 3) → si 3 tâches sont déjà en cours,
personne ne peut en démarrer une nouvelle.
L'équipe doit d'abord en terminer une.
```

### Comment choisir les limites WIP ?

Il n'y a pas de règle absolue. Un point de départ courant : **nombre de membres de l'équipe − 1**. Ajuster ensuite selon l'observation du flux réel.

---

## Le flux de travail

### Réunions Kanban

Kanban propose des cérémonies légères, toutes optionnelles et adaptables :

| Réunion | Fréquence | Objectif |
|---|---|---|
| **Standup / Daily** | Quotidien | Parcourir le tableau de droite à gauche, identifier les blocages |
| **Replenishment** | Hebdomadaire | Alimenter le backlog avec de nouvelles tâches priorisées |
| **Review / Demo** | À la livraison | Présenter ce qui a été livré |
| **Retrospective** | Régulière | Améliorer le processus |
| **Service Delivery Review** | Mensuel | Analyser les métriques de flux |

> **Astuce Daily Kanban** : parcourir le tableau de **droite à gauche** (des tâches les plus avancées vers le backlog) pour prioriser ce qui est proche de la livraison.

### Classes de service

Les classes de service permettent de différencier les tâches selon leur urgence :

| Classe | Description | Exemple |
|---|---|---|
| **Urgence** | Traitement immédiat, bypass du WIP | Bug critique en production |
| **Date fixe** | Deadline connue | Livraison client le 15 |
| **Standard** | Flux normal | Développement de fonctionnalité |
| **Intangible** | Faible urgence, fort ROI à long terme | Dette technique |

---

## Métriques

### Lead Time

> Temps total entre la **création** d'une tâche et sa **livraison**.

```
Lead Time = date de livraison − date de création
```

Inclut le temps d'attente dans le backlog. Représente la durée perçue par le client.

### Cycle Time

> Temps entre le **début actif** d'une tâche et sa **livraison**.

```
Cycle Time = date de livraison − date de démarrage
```

Mesure l'efficacité réelle de l'équipe. Un cycle time stable et prévisible est l'objectif principal.

### Throughput

> Nombre de tâches **livrées par unité de temps** (ex. par semaine).

Mesure la capacité de livraison de l'équipe. Utile pour les prévisions :

```
Combien de tâches peut-on livrer en 4 semaines ?
→ Si throughput moyen = 8 tâches/semaine → ~32 tâches
```

### Loi de Little

Relation fondamentale entre les trois métriques :

```
Cycle Time = WIP / Throughput
```

Réduire le WIP ou augmenter le throughput réduit le cycle time.

### Diagramme de flux cumulatif (CFD)

Le CFD (Cumulative Flow Diagram) trace le nombre de tâches dans chaque colonne au fil du temps.

```
Tâches
  │    ████████████████  Terminé
  │   ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  En revue
  │  ░░░░░░░░░░░░░░░░░░  En cours
  │ ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒  Backlog
  └──────────────────────────── Temps
```

Ce qu'on cherche :
- **Bandes parallèles et régulières** → flux sain
- **Bande qui s'élargit** → accumulation, goulot d'étranglement
- **Bande qui rétrécit** → la colonne se vide, risque de sous-charge

---

## Kanban vs Scrum

| | **Kanban** | **Scrum** |
|---|---|---|
| **Cadence** | Flux continu | Sprints fixes (1-4 semaines) |
| **Rôles** | Aucun imposé | Product Owner, Scrum Master, Developers |
| **Planification** | En continu (replenishment) | Sprint Planning |
| **Livraison** | Dès qu'une tâche est prête | À la fin du Sprint |
| **Changements** | Acceptés à tout moment | Protégés pendant le Sprint |
| **Métriques clés** | Cycle time, throughput, CFD | Velocity, burndown |
| **Adapté à** | Support, maintenance, flux imprévisible | Produit avec roadmap, équipe stable |

> **Scrumban** : hybride qui combine les itérations de Scrum avec la gestion du flux et les limites WIP de Kanban. Courant dans les équipes qui souhaitent plus de flexibilité que Scrum sans abandonner les Sprints.
