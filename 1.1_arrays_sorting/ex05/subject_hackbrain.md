<thinking>
## Analyse du Concept
- Concept : Prefix Sums & Difference Arrays (sommes préfixes et tableaux de différences)
- Phase demandée : 1
- Adapté ? OUI — Concepts intermédiaires parfaits pour Phase 1. Transforme O(n) par requête en O(1) avec prétraitement O(n).

## Combo Base + Bonus
- Exercice de base : Implémenter les 3 structures (PrefixSum, PrefixSum2D, DifferenceArray) et 8 fonctions utilitaires
- Bonus : Optimiser max_sum_rectangle avec compression de coordonnées et Kadane 2D optimisé
- Palier bonus : 🔥 Avancé
- Progression logique ? OUI — Base = structures + algos classiques, Bonus = optimisation avancée

## Prérequis & Difficulté
- Prérequis réels : Tableaux, boucles, indexation, sommes cumulées basiques
- Difficulté estimée : 5/10
- Cohérent avec phase ? OUI

## Aspect Fun/Culture
- Contexte choisi : Breaking Bad - Walter White et sa comptabilité de l'empire
- MEME mnémotechnique : "I am the one who counts" / "Say my sum"
- Pourquoi c'est fun : L'analogie parfaite entre un grand livre comptable criminel et les prefix sums (running totals). Les difference arrays = tracking des variations du business. 2D = le hangar rempli de billets.

## Scénarios d'Échec (5 mutants)
1. Mutant A (Boundary) : Off-by-one dans range_sum — utilise prefix[right] au lieu de prefix[right+1]
2. Mutant B (Safety) : Ne vérifie pas si left > right dans range_sum — crash ou résultat incorrect
3. Mutant C (Overflow) : Utilise i32 au lieu de i64 pour les grandes sommes — overflow silencieux
4. Mutant D (Logic) : Oublie le -prefix[r1][c1] dans la formule 2D — résultat doublé
5. Mutant E (Return) : range_add ne gère pas le cas r+1 >= n — corruption mémoire

## Verdict
VALIDE — Analogie Breaking Bad excellente, exercice riche couvrant 3 structures + 8 fonctions
Note créativité : 97/100
</thinking>

---

# Exercice 1.1.5 : heisenberg_ledger

**Module :**
1.1 — Arrays & Sorting

**Concept :**
k — Prefix Sums & Difference Arrays

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (concepts prefix sums 1D + 2D + difference arrays + applications)

**Langages :**
Rust Edition 2024 / C17

**Prérequis :**
- Manipulation de tableaux et indexation
- Boucles et accumulateurs
- Compréhension des indices inclusifs/exclusifs

**Domaines :**
Struct, MD, DP

**Durée estimée :**
60 min

**XP Base :**
150

**Complexité :**
T2 O(n) construction × S2 O(n) stockage, T1 O(1) requêtes

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- **Rust :** `src/lib.rs`, `Cargo.toml`
- **C :** `heisenberg_ledger.c`, `heisenberg_ledger.h`

**Fonctions autorisées :**
- Rust : std (Vec, HashMap, collections)
- C : malloc, free, memset, memcpy

**Fonctions interdites :**
- Bibliothèques mathématiques avancées externes

---

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🧪 BREAKING BAD — "I am the one who counts"**

Walter White a un problème. Son empire de méthamphétamine génère tellement d'argent qu'il ne peut plus le compter manuellement. Il a besoin d'un système de comptabilité... créatif.

Tu vas créer le **Heisenberg Ledger** — un système de tracking financier pour suivre les revenus de l'empire à travers le temps et l'espace.

**Le problème de Walt :**
- "Combien j'ai gagné entre l'épisode 3 et l'épisode 7 ?" → **Range Query** (O(1) avec prefix sum)
- "Jesse a perdu 10% sur chaque batch de la semaine 2 à 5" → **Range Update** (O(1) avec difference array)
- "Le hangar avec les palettes de billets — combien dans ce coin ?" → **2D Range Query**

**L'insight de Heisenberg :**
Au lieu de recompter tout à chaque fois (O(n)), on maintient un **running total** (comme un grand livre comptable). Chaque entrée stocke la somme CUMULATIVE depuis le début.

```
Revenus par batch :     [50K, 100K, 75K, 200K, 150K]
                            ↓ PRÉTRAITEMENT ↓
Prefix Sum (cumul) :    [0, 50K, 150K, 225K, 425K, 575K]

"Combien de batch 1 à batch 3 ?"
→ prefix[4] - prefix[1] = 425K - 50K = 375K
→ Temps : O(1) ! 🎯
```

**Ta mission :**

Implémenter le système complet de comptabilité de l'empire Heisenberg :

---

#### 1.2.2 Version Académique

**Prefix Sums (Sommes Préfixes) :**

Une technique de prétraitement qui transforme un tableau en tableau cumulatif, permettant des requêtes de somme de plage en O(1) après un prétraitement O(n).

**Formule 1D :**
```
prefix[0] = 0
prefix[i] = prefix[i-1] + arr[i-1]

range_sum(l, r) = prefix[r+1] - prefix[l]
```

**Difference Arrays (Tableaux de Différences) :**

Structure duale des prefix sums. Permet des mises à jour de plage en O(1) avec reconstruction finale en O(n).

```
diff[0] = arr[0]
diff[i] = arr[i] - arr[i-1]

range_add(l, r, val) :
    diff[l] += val
    diff[r+1] -= val (si r+1 < n)

build() : calcule prefix sum de diff pour reconstruire
```

**Prefix Sum 2D :**

Extension au cas matriciel pour requêtes de sous-matrices en O(1).

```
prefix[i][j] = matrix[i-1][j-1] + prefix[i-1][j] + prefix[i][j-1] - prefix[i-1][j-1]

range_sum(r1,c1,r2,c2) = prefix[r2+1][c2+1] - prefix[r1][c2+1] - prefix[r2+1][c1] + prefix[r1][c1]
```

---

### 1.3 Prototypes

#### Rust

```rust
pub mod heisenberg_ledger {
    use std::collections::HashMap;

    /// 1D Prefix Sum — "The Ledger"
    pub struct EmpireLedger {
        prefix: Vec<i64>,
    }

    impl EmpireLedger {
        /// Construit le grand livre depuis les revenus par batch - O(n)
        pub fn new(revenues: &[i32]) -> Self;

        /// Requête : somme des revenus de left à right (inclusif) - O(1)
        pub fn range_sum(&self, left: usize, right: usize) -> i64;

        /// Requête : total des k premiers batchs - O(1)
        pub fn sum_first_k(&self, k: usize) -> i64;
    }

    /// 2D Prefix Sum — "The Money Warehouse"
    pub struct WarehouseGrid {
        prefix: Vec<Vec<i64>>,
    }

    impl WarehouseGrid {
        /// Construit la grille de tracking - O(n*m)
        pub fn new(stacks: &[Vec<i32>]) -> Self;

        /// Requête : somme du sous-rectangle - O(1)
        pub fn range_sum(&self, r1: usize, c1: usize, r2: usize, c2: usize) -> i64;
    }

    /// Difference Array — "The Delta Tracker"
    pub struct DeltaTracker {
        diff: Vec<i64>,
    }

    impl DeltaTracker {
        /// Crée depuis un tableau existant
        pub fn from_array(arr: &[i32]) -> Self;

        /// Crée un tracker vide de taille n
        pub fn with_size(n: usize) -> Self;

        /// Ajoute une valeur à la plage [left, right] - O(1)
        pub fn range_add(&mut self, left: usize, right: usize, value: i64);

        /// Reconstruit le tableau final après toutes les opérations - O(n)
        pub fn build(&self) -> Vec<i64>;
    }

    // ═══════════════════════════════════════════════════════
    // FONCTIONS UTILITAIRES — "The Heisenberg Toolkit"
    // ═══════════════════════════════════════════════════════

    /// Compte les sous-tableaux dont la somme égale k
    /// "Combien de périodes ont généré exactement k dollars ?"
    pub fn count_sum_periods(revenues: &[i32], k: i32) -> i64;

    /// Trouve l'index pivot où somme gauche == somme droite
    /// "Le point d'équilibre de l'empire"
    pub fn find_balance_point(arr: &[i32]) -> Option<usize>;

    /// Maximum subarray sum (Kadane) — "Best streak"
    /// "Quelle a été notre meilleure série consécutive ?"
    pub fn best_streak(revenues: &[i32]) -> i64;

    /// Produit sauf soi-même (sans division)
    /// "Impact de retirer chaque distributeur"
    pub fn impact_without(arr: &[i32]) -> Vec<i64>;

    /// Prefix XOR et requêtes
    /// "Encryption keys par période"
    pub fn build_xor_keys(arr: &[i32]) -> Vec<i32>;
    pub fn range_xor(keys: &[i32], left: usize, right: usize) -> i32;

    /// Trouve tous les points d'équilibre
    pub fn all_balance_points(arr: &[i32]) -> Vec<usize>;

    /// Maximum sum rectangle — "Best warehouse section"
    pub fn best_warehouse_section(grid: &[Vec<i32>]) -> i64;
}
```

