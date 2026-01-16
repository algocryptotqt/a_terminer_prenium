# Exercice 1.1.4 : the_truman_window

**Module :**
1.1 — Arrays & Sorting

**Concept :**
j — Sliding Window Technique

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (patterns sliding window + applications)

**Langages :**
Rust Edition 2024 + C (c17)

**Prérequis :**
- Manipulation de tableaux et chaînes
- HashMap / HashSet
- Deque (double-ended queue)
- Two Pointers (ex03)

**Domaines :**
Algo, Struct, MD

**Durée estimée :**
60 min

**XP Base :**
150

**Complexité :**
T2 O(n) × S2 O(k) où k = taille fenêtre

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `sliding_window.c`, `sliding_window.h`

**Fonctions autorisées :**
- Collections standard (HashMap, VecDeque, HashSet)
- Allocation mémoire

**Fonctions interdites :**
- Algorithmes de tri O(n log n) dans la fenêtre (utiliser monotonic deque)

---

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🎬 THE TRUMAN SHOW — La Réalité à Travers la Fenêtre**

*"Good morning, and in case I don't see ya, good afternoon, good evening, and good night!"*

Truman Burbank vit dans un monde artificiel où tout est filmé. Des **caméras glissantes** suivent chaque moment de sa vie, révélant une "fenêtre" sur sa réalité.

En algorithmique, le **Sliding Window** c'est exactement ça :
- Une **fenêtre** de taille fixe ou variable
- Qui **glisse** sur les données
- Révélant des **informations** à chaque position

**Trois types de caméras :**

1. **Caméra Fixe** (Fixed Window) : Toujours la même taille de cadrage
   - "Je filme les 3 dernières minutes de Truman"
   - Max sum, averages, etc.

2. **Caméra Zoom** (Variable Window) : S'adapte à l'action
   - "Je filme jusqu'à ce que quelque chose d'intéressant se passe"
   - Longest substring, min window, etc.

3. **Caméra Time-lapse** (Monotonic Deque) : Garde les meilleurs moments
   - "Je garde seulement les pics d'action"
   - Sliding maximum/minimum

**Ta mission :**

Implémenter une bibliothèque complète de techniques **Sliding Window** avec **14 algorithmes** différents.

#### 1.2.2 Version Académique

La technique de la fenêtre glissante (Sliding Window) optimise les problèmes de sous-tableaux contigus. Au lieu de recalculer pour chaque position (O(n*k)), on maintient l'état de la fenêtre et on le met à jour incrémentalement (O(n)).

**Trois patterns principaux :**
1. **Fixed Size** : Fenêtre de taille k constante
2. **Variable Size** : Fenêtre qui s'étend/se contracte selon une condition
3. **Monotonic Deque** : Maintient un invariant (max/min) dans la fenêtre

---

**Entrée :**
- `arr` / `s` : tableau d'entiers ou chaîne de caractères
- `k` : taille de fenêtre (pour fixed window)
- `target` / `pattern` : valeur cible ou pattern à chercher

**Sortie :**
- Valeurs calculées, indices, ou chaînes selon la fonction

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  0 ≤ arr.len() ≤ 10⁵                                            │
│  Complexité temps : O(n) pour toutes les fonctions              │
│  Complexité espace : O(k) ou O(alphabet) selon la fonction      │
│  Pour les chaînes : UTF-8 / ASCII supporté                      │
└─────────────────────────────────────────────────────────────────┘
```

---

### 1.3 Prototype

#### Rust (Edition 2024)

```rust
pub mod sliding_window {
    use std::collections::{HashMap, VecDeque, HashSet};

    /// Maximum sum of any contiguous subarray of size k
    pub fn max_sum_subarray(arr: &[i32], k: usize) -> Option<i64>;

    /// Average of all contiguous subarrays of size k
    pub fn subarray_averages(arr: &[i32], k: usize) -> Vec<f64>;

    /// Maximum of each sliding window of size k (monotonic deque)
    pub fn sliding_window_max(arr: &[i32], k: usize) -> Vec<i32>;

    /// Minimum of each sliding window of size k
    pub fn sliding_window_min(arr: &[i32], k: usize) -> Vec<i32>;

    /// Count subarrays with sum equal to target (prefix sum + hash map)
    pub fn count_subarrays_with_sum(arr: &[i32], target: i32) -> i64;

    /// Count subarrays with sum ≤ max_sum (positive numbers only)
    pub fn count_subarrays_at_most_sum(arr: &[i32], max_sum: i32) -> i64;

    /// Longest substring without repeating characters
    pub fn longest_unique_substring(s: &str) -> usize;

    /// Longest substring with at most k distinct characters
    pub fn longest_with_k_distinct(s: &str, k: usize) -> usize;

    /// Find all anagram occurrences of pattern in text
    pub fn find_anagrams(text: &str, pattern: &str) -> Vec<usize>;

    /// Minimum window substring containing all characters of pattern
    pub fn min_window_substring(s: &str, pattern: &str) -> String;

    /// Maximum consecutive 1s if you can flip at most k 0s
    pub fn max_ones_with_k_flips(arr: &[i32], k: usize) -> usize;

    /// Longest repeating character replacement with at most k changes
    pub fn character_replacement(s: &str, k: usize) -> usize;

    /// Fruit into baskets (longest subarray with at most 2 types)
    pub fn total_fruit(fruits: &[i32]) -> usize;

    /// Is s1's permutation a substring of s2?
    pub fn check_inclusion(s1: &str, s2: &str) -> bool;
}
```

#### C (c17)

```c
#ifndef SLIDING_WINDOW_H
# define SLIDING_WINDOW_H

# include <stddef.h>
# include <stdbool.h>
# include <stdint.h>

typedef struct {
    int64_t value;
    bool valid;
} optional_i64_t;

typedef struct {
    size_t *indices;
    size_t count;
    size_t capacity;
} index_list_t;

