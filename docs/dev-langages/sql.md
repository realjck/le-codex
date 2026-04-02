# SQL

SQL (Structured Query Language) est le langage standard pour interagir avec les bases de données relationnelles (MySQL, PostgreSQL, SQLite, SQL Server…). Il permet de créer des structures de données, d'y insérer des enregistrements, de les interroger et de les modifier.

## Opérations CRUD

### SELECT — Lire des données

```sql
-- Toutes les colonnes
SELECT * FROM utilisateurs;

-- Colonnes spécifiques
SELECT nom, email FROM utilisateurs;

-- Avec alias
SELECT nom AS "Nom complet", email AS "Adresse mail" FROM utilisateurs;

-- Valeurs distinctes
SELECT DISTINCT pays FROM utilisateurs;

-- Limiter les résultats
SELECT * FROM produits LIMIT 10;
SELECT * FROM produits LIMIT 10 OFFSET 20; -- pagination
```

### INSERT — Insérer des données

```sql
-- Insérer une ligne
INSERT INTO utilisateurs (nom, email, age)
VALUES ('Alice Martin', 'alice@example.com', 28);

-- Insérer plusieurs lignes
INSERT INTO utilisateurs (nom, email, age) VALUES
  ('Bob Dupont', 'bob@example.com', 34),
  ('Clara Petit', 'clara@example.com', 22);
```

### UPDATE — Modifier des données

```sql
-- Toujours utiliser WHERE pour éviter de modifier toute la table
UPDATE utilisateurs
SET email = 'alice.martin@example.com', age = 29
WHERE id = 1;
```

### DELETE — Supprimer des données

```sql
-- Supprimer des lignes ciblées
DELETE FROM utilisateurs WHERE id = 5;

-- Vider une table (plus rapide que DELETE sans WHERE)
TRUNCATE TABLE logs;
```

## Filtrer avec WHERE

```sql
-- Comparaisons
SELECT * FROM produits WHERE prix > 50;
SELECT * FROM commandes WHERE statut = 'en cours';
SELECT * FROM produits WHERE stock IS NULL;

-- Opérateurs logiques
SELECT * FROM produits WHERE prix > 20 AND stock > 0;
SELECT * FROM produits WHERE categorie = 'livre' OR categorie = 'jeu';
SELECT * FROM utilisateurs WHERE pays NOT IN ('FR', 'BE');

-- Plage de valeurs
SELECT * FROM produits WHERE prix BETWEEN 10 AND 50;

-- Recherche de texte (LIKE)
SELECT * FROM utilisateurs WHERE nom LIKE 'Mar%';   -- commence par "Mar"
SELECT * FROM produits WHERE description LIKE '%bio%'; -- contient "bio"
```

## Trier et Grouper

### ORDER BY

```sql
SELECT * FROM produits ORDER BY prix ASC;   -- croissant
SELECT * FROM produits ORDER BY prix DESC;  -- décroissant
SELECT * FROM commandes ORDER BY date DESC, montant ASC; -- tri multiple
```

### GROUP BY et fonctions d'agrégation

```sql
-- Fonctions d'agrégation courantes : COUNT, SUM, AVG, MIN, MAX
SELECT pays, COUNT(*) AS nb_utilisateurs
FROM utilisateurs
GROUP BY pays;

SELECT categorie, AVG(prix) AS prix_moyen, MAX(prix) AS prix_max
FROM produits
GROUP BY categorie;

-- HAVING : filtrer après le GROUP BY (contrairement à WHERE)
SELECT pays, COUNT(*) AS nb
FROM utilisateurs
GROUP BY pays
HAVING COUNT(*) > 10;
```

## Jointures (JOIN)

Les jointures combinent des données de plusieurs tables.

```sql
-- INNER JOIN : uniquement les lignes qui correspondent dans les deux tables
SELECT c.id, u.nom, c.montant
FROM commandes c
INNER JOIN utilisateurs u ON c.utilisateur_id = u.id;

-- LEFT JOIN : toutes les lignes de gauche + correspondances de droite (NULL si aucune)
SELECT u.nom, c.montant
FROM utilisateurs u
LEFT JOIN commandes c ON u.id = c.utilisateur_id;

-- RIGHT JOIN : toutes les lignes de droite + correspondances de gauche
SELECT u.nom, c.montant
FROM utilisateurs u
RIGHT JOIN commandes c ON u.id = c.utilisateur_id;

-- FULL OUTER JOIN : toutes les lignes des deux tables (non supporté en MySQL, utiliser UNION)
SELECT u.nom, c.montant
FROM utilisateurs u
LEFT JOIN commandes c ON u.id = c.utilisateur_id
UNION
SELECT u.nom, c.montant
FROM utilisateurs u
RIGHT JOIN commandes c ON u.id = c.utilisateur_id;
```

