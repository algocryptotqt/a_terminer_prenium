<thinking>
## Analyse du Concept
- Concept : Arithmetique modulaire (operations mod, exponentiation rapide, theoreme de Fermat, log discret, racine carree mod p)
- Phase demandee : 1 (Intermediaire)
- Adapte ? OUI - Ces concepts sont fondamentaux en theorie des nombres et cryptographie, parfaitement adaptes a la Phase 1

## Combo Base + Bonus
- Exercice de base : Implementation des operations modulaires fondamentales (mod_add, mod_mul, mod_pow, mod_inv)
- Bonus : Implementation de l'algorithme de Tonelli-Shanks pour racine carree modulaire + Baby-step Giant-step pour log discret
- Palier bonus : 🔥 Avance (difficulte 7/10 -> bonus 9/10)
- Progression logique ? OUI - Base etablit les fondations, bonus ajoute complexite algorithmique

## Prerequis & Difficulte
- Prerequis reels : GCD/Extended Euclidean (ex01), arithmetique de base, notion de nombres premiers
- Difficulte estimee : 7/10
- Coherent avec phase ? OUI - Phase 1 = 3-5/10 base, on est un peu au-dessus mais justifie par la nature mathematique

## Aspect Fun/Culture
- Contexte choisi : Doctor Strange - Les Arts Mystiques de Kamar-Taj
- MEME mnémotechnique : "Dormammu, I've come to bargain" pour l'inverse modulaire (on negocie avec l'infini)
- Pourquoi c'est fun : La magie de Doctor Strange est basee sur des boucles temporelles et des dimensions miroir - parfaite analogie avec l'arithmetique modulaire cyclique

## Scenarios d'Echec (5 mutants concrets)
1. Mutant A (Boundary) : mod_pow avec exp=0 retourne 0 au lieu de 1
2. Mutant B (Safety) : Pas de verification m <= 0 dans les operations mod
3. Mutant C (Resource) : Overflow dans mod_mul sans cast en i128/u128
4. Mutant D (Logic) : mod_inv utilise p au lieu de p-2 dans Fermat (a^(p-1) vs a^(p-2))
5. Mutant E (Return) : discrete_log retourne Some(0) au lieu de None quand pas de solution

## Verdict
VALIDE - Exercice complet, theme parfait, difficulte appropriee
</thinking>

---

# Exercice 1.6.3-a : kamar_taj_mod

**Module :**
1.6.3 — Mathematiques Modulaires

**Concept :**
a — Operations modulaires fondamentales, Exponentiation rapide, Theoreme de Fermat, Logarithme discret, Racine carree mod p

**Difficulte :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
2 — Melange (concepts mod_ops + fast_exp + fermat_inv + discrete_log + sqrt_mod)

**Langage :**
Rust Edition 2024 ET C (C17)

**Prerequis :**
- 1.6.2 : GCD et Algorithme d'Euclide etendu
- Notions de nombres premiers
- Arithmetique de base (addition, multiplication, puissance)

**Domaines :**
MD, Crypto, Algo

**Duree estimee :**
90 min

**XP Base :**
150

**Complexite :**
T3 O(log n) pour mod_pow × S1 O(1) | T5 O(sqrt(m)) pour discrete_log × S3 O(sqrt(m))

---

## SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers a rendre :**
- `kamar_taj_mod.rs` (Rust Edition 2024)
- `kamar_taj_mod.c` + `kamar_taj_mod.h` (C17)

**Fonctions autorisees :**
- Rust : Aucune restriction (standard library)
- C : `<stdint.h>`, `<stdbool.h>`, `<stdlib.h>`, `<math.h>`

**Fonctions interdites :**
- Bibliotheques de cryptographie externes
- Fonctions de puissance modulaire pre-implementees

---

### 1.2 Consigne

**🎬 CONTEXTE FUN — Les Arts Mystiques de Kamar-Taj**

*"The language of the mystic arts is as old as civilization. The sorcerers of antiquity called the use of this language 'spells.' But if that word offends your modern sensibilities, you can call it a program."*
— The Ancient One

Tu viens d'arriver a Kamar-Taj, le sanctuaire secret ou Wong et l'Ancienne enseignent les arts mystiques. Mais avant de manipuler la Pierre du Temps ou d'ouvrir des portails vers d'autres dimensions, tu dois maitriser les **calculs modulaires** — la base mathematique de toute magie.

Dans l'univers de Doctor Strange, la magie fonctionne en **cycles** : le temps boucle, les dimensions se replient sur elles-memes, et les sorts suivent des patterns repetitifs. C'est exactement ce que fait l'arithmetique modulaire !

Wong t'explique :
- **"Le Sanctuaire calcule en cycles"** : Quand tu ajoutes de l'energie magique, elle "wrappe" apres avoir atteint le maximum — comme `(a + b) mod m`
- **"La Pierre du Temps accelere les calculs"** : L'exponentiation binaire permet de calculer `a^n mod m` en O(log n) au lieu de O(n)
- **"Negocier avec Dormammu"** : L'inverse modulaire via Fermat te permet de "diviser" dans un monde ou la division n'existe pas
- **"Lire le futur avec l'Ancienne"** : Le logarithme discret retrouve l'exposant cache
- **"La Dimension Miroir reflete"** : La racine carree modulaire trouve x tel que x² ≡ a (mod p)

---

**Ta mission :**

Implementer un module complet d'arithmetique modulaire avec les fonctions suivantes :

#### Fonctions de base (Le Sanctuaire)

