# Exercice 1.7.5-synth : Apollo 13 Approximation Suite

**Module :**
1.7.5 — Approximation Algorithms

**Concept :**
synth — Vertex Cover 2-approx, Set Cover Greedy, TSP MST, Knapsack FPTAS, Bin Packing FFD

**Difficulte :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
3 — Synthese (tous concepts a-e)

**Langage :**
Rust Edition 2024 + C (C17)

**Prerequis :**
- Theorie des graphes (adjacence, parcours DFS/BFS)
- Structures de donnees (arbres, heaps, hash sets)
- Complexite algorithmique (Big-O)
- Algorithmes gloutons
- Programmation dynamique de base
- Notions de NP-completude

**Domaines :**
Algo, Struct, MD, Probas

**Duree estimee :**
120 min

**XP Base :**
150

**Complexite :**
T7 O(n^2) a O(n^3) selon algo x S5 O(n) a O(n^2)

---

## SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers a rendre :**
- `vertex_cover.rs` / `vertex_cover.c`
- `set_cover.rs` / `set_cover.c`
- `tsp_approx.rs` / `tsp_approx.c`
- `knapsack_fptas.rs` / `knapsack_fptas.c`
- `bin_packing.rs` / `bin_packing.c`

**Fonctions autorisees :**
- Rust : std::collections (HashMap, HashSet, BinaryHeap, VecDeque), std::cmp, iterateurs
- C : malloc, free, memcpy, memset, qsort, stdlib.h, string.h

**Fonctions interdites :**
- Bibliotheques d'optimisation externes
- Solveurs LP/ILP
- Rust : unsafe (sauf pour FFI avec C)

### 1.2 Consigne

**Section 1.2.1 : Version Culture Pop**

#### "HOUSTON, WE HAVE A PROBLEM... BUT A 2-APPROXIMATION SOLUTION!"

*13 avril 1970. A 321 860 kilometres de la Terre, le module de service d'Apollo 13 vient d'exploser. L'oxygene fuit. L'electricite diminue. Le CO2 augmente. Et trois astronautes comptent sur toi pour les ramener vivants.*

*Gene Kranz, directeur de vol, prononce sa phrase legendaire : "Failure is not an option." Mais ce qu'il ne dit pas, c'est que la perfection non plus n'est pas une option. Pas quand tu as 4 jours pour ramener un equipage avec des ressources limitees.*

*Les ingenieurs du Mission Control ont du resoudre des problemes que meme les ordinateurs modernes ne peuvent pas resoudre optimalement en temps polynomial. Ils ont fait ce que font les meilleurs : trouver des solutions "suffisamment bonnes" avec des GARANTIES mathematiques.*

**Ta mission :**

Tu es Ken Mattingly, l'astronaute cloue au sol par la rougeole, devenu le cerveau du sauvetage. Tu dois implementer 5 algorithmes d'approximation qui auraient pu sauver Apollo 13 :

---

#### Algorithme 1 : `vertex_cover_2approx` — Couverture d'Equipage

*Le LEM Aquarius n'a pas ete concu pour supporter 3 personnes pendant 4 jours. Chaque systeme critique (O2, CO2, electricite, navigation, communication) doit etre surveille par au moins un astronaute. Mais les astronautes sont epuises. Comment assigner le MINIMUM d'astronautes pour couvrir TOUS les systemes critiques ?*

C'est le probleme du **Vertex Cover** : etant donne un graphe ou les aretes representent des paires de systemes qui partagent des dependances, trouver le plus petit ensemble de sommets (astronautes) tel que chaque arete a au moins une extremite dans l'ensemble.

**Garantie :** Ton algorithme doit retourner un cover d'au plus **2 x OPT** sommets.

---

#### Algorithme 2 : `set_cover_greedy` — Couverture des Problemes

*La liste des problemes a resoudre s'allonge : fuite d'oxygene, accumulation de CO2, panne electrique, trajectoire incorrecte, froid extreme, deshydratation. Chaque "patch" disponible (procedure, outil, manoeuvre) resout un sous-ensemble de problemes. Comment couvrir TOUS les problemes avec le MINIMUM de patchs ?*

C'est le **Set Cover Problem** : etant donne un univers U d'elements et une collection de sous-ensembles, trouver la plus petite sous-collection qui couvre tout U.

**Garantie :** Ton algorithme glouton doit retourner au plus **ln(n) x OPT** ensembles.

---

#### Algorithme 3 : `tsp_mst_approx` — Trajectoire de Retour

*Apollo 13 ne peut pas rentrer directement. Il doit contourner la Lune, utiliser sa gravite comme fronde, et effectuer des corrections de trajectoire a des points precis. Chaque point de correction consomme du carburant precieux. Comment visiter tous les checkpoints et revenir au point de depart avec la DISTANCE MINIMALE ?*

C'est le **Travelling Salesman Problem (TSP)** pour les graphes metriques. En utilisant un arbre couvrant minimal (MST), on peut construire une tournee.

**Garantie :** Ton algorithme doit retourner une tournee d'au plus **2 x OPT** de longueur.

---

#### Algorithme 4 : `knapsack_fptas` — Chargement du Module

*Le LEM a une capacite de charge limitee. Chaque objet (eau, nourriture, equipement medical, outils de reparation) a un poids et une valeur de survie. Comment maximiser la valeur totale sans depasser la capacite ?*

C'est le **Knapsack Problem**. Le FPTAS (Fully Polynomial-Time Approximation Scheme) permet d'obtenir une solution (1-epsilon)-optimale en temps polynomial en n et 1/epsilon.

**Garantie :** Pour tout epsilon > 0, retourner une valeur >= **(1 - epsilon) x OPT**.

---

#### Algorithme 5 : `bin_packing_ffd` — Rangement des Supplies

*Les compartiments du LEM ont chacun une capacite fixe. Les supplies ont des tailles variees. Comment ranger TOUS les items dans le MINIMUM de compartiments ?*

C'est le **Bin Packing Problem**. L'algorithme First Fit Decreasing (FFD) trie les items par taille decroissante et les place dans le premier bin qui peut les accueillir.

**Garantie :** FFD utilise au plus **(11/9) x OPT + 6/9** bins.

---

**Section 1.2.2 : Version Academique**

#### Algorithmes d'Approximation : Garanties Theoriques pour Problemes NP-difficiles

Les problemes NP-difficiles ne peuvent probablement pas etre resolus optimalement en temps polynomial. Les algorithmes d'approximation offrent un compromis : une solution sous-optimale mais avec une **garantie de qualite** prouvee mathematiquement.

