# Docker

Docker est une plateforme de virtualisation légère permettant de créer, déployer et exécuter des applications dans des conteneurs. Un conteneur regroupe tout le nécessaire pour exécuter une application : code, bibliothèques, dépendances et configurations. Contrairement aux machines virtuelles, les conteneurs partagent le noyau du système hôte, ce qui les rend plus légers et plus rapides.

## Images

### Construire une image (via `Dockerfile`)

```Docker
docker build -t <nom-image> .
```

### Voir les images disponibles

```Docker
docker images
```

### Effacer une image

```Docker
docker rmi -f <nom-image>
```

### Lancer une image dans un conteneur

```Docker
# exemple pour une app flask:
docker run -d -p 80:5000 <nom-image>   
```

## Conteneurs

### Voir les conteneurs en cours

```Docker
docker ps
```

### Arrêter un conteneur

```Docker
docker stop <nom-conteneur>
```

### Effacer un conteneur

```Docker
docker rm <nom-conteneur>
```

### Remise à zéro des conteneurs, des images et des volumes

```bash
sudo docker system prune -a --volumes
```

## Créer des conteneurs

```Shell
docker pull nom_du_package:tag(version)
exemple: docker pull nginx:1.22-alpine
```

ou directement :

```Shell
docker run -d -p 8080:80 nginx:1.22-alpine

-d (option) pour "detached"
-p (port)
```

## Dockerfile

Un Dockerfile est un fichier texte qui contient les instructions nécessaires pour construire une image Docker. Il décrit les étapes de création de l'environnement d'exécution d'une application, comme la base d'image à utiliser, les dépendances à installer et les commandes à exécuter. Une fois le Dockerfile rédigé, la commande `docker build` permet de générer l'image correspondante.

"Dockerfile" doit être à la racine du projet,  
- exemple de Dockerfile pour une application node js :  

```Shell
FROM node:19-alpine

COPY package.json /app/
COPY src /app/

WORKDIR /app

RUN npm install

CMD ["node", "server.js"]
```

- Build des images docker :

```Shell
docker build -t my-app:1.0 .

-t : titre:tag
. : path
```

### Voir les conteneurs en cours

```Shell
docker ps --all
```

### Explorer un conteneur

```Bash
docker exec -it <id_ou_nom_du_conteneur> bash
```

### Dockerfile exemple

```Shell
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

### Ensuite, lancer le build

```Shell
> docker build -t myApp .
```

### Exemple de conteneur avec volume de données hors conteneur

```Shell
docker run --name myMongo -v /my/workdir/data:/data/db -d mongo:
```

## Docker compose

Docker Compose est un outil qui facilite la gestion de configurations multi-conteneurs. Grâce à un fichier `docker-compose.yml`, il permet de définir et d'orchestrer plusieurs services Docker, leurs réseaux et volumes associés. La commande `docker-compose up` permet de lancer l'ensemble des services définis, simplifiant ainsi le déploiement d'applications complexes.

(isolation réseau : les conteneurs peuvent se voir entre eux)

Gérer l’application :

```Shell
docker-compose up
docker-compose down
```

Exemple de Docker compose :

```Shell
services:
	db:
		build: aci-demo/db
		images: gtardif/sentences-db
	service1:
		container_name: words
		image: gtardif/sentence-api
		ports:
		- "8080:8080"
	web:
		image: nginx
		volumes:
		- ./static:/usr/share/nginx/html
	ports:
		- "80:80"
```