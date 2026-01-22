# Rapport de projet — Méta-heuristiques pour le placement de capteurs

## Résumé
Ce projet implémente et compare plusieurs **heuristiques** et **méta-heuristiques** pour un problème de **placement de capteurs**. Une solution doit (i) **couvrir** toutes les cibles avec un rayon `Rcapt`, (ii) assurer la **connectivité** des capteurs sélectionnés au **puits** via un rayon de communication `Rcom`, tout en (iii) **minimisant le nombre de capteurs** utilisés. L’ensemble est regroupé dans un notebook unique qui inclut la modélisation, les pré-calculs, les algorithmes, et des campagnes de tests.

---

## 1. Problème étudié

### 1.1 Données d’entrée
Une instance est définie par :
- un ensemble de `N` cibles/positions candidates `V = {0,…,N−1}`, avec coordonnées 2D `coords[i]`,
- un puits (sink) de coordonnées `sink`,
- deux paramètres :
  - `Rcapt` : rayon de couverture,
  - `Rcom` : rayon de communication.

> Hypothèse : `Rcom ≥ Rcapt` (communication au moins aussi “large” que la couverture).

### 1.2 Variables de décision
Une solution est un vecteur booléen `selected ∈ {0,1}^N` :
- `selected[v] = 1` signifie qu’un capteur est placé sur la cible `v`.

### 1.3 Contraintes

**(C1) Couverture.**  
Un capteur placé en `v` couvre l’ensemble :
\[
cover(v) = \{u \in V \mid d(u,v) \le Rcapt\}.
\]
La couverture est complète si :
\[
\forall u\in V,\ \exists v\in V \text{ tel que } selected[v]=1 \text{ et } u\in cover(v).
\]

**(C2) Connectivité au puits.**  
On définit un graphe de communication sur `V ∪ {sink}` :
- une arête existe entre deux nœuds si la distance est ≤ `Rcom`.
La solution est connectée si tous les capteurs sélectionnés appartiennent à la composante connexe contenant le puits dans le sous-graphe induit par `selected` (plus le puits).

### 1.4 Objectif
Objectif principal : minimiser le nombre de capteurs.
\[
\min \sum_{v\in V} selected[v]
\]
Dans les méta-heuristiques, une fonction pénalisée est utilisée pour comparer des solutions éventuellement infaisables :
\[
f(selected)=|selected|+\lambda_{cov}\cdot \#uncovered+\lambda_{con}\cdot penalty_{con}.
\]
Dans l’implémentation actuelle, `penalty_con` est **binaire** : `0` si la solution est connectée au puits, `1` sinon.

---

## 2. Instances et jeux de données

### 2.1 Instances synthétiques
- `make_grid(n, m, sink=(0.0, 0.0), name="grid")` : grille régulière `n × m`.
- `make_truncated_grid(n, m, holes, sink=(0.0, 0.0), name="trunc")` : grille avec suppression de rectangles (“trous”).

### 2.2 Instances depuis fichiers
- `load_points_txt(path, sink=(0.0, 0.0), name=None, truncated=False)` :
  - **mode normal** : lit des lignes `(x, y)` ou `id x y` (commentaires ignorés),
  - **mode tronqué** (`truncated=True`) : reconstruit la grille complète puis supprime les points listés comme “à enlever”.

Les instances `.dat` sont organisées en deux ensembles :
- grilles tronquées,
- cibles générées aléatoirement.

Deux variables du notebook facilitent l’accès :
- `DATA_DIR_TRONQ` (grilles tronquées),
- `DATA_DIR_ALEA` (instances aléatoires).

---

## 3. Pré-calculs (accélération des algorithmes)

La fonction `build_precomp(inst, Rcapt, Rcom)` construit les structures communes :

- **Couverture** : `cover[v]` = indices des cibles couvertes par un capteur en `v` (seuil `Rcapt`).
- **Communication** : `com_adj` = liste d’adjacence du graphe de communication sur `V ∪ {sink}` (seuil `Rcom`).
- **Arbre BFS depuis le puits** : `parent_from_sink` = parent de chaque nœud dans un BFS sur le graphe complet (sert à reconstruire des chemins “vers le puits” pour connecter des capteurs).

Ces pré-calculs permettent d’éviter des recalculs coûteux dans les boucles d’heuristiques.

---

## 4. Évaluation et faisabilité

