<thinking>
## Analyse du Concept
- Concept : Mini-Project - Complete Sorting & Searching Library (Synthèse de 110 concepts)
- Phase demandée : 1 (Intermédiaire)
- Adapté ? OUI - C'est la synthèse finale du module, difficulté maximale Phase 1

## Combo Base + Bonus
- Exercice de base : Implémenter bibliothèque complète avec 12+ sorts, 8+ searches, techniques, arena
- Bonus : Benchmarking automatisé, visualisation, documentation générée, CI/CD ready
- Palier bonus : 💀 Expert (production-quality library)
- Progression logique ? OUI - base = bibliothèque fonctionnelle, bonus = production-ready

## Prérequis & Difficulté
- Prérequis réels : TOUS les exercices précédents du module 1.1 (ex00-ex09)
- Difficulté estimée : 7/10
- Cohérent avec phase ? OUI (Tier 3 = max Phase 1 + 2)

## Aspect Fun/Culture
- Contexte choisi : Fullmetal Alchemist: Brotherhood
- MEME mnémotechnique : "Equivalent Exchange" = Trade-offs algorithmiques
- Pourquoi c'est fun :
  * L'alchimie = transmutation des données (tri)
  * La Pierre Philosophale = la bibliothèque parfaite qu'on cherche à créer
  * Le cercle de transmutation = l'API complète et cohérente
  * Edward & Alphonse = les deux types d'algorithmes (comparison vs non-comparison)
  * "In order to gain something, you must sacrifice something of equal value"
    = Time-Space trade-offs (O(n log n) time vs O(n) space)
  * Les Homunculi = Les bugs à éliminer
  * The Truth = La complexité théorique

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Stability) : merge_sort qui perd la stabilité
2. Mutant B (Edge) : quick_sort qui overflow sur tableau vide
3. Mutant C (Integration) : binary_search qui utilise mauvais comparateur
4. Mutant D (Memory) : arena.reset() qui ne reset pas correctement
5. Mutant E (Benchmark) : benchmark qui ne fait pas de warmup

## Verdict
VALIDE - Excellent projet de synthèse avec référence FMA parfaite
Note qualité : 98/100
</thinking>

---

# Exercice 1.1.10 : transmutation_circle

**Module :**
1.1.10 — Complete Sorting & Searching Library (Mini-Project)

**Concept :**
ALL — Synthèse de 110 concepts du Module 1.1

**Difficulté :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (TOUS les concepts du module 1.1)

**Langage :**
Rust Edition 2024 / C17

**Prérequis :**
- TOUS les exercices précédents (ex00-ex09)
- GenericVec, Sorting Suite, Binary Search, Two Pointers
- Sliding Window, Prefix Sums, Coordinate Compression
- Ternary Search, Complexity Analysis, Arena Allocator

**Domaines :**
Tri, Struct, Mem, MD, CPU

**Durée estimée :**
180 min (3h)

**XP Base :**
500

**Complexité :**
Variable (selon algorithme) × Variable (selon structure)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
transmutation_circle/
├── Cargo.toml
├── README.md
├── src/
│   ├── lib.rs           # Public API
│   ├── vector.rs        # GenericVec
│   ├── sorting/
│   │   ├── mod.rs
│   │   ├── comparison.rs
│   │   ├── non_comparison.rs
│   │   └── hybrid.rs
│   ├── searching/
│   │   ├── mod.rs
│   │   ├── binary.rs
│   │   └── ternary.rs
│   ├── techniques/
│   │   ├── mod.rs
│   │   ├── two_pointers.rs
│   │   ├── sliding_window.rs
│   │   ├── prefix_sum.rs
│   │   └── compression.rs
│   ├── memory/
│   │   └── arena.rs
│   └── analysis/
│       ├── complexity.rs
│       └── benchmark.rs
├── benches/
│   └── alchemy_bench.rs
└── tests/
    ├── sorting_tests.rs
    └── integration_tests.rs
```

**Fonctions autorisées :**
- Toutes les fonctions de la bibliothèque standard Rust/C
- Pas de crates externes pour les algorithmes eux-mêmes
- Criterion autorisé pour les benchmarks

**Fonctions interdites :**
- `slice.sort()` ou équivalent (doit implémenter soi-même)
- Bibliothèques de tri externes

### 1.2 Consigne

**🎮 FULLMETAL ALCHEMIST: BROTHERHOOD — Le Cercle de Transmutation Ultime**

*"Humankind cannot gain anything without first giving something in return. To obtain, something of equal value must be lost. That is Alchemy's First Law of Equivalent Exchange."*

Tu es un **Alchimiste d'État** travaillant sur le projet le plus ambitieux de Central : créer le **Cercle de Transmutation Ultime** — une bibliothèque qui peut transmuter (trier) n'importe quel matériau (données) avec une efficacité parfaite.

Le Colonel Mustang a besoin de cette bibliothèque pour le QG de l'armée. Elle doit contenir:

1. **Les Transmutations de Base** (Sorting Algorithms) — Comme les différentes formes d'alchimie
2. **La Recherche de la Vérité** (Search Algorithms) — Comme la quête de la Pierre Philosophale
3. **Les Techniques Avancées** — Comme les techniques interdites des Homunculi
4. **Le Cercle d'Allocation** (Arena) — La mémoire parfaitement contrôlée
5. **L'Œil de la Vérité** (Analysis) — Voir la complexité réelle

**Analogies FMA → Code:**

| FMA | Code |
|-----|------|
| Cercle de transmutation | API de la bibliothèque |
| Transmutation | Tri/transformation des données |
| Pierre Philosophale | Algorithme optimal recherché |
| Échange équivalent | Trade-off temps/espace |
| The Truth (La Vérité) | Complexité théorique |
| Homunculi | Bugs et edge cases |
| Gate of Truth | Interface publique |

**Ta mission :**

Construire `transmutation_circle` — la bibliothèque ultime de tri et recherche, digne d'un Alchimiste d'État.

### 1.2.2 Version Académique

Implémenter une **bibliothèque production-ready** de tri et recherche comprenant:

1. **12+ algorithmes de tri** : comparison-based et non-comparison-based
2. **8+ algorithmes de recherche** : binary search et variantes
3. **Techniques sur tableaux** : two pointers, sliding window, prefix sums
4. **Gestion mémoire** : arena allocator
5. **Analyse** : benchmarking et estimation de complexité
6. **Tests** : couverture ≥ 80%
7. **Documentation** : complète pour chaque fonction publique

**Structure de sortie :**
- Bibliothèque Rust compilable (`cargo build`)
- Tests passants (`cargo test`)
- Benchmarks exécutables (`cargo bench`)
- Documentation générée (`cargo doc`)

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  SORTING (12+ obligatoires):                                    │
│  - Bubble, Selection, Insertion, Shell                          │
│  - Merge (recursive + iterative)                                │
│  - Quick (standard, 3-way)                                      │
│  - Heap                                                         │
│  - Intro, Tim                                                   │
│  - Counting, Radix, Bucket                                      │
│                                                                 │
│  SEARCHING (8+ obligatoires):                                   │
│  - Binary (standard, lower_bound, upper_bound)                  │
│  - Search in rotated array                                      │
│  - Find peak element                                            │
│  - Ternary search                                               │
│  - Interpolation search                                         │
│  - Exponential search                                           │
│                                                                 │
│  TECHNIQUES:                                                    │
│  - Two pointers (two_sum, three_sum, partition)                 │
│  - Sliding window (max, sum, distinct)                          │
│  - Prefix sums (1D, 2D, difference array)                       │
│  - Coordinate compression                                        │
│                                                                 │
│  MEMORY:                                                        │
│  - Arena allocator avec alignment                               │
│                                                                 │
│  QUALITY:                                                       │
│  - ≥80% test coverage                                           │
│  - Documentation complète                                       │
│  - No unsafe sans justification                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 Prototype

**Rust (Edition 2024) :**

```rust
//! # Transmutation Circle
//!
//! La bibliothèque ultime de tri et recherche, créée par les Alchimistes d'État.
//!
//! ## Equivalent Exchange
//!
//! Chaque algorithme a son trade-off temps/espace:
//! - Merge Sort: O(n log n) temps, O(n) espace
//! - Quick Sort: O(n log n) moyen, O(log n) espace
//! - Counting Sort: O(n+k) temps, O(k) espace

