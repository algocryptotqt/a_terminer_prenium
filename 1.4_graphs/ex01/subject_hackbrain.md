<thinking>
## Analyse du Concept
- Concept : Union-Find / Disjoint Set Union avec optimisations
- Phase demandée : 1 (Transition débutant → intermédiaire)
- Adapté ? OUI - Union-Find est une structure fondamentale avec plusieurs variantes enseignables

## Combo Base + Bonus
- Exercice de base : Union-Find basique avec path compression et union by rank
- Bonus Standard : Weighted Union-Find pour différences de poids
- Bonus Expert : Rollback Union-Find avec checkpoints
- Bonus Génie : Union-Find persistant (fully persistent)
- Palier bonus : ⚡🔥🧠
- Progression logique ? OUI

## Prérequis & Difficulté
- Prérequis réels : Vecteurs, récursivité, structures de données de base
- Difficulté estimée : 4/10
- Cohérent avec phase ? OUI (Phase 1 = 3-5/10)

## Aspect Fun/Culture
- Contexte choisi : Attack on Titan (Shingeki no Kyojin) - "Les Chemins" (The Paths)
- MEME mnémotechnique : "All Subjects of Ymir are connected through the Paths" = tous les éléments d'un set partagent la même racine
- Pourquoi c'est fun : Dans AoT, tous les Eldiens sont mystérieusement connectés via les "Chemins" - une dimension invisible où le Titan Fondateur peut accéder à tous. C'est EXACTEMENT Union-Find : find() trouve la racine (le Titan Fondateur), union() connecte des groupes (conquête de territoire), path compression = raccourci via les Chemins, rollback = manipulation des mémoires dans le temps.
- Score d'intelligence : 98/100 - Analogie exceptionnellement pertinente

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : find() sans vérification x < n → panic index out of bounds
2. Mutant B (Safety) : Pas de path compression → complexité O(n) au lieu de O(α(n))
3. Mutant C (Resource) : union() ne met pas à jour le compteur de composantes
4. Mutant D (Logic) : union by rank attache le plus grand au plus petit (inverse)
5. Mutant E (Return) : union() retourne true même si déjà dans le même composant

## Verdict
VALIDE - L'exercice est excellent avec une analogie Attack on Titan parfaitement adaptée à Union-Find
</thinking>

---

# Exercice 1.4.1-a : paths_of_ymir

**Module :**
1.4.1 — Union-Find (Disjoint Set Union)

**Concept :**
a — Structure Union-Find avec path compression et union by rank

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
- Récursivité basique
- Structures de données

**Domaines :**
Struct, MD

**Durée estimée :**
45 min

**XP Base :**
100

**Complexité :**
T1 O(α(n)) amorti × S1 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `paths_of_ymir.c`, `paths_of_ymir.h`

**Fonctions autorisées :**
- Rust : `Vec`, `std::mem::swap`
- C : `malloc`, `realloc`, `free`, `memcpy`

**Fonctions interdites :**
- Bibliothèques de graphes externes

### 1.2 Consigne

#### 1.2.1 Version Culture Pop : Attack on Titan - Les Chemins (The Paths)

**🎮 "Tous les Sujets d'Ymir sont connectés par les Chemins."**

