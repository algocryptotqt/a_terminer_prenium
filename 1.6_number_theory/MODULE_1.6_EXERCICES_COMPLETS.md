# MODULE 1.6 - NUMBER THEORY & MATHEMATICS
## 202 Concepts - 20 Exercices Progressifs

---

# BLOC A: Arithmétique de Base (Exercices 01-05)

## Exercice 01: Divisibility Foundations
**Concepts couverts**: 1.6.1.a-j (10 concepts)
**Difficulté**: ⭐⭐

```rust
// GCD et LCM
pub fn gcd(a: u64, b: u64) -> u64;                            // 1.6.1.a (Euclide)
pub fn gcd_extended(a: i64, b: i64) -> (i64, i64, i64);       // 1.6.1.b (ax + by = gcd)
pub fn lcm(a: u64, b: u64) -> u64;                            // 1.6.1.c

// Divisibility
pub fn divisors(n: u64) -> Vec<u64>;                          // 1.6.1.d
pub fn divisor_count(n: u64) -> u64;                          // 1.6.1.e
pub fn divisor_sum(n: u64) -> u64;                            // 1.6.1.f

// Perfect numbers
pub fn is_perfect(n: u64) -> bool;                            // 1.6.1.g
pub fn is_abundant(n: u64) -> bool;                           // 1.6.1.h
pub fn is_deficient(n: u64) -> bool;                          // 1.6.1.i

// GCD properties
pub fn gcd_array(nums: &[u64]) -> u64;                        // 1.6.1.j
```

---

## Exercice 02: Prime Numbers
**Concepts couverts**: 1.6.2.a-j (10 concepts)
**Difficulté**: ⭐⭐

```rust
pub fn is_prime(n: u64) -> bool;                              // 1.6.2.a (trial division)
pub fn is_prime_fast(n: u64) -> bool;                         // 1.6.2.b (optimized)

// Sieve of Eratosthenes
pub fn sieve(n: usize) -> Vec<bool>;                          // 1.6.2.c
pub fn primes_up_to(n: usize) -> Vec<u64>;                    // 1.6.2.d

// Segmented Sieve
pub fn segmented_sieve(lo: u64, hi: u64) -> Vec<u64>;         // 1.6.2.e

// Prime factorization
pub fn factorize(n: u64) -> Vec<(u64, u32)>;                  // 1.6.2.f
pub fn smallest_prime_factor(n: usize) -> Vec<u64>;           // 1.6.2.g (SPF sieve)

// Special primes
pub fn is_mersenne_prime(p: u32) -> bool;                     // 1.6.2.h
pub fn twin_primes_up_to(n: usize) -> Vec<(u64, u64)>;        // 1.6.2.i

// Prime counting
pub fn prime_count(n: u64) -> u64;                            // 1.6.2.j (approximation)
```

---

## Exercice 03: Modular Arithmetic
**Concepts couverts**: 1.6.3.a-j (10 concepts)
**Difficulté**: ⭐⭐⭐

```rust
pub const MOD: u64 = 1_000_000_007;

pub fn mod_add(a: u64, b: u64, m: u64) -> u64;                // 1.6.3.a
pub fn mod_sub(a: u64, b: u64, m: u64) -> u64;                // 1.6.3.b
pub fn mod_mul(a: u64, b: u64, m: u64) -> u64;                // 1.6.3.c

// Fast exponentiation
pub fn mod_pow(base: u64, exp: u64, m: u64) -> u64;           // 1.6.3.d

// Modular inverse
pub fn mod_inverse(a: u64, m: u64) -> Option<u64>;            // 1.6.3.e (extended GCD)
pub fn mod_inverse_fermat(a: u64, p: u64) -> u64;             // 1.6.3.f (p prime)

// Modular division
pub fn mod_div(a: u64, b: u64, m: u64) -> Option<u64>;        // 1.6.3.g

// Linear congruence
pub fn solve_congruence(a: u64, b: u64, m: u64) -> Option<u64>; // 1.6.3.h (ax ≡ b mod m)

// Properties
pub fn euler_totient(n: u64) -> u64;                          // 1.6.3.i (φ(n))
pub fn carmichael_lambda(n: u64) -> u64;                      // 1.6.3.j (λ(n))
```

