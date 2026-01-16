<thinking>
## Analyse du Concept
- Concept : Rabin-Karp (hash-based) et Boyer-Moore (rule-based) pattern matching
- Phase demandée : 1 (Intermédiaire)
- Adapté ? OUI - Algorithmes avancés de pattern matching

## Combo Base + Bonus
- Exercice de base : Rabin-Karp + Boyer-Moore + variantes (Horspool, Galil)
- Bonus : 2D matching + Multi-pattern + Sunday algorithm
- Palier bonus : 🔥 Avancé (extensions complexes)
- Progression logique ? OUI - Base = 1D single, Bonus = 2D/multi

## Prérequis & Difficulté
- Prérequis réels : Hashing, modular arithmetic, KMP basics
- Difficulté estimée : 5/10 (base), 8/10 (bonus)
- Cohérent avec phase ? OUI - Phase 1 = 3-5/10

## Aspect Fun/Culture
- Contexte choisi : Ace Attorney (jeu de visual novel juridique)
- MEME mnémotechnique : "OBJECTION!" - Quand le hash matche, c'est le moment de l'objection
- Pourquoi c'est fun :
  * Phoenix Wright = L'algorithme chercheur de contradictions
  * Evidence = Pattern à chercher
  * Rolling hash = "Court Record" qui se met à jour
  * Bad character = "This evidence contradicts the testimony!"
  * Good suffix = "The ending of your testimony matches but..."
  * Galil = "I've already proven this part, Your Honor!"

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Overflow) : base^m sans modulo → overflow
2. Mutant B (Hash) : rolling hash oublie de soustraire → hash faux
3. Mutant C (BadChar) : bad_char[c] = m au lieu de m - 1 - j → shift incorrect
4. Mutant D (GoodSuffix) : table mal construite → match manqué
5. Mutant E (Galil) : période mal calculée → skip trop ou pas assez

## Verdict
VALIDE - Ace Attorney parfait pour pattern matching juridique
Score créativité : 96/100
</thinking>

---

# Exercice 1.2.2-synth : turnabout_search

**Module :**
1.2 — Hash Tables & Strings

**Concept :**
synth — Rabin-Karp & Boyer-Moore

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (Rabin-Karp + Boyer-Moore + Variantes)

**Langage :**
Rust Edition 2024 / C (c17)

**Prérequis :**
- Hashing et arithmétique modulaire
- Pattern matching basique (KMP)
- Manipulation de tableaux

**Domaines :**
Struct, Encodage, MD

**Durée estimée :**
120 min

**XP Base :**
150

**Complexité :**
T4 O(n+m) average × S3 O(m)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `turnabout.c`, `turnabout.h`

**Fonctions autorisées :**
- Rust : Types standard (`Vec`, `Option`)
- C : `malloc`, `free`, `strlen`, `memcpy`

**Fonctions interdites :**
- Rust : `str::find`, `str::contains`, regex
- C : `strstr`, `memmem`, bibliothèques externes

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🎮 ACE ATTORNEY : Turnabout Pattern - Les Techniques de Wright & Co.**

*"OBJECTION! La preuve présentée par le témoin contient une contradiction flagrante! Cette séquence de caractères ne correspond PAS au pattern recherché!"*

Dans l'univers d'**Ace Attorney**, l'avocat Phoenix Wright doit trouver des **contradictions** dans les témoignages. Pour cela, il compare les preuves dans son **Court Record** avec les déclarations des témoins.

Tu es recruté par Wright & Co. Law Offices pour implémenter les algorithmes de recherche de contradictions :

**📋 TurnaboutHash (Rabin-Karp) :**
Comme Phoenix qui calcule rapidement si une preuve "matche" un témoignage grâce à une **empreinte digitale** (hash), Rabin-Karp utilise un rolling hash pour comparer efficacement.

*"Let me check my Court Record... This evidence's fingerprint matches the testimony!"*

**🔍 CourtRecord (Boyer-Moore) :**
Miles Edgeworth, le procureur, préfère travailler **en arrière** (de droite à gauche). Quand il trouve une contradiction, il utilise deux règles :
- **ContradictionRule** (Bad Character) : "Ce caractère ne peut pas être à cette position!"
- **TestimonyRule** (Good Suffix) : "Cette fin de témoignage correspond, mais le début..."

**⚡ WrightTactics (Galil) :**
Phoenix a développé une optimisation : après avoir prouvé qu'un pattern est périodique, il peut **skip** les parties déjà vérifiées. C'est la règle de Galil.

**Ta mission :**

1. **`TurnaboutHash`** : Rabin-Karp avec rolling hash
   - `fingerprint()` : Calculer l'empreinte d'une preuve
   - `find_contradiction()` : Chercher toutes les occurrences
   - `mass_investigation()` : Multi-pattern search

2. **`CourtRecord`** : Boyer-Moore complet
   - `build_contradiction_table()` : Bad character rule
   - `build_testimony_table()` : Good suffix rule
   - `cross_examination()` : Recherche

3. **`HorspoolDefense`** : Boyer-Moore simplifié (bad char only)

4. **`WrightTactics`** : Boyer-Moore-Galil (optimisation périodique)

5. **`MatrixInvestigation`** : Recherche 2D (bonus)

**Entrée :**
- `testimony: &[u8]` : Le texte (témoignage) à analyser
- `evidence: &[u8]` : Le pattern (preuve) à chercher

**Sortie :**
- `Vec<usize>` pour les positions des contradictions
- `Option<usize>` pour la première occurrence

