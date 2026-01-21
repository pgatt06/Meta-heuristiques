# Meta-heuristiques pour le placement de capteurs

Ce dépôt contient un notebook qui implémente et compare plusieurs heuristiques et méta-heuristiques pour résoudre un problème de placement de capteurs. L'objectif est de couvrir toutes les cibles et d'assurer la connectivité au puits, tout en minimisant le nombre de capteurs utilisés.

## Contenu principal

Le cœur du projet se trouve dans le fichier `projet_capteurs_metaheuristiques.ipynb`. Ce notebook est structuré par sections :
- Génération et chargement d'instances, avec visualisation,
- Pré-calculs (couverture, graphe de communication, plus courts chemins),
- Évaluation et faisabilité des solutions (couverture + connexité),
- Heuristiques constructives (plus proche voisin, insertion, glouton profit/coût),
- Réparation et élagage des solutions,
- Recherche locale (swap, fusion 2->1, fusion n->1),
- Méta-heuristiques mono-solution (VNS, VND, GRASP),
- Méta-heuristique populationnelle (algorithme génétique),
- Campagnes de tests sur les jeux d'instances.

Le notebook s'appuie sur deux rayons `Rcapt` (couverture) et `Rcom` (communication), avec la contrainte `Rcom >= Rcapt`.

## Données d'instances

Les jeux d'instances sont fournis sous forme de fichiers `.dat` disponibles dans les dossiers suivants :
- `Projet de métaheuristiques - Instances grilles tronquées`
- `Projet de métaheuristiques - Instances cibles générées aléatoirement`

Des archives ZIP contenant ces dossiers sont également disponibles à la racine du dépôt.

Pour simplifier l'accès aux données, deux variables ont été créées :
- `DATA_DIR_TRONQ` : chemin vers `Projet de métaheuristiques - Instances grilles tronquées`
- `DATA_DIR_ALEA` : chemin vers `Projet de métaheuristiques - Instances cibles générées aléatoirement`

Ces variables peuvent être utilisées dans le notebook pour référencer les chemins des fichiers.


## Instructions pour exécuter le notebook

1. Ouvrez le fichier `projet_capteurs_metaheuristiques.ipynb` dans Jupyter.
2. Exécutez les cellules dans l'ordre.
3. Si nécessaire, ajustez `DATA_DIR_TRONQ` et `DATA_DIR_ALEA` pour pointer vers les dossiers d'instances locaux.

## Résultats

Les résultats (temps d'exécution, taille des solutions, faisabilité) sont affichés directement dans le notebook. Des figures sont également générées pour visualiser les placements de capteurs et les connexions.

## Notes

Le notebook contient des cellules d'exemples et de benchmarks qui parcourent les instances des deux jeux de données pour comparer temps d'exécution, taille de solution et faisabilité.
