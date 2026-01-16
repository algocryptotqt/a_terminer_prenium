# Exercise 07: Ternary Search & Unimodal Functions

## Concepts Covered
- **1.1.h** Ternary search
- **1.1.25.e** Complexity analysis
- **1.1.25.f** Floating point precision
- **1.1.25.g** Applications
- **1.1.25.h** Alternative to binary search

## Objective

Implement ternary search for finding extrema of unimodal functions. Understand when to use ternary search vs binary search on derivative.

## Requirements

### Rust Implementation

```rust
pub mod ternary_search {
    /// Ternary search for minimum of unimodal function (discrete)
    /// Function decreases then increases
    pub fn find_minimum<F>(lo: i64, hi: i64, f: F) -> i64
    where
        F: Fn(i64) -> i64;

    /// Ternary search for maximum of unimodal function (discrete)
    /// Function increases then decreases
    pub fn find_maximum<F>(lo: i64, hi: i64, f: F) -> i64
    where
        F: Fn(i64) -> i64;

    /// Ternary search for minimum (floating point)
    pub fn find_minimum_float<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64;

    /// Ternary search for maximum (floating point)
    pub fn find_maximum_float<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64;

    /// Golden section search (more efficient than ternary)
    pub fn golden_section_min<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64;

    // Applications

    /// Find rotation angle that minimizes total distance
    pub fn optimal_rotation(points: &[(f64, f64)], target: (f64, f64)) -> f64;

    /// Minimum of quadratic function ax^2 + bx + c
    pub fn quadratic_minimum(a: f64, b: f64, c: f64, lo: f64, hi: f64) -> f64;

    /// Maximum area of rectangle inscribed in semicircle of radius r
    pub fn max_rectangle_in_semicircle(r: f64) -> f64;

    /// Minimum time to reach destination with variable speed
    /// Speed function: v(t) = max_speed * (1 - |t - peak_time| / duration)
    pub fn min_travel_time(
        distance: f64,
        max_speed: f64,
        peak_time: f64,
        duration: f64,
    ) -> f64;

    /// Find point on line segment closest to given point
    pub fn closest_point_on_segment(
        segment: ((f64, f64), (f64, f64)),
        point: (f64, f64),
    ) -> (f64, f64);

    /// Minimize maximum distance from point to set of points
    pub fn minimax_distance(points: &[(f64, f64)]) -> ((f64, f64), f64);

    /// Binary search on derivative (alternative approach)
    pub fn find_minimum_derivative<F, G>(lo: f64, hi: f64, eps: f64, derivative: G) -> f64
    where
        G: Fn(f64) -> f64;
}
```

### Python Implementation

```python
from typing import Callable

def find_minimum(lo: int, hi: int, f: Callable[[int], int]) -> int: ...
def find_maximum(lo: int, hi: int, f: Callable[[int], int]) -> int: ...
def find_minimum_float(lo: float, hi: float, eps: float, f: Callable[[float], float]) -> float: ...
def find_maximum_float(lo: float, hi: float, eps: float, f: Callable[[float], float]) -> float: ...
def golden_section_min(lo: float, hi: float, eps: float, f: Callable[[float], float]) -> float: ...
def optimal_rotation(points: list[tuple[float, float]], target: tuple[float, float]) -> float: ...
def quadratic_minimum(a: float, b: float, c: float, lo: float, hi: float) -> float: ...
def max_rectangle_in_semicircle(r: float) -> float: ...
def closest_point_on_segment(segment: tuple[tuple[float, float], tuple[float, float]], point: tuple[float, float]) -> tuple[float, float]: ...
def minimax_distance(points: list[tuple[float, float]]) -> tuple[tuple[float, float], float]: ...
```

## Algorithm Details

### Ternary Search (Discrete)
```rust
fn ternary_search_min<F: Fn(i64) -> i64>(mut lo: i64, mut hi: i64, f: F) -> i64 {
    while hi - lo > 2 {
        let m1 = lo + (hi - lo) / 3;
        let m2 = hi - (hi - lo) / 3;

        if f(m1) < f(m2) {
            hi = m2;
        } else {
            lo = m1;
        }
    }

    // Check remaining candidates
    let mut best = lo;
    let mut best_val = f(lo);
    for x in lo + 1..=hi {
        let val = f(x);
        if val < best_val {
            best_val = val;
            best = x;
        }
    }
    best
}
```