pub mod vector;
pub mod sorting;
pub mod searching;
pub mod techniques;
pub mod memory;
pub mod analysis;

// === RE-EXPORTS (Gate of Truth) ===

pub use vector::AlchemyVec;

// Sorting - Les Transmutations
pub use sorting::{
    // Transmutations simples (O(n²))
    bubble_transmute,
    selection_transmute,
    insertion_transmute,
    shell_transmute,

    // Transmutations avancées (O(n log n))
    merge_transmute,
    merge_transmute_iterative,
    quick_transmute,
    quick_transmute_3way,
    heap_transmute,

    // Transmutations hybrides
    intro_transmute,
    tim_transmute,

    // Transmutations non-comparison (O(n))
    counting_transmute,
    radix_transmute,
    bucket_transmute,
};

// Searching - La Quête de la Vérité
pub use searching::{
    seek_truth,           // binary_search
    seek_lower_bound,     // lower_bound
    seek_upper_bound,     // upper_bound
    seek_in_rotated,      // search rotated array
    seek_peak,            // find peak element
    ternary_seek,         // ternary search
    interpolation_seek,   // interpolation search
    exponential_seek,     // exponential search
};

// Techniques - Les Techniques Interdites
pub use techniques::{
    // Two Pointers
    TwoPointerAlchemy,
    equivalent_exchange_pair,  // two_sum
    trinity_exchange,          // three_sum

    // Sliding Window
    WindowAlchemy,
    sliding_maximum,
    sliding_sum,

    // Prefix Sums
    PrefixCircle,
    DifferenceArray,

    // Compression
    CoordinateCompressor,
};

// Memory - Le Cercle d'Allocation
pub use memory::TransmutationArena;

// Analysis - L'Œil de la Vérité
pub use analysis::{
    TruthEye,             // Complexity estimator
    AlchemyBench,         // Benchmarking
    AlchemyReport,        // Results
};

/// Trait unifié pour tous les algorithmes de tri
pub trait Transmutable<T: Ord> {
    /// Transmuter (trier) les éléments en place
    fn transmute(&mut self);

    /// Transmuter avec un comparateur personnalisé
    fn transmute_by<F>(&mut self, compare: F)
    where
        F: FnMut(&T, &T) -> std::cmp::Ordering;

    /// Vérifier si la transmutation est complète (trié)
    fn is_transmuted(&self) -> bool;
}

/// Configuration de benchmark
#[derive(Clone)]
pub struct BenchConfig {
    pub sizes: Vec<usize>,
    pub iterations: usize,
    pub warmup_iterations: usize,
}

impl Default for BenchConfig {
    fn default() -> Self {
        BenchConfig {
            sizes: vec![100, 1000, 10000, 100000],
            iterations: 100,
            warmup_iterations: 10,
        }
    }
}

/// Exécuter tous les benchmarks
pub fn benchmark_all_transmutations(config: &BenchConfig) -> AlchemyReport;

/// Vérifier tous les algorithmes
pub fn verify_all_transmutations() -> TestReport;

/// Rapport de test
#[derive(Debug)]
pub struct TestReport {
    pub total: usize,
    pub passed: usize,
    pub failed: Vec<String>,
}
```

**Structure des modules :**

```rust
// === src/sorting/mod.rs ===

/// Transmutations de comparaison (comparison-based)
pub mod comparison;

/// Transmutations sans comparaison
pub mod non_comparison;

/// Transmutations hybrides
pub mod hybrid;

pub use comparison::*;
pub use non_comparison::*;
pub use hybrid::*;

// === src/sorting/comparison.rs ===

/// Bubble Sort - La transmutation la plus simple
/// Time: O(n²), Space: O(1), Stable: Yes
pub fn bubble_transmute<T: Ord>(elements: &mut [T]);

/// Selection Sort - Sélectionner le minimum
/// Time: O(n²), Space: O(1), Stable: No
pub fn selection_transmute<T: Ord>(elements: &mut [T]);

/// Insertion Sort - Insérer à la bonne place
/// Time: O(n²), Space: O(1), Stable: Yes
pub fn insertion_transmute<T: Ord>(elements: &mut [T]);