typedef struct {
    char *str;
    size_t len;
} string_result_t;

// Fixed window operations
optional_i64_t sw_max_sum_subarray(const int *arr, size_t len, size_t k);
double *sw_subarray_averages(const int *arr, size_t len, size_t k, size_t *out_len);

// Monotonic deque operations
int *sw_sliding_window_max(const int *arr, size_t len, size_t k, size_t *out_len);
int *sw_sliding_window_min(const int *arr, size_t len, size_t k, size_t *out_len);

// Sum operations
int64_t sw_count_subarrays_with_sum(const int *arr, size_t len, int target);
int64_t sw_count_subarrays_at_most_sum(const int *arr, size_t len, int max_sum);

// String operations
size_t sw_longest_unique_substring(const char *s);
size_t sw_longest_with_k_distinct(const char *s, size_t k);
index_list_t sw_find_anagrams(const char *text, const char *pattern);
string_result_t sw_min_window_substring(const char *s, const char *pattern);

// Binary array operations
size_t sw_max_ones_with_k_flips(const int *arr, size_t len, size_t k);
size_t sw_character_replacement(const char *s, size_t k);
size_t sw_total_fruit(const int *fruits, size_t len);
bool sw_check_inclusion(const char *s1, const char *s2);

// Cleanup functions
void sw_free_averages(double *arr);
void sw_free_window_result(int *arr);
void sw_free_index_list(index_list_t *list);
void sw_free_string_result(string_result_t *result);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Fun Facts

**🎯 L'optimisation spectaculaire**

Pour trouver le maximum dans chaque fenêtre de taille k :

| Approche | Complexité | Pour n=10⁶, k=1000 |
|----------|-----------|-------------------|
| Naïve | O(n * k) | 10⁹ opérations |
| Monotonic Deque | O(n) | 10⁶ opérations |

**1000x plus rapide !**

**📊 Utilisations réelles**

- **Trading algorithmique** : Moving averages (SMA, EMA)
- **Streaming** : Rate limiting, traffic analysis
- **Games** : Score rolling windows
- **IoT** : Sensor data smoothing

**🔬 L'histoire du Monotonic Deque**

Cette technique a été popularisée par les compétitions de programmation (ACM-ICPC, Codeforces) dans les années 2000. Elle est maintenant enseignée dans les meilleurs cours d'algorithmes (MIT, Stanford).

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation |
|--------|-------------|
| **Data Scientist** | Rolling statistics, feature engineering |
| **Quant Developer** | Moving averages, Bollinger bands |
| **Backend Developer** | Rate limiting, request throttling |
| **DevOps** | Log analysis, anomaly detection |
| **Game Dev** | FPS smoothing, score tracking |
| **Network Engineer** | Packet analysis, bandwidth monitoring |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
sliding_window.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 14 tests
test test_max_sum_subarray ... ok
test test_subarray_averages ... ok
test test_sliding_window_max ... ok
test test_sliding_window_min ... ok
test test_count_subarrays_sum ... ok
test test_count_subarrays_at_most ... ok
test test_longest_unique ... ok
test test_k_distinct ... ok
test test_find_anagrams ... ok
test test_min_window ... ok
test test_max_ones ... ok
test test_char_replacement ... ok
test test_total_fruit ... ok
test test_check_inclusion ... ok

test result: ok. 14 passed; 0 failed
```

---

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n) strict avec analyse amortie

**Space Complexity attendue :**
O(k) strict

**Domaines Bonus :**
`MD, DP`

#### 3.1.1 Consigne Bonus

**🎬 THE TRUMAN SHOW: DIRECTOR'S CUT**

Le réalisateur veut des techniques encore plus avancées pour la nouvelle saison :

1. **Median of Sliding Window** : Médiane de chaque fenêtre en O(n log k)
2. **Maximum Sum Circular Subarray** : Fenêtre qui peut "wrap around"
3. **Shortest Subarray with Sum ≥ K** : Avec nombres négatifs possibles
4. **Subarrays with Bounded Maximum** : Count subarrays where max ∈ [L, R]

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  Median : Utiliser deux heaps ou balanced BST                   │
│  Circular : Kadane modifié + wrap-around logic                  │
│  Shortest with negative : Monotonic deque sur prefix sums       │
│  Bounded max : Inclusion-exclusion avec sliding window          │
└─────────────────────────────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
/// Median of each sliding window of size k
pub fn sliding_window_median(arr: &[i32], k: usize) -> Vec<f64>;

/// Maximum sum of circular subarray
pub fn max_sum_circular(arr: &[i32]) -> i64;

/// Shortest subarray with sum >= k (negatives allowed)
pub fn shortest_subarray_sum_at_least(arr: &[i32], k: i32) -> Option<usize>;

/// Count subarrays where maximum element is in [L, R]
pub fn count_subarrays_bounded_max(arr: &[i32], left: i32, right: i32) -> i64;
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Médiane | Non | O(n log k) avec heaps |
| Nombres négatifs | Limité | Full support |
| Circular arrays | Non | Wrap-around |
| Bounded queries | Non | [L, R] range |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `max_sum_basic` | `[2,1,5,1,3,2], k=3` | `Some(9)` | 2 | Non |
| `max_sum_k_too_big` | `[1,2], k=5` | `None` | 2 | Edge |
| `max_sum_empty` | `[], k=1` | `None` | 2 | Edge |
| `averages_basic` | `[1,2,3,4,5], k=3` | `[2.0,3.0,4.0]` | 3 | Non |
| `sliding_max_basic` | `[1,3,-1,-3,5,3,6,7], k=3` | `[3,3,5,5,6,7]` | 5 | Non |
| `sliding_max_all_same` | `[1,1,1,1], k=2` | `[1,1,1]` | 3 | Edge |
| `sliding_min_basic` | `[1,3,-1,-3,5,3,6,7], k=3` | `[-1,-3,-3,-3,3,3]` | 5 | Non |
| `count_sum_exact` | `[1,1,1], target=2` | `2` | 4 | Non |
| `count_sum_zero` | `[1,-1,0], target=0` | `4` | 3 | Trap |
| `count_at_most` | `[1,2,3], max=4` | `6` | 3 | Non |
| `longest_unique_basic` | `"abcabcbb"` | `3` | 3 | Non |
| `longest_unique_all_same` | `"bbbb"` | `1` | 2 | Edge |
| `longest_unique_empty` | `""` | `0` | 2 | Edge |
| `k_distinct_basic` | `"eceba", k=2` | `3` | 3 | Non |
| `anagrams_basic` | `"cbaebabacd", "abc"` | `[0,6]` | 4 | Non |
| `anagrams_overlap` | `"abab", "ab"` | `[0,1,2]` | 3 | Trap |
| `min_window_basic` | `"ADOBECODEBANC", "ABC"` | `"BANC"` | 5 | Non |
| `min_window_none` | `"a", "aa"` | `""` | 3 | Edge |
| `max_ones_basic` | `[1,1,0,0,0,1,1,1,1,0], k=2` | `6` | 4 | Non |
| `max_ones_all_zeros` | `[0,0,0], k=2` | `2` | 2 | Edge |
| `char_replace_basic` | `"ABAB", k=2` | `4` | 4 | Non |
| `char_replace_no_change` | `"AAAA", k=0` | `4` | 2 | Edge |
| `total_fruit_basic` | `[1,2,1], 2 types` | `3` | 3 | Non |
| `total_fruit_many` | `[1,2,3,2,2]` | `4` | 3 | Non |
| `inclusion_yes` | `"ab", "eidbaooo"` | `true` | 3 | Non |
| `inclusion_no` | `"ab", "eidboaoo"` | `false` | 2 | Non |

**Total : 100 points**

---

### 4.2 main.c de test

```c
#include <stdio.h>
#include <assert.h>
#include <string.h>
#include <math.h>
#include "sliding_window.h"

