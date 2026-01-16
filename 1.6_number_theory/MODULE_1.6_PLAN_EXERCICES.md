# MODULE 1.6 — NUMBER THEORY & MATHEMATICS
## Plan d'Exercices Couvrant 43 Concepts

---

## PROJET 1: `prime_laboratory` — Nombres Premiers et Factorisation (9 concepts)

### Concepts couverts:
- 1.6.1.a-e (Nombres Premiers: 5)
- 1.6.2.a-d (Factorisation: 4)

### Structure:
```
prime_laboratory/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── primality/
│   │   ├── mod.rs
│   │   ├── trial_division.rs  # Test basique √n
│   │   ├── sieve.rs           # Eratosthène classique
│   │   ├── linear_sieve.rs    # O(n) avec SPF
│   │   └── segmented.rs       # Pour grandes plages
│   ├── factorization/
│   │   ├── mod.rs
│   │   ├── trial.rs           # Factorisation basique
│   │   ├── spf_factorize.rs   # Via SPF précalculé
│   │   └── divisors.rs        # Count, sum divisors
│   └── prime_utils.rs         # Fonctions utilitaires
├── tests/
│   ├── primality_tests.rs
│   ├── factorization_tests.rs
│   └── stress_tests.rs
└── benches/
    └── sieve_comparison.rs    # Comparer les cribles
```

### Exercices:
1. `is_prime_trial(n: u64) -> bool` — Test jusqu'à √n
2. `sieve_eratosthenes(limit: usize) -> Vec<bool>` — Crible classique
3. `sieve_primes(limit: usize) -> Vec<u64>` — Liste des premiers
4. `linear_sieve(limit: usize) -> (Vec<bool>, Vec<usize>)` — Avec SPF
5. `segmented_sieve(lo: u64, hi: u64) -> Vec<u64>` — Plage [lo, hi]
6. `prime_factors(n: u64) -> Vec<(u64, u32)>` — Factorisation avec exposants
7. `fast_factorize(n: u64, spf: &[usize]) -> Vec<(u64, u32)>` — Via SPF
8. `count_divisors(n: u64) -> u64` — Nombre de diviseurs
9. `sum_divisors(n: u64) -> u64` — Somme des diviseurs
10. `divisors_list(n: u64) -> Vec<u64>` — Liste ordonnée

### Tests moulinette:
- Crible jusqu'à 10^8
- Factorisation de nombres jusqu'à 10^12
- Segmented sieve pour plages de 10^6

---

## PROJET 2: `modular_arithmetic` — Arithmétique Modulaire Complète (13 concepts)

### Concepts couverts:
- 1.6.3.a-d (GCD/LCM: 4)
- 1.6.4.a-b (Extended Euclid: 2)
- 1.6.5.a-c (Arithmétique Mod: 3)
- 1.6.6.a-b (Exponentiation: 2)
- 1.6.7.a-c (Inverse: 3)

### Structure:
```
modular_arithmetic/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── gcd/
│   │   ├── mod.rs
│   │   ├── euclidean.rs       # GCD classique
│   │   ├── extended.rs        # Extended GCD
│   │   └── lcm.rs             # LCM
│   ├── modint/
│   │   ├── mod.rs
│   │   ├── modint.rs          # Wrapper type
│   │   ├── ops.rs             # +, -, *, /
│   │   └── const_mod.rs       # Compile-time modulus
│   ├── exponentiation/
│   │   ├── mod.rs
│   │   ├── binary_exp.rs      # a^n mod m
│   │   └── matrix_exp.rs      # M^n mod m
│   └── inverse/
│       ├── mod.rs
│       ├── ext_gcd.rs         # Via Extended GCD
│       ├── fermat.rs          # Via Fermat
│       └── precompute.rs      # Inverse 1..n
├── tests/
│   ├── gcd_tests.rs
│   ├── modint_tests.rs
│   └── inverse_tests.rs
└── examples/
    └── fibonacci_matrix.rs    # Fib via matrix exp
```

