# Bash

Bash (Bourne Again SHell) est le shell Unix par défaut sur la plupart des systèmes Linux et macOS. Il s'utilise de deux façons complémentaires : en mode interactif pour exécuter des commandes dans le terminal, et en mode script pour automatiser des tâches répétitives.

## Navigation et fichiers

### Se repérer et se déplacer

```bash
pwd                     # affiche le répertoire courant
ls                      # liste les fichiers
ls -la                  # liste détaillée avec fichiers cachés
cd /chemin/vers/dossier # se déplacer
cd ..                   # remonter d'un niveau
cd ~                    # aller dans le répertoire home
cd -                    # revenir au répertoire précédent
```

### Créer, copier, déplacer, supprimer

```bash
mkdir mon-dossier           # créer un dossier
mkdir -p a/b/c              # créer les dossiers intermédiaires si besoin
touch fichier.txt           # créer un fichier vide

cp source.txt dest.txt      # copier un fichier
cp -r dossier/ copie/       # copier un dossier récursivement
mv ancien.txt nouveau.txt   # renommer ou déplacer
rm fichier.txt              # supprimer un fichier
rm -rf dossier/             # ⚠️ supprimer un dossier et son contenu
```

### Lire des fichiers

```bash
cat fichier.txt             # afficher tout le contenu
less fichier.txt            # afficher page par page (q pour quitter)
head -n 20 fichier.txt      # afficher les 20 premières lignes
tail -n 20 fichier.txt      # afficher les 20 dernières lignes
tail -f fichier.log         # suivre un fichier en temps réel
```

### Redirections et pipes

```bash
commande > fichier.txt      # redirige la sortie (écrase)
commande >> fichier.txt     # redirige la sortie (ajoute)
commande 2> erreurs.txt     # redirige les erreurs
commande > out.txt 2>&1     # redirige sortie ET erreurs

commande1 | commande2       # pipe : envoie la sortie de 1 vers 2
echo "hello" | tr 'a-z' 'A-Z'  # exemple : convertit en majuscules
```

---

## Permissions

### Lire les permissions

```bash
ls -la
# -rwxr-xr-- 1 user group 1234 jan 1 fichier.txt
#  ↑↑↑↑↑↑↑↑↑
#  │└──┬───┘└──┬───┘└──┬──┘
#  │  owner   group   others
#  └─ type (- fichier, d dossier, l lien)
```

`r` = lecture (4), `w` = écriture (2), `x` = exécution (1)

### Modifier les permissions

```bash
# Notation symbolique
chmod +x script.sh          # ajouter l'exécution
chmod u+w fichier.txt       # ajouter l'écriture pour le propriétaire
chmod go-r fichier.txt      # retirer la lecture pour group et others

# Notation octale
chmod 755 script.sh         # rwxr-xr-x
chmod 644 fichier.txt       # rw-r--r--
chmod 600 cle_privee        # rw------- (clé SSH)
```

### Changer le propriétaire

```bash
chown user fichier.txt
chown user:group fichier.txt
chown -R user:group dossier/    # récursif
```

---

## Processus

### Surveiller les processus

```bash
ps aux                      # liste tous les processus
ps aux | grep nginx         # filtrer par nom
top                         # vue temps réel (q pour quitter)
htop                        # vue améliorée (si installé)
```

### Contrôler les processus

```bash
kill 1234                   # envoie SIGTERM au PID 1234
kill -9 1234                # envoie SIGKILL (force)
killall nginx               # tue tous les processus nommés nginx

commande &                  # lancer en arrière-plan
jobs                        # liste les tâches en arrière-plan
fg %1                       # ramener la tâche 1 au premier plan
bg %1                       # reprendre la tâche 1 en arrière-plan

nohup commande &            # lancer sans être tué à la fermeture du terminal
```

### Variables de processus

```bash
$$      # PID du script en cours
$!      # PID du dernier processus lancé en arrière-plan
$?      # code de retour de la dernière commande (0 = succès)
```

---

## find et grep

### find — rechercher des fichiers

```bash
find . -name "*.log"                    # par nom (récursif depuis .)
find /var -name "*.log" -type f         # fichiers uniquement (pas dossiers)
find . -type d -name "node_modules"     # dossiers nommés node_modules
find . -mtime -7                        # modifiés il y a moins de 7 jours
find . -size +10M                       # fichiers de plus de 10 Mo

# Exécuter une commande sur chaque résultat
find . -name "*.tmp" -exec rm {} \;     # supprimer tous les .tmp
find . -name "*.sh" -exec chmod +x {} \;
```

### grep — rechercher dans le contenu

```bash
grep "motif" fichier.txt                # chercher dans un fichier
grep -r "motif" ./dossier               # chercher récursivement
grep -i "motif" fichier.txt             # insensible à la casse
grep -n "motif" fichier.txt             # affiche les numéros de ligne
grep -l "motif" *.txt                   # affiche seulement les noms de fichiers
grep -v "motif" fichier.txt             # lignes ne contenant PAS le motif
grep -E "erreur|warning" fichier.log    # regex étendue (alternance)
```