---

## Exercice 04: Chinese Remainder Theorem
**Concepts couverts**: 1.6.4.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// CRT
pub fn crt(remainders: &[u64], moduli: &[u64]) -> Option<u64>; // 1.6.4.a

// CRT avec moduli non copremiers
pub fn crt_general(remainders: &[u64], moduli: &[u64]) -> Option<(u64, u64)>; // 1.6.4.b

// Garner's algorithm (pour grands nombres)
pub fn garner(remainders: &[u64], moduli: &[u64]) -> u64;     // 1.6.4.c

// Applications
pub fn solve_system_congruences(eqs: &[(u64, u64)]) -> Option<u64>; // 1.6.4.d

// CRT reconstruction
pub fn crt_reconstruct(values: &[u64], moduli: &[u64]) -> u64; // 1.6.4.e

// Multi-modular computation
pub fn multi_mod_add(a: &[u64], b: &[u64], moduli: &[u64]) -> Vec<u64>; // 1.6.4.f
pub fn multi_mod_mul(a: &[u64], b: &[u64], moduli: &[u64]) -> Vec<u64>; // 1.6.4.g

// Parallel CRT
pub fn parallel_crt(remainders: &[u64], moduli: &[u64]) -> u64; // 1.6.4.h
```

---

## Exercice 05: Primality Testing
**Concepts couverts**: 1.6.5.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Miller-Rabin
pub fn miller_rabin(n: u64, k: u32) -> bool;                  // 1.6.5.a
pub fn miller_rabin_deterministic(n: u64) -> bool;            // 1.6.5.b (for n < 2^64)

// Fermat's little theorem test
pub fn fermat_test(n: u64, k: u32) -> bool;                   // 1.6.5.c

// Carmichael numbers (Fermat liars)
pub fn is_carmichael(n: u64) -> bool;                         // 1.6.5.d

// Solovay-Strassen
pub fn solovay_strassen(n: u64, k: u32) -> bool;              // 1.6.5.e

// Jacobi symbol
pub fn jacobi(a: i64, n: u64) -> i32;                         // 1.6.5.f

// AKS (polynomial time, impractical)
pub fn aks_concept() -> String;                               // 1.6.5.g

// BPSW (practical deterministic)
pub fn bpsw(n: u64) -> bool;                                  // 1.6.5.h
```

---

# BLOC B: Théorie des Nombres Avancée (Exercices 06-10)

## Exercice 06: Factorization Algorithms
**Concepts couverts**: 1.6.6.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Trial division
pub fn trial_division(n: u64) -> Vec<u64>;                    // 1.6.6.a

// Pollard's rho
pub fn pollard_rho(n: u64) -> u64;                            // 1.6.6.b
pub fn pollard_rho_brent(n: u64) -> u64;                      // 1.6.6.c

// Fermat's factorization
pub fn fermat_factor(n: u64) -> (u64, u64);                   // 1.6.6.d

// Quadratic sieve (concept)
pub fn quadratic_sieve_concept() -> String;                   // 1.6.6.e

// Complete factorization
pub fn full_factorization(n: u64) -> Vec<(u64, u32)>;         // 1.6.6.f

// Smooth numbers
pub fn is_b_smooth(n: u64, b: u64) -> bool;                   // 1.6.6.g
pub fn smooth_numbers(limit: u64, b: u64) -> Vec<u64>;        // 1.6.6.h
```

---

## Exercice 07: Multiplicative Functions
**Concepts couverts**: 1.6.7.a-j (10 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Euler's totient
pub fn totient(n: u64) -> u64;                                // 1.6.7.a
pub fn totient_sieve(n: usize) -> Vec<u64>;                   // 1.6.7.b

// Möbius function
pub fn mobius(n: u64) -> i32;                                 // 1.6.7.c
pub fn mobius_sieve(n: usize) -> Vec<i32>;                    // 1.6.7.d

// Möbius inversion
pub fn mobius_inversion<F, G>(f: F, n: u64) -> G
where F: Fn(u64) -> u64, G: Fn(u64) -> i64;                   // 1.6.7.e

// Divisor functions
pub fn tau(n: u64) -> u64;                                    // 1.6.7.f (number of divisors)
pub fn sigma(n: u64, k: u32) -> u64;                          // 1.6.7.g (sum of k-th powers)

// Liouville function
pub fn liouville(n: u64) -> i32;                              // 1.6.7.h

// Radical of n
pub fn radical(n: u64) -> u64;                                // 1.6.7.i

// Multiplicative function evaluation
pub fn eval_multiplicative<F>(n: u64, f: F) -> u64
where F: Fn(u64, u32) -> u64;                                 // 1.6.7.j
```

