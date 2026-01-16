# Exercice 1.1.2 : the_price_is_right

**Module :**
1.1 — Arrays & Sorting

**Concept :**
g — Binary Search Variants

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (concepts g + variantes 22.g-k + 23.h-n + 24.f-i)

**Langages :**
Rust Edition 2024 + C (c17)

**Prérequis :**
- Notion de tableau trié
- Comparaisons et opérateurs logiques
- Récursivité ou itération avec while

**Domaines :**
Tri, MD, Algo

**Durée estimée :**
60 min

**XP Base :**
150

**Complexité :**
T3 O(log n) × S1 O(1)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `binary_search.c`, `binary_search.h`

**Fonctions autorisées :**
- Aucune fonction de la bibliothèque standard pour la recherche
- Allocation mémoire si nécessaire (malloc/free en C)

**Fonctions interdites :**
- `bsearch` (C)
- `binary_search` de la STL (Rust std déjà fait ça, tu dois l'implémenter)

---

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🎮 THE PRICE IS RIGHT — L'Art de Deviner Sans Se Tromper**

*"Come on down! You're the next contestant on The Price Is Right!"*

Tu connais le jeu **"Le Juste Prix"** ? Le présentateur te montre un objet, tu proposes un prix, et il te dit "PLUS !" ou "MOINS !" jusqu'à ce que tu trouves.

**Le secret des champions ?** Ils ne devinent pas au hasard. Ils coupent TOUJOURS en deux.

Si le prix est entre 1€ et 1000€ :
- Tu dis 500€ → "PLUS !" → maintenant c'est entre 501€ et 1000€
- Tu dis 750€ → "MOINS !" → maintenant c'est entre 501€ et 749€
- Tu dis 625€ → "PLUS !" → entre 626€ et 749€
- ...et ainsi de suite

En **10 coups maximum**, tu peux trouver parmi **1024 possibilités** ! C'est la magie de la **recherche binaire**.

Mais attention, le vrai jeu est plus complexe :
- Parfois tu cherches le **premier prix valide** (lower bound)
- Parfois le **dernier** (upper bound)
- Parfois l'objet a été **déplacé** dans un catalogue rotatif (rotated array)
- Parfois tu dois trouver le **pic** d'une enchère (peak element)

**Ta mission :**

Implémenter une bibliothèque complète de recherche binaire avec **13 variantes** différentes, chacune optimisée pour un cas d'usage spécifique.

#### 1.2.2 Version Académique

La recherche binaire (binary search) est un algorithme de recherche en O(log n) qui fonctionne sur des données triées. À chaque itération, l'algorithme divise l'espace de recherche en deux, éliminant la moitié des candidats.

Cet exercice couvre les variantes suivantes :
1. **Recherche standard** : trouver l'index d'une valeur exacte
2. **Lower bound** : premier élément ≥ cible
3. **Upper bound** : premier élément > cible
4. **Ceil/Floor** : plus petit élément ≥ cible / plus grand élément ≤ cible
5. **Peak element** : sommet dans un tableau bitonique
6. **Rotated array** : recherche dans un tableau trié puis pivoté
7. **Integer sqrt** : racine carrée entière par recherche binaire
8. **Kth smallest in matrix** : kème plus petit dans une matrice triée
9. **Binary search on answer** : recherche sur l'espace des solutions

---

**Entrée :**
- `arr` : slice/tableau trié (sauf cas spéciaux)
- `target` : valeur recherchée
- Paramètres additionnels selon la variante

**Sortie :**
- Index trouvé ou `None`/`-1` si non trouvé
- Valeur calculée pour certaines variantes (isqrt, kth_smallest)

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  0 ≤ arr.len() ≤ 10⁶                                            │
│  Complexité temps : O(log n) pour chaque recherche              │
│  Complexité espace : O(1) auxiliaire                            │
│  Pour isqrt : 0 ≤ n ≤ 10¹⁸                                      │
│  Éviter overflow : utiliser lo + (hi - lo) / 2                  │
└─────────────────────────────────────────────────────────────────┘
```

---

### 1.3 Prototype

#### Rust (Edition 2024)

```rust
pub mod binary_search {
    /// Recherche standard - retourne Some(index) si trouvé
    pub fn search<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

    /// Lower bound - premier élément >= target
    pub fn lower_bound<T: Ord>(arr: &[T], target: &T) -> usize;

    /// Upper bound - premier élément > target
    pub fn upper_bound<T: Ord>(arr: &[T], target: &T) -> usize;

    /// Ceil - plus petit élément >= target (ou None)
    pub fn ceil<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

    /// Floor - plus grand élément <= target (ou None)
    pub fn floor<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

    /// Trouve le pic dans un tableau bitonique
    pub fn find_peak(arr: &[i32]) -> usize;

    /// Recherche dans un tableau trié puis pivoté (sans duplicats)
    pub fn search_rotated<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

    /// Recherche dans un tableau pivoté avec duplicats
    pub fn search_rotated_with_dups<T: Ord>(arr: &[T], target: &T) -> bool;

    /// Trouve le point de rotation (index du minimum)
    pub fn find_rotation_point<T: Ord>(arr: &[T]) -> usize;

    /// Racine carrée entière : floor(sqrt(n))
    pub fn isqrt(n: u64) -> u64;

    /// Kème plus petit dans une matrice triée par lignes et colonnes
    pub fn kth_smallest_matrix(matrix: &[Vec<i32>], k: usize) -> i32;

    /// Binary search on answer : minimum X tel que predicate(X) est vrai
    pub fn binary_search_answer<F>(lo: i64, hi: i64, predicate: F) -> i64
    where
        F: Fn(i64) -> bool;

    /// Binary search sur flottants avec précision epsilon
    pub fn binary_search_float<F>(lo: f64, hi: f64, eps: f64, predicate: F) -> f64
    where
        F: Fn(f64) -> bool;
}
```

#### C (c17)

```c
#ifndef BINARY_SEARCH_H
# define BINARY_SEARCH_H

# include <stddef.h>
# include <stdint.h>
# include <stdbool.h>

// Comparateur générique : retourne <0, 0, ou >0
typedef int (*comparator_fn)(const void *a, const void *b);

// Recherche standard - retourne index ou -1
ssize_t bs_search(const void *arr, size_t len, size_t elem_size,
                  const void *target, comparator_fn cmp);

// Lower bound - premier élément >= target
size_t bs_lower_bound(const void *arr, size_t len, size_t elem_size,
                      const void *target, comparator_fn cmp);

// Upper bound - premier élément > target
size_t bs_upper_bound(const void *arr, size_t len, size_t elem_size,
                      const void *target, comparator_fn cmp);

// Ceil - retourne index ou -1 si aucun
ssize_t bs_ceil(const void *arr, size_t len, size_t elem_size,
                const void *target, comparator_fn cmp);

// Floor - retourne index ou -1 si aucun
ssize_t bs_floor(const void *arr, size_t len, size_t elem_size,
                 const void *target, comparator_fn cmp);

// Peak element dans tableau bitonique (spécialisé int)
size_t bs_find_peak(const int *arr, size_t len);

// Recherche dans tableau pivoté (spécialisé int)
ssize_t bs_search_rotated(const int *arr, size_t len, int target);

// Recherche avec duplicats
bool bs_search_rotated_dups(const int *arr, size_t len, int target);

// Point de rotation
size_t bs_find_rotation_point(const int *arr, size_t len);

// Racine carrée entière
uint64_t bs_isqrt(uint64_t n);

// Kème plus petit dans matrice (matrix[rows][cols])
int bs_kth_smallest_matrix(const int **matrix, size_t rows, size_t cols, size_t k);

// Binary search on answer avec prédicat
typedef bool (*predicate_fn)(int64_t value, void *context);
int64_t bs_search_answer(int64_t lo, int64_t hi, predicate_fn pred, void *ctx);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Fun Facts

**🎯 Le bug qui a duré 20 ans**

En 1986, Jon Bentley publie "Programming Pearls" avec une implémentation de binary search. En 2006, Joshua Bloch (ingénieur Google) découvre un bug : `(lo + hi) / 2` peut overflow !

Le fix : `lo + (hi - lo) / 2`

Ce bug existait dans le JDK Java et des milliers de programmes pendant **20 ans**.

**📊 La puissance du log**

| Taille | Recherche linéaire | Binary search |
|--------|-------------------|---------------|
| 1,000 | 1,000 ops | 10 ops |
| 1,000,000 | 1,000,000 ops | 20 ops |
| 1,000,000,000 | 1 milliard ops | 30 ops |

Avec 1 milliard d'éléments, binary search est **33 millions de fois plus rapide**.

**🔍 Git bisect**

Git utilise binary search pour trouver le commit qui a introduit un bug. Au lieu de tester 1000 commits un par un, `git bisect` en teste ~10.

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation |
|--------|-------------|
| **DevOps** | `git bisect` pour trouver le commit fautif |
| **Game Dev** | Recherche de collision dans un espace trié |
| **Finance** | Trouver le seuil optimal (binary search on answer) |
| **Data Engineer** | Index B-Tree dans les bases de données |
| **ML Engineer** | Hyperparameter tuning avec binary search |
| **System Admin** | Trouver l'entrée dans les logs triés par timestamp |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
binary_search.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 13 tests
test test_standard_search ... ok
test test_lower_bound ... ok
test test_upper_bound ... ok
test test_ceil ... ok
test test_floor ... ok
test test_find_peak ... ok
test test_search_rotated ... ok
test test_search_rotated_dups ... ok
test test_rotation_point ... ok
test test_isqrt ... ok
test test_kth_matrix ... ok
test test_search_answer ... ok
test test_search_float ... ok

test result: ok. 13 passed; 0 failed
```

---

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(log n) strict avec analyse des constantes

**Space Complexity attendue :**
O(1) strict — pas de récursion, itératif uniquement

**Domaines Bonus :**
`MD, Probas`

#### 3.1.1 Consigne Bonus

**🎮 THE PRICE IS RIGHT: TOURNAMENT EDITION**

Tu participes au championnat mondial du Juste Prix. Les règles sont plus strictes :

1. **Ternary Search** : Implémente une recherche ternaire pour trouver le maximum d'une fonction unimodale (une seule bosse)
2. **Exponential Search** : Pour les tableaux de taille inconnue (ou infinie), trouve d'abord une borne avec une croissance exponentielle
3. **Interpolation Search** : Prédit la position en fonction de la distribution (O(log log n) pour données uniformes)
4. **Fractional Cascading** : Recherche simultanée dans plusieurs tableaux triés

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  Ternary search : 2 comparaisons par itération max              │
│  Exponential : O(log i) où i est la position de la cible        │
│  Interpolation : Formule exacte de prédiction requise           │
│  Fractional cascading : O(log n + k) pour k tableaux            │
└─────────────────────────────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
/// Ternary search pour maximum d'une fonction unimodale
pub fn ternary_search_max<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
where
    F: Fn(f64) -> f64;

/// Exponential search (tableau potentiellement infini)
pub fn exponential_search<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

/// Interpolation search (distribution uniforme)
pub fn interpolation_search(arr: &[i64], target: i64) -> Option<usize>;

/// Fractional cascading sur plusieurs tableaux
pub fn fractional_cascade_search(arrays: &[&[i32]], target: i32) -> Vec<Option<usize>>;
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Nombre de variantes | 13 | 17 |
| Complexité analyse | Intuitive | Formelle avec preuves |
| Distribution données | Quelconque | Uniforme pour interpolation |
| Structures | 1 tableau | Multiple (cascading) |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `search_found` | `[1,3,5,7,9], 5` | `Some(2)` | 2 | Non |
| `search_not_found` | `[1,3,5,7,9], 4` | `None` | 2 | Oui |
| `search_first` | `[1,3,5,7,9], 1` | `Some(0)` | 2 | Boundary |
| `search_last` | `[1,3,5,7,9], 9` | `Some(4)` | 2 | Boundary |
| `search_empty` | `[], 5` | `None` | 2 | Edge |
| `lower_bound_exact` | `[1,2,2,2,3], 2` | `1` | 3 | Non |
| `lower_bound_between` | `[1,3,5,7], 4` | `2` | 3 | Oui |
| `lower_bound_start` | `[1,3,5,7], 0` | `0` | 3 | Boundary |
| `lower_bound_end` | `[1,3,5,7], 8` | `4` | 3 | Boundary |
| `upper_bound_exact` | `[1,2,2,2,3], 2` | `4` | 3 | Non |
| `ceil_exists` | `[1,3,5,7], 4` | `Some(2)` | 2 | Non |
| `ceil_none` | `[1,3,5,7], 8` | `None` | 2 | Edge |
| `floor_exists` | `[1,3,5,7], 4` | `Some(1)` | 2 | Non |
| `floor_none` | `[1,3,5,7], 0` | `None` | 2 | Edge |
| `peak_middle` | `[1,3,5,3,1]` | `2` | 3 | Non |
| `peak_start` | `[5,3,1]` | `0` | 3 | Boundary |
| `peak_end` | `[1,3,5]` | `2` | 3 | Boundary |
| `rotated_found` | `[4,5,6,7,0,1,2], 0` | `Some(4)` | 4 | Non |
| `rotated_not_rotated` | `[1,2,3,4,5], 3` | `Some(2)` | 3 | Edge |
| `rotation_point` | `[4,5,6,7,0,1,2]` | `4` | 3 | Non |
| `rotation_not_rotated` | `[1,2,3,4,5]` | `0` | 3 | Edge |
| `isqrt_perfect` | `100` | `10` | 2 | Non |
| `isqrt_non_perfect` | `99` | `9` | 2 | Oui |
| `isqrt_zero` | `0` | `0` | 2 | Edge |
| `isqrt_large` | `10^18` | `10^9` | 3 | Overflow |
| `kth_matrix_first` | `matrix, 1` | `1` | 3 | Boundary |
| `kth_matrix_middle` | `matrix, 5` | `11` | 3 | Non |
| `search_answer` | `books=[3,6,7,11], days=8` | `11` | 5 | Non |
| `float_sqrt` | `sqrt(2), eps=0.0001` | `~1.4142` | 4 | Precision |

**Total : 100 points**

---

### 4.2 main.c de test

```c
#include <stdio.h>
#include <assert.h>
#include "binary_search.h"

int cmp_int(const void *a, const void *b) {
    return *(const int*)a - *(const int*)b;
}

void test_search(void) {
    int arr[] = {1, 3, 5, 7, 9};
    size_t len = 5;

    assert(bs_search(arr, len, sizeof(int), &(int){5}, cmp_int) == 2);
    assert(bs_search(arr, len, sizeof(int), &(int){4}, cmp_int) == -1);
    assert(bs_search(arr, len, sizeof(int), &(int){1}, cmp_int) == 0);
    assert(bs_search(arr, len, sizeof(int), &(int){9}, cmp_int) == 4);
    printf("test_search: OK\n");
}

void test_lower_bound(void) {
    int arr[] = {1, 2, 2, 2, 3, 4, 5};
    size_t len = 7;

    assert(bs_lower_bound(arr, len, sizeof(int), &(int){2}, cmp_int) == 1);
    assert(bs_lower_bound(arr, len, sizeof(int), &(int){0}, cmp_int) == 0);
    assert(bs_lower_bound(arr, len, sizeof(int), &(int){6}, cmp_int) == 7);
    printf("test_lower_bound: OK\n");
}

void test_upper_bound(void) {
    int arr[] = {1, 2, 2, 2, 3, 4, 5};
    size_t len = 7;

    assert(bs_upper_bound(arr, len, sizeof(int), &(int){2}, cmp_int) == 4);
    assert(bs_upper_bound(arr, len, sizeof(int), &(int){5}, cmp_int) == 7);
    printf("test_upper_bound: OK\n");
}

void test_peak(void) {
    int arr1[] = {1, 3, 5, 7, 6, 4, 2};
    assert(bs_find_peak(arr1, 7) == 3);

    int arr2[] = {1, 2, 3, 4, 5};
    assert(bs_find_peak(arr2, 5) == 4);

    int arr3[] = {5, 4, 3, 2, 1};
    assert(bs_find_peak(arr3, 5) == 0);
    printf("test_peak: OK\n");
}

void test_rotated(void) {
    int arr[] = {4, 5, 6, 7, 0, 1, 2};

    assert(bs_search_rotated(arr, 7, 0) == 4);
    assert(bs_search_rotated(arr, 7, 4) == 0);
    assert(bs_search_rotated(arr, 7, 3) == -1);
    assert(bs_find_rotation_point(arr, 7) == 4);
    printf("test_rotated: OK\n");
}

void test_isqrt(void) {
    assert(bs_isqrt(0) == 0);
    assert(bs_isqrt(1) == 1);
    assert(bs_isqrt(4) == 2);
    assert(bs_isqrt(8) == 2);
    assert(bs_isqrt(9) == 3);
    assert(bs_isqrt(100) == 10);
    assert(bs_isqrt(1000000000000ULL) == 1000000);
    printf("test_isqrt: OK\n");
}

int main(void) {
    test_search();
    test_lower_bound();
    test_upper_bound();
    test_peak();
    test_rotated();
    test_isqrt();

    printf("\nTous les tests passent!\n");
    return 0;
}
```

---

### 4.3 Solution de référence

#### Rust

```rust
pub mod binary_search {
    /// Recherche standard
    pub fn search<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
        if arr.is_empty() {
            return None;
        }
        let mut lo: usize = 0;
        let mut hi: usize = arr.len() - 1;

        while lo <= hi {
            let mid = lo + (hi - lo) / 2;
            match arr[mid].cmp(target) {
                std::cmp::Ordering::Equal => return Some(mid),
                std::cmp::Ordering::Less => lo = mid + 1,
                std::cmp::Ordering::Greater => {
                    if mid == 0 { break; }
                    hi = mid - 1;
                }
            }
        }
        None
    }

    /// Lower bound - premier élément >= target
    pub fn lower_bound<T: Ord>(arr: &[T], target: &T) -> usize {
        let mut lo: usize = 0;
        let mut hi: usize = arr.len();

        while lo < hi {
            let mid = lo + (hi - lo) / 2;
            if arr[mid] < *target {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        lo
    }

    /// Upper bound - premier élément > target
    pub fn upper_bound<T: Ord>(arr: &[T], target: &T) -> usize {
        let mut lo: usize = 0;
        let mut hi: usize = arr.len();

        while lo < hi {
            let mid = lo + (hi - lo) / 2;
            if arr[mid] <= *target {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        lo
    }

    /// Ceil - plus petit élément >= target
    pub fn ceil<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
        let idx = lower_bound(arr, target);
        if idx < arr.len() {
            Some(idx)
        } else {
            None
        }
    }

    /// Floor - plus grand élément <= target
    pub fn floor<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
        let idx = upper_bound(arr, target);
        if idx > 0 {
            Some(idx - 1)
        } else {
            None
        }
    }

    /// Find peak in bitonic array
    pub fn find_peak(arr: &[i32]) -> usize {
        if arr.is_empty() {
            return 0;
        }
        if arr.len() == 1 {
            return 0;
        }

        let mut lo: usize = 0;
        let mut hi: usize = arr.len() - 1;

        while lo < hi {
            let mid = lo + (hi - lo) / 2;
            if arr[mid] < arr[mid + 1] {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        lo
    }

    /// Search in rotated sorted array (no duplicates)
    pub fn search_rotated<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
        if arr.is_empty() {
            return None;
        }

        let mut lo: usize = 0;
        let mut hi: usize = arr.len() - 1;

        while lo <= hi {
            let mid = lo + (hi - lo) / 2;

            if arr[mid] == *target {
                return Some(mid);
            }

            // Left half is sorted
            if arr[lo] <= arr[mid] {
                if arr[lo] <= *target && *target < arr[mid] {
                    if mid == 0 { break; }
                    hi = mid - 1;
                } else {
                    lo = mid + 1;
                }
            }
            // Right half is sorted
            else {
                if arr[mid] < *target && *target <= arr[hi] {
                    lo = mid + 1;
                } else {
                    if mid == 0 { break; }
                    hi = mid - 1;
                }
            }
        }
        None
    }

    /// Search rotated with duplicates
    pub fn search_rotated_with_dups<T: Ord>(arr: &[T], target: &T) -> bool {
        if arr.is_empty() {
            return false;
        }

        let mut lo: usize = 0;
        let mut hi: usize = arr.len() - 1;

        while lo <= hi {
            let mid = lo + (hi - lo) / 2;

            if arr[mid] == *target {
                return true;
            }

            // Handle duplicates at boundaries
            if arr[lo] == arr[mid] && arr[mid] == arr[hi] {
                lo += 1;
                if hi > 0 { hi -= 1; }
                continue;
            }

            if arr[lo] <= arr[mid] {
                if arr[lo] <= *target && *target < arr[mid] {
                    if mid == 0 { break; }
                    hi = mid - 1;
                } else {
                    lo = mid + 1;
                }
            } else {
                if arr[mid] < *target && *target <= arr[hi] {
                    lo = mid + 1;
                } else {
                    if mid == 0 { break; }
                    hi = mid - 1;
                }
            }
        }
        false
    }

    /// Find rotation point (index of minimum)
    pub fn find_rotation_point<T: Ord>(arr: &[T]) -> usize {
        if arr.is_empty() || arr.len() == 1 {
            return 0;
        }

        let mut lo: usize = 0;
        let mut hi: usize = arr.len() - 1;

        // Not rotated
        if arr[lo] < arr[hi] {
            return 0;
        }

        while lo < hi {
            let mid = lo + (hi - lo) / 2;
            if arr[mid] > arr[hi] {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        lo
    }

    /// Integer square root
    pub fn isqrt(n: u64) -> u64 {
        if n == 0 {
            return 0;
        }

        let mut lo: u64 = 1;
        let mut hi: u64 = n.min(3_037_000_499); // sqrt(u64::MAX) ~= 4.29e9

        while lo < hi {
            let mid = lo + (hi - lo + 1) / 2;
            if mid <= n / mid {
                lo = mid;
            } else {
                hi = mid - 1;
            }
        }
        lo
    }

    /// Kth smallest in sorted matrix
    pub fn kth_smallest_matrix(matrix: &[Vec<i32>], k: usize) -> i32 {
        if matrix.is_empty() || matrix[0].is_empty() || k == 0 {
            return 0;
        }

        let rows = matrix.len();
        let cols = matrix[0].len();

        let mut lo = matrix[0][0];
        let mut hi = matrix[rows - 1][cols - 1];

        while lo < hi {
            let mid = lo + (hi - lo) / 2;
            let count = count_less_equal(matrix, mid);

            if count < k {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        lo
    }

    fn count_less_equal(matrix: &[Vec<i32>], target: i32) -> usize {
        let mut count = 0;
        let rows = matrix.len();
        let cols = matrix[0].len();

        let mut row = rows - 1;
        let mut col = 0;

        loop {
            if matrix[row][col] <= target {
                count += row + 1;
                col += 1;
                if col >= cols { break; }
            } else {
                if row == 0 { break; }
                row -= 1;
            }
        }
        count
    }

    /// Binary search on answer
    pub fn binary_search_answer<F>(lo: i64, hi: i64, predicate: F) -> i64
    where
        F: Fn(i64) -> bool,
    {
        let mut lo = lo;
        let mut hi = hi;

        while lo < hi {
            let mid = lo + (hi - lo) / 2;
            if predicate(mid) {
                hi = mid;
            } else {
                lo = mid + 1;
            }
        }
        lo
    }

    /// Floating point binary search
    pub fn binary_search_float<F>(lo: f64, hi: f64, eps: f64, predicate: F) -> f64
    where
        F: Fn(f64) -> bool,
    {
        let mut lo = lo;
        let mut hi = hi;

        while hi - lo > eps {
            let mid = lo + (hi - lo) / 2.0;
            if predicate(mid) {
                hi = mid;
            } else {
                lo = mid;
            }
        }
        lo + (hi - lo) / 2.0
    }
}
```

#### C

```c
#include "binary_search.h"
#include <string.h>

ssize_t bs_search(const void *arr, size_t len, size_t elem_size,
                  const void *target, comparator_fn cmp) {
    if (arr == NULL || len == 0) return -1;

    size_t lo = 0;
    size_t hi = len - 1;

    while (lo <= hi) {
        size_t mid = lo + (hi - lo) / 2;
        const void *mid_elem = (const char*)arr + mid * elem_size;
        int result = cmp(mid_elem, target);

        if (result == 0) return (ssize_t)mid;
        if (result < 0) lo = mid + 1;
        else {
            if (mid == 0) break;
            hi = mid - 1;
        }
    }
    return -1;
}

size_t bs_lower_bound(const void *arr, size_t len, size_t elem_size,
                      const void *target, comparator_fn cmp) {
    size_t lo = 0;
    size_t hi = len;

    while (lo < hi) {
        size_t mid = lo + (hi - lo) / 2;
        const void *mid_elem = (const char*)arr + mid * elem_size;

        if (cmp(mid_elem, target) < 0) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    return lo;
}

size_t bs_upper_bound(const void *arr, size_t len, size_t elem_size,
                      const void *target, comparator_fn cmp) {
    size_t lo = 0;
    size_t hi = len;

    while (lo < hi) {
        size_t mid = lo + (hi - lo) / 2;
        const void *mid_elem = (const char*)arr + mid * elem_size;

        if (cmp(mid_elem, target) <= 0) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    return lo;
}

ssize_t bs_ceil(const void *arr, size_t len, size_t elem_size,
                const void *target, comparator_fn cmp) {
    size_t idx = bs_lower_bound(arr, len, elem_size, target, cmp);
    return (idx < len) ? (ssize_t)idx : -1;
}

ssize_t bs_floor(const void *arr, size_t len, size_t elem_size,
                 const void *target, comparator_fn cmp) {
    size_t idx = bs_upper_bound(arr, len, elem_size, target, cmp);
    return (idx > 0) ? (ssize_t)(idx - 1) : -1;
}

size_t bs_find_peak(const int *arr, size_t len) {
    if (arr == NULL || len == 0) return 0;
    if (len == 1) return 0;

    size_t lo = 0;
    size_t hi = len - 1;

    while (lo < hi) {
        size_t mid = lo + (hi - lo) / 2;
        if (arr[mid] < arr[mid + 1]) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    return lo;
}

ssize_t bs_search_rotated(const int *arr, size_t len, int target) {
    if (arr == NULL || len == 0) return -1;

    size_t lo = 0;
    size_t hi = len - 1;

    while (lo <= hi) {
        size_t mid = lo + (hi - lo) / 2;

        if (arr[mid] == target) return (ssize_t)mid;

        if (arr[lo] <= arr[mid]) {
            if (arr[lo] <= target && target < arr[mid]) {
                if (mid == 0) break;
                hi = mid - 1;
            } else {
                lo = mid + 1;
            }
        } else {
            if (arr[mid] < target && target <= arr[hi]) {
                lo = mid + 1;
            } else {
                if (mid == 0) break;
                hi = mid - 1;
            }
        }
    }
    return -1;
}

bool bs_search_rotated_dups(const int *arr, size_t len, int target) {
    if (arr == NULL || len == 0) return false;

    size_t lo = 0;
    size_t hi = len - 1;

    while (lo <= hi) {
        size_t mid = lo + (hi - lo) / 2;

        if (arr[mid] == target) return true;

        if (arr[lo] == arr[mid] && arr[mid] == arr[hi]) {
            lo++;
            if (hi > 0) hi--;
            continue;
        }

        if (arr[lo] <= arr[mid]) {
            if (arr[lo] <= target && target < arr[mid]) {
                if (mid == 0) break;
                hi = mid - 1;
            } else {
                lo = mid + 1;
            }
        } else {
            if (arr[mid] < target && target <= arr[hi]) {
                lo = mid + 1;
            } else {
                if (mid == 0) break;
                hi = mid - 1;
            }
        }
    }
    return false;
}

size_t bs_find_rotation_point(const int *arr, size_t len) {
    if (arr == NULL || len <= 1) return 0;
    if (arr[0] < arr[len - 1]) return 0;

    size_t lo = 0;
    size_t hi = len - 1;

    while (lo < hi) {
        size_t mid = lo + (hi - lo) / 2;
        if (arr[mid] > arr[hi]) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    return lo;
}

uint64_t bs_isqrt(uint64_t n) {
    if (n == 0) return 0;

    uint64_t lo = 1;
    uint64_t hi = (n < 3037000499ULL) ? n : 3037000499ULL;

    while (lo < hi) {
        uint64_t mid = lo + (hi - lo + 1) / 2;
        if (mid <= n / mid) {
            lo = mid;
        } else {
            hi = mid - 1;
        }
    }
    return lo;
}

static size_t count_le(const int **matrix, size_t rows, size_t cols, int target) {
    size_t count = 0;
    size_t row = rows - 1;
    size_t col = 0;

    while (1) {
        if (matrix[row][col] <= target) {
            count += row + 1;
            col++;
            if (col >= cols) break;
        } else {
            if (row == 0) break;
            row--;
        }
    }
    return count;
}

int bs_kth_smallest_matrix(const int **matrix, size_t rows, size_t cols, size_t k) {
    if (matrix == NULL || rows == 0 || cols == 0 || k == 0) return 0;

    int lo = matrix[0][0];
    int hi = matrix[rows - 1][cols - 1];

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        size_t count = count_le(matrix, rows, cols, mid);

        if (count < k) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    return lo;
}

int64_t bs_search_answer(int64_t lo, int64_t hi, predicate_fn pred, void *ctx) {
    while (lo < hi) {
        int64_t mid = lo + (hi - lo) / 2;
        if (pred(mid, ctx)) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    return lo;
}
```

---

### 4.4 Solutions alternatives acceptées

#### Alternative 1 : Récursive

```rust
pub fn search_recursive<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    fn helper<T: Ord>(arr: &[T], target: &T, lo: usize, hi: usize) -> Option<usize> {
        if lo > hi {
            return None;
        }
        let mid = lo + (hi - lo) / 2;
        match arr[mid].cmp(target) {
            std::cmp::Ordering::Equal => Some(mid),
            std::cmp::Ordering::Less => helper(arr, target, mid + 1, hi),
            std::cmp::Ordering::Greater => {
                if mid == 0 { None }
                else { helper(arr, target, lo, mid - 1) }
            }
        }
    }

    if arr.is_empty() { return None; }
    helper(arr, target, 0, arr.len() - 1)
}
```

#### Alternative 2 : Avec Result pour les erreurs

```rust
pub fn search_result<T: Ord>(arr: &[T], target: &T) -> Result<usize, usize> {
    arr.binary_search(target)
}
// Note: Utilise la méthode standard, acceptable si l'exercice le permet
```

---

### 4.5 Solutions refusées (avec explications)

#### Refusée 1 : Overflow potentiel

```rust
// ❌ REFUSÉ - Overflow sur grands indices
pub fn search_overflow<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    let mut lo = 0;
    let mut hi = arr.len() - 1;  // ❌ Underflow si arr.is_empty()

    while lo <= hi {
        let mid = (lo + hi) / 2;  // ❌ OVERFLOW si lo + hi > usize::MAX
        // ...
    }
    None
}
// Pourquoi c'est faux : (lo + hi) peut overflow
// Fix : lo + (hi - lo) / 2
```

#### Refusée 2 : Boucle infinie

```rust
// ❌ REFUSÉ - Boucle infinie possible
pub fn lower_bound_infinite<T: Ord>(arr: &[T], target: &T) -> usize {
    let mut lo = 0;
    let mut hi = arr.len();

    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if arr[mid] < *target {
            lo = mid;  // ❌ ERREUR : devrait être mid + 1
        } else {
            hi = mid;
        }
    }
    lo
}
// Pourquoi c'est faux : Si arr[mid] < target et mid = lo, on ne progresse jamais
```

#### Refusée 3 : Off-by-one sur floor

```rust
// ❌ REFUSÉ - Off-by-one error
pub fn floor_wrong<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    let idx = lower_bound(arr, target);
    if idx < arr.len() && arr[idx] == *target {
        Some(idx)
    } else if idx > 0 {
        Some(idx)  // ❌ ERREUR : devrait être idx - 1
    } else {
        None
    }
}
// Pourquoi c'est faux : lower_bound donne le premier >= target
// floor veut le dernier <= target, donc idx - 1 si arr[idx] > target
```

---

### 4.6 Solution bonus de référence

```rust
/// Ternary search pour maximum d'une fonction unimodale
pub fn ternary_search_max<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
where
    F: Fn(f64) -> f64,
{
    let mut lo = lo;
    let mut hi = hi;

    while hi - lo > eps {
        let mid1 = lo + (hi - lo) / 3.0;
        let mid2 = hi - (hi - lo) / 3.0;

        if f(mid1) < f(mid2) {
            lo = mid1;
        } else {
            hi = mid2;
        }
    }
    lo + (hi - lo) / 2.0
}

/// Exponential search
pub fn exponential_search<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    if arr.is_empty() {
        return None;
    }
    if arr[0] == *target {
        return Some(0);
    }

    // Find range with exponential jumps
    let mut bound = 1;
    while bound < arr.len() && arr[bound] < *target {
        bound *= 2;
    }

    // Binary search in found range
    let lo = bound / 2;
    let hi = bound.min(arr.len() - 1);

    let sub = &arr[lo..=hi];
    search(sub, target).map(|i| i + lo)
}

/// Interpolation search
pub fn interpolation_search(arr: &[i64], target: i64) -> Option<usize> {
    if arr.is_empty() {
        return None;
    }

    let mut lo: usize = 0;
    let mut hi: usize = arr.len() - 1;

    while lo <= hi && target >= arr[lo] && target <= arr[hi] {
        if lo == hi {
            return if arr[lo] == target { Some(lo) } else { None };
        }

        // Interpolation formula
        let pos = lo + ((target - arr[lo]) as usize * (hi - lo))
                      / ((arr[hi] - arr[lo]) as usize);

        let pos = pos.min(hi).max(lo);

        if arr[pos] == target {
            return Some(pos);
        }
        if arr[pos] < target {
            lo = pos + 1;
        } else {
            if pos == 0 { break; }
            hi = pos - 1;
        }
    }
    None
}

/// Fractional cascading (simplified version)
pub fn fractional_cascade_search(arrays: &[&[i32]], target: i32) -> Vec<Option<usize>> {
    arrays.iter()
        .map(|arr| {
            let idx = lower_bound(*arr, &target);
            if idx < arr.len() && arr[idx] == target {
                Some(idx)
            } else {
                None
            }
        })
        .collect()
}
```

---

### 4.7 Solutions alternatives bonus

```rust
// Alternative : Ternary avec nombre d'itérations fixe
pub fn ternary_search_iterations<F>(lo: f64, hi: f64, iterations: usize, f: F) -> f64
where
    F: Fn(f64) -> f64,
{
    let mut lo = lo;
    let mut hi = hi;

    for _ in 0..iterations {
        let mid1 = lo + (hi - lo) / 3.0;
        let mid2 = hi - (hi - lo) / 3.0;

        if f(mid1) < f(mid2) {
            lo = mid1;
        } else {
            hi = mid2;
        }
    }
    (lo + hi) / 2.0
}
```

---

### 4.8 Solutions refusées bonus

```rust
// ❌ REFUSÉ - Mauvais calcul de position dans interpolation
pub fn interpolation_wrong(arr: &[i64], target: i64) -> Option<usize> {
    // ...
    let pos = (target - arr[lo]) / (arr[hi] - arr[lo]) * (hi - lo);
    // ❌ ERREUR : Division entière trop tôt, perte de précision
    // ...
}
// Fix : Multiplier d'abord, diviser ensuite
```

---

### 4.9 spec.json

```json
{
  "name": "the_price_is_right",
  "language": "rust",
  "secondary_language": "c",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse (variantes binary search)",
  "tags": ["binary_search", "algorithms", "phase1", "divide_conquer"],
  "passing_score": 70,

  "functions": [
    {
      "name": "search",
      "prototype": "pub fn search<T: Ord>(arr: &[T], target: &T) -> Option<usize>",
      "return_type": "Option<usize>"
    },
    {
      "name": "lower_bound",
      "prototype": "pub fn lower_bound<T: Ord>(arr: &[T], target: &T) -> usize",
      "return_type": "usize"
    },
    {
      "name": "upper_bound",
      "prototype": "pub fn upper_bound<T: Ord>(arr: &[T], target: &T) -> usize",
      "return_type": "usize"
    },
    {
      "name": "ceil",
      "prototype": "pub fn ceil<T: Ord>(arr: &[T], target: &T) -> Option<usize>",
      "return_type": "Option<usize>"
    },
    {
      "name": "floor",
      "prototype": "pub fn floor<T: Ord>(arr: &[T], target: &T) -> Option<usize>",
      "return_type": "Option<usize>"
    },
    {
      "name": "find_peak",
      "prototype": "pub fn find_peak(arr: &[i32]) -> usize",
      "return_type": "usize"
    },
    {
      "name": "search_rotated",
      "prototype": "pub fn search_rotated<T: Ord>(arr: &[T], target: &T) -> Option<usize>",
      "return_type": "Option<usize>"
    },
    {
      "name": "find_rotation_point",
      "prototype": "pub fn find_rotation_point<T: Ord>(arr: &[T]) -> usize",
      "return_type": "usize"
    },
    {
      "name": "isqrt",
      "prototype": "pub fn isqrt(n: u64) -> u64",
      "return_type": "u64"
    },
    {
      "name": "kth_smallest_matrix",
      "prototype": "pub fn kth_smallest_matrix(matrix: &[Vec<i32>], k: usize) -> i32",
      "return_type": "i32"
    },
    {
      "name": "binary_search_answer",
      "prototype": "pub fn binary_search_answer<F>(lo: i64, hi: i64, predicate: F) -> i64 where F: Fn(i64) -> bool",
      "return_type": "i64"
    },
    {
      "name": "binary_search_float",
      "prototype": "pub fn binary_search_float<F>(lo: f64, hi: f64, eps: f64, predicate: F) -> f64 where F: Fn(f64) -> bool",
      "return_type": "f64"
    }
  ],

  "driver": {
    "edge_cases": [
      {
        "name": "search_empty",
        "function": "search",
        "args": [[], 5],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "Tableau vide doit retourner None"
      },
      {
        "name": "search_single_found",
        "function": "search",
        "args": [[42], 42],
        "expected": 0
      },
      {
        "name": "search_single_not_found",
        "function": "search",
        "args": [[42], 7],
        "expected": null,
        "is_trap": true
      },
      {
        "name": "search_first_element",
        "function": "search",
        "args": [[1,3,5,7,9], 1],
        "expected": 0,
        "is_trap": true,
        "trap_explanation": "Boundary: premier élément"
      },
      {
        "name": "search_last_element",
        "function": "search",
        "args": [[1,3,5,7,9], 9],
        "expected": 4,
        "is_trap": true,
        "trap_explanation": "Boundary: dernier élément"
      },
      {
        "name": "lower_bound_duplicates",
        "function": "lower_bound",
        "args": [[1,2,2,2,3], 2],
        "expected": 1
      },
      {
        "name": "lower_bound_not_present",
        "function": "lower_bound",
        "args": [[1,3,5,7], 4],
        "expected": 2,
        "is_trap": true,
        "trap_explanation": "Élément absent, retourne position d'insertion"
      },
      {
        "name": "lower_bound_beyond",
        "function": "lower_bound",
        "args": [[1,3,5,7], 10],
        "expected": 4,
        "is_trap": true,
        "trap_explanation": "Tous les éléments sont plus petits"
      },
      {
        "name": "upper_bound_duplicates",
        "function": "upper_bound",
        "args": [[1,2,2,2,3], 2],
        "expected": 4
      },
      {
        "name": "peak_middle",
        "function": "find_peak",
        "args": [[1,3,5,7,6,4,2]],
        "expected": 3
      },
      {
        "name": "peak_at_start",
        "function": "find_peak",
        "args": [[7,6,5,4,3]],
        "expected": 0,
        "is_trap": true,
        "trap_explanation": "Peak au début (descente monotone)"
      },
      {
        "name": "peak_at_end",
        "function": "find_peak",
        "args": [[1,2,3,4,5]],
        "expected": 4,
        "is_trap": true,
        "trap_explanation": "Peak à la fin (montée monotone)"
      },
      {
        "name": "rotated_standard",
        "function": "search_rotated",
        "args": [[4,5,6,7,0,1,2], 0],
        "expected": 4
      },
      {
        "name": "rotated_not_rotated",
        "function": "search_rotated",
        "args": [[1,2,3,4,5], 3],
        "expected": 2,
        "is_trap": true,
        "trap_explanation": "Tableau non pivoté"
      },
      {
        "name": "rotation_point_standard",
        "function": "find_rotation_point",
        "args": [[4,5,6,7,0,1,2]],
        "expected": 4
      },
      {
        "name": "rotation_point_not_rotated",
        "function": "find_rotation_point",
        "args": [[1,2,3,4,5]],
        "expected": 0,
        "is_trap": true,
        "trap_explanation": "Non pivoté = rotation point à 0"
      },
      {
        "name": "isqrt_zero",
        "function": "isqrt",
        "args": [0],
        "expected": 0,
        "is_trap": true
      },
      {
        "name": "isqrt_perfect",
        "function": "isqrt",
        "args": [100],
        "expected": 10
      },
      {
        "name": "isqrt_imperfect",
        "function": "isqrt",
        "args": [99],
        "expected": 9,
        "is_trap": true,
        "trap_explanation": "Doit arrondir vers le bas"
      },
      {
        "name": "isqrt_large",
        "function": "isqrt",
        "args": [1000000000000],
        "expected": 1000000,
        "is_trap": true,
        "trap_explanation": "Attention overflow dans mid*mid"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 5000,
      "generators": [
        {
          "type": "sorted_array_int",
          "param_index": 0,
          "params": {
            "min_len": 0,
            "max_len": 10000,
            "min_val": -1000000,
            "max_val": 1000000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": [],
    "forbidden_functions": ["bsearch"],
    "check_complexity": true,
    "expected_complexity": "O(log n)",
    "check_memory": true,
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes (minimum 5)

#### Mutant A (Boundary) : Off-by-one dans lower_bound

```rust
/* Mutant A (Boundary) : Utilise <= au lieu de < */
pub fn lower_bound_mutant<T: Ord>(arr: &[T], target: &T) -> usize {
    let mut lo: usize = 0;
    let mut hi: usize = arr.len();

    while lo <= hi {  // ❌ ERREUR : devrait être lo < hi
        let mid = lo + (hi - lo) / 2;
        if arr[mid] < *target {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    lo
}
// Pourquoi c'est faux : Boucle infinie quand lo == hi == arr.len()
// Ce qui était pensé : "Je dois inclure le cas lo == hi"
```

#### Mutant B (Safety) : Pas de vérification tableau vide

```rust
/* Mutant B (Safety) : Pas de check pour tableau vide */
pub fn search_mutant<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    let mut lo: usize = 0;
    let mut hi: usize = arr.len() - 1;  // ❌ PANIC si arr.is_empty()

    while lo <= hi {
        let mid = lo + (hi - lo) / 2;
        match arr[mid].cmp(target) {
            std::cmp::Ordering::Equal => return Some(mid),
            std::cmp::Ordering::Less => lo = mid + 1,
            std::cmp::Ordering::Greater => {
                if mid == 0 { break; }
                hi = mid - 1;
            }
        }
    }
    None
}
// Pourquoi c'est faux : arr.len() - 1 underflow si arr est vide
// Ce qui était pensé : "Le tableau aura toujours au moins un élément"
```

#### Mutant C (Overflow) : Calcul de mid qui overflow

```rust
/* Mutant C (Overflow) : mid = (lo + hi) / 2 peut overflow */
pub fn search_overflow<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    if arr.is_empty() { return None; }

    let mut lo: usize = 0;
    let mut hi: usize = arr.len() - 1;

    while lo <= hi {
        let mid = (lo + hi) / 2;  // ❌ OVERFLOW si lo + hi > usize::MAX
        // ...
    }
    None
}
// Pourquoi c'est faux : Pour des indices très grands, lo + hi peut overflow
// Ce qui était pensé : "La somme de deux indices ne dépassera jamais usize::MAX"
// Fix : lo + (hi - lo) / 2
```

#### Mutant D (Logic) : Mauvaise direction dans rotated search

```rust
/* Mutant D (Logic) : Mauvaise condition pour le côté trié */
pub fn search_rotated_mutant<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    if arr.is_empty() { return None; }

    let mut lo: usize = 0;
    let mut hi: usize = arr.len() - 1;

    while lo <= hi {
        let mid = lo + (hi - lo) / 2;

        if arr[mid] == *target {
            return Some(mid);
        }

        // ❌ ERREUR : Utilise < au lieu de <=
        if arr[lo] < arr[mid] {  // Devrait être arr[lo] <= arr[mid]
            if arr[lo] <= *target && *target < arr[mid] {
                hi = mid - 1;
            } else {
                lo = mid + 1;
            }
        } else {
            // ...
        }
    }
    None
}
// Pourquoi c'est faux : Quand arr[lo] == arr[mid], on va dans le mauvais cas
// Ce qui était pensé : "< et <= c'est pareil ici"
```

#### Mutant E (Return) : isqrt retourne hi au lieu de lo

```rust
/* Mutant E (Return) : Retourne la mauvaise borne */
pub fn isqrt_mutant(n: u64) -> u64 {
    if n == 0 { return 0; }

    let mut lo: u64 = 1;
    let mut hi: u64 = n.min(3_037_000_499);

    while lo < hi {
        let mid = lo + (hi - lo + 1) / 2;
        if mid <= n / mid {
            lo = mid;
        } else {
            hi = mid - 1;
        }
    }
    hi  // ❌ ERREUR : devrait retourner lo
}
// Pourquoi c'est faux : À la fin de la boucle, lo == hi, mais hi peut avoir été décrémenté
// après un test qui retournait faux, donc hi peut être < sqrt(n)
// Ce qui était pensé : "lo et hi sont égaux à la fin, peu importe lequel"
```

#### Mutant F (Infinite Loop) : Template 3 sans +1

```rust
/* Mutant F (Infinite Loop) : Oubli du +1 dans template rightmost */
pub fn floor_mutant<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    if arr.is_empty() { return None; }

    let mut lo: usize = 0;
    let mut hi: usize = arr.len() - 1;

    while lo < hi {
        let mid = lo + (hi - lo) / 2;  // ❌ ERREUR : devrait être (hi - lo + 1) / 2
        if arr[mid] <= *target {
            lo = mid;  // Si mid == lo, on ne progresse jamais
        } else {
            hi = mid - 1;
        }
    }
    if arr[lo] <= *target { Some(lo) } else { None }
}
// Pourquoi c'est faux : Quand hi = lo + 1, mid = lo, et si arr[lo] <= target,
// on fait lo = mid = lo → boucle infinie
// Ce qui était pensé : "Le +1 n'est pas nécessaire"
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Le paradigme "Divide and Conquer"** : Diviser le problème en sous-problèmes plus petits
2. **Les 3 templates de binary search** : Standard, Leftmost, Rightmost
3. **La gestion des edge cases** : Tableau vide, élément absent, boundaries
4. **L'évitement des bugs classiques** : Overflow, off-by-one, boucle infinie
5. **L'application à des problèmes variés** : Rotated arrays, peak finding, search on answer

---

### 5.2 LDA — Traduction littérale

```
FONCTION search QUI RETOURNE UN OPTIONNEL D'ENTIER NON SIGNÉ ET PREND EN PARAMÈTRES arr QUI EST UNE TRANCHE ET target QUI EST UNE RÉFÉRENCE
DÉBUT FONCTION
    SI arr EST VIDE ALORS
        RETOURNER AUCUN
    FIN SI

    DÉCLARER lo COMME ENTIER NON SIGNÉ
    DÉCLARER hi COMME ENTIER NON SIGNÉ

    AFFECTER 0 À lo
    AFFECTER LA LONGUEUR DE arr MOINS 1 À hi

    TANT QUE lo EST INFÉRIEUR OU ÉGAL À hi FAIRE
        DÉCLARER mid COMME ENTIER NON SIGNÉ
        AFFECTER lo PLUS LA DIVISION ENTIÈRE DE hi MOINS lo PAR 2 À mid

        SI L'ÉLÉMENT À LA POSITION mid DANS arr EST ÉGAL À target ALORS
            RETOURNER QUELQUE(mid)
        SINON SI L'ÉLÉMENT À LA POSITION mid DANS arr EST INFÉRIEUR À target ALORS
            AFFECTER mid PLUS 1 À lo
        SINON
            SI mid EST ÉGAL À 0 ALORS
                SORTIR DE LA BOUCLE
            FIN SI
            AFFECTER mid MOINS 1 À hi
        FIN SI
    FIN TANT QUE

    RETOURNER AUCUN
FIN FONCTION
```

---

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : Binary Search Standard
---
1. VÉRIFIER si le tableau est vide
   |-- Si OUI : RETOURNER None (pas d'élément)

2. INITIALISER les bornes :
   |-- lo = 0 (début du tableau)
   |-- hi = longueur - 1 (fin du tableau)

3. BOUCLE tant que lo <= hi :
   a. CALCULER mid = lo + (hi - lo) / 2
      (Évite l'overflow de (lo + hi) / 2)

   b. COMPARER arr[mid] avec target :
      - CAS ÉGAL : RETOURNER mid (trouvé!)
      - CAS INFÉRIEUR : lo = mid + 1 (chercher à droite)
      - CAS SUPÉRIEUR : hi = mid - 1 (chercher à gauche)
        Note: Vérifier mid > 0 avant de décrémenter

4. RETOURNER None (pas trouvé)
```

---

### 5.2.3.1 Logique de Garde (Fail Fast)

```
FONCTION : search (arr, target)
---
INIT résultat = None

1. GARDE : tableau vide
   |-- VÉRIFIER arr.is_empty()
   |     SI OUI → RETOURNER None immédiatement
   |
   |-- Raison : Évite underflow sur arr.len() - 1

2. GARDE : élément unique
   |-- VÉRIFIER arr.len() == 1
   |     SI arr[0] == target → RETOURNER Some(0)
   |     SINON → RETOURNER None
   |
   |-- Raison : Optimisation pour cas trivial

3. TRAITEMENT PRINCIPAL :
   |-- Boucle de recherche binaire
   |-- À chaque itération, la taille du problème diminue de moitié
   |-- Maximum log2(n) itérations

4. RETOURNER résultat
```

---

### 5.3 Visualisation ASCII

#### L'invariant de la boucle

```
Recherche de 7 dans [1, 3, 5, 7, 9, 11, 13]

Itération 1:
┌───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 3 │ 5 │ 7 │ 9 │ 11│ 13│
└───┴───┴───┴───┴───┴───┴───┘
  ↑           ↑           ↑
  lo         mid          hi

  arr[mid] = 7 == target → TROUVÉ à index 3!

──────────────────────────────────────────

Recherche de 8 dans [1, 3, 5, 7, 9, 11, 13]

Itération 1:
┌───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 3 │ 5 │ 7 │ 9 │ 11│ 13│
└───┴───┴───┴───┴───┴───┴───┘
  ↑           ↑           ↑
  lo         mid          hi

  arr[mid] = 7 < 8 → chercher à droite, lo = mid + 1

Itération 2:
┌───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 3 │ 5 │ 7 │ 9 │ 11│ 13│
└───┴───┴───┴───┴───┴───┴───┘
              ↑   ↑       ↑
             old  lo     mid
                          hi

  arr[mid] = 11 > 8 → chercher à gauche, hi = mid - 1

Itération 3:
┌───┬───┬───┬───┬───┬───┬───┐
│ 1 │ 3 │ 5 │ 7 │ 9 │ 11│ 13│
└───┴───┴───┴───┴───┴───┴───┘
                  ↑
              lo=mid=hi

  arr[mid] = 9 > 8 → hi = mid - 1

  hi < lo → SORTIE DE BOUCLE → pas trouvé!
```

#### Les 3 Templates

```
TEMPLATE 1 : STANDARD (recherche exacte)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Condition : while lo <= hi
Calcul mid : lo + (hi - lo) / 2
Si < target : lo = mid + 1
Si > target : hi = mid - 1
Si == target : return mid

        [=======recherche=======]
         ↑                     ↑
        lo                    hi

TEMPLATE 2 : LEFTMOST (lower_bound)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Condition : while lo < hi
Calcul mid : lo + (hi - lo) / 2
Si < target : lo = mid + 1
Sinon : hi = mid
Return lo

        [<target][≥target]
                 ↑
               résultat

TEMPLATE 3 : RIGHTMOST (upper_bound - 1)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Condition : while lo < hi
Calcul mid : lo + (hi - lo + 1) / 2  ← ATTENTION AU +1!
Si ≤ target : lo = mid
Sinon : hi = mid - 1
Return lo

        [≤target][>target]
                ↑
              résultat
```

#### Rotated Array

```
Tableau original trié : [0, 1, 2, 4, 5, 6, 7]
                         ↑              ↑
                       début           fin

Après rotation de 4 positions :
                        [4, 5, 6, 7, 0, 1, 2]
                         ↑        ↑  ↑
                        début    max min
                                    (rotation point)

Deux parties triées :
┌─────────────┬─────────────┐
│ [4, 5, 6, 7]│ [0, 1, 2]   │
│   TRIÉE     │   TRIÉE     │
│  (partie 1) │  (partie 2) │
└─────────────┴─────────────┘
              ↑
        point de rotation
```

---

### 5.4 Les pièges en détail

| Piège | Description | Exemple | Solution |
|-------|-------------|---------|----------|
| **Overflow** | `(lo + hi) / 2` dépasse la capacité | `lo=2^62, hi=2^62` | `lo + (hi - lo) / 2` |
| **Underflow** | `arr.len() - 1` quand arr est vide | `[].len() - 1` | Vérifier `is_empty()` d'abord |
| **Off-by-one** | `<=` vs `<` dans la condition | Template 1 vs 2 | Choisir le bon template |
| **Boucle infinie** | `lo = mid` sans progression | Template 3 sans `+1` | Utiliser `(hi - lo + 1) / 2` |
| **Mauvais côté** | Chercher à gauche au lieu de droite | Rotated array | Identifier quelle moitié est triée |

---

### 5.5 Cours Complet : La Recherche Binaire

#### 5.5.1 Introduction

La recherche binaire est l'un des algorithmes les plus importants en informatique. Son principe est simple : à chaque étape, on élimine la moitié des candidats possibles.

**Complexité :**
- Temps : O(log n) — car on divise par 2 à chaque itération
- Espace : O(1) pour la version itérative, O(log n) pour la récursive

**Prérequis :** Le tableau DOIT être trié (ou avoir une propriété monotone).

#### 5.5.2 Pourquoi O(log n) ?

Si on a `n` éléments :
- Après 1 itération : n/2 éléments restants
- Après 2 itérations : n/4 éléments
- Après k itérations : n/2^k éléments

On s'arrête quand n/2^k = 1, donc k = log₂(n).

| n | Recherche linéaire | Binary search |
|---|-------------------|---------------|
| 10 | 10 | 4 |
| 100 | 100 | 7 |
| 1,000 | 1,000 | 10 |
| 1,000,000 | 1,000,000 | 20 |
| 1,000,000,000 | 1,000,000,000 | 30 |

#### 5.5.3 Les 3 Templates Fondamentaux

**Template 1 : Recherche exacte**
```
while lo <= hi
    mid = lo + (hi - lo) / 2
    if found: return
    if too_small: lo = mid + 1
    if too_big: hi = mid - 1
```

**Template 2 : Premier élément satisfaisant une condition**
```
while lo < hi
    mid = lo + (hi - lo) / 2
    if condition: hi = mid
    else: lo = mid + 1
return lo
```

**Template 3 : Dernier élément satisfaisant une condition**
```
while lo < hi
    mid = lo + (hi - lo + 1) / 2  // +1 important!
    if condition: lo = mid
    else: hi = mid - 1
return lo
```

#### 5.5.4 Binary Search on Answer

Parfois, on ne cherche pas dans un tableau mais dans un **espace de solutions**.

**Exemple : Capacité minimale pour livrer des colis en D jours**

On a des colis de poids `[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]` à livrer en 5 jours.
- Capacité minimale possible : `max(weights) = 10` (sinon un colis ne rentre pas)
- Capacité maximale utile : `sum(weights) = 55` (tout en un jour)

On fait une binary search sur la capacité :
- Si on peut livrer en ≤5 jours avec capacité `mid` → essayer plus petit
- Sinon → essayer plus grand

---

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais dangereux)                         │
├─────────────────────────────────────────────────────────────────┤
│ let mid = (lo + hi) / 2;                                        │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ let mid = lo + (hi - lo) / 2;                                   │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Overflow : lo + hi peut dépasser usize::MAX                   │
│ • Exemple : lo = 2^63, hi = 2^63 → overflow en u64              │
│ • La formule sûre évite ce problème mathématiquement            │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (panic possible)                                  │
├─────────────────────────────────────────────────────────────────┤
│ let hi = arr.len() - 1;  // Sans vérifier si arr est vide       │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ if arr.is_empty() { return None; }                              │
│ let hi = arr.len() - 1;                                         │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Underflow : 0usize - 1 = panic! en debug, wrap en release     │
│ • Comportement indéfini dangereux                               │
│ • Toujours vérifier le cas vide d'abord                         │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.7 Simulation avec trace d'exécution

#### Exemple : `search([1, 3, 5, 7, 9, 11, 13], 9)`

```
┌───────┬────────────────────────────────┬─────┬─────┬─────┬─────────┬──────────────────┐
│ Étape │ Instruction                    │ lo  │ hi  │ mid │ arr[mid]│ Explication      │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   1   │ AFFECTER 0 À lo                │  0  │  -  │  -  │    -    │ Initialisation   │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   2   │ AFFECTER 6 À hi                │  0  │  6  │  -  │    -    │ len-1 = 6        │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   3   │ lo <= hi ? (0 <= 6)            │  0  │  6  │  -  │    -    │ VRAI → entrer    │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   4   │ mid = 0 + (6-0)/2 = 3          │  0  │  6  │  3  │    7    │ Calcul mid       │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   5   │ 7 == 9 ?                       │  0  │  6  │  3  │    7    │ FAUX             │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   6   │ 7 < 9 ?                        │  0  │  6  │  3  │    7    │ VRAI → droite    │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   7   │ lo = mid + 1 = 4               │  4  │  6  │  3  │    -    │ Avancer lo       │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   8   │ lo <= hi ? (4 <= 6)            │  4  │  6  │  -  │    -    │ VRAI → continuer │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│   9   │ mid = 4 + (6-4)/2 = 5          │  4  │  6  │  5  │   11    │ Calcul mid       │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│  10   │ 11 == 9 ?                      │  4  │  6  │  5  │   11    │ FAUX             │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│  11   │ 11 < 9 ?                       │  4  │  6  │  5  │   11    │ FAUX → gauche    │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│  12   │ hi = mid - 1 = 4               │  4  │  4  │  5  │    -    │ Reculer hi       │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│  13   │ lo <= hi ? (4 <= 4)            │  4  │  4  │  -  │    -    │ VRAI → continuer │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│  14   │ mid = 4 + (4-4)/2 = 4          │  4  │  4  │  4  │    9    │ Calcul mid       │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│  15   │ 9 == 9 ?                       │  4  │  4  │  4  │    9    │ VRAI → TROUVÉ!   │
├───────┼────────────────────────────────┼─────┼─────┼─────┼─────────┼──────────────────┤
│  16   │ RETOURNER Some(4)              │  -  │  -  │  -  │    -    │ Résultat : 4     │
└───────┴────────────────────────────────┴─────┴─────┴─────┴─────────┴──────────────────┘
```

**Nombre d'itérations : 3** (log₂(7) ≈ 2.8)

---

### 5.8 Mnémotechniques

#### 🎮 MEME : "The Price is Right" — Higher or Lower

Imagine que tu joues au Juste Prix :
- Tu proposes un prix au milieu de ta fourchette
- Le présentateur dit "PLUS !" → tu cherches dans la moitié supérieure
- Le présentateur dit "MOINS !" → tu cherches dans la moitié inférieure

```
Prix entre 1€ et 1000€, tu cherches 731€

Toi: "500€ ?"     → "PLUS !"   [501-1000]
Toi: "750€ ?"     → "MOINS !"  [501-749]
Toi: "625€ ?"     → "PLUS !"   [626-749]
Toi: "687€ ?"     → "PLUS !"   [688-749]
Toi: "718€ ?"     → "PLUS !"   [719-749]
Toi: "734€ ?"     → "MOINS !"  [719-733]
Toi: "726€ ?"     → "PLUS !"   [727-733]
Toi: "730€ ?"     → "PLUS !"   [731-733]
Toi: "731€ ?"     → "GAGNÉ !"

9 essais pour trouver parmi 1000 possibilités !
```

#### 💀 MEME : "Thanos Snap" — Éliminer la moitié

Comme Thanos qui élimine la moitié de l'univers d'un claquement de doigts, binary search élimine la moitié des candidats à chaque itération.

```
*SNAP* → moitié éliminée
*SNAP* → encore la moitié
*SNAP* → encore...

En 6 snaps, Thanos élimine 98.4% de l'univers
En 6 itérations, binary search cherche parmi 64 éléments
```

#### 🔍 MEME : "Où est Charlie ?" — Version efficace

Méthode naïve : regarder chaque personne une par une → O(n)
Méthode binaire (si les gens sont triés par taille de bonnet) : → O(log n)

**"Dans un monde trié, Charlie se trouve en log(n) temps."**

---

### 5.9 Applications pratiques

| Application | Utilisation |
|-------------|-------------|
| **Git bisect** | Trouver le commit qui a introduit un bug |
| **Index B-Tree** | Bases de données (MySQL, PostgreSQL) |
| **Systèmes de fichiers** | Recherche dans les répertoires triés |
| **Routage réseau** | Longest prefix match pour les tables de routage |
| **Jeux vidéo** | Détection de collision avec spatial partitioning |
| **Finance** | Recherche de prix dans les order books |
| **ML/AI** | Hyperparameter tuning (binary search on answer) |
| **Compression** | Recherche dans les dictionnaires (LZ77, LZ78) |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Gravité | Comment l'éviter |
|---|-------|---------|------------------|
| 1 | Integer overflow dans `(lo + hi) / 2` | 🔴 Critique | Utiliser `lo + (hi - lo) / 2` |
| 2 | Underflow sur tableau vide | 🔴 Critique | Vérifier `is_empty()` d'abord |
| 3 | Off-by-one (`<` vs `<=`) | 🟡 Majeur | Choisir le bon template |
| 4 | Boucle infinie (template 3 sans +1) | 🔴 Critique | `(hi - lo + 1) / 2` pour rightmost |
| 5 | Mauvais côté dans rotated array | 🟡 Majeur | Identifier la partie triée |
| 6 | Oublier le cas "non trouvé" | 🟡 Majeur | Retourner None/−1 après la boucle |
| 7 | Confondre lower_bound et upper_bound | 🟢 Mineur | `<` vs `<=` dans la condition |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quelle est la complexité temporelle de la recherche binaire ?**

- A) O(1)
- B) O(log n)
- C) O(n)
- D) O(n log n)
- E) O(n²)
- F) O(2^n)
- G) Ça dépend des données
- H) O(√n)
- I) O(log² n)
- J) O(n/2)

**Réponse : B**

---

### Question 2
**Pourquoi utilise-t-on `lo + (hi - lo) / 2` au lieu de `(lo + hi) / 2` ?**

- A) C'est plus rapide
- B) C'est plus lisible
- C) Ça évite l'overflow quand lo + hi dépasse la capacité
- D) C'est une convention historique
- E) Ça donne un résultat plus précis
- F) C'est obligatoire en Rust
- G) Ça économise de la mémoire
- H) C'est pour la compatibilité avec le C
- I) Ça évite les nombres négatifs
- J) C'est équivalent, juste un style différent

