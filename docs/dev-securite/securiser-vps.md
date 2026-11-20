# Sécuriser un VPS Linux

Checklist des vérifications à faire avant de mettre un VPS en production, avec les commandes associées. Les exemples sont donnés pour **Debian / Ubuntu**.

> Les robots ne se fatiguent jamais : un VPS fraîchement exposé reçoit des tentatives de connexion SSH dans les minutes qui suivent sa mise en ligne.

---

## 1) Mettre à jour le serveur

### Mise à jour manuelle

```bash
sudo apt update                 # rafraîchir la liste des paquets
sudo apt full-upgrade -y        # installer les mises à jour (y compris changements de dépendances)
sudo apt autoremove --purge -y  # supprimer les paquets devenus inutiles

# Un redémarrage est-il nécessaire (nouveau noyau, libc...) ?
[ -f /var/run/reboot-required ] && cat /var/run/reboot-required.pkgs
sudo reboot
```

### Automatiser les mises à jour de sécurité

```bash
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure -plow unattended-upgrades   # répondre "Oui"
```

Cela crée `/etc/apt/apt.conf.d/20auto-upgrades` :

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

Options utiles dans `/etc/apt/apt.conf.d/50unattended-upgrades` :

```
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "true";          // redémarre si nécessaire...
Unattended-Upgrade::Automatic-Reboot-Time "04:00";    // ...à une heure creuse
Unattended-Upgrade::Mail "admin@exemple.com";         // nécessite un MTA configuré
```

```bash
# Tester la configuration sans rien installer
sudo unattended-upgrade --dry-run --debug

# Consulter l'historique
cat /var/log/unattended-upgrades/unattended-upgrades.log
```

---

## 2) Sécuriser SSH

> Le détail de la configuration SSH (clés, `~/.ssh/config`, tunnels) est dans la fiche [SSH](../dev-ops/ssh.md).

### Créer un utilisateur d'administration

Ne pas travailler en `root` au quotidien :

```bash
# Sur le serveur (en root)
adduser admin
usermod -aG sudo admin
```

### Se connecter par clé

```bash
# Sur le poste local
ssh-keygen -t ed25519 -f ~/.ssh/vps_prod -C "vps-prod"
ssh-copy-id -i ~/.ssh/vps_prod.pub admin@mon-vps.com

# Vérifier que la connexion par clé fonctionne AVANT de désactiver les mots de passe
ssh -i ~/.ssh/vps_prod admin@mon-vps.com
```

### Durcir la configuration du démon

Plutôt que de modifier `/etc/ssh/sshd_config`, créer un fichier dédié (lu en priorité grâce à la directive `Include` présente par défaut) :

```bash
sudo nano /etc/ssh/sshd_config.d/99-hardening.conf
```

```
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
AllowUsers admin
```

```bash
sudo sshd -t                     # vérifier la syntaxe (aucune sortie = OK)
sudo systemctl restart ssh       # "ssh" sur Debian/Ubuntu, "sshd" sur RHEL/Fedora
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication|allowusers'  # config effective
```

⚠️ **Garder la session actuelle ouverte** et tester depuis un **nouveau** terminal avant de se déconnecter.

> **Changer le port SSH** (ex. `Port 2222`) réduit le bruit dans les logs mais n'est pas une sécurité en soi. Sur Ubuntu 24.04+ (activation par socket), il faut ensuite `sudo systemctl daemon-reload && sudo systemctl restart ssh.socket`. Penser à ouvrir le nouveau port dans le pare-feu **avant** de redémarrer.

---

## 3) Configurer le pare-feu (UFW)

Principe : tout bloquer en entrée, n'ouvrir que le nécessaire.

```bash
sudo apt install ufw

sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH d'abord, sinon on se coupe l'accès
sudo ufw allow OpenSSH                 # ou : sudo ufw allow 2222/tcp si port personnalisé
sudo ufw limit OpenSSH                 # limite le débit de connexions (anti brute-force basique)

# Web
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

sudo ufw enable
sudo ufw status verbose
```

Commandes utiles :

