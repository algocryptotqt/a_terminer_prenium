# Exercice 1.1.3 : rush_hour_pointers

**Module :**
1.1 — Arrays & Sorting

**Concept :**
i — Two Pointers Technique

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (patterns two pointers + applications)

**Langages :**
Rust Edition 2024 + C (c17)

**Prérequis :**
- Manipulation de tableaux
- Boucles et conditions
- Notion de complexité O(n)

**Domaines :**
Algo, Struct, MD

**Durée estimée :**
60 min

**XP Base :**
150

**Complexité :**
T2-3 O(n) à O(n²) × S1 O(1)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `two_pointers.c`, `two_pointers.h`

**Fonctions autorisées :**
- Manipulation mémoire de base (swap)
- Aucune bibliothèque de tri externe

**Fonctions interdites :**
- `sort` de la STL (tu dois trier toi-même si nécessaire)
- Structures de données additionnelles (sauf pour `pair_with_sum_unsorted`)

---

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🎬 RUSH HOUR — Quand Deux Partenaires Font des Miracles**

*"Do you understand the words that are coming out of my mouth?!"* — Chris Tucker

Tu te souviens du film **Rush Hour** ? L'inspecteur Lee (Jackie Chan) vient de Hong Kong, Carter (Chris Tucker) vient de Los Angeles. Deux mondes opposés, deux styles différents... mais ensemble, ils sont **imbattables**.

C'est EXACTEMENT le principe des **Two Pointers** :
- **Lee** part de la gauche (index 0)
- **Carter** part de la droite (index n-1)
- Ils se rapprochent jusqu'à résoudre l'affaire

Parfois ils se suivent l'un l'autre (fast/slow), parfois ils encerclent le problème (opposite ends), parfois ils font un **"3-way split"** comme dans Rush Hour 3 avec Rémy le chauffeur de taxi !

**Ta mission :**

Implémenter une bibliothèque complète de techniques **Two Pointers** avec **13 algorithmes** différents, chacun utilisant une variante du pattern.

#### 1.2.2 Version Académique

La technique des deux pointeurs (Two Pointers) est un paradigme algorithmique qui utilise deux indices pour parcourir un tableau de manière coordonnée. Cette technique permet souvent de transformer une complexité O(n²) en O(n).

**Trois patterns principaux :**
1. **Opposite Ends** : Un pointeur à chaque extrémité, ils convergent vers le centre
2. **Same Direction** : Deux pointeurs qui avancent ensemble (slow/fast)
3. **Dutch Flag** : Trois pointeurs pour partitionner en 3 groupes

---

**Entrée :**
- `arr` : slice/tableau (trié ou non selon la fonction)
- `target` : valeur cible (pour les problèmes de somme)

**Sortie :**
- Indices, nouvelles longueurs, ou modifications in-place selon la fonction

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  0 ≤ arr.len() ≤ 10⁵                                            │
│  Modifications in-place obligatoires (sauf exceptions notées)   │
│  Espace auxiliaire O(1) sauf pair_with_sum_unsorted O(n)        │
│  Pas de duplicats dans les résultats de three_sum               │
└─────────────────────────────────────────────────────────────────┘
```

---

### 1.3 Prototype

#### Rust (Edition 2024)

```rust
pub mod two_pointers {
    /// Trouve une paire avec somme donnée dans un tableau TRIÉ
    /// Retourne les indices (i, j) tels que arr[i] + arr[j] == target
    pub fn pair_with_sum(arr: &[i32], target: i32) -> Option<(usize, usize)>;

    /// Trouve une paire avec somme donnée dans un tableau NON TRIÉ
    /// Utilise un HashSet, donc O(n) espace
    pub fn pair_with_sum_unsorted(arr: &[i32], target: i32) -> Option<(usize, usize)>;

    /// Trouve tous les triplets uniques qui somment à zéro
    pub fn three_sum(arr: &mut [i32]) -> Vec<[i32; 3]>;

    /// Trouve le triplet dont la somme est la plus proche de target
    pub fn three_sum_closest(arr: &mut [i32], target: i32) -> i32;

    /// Container With Most Water : aire maximale entre deux lignes
    pub fn max_area(heights: &[i32]) -> i64;

    /// Trapping Rain Water : quantité d'eau piégée
    pub fn trap_water(heights: &[i32]) -> i64;

    /// Supprime les duplicats d'un tableau trié in-place
    /// Retourne la nouvelle longueur
    pub fn remove_duplicates(arr: &mut [i32]) -> usize;

    /// Dutch National Flag : partitionne en 3 groupes (0s, 1s, 2s)
    pub fn dutch_flag(arr: &mut [i32]);

    /// Déplace tous les zéros à la fin en gardant l'ordre relatif
    pub fn move_zeros(arr: &mut [i32]);

    /// Vérifie si le tableau est un palindrome
    pub fn is_palindrome(arr: &[i32]) -> bool;

    /// Inverse un segment [start, end] du tableau
    pub fn reverse_segment<T>(arr: &mut [T], start: usize, end: usize);

    /// Fusionne deux tableaux triés (arr1 a de la place pour arr2)
    pub fn merge_sorted(arr1: &mut [i32], len1: usize, arr2: &[i32]);

    /// Trouve un sous-tableau contigu avec somme donnée (nombres positifs)
    pub fn subarray_sum(arr: &[i32], target: i32) -> Option<(usize, usize)>;
}
```

#### C (c17)

```c
#ifndef TWO_POINTERS_H
# define TWO_POINTERS_H

# include <stddef.h>
# include <stdbool.h>

typedef struct {
    size_t first;
    size_t second;
    bool found;
} pair_result_t;

typedef struct {
    int values[3];
} triplet_t;

typedef struct {
    triplet_t *triplets;
    size_t count;
    size_t capacity;
} triplet_list_t;

// Paire avec somme (tableau trié)
pair_result_t tp_pair_with_sum(const int *arr, size_t len, int target);

// Paire avec somme (tableau non trié) - alloue un HashSet interne
pair_result_t tp_pair_with_sum_unsorted(const int *arr, size_t len, int target);

// Three sum - retourne liste de triplets (caller doit free)
triplet_list_t tp_three_sum(int *arr, size_t len);
void tp_free_triplets(triplet_list_t *list);

// Three sum closest
int tp_three_sum_closest(int *arr, size_t len, int target);

// Container with most water
long long tp_max_area(const int *heights, size_t len);

// Trapping rain water
long long tp_trap_water(const int *heights, size_t len);

// Remove duplicates (retourne nouvelle longueur)
size_t tp_remove_duplicates(int *arr, size_t len);

// Dutch flag partition
void tp_dutch_flag(int *arr, size_t len);

// Move zeros
void tp_move_zeros(int *arr, size_t len);

// Is palindrome
bool tp_is_palindrome(const int *arr, size_t len);

// Reverse segment
void tp_reverse_segment(int *arr, size_t start, size_t end);

// Merge sorted arrays
void tp_merge_sorted(int *arr1, size_t len1, const int *arr2, size_t len2);

// Subarray sum
pair_result_t tp_subarray_sum(const int *arr, size_t len, int target);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Fun Facts

**🎯 L'origine du Dutch National Flag**

