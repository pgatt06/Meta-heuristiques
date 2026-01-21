# Meta-heuristiques pour placement de capteurs

Ce depot contient un notebook qui implemente et compare plusieurs heuristiques et meta-heuristiques pour un probleme de placement de capteurs. L'objectif est de couvrir toutes les cibles et d'assurer la connectivite au puits, tout en minimisant le nombre de capteurs.

## Notebook principal

Le coeur du projet est dans `projet_capteurs_metaheuristiques.ipynb`. Il inclut :
- generation/chargement d'instances et visualisation,
- pre-calculs (couverture, graphe de communication, plus courts chemins),
- evaluation/faisabilite des solutions,
- heuristiques constructives (plus proche voisin, insertion, glouton gain/cout),
- reparation et elagage,
- recherche locale (swap, fusion 2->1, fusion n->1),
- meta-heuristiques (VNS, VND, GRASP, genetique),
- campagnes de tests sur des instances fournies.

## Donnees d'instances

Les jeux d'instances sont fournis sous forme de fichiers `.dat` dans :
- `Projet de métaheuristiques - Instances grilles tronquées`
- `Projet de métaheuristiques - Instances cibles générées aléatoirement`

Des archives zip des memes dossiers sont aussi presentes a la racine.

## Prerequis

- Python 3
- Jupyter Notebook ou JupyterLab
- Bibliotheques Python : `numpy`, `matplotlib`

## Lancer le notebook

1) Ouvrir `projet_capteurs_metaheuristiques.ipynb` dans Jupyter.
2) Executer les cellules dans l'ordre.
3) Adapter les chemins de fichiers dans les cellules de tests (chemins Windows d'exemple) pour pointer vers les dossiers d'instances du depot. Exemple :

```python
path = "Projet de métaheuristiques - Instances grilles tronquées/Projet de métaheuristiques - Instances grilles tronquées"
```

## Resultats

Les resultats (temps, taille de solution, faisabilite) sont affiches dans le notebook, et des figures sont generees pour visualiser les placements de capteurs et les connexions.