#### C

```c
#ifndef HEISENBERG_LEDGER_H
#define HEISENBERG_LEDGER_H

#include <stddef.h>
#include <stdint.h>

// ═══════════════════════════════════════════════════════════════
// STRUCTURES — "The Empire's Data"
// ═══════════════════════════════════════════════════════════════

// 1D Prefix Sum — The Ledger
typedef struct s_empire_ledger {
    int64_t *prefix;
    size_t   size;
} t_empire_ledger;

// 2D Prefix Sum — The Warehouse Grid
typedef struct s_warehouse_grid {
    int64_t **prefix;
    size_t    rows;
    size_t    cols;
} t_warehouse_grid;

// Difference Array — The Delta Tracker
typedef struct s_delta_tracker {
    int64_t *diff;
    size_t   size;
} t_delta_tracker;

// ═══════════════════════════════════════════════════════════════
// CONSTRUCTEURS & DESTRUCTEURS
// ═══════════════════════════════════════════════════════════════

t_empire_ledger   *empire_ledger_new(const int *revenues, size_t n);
void               empire_ledger_free(t_empire_ledger *ledger);

t_warehouse_grid  *warehouse_grid_new(const int **stacks, size_t rows, size_t cols);
void               warehouse_grid_free(t_warehouse_grid *grid);

t_delta_tracker   *delta_tracker_from_array(const int *arr, size_t n);
t_delta_tracker   *delta_tracker_with_size(size_t n);
void               delta_tracker_free(t_delta_tracker *tracker);

// ═══════════════════════════════════════════════════════════════
// OPÉRATIONS — "Running the Empire"
// ═══════════════════════════════════════════════════════════════

// Ledger operations
int64_t ledger_range_sum(const t_empire_ledger *ledger, size_t left, size_t right);
int64_t ledger_sum_first_k(const t_empire_ledger *ledger, size_t k);

// Warehouse operations
int64_t warehouse_range_sum(const t_warehouse_grid *grid,
                           size_t r1, size_t c1, size_t r2, size_t c2);

// Delta operations
void    delta_range_add(t_delta_tracker *tracker, size_t left, size_t right, int64_t value);
int64_t *delta_build(const t_delta_tracker *tracker);

// ═══════════════════════════════════════════════════════════════
// FONCTIONS UTILITAIRES
// ═══════════════════════════════════════════════════════════════

int64_t count_sum_periods(const int *revenues, size_t n, int k);
ssize_t find_balance_point(const int *arr, size_t n);  // -1 si non trouvé
int64_t best_streak(const int *revenues, size_t n);
int64_t *impact_without(const int *arr, size_t n);
int    *build_xor_keys(const int *arr, size_t n);
int     range_xor(const int *keys, size_t left, size_t right);
size_t *all_balance_points(const int *arr, size_t n, size_t *out_count);
int64_t best_warehouse_section(const int **grid, size_t rows, size_t cols);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Faits Fascinants

**💰 L'origine comptable :**
Les prefix sums sont littéralement ce que fait un comptable depuis des millénaires ! Le "running balance" d'un grand livre comptable est une prefix sum. Quand tu regardes ton relevé bancaire, chaque ligne montre le solde CUMULATIF — c'est une prefix sum.

**🎮 Usage dans les jeux :**
Les MMORPGs utilisent des prefix sums pour calculer l'XP cumulé, les dégâts totaux sur une période, ou les statistiques de session. "Damage done this raid" = range query sur l'historique.

**📊 Histogrammes cumulatifs :**
En statistiques, le CDF (Cumulative Distribution Function) EST une prefix sum normalisée. C'est fondamental en probabilités.

### 2.2 Propriété Magique

```
            DUALITÉ PREFIX ↔ DIFFERENCE

Prefix Sum :    Convertit DIFFÉRENCES en VALEURS
Difference :    Convertit VALEURS en DIFFÉRENCES

arr       →(diff)→  diff_arr  →(prefix)→  arr
[3,5,8,2] →(diff)→ [3,2,3,-6] →(prefix)→ [3,5,8,2]

C'est une INVOLUTION ! (f(f(x)) = x)
```

### 2.5 Dans la Vraie Vie

| Métier | Utilisation |
|--------|-------------|
| **Data Scientist** | Calcul de métriques cumulatives, rolling sums, fenêtres glissantes |
| **Quant Finance** | Running P&L, cumulative returns, drawdown calculation |
| **Game Developer** | Damage meters, XP tracking, score history |
| **DevOps** | Cumulative request counts, bandwidth usage over time |
| **Database Engineer** | Fenêtrage SQL (window functions), OLAP cubes |
| **Image Processing** | Integral images (Summed Area Tables) pour blur/detection |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
heisenberg_ledger.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 12 tests
test test_empire_ledger ... ok
test test_warehouse_grid ... ok
test test_delta_tracker ... ok
test test_count_sum_periods ... ok
test test_balance_point ... ok
test test_best_streak ... ok
test test_impact_without ... ok
test test_xor_keys ... ok
test test_all_balance_points ... ok
test test_best_warehouse ... ok
test test_edge_empty ... ok
test test_edge_single ... ok

test result: ok. 12 passed; 0 failed
```

---

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n*m*min(n,m)) pour max_sum_rectangle optimisé

**Space Complexity attendue :**
O(n) auxiliaire

**Domaines Bonus :**
`DP, Calcul`

#### 3.1.1 Consigne Bonus

**🧪 BREAKING BAD — "The Perfect Cook"**

Heisenberg veut optimiser encore plus. Le warehouse a des millions de stacks, et certaines sections ont des valeurs négatives (dettes, pertes).

**Ta mission bonus :**

1. **`best_warehouse_section_optimized`** — Trouver le sous-rectangle de somme maximale en O(n²m) au lieu de O(n²m²)

2. **`sparse_delta_tracker`** — DifferenceArray optimisé pour updates sparse (HashMaps au lieu de tableaux)

3. **`streaming_prefix`** — PrefixSum qui supporte les insertions en fin en O(1) amorti

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  1 ≤ n, m ≤ 10⁴                         │
│  Temps : O(n² × m) pour rectangle       │
│  Espace : O(n) auxiliaire               │
│  Pas de récursion pour streaming        │
└─────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
/// Rectangle de somme max optimisé avec Kadane 2D
pub fn best_warehouse_section_optimized(grid: &[Vec<i32>]) -> (i64, usize, usize, usize, usize);

