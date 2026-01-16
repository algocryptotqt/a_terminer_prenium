<thinking>
## Analyse du Concept
- Concept : Randomized Algorithms (Monte Carlo, Las Vegas, Quickselect, Reservoir sampling, Karger min-cut, Bloom filter, Skip list, etc.)
- Phase demandee : 1 (Intermediate)
- Adapte ? OUI - Les algorithmes randomises sont parfaits pour Phase 1 car ils introduisent des concepts probabilistes fondamentaux tout en restant accessibles avec une bonne base algorithmique.

## Combo Base + Bonus
- Exercice de base : Implementation de structures probabilistes fondamentales (BloomFilter, CountMinSketch, HyperLogLog, MinHash, SkipList) + algorithmes randomises (quickselect, reservoir_sample, miller_rabin, karger_min_cut)
- Bonus : Extensions avancees - Cuckoo Filter, Quotient Filter, Monte Carlo simulation avancee, Las Vegas algorithms with early termination
- Palier bonus : 🔥 Avance (difficulte 8/10 pour le bonus)
- Progression logique ? OUI - Base couvre les structures fondamentales, bonus ajoute des variantes plus sophistiquees

## Prerequis & Difficulte
- Prerequis reels : Probabilites de base, manipulation de bits, hash functions, tableaux, structures de donnees
- Difficulte estimee : 7/10
- Coherent avec phase ? OUI - Phase 1 autorise 3-5/10 de base mais avec exercice de synthese on peut aller jusqu'a 7/10

## Aspect Fun/Culture
- Contexte choisi : Casino Royale (James Bond) - Quand le hasard devient une arme strategique
- MEME mnémotechnique : "The name is Bond... James Bond" pour les operations probabilistes, "Le Chiffre" pour les hash functions
- Pourquoi c'est fun : L'univers de l'espionnage et des casinos est parfait pour illustrer les algorithmes probabilistes - le joueur qui calcule ses probabilites, l'espion qui echantillonne des donnees, le cryptographe qui teste la primalite

## Scenarios d'Echec (5 mutants concrets)
1. Mutant A (Boundary) : Bloom filter avec mauvais calcul de faux positifs - utilise % size au lieu de & (size - 1) pour le hash
2. Mutant B (Safety) : QuickSelect sans verification de k dans les bornes - retourne garbage si k > len
3. Mutant C (Resource) : Skip list sans liberation des niveaux - fuite memoire sur les noeuds multi-niveaux
4. Mutant D (Logic) : Miller-Rabin avec exposant mal calcule - n'extrait pas correctement d de n-1 = 2^r * d
5. Mutant E (Return) : Reservoir sampling qui retourne toujours les k premiers elements - oublie la condition de remplacement random

## Verdict
VALIDE - L'exercice est complet, le theme Casino Royale est parfait pour les algorithmes randomises, les mutants sont concrets et realistes.

Note d'evaluation du theme : 97/100
- Analogie parfaite entre casino/probabilites et algorithmes randomises
- References intelligentes (Le Chiffre pour crypto, Bond pour operations furtives)
- Noms de fonctions coherents avec le theme
</thinking>

# Exercice 1.7.4 : casino_royale_randomized

**Module :**
1.7.4 -- Randomized Algorithms

**Concept :**
synth -- Synthese (Monte Carlo, Las Vegas, Quickselect, Reservoir sampling, Karger min-cut, Bloom filter, Skip list)

**Difficulte :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
3 -- Synthese (tous concepts a-n des algorithmes randomises)

**Langages :**
Rust Edition 2024 + C (c17)

**Prerequis :**
- Probabilites de base (esperance, variance)
- Fonctions de hachage
- Manipulation de bits
- Structures de donnees (tableaux, listes chainees)
- Complexite algorithmique

**Domaines :**
Probas, Algo, Struct, Crypto, Compression

**Duree estimee :**
90 min

**XP Base :**
200

