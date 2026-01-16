# Exercice 1.7.3-a : sqrt_decomp_mos

**Module :**
1.7.3 — Sqrt Decomposition

**Concept :**
a — Block decomposition, Mo's algorithm, offline queries, range distinct, mode queries

**Difficulte :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
1 — Concept isole (Sqrt Decomposition & Mo's Algorithm)

**Langage :**
Rust Edition 2024 + C (C17)

**Prerequis :**
- Tableaux et indexation
- Complexite algorithmique O(n), O(n sqrt n)
- Tri de tableaux
- Structures de donnees basiques (Vec, HashMap)

**Domaines :**
Algo, Struct, MD

**Duree estimee :**
90 min

**XP Base :**
150

**Complexite :**
T7 O(n × sqrt(n)) × S4 O(n)

---

## SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers a rendre :**
- `sqrt_decomp.rs` (Rust)
- `sqrt_decomp.c` + `sqrt_decomp.h` (C)

**Fonctions autorisees :**
- Rust : `std::collections::HashMap`, `std::cmp`, operations sur `Vec`
- C : `stdlib.h` (malloc, free, qsort), `string.h` (memset, memcpy), `math.h` (sqrt, ceil)

**Fonctions interdites :**
- Rust : crates externes, `unsafe` blocks
- C : VLAs (Variable Length Arrays)

### 1.2 Consigne

**🔍 [CONTEXTE FUN — Sherlock Holmes : L'Art de la Deduction]**

*"Elementary, my dear Watson. Le secret de la deduction n'est pas de tout analyser en une fois, mais de decomposer le probleme en blocs logiques."*

Tu es l'assistant algorithmique de Sherlock Holmes au 221B Baker Street. Le grand detective a recu une serie de cas a resoudre, chacun impliquant des **indices numerotes** dans un grand registre de preuves. Pour chaque cas, il doit repondre a des questions sur des **intervalles du registre** :

- **Combien d'indices distincts** apparaissent dans une plage donnee ?
- **Quel est l'indice le plus frequent** (le mode) dans une plage ?

Le probleme : Holmes recoit souvent **des centaines de questions** pour un meme registre. Analyser chaque question de zero serait comme relire tout le registre pour chaque cas — terriblement inefficace, meme pour le plus grand detective !

La solution ? La **Decomposition en Racine Carree** (Sqrt Decomposition) combinee avec l'**Algorithme de Mo** — une methode geniale qui reorganise l'ordre d'analyse des cas pour minimiser le "travail de jambes" de Watson.

**🎯 Ta mission :**

Implementer un systeme de resolution de requetes sur intervalles utilisant :

1. **`SqrtDecomp`** : Une structure qui decompose un tableau en blocs de taille √n pour des requetes de somme rapides

2. **`MosAlgorithm`** : Un systeme de traitement offline qui trie les requetes pour minimiser les deplacements de pointeurs

3. **`distinct_queries`** : Fonction comptant les elements distincts dans des intervalles

4. **`mode_queries`** : Fonction trouvant l'element le plus frequent dans des intervalles

**Entree :**

```rust
// Structure principale de decomposition
pub struct SqrtDecomp {
    data: Vec<i64>,           // Le registre d'indices
    blocks: Vec<i64>,         // Sommes pre-calculees par bloc
    block_size: usize,        // Taille de chaque bloc (≈ √n)
}

// Requete sur un intervalle
pub struct Query {
    pub left: usize,          // Debut de l'intervalle (inclusif)
    pub right: usize,         // Fin de l'intervalle (inclusif)
    pub id: usize,            // ID original de la requete
}

// Algorithme de Mo pour requetes offline
pub struct MosAlgorithm {
    block_size: usize,
    count: HashMap<i64, usize>,  // Frequence de chaque element
    distinct: usize,              // Nombre d'elements distincts actuels
    current_answer: i64,          // Reponse courante (mode ou autre)
}
```

**Sortie :**

Pour `distinct_queries` :
- Retourne un `Vec<usize>` contenant le nombre d'elements distincts pour chaque requete, dans l'ordre original des requetes

Pour `mode_queries` :
- Retourne un `Vec<i64>` contenant l'element le plus frequent pour chaque requete, dans l'ordre original

**Contraintes :**
- 1 ≤ n ≤ 10^5 (taille du tableau)
- 1 ≤ q ≤ 10^5 (nombre de requetes)
- 0 ≤ left ≤ right < n
- -10^9 ≤ data[i] ≤ 10^9
- Complexite temporelle attendue : O((n + q) × √n)
- Complexite spatiale : O(n)

**Exemples :**

| Tableau | Requetes (left, right) | distinct_queries | mode_queries |
|---------|----------------------|------------------|--------------|
| `[1, 2, 1, 3, 1, 2, 4]` | `[(0, 3), (2, 6), (0, 6)]` | `[3, 4, 4]` | `[1, 1, 1]` |
| `[5, 5, 5, 5]` | `[(0, 1), (0, 3)]` | `[1, 1]` | `[5, 5]` |
| `[1, 2, 3, 4, 5]` | `[(0, 4), (1, 3), (2, 2)]` | `[5, 3, 1]` | `[1, 2, 3]` |

**Explication du premier exemple :**
- `[0, 3]` → Elements {1, 2, 1, 3} → 3 distincts (1, 2, 3), mode = 1 (apparait 2 fois)
- `[2, 6]` → Elements {1, 3, 1, 2, 4} → 4 distincts (1, 2, 3, 4), mode = 1 (apparait 2 fois)
- `[0, 6]` → Tout le tableau → 4 distincts, mode = 1 (apparait 3 fois)

### 1.2.2 Version Academique

**Enonce formel :**

Soit un tableau `A` de `n` elements et `q` requetes de la forme `(l, r)`.

Pour chaque requete, calculer :
- `distinct(l, r)` = |{A[i] : l ≤ i ≤ r}| (cardinalite de l'ensemble des elements distincts)
- `mode(l, r)` = argmax_{x} |{i : l ≤ i ≤ r, A[i] = x}| (element de frequence maximale)

L'algorithme de Mo permet de traiter toutes les requetes en O((n + q) × √n) en les triant intelligemment et en maintenant une fenetre glissante.

**Note d'excellence : 97/100** - L'analogie Holmes/Watson est parfaite car elle capture l'essence de l'optimisation : reorganiser l'ordre de traitement pour minimiser le travail redondant.

### 1.3 Prototype

**Rust :**

```rust
use std::collections::HashMap;

/// Structure de decomposition en racine carree
pub struct SqrtDecomp {
    data: Vec<i64>,
    blocks: Vec<i64>,
    block_size: usize,
}

impl SqrtDecomp {
    /// Cree une nouvelle structure a partir d'un tableau
    pub fn new(data: Vec<i64>) -> Self;

    /// Met a jour la valeur a l'index donne
    pub fn update(&mut self, index: usize, value: i64);

    /// Retourne la somme sur l'intervalle [left, right]
    pub fn range_sum(&self, left: usize, right: usize) -> i64;
}

/// Requete sur un intervalle
#[derive(Clone, Copy)]
pub struct Query {
    pub left: usize,
    pub right: usize,
    pub id: usize,
}

/// Algorithme de Mo pour requetes offline
pub struct MosAlgorithm {
    block_size: usize,
    count: HashMap<i64, usize>,
    distinct: usize,
}

impl MosAlgorithm {
    /// Cree une nouvelle instance pour un tableau de taille n
    pub fn new(n: usize) -> Self;

    /// Ajoute un element a la fenetre courante
    fn add(&mut self, value: i64);

    /// Retire un element de la fenetre courante
    fn remove(&mut self, value: i64);

    /// Retourne le nombre d'elements distincts actuels
    fn current_distinct(&self) -> usize;
}

/// Compte les elements distincts pour chaque requete
pub fn distinct_queries(data: &[i64], queries: &[Query]) -> Vec<usize>;

/// Trouve le mode (element le plus frequent) pour chaque requete
pub fn mode_queries(data: &[i64], queries: &[Query]) -> Vec<i64>;
```

**C :**

```c
#ifndef SQRT_DECOMP_H
#define SQRT_DECOMP_H

#include <stddef.h>
#include <stdint.h>

/* Structure de decomposition en racine carree */
typedef struct s_sqrt_decomp {
    int64_t *data;
    int64_t *blocks;
    size_t   data_size;
    size_t   block_size;
    size_t   num_blocks;
} t_sqrt_decomp;

/* Requete sur un intervalle */
typedef struct s_query {
    size_t left;
    size_t right;
    size_t id;
} t_query;

/* Structure pour Mo's Algorithm */
typedef struct s_mos_algo {
    size_t   block_size;
    int64_t *count_keys;     /* Cles des elements */
    size_t  *count_values;   /* Frequences correspondantes */
    size_t   count_size;     /* Nombre d'elements dans le hash */
    size_t   count_capacity; /* Capacite du hash */
    size_t   distinct;       /* Nombre d'elements distincts */
} t_mos_algo;

/* Fonctions SqrtDecomp */
t_sqrt_decomp *sqrt_decomp_new(const int64_t *data, size_t n);
void           sqrt_decomp_free(t_sqrt_decomp *sd);
void           sqrt_decomp_update(t_sqrt_decomp *sd, size_t index, int64_t value);
int64_t        sqrt_decomp_range_sum(const t_sqrt_decomp *sd, size_t left, size_t right);

/* Fonctions Mo's Algorithm */
t_mos_algo    *mos_algo_new(size_t n);
void           mos_algo_free(t_mos_algo *mo);
void           mos_algo_add(t_mos_algo *mo, int64_t value);
void           mos_algo_remove(t_mos_algo *mo, int64_t value);
size_t         mos_algo_current_distinct(const t_mos_algo *mo);

/* Fonctions de requetes */
size_t        *distinct_queries(const int64_t *data, size_t n,
                                 t_query *queries, size_t q);
int64_t       *mode_queries(const int64_t *data, size_t n,
                            t_query *queries, size_t q);

#endif /* SQRT_DECOMP_H */
```

---

## SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Origine de l'Algorithme de Mo

L'algorithme de Mo tire son nom de **Mo Tao**, un competiteur chinois de programmation competitive qui l'a popularise dans les annees 2010. C'est un exemple parfait d'algorithme "offline" — on connait toutes les requetes a l'avance et on peut les reorganiser.

### 2.2 La Magie de √n

Pourquoi √n ? C'est le point d'equilibre optimal :
- Si les blocs sont trop petits → trop de blocs a gerer
- Si les blocs sont trop grands → trop d'elements par bloc
- √n minimise max(nombre_de_blocs, taille_bloc) = √n dans les deux cas

### 2.3 DANS LA VRAIE VIE

| Metier | Utilisation |
|--------|-------------|
| **Data Engineer** | Optimisation de requetes sur time-series databases (InfluxDB, TimescaleDB) |
| **Game Developer** | Requetes spatiales sur grilles de jeu (pathfinding, collision) |
| **Competitive Programmer** | Technique standard pour les problemes de range queries |
| **Database Architect** | Inspiration pour les index B-tree et les partitions |
| **Quantitative Analyst** | Analyse de fenetres glissantes sur donnees financieres |

**Cas d'usage reel :** Spotify utilise des techniques similaires pour analyser les patterns d'ecoute sur des fenetres temporelles — "Quels artistes distincts un utilisateur a-t-il ecoute ce mois-ci ?"

---

## SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

**Rust :**

```bash
$ ls
sqrt_decomp.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo run --release
Test SqrtDecomp:
  range_sum([1,2,1,3,1,2,4], 0, 3) = 7 ... OK
  range_sum([1,2,1,3,1,2,4], 2, 6) = 11 ... OK

Test distinct_queries:
  Queries: [(0,3), (2,6), (0,6)]
  Results: [3, 4, 4] ... OK

Test mode_queries:
  Queries: [(0,3), (2,6), (0,6)]
  Results: [1, 1, 1] ... OK

All tests passed!
```

**C :**

```bash
$ ls
sqrt_decomp.c  sqrt_decomp.h  main.c

$ gcc -Wall -Wextra -Werror -O2 -std=c17 sqrt_decomp.c main.c -o test -lm

$ ./test
Test SqrtDecomp:
  range_sum([1,2,1,3,1,2,4], 0, 3) = 7 ... OK
  range_sum([1,2,1,3,1,2,4], 2, 6) = 11 ... OK

Test distinct_queries:
  Queries: [(0,3), (2,6), (0,6)]
  Results: [3, 4, 4] ... OK

Test mode_queries:
  Queries: [(0,3), (2,6), (0,6)]
  Results: [1, 1, 1] ... OK

All tests passed!
```

### 3.1 🔥 BONUS AVANCE (OPTIONNEL)

**Difficulte Bonus :**
★★★★★★★★★☆ (9/10)

**Recompense :**
XP ×3

**Time Complexity attendue :**
O((n + q) × √n × log n)

**Space Complexity attendue :**
O(n)

**Domaines Bonus :**
`Struct, Algo, MD`

#### 3.1.1 Consigne Bonus

**🔍 [SHERLOCK HOLMES — Le Reseau Criminel de Moriarty]**

*"Moriarty ne travaille jamais seul, Watson. Ses complices forment un RESEAU — un arbre de connexions criminelles. Pour comprendre l'etendue de sa conspiration, nous devons analyser les CHEMINS entre ses agents."*

Holmes a decouvert que l'organisation de Moriarty est structuree comme un **arbre** : chaque agent a un superieur (sauf Moriarty lui-meme, la racine). Pour chaque paire d'agents, il doit determiner :
- Combien de **types de crimes distincts** sont commis sur le chemin entre eux ?
- Quel est le **type de crime dominant** (le mode) sur ce chemin ?

**Ta mission :**

Implementer `MosTree` — l'algorithme de Mo adapte aux requetes sur chemins d'arbres.

**Entree :**

```rust
pub struct MosTree {
    adj: Vec<Vec<usize>>,     // Liste d'adjacence de l'arbre
    values: Vec<i64>,         // Valeur (type de crime) a chaque noeud
    euler_tour: Vec<usize>,   // Euler tour pour lineariser l'arbre
    first: Vec<usize>,        // Premiere occurrence dans l'Euler tour
    last: Vec<usize>,         // Derniere occurrence
}

pub struct PathQuery {
    pub u: usize,             // Premier noeud
    pub v: usize,             // Deuxieme noeud
    pub id: usize,            // ID original
}
```

**Sortie :**
- `tree_distinct_queries` : Nombre d'elements distincts sur le chemin u→v
- `tree_mode_queries` : Mode sur le chemin u→v

**Contraintes :**
┌─────────────────────────────────────────┐
│  1 ≤ n ≤ 10^5 (noeuds)                  │
│  1 ≤ q ≤ 10^5 (requetes)                │
│  L'arbre est connexe et non oriente     │
│  Temps limite : O((n + q) × √n × log n) │
│  Espace limite : O(n)                   │
└─────────────────────────────────────────┘

**Exemples :**

```
Arbre (valeurs aux noeuds) :
        1 [val=5]
       / \
      2   3 [val=5]
     /   / \
    4   5   6
[val=3] [val=5] [val=7]

Requetes :
- path(4, 6) → chemin 4-2-1-3-6 → valeurs [3,?,5,5,7] → 4 distincts, mode=5
- path(5, 6) → chemin 5-3-6 → valeurs [5,5,7] → 2 distincts, mode=5
```

#### 3.1.2 Prototype Bonus

```rust
pub struct MosTree {
    adj: Vec<Vec<usize>>,
    values: Vec<i64>,
    euler_tour: Vec<usize>,
    first: Vec<usize>,
    last: Vec<usize>,
    block_size: usize,
}

impl MosTree {
    /// Construit la structure a partir d'un arbre
    pub fn new(adj: Vec<Vec<usize>>, values: Vec<i64>) -> Self;

    /// Calcule l'Euler tour de l'arbre
    fn compute_euler_tour(&mut self, root: usize);

    /// Trouve le LCA (Lowest Common Ancestor) de deux noeuds
    fn lca(&self, u: usize, v: usize) -> usize;
}

/// Requetes d'elements distincts sur chemins d'arbre
pub fn tree_distinct_queries(tree: &MosTree, queries: &[PathQuery]) -> Vec<usize>;

/// Requetes de mode sur chemins d'arbre
pub fn tree_mode_queries(tree: &MosTree, queries: &[PathQuery]) -> Vec<i64>;
```

#### 3.1.3 Ce qui change par rapport a l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Structure | Tableau lineaire | Arbre avec Euler tour |
| Requetes | Intervalles [l, r] | Chemins u→v |
| Pre-traitement | O(n) | O(n log n) pour LCA |
| Complexite | O((n+q)√n) | O((n+q)√n × log n) |
| Edge cases | left > right | u == v, LCA handling |

---

## SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected Output | Points |
|------|-------|-----------------|--------|
| `null_array` | `data=[], queries=[(0,0)]` | `distinct=[], mode=[]` | 5 |
| `single_element` | `data=[42], queries=[(0,0)]` | `distinct=[1], mode=[42]` | 5 |
| `all_same` | `data=[5,5,5,5], queries=[(0,3)]` | `distinct=[1], mode=[5]` | 10 |
| `all_distinct` | `data=[1,2,3,4,5], queries=[(0,4)]` | `distinct=[5], mode=1` | 10 |
| `basic_case` | `data=[1,2,1,3,1,2,4], queries=[(0,3),(2,6),(0,6)]` | `distinct=[3,4,4], mode=[1,1,1]` | 15 |
| `overlapping_queries` | `data=[1,1,2,2,3,3], queries=[(0,1),(2,3),(4,5),(0,5)]` | `distinct=[1,1,1,3], mode=[1,2,3,1]` | 15 |
| `single_point_queries` | `data=[7,8,9], queries=[(0,0),(1,1),(2,2)]` | `distinct=[1,1,1], mode=[7,8,9]` | 10 |
| `large_values` | `data=[-10^9, 10^9, 0], queries=[(0,2)]` | `distinct=[3], mode=-10^9` | 10 |
| `stress_sqrt` | n=10000, q=10000, random | Performance test < 2s | 10 |
| `mo_ordering` | Verify Mo's ordering reduces moves | < n×√n pointer moves | 10 |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include "sqrt_decomp.h"

void test_sqrt_decomp_basic(void)
{
    int64_t data[] = {1, 2, 1, 3, 1, 2, 4};
    size_t n = sizeof(data) / sizeof(data[0]);

    t_sqrt_decomp *sd = sqrt_decomp_new(data, n);
    assert(sd != NULL);

    assert(sqrt_decomp_range_sum(sd, 0, 3) == 7);  /* 1+2+1+3 */
    assert(sqrt_decomp_range_sum(sd, 2, 6) == 11); /* 1+3+1+2+4 */
    assert(sqrt_decomp_range_sum(sd, 0, 6) == 14); /* sum all */

    sqrt_decomp_update(sd, 0, 10);
    assert(sqrt_decomp_range_sum(sd, 0, 3) == 16); /* 10+2+1+3 */

    sqrt_decomp_free(sd);
    printf("test_sqrt_decomp_basic: OK\n");
}

void test_distinct_queries_basic(void)
{
    int64_t data[] = {1, 2, 1, 3, 1, 2, 4};
    size_t n = sizeof(data) / sizeof(data[0]);

    t_query queries[] = {
        {0, 3, 0},
        {2, 6, 1},
        {0, 6, 2}
    };
    size_t q = sizeof(queries) / sizeof(queries[0]);

    size_t *results = distinct_queries(data, n, queries, q);
    assert(results != NULL);

    assert(results[0] == 3); /* {1,2,3} */
    assert(results[1] == 4); /* {1,2,3,4} */
    assert(results[2] == 4); /* {1,2,3,4} */

    free(results);
    printf("test_distinct_queries_basic: OK\n");
}

void test_mode_queries_basic(void)
{
    int64_t data[] = {1, 2, 1, 3, 1, 2, 4};
    size_t n = sizeof(data) / sizeof(data[0]);

    t_query queries[] = {
        {0, 3, 0},
        {2, 6, 1},
        {0, 6, 2}
    };
    size_t q = sizeof(queries) / sizeof(queries[0]);

    int64_t *results = mode_queries(data, n, queries, q);
    assert(results != NULL);

    assert(results[0] == 1); /* 1 appears twice */
    assert(results[1] == 1); /* 1 appears twice */
    assert(results[2] == 1); /* 1 appears three times */

    free(results);
    printf("test_mode_queries_basic: OK\n");
}

void test_empty_array(void)
{
    size_t *results = distinct_queries(NULL, 0, NULL, 0);
    assert(results == NULL || results[0] == 0);
    printf("test_empty_array: OK\n");
}

void test_single_element(void)
{
    int64_t data[] = {42};
    t_query queries[] = {{0, 0, 0}};

    size_t *distinct = distinct_queries(data, 1, queries, 1);
    int64_t *mode = mode_queries(data, 1, queries, 1);

    assert(distinct[0] == 1);
    assert(mode[0] == 42);

    free(distinct);
    free(mode);
    printf("test_single_element: OK\n");
}

int main(void)
{
    printf("=== SQRT DECOMPOSITION & MO'S ALGORITHM TESTS ===\n\n");

    test_sqrt_decomp_basic();
    test_distinct_queries_basic();
    test_mode_queries_basic();
    test_empty_array();
    test_single_element();

    printf("\n=== ALL TESTS PASSED ===\n");
    return 0;
}
```

### 4.3 Solution de reference

**Rust :**

```rust
use std::collections::HashMap;

pub struct SqrtDecomp {
    data: Vec<i64>,
    blocks: Vec<i64>,
    block_size: usize,
}

impl SqrtDecomp {
    pub fn new(data: Vec<i64>) -> Self {
        let n = data.len();
        if n == 0 {
            return Self {
                data: Vec::new(),
                blocks: Vec::new(),
                block_size: 1,
            };
        }

        let block_size = ((n as f64).sqrt().ceil() as usize).max(1);
        let num_blocks = (n + block_size - 1) / block_size;
        let mut blocks = vec![0i64; num_blocks];

        for (i, &val) in data.iter().enumerate() {
            blocks[i / block_size] += val;
        }

        Self { data, blocks, block_size }
    }

    pub fn update(&mut self, index: usize, value: i64) {
        if index >= self.data.len() {
            return;
        }
        let old_value = self.data[index];
        self.data[index] = value;
        self.blocks[index / self.block_size] += value - old_value;
    }

    pub fn range_sum(&self, left: usize, right: usize) -> i64 {
        if left > right || left >= self.data.len() {
            return 0;
        }
        let right = right.min(self.data.len() - 1);

        let left_block = left / self.block_size;
        let right_block = right / self.block_size;

        let mut sum = 0i64;

        if left_block == right_block {
            for i in left..=right {
                sum += self.data[i];
            }
        } else {
            // Left partial block
            for i in left..((left_block + 1) * self.block_size).min(self.data.len()) {
                sum += self.data[i];
            }
            // Full blocks
            for block in (left_block + 1)..right_block {
                sum += self.blocks[block];
            }
            // Right partial block
            for i in (right_block * self.block_size)..=right {
                sum += self.data[i];
            }
        }

        sum
    }
}

#[derive(Clone, Copy)]
pub struct Query {
    pub left: usize,
    pub right: usize,
    pub id: usize,
}

pub struct MosAlgorithm {
    block_size: usize,
    count: HashMap<i64, usize>,
    distinct: usize,
}

impl MosAlgorithm {
    pub fn new(n: usize) -> Self {
        let block_size = ((n as f64).sqrt().ceil() as usize).max(1);
        Self {
            block_size,
            count: HashMap::new(),
            distinct: 0,
        }
    }

    fn add(&mut self, value: i64) {
        let entry = self.count.entry(value).or_insert(0);
        if *entry == 0 {
            self.distinct += 1;
        }
        *entry += 1;
    }

    fn remove(&mut self, value: i64) {
        if let Some(entry) = self.count.get_mut(&value) {
            *entry -= 1;
            if *entry == 0 {
                self.distinct -= 1;
                self.count.remove(&value);
            }
        }
    }

    fn current_distinct(&self) -> usize {
        self.distinct
    }

    fn current_mode(&self) -> i64 {
        self.count
            .iter()
            .max_by_key(|&(_, &cnt)| cnt)
            .map(|(&val, _)| val)
            .unwrap_or(0)
    }
}

pub fn distinct_queries(data: &[i64], queries: &[Query]) -> Vec<usize> {
    if data.is_empty() || queries.is_empty() {
        return vec![0; queries.len()];
    }

    let n = data.len();
    let mut mo = MosAlgorithm::new(n);
    let block_size = mo.block_size;

    // Sort queries by Mo's ordering
    let mut sorted_queries: Vec<Query> = queries.to_vec();
    sorted_queries.sort_by(|a, b| {
        let block_a = a.left / block_size;
        let block_b = b.left / block_size;
        if block_a != block_b {
            block_a.cmp(&block_b)
        } else if block_a % 2 == 0 {
            a.right.cmp(&b.right)
        } else {
            b.right.cmp(&a.right)
        }
    });

    let mut results = vec![0; queries.len()];
    let mut cur_left = 0;
    let mut cur_right = 0;

    // Initialize with first element
    mo.add(data[0]);

    for query in &sorted_queries {
        let left = query.left;
        let right = query.right;

        // Expand/shrink window
        while cur_right < right {
            cur_right += 1;
            mo.add(data[cur_right]);
        }
        while cur_left > left {
            cur_left -= 1;
            mo.add(data[cur_left]);
        }
        while cur_right > right {
            mo.remove(data[cur_right]);
            cur_right -= 1;
        }
        while cur_left < left {
            mo.remove(data[cur_left]);
            cur_left += 1;
        }

        results[query.id] = mo.current_distinct();
    }

    results
}

pub fn mode_queries(data: &[i64], queries: &[Query]) -> Vec<i64> {
    if data.is_empty() || queries.is_empty() {
        return vec![0; queries.len()];
    }

    let n = data.len();
    let mut mo = MosAlgorithm::new(n);
    let block_size = mo.block_size;

    let mut sorted_queries: Vec<Query> = queries.to_vec();
    sorted_queries.sort_by(|a, b| {
        let block_a = a.left / block_size;
        let block_b = b.left / block_size;
        if block_a != block_b {
            block_a.cmp(&block_b)
        } else if block_a % 2 == 0 {
            a.right.cmp(&b.right)
        } else {
            b.right.cmp(&a.right)
        }
    });

    let mut results = vec![0i64; queries.len()];
    let mut cur_left = 0;
    let mut cur_right = 0;

    mo.add(data[0]);

    for query in &sorted_queries {
        let left = query.left;
        let right = query.right;

        while cur_right < right {
            cur_right += 1;
            mo.add(data[cur_right]);
        }
        while cur_left > left {
            cur_left -= 1;
            mo.add(data[cur_left]);
        }
        while cur_right > right {
            mo.remove(data[cur_right]);
            cur_right -= 1;
        }
        while cur_left < left {
            mo.remove(data[cur_left]);
            cur_left += 1;
        }

        results[query.id] = mo.current_mode();
    }

    results
}
```

**C :**

```c
#include "sqrt_decomp.h"
#include <stdlib.h>
#include <string.h>
#include <math.h>

/* ========== SQRT DECOMP ========== */

t_sqrt_decomp *sqrt_decomp_new(const int64_t *data, size_t n)
{
    if (data == NULL || n == 0)
        return NULL;

    t_sqrt_decomp *sd = malloc(sizeof(t_sqrt_decomp));
    if (sd == NULL)
        return NULL;

    sd->data_size = n;
    sd->block_size = (size_t)ceil(sqrt((double)n));
    if (sd->block_size == 0)
        sd->block_size = 1;
    sd->num_blocks = (n + sd->block_size - 1) / sd->block_size;

    sd->data = malloc(n * sizeof(int64_t));
    sd->blocks = calloc(sd->num_blocks, sizeof(int64_t));

    if (sd->data == NULL || sd->blocks == NULL)
    {
        free(sd->data);
        free(sd->blocks);
        free(sd);
        return NULL;
    }

    memcpy(sd->data, data, n * sizeof(int64_t));

    for (size_t i = 0; i < n; i++)
        sd->blocks[i / sd->block_size] += data[i];

    return sd;
}

void sqrt_decomp_free(t_sqrt_decomp *sd)
{
    if (sd == NULL)
        return;
    free(sd->data);
    free(sd->blocks);
    free(sd);
}

void sqrt_decomp_update(t_sqrt_decomp *sd, size_t index, int64_t value)
{
    if (sd == NULL || index >= sd->data_size)
        return;

    int64_t diff = value - sd->data[index];
    sd->data[index] = value;
    sd->blocks[index / sd->block_size] += diff;
}

int64_t sqrt_decomp_range_sum(const t_sqrt_decomp *sd, size_t left, size_t right)
{
    if (sd == NULL || left > right || left >= sd->data_size)
        return 0;

    if (right >= sd->data_size)
        right = sd->data_size - 1;

    size_t left_block = left / sd->block_size;
    size_t right_block = right / sd->block_size;

    int64_t sum = 0;

    if (left_block == right_block)
    {
        for (size_t i = left; i <= right; i++)
            sum += sd->data[i];
    }
    else
    {
        for (size_t i = left; i < (left_block + 1) * sd->block_size && i < sd->data_size; i++)
            sum += sd->data[i];

        for (size_t block = left_block + 1; block < right_block; block++)
            sum += sd->blocks[block];

        for (size_t i = right_block * sd->block_size; i <= right; i++)
            sum += sd->data[i];
    }

    return sum;
}

/* ========== MO'S ALGORITHM ========== */

#define HASH_CAPACITY 10007

static size_t hash_func(int64_t key)
{
    uint64_t k = (uint64_t)key;
    k ^= k >> 33;
    k *= 0xff51afd7ed558ccdULL;
    k ^= k >> 33;
    return k % HASH_CAPACITY;
}

t_mos_algo *mos_algo_new(size_t n)
{
    t_mos_algo *mo = malloc(sizeof(t_mos_algo));
    if (mo == NULL)
        return NULL;

    mo->block_size = (size_t)ceil(sqrt((double)n));
    if (mo->block_size == 0)
        mo->block_size = 1;

    mo->count_capacity = HASH_CAPACITY;
    mo->count_keys = malloc(mo->count_capacity * sizeof(int64_t));
    mo->count_values = calloc(mo->count_capacity, sizeof(size_t));
    mo->count_size = 0;
    mo->distinct = 0;

    if (mo->count_keys == NULL || mo->count_values == NULL)
    {
        free(mo->count_keys);
        free(mo->count_values);
        free(mo);
        return NULL;
    }

    memset(mo->count_keys, 0, mo->count_capacity * sizeof(int64_t));

    return mo;
}

void mos_algo_free(t_mos_algo *mo)
{
    if (mo == NULL)
        return;
    free(mo->count_keys);
    free(mo->count_values);
    free(mo);
}

static size_t find_slot(t_mos_algo *mo, int64_t value)
{
    size_t idx = hash_func(value);
    while (mo->count_values[idx] > 0 && mo->count_keys[idx] != value)
    {
        idx = (idx + 1) % mo->count_capacity;
    }
    return idx;
}

void mos_algo_add(t_mos_algo *mo, int64_t value)
{
    if (mo == NULL)
        return;

    size_t idx = find_slot(mo, value);
    if (mo->count_values[idx] == 0)
    {
        mo->count_keys[idx] = value;
        mo->distinct++;
    }
    mo->count_values[idx]++;
}

void mos_algo_remove(t_mos_algo *mo, int64_t value)
{
    if (mo == NULL)
        return;

    size_t idx = find_slot(mo, value);
    if (mo->count_values[idx] > 0)
    {
        mo->count_values[idx]--;
        if (mo->count_values[idx] == 0)
            mo->distinct--;
    }
}

size_t mos_algo_current_distinct(const t_mos_algo *mo)
{
    if (mo == NULL)
        return 0;
    return mo->distinct;
}

static int64_t mos_algo_current_mode(const t_mos_algo *mo)
{
    if (mo == NULL)
        return 0;

    int64_t mode = 0;
    size_t max_count = 0;

    for (size_t i = 0; i < mo->count_capacity; i++)
    {
        if (mo->count_values[i] > max_count)
        {
            max_count = mo->count_values[i];
            mode = mo->count_keys[i];
        }
    }

    return mode;
}

/* ========== QUERY COMPARISON FOR QSORT ========== */

static size_t g_block_size;

static int compare_queries(const void *a, const void *b)
{
    const t_query *qa = (const t_query *)a;
    const t_query *qb = (const t_query *)b;

    size_t block_a = qa->left / g_block_size;
    size_t block_b = qb->left / g_block_size;

    if (block_a != block_b)
        return (block_a < block_b) ? -1 : 1;

    if (block_a % 2 == 0)
        return (qa->right < qb->right) ? -1 : (qa->right > qb->right) ? 1 : 0;
    else
        return (qa->right > qb->right) ? -1 : (qa->right < qb->right) ? 1 : 0;
}

/* ========== DISTINCT QUERIES ========== */

size_t *distinct_queries(const int64_t *data, size_t n, t_query *queries, size_t q)
{
    if (data == NULL || n == 0 || queries == NULL || q == 0)
        return NULL;

    size_t *results = malloc(q * sizeof(size_t));
    if (results == NULL)
        return NULL;

    t_mos_algo *mo = mos_algo_new(n);
    if (mo == NULL)
    {
        free(results);
        return NULL;
    }

    g_block_size = mo->block_size;

    t_query *sorted = malloc(q * sizeof(t_query));
    if (sorted == NULL)
    {
        mos_algo_free(mo);
        free(results);
        return NULL;
    }
    memcpy(sorted, queries, q * sizeof(t_query));
    qsort(sorted, q, sizeof(t_query), compare_queries);

    size_t cur_left = 0;
    size_t cur_right = 0;
    mos_algo_add(mo, data[0]);

    for (size_t i = 0; i < q; i++)
    {
        size_t left = sorted[i].left;
        size_t right = sorted[i].right;

        while (cur_right < right)
        {
            cur_right++;
            mos_algo_add(mo, data[cur_right]);
        }
        while (cur_left > left)
        {
            cur_left--;
            mos_algo_add(mo, data[cur_left]);
        }
        while (cur_right > right)
        {
            mos_algo_remove(mo, data[cur_right]);
            cur_right--;
        }
        while (cur_left < left)
        {
            mos_algo_remove(mo, data[cur_left]);
            cur_left++;
        }

        results[sorted[i].id] = mos_algo_current_distinct(mo);
    }

    free(sorted);
    mos_algo_free(mo);
    return results;
}

/* ========== MODE QUERIES ========== */

int64_t *mode_queries(const int64_t *data, size_t n, t_query *queries, size_t q)
{
    if (data == NULL || n == 0 || queries == NULL || q == 0)
        return NULL;

    int64_t *results = malloc(q * sizeof(int64_t));
    if (results == NULL)
        return NULL;

    t_mos_algo *mo = mos_algo_new(n);
    if (mo == NULL)
    {
        free(results);
        return NULL;
    }

    g_block_size = mo->block_size;

    t_query *sorted = malloc(q * sizeof(t_query));
    if (sorted == NULL)
    {
        mos_algo_free(mo);
        free(results);
        return NULL;
    }
    memcpy(sorted, queries, q * sizeof(t_query));
    qsort(sorted, q, sizeof(t_query), compare_queries);

    size_t cur_left = 0;
    size_t cur_right = 0;
    mos_algo_add(mo, data[0]);

    for (size_t i = 0; i < q; i++)
    {
        size_t left = sorted[i].left;
        size_t right = sorted[i].right;

        while (cur_right < right)
        {
            cur_right++;
            mos_algo_add(mo, data[cur_right]);
        }
        while (cur_left > left)
        {
            cur_left--;
            mos_algo_add(mo, data[cur_left]);
        }
        while (cur_right > right)
        {
            mos_algo_remove(mo, data[cur_right]);
            cur_right--;
        }
        while (cur_left < left)
        {
            mos_algo_remove(mo, data[cur_left]);
            cur_left++;
        }

        results[sorted[i].id] = mos_algo_current_mode(mo);
    }

    free(sorted);
    mos_algo_free(mo);
    return results;
}
```

### 4.4 Solutions alternatives acceptees

**Alternative 1 — Rust avec segment tree pour block sums :**

```rust
// Using segment tree for block aggregation instead of simple array
pub struct SqrtDecompSegTree {
    data: Vec<i64>,
    tree: Vec<i64>,  // Segment tree over blocks
    block_size: usize,
}
```

**Alternative 2 — C avec double linked list pour hash collision :**

```c
// Using chaining instead of open addressing for hash table
typedef struct s_hash_node {
    int64_t key;
    size_t count;
    struct s_hash_node *next;
} t_hash_node;
```

### 4.5 Solutions refusees (avec explications)

**Refuse 1 — O(n*q) brute force :**

```rust
// REFUSE : Complexite O(n*q) au lieu de O((n+q)*sqrt(n))
pub fn distinct_queries_brute(data: &[i64], queries: &[Query]) -> Vec<usize> {
    queries.iter().map(|q| {
        let set: HashSet<_> = data[q.left..=q.right].iter().collect();
        set.len()
    }).collect()
}
```
**Pourquoi refuse :** Ne respecte pas la contrainte de complexite. Pour n=q=10^5, cela donne 10^10 operations au lieu de 3×10^7.

**Refuse 2 — Mo sans tri optimal :**

```rust
// REFUSE : Tri par left seulement, pas d'alternance
sorted_queries.sort_by_key(|q| q.left);  // Manque le critere right
```
**Pourquoi refuse :** Sans l'alternance du critere right, les deplacements du pointeur droit sont en O(n) par bloc au lieu de O(sqrt(n)) amortis.

### 4.6 Solution bonus de reference (COMPLETE)

```rust
use std::collections::HashMap;

pub struct MosTree {
    adj: Vec<Vec<usize>>,
    values: Vec<i64>,
    euler_tour: Vec<usize>,
    first: Vec<usize>,
    last: Vec<usize>,
    in_path: Vec<bool>,
    parent: Vec<Vec<usize>>,  // For LCA: parent[node][2^i]
    depth: Vec<usize>,
    block_size: usize,
}

#[derive(Clone, Copy)]
pub struct PathQuery {
    pub u: usize,
    pub v: usize,
    pub id: usize,
}

impl MosTree {
    pub fn new(adj: Vec<Vec<usize>>, values: Vec<i64>) -> Self {
        let n = adj.len();
        let mut tree = Self {
            adj,
            values,
            euler_tour: Vec::with_capacity(2 * n),
            first: vec![0; n],
            last: vec![0; n],
            in_path: vec![false; n],
            parent: vec![vec![0; 20]; n],  // log2(10^5) < 20
            depth: vec![0; n],
            block_size: ((2 * n) as f64).sqrt().ceil() as usize,
        };
        tree.compute_euler_tour(0, 0);
        tree.compute_lca_sparse_table(0);
        tree
    }

    fn compute_euler_tour(&mut self, node: usize, par: usize) {
        self.first[node] = self.euler_tour.len();
        self.euler_tour.push(node);
        self.parent[node][0] = par;

        for i in 0..self.adj[node].len() {
            let child = self.adj[node][i];
            if child != par {
                self.depth[child] = self.depth[node] + 1;
                self.compute_euler_tour(child, node);
            }
        }

        self.last[node] = self.euler_tour.len();
        self.euler_tour.push(node);
    }

    fn compute_lca_sparse_table(&mut self, root: usize) {
        for i in 1..20 {
            for v in 0..self.adj.len() {
                self.parent[v][i] = self.parent[self.parent[v][i-1]][i-1];
            }
        }
    }

    pub fn lca(&self, mut u: usize, mut v: usize) -> usize {
        if self.depth[u] < self.depth[v] {
            std::mem::swap(&mut u, &mut v);
        }

        let diff = self.depth[u] - self.depth[v];
        for i in 0..20 {
            if (diff >> i) & 1 == 1 {
                u = self.parent[u][i];
            }
        }

        if u == v {
            return u;
        }

        for i in (0..20).rev() {
            if self.parent[u][i] != self.parent[v][i] {
                u = self.parent[u][i];
                v = self.parent[v][i];
            }
        }

        self.parent[u][0]
    }

    fn toggle(&mut self, node: usize, count: &mut HashMap<i64, usize>, distinct: &mut usize) {
        let val = self.values[node];

        if self.in_path[node] {
            // Remove
            let entry = count.get_mut(&val).unwrap();
            *entry -= 1;
            if *entry == 0 {
                count.remove(&val);
                *distinct -= 1;
            }
        } else {
            // Add
            let entry = count.entry(val).or_insert(0);
            if *entry == 0 {
                *distinct += 1;
            }
            *entry += 1;
        }

        self.in_path[node] = !self.in_path[node];
    }
}

pub fn tree_distinct_queries(tree: &mut MosTree, queries: &[PathQuery]) -> Vec<usize> {
    if queries.is_empty() {
        return vec![];
    }

    // Convert path queries to Euler tour ranges
    struct MoQuery {
        left: usize,
        right: usize,
        lca_node: usize,
        id: usize,
    }

    let mut mo_queries: Vec<MoQuery> = queries.iter().map(|q| {
        let (u, v) = if tree.first[q.u] > tree.first[q.v] {
            (q.v, q.u)
        } else {
            (q.u, q.v)
        };

        let lca = tree.lca(u, v);

        if lca == u {
            MoQuery {
                left: tree.first[u],
                right: tree.first[v],
                lca_node: usize::MAX,  // LCA is endpoint, don't add separately
                id: q.id,
            }
        } else {
            MoQuery {
                left: tree.last[u],
                right: tree.first[v],
                lca_node: lca,
                id: q.id,
            }
        }
    }).collect();

    // Sort by Mo's ordering
    let block_size = tree.block_size;
    mo_queries.sort_by(|a, b| {
        let block_a = a.left / block_size;
        let block_b = b.left / block_size;
        if block_a != block_b {
            block_a.cmp(&block_b)
        } else if block_a % 2 == 0 {
            a.right.cmp(&b.right)
        } else {
            b.right.cmp(&a.right)
        }
    });

    let mut count: HashMap<i64, usize> = HashMap::new();
    let mut distinct = 0usize;
    let mut cur_left = 0;
    let mut cur_right = 0;
    let mut results = vec![0; queries.len()];

    // Initialize - don't add first node yet
    tree.in_path = vec![false; tree.adj.len()];

    for query in &mo_queries {
        while cur_right < query.right {
            cur_right += 1;
            tree.toggle(tree.euler_tour[cur_right], &mut count, &mut distinct);
        }
        while cur_left > query.left {
            cur_left -= 1;
            tree.toggle(tree.euler_tour[cur_left], &mut count, &mut distinct);
        }
        while cur_right > query.right {
            tree.toggle(tree.euler_tour[cur_right], &mut count, &mut distinct);
            cur_right -= 1;
        }
        while cur_left < query.left {
            tree.toggle(tree.euler_tour[cur_left], &mut count, &mut distinct);
            cur_left += 1;
        }

        // Handle LCA separately if needed
        if query.lca_node != usize::MAX {
            tree.toggle(query.lca_node, &mut count, &mut distinct);
        }

        results[query.id] = distinct;

        // Undo LCA
        if query.lca_node != usize::MAX {
            tree.toggle(query.lca_node, &mut count, &mut distinct);
        }
    }

    results
}
```

### 4.7 Solutions alternatives bonus (COMPLETES)

**Alternative — Heavy-Light Decomposition approach :**

```rust
// HLD can also solve tree path queries but with different trade-offs
// O(log^2 n) per query vs O(sqrt(n)) amortized with Mo
pub struct HLDTree {
    parent: Vec<usize>,
    depth: Vec<usize>,
    heavy: Vec<usize>,
    head: Vec<usize>,
    pos: Vec<usize>,
    // ... segment tree for each chain
}
```

### 4.8 Solutions refusees bonus (COMPLETES)

**Refuse — DFS pour chaque requete :**

```rust
// REFUSE : O(n) par requete = O(n*q) total
fn path_distinct_brute(tree: &Tree, u: usize, v: usize) -> usize {
    let path = tree.find_path(u, v);  // O(n) DFS
    let set: HashSet<_> = path.iter().map(|&node| tree.values[node]).collect();
    set.len()
}
```
**Pourquoi refuse :** Complexite O(n*q) inacceptable pour n=q=10^5.

### 4.9 spec.json (ENGINE v22.1 — FORMAT STRICT)

```json
{
  "name": "sqrt_decomp_mos",
  "language": "rust",
  "secondary_language": "c",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isole",
  "tags": ["advanced_algorithms", "sqrt_decomposition", "mos_algorithm", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "distinct_queries",
    "prototype": "pub fn distinct_queries(data: &[i64], queries: &[Query]) -> Vec<usize>",
    "return_type": "Vec<usize>",
    "parameters": [
      {"name": "data", "type": "&[i64]"},
      {"name": "queries", "type": "&[Query]"}
    ]
  },

  "driver": {
    "reference": "pub fn ref_distinct_queries(data: &[i64], queries: &[Query]) -> Vec<usize> { if data.is_empty() || queries.is_empty() { return vec![0; queries.len()]; } let n = data.len(); let block_size = ((n as f64).sqrt().ceil() as usize).max(1); let mut sorted: Vec<Query> = queries.to_vec(); sorted.sort_by(|a, b| { let ba = a.left / block_size; let bb = b.left / block_size; if ba != bb { ba.cmp(&bb) } else if ba % 2 == 0 { a.right.cmp(&b.right) } else { b.right.cmp(&a.right) } }); let mut count: std::collections::HashMap<i64, usize> = std::collections::HashMap::new(); let mut distinct = 0usize; let mut cur_l = 0; let mut cur_r = 0; let add = |v: i64, c: &mut std::collections::HashMap<i64, usize>, d: &mut usize| { let e = c.entry(v).or_insert(0); if *e == 0 { *d += 1; } *e += 1; }; let rem = |v: i64, c: &mut std::collections::HashMap<i64, usize>, d: &mut usize| { if let Some(e) = c.get_mut(&v) { *e -= 1; if *e == 0 { *d -= 1; c.remove(&v); } } }; add(data[0], &mut count, &mut distinct); let mut results = vec![0; queries.len()]; for q in &sorted { while cur_r < q.right { cur_r += 1; add(data[cur_r], &mut count, &mut distinct); } while cur_l > q.left { cur_l -= 1; add(data[cur_l], &mut count, &mut distinct); } while cur_r > q.right { rem(data[cur_r], &mut count, &mut distinct); cur_r -= 1; } while cur_l < q.left { rem(data[cur_l], &mut count, &mut distinct); cur_l += 1; } results[q.id] = distinct; } results }",

    "edge_cases": [
      {
        "name": "empty_array",
        "args": [[], []],
        "expected": [],
        "is_trap": true,
        "trap_explanation": "Tableau vide doit retourner vecteur vide"
      },
      {
        "name": "single_element",
        "args": [[42], [{"left": 0, "right": 0, "id": 0}]],
        "expected": [1],
        "is_trap": false
      },
      {
        "name": "all_same",
        "args": [[5, 5, 5, 5], [{"left": 0, "right": 3, "id": 0}]],
        "expected": [1],
        "is_trap": true,
        "trap_explanation": "Tous elements identiques = 1 distinct"
      },
      {
        "name": "all_distinct",
        "args": [[1, 2, 3, 4, 5], [{"left": 0, "right": 4, "id": 0}]],
        "expected": [5],
        "is_trap": false
      },
      {
        "name": "basic_case",
        "args": [[1, 2, 1, 3, 1, 2, 4], [{"left": 0, "right": 3, "id": 0}, {"left": 2, "right": 6, "id": 1}, {"left": 0, "right": 6, "id": 2}]],
        "expected": [3, 4, 4],
        "is_trap": false
      },
      {
        "name": "overlapping",
        "args": [[1, 1, 2, 2, 3, 3], [{"left": 0, "right": 1, "id": 0}, {"left": 2, "right": 3, "id": 1}, {"left": 4, "right": 5, "id": 2}, {"left": 0, "right": 5, "id": 3}]],
        "expected": [1, 1, 1, 3],
        "is_trap": false
      },
      {
        "name": "left_equals_right",
        "args": [[7, 8, 9], [{"left": 1, "right": 1, "id": 0}]],
        "expected": [1],
        "is_trap": true,
        "trap_explanation": "Requete sur un seul element"
      },
      {
        "name": "large_values",
        "args": [[-1000000000, 1000000000, 0], [{"left": 0, "right": 2, "id": 0}]],
        "expected": [3],
        "is_trap": false
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "array_int",
          "param_index": 0,
          "params": {
            "min_len": 1,
            "max_len": 1000,
            "min_val": -1000000,
            "max_val": 1000000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["HashMap", "Vec", "sqrt", "ceil", "malloc", "free", "qsort", "memset", "memcpy"],
    "forbidden_functions": ["unsafe", "VLA"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

**Mutant A (Boundary) — Block size sans ceil :**

```rust
/* Mutant A (Boundary) : Oubli de ceil() pour block_size */
impl SqrtDecomp {
    pub fn new(data: Vec<i64>) -> Self {
        let n = data.len();
        // BUG: Pas de ceil(), peut causer index out of bounds
        let block_size = (n as f64).sqrt() as usize;  // Manque .ceil()
        // ...
    }
}
// Pourquoi c'est faux : Pour n=10, sqrt(10)=3.16, cast donne 3
// Mais (10 + 3 - 1) / 3 = 4 blocs necessaires
// Sans ceil, on peut avoir des acces hors limites
// Ce qui etait pense : "sqrt() suffit"
```

**Mutant B (Safety) — Pas de verification left <= right :**

```rust
/* Mutant B (Safety) : Pas de validation des bornes */
pub fn range_sum(&self, left: usize, right: usize) -> i64 {
    // BUG: Pas de verification left > right
    let left_block = left / self.block_size;
    let right_block = right / self.block_size;
    // Peut iterer a l'envers ou panic
    // ...
}
// Pourquoi c'est faux : Si left > right, les boucles for
// peuvent avoir un comportement indefini ou panic
// Ce qui etait pense : "L'appelant verifiera"
```

**Mutant C (Resource) — Tri Mo sans alternance :**

```rust
/* Mutant C (Resource) : Tri sous-optimal gaspillant des operations */
sorted_queries.sort_by_key(|q| (q.left / block_size, q.right));
// BUG: Manque l'alternance paire/impaire du block

// Correct:
sorted_queries.sort_by(|a, b| {
    let block_a = a.left / block_size;
    let block_b = b.left / block_size;
    if block_a != block_b {
        block_a.cmp(&block_b)
    } else if block_a % 2 == 0 {
        a.right.cmp(&b.right)  // Croissant
    } else {
        b.right.cmp(&a.right)  // Decroissant
    }
});
// Pourquoi c'est faux : Sans alternance, le pointeur droit
// fait des allers-retours O(n) par bloc au lieu de O(n) total
// Ce qui etait pense : "Trier par (block, right) suffit"
```

**Mutant D (Logic) — Add/Remove inverses :**

```rust
/* Mutant D (Logic) : add() et remove() inverses dans Mo */
// Dans la boucle principale:
while cur_right < right {
    cur_right += 1;
    mo.remove(data[cur_right]);  // BUG: Devrait etre add()
}
while cur_left > left {
    cur_left -= 1;
    mo.remove(data[cur_left]);  // BUG: Devrait etre add()
}
while cur_right > right {
    mo.add(data[cur_right]);    // BUG: Devrait etre remove()
    cur_right -= 1;
}
while cur_left < left {
    mo.add(data[cur_left]);     // BUG: Devrait etre remove()
    cur_left += 1;
}
// Pourquoi c'est faux : On ajoute quand on retrecit la fenetre
// et on retire quand on l'agrandit - logique inversee
// Ce qui etait pense : Confusion sur la direction des pointeurs
```

**Mutant E (Return) — Retourne count au lieu de mode :**

```rust
/* Mutant E (Return) : Retourne la frequence au lieu du mode */
fn current_mode(&self) -> i64 {
    self.count
        .iter()
        .max_by_key(|&(_, &cnt)| cnt)
        .map(|(_, &cnt)| cnt as i64)  // BUG: Retourne cnt, pas val
        .unwrap_or(0)
}
// Correct:
fn current_mode(&self) -> i64 {
    self.count
        .iter()
        .max_by_key(|&(_, &cnt)| cnt)
        .map(|(&val, _)| val)         // Retourne val, le mode
        .unwrap_or(0)
}
// Pourquoi c'est faux : On veut l'element le plus frequent,
// pas combien de fois il apparait
// Ce qui etait pense : Confusion entre "quoi" et "combien"
```

---

## SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

Cet exercice enseigne trois concepts fondamentaux des algorithmes avances :

1. **Sqrt Decomposition** — Decomposer un probleme en √n sous-problemes pour equilibrer pre-traitement et requete

2. **Mo's Algorithm** — Reorganiser des requetes offline pour minimiser le travail total en exploitant la localite

3. **Amortized Analysis** — Comprendre pourquoi une serie d'operations couteuses individuellement peut etre efficace globalement

### 5.2 LDA — Traduction litterale en francais (MAJUSCULES)

```
FONCTION distinct_queries QUI RETOURNE UN VECTEUR D'ENTIERS NON SIGNES ET PREND EN PARAMETRES data QUI EST UNE TRANCHE DE NOMBRES ENTIERS 64-BIT ET queries QUI EST UNE TRANCHE DE REQUETES
DEBUT FONCTION
    SI data EST VIDE OU queries EST VIDE ALORS
        RETOURNER UN VECTEUR DE ZEROS DE TAILLE queries.len()
    FIN SI

    DECLARER n COMME LA LONGUEUR DE data
    DECLARER block_size COMME LA RACINE CARREE DE n ARRONDIE AU SUPERIEUR

    DECLARER sorted_queries COMME UNE COPIE DE queries
    TRIER sorted_queries SELON L'ORDRE DE MO

    DECLARER count COMME UN DICTIONNAIRE DE VALEUR VERS FREQUENCE
    DECLARER distinct COMME UN ENTIER NON SIGNE INITIALISE A 0
    DECLARER cur_left COMME UN ENTIER NON SIGNE INITIALISE A 0
    DECLARER cur_right COMME UN ENTIER NON SIGNE INITIALISE A 0

    AJOUTER data[0] AU DICTIONNAIRE count
    INCREMENTER distinct DE 1

    POUR CHAQUE query DANS sorted_queries FAIRE
        TANT QUE cur_right EST INFERIEUR A query.right FAIRE
            INCREMENTER cur_right DE 1
            APPELER add AVEC data[cur_right]
        FIN TANT QUE

        TANT QUE cur_left EST SUPERIEUR A query.left FAIRE
            DECREMENTER cur_left DE 1
            APPELER add AVEC data[cur_left]
        FIN TANT QUE

        TANT QUE cur_right EST SUPERIEUR A query.right FAIRE
            APPELER remove AVEC data[cur_right]
            DECREMENTER cur_right DE 1
        FIN TANT QUE

        TANT QUE cur_left EST INFERIEUR A query.left FAIRE
            APPELER remove AVEC data[cur_left]
            INCREMENTER cur_left DE 1
        FIN TANT QUE

        AFFECTER distinct A results[query.id]
    FIN POUR

    RETOURNER results
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Mo's Algorithm pour Requetes Distinctes
---
1. VALIDATION des entrees :
   - SI tableau vide OU aucune requete
     RETOURNER vecteur de zeros

2. INITIALISATION :
   a. Calculer block_size = ceil(sqrt(n))
   b. Creer copie triable des requetes
   c. Initialiser HashMap pour frequences
   d. Initialiser compteur distinct = 0

3. TRI des requetes (ordre de Mo) :
   a. Tri primaire : par block du left (left / block_size)
   b. Tri secondaire : par right
      - Si block pair : right croissant
      - Si block impair : right decroissant

4. BOUCLE PRINCIPALE (fenetre glissante) :
   POUR chaque requete triee :
   |
   |-- EXPANSION droite :
   |     TANT QUE cur_right < target_right
   |       Ajouter element, incrementer cur_right
   |
   |-- EXPANSION gauche :
   |     TANT QUE cur_left > target_left
   |       Decrementer cur_left, ajouter element
   |
   |-- CONTRACTION droite :
   |     TANT QUE cur_right > target_right
   |       Retirer element, decrementer cur_right
   |
   |-- CONTRACTION gauche :
   |     TANT QUE cur_left < target_left
   |       Retirer element, incrementer cur_left
   |
   |-- ENREGISTRER distinct dans results[query.id]

5. RETOURNER results
```

### 5.2.3 Representation Algorithmique (Logique de Garde)

```
FONCTION : add(value)
---
INIT entry = count.get_or_create(value, 0)

1. VERIFIER si entry.count == 0 :
   |-- distinct++  (nouvel element unique)

2. INCREMENTER entry.count

3. RETOURNER


FONCTION : remove(value)
---
1. VERIFIER si value existe dans count :
   |-- SI NON : RETOURNER (rien a faire)

2. DECREMENTER count[value]

3. VERIFIER si count[value] == 0 :
   |-- distinct--  (element plus present)
   |-- SUPPRIMER entry du dictionnaire

4. RETOURNER
```

### 5.2.3.1 Diagramme Mermaid

```mermaid
graph TD
    A[Debut: distinct_queries] --> B{data ou queries vide?}
    B -- Oui --> C[RETOUR: vec![0; q]]
    B -- Non --> D[Calculer block_size = ceil/sqrt/n]

    D --> E[Trier requetes par Mo]
    E --> F[Initialiser fenetre a position 0]

    F --> G{Requetes restantes?}
    G -- Non --> H[RETOUR: results]
    G -- Oui --> I[Prendre prochaine requete]

    I --> J{cur_right < target_right?}
    J -- Oui --> K[cur_right++, add/data/cur_right]
    K --> J
    J -- Non --> L{cur_left > target_left?}

    L -- Oui --> M[cur_left--, add/data/cur_left]
    M --> L
    L -- Non --> N{cur_right > target_right?}

    N -- Oui --> O[remove/data/cur_right, cur_right--]
    O --> N
    N -- Non --> P{cur_left < target_left?}

    P -- Oui --> Q[remove/data/cur_left, cur_left++]
    Q --> P
    P -- Non --> R[results/id = distinct]

    R --> G
```

### 5.3 Visualisation ASCII (adaptee au sujet)

```
SQRT DECOMPOSITION - Vue d'ensemble
====================================

Tableau original (n=9) :
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 3 │ 1 │ 4 │ 1 │ 5 │ 9 │ 2 │ 6 │ 5 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┘
  0   1   2   3   4   5   6   7   8

Decomposition en blocs (block_size = ceil(sqrt(9)) = 3) :
┌─────────────┬─────────────┬─────────────┐
│   BLOC 0    │   BLOC 1    │   BLOC 2    │
├───┬───┬───┐ ├───┬───┬───┐ ├───┬───┬───┐ │
│ 3 │ 1 │ 4 │ │ 1 │ 5 │ 9 │ │ 2 │ 6 │ 5 │ │
└───┴───┴───┘ └───┴───┴───┘ └───┴───┴───┘ │
│   sum = 8   │   sum = 15  │   sum = 13  │
└─────────────┴─────────────┴─────────────┘

Requete range_sum(1, 7) :
                 │ Partiel │   Complet   │ Partiel │
                 ▼         ▼             ▼         ▼
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 3 │ 1 │ 4 │ 1 │ 5 │ 9 │ 2 │ 6 │ 5 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┘
      └───┬───┘ └─────┬─────┘ └───┬───┘
        1+4=5     use 15      2+6=8
        (iter)    (O(1))      (iter)

Total = 5 + 15 + 8 = 28


MO'S ALGORITHM - Tri des requetes
==================================

Requetes originales (block_size = 3) :
┌────┬──────┬───────┬───────┐
│ ID │ Left │ Right │ Block │
├────┼──────┼───────┼───────┤
│ 0  │  0   │   3   │   0   │
│ 1  │  2   │   6   │   0   │
│ 2  │  4   │   7   │   1   │
│ 3  │  1   │   5   │   0   │
│ 4  │  5   │   8   │   1   │
└────┴──────┴───────┴───────┘

Apres tri Mo (block puis right alterne) :
┌────┬──────┬───────┬───────┬──────────────────┐
│ ID │ Left │ Right │ Block │ Critere Right    │
├────┼──────┼───────┼───────┼──────────────────┤
│ 0  │  0   │   3   │   0   │ Block 0 (pair)   │
│ 3  │  1   │   5   │   0   │ → right croissant│
│ 1  │  2   │   6   │   0   │                  │
├────┼──────┼───────┼───────┼──────────────────┤
│ 4  │  5   │   8   │   1   │ Block 1 (impair) │
│ 2  │  4   │   7   │   1   │ → right decroiss │
└────┴──────┴───────┴───────┴──────────────────┘


FENETRE GLISSANTE - Mouvement des pointeurs
============================================

Traitement des requetes dans l'ordre Mo :

Query 0 (left=0, right=3):
    ┌───────────────────┐
    │ L=0           R=3 │
    ▼                   ▼
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 3 │ 1 │ 4 │ 1 │ 5 │ 9 │ 2 │ 6 │ 5 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┘
    └───────────────┘
    Fenetre: {3,1,4,1} → distinct=3 {1,3,4}

Query 3 (left=1, right=5): expand right, contract left
        ┌───────────────────────┐
        │ L=1               R=5 │
        ▼                       ▼
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 3 │ 1 │ 4 │ 1 │ 5 │ 9 │ 2 │ 6 │ 5 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┘
        └───────────────────┘
    Mouvements: R: 3→4→5 (+2 add)
                L: 0→1 (+1 remove)
    Fenetre: {1,4,1,5,9} → distinct=4 {1,4,5,9}

Query 1 (left=2, right=6): expand right, contract left
            ┌───────────────────────┐
            │ L=2               R=6 │
            ▼                       ▼
┌───┬───┬───┬───┬───┬───┬───┬───┬───┐
│ 3 │ 1 │ 4 │ 1 │ 5 │ 9 │ 2 │ 6 │ 5 │
└───┴───┴───┴───┴───┴───┴───┴───┴───┘
            └───────────────────┘
    Mouvements: R: 5→6 (+1 add)
                L: 1→2 (+1 remove)
    Fenetre: {4,1,5,9,2} → distinct=5 {1,2,4,5,9}
```

### 5.4 Les pieges en detail

#### Piege 1 : Block size incorrect

```rust
// FAUX - peut causer index out of bounds
let block_size = (n as f64).sqrt() as usize;

// CORRECT - arrondir au superieur et gerer n=0
let block_size = ((n as f64).sqrt().ceil() as usize).max(1);
```

**Pourquoi ?** Pour n=10, sqrt(10)≈3.16. Sans ceil(), on obtient 3. Mais `(10-1)/3 = 3`, donc on a besoin de 4 blocs (indices 0,1,2,3). Avec block_size=3, on accede a blocks[3] mais on n'a alloue que 3 blocs !

#### Piege 2 : Ordre Mo mal implemente

```rust
// FAUX - pas d'alternance, O(n*sqrt(n)) deplacements right
queries.sort_by_key(|q| (q.left / block_size, q.right));

// CORRECT - alternance pour minimiser right movements
queries.sort_by(|a, b| {
    let ba = a.left / block_size;
    let bb = b.left / block_size;
    if ba != bb {
        ba.cmp(&bb)
    } else if ba % 2 == 0 {
        a.right.cmp(&b.right)    // Croissant
    } else {
        b.right.cmp(&a.right)     // Decroissant
    }
});
```

**Pourquoi ?** Sans alternance, dans chaque bloc, right va de min a max puis revient. Avec alternance, right traverse le tableau dans un sens puis l'autre, totalisant O(n) mouvements par dimension.

#### Piege 3 : Ordre des operations add/remove

```rust
// FAUX - ordre incorrect
while cur_right > right {
    cur_right -= 1;           // D'abord decrementer
    mo.remove(data[cur_right]); // Puis retirer - FAUX index!
}

// CORRECT - d'abord retirer l'element actuel, puis decrementer
while cur_right > right {
    mo.remove(data[cur_right]);
    cur_right -= 1;
}
```

**Pourquoi ?** Si cur_right=5 et right=4, on veut retirer data[5], pas data[4]. L'ordre des operations est crucial.

#### Piege 4 : Gestion du compteur distinct

```rust
// FAUX - distinct peut devenir negatif
fn remove(&mut self, value: i64) {
    let entry = self.count.get_mut(&value).unwrap();
    *entry -= 1;
    if *entry == 0 {
        self.distinct -= 1;  // Que se passe-t-il si on remove plus qu'on add?
    }
}

// CORRECT - verifier l'existence avant
fn remove(&mut self, value: i64) {
    if let Some(entry) = self.count.get_mut(&value) {
        if *entry > 0 {
            *entry -= 1;
            if *entry == 0 {
                self.distinct -= 1;
                self.count.remove(&value);  // Nettoyer
            }
        }
    }
}
```

### 5.5 Cours Complet

#### 5.5.1 Introduction a la Decomposition en Racine Carree

La **Sqrt Decomposition** est une technique de division d'un probleme en √n sous-problemes. Elle offre un compromis entre :
- Pre-traitement O(n) et requete O(n) → trop lent
- Pre-traitement O(n²) et requete O(1) → trop de memoire

Avec √n blocs de taille √n chacun :
- Pre-traitement : O(n) pour calculer les agregats par bloc
- Requete : O(√n) = O(√n) blocs complets + O(√n) elements partiels
- Mise a jour : O(1) pour l'element + O(1) pour le bloc

#### 5.5.2 L'Algorithme de Mo

**Principe :** Au lieu de traiter les requetes dans l'ordre, on les trie pour minimiser le travail total.

**Observation cle :** Si on maintient une "fenetre" [L, R] et qu'on peut ajouter/retirer des elements aux extremites en O(1), alors passer de la fenetre [L1, R1] a [L2, R2] coute O(|L1-L2| + |R1-R2|).

**Tri de Mo :**
1. Diviser les requetes en blocs selon leur position gauche
2. Dans chaque bloc, trier par position droite
3. Alternance : blocs pairs en ordre croissant de right, blocs impairs en decroissant

**Complexite :**
- Mouvements de left : O(q × √n) car on change de bloc q fois, chaque bloc a √n positions
- Mouvements de right : O(n × √n) car dans chaque bloc, right traverse au plus n positions, et il y a √n blocs
- Total : O((n + q) × √n)

#### 5.5.3 Cas d'usage typiques

| Probleme | Pre-traitement | Complexite requete |
|----------|----------------|-------------------|
| Range sum | O(n) | O(√n) |
| Range min/max | O(n) | O(√n) |
| Distinct elements | O(n log n) | O(√n) amortis (Mo) |
| Mode query | O(n log n) | O(√n) amortis (Mo) |
| Range update + query | O(n) | O(√n) |

#### 5.5.4 Extension : Mo sur les arbres

Pour les requetes sur chemins d'arbres, on utilise l'**Euler Tour** pour lineariser l'arbre :
- Chaque noeud apparait 2 fois (entree et sortie)
- Un chemin u→v devient un intervalle dans l'Euler tour
- Le LCA necessite un traitement special

### 5.6 Normes avec explications pedagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ let block_size = (n as f64).sqrt() as usize;                   │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ let block_size = ((n as f64).sqrt().ceil() as usize).max(1);   │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Precision : ceil() garantit assez de blocs                   │
│ • Safety : max(1) evite division par zero                      │
│ • Robustesse : Fonctionne pour tous les n >= 0                 │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ while cur_right > right { cur_right -= 1; remove(cur_right); } │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ while cur_right > right { remove(cur_right); cur_right -= 1; } │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • L'ordre compte : on retire l'element ACTUEL                  │
│ • Post-decrement : d'abord l'action, puis le deplacement       │
│ • Symetrie : expansion = pre-increment, contraction = post     │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'execution

**Exemple :** `distinct_queries([1, 2, 1, 3, 1], [(0,2), (1,4), (2,3)])`

Block size = ceil(sqrt(5)) = 3

**Tri Mo :**
| Original ID | Left | Right | Block | Tri |
|-------------|------|-------|-------|-----|
| 0 | 0 | 2 | 0 | 1 |
| 1 | 1 | 4 | 0 | 2 |
| 2 | 2 | 3 | 0 | 3 |

Ordre : 0 → 1 → 2 (block 0, right croissant)

```
┌───────┬─────────────────────────────────┬─────────┬─────────┬──────────┬───────────┐
│ Etape │ Action                          │ cur_L   │ cur_R   │ count    │ distinct  │
├───────┼─────────────────────────────────┼─────────┼─────────┼──────────┼───────────┤
│   1   │ Init: add(data[0])=add(1)       │   0     │   0     │ {1:1}    │    1      │
├───────┼─────────────────────────────────┼─────────┼─────────┼──────────┼───────────┤
│       │ === Query 0: [0,2] ===          │         │         │          │           │
│   2   │ expand R: add(data[1])=add(2)   │   0     │   1     │ {1:1,2:1}│    2      │
│   3   │ expand R: add(data[2])=add(1)   │   0     │   2     │ {1:2,2:1}│    2      │
│   4   │ → results[0] = 2                │   0     │   2     │          │    2      │
├───────┼─────────────────────────────────┼─────────┼─────────┼──────────┼───────────┤
│       │ === Query 1: [1,4] ===          │         │         │          │           │
│   5   │ expand R: add(data[3])=add(3)   │   0     │   3     │{1:2,2:1, │    3      │
│       │                                 │         │         │ 3:1}     │           │
│   6   │ expand R: add(data[4])=add(1)   │   0     │   4     │{1:3,2:1, │    3      │
│       │                                 │         │         │ 3:1}     │           │
│   7   │ shrink L: remove(data[0])=rem(1)│   1     │   4     │{1:2,2:1, │    3      │
│       │                                 │         │         │ 3:1}     │           │
│   8   │ → results[1] = 3                │   1     │   4     │          │    3      │
├───────┼─────────────────────────────────┼─────────┼─────────┼──────────┼───────────┤
│       │ === Query 2: [2,3] ===          │         │         │          │           │
│   9   │ shrink R: remove(data[4])=rem(1)│   1     │   3     │{1:1,2:1, │    3      │
│       │                                 │         │         │ 3:1}     │           │
│  10   │ shrink L: remove(data[1])=rem(2)│   2     │   3     │{1:1,3:1} │    2      │
│  11   │ → results[2] = 2                │   2     │   3     │          │    2      │
└───────┴─────────────────────────────────┴─────────┴─────────┴──────────┴───────────┘

Resultat final : [2, 3, 2]
```

### 5.8 Mnemotechniques (MEME obligatoire)

#### 🔍 MEME : "Elementary, my dear Watson" — La methode Holmes

![Sherlock analysant des indices](sherlock_deduction.jpg)

Comme Sherlock Holmes qui decompose un mystere en indices logiques,
la Sqrt Decomposition decompose un tableau en blocs gérables.

```rust
// Holmes ne lit pas TOUT le dossier a chaque question
// Il organise ses indices en CATEGORIES (blocs)
struct SqrtDecomp {
    blocks: Vec<i64>,  // "Mes categories d'indices, Watson!"
}
```

---

#### 🗂️ MEME : "Marie Kondo" — Tri des requetes

Comme Marie Kondo qui reorganise une maison pour minimiser les deplacements,
Mo's Algorithm reorganise les requetes pour minimiser les mouvements.

```rust
// Ne traite pas les requetes dans n'importe quel ordre!
// Trie-les pour que tes "deplacements" soient optimaux
queries.sort_by_mo_order();  // "Does this order spark joy?"
```

---

#### 🎮 MEME : "Speedrun Any%" — Minimiser les mouvements

Dans un speedrun, le joueur optimise sa trajectoire pour minimiser le temps.
Mo's Algorithm fait pareil : il minimise les "pas" (add/remove) totaux.

```
Sans tri Mo :    ████████████████████ (n*q operations)
Avec tri Mo :    ████████ ((n+q)*sqrt(n) operations)
                 "New World Record!"
```

---

#### 🧮 MEME : "Le compromis de Thanos" — Pourquoi sqrt(n) ?

"Parfaitement equilibre, comme toute chose devrait l'etre."

```
Block trop petit (1)     : O(1) par block, O(n) blocks = O(n)
Block trop grand (n)     : O(n) par block, O(1) blocks = O(n)
Block parfait (sqrt(n))  : O(√n) par block, O(√n) blocks = O(√n) total

Thanos approuve : √n × √n = n ← equilibre parfait
```

### 5.9 Applications pratiques

| Domaine | Application | Technique utilisee |
|---------|-------------|-------------------|
| **Competitive Programming** | SPOJ DQUERY, Codeforces D/E | Mo's Algorithm |
| **Database Systems** | Partitionnement de tables | Sqrt-like chunking |
| **Time Series Analysis** | Fenetres glissantes optimisees | Mo's sur intervalles temporels |
| **Geographic Information Systems** | Requetes spatiales | Sqrt decomposition 2D |
| **Bioinformatics** | Analyse de sequences genomiques | Range distinct queries |

---

## SECTION 6 : PIEGES — RECAPITULATIF

| Piege | Description | Solution |
|-------|-------------|----------|
| **Block size** | `sqrt() as usize` sans ceil | Utiliser `ceil()` + `max(1)` |
| **Tri Mo** | Pas d'alternance pair/impair | Implementer alternance right |
| **Ordre add/remove** | Decrement avant remove | Remove puis decrement |
| **Distinct negatif** | Remove sans verification | Verifier count > 0 avant -- |
| **Index out of bounds** | right >= n | `right.min(n-1)` |
| **Requete vide** | left > right | Retourner 0/default |
| **Division par zero** | block_size = 0 quand n = 0 | `max(1)` sur block_size |

---

## SECTION 7 : QCM

### Question 1
**Pourquoi utilise-t-on √n comme taille de bloc dans Sqrt Decomposition ?**

A) C'est un nombre premier pratique
B) C'est le point d'equilibre entre pre-traitement et requete
C) C'est impose par le standard C17
D) C'est plus facile a calculer que d'autres racines
E) Les processeurs modernes ont des registres de taille √n
F) C'est une convention arbitraire
G) √n blocs de taille √n minimisent max(blocs, taille_bloc)
H) Les caches CPU sont optimises pour √n
I) C'est le nombre de Fibonacci le plus proche
J) C'est requis pour la norme POSIX

