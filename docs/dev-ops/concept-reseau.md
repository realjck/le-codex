# Concepts de réseau essentiels

## 1. Notation CIDR (Classless Inter-Domain Routing)

Cette notation définit une plage d'adresses IP.

| Notation | Signification | Plage d'adresses | Exemple |
| :--- | :--- | :--- | :--- |
| **`/32`** | Une seule adresse IP. (Souvent omis, mais requis dans certains outils/formulaires). | L'adresse commence et se termine à la même adresse. | `1.2.3.4/32` |
| **`/24`** | Toutes les adresses IP avec les trois premiers octets identiques. | Contient 256 adresses. | `10.0.0.0/24` (de `10.0.0.1` à `10.0.0.255`) |
| **`/16`** | Toutes les adresses IP avec les deux premiers octets identiques. | Contient 65 536 adresses. | `10.0.0.0/16` (de `10.0.0.1` à `10.0.255.255`) |
| **`/8`** | Toutes les adresses IP avec le premier octet identique. | Contient 16 millions d'adresses. (Rarement utilisé) | `10.0.0.0/8` (de `10.0.0.1` à `10.255.255.255`) |

**Utilité pour le développeur :** Configurer des firewalls ou des *Security Groups* sur le cloud.

## 2. IPs Publiques vs. IPs Privées

* **IP Privée :** Fonctionne uniquement dans un réseau local (téléphone, ordinateur, tablette connectés à la même box). Elles ne sont pas accessibles depuis Internet.
* **IP Publique :** Toute autre adresse IP qui est accessible depuis Internet.

### Plages d'Adresses IP Privées à Reconnaître

| Plage | Notation CIDR | Utilisé pour |
| :--- | :--- | :--- |
| **`192.168.0.0/16`** | Débute par `192.168.` | Le plus souvent utilisé par les box Internet. |
| **`10.0.0.0/8`** | Débute par `10.` | Souvent utilisé dans les réseaux d'entreprise. |
| **`172.16.0.0/12`** | Débute par `172.16.` jusqu'à `172.31.` | Moins fréquent, mais à reconnaître. |

### Adresses IP Spéciales

| Adresse/Plage | Nom | Utilité |
| :--- | :--- | :--- |
| **`127.0.0.1`** (et tout le `/8`) | **Localhost** | IP interne à la machine, utilisée pour que deux processus sur la même machine communiquent (ex: un serveur web local). |
| **`169.254.0.0/16`** | **APIPA** (Automatic Private IP Addressing) | Adresse que l'ordinateur s'attribue automatiquement s'il ne parvient pas à obtenir d'IP via un routeur (par exemple, si le câble est débranché ou le routeur est défectueux). |
| **`100.64.0.0/10`** | **CGNAT** (Carrier Grade NAT) | Utilisé pour le *tethering* (partage de connexion mobile) ou par certains VPNs. C'est une IP privée. |

## 3. DNS (Domain Name System)

Le DNS est l'annuaire d'Internet. Il permet de traduire un nom de domaine facile à mémoriser (ex: `google.com`) en une adresse IP (ex: `142.250.69.78`) qui est l'adresse réelle de destination.

### Types d'Enregistrements DNS les plus courants

