# SSH

SSH (Secure Shell) est un protocole de communication chiffré permettant d'accéder à distance à des machines, de transférer des fichiers et de créer des tunnels réseau sécurisés. Il remplace les protocoles non chiffrés comme Telnet ou FTP.

> **Clés SSH pour GitHub/GitLab** : la génération de clés et l'ajout à `ssh-agent` sont couverts dans la fiche [Git](git.md).

---

## Connexion à un serveur

### Connexion de base

```bash
ssh utilisateur@adresse-ip
ssh utilisateur@mon-serveur.com
ssh -p 2222 utilisateur@mon-serveur.com    # port personnalisé
```

### Connexion avec une clé privée spécifique

```bash
ssh -i ~/.ssh/ma_cle_privee utilisateur@mon-serveur.com
```

### Première connexion — empreinte du serveur

Lors de la première connexion, SSH demande de valider l'empreinte du serveur :

```
The authenticity of host 'mon-serveur.com' can't be established.
ED25519 key fingerprint is SHA256:xxxx...
Are you sure you want to continue connecting? (yes/no)?
```

Répondre `yes` ajoute le serveur dans `~/.ssh/known_hosts`. Si l'empreinte change (réinstallation serveur), supprimer l'ancienne entrée :

```bash
ssh-keygen -R mon-serveur.com
```

### Copier sa clé publique sur le serveur

```bash
ssh-copy-id utilisateur@mon-serveur.com
ssh-copy-id -i ~/.ssh/ma_cle.pub utilisateur@mon-serveur.com
```

Ajoute la clé dans `~/.ssh/authorized_keys` sur le serveur. Ensuite, la connexion se fait sans mot de passe.

---

## Gestion des clés et ~/.ssh/config

### Générer une paire de clés

```bash
ssh-keygen -t ed25519 -C "commentaire@exemple.com"   # recommandé
ssh-keygen -t rsa -b 4096 -C "commentaire@exemple.com"  # compatibilité anciens systèmes

# Les clés sont créées dans ~/.ssh/
# id_ed25519      → clé privée (ne jamais partager)
# id_ed25519.pub  → clé publique (à copier sur les serveurs)
```

> Nommer les clés par usage plutôt que laisser le nom par défaut : `serveur_prod`, `vps_ovh`, `github_perso`…

### Le fichier ~/.ssh/config

Ce fichier permet de définir des alias et des paramètres par hôte, évitant de retaper les options à chaque fois.

```
# ~/.ssh/config

# Serveur de production
Host prod
    HostName 192.168.1.100
    User deploy
    IdentityFile ~/.ssh/serveur_prod
    Port 22

# VPS avec port personnalisé
Host monvps
    HostName mon-vps.com
    User ubuntu
    IdentityFile ~/.ssh/vps_ovh
    Port 2222

# Paramètres globaux
Host *
    ServerAliveInterval 60      # envoie un ping toutes les 60s pour maintenir la connexion
    ServerAliveCountMax 3
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_ed25519
```

Avec ce fichier, `ssh prod` remplace `ssh -i ~/.ssh/serveur_prod deploy@192.168.1.100`.

### Permissions requises

SSH est strict sur les permissions des fichiers de clés :

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519         # clé privée
chmod 644 ~/.ssh/id_ed25519.pub     # clé publique
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/authorized_keys
```

---

## Transfert de fichiers

### scp — copie simple

```bash
# Local → Serveur
scp fichier.txt utilisateur@serveur:/chemin/distant/
scp -r dossier/ utilisateur@serveur:/chemin/distant/

# Serveur → Local
scp utilisateur@serveur:/chemin/distant/fichier.txt ./
scp -r utilisateur@serveur:/chemin/distant/dossier/ ./

# Entre deux serveurs distants
scp utilisateur@serveur1:/fichier utilisateur@serveur2:/destination/

# Avec port personnalisé (attention : -P majuscule pour scp)
scp -P 2222 fichier.txt utilisateur@serveur:/destination/
```

### rsync — synchronisation intelligente

`rsync` ne transfère que les fichiers modifiés, bien plus efficace que `scp` pour les gros dossiers.

```bash
# Synchroniser un dossier local vers le serveur
rsync -avz dossier/ utilisateur@serveur:/chemin/distant/

# Avec suppression des fichiers supprimés en local
rsync -avz --delete dossier/ utilisateur@serveur:/chemin/distant/

# Avec port SSH personnalisé
rsync -avz -e "ssh -p 2222" dossier/ utilisateur@serveur:/chemin/distant/

