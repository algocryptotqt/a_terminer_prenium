# Exercise 05: Prefix Sums & Difference Arrays

## Concepts Covered
- **1.1.k** Prefix sums technique
- **1.1.13.h** Difference array
- **1.1.13.i** Difference construction
- **1.1.13.j** Range add operations
- **1.1.13.k** Reconstruct original array
- **1.1.13.l** Applications

## Objective

Master prefix sums and difference arrays for efficient range queries and updates.

## Requirements

### Rust Implementation

```rust
pub mod prefix_sums {
    /// 1D Prefix Sum array
    pub struct PrefixSum {
        prefix: Vec<i64>,
    }

    impl PrefixSum {
        /// Build prefix sum from array - O(n)
        pub fn new(arr: &[i32]) -> Self;

        /// Query sum of range [left, right] inclusive - O(1)
        pub fn range_sum(&self, left: usize, right: usize) -> i64;

        /// Query sum of first k elements - O(1)
        pub fn sum_first_k(&self, k: usize) -> i64;
    }

    /// 2D Prefix Sum for matrix
    pub struct PrefixSum2D {
        prefix: Vec<Vec<i64>>,
    }

    impl PrefixSum2D {
        /// Build 2D prefix sum - O(n*m)
        pub fn new(matrix: &[Vec<i32>]) -> Self;

        /// Query sum of submatrix [(r1,c1), (r2,c2)] - O(1)
        pub fn range_sum(&self, r1: usize, c1: usize, r2: usize, c2: usize) -> i64;
    }

    /// Difference Array for range updates
    pub struct DifferenceArray {
        diff: Vec<i64>,
    }

    impl DifferenceArray {
        /// Create from original array
        pub fn new(arr: &[i32]) -> Self;

        /// Create empty difference array of size n
        pub fn with_size(n: usize) -> Self;

        /// Add value to range [left, right] - O(1)
        pub fn range_add(&mut self, left: usize, right: usize, value: i64);

        /// Reconstruct the original array after all updates - O(n)
        pub fn build(&self) -> Vec<i64>;
    }

    // Standalone functions

    /// Count subarrays with sum equal to k
    pub fn subarrays_with_sum_k(arr: &[i32], k: i32) -> i64;

    /// Find pivot index where left sum equals right sum
    pub fn find_pivot_index(arr: &[i32]) -> Option<usize>;

    /// Maximum subarray sum (Kadane's algorithm)
    pub fn max_subarray_sum(arr: &[i32]) -> i64;

    /// Product of array except self (without division)
    pub fn product_except_self(arr: &[i32]) -> Vec<i64>;

    /// Range XOR queries
    pub fn build_xor_prefix(arr: &[i32]) -> Vec<i32>;
    pub fn range_xor(prefix: &[i32], left: usize, right: usize) -> i32;

    /// Check if subarray can be made zero by flipping signs
    /// (equivalent to: sum of subarray is even)
    pub fn can_make_zero(arr: &[i32], left: usize, right: usize) -> bool;

    /// Equilibrium points: indices where left sum == right sum
    pub fn find_equilibrium_points(arr: &[i32]) -> Vec<usize>;

    /// Maximum sum rectangle in 2D matrix
    pub fn max_sum_rectangle(matrix: &[Vec<i32>]) -> i64;
}
```

### Python Implementation

```python
class PrefixSum:
    def __init__(self, arr: list[int]) -> None: ...
    def range_sum(self, left: int, right: int) -> int: ...
    def sum_first_k(self, k: int) -> int: ...

class PrefixSum2D:
    def __init__(self, matrix: list[list[int]]) -> None: ...
    def range_sum(self, r1: int, c1: int, r2: int, c2: int) -> int: ...

class DifferenceArray:
    def __init__(self, arr: list[int] | None = None, size: int = 0) -> None: ...
    def range_add(self, left: int, right: int, value: int) -> None: ...
    def build(self) -> list[int]: ...

def subarrays_with_sum_k(arr: list[int], k: int) -> int: ...
def find_pivot_index(arr: list[int]) -> int | None: ...
def max_subarray_sum(arr: list[int]) -> int: ...
def product_except_self(arr: list[int]) -> list[int]: ...
def build_xor_prefix(arr: list[int]) -> list[int]: ...
def range_xor(prefix: list[int], left: int, right: int) -> int: ...
def find_equilibrium_points(arr: list[int]) -> list[int]: ...
def max_sum_rectangle(matrix: list[list[int]]) -> int: ...
```

## Key Concepts

### Prefix Sum Construction
```
prefix[0] = 0
prefix[i] = prefix[i-1] + arr[i-1]  for i >= 1

Range sum [l, r] = prefix[r+1] - prefix[l]
```