### Exercices:
1. `gcd(a: u64, b: u64) -> u64` — Algorithme d'Euclide
2. `gcd_recursive(a: u64, b: u64) -> u64` — Version récursive
3. `binary_gcd(a: u64, b: u64) -> u64` — Sans division (Stein)
4. `lcm(a: u64, b: u64) -> u64` — Via GCD
5. `extended_gcd(a: i64, b: i64) -> (i64, i64, i64)` — (gcd, x, y)
6. `solve_linear_diophantine(a, b, c) -> Option<(i64, i64)>` — ax + by = c
7. `ModInt<M>` — Struct avec Add, Sub, Mul, Div
8. `mod_pow(base: u64, exp: u64, m: u64) -> u64` — Exponentiation rapide
9. `matrix_pow<const N: usize>(m: [[u64; N]; N], exp: u64, mod: u64)` — Matrix exp
10. `mod_inverse_ext(a: u64, m: u64) -> Option<u64>` — Via extended GCD
11. `mod_inverse_fermat(a: u64, p: u64) -> u64` — Via Fermat (p premier)
12. `precompute_inverses(n: usize, m: u64) -> Vec<u64>` — Tous les inverses

### Tests moulinette:
- GCD pour nombres 64-bit
- Matrix exp pour Fibonacci n=10^18
- ModInt: overflow protection

---

## PROJET 3: `number_theorems` — Théorèmes Fondamentaux (8 concepts)

### Concepts couverts:
- 1.6.8.a-c (Fermat/Euler: 3)
- 1.6.9.a-c (CRT: 3)
- 1.6.10.a-b (Primalité Avancée: 2)

### Structure:
```
number_theorems/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── totient/
│   │   ├── mod.rs
│   │   ├── single.rs          # φ(n) individuel
│   │   ├── sieve.rs           # φ(1..n)
│   │   └── properties.rs      # Propriétés multiplicatives
│   ├── crt/
│   │   ├── mod.rs
│   │   ├── two_equations.rs   # Cas simple
│   │   ├── general.rs         # n équations
│   │   └── extended.rs        # Non-coprimes
│   └── primality/
│       ├── mod.rs
│       ├── fermat.rs          # Test de Fermat
│       ├── miller_rabin.rs    # Probabiliste
│       └── deterministic.rs   # Déterministe 64-bit
├── tests/
│   ├── totient_tests.rs
│   ├── crt_tests.rs
│   └── primality_tests.rs
└── examples/
    └── rsa_toy.rs             # RSA simplifié
```

### Exercices:
1. `euler_totient(n: u64) -> u64` — Calcul direct
2. `totient_sieve(limit: usize) -> Vec<u64>` — φ(1..n)
3. `sum_totients(n: u64) -> u64` — Σφ(i) pour i=1..n
4. `crt_two(r1: i64, m1: i64, r2: i64, m2: i64) -> Option<(i64, i64)>` — Deux équations
5. `crt_general(remainders: &[i64], moduli: &[i64]) -> Option<(i64, i64)>` — n équations
6. `crt_extended(remainders: &[i64], moduli: &[i64]) -> Option<(i64, i64)>` — Non-coprimes
7. `fermat_test(n: u64, witnesses: &[u64]) -> bool` — Test de Fermat
8. `miller_rabin(n: u64, k: u32) -> bool` — k rounds probabiliste
9. `is_prime_deterministic(n: u64) -> bool` — Déterministe pour 64-bit

### Tests moulinette:
- Totient pour n jusqu'à 10^12
- CRT: systèmes de 10 équations
- Miller-Rabin: aucun faux négatif

---

## PROJET 4: `advanced_factorization` — Factorisation Avancée (2 concepts)

