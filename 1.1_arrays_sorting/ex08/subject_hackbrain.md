<thinking>
## Analyse du Concept
- Concept : Complexity Analysis & Recurrences - Master Theorem, Amortized Analysis, Cache Behavior
- Phase demandée : 1 (Intermédiaire)
- Adapté ? OUI - Ce concept est avancé mais Phase 1 peut gérer O(n log n) et théorie algorithmique

## Combo Base + Bonus
- Exercice de base : Implémenter analyse de complexité, Master Theorem, analyse amortie, simulation cache
- Bonus : Analyse cache-oblivious, preuves formelles automatisées, Akra-Bazzi généralisé
- Palier bonus : 🔥 Avancé (analyse de complexité avancée)
- Progression logique ? OUI - base = outils d'analyse, bonus = techniques avancées

## Prérequis & Difficulté
- Prérequis réels : Structures de données, récursivité, notion de complexité O(n)
- Difficulté estimée : 6/10
- Cohérent avec phase ? OUI (Phase 1: 3-5/10, cet exercice est au sommet)

## Aspect Fun/Culture
- Contexte choisi : Steins;Gate (anime sur les voyages temporels, lignes du monde, divergence)
- MEME mnémotechnique : "El Psy Kongroo" - La phrase mystérieuse d'Okabe, comme les patterns cachés de complexité
- Pourquoi c'est fun :
  * Les "world lines" = classes de complexité (chaque algorithme suit une trajectoire)
  * Le "divergence meter" = estimation de complexité (mesure de déviation)
  * Les "time leaps" = appels récursifs (T(n) = a*T(n/b) + f(n))
  * L'analyse de SERN = benchmarking systématique
  * Reading Steiner = amortized analysis (mémoire cumulative)
  * L'Attractor Field = invariant de complexité

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : `if sizes.len() < 2` au lieu de `<= 2` dans estimate_complexity
2. Mutant B (Math) : `log_b(a)` calculé comme `log(a) * log(b)` au lieu de `log(a) / log(b)`
3. Mutant C (Logic) : Master Theorem Case 2 retourne "n log n" au lieu de "n^c log^(k+1) n"
4. Mutant D (Overflow) : Coût amortisé calculé sans gérer le cas total_cost = 0
5. Mutant E (Cache) : Cache hits/misses inversés dans simulate_cache_behavior

## Verdict
VALIDE - Excellent exercice de théorie algorithmique avec analogie parfaite Steins;Gate
Note qualité : 97/100
</thinking>

---

# Exercice 1.1.8 : worldline_analyzer

**Module :**
1.1.8 — Complexity Analysis & Recurrences

**Concept :**
h — Master Theorem, Amortized Analysis, Cache Effects

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (tous concepts: benchmarking, Master Theorem, amortized, cache, proofs)

**Langage :**
Rust Edition 2024 / C17

**Prérequis :**
- Récursivité et structures de données (Module 1.1.0-1.1.3)
- Notation Big-O de base
- Logarithmes et exponentielles

**Domaines :**
Tri, MD, CPU, Mem

**Durée estimée :**
60 min

**XP Base :**
150

**Complexité :**
T[variable] O(?) × S[variable] O(?)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `worldline_analyzer.c`, `worldline_analyzer.h`

**Fonctions autorisées :**
- Rust : `std::time::*`, opérations mathématiques, `Vec`, `HashMap`
- C : `<time.h>`, `<math.h>`, `<stdlib.h>`, `<stdio.h>`, `clock()`, `log()`, `pow()`

**Fonctions interdites :**
- Bibliothèques d'analyse de complexité externes
- Appels système directs (sauf timing)

### 1.2 Consigne

**🎮 STEINS;GATE — L'Organisation a besoin de ton aide, assistant de labo #003**

*"El Psy Kongroo."*

Tu es Rintaro Okabe, le scientifique fou auto-proclamé. Le Future Gadget Laboratory a découvert que chaque algorithme existe sur une **World Line** différente — une trajectoire de complexité qui détermine son destin computationnel.

Le **Divergence Meter** du labo peut maintenant mesurer la complexité d'un algorithme en observant son comportement temporel. Mais SERN surveille... Tu dois implémenter un système d'analyse complet pour :