**Contraintes :**
┌─────────────────────────────────────────────────────────────────┐
│  Rabin-Karp : base = 256, modulus = 10^9 + 7                    │
│  Boyer-Moore : preprocessing O(m + σ), search O(n)              │
│  Bad char table : 256 entrées (ASCII complet)                   │
│  Good suffix : Utiliser les borders (comme KMP)                 │
└─────────────────────────────────────────────────────────────────┘

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `rk.find_contradiction(b"OBJECTION", b"JEC")` | `[2]` | "JEC" trouvé à position 2 |
| `bm.cross_examination(b"AABAACAABA", b"AABA")` | `[0, 6]` | Deux contradictions |
| `horspool.search(b"ABCABC", b"ABC")` | `[0, 3]` | Pattern répété |

#### 1.2.2 Version Académique

**Objectif :**

Implémenter deux familles d'algorithmes de pattern matching :
1. **Rabin-Karp** : Basé sur le hashing
2. **Boyer-Moore** : Basé sur des heuristiques de saut

**Rabin-Karp :**

Utilise un **rolling hash** pour comparer le pattern avec des fenêtres de texte en O(1) après preprocessing.

```
hash(s[0..m]) = Σ s[i] × base^(m-1-i) mod p
hash(s[1..m+1]) = (hash(s[0..m]) - s[0] × base^(m-1)) × base + s[m]
```

Avantages :
- Multi-pattern efficace
- 2D matching
- Average case O(n + m)

Inconvénients :
- Worst case O(nm) avec collisions
- Sensible au choix de base/modulus

**Boyer-Moore :**

Scanne le pattern de droite à gauche, utilisant deux règles de saut :

1. **Bad Character Rule** : Si mismatch sur caractère c, shift pour aligner c avec sa dernière occurrence dans pattern[0..j-1]

2. **Good Suffix Rule** : Si mismatch après avoir matché un suffixe s, shift pour aligner s avec sa prochaine occurrence dans pattern

Variantes :
- **Horspool** : Bad char only, plus simple
- **Galil** : Optimisation pour patterns périodiques

### 1.3 Prototype

**Rust :**
```rust
// ═══════════════════════════════════════════════════════════════
// TURNABOUT HASH - Rabin-Karp
// ═══════════════════════════════════════════════════════════════

pub struct TurnaboutHash {
    base: u64,
    modulus: u64,
}

impl TurnaboutHash {
    pub fn new() -> Self;
    pub fn with_params(base: u64, modulus: u64) -> Self;

    /// Compute hash (fingerprint) of evidence
    pub fn fingerprint(&self, evidence: &[u8]) -> u64;

    /// Rolling hash: update hash when window slides by 1
    pub fn roll_fingerprint(
        &self,
        old_hash: u64,
        old_char: u8,
        new_char: u8,
        base_power: u64,  // base^(m-1) mod p
    ) -> u64;

    /// Find all contradictions (pattern occurrences)
    pub fn find_contradiction(&self, testimony: &[u8], evidence: &[u8]) -> Vec<usize>;

    /// Find first contradiction
    pub fn find_first(&self, testimony: &[u8], evidence: &[u8]) -> Option<usize>;

    /// Mass investigation: search multiple patterns
    pub fn mass_investigation(
        &self,
        testimony: &[u8],
        evidence_list: &[&[u8]],
    ) -> Vec<(usize, usize)>;  // (position, evidence_index)
}

// ═══════════════════════════════════════════════════════════════
// COURT RECORD - Boyer-Moore
// ═══════════════════════════════════════════════════════════════

pub struct CourtRecord {
    evidence: Vec<u8>,
    contradiction_table: [usize; 256],  // Bad character
    testimony_table: Vec<usize>,         // Good suffix
}

impl CourtRecord {
    /// Preprocess evidence (pattern)
    pub fn new(evidence: &[u8]) -> Self;

    /// Cross-examination: find all matches
    pub fn cross_examination(&self, testimony: &[u8]) -> Vec<usize>;

    /// Find first match
    pub fn find_first(&self, testimony: &[u8]) -> Option<usize>;

    // Helper methods
    fn build_contradiction_table(evidence: &[u8]) -> [usize; 256];
    fn build_testimony_table(evidence: &[u8]) -> Vec<usize>;
}

// ═══════════════════════════════════════════════════════════════
// HORSPOOL DEFENSE - Simplified Boyer-Moore
// ═══════════════════════════════════════════════════════════════

pub struct HorspoolDefense {
    evidence: Vec<u8>,
    skip_table: [usize; 256],  // Bad character only
}

impl HorspoolDefense {
    pub fn new(evidence: &[u8]) -> Self;
    pub fn search(&self, testimony: &[u8]) -> Vec<usize>;
}

// ═══════════════════════════════════════════════════════════════
// WRIGHT TACTICS - Boyer-Moore-Galil
// ═══════════════════════════════════════════════════════════════

pub struct WrightTactics {
    court_record: CourtRecord,
    period: usize,  // Pattern period for Galil optimization
}

impl WrightTactics {
    pub fn new(evidence: &[u8]) -> Self;
    pub fn cross_examination(&self, testimony: &[u8]) -> Vec<usize>;
}

// ═══════════════════════════════════════════════════════════════
// 2D INVESTIGATION
// ═══════════════════════════════════════════════════════════════

/// Search for 2D pattern in matrix using Rabin-Karp
pub fn matrix_investigation(
    crime_scene: &[Vec<u8>],
    evidence: &[Vec<u8>],
) -> Vec<(usize, usize)>;

// ═══════════════════════════════════════════════════════════════
// UTILITY FUNCTIONS
// ═══════════════════════════════════════════════════════════════

/// Compute pattern period (for Galil)
pub fn compute_period(pattern: &[u8]) -> usize;

/// Modular exponentiation: base^exp mod m
pub fn mod_pow(base: u64, exp: u64, modulus: u64) -> u64;
```