**Definitions :**
- **Ratio d'approximation** rho(n) : Pour tout input de taille n, l'algorithme retourne une solution de cout C tel que max(C/OPT, OPT/C) <= rho(n)
- **PTAS** : Pour tout epsilon > 0, algorithme en temps polynomial en n (mais pas forcement en 1/epsilon)
- **FPTAS** : Temps polynomial en n ET en 1/epsilon

**Les 5 algorithmes a implementer :**

| Algorithme | Probleme | Ratio | Complexite |
|------------|----------|-------|------------|
| Vertex Cover 2-approx | Minimisation | 2 | O(V + E) |
| Set Cover Greedy | Minimisation | O(ln n) | O(n * m) |
| TSP-MST | Minimisation (metrique) | 2 | O(n^2 log n) |
| Knapsack FPTAS | Maximisation | 1 - epsilon | O(n^3 / epsilon) |
| Bin Packing FFD | Minimisation | 11/9 + O(1) | O(n log n) |

---

### 1.3 Prototypes

#### Rust Edition 2024

```rust
// vertex_cover.rs
use std::collections::HashSet;

/// Graphe non-oriente represente par liste d'adjacence
pub struct Graph {
    pub adj: Vec<HashSet<usize>>,
    pub num_vertices: usize,
}

/// Retourne un vertex cover de taille <= 2 * OPT
/// Garantie : couvre toutes les aretes
pub fn vertex_cover_2approx(graph: &Graph) -> HashSet<usize>;


// set_cover.rs
use std::collections::HashSet;

/// Retourne les indices des ensembles selectionnes
/// Garantie : couvre tout l'univers, taille <= ln(|U|) * OPT
pub fn set_cover_greedy(
    universe: &HashSet<u32>,
    sets: &[HashSet<u32>]
) -> Result<Vec<usize>, &'static str>;


// tsp_approx.rs
/// Matrice de distances (symetrique, inegalite triangulaire)
pub type DistanceMatrix = Vec<Vec<f64>>;

/// Retourne l'ordre de visite des villes (tournee)
/// Garantie : longueur <= 2 * OPT
pub fn tsp_mst_approx(distances: &DistanceMatrix) -> Vec<usize>;


// knapsack_fptas.rs
/// Item avec poids et valeur
#[derive(Clone, Debug)]
pub struct Item {
    pub weight: u64,
    pub value: u64,
}

/// Retourne les indices des items selectionnes
/// Garantie : valeur >= (1 - epsilon) * OPT
pub fn knapsack_fptas(
    items: &[Item],
    capacity: u64,
    epsilon: f64
) -> Vec<usize>;


// bin_packing.rs
/// Retourne le nombre de bins utilises
/// Garantie : bins <= (11/9) * OPT + 1
pub fn bin_packing_ffd(items: &[f64], bin_capacity: f64) -> usize;
```

#### C (C17)

```c
// vertex_cover.h
#ifndef VERTEX_COVER_H
#define VERTEX_COVER_H

#include <stddef.h>
#include <stdbool.h>

typedef struct {
    size_t** adj;           // Liste d'adjacence
    size_t* adj_sizes;      // Taille de chaque liste
    size_t num_vertices;
} Graph;

typedef struct {
    size_t* vertices;       // Sommets dans le cover
    size_t size;
} VertexCover;

// Retourne un vertex cover, appelant doit free le resultat
VertexCover* vertex_cover_2approx(const Graph* graph);
void free_vertex_cover(VertexCover* cover);

#endif


// set_cover.h
#ifndef SET_COVER_H
#define SET_COVER_H

#include <stddef.h>
#include <stdint.h>

typedef struct {
    uint32_t* elements;
    size_t size;
} Set;

typedef struct {
    size_t* set_indices;
    size_t num_sets;
} SetCoverResult;

// universe_size = taille de l'univers {0, 1, ..., universe_size-1}
SetCoverResult* set_cover_greedy(
    size_t universe_size,
    const Set* sets,
    size_t num_sets
);
void free_set_cover_result(SetCoverResult* result);

#endif


// tsp_approx.h
#ifndef TSP_APPROX_H
#define TSP_APPROX_H

#include <stddef.h>

typedef struct {
    size_t* tour;           // Ordre de visite
    size_t num_cities;
    double total_distance;
} TSPTour;

// distances est une matrice num_cities x num_cities
TSPTour* tsp_mst_approx(const double** distances, size_t num_cities);
void free_tsp_tour(TSPTour* tour);

#endif


// knapsack_fptas.h
#ifndef KNAPSACK_FPTAS_H
#define KNAPSACK_FPTAS_H

#include <stddef.h>
#include <stdint.h>

typedef struct {
    uint64_t weight;
    uint64_t value;
} KnapsackItem;

typedef struct {
    size_t* selected_indices;
    size_t num_selected;
    uint64_t total_value;
    uint64_t total_weight;
} KnapsackResult;

KnapsackResult* knapsack_fptas(
    const KnapsackItem* items,
    size_t num_items,
    uint64_t capacity,
    double epsilon
);
void free_knapsack_result(KnapsackResult* result);

#endif


// bin_packing.h
#ifndef BIN_PACKING_H
#define BIN_PACKING_H

#include <stddef.h>

// Retourne le nombre de bins utilises
size_t bin_packing_ffd(const double* items, size_t num_items, double bin_capacity);

#endif
```

---

## SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Histoire d'Apollo 13

Le 13 avril 1970, a 21h07 (heure de Houston), un reservoir d'oxygene explose a bord d'Apollo 13, a 321 860 km de la Terre. Ce qui devait etre la troisieme mission lunaire habitee devient la plus grande operation de sauvetage spatiale de l'histoire.

Les ingenieurs du Mission Control Center ont du resoudre des dizaines de problemes d'optimisation en temps reel :
- **Allocation des ressources** : Comment repartir l'oxygene, l'eau et l'electricite sur 4 jours avec des reserves prevues pour 2 ?
- **Trajectoire** : Quelle sequence de manoeuvres pour revenir sur Terre avec le minimum de carburant ?
- **CO2** : Comment adapter les filtres carres du module de commande aux ouvertures rondes du LEM ?

Chacun de ces problemes est NP-difficile dans sa forme generale. Les ingenieurs ont utilise intuitivement des heuristiques qui se sont revelees etre des algorithmes d'approximation.

### 2.2 Pourquoi les Algorithmes d'Approximation ?