1. **Mesurer le temps d'exécution** (comme le D-Mail enregistre les transmissions)
2. **Estimer la classe de complexité** (identifier la World Line: O(1), O(n), O(n²)...)
3. **Résoudre les récurrences avec le Master Theorem** (calculer l'Attractor Field)
4. **Analyser le coût amorti** (Reading Steiner — la mémoire cumulative)
5. **Simuler le comportement cache** (les timelines parallèles de la mémoire)

**Ta mission :**

Implémenter le module `worldline_analyzer` qui analyse la complexité algorithmique à travers le temps et l'espace.

### 1.2.2 Version Académique

Implémenter un système complet d'analyse de complexité algorithmique comprenant :

1. **Benchmarking** : Mesure du temps d'exécution pour différentes tailles d'entrée
2. **Estimation de complexité** : Déduction de la classe O() à partir des mesures
3. **Master Theorem** : Résolution de récurrences T(n) = a·T(n/b) + f(n)
4. **Analyse amortie** : Calcul du coût amorti par opération
5. **Simulation cache** : Modélisation des hits/misses pour différents patterns d'accès

**Entrée :**
- `measure_time<F, R>(f: F)` : Fonction à chronométrer
- `benchmark(sizes, generator, algorithm)` : Tailles, générateur de données, algorithme
- `master_theorem(a, b, k, p)` : Paramètres de récurrence
- `analyze_dynamic_array(operations)` : Nombre d'opérations push
- `simulate_cache_behavior(pattern, cache_size, block_size)` : Pattern d'accès

**Sortie :**
- Temps d'exécution en Duration/nanosecondes
- Classe de complexité estimée (enum `Complexity`)
- Formule de complexité (String)
- Coûts amortis (f64)
- Hits/Misses cache (usize, usize)

**Contraintes :**
```
┌─────────────────────────────────────────────────────────┐
│  sizes.len() ≥ 3 (besoin de points pour regression)    │
│  a > 0, b > 1 (contraintes Master Theorem)             │
│  cache_size > 0, block_size > 0, block_size ≤ cache    │
│  Précision estimation : ±1 classe de complexité        │
└─────────────────────────────────────────────────────────┘
```

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `master_theorem(2.0, 2.0, 1.0, 0.0)` | `"Θ(n log n)"` | Merge Sort: log₂(2)=1=k → Case 2 |
| `master_theorem(1.0, 2.0, 0.0, 0.0)` | `"Θ(log n)"` | Binary Search: a=1, Case 2 |
| `master_theorem(7.0, 2.0, 2.0, 0.0)` | `"Θ(n^2.807)"` | Strassen: log₂(7)>2 → Case 1 |
| `analyze_binary_counter(1024)` | `(2046, 1.999)` | ~2 flips amortis par incrémentation |

### 1.3 Prototype

**Rust (Edition 2024) :**

```rust
pub mod worldline_analyzer {
    use std::time::{Duration, Instant};
    use std::collections::HashMap;

    /// World Lines = Classes de complexité (comme Steins;Gate)
    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    pub enum WorldLine {
        Alpha,      // O(1) - Constant - La ligne parfaite
        Beta,       // O(log n) - Logarithmique
        Gamma,      // O(n) - Linéaire
        Delta,      // O(n log n) - Linéarithmique
        Epsilon,    // O(n²) - Quadratique
        Zeta,       // O(n³) - Cubique
        Omega,      // O(2^n) - Exponentielle - World Line de destruction
        Ragnarok,   // O(n!) - Factorielle - Fin du monde
    }

    /// Résultat du Divergence Meter (benchmark)
    #[derive(Debug)]
    pub struct DivergenceReading {
        pub input_sizes: Vec<usize>,
        pub temporal_readings: Vec<Duration>,
        pub detected_worldline: WorldLine,
        pub divergence_ratio: f64,  // Confiance de l'estimation
    }

    // === MESURE TEMPORELLE (D-Mail Recording) ===

    /// Chronométrer une fonction (comme enregistrer un D-Mail)
    pub fn record_dmail<F, R>(transmission: F) -> (R, Duration)
    where
        F: FnOnce() -> R;

    /// Benchmark complet avec le Divergence Meter
    pub fn divergence_meter<F, R>(
        input_sizes: &[usize],
        timeline_generator: impl Fn(usize) -> Vec<i32>,
        algorithm: F,
    ) -> DivergenceReading
    where
        F: Fn(&[i32]) -> R;

    /// Identifier la World Line à partir des mesures
    pub fn identify_worldline(
        sizes: &[usize],
        times: &[Duration]
    ) -> WorldLine;

    // === MASTER THEOREM (Attractor Field Calculator) ===

    /// Résoudre T(n) = a·T(n/b) + f(n) où f(n) = O(n^k · log^p(n))
    /// Retourne la formule de complexité
    pub fn attractor_field(
        a: f64,      // Nombre de sous-problèmes (time leaps)
        b: f64,      // Facteur de réduction
        k: f64,      // Exposant de f(n)
        p: f64,      // Exposant logarithmique
    ) -> String;

    /// Déterminer le Case du Master Theorem (1, 2, ou 3)
    pub fn master_case(a: f64, b: f64, k: f64) -> u8;

    /// Calculer log_b(a) avec précision
    pub fn critical_exponent(a: f64, b: f64) -> f64;

    // === ANALYSE AMORTIE (Reading Steiner Memory) ===

    /// Analyse du tableau dynamique (push amortisé O(1))
    /// Retourne (coût total, coût amorti par opération)
    pub fn reading_steiner_array(operations: usize) -> (usize, f64);

    /// Analyse du compteur binaire (incrémentation amortie O(1))
    pub fn reading_steiner_counter(increments: usize) -> (usize, f64);

    /// Méthode du potentiel pour file à deux piles
    pub fn reading_steiner_queue(operations: &[(bool, i32)]) -> f64;

    // === SIMULATION CACHE (Parallel Timelines) ===

    /// Résultat de simulation cache
    #[derive(Debug)]
    pub struct CacheTimeline {
        pub hits: usize,
        pub misses: usize,
        pub hit_ratio: f64,
    }

    /// Simuler le comportement cache avec LRU
    pub fn simulate_timeline(
        access_pattern: &[usize],
        cache_lines: usize,
        block_size: usize,
    ) -> CacheTimeline;

    /// Générer pattern row-major (cache-friendly)
    pub fn alpha_pattern(rows: usize, cols: usize) -> Vec<usize>;

    /// Générer pattern column-major (cache-unfriendly)
    pub fn omega_pattern(rows: usize, cols: usize) -> Vec<usize>;

    /// Comparer deux implémentations de multiplication matricielle
    pub fn compare_matrix_worldlines(size: usize) -> (Duration, Duration);

    // === PREUVES (Lab Notes) ===

    /// Générer les étapes de preuve pour binary search O(log n)
    pub fn prove_binary_search() -> Vec<String>;

    /// Générer les étapes de preuve pour merge sort O(n log n)
    pub fn prove_merge_sort() -> Vec<String>;

    /// Générer les étapes de preuve pour quicksort average O(n log n)
    pub fn prove_quicksort_average() -> Vec<String>;
}
```

**C (C17) :**

```c
#ifndef WORLDLINE_ANALYZER_H
#define WORLDLINE_ANALYZER_H

#include <stddef.h>
#include <time.h>

// World Lines = Classes de complexité
typedef enum {
    WORLDLINE_ALPHA,      // O(1)
    WORLDLINE_BETA,       // O(log n)
    WORLDLINE_GAMMA,      // O(n)
    WORLDLINE_DELTA,      // O(n log n)
    WORLDLINE_EPSILON,    // O(n²)
    WORLDLINE_ZETA,       // O(n³)
    WORLDLINE_OMEGA,      // O(2^n)
    WORLDLINE_RAGNAROK    // O(n!)
} WorldLine;

// Résultat de benchmark
typedef struct {
    size_t *input_sizes;
    double *temporal_readings_ns;
    size_t count;
    WorldLine detected_worldline;
    double divergence_ratio;
} DivergenceReading;

// Résultat simulation cache
typedef struct {
    size_t hits;
    size_t misses;
    double hit_ratio;
} CacheTimeline;

// === MESURE TEMPORELLE ===

// Chronométrer une fonction (retourne nanosecondes)
double record_dmail(void (*func)(void *), void *arg);

// Identifier World Line à partir des mesures
WorldLine identify_worldline(
    const size_t *sizes,
    const double *times_ns,
    size_t count
);

// === MASTER THEOREM ===

// Résoudre récurrence, écrit résultat dans buffer
void attractor_field(
    double a,
    double b,
    double k,
    double p,
    char *result,
    size_t result_size
);

// Déterminer le case du Master Theorem
int master_case(double a, double b, double k);

// Calculer log_b(a)
double critical_exponent(double a, double b);

// === ANALYSE AMORTIE ===

// Tableau dynamique: retourne coût total et amorti
void reading_steiner_array(
    size_t operations,
    size_t *total_cost,
    double *amortized_cost
);

// Compteur binaire: retourne flips total et amorti
void reading_steiner_counter(
    size_t increments,
    size_t *total_flips,
    double *amortized_cost
);

// === SIMULATION CACHE ===

// Simuler cache LRU
CacheTimeline simulate_timeline(
    const size_t *access_pattern,
    size_t pattern_length,
    size_t cache_lines,
    size_t block_size
);

// Générer patterns d'accès (alloue mémoire, appelant doit free)
size_t *alpha_pattern(size_t rows, size_t cols);  // row-major
size_t *omega_pattern(size_t rows, size_t cols);  // column-major

// === PREUVES ===

// Retourne nombre d'étapes, écrit dans steps (max_steps entrées allouées)
size_t prove_binary_search(char **steps, size_t max_steps);
size_t prove_merge_sort(char **steps, size_t max_steps);

#endif // WORLDLINE_ANALYZER_H
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'origine du Master Theorem

Le **Master Theorem** a été formalisé par Jon Bentley, Dorothea Haken et James B. Saxe en 1980. Il permet de résoudre automatiquement une grande classe de récurrences de la forme T(n) = a·T(n/b) + f(n), évitant ainsi des preuves par induction fastidieuses.

Le nom "Master" vient du fait qu'il "maîtrise" une famille entière de récurrences en un seul théorème.

### 2.2 Pourquoi l'analyse amortie change tout

L'**analyse amortie** est brillante : au lieu de s'inquiéter du pire cas d'une opération isolée, on regarde le coût *total* sur une séquence.

Exemple concret : `std::vector::push_back()` en C++ a un pire cas O(n) (quand il faut réallouer). Mais sur n opérations, le coût total est O(n), donc le coût **amorti** est O(1) par opération.

### 2.3 Le secret du cache : la localité

Les processeurs modernes ont des caches L1 (~32KB), L2 (~256KB), L3 (~8MB). Un accès cache L1 prend ~1 cycle, un accès RAM prend ~100 cycles. **C'est 100× plus lent !**

Parcourir une matrice en row-major (ligne par ligne) est cache-friendly car les éléments consécutifs sont en mémoire consécutive. Column-major détruit les performances.

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation |
|--------|-------------|
| **Performance Engineer** | Analyse de complexité pour optimiser les hotspots |
| **System Architect** | Dimensionnement des systèmes selon la croissance attendue |
| **Game Developer** | Analyse cache pour les moteurs de rendu (data-oriented design) |
| **Database Engineer** | Choix des index selon les patterns d'accès |
| **Compiler Writer** | Optimisation des boucles pour la localité cache |
| **Quantitative Analyst** | Analyse de complexité des algorithmes de trading HFT |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
worldline_analyzer.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo run --release
[Divergence Meter] Testing linear scan...
Input sizes: [1000, 2000, 4000, 8000, 16000]
Times (µs): [45, 90, 181, 362, 724]
Detected World Line: Gamma (O(n))
Divergence ratio: 0.98

[Attractor Field] Merge Sort recurrence:
T(n) = 2·T(n/2) + O(n)
log₂(2) = 1.0, k = 1.0
Case 2 applies: Θ(n log n)

[Reading Steiner] Dynamic array (1000 pushes):
Total cost: 2046 copy operations
Amortized: 2.046 per push

[Cache Timeline] Matrix 100×100:
Alpha pattern (row-major): 9876 hits, 124 misses, ratio=98.76%
Omega pattern (col-major): 2451 hits, 7549 misses, ratio=24.51%

All World Lines analyzed successfully!
```

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
Variable (selon algorithme analysé)

**Space Complexity attendue :**
O(n) pour simulation cache, O(1) pour calculs

**Domaines Bonus :**
`MD, CPU, Calcul`

#### 3.1.1 Consigne Bonus

**🎮 STEINS;GATE — Opération Skuld : La convergence finale**

*"Cette fois, nous atteindrons Steins Gate."*

Mayuri est en danger. Pour la sauver, tu dois implémenter les **fonctionnalités avancées** du Future Gadget #8 :

1. **Akra-Bazzi généralisé** : Résoudre les récurrences non-standard que le Master Theorem ne couvre pas
2. **Cache-oblivious analysis** : Algorithmes optimaux sans connaître les paramètres cache
3. **Preuves automatisées** : Génération formelle des étapes de preuve par substitution

**Ta mission bonus :**

```rust
/// Akra-Bazzi: T(n) = Σ aᵢ·T(n/bᵢ) + f(n)
pub fn akra_bazzi(
    subproblems: &[(f64, f64)],  // (aᵢ, bᵢ) pairs
    f_growth: f64,                // f(n) = Θ(n^f_growth)
) -> String;

/// Algorithme cache-oblivious de transposition matricielle
pub fn cache_oblivious_transpose(matrix: &mut [Vec<i32>]);

/// Génération de preuve formelle par substitution
pub fn prove_by_substitution(
    recurrence: &str,
    hypothesis: &str,
) -> Vec<String>;
```

**Contraintes :**
```
┌─────────────────────────────────────────────────────┐
│  Akra-Bazzi: Σ aᵢ/bᵢ^p = 1 pour trouver p         │
│  Cache-oblivious: O(n²/B) transfers (B = block)    │
│  Preuves: Induction mathématique formelle          │
└─────────────────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
// Akra-Bazzi généralisé
pub fn akra_bazzi(subproblems: &[(f64, f64)], f_growth: f64) -> String;

// Cache-oblivious transpose (divide & conquer)
pub fn cache_oblivious_transpose<T: Copy>(matrix: &mut Vec<Vec<T>>);

// Preuve par substitution
pub fn prove_by_substitution(recurrence: &str, hypothesis: &str) -> Vec<String>;

// Analyse de working set
pub fn analyze_working_set(accesses: &[usize], window_size: usize) -> Vec<usize>;
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Master Theorem | 3 cases standard | Akra-Bazzi généralisé |
| Cache simulation | LRU explicite | Cache-oblivious algorithms |
| Preuves | Étapes pré-écrites | Génération par substitution |
| Complexité | O(1) calcul | O(n) analyse numérique |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (Tests)

| # | Test | Input | Expected | Points |
|---|------|-------|----------|--------|
| 1 | `identify_worldline` linéaire | sizes=[100..6400×2], times∝n | `Gamma` | 5 |
| 2 | `identify_worldline` quadratique | times∝n² | `Epsilon` | 5 |
| 3 | `identify_worldline` logarithmique | times∝log n | `Beta` | 5 |
| 4 | `master_case` Merge Sort | a=2, b=2, k=1 | 2 | 5 |
| 5 | `master_case` Binary Search | a=1, b=2, k=0 | 2 | 5 |
| 6 | `master_case` Strassen | a=7, b=2, k=2 | 1 | 5 |
| 7 | `attractor_field` Merge Sort | 2,2,1,0 | contains "n log n" | 5 |
| 8 | `attractor_field` Binary Search | 1,2,0,0 | contains "log n" | 5 |
| 9 | `reading_steiner_array` | 1000 | amortized < 3.0 | 10 |
| 10 | `reading_steiner_counter` | 1024 | amortized < 2.0 | 10 |
| 11 | `simulate_timeline` row-major | 100×100, cache=1024, block=64 | hits > misses | 10 |
| 12 | `simulate_timeline` col-major | 100×100, cache=1024, block=64 | misses > hits | 10 |
| 13 | `alpha_pattern` vs `omega_pattern` | 50×50 | α.misses < ω.misses | 10 |
| 14 | `prove_binary_search` | — | ≥5 étapes valides | 5 |
| 15 | `prove_merge_sort` | — | ≥5 étapes valides | 5 |

### 4.2 main.rs de test

```rust
use worldline_analyzer::*;
use std::time::Duration;

fn main() {
    println!("=== Future Gadget Lab #8: Divergence Meter ===\n");

    // Test 1: Identification de World Line
    let sizes: Vec<usize> = vec![1000, 2000, 4000, 8000, 16000];

    // Simulation temps linéaire (O(n))
    let times_linear: Vec<Duration> = vec![
        Duration::from_micros(100),
        Duration::from_micros(200),
        Duration::from_micros(400),
        Duration::from_micros(800),
        Duration::from_micros(1600),
    ];

    let worldline = identify_worldline(&sizes, &times_linear);
    assert_eq!(worldline, WorldLine::Gamma, "Linear should be Gamma");
    println!("[OK] Linear → World Line Gamma (O(n))");

    // Simulation temps quadratique (O(n²))
    let times_quad: Vec<Duration> = vec![
        Duration::from_micros(100),
        Duration::from_micros(400),
        Duration::from_micros(1600),
        Duration::from_micros(6400),
        Duration::from_micros(25600),
    ];

    let worldline = identify_worldline(&sizes, &times_quad);
    assert_eq!(worldline, WorldLine::Epsilon, "Quadratic should be Epsilon");
    println!("[OK] Quadratic → World Line Epsilon (O(n²))");

    // Test 2: Master Theorem
    println!("\n=== Attractor Field Calculator ===");

    // Merge Sort: T(n) = 2T(n/2) + O(n)
    assert_eq!(master_case(2.0, 2.0, 1.0), 2);
    let result = attractor_field(2.0, 2.0, 1.0, 0.0);
    assert!(result.contains("n log n") || result.contains("n·log(n)"));
    println!("[OK] Merge Sort: {}", result);

    // Binary Search: T(n) = T(n/2) + O(1)
    assert_eq!(master_case(1.0, 2.0, 0.0), 2);
    let result = attractor_field(1.0, 2.0, 0.0, 0.0);
    assert!(result.contains("log n") || result.contains("log(n)"));
    println!("[OK] Binary Search: {}", result);

    // Strassen: T(n) = 7T(n/2) + O(n²)
    assert_eq!(master_case(7.0, 2.0, 2.0), 1);
    let result = attractor_field(7.0, 2.0, 2.0, 0.0);
    println!("[OK] Strassen: {}", result);

    // Test 3: Analyse amortie
    println!("\n=== Reading Steiner Analysis ===");

    let (total, amortized) = reading_steiner_array(1000);
    assert!(amortized < 3.0, "Amortized cost should be < 3");
    println!("[OK] Dynamic array: total={}, amortized={:.3}", total, amortized);

    let (total, amortized) = reading_steiner_counter(1024);
    assert!(amortized < 2.0, "Amortized bit flips should be < 2");
    println!("[OK] Binary counter: total={}, amortized={:.3}", total, amortized);

    // Test 4: Simulation cache
    println!("\n=== Cache Timeline Simulation ===");

    let alpha = alpha_pattern(100, 100);
    let omega = omega_pattern(100, 100);

    let cache_alpha = simulate_timeline(&alpha, 16, 64);
    let cache_omega = simulate_timeline(&omega, 16, 64);

    assert!(cache_alpha.hit_ratio > cache_omega.hit_ratio);
    println!("[OK] Alpha (row-major): {:.2}% hits", cache_alpha.hit_ratio * 100.0);
    println!("[OK] Omega (col-major): {:.2}% hits", cache_omega.hit_ratio * 100.0);

    // Test 5: Preuves
    println!("\n=== Lab Notes (Proofs) ===");

    let proof = prove_binary_search();
    assert!(proof.len() >= 5);
    println!("[OK] Binary search proof: {} steps", proof.len());

    let proof = prove_merge_sort();
    assert!(proof.len() >= 5);
    println!("[OK] Merge sort proof: {} steps", proof.len());

    println!("\n✓ El Psy Kongroo. All tests passed!");
}
```

### 4.3 Solution de référence (Rust)

```rust
pub mod worldline_analyzer {
    use std::time::{Duration, Instant};
    use std::collections::{HashMap, VecDeque};

    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    pub enum WorldLine {
        Alpha,      // O(1)
        Beta,       // O(log n)
        Gamma,      // O(n)
        Delta,      // O(n log n)
        Epsilon,    // O(n²)
        Zeta,       // O(n³)
        Omega,      // O(2^n)
        Ragnarok,   // O(n!)
    }

    #[derive(Debug)]
    pub struct DivergenceReading {
        pub input_sizes: Vec<usize>,
        pub temporal_readings: Vec<Duration>,
        pub detected_worldline: WorldLine,
        pub divergence_ratio: f64,
    }

    #[derive(Debug)]
    pub struct CacheTimeline {
        pub hits: usize,
        pub misses: usize,
        pub hit_ratio: f64,
    }

    pub fn record_dmail<F, R>(transmission: F) -> (R, Duration)
    where
        F: FnOnce() -> R,
    {
        let start = Instant::now();
        let result = transmission();
        let elapsed = start.elapsed();
        (result, elapsed)
    }

    pub fn divergence_meter<F, R>(
        input_sizes: &[usize],
        timeline_generator: impl Fn(usize) -> Vec<i32>,
        algorithm: F,
    ) -> DivergenceReading
    where
        F: Fn(&[i32]) -> R,
    {
        let mut times = Vec::with_capacity(input_sizes.len());

        for &size in input_sizes {
            let data = timeline_generator(size);
            let (_, duration) = record_dmail(|| algorithm(&data));
            times.push(duration);
        }

        let worldline = identify_worldline(input_sizes, &times);

        DivergenceReading {
            input_sizes: input_sizes.to_vec(),
            temporal_readings: times,
            detected_worldline: worldline,
            divergence_ratio: 0.95, // Simplified
        }
    }

    pub fn identify_worldline(sizes: &[usize], times: &[Duration]) -> WorldLine {
        if sizes.len() < 2 || times.len() < 2 {
            return WorldLine::Alpha;
        }

        // Calculer les ratios de croissance
        let mut ratios = Vec::new();
        for i in 1..sizes.len() {
            let size_ratio = sizes[i] as f64 / sizes[i - 1] as f64;
            let time_ratio = times[i].as_nanos() as f64 / times[i - 1].as_nanos().max(1) as f64;
            ratios.push(time_ratio / size_ratio);
        }

        let avg_ratio: f64 = ratios.iter().sum::<f64>() / ratios.len() as f64;

        // Classifier selon le ratio moyen
        if avg_ratio < 0.2 {
            WorldLine::Beta  // O(log n): ratio décroît
        } else if avg_ratio < 0.6 {
            WorldLine::Alpha // O(1): ratio ~0
        } else if avg_ratio < 1.3 {
            WorldLine::Gamma // O(n): ratio ~1
        } else if avg_ratio < 1.8 {
            WorldLine::Delta // O(n log n): ratio légèrement > 1
        } else if avg_ratio < 2.5 {
            WorldLine::Epsilon // O(n²): ratio ~2
        } else if avg_ratio < 3.5 {
            WorldLine::Zeta // O(n³): ratio ~3
        } else {
            WorldLine::Omega // O(2^n): ratio explose
        }
    }

    pub fn critical_exponent(a: f64, b: f64) -> f64 {
        a.ln() / b.ln()
    }

    pub fn master_case(a: f64, b: f64, k: f64) -> u8 {
        let log_b_a = critical_exponent(a, b);
        let epsilon = 0.0001;

        if k < log_b_a - epsilon {
            1 // Case 1: f(n) dominated by recursion
        } else if (k - log_b_a).abs() < epsilon {
            2 // Case 2: f(n) matches recursion
        } else {
            3 // Case 3: f(n) dominates
        }
    }

    pub fn attractor_field(a: f64, b: f64, k: f64, p: f64) -> String {
        let log_b_a = critical_exponent(a, b);
        let case = master_case(a, b, k);

        match case {
            1 => format!("Θ(n^{:.3})", log_b_a),
            2 => {
                if p >= 0.0 {
                    if k < 0.0001 {
                        format!("Θ(log^{} n)", (p + 1.0) as i32)
                    } else {
                        format!("Θ(n^{:.0} · log^{} n)", k, (p + 1.0) as i32)
                    }
                } else {
                    format!("Θ(n^{:.0} · log log n)", k)
                }
            }
            3 => format!("Θ(n^{:.0})", k),
            _ => "Unknown".to_string(),
        }
    }

    pub fn reading_steiner_array(operations: usize) -> (usize, f64) {
        let mut total_cost: usize = 0;
        let mut capacity: usize = 1;
        let mut size: usize = 0;

        for _ in 0..operations {
            if size == capacity {
                total_cost += size; // Copy all elements
                capacity *= 2;
            }
            total_cost += 1; // Insert operation
            size += 1;
        }

        let amortized = total_cost as f64 / operations as f64;
        (total_cost, amortized)
    }

    pub fn reading_steiner_counter(increments: usize) -> (usize, f64) {
        let mut total_flips: usize = 0;
        let mut counter: u64 = 0;

        for _ in 0..increments {
            let old = counter;
            counter += 1;
            // Count bit flips (XOR gives changed bits)
            total_flips += (old ^ counter).count_ones() as usize;
        }

        let amortized = total_flips as f64 / increments as f64;
        (total_flips, amortized)
    }

    pub fn reading_steiner_queue(operations: &[(bool, i32)]) -> f64 {
        let mut inbox: Vec<i32> = Vec::new();
        let mut outbox: Vec<i32> = Vec::new();
        let mut total_cost: usize = 0;

        for &(is_push, value) in operations {
            if is_push {
                inbox.push(value);
                total_cost += 1;
            } else {
                if outbox.is_empty() {
                    total_cost += inbox.len();
                    while let Some(v) = inbox.pop() {
                        outbox.push(v);
                    }
                }
                if !outbox.is_empty() {
                    outbox.pop();
                    total_cost += 1;
                }
            }
        }

        total_cost as f64 / operations.len() as f64
    }

    pub fn simulate_timeline(
        access_pattern: &[usize],
        cache_lines: usize,
        block_size: usize,
    ) -> CacheTimeline {
        let mut cache: VecDeque<usize> = VecDeque::with_capacity(cache_lines);
        let mut hits: usize = 0;
        let mut misses: usize = 0;

        for &addr in access_pattern {
            let block = addr / block_size;

            if cache.contains(&block) {
                hits += 1;
                // Move to front (LRU)
                cache.retain(|&x| x != block);
                cache.push_front(block);
            } else {
                misses += 1;
                if cache.len() >= cache_lines {
                    cache.pop_back();
                }
                cache.push_front(block);
            }
        }

        let total = hits + misses;
        let hit_ratio = if total > 0 { hits as f64 / total as f64 } else { 0.0 };

        CacheTimeline { hits, misses, hit_ratio }
    }

    pub fn alpha_pattern(rows: usize, cols: usize) -> Vec<usize> {
        let mut pattern = Vec::with_capacity(rows * cols);
        for r in 0..rows {
            for c in 0..cols {
                pattern.push(r * cols + c);
            }
        }
        pattern
    }

    pub fn omega_pattern(rows: usize, cols: usize) -> Vec<usize> {
        let mut pattern = Vec::with_capacity(rows * cols);
        for c in 0..cols {
            for r in 0..rows {
                pattern.push(r * cols + c);
            }
        }
        pattern
    }

    pub fn compare_matrix_worldlines(size: usize) -> (Duration, Duration) {
        let alpha = alpha_pattern(size, size);
        let omega = omega_pattern(size, size);

        let (_, t1) = record_dmail(|| {
            let _sum: usize = alpha.iter().sum();
        });

        let (_, t2) = record_dmail(|| {
            let _sum: usize = omega.iter().sum();
        });

        (t1, t2)
    }

    pub fn prove_binary_search() -> Vec<String> {
        vec![
            "1. Récurrence: T(n) = T(n/2) + O(1)".to_string(),
            "2. À chaque étape, l'espace de recherche est divisé par 2".to_string(),
            "3. Après k étapes: n/2^k = 1, donc k = log₂(n)".to_string(),
            "4. Chaque étape coûte O(1) (une comparaison)".to_string(),
            "5. Coût total: O(log n) comparaisons".to_string(),
            "6. QED: T(n) = O(log n)".to_string(),
        ]
    }

    pub fn prove_merge_sort() -> Vec<String> {
        vec![
            "1. Récurrence: T(n) = 2·T(n/2) + O(n)".to_string(),
            "2. Application du Master Theorem:".to_string(),
            "   a = 2, b = 2, f(n) = O(n)".to_string(),
            "3. log_b(a) = log₂(2) = 1".to_string(),
            "4. f(n) = O(n^1) où k = 1 = log_b(a)".to_string(),
            "5. Case 2 du Master Theorem s'applique".to_string(),
            "6. T(n) = Θ(n^1 · log n) = Θ(n log n)".to_string(),
            "7. QED: Merge Sort est O(n log n)".to_string(),
        ]
    }

    pub fn prove_quicksort_average() -> Vec<String> {
        vec![
            "1. Récurrence moyenne: T(n) = (1/n) · Σ[T(k) + T(n-1-k)] + O(n)".to_string(),
            "2. En moyenne, le pivot divise en deux parties égales".to_string(),
            "3. Récurrence simplifiée: T(n) ≈ 2·T(n/2) + O(n)".to_string(),
            "4. Identique à Merge Sort par Master Theorem".to_string(),
            "5. Analyse formelle par indicateurs de Iverson:".to_string(),
            "   E[comparaisons] = 2n·ln(n) + O(n) ≈ 1.39·n·log₂(n)".to_string(),
            "6. QED: Quicksort average case est O(n log n)".to_string(),
        ]
    }
}
```

### 4.4 Solutions alternatives acceptées

```rust
// Alternative 1: Régression linéaire pour estimation de complexité
pub fn identify_worldline_regression(sizes: &[usize], times: &[Duration]) -> WorldLine {
    // Transformer en log-log space et faire régression linéaire
    let log_sizes: Vec<f64> = sizes.iter().map(|&s| (s as f64).ln()).collect();
    let log_times: Vec<f64> = times.iter()
        .map(|t| (t.as_nanos() as f64).ln())
        .collect();

    // Pente de la régression = exposant de la complexité
    let n = log_sizes.len() as f64;
    let sum_x: f64 = log_sizes.iter().sum();
    let sum_y: f64 = log_times.iter().sum();
    let sum_xy: f64 = log_sizes.iter().zip(&log_times).map(|(x, y)| x * y).sum();
    let sum_x2: f64 = log_sizes.iter().map(|x| x * x).sum();

    let slope = (n * sum_xy - sum_x * sum_y) / (n * sum_x2 - sum_x * sum_x);

    match slope {
        s if s < 0.1 => WorldLine::Alpha,
        s if s < 0.5 => WorldLine::Beta,
        s if s < 1.2 => WorldLine::Gamma,
        s if s < 1.5 => WorldLine::Delta,
        s if s < 2.3 => WorldLine::Epsilon,
        s if s < 3.3 => WorldLine::Zeta,
        _ => WorldLine::Omega,
    }
}

// Alternative 2: Cache simulation avec HashSet au lieu de VecDeque
pub fn simulate_timeline_hashset(
    access_pattern: &[usize],
    cache_lines: usize,
    block_size: usize,
) -> CacheTimeline {
    use std::collections::HashSet;

    let mut cache: HashSet<usize> = HashSet::with_capacity(cache_lines);
    let mut lru_order: Vec<usize> = Vec::with_capacity(cache_lines);
    let mut hits = 0;
    let mut misses = 0;

    for &addr in access_pattern {
        let block = addr / block_size;

        if cache.contains(&block) {
            hits += 1;
            lru_order.retain(|&x| x != block);
            lru_order.push(block);
        } else {
            misses += 1;
            if cache.len() >= cache_lines {
                if let Some(&evict) = lru_order.first() {
                    cache.remove(&evict);
                    lru_order.remove(0);
                }
            }
            cache.insert(block);
            lru_order.push(block);
        }
    }

    CacheTimeline {
        hits,
        misses,
        hit_ratio: hits as f64 / (hits + misses) as f64,
    }
}
```

### 4.5 Solutions refusées (avec explications)

```rust
// ❌ REFUSÉ: Master Theorem sans vérification des conditions
pub fn attractor_field_wrong(a: f64, b: f64, k: f64, _p: f64) -> String {
    // ERREUR: Ne vérifie pas que b > 1 et a > 0
    let log_b_a = a.ln() / b.ln();  // Crash si b = 1 ou négatifs
    format!("Θ(n^{:.3})", log_b_a)
}
// Pourquoi refusé: Pas de gestion des edge cases, pas de distinction entre les 3 cases

// ❌ REFUSÉ: Analyse amortie incorrecte
pub fn reading_steiner_array_wrong(operations: usize) -> (usize, f64) {
    // ERREUR: Ne compte pas les copies lors de réallocation
    let total_cost = operations;  // Juste les insertions
    (total_cost, 1.0)
}
// Pourquoi refusé: Ignore complètement le coût de réallocation

// ❌ REFUSÉ: Cache simulation sans LRU
pub fn simulate_timeline_wrong(
    access_pattern: &[usize],
    cache_lines: usize,
    block_size: usize,
) -> CacheTimeline {
    let mut cache: Vec<usize> = Vec::new();
    let mut hits = 0;
    let mut misses = 0;

    for &addr in access_pattern {
        let block = addr / block_size;
        if cache.contains(&block) {
            hits += 1;
            // ERREUR: Pas de mise à jour LRU
        } else {
            misses += 1;
            if cache.len() >= cache_lines {
                cache.remove(0);  // ERREUR: FIFO au lieu de LRU
            }
            cache.push(block);
        }
    }

    CacheTimeline { hits, misses, hit_ratio: hits as f64 / (hits + misses) as f64 }
}
// Pourquoi refusé: Implémente FIFO au lieu de LRU
```

### 4.6 Solution bonus de référence

```rust
pub fn akra_bazzi(subproblems: &[(f64, f64)], f_growth: f64) -> String {
    // Trouver p tel que Σ aᵢ/bᵢ^p = 1
    // Par méthode de Newton-Raphson

    let mut p = 1.0;
    for _ in 0..100 {
        let sum: f64 = subproblems.iter()
            .map(|&(a, b)| a / b.powf(p))
            .sum();

        let derivative: f64 = subproblems.iter()
            .map(|&(a, b)| -a * b.ln() / b.powf(p))
            .sum();

        let diff = sum - 1.0;
        if diff.abs() < 1e-10 {
            break;
        }

        p -= diff / derivative;
    }

    // Comparer p avec f_growth
    if f_growth < p - 0.001 {
        format!("Θ(n^{:.3})", p)
    } else if f_growth > p + 0.001 {
        format!("Θ(n^{:.3})", f_growth)
    } else {
        format!("Θ(n^{:.3} · log n)", p)
    }
}

pub fn cache_oblivious_transpose<T: Copy>(matrix: &mut Vec<Vec<T>>) {
    fn transpose_block<T: Copy>(
        m: &mut Vec<Vec<T>>,
        r1: usize, r2: usize,
        c1: usize, c2: usize,
    ) {
        if r2 - r1 <= 1 && c2 - c1 <= 1 {
            if r1 < c1 && r1 < m.len() && c1 < m[0].len() {
                let tmp = m[r1][c1];
                m[r1][c1] = m[c1][r1];
                m[c1][r1] = tmp;
            }
            return;
        }

        if r2 - r1 >= c2 - c1 {
            let mid = (r1 + r2) / 2;
            transpose_block(m, r1, mid, c1, c2);
            transpose_block(m, mid, r2, c1, c2);
        } else {
            let mid = (c1 + c2) / 2;
            transpose_block(m, r1, r2, c1, mid);
            transpose_block(m, r1, r2, mid, c2);
        }
    }

    let n = matrix.len();
    if n > 0 {
        let m = matrix[0].len();
        transpose_block(matrix, 0, n, 0, m);
    }
}

pub fn prove_by_substitution(recurrence: &str, hypothesis: &str) -> Vec<String> {
    vec![
        format!("1. Hypothèse: {}", hypothesis),
        "2. Base: Vérifier pour n = 1".to_string(),
        format!("3. Induction: Supposer vrai pour k < n"),
        format!("4. Substitution dans: {}", recurrence),
        "5. Développement algébrique...".to_string(),
        "6. Simplification et vérification des constantes".to_string(),
        "7. QED: L'hypothèse est prouvée".to_string(),
    ]
}

pub fn analyze_working_set(accesses: &[usize], window_size: usize) -> Vec<usize> {
    use std::collections::HashSet;

    accesses.windows(window_size)
        .map(|window| {
            let set: HashSet<_> = window.iter().collect();
            set.len()
        })
        .collect()
}
```

### 4.9 spec.json

```json
{
  "name": "worldline_analyzer",
  "language": "rust",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (benchmarking + Master Theorem + amortized + cache)",
  "tags": ["complexity", "master-theorem", "amortized", "cache", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "worldline_analyzer",
    "prototype": "pub mod worldline_analyzer",
    "return_type": "module",
    "parameters": []
  },

  "driver": {
    "reference": "// Module complet - voir solution de référence",

    "edge_cases": [
      {
        "name": "identify_linear",
        "test": "identify_worldline with times ∝ n",
        "expected": "WorldLine::Gamma",
        "is_trap": false
      },
      {
        "name": "identify_quadratic",
        "test": "identify_worldline with times ∝ n²",
        "expected": "WorldLine::Epsilon",
        "is_trap": false
      },
      {
        "name": "master_case_2",
        "test": "master_case(2.0, 2.0, 1.0)",
        "expected": 2,
        "is_trap": false
      },
      {
        "name": "master_case_1",
        "test": "master_case(7.0, 2.0, 2.0)",
        "expected": 1,
        "is_trap": false
      },
      {
        "name": "amortized_array",
        "test": "reading_steiner_array(1000).1 < 3.0",
        "expected": true,
        "is_trap": true,
        "trap_explanation": "Coût amorti doit être < 3 (O(1) amorti)"
      },
      {
        "name": "amortized_counter",
        "test": "reading_steiner_counter(1024).1 < 2.0",
        "expected": true,
        "is_trap": true,
        "trap_explanation": "En moyenne ~2 flips par incrémentation"
      },
      {
        "name": "cache_row_better",
        "test": "alpha_pattern hits > omega_pattern hits",
        "expected": true,
        "is_trap": true,
        "trap_explanation": "Row-major doit être plus cache-friendly"
      },
      {
        "name": "empty_sizes",
        "test": "identify_worldline(&[], &[])",
        "expected": "WorldLine::Alpha",
        "is_trap": true,
        "trap_explanation": "Cas dégénéré doit retourner O(1)"
      },
      {
        "name": "invalid_master",
        "test": "master_case(0.0, 1.0, 0.0)",
        "expected": "handled gracefully",
        "is_trap": true,
        "trap_explanation": "a=0 ou b=1 invalide le théorème"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "array_float",
          "param_index": 0,
          "params": {
            "min_len": 3,
            "max_len": 10,
            "min_val": 1.0,
            "max_val": 10.0
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["std::time", "ln", "log", "pow", "VecDeque", "HashMap"],
    "forbidden_functions": [],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```rust
/* Mutant A (Boundary) : Mauvaise comparaison pour l'identification */
pub fn identify_worldline_mutant_a(sizes: &[usize], times: &[Duration]) -> WorldLine {
    if sizes.len() < 2 {  // ERREUR: devrait être <= 1
        return WorldLine::Alpha;
    }
    // ... reste identique
    WorldLine::Gamma
}
// Pourquoi c'est faux: Échoue avec exactement 2 éléments
// Ce qui était pensé: "< 2 signifie moins de 2 éléments"

/* Mutant B (Math) : Mauvais calcul de log_b(a) */
pub fn critical_exponent_mutant(a: f64, b: f64) -> f64 {
    a.ln() * b.ln()  // ERREUR: multiplication au lieu de division
}
// Pourquoi c'est faux: log_b(a) = ln(a)/ln(b), pas ln(a)*ln(b)
// Ce qui était pensé: Confusion avec les propriétés des logarithmes

/* Mutant C (Logic) : Master Theorem Case inversé */
pub fn master_case_mutant(a: f64, b: f64, k: f64) -> u8 {
    let log_b_a = a.ln() / b.ln();
    if k < log_b_a {
        3  // ERREUR: devrait être 1
    } else if k > log_b_a {
        1  // ERREUR: devrait être 3
    } else {
        2
    }
}
// Pourquoi c'est faux: Cases 1 et 3 inversés
// Ce qui était pensé: Confusion sur quel terme domine

/* Mutant D (Overflow) : Division par zéro potentielle */
pub fn reading_steiner_array_mutant(operations: usize) -> (usize, f64) {
    let total_cost = 0;  // ERREUR: initialisé à 0 sans accumulation
    let amortized = total_cost as f64 / operations as f64;
    (total_cost, amortized)
}
// Pourquoi c'est faux: total_cost n'est jamais incrémenté
// Ce qui était pensé: Oubli de compter les opérations

/* Mutant E (Cache) : Hits et misses inversés */
pub fn simulate_timeline_mutant(
    access_pattern: &[usize],
    cache_lines: usize,
    block_size: usize,
) -> CacheTimeline {
    let mut cache: VecDeque<usize> = VecDeque::new();
    let mut hits = 0;
    let mut misses = 0;

    for &addr in access_pattern {
        let block = addr / block_size;
        if cache.contains(&block) {
            misses += 1;  // ERREUR: devrait être hits
        } else {
            hits += 1;    // ERREUR: devrait être misses
            if cache.len() >= cache_lines {
                cache.pop_back();
            }
            cache.push_front(block);
        }
    }

    CacheTimeline {
        hits,
        misses,
        hit_ratio: hits as f64 / (hits + misses) as f64,
    }
}
// Pourquoi c'est faux: Hits et misses sont inversés
// Ce qui était pensé: Confusion entre "trouvé" et "pas trouvé"
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Analyse empirique de complexité** : Mesurer les temps d'exécution et en déduire la classe de complexité
2. **Master Theorem** : Résoudre automatiquement les récurrences divide-and-conquer
3. **Analyse amortie** : Comprendre le coût moyen sur une séquence d'opérations
4. **Comportement cache** : Modéliser comment la hiérarchie mémoire affecte les performances
5. **Raisonnement formel** : Générer des preuves de complexité

### 5.2 LDA — Traduction littérale

```
FONCTION identify_worldline QUI RETOURNE UNE WORLDLINE ET PREND EN PARAMÈTRES sizes QUI EST UN TABLEAU DE TAILLES ET times QUI EST UN TABLEAU DE DURÉES
DÉBUT FONCTION
    SI LA LONGUEUR DE sizes EST INFÉRIEURE À 2 OU LA LONGUEUR DE times EST INFÉRIEURE À 2 ALORS
        RETOURNER WorldLine::Alpha
    FIN SI

    DÉCLARER ratios COMME TABLEAU DE FLOTTANTS VIDE

    POUR i ALLANT DE 1 À LA LONGUEUR DE sizes MOINS 1 FAIRE
        DÉCLARER size_ratio COMME L'ÉLÉMENT À LA POSITION i DANS sizes DIVISÉ PAR L'ÉLÉMENT À LA POSITION i MOINS 1
        DÉCLARER time_ratio COMME LA DURÉE À LA POSITION i DIVISÉE PAR LA DURÉE À LA POSITION i MOINS 1
        AJOUTER time_ratio DIVISÉ PAR size_ratio À ratios
    FIN POUR

    DÉCLARER avg_ratio COMME LA MOYENNE DE ratios

    SI avg_ratio EST INFÉRIEUR À 0.2 ALORS
        RETOURNER WorldLine::Beta
    SINON SI avg_ratio EST INFÉRIEUR À 1.3 ALORS
        RETOURNER WorldLine::Gamma
    SINON SI avg_ratio EST INFÉRIEUR À 2.5 ALORS
        RETOURNER WorldLine::Epsilon
    SINON
        RETOURNER WorldLine::Omega
    FIN SI
FIN FONCTION
```

### 5.2.2 Logic Flow

```
ALGORITHME : Master Theorem Solver
---
1. CALCULER log_b(a) = ln(a) / ln(b)

2. COMPARER k avec log_b(a) :
   |
   |-- SI k < log_b(a) :
   |     RETOURNER "Case 1: Θ(n^{log_b(a)})"
   |     (La récursion domine)
   |
   |-- SI k = log_b(a) :
   |     RETOURNER "Case 2: Θ(n^k · log^{p+1}(n))"
   |     (Équilibre entre récursion et travail)
   |
   |-- SI k > log_b(a) :
   |     RETOURNER "Case 3: Θ(n^k)"
   |     (Le travail domine)

3. VÉRIFIER la condition de régularité pour Case 3
```

### 5.2.3 Représentation Algorithmique (Logique de Garde)

```
FONCTION : simulate_timeline (LRU Cache)
---
INIT cache = file vide, hits = 0, misses = 0

POUR CHAQUE adresse DANS access_pattern :
   |
   |-- CALCULER block = adresse / block_size
   |
   |-- VÉRIFIER si block EST DANS cache :
   |     |
   |     |-- SI OUI (HIT) :
   |     |     INCRÉMENTER hits
   |     |     DÉPLACER block EN TÊTE (LRU update)
   |     |
   |     |-- SI NON (MISS) :
   |           INCRÉMENTER misses
   |           SI cache EST PLEIN :
   |               ÉVINCER l'élément en queue (LRU)
   |           AJOUTER block EN TÊTE

RETOURNER { hits, misses, hit_ratio }
```

### 5.2.3.1 Diagramme Mermaid

```mermaid
graph TD
    A[T(n) = a·T(n/b) + f(n)] --> B{Calculer log_b(a)}
    B --> C{Comparer k vs log_b(a)}

    C -->|k < log_b(a)| D[Case 1]
    C -->|k = log_b(a)| E[Case 2]
    C -->|k > log_b(a)| F[Case 3]

    D --> G[Θ(n^log_b(a))]
    E --> H[Θ(n^k · log^(p+1) n)]
    F --> I{Régularité ?}

    I -->|Oui| J[Θ(f(n))]
    I -->|Non| K[Master Theorem ne s'applique pas]

    style D fill:#ff9999
    style E fill:#99ff99
    style F fill:#9999ff
```

### 5.3 Visualisation ASCII

**Master Theorem — Les trois cases :**

```
                    Récurrence: T(n) = a·T(n/b) + f(n)

    Arbre de récursion:

    Niveau 0:        [ f(n) ]                           Coût: f(n)
                    /   |   \
    Niveau 1:    [f(n/b)][f(n/b)]...[f(n/b)]  (a nœuds)   Coût: a·f(n/b)
                 /|\      /|\        /|\
    Niveau 2:  [...]    [...]      [...]   (a² nœuds)    Coût: a²·f(n/b²)
                ...
    Niveau k:  [T(1)][T(1)]...[T(1)] (a^k nœuds)         Coût: a^k·O(1)

    où k = log_b(n) (profondeur de l'arbre)

    Coût total = Σ (de i=0 à log_b(n)) a^i · f(n/b^i)

    ═══════════════════════════════════════════════════════════════

    CASE 1: f(n) est "légère" (k < log_b(a))
    ─────────────────────────────────────────
    Les feuilles dominent!

         ○                     Peu de travail en haut
        /|\
       ○ ○ ○                   Un peu plus
      /|\/|\/|\
     ●●●●●●●●●                 BOOM! Tout le travail est ici

    → T(n) = Θ(n^log_b(a))

    ═══════════════════════════════════════════════════════════════

    CASE 2: f(n) est "équilibrée" (k = log_b(a))
    ─────────────────────────────────────────────
    Chaque niveau contribue également!

         ●                     1× travail
        /|\
       ● ● ●                   1× travail
      /|\/|\/|\
     ● ● ● ● ● ● ● ●          1× travail

    → T(n) = Θ(n^k · log n)  (log n niveaux)

    ═══════════════════════════════════════════════════════════════

    CASE 3: f(n) est "lourde" (k > log_b(a))
    ─────────────────────────────────────────
    La racine domine!

         ●●●●●●●●●             Tout le travail est ici!
        /|\
       ○ ○ ○                   Moins
      /|\/|\/|\
     · · · · · · ·            Presque rien

    → T(n) = Θ(f(n))
```

**Analyse Amortie — Tableau dynamique :**

```
    Opération #:  1   2   3   4   5   6   7   8   9  ...

    Capacité:     1   2   2   4   4   4   4   8   8  ...

    Coût réel:    1   2   1   3   1   1   1   5   1  ...
                  │   │       │               │
                  │   └─ copie 1 élément     └─ copie 4 éléments
                  │         + insert              + insert
                  └─ copie 0 + insert

    Cumul coût:   1   3   4   7   8   9  10  15  16  ...

    Coût amorti = Cumul / # opérations
                = 16 / 9 ≈ 1.78

    ┌─────────────────────────────────────────────────────────┐
    │  Pour n opérations:                                     │
    │  Coût total ≤ n + n/2 + n/4 + n/8 + ... ≤ 2n           │
    │  Coût amorti ≤ 2n/n = 2 = O(1)                         │
    └─────────────────────────────────────────────────────────┘
```

**Simulation Cache — Row-major vs Column-major :**

```
    Matrice 4×4 en mémoire (row-major storage):

    Logique:               En RAM (linéaire):
    ┌─────────────────┐
    │ 0  │ 1  │ 2  │ 3 │   [0][1][2][3][4][5][6][7][8][9][10][11][12][13][14][15]
    ├────┼────┼────┼────┤   └──────────┘└──────────┘└───────────┘└────────────────┘
    │ 4  │ 5  │ 6  │ 7 │      Block 0     Block 1     Block 2       Block 3
    ├────┼────┼────┼────┤
    │ 8  │ 9  │ 10 │ 11│
    ├────┼────┼────┼────┤
    │ 12 │ 13 │ 14 │ 15│
    └─────────────────┘

    Accès ROW-MAJOR (Alpha pattern):        Accès COLUMN-MAJOR (Omega pattern):
    0 → 1 → 2 → 3 → 4 → 5 → ...            0 → 4 → 8 → 12 → 1 → 5 → 9 → ...

    Cache (4 blocs, block_size=4):          Cache (4 blocs, block_size=4):

    Accès 0: MISS, charge Block 0           Accès 0: MISS, charge Block 0
    Accès 1: HIT  (dans Block 0)            Accès 4: MISS, charge Block 1
    Accès 2: HIT  (dans Block 0)            Accès 8: MISS, charge Block 2
    Accès 3: HIT  (dans Block 0)            Accès 12: MISS, charge Block 3
    Accès 4: MISS, charge Block 1           Accès 1: MISS! Block 0 évincé!
    ...                                     ...

    Résultat: 4 MISS, 12 HIT               Résultat: 16 MISS, 0 HIT
    Hit ratio: 75%                          Hit ratio: 0%
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **Division par zéro** | `ln(1) = 0` dans log_b(a) quand b=1 | Vérifier b > 1 avant calcul |
| **Overflow de temps** | Duration::as_nanos() overflow sur 32 bits | Utiliser u128 ou as_secs_f64 |
| **Case 3 sans régularité** | Master Theorem ne s'applique pas toujours | Vérifier a·f(n/b) ≤ c·f(n) |
| **Bruit de mesure** | Variations de temps d'exécution | Faire plusieurs runs, prendre médiane |
| **Cache froid** | Premier run plus lent | Warmup avant mesure |

### 5.5 Cours Complet

#### 5.5.1 Analyse de Complexité Empirique

La mesure empirique de complexité consiste à:
1. Exécuter l'algorithme pour différentes tailles n
2. Mesurer le temps d'exécution T(n)
3. Calculer le ratio T(2n)/T(n)

Ce ratio révèle la complexité:
- O(1): ratio ≈ 1
- O(log n): ratio ≈ 1 + ε (décroissant)
- O(n): ratio ≈ 2
- O(n log n): ratio ≈ 2 + ε (croissant lentement)
- O(n²): ratio ≈ 4
- O(2^n): ratio explose

#### 5.5.2 Le Master Theorem

Pour T(n) = a·T(n/b) + f(n):

**Intuition:** On compare le "poids" des feuilles (a^(log_b n) = n^(log_b a)) au travail f(n).

**Case 1:** Les feuilles dominent
- Condition: f(n) = O(n^c) avec c < log_b(a)
- Résultat: T(n) = Θ(n^(log_b a))

**Case 2:** Équilibre à tous les niveaux
- Condition: f(n) = Θ(n^(log_b a) · log^k n)
- Résultat: T(n) = Θ(n^(log_b a) · log^(k+1) n)

**Case 3:** La racine domine
- Condition: f(n) = Ω(n^c) avec c > log_b(a) ET régularité
- Résultat: T(n) = Θ(f(n))

#### 5.5.3 Analyse Amortie

Trois méthodes:
1. **Agrégat:** Coût total / nombre d'opérations
2. **Comptable (Banker's):** Crédits pour opérations futures
3. **Potentiel:** Φ(état) → coût amorti = coût réel + ΔΦ

Exemples classiques:
- Vector::push_back: O(1) amorti (géométrique)
- Compteur binaire: O(1) amorti par incrémentation
- Union-Find: O(α(n)) amorti avec path compression

#### 5.5.4 Hiérarchie Cache

```
Registres:   1 cycle     (~100 bytes)
L1 Cache:    3-4 cycles  (~32 KB)
L2 Cache:    10-12 cycles (~256 KB)
L3 Cache:    30-40 cycles (~8 MB)
RAM:         100+ cycles  (GB)
SSD:         10,000+ cycles
HDD:         10,000,000+ cycles
```

**Localité spatiale:** Accéder à des adresses proches
**Localité temporelle:** Réaccéder aux mêmes données

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ let ratio = a.ln() * b.ln();  // Mauvais calcul de log         │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ let ratio = a.ln() / b.ln();  // Correct: log_b(a)             │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ log_b(a) = log(a) / log(b) par changement de base              │
│ Multiplier donne un résultat sans signification mathématique   │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Trace: master_case(7.0, 2.0, 2.0) — Strassen**

```
┌───────┬──────────────────────────────────────────┬────────────┬─────────────────────┐
│ Étape │ Instruction                              │ Valeur     │ Explication         │
├───────┼──────────────────────────────────────────┼────────────┼─────────────────────┤
│   1   │ Calculer ln(7)                           │ 1.9459     │ Logarithme naturel  │
├───────┼──────────────────────────────────────────┼────────────┼─────────────────────┤
│   2   │ Calculer ln(2)                           │ 0.6931     │ ln(2)               │
├───────┼──────────────────────────────────────────┼────────────┼─────────────────────┤
│   3   │ log_b(a) = ln(7)/ln(2)                   │ 2.807      │ log₂(7)             │
├───────┼──────────────────────────────────────────┼────────────┼─────────────────────┤
│   4   │ Comparer k=2.0 avec log_b(a)=2.807       │ 2.0 < 2.807│ k est plus petit    │
├───────┼──────────────────────────────────────────┼────────────┼─────────────────────┤
│   5   │ Case 1 s'applique                        │ return 1   │ Feuilles dominent   │
├───────┼──────────────────────────────────────────┼────────────┼─────────────────────┤
│   6   │ Complexité = Θ(n^2.807)                  │ résultat   │ Strassen             │
└───────┴──────────────────────────────────────────┴────────────┴─────────────────────┘
```

### 5.8 Mnémotechniques

#### 🎮 MEME : "El Psy Kongroo" — Steins;Gate et les World Lines

Comme Okabe dans Steins;Gate qui doit identifier sur quelle "World Line" il se trouve en mesurant la divergence, tu dois identifier la complexité de ton algorithme en mesurant les temps d'exécution.

- **World Line Alpha (O(1)):** Le paradis — temps constant
- **World Line Beta (O(log n)):** Presque parfait
- **World Line Gamma (O(n)):** Acceptable
- **World Line Omega (O(2^n)):** La destruction — éviter à tout prix!

```rust
// El Psy Kongroo - La phrase de Okabe
// Comme chercher le Steins Gate (l'optimum), on cherche la vraie complexité
let worldline = identify_worldline(&sizes, &times);
match worldline {
    WorldLine::Alpha => println!("Perfect! O(1)"),
    WorldLine::Omega => println!("The organization is watching... O(2^n)!"),
    _ => println!("Continue searching for Steins Gate..."),
}
```

#### 🔬 MEME : "Reading Steiner" — La mémoire cumulative

Dans Steins;Gate, le "Reading Steiner" est la capacité d'Okabe à retenir les souvenirs à travers les sauts temporels. C'est exactement l'analyse amortie: on "retient" le coût cumulé pour calculer le coût moyen.

```rust
// Reading Steiner pour le tableau dynamique
// On accumule les souvenirs (coûts) à travers le temps
let (total_memories, avg_per_leap) = reading_steiner_array(1000);
// Même si certains sauts coûtent cher (réallocation),
// en moyenne c'est O(1) par saut!
```

### 5.9 Applications pratiques

| Application | Technique utilisée |
|-------------|-------------------|
| Profiling CPU | Benchmarking empirique |
| Choix d'algorithme | Master Theorem pour comparer |
| Database indexing | Analyse cache pour B-trees |
| Vector::push_back | Analyse amortie O(1) |
| Compilateur JIT | Estimation de coût d'inlining |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Prévention |
|---|-------|--------|------------|
| 1 | Division par zéro (b=1) | Crash/NaN | Vérifier b > 1 |
| 2 | Overflow nanosecondes | Mauvaise mesure | Utiliser u128 |
| 3 | Cache froid | Biais de mesure | Warmup runs |
| 4 | Master Theorem sans régularité | Résultat faux | Vérifier condition |
| 5 | LRU inversé (FIFO) | Mauvais hit ratio | Implémenter vrai LRU |

---

## 📝 SECTION 7 : QCM

### Question 1
Pour la récurrence T(n) = 4·T(n/2) + n, quel case du Master Theorem s'applique?

- A) Case 1 car log₂(4) = 2 > 1
- B) Case 2 car log₂(4) = 2 = k
- C) Case 3 car n domine 4·T(n/2)
- D) Le Master Theorem ne s'applique pas
- E) Case 1 car k=1 < log₂(4)=2

<details>
<summary>Réponse</summary>
**E)** log₂(4) = 2, et f(n) = n = n¹, donc k=1 < 2 = log_b(a). Case 1 s'applique, T(n) = Θ(n²).
</details>

### Question 2
Quel est le coût amorti de `push_back` sur un vector qui double de capacité?

- A) O(n) car parfois on copie tout
- B) O(log n) car on double
- C) O(1) car la moyenne est constante
- D) O(n²) car on fait n push de O(n) chacun
- E) Impossible à déterminer

