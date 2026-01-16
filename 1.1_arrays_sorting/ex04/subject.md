# Exercise 04: Sliding Window

## Concepts Covered
- **1.1.j** Sliding window technique
- **1.1.12.k** Maximum of all subarrays of size k
- **1.1.12.l** Count subarrays with sum
- **1.1.12.m** Anagram search
- **1.1.12.n** Complexity analysis

## Objective

Master the sliding window technique for subarray problems. Transform O(n*k) brute force into O(n) solutions.

## Requirements

### Rust Implementation

```rust
pub mod sliding_window {
    use std::collections::{HashMap, VecDeque};

    /// Maximum sum of any contiguous subarray of size k
    pub fn max_sum_subarray(arr: &[i32], k: usize) -> Option<i64>;

    /// Average of all contiguous subarrays of size k
    pub fn subarray_averages(arr: &[i32], k: usize) -> Vec<f64>;

    /// Maximum of each sliding window of size k
    /// Uses monotonic deque for O(n) solution
    pub fn sliding_window_max(arr: &[i32], k: usize) -> Vec<i32>;

    /// Minimum of each sliding window of size k
    pub fn sliding_window_min(arr: &[i32], k: usize) -> Vec<i32>;

    /// Count subarrays with sum equal to target
    /// Uses prefix sum + hash map
    pub fn count_subarrays_with_sum(arr: &[i32], target: i32) -> i64;

    /// Count subarrays with sum less than or equal to target
    /// Assumes all elements are positive
    pub fn count_subarrays_at_most_sum(arr: &[i32], max_sum: i32) -> i64;

    /// Longest substring without repeating characters
    pub fn longest_unique_substring(s: &str) -> usize;

    /// Longest substring with at most k distinct characters
    pub fn longest_with_k_distinct(s: &str, k: usize) -> usize;

    /// Find all anagram occurrences of pattern in text
    /// Returns starting indices of all anagrams
    pub fn find_anagrams(text: &str, pattern: &str) -> Vec<usize>;

    /// Minimum window substring containing all characters of pattern
    pub fn min_window_substring(s: &str, pattern: &str) -> String;

    /// Maximum number of consecutive 1s if you can flip at most k 0s
    pub fn max_ones_with_k_flips(arr: &[i32], k: usize) -> usize;

    /// Longest repeating character replacement with at most k changes
    pub fn character_replacement(s: &str, k: usize) -> usize;

    /// Fruit into baskets (longest subarray with at most 2 types)
    pub fn total_fruit(fruits: &[i32]) -> usize;

    /// Permutation in string (is s1's permutation a substring of s2?)
    pub fn check_inclusion(s1: &str, s2: &str) -> bool;
}
```

### Python Implementation

```python
from collections import deque

def max_sum_subarray(arr: list[int], k: int) -> int | None: ...
def subarray_averages(arr: list[int], k: int) -> list[float]: ...
def sliding_window_max(arr: list[int], k: int) -> list[int]: ...
def sliding_window_min(arr: list[int], k: int) -> list[int]: ...
def count_subarrays_with_sum(arr: list[int], target: int) -> int: ...
def count_subarrays_at_most_sum(arr: list[int], max_sum: int) -> int: ...
def longest_unique_substring(s: str) -> int: ...
def longest_with_k_distinct(s: str, k: int) -> int: ...
def find_anagrams(text: str, pattern: str) -> list[int]: ...
def min_window_substring(s: str, pattern: str) -> str: ...
def max_ones_with_k_flips(arr: list[int], k: int) -> int: ...
def character_replacement(s: str, k: int) -> int: ...
def total_fruit(fruits: list[int]) -> int: ...
def check_inclusion(s1: str, s2: str) -> bool: ...
```

## Sliding Window Patterns

### Pattern 1: Fixed Size Window
```rust
fn fixed_window(arr: &[i32], k: usize) -> Vec<i32> {
    let mut result = Vec::new();
    let mut window_sum = 0;

    for i in 0..arr.len() {
        window_sum += arr[i];  // Add right element

        if i >= k {
            window_sum -= arr[i - k];  // Remove left element
        }

        if i >= k - 1 {
            result.push(window_sum);
        }
    }
    result
}
```

