# Exercise 08: Complexity Analysis & Recurrences

## Concepts Covered
- **1.1.1.g-h** Cache hierarchy, Locality
- **1.1.4.h** Proofs
- **1.1.5.i-n** O(n^2), O(n^3), O(2^n), O(n!), comparisons
- **1.1.7.h-l** Master theorem cases, Akra-Bazzi
- **1.1.8.i-j** Amortized analysis examples
- **1.1.26.f-i** Cache effects analysis

## Objective

Develop deep understanding of algorithm complexity through implementation and empirical analysis. Prove complexity bounds and understand cache behavior.

## Requirements

### Rust Implementation

```rust
pub mod complexity {
    use std::time::{Duration, Instant};

    /// Complexity class enumeration
    #[derive(Debug, Clone, Copy, PartialEq)]
    pub enum Complexity {
        O1,           // Constant
        OLogN,        // Logarithmic
        ON,           // Linear
        ONLogN,       // Linearithmic
        ON2,          // Quadratic
        ON3,          // Cubic
        O2N,          // Exponential
        ONFactorial,  // Factorial
    }

    /// Benchmark result
    pub struct BenchmarkResult {
        pub sizes: Vec<usize>,
        pub times: Vec<Duration>,
        pub estimated_complexity: Complexity,
    }

    /// Measure execution time of a function
    pub fn measure_time<F, R>(f: F) -> (R, Duration)
    where
        F: FnOnce() -> R;

    /// Benchmark function with multiple input sizes
    pub fn benchmark<F, R>(
        sizes: &[usize],
        generator: impl Fn(usize) -> Vec<i32>,
        algorithm: F,
    ) -> BenchmarkResult
    where
        F: Fn(&[i32]) -> R;

    /// Estimate complexity from timing data
    pub fn estimate_complexity(sizes: &[usize], times: &[Duration]) -> Complexity;

    // Master Theorem solver

    /// Solve recurrence T(n) = a*T(n/b) + f(n)
    /// Returns the complexity class
    pub fn master_theorem(
        a: f64,      // Number of subproblems
        b: f64,      // Size reduction factor
        k: f64,      // f(n) = O(n^k)
        p: f64,      // f(n) = O(n^k * log^p(n))
    ) -> String;

    /// Check which Master Theorem case applies
    pub fn master_case(a: f64, b: f64, k: f64) -> u8;

    // Amortized Analysis

    /// Dynamic array with amortized O(1) push
    /// Returns total cost and per-operation amortized cost
    pub fn analyze_dynamic_array(operations: usize) -> (usize, f64);

    /// Binary counter increment analysis
    /// Returns total bit flips and amortized cost per increment
    pub fn analyze_binary_counter(increments: usize) -> (usize, f64);

    /// Splay tree access sequence analysis (potential method)
    pub fn analyze_splay_sequence(accesses: &[usize], tree_size: usize) -> f64;

    // Cache Analysis

    /// Matrix multiplication comparing row-major vs cache-oblivious
    pub fn compare_matrix_mult(size: usize) -> (Duration, Duration);

    /// Measure cache misses (simulation)
    pub fn simulate_cache_behavior(
        access_pattern: &[usize],
        cache_size: usize,
        block_size: usize,
    ) -> (usize, usize);  // (hits, misses)

    /// Generate cache-friendly vs cache-unfriendly access patterns
    pub fn row_major_pattern(rows: usize, cols: usize) -> Vec<usize>;
    pub fn column_major_pattern(rows: usize, cols: usize) -> Vec<usize>;

    // Complexity Proofs (return proof steps as strings)

    /// Prove binary search is O(log n)
    pub fn prove_binary_search() -> Vec<String>;

    /// Prove merge sort is O(n log n)
    pub fn prove_merge_sort() -> Vec<String>;

    /// Prove quicksort average case is O(n log n)
    pub fn prove_quicksort_average() -> Vec<String>;
}
```

### Python Implementation