1. **`kamar_taj_mod(a, b, m)`** — Addition modulaire securisee
2. **`kamar_taj_mul(a, b, m)`** — Multiplication modulaire (gere l'overflow)
3. **`kamar_taj_sub(a, b, m)`** — Soustraction modulaire (gere les negatifs)

#### Fonctions avancees (Les Reliques)

4. **`time_stone_pow(base, exp, m)`** — Exponentiation modulaire rapide O(log exp)
5. **`dormammu_inverse(a, m)`** — Inverse modulaire via theoreme de Fermat (m premier)
6. **`ancient_one_log(a, b, m)`** — Logarithme discret Baby-step Giant-step
7. **`mirror_sqrt(a, p)`** — Racine carree modulaire Tonelli-Shanks

#### Structure wrapper (Protection du Sanctuaire)

8. **`SanctumModInt`** — Structure encapsulant un entier modulaire avec operations surcharges

---

**Entree :**
- `a`, `b` : entiers (i64 en Rust, int64_t en C)
- `m`, `p` : module (i64 en Rust, int64_t en C) — `p` doit etre premier pour certaines fonctions
- `exp` : exposant (u64 en Rust, uint64_t en C)

**Sortie :**
- Resultat de l'operation modulaire
- `None`/`NULL` ou `-1` si l'operation est impossible

**Contraintes :**
- 1 ≤ m ≤ 10^18
- Les entrees peuvent etre negatives (sauf exposants)
- Gerer les overflows pour les grands nombres
- `dormammu_inverse` requiert m premier
- `mirror_sqrt` requiert p premier impair

---

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `kamar_taj_mod(7, 5, 3)` | `0` | (7 + 5) mod 3 = 12 mod 3 = 0 |
| `kamar_taj_mod(-3, 5, 7)` | `2` | (-3 + 5) mod 7 = 2 |
| `time_stone_pow(2, 10, 1000)` | `24` | 2^10 = 1024, 1024 mod 1000 = 24 |
| `time_stone_pow(3, 1000000007, 1000000007)` | `3` | Fermat: a^p ≡ a (mod p) |
| `dormammu_inverse(3, 7)` | `5` | 3 * 5 = 15 ≡ 1 (mod 7) |
| `ancient_one_log(2, 3, 5)` | `Some(3)` | 2^3 = 8 ≡ 3 (mod 5) |
| `mirror_sqrt(2, 7)` | `Some(3)` | 3² = 9 ≡ 2 (mod 7) |
| `mirror_sqrt(3, 7)` | `None` | 3 n'est pas un residu quadratique mod 7 |

---

### 1.2.2 Enonce Academique

Implementer un module d'arithmetique modulaire comprenant :

1. **Operations de base** : Addition, soustraction, multiplication modulaires avec gestion des nombres negatifs et des overflows.

2. **Exponentiation modulaire rapide** : Algorithme d'exponentiation binaire en O(log n).

3. **Inverse modulaire** : Utilisation du petit theoreme de Fermat pour calculer a^(-1) mod p quand p est premier.

4. **Logarithme discret** : Algorithme Baby-step Giant-step pour resoudre a^x ≡ b (mod m).

5. **Racine carree modulaire** : Algorithme de Tonelli-Shanks pour trouver x tel que x² ≡ a (mod p).

6. **Theoreme de Lucas** : Calcul de C(n,k) mod p pour grands n.

7. **Structure ModInt** : Encapsulation des operations avec surcharge des operateurs.

---

### 1.3 Prototype

#### Rust Edition 2024

```rust
//! Kamar-Taj Modular Arithmetic Library
//! Les Arts Mystiques de l'Arithmetique Modulaire

pub const SANCTUM_MOD: i64 = 1_000_000_007;

/// Operations de base du Sanctuaire
pub mod sanctum_ops {
    /// Addition modulaire securisee (gere les negatifs)
    pub fn kamar_taj_mod(a: i64, b: i64, m: i64) -> i64;

    /// Multiplication modulaire (gere l'overflow via i128)
    pub fn kamar_taj_mul(a: i64, b: i64, m: i64) -> i64;

    /// Soustraction modulaire
    pub fn kamar_taj_sub(a: i64, b: i64, m: i64) -> i64;
}

/// Reliques de pouvoir
pub mod relics {
    /// Exponentiation binaire - O(log exp)
    /// "With the Time Stone, I can calculate powers across millennia in milliseconds"
    pub fn time_stone_pow(base: i64, exp: u64, m: i64) -> i64;

    /// Inverse modulaire via Fermat (m DOIT etre premier)
    /// "Dormammu, I've come to bargain... for your multiplicative inverse"
    pub fn dormammu_inverse(a: i64, m: i64) -> Option<i64>;

    /// Inverse modulaire via Euclide etendu (fonctionne si gcd(a,m)=1)
    pub fn extended_inverse(a: i64, m: i64) -> Option<i64>;

    /// Division modulaire: a / b mod m
    pub fn sanctum_divide(a: i64, b: i64, m: i64) -> Option<i64>;
}

/// Arts anciens avances
pub mod ancient_arts {
    /// Logarithme discret Baby-step Giant-step - O(sqrt(m))
    /// "The Ancient One could see all possible futures..."
    pub fn ancient_one_log(a: i64, b: i64, m: i64) -> Option<u64>;

    /// Racine carree modulaire Tonelli-Shanks
    /// "In the Mirror Dimension, every reflection has a square root"
    pub fn mirror_sqrt(a: i64, p: i64) -> Option<i64>;

    /// Symbole de Legendre (a/p)
    pub fn legendre_symbol(a: i64, p: i64) -> i8;

    /// Verification residu quadratique
    pub fn is_quadratic_residue(a: i64, p: i64) -> bool;
}

/// Bibliotheque de Wong - Combinatoire modulaire
pub mod wong_library {
    /// Theoreme de Lucas pour C(n,k) mod p
    pub fn wong_lucas(n: u64, k: u64, p: u64) -> u64;

    /// Factorielle modulaire
    pub fn factorial_mod(n: u64, m: i64) -> i64;

    /// Coefficient binomial modulaire
    pub fn binomial_mod(n: usize, k: usize, m: i64) -> i64;

    /// Nombre de Catalan modulaire
    pub fn catalan_mod(n: usize, m: i64) -> i64;
}

/// Structure ModInt - Protection du Sanctuaire
#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
pub struct SanctumModInt<const M: i64> {
    val: i64,
}

impl<const M: i64> SanctumModInt<M> {
    pub fn new(v: i64) -> Self;
    pub fn val(&self) -> i64;
    pub fn pow(self, exp: u64) -> Self;
    pub fn inv(self) -> Option<Self>;
}

impl<const M: i64> std::ops::Add for SanctumModInt<M> { /* ... */ }
impl<const M: i64> std::ops::Sub for SanctumModInt<M> { /* ... */ }
impl<const M: i64> std::ops::Mul for SanctumModInt<M> { /* ... */ }
impl<const M: i64> std::ops::Div for SanctumModInt<M> { /* ... */ }
impl<const M: i64> std::ops::Neg for SanctumModInt<M> { /* ... */ }
```

#### C (C17)

```c
// kamar_taj_mod.h
#ifndef KAMAR_TAJ_MOD_H
#define KAMAR_TAJ_MOD_H

#include <stdint.h>
#include <stdbool.h>

#define SANCTUM_MOD 1000000007LL

// ============== Operations de base ==============

// Addition modulaire securisee
int64_t kamar_taj_mod(int64_t a, int64_t b, int64_t m);

// Multiplication modulaire (gere overflow)
int64_t kamar_taj_mul(int64_t a, int64_t b, int64_t m);

// Soustraction modulaire
int64_t kamar_taj_sub(int64_t a, int64_t b, int64_t m);

// ============== Reliques de pouvoir ==============

// Exponentiation binaire O(log exp)
int64_t time_stone_pow(int64_t base, uint64_t exp, int64_t m);

// Inverse modulaire via Fermat (m premier)
// Retourne -1 si impossible
int64_t dormammu_inverse(int64_t a, int64_t m);

// Inverse via Euclide etendu
// Retourne -1 si gcd(a,m) != 1
int64_t extended_inverse(int64_t a, int64_t m);

// Division modulaire
// Retourne -1 si impossible
int64_t sanctum_divide(int64_t a, int64_t b, int64_t m);

// ============== Arts anciens ==============

// Logarithme discret Baby-step Giant-step
// Retourne -1 si pas de solution
int64_t ancient_one_log(int64_t a, int64_t b, int64_t m);

// Racine carree modulaire Tonelli-Shanks
// Retourne -1 si pas de solution
int64_t mirror_sqrt(int64_t a, int64_t p);

// Symbole de Legendre
int8_t legendre_symbol(int64_t a, int64_t p);

// ============== Bibliotheque de Wong ==============

// Theoreme de Lucas
uint64_t wong_lucas(uint64_t n, uint64_t k, uint64_t p);

// Factorielle mod m
int64_t factorial_mod(uint64_t n, int64_t m);

// Coefficient binomial mod m
int64_t binomial_mod(uint64_t n, uint64_t k, int64_t m);

// ============== Structure ModInt ==============

typedef struct {
    int64_t val;
    int64_t mod;
} SanctumModInt;

SanctumModInt sanctum_new(int64_t v, int64_t m);
int64_t sanctum_val(SanctumModInt x);
SanctumModInt sanctum_add(SanctumModInt a, SanctumModInt b);
SanctumModInt sanctum_sub(SanctumModInt a, SanctumModInt b);
SanctumModInt sanctum_mul(SanctumModInt a, SanctumModInt b);
SanctumModInt sanctum_div(SanctumModInt a, SanctumModInt b);
SanctumModInt sanctum_pow(SanctumModInt base, uint64_t exp);
SanctumModInt sanctum_inv(SanctumModInt x);

#endif // KAMAR_TAJ_MOD_H
```

---

## SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Pourquoi Doctor Strange ?

L'analogie entre Doctor Strange et l'arithmetique modulaire est remarquablement profonde :

| Element Doctor Strange | Concept Mathematique |
|------------------------|----------------------|
| **Boucle temporelle** ("Dormammu, I've come to bargain") | Propriete cyclique de mod : apres m, on revient a 0 |
| **Pierre du Temps** | Exponentiation rapide : manipuler le temps logarithmiquement |
| **Dimension Miroir** | Racines carrees : chaque nombre a (ou pas) un "reflet" |
| **L'Ancienne voit tous les futurs** | Log discret : retrouver le chemin qui mene a un resultat |
| **Sanctuaires proteges** | Structure ModInt : encapsulation et protection des calculs |
| **Wong et sa bibliotheque** | Theoreme de Lucas : acces rapide aux grands coefficients binomiaux |

### 2.2 Histoire de l'arithmetique modulaire

- **Gauss (1801)** : Formalise la congruence dans "Disquisitiones Arithmeticae"
- **Fermat (1640)** : Petit theoreme (a^(p-1) ≡ 1 mod p)
- **Euler (1736)** : Generalisation avec la fonction phi
- **Tonelli (1891)** : Algorithme pour racines carrees modulaires
- **Shanks (1973)** : Amelioration de l'algorithme de Tonelli
- **Baby-step Giant-step (Shanks, 1971)** : Logarithme discret en O(sqrt(m))

---

### SECTION 2.5 : DANS LA VRAIE VIE

#### Metiers utilisant l'arithmetique modulaire

| Metier | Cas d'usage |
|--------|-------------|
| **Cryptographe** | RSA, Diffie-Hellman, courbes elliptiques — TOUT repose sur mod_pow et mod_inv |
| **Developpeur Blockchain** | Signatures ECDSA, verification de transactions |
| **Ingenieur Securite** | Generation de cles, protocoles d'authentification |
| **Data Scientist** | Hashing modulaire pour tables de hachage distribuees |
| **Developpeur Jeux Video** | Generateurs pseudo-aleatoires (LCG), checksums |
| **Ingenieur Telecom** | Codes correcteurs d'erreurs (Reed-Solomon) |

#### Exemples concrets

1. **RSA** : `message^e mod n` pour chiffrer, `cipher^d mod n` pour dechiffrer
2. **Diffie-Hellman** : `g^a mod p` echange de cles
3. **Bitcoin** : Signatures ECDSA sur courbe secp256k1
4. **Git** : SHA-1 utilise des operations modulaires internes
5. **PostgreSQL** : Fonctions de hachage pour index

---

## SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
kamar_taj_mod.rs  kamar_taj_mod.c  kamar_taj_mod.h  main.c  main.rs

$ rustc --edition 2024 kamar_taj_mod.rs main.rs -o test_rust

$ ./test_rust
=== Sanctum Operations Test ===
kamar_taj_mod(7, 5, 3) = 0 ... OK
kamar_taj_mod(-3, 5, 7) = 2 ... OK
kamar_taj_mul(1000000, 1000000, 1000000007) = 999999993 ... OK

=== Time Stone Power Test ===
time_stone_pow(2, 10, 1000) = 24 ... OK
time_stone_pow(3, 100, 1000000007) = 981985467 ... OK

=== Dormammu Inverse Test ===
dormammu_inverse(3, 7) = 5 ... OK (3 * 5 = 15 ≡ 1 mod 7)
dormammu_inverse(2, 1000000007) = 500000004 ... OK

=== Ancient One Log Test ===
ancient_one_log(2, 3, 5) = Some(3) ... OK (2^3 = 8 ≡ 3 mod 5)
ancient_one_log(3, 13, 17) = Some(4) ... OK

=== Mirror Sqrt Test ===
mirror_sqrt(2, 7) = Some(3) ... OK (3^2 = 9 ≡ 2 mod 7)
mirror_sqrt(3, 7) = None ... OK (3 is not a QR mod 7)

=== Wong Lucas Test ===
wong_lucas(100, 10, 13) = 10 ... OK

=== SanctumModInt Test ===
ModInt<1000000007>: (500000000 + 600000000).val() = 99999993 ... OK

All tests passed! The Sanctum is protected.

$ gcc -std=c17 -Wall -Wextra -Werror kamar_taj_mod.c main.c -o test_c -lm

$ ./test_c
[C17 Tests]
kamar_taj_mod(7, 5, 3) = 0 ... OK
time_stone_pow(2, 10, 1000) = 24 ... OK
dormammu_inverse(3, 7) = 5 ... OK
ancient_one_log(2, 3, 5) = 3 ... OK
mirror_sqrt(2, 7) = 3 ... OK
All tests passed!
```

---

### 3.1 BONUS AVANCE (OPTIONNEL)

**Difficulte Bonus :**
★★★★★★★★★☆ (9/10)

**Recompense :**
XP x3

**Time Complexity attendue :**
O(log^2 p) pour Tonelli-Shanks optimise

**Space Complexity attendue :**
O(1) pour toutes les operations

**Domaines Bonus :**
`Crypto, DP`

#### 3.1.1 Consigne Bonus

**🎬 L'EVEIL DU MULTIVERS**

*"In the multiverse, everything happens. You just have to look in the right dimension."*

Doctor Strange a ouvert les portes du Multivers. Tu dois maintenant implementer des operations encore plus avancees :

**Ta mission bonus :**

1. **`multiverse_crt(remainders, moduli)`** — Theoreme des Restes Chinois generalise
2. **`darkhold_primitive_root(p)`** — Trouver une racine primitive modulo p
3. **`vishanti_nth_root(a, n, p)`** — Racine n-ieme modulaire
4. **`batch_dormammu_inverse(arr, m)`** — Inverses en batch O(n) au lieu de O(n log m)

**Contraintes Bonus :**
```
┌─────────────────────────────────────────┐
│  Tous les moduli premiers entre eux     │
│  1 ≤ |remainders| ≤ 100                 │
│  batch_inverse en O(n) exact            │
│  Pas de HashMap pour primitive_root     │
└─────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
/// Theoreme des Restes Chinois
pub fn multiverse_crt(remainders: &[i64], moduli: &[i64]) -> Option<i64>;

/// Racine primitive modulo p
pub fn darkhold_primitive_root(p: i64) -> Option<i64>;

/// Racine n-ieme: x^n ≡ a (mod p)
pub fn vishanti_nth_root(a: i64, n: i64, p: i64) -> Option<i64>;

/// Batch inverse en O(n)
pub fn batch_dormammu_inverse(arr: &[i64], m: i64) -> Vec<i64>;
```

#### 3.1.3 Ce qui change par rapport a l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Inverse | Un seul a la fois O(log m) | Batch O(n) total |
| Racines | Carree seulement | N-ieme generalise |
| CRT | Non inclus | Implementation complete |
| Complexite | O(log m) par op | Optimisations avancees |

---

## SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette (tableau des tests)

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `mod_add_basic` | `(7, 5, 3)` | `0` | 2 | Non |
| `mod_add_negative` | `(-3, 5, 7)` | `2` | 3 | Oui |
| `mod_add_overflow` | `(10^18, 10^18, 10^9+7)` | correct | 5 | Oui |
| `mod_mul_basic` | `(6, 7, 5)` | `2` | 2 | Non |
| `mod_mul_overflow` | `(10^9, 10^9, 10^9+7)` | `999999993` | 5 | Oui |
| `pow_zero_exp` | `(5, 0, 7)` | `1` | 3 | Oui |
| `pow_basic` | `(2, 10, 1000)` | `24` | 3 | Non |
| `pow_large` | `(3, 10^18, 10^9+7)` | correct | 5 | Non |
| `pow_fermat` | `(a, p-1, p)` | `1` | 5 | Non |
| `inv_basic` | `(3, 7)` | `5` | 5 | Non |
| `inv_one` | `(1, p)` | `1` | 3 | Oui |
| `inv_zero` | `(0, p)` | `None/-1` | 5 | Oui |
| `log_basic` | `(2, 3, 5)` | `3` | 10 | Non |
| `log_no_solution` | `(2, 3, 7)` | `None/-1` | 5 | Oui |
| `sqrt_basic` | `(2, 7)` | `3 ou 4` | 10 | Non |
| `sqrt_non_qr` | `(3, 7)` | `None/-1` | 5 | Oui |
| `sqrt_zero` | `(0, 7)` | `0` | 3 | Oui |
| `lucas_basic` | `(100, 10, 13)` | `10` | 10 | Non |
| `lucas_large` | `(10^18, 10^9, 13)` | correct | 5 | Non |
| `modint_add` | `new(5)+new(3)` | `val()=8` | 5 | Non |
| `modint_overflow` | `new(10^9)+new(10^9)` | wrapped | 5 | Oui |

**Total : 100 points**

---

### 4.2 main.c de test

```c
#include <stdio.h>
#include <assert.h>
#include <stdint.h>
#include "kamar_taj_mod.h"

#define TEST(name, cond) do { \
    if (cond) { printf("[OK] %s\n", name); passed++; } \
    else { printf("[FAIL] %s\n", name); failed++; } \
} while(0)