## Sous-requêtes

Une sous-requête est une requête imbriquée dans une autre.

```sql
-- Dans un WHERE
SELECT nom FROM utilisateurs
WHERE id IN (
  SELECT utilisateur_id FROM commandes WHERE montant > 100
);

-- Dans un FROM (table dérivée)
SELECT pays, total
FROM (
  SELECT pays, COUNT(*) AS total FROM utilisateurs GROUP BY pays
) AS stats
WHERE total > 5;

-- Sous-requête corrélée (fait référence à la requête externe)
SELECT nom, salaire
FROM employes e
WHERE salaire > (
  SELECT AVG(salaire) FROM employes WHERE departement = e.departement
);
```

## Créer et Modifier des Tables (DDL)

```sql
-- Créer une table
CREATE TABLE produits (
  id        INT PRIMARY KEY AUTO_INCREMENT,
  nom       VARCHAR(100) NOT NULL,
  prix      DECIMAL(10, 2) NOT NULL,
  stock     INT DEFAULT 0,
  categorie VARCHAR(50),
  cree_le   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Ajouter une colonne
ALTER TABLE produits ADD COLUMN description TEXT;

-- Modifier une colonne
ALTER TABLE produits MODIFY COLUMN nom VARCHAR(200) NOT NULL;

-- Supprimer une colonne
ALTER TABLE produits DROP COLUMN description;

-- Supprimer une table
DROP TABLE IF EXISTS produits;
```

## Index

Les index accélèrent les recherches mais ralentissent les écritures. À utiliser sur les colonnes fréquemment utilisées dans les `WHERE` et les `JOIN`.

```sql
-- Créer un index simple
CREATE INDEX idx_email ON utilisateurs (email);

-- Index unique (contrainte d'unicité)
CREATE UNIQUE INDEX idx_email_unique ON utilisateurs (email);

-- Index composé (plusieurs colonnes)
CREATE INDEX idx_nom_pays ON utilisateurs (nom, pays);

-- Voir les index d'une table (MySQL)
SHOW INDEX FROM utilisateurs;

-- Supprimer un index
DROP INDEX idx_email ON utilisateurs;
```

## Transactions

Les transactions garantissent que plusieurs opérations s'exécutent entièrement ou pas du tout (principe ACID).

```sql
START TRANSACTION;

UPDATE comptes SET solde = solde - 100 WHERE id = 1;
UPDATE comptes SET solde = solde + 100 WHERE id = 2;

-- Si tout s'est bien passé
COMMIT;

-- En cas d'erreur, annuler toutes les opérations
ROLLBACK;
```

## Fonctions utiles

```sql
-- Chaînes
SELECT UPPER(nom), LOWER(email) FROM utilisateurs;
SELECT CONCAT(prenom, ' ', nom) AS nom_complet FROM utilisateurs;
SELECT LENGTH(nom) AS longueur FROM utilisateurs;
SELECT TRIM('  bonjour  '); -- supprime les espaces
SELECT SUBSTRING(email, 1, 5); -- 5 premiers caractères

-- Dates
SELECT NOW();                        -- date et heure actuelles
SELECT CURDATE();                    -- date du jour
SELECT DATE_FORMAT(cree_le, '%d/%m/%Y') FROM commandes;
SELECT DATEDIFF(NOW(), cree_le) AS jours_depuis FROM commandes;

-- Numériques
SELECT ROUND(prix, 2) FROM produits;
SELECT CEIL(4.1);   -- 5
SELECT FLOOR(4.9);  -- 4

-- Valeur par défaut si NULL
SELECT COALESCE(telephone, 'Non renseigné') FROM utilisateurs;
```

## Conclusion

Pour aller plus loin, se référer à la documentation officielle de votre SGBD :
[PostgreSQL](https://www.postgresql.org/docs/) · [MySQL](https://dev.mysql.com/doc/) · [SQLite](https://www.sqlite.org/docs.html)