| Type d'Enregistrement | Description | Exemple |
| :--- | :--- | :--- |
| **A** | *Address* : Fait le lien entre un nom de domaine et une adresse **IP V4**. | `serveur.com` -> `1.2.3.4` |
| **AAAA** | *Address* : Fait le lien entre un nom de domaine et une adresse **IP V6**. | |
| **CNAME** | *Canonical Name* : Crée un alias d'un nom de domaine vers un **autre nom de domaine**. | `www.youtube.com` -> `youtube.com` |
| **MX** | *Mail eXchanger* : Indique quel serveur doit recevoir les emails pour le nom de domaine. | |
| **TXT** | *Text* : Utilisé pour des informations diverses (Dkim, Dmarc, SPF pour l'email, preuves de propriété de domaine pour les certificats SSL/TLS). | |

## 4. Protocoles de Transport TCP/UDP/ICMP

Ces protocoles se situent à la **couche 4 (Transport)** du modèle OSI (ou TCP/IP), juste en dessous des protocoles applicatifs (HTTP, SSH, DNS).

| Protocole | Caractéristique principale | Exemples d'utilisation |
| :--- | :--- | :--- |
| **TCP** (Transmission Control Protocol) | **Fiable** (assure la réception de tous les paquets, dans le bon ordre et sans corruption). Plus lourd en *overhead*. | HTTP, HTTPS, SSH, FTP, POP, IMAP. |
| **UDP** (User Datagram Protocol) | **Rapide** (optimisé pour la latence). Moins fiable (les paquets peuvent être perdus ou arriver dans le désordre). Moins d'*overhead*. | VoIP, jeux en ligne (position), DNS (pour les requêtes courtes et rapides). |
| **ICMP** (Internet Control Message Protocol) | **Diagnostic** (ne transporte pas de données applicatives). | Outil `ping`. Permet de tester la connectivité réseau entre deux machines. |

### Ports de Communication

* Les ports vont de 1 à 65535.
* Les **1024 premiers ports** sont généralement réservés (sur Linux, il faut être *root* pour les utiliser) :
    * Port 80 : HTTP
    * Port 22 : SSH
    * Port 21 : FTP
    * Port 53 : DNS

## 5. HTTP / HTTPS / Load Balancer

### Protocole HTTP (Hypertext Transfer Protocol)

Le protocole HTTP est textuel et est utilisé par la plupart des applications web et API.

**Structure d'une requête HTTP manuelle (Telnet sur le port 80) :**
1.  **Verbe :** `GET`, `POST`, `DELETE`, `PUT`, etc.
2.  **Ressource :** `/` (racine) ou `/index.html`.
3.  **Version du Protocole :** `HTTP/1.1` (le plus courant manuellement).
4.  **Header `Host` :** Indique le site web demandé (`Host: www.google.com`). Ceci est crucial car un seul serveur IP peut héberger plusieurs sites web.
5.  **Corps de la réponse :** Le serveur répond avec des *headers* (incluant les codes de réponse) et le corps de la page web.

**Codes de Réponse HTTP :**
* **200 OK :** La requête a réussi.
* **301 Moved Permanently :** Redirection. Le serveur demande au client de faire une nouvelle requête vers l'URL spécifiée.

### HTTPS (Hypertext Transfer Protocol Secure)

* C'est le protocole HTTP encapsulé dans une couche de chiffrement : **TLS** (Transport Layer Security). Le terme **SSL** (ancienne version) est souvent conservé par habitude (ex: certificat SSL).
* **Objectifs :**
    1.  **Chiffrement :** Rendre le trafic illisible aux intermédiaires.
    2.  **Authentification :** Le certificat (émis par une Autorité de Certification) prouve que le serveur est bien le propriétaire du nom de domaine, assurant que vous communiquez avec la bonne entité.

### Load Balancer (Répartiteur de charge)

Aussi appelé *Reverse Proxy* (ou serveur mandataire). Il est le seul point d'accès public à l'infrastructure. Il reçoit les requêtes et les distribue à plusieurs serveurs en arrière-plan (qui sont dans un réseau privé).

| Type de Load Balancer | Niveau (Couche OSI) | Comportement | Avantages | Inconvénients |
| :--- | :--- | :--- | :--- | :--- |
| **Niveau 4 (NLB)** | Couche Transport (TCP) | Inspecte uniquement les informations TCP (IP source/destination, Port). | Très performant, simple. | Moins "intelligent". |
| **Niveau 7 (ALB)** | Couche Applicative (HTTP) | Inspecte le contenu de la requête (Verbe, Ressource, Header `Host`). | Peut gérer le **SSL Offloading** et rediriger les requêtes vers des groupes de serveurs différents selon le nom de domaine (`Host: fanta.com` vs `Host: coca-cola.com`). | Un peu plus lent qu'un NLB (plus de travail d'inspection). |

## 6. Outils de Debug/Diagnostic

| Outil | Protocole(s) ciblé(s) | Usage principal |
| :--- | :--- | :--- |
| **`nslookup` / `dig`** | DNS | Tester la résolution de noms de domaine en adresse IP. |
| **`netstat -tpnl` / `ss`** | TCP/UDP | Lister les connexions réseau actives et les ports en écoute (et par quel programme). |
| **`ping`** | ICMP | Tester la connectivité de bas niveau entre deux machines. |
| **`traceroute` / `tracert` / `mtr`** | ICMP | Afficher le chemin (les différents "sauts" ou routeurs) que prend un paquet pour atteindre une destination. |
| **`telnet` / `nc` (netcat)** | TCP | Client TCP permettant d'établir une connexion manuelle sur un port donné pour tester le protocole applicatif (ex: taper une requête HTTP à la main). |
| **`curl` / `wget`** | HTTP | Client HTTP/HTTPS pour envoyer des requêtes et voir les détails des en-têtes (headers) et des codes de réponse (utiliser l'option `-v` pour *verbose* dans `curl`). |