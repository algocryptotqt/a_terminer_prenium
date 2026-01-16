# Exercise 01: GCD, LCM & Extended Euclidean Algorithm

## Concepts Covered
- **1.6.2.d-l** Euclidean algorithm, GCD properties, LCM
- **1.6.3.d-k** Extended Euclidean, Bézout's identity, modular inverse

## Objective

Master the fundamental algorithms for computing GCD and their extensions.

## Requirements

### Rust Implementation

```rust
pub mod gcd_lcm {
    /// GCD using Euclidean algorithm
    pub fn gcd(a: u64, b: u64) -> u64;

    /// GCD of multiple numbers
    pub fn gcd_multiple(nums: &[u64]) -> u64;

    /// LCM of two numbers
    pub fn lcm(a: u64, b: u64) -> u64;

    /// LCM of multiple numbers
    pub fn lcm_multiple(nums: &[u64]) -> u64;

    /// Binary GCD (Stein's algorithm) - avoids division
    pub fn binary_gcd(a: u64, b: u64) -> u64;

    /// Extended Euclidean algorithm
    /// Returns (gcd, x, y) where ax + by = gcd
    pub fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64);

    /// Modular multiplicative inverse
    /// Returns x such that (a * x) % m = 1, or None if not exists
    pub fn mod_inverse(a: i64, m: i64) -> Option<i64>;

    /// Solve linear Diophantine equation ax + by = c
    /// Returns (x0, y0) for one solution, or None if no solution
    pub fn solve_diophantine(a: i64, b: i64, c: i64) -> Option<(i64, i64)>;

    /// Count solutions to ax + by = c in range [x_min, x_max], [y_min, y_max]
    pub fn count_diophantine_solutions(
        a: i64, b: i64, c: i64,
        x_min: i64, x_max: i64,
        y_min: i64, y_max: i64,
    ) -> i64;

    /// Chinese Remainder Theorem
    /// Solve x ≡ r[i] (mod m[i]) for all i
    pub fn chinese_remainder(r: &[i64], m: &[i64]) -> Option<(i64, i64)>;  // (x, lcm)

    /// CRT for non-coprime moduli
    pub fn chinese_remainder_general(r: &[i64], m: &[i64]) -> Option<(i64, i64)>;
}

pub mod coprimality {
    /// Euler's totient function φ(n)
    pub fn euler_phi(n: u64) -> u64;

    /// Phi for multiple values up to n (sieve)
    pub fn phi_sieve(n: usize) -> Vec<u64>;

    /// Sum of φ(i) for i = 1 to n
    pub fn phi_sum(n: u64) -> u64;

    /// Count coprime pairs (a, b) where 1 <= a < b <= n
    pub fn count_coprime_pairs(n: u64) -> u64;

    /// Möbius function μ(n)
    pub fn mobius(n: u64) -> i8;

    /// Möbius sieve
    pub fn mobius_sieve(n: usize) -> Vec<i8>;

    /// Mertens function M(n) = Σμ(i) for i=1 to n
    pub fn mertens(n: u64) -> i64;
}

pub mod applications {
    /// Fraction reduction
    pub fn reduce_fraction(num: i64, den: i64) -> (i64, i64);

    /// Add fractions a/b + c/d
    pub fn add_fractions(a: i64, b: i64, c: i64, d: i64) -> (i64, i64);

    /// LCM of denominators (common denominator)
    pub fn common_denominator(fractions: &[(i64, i64)]) -> i64;

    /// Stern-Brocot tree: find fraction closest to x/y with denominator <= max_d
    pub fn best_rational_approx(x: i64, y: i64, max_d: i64) -> (i64, i64);

    /// Farey sequence of order n
    pub fn farey_sequence(n: u64) -> Vec<(u64, u64)>;

    /// Count fractions in Farey sequence
    pub fn farey_length(n: u64) -> u64;
}
```

### Python Implementation

```python
from typing import List, Tuple, Optional

def gcd(a: int, b: int) -> int: ...
def lcm(a: int, b: int) -> int: ...
def gcd_multiple(nums: List[int]) -> int: ...
def lcm_multiple(nums: List[int]) -> int: ...

def extended_gcd(a: int, b: int) -> Tuple[int, int, int]: ...
def mod_inverse(a: int, m: int) -> Optional[int]: ...

def solve_diophantine(a: int, b: int, c: int) -> Optional[Tuple[int, int]]: ...
def chinese_remainder(r: List[int], m: List[int]) -> Optional[Tuple[int, int]]: ...

def euler_phi(n: int) -> int: ...
def phi_sieve(n: int) -> List[int]: ...
def mobius(n: int) -> int: ...

def reduce_fraction(num: int, den: int) -> Tuple[int, int]: ...
def add_fractions(a: int, b: int, c: int, d: int) -> Tuple[int, int]: ...
```

