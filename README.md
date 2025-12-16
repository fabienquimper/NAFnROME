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

## Limitations connues
- L’association ROME → NAF repose sur la similarité de texte : elle est indicative et peut contenir des erreurs.
- Les descriptions NAF extraites du PDF peuvent parfois inclure du bruit résiduel selon la mise en page.

## Sorties clés
- `rome.csv` : métiers ROME + intitulés + appellations.
- `naf_codes_001_desc.csv` : codes NAF avec descriptions longues INSEE.
- `rome_with_naf_all-MiniLM-L6-v2.csv` : tentative de mapping ROME → NAF.
- `fusion_naf_rome_001_allMiniLM_L6_v2.csv` : base commune NAF + ROME pour recherche et prototypage.

