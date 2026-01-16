# Exercise 06: Coordinate Compression

## Concepts Covered
- **1.1.14.e** Implementation
- **1.1.14.f-h** Applications
- **1.1.14.i** Complexity analysis

## Objective

Implement coordinate compression to map large sparse values to small dense indices. This technique is essential when values can be very large but the number of unique values is small.

## Requirements

### Rust Implementation

```rust
pub mod coordinate_compression {
    use std::collections::HashMap;

    /// Coordinate compressor for any Ord type
    pub struct Compressor<T: Ord + Clone> {
        sorted_values: Vec<T>,
        index_map: HashMap<T, usize>,
    }

    impl<T: Ord + Clone + std::hash::Hash> Compressor<T> {
        /// Build compressor from array of values
        pub fn new(values: &[T]) -> Self;

        /// Compress a single value to its index
        pub fn compress(&self, value: &T) -> usize;

        /// Decompress index back to original value
        pub fn decompress(&self, index: usize) -> &T;

        /// Number of unique compressed values
        pub fn size(&self) -> usize;

        /// Compress entire array
        pub fn compress_all(&self, values: &[T]) -> Vec<usize>;
    }

    /// Compress coordinates for 2D points
    pub struct Compressor2D {
        x_comp: Compressor<i64>,
        y_comp: Compressor<i64>,
    }

    impl Compressor2D {
        pub fn new(points: &[(i64, i64)]) -> Self;
        pub fn compress_point(&self, point: (i64, i64)) -> (usize, usize);
        pub fn decompress_point(&self, compressed: (usize, usize)) -> (i64, i64);
    }

    // Applications

    /// Count rectangles containing query points
    /// Uses coordinate compression + 2D prefix sums
    pub fn count_points_in_rectangles(
        points: &[(i64, i64)],
        rectangles: &[(i64, i64, i64, i64)],  // (x1, y1, x2, y2)
    ) -> Vec<i64>;

    /// Number of distinct elements in range queries
    /// Uses coordinate compression + offline processing
    pub fn distinct_elements_in_ranges(
        arr: &[i32],
        queries: &[(usize, usize)],
    ) -> Vec<usize>;

    /// Count of smaller elements to the right
    /// Uses coordinate compression + Fenwick tree
    pub fn count_smaller_to_right(arr: &[i32]) -> Vec<i32>;

    /// Longest Increasing Subsequence using compression + DP
    pub fn lis_with_compression(arr: &[i64]) -> usize;

    /// Range frequency query: count of value in range
    pub struct RangeFrequency {
        // Implementation
    }

    impl RangeFrequency {
        pub fn new(arr: &[i32]) -> Self;
        pub fn query(&self, left: usize, right: usize, value: i32) -> usize;
    }
}
```

### Python Implementation

```python
from typing import TypeVar, Generic

T = TypeVar("T")

class Compressor(Generic[T]):
    def __init__(self, values: list[T]) -> None: ...
    def compress(self, value: T) -> int: ...
    def decompress(self, index: int) -> T: ...
    def size(self) -> int: ...
    def compress_all(self, values: list[T]) -> list[int]: ...

class Compressor2D:
    def __init__(self, points: list[tuple[int, int]]) -> None: ...
    def compress_point(self, point: tuple[int, int]) -> tuple[int, int]: ...
    def decompress_point(self, compressed: tuple[int, int]) -> tuple[int, int]: ...

def count_points_in_rectangles(
    points: list[tuple[int, int]],
    rectangles: list[tuple[int, int, int, int]]
) -> list[int]: ...

def distinct_elements_in_ranges(
    arr: list[int],
    queries: list[tuple[int, int]]
) -> list[int]: ...

def count_smaller_to_right(arr: list[int]) -> list[int]: ...

def lis_with_compression(arr: list[int]) -> int: ...

class RangeFrequency:
    def __init__(self, arr: list[int]) -> None: ...
    def query(self, left: int, right: int, value: int) -> int: ...
```

## Algorithm Details

