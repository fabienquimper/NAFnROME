# Génération et recherche de codes NAF/ROME

Ce projet génère deux ressources principales :
- un CSV enrichi des codes NAF avec des descriptions longues provenant de l’INSEE ;
- un CSV fusionnant NAF + ROME avec une tentative d’association d’un code NAF pour chaque métier ROME (approche heuristique, résultats non garantis).

## Pipeline des notebooks
- `01_BUILD__rome_csv.ipynb` : télécharge le référentiel ROME, extrait le XML et le convertit en `rome.csv`.
- `02_BUILD__naf_codes_001_csv.ipynb` : télécharge le fichier XLS NAF INSEE et produit `naf_codes_001.csv` (codes + intitulés).
- `03_EXTRACT_naf_codes_001_desc_csv__from_pdf.ipynb` : télécharge le PDF NAF/CPF INSEE, extrait et nettoie les descriptions longues, puis génère `naf_codes_001_desc.csv`.
- `04_GEN__rome_with_naf_csv.ipynb` : calcule des similarités (SentenceTransformer all-MiniLM-L6-v2) pour associer un code NAF à chaque entrée ROME, produit `rome_with_naf_all-MiniLM-L6-v2.csv` (association perfectible).
- `05_BUILD__fusion_naf_rome_001_csv.ipynb` : concatène les données NAF et ROME enrichies en un seul fichier `fusion_naf_rome_001_allMiniLM_L6_v2.csv`.
- `06_FIND_best_NAF_codes_from_keywords_.ipynb` : quelques tests de recherche par mots-clés ou embeddings pour retrouver des codes NAF/ROME (exemples : « professeur yoga », « traiteur », « fast food »).

## Usage rapide
1) Exécuter les notebooks dans l’ordre 01 → 05 pour produire les CSV intermédiaires et finaux.
2) Utiliser le notebook 06 pour tester la recherche de codes NAF à partir de mots-clés simples et inspecter les résultats.

## Installation & exécution locale
- Prérequis : Python 3.10+ recommandé ; git ; accès Internet (téléchargements INSEE/ROME et modèles de phrases).
- Option venv + pip  
  - PowerShell : `python -m venv .venv && .\.venv\Scripts\Activate.ps1`  
  - Unix/macOS : `python -m venv .venv && source .venv/bin/activate`  
  - Dépendances : `pip install -r requirements.txt`
- Option Poetry  
  - Installer Poetry (https://python-poetry.org/docs/).  
  - `poetry install` (crée un env isolé).  
  - `poetry shell` pour activer l’environnement.
- Enregistrer le venv comme kernel Jupyter (à faire une seule fois) :
  `pip install ipykernel`
  `python -m ipykernel install --user --name=NAFnROME --display-name "Python (NAFnROME)"`
  > **Pourquoi ?** Si Jupyter est installé sur le système (ex: `/usr/bin/jupyter`), il utilise par défaut le kernel `python3` système, pas le venv. Sans cette étape, les imports échouent (`ModuleNotFoundError`) même si les dépendances sont bien dans le venv.
- Lancer Jupyter/Lab : `jupyter lab` ou `jupyter notebook`, puis **changer le kernel** : *Kernel → Change Kernel → "Python (NAFnROME)"*, ensuite exécuter les notebooks dans l’ordre (01 → 06).
- Les poids du modèle `sentence-transformers/all-MiniLM-L6-v2` seront téléchargés automatiquement au premier run.

## Limitations connues
- L’association ROME → NAF repose sur la similarité de texte : elle est indicative et peut contenir des erreurs.
- Les descriptions NAF extraites du PDF peuvent parfois inclure du bruit résiduel selon la mise en page.

## Sorties clés

### Fichier final : `fusion_naf_rome_001_allMiniLM_L6_v2.csv`

C'est le fichier principal, produit par le notebook `05`. Il regroupe codes NAF et ROME en une seule base exploitable pour la recherche et le prototypage.

**Colonnes :**

| Colonne | Description |
|---|---|
| `code_naf` | Code NAF INSEE (ex: `46.61Z`) |
| `code_rome` | Code ROME France Travail (ex: `A1101`) |
| `name` | Intitulé du métier ROME |
| `desc` | Appellation détaillée avec code OGR |

**Exemple de lignes :**

```
code_naf,code_rome,name,desc
46.61Z,A1101,Conducteur / Conductrice d'engins agricoles,Chauffeur / Chauffeuse de machines agricoles (code OGR:11987) ...
86.90A,A1101,Conducteur / Conductrice d'engins agricoles,Conducteur / Conductrice d'automoteur de récolte (code OGR:38874) ...
```

### Fichiers intermédiaires
- `rome.csv` : métiers ROME + intitulés + appellations.
- `naf_codes_001_desc.csv` : codes NAF avec descriptions longues INSEE.
- `rome_with_naf_all-MiniLM-L6-v2.csv` : tentative de mapping ROME → NAF (fichier intermédiaire, association indicative).