/// Shell Sort - Insertion avec gaps décroissants
/// Time: O(n^1.3), Space: O(1), Stable: No
pub fn shell_transmute<T: Ord>(elements: &mut [T]);

/// Merge Sort - Diviser pour régner
/// Time: O(n log n), Space: O(n), Stable: Yes
pub fn merge_transmute<T: Ord + Clone>(elements: &mut [T]);

/// Merge Sort itératif (bottom-up)
pub fn merge_transmute_iterative<T: Ord + Clone>(elements: &mut [T]);

/// Quick Sort - Partition autour d'un pivot
/// Time: O(n log n) average, O(n²) worst, Space: O(log n), Stable: No
pub fn quick_transmute<T: Ord>(elements: &mut [T]);

/// Quick Sort 3-way (Dutch National Flag)
/// Optimisé pour les doublons
pub fn quick_transmute_3way<T: Ord>(elements: &mut [T]);

/// Heap Sort - Utiliser un tas binaire
/// Time: O(n log n), Space: O(1), Stable: No
pub fn heap_transmute<T: Ord>(elements: &mut [T]);

// === src/sorting/non_comparison.rs ===

/// Counting Sort - Compter les occurrences
/// Time: O(n+k), Space: O(k), Stable: Yes
/// Fonctionne pour les entiers dans [0, k)
pub fn counting_transmute(elements: &mut [u32], max_value: u32);

/// Radix Sort - Trier par digit
/// Time: O(d*(n+k)), Space: O(n+k), Stable: Yes
pub fn radix_transmute(elements: &mut [u32]);

/// Bucket Sort - Distribuer dans des seaux
/// Time: O(n+k) average, Space: O(n), Stable: Yes
pub fn bucket_transmute(elements: &mut [f64]);

// === src/sorting/hybrid.rs ===

/// Intro Sort - Quick + Heap + Insertion
/// Time: O(n log n) guaranteed, Space: O(log n)
pub fn intro_transmute<T: Ord>(elements: &mut [T]);

/// Tim Sort - Merge + Insertion avec runs
/// Time: O(n log n), Space: O(n), Stable: Yes
pub fn tim_transmute<T: Ord + Clone>(elements: &mut [T]);

// === src/searching/mod.rs ===

pub mod binary;
pub mod ternary;

pub use binary::*;
pub use ternary::*;

/// Binary Search standard
/// Retourne Some(index) si trouvé, None sinon
pub fn seek_truth<T: Ord>(elements: &[T], target: &T) -> Option<usize>;

/// Lower bound - premier élément >= target
pub fn seek_lower_bound<T: Ord>(elements: &[T], target: &T) -> usize;

/// Upper bound - premier élément > target
pub fn seek_upper_bound<T: Ord>(elements: &[T], target: &T) -> usize;

/// Recherche dans tableau rotaté
pub fn seek_in_rotated<T: Ord>(elements: &[T], target: &T) -> Option<usize>;

/// Trouver un pic (élément plus grand que ses voisins)
pub fn seek_peak<T: Ord>(elements: &[T]) -> usize;

/// Ternary search pour fonction unimodale
pub fn ternary_seek_min<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
where
    F: Fn(f64) -> f64;

pub fn ternary_seek_max<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
where
    F: Fn(f64) -> f64;

/// Interpolation search
pub fn interpolation_seek(elements: &[i32], target: i32) -> Option<usize>;

/// Exponential search
pub fn exponential_seek<T: Ord>(elements: &[T], target: &T) -> Option<usize>;
```

**C Header (C17) :**

```c
#ifndef TRANSMUTATION_CIRCLE_H
#define TRANSMUTATION_CIRCLE_H

#include <stddef.h>
#include <stdbool.h>
#include <stdint.h>

// === SORTING ===

// Comparison-based
void bubble_transmute(int *arr, size_t n);
void selection_transmute(int *arr, size_t n);
void insertion_transmute(int *arr, size_t n);
void shell_transmute(int *arr, size_t n);
void merge_transmute(int *arr, size_t n);
void quick_transmute(int *arr, size_t n);
void heap_transmute(int *arr, size_t n);
void intro_transmute(int *arr, size_t n);

// Non-comparison
void counting_transmute(uint32_t *arr, size_t n, uint32_t max_val);
void radix_transmute(uint32_t *arr, size_t n);

// === SEARCHING ===

// Binary search
ssize_t seek_truth(const int *arr, size_t n, int target);
size_t seek_lower_bound(const int *arr, size_t n, int target);
size_t seek_upper_bound(const int *arr, size_t n, int target);
ssize_t seek_in_rotated(const int *arr, size_t n, int target);
size_t seek_peak(const int *arr, size_t n);

// === TECHNIQUES ===

// Two pointers
typedef struct { size_t i; size_t j; bool found; } PairResult;
PairResult equivalent_exchange_pair(const int *arr, size_t n, int target);

// Sliding window
int *sliding_maximum(const int *arr, size_t n, size_t k, size_t *out_len);

// Prefix sums
typedef struct {
    int64_t *prefix;
    size_t len;
} PrefixCircle;

PrefixCircle *prefix_circle_new(const int *arr, size_t n);
int64_t prefix_circle_range_sum(const PrefixCircle *pc, size_t l, size_t r);
void prefix_circle_destroy(PrefixCircle *pc);

// === MEMORY ===

typedef struct TransmutationArena TransmutationArena;

TransmutationArena *arena_new(size_t capacity);
void *arena_alloc(TransmutationArena *arena, size_t size, size_t align);
void arena_reset(TransmutationArena *arena);
void arena_destroy(TransmutationArena *arena);

// === UTILITIES ===

bool is_transmuted(const int *arr, size_t n);
void verify_all_transmutations(void);

#endif // TRANSMUTATION_CIRCLE_H
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 La bibliothèque standard de Rust utilise des algorithmes similaires

`slice::sort()` de Rust utilise un algorithme hybride proche de `pdqsort` (pattern-defeating quicksort), qui combine:
- Quick sort avec choix de pivot intelligent
- Insertion sort pour les petits tableaux
- Heap sort comme fallback

C'est exactement ce que fait `intro_transmute`!

### 2.2 Tim Sort: l'algorithme de Python

Tim Sort, inventé par Tim Peters pour Python, est utilisé aussi par Java (depuis 1.7). Il exploite les "runs" naturels dans les données réelles, où des sous-séquences sont déjà triées.

