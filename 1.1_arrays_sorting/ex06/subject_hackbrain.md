<thinking>
## Analyse du Concept
- Concept : Coordinate Compression (compression de coordonnées)
- Phase demandée : 1
- Adapté ? OUI — Technique intermédiaire-avancée pour Phase 1. Transforme des valeurs sparses en indices denses.

## Combo Base + Bonus
- Exercice de base : Implémenter Compressor 1D/2D + applications (count_smaller, LIS, RangeFrequency)
- Bonus : Segment Tree avec compression pour requêtes dynamiques
- Palier bonus : 🔥 Avancé
- Progression logique ? OUI — Base = compression + requêtes statiques, Bonus = structures dynamiques

## Prérequis & Difficulté
- Prérequis réels : Sorting, HashMap, Binary Search, prefix sums
- Difficulté estimée : 6/10
- Cohérent avec phase ? OUI — Proche du max Phase 1

## Aspect Fun/Culture
- Contexte choisi : Ant-Man / Marvel — Pym Particles
- MEME mnémotechnique : "I shrink therefore I am" / "What is this, a coordinate for ants?"
- Pourquoi c'est fun : Analogie parfaite entre compression (shrinking) et Pym Particles. Les milliards deviennent des petits indices, comme Scott Lang qui devient microscopique.

## Scénarios d'Échec (5 mutants)
1. Mutant A (Boundary) : Off-by-one dans binary_search pour compress
2. Mutant B (Safety) : Ne gère pas la valeur non trouvée dans decompress
3. Mutant C (Order) : Oublie de dédupliquer les valeurs triées
4. Mutant D (Logic) : Mauvais sens de tri pour count_smaller_to_right
5. Mutant E (Return) : LIS retourne la longueur du tableau dp au lieu du max

## Verdict
VALIDE — Analogie Ant-Man excellente, exercice technique couvrant compression + applications avancées
Note créativité : 96/100
</thinking>

---

# Exercice 1.1.6 : pym_particles

**Module :**
1.1 — Arrays & Sorting

**Concept :**
l — Coordinate Compression

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (compression 1D + 2D + applications Fenwick/DP)

**Langages :**
Rust Edition 2024 / C17

**Prérequis :**
- Binary Search
- HashMap / Sorting
- Prefix Sums (ex05)

**Domaines :**
Struct, Tri, DP

**Durée estimée :**
75 min

**XP Base :**
180

**Complexité :**
T2 O(n log n) construction × S2 O(n) stockage

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- **Rust :** `src/lib.rs`, `Cargo.toml`
- **C :** `pym_particles.c`, `pym_particles.h`

**Fonctions autorisées :**
- Rust : std (Vec, HashMap, BTreeMap, collections)
- C : malloc, free, qsort, bsearch, memset

**Fonctions interdites :**
- Bibliothèques externes de compression

---

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🐜 ANT-MAN — "What is this, a coordinate for ants?"**

Scott Lang a un problème. Les coordonnées du Quantum Realm sont astronomiques — des valeurs en milliards. Mais son équipement ne peut stocker que quelques milliers d'indices.

**La solution de Hank Pym :** Les **Pym Particles** de compression !

Au lieu de manipuler directement les coordonnées géantes, on les **shrink** vers de petits indices (0, 1, 2, ...) tout en gardant l'ordre relatif. Quand on a besoin de la vraie valeur, on **grow** back !

```
Coordonnées Quantum :  [1_000_000_000, 42, 999_999_999]
                              ↓ SHRINK (Pym Particles) ↓
Indices compressés :   [2, 0, 1]

    42 → 0        (le plus petit)
    999_999_999 → 1
    1_000_000_000 → 2    (le plus grand)
```

**Pourquoi ça marche ?**
On ne se soucie que de l'**ordre relatif**, pas des valeurs absolues. Si tu as 10^9 mais seulement 1000 valeurs uniques, tu peux tout mapper sur [0, 999].

**Ta mission :**

Créer le **Pym Particle Compressor** — un système de compression de coordonnées pour naviguer dans le Quantum Realm.

---

#### 1.2.2 Version Académique

**Coordinate Compression :**

Technique qui remplace un ensemble de valeurs larges et sparses par des indices consécutifs [0, k-1] où k est le nombre de valeurs uniques.

**Propriétés préservées :**
- L'ordre relatif est maintenu
- Égalité préservée (mêmes valeurs → même index)
- Bijection réversible (decompress possible)

**Algorithme :**
1. Collecter toutes les valeurs
2. Trier et dédupliquer → `sorted_unique`
3. Créer mapping valeur → index (HashMap ou binary search)
4. Pour compresser : lookup dans le mapping

**Applications :**
- Structures de données sur coordonnées (Fenwick Tree, Segment Tree)
- LIS en O(n log n) avec DP
- Requêtes de plage avec valeurs arbitraires

---

### 1.3 Prototypes

#### Rust

```rust
pub mod pym_particles {
    use std::collections::HashMap;
    use std::hash::Hash;

    /// 1D Coordinate Compressor — "The Pym Suit"
    pub struct PymCompressor<T: Ord + Clone + Hash> {
        sorted_unique: Vec<T>,
        value_to_index: HashMap<T, usize>,
    }

    impl<T: Ord + Clone + Hash> PymCompressor<T> {
        /// Construire le compresseur depuis un ensemble de coordonnées
        pub fn new(coordinates: &[T]) -> Self;

        /// Shrink une coordonnée vers son index compressé
        pub fn shrink(&self, value: &T) -> Option<usize>;

        /// Grow un index vers sa coordonnée originale
        pub fn grow(&self, index: usize) -> Option<&T>;

        /// Nombre de coordonnées uniques
        pub fn size(&self) -> usize;

        /// Shrink un tableau entier
        pub fn shrink_all(&self, values: &[T]) -> Vec<usize>;

        /// Lower bound: plus petit index >= value
        pub fn lower_bound(&self, value: &T) -> usize;

        /// Upper bound: plus petit index > value
        pub fn upper_bound(&self, value: &T) -> usize;
    }

    /// 2D Compressor — "Quantum Realm Navigator"
    pub struct QuantumNavigator {
        x_pym: PymCompressor<i64>,
        y_pym: PymCompressor<i64>,
    }

    impl QuantumNavigator {
        pub fn new(points: &[(i64, i64)]) -> Self;
        pub fn shrink_point(&self, point: (i64, i64)) -> (usize, usize);
        pub fn grow_point(&self, compressed: (usize, usize)) -> (i64, i64);
        pub fn grid_size(&self) -> (usize, usize);
    }

    // ═══════════════════════════════════════════════════════════
    // APPLICATIONS — "Using the Pym Suit"
    // ═══════════════════════════════════════════════════════════

    /// Count smaller elements to the right
    /// "How many smaller ants are behind me?"
    /// Uses compression + Fenwick Tree
    pub fn count_smaller_behind(arr: &[i32]) -> Vec<i32>;

    /// Longest Increasing Subsequence
    /// "The longest chain of growing ants"
    /// Uses compression + patience sort / DP with binary search
    pub fn longest_growth_chain(arr: &[i64]) -> usize;

    /// Distinct elements in range queries
    /// "How many unique ant species in this sector?"
    /// Uses compression + offline processing
    pub fn unique_species_in_sectors(
        arr: &[i32],
        queries: &[(usize, usize)],
    ) -> Vec<usize>;

    /// Range frequency query structure
    /// "How many times does ant #X appear in sector?"
    pub struct AntCensus {
        // Positions de chaque valeur compressée
        positions: Vec<Vec<usize>>,
        compressor: PymCompressor<i32>,
    }

    impl AntCensus {
        pub fn new(arr: &[i32]) -> Self;
        pub fn count_ant(&self, left: usize, right: usize, value: i32) -> usize;
    }

    /// Count points in rectangles
    /// "How many ants in each Quantum zone?"
    pub fn ants_in_zones(
        ant_positions: &[(i64, i64)],
        zones: &[(i64, i64, i64, i64)],  // (x1, y1, x2, y2)
    ) -> Vec<i64>;
}
```