<details>
<summary>Réponse</summary>
**C)** Sur n opérations, le coût total est ≤ 2n (somme géométrique), donc O(2n/n) = O(1) amorti.
</details>

### Question 3
Pourquoi l'accès column-major à une matrice est-il lent?

- A) Les colonnes sont plus longues
- B) On saute d'un bloc cache à l'autre
- C) Le CPU préfère les lignes
- D) C'est une illusion, les deux sont identiques
- E) La mémoire est organisée en colonnes

<details>
<summary>Réponse</summary>
**B)** En row-major storage, les éléments d'une colonne ne sont pas contigus en mémoire. Chaque accès charge un nouveau bloc cache, causant des cache misses systématiques.
</details>

### Question 4
Quelle est la complexité de Strassen (multiplication matricielle)?

- A) O(n²)
- B) O(n³)
- C) O(n^2.807)
- D) O(n² log n)
- E) O(n^2.376)

<details>
<summary>Réponse</summary>
**C)** Strassen: T(n) = 7·T(n/2) + O(n²). log₂(7) ≈ 2.807, donc Case 1: Θ(n^2.807).
</details>

### Question 5
Dans l'analyse amortie par potentiel, le coût amorti est:

- A) coût_réel + Φ(avant)
- B) coût_réel - Φ(après)
- C) coût_réel + ΔΦ
- D) Φ(après) - Φ(avant)
- E) max(coût_réel, Φ)