### 2.3 Pourquoi l'arène est cruciale

Dans un tri comme Merge Sort qui alloue beaucoup de mémoire temporaire, utiliser une arène peut réduire le temps d'exécution de 20-30% en évitant les appels répétés à malloc/free.

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation de cette bibliothèque |
|--------|-----------------------------------|
| **Systems Programmer** | Bibliothèque de tri custom pour kernel |
| **Database Developer** | Algorithms pour index B-tree |
| **Game Developer** | Sorting pour rendering order |
| **Financial Developer** | Tri de transactions en temps réel |
| **Compiler Engineer** | Tri de symboles pour linking |
| **Embedded Developer** | Algorithmes sans allocation |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls transmutation_circle/
Cargo.toml  README.md  benches/  src/  tests/

$ cd transmutation_circle && cargo build --release
   Compiling transmutation_circle v1.0.0
    Finished release [optimized] target(s)

$ cargo test
running 45 tests
test sorting::bubble_transmute ... ok
test sorting::merge_transmute ... ok
test sorting::quick_transmute ... ok
...
test integration::stress_test ... ok
test result: ok. 45 passed; 0 failed

$ cargo bench
transmutation/bubble/1000    time: [1.2345 ms 1.2567 ms 1.2789 ms]
transmutation/merge/1000     time: [45.234 µs 46.123 µs 47.012 µs]
transmutation/quick/1000     time: [38.456 µs 39.234 µs 40.012 µs]
...

$ cargo doc --open
Documenting transmutation_circle v1.0.0
   Generated docs at target/doc/transmutation_circle/index.html

All transmutations complete! The Philosopher's Stone is yours.
```

### 3.1 💀 BONUS EXPERT (OPTIONNEL)

**Difficulté Bonus :**
🧠 (11/10)

**Récompense :**
XP ×6

**Domaines Bonus :**
`MD, CPU, Calcul, Mem`

#### 3.1.1 Consigne Bonus

**🎮 FULLMETAL ALCHEMIST — La Pierre Philosophale**

*"The Philosopher's Stone is not about power. It's about understanding."*

Pour créer la vraie Pierre Philosophale, tu dois transcender les limites normales:

1. **SIMD Sorting** : Utiliser AVX2/AVX-512 pour trier 8-16 éléments en parallèle
2. **Parallel Sorting** : Utiliser Rayon pour parallel merge/quick sort
3. **Adaptive Sorting** : Détecter les patterns et choisir l'algorithme optimal
4. **Memory-Mapped Sorting** : Trier des fichiers plus grands que la RAM
5. **Visualization** : Générer des graphiques de performance

```rust
/// SIMD-optimized sorting network pour petits tableaux
#[cfg(target_feature = "avx2")]
pub fn simd_sort_8(elements: &mut [i32; 8]);

/// Parallel merge sort avec Rayon
pub fn parallel_merge_transmute<T: Ord + Clone + Send>(elements: &mut [T]);

/// Détection automatique d'algorithme optimal
pub fn adaptive_transmute<T: Ord + Clone>(elements: &mut [T]);

/// Tri de fichier (external sort)
pub fn external_transmute(input_path: &Path, output_path: &Path, memory_limit: usize) -> io::Result<()>;

/// Générer rapport HTML avec graphiques
pub fn generate_alchemy_report(config: &BenchConfig) -> String;
```

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  SIMD: Doit fonctionner sans SIMD comme fallback               │
│  Parallel: Speedup ≥ 2x sur 4 cores                            │
│  Adaptive: Choisir correctement pour sorted, reversed, random  │
│  External: Doit pouvoir trier 10GB avec 1GB RAM                │
│  Report: HTML avec graphiques (peut utiliser plotters crate)   │
└─────────────────────────────────────────────────────────────────┘
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (Tests)

| # | Catégorie | Test | Points |
|---|-----------|------|--------|
| 1 | Sorting | bubble_transmute correctness | 2 |
| 2 | Sorting | selection_transmute correctness | 2 |
| 3 | Sorting | insertion_transmute correctness | 2 |
| 4 | Sorting | shell_transmute correctness | 2 |
| 5 | Sorting | merge_transmute correctness | 3 |
| 6 | Sorting | merge_transmute stability | 2 |
| 7 | Sorting | quick_transmute correctness | 3 |
| 8 | Sorting | quick_transmute_3way correctness | 2 |
| 9 | Sorting | heap_transmute correctness | 3 |
| 10 | Sorting | intro_transmute correctness | 3 |
| 11 | Sorting | tim_transmute correctness | 3 |
| 12 | Sorting | counting_transmute correctness | 2 |
| 13 | Sorting | radix_transmute correctness | 2 |
| 14 | Sorting | bucket_transmute correctness | 2 |
| 15 | Searching | seek_truth correctness | 2 |
| 16 | Searching | seek_lower_bound correctness | 2 |
| 17 | Searching | seek_upper_bound correctness | 2 |
| 18 | Searching | seek_in_rotated correctness | 3 |
| 19 | Searching | seek_peak correctness | 2 |
| 20 | Searching | ternary_seek correctness | 2 |
| 21 | Techniques | equivalent_exchange_pair (two_sum) | 3 |
| 22 | Techniques | sliding_maximum correctness | 3 |
| 23 | Techniques | PrefixCircle range queries | 3 |
| 24 | Techniques | CoordinateCompressor correctness | 3 |
| 25 | Memory | TransmutationArena alloc | 3 |
| 26 | Memory | TransmutationArena alignment | 3 |
| 27 | Memory | TransmutationArena reset | 2 |
| 28 | Quality | cargo test passes | 5 |
| 29 | Quality | cargo clippy clean | 3 |
| 30 | Quality | Documentation présente | 5 |
| 31 | Quality | Benchmarks exécutables | 5 |
| 32 | Integration | Large scale test (10000 elements) | 5 |
| 33 | Integration | All edge cases (empty, single, sorted, reversed) | 10 |

### 4.2 Tests principaux

```rust
// tests/sorting_tests.rs

use transmutation_circle::*;