#### C

```c
#ifndef PYM_PARTICLES_H
#define PYM_PARTICLES_H

#include <stddef.h>
#include <stdint.h>

// ═══════════════════════════════════════════════════════════════
// STRUCTURES
// ═══════════════════════════════════════════════════════════════

// 1D Compressor
typedef struct s_pym_compressor {
    int64_t *sorted_unique;
    size_t   unique_count;
} t_pym_compressor;

// 2D Compressor
typedef struct s_quantum_navigator {
    t_pym_compressor *x_pym;
    t_pym_compressor *y_pym;
} t_quantum_navigator;

// Range Frequency
typedef struct s_ant_census {
    size_t **positions;      // positions[value_idx] = array of positions
    size_t  *pos_counts;     // count per value
    t_pym_compressor *comp;
} t_ant_census;

// ═══════════════════════════════════════════════════════════════
// CONSTRUCTEURS & DESTRUCTEURS
// ═══════════════════════════════════════════════════════════════

t_pym_compressor   *pym_compressor_new(const int64_t *coords, size_t n);
void                pym_compressor_free(t_pym_compressor *comp);

t_quantum_navigator *quantum_navigator_new(const int64_t *xs, const int64_t *ys, size_t n);
void                 quantum_navigator_free(t_quantum_navigator *nav);

t_ant_census       *ant_census_new(const int *arr, size_t n);
void                ant_census_free(t_ant_census *census);

// ═══════════════════════════════════════════════════════════════
// OPÉRATIONS — "Pym Particle Manipulation"
// ═══════════════════════════════════════════════════════════════

// 1D Compression
ssize_t pym_shrink(const t_pym_compressor *comp, int64_t value);
int64_t pym_grow(const t_pym_compressor *comp, size_t index);
size_t  pym_size(const t_pym_compressor *comp);
size_t *pym_shrink_all(const t_pym_compressor *comp, const int64_t *values, size_t n);
size_t  pym_lower_bound(const t_pym_compressor *comp, int64_t value);
size_t  pym_upper_bound(const t_pym_compressor *comp, int64_t value);

// 2D Compression
void    quantum_shrink_point(const t_quantum_navigator *nav, int64_t x, int64_t y,
                            size_t *out_x, size_t *out_y);
void    quantum_grow_point(const t_quantum_navigator *nav, size_t cx, size_t cy,
                          int64_t *out_x, int64_t *out_y);

// Applications
int    *count_smaller_behind(const int *arr, size_t n);
size_t  longest_growth_chain(const int64_t *arr, size_t n);
size_t *unique_species_in_sectors(const int *arr, size_t n,
                                  const size_t *lefts, const size_t *rights,
                                  size_t q);
size_t  ant_census_count(const t_ant_census *census, size_t left, size_t right, int value);
int64_t *ants_in_zones(const int64_t *xs, const int64_t *ys, size_t n_points,
                       const int64_t *zones, size_t n_zones);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Faits Fascinants

**🌌 L'origine compétitive :**
La coordinate compression est une technique FONDAMENTALE en programmation compétitive. Sans elle, impossible de résoudre des problèmes avec coordonnées jusqu'à 10^18 en temps/mémoire raisonnable.

**📊 Usage en data science :**
Le "label encoding" en machine learning EST de la coordinate compression ! `["chat", "chien", "oiseau"]` → `[0, 1, 2]`.

**🎮 Dans les jeux :**
Les moteurs de jeu utilisent la compression de coordonnées pour les chunks (Minecraft), les sectors (Elite Dangerous), ou les zones de spawn.

### 2.2 Propriété Clé

```
            INVARIANT FONDAMENTAL

La compression préserve l'ORDRE RELATIF :

Si a < b dans l'original
→ compress(a) < compress(b) dans le compressé

C'est TOUT ce dont la plupart des algorithmes ont besoin !
```

### 2.5 Dans la Vraie Vie

| Métier | Utilisation |
|--------|-------------|
| **ML Engineer** | Label encoding, categorical features |
| **Game Developer** | Chunk coordinates, spatial hashing |
| **Competitive Programmer** | Segment trees, Fenwick trees on large ranges |
| **Database Engineer** | Dictionary encoding, columnar compression |
| **GIS Specialist** | Tile coordinates, quadtree keys |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
pym_particles.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 10 tests
test test_basic_compression ... ok
test test_large_values ... ok
test test_2d_compression ... ok
test test_count_smaller ... ok
test test_lis ... ok
test test_range_frequency ... ok
test test_unique_in_range ... ok
test test_bounds ... ok
test test_edge_empty ... ok
test test_edge_duplicates ... ok

test result: ok. 10 passed; 0 failed
```

---

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n log n) pour toutes les opérations dynamiques

**Space Complexity attendue :**
O(n)

**Domaines Bonus :**
`DP, Struct`

#### 3.1.1 Consigne Bonus

**🐜 ANT-MAN — "Quantum Realm Dynamics"**

Janet Van Dyne a besoin de tracker les mouvements dans le Quantum Realm en TEMPS RÉEL. Les coordonnées changent constamment !

**Ta mission bonus :**

1. **`DynamicPymCompressor`** — Supporte l'ajout de nouvelles valeurs en O(log n)

2. **`MergeSort + Compression`** — Compte les inversions avec compression