**Le dilemme de l'optimisation :**
```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   PROBLEME NP-DIFFICILE                                         │
│   ┌───────────────────────────────────────────────────────┐     │
│   │                                                       │     │
│   │   Option 1 : Solution OPTIMALE                        │     │
│   │   → Temps exponentiel O(2^n)                          │     │
│   │   → Inutilisable pour n > 30                          │     │
│   │                                                       │     │
│   │   Option 2 : Heuristique RAPIDE                       │     │
│   │   → Temps polynomial                                  │     │
│   │   → AUCUNE garantie de qualite                        │     │
│   │                                                       │     │
│   │   Option 3 : APPROXIMATION                            │     │
│   │   → Temps polynomial                                  │     │
│   │   → GARANTIE mathematique : solution <= rho x OPT     │     │
│   │                                                       │     │
│   └───────────────────────────────────────────────────────┘     │
│                                                                 │
│   "The best of both worlds"                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Les Garanties Mathematiques

| Algorithme | Ratio | Signification |
|------------|-------|---------------|
| Vertex Cover 2-approx | 2 | Au pire 2x la solution optimale |
| Set Cover Greedy | ln(n) | Au pire ln(n)x, mais souvent bien mieux |
| TSP-MST | 2 | Pour graphes metriques uniquement |
| Knapsack FPTAS | 1/(1-epsilon) | Arbitrairement proche de l'optimal |
| Bin Packing FFD | 11/9 | Environ 22% de plus que l'optimal |

---

## SECTION 2.5 : DANS LA VRAIE VIE

### Qui utilise ces algorithmes ?

| Metier | Algorithme | Cas d'usage |
|--------|------------|-------------|
| **DevOps / SRE** | Vertex Cover | Placement minimal de moniteurs dans un reseau |
| **Data Scientist** | Set Cover | Selection de features, echantillonnage representatif |
| **Logisticien** | TSP, Bin Packing | Tournees de livraison, chargement de camions |
| **Game Developer** | TSP, Knapsack | Pathfinding NPC, systemes de loot |
| **Network Engineer** | Vertex Cover, Set Cover | Placement de routeurs, couverture WiFi |
| **Bioinformaticien** | Set Cover | Conception de sondes ADN |
| **Cloud Architect** | Bin Packing | Allocation de VMs sur serveurs physiques |

### Exemple concret : Amazon Delivery

Amazon utilise des variantes de TSP et Bin Packing pour :
1. **Bin Packing** : Combien de camions pour les colis du jour ?
2. **TSP** : Quelle route pour chaque camion ?
3. **Set Cover** : Quels entrepots couvrent quelles zones ?

Le temps de calcul doit etre de quelques secondes, pas de minutes. Les algorithmes d'approximation sont la seule option viable.

---

## SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

#### Rust

```bash
$ ls
vertex_cover.rs  set_cover.rs  tsp_approx.rs  knapsack_fptas.rs  bin_packing.rs  main.rs  Cargo.toml

$ cargo build --release
   Compiling apollo13_approx v0.1.0
    Finished release [optimized] target(s) in 2.34s

$ cargo run --release
=== APOLLO 13 APPROXIMATION SUITE ===

[TEST 1] Vertex Cover 2-approx
Graph: 6 vertices, 7 edges
Cover found: {0, 2, 4} (size: 3)
OPT lower bound: 2, Ratio: 1.5 <= 2 ✓

[TEST 2] Set Cover Greedy
Universe: {0, 1, 2, 3, 4, 5, 6, 7}
Sets selected: [0, 2, 4] (3 sets)
All elements covered: ✓

[TEST 3] TSP MST Approximation
Cities: 5
Tour: [0, 1, 3, 4, 2, 0]
Tour length: 18.5
MST lower bound: 10.2, Ratio: 1.81 <= 2 ✓

[TEST 4] Knapsack FPTAS (epsilon=0.1)
Items: 10, Capacity: 50
Selected: [0, 2, 5, 7]
Total value: 142, Total weight: 48
(1-epsilon)*OPT estimate: 138, Achieved: 142 >= 138 ✓

[TEST 5] Bin Packing FFD
Items: 15, Bin capacity: 1.0
Bins used: 7
Lower bound (sum/capacity): 6, Ratio: 1.17 <= 11/9 ✓

All tests passed! Houston, we have a solution!
```

#### C

```bash
$ ls
vertex_cover.c  vertex_cover.h  set_cover.c  set_cover.h  tsp_approx.c  tsp_approx.h  knapsack_fptas.c  knapsack_fptas.h  bin_packing.c  bin_packing.h  main.c

$ gcc -std=c17 -Wall -Wextra -Werror -O2 *.c -o apollo13 -lm

$ ./apollo13
=== APOLLO 13 APPROXIMATION SUITE ===

[TEST 1] Vertex Cover 2-approx
Graph: 6 vertices, 7 edges
Cover found: 3 vertices
OPT lower bound: 2, Ratio: 1.5 <= 2 OK

[TEST 2] Set Cover Greedy
Universe size: 8
Sets selected: 3
All elements covered: OK

[TEST 3] TSP MST Approximation
Cities: 5
Tour length: 18.50
Ratio: 1.81 <= 2 OK

[TEST 4] Knapsack FPTAS (epsilon=0.1)
Total value: 142
Achieved >= (1-epsilon)*OPT: OK

[TEST 5] Bin Packing FFD
Bins used: 7
Ratio: 1.17 <= 1.22 OK

All tests passed!
Memory: No leaks detected (valgrind clean)
```

---

## SECTION 3.1 : BONUS AVANCE (OPTIONNEL)

**Difficulte Bonus :**
★★★★★★★★★☆ (9/10)

**Recompense :**
XP x3

**Time Complexity attendue :**
Variable selon optimisation

**Space Complexity attendue :**
Optimisee

**Domaines Bonus :**
`Algo, DP, Probas`

### 3.1.1 Consigne Bonus

#### "FAILURE IS STILL NOT AN OPTION... BUT NOW WE OPTIMIZE HARDER"

*Les algorithmes de base ont sauve l'equipage. Mais Ken Mattingly veut faire mieux. Chaque pourcentage d'amelioration compte quand des vies sont en jeu.*

**Ta mission bonus :**

Implementer des variantes optimisees avec de meilleures garanties :

1. **Vertex Cover Weighted** : Chaque sommet a un cout, minimiser le cout total (pas juste le nombre)

2. **Set Cover Primal-Dual** : Utiliser la dualite LP pour obtenir une meilleure constante

3. **TSP Christofides-style** : Ameliorer le ratio de 2 a 1.5 en ajoutant un matching parfait

4. **Knapsack Branch & Bound avec FPTAS pruning** : Hybrid qui trouve souvent l'optimal

5. **Bin Packing Best Fit Decreasing** : Legere amelioration pratique sur FFD

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  Vertex Cover : ratio < 2 sur graphes   │
│                 ponderes                │
│  TSP : ratio <= 1.5 (metrique)          │
│  Knapsack : trouver OPT exact si        │
│             possible en temps raisonnable│
│  Bin Packing : ameliorer pratiquement   │
│                le nombre de bins        │
└─────────────────────────────────────────┘
```