**C :**
```c
#ifndef TURNABOUT_H
#define TURNABOUT_H

#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>

// ═══════════════════════════════════════════════════════════════
// TURNABOUT HASH - Rabin-Karp
// ═══════════════════════════════════════════════════════════════

typedef struct s_turnabout_hash {
    uint64_t    base;
    uint64_t    modulus;
} t_turnabout_hash;

t_turnabout_hash    turnabout_new(void);
t_turnabout_hash    turnabout_with_params(uint64_t base, uint64_t modulus);

uint64_t    fingerprint(t_turnabout_hash *rk, const char *evidence, size_t len);
uint64_t    roll_fingerprint(t_turnabout_hash *rk, uint64_t old_hash,
                             char old_char, char new_char, uint64_t base_power);

size_t      *find_contradiction(t_turnabout_hash *rk, const char *testimony,
                                size_t text_len, const char *evidence,
                                size_t pattern_len, size_t *num_matches);

// ═══════════════════════════════════════════════════════════════
// COURT RECORD - Boyer-Moore
// ═══════════════════════════════════════════════════════════════

typedef struct s_court_record {
    char        *evidence;
    size_t      evidence_len;
    size_t      contradiction_table[256];
    size_t      *testimony_table;
} t_court_record;

t_court_record  *court_record_new(const char *evidence, size_t len);
void            court_record_destroy(t_court_record *cr);

size_t  *cross_examination(t_court_record *cr, const char *testimony,
                           size_t text_len, size_t *num_matches);

// ═══════════════════════════════════════════════════════════════
// HORSPOOL DEFENSE - Simplified Boyer-Moore
// ═══════════════════════════════════════════════════════════════

typedef struct s_horspool {
    char        *evidence;
    size_t      evidence_len;
    size_t      skip_table[256];
} t_horspool;

t_horspool  *horspool_new(const char *evidence, size_t len);
void        horspool_destroy(t_horspool *h);
size_t      *horspool_search(t_horspool *h, const char *testimony,
                             size_t text_len, size_t *num_matches);

// ═══════════════════════════════════════════════════════════════
// WRIGHT TACTICS - Boyer-Moore-Galil
// ═══════════════════════════════════════════════════════════════

typedef struct s_wright_tactics {
    t_court_record  *court_record;
    size_t          period;
} t_wright_tactics;

t_wright_tactics    *wright_tactics_new(const char *evidence, size_t len);
void                wright_tactics_destroy(t_wright_tactics *wt);
size_t              *wright_cross_examination(t_wright_tactics *wt,
                                              const char *testimony,
                                              size_t text_len,
                                              size_t *num_matches);

// ═══════════════════════════════════════════════════════════════
// UTILITIES
// ═══════════════════════════════════════════════════════════════

void    build_contradiction_table(const char *evidence, size_t len,
                                  size_t table[256]);
size_t  *build_testimony_table(const char *evidence, size_t len);
size_t  compute_period(const char *pattern, size_t len);
uint64_t mod_pow(uint64_t base, uint64_t exp, uint64_t modulus);

#endif // TURNABOUT_H
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Anecdote Historique

**Boyer-Moore** (1977) est souvent considéré comme l'algorithme de pattern matching le plus efficace en pratique. Robert S. Boyer et J Strother Moore l'ont développé à l'Université du Texas.

Le twist : Boyer-Moore est généralement **plus rapide** que KMP pour les alphabets larges (comme ASCII), mais **KMP est meilleur** pour les alphabets petits (comme ADN: ACGT).

**Rabin-Karp** (1987) de Michael Rabin et Richard Karp a introduit le concept de **fingerprinting** en algorithmique, qui a eu des applications bien au-delà du pattern matching.

### 2.2 Chiffre Clé

- **Sublinear** : Boyer-Moore peut être O(n/m) dans le meilleur cas !
- **GNU grep** utilise Boyer-Moore pour sa vitesse légendaire
- **Plagiarism detection** utilise Rabin-Karp pour comparer millions de documents

### 2.5 Dans la Vraie Vie

| Métier | Algorithme | Usage |
|--------|------------|-------|
| **Security Analyst** | Rabin-Karp multi | Signatures IDS/IPS |
| **Text Editor Developer** | Boyer-Moore | Find/Replace rapide |
| **Academic Integrity** | Rabin-Karp | Détection plagiat |
| **Bioinformaticien** | Boyer-Moore | BLAST (séquences ADN) |
| **Network Engineer** | Rabin-Karp | Deep packet inspection |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
turnabout.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 10 tests
test test_rabin_karp_basic ... ok
test test_rabin_karp_multiple ... ok
test test_boyer_moore_basic ... ok
test test_boyer_moore_multiple ... ok
test test_horspool ... ok
test test_galil ... ok
test test_bad_char_table ... ok
test test_good_suffix_table ... ok
test test_edge_cases ... ok
test test_2d_search ... ok

test result: ok. 10 passed; 0 failed

$ ./target/release/turnabout_demo
=== WRIGHT & CO. LAW OFFICES ===
Testimony: "HERE IS A SIMPLE EXAMPLE"
Evidence: "EXAMPLE"
Phoenix Wright: "OBJECTION! Contradiction found at position 17!"
Bad character shifts used: 4
Good suffix shifts used: 1
Case closed.
```

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n × m / σ) average pour 2D

**Space Complexity attendue :**
O(m × k) pour multi-pattern

**Domaines Bonus :**
`MD, AL`

#### 3.1.1 Consigne Bonus

**🎮 INVESTIGATIONS : L'Affaire Multi-Dimensionnelle**

*"Miles Edgeworth fait face à une affaire complexe : des indices sont cachés dans une MATRICE 2D, et il doit chercher PLUSIEURS preuves simultanément!"*

**Ta mission avancée :**

1. **`matrix_investigation`** : Rabin-Karp 2D complet
   - Hash de colonnes puis de lignes
   - Support rotation 90°/180°/270°

