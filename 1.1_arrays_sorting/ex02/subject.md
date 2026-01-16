# Exercise 02: Binary Search Variants

## Concepts Covered
- **1.1.g** Binary search variants
- **1.1.22.g-k** Termination, off-by-one, templates
- **1.1.23.h-n** Ceil, peak element, rotated array, sqrt
- **1.1.24.f-i** Binary search on answer

## Objective

Master binary search through multiple variants and applications. This exercise emphasizes avoiding off-by-one errors and choosing the right template.

## Requirements

### Rust Implementation

```rust
pub mod binary_search {
    /// Standard binary search - returns Some(index) if found
    pub fn search<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

    /// Lower bound - first element >= target
    pub fn lower_bound<T: Ord>(arr: &[T], target: &T) -> usize;

    /// Upper bound - first element > target
    pub fn upper_bound<T: Ord>(arr: &[T], target: &T) -> usize;

    /// Ceil - smallest element >= target (or None)
    pub fn ceil<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

    /// Floor - largest element <= target (or None)
    pub fn floor<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

    /// Find peak element in bitonic array
    pub fn find_peak(arr: &[i32]) -> usize;

    /// Search in rotated sorted array (no duplicates)
    pub fn search_rotated<T: Ord>(arr: &[T], target: &T) -> Option<usize>;

    /// Search in rotated sorted array with duplicates
    pub fn search_rotated_with_dups<T: Ord>(arr: &[T], target: &T) -> bool;

    /// Find rotation point (index of minimum element)
    pub fn find_rotation_point<T: Ord>(arr: &[T]) -> usize;

    /// Integer square root: floor(sqrt(n))
    pub fn isqrt(n: u64) -> u64;

    /// Kth smallest in row-wise & column-wise sorted matrix
    pub fn kth_smallest_matrix(matrix: &[Vec<i32>], k: usize) -> i32;

    /// Binary search on answer: can we achieve X?
    /// Returns minimum X such that predicate(X) is true
    pub fn binary_search_answer<F>(lo: i64, hi: i64, predicate: F) -> i64
    where
        F: Fn(i64) -> bool;

    /// Floating-point binary search with epsilon precision
    pub fn binary_search_float<F>(lo: f64, hi: f64, eps: f64, predicate: F) -> f64
    where
        F: Fn(f64) -> bool;
}
```

### Python Implementation

```python
from typing import TypeVar, Callable, Sequence
from collections.abc import Sequence as Seq

T = TypeVar("T")

def search(arr: Sequence[T], target: T) -> int | None: ...
def lower_bound(arr: Sequence[T], target: T) -> int: ...
def upper_bound(arr: Sequence[T], target: T) -> int: ...
def ceil(arr: Sequence[T], target: T) -> int | None: ...
def floor(arr: Sequence[T], target: T) -> int | None: ...
def find_peak(arr: Sequence[int]) -> int: ...
def search_rotated(arr: Sequence[T], target: T) -> int | None: ...
def search_rotated_with_dups(arr: Sequence[T], target: T) -> bool: ...
def find_rotation_point(arr: Sequence[T]) -> int: ...
def isqrt(n: int) -> int: ...
def kth_smallest_matrix(matrix: Sequence[Sequence[int]], k: int) -> int: ...
def binary_search_answer(lo: int, hi: int, predicate: Callable[[int], bool]) -> int: ...
def binary_search_float(lo: float, hi: float, eps: float, predicate: Callable[[float], bool]) -> float: ...
```

## Binary Search Templates

### Template 1: Standard (lo <= hi)
```
Use when: Searching for exact match
Returns: Index of target or -1

while lo <= hi:
    mid = lo + (hi - lo) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        lo = mid + 1
    else:
        hi = mid - 1
return -1
```

### Template 2: Leftmost (lo < hi)
```
Use when: Finding first occurrence, lower_bound
Returns: Index of first element >= target

while lo < hi:
    mid = lo + (hi - lo) // 2
    if arr[mid] < target:
        lo = mid + 1
    else:
        hi = mid
return lo
```

### Template 3: Rightmost with post-processing
```
Use when: Finding last occurrence, floor
Returns: Index of last element <= target

while lo < hi:
    mid = lo + (hi - lo + 1) // 2  # Note: +1 to avoid infinite loop
    if arr[mid] <= target:
        lo = mid
    else:
        hi = mid - 1
return lo
```

## Test Cases

