# PHP

PHP signifie **Hypertext Preprocessor**. C'est un langage de programmation libre principalement utilisé pour produire des pages Web dynamiques via un serveur Web.

Cet aide-mémoire est une référence rapide et pratique. Il couvre les concepts fondamentaux du langage et de ses fonctionnalités courantes.

## Syntaxe de base

```php
<?php
// Commentaire sur une ligne

/*
  Commentaires multi-lignes
*/

// Variables
$variable_name = "Value";  // Chaine de caractère
$number = 123;             // Entier
$float = 12.34;            // Décimal
$boolean = true;           // Booléen
$array = [1, 2, 3];        // Tableau

// Constants
define("CONSTANT_NAME", "Value");
const ANOTHER_CONSTANT = "Value";
?>
```

## Type de données

- Chaine de caractères : `"Hello, World!"`
- Entier : `123`
- Décimal (Float) : `12.34`
- Booléen : `true` or `false`
- Tableau : `["apple", "banana", "cherry"]`
- Objet
- NULL

### Chaines de caractères

```php
<?php
$str = "Hello";
$str2 = 'World';
$combined = $str . " " . $str2; // Concaténation

// Fonctions de chaines de caractères
strlen($str);            // Longueur d'une chaine
strpos($str, "e");       // Position de la première occurence
str_replace("e", "a", $str); // Remplacer toutes les occurrences
?>
```

### Tableaux

```php
<?php
$array = [1, 2, 3];
$assoc_array = ["key1" => "value1", "key2" => "value2"];

// Fonctions de tableaux
count($array);                  // Nombre d'éléments
array_push($array, 4);          // Ajouter un élément
array_merge($array, [4, 5]);    // Fusionner des tableaux
in_array(2, $array);            // Vérifie si un élément existe
?>
```

## Structures de contrôle

### If-else (si-sinon)

```php
<?php
if ($condition) {
    // code à exécuter si vraie
} elseif ($another_condition) {
    // code a exécuter si une autre condition est vraie
} else {
    // code à exécuter si toutes les conditions sont fausses
}
?>
```

### Switch (aiguillage)

```php
<?php
switch ($variable) {
    case "value1":
        // code à exécuter si la variable est égale à value1
        break;
    case "value2":
        // code à exécuter si la variable est égale à value2
        break;
    default:
        // code à exécuter si aucun cas ne correspond
}
?>
```

### Loops (boucles)

```php
<?php
// Boucle de type 'for'
for ($i = 0; $i < 10; $i++) {
    // Code à exécuter
}

// Boucle de type 'while' (tant que)
while ($condition) {
    // code à exécuter
}

// Boucle de type 'do-while' 
do {
    // code to execute
} while ($condition);

// Boucle de type foreach (énuération de tableau)
foreach ($array as $value) {
    // code à exécuter
}
?>
```

## Fonctions

```php
<?php
function functionName($param1, $param2) {
    // code à exécuter
    return $result;
}

$result = functionName($arg1, $arg2);
?>
```

### Variables superglobales

- `$_GET` – Variables envoyées par les paramètres URL
- `$_POST` – Variables envoyées par HTTP POST
- `$_REQUEST` – Variables envoyées par à la fois GET et POST
- `$_SERVER` – Information du serveur et de l'environnement d'exécution
- `$_SESSION` – Variables de session
- `$_COOKIE` – Cookies HTTP

## Traitement de fichiers

```php
<?php
// Lire un fichier
$file = fopen("filename.txt", "r");
$content = fread($file, filesize("filename.txt"));
fclose($file);

// Ecrire dans un fichier
$file = fopen("filename.txt", "w");
fwrite($file, "Hello, World!");
fclose($file);
?>
```

## Traitement des erreurs

```php
<?php
try {
    // Code qui peut lever une exception
    if ($condition) {
        throw new Exception("Error message");
    }
} catch (Exception $e) {
    // Code pour gérer l'exception
    echo "Caught exception: " . $e->getMessage();
} finally {
    // Code à toujours exécuter
}
?>
```

## Base de données (MySQLi)

```php
<?php
// Créer une connexion
$conn = new mysqli($servername, $username, $password, $dbname);

// Vérifier la connexion
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

// Récupérer les données
$sql = "SELECT id, firstname, lastname FROM MyGuests";
$result = $conn->query($sql);

if ($result->num_rows > 0) {
    while($row = $result->fetch_assoc()) {
        echo "id: " . $row["id"]. " - Name: " . $row["firstname"]. " " . $row["lastname"]. "<br>";
    }
} else {
    echo "0 results";
}

$conn->close();
?>
```

## Gestion de session

```php
<?php
// Démarrer la session
session_start();

// Définir les variables de session
$_SESSION["username"] = "JohnDoe";
$_SESSION["email"] = "john@example.com";

// Récupérer les variables de session
echo $_SESSION["username"];

// Terminer la session
session_destroy();
?>
```

## Include & Require

```php
<?php
include 'filename.php';  // Inclus le fichier, alerte si non trouvé
require 'filename.php';  // Nécessite le fichier, erreur si absent

include_once 'filename.php';  // Inclus le fichier, vérifie si déjà présent
require_once 'filename.php';  // Nécessite le fichier, vérifie si déjà présent
?>
```