Dans l'univers d'Attack on Titan, les **Eldiens** (Sujets d'Ymir) sont mystérieusement connectés via une dimension invisible appelée **Les Chemins** (The Paths). Cette connexion permet au **Titan Fondateur** (Founding Titan) d'accéder aux mémoires de tous les Eldiens et de manipuler leur corps.

Tu es Ymir Fritz, la première des Titans. Tu dois gérer les connexions entre tous tes descendants à travers les Chemins. Quand deux familles Eldiennes s'unissent (mariage, alliance), elles deviennent connectées via toi - la racine ultime.

**Analogie parfaite :**
- **Eldiens** = Éléments du set
- **Chemins** = Structure Union-Find
- **Trouver la racine** = Remonter les Chemins jusqu'à Ymir
- **Path compression** = Raccourcir le chemin direct vers Ymir (comme quand Eren accède directement aux Chemins)
- **Union** = Mariage/Alliance entre familles Eldiennes
- **Connected** = "Êtes-vous de la même lignée ?"

**Ta mission :**

Implémenter `PathsOfYmir`, une structure Union-Find qui permet de :

1. **`find(x)`** : Trouver la racine (l'ancêtre commun) d'un Eldien x
   - Utilise **path compression** pour optimiser les futures recherches

2. **`union(x, y)`** : Unir deux familles Eldiennes
   - Utilise **union by rank** pour garder l'arbre équilibré

3. **`connected(x, y)`** : Vérifier si deux Eldiens partagent un ancêtre commun

4. **`count()`** : Nombre de familles distinctes (composantes connexes)

5. **`size(x)`** : Taille de la famille de x

**Entrée :**
- `n: usize` : Nombre total d'Eldiens dans les Chemins

**Sortie :**
- Chaque méthode a son type de retour spécifié
- `find` : racine de l'élément
- `union` : `true` si union effectuée, `false` si déjà connectés
- `connected` : `true` si même composante
- `count` : nombre de composantes
- `size` : taille du composant

**Contraintes :**
- 0 ≤ x, y < n
- Complexité amortie O(α(n)) pour find et union (α = fonction d'Ackermann inverse)
- α(n) < 5 pour tout n pratique (quasi-constant)

**Exemples :**

| Opération | Résultat | Explication |
|-----------|----------|-------------|
| `PathsOfYmir::new(10)` | 10 familles | Chaque Eldien est sa propre famille |
| `poy.union(0, 1)` | `true` | Eren(0) et Mikasa(1) s'unissent |
| `poy.union(2, 3)` | `true` | Armin(2) et Annie(3) s'unissent |
| `poy.union(0, 2)` | `true` | Les deux groupes fusionnent |
| `poy.connected(1, 3)` | `true` | Mikasa et Annie sont maintenant connectées |
| `poy.count()` | `7` | 10 - 3 unions = 7 composantes |
| `poy.size(0)` | `4` | Groupe de 4 personnes |

#### 1.2.2 Version Académique

Implémenter une structure de données Union-Find (Disjoint Set Union) avec :

1. **Path compression** dans l'opération `find` : après avoir trouvé la racine, faire pointer tous les nœuds traversés directement vers la racine.

2. **Union by rank** dans l'opération `union` : attacher l'arbre de plus petite hauteur sous la racine de l'arbre de plus grande hauteur.

Ces deux optimisations combinées donnent une complexité amortie O(α(n)) où α est la fonction d'Ackermann inverse, quasi-constante en pratique.

### 1.3 Prototype

```rust
// Rust - Edition 2024
pub mod paths_of_ymir {

    /// The Paths connecting all Subjects of Ymir
    pub struct PathsOfYmir {
        parent: Vec<usize>,
        rank: Vec<usize>,
        size: Vec<usize>,
        count: usize,
    }

    impl PathsOfYmir {
        /// Create Paths with n isolated Eldians
        pub fn new(n: usize) -> Self;

        /// Find the Founding Titan (root) of Eldian x with path compression
        pub fn find(&mut self, x: usize) -> usize;

        /// Unite two Eldian families - returns true if new union
        pub fn union(&mut self, x: usize, y: usize) -> bool;

        /// Are x and y connected through the Paths?
        pub fn connected(&mut self, x: usize, y: usize) -> bool;

        /// Number of distinct families
        pub fn count(&self) -> usize;

        /// Size of x's family
        pub fn size(&mut self, x: usize) -> usize;
    }

    // === Applications ===

    /// Kruskal's MST using Union-Find
    pub fn rumbling_mst(n: usize, edges: &[(usize, usize, i64)]) -> (i64, Vec<(usize, usize, i64)>);

    /// Count connected components in an undirected graph
    pub fn count_eldian_families(n: usize, edges: &[(usize, usize)]) -> usize;

    /// Detect cycle in undirected graph (Marley invasion loop)
    pub fn detect_invasion_cycle(n: usize, edges: &[(usize, usize)]) -> bool;

    /// Find earliest time all Eldians are connected
    pub fn coordinate_activation_time(n: usize, edges: &[(usize, usize, i32)]) -> i32;
}
```

```c
// C17
#ifndef PATHS_OF_YMIR_H
#define PATHS_OF_YMIR_H

#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>

typedef struct {
    size_t *parent;
    size_t *rank;
    size_t *size;
    size_t count;
    size_t n;
} PathsOfYmir;

// Core operations
PathsOfYmir *poy_new(size_t n);
void poy_free(PathsOfYmir *poy);
size_t poy_find(PathsOfYmir *poy, size_t x);
bool poy_union(PathsOfYmir *poy, size_t x, size_t y);
bool poy_connected(PathsOfYmir *poy, size_t x, size_t y);
size_t poy_count(const PathsOfYmir *poy);
size_t poy_size(PathsOfYmir *poy, size_t x);

// Applications
typedef struct {
    size_t u;
    size_t v;
    int64_t weight;
} Edge;

typedef struct {
    int64_t total_cost;
    Edge *edges;
    size_t edge_count;
} MSTResult;

MSTResult rumbling_mst(size_t n, const Edge *edges, size_t edge_count);
void mst_result_free(MSTResult *result);
size_t count_eldian_families(size_t n, const Edge *edges, size_t edge_count);
bool detect_invasion_cycle(size_t n, const Edge *edges, size_t edge_count);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Fait Historique

Union-Find a été inventé par Bernard A. Galler et Michael J. Fischer en 1964. L'analyse de la complexité O(α(n)) a été prouvée par Robert Tarjan en 1975. La fonction d'Ackermann inverse α(n) croît si lentement que pour n = nombre d'atomes dans l'univers (≈10⁸⁰), α(n) < 5.

### 2.2 Pourquoi Path Compression + Union by Rank ?

| Optimisation | Seule | Combinée |
|--------------|-------|----------|
| Aucune | O(n) par opération | - |
| Path compression seule | O(log n) amorti | - |
| Union by rank seule | O(log n) | - |
| **Les deux** | - | **O(α(n)) ≈ O(1)** |

### 2.3 DANS LA VRAIE VIE

| Métier | Utilisation | Cas d'usage |
|--------|-------------|-------------|
| **Network Engineer** | Détection de boucles réseau | Spanning Tree Protocol |
| **Data Scientist** | Clustering hiérarchique | Groupement de données similaires |
| **Game Developer** | Génération procédurale de labyrinthes | Kruskal pour MST |
| **Social Network Analyst** | Communautés et cliques | Facebook friend suggestions |
| **Image Processing** | Segmentation d'images | Connected component labeling |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
paths_of_ymir.c  paths_of_ymir.h  main.c  Cargo.toml  src/

$ gcc -Wall -Wextra -Werror -std=c17 paths_of_ymir.c main.c -o test_c

$ ./test_c
=== PATHS OF YMIR TEST SUITE ===
Test creation: OK (10 isolated Eldians)
Test union(0,1): OK (9 components)
Test union(2,3): OK (8 components)
Test union(0,2): OK (7 components)
Test connected(1,3): OK (true)
Test connected(1,5): OK (false)
Test size(0): OK (4)
Test cycle detection: OK
Test MST: OK (cost=37, 8 edges)
All tests passed! The Rumbling is ready.

$ cargo test
   Compiling paths_of_ymir v0.1.0
    Finished test [unoptimized + debuginfo]
     Running unittests src/lib.rs

running 14 tests
test paths_of_ymir::tests::test_new ... ok
test paths_of_ymir::tests::test_union ... ok
test paths_of_ymir::tests::test_find_compression ... ok
test paths_of_ymir::tests::test_connected ... ok
test paths_of_ymir::tests::test_count ... ok
test paths_of_ymir::tests::test_size ... ok
test paths_of_ymir::tests::test_self_union ... ok
test paths_of_ymir::tests::test_large_scale ... ok
test paths_of_ymir::tests::test_chain ... ok
test paths_of_ymir::tests::test_star ... ok
test paths_of_ymir::tests::test_kruskal_mst ... ok
test paths_of_ymir::tests::test_cycle_detection ... ok
test paths_of_ymir::tests::test_no_cycle ... ok
test paths_of_ymir::tests::test_earliest_connection ... ok

test result: ok. 14 passed; 0 failed
```

---

## ⚡ SECTION 3.1 : BONUS STANDARD — Weighted Union-Find

**Difficulté Bonus :**
★★★★★☆☆☆☆☆ (5/10)

**Récompense :**
XP ×2

**Time Complexity attendue :**
O(α(n))

**Space Complexity attendue :**
O(n)

### 3.1.1 Consigne Bonus : Les Mémoires à travers les Chemins

**🎮 "À travers les Chemins, Eren peut voir les mémoires de ses prédécesseurs..."**

Le Titan Assaillant peut voir les mémoires des Titans qui l'ont précédé. Ces mémoires ont des "distances temporelles" - combien d'années séparent deux détenteurs.

Implémente `TimePathsOfYmir` qui stocke les **différences temporelles** entre Eldiens connectés :
- `union(x, y, w)` : x et y sont connectés, y est w années après x
- `diff(x, y)` : retourne combien d'années séparent x de y (si connectés)

**Contraintes :**
┌─────────────────────────────────────────┐
│  weight(y) - weight(x) = w lors de union│
│  Si déjà connectés avec diff ≠ w: erreur│
│  diff(x,y) = -diff(y,x)                 │
└─────────────────────────────────────────┘

### 3.1.2 Prototype Bonus

```rust
pub struct TimePathsOfYmir {
    parent: Vec<usize>,
    rank: Vec<usize>,
    diff: Vec<i64>,  // diff[x] = time(x) - time(parent[x])
}

impl TimePathsOfYmir {
    pub fn new(n: usize) -> Self;
    pub fn find(&mut self, x: usize) -> (usize, i64);  // (root, accumulated_diff)
    pub fn union(&mut self, x: usize, y: usize, w: i64) -> Result<bool, &'static str>;
    pub fn diff(&mut self, x: usize, y: usize) -> Option<i64>;
}
```

---

## 🔥 SECTION 3.2 : BONUS EXPERT — Rollback Union-Find

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

### 3.2.1 Consigne Bonus : Le Pouvoir de la Coordonnée

**🎮 "Avec le pouvoir de la Coordonnée, on peut remonter dans le temps..."**

Le Titan Fondateur peut manipuler le temps dans les Chemins. Implémente `CoordinateUnionFind` qui permet de sauvegarder des états (checkpoints) et d'y revenir (rollback).

**Important :** Pas de path compression (sinon rollback impossible).

```rust
pub struct CoordinateUnionFind {
    parent: Vec<usize>,
    rank: Vec<usize>,
    history: Vec<(usize, usize, usize)>,  // (node, old_parent, old_rank)
}

impl CoordinateUnionFind {
    pub fn new(n: usize) -> Self;
    pub fn find(&self, x: usize) -> usize;  // Pas de path compression!
    pub fn union(&mut self, x: usize, y: usize) -> bool;
    pub fn save(&self) -> usize;  // Retourne checkpoint (taille history)
    pub fn rollback(&mut self, checkpoint: usize);
}
```

---

## 🧠 SECTION 3.3 : BONUS GÉNIE — Persistent Union-Find

**Difficulté Bonus :**
🧠 (12/10)

**Récompense :**
XP ×6

### 3.3.1 Consigne Bonus : Les Chemins Parallèles

Implémente un Union-Find **fully persistent** où chaque version est accessible en O(log n).

```rust
pub struct PersistentUnionFind {
    // Fat node representation ou path copying
}

impl PersistentUnionFind {
    pub fn new(n: usize) -> Self;
    pub fn union(&self, x: usize, y: usize) -> Self;  // Retourne nouvelle version
    pub fn find(&self, x: usize) -> usize;
    pub fn connected(&self, x: usize, y: usize) -> bool;
}
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `test_new` | `n=10` | count=10, chaque parent[i]=i | 5 | Init incorrecte |
| `test_union_simple` | `union(0,1)` | count=9, connected=true | 5 | - |
| `test_union_return` | `union(0,1)` puis `union(0,1)` | true puis false | 5 | Retourne toujours true |
| `test_find_self` | `find(5)` sans union | 5 | 5 | - |
| `test_path_compression` | Chaîne 0→1→2→3→4, find(0) | Tous pointent vers 4 | 10 | Pas de compression |
| `test_union_by_rank` | Petits arbres sous grands | Hauteur minimisée | 10 | Rank inversé |
| `test_connected` | Après unions | Transitivité correcte | 5 | - |
| `test_size` | 4 éléments unis | size=4 pour tous | 5 | Taille pas mise à jour |
| `test_count` | 10 éléments, 3 unions | count=7 | 5 | Compteur pas décrémenté |
| `test_self_union` | `union(5,5)` | false (déjà connecté) | 5 | true ou crash |
| `test_bounds` | `find(n)` ou `union(n,0)` | Gestion propre | 5 | Panic |
| `test_large` | n=100000, random unions | Performances < 1s | 10 | Timeout (pas d'optim) |
| `test_kruskal` | Graphe 9 nœuds | MST cost=37 | 10 | - |
| `test_cycle` | Triangle | true | 5 | - |
| `test_no_cycle` | Arbre | false | 5 | Faux positif |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include "paths_of_ymir.h"

void test_basic(void)
{
    PathsOfYmir *poy = poy_new(10);
    assert(poy != NULL);
    assert(poy_count(poy) == 10);

    // Each Eldian is their own ancestor
    for (size_t i = 0; i < 10; i++) {
        assert(poy_find(poy, i) == i);
    }

    poy_free(poy);
    printf("Test basic: OK\n");
}

void test_union(void)
{
    PathsOfYmir *poy = poy_new(10);

    assert(poy_union(poy, 0, 1) == true);  // New union
    assert(poy_count(poy) == 9);

    assert(poy_union(poy, 2, 3) == true);
    assert(poy_count(poy) == 8);

    assert(poy_union(poy, 0, 2) == true);  // Merge two groups
    assert(poy_count(poy) == 7);

    assert(poy_union(poy, 1, 3) == false);  // Already connected!
    assert(poy_count(poy) == 7);  // Unchanged

    poy_free(poy);
    printf("Test union: OK\n");
}

void test_connected(void)
{
    PathsOfYmir *poy = poy_new(5);

    poy_union(poy, 0, 1);
    poy_union(poy, 2, 3);

    assert(poy_connected(poy, 0, 1) == true);
    assert(poy_connected(poy, 2, 3) == true);
    assert(poy_connected(poy, 0, 2) == false);
    assert(poy_connected(poy, 1, 4) == false);

    poy_union(poy, 0, 2);
    assert(poy_connected(poy, 1, 3) == true);  // Transitive!

    poy_free(poy);
    printf("Test connected: OK\n");
}

void test_size(void)
{
    PathsOfYmir *poy = poy_new(5);

    poy_union(poy, 0, 1);
    poy_union(poy, 0, 2);

    assert(poy_size(poy, 0) == 3);
    assert(poy_size(poy, 1) == 3);
    assert(poy_size(poy, 2) == 3);
    assert(poy_size(poy, 3) == 1);

    poy_free(poy);
    printf("Test size: OK\n");
}

void test_path_compression(void)
{
    PathsOfYmir *poy = poy_new(5);

    // Create chain: 0 → 1 → 2 → 3 → 4
    // (manually setting parents - in practice via unions)
    poy_union(poy, 0, 1);
    poy_union(poy, 1, 2);
    poy_union(poy, 2, 3);
    poy_union(poy, 3, 4);

    // After find(0), path should be compressed
    size_t root = poy_find(poy, 0);

    // All should point directly to root now (or close to it)
    // We verify by checking find is fast and consistent
    assert(poy_find(poy, 0) == root);
    assert(poy_find(poy, 1) == root);
    assert(poy_find(poy, 2) == root);
    assert(poy_find(poy, 3) == root);
    assert(poy_find(poy, 4) == root);

    poy_free(poy);
    printf("Test path compression: OK\n");
}

void test_cycle_detection(void)
{
    // Triangle: 0-1, 1-2, 2-0 (has cycle)
    Edge edges1[] = {{0, 1, 0}, {1, 2, 0}, {2, 0, 0}};
    assert(detect_invasion_cycle(3, edges1, 3) == true);

    // Tree: 0-1, 1-2, 2-3 (no cycle)
    Edge edges2[] = {{0, 1, 0}, {1, 2, 0}, {2, 3, 0}};
    assert(detect_invasion_cycle(4, edges2, 3) == false);

    printf("Test cycle detection: OK\n");
}

void test_kruskal_mst(void)
{
    // Classic MST example
    Edge edges[] = {
        {0, 1, 4}, {0, 7, 8}, {1, 2, 8}, {1, 7, 11},
        {2, 3, 7}, {2, 5, 4}, {2, 8, 2}, {3, 4, 9},
        {3, 5, 14}, {4, 5, 10}, {5, 6, 2}, {6, 7, 1}, {6, 8, 6}, {7, 8, 7}
    };

    MSTResult result = rumbling_mst(9, edges, 14);
    assert(result.total_cost == 37);
    assert(result.edge_count == 8);  // n-1 edges

    mst_result_free(&result);
    printf("Test MST: OK\n");
}

int main(void)
{
    printf("=== PATHS OF YMIR TEST SUITE ===\n");
    test_basic();
    test_union();
    test_connected();
    test_size();
    test_path_compression();
    test_cycle_detection();
    test_kruskal_mst();
    printf("All tests passed! The Rumbling is ready.\n");
    return 0;
}
```

### 4.3 Solution de référence (Rust)

```rust
pub mod paths_of_ymir {

    pub struct PathsOfYmir {
        parent: Vec<usize>,
        rank: Vec<usize>,
        size: Vec<usize>,
        count: usize,
    }

    impl PathsOfYmir {
        pub fn new(n: usize) -> Self {
            Self {
                parent: (0..n).collect(),
                rank: vec![0; n],
                size: vec![1; n],
                count: n,
            }
        }

        pub fn find(&mut self, x: usize) -> usize {
            if x >= self.parent.len() {
                return x;  // Invalid index protection
            }
            if self.parent[x] != x {
                // Path compression: make x point directly to root
                self.parent[x] = self.find(self.parent[x]);
            }
            self.parent[x]
        }

        pub fn union(&mut self, x: usize, y: usize) -> bool {
            if x >= self.parent.len() || y >= self.parent.len() {
                return false;
            }

            let root_x = self.find(x);
            let root_y = self.find(y);

            if root_x == root_y {
                return false;  // Already in same component
            }

            // Union by rank: attach smaller tree under larger
            if self.rank[root_x] < self.rank[root_y] {
                self.parent[root_x] = root_y;
                self.size[root_y] += self.size[root_x];
            } else if self.rank[root_x] > self.rank[root_y] {
                self.parent[root_y] = root_x;
                self.size[root_x] += self.size[root_y];
            } else {
                self.parent[root_y] = root_x;
                self.size[root_x] += self.size[root_y];
                self.rank[root_x] += 1;
            }

            self.count -= 1;
            true
        }

        pub fn connected(&mut self, x: usize, y: usize) -> bool {
            self.find(x) == self.find(y)
        }

        pub fn count(&self) -> usize {
            self.count
        }

        pub fn size(&mut self, x: usize) -> usize {
            if x >= self.parent.len() {
                return 0;
            }
            let root = self.find(x);
            self.size[root]
        }
    }

    // === Applications ===

    pub fn rumbling_mst(n: usize, edges: &[(usize, usize, i64)]) -> (i64, Vec<(usize, usize, i64)>) {
        let mut sorted_edges = edges.to_vec();
        sorted_edges.sort_by_key(|&(_, _, w)| w);

        let mut uf = PathsOfYmir::new(n);
        let mut mst = Vec::new();
        let mut total_cost = 0i64;

        for (u, v, w) in sorted_edges {
            if uf.union(u, v) {
                mst.push((u, v, w));
                total_cost += w;
            }
            if mst.len() == n - 1 {
                break;
            }
        }

        (total_cost, mst)
    }

    pub fn count_eldian_families(n: usize, edges: &[(usize, usize)]) -> usize {
        let mut uf = PathsOfYmir::new(n);
        for &(u, v) in edges {
            uf.union(u, v);
        }
        uf.count()
    }

    pub fn detect_invasion_cycle(n: usize, edges: &[(usize, usize)]) -> bool {
        let mut uf = PathsOfYmir::new(n);
        for &(u, v) in edges {
            if uf.connected(u, v) {
                return true;  // Already connected = cycle!
            }
            uf.union(u, v);
        }
        false
    }

    pub fn coordinate_activation_time(n: usize, edges: &[(usize, usize, i32)]) -> i32 {
        let mut sorted = edges.to_vec();
        sorted.sort_by_key(|&(_, _, t)| t);

        let mut uf = PathsOfYmir::new(n);
        for (u, v, t) in sorted {
            uf.union(u, v);
            if uf.count() == 1 {
                return t;
            }
        }
        -1  // Never fully connected
    }

    #[cfg(test)]
    mod tests {
        use super::*;

        #[test]
        fn test_new() {
            let poy = PathsOfYmir::new(10);
            assert_eq!(poy.count(), 10);
        }

        #[test]
        fn test_union() {
            let mut poy = PathsOfYmir::new(10);
            assert!(poy.union(0, 1));
            assert_eq!(poy.count(), 9);
            assert!(!poy.union(0, 1));  // Already same
            assert_eq!(poy.count(), 9);
        }

        #[test]
        fn test_find_compression() {
            let mut poy = PathsOfYmir::new(5);
            poy.union(0, 1);
            poy.union(1, 2);
            poy.union(2, 3);
            poy.union(3, 4);

            let root = poy.find(0);
            // After compression, all should point to same root
            assert_eq!(poy.find(1), root);
            assert_eq!(poy.find(2), root);
            assert_eq!(poy.find(3), root);
            assert_eq!(poy.find(4), root);
        }

        #[test]
        fn test_connected() {
            let mut poy = PathsOfYmir::new(5);
            poy.union(0, 1);
            poy.union(2, 3);

            assert!(poy.connected(0, 1));
            assert!(!poy.connected(0, 2));

            poy.union(0, 2);
            assert!(poy.connected(1, 3));  // Transitive
        }

        #[test]
        fn test_count() {
            let mut poy = PathsOfYmir::new(10);
            poy.union(0, 1);
            poy.union(2, 3);
            poy.union(0, 2);
            assert_eq!(poy.count(), 7);
        }

        #[test]
        fn test_size() {
            let mut poy = PathsOfYmir::new(5);
            poy.union(0, 1);
            poy.union(0, 2);

            assert_eq!(poy.size(0), 3);
            assert_eq!(poy.size(1), 3);
            assert_eq!(poy.size(3), 1);
        }

        #[test]
        fn test_self_union() {
            let mut poy = PathsOfYmir::new(5);
            assert!(!poy.union(2, 2));  // Self-union should be false
        }

        #[test]
        fn test_large_scale() {
            let mut poy = PathsOfYmir::new(100_000);
            for i in 0..99_999 {
                poy.union(i, i + 1);
            }
            assert_eq!(poy.count(), 1);
            assert_eq!(poy.size(0), 100_000);
        }

        #[test]
        fn test_chain() {
            let mut poy = PathsOfYmir::new(100);
            for i in 0..99 {
                poy.union(i, i + 1);
            }
            assert!(poy.connected(0, 99));
        }

        #[test]
        fn test_star() {
            let mut poy = PathsOfYmir::new(100);
            for i in 1..100 {
                poy.union(0, i);
            }
            assert_eq!(poy.count(), 1);
            assert_eq!(poy.size(50), 100);
        }

        #[test]
        fn test_kruskal_mst() {
            let edges = vec![
                (0, 1, 4), (0, 7, 8), (1, 2, 8), (1, 7, 11),
                (2, 3, 7), (2, 5, 4), (2, 8, 2), (3, 4, 9),
                (3, 5, 14), (4, 5, 10), (5, 6, 2), (6, 7, 1), (6, 8, 6), (7, 8, 7)
            ];
            let (cost, mst) = rumbling_mst(9, &edges);
            assert_eq!(cost, 37);
            assert_eq!(mst.len(), 8);
        }

        #[test]
        fn test_cycle_detection() {
            assert!(detect_invasion_cycle(3, &[(0, 1), (1, 2), (2, 0)]));
        }

        #[test]
        fn test_no_cycle() {
            assert!(!detect_invasion_cycle(4, &[(0, 1), (1, 2), (2, 3)]));
        }

        #[test]
        fn test_earliest_connection() {
            let edges = vec![(0, 1, 10), (1, 2, 20), (0, 2, 15)];
            assert_eq!(coordinate_activation_time(3, &edges), 20);
        }
    }
}
```

### 4.4 Solutions alternatives acceptées

```rust
// Alternative 1: Iterative find (no recursion)
pub fn find(&mut self, mut x: usize) -> usize {
    let mut root = x;
    while self.parent[root] != root {
        root = self.parent[root];
    }
    // Path compression
    while self.parent[x] != root {
        let next = self.parent[x];
        self.parent[x] = root;
        x = next;
    }
    root
}

// Alternative 2: Union by size instead of rank
pub fn union(&mut self, x: usize, y: usize) -> bool {
    let root_x = self.find(x);
    let root_y = self.find(y);
    if root_x == root_y { return false; }

    if self.size[root_x] < self.size[root_y] {
        self.parent[root_x] = root_y;
        self.size[root_y] += self.size[root_x];
    } else {
        self.parent[root_y] = root_x;
        self.size[root_x] += self.size[root_y];
    }
    self.count -= 1;
    true
}
```

### 4.5 Solutions refusées

```rust
// REFUSÉ: Pas de path compression
pub fn find(&self, x: usize) -> usize {
    if self.parent[x] == x { x }
    else { self.find(self.parent[x]) }  // Pas de mise à jour!
}
// Pourquoi: Complexité O(n) au lieu de O(α(n))

// REFUSÉ: Union sans union by rank
pub fn union(&mut self, x: usize, y: usize) -> bool {
    let root_x = self.find(x);
    let root_y = self.find(y);
    if root_x == root_y { return false; }
    self.parent[root_x] = root_y;  // Toujours attache x sous y
    true
}
// Pourquoi: Crée des chaînes déséquilibrées, O(n) dans le pire cas

// REFUSÉ: Ne retourne pas false quand déjà connectés
pub fn union(&mut self, x: usize, y: usize) -> bool {
    let root_x = self.find(x);
    let root_y = self.find(y);
    self.parent[root_x] = root_y;
    self.count -= 1;  // Décrémente même si déjà connectés!
    true
}
// Pourquoi: count devient négatif ou incorrect
```

### 4.6 Solution bonus de référence (Weighted Union-Find)

```rust
pub struct TimePathsOfYmir {
    parent: Vec<usize>,
    rank: Vec<usize>,
    diff: Vec<i64>,
}

impl TimePathsOfYmir {
    pub fn new(n: usize) -> Self {
        Self {
            parent: (0..n).collect(),
            rank: vec![0; n],
            diff: vec![0; n],
        }
    }

    pub fn find(&mut self, x: usize) -> (usize, i64) {
        if self.parent[x] == x {
            return (x, 0);
        }
        let (root, parent_diff) = self.find(self.parent[x]);
        let total_diff = self.diff[x] + parent_diff;
        self.parent[x] = root;
        self.diff[x] = total_diff;
        (root, total_diff)
    }

    pub fn union(&mut self, x: usize, y: usize, w: i64) -> Result<bool, &'static str> {
        let (root_x, diff_x) = self.find(x);
        let (root_y, diff_y) = self.find(y);

        if root_x == root_y {
            // Already connected - verify consistency
            if diff_y - diff_x != w {
                return Err("Inconsistent weight constraint");
            }
            return Ok(false);
        }

        // w = weight(y) - weight(x)
        // We need: diff[root_x] such that diff_x + diff[root_x] = diff_y + w
        // => diff[root_x] = diff_y - diff_x + w

        if self.rank[root_x] < self.rank[root_y] {
            self.parent[root_x] = root_y;
            self.diff[root_x] = diff_y - diff_x - w;
        } else {
            self.parent[root_y] = root_x;
            self.diff[root_y] = diff_x - diff_y + w;
            if self.rank[root_x] == self.rank[root_y] {
                self.rank[root_x] += 1;
            }
        }
        Ok(true)
    }

    pub fn diff(&mut self, x: usize, y: usize) -> Option<i64> {
        let (root_x, diff_x) = self.find(x);
        let (root_y, diff_y) = self.find(y);
        if root_x == root_y {
            Some(diff_y - diff_x)
        } else {
            None
        }
    }
}
```

### 4.9 spec.json

```json
{
  "name": "paths_of_ymir",
  "language": "rust",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isolé",
  "tags": ["graphs", "union-find", "dsu", "phase1", "attack-on-titan"],
  "passing_score": 70,

  "function": {
    "name": "PathsOfYmir",
    "prototype": "impl PathsOfYmir { pub fn new(n: usize) -> Self; pub fn find(&mut self, x: usize) -> usize; pub fn union(&mut self, x: usize, y: usize) -> bool; }",
    "return_type": "Self",
    "parameters": [
      {"name": "n", "type": "usize"}
    ]
  },

  "driver": {
    "reference": "pub struct PathsOfYmir { parent: Vec<usize>, rank: Vec<usize>, size: Vec<usize>, count: usize } impl PathsOfYmir { pub fn new(n: usize) -> Self { Self { parent: (0..n).collect(), rank: vec![0; n], size: vec![1; n], count: n } } pub fn find(&mut self, x: usize) -> usize { if x >= self.parent.len() { return x; } if self.parent[x] != x { self.parent[x] = self.find(self.parent[x]); } self.parent[x] } pub fn union(&mut self, x: usize, y: usize) -> bool { let rx = self.find(x); let ry = self.find(y); if rx == ry { return false; } if self.rank[rx] < self.rank[ry] { self.parent[rx] = ry; self.size[ry] += self.size[rx]; } else { self.parent[ry] = rx; self.size[rx] += self.size[ry]; if self.rank[rx] == self.rank[ry] { self.rank[rx] += 1; } } self.count -= 1; true } pub fn connected(&mut self, x: usize, y: usize) -> bool { self.find(x) == self.find(y) } pub fn count(&self) -> usize { self.count } pub fn size(&mut self, x: usize) -> usize { let r = self.find(x); self.size[r] } }",

    "edge_cases": [
      {
        "name": "empty_union_find",
        "args": [0],
        "expected": "PathsOfYmir with n=0, count=0",
        "is_trap": true,
        "trap_explanation": "n=0 doit créer une structure vide valide"
      },
      {
        "name": "self_union",
        "args": ["union(5, 5)"],
        "expected": "false",
        "is_trap": true,
        "trap_explanation": "Union d'un élément avec lui-même doit retourner false"
      },
      {
        "name": "bounds_check",
        "args": ["find(100) on n=10"],
        "expected": "100 (retourne l'entrée invalide)",
        "is_trap": true,
        "trap_explanation": "Indices >= n ne doivent pas causer de panic"
      },
      {
        "name": "transitive_connection",
        "args": ["union(0,1), union(1,2), connected(0,2)"],
        "expected": "true"
      },
      {
        "name": "path_compression",
        "args": ["chain 0-1-2-3-4, find(0)"],
        "expected": "tous pointent vers la même racine après find"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 5000,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {"min": 0, "max": 10000}
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["Vec::new", "vec!", "collect", "swap", "malloc", "free"],
    "forbidden_functions": ["external_union_find_lib"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```rust
/* Mutant A (Boundary) : Pas de vérification des bornes */
pub fn find(&mut self, x: usize) -> usize {
    // MANQUE: if x >= self.parent.len() { return x; }
    if self.parent[x] != x {
        self.parent[x] = self.find(self.parent[x]);
    }
    self.parent[x]  // PANIC si x >= len
}
// Pourquoi c'est faux : Index out of bounds panic
// Ce qui était pensé : "L'utilisateur ne passera jamais d'index invalide"

/* Mutant B (Safety) : Pas de path compression */
pub fn find(&self, x: usize) -> usize {
    if self.parent[x] == x {
        return x;
    }
    self.find(self.parent[x])  // Pas de mise à jour de parent[x]!
}
// Pourquoi c'est faux : Complexité O(n) au lieu de O(α(n))
// Ce qui était pensé : "La compression n'est pas nécessaire"

/* Mutant C (Resource) : Compteur pas mis à jour */
pub fn union(&mut self, x: usize, y: usize) -> bool {
    let root_x = self.find(x);
    let root_y = self.find(y);
    if root_x == root_y { return false; }

    self.parent[root_x] = root_y;
    // MANQUE: self.count -= 1;
    true
}
// Pourquoi c'est faux : count() retourne toujours n
// Ce qui était pensé : "Je mettrai à jour count plus tard"

/* Mutant D (Logic) : Union by rank inversé */
pub fn union(&mut self, x: usize, y: usize) -> bool {
    let root_x = self.find(x);
    let root_y = self.find(y);
    if root_x == root_y { return false; }

    // BUG: attache le PLUS GRAND sous le plus petit
    if self.rank[root_x] > self.rank[root_y] {  // > au lieu de <
        self.parent[root_x] = root_y;
    } else {
        self.parent[root_y] = root_x;
    }
    self.count -= 1;
    true
}
// Pourquoi c'est faux : Arbres déséquilibrés, hauteur O(n)
// Ce qui était pensé : Confusion sur le sens de l'inégalité

/* Mutant E (Return) : Retourne toujours true */
pub fn union(&mut self, x: usize, y: usize) -> bool {
    let root_x = self.find(x);
    let root_y = self.find(y);

    // BUG: Pas de check si déjà connectés
    self.parent[root_x] = root_y;
    self.count -= 1;  // Décrémente même si déjà connectés!
    true  // Retourne toujours true
}
// Pourquoi c'est faux : count devient négatif, logique incorrecte
// Ce qui était pensé : "union fait toujours quelque chose"
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Union-Find** : structure pour gérer des ensembles disjoints
2. **Path compression** : optimisation qui aplatit l'arbre lors du find
3. **Union by rank** : optimisation qui garde les arbres équilibrés
4. **Complexité amortie** : O(α(n)) quasi-constant
5. **Applications** : MST (Kruskal), détection de cycles, composantes connexes

### 5.2 LDA — Traduction littérale en MAJUSCULES

```
FONCTION find QUI RETOURNE UN ENTIER NON SIGNÉ ET PREND EN PARAMÈTRE x QUI EST UN ENTIER NON SIGNÉ
DÉBUT FONCTION
    SI x EST SUPÉRIEUR OU ÉGAL À LA TAILLE DU TABLEAU parent ALORS
        RETOURNER x
    FIN SI

    SI L'ÉLÉMENT À LA POSITION x DANS parent EST DIFFÉRENT DE x ALORS
        AFFECTER LE RÉSULTAT DE find(parent[x]) À L'ÉLÉMENT À LA POSITION x DANS parent
    FIN SI

    RETOURNER L'ÉLÉMENT À LA POSITION x DANS parent
FIN FONCTION

FONCTION union QUI RETOURNE UN BOOLÉEN ET PREND EN PARAMÈTRES x ET y QUI SONT DES ENTIERS NON SIGNÉS
DÉBUT FONCTION
    DÉCLARER root_x COMME LE RÉSULTAT DE find(x)
    DÉCLARER root_y COMME LE RÉSULTAT DE find(y)

    SI root_x EST ÉGAL À root_y ALORS
        RETOURNER FAUX
    FIN SI

    SI L'ÉLÉMENT À LA POSITION root_x DANS rank EST INFÉRIEUR À L'ÉLÉMENT À LA POSITION root_y DANS rank ALORS
        AFFECTER root_y À L'ÉLÉMENT À LA POSITION root_x DANS parent
        AFFECTER size[root_y] PLUS size[root_x] À L'ÉLÉMENT À LA POSITION root_y DANS size
    SINON SI L'ÉLÉMENT À LA POSITION root_x DANS rank EST SUPÉRIEUR À L'ÉLÉMENT À LA POSITION root_y DANS rank ALORS
        AFFECTER root_x À L'ÉLÉMENT À LA POSITION root_y DANS parent
        AFFECTER size[root_x] PLUS size[root_y] À L'ÉLÉMENT À LA POSITION root_x DANS size
    SINON
        AFFECTER root_x À L'ÉLÉMENT À LA POSITION root_y DANS parent
        AFFECTER size[root_x] PLUS size[root_y] À L'ÉLÉMENT À LA POSITION root_x DANS size
        INCRÉMENTER L'ÉLÉMENT À LA POSITION root_x DANS rank DE 1
    FIN SI

    DÉCRÉMENTER count DE 1
    RETOURNER VRAI
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Union-Find avec Path Compression
---
1. FIND(x) :
   a. SI parent[x] == x :
      RETOURNER x (c'est la racine)

   b. SINON :
      root = FIND(parent[x])  // Récursion
      parent[x] = root        // Path compression
      RETOURNER root

2. UNION(x, y) :
   a. TROUVER root_x = FIND(x)
   b. TROUVER root_y = FIND(y)

   c. SI root_x == root_y :
      RETOURNER false (déjà connectés)

   d. ATTACHER l'arbre de plus petit rank sous l'autre
      (Union by rank pour garder l'équilibre)

   e. DÉCRÉMENTER le compteur de composantes
   f. RETOURNER true
```

### 5.2.3 Logique de Garde (Fail Fast)

```
FONCTION : find (x)
---
1. VÉRIFIER si x >= n :
   |
   |-- RETOURNER x (index invalide, pas de crash)

2. SI parent[x] == x :
   |
   |-- RETOURNER x (racine trouvée)

3. SINON :
   |
   |-- APPELER récursivement find(parent[x])
   |-- METTRE À JOUR parent[x] = root (compression)
   |-- RETOURNER root

FONCTION : union (x, y)
---
1. TROUVER les racines
   root_x = find(x)
   root_y = find(y)

2. VÉRIFIER si déjà connectés :
   |
   |-- SI root_x == root_y :
         RETOURNER false (pas de nouvelle union)

3. ATTACHER le plus petit arbre sous le plus grand
   |
   |-- SI rank[root_x] < rank[root_y] :
   |     parent[root_x] = root_y
   |
   |-- SINON :
         parent[root_y] = root_x
         SI ranks égaux : incrémenter rank

4. RETOURNER true (union effectuée)
```

### 5.3 Visualisation ASCII

```
=== LES CHEMINS D'YMIR (THE PATHS) ===

État initial : 10 Eldiens isolés
┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
│ 0 │ │ 1 │ │ 2 │ │ 3 │ │ 4 │ │ 5 │ │ 6 │ │ 7 │ │ 8 │ │ 9 │
└───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘ └───┘
count = 10

=== Après union(0, 1) - Eren(0) et Mikasa(1) ===

    ┌───┐
    │ 1 │ ← root (rank=1)
    └─┬─┘
      │
    ┌─┴─┐
    │ 0 │
    └───┘

count = 9

=== Après union(2, 3) et union(4, 5) ===

    ┌───┐     ┌───┐     ┌───┐
    │ 1 │     │ 3 │     │ 5 │
    └─┬─┘     └─┬─┘     └─┬─┘
      │         │         │
    ┌─┴─┐     ┌─┴─┐     ┌─┴─┐
    │ 0 │     │ 2 │     │ 4 │
    └───┘     └───┘     └───┘

count = 7

=== Après union(0, 2) - Union by rank ===

         ┌───┐
         │ 1 │ ← root (rank reste 1)
       ┌─┴─┬─┴─┐
       │   │   │
     ┌─┴─┐│ ┌─┴─┐
     │ 0 ││ │ 3 │
     └───┘│ └─┬─┘
          │   │
        ┌─┴─┐ │
        │ 2 ├─┘ ← attaché via path compression future
        └───┘

count = 6

=== PATH COMPRESSION en action ===

Avant find(2):        Après find(2):
     [1]                   [1]
      │                 ┌───┼───┐
     [3]               [0] [3] [2]
      │
     [2]

Tous les nœuds traversés pointent maintenant directement vers la racine!

=== CYCLE DETECTION avec Union-Find ===

Graphe: 0 ─── 1 ─── 2
              │
              └───── 0  ← Cycle!

union(0, 1) → true, count = 2
union(1, 2) → true, count = 1
union(2, 0) → find(2)=find(0) → DÉJÀ CONNECTÉS → CYCLE DÉTECTÉ!

=== KRUSKAL's MST ===

Edges triées par poids:
(6,7,1), (5,6,2), (2,8,2), (0,1,4), (2,5,4), (2,3,7), ...

Étape 1: union(6,7) ─── poids 1 ✓
Étape 2: union(5,6) ─── poids 2 ✓
Étape 3: union(2,8) ─── poids 2 ✓
Étape 4: union(0,1) ─── poids 4 ✓
...
Étape n-1: Arbre couvrant minimal complet!

Total = 37
```

### 5.4 Les pièges en détail

| Piège | Symptôme | Solution |
|-------|----------|----------|
| Pas de path compression | Timeout sur grands inputs | `parent[x] = find(parent[x])` |
| Union sans rank | Arbres très hauts | Comparer ranks avant attacher |
| Self-union retourne true | count incorrect | Check `root_x == root_y` |
| Oublie décrémenter count | count() toujours = n | `count -= 1` dans union |
| Index out of bounds | Panic | `if x >= n { return x; }` |

### 5.5 Cours Complet

#### 5.5.1 Qu'est-ce que Union-Find ?

**Union-Find** (aussi appelé **Disjoint Set Union** ou **DSU**) est une structure de données qui gère une partition d'un ensemble en sous-ensembles disjoints.

Opérations principales :
- **MakeSet(x)** : Créer un singleton {x}
- **Find(x)** : Trouver le représentant (racine) du set contenant x
- **Union(x, y)** : Fusionner les sets contenant x et y

#### 5.5.2 Représentation par forêt

Chaque set est représenté comme un arbre où :
- Chaque nœud pointe vers son parent
- La racine pointe vers elle-même
- La racine est le "représentant" du set

```rust
struct UnionFind {
    parent: Vec<usize>,  // parent[x] = parent de x
    // ...
}

// Initialisation: chaque élément est sa propre racine
fn new(n: usize) -> Self {
    Self {
        parent: (0..n).collect(),  // parent[i] = i
    }
}
```

#### 5.5.3 Path Compression

**Problème** : Sans optimisation, `find` peut être O(n) si l'arbre est une chaîne.

**Solution** : Pendant le find, faire pointer tous les nœuds traversés directement vers la racine.

```rust
// Sans compression (O(n))
fn find_slow(&self, x: usize) -> usize {
    if self.parent[x] == x { x }
    else { self.find_slow(self.parent[x]) }
}

// Avec compression (O(α(n)))
fn find(&mut self, x: usize) -> usize {
    if self.parent[x] != x {
        self.parent[x] = self.find(self.parent[x]);  // ← Compression!
    }
    self.parent[x]
}
```

#### 5.5.4 Union by Rank

**Problème** : Si on attache toujours arbitrairement, on peut créer des arbres déséquilibrés.

**Solution** : Toujours attacher le plus petit arbre sous le plus grand.

```rust
fn union(&mut self, x: usize, y: usize) -> bool {
    let root_x = self.find(x);
    let root_y = self.find(y);

    if root_x == root_y { return false; }

    // Attacher le plus petit sous le plus grand
    if self.rank[root_x] < self.rank[root_y] {
        self.parent[root_x] = root_y;
    } else {
        self.parent[root_y] = root_x;
        if self.rank[root_x] == self.rank[root_y] {
            self.rank[root_x] += 1;  // Hauteur augmente seulement si égaux
        }
    }
    true
}
```

#### 5.5.5 La fonction d'Ackermann inverse

La complexité combinée est O(α(n)) où α est la **fonction d'Ackermann inverse**.

| n | α(n) |
|---|------|
| 1 | 0 |
| 2 | 1 |
| 4 | 2 |
| 16 | 3 |
| 65536 | 4 |
| 2^65536 | 5 |

Pour tout n pratiquement réalisable, α(n) ≤ 4. C'est **quasi-constant**.

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ fn find(&self, x: usize) -> usize { ... }  // Pas de mut       │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ fn find(&mut self, x: usize) -> usize { ... }  // mut requis   │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Path compression MODIFIE la structure (parent[x] = root)     │
│ • Sans mut, pas de compression = O(n)                          │
│ • C'est un cas de "mutation logiquement invisible"             │
│ • L'observable (résultat) est le même, mais l'état interne     │
│   change pour optimiser les futures requêtes                   │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario :** `union(0,1), union(2,3), union(0,2), connected(1,3)`

```
┌───────┬────────────────────────────┬────────────────────────────────────────────┐
│ Étape │ Opération                  │ État (parent, rank, count)                 │
├───────┼────────────────────────────┼────────────────────────────────────────────┤
│   0   │ new(5)                     │ parent=[0,1,2,3,4], rank=[0,0,0,0,0], c=5  │
├───────┼────────────────────────────┼────────────────────────────────────────────┤
│   1   │ union(0, 1)                │ find(0)=0, find(1)=1                       │
│       │ → attache 0 sous 1         │ parent=[1,1,2,3,4], rank=[0,1,0,0,0], c=4  │
├───────┼────────────────────────────┼────────────────────────────────────────────┤
│   2   │ union(2, 3)                │ find(2)=2, find(3)=3                       │
│       │ → attache 2 sous 3         │ parent=[1,1,3,3,4], rank=[0,1,0,1,0], c=3  │
├───────┼────────────────────────────┼────────────────────────────────────────────┤
│   3   │ union(0, 2)                │ find(0)=1, find(2)=3 (compression: 2→3)    │
│       │ → attache 3 sous 1         │ parent=[1,1,3,1,4], rank=[0,1,0,1,0], c=2  │
│       │ (ranks égaux, 1 incrémenté)│                                            │
├───────┼────────────────────────────┼────────────────────────────────────────────┤
│   4   │ connected(1, 3)            │ find(1)=1, find(3)=1                       │
│       │                            │ 1 == 1 → VRAI, ils sont connectés!         │
└───────┴────────────────────────────┴────────────────────────────────────────────┘
```

### 5.8 Mnémotechniques

#### 🔥 MEME : "All Subjects of Ymir are connected through the Paths"

Dans Attack on Titan, tous les Eldiens partagent une connexion mystique via les Chemins. Union-Find fait exactement la même chose : `connected(x, y)` vérifie si x et y partagent le même ancêtre (la même racine).

```rust
pub fn connected(&mut self, eren: usize, mikasa: usize) -> bool {
    // "Sommes-nous connectés par les Chemins ?"
    self.find(eren) == self.find(mikasa)
}
```

#### ⚡ MEME : "The Founding Titan can access all memories"

Le Titan Fondateur est la "racine" de tous les Eldiens. `find(x)` remonte les Chemins jusqu'à trouver cette racine.

```
Eldien random → parent → parent → ... → FOUNDING TITAN (racine)
```

#### 🧠 MEME : "Eren shortcuts through the Paths"

Quand Eren accède aux Chemins, il peut communiquer directement avec Ymir sans passer par les intermédiaires. C'est exactement **path compression** :

```rust
// Avant: 0 → 1 → 2 → 3 → 4 (racine)
// Après find(0):
// 0 → 4, 1 → 4, 2 → 4, 3 → 4 (tous pointent direct vers racine)
```

### 5.9 Applications pratiques

1. **Kruskal's MST** : Trier les arêtes par poids, ajouter si elles ne créent pas de cycle
2. **Détection de cycles** : Si `find(u) == find(v)` avant union → cycle
3. **Composantes connexes** : `count()` = nombre de composantes
4. **Percolation** : Simulation physique (fluides traversant une grille)
5. **Clustering** : Regroupement hiérarchique de données

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Test qui l'attrape |
|---|-------|--------|-------------------|
| 1 | Pas de path compression | O(n) timeout | test_large_scale |
| 2 | Union sans rank | Arbres déséquilibrés | test_chain |
| 3 | Self-union retourne true | count incorrect | test_self_union |
| 4 | Oublie décrémenter count | count() bug | test_count |
| 5 | Index >= n panic | Crash | test_bounds |

---

## 📝 SECTION 7 : QCM

### Question 1
Quelle est la complexité amortie de find avec path compression + union by rank ?

- A) O(1)
- B) O(log n)
- C) O(α(n)) ≈ O(1) ✓
- D) O(n)