int main(void) {
    int passed = 0, failed = 0;

    // === Operations de base ===
    TEST("mod_add_basic", kamar_taj_mod(7, 5, 3) == 0);
    TEST("mod_add_negative", kamar_taj_mod(-3, 5, 7) == 2);
    TEST("mod_sub_basic", kamar_taj_sub(3, 5, 7) == 5); // (3-5) mod 7 = -2 mod 7 = 5
    TEST("mod_mul_basic", kamar_taj_mul(6, 7, 5) == 2);

    // === Overflow ===
    int64_t big = 1000000000LL;
    int64_t mod = 1000000007LL;
    TEST("mod_mul_overflow", kamar_taj_mul(big, big, mod) == 999999993LL);

    // === Exponentiation ===
    TEST("pow_zero", time_stone_pow(5, 0, 7) == 1);
    TEST("pow_basic", time_stone_pow(2, 10, 1000) == 24);
    TEST("pow_one", time_stone_pow(2, 1, 1000) == 2);
    TEST("pow_fermat", time_stone_pow(3, mod - 1, mod) == 1);

    // === Inverse ===
    TEST("inv_basic", dormammu_inverse(3, 7) == 5);
    TEST("inv_verify", kamar_taj_mul(3, dormammu_inverse(3, 7), 7) == 1);
    TEST("inv_one", dormammu_inverse(1, 7) == 1);
    TEST("inv_zero", dormammu_inverse(0, 7) == -1);

    // === Log discret ===
    int64_t log_result = ancient_one_log(2, 3, 5);
    TEST("log_basic", log_result == 3);
    TEST("log_verify", time_stone_pow(2, (uint64_t)log_result, 5) == 3);
    TEST("log_no_solution", ancient_one_log(2, 3, 7) == -1);

    // === Racine carree ===
    int64_t sqrt_result = mirror_sqrt(2, 7);
    TEST("sqrt_basic", sqrt_result == 3 || sqrt_result == 4);
    TEST("sqrt_verify", kamar_taj_mul(sqrt_result, sqrt_result, 7) == 2);
    TEST("sqrt_non_qr", mirror_sqrt(3, 7) == -1);
    TEST("sqrt_zero", mirror_sqrt(0, 7) == 0);

    // === Lucas ===
    TEST("lucas_basic", wong_lucas(5, 2, 13) == 10);
    TEST("lucas_100_10", wong_lucas(100, 10, 13) == 10);

    // === ModInt ===
    SanctumModInt a = sanctum_new(500000000, mod);
    SanctumModInt b = sanctum_new(600000000, mod);
    SanctumModInt c = sanctum_add(a, b);
    TEST("modint_add", sanctum_val(c) == 99999993);

    SanctumModInt d = sanctum_new(3, 7);
    SanctumModInt e = sanctum_inv(d);
    SanctumModInt f = sanctum_mul(d, e);
    TEST("modint_inv", sanctum_val(f) == 1);

    printf("\n=== Results: %d passed, %d failed ===\n", passed, failed);
    return failed > 0 ? 1 : 0;
}
```

---

### 4.3 Solution de reference

#### Rust

```rust
//! Solution de reference - Kamar-Taj Modular Arithmetic