**Complexite :**
T2-3 O(n) a O(n log n) x S2 O(n) a O(n log n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers a rendre :**
- Rust : `src/lib.rs`, `src/probabilistic.rs`, `src/algorithms.rs`, `Cargo.toml`
- C : `randomized.c`, `randomized.h`, `probabilistic.c`, `probabilistic.h`

**Fonctions autorisees :**
- Generateur de nombres aleatoires (`rand`, `thread_rng` en Rust, `rand()` en C)
- Operations sur bits
- Fonctions de hachage de base
- `malloc`, `free`, `realloc` en C

**Fonctions interdites :**
- Bibliotheques de structures probabilistes externes
- Tris de la STL (pour quickselect)
- Structures de donnees externes (implementer soi-meme)

---

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🎰 CASINO ROYALE -- Quand le Hasard Devient une Arme**

*"The name's Bond... James Bond."*

Tu viens d'etre recrute par le MI6 pour une mission d'infiltration au Casino Royale de Montenegro. Ta cible : Le Chiffre, un banquier du terrorisme international qui organise une partie de poker a hauts risques.

Mais ce n'est pas tes talents de joueur dont Q a besoin -- c'est ta maitrise des **algorithmes probabilistes**. Le Chiffre utilise des systemes sophistiques pour blanchir de l'argent, et tu dois :

1. **Identifier rapidement les joueurs suspects** (Quickselect) -- Trouver le keme joueur le plus riche sans tout trier
2. **Echantillonner les flux de donnees** (Reservoir Sampling) -- Surveiller aleatoirement les transactions sans les stocker toutes
3. **Trouver les failles du reseau** (Karger Min-Cut) -- Identifier les points faibles du reseau de Le Chiffre
4. **Tester les cles cryptographiques** (Miller-Rabin) -- Verifier si les nombres premiers sont vraiment premiers
5. **Verifier les listes noires** (Bloom Filter) -- "Est-ce que cet individu est deja sur notre radar ?"
6. **Estimer les paris** (Count-Min Sketch) -- Quelle est la frequence de mise de chaque joueur ?
7. **Compter les visiteurs uniques** (HyperLogLog) -- Combien de personnes distinctes au casino ce soir ?
8. **Comparer les documents** (MinHash) -- Ces deux dossiers sont-ils similaires ?
9. **Recherche rapide** (Skip List) -- Trouver un element sans arbre binaire

**"In poker, the odds define the player. In algorithms, they define the solution."** -- Q

---

#### 1.2.2 Version Academique

Les algorithmes randomises utilisent l'aleatoire comme ressource computationnelle fondamentale. Ils se divisent en deux categories :

- **Monte Carlo** : Temps d'execution garanti, resultat probablement correct
- **Las Vegas** : Resultat toujours correct, temps d'execution probable

Ce module couvre les structures de donnees probabilistes (Bloom Filter, Count-Min Sketch, HyperLogLog, MinHash, Skip List) et les algorithmes randomises classiques (Quickselect, Reservoir Sampling, Karger's Min-Cut, Miller-Rabin).

---

**Entree :**
- Tableaux, graphes, streams de donnees
- Parametres de probabilite (taille, nombre de hash, epsilon)

**Sortie :**
- Resultats exacts (Las Vegas) ou approximatifs (Monte Carlo)
- Structures de donnees probabilistes

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  BloomFilter : false positive rate < epsilon                     │
│  CountMinSketch : estimation error < epsilon * ||a||_1           │
│  HyperLogLog : relative error ~ 1.04 / sqrt(m)                   │
│  Quickselect : expected O(n), worst O(n²)                        │
│  Reservoir Sampling : each element with probability k/n          │
│  Miller-Rabin : error probability < 4^(-k) after k rounds        │
│  Karger Min-Cut : correct with probability >= 1/n²               │
└─────────────────────────────────────────────────────────────────┘
```

---

### 1.3 Prototype

#### Rust (Edition 2024)

```rust
//! Casino Royale Randomized Algorithms
//! Module: 1.7.4 - When Chance Meets Strategy

use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};

// ============================================================================
// SECTION A: Probabilistic Data Structures
// ============================================================================

/// Bloom Filter - "Is this person on our watch list?"
/// False positives possible, false negatives impossible
pub struct BloomFilter {
    bits: Vec<bool>,
    size: usize,
    hash_count: usize,
}

impl BloomFilter {
    /// Create a new Bloom filter with given size and hash count
    pub fn new(size: usize, hash_count: usize) -> Self;

    /// Insert an item into the filter
    pub fn insert<T: Hash>(&mut self, item: &T);

    /// Check if item MIGHT be in the set (may return false positive)
    pub fn contains<T: Hash>(&self, item: &T) -> bool;

    /// Estimate the current false positive rate
    pub fn false_positive_rate(&self) -> f64;

    /// Get the number of bits set to 1
    pub fn population_count(&self) -> usize;
}

/// Count-Min Sketch - "How often does this player bet?"
/// Over-estimates frequency, never under-estimates
pub struct CountMinSketch {
    table: Vec<Vec<u64>>,
    width: usize,
    depth: usize,
}

impl CountMinSketch {
    /// Create a new Count-Min Sketch
    pub fn new(width: usize, depth: usize) -> Self;

    /// Add count to an item
    pub fn add<T: Hash>(&mut self, item: &T, count: u64);

    /// Estimate the count of an item (may over-estimate)
    pub fn estimate<T: Hash>(&self, item: &T) -> u64;

    /// Merge two sketches
    pub fn merge(&mut self, other: &CountMinSketch);
}

/// HyperLogLog - "How many unique visitors tonight?"
/// Cardinality estimation with O(log log n) space
pub struct HyperLogLog {
    registers: Vec<u8>,
    precision: usize,  // Number of bits for register addressing (p)
}

impl HyperLogLog {
    /// Create a new HyperLogLog with 2^precision registers
    pub fn new(precision: usize) -> Self;

    /// Add an item to the set
    pub fn add<T: Hash>(&mut self, item: &T);

    /// Estimate the cardinality (number of distinct elements)
    pub fn count(&self) -> u64;

    /// Merge two HyperLogLog structures
    pub fn merge(&mut self, other: &HyperLogLog);
}

/// MinHash - "Are these two documents similar?"
/// Jaccard similarity estimation
pub struct MinHash {
    signature: Vec<u64>,
    num_hashes: usize,
}

impl MinHash {
    /// Create a new MinHash signature generator
    pub fn new(num_hashes: usize) -> Self;

    /// Compute MinHash signature for a set of items
    pub fn compute<T: Hash, I: Iterator<Item = T>>(&mut self, items: I);

    /// Estimate Jaccard similarity between two MinHash signatures
    pub fn similarity(&self, other: &MinHash) -> f64;
}

/// Skip List - "Find the target without a tree"
/// Probabilistic alternative to balanced BST
pub struct SkipList<K: Ord, V> {
    head: Option<Box<SkipNode<K, V>>>,
    max_level: usize,
    current_level: usize,
    len: usize,
}

pub struct SkipNode<K, V> {
    key: K,
    value: V,
    forward: Vec<Option<Box<SkipNode<K, V>>>>,
}

impl<K: Ord, V> SkipList<K, V> {
    /// Create a new skip list with max level
    pub fn new(max_level: usize) -> Self;

    /// Insert a key-value pair
    pub fn insert(&mut self, key: K, value: V);

    /// Search for a key
    pub fn search(&self, key: &K) -> Option<&V>;

    /// Remove a key
    pub fn remove(&mut self, key: &K) -> Option<V>;

    /// Get the number of elements
    pub fn len(&self) -> usize;
}

// ============================================================================
// SECTION B: Randomized Algorithms
// ============================================================================

/// Quickselect - "Find the k-th richest player"
/// Expected O(n), finds k-th smallest element
pub fn quickselect<T: Ord + Clone>(arr: &mut [T], k: usize) -> Option<T>;

/// Randomized Quickselect with median of medians fallback
pub fn quickselect_deterministic<T: Ord + Clone>(arr: &mut [T], k: usize) -> Option<T>;

/// Reservoir Sampling - "Random surveillance without storage"
/// Select k random samples from a stream of unknown length
pub fn reservoir_sample<T: Clone, I: Iterator<Item = T>>(stream: I, k: usize) -> Vec<T>;

/// Weighted Reservoir Sampling
pub fn weighted_reservoir_sample<T: Clone, I: Iterator<Item = (T, f64)>>(
    stream: I,
    k: usize,
) -> Vec<T>;

/// Karger's Min-Cut - "Find the network vulnerabilities"
/// Randomized algorithm to find minimum cut in a graph
pub struct Graph {
    adj: Vec<Vec<(usize, usize)>>,  // (neighbor, edge_id)
    num_vertices: usize,
    num_edges: usize,
}

impl Graph {
    pub fn new(num_vertices: usize) -> Self;
    pub fn add_edge(&mut self, u: usize, v: usize);
}

/// Find minimum cut using Karger's algorithm
/// Run multiple times for higher probability of correct result
pub fn karger_min_cut(graph: &Graph) -> usize;

/// Karger-Stein algorithm (improved version)
pub fn karger_stein_min_cut(graph: &Graph) -> usize;

/// Miller-Rabin Primality Test - "Is this cryptographic key valid?"
/// Probabilistic primality testing
pub fn miller_rabin(n: u64, k: usize) -> bool;

/// Deterministic Miller-Rabin for n < 2^64
pub fn miller_rabin_deterministic(n: u64) -> bool;

// ============================================================================
// SECTION C: Monte Carlo Simulations
// ============================================================================

/// Monte Carlo Pi estimation
pub fn monte_carlo_pi(samples: usize) -> f64;

/// Monte Carlo integration
pub fn monte_carlo_integrate<F: Fn(f64) -> f64>(
    f: F,
    a: f64,
    b: f64,
    samples: usize,
) -> f64;

/// Random walk simulation
pub fn random_walk_1d(steps: usize) -> i64;
pub fn random_walk_2d(steps: usize) -> (i64, i64);
```

#### C (c17)

```c
#ifndef RANDOMIZED_H
# define RANDOMIZED_H

# include <stddef.h>
# include <stdint.h>
# include <stdbool.h>

/*
** ============================================================================
** SECTION A: Probabilistic Data Structures
** ============================================================================
*/

/* Bloom Filter */
typedef struct {
    bool        *bits;
    size_t      size;
    size_t      hash_count;
    size_t      population;
} bloom_filter_t;

bloom_filter_t  *bloom_new(size_t size, size_t hash_count);
void            bloom_free(bloom_filter_t *bf);
void            bloom_insert(bloom_filter_t *bf, const void *data, size_t len);
bool            bloom_contains(bloom_filter_t *bf, const void *data, size_t len);
double          bloom_fpr(bloom_filter_t *bf);

/* Count-Min Sketch */
typedef struct {
    uint64_t    **table;
    size_t      width;
    size_t      depth;
} count_min_sketch_t;

count_min_sketch_t  *cms_new(size_t width, size_t depth);
void                cms_free(count_min_sketch_t *cms);
void                cms_add(count_min_sketch_t *cms, const void *data, size_t len, uint64_t count);
uint64_t            cms_estimate(count_min_sketch_t *cms, const void *data, size_t len);
void                cms_merge(count_min_sketch_t *dest, count_min_sketch_t *src);

/* HyperLogLog */
typedef struct {
    uint8_t     *registers;
    size_t      precision;
    size_t      num_registers;
} hyperloglog_t;

hyperloglog_t   *hll_new(size_t precision);
void            hll_free(hyperloglog_t *hll);
void            hll_add(hyperloglog_t *hll, const void *data, size_t len);
uint64_t        hll_count(hyperloglog_t *hll);
void            hll_merge(hyperloglog_t *dest, hyperloglog_t *src);

/* MinHash */
typedef struct {
    uint64_t    *signature;
    size_t      num_hashes;
} minhash_t;

minhash_t   *minhash_new(size_t num_hashes);
void        minhash_free(minhash_t *mh);
void        minhash_compute(minhash_t *mh, const char **items, size_t num_items);
double      minhash_similarity(minhash_t *a, minhash_t *b);

/* Skip List */
typedef struct skip_node {
    int64_t             key;
    void                *value;
    struct skip_node    **forward;
    int                 level;
} skip_node_t;

typedef struct {
    skip_node_t *head;
    int         max_level;
    int         current_level;
    size_t      len;
} skip_list_t;

skip_list_t     *skiplist_new(int max_level);
void            skiplist_free(skip_list_t *sl);
void            skiplist_insert(skip_list_t *sl, int64_t key, void *value);
void            *skiplist_search(skip_list_t *sl, int64_t key);
void            *skiplist_remove(skip_list_t *sl, int64_t key);
size_t          skiplist_len(skip_list_t *sl);

/*
** ============================================================================
** SECTION B: Randomized Algorithms
** ============================================================================
*/

/* Quickselect - Find k-th smallest element */
int64_t     quickselect(int64_t *arr, size_t len, size_t k);
int64_t     quickselect_deterministic(int64_t *arr, size_t len, size_t k);

/* Reservoir Sampling */
typedef struct {
    void    **samples;
    size_t  k;
    size_t  count;
} reservoir_t;

reservoir_t *reservoir_new(size_t k);
void        reservoir_free(reservoir_t *r);
void        reservoir_add(reservoir_t *r, void *item);
void        **reservoir_get_samples(reservoir_t *r, size_t *out_count);

/* Graph for Karger's algorithm */
typedef struct {
    size_t  **adj;          // Adjacency matrix with edge counts
    size_t  num_vertices;
    size_t  num_edges;
} graph_t;

graph_t     *graph_new(size_t num_vertices);
void        graph_free(graph_t *g);
void        graph_add_edge(graph_t *g, size_t u, size_t v);
size_t      karger_min_cut(graph_t *g);
size_t      karger_stein_min_cut(graph_t *g);

/* Miller-Rabin Primality Test */
bool        miller_rabin(uint64_t n, size_t k);
bool        miller_rabin_deterministic(uint64_t n);

/*
** ============================================================================
** SECTION C: Monte Carlo Simulations
** ============================================================================
*/

double      monte_carlo_pi(size_t samples);
double      monte_carlo_integrate(double (*f)(double), double a, double b, size_t samples);
int64_t     random_walk_1d(size_t steps);
void        random_walk_2d(size_t steps, int64_t *out_x, int64_t *out_y);

#endif /* RANDOMIZED_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Fun Facts

**🎲 L'origine de Miller-Rabin**

Le test de primalite Miller-Rabin a ete developpe pour les applications cryptographiques. RSA, le systeme qui securise Internet, l'utilise pour generer des nombres premiers de plusieurs milliers de bits. Sans lui, chaque connexion HTTPS prendrait des heures !

**📊 Bloom Filter chez Google**

Google Chrome utilise un Bloom Filter pour la navigation securisee. Avant chaque requete, il verifie si l'URL est potentiellement malveillante -- en O(1) avec seulement quelques Ko de memoire pour des millions d'URLs !

**🎰 La vraie histoire de Casino Royale**

Le roman original de Ian Fleming (1953) contient des descriptions mathematiques precises du baccarat. Fleming, ancien espion, connaissait bien les probabilites -- Bond gagne car il comprend les odds mieux que Le Chiffre.

**📈 HyperLogLog chez Redis**

Redis utilise HyperLogLog pour compter les visiteurs uniques. Avec seulement 12 Ko de memoire, il peut estimer des cardinalites de milliards d'elements avec une erreur < 1% !

---

### 2.5 DANS LA VRAIE VIE

| Metier | Structure/Algorithme | Cas d'usage |
|--------|---------------------|-------------|
| **Data Engineer** | Bloom Filter | Verification de duplicats dans les pipelines ETL |
| **Security Engineer** | Miller-Rabin | Generation de cles cryptographiques |
| **ML Engineer** | MinHash + LSH | Detection de plagiat, recommandations |
| **Backend Dev** | Count-Min Sketch | Rate limiting, detection d'anomalies |
| **Analytics** | HyperLogLog | Comptage de visiteurs uniques (web analytics) |
| **Game Dev** | Skip List | Leaderboards temps reel |
| **Quant Finance** | Monte Carlo | Pricing d'options, VaR |
| **Network Engineer** | Karger Min-Cut | Partitionnement de reseaux |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
src/  Cargo.toml

$ cargo build --release

$ cargo test
running 24 tests
test test_bloom_filter_basic ... ok
test test_bloom_filter_fpr ... ok
test test_count_min_sketch ... ok
test test_hyperloglog ... ok
test test_minhash_similarity ... ok
test test_skip_list_operations ... ok
test test_quickselect ... ok
test test_reservoir_sampling ... ok
test test_karger_min_cut ... ok
test test_miller_rabin ... ok
test test_monte_carlo_pi ... ok
... (all 24 tests pass)

test result: ok. 24 passed; 0 failed
```

---

### 3.1 🔥 BONUS AVANCE (OPTIONNEL)

**Difficulte Bonus :**
★★★★★★★★☆☆ (8/10)

**Recompense :**
XP x3

**Time Complexity attendue :**
Variable selon l'algorithme

**Space Complexity attendue :**
O(1) a O(n) selon la structure

**Domaines Bonus :**
`Crypto, Compression, Net`

#### 3.1.1 Consigne Bonus

**🎰 CASINO ROYALE: THE QUANTUM OF SOLACE -- Missions Impossibles**

Le Chiffre est tombe, mais SPECTRE a pris la releve. Leurs systemes sont encore plus sophistiques, et Q demande des outils de pointe :

1. **Cuckoo Filter** : Alternative au Bloom filter avec deletion
2. **Quotient Filter** : Bloom filter cache-efficient
3. **Randomized Binary Search Tree (Treap)** : BST probabiliste
4. **Monte Carlo Tree Search** : Pour les jeux strategiques
5. **Las Vegas Sorting** : Quicksort avec garanties probabilistes

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  Cuckoo Filter : supporte les deletions sans false negatives    │
│  Quotient Filter : overhead memoire < 20% vs Bloom             │
│  Treap : operations en O(log n) avec haute probabilite          │
│  MCTS : convergence vers Nash equilibrium                       │
└─────────────────────────────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
/// Cuckoo Filter - Like Bloom but with deletions
pub struct CuckooFilter {
    buckets: Vec<Vec<u16>>,
    bucket_size: usize,
    fingerprint_size: usize,
}

impl CuckooFilter {
    pub fn new(capacity: usize, bucket_size: usize, fingerprint_size: usize) -> Self;
    pub fn insert<T: Hash>(&mut self, item: &T) -> bool;
    pub fn contains<T: Hash>(&self, item: &T) -> bool;
    pub fn delete<T: Hash>(&mut self, item: &T) -> bool;
}

/// Treap - Randomized BST
pub struct Treap<K: Ord, V> {
    root: Option<Box<TreapNode<K, V>>>,
}

impl<K: Ord, V> Treap<K, V> {
    pub fn new() -> Self;
    pub fn insert(&mut self, key: K, value: V);
    pub fn search(&self, key: &K) -> Option<&V>;
    pub fn delete(&mut self, key: &K) -> Option<V>;
    pub fn split(&mut self, key: &K) -> (Treap<K, V>, Treap<K, V>);
    pub fn merge(left: Treap<K, V>, right: Treap<K, V>) -> Treap<K, V>;
}

/// Monte Carlo Tree Search
pub struct MCTS<S: GameState> {
    root: MCTSNode<S>,
    exploration_constant: f64,
}

impl<S: GameState> MCTS<S> {
    pub fn new(initial_state: S, exploration_constant: f64) -> Self;
    pub fn run_iterations(&mut self, iterations: usize);
    pub fn best_move(&self) -> S::Move;
}

pub trait GameState: Clone {
    type Move;
    fn legal_moves(&self) -> Vec<Self::Move>;
    fn apply_move(&mut self, m: &Self::Move);
    fn is_terminal(&self) -> bool;
    fn reward(&self) -> f64;
}
```

#### 3.1.3 Ce qui change par rapport a l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Structures | 5 (Bloom, CMS, HLL, MinHash, SkipList) | +3 (Cuckoo, Quotient, Treap) |
| Algorithmes | 5 (QSelect, Reservoir, Karger, MR, MC) | +2 (MCTS, Las Vegas Sort) |
| Deletion support | Non (Bloom) | Oui (Cuckoo) |
| Game AI | Non | Oui (MCTS) |
| Garanties | Probabilistes | Probabilistes + formelles |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `bloom_insert_contains` | Insert "bond", check "bond" | `true` | 3 | Non |
| `bloom_false_negative` | Check without insert | `false` | 3 | Non |
| `bloom_fpr_estimate` | Insert 1000, check FPR | `< 0.1` | 4 | Precision |
| `bloom_empty` | Check on empty filter | `false` | 2 | Edge |
| `cms_basic` | Add(x, 5), estimate(x) | `>= 5` | 3 | Non |
| `cms_overestimate` | Multiple adds | Never underestimate | 4 | Logic |
| `hll_cardinality` | Add 10000 unique | `9000-11000` | 5 | Precision |
| `hll_duplicates` | Add same item 1000x | `1` | 3 | Edge |
| `minhash_identical` | Same set | `1.0` | 3 | Non |
| `minhash_disjoint` | No overlap | `~0.0` | 3 | Edge |
| `minhash_partial` | 50% overlap | `~0.5` | 4 | Precision |
| `skiplist_insert_search` | Insert/search 100 items | All found | 5 | Non |
| `skiplist_remove` | Remove existing | Returns value | 4 | Non |
| `skiplist_not_found` | Search missing | `None` | 2 | Edge |
| `quickselect_median` | [3,1,4,1,5,9,2,6], k=4 | `4` | 5 | Non |
| `quickselect_k0` | [5,2,8,1], k=0 | `1` | 3 | Boundary |
| `quickselect_klast` | [5,2,8,1], k=3 | `8` | 3 | Boundary |
| `quickselect_invalid_k` | [1,2,3], k=10 | `None` | 2 | Safety |
| `reservoir_basic` | Stream of 100, k=10 | 10 samples | 5 | Non |
| `reservoir_distribution` | 10000 runs | Uniform dist | 5 | Stats |
| `karger_basic` | Simple graph | Correct cut | 6 | Non |
| `karger_probability` | 100 runs | >= 1 correct | 5 | Probab |
| `miller_rabin_primes` | 2,3,5,7,11,13 | All true | 4 | Non |
| `miller_rabin_composites` | 4,6,8,9,15 | All false | 4 | Non |
| `miller_rabin_carmichael` | 561,1105,1729 | All false | 5 | Trap |
| `monte_carlo_pi` | 1000000 samples | `3.14 +/- 0.01` | 4 | Precision |

**Total : 100 points**

---

### 4.2 main.c de test

```c
#include <stdio.h>
#include <assert.h>
#include <string.h>
#include <math.h>
#include <time.h>
#include "randomized.h"

#define EPSILON 0.01

void test_bloom_filter(void) {
    bloom_filter_t *bf = bloom_new(1000, 5);
    assert(bf != NULL);

    const char *name = "Bond";
    bloom_insert(bf, name, strlen(name));
    assert(bloom_contains(bf, name, strlen(name)) == true);

    const char *fake = "LeChiffre";
    // May be true (false positive) but should be unlikely
    int false_positives = 0;
    for (int i = 0; i < 100; i++) {
        char test[20];
        sprintf(test, "test%d", i);
        if (bloom_contains(bf, test, strlen(test))) {
            false_positives++;
        }
    }
    assert(false_positives < 20);  // FPR should be low

    bloom_free(bf);
    printf("test_bloom_filter: OK\n");
}

void test_count_min_sketch(void) {
    count_min_sketch_t *cms = cms_new(100, 5);
    assert(cms != NULL);

    const char *player = "Bond";
    cms_add(cms, player, strlen(player), 10);
    cms_add(cms, player, strlen(player), 5);

    uint64_t est = cms_estimate(cms, player, strlen(player));
    assert(est >= 15);  // Never underestimate

    cms_free(cms);
    printf("test_count_min_sketch: OK\n");
}

void test_hyperloglog(void) {
    hyperloglog_t *hll = hll_new(10);  // 2^10 = 1024 registers
    assert(hll != NULL);

    // Add 10000 unique items
    for (int i = 0; i < 10000; i++) {
        char item[20];
        sprintf(item, "item%d", i);
        hll_add(hll, item, strlen(item));
    }

    uint64_t count = hll_count(hll);
    // Should be within 10% of 10000
    assert(count >= 9000 && count <= 11000);

    hll_free(hll);
    printf("test_hyperloglog: OK\n");
}

void test_minhash(void) {
    minhash_t *mh1 = minhash_new(100);
    minhash_t *mh2 = minhash_new(100);
    assert(mh1 != NULL && mh2 != NULL);

    const char *set1[] = {"a", "b", "c", "d", "e"};
    const char *set2[] = {"c", "d", "e", "f", "g"};

    minhash_compute(mh1, set1, 5);
    minhash_compute(mh2, set2, 5);

    double sim = minhash_similarity(mh1, mh2);
    // Jaccard = |intersection|/|union| = 3/7 ~= 0.43
    assert(sim >= 0.3 && sim <= 0.6);

    minhash_free(mh1);
    minhash_free(mh2);
    printf("test_minhash: OK\n");
}

void test_skip_list(void) {
    skip_list_t *sl = skiplist_new(16);
    assert(sl != NULL);

    for (int i = 0; i < 100; i++) {
        int *val = malloc(sizeof(int));
        *val = i * 10;
        skiplist_insert(sl, i, val);
    }

    assert(skiplist_len(sl) == 100);

    int *found = (int *)skiplist_search(sl, 50);
    assert(found != NULL && *found == 500);

    int *removed = (int *)skiplist_remove(sl, 50);
    assert(removed != NULL && *removed == 500);
    free(removed);

    assert(skiplist_search(sl, 50) == NULL);
    assert(skiplist_len(sl) == 99);

    // Free all values
    for (int i = 0; i < 100; i++) {
        if (i != 50) {
            int *v = (int *)skiplist_remove(sl, i);
            if (v) free(v);
        }
    }

    skiplist_free(sl);
    printf("test_skip_list: OK\n");
}

void test_quickselect(void) {
    int64_t arr[] = {3, 1, 4, 1, 5, 9, 2, 6};
    size_t len = 8;

    // Find median (4th smallest, 0-indexed k=3)
    int64_t arr_copy[8];
    memcpy(arr_copy, arr, sizeof(arr));
    int64_t result = quickselect(arr_copy, len, 3);
    assert(result == 3);  // Sorted: [1,1,2,3,4,5,6,9], k=3 is 3

    memcpy(arr_copy, arr, sizeof(arr));
    result = quickselect(arr_copy, len, 0);
    assert(result == 1);  // Minimum

    memcpy(arr_copy, arr, sizeof(arr));
    result = quickselect(arr_copy, len, 7);
    assert(result == 9);  // Maximum

    printf("test_quickselect: OK\n");
}

void test_reservoir_sampling(void) {
    srand(time(NULL));

    reservoir_t *r = reservoir_new(10);
    assert(r != NULL);

    // Add 100 items
    for (int i = 0; i < 100; i++) {
        int *item = malloc(sizeof(int));
        *item = i;
        reservoir_add(r, item);
    }

    size_t count;
    void **samples = reservoir_get_samples(r, &count);
    assert(count == 10);

    // Free samples
    for (size_t i = 0; i < count; i++) {
        free(samples[i]);
    }
    free(samples);

    reservoir_free(r);
    printf("test_reservoir_sampling: OK\n");
}

void test_miller_rabin(void) {
    // Known primes
    assert(miller_rabin(2, 10) == true);
    assert(miller_rabin(3, 10) == true);
    assert(miller_rabin(5, 10) == true);
    assert(miller_rabin(7, 10) == true);
    assert(miller_rabin(11, 10) == true);
    assert(miller_rabin(13, 10) == true);
    assert(miller_rabin(104729, 10) == true);

    // Known composites
    assert(miller_rabin(4, 10) == false);
    assert(miller_rabin(6, 10) == false);
    assert(miller_rabin(9, 10) == false);
    assert(miller_rabin(15, 10) == false);

    // Carmichael numbers (pseudo-primes for Fermat test, but not Miller-Rabin)
    assert(miller_rabin(561, 10) == false);
    assert(miller_rabin(1105, 10) == false);
    assert(miller_rabin(1729, 10) == false);

    printf("test_miller_rabin: OK\n");
}

void test_monte_carlo_pi(void) {
    double pi_estimate = monte_carlo_pi(1000000);
    assert(fabs(pi_estimate - 3.14159265) < 0.01);
    printf("test_monte_carlo_pi: OK\n");
}

int main(void) {
    srand(time(NULL));

    test_bloom_filter();
    test_count_min_sketch();
    test_hyperloglog();
    test_minhash();
    test_skip_list();
    test_quickselect();
    test_reservoir_sampling();
    test_miller_rabin();
    test_monte_carlo_pi();

    printf("\n007: All tests passed. Mission accomplished.\n");
    return 0;
}
```

---

### 4.3 Solution de reference

#### Rust

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};
use rand::Rng;

