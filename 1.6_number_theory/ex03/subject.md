# Exercise 03: Factorization & Divisibility

## Concepts Covered
- **1.6.6.d-l** Prime factorization, divisor enumeration
- **1.6.7.d-k** Pollard's rho, Miller-Rabin primality test

## Objective

Implement efficient factorization algorithms and divisibility functions.

## Requirements

### Rust Implementation

```rust
pub mod factorization {
    /// Trial division factorization - O(√n)
    pub fn factorize(n: u64) -> Vec<(u64, u32)>;  // [(prime, exponent)]

    /// Factorize with prime sieve (for many queries)
    pub fn factorize_with_sieve(n: u64, spf: &[usize]) -> Vec<(u64, u32)>;

    /// Smallest prime factor sieve
    pub fn smallest_prime_factor(max_n: usize) -> Vec<usize>;

    /// Pollard's rho algorithm - O(n^1/4)
    pub fn pollard_rho(n: u64) -> u64;

    /// Complete factorization using Pollard's rho
    pub fn factorize_large(n: u64) -> Vec<(u64, u32)>;

    /// Miller-Rabin primality test
    pub fn is_prime_miller_rabin(n: u64) -> bool;

    /// Deterministic Miller-Rabin for n < 2^64
    pub fn is_prime(n: u64) -> bool;

    /// Check if n is a perfect power (n = a^k for some k > 1)
    pub fn is_perfect_power(n: u64) -> Option<(u64, u32)>;

    /// Check if n is a perfect square
    pub fn is_perfect_square(n: u64) -> bool;

    /// Integer square root
    pub fn isqrt(n: u64) -> u64;

    /// Integer cube root
    pub fn icbrt(n: u64) -> u64;
}

pub mod divisors {
    /// Count divisors - O(√n)
    pub fn count_divisors(n: u64) -> u64;

    /// Sum of divisors σ(n)
    pub fn sum_divisors(n: u64) -> u64;

    /// List all divisors - O(√n)
    pub fn list_divisors(n: u64) -> Vec<u64>;

    /// Divisor count from factorization
    pub fn divisor_count_factored(factors: &[(u64, u32)]) -> u64;

    /// Sum of divisors from factorization
    pub fn divisor_sum_factored(factors: &[(u64, u32)]) -> u64;

    /// Sieve for divisor count up to n
    pub fn divisor_count_sieve(n: usize) -> Vec<u32>;

    /// Sieve for divisor sum up to n
    pub fn divisor_sum_sieve(n: usize) -> Vec<u64>;

    /// Product of divisors
    pub fn product_divisors(n: u64) -> u64;

    /// Number of divisor pairs (d1, d2) where d1 * d2 = n
    pub fn divisor_pairs(n: u64) -> Vec<(u64, u64)>;

    /// Highly composite numbers up to n
    pub fn highly_composite(n: u64) -> Vec<u64>;
}

pub mod number_functions {
    /// Radical of n (product of distinct prime factors)
    pub fn radical(n: u64) -> u64;

    /// Largest prime factor
    pub fn largest_prime_factor(n: u64) -> u64;

    /// Is n squarefree?
    pub fn is_squarefree(n: u64) -> bool;

    /// Omega function: number of distinct prime factors
    pub fn omega(n: u64) -> u32;

    /// Big omega: number of prime factors with multiplicity
    pub fn big_omega(n: u64) -> u32;

    /// Liouville function λ(n) = (-1)^Ω(n)
    pub fn liouville(n: u64) -> i8;

    /// Carmichael function λ(n)
    pub fn carmichael(n: u64) -> u64;

    /// Is n a powerful number? (p | n implies p² | n)
    pub fn is_powerful(n: u64) -> bool;

    /// Is n a smooth number? (all prime factors <= bound)
    pub fn is_smooth(n: u64, bound: u64) -> bool;
}

pub mod advanced {
    /// Compute f(d) for all divisors d of n
    pub fn divisor_transform<F, T>(n: u64, f: F) -> Vec<(u64, T)>
    where
        F: Fn(u64) -> T;

    /// Dirichlet convolution f * g
    pub fn dirichlet_convolution(
        f: &[i64],
        g: &[i64],
        n: usize,
    ) -> Vec<i64>;

    /// Möbius inversion: g(n) = Σ_{d|n} f(d) → f(n) = Σ_{d|n} μ(n/d)g(d)
    pub fn mobius_inversion(g: &[i64]) -> Vec<i64>;

    /// Count pairs (a, b) where lcm(a, b) = n
    pub fn count_lcm_pairs(n: u64) -> u64;

    /// Count pairs (a, b) where gcd(a, b) = g and a, b <= n
    pub fn count_gcd_pairs(n: u64, g: u64) -> u64;

    /// Sum of gcd(i, n) for i = 1 to n
    pub fn gcd_sum(n: u64) -> u64;
}
```

### Python Implementation