pub const SANCTUM_MOD: i64 = 1_000_000_007;

pub mod sanctum_ops {
    /// Normalise un nombre dans [0, m)
    #[inline]
    fn normalize(mut x: i64, m: i64) -> i64 {
        x %= m;
        if x < 0 { x + m } else { x }
    }

    pub fn kamar_taj_mod(a: i64, b: i64, m: i64) -> i64 {
        if m <= 0 { return 0; }
        normalize(normalize(a, m) + normalize(b, m), m)
    }

    pub fn kamar_taj_mul(a: i64, b: i64, m: i64) -> i64 {
        if m <= 0 { return 0; }
        let a = normalize(a, m);
        let b = normalize(b, m);
        ((a as i128 * b as i128) % m as i128) as i64
    }

    pub fn kamar_taj_sub(a: i64, b: i64, m: i64) -> i64 {
        if m <= 0 { return 0; }
        normalize(normalize(a, m) - normalize(b, m), m)
    }
}

pub mod relics {
    use super::sanctum_ops::*;

    pub fn time_stone_pow(base: i64, mut exp: u64, m: i64) -> i64 {
        if m <= 0 { return 0; }
        if m == 1 { return 0; }

        let mut result: i64 = 1;
        let mut base = ((base % m) + m) % m;

        while exp > 0 {
            if exp & 1 == 1 {
                result = kamar_taj_mul(result, base, m);
            }
            base = kamar_taj_mul(base, base, m);
            exp >>= 1;
        }
        result
    }

    pub fn dormammu_inverse(a: i64, m: i64) -> Option<i64> {
        if m <= 1 { return None; }
        let a = ((a % m) + m) % m;
        if a == 0 { return None; }

        // Fermat: a^(-1) = a^(m-2) mod m (m premier)
        Some(time_stone_pow(a, (m - 2) as u64, m))
    }

    pub fn extended_inverse(a: i64, m: i64) -> Option<i64> {
        fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64) {
            if b == 0 {
                (a, 1, 0)
            } else {
                let (g, x, y) = extended_gcd(b, a % b);
                (g, y, x - (a / b) * y)
            }
        }

        let a = ((a % m) + m) % m;
        if a == 0 { return None; }

        let (g, x, _) = extended_gcd(a, m);
        if g != 1 { return None; }

        Some(((x % m) + m) % m)
    }

    pub fn sanctum_divide(a: i64, b: i64, m: i64) -> Option<i64> {
        let inv_b = dormammu_inverse(b, m)?;
        Some(kamar_taj_mul(a, inv_b, m))
    }
}

