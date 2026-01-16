# Exercise 04: Linear Algebra & Matrices

## Concepts Covered
- **1.6.8.d-l** Matrix operations, matrix exponentiation
- **1.6.9.d-k** Gaussian elimination, matrix inverse, determinant

## Objective

Implement matrix algorithms for solving linear systems and recurrences.

## Requirements

### Rust Implementation

```rust
pub mod matrix {
    /// Generic matrix type
    #[derive(Clone, Debug, PartialEq)]
    pub struct Matrix<T> {
        rows: usize,
        cols: usize,
        data: Vec<Vec<T>>,
    }

    impl<T: Clone + Default> Matrix<T> {
        pub fn new(rows: usize, cols: usize) -> Self;
        pub fn from_vec(data: Vec<Vec<T>>) -> Self;
        pub fn identity(n: usize) -> Self where T: From<u8>;
        pub fn zeros(rows: usize, cols: usize) -> Self;

        pub fn rows(&self) -> usize;
        pub fn cols(&self) -> usize;
        pub fn get(&self, i: usize, j: usize) -> &T;
        pub fn set(&mut self, i: usize, j: usize, val: T);
        pub fn transpose(&self) -> Self;
    }

    /// Matrix operations for numeric types
    impl<T> Matrix<T>
    where
        T: Clone + Default + std::ops::Add<Output = T>
            + std::ops::Sub<Output = T>
            + std::ops::Mul<Output = T>
            + std::ops::Div<Output = T>
            + PartialEq
    {
        pub fn add(&self, other: &Self) -> Self;
        pub fn sub(&self, other: &Self) -> Self;
        pub fn mul(&self, other: &Self) -> Self;
        pub fn scalar_mul(&self, scalar: T) -> Self;
    }

    /// Matrix exponentiation - O(n³ log k)
    pub fn matrix_pow<T>(mat: &Matrix<T>, k: u64) -> Matrix<T>
    where
        T: Clone + Default + std::ops::Add<Output = T>
            + std::ops::Mul<Output = T>
            + From<u8>;

    /// Matrix exponentiation mod m
    pub fn matrix_pow_mod(mat: &Matrix<i64>, k: u64, m: i64) -> Matrix<i64>;
}

pub mod gaussian {
    use super::matrix::Matrix;

    /// Gaussian elimination (row echelon form)
    pub fn row_echelon(mat: &mut Matrix<f64>) -> usize;  // Returns rank

    /// Reduced row echelon form
    pub fn rref(mat: &mut Matrix<f64>) -> usize;

    /// Solve Ax = b
    pub fn solve_linear(a: &Matrix<f64>, b: &[f64]) -> Option<Vec<f64>>;

    /// Solve Ax = b mod m
    pub fn solve_linear_mod(a: &Matrix<i64>, b: &[i64], m: i64) -> Option<Vec<i64>>;

    /// Matrix rank
    pub fn rank(mat: &Matrix<f64>) -> usize;

    /// Matrix determinant
    pub fn determinant(mat: &Matrix<f64>) -> f64;

    /// Determinant mod m
    pub fn determinant_mod(mat: &Matrix<i64>, m: i64) -> i64;

    /// Matrix inverse
    pub fn inverse(mat: &Matrix<f64>) -> Option<Matrix<f64>>;

    /// Matrix inverse mod m
    pub fn inverse_mod(mat: &Matrix<i64>, m: i64) -> Option<Matrix<i64>>;

    /// Null space (kernel) of matrix
    pub fn null_space(mat: &Matrix<f64>) -> Vec<Vec<f64>>;

    /// Column space (image) of matrix
    pub fn column_space(mat: &Matrix<f64>) -> Vec<Vec<f64>>;
}

pub mod linear_recurrence {
    use super::matrix::Matrix;

    /// Solve linear recurrence: f(n) = c1*f(n-1) + c2*f(n-2) + ... + ck*f(n-k)
    /// Returns f(n) given coefficients and initial values
    pub fn linear_recurrence(
        coeffs: &[i64],      // [c1, c2, ..., ck]
        initial: &[i64],     // [f(0), f(1), ..., f(k-1)]
        n: u64,
        m: i64,
    ) -> i64;

    /// Fibonacci using matrix exponentiation
    pub fn fibonacci(n: u64, m: i64) -> i64;

    /// Tribonacci: T(n) = T(n-1) + T(n-2) + T(n-3)
    pub fn tribonacci(n: u64, m: i64) -> i64;

    /// Number of ways to tile 2×n with 1×2 dominoes
    pub fn tiling_recurrence(n: u64, m: i64) -> i64;

    /// Number of paths in graph after k steps
    pub fn paths_after_k_steps(adj_matrix: &Matrix<i64>, k: u64, m: i64) -> Matrix<i64>;

    /// Number of walks of length k from u to v
    pub fn count_walks(adj_matrix: &Matrix<i64>, u: usize, v: usize, k: u64, m: i64) -> i64;

    /// Berlekamp-Massey: find minimal linear recurrence from sequence
    pub fn berlekamp_massey(seq: &[i64], m: i64) -> Vec<i64>;
}

pub mod xor_basis {
    /// XOR basis / Linear basis over GF(2)
    pub struct XorBasis {
        basis: Vec<u64>,
    }

    impl XorBasis {
        pub fn new() -> Self;

        /// Try to add element, returns true if basis expanded
        pub fn add(&mut self, x: u64) -> bool;

        /// Check if x can be represented
        pub fn contains(&self, x: u64) -> bool;

        /// Maximum XOR achievable
        pub fn max_xor(&self) -> u64;

        /// Minimum non-zero XOR achievable
        pub fn min_xor(&self) -> u64;

        /// K-th smallest XOR
        pub fn kth_xor(&self, k: u64) -> Option<u64>;

        /// Size of basis (rank)
        pub fn size(&self) -> usize;

        /// Count distinct XOR values
        pub fn count_distinct(&self) -> u64;
    }

    /// Maximum XOR of any subset
    pub fn max_subset_xor(nums: &[u64]) -> u64;

    /// Can we make target XOR?
    pub fn can_make_xor(nums: &[u64], target: u64) -> bool;
}
```