3. **`2D Range Queries Dynamiques`** — Segment Tree 2D avec compression paresseuse

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  1 ≤ n ≤ 10⁵                            │
│  -10¹⁸ ≤ valeurs ≤ 10¹⁸                 │
│  Temps : O(log n) par opération         │
│  Support des mises à jour dynamiques    │
└─────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
/// Dynamic Compressor avec insertion
pub struct DynamicPymCompressor<T: Ord + Clone + Hash> {
    tree: BTreeMap<T, usize>,  // Balanced BST
    reverse: Vec<T>,
}

impl<T: Ord + Clone + Hash> DynamicPymCompressor<T> {
    pub fn new() -> Self;
    pub fn insert(&mut self, value: T) -> usize;
    pub fn shrink(&self, value: &T) -> Option<usize>;
    pub fn grow(&self, index: usize) -> Option<&T>;
    pub fn size(&self) -> usize;
}

/// Count inversions avec compression
pub fn count_inversions(arr: &[i64]) -> i64;

/// 2D Segment Tree avec lazy compression
pub struct QuantumGrid {
    // Implementation avec compression différée
}

impl QuantumGrid {
    pub fn new(points: &[(i64, i64, i64)]) -> Self;  // (x, y, value)
    pub fn update(&mut self, x: i64, y: i64, delta: i64);
    pub fn query(&self, x1: i64, y1: i64, x2: i64, y2: i64) -> i64;
}
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Construction | O(n log n) statique | O(log n) par insertion |
| Nouvelles valeurs | Reconstruction | Insertion dynamique |
| Inversions | Non supporté | O(n log n) merge sort |
| 2D queries | Statique | Dynamique avec updates |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected Output | Points |
|------|-------|-----------------|--------|
| compress_basic | `[100, 200, 50, 200, 100]` | size=3, shrink(50)=0 | 3 |
| compress_large | `[10^9, 1, 5×10^8]` | `[2, 0, 1]` | 3 |
| compress_negative | `[-5, 0, 5, -10]` | size=4, order preserved | 3 |
| decompress_valid | shrink then grow | original value | 2 |
| 2d_compress | 3 points | correct (x,y) pairs | 4 |
| smaller_basic | `[5, 2, 6, 1]` | `[2, 1, 1, 0]` | 5 |
| smaller_duplicates | `[2, 2, 2]` | `[0, 0, 0]` | 3 |
| lis_basic | `[10,9,2,5,3,7,101,18]` | `4` | 5 |
| lis_all_same | `[7, 7, 7]` | `1` | 2 |
| lis_increasing | `[1, 2, 3, 4]` | `4` | 2 |
| unique_range | `[1,1,2,1,3]`, queries | `[3, 2, 3]` | 4 |
| frequency_basic | census queries | correct counts | 4 |
| bounds_lower | lower_bound tests | correct indices | 3 |
| bounds_upper | upper_bound tests | correct indices | 3 |
| edge_empty | `[]` | handle gracefully | 2 |
| edge_single | `[42]` | size=1, shrink(42)=0 | 2 |

### 4.2 main.rs de test

```rust
use pym_particles::*;

fn main() {
    println!("=== PYM PARTICLES TESTS ===\n");

    // Test 1: Basic Compression
    let coords = vec![100i64, 200, 50, 200, 100];
    let pym = PymCompressor::new(&coords);

    assert_eq!(pym.size(), 3);
    assert_eq!(pym.shrink(&50), Some(0));
    assert_eq!(pym.shrink(&100), Some(1));
    assert_eq!(pym.shrink(&200), Some(2));
    assert_eq!(pym.grow(1), Some(&100));
    println!("[OK] Basic 1D Compression");

    // Test 2: Large Values
    let large = vec![1_000_000_000i64, 1, 500_000_000];
    let pym = PymCompressor::new(&large);
    assert_eq!(pym.shrink_all(&large), vec![2, 0, 1]);
    println!("[OK] Large Values");

    // Test 3: 2D Compression
    let points = vec![(1000i64, 2000i64), (500, 3000), (1000, 1000)];
    let nav = QuantumNavigator::new(&points);

    assert_eq!(nav.shrink_point((500, 1000)), (0, 0));
    assert_eq!(nav.shrink_point((1000, 3000)), (1, 2));
    println!("[OK] 2D Compression");

    // Test 4: Count Smaller to Right
    let arr = vec![5, 2, 6, 1];
    assert_eq!(count_smaller_behind(&arr), vec![2, 1, 1, 0]);

    let arr = vec![2, 0, 1];
    assert_eq!(count_smaller_behind(&arr), vec![2, 0, 0]);
    println!("[OK] Count Smaller Behind");

    // Test 5: LIS
    let arr = vec![10i64, 9, 2, 5, 3, 7, 101, 18];
    assert_eq!(longest_growth_chain(&arr), 4);

    let arr = vec![0i64, 1, 0, 3, 2, 3];
    assert_eq!(longest_growth_chain(&arr), 4);
    println!("[OK] Longest Growth Chain (LIS)");

    // Test 6: Unique in Range
    let arr = vec![1, 1, 2, 1, 3];
    let queries = vec![(0, 4), (1, 3), (2, 4)];
    assert_eq!(unique_species_in_sectors(&arr, &queries), vec![3, 2, 3]);
    println!("[OK] Unique Species in Sectors");

    // Test 7: Range Frequency
    let arr = vec![12, 33, 4, 56, 22, 2, 34, 33, 22, 12, 34, 56];
    let census = AntCensus::new(&arr);
    assert_eq!(census.count_ant(1, 2, 4), 1);
    assert_eq!(census.count_ant(0, 11, 33), 2);
    println!("[OK] Ant Census");

    // Test 8: Bounds
    let coords = vec![10i64, 20, 30, 40, 50];
    let pym = PymCompressor::new(&coords);
    assert_eq!(pym.lower_bound(&25), 2);  // 30 is first >= 25
    assert_eq!(pym.upper_bound(&30), 3);  // 40 is first > 30
    println!("[OK] Lower/Upper Bounds");

    println!("\n=== ALL TESTS PASSED ===");
    println!("I am Ant-Man. And I just compressed your coordinates!");
}
```

### 4.3 Solution de référence (Rust)