## Test Cases

```rust
#[test]
fn test_gcd() {
    assert_eq!(gcd(48, 18), 6);
    assert_eq!(gcd(0, 5), 5);
    assert_eq!(gcd(5, 0), 5);
    assert_eq!(gcd(17, 23), 1);  // Coprime
}

#[test]
fn test_lcm() {
    assert_eq!(lcm(4, 6), 12);
    assert_eq!(lcm(21, 6), 42);
    assert_eq!(lcm(1, 1), 1);
}

#[test]
fn test_gcd_multiple() {
    assert_eq!(gcd_multiple(&[12, 18, 24]), 6);
    assert_eq!(gcd_multiple(&[17, 23, 29]), 1);
}

#[test]
fn test_binary_gcd() {
    assert_eq!(binary_gcd(48, 18), 6);
    assert_eq!(binary_gcd(1071, 462), 21);
}

#[test]
fn test_extended_gcd() {
    let (g, x, y) = extended_gcd(35, 15);
    assert_eq!(g, 5);
    assert_eq!(35 * x + 15 * y, 5);

    let (g, x, y) = extended_gcd(161, 28);
    assert_eq!(g, 7);
    assert_eq!(161 * x + 28 * y, 7);
}

#[test]
fn test_mod_inverse() {
    assert_eq!(mod_inverse(3, 7), Some(5));   // 3*5 = 15 ≡ 1 (mod 7)
    assert_eq!(mod_inverse(6, 9), None);      // gcd(6,9) = 3 ≠ 1
    assert_eq!(mod_inverse(17, 43), Some(38)); // Verify: 17*38 mod 43 = 1
}

#[test]
fn test_diophantine() {
    // 3x + 5y = 11
    let sol = solve_diophantine(3, 5, 11);
    assert!(sol.is_some());
    let (x, y) = sol.unwrap();
    assert_eq!(3 * x + 5 * y, 11);

    // 6x + 9y = 11 - no solution since gcd(6,9)=3 doesn't divide 11
    assert!(solve_diophantine(6, 9, 11).is_none());
}

#[test]
fn test_chinese_remainder() {
    // x ≡ 2 (mod 3), x ≡ 3 (mod 5), x ≡ 2 (mod 7)
    let (x, m) = chinese_remainder(&[2, 3, 2], &[3, 5, 7]).unwrap();
    assert_eq!(x % 3, 2);
    assert_eq!(x % 5, 3);
    assert_eq!(x % 7, 2);
    assert_eq!(m, 105);  // lcm(3,5,7)
}

#[test]
fn test_euler_phi() {
    assert_eq!(euler_phi(1), 1);
    assert_eq!(euler_phi(2), 1);
    assert_eq!(euler_phi(6), 2);   // 1, 5
    assert_eq!(euler_phi(10), 4);  // 1, 3, 7, 9
    assert_eq!(euler_phi(12), 4);  // 1, 5, 7, 11
}

#[test]
fn test_phi_sieve() {
    let phi = phi_sieve(10);
    assert_eq!(phi[1], 1);
    assert_eq!(phi[6], 2);
    assert_eq!(phi[10], 4);
}

#[test]
fn test_mobius() {
    assert_eq!(mobius(1), 1);
    assert_eq!(mobius(2), -1);
    assert_eq!(mobius(4), 0);   // 4 = 2² has squared factor
    assert_eq!(mobius(6), 1);   // 6 = 2×3, even number of prime factors
    assert_eq!(mobius(30), -1); // 30 = 2×3×5, odd number
}

#[test]
fn test_reduce_fraction() {
    assert_eq!(reduce_fraction(4, 8), (1, 2));
    assert_eq!(reduce_fraction(-6, 9), (-2, 3));
    assert_eq!(reduce_fraction(17, 23), (17, 23));
}

#[test]
fn test_farey() {
    let f3 = farey_sequence(3);
    // F₃ = 0/1, 1/3, 1/2, 2/3, 1/1
    assert_eq!(f3, vec![(0, 1), (1, 3), (1, 2), (2, 3), (1, 1)]);
}

#[test]
fn test_best_rational() {
    // Best approximation to π ≈ 355/113
    let (num, den) = best_rational_approx(314159265, 100000000, 1000);
    // Should be close to 355/113 or 22/7
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| GCD/LCM | 10 |
| Extended Euclidean | 15 |
| Modular inverse | 15 |
| Diophantine equations | 15 |
| Chinese Remainder Theorem | 20 |
| Euler's totient | 15 |
| Fractions & Farey | 5 |
| Edge cases | 5 |
| **Total** | **100** |