#[test]
fn test_all_sorting_algorithms() {
    let algorithms: Vec<(&str, fn(&mut [i32]))> = vec![
        ("bubble", |arr| bubble_transmute(arr)),
        ("selection", |arr| selection_transmute(arr)),
        ("insertion", |arr| insertion_transmute(arr)),
        ("shell", |arr| shell_transmute(arr)),
        ("merge", |arr| merge_transmute(arr)),
        ("quick", |arr| quick_transmute(arr)),
        ("quick_3way", |arr| quick_transmute_3way(arr)),
        ("heap", |arr| heap_transmute(arr)),
        ("intro", |arr| intro_transmute(arr)),
        ("tim", |arr| tim_transmute(arr)),
    ];

    for (name, sort_fn) in algorithms {
        // Random
        let mut arr = vec![5, 2, 8, 1, 9, 3, 7, 4, 6];
        sort_fn(&mut arr);
        assert!(arr.windows(2).all(|w| w[0] <= w[1]), "{} failed on random", name);

        // Already sorted
        let mut arr = vec![1, 2, 3, 4, 5];
        sort_fn(&mut arr);
        assert!(arr.windows(2).all(|w| w[0] <= w[1]), "{} failed on sorted", name);

        // Reverse sorted
        let mut arr = vec![5, 4, 3, 2, 1];
        sort_fn(&mut arr);
        assert!(arr.windows(2).all(|w| w[0] <= w[1]), "{} failed on reverse", name);

        // Empty
        let mut arr: Vec<i32> = vec![];
        sort_fn(&mut arr);

        // Single
        let mut arr = vec![42];
        sort_fn(&mut arr);
        assert_eq!(arr, [42], "{} failed on single", name);

        // All same
        let mut arr = vec![5, 5, 5, 5, 5];
        sort_fn(&mut arr);
        assert_eq!(arr, [5, 5, 5, 5, 5], "{} failed on same", name);

        // Large
        let mut arr: Vec<i32> = (0..1000).rev().collect();
        sort_fn(&mut arr);
        assert!(arr.windows(2).all(|w| w[0] <= w[1]), "{} failed on large", name);
    }
}

#[test]
fn test_stable_sorts() {
    #[derive(Clone, Debug)]
    struct Item { key: i32, order: usize }

    impl PartialEq for Item {
        fn eq(&self, other: &Self) -> bool { self.key == other.key }
    }
    impl Eq for Item {}
    impl PartialOrd for Item {
        fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
            Some(self.cmp(other))
        }
    }
    impl Ord for Item {
        fn cmp(&self, other: &Self) -> std::cmp::Ordering {
            self.key.cmp(&other.key)
        }
    }

    let original = vec![
        Item { key: 2, order: 0 },
        Item { key: 1, order: 1 },
        Item { key: 2, order: 2 },
        Item { key: 1, order: 3 },
    ];

    // Merge sort should be stable
    let mut arr = original.clone();
    merge_transmute(&mut arr);
    assert_eq!(arr[0].order, 1); // First 1
    assert_eq!(arr[1].order, 3); // Second 1
    assert_eq!(arr[2].order, 0); // First 2
    assert_eq!(arr[3].order, 2); // Second 2
}

#[test]
fn test_search_algorithms() {
    let arr = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // Binary search
    assert_eq!(seek_truth(&arr, &5), Some(4));
    assert_eq!(seek_truth(&arr, &11), None);
    assert_eq!(seek_truth(&arr, &0), None);

    // Lower/upper bounds
    let arr = vec![1, 2, 2, 2, 3, 4, 5];
    assert_eq!(seek_lower_bound(&arr, &2), 1);
    assert_eq!(seek_upper_bound(&arr, &2), 4);

    // Rotated
    let arr = vec![4, 5, 6, 7, 0, 1, 2];
    assert_eq!(seek_in_rotated(&arr, &0), Some(4));
    assert_eq!(seek_in_rotated(&arr, &3), None);

    // Peak
    let arr = vec![1, 3, 5, 7, 6, 4, 2];
    let peak = seek_peak(&arr);
    assert!(arr[peak] >= arr[peak.saturating_sub(1)]);
    assert!(arr[peak] >= arr.get(peak + 1).copied().unwrap_or(i32::MIN));
}

#[test]
fn test_techniques() {
    // Two sum
    let arr = vec![2, 7, 11, 15];
    let result = equivalent_exchange_pair(&arr, 9);
    assert!(result.found);
    assert_eq!(arr[result.i] + arr[result.j], 9);

    // Sliding window max
    let arr = vec![1, 3, -1, -3, 5, 3, 6, 7];
    let maxs = sliding_maximum(&arr, 3);
    assert_eq!(maxs, vec![3, 3, 5, 5, 6, 7]);

    // Prefix sums
    let arr = vec![1, 2, 3, 4, 5];
    let ps = PrefixCircle::new(&arr);
    assert_eq!(ps.range_sum(0, 4), 15);
    assert_eq!(ps.range_sum(1, 3), 9);
}

#[test]
fn test_arena() {
    let arena = TransmutationArena::new(1024);

    let a = arena.alloc(42i32).unwrap();
    assert_eq!(*a, 42);

    let b = arena.alloc(3.14f64).unwrap();
    let b_addr = b as *const f64 as usize;
    assert_eq!(b_addr % 8, 0); // Alignment

    let slice = arena.alloc_slice(0i32, 10).unwrap();
    assert_eq!(slice.len(), 10);

    unsafe { arena.reset(); }
    assert_eq!(arena.used(), 0);
}
```

### 4.3 Structure de la solution de référence

La solution complète est trop longue pour ce document. Voici la structure:

```
transmutation_circle/
├── Cargo.toml
│   [package]
│   name = "transmutation_circle"
│   version = "1.0.0"
│   edition = "2024"
│
│   [dependencies]
│   # Aucune dépendance pour les algos eux-mêmes
│
│   [dev-dependencies]
│   criterion = "0.5"
│   rand = "0.8"
│
│   [[bench]]
│   name = "alchemy_bench"
│   harness = false
│
├── src/lib.rs             # ~200 lignes (re-exports, traits)
├── src/vector.rs          # ~150 lignes (AlchemyVec)
├── src/sorting/
│   ├── mod.rs             # ~50 lignes
│   ├── comparison.rs      # ~400 lignes (9 algos)
│   ├── non_comparison.rs  # ~150 lignes (3 algos)
│   └── hybrid.rs          # ~300 lignes (intro, tim)
├── src/searching/
│   ├── mod.rs             # ~30 lignes
│   ├── binary.rs          # ~150 lignes (5 variantes)
│   └── ternary.rs         # ~100 lignes
├── src/techniques/
│   ├── mod.rs             # ~30 lignes
│   ├── two_pointers.rs    # ~100 lignes
│   ├── sliding_window.rs  # ~100 lignes
│   ├── prefix_sum.rs      # ~150 lignes
│   └── compression.rs     # ~100 lignes
├── src/memory/
│   └── arena.rs           # ~200 lignes
└── src/analysis/
    ├── complexity.rs      # ~150 lignes
    └── benchmark.rs       # ~200 lignes