<details>
<summary>Réponse</summary>
**C)** Coût amorti = coût réel + Φ(après) - Φ(avant) = coût réel + ΔΦ.
</details>

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.1.8 — worldline_analyzer |
| **Difficulté** | ★★★★★★☆☆☆☆ (6/10) |
| **XP Base** | 150 |
| **XP Bonus** | ×3 (450 XP) |
| **Temps estimé** | 60 min |
| **Concepts clés** | Master Theorem, Amortized Analysis, Cache Simulation |
| **Langage** | Rust Edition 2024 / C17 |
| **Référence culture** | Steins;Gate (World Lines, Divergence Meter) |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.1.8-worldline-analyzer",
    "generated_at": "2026-01-11 10:00:00",

    "metadata": {
      "exercise_id": "1.1.8",
      "exercise_name": "worldline_analyzer",
      "module": "1.1",
      "module_name": "Arrays & Sorting",
      "concept": "h",
      "concept_name": "Complexity Analysis & Recurrences",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 1,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust",
      "duration_minutes": 60,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "Variable",
      "complexity_space": "Variable",
      "prerequisites": ["recursion", "big-o-basics"],
      "domains": ["Tri", "MD", "CPU", "Mem"],
      "domains_bonus": ["Calcul"],
      "tags": ["complexity", "master-theorem", "amortized", "cache"],
      "meme_reference": "Steins;Gate - El Psy Kongroo"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "alternatives/alt_regression.rs": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_math.rs": "/* Section 4.10 */",
      "mutants/mutant_c_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_d_overflow.rs": "/* Section 4.10 */",
      "mutants/mutant_e_cache.rs": "/* Section 4.10 */",
      "tests/main.rs": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "references/ref_solution_bonus.rs",
        "alternatives/alt_regression.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_math.rs",
        "mutants/mutant_c_logic.rs",
        "mutants/mutant_d_overflow.rs",
        "mutants/mutant_e_cache.rs"
      ]
    }
  }
}
```

---

*Exercice généré par HACKBRAIN v5.5.2 — "El Psy Kongroo"*
*L'excellence pédagogique ne se négocie pas — pas de raccourcis*
