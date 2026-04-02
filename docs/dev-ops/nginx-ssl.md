# Configuration d'un serveur web Nginx avec SSL

Cette fiche énumère les étapes pour configurer rapidement un serveur web Linux avec Nginx et encryption SSL grâce à Certbot.

## 1) Créer une entrée DNS de Type A pour le sous-domaine

Exemple pour [stuff.devjck.fr](http://stuff.devjck.fr) :
- Type : A
- Nom : stuff
- Valeur : 12.34.56.178
- TTL : 1/2 heure

Exemple pour [pxly.fr](http://pxly.fr) :
- Type : A
- Nom : laisser vide (@)
- Valeur : 12.34.56.178
- TTL : 1/2 heure

## 2) Créer et configurer le dossier web

```Shell
sudo mkdir /var/www/stuff.devjck.fr
# Remplacer <user> par votre compte sftp :
sudo chown -R <user>:<user> /var/www/stuff.devjck.fr
sudo chmod -R 755 /var/www/stuff.devjck.fr
```

## 3) Générer un certificat sur le serveur

Avoir `certbot` d'installé avec le plugin pour Nginx :
```Shell
# Si non installé :
sudo apt install certbot
sudo apt-get install python3-certbot-nginx
```

Générer le certificat :
```Shell
sudo certbot certonly --nginx -d stuff.devjck.fr
```

Choisir l’option Nginx Web Server plugin si vous l’utilisez.

## 4) Configurer Nginx pour pointer vers le dossier web

**Créer un nouveau fichier `.conf` dans `/etc/nginx/sites-available/`:**

```Shell
sudo touch /etc/nginx/sites-available/mon-site.fr.conf
```

Inclure ce fichier dans `/etc/nginx/sites-available/default` :

```shell
include /etc/nginx/sites-available/mon-site.fr.conf;
# ou bien
include /etc/nginx/sites-available/*.conf;
```

**Contenu du fichier:**

**Important:** Avant de configurer le port SSL, il est nécessaire d'avoir la connexion sur le port :80 configurée

```Shell
# pxly.fr
server {
    listen 80;
    listen [::]:80;

    server_name pxly.fr www.pxly.fr;

    root /var/www/pxly.fr;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Rajouter pour le SSL:**

```Shell
# add to the end
# replace servername and path of certificates to your own one

server {
       listen       443 ssl http2 default_server;
       listen       [::]:443 ssl http2 default_server;
       server_name  www.srv.world;
       root         /var/www/html;
       index index.html index.htm index.nginx-debian.html;

       ssl_certificate "/etc/letsencrypt/live/www.srv.world/fullchain.pem";
       ssl_certificate_key "/etc/letsencrypt/live/www.srv.world/privkey.pem";
       ssl_session_cache shared:SSL:1m;
       ssl_session_timeout  10m;

       location / {
              try_files $uri $uri/ =404;
       }
}
```

Possibilité de rajouter une ligne `add_header Cache-Control "no-cache";` dans location / { } si nécessaire de désactiver le cache serveur.

**Tester la configuration Nginx:**

```Shell
sudo nginx -t
```


**Relancer Nginx:**

```Shell
sudo systemctl restart nginx
```

C'est tout ! Votre serveur web est maintenant opérationnel et configuré en SSL.

## Annexe

Avec Certbot, vous pouvez :

### Voir les certificats du serveur :

```Shell
sudo certbot certificates
```

### Effacer un certificat :
```Shell
sudo certbot delete --cert-name example.com
```