### Question 2
Que fait path compression ?

- A) Compresse les données pour économiser de la mémoire
- B) Fait pointer tous les nœuds traversés directement vers la racine ✓
- C) Trie les éléments par ordre croissant
- D) Supprime les doublons

### Question 3
Que retourne `union(x, y)` si x et y sont déjà dans le même composant ?

- A) true
- B) false ✓
- C) L'indice de la racine commune
- D) Une erreur

### Question 4
Après `new(10)`, que vaut `count()` ?

- A) 0
- B) 1
- C) 10 ✓
- D) Indéfini

### Question 5
Pourquoi utiliser union by rank ?

- A) Pour économiser de la mémoire
- B) Pour garder les arbres équilibrés ✓
- C) Pour accélérer le tri
- D) Pour détecter les cycles plus vite

### Question 6
Quel algorithme de MST utilise Union-Find ?

- A) Dijkstra
- B) Prim
- C) Kruskal ✓
- D) Floyd-Warshall

### Question 7
Comment détecter un cycle avec Union-Find ?

- A) Si count() == 1
- B) Si une arête connecte deux nœuds déjà dans le même composant ✓
- C) Si find(x) == x pour tout x
- D) Si rank > log(n)

### Question 8
Que vaut α(10^80) (nombre d'atomes dans l'univers) ?