### 3.1.2 Prototypes Bonus

```rust
// Rust Edition 2024

pub fn vertex_cover_weighted(
    graph: &Graph,
    weights: &[f64]
) -> (HashSet<usize>, f64);  // (cover, total_weight)

pub fn tsp_christofides_approx(
    distances: &DistanceMatrix
) -> Vec<usize>;  // Ratio 1.5

pub fn knapsack_branch_bound(
    items: &[Item],
    capacity: u64
) -> (Vec<usize>, u64);  // Essaie de trouver OPT

pub fn bin_packing_bfd(
    items: &[f64],
    bin_capacity: f64
) -> (usize, Vec<Vec<usize>>);  // (num_bins, assignment)
```

### 3.1.3 Ce qui change par rapport a l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Vertex Cover | Non-pondere, ratio 2 | Pondere, meilleure qualite |
| TSP | Ratio 2 | Ratio 1.5 (Christofides) |
| Knapsack | FPTAS (1-eps) | Branch&Bound + optimal possible |
| Bin Packing | FFD simple | BFD + tracking des assignments |
| Complexite | Standard | Optimisee pour cas pratiques |

---

## SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette (tableau des tests)

| Test | Input | Expected Output | Points | Type |
|------|-------|-----------------|--------|------|
| VC_001 | Triangle graph | Cover size <= 2*OPT | 5 | Ratio |
| VC_002 | Star graph (n=10) | Cover = {center} | 5 | Optimal |
| VC_003 | Complete bipartite K(3,3) | Size <= 6 | 5 | Ratio |
| VC_004 | Empty graph | Empty cover | 3 | Edge |
| VC_005 | Null graph | Error/Empty | 2 | Safety |
| SC_001 | U={1..5}, sets covering all | Covers all | 5 | Correctness |
| SC_002 | Disjoint sets | All sets needed | 5 | Greedy |
| SC_003 | One set covers all | 1 set selected | 5 | Optimal |
| SC_004 | Empty universe | Empty result | 3 | Edge |
| SC_005 | Impossible cover | Error | 2 | Safety |
| TSP_001 | 4 cities square | Tour length <= 2*OPT | 5 | Ratio |
| TSP_002 | Collinear cities | Optimal possible | 5 | Special |
| TSP_003 | Single city | Tour = [0] | 3 | Edge |
| TSP_004 | Two cities | Tour = [0,1,0] | 3 | Edge |
| TSP_005 | Non-metric (should still work) | Valid tour | 4 | Robustness |
| KS_001 | Standard knapsack | Value >= (1-eps)*OPT | 5 | FPTAS |
| KS_002 | All items fit | All selected | 5 | Optimal |
| KS_003 | No items fit | Empty selection | 3 | Edge |
| KS_004 | eps=0.5 (loose) | Fast, >= 0.5*OPT | 4 | Epsilon |
| KS_005 | eps=0.01 (tight) | Slow, >= 0.99*OPT | 3 | Epsilon |
| BP_001 | Items sum to capacity | 1 bin | 5 | Optimal |
| BP_002 | Uniform items | Predictable | 5 | Pattern |
| BP_003 | Decreasing sizes | FFD shines | 5 | FFD |
| BP_004 | All items = 0.5+eps | 1 per bin | 4 | Edge |
| BP_005 | Empty items | 0 bins | 3 | Edge |

**Total : 100 points**

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <assert.h>

#include "vertex_cover.h"
#include "set_cover.h"
#include "tsp_approx.h"
#include "knapsack_fptas.h"
#include "bin_packing.h"

// Helper: create simple graph
Graph* create_triangle_graph() {
    Graph* g = malloc(sizeof(Graph));
    g->num_vertices = 3;
    g->adj = malloc(3 * sizeof(size_t*));
    g->adj_sizes = malloc(3 * sizeof(size_t));

    // 0-1, 1-2, 2-0
    g->adj[0] = malloc(2 * sizeof(size_t)); g->adj[0][0] = 1; g->adj[0][1] = 2; g->adj_sizes[0] = 2;
    g->adj[1] = malloc(2 * sizeof(size_t)); g->adj[1][0] = 0; g->adj[1][1] = 2; g->adj_sizes[1] = 2;
    g->adj[2] = malloc(2 * sizeof(size_t)); g->adj[2][0] = 0; g->adj[2][1] = 1; g->adj_sizes[2] = 2;

    return g;
}

void free_graph(Graph* g) {
    for (size_t i = 0; i < g->num_vertices; i++) {
        free(g->adj[i]);
    }
    free(g->adj);
    free(g->adj_sizes);
    free(g);
}

// Test Vertex Cover
int test_vertex_cover() {
    printf("[TEST] Vertex Cover 2-approx\n");

    Graph* g = create_triangle_graph();
    VertexCover* cover = vertex_cover_2approx(g);

    // Triangle optimal = 2, so cover should be <= 4
    int passed = (cover != NULL && cover->size <= 4 && cover->size >= 2);

    printf("  Cover size: %zu (OPT=2, max allowed=4)\n", cover ? cover->size : 0);
    printf("  Result: %s\n", passed ? "PASS" : "FAIL");

    free_vertex_cover(cover);
    free_graph(g);

    return passed;
}

// Test Set Cover
int test_set_cover() {
    printf("[TEST] Set Cover Greedy\n");

    // Universe = {0, 1, 2, 3}
    // Set 0 = {0, 1}, Set 1 = {2, 3}, Set 2 = {0, 2}
    Set sets[3];
    uint32_t s0[] = {0, 1}; sets[0].elements = s0; sets[0].size = 2;
    uint32_t s1[] = {2, 3}; sets[1].elements = s1; sets[1].size = 2;
    uint32_t s2[] = {0, 2}; sets[2].elements = s2; sets[2].size = 2;

    SetCoverResult* result = set_cover_greedy(4, sets, 3);

    // Optimal: {0, 1} covers all with 2 sets
    int passed = (result != NULL && result->num_sets <= 3);

    printf("  Sets selected: %zu\n", result ? result->num_sets : 0);
    printf("  Result: %s\n", passed ? "PASS" : "FAIL");

    free_set_cover_result(result);

    return passed;
}

// Test TSP
int test_tsp() {
    printf("[TEST] TSP MST Approximation\n");

    // 4 cities in a square: (0,0), (0,1), (1,1), (1,0)
    double row0[] = {0.0, 1.0, 1.414, 1.0};
    double row1[] = {1.0, 0.0, 1.0, 1.414};
    double row2[] = {1.414, 1.0, 0.0, 1.0};
    double row3[] = {1.0, 1.414, 1.0, 0.0};
    const double* distances[] = {row0, row1, row2, row3};

    TSPTour* tour = tsp_mst_approx(distances, 4);

    // Optimal tour = 4.0, so we should get <= 8.0
    int passed = (tour != NULL && tour->total_distance <= 8.0);

    printf("  Tour length: %.2f (OPT=4.0, max=8.0)\n", tour ? tour->total_distance : 0);
    printf("  Result: %s\n", passed ? "PASS" : "FAIL");

    free_tsp_tour(tour);

    return passed;
}