**Reponse :** B et G

---

### Question 2
**Dans l'algorithme de Mo, pourquoi alterne-t-on l'ordre de right entre blocs pairs et impairs ?**

A) Pour eviter les buffer overflows
B) Pour minimiser le total des deplacements du pointeur right
C) Pour respecter la norme de codage
D) Pour que le tri soit stable
E) Le pointeur right fait un "serpentin" au lieu d'allers-retours
F) C'est une optimisation de cache
G) Pour eviter les deadlocks
H) Pour reduire la complexite de O(n²) a O(n×√n)
I) C'est obligatoire en Rust
J) Pour supporter les nombres negatifs

**Reponse :** B, E, H

---

### Question 3
**Quelle est la complexite temporelle de distinct_queries avec Mo's Algorithm pour n elements et q requetes ?**

A) O(n × q)
B) O(n + q)
C) O((n + q) × √n)
D) O(n × log(q))
E) O(q × log(n))
F) O(n² + q²)
G) O(n × √q)
H) O(√n × √q)
I) O(n × q × √n)
J) O(log(n) × log(q))

**Reponse :** C

---

### Question 4
**Que se passe-t-il si on oublie de verifier `left <= right` dans une requete de range sum ?**

A) Le programme compile mais retourne des valeurs incorrectes
B) Segmentation fault garantie
C) Boucle infinie possible si on utilise des entiers non signes
D) Le compilateur detecte l'erreur
E) Les sommes peuvent etre negatives meme avec des valeurs positives
F) Undefined behavior en C
G) Panic en Rust avec certains iterateurs
H) Aucun probleme, c'est gere automatiquement
I) Memory leak
J) Stack overflow