/// Difference array sparse pour grandes plages
pub struct SparseDeltaTracker {
    deltas: HashMap<usize, i64>,
    size: usize,
}

impl SparseDeltaTracker {
    pub fn new(size: usize) -> Self;
    pub fn range_add(&mut self, left: usize, right: usize, value: i64);
    pub fn build(&self) -> Vec<i64>;
}

/// Prefix sum avec streaming
pub struct StreamingLedger {
    prefix: Vec<i64>,
}

impl StreamingLedger {
    pub fn new() -> Self;
    pub fn push(&mut self, value: i32);
    pub fn range_sum(&self, left: usize, right: usize) -> i64;
    pub fn total(&self) -> i64;
}
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Rectangle max | O(n²m²) brute | O(n²m) avec Kadane |
| Delta storage | Dense O(n) | Sparse O(updates) |
| Prefix updates | Reconstruction totale | Insertion O(1) amorti |
| Retour rectangle | Somme seulement | Somme + coordonnées |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected Output | Points |
|------|-------|-----------------|--------|
| ledger_basic | `[1,2,3,4,5]`, range(0,4) | `15` | 2 |
| ledger_partial | `[1,2,3,4,5]`, range(1,3) | `9` | 2 |
| ledger_single | `[1,2,3,4,5]`, range(2,2) | `3` | 2 |
| ledger_first_k | `[1,2,3,4,5]`, k=3 | `6` | 2 |
| warehouse_full | 3x3 matrix, range(0,0,2,2) | `45` | 3 |
| warehouse_sub | 3x3 matrix, range(1,1,2,2) | `28` | 3 |
| warehouse_single | 3x3 matrix, range(0,0,0,0) | `1` | 2 |
| delta_basic | 5 zeros, add(1,3,10) | `[0,10,10,10,0]` | 3 |
| delta_overlap | 5 zeros, add(1,3,10)+add(2,4,5) | `[0,10,15,15,5]` | 3 |
| count_sum_2 | `[1,1,1]`, k=2 | `2` | 2 |
| count_sum_3 | `[1,2,3]`, k=3 | `2` | 2 |
| balance_found | `[1,7,3,6,5,6]` | `Some(3)` | 2 |
| balance_none | `[1,2,3]` | `None` | 2 |
| kadane_mixed | `[-2,1,-3,4,-1,2,1,-5,4]` | `6` | 3 |
| kadane_negative | `[-1]` | `-1` | 2 |
| kadane_positive | `[5,4,-1,7,8]` | `23` | 2 |
| product_basic | `[1,2,3,4]` | `[24,12,8,6]` | 3 |
| product_zero | `[-1,1,0,-3,3]` | `[0,0,9,0,0]` | 3 |
| xor_range | `[1,3,4,8]`, range(0,1) | `2` | 2 |
| xor_full | `[1,3,4,8]`, range(0,3) | `14` | 2 |
| rectangle_max | 4x5 matrix | `29` | 5 |
| edge_empty | `[]` | handle gracefully | 2 |
| edge_large | 10⁵ elements | completes < 1s | 3 |

### 4.2 main.rs de test

```rust
use heisenberg_ledger::*;

fn main() {
    println!("=== HEISENBERG LEDGER TESTS ===\n");

    // Test 1: Empire Ledger
    let revenues = vec![50, 100, 75, 200, 150];
    let ledger = EmpireLedger::new(&revenues);

    assert_eq!(ledger.range_sum(0, 4), 575);
    assert_eq!(ledger.range_sum(1, 3), 375);
    assert_eq!(ledger.sum_first_k(3), 225);
    println!("[OK] Empire Ledger");

    // Test 2: Warehouse Grid
    let stacks = vec![
        vec![1, 2, 3],
        vec![4, 5, 6],
        vec![7, 8, 9],
    ];
    let warehouse = WarehouseGrid::new(&stacks);

    assert_eq!(warehouse.range_sum(0, 0, 2, 2), 45);
    assert_eq!(warehouse.range_sum(1, 1, 2, 2), 28);
    println!("[OK] Warehouse Grid");

    // Test 3: Delta Tracker
    let mut tracker = DeltaTracker::with_size(5);
    tracker.range_add(1, 3, 10);
    tracker.range_add(2, 4, 5);

    assert_eq!(tracker.build(), vec![0, 10, 15, 15, 5]);
    println!("[OK] Delta Tracker");

    // Test 4: Count Sum Periods
    assert_eq!(count_sum_periods(&[1, 1, 1], 2), 2);
    assert_eq!(count_sum_periods(&[1, 2, 3], 3), 2);
    println!("[OK] Count Sum Periods");

    // Test 5: Balance Point
    assert_eq!(find_balance_point(&[1, 7, 3, 6, 5, 6]), Some(3));
    assert_eq!(find_balance_point(&[1, 2, 3]), None);
    println!("[OK] Balance Point");

    // Test 6: Kadane (Best Streak)
    assert_eq!(best_streak(&[-2, 1, -3, 4, -1, 2, 1, -5, 4]), 6);
    assert_eq!(best_streak(&[-1]), -1);
    println!("[OK] Best Streak (Kadane)");

    // Test 7: Product Except Self
    assert_eq!(impact_without(&[1, 2, 3, 4]), vec![24, 12, 8, 6]);
    assert_eq!(impact_without(&[-1, 1, 0, -3, 3]), vec![0, 0, 9, 0, 0]);
    println!("[OK] Impact Without");

    // Test 8: XOR Range
    let keys = build_xor_keys(&[1, 3, 4, 8]);
    assert_eq!(range_xor(&keys, 0, 1), 2);
    assert_eq!(range_xor(&keys, 0, 3), 14);
    println!("[OK] XOR Range");

    // Test 9: Max Rectangle
    let grid = vec![
        vec![1, 2, -1, -4, -20],
        vec![-8, -3, 4, 2, 1],
        vec![3, 8, 10, 1, 3],
        vec![-4, -1, 1, 7, -6],
    ];
    assert_eq!(best_warehouse_section(&grid), 29);
    println!("[OK] Best Warehouse Section");

    println!("\n=== ALL TESTS PASSED ===");
    println!("Say my name. You're Heisenberg. You're goddamn right.");
}
```

### 4.3 Solution de référence (Rust)