// Test Knapsack FPTAS
int test_knapsack() {
    printf("[TEST] Knapsack FPTAS (epsilon=0.1)\n");

    KnapsackItem items[] = {
        {10, 60}, {20, 100}, {30, 120}
    };

    KnapsackResult* result = knapsack_fptas(items, 3, 50, 0.1);

    // Optimal = 220 (items 1 and 2), so we need >= 198
    int passed = (result != NULL && result->total_value >= 198);

    printf("  Total value: %lu (target >= 198)\n", result ? (unsigned long)result->total_value : 0);
    printf("  Result: %s\n", passed ? "PASS" : "FAIL");

    free_knapsack_result(result);

    return passed;
}

// Test Bin Packing
int test_bin_packing() {
    printf("[TEST] Bin Packing FFD\n");

    double items[] = {0.5, 0.7, 0.3, 0.2, 0.4, 0.1, 0.8};

    size_t bins = bin_packing_ffd(items, 7, 1.0);

    // Sum = 3.0, so OPT >= 3. FFD should give <= 4
    int passed = (bins >= 3 && bins <= 5);

    printf("  Bins used: %zu (expected 3-5)\n", bins);
    printf("  Result: %s\n", passed ? "PASS" : "FAIL");

    return passed;
}

int main() {
    printf("=== APOLLO 13 APPROXIMATION SUITE TESTS ===\n\n");

    int total = 0;
    int passed = 0;

    total++; if (test_vertex_cover()) passed++;
    total++; if (test_set_cover()) passed++;
    total++; if (test_tsp()) passed++;
    total++; if (test_knapsack()) passed++;
    total++; if (test_bin_packing()) passed++;

    printf("\n=== RESULTS: %d/%d tests passed ===\n", passed, total);

    return (passed == total) ? 0 : 1;
}
```

### 4.3 Solution de reference

#### Rust - vertex_cover.rs

```rust
use std::collections::HashSet;

pub struct Graph {
    pub adj: Vec<HashSet<usize>>,
    pub num_vertices: usize,
}

impl Graph {
    pub fn new(n: usize) -> Self {
        Graph {
            adj: vec![HashSet::new(); n],
            num_vertices: n,
        }
    }

    pub fn add_edge(&mut self, u: usize, v: usize) {
        if u < self.num_vertices && v < self.num_vertices {
            self.adj[u].insert(v);
            self.adj[v].insert(u);
        }
    }
}

/// 2-approximation algorithm for Vertex Cover
/// Uses the maximal matching approach
pub fn vertex_cover_2approx(graph: &Graph) -> HashSet<usize> {
    if graph.num_vertices == 0 {
        return HashSet::new();
    }

    let mut cover = HashSet::new();
    let mut covered_edges: HashSet<(usize, usize)> = HashSet::new();

    // Build edge set
    let mut edges: Vec<(usize, usize)> = Vec::new();
    for u in 0..graph.num_vertices {
        for &v in &graph.adj[u] {
            if u < v {
                edges.push((u, v));
            }
        }
    }

    // Greedy maximal matching
    for (u, v) in edges {
        let edge_key = (u.min(v), u.max(v));

        // If this edge is not covered, add both endpoints
        if !covered_edges.contains(&edge_key) {
            cover.insert(u);
            cover.insert(v);

            // Mark all edges incident to u and v as covered
            for &w in &graph.adj[u] {
                covered_edges.insert((u.min(w), u.max(w)));
            }
            for &w in &graph.adj[v] {
                covered_edges.insert((v.min(w), v.max(w)));
            }
        }
    }

    cover
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_triangle() {
        let mut g = Graph::new(3);
        g.add_edge(0, 1);
        g.add_edge(1, 2);
        g.add_edge(2, 0);

        let cover = vertex_cover_2approx(&g);
        assert!(cover.len() <= 4); // 2 * OPT where OPT = 2
        assert!(cover.len() >= 2);
    }

    #[test]
    fn test_empty_graph() {
        let g = Graph::new(0);
        let cover = vertex_cover_2approx(&g);
        assert!(cover.is_empty());
    }
}
```

#### Rust - set_cover.rs

```rust
use std::collections::HashSet;

/// Greedy Set Cover algorithm
/// Returns indices of selected sets, or error if impossible
pub fn set_cover_greedy(
    universe: &HashSet<u32>,
    sets: &[HashSet<u32>]
) -> Result<Vec<usize>, &'static str> {
    if universe.is_empty() {
        return Ok(vec![]);
    }

    if sets.is_empty() {
        return Err("No sets provided to cover universe");
    }

    let mut uncovered: HashSet<u32> = universe.clone();
    let mut selected: Vec<usize> = Vec::new();
    let mut used: HashSet<usize> = HashSet::new();

    while !uncovered.is_empty() {
        // Find set that covers most uncovered elements
        let mut best_idx = None;
        let mut best_count = 0;

        for (idx, set) in sets.iter().enumerate() {
            if used.contains(&idx) {
                continue;
            }

            let count = set.intersection(&uncovered).count();
            if count > best_count {
                best_count = count;
                best_idx = Some(idx);
            }
        }

        match best_idx {
            Some(idx) => {
                // Remove covered elements
                for elem in &sets[idx] {
                    uncovered.remove(elem);
                }
                selected.push(idx);
                used.insert(idx);
            }
            None => {
                return Err("Cannot cover all elements with given sets");
            }
        }
    }

    Ok(selected)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_simple_cover() {
        let universe: HashSet<u32> = [1, 2, 3, 4, 5].into_iter().collect();
        let sets = vec![
            [1, 2, 3].into_iter().collect(),
            [4, 5].into_iter().collect(),
            [1, 4].into_iter().collect(),
        ];

        let result = set_cover_greedy(&universe, &sets).unwrap();

        // Verify all elements are covered
        let mut covered = HashSet::new();
        for &idx in &result {
            covered.extend(&sets[idx]);
        }
        assert!(universe.is_subset(&covered));
    }

    #[test]
    fn test_empty_universe() {
        let universe: HashSet<u32> = HashSet::new();
        let sets: Vec<HashSet<u32>> = vec![];

        let result = set_cover_greedy(&universe, &sets).unwrap();
        assert!(result.is_empty());
    }
}
```

#### Rust - tsp_approx.rs

```rust
use std::collections::HashSet;