### Combinaisons utiles

```bash
find . -name "*.js" | xargs grep -l "console.log"   # fichiers JS contenant console.log
grep -r "TODO" . --include="*.py"                    # limiter aux .py
ps aux | grep -v grep | grep nginx                   # exclure la ligne grep elle-même
```

---

## Écrire un script

### Structure de base

```bash
#!/bin/bash
# Description courte du script

set -e          # quitter si une commande échoue
set -u          # erreur si variable non définie
set -o pipefail # propager les erreurs dans les pipes

echo "Début du script"
# ... commandes ...
echo "Terminé"
```

### Rendre un script exécutable et le lancer

```bash
chmod +x mon-script.sh
./mon-script.sh

# ou sans le rendre exécutable :
bash mon-script.sh
```

---

## Variables et paramètres

### Variables

```bash
NOM="Alice"                 # affectation (pas d'espaces autour de =)
echo $NOM                   # utiliser la variable
echo "${NOM}s"              # délimiter le nom de variable : affiche "Alices"
echo '$NOM'                 # guillemets simples : pas d'interpolation → $NOM

readonly VERSION="1.0.0"    # variable en lecture seule
unset NOM                   # supprimer une variable
```

### Paramètres du script

```bash
$0          # nom du script
$1, $2 ...  # arguments positionnels
$@          # tous les arguments (liste)
$#          # nombre d'arguments

# Exemple :
./deploy.sh prod v1.2.0
# $1 = "prod", $2 = "v1.2.0", $# = 2
```

### Substitution et valeurs par défaut

```bash
echo ${NOM:-"inconnu"}      # utilise "inconnu" si NOM est vide ou non défini
echo ${NOM:="inconnu"}      # assigne "inconnu" si NOM est vide ou non défini
NOM_UPPER=${NOM^^}          # convertir en majuscules (bash 4+)
NOM_LOWER=${NOM,,}          # convertir en minuscules (bash 4+)
```

---

## Conditions et boucles

### if / elif / else

```bash
if [ condition ]; then
    echo "vrai"
elif [ autre_condition ]; then
    echo "autre"
else
    echo "faux"
fi
```

### Opérateurs de test courants

```bash
# Fichiers
[ -f fichier ]      # existe et est un fichier
[ -d dossier ]      # existe et est un dossier
[ -e chemin ]       # existe (fichier ou dossier)
[ -x fichier ]      # est exécutable

# Chaînes
[ -z "$var" ]       # chaîne vide
[ -n "$var" ]       # chaîne non vide
[ "$a" = "$b" ]     # égalité de chaînes
[ "$a" != "$b" ]    # inégalité

# Nombres
[ $a -eq $b ]       # égal
[ $a -ne $b ]       # différent
[ $a -lt $b ]       # inférieur
[ $a -gt $b ]       # supérieur
```

### Boucle for

```bash
for i in 1 2 3 4 5; do
    echo "Valeur : $i"
done

# Sur une plage
for i in {1..10}; do echo $i; done

# Sur des fichiers
for fichier in *.txt; do
    echo "Traitement de $fichier"
done

# Sur les arguments du script
for arg in "$@"; do
    echo "$arg"
done
```

### Boucle while

```bash
compteur=0
while [ $compteur -lt 5 ]; do
    echo "Compteur : $compteur"
    ((compteur++))
done

# Lire un fichier ligne par ligne
while IFS= read -r ligne; do
    echo "$ligne"
done < fichier.txt
```

### case

```bash
case "$1" in
    start)
        echo "Démarrage..."
        ;;
    stop)
        echo "Arrêt..."
        ;;
    restart)
        echo "Redémarrage..."
        ;;
    *)
        echo "Usage : $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

---

## Fonctions

```bash
# Déclaration
ma_fonction() {
    local param1="$1"       # variable locale à la fonction
    local param2="$2"
    echo "Paramètres : $param1, $param2"
    return 0                # code de retour (0 = succès, 1+ = erreur)
}

# Appel
ma_fonction "hello" "world"

# Vérifier le code de retour
if ! ma_fonction "test"; then
    echo "La fonction a échoué"
fi
```

### Récupérer une valeur de retour

Bash ne peut retourner que des entiers via `return`. Pour retourner une chaîne, utiliser `echo` et capturer avec `$()` :

```bash
obtenir_date() {
    echo "$(date +%Y-%m-%d)"
}

AUJOURD_HUI=$(obtenir_date)
echo "Nous sommes le $AUJOURD_HUI"
```

### Codes de retour

```bash
commande
echo $?         # 0 = succès, autre = erreur

# Convention
exit 0          # fin de script avec succès
exit 1          # fin de script avec erreur
```