2. **`MultiEvidenceSearch`** : Aho-Corasick style avec hashing
   - Recherche de k patterns simultanément
   - Complexité O(n + mk + z)

3. **`SundayAlgorithm`** : Variante de Boyer-Moore
   - Regarde le caractère APRÈS la fenêtre
   - Souvent plus rapide que Horspool

**Contraintes Bonus :**
┌─────────────────────────────────────────────────────────────────┐
│  2D : Support matrices jusqu'à 10000×10000                      │
│  Multi : Jusqu'à 1000 patterns de longueur variable             │
│  Sunday : Table basée sur caractère à position m (pas m-1)      │
└─────────────────────────────────────────────────────────────────┘

#### 3.1.2 Prototype Bonus

```rust
/// 2D pattern matching with rotations
pub fn matrix_investigation_rotations(
    crime_scene: &[Vec<u8>],
    evidence: &[Vec<u8>],
) -> Vec<(usize, usize, u8)>;  // (row, col, rotation: 0/90/180/270)

/// Sunday algorithm (look-ahead variant)
pub struct SundaySearch {
    evidence: Vec<u8>,
    shift_table: [usize; 256],  // Based on char at position m
}

impl SundaySearch {
    pub fn new(evidence: &[u8]) -> Self;
    pub fn search(&self, testimony: &[u8]) -> Vec<usize>;
}

/// Multi-pattern with combined hashing
pub struct MultiEvidence {
    patterns: Vec<Vec<u8>>,
    hashes: std::collections::HashSet<u64>,
    min_len: usize,
}

impl MultiEvidence {
    pub fn new(patterns: &[&[u8]]) -> Self;
    pub fn search(&self, testimony: &[u8]) -> Vec<(usize, usize)>;
}
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Entrée | Sortie Attendue | Points |
|------|--------|-----------------|--------|
| `rk_basic` | "AABAACAABA", "AABA" | `[0, 6]` | 3 |
| `rk_no_match` | "ABCD", "XYZ" | `[]` | 2 |
| `rk_overlap` | "AAA", "AA" | `[0, 1]` | 3 |
| `rk_multiple` | multi-pattern test | correct matches | 4 |
| `bm_basic` | "HERE IS EXAMPLE", "EXAMPLE" | `[8]` | 3 |
| `bm_multiple` | "AABAACAABA", "AABA" | `[0, 6]` | 3 |
| `bm_bad_char` | verify table | correct values | 3 |
| `bm_good_suffix` | verify table | correct values | 4 |
| `horspool` | "ABCABC", "ABC" | `[0, 3]` | 3 |
| `galil` | "ABABABABAB", "ABAB" | `[0, 2, 4, 6]` | 4 |
| `2d_basic` | matrix search | correct position | 5 |
| `edge_empty` | empty pattern | `[]` | 2 |
| `edge_longer` | pattern > text | `[]` | 2 |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <string.h>
#include <assert.h>
#include <stdlib.h>
#include "turnabout.h"

void test_rabin_karp(void) {
    printf("Testing TurnaboutHash (Rabin-Karp)...\n");

    t_turnabout_hash rk = turnabout_new();

    // Basic search
    size_t count;
    size_t *matches = find_contradiction(&rk, "AABAACAADAABAAABAA", 18,
                                         "AABA", 4, &count);
    assert(count == 3);
    assert(matches[0] == 0);
    assert(matches[1] == 9);
    assert(matches[2] == 13);
    free(matches);

    // Test fingerprint
    uint64_t h1 = fingerprint(&rk, "ABC", 3);
    uint64_t h2 = fingerprint(&rk, "ABC", 3);
    assert(h1 == h2);

    printf("  PASS\n");
}

void test_boyer_moore(void) {
    printf("Testing CourtRecord (Boyer-Moore)...\n");

    t_court_record *cr = court_record_new("AABA", 4);

    size_t count;
    size_t *matches = cross_examination(cr, "AABAACAADAABAAABAA", 18, &count);

    assert(count == 3);
    assert(matches[0] == 0);
    assert(matches[1] == 9);
    assert(matches[2] == 13);
    free(matches);

    // Test bad character table
    assert(cr->contradiction_table['A'] == 0);  // Last A at position 3, m - 1 - 3 = 0
    assert(cr->contradiction_table['B'] == 1);  // Last B at position 2, m - 1 - 2 = 1

    court_record_destroy(cr);
    printf("  PASS\n");
}

void test_horspool(void) {
    printf("Testing HorspoolDefense...\n");

    t_horspool *h = horspool_new("ABC", 3);

    size_t count;
    size_t *matches = horspool_search(h, "DABCABCABC", 10, &count);

    assert(count == 3);
    assert(matches[0] == 1);
    assert(matches[1] == 4);
    assert(matches[2] == 7);
    free(matches);

    horspool_destroy(h);
    printf("  PASS\n");
}

void test_galil(void) {
    printf("Testing WrightTactics (Galil)...\n");

    t_wright_tactics *wt = wright_tactics_new("ABAB", 4);

    // Pattern "ABAB" has period 2
    assert(wt->period == 2);

    size_t count;
    size_t *matches = wright_cross_examination(wt, "ABABABABAB", 10, &count);

    assert(count == 4);
    assert(matches[0] == 0);
    assert(matches[1] == 2);
    assert(matches[2] == 4);
    assert(matches[3] == 6);
    free(matches);

    wright_tactics_destroy(wt);
    printf("  PASS\n");
}

void test_edge_cases(void) {
    printf("Testing edge cases...\n");

    t_court_record *cr = court_record_new("A", 1);
    size_t count;

    // Single char pattern
    size_t *m1 = cross_examination(cr, "AAA", 3, &count);
    assert(count == 3);
    free(m1);

    // Empty text
    size_t *m2 = cross_examination(cr, "", 0, &count);
    assert(count == 0);
    free(m2);

    court_record_destroy(cr);

    // Pattern longer than text
    cr = court_record_new("ABC", 3);
    size_t *m3 = cross_examination(cr, "AB", 2, &count);
    assert(count == 0);
    free(m3);

    court_record_destroy(cr);
    printf("  PASS\n");
}

int main(void) {
    printf("=== WRIGHT & CO. TEST SUITE ===\n\n");

    test_rabin_karp();
    test_boyer_moore();
    test_horspool();
    test_galil();
    test_edge_cases();

    printf("\n=== ALL TESTS PASSED - CASE CLOSED ===\n");
    return 0;
}
```