pub type DistanceMatrix = Vec<Vec<f64>>;

/// MST-based 2-approximation for metric TSP
pub fn tsp_mst_approx(distances: &DistanceMatrix) -> Vec<usize> {
    let n = distances.len();

    if n == 0 {
        return vec![];
    }
    if n == 1 {
        return vec![0];
    }
    if n == 2 {
        return vec![0, 1, 0];
    }

    // Build MST using Prim's algorithm
    let mst = build_mst(distances);

    // DFS preorder traversal of MST
    let mut tour = Vec::new();
    let mut visited = vec![false; n];
    dfs_preorder(&mst, 0, &mut visited, &mut tour);

    // Return to start
    tour.push(0);

    tour
}

fn build_mst(distances: &DistanceMatrix) -> Vec<Vec<usize>> {
    let n = distances.len();
    let mut mst: Vec<Vec<usize>> = vec![Vec::new(); n];
    let mut in_mst = vec![false; n];
    let mut key = vec![f64::INFINITY; n];
    let mut parent = vec![None; n];

    key[0] = 0.0;

    for _ in 0..n {
        // Find minimum key vertex not in MST
        let mut min_key = f64::INFINITY;
        let mut u = 0;

        for v in 0..n {
            if !in_mst[v] && key[v] < min_key {
                min_key = key[v];
                u = v;
            }
        }

        in_mst[u] = true;

        // Add edge to MST
        if let Some(p) = parent[u] {
            mst[p].push(u);
            mst[u].push(p);
        }

        // Update keys of adjacent vertices
        for v in 0..n {
            if !in_mst[v] && distances[u][v] < key[v] {
                key[v] = distances[u][v];
                parent[v] = Some(u);
            }
        }
    }

    mst
}

fn dfs_preorder(adj: &[Vec<usize>], node: usize, visited: &mut [bool], tour: &mut Vec<usize>) {
    visited[node] = true;
    tour.push(node);

    for &neighbor in &adj[node] {
        if !visited[neighbor] {
            dfs_preorder(adj, neighbor, visited, tour);
        }
    }
}

/// Calculate total tour length
pub fn tour_length(tour: &[usize], distances: &DistanceMatrix) -> f64 {
    if tour.len() < 2 {
        return 0.0;
    }

    let mut total = 0.0;
    for i in 0..tour.len() - 1 {
        total += distances[tour[i]][tour[i + 1]];
    }
    total
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_square_cities() {
        // Square with side 1
        let distances = vec![
            vec![0.0, 1.0, 1.414, 1.0],
            vec![1.0, 0.0, 1.0, 1.414],
            vec![1.414, 1.0, 0.0, 1.0],
            vec![1.0, 1.414, 1.0, 0.0],
        ];

        let tour = tsp_mst_approx(&distances);
        let length = tour_length(&tour, &distances);

        // Optimal = 4.0, so length <= 8.0
        assert!(length <= 8.0);
    }
}
```

#### Rust - knapsack_fptas.rs

```rust
#[derive(Clone, Debug)]
pub struct Item {
    pub weight: u64,
    pub value: u64,
}

/// FPTAS for 0/1 Knapsack
/// Returns indices of selected items
pub fn knapsack_fptas(items: &[Item], capacity: u64, epsilon: f64) -> Vec<usize> {
    if items.is_empty() || capacity == 0 || epsilon <= 0.0 {
        return vec![];
    }

    let n = items.len();

    // Find max value
    let max_value = items.iter().map(|i| i.value).max().unwrap_or(0);

    if max_value == 0 {
        return vec![];
    }

    // Scaling factor
    let k = (epsilon * max_value as f64) / (n as f64);

    if k < 1.0 {
        // K too small, use exact DP
        return knapsack_exact(items, capacity);
    }

    // Scale down values
    let scaled_items: Vec<Item> = items
        .iter()
        .map(|item| Item {
            weight: item.weight,
            value: (item.value as f64 / k).floor() as u64,
        })
        .collect();

    // Solve scaled problem
    let scaled_result = knapsack_exact(&scaled_items, capacity);

    scaled_result
}

fn knapsack_exact(items: &[Item], capacity: u64) -> Vec<usize> {
    let n = items.len();
    let cap = capacity as usize;

    // DP table: dp[i][w] = max value using first i items with capacity w
    let mut dp = vec![vec![0u64; cap + 1]; n + 1];

    for i in 1..=n {
        let item = &items[i - 1];
        for w in 0..=cap {
            dp[i][w] = dp[i - 1][w];

            if item.weight as usize <= w {
                let with_item = dp[i - 1][w - item.weight as usize] + item.value;
                if with_item > dp[i][w] {
                    dp[i][w] = with_item;
                }
            }
        }
    }

    // Backtrack to find selected items
    let mut selected = Vec::new();
    let mut w = cap;

    for i in (1..=n).rev() {
        if dp[i][w] != dp[i - 1][w] {
            selected.push(i - 1);
            w -= items[i - 1].weight as usize;
        }
    }

    selected.reverse();
    selected
}

/// Calculate total value of selected items
pub fn total_value(items: &[Item], selected: &[usize]) -> u64 {
    selected.iter().map(|&i| items[i].value).sum()
}

/// Calculate total weight of selected items
pub fn total_weight(items: &[Item], selected: &[usize]) -> u64 {
    selected.iter().map(|&i| items[i].weight).sum()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_basic_knapsack() {
        let items = vec![
            Item { weight: 10, value: 60 },
            Item { weight: 20, value: 100 },
            Item { weight: 30, value: 120 },
        ];

        let selected = knapsack_fptas(&items, 50, 0.1);
        let value = total_value(&items, &selected);
        let weight = total_weight(&items, &selected);

        // Optimal = 220 (items 1 and 2)
        // (1 - 0.1) * 220 = 198
        assert!(value >= 198);
        assert!(weight <= 50);
    }
}
```

#### Rust - bin_packing.rs

```rust
/// First Fit Decreasing algorithm for Bin Packing
pub fn bin_packing_ffd(items: &[f64], bin_capacity: f64) -> usize {
    if items.is_empty() || bin_capacity <= 0.0 {
        return 0;
    }

    // Sort items in decreasing order
    let mut sorted_items: Vec<f64> = items.to_vec();
    sorted_items.sort_by(|a, b| b.partial_cmp(a).unwrap_or(std::cmp::Ordering::Equal));

    // Filter out items that don't fit
    let valid_items: Vec<f64> = sorted_items
        .into_iter()
        .filter(|&x| x <= bin_capacity && x > 0.0)
        .collect();

    if valid_items.is_empty() {
        return 0;
    }

    // Track remaining capacity of each bin
    let mut bins: Vec<f64> = Vec::new();

    for item in valid_items {
        // Find first bin that can fit this item
        let mut placed = false;

        for bin_remaining in &mut bins {
            if *bin_remaining >= item {
                *bin_remaining -= item;
                placed = true;
                break;
            }
        }

        // If no bin can fit, create new bin
        if !placed {
            bins.push(bin_capacity - item);
        }
    }

    bins.len()
}