---

## Exercice 08: Quadratic Residues
**Concepts couverts**: 1.6.8.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Legendre symbol
pub fn legendre(a: i64, p: u64) -> i32;                       // 1.6.8.a

// Euler's criterion
pub fn is_quadratic_residue(a: u64, p: u64) -> bool;          // 1.6.8.b

// Tonelli-Shanks (square root mod p)
pub fn sqrt_mod(a: u64, p: u64) -> Option<u64>;               // 1.6.8.c

// Cipolla's algorithm
pub fn cipolla(a: u64, p: u64) -> Option<u64>;                // 1.6.8.d

// All square roots mod n
pub fn all_sqrt_mod(a: u64, n: u64) -> Vec<u64>;              // 1.6.8.e

// Hensel's lemma (lifting)
pub fn hensel_lift(a: u64, p: u64, k: u32) -> u64;            // 1.6.8.f

// Quadratic residues enumeration
pub fn quadratic_residues(p: u64) -> Vec<u64>;                // 1.6.8.g

// Quadratic non-residue (generator finding)
pub fn find_non_residue(p: u64) -> u64;                       // 1.6.8.h
```

---

## Exercice 09: Discrete Logarithm
**Concepts couverts**: 1.6.9.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐⭐

```rust
// Baby-step Giant-step
pub fn baby_giant(g: u64, h: u64, p: u64) -> Option<u64>;     // 1.6.9.a

// Pohlig-Hellman
pub fn pohlig_hellman(g: u64, h: u64, p: u64) -> Option<u64>; // 1.6.9.b

// Index calculus (concept)
pub fn index_calculus_concept() -> String;                    // 1.6.9.c

// Primitive roots
pub fn is_primitive_root(g: u64, p: u64) -> bool;             // 1.6.9.d
pub fn find_primitive_root(p: u64) -> u64;                    // 1.6.9.e
pub fn all_primitive_roots(p: u64) -> Vec<u64>;               // 1.6.9.f

// Order of element
pub fn multiplicative_order(a: u64, n: u64) -> u64;           // 1.6.9.g

// Discrete log in any group
pub fn dlog_generic<G>(g: G, h: G, n: u64) -> Option<u64>
where G: Clone + Eq + std::hash::Hash;                        // 1.6.9.h
```

---

## Exercice 10: Continued Fractions
**Concepts couverts**: 1.6.10.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Continued fraction expansion
pub fn cf_expand(n: u64, d: u64) -> Vec<u64>;                 // 1.6.10.a (rational)
pub fn cf_sqrt(n: u64) -> (Vec<u64>, Vec<u64>);               // 1.6.10.b (sqrt periodic)

// Convergents
pub fn convergents(cf: &[u64]) -> Vec<(u64, u64)>;            // 1.6.10.c

// Best rational approximation
pub fn best_rational(x: f64, max_denom: u64) -> (u64, u64);   // 1.6.10.d

// Pell's equation
pub fn pell_fundamental(d: u64) -> (u64, u64);                // 1.6.10.e (x² - dy² = 1)
pub fn pell_all_solutions(d: u64, limit: u64) -> Vec<(u64, u64)>; // 1.6.10.f

// Generalized Pell
pub fn pell_negative(d: u64) -> Option<(u64, u64)>;           // 1.6.10.g (x² - dy² = -1)

// Chakravala method
pub fn chakravala(d: u64) -> (u64, u64);                      // 1.6.10.h
```

---

# BLOC C: Combinatorique (Exercices 11-14)

