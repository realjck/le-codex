# Configurer Ollama

Dans ce guide, nous allons parcourir les étapes pour installer OLLaMA (Open Large Language Model Assistant) sur macOS, Linux et Windows. OLLaMA est un modèle de langage open-source qui peut être utilisé pour diverses tâches de traitement du langage naturel en mode local. Nous fournirons également un exemple d'utilisation d'OLLaMA en Python.

## Installation

L'installation fonctionne sur Linux, macOS et Windows.

**Téléchargement et installation :**

- Téléchargez l'installeur OLLaMA depuis le [site officiel](https://ollama.com/).

- Double-cliquez sur le fichier téléchargé et suivez l'assistant d'installation.

## Utilisation d'un modèle préconstruit

Pour exécuter et discuter depuis le terminal avec un modèle préconstruit comme Llama 3.2, suivez ces étapes :

1. **Télécharger le modèle :**

```bash
ollama pull llama3.2
```

2. **Exécuter le modèle :**

```bash
ollama run llama3.2
```

## Bibliothèque de modèles

OLLaMA propose une bibliothèque de modèles préconstruits que vous pouvez facilement télécharger et utiliser. Les modèles à choisir se trouvent sur [la bibliothèque Ollama](https://ollama.com/library?sort=popular).

🧐 **Note :** Assurez-vous que votre GPU dispose de suffisamment de RAM pour exécuter le modèle choisi. Par exemple, vous aurez besoin d'au moins 8 Go de RAM pour les modèles 7B, 16 Go pour les modèles 13B, et 32 Go pour les modèles 33B. Pour savoir quel modèle exécuter sur votre carte graphique, consultez les [spécifications GPU](https://github.com/ollama/ollama/blob/main/docs/gpu.md).

## Utilisation d'OLLaMA en Python

Voici un exemple d'utilisation d'OLLaMA en Python :

```python
import ollama

# Générer du texte
response = ollama.generate(model='llama3.2', prompt='Quelle est la capitale de la France ?')
print(response['response'])
```

Dans cet exemple, nous importons la classe OLLaMA, initialisons le modèle avec le modèle préconstruit souhaité (par exemple, "llama3.2"), et générons du texte à partir d'une invite donnée.

## Commandes supplémentaires

- `ollama rm llama3.2` : Supprimer un modèle téléchargé
- `ollama cp llama3.2 my-model` : Copier un modèle
- `ollama list` : Lister les modèles téléchargés sur votre ordinateur
- `ollama ps` : Lister les modèles actuellement chargés
- `ollama stop llama3.2` : Arrêter un modèle en cours d'exécution

## Utilisation via Docker

Pour utiliser OLLaMA via Docker, suivez ces étapes :

1. **Télécharger l'image Docker :**
    
    Vous pouvez tirer l'image Docker officielle d'OLLaMA depuis Docker Hub en utilisant la commande suivante :
    
    `docker pull ollama/ollama`
    
2. **Exécuter un conteneur Docker :**
    
    Une fois l'image téléchargée, vous pouvez exécuter un conteneur Docker avec OLLaMA. Par exemple, pour exécuter le modèle `llama3.2` :
    
    `docker run -it --gpus all ollama/ollama ollama run llama3.2`
    
    Assurez-vous que Docker est configuré pour utiliser votre GPU, car les modèles de langage nécessitent souvent des ressources GPU pour fonctionner efficacement.
    
3. **Utiliser des volumes pour les modèles :**
    
    Si vous souhaitez persister les modèles téléchargés entre les exécutions de conteneurs, vous pouvez monter un volume Docker. Cela permet de stocker les modèles sur votre machine hôte :
    
    `docker run -it --gpus all -v /path/to/models:/root/.ollama ollama/ollama ollama run llama3.2`
    
    Remplacez `/path/to/models` par le chemin où vous souhaitez stocker les modèles sur votre machine hôte.
    

En utilisant Docker, vous pouvez facilement gérer les dépendances et les environnements pour OLLaMA, ce qui est particulièrement utile pour les déploiements sur différentes machines ou dans des environnements de développement isolés.
