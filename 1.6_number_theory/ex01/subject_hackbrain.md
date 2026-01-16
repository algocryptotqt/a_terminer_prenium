<thinking>
## Analyse du Concept
- Concept : GCD, LCM, Extended Euclidean Algorithm, Modular Inverse, Chinese Remainder Theorem, Euler's Totient, Farey Sequence
- Phase demandee : 1 (Intermediaire)
- Adapte ? OUI - Ces algorithmes fondamentaux de theorie des nombres sont parfaits pour Phase 1. Ils necessitent une comprehension de la recursion et de l'arithmetique modulaire, accessibles a ce niveau.

## Combo Base + Bonus
- Exercice de base : 6 fonctions thematiques Interstellar couvrant les concepts fondamentaux
  - gcd : Plus grand commun diviseur (synchronisation orbitale)
  - extended_gcd : Coefficients de Bezout (calculs de trous de ver)
  - mod_inverse : Inverse modulaire (inversion de dilatation temporelle)
  - chinese_remainder : Theoreme des restes chinois (coordonnees multi-planetes)
  - euler_phi : Fonction phi d'Euler (comptage de dimensions)
  - farey_sequence : Suite de Farey (frequences d'ondes gravitationnelles)
- Bonus : Binary GCD (Stein's Algorithm) - evite les divisions, plus efficace au niveau binaire
- Palier bonus : 🔥 Avance (multiplicateur x3) - optimisation low-level
- Progression logique ? OUI - Base = implementation classique, Bonus = optimisation binaire

## Prerequis & Difficulte
- Prerequis reels : Arithmetique de base, Recursion, Modulo
- Difficulte estimee : 6/10
- Coherent avec phase ? OUI - Phase 1 autorise 3-5/10, 6/10 est justifie car synthese de plusieurs concepts

## Aspect Fun/Culture
- Contexte choisi : Film "Interstellar" de Christopher Nolan (2014)
- MEME mnémotechnique : "This is no time for caution" - l'urgence de verifier les cas limites
- Pourquoi c'est fun :
  1. L'analogie orbite/GCD est parfaite : deux orbites se synchronisent quand leur periode a un diviseur commun
  2. Le trou de ver = Extended GCD : trouver les coefficients de Bezout c'est trouver le chemin inverse
  3. La dilatation temporelle = inverse modulaire : inverser le temps dans un systeme cyclique
  4. Le CRT = coordonnees sur plusieurs planetes : chaque planete a son propre "modulo"
  5. TARS et CASE = robots qui executent les algorithmes, parfait pour la metaphore
  6. Reference culturelle majeure, film sur la science et les maths
- Score d'intelligence de l'analogie : 98/100 - Les metaphores sont scientifiquement coherentes

## Scenarios d'Echec (5 mutants concrets)

### Mutant A (Boundary) : gcd avec b=0
```rust
// MUTANT: Ne gere pas le cas de base correctement
pub fn gcd(a: u64, b: u64) -> u64 {
    if b == 0 { return 0; }  // ERREUR: gcd(a, 0) = a, pas 0
    gcd(b, a % b)
}
```
Erreur : gcd(a, 0) doit retourner a, pas 0

### Mutant B (Safety) : extended_gcd avec valeurs negatives mal gerees
```rust
// MUTANT: Oublie de gerer le signe pour les coefficients de Bezout
pub fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    if b == 0 {
        return (a, 1, 0);  // ERREUR: ne gere pas a negatif
    }
    let (g, x, y) = extended_gcd(b, a % b);
    (g, y, x - (a / b) * y)
}
```
Erreur : Pour a negatif, le GCD doit etre positif mais les coefficients ajustes

### Mutant C (Logic) : mod_inverse sans verifier gcd != 1
```rust
// MUTANT: Calcule l'inverse meme si gcd(a,m) != 1
pub fn mod_inverse(a: i64, m: i64) -> Option<i64> {
    let (_, x, _) = extended_gcd(a, m);
    Some(((x % m) + m) % m)  // ERREUR: Doit retourner None si gcd != 1
}
```
Erreur : L'inverse modulaire n'existe que si gcd(a,m) = 1

### Mutant D (Logic) : chinese_remainder avec mauvaise combinaison
```rust
// MUTANT: Oublie de multiplier par l'inverse modulaire
pub fn chinese_remainder(remainders: &[i64], moduli: &[i64]) -> Option<(i64, i64)> {
    let m = moduli.iter().product::<i64>();
    let mut x = 0;
    for i in 0..remainders.len() {
        let mi = m / moduli[i];
        x += remainders[i] * mi;  // ERREUR: manque * mod_inverse(mi, moduli[i])
    }
    Some((x % m, m))
}
```
Erreur : Le CRT necessite la multiplication par l'inverse modulaire

### Mutant E (Return) : euler_phi retourne n au lieu de phi(n)
```rust
// MUTANT: Confusion entre n et phi(n)
pub fn euler_phi(n: u64) -> u64 {
    if n <= 1 { return n; }
    let mut result = n;  // Correct
    let mut temp = n;
    let mut p = 2;
    while p * p <= temp {
        if temp % p == 0 {
            while temp % p == 0 { temp /= p; }
            result -= result / p;
        }
        p += 1;
    }
    // ERREUR: Oublie le dernier facteur premier > sqrt(n)
    result
}
```
Erreur : Oublie de traiter le cas ou temp > 1 apres la boucle (dernier facteur premier)

## Verdict
VALIDE - L'exercice est complet, l'analogie Interstellar est scientifiquement coherente et pedagogique, les 5 mutants sont concrets et testables.
</thinking>

# Exercice [1.6.2-a] : interstellar_euclidean

**Module :**
1.6.2 — Euclidean Algorithms

**Concept :**
a — GCD, Extended GCD, Modular Inverse, CRT, Euler Phi, Farey

**Difficulte :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
3 — Synthese (concepts d→l combines)

**Langage :**
Rust Edition 2024 & C (c17)

**Prerequis :**
- Arithmetique de base (Module 1.6.1)
- Recursion
- Operateur modulo (%)
- Notion de divisibilite

**Domaines :**
MD, Algo, Crypto

**Duree estimee :**
90 min

**XP Base :**
200

**Complexite :**
T4 O(log(min(a,b))) × S2 O(log(min(a,b)))

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers a rendre :**
```
Rust:  src/lib.rs (module interstellar_euclidean)
C:     interstellar_euclidean.c, interstellar_euclidean.h
```

**Fonctions autorisees :**
- Rust : Standard library (std::cmp, std::collections)
- C : malloc, free, abs, labs

**Fonctions interdites :**
- Bibliotheques mathematiques externes (GMP, num-bigint en mode non-standard)
- Fonctions pre-implementees de GCD/LCM

---

### 1.2 Consigne

#### 1.2.1 Version Culture Pop — "Interstellar: Cooper's Cosmic Mathematics"

**🚀 CONTEXTE : La Mission Lazarus**

*"We used to look up at the sky and wonder at our place in the stars. Now we just look down, and worry about our place in the dirt."* — Cooper

L'humanite est au bord de l'extinction. Cooper, ancien pilote de la NASA devenu fermier, decouvre qu'une anomalie gravitationnelle dans la chambre de sa fille Murphy transmet des coordonnees en binaire. Ces coordonnees menent a une base secrete de la NASA ou le Professeur Brand lui revele le Plan A : resoudre l'equation de la gravite pour sauver l'humanite.

Mais pour naviguer a travers le trou de ver Gargantua et explorer les planetes candidates, l'equipe a besoin d'outils mathematiques puissants. TARS et CASE, les robots de l'Endurance, ont ete programmes avec des algorithmes euclidiens — mais leur firmware a ete corrompu par les radiations du trou de ver.

**Ta mission : Reprogrammer les systemes mathematiques de l'Endurance.**

---

**L'equipage et leurs besoins mathematiques :**

| Membre | Role | Besoin Mathematique |
|--------|------|---------------------|
| **Cooper** | Pilote | `gcd` — Synchronisation des orbites |
| **TARS** | Robot | `extended_gcd` — Calculs de navigation trou de ver |
| **Brand** | Biologiste | `mod_inverse` — Inversion de dilatation temporelle |
| **Romilly** | Physicien | `chinese_remainder` — Coordonnees multi-planetes |
| **CASE** | Robot | `euler_phi` — Comptage des dimensions traversables |
| **Murphy** | Scientifique (Terre) | `farey_sequence` — Frequences d'ondes gravitationnelles |

---

**Fonction 1 : `gcd(a, b)` — La Synchronisation Orbitale de Cooper**

Cooper doit synchroniser l'orbite de l'Endurance avec celle des planetes candidates. Deux corps celestes avec des periodes orbitales de `a` et `b` unites de temps se retrouveront alignes tous les `lcm(a, b)` unites — mais pour calculer le LCM efficacement, il faut d'abord le GCD.

*"Those aren't mountains... they're waves."* — Cooper

L'algorithme d'Euclide permet de calculer le Plus Grand Commun Diviseur (GCD) de deux nombres en remplacant iterativement le plus grand par le reste de la division.

**Entree :**
- `a` : Premiere periode orbitale (0 ≤ a ≤ 10^18)
- `b` : Seconde periode orbitale (0 ≤ b ≤ 10^18)

**Sortie :**
- `gcd(a, b)` — Le plus grand entier divisant a la fois a et b

**Proprietes fondamentales :**
```
gcd(a, 0) = a
gcd(a, b) = gcd(b, a % b)
gcd(a, b) = gcd(b, a)
gcd(a, b) * lcm(a, b) = a * b
```

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `gcd(48, 18)` | `6` | 48 = 6×8, 18 = 6×3, GCD = 6 |
| `gcd(0, 5)` | `5` | gcd(0, n) = n |
| `gcd(5, 0)` | `5` | gcd(n, 0) = n |
| `gcd(17, 23)` | `1` | Nombres premiers entre eux |
| `gcd(1071, 462)` | `21` | 1071 = 21×51, 462 = 21×22 |

---

**Fonction 2 : `extended_gcd(a, b)` — Les Calculs de Trou de Ver de TARS**

Pour traverser le trou de ver, TARS doit resoudre l'equation de Bezout : trouver `x` et `y` tels que `ax + by = gcd(a, b)`. Ces coefficients representent les "vecteurs de correction" necessaires pour stabiliser la trajectoire.

*"It's not possible." "No, it's necessary."* — TARS & Cooper

L'algorithme d'Euclide etendu retourne non seulement le GCD, mais aussi les coefficients de Bezout (x, y).

**Entree :**
- `a` : Premier coefficient (peut etre negatif)
- `b` : Second coefficient (peut etre negatif)

**Sortie :**
- Tuple `(gcd, x, y)` tel que `a*x + b*y = gcd`

**Identite de Bezout :**
```
Pour tout a, b, il existe x, y tels que : ax + by = gcd(a, b)
```

**Exemples :**

| Appel | Retour (g, x, y) | Verification |
|-------|------------------|--------------|
| `extended_gcd(35, 15)` | `(5, 1, -2)` | 35×1 + 15×(-2) = 35 - 30 = 5 ✓ |
| `extended_gcd(161, 28)` | `(7, -1, 6)` | 161×(-1) + 28×6 = -161 + 168 = 7 ✓ |
| `extended_gcd(17, 0)` | `(17, 1, 0)` | 17×1 + 0×0 = 17 ✓ |
| `extended_gcd(0, 23)` | `(23, 0, 1)` | 0×0 + 23×1 = 23 ✓ |

---

**Fonction 3 : `mod_inverse(a, m)` — L'Inversion Temporelle de Brand**

Sur la planete de Miller, une heure equivaut a 7 ans sur Terre a cause de la dilatation temporelle pres de Gargantua. Brand doit "inverser" les calculs de temps : si `a` secondes sur Miller correspondent a un certain temps terrestre, quel est le facteur inverse dans l'arithmetique modulaire ?

L'inverse modulaire de `a` modulo `m` est l'entier `x` tel que `(a × x) mod m = 1`. Il n'existe que si `gcd(a, m) = 1`.

*"Love is the one thing we're capable of perceiving that transcends dimensions of time and space."* — Brand

**Entree :**
- `a` : Valeur dont on cherche l'inverse (1 ≤ |a| ≤ 10^9)
- `m` : Le modulo (m > 1)

**Sortie :**
- `Some(x)` tel que `(a × x) mod m = 1`, ou `None` si impossible

**Condition d'existence :**
```
L'inverse modulaire de a mod m existe ⟺ gcd(a, m) = 1
```

**Exemples :**

| Appel | Retour | Verification |
|-------|--------|--------------|
| `mod_inverse(3, 7)` | `Some(5)` | 3×5 = 15 ≡ 1 (mod 7) ✓ |
| `mod_inverse(6, 9)` | `None` | gcd(6,9) = 3 ≠ 1 |
| `mod_inverse(17, 43)` | `Some(38)` | 17×38 = 646 = 15×43 + 1 ✓ |
| `mod_inverse(1, 100)` | `Some(1)` | 1×1 ≡ 1 (mod 100) ✓ |

---

**Fonction 4 : `chinese_remainder(remainders, moduli)` — Les Coordonnees Multi-Planetes de Romilly**

Romilly reste en orbite autour de Gargantua pendant que Cooper et Brand explorent les planetes. Pour synchroniser les communications avec trois planetes ayant des periodes de rotation differentes, il utilise le Theoreme des Restes Chinois (CRT).

Le CRT permet de trouver un nombre `x` qui laisse des restes specifiques quand on le divise par plusieurs moduli copremiers.

*"I've waited years..."* — Romilly

**Probleme :**
```
Trouver x tel que :
  x ≡ r[0] (mod m[0])
  x ≡ r[1] (mod m[1])
  ...
  x ≡ r[n-1] (mod m[n-1])
```

**Entree :**
- `remainders` : Tableau des restes [r0, r1, ..., rn-1]
- `moduli` : Tableau des moduli [m0, m1, ..., mn-1] (doivent etre copremiers deux a deux)

**Sortie :**
- `Some((x, M))` ou x est la solution minimale positive et M = m0 × m1 × ... × mn-1
- `None` si les moduli ne sont pas copremiers

**Exemples :**

| Appel | Retour | Verification |
|-------|--------|--------------|
| `chinese_remainder(&[2, 3, 2], &[3, 5, 7])` | `Some((23, 105))` | 23%3=2, 23%5=3, 23%7=2 ✓ |
| `chinese_remainder(&[1, 2], &[3, 5])` | `Some((7, 15))` | 7%3=1, 7%5=2 ✓ |
| `chinese_remainder(&[0, 0], &[4, 6])` | `None` | gcd(4,6)=2≠1 |

---

**Fonction 5 : `euler_phi(n)` — Le Comptage Dimensionnel de CASE**

CASE analyse les dimensions traversables du tesseract. La fonction phi d'Euler φ(n) compte combien de nombres entre 1 et n sont copremiers avec n — c'est-a-dire combien de "chemins independants" existent dans l'espace a n dimensions.

*"What happens now?"* — Cooper dans le tesseract

**Definition :**
```
φ(n) = |{k : 1 ≤ k ≤ n et gcd(k, n) = 1}|
```

**Formule :**
```
φ(n) = n × ∏(p|n) (1 - 1/p)

ou p parcourt les facteurs premiers de n.
```

**Entree :**
- `n` : L'entier dont on calcule la fonction phi (1 ≤ n ≤ 10^9)

**Sortie :**
- `φ(n)` — Nombre d'entiers copremiers avec n dans [1, n]

**Exemples :**

| Appel | Retour | Copremiers |
|-------|--------|------------|
| `euler_phi(1)` | `1` | {1} |
| `euler_phi(2)` | `1` | {1} |
| `euler_phi(6)` | `2` | {1, 5} |
| `euler_phi(10)` | `4` | {1, 3, 7, 9} |
| `euler_phi(12)` | `4` | {1, 5, 7, 11} |
| `euler_phi(7)` | `6` | {1, 2, 3, 4, 5, 6} (7 est premier) |

---

**Fonction 6 : `farey_sequence(n)` — Les Ondes Gravitationnelles de Murphy**

Murphy recoit des messages de son pere a travers les ondes gravitationnelles du tesseract. Ces ondes sont codees comme des fractions irreductibles — la suite de Farey F_n contient toutes les fractions p/q avec 0 ≤ p ≤ q ≤ n et gcd(p, q) = 1.

*"Don't let me leave, Murph!"* — Cooper

La suite de Farey d'ordre n est la sequence croissante de toutes les fractions irreductibles entre 0 et 1 dont le denominateur est au plus n.

**Entree :**
- `n` : L'ordre de la suite de Farey (1 ≤ n ≤ 100)

**Sortie :**
- Vecteur de tuples (numerateur, denominateur) tries par valeur croissante

**Exemples :**

| Appel | Retour |
|-------|--------|
| `farey_sequence(1)` | `[(0,1), (1,1)]` |
| `farey_sequence(2)` | `[(0,1), (1,2), (1,1)]` |
| `farey_sequence(3)` | `[(0,1), (1,3), (1,2), (2,3), (1,1)]` |
| `farey_sequence(4)` | `[(0,1), (1,4), (1,3), (1,2), (2,3), (3,4), (1,1)]` |

**Propriete de mediant :**
```
Si a/b et c/d sont consecutifs dans F_n, alors :
  bc - ad = 1  (adjacence de Farey)
  Le mediant (a+c)/(b+d) est la premiere fraction entre eux dans F_(b+d)
```

---

#### 1.2.2 Version Academique

**Objectif :**
Implementer 6 fonctions fondamentales de theorie des nombres :

1. **gcd(a, b)** : Calcule le plus grand commun diviseur par l'algorithme d'Euclide
2. **extended_gcd(a, b)** : Retourne (gcd, x, y) avec ax + by = gcd (identite de Bezout)
3. **mod_inverse(a, m)** : Calcule l'inverse modulaire de a mod m si gcd(a,m)=1
4. **chinese_remainder(r, m)** : Resout le systeme de congruences par le CRT
5. **euler_phi(n)** : Calcule la fonction indicatrice d'Euler φ(n)
6. **farey_sequence(n)** : Genere la suite de Farey d'ordre n

**Contraintes :**
- Gerer les cas limites (a=0, b=0, n=1, etc.)
- Les moduli du CRT doivent etre copremiers deux a deux
- L'inverse modulaire n'existe que si gcd(a,m)=1
- Complexite O(log(min(a,b))) pour GCD/Extended GCD
- Complexite O(sqrt(n)) pour euler_phi

---

### 1.3 Prototype

#### Rust (Edition 2024)

```rust
//! Module: Interstellar Euclidean
//! Cooper's team needs your number theory skills to navigate Gargantua

/// Greatest Common Divisor using Euclidean algorithm
/// Returns gcd(a, b) - the largest number dividing both a and b
pub fn gcd(a: u64, b: u64) -> u64;

/// Extended Euclidean algorithm
/// Returns (gcd, x, y) such that a*x + b*y = gcd(a, b)
pub fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64);

/// Modular multiplicative inverse
/// Returns Some(x) where (a * x) % m = 1, or None if gcd(a, m) != 1
pub fn mod_inverse(a: i64, m: i64) -> Option<i64>;

/// Chinese Remainder Theorem
/// Solves x ≡ r[i] (mod m[i]) for all i
/// Returns Some((x, M)) where M = product of moduli, or None if not coprime
pub fn chinese_remainder(remainders: &[i64], moduli: &[i64]) -> Option<(i64, i64)>;

/// Euler's totient function φ(n)
/// Counts integers in [1, n] coprime with n
pub fn euler_phi(n: u64) -> u64;

/// Farey sequence of order n
/// Returns all irreducible fractions p/q with 0 ≤ p ≤ q ≤ n, sorted
pub fn farey_sequence(n: u64) -> Vec<(u64, u64)>;

/// LCM using GCD (bonus utility)
pub fn lcm(a: u64, b: u64) -> u64;

/// GCD of multiple numbers (bonus utility)
pub fn gcd_multiple(nums: &[u64]) -> u64;
```

#### C (c17)

```c
#ifndef INTERSTELLAR_EUCLIDEAN_H
#define INTERSTELLAR_EUCLIDEAN_H

#include <stdint.h>
#include <stdbool.h>
#include <stdlib.h>

// Greatest Common Divisor
uint64_t gcd(uint64_t a, uint64_t b);

// Extended GCD result structure
typedef struct {
    int64_t gcd;
    int64_t x;
    int64_t y;
} ExtendedGcdResult;

// Extended Euclidean algorithm
ExtendedGcdResult extended_gcd(int64_t a, int64_t b);

// Modular inverse result
typedef struct {
    bool exists;
    int64_t value;
} ModInverseResult;

// Modular multiplicative inverse
ModInverseResult mod_inverse(int64_t a, int64_t m);

// Chinese Remainder Theorem result
typedef struct {
    bool valid;
    int64_t x;
    int64_t m;
} CrtResult;

// Chinese Remainder Theorem
CrtResult chinese_remainder(const int64_t* remainders, const int64_t* moduli, size_t n);

// Euler's totient function
uint64_t euler_phi(uint64_t n);

// Farey sequence (caller must free the result)
typedef struct {
    uint64_t num;
    uint64_t den;
} Fraction;

typedef struct {
    Fraction* fractions;
    size_t length;
} FareyResult;

FareyResult farey_sequence(uint64_t n);
void free_farey(FareyResult* result);

// Utility functions
uint64_t lcm(uint64_t a, uint64_t b);

#endif // INTERSTELLAR_EUCLIDEAN_H
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Histoire de l'Algorithme d'Euclide

L'algorithme d'Euclide est decrit dans le Livre VII des *Elements* d'Euclide (~300 av. J.-C.), ce qui en fait l'un des plus anciens algorithmes non-triviaux encore utilises aujourd'hui — plus de 2300 ans d'histoire !

Le theoreme de Bezout (1766) et l'algorithme etendu ont ete formalises par Etienne Bezout, mais l'idee etait deja connue des mathematiciens indiens du VIe siecle comme Aryabhata qui l'utilisait pour resoudre des equations diophantiennes.

### 2.2 Le Theoreme des Restes Chinois

Le CRT tire son nom du mathematicien chinois Sun Zi (IIIe siecle) qui l'utilisait pour compter les soldats : "Quand on compte par 3, il reste 2; par 5, il reste 3; par 7, il reste 2. Combien y a-t-il de soldats ?" La reponse est 23 (modulo 105).

### 2.3 Applications Modernes

- **RSA (Cryptographie)** : L'inverse modulaire est fondamental pour le dechiffrement RSA
- **Codes correcteurs d'erreurs** : Le CRT permet la reconstruction de donnees partielles
- **Calculs astronomiques** : Le GCD/LCM synchronise les orbites planetaires
- **GPS** : Les satellites utilisent des algorithmes similaires pour la triangulation

---

### 2.5 DANS LA VRAIE VIE

| Metier | Utilisation | Cas d'Usage |
|--------|-------------|-------------|
| **Cryptographe** | RSA, Diffie-Hellman | L'inverse modulaire est au coeur de RSA : d = e^(-1) mod φ(n) |
| **Ingenieur Spatial** | Synchronisation orbitale | Calcul des fenetres de lancement (GCD des periodes) |
| **Developpeur Blockchain** | Hash et signatures | ECDSA utilise l'inverse modulaire sur courbes elliptiques |
| **Data Scientist** | Echantillonnage | CRT pour generer des nombres pseudo-aleatoires |
| **DevOps** | Scheduling | GCD pour determiner les intervalles de synchronisation |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash (Rust)

```bash
$ ls
src/lib.rs  Cargo.toml  tests/

$ cargo build --release
   Compiling interstellar_euclidean v0.1.0
    Finished release [optimized] target(s) in 0.42s

$ cargo test
running 12 tests
test test_gcd_basic ... ok
test test_gcd_zero ... ok
test test_extended_gcd ... ok
test test_mod_inverse_exists ... ok
test test_mod_inverse_none ... ok
test test_chinese_remainder ... ok
test test_euler_phi ... ok
test test_farey_sequence ... ok
test test_lcm ... ok
test test_gcd_multiple ... ok
test test_edge_cases ... ok
test test_large_numbers ... ok

test result: ok. 12 passed; 0 failed; 0 ignored
```

### 3.0.1 Session bash (C)

```bash
$ ls
interstellar_euclidean.c  interstellar_euclidean.h  main.c

$ gcc -Wall -Wextra -Werror -std=c17 -O2 interstellar_euclidean.c main.c -o test

$ ./test
[GCD] gcd(48, 18) = 6 ... OK
[GCD] gcd(0, 5) = 5 ... OK
[EGCD] extended_gcd(35, 15) = (5, 1, -2) ... OK
[MODINV] mod_inverse(3, 7) = 5 ... OK
[MODINV] mod_inverse(6, 9) = None ... OK
[CRT] chinese_remainder([2,3,2], [3,5,7]) = 23 ... OK
[PHI] euler_phi(12) = 4 ... OK
[FAREY] farey_sequence(3) correct ... OK
All tests passed!
```

---

### ⚡ SECTION 3.1 : BONUS STANDARD (OPTIONNEL)

**Difficulte Bonus :**
★★★★★★★☆☆☆ (7/10)

**Recompense :**
XP ×2

**Time Complexity attendue :**
O(log(min(a,b))) avec operations bitwise uniquement

**Space Complexity attendue :**
O(1) iteratif

**Domaines Bonus :**
`CPU, ASM`

#### 3.1.1 Consigne Bonus

**🔧 CONTEXTE : Le Mode Basse Energie de TARS**

Apres des annees a orbiter autour de Gargantua, TARS fonctionne en mode basse energie. Son processeur ne peut plus effectuer de divisions — seulement des operations bitwise (AND, OR, XOR, shifts). Cooper doit reprogrammer l'algorithme GCD en utilisant uniquement ces operations.

*"Humor setting: 75%."* — TARS

**Ta mission :**

Implementer `binary_gcd(a, b)` — l'algorithme de Stein (1967) qui calcule le GCD sans division, en utilisant uniquement :
- Comparaisons
- Soustractions
- Bit shifts (>> et <<)
- Test de parite (& 1)

**Entree :**
- `a`, `b` : Entiers positifs (0 ≤ a, b ≤ 10^18)

**Sortie :**
- `gcd(a, b)` calcule sans division

**Algorithme de Stein :**
```
1. gcd(0, v) = v, gcd(u, 0) = u
2. Si u et v sont pairs : gcd(u, v) = 2 × gcd(u/2, v/2)
3. Si u est pair et v impair : gcd(u, v) = gcd(u/2, v)
4. Si u est impair et v pair : gcd(u, v) = gcd(u, v/2)
5. Si u et v sont impairs : gcd(u, v) = gcd(|u-v|/2, min(u,v))
```

**Contraintes :**
┌─────────────────────────────────────────┐
│  Aucune division (/, %) autorisee       │
│  Uniquement &, |, ^, >>, <<, -, <, >    │
│  Temps : O(log(max(a,b)))               │
│  Espace : O(1)                          │
└─────────────────────────────────────────┘

**Exemples :**

| Appel | Retour | Verification |
|-------|--------|--------------|
| `binary_gcd(48, 18)` | `6` | Identique a gcd() |
| `binary_gcd(1071, 462)` | `21` | Sans aucune division |
| `binary_gcd(2^60, 2^40)` | `2^40` | Efficace sur grands nombres |

#### 3.1.2 Prototype Bonus

```rust
/// Binary GCD (Stein's algorithm) - no division, only bitwise ops
pub fn binary_gcd(a: u64, b: u64) -> u64;
```

```c
// Binary GCD without division
uint64_t binary_gcd(uint64_t a, uint64_t b);
```

#### 3.1.3 Ce qui change par rapport a l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Operations | Division, modulo | Bitwise seulement |
| Complexite | O(log(min)) avec div | O(log(max)) sans div |
| Hardware | Standard | Optimise low-level |
| Applications | General | Embedded, crypto |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette (tableau des tests)

| Test | Input | Expected | Points | Piege |
|------|-------|----------|--------|-------|
| `gcd_basic` | (48, 18) | 6 | 2 | Non |
| `gcd_zero_a` | (0, 5) | 5 | 2 | Oui - cas limite |
| `gcd_zero_b` | (5, 0) | 5 | 2 | Oui - cas limite |
| `gcd_coprime` | (17, 23) | 1 | 2 | Non |
| `gcd_same` | (42, 42) | 42 | 2 | Non |
| `egcd_basic` | (35, 15) | (5, 1, -2) | 3 | Verifier Bezout |
| `egcd_swap` | (15, 35) | (5, -2, 1) | 3 | Ordre inverse |
| `egcd_zero` | (17, 0) | (17, 1, 0) | 3 | Oui - cas limite |
| `modinv_exists` | (3, 7) | Some(5) | 3 | Non |
| `modinv_none` | (6, 9) | None | 3 | Oui - gcd != 1 |
| `modinv_one` | (1, 100) | Some(1) | 2 | Cas trivial |
| `crt_basic` | ([2,3,2], [3,5,7]) | (23, 105) | 4 | Non |
| `crt_two` | ([1,2], [3,5]) | (7, 15) | 4 | Non |
| `crt_invalid` | ([0,0], [4,6]) | None | 4 | Oui - non coprime |
| `phi_small` | 6 | 2 | 2 | Non |
| `phi_prime` | 7 | 6 | 2 | φ(p) = p-1 |
| `phi_one` | 1 | 1 | 2 | Cas limite |
| `phi_power` | 8 | 4 | 2 | φ(2^k) = 2^(k-1) |
| `farey_1` | 1 | [(0,1),(1,1)] | 3 | Non |
| `farey_3` | 3 | 5 fractions | 3 | Non |
| `binary_gcd` | (48, 18) | 6 | 5 | Bonus |

**Total : 100 points**

---

### 4.2 main.c de test

```c
#include <stdio.h>
#include <assert.h>
#include <stdbool.h>
#include "interstellar_euclidean.h"

#define TEST(name, expr) do { \
    printf("[TEST] %s ... ", name); \
    if (expr) { printf("OK\n"); } \
    else { printf("FAIL\n"); failures++; } \
} while(0)

int main(void) {
    int failures = 0;

    // GCD tests
    TEST("gcd(48, 18) == 6", gcd(48, 18) == 6);
    TEST("gcd(0, 5) == 5", gcd(0, 5) == 5);
    TEST("gcd(5, 0) == 5", gcd(5, 0) == 5);
    TEST("gcd(17, 23) == 1", gcd(17, 23) == 1);
    TEST("gcd(1071, 462) == 21", gcd(1071, 462) == 21);

    // Extended GCD tests
    ExtendedGcdResult r1 = extended_gcd(35, 15);
    TEST("extended_gcd(35, 15).gcd == 5", r1.gcd == 5);
    TEST("extended_gcd(35, 15) Bezout", 35 * r1.x + 15 * r1.y == 5);

    ExtendedGcdResult r2 = extended_gcd(161, 28);
    TEST("extended_gcd(161, 28).gcd == 7", r2.gcd == 7);
    TEST("extended_gcd(161, 28) Bezout", 161 * r2.x + 28 * r2.y == 7);

    // Modular inverse tests
    ModInverseResult inv1 = mod_inverse(3, 7);
    TEST("mod_inverse(3, 7) exists", inv1.exists == true);
    TEST("mod_inverse(3, 7) == 5", inv1.value == 5);
    TEST("3 * 5 mod 7 == 1", (3 * inv1.value) % 7 == 1);

    ModInverseResult inv2 = mod_inverse(6, 9);
    TEST("mod_inverse(6, 9) == None", inv2.exists == false);

    ModInverseResult inv3 = mod_inverse(17, 43);
    TEST("mod_inverse(17, 43) exists", inv3.exists == true);
    TEST("17 * 38 mod 43 == 1", (17 * inv3.value) % 43 == 1);

    // Chinese Remainder Theorem tests
    int64_t r[] = {2, 3, 2};
    int64_t m[] = {3, 5, 7};
    CrtResult crt1 = chinese_remainder(r, m, 3);
    TEST("CRT([2,3,2], [3,5,7]) valid", crt1.valid == true);
    TEST("CRT result x == 23", crt1.x == 23);
    TEST("CRT result M == 105", crt1.m == 105);

    int64_t r2_arr[] = {0, 0};
    int64_t m2_arr[] = {4, 6};
    CrtResult crt2 = chinese_remainder(r2_arr, m2_arr, 2);
    TEST("CRT([0,0], [4,6]) invalid", crt2.valid == false);

    // Euler phi tests
    TEST("euler_phi(1) == 1", euler_phi(1) == 1);
    TEST("euler_phi(2) == 1", euler_phi(2) == 1);
    TEST("euler_phi(6) == 2", euler_phi(6) == 2);
    TEST("euler_phi(10) == 4", euler_phi(10) == 4);
    TEST("euler_phi(12) == 4", euler_phi(12) == 4);
    TEST("euler_phi(7) == 6", euler_phi(7) == 6);

    // Farey sequence tests
    FareyResult f3 = farey_sequence(3);
    TEST("farey_sequence(3).length == 5", f3.length == 5);
    TEST("F3[0] == 0/1", f3.fractions[0].num == 0 && f3.fractions[0].den == 1);
    TEST("F3[1] == 1/3", f3.fractions[1].num == 1 && f3.fractions[1].den == 3);
    TEST("F3[2] == 1/2", f3.fractions[2].num == 1 && f3.fractions[2].den == 2);
    TEST("F3[3] == 2/3", f3.fractions[3].num == 2 && f3.fractions[3].den == 3);
    TEST("F3[4] == 1/1", f3.fractions[4].num == 1 && f3.fractions[4].den == 1);
    free_farey(&f3);

    // Summary
    printf("\n=== RESULTS ===\n");
    if (failures == 0) {
        printf("All tests passed!\n");
        return 0;
    } else {
        printf("%d test(s) failed.\n", failures);
        return 1;
    }
}
```

---

### 4.3 Solution de reference (Rust)

```rust
/// GCD using Euclidean algorithm
pub fn gcd(a: u64, b: u64) -> u64 {
    if b == 0 {
        a
    } else {
        gcd(b, a % b)
    }
}

/// Extended Euclidean algorithm
pub fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    if b == 0 {
        (a.abs(), if a >= 0 { 1 } else { -1 }, 0)
    } else {
        let (g, x, y) = extended_gcd(b, a % b);
        (g, y, x - (a / b) * y)
    }
}