```rust
pub mod pym_particles {
    use std::collections::HashMap;
    use std::hash::Hash;

    // ═══════════════════════════════════════════════════════════
    // PYM COMPRESSOR — 1D Coordinate Compression
    // ═══════════════════════════════════════════════════════════

    pub struct PymCompressor<T: Ord + Clone + Hash> {
        sorted_unique: Vec<T>,
        value_to_index: HashMap<T, usize>,
    }

    impl<T: Ord + Clone + Hash> PymCompressor<T> {
        pub fn new(coordinates: &[T]) -> Self {
            let mut sorted_unique: Vec<T> = coordinates.to_vec();
            sorted_unique.sort();
            sorted_unique.dedup();

            let value_to_index: HashMap<T, usize> = sorted_unique
                .iter()
                .enumerate()
                .map(|(i, v)| (v.clone(), i))
                .collect();

            PymCompressor { sorted_unique, value_to_index }
        }

        pub fn shrink(&self, value: &T) -> Option<usize> {
            self.value_to_index.get(value).copied()
        }

        pub fn grow(&self, index: usize) -> Option<&T> {
            self.sorted_unique.get(index)
        }

        pub fn size(&self) -> usize {
            self.sorted_unique.len()
        }

        pub fn shrink_all(&self, values: &[T]) -> Vec<usize> {
            values.iter()
                .filter_map(|v| self.shrink(v))
                .collect()
        }

        pub fn lower_bound(&self, value: &T) -> usize {
            self.sorted_unique.partition_point(|x| x < value)
        }

        pub fn upper_bound(&self, value: &T) -> usize {
            self.sorted_unique.partition_point(|x| x <= value)
        }
    }

    // ═══════════════════════════════════════════════════════════
    // QUANTUM NAVIGATOR — 2D Compression
    // ═══════════════════════════════════════════════════════════

    pub struct QuantumNavigator {
        x_pym: PymCompressor<i64>,
        y_pym: PymCompressor<i64>,
    }

    impl QuantumNavigator {
        pub fn new(points: &[(i64, i64)]) -> Self {
            let xs: Vec<i64> = points.iter().map(|p| p.0).collect();
            let ys: Vec<i64> = points.iter().map(|p| p.1).collect();

            QuantumNavigator {
                x_pym: PymCompressor::new(&xs),
                y_pym: PymCompressor::new(&ys),
            }
        }

        pub fn shrink_point(&self, point: (i64, i64)) -> (usize, usize) {
            (
                self.x_pym.shrink(&point.0).unwrap_or(0),
                self.y_pym.shrink(&point.1).unwrap_or(0),
            )
        }

        pub fn grow_point(&self, compressed: (usize, usize)) -> (i64, i64) {
            (
                *self.x_pym.grow(compressed.0).unwrap_or(&0),
                *self.y_pym.grow(compressed.1).unwrap_or(&0),
            )
        }

        pub fn grid_size(&self) -> (usize, usize) {
            (self.x_pym.size(), self.y_pym.size())
        }
    }

    // ═══════════════════════════════════════════════════════════
    // FENWICK TREE — Helper for count_smaller
    // ═══════════════════════════════════════════════════════════

    struct Fenwick {
        tree: Vec<i32>,
    }

    impl Fenwick {
        fn new(n: usize) -> Self {
            Fenwick { tree: vec![0; n + 1] }
        }

        fn update(&mut self, mut i: usize, delta: i32) {
            i += 1;
            while i < self.tree.len() {
                self.tree[i] += delta;
                i += i & i.wrapping_neg();
            }
        }

        fn prefix_sum(&self, mut i: usize) -> i32 {
            let mut sum = 0;
            i += 1;
            while i > 0 {
                sum += self.tree[i];
                i -= i & i.wrapping_neg();
            }
            sum
        }
    }

    // ═══════════════════════════════════════════════════════════
    // COUNT SMALLER BEHIND
    // ═══════════════════════════════════════════════════════════

    pub fn count_smaller_behind(arr: &[i32]) -> Vec<i32> {
        if arr.is_empty() {
            return vec![];
        }

        // Compress coordinates
        let arr_i64: Vec<i64> = arr.iter().map(|&x| x as i64).collect();
        let comp = PymCompressor::new(&arr_i64);

        let n = arr.len();
        let mut fenwick = Fenwick::new(comp.size());
        let mut result = vec![0i32; n];

        // Process from right to left
        for i in (0..n).rev() {
            let compressed = comp.shrink(&(arr[i] as i64)).unwrap();

            // Count elements smaller than current
            if compressed > 0 {
                result[i] = fenwick.prefix_sum(compressed - 1);
            }

            // Add current element
            fenwick.update(compressed, 1);
        }

        result
    }

    // ═══════════════════════════════════════════════════════════
    // LIS WITH COMPRESSION
    // ═══════════════════════════════════════════════════════════

    pub fn longest_growth_chain(arr: &[i64]) -> usize {
        if arr.is_empty() {
            return 0;
        }

        // Use patience sort approach (no compression needed, but binary search)
        let mut tails: Vec<i64> = vec![];

        for &x in arr {
            let pos = tails.partition_point(|&t| t < x);
            if pos == tails.len() {
                tails.push(x);
            } else {
                tails[pos] = x;
            }
        }

        tails.len()
    }

    // ═══════════════════════════════════════════════════════════
    // UNIQUE SPECIES IN SECTORS — Offline Mo's Algorithm style
    // ═══════════════════════════════════════════════════════════

    pub fn unique_species_in_sectors(
        arr: &[i32],
        queries: &[(usize, usize)],
    ) -> Vec<usize> {
        if arr.is_empty() || queries.is_empty() {
            return vec![0; queries.len()];
        }

        let n = arr.len();
        let q = queries.len();

        // Compress values
        let arr_i64: Vec<i64> = arr.iter().map(|&x| x as i64).collect();
        let comp = PymCompressor::new(&arr_i64);
        let compressed: Vec<usize> = arr.iter()
            .map(|&x| comp.shrink(&(x as i64)).unwrap())
            .collect();

        // Sort queries by right endpoint
        let mut sorted_queries: Vec<(usize, usize, usize)> = queries
            .iter()
            .enumerate()
            .map(|(i, &(l, r))| (l, r, i))
            .collect();
        sorted_queries.sort_by_key(|q| q.1);

        // Process with last occurrence tracking
        let mut last_occurrence = vec![n; comp.size()];  // Beyond array
        let mut fenwick = Fenwick::new(n);
        let mut results = vec![0usize; q];
        let mut current_right = 0;

        for (left, right, query_idx) in sorted_queries {
            // Extend to include all elements up to right
            while current_right <= right && current_right < n {
                let val = compressed[current_right];
                // Remove previous occurrence contribution
                if last_occurrence[val] < n {
                    fenwick.update(last_occurrence[val], -1);
                }
                // Add new occurrence
                fenwick.update(current_right, 1);
                last_occurrence[val] = current_right;
                current_right += 1;
            }

            // Query distinct count in [left, right]
            let total = fenwick.prefix_sum(right);
            let before = if left > 0 { fenwick.prefix_sum(left - 1) } else { 0 };
            results[query_idx] = (total - before) as usize;
        }

        results
    }

    // ═══════════════════════════════════════════════════════════
    // ANT CENSUS — Range Frequency
    // ═══════════════════════════════════════════════════════════

    pub struct AntCensus {
        positions: Vec<Vec<usize>>,
        compressor: PymCompressor<i64>,
    }

    impl AntCensus {
        pub fn new(arr: &[i32]) -> Self {
            let arr_i64: Vec<i64> = arr.iter().map(|&x| x as i64).collect();
            let compressor = PymCompressor::new(&arr_i64);

            let mut positions = vec![vec![]; compressor.size()];
            for (i, &val) in arr.iter().enumerate() {
                if let Some(compressed) = compressor.shrink(&(val as i64)) {
                    positions[compressed].push(i);
                }
            }

            AntCensus { positions, compressor }
        }

        pub fn count_ant(&self, left: usize, right: usize, value: i32) -> usize {
            if let Some(compressed) = self.compressor.shrink(&(value as i64)) {
                let pos = &self.positions[compressed];

                // Binary search for count in [left, right]
                let start = pos.partition_point(|&p| p < left);
                let end = pos.partition_point(|&p| p <= right);

                end - start
            } else {
                0
            }
        }
    }

    // ═══════════════════════════════════════════════════════════
    // ANTS IN ZONES — 2D Range Count
    // ═══════════════════════════════════════════════════════════

    pub fn ants_in_zones(
        ant_positions: &[(i64, i64)],
        zones: &[(i64, i64, i64, i64)],
    ) -> Vec<i64> {
        if ant_positions.is_empty() || zones.is_empty() {
            return vec![0; zones.len()];
        }

        // Simple O(n*q) approach for base implementation
        zones.iter()
            .map(|&(x1, y1, x2, y2)| {
                ant_positions.iter()
                    .filter(|&&(x, y)| x >= x1 && x <= x2 && y >= y1 && y <= y2)
                    .count() as i64
            })
            .collect()
    }
}
```