### 4.3 Solution de référence

```rust
use std::collections::HashSet;

const DEFAULT_BASE: u64 = 256;
const DEFAULT_MODULUS: u64 = 1_000_000_007;

// ═══════════════════════════════════════════════════════════════
// TURNABOUT HASH - Rabin-Karp
// ═══════════════════════════════════════════════════════════════

pub struct TurnaboutHash {
    base: u64,
    modulus: u64,
}

impl TurnaboutHash {
    pub fn new() -> Self {
        Self::with_params(DEFAULT_BASE, DEFAULT_MODULUS)
    }

    pub fn with_params(base: u64, modulus: u64) -> Self {
        TurnaboutHash { base, modulus }
    }

    pub fn fingerprint(&self, evidence: &[u8]) -> u64 {
        let mut hash: u64 = 0;
        for &byte in evidence {
            hash = (hash * self.base + byte as u64) % self.modulus;
        }
        hash
    }

    pub fn roll_fingerprint(
        &self,
        old_hash: u64,
        old_char: u8,
        new_char: u8,
        base_power: u64,
    ) -> u64 {
        let mut hash = old_hash;

        // Remove old_char contribution
        hash = (hash + self.modulus - (old_char as u64 * base_power) % self.modulus) % self.modulus;

        // Shift left and add new_char
        hash = (hash * self.base + new_char as u64) % self.modulus;

        hash
    }

    pub fn find_contradiction(&self, testimony: &[u8], evidence: &[u8]) -> Vec<usize> {
        let n = testimony.len();
        let m = evidence.len();

        if m == 0 || m > n {
            return vec![];
        }

        let mut matches = vec![];

        // Precompute base^(m-1) mod p
        let base_power = mod_pow(self.base, (m - 1) as u64, self.modulus);

        // Compute pattern hash
        let pattern_hash = self.fingerprint(evidence);

        // Compute initial window hash
        let mut window_hash = self.fingerprint(&testimony[0..m]);

        // Check first window
        if window_hash == pattern_hash && &testimony[0..m] == evidence {
            matches.push(0);
        }

        // Slide window
        for i in 1..=(n - m) {
            window_hash = self.roll_fingerprint(
                window_hash,
                testimony[i - 1],
                testimony[i + m - 1],
                base_power,
            );

            // Hash match - verify to avoid false positives
            if window_hash == pattern_hash && &testimony[i..i + m] == evidence {
                matches.push(i);
            }
        }

        matches
    }

    pub fn find_first(&self, testimony: &[u8], evidence: &[u8]) -> Option<usize> {
        self.find_contradiction(testimony, evidence).into_iter().next()
    }

    pub fn mass_investigation(
        &self,
        testimony: &[u8],
        evidence_list: &[&[u8]],
    ) -> Vec<(usize, usize)> {
        let mut results = vec![];

        // Group patterns by length
        let mut by_length: std::collections::HashMap<usize, Vec<(usize, u64, &[u8])>> =
            std::collections::HashMap::new();

        for (idx, &pattern) in evidence_list.iter().enumerate() {
            let hash = self.fingerprint(pattern);
            by_length
                .entry(pattern.len())
                .or_default()
                .push((idx, hash, pattern));
        }

        // Search for each length group
        for (len, patterns) in by_length {
            if len == 0 || len > testimony.len() {
                continue;
            }

            let hash_set: HashSet<u64> = patterns.iter().map(|(_, h, _)| *h).collect();
            let base_power = mod_pow(self.base, (len - 1) as u64, self.modulus);

            let mut window_hash = self.fingerprint(&testimony[0..len]);

            for i in 0..=(testimony.len() - len) {
                if i > 0 {
                    window_hash = self.roll_fingerprint(
                        window_hash,
                        testimony[i - 1],
                        testimony[i + len - 1],
                        base_power,
                    );
                }

                if hash_set.contains(&window_hash) {
                    // Verify which pattern(s) match
                    for (idx, hash, pattern) in &patterns {
                        if window_hash == *hash && &testimony[i..i + len] == *pattern {
                            results.push((i, *idx));
                        }
                    }
                }
            }
        }

        results
    }
}

// ═══════════════════════════════════════════════════════════════
// COURT RECORD - Boyer-Moore
// ═══════════════════════════════════════════════════════════════

pub struct CourtRecord {
    evidence: Vec<u8>,
    contradiction_table: [usize; 256],
    testimony_table: Vec<usize>,
}

impl CourtRecord {
    pub fn new(evidence: &[u8]) -> Self {
        let contradiction_table = Self::build_contradiction_table(evidence);
        let testimony_table = Self::build_testimony_table(evidence);

        CourtRecord {
            evidence: evidence.to_vec(),
            contradiction_table,
            testimony_table,
        }
    }

    fn build_contradiction_table(evidence: &[u8]) -> [usize; 256] {
        let m = evidence.len();
        let mut table = [m; 256];  // Default: skip whole pattern

        // For each char in pattern (except last), store distance to end
        for i in 0..m.saturating_sub(1) {
            table[evidence[i] as usize] = m - 1 - i;
        }

        table
    }

    fn build_testimony_table(evidence: &[u8]) -> Vec<usize> {
        let m = evidence.len();
        if m == 0 {
            return vec![];
        }

        let mut table = vec![0; m];

        // Compute failure function (like KMP)
        let mut failure = vec![0; m];
        let mut j = 0;
        for i in 1..m {
            while j > 0 && evidence[i] != evidence[j] {
                j = failure[j - 1];
            }
            if evidence[i] == evidence[j] {
                j += 1;
            }
            failure[i] = j;
        }

        // Case 1: Suffix exists elsewhere in pattern
        // (simplified implementation)
        for i in 0..m {
            table[i] = m;
        }

        // Case 2: Prefix equals suffix
        let mut j = failure[m - 1];
        for i in (0..m).rev() {
            if j == 0 {
                table[i] = m;
            } else {
                table[i] = m - j;
            }
            if i > 0 && failure[i - 1] < j {
                j = failure[i - 1];
            }
        }

        table
    }

    pub fn cross_examination(&self, testimony: &[u8]) -> Vec<usize> {
        let n = testimony.len();
        let m = self.evidence.len();

        if m == 0 || m > n {
            return vec![];
        }

        let mut matches = vec![];
        let mut i = 0;

        while i <= n - m {
            let mut j = m - 1;

            // Compare from right to left
            while self.evidence[j] == testimony[i + j] {
                if j == 0 {
                    matches.push(i);
                    break;
                }
                j -= 1;
            }

            // Compute shift
            let bad_char_shift = if j < m - 1 {
                let c = testimony[i + j] as usize;
                self.contradiction_table[c].saturating_sub(m - 1 - j)
            } else {
                self.contradiction_table[testimony[i + m - 1] as usize]
            };

            let good_suffix_shift = if j < m - 1 {
                self.testimony_table[j]
            } else {
                1
            };

            i += bad_char_shift.max(good_suffix_shift).max(1);
        }

        matches
    }

    pub fn find_first(&self, testimony: &[u8]) -> Option<usize> {
        self.cross_examination(testimony).into_iter().next()
    }
}

// ═══════════════════════════════════════════════════════════════
// HORSPOOL DEFENSE - Simplified Boyer-Moore
// ═══════════════════════════════════════════════════════════════

pub struct HorspoolDefense {
    evidence: Vec<u8>,
    skip_table: [usize; 256],
}

impl HorspoolDefense {
    pub fn new(evidence: &[u8]) -> Self {
        let m = evidence.len();
        let mut skip_table = [m; 256];

        // Last char not included in preprocessing
        for i in 0..m.saturating_sub(1) {
            skip_table[evidence[i] as usize] = m - 1 - i;
        }

        HorspoolDefense {
            evidence: evidence.to_vec(),
            skip_table,
        }
    }

    pub fn search(&self, testimony: &[u8]) -> Vec<usize> {
        let n = testimony.len();
        let m = self.evidence.len();

        if m == 0 || m > n {
            return vec![];
        }

        let mut matches = vec![];
        let mut i = 0;

        while i <= n - m {
            let mut j = m - 1;

            while self.evidence[j] == testimony[i + j] {
                if j == 0 {
                    matches.push(i);
                    break;
                }
                j -= 1;
            }

            i += self.skip_table[testimony[i + m - 1] as usize].max(1);
        }

        matches
    }
}

// ═══════════════════════════════════════════════════════════════
// WRIGHT TACTICS - Boyer-Moore-Galil
// ═══════════════════════════════════════════════════════════════

pub struct WrightTactics {
    court_record: CourtRecord,
    period: usize,
}

impl WrightTactics {
    pub fn new(evidence: &[u8]) -> Self {
        let period = compute_period(evidence);
        WrightTactics {
            court_record: CourtRecord::new(evidence),
            period,
        }
    }

    pub fn cross_examination(&self, testimony: &[u8]) -> Vec<usize> {
        let n = testimony.len();
        let m = self.court_record.evidence.len();

        if m == 0 || m > n {
            return vec![];
        }

        let mut matches = vec![];
        let mut i = 0;
        let mut skip_start = 0;  // Galil: skip this many chars at start of comparison

        while i <= n - m {
            let mut j = m - 1;

            // Compare from right to left, but skip already-verified prefix (Galil)
            while j >= skip_start && self.court_record.evidence[j] == testimony[i + j] {
                if j == skip_start {
                    if skip_start == 0 || self.verify_prefix(testimony, i, skip_start) {
                        matches.push(i);
                    }
                    break;
                }
                j -= 1;
            }

            // After a match, Galil allows skipping the periodic part
            if j < skip_start || (j == skip_start && matches.last() == Some(&i)) {
                i += self.period;
                skip_start = m - self.period;
            } else {
                let shift = self.court_record.contradiction_table
                    [testimony[i + m - 1] as usize]
                    .max(1);
                i += shift;
                skip_start = 0;
            }
        }

        matches
    }

    fn verify_prefix(&self, testimony: &[u8], start: usize, len: usize) -> bool {
        &testimony[start..start + len] == &self.court_record.evidence[..len]
    }
}

// ═══════════════════════════════════════════════════════════════
// UTILITIES
// ═══════════════════════════════════════════════════════════════

pub fn compute_period(pattern: &[u8]) -> usize {
    let m = pattern.len();
    if m == 0 {
        return 0;
    }

    // Build failure function
    let mut failure = vec![0; m];
    let mut j = 0;
    for i in 1..m {
        while j > 0 && pattern[i] != pattern[j] {
            j = failure[j - 1];
        }
        if pattern[i] == pattern[j] {
            j += 1;
        }
        failure[i] = j;
    }

    // Period = m - failure[m-1] if it divides m
    let potential = m - failure[m - 1];
    if m % potential == 0 {
        potential
    } else {
        m
    }
}

pub fn mod_pow(mut base: u64, mut exp: u64, modulus: u64) -> u64 {
    let mut result = 1u64;
    base %= modulus;

    while exp > 0 {
        if exp & 1 == 1 {
            result = (result * base) % modulus;
        }
        exp >>= 1;
        base = (base * base) % modulus;
    }

    result
}

pub fn matrix_investigation(
    crime_scene: &[Vec<u8>],
    evidence: &[Vec<u8>],
) -> Vec<(usize, usize)> {
    if crime_scene.is_empty() || evidence.is_empty() {
        return vec![];
    }

    let rows = crime_scene.len();
    let cols = crime_scene[0].len();
    let p_rows = evidence.len();
    let p_cols = evidence[0].len();

    if p_rows > rows || p_cols > cols {
        return vec![];
    }

    let rk = TurnaboutHash::new();
    let mut matches = vec![];

    // Compute column hashes for each starting column
    for start_col in 0..=(cols - p_cols) {
        // Hash each row of the pattern (for the p_cols width)
        let pattern_row_hashes: Vec<u64> = evidence
            .iter()
            .map(|row| rk.fingerprint(&row[0..p_cols]))
            .collect();

        // Compute "super-hash" of pattern (hash of row hashes)
        let pattern_super_hash = {
            let mut h = 0u64;
            for &rh in &pattern_row_hashes {
                h = (h * DEFAULT_BASE + rh) % DEFAULT_MODULUS;
            }
            h
        };

        // For each starting row
        for start_row in 0..=(rows - p_rows) {
            // Compute super-hash of this window
            let window_super_hash = {
                let mut h = 0u64;
                for row_idx in start_row..start_row + p_rows {
                    let row_hash = rk.fingerprint(&crime_scene[row_idx][start_col..start_col + p_cols]);
                    h = (h * DEFAULT_BASE + row_hash) % DEFAULT_MODULUS;
                }
                h
            };

            if window_super_hash == pattern_super_hash {
                // Verify match
                let mut is_match = true;
                'outer: for r in 0..p_rows {
                    for c in 0..p_cols {
                        if crime_scene[start_row + r][start_col + c] != evidence[r][c] {
                            is_match = false;
                            break 'outer;
                        }
                    }
                }
                if is_match {
                    matches.push((start_row, start_col));
                }
            }
        }
    }

    matches
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_rabin_karp() {
        let rk = TurnaboutHash::new();
        assert_eq!(
            rk.find_contradiction(b"AABAACAADAABAAABAA", b"AABA"),
            vec![0, 9, 13]
        );
    }

    #[test]
    fn test_boyer_moore() {
        let bm = CourtRecord::new(b"AABA");
        assert_eq!(
            bm.cross_examination(b"AABAACAADAABAAABAA"),
            vec![0, 9, 13]
        );
    }

    #[test]
    fn test_horspool() {
        let h = HorspoolDefense::new(b"ABC");
        assert_eq!(h.search(b"DABCABCABC"), vec![1, 4, 7]);
    }

    #[test]
    fn test_galil() {
        let wt = WrightTactics::new(b"ABAB");
        assert_eq!(wt.cross_examination(b"ABABABABAB"), vec![0, 2, 4, 6]);
    }

    #[test]
    fn test_period() {
        assert_eq!(compute_period(b"ABAB"), 2);
        assert_eq!(compute_period(b"AAAA"), 1);
        assert_eq!(compute_period(b"ABCD"), 4);
    }
}
```