```rust
pub mod heisenberg_ledger {
    use std::collections::HashMap;

    // ═══════════════════════════════════════════════════════════
    // EMPIRE LEDGER — 1D PREFIX SUM
    // ═══════════════════════════════════════════════════════════

    pub struct EmpireLedger {
        prefix: Vec<i64>,
    }

    impl EmpireLedger {
        pub fn new(revenues: &[i32]) -> Self {
            let n = revenues.len();
            let mut prefix = vec![0i64; n + 1];
            for i in 0..n {
                prefix[i + 1] = prefix[i] + revenues[i] as i64;
            }
            EmpireLedger { prefix }
        }

        pub fn range_sum(&self, left: usize, right: usize) -> i64 {
            if left > right || right >= self.prefix.len() - 1 {
                return 0;
            }
            self.prefix[right + 1] - self.prefix[left]
        }

        pub fn sum_first_k(&self, k: usize) -> i64 {
            if k == 0 || k > self.prefix.len() - 1 {
                return 0;
            }
            self.prefix[k]
        }
    }

    // ═══════════════════════════════════════════════════════════
    // WAREHOUSE GRID — 2D PREFIX SUM
    // ═══════════════════════════════════════════════════════════

    pub struct WarehouseGrid {
        prefix: Vec<Vec<i64>>,
    }

    impl WarehouseGrid {
        pub fn new(stacks: &[Vec<i32>]) -> Self {
            if stacks.is_empty() || stacks[0].is_empty() {
                return WarehouseGrid { prefix: vec![vec![0]] };
            }

            let rows = stacks.len();
            let cols = stacks[0].len();
            let mut prefix = vec![vec![0i64; cols + 1]; rows + 1];

            for i in 1..=rows {
                for j in 1..=cols {
                    prefix[i][j] = stacks[i - 1][j - 1] as i64
                        + prefix[i - 1][j]
                        + prefix[i][j - 1]
                        - prefix[i - 1][j - 1];
                }
            }

            WarehouseGrid { prefix }
        }

        pub fn range_sum(&self, r1: usize, c1: usize, r2: usize, c2: usize) -> i64 {
            if r1 > r2 || c1 > c2 {
                return 0;
            }
            if r2 >= self.prefix.len() - 1 || c2 >= self.prefix[0].len() - 1 {
                return 0;
            }

            self.prefix[r2 + 1][c2 + 1]
                - self.prefix[r1][c2 + 1]
                - self.prefix[r2 + 1][c1]
                + self.prefix[r1][c1]
        }
    }

    // ═══════════════════════════════════════════════════════════
    // DELTA TRACKER — DIFFERENCE ARRAY
    // ═══════════════════════════════════════════════════════════

    pub struct DeltaTracker {
        diff: Vec<i64>,
    }

    impl DeltaTracker {
        pub fn from_array(arr: &[i32]) -> Self {
            let n = arr.len();
            if n == 0 {
                return DeltaTracker { diff: vec![] };
            }

            let mut diff = vec![0i64; n];
            diff[0] = arr[0] as i64;
            for i in 1..n {
                diff[i] = arr[i] as i64 - arr[i - 1] as i64;
            }

            DeltaTracker { diff }
        }

        pub fn with_size(n: usize) -> Self {
            DeltaTracker { diff: vec![0; n] }
        }

        pub fn range_add(&mut self, left: usize, right: usize, value: i64) {
            if left >= self.diff.len() || left > right {
                return;
            }

            self.diff[left] += value;
            if right + 1 < self.diff.len() {
                self.diff[right + 1] -= value;
            }
        }

        pub fn build(&self) -> Vec<i64> {
            let n = self.diff.len();
            if n == 0 {
                return vec![];
            }

            let mut result = vec![0i64; n];
            result[0] = self.diff[0];
            for i in 1..n {
                result[i] = result[i - 1] + self.diff[i];
            }

            result
        }
    }

    // ═══════════════════════════════════════════════════════════
    // UTILITAIRES — THE HEISENBERG TOOLKIT
    // ═══════════════════════════════════════════════════════════

    /// Compte les sous-tableaux dont la somme égale k
    pub fn count_sum_periods(revenues: &[i32], k: i32) -> i64 {
        let mut count = 0i64;
        let mut prefix_sum = 0i64;
        let mut freq: HashMap<i64, i64> = HashMap::new();
        freq.insert(0, 1);

        for &rev in revenues {
            prefix_sum += rev as i64;
            let target = prefix_sum - k as i64;
            if let Some(&c) = freq.get(&target) {
                count += c;
            }
            *freq.entry(prefix_sum).or_insert(0) += 1;
        }

        count
    }

    /// Trouve l'index pivot où somme gauche == somme droite
    pub fn find_balance_point(arr: &[i32]) -> Option<usize> {
        if arr.is_empty() {
            return None;
        }

        let total: i64 = arr.iter().map(|&x| x as i64).sum();
        let mut left_sum = 0i64;

        for (i, &val) in arr.iter().enumerate() {
            let right_sum = total - left_sum - val as i64;
            if left_sum == right_sum {
                return Some(i);
            }
            left_sum += val as i64;
        }

        None
    }

    /// Maximum subarray sum (Kadane's algorithm)
    pub fn best_streak(revenues: &[i32]) -> i64 {
        if revenues.is_empty() {
            return 0;
        }

        let mut max_ending_here = revenues[0] as i64;
        let mut max_so_far = revenues[0] as i64;

        for &rev in revenues.iter().skip(1) {
            max_ending_here = (rev as i64).max(max_ending_here + rev as i64);
            max_so_far = max_so_far.max(max_ending_here);
        }

        max_so_far
    }

    /// Produit sauf soi-même (sans division)
    pub fn impact_without(arr: &[i32]) -> Vec<i64> {
        let n = arr.len();
        if n == 0 {
            return vec![];
        }

        let mut result = vec![1i64; n];

        // Left products
        let mut left = 1i64;
        for i in 0..n {
            result[i] = left;
            left *= arr[i] as i64;
        }

        // Right products
        let mut right = 1i64;
        for i in (0..n).rev() {
            result[i] *= right;
            right *= arr[i] as i64;
        }

        result
    }

    /// Build XOR prefix array
    pub fn build_xor_keys(arr: &[i32]) -> Vec<i32> {
        let n = arr.len();
        let mut prefix = vec![0; n + 1];
        for i in 0..n {
            prefix[i + 1] = prefix[i] ^ arr[i];
        }
        prefix
    }

    /// Range XOR query
    pub fn range_xor(keys: &[i32], left: usize, right: usize) -> i32 {
        if left > right || right >= keys.len() - 1 {
            return 0;
        }
        keys[right + 1] ^ keys[left]
    }

    /// Trouve tous les points d'équilibre
    pub fn all_balance_points(arr: &[i32]) -> Vec<usize> {
        if arr.is_empty() {
            return vec![];
        }

        let total: i64 = arr.iter().map(|&x| x as i64).sum();
        let mut left_sum = 0i64;
        let mut points = vec![];

        for (i, &val) in arr.iter().enumerate() {
            let right_sum = total - left_sum - val as i64;
            if left_sum == right_sum {
                points.push(i);
            }
            left_sum += val as i64;
        }

        points
    }

    /// Maximum sum rectangle using Kadane 2D
    pub fn best_warehouse_section(grid: &[Vec<i32>]) -> i64 {
        if grid.is_empty() || grid[0].is_empty() {
            return 0;
        }

        let rows = grid.len();
        let cols = grid[0].len();
        let mut max_sum = i64::MIN;

        for left in 0..cols {
            let mut temp = vec![0i64; rows];

            for right in left..cols {
                for i in 0..rows {
                    temp[i] += grid[i][right] as i64;
                }

                // Apply Kadane on temp
                let mut current = temp[0];
                let mut best = temp[0];
                for i in 1..rows {
                    current = temp[i].max(current + temp[i]);
                    best = best.max(current);
                }
                max_sum = max_sum.max(best);
            }
        }

        max_sum
    }
}
```

### 4.4 Solutions alternatives acceptées

```rust
// Alternative 1: Prefix sum avec indices 0-based différents
impl EmpireLedger {
    pub fn new_alt(revenues: &[i32]) -> Self {
        // Garde la valeur originale à l'index 0
        let mut prefix: Vec<i64> = revenues.iter().map(|&x| x as i64).collect();
        for i in 1..prefix.len() {
            prefix[i] += prefix[i - 1];
        }
        // Ajuste les requêtes en conséquence
        EmpireLedger { prefix }
    }
}

// Alternative 2: count_sum_periods avec double boucle O(n²)
// Accepté car correct, mais moins efficace
pub fn count_sum_periods_alt(revenues: &[i32], k: i32) -> i64 {
    let n = revenues.len();
    let mut count = 0;
    for i in 0..n {
        let mut sum = 0i64;
        for j in i..n {
            sum += revenues[j] as i64;
            if sum == k as i64 {
                count += 1;
            }
        }
    }
    count
}

// Alternative 3: Kadane avec tracking des indices
pub fn best_streak_with_indices(revenues: &[i32]) -> (i64, usize, usize) {
    if revenues.is_empty() {
        return (0, 0, 0);
    }

    let mut max_sum = revenues[0] as i64;
    let mut current_sum = revenues[0] as i64;
    let mut start = 0;
    let mut end = 0;
    let mut temp_start = 0;

    for i in 1..revenues.len() {
        if current_sum < 0 {
            current_sum = revenues[i] as i64;
            temp_start = i;
        } else {
            current_sum += revenues[i] as i64;
        }

        if current_sum > max_sum {
            max_sum = current_sum;
            start = temp_start;
            end = i;
        }
    }

    (max_sum, start, end)
}
```