- `compute_coverage_counts(selected, pc)` : nombre de capteurs couvrant chaque cible.
- `is_fully_covered(selected, pc)` : test de couverture complète.
- `is_connected_to_sink(selected, pc)` : BFS sur le sous-graphe induit par `selected` (plus le puits).
- `is_feasible(selected, pc, debug=False)` : solution faisable ssi couverture complète **et** connectivité au puits.
- `fitness_penalized(selected, pc, lam_cov=10.0, lam_con=10.0)` : valeur pénalisée (utile pour comparer des solutions pendant l’exploration).

> Remarque importante : le notebook utilise un **timeout soft** (`evaluate_algorithm_with_timeout`) : en cas de dépassement, on arrête l’attente du résultat, mais on ne “tue” pas nécessairement un calcul bloqué.

---

## 5. Méthodes implémentées

### 5.1 Outils de construction et de modification
- `add_sensor(v, selected, cov, uncovered, pc)` : ajoute un capteur et met à jour couverture et cibles non couvertes.
- `path_to_sink(node, pc, active_mask)` : calcule les relais à activer pour relier un nœud au puits via l’arbre BFS.
- `add_path_to_sink(v, selected, cov, uncovered, pc)` : ajoute les relais nécessaires pour connecter `v` (si possible).
- `plot_solution(selected, pc, show_links=False, max_links=3000)` : visualisation (option liens de communication).

### 5.2 Heuristiques constructives
Objectif : construire rapidement une solution “raisonnable”, souvent suivie d’une réparation.

- `nearest_neighbor_heuristic(pc)` : sélection itérative de la cible non couverte la plus proche du puits, avec tentative de connexion.
- `randomized_nearest_neighbor_heuristic(pc, alpha=0.2)` : variante aléatoire via une RCL (Restricted Candidate List) basée sur la distance au puits.
- `insertion_heuristic(pc)` : insertion guidée par un ratio type `gain / (1 + relais_ajoutés)`.
- `greedy_construct(pc, ...)` : glouton “profit/coût” basé sur `gain / (1 + coût_connexion)`, avec échantillonnage possible.
- `greedy_randomized_construction_cover(pc, alpha=0.2, rng=None)` : construction GRASP (RCL sur gain de couverture).

### 5.3 Réparation, élagage, recherche locale
Ces étapes transforment une solution quelconque en solution faisable et/ou plus compacte.

- `repair_coverage(selected, pc)` : ajoute des capteurs jusqu’à couvrir toutes les cibles.
- `repair_connectivity(selected, pc, max_passes=10)` : ajoute des relais pour connecter les capteurs au puits (via `parent_from_sink`).
- `prune_redundant(selected, pc, max_passes=5)` : supprime des capteurs redondants en conservant faisabilité.
- `repair(selected, pc)` : pipeline standard : `repair_coverage → repair_connectivity → prune_redundant`.
- `local_search_swaps(selected, pc, trials=500)` : voisinage “swap” (retirer un capteur, en ajouter un autre), suivi d’une réparation ; acceptation si réduction du nombre de capteurs.
- `merge_2_en_1(...)` / `merge_sensors_to_1(...)` : voisinages de fusion (2→1 puis k→1) si amélioration et faisabilité.

### 5.4 Méta-heuristiques
Ces méthodes pilotent l’exploration globale de l’espace des solutions.

- `vns(pc, kmax=8, iters=60, do_swaps=True)` : Variable Neighborhood Search (perturbation + réparation + amélioration).
- `vnd(selected, pc, ...)` : Variable Neighborhood Descent (enchaîne plusieurs voisinages et redémarre au premier lors d’une amélioration).
- `grasp(pc, max_iters=100, alpha=0.2, ...)` : GRASP = construction gloutonne randomisée (RCL) + recherche locale + réparation, répété.
- `genetic_algorithm(pc, ...)` : algorithme génétique (population, sélection tournoi, croisement, mutation), avec réparation des individus.

---

## 6. Reproductibilité : exécution

1. Ouvrir `projet_capteurs_metaheuristiques.ipynb` dans Jupyter.
2. Exécuter les cellules dans l’ordre.
3. Ajuster `DATA_DIR_TRONQ` et `DATA_DIR_ALEA` selon l’emplacement local des instances.

Les résultats (temps, taille, faisabilité) sont affichés dans le notebook. Des figures sont produites pour visualiser les solutions et, optionnellement, les liens de communication.

---

## 7. Livrable principal
- **Notebook** : `projet_capteurs_metaheuristiques.ipynb`  
Il contient l’ensemble du code (modélisation + algorithmes + expérimentations) et sert de support d’analyse comparative.