### Python Implementation

```python
from typing import List, Optional, Tuple

class Matrix:
    def __init__(self, data: List[List[float]]):
        self.data = data
        self.rows = len(data)
        self.cols = len(data[0]) if data else 0

    def __mul__(self, other: 'Matrix') -> 'Matrix': ...
    def __add__(self, other: 'Matrix') -> 'Matrix': ...
    def transpose(self) -> 'Matrix': ...

    @staticmethod
    def identity(n: int) -> 'Matrix': ...
    @staticmethod
    def zeros(rows: int, cols: int) -> 'Matrix': ...

def matrix_pow(mat: Matrix, k: int) -> Matrix: ...
def matrix_pow_mod(mat: List[List[int]], k: int, m: int) -> List[List[int]]: ...

def solve_linear(a: Matrix, b: List[float]) -> Optional[List[float]]: ...
def rank(mat: Matrix) -> int: ...
def determinant(mat: Matrix) -> float: ...
def inverse(mat: Matrix) -> Optional[Matrix]: ...

def linear_recurrence(coeffs: List[int], initial: List[int], n: int, m: int) -> int: ...
def fibonacci(n: int, m: int) -> int: ...
def berlekamp_massey(seq: List[int], m: int) -> List[int]: ...

class XorBasis:
    def __init__(self): ...
    def add(self, x: int) -> bool: ...
    def max_xor(self) -> int: ...
    def contains(self, x: int) -> bool: ...
```

## Test Cases