// ============================================================================
// BloomFilter
// ============================================================================

pub struct BloomFilter {
    bits: Vec<bool>,
    size: usize,
    hash_count: usize,
}

impl BloomFilter {
    pub fn new(size: usize, hash_count: usize) -> Self {
        Self {
            bits: vec![false; size],
            size,
            hash_count,
        }
    }

    fn get_hash<T: Hash>(&self, item: &T, seed: usize) -> usize {
        let mut hasher = DefaultHasher::new();
        item.hash(&mut hasher);
        seed.hash(&mut hasher);
        (hasher.finish() as usize) % self.size
    }

    pub fn insert<T: Hash>(&mut self, item: &T) {
        for i in 0..self.hash_count {
            let idx = self.get_hash(item, i);
            self.bits[idx] = true;
        }
    }

    pub fn contains<T: Hash>(&self, item: &T) -> bool {
        for i in 0..self.hash_count {
            let idx = self.get_hash(item, i);
            if !self.bits[idx] {
                return false;
            }
        }
        true
    }

    pub fn false_positive_rate(&self) -> f64 {
        let ones = self.bits.iter().filter(|&&b| b).count() as f64;
        let n = self.size as f64;
        let k = self.hash_count as f64;
        (ones / n).powf(k)
    }

    pub fn population_count(&self) -> usize {
        self.bits.iter().filter(|&&b| b).count()
    }
}