```bash
sudo ufw status numbered               # lister les règles avec leur numéro
sudo ufw delete 3                      # supprimer la règle n°3
sudo ufw allow from 203.0.113.10 to any port 22 proto tcp   # SSH réservé à une IP fixe
```

> ⚠️ **Docker contourne UFW** : un conteneur lancé avec `-p 5432:5432` est exposé sur Internet même si UFW bloque le port. Publier les ports sur la boucle locale (`-p 127.0.0.1:5432:5432`) ou ne pas les publier du tout et passer par un reverse proxy. Voir la fiche [Docker](../dev-ops/docker.md).

> Penser aussi au pare-feu de l'hébergeur (OVH, Scaleway, Hetzner...) qui offre une seconde barrière en amont du VPS.

---

## 4) Restreindre l'exposition Internet

### Lister les ports en écoute

```bash
sudo ss -tulpn
# -t TCP, -u UDP, -l en écoute, -p processus, -n sans résolution de noms
```

À lire dans la colonne `Local Address` :

| Adresse | Signification |
|---|---|
| `127.0.0.1:5432` | accessible uniquement depuis le serveur (OK) |
| `0.0.0.0:5432` ou `*:5432` | accessible depuis toutes les interfaces (à vérifier) |
| `[::]:5432` | idem en IPv6 |

### Vérifier depuis l'extérieur

Depuis le **poste local** (pas depuis le serveur) :

```bash
nmap -Pn mon-vps.com              # 1000 ports les plus courants
nmap -Pn -p- mon-vps.com          # tous les ports (plus long)
```

Idéalement, seuls `22` (ou le port SSH choisi), `80` et `443` apparaissent `open`.

### Rediriger HTTP vers HTTPS

Le port 80 ne sert qu'à rediriger vers 443 (et au challenge Let's Encrypt) :

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name mon-site.fr;
    return 301 https://$host$request_uri;
}
```

> Configuration complète Nginx + Certbot : voir la fiche [Nginx SSL](../dev-ops/nginx-ssl.md).

### Désactiver les services inutiles

```bash
systemctl list-units --type=service --state=running
sudo systemctl disable --now nom-du-service
```

---

## 5) Séparer les privilèges

### Créer un utilisateur système dédié à l'application

```bash
sudo useradd --system --create-home --home-dir /opt/monapp --shell /usr/sbin/nologin monapp
sudo chown -R monapp:monapp /opt/monapp
sudo chmod 750 /opt/monapp
```

L'utilisateur n'a ni mot de passe, ni shell, ni droits `sudo`.

### Lancer l'application avec ce compte (systemd)

```ini
# /etc/systemd/system/monapp.service
[Unit]
Description=Mon application
After=network.target

[Service]
User=monapp
Group=monapp
WorkingDirectory=/opt/monapp
ExecStart=/usr/bin/node /opt/monapp/server.js
EnvironmentFile=/etc/monapp/monapp.env
Restart=on-failure

# Durcissement
NoNewPrivileges=true
ProtectSystem=strict
ProtectHome=true
PrivateTmp=true
ReadWritePaths=/opt/monapp/data

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now monapp
ps -o user,pid,cmd -C node              # vérifier que le processus ne tourne pas en root
systemd-analyze security monapp         # score d'exposition du service
```

> L'application écoute sur un port > 1024 (ex. `127.0.0.1:3000`) et Nginx fait reverse proxy : pas besoin de `root` pour se lier au port 443.

### Ne pas exposer la base de données

```bash
# PostgreSQL : /etc/postgresql/<version>/main/postgresql.conf
listen_addresses = 'localhost'

# MySQL / MariaDB : /etc/mysql/mariadb.conf.d/50-server.cnf
bind-address = 127.0.0.1