/// Returns (num_bins, assignment) where assignment[i] = bin index for item i
pub fn bin_packing_ffd_detailed(items: &[f64], bin_capacity: f64) -> (usize, Vec<Option<usize>>) {
    if items.is_empty() || bin_capacity <= 0.0 {
        return (0, vec![]);
    }

    // Create indexed items and sort
    let mut indexed: Vec<(usize, f64)> = items.iter().copied().enumerate().collect();
    indexed.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap_or(std::cmp::Ordering::Equal));

    let mut bins: Vec<f64> = Vec::new();
    let mut assignment: Vec<Option<usize>> = vec![None; items.len()];

    for (orig_idx, item) in indexed {
        if item > bin_capacity || item <= 0.0 {
            continue;
        }

        // Find first fit
        let mut placed = false;
        for (bin_idx, bin_remaining) in bins.iter_mut().enumerate() {
            if *bin_remaining >= item {
                *bin_remaining -= item;
                assignment[orig_idx] = Some(bin_idx);
                placed = true;
                break;
            }
        }

        if !placed {
            assignment[orig_idx] = Some(bins.len());
            bins.push(bin_capacity - item);
        }
    }

    (bins.len(), assignment)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_ffd_basic() {
        let items = vec![0.5, 0.7, 0.3, 0.2, 0.4, 0.1, 0.8];
        let bins = bin_packing_ffd(&items, 1.0);

        // Sum = 3.0, so at least 3 bins needed
        // FFD should use at most 4-5 bins
        assert!(bins >= 3);
        assert!(bins <= 5);
    }

    #[test]
    fn test_ffd_perfect_fit() {
        let items = vec![0.5, 0.5, 0.5, 0.5];
        let bins = bin_packing_ffd(&items, 1.0);

        // Optimal = 2 bins
        assert_eq!(bins, 2);
    }

    #[test]
    fn test_empty() {
        let items: Vec<f64> = vec![];
        let bins = bin_packing_ffd(&items, 1.0);
        assert_eq!(bins, 0);
    }
}
```

### 4.4 Solutions C de reference

#### C - vertex_cover.c

```c
#include "vertex_cover.h"
#include <stdlib.h>
#include <stdbool.h>

VertexCover* vertex_cover_2approx(const Graph* graph) {
    if (graph == NULL || graph->num_vertices == 0) {
        VertexCover* cover = malloc(sizeof(VertexCover));
        if (cover) {
            cover->vertices = NULL;
            cover->size = 0;
        }
        return cover;
    }

    size_t n = graph->num_vertices;
    bool* in_cover = calloc(n, sizeof(bool));
    bool** edge_covered = calloc(n, sizeof(bool*));

    for (size_t i = 0; i < n; i++) {
        edge_covered[i] = calloc(n, sizeof(bool));
    }

    size_t cover_size = 0;

    // Process each edge
    for (size_t u = 0; u < n; u++) {
        for (size_t j = 0; j < graph->adj_sizes[u]; j++) {
            size_t v = graph->adj[u][j];

            if (u < v && !edge_covered[u][v]) {
                // Add both endpoints to cover
                if (!in_cover[u]) {
                    in_cover[u] = true;
                    cover_size++;
                }
                if (!in_cover[v]) {
                    in_cover[v] = true;
                    cover_size++;
                }

                // Mark all incident edges as covered
                for (size_t k = 0; k < graph->adj_sizes[u]; k++) {
                    size_t w = graph->adj[u][k];
                    size_t a = (u < w) ? u : w;
                    size_t b = (u < w) ? w : u;
                    edge_covered[a][b] = true;
                }
                for (size_t k = 0; k < graph->adj_sizes[v]; k++) {
                    size_t w = graph->adj[v][k];
                    size_t a = (v < w) ? v : w;
                    size_t b = (v < w) ? w : v;
                    edge_covered[a][b] = true;
                }
            }
        }
    }

    // Build result
    VertexCover* cover = malloc(sizeof(VertexCover));
    cover->vertices = malloc(cover_size * sizeof(size_t));
    cover->size = cover_size;

    size_t idx = 0;
    for (size_t i = 0; i < n; i++) {
        if (in_cover[i]) {
            cover->vertices[idx++] = i;
        }
    }

    // Cleanup
    free(in_cover);
    for (size_t i = 0; i < n; i++) {
        free(edge_covered[i]);
    }
    free(edge_covered);

    return cover;
}

void free_vertex_cover(VertexCover* cover) {
    if (cover) {
        free(cover->vertices);
        free(cover);
    }
}
```

#### C - bin_packing.c

```c
#include "bin_packing.h"
#include <stdlib.h>
#include <string.h>

static int compare_desc(const void* a, const void* b) {
    double diff = *(const double*)b - *(const double*)a;
    if (diff > 0) return 1;
    if (diff < 0) return -1;
    return 0;
}