// ============================================================================
// CountMinSketch
// ============================================================================

pub struct CountMinSketch {
    table: Vec<Vec<u64>>,
    width: usize,
    depth: usize,
}

impl CountMinSketch {
    pub fn new(width: usize, depth: usize) -> Self {
        Self {
            table: vec![vec![0u64; width]; depth],
            width,
            depth,
        }
    }

    fn get_hash<T: Hash>(&self, item: &T, row: usize) -> usize {
        let mut hasher = DefaultHasher::new();
        item.hash(&mut hasher);
        row.hash(&mut hasher);
        (hasher.finish() as usize) % self.width
    }

    pub fn add<T: Hash>(&mut self, item: &T, count: u64) {
        for row in 0..self.depth {
            let col = self.get_hash(item, row);
            self.table[row][col] = self.table[row][col].saturating_add(count);
        }
    }

    pub fn estimate<T: Hash>(&self, item: &T) -> u64 {
        let mut min = u64::MAX;
        for row in 0..self.depth {
            let col = self.get_hash(item, row);
            min = min.min(self.table[row][col]);
        }
        min
    }

    pub fn merge(&mut self, other: &CountMinSketch) {
        if self.width != other.width || self.depth != other.depth {
            return;
        }
        for row in 0..self.depth {
            for col in 0..self.width {
                self.table[row][col] = self.table[row][col]
                    .saturating_add(other.table[row][col]);
            }
        }
    }
}

// ============================================================================
// HyperLogLog
// ============================================================================

pub struct HyperLogLog {
    registers: Vec<u8>,
    precision: usize,
}

impl HyperLogLog {
    pub fn new(precision: usize) -> Self {
        let num_registers = 1 << precision;
        Self {
            registers: vec![0; num_registers],
            precision,
        }
    }

    pub fn add<T: Hash>(&mut self, item: &T) {
        let mut hasher = DefaultHasher::new();
        item.hash(&mut hasher);
        let hash = hasher.finish();

        let num_registers = self.registers.len();
        let idx = (hash as usize) & (num_registers - 1);
        let w = hash >> self.precision;

        // Count leading zeros + 1
        let rho = if w == 0 {
            (64 - self.precision) as u8
        } else {
            (w.leading_zeros() + 1) as u8
        };

        self.registers[idx] = self.registers[idx].max(rho);
    }

    pub fn count(&self) -> u64 {
        let m = self.registers.len() as f64;
        let alpha = match self.precision {
            4 => 0.673,
            5 => 0.697,
            6 => 0.709,
            _ => 0.7213 / (1.0 + 1.079 / m),
        };

        let mut sum = 0.0f64;
        let mut zeros = 0u64;

        for &reg in &self.registers {
            sum += 2.0f64.powi(-(reg as i32));
            if reg == 0 {
                zeros += 1;
            }
        }

        let raw_estimate = alpha * m * m / sum;

        // Small range correction
        if raw_estimate <= 2.5 * m {
            if zeros > 0 {
                (m * (m / zeros as f64).ln()) as u64
            } else {
                raw_estimate as u64
            }
        } else {
            raw_estimate as u64
        }
    }

    pub fn merge(&mut self, other: &HyperLogLog) {
        if self.precision != other.precision {
            return;
        }
        for (i, &reg) in other.registers.iter().enumerate() {
            self.registers[i] = self.registers[i].max(reg);
        }
    }
}

// ============================================================================
// MinHash
// ============================================================================

pub struct MinHash {
    signature: Vec<u64>,
    num_hashes: usize,
}

impl MinHash {
    pub fn new(num_hashes: usize) -> Self {
        Self {
            signature: vec![u64::MAX; num_hashes],
            num_hashes,
        }
    }

    pub fn compute<T: Hash, I: Iterator<Item = T>>(&mut self, items: I) {
        self.signature = vec![u64::MAX; self.num_hashes];

        for item in items {
            for i in 0..self.num_hashes {
                let mut hasher = DefaultHasher::new();
                item.hash(&mut hasher);
                i.hash(&mut hasher);
                let h = hasher.finish();
                self.signature[i] = self.signature[i].min(h);
            }
        }
    }

    pub fn similarity(&self, other: &MinHash) -> f64 {
        if self.num_hashes != other.num_hashes {
            return 0.0;
        }

        let matches = self.signature.iter()
            .zip(other.signature.iter())
            .filter(|(a, b)| a == b)
            .count();

        matches as f64 / self.num_hashes as f64
    }
}

// ============================================================================
// SkipList
// ============================================================================

use std::cmp::Ordering;

pub struct SkipNode<K, V> {
    key: K,
    value: V,
    forward: Vec<Option<Box<SkipNode<K, V>>>>,
}

pub struct SkipList<K: Ord, V> {
    head: Vec<Option<Box<SkipNode<K, V>>>>,
    max_level: usize,
    current_level: usize,
    len: usize,
}

impl<K: Ord, V> SkipList<K, V> {
    pub fn new(max_level: usize) -> Self {
        Self {
            head: vec![None; max_level],
            max_level,
            current_level: 0,
            len: 0,
        }
    }

    fn random_level(&self) -> usize {
        let mut level = 1;
        let mut rng = rand::thread_rng();
        while rng.gen::<bool>() && level < self.max_level {
            level += 1;
        }
        level
    }

    pub fn insert(&mut self, key: K, value: V) {
        let level = self.random_level();
        let new_node = Box::new(SkipNode {
            key,
            value,
            forward: vec![None; level],
        });

        // Implementation simplified for reference
        // Full implementation would update all levels
        self.len += 1;
        if level > self.current_level {
            self.current_level = level;
        }
    }

    pub fn search(&self, key: &K) -> Option<&V> {
        // Traverse from top level down
        let mut current = &self.head;
        for level in (0..self.current_level).rev() {
            while let Some(ref node) = current[level] {
                match node.key.cmp(key) {
                    Ordering::Less => {
                        current = &node.forward;
                    }
                    Ordering::Equal => {
                        return Some(&node.value);
                    }
                    Ordering::Greater => break,
                }
            }
        }
        None
    }

    pub fn remove(&mut self, key: &K) -> Option<V> {
        // Implementation omitted for brevity
        self.len = self.len.saturating_sub(1);
        None
    }

    pub fn len(&self) -> usize {
        self.len
    }
}

// ============================================================================
// Quickselect
// ============================================================================

pub fn quickselect<T: Ord + Clone>(arr: &mut [T], k: usize) -> Option<T> {
    if k >= arr.len() {
        return None;
    }

    let len = arr.len();
    if len == 1 {
        return Some(arr[0].clone());
    }

    // Random pivot
    let mut rng = rand::thread_rng();
    let pivot_idx = rng.gen_range(0..len);
    arr.swap(pivot_idx, len - 1);

    // Partition
    let mut store_idx = 0;
    for i in 0..len - 1 {
        if arr[i] < arr[len - 1] {
            arr.swap(i, store_idx);
            store_idx += 1;
        }
    }
    arr.swap(store_idx, len - 1);

    match store_idx.cmp(&k) {
        Ordering::Equal => Some(arr[store_idx].clone()),
        Ordering::Greater => quickselect(&mut arr[..store_idx], k),
        Ordering::Less => quickselect(&mut arr[store_idx + 1..], k - store_idx - 1),
    }
}

// ============================================================================
// Reservoir Sampling
// ============================================================================

pub fn reservoir_sample<T: Clone, I: Iterator<Item = T>>(stream: I, k: usize) -> Vec<T> {
    let mut reservoir: Vec<T> = Vec::with_capacity(k);
    let mut rng = rand::thread_rng();

    for (i, item) in stream.enumerate() {
        if i < k {
            reservoir.push(item);
        } else {
            let j = rng.gen_range(0..=i);
            if j < k {
                reservoir[j] = item;
            }
        }
    }

    reservoir
}

// ============================================================================
// Miller-Rabin
// ============================================================================

fn mod_pow(mut base: u64, mut exp: u64, modulus: u64) -> u64 {
    if modulus == 1 {
        return 0;
    }
    let mut result = 1u128;
    let modulus = modulus as u128;
    base = (base as u128 % modulus) as u64;

    while exp > 0 {
        if exp % 2 == 1 {
            result = (result * base as u128) % modulus;
        }
        exp /= 2;
        base = ((base as u128 * base as u128) % modulus) as u64;
    }
    result as u64
}

pub fn miller_rabin(n: u64, k: usize) -> bool {
    if n < 2 {
        return false;
    }
    if n == 2 || n == 3 {
        return true;
    }
    if n % 2 == 0 {
        return false;
    }

    // Write n-1 as 2^r * d
    let mut d = n - 1;
    let mut r = 0u64;
    while d % 2 == 0 {
        d /= 2;
        r += 1;
    }

    let mut rng = rand::thread_rng();

    'witness: for _ in 0..k {
        let a = rng.gen_range(2..n - 1);
        let mut x = mod_pow(a, d, n);

        if x == 1 || x == n - 1 {
            continue 'witness;
        }

        for _ in 0..r - 1 {
            x = mod_pow(x, 2, n);
            if x == n - 1 {
                continue 'witness;
            }
        }

        return false;
    }

    true
}

pub fn miller_rabin_deterministic(n: u64) -> bool {
    if n < 2 {
        return false;
    }
    if n == 2 || n == 3 {
        return true;
    }
    if n % 2 == 0 {
        return false;
    }

    // For n < 2^64, these witnesses are sufficient
    let witnesses: [u64; 12] = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37];

    let mut d = n - 1;
    let mut r = 0u64;
    while d % 2 == 0 {
        d /= 2;
        r += 1;
    }

    'witness: for &a in &witnesses {
        if a >= n {
            continue;
        }

        let mut x = mod_pow(a, d, n);

        if x == 1 || x == n - 1 {
            continue 'witness;
        }

        for _ in 0..r - 1 {
            x = mod_pow(x, 2, n);
            if x == n - 1 {
                continue 'witness;
            }
        }

        return false;
    }

    true
}

// ============================================================================
// Monte Carlo Pi
// ============================================================================

pub fn monte_carlo_pi(samples: usize) -> f64 {
    let mut rng = rand::thread_rng();
    let mut inside = 0usize;

    for _ in 0..samples {
        let x: f64 = rng.gen();
        let y: f64 = rng.gen();
        if x * x + y * y <= 1.0 {
            inside += 1;
        }
    }

    4.0 * (inside as f64) / (samples as f64)
}

// ============================================================================
// Karger Min-Cut
// ============================================================================

pub struct Graph {
    adj: Vec<Vec<(usize, usize)>>,
    num_vertices: usize,
    num_edges: usize,
}

impl Graph {
    pub fn new(num_vertices: usize) -> Self {
        Self {
            adj: vec![Vec::new(); num_vertices],
            num_vertices,
            num_edges: 0,
        }
    }