pub mod ancient_arts {
    use super::relics::*;
    use super::sanctum_ops::*;
    use std::collections::HashMap;

    pub fn legendre_symbol(a: i64, p: i64) -> i8 {
        let a = ((a % p) + p) % p;
        if a == 0 { return 0; }

        let result = time_stone_pow(a, ((p - 1) / 2) as u64, p);
        if result == 1 { 1 }
        else if result == p - 1 { -1 }
        else { 0 }
    }

    pub fn is_quadratic_residue(a: i64, p: i64) -> bool {
        legendre_symbol(a, p) == 1
    }

    pub fn ancient_one_log(a: i64, b: i64, m: i64) -> Option<u64> {
        if m <= 1 { return None; }

        let a = ((a % m) + m) % m;
        let b = ((b % m) + m) % m;

        if b == 1 { return Some(0); }

        // Baby-step Giant-step
        let n = (m as f64).sqrt().ceil() as u64 + 1;

        // Baby steps: store a^j for j in [0, n)
        let mut table: HashMap<i64, u64> = HashMap::new();
        let mut pow_aj: i64 = 1;
        for j in 0..n {
            table.entry(pow_aj).or_insert(j);
            pow_aj = kamar_taj_mul(pow_aj, a, m);
        }

        // Giant step multiplier: a^(-n) mod m
        let a_inv = dormammu_inverse(a, m)?;
        let factor = time_stone_pow(a_inv, n, m);

        // Giant steps: check if b * (a^(-n))^i is in table
        let mut gamma = b;
        for i in 0..n {
            if let Some(&j) = table.get(&gamma) {
                let result = i * n + j;
                // Verify
                if time_stone_pow(a, result, m) == b {
                    return Some(result);
                }
            }
            gamma = kamar_taj_mul(gamma, factor, m);
        }

        None
    }

    pub fn mirror_sqrt(a: i64, p: i64) -> Option<i64> {
        if p <= 1 { return None; }

        let a = ((a % p) + p) % p;
        if a == 0 { return Some(0); }
        if p == 2 { return Some(a); }

        // Check if a is a quadratic residue
        if legendre_symbol(a, p) != 1 {
            return None;
        }

        // Special case: p ≡ 3 (mod 4)
        if p % 4 == 3 {
            return Some(time_stone_pow(a, ((p + 1) / 4) as u64, p));
        }

        // Tonelli-Shanks algorithm
        // Write p - 1 = Q * 2^S
        let mut q = p - 1;
        let mut s: u64 = 0;
        while q % 2 == 0 {
            q /= 2;
            s += 1;
        }

        // Find a quadratic non-residue z
        let mut z: i64 = 2;
        while legendre_symbol(z, p) != -1 {
            z += 1;
        }

        let mut m = s;
        let mut c = time_stone_pow(z, q as u64, p);
        let mut t = time_stone_pow(a, q as u64, p);
        let mut r = time_stone_pow(a, ((q + 1) / 2) as u64, p);

        loop {
            if t == 0 { return Some(0); }
            if t == 1 { return Some(r); }

            // Find least i such that t^(2^i) = 1
            let mut i: u64 = 1;
            let mut temp = kamar_taj_mul(t, t, p);
            while temp != 1 {
                temp = kamar_taj_mul(temp, temp, p);
                i += 1;
                if i == m { return None; }
            }

            // Update
            let b = time_stone_pow(c, 1u64 << (m - i - 1), p);
            m = i;
            c = kamar_taj_mul(b, b, p);
            t = kamar_taj_mul(t, c, p);
            r = kamar_taj_mul(r, b, p);
        }
    }
}

pub mod wong_library {
    use super::relics::*;
    use super::sanctum_ops::*;

    pub fn factorial_mod(n: u64, m: i64) -> i64 {
        if m <= 1 { return 0; }
        let mut result: i64 = 1;
        for i in 2..=n {
            result = kamar_taj_mul(result, i as i64, m);
        }
        result
    }

    pub fn binomial_mod(n: usize, k: usize, m: i64) -> i64 {
        if k > n { return 0; }
        if k == 0 || k == n { return 1; }

        let k = k.min(n - k);
        let mut num: i64 = 1;
        let mut den: i64 = 1;

        for i in 0..k {
            num = kamar_taj_mul(num, (n - i) as i64, m);
            den = kamar_taj_mul(den, (i + 1) as i64, m);
        }

        let inv_den = dormammu_inverse(den, m).unwrap_or(0);
        kamar_taj_mul(num, inv_den, m)
    }

    pub fn wong_lucas(n: u64, k: u64, p: u64) -> u64 {
        if k > n { return 0; }
        if k == 0 { return 1; }

        let p_i64 = p as i64;
        let mut result: i64 = 1;
        let mut n = n;
        let mut k = k;

        while n > 0 || k > 0 {
            let ni = (n % p) as usize;
            let ki = (k % p) as usize;

            if ki > ni { return 0; }

            result = kamar_taj_mul(result, binomial_mod(ni, ki, p_i64), p_i64);
            n /= p;
            k /= p;
        }

        result as u64
    }

    pub fn catalan_mod(n: usize, m: i64) -> i64 {
        // C_n = C(2n, n) / (n + 1)
        let c2n_n = binomial_mod(2 * n, n, m);
        let inv_n1 = dormammu_inverse((n + 1) as i64, m).unwrap_or(0);
        kamar_taj_mul(c2n_n, inv_n1, m)
    }
}

#[derive(Clone, Copy, Debug, PartialEq, Eq, Hash)]
pub struct SanctumModInt<const M: i64> {
    val: i64,
}

impl<const M: i64> SanctumModInt<M> {
    pub fn new(v: i64) -> Self {
        let val = ((v % M) + M) % M;
        Self { val }
    }

    pub fn val(&self) -> i64 {
        self.val
    }

    pub fn pow(self, exp: u64) -> Self {
        Self::new(relics::time_stone_pow(self.val, exp, M))
    }

    pub fn inv(self) -> Option<Self> {
        relics::dormammu_inverse(self.val, M).map(Self::new)
    }
}

impl<const M: i64> std::ops::Add for SanctumModInt<M> {
    type Output = Self;
    fn add(self, rhs: Self) -> Self {
        Self::new(sanctum_ops::kamar_taj_mod(self.val, rhs.val, M))
    }
}

impl<const M: i64> std::ops::Sub for SanctumModInt<M> {
    type Output = Self;
    fn sub(self, rhs: Self) -> Self {
        Self::new(sanctum_ops::kamar_taj_sub(self.val, rhs.val, M))
    }
}

impl<const M: i64> std::ops::Mul for SanctumModInt<M> {
    type Output = Self;
    fn mul(self, rhs: Self) -> Self {
        Self::new(sanctum_ops::kamar_taj_mul(self.val, rhs.val, M))
    }
}

impl<const M: i64> std::ops::Div for SanctumModInt<M> {
    type Output = Self;
    fn div(self, rhs: Self) -> Self {
        let inv = rhs.inv().expect("Division by non-invertible element");
        self * inv
    }
}

impl<const M: i64> std::ops::Neg for SanctumModInt<M> {
    type Output = Self;
    fn neg(self) -> Self {
        Self::new(M - self.val)
    }
}
```

#### C (C17)

```c
// kamar_taj_mod.c - Solution de reference

#include "kamar_taj_mod.h"
#include <stdlib.h>
#include <math.h>

// ============== Helpers ==============

static inline int64_t normalize(int64_t x, int64_t m) {
    x %= m;
    return x < 0 ? x + m : x;
}

// ============== Operations de base ==============

