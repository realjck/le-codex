# RegEx

Les **expressions régulières**, souvent abrégées en RegEx, sont des outils puissants pour rechercher et manipuler des séquences de caractères. Elles permettent de trouver, remplacer, extraire et analyser des informations textuelles complexes avec une grande précision et flexibilité.

Cette fiche recense quelques syntaxes couramment utilisées.

## Méthodes de vérification (JS)

### Rechercher les correspondances :
```js
'string'.match(/regex/);
// [...]
```

### Vérifier la correspondance :
```js
/regex/.test('string');
// true | false
```

## Syntaxe de base

- Utiliser les flags `g` (global) et `i` (ignore case) : `/abc/gi`

- Échapper les caractères spéciaux avec l'antislash : `/\$/`

- Wildcard (tout caractère) : `.`

- Apparait une ou plusieurs fois à la suite : `/a+/g`

- Apparaît zéro ou plusieurs fois à la suite : `/a*/g`

- Début de chaîne : `/^Ricky/`

- Fin de chaîne : `/story$/`

- Classe de caractère : `/b[aiu]g/`

- Champs de caractères dans une classe : `/[a-e]at/`

- Classes négatives : `/[^aeiou]/`

- Caractères spéciaux (espace blanc, retour chariot, tabulation, saut de page et caractères de nouvelle ligne) : `\s\r\t\f\n\v`

## Nota bene

- Pensez à utiliser des outils d'IA générative pour vous aider à créer vos regex. Ces outils peuvent vous faire gagner du temps et vous éviter des erreurs.

- N'oubliez pas de commenter vos RegEx pour en faciliter la compréhension.