Le problème du "Dutch National Flag" a été formulé par Edsger Dijkstra (le créateur de l'algorithme de plus court chemin). Le drapeau néerlandais a trois bandes : rouge, blanc, bleu. L'algorithme partitionne un tableau en trois groupes, exactement comme les bandes du drapeau !

**📊 LeetCode loves Two Pointers**

Sur LeetCode, plus de **150 problèmes** utilisent la technique Two Pointers. C'est l'un des patterns les plus fréquents en entretien technique chez Google, Facebook, Amazon.

| Pattern | Nombre de problèmes LeetCode |
|---------|------------------------------|
| Two Pointers | 150+ |
| Sliding Window | 100+ |
| Binary Search | 200+ |

**🚀 Optimisation spectaculaire**

Le problème "Container With Most Water" :
- Approche brute force : O(n²) — teste tous les paires
- Two Pointers : O(n) — une seule passe !

Pour n = 100,000 : **10 milliards vs 100,000 opérations**

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation |
|--------|-------------|
| **Backend Developer** | Merge de streams triés (logs, events) |
| **Data Engineer** | Merge-sort externe pour gros fichiers |
| **Game Dev** | Détection collision avec sweep line |
| **Finance** | Analyse de séries temporelles (sliding window) |
| **DevOps** | Diff de fichiers (comme git diff) |
| **ML Engineer** | Feature engineering avec fenêtres glissantes |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
two_pointers.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 13 tests
test test_pair_sum_sorted ... ok
test test_pair_sum_unsorted ... ok
test test_three_sum ... ok
test test_three_sum_closest ... ok
test test_max_area ... ok
test test_trap_water ... ok
test test_remove_duplicates ... ok
test test_dutch_flag ... ok
test test_move_zeros ... ok
test test_is_palindrome ... ok
test test_reverse_segment ... ok
test test_merge_sorted ... ok
test test_subarray_sum ... ok

test result: ok. 13 passed; 0 failed
```

---

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n) strict pour tous les problèmes

**Space Complexity attendue :**
O(1) strict — même pour three_sum (output excepté)

**Domaines Bonus :**
`MD, DP`

#### 3.1.1 Consigne Bonus

**🎬 RUSH HOUR 4: THE IMPOSSIBLE MISSIONS**

Carter et Lee sont de retour pour des missions encore plus complexes !

1. **4Sum** : Trouve tous les quadruplets qui somment à target
2. **Longest Substring Without Repeating** : Sliding window sur chaînes
3. **Minimum Window Substring** : Trouve la plus petite fenêtre contenant tous les caractères
4. **Sort Colors Extended** : Partition en K groupes (généralisation Dutch Flag)

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  4Sum : O(n³) temps, O(1) espace auxiliaire                     │
│  Longest Substring : O(n) temps, O(k) espace (k = alphabet)     │
│  Minimum Window : O(n) temps, O(k) espace                       │
│  K-way Partition : O(n) temps, O(1) espace                      │
└─────────────────────────────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
/// Trouve tous les quadruplets uniques qui somment à target
pub fn four_sum(arr: &mut [i32], target: i32) -> Vec<[i32; 4]>;

/// Longueur du plus long substring sans caractères répétés
pub fn longest_unique_substring(s: &str) -> usize;

/// Plus petite fenêtre contenant tous les caractères de pattern
pub fn min_window(s: &str, pattern: &str) -> Option<String>;

/// Partition en K groupes (0..K-1)
pub fn k_way_partition(arr: &mut [i32], k: i32);
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Max éléments combinés | 3 (three_sum) | 4 (four_sum) |
| Types de données | Entiers | Chaînes aussi |
| Partitions | 3 groupes | K groupes |
| Fenêtres | Fixes | Variables avec contraintes |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `pair_sum_found` | `[2,7,11,15], 9` | `Some((0,1))` | 3 | Non |
| `pair_sum_not_found` | `[2,7,11,15], 5` | `None` | 3 | Oui |
| `pair_sum_same_element` | `[3,3], 6` | `Some((0,1))` | 2 | Boundary |
| `pair_unsorted` | `[3,2,4], 6` | `Some((1,2))` | 4 | Non |
| `three_sum_basic` | `[-1,0,1,2,-1,-4]` | `[[-1,-1,2],[-1,0,1]]` | 5 | Non |
| `three_sum_no_dups` | `[0,0,0,0]` | `[[0,0,0]]` | 3 | Duplicate |
| `three_sum_empty` | `[1,2]` | `[]` | 2 | Edge |
| `three_sum_closest` | `[-1,2,1,-4], 1` | `2` | 4 | Non |
| `max_area_basic` | `[1,8,6,2,5,4,8,3,7]` | `49` | 4 | Non |
| `max_area_two` | `[1,1]` | `1` | 2 | Edge |
| `trap_water_basic` | `[0,1,0,2,1,0,1,3,2,1,2,1]` | `6` | 5 | Non |
| `trap_water_hill` | `[4,2,0,3,2,5]` | `9` | 4 | Non |
| `remove_dup_basic` | `[1,1,2,2,3]` | `3, [1,2,3,_,_]` | 4 | Non |
| `remove_dup_no_dup` | `[1,2,3]` | `3` | 2 | Edge |
| `dutch_flag_basic` | `[2,0,2,1,1,0]` | `[0,0,1,1,2,2]` | 4 | Non |
| `dutch_flag_sorted` | `[0,0,1,1,2,2]` | `[0,0,1,1,2,2]` | 2 | Edge |
| `move_zeros_basic` | `[0,1,0,3,12]` | `[1,3,12,0,0]` | 3 | Non |
| `move_zeros_all` | `[0,0,0]` | `[0,0,0]` | 2 | Edge |
| `is_palindrome_yes` | `[1,2,3,2,1]` | `true` | 2 | Non |
| `is_palindrome_no` | `[1,2,3]` | `false` | 2 | Non |
| `merge_sorted_basic` | `[1,2,3,0,0,0], [2,5,6]` | `[1,2,2,3,5,6]` | 5 | Non |
| `subarray_sum_found` | `[1,4,20,3,10,5], 33` | `Some((2,4))` | 4 | Non |
| `subarray_sum_not` | `[1,2,3], 100` | `None` | 2 | Edge |

**Total : 100 points**

---

### 4.2 main.c de test

```c
#include <stdio.h>
#include <assert.h>
#include <string.h>
#include "two_pointers.h"

void test_pair_sum(void) {
    int arr[] = {2, 7, 11, 15};
    pair_result_t result = tp_pair_with_sum(arr, 4, 9);
    assert(result.found == true);
    assert(result.first == 0 && result.second == 1);

    result = tp_pair_with_sum(arr, 4, 5);
    assert(result.found == false);
    printf("test_pair_sum: OK\n");
}

void test_three_sum(void) {
    int arr[] = {-1, 0, 1, 2, -1, -4};
    triplet_list_t result = tp_three_sum(arr, 6);
    assert(result.count == 2);
    tp_free_triplets(&result);
    printf("test_three_sum: OK\n");
}

void test_max_area(void) {
    int heights[] = {1, 8, 6, 2, 5, 4, 8, 3, 7};
    assert(tp_max_area(heights, 9) == 49);

    int heights2[] = {1, 1};
    assert(tp_max_area(heights2, 2) == 1);
    printf("test_max_area: OK\n");
}

void test_trap_water(void) {
    int heights[] = {0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1};
    assert(tp_trap_water(heights, 12) == 6);

    int heights2[] = {4, 2, 0, 3, 2, 5};
    assert(tp_trap_water(heights2, 6) == 9);
    printf("test_trap_water: OK\n");
}

void test_dutch_flag(void) {
    int arr[] = {2, 0, 2, 1, 1, 0};
    tp_dutch_flag(arr, 6);
    int expected[] = {0, 0, 1, 1, 2, 2};
    assert(memcmp(arr, expected, sizeof(expected)) == 0);
    printf("test_dutch_flag: OK\n");
}

void test_remove_duplicates(void) {
    int arr[] = {1, 1, 2, 2, 2, 3, 4, 4, 5};
    size_t len = tp_remove_duplicates(arr, 9);
    assert(len == 5);
    assert(arr[0] == 1 && arr[1] == 2 && arr[2] == 3 && arr[3] == 4 && arr[4] == 5);
    printf("test_remove_duplicates: OK\n");
}

void test_move_zeros(void) {
    int arr[] = {0, 1, 0, 3, 12};
    tp_move_zeros(arr, 5);
    int expected[] = {1, 3, 12, 0, 0};
    assert(memcmp(arr, expected, sizeof(expected)) == 0);
    printf("test_move_zeros: OK\n");
}

void test_merge_sorted(void) {
    int arr1[] = {1, 2, 3, 0, 0, 0};
    int arr2[] = {2, 5, 6};
    tp_merge_sorted(arr1, 3, arr2, 3);
    int expected[] = {1, 2, 2, 3, 5, 6};
    assert(memcmp(arr1, expected, sizeof(expected)) == 0);
    printf("test_merge_sorted: OK\n");
}

int main(void) {
    test_pair_sum();
    test_three_sum();
    test_max_area();
    test_trap_water();
    test_dutch_flag();
    test_remove_duplicates();
    test_move_zeros();
    test_merge_sorted();

    printf("\nTous les tests passent!\n");
    return 0;
}
```

---

### 4.3 Solution de référence

#### Rust

```rust
pub mod two_pointers {
    use std::collections::HashSet;

    /// Pair with sum - sorted array
    pub fn pair_with_sum(arr: &[i32], target: i32) -> Option<(usize, usize)> {
        if arr.len() < 2 {
            return None;
        }

        let mut left = 0;
        let mut right = arr.len() - 1;

        while left < right {
            let sum = arr[left] + arr[right];
            if sum == target {
                return Some((left, right));
            } else if sum < target {
                left += 1;
            } else {
                right -= 1;
            }
        }
        None
    }

    /// Pair with sum - unsorted array (uses HashSet)
    pub fn pair_with_sum_unsorted(arr: &[i32], target: i32) -> Option<(usize, usize)> {
        use std::collections::HashMap;

        let mut seen: HashMap<i32, usize> = HashMap::new();

        for (i, &num) in arr.iter().enumerate() {
            let complement = target - num;
            if let Some(&j) = seen.get(&complement) {
                return Some((j, i));
            }
            seen.insert(num, i);
        }
        None
    }

    /// Three sum - find all unique triplets summing to zero
    pub fn three_sum(arr: &mut [i32]) -> Vec<[i32; 3]> {
        let mut result = Vec::new();
        let n = arr.len();

        if n < 3 {
            return result;
        }

        arr.sort();

        for i in 0..n - 2 {
            // Skip duplicates for first element
            if i > 0 && arr[i] == arr[i - 1] {
                continue;
            }

            let mut left = i + 1;
            let mut right = n - 1;

            while left < right {
                let sum = arr[i] + arr[left] + arr[right];

                if sum == 0 {
                    result.push([arr[i], arr[left], arr[right]]);

                    // Skip duplicates
                    while left < right && arr[left] == arr[left + 1] {
                        left += 1;
                    }
                    while left < right && arr[right] == arr[right - 1] {
                        right -= 1;
                    }

                    left += 1;
                    right -= 1;
                } else if sum < 0 {
                    left += 1;
                } else {
                    right -= 1;
                }
            }
        }
        result
    }

    /// Three sum closest
    pub fn three_sum_closest(arr: &mut [i32], target: i32) -> i32 {
        if arr.len() < 3 {
            return 0;
        }

        arr.sort();
        let n = arr.len();
        let mut closest = arr[0] + arr[1] + arr[2];

        for i in 0..n - 2 {
            let mut left = i + 1;
            let mut right = n - 1;

            while left < right {
                let sum = arr[i] + arr[left] + arr[right];

                if (sum - target).abs() < (closest - target).abs() {
                    closest = sum;
                }

                if sum == target {
                    return sum;
                } else if sum < target {
                    left += 1;
                } else {
                    right -= 1;
                }
            }
        }
        closest
    }

    /// Container with most water
    pub fn max_area(heights: &[i32]) -> i64 {
        if heights.len() < 2 {
            return 0;
        }

        let mut left = 0;
        let mut right = heights.len() - 1;
        let mut max_area: i64 = 0;

        while left < right {
            let width = (right - left) as i64;
            let height = heights[left].min(heights[right]) as i64;
            let area = width * height;
            max_area = max_area.max(area);

            if heights[left] < heights[right] {
                left += 1;
            } else {
                right -= 1;
            }
        }
        max_area
    }

    /// Trapping rain water
    pub fn trap_water(heights: &[i32]) -> i64 {
        if heights.len() < 3 {
            return 0;
        }

        let mut left = 0;
        let mut right = heights.len() - 1;
        let mut left_max = heights[left];
        let mut right_max = heights[right];
        let mut water: i64 = 0;

        while left < right {
            if left_max < right_max {
                left += 1;
                left_max = left_max.max(heights[left]);
                water += (left_max - heights[left]) as i64;
            } else {
                right -= 1;
                right_max = right_max.max(heights[right]);
                water += (right_max - heights[right]) as i64;
            }
        }
        water
    }

    /// Remove duplicates from sorted array
    pub fn remove_duplicates(arr: &mut [i32]) -> usize {
        if arr.is_empty() {
            return 0;
        }

        let mut slow = 0;

        for fast in 1..arr.len() {
            if arr[fast] != arr[slow] {
                slow += 1;
                arr[slow] = arr[fast];
            }
        }
        slow + 1
    }

    /// Dutch National Flag
    pub fn dutch_flag(arr: &mut [i32]) {
        if arr.is_empty() {
            return;
        }

        let mut low = 0;
        let mut mid = 0;
        let mut high = arr.len() - 1;

        while mid <= high {
            match arr[mid] {
                0 => {
                    arr.swap(low, mid);
                    low += 1;
                    mid += 1;
                }
                1 => {
                    mid += 1;
                }
                2 => {
                    arr.swap(mid, high);
                    if high == 0 {
                        break;
                    }
                    high -= 1;
                }
                _ => mid += 1,
            }
        }
    }

    /// Move zeros to end
    pub fn move_zeros(arr: &mut [i32]) {
        let mut slow = 0;

        for fast in 0..arr.len() {
            if arr[fast] != 0 {
                arr.swap(slow, fast);
                slow += 1;
            }
        }
    }

    /// Check if palindrome
    pub fn is_palindrome(arr: &[i32]) -> bool {
        if arr.is_empty() {
            return true;
        }

        let mut left = 0;
        let mut right = arr.len() - 1;

        while left < right {
            if arr[left] != arr[right] {
                return false;
            }
            left += 1;
            right -= 1;
        }
        true
    }

    /// Reverse segment
    pub fn reverse_segment<T>(arr: &mut [T], start: usize, end: usize) {
        if start >= end || start >= arr.len() || end >= arr.len() {
            return;
        }

        let mut left = start;
        let mut right = end;

        while left < right {
            arr.swap(left, right);
            left += 1;
            right -= 1;
        }
    }

    /// Merge sorted arrays (arr1 has space for arr2 at the end)
    pub fn merge_sorted(arr1: &mut [i32], len1: usize, arr2: &[i32]) {
        if arr2.is_empty() {
            return;
        }

        let len2 = arr2.len();
        let mut p1 = len1 as isize - 1;
        let mut p2 = len2 as isize - 1;
        let mut p = (len1 + len2) as isize - 1;

        while p2 >= 0 {
            if p1 >= 0 && arr1[p1 as usize] > arr2[p2 as usize] {
                arr1[p as usize] = arr1[p1 as usize];
                p1 -= 1;
            } else {
                arr1[p as usize] = arr2[p2 as usize];
                p2 -= 1;
            }
            p -= 1;
        }
    }

    /// Subarray sum (positive numbers only)
    pub fn subarray_sum(arr: &[i32], target: i32) -> Option<(usize, usize)> {
        if arr.is_empty() {
            return None;
        }

        let mut left = 0;
        let mut sum = 0;

        for right in 0..arr.len() {
            sum += arr[right];

            while sum > target && left < right {
                sum -= arr[left];
                left += 1;
            }

            if sum == target {
                return Some((left, right));
            }
        }
        None
    }
}
```

#### C

```c
#include "two_pointers.h"
#include <stdlib.h>
#include <string.h>

static void swap(int *a, int *b) {
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

static int compare_int(const void *a, const void *b) {
    return *(const int*)a - *(const int*)b;
}

pair_result_t tp_pair_with_sum(const int *arr, size_t len, int target) {
    pair_result_t result = {0, 0, false};

    if (arr == NULL || len < 2) return result;

    size_t left = 0;
    size_t right = len - 1;

    while (left < right) {
        int sum = arr[left] + arr[right];
        if (sum == target) {
            result.first = left;
            result.second = right;
            result.found = true;
            return result;
        } else if (sum < target) {
            left++;
        } else {
            right--;
        }
    }
    return result;
}

pair_result_t tp_pair_with_sum_unsorted(const int *arr, size_t len, int target) {
    pair_result_t result = {0, 0, false};
    if (arr == NULL || len < 2) return result;

    // Simple O(n²) approach for C (avoiding hash table complexity)
    for (size_t i = 0; i < len - 1; i++) {
        for (size_t j = i + 1; j < len; j++) {
            if (arr[i] + arr[j] == target) {
                result.first = i;
                result.second = j;
                result.found = true;
                return result;
            }
        }
    }
    return result;
}

triplet_list_t tp_three_sum(int *arr, size_t len) {
    triplet_list_t result = {NULL, 0, 0};

    if (arr == NULL || len < 3) return result;

    qsort(arr, len, sizeof(int), compare_int);

    result.capacity = 16;
    result.triplets = malloc(result.capacity * sizeof(triplet_t));

    for (size_t i = 0; i < len - 2; i++) {
        if (i > 0 && arr[i] == arr[i - 1]) continue;

        size_t left = i + 1;
        size_t right = len - 1;

        while (left < right) {
            int sum = arr[i] + arr[left] + arr[right];

            if (sum == 0) {
                if (result.count >= result.capacity) {
                    result.capacity *= 2;
                    result.triplets = realloc(result.triplets,
                                              result.capacity * sizeof(triplet_t));
                }
                result.triplets[result.count].values[0] = arr[i];
                result.triplets[result.count].values[1] = arr[left];
                result.triplets[result.count].values[2] = arr[right];
                result.count++;

                while (left < right && arr[left] == arr[left + 1]) left++;
                while (left < right && arr[right] == arr[right - 1]) right--;

                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return result;
}

void tp_free_triplets(triplet_list_t *list) {
    if (list && list->triplets) {
        free(list->triplets);
        list->triplets = NULL;
        list->count = 0;
        list->capacity = 0;
    }
}

int tp_three_sum_closest(int *arr, size_t len, int target) {
    if (arr == NULL || len < 3) return 0;

    qsort(arr, len, sizeof(int), compare_int);
    int closest = arr[0] + arr[1] + arr[2];

    for (size_t i = 0; i < len - 2; i++) {
        size_t left = i + 1;
        size_t right = len - 1;

        while (left < right) {
            int sum = arr[i] + arr[left] + arr[right];

            if (abs(sum - target) < abs(closest - target)) {
                closest = sum;
            }

            if (sum == target) return sum;
            else if (sum < target) left++;
            else right--;
        }
    }
    return closest;
}

long long tp_max_area(const int *heights, size_t len) {
    if (heights == NULL || len < 2) return 0;

    size_t left = 0;
    size_t right = len - 1;
    long long max_area = 0;

    while (left < right) {
        long long width = right - left;
        int height = (heights[left] < heights[right]) ? heights[left] : heights[right];
        long long area = width * height;
        if (area > max_area) max_area = area;

        if (heights[left] < heights[right]) left++;
        else right--;
    }
    return max_area;
}

long long tp_trap_water(const int *heights, size_t len) {
    if (heights == NULL || len < 3) return 0;

    size_t left = 0;
    size_t right = len - 1;
    int left_max = heights[left];
    int right_max = heights[right];
    long long water = 0;

    while (left < right) {
        if (left_max < right_max) {
            left++;
            if (heights[left] > left_max) left_max = heights[left];
            water += left_max - heights[left];
        } else {
            right--;
            if (heights[right] > right_max) right_max = heights[right];
            water += right_max - heights[right];
        }
    }
    return water;
}

size_t tp_remove_duplicates(int *arr, size_t len) {
    if (arr == NULL || len == 0) return 0;

    size_t slow = 0;

    for (size_t fast = 1; fast < len; fast++) {
        if (arr[fast] != arr[slow]) {
            slow++;
            arr[slow] = arr[fast];
        }
    }
    return slow + 1;
}

void tp_dutch_flag(int *arr, size_t len) {
    if (arr == NULL || len == 0) return;

    size_t low = 0;
    size_t mid = 0;
    size_t high = len - 1;

    while (mid <= high) {
        if (arr[mid] == 0) {
            swap(&arr[low], &arr[mid]);
            low++;
            mid++;
        } else if (arr[mid] == 1) {
            mid++;
        } else {
            swap(&arr[mid], &arr[high]);
            if (high == 0) break;
            high--;
        }
    }
}

void tp_move_zeros(int *arr, size_t len) {
    if (arr == NULL || len == 0) return;

    size_t slow = 0;

    for (size_t fast = 0; fast < len; fast++) {
        if (arr[fast] != 0) {
            swap(&arr[slow], &arr[fast]);
            slow++;
        }
    }
}

bool tp_is_palindrome(const int *arr, size_t len) {
    if (arr == NULL || len == 0) return true;

    size_t left = 0;
    size_t right = len - 1;

    while (left < right) {
        if (arr[left] != arr[right]) return false;
        left++;
        right--;
    }
    return true;
}

void tp_reverse_segment(int *arr, size_t start, size_t end) {
    if (arr == NULL || start >= end) return;

    while (start < end) {
        swap(&arr[start], &arr[end]);
        start++;
        end--;
    }
}

void tp_merge_sorted(int *arr1, size_t len1, const int *arr2, size_t len2) {
    if (arr1 == NULL || arr2 == NULL || len2 == 0) return;

    ssize_t p1 = (ssize_t)len1 - 1;
    ssize_t p2 = (ssize_t)len2 - 1;
    ssize_t p = (ssize_t)(len1 + len2) - 1;

    while (p2 >= 0) {
        if (p1 >= 0 && arr1[p1] > arr2[p2]) {
            arr1[p] = arr1[p1];
            p1--;
        } else {
            arr1[p] = arr2[p2];
            p2--;
        }
        p--;
    }
}

pair_result_t tp_subarray_sum(const int *arr, size_t len, int target) {
    pair_result_t result = {0, 0, false};

    if (arr == NULL || len == 0) return result;

    size_t left = 0;
    int sum = 0;

    for (size_t right = 0; right < len; right++) {
        sum += arr[right];

        while (sum > target && left < right) {
            sum -= arr[left];
            left++;
        }

        if (sum == target) {
            result.first = left;
            result.second = right;
            result.found = true;
            return result;
        }
    }
    return result;
}
```

---

### 4.4 Solutions alternatives acceptées

#### Alternative 1 : Three sum avec HashSet

```rust
pub fn three_sum_hash(arr: &mut [i32]) -> Vec<[i32; 3]> {
    use std::collections::HashSet;

    arr.sort();
    let mut result = Vec::new();
    let mut seen_first: HashSet<i32> = HashSet::new();

    for i in 0..arr.len().saturating_sub(2) {
        if seen_first.contains(&arr[i]) { continue; }
        seen_first.insert(arr[i]);

        let target = -arr[i];
        let mut seen: HashSet<i32> = HashSet::new();

        for j in (i + 1)..arr.len() {
            let complement = target - arr[j];
            if seen.contains(&complement) {
                let triplet = [arr[i], complement, arr[j]];
                if !result.contains(&triplet) {
                    result.push(triplet);
                }
            }
            seen.insert(arr[j]);
        }
    }
    result
}
```

---

### 4.5 Solutions refusées (avec explications)

#### Refusée 1 : Dutch flag sans gérer high = 0

```rust
// ❌ REFUSÉ - Underflow possible
pub fn dutch_flag_wrong(arr: &mut [i32]) {
    let mut low = 0;
    let mut mid = 0;
    let mut high = arr.len() - 1;  // ❌ Panic si arr.is_empty()

    while mid <= high {
        match arr[mid] {
            2 => {
                arr.swap(mid, high);
                high -= 1;  // ❌ Underflow si high = 0
            }
            // ...
        }
    }
}
// Fix : Vérifier arr.is_empty() et high == 0 avant de décrémenter
```

#### Refusée 2 : Trap water sans deux pointeurs

```rust
// ❌ REFUSÉ - O(n) espace au lieu de O(1)
pub fn trap_water_wrong(heights: &[i32]) -> i64 {
    let n = heights.len();
    let mut left_max = vec![0; n];
    let mut right_max = vec![0; n];

    // Pré-calcul des max gauche/droite → O(n) espace
    // ...
}
// Pourquoi c'est refusé : La contrainte demande O(1) espace
```

---

### 4.6 Solution bonus de référence

```rust
/// Four sum
pub fn four_sum(arr: &mut [i32], target: i32) -> Vec<[i32; 4]> {
    let mut result = Vec::new();
    let n = arr.len();

    if n < 4 {
        return result;
    }

    arr.sort();

    for i in 0..n - 3 {
        if i > 0 && arr[i] == arr[i - 1] { continue; }

        for j in (i + 1)..n - 2 {
            if j > i + 1 && arr[j] == arr[j - 1] { continue; }

            let mut left = j + 1;
            let mut right = n - 1;
            let remaining = target as i64 - arr[i] as i64 - arr[j] as i64;

            while left < right {
                let sum = arr[left] as i64 + arr[right] as i64;

                if sum == remaining {
                    result.push([arr[i], arr[j], arr[left], arr[right]]);

                    while left < right && arr[left] == arr[left + 1] { left += 1; }
                    while left < right && arr[right] == arr[right - 1] { right -= 1; }

                    left += 1;
                    right -= 1;
                } else if sum < remaining {
                    left += 1;
                } else {
                    right -= 1;
                }
            }
        }
    }
    result
}

/// Longest unique substring
pub fn longest_unique_substring(s: &str) -> usize {
    use std::collections::HashMap;

    let chars: Vec<char> = s.chars().collect();
    let mut char_index: HashMap<char, usize> = HashMap::new();
    let mut max_len = 0;
    let mut left = 0;

    for (right, &c) in chars.iter().enumerate() {
        if let Some(&prev) = char_index.get(&c) {
            if prev >= left {
                left = prev + 1;
            }
        }
        char_index.insert(c, right);
        max_len = max_len.max(right - left + 1);
    }
    max_len
}

/// K-way partition
pub fn k_way_partition(arr: &mut [i32], k: i32) {
    // Counting sort approach for small k
    if arr.is_empty() || k <= 0 { return; }

    let mut counts = vec![0usize; k as usize];

    for &x in arr.iter() {
        if x >= 0 && x < k {
            counts[x as usize] += 1;
        }
    }

    let mut idx = 0;
    for val in 0..k {
        for _ in 0..counts[val as usize] {
            arr[idx] = val;
            idx += 1;
        }
    }
}
```

---

### 4.9 spec.json

```json
{
  "name": "rush_hour_pointers",
  "language": "rust",
  "secondary_language": "c",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse (Two Pointers patterns)",
  "tags": ["two_pointers", "algorithms", "phase1", "optimization"],
  "passing_score": 70,

  "functions": [
    {
      "name": "pair_with_sum",
      "prototype": "pub fn pair_with_sum(arr: &[i32], target: i32) -> Option<(usize, usize)>",
      "return_type": "Option<(usize, usize)>"
    },
    {
      "name": "three_sum",
      "prototype": "pub fn three_sum(arr: &mut [i32]) -> Vec<[i32; 3]>",
      "return_type": "Vec<[i32; 3]>"
    },
    {
      "name": "max_area",
      "prototype": "pub fn max_area(heights: &[i32]) -> i64",
      "return_type": "i64"
    },
    {
      "name": "trap_water",
      "prototype": "pub fn trap_water(heights: &[i32]) -> i64",
      "return_type": "i64"
    },
    {
      "name": "dutch_flag",
      "prototype": "pub fn dutch_flag(arr: &mut [i32])",
      "return_type": "()"
    },
    {
      "name": "remove_duplicates",
      "prototype": "pub fn remove_duplicates(arr: &mut [i32]) -> usize",
      "return_type": "usize"
    },
    {
      "name": "move_zeros",
      "prototype": "pub fn move_zeros(arr: &mut [i32])",
      "return_type": "()"
    }
  ],

  "driver": {
    "edge_cases": [
      {
        "name": "pair_sum_empty",
        "function": "pair_with_sum",
        "args": [[], 5],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "Tableau vide"
      },
      {
        "name": "pair_sum_single",
        "function": "pair_with_sum",
        "args": [[5], 5],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "Un seul élément, pas de paire"
      },
      {
        "name": "three_sum_all_zeros",
        "function": "three_sum",
        "args": [[0, 0, 0, 0]],
        "expected": [[0, 0, 0]],
        "is_trap": true,
        "trap_explanation": "Doit retourner un seul triplet sans duplicats"
      },
      {
        "name": "dutch_flag_empty",
        "function": "dutch_flag",
        "args": [[]],
        "expected": [],
        "is_trap": true
      },
      {
        "name": "dutch_flag_single",
        "function": "dutch_flag",
        "args": [[2]],
        "expected": [2],
        "is_trap": true,
        "trap_explanation": "Un seul élément 2, attention à high=0"
      },
      {
        "name": "trap_water_flat",
        "function": "trap_water",
        "args": [[1, 1, 1, 1]],
        "expected": 0,
        "is_trap": true,
        "trap_explanation": "Surface plate = 0 eau"
      },
      {
        "name": "max_area_decreasing",
        "function": "max_area",
        "args": [[5, 4, 3, 2, 1]],
        "expected": 6,
        "is_trap": true,
        "trap_explanation": "Doit explorer toutes les combinaisons"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 5000,
      "generators": [
        {
          "type": "array_int",
          "param_index": 0,
          "params": {
            "min_len": 0,
            "max_len": 1000,
            "min_val": -1000,
            "max_val": 1000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": [],
    "forbidden_functions": ["sort"],
    "check_complexity": true,
    "expected_complexity": "O(n) for most, O(n²) for three_sum",
    "check_space": true,
    "expected_space": "O(1) auxiliary",
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes (minimum 5)

#### Mutant A (Boundary) : Off-by-one dans pair_with_sum

```rust
/* Mutant A (Boundary) : Condition while incorrecte */
pub fn pair_with_sum_mutant(arr: &[i32], target: i32) -> Option<(usize, usize)> {
    let mut left = 0;
    let mut right = arr.len() - 1;

    while left <= right {  // ❌ ERREUR : devrait être left < right
        let sum = arr[left] + arr[right];
        if sum == target {
            return Some((left, right));  // ❌ Peut retourner (i, i)
        }
        // ...
    }
    None
}
// Pourquoi c'est faux : Permet left == right, retournant le même élément deux fois
```

#### Mutant B (Safety) : Pas de check tableau vide

```rust
/* Mutant B (Safety) : Underflow */
pub fn dutch_flag_mutant(arr: &mut [i32]) {
    let mut high = arr.len() - 1;  // ❌ Panic si arr.is_empty()
    // ...
}
// Fix : if arr.is_empty() { return; }
```

#### Mutant C (Logic) : Mauvaise direction dans max_area

```rust
/* Mutant C (Logic) : Avance le mauvais pointeur */
pub fn max_area_mutant(heights: &[i32]) -> i64 {
    let mut left = 0;
    let mut right = heights.len() - 1;
    let mut max_area: i64 = 0;

    while left < right {
        // ...
        if heights[left] < heights[right] {
            right -= 1;  // ❌ ERREUR : devrait être left += 1
        } else {
            left += 1;   // ❌ ERREUR : devrait être right -= 1
        }
    }
    max_area
}
// Pourquoi c'est faux : On doit avancer le plus petit côté pour potentiellement trouver mieux
```

#### Mutant D (Duplicate) : Three sum avec duplicats

```rust
/* Mutant D (Duplicate) : Ne skip pas les duplicats */
pub fn three_sum_mutant(arr: &mut [i32]) -> Vec<[i32; 3]> {
    let mut result = Vec::new();
    arr.sort();

    for i in 0..arr.len() - 2 {
        // ❌ MANQUE : if i > 0 && arr[i] == arr[i-1] { continue; }

        let mut left = i + 1;
        let mut right = arr.len() - 1;

        while left < right {
            let sum = arr[i] + arr[left] + arr[right];
            if sum == 0 {
                result.push([arr[i], arr[left], arr[right]]);
                // ❌ MANQUE : skip duplicates for left and right
                left += 1;
                right -= 1;
            }
            // ...
        }
    }
    result
}
// Pourquoi c'est faux : Retourne des triplets dupliqués
```

#### Mutant E (Return) : Remove duplicates retourne count au lieu de longueur

```rust
/* Mutant E (Return) : Mauvaise valeur de retour */
pub fn remove_duplicates_mutant(arr: &mut [i32]) -> usize {
    if arr.is_empty() { return 0; }

    let mut slow = 0;

    for fast in 1..arr.len() {
        if arr[fast] != arr[slow] {
            slow += 1;
            arr[slow] = arr[fast];
        }
    }
    slow  // ❌ ERREUR : devrait être slow + 1
}
// Pourquoi c'est faux : slow est un index, la longueur est slow + 1
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Le pattern "Opposite Ends"** : Convergence de deux pointeurs vers le centre
2. **Le pattern "Same Direction"** : Slow/Fast pour les modifications in-place
3. **Le pattern "Dutch Flag"** : Partition en 3 groupes avec 3 pointeurs
4. **L'optimisation O(n²) → O(n)** : Comment les deux pointeurs réduisent la complexité
5. **La manipulation in-place** : Modifier sans espace auxiliaire

---

### 5.2 LDA — Traduction littérale

```
FONCTION pair_with_sum QUI RETOURNE UN OPTIONNEL DE TUPLE D'ENTIERS ET PREND EN PARAMÈTRES arr ET target
DÉBUT FONCTION
    SI arr EST VIDE OU LA LONGUEUR DE arr EST INFÉRIEURE À 2 ALORS
        RETOURNER AUCUN
    FIN SI

    DÉCLARER left COMME ENTIER NON SIGNÉ
    DÉCLARER right COMME ENTIER NON SIGNÉ

    AFFECTER 0 À left
    AFFECTER LA LONGUEUR DE arr MOINS 1 À right

    TANT QUE left EST INFÉRIEUR À right FAIRE
        DÉCLARER sum COMME ENTIER
        AFFECTER L'ÉLÉMENT À LA POSITION left PLUS L'ÉLÉMENT À LA POSITION right À sum

        SI sum EST ÉGAL À target ALORS
            RETOURNER QUELQUE(left, right)
        SINON SI sum EST INFÉRIEUR À target ALORS
            INCRÉMENTER left DE 1
        SINON
            DÉCRÉMENTER right DE 1
        FIN SI
    FIN TANT QUE

    RETOURNER AUCUN
FIN FONCTION
```

---

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : Two Pointers - Opposite Ends
---
1. INITIALISER left = 0, right = len - 1

2. BOUCLE tant que left < right :
   a. CALCULER valeur = process(arr[left], arr[right])

   b. SELON le résultat :
      - CAS TROUVÉ : RETOURNER (left, right)
      - CAS TROP PETIT : left++ (avancer vers la droite)
      - CAS TROP GRAND : right-- (reculer vers la gauche)

3. RETOURNER "pas trouvé"

INVARIANT : À chaque itération, la solution (si elle existe)
            est toujours dans la fenêtre [left, right]
```

---

### 5.2.3.1 Logique de Garde (Fail Fast)

```
FONCTION : dutch_flag (arr)
---
INIT low = 0, mid = 0, high = len - 1

1. GARDE : tableau vide
   |-- VÉRIFIER arr.is_empty()
   |     SI OUI → RETOURNER immédiatement
   |
   |-- Raison : Évite underflow sur len - 1

2. GARDE : high devient négatif
   |-- VÉRIFIER high == 0 avant de décrémenter
   |     SI OUI → SORTIR de la boucle
   |
   |-- Raison : Évite panic en mode release

3. TRAITEMENT PRINCIPAL :
   |-- BOUCLE tant que mid <= high
   |     SI arr[mid] == 0 : swap(low, mid), low++, mid++
   |     SI arr[mid] == 1 : mid++
   |     SI arr[mid] == 2 : swap(mid, high), high--

4. RETOURNER (tableau modifié in-place)
```

---

### 5.3 Visualisation ASCII

#### Pattern 1 : Opposite Ends

```
Recherche de paire avec somme = 9 dans [2, 7, 11, 15]

Étape 1:
┌───┬───┬───┬───┐
│ 2 │ 7 │ 11│ 15│
└───┴───┴───┴───┘
  ↑           ↑
 left       right

sum = 2 + 15 = 17 > 9 → right--

Étape 2:
┌───┬───┬───┬───┐
│ 2 │ 7 │ 11│ 15│
└───┴───┴───┴───┘
  ↑       ↑
 left   right

sum = 2 + 11 = 13 > 9 → right--

Étape 3:
┌───┬───┬───┬───┐
│ 2 │ 7 │ 11│ 15│
└───┴───┴───┴───┘
  ↑   ↑
 left right

sum = 2 + 7 = 9 == 9 → TROUVÉ! (0, 1)
```

#### Pattern 2 : Same Direction (Slow/Fast)

```
Move Zeros dans [0, 1, 0, 3, 12]

Étape 1:
┌───┬───┬───┬───┬───┐
│ 0 │ 1 │ 0 │ 3 │ 12│
└───┴───┴───┴───┴───┘
  ↑
 S,F
arr[fast] == 0 → fast++

Étape 2:
┌───┬───┬───┬───┬───┐
│ 0 │ 1 │ 0 │ 3 │ 12│
└───┴───┴───┴───┴───┘
  ↑   ↑
  S   F
arr[fast] == 1 → swap, slow++, fast++

Après swap:
┌───┬───┬───┬───┬───┐
│ 1 │ 0 │ 0 │ 3 │ 12│
└───┴───┴───┴───┴───┘
      ↑   ↑
      S   F

... continue jusqu'à:

Résultat final:
┌───┬───┬───┬───┬───┐
│ 1 │ 3 │ 12│ 0 │ 0 │
└───┴───┴───┴───┴───┘
              ↑       ↑
              S       F
```

#### Pattern 3 : Dutch National Flag

```
Partition de [2, 0, 2, 1, 1, 0]

LÉGENDE: low=L, mid=M, high=H

Initial:
┌───┬───┬───┬───┬───┬───┐
│ 2 │ 0 │ 2 │ 1 │ 1 │ 0 │
└───┴───┴───┴───┴───┴───┘
  ↑                   ↑
 L,M                  H

arr[M]=2 → swap(M,H), H--:
┌───┬───┬───┬───┬───┬───┐
│ 0 │ 0 │ 2 │ 1 │ 1 │ 2 │
└───┴───┴───┴───┴───┴───┘
  ↑               ↑
 L,M              H

arr[M]=0 → swap(L,M), L++, M++:
┌───┬───┬───┬───┬───┬───┐
│ 0 │ 0 │ 2 │ 1 │ 1 │ 2 │
└───┴───┴───┴───┴───┴───┘
      ↑           ↑
     L,M          H

... continue ...

Résultat:
┌───┬───┬───┬───┬───┬───┐
│ 0 │ 0 │ 1 │ 1 │ 2 │ 2 │
└───┴───┴───┴───┴───┴───┘
          ↑   ↑
          L   H
         (M hors bornes)
```

#### Trapping Rain Water

```
heights = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]

Visualisation:
                        █
            █           █ █   █
    █       █ █   █     █ █ █ █ █
────────────────────────────────────
    0 1 0 2 1 0 1 3 2 1 2 1

Eau piégée (en ~):
                        █
            █ ~ ~ ~ ~ ~ █ █ ~ █
    █ ~ ~ ~ █ █ ~ █ ~ ~ █ █ █ █ █
────────────────────────────────────

Total eau = 6 unités
```

---

### 5.4 Les pièges en détail

| Piège | Description | Exemple | Solution |
|-------|-------------|---------|----------|
| **Same element twice** | `left == right` retourne même élément | pair(1,1) dans [2] | Condition `left < right` (pas `<=`) |
| **Empty array** | Underflow sur `len - 1` | `[].len() - 1` | Check `is_empty()` d'abord |
| **Duplicates in three_sum** | Triplets identiques | `[-1,-1,0,1,1]` | Skip duplicates après chaque trouvaille |
| **High underflow** | Dutch flag avec high = 0 | `[2]` | Break avant high -= 1 si high == 0 |
| **Wrong pointer move** | Avancer left au lieu de right | max_area | Logique claire sur quel pointeur bouger |

---

### 5.5 Cours Complet : Two Pointers

#### 5.5.1 Quand utiliser Two Pointers ?

La technique s'applique quand :
1. Les données sont **triées** (ou peuvent l'être)
2. On cherche une **paire/triplet** satisfaisant une condition
3. On veut faire des **modifications in-place**
4. On peut réduire **O(n²) en O(n)**

#### 5.5.2 Les 3 Patterns Fondamentaux

**Pattern 1: Opposite Ends**
- Deux pointeurs aux extrémités
- Convergent vers le centre
- Utilisé pour : pair sum, palindrome, container with water

**Pattern 2: Same Direction (Slow/Fast)**
- Deux pointeurs qui avancent ensemble
- Slow = position de destination
- Fast = exploration
- Utilisé pour : remove duplicates, move zeros

**Pattern 3: Dutch Flag (3-way partition)**
- Trois pointeurs : low, mid, high
- Partition en trois groupes
- Utilisé pour : sort colors, partition problems

#### 5.5.3 Analyse de Complexité

| Pattern | Temps | Espace | Passes sur données |
|---------|-------|--------|-------------------|
| Opposite Ends | O(n) | O(1) | 1 |
| Slow/Fast | O(n) | O(1) | 1 |
| Dutch Flag | O(n) | O(1) | 1 |
| Three Sum | O(n²) | O(1) aux | n passes |
| Four Sum | O(n³) | O(1) aux | n² passes |

---

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (O(n) espace)                                     │
├─────────────────────────────────────────────────────────────────┤
│ let mut result = arr.to_vec();  // Copie le tableau            │
│ // Travaille sur result                                         │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME (O(1) espace)                                       │
├─────────────────────────────────────────────────────────────────┤
│ arr.swap(i, j);  // Modification in-place                       │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Two pointers = modification in-place                          │
│ • Copier annule l'avantage de la technique                      │
│ • La plupart des problèmes demandent O(1) espace auxiliaire     │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.7 Simulation avec trace d'exécution

#### Exemple : `dutch_flag([2, 0, 2, 1, 1, 0])`

```
┌───────┬───────────────────────────────────┬─────┬─────┬──────┬──────────────────────────────────┐
│ Étape │ Action                            │ low │ mid │ high │ Tableau                          │
├───────┼───────────────────────────────────┼─────┼─────┼──────┼──────────────────────────────────┤
│   0   │ Initialisation                    │  0  │  0  │   5  │ [2, 0, 2, 1, 1, 0]               │
├───────┼───────────────────────────────────┼─────┼─────┼──────┼──────────────────────────────────┤
│   1   │ arr[0]=2 → swap(0,5), high--      │  0  │  0  │   4  │ [0, 0, 2, 1, 1, 2]               │
├───────┼───────────────────────────────────┼─────┼─────┼──────┼──────────────────────────────────┤
│   2   │ arr[0]=0 → swap(0,0), low++, mid++│  1  │  1  │   4  │ [0, 0, 2, 1, 1, 2]               │
├───────┼───────────────────────────────────┼─────┼─────┼──────┼──────────────────────────────────┤
│   3   │ arr[1]=0 → swap(1,1), low++, mid++│  2  │  2  │   4  │ [0, 0, 2, 1, 1, 2]               │
├───────┼───────────────────────────────────┼─────┼─────┼──────┼──────────────────────────────────┤
│   4   │ arr[2]=2 → swap(2,4), high--      │  2  │  2  │   3  │ [0, 0, 1, 1, 2, 2]               │
├───────┼───────────────────────────────────┼─────┼─────┼──────┼──────────────────────────────────┤
│   5   │ arr[2]=1 → mid++                  │  2  │  3  │   3  │ [0, 0, 1, 1, 2, 2]               │
├───────┼───────────────────────────────────┼─────┼─────┼──────┼──────────────────────────────────┤
│   6   │ arr[3]=1 → mid++                  │  2  │  4  │   3  │ [0, 0, 1, 1, 2, 2]               │
├───────┼───────────────────────────────────┼─────┼─────┼──────┼──────────────────────────────────┤
│   7   │ mid > high → SORTIE               │  -  │  -  │   -  │ [0, 0, 1, 1, 2, 2] ✓             │
└───────┴───────────────────────────────────┴─────┴─────┴──────┴──────────────────────────────────┘
```

---

### 5.8 Mnémotechniques

#### 🎬 MEME : "Rush Hour" — Lee et Carter

**"You don't know Kung Fu?!"** — Carter à Lee

- **Lee (Jackie Chan)** = pointeur left → technique précise, méthodique
- **Carter (Chris Tucker)** = pointeur right → imprévisible, énergique
- **Ensemble** ils convergent vers le criminel (la solution)

```
     LEE (left)                    CARTER (right)
         ↓                              ↓
    [2, 7, 11, 15]
         └─────── convergent ──────────┘

"We don't just solve cases... we CONVERGE on them!"
```

#### 🇳🇱 MEME : Dutch Flag — Le Drapeau de Dijkstra

Le drapeau néerlandais a 3 couleurs :
- 🔴 Rouge (0) → en bas
- ⚪ Blanc (1) → au milieu
- 🔵 Bleu (2) → en haut

Dijkstra, le créateur de l'algorithme, était néerlandais. C'est pourquoi ça s'appelle "Dutch National Flag Problem" !

```
Avant:          Après:
[2,0,2,1,1,0]   [0,0,1,1,2,2]
                🔴🔴⚪⚪🔵🔵
```

#### 💧 MEME : "Water Bending" — Trapping Rain Water

Comme dans Avatar, l'eau suit les lois de la physique :
- Elle remplit les creux
- Elle s'arrête aux murs
- Le niveau max = min(left_max, right_max)

**"The water always finds a way... downward."**

---

### 5.9 Applications pratiques

| Application | Pattern | Utilisation |
|-------------|---------|-------------|
| **Git Merge** | Same Direction | Fusion de deux branches triées |
| **Video Sync** | Opposite Ends | Synchronisation audio/vidéo |
| **Database Index** | Opposite Ends | Merge join dans les SGBD |
| **Image Processing** | Slow/Fast | Compression run-length |
| **Network Packets** | Dutch Flag | Priorisation QoS (high/medium/low) |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Gravité | Pattern | Comment l'éviter |
|---|-------|---------|---------|------------------|
| 1 | `left <= right` | 🔴 | Opposite Ends | Utiliser `<` strict |
| 2 | Tableau vide | 🔴 | Tous | Check `is_empty()` |
| 3 | Duplicats three_sum | 🟡 | Three Sum | Skip while equal |
| 4 | `high -= 1` underflow | 🔴 | Dutch Flag | Check `high == 0` |
| 5 | Wrong pointer move | 🟡 | Max Area | Move le plus petit |
| 6 | Return slow vs slow+1 | 🟢 | Same Direction | slow = index, len = slow+1 |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quelle est la complexité de pair_with_sum sur un tableau TRIÉ ?**

- A) O(1)
- B) O(log n)
- C) O(n)
- D) O(n log n)
- E) O(n²)

**Réponse : C**

---

### Question 2
**Pourquoi Dutch Flag utilise-t-il 3 pointeurs au lieu de 2 ?**

- A) Pour aller plus vite
- B) Pour partitionner en 3 groupes en une seule passe
- C) C'est une erreur de conception
- D) Pour éviter les duplicats
- E) Pour la compatibilité avec C

**Réponse : B**

---

### Question 3
**Dans three_sum, pourquoi trie-t-on d'abord le tableau ?**

- A) Pour pouvoir utiliser two pointers
- B) Pour éviter les duplicats facilement
- C) Pour réduire la complexité à O(n²)
- D) Toutes les réponses ci-dessus
- E) Aucune des réponses ci-dessus