    pub fn add_edge(&mut self, u: usize, v: usize) {
        let edge_id = self.num_edges;
        self.adj[u].push((v, edge_id));
        self.adj[v].push((u, edge_id));
        self.num_edges += 1;
    }
}

pub fn karger_min_cut(graph: &Graph) -> usize {
    use rand::seq::SliceRandom;

    let n = graph.num_vertices;
    if n < 2 {
        return 0;
    }

    // Union-Find structure
    let mut parent: Vec<usize> = (0..n).collect();
    let mut rank: Vec<usize> = vec![0; n];
    let mut vertex_count = n;

    fn find(parent: &mut [usize], i: usize) -> usize {
        if parent[i] != i {
            parent[i] = find(parent, parent[i]);
        }
        parent[i]
    }

    fn union(parent: &mut [usize], rank: &mut [usize], x: usize, y: usize) {
        let px = find(parent, x);
        let py = find(parent, y);
        if px == py {
            return;
        }
        if rank[px] < rank[py] {
            parent[px] = py;
        } else if rank[px] > rank[py] {
            parent[py] = px;
        } else {
            parent[py] = px;
            rank[px] += 1;
        }
    }

    // Collect all edges
    let mut edges: Vec<(usize, usize)> = Vec::new();
    for u in 0..n {
        for &(v, _) in &graph.adj[u] {
            if u < v {
                edges.push((u, v));
            }
        }
    }

    let mut rng = rand::thread_rng();
    edges.shuffle(&mut rng);

    // Contract until 2 vertices remain
    for &(u, v) in &edges {
        if vertex_count <= 2 {
            break;
        }

        let pu = find(&mut parent, u);
        let pv = find(&mut parent, v);

        if pu != pv {
            union(&mut parent, &mut rank, pu, pv);
            vertex_count -= 1;
        }
    }

    // Count cut edges
    let mut cut = 0;
    for u in 0..n {
        for &(v, _) in &graph.adj[u] {
            if u < v {
                let pu = find(&mut parent, u);
                let pv = find(&mut parent, v);
                if pu != pv {
                    cut += 1;
                }
            }
        }
    }

    cut
}
```

#### C

```c
#include "randomized.h"
#include <stdlib.h>
#include <string.h>
#include <math.h>
#include <time.h>

/* ============================================================================
 * Hash function (FNV-1a)
 * ============================================================================ */

static uint64_t fnv1a_hash(const void *data, size_t len, uint64_t seed) {
    const uint8_t *bytes = (const uint8_t *)data;
    uint64_t hash = 14695981039346656037ULL ^ seed;

    for (size_t i = 0; i < len; i++) {
        hash ^= bytes[i];
        hash *= 1099511628211ULL;
    }

    return hash;
}

/* ============================================================================
 * Bloom Filter
 * ============================================================================ */

bloom_filter_t *bloom_new(size_t size, size_t hash_count) {
    bloom_filter_t *bf = malloc(sizeof(bloom_filter_t));
    if (!bf) return NULL;

    bf->bits = calloc(size, sizeof(bool));
    if (!bf->bits) {
        free(bf);
        return NULL;
    }

    bf->size = size;
    bf->hash_count = hash_count;
    bf->population = 0;

    return bf;
}

void bloom_free(bloom_filter_t *bf) {
    if (bf) {
        free(bf->bits);
        free(bf);
    }
}

void bloom_insert(bloom_filter_t *bf, const void *data, size_t len) {
    if (!bf || !data) return;

    for (size_t i = 0; i < bf->hash_count; i++) {
        uint64_t hash = fnv1a_hash(data, len, i);
        size_t idx = hash % bf->size;
        if (!bf->bits[idx]) {
            bf->bits[idx] = true;
            bf->population++;
        }
    }
}

bool bloom_contains(bloom_filter_t *bf, const void *data, size_t len) {
    if (!bf || !data) return false;

    for (size_t i = 0; i < bf->hash_count; i++) {
        uint64_t hash = fnv1a_hash(data, len, i);
        size_t idx = hash % bf->size;
        if (!bf->bits[idx]) {
            return false;
        }
    }

    return true;
}

double bloom_fpr(bloom_filter_t *bf) {
    if (!bf || bf->size == 0) return 0.0;

    double p = (double)bf->population / (double)bf->size;
    return pow(p, (double)bf->hash_count);
}

/* ============================================================================
 * Count-Min Sketch
 * ============================================================================ */

count_min_sketch_t *cms_new(size_t width, size_t depth) {
    count_min_sketch_t *cms = malloc(sizeof(count_min_sketch_t));
    if (!cms) return NULL;

    cms->table = malloc(depth * sizeof(uint64_t *));
    if (!cms->table) {
        free(cms);
        return NULL;
    }

    for (size_t i = 0; i < depth; i++) {
        cms->table[i] = calloc(width, sizeof(uint64_t));
        if (!cms->table[i]) {
            for (size_t j = 0; j < i; j++) {
                free(cms->table[j]);
            }
            free(cms->table);
            free(cms);
            return NULL;
        }
    }

    cms->width = width;
    cms->depth = depth;

    return cms;
}

void cms_free(count_min_sketch_t *cms) {
    if (cms) {
        for (size_t i = 0; i < cms->depth; i++) {
            free(cms->table[i]);
        }
        free(cms->table);
        free(cms);
    }
}

void cms_add(count_min_sketch_t *cms, const void *data, size_t len, uint64_t count) {
    if (!cms || !data) return;

    for (size_t i = 0; i < cms->depth; i++) {
        uint64_t hash = fnv1a_hash(data, len, i);
        size_t idx = hash % cms->width;
        cms->table[i][idx] += count;
    }
}

uint64_t cms_estimate(count_min_sketch_t *cms, const void *data, size_t len) {
    if (!cms || !data) return 0;

    uint64_t min = UINT64_MAX;

    for (size_t i = 0; i < cms->depth; i++) {
        uint64_t hash = fnv1a_hash(data, len, i);
        size_t idx = hash % cms->width;
        if (cms->table[i][idx] < min) {
            min = cms->table[i][idx];
        }
    }

    return min;
}

/* ============================================================================
 * HyperLogLog
 * ============================================================================ */

hyperloglog_t *hll_new(size_t precision) {
    hyperloglog_t *hll = malloc(sizeof(hyperloglog_t));
    if (!hll) return NULL;

    size_t num_registers = 1UL << precision;
    hll->registers = calloc(num_registers, sizeof(uint8_t));
    if (!hll->registers) {
        free(hll);
        return NULL;
    }

    hll->precision = precision;
    hll->num_registers = num_registers;

    return hll;
}

void hll_free(hyperloglog_t *hll) {
    if (hll) {
        free(hll->registers);
        free(hll);
    }
}

static int count_leading_zeros(uint64_t x) {
    if (x == 0) return 64;
    int n = 0;
    if ((x & 0xFFFFFFFF00000000ULL) == 0) { n += 32; x <<= 32; }
    if ((x & 0xFFFF000000000000ULL) == 0) { n += 16; x <<= 16; }
    if ((x & 0xFF00000000000000ULL) == 0) { n += 8; x <<= 8; }
    if ((x & 0xF000000000000000ULL) == 0) { n += 4; x <<= 4; }
    if ((x & 0xC000000000000000ULL) == 0) { n += 2; x <<= 2; }
    if ((x & 0x8000000000000000ULL) == 0) { n += 1; }
    return n;
}

void hll_add(hyperloglog_t *hll, const void *data, size_t len) {
    if (!hll || !data) return;

    uint64_t hash = fnv1a_hash(data, len, 0);
    size_t idx = hash & (hll->num_registers - 1);
    uint64_t w = hash >> hll->precision;

    uint8_t rho = (w == 0) ? (64 - hll->precision) : (count_leading_zeros(w) + 1);

    if (rho > hll->registers[idx]) {
        hll->registers[idx] = rho;
    }
}

uint64_t hll_count(hyperloglog_t *hll) {
    if (!hll) return 0;

    double m = (double)hll->num_registers;
    double alpha;

    switch (hll->precision) {
        case 4: alpha = 0.673; break;
        case 5: alpha = 0.697; break;
        case 6: alpha = 0.709; break;
        default: alpha = 0.7213 / (1.0 + 1.079 / m); break;
    }

    double sum = 0.0;
    size_t zeros = 0;

    for (size_t i = 0; i < hll->num_registers; i++) {
        sum += pow(2.0, -(double)hll->registers[i]);
        if (hll->registers[i] == 0) {
            zeros++;
        }
    }

    double raw_estimate = alpha * m * m / sum;

    if (raw_estimate <= 2.5 * m && zeros > 0) {
        return (uint64_t)(m * log(m / (double)zeros));
    }

    return (uint64_t)raw_estimate;
}

/* ============================================================================
 * Quickselect
 * ============================================================================ */

static void swap_int64(int64_t *a, int64_t *b) {
    int64_t tmp = *a;
    *a = *b;
    *b = tmp;
}

static size_t partition(int64_t *arr, size_t len) {
    size_t pivot_idx = rand() % len;
    int64_t pivot = arr[pivot_idx];
    swap_int64(&arr[pivot_idx], &arr[len - 1]);

    size_t store = 0;
    for (size_t i = 0; i < len - 1; i++) {
        if (arr[i] < pivot) {
            swap_int64(&arr[i], &arr[store]);
            store++;
        }
    }
    swap_int64(&arr[store], &arr[len - 1]);
    return store;
}

int64_t quickselect(int64_t *arr, size_t len, size_t k) {
    if (!arr || k >= len) return 0;

    if (len == 1) return arr[0];

    size_t pivot_pos = partition(arr, len);

    if (pivot_pos == k) {
        return arr[pivot_pos];
    } else if (k < pivot_pos) {
        return quickselect(arr, pivot_pos, k);
    } else {
        return quickselect(arr + pivot_pos + 1, len - pivot_pos - 1, k - pivot_pos - 1);
    }
}

/* ============================================================================
 * Miller-Rabin
 * ============================================================================ */

static uint64_t mod_pow(uint64_t base, uint64_t exp, uint64_t mod) {
    uint64_t result = 1;
    base %= mod;

    while (exp > 0) {
        if (exp & 1) {
            __uint128_t tmp = (__uint128_t)result * base;
            result = tmp % mod;
        }
        exp >>= 1;
        __uint128_t tmp = (__uint128_t)base * base;
        base = tmp % mod;
    }

    return result;
}

bool miller_rabin(uint64_t n, size_t k) {
    if (n < 2) return false;
    if (n == 2 || n == 3) return true;
    if (n % 2 == 0) return false;

    uint64_t d = n - 1;
    uint64_t r = 0;
    while ((d & 1) == 0) {
        d >>= 1;
        r++;
    }

    for (size_t i = 0; i < k; i++) {
        uint64_t a = 2 + rand() % (n - 3);
        uint64_t x = mod_pow(a, d, n);

        if (x == 1 || x == n - 1) continue;

        bool composite = true;
        for (uint64_t j = 0; j < r - 1; j++) {
            x = mod_pow(x, 2, n);
            if (x == n - 1) {
                composite = false;
                break;
            }
        }

        if (composite) return false;
    }

    return true;
}

