# Exercice [1.6.1-a] : sieve_matrix_awakening

**Module :**
1.6.1 — Prime Numbers & Sieves

**Concept :**
a — Sieve of Eratosthenes, Linear Sieve, Segmented Sieve, SPF Factorization, Mobius Function, nth Prime

**Difficulte :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
1 — Concept isole (Cribles de nombres premiers)

**Langage :**
Rust Edition 2024 + C (C17)

**Prerequis :**
- Boucles et tableaux
- Arithmetique de base (division, modulo)
- Notion de complexite algorithmique O(n)
- Pointeurs et allocation memoire (C)

**Domaines :**
MD, Tri, Mem

**Duree estimee :**
90 min

**XP Base :**
150

**Complexite :**
T5 O(n log log n) × S4 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers a rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `primes.c`, `primes.h`

**Fonctions autorisees :**
- Rust : `Vec::new()`, `vec![]`, `push()`, operations arithmetiques standard
- C : `malloc()`, `calloc()`, `free()`, `realloc()`, operations arithmetiques standard

**Fonctions interdites :**
- Rust : Aucune bibliotheque de theorie des nombres (`primal`, `num-prime`, etc.)
- C : `sqrt()` de `<math.h>` (tu dois implementer ta propre racine carree entiere)

---

### 1.2 Consigne

#### 🎬 Section 2.4.1 : CONTEXTE FUN — "The Matrix: Awakening to Primes"

**"Free your mind, Neo. The Matrix is built on primes."**

Tu es Neo. Tu viens d'avaler la pilule rouge et tu decouvres une verite fondamentale : la Matrix n'est pas construite sur du code binaire ordinaire. Elle est construite sur les **nombres premiers** — les atomes indivisibles des mathematiques.

Morpheus t'explique :
> *"Les composites sont des illusions, Neo. Ils se decomposent. Seuls les primes sont reels. Ils ne peuvent pas etre divises. Ils sont... The One."*

L'Agent Smith (le nombre 12) peut etre decompose : `12 = 2 × 2 × 3`. Il n'est pas reel.
Neo (le nombre 13) ne peut pas etre decompose. Il EST reel. Il est premier.

**Ta mission : Construire le Crible de la Realite**

Pour voir la Matrix comme Neo, tu dois implementer **6 fonctions de crible** qui revelent les nombres premiers caches dans la sequence des entiers :