/// Modular multiplicative inverse
pub fn mod_inverse(a: i64, m: i64) -> Option<i64> {
    let (g, x, _) = extended_gcd(a, m);
    if g != 1 {
        None
    } else {
        Some(((x % m) + m) % m)
    }
}

/// Chinese Remainder Theorem
pub fn chinese_remainder(remainders: &[i64], moduli: &[i64]) -> Option<(i64, i64)> {
    if remainders.len() != moduli.len() || remainders.is_empty() {
        return None;
    }

    // Check pairwise coprimality
    for i in 0..moduli.len() {
        for j in (i + 1)..moduli.len() {
            if gcd(moduli[i].unsigned_abs(), moduli[j].unsigned_abs()) != 1 {
                return None;
            }
        }
    }

    let m: i64 = moduli.iter().product();
    let mut x = 0i64;

    for i in 0..remainders.len() {
        let mi = m / moduli[i];
        let inv = mod_inverse(mi, moduli[i])?;
        x = (x + remainders[i] * mi * inv) % m;
    }

    Some(((x % m + m) % m, m))
}

/// Euler's totient function
pub fn euler_phi(n: u64) -> u64 {
    if n <= 1 {
        return n;
    }

    let mut result = n;
    let mut temp = n;
    let mut p = 2u64;

    while p * p <= temp {
        if temp % p == 0 {
            while temp % p == 0 {
                temp /= p;
            }
            result -= result / p;
        }
        p += 1;
    }

    if temp > 1 {
        result -= result / temp;
    }

    result
}