### 4.5 Solutions refusées (avec explications)

```rust
// ❌ REFUSÉ: Off-by-one dans range_sum
pub fn range_sum_bad(&self, left: usize, right: usize) -> i64 {
    // BUG: Utilise right au lieu de right + 1
    self.prefix[right] - self.prefix[left]
}
// Retourne une somme incorrecte (manque le dernier élément)

// ❌ REFUSÉ: Formule 2D incomplète
pub fn range_sum_2d_bad(&self, r1: usize, c1: usize, r2: usize, c2: usize) -> i64 {
    // BUG: Oublie le + prefix[r1][c1]
    self.prefix[r2 + 1][c2 + 1]
        - self.prefix[r1][c2 + 1]
        - self.prefix[r2 + 1][c1]
    // Manque: + self.prefix[r1][c1]
}

// ❌ REFUSÉ: range_add sans vérification de borne
pub fn range_add_bad(&mut self, left: usize, right: usize, value: i64) {
    self.diff[left] += value;
    self.diff[right + 1] -= value;  // BUG: Peut dépasser la taille!
}

// ❌ REFUSÉ: i32 overflow dans les grandes sommes
pub fn range_sum_overflow(&self, left: usize, right: usize) -> i32 {
    // BUG: Utilise i32, overflow sur grandes valeurs
    (self.prefix[right + 1] - self.prefix[left]) as i32
}

// ❌ REFUSÉ: product_except_self avec division
pub fn impact_without_division(arr: &[i32]) -> Vec<i64> {
    let total: i64 = arr.iter().map(|&x| x as i64).product();
    arr.iter().map(|&x| total / x as i64).collect()
    // INTERDIT: Division par zéro si arr contient 0
}
```

### 4.6 Solution bonus de référence