- A) 10^80
- B) 80
- C) Environ 5 ✓
- D) 1

### Question 9
Dans union by rank, quand incrémente-t-on le rank ?

- A) À chaque union
- B) Quand on attache un arbre plus petit
- C) Quand les deux arbres ont le même rank ✓
- D) Jamais

### Question 10
Quel est l'espace mémoire utilisé par Union-Find pour n éléments ?

- A) O(1)
- B) O(log n)
- C) O(n) ✓
- D) O(n²)

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| Exercice | 1.4.1-a : paths_of_ymir |
| Thème | Attack on Titan - Les Chemins (The Paths) |
| Concepts | Union-Find, path compression, union by rank |
| Difficulté Base | ★★★★☆☆☆☆☆☆ (4/10) |
| Bonus Standard | ★★★★★☆☆☆☆☆ (5/10) — Weighted UF |
| Bonus Expert | ★★★★★★★☆☆☆ (7/10) — Rollback UF |
| Bonus Génie | 🧠 (12/10) — Persistent UF |
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
    "exercise_slug": "1.4.1-a-paths-of-ymir",
    "generated_at": "2026-01-11",

    "metadata": {
      "exercise_id": "1.4.1-a",
      "exercise_name": "paths_of_ymir",
      "module": "1.4.1",
      "module_name": "Union-Find (Disjoint Set Union)",
      "concept": "a",
      "concept_name": "Basic Union-Find with optimizations",
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
      "complexity_time": "T1 O(α(n))",
      "complexity_space": "S1 O(n)",
      "prerequisites": ["vectors", "recursion", "structs"],
      "domains": ["Struct", "MD"],
      "domains_bonus": [],
      "tags": ["union-find", "dsu", "path-compression", "attack-on-titan"],
      "meme_reference": "All Subjects of Ymir are connected through the Paths"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "alternatives/alt_iterative.rs": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_no_compression.rs": "/* Section 4.10 */",
      "mutants/mutant_c_no_count.rs": "/* Section 4.10 */",
      "mutants/mutant_d_wrong_rank.rs": "/* Section 4.10 */",
      "mutants/mutant_e_always_true.rs": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "references/ref_solution_bonus.rs",
        "alternatives/alt_iterative.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_no_compression.rs",
        "mutants/mutant_c_no_count.rs",
        "mutants/mutant_d_wrong_rank.rs",
        "mutants/mutant_e_always_true.rs"
      ]
    },

    "commands": {
      "validate_spec": "python3 hackbrain_engine_v22.py --validate-spec spec.json",
      "test_reference": "cargo test --lib",
      "test_c": "gcc -Wall -Wextra -Werror -std=c17 paths_of_ymir.c main.c -o test && ./test",
      "test_mutants": "python3 hackbrain_mutation_tester.py -r references/ref_solution.rs -s spec.json --validate"
    }
  }
}
```

---

*"Tatakae. Tatakae. TATAKAE!"*

*— Eren Yeager, while fighting through the Paths*

---

*HACKBRAIN v5.5.2 — L'excellence pédagogique ne se négocie pas*
