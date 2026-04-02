# Git

Git est un système de contrôle de version distribué, permettant de suivre les modifications apportées à un ensemble de fichiers (un "dépôt"). Il offre un moyen efficace de collaborer sur des projets en équipe et de maintenir un historique complet des modifications.

Cette fiche recense les principales commandes Git. Il s'agit d'une liste non exhaustive des fonctionnalités de cet outil.

L'intégralité du livre Pro Git, écrit par Scott Chacon et Ben Straub est disponible à cette adresse [https://git-scm.com/book/fr/v2](https://git-scm.com/book/fr/v2)

## Commandes principales

### Initialisation d'un nouveau dépôt :

```Shell
git init
```

Initialise un nouveau dépôt Git dans le répertoire courant.

### Paramétrer ses informations utilisateur (`name,` `email`) :

```Bash
git config --global user.name "NouveauNom"

# au niveau du dépôt seulement :
git config user.name "NouveauNom"
```

### Clonage d'un dépôt existant à l’intérieur d’un nouveau dossier :

```Shell
git clone <url>
```

### Une fois dans le dossier : ajout de fichiers au suivi de Git :

```Shell
git add <nom_fichier>
```

### Ajoute tous les fichiers modifiés à partir du dossier courant au suivi de Git :

```Shell
git add .
```

### Voir les fichiers du suivi de Git prêts à être validés :

```Bash
git status
```