**Réponse : D**

---

### Question 4
**Quel pattern utilise-t-on pour "Container With Most Water" ?**

- A) Same Direction
- B) Opposite Ends
- C) Dutch Flag
- D) Sliding Window
- E) Binary Search

**Réponse : B**

---

### Question 5
**Que retourne `remove_duplicates` pour `[1,1,2,2,3]` ?**

- A) 2
- B) 3
- C) 5
- D) Le tableau modifié
- E) Un nouveau tableau

**Réponse : B** (la nouvelle longueur, tableau devient [1,2,3,_,_])

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Détail |
|---------|--------|
| **Nom** | rush_hour_pointers |
| **Concept** | Two Pointers Technique |
| **Difficulté** | ★★★★★☆☆☆☆☆ (5/10) |
| **Fonctions** | 13 (base) + 4 (bonus) |
| **Patterns** | 3 (Opposite Ends, Same Direction, Dutch Flag) |
| **Complexité** | O(n) à O(n²) selon la fonction |
| **MEME** | Rush Hour (Lee & Carter) / Dutch Flag |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.1.3-rush_hour_pointers",
    "generated_at": "2026-01-11 00:00:00",

    "metadata": {
      "exercise_id": "1.1.3",
      "exercise_name": "rush_hour_pointers",
      "module": "1.1",
      "module_name": "Arrays & Sorting",
      "concept": "i",
      "concept_name": "Two Pointers Technique",
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
      "complexity_time": "T2-3 O(n) to O(n²)",
      "complexity_space": "S1 O(1)",
      "prerequisites": ["arrays", "loops", "conditions"],
      "domains": ["Algo", "Struct", "MD"],
      "domains_bonus": ["DP"],
      "tags": ["two_pointers", "optimization", "in_place"],
      "meme_reference": "Rush Hour / Dutch Flag"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 Rust */",
      "references/ref_solution.c": "/* Section 4.3 C */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 */",
      "mutants/mutant_c_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_d_duplicate.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "L'excellence pédagogique ne se négocie pas"*
*Exercice généré automatiquement — Compatible ENGINE v22.1 + Mutation Tester*