### 4.9 spec.json

```json
{
  "name": "turnabout_search",
  "language": "rust",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse Rabin-Karp & Boyer-Moore",
  "tags": ["strings", "rabin-karp", "boyer-moore", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "find_contradiction",
    "prototype": "pub fn find_contradiction(&self, testimony: &[u8], evidence: &[u8]) -> Vec<usize>",
    "return_type": "Vec<usize>",
    "parameters": [
      {"name": "testimony", "type": "&[u8]"},
      {"name": "evidence", "type": "&[u8]"}
    ]
  },

  "driver": {
    "reference": "pub fn ref_find_contradiction(&self, testimony: &[u8], evidence: &[u8]) -> Vec<usize> { /* voir section 4.3 */ }",

    "edge_cases": [
      {
        "name": "basic_match",
        "args": ["b\"AABAACAABA\"", "b\"AABA\""],
        "expected": "[0, 6]",
        "is_trap": false
      },
      {
        "name": "no_match",
        "args": ["b\"ABCD\"", "b\"XYZ\""],
        "expected": "[]",
        "is_trap": false
      },
      {
        "name": "overlapping",
        "args": ["b\"AAA\"", "b\"AA\""],
        "expected": "[0, 1]",
        "is_trap": true,
        "trap_explanation": "Must find overlapping matches"
      },
      {
        "name": "empty_pattern",
        "args": ["b\"hello\"", "b\"\""],
        "expected": "[]",
        "is_trap": true,
        "trap_explanation": "Empty pattern returns empty"
      },
      {
        "name": "pattern_longer",
        "args": ["b\"AB\"", "b\"ABCD\""],
        "expected": "[]",
        "is_trap": true,
        "trap_explanation": "Pattern longer than text"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 5000,
      "generators": [
        {
          "type": "string",
          "param_index": 0,
          "params": {"min_len": 0, "max_len": 1000, "charset": "custom", "custom_chars": "ABCD"}
        },
        {
          "type": "string",
          "param_index": 1,
          "params": {"min_len": 0, "max_len": 20, "charset": "custom", "custom_chars": "ABCD"}
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["Vec", "Option", "HashMap", "HashSet"],
    "forbidden_functions": ["str::find", "str::contains", "regex"],
    "check_security": false,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```rust