Total: ~2000+ lignes de code bien documenté
```

### 4.9 spec.json

```json
{
  "name": "transmutation_circle",
  "language": "rust",
  "type": "project",
  "tier": 3,
  "tier_info": "Synthèse - Projet complet Module 1.1",
  "tags": ["sorting", "searching", "library", "production", "phase1", "mini-project"],
  "passing_score": 70,

  "project": {
    "structure": "cargo library with tests and benches",
    "min_algorithms": {
      "sorting": 12,
      "searching": 8
    },
    "required_modules": [
      "vector",
      "sorting",
      "searching",
      "techniques",
      "memory",
      "analysis"
    ]
  },

  "tests": {
    "categories": [
      {
        "name": "sorting_correctness",
        "tests": 14,
        "points": 30
      },
      {
        "name": "searching_correctness",
        "tests": 6,
        "points": 12
      },
      {
        "name": "techniques",
        "tests": 4,
        "points": 12
      },
      {
        "name": "memory",
        "tests": 3,
        "points": 8
      },
      {
        "name": "quality",
        "tests": 4,
        "points": 18
      },
      {
        "name": "integration",
        "tests": 2,
        "points": 20
      }
    ]
  },

  "quality": {
    "min_test_coverage": 80,
    "documentation_required": true,
    "clippy_clean": true,
    "no_warnings": true
  }
}
```

### 4.10 Erreurs communes (Mutants conceptuels)

```rust
/* Mutant A: Merge sort instable */
// ERREUR: Utilise > au lieu de >= pour la comparaison
fn merge_wrong<T: Ord + Clone>(left: &[T], right: &[T], result: &mut [T]) {
    let (mut i, mut j, mut k) = (0, 0, 0);
    while i < left.len() && j < right.len() {
        if left[i] > right[j] {  // ERREUR: devrait être <=
            result[k] = right[j].clone();
            j += 1;
        } else {
            result[k] = left[i].clone();
            i += 1;
        }
        k += 1;
    }
}
// Impact: Perd la stabilité du tri

/* Mutant B: Quick sort stack overflow sur tableau vide */
fn quick_transmute_wrong<T: Ord>(arr: &mut [T]) {
    // ERREUR: Pas de vérification de longueur
    let pivot_idx = partition(arr);  // Crash si arr.len() == 0
    quick_transmute_wrong(&mut arr[..pivot_idx]);
    quick_transmute_wrong(&mut arr[pivot_idx + 1..]);
}

/* Mutant C: Binary search avec off-by-one */
fn seek_truth_wrong<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    let (mut lo, mut hi) = (0, arr.len());  // ERREUR: hi devrait être arr.len() - 1
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        // ... boucle infinie possible
    }
    None
}

/* Mutant D: Arena reset incomplet */
impl TransmutationArena {
    pub unsafe fn reset_wrong(&self) {
        // ERREUR: Oublie de reset le pointeur
        // self.ptr.set(self.start);  <- manquant
    }
}

/* Mutant E: Benchmark sans warmup */
fn benchmark_wrong(sort_fn: fn(&mut [i32]), size: usize) -> Duration {
    let mut arr: Vec<i32> = (0..size).rev().collect();
    let start = Instant::now();
    sort_fn(&mut arr);
    start.elapsed()
    // ERREUR: Pas de warmup, résultats biaisés par le cache froid
}
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que ce mini-projet enseigne

1. **Architecture logicielle** : Organiser un projet en modules cohérents
2. **API Design** : Créer une interface publique intuitive et documentée
3. **Testing** : Écrire des tests exhaustifs avec edge cases
4. **Benchmarking** : Mesurer les performances de façon rigoureuse
5. **Documentation** : Documenter pour les utilisateurs et les contributeurs
6. **Intégration** : Faire fonctionner ensemble des composants complexes

### 5.2 LDA — Architecture globale

```
BIBLIOTHÈQUE transmutation_circle
DÉBUT BIBLIOTHÈQUE
    MODULE vector
        STRUCTURE AlchemyVec<T>
        FONCTIONS push, pop, get, len, capacity, reserve

    MODULE sorting
        SOUS-MODULE comparison
            FONCTIONS bubble, selection, insertion, shell, merge, quick, heap
        SOUS-MODULE non_comparison
            FONCTIONS counting, radix, bucket
        SOUS-MODULE hybrid
            FONCTIONS intro, tim

    MODULE searching
        FONCTIONS binary_search, lower_bound, upper_bound
        FONCTIONS rotated_search, peak_find
        FONCTIONS ternary, interpolation, exponential

    MODULE techniques
        STRUCTURES TwoPointerAlchemy, WindowAlchemy, PrefixCircle
        FONCTIONS two_sum, three_sum, sliding_max, range_sum

    MODULE memory
        STRUCTURE TransmutationArena
        FONCTIONS new, alloc, alloc_slice, reset

    MODULE analysis
        STRUCTURES TruthEye, AlchemyBench, AlchemyReport
        FONCTIONS benchmark_all, estimate_complexity
FIN BIBLIOTHÈQUE
```

### 5.2.2.1 Logic Flow — Pipeline de benchmark

```
ALGORITHME : Benchmark Pipeline
---
1. CONFIGURATION :
   |-- Définir sizes = [100, 1000, 10000, 100000]
   |-- Définir iterations = 100
   |-- Définir warmup = 10

2. POUR CHAQUE algorithme DANS [bubble, merge, quick, ...] :
   |
   |-- POUR CHAQUE size DANS sizes :
   |     |
   |     |-- GÉNÉRER tableau aléatoire de taille size
   |     |
   |     |-- WARMUP (répéter warmup fois sans mesurer) :
   |     |     Exécuter algorithme sur copie
   |     |
   |     |-- MESURER (répéter iterations fois) :
   |     |     |-- Cloner le tableau
   |     |     |-- Démarrer chrono
   |     |     |-- Exécuter algorithme
   |     |     |-- Arrêter chrono
   |     |     |-- Enregistrer durée
   |     |
   |     |-- CALCULER statistiques :
   |           mean, median, std_dev, min, max
   |
   |-- AJOUTER résultats au rapport

3. GÉNÉRER rapport final avec comparaisons
```