# Redis : /etc/redis/redis.conf
bind 127.0.0.1 ::1
protected-mode yes
```

```bash
sudo systemctl restart postgresql       # ou mariadb / redis-server
sudo ss -tlpn | grep -E '5432|3306|6379'   # doit afficher 127.0.0.1
```

Créer un utilisateur de base de données par application, avec uniquement les droits sur sa propre base :

```sql
-- PostgreSQL
CREATE ROLE monapp LOGIN PASSWORD 'mot-de-passe-fort';
CREATE DATABASE monapp OWNER monapp;
```

Pour administrer la base à distance, passer par un tunnel SSH plutôt que d'ouvrir le port :

```bash
ssh -L 5432:localhost:5432 admin@mon-vps.com
```

### Auditer les droits

```bash
getent group sudo                       # qui est sudoer ?
sudo -l -U admin                        # droits sudo d'un utilisateur
awk -F: '$3 == 0 {print $1}' /etc/passwd   # comptes avec UID 0 (seul root attendu)
sudo find / -xdev -perm -0002 -type f 2>/dev/null   # fichiers modifiables par tout le monde
```

---

## 6) Bloquer les tentatives d'intrusion (Fail2ban)

Fail2ban surveille les logs et bannit temporairement (via le pare-feu) les IP qui échouent trop souvent.

```bash
sudo apt install fail2ban
```

Ne jamais modifier `jail.conf` (écrasé lors des mises à jour) : créer `/etc/fail2ban/jail.local` :

```ini
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5
bantime.increment = true        # durée de bannissement croissante pour les récidivistes
ignoreip = 127.0.0.1/8 ::1      # ajouter son IP fixe si on en a une
banaction = ufw

[sshd]
enabled = true
port    = ssh                   # ou 2222 si port personnalisé
backend = systemd               # Debian 12+ / Ubuntu 24.04+ : pas de /var/log/auth.log par défaut
```

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status              # jails actives
sudo fail2ban-client status sshd         # IP bannies et compteurs
sudo fail2ban-client set sshd unbanip 203.0.113.10   # débannir une IP
sudo tail -f /var/log/fail2ban.log
```

> Des jails existent aussi pour Nginx (`nginx-http-auth`, `nginx-botsearch`...) : voir `ls /etc/fail2ban/filter.d/`.

---

## 7) Mettre en place des sauvegardes

Règle **3-2-1** : 3 copies, sur 2 supports différents, dont 1 hors du VPS. Un snapshot chez l'hébergeur est un plus, pas une sauvegarde suffisante (même fournisseur, même compte).

### Sauvegarder la base de données

```bash
# PostgreSQL
sudo -u postgres pg_dump -Fc monapp > /var/backups/monapp_$(date +%F).dump

# MySQL / MariaDB
sudo mysqldump --single-transaction monapp | gzip > /var/backups/monapp_$(date +%F).sql.gz
```

### Envoyer les sauvegardes ailleurs avec restic

`restic` chiffre, déduplique et gère la rétention. Il supporte SFTP, S3, Backblaze B2...

```bash
sudo apt install restic

# Initialiser un dépôt distant (ici via SFTP)
export RESTIC_REPOSITORY="sftp:backup@serveur-backup.com:/srv/restic/vps-prod"
export RESTIC_PASSWORD_FILE=/root/.restic-password      # chmod 600
restic init

# Sauvegarder
restic backup /opt/monapp /etc /var/backups

# Rétention : 7 jours, 4 semaines, 6 mois
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune

restic snapshots                         # lister les sauvegardes
restic check                             # vérifier l'intégrité du dépôt
```

### Planifier

```bash
sudo crontab -e
```

```
# Tous les jours à 3h
0 3 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

### Tester la restauration

> Une sauvegarde qui n'a jamais été restaurée n'est pas une garantie de récupération.

```bash
restic restore latest --target /tmp/restore-test
ls -la /tmp/restore-test/opt/monapp