void test_max_sum(void) {
    int arr[] = {2, 1, 5, 1, 3, 2};
    optional_i64_t result = sw_max_sum_subarray(arr, 6, 3);
    assert(result.valid && result.value == 9);

    result = sw_max_sum_subarray(arr, 6, 10);
    assert(!result.valid);
    printf("test_max_sum: OK\n");
}

void test_sliding_max(void) {
    int arr[] = {1, 3, -1, -3, 5, 3, 6, 7};
    size_t out_len;
    int *result = sw_sliding_window_max(arr, 8, 3, &out_len);

    assert(out_len == 6);
    int expected[] = {3, 3, 5, 5, 6, 7};
    for (size_t i = 0; i < out_len; i++) {
        assert(result[i] == expected[i]);
    }
    sw_free_window_result(result);
    printf("test_sliding_max: OK\n");
}

void test_count_subarrays(void) {
    int arr[] = {1, 1, 1};
    assert(sw_count_subarrays_with_sum(arr, 3, 2) == 2);
    printf("test_count_subarrays: OK\n");
}

void test_longest_unique(void) {
    assert(sw_longest_unique_substring("abcabcbb") == 3);
    assert(sw_longest_unique_substring("bbbbb") == 1);
    assert(sw_longest_unique_substring("pwwkew") == 3);
    printf("test_longest_unique: OK\n");
}

void test_find_anagrams(void) {
    index_list_t result = sw_find_anagrams("cbaebabacd", "abc");
    assert(result.count == 2);
    assert(result.indices[0] == 0);
    assert(result.indices[1] == 6);
    sw_free_index_list(&result);
    printf("test_find_anagrams: OK\n");
}

void test_min_window(void) {
    string_result_t result = sw_min_window_substring("ADOBECODEBANC", "ABC");
    assert(strcmp(result.str, "BANC") == 0);
    sw_free_string_result(&result);
    printf("test_min_window: OK\n");
}

void test_max_ones(void) {
    int arr[] = {1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 0};
    assert(sw_max_ones_with_k_flips(arr, 11, 2) == 6);
    printf("test_max_ones: OK\n");
}

void test_check_inclusion(void) {
    assert(sw_check_inclusion("ab", "eidbaooo") == true);
    assert(sw_check_inclusion("ab", "eidboaoo") == false);
    printf("test_check_inclusion: OK\n");
}

int main(void) {
    test_max_sum();
    test_sliding_max();
    test_count_subarrays();
    test_longest_unique();
    test_find_anagrams();
    test_min_window();
    test_max_ones();
    test_check_inclusion();

    printf("\nTous les tests passent!\n");
    return 0;
}
```

---

### 4.3 Solution de référence

#### Rust

```rust
pub mod sliding_window {
    use std::collections::{HashMap, VecDeque, HashSet};

    /// Maximum sum of any contiguous subarray of size k
    pub fn max_sum_subarray(arr: &[i32], k: usize) -> Option<i64> {
        if k == 0 || k > arr.len() {
            return None;
        }

        let mut window_sum: i64 = arr[..k].iter().map(|&x| x as i64).sum();
        let mut max_sum = window_sum;

        for i in k..arr.len() {
            window_sum += arr[i] as i64 - arr[i - k] as i64;
            max_sum = max_sum.max(window_sum);
        }
        Some(max_sum)
    }

    /// Average of all contiguous subarrays of size k
    pub fn subarray_averages(arr: &[i32], k: usize) -> Vec<f64> {
        if k == 0 || k > arr.len() {
            return Vec::new();
        }

        let mut result = Vec::with_capacity(arr.len() - k + 1);
        let mut window_sum: i64 = arr[..k].iter().map(|&x| x as i64).sum();
        result.push(window_sum as f64 / k as f64);

        for i in k..arr.len() {
            window_sum += arr[i] as i64 - arr[i - k] as i64;
            result.push(window_sum as f64 / k as f64);
        }
        result
    }