## Exercice 11: Binomial Coefficients
**Concepts couverts**: 1.6.11.a-j (10 concepts)
**Difficulté**: ⭐⭐

```rust
// Pascal's triangle
pub fn binomial(n: u64, k: u64) -> u64;                       // 1.6.11.a
pub fn binomial_mod(n: u64, k: u64, m: u64) -> u64;           // 1.6.11.b

// Lucas' theorem
pub fn lucas(n: u64, k: u64, p: u64) -> u64;                  // 1.6.11.c

// Extended Lucas (prime power moduli)
pub fn extended_lucas(n: u64, k: u64, p: u64, e: u32) -> u64; // 1.6.11.d

// Precomputation
pub fn precompute_binomial(n: usize, m: u64) -> Vec<Vec<u64>>; // 1.6.11.e
pub fn precompute_factorials(n: usize, m: u64) -> (Vec<u64>, Vec<u64>); // 1.6.11.f

// Catalan numbers
pub fn catalan(n: u64) -> u64;                                // 1.6.11.g

// Stirling numbers
pub fn stirling_first(n: u64, k: u64) -> i64;                 // 1.6.11.h
pub fn stirling_second(n: u64, k: u64) -> u64;                // 1.6.11.i

// Bell numbers
pub fn bell(n: u64) -> u64;                                   // 1.6.11.j
```

---

## Exercice 12: Permutations & Cycles
**Concepts couverts**: 1.6.12.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Permutation representation
pub struct Permutation {
    perm: Vec<usize>,
}

impl Permutation {
    pub fn from_array(arr: &[usize]) -> Self;                 // 1.6.12.a
    pub fn compose(&self, other: &Self) -> Self;              // 1.6.12.b
    pub fn inverse(&self) -> Self;                            // 1.6.12.c
    pub fn order(&self) -> usize;                             // 1.6.12.d

    // Cycle decomposition
    pub fn cycles(&self) -> Vec<Vec<usize>>;                  // 1.6.12.e
    pub fn cycle_type(&self) -> Vec<usize>;                   // 1.6.12.f

    // Sign
    pub fn sign(&self) -> i32;                                // 1.6.12.g
    pub fn is_even(&self) -> bool;

    // Derangements
    pub fn derangement_count(n: usize) -> u64;                // 1.6.12.h
}
```

---

## Exercice 13: Partition Numbers
**Concepts couverts**: 1.6.13.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Integer partitions
pub fn partition_count(n: u64) -> u64;                        // 1.6.13.a
pub fn partition_count_dp(n: usize) -> Vec<u64>;              // 1.6.13.b

// Partitions into distinct parts
pub fn partition_distinct(n: u64) -> u64;                     // 1.6.13.c

// Partitions into k parts
pub fn partition_k_parts(n: u64, k: u64) -> u64;              // 1.6.13.d

// Partition enumeration
pub fn enumerate_partitions(n: u64) -> Vec<Vec<u64>>;         // 1.6.13.e

// Euler's pentagonal theorem
pub fn pentagonal_partition(n: usize) -> Vec<u64>;            // 1.6.13.f

// Conjugate partition
pub fn conjugate_partition(p: &[u64]) -> Vec<u64>;            // 1.6.13.g

// Young tableaux count
pub fn standard_young_tableaux(shape: &[usize]) -> u64;       // 1.6.13.h
```

---

## Exercice 14: Generating Functions
**Concepts couverts**: 1.6.14.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Polynomial representation
pub struct Polynomial {
    coeffs: Vec<i64>,
}

impl Polynomial {
    pub fn add(&self, other: &Self) -> Self;                  // 1.6.14.a
    pub fn mul(&self, other: &Self) -> Self;                  // 1.6.14.b
    pub fn mul_mod(&self, other: &Self, m: u64) -> Self;

    // Coefficient extraction
    pub fn coefficient(&self, n: usize) -> i64;               // 1.6.14.c

    // Generating function operations
    pub fn derivative(&self) -> Self;                         // 1.6.14.d
    pub fn integral(&self) -> Self;                           // 1.6.14.e

    // Common generating functions
    pub fn geometric_gf(n: usize) -> Self;                    // 1.6.14.f
    pub fn exponential_gf(n: usize) -> Self;                  // 1.6.14.g