/// Farey sequence of order n
pub fn farey_sequence(n: u64) -> Vec<(u64, u64)> {
    let mut result = vec![(0, 1)];
    let (mut a, mut b, mut c, mut d) = (0u64, 1u64, 1u64, n);

    while c <= n {
        let k = (n + b) / d;
        let (new_a, new_b, new_c, new_d) = (c, d, k * c - a, k * d - b);
        result.push((new_a, new_b));
        a = new_a;
        b = new_b;
        c = new_c;
        d = new_d;
    }

    result
}

/// LCM using GCD
pub fn lcm(a: u64, b: u64) -> u64 {
    if a == 0 || b == 0 {
        0
    } else {
        a / gcd(a, b) * b
    }
}

/// GCD of multiple numbers
pub fn gcd_multiple(nums: &[u64]) -> u64 {
    nums.iter().copied().reduce(gcd).unwrap_or(0)
}
```

---

### 4.4 Solutions alternatives acceptees

#### Alternative 1 : GCD iteratif

```rust
pub fn gcd(mut a: u64, mut b: u64) -> u64 {
    while b != 0 {
        let temp = b;
        b = a % b;
        a = temp;
    }
    a
}
```

#### Alternative 2 : Extended GCD iteratif

```rust
pub fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    let (mut old_r, mut r) = (a, b);
    let (mut old_s, mut s) = (1i64, 0i64);
    let (mut old_t, mut t) = (0i64, 1i64);

    while r != 0 {
        let quotient = old_r / r;
        (old_r, r) = (r, old_r - quotient * r);
        (old_s, s) = (s, old_s - quotient * s);
        (old_t, t) = (t, old_t - quotient * t);
    }

    (old_r.abs(), if old_r >= 0 { old_s } else { -old_s },
     if old_r >= 0 { old_t } else { -old_t })
}
```

#### Alternative 3 : Euler phi avec sieve

```rust
pub fn euler_phi(n: u64) -> u64 {
    let n = n as usize;
    if n <= 1 { return n as u64; }

    let mut phi: Vec<u64> = (0..=n).map(|i| i as u64).collect();

    for i in 2..=n {
        if phi[i] == i as u64 {
            for j in (i..=n).step_by(i) {
                phi[j] -= phi[j] / i as u64;
            }
        }
    }

    phi[n]
}
```

---

### 4.5 Solutions refusees (avec explications)

#### Refusee 1 : GCD sans cas de base correct

```rust
// REFUSE : Retourne 0 au lieu de a quand b == 0
pub fn gcd(a: u64, b: u64) -> u64 {
    if b == 0 {
        return 0;  // ERREUR: devrait retourner a
    }
    gcd(b, a % b)
}
```
**Pourquoi c'est faux :** `gcd(5, 0)` doit retourner 5, pas 0. Par definition, gcd(a, 0) = a.

#### Refusee 2 : Mod inverse sans verification gcd

```rust
// REFUSE : Ne verifie pas si l'inverse existe
pub fn mod_inverse(a: i64, m: i64) -> Option<i64> {
    let (_, x, _) = extended_gcd(a, m);
    Some(((x % m) + m) % m)  // Retourne toujours Some, meme si gcd != 1
}
```
**Pourquoi c'est faux :** Si gcd(a, m) != 1, l'inverse modulaire n'existe pas.

#### Refusee 3 : CRT sans verification de coprimarite

```rust
// REFUSE : Ne verifie pas que les moduli sont copremiers
pub fn chinese_remainder(r: &[i64], m: &[i64]) -> Option<(i64, i64)> {
    let total: i64 = m.iter().product();
    let mut x = 0;
    for i in 0..r.len() {
        let mi = total / m[i];
        let inv = mod_inverse(mi, m[i]).unwrap();  // Peut panic!
        x += r[i] * mi * inv;
    }
    Some((x % total, total))
}
```
**Pourquoi c'est faux :** Si les moduli ne sont pas copremiers deux a deux, mod_inverse retournera None et le unwrap paniquera.

---

### 4.6 Solution bonus de reference (COMPLETE)

```rust
/// Binary GCD (Stein's algorithm) - no division, only bitwise ops
pub fn binary_gcd(mut u: u64, mut v: u64) -> u64 {
    // Base cases
    if u == 0 { return v; }
    if v == 0 { return u; }

    // Find common factors of 2
    let shift = (u | v).trailing_zeros();

    // Remove factors of 2 from u
    u >>= u.trailing_zeros();

    loop {
        // Remove factors of 2 from v
        v >>= v.trailing_zeros();

        // Ensure u <= v
        if u > v {
            std::mem::swap(&mut u, &mut v);
        }

        // v = v - u (both are odd, so v - u is even)
        v -= u;

        if v == 0 {
            break;
        }
    }

    u << shift
}
```

**Version C :**

```c
uint64_t binary_gcd(uint64_t u, uint64_t v) {
    if (u == 0) return v;
    if (v == 0) return u;

    // Find common factors of 2
    int shift = 0;
    while (((u | v) & 1) == 0) {
        u >>= 1;
        v >>= 1;
        shift++;
    }

    // Remove remaining factors of 2 from u
    while ((u & 1) == 0) {
        u >>= 1;
    }

    do {
        // Remove factors of 2 from v
        while ((v & 1) == 0) {
            v >>= 1;
        }

        // Ensure u <= v
        if (u > v) {
            uint64_t temp = u;
            u = v;
            v = temp;
        }

        v = v - u;
    } while (v != 0);

    return u << shift;
}
```

---

### 4.7 Solutions alternatives bonus (COMPLETES)

#### Alternative Bonus 1 : Version avec comptage de bits explicite

```rust
pub fn binary_gcd(mut u: u64, mut v: u64) -> u64 {
    if u == 0 { return v; }
    if v == 0 { return u; }

    let mut shift = 0u32;

    // Compter les facteurs de 2 communs
    while (u & 1) == 0 && (v & 1) == 0 {
        u >>= 1;
        v >>= 1;
        shift += 1;
    }

    // Supprimer les facteurs de 2 restants de u
    while (u & 1) == 0 {
        u >>= 1;
    }

    loop {
        while (v & 1) == 0 {
            v >>= 1;
        }

        if u > v {
            let temp = u;
            u = v;
            v = temp;
        }

        v -= u;

        if v == 0 {
            return u << shift;
        }
    }
}
```

---

### 4.8 Solutions refusees bonus (COMPLETES)

#### Refusee Bonus 1 : Utilise modulo

```rust
// REFUSE : Utilise l'operateur % qui est une division deguisee
pub fn binary_gcd(mut a: u64, mut b: u64) -> u64 {
    if b == 0 { return a; }
    binary_gcd(b, a % b)  // ERREUR: % interdit dans le bonus
}
```
**Pourquoi c'est faux :** Le bonus exige de ne pas utiliser de division, et `%` est une division.

#### Refusee Bonus 2 : Oublie de multiplier par les facteurs de 2

```rust
// REFUSE : Ne restaure pas les facteurs de 2 communs
pub fn binary_gcd(mut u: u64, mut v: u64) -> u64 {
    if u == 0 { return v; }
    if v == 0 { return u; }

    while (u & 1) == 0 { u >>= 1; }

    loop {
        while (v & 1) == 0 { v >>= 1; }
        if u > v { std::mem::swap(&mut u, &mut v); }
        v -= u;
        if v == 0 { return u; }  // ERREUR: oublie << shift
    }
}
```
**Pourquoi c'est faux :** Si u et v partagent des facteurs de 2, le resultat doit les inclure.

---

### 4.9 spec.json (ENGINE v22.1 — FORMAT STRICT)

```json
{
  "name": "interstellar_euclidean",
  "language": "rust",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthese (concepts d-l combines)",
  "tags": ["number_theory", "gcd", "euclidean", "modular", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "gcd",
    "prototype": "pub fn gcd(a: u64, b: u64) -> u64",
    "return_type": "u64",
    "parameters": [
      {"name": "a", "type": "u64"},
      {"name": "b", "type": "u64"}
    ]
  },

  "driver": {
    "reference": "pub fn ref_gcd(a: u64, b: u64) -> u64 { if b == 0 { a } else { ref_gcd(b, a % b) } }",

    "edge_cases": [
      {
        "name": "gcd_basic",
        "args": [48, 18],
        "expected": 6,
        "is_trap": false
      },
      {
        "name": "gcd_zero_a",
        "args": [0, 5],
        "expected": 5,
        "is_trap": true,
        "trap_explanation": "gcd(0, n) = n, not 0"
      },
      {
        "name": "gcd_zero_b",
        "args": [5, 0],
        "expected": 5,
        "is_trap": true,
        "trap_explanation": "gcd(n, 0) = n, not 0"
      },
      {
        "name": "gcd_coprime",
        "args": [17, 23],
        "expected": 1,
        "is_trap": false
      },
      {
        "name": "gcd_same",
        "args": [42, 42],
        "expected": 42,
        "is_trap": false
      },
      {
        "name": "gcd_one",
        "args": [1, 1000000],
        "expected": 1,
        "is_trap": false
      },
      {
        "name": "gcd_large",
        "args": [1071, 462],
        "expected": 21,
        "is_trap": false
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 1000,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {"min": 0, "max": 1000000}
        },
        {
          "type": "int",
          "param_index": 1,
          "params": {"min": 0, "max": 1000000}
        }
      ]
    }
  },

  "additional_functions": [
    {
      "name": "extended_gcd",
      "prototype": "pub fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64)",
      "edge_cases": [
        {"name": "egcd_35_15", "args": [35, 15], "expected": [5, 1, -2]},
        {"name": "egcd_161_28", "args": [161, 28], "expected": [7, -1, 6]},
        {"name": "egcd_zero", "args": [17, 0], "expected": [17, 1, 0]}
      ]
    },
    {
      "name": "mod_inverse",
      "prototype": "pub fn mod_inverse(a: i64, m: i64) -> Option<i64>",
      "edge_cases": [
        {"name": "inv_3_7", "args": [3, 7], "expected": "Some(5)"},
        {"name": "inv_6_9", "args": [6, 9], "expected": "None"},
        {"name": "inv_17_43", "args": [17, 43], "expected": "Some(38)"}
      ]
    },
    {
      "name": "euler_phi",
      "prototype": "pub fn euler_phi(n: u64) -> u64",
      "edge_cases": [
        {"name": "phi_1", "args": [1], "expected": 1},
        {"name": "phi_6", "args": [6], "expected": 2},
        {"name": "phi_12", "args": [12], "expected": 4},
        {"name": "phi_prime", "args": [7], "expected": 6}
      ]
    }
  ],

  "norm": {
    "allowed_functions": ["abs", "swap"],
    "forbidden_functions": [],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes (minimum 5)

#### Mutant A (Boundary) : GCD retourne 0 au lieu de a

```rust
/* Mutant A (Boundary) : Cas de base incorrect */
pub fn gcd(a: u64, b: u64) -> u64 {
    if b == 0 {
        return 0;  // BUG: devrait retourner a
    }
    gcd(b, a % b)
}
// Pourquoi c'est faux : gcd(5, 0) retourne 0 au lieu de 5
// Ce qui etait pense : "Si b est 0, le GCD est 0"
// Realite : gcd(a, 0) = a par definition
```

#### Mutant B (Safety) : Extended GCD ne gere pas les negatifs

```rust
/* Mutant B (Safety) : Pas de gestion du signe */
pub fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    if b == 0 {
        return (a, 1, 0);  // BUG: a pourrait etre negatif
    }
    let (g, x, y) = extended_gcd(b, a % b);
    (g, y, x - (a / b) * y)
}
// Pourquoi c'est faux : extended_gcd(-35, 15) retourne (-5, ...) au lieu de (5, ...)
// Ce qui etait pense : "Le GCD conserve le signe"
// Realite : Le GCD est toujours positif
```

#### Mutant C (Logic) : Mod inverse sans verification gcd == 1

```rust
/* Mutant C (Logic) : Retourne un inverse meme si inexistant */
pub fn mod_inverse(a: i64, m: i64) -> Option<i64> {
    let (_, x, _) = extended_gcd(a, m);
    Some(((x % m) + m) % m)  // BUG: devrait verifier gcd == 1
}
// Pourquoi c'est faux : mod_inverse(6, 9) retourne Some(x) au lieu de None
// Ce qui etait pense : "L'inverse existe toujours"
// Realite : L'inverse n'existe que si gcd(a, m) = 1
```

#### Mutant D (Logic) : CRT oublie l'inverse modulaire

```rust
/* Mutant D (Logic) : Formule CRT incomplete */
pub fn chinese_remainder(r: &[i64], m: &[i64]) -> Option<(i64, i64)> {
    let total: i64 = m.iter().product();
    let mut x = 0;
    for i in 0..r.len() {
        let mi = total / m[i];
        x += r[i] * mi;  // BUG: manque * mod_inverse(mi, m[i])
    }
    Some((x % total, total))
}
// Pourquoi c'est faux : CRT([2,3,2], [3,5,7]) donne mauvais resultat
// Ce qui etait pense : "La formule est juste x = sum(r[i] * M/m[i])"
// Realite : Il faut multiplier par l'inverse modulaire
```

#### Mutant E (Return) : Euler phi oublie le dernier facteur premier

```rust
/* Mutant E (Return) : Oublie le facteur premier > sqrt(n) */
pub fn euler_phi(n: u64) -> u64 {
    if n <= 1 { return n; }
    let mut result = n;
    let mut temp = n;
    let mut p = 2u64;

    while p * p <= temp {
        if temp % p == 0 {
            while temp % p == 0 {
                temp /= p;
            }
            result -= result / p;
        }
        p += 1;
    }
    // BUG: oublie de traiter temp > 1
    result
}
// Pourquoi c'est faux : euler_phi(6) = 6 * (1 - 1/2) = 3, mais devrait etre 2
// Ce qui etait pense : "Tous les facteurs premiers sont <= sqrt(n)"
// Realite : Si temp > 1 apres la boucle, c'est un facteur premier
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

1. **L'algorithme d'Euclide** — Le plus ancien algorithme non-trivial, base sur la propriete gcd(a, b) = gcd(b, a mod b)
2. **L'identite de Bezout** — Existence de x, y tels que ax + by = gcd(a, b)
3. **L'inverse modulaire** — Fondamental en cryptographie (RSA, ECC)
4. **Le Theoreme des Restes Chinois** — Reconstruction d'un nombre a partir de ses restes
5. **La fonction phi d'Euler** — Compte les copremiers, liee au petit theoreme de Fermat
6. **Les suites de Farey** — Fractions irreductibles, liees a la geometrie et la theorie des nombres

---

### 5.2 LDA — Traduction litterale en francais (MAJUSCULES)

#### GCD (Algorithme d'Euclide)

```
FONCTION gcd QUI RETOURNE UN ENTIER ET PREND EN PARAMETRES a ET b QUI SONT DES ENTIERS NON SIGNES
DEBUT FONCTION
    SI b EST EGAL A 0 ALORS
        RETOURNER LA VALEUR DE a
    FIN SI
    RETOURNER gcd(b, a MODULO b)
FIN FONCTION
```

#### Extended GCD

```
FONCTION extended_gcd QUI RETOURNE UN TRIPLET (gcd, x, y) ET PREND EN PARAMETRES a ET b QUI SONT DES ENTIERS SIGNES
DEBUT FONCTION
    SI b EST EGAL A 0 ALORS
        RETOURNER (VALEUR ABSOLUE DE a, SI a EST POSITIF ALORS 1 SINON -1, 0)
    FIN SI

    DECLARER (g, x, y) COMME RESULTAT DE extended_gcd(b, a MODULO b)
    RETOURNER (g, y, x MOINS (a DIVISE PAR b) MULTIPLIE PAR y)
FIN FONCTION
```

#### Mod Inverse

```
FONCTION mod_inverse QUI RETOURNE UN OPTIONNEL ENTIER ET PREND EN PARAMETRES a ET m QUI SONT DES ENTIERS
DEBUT FONCTION
    DECLARER (g, x, _) COMME RESULTAT DE extended_gcd(a, m)

    SI g EST DIFFERENT DE 1 ALORS
        RETOURNER RIEN (None)
    FIN SI

    RETOURNER QUELQUE CHOSE ((x MODULO m) PLUS m) MODULO m
FIN FONCTION
```

---

### 5.2.2 LDA Style Academique (Universite Francaise)

```
ALGORITHME : Plus Grand Commun Diviseur (Euclide)
ENTREES : a, b entiers naturels
SORTIE : gcd(a, b) entier naturel

DEBUT
    TANT QUE b ≠ 0 FAIRE
        r <- a MOD b
        a <- b
        b <- r
    FIN TANT QUE
    RETOURNER a
FIN
```

---

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : GCD Euclidean
---
1. SI b = 0 :
   RETOURNER a

2. SINON :
   a. CALCULER reste = a MOD b
   b. RETOURNER GCD(b, reste)
```

---

### 5.2.3.1 Logique de Garde (Fail Fast)

```
FONCTION : mod_inverse(a, m)
---
INIT resultat = None

1. CALCULER (gcd, x, _) = extended_gcd(a, m)

2. VERIFIER si gcd != 1 :
   |
   |-- RETOURNER None (pas d'inverse)

3. CALCULER inverse = ((x % m) + m) % m

4. RETOURNER Some(inverse)
```

---

### 5.3 Visualisation ASCII (adaptee au sujet)

#### Algorithme d'Euclide — Trace visuelle

```
GCD(48, 18):

Etape 1:  48 = 2 × 18 + 12
          │      │     │
          a      b    reste

Etape 2:  18 = 1 × 12 + 6
          │      │     │
          a      b    reste

Etape 3:  12 = 2 × 6 + 0
          │      │   │
          a      b   reste = 0 → STOP

Resultat: GCD = 6
```

#### Extended GCD — Coefficients de Bezout

```
extended_gcd(35, 15):

     a    b   |   x    y   |  Verification
   ─────────────────────────────────────────
    35   15   |   1    0   |
    15    5   |  -2    1   |
     5    0   |   1   -2   |  35×1 + 15×(-2) = 5 ✓
              |            |
              └── Coefficients de Bezout
```

#### Theoreme des Restes Chinois

```
CRT: x ≡ 2 (mod 3)
     x ≡ 3 (mod 5)
     x ≡ 2 (mod 7)

Planete Miller (mod 3):  x = ..., 2, 5, 8, 11, 14, 17, 20, 23, ...
Planete Mann (mod 5):    x = ..., 3, 8, 13, 18, 23, ...
Planete Edmund (mod 7):  x = ..., 2, 9, 16, 23, ...
                                              ↑
                                   Intersection = 23

Solution: x ≡ 23 (mod 105)
```

#### Fonction Phi d'Euler

```
φ(12) = ?

  12 = 2² × 3

  Nombres de 1 a 12:  1  2  3  4  5  6  7  8  9 10 11 12
                      ↑     ×  ×  ↑  ×  ↑  ×  ×  ×  ↑  ×

  Copremiers avec 12: {1, 5, 7, 11}

  φ(12) = 12 × (1 - 1/2) × (1 - 1/3) = 12 × 1/2 × 2/3 = 4 ✓
```

---

### 5.4 Les pieges en detail

#### Piege 1 : gcd(a, 0) = a, pas 0

Le cas de base est souvent mal compris. Par definition mathematique, le GCD de n'importe quel nombre avec 0 est ce nombre lui-meme.

```rust
// CORRECT
if b == 0 { return a; }

// INCORRECT (erreur courante)
if b == 0 { return 0; }
```

#### Piege 2 : L'inverse modulaire n'existe pas toujours

L'inverse de a mod m n'existe que si gcd(a, m) = 1. Oublier cette verification mene a des resultats incorrects.

```rust
// CORRECT
let (g, x, _) = extended_gcd(a, m);
if g != 1 { return None; }

// INCORRECT
let (_, x, _) = extended_gcd(a, m);
return Some(x);  // Peut retourner un "faux" inverse
```

#### Piege 3 : Euler phi oublie le dernier facteur premier

Si n = 6 = 2 × 3, apres avoir traite p=2, il reste temp=3 > sqrt(6)≈2.45. Ce facteur doit etre traite.

```rust
// CORRECT
if temp > 1 {
    result -= result / temp;
}

// INCORRECT (oublie ce cas)
// Retourne result sans ajuster pour le dernier facteur
```

#### Piege 4 : CRT avec moduli non copremiers

Le CRT standard ne fonctionne que si les moduli sont copremiers deux a deux.

```rust
// CORRECT
for i in 0..moduli.len() {
    for j in (i+1)..moduli.len() {
        if gcd(moduli[i], moduli[j]) != 1 {
            return None;
        }
    }
}

// INCORRECT (pas de verification)
```

---

### 5.5 Cours Complet (VRAI cours, pas un resume)

#### 5.5.1 L'Algorithme d'Euclide

**Theoreme fondamental :**
Pour tous entiers a, b avec b ≠ 0 : gcd(a, b) = gcd(b, a mod b)

**Preuve :**
Soit d = gcd(a, b). Alors d | a et d | b.
Puisque a mod b = a - (a/b)×b, on a d | (a mod b).
Donc d divise a et (a mod b), donc d ≤ gcd(b, a mod b).

Reciproquement, soit e = gcd(b, a mod b).
Puisque a = (a/b)×b + (a mod b), e | a.
Donc e ≤ gcd(a, b) = d.

Conclusion : d = e. CQFD.

**Complexite :**
O(log(min(a, b))) iterations au maximum (theoreme de Lame).

---

#### 5.5.2 L'Algorithme d'Euclide Etendu

L'idee est de "remonter" les calculs pour trouver les coefficients de Bezout.

**Recurrence :**
- Si b = 0 : gcd(a, 0) = a = a×1 + 0×0
- Sinon : Si gcd(b, a mod b) = b×x' + (a mod b)×y'
  - Alors gcd(a, b) = a×y' + b×(x' - (a/b)×y')

**Exemple detaille :**
```
extended_gcd(35, 15):

Descente:
  35 = 2×15 + 5
  15 = 3×5 + 0    → gcd = 5

Remontee:
  5 = 35 - 2×15
  5 = 35×1 + 15×(-2)

Donc x=1, y=-2, et 35×1 + 15×(-2) = 35 - 30 = 5 ✓
```

---

#### 5.5.3 L'Inverse Modulaire

**Definition :**
L'inverse de a modulo m est l'entier x tel que a×x ≡ 1 (mod m).

**Condition d'existence :**
x existe ⟺ gcd(a, m) = 1

**Calcul via Bezout :**
Si ax + my = 1, alors ax ≡ 1 (mod m), donc x est l'inverse.

**Application : RSA**
Dans RSA, on choisit e tel que gcd(e, φ(n)) = 1, puis d = e^(-1) mod φ(n).

---

#### 5.5.4 Le Theoreme des Restes Chinois

**Enonce :**
Soient m1, m2, ..., mk des entiers deux a deux copremiers.
Le systeme :
  x ≡ a1 (mod m1)
  x ≡ a2 (mod m2)
  ...
  x ≡ ak (mod mk)

A une solution unique modulo M = m1×m2×...×mk.

**Construction de la solution :**
Pour chaque i :
  - Mi = M / mi
  - yi = Mi^(-1) mod mi

Solution : x = Σ ai × Mi × yi (mod M)

---

#### 5.5.5 La Fonction Phi d'Euler

**Definition :**
φ(n) = |{k : 1 ≤ k ≤ n, gcd(k, n) = 1}|

**Proprietes :**
- φ(1) = 1
- φ(p) = p - 1 pour p premier
- φ(p^k) = p^(k-1) × (p - 1)
- φ(m×n) = φ(m) × φ(n) si gcd(m, n) = 1 (multiplicativite)

**Formule generale :**
φ(n) = n × Π(p|n) (1 - 1/p)

**Application : Petit theoreme de Fermat generalise**
a^φ(n) ≡ 1 (mod n) si gcd(a, n) = 1

---

### 5.6 Normes avec explications pedagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ pub fn gcd(a: u64, b: u64) -> u64 { if b==0 {a} else {gcd(b,a%b)} } │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ pub fn gcd(a: u64, b: u64) -> u64 {                             │
│     if b == 0 {                                                 │
│         a                                                       │
│     } else {                                                    │
│         gcd(b, a % b)                                           │
│     }                                                           │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Lisibilite : Structure claire avec indentation               │
│ • Debugging : Points d'arret faciles a placer                  │
│ • Maintenance : Modifications localisees                       │
│ • Review : Diff git clair                                       │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.7 Simulation avec trace d'execution

#### Trace : gcd(48, 18)

```
┌───────┬──────────────────────────────────┬──────┬──────┬─────────────────────┐
│ Etape │ Instruction                      │  a   │  b   │ Explication         │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   1   │ APPEL gcd(48, 18)                │  48  │  18  │ Debut               │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   2   │ b != 0 ? 18 != 0 → VRAI          │  48  │  18  │ On continue         │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   3   │ APPEL gcd(18, 48 % 18)           │  18  │  12  │ 48 mod 18 = 12      │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   4   │ b != 0 ? 12 != 0 → VRAI          │  18  │  12  │ On continue         │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   5   │ APPEL gcd(12, 18 % 12)           │  12  │   6  │ 18 mod 12 = 6       │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   6   │ b != 0 ? 6 != 0 → VRAI           │  12  │   6  │ On continue         │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   7   │ APPEL gcd(6, 12 % 6)             │   6  │   0  │ 12 mod 6 = 0        │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   8   │ b == 0 ? 0 == 0 → VRAI           │   6  │   0  │ Cas de base atteint │
├───────┼──────────────────────────────────┼──────┼──────┼─────────────────────┤
│   9   │ RETOURNER 6                      │   6  │   —  │ Resultat final      │
└───────┴──────────────────────────────────┴──────┴──────┴─────────────────────┘

Resultat : gcd(48, 18) = 6
```

---

### 5.8 Mnemotechniques (MEME obligatoire)

#### 🚀 MEME : "This is no time for caution" — Verifier les cas limites

![Interstellar Docking Scene](meme_no_time_for_caution.jpg)

Dans Interstellar, Cooper tente un docking impossible avec une station en rotation. Il pourrait echouer a tout moment — comme ton code sans verification des cas limites.

```rust
pub fn gcd(a: u64, b: u64) -> u64 {
    // 🚀 "This is no time for caution" - mais on verifie quand meme!
    if b == 0 {
        return a;  // Cas limite CRUCIAL
    }
    gcd(b, a % b)
}
```

---

#### ⏰ MEME : "One Hour = Seven Years" — La dilatation du modulo

![Miller's Planet Wave](meme_millers_planet.jpg)

Sur la planete de Miller, 1 heure = 7 ans terrestres. L'inverse modulaire "inverse" cette relation :

```rust
// Sur Miller : temps_terre = temps_miller × 7
// Inverse :   temps_miller = temps_terre × mod_inverse(7, periode_orbitale)
```

Si gcd(7, periode) != 1, pas d'inverse — comme si le temps etait irreversible.

---

#### 🌌 MEME : "TARS, get us coordinates" — Le CRT

![TARS Robot](meme_tars.jpg)

TARS calcule les coordonnees en combinant les informations de plusieurs planetes — exactement comme le CRT combine plusieurs congruences.

```rust
// Position dans la galaxie :
//   x ≡ 2 (mod 3)   // Signal de Miller
//   x ≡ 3 (mod 5)   // Signal de Mann
//   x ≡ 2 (mod 7)   // Signal d'Edmund
//
// TARS calcule : x = 23 (mod 105)
```

---

#### 💀 MEME : "Murphy's Law" — Ce qui peut mal tourner

La loi de Murphy dans le code : si tu ne verifies pas gcd == 1, l'inverse modulaire te trahira.

```rust
// Murphy (la scientifique) verifierait TOUJOURS
if gcd != 1 {
    return None;  // "Whatever can go wrong, will go wrong"
}
```

---

### 5.9 Applications pratiques

| Application | Fonction | Exemple Reel |
|-------------|----------|--------------|
| **Cryptographie RSA** | mod_inverse | d = e^(-1) mod φ(n) pour dechiffrement |
| **Reduction de fractions** | gcd | 48/18 → 8/3 via gcd(48,18)=6 |
| **Scheduling** | lcm | Synchroniser taches de periodes 4 et 6 → toutes les 12 unites |
| **Codes correcteurs** | CRT | Reconstruction de donnees partielles |
| **Generateurs PRNG** | euler_phi | Periode maximale = φ(m) pour un LCG |
| **Musique/Rythme** | gcd/lcm | Polymetre : superposition de rythmes |

---

## ⚠️ SECTION 6 : PIEGES — RECAPITULATIF

| Piege | Description | Solution |
|-------|-------------|----------|
| **gcd(a, 0)** | Retourner 0 au lieu de a | `if b == 0 { return a; }` |
| **Signe GCD** | GCD negatif pour entrees negatives | Toujours retourner `abs(gcd)` |
| **Inverse inexistant** | Ne pas verifier gcd(a, m) == 1 | `if g != 1 { return None; }` |
| **CRT non-coprime** | Pas de verification coprimarite | Verifier gcd deux a deux |
| **Phi dernier facteur** | Oublier le facteur > sqrt(n) | `if temp > 1 { result -= result/temp; }` |
| **Overflow** | Multiplication avant division | `a / gcd(a,b) * b` au lieu de `a * b / gcd` |

---

## 📝 SECTION 7 : QCM

### Question 1
Quelle est la valeur de gcd(0, 42) ?

A) 0
B) 1
C) 42
D) Indefini
E) -42
F) 21
G) Erreur
H) Infinity
I) NaN
J) 84