int64_t kamar_taj_mod(int64_t a, int64_t b, int64_t m) {
    if (m <= 0) return 0;
    return normalize(normalize(a, m) + normalize(b, m), m);
}

int64_t kamar_taj_mul(int64_t a, int64_t b, int64_t m) {
    if (m <= 0) return 0;
    a = normalize(a, m);
    b = normalize(b, m);
    return (int64_t)(((__int128)a * b) % m);
}

int64_t kamar_taj_sub(int64_t a, int64_t b, int64_t m) {
    if (m <= 0) return 0;
    return normalize(normalize(a, m) - normalize(b, m), m);
}

// ============== Reliques ==============

int64_t time_stone_pow(int64_t base, uint64_t exp, int64_t m) {
    if (m <= 0) return 0;
    if (m == 1) return 0;

    int64_t result = 1;
    base = normalize(base, m);

    while (exp > 0) {
        if (exp & 1) {
            result = kamar_taj_mul(result, base, m);
        }
        base = kamar_taj_mul(base, base, m);
        exp >>= 1;
    }
    return result;
}

int64_t dormammu_inverse(int64_t a, int64_t m) {
    if (m <= 1) return -1;
    a = normalize(a, m);
    if (a == 0) return -1;

    // Fermat: a^(-1) = a^(m-2) mod m
    return time_stone_pow(a, (uint64_t)(m - 2), m);
}

int64_t extended_inverse(int64_t a, int64_t m) {
    a = normalize(a, m);
    if (a == 0) return -1;

    int64_t old_r = a, r = m;
    int64_t old_s = 1, s = 0;

    while (r != 0) {
        int64_t q = old_r / r;
        int64_t temp = old_r - q * r;
        old_r = r; r = temp;
        temp = old_s - q * s;
        old_s = s; s = temp;
    }

    if (old_r != 1) return -1;
    return normalize(old_s, m);
}

int64_t sanctum_divide(int64_t a, int64_t b, int64_t m) {
    int64_t inv_b = dormammu_inverse(b, m);
    if (inv_b == -1) return -1;
    return kamar_taj_mul(a, inv_b, m);
}

// ============== Arts anciens ==============

int8_t legendre_symbol(int64_t a, int64_t p) {
    a = normalize(a, p);
    if (a == 0) return 0;

    int64_t result = time_stone_pow(a, (uint64_t)((p - 1) / 2), p);
    if (result == 1) return 1;
    if (result == p - 1) return -1;
    return 0;
}

int64_t ancient_one_log(int64_t a, int64_t b, int64_t m) {
    if (m <= 1) return -1;

    a = normalize(a, m);
    b = normalize(b, m);

    if (b == 1) return 0;

    // Baby-step Giant-step
    uint64_t n = (uint64_t)(sqrt((double)m)) + 1;

    // Simple array-based lookup (for small m)
    // For production, use a hash table
    int64_t *baby = (int64_t *)malloc(n * sizeof(int64_t));
    if (!baby) return -1;

    int64_t pow_aj = 1;
    for (uint64_t j = 0; j < n; j++) {
        baby[j] = pow_aj;
        pow_aj = kamar_taj_mul(pow_aj, a, m);
    }

    // Giant step
    int64_t a_inv = dormammu_inverse(a, m);
    if (a_inv == -1) { free(baby); return -1; }

    int64_t factor = time_stone_pow(a_inv, n, m);
    int64_t gamma = b;

    for (uint64_t i = 0; i < n; i++) {
        for (uint64_t j = 0; j < n; j++) {
            if (baby[j] == gamma) {
                uint64_t result = i * n + j;
                if (time_stone_pow(a, result, m) == b) {
                    free(baby);
                    return (int64_t)result;
                }
            }
        }
        gamma = kamar_taj_mul(gamma, factor, m);
    }

    free(baby);
    return -1;
}

int64_t mirror_sqrt(int64_t a, int64_t p) {
    if (p <= 1) return -1;

    a = normalize(a, p);
    if (a == 0) return 0;
    if (p == 2) return a;

    // Check quadratic residue
    if (legendre_symbol(a, p) != 1) return -1;

    // p ≡ 3 (mod 4)
    if (p % 4 == 3) {
        return time_stone_pow(a, (uint64_t)((p + 1) / 4), p);
    }

    // Tonelli-Shanks
    int64_t q = p - 1;
    uint64_t s = 0;
    while (q % 2 == 0) {
        q /= 2;
        s++;
    }

    int64_t z = 2;
    while (legendre_symbol(z, p) != -1) z++;

    uint64_t m_val = s;
    int64_t c = time_stone_pow(z, (uint64_t)q, p);
    int64_t t = time_stone_pow(a, (uint64_t)q, p);
    int64_t r = time_stone_pow(a, (uint64_t)((q + 1) / 2), p);

    while (1) {
        if (t == 0) return 0;
        if (t == 1) return r;

        uint64_t i = 1;
        int64_t temp = kamar_taj_mul(t, t, p);
        while (temp != 1) {
            temp = kamar_taj_mul(temp, temp, p);
            i++;
            if (i == m_val) return -1;
        }

        int64_t b_val = time_stone_pow(c, 1ULL << (m_val - i - 1), p);
        m_val = i;
        c = kamar_taj_mul(b_val, b_val, p);
        t = kamar_taj_mul(t, c, p);
        r = kamar_taj_mul(r, b_val, p);
    }
}

// ============== Wong Library ==============

int64_t factorial_mod(uint64_t n, int64_t m) {
    if (m <= 1) return 0;
    int64_t result = 1;
    for (uint64_t i = 2; i <= n; i++) {
        result = kamar_taj_mul(result, (int64_t)i, m);
    }
    return result;
}

int64_t binomial_mod(uint64_t n, uint64_t k, int64_t m) {
    if (k > n) return 0;
    if (k == 0 || k == n) return 1;

    if (k > n - k) k = n - k;

    int64_t num = 1, den = 1;
    for (uint64_t i = 0; i < k; i++) {
        num = kamar_taj_mul(num, (int64_t)(n - i), m);
        den = kamar_taj_mul(den, (int64_t)(i + 1), m);
    }

    int64_t inv_den = dormammu_inverse(den, m);
    if (inv_den == -1) return 0;
    return kamar_taj_mul(num, inv_den, m);
}

uint64_t wong_lucas(uint64_t n, uint64_t k, uint64_t p) {
    if (k > n) return 0;
    if (k == 0) return 1;

    int64_t result = 1;
    while (n > 0 || k > 0) {
        uint64_t ni = n % p;
        uint64_t ki = k % p;

        if (ki > ni) return 0;

        result = kamar_taj_mul(result, binomial_mod(ni, ki, (int64_t)p), (int64_t)p);
        n /= p;
        k /= p;
    }
    return (uint64_t)result;
}

// ============== ModInt ==============

SanctumModInt sanctum_new(int64_t v, int64_t m) {
    SanctumModInt x;
    x.mod = m;
    x.val = normalize(v, m);
    return x;
}

int64_t sanctum_val(SanctumModInt x) {
    return x.val;
}

SanctumModInt sanctum_add(SanctumModInt a, SanctumModInt b) {
    return sanctum_new(kamar_taj_mod(a.val, b.val, a.mod), a.mod);
}

SanctumModInt sanctum_sub(SanctumModInt a, SanctumModInt b) {
    return sanctum_new(kamar_taj_sub(a.val, b.val, a.mod), a.mod);
}

SanctumModInt sanctum_mul(SanctumModInt a, SanctumModInt b) {
    return sanctum_new(kamar_taj_mul(a.val, b.val, a.mod), a.mod);
}