    /// Maximum of each sliding window of size k
    pub fn sliding_window_max(arr: &[i32], k: usize) -> Vec<i32> {
        if k == 0 || k > arr.len() {
            return Vec::new();
        }

        let mut result = Vec::with_capacity(arr.len() - k + 1);
        let mut deque: VecDeque<usize> = VecDeque::new();

        for i in 0..arr.len() {
            // Remove elements outside window
            while !deque.is_empty() && *deque.front().unwrap() + k <= i {
                deque.pop_front();
            }

            // Maintain monotonic decreasing deque
            while !deque.is_empty() && arr[*deque.back().unwrap()] <= arr[i] {
                deque.pop_back();
            }

            deque.push_back(i);

            if i >= k - 1 {
                result.push(arr[*deque.front().unwrap()]);
            }
        }
        result
    }

    /// Minimum of each sliding window of size k
    pub fn sliding_window_min(arr: &[i32], k: usize) -> Vec<i32> {
        if k == 0 || k > arr.len() {
            return Vec::new();
        }

        let mut result = Vec::with_capacity(arr.len() - k + 1);
        let mut deque: VecDeque<usize> = VecDeque::new();

        for i in 0..arr.len() {
            while !deque.is_empty() && *deque.front().unwrap() + k <= i {
                deque.pop_front();
            }

            while !deque.is_empty() && arr[*deque.back().unwrap()] >= arr[i] {
                deque.pop_back();
            }

            deque.push_back(i);

            if i >= k - 1 {
                result.push(arr[*deque.front().unwrap()]);
            }
        }
        result
    }

    /// Count subarrays with sum equal to target
    pub fn count_subarrays_with_sum(arr: &[i32], target: i32) -> i64 {
        let mut count: i64 = 0;
        let mut prefix_sum: i64 = 0;
        let mut prefix_counts: HashMap<i64, i64> = HashMap::new();
        prefix_counts.insert(0, 1);

        for &num in arr {
            prefix_sum += num as i64;
            let needed = prefix_sum - target as i64;

            if let Some(&c) = prefix_counts.get(&needed) {
                count += c;
            }

            *prefix_counts.entry(prefix_sum).or_insert(0) += 1;
        }
        count
    }

    /// Count subarrays with sum ≤ max_sum (positive numbers only)
    pub fn count_subarrays_at_most_sum(arr: &[i32], max_sum: i32) -> i64 {
        if arr.is_empty() {
            return 0;
        }

        let mut count: i64 = 0;
        let mut left = 0;
        let mut window_sum: i64 = 0;

        for right in 0..arr.len() {
            window_sum += arr[right] as i64;

            while window_sum > max_sum as i64 && left <= right {
                window_sum -= arr[left] as i64;
                left += 1;
            }

            count += (right - left + 1) as i64;
        }
        count
    }

