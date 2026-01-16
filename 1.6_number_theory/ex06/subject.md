# Exercise 06: FFT & Polynomial Multiplication

## Concepts Covered
- **1.6.14.d-o** Roots of unity, DFT, IDFT, Cooley-Tukey, NTT, applications

## Objective

Implement FFT for O(n log n) polynomial multiplication.

## Requirements

### Rust Implementation

```rust
pub mod fft {
    use std::f64::consts::PI;

    #[derive(Clone, Copy, Debug)]
    pub struct Complex {
        pub re: f64,
        pub im: f64,
    }

    impl Complex {
        pub fn new(re: f64, im: f64) -> Self;
        pub fn from_polar(r: f64, theta: f64) -> Self;
        pub fn conjugate(&self) -> Self;
        pub fn abs(&self) -> f64;
    }

    /// FFT (Cooley-Tukey algorithm)
    pub fn fft(a: &mut [Complex], invert: bool);

    /// Polynomial multiplication using FFT
    pub fn multiply_poly(a: &[i64], b: &[i64]) -> Vec<i64>;

    /// Multiply two big integers represented as digit arrays
    pub fn multiply_bigint(a: &[i32], b: &[i32]) -> Vec<i32>;

    /// Number Theoretic Transform (NTT) - exact arithmetic
    pub fn ntt(a: &mut [i64], invert: bool, mod_p: i64, root: i64);

    /// Polynomial multiplication using NTT (no floating point errors)
    pub fn multiply_poly_ntt(a: &[i64], b: &[i64], mod_p: i64) -> Vec<i64>;

    // Applications

    /// Count ways to make sum using given coins (convolution)
    pub fn count_sums(coins: &[usize], max_sum: usize) -> Vec<i64>;

    /// Multiply all polynomials efficiently
    pub fn multiply_all(polys: &[Vec<i64>]) -> Vec<i64>;

    /// Convolution: c[k] = sum(a[i] * b[k-i])
    pub fn convolve(a: &[i64], b: &[i64]) -> Vec<i64>;
}
```

## FFT Algorithm

```rust
fn fft(a: &mut [Complex], invert: bool) {
    let n = a.len();
    if n == 1 { return; }

    // Bit-reversal permutation
    let mut j = 0;
    for i in 1..n {
        let mut bit = n >> 1;
        while j & bit != 0 {
            j ^= bit;
            bit >>= 1;
        }
        j ^= bit;
        if i < j {
            a.swap(i, j);
        }
    }

    // Cooley-Tukey
    let mut len = 2;
    while len <= n {
        let ang = 2.0 * PI / len as f64 * if invert { -1.0 } else { 1.0 };
        let wlen = Complex::from_polar(1.0, ang);

        for i in (0..n).step_by(len) {
            let mut w = Complex::new(1.0, 0.0);
            for j in 0..len/2 {
                let u = a[i + j];
                let v = a[i + j + len/2] * w;
                a[i + j] = u + v;
                a[i + j + len/2] = u - v;
                w = w * wlen;
            }
        }
        len *= 2;
    }

    if invert {
        for x in a.iter_mut() {
            *x = *x / n as f64;
        }
    }
}
```

## Test Cases

```rust
#[test]
fn test_multiply_poly() {
    let a = vec![1, 2, 3];  // 1 + 2x + 3x²
    let b = vec![4, 5];     // 4 + 5x
    let c = multiply_poly(&a, &b);
    // (1 + 2x + 3x²)(4 + 5x) = 4 + 13x + 22x² + 15x³
    assert_eq!(c, vec![4, 13, 22, 15]);
}

#[test]
fn test_ntt() {
    let a = vec![1, 2, 3];
    let b = vec![4, 5, 6];
    let c = multiply_poly_ntt(&a, &b, 998244353);
    assert_eq!(c, vec![4, 13, 28, 27, 18]);
}

#[test]
fn test_bigint_multiply() {
    let a = vec![9, 9, 9];  // 999
    let b = vec![9, 9, 9];  // 999
    let c = multiply_bigint(&a, &b);
    // 999 * 999 = 998001
    // Verify after carry propagation
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Complex number operations | 10 |
| FFT implementation | 30 |
| Polynomial multiplication | 15 |
| NTT implementation | 25 |
| Big integer multiply | 15 |
| Edge cases | 5 |
| **Total** | **100** |