# Restaurer la base dans une base de test
sudo -u postgres createdb monapp_test
sudo -u postgres pg_restore -d monapp_test /tmp/restore-test/var/backups/monapp_2026-11-20.dump
```

Planifier ce test régulièrement (par exemple une fois par mois).

---

## 8) Configurer des alertes et la supervision

### Commandes de vérification rapide

```bash
df -h                                    # espace disque
free -h                                  # mémoire
uptime                                   # charge
systemctl --failed                       # services en échec
journalctl -p err -b                     # erreurs depuis le dernier démarrage
journalctl -u ssh --since "1 hour ago"   # logs d'un service
last -n 20                               # dernières connexions réussies
sudo lastb -n 20                         # dernières tentatives échouées
```

### Alerte simple sur l'espace disque

```bash
#!/bin/bash
# /usr/local/bin/check-disk.sh
THRESHOLD=85
USAGE=$(df / --output=pcent | tail -1 | tr -dc '0-9')
if [ "$USAGE" -ge "$THRESHOLD" ]; then
    echo "Disque / plein a ${USAGE}% sur $(hostname)" | mail -s "[ALERTE] Disque" admin@exemple.com
fi
```

```
# crontab : toutes les heures
0 * * * * /usr/local/bin/check-disk.sh
```

### Supervision externe

La surveillance doit tourner **hors du VPS**, sinon elle tombe avec lui :

- **Disponibilité** : [Uptime Kuma](https://github.com/louislam/uptime-kuma) (auto-hébergé ailleurs), UptimeRobot... interrogent le site à intervalles réguliers.
- **Tâches planifiées** : [healthchecks.io](https://healthchecks.io) alerte si un cron ne s'est pas exécuté (utile pour les sauvegardes) :

```bash
# À la fin du script de sauvegarde
restic backup /opt/monapp /etc /var/backups && curl -fsS -m 10 --retry 3 https://hc-ping.com/<uuid>
```

- **Métriques** : Netdata, ou Prometheus + Grafana pour une infrastructure plus large.

---

## Compléments

### Dépendances applicatives

Le système à jour ne protège pas d'une faille dans le code ou ses dépendances :

```bash
npm audit                     # Node.js
pip-audit                     # Python (pip install pip-audit)
composer audit                # PHP
```

Activer Dependabot ou Renovate sur le dépôt pour être notifié automatiquement.

### Secrets

Jamais de clé API ou de mot de passe dans le code source ni dans Git (voir la fiche [Gestion des secrets](secrets.md)) :

```bash
sudo mkdir -p /etc/monapp
sudo nano /etc/monapp/monapp.env         # DATABASE_URL=..., API_KEY=...
sudo chown root:monapp /etc/monapp/monapp.env
sudo chmod 640 /etc/monapp/monapp.env
```

Le fichier est chargé par systemd via `EnvironmentFile=` (voir section 5). Ajouter `.env` au `.gitignore` côté projet.

### Logs

```bash
sudo journalctl -f                       # suivre tous les logs en direct
sudo tail -f /var/log/nginx/error.log
sudo apt install logwatch                # rapport quotidien par mail des événements système
```

---

## Checklist finale

| # | Vérification | Commande de contrôle |
|---|---|---|
| 1 | Système à jour | `apt list --upgradable` |
| 2 | Mises à jour de sécurité automatiques | `systemctl status unattended-upgrades` |
| 3 | Connexion SSH par clé | `ssh -i ~/.ssh/vps_prod admin@mon-vps.com` |
| 4 | Mot de passe SSH désactivé | `sudo sshd -T \| grep passwordauthentication` |
| 5 | Connexion root SSH interdite | `sudo sshd -T \| grep permitrootlogin` |
| 6 | Pare-feu actif | `sudo ufw status verbose` |
| 7 | Seuls SSH, 80 et 443 exposés | `nmap -Pn mon-vps.com` (depuis l'extérieur) |
| 8 | Application lancée avec un utilisateur dédié | `ps -o user,cmd -C node` |
| 9 | Base de données non exposée | `sudo ss -tlpn` (écoute sur 127.0.0.1) |
| 10 | Fail2ban actif | `sudo fail2ban-client status sshd` |
| 11 | Sauvegardes externalisées | `restic snapshots` |
| 12 | Restauration testée | `restic restore latest --target /tmp/restore-test` |
| 13 | Alertes et supervision configurées | ping healthchecks.io / Uptime Kuma |

> **En résumé** : sécuriser un VPS, ce n'est pas seulement installer un pare-feu. C'est appliquer le moindre privilège, limiter l'exposition, prévoir la récupération après incident et surveiller le serveur en continu.