```rust
/// Rectangle de somme max avec Kadane 2D optimisé
/// Retourne (somme, r1, c1, r2, c2)
pub fn best_warehouse_section_optimized(grid: &[Vec<i32>]) -> (i64, usize, usize, usize, usize) {
    if grid.is_empty() || grid[0].is_empty() {
        return (0, 0, 0, 0, 0);
    }

    let rows = grid.len();
    let cols = grid[0].len();
    let mut max_sum = i64::MIN;
    let mut coords = (0, 0, 0, 0);

    for left in 0..cols {
        let mut temp = vec![0i64; rows];

        for right in left..cols {
            for i in 0..rows {
                temp[i] += grid[i][right] as i64;
            }

            // Kadane avec tracking
            let (sum, start, end) = kadane_with_indices(&temp);
            if sum > max_sum {
                max_sum = sum;
                coords = (start, left, end, right);
            }
        }
    }

    (max_sum, coords.0, coords.1, coords.2, coords.3)
}

fn kadane_with_indices(arr: &[i64]) -> (i64, usize, usize) {
    let mut max_sum = arr[0];
    let mut current = arr[0];
    let mut start = 0;
    let mut end = 0;
    let mut temp_start = 0;

    for i in 1..arr.len() {
        if current < 0 {
            current = arr[i];
            temp_start = i;
        } else {
            current += arr[i];
        }

        if current > max_sum {
            max_sum = current;
            start = temp_start;
            end = i;
        }
    }

    (max_sum, start, end)
}

/// Sparse Delta Tracker
pub struct SparseDeltaTracker {
    deltas: HashMap<usize, i64>,
    size: usize,
}

impl SparseDeltaTracker {
    pub fn new(size: usize) -> Self {
        SparseDeltaTracker {
            deltas: HashMap::new(),
            size,
        }
    }

    pub fn range_add(&mut self, left: usize, right: usize, value: i64) {
        if left >= self.size || left > right {
            return;
        }
        *self.deltas.entry(left).or_insert(0) += value;
        if right + 1 < self.size {
            *self.deltas.entry(right + 1).or_insert(0) -= value;
        }
    }

    pub fn build(&self) -> Vec<i64> {
        let mut result = vec![0i64; self.size];
        let mut current = 0i64;

        for i in 0..self.size {
            if let Some(&delta) = self.deltas.get(&i) {
                current += delta;
            }
            result[i] = current;
        }

        result
    }
}

/// Streaming Ledger avec insertions O(1) amorti
pub struct StreamingLedger {
    prefix: Vec<i64>,
}

impl StreamingLedger {
    pub fn new() -> Self {
        StreamingLedger { prefix: vec![0] }
    }

    pub fn push(&mut self, value: i32) {
        let last = *self.prefix.last().unwrap();
        self.prefix.push(last + value as i64);
    }

    pub fn range_sum(&self, left: usize, right: usize) -> i64 {
        if left > right || right >= self.prefix.len() - 1 {
            return 0;
        }
        self.prefix[right + 1] - self.prefix[left]
    }

    pub fn total(&self) -> i64 {
        *self.prefix.last().unwrap()
    }

    pub fn len(&self) -> usize {
        self.prefix.len() - 1
    }
}
```

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "heisenberg_ledger",
  "language": "rust",
  "version": "edition_2024",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (prefix sums 1D + 2D + difference arrays + applications)",
  "tags": ["arrays", "prefix_sum", "difference_array", "range_query", "kadane", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "heisenberg_ledger",
    "module": true,
    "components": [
      {
        "name": "EmpireLedger",
        "type": "struct",
        "methods": ["new", "range_sum", "sum_first_k"]
      },
      {
        "name": "WarehouseGrid",
        "type": "struct",
        "methods": ["new", "range_sum"]
      },
      {
        "name": "DeltaTracker",
        "type": "struct",
        "methods": ["from_array", "with_size", "range_add", "build"]
      }
    ],
    "standalone_functions": [
      "count_sum_periods",
      "find_balance_point",
      "best_streak",
      "impact_without",
      "build_xor_keys",
      "range_xor",
      "all_balance_points",
      "best_warehouse_section"
    ]
  },

  "driver": {
    "reference_file": "solutions/ref_heisenberg_ledger.rs",

    "edge_cases": [
      {
        "name": "ledger_basic",
        "construct": ["EmpireLedger", [1, 2, 3, 4, 5]],
        "call": ["range_sum", 0, 4],
        "expected": 15
      },
      {
        "name": "ledger_partial",
        "construct": ["EmpireLedger", [1, 2, 3, 4, 5]],
        "call": ["range_sum", 1, 3],
        "expected": 9
      },
      {
        "name": "ledger_single",
        "construct": ["EmpireLedger", [1, 2, 3, 4, 5]],
        "call": ["range_sum", 2, 2],
        "expected": 3
      },
      {
        "name": "warehouse_full",
        "construct": ["WarehouseGrid", [[1,2,3],[4,5,6],[7,8,9]]],
        "call": ["range_sum", 0, 0, 2, 2],
        "expected": 45
      },
      {
        "name": "warehouse_sub",
        "construct": ["WarehouseGrid", [[1,2,3],[4,5,6],[7,8,9]]],
        "call": ["range_sum", 1, 1, 2, 2],
        "expected": 28
      },
      {
        "name": "delta_basic",
        "construct": ["DeltaTracker", "with_size", 5],
        "calls": [
          ["range_add", 1, 3, 10]
        ],
        "final_call": ["build"],
        "expected": [0, 10, 10, 10, 0]
      },
      {
        "name": "delta_overlap",
        "construct": ["DeltaTracker", "with_size", 5],
        "calls": [
          ["range_add", 1, 3, 10],
          ["range_add", 2, 4, 5]
        ],
        "final_call": ["build"],
        "expected": [0, 10, 15, 15, 5]
      },
      {
        "name": "count_sum_k2",
        "function": "count_sum_periods",
        "args": [[1, 1, 1], 2],
        "expected": 2
      },
      {
        "name": "balance_found",
        "function": "find_balance_point",
        "args": [[1, 7, 3, 6, 5, 6]],
        "expected": {"Some": 3}
      },
      {
        "name": "balance_none",
        "function": "find_balance_point",
        "args": [[1, 2, 3]],
        "expected": "None"
      },
      {
        "name": "kadane_mixed",
        "function": "best_streak",
        "args": [[-2, 1, -3, 4, -1, 2, 1, -5, 4]],
        "expected": 6
      },
      {
        "name": "kadane_all_negative",
        "function": "best_streak",
        "args": [[-1]],
        "expected": -1
      },
      {
        "name": "product_basic",
        "function": "impact_without",
        "args": [[1, 2, 3, 4]],
        "expected": [24, 12, 8, 6]
      },
      {
        "name": "product_with_zero",
        "function": "impact_without",
        "args": [[-1, 1, 0, -3, 3]],
        "expected": [0, 0, 9, 0, 0],
        "is_trap": true,
        "trap_explanation": "Contient un zéro - division interdite"
      },
      {
        "name": "xor_range",
        "setup": ["build_xor_keys", [1, 3, 4, 8]],
        "function": "range_xor",
        "args": ["$setup", 0, 1],
        "expected": 2
      },
      {
        "name": "rectangle_max",
        "function": "best_warehouse_section",
        "args": [[[1,2,-1,-4,-20],[-8,-3,4,2,1],[3,8,10,1,3],[-4,-1,1,7,-6]]],
        "expected": 29
      },
      {
        "name": "empty_array",
        "function": "best_streak",
        "args": [[]],
        "expected": 0,
        "is_trap": true,
        "trap_explanation": "Tableau vide - doit gérer gracieusement"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 1000,
      "generators": [
        {
          "type": "array_int",
          "param_index": 0,
          "params": {
            "min_len": 1,
            "max_len": 1000,
            "min_val": -10000,
            "max_val": 10000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["malloc", "free", "memset", "memcpy"],
    "forbidden_functions": [],
    "check_overflow": true,
    "check_memory": true,
    "blocking": true
  },

  "bonus": {
    "tier": "ADVANCED",
    "icon": "🔥",
    "xp_multiplier": 3,
    "functions": [
      "best_warehouse_section_optimized",
      "SparseDeltaTracker",
      "StreamingLedger"
    ]
  }
}
```

### 4.10 Solutions Mutantes

```rust
// ═══════════════════════════════════════════════════════════════
// MUTANT A (Boundary) : Off-by-one dans range_sum
// ═══════════════════════════════════════════════════════════════

impl EmpireLedger {
    pub fn range_sum(&self, left: usize, right: usize) -> i64 {
        // BUG: Utilise right au lieu de right + 1
        self.prefix[right] - self.prefix[left]
    }
}
// Pourquoi c'est faux : Omet le dernier élément de la plage
// Ce qui était pensé : Confusion entre indices inclusifs et exclusifs

// ═══════════════════════════════════════════════════════════════
// MUTANT B (Safety) : Pas de vérification left > right
// ═══════════════════════════════════════════════════════════════

impl EmpireLedger {
    pub fn range_sum(&self, left: usize, right: usize) -> i64 {
        // BUG: Pas de vérification left <= right
        self.prefix[right + 1] - self.prefix[left]
    }
}
// Pourquoi c'est faux : Si left > right, retourne une valeur négative incorrecte
// Ce qui était pensé : "Les utilisateurs passeront toujours des indices valides"

// ═══════════════════════════════════════════════════════════════
// MUTANT C (Overflow) : Utilise i32 au lieu de i64
// ═══════════════════════════════════════════════════════════════

pub struct EmpireLedgerBad {
    prefix: Vec<i32>,  // BUG: i32 au lieu de i64
}

impl EmpireLedgerBad {
    pub fn new(revenues: &[i32]) -> Self {
        let mut prefix = vec![0i32; revenues.len() + 1];
        for i in 0..revenues.len() {
            prefix[i + 1] = prefix[i] + revenues[i];  // Overflow possible!
        }
        EmpireLedgerBad { prefix }
    }
}
// Pourquoi c'est faux : Overflow silencieux avec grandes sommes
// Ce qui était pensé : "Les valeurs sont des i32, le résultat aussi"

// ═══════════════════════════════════════════════════════════════
// MUTANT D (Logic) : Formule 2D incomplète
// ═══════════════════════════════════════════════════════════════

impl WarehouseGrid {
    pub fn range_sum(&self, r1: usize, c1: usize, r2: usize, c2: usize) -> i64 {
        // BUG: Oublie le + self.prefix[r1][c1]
        self.prefix[r2 + 1][c2 + 1]
            - self.prefix[r1][c2 + 1]
            - self.prefix[r2 + 1][c1]
        // Manque: + self.prefix[r1][c1]
    }
}
// Pourquoi c'est faux : Le coin supérieur gauche est soustrait deux fois
// Ce qui était pensé : "J'enlève les deux rectangles en trop"

// ═══════════════════════════════════════════════════════════════
// MUTANT E (Return) : range_add sans vérification de borne
// ═══════════════════════════════════════════════════════════════

impl DeltaTracker {
    pub fn range_add(&mut self, left: usize, right: usize, value: i64) {
        self.diff[left] += value;
        // BUG: Pas de vérification right + 1 < n
        self.diff[right + 1] -= value;  // Panic si right == n-1
    }
}
// Pourquoi c'est faux : Index out of bounds si right est le dernier index
// Ce qui était pensé : "right + 1 existe toujours"

// ═══════════════════════════════════════════════════════════════
// MUTANT F (Logic) : Kadane reset mal placé
// ═══════════════════════════════════════════════════════════════

pub fn best_streak_bad(revenues: &[i32]) -> i64 {
    let mut max_so_far = 0i64;  // BUG: Devrait être revenues[0]
    let mut max_ending = 0i64;

    for &rev in revenues {
        max_ending = (rev as i64).max(max_ending + rev as i64);
        max_so_far = max_so_far.max(max_ending);
    }

    max_so_far
}
// Pourquoi c'est faux : Si tous les éléments sont négatifs, retourne 0
// Ce qui était pensé : "Le max commence à 0"
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

| Concept | Description | Pourquoi c'est important |
|---------|-------------|-------------------------|
| **Prefix Sum** | Tableau cumulatif pour requêtes O(1) | Fondamental en algorithmique |
| **Difference Array** | Dual des prefix sums pour updates O(1) | Optimise les modifications de plage |
| **2D Extension** | Prefix sums sur matrices | Images, grilles, data science |
| **Kadane** | Maximum subarray en O(n) | Problème classique d'entretien |
| **Hash + Prefix** | Compter sous-tableaux avec somme k | Technique avancée puissante |

### 5.2 LDA — Traduction Littérale

**EmpireLedger::new**
```
FONCTION new QUI RETOURNE UN EmpireLedger ET PREND EN PARAMÈTRE revenues QUI EST UN SLICE D'ENTIERS
DÉBUT FONCTION
    DÉCLARER n COMME ENTIER ÉGAL À LA LONGUEUR DE revenues
    DÉCLARER prefix COMME VECTEUR D'ENTIERS 64 BITS DE TAILLE n PLUS 1 INITIALISÉ À 0

    POUR i ALLANT DE 0 À n MOINS 1 FAIRE
        AFFECTER prefix[i] PLUS revenues[i] À prefix[i PLUS 1]
    FIN POUR

    RETOURNER EmpireLedger AVEC prefix
FIN FONCTION
```

**range_sum**
```
FONCTION range_sum QUI RETOURNE UN ENTIER 64 BITS ET PREND EN PARAMÈTRES left ET right QUI SONT DES ENTIERS NON SIGNÉS
DÉBUT FONCTION
    SI left EST SUPÉRIEUR À right ALORS
        RETOURNER 0
    FIN SI

    RETOURNER prefix[right PLUS 1] MOINS prefix[left]
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Range Sum Query avec Prefix Sum
---
1. PRÉTRAITEMENT (une seule fois) :
   a. CRÉER tableau prefix de taille n+1
   b. prefix[0] = 0
   c. POUR chaque élément i de 0 à n-1 :
      - prefix[i+1] = prefix[i] + arr[i]

2. REQUÊTE range_sum(left, right) :
   a. VÉRIFIER validité : left <= right
   b. RETOURNER prefix[right+1] - prefix[left]

3. COMPLEXITÉ :
   - Prétraitement : O(n) temps, O(n) espace
   - Requête : O(1) temps, O(1) espace
```

### 5.2.3 Représentation Algorithmique (Logique de Garde)

```
FONCTION : range_sum(left, right)
---
INIT result = 0

1. VÉRIFIER les gardes (Fail Fast) :
   |
   |-- SI left > right :
   |     RETOURNER 0 (plage invalide)
   |
   |-- SI right >= n :
   |     RETOURNER 0 (hors limites)
   |
   |-- CONTINUER si valide

2. CALCUL :
   |-- result = prefix[right + 1] - prefix[left]
   |-- RETOURNER result
```

### 5.3 Visualisation ASCII

**Prefix Sum Construction:**
```
Original:    [ 3 ]  [ 1 ]  [ 4 ]  [ 1 ]  [ 5 ]
               ↓      ↓      ↓      ↓      ↓
Prefix:  [0] → [3] → [4] → [8] → [9] → [14]
          ↑     ↑     ↑     ↑     ↑      ↑
         0+3   3+1   4+4   8+1   9+5   Total

Range Sum [1, 3] = prefix[4] - prefix[1]
                 = 9 - 3
                 = 6  ✓ (1 + 4 + 1 = 6)
```

**2D Prefix Sum (Inclusion-Exclusion):**
```
         c1      c2
        ↓       ↓
    ┌───────────────┐
    │ A │     B     │
r1→ │───┼───────────│
    │   │           │
    │ C │     D     │  ← Zone cible D
    │   │           │
r2→ └───────────────┘

sum(D) = prefix(r2,c2) - prefix(r1-1,c2) - prefix(r2,c1-1) + prefix(r1-1,c1-1)
       = (A+B+C+D)     - (A+B)           - (A+C)           + A
       = D ✓
```

**Difference Array Update:**
```
Original:    [ 0 ]  [ 0 ]  [ 0 ]  [ 0 ]  [ 0 ]
                      ↓                   ↓
range_add(1, 3, +10):

Diff:        [ 0 ]  [+10]  [ 0 ]  [ 0 ]  [-10]
                     ↑                     ↑
                   start                end+1

After build (prefix sum of diff):
Result:      [ 0 ]  [ 10 ] [ 10 ] [ 10 ] [ 0 ]
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **Off-by-one** | prefix[right] vs prefix[right+1] | Dessiner l'indexation sur papier |
| **Overflow i32** | Grandes sommes dépassent i32::MAX | Toujours utiliser i64 pour les sommes |
| **2D formula** | Oublier +prefix[r1][c1] | Inclusion-exclusion: ce qu'on retire 2x |
| **Diff bounds** | right+1 peut dépasser | Vérifier avant d'écrire |
| **Kadane init** | Commencer à 0 | Commencer à arr[0] |

### 5.5 Cours Complet

#### 5.5.1 Introduction aux Prefix Sums

Les **prefix sums** (ou sommes préfixes) sont une technique de prétraitement qui permet de répondre à des requêtes de somme de plage en O(1) après un prétraitement O(n).

**Intuition :**

Imagine que tu dois calculer la somme des éléments de l'indice 2 à 5 d'un tableau. L'approche naïve est de parcourir et additionner (O(n) par requête). Mais si tu dois faire 1 million de requêtes sur le même tableau ?

La clé : **précalculer les sommes cumulatives**.

```
arr:    [3, 1, 4, 1, 5, 9, 2, 6]
prefix: [0, 3, 4, 8, 9, 14, 23, 25, 31]
         ↑
    Sentinel (prefix[0] = 0 pour simplifier)
```

Pour obtenir `sum(arr[l..=r])` :
- C'est la différence entre "somme de tout jusqu'à r" et "somme de tout avant l"
- `= prefix[r+1] - prefix[l]`

#### 5.5.2 Difference Arrays

Les **difference arrays** sont le dual des prefix sums :

- **Prefix sum** : Convertit des DIFFÉRENCES en VALEURS
- **Difference array** : Convertit des VALEURS en DIFFÉRENCES

**Propriété clé :**
```
Si D est le difference array de A, alors A est le prefix sum de D.
```

Cette dualité permet des **range updates en O(1)** :

Pour ajouter `val` à tous les éléments de `arr[l..=r]` :
```
diff[l] += val
diff[r+1] -= val  (si r+1 < n)
```

Pourquoi ça marche ? Parce que le prefix sum de `diff` va propager l'ajout de `l` jusqu'à la fin, mais le `-val` à `r+1` annule l'effet pour les éléments après `r`.

#### 5.5.3 Extension 2D

Pour une matrice, on étend le concept :

```
prefix[i][j] = somme de tous les éléments dans le rectangle (0,0) à (i-1, j-1)
```

**Construction :**
```
prefix[i][j] = matrix[i-1][j-1]
             + prefix[i-1][j]      // rectangle au-dessus
             + prefix[i][j-1]      // rectangle à gauche
             - prefix[i-1][j-1]    // intersection (comptée 2 fois)
```

**Requête :**
```
sum(r1,c1,r2,c2) = prefix[r2+1][c2+1]
                 - prefix[r1][c2+1]
                 - prefix[r2+1][c1]
                 + prefix[r1][c1]
```

C'est le principe d'**inclusion-exclusion**.

#### 5.5.4 Kadane's Algorithm

Pour trouver le sous-tableau de somme maximale :

```
max_ending_here = arr[0]
max_so_far = arr[0]

for i in 1..n:
    max_ending_here = max(arr[i], max_ending_here + arr[i])
    max_so_far = max(max_so_far, max_ending_here)

return max_so_far
```

**Intuition :** À chaque position, on décide si on :
1. Continue le sous-tableau courant
2. Recommence un nouveau sous-tableau ici

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais dangereux)                         │
├─────────────────────────────────────────────────────────────────┤
│ let sum = arr.iter().sum::<i32>();  // Peut overflow            │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ let sum: i64 = arr.iter().map(|&x| x as i64).sum();            │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ • 10^5 éléments × valeur max 10^5 = 10^10 > i32::MAX           │
│ • Toujours utiliser i64 pour les accumulateurs                  │
│ • Le cast individuel évite l'overflow pendant l'accumulation    │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**range_sum([1, 2, 3, 4, 5], 1, 3)**

```
┌───────┬────────────────────────────────────────┬─────────────────┬─────────────────────────┐
│ Étape │ Instruction                            │ prefix[]        │ Explication             │
├───────┼────────────────────────────────────────┼─────────────────┼─────────────────────────┤
│   1   │ CONSTRUCTION prefix                    │ [0,1,3,6,10,15] │ Sommes cumulatives      │
├───────┼────────────────────────────────────────┼─────────────────┼─────────────────────────┤
│   2   │ left = 1, right = 3                    │                 │ Requête reçue           │
├───────┼────────────────────────────────────────┼─────────────────┼─────────────────────────┤
│   3   │ Vérifier 1 <= 3 ?                      │                 │ VRAI → continuer        │
├───────┼────────────────────────────────────────┼─────────────────┼─────────────────────────┤
│   4   │ prefix[3+1] = prefix[4]                │ → 10            │ Somme jusqu'à index 3   │
├───────┼────────────────────────────────────────┼─────────────────┼─────────────────────────┤
│   5   │ prefix[1] = 1                          │ → 1             │ Somme avant index 1     │
├───────┼────────────────────────────────────────┼─────────────────┼─────────────────────────┤
│   6   │ RETOURNER 10 - 1                       │ → 9             │ 2 + 3 + 4 = 9 ✓        │
└───────┴────────────────────────────────────────┴─────────────────┴─────────────────────────┘
```

### 5.8 Mnémotechniques

#### 🧪 MEME : "I am the one who counts" — Walter White et les Prefix Sums

Comme Walter White qui compte méticuleusement son argent sale, tu dois maintenir un running total de tout ce qui est gagné.

```rust
// Walt compte chaque batch
let mut total = 0;
for batch in revenues {
    total += batch;
    ledger.push(total);  // Le grand livre ne ment jamais
}
```

**"Say my sum!"** — Quand quelqu'un te demande combien tu as gagné du batch 3 au batch 7, tu réponds instantanément grâce au prefix sum.

#### 💊 MEME : "Blue Sky" — La pureté du O(1)

Comme la méthamphétamine de Heisenberg qui est pure à 99.1%, les requêtes prefix sum sont pures O(1) — aucune impureté algorithmique.

```
Brute force: O(n) par requête  → Impure, comme le produit des autres
Prefix sum:  O(1) par requête  → Blue Sky, 99.1% pure
```

### 5.9 Applications pratiques

| Domaine | Application |
|---------|-------------|
| **Finance** | Running P&L, cumulative returns |
| **Gaming** | Damage meters, XP tracking |
| **Image** | Integral images (box blur, SURF features) |
| **Database** | Window functions, OLAP |
| **ML** | Feature engineering, rolling statistics |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Détection |
|---|-------|--------|-----------|
| 1 | Off-by-one (right vs right+1) | Résultat incorrect | Tests range avec single element |
| 2 | Overflow i32 sur grandes sommes | Valeurs négatives aberrantes | Tests avec 10^5 éléments |
| 3 | Formule 2D incomplète | Résultat doublé/manquant | Tests sous-matrices variées |
| 4 | diff[r+1] hors limites | Panic/crash | Tests avec right = n-1 |
| 5 | Kadane init à 0 | Faux pour all-negative | Tests [-5, -3, -1] |

---

## 📝 SECTION 7 : QCM

### Q1. Prefix Sum Construction
Quel est `prefix[3]` pour `arr = [2, 5, 1, 8]` ?

A) 7
B) 8
C) 14
D) 6

**Réponse : B**

`prefix[3] = arr[0] + arr[1] + arr[2] = 2 + 5 + 1 = 8`

---

### Q2. Range Sum Formula
Pour calculer `sum(arr[2..=4])` avec un prefix sum, on utilise :

A) `prefix[4] - prefix[2]`
B) `prefix[5] - prefix[2]`
C) `prefix[4] - prefix[1]`
D) `prefix[5] - prefix[1]`

**Réponse : B**

Range inclusif [2, 4] → `prefix[4+1] - prefix[2] = prefix[5] - prefix[2]`

---

### Q3. Difference Array Update
Après `range_add(1, 3, 5)` sur `diff = [0, 0, 0, 0, 0]`, que contient diff ?

A) `[0, 5, 5, 5, -5]`
B) `[0, 5, 0, 0, -5]`
C) `[0, 5, 5, 5, 0]`
D) `[5, 0, 0, 0, -5]`

**Réponse : B**

On ajoute +5 à diff[1] et -5 à diff[4] (= 3+1)

---

### Q4. 2D Prefix Sum
Pour la formule de requête 2D, combien de termes sont nécessaires ?

A) 2
B) 3
C) 4
D) 5

**Réponse : C**

`prefix[r2+1][c2+1] - prefix[r1][c2+1] - prefix[r2+1][c1] + prefix[r1][c1]`

---

### Q5. Kadane Edge Case
Que retourne Kadane sur `[-3, -1, -4]` ?

A) 0
B) -1
C) -3
D) -8

**Réponse : B**

Le sous-tableau de somme max est `[-1]` avec somme -1

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.1.5 - heisenberg_ledger |
| **Difficulté** | 5/10 (★★★★★☆☆☆☆☆) |
| **Structures** | 3 (EmpireLedger, WarehouseGrid, DeltaTracker) |
| **Fonctions** | 8 utilitaires |
| **Complexité Construction** | O(n) pour 1D, O(nm) pour 2D |
| **Complexité Requête** | O(1) |
| **Bonus** | 🔥 Avancé (×3 XP) |
| **Points totaux** | 100 base + 50 bonus |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.1.5-heisenberg_ledger",
    "generated_at": "2026-01-11T10:30:00Z",

    "metadata": {
      "exercise_id": "1.1.5",
      "exercise_name": "heisenberg_ledger",
      "module": "1.1",
      "module_name": "Arrays & Sorting",
      "concept": "k",
      "concept_name": "Prefix Sums & Difference Arrays",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse (prefix 1D + 2D + difference + applications)",
      "phase": 1,
      "difficulty": 5,
      "difficulty_stars": "★★★★★☆☆☆☆☆",
      "language": "rust",
      "language_version": "edition_2024",
      "language_alt": "c17",
      "duration_minutes": 60,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "ADVANCED",
      "bonus_icon": "🔥",
      "complexity_time": "T2 O(n)",
      "complexity_space": "S2 O(n)",
      "prerequisites": ["arrays", "loops", "indexation"],
      "domains": ["Struct", "MD", "DP"],
      "domains_bonus": ["DP", "Calcul"],
      "tags": ["prefix_sum", "difference_array", "kadane", "range_query"],
      "meme_reference": "Breaking Bad - I am the one who counts"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_heisenberg_ledger.rs": "/* Section 4.3 */",
      "references/ref_heisenberg_ledger.c": "/* C implementation */",
      "references/ref_bonus.rs": "/* Section 4.6 */",
      "alternatives/alt_kadane_tracked.rs": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 */",
      "mutants/mutant_c_overflow.rs": "/* Section 4.10 */",
      "mutants/mutant_d_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
      "mutants/mutant_f_kadane.rs": "/* Section 4.10 */",
      "tests/main.rs": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_heisenberg_ledger.rs",
        "references/ref_bonus.rs",
        "alternatives/alt_kadane_tracked.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_safety.rs",
        "mutants/mutant_c_overflow.rs",
        "mutants/mutant_d_logic.rs",
        "mutants/mutant_e_return.rs",
        "mutants/mutant_f_kadane.rs"
      ]
    },

    "commands": {
      "validate_spec": "hackbrain-engine validate spec.json",
      "test_reference": "hackbrain-engine test -s spec.json -f references/ref_heisenberg_ledger.rs",
      "test_mutants": "hackbrain-mutation-tester -r references/ref_heisenberg_ledger.rs -s spec.json --validate"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "I am the one who counts"*
*L'excellence pédagogique ne se négocie pas*