```rust
#[test]
fn test_matrix_mul() {
    let a = Matrix::from_vec(vec![
        vec![1, 2],
        vec![3, 4],
    ]);
    let b = Matrix::from_vec(vec![
        vec![5, 6],
        vec![7, 8],
    ]);
    let c = a.mul(&b);
    assert_eq!(c.get(0, 0), &19);  // 1*5 + 2*7
    assert_eq!(c.get(1, 1), &50);  // 3*6 + 4*8
}

#[test]
fn test_matrix_pow() {
    let mat = Matrix::from_vec(vec![
        vec![1i64, 1],
        vec![1, 0],
    ]);
    let result = matrix_pow_mod(&mat, 10, 1_000_000_007);
    // This computes Fibonacci
    assert_eq!(*result.get(0, 0), 89);  // F(11)
}

#[test]
fn test_fibonacci_matrix() {
    let m = 1_000_000_007;
    assert_eq!(fibonacci(0, m), 0);
    assert_eq!(fibonacci(1, m), 1);
    assert_eq!(fibonacci(10, m), 55);
    assert_eq!(fibonacci(50, m), 586268941);
}

#[test]
fn test_linear_recurrence() {
    // Fibonacci: f(n) = f(n-1) + f(n-2)
    let coeffs = vec![1, 1];
    let initial = vec![0, 1];
    let m = 1_000_000_007;

    assert_eq!(linear_recurrence(&coeffs, &initial, 10, m), 55);
}

#[test]
fn test_tribonacci() {
    let m = 1_000_000_007;
    // T(0)=0, T(1)=0, T(2)=1, T(n)=T(n-1)+T(n-2)+T(n-3)
    assert_eq!(tribonacci(0, m), 0);
    assert_eq!(tribonacci(3, m), 1);
    assert_eq!(tribonacci(4, m), 2);
    assert_eq!(tribonacci(10, m), 81);
}

#[test]
fn test_determinant() {
    let mat = Matrix::from_vec(vec![
        vec![1.0, 2.0],
        vec![3.0, 4.0],
    ]);
    assert!((determinant(&mat) - (-2.0)).abs() < 1e-9);

    let mat3 = Matrix::from_vec(vec![
        vec![1.0, 2.0, 3.0],
        vec![4.0, 5.0, 6.0],
        vec![7.0, 8.0, 9.0],
    ]);
    assert!((determinant(&mat3) - 0.0).abs() < 1e-9);  // Singular
}

#[test]
fn test_solve_linear() {
    // 2x + y = 5, x + 3y = 6
    let a = Matrix::from_vec(vec![
        vec![2.0, 1.0],
        vec![1.0, 3.0],
    ]);
    let b = vec![5.0, 6.0];
    let x = solve_linear(&a, &b).unwrap();
    assert!((x[0] - 1.8).abs() < 1e-9);
    assert!((x[1] - 1.4).abs() < 1e-9);
}

#[test]
fn test_inverse() {
    let mat = Matrix::from_vec(vec![
        vec![4.0, 7.0],
        vec![2.0, 6.0],
    ]);
    let inv = inverse(&mat).unwrap();

    // Verify A * A^-1 = I
    let product = mat.mul(&inv);
    assert!((product.get(0, 0) - 1.0).abs() < 1e-9);
    assert!((product.get(1, 1) - 1.0).abs() < 1e-9);
}

#[test]
fn test_rank() {
    let mat = Matrix::from_vec(vec![
        vec![1.0, 2.0, 3.0],
        vec![4.0, 5.0, 6.0],
        vec![7.0, 8.0, 9.0],
    ]);
    assert_eq!(rank(&mat), 2);

    let mat2 = Matrix::from_vec(vec![
        vec![1.0, 0.0],
        vec![0.0, 1.0],
    ]);
    assert_eq!(rank(&mat2), 2);
}

#[test]
fn test_xor_basis() {
    let mut basis = XorBasis::new();
    basis.add(1);
    basis.add(2);
    basis.add(4);

    assert_eq!(basis.max_xor(), 7);  // 1^2^4
    assert!(basis.contains(3));       // 1^2
    assert!(basis.contains(7));       // 1^2^4
    assert!(!basis.contains(8));
}

#[test]
fn test_max_subset_xor() {
    assert_eq!(max_subset_xor(&[1, 2, 3, 4]), 7);
    assert_eq!(max_subset_xor(&[8, 1, 2]), 11);  // 8^1^2
}

#[test]
fn test_count_walks() {
    // Graph: 0-1-2 (path)
    let adj = Matrix::from_vec(vec![
        vec![0i64, 1, 0],
        vec![1, 0, 1],
        vec![0, 1, 0],
    ]);

    // Walks of length 2 from 0 to 2
    assert_eq!(count_walks(&adj, 0, 2, 2, 1_000_000_007), 1);

    // Walks of length 4 from 0 to 0
    assert_eq!(count_walks(&adj, 0, 0, 4, 1_000_000_007), 2);
}

#[test]
fn test_berlekamp_massey() {
    // Fibonacci sequence
    let fib = vec![0, 1, 1, 2, 3, 5, 8, 13];
    let rec = berlekamp_massey(&fib, 1_000_000_007);
    // Should find coefficients [1, 1]
    assert_eq!(rec, vec![1, 1]);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Matrix operations | 10 |
| Matrix exponentiation | 20 |
| Gaussian elimination | 15 |
| Linear system solving | 15 |
| Linear recurrences | 15 |
| XOR basis | 15 |
| Berlekamp-Massey | 5 |
| Edge cases | 5 |
| **Total** | **100** |