```python
from enum import Enum
from typing import Callable, Any
from dataclasses import dataclass
import time

class Complexity(Enum):
    O1 = "O(1)"
    O_LOG_N = "O(log n)"
    O_N = "O(n)"
    O_N_LOG_N = "O(n log n)"
    O_N2 = "O(n^2)"
    O_N3 = "O(n^3)"
    O_2N = "O(2^n)"
    O_N_FACTORIAL = "O(n!)"

@dataclass
class BenchmarkResult:
    sizes: list[int]
    times: list[float]
    estimated_complexity: Complexity

def measure_time(f: Callable[[], Any]) -> tuple[Any, float]: ...
def benchmark(sizes: list[int], generator: Callable[[int], list[int]], algorithm: Callable[[list[int]], Any]) -> BenchmarkResult: ...
def estimate_complexity(sizes: list[int], times: list[float]) -> Complexity: ...
def master_theorem(a: float, b: float, k: float, p: float = 0) -> str: ...
def master_case(a: float, b: float, k: float) -> int: ...
def analyze_dynamic_array(operations: int) -> tuple[int, float]: ...
def analyze_binary_counter(increments: int) -> tuple[int, float]: ...
def compare_matrix_mult(size: int) -> tuple[float, float]: ...
def simulate_cache_behavior(access_pattern: list[int], cache_size: int, block_size: int) -> tuple[int, int]: ...
```

## Master Theorem Reference

For T(n) = a*T(n/b) + f(n):

**Case 1:** If f(n) = O(n^c) where c < log_b(a)
- Then T(n) = Θ(n^(log_b(a)))

**Case 2:** If f(n) = Θ(n^c * log^k(n)) where c = log_b(a)
- Then T(n) = Θ(n^c * log^(k+1)(n))

**Case 3:** If f(n) = Ω(n^c) where c > log_b(a) and regularity holds
- Then T(n) = Θ(f(n))

## Test Cases

```rust
#[test]
fn test_complexity_estimation() {
    // Linear function
    let sizes = vec![1000, 2000, 4000, 8000, 16000];
    let times_linear = vec![
        Duration::from_micros(100),
        Duration::from_micros(200),
        Duration::from_micros(400),
        Duration::from_micros(800),
        Duration::from_micros(1600),
    ];
    assert_eq!(estimate_complexity(&sizes, &times_linear), Complexity::ON);

    // Quadratic function
    let times_quadratic = vec![
        Duration::from_micros(100),
        Duration::from_micros(400),
        Duration::from_micros(1600),
        Duration::from_micros(6400),
        Duration::from_micros(25600),
    ];
    assert_eq!(estimate_complexity(&sizes, &times_quadratic), Complexity::ON2);
}

#[test]
fn test_master_theorem() {
    // Merge sort: T(n) = 2T(n/2) + O(n)
    assert_eq!(master_case(2.0, 2.0, 1.0), 2);  // Case 2
    assert!(master_theorem(2.0, 2.0, 1.0, 0.0).contains("n log n"));

    // Binary search: T(n) = T(n/2) + O(1)
    assert_eq!(master_case(1.0, 2.0, 0.0), 2);  // Case 2
    assert!(master_theorem(1.0, 2.0, 0.0, 0.0).contains("log n"));

    // Strassen: T(n) = 7T(n/2) + O(n^2)
    assert_eq!(master_case(7.0, 2.0, 2.0), 1);  // Case 1
    // log_2(7) ≈ 2.807
}

#[test]
fn test_amortized_analysis() {
    // Dynamic array: n pushes should cost O(n) total
    let (total_cost, amortized) = analyze_dynamic_array(1000);
    assert!(amortized < 3.0);  // Amortized cost < 3 per operation

    // Binary counter: n increments
    let (total_flips, amortized) = analyze_binary_counter(1000);
    assert!(amortized < 2.0);  // Amortized cost < 2 per increment
}

#[test]
fn test_cache_simulation() {
    // Row-major access (cache-friendly)
    let pattern = row_major_pattern(100, 100);
    let (hits1, misses1) = simulate_cache_behavior(&pattern, 1024, 64);

    // Column-major access (cache-unfriendly)
    let pattern = column_major_pattern(100, 100);
    let (hits2, misses2) = simulate_cache_behavior(&pattern, 1024, 64);

    // Row-major should have fewer misses
    assert!(misses1 < misses2);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Timing and benchmarking | 15 |
| Complexity estimation | 15 |
| Master Theorem implementation | 20 |
| Amortized analysis | 20 |
| Cache simulation | 20 |
| Proof generation | 10 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `complexity.py`