(Affiche l'état actuel du dépôt, montrant les fichiers modifiés, en attente de commit, etc.)

### Valider les modifications :

```Shell
git commit -m "message du commit"
```

### Voir l'historique des commits :

```Shell
git log
```

Affiche l'historique des commits effectués, avec les informations de chaque commit.

### Revenir à une version précédente :

> Attention : faire cela passe en mode "tête détachée", pour revenir à l'état initial faire `git checkout <nom-branche>`

```Shell
git checkout <identifiant_commit>
```

### Pousser les mises à jour de toutes les branches du dépôt :

```Bash
git push origin --all
```

## Les branches

Une branche est essentiellement un pointeur mobile vers une sauvegarde spécifique. Elle permet de travailler sur différentes versions du projet en parallèle.

### Voir les branches :

(`-a` : toutes les branches du dépôt, distantes et locales)

```Bash
git branch -a
```

### Crée une nouvelle branche :

```Shell
git branch <nom_branche>
```

### Changer le nom de la branche en cours :

```Bash
git branch -m <new_branch_name>

# renommer dans le dépôt distant :
git push origin HEAD
```

### Vérifier le nom de la branche distante associée à votre branche locale :

```Bash
git branch -vv
```

### Supprimer une branche locale :

```Shell
git branch -d <nom_branche>
```

### Supprimer la branche dans le dépôt (⚠️ irréversible) :

```Bash
git push origin --delete <nom-de-la-branche>
```

### Changer de branche :

```Shell
git checkout <branche>
```

Permet de passer à une branche spécifique.

### Fusionner individuellement des fichiers d’une autre branche :

```Bash
git checkout --patch <branche_source> .\path\filename.ext
```

### Fusionner des branches :

⚠️ Avant de fusionner il est utile de faire un git status pour s’assurer être bien à jour sur sa branche actuelle en vue d’accueillir la fusion.  

```Shell
git merge <branche_source>
```

Fusionne la `branche_source` dans la branche actuelle.

## Les connexions aux dépôts distants (remotes ou pairs)

Une remote (pair) est une référence à un autre dépôt Git situé sur un serveur distant. Elle vous permet de connecter votre dépôt local à un dépôt distant, comme GitHub ou GitLab.

### Afficher les adresses des dépôts distants :

```Shell
git remote -v
```

### Ajouter une adresse de dépôt distant :

```Bash
git remote add <remote_name> <url>
```

### Activer le suivi du dépôt distant sur la branche (tracking) :

```Bash
git branch --set-upstream-to=origin/main main
```

### Supprimer une adresse de dépôt distant :

```Shell
git remote rm <remote>
```

### Changer l’adresse d’origine du dépôt :

```Shell
git remote set-url origin https://github.com/votre-nom/votre-fork
```

### Renommer un pair :

```Bash
git remote rename <ancien-nom> <nouveau-nom>
```

### Ajouter d’autres pairs :

```Shell
git remote add my-upstream https://github.com/votre-nom/votre-fork
```

## Pull & Push

### Récupérer les mises à jour depuis un dépôt distant :

```Shell
git pull <remote> <branche>

# ou

git pull <remote> <branche> --allow-unrelated-histories
```

### Pousser des modifications vers un dépôt distant :

```Shell
git push <remote> <branche>
```

Pousse les commits locaux vers le dépôt distant dans la branche spécifiée.

## Les worktrees

Un worktree Git est un répertoire de travail distinct qui pointe vers un dépôt Git. Il vous permet d'avoir plusieurs environnements de travail basés sur le même dépôt, chacun avec son propre historique de modifications et son propre état de travail.

### Créer un arbre de travail (clone) avec sa nouvelle branche :

```Bash
git worktree add -b <new-branch> ../<folder-name>
```

### Renommer le nom de dossier d’un worktree :

```Bash
git worktree move ../<folder-name> ../<new-folder-name>
```

### Lister les worktrees :

```Bash
git worktree list
```

### Supprimer un worktree :

```Bash
git worktree remove <folder-name>
```

## Conflits, tag, stash et +

### Résoudre les conflits de fusion :

Si une fusion provoque des conflits, éditez les fichiers pour résoudre les conflits, puis effectuez un commit pour finaliser la fusion.

⚠️ Attention, en cas de fusion Git peut ouvrir l'éditeur de texte Vi pour éditer votre commit. Connaissez les raccourcis.

### Revenir à l’état du dernier commit :

```Bash
# Déposer tous les éléments du stage :
git reset
# Restaurer à l'état du dernier commit :
git restore .
```

### Supprimer un fichier du dépôt :

```Shell
# en le gardant en local:
git rm --cached <file>

# complètement (distant et local);
git rm <file>
```

### Récupérer la version d’un fichier d’une autre branche :

```Bash
git checkout <branche_source> -- <chemin/vers/le/fichier>
```

### Créer des étiquettes (tags) :

```Shell
git tag -a <nom_etiquette> -m "message de l'étiquette"
```

Crée une étiquette annotée pour marquer un point spécifique dans l'historique.

### Réécrire l'historique :

```Shell
git rebase <branche>
```

Réécrit l'historique en basant la branche actuelle sur la branche spécifiée.

### Mettre de côté les modifications :

```Shell
git stash \#mettre de coté

git stash pop \#réappliquer stash
```

Met de côté temporairement les modifications non validées pour les sauvegarder et les récupérer ultérieurement.

```Shell
git stash list

git stash drop stash@{0}

git stash clear
```

### Voir les différences :

```Shell
git diff <identifiant_commit> <chemin_du_fichier>
```  

## Connexion par clés SSH

### Crée une clé / Ajoute la clé à l’agent du système

_(NB : Les clés SSH nécessitent un service SSH actif sur votre système.)_

```Bash
Le répertoire par défaut des clés est `~/.ssh/`, s'y positionner : `cd ~./ssh/`

# Lister les clés actives :
ssh-add -l

# Désactiver une clé :
ssh-add -d <chemin-de-la-clé>

# Créer une clé :
ssh-keygen
# (la passphrase est optionnelle)

# Ajouter la clé ssh au système :
ssh-add ~/.ssh/id_rsa

# Tester la connexion au service
ssh -T git@github.com
```

Le fichier sans extension (ex. `github_rsa`) est votre clé privée. elle ne doit pas être partagée.

Rajouter/coller ensuite le contenu de votre fichier clé publique `github_rsa.pub` dans les settings du service (**Pour GitHub :** clic icone profil, onglet “SSH and GPG Keys”)

**NB:** Préférez des clés uniques (ex. : `gitlab_rsa`, `workpc_rsa`, `nas_rsa`, etc.)

## Rédaction des commit, préfixes

Préfixes les plus courants : `feat:` , ou `fix:`

autres : `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `perf:`, `test:`, …

### Exemple d’un message complet :

```Markup
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: \#123
```

### Exemple multiligne en cli :
```Shell
git commit -m "Initial Commit

Create new files with npm init
Add .gitignore"
```

### Conventions de nommage :

**Commits Conventionnels** : Une spécification ajoutant une signification lisible pour l'humain et pour la machine dans les messages des commits

[https://www.conventionalcommits.org/fr/v1.0.0/](https://www.conventionalcommits.org/fr/v1.0.0/)

## Mémo commandes utiles (cheat sheet)

### Sortir les éléments ajoutés du plateau, puis revenir à l’état du dernier commit :

```Shell
git reset
git restore .
```

### Faire un nettoyage :

(Réapplique le `.gitignore`, supprime les fichiers non suivis, …)

```Bash
git clean -dfx
```

### Annuler la dernière sauvegarde :

- Tout en conservant les modifications dans le répertoire de travail :

```Bash
git reset --soft HEAD^
```

-  En supprimant toutes les modifications associées :

```Bash
git reset --hard HEAD^
```

(Faire ensuite un nettoyage avec `git clean -df` pour bien resynchroniser votre dossier)

## Liens et +

### Guide contributions sur Github [FR] :

[https://github.com/firstcontributions/first-contributions/blob/main/translations/README.fr.md](https://github.com/firstcontributions/first-contributions/blob/main/translations/README.fr.md)

### Articles [dev.to](http://dev.to) sur GIT [EN] :

[https://dev.to/ello/series/23990](https://dev.to/ello/series/23990)