**Reponse : C**

---

### Question 2
Pour quels couples (a, m) l'inverse modulaire de a mod m existe-t-il ?

A) (6, 9) — gcd = 3
B) (3, 7) — gcd = 1
C) (4, 8) — gcd = 4
D) (15, 25) — gcd = 5
E) (17, 43) — gcd = 1
F) (2, 4) — gcd = 2
G) (1, 100) — gcd = 1
H) (10, 15) — gcd = 5
I) (7, 11) — gcd = 1
J) (12, 18) — gcd = 6

**Reponses correctes : B, E, G, I**

---

### Question 3
Que retourne extended_gcd(35, 15) ?

A) (5, 1, -2)
B) (5, -2, 1)
C) (5, 1, 2)
D) (5, 2, -1)
E) (15, 1, -2)
F) (35, 1, 0)
G) (1, 5, -2)
H) (5, 0, 1)
I) (-5, 1, -2)
J) (5, -1, 2)

**Reponse : A** (car 35×1 + 15×(-2) = 35 - 30 = 5)

---

### Question 4
Quelle est la complexite temporelle de l'algorithme d'Euclide ?

A) O(1)
B) O(n)
C) O(log n)
D) O(n log n)
E) O(n²)
F) O(log(min(a,b)))
G) O(sqrt(n))
H) O(2^n)
I) O(n!)
J) O(log(max(a,b)))