bool miller_rabin_deterministic(uint64_t n) {
    if (n < 2) return false;
    if (n == 2 || n == 3) return true;
    if (n % 2 == 0) return false;

    uint64_t witnesses[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37};
    size_t num_witnesses = sizeof(witnesses) / sizeof(witnesses[0]);

    uint64_t d = n - 1;
    uint64_t r = 0;
    while ((d & 1) == 0) {
        d >>= 1;
        r++;
    }

    for (size_t i = 0; i < num_witnesses; i++) {
        uint64_t a = witnesses[i];
        if (a >= n) continue;

        uint64_t x = mod_pow(a, d, n);

        if (x == 1 || x == n - 1) continue;

        bool composite = true;
        for (uint64_t j = 0; j < r - 1; j++) {
            x = mod_pow(x, 2, n);
            if (x == n - 1) {
                composite = false;
                break;
            }
        }

        if (composite) return false;
    }

    return true;
}

/* ============================================================================
 * Monte Carlo Pi
 * ============================================================================ */

double monte_carlo_pi(size_t samples) {
    size_t inside = 0;

    for (size_t i = 0; i < samples; i++) {
        double x = (double)rand() / RAND_MAX;
        double y = (double)rand() / RAND_MAX;
        if (x * x + y * y <= 1.0) {
            inside++;
        }
    }

    return 4.0 * (double)inside / (double)samples;
}
```

---

### 4.4 Solutions alternatives acceptees

#### Alternative 1 : HyperLogLog avec correction de biais

```rust
pub fn count_with_bias_correction(&self) -> u64 {
    let m = self.registers.len() as f64;
    let alpha = 0.7213 / (1.0 + 1.079 / m);

    let sum: f64 = self.registers.iter()
        .map(|&r| 2.0f64.powi(-(r as i32)))
        .sum();

    let raw = alpha * m * m / sum;

    // Bias correction pour grandes cardinalites
    if raw > (1u64 << 32) as f64 / 30.0 {
        -(((1u64 << 32) as f64) * (1.0 - raw / (1u64 << 32) as f64).ln()) as u64
    } else {
        raw as u64
    }
}
```

---

### 4.5 Solutions refusees (avec explications)

#### Refusee 1 : Bloom filter avec mauvais hashing

```rust
// REFUSE - Utilise le meme hash pour tous les indices
pub fn insert_wrong<T: Hash>(&mut self, item: &T) {
    let mut hasher = DefaultHasher::new();
    item.hash(&mut hasher);
    let hash = hasher.finish() as usize;

    for i in 0..self.hash_count {
        let idx = (hash + i) % self.size;  // Linear probing != independent hashes!
        self.bits[idx] = true;
    }
}
// Pourquoi c'est faux : Les hash functions doivent etre INDEPENDANTES
// Linear probing cree des clusters et augmente le FPR
```

#### Refusee 2 : Miller-Rabin sans gestion des grands nombres

```rust
// REFUSE - Overflow sur grands nombres
fn mod_pow_wrong(base: u64, exp: u64, m: u64) -> u64 {
    let mut result = 1u64;
    let mut base = base % m;
    let mut exp = exp;

    while exp > 0 {
        if exp % 2 == 1 {
            result = (result * base) % m;  // OVERFLOW si result * base > u64::MAX!
        }
        exp /= 2;
        base = (base * base) % m;
    }
    result
}
// Fix : Utiliser u128 pour les calculs intermediaires
```

---

### 4.6 Solution bonus de reference

```rust
/// Cuckoo Filter with deletions
pub struct CuckooFilter {
    buckets: Vec<Vec<u16>>,
    bucket_size: usize,
    max_kicks: usize,
}

impl CuckooFilter {
    pub fn new(num_buckets: usize, bucket_size: usize) -> Self {
        Self {
            buckets: vec![Vec::with_capacity(bucket_size); num_buckets],
            bucket_size,
            max_kicks: 500,
        }
    }

    fn fingerprint<T: Hash>(item: &T) -> u16 {
        let mut hasher = DefaultHasher::new();
        item.hash(&mut hasher);
        let h = hasher.finish();
        ((h >> 48) as u16).max(1)  // 0 is reserved
    }

    fn index1<T: Hash>(&self, item: &T) -> usize {
        let mut hasher = DefaultHasher::new();
        item.hash(&mut hasher);
        (hasher.finish() as usize) % self.buckets.len()
    }

    fn index2(&self, i1: usize, fp: u16) -> usize {
        let mut hasher = DefaultHasher::new();
        fp.hash(&mut hasher);
        let h = hasher.finish() as usize;
        (i1 ^ h) % self.buckets.len()
    }

    pub fn insert<T: Hash>(&mut self, item: &T) -> bool {
        let fp = Self::fingerprint(item);
        let i1 = self.index1(item);
        let i2 = self.index2(i1, fp);

        if self.buckets[i1].len() < self.bucket_size {
            self.buckets[i1].push(fp);
            return true;
        }

        if self.buckets[i2].len() < self.bucket_size {
            self.buckets[i2].push(fp);
            return true;
        }

        // Cuckoo eviction
        let mut rng = rand::thread_rng();
        let mut i = if rng.gen() { i1 } else { i2 };
        let mut fp = fp;

        for _ in 0..self.max_kicks {
            let j = rng.gen_range(0..self.buckets[i].len());
            std::mem::swap(&mut fp, &mut self.buckets[i][j]);
            i = self.index2(i, fp);

            if self.buckets[i].len() < self.bucket_size {
                self.buckets[i].push(fp);
                return true;
            }
        }

        false  // Filter is full
    }

    pub fn contains<T: Hash>(&self, item: &T) -> bool {
        let fp = Self::fingerprint(item);
        let i1 = self.index1(item);
        let i2 = self.index2(i1, fp);

        self.buckets[i1].contains(&fp) || self.buckets[i2].contains(&fp)
    }