### 5.3 Visualisation ASCII

**Architecture de la bibliothèque :**

```
                      ┌─────────────────────────────────────┐
                      │     TRANSMUTATION CIRCLE            │
                      │        (Public API)                 │
                      │                                     │
                      │  ┌─────────┐ ┌─────────┐           │
                      │  │ Traits  │ │ Types   │           │
                      │  └────┬────┘ └────┬────┘           │
                      └───────┼───────────┼─────────────────┘
                              │           │
         ┌────────────────────┼───────────┼────────────────────┐
         │                    │           │                    │
         ▼                    ▼           ▼                    ▼
    ┌─────────┐         ┌─────────┐ ┌─────────┐         ┌─────────┐
    │ sorting │         │searching│ │techniques│        │ memory  │
    │         │         │         │ │          │        │         │
    ├─────────┤         ├─────────┤ ├──────────┤        ├─────────┤
    │comparison         │binary   │ │two_ptr   │        │arena    │
    │non_comp │         │ternary  │ │sliding   │        │         │
    │hybrid   │         │         │ │prefix    │        │         │
    └─────────┘         └─────────┘ └──────────┘        └─────────┘
         │                    │           │                    │
         └────────────────────┴───────────┴────────────────────┘
                                    │
                              ┌─────▼─────┐
                              │ analysis  │
                              │           │
                              │complexity │
                              │benchmark  │
                              └───────────┘
```

**Comparaison des algorithmes :**

```
    Complexité temporelle des algorithmes de tri:

    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │  O(n²)     ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  Bubble, Selection     │
    │            ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  Insertion             │
    │                                                             │
    │  O(n^1.3)  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓        Shell                  │
    │                                                             │
    │  O(n log n)▓▓▓▓▓▓▓▓▓▓                Merge, Quick, Heap     │
    │            ▓▓▓▓▓▓▓▓▓▓                Intro, Tim             │
    │                                                             │
    │  O(n)      ▓▓▓▓▓                      Counting, Radix       │
    │            ▓▓▓▓▓                      (non-comparison)      │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘

    Espace auxiliaire:

    │ O(1)      │ Bubble, Selection, Insertion, Shell, Heap       │
    │ O(log n)  │ Quick Sort (stack)                              │
    │ O(n)      │ Merge Sort, Tim Sort, Counting, Radix           │
```

### 5.4 Les pièges en détail

| Piège | Description | Impact | Solution |
|-------|-------------|--------|----------|
| **Stabilité** | Certains tris perdent l'ordre relatif | Tests de stabilité échouent | Utiliser <= au lieu de < dans merge |
| **Recursion** | Stack overflow sur grands tableaux | Crash | Limiter profondeur ou utiliser itératif |
| **Edge cases** | Tableau vide, singleton | Panic/crash | Vérifier len() == 0 ou 1 |
| **Benchmark bias** | Cache froid | Résultats faux | Warmup avant mesure |
| **Overflow** | Calcul de milieu | Index invalide | `lo + (hi - lo) / 2` |

### 5.5 Cours Complet

#### 5.5.1 Choisir le bon algorithme

| Situation | Algorithme recommandé | Pourquoi |
|-----------|----------------------|----------|
| Données petites (< 50) | Insertion Sort | Overhead faible |
| Données génériques | Quick Sort / Intro Sort | O(n log n) moyen |
| Stabilité requise | Merge Sort / Tim Sort | Stable garanti |
| Mémoire limitée | Heap Sort | O(1) espace |
| Entiers bornés | Counting Sort | O(n) temps |
| Presque trié | Tim Sort | Exploite les runs |

#### 5.5.2 Écrire des tests robustes

```rust
fn test_sort_algorithm<F>(sort_fn: F)
where
    F: Fn(&mut [i32]),
{
    // 1. Cas de base
    let mut empty: Vec<i32> = vec![];
    sort_fn(&mut empty);
    assert!(empty.is_empty());

    let mut single = vec![42];
    sort_fn(&mut single);
    assert_eq!(single, [42]);

    // 2. Cas triviaux
    let mut sorted = vec![1, 2, 3, 4, 5];
    sort_fn(&mut sorted);
    assert_is_sorted(&sorted);

    let mut reversed = vec![5, 4, 3, 2, 1];
    sort_fn(&mut reversed);
    assert_is_sorted(&reversed);

    // 3. Cas avec doublons
    let mut duplicates = vec![3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5];
    sort_fn(&mut duplicates);
    assert_is_sorted(&duplicates);

    // 4. Grand tableau aléatoire
    let mut large: Vec<i32> = (0..10000).map(|_| rand::random()).collect();
    sort_fn(&mut large);
    assert_is_sorted(&large);

    // 5. Patterns adversariaux
    let mut pipe_organ: Vec<i32> = (0..100).chain((0..100).rev()).collect();
    sort_fn(&mut pipe_organ);
    assert_is_sorted(&pipe_organ);
}
```

### 5.8 Mnémotechniques

#### 🎮 MEME : "Equivalent Exchange" — FMA et les Trade-offs

*"Humankind cannot gain anything without first giving something in return."*

C'est la Première Loi de l'Alchimie, et c'est EXACTEMENT comme les trade-offs algorithmiques:

| Trade-off | Équivalent FMA |
|-----------|----------------|
| Time vs Space | Échanger sa vie pour le pouvoir |
| Merge (O(n) space) vs Quick (O(log n)) | Pierre Philosophale vs alchimie normale |
| Stabilité vs Performance | Préserver l'âme vs efficacité |
| Généricité vs Spécialisation | Alchimie vs Alkahestry |

```rust
// Equivalent Exchange en action:
// Tu veux O(n log n) garanti? Tu paies avec O(n) mémoire (Merge)
// Tu veux O(1) mémoire? Tu risques O(n²) worst case (Quick)
// Tu veux les deux? Tu perds la stabilité (Intro)
```