    // Composition
    pub fn compose(&self, other: &Self) -> Self;              // 1.6.14.h
}
```

---

# BLOC D: Théorie des Groupes & Crypto (Exercices 15-18)

## Exercice 15: Group Theory Basics
**Concepts couverts**: 1.6.15.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Finite group operations
pub trait Group {
    fn identity() -> Self;                                    // 1.6.15.a
    fn op(&self, other: &Self) -> Self;                       // 1.6.15.b
    fn inverse(&self) -> Self;                                // 1.6.15.c
}

// Z/nZ
pub struct Zn { val: u64, n: u64 }

impl Group for Zn { ... }                                     // 1.6.15.d

// (Z/nZ)*
pub struct ZnStar { val: u64, n: u64 }

impl ZnStar {
    pub fn is_valid(&self) -> bool;                           // 1.6.15.e (gcd = 1)
    pub fn order(&self) -> u64;                               // 1.6.15.f
}

// Subgroup
pub fn is_subgroup<G: Group>(elements: &[G]) -> bool;         // 1.6.15.g

// Lagrange's theorem
pub fn subgroup_order_divides<G: Group>(subgroup_size: usize, group_size: usize) -> bool; // 1.6.15.h
```

---

## Exercice 16: Elliptic Curves Basics
**Concepts couverts**: 1.6.16.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐⭐

```rust
// Elliptic curve y² = x³ + ax + b mod p
pub struct EllipticCurve {
    a: u64,
    b: u64,
    p: u64,
}

pub struct Point {
    x: Option<u64>,
    y: Option<u64>,
}

impl EllipticCurve {
    pub fn new(a: u64, b: u64, p: u64) -> Self;               // 1.6.16.a
    pub fn is_valid(&self) -> bool;                           // 1.6.16.b (4a³ + 27b² ≠ 0)

    pub fn is_on_curve(&self, point: &Point) -> bool;         // 1.6.16.c
    pub fn add(&self, p1: &Point, p2: &Point) -> Point;       // 1.6.16.d
    pub fn double(&self, p: &Point) -> Point;                 // 1.6.16.e
    pub fn scalar_mul(&self, p: &Point, k: u64) -> Point;     // 1.6.16.f

    pub fn order(&self) -> u64;                               // 1.6.16.g (Schoof's concept)
    pub fn enumerate_points(&self) -> Vec<Point>;             // 1.6.16.h
}
```

---

## Exercice 17: RSA Implementation
**Concepts couverts**: 1.6.17.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
pub struct RSAKey {
    n: u128,     // modulus
    e: u64,      // public exponent
    d: u128,     // private exponent
}

impl RSAKey {
    pub fn generate(bits: u32) -> Self;                       // 1.6.17.a
    pub fn encrypt(&self, m: u128) -> u128;                   // 1.6.17.b
    pub fn decrypt(&self, c: u128) -> u128;                   // 1.6.17.c
    pub fn sign(&self, m: u128) -> u128;                      // 1.6.17.d
    pub fn verify(&self, m: u128, sig: u128) -> bool;         // 1.6.17.e
}

// Key generation steps
pub fn generate_safe_prime(bits: u32) -> u128;                // 1.6.17.f
pub fn compute_d(e: u64, phi: u128) -> u128;                  // 1.6.17.g
pub fn textbook_rsa_weakness() -> String;                     // 1.6.17.h
```

---

## Exercice 18: Diffie-Hellman
**Concepts couverts**: 1.6.18.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
pub struct DHParams {
    p: u64,      // prime modulus
    g: u64,      // generator
}

impl DHParams {
    pub fn generate(bits: u32) -> Self;                       // 1.6.18.a
    pub fn is_safe_prime(&self) -> bool;                      // 1.6.18.b

    pub fn private_key(&self) -> u64;                         // 1.6.18.c
    pub fn public_key(&self, private: u64) -> u64;            // 1.6.18.d
    pub fn shared_secret(&self, my_private: u64, other_public: u64) -> u64; // 1.6.18.e
}

// ECDH
pub struct ECDH {
    curve: EllipticCurve,
    g: Point,
}

impl ECDH {
    pub fn new() -> Self;                                     // 1.6.18.f (secp256k1 example)
    pub fn key_exchange(&self, private: u64, other_public: &Point) -> Point; // 1.6.18.g
}

// Man-in-the-middle explanation
pub fn mitm_attack_concept() -> String;                       // 1.6.18.h
```