```python
from typing import List, Tuple, Optional

def factorize(n: int) -> List[Tuple[int, int]]: ...
def factorize_large(n: int) -> List[Tuple[int, int]]: ...
def pollard_rho(n: int) -> int: ...
def is_prime(n: int) -> bool: ...
def is_prime_miller_rabin(n: int) -> bool: ...

def count_divisors(n: int) -> int: ...
def sum_divisors(n: int) -> int: ...
def list_divisors(n: int) -> List[int]: ...
def divisor_count_sieve(n: int) -> List[int]: ...

def radical(n: int) -> int: ...
def omega(n: int) -> int: ...
def is_squarefree(n: int) -> bool: ...
def carmichael(n: int) -> int: ...
```

## Test Cases

```rust
#[test]
fn test_factorize() {
    assert_eq!(factorize(60), vec![(2, 2), (3, 1), (5, 1)]);
    assert_eq!(factorize(1), vec![]);
    assert_eq!(factorize(17), vec![(17, 1)]);
    assert_eq!(factorize(100), vec![(2, 2), (5, 2)]);
}

#[test]
fn test_miller_rabin() {
    assert!(is_prime(2));
    assert!(is_prime(3));
    assert!(!is_prime(4));
    assert!(is_prime(104729));  // 10000th prime
    assert!(is_prime(1_000_000_007));
    assert!(!is_prime(1_000_000_011));

    // Large primes
    assert!(is_prime(999999999999999989));
}

#[test]
fn test_pollard_rho() {
    let n = 1000000007u64 * 1000000009u64;
    let factors = factorize_large(n);
    assert_eq!(factors.len(), 2);
}

#[test]
fn test_perfect_power() {
    assert_eq!(is_perfect_power(8), Some((2, 3)));
    assert_eq!(is_perfect_power(9), Some((3, 2)));
    assert_eq!(is_perfect_power(64), Some((2, 6)));  // or (4, 3) or (8, 2)
    assert_eq!(is_perfect_power(7), None);
}

#[test]
fn test_count_divisors() {
    assert_eq!(count_divisors(1), 1);
    assert_eq!(count_divisors(12), 6);   // 1,2,3,4,6,12
    assert_eq!(count_divisors(60), 12);
    assert_eq!(count_divisors(100), 9);
}

#[test]
fn test_sum_divisors() {
    assert_eq!(sum_divisors(1), 1);
    assert_eq!(sum_divisors(12), 28);    // 1+2+3+4+6+12
    assert_eq!(sum_divisors(6), 12);     // Perfect number!
    assert_eq!(sum_divisors(28), 56);    // Perfect number!
}

#[test]
fn test_list_divisors() {
    assert_eq!(list_divisors(12), vec![1, 2, 3, 4, 6, 12]);
    assert_eq!(list_divisors(1), vec![1]);
    assert_eq!(list_divisors(17), vec![1, 17]);
}

#[test]
fn test_divisor_sieve() {
    let counts = divisor_count_sieve(10);
    assert_eq!(counts[1], 1);
    assert_eq!(counts[6], 4);   // 1,2,3,6
    assert_eq!(counts[10], 4);  // 1,2,5,10
}

#[test]
fn test_radical() {
    assert_eq!(radical(12), 6);   // 2 × 3
    assert_eq!(radical(100), 10); // 2 × 5
    assert_eq!(radical(17), 17);
}

#[test]
fn test_omega() {
    assert_eq!(omega(12), 2);   // 2, 3
    assert_eq!(omega(60), 3);   // 2, 3, 5
    assert_eq!(omega(17), 1);
}

#[test]
fn test_big_omega() {
    assert_eq!(big_omega(12), 3);  // 2² × 3
    assert_eq!(big_omega(60), 4);  // 2² × 3 × 5
    assert_eq!(big_omega(8), 3);   // 2³
}

#[test]
fn test_squarefree() {
    assert!(is_squarefree(6));    // 2 × 3
    assert!(!is_squarefree(12));  // 2² × 3
    assert!(is_squarefree(30));   // 2 × 3 × 5
    assert!(!is_squarefree(100)); // 2² × 5²
}

#[test]
fn test_carmichael() {
    assert_eq!(carmichael(1), 1);
    assert_eq!(carmichael(2), 1);
    assert_eq!(carmichael(8), 2);
    assert_eq!(carmichael(12), 2);
    assert_eq!(carmichael(100), 20);
}

#[test]
fn test_highly_composite() {
    let hc = highly_composite(100);
    // 1, 2, 4, 6, 12, 24, 36, 48, 60
    assert!(hc.contains(&1));
    assert!(hc.contains(&12));
    assert!(hc.contains(&60));
}

#[test]
fn test_gcd_sum() {
    // Σ gcd(i, 6) for i = 1..6 = gcd(1,6)+gcd(2,6)+...+gcd(6,6) = 1+2+3+2+1+6 = 15
    assert_eq!(gcd_sum(6), 15);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Trial division factorization | 10 |
| Pollard's rho | 20 |
| Miller-Rabin | 20 |
| Divisor functions | 15 |
| Divisor sieves | 10 |
| Number-theoretic functions | 15 |
| Dirichlet convolution | 5 |
| Edge cases | 5 |
| **Total** | **100** |