**Réponse : C**

---

### Question 3
**Quelle est la différence entre lower_bound et upper_bound ?**

- A) lower_bound est plus rapide
- B) lower_bound retourne le premier élément ≥ target, upper_bound le premier > target
- C) lower_bound cherche en bas du tableau, upper_bound en haut
- D) Il n'y a pas de différence
- E) lower_bound est pour les entiers, upper_bound pour les flottants
- F) lower_bound utilise <, upper_bound utilise >
- G) lower_bound est inclusif, upper_bound est exclusif
- H) lower_bound retourne l'index, upper_bound retourne la valeur
- I) lower_bound est O(log n), upper_bound est O(n)
- J) lower_bound modifie le tableau, upper_bound non

**Réponse : B**

---

### Question 4
**Dans un tableau pivoté [4,5,6,7,0,1,2], où est le point de rotation ?**

- A) 0
- B) 1
- C) 2
- D) 3
- E) 4
- F) 5
- G) 6
- H) Le tableau n'est pas pivoté
- I) Il y a plusieurs points de rotation
- J) Impossible à déterminer

**Réponse : E** (index du minimum, qui est 0)

---

### Question 5
**Quelle condition provoque une boucle infinie dans le template rightmost ?**

- A) Utiliser `<=` au lieu de `<`
- B) Oublier le `+ 1` dans le calcul de mid
- C) Retourner hi au lieu de lo
- D) Inverser lo et hi au départ
- E) Utiliser `>` au lieu de `>=`
- F) Ne pas vérifier le tableau vide
- G) Utiliser une comparaison stricte
- H) Utiliser signed au lieu de unsigned
- I) Faire mid = hi au lieu de mid = lo
- J) A et B sont correctes