### Concepts couverts:
- 1.6.11.a-b (Pollard's Rho, Brent: 2)

### Structure:
```
advanced_factorization/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── pollard_rho.rs         # Algorithme original
│   ├── brent.rs               # Amélioration de Brent
│   ├── complete.rs            # Factorisation complète
│   └── utils.rs               # Montgomery mult, etc.
├── tests/
│   └── factorization_tests.rs
└── benches/
    └── factor_large.rs        # Benchmark grands nombres
```

### Exercices:
1. `pollard_rho(n: u64) -> u64` — Trouver un facteur
2. `pollard_rho_brent(n: u64) -> u64` — Version Brent
3. `complete_factorization(n: u64) -> Vec<(u64, u32)>` — Factorisation complète
4. `factor_large(n: u128) -> Vec<(u128, u32)>` — Pour très grands nombres

### Tests moulinette:
- Factoriser produits de deux premiers de 30 bits
- Temps < 100ms pour 64-bit

---

## PROJET 5: `combinatorics` — Combinatoire (4 concepts)

### Concepts couverts:
- 1.6.12.a-d (Combinatoire: 4)

### Structure:
```
combinatorics/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── binomial/
│   │   ├── mod.rs
│   │   ├── basic.rs           # C(n,k) direct
│   │   ├── pascal.rs          # Triangle de Pascal
│   │   ├── mod_prime.rs       # C(n,k) mod p
│   │   └── lucas.rs           # Théorème de Lucas
│   ├── sequences/
│   │   ├── mod.rs
│   │   ├── catalan.rs         # Nombres de Catalan
│   │   ├── stirling.rs        # Nombres de Stirling
│   │   └── derangements.rs    # Dérangements
│   └── counting/
│       ├── mod.rs
│       └── inclusion_exclusion.rs  # PIE
├── tests/
│   ├── binomial_tests.rs
│   └── sequences_tests.rs
└── examples/
    └── counting_problems.rs
```

### Exercices:
1. `binomial(n: u64, k: u64) -> u64` — C(n,k) overflow-safe
2. `binomial_mod(n: u64, k: u64, m: u64) -> u64` — C(n,k) mod m
3. `pascal_row(n: usize) -> Vec<u64>` — n-ième ligne du triangle
4. `lucas(n: u64, k: u64, p: u64) -> u64` — Théorème de Lucas
5. `catalan(n: u32) -> u64` — n-ième nombre de Catalan
6. `catalan_mod(n: u32, m: u64) -> u64` — Catalan mod m
7. `stirling_first(n: u32, k: u32) -> i64` — Stirling première espèce
8. `stirling_second(n: u32, k: u32) -> u64` — Stirling deuxième espèce
9. `derangements(n: u32) -> u64` — Nombre de dérangements
10. `inclusion_exclusion<F>(n: usize, count: F) -> u64` — PIE générique

### Tests moulinette:
- Binomial pour n jusqu'à 10^6
- Lucas pour n jusqu'à 10^18
- Catalan jusqu'à C(1000)

---

## PROJET 6: `fft_ntt` — Transformées Rapides (3 concepts)

### Concepts couverts:
- 1.6.13.a-c (FFT/NTT: 3)

### Structure:
```
fft_ntt/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── complex.rs             # Nombres complexes
│   ├── fft.rs                 # FFT classique
│   ├── ntt.rs                 # NTT modulaire
│   ├── convolution.rs         # Multiplication polynômes
│   └── applications/
│       ├── mod.rs
│       ├── big_multiply.rs    # Grands entiers
│       └── string_matching.rs # Pattern matching
├── tests/
│   ├── fft_tests.rs
│   └── ntt_tests.rs
└── benches/
    └── multiply_bench.rs
```

### Exercices:
1. `Complex { re: f64, im: f64 }` — Type complexe
2. `fft(a: &mut [Complex])` — FFT in-place
3. `ifft(a: &mut [Complex])` — FFT inverse
4. `multiply_polynomials_fft(a: &[i64], b: &[i64]) -> Vec<i64>` — Via FFT
5. `ntt(a: &mut [u64], mod: u64)` — NTT modulaire
6. `intt(a: &mut [u64], mod: u64)` — NTT inverse
7. `multiply_polynomials_ntt(a: &[u64], b: &[u64], mod: u64) -> Vec<u64>` — Via NTT
8. `multiply_big_integers(a: &str, b: &str) -> String` — Karatsuba/FFT
9. `convolution(a: &[i64], b: &[i64]) -> Vec<i64>` — Convolution générique

### Tests moulinette:
- Polynômes de degré 10^6
- NTT pour mod 998244353
- Big integers de 10^5 chiffres

---

## PROJET 7: `game_theory` — Théorie des Jeux (3 concepts)

### Concepts couverts:
- 1.6.14.a-c (Game Theory: 3)

### Structure:
```
game_theory/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── nim/
│   │   ├── mod.rs
│   │   ├── classic.rs         # Nim classique
│   │   └── variations.rs      # Variantes
│   ├── grundy/
│   │   ├── mod.rs
│   │   ├── calculation.rs     # Calcul Grundy
│   │   └── composition.rs     # Jeux composés
│   └── games/
│       ├── mod.rs
│       ├── subtraction.rs     # Jeu de soustraction
│       └── graph_games.rs     # Jeux sur graphes
├── tests/
│   └── game_tests.rs
└── examples/
    └── interactive_nim.rs
```

### Exercices:
1. `nim_winner(piles: &[u32]) -> Player` — Qui gagne au Nim?
2. `nim_optimal_move(piles: &[u32]) -> Option<(usize, u32)>` — Coup optimal
3. `mex(set: &HashSet<u32>) -> u32` — Minimum excludant
4. `grundy_number(state: S, moves: F) -> u32` — Calcul Sprague-Grundy
5. `combined_grundy(games: &[u32]) -> u32` — XOR des Grundy
6. `subtraction_game(n: u32, S: &[u32]) -> u32` — Grundy pour jeu S
7. `graph_game_grundy(adj: &[Vec<usize>]) -> Vec<u32>` — Grundy sur DAG

### Tests moulinette:
- Nim avec 100 piles
- Jeux de soustraction avec S jusqu'à 100 éléments
- Jeux sur graphes de 10^4 nœuds

---

## RÉCAPITULATIF

| Projet | Concepts | Sections couvertes |
|--------|----------|-------------------|
| prime_laboratory | 9 | 1.6.1, 1.6.2 |
| modular_arithmetic | 13 | 1.6.3, 1.6.4, 1.6.5, 1.6.6, 1.6.7 |
| number_theorems | 8 | 1.6.8, 1.6.9, 1.6.10 |
| advanced_factorization | 2 | 1.6.11 |
| combinatorics | 4 | 1.6.12 |
| fft_ntt | 3 | 1.6.13 |
| game_theory | 3 | 1.6.14 |
| **TOTAL** | **43** | **100%** |

---

## CRITÈRES DE QUALITÉ (Score visé: 97/100)

### Originalité (25/25)
- ModInt wrapper avec const generics
- FFT et NTT côte à côte pour comparaison
- Applications pratiques: RSA toy, big integers
- Interactive game player

### Couverture Concepts (25/25)
- 43/43 concepts couverts (100%)
- Théorèmes fondamentaux avec preuves implicites dans le code
- Du trial division à Pollard's Rho

### Testabilité Moulinette (25/25)
- Vérification via inverse: a * inv(a) ≡ 1
- Tests de primalité: aucun faux positif
- FFT: vérifier multiplication via naive O(n²)

### Pédagogie (22/25)
- Progression: simple → avancé
- Comparaison algorithmes (Eratosthène vs linéaire)
- Applications concrètes (cryptographie)
- Benchmarks pour visualiser complexité

---

## ORDRE D'IMPLÉMENTATION RECOMMANDÉ

1. **prime_laboratory** — Fondation: cribles et factorisation
2. **modular_arithmetic** — Outils essentiels pour tout le reste
3. **number_theorems** — Théorèmes clés (Fermat, Euler, CRT)
4. **advanced_factorization** — Pour très grands nombres
5. **combinatorics** — Comptage et formules
6. **fft_ntt** — Transformées rapides
7. **game_theory** — Applications ludiques