size_t bin_packing_ffd(const double* items, size_t num_items, double bin_capacity) {
    if (items == NULL || num_items == 0 || bin_capacity <= 0) {
        return 0;
    }

    // Copy and sort items in decreasing order
    double* sorted = malloc(num_items * sizeof(double));
    if (!sorted) return 0;

    memcpy(sorted, items, num_items * sizeof(double));
    qsort(sorted, num_items, sizeof(double), compare_desc);

    // Track remaining capacity of each bin
    double* bins = malloc(num_items * sizeof(double));  // Max possible bins
    if (!bins) {
        free(sorted);
        return 0;
    }

    size_t num_bins = 0;

    for (size_t i = 0; i < num_items; i++) {
        double item = sorted[i];

        if (item <= 0 || item > bin_capacity) {
            continue;
        }

        // Find first bin that can fit
        bool placed = false;
        for (size_t b = 0; b < num_bins; b++) {
            if (bins[b] >= item) {
                bins[b] -= item;
                placed = true;
                break;
            }
        }

        // Create new bin if needed
        if (!placed) {
            bins[num_bins] = bin_capacity - item;
            num_bins++;
        }
    }

    free(sorted);
    free(bins);

    return num_bins;
}
```

### 4.5 Solutions refusees (avec explications)

#### Solution 1 : Vertex Cover qui retourne tous les sommets

```c
// REFUSE : Trivial, pas d'approximation
VertexCover* vertex_cover_all(const Graph* graph) {
    VertexCover* cover = malloc(sizeof(VertexCover));
    cover->size = graph->num_vertices;
    cover->vertices = malloc(cover->size * sizeof(size_t));
    for (size_t i = 0; i < cover->size; i++) {
        cover->vertices[i] = i;
    }
    return cover;
}
// POURQUOI REFUSE : Retourne n sommets au lieu de ~2*OPT
// Le ratio devient n/OPT au lieu de 2, inacceptable
```

#### Solution 2 : Set Cover sans verification de couverture

```rust
// REFUSE : Ne garantit pas la couverture complete
pub fn set_cover_bad(sets: &[HashSet<u32>]) -> Vec<usize> {
    // Prend juste les 3 premiers sets
    (0..3.min(sets.len())).collect()
}
// POURQUOI REFUSE : Ne verifie pas que l'univers est couvert
// Peut retourner une couverture incomplete
```

#### Solution 3 : TSP qui ne retourne pas au depart

```rust
// REFUSE : Ce n'est pas une tournee valide
pub fn tsp_no_return(distances: &DistanceMatrix) -> Vec<usize> {
    let n = distances.len();
    (0..n).collect()  // Visite mais ne revient pas
}
// POURQUOI REFUSE : Une tournee TSP doit revenir au point de depart
// Le dernier element doit etre 0 (ou le point de depart)
```

### 4.6 Solutions Mutantes (minimum 5)

#### Mutant A (Boundary) : Vertex Cover off-by-one

```rust
// Bug: Condition de boucle incorrecte
pub fn vertex_cover_mutant_a(graph: &Graph) -> HashSet<usize> {
    let mut cover = HashSet::new();

    for u in 0..graph.num_vertices {
        for &v in &graph.adj[u] {
            // BUG: u <= v au lieu de u < v
            // Compte chaque arete deux fois!
            if u <= v {
                cover.insert(u);
                cover.insert(v);
            }
        }
    }

    cover
}
// POURQUOI C'EST FAUX : Avec u <= v, on traite (u,u) comme une arete
// et on double-compte toutes les aretes, causant des insertions redondantes
// Le resultat peut etre correct par chance, mais la logique est fausse
```

#### Mutant B (Safety) : Set Cover sans verification NULL

```rust
pub fn set_cover_mutant_b(
    universe: &HashSet<u32>,
    sets: &[HashSet<u32>]
) -> Result<Vec<usize>, &'static str> {
    // BUG: Pas de verification si universe est vide
    // BUG: Pas de verification si sets est vide

    let mut uncovered = universe.clone();
    let mut selected = Vec::new();

    while !uncovered.is_empty() {
        let mut best_idx = 0;  // BUG: Suppose qu'il y a au moins un set
        let mut best_count = 0;

        for (idx, set) in sets.iter().enumerate() {
            let count = set.intersection(&uncovered).count();
            if count > best_count {
                best_count = count;
                best_idx = idx;
            }
        }

        // BUG: Si best_count == 0, on ajoute quand meme un set inutile
        // et on boucle infiniment car uncovered ne diminue pas
        for elem in &sets[best_idx] {
            uncovered.remove(elem);
        }
        selected.push(best_idx);
    }

    Ok(selected)
}
// POURQUOI C'EST FAUX : Boucle infinie si aucun set ne couvre les elements restants
// Crash si sets est vide (acces sets[0])
```

#### Mutant C (Resource) : TSP avec fuite memoire (C)

```c
// Bug: Ne libere pas le MST
TSPTour* tsp_mutant_c(const double** distances, size_t num_cities) {
    // Alloue le MST
    size_t** mst = malloc(num_cities * sizeof(size_t*));
    size_t* mst_sizes = malloc(num_cities * sizeof(size_t));

    for (size_t i = 0; i < num_cities; i++) {
        mst[i] = malloc(num_cities * sizeof(size_t));
        mst_sizes[i] = 0;
    }

    // Construit MST avec Prim...
    build_mst(distances, num_cities, mst, mst_sizes);

    // DFS pour construire tournee
    TSPTour* tour = malloc(sizeof(TSPTour));
    tour->tour = malloc((num_cities + 1) * sizeof(size_t));
    tour->num_cities = num_cities;

    dfs_tour(mst, mst_sizes, 0, tour);

    // BUG: OUBLIE DE LIBERER mst et mst_sizes!
    // for (size_t i = 0; i < num_cities; i++) free(mst[i]);
    // free(mst);
    // free(mst_sizes);

    return tour;
}
// POURQUOI C'EST FAUX : Fuite memoire de O(n^2) a chaque appel
// Valgrind detectera : "definitely lost: X bytes"
```

#### Mutant D (Logic) : Knapsack avec mauvais scaling

```rust
pub fn knapsack_mutant_d(items: &[Item], capacity: u64, epsilon: f64) -> Vec<usize> {
    let n = items.len();
    let max_value = items.iter().map(|i| i.value).max().unwrap_or(0);

    // BUG: Divise par n^2 au lieu de n
    // Cela sur-scale les valeurs, perdant trop de precision
    let k = (epsilon * max_value as f64) / (n * n) as f64;

    let scaled_items: Vec<Item> = items
        .iter()
        .map(|item| Item {
            weight: item.weight,
            value: (item.value as f64 / k).floor() as u64,
        })
        .collect();

    knapsack_exact(&scaled_items, capacity)
}
// POURQUOI C'EST FAUX : Le scaling trop agressif cause une perte de precision
// La garantie (1-epsilon) n'est plus respectee
// Peut retourner une solution tres sous-optimale
```

#### Mutant E (Return) : Bin Packing retourne le mauvais compte

```rust
pub fn bin_packing_mutant_e(items: &[f64], bin_capacity: f64) -> usize {
    let mut sorted_items: Vec<f64> = items.to_vec();
    sorted_items.sort_by(|a, b| b.partial_cmp(a).unwrap());

    let valid_items: Vec<f64> = sorted_items
        .into_iter()
        .filter(|&x| x <= bin_capacity && x > 0.0)
        .collect();

    let mut bins: Vec<f64> = Vec::new();

    for item in &valid_items {
        let mut placed = false;
        for bin_remaining in &mut bins {
            if *bin_remaining >= *item {
                *bin_remaining -= *item;
                placed = true;
                break;
            }
        }
        if !placed {
            bins.push(bin_capacity - *item);
        }
    }

    // BUG: Retourne le nombre d'items au lieu du nombre de bins!
    valid_items.len()  // Devrait etre bins.len()
}
// POURQUOI C'EST FAUX : Retourne toujours le nombre d'items
// Ce n'est jamais le bon resultat (sauf si chaque item a son propre bin)
```