### Pattern 2: Variable Size Window (Shrinkable)
```rust
fn variable_window(arr: &[i32], target: i32) -> usize {
    let mut left = 0;
    let mut window_sum = 0;
    let mut max_len = 0;

    for right in 0..arr.len() {
        window_sum += arr[right];  // Expand

        while window_sum > target && left <= right {
            window_sum -= arr[left];  // Shrink
            left += 1;
        }

        max_len = max_len.max(right - left + 1);
    }
    max_len
}
```

### Pattern 3: Monotonic Deque
```rust
fn sliding_max(arr: &[i32], k: usize) -> Vec<i32> {
    let mut deque: VecDeque<usize> = VecDeque::new();
    let mut result = Vec::new();

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
```

## Test Cases

```rust
#[test]
fn test_max_sum_subarray() {
    let arr = vec![2, 1, 5, 1, 3, 2];
    assert_eq!(max_sum_subarray(&arr, 3), Some(9)); // [5, 1, 3]
    assert_eq!(max_sum_subarray(&arr, 1), Some(5));
    assert_eq!(max_sum_subarray(&arr, 7), None);
}

#[test]
fn test_sliding_window_max() {
    let arr = vec![1, 3, -1, -3, 5, 3, 6, 7];
    let result = sliding_window_max(&arr, 3);
    assert_eq!(result, vec![3, 3, 5, 5, 6, 7]);
}

#[test]
fn test_count_subarrays() {
    let arr = vec![1, 1, 1];
    assert_eq!(count_subarrays_with_sum(&arr, 2), 2);

    let arr = vec![1, 2, 3];
    assert_eq!(count_subarrays_with_sum(&arr, 3), 2); // [1,2] and [3]
}

#[test]
fn test_longest_unique() {
    assert_eq!(longest_unique_substring("abcabcbb"), 3); // "abc"
    assert_eq!(longest_unique_substring("bbbbb"), 1);    // "b"
    assert_eq!(longest_unique_substring("pwwkew"), 3);   // "wke"
}

#[test]
fn test_find_anagrams() {
    assert_eq!(find_anagrams("cbaebabacd", "abc"), vec![0, 6]);
    assert_eq!(find_anagrams("abab", "ab"), vec![0, 1, 2]);
}

#[test]
fn test_min_window() {
    assert_eq!(min_window_substring("ADOBECODEBANC", "ABC"), "BANC");
    assert_eq!(min_window_substring("a", "a"), "a");
    assert_eq!(min_window_substring("a", "aa"), "");
}

#[test]
fn test_max_ones() {
    let arr = vec![1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 0];
    assert_eq!(max_ones_with_k_flips(&arr, 2), 6); // [1,1,1,0,0,1,1,1,1]

    let arr = vec![0, 0, 1, 1, 0, 0, 1, 1, 1, 0, 1, 1, 0, 0, 0, 1, 1, 1, 1];
    assert_eq!(max_ones_with_k_flips(&arr, 3), 10);
}

#[test]
fn test_character_replacement() {
    assert_eq!(character_replacement("ABAB", 2), 4);
    assert_eq!(character_replacement("AABABBA", 1), 4);
}

#[test]
fn test_check_inclusion() {
    assert!(check_inclusion("ab", "eidbaooo"));
    assert!(!check_inclusion("ab", "eidboaoo"));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Fixed window (max sum, averages) | 10 |
| Monotonic deque (sliding max/min) | 15 |
| Count subarrays with sum | 15 |
| Longest unique substring | 10 |
| K distinct characters | 10 |
| Find anagrams | 10 |
| Minimum window substring | 15 |
| Max ones with k flips | 10 |
| Check inclusion | 5 |
| **Total** | **100** |

## Complexity Requirements

All solutions must be O(n) time complexity:
- Fixed window: O(n)
- Variable window: O(n)
- Monotonic deque: O(n) amortized
- Hash map sliding window: O(n)

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `sliding_window.py`
