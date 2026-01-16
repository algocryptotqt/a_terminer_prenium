# Exercise 02: Modular Arithmetic

## Concepts Covered
- **1.6.4.d-l** Modular addition, multiplication, exponentiation
- **1.6.5.d-k** Fermat's little theorem, Euler's theorem, Wilson's theorem

## Objective

Implement efficient modular arithmetic operations and their applications.

## Requirements

### Rust Implementation

```rust
pub mod modular {
    pub const MOD: i64 = 1_000_000_007;

    /// Modular addition (handles negative numbers)
    pub fn mod_add(a: i64, b: i64, m: i64) -> i64;

    /// Modular subtraction
    pub fn mod_sub(a: i64, b: i64, m: i64) -> i64;

    /// Modular multiplication (handles overflow)
    pub fn mod_mul(a: i64, b: i64, m: i64) -> i64;

    /// Modular multiplication for large numbers using u128
    pub fn mod_mul_large(a: u64, b: u64, m: u64) -> u64;

    /// Binary modular exponentiation - O(log exp)
    pub fn mod_pow(base: i64, exp: u64, m: i64) -> i64;

    /// Modular inverse using Fermat's little theorem (m must be prime)
    pub fn mod_inv_fermat(a: i64, m: i64) -> i64;

    /// Modular inverse using extended GCD (works for any coprime m)
    pub fn mod_inv_ext(a: i64, m: i64) -> Option<i64>;

    /// Modular division: a / b mod m
    pub fn mod_div(a: i64, b: i64, m: i64) -> Option<i64>;

    /// Batch modular inverse - O(n) using prefix products
    pub fn batch_mod_inv(arr: &[i64], m: i64) -> Vec<i64>;

    /// Solve a^x ≡ b (mod m) - Baby-step Giant-step
    pub fn discrete_log(a: i64, b: i64, m: i64) -> Option<u64>;

    /// Primitive root modulo m
    pub fn primitive_root(m: i64) -> Option<i64>;

    /// Check if g is primitive root of m
    pub fn is_primitive_root(g: i64, m: i64) -> bool;

    /// Nth root mod p: find x such that x^n ≡ a (mod p)
    pub fn nth_root(a: i64, n: i64, p: i64) -> Option<i64>;
}

pub mod theorems {
    /// Verify Fermat's little theorem: a^(p-1) ≡ 1 (mod p) for prime p
    pub fn verify_fermat(a: i64, p: i64) -> bool;

    /// Verify Euler's theorem: a^φ(n) ≡ 1 (mod n) for gcd(a,n)=1
    pub fn verify_euler(a: i64, n: i64) -> bool;

    /// Wilson's theorem: (p-1)! ≡ -1 (mod p) for prime p
    pub fn verify_wilson(p: i64) -> bool;

    /// Compute (n!) mod p using Wilson's theorem optimization
    pub fn factorial_mod_prime(n: u64, p: u64) -> u64;

    /// Quadratic residue check: does x² ≡ a (mod p) have solution?
    pub fn is_quadratic_residue(a: i64, p: i64) -> bool;

    /// Legendre symbol (a/p)
    pub fn legendre_symbol(a: i64, p: i64) -> i8;

    /// Tonelli-Shanks: square root mod p
    pub fn sqrt_mod_prime(a: i64, p: i64) -> Option<i64>;
}

pub mod combinations {
    /// Modular factorial
    pub fn factorial_mod(n: usize, m: i64) -> i64;

    /// Precompute factorials mod m
    pub fn factorial_table(max_n: usize, m: i64) -> Vec<i64>;

    /// Precompute inverse factorials mod m
    pub fn inv_factorial_table(max_n: usize, m: i64) -> Vec<i64>;

    /// Binomial coefficient C(n, k) mod m
    pub fn binomial_mod(n: usize, k: usize, m: i64) -> i64;

    /// Lucas' theorem for C(n, k) mod p (prime)
    pub fn lucas(n: u64, k: u64, p: u64) -> u64;

    /// C(n, k) mod m for large n using Lucas
    pub fn binomial_large(n: u64, k: u64, p: u64) -> u64;

    /// Multinomial coefficient mod m
    pub fn multinomial_mod(n: usize, ks: &[usize], m: i64) -> i64;

    /// Catalan number mod m
    pub fn catalan_mod(n: usize, m: i64) -> i64;

    /// Stirling numbers of second kind S(n, k) mod m
    pub fn stirling2_mod(n: usize, k: usize, m: i64) -> i64;
}

/// A modular integer wrapper
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub struct ModInt<const M: i64> {
    val: i64,
}

impl<const M: i64> ModInt<M> {
    pub fn new(v: i64) -> Self;
    pub fn val(&self) -> i64;
    pub fn pow(self, exp: u64) -> Self;
    pub fn inv(self) -> Self;
}

impl<const M: i64> std::ops::Add for ModInt<M> { ... }
impl<const M: i64> std::ops::Sub for ModInt<M> { ... }
impl<const M: i64> std::ops::Mul for ModInt<M> { ... }
impl<const M: i64> std::ops::Div for ModInt<M> { ... }
```

### Python Implementation