# Simulation (dry run) — voir ce qui serait transféré sans l'appliquer
rsync -avz --dry-run dossier/ utilisateur@serveur:/chemin/distant/

# Options courantes
# -a : archive (récursif + conserve permissions, timestamps, liens)
# -v : verbose
# -z : compression pendant le transfert
# --progress : afficher la progression
```

---

## Tunnels et port forwarding

Les tunnels SSH permettent de faire transiter du trafic chiffré via une connexion SSH.

### Local port forwarding (-L)

Redirige un port local vers un port distant via le serveur SSH. Utile pour accéder à un service interne non exposé publiquement.

```bash
ssh -L port_local:hôte_cible:port_cible utilisateur@serveur

# Exemples
ssh -L 8080:localhost:80 utilisateur@serveur      # accéder au port 80 du serveur via localhost:8080
ssh -L 5432:localhost:5432 utilisateur@serveur    # accéder à PostgreSQL du serveur en local
ssh -L 8080:serveur-interne:80 utilisateur@bastion  # accéder à un serveur interne via un bastion
```

Après cette commande, ouvrir `http://localhost:8080` dans le navigateur accède au service distant.

### Remote port forwarding (-R)

Expose un port local sur le serveur distant. Utile pour rendre accessible depuis internet un service tournant en local.

```bash
ssh -R port_distant:localhost:port_local utilisateur@serveur

# Exemple : exposer son serveur local (port 3000) sur le serveur distant (port 9000)
ssh -R 9000:localhost:3000 utilisateur@serveur
# → http://serveur:9000 accède à localhost:3000
```

### Dynamic port forwarding / proxy SOCKS (-D)

Crée un proxy SOCKS5 local qui fait transiter tout le trafic via le serveur SSH.

```bash
ssh -D 1080 utilisateur@serveur
# Configurer ensuite le navigateur pour utiliser SOCKS5 sur localhost:1080
```

### Tunnel en arrière-plan

```bash
ssh -f -N -L 5432:localhost:5432 utilisateur@serveur
# -f : passe en arrière-plan
# -N : ne pas exécuter de commande (tunnel uniquement)
```

### Jump host (-J)

Se connecter à un serveur cible via un serveur intermédiaire (bastion) :

```bash
ssh -J utilisateur@bastion utilisateur@serveur-cible

# Dans ~/.ssh/config
Host serveur-cible
    HostName 10.0.0.50
    User deploy
    ProxyJump bastion
```

---

## Sécuriser son serveur (sshd_config)

Le fichier de configuration du démon SSH se trouve dans `/etc/ssh/sshd_config`.

### Paramètres essentiels

```bash
# /etc/ssh/sshd_config

Port 2222                       # changer le port par défaut (évite les scans automatiques)
PermitRootLogin no              # interdire la connexion directe en root
PasswordAuthentication no       # n'autoriser que les clés SSH (désactiver les mots de passe)
PubkeyAuthentication yes        # autoriser l'authentification par clé
AuthorizedKeysFile .ssh/authorized_keys

MaxAuthTries 3                  # limiter les tentatives d'authentification
LoginGraceTime 30               # délai maximum pour s'authentifier (secondes)

X11Forwarding no                # désactiver le forwarding X11 si inutilisé
AllowTcpForwarding yes          # nécessaire pour les tunnels (mettre no si non utilisé)

# Restreindre les utilisateurs autorisés (optionnel mais recommandé)
AllowUsers deploy ubuntu
```

### Appliquer les changements

```bash
# Vérifier la syntaxe avant de redémarrer (évite de se bloquer hors du serveur)
sshd -t

# Redémarrer le service
sudo systemctl restart sshd
sudo systemctl status sshd
```

⚠️ **Avant de fermer la session en cours**, ouvrir une **nouvelle** connexion pour vérifier que les changements fonctionnent. Si quelque chose ne va pas, la session existante permet de corriger.

### fail2ban — bloquer les attaques par force brute

```bash
# Installation
sudo apt install fail2ban

# Configuration de base pour SSH
# /etc/fail2ban/jail.local
[sshd]
enabled  = true
port     = 2222          # adapter si port personnalisé
maxretry = 5             # nombre de tentatives avant bannissement
bantime  = 3600          # durée du bannissement en secondes (1h)
findtime = 600           # fenêtre de temps pour compter les tentatives

# Démarrer le service
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Vérifier les IPs bannies
sudo fail2ban-client status sshd
```