```rust
#[test]
fn test_standard_search() {
    let arr = vec![1, 3, 5, 7, 9, 11, 13];
    assert_eq!(search(&arr, &7), Some(3));
    assert_eq!(search(&arr, &1), Some(0));
    assert_eq!(search(&arr, &13), Some(6));
    assert_eq!(search(&arr, &6), None);
    assert_eq!(search(&arr, &0), None);
    assert_eq!(search(&arr, &14), None);
}

#[test]
fn test_lower_upper_bound() {
    let arr = vec![1, 2, 2, 2, 3, 4, 5];
    assert_eq!(lower_bound(&arr, &2), 1);
    assert_eq!(upper_bound(&arr, &2), 4);
    assert_eq!(lower_bound(&arr, &0), 0);
    assert_eq!(upper_bound(&arr, &5), 7);
}

#[test]
fn test_ceil_floor() {
    let arr = vec![1, 3, 5, 7, 9];
    assert_eq!(ceil(&arr, &4), Some(2));   // 5 at index 2
    assert_eq!(ceil(&arr, &5), Some(2));   // 5 at index 2
    assert_eq!(floor(&arr, &4), Some(1));  // 3 at index 1
    assert_eq!(floor(&arr, &5), Some(2));  // 5 at index 2
    assert_eq!(ceil(&arr, &10), None);     // No element >= 10
    assert_eq!(floor(&arr, &0), None);     // No element <= 0
}

#[test]
fn test_peak_element() {
    assert_eq!(find_peak(&[1, 3, 5, 7, 6, 4, 2]), 3); // Peak at 7
    assert_eq!(find_peak(&[1, 2, 3, 4, 5]), 4);      // Peak at end
    assert_eq!(find_peak(&[5, 4, 3, 2, 1]), 0);      // Peak at start
}

#[test]
fn test_rotated_array() {
    let arr = vec![4, 5, 6, 7, 0, 1, 2];
    assert_eq!(search_rotated(&arr, &0), Some(4));
    assert_eq!(search_rotated(&arr, &4), Some(0));
    assert_eq!(search_rotated(&arr, &3), None);
    assert_eq!(find_rotation_point(&arr), 4);

    let not_rotated = vec![1, 2, 3, 4, 5];
    assert_eq!(find_rotation_point(&not_rotated), 0);
}

#[test]
fn test_isqrt() {
    assert_eq!(isqrt(0), 0);
    assert_eq!(isqrt(1), 1);
    assert_eq!(isqrt(4), 2);
    assert_eq!(isqrt(8), 2);
    assert_eq!(isqrt(9), 3);
    assert_eq!(isqrt(10), 3);
    assert_eq!(isqrt(100), 10);
    assert_eq!(isqrt(1_000_000_000_000), 1_000_000);
}

#[test]
fn test_kth_smallest_matrix() {
    let matrix = vec![
        vec![1,  5,  9],
        vec![10, 11, 13],
        vec![12, 13, 15],
    ];
    assert_eq!(kth_smallest_matrix(&matrix, 1), 1);
    assert_eq!(kth_smallest_matrix(&matrix, 8), 13);
}

#[test]
fn test_binary_search_on_answer() {
    // Find minimum books per day to finish in D days
    let books = vec![3, 6, 7, 11];
    let days = 8;

    let can_finish = |books_per_day: i64| -> bool {
        let mut days_needed = 0i64;
        let mut current = 0i64;
        for &b in &books {
            if current + b as i64 > books_per_day {
                days_needed += 1;
                current = b as i64;
            } else {
                current += b as i64;
            }
        }
        if current > 0 { days_needed += 1; }
        days_needed <= days
    };

    let min_books = binary_search_answer(11, 27, can_finish);
    assert!(can_finish(min_books));
    assert!(!can_finish(min_books - 1) || min_books == 11);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Standard search correct | 10 |
| Lower/upper bound correct | 15 |
| Ceil/floor correct | 10 |
| Peak element O(log n) | 10 |
| Rotated array search | 15 |
| Rotation point finding | 10 |
| Integer sqrt | 10 |
| Kth in matrix | 10 |
| Search on answer | 10 |
| **Total** | **100** |

## Common Pitfalls

1. **Off-by-one errors**: Always trace through with small examples
2. **Infinite loops**: Ensure `lo` and `hi` always move toward each other
3. **Integer overflow**: Use `lo + (hi - lo) / 2` instead of `(lo + hi) / 2`
4. **Wrong template**: Match the template to the problem type
5. **Rotated array edge cases**: Handle non-rotated arrays correctly

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `binary_search.py`