**Réponse : B**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Détail |
|---------|--------|
| **Nom** | the_price_is_right |
| **Concept** | Binary Search Variants |
| **Difficulté** | ★★★★★☆☆☆☆☆ (5/10) |
| **Variantes** | 13 (base) + 4 (bonus) |
| **Complexité** | O(log n) temps, O(1) espace |
| **Templates** | 3 (standard, leftmost, rightmost) |
| **Points clés** | Éviter overflow, choisir le bon template, gérer les edge cases |
| **MEME** | "The Price is Right" / "Thanos Snap" |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.1.2-the_price_is_right",
    "generated_at": "2026-01-11 00:00:00",

    "metadata": {
      "exercise_id": "1.1.2",
      "exercise_name": "the_price_is_right",
      "module": "1.1",
      "module_name": "Arrays & Sorting",
      "concept": "g",
      "concept_name": "Binary Search Variants",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 1,
      "difficulty": 5,
      "difficulty_stars": "★★★★★☆☆☆☆☆",
      "languages": ["rust", "c"],
      "duration_minutes": 60,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T3 O(log n)",
      "complexity_space": "S1 O(1)",
      "prerequisites": ["sorted_arrays", "comparisons", "loops"],
      "domains": ["Tri", "MD", "Algo"],
      "domains_bonus": ["Probas"],
      "tags": ["binary_search", "divide_conquer", "algorithms"],
      "meme_reference": "The Price is Right / Thanos Snap"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 Rust */",
      "references/ref_solution.c": "/* Section 4.3 C */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "alternatives/alt_recursive.rs": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 */",
      "mutants/mutant_c_overflow.rs": "/* Section 4.10 */",
      "mutants/mutant_d_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
      "mutants/mutant_f_infinite.rs": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "references/ref_solution.c",
        "references/ref_solution_bonus.rs",
        "alternatives/alt_recursive.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_safety.rs",
        "mutants/mutant_c_overflow.rs",
        "mutants/mutant_d_logic.rs",
        "mutants/mutant_e_return.rs",
        "mutants/mutant_f_infinite.rs"
      ]
    },

    "commands": {
      "validate_spec": "python3 hackbrain_engine_v22.py --validate-spec spec.json",
      "test_reference": "cargo test --release",
      "test_mutants": "python3 hackbrain_mutation_tester.py -r references/ -s spec.json --validate"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "L'excellence pédagogique ne se négocie pas"*
*Exercice généré automatiquement — Compatible ENGINE v22.1 + Mutation Tester*