### Ternary Search (Floating Point)
```rust
fn ternary_search_float<F: Fn(f64) -> f64>(
    mut lo: f64,
    mut hi: f64,
    eps: f64,
    f: F,
) -> f64 {
    // Fixed number of iterations for precision
    for _ in 0..200 {  // ~10^-60 precision
        if hi - lo < eps {
            break;
        }

        let m1 = lo + (hi - lo) / 3.0;
        let m2 = hi - (hi - lo) / 3.0;

        if f(m1) < f(m2) {
            hi = m2;
        } else {
            lo = m1;
        }
    }

    (lo + hi) / 2.0
}
```

### Golden Section Search
More efficient than ternary search (fewer function evaluations):
```
phi = (1 + sqrt(5)) / 2  ≈ 1.618
resphi = 2 - phi         ≈ 0.382

Key insight: Reuse one evaluation point between iterations
```

### Complexity
- Ternary search: O(log₃ n) iterations, but 2 function calls each = O(2 log₃ n)
- Golden section: O(log φ n) iterations, 1 function call each = O(log φ n)
- For n = 10⁹: ternary ≈ 120 calls, golden ≈ 90 calls

## Test Cases

```rust
#[test]
fn test_discrete_minimum() {
    // Parabola-like function: (x - 50)^2
    let f = |x: i64| (x - 50) * (x - 50);
    assert_eq!(find_minimum(0, 100, f), 50);

    // Asymmetric function
    let g = |x: i64| if x < 30 { 100 - x } else { x - 30 };
    assert_eq!(find_minimum(0, 100, g), 30);
}

#[test]
fn test_discrete_maximum() {
    // Inverted parabola
    let f = |x: i64| -((x - 75) * (x - 75));
    assert_eq!(find_maximum(0, 100, f), 75);
}

#[test]
fn test_float_minimum() {
    // x^2 - 4x + 5, minimum at x = 2
    let f = |x: f64| x * x - 4.0 * x + 5.0;
    let result = find_minimum_float(0.0, 10.0, 1e-9, f);
    assert!((result - 2.0).abs() < 1e-6);
}

#[test]
fn test_golden_section() {
    let f = |x: f64| (x - 3.14159).powi(2);
    let result = golden_section_min(0.0, 10.0, 1e-9, f);
    assert!((result - 3.14159).abs() < 1e-6);
}

#[test]
fn test_quadratic_minimum() {
    // f(x) = 2x^2 - 8x + 10, minimum at x = 2
    let result = quadratic_minimum(2.0, -8.0, 10.0, -10.0, 10.0);
    assert!((result - 2.0).abs() < 1e-6);
}

#[test]
fn test_max_rectangle_semicircle() {
    // For radius 1, max area = 1 (at 45 degrees)
    let area = max_rectangle_in_semicircle(1.0);
    assert!((area - 1.0).abs() < 1e-6);

    // For radius 2, max area = 4
    let area = max_rectangle_in_semicircle(2.0);
    assert!((area - 4.0).abs() < 1e-6);
}

#[test]
fn test_closest_point_on_segment() {
    let segment = ((0.0, 0.0), (10.0, 0.0));
    let point = (5.0, 5.0);
    let closest = closest_point_on_segment(segment, point);
    assert!((closest.0 - 5.0).abs() < 1e-6);
    assert!((closest.1 - 0.0).abs() < 1e-6);

    // Point projects outside segment
    let point = (-5.0, 3.0);
    let closest = closest_point_on_segment(segment, point);
    assert!((closest.0 - 0.0).abs() < 1e-6);
    assert!((closest.1 - 0.0).abs() < 1e-6);
}
```

## When to Use Ternary Search

| Use Ternary Search | Use Binary Search on Derivative |
|--------------------|--------------------------------|
| Unimodal function | Monotonic derivative |
| No explicit derivative | Derivative is easy to compute |
| Discrete optimization | Continuous optimization |
| Noisy function | Smooth function |

## Grading

| Criterion | Points |
|-----------|--------|
| Discrete ternary search (min/max) | 15 |
| Floating point ternary search | 15 |
| Golden section search | 15 |
| Quadratic minimum | 10 |
| Max rectangle in semicircle | 15 |
| Closest point on segment | 15 |
| Minimax distance | 15 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `ternary_search.py`