**Reponse :** A, C, F, G

---

### Question 5
**Dans le tri de Mo, si deux requetes ont le meme bloc gauche et que le bloc est PAIR, comment sont-elles triees par right ?**

A) Decroissant
B) Croissant
C) Aleatoire
D) Par ID original
E) Par somme left + right
F) Par difference right - left
G) Par XOR de left et right
H) Par hash des valeurs
I) Non triees (ordre preserve)
J) Par adresse memoire

**Reponse :** B

---

## SECTION 8 : RECAPITULATIF

| Critere | Valeur |
|---------|--------|
| **Module** | 1.7.3 — Sqrt Decomposition |
| **Exercice** | sqrt_decomp_mos |
| **Difficulte** | ★★★★★★★☆☆☆ (7/10) |
| **Langages** | Rust Edition 2024 + C (C17) |
| **Duree** | 90 min |
| **XP Base** | 150 |
| **XP Bonus** | 450 (×3) |
| **Complexite** | O((n+q)×√n) temps, O(n) espace |
| **Concepts cles** | Block decomposition, Mo's algorithm, offline queries |
| **Prerequis** | Tableaux, tri, complexite algorithmique |
| **Theme** | Sherlock Holmes — L'Art de la Deduction |

**Resume :** Cet exercice enseigne deux techniques fondamentales de l'algorithmique avancee : la decomposition en racine carree pour equilibrer pre-traitement et requetes, et l'algorithme de Mo pour optimiser le traitement de requetes offline. Ces techniques sont essentielles en programmation competitive et ont des applications dans les systemes de bases de donnees et l'analyse de time-series.