SanctumModInt sanctum_pow(SanctumModInt base, uint64_t exp) {
    return sanctum_new(time_stone_pow(base.val, exp, base.mod), base.mod);
}

SanctumModInt sanctum_inv(SanctumModInt x) {
    return sanctum_new(dormammu_inverse(x.val, x.mod), x.mod);
}

SanctumModInt sanctum_div(SanctumModInt a, SanctumModInt b) {
    SanctumModInt inv_b = sanctum_inv(b);
    return sanctum_mul(a, inv_b);
}
```

---

### 4.4 Solutions alternatives acceptees

#### Alternative 1 : Exponentiation iterative avec tableau de bits

```rust
pub fn time_stone_pow_alt(base: i64, exp: u64, m: i64) -> i64 {
    if m <= 1 { return 0; }

    let mut powers: Vec<i64> = Vec::new();
    let mut current = ((base % m) + m) % m;
    let mut e = exp;

    // Precalcule base^(2^i)
    while e > 0 {
        powers.push(current);
        current = kamar_taj_mul(current, current, m);
        e >>= 1;
    }

    let mut result = 1i64;
    for (i, &power) in powers.iter().enumerate() {
        if (exp >> i) & 1 == 1 {
            result = kamar_taj_mul(result, power, m);
        }
    }
    result
}
```

#### Alternative 2 : Inverse via Extended GCD (plus general)

```rust
pub fn dormammu_inverse_ext(a: i64, m: i64) -> Option<i64> {
    fn ext_gcd(a: i64, b: i64) -> (i64, i64, i64) {
        if a == 0 { (b, 0, 1) }
        else {
            let (g, x, y) = ext_gcd(b % a, a);
            (g, y - (b / a) * x, x)
        }
    }

    let a = ((a % m) + m) % m;
    if a == 0 { return None; }

    let (g, x, _) = ext_gcd(a, m);
    if g != 1 { None }
    else { Some(((x % m) + m) % m) }
}
```

---

### 4.5 Solutions refusees (avec explications)

#### Refusee 1 : Sans gestion d'overflow

```rust
// REFUSE : Overflow pour grands nombres!
pub fn kamar_taj_mul_bad(a: i64, b: i64, m: i64) -> i64 {
    (a * b) % m  // OVERFLOW quand a,b > 10^9
}
// Pourquoi c'est faux : a * b peut depasser i64::MAX
```

#### Refusee 2 : Exponentiation naive O(n)

```rust
// REFUSE : Complexite O(exp) inacceptable!
pub fn time_stone_pow_naive(base: i64, exp: u64, m: i64) -> i64 {
    let mut result = 1;
    for _ in 0..exp {
        result = (result * base) % m;
    }
    result
}
// Pourquoi : Pour exp = 10^18, ca prendrait des millenaires
```

#### Refusee 3 : Inverse sans verification de zero

```rust
// REFUSE : Division par zero!
pub fn dormammu_inverse_bad(a: i64, m: i64) -> i64 {
    time_stone_pow(a, (m - 2) as u64, m)  // Pas de check si a == 0!
}
// Pourquoi : 0^(-1) n'existe pas
```

---

### 4.6 Solution bonus de reference (COMPLETE)

```rust
/// Theoreme des Restes Chinois
pub fn multiverse_crt(remainders: &[i64], moduli: &[i64]) -> Option<i64> {
    if remainders.len() != moduli.len() || remainders.is_empty() {
        return None;
    }

    let mut result: i64 = 0;
    let mut product: i64 = 1;

    for &m in moduli {
        product = kamar_taj_mul(product, m, i64::MAX);
    }

    for i in 0..remainders.len() {
        let mi = moduli[i];
        let ai = remainders[i];
        let bi = product / mi;

        let inv_bi = extended_inverse(bi % mi, mi)?;
        let term = kamar_taj_mul(
            kamar_taj_mul(ai, bi, product),
            inv_bi,
            product
        );
        result = kamar_taj_mod(result, term, product);
    }

    Some(result)
}

/// Racine primitive modulo p
pub fn darkhold_primitive_root(p: i64) -> Option<i64> {
    if p <= 1 { return None; }
    if p == 2 { return Some(1); }

    // Factoriser phi(p) = p - 1
    let mut phi = p - 1;
    let mut factors: Vec<i64> = Vec::new();

    let mut d = 2i64;
    while d * d <= phi {
        if phi % d == 0 {
            factors.push(d);
            while phi % d == 0 { phi /= d; }
        }
        d += 1;
    }
    if phi > 1 { factors.push(phi); }

    phi = p - 1;

    // Chercher g tel que g^((p-1)/q) != 1 pour tout facteur premier q
    for g in 2..p {
        let mut is_primitive = true;
        for &q in &factors {
            if time_stone_pow(g, (phi / q) as u64, p) == 1 {
                is_primitive = false;
                break;
            }
        }
        if is_primitive { return Some(g); }
    }
    None
}