```python
from typing import List, Optional, Tuple

MOD = 10**9 + 7

def mod_add(a: int, b: int, m: int) -> int: ...
def mod_sub(a: int, b: int, m: int) -> int: ...
def mod_mul(a: int, b: int, m: int) -> int: ...
def mod_pow(base: int, exp: int, m: int) -> int: ...
def mod_inv(a: int, m: int) -> int: ...
def mod_div(a: int, b: int, m: int) -> Optional[int]: ...

def discrete_log(a: int, b: int, m: int) -> Optional[int]: ...
def primitive_root(m: int) -> Optional[int]: ...
def sqrt_mod_prime(a: int, p: int) -> Optional[int]: ...

def factorial_mod(n: int, m: int) -> int: ...
def binomial_mod(n: int, k: int, m: int) -> int: ...
def lucas(n: int, k: int, p: int) -> int: ...
def catalan_mod(n: int, m: int) -> int: ...

class ModInt:
    def __init__(self, val: int, mod: int = MOD): ...
    def __add__(self, other): ...
    def __sub__(self, other): ...
    def __mul__(self, other): ...
    def __truediv__(self, other): ...
    def __pow__(self, exp: int): ...
    def inv(self) -> 'ModInt': ...
```

## Test Cases

```rust
#[test]
fn test_mod_arithmetic() {
    let m = 1_000_000_007;
    assert_eq!(mod_add(500_000_000, 600_000_000, m), 100_000_000 - 7);
    assert_eq!(mod_sub(100, 200, m), m - 100);
    assert_eq!(mod_mul(100000, 100000, m), 10_000_000_000 % m);
}

#[test]
fn test_mod_pow() {
    assert_eq!(mod_pow(2, 10, 1000), 24);     // 1024 mod 1000
    assert_eq!(mod_pow(3, 100, 1_000_000_007), 981985467);
    assert_eq!(mod_pow(5, 0, 13), 1);
}

#[test]
fn test_mod_inv() {
    let m = 1_000_000_007;
    let inv3 = mod_inv_fermat(3, m);
    assert_eq!(mod_mul(3, inv3, m), 1);

    let inv7 = mod_inv_ext(7, 11).unwrap();
    assert_eq!((7 * inv7) % 11, 1);
}

#[test]
fn test_batch_inverse() {
    let arr = vec![2, 3, 4, 5];
    let m = 1_000_000_007;
    let invs = batch_mod_inv(&arr, m);

    for i in 0..arr.len() {
        assert_eq!(mod_mul(arr[i], invs[i], m), 1);
    }
}

#[test]
fn test_discrete_log() {
    // 2^x ≡ 3 (mod 5)
    let x = discrete_log(2, 3, 5);
    assert!(x.is_some());
    assert_eq!(mod_pow(2, x.unwrap(), 5), 3);

    // 3^x ≡ 13 (mod 17)
    let x = discrete_log(3, 13, 17).unwrap();
    assert_eq!(mod_pow(3, x, 17), 13);
}

#[test]
fn test_primitive_root() {
    // 3 is a primitive root of 7
    assert!(is_primitive_root(3, 7));

    let g = primitive_root(13).unwrap();
    assert!(is_primitive_root(g, 13));
}

#[test]
fn test_sqrt_mod() {
    // √2 mod 7: x² ≡ 2 (mod 7)
    let x = sqrt_mod_prime(2, 7).unwrap();
    assert_eq!((x * x) % 7, 2);

    // √5 mod 11
    let x = sqrt_mod_prime(5, 11).unwrap();
    assert_eq!((x * x) % 11, 5);

    // No square root
    assert!(sqrt_mod_prime(3, 7).is_none());  // 3 is not QR mod 7
}

#[test]
fn test_binomial() {
    let m = 1_000_000_007;
    assert_eq!(binomial_mod(5, 2, m), 10);
    assert_eq!(binomial_mod(10, 5, m), 252);
    assert_eq!(binomial_mod(100, 50, m), 538992043);
}

#[test]
fn test_lucas() {
    // C(100, 10) mod 13
    let result = lucas(100, 10, 13);
    assert_eq!(result, binomial_mod(100, 10, 13) as u64);

    // Large numbers
    let result = lucas(1_000_000_000_000, 1_000_000, 13);
    // Verify using direct computation
}

#[test]
fn test_catalan() {
    let m = 1_000_000_007;
    assert_eq!(catalan_mod(0, m), 1);
    assert_eq!(catalan_mod(1, m), 1);
    assert_eq!(catalan_mod(2, m), 2);
    assert_eq!(catalan_mod(3, m), 5);
    assert_eq!(catalan_mod(4, m), 14);
    assert_eq!(catalan_mod(10, m), 16796);
}

#[test]
fn test_fermat_theorem() {
    assert!(verify_fermat(2, 7));   // 2^6 ≡ 1 (mod 7)
    assert!(verify_fermat(3, 11));  // 3^10 ≡ 1 (mod 11)
}

#[test]
fn test_wilson() {
    assert!(verify_wilson(5));   // 4! = 24 ≡ -1 (mod 5)
    assert!(verify_wilson(7));   // 6! = 720 ≡ -1 (mod 7)
}

#[test]
fn test_modint() {
    type M = ModInt<1_000_000_007>;

    let a = M::new(500000000);
    let b = M::new(600000000);

    let c = a + b;
    assert_eq!(c.val(), 99999993);

    let d = M::new(3);
    let e = d.inv();
    assert_eq!((d * e).val(), 1);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic mod operations | 10 |
| Modular exponentiation | 15 |
| Modular inverse | 15 |
| Discrete logarithm | 15 |
| Square root mod p | 15 |
| Binomial coefficients | 15 |
| Lucas theorem | 10 |
| Edge cases | 5 |
| **Total** | **100** |
