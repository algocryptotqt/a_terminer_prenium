<thinking>
## Analyse du Concept
- Concept : Représentations de graphes (Adjacency Matrix, Adjacency List, Edge List, Grid)
- Phase demandée : 1 (Transition débutant → intermédiaire)
- Adapté ? OUI - Les représentations de graphes sont fondamentales et enseignables en un exercice

## Combo Base + Bonus
- Exercice de base : Implémenter les 3 représentations classiques (matrix, list, edges) avec conversions
- Bonus Standard : Grid Graph avec voisins 4-directions et 8-directions
- Bonus Expert : Graphe implicite pour jeu de plateau (échecs/dames)
- Bonus Génie : Représentation compressée CSR (Compressed Sparse Row)
- Palier bonus : ⚡🔥🧠
- Progression logique ? OUI

## Prérequis & Difficulté
- Prérequis réels : Vecteurs/tableaux dynamiques, structures, allocation mémoire
- Difficulté estimée : 4/10
- Cohérent avec phase ? OUI (Phase 1 = 3-5/10)

## Aspect Fun/Culture
- Contexte choisi : TRON: Legacy - The Grid
- MEME mnémotechnique : "I fight for the Users!" (Tron's catchphrase) pour la gestion des connexions
- Pourquoi c'est fun : Le monde de TRON EST littéralement un graphe - des programmes (nodes) connectés dans The Grid. Les Light Cycles circulent sur une grille, les programmes communiquent via des connexions pondérées (niveaux de sécurité). C'est une analogie PARFAITE pour les représentations de graphes.
- Score d'intelligence : 97/100 - L'analogie est profondément cohérente avec le sujet

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : Matrice non symétrique pour graphe non-dirigé - oubli d'ajouter l'arête inverse
2. Mutant B (Safety) : Pas de vérification des indices u, v < n avant accès au tableau
3. Mutant C (Resource) : Fuite mémoire dans les conversions - allocation sans libération de l'ancienne structure
4. Mutant D (Logic) : has_edge retourne true pour poids 0 au lieu de vérifier contre INF
5. Mutant E (Return) : neighbors() retourne un vecteur vide au lieu des vrais voisins (boucle jamais exécutée)

## Verdict
VALIDE - L'exercice est excellent avec une analogie TRON parfaitement adaptée aux graphes
</thinking>

---

# Exercice 1.4.0-a : tron_grid_architect

**Module :**
1.4.0 — Représentations de Graphes

**Concept :**
a — Structures de données pour graphes (Matrix, List, Edges, Grid)

**Difficulté :**
★★★★☆☆☆☆☆☆ (4/10)

**Type :**
code

**Tiers :**
1 — Concept isolé

**Langage :**
Rust Edition 2024 / C17

**Prérequis :**
- Vecteurs et tableaux dynamiques
- Structures et allocation mémoire
- Pointeurs (pour C)

**Domaines :**
Struct, MD

**Durée estimée :**
45 min

**XP Base :**
100

**Complexité :**
T2 O(n²) × S2 O(n²) pour matrice, T1 O(V+E) × S1 O(V+E) pour liste

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `tron_grid.c`, `tron_grid.h`

**Fonctions autorisées :**
- Rust : `Vec`, `Option`, standard library
- C : `malloc`, `realloc`, `free`, `memset`

**Fonctions interdites :**
- Bibliothèques de graphes externes

### 1.2 Consigne

#### 1.2.1 Version Culture Pop : TRON: Legacy - The Grid

**🎮 "The Grid. A digital frontier. I tried to picture clusters of information as they moved through the computer."**

Tu es Kevin Flynn, créateur de The Grid. Pour gérer les millions de programmes qui habitent ton monde numérique, tu dois construire différentes représentations de leurs connexions.

Dans The Grid, chaque **programme** est un nœud. Les **connexions I/O** entre programmes sont des arêtes avec un **niveau de sécurité** (poids). CLU, ton alter-ego corrompu, utilise une matrice de surveillance pour traquer TOUTES les connexions possibles. Tron utilise une liste d'adjacence plus efficace pour ne stocker que les connexions réelles.

**Ta mission :**

Implémenter trois représentations du réseau de programmes de The Grid :

1. **`TronMatrix`** (Matrice d'adjacence) : La surveillance totale de CLU
   - Stocke le poids de TOUTE connexion possible entre programmes
   - `i32::MAX` = pas de connexion (programme isolé)

2. **`TronNetwork`** (Liste d'adjacence) : Le réseau efficace de Tron
   - Stocke seulement les connexions qui existent vraiment
   - Chaque programme a sa liste de voisins avec poids

3. **`TronEdges`** (Liste d'arêtes) : Le journal des connexions
   - Liste plate de toutes les connexions (from, to, weight)
   - Utile pour les algorithmes qui parcourent toutes les arêtes

**Entrée (commune à toutes les structures) :**
- `n: usize` : nombre de programmes dans The Grid
- `add_edge(u, v, weight)` : ajoute une connexion de sécurité entre programmes u et v

**Sortie :**
- `has_edge(u, v)` : `true` si une connexion existe
- `weight(u, v)` : `Some(w)` si connexion existe, `None` sinon
- `neighbors(u)` : liste des programmes connectés à u
- `degree(u)` : nombre de connexions de u

**Contraintes :**
- 0 ≤ u, v < n
- Les poids sont des entiers signés (peuvent être négatifs pour représenter des connexions corrompues)
- Graphe dirigé par défaut (sauf méthode `add_undirected_edge`)

**Exemples :**

| Structure | Opération | Résultat | Explication |
|-----------|-----------|----------|-------------|
| `TronNetwork::new(4)` | `add_edge(0, 1, 5)` | OK | Tron (0) → Sam (1), sécurité 5 |
| `network` | `has_edge(0, 1)` | `true` | Connexion existe |
| `network` | `has_edge(1, 0)` | `false` | Dirigé: pas de retour |
| `network` | `neighbors(0)` | `[(1, 5)]` | Tron voit Sam |
| `network` | `degree(0)` | `1` | Une seule connexion sortante |

#### 1.2.2 Version Académique

Implémenter trois représentations classiques de graphes pondérés :

1. **Matrice d'adjacence** : Tableau 2D où `matrix[i][j]` contient le poids de l'arête (i,j) ou une valeur sentinelle (∞) si l'arête n'existe pas. Espace O(V²).

2. **Liste d'adjacence** : Tableau de listes où `adj[i]` contient les paires (voisin, poids) pour toutes les arêtes sortantes du sommet i. Espace O(V+E).

3. **Liste d'arêtes** : Liste de triplets (source, destination, poids) représentant chaque arête. Espace O(E).

Implémenter également les conversions entre représentations.

### 1.3 Prototype

```rust
// Rust - Edition 2024
pub mod tron_grid {
    pub const DISCONNECTED: i32 = i32::MAX;

    /// CLU's Surveillance Matrix - O(V²) space
    pub struct TronMatrix {
        matrix: Vec<Vec<i32>>,
        n: usize,
    }

    impl TronMatrix {
        pub fn new(n: usize) -> Self;
        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn add_undirected_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn has_edge(&self, u: usize, v: usize) -> bool;
        pub fn weight(&self, u: usize, v: usize) -> Option<i32>;
        pub fn neighbors(&self, u: usize) -> Vec<usize>;
        pub fn size(&self) -> usize;
    }

    /// Tron's Efficient Network - O(V+E) space
    pub struct TronNetwork {
        adj: Vec<Vec<(usize, i32)>>,
        n: usize,
        m: usize,
    }

    impl TronNetwork {
        pub fn new(n: usize) -> Self;
        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn add_undirected_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn neighbors(&self, u: usize) -> &[(usize, i32)];
        pub fn degree(&self, u: usize) -> usize;
        pub fn vertex_count(&self) -> usize;
        pub fn edge_count(&self) -> usize;
    }

    /// Edge Journal - O(E) space
    pub struct TronEdges {
        edges: Vec<(usize, usize, i32)>,
        n: usize,
    }

    impl TronEdges {
        pub fn new(n: usize) -> Self;
        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn edges(&self) -> &[(usize, usize, i32)];
        pub fn sort_by_weight(&mut self);
        pub fn vertex_count(&self) -> usize;
    }

    // Conversions - "Derezzed transformation"
    pub fn matrix_to_network(matrix: &TronMatrix) -> TronNetwork;
    pub fn network_to_matrix(network: &TronNetwork) -> TronMatrix;
    pub fn network_to_edges(network: &TronNetwork) -> TronEdges;
    pub fn edges_to_network(edges: &TronEdges) -> TronNetwork;
}
```

```c
// C17
#ifndef TRON_GRID_H
#define TRON_GRID_H

#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>

#define DISCONNECTED INT32_MAX

// Edge structure for adjacency list
typedef struct {
    size_t neighbor;
    int32_t weight;
} TronEdge;

// CLU's Surveillance Matrix
typedef struct {
    int32_t **matrix;
    size_t n;
} TronMatrix;

TronMatrix *tron_matrix_new(size_t n);
void tron_matrix_free(TronMatrix *tm);
void tron_matrix_add_edge(TronMatrix *tm, size_t u, size_t v, int32_t weight);
void tron_matrix_add_undirected(TronMatrix *tm, size_t u, size_t v, int32_t weight);
bool tron_matrix_has_edge(const TronMatrix *tm, size_t u, size_t v);
int32_t tron_matrix_weight(const TronMatrix *tm, size_t u, size_t v, bool *exists);
size_t *tron_matrix_neighbors(const TronMatrix *tm, size_t u, size_t *count);

// Tron's Network (Adjacency List)
typedef struct {
    TronEdge **adj;
    size_t *adj_sizes;
    size_t *adj_caps;
    size_t n;
    size_t m;
} TronNetwork;

TronNetwork *tron_network_new(size_t n);
void tron_network_free(TronNetwork *tn);
void tron_network_add_edge(TronNetwork *tn, size_t u, size_t v, int32_t weight);
void tron_network_add_undirected(TronNetwork *tn, size_t u, size_t v, int32_t weight);
const TronEdge *tron_network_neighbors(const TronNetwork *tn, size_t u, size_t *count);
size_t tron_network_degree(const TronNetwork *tn, size_t u);
size_t tron_network_vertex_count(const TronNetwork *tn);
size_t tron_network_edge_count(const TronNetwork *tn);

// Edge List representation
typedef struct {
    size_t from;
    size_t to;
    int32_t weight;
} TronEdgeEntry;

typedef struct {
    TronEdgeEntry *edges;
    size_t count;
    size_t capacity;
    size_t n;
} TronEdges;

TronEdges *tron_edges_new(size_t n);
void tron_edges_free(TronEdges *te);
void tron_edges_add(TronEdges *te, size_t u, size_t v, int32_t weight);
void tron_edges_sort_by_weight(TronEdges *te);

// Conversions
TronNetwork *tron_matrix_to_network(const TronMatrix *tm);
TronMatrix *tron_network_to_matrix(const TronNetwork *tn);
TronEdges *tron_network_to_edges(const TronNetwork *tn);
TronNetwork *tron_edges_to_network(const TronEdges *te);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Fait Historique

Le film TRON (1982) a été le premier à utiliser massivement des images de synthèse pour représenter un monde numérique. La "Grid" du film est une métaphore visuelle parfaite d'un graphe : des programmes (nœuds) reliés par des flux de données (arêtes). Disney a d'abord refusé le projet car l'utilisation d'ordinateurs était considérée comme "de la triche" !

### 2.2 Pourquoi Plusieurs Représentations ?

| Représentation | Espace | has_edge | Tous voisins | Ajout arête | Quand l'utiliser |
|----------------|--------|----------|--------------|-------------|------------------|
| Matrice | O(V²) | O(1) | O(V) | O(1) | Graphes denses, algos type Floyd-Warshall |
| Liste | O(V+E) | O(deg) | O(deg) | O(1) | Graphes épars, BFS/DFS, la plupart des cas |
| Arêtes | O(E) | O(E) | O(E) | O(1) | Kruskal, Bellman-Ford, tri des arêtes |

### 2.3 DANS LA VRAIE VIE

| Métier | Utilisation | Représentation préférée |
|--------|-------------|-------------------------|
| **Data Engineer (Meta/LinkedIn)** | Graphe social (amis, connexions) | Liste d'adjacence (billions d'utilisateurs, connexions éparses) |
| **DevOps/SRE** | Dépendances de microservices | Liste d'arêtes (pour détecter les cycles) |
| **Game Developer** | Carte de jeu, pathfinding | Grille implicite ou liste d'adjacence |
| **Network Engineer** | Topologie réseau, routage | Matrice (petit réseau) ou liste (grand réseau) |
| **Data Scientist** | Knowledge graphs, recommandations | Liste d'adjacence avec propriétés |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
tron_grid.c  tron_grid.h  main.c  Cargo.toml  src/

$ gcc -Wall -Wextra -Werror -std=c17 tron_grid.c main.c -o test_c

$ ./test_c
=== TRON GRID TEST SUITE ===
Test TronMatrix creation: OK
Test TronMatrix add_edge: OK
Test TronMatrix has_edge: OK
Test TronMatrix neighbors: OK
Test TronNetwork creation: OK
Test TronNetwork add_edge: OK
Test TronNetwork degree: OK
Test Conversion matrix->network: OK
Test Conversion network->edges: OK
All tests passed! The Grid is online.

$ cargo test
   Compiling tron_grid v0.1.0
    Finished test [unoptimized + debuginfo]
     Running unittests src/lib.rs

running 12 tests
test tron_grid::tests::test_matrix_basic ... ok
test tron_grid::tests::test_matrix_neighbors ... ok
test tron_grid::tests::test_network_basic ... ok
test tron_grid::tests::test_network_degree ... ok
test tron_grid::tests::test_edges_sort ... ok
test tron_grid::tests::test_matrix_to_network ... ok
test tron_grid::tests::test_network_to_edges ... ok
test tron_grid::tests::test_round_trip ... ok
test tron_grid::tests::test_undirected ... ok
test tron_grid::tests::test_empty_graph ... ok
test tron_grid::tests::test_self_loop ... ok
test tron_grid::tests::test_negative_weights ... ok

test result: ok. 12 passed; 0 failed
```

---

## ⚡ SECTION 3.1 : BONUS STANDARD (OPTIONNEL)

**Difficulté Bonus :**
★★★★★☆☆☆☆☆ (5/10)

**Récompense :**
XP ×2

**Time Complexity attendue :**
O(1) pour neighbors

**Space Complexity attendue :**
O(rows × cols)

### 3.1.1 Consigne Bonus : Light Cycle Arena

**🎮 "Flynn Lives! Now let's play some Light Cycles."**

Les Light Cycles de TRON circulent sur une **grille rectangulaire**. Chaque cellule peut être :
- **Libre** : le cycle peut y passer
- **Bloquée** : mur de lumière, obstacle

Implémente `TronArena` (GridGraph) avec :
- Voisins 4-directions (haut, bas, gauche, droite)
- Voisins 8-directions (inclut diagonales)

**Contraintes :**
┌─────────────────────────────────────┐
│  1 ≤ rows, cols ≤ 10⁴              │
│  Cellules hors grille = ignorées   │
│  Cellules bloquées = pas voisines  │
└─────────────────────────────────────┘

### 3.1.2 Prototype Bonus

```rust
pub struct TronArena {
    rows: usize,
    cols: usize,
    blocked: Vec<Vec<bool>>,
}

impl TronArena {
    pub fn new(rows: usize, cols: usize) -> Self;
    pub fn block(&mut self, r: usize, c: usize);
    pub fn unblock(&mut self, r: usize, c: usize);
    pub fn is_blocked(&self, r: usize, c: usize) -> bool;
    pub fn neighbors_4(&self, r: usize, c: usize) -> Vec<(usize, usize)>;
    pub fn neighbors_8(&self, r: usize, c: usize) -> Vec<(usize, usize)>;
    pub fn to_vertex_id(&self, r: usize, c: usize) -> usize;
    pub fn from_vertex_id(&self, id: usize) -> (usize, usize);
}
```

---

## 🔥 SECTION 3.2 : BONUS EXPERT (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

### 3.2.1 Consigne Bonus : ISO Chess Graph

Implémente un graphe implicite pour un échiquier où les arêtes représentent les mouvements valides des pièces.

```rust
pub enum ChessPiece { King, Queen, Rook, Bishop, Knight, Pawn(bool) } // bool = white?

pub struct IsoChessGraph {
    board: [[Option<ChessPiece>; 8]; 8],
}

impl IsoChessGraph {
    pub fn valid_moves(&self, r: usize, c: usize) -> Vec<(usize, usize)>;
    pub fn is_check(&self, white_king: bool) -> bool;
}
```

---

## 🧠 SECTION 3.3 : BONUS GÉNIE (OPTIONNEL)

**Difficulté Bonus :**
🧠 (12/10)

**Récompense :**
XP ×6

### 3.3.1 Consigne Bonus : CSR Compression

Implémente la représentation **Compressed Sparse Row (CSR)** utilisée dans les bibliothèques de calcul scientifique (SciPy, cuSPARSE).

```rust
pub struct TronCSR {
    values: Vec<i32>,      // Poids des arêtes (taille E)
    col_idx: Vec<usize>,   // Destinations (taille E)
    row_ptr: Vec<usize>,   // Pointeurs de début par ligne (taille V+1)
    n: usize,
}

impl TronCSR {
    pub fn from_network(network: &TronNetwork) -> Self;
    pub fn neighbors(&self, u: usize) -> &[(usize, i32)]; // Slice view
    pub fn to_network(&self) -> TronNetwork;
}
```

Avantage : Cache-friendly, optimal pour SpMV (Sparse Matrix-Vector multiplication).

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `test_matrix_new` | `n=5` | matrix 5x5 remplie de DISCONNECTED | 5 | Init à 0 au lieu de MAX |
| `test_matrix_edge` | `add(0,1,10)` | `has(0,1)=true, weight=10` | 5 | - |
| `test_matrix_no_edge` | `has(0,2)` sur graphe vide | `false` | 5 | Retourne true par défaut |
| `test_matrix_neighbors` | 3 voisins | Vec de taille 3 | 5 | Inclut les non-voisins |
| `test_network_new` | `n=5` | 5 listes vides | 5 | - |
| `test_network_edge` | `add(0,1,10)` | degree(0)=1 | 5 | - |
| `test_network_undirected` | `add_undirected(0,1,10)` | degree(0)=1, degree(1)=1 | 5 | Oublie l'arête retour |
| `test_edges_sort` | edges non triées | triées par poids croissant | 5 | Tri décroissant |
| `test_conv_m2n` | matrix 3x3 | network équivalent | 10 | Perd des arêtes |
| `test_conv_n2e` | network | edges équivalent | 10 | - |
| `test_round_trip` | matrix→network→matrix | identique | 10 | - |
| `test_empty` | n=0 | structures vides valides | 5 | Crash sur n=0 |
| `test_self_loop` | add(0,0,5) | boucle sur soi | 5 | Ignore les self-loops |
| `test_negative_weight` | weight=-10 | accepté | 5 | Rejette négatifs |
| `test_bounds` | u ou v >= n | gestion propre | 5 | Panic/crash |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include "tron_grid.h"

void test_matrix_basic(void)
{
    TronMatrix *tm = tron_matrix_new(4);
    assert(tm != NULL);
    assert(!tron_matrix_has_edge(tm, 0, 1));

    tron_matrix_add_edge(tm, 0, 1, 10);
    tron_matrix_add_edge(tm, 0, 2, 20);
    tron_matrix_add_edge(tm, 1, 2, 5);

    assert(tron_matrix_has_edge(tm, 0, 1));
    assert(!tron_matrix_has_edge(tm, 1, 0)); // Dirigé

    bool exists;
    assert(tron_matrix_weight(tm, 0, 1, &exists) == 10 && exists);
    assert(tron_matrix_weight(tm, 2, 0, &exists) == DISCONNECTED && !exists);

    size_t count;
    size_t *neighbors = tron_matrix_neighbors(tm, 0, &count);
    assert(count == 2);
    free(neighbors);

    tron_matrix_free(tm);
    printf("Test TronMatrix basic: OK\n");
}

void test_network_basic(void)
{
    TronNetwork *tn = tron_network_new(5);
    assert(tn != NULL);

    tron_network_add_edge(tn, 0, 1, 10);
    tron_network_add_edge(tn, 0, 2, 20);

    assert(tron_network_degree(tn, 0) == 2);
    assert(tron_network_degree(tn, 1) == 0);
    assert(tron_network_edge_count(tn) == 2);

    size_t count;
    const TronEdge *neighbors = tron_network_neighbors(tn, 0, &count);
    assert(count == 2);
    assert(neighbors[0].neighbor == 1 && neighbors[0].weight == 10);

    tron_network_free(tn);
    printf("Test TronNetwork basic: OK\n");
}

void test_network_undirected(void)
{
    TronNetwork *tn = tron_network_new(3);
    tron_network_add_undirected(tn, 0, 1, 5);

    assert(tron_network_degree(tn, 0) == 1);
    assert(tron_network_degree(tn, 1) == 1);
    assert(tron_network_edge_count(tn) == 2);

    tron_network_free(tn);
    printf("Test TronNetwork undirected: OK\n");
}

void test_edges_sort(void)
{
    TronEdges *te = tron_edges_new(4);
    tron_edges_add(te, 0, 1, 30);
    tron_edges_add(te, 1, 2, 10);
    tron_edges_add(te, 2, 3, 20);

    tron_edges_sort_by_weight(te);

    assert(te->edges[0].weight == 10);
    assert(te->edges[1].weight == 20);
    assert(te->edges[2].weight == 30);

    tron_edges_free(te);
    printf("Test TronEdges sort: OK\n");
}

void test_conversion_m2n(void)
{
    TronMatrix *tm = tron_matrix_new(3);
    tron_matrix_add_edge(tm, 0, 1, 5);
    tron_matrix_add_edge(tm, 1, 2, 10);

    TronNetwork *tn = tron_matrix_to_network(tm);
    assert(tron_network_degree(tn, 0) == 1);
    assert(tron_network_degree(tn, 1) == 1);
    assert(tron_network_edge_count(tn) == 2);

    tron_matrix_free(tm);
    tron_network_free(tn);
    printf("Test Conversion matrix->network: OK\n");
}

void test_conversion_n2e(void)
{
    TronNetwork *tn = tron_network_new(3);
    tron_network_add_edge(tn, 0, 1, 5);
    tron_network_add_edge(tn, 1, 2, 10);

    TronEdges *te = tron_network_to_edges(tn);
    assert(te->count == 2);

    tron_network_free(tn);
    tron_edges_free(te);
    printf("Test Conversion network->edges: OK\n");
}

void test_empty_graph(void)
{
    TronMatrix *tm = tron_matrix_new(0);
    assert(tm != NULL);
    tron_matrix_free(tm);

    TronNetwork *tn = tron_network_new(0);
    assert(tn != NULL);
    assert(tron_network_vertex_count(tn) == 0);
    tron_network_free(tn);

    printf("Test empty graph: OK\n");
}

void test_self_loop(void)
{
    TronNetwork *tn = tron_network_new(3);
    tron_network_add_edge(tn, 1, 1, 42);

    assert(tron_network_degree(tn, 1) == 1);

    size_t count;
    const TronEdge *neighbors = tron_network_neighbors(tn, 1, &count);
    assert(count == 1 && neighbors[0].neighbor == 1);

    tron_network_free(tn);
    printf("Test self-loop: OK\n");
}

int main(void)
{
    printf("=== TRON GRID TEST SUITE ===\n");
    test_matrix_basic();
    test_network_basic();
    test_network_undirected();
    test_edges_sort();
    test_conversion_m2n();
    test_conversion_n2e();
    test_empty_graph();
    test_self_loop();
    printf("All tests passed! The Grid is online.\n");
    return 0;
}
```

### 4.3 Solution de référence (Rust)

```rust
pub mod tron_grid {
    pub const DISCONNECTED: i32 = i32::MAX;

    // === TronMatrix ===
    pub struct TronMatrix {
        matrix: Vec<Vec<i32>>,
        n: usize,
    }

    impl TronMatrix {
        pub fn new(n: usize) -> Self {
            Self {
                matrix: vec![vec![DISCONNECTED; n]; n],
                n,
            }
        }

        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32) {
            if u < self.n && v < self.n {
                self.matrix[u][v] = weight;
            }
        }

        pub fn add_undirected_edge(&mut self, u: usize, v: usize, weight: i32) {
            self.add_edge(u, v, weight);
            self.add_edge(v, u, weight);
        }

        pub fn has_edge(&self, u: usize, v: usize) -> bool {
            u < self.n && v < self.n && self.matrix[u][v] != DISCONNECTED
        }

        pub fn weight(&self, u: usize, v: usize) -> Option<i32> {
            if self.has_edge(u, v) {
                Some(self.matrix[u][v])
            } else {
                None
            }
        }

        pub fn neighbors(&self, u: usize) -> Vec<usize> {
            if u >= self.n {
                return Vec::new();
            }
            (0..self.n)
                .filter(|&v| self.matrix[u][v] != DISCONNECTED)
                .collect()
        }

        pub fn size(&self) -> usize {
            self.n
        }
    }

    // === TronNetwork ===
    pub struct TronNetwork {
        adj: Vec<Vec<(usize, i32)>>,
        n: usize,
        m: usize,
    }

    impl TronNetwork {
        pub fn new(n: usize) -> Self {
            Self {
                adj: vec![Vec::new(); n],
                n,
                m: 0,
            }
        }

        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32) {
            if u < self.n && v < self.n {
                self.adj[u].push((v, weight));
                self.m += 1;
            }
        }

        pub fn add_undirected_edge(&mut self, u: usize, v: usize, weight: i32) {
            self.add_edge(u, v, weight);
            self.add_edge(v, u, weight);
        }

        pub fn neighbors(&self, u: usize) -> &[(usize, i32)] {
            if u < self.n {
                &self.adj[u]
            } else {
                &[]
            }
        }

        pub fn degree(&self, u: usize) -> usize {
            if u < self.n {
                self.adj[u].len()
            } else {
                0
            }
        }

        pub fn vertex_count(&self) -> usize {
            self.n
        }

        pub fn edge_count(&self) -> usize {
            self.m
        }
    }

    // === TronEdges ===
    pub struct TronEdges {
        edges: Vec<(usize, usize, i32)>,
        n: usize,
    }

    impl TronEdges {
        pub fn new(n: usize) -> Self {
            Self {
                edges: Vec::new(),
                n,
            }
        }

        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32) {
            if u < self.n && v < self.n {
                self.edges.push((u, v, weight));
            }
        }

        pub fn edges(&self) -> &[(usize, usize, i32)] {
            &self.edges
        }

        pub fn sort_by_weight(&mut self) {
            self.edges.sort_by_key(|&(_, _, w)| w);
        }

        pub fn vertex_count(&self) -> usize {
            self.n
        }
    }

    // === Conversions ===
    pub fn matrix_to_network(matrix: &TronMatrix) -> TronNetwork {
        let mut network = TronNetwork::new(matrix.size());
        for u in 0..matrix.size() {
            for v in 0..matrix.size() {
                if let Some(w) = matrix.weight(u, v) {
                    network.add_edge(u, v, w);
                }
            }
        }
        network
    }

    pub fn network_to_matrix(network: &TronNetwork) -> TronMatrix {
        let mut matrix = TronMatrix::new(network.vertex_count());
        for u in 0..network.vertex_count() {
            for &(v, w) in network.neighbors(u) {
                matrix.add_edge(u, v, w);
            }
        }
        matrix
    }

    pub fn network_to_edges(network: &TronNetwork) -> TronEdges {
        let mut edges = TronEdges::new(network.vertex_count());
        for u in 0..network.vertex_count() {
            for &(v, w) in network.neighbors(u) {
                edges.add_edge(u, v, w);
            }
        }
        edges
    }

    pub fn edges_to_network(edges: &TronEdges) -> TronNetwork {
        let mut network = TronNetwork::new(edges.vertex_count());
        for &(u, v, w) in edges.edges() {
            network.add_edge(u, v, w);
        }
        network
    }

    #[cfg(test)]
    mod tests {
        use super::*;

        #[test]
        fn test_matrix_basic() {
            let mut m = TronMatrix::new(4);
            assert!(!m.has_edge(0, 1));
            m.add_edge(0, 1, 10);
            assert!(m.has_edge(0, 1));
            assert_eq!(m.weight(0, 1), Some(10));
            assert_eq!(m.weight(1, 0), None);
        }

        #[test]
        fn test_matrix_neighbors() {
            let mut m = TronMatrix::new(4);
            m.add_edge(0, 1, 1);
            m.add_edge(0, 2, 2);
            m.add_edge(0, 3, 3);
            assert_eq!(m.neighbors(0), vec![1, 2, 3]);
        }

        #[test]
        fn test_network_basic() {
            let mut n = TronNetwork::new(5);
            n.add_edge(0, 1, 10);
            n.add_edge(0, 2, 20);
            assert_eq!(n.degree(0), 2);
            assert_eq!(n.edge_count(), 2);
        }

        #[test]
        fn test_network_degree() {
            let mut n = TronNetwork::new(3);
            n.add_undirected_edge(0, 1, 5);
            assert_eq!(n.degree(0), 1);
            assert_eq!(n.degree(1), 1);
            assert_eq!(n.edge_count(), 2);
        }

        #[test]
        fn test_edges_sort() {
            let mut e = TronEdges::new(4);
            e.add_edge(0, 1, 30);
            e.add_edge(1, 2, 10);
            e.add_edge(2, 3, 20);
            e.sort_by_weight();
            let edges = e.edges();
            assert_eq!(edges[0].2, 10);
            assert_eq!(edges[1].2, 20);
            assert_eq!(edges[2].2, 30);
        }

        #[test]
        fn test_matrix_to_network() {
            let mut m = TronMatrix::new(3);
            m.add_edge(0, 1, 5);
            m.add_edge(1, 2, 10);
            let n = matrix_to_network(&m);
            assert_eq!(n.degree(0), 1);
            assert_eq!(n.edge_count(), 2);
        }

        #[test]
        fn test_network_to_edges() {
            let mut n = TronNetwork::new(3);
            n.add_edge(0, 1, 5);
            n.add_edge(1, 2, 10);
            let e = network_to_edges(&n);
            assert_eq!(e.edges().len(), 2);
        }

        #[test]
        fn test_round_trip() {
            let mut m1 = TronMatrix::new(3);
            m1.add_edge(0, 1, 5);
            m1.add_edge(1, 2, 10);
            let n = matrix_to_network(&m1);
            let m2 = network_to_matrix(&n);
            assert_eq!(m1.weight(0, 1), m2.weight(0, 1));
            assert_eq!(m1.weight(1, 2), m2.weight(1, 2));
        }

        #[test]
        fn test_undirected() {
            let mut m = TronMatrix::new(3);
            m.add_undirected_edge(0, 1, 5);
            assert!(m.has_edge(0, 1));
            assert!(m.has_edge(1, 0));
        }

        #[test]
        fn test_empty_graph() {
            let m = TronMatrix::new(0);
            assert_eq!(m.size(), 0);
            let n = TronNetwork::new(0);
            assert_eq!(n.vertex_count(), 0);
        }

        #[test]
        fn test_self_loop() {
            let mut n = TronNetwork::new(3);
            n.add_edge(1, 1, 42);
            assert_eq!(n.degree(1), 1);
            assert_eq!(n.neighbors(1)[0], (1, 42));
        }

        #[test]
        fn test_negative_weights() {
            let mut n = TronNetwork::new(3);
            n.add_edge(0, 1, -10);
            assert_eq!(n.neighbors(0)[0].1, -10);
        }
    }
}
```

### 4.4 Solutions alternatives acceptées

```rust
// Alternative 1: Using HashSet for neighbors lookup
use std::collections::HashSet;

impl TronNetwork {
    pub fn has_edge(&self, u: usize, v: usize) -> bool {
        self.adj[u].iter().any(|&(neighbor, _)| neighbor == v)
    }
}

// Alternative 2: Builder pattern
impl TronNetwork {
    pub fn with_edge(mut self, u: usize, v: usize, w: i32) -> Self {
        self.add_edge(u, v, w);
        self
    }
}
```

### 4.5 Solutions refusées

```rust
// REFUSÉ: Pas de vérification des bornes
impl TronMatrix {
    pub fn add_edge(&mut self, u: usize, v: usize, weight: i32) {
        self.matrix[u][v] = weight; // CRASH si u >= n ou v >= n
    }
}
// Pourquoi: Panic en production sur entrée invalide

// REFUSÉ: Mauvaise initialisation
impl TronMatrix {
    pub fn new(n: usize) -> Self {
        Self {
            matrix: vec![vec![0; n]; n], // 0 au lieu de DISCONNECTED
            n,
        }
    }
}
// Pourquoi: has_edge retourne true pour 0 qui n'est pas une arête
```

### 4.6 Solution bonus de référence (TronArena)

```rust
pub struct TronArena {
    rows: usize,
    cols: usize,
    blocked: Vec<Vec<bool>>,
}

impl TronArena {
    const DIRS_4: [(i32, i32); 4] = [(-1, 0), (1, 0), (0, -1), (0, 1)];
    const DIRS_8: [(i32, i32); 8] = [
        (-1, 0), (1, 0), (0, -1), (0, 1),
        (-1, -1), (-1, 1), (1, -1), (1, 1)
    ];

    pub fn new(rows: usize, cols: usize) -> Self {
        Self {
            rows,
            cols,
            blocked: vec![vec![false; cols]; rows],
        }
    }

    pub fn block(&mut self, r: usize, c: usize) {
        if r < self.rows && c < self.cols {
            self.blocked[r][c] = true;
        }
    }

    pub fn unblock(&mut self, r: usize, c: usize) {
        if r < self.rows && c < self.cols {
            self.blocked[r][c] = false;
        }
    }

    pub fn is_blocked(&self, r: usize, c: usize) -> bool {
        r >= self.rows || c >= self.cols || self.blocked[r][c]
    }

    fn get_neighbors(&self, r: usize, c: usize, dirs: &[(i32, i32)]) -> Vec<(usize, usize)> {
        if self.is_blocked(r, c) {
            return Vec::new();
        }
        dirs.iter()
            .filter_map(|&(dr, dc)| {
                let nr = r as i32 + dr;
                let nc = c as i32 + dc;
                if nr >= 0 && nc >= 0 {
                    let (nr, nc) = (nr as usize, nc as usize);
                    if !self.is_blocked(nr, nc) {
                        return Some((nr, nc));
                    }
                }
                None
            })
            .collect()
    }

    pub fn neighbors_4(&self, r: usize, c: usize) -> Vec<(usize, usize)> {
        self.get_neighbors(r, c, &Self::DIRS_4)
    }

    pub fn neighbors_8(&self, r: usize, c: usize) -> Vec<(usize, usize)> {
        self.get_neighbors(r, c, &Self::DIRS_8)
    }

    pub fn to_vertex_id(&self, r: usize, c: usize) -> usize {
        r * self.cols + c
    }

    pub fn from_vertex_id(&self, id: usize) -> (usize, usize) {
        (id / self.cols, id % self.cols)
    }
}
```

### 4.9 spec.json

```json
{
  "name": "tron_grid_architect",
  "language": "rust",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isolé",
  "tags": ["graphs", "representations", "phase1", "tron"],
  "passing_score": 70,

  "function": {
    "name": "TronNetwork",
    "prototype": "impl TronNetwork { pub fn new(n: usize) -> Self; pub fn add_edge(&mut self, u: usize, v: usize, weight: i32); }",
    "return_type": "Self",
    "parameters": [
      {"name": "n", "type": "usize"}
    ]
  },

  "driver": {
    "reference": "pub struct TronNetwork { adj: Vec<Vec<(usize, i32)>>, n: usize, m: usize } impl TronNetwork { pub fn new(n: usize) -> Self { Self { adj: vec![Vec::new(); n], n, m: 0 } } pub fn add_edge(&mut self, u: usize, v: usize, weight: i32) { if u < self.n && v < self.n { self.adj[u].push((v, weight)); self.m += 1; } } pub fn degree(&self, u: usize) -> usize { if u < self.n { self.adj[u].len() } else { 0 } } }",

    "edge_cases": [
      {
        "name": "empty_graph",
        "args": [0],
        "expected": "TronNetwork with n=0",
        "is_trap": true,
        "trap_explanation": "n=0 doit créer un graphe vide valide, pas crash"
      },
      {
        "name": "single_node",
        "args": [1],
        "expected": "TronNetwork with 1 node, no edges"
      },
      {
        "name": "self_loop",
        "args": ["add_edge(0, 0, 5)"],
        "expected": "degree(0) = 1",
        "is_trap": true,
        "trap_explanation": "Les boucles sur soi doivent être acceptées"
      },
      {
        "name": "bounds_check",
        "args": ["add_edge(10, 0, 5) on n=3"],
        "expected": "no crash, ignored",
        "is_trap": true,
        "trap_explanation": "u >= n doit être ignoré, pas panic"
      },
      {
        "name": "negative_weight",
        "args": ["add_edge(0, 1, -10)"],
        "expected": "weight stored as -10"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 1000,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {"min": 0, "max": 1000}
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["Vec::new", "vec!", "push", "len", "iter", "filter", "collect", "malloc", "realloc", "free"],
    "forbidden_functions": ["petgraph", "graph_crate"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```rust
/* Mutant A (Boundary) : Pas de vérification des bornes */
impl TronNetwork {
    pub fn add_edge(&mut self, u: usize, v: usize, weight: i32) {
        // MANQUE: if u < self.n && v < self.n
        self.adj[u].push((v, weight)); // PANIC si u >= n
        self.m += 1;
    }
}
// Pourquoi c'est faux : Panic index out of bounds sur entrée invalide
// Ce qui était pensé : "L'utilisateur passera toujours des indices valides"

/* Mutant B (Safety) : Initialisation à 0 au lieu de DISCONNECTED */
impl TronMatrix {
    pub fn new(n: usize) -> Self {
        Self {
            matrix: vec![vec![0; n]; n], // BUG: 0 au lieu de DISCONNECTED
            n,
        }
    }

    pub fn has_edge(&self, u: usize, v: usize) -> bool {
        self.matrix[u][v] != 0 // BUG: 0 pourrait être un poids valide!
    }
}
// Pourquoi c'est faux : Une arête de poids 0 serait ignorée
// Ce qui était pensé : "0 signifie pas de connexion"

/* Mutant C (Resource) : Fuite mémoire en C - pas de free dans conversion */
TronNetwork *tron_matrix_to_network(const TronMatrix *tm) {
    TronNetwork *tn = tron_network_new(tm->n);
    // ... conversion ...
    // MANQUE: La matrice source n'est pas le problème, mais si on
    // alloue des buffers temporaires sans les libérer
    char *temp = malloc(1000); // Oublié de free
    return tn;
}
// Pourquoi c'est faux : Memory leak à chaque conversion
// Ce qui était pensé : "Le garbage collector s'en occupera" (il n'y en a pas en C)

/* Mutant D (Logic) : Oublie l'arête retour pour undirected */
impl TronNetwork {
    pub fn add_undirected_edge(&mut self, u: usize, v: usize, weight: i32) {
        self.add_edge(u, v, weight);
        // MANQUE: self.add_edge(v, u, weight);
    }
}
// Pourquoi c'est faux : Le graphe reste dirigé, degree(v) = 0
// Ce qui était pensé : "Une seule arête suffit pour undirected"

/* Mutant E (Return) : neighbors retourne slice vide */
impl TronNetwork {
    pub fn neighbors(&self, u: usize) -> &[(usize, i32)] {
        // BUG: Retourne toujours slice vide
        &[]
    }
}
// Pourquoi c'est faux : Aucun algorithme de parcours ne fonctionnera
// Ce qui était pensé : Erreur de copier-coller ou placeholder non remplacé
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Les trois représentations fondamentales** des graphes et leurs trade-offs
2. **Quand utiliser quelle représentation** selon l'algorithme et la densité
3. **Conversion entre représentations** sans perte d'information
4. **Graphes implicites** (Grid) où les arêtes sont calculées à la volée

### 5.2 LDA — Traduction littérale en MAJUSCULES

```
FONCTION add_edge QUI PREND EN PARAMÈTRES u ET v QUI SONT DES ENTIERS NON SIGNÉS ET weight QUI EST UN ENTIER SIGNÉ
DÉBUT FONCTION
    SI u EST INFÉRIEUR À n ET v EST INFÉRIEUR À n ALORS
        AJOUTER LE COUPLE (v, weight) À LA FIN DU VECTEUR adj[u]
        INCRÉMENTER m DE 1
    FIN SI
FIN FONCTION

FONCTION neighbors QUI RETOURNE UNE RÉFÉRENCE VERS UNE SLICE DE COUPLES ET PREND EN PARAMÈTRE u QUI EST UN ENTIER NON SIGNÉ
DÉBUT FONCTION
    SI u EST INFÉRIEUR À n ALORS
        RETOURNER UNE RÉFÉRENCE VERS adj[u]
    SINON
        RETOURNER UNE SLICE VIDE
    FIN SI
FIN FONCTION

FONCTION matrix_to_network QUI RETOURNE UN TronNetwork ET PREND EN PARAMÈTRE matrix QUI EST UNE RÉFÉRENCE VERS TronMatrix
DÉBUT FONCTION
    DÉCLARER network COMME NOUVEAU TronNetwork DE TAILLE matrix.size()
    POUR u ALLANT DE 0 À matrix.size() MOINS 1 FAIRE
        POUR v ALLANT DE 0 À matrix.size() MOINS 1 FAIRE
            SI matrix.weight(u, v) EST DIFFÉRENT DE None ALORS
                APPELER network.add_edge(u, v, w)
            FIN SI
        FIN POUR
    FIN POUR
    RETOURNER network
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Conversion Matrix → Network
---
1. CRÉER un nouveau network de même taille que matrix

2. POUR chaque sommet source u :
   |
   |-- POUR chaque sommet destination v :
   |     |
   |     |-- SI matrix.weight(u, v) existe :
   |     |     AJOUTER arête (u, v, weight) au network
   |     |
   |     |-- SINON :
   |           IGNORER (pas de connexion)

3. RETOURNER le network construit
```

### 5.2.3 Logique de Garde (Fail Fast)

```
FONCTION : add_edge (u, v, weight)
---
INIT résultat = échec implicite

1. VÉRIFIER si u >= n :
   |
   |-- RETOURNER immédiatement (rien à faire, pas d'erreur)

2. VÉRIFIER si v >= n :
   |
   |-- RETOURNER immédiatement

3. AJOUTER l'arête (v, weight) à adj[u]

4. INCRÉMENTER le compteur d'arêtes

5. RETOURNER (succès implicite)
```

### 5.3 Visualisation ASCII

```
=== THE GRID : REPRESENTATIONS ===

Programmes: Tron(0), Sam(1), Quorra(2), CLU(3)

Connexions:
  Tron ──5──> Sam
  Tron ──3──> Quorra
  Sam  ──2──> Quorra
  CLU  ──9──> Tron

=== MATRICE D'ADJACENCE (CLU's Surveillance) ===

           To:  0     1     2     3
              Tron  Sam  Quorra CLU
       ┌─────┬─────┬─────┬─────┬─────┐
From 0 │Tron │  ∞  │  5  │  3  │  ∞  │
       ├─────┼─────┼─────┼─────┼─────┤
     1 │Sam  │  ∞  │  ∞  │  2  │  ∞  │
       ├─────┼─────┼─────┼─────┼─────┤
     2 │Quorra│ ∞  │  ∞  │  ∞  │  ∞  │
       ├─────┼─────┼─────┼─────┼─────┤
     3 │CLU  │  9  │  ∞  │  ∞  │  ∞  │
       └─────┴─────┴─────┴─────┴─────┘

Espace: O(V²) = 16 cellules pour 4 programmes
Accès arête: O(1)

=== LISTE D'ADJACENCE (Tron's Network) ===

adj[0] (Tron):   [(1,5), (2,3)] ─────► Sam:5, Quorra:3
adj[1] (Sam):    [(2,2)] ────────────► Quorra:2
adj[2] (Quorra): [] ─────────────────► (aucune sortie)
adj[3] (CLU):    [(0,9)] ────────────► Tron:9

Espace: O(V + E) = 4 listes + 4 arêtes
Accès voisins: O(1)

=== LISTE D'ARÊTES (Edge Journal) ===

edges = [
    (0, 1, 5),   // Tron → Sam
    (0, 2, 3),   // Tron → Quorra
    (1, 2, 2),   // Sam → Quorra
    (3, 0, 9),   // CLU → Tron
]

Espace: O(E) = 4 arêtes
Tri par poids: O(E log E)

=== GRID GRAPH (Light Cycle Arena) ===

  0   1   2   3   4
┌───┬───┬───┬───┬───┐
│ . │ . │ X │ . │ . │ 0   X = blocked
├───┼───┼───┼───┼───┤
│ . │ . │ X │ . │ . │ 1
├───┼───┼───┼───┼───┤
│ . │ . │ . │ . │ . │ 2
└───┴───┴───┴───┴───┘

neighbors_4(1, 1) = [(0,1), (2,1), (1,0)]  // pas (1,2) car bloqué
neighbors_8(2, 2) = [(1,1), (1,2), (1,3), (2,1), (2,3), (3,1), (3,2), (3,3)]
```

### 5.4 Les pièges en détail

| Piège | Symptôme | Solution |
|-------|----------|----------|
| Init matrice à 0 | has_edge(i,j) true pour tout | Utiliser DISCONNECTED (MAX) |
| Oublier arête retour | Graphe dirigé au lieu de non-dirigé | add_undirected ajoute 2 arêtes |
| Pas de vérif bornes | Panic/crash sur u >= n | if u < n && v < n |
| Confusion index | Off-by-one sur voisins | Itérer 0..n, pas 1..=n |
| Grid: coords négatives | Wrap-around ou crash | Vérifier >= 0 avant cast |

### 5.5 Cours Complet

#### 5.5.1 Qu'est-ce qu'un graphe ?

Un **graphe** G = (V, E) est composé de :
- **V** (Vertices) : ensemble des sommets/nœuds
- **E** (Edges) : ensemble des arêtes/connexions entre sommets

Types de graphes :
- **Dirigé** : arêtes ont une direction (u → v ≠ v → u)
- **Non-dirigé** : arêtes symétriques (u ↔ v)
- **Pondéré** : arêtes ont un poids/coût
- **Dense** : E ≈ V² (presque toutes les connexions existent)
- **Épars** : E << V² (peu de connexions)

#### 5.5.2 Matrice d'adjacence

Structure : tableau 2D de taille V × V
- `matrix[u][v] = weight` si arête existe
- `matrix[u][v] = ∞` sinon

```rust
struct AdjMatrix {
    matrix: Vec<Vec<i32>>,  // matrix[u][v]
    n: usize,               // nombre de sommets
}
```

**Avantages :**
- Accès O(1) pour vérifier une arête
- Simple à implémenter
- Idéal pour Floyd-Warshall (tous les plus courts chemins)

**Inconvénients :**
- Espace O(V²) même pour graphe épars
- Lister tous les voisins : O(V)

#### 5.5.3 Liste d'adjacence

Structure : tableau de listes (une liste par sommet)
- `adj[u]` = liste des (voisin, poids) pour les arêtes sortantes de u

```rust
struct AdjList {
    adj: Vec<Vec<(usize, i32)>>,  // adj[u] = [(v1, w1), (v2, w2), ...]
    n: usize,
    m: usize,  // nombre d'arêtes
}
```

**Avantages :**
- Espace O(V + E) optimal
- Lister voisins : O(degree)
- Idéal pour BFS, DFS, Dijkstra

**Inconvénients :**
- Vérifier si arête existe : O(degree)

#### 5.5.4 Liste d'arêtes

Structure : liste plate de triplets (source, destination, poids)

```rust
struct EdgeList {
    edges: Vec<(usize, usize, i32)>,  // (from, to, weight)
    n: usize,
}
```

**Avantages :**
- Espace O(E) minimal
- Facile à trier par poids
- Idéal pour Kruskal, Bellman-Ford

**Inconvénients :**
- Accès aléatoire O(E)
- Pas de notion de "voisins d'un sommet"

#### 5.5.5 Graphe implicite (Grid)

Les arêtes ne sont pas stockées mais **calculées à la demande**.

```rust
struct GridGraph {
    rows: usize,
    cols: usize,
    blocked: Vec<Vec<bool>>,
}

impl GridGraph {
    fn neighbors_4(&self, r: usize, c: usize) -> Vec<(usize, usize)> {
        // Calcule les 4 voisins (haut, bas, gauche, droite)
        // sans stocker les arêtes
    }
}
```

**Avantages :**
- Espace O(V) au lieu de O(E)
- Idéal pour jeux, pathfinding sur grille

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ if u >= self.n { panic!("out of bounds"); }                    │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ if u >= self.n { return; }  // Silencieux                      │
│ // OU                                                          │
│ if u >= self.n { return Err(GraphError::OutOfBounds); }       │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • En production, panic = crash du serveur                       │
│ • Préférer le traitement gracieux des erreurs                   │
│ • Result/Option pour les erreurs récupérables                   │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario :** Construire un réseau et le convertir

```
┌───────┬─────────────────────────────────────┬──────────────────────────────────┐
│ Étape │ Instruction                         │ État                             │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   1   │ let mut n = TronNetwork::new(3)     │ adj=[[],[],[]], m=0              │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   2   │ n.add_edge(0, 1, 5)                 │ adj=[[(1,5)],[],[]], m=1         │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   3   │ n.add_edge(0, 2, 3)                 │ adj=[[(1,5),(2,3)],[],[]], m=2   │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   4   │ n.add_edge(1, 2, 2)                 │ adj=[[(1,5),(2,3)],[(2,2)],[]]   │
│       │                                     │ m=3                               │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   5   │ n.degree(0)                         │ Retourne 2 (len de adj[0])       │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   6   │ network_to_matrix(&n)               │ Crée matrice 3x3, parcourt adj   │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   7   │ Itère u=0, v∈{1,2}                  │ matrix[0][1]=5, matrix[0][2]=3   │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   8   │ Itère u=1, v∈{2}                    │ matrix[1][2]=2                   │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│   9   │ Itère u=2, v∈{}                     │ (rien à ajouter)                 │
├───────┼─────────────────────────────────────┼──────────────────────────────────┤
│  10   │ Retourne matrix                     │ Matrice avec 3 arêtes, reste ∞   │
└───────┴─────────────────────────────────────┴──────────────────────────────────┘
```

### 5.8 Mnémotechniques

#### 🎬 MEME : "I fight for the Users!" — Vérification des bornes

Dans TRON, Tron se bat pour protéger les Users (utilisateurs). Dans ton code, tu dois te battre pour protéger contre les indices invalides :

```rust
pub fn add_edge(&mut self, u: usize, v: usize, weight: i32) {
    // 🛡️ "I fight for the Users!"
    if u >= self.n || v >= self.n {
        return; // Protection active
    }
    // Safe zone
    self.adj[u].push((v, weight));
}
```

#### 💡 MEME : "The Grid. A digital frontier." — Choisir la représentation

| Densité | Représentation | Analogie TRON |
|---------|----------------|---------------|
| Dense (E ≈ V²) | Matrice | CLU surveille TOUT |
| Épars (E << V²) | Liste | Tron ne traque que les alliés |
| Algorithme sur arêtes | Edge List | Les cycles de lumière tracent des lignes |
| Grille 2D | Implicite | L'arène des Light Cycles |

#### 🔄 MEME : "Derezzed" — Conversion entre représentations

Comme les programmes de TRON peuvent être "derezzed" (décompilés) et reconstruits, les graphes peuvent être convertis entre représentations sans perte d'information :

```
Matrix ←→ List ←→ Edges
   ↑         ↓
   └─────────┘
     (Round trip = identique)
```

### 5.9 Applications pratiques

1. **Réseaux sociaux** : Liste d'adjacence (Facebook a 3 milliards d'utilisateurs, graphe très épars)

2. **GPS/Navigation** : Liste d'adjacence + Grid implicite (routes entre villes, grille de tuiles)

3. **Jeux vidéo** : Grid graph pour pathfinding (A*, BFS sur grille)

4. **Compilation** : Edge list pour détection de cycles (dépendances entre modules)

5. **Machine Learning** : Matrice d'adjacence pour GNN (Graph Neural Networks)

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Test qui l'attrape |
|---|-------|--------|-------------------|
| 1 | Init matrice à 0 | has_edge toujours true pour 0 | test_weight_zero |
| 2 | Pas de vérif bornes | Panic/crash | test_bounds |
| 3 | Oubli arête retour (undirected) | Graphe reste dirigé | test_undirected |
| 4 | Self-loop ignorée | Perd information | test_self_loop |
| 5 | Confusion ∞ et None | weight() incohérent | test_no_edge |

---

## 📝 SECTION 7 : QCM

### Question 1
Quelle représentation utilise O(V²) d'espace ?

- A) Liste d'adjacence
- B) Matrice d'adjacence ✓
- C) Liste d'arêtes
- D) Graphe implicite

### Question 2
Pour un graphe avec V=1000 sommets et E=5000 arêtes, quelle représentation est la plus efficace en mémoire ?

- A) Matrice (1M cellules)
- B) Liste d'adjacence (1000 + 5000) ✓
- C) Les deux sont équivalentes
- D) Ça dépend du type de processeur

### Question 3
Quelle opération est O(1) avec une matrice mais O(degree) avec une liste ?

- A) Ajouter une arête
- B) Vérifier si arête existe ✓
- C) Lister tous les voisins
- D) Compter le nombre de sommets

### Question 4
Quelle représentation est idéale pour l'algorithme de Kruskal (MST) ?

- A) Matrice d'adjacence
- B) Liste d'adjacence
- C) Liste d'arêtes triées par poids ✓
- D) Graphe implicite

### Question 5
Dans un GridGraph, que retourne neighbors_4(0, 0) pour une grille 3x3 sans obstacles ?

- A) []
- B) [(0,1), (1,0)] ✓
- C) [(0,1), (1,0), (1,1)]
- D) [(0,0), (0,1), (1,0)]

### Question 6
Pour convertir une matrice en liste d'adjacence, quelle est la complexité ?

- A) O(V)
- B) O(E)
- C) O(V²) ✓
- D) O(V + E)

### Question 7
Quelle valeur utilise-t-on pour représenter "pas de connexion" dans une matrice ?

- A) 0
- B) -1
- C) NULL
- D) i32::MAX (∞) ✓

### Question 8
Un graphe non-dirigé avec add_undirected_edge(u, v) ajoute combien d'entrées dans une liste d'adjacence ?

- A) 1
- B) 2 ✓
- C) 0
- D) Dépend du poids

### Question 9
Quel avantage principal a un graphe implicite (Grid) ?

- A) Accès O(1) aux arêtes
- B) Tri facile des arêtes
- C) Ne stocke pas les arêtes, les calcule à la demande ✓
- D) Supporte les poids négatifs

### Question 10
Si degree(u) = 5 dans une liste d'adjacence, que signifie ce chiffre ?

- A) u a 5 sommets à distance 1
- B) u a 5 arêtes sortantes ✓
- C) u est le 5ème sommet
- D) Le poids total des arêtes de u est 5

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| Exercice | 1.4.0-a : tron_grid_architect |
| Thème | TRON: Legacy - The Grid |
| Concepts | Matrice, Liste, Arêtes, Grid |
| Difficulté Base | ★★★★☆☆☆☆☆☆ (4/10) |
| Bonus Standard | ★★★★★☆☆☆☆☆ (5/10) — TronArena |
| Bonus Expert | ★★★★★★★☆☆☆ (7/10) — Chess Graph |
| Bonus Génie | 🧠 (12/10) — CSR Compression |
| XP Base | 100 |
| XP Max (avec bonus) | 100 × (1 + 2 + 3 + 6) = 1200 |
| Temps estimé | 45 min base, +90 min bonus |
| Langages | Rust Edition 2024, C17 |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.4.0-a-tron-grid-architect",
    "generated_at": "2026-01-11",

    "metadata": {
      "exercise_id": "1.4.0-a",
      "exercise_name": "tron_grid_architect",
      "module": "1.4.0",
      "module_name": "Représentations de Graphes",
      "concept": "a",
      "concept_name": "Structures de données pour graphes",
      "type": "code",
      "tier": 1,
      "tier_info": "Concept isolé",
      "phase": 1,
      "difficulty": 4,
      "difficulty_stars": "★★★★☆☆☆☆☆☆",
      "language": "rust",
      "language_version": "Edition 2024",
      "duration_minutes": 45,
      "xp_base": 100,
      "xp_bonus_multiplier": 2,
      "bonus_tier": "STANDARD",
      "bonus_icon": "⚡",
      "complexity_time": "T2 O(V²)",
      "complexity_space": "S2 O(V²)",
      "prerequisites": ["vectors", "structs", "memory_allocation"],
      "domains": ["Struct", "MD"],
      "domains_bonus": ["CPU"],
      "tags": ["graphs", "representations", "matrix", "adjacency", "tron"],
      "meme_reference": "I fight for the Users!"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "alternatives/alt_hashset.rs": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "references/ref_solution_bonus.rs",
        "alternatives/alt_hashset.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_safety.rs",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.rs",
        "mutants/mutant_e_return.rs"
      ]
    },

    "commands": {
      "validate_spec": "python3 hackbrain_engine_v22.py --validate-spec spec.json",
      "test_reference": "cargo test --lib",
      "test_c": "gcc -Wall -Wextra -Werror -std=c17 tron_grid.c main.c -o test && ./test",
      "test_mutants": "python3 hackbrain_mutation_tester.py -r references/ref_solution.rs -s spec.json --validate"
    }
  }
}
```

---

*"The Grid. A digital frontier. I tried to picture clusters of information as they moved through the computer. What did they look like? Ships? Motorcycles? Were the circuits like freeways? I kept dreaming of a world I thought I'd never see. And then one day... I got in."*

— Kevin Flynn, TRON: Legacy

---

*HACKBRAIN v5.5.2 — L'excellence pédagogique ne se négocie pas*
