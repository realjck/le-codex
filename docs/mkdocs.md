# Site avec MkDocs

MkDocs est un générateur de site statique conçu pour la création de documentation de projet. Il utilise le langage Markdown. 

Ci-dessous les consignes pour construire un site web avec [MkDocs](https://www.mkdocs.org/getting-started/) comme celui-ci :

- Avoir Python installé

```bash
# Installer MkDocs
pip install mkdocs
```

- Créer votre projet

```bash
mkdocs new <mon-projet>
cd <mon-projet>
```

_**Note:** Bien avoir le chemin des exécutables Python dans les variables d'environnement pour pouvoir exécuter MkDocs, sinon faire `python -m mkdocs ...`_

Le fichier de configuration se trouve à la racine du projet `mkdocs.yml`.

Les thèmes doivent être installés (ex.: `pip install mkdocs-material`), voir [https://www.mkdocs.org/user-guide/choosing-your-theme/](https://www.mkdocs.org/user-guide/choosing-your-theme/)

**Exemple de configuration :**

```yaml
site_name: TECH-BOOK-JCK
theme:
  name: readthedocs
  locale: fr
  logo: logo.png
  navigation_depth: 2
extra_css:
    - extra.css
```

Les fichiers extra.css et logo.png doivent être placés à la racine du dossier documents `docs`

## Servir votre projet

Une fois votre arborescence de fichier markdown placée dans `docs`, faire :

```bash
mkdocs serve
```

Le site est servi sur le port :8000

## Construire le site web

```bash
mkdocs build
```

Le build est généré dans `site`.