1. **`sieve_basic`** — Le premier eveil (Crible d'Eratosthene classique)
2. **`linear_sieve`** — La pilule rouge (Crible lineaire O(n) avec SPF)
3. **`segmented_sieve`** — Naviguer entre les secteurs de la Matrix
4. **`factorize_spf`** — Decomposer les Agents en leurs composants premiers
5. **`mobius_sieve`** — Detecter les glitches (fonction de Mobius)
6. **`nth_prime`** — Localiser The One (trouver le n-ieme premier)

---

#### Section 2.4.2 : ENONCE ACADEMIQUE

**Objectif :** Implementer une suite complete d'algorithmes de crible pour les nombres premiers.

**Contexte mathematique :**
Les nombres premiers sont les entiers naturels superieurs a 1 qui ne sont divisibles que par 1 et eux-memes. Le Crible d'Eratosthene est un algorithme antique (circa 240 BCE) pour trouver tous les nombres premiers jusqu'a une limite donnee.

**Fonctions a implementer :**

---

**Ta mission :**

Ecrire 6 fonctions qui implementent differents aspects des cribles de nombres premiers.

---

**Entree/Sortie par fonction :**

| Fonction | Entree | Sortie |
|----------|--------|--------|
| `sieve_basic(n)` | `n: usize` limite superieure | `Vec<bool>` ou `is_prime[i]` = true si i est premier |
| `linear_sieve(n)` | `n: usize` limite superieure | `(Vec<usize>, Vec<usize>)` = (liste des primes, tableau SPF) |
| `segmented_sieve(lo, hi)` | `lo, hi: u64` bornes de l'intervalle | `Vec<u64>` liste des primes dans [lo, hi] |
| `factorize_spf(n, spf)` | `n: usize`, `spf: &[usize]` | `Vec<(usize, usize)>` = [(prime, exposant), ...] |
| `mobius_sieve(n)` | `n: usize` limite superieure | `Vec<i8>` ou mu[i] = fonction de Mobius de i |
| `nth_prime(n)` | `n: usize` rang du premier | `u64` le n-ieme nombre premier |

---

**Contraintes :**

- `sieve_basic` : Complexite O(n log log n) temps, O(n) espace
- `linear_sieve` : Complexite **exactement** O(n) temps, O(n) espace
- `segmented_sieve` : Doit fonctionner pour hi jusqu'a 10^12 avec O(sqrt(hi)) memoire
- `factorize_spf` : Complexite O(log n) en utilisant le tableau SPF precalcule
- `mobius_sieve` : mu(n) = 0 si n a un facteur carre, (-1)^k si n = p1*p2*...*pk distincts
- `nth_prime` : Doit etre correct pour n jusqu'a 10^6

**Edge cases obligatoires :**
- n = 0 : retourner tableau vide ou equivalent logique
- n = 1 : 1 n'est PAS premier
- n = 2 : 2 EST premier (le seul premier pair)
- lo > hi : retourner tableau vide
- lo = 0 : traiter comme lo = 2

---

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `sieve_basic(10)` | `[F,F,T,T,F,T,F,T,F,F,F]` | 2,3,5,7 sont premiers |
| `linear_sieve(10).0` | `[2,3,5,7]` | Liste des primes jusqu'a 10 |
| `linear_sieve(10).1[6]` | `2` | SPF de 6 est 2 (6=2*3) |
| `segmented_sieve(100,110)` | `[101,103,107,109]` | Primes entre 100 et 110 |
| `factorize_spf(60, spf)` | `[(2,2),(3,1),(5,1)]` | 60 = 2^2 * 3^1 * 5^1 |
| `mobius_sieve(6)[6]` | `1` | 6=2*3, 2 facteurs distincts, (-1)^2=1 |
| `mobius_sieve(4)[4]` | `0` | 4=2^2, facteur carre, mu=0 |
| `nth_prime(10)` | `29` | Le 10eme premier est 29 |

---

### 1.3 Prototype

#### Rust (Edition 2024)

```rust
pub mod primes {
    /// Crible d'Eratosthene basique - O(n log log n)
    /// Retourne un vecteur ou is_prime[i] = true si i est premier
    pub fn sieve_basic(n: usize) -> Vec<bool>;

    /// Crible lineaire - O(n) exact
    /// Retourne (liste des primes, tableau SPF)
    /// SPF[i] = plus petit facteur premier de i
    pub fn linear_sieve(n: usize) -> (Vec<usize>, Vec<usize>);

    /// Crible segmente pour grands intervalles [lo, hi]
    /// Fonctionne pour hi jusqu'a 10^12
    pub fn segmented_sieve(lo: u64, hi: u64) -> Vec<u64>;

    /// Factorisation en utilisant le tableau SPF precalcule
    /// Complexite O(log n)
    pub fn factorize_spf(n: usize, spf: &[usize]) -> Vec<(usize, usize)>;

    /// Calcule la fonction de Mobius pour tous les entiers 0..=n
    /// mu(n) = 0 si n a un facteur carre
    /// mu(n) = (-1)^k si n = p1*p2*...*pk (k facteurs premiers distincts)
    pub fn mobius_sieve(n: usize) -> Vec<i8>;

    /// Trouve le n-ieme nombre premier (1-indexe)
    /// nth_prime(1) = 2, nth_prime(10) = 29
    pub fn nth_prime(n: usize) -> u64;
}
```

#### C (C17)

```c
#ifndef PRIMES_H
# define PRIMES_H

# include <stddef.h>
# include <stdint.h>
# include <stdbool.h>

// Structure pour retourner les resultats du crible lineaire
typedef struct s_linear_sieve_result {
    uint64_t    *primes;      // Liste des nombres premiers
    size_t      primes_count; // Nombre de premiers trouves
    size_t      *spf;         // Smallest Prime Factor array
    size_t      spf_size;     // Taille du tableau SPF
} t_linear_sieve;

// Structure pour la factorisation
typedef struct s_factor {
    size_t  prime;
    size_t  exponent;
} t_factor;

typedef struct s_factorization {
    t_factor    *factors;
    size_t      count;
} t_factorization;

// Prototypes
bool            *sieve_basic(size_t n, size_t *out_size);
t_linear_sieve  *linear_sieve(size_t n);
uint64_t        *segmented_sieve(uint64_t lo, uint64_t hi, size_t *out_count);
t_factorization *factorize_spf(size_t n, size_t *spf, size_t spf_size);
int8_t          *mobius_sieve(size_t n, size_t *out_size);
uint64_t        nth_prime(size_t n);

// Fonctions de liberation memoire
void            free_linear_sieve(t_linear_sieve *result);
void            free_factorization(t_factorization *result);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Histoire des Cribles

Le Crible d'Eratosthene porte le nom d'Eratosthene de Cyrene (276-194 BCE), mathematicien grec qui fut aussi le premier a calculer la circonference de la Terre avec une precision remarquable.

Son algorithme est si elegant qu'il est reste essentiellement inchange pendant plus de 2200 ans. C'est l'un des plus anciens algorithmes encore utilises en informatique moderne.

### 2.2 Pourquoi les Nombres Premiers ?

Les nombres premiers sont appeles les "atomes" des mathematiques car tout entier positif peut etre decompose de maniere unique en produit de nombres premiers (Theoreme Fondamental de l'Arithmetique).

**Analogie Matrix :** Chaque "Agent" (nombre composite) est en realite une combinaison d'entites premieres. Quand Neo (factorize_spf) analyse un Agent, il voit sa vraie nature.

### 2.3 La Distribution des Primes

Le Theoreme des Nombres Premiers (1896) dit que le nombre de premiers <= n est approximativement n / ln(n).

```
n = 100      : 25 primes    (theorique: ~21.7)
n = 1000     : 168 primes   (theorique: ~144.8)
n = 1000000  : 78498 primes (theorique: ~72382)
```

---

## 📊 SECTION 2.5 : DANS LA VRAIE VIE

### Qui utilise ces concepts ?

| Metier | Cas d'usage |
|--------|-------------|
| **Cryptographe** | Generation de cles RSA (besoin de grands nombres premiers) |
| **Ingenieur Securite** | Verification de primalite pour protocoles TLS/SSL |
| **Data Scientist** | Hachage universel avec nombres premiers |
| **Developpeur Jeux** | Generation procedurale avec proprietes des primes |
| **Chercheur IA** | Embeddings de grande dimension (techniques FFT/NTT) |
| **Ingenieur Reseau** | Protocoles de synchronisation (CRT) |

### Exemple Concret : RSA

Pour generer une cle RSA de 2048 bits :
1. Trouver deux grands nombres premiers p et q (~1024 bits chacun)
2. Calculer n = p * q (module RSA)
3. La securite repose sur la difficulte de factoriser n

Notre `sieve_basic` et `linear_sieve` sont les bases de ces algorithmes.

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
primes.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo run --release
Test sieve_basic(30):
Primes: [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
Count: 10

Test linear_sieve(20):
Primes: [2, 3, 5, 7, 11, 13, 17, 19]
SPF[12] = 2  (12 = 2 * 6)
SPF[15] = 3  (15 = 3 * 5)

Test segmented_sieve(100, 120):
Primes in range: [101, 103, 107, 109, 113]

Test factorize_spf(60):
60 = 2^2 * 3^1 * 5^1

Test mobius_sieve(10):
mu[1] = 1, mu[2] = -1, mu[4] = 0, mu[6] = 1

Test nth_prime(100):
The 100th prime is 541

All tests passed!
```

---

## ⚡ SECTION 3.1 : BONUS AVANCE (OPTIONNEL)

**Difficulte Bonus :**
★★★★★★★★☆☆ (8/10)

**Recompense :**
XP ×3

**Time Complexity attendue :**
O(n / log log n) pour le crible bitwise

**Space Complexity attendue :**
O(n / 64) — 64x moins de memoire que la version de base

**Domaines Bonus :**
`CPU, ASM, Mem`

### 3.1.1 Consigne Bonus

**🎬 "There is no spoon" — L'Ultime Optimisation**

L'enfant prodige dans la salle d'attente de l'Oracle dit a Neo : *"N'essaie pas de plier la cuillere. C'est impossible. Realise plutot la verite : il n'y a pas de cuillere."*

De meme, n'essaie pas de stocker des `bool` individuels. Realise la verite : **il n'y a que des bits**.

**Ta mission : Implementer un crible bitwise ultra-optimise**

```rust
/// Crible bitwise - 64x moins de memoire
/// Chaque u64 stocke 64 flags de primalite
pub fn sieve_bitwise(n: usize) -> Vec<u64>;

/// Crible de Sundaram - approche alternative
pub fn sieve_sundaram(n: usize) -> Vec<bool>;

/// Crible wheel-optimized (skip 2, 3, 5)
pub fn sieve_wheel(n: usize) -> Vec<bool>;
```

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  n ≤ 10^9                               │
│  Memoire : O(n / 64) = ~15 MB pour 10^9 │
│  Temps : < 1 seconde pour n = 10^8      │
│  Pas de SIMD explicite (mais autorise)  │
└─────────────────────────────────────────┘
```

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `sieve_bitwise(64)[1]` | `0x28208A20A08A28AC` | Bits des primes encodees |
| `is_prime_bitwise(sieve, 17)` | `true` | Extraction du bit 17 |

### 3.1.2 Prototype Bonus

```rust
pub mod primes_optimized {
    /// Crible bitwise avec compression 64:1
    pub fn sieve_bitwise(n: usize) -> Vec<u64>;

    /// Verifie si n est premier en utilisant le crible bitwise
    pub fn is_prime_bitwise(sieve: &[u64], n: usize) -> bool;

    /// Crible de Sundaram (ne trouve que les impairs > 2)
    pub fn sieve_sundaram(n: usize) -> Vec<bool>;

    /// Crible avec roue de 30 (2*3*5)
    pub fn sieve_wheel_30(n: usize) -> Vec<bool>;

    /// Compte les primes jusqu'a n en O(n^(2/3))
    pub fn count_primes_fast(n: u64) -> u64;
}
```

### 3.1.3 Ce qui change par rapport a l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Stockage | 1 byte par nombre | 1 bit par nombre |
| Memoire pour 10^9 | ~1 GB | ~15 MB |
| Skip pairs | Non | Oui (wheel optimization) |
| Complexite | O(n log log n) | O(n / log log n) |
| Edge cases | n < 10^7 | n < 10^9 |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap? |
|------|-------|----------|--------|-------|
| sieve_basic_null | n=0 | [] or [F] | 2 | Yes |
| sieve_basic_one | n=1 | [F,F] | 2 | Yes |
| sieve_basic_two | n=2 | [F,F,T] | 3 | Yes |
| sieve_basic_10 | n=10 | [F,F,T,T,F,T,F,T,F,F,F] | 5 | No |
| sieve_basic_100 | n=100 | 25 primes | 5 | No |
| linear_primes | n=20 | [2,3,5,7,11,13,17,19] | 5 | No |
| linear_spf_12 | n=20, check spf[12] | 2 | 5 | No |
| linear_spf_15 | n=20, check spf[15] | 3 | 5 | No |
| segmented_100_120 | lo=100, hi=120 | [101,103,107,109,113] | 10 | No |
| segmented_empty | lo=120, hi=100 | [] | 5 | Yes |
| segmented_large | lo=10^9, hi=10^9+100 | correct primes | 10 | No |
| factorize_60 | n=60, spf | [(2,2),(3,1),(5,1)] | 10 | No |
| factorize_prime | n=97, spf | [(97,1)] | 5 | No |
| factorize_power | n=64, spf | [(2,6)] | 5 | No |
| mobius_1 | n=10, check mu[1] | 1 | 3 | No |
| mobius_prime | n=10, check mu[7] | -1 | 3 | No |
| mobius_square | n=10, check mu[4] | 0 | 3 | Yes |
| mobius_squarefree | n=10, check mu[6] | 1 | 3 | No |
| nth_1 | n=1 | 2 | 3 | No |
| nth_10 | n=10 | 29 | 3 | No |
| nth_100 | n=100 | 541 | 5 | No |
| nth_1000 | n=1000 | 7919 | 5 | No |
| **TOTAL** | | | **100** | |

---

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include "primes.h"

void test_sieve_basic(void)
{
    size_t size;
    bool *is_prime;

    // Test n = 0
    is_prime = sieve_basic(0, &size);
    assert(size <= 1);
    free(is_prime);

    // Test n = 10
    is_prime = sieve_basic(10, &size);
    assert(size == 11);
    assert(is_prime[0] == false);
    assert(is_prime[1] == false);
    assert(is_prime[2] == true);
    assert(is_prime[3] == true);
    assert(is_prime[4] == false);
    assert(is_prime[5] == true);
    assert(is_prime[7] == true);
    assert(is_prime[9] == false);
    free(is_prime);

    printf("sieve_basic: OK\n");
}

void test_linear_sieve(void)
{
    t_linear_sieve *result = linear_sieve(20);

    assert(result != NULL);
    assert(result->primes_count == 8);
    assert(result->primes[0] == 2);
    assert(result->primes[7] == 19);
    assert(result->spf[12] == 2);
    assert(result->spf[15] == 3);

    free_linear_sieve(result);
    printf("linear_sieve: OK\n");
}

void test_segmented_sieve(void)
{
    size_t count;
    uint64_t *primes;

    primes = segmented_sieve(100, 120, &count);
    assert(count == 5);
    assert(primes[0] == 101);
    assert(primes[1] == 103);
    assert(primes[2] == 107);
    assert(primes[3] == 109);
    assert(primes[4] == 113);
    free(primes);

    // Empty range
    primes = segmented_sieve(120, 100, &count);
    assert(count == 0);
    free(primes);

    printf("segmented_sieve: OK\n");
}

void test_factorize_spf(void)
{
    t_linear_sieve *sieve = linear_sieve(100);
    t_factorization *fact;

    fact = factorize_spf(60, sieve->spf, sieve->spf_size);
    assert(fact->count == 3);
    assert(fact->factors[0].prime == 2 && fact->factors[0].exponent == 2);
    assert(fact->factors[1].prime == 3 && fact->factors[1].exponent == 1);
    assert(fact->factors[2].prime == 5 && fact->factors[2].exponent == 1);

    free_factorization(fact);
    free_linear_sieve(sieve);
    printf("factorize_spf: OK\n");
}

void test_mobius_sieve(void)
{
    size_t size;
    int8_t *mu = mobius_sieve(10, &size);

    assert(mu[1] == 1);   // mu(1) = 1
    assert(mu[2] == -1);  // prime
    assert(mu[3] == -1);  // prime
    assert(mu[4] == 0);   // 4 = 2^2
    assert(mu[5] == -1);  // prime
    assert(mu[6] == 1);   // 6 = 2*3, deux facteurs
    assert(mu[7] == -1);  // prime
    assert(mu[8] == 0);   // 8 = 2^3
    assert(mu[9] == 0);   // 9 = 3^2
    assert(mu[10] == 1);  // 10 = 2*5, deux facteurs

    free(mu);
    printf("mobius_sieve: OK\n");
}

void test_nth_prime(void)
{
    assert(nth_prime(1) == 2);
    assert(nth_prime(10) == 29);
    assert(nth_prime(100) == 541);
    assert(nth_prime(1000) == 7919);

    printf("nth_prime: OK\n");
}

int main(void)
{
    printf("=== Prime Sieve Tests ===\n\n");

    test_sieve_basic();
    test_linear_sieve();
    test_segmented_sieve();
    test_factorize_spf();
    test_mobius_sieve();
    test_nth_prime();

    printf("\n=== All tests passed! ===\n");
    return 0;
}
```

---

### 4.3 Solution de reference (Rust)

```rust
pub mod primes {
    /// Crible d'Eratosthene basique - O(n log log n)
    pub fn sieve_basic(n: usize) -> Vec<bool> {
        if n < 2 {
            return vec![false; n + 1];
        }

        let mut is_prime = vec![true; n + 1];
        is_prime[0] = false;
        is_prime[1] = false;

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

    /// Crible lineaire - O(n) exact avec SPF
    pub fn linear_sieve(n: usize) -> (Vec<usize>, Vec<usize>) {
        if n < 2 {
            return (Vec::new(), vec![0; n + 1]);
        }

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

    /// Crible segmente pour grands intervalles
    pub fn segmented_sieve(lo: u64, hi: u64) -> Vec<u64> {
        if lo > hi {
            return Vec::new();
        }

        let lo = lo.max(2);
        let sqrt_hi = (hi as f64).sqrt() as u64 + 1;

        // Generer les petits primes
        let small_primes: Vec<u64> = {
            let sieve = sieve_basic(sqrt_hi as usize);
            sieve.iter()
                .enumerate()
                .filter(|(_, &is_p)| is_p)
                .map(|(i, _)| i as u64)
                .collect()
        };

        // Crible segmente
        let size = (hi - lo + 1) as usize;
        let mut is_prime = vec![true; size];

        for &p in &small_primes {
            if p * p > hi {
                break;
            }

            let start = if p * p >= lo {
                (p * p - lo) as usize
            } else {
                let rem = lo % p;
                if rem == 0 { 0 } else { (p - rem) as usize }
            };

            let mut j = start;
            while j < size {
                // Ne pas marquer p lui-meme comme composite
                if lo + j as u64 != p {
                    is_prime[j] = false;
                }
                j += p as usize;
            }
        }

        is_prime.iter()
            .enumerate()
            .filter(|(_, &is_p)| is_p)
            .map(|(i, _)| lo + i as u64)
            .collect()
    }

    /// Factorisation utilisant SPF - O(log n)
    pub fn factorize_spf(mut n: usize, spf: &[usize]) -> Vec<(usize, usize)> {
        if n <= 1 || n >= spf.len() {
            return Vec::new();
        }

        let mut factors = Vec::new();

        while n > 1 {
            let p = spf[n];
            let mut exp = 0;

            while n > 1 && spf[n] == p {
                n /= p;
                exp += 1;
            }

            factors.push((p, exp));
        }

        factors
    }

    /// Fonction de Mobius
    pub fn mobius_sieve(n: usize) -> Vec<i8> {
        if n == 0 {
            return vec![0];
        }

        let (primes, spf) = linear_sieve(n);
        let mut mu = vec![1i8; n + 1];
        mu[0] = 0;

        for i in 2..=n {
            let p = spf[i];
            let i_div_p = i / p;

            if i_div_p % p == 0 {
                // i a un facteur carre
                mu[i] = 0;
            } else {
                // i = p * (i/p) ou p n'apparait qu'une fois
                mu[i] = -mu[i_div_p];
            }
        }

        mu
    }

    /// Trouve le n-ieme premier (1-indexe)
    pub fn nth_prime(n: usize) -> u64 {
        if n == 0 {
            return 0;
        }
        if n == 1 {
            return 2;
        }

        // Estimation haute du n-ieme premier
        let estimate = if n < 6 {
            15
        } else {
            let ln_n = (n as f64).ln();
            let ln_ln_n = ln_n.ln();
            (n as f64 * (ln_n + ln_ln_n)) as usize + 100
        };

        let sieve = sieve_basic(estimate);
        let mut count = 0;

        for (i, &is_p) in sieve.iter().enumerate() {
            if is_p {
                count += 1;
                if count == n {
                    return i as u64;
                }
            }
        }

        // Si pas trouve, doubler l'estimation
        nth_prime_recursive(n, estimate * 2)
    }

    fn nth_prime_recursive(n: usize, limit: usize) -> u64 {
        let sieve = sieve_basic(limit);
        let mut count = 0;

        for (i, &is_p) in sieve.iter().enumerate() {
            if is_p {
                count += 1;
                if count == n {
                    return i as u64;
                }
            }
        }

        nth_prime_recursive(n, limit * 2)
    }
}
```

---

### 4.4 Solutions alternatives acceptees

#### Alternative 1 : Sieve avec skip des pairs

```rust
pub fn sieve_basic_odd_only(n: usize) -> Vec<bool> {
    if n < 2 {
        return vec![false; n + 1];
    }

    let mut is_prime = vec![true; n + 1];
    is_prime[0] = false;
    is_prime[1] = false;

    // Marquer tous les pairs > 2
    for i in (4..=n).step_by(2) {
        is_prime[i] = false;
    }

    // Ne verifier que les impairs
    let mut i = 3;
    while i * i <= n {
        if is_prime[i] {
            let mut j = i * i;
            while j <= n {
                is_prime[j] = false;
                j += 2 * i;  // Skip pairs
            }
        }
        i += 2;
    }

    is_prime
}
```

#### Alternative 2 : Linear sieve avec vecteur pre-alloue

```rust
pub fn linear_sieve_preallocated(n: usize) -> (Vec<usize>, Vec<usize>) {
    let mut spf = vec![0; n + 1];
    // Pre-allocation approximative
    let mut primes = Vec::with_capacity(n / (n.max(10) as f64).ln() as usize + 10);

    for i in 2..=n {
        if spf[i] == 0 {
            spf[i] = i;
            primes.push(i);
        }
        for &p in &primes {
            let ip = i * p;
            if p > spf[i] || ip > n { break; }
            spf[ip] = p;
        }
    }

    (primes, spf)
}
```

---

### 4.5 Solutions refusees (avec explications)

#### REFUSE 1 : Ne verifie pas n < 2

```rust
// REFUSE : Panic ou comportement indefini pour n = 0 ou 1
pub fn sieve_basic_bad(n: usize) -> Vec<bool> {
    let mut is_prime = vec![true; n + 1];  // OK pour n >= 1
    is_prime[0] = false;
    is_prime[1] = false;  // PANIC si n = 0!
    // ...
}
// Pourquoi c'est refuse : Crash sur entree n = 0
```

#### REFUSE 2 : Linear sieve sans condition de break

```rust
// REFUSE : O(n log n) au lieu de O(n)
pub fn linear_sieve_bad(n: usize) -> (Vec<usize>, Vec<usize>) {
    let mut spf = vec![0; n + 1];
    let mut primes = Vec::new();

    for i in 2..=n {
        if spf[i] == 0 {
            spf[i] = i;
            primes.push(i);
        }
        for &p in &primes {
            if i * p > n { break; }
            // MANQUE : if p > spf[i] { break; }
            spf[i * p] = p;  // Ecrit plusieurs fois!
        }
    }
    (primes, spf)
}
// Pourquoi c'est refuse : Complexite incorrecte, certains composites marques plusieurs fois
```

#### REFUSE 3 : Mobius ne detecte pas les carres

```rust
// REFUSE : Retourne des valeurs incorrectes pour les carres parfaits
pub fn mobius_sieve_bad(n: usize) -> Vec<i8> {
    let mut mu = vec![1i8; n + 1];

    for i in 2..=n {
        for j in (i..=n).step_by(i) {
            mu[j] = -mu[j];  // Flip le signe
        }
        // MANQUE : Detection des facteurs carres!
    }
    mu
}
// Pourquoi c'est refuse : mu[4] = 1 au lieu de 0
```

---

### 4.6 Solution bonus de reference (COMPLETE)

```rust
pub mod primes_optimized {
    /// Crible bitwise - 64x compression
    pub fn sieve_bitwise(n: usize) -> Vec<u64> {
        if n < 2 {
            return vec![0];
        }

        let size = (n / 64) + 1;
        let mut bits = vec![!0u64; size];

        // 0 et 1 ne sont pas premiers
        bits[0] &= !3u64;  // Clear bits 0 and 1

        let mut i = 2;
        while i * i <= n {
            if is_prime_bitwise(&bits, i) {
                let mut j = i * i;
                while j <= n {
                    // Clear bit j
                    bits[j / 64] &= !(1u64 << (j % 64));
                    j += i;
                }
            }
            i += 1;
        }

        bits
    }

    /// Verifie si n est premier dans le crible bitwise
    pub fn is_prime_bitwise(sieve: &[u64], n: usize) -> bool {
        if n / 64 >= sieve.len() {
            return false;
        }
        (sieve[n / 64] >> (n % 64)) & 1 == 1
    }

    /// Crible de Sundaram
    pub fn sieve_sundaram(n: usize) -> Vec<bool> {
        if n < 2 {
            return vec![false; n + 1];
        }

        let k = (n - 1) / 2;
        let mut marked = vec![false; k + 1];

        let mut i = 1;
        while i <= k {
            let mut j = i;
            while i + j + 2 * i * j <= k {
                marked[i + j + 2 * i * j] = true;
                j += 1;
            }
            i += 1;
        }

        let mut is_prime = vec![false; n + 1];
        if n >= 2 {
            is_prime[2] = true;
        }

        for i in 1..=k {
            if !marked[i] {
                let p = 2 * i + 1;
                if p <= n {
                    is_prime[p] = true;
                }
            }
        }

        is_prime
    }

    /// Crible avec roue de 30 (2*3*5)
    pub fn sieve_wheel_30(n: usize) -> Vec<bool> {
        if n < 2 {
            return vec![false; n + 1];
        }

        let mut is_prime = vec![false; n + 1];

        if n >= 2 { is_prime[2] = true; }
        if n >= 3 { is_prime[3] = true; }
        if n >= 5 { is_prime[5] = true; }

        // Residus modulo 30 qui peuvent etre premiers
        let wheel = [1, 7, 11, 13, 17, 19, 23, 29];

        // Marquer les candidats initiaux
        for base in (0..=n).step_by(30) {
            for &r in &wheel {
                let num = base + r;
                if num <= n && num > 1 {
                    is_prime[num] = true;
                }
            }
        }

        // Cribler
        let mut i = 7;
        while i * i <= n {
            if is_prime[i] {
                let mut j = i * i;
                while j <= n {
                    is_prime[j] = false;
                    j += i;
                }
            }
            i += 1;
            // Sauter au prochain candidat wheel
            while i * i <= n && !is_prime.get(i).copied().unwrap_or(false) {
                i += 1;
            }
        }

        is_prime
    }

    /// Compte les primes - version optimisee
    pub fn count_primes_fast(n: u64) -> u64 {
        if n < 2 {
            return 0;
        }

        let sieve = super::primes::sieve_basic(n as usize);
        sieve.iter().filter(|&&x| x).count() as u64
    }
}
```

---

### 4.7 Solutions alternatives bonus (COMPLETES)

```rust
// Alternative bonus : Crible bitwise avec SIMD-style operations
pub fn sieve_bitwise_fast(n: usize) -> Vec<u64> {
    if n < 2 {
        return vec![0];
    }

    let size = (n / 64) + 1;
    let mut bits = vec![!0u64; size];
    bits[0] &= !3u64;

    // Optimisation : traiter 2 separement
    {
        let mut j = 4;
        while j <= n {
            bits[j / 64] &= !(1u64 << (j % 64));
            j += 2;
        }
    }

    // Ne traiter que les impairs
    let mut i = 3;
    while i * i <= n {
        if (bits[i / 64] >> (i % 64)) & 1 == 1 {
            let mut j = i * i;
            let step = 2 * i;  // Skip pairs
            while j <= n {
                bits[j / 64] &= !(1u64 << (j % 64));
                j += step;
            }
        }
        i += 2;
    }

    bits
}
```

---

### 4.8 Solutions refusees bonus (COMPLETES)

```rust
// REFUSE : Utilise f64 pour sqrt (interdit en C)
pub fn sieve_with_sqrt(n: usize) -> Vec<bool> {
    let limit = (n as f64).sqrt() as usize;  // INTERDIT!
    // ...
}
// Pourquoi : Les fonctions math de <math.h> sont interdites
```

---

### 4.9 spec.json (ENGINE v22.1 — FORMAT STRICT)

```json
{
  "name": "sieve_matrix_awakening",
  "language": "rust",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isole - Cribles de nombres premiers",
  "tags": ["primes", "sieve", "number-theory", "phase1", "matrix"],
  "passing_score": 70,

  "function": {
    "name": "sieve_basic",
    "prototype": "pub fn sieve_basic(n: usize) -> Vec<bool>",
    "return_type": "Vec<bool>",
    "parameters": [
      {"name": "n", "type": "usize"}
    ]
  },

  "additional_functions": [
    {
      "name": "linear_sieve",
      "prototype": "pub fn linear_sieve(n: usize) -> (Vec<usize>, Vec<usize>)",
      "return_type": "(Vec<usize>, Vec<usize>)"
    },
    {
      "name": "segmented_sieve",
      "prototype": "pub fn segmented_sieve(lo: u64, hi: u64) -> Vec<u64>",
      "return_type": "Vec<u64>"
    },
    {
      "name": "factorize_spf",
      "prototype": "pub fn factorize_spf(n: usize, spf: &[usize]) -> Vec<(usize, usize)>",
      "return_type": "Vec<(usize, usize)>"
    },
    {
      "name": "mobius_sieve",
      "prototype": "pub fn mobius_sieve(n: usize) -> Vec<i8>",
      "return_type": "Vec<i8>"
    },
    {
      "name": "nth_prime",
      "prototype": "pub fn nth_prime(n: usize) -> u64",
      "return_type": "u64"
    }
  ],

  "driver": {
    "reference": "pub fn ref_sieve_basic(n: usize) -> Vec<bool> { if n < 2 { return vec![false; n + 1]; } let mut is_prime = vec![true; n + 1]; is_prime[0] = false; is_prime[1] = false; let mut i = 2; while i * i <= n { if is_prime[i] { let mut j = i * i; while j <= n { is_prime[j] = false; j += i; } } i += 1; } is_prime }",

    "edge_cases": [
      {
        "name": "n_zero",
        "args": [0],
        "expected": "vec![false]",
        "is_trap": true,
        "trap_explanation": "n=0 doit retourner [false] ou tableau vide, pas panic"
      },
      {
        "name": "n_one",
        "args": [1],
        "expected": "vec![false, false]",
        "is_trap": true,
        "trap_explanation": "1 n'est PAS premier"
      },
      {
        "name": "n_two",
        "args": [2],
        "expected": "vec![false, false, true]",
        "is_trap": true,
        "trap_explanation": "2 EST premier (le seul pair)"
      },
      {
        "name": "n_ten",
        "args": [10],
        "expected": "primes at 2,3,5,7"
      },
      {
        "name": "n_hundred",
        "args": [100],
        "expected": "25 primes"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 1000,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {
            "min": 0,
            "max": 10000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["Vec::new", "vec!", "push", "iter", "enumerate", "filter", "map", "collect"],
    "forbidden_functions": ["primal", "num_prime", "primes"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes (minimum 5)

#### Mutant A (Boundary) : Commence a i=1

```rust
/* Mutant A (Boundary) : Commence le crible a i=1 au lieu de i=2 */
pub fn sieve_basic_mutant_a(n: usize) -> Vec<bool> {
    if n < 2 {
        return vec![false; n + 1];
    }

    let mut is_prime = vec![true; n + 1];
    is_prime[0] = false;
    // BUG: Oublie de marquer 1 comme non-premier!

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
// Pourquoi c'est faux : is_prime[1] = true, mais 1 n'est pas premier
// Ce qui etait pense : "J'ai marque 0 comme faux, c'est bon"
```

#### Mutant B (Safety) : Pas de verification n < 2

```rust
/* Mutant B (Safety) : Ne verifie pas les petites valeurs de n */
pub fn sieve_basic_mutant_b(n: usize) -> Vec<bool> {
    let mut is_prime = vec![true; n + 1];
    is_prime[0] = false;
    is_prime[1] = false;  // PANIC si n = 0!

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
// Pourquoi c'est faux : Panic sur n=0 car is_prime[1] n'existe pas
// Ce qui etait pense : "Qui va appeler sieve_basic(0) ?"
```

#### Mutant C (Logic) : Linear sieve sans break sur p > spf[i]

```rust
/* Mutant C (Logic) : Manque la condition d'arret dans linear_sieve */
pub fn linear_sieve_mutant_c(n: usize) -> (Vec<usize>, Vec<usize>) {
    let mut spf = vec![0; n + 1];
    let mut primes = Vec::new();

    for i in 2..=n {
        if spf[i] == 0 {
            spf[i] = i;
            primes.push(i);
        }
        for &p in &primes {
            if i * p > n {
                break;
            }
            // BUG: Manque "if p > spf[i] { break; }"
            spf[i * p] = p;
        }
    }

    (primes, spf)
}
// Pourquoi c'est faux : Complexite O(n log n) au lieu de O(n), et spf peut etre ecrase
// Ce qui etait pense : "La condition i*p > n suffit"
```

#### Mutant D (Off-by-one) : Segmented sieve avec mauvais offset

```rust
/* Mutant D (Off-by-one) : Calcul incorrect du point de depart */
pub fn segmented_sieve_mutant_d(lo: u64, hi: u64) -> Vec<u64> {
    if lo > hi {
        return Vec::new();
    }

    let sqrt_hi = ((hi as f64).sqrt() as u64) + 1;
    let small_primes: Vec<u64> = /* ... crible basique ... */;

    let size = (hi - lo + 1) as usize;
    let mut is_prime = vec![true; size];

    for &p in &small_primes {
        // BUG: Mauvais calcul du start
        let start = lo / p * p;  // Devrait etre: max(p*p, ((lo + p - 1) / p) * p)

        let mut j = if start >= lo { (start - lo) as usize } else { 0 };
        while j < size {
            is_prime[j] = false;
            j += p as usize;
        }
    }

    // ...
}
// Pourquoi c'est faux : Marque des primes comme composites (quand start = p)
// Ce qui etait pense : "lo / p * p donne le premier multiple de p >= lo"
```

#### Mutant E (Return) : Mobius ne detecte pas les facteurs carres

```rust
/* Mutant E (Return) : Mobius ignore les facteurs carres */
pub fn mobius_sieve_mutant_e(n: usize) -> Vec<i8> {
    let (_, spf) = linear_sieve(n);
    let mut mu = vec![1i8; n + 1];
    mu[0] = 0;

    for i in 2..=n {
        let p = spf[i];
        let i_div_p = i / p;

        // BUG: Pas de detection des facteurs carres!
        mu[i] = -mu[i_div_p];
    }

    mu
}
// Pourquoi c'est faux : mu[4] = 1 au lieu de 0, mu[8] = -1 au lieu de 0
// Ce qui etait pense : "Chaque facteur premier flip le signe"
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

Cet exercice enseigne **6 concepts fondamentaux** de la theorie des nombres :

1. **Crible d'Eratosthene** — L'algorithme classique pour trouver tous les nombres premiers
2. **Crible lineaire** — Optimisation a O(n) exact avec calcul simultane du SPF
3. **Crible segmente** — Technique pour traiter de tres grands intervalles
4. **Factorisation SPF** — Decomposition ultra-rapide en O(log n)
5. **Fonction de Mobius** — Fonction multiplicative fondamentale
6. **Approximation des primes** — Le theoreme des nombres premiers en pratique

---

### 5.2 LDA — Traduction litterale en francais (MAJUSCULES)

#### sieve_basic(n)

```
FONCTION sieve_basic QUI RETOURNE UN VECTEUR DE BOOLEENS ET PREND EN PARAMETRE n QUI EST UN ENTIER NON SIGNE
DEBUT FONCTION
    SI n EST INFERIEUR A 2 ALORS
        RETOURNER UN VECTEUR DE n PLUS 1 ELEMENTS TOUS FAUX
    FIN SI

    DECLARER is_prime COMME VECTEUR DE BOOLEENS DE TAILLE n PLUS 1 INITIALISE A VRAI
    AFFECTER FAUX A is_prime A LA POSITION 0
    AFFECTER FAUX A is_prime A LA POSITION 1

    DECLARER i COMME ENTIER
    AFFECTER 2 A i

    TANT QUE i MULTIPLIE PAR i EST INFERIEUR OU EGAL A n FAIRE
        SI is_prime A LA POSITION i EST VRAI ALORS
            DECLARER j COMME ENTIER
            AFFECTER i MULTIPLIE PAR i A j

            TANT QUE j EST INFERIEUR OU EGAL A n FAIRE
                AFFECTER FAUX A is_prime A LA POSITION j
                AFFECTER j PLUS i A j
            FIN TANT QUE
        FIN SI
        INCREMENTER i DE 1
    FIN TANT QUE

    RETOURNER is_prime
FIN FONCTION
```

---

### 5.2.2 Style Academique Francais

```
ALGORITHME : Crible d'Eratosthene
ENTREE : n (entier naturel)
SORTIE : tableau booleen indiquant la primalite de chaque entier de 0 a n

DEBUT
    CREER tableau is_prime[0..n] initialise a VRAI
    is_prime[0] <- FAUX
    is_prime[1] <- FAUX

    POUR i DE 2 A racine(n) FAIRE
        SI is_prime[i] = VRAI ALORS
            POUR j DE i*i A n PAR PAS DE i FAIRE
                is_prime[j] <- FAUX
            FIN POUR
        FIN SI
    FIN POUR

    RETOURNER is_prime
FIN
```

---

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : Crible d'Eratosthene
---
1. INITIALISER tableau is_prime[0..n] a VRAI

2. MARQUER 0 et 1 comme NON PREMIERS

3. POUR chaque i de 2 a sqrt(n) :
   a. SI i est encore marque premier :
      |
      |-- POUR j = i*i, i*i+i, i*i+2i, ... jusqu'a n :
      |     MARQUER j comme NON PREMIER
      |
   b. SINON : passer au suivant

4. RETOURNER le tableau is_prime
```

---

### 5.2.3 Representation Algorithmique

```
FONCTION : sieve_basic(n)
---
INIT is_prime = [VRAI] * (n+1)

1. GARDES (Fail Fast) :
   |
   |-- VERIFIER si n < 2 :
   |     RETOURNER [FAUX] * (n+1)
   |
   |-- MARQUER is_prime[0] = FAUX
   |-- MARQUER is_prime[1] = FAUX

2. PHASE DE CRIBLAGE :
   |
   |-- POUR i = 2 TANT QUE i*i <= n :
   |     |
   |     |-- SI is_prime[i] = VRAI :
   |     |     POUR j = i*i, i*i+i, ... <= n :
   |     |       is_prime[j] <- FAUX
   |     |
   |     |-- i <- i + 1

3. RETOURNER is_prime
```

---

### 5.2.3.1 Diagramme Mermaid

```mermaid
graph TD
    A[Debut: sieve_basic] --> B{n < 2 ?}
    B -- Oui --> C[Retourner tableau de FAUX]
    B -- Non --> D[Initialiser is_prime a VRAI]

    D --> E[Marquer 0 et 1 comme FAUX]
    E --> F[i = 2]

    F --> G{i * i <= n ?}
    G -- Non --> H[Retourner is_prime]
    G -- Oui --> I{is_prime[i] ?}

    I -- Non --> J[i = i + 1]
    I -- Oui --> K[j = i * i]

    K --> L{j <= n ?}
    L -- Non --> J
    L -- Oui --> M[is_prime[j] = FAUX]
    M --> N[j = j + i]
    N --> L

    J --> G
```

---

### 5.3 Visualisation ASCII (adaptee au sujet)

#### Le Crible en Action (n = 30)

```
ETAPE 0 : Initialisation
Position:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
          [X][X][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T][T]
                 ↑
                 i=2 commence

ETAPE 1 : Cribler les multiples de 2
Position:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
          [X][X][T][T][X][T][X][T][X][T][X][T][X][T][X][T][X][T][X][T][X][T][X][T][X][T][X][T][X][T][X]
                       ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑
                       4     6     8    10    12    14    16    18    20    22    24    26    28    30

ETAPE 2 : Cribler les multiples de 3 (a partir de 9)
Position:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
          [X][X][T][T][X][T][X][T][X][X][X][T][X][T][X][X][X][T][X][T][X][X][X][T][X][T][X][X][X][T][X]
                                   ↑                 ↑                 ↑                 ↑
                                   9                15                21                27

ETAPE 3 : Cribler les multiples de 5 (a partir de 25)
Position:  0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
          [X][X][T][T][X][T][X][T][X][X][X][T][X][T][X][X][X][T][X][T][X][X][X][T][X][X][X][X][X][T][X]
                                                                                  ↑
                                                                                 25

RESULTAT FINAL : Primes = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29}
```

#### Structure SPF (Smallest Prime Factor)

```
Nombre:     2   3   4   5   6   7   8   9  10  11  12  13  14  15
           ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
SPF:       │ 2 │ 3 │ 2 │ 5 │ 2 │ 7 │ 2 │ 3 │ 2 │11 │ 2 │13 │ 2 │ 3 │
           └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
             ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑
            p   p  2*2  p  2*3  p  2*4 3*3 2*5  p  2*6  p  2*7 3*5

Legende: p = premier (SPF = lui-meme)

Factorisation de 60 avec SPF:
60 → SPF[60]=2 → 60/2=30 → SPF[30]=2 → 30/2=15 → SPF[15]=3 → 15/3=5 → SPF[5]=5 → 5/5=1
Resultat: 60 = 2² × 3¹ × 5¹
```

#### Fonction de Mobius

```
n:      1   2   3   4   5   6   7   8   9  10
       ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
mu(n): │ 1 │-1 │-1 │ 0 │-1 │ 1 │-1 │ 0 │ 0 │ 1 │
       └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
         │   │   │   │   │   │   │   │   │   │
         │   │   │   │   │   │   │   │   │   └── 10=2×5 → 2 facteurs → (-1)²=1
         │   │   │   │   │   │   │   │   └────── 9=3² → facteur carre → 0
         │   │   │   │   │   │   │   └────────── 8=2³ → facteur carre → 0
         │   │   │   │   │   │   └────────────── 7 premier → 1 facteur → (-1)¹=-1
         │   │   │   │   │   └────────────────── 6=2×3 → 2 facteurs → (-1)²=1
         │   │   │   │   └────────────────────── 5 premier → 1 facteur → (-1)¹=-1
         │   │   │   └────────────────────────── 4=2² → facteur carre → 0
         │   │   └────────────────────────────── 3 premier → 1 facteur → (-1)¹=-1
         │   └────────────────────────────────── 2 premier → 1 facteur → (-1)¹=-1
         └────────────────────────────────────── mu(1) = 1 par definition
```

---

### 5.4 Les pieges en detail

#### Piege 1 : Oublier que 0 et 1 ne sont pas premiers

```rust
// BUG : 1 sera marque comme premier!
let mut is_prime = vec![true; n + 1];
is_prime[0] = false;
// Oubli de : is_prime[1] = false;
```

**Solution :** Toujours explicitement marquer 0 et 1 comme non-premiers.

#### Piege 2 : Commencer le criblage a i*2 au lieu de i*i

```rust
// INEFFICACE : Les multiples < i*i sont deja marques
let mut j = i * 2;  // Devrait etre i * i
while j <= n {
    is_prime[j] = false;
    j += i;
}
```

**Pourquoi i*i ?** Si j < i*i, alors j = k*i ou k < i. Donc j a deja ete marque par le premier k.

#### Piege 3 : Linear sieve sans la condition p > spf[i]

Cette condition est **cruciale** pour garantir O(n) :
- Chaque composite n est visite **exactement une fois** : quand on traite n/spf[n]
- Sans cette condition, on visite certains composites plusieurs fois

```rust
for &p in &primes {
    if p > spf[i] || i * p > n {  // LES DEUX conditions!
        break;
    }
    spf[i * p] = p;
}
```

#### Piege 4 : Segmented sieve - mauvais point de depart

Le premier multiple de p dans [lo, hi] n'est pas simplement `lo / p * p` :
- Si lo = 100 et p = 7, lo/p*p = 98, mais on veut le premier >= 100
- Formule correcte : `((lo + p - 1) / p) * p` ou `max(p*p, ceil(lo/p)*p)`

#### Piege 5 : Mobius oublie les facteurs carres

mu(n) = 0 si n contient un facteur premier au carre. Beaucoup oublient ce cas.

```rust
// Verifier si i/p contient encore p comme facteur
if (i / p) % p == 0 {
    mu[i] = 0;  // Facteur carre detecte!
}
```

---

### 5.5 Cours Complet (VRAI cours, pas un resume)

#### 5.5.1 Introduction aux Nombres Premiers

**Definition :** Un nombre premier est un entier naturel > 1 qui n'a que deux diviseurs : 1 et lui-meme.

```
Premiers: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, ...
Non-premiers (composites): 4, 6, 8, 9, 10, 12, 14, 15, 16, 18, ...
```

**Theoreme Fondamental de l'Arithmetique :**
Tout entier n > 1 s'ecrit de maniere unique comme produit de puissances de nombres premiers :
```
n = p1^a1 × p2^a2 × ... × pk^ak
```

Exemple : `360 = 2³ × 3² × 5¹`

#### 5.5.2 Le Crible d'Eratosthene

**Principe :** Pour trouver tous les premiers jusqu'a n :
1. Ecrire tous les entiers de 2 a n
2. Le premier non-barre (2) est premier
3. Barrer tous ses multiples (4, 6, 8, ...)
4. Passer au prochain non-barre (3), c'est premier
5. Barrer ses multiples (6, 9, 12, ...)
6. Repeter jusqu'a sqrt(n)

**Pourquoi s'arreter a sqrt(n) ?**
Si n = a × b avec a <= b, alors a <= sqrt(n). Donc tout composite <= n a un facteur <= sqrt(n).

**Complexite :**
- Temps : O(n log log n)
- Espace : O(n)

La somme des inverses des premiers diverge comme log log n (theoreme de Mertens).

#### 5.5.3 Le Crible Lineaire

**Probleme du crible classique :** Certains composites sont marques plusieurs fois.
Exemple : 12 est marque par 2 (12=2×6) et par 3 (12=3×4).

**Solution :** Chaque composite n est marque **exactement une fois** par son plus petit facteur premier (SPF).

**Algorithme :**
```
Pour i de 2 a n:
    Si spf[i] = 0:  # i est premier
        spf[i] = i
        ajouter i a la liste des premiers

    Pour chaque premier p <= spf[i]:
        Si i*p > n: break
        spf[i*p] = p
```

**Invariant cle :** `p <= spf[i]` garantit que p est bien le SPF de i*p.

**Complexite :** O(n) temps et espace (exactement n-1 affectations a spf).

#### 5.5.4 Le Crible Segmente

**Probleme :** Pour n = 10^12, on ne peut pas allouer 10^12 bytes.

**Solution :** Traiter l'intervalle [lo, hi] par segments de taille sqrt(hi).

**Algorithme :**
1. Generer les petits premiers jusqu'a sqrt(hi) avec un crible classique
2. Pour chaque segment [L, L+S) :
   - Creer un tableau de taille S
   - Pour chaque petit premier p, marquer ses multiples dans le segment
3. Concatener les resultats

**Complexite :**
- Temps : O((hi - lo + 1) log log hi + sqrt(hi))
- Espace : O(sqrt(hi))

#### 5.5.5 Factorisation avec SPF

Une fois le tableau SPF calcule, factoriser n devient trivial :

```
Tant que n > 1:
    p = SPF[n]
    compter combien de fois p divise n
    n = n / p^k
```

**Complexite :** O(log n) car n est divise par au moins 2 a chaque etape.

#### 5.5.6 La Fonction de Mobius

**Definition :**
```
mu(1) = 1
mu(n) = 0           si n a un facteur premier au carre
mu(n) = (-1)^k      si n = p1 × p2 × ... × pk (k premiers distincts)
```

**Propriete fondamentale :**
```
Somme(mu(d), d | n) = {
    1 si n = 1
    0 sinon
}
```

Cette propriete est a la base de l'inversion de Mobius, un outil puissant en combinatoire.

**Calcul par crible :**
En utilisant le SPF, on peut calculer mu(i) a partir de mu(i/p) :
- Si (i/p) % p == 0 : mu(i) = 0 (facteur carre)
- Sinon : mu(i) = -mu(i/p) (un facteur premier de plus)

#### 5.5.7 Le n-ieme Nombre Premier

**Theoreme des Nombres Premiers :**
```
pi(n) ~ n / ln(n)
```

ou pi(n) est le nombre de premiers <= n.

**Corollaire :** Le n-ieme premier p_n satisfait :
```
p_n ~ n × ln(n)
```

Pour trouver p_n :
1. Estimer une borne superieure (n × ln(n) + n × ln(ln(n)))
2. Cribler jusqu'a cette borne
3. Compter les premiers jusqu'au n-ieme

---

### 5.6 Normes avec explications pedagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ let mut is_prime = vec![true; n + 1];                           │
│ is_prime[0] = false; is_prime[1] = false;                       │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ let mut is_prime = vec![true; n + 1];                           │
│ is_prime[0] = false;                                            │
│ is_prime[1] = false;                                            │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Lisibilite : Une operation par ligne                          │
│ • Debugging : Facile de commenter une ligne                     │
│ • Git diff : Changements visibles individuellement              │
└─────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ while i*i<=n{if is_prime[i]{...}}                               │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ while i * i <= n {                                              │
│     if is_prime[i] {                                            │
│         // ...                                                  │
│     }                                                           │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Espaces autour des operateurs : lisibilite                    │
│ • Indentation : structure visuelle claire                       │
│ • Convention Rust : rustfmt applique ce style                   │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.7 Simulation avec trace d'execution

#### sieve_basic(10)

```
┌───────┬─────────────────────────────────────────────┬─────┬─────────────────────────────────┐
│ Etape │ Instruction                                 │  i  │ is_prime[]                      │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   1   │ Initialiser is_prime a [true; 11]          │  -  │ [T,T,T,T,T,T,T,T,T,T,T]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   2   │ is_prime[0] = false                        │  -  │ [F,T,T,T,T,T,T,T,T,T,T]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   3   │ is_prime[1] = false                        │  -  │ [F,F,T,T,T,T,T,T,T,T,T]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   4   │ i = 2, 2*2=4 <= 10 ? OUI                   │  2  │ [F,F,T,T,T,T,T,T,T,T,T]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   5   │ is_prime[2] = true ? OUI, cribler          │  2  │ [F,F,T,T,T,T,T,T,T,T,T]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   6   │ j=4: is_prime[4]=false                     │  2  │ [F,F,T,T,F,T,T,T,T,T,T]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   7   │ j=6: is_prime[6]=false                     │  2  │ [F,F,T,T,F,T,F,T,T,T,T]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   8   │ j=8: is_prime[8]=false                     │  2  │ [F,F,T,T,F,T,F,T,F,T,T]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│   9   │ j=10: is_prime[10]=false                   │  2  │ [F,F,T,T,F,T,F,T,F,T,F]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│  10   │ j=12 > 10, sortir boucle                   │  2  │ [F,F,T,T,F,T,F,T,F,T,F]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│  11   │ i = 3, 3*3=9 <= 10 ? OUI                   │  3  │ [F,F,T,T,F,T,F,T,F,T,F]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│  12   │ is_prime[3] = true ? OUI, cribler          │  3  │ [F,F,T,T,F,T,F,T,F,T,F]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│  13   │ j=9: is_prime[9]=false                     │  3  │ [F,F,T,T,F,T,F,T,F,F,F]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│  14   │ j=12 > 10, sortir boucle                   │  3  │ [F,F,T,T,F,T,F,T,F,F,F]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│  15   │ i = 4, 4*4=16 > 10 ? OUI, sortir           │  4  │ [F,F,T,T,F,T,F,T,F,F,F]         │
├───────┼─────────────────────────────────────────────┼─────┼─────────────────────────────────┤
│  16   │ Retourner is_prime                         │  -  │ [F,F,T,T,F,T,F,T,F,F,F]         │
└───────┴─────────────────────────────────────────────┴─────┴─────────────────────────────────┘

Resultat: Primes = {2, 3, 5, 7}
```

---

### 5.8 Mnemotechniques (MEME obligatoire)

#### 🎬 MEME : "There is no spoon" — Les composites n'existent pas vraiment

![There is no spoon](meme_spoon.jpg)

Dans Matrix, l'enfant dit a Neo : *"N'essaie pas de plier la cuillere. C'est impossible. Realise plutot qu'il n'y a pas de cuillere."*

Pour les nombres premiers, c'est pareil :
> *"N'essaie pas de trouver les composites. Realise plutot qu'apres le crible, il n'y a que des primes."*

```rust
// There is no composite...
for i in 2..=n {
    if is_prime[i] {
        // ...only primes remain
        primes.push(i);
    }
}
```

---

#### 🔴🔵 MEME : "Red Pill vs Blue Pill" — Linear Sieve vs Basic Sieve

**Blue Pill (Basic Sieve) :** O(n log log n) — Tu restes dans l'ignorance, certains composites sont visites plusieurs fois.

**Red Pill (Linear Sieve) :** O(n) exact — Tu vois la verite, chaque nombre est visite exactement une fois.

```rust
// Tu prends la pilule rouge...
if p > spf[i] {
    break;  // Tu ne revisites jamais le meme composite
}
// ...et tu decouvres que la Matrix est O(n)
```

---

#### 🕴️ MEME : "Agent Smith = Composite Number"

L'Agent Smith peut se dupliquer (12 = 2 × 6 = 3 × 4 = 2 × 2 × 3).
Neo (nombre premier) ne peut pas etre decompose. Il est The One.

```rust
// Decomposition d'un Agent
pub fn factorize_spf(mut agent: usize, spf: &[usize]) -> Vec<(usize, usize)> {
    // L'Agent se decompose en ses composants premiers
    while agent > 1 {
        let prime_component = spf[agent];
        // Chaque composant est un "Neo" irreductible
    }
}
```

---

#### 📞 MEME : "The Matrix has you" — Mobius Function

La fonction mu detecte les "glitches" dans la Matrix (facteurs carres).

```
mu(n) = 0  ⟹ "Glitch detected" (facteur carre)
mu(n) ≠ 0 ⟹ "Clean signal" (squarefree)
```

---

### 5.9 Applications pratiques

#### 1. Cryptographie RSA
```rust
// Generation de cles RSA
fn generate_rsa_key(bits: u32) -> (BigInt, BigInt, BigInt) {
    let p = random_prime(bits / 2);  // Utilise notre sieve pour tester
    let q = random_prime(bits / 2);
    let n = p * q;
    let phi = (p - 1) * (q - 1);
    let e = 65537;
    let d = mod_inverse(e, phi);
    (n, e, d)
}
```

#### 2. Hachage Universel
```rust
// Fonction de hachage avec premier aleatoire
fn universal_hash(key: u64, prime: u64, table_size: usize) -> usize {
    let a = random() % prime;
    let b = random() % prime;
    ((a * key + b) % prime) as usize % table_size
}
```

#### 3. Inclusion-Exclusion
```rust
// Compter les entiers copremiers avec n utilisant Mobius
fn count_coprime(limit: u64, n: u64) -> u64 {
    let divisors = all_divisors(n);
    let mu = mobius_values(&divisors);

    divisors.iter()
        .zip(mu.iter())
        .map(|(d, m)| *m as i64 * (limit / d) as i64)
        .sum::<i64>() as u64
}
```

---

## ⚠️ SECTION 6 : PIEGES — RECAPITULATIF

| Piege | Description | Solution |
|-------|-------------|----------|
| **0 et 1** | Oublier de marquer 0 et 1 comme non-premiers | Toujours `is_prime[0] = is_prime[1] = false` |
| **i*2 vs i*i** | Commencer le criblage a i*2 au lieu de i*i | Commencer a `j = i * i` |
| **Linear break** | Oublier `p > spf[i]` dans le crible lineaire | Ajouter les deux conditions de break |
| **Segmented offset** | Mauvais calcul du premier multiple dans [lo, hi] | `max(p*p, ceil(lo/p)*p)` |
| **Mobius square** | Ignorer les facteurs carres | Verifier `(i/p) % p == 0` |
| **nth_prime bound** | Sous-estimer la borne superieure | Utiliser `n * (ln(n) + ln(ln(n)))` |

---

## 📝 SECTION 7 : QCM

### Question 1 : Complexite du crible d'Eratosthene

Quelle est la complexite temporelle du crible d'Eratosthene classique pour trouver tous les premiers jusqu'a n ?

- A) O(n)
- B) O(n log n)
- C) O(n log log n)
- D) O(n sqrt(n))
- E) O(n^2)
- F) O(sqrt(n))
- G) O(n / log n)
- H) O(n^(3/2))
- I) O(2^n)
- J) O(log n)

**Reponse : C**

---

### Question 2 : SPF de 84

Quel est le Smallest Prime Factor (SPF) de 84 ?

- A) 1
- B) 2
- C) 3
- D) 4
- E) 7
- F) 12
- G) 21
- H) 42
- I) 84
- J) Aucun

**Reponse : B** (84 = 2 × 42)

---

### Question 3 : Fonction de Mobius

Quelle est la valeur de mu(30) ?

- A) -3
- B) -2
- C) -1
- D) 0
- E) 1
- F) 2
- G) 3
- H) 30
- I) Undefined
- J) Depends on implementation

**Reponse : C** (30 = 2 × 3 × 5, trois facteurs premiers distincts, donc (-1)^3 = -1)

---

### Question 4 : Condition du crible lineaire

Pourquoi la condition `p > spf[i]` est-elle necessaire dans le crible lineaire ?

- A) Pour accelerer l'algorithme
- B) Pour garantir que chaque composite est marque exactement une fois
- C) Pour eviter les depassements de tableau
- D) Pour trier les premiers
- E) Pour calculer mu correctement
- F) Ce n'est pas necessaire
- G) Pour la lisibilite du code
- H) Pour la compatibilite C/Rust
- I) Pour eviter les nombres negatifs
- J) Pour le parallelisme

**Reponse : B**

---

### Question 5 : Crible segmente

Quel est l'avantage principal du crible segmente ?

- A) Plus rapide que le crible classique
- B) Permet de traiter de tres grands intervalles avec peu de memoire
- C) Plus simple a implementer
- D) Fonctionne pour les nombres negatifs
- E) Calcule automatiquement SPF
- F) Parallilisable
- G) Ne necessite pas de pretraitement
- H) Fonctionne pour les flottants
- I) Genere des nombres aleatoires
- J) Toutes les reponses ci-dessus

**Reponse : B**

---

### Question 6 : 10eme nombre premier

Quel est le 10eme nombre premier ?

- A) 19
- B) 23
- C) 27
- D) 29
- E) 31
- F) 37
- G) 41
- H) 43
- I) 47
- J) 53

**Reponse : D** (2, 3, 5, 7, 11, 13, 17, 19, 23, 29)

---

### Question 7 : mu(1)

Quelle est la valeur de mu(1) ?

- A) -1
- B) 0
- C) 1
- D) Undefined
- E) Depends on n
- F) Infinity
- G) -Infinity
- H) Empty set
- I) 2
- J) pi

**Reponse : C** (Par definition, mu(1) = 1)

---

### Question 8 : Pourquoi sqrt(n) ?

Dans le crible, pourquoi s'arrete-t-on a i <= sqrt(n) ?

- A) Pour la performance
- B) Parce que tout composite <= n a un facteur <= sqrt(n)
- C) Convention historique
- D) Limitation du hardware
- E) Pour eviter l'overflow
- F) Theoreme de Fermat
- G) Hypothese de Riemann
- H) Arbitraire
- I) Pour le debugging
- J) Standard POSIX

**Reponse : B**

---

### Question 9 : 2 est-il premier ?

Le nombre 2 est-il premier ?

- A) Oui, c'est le seul premier pair
- B) Non, les premiers sont impairs
- C) Depends de la definition
- D) Seulement en mathematiques
- E) Seulement en informatique
- F) C'est un cas special
- G) 2 n'est pas un nombre
- H) Question mal posee
- I) Oui, mais seulement pour n > 10
- J) Non, 2 = 1 + 1

**Reponse : A**

---

### Question 10 : Factorisation avec SPF

Quelle est la complexite de la factorisation avec SPF precalcule ?

- A) O(1)
- B) O(log n)
- C) O(sqrt(n))
- D) O(n)
- E) O(n log n)
- F) O(n^2)
- G) O(2^n)
- H) O(log log n)
- I) O(n / log n)
- J) Depends du nombre

**Reponse : B** (On divise par au moins 2 a chaque etape)

---

## 📊 SECTION 8 : RECAPITULATIF

| Critere | Valeur |
|---------|--------|
| **Exercice ID** | 1.6.1-a |
| **Module** | 1.6.1 — Prime Numbers & Sieves |
| **Difficulte** | 6/10 |
| **Type** | complet (cours + QCM + code) |
| **Tiers** | 1 — Concept isole |
| **Langages** | Rust Edition 2024, C (C17) |
| **Duree estimee** | 90 min |
| **XP Base** | 150 |
| **XP Bonus** | 150 × 3 = 450 |
| **Complexite temps** | O(n log log n) |
| **Complexite espace** | O(n) |
| **Fonctions** | 6 (sieve_basic, linear_sieve, segmented_sieve, factorize_spf, mobius_sieve, nth_prime) |
| **Edge cases** | n=0, n=1, n=2, lo>hi, facteurs carres |
| **Theme** | The Matrix |
| **MEME** | "There is no spoon" |

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.6.1-a-sieve-matrix-awakening",
    "generated_at": "2026-01-12 12:00:00",

    "metadata": {
      "exercise_id": "1.6.1-a",
      "exercise_name": "sieve_matrix_awakening",
      "module": "1.6.1",
      "module_name": "Prime Numbers & Sieves",
      "concept": "a",
      "concept_name": "Sieve algorithms and prime factorization",
      "type": "complet",
      "tier": 1,
      "tier_info": "Concept isole",
      "phase": 1,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust,c",
      "language_version": "Rust Edition 2024, C17",
      "duration_minutes": 90,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCE",
      "bonus_icon": "🔥",
      "complexity_time": "T5 O(n log log n)",
      "complexity_space": "S4 O(n)",
      "prerequisites": ["loops", "arrays", "basic-arithmetic", "pointers"],
      "domains": ["MD", "Tri", "Mem"],
      "domains_bonus": ["CPU", "ASM"],
      "tags": ["primes", "sieve", "eratosthenes", "linear-sieve", "segmented", "mobius", "number-theory"],
      "meme_reference": "The Matrix - There is no spoon"
    },

    "files": {
      "spec.json": "/* Section 4.9 content */",
      "references/ref_sieve_basic.rs": "/* Section 4.3 sieve_basic */",
      "references/ref_linear_sieve.rs": "/* Section 4.3 linear_sieve */",
      "references/ref_segmented_sieve.rs": "/* Section 4.3 segmented_sieve */",
      "references/ref_factorize_spf.rs": "/* Section 4.3 factorize_spf */",
      "references/ref_mobius_sieve.rs": "/* Section 4.3 mobius_sieve */",
      "references/ref_nth_prime.rs": "/* Section 4.3 nth_prime */",
      "references/ref_bonus.rs": "/* Section 4.6 */",
      "alternatives/alt_sieve_odd_only.rs": "/* Section 4.4 */",
      "alternatives/alt_linear_preallocated.rs": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 Mutant A */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 Mutant B */",
      "mutants/mutant_c_logic.rs": "/* Section 4.10 Mutant C */",
      "mutants/mutant_d_offbyone.rs": "/* Section 4.10 Mutant D */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 Mutant E */",
      "tests/main.c": "/* Section 4.2 */",
      "tests/test_rust.rs": "/* Rust test suite */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_sieve_basic.rs",
        "references/ref_linear_sieve.rs",
        "references/ref_segmented_sieve.rs",
        "references/ref_factorize_spf.rs",
        "references/ref_mobius_sieve.rs",
        "references/ref_nth_prime.rs",
        "references/ref_bonus.rs",
        "alternatives/alt_sieve_odd_only.rs",
        "alternatives/alt_linear_preallocated.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_safety.rs",
        "mutants/mutant_c_logic.rs",
        "mutants/mutant_d_offbyone.rs",
        "mutants/mutant_e_return.rs"
      ]
    },

    "commands": {
      "validate_spec": "python3 hackbrain_engine_v22.py --validate-spec spec.json",
      "test_reference_rust": "cargo test --release",
      "test_reference_c": "gcc -Wall -Wextra -Werror -std=c17 primes.c main.c -o test && ./test",
      "test_mutants": "python3 hackbrain_mutation_tester.py -r references/ -s spec.json --validate"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — L'excellence pedagogique ne se negocie pas*
*Module 1.6.1 — Prime Numbers & Sieves*
*Theme: The Matrix — "Free your mind, Neo. The Matrix is built on primes."*
