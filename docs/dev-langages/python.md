# Python

Python est un langage de programmation interprété, multi-paradigme et open-source, créé par Guido van Rossum et sorti pour la première fois en 1991. Python favorise un style de codage où le nombre de lignes de code est réduit au minimum, ce qui le rend facile à apprendre et à utiliser. Cette fiche technique vous fournira une compréhension de base de Python et de son utilisation.

## Bases du langage


```Python
>>> print ('abc' 'def')
# 'abcdef'

>>> a = 123_456_789
# 123456789

>>> arr1 = [1,2,3]
>>> arr2 = [4,5,6]
>>> arr1 + arr2
# 1,2,3,4,5,6

>>> print(r'no break\n')
# 'no break\n'

>>> print("""\
>>> one line
>>> two lines""")
# 'one line'
# 'two lines'

>>> word = 'python'
>>> word[-1]
# 'n'
>>> word[-2]
# 'o'
```

## Syntaxe

- l’Indentation par convention est **quatre espaces**
- les commentaires sont avec #

**On rajoute** **deux espaces** en fin de ligne avant un commentaire :

```Python
a = 1 + 2  # this is comment
```

- `f` pour formatter les chaines avec des variables :

```Python
name = 'Lulu'
hello = f'Salut {name}!'
# Salut Lulu!

addition = f'somme : {a+b}!'
```

### condition avec `elif` :

```Python
if a == 0:
    print('zero')
elif a == 1:
    print('one')
else:
    print('two or more')
```

### Booléens : conditions `and`, `or` et `not` en toutes lettres

```Python
if cond1 and cond2: ...

if not cond3: ...
```

### Boucles

```Python
for a in (1,2,3):
    print(a)
# 1 2 3		
		
while a > 0:
    print(a)
    a = a - 1
# 3 2 1
```

### Listes et Tuples

```Python
new_list = ['a', 'b', 2]  # CAN MODIFY
new_tuple = ('a', 'b', 2)  # CAN NOT MODIFY
```

### Dictionnaires

```Python
fruits_color = {
    "banana" : "yellow",
    "kiwi" : "green"
}
print(fruit_color["banana"])  # yellow
```

### Sequences

```Python
seq = 'Rocket'

print(seq[0:3])  # roc
print(seq[3:0:-1])  # kco
```

  
### Itérables

```Python
iterable = 'Rocket'

for i in iterable:
    print(i)
```

## Fonctions

```Python
def double(number):
    """Retourne nombre donné multiplé par deux."""
    return number * 2
```

Afficher la documentation :

```Python
help(double)
```

Valeur par défaut des paramètres :

```Python
def fly(animal="bird"):
    print(f"{animal} can fly")
		
fly()
```

### Technique pour trouver les nombres impairs :

```Python
l = [i for i in range(10) if i % 2]
```

## Définition de classes

```Python
class Animal:
    """Representation of an animal"""
    def __init__(self, name):
        self.name = name
				
    def say_hello(self):
        """Print a little presentation of the animal."""
        print(f"Hello, my name is {self.name}.")
				
tweety = Animal("Tweety")
tweety.say_hello()
```

### Héritage

```Python
class Duck(Animal):
    """Representation of a duck"""
    # surcharge :
    def say_hello(self):
        print(f"I'm a duck and my name is {self.name}")
			
bugs = Duck("Daffy")
bugs.say_hello()
```

Avec execution de la méthode parent :

```Python
class Bunny(Animal):
    """Representation of a bunny"""
    def say_hello(self):
        super().say_hello()
        print("I'm a bunny")
				
bugs = Bunny("Bugs")
bugs.say_hello()
```

## Mise en pratique

### Créez un environnement virtuel (`.venv/`)

```Bash
# python3 -m venv <folder>
python3 -m venv .venv
```

### Activez l'environnement virtuel (au lieu de l’environnement système) :

Linux/Mac :

```Shell
source venv/bin/activate

# verifier le chemin
which python3
```

(Windows :)

```PowerShell
.venv\Scripts\activate
```

### Installez les dépendances :

```Shell
pip install <package_foo> <package_bar>
```

### Générer le fichier de requirements.txt

```Bash
pip freeze > requirements.txt
```

### Installer les dépendances depuis requirements.txt

```Bash
pip install -r requirements.txt
```

### Désactiver l’environnement virtuel

```Bash
deactivate
```