### Difference Array
```
Given array A, difference array D:
D[0] = A[0]
D[i] = A[i] - A[i-1]  for i >= 1

Property: A[i] = sum(D[0..i])

Range add [l, r] by v:
D[l] += v
D[r+1] -= v  (if r+1 < n)
```

### 2D Prefix Sum
```
prefix[i][j] = sum of rectangle (0,0) to (i-1,j-1)

prefix[i][j] = matrix[i-1][j-1]
             + prefix[i-1][j]
             + prefix[i][j-1]
             - prefix[i-1][j-1]

Range sum (r1,c1) to (r2,c2):
= prefix[r2+1][c2+1]
- prefix[r1][c2+1]
- prefix[r2+1][c1]
+ prefix[r1][c1]
```

## Test Cases

```rust
#[test]
fn test_prefix_sum_1d() {
    let arr = vec![1, 2, 3, 4, 5];
    let ps = PrefixSum::new(&arr);

    assert_eq!(ps.range_sum(0, 4), 15);  // 1+2+3+4+5
    assert_eq!(ps.range_sum(1, 3), 9);   // 2+3+4
    assert_eq!(ps.range_sum(2, 2), 3);   // just 3
    assert_eq!(ps.sum_first_k(3), 6);    // 1+2+3
}

#[test]
fn test_prefix_sum_2d() {
    let matrix = vec![
        vec![1, 2, 3],
        vec![4, 5, 6],
        vec![7, 8, 9],
    ];
    let ps = PrefixSum2D::new(&matrix);

    assert_eq!(ps.range_sum(0, 0, 2, 2), 45);  // entire matrix
    assert_eq!(ps.range_sum(1, 1, 2, 2), 28);  // bottom-right 2x2
    assert_eq!(ps.range_sum(0, 0, 0, 0), 1);   // single element
}

#[test]
fn test_difference_array() {
    let arr = vec![0, 0, 0, 0, 0];
    let mut diff = DifferenceArray::new(&arr);

    diff.range_add(1, 3, 10);  // [0, 10, 10, 10, 0]
    diff.range_add(2, 4, 5);   // [0, 10, 15, 15, 5]

    assert_eq!(diff.build(), vec![0, 10, 15, 15, 5]);
}

#[test]
fn test_subarrays_with_sum() {
    let arr = vec![1, 1, 1];
    assert_eq!(subarrays_with_sum_k(&arr, 2), 2);  // [1,1] at 0-1 and 1-2

    let arr = vec![1, 2, 3];
    assert_eq!(subarrays_with_sum_k(&arr, 3), 2);  // [1,2] and [3]
}

#[test]
fn test_pivot_index() {
    let arr = vec![1, 7, 3, 6, 5, 6];
    assert_eq!(find_pivot_index(&arr), Some(3));  // left=11, right=11

    let arr = vec![1, 2, 3];
    assert_eq!(find_pivot_index(&arr), None);
}

#[test]
fn test_max_subarray() {
    let arr = vec![-2, 1, -3, 4, -1, 2, 1, -5, 4];
    assert_eq!(max_subarray_sum(&arr), 6);  // [4, -1, 2, 1]

    let arr = vec![-1];
    assert_eq!(max_subarray_sum(&arr), -1);

    let arr = vec![5, 4, -1, 7, 8];
    assert_eq!(max_subarray_sum(&arr), 23);
}

#[test]
fn test_product_except_self() {
    let arr = vec![1, 2, 3, 4];
    assert_eq!(product_except_self(&arr), vec![24, 12, 8, 6]);

    let arr = vec![-1, 1, 0, -3, 3];
    assert_eq!(product_except_self(&arr), vec![0, 0, 9, 0, 0]);
}

#[test]
fn test_range_xor() {
    let arr = vec![1, 3, 4, 8];
    let prefix = build_xor_prefix(&arr);
    assert_eq!(range_xor(&prefix, 0, 1), 2);  // 1 ^ 3
    assert_eq!(range_xor(&prefix, 1, 3), 15); // 3 ^ 4 ^ 8
    assert_eq!(range_xor(&prefix, 0, 3), 14); // 1 ^ 3 ^ 4 ^ 8
}

#[test]
fn test_max_sum_rectangle() {
    let matrix = vec![
        vec![1, 2, -1, -4, -20],
        vec![-8, -3, 4, 2, 1],
        vec![3, 8, 10, 1, 3],
        vec![-4, -1, 1, 7, -6],
    ];
    assert_eq!(max_sum_rectangle(&matrix), 29);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| 1D Prefix Sum | 10 |
| 2D Prefix Sum | 15 |
| Difference Array | 15 |
| Subarrays with sum k | 10 |
| Pivot index | 5 |
| Max subarray (Kadane) | 10 |
| Product except self | 10 |
| Range XOR | 10 |
| Max sum rectangle | 15 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `prefix_sums.py`