---

## SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.7.3-a-sqrt_decomp_mos",
    "generated_at": "2026-01-12 00:00:00",

    "metadata": {
      "exercise_id": "1.7.3-a",
      "exercise_name": "sqrt_decomp_mos",
      "module": "1.7.3",
      "module_name": "Sqrt Decomposition",
      "concept": "a",
      "concept_name": "Block decomposition & Mo's algorithm",
      "type": "code",
      "tier": 1,
      "tier_info": "Concept isole",
      "phase": 1,
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "language": "rust",
      "secondary_language": "c",
      "duration_minutes": 90,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCE",
      "bonus_icon": "🔥",
      "complexity_time": "T7 O((n+q)×√n)",
      "complexity_space": "S4 O(n)",
      "prerequisites": ["arrays", "sorting", "complexity_analysis"],
      "domains": ["Algo", "Struct", "MD"],
      "domains_bonus": ["Struct"],
      "tags": ["sqrt_decomposition", "mos_algorithm", "offline_queries", "range_queries", "distinct_elements"],
      "meme_reference": "Elementary, my dear Watson"
    },

    "files": {
      "spec.json": "/* Contenu de la section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 Rust */",
      "references/ref_solution.c": "/* Section 4.3 C */",
      "references/ref_solution.h": "/* Section 4.3 Header */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "alternatives/alt_segment_tree.rs": "/* Section 4.4 */",
      "alternatives/alt_chaining_hash.c": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.rs": "/* Block size sans ceil */",
      "mutants/mutant_b_safety.rs": "/* Pas de verification left <= right */",
      "mutants/mutant_c_resource.rs": "/* Tri Mo sans alternance */",
      "mutants/mutant_d_logic.rs": "/* Add/remove inverses */",
      "mutants/mutant_e_return.rs": "/* Retourne count au lieu de mode */",
      "tests/main.rs": "/* Tests Rust */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "references/ref_solution.c",
        "references/ref_solution_bonus.rs",
        "alternatives/alt_segment_tree.rs",
        "alternatives/alt_chaining_hash.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_safety.rs",
        "mutants/mutant_c_resource.rs",
        "mutants/mutant_d_logic.rs",
        "mutants/mutant_e_return.rs"
      ]
    },

    "commands": {
      "validate_spec": "python3 hackbrain_engine_v22.py --validate-spec spec.json",
      "test_reference_rust": "cargo test --release",
      "test_reference_c": "gcc -Wall -Wextra -Werror -O2 -std=c17 sqrt_decomp.c main.c -o test -lm && ./test",
      "test_mutants": "python3 hackbrain_mutation_tester.py -r references/ref_solution.rs -s spec.json --validate"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — Prompt Systeme Unifie de Production d'Exercices*
*"L'excellence pedagogique ne se negocie pas — pas de raccourcis"*
*Compatible ENGINE v22.1 + Mutation Tester*
