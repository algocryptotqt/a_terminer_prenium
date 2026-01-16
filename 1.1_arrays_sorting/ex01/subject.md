# Exercise 01: Sorting Suite

## Concepts Covered
- **1.1.d** 8+ sorting algorithms
- **1.1.e** 3-way quicksort
- **1.1.f** Counting sort
- **1.1.15.g-k** Insertion sort analysis, Shell sort
- **1.1.16.h-m** Merge sort variants
- **1.1.17.j-p** Quick sort optimizations
- **1.1.18.i-k** Heap sort analysis
- **1.1.20.i-o** Non-comparison sorts

## Objective

Implement a comprehensive sorting library with 8+ algorithms, each optimized for different scenarios.

## Requirements

### Rust Implementation

```rust
pub mod sorting {
    /// Bubble sort - Educational, O(n^2)
    pub fn bubble_sort<T: Ord>(arr: &mut [T]);

    /// Selection sort - Minimum swaps, O(n^2)
    pub fn selection_sort<T: Ord>(arr: &mut [T]);

    /// Insertion sort - Fast for nearly sorted, O(n^2)
    pub fn insertion_sort<T: Ord>(arr: &mut [T]);

    /// Shell sort - Improved insertion sort, O(n^1.5)
    pub fn shell_sort<T: Ord>(arr: &mut [T]);

    /// Merge sort - Stable, guaranteed O(n log n)
    pub fn merge_sort<T: Ord + Clone>(arr: &mut [T]);

    /// Bottom-up merge sort - Non-recursive
    pub fn merge_sort_bottom_up<T: Ord + Clone>(arr: &mut [T]);

    /// Quick sort - Average O(n log n), in-place
    pub fn quick_sort<T: Ord>(arr: &mut [T]);

    /// 3-way quick sort - Handles duplicates efficiently
    pub fn quick_sort_3way<T: Ord>(arr: &mut [T]);

    /// Dual-pivot quick sort - Modern approach
    pub fn quick_sort_dual_pivot<T: Ord>(arr: &mut [T]);

    /// Intro sort - Quick + Heap + Insertion hybrid
    pub fn intro_sort<T: Ord>(arr: &mut [T]);

    /// Heap sort - In-place, O(n log n)
    pub fn heap_sort<T: Ord>(arr: &mut [T]);

    /// Counting sort - O(n + k) for integers in range [0, k)
    pub fn counting_sort(arr: &mut [u32], max_value: u32);

    /// Radix sort - O(d * (n + k)) for d-digit numbers
    pub fn radix_sort(arr: &mut [u32]);

    /// Bucket sort - O(n) average for uniform distribution
    pub fn bucket_sort(arr: &mut [f64]);
}
```

### Python Implementation

```python
from typing import TypeVar, Callable, Protocol
from collections.abc import MutableSequence

class Comparable(Protocol):
    def __lt__(self, other: "Comparable") -> bool: ...

T = TypeVar("T", bound=Comparable)

def bubble_sort(arr: MutableSequence[T]) -> None: ...
def selection_sort(arr: MutableSequence[T]) -> None: ...
def insertion_sort(arr: MutableSequence[T]) -> None: ...
def shell_sort(arr: MutableSequence[T]) -> None: ...
def merge_sort(arr: MutableSequence[T]) -> None: ...
def merge_sort_bottom_up(arr: MutableSequence[T]) -> None: ...
def quick_sort(arr: MutableSequence[T]) -> None: ...
def quick_sort_3way(arr: MutableSequence[T]) -> None: ...
def quick_sort_dual_pivot(arr: MutableSequence[T]) -> None: ...
def intro_sort(arr: MutableSequence[T]) -> None: ...
def heap_sort(arr: MutableSequence[T]) -> None: ...
def counting_sort(arr: MutableSequence[int], max_value: int) -> None: ...
def radix_sort(arr: MutableSequence[int]) -> None: ...
def bucket_sort(arr: MutableSequence[float]) -> None: ...
```

## Algorithm Specifications

### 3-Way Quick Sort (Dutch National Flag)
Handles arrays with many duplicates efficiently:
- Partition into three regions: < pivot, == pivot, > pivot
- Elements equal to pivot are in final position
- Reduces comparisons when duplicates exist

```
Algorithm:
1. Choose pivot (median-of-three recommended)
2. Maintain three pointers: lt, i, gt
3. Elements in [lo, lt) < pivot
4. Elements in [lt, i) == pivot
5. Elements in (gt, hi] > pivot
6. Process until i > gt
```

### Intro Sort
Hybrid algorithm that:
1. Starts with quick sort
2. Switches to heap sort when recursion depth exceeds 2*log(n)
3. Uses insertion sort for small subarrays (n < 16)

### Shell Sort Gaps
Use Tokuda's gap sequence:
```
h_k = ceil((9 * (9/4)^k - 4) / 5)
gaps = [1, 4, 9, 20, 46, 103, ...]
```

### Counting Sort
For integers in range [0, max_value]:
1. Count occurrences of each value
2. Compute cumulative counts
3. Place elements in sorted order