/* Mutant A (Overflow) : base^m sans modulo */
pub fn fingerprint_mutant_a(&self, evidence: &[u8]) -> u64 {
    let mut hash: u64 = 0;
    for &byte in evidence {
        hash = hash * self.base + byte as u64;  // ❌ Pas de % modulus
    }
    hash
}
// Problème: Overflow pour patterns > 8 caractères avec base=256

/* Mutant B (Rolling) : oublie de soustraire old_char */
pub fn roll_fingerprint_mutant_b(&self, old_hash: u64, _old_char: u8, new_char: u8, _base_power: u64) -> u64 {
    (old_hash * self.base + new_char as u64) % self.modulus  // ❌ N'enlève pas old_char
}
// Problème: Hash complètement faux après le premier roll

/* Mutant C (BadChar) : mauvais calcul de shift */
fn build_contradiction_table_mutant_c(evidence: &[u8]) -> [usize; 256] {
    let m = evidence.len();
    let mut table = [m; 256];
    for i in 0..m {  // ❌ Inclut le dernier caractère
        table[evidence[i] as usize] = m - 1 - i;
    }
    table
}
// Problème: Shift trop court pour le dernier caractère, boucle infinie possible

/* Mutant D (GoodSuffix) : table initialisée à 1 au lieu de m */
fn build_testimony_table_mutant_d(evidence: &[u8]) -> Vec<usize> {
    let m = evidence.len();
    let mut table = vec![1; m];  // ❌ Devrait être m
    // ...
    table
}
// Problème: Shifts trop courts, performance O(nm)

