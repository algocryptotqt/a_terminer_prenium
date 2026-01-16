# Exercise 00: Prime Numbers & Sieves

## Concepts Covered
- **1.6.1.f-l** Sieve complexity, linear sieve, segmented sieve, count primes
- **1.6.2.e-j** SPF, factorization, divisor functions

## Objective

Implement various prime sieve algorithms and factorization techniques.

## Requirements

### Rust Implementation

```rust
pub mod primes {
    /// Basic Sieve of Eratosthenes - O(n log log n)
    pub fn sieve_basic(n: usize) -> Vec<bool>;

    /// Sieve returning list of primes
    pub fn sieve_list(n: usize) -> Vec<usize>;

    /// Linear sieve - O(n), also computes SPF
    pub fn linear_sieve(n: usize) -> (Vec<usize>, Vec<usize>);

    /// Segmented sieve for range [lo, hi]
    pub fn segmented_sieve(lo: u64, hi: u64) -> Vec<u64>;

    /// Count primes up to n
    pub fn count_primes(n: usize) -> usize;

    /// Check if n is prime (simple)
    pub fn is_prime_simple(n: u64) -> bool;

    /// Smallest Prime Factor array
    pub fn compute_spf(n: usize) -> Vec<usize>;

    /// Prime factorization using SPF - O(log n)
    pub fn factorize_spf(n: usize, spf: &[usize]) -> Vec<(usize, usize)>;

    /// Prime factorization (trial division)
    pub fn factorize(n: u64) -> Vec<(u64, u32)>;

    /// Count divisors of n
    pub fn count_divisors(n: u64) -> u64;

    /// Sum of divisors
    pub fn sum_divisors(n: u64) -> u64;

    /// All divisors of n
    pub fn all_divisors(n: u64) -> Vec<u64>;

    /// Sieve to compute number of divisors for 1..n
    pub fn divisor_count_sieve(n: usize) -> Vec<usize>;

    /// Sieve to compute sum of divisors for 1..n
    pub fn divisor_sum_sieve(n: usize) -> Vec<u64>;

    /// Mobius function sieve
    pub fn mobius_sieve(n: usize) -> Vec<i8>;

    /// Nth prime (approximate + sieve)
    pub fn nth_prime(n: usize) -> u64;

    /// Prime gap: smallest prime > n
    pub fn next_prime(n: u64) -> u64;

    /// Twin primes up to n
    pub fn twin_primes(n: usize) -> Vec<(usize, usize)>;
}
```

### Python Implementation

```python
def sieve_basic(n: int) -> list[bool]: ...
def sieve_list(n: int) -> list[int]: ...
def linear_sieve(n: int) -> tuple[list[int], list[int]]: ...
def segmented_sieve(lo: int, hi: int) -> list[int]: ...
def count_primes(n: int) -> int: ...
def is_prime_simple(n: int) -> bool: ...
def compute_spf(n: int) -> list[int]: ...
def factorize_spf(n: int, spf: list[int]) -> list[tuple[int, int]]: ...
def factorize(n: int) -> list[tuple[int, int]]: ...
def count_divisors(n: int) -> int: ...
def sum_divisors(n: int) -> int: ...
def all_divisors(n: int) -> list[int]: ...
def mobius_sieve(n: int) -> list[int]: ...
def nth_prime(n: int) -> int: ...
def next_prime(n: int) -> int: ...
```

## Algorithm Details

### Sieve of Eratosthenes
```rust
fn sieve_basic(n: usize) -> Vec<bool> {
    let mut is_prime = vec![true; n + 1];
    is_prime[0] = false;
    if n >= 1 { is_prime[1] = false; }

    let mut i = 2;
    while i * i <= n {
        if is_prime[i] {
            let mut j = i * i;
            while j <= n {
                is_prime[j] = false;
                j += i;
            }
        }
        i += 1;
    }
    is_prime
}
```

### Linear Sieve with SPF
```rust
fn linear_sieve(n: usize) -> (Vec<usize>, Vec<usize>) {
    let mut spf = vec![0; n + 1];
    let mut primes = Vec::new();

    for i in 2..=n {
        if spf[i] == 0 {
            spf[i] = i;
            primes.push(i);
        }
        for &p in &primes {
            if p > spf[i] || i * p > n {
                break;
            }
            spf[i * p] = p;
        }
    }
    (primes, spf)
}
```

## Test Cases

```rust
#[test]
fn test_sieve() {
    let primes = sieve_list(30);
    assert_eq!(primes, vec![2, 3, 5, 7, 11, 13, 17, 19, 23, 29]);
}

#[test]
fn test_linear_sieve() {
    let (primes, spf) = linear_sieve(20);
    assert_eq!(primes, vec![2, 3, 5, 7, 11, 13, 17, 19]);
    assert_eq!(spf[12], 2);  // 12 = 2 * 6
    assert_eq!(spf[15], 3);  // 15 = 3 * 5
}

#[test]
fn test_segmented_sieve() {
    let primes = segmented_sieve(100, 120);
    assert_eq!(primes, vec![101, 103, 107, 109, 113]);
}

#[test]
fn test_count_primes() {
    assert_eq!(count_primes(10), 4);
    assert_eq!(count_primes(100), 25);
    assert_eq!(count_primes(1_000_000), 78498);
}

#[test]
fn test_factorize() {
    assert_eq!(factorize(12), vec![(2, 2), (3, 1)]);
    assert_eq!(factorize(100), vec![(2, 2), (5, 2)]);
    assert_eq!(factorize(97), vec![(97, 1)]);
}

#[test]
fn test_factorize_spf() {
    let spf = compute_spf(100);
    assert_eq!(factorize_spf(60, &spf), vec![(2, 2), (3, 1), (5, 1)]);
}

#[test]
fn test_divisors() {
    assert_eq!(count_divisors(12), 6);  // 1,2,3,4,6,12
    assert_eq!(sum_divisors(12), 28);   // 1+2+3+4+6+12
    assert_eq!(all_divisors(12), vec![1, 2, 3, 4, 6, 12]);
}

#[test]
fn test_mobius() {
    let mu = mobius_sieve(10);
    assert_eq!(mu[1], 1);
    assert_eq!(mu[2], -1);  // prime
    assert_eq!(mu[4], 0);   // 4 = 2^2
    assert_eq!(mu[6], 1);   // 6 = 2*3
}

#[test]
fn test_nth_prime() {
    assert_eq!(nth_prime(1), 2);
    assert_eq!(nth_prime(10), 29);
    assert_eq!(nth_prime(100), 541);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic sieve | 10 |
| Linear sieve with SPF | 15 |
| Segmented sieve | 15 |
| Count primes | 10 |
| Factorization (both methods) | 15 |
| Divisor functions | 15 |
| Mobius sieve | 10 |
| nth prime | 10 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `primes.py`