**Reponse : F**

---

### Question 5
Quelle est la valeur de φ(12) ?

A) 1
B) 2
C) 3
D) 4
E) 5
F) 6
G) 8
H) 10
I) 11
J) 12

**Reponse : D** (copremiers de 12 : {1, 5, 7, 11})

---

### Question 6
Le systeme x ≡ 2 (mod 4), x ≡ 0 (mod 6) a-t-il une solution ?

A) Oui, x = 6
B) Oui, x = 12
C) Oui, x = 18
D) Non, car gcd(4, 6) = 2 ≠ 1
E) Oui, x = 0
F) Non, car 2 ≢ 0 (mod gcd(4,6))
G) Oui, x = 2
H) Indetermine
I) Non, impossible
J) Oui, x = 24

**Reponse : D ou F** (les moduli ne sont pas copremiers)

---

### Question 7
Dans l'algorithme de Stein (binary GCD), quelle operation remplace la division ?

A) Addition
B) Multiplication
C) Bit shift (>>)
D) XOR
E) AND
F) Negation
G) Rotation
H) Modulo
I) Soustraction uniquement
J) Aucune des reponses

**Reponse : C**

---

### Question 8
Quelle est la longueur de la suite de Farey F_3 ?

A) 3
B) 4
C) 5
D) 6
E) 7
F) 8
G) 9
H) 10
I) 2
J) 1