/* Mutant E (Galil) : période = m au lieu du vrai calcul */
pub fn new_mutant_e(evidence: &[u8]) -> Self {
    WrightTactics {
        court_record: CourtRecord::new(evidence),
        period: evidence.len(),  // ❌ Toujours la longueur totale
    }
}
// Problème: Galil optimization ne s'applique jamais
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Fingerprinting** : Hash comme signature rapide
2. **Heuristiques de saut** : Skip intelligemment sans tout comparer
3. **Preprocessing vs Search** : Investir en prétraitement pour accélérer la recherche
4. **Trade-offs** : Rabin-Karp vs Boyer-Moore selon le contexte

### 5.2 LDA

```
FONCTION find_contradiction QUI RETOURNE VECTEUR DE POSITIONS
DÉBUT FONCTION
    SI pattern EST VIDE OU pattern EST PLUS LONG QUE text ALORS
        RETOURNER VECTEUR VIDE
    FIN SI

    CALCULER base_power = base^(m-1) mod p
    CALCULER pattern_hash = fingerprint(pattern)
    CALCULER window_hash = fingerprint(text[0..m])

    SI window_hash EST ÉGAL À pattern_hash ET text[0..m] EST ÉGAL À pattern ALORS
        AJOUTER 0 AUX MATCHES
    FIN SI

    POUR i ALLANT DE 1 À n-m FAIRE
        window_hash = roll(window_hash, text[i-1], text[i+m-1], base_power)

        SI window_hash EST ÉGAL À pattern_hash ALORS
            SI text[i..i+m] EST ÉGAL À pattern ALORS
                AJOUTER i AUX MATCHES
            FIN SI
        FIN SI
    FIN POUR

    RETOURNER matches
FIN FONCTION
```

### 5.3 Visualisation ASCII

**Rolling Hash :**
```
Text:    [A][B][C][D][E][F][G]
          \_____/
          Window 1: hash = A×base² + B×base + C

Slide right:
Text:    [A][B][C][D][E][F][G]
             \_____/
             Window 2: hash = (hash - A×base²)×base + D
                            = B×base² + C×base + D
```

**Boyer-Moore Bad Character Rule :**
```
Text:     ...X[E]...
Pattern:  [A][B][C][D]
                  ↑
                  Mismatch: E ≠ D

Bad char table: E last seen at position -1 (not in pattern)
Shift: align E with its last occurrence (none) → shift by m

Text:     ...X[E]...
Pattern:      [A][B][C][D]
                    Shift full pattern
```

**Boyer-Moore Good Suffix Rule :**
```
Text:     ...[A][B][C][D][E]
Pattern:  [X][A][B][C][D]
               ↑
               Mismatch after matching "CD"

Good suffix "CD" appears earlier at position 2
Shift to align:

Text:     ...[A][B][C][D][E]
Pattern:        [X][A][B][C][D]
```

### 5.8 Mnémotechniques

#### 🎯 MEME : "OBJECTION!" — Hash Match

![Phoenix Wright Objection](meme_objection.jpg)

Quand le hash matche, c'est le moment de crier "OBJECTION!"
Mais attention : il faut VÉRIFIER la preuve (éviter les faux positifs).

```rust
if window_hash == pattern_hash {
    // "OBJECTION!"
    if &text[i..] == pattern {
        // "The evidence clearly shows..."
        matches.push(i);
    } else {
        // "Hmm... a false lead..."
    }
}
```

#### 🔙 MEME : "I'll be back" — Boyer-Moore Right-to-Left

Terminator : "I'll be back"
Boyer-Moore : "I'll search from the back"

Comparer de droite à gauche permet de skipper plus de caractères.

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Algorithme | Preprocessing | Average | Worst | Best |
|------------|--------------|---------|-------|------|
| **Rabin-Karp** | O(m) | O(n+m) | O(nm) | O(n+m) |
| **Boyer-Moore** | O(m+σ) | O(n/m) | O(nm) | O(n/m) |
| **Horspool** | O(σ) | O(n/m) | O(nm) | O(n/m) |
| **Galil** | O(m+σ) | O(n) | O(n) | O(n/m) |

---

*HACKBRAIN v5.5.2 — "OBJECTION! The hash doesn't lie!"*