/// Batch inverse en O(n)
pub fn batch_dormammu_inverse(arr: &[i64], m: i64) -> Vec<i64> {
    let n = arr.len();
    if n == 0 { return vec![]; }

    // Prefix products
    let mut prefix: Vec<i64> = vec![1; n];
    prefix[0] = ((arr[0] % m) + m) % m;
    for i in 1..n {
        let ai = ((arr[i] % m) + m) % m;
        prefix[i] = kamar_taj_mul(prefix[i-1], ai, m);
    }

    // Inverse of total product
    let inv_total = dormammu_inverse(prefix[n-1], m).unwrap_or(0);

    // Suffix inverses
    let mut result: Vec<i64> = vec![0; n];
    let mut suffix_inv = inv_total;

    for i in (0..n).rev() {
        if i == 0 {
            result[i] = suffix_inv;
        } else {
            result[i] = kamar_taj_mul(suffix_inv, prefix[i-1], m);
            let ai = ((arr[i] % m) + m) % m;
            suffix_inv = kamar_taj_mul(suffix_inv, ai, m);
        }
    }

    result
}
```

---

### 4.7 Solutions alternatives bonus (COMPLETES)

```rust
/// CRT iteratif (alternative)
pub fn multiverse_crt_iter(remainders: &[i64], moduli: &[i64]) -> Option<i64> {
    let mut r = remainders[0];
    let mut m = moduli[0];

    for i in 1..remainders.len() {
        let (g, x, _) = extended_gcd(m, moduli[i]);
        if (remainders[i] - r) % g != 0 { return None; }

        let lcm = m / g * moduli[i];
        r = (r + m * ((remainders[i] - r) / g) % (moduli[i] / g) * x) % lcm;
        r = ((r % lcm) + lcm) % lcm;
        m = lcm;
    }
    Some(r)
}
```

---

### 4.8 Solutions refusees bonus (COMPLETES)

```rust
// REFUSE : Batch inverse O(n log m) au lieu de O(n)
pub fn batch_inverse_bad(arr: &[i64], m: i64) -> Vec<i64> {
    arr.iter().map(|&a| dormammu_inverse(a, m).unwrap_or(0)).collect()
}
// Pourquoi : Chaque inverse coute O(log m), total O(n log m)
```

---

### 4.9 spec.json (ENGINE v22.1 - FORMAT STRICT)

```json
{
  "name": "kamar_taj_mod",
  "language": "rust,c",
  "type": "complet",
  "tier": 2,
  "tier_info": "Melange (mod_ops + fast_exp + fermat_inv + discrete_log + sqrt_mod)",
  "tags": ["modular", "number-theory", "cryptography", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "kamar_taj_mod",
    "prototype": "pub fn kamar_taj_mod(a: i64, b: i64, m: i64) -> i64",
    "return_type": "i64",
    "parameters": [
      {"name": "a", "type": "i64"},
      {"name": "b", "type": "i64"},
      {"name": "m", "type": "i64"}
    ]
  },

  "driver": {
    "reference": "pub fn ref_kamar_taj_mod(a: i64, b: i64, m: i64) -> i64 { if m <= 0 { return 0; } let a = ((a % m) + m) % m; let b = ((b % m) + m) % m; ((a + b) % m) }",

    "edge_cases": [
      {
        "name": "basic_add",
        "args": [7, 5, 3],
        "expected": 0,
        "is_trap": false
      },
      {
        "name": "negative_input",
        "args": [-3, 5, 7],
        "expected": 2,
        "is_trap": true,
        "trap_explanation": "Nombres negatifs doivent etre normalises"
      },
      {
        "name": "zero_modulus",
        "args": [5, 3, 0],
        "expected": 0,
        "is_trap": true,
        "trap_explanation": "Division par zero interdite"
      },
      {
        "name": "large_overflow",
        "args": [1000000000000000000, 1000000000000000000, 1000000007],
        "expected": "computed",
        "is_trap": true,
        "trap_explanation": "Overflow si pas de cast i128"
      },
      {
        "name": "both_negative",
        "args": [-10, -5, 7],
        "expected": 6,
        "is_trap": true,
        "trap_explanation": "Double negatif"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 5000,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {"min": -1000000000000, "max": 1000000000000}
        },
        {
          "type": "int",
          "param_index": 1,
          "params": {"min": -1000000000000, "max": 1000000000000}
        },
        {
          "type": "int",
          "param_index": 2,
          "params": {"min": 1, "max": 1000000007}
        }
      ]
    }
  },

  "additional_functions": [
    {
      "name": "time_stone_pow",
      "prototype": "pub fn time_stone_pow(base: i64, exp: u64, m: i64) -> i64",
      "edge_cases": [
        {"name": "exp_zero", "args": [5, 0, 7], "expected": 1, "is_trap": true},
        {"name": "base_zero", "args": [0, 10, 7], "expected": 0},
        {"name": "mod_one", "args": [5, 10, 1], "expected": 0, "is_trap": true}
      ]
    },
    {
      "name": "dormammu_inverse",
      "prototype": "pub fn dormammu_inverse(a: i64, m: i64) -> Option<i64>",
      "edge_cases": [
        {"name": "inv_basic", "args": [3, 7], "expected": 5},
        {"name": "inv_zero", "args": [0, 7], "expected": null, "is_trap": true},
        {"name": "inv_one", "args": [1, 7], "expected": 1}
      ]
    },
    {
      "name": "ancient_one_log",
      "prototype": "pub fn ancient_one_log(a: i64, b: i64, m: i64) -> Option<u64>",
      "edge_cases": [
        {"name": "log_basic", "args": [2, 3, 5], "expected": 3},
        {"name": "log_none", "args": [2, 3, 7], "expected": null, "is_trap": true}
      ]
    },
    {
      "name": "mirror_sqrt",
      "prototype": "pub fn mirror_sqrt(a: i64, p: i64) -> Option<i64>",
      "edge_cases": [
        {"name": "sqrt_basic", "args": [2, 7], "expected": 3},
        {"name": "sqrt_non_qr", "args": [3, 7], "expected": null, "is_trap": true}
      ]
    },
    {
      "name": "wong_lucas",
      "prototype": "pub fn wong_lucas(n: u64, k: u64, p: u64) -> u64",
      "edge_cases": [
        {"name": "lucas_basic", "args": [5, 2, 13], "expected": 10},
        {"name": "lucas_large", "args": [100, 10, 13], "expected": 10}
      ]
    }
  ],

  "norm": {
    "allowed_functions": ["sqrt", "HashMap::new", "HashMap::insert", "HashMap::get"],
    "forbidden_functions": ["pow", "powf"],
    "check_security": true,
    "check_memory": true,
    "check_overflow": true,
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes (minimum 5)

#### Mutant A (Boundary) : Exposant zero retourne zero

```rust
/* Mutant A (Boundary) : exp == 0 retourne 0 au lieu de 1 */
pub fn time_stone_pow_mutant_a(base: i64, mut exp: u64, m: i64) -> i64 {
    if m <= 1 { return 0; }
    // BUG: Manque le cas exp == 0 retourne 1

    let mut result: i64 = 0;  // BUG: devrait etre 1
    let mut base = ((base % m) + m) % m;

    while exp > 0 {
        if exp & 1 == 1 {
            result = kamar_taj_mul(result, base, m);
        }
        base = kamar_taj_mul(base, base, m);
        exp >>= 1;
    }
    result
}
// Pourquoi c'est faux : x^0 = 1 pour tout x != 0, pas 0
// Ce qui etait pense : "0 est la valeur neutre" (FAUX, c'est 1 pour *)
```

#### Mutant B (Safety) : Pas de verification modulus <= 0

```rust
/* Mutant B (Safety) : Pas de check m <= 0 */
pub fn kamar_taj_mod_mutant_b(a: i64, b: i64, m: i64) -> i64 {
    // BUG: Manque if m <= 0 { return 0; }
    let a = ((a % m) + m) % m;  // Division par zero si m == 0!
    let b = ((b % m) + m) % m;
    (a + b) % m
}
// Pourquoi c'est faux : Division par zero = crash ou UB
// Ce qui etait pense : "L'appelant verifiera"
```

#### Mutant C (Resource) : Overflow sans i128

```rust
/* Mutant C (Resource) : Overflow dans multiplication */
pub fn kamar_taj_mul_mutant_c(a: i64, b: i64, m: i64) -> i64 {
    if m <= 0 { return 0; }
    let a = ((a % m) + m) % m;
    let b = ((b % m) + m) % m;
    (a * b) % m  // BUG: Overflow si a,b > 10^9!
}
// Pourquoi c'est faux : i64::MAX ~ 9.2 * 10^18, mais (10^9)^2 = 10^18
// Avec des valeurs > 3*10^9, on deborde
// Ce qui etait pense : "i64 c'est assez grand"
```

#### Mutant D (Logic) : Inverse utilise p-1 au lieu de p-2

```rust
/* Mutant D (Logic) : Mauvais exposant pour Fermat */
pub fn dormammu_inverse_mutant_d(a: i64, m: i64) -> Option<i64> {
    if m <= 1 { return None; }
    let a = ((a % m) + m) % m;
    if a == 0 { return None; }

    // BUG: Devrait etre m-2, pas m-1!
    Some(time_stone_pow(a, (m - 1) as u64, m))  // Retourne 1, pas l'inverse!
}
// Pourquoi c'est faux : Fermat dit a^(p-1) = 1, donc a^(-1) = a^(p-2)
// Ce qui etait pense : Confusion avec le theoreme
```

#### Mutant E (Return) : Log discret retourne 0 au lieu de None

```rust
/* Mutant E (Return) : Retourne 0 quand pas de solution */
pub fn ancient_one_log_mutant_e(a: i64, b: i64, m: i64) -> Option<u64> {
    if m <= 1 { return Some(0); }  // BUG: devrait etre None

    let a = ((a % m) + m) % m;
    let b = ((b % m) + m) % m;

    // ... algorithme ...

    // BUG: Retourne Some(0) au lieu de None quand pas trouve
    Some(0)
}
// Pourquoi c'est faux : Some(0) signifie a^0 = 1, mais 1 != b en general
// Ce qui etait pense : "0 c'est comme pas de solution"
```