#### 🔮 MEME : "The Gate of Truth" — L'API publique

Comme la Porte de la Vérité qui est l'interface entre les mondes, ton `lib.rs` est l'interface entre ta bibliothèque et les utilisateurs.

Ce qui passe la Porte = ce qui est `pub`
Ce qui reste caché = les détails d'implémentation

```rust
// lib.rs est ta Gate of Truth
pub use sorting::quick_transmute;  // Passe la porte
// Les détails internes restent derrière la porte
```

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Catégorie | Gravité |
|---|-------|-----------|---------|
| 1 | Merge sort instable | Correctness | Haute |
| 2 | Stack overflow quick sort | Runtime | Haute |
| 3 | Off-by-one binary search | Correctness | Haute |
| 4 | Pas de warmup benchmark | Performance | Moyenne |
| 5 | Arena reset incomplet | Memory | Haute |
| 6 | Pas de tests edge cases | Quality | Moyenne |
| 7 | Documentation manquante | Quality | Basse |
| 8 | Overflow calcul milieu | Runtime | Haute |

---

## 📝 SECTION 7 : QCM

### Question 1
Quel algorithme de tri a la meilleure complexité dans le PIRE cas?

- A) Quick Sort
- B) Merge Sort
- C) Counting Sort
- D) Insertion Sort
- E) Selection Sort

<details>
<summary>Réponse</summary>
**B ou C** selon le contexte. Merge Sort est O(n log n) worst case pour les comparison-based. Counting Sort est O(n+k) mais nécessite des entiers bornés.
</details>

### Question 2
Pourquoi Quick Sort est souvent plus rapide que Merge Sort en pratique malgré le même O(n log n)?

- A) Il est stable
- B) Meilleure localité cache (in-place)
- C) Il utilise moins de comparaisons
- D) Il ne récurse pas
- E) Il utilise plus de mémoire

<details>
<summary>Réponse</summary>
**B)** Quick Sort est in-place et a donc une meilleure localité cache. Merge Sort nécessite O(n) mémoire auxiliaire qui peut causer des cache misses.
</details>

### Question 3
Qu'est-ce qui fait de Tim Sort un bon choix pour des données réelles?

- A) Il est O(n) dans tous les cas
- B) Il exploite les "runs" naturellement triés
- C) Il n'utilise pas de mémoire
- D) Il est parallélisable
- E) Il ne compare pas les éléments

<details>
<summary>Réponse</summary>
**B)** Tim Sort détecte et exploite les sous-séquences déjà triées (runs) dans les données réelles, ce qui arrive souvent.
</details>

### Question 4
Quel est l'avantage de Counting Sort sur les comparison-based sorts?

- A) Il est stable
- B) Il utilise moins de mémoire
- C) Il peut être O(n) car il ne compare pas
- D) Il fonctionne sur tous les types
- E) Il est in-place

<details>
<summary>Réponse</summary>
**C)** Counting Sort ne fait pas de comparaisons et peut donc briser la borne Ω(n log n) des comparison-based sorts. Mais il nécessite des entiers bornés.
</details>

### Question 5
Pourquoi fait-on un "warmup" avant les benchmarks?

- A) Pour éviter les bugs
- B) Pour chauffer le CPU
- C) Pour remplir les caches et stabiliser les mesures
- D) Pour compiler le code
- E) Pour initialiser le RNG

<details>
<summary>Réponse</summary>
**C)** Le warmup remplit les caches (instruction et data) et permet d'avoir des mesures stables, non biaisées par le "cold start".
</details>

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.1.10 — transmutation_circle |
| **Type** | Mini-Projet |
| **Difficulté** | ★★★★★★★☆☆☆ (7/10) |
| **XP Base** | 500 |
| **XP Bonus** | ×6 (3000 XP) |
| **Temps estimé** | 180 min (3h) |
| **Lignes de code** | ~2000+ |
| **Algorithmes** | 20+ (12 tri, 8 recherche) |
| **Langage** | Rust Edition 2024 / C17 |
| **Référence culture** | Fullmetal Alchemist: Brotherhood |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.1.10-transmutation-circle",
    "generated_at": "2026-01-11 11:00:00",

    "metadata": {
      "exercise_id": "1.1.10",
      "exercise_name": "transmutation_circle",
      "module": "1.1",
      "module_name": "Arrays & Sorting",
      "concept": "ALL",
      "concept_name": "Complete Library (110 concepts)",
      "type": "project",
      "tier": 3,
      "tier_info": "Synthèse - Mini-Project",
      "phase": 1,
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "language": "rust",
      "duration_minutes": 180,
      "xp_base": 500,
      "xp_bonus_multiplier": 6,
      "bonus_tier": "GÉNIE",
      "bonus_icon": "🧠",
      "complexity_time": "Variable",
      "complexity_space": "Variable",
      "prerequisites": ["all-1.1-exercises"],
      "domains": ["Tri", "Struct", "Mem", "MD", "CPU"],
      "domains_bonus": ["Calcul"],
      "tags": ["mini-project", "library", "sorting", "searching", "production"],
      "meme_reference": "Fullmetal Alchemist - Equivalent Exchange"
    },

    "project_structure": {
      "cargo_toml": true,
      "readme": true,
      "src/lib.rs": true,
      "src/vector.rs": true,
      "src/sorting/": ["mod.rs", "comparison.rs", "non_comparison.rs", "hybrid.rs"],
      "src/searching/": ["mod.rs", "binary.rs", "ternary.rs"],
      "src/techniques/": ["mod.rs", "two_pointers.rs", "sliding_window.rs", "prefix_sum.rs", "compression.rs"],
      "src/memory/": ["arena.rs"],
      "src/analysis/": ["complexity.rs", "benchmark.rs"],
      "benches/": ["alchemy_bench.rs"],
      "tests/": ["sorting_tests.rs", "integration_tests.rs"]
    },

    "validation": {
      "cargo_build": true,
      "cargo_test": true,
      "cargo_clippy": true,
      "cargo_doc": true,
      "min_coverage": 80
    }
  }
}
```

---

*Exercice généré par HACKBRAIN v5.5.2 — "Equivalent Exchange: To obtain, something of equal value must be lost."*
*L'excellence pédagogique ne se négocie pas — pas de raccourcis*