### Radix Sort
LSD (Least Significant Digit) approach:
1. Sort by each digit from least to most significant
2. Use counting sort as stable subroutine
3. For 32-bit integers, use base 256 (4 passes)

## Test Cases

```rust
#[test]
fn test_sorting_correctness() {
    let algorithms: Vec<(&str, fn(&mut [i32]))> = vec![
        ("bubble", bubble_sort),
        ("selection", selection_sort),
        ("insertion", insertion_sort),
        ("shell", shell_sort),
        ("merge", merge_sort),
        ("quick", quick_sort),
        ("quick_3way", quick_sort_3way),
        ("heap", heap_sort),
        ("intro", intro_sort),
    ];

    for (name, sort_fn) in algorithms {
        let mut arr = vec![5, 2, 8, 1, 9, 3, 7, 4, 6];
        sort_fn(&mut arr);
        assert_eq!(arr, vec![1, 2, 3, 4, 5, 6, 7, 8, 9], "{} failed", name);
    }
}

#[test]
fn test_stability() {
    // Merge sort and insertion sort should be stable
    #[derive(Clone, PartialEq, Eq, Debug)]
    struct Item { key: i32, order: usize }
    impl Ord for Item {
        fn cmp(&self, other: &Self) -> std::cmp::Ordering {
            self.key.cmp(&other.key)
        }
    }
    impl PartialOrd for Item {
        fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
            Some(self.cmp(other))
        }
    }

    let mut arr = vec![
        Item { key: 2, order: 0 },
        Item { key: 1, order: 1 },
        Item { key: 2, order: 2 },
        Item { key: 1, order: 3 },
    ];
    merge_sort(&mut arr);
    // Elements with same key should maintain relative order
    assert_eq!(arr[0].order, 1); // first 1
    assert_eq!(arr[1].order, 3); // second 1
    assert_eq!(arr[2].order, 0); // first 2
    assert_eq!(arr[3].order, 2); // second 2
}

#[test]
fn test_3way_with_duplicates() {
    let mut arr = vec![1, 1, 1, 1, 1];
    quick_sort_3way(&mut arr);
    assert_eq!(arr, vec![1, 1, 1, 1, 1]);

    let mut arr = vec![2, 1, 2, 1, 2, 1];
    quick_sort_3way(&mut arr);
    assert_eq!(arr, vec![1, 1, 1, 2, 2, 2]);
}

#[test]
fn test_counting_sort() {
    let mut arr: Vec<u32> = vec![4, 2, 2, 8, 3, 3, 1];
    counting_sort(&mut arr, 9);
    assert_eq!(arr, vec![1, 2, 2, 3, 3, 4, 8]);
}

#[test]
fn test_radix_sort() {
    let mut arr: Vec<u32> = vec![170, 45, 75, 90, 802, 24, 2, 66];
    radix_sort(&mut arr);
    assert_eq!(arr, vec![2, 24, 45, 66, 75, 90, 170, 802]);
}

#[test]
fn test_edge_cases() {
    // Empty array
    let mut empty: Vec<i32> = vec![];
    quick_sort(&mut empty);
    assert!(empty.is_empty());

    // Single element
    let mut single = vec![42];
    merge_sort(&mut single);
    assert_eq!(single, vec![42]);

    // Already sorted
    let mut sorted = vec![1, 2, 3, 4, 5];
    heap_sort(&mut sorted);
    assert_eq!(sorted, vec![1, 2, 3, 4, 5]);

    // Reverse sorted
    let mut reverse = vec![5, 4, 3, 2, 1];
    intro_sort(&mut reverse);
    assert_eq!(reverse, vec![1, 2, 3, 4, 5]);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic sorts (bubble, selection, insertion) | 15 |
| Shell sort with proper gaps | 10 |
| Merge sort (recursive + bottom-up) | 15 |
| Quick sort (standard + 3-way + dual-pivot) | 20 |
| Intro sort (hybrid) | 10 |
| Heap sort | 10 |
| Non-comparison sorts (counting, radix, bucket) | 15 |
| Edge cases handled | 5 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs` - All sorting implementations
- `Cargo.toml`

### Python
- `sorting.py` - All sorting implementations

## Performance Notes

Expected time complexities:
| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble | O(n) | O(n^2) | O(n^2) | O(1) | Yes |
| Selection | O(n^2) | O(n^2) | O(n^2) | O(1) | No |
| Insertion | O(n) | O(n^2) | O(n^2) | O(1) | Yes |
| Shell | O(n log n) | O(n^1.5) | O(n^2) | O(1) | No |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick | O(n log n) | O(n log n) | O(n^2) | O(log n) | No |
| 3-way Quick | O(n) | O(n log n) | O(n^2) | O(log n) | No |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Intro | O(n log n) | O(n log n) | O(n log n) | O(log n) | No |
| Counting | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| Radix | O(d*n) | O(d*n) | O(d*n) | O(n+k) | Yes |