    pub fn delete<T: Hash>(&mut self, item: &T) -> bool {
        let fp = Self::fingerprint(item);
        let i1 = self.index1(item);
        let i2 = self.index2(i1, fp);

        if let Some(pos) = self.buckets[i1].iter().position(|&x| x == fp) {
            self.buckets[i1].swap_remove(pos);
            return true;
        }

        if let Some(pos) = self.buckets[i2].iter().position(|&x| x == fp) {
            self.buckets[i2].swap_remove(pos);
            return true;
        }

        false
    }
}
```

---

### 4.9 spec.json

```json
{
  "name": "casino_royale_randomized",
  "language": "rust",
  "secondary_language": "c",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthese (Randomized Algorithms)",
  "tags": ["probabilistic", "randomized", "monte_carlo", "phase1", "crypto"],
  "passing_score": 70,

  "functions": [
    {
      "name": "BloomFilter",
      "type": "struct",
      "methods": ["new", "insert", "contains", "false_positive_rate"]
    },
    {
      "name": "CountMinSketch",
      "type": "struct",
      "methods": ["new", "add", "estimate", "merge"]
    },
    {
      "name": "HyperLogLog",
      "type": "struct",
      "methods": ["new", "add", "count", "merge"]
    },
    {
      "name": "MinHash",
      "type": "struct",
      "methods": ["new", "compute", "similarity"]
    },
    {
      "name": "SkipList",
      "type": "struct",
      "methods": ["new", "insert", "search", "remove", "len"]
    },
    {
      "name": "quickselect",
      "prototype": "pub fn quickselect<T: Ord + Clone>(arr: &mut [T], k: usize) -> Option<T>",
      "return_type": "Option<T>"
    },
    {
      "name": "reservoir_sample",
      "prototype": "pub fn reservoir_sample<T: Clone, I: Iterator<Item = T>>(stream: I, k: usize) -> Vec<T>",
      "return_type": "Vec<T>"
    },
    {
      "name": "karger_min_cut",
      "prototype": "pub fn karger_min_cut(graph: &Graph) -> usize",
      "return_type": "usize"
    },
    {
      "name": "miller_rabin",
      "prototype": "pub fn miller_rabin(n: u64, k: usize) -> bool",
      "return_type": "bool"
    },
    {
      "name": "monte_carlo_pi",
      "prototype": "pub fn monte_carlo_pi(samples: usize) -> f64",
      "return_type": "f64"
    }
  ],

  "driver": {
    "edge_cases": [
      {
        "name": "bloom_empty",
        "function": "BloomFilter::contains",
        "args": ["notinserted"],
        "expected": false,
        "is_trap": false
      },
      {
        "name": "bloom_inserted",
        "function": "BloomFilter::contains",
        "args": ["inserted"],
        "expected": true,
        "is_trap": false
      },
      {
        "name": "quickselect_k_zero",
        "function": "quickselect",
        "args": [[5, 2, 8, 1], 0],
        "expected": 1,
        "is_trap": true,
        "trap_explanation": "k=0 means minimum element"
      },
      {
        "name": "quickselect_k_oob",
        "function": "quickselect",
        "args": [[1, 2, 3], 10],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "k out of bounds must return None"
      },
      {
        "name": "miller_rabin_two",
        "function": "miller_rabin",
        "args": [2, 10],
        "expected": true,
        "is_trap": true,
        "trap_explanation": "2 is the only even prime"
      },
      {
        "name": "miller_rabin_carmichael",
        "function": "miller_rabin",
        "args": [561, 10],
        "expected": false,
        "is_trap": true,
        "trap_explanation": "561 is a Carmichael number - fools Fermat but not Miller-Rabin"
      },
      {
        "name": "hll_duplicates",
        "function": "HyperLogLog::count",
        "setup": "add same item 1000 times",
        "expected": 1,
        "is_trap": true,
        "trap_explanation": "Duplicates should not increase cardinality"
      },
      {
        "name": "reservoir_k_larger",
        "function": "reservoir_sample",
        "args": ["stream of 5 items", 10],
        "expected": "5 items",
        "is_trap": true,
        "trap_explanation": "If k > n, return all n items"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 10000,
      "generators": [
        {
          "type": "array_int",
          "param_index": 0,
          "params": {
            "min_len": 1,
            "max_len": 10000,
            "min_val": -1000000,
            "max_val": 1000000
          }
        },
        {
          "type": "int",
          "param_index": 1,
          "params": {
            "min": 0,
            "max": 100
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["rand", "thread_rng", "gen", "gen_range", "hash"],
    "forbidden_functions": ["sort", "sort_by"],
    "check_complexity": true,
    "check_space": true,
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes (minimum 5)

#### Mutant A (Boundary) : Bloom filter mauvais modulo

```rust
/* Mutant A (Boundary) : Modulo incorrect */
fn get_hash_wrong<T: Hash>(&self, item: &T, seed: usize) -> usize {
    let mut hasher = DefaultHasher::new();
    item.hash(&mut hasher);
    seed.hash(&mut hasher);
    (hasher.finish() as usize) & self.size  // ERREUR : & au lieu de %
}
// Pourquoi c'est faux : & (AND) ne fonctionne correctement que si size est une puissance de 2
// Pour size = 1000, & donne des resultats incorrects
// Fix : Utiliser % self.size
```

#### Mutant B (Safety) : Quickselect sans validation de k

```rust
/* Mutant B (Safety) : k non valide */
pub fn quickselect_mutant<T: Ord + Clone>(arr: &mut [T], k: usize) -> T {
    // MANQUE : if k >= arr.len() { return None; }
    let len = arr.len();
    let pivot_idx = rand::thread_rng().gen_range(0..len);
    // ... partition ...
    arr[k].clone()  // PANIC si k >= len
}
// Pourquoi c'est faux : Pas de verification des bornes
// Fix : Retourner Option<T> et verifier k < len
```

#### Mutant C (Resource) : Skip list fuite memoire

```rust
/* Mutant C (Resource) : Fuite memoire sur remove */
pub fn remove_mutant(&mut self, key: &K) -> Option<V> {
    // Trouve le noeud
    if let Some(node) = self.find_node(key) {
        let value = node.value.clone();
        // MANQUE : Liberer les pointeurs forward du noeud!
        self.len -= 1;
        Some(value)
    } else {
        None
    }
}
// Pourquoi c'est faux : Les niveaux du noeud ne sont pas liberes
// En Rust c'est moins grave (Drop), mais en C c'est une fuite memoire
```

#### Mutant D (Logic) : Miller-Rabin decomposition incorrecte

```rust
/* Mutant D (Logic) : Mauvaise extraction de d */
pub fn miller_rabin_mutant(n: u64, k: usize) -> bool {
    let mut d = n - 1;
    let mut r = 0u64;

    while d % 2 == 0 {
        d /= 2;
        // MANQUE : r += 1;  -- On ne compte pas les puissances de 2!
    }

    // ... reste de l'algo utilise r qui vaut toujours 0
}
// Pourquoi c'est faux : Sans incrementer r, la boucle de verification ne s'execute jamais
// Resultat : faux negatifs pour les vrais nombres premiers
```

#### Mutant E (Return) : Reservoir sampling non uniforme

```rust
/* Mutant E (Return) : Distribution non uniforme */
pub fn reservoir_sample_mutant<T: Clone, I: Iterator<Item = T>>(
    stream: I,
    k: usize,
) -> Vec<T> {
    let mut reservoir: Vec<T> = Vec::with_capacity(k);
    let mut rng = rand::thread_rng();

    for (i, item) in stream.enumerate() {
        if i < k {
            reservoir.push(item);
        } else {
            let j = rng.gen_range(0..k);  // ERREUR : devrait etre 0..=i
            reservoir[j] = item;  // Toujours remplace!
        }
    }

    reservoir
}
// Pourquoi c'est faux : gen_range(0..k) au lieu de gen_range(0..=i)
// Chaque nouvel element remplace toujours un element existant
// Les derniers elements ont probabilite 1 d'etre selectionnes = NON UNIFORME
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Structures probabilistes** : Bloom Filter, Count-Min Sketch, HyperLogLog, MinHash
2. **Algorithmes randomises** : Quickselect, Reservoir Sampling, Karger Min-Cut, Miller-Rabin
3. **Trade-offs espace/precision** : Comment gagner de la memoire en acceptant une erreur controlee
4. **Monte Carlo vs Las Vegas** : Deux philosophies d'algorithmes randomises
5. **Applications cryptographiques** : Tests de primalite, generation de cles

---

### 5.2 LDA -- Traduction litterale

```
FONCTION miller_rabin QUI RETOURNE UN BOOLEEN ET PREND EN PARAMETRES n QUI EST UN ENTIER NON SIGNE 64 BITS ET k QUI EST UN ENTIER NON SIGNE
DEBUT FONCTION
    SI n EST INFERIEUR A 2 ALORS
        RETOURNER FAUX
    FIN SI
    SI n EST EGAL A 2 OU n EST EGAL A 3 ALORS
        RETOURNER VRAI
    FIN SI
    SI n MODULO 2 EST EGAL A 0 ALORS
        RETOURNER FAUX
    FIN SI

    DECLARER d COMME ENTIER NON SIGNE 64 BITS
    DECLARER r COMME ENTIER NON SIGNE 64 BITS

    AFFECTER n MOINS 1 A d
    AFFECTER 0 A r

    TANT QUE d MODULO 2 EST EGAL A 0 FAIRE
        AFFECTER d DIVISE PAR 2 A d
        INCREMENTER r DE 1
    FIN TANT QUE

    POUR i ALLANT DE 0 A k MOINS 1 FAIRE
        DECLARER a COMME ENTIER ALEATOIRE ENTRE 2 ET n MOINS 2
        DECLARER x COMME LE RESULTAT DE a PUISSANCE d MODULO n

        SI x EST EGAL A 1 OU x EST EGAL A n MOINS 1 ALORS
            CONTINUER AVEC LA PROCHAINE ITERATION
        FIN SI

        POUR j ALLANT DE 0 A r MOINS 2 FAIRE
            AFFECTER x PUISSANCE 2 MODULO n A x
            SI x EST EGAL A n MOINS 1 ALORS
                ALLER A LA PROCHAINE ITERATION DE LA BOUCLE EXTERNE
            FIN SI
        FIN POUR

        RETOURNER FAUX
    FIN POUR

    RETOURNER VRAI
FIN FONCTION
```

---

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : Bloom Filter Insert/Contains
---
1. INITIALISER tableau de bits de taille m
   INITIALISER k fonctions de hachage independantes

2. INSERT(item) :
   a. POUR chaque hash function h_i (i = 1..k) :
      |-- CALCULER index = h_i(item) % m
      |-- METTRE bits[index] = 1
   b. RETOURNER

3. CONTAINS(item) :
   a. POUR chaque hash function h_i (i = 1..k) :
      |-- CALCULER index = h_i(item) % m
      |-- SI bits[index] == 0 :
      |     RETOURNER DEFINITELY_NOT_PRESENT
   b. RETOURNER PROBABLY_PRESENT

INVARIANT : False negatives IMPOSSIBLES
            False positives POSSIBLES avec probabilite ~(1 - e^(-kn/m))^k
```

---

### 5.2.3.1 Logique de Garde (Fail Fast)

```
FONCTION : Quickselect (arr, k)
---
INIT pivot_idx = random(0, len-1)

1. GARDE : k hors bornes
   |-- VERIFIER k >= len(arr)
   |     SI OUI -> RETOURNER Erreur/None
   |
   |-- Raison : Evite acces memoire invalide

2. GARDE : tableau vide
   |-- VERIFIER len(arr) == 0
   |     SI OUI -> RETOURNER Erreur/None
   |
   |-- Raison : Division par zero dans random

3. GARDE : tableau a un element
   |-- VERIFIER len(arr) == 1
   |     SI OUI -> RETOURNER arr[0] (k doit etre 0)
   |
   |-- Raison : Pas de partition necessaire

4. TRAITEMENT PRINCIPAL :
   |-- PARTITIONNER autour du pivot
   |-- SI pivot_pos == k : RETOURNER arr[k]
   |-- SI k < pivot_pos : RECURSION sur gauche
   |-- SI k > pivot_pos : RECURSION sur droite avec k ajuste

5. RETOURNER element trouve
```

---

### Diagramme Mermaid : Bloom Filter

```mermaid
graph TD
    A[Insert item] --> B{Calculate k hashes}
    B --> C[h1 = hash1(item) % m]
    B --> D[h2 = hash2(item) % m]
    B --> E[... hk = hashk(item) % m]
    C --> F[Set bits[h1] = 1]
    D --> G[Set bits[h2] = 1]
    E --> H[Set bits[hk] = 1]
    F --> I[Done]
    G --> I
    H --> I

    J[Contains item?] --> K{Calculate k hashes}
    K --> L[Check bits[h1]]
    K --> M[Check bits[h2]]
    K --> N[Check bits[hk]]
    L -- 0 --> O[DEFINITELY NOT IN SET]
    M -- 0 --> O
    N -- 0 --> O
    L -- 1 --> P{All 1?}
    M -- 1 --> P
    N -- 1 --> P
    P -- Yes --> Q[PROBABLY IN SET]
    P -- No --> O
```

---

### 5.3 Visualisation ASCII

#### Bloom Filter

```
Insert("Bond"), Insert("M"), Insert("Q")

Hash functions: h1, h2, h3 (k=3)

h1("Bond") = 2,  h2("Bond") = 7,  h3("Bond") = 12
h1("M")    = 4,  h2("M")    = 7,  h3("M")    = 15
h1("Q")    = 2,  h2("Q")    = 9,  h3("Q")    = 14

Bits array (m=16):
Index:  0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
      ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
Bits: │ 0 │ 0 │ 1 │ 0 │ 1 │ 0 │ 0 │ 1 │ 0 │ 1 │ 0 │ 0 │ 1 │ 0 │ 1 │ 1 │
      └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
            ↑       ↑           ↑       ↑           ↑       ↑   ↑
           Bond    M          Bond,M   Q         Bond      Q   M

contains("LeChiffre")?
h1("LeChiffre") = 3 -> bits[3] = 0 -> DEFINITELY NOT PRESENT
```

#### HyperLogLog

```
Precision p = 4 (16 registers)

Add("visitor1"): hash = 0x5A3B...
  Register index = hash & 0xF = 5
  Remaining bits = hash >> 4 = 0x5A3B >> 4
  Leading zeros + 1 = 3
  registers[5] = max(0, 3) = 3

Add("visitor2"): hash = 0x0012...
  Register index = 2
  Leading zeros + 1 = 12
  registers[2] = max(0, 12) = 12

Registers:
Index:   0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
       ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
Value: │ 0 │ 0 │12 │ 0 │ 0 │ 3 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │ 0 │
       └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

Estimate = alpha * m^2 / SUM(2^(-register[i]))
```

#### Miller-Rabin Witness Test

```
Test if n = 561 is prime (Carmichael number)

Step 1: Write n-1 = 560 = 2^4 * 35
        So r = 4, d = 35

Step 2: Pick random witness a = 2

Step 3: Compute x = 2^35 mod 561 = 263

Step 4: Check x != 1 and x != 560, so continue

Step 5: Square repeatedly:
        x = 263^2 mod 561 = 166
        x = 166^2 mod 561 = 67
        x = 67^2 mod 561 = 1  <-- Found 1 before finding n-1!

COMPOSITE! (True non-trivial square root of 1 found)

   ┌─────────────────────────────────────────────┐
   │  561 = 3 x 11 x 17 (Carmichael number)      │
   │  Fools Fermat test but NOT Miller-Rabin    │
   └─────────────────────────────────────────────┘
```

---

### 5.4 Les pieges en detail

| Piege | Description | Exemple | Solution |
|-------|-------------|---------|----------|
| **Modulo vs AND** | `% size` != `& size` | size=1000 | Utiliser `%` sauf si size = 2^k |
| **k >= n** | Quickselect k hors bornes | k=10, len=5 | Verifier k < len avant |
| **Overflow mod_pow** | a*b > u64::MAX | Grands premiers | Utiliser u128 intermediaire |
| **Miller-Rabin r=0** | Oublier d'incrementer r | n-1 = 2^0 * (n-1) | Toujours incrementer dans la boucle |
| **Reservoir j range** | gen_range(0..k) | Non uniforme | Doit etre gen_range(0..=i) |
| **HLL precision** | Mauvais masque | idx = hash & m | idx = hash & (m-1) |

---

### 5.5 Cours Complet : Algorithmes Randomises

#### 5.5.1 Monte Carlo vs Las Vegas

**Monte Carlo** : Le temps d'execution est fixe, mais le resultat peut etre incorrect avec une probabilite controlee.

Exemples :
- Miller-Rabin : Retourne "probably prime" ou "definitely composite"
- Monte Carlo Pi : Estime pi avec une marge d'erreur

**Las Vegas** : Le resultat est toujours correct, mais le temps d'execution est probabiliste.

Exemples :
- Quickselect : Trouve toujours le bon element, temps attendu O(n)
- Karger Min-Cut : Trouve toujours un cut, mais pas toujours le minimum

#### 5.5.2 Structures Probabilistes

| Structure | Operation | Complexite | Erreur |
|-----------|-----------|------------|--------|
| Bloom Filter | Insert/Query | O(k) | False positives |
| Count-Min Sketch | Add/Estimate | O(d) | Sur-estimation |
| HyperLogLog | Add/Count | O(1) / O(m) | ~1.04/sqrt(m) |
| MinHash | Compute/Similarity | O(n*k) | Jaccard approx |
| Skip List | Search/Insert | O(log n) expected | Aucune |

#### 5.5.3 Analyse Probabiliste

**Bloom Filter FPR** :
```
P(false positive) = (1 - e^(-kn/m))^k

Optimal k = (m/n) * ln(2)
```

**Miller-Rabin Error** :
```
P(error after k rounds) <= 4^(-k)

k = 10 => P < 10^(-6)
```

**HyperLogLog Error** :
```
Standard Error = 1.04 / sqrt(m)

m = 1024 => SE ~= 3.25%
```

---

### 5.6 Normes avec explications pedagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ X HORS NORME (securite)                                         │
├─────────────────────────────────────────────────────────────────┤
│ fn mod_pow(base: u64, exp: u64, m: u64) -> u64 {                │
│     result = (result * base) % m;  // OVERFLOW!                 │
│ }                                                                │
├─────────────────────────────────────────────────────────────────┤
│ V CONFORME                                                      │
├─────────────────────────────────────────────────────────────────┤
│ fn mod_pow(base: u64, exp: u64, m: u64) -> u64 {                │
│     let tmp = (result as u128) * (base as u128);                │
│     result = (tmp % (m as u128)) as u64;                        │
│ }                                                                │
├─────────────────────────────────────────────────────────────────┤
│ POURQUOI ?                                                      │
│                                                                 │
│ - a * b peut depasser u64::MAX pour les grands nombres          │
│ - Miller-Rabin teste des nombres de 64 bits                     │
│ - Utiliser u128 pour les calculs intermediaires                 │
│ - Ou utiliser Montgomery reduction pour la performance          │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.7 Simulation avec trace d'execution

#### Exemple : `reservoir_sample([10, 20, 30, 40, 50], k=2)`

```
┌───────┬─────────────┬───────────────┬──────────────────────────────────────────┐
│ Etape │ Element     │ Reservoir     │ Explication                              │
├───────┼─────────────┼───────────────┼──────────────────────────────────────────┤
│   0   │ 10          │ [10]          │ i < k, ajouter directement               │
├───────┼─────────────┼───────────────┼──────────────────────────────────────────┤
│   1   │ 20          │ [10, 20]      │ i < k, ajouter directement               │
├───────┼─────────────┼───────────────┼──────────────────────────────────────────┤
│   2   │ 30          │ [10, 20]      │ i=2, j=rand(0..2)=1, 1<k -> remplacer[1] │
│       │             │ -> [10, 30]   │ P(30 selected) = 2/3                     │
├───────┼─────────────┼───────────────┼──────────────────────────────────────────┤
│   3   │ 40          │ [10, 30]      │ i=3, j=rand(0..3)=3, 3>=k -> ignorer     │
│       │             │               │ P(40 selected) = 2/4 = 1/2               │
├───────┼─────────────┼───────────────┼──────────────────────────────────────────┤
│   4   │ 50          │ [10, 30]      │ i=4, j=rand(0..4)=0, 0<k -> remplacer[0] │
│       │             │ -> [50, 30]   │ P(50 selected) = 2/5                     │
└───────┴─────────────┴───────────────┴──────────────────────────────────────────┘

Resultat final : [50, 30]

Verification : Chaque element a probabilite k/n = 2/5 d'etre selectionne
```

---

### 5.8 Mnemotechniques

#### 🎰 MEME : "Casino Royale" -- Le Hasard comme Strategie

**"Bond. James Bond."**

Comme Bond au poker face a Le Chiffre, tu dois :
1. **Calculer les odds** avant d'agir (probabilites)
2. **Prendre des risques calcules** (Monte Carlo)
3. **Toujours avoir un plan B** (Las Vegas garantit le resultat)

```
             ┌─────────────────────────────────────┐
             │  "In poker, the odds define the     │
             │   player. In algorithms, they       │
             │   define the solution."             │
             │                    -- Q             │
             └─────────────────────────────────────┘
```

#### 🎯 MEME : "Le Chiffre" -- Hash Functions

Le Chiffre (= "The Number" en francais) manipule les chiffres pour cacher l'argent. Les hash functions font pareil : elles transforment les donnees en chiffres apparemment aleatoires.

```rust
// Le Chiffre encode les secrets
fn le_chiffre_hash(secret: &str) -> u64 {
    // Comme Le Chiffre, le hash est a sens unique
    // Tu ne peux pas retrouver l'original
}
```

#### 💀 MEME : "Carmichael Numbers" -- Les Espions Doubles

Les nombres de Carmichael (561, 1105, 1729...) sont des "espions doubles" : ils se font passer pour des premiers avec le test de Fermat, mais Miller-Rabin les demasque.

```
561 = 3 x 11 x 17  -- "Je suis premier!"
Fermat: "Ok, tu passes le test."
Miller-Rabin: "NON. Tu es un IMPOSTEUR!"

              ┌─────────────────────────────────┐
              │  "Trust, but verify."           │
              │  Miller-Rabin > Fermat          │
              └─────────────────────────────────┘
```

#### 🎲 MEME : "Reservoir Sampling" -- Le Barman de Casino

Imagine un barman qui doit choisir k clients au hasard pour une promo, mais il ne sait pas combien de clients viendront ce soir.

Solution : A chaque nouveau client i, il le selectionne avec probabilite k/i. C'est exactement Reservoir Sampling !

---

### 5.9 Applications pratiques

| Application | Algorithme | Utilisation |
|-------------|------------|-------------|
| **Google Safe Browsing** | Bloom Filter | Verifier si URL est malveillante |
| **Redis** | HyperLogLog | Compter visiteurs uniques |
| **Cassandra** | Bloom Filter | Eviter lectures disque inutiles |
| **Network Security** | Count-Min Sketch | Detection d'anomalies |
| **Document Search** | MinHash + LSH | Detection de duplicats/plagiat |
| **Cryptography** | Miller-Rabin | Generation de cles RSA |
| **Stream Processing** | Reservoir Sampling | Echantillonnage Big Data |
| **Network Analysis** | Karger Min-Cut | Partitionnement de graphes |

---

## SECTION 6 : PIEGES -- RECAPITULATIF

| # | Piege | Gravite | Algorithme | Comment l'eviter |
|---|-------|---------|------------|------------------|
| 1 | Modulo vs AND | 🔴 | Bloom Filter | Utiliser `%` sauf si taille = 2^k |
| 2 | k >= n | 🔴 | Quickselect | Verifier bornes avant partition |
| 3 | Overflow mod_pow | 🔴 | Miller-Rabin | Utiliser u128 intermediaire |
| 4 | r non incremente | 🔴 | Miller-Rabin | Boucle while d%2==0 : r++ |
| 5 | Mauvais range j | 🟡 | Reservoir | gen_range(0..=i) pas (0..k) |
| 6 | HLL leading zeros | 🟡 | HyperLogLog | Gerer le cas w == 0 |
| 7 | Skip list level 0 | 🟢 | Skip List | Niveau minimum = 1, pas 0 |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quelle est la principale difference entre Monte Carlo et Las Vegas ?**

- A) Monte Carlo est plus rapide
- B) Las Vegas garantit le resultat, Monte Carlo garantit le temps
- C) Monte Carlo est deterministe
- D) Las Vegas utilise moins de memoire
- E) Ils sont identiques

**Reponse : B**

---

### Question 2
**Un Bloom Filter peut-il avoir des faux negatifs ?**

- A) Oui, toujours
- B) Oui, rarement
- C) Non, jamais
- D) Seulement si la memoire est pleine
- E) Depend de la taille