**Reponse : C** (F_3 = {0/1, 1/3, 1/2, 2/3, 1/1})

---

### Question 9
Si gcd(a, b) = d et ax + by = d, que peut-on dire de x et y ?

A) x et y sont uniques
B) x et y peuvent etre negatifs
C) x > 0 et y > 0 toujours
D) x = y toujours
E) |x| + |y| < a + b
F) x × y = 0
G) x et y sont premiers
H) x + y = 1
I) x et y n'existent pas toujours
J) x = b/d et y = -a/d

**Reponse : B**

---

### Question 10
Quel est l'inverse de 3 modulo 11 ?

A) 1
B) 2
C) 3
D) 4
E) 5
F) 6
G) 7
H) 8
I) 9
J) 10

**Reponse : D** (car 3 × 4 = 12 ≡ 1 (mod 11))

---

## 📊 SECTION 8 : RECAPITULATIF

| Element | Valeur |
|---------|--------|
| **Module** | 1.6.2 — Euclidean Algorithms |
| **Exercice** | 1.6.2-a : interstellar_euclidean |
| **Difficulte** | 6/10 (★★★★★★☆☆☆☆) |
| **Type** | complet |
| **Tiers** | 3 — Synthese |
| **Langages** | Rust Edition 2024, C (c17) |
| **Duree** | 90 min |
| **XP Base** | 200 |
| **XP Bonus** | 200 × 2 = 400 |
| **Complexite** | O(log(min(a,b))) temps, O(log) espace |
| **Fonctions** | gcd, extended_gcd, mod_inverse, chinese_remainder, euler_phi, farey_sequence |
| **Bonus** | binary_gcd (Stein's Algorithm) |
| **Domaines** | MD, Algo, Crypto |
| **Theme** | Interstellar (Christopher Nolan, 2014) |

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.6.2-a-interstellar-euclidean",
    "generated_at": "2026-01-12 14:30:00",

    "metadata": {
      "exercise_id": "1.6.2-a",
      "exercise_name": "interstellar_euclidean",
      "module": "1.6.2",
      "module_name": "Euclidean Algorithms",
      "concept": "a",
      "concept_name": "GCD, Extended GCD, Modular Inverse, CRT, Euler Phi, Farey",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthese (concepts d-l combines)",
      "phase": 1,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust",
      "language_version": "Edition 2024",
      "alt_language": "c",
      "alt_language_version": "c17",
      "duration_minutes": 90,
      "xp_base": 200,
      "xp_bonus_multiplier": 2,
      "bonus_tier": "STANDARD",
      "bonus_icon": "⚡",
      "complexity_time": "T4 O(log(min(a,b)))",
      "complexity_space": "S2 O(log(min(a,b)))",
      "prerequisites": ["Arithmetique de base", "Recursion", "Modulo", "Divisibilite"],
      "domains": ["MD", "Algo", "Crypto"],
      "domains_bonus": ["CPU", "ASM"],
      "tags": ["number_theory", "gcd", "euclidean", "modular", "bezout", "crt", "euler_phi", "farey"],
      "meme_reference": "Interstellar - This is no time for caution",
      "culture_theme": "Interstellar (Christopher Nolan, 2014)"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 - Rust */",
      "references/ref_solution.c": "/* Section 4.3 - C */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "alternatives/alt_iterative.rs": "/* Section 4.4 - Alternative 1 */",
      "alternatives/alt_extended_iterative.rs": "/* Section 4.4 - Alternative 2 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 - gcd(a,0) = 0 */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 - extended_gcd negatifs */",
      "mutants/mutant_c_logic.rs": "/* Section 4.10 - mod_inverse sans check */",
      "mutants/mutant_d_logic.rs": "/* Section 4.10 - CRT sans inverse */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 - euler_phi incomplet */",
      "tests/main.c": "/* Section 4.2 */",
      "tests/test.rs": "/* Tests Rust */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "references/ref_solution.c",
        "references/ref_solution_bonus.rs",
        "alternatives/alt_iterative.rs",
        "alternatives/alt_extended_iterative.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_safety.rs",
        "mutants/mutant_c_logic.rs",
        "mutants/mutant_d_logic.rs",
        "mutants/mutant_e_return.rs"
      ]
    },

    "commands": {
      "validate_spec": "python3 hackbrain_engine_v22.py --validate-spec spec.json",
      "test_reference_rust": "cargo test --release",
      "test_reference_c": "gcc -Wall -Wextra -Werror -std=c17 -O2 *.c -o test && ./test",
      "test_mutants": "python3 hackbrain_mutation_tester.py -r references/ref_solution.rs -s spec.json --validate"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — Exercice 1.6.2-a : interstellar_euclidean*
*"L'excellence pedagogique ne se negocie pas — pas de raccourcis"*
*Theme : Interstellar (2014) — "Do not go gentle into that good night"*