### Basic Coordinate Compression
```rust
fn compress(values: &[i64]) -> (Vec<usize>, Vec<i64>) {
    let mut sorted: Vec<i64> = values.to_vec();
    sorted.sort();
    sorted.dedup();

    let index_map: HashMap<i64, usize> = sorted
        .iter()
        .enumerate()
        .map(|(i, &v)| (v, i))
        .collect();

    let compressed: Vec<usize> = values
        .iter()
        .map(|v| *index_map.get(v).unwrap())
        .collect();

    (compressed, sorted)  // compressed values and mapping for decompression
}
```

### Count Smaller to Right
Uses coordinate compression + Fenwick tree:
1. Compress all values to [0, n-1]
2. Process from right to left
3. For each element, query Fenwick tree for count of smaller elements
4. Update Fenwick tree with current element

### LIS with Compression
1. Compress values to [0, k-1] where k = unique count
2. Use DP array of size k
3. For each element, binary search for insertion point
4. This enables O(n log n) LIS

## Test Cases

```rust
#[test]
fn test_basic_compression() {
    let values = vec![100, 200, 50, 200, 100];
    let comp = Compressor::new(&values);

    assert_eq!(comp.size(), 3);  // 50, 100, 200
    assert_eq!(comp.compress(&50), 0);
    assert_eq!(comp.compress(&100), 1);
    assert_eq!(comp.compress(&200), 2);
    assert_eq!(comp.decompress(1), &100);
}

#[test]
fn test_large_values() {
    let values = vec![1_000_000_000, 1, 500_000_000];
    let comp = Compressor::new(&values);

    assert_eq!(comp.size(), 3);
    assert_eq!(comp.compress_all(&values), vec![2, 0, 1]);
}

#[test]
fn test_2d_compression() {
    let points = vec![(1000, 2000), (500, 3000), (1000, 1000)];
    let comp = Compressor2D::new(&points);

    assert_eq!(comp.compress_point((500, 1000)), (0, 0));
    assert_eq!(comp.compress_point((1000, 2000)), (1, 1));
    assert_eq!(comp.compress_point((1000, 3000)), (1, 2));
}

#[test]
fn test_count_smaller_to_right() {
    let arr = vec![5, 2, 6, 1];
    assert_eq!(count_smaller_to_right(&arr), vec![2, 1, 1, 0]);

    let arr = vec![2, 0, 1];
    assert_eq!(count_smaller_to_right(&arr), vec![2, 0, 0]);
}

#[test]
fn test_distinct_in_ranges() {
    let arr = vec![1, 1, 2, 1, 3];
    let queries = vec![(0, 4), (1, 3), (2, 4)];
    assert_eq!(distinct_elements_in_ranges(&arr, &queries), vec![3, 2, 3]);
}

#[test]
fn test_lis() {
    let arr = vec![10, 9, 2, 5, 3, 7, 101, 18];
    assert_eq!(lis_with_compression(&arr), 4);  // [2, 3, 7, 18]

    let arr = vec![0, 1, 0, 3, 2, 3];
    assert_eq!(lis_with_compression(&arr), 4);
}

#[test]
fn test_range_frequency() {
    let arr = vec![12, 33, 4, 56, 22, 2, 34, 33, 22, 12, 34, 56];
    let rf = RangeFrequency::new(&arr);

    assert_eq!(rf.query(1, 2, 4), 1);
    assert_eq!(rf.query(0, 11, 33), 2);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic 1D compression | 15 |
| 2D coordinate compression | 15 |
| Count smaller to right | 20 |
| Distinct elements in ranges | 15 |
| LIS with compression | 15 |
| Range frequency queries | 15 |
| Edge cases handled | 5 |
| **Total** | **100** |

## Complexity Requirements

| Operation | Time | Space |
|-----------|------|-------|
| Build compressor | O(n log n) | O(n) |
| Compress single value | O(log n) or O(1) | O(1) |
| Count smaller to right | O(n log n) | O(n) |
| Distinct in ranges (offline) | O((n+q) log n) | O(n) |
| LIS | O(n log n) | O(n) |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `coordinate_compression.py`