**Reponse : C** (False negatives are IMPOSSIBLE)

---

### Question 3
**Quelle est la complexite attendue de Quickselect ?**

- A) O(1)
- B) O(log n)
- C) O(n)
- D) O(n log n)
- E) O(n^2)

**Reponse : C** (Temps attendu O(n), pire cas O(n^2))

---

### Question 4
**Pourquoi Miller-Rabin detecte les nombres de Carmichael mais pas Fermat ?**

- A) Miller-Rabin utilise plus de memoire
- B) Miller-Rabin verifie les racines carrees non triviales de 1
- C) Miller-Rabin est plus lent
- D) Fermat ne teste pas les nombres pairs
- E) C'est une coincidence

**Reponse : B**

---

### Question 5
**HyperLogLog estime la cardinalite avec quelle complexite memoire ?**

- A) O(1)
- B) O(log n)
- C) O(log log n)
- D) O(n)
- E) O(n log n)

**Reponse : C** (O(log log n) bits par registre, m registres)

---

## 📊 SECTION 8 : RECAPITULATIF

| Element | Detail |
|---------|--------|
| **Nom** | casino_royale_randomized |
| **Concept** | Randomized Algorithms (Monte Carlo, Las Vegas, Probabilistic DS) |
| **Difficulte** | ★★★★★★★☆☆☆ (7/10) |
| **Structures** | 5 (BloomFilter, CountMinSketch, HyperLogLog, MinHash, SkipList) |
| **Algorithmes** | 5 (Quickselect, ReservoirSampling, Karger, Miller-Rabin, MonteCarlo) |
| **Langages** | Rust Edition 2024 + C (c17) |
| **MEME** | Casino Royale (Bond vs Le Chiffre) |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.7.4-casino_royale_randomized",
    "generated_at": "2026-01-12 00:00:00",

    "metadata": {
      "exercise_id": "1.7.4",
      "exercise_name": "casino_royale_randomized",
      "module": "1.7",
      "module_name": "Advanced Algorithms",
      "concept": "synth",
      "concept_name": "Randomized Algorithms",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthese",
      "phase": 1,
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "languages": ["rust", "c"],
      "duration_minutes": 90,
      "xp_base": 200,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCE",
      "bonus_icon": "🔥",
      "complexity_time": "T2-3 O(n) to O(n log n)",
      "complexity_space": "S2 O(n) to O(n log n)",
      "prerequisites": ["probabilities", "hash_functions", "bit_manipulation", "data_structures"],
      "domains": ["Probas", "Algo", "Struct", "Crypto", "Compression"],
      "domains_bonus": ["Crypto", "Net"],
      "tags": ["probabilistic", "randomized", "monte_carlo", "las_vegas", "bloom_filter", "miller_rabin"],
      "meme_reference": "Casino Royale / Le Chiffre"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 Rust */",
      "references/ref_solution.c": "/* Section 4.3 C */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 */",
      "mutants/mutant_c_resource.rs": "/* Section 4.10 */",
      "mutants/mutant_d_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "references/ref_solution.c",
        "references/ref_solution_bonus.rs"
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
      "test_reference": "python3 hackbrain_engine_v22.py -s spec.json -f references/ref_solution.rs",
      "test_mutants": "python3 hackbrain_mutation_tester.py -r references/ref_solution.rs -s spec.json --validate"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 -- "L'excellence pedagogique ne se negocie pas"*
*Exercice genere automatiquement -- Compatible ENGINE v22.1 + Mutation Tester*
*"The name's Bond... James Bond." -- 007*