### 4.5 Solutions refusées (avec explications)

```rust
// ❌ REFUSÉ: Pas de déduplication
impl<T: Ord + Clone + Hash> PymCompressor<T> {
    pub fn new_bad(coordinates: &[T]) -> Self {
        let mut sorted: Vec<T> = coordinates.to_vec();
        sorted.sort();
        // BUG: Pas de dedup() !
        // Résultat: indices incorrects pour les doublons
    }
}

// ❌ REFUSÉ: shrink sans HashMap (O(n) au lieu de O(1))
pub fn shrink_bad(&self, value: &T) -> Option<usize> {
    // BUG: Linear search au lieu de HashMap lookup
    self.sorted_unique.iter().position(|x| x == value)
}
// Pourquoi c'est faux: O(n) par lookup au lieu de O(1)

// ❌ REFUSÉ: count_smaller de gauche à droite
pub fn count_smaller_behind_bad(arr: &[i32]) -> Vec<i32> {
    // BUG: Processe de gauche à droite
    // Compte les éléments à GAUCHE, pas à DROITE
}

// ❌ REFUSÉ: LIS retourne mauvaise valeur
pub fn longest_growth_chain_bad(arr: &[i64]) -> usize {
    let mut tails: Vec<i64> = vec![];
    for &x in arr {
        let pos = tails.partition_point(|&t| t < x);
        if pos == tails.len() {
            tails.push(x);
        } else {
            tails[pos] = x;
        }
    }
    // BUG: Retourne la dernière valeur au lieu de la longueur
    tails.last().copied().unwrap_or(0) as usize
}
```

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "pym_particles",
  "language": "rust",
  "version": "edition_2024",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (compression 1D + 2D + applications Fenwick/DP)",
  "tags": ["arrays", "coordinate_compression", "fenwick", "lis", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "pym_particles",
    "module": true,
    "components": [
      {
        "name": "PymCompressor",
        "type": "struct",
        "generic": "T: Ord + Clone + Hash",
        "methods": ["new", "shrink", "grow", "size", "shrink_all", "lower_bound", "upper_bound"]
      },
      {
        "name": "QuantumNavigator",
        "type": "struct",
        "methods": ["new", "shrink_point", "grow_point", "grid_size"]
      },
      {
        "name": "AntCensus",
        "type": "struct",
        "methods": ["new", "count_ant"]
      }
    ],
    "standalone_functions": [
      "count_smaller_behind",
      "longest_growth_chain",
      "unique_species_in_sectors",
      "ants_in_zones"
    ]
  },

  "driver": {
    "reference_file": "solutions/ref_pym_particles.rs",

    "edge_cases": [
      {
        "name": "compress_basic",
        "construct": ["PymCompressor", [100, 200, 50, 200, 100]],
        "tests": [
          {"call": ["size"], "expected": 3},
          {"call": ["shrink", 50], "expected": {"Some": 0}},
          {"call": ["shrink", 100], "expected": {"Some": 1}},
          {"call": ["shrink", 200], "expected": {"Some": 2}}
        ]
      },
      {
        "name": "compress_large",
        "construct": ["PymCompressor", [1000000000, 1, 500000000]],
        "call": ["shrink_all", [1000000000, 1, 500000000]],
        "expected": [2, 0, 1]
      },
      {
        "name": "smaller_basic",
        "function": "count_smaller_behind",
        "args": [[5, 2, 6, 1]],
        "expected": [2, 1, 1, 0]
      },
      {
        "name": "smaller_duplicates",
        "function": "count_smaller_behind",
        "args": [[2, 2, 2]],
        "expected": [0, 0, 0]
      },
      {
        "name": "lis_basic",
        "function": "longest_growth_chain",
        "args": [[10, 9, 2, 5, 3, 7, 101, 18]],
        "expected": 4
      },
      {
        "name": "lis_all_same",
        "function": "longest_growth_chain",
        "args": [[7, 7, 7]],
        "expected": 1
      },
      {
        "name": "unique_range",
        "function": "unique_species_in_sectors",
        "args": [[1, 1, 2, 1, 3], [[0, 4], [1, 3], [2, 4]]],
        "expected": [3, 2, 3]
      },
      {
        "name": "census_basic",
        "construct": ["AntCensus", [12, 33, 4, 56, 22, 2, 34, 33, 22, 12, 34, 56]],
        "tests": [
          {"call": ["count_ant", 1, 2, 4], "expected": 1},
          {"call": ["count_ant", 0, 11, 33], "expected": 2}
        ]
      },
      {
        "name": "bounds_test",
        "construct": ["PymCompressor", [10, 20, 30, 40, 50]],
        "tests": [
          {"call": ["lower_bound", 25], "expected": 2},
          {"call": ["upper_bound", 30], "expected": 3}
        ]
      },
      {
        "name": "empty_array",
        "function": "count_smaller_behind",
        "args": [[]],
        "expected": [],
        "is_trap": true,
        "trap_explanation": "Tableau vide - doit retourner vecteur vide"
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
            "min_val": -1000000000,
            "max_val": 1000000000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["malloc", "free", "qsort", "bsearch", "memset"],
    "forbidden_functions": [],
    "check_memory": true,
    "blocking": true
  },

  "bonus": {
    "tier": "ADVANCED",
    "icon": "🔥",
    "xp_multiplier": 3,
    "functions": [
      "DynamicPymCompressor",
      "count_inversions",
      "QuantumGrid"
    ]
  }
}
```

### 4.10 Solutions Mutantes

```rust
// ═══════════════════════════════════════════════════════════════
// MUTANT A (Boundary) : Off-by-one dans binary search
// ═══════════════════════════════════════════════════════════════

impl<T: Ord + Clone + Hash> PymCompressor<T> {
    pub fn lower_bound_bad(&self, value: &T) -> usize {
        // BUG: <= au lieu de <
        self.sorted_unique.partition_point(|x| x <= value)
    }
}
// Pourquoi c'est faux : Retourne l'index APRÈS la valeur, pas celui de la valeur
// Ce qui était pensé : Confusion entre lower_bound et upper_bound

// ═══════════════════════════════════════════════════════════════
// MUTANT B (Safety) : Pas de gestion du None dans shrink
// ═══════════════════════════════════════════════════════════════

impl<T: Ord + Clone + Hash> PymCompressor<T> {
    pub fn shrink_bad(&self, value: &T) -> usize {
        // BUG: unwrap() au lieu de Option
        *self.value_to_index.get(value).unwrap()
    }
}
// Pourquoi c'est faux : Panic si la valeur n'existe pas
// Ce qui était pensé : "Toutes les valeurs seront présentes"

// ═══════════════════════════════════════════════════════════════
// MUTANT C (Order) : Pas de déduplication
// ═══════════════════════════════════════════════════════════════

impl<T: Ord + Clone + Hash> PymCompressor<T> {
    pub fn new_bad(coordinates: &[T]) -> Self {
        let mut sorted_unique: Vec<T> = coordinates.to_vec();
        sorted_unique.sort();
        // BUG: Pas de dedup()

        let value_to_index: HashMap<T, usize> = sorted_unique
            .iter()
            .enumerate()
            .map(|(i, v)| (v.clone(), i))
            .collect();
        // HashMap écrase les doublons, mais sorted_unique est incorrect

        PymCompressor { sorted_unique, value_to_index }
    }
}
// Pourquoi c'est faux : sorted_unique contient des doublons, grow() retourne des valeurs incorrectes
// Ce qui était pensé : "Le HashMap gère les doublons"

// ═══════════════════════════════════════════════════════════════
// MUTANT D (Logic) : count_smaller dans le mauvais sens
// ═══════════════════════════════════════════════════════════════

pub fn count_smaller_behind_bad(arr: &[i32]) -> Vec<i32> {
    let arr_i64: Vec<i64> = arr.iter().map(|&x| x as i64).collect();
    let comp = PymCompressor::new(&arr_i64);

    let n = arr.len();
    let mut fenwick = Fenwick::new(comp.size());
    let mut result = vec![0i32; n];

    // BUG: Process from left to right (counts smaller to LEFT)
    for i in 0..n {
        let compressed = comp.shrink(&(arr[i] as i64)).unwrap();
        if compressed > 0 {
            result[i] = fenwick.prefix_sum(compressed - 1);
        }
        fenwick.update(compressed, 1);
    }

    result
}
// Pourquoi c'est faux : Compte les éléments à gauche, pas à droite
// Ce qui était pensé : "L'ordre d'itération n'importe pas"

// ═══════════════════════════════════════════════════════════════
// MUTANT E (Return) : LIS retourne le dernier élément
// ═══════════════════════════════════════════════════════════════

pub fn longest_growth_chain_bad(arr: &[i64]) -> usize {
    if arr.is_empty() {
        return 0;
    }

    let mut tails: Vec<i64> = vec![];

    for &x in arr {
        let pos = tails.partition_point(|&t| t < x);
        if pos == tails.len() {
            tails.push(x);
        } else {
            tails[pos] = x;
        }
    }

    // BUG: Retourne la dernière valeur au lieu de la longueur
    *tails.last().unwrap() as usize
}
// Pourquoi c'est faux : Retourne une valeur du tableau, pas la longueur LIS
// Ce qui était pensé : "tails contient la LIS donc le dernier = longueur"
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

| Concept | Description | Pourquoi c'est important |
|---------|-------------|-------------------------|
| **Coordinate Compression** | Mapper valeurs sparses → indices denses | Fondamental en competitive programming |
| **Préservation d'ordre** | L'ordre relatif suffit pour beaucoup d'algos | Insight clé pour optimisation |
| **Fenwick + Compression** | Combiner structures avancées | Pattern puissant |
| **LIS O(n log n)** | Patience sort avec binary search | Algorithme classique optimisé |
| **Range Queries** | Requêtes sur intervalles | Omniprésent en algorithmique |

### 5.2 LDA — Traduction Littérale

**PymCompressor::new**
```
FONCTION new QUI RETOURNE UN PymCompressor ET PREND EN PARAMÈTRE coordinates QUI EST UN SLICE
DÉBUT FONCTION
    DÉCLARER sorted_unique COMME VECTEUR COPIE DE coordinates
    TRIER sorted_unique EN ORDRE CROISSANT
    SUPPRIMER LES DOUBLONS CONSÉCUTIFS DE sorted_unique

    DÉCLARER value_to_index COMME HASHMAP VIDE
    POUR CHAQUE élément v À LA POSITION i DANS sorted_unique FAIRE
        INSÉRER (v, i) DANS value_to_index
    FIN POUR

    RETOURNER PymCompressor AVEC sorted_unique ET value_to_index
FIN FONCTION
```

**shrink**
```
FONCTION shrink QUI RETOURNE UN OPTION<INDEX> ET PREND EN PARAMÈTRE value
DÉBUT FONCTION
    RETOURNER LE RÉSULTAT DE LA RECHERCHE DE value DANS value_to_index
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Coordinate Compression
---
1. CONSTRUCTION (une seule fois) :
   a. COPIER toutes les valeurs
   b. TRIER par ordre croissant
   c. DÉDUPLIQUER (supprimer les doublons consécutifs)
   d. CRÉER HashMap : valeur → index

2. SHRINK (compression) :
   a. CHERCHER la valeur dans HashMap
   b. RETOURNER l'index (ou None si absent)

3. GROW (décompression) :
   a. ACCÉDER à sorted_unique[index]
   b. RETOURNER la valeur originale

4. COMPLEXITÉ :
   - Construction : O(n log n)
   - Shrink : O(1) avec HashMap
   - Grow : O(1) accès tableau
```

### 5.2.3 Représentation Algorithmique (Logique de Garde)

```
FONCTION : count_smaller_behind(arr)
---
INIT result = tableau de 0

1. COMPRESSER toutes les valeurs :
   |
   |-- CRÉER PymCompressor avec arr

2. CRÉER Fenwick Tree de taille compressed_size

3. ITÉRER DE DROITE À GAUCHE (i = n-1 down to 0) :
   |
   |-- compressed = shrink(arr[i])
   |
   |-- SI compressed > 0 :
   |     result[i] = fenwick.prefix_sum(compressed - 1)
   |
   |-- fenwick.update(compressed, +1)

4. RETOURNER result
```

### 5.3 Visualisation ASCII

**Coordinate Compression:**
```
Valeurs originales (sparses) :

    [1_000_000_000]  [42]  [999_999_999]  [42]  [500_000_000]
           ↓          ↓          ↓          ↓          ↓

Étape 1 - Tri + Dédup :
    sorted = [42, 500_000_000, 999_999_999, 1_000_000_000]

Étape 2 - Mapping :
    42 → 0
    500_000_000 → 1
    999_999_999 → 2
    1_000_000_000 → 3

Résultat compressé :
    [3]  [0]  [2]  [0]  [1]
```

**Count Smaller Behind avec Fenwick:**
```
arr = [5, 2, 6, 1]

Étape 1: Compression
    sorted = [1, 2, 5, 6]
    mapping: 1→0, 2→1, 5→2, 6→3

Étape 2: Traitement droite → gauche

    i=3, val=1, compressed=0
    fenwick = [0,0,0,0]
    smaller = 0 (pas d'élément à droite)
    fenwick[0] += 1 → [1,0,0,0]
    result[3] = 0

    i=2, val=6, compressed=3
    fenwick = [1,0,0,0]
    smaller = prefix(2) = 1  (le "1" est à droite et plus petit)
    fenwick[3] += 1 → [1,0,0,1]
    result[2] = 1

    i=1, val=2, compressed=1
    fenwick = [1,0,0,1]
    smaller = prefix(0) = 1  (le "1" est plus petit)
    fenwick[1] += 1 → [1,1,0,1]
    result[1] = 1

    i=0, val=5, compressed=2
    fenwick = [1,1,0,1]
    smaller = prefix(1) = 2  ("1" et "2" sont plus petits)
    fenwick[2] += 1 → [1,1,1,1]
    result[0] = 2

Résultat: [2, 1, 1, 0] ✓
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **Oublier dedup()** | Doublons créent des indices incorrects | Toujours sort PUIS dedup |
| **Direction count_smaller** | Gauche vs droite | Itérer de droite à gauche |
| **lower vs upper bound** | Confusion sur l'inclusivité | `<` = lower, `<=` = upper |
| **LIS longueur vs valeur** | Retourner len() pas last() | C'est la LONGUEUR du tableau tails |
| **Valeur non trouvée** | Panic sur unwrap() | Utiliser Option ou vérifier |

### 5.5 Cours Complet

#### 5.5.1 Pourquoi la Coordinate Compression ?

Imagine que tu dois créer un Segment Tree pour des valeurs allant de -10^18 à 10^18. C'est impossible en mémoire !

Mais si tu as seulement 1000 valeurs uniques, tu peux les mapper sur [0, 999] et créer un arbre de taille 1000.

**Le principe clé :**
La plupart des algorithmes ne se soucient que de l'**ordre relatif** des valeurs, pas de leurs valeurs absolues.

#### 5.5.2 Algorithme de Compression

```rust
fn compress(values: &[i64]) -> (Vec<usize>, Vec<i64>) {
    // 1. Copier et trier
    let mut sorted = values.to_vec();
    sorted.sort();

    // 2. Dédupliquer
    sorted.dedup();

    // 3. Créer le mapping inversé
    let map: HashMap<i64, usize> = sorted.iter()
        .enumerate()
        .map(|(i, &v)| (v, i))
        .collect();

    // 4. Compresser le tableau original
    let compressed = values.iter()
        .map(|v| map[v])
        .collect();

    (compressed, sorted)  // sorted sert à décompresser
}
```

#### 5.5.3 Count Smaller to Right

Ce problème classique devient O(n log n) avec compression + Fenwick :

1. **Compresser** toutes les valeurs vers [0, k-1]
2. **Initialiser** un Fenwick Tree de taille k
3. **Itérer de droite à gauche** :
   - Pour chaque élément, query le nombre d'éléments plus petits déjà vus
   - Ajouter l'élément courant au Fenwick Tree

Pourquoi de droite à gauche ? Parce qu'on veut compter les éléments **à droite**, qui sont donc traités **avant** dans notre itération inversée.

#### 5.5.4 LIS (Longest Increasing Subsequence)

L'approche "patience sort" donne O(n log n) :

```rust
fn lis(arr: &[i64]) -> usize {
    let mut tails = vec![];

    for &x in arr {
        // Trouver où insérer x
        let pos = tails.partition_point(|&t| t < x);

        if pos == tails.len() {
            tails.push(x);  // Nouveau plus grand
        } else {
            tails[pos] = x;  // Remplacer
        }
    }

    tails.len()  // La longueur de tails = longueur LIS
}
```

**Intuition :** `tails[i]` contient le plus petit élément de fin possible pour une sous-séquence croissante de longueur `i+1`.

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (fonctionne mais inefficace)                      │
├─────────────────────────────────────────────────────────────────┤
│ // Linear search pour chaque compression                        │
│ sorted.iter().position(|x| x == value)  // O(n)                 │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ // HashMap lookup O(1)                                          │
│ value_to_index.get(value)                                       │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ • 10^5 compressions × O(n) search = O(n²) → timeout             │
│ • HashMap lookup = O(1) amorti                                   │
│ • Le surcoût mémoire est négligeable vs le gain en temps        │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**count_smaller_behind([5, 2, 6, 1])**

```
┌───────┬──────────────────────────────────────────┬─────────────┬───────────────────────────┐
│ Étape │ Instruction                              │ Fenwick     │ result                    │
├───────┼──────────────────────────────────────────┼─────────────┼───────────────────────────┤
│   1   │ Compression: 1→0, 2→1, 5→2, 6→3         │             │                           │
├───────┼──────────────────────────────────────────┼─────────────┼───────────────────────────┤
│   2   │ i=3, val=1, comp=0                       │ [0,0,0,0]   │ [?,?,?,0]                │
│       │ query(comp-1) = query(-1) = 0            │             │                           │
│       │ update(0, +1)                            │ [1,0,0,0]   │                           │
├───────┼──────────────────────────────────────────┼─────────────┼───────────────────────────┤
│   3   │ i=2, val=6, comp=3                       │ [1,0,0,0]   │ [?,?,1,0]                │
│       │ query(2) = 1 (le "1" à droite)           │             │                           │
│       │ update(3, +1)                            │ [1,0,0,1]   │                           │
├───────┼──────────────────────────────────────────┼─────────────┼───────────────────────────┤
│   4   │ i=1, val=2, comp=1                       │ [1,0,0,1]   │ [?,1,1,0]                │
│       │ query(0) = 1                             │             │                           │
│       │ update(1, +1)                            │ [1,1,0,1]   │                           │
├───────┼──────────────────────────────────────────┼─────────────┼───────────────────────────┤
│   5   │ i=0, val=5, comp=2                       │ [1,1,0,1]   │ [2,1,1,0]                │
│       │ query(1) = 2 ("1" et "2")                │             │                           │
│       │ update(2, +1)                            │ [1,1,1,1]   │                           │
└───────┴──────────────────────────────────────────┴─────────────┴───────────────────────────┘
```

### 5.8 Mnémotechniques

#### 🐜 MEME : "What is this, a coordinate for ants?" — Coordinate Compression

Comme Scott Lang qui rétrécit avec les Pym Particles, tes coordonnées géantes deviennent minuscules.

```rust
// Avant: Coordonnées GÉANTES
let giant = [1_000_000_000, 42, 999_999_999];

// Après: Coordonnées pour fourmis 🐜
let ant_sized = [2, 0, 1];
```

**"I shrink, therefore I fit in memory!"**

#### 🦸 MEME : "Avengers, Assemble!" — Sorted + Dedup

Comme les Avengers qui se rassemblent et éliminent les doublons (il n'y a qu'un seul Iron Man), la compression trie et déduplique.

```rust
// Les héros arrivent dans le désordre avec des doublons
let heroes = ["Thor", "Iron Man", "Thor", "Hulk"];

// Après assemblage
let unique_heroes = ["Hulk", "Iron Man", "Thor"];
//                      0         1         2
```

#### 💥 MEME : "Thanos était un problème O(n²)" — Optimisation

Thanos voulait réduire la population de moitié avec une solution O(n) (le snap).

La coordinate compression, c'est pareil : transformer un problème O(n²) en O(n log n).

### 5.9 Applications pratiques

| Domaine | Application |
|---------|-------------|
| **ML** | Label encoding des features catégorielles |
| **Databases** | Dictionary encoding pour compression colonnes |
| **Gaming** | Chunk coordinates, spatial indexing |
| **Competitive** | Fenwick/Segment trees sur grands ranges |
| **GIS** | Tile coordinates, Z-order curves |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Détection |
|---|-------|--------|-----------|
| 1 | Pas de dedup() | Indices incorrects pour doublons | Test avec valeurs répétées |
| 2 | shrink retourne usize au lieu de Option | Panic sur valeur inconnue | Test avec valeur absente |
| 3 | count_smaller gauche→droite | Compte à gauche pas à droite | Test [5,2,6,1] |
| 4 | LIS retourne tails.last() | Retourne valeur pas longueur | Test séquences variées |
| 5 | Confusion lower/upper bound | Off-by-one | Tests exhaustifs sur bounds |

---

## 📝 SECTION 7 : QCM

### Q1. Compression de [100, 50, 100, 200]
Quelle est la taille après compression ?

A) 2
B) 3
C) 4
D) 5

**Réponse : B**

Valeurs uniques triées : [50, 100, 200] → 3 éléments

---

### Q2. shrink(75) sur [50, 100, 200]
Que retourne shrink(75) ?

A) 0
B) 1
C) None / erreur
D) 2

**Réponse : C**

75 n'existe pas dans les valeurs compressées

---

### Q3. count_smaller_behind direction
Pourquoi itère-t-on de droite à gauche ?

A) C'est plus rapide
B) Pour compter les éléments à droite
C) Pour éviter les doublons
D) Par convention

**Réponse : B**

On veut compter les éléments à DROITE, donc on les traite AVANT (en itérant depuis la droite)

---

### Q4. LIS de [3, 1, 2, 1, 4]
Quelle est la longueur de la LIS ?

A) 2
B) 3
C) 4
D) 5

**Réponse : B**

LIS = [1, 2, 4] → longueur 3

---

### Q5. lower_bound vs upper_bound
Pour sorted = [10, 20, 30], que retourne lower_bound(20) ?

A) 0
B) 1
C) 2
D) 3

**Réponse : B**

lower_bound(20) = premier index ≥ 20 = index de 20 = 1

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.1.6 - pym_particles |
| **Difficulté** | 6/10 (★★★★★★☆☆☆☆) |
| **Structures** | 3 (PymCompressor, QuantumNavigator, AntCensus) |
| **Fonctions** | 4 applications + bounds |
| **Complexité Construction** | O(n log n) |
| **Complexité Query** | O(1) shrink/grow, O(log n) bounds |
| **Bonus** | 🔥 Avancé (×3 XP) |
| **Points totaux** | 100 base + 50 bonus |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.1.6-pym_particles",
    "generated_at": "2026-01-11T10:45:00Z",

    "metadata": {
      "exercise_id": "1.1.6",
      "exercise_name": "pym_particles",
      "module": "1.1",
      "module_name": "Arrays & Sorting",
      "concept": "l",
      "concept_name": "Coordinate Compression",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse (compression 1D + 2D + Fenwick + DP)",
      "phase": 1,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust",
      "language_version": "edition_2024",
      "language_alt": "c17",
      "duration_minutes": 75,
      "xp_base": 180,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "ADVANCED",
      "bonus_icon": "🔥",
      "complexity_time": "T2 O(n log n)",
      "complexity_space": "S2 O(n)",
      "prerequisites": ["binary_search", "hashmap", "prefix_sums"],
      "domains": ["Struct", "Tri", "DP"],
      "domains_bonus": ["DP", "Struct"],
      "tags": ["coordinate_compression", "fenwick", "lis", "range_query"],
      "meme_reference": "Ant-Man - What is this, a coordinate for ants?"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_pym_particles.rs": "/* Section 4.3 */",
      "references/ref_pym_particles.c": "/* C implementation */",
      "references/ref_bonus.rs": "/* Section 4.6 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 */",
      "mutants/mutant_c_order.rs": "/* Section 4.10 */",
      "mutants/mutant_d_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
      "tests/main.rs": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_pym_particles.rs",
        "references/ref_bonus.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_safety.rs",
        "mutants/mutant_c_order.rs",
        "mutants/mutant_d_logic.rs",
        "mutants/mutant_e_return.rs"
      ]
    },

    "commands": {
      "validate_spec": "hackbrain-engine validate spec.json",
      "test_reference": "hackbrain-engine test -s spec.json -f references/ref_pym_particles.rs",
      "test_mutants": "hackbrain-mutation-tester -r references/ref_pym_particles.rs -s spec.json --validate"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "What is this, a coordinate for ants?"*
*L'excellence pédagogique ne se négocie pas*