    /// Longest substring without repeating characters
    pub fn longest_unique_substring(s: &str) -> usize {
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

    /// Longest substring with at most k distinct characters
    pub fn longest_with_k_distinct(s: &str, k: usize) -> usize {
        if k == 0 {
            return 0;
        }

        let chars: Vec<char> = s.chars().collect();
        let mut char_count: HashMap<char, usize> = HashMap::new();
        let mut max_len = 0;
        let mut left = 0;

        for right in 0..chars.len() {
            *char_count.entry(chars[right]).or_insert(0) += 1;

            while char_count.len() > k {
                let left_char = chars[left];
                *char_count.get_mut(&left_char).unwrap() -= 1;
                if char_count[&left_char] == 0 {
                    char_count.remove(&left_char);
                }
                left += 1;
            }

            max_len = max_len.max(right - left + 1);
        }
        max_len
    }

    /// Find all anagram occurrences of pattern in text
    pub fn find_anagrams(text: &str, pattern: &str) -> Vec<usize> {
        let text_chars: Vec<char> = text.chars().collect();
        let pattern_chars: Vec<char> = pattern.chars().collect();
        let mut result = Vec::new();

        if pattern_chars.len() > text_chars.len() {
            return result;
        }

        let mut pattern_count: HashMap<char, i32> = HashMap::new();
        let mut window_count: HashMap<char, i32> = HashMap::new();

        for &c in &pattern_chars {
            *pattern_count.entry(c).or_insert(0) += 1;
        }

        let k = pattern_chars.len();

        for i in 0..text_chars.len() {
            *window_count.entry(text_chars[i]).or_insert(0) += 1;

            if i >= k {
                let left_char = text_chars[i - k];
                *window_count.get_mut(&left_char).unwrap() -= 1;
                if window_count[&left_char] == 0 {
                    window_count.remove(&left_char);
                }
            }

            if i >= k - 1 && window_count == pattern_count {
                result.push(i - k + 1);
            }
        }
        result
    }

    /// Minimum window substring containing all characters of pattern
    pub fn min_window_substring(s: &str, pattern: &str) -> String {
        if pattern.is_empty() || s.len() < pattern.len() {
            return String::new();
        }

        let s_chars: Vec<char> = s.chars().collect();
        let mut pattern_count: HashMap<char, i32> = HashMap::new();
        let mut window_count: HashMap<char, i32> = HashMap::new();

        for c in pattern.chars() {
            *pattern_count.entry(c).or_insert(0) += 1;
        }

        let required = pattern_count.len();
        let mut formed = 0;
        let mut left = 0;
        let mut min_len = usize::MAX;
        let mut result = (0, 0);

        for right in 0..s_chars.len() {
            let c = s_chars[right];
            *window_count.entry(c).or_insert(0) += 1;

            if pattern_count.contains_key(&c)
                && window_count[&c] == pattern_count[&c]
            {
                formed += 1;
            }

            while formed == required && left <= right {
                if right - left + 1 < min_len {
                    min_len = right - left + 1;
                    result = (left, right + 1);
                }

                let left_char = s_chars[left];
                *window_count.get_mut(&left_char).unwrap() -= 1;

                if pattern_count.contains_key(&left_char)
                    && window_count[&left_char] < pattern_count[&left_char]
                {
                    formed -= 1;
                }
                left += 1;
            }
        }

        if min_len == usize::MAX {
            String::new()
        } else {
            s_chars[result.0..result.1].iter().collect()
        }
    }

    /// Maximum consecutive 1s if you can flip at most k 0s
    pub fn max_ones_with_k_flips(arr: &[i32], k: usize) -> usize {
        let mut max_len = 0;
        let mut zeros_count = 0;
        let mut left = 0;

        for right in 0..arr.len() {
            if arr[right] == 0 {
                zeros_count += 1;
            }

            while zeros_count > k {
                if arr[left] == 0 {
                    zeros_count -= 1;
                }
                left += 1;
            }

            max_len = max_len.max(right - left + 1);
        }
        max_len
    }

    /// Longest repeating character replacement with at most k changes
    pub fn character_replacement(s: &str, k: usize) -> usize {
        let chars: Vec<char> = s.chars().collect();
        let mut char_count: HashMap<char, usize> = HashMap::new();
        let mut max_count = 0;
        let mut max_len = 0;
        let mut left = 0;

        for right in 0..chars.len() {
            *char_count.entry(chars[right]).or_insert(0) += 1;
            max_count = max_count.max(char_count[&chars[right]]);

            while (right - left + 1) - max_count > k {
                *char_count.get_mut(&chars[left]).unwrap() -= 1;
                left += 1;
            }

            max_len = max_len.max(right - left + 1);
        }
        max_len
    }

    /// Fruit into baskets (longest subarray with at most 2 types)
    pub fn total_fruit(fruits: &[i32]) -> usize {
        longest_with_k_distinct_arr(fruits, 2)
    }

    fn longest_with_k_distinct_arr(arr: &[i32], k: usize) -> usize {
        if k == 0 || arr.is_empty() {
            return 0;
        }

        let mut fruit_count: HashMap<i32, usize> = HashMap::new();
        let mut max_len = 0;
        let mut left = 0;

        for right in 0..arr.len() {
            *fruit_count.entry(arr[right]).or_insert(0) += 1;

            while fruit_count.len() > k {
                let left_fruit = arr[left];
                *fruit_count.get_mut(&left_fruit).unwrap() -= 1;
                if fruit_count[&left_fruit] == 0 {
                    fruit_count.remove(&left_fruit);
                }
                left += 1;
            }

            max_len = max_len.max(right - left + 1);
        }
        max_len
    }

    /// Is s1's permutation a substring of s2?
    pub fn check_inclusion(s1: &str, s2: &str) -> bool {
        !find_anagrams(s2, s1).is_empty()
    }
}
```

---

### 4.5 Solutions refusées (avec explications)

#### Refusée 1 : O(n*k) pour sliding max

```rust
// ❌ REFUSÉ - O(n*k) au lieu de O(n)
pub fn sliding_window_max_slow(arr: &[i32], k: usize) -> Vec<i32> {
    let mut result = Vec::new();
    for i in 0..=(arr.len() - k) {
        let window = &arr[i..i+k];
        result.push(*window.iter().max().unwrap());  // ❌ O(k) à chaque fenêtre
    }
    result
}
// Pourquoi c'est refusé : Complexité O(n*k), pas O(n)
```

#### Refusée 2 : Mauvaise gestion du deque

```rust
// ❌ REFUSÉ - Ne maintient pas l'invariant monotone
pub fn sliding_window_max_wrong(arr: &[i32], k: usize) -> Vec<i32> {
    let mut deque: VecDeque<usize> = VecDeque::new();
    let mut result = Vec::new();

    for i in 0..arr.len() {
        // ❌ MANQUE : Retirer les éléments plus petits à l'arrière
        deque.push_back(i);

        while !deque.is_empty() && *deque.front().unwrap() + k <= i {
            deque.pop_front();
        }

        if i >= k - 1 {
            result.push(arr[*deque.front().unwrap()]);
        }
    }
    result
}
// Pourquoi c'est faux : Le front du deque n'est pas forcément le max
```

---

### 4.9 spec.json

```json
{
  "name": "the_truman_window",
  "language": "rust",
  "secondary_language": "c",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse (Sliding Window patterns)",
  "tags": ["sliding_window", "algorithms", "phase1", "optimization"],
  "passing_score": 70,

  "functions": [
    {
      "name": "max_sum_subarray",
      "prototype": "pub fn max_sum_subarray(arr: &[i32], k: usize) -> Option<i64>",
      "return_type": "Option<i64>"
    },
    {
      "name": "sliding_window_max",
      "prototype": "pub fn sliding_window_max(arr: &[i32], k: usize) -> Vec<i32>",
      "return_type": "Vec<i32>"
    },
    {
      "name": "count_subarrays_with_sum",
      "prototype": "pub fn count_subarrays_with_sum(arr: &[i32], target: i32) -> i64",
      "return_type": "i64"
    },
    {
      "name": "longest_unique_substring",
      "prototype": "pub fn longest_unique_substring(s: &str) -> usize",
      "return_type": "usize"
    },
    {
      "name": "find_anagrams",
      "prototype": "pub fn find_anagrams(text: &str, pattern: &str) -> Vec<usize>",
      "return_type": "Vec<usize>"
    },
    {
      "name": "min_window_substring",
      "prototype": "pub fn min_window_substring(s: &str, pattern: &str) -> String",
      "return_type": "String"
    }
  ],

  "driver": {
    "edge_cases": [
      {
        "name": "max_sum_k_zero",
        "function": "max_sum_subarray",
        "args": [[1,2,3], 0],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "k=0 est invalide"
      },
      {
        "name": "max_sum_k_too_big",
        "function": "max_sum_subarray",
        "args": [[1,2], 5],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "k > len est invalide"
      },
      {
        "name": "sliding_max_empty",
        "function": "sliding_window_max",
        "args": [[], 3],
        "expected": [],
        "is_trap": true
      },
      {
        "name": "longest_unique_empty",
        "function": "longest_unique_substring",
        "args": [""],
        "expected": 0,
        "is_trap": true
      },
      {
        "name": "min_window_impossible",
        "function": "min_window_substring",
        "args": ["a", "aa"],
        "expected": "",
        "is_trap": true,
        "trap_explanation": "Pattern plus long que s"
      },
      {
        "name": "anagrams_pattern_longer",
        "function": "find_anagrams",
        "args": ["ab", "abc"],
        "expected": [],
        "is_trap": true
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
    "allowed_functions": ["HashMap", "VecDeque", "HashSet"],
    "forbidden_functions": [],
    "check_complexity": true,
    "expected_complexity": "O(n)",
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes (minimum 5)

#### Mutant A (Complexity) : O(n*k) au lieu de O(n)

```rust
/* Mutant A (Complexity) : Recalcule le max à chaque fenêtre */
pub fn sliding_window_max_mutant(arr: &[i32], k: usize) -> Vec<i32> {
    let mut result = Vec::new();
    for i in 0..=(arr.len() - k) {
        // ❌ O(k) pour chaque fenêtre → O(n*k) total
        result.push(*arr[i..i+k].iter().max().unwrap());
    }
    result
}
// Pourquoi c'est faux : Ne respecte pas la contrainte O(n)
```

#### Mutant B (Boundary) : k > len non géré

```rust
/* Mutant B (Boundary) : Pas de vérification k > len */
pub fn max_sum_subarray_mutant(arr: &[i32], k: usize) -> Option<i64> {
    // ❌ MANQUE : if k > arr.len() { return None; }
    let mut sum: i64 = arr[..k].iter().map(|&x| x as i64).sum();  // ❌ Panic
    // ...
}
// Pourquoi c'est faux : Panic si k > arr.len()
```

#### Mutant C (Logic) : Mauvais index dans deque

```rust
/* Mutant C (Logic) : Utilise la valeur au lieu de l'index */
pub fn sliding_window_max_mutant(arr: &[i32], k: usize) -> Vec<i32> {
    let mut deque: VecDeque<i32> = VecDeque::new();  // ❌ Stocke valeurs, pas indices
    // ...
    // Ne peut pas vérifier si l'élément est hors fenêtre
}
// Pourquoi c'est faux : Sans les indices, impossible de savoir si élément est hors fenêtre
```

#### Mutant D (Off-by-one) : Fenêtre décalée

```rust
/* Mutant D (Off-by-one) : Commence à enregistrer trop tôt */
pub fn subarray_averages_mutant(arr: &[i32], k: usize) -> Vec<f64> {
    let mut result = Vec::new();
    let mut sum: i64 = 0;

    for i in 0..arr.len() {
        sum += arr[i] as i64;
        if i >= k {
            sum -= arr[i - k] as i64;
        }
        if i >= k - 2 {  // ❌ ERREUR : devrait être k - 1
            result.push(sum as f64 / k as f64);
        }
    }
    result
}
// Pourquoi c'est faux : Enregistre avant que la fenêtre soit pleine
```

#### Mutant E (Return) : Min window retourne première occurrence

```rust
/* Mutant E (Return) : Ne cherche pas la plus petite */
pub fn min_window_substring_mutant(s: &str, pattern: &str) -> String {
    // ... trouve une fenêtre valide ...
    // ❌ Retourne immédiatement sans chercher une plus petite
    return found_window;  // ❌ Pas forcément la minimale
}
// Pourquoi c'est faux : Doit trouver la PLUS PETITE fenêtre
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Fixed Window** : Maintenir une somme/moyenne glissante en O(1) par élément
2. **Variable Window** : Étendre/contracter selon une condition
3. **Monotonic Deque** : Maintenir min/max en O(1) amorti
4. **HashMap Window** : Compter les occurrences dans la fenêtre
5. **Prefix Sum** : Transformer count-sum en lookup

---

### 5.2 LDA — Traduction littérale

```
FONCTION sliding_window_max QUI RETOURNE UN VECTEUR D'ENTIERS ET PREND EN PARAMÈTRES arr ET k
DÉBUT FONCTION
    SI k EST ÉGAL À 0 OU k EST SUPÉRIEUR À LA LONGUEUR DE arr ALORS
        RETOURNER UN VECTEUR VIDE
    FIN SI

    DÉCLARER result COMME VECTEUR D'ENTIERS
    DÉCLARER deque COMME FILE DOUBLE D'INDICES

    POUR i ALLANT DE 0 À LA LONGUEUR DE arr MOINS 1 FAIRE
        // Retirer les éléments hors fenêtre
        TANT QUE deque N'EST PAS VIDE ET L'ÉLÉMENT EN TÊTE DE deque PLUS k EST INFÉRIEUR OU ÉGAL À i FAIRE
            RETIRER L'ÉLÉMENT EN TÊTE DE deque
        FIN TANT QUE

        // Maintenir le deque décroissant monotone
        TANT QUE deque N'EST PAS VIDE ET arr[ÉLÉMENT EN QUEUE DE deque] EST INFÉRIEUR OU ÉGAL À arr[i] FAIRE
            RETIRER L'ÉLÉMENT EN QUEUE DE deque
        FIN TANT QUE

        AJOUTER i EN QUEUE DE deque

        SI i EST SUPÉRIEUR OU ÉGAL À k MOINS 1 ALORS
            AJOUTER arr[ÉLÉMENT EN TÊTE DE deque] À result
        FIN SI
    FIN POUR

    RETOURNER result
FIN FONCTION
```

---

### 5.3 Visualisation ASCII

#### Pattern 1 : Fixed Window

```
Max Sum Subarray, k=3 dans [2, 1, 5, 1, 3, 2]

Fenêtre 1: [2, 1, 5] = 8
           ├────────┤
           └─ sum=8

Fenêtre 2: [1, 5, 1] = 7
              ├────────┤
              └─ sum=7

Fenêtre 3: [5, 1, 3] = 9 ← MAX!
                 ├────────┤
                 └─ sum=9

Fenêtre 4: [1, 3, 2] = 6
                    ├────────┤
                    └─ sum=6

Résultat: max = 9
```

#### Pattern 2 : Monotonic Deque

```
Sliding Max, k=3 dans [1, 3, -1, -3, 5, 3, 6, 7]

État du deque (stocke INDICES, valeurs montrées pour clarté):

i=0: deque=[1]
i=1: deque=[3]      (1 ≤ 3, pop 1)
i=2: deque=[3, -1]  (première fenêtre complète) → max=3

i=3: deque=[3, -1, -3] → max=3
i=4: deque=[5]      (3 hors fenêtre, 5 > tous) → max=5
i=5: deque=[5, 3]   → max=5
i=6: deque=[6]      (5 hors fenêtre) → max=6
i=7: deque=[7]      → max=7

Résultat: [3, 3, 5, 5, 6, 7]

Invariant du deque:
┌───────────────────────────────────────┐
│  Toujours DÉCROISSANT de gauche à     │
│  droite (le max est toujours en tête) │
└───────────────────────────────────────┘
```

#### Pattern 3 : Variable Window

```
Longest Unique Substring dans "abcabcbb"

                 a b c a b c b b
Index:           0 1 2 3 4 5 6 7

left=0, right=0: [a]             len=1
left=0, right=1: [a,b]           len=2
left=0, right=2: [a,b,c]         len=3 ← max jusqu'ici
left=0, right=3: [a,b,c,a]       'a' existe! left=1
left=1, right=3: [b,c,a]         len=3
left=1, right=4: [b,c,a,b]       'b' existe! left=2
...

Résultat: 3 ("abc")
```

---

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **k > len** | Fenêtre plus grande que tableau | Vérifier et retourner None/empty |
| **k = 0** | Fenêtre vide | Cas spécial, retourner None/empty |
| **Deque values vs indices** | Stocker valeurs au lieu d'indices | Toujours stocker les indices |
| **Off-by-one start** | Enregistrer avant fenêtre pleine | Check `i >= k - 1` |
| **HashMap not cleaned** | Compteur à 0 non retiré | `if count == 0 { remove() }` |

---

### 5.5 Cours Complet : Sliding Window

#### 5.5.1 Les 3 Patterns

**1. Fixed Size Window**
```
Cas d'utilisation: max/min/sum/avg sur fenêtres de taille constante
Complexité: O(n) temps, O(1) espace
```

**2. Variable Size Window**
```
Cas d'utilisation: trouver la plus grande/petite fenêtre satisfaisant une condition
Complexité: O(n) temps, O(1) ou O(k) espace
```

**3. Monotonic Deque**
```
Cas d'utilisation: maintenir max/min dans une fenêtre glissante
Complexité: O(n) amorti (chaque élément entre et sort une seule fois)
```

#### 5.5.2 Quand utiliser chaque pattern ?

| Problème | Pattern |
|----------|---------|
| Somme/moyenne fixe | Fixed |
| Max/min dans fenêtre | Monotonic Deque |
| Longest substring avec contrainte | Variable |
| Count subarrays avec sum | Prefix Sum + HashMap |
| Find anagram | Fixed + HashMap |

---

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (O(n*k))                                          │
├─────────────────────────────────────────────────────────────────┤
│ for window in arr.windows(k) {                                  │
│     result.push(*window.iter().max().unwrap());                 │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME (O(n))                                              │
├─────────────────────────────────────────────────────────────────┤
│ // Utiliser monotonic deque                                     │
│ while !deque.is_empty() && arr[*deque.back().unwrap()] <= arr[i]│
│     deque.pop_back();                                           │
│ deque.push_back(i);                                             │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Le deque maintient l'invariant monotone                       │
│ • Chaque élément entre et sort au plus une fois                 │
│ • Complexité amortie O(1) par opération                         │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.7 Simulation avec trace d'exécution

#### Exemple : `sliding_window_max([1,3,-1,-3,5,3,6,7], k=3)`

```
┌───────┬────────────────────────────────┬──────────────────┬─────────────┐
│ i     │ Opérations                     │ Deque (indices)  │ Result      │
├───────┼────────────────────────────────┼──────────────────┼─────────────┤
│   0   │ push(0)                        │ [0]              │ -           │
├───────┼────────────────────────────────┼──────────────────┼─────────────┤
│   1   │ arr[0]=1 ≤ arr[1]=3, pop_back  │ []               │ -           │
│       │ push(1)                        │ [1]              │             │
├───────┼────────────────────────────────┼──────────────────┼─────────────┤
│   2   │ arr[1]=3 > arr[2]=-1, keep     │ [1]              │ -           │
│       │ push(2)                        │ [1, 2]           │             │
│       │ i >= k-1, record arr[1]=3      │ [1, 2]           │ [3]         │
├───────┼────────────────────────────────┼──────────────────┼─────────────┤
│   3   │ 1+3 > 3, no pop_front          │ [1, 2]           │ -           │
│       │ arr[2]=-1 > arr[3]=-3, keep    │ [1, 2]           │             │
│       │ push(3)                        │ [1, 2, 3]        │             │
│       │ record arr[1]=3                │ [1, 2, 3]        │ [3, 3]      │
├───────┼────────────────────────────────┼──────────────────┼─────────────┤
│   4   │ 1+3 ≤ 4, pop_front             │ [2, 3]           │ -           │
│       │ arr[2]=-1 ≤ arr[4]=5, pop_back │ [3]              │             │
│       │ arr[3]=-3 ≤ arr[4]=5, pop_back │ []               │             │
│       │ push(4)                        │ [4]              │             │
│       │ record arr[4]=5                │ [4]              │ [3,3,5]     │
├───────┼────────────────────────────────┼──────────────────┼─────────────┤
│   5   │ arr[4]=5 > arr[5]=3, keep      │ [4]              │ -           │
│       │ push(5)                        │ [4, 5]           │             │
│       │ record arr[4]=5                │ [4, 5]           │ [3,3,5,5]   │
├───────┼────────────────────────────────┼──────────────────┼─────────────┤
│   6   │ 4+3 ≤ 6, pop_front             │ [5]              │ -           │
│       │ arr[5]=3 ≤ arr[6]=6, pop_back  │ []               │             │
│       │ push(6)                        │ [6]              │             │
│       │ record arr[6]=6                │ [6]              │ [3,3,5,5,6] │
├───────┼────────────────────────────────┼──────────────────┼─────────────┤
│   7   │ arr[6]=6 ≤ arr[7]=7, pop_back  │ []               │ -           │
│       │ push(7)                        │ [7]              │             │
│       │ record arr[7]=7                │ [7]              │ [3,3,5,5,6,7]│
└───────┴────────────────────────────────┴──────────────────┴─────────────┘
```

---

### 5.8 Mnémotechniques

#### 🎬 MEME : "The Truman Show" — Caméra Glissante

*"Good morning! And in case I don't see ya, good afternoon, good evening, and good night!"*

La vie de Truman est filmée 24/7 par des caméras qui **glissent** pour capturer chaque moment. C'est exactement ce que fait Sliding Window :

```
        CAMÉRA (fenêtre)
            │
            ▼
    ┌───────────────┐
    │  [a] [b] [c]  │ [d] [e] [f] [g]
    └───────────────┘
         La vie de Truman (le tableau)

    GLISSE →

        ┌───────────────┐
    [a] │  [b] [c] [d]  │ [e] [f] [g]
        └───────────────┘
```

**"Truman ne sait pas qu'il est filmé, mais la caméra capture tout... en O(n)."**

#### 📺 MEME : Netflix "Skip Intro"

Le bouton "Skip Intro" analyse une fenêtre glissante de l'audio pour détecter le générique. Si les 30 dernières secondes matchent le pattern "musique d'intro", il affiche le bouton.

**C'est exactement `find_anagrams` / `check_inclusion` !**

```
Audio: [bla] [bla] [INTRO] [INTRO] [INTRO] [episode]
                    ↑───────────────↑
                    "Skip Intro" détecté!
```

---

### 5.9 Applications pratiques

| Application | Pattern | Exemple |
|-------------|---------|---------|
| **Trading** | Fixed | Moving Average (SMA) |
| **Streaming** | Variable | Rate limiting |
| **Security** | HashMap | Intrusion detection patterns |
| **Games** | Fixed | Rolling FPS counter |
| **Search** | HashMap | Fuzzy matching |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Gravité | Comment l'éviter |
|---|-------|---------|------------------|
| 1 | k > arr.len() | 🔴 | Vérifier au début |
| 2 | k = 0 | 🔴 | Retourner empty |
| 3 | Valeurs au lieu d'indices | 🔴 | Toujours indices dans deque |
| 4 | Off-by-one start | 🟡 | `i >= k - 1` |
| 5 | HashMap leak | 🟡 | Remove quand count = 0 |
| 6 | O(n*k) | 🔴 | Utiliser monotonic deque |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quelle structure utilise-t-on pour sliding_window_max en O(n) ?**

- A) Stack
- B) Queue simple
- C) Monotonic Deque
- D) Heap
- E) HashMap

**Réponse : C**

---

### Question 2
**Pourquoi le deque doit-il être monotone DÉCROISSANT pour le max ?**

- A) Pour économiser de la mémoire
- B) Pour que le max soit toujours en tête
- C) Pour éviter les duplicats
- D) C'est une convention
- E) Pour la compatibilité avec C

**Réponse : B**

---

### Question 3
**Quelle est la complexité amortie de sliding_window_max ?**

- A) O(1)
- B) O(log n)
- C) O(n)
- D) O(n log n)
- E) O(n²)

**Réponse : C** (chaque élément entre et sort au plus une fois)

---

### Question 4
**Pour count_subarrays_with_sum, quelle technique utilise-t-on ?**

- A) Monotonic Deque
- B) Two Pointers
- C) Prefix Sum + HashMap
- D) Binary Search
- E) Divide and Conquer

**Réponse : C**

---

### Question 5
**Que retourne longest_unique_substring("bbbb") ?**

- A) 0
- B) 1
- C) 4
- D) "b"
- E) None

**Réponse : B** (le plus long substring unique est "b", longueur 1)

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Détail |
|---------|--------|
| **Nom** | the_truman_window |
| **Concept** | Sliding Window Technique |
| **Difficulté** | ★★★★★☆☆☆☆☆ (5/10) |
| **Fonctions** | 14 (base) + 4 (bonus) |
| **Patterns** | 3 (Fixed, Variable, Monotonic Deque) |
| **Complexité** | O(n) pour toutes |
| **MEME** | The Truman Show / Netflix Skip Intro |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.1.4-the_truman_window",
    "generated_at": "2026-01-11 00:00:00",

    "metadata": {
      "exercise_id": "1.1.4",
      "exercise_name": "the_truman_window",
      "module": "1.1",
      "module_name": "Arrays & Sorting",
      "concept": "j",
      "concept_name": "Sliding Window Technique",
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
      "complexity_time": "T2 O(n)",
      "complexity_space": "S2 O(k)",
      "prerequisites": ["arrays", "hashmap", "two_pointers"],
      "domains": ["Algo", "Struct", "MD"],
      "tags": ["sliding_window", "monotonic_deque", "optimization"],
      "meme_reference": "The Truman Show"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "L'excellence pédagogique ne se négocie pas"*
*Exercice généré automatiquement — Compatible ENGINE v22.1 + Mutation Tester*