---

# BLOC E: Projet Final (Exercices 19-20)

## Exercice 19: Number Theory Library
**Concepts couverts**: All library functions
**Difficulté**: ⭐⭐⭐⭐

```rust
pub mod number_theory {
    pub mod primes {
        pub fn is_prime(n: u64) -> bool;
        pub fn sieve(n: usize) -> Vec<bool>;
        pub fn factorize(n: u64) -> Vec<(u64, u32)>;
        pub fn miller_rabin(n: u64) -> bool;
    }

    pub mod modular {
        pub fn mod_pow(base: u64, exp: u64, m: u64) -> u64;
        pub fn mod_inverse(a: u64, m: u64) -> Option<u64>;
        pub fn crt(remainders: &[u64], moduli: &[u64]) -> Option<u64>;
    }

    pub mod multiplicative {
        pub fn totient(n: u64) -> u64;
        pub fn mobius(n: u64) -> i32;
        pub fn divisors(n: u64) -> Vec<u64>;
    }

    pub mod combinatorics {
        pub fn binomial(n: u64, k: u64) -> u64;
        pub fn catalan(n: u64) -> u64;
        pub fn partition_count(n: u64) -> u64;
    }
}
```

---

## Exercice 20: Crypto Toolkit (Projet)
**Concepts couverts**: 1.6.a-n (14 concepts projet)
**Difficulté**: ⭐⭐⭐⭐⭐

```rust
pub struct CryptoToolkit {
    // Components
    rsa: Option<RSAKey>,
    dh: Option<DHParams>,
    ec: Option<EllipticCurve>,
}

impl CryptoToolkit {
    // Prime generation and testing
    pub fn generate_prime(&self, bits: u32) -> u64;           // 1.6.a
    pub fn is_prime(&self, n: u64) -> bool;                   // 1.6.b

    // Factorization
    pub fn factor(&self, n: u64) -> Vec<(u64, u32)>;          // 1.6.c

    // Modular arithmetic
    pub fn mod_operations(&self) -> ModArithmeticAPI;         // 1.6.d

    // CRT
    pub fn chinese_remainder(&self, eqs: &[(u64, u64)]) -> Option<u64>; // 1.6.e

    // RSA
    pub fn rsa_keygen(&mut self, bits: u32);                  // 1.6.f
    pub fn rsa_encrypt(&self, m: &[u8]) -> Vec<u8>;           // 1.6.g
    pub fn rsa_decrypt(&self, c: &[u8]) -> Vec<u8>;

    // Diffie-Hellman
    pub fn dh_exchange(&self) -> DHSession;                   // 1.6.h

    // Elliptic curves
    pub fn ec_operations(&self) -> ECOperationsAPI;           // 1.6.i

    // CLI
    pub fn cli_handler(args: &[String]) -> Result<String, Error>; // 1.6.j

    // Benchmarks
    pub fn benchmark(&self) -> BenchmarkResults;              // 1.6.k

    // Bonus
    pub fn polynomial_arithmetic(&self) -> PolynomialAPI;     // 1.6.l
    pub fn ntt(&self, coeffs: &[u64]) -> Vec<u64>;            // 1.6.m (Number Theoretic Transform)
    pub fn lattice_basics(&self) -> LatticeAPI;               // 1.6.n
}
```

---

# RÉCAPITULATIF

| Bloc | Exercices | Concepts | Description |
|------|-----------|----------|-------------|
| A | 01-05 | 44 | Arithmétique de Base |
| B | 06-10 | 42 | Théorie Avancée |
| C | 11-14 | 34 | Combinatorique |
| D | 15-18 | 32 | Groupes & Crypto |
| E | 19-20 | 50 | Projet & Library |
| **TOTAL** | **20** | **202** | **Module 1.6 complet** |

---
