# Questions-réponses sur PDF, par génération augmentée

Une application Streamlit qui laisse interroger un document PDF en langage naturel, plutôt que de le lire. On dépose le fichier, on pose une question, l'application retrouve les passages pertinents et rédige la réponse à partir d'eux.

Le modèle est Gemini Pro, la chaîne est bâtie avec LangChain, et les vecteurs sont stockés dans Chroma.

## Comment ça marche

Le principe de la génération augmentée par la recherche : ne pas demander au modèle ce qu'il sait, mais lui donner les bons extraits et lui demander de rédiger à partir d'eux. On évite ainsi les réponses inventées sur un document qu'il n'a jamais vu.

| Étape | Ce qui se passe |
| --- | --- |
| **Extraction** | Le texte est tiré du PDF avec PyPDF2. |
| **Découpage** | `RecursiveCharacterTextSplitter` de LangChain coupe le texte en fragments de taille exploitable, avec recouvrement pour ne pas trancher au milieu d'une idée. |
| **Vectorisation** | Chaque fragment est plongé dans un espace vectoriel par `GoogleGenerativeAIEmbeddings`. |
| **Stockage** | Les vecteurs vont dans une base Chroma. |
| **Recherche** | À la question posée, les fragments les plus proches sémantiquement sont retrouvés. |
| **Rédaction** | Gemini Pro compose la réponse à partir de ces seuls fragments. |

Le point qui compte est le dernier : la réponse est contrainte par ce qui a été retrouvé. Si le document ne contient pas l'information, le modèle n'a rien pour la fabriquer.

## Contenu du dépôt

| Fichier | Rôle |
| --- | --- |
| `rag.py` | La chaîne complète et l'interface Streamlit |
| `config/globals.py` | Paramètres : modèle, taille des fragments, recouvrement |
| `requirements.txt` | Dépendances |

## Exécution

```bash
pip install -r requirements.txt
```

Renseignez votre clé dans un fichier `.env` à la racine :

```
GOOGLE_API_KEY=votre_clé
```

Puis :

```bash
streamlit run rag.py
```

## Limites connues

Le découpage en fragments de taille fixe ignore la structure du document : un tableau ou une section peut se retrouver coupé en deux, et la recherche ne ramène alors qu'une moitié de réponse. Un découpage guidé par les titres donnerait de meilleurs résultats sur des documents longs et structurés.

La recherche est purement sémantique. Une question portant sur un chiffre précis ou une référence exacte est mieux servie par une recherche hybride, combinant vecteurs et correspondance de termes.
