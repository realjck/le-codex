# Système de fichiers Linux

## Introduction à la structure des répertoires sous Linux

La plupart des utilisateurs de Linux ont déjà vu le répertoire racine (`/`), mais tous ne comprennent pas nécessairement l'utilité des différents répertoires qu'il contient. Pour un utilisateur venant de Windows, l'exploration du gestionnaire de fichiers sous Linux peut sembler familière au premier abord : on y retrouve des dossiers comme "Documents", "Téléchargements", "Images", et "Vidéos". Cependant, cette familiarité s'estompe rapidement lorsqu'on cherche des répertoires comme "Program Files" ou le lecteur C, omniprésents sous Windows.

Pour comprendre pourquoi Linux est structuré différemment, il est utile de revenir sur l'histoire des systèmes d'exploitation.

- **MS-DOS** : Au commencement, il y avait MS-DOS, un système d'exploitation en ligne de commande. On pouvait y exécuter des programmes, des jeux, et des logiciels comme WordPerfect. MS-DOS utilisait des lettres pour désigner les lecteurs : A et B pour les disquettes, et C pour le disque dur principal.
    
- **Windows** : Windows est venu se greffer par-dessus MS-DOS. Pour démarrer Windows, il fallait taper `win` dans l'interface en ligne de commande de MS-DOS. Au fil du temps, Microsoft a rendu Windows de plus en plus indépendant de MS-DOS, jusqu'à ce qu'il puisse démarrer sans lui.


Linux, quant à lui, suit les traditions UNIX, ce qui explique plusieurs différences fondamentales avec Windows :

- **Utilisation du slash (`/`)** : Contrairement à Windows qui utilise le backslash (`\`), Linux utilise le slash pour séparer les répertoires.

- **Sensibilité à la casse** : Linux est sensible à la casse, ce qui signifie que "file", "File", et "FILE" sont considérés comme des fichiers différents.

- **Pas de répertoire "Program Files"** : Sous Linux, les applications ne sont pas toutes installées dans un répertoire centralisé comme "Program Files". Elles sont réparties dans divers répertoires en fonction de leur nature et de leur utilisation.

## Les principaux répertoires sous Linux

La structure des répertoires sous Linux est définie par le Filesystem Hierarchy Standard (FHS), bien que toutes les distributions ne suivent pas ce standard à la lettre. Voici une description détaillée des principaux répertoires :

- **`/bin`** : Contient les binaires essentiels au fonctionnement du système, accessibles à tous les utilisateurs. On y trouve des commandes de base comme `ls`, `cat`, `cp`, et `mv`.

- **`/sbin`** : Contient les binaires système, utilisés principalement par l'administrateur pour des tâches de maintenance et de réparation. Par exemple, `ifconfig` ou `reboot`.

- **`/boot`** : Contient les fichiers nécessaires au démarrage du système, comme les chargeurs de démarrage (GRUB) et le noyau Linux.

- **`/dev`** : Contient les fichiers de périphériques. Sous Linux, tout est considéré comme un fichier, y compris les périphériques matériels. Par exemple, `/dev/sda` représente le premier disque dur.

- **`/etc`** : Contient les fichiers de configuration système. Par exemple, les fichiers de configuration des services réseau, les listes de sources pour les gestionnaires de paquets, etc.

- **`/home`** : Répertoire personnel des utilisateurs, où chaque utilisateur dispose de son propre sous-répertoire pour stocker ses fichiers et paramètres personnels.

- **`/lib`, `/lib32`, `/lib64`** : Contiennent les bibliothèques partagées nécessaires aux binaires situés dans `/bin` et `/sbin`.

- **`/media` et `/mnt`** : Points de montage pour les périphériques amovibles. `/media` est souvent utilisé pour le montage automatique des périphériques, tandis que `/mnt` est utilisé pour les montages manuels.

- **`/opt`** : Contient des logiciels optionnels, souvent installés manuellement ou provenant de fournisseurs tiers.

- **`/proc`** : Système de fichiers virtuel contenant des informations sur les processus en cours d'exécution et les ressources système. Par exemple, `/proc/cpuinfo` contient des informations sur le processeur.

- **`/root`** : Répertoire personnel de l'utilisateur root, l'administrateur système.

- **`/run`** : Répertoire temporaire en RAM, utilisé pour stocker des informations d'exécution pour les processus démarrés tôt dans le processus de démarrage.

- **`/snap`** : Contient les paquets Snap, un format de paquet auto-suffisant utilisé principalement par Ubuntu.

- **`/srv`** : Contient les données des services, comme les fichiers d'un serveur web ou FTP.

- **`/sys`** : Interface avec le noyau pour des informations sur le matériel et les périphériques. Par exemple, des informations sur les périphériques USB connectés.

- **`/tmp`** : Répertoire temporaire pour les fichiers utilisés pendant une session. Les fichiers dans `/tmp` sont généralement supprimés au redémarrage du système.

- **`/usr`** : Contient les applications et bibliothèques utilisateur. Par exemple, `/usr/bin` contient les binaires des applications utilisateur.

- **`/var`** : Contient des fichiers variables, comme les logs système, les bases de données, et les files d'attente d'impression.


### Conclusion

Bien que la structure des répertoires sous Linux puisse sembler complexe au premier abord, elle permet une gestion efficace des ressources et des applications. Les gestionnaires de paquets, comme `apt` pour Debian/Ubuntu ou `yum` pour Fedora, facilitent l'installation, la mise à jour, et la suppression des logiciels en suivant ces conventions.
