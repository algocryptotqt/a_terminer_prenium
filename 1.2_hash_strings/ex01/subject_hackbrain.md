<thinking>
## Analyse du Concept
- Concept : Pattern Matching avec KMP et Z-Algorithm
- Phase demandée : 1 (Intermédiaire)
- Adapté ? OUI - Algorithmes fondamentaux de string matching

## Combo Base + Bonus
- Exercice de base : KMP + Z-Algorithm + applications (rotation, période)
- Bonus : Aho-Corasick (multi-pattern), Suffix Automaton
- Palier bonus : 🔥 Avancé (algorithmes avancés mais connus)
- Progression logique ? OUI - Base = single pattern, Bonus = multi-pattern

## Prérequis & Difficulté
- Prérequis réels : Arrays, strings, notion de préfixe/suffixe
- Difficulté estimée : 5/10 (base), 8/10 (bonus)
- Cohérent avec phase ? OUI - Phase 1 = 3-5/10

## Aspect Fun/Culture
- Contexte choisi : ID:Invaded (anime 2020 sur des détectives qui plongent dans les "id wells" de tueurs en série)
- MEME mnémotechnique : "Find Kaeru" - Dans l'anime, ils cherchent toujours "Kaeru" dans chaque id well, comme on cherche un pattern
- Pourquoi c'est fun :
  * Mizuhanome System = KMP (le système qui permet de matcher les patterns cognitifs)
  * Cognition Particles = Prefix function (les particules qui construisent les patterns)
  * Id Well = Z-Array (la construction du well à partir des patterns)
  * Sakaido = L'algorithme qui "plonge" et recherche
  * John Walker = Le serial killer dont on cherche les "borders" (traces)
  * Les patterns de tueurs en série = La période répétitive dans leurs crimes

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : failure[0] = 1 au lieu de 0 → décalage de tout
2. Mutant B (Logic) : j = failure[i] au lieu de j = failure[i-1] → boucle infinie
3. Mutant C (Off-by-one) : while r < n-1 au lieu de r < n → miss last char
4. Mutant D (Z-box) : oubli de r -= 1 → z-values incorrects
5. Mutant E (Search) : retourne i au lieu de i - m → positions décalées

## Verdict
VALIDE - ID:Invaded est une analogie parfaite pour le pattern matching
Score créativité : 98/100 (niche mais excellente correspondance conceptuelle)
</thinking>

---

# Exercice 1.2.1-synth : mizuhanome_search

**Module :**
1.2 — Hash Tables & Strings

**Concept :**
synth — Pattern Matching (KMP & Z-Algorithm)

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (KMP + Z + Applications)

**Langage :**
Rust Edition 2024 / C (c17)

**Prérequis :**
- Manipulation de strings et arrays
- Notion de préfixe et suffixe
- Complexité algorithmique basique

**Domaines :**
Struct, Encodage

**Durée estimée :**
90 min

**XP Base :**
120

**Complexité :**
T4 O(n+m) × S3 O(m)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `mizuhanome.c`, `mizuhanome.h`

**Fonctions autorisées :**
- Rust : Types standard (`Vec`, `String`, `Option`)
- C : `malloc`, `free`, `strlen`, `memcpy`

**Fonctions interdites :**
- Rust : `str::find`, `str::contains`, regex
- C : `strstr`, `memmem`, bibliothèques externes

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🎮 ID:INVADED : Le Système Mizuhanome - Plongée dans les Id Wells**

*"Dans un futur proche, les détectives de la Kura utilisent le système Mizuhanome pour plonger dans les 'id wells' - des représentations mentales construites à partir des 'cognition particles' laissées par les tueurs en série. Chaque well contient un pattern unique... et quelque part, la victime 'Kaeru' attend d'être trouvée."*

Dans l'univers de **ID:Invaded**, le détective Sakaido (alias Akihito Narihisago) plonge dans les id wells pour résoudre des crimes. Le système Mizuhanome analyse les **cognition particles** (particules cognitives) pour construire ces wells et y chercher des patterns.

Tu es recruté comme ingénieur du système Mizuhanome. Ton travail : implémenter les algorithmes de pattern matching qui permettent de :

**🔬 Cognition Particles (Prefix Function) :**
Comme les particules cognitives qui révèlent les répétitions dans la psyché d'un tueur, la **failure function** de KMP révèle les répétitions internes d'un pattern.

**🌊 Mizuhanome Search (KMP) :**
Plonger dans le texte (l'id well) à la recherche du pattern (le tueur). Quand on trouve un mismatch, on ne recommence pas du début - on utilise les cognition particles pour savoir où "réatterrir".

**💠 Id Well Construction (Z-Array) :**
Construire le Z-array, c'est comme construire un id well : pour chaque position, on calcule "combien de cette position ressemble au début du well".

**🎯 Find Kaeru (Applications) :**
- Trouver la **période** d'un pattern (le cycle du tueur)
- Vérifier si deux cas sont des **rotations** l'un de l'autre
- Identifier les **borders** (les traces laissées entre les crimes)

**Ta mission :**

1. **`compute_cognition_particles`** : Calculer la failure function de KMP
2. **`mizuhanome_search`** : Trouver la première occurrence
3. **`mizuhanome_search_all`** : Trouver toutes les occurrences
4. **`construct_id_well`** : Calculer le Z-array
5. **`well_search`** : Recherche avec Z-algorithm
6. **`serial_killer_period`** : Trouver la plus courte période
7. **`case_rotation`** : Vérifier si deux cas sont des rotations
8. **`john_walker_borders`** : Trouver tous les borders d'un string

**Entrée :**
- `text: &[u8]` : Le texte/id well dans lequel chercher
- `pattern: &[u8]` : Le pattern/signature du tueur à trouver

**Sortie :**
- `Vec<usize>` pour les positions de match
- `Option<usize>` pour la première occurrence
- `usize` pour la période
- `bool` pour la rotation

**Contraintes :**
┌─────────────────────────────────────────────────────────────────┐
│  Temps : O(n + m) pour KMP et Z                                 │
│  Espace : O(m) pour KMP, O(n + m) pour Z                        │
│  Pattern vide : Retourner vecteur vide                          │
│  Text plus court que pattern : Retourner vecteur vide           │
└─────────────────────────────────────────────────────────────────┘

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `compute_cognition_particles(b"ABAB")` | `[0, 0, 1, 2]` | A=0, B=0, AB=1, ABA→AB=2 |
| `mizuhanome_search(b"AABAABAAA", b"AABA")` | `Some(0)` | Match à position 0 |
| `mizuhanome_search_all(b"AABAABA", b"ABA")` | `[1, 4]` | Deux occurrences |
| `serial_killer_period(b"abcabc")` | `3` | "abc" se répète |
| `case_rotation("waterbottle", "erbottlewat")` | `true` | Rotation valide |

#### 1.2.2 Version Académique

**Objectif :**

Implémenter les algorithmes de pattern matching KMP et Z-algorithm, ainsi que leurs applications.

**Algorithme KMP (Knuth-Morris-Pratt) :**

L'algorithme KMP utilise une **failure function** (ou prefix function) pour éviter de reconsidérer des caractères déjà matchés. La failure function `f[i]` donne la longueur du plus long préfixe propre de `pattern[0..i+1]` qui est aussi un suffixe.

Construction de la failure function :
1. `f[0] = 0` (pas de préfixe propre pour un seul caractère)
2. Pour `i` de 1 à m-1 :
   - `j = f[i-1]`
   - Tant que `j > 0` et `pattern[i] != pattern[j]` : `j = f[j-1]`
   - Si `pattern[i] == pattern[j]` : `j++`
   - `f[i] = j`

Recherche KMP :
1. Comparer pattern et texte caractère par caractère
2. En cas de mismatch à position j dans pattern, sauter à `f[j-1]` au lieu de recommencer

**Algorithme Z :**

Le Z-array `z[i]` donne la longueur du plus long substring commençant à position `i` qui matche un préfixe du string.

Utilise le concept de "Z-box" `[l, r]` pour éviter les recomparaisons.

**Applications :**
- **Période** : Si `n % (n - f[n-1]) == 0`, la période est `n - f[n-1]`
- **Rotation** : `s1` est rotation de `s2` si `s2` est substring de `s1 + s1`
- **Borders** : Tous les préfixes qui sont aussi des suffixes

### 1.3 Prototype

**Rust :**
```rust
/// Compute KMP failure function (cognition particles)
/// failure[i] = length of longest proper prefix of pattern[0..i+1]
///              that is also a suffix
pub fn compute_cognition_particles(pattern: &[u8]) -> Vec<usize>;

/// KMP search - find first occurrence (mizuhanome dive)
pub fn mizuhanome_search(text: &[u8], pattern: &[u8]) -> Option<usize>;

/// KMP search - find all occurrences
pub fn mizuhanome_search_all(text: &[u8], pattern: &[u8]) -> Vec<usize>;

/// KMP with custom comparator (for case-insensitive, etc.)
pub fn mizuhanome_search_by<F>(text: &[u8], pattern: &[u8], eq: F) -> Vec<usize>
where
    F: Fn(u8, u8) -> bool;

/// Compute Z-array (id well construction)
/// z[i] = length of longest substring starting at i that matches prefix
pub fn construct_id_well(s: &[u8]) -> Vec<usize>;

/// Z-algorithm search
pub fn well_search(text: &[u8], pattern: &[u8]) -> Vec<usize>;

/// Find the shortest period of a string (serial killer cycle)
/// Period p: s[i] = s[i % p] for all i
pub fn serial_killer_period(s: &[u8]) -> usize;

/// Check if s1 is a rotation of s2
pub fn case_rotation(s1: &str, s2: &str) -> bool;

/// Find all borders (prefixes that are also suffixes)
pub fn john_walker_borders(s: &[u8]) -> Vec<usize>;

/// Find the lexicographically smallest rotation
pub fn min_rotation(s: &str) -> String;

/// Count distinct substrings using Z-array
pub fn count_distinct_substrings(s: &str) -> usize;

/// Pattern matching with wildcards (? matches any single char)
pub fn wildcard_match(text: &[u8], pattern: &[u8]) -> Vec<usize>;
```

**C :**
```c
#ifndef MIZUHANOME_H
#define MIZUHANOME_H

#include <stddef.h>
#include <stdbool.h>

// ═══════════════════════════════════════════════════════════════
// KMP ALGORITHM (Cognition Particles)
// ═══════════════════════════════════════════════════════════════

/// Compute KMP failure function
/// Returns malloc'd array of size pattern_len, caller must free
size_t  *compute_cognition_particles(const char *pattern, size_t pattern_len);

/// KMP search - find first occurrence
/// Returns index or (size_t)-1 if not found
size_t  mizuhanome_search(const char *text, size_t text_len,
                          const char *pattern, size_t pattern_len);

/// KMP search - find all occurrences
/// Returns malloc'd array of indices, count stored in *num_matches
/// Caller must free returned array
size_t  *mizuhanome_search_all(const char *text, size_t text_len,
                               const char *pattern, size_t pattern_len,
                               size_t *num_matches);

// ═══════════════════════════════════════════════════════════════
// Z-ALGORITHM (Id Well Construction)
// ═══════════════════════════════════════════════════════════════

/// Compute Z-array
/// Returns malloc'd array of size len, caller must free
size_t  *construct_id_well(const char *s, size_t len);

/// Z-algorithm search - find all occurrences
size_t  *well_search(const char *text, size_t text_len,
                     const char *pattern, size_t pattern_len,
                     size_t *num_matches);

// ═══════════════════════════════════════════════════════════════
// APPLICATIONS
// ═══════════════════════════════════════════════════════════════

/// Find shortest period of string
size_t  serial_killer_period(const char *s, size_t len);

/// Check if s1 is a rotation of s2
bool    case_rotation(const char *s1, const char *s2);

/// Find all borders (prefixes that are also suffixes)
/// Returns malloc'd array, count in *num_borders
size_t  *john_walker_borders(const char *s, size_t len, size_t *num_borders);

/// Find lexicographically smallest rotation
/// Returns malloc'd string, caller must free
char    *min_rotation(const char *s, size_t len);

/// Count distinct substrings
size_t  count_distinct_substrings(const char *s, size_t len);

/// Pattern matching with '?' wildcards
size_t  *wildcard_match(const char *text, size_t text_len,
                        const char *pattern, size_t pattern_len,
                        size_t *num_matches);

#endif // MIZUHANOME_H
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Anecdote Historique

L'algorithme KMP a été inventé par **Knuth**, **Morris** et **Pratt** en 1977, mais découvert indépendamment par James H. Morris en 1970. Donald Knuth a prouvé qu'il était optimal en termes de comparaisons.

Le **Z-algorithm** est moins connu mais tout aussi puissant. Il a été popularisé dans les compétitions de programmation car il est souvent plus facile à coder que KMP tout en ayant la même complexité.

### 2.2 Chiffre Clé

- **Complexité** : O(n + m) pour les deux algorithmes - linéaire !
- **Bio-informatique** : KMP est utilisé pour chercher des séquences ADN dans des génomes de milliards de bases
- **Éditeurs de texte** : Ctrl+F utilise des variantes de ces algorithmes

### 2.3 Culture Geek

ID:Invaded (2020) est un anime qui mélange science-fiction et mystère. Le concept de "plonger" dans l'esprit des criminels ressemble étrangement à ce que font les algorithmes de pattern matching : plonger dans un texte pour y trouver des patterns spécifiques.

### 2.5 Dans la Vraie Vie

| Métier | Utilisation |
|--------|-------------|
| **Bioinformaticien** | Recherche de séquences génétiques (KMP/Z) |
| **Security Analyst** | Détection de signatures de malware |
| **Search Engineer** | Indexation et recherche full-text |
| **Compiler Developer** | Lexical analysis, tokenization |
| **Network Engineer** | Deep packet inspection |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
mizuhanome.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 11 tests
test test_cognition_particles ... ok
test test_mizuhanome_search ... ok
test test_mizuhanome_all ... ok
test test_id_well ... ok
test test_well_search ... ok
test test_period ... ok
test test_rotation ... ok
test test_borders ... ok
test test_min_rotation ... ok
test test_wildcard ... ok
test test_distinct_substrings ... ok

test result: ok. 11 passed; 0 failed

$ ./target/release/mizuhanome_demo
=== MIZUHANOME SYSTEM ACTIVATED ===
Text: "AABAACAADAABAAABAA"
Pattern: "AABA"
Cognition Particles: [0, 1, 0, 1]
Matches found at: [0, 9, 13]
Shortest period of "abcabc": 3
"waterbottle" is rotation of "erbottlewat": true
System Status: KAERU FOUND
```

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n + m × k) pour Aho-Corasick (k patterns)

**Space Complexity attendue :**
O(sum of pattern lengths)

**Domaines Bonus :**
`Struct, DP`

#### 3.1.1 Consigne Bonus

**🎮 MIZUHANOME 2.0 : Multi-Pattern Hunt**

*"Le nombre de tueurs en série a explosé. La Kura a besoin d'un système capable de chercher PLUSIEURS patterns simultanément dans un id well. Bienvenue dans le monde d'Aho-Corasick."*

**Ta mission avancée :**

1. **`AhoCorasickMachine`** : Automate d'Aho-Corasick
   - Construction du trie
   - Failure links (comme KMP mais pour un trie)
   - Output links pour patterns overlapping

2. **`multi_pattern_search`** : Recherche multi-pattern en O(n + m + z)
   - z = nombre total de matches

3. **`SuffixAutomaton`** : Structure pour toutes les requêtes substring
   - Construction en O(n)
   - Requêtes substring en O(m)

4. **`longest_common_substring`** : LCS de deux strings
   - Utiliser le suffix automaton

**Contraintes Bonus :**
┌─────────────────────────────────────────────────────────────────┐
│  Aho-Corasick : Construction O(sum of patterns)                 │
│  Search : O(n + z) où z = nombre de matches                     │
│  Suffix Automaton : O(n) construction, O(m) query               │
│  Memory : Trie nodes = O(alphabet × patterns)                   │
└─────────────────────────────────────────────────────────────────┘

#### 3.1.2 Prototype Bonus

```rust
/// Aho-Corasick automaton for multi-pattern matching
pub struct AhoCorasickMachine {
    goto: Vec<[Option<usize>; 256]>,  // State transitions
    fail: Vec<usize>,                   // Failure links
    output: Vec<Vec<usize>>,            // Pattern indices at each state
}

impl AhoCorasickMachine {
    /// Build automaton from patterns
    pub fn new(patterns: &[&[u8]]) -> Self;

    /// Search for all patterns in text
    /// Returns (position, pattern_index) pairs
    pub fn search(&self, text: &[u8]) -> Vec<(usize, usize)>;

    /// Stream search - process text incrementally
    pub fn search_stream<'a>(&'a self, text: &'a [u8])
        -> impl Iterator<Item = (usize, usize)> + 'a;
}

/// Suffix Automaton (DAWG - Directed Acyclic Word Graph)
pub struct SuffixAutomaton {
    states: Vec<SuffixState>,
    last: usize,
}

#[derive(Clone)]
struct SuffixState {
    len: usize,                          // Longest string length in equivalence class
    link: Option<usize>,                 // Suffix link
    next: std::collections::HashMap<u8, usize>,  // Transitions
}

impl SuffixAutomaton {
    /// Build automaton from string
    pub fn new(s: &[u8]) -> Self;

    /// Check if pattern is substring
    pub fn contains(&self, pattern: &[u8]) -> bool;

    /// Count occurrences of pattern
    pub fn count(&self, pattern: &[u8]) -> usize;

    /// Find first occurrence of pattern
    pub fn find(&self, pattern: &[u8]) -> Option<usize>;
}

/// Find longest common substring using suffix automaton
pub fn longest_common_substring(s1: &[u8], s2: &[u8]) -> Vec<u8>;
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Patterns | 1 pattern | k patterns simultanés |
| Structure | Array linéaire | Trie + automate |
| Complexité recherche | O(n + m) | O(n + z) pour k patterns |
| Applications | Substring simple | Occurrences multiples, LCS |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Entrée | Sortie Attendue | Points |
|------|--------|-----------------|--------|
| `particles_aaaa` | `"AAAA"` | `[0, 1, 2, 3]` | 3 |
| `particles_abab` | `"ABAB"` | `[0, 0, 1, 2]` | 3 |
| `particles_complex` | `"AABAACAABAA"` | `[0,1,0,1,2,0,1,2,3,4,5]` | 3 |
| `search_single` | text="AABA...", pat="AABA" | `Some(0)` | 3 |
| `search_all` | text, pattern | `[0, 9, 13]` | 4 |
| `search_no_match` | `"ABCD"`, `"XYZ"` | `None` | 2 |
| `z_array_basic` | `"aabxaab"` | `[7,1,0,0,3,1,0]` | 4 |
| `z_array_all_same` | `"aaaa"` | `[4,3,2,1]` | 3 |
| `z_search` | text, pattern | Matches KMP | 4 |
| `period_abc` | `"abcabc"` | `3` | 3 |
| `period_single` | `"aaaa"` | `1` | 2 |
| `rotation_yes` | `"waterbottle"`, `"erbottlewat"` | `true` | 3 |
| `rotation_no` | `"abc"`, `"acb"` | `false` | 2 |
| `borders` | `"abacaba"` | `[1, 3, 7]` | 4 |
| `empty_pattern` | text, `""` | `[]` | 2 |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <string.h>
#include <assert.h>
#include <stdlib.h>
#include "mizuhanome.h"

void test_cognition_particles(void) {
    printf("Testing compute_cognition_particles...\n");

    // Test "AAAA" → [0, 1, 2, 3]
    size_t *f = compute_cognition_particles("AAAA", 4);
    assert(f[0] == 0 && f[1] == 1 && f[2] == 2 && f[3] == 3);
    free(f);

    // Test "ABAB" → [0, 0, 1, 2]
    f = compute_cognition_particles("ABAB", 4);
    assert(f[0] == 0 && f[1] == 0 && f[2] == 1 && f[3] == 2);
    free(f);

    // Test "AABAACAABAA" → [0,1,0,1,2,0,1,2,3,4,5]
    f = compute_cognition_particles("AABAACAABAA", 11);
    size_t expected[] = {0, 1, 0, 1, 2, 0, 1, 2, 3, 4, 5};
    for (int i = 0; i < 11; i++) {
        assert(f[i] == expected[i]);
    }
    free(f);

    printf("  PASS\n");
}

void test_mizuhanome_search(void) {
    printf("Testing mizuhanome_search...\n");

    const char *text = "AABAACAADAABAAABAA";
    const char *pattern = "AABA";

    // First occurrence
    size_t pos = mizuhanome_search(text, strlen(text), pattern, strlen(pattern));
    assert(pos == 0);

    // All occurrences
    size_t count;
    size_t *matches = mizuhanome_search_all(text, strlen(text),
                                            pattern, strlen(pattern), &count);
    assert(count == 3);
    assert(matches[0] == 0);
    assert(matches[1] == 9);
    assert(matches[2] == 13);
    free(matches);

    // No match
    pos = mizuhanome_search("ABCD", 4, "XYZ", 3);
    assert(pos == (size_t)-1);

    printf("  PASS\n");
}

void test_z_array(void) {
    printf("Testing construct_id_well (Z-array)...\n");

    size_t *z = construct_id_well("aabxaab", 7);
    size_t expected[] = {7, 1, 0, 0, 3, 1, 0};
    for (int i = 0; i < 7; i++) {
        assert(z[i] == expected[i]);
    }
    free(z);

    z = construct_id_well("aaaa", 4);
    assert(z[0] == 4 && z[1] == 3 && z[2] == 2 && z[3] == 1);
    free(z);

    printf("  PASS\n");
}

void test_period(void) {
    printf("Testing serial_killer_period...\n");

    assert(serial_killer_period("abcabc", 6) == 3);
    assert(serial_killer_period("aaaa", 4) == 1);
    assert(serial_killer_period("abcd", 4) == 4);
    assert(serial_killer_period("abab", 4) == 2);

    printf("  PASS\n");
}

void test_rotation(void) {
    printf("Testing case_rotation...\n");

    assert(case_rotation("waterbottle", "erbottlewat") == true);
    assert(case_rotation("abcd", "cdab") == true);
    assert(case_rotation("abc", "acb") == false);
    assert(case_rotation("a", "a") == true);

    printf("  PASS\n");
}

void test_borders(void) {
    printf("Testing john_walker_borders...\n");

    size_t count;
    size_t *borders = john_walker_borders("abacaba", 7, &count);
    assert(count == 3);
    assert(borders[0] == 1);   // "a"
    assert(borders[1] == 3);   // "aba"
    assert(borders[2] == 7);   // "abacaba"
    free(borders);

    printf("  PASS\n");
}

int main(void) {
    printf("=== MIZUHANOME SYSTEM TEST SUITE ===\n\n");

    test_cognition_particles();
    test_mizuhanome_search();
    test_z_array();
    test_period();
    test_rotation();
    test_borders();

    printf("\n=== ALL TESTS PASSED - KAERU FOUND ===\n");
    return 0;
}
```

### 4.3 Solution de référence

```rust
// ═══════════════════════════════════════════════════════════════
// COGNITION PARTICLES - KMP Failure Function
// ═══════════════════════════════════════════════════════════════

pub fn compute_cognition_particles(pattern: &[u8]) -> Vec<usize> {
    let m = pattern.len();
    if m == 0 {
        return vec![];
    }

    let mut failure = vec![0; m];

    // failure[0] is always 0
    let mut j = 0;

    for i in 1..m {
        // Follow failure links until we find a match or reach 0
        while j > 0 && pattern[i] != pattern[j] {
            j = failure[j - 1];
        }

        if pattern[i] == pattern[j] {
            j += 1;
        }

        failure[i] = j;
    }

    failure
}

// ═══════════════════════════════════════════════════════════════
// MIZUHANOME SEARCH - KMP Pattern Matching
// ═══════════════════════════════════════════════════════════════

pub fn mizuhanome_search(text: &[u8], pattern: &[u8]) -> Option<usize> {
    mizuhanome_search_all(text, pattern).into_iter().next()
}

pub fn mizuhanome_search_all(text: &[u8], pattern: &[u8]) -> Vec<usize> {
    let n = text.len();
    let m = pattern.len();

    if m == 0 || m > n {
        return vec![];
    }

    let failure = compute_cognition_particles(pattern);
    let mut matches = vec![];
    let mut j = 0;  // Index in pattern

    for i in 0..n {
        // Follow failure links on mismatch
        while j > 0 && text[i] != pattern[j] {
            j = failure[j - 1];
        }

        if text[i] == pattern[j] {
            j += 1;
        }

        // Full match found
        if j == m {
            matches.push(i + 1 - m);
            j = failure[j - 1];  // Continue searching for more matches
        }
    }

    matches
}

pub fn mizuhanome_search_by<F>(text: &[u8], pattern: &[u8], eq: F) -> Vec<usize>
where
    F: Fn(u8, u8) -> bool,
{
    let n = text.len();
    let m = pattern.len();

    if m == 0 || m > n {
        return vec![];
    }

    // Build failure function with custom equality
    let mut failure = vec![0; m];
    let mut j = 0;
    for i in 1..m {
        while j > 0 && !eq(pattern[i], pattern[j]) {
            j = failure[j - 1];
        }
        if eq(pattern[i], pattern[j]) {
            j += 1;
        }
        failure[i] = j;
    }

    // Search with custom equality
    let mut matches = vec![];
    let mut j = 0;

    for i in 0..n {
        while j > 0 && !eq(text[i], pattern[j]) {
            j = failure[j - 1];
        }
        if eq(text[i], pattern[j]) {
            j += 1;
        }
        if j == m {
            matches.push(i + 1 - m);
            j = failure[j - 1];
        }
    }

    matches
}

// ═══════════════════════════════════════════════════════════════
// ID WELL CONSTRUCTION - Z-Array
// ═══════════════════════════════════════════════════════════════

pub fn construct_id_well(s: &[u8]) -> Vec<usize> {
    let n = s.len();
    if n == 0 {
        return vec![];
    }

    let mut z = vec![0; n];
    z[0] = n;  // By convention, z[0] = length of string

    let mut l = 0;
    let mut r = 0;

    for i in 1..n {
        if i > r {
            // Case 1: i is outside the Z-box, compute from scratch
            l = i;
            r = i;
            while r < n && s[r - l] == s[r] {
                r += 1;
            }
            z[i] = r - l;
            r -= 1;
        } else {
            // Case 2: i is inside the Z-box
            let k = i - l;

            if z[k] < r - i + 1 {
                // Case 2a: z[k] doesn't reach the end of Z-box
                z[i] = z[k];
            } else {
                // Case 2b: z[k] reaches or exceeds the end of Z-box
                l = i;
                while r < n && s[r - l] == s[r] {
                    r += 1;
                }
                z[i] = r - l;
                r -= 1;
            }
        }
    }

    z
}

// ═══════════════════════════════════════════════════════════════
// WELL SEARCH - Z-Algorithm Pattern Matching
// ═══════════════════════════════════════════════════════════════

pub fn well_search(text: &[u8], pattern: &[u8]) -> Vec<usize> {
    let m = pattern.len();
    let n = text.len();

    if m == 0 || m > n {
        return vec![];
    }

    // Concatenate: pattern + '$' + text
    let mut concat = Vec::with_capacity(m + 1 + n);
    concat.extend_from_slice(pattern);
    concat.push(b'$');  // Separator not in alphabet
    concat.extend_from_slice(text);

    let z = construct_id_well(&concat);

    let mut matches = vec![];
    for i in (m + 1)..concat.len() {
        if z[i] == m {
            matches.push(i - m - 1);
        }
    }

    matches
}

// ═══════════════════════════════════════════════════════════════
// APPLICATIONS
// ═══════════════════════════════════════════════════════════════

pub fn serial_killer_period(s: &[u8]) -> usize {
    let n = s.len();
    if n == 0 {
        return 0;
    }

    let failure = compute_cognition_particles(s);
    let longest_border = failure[n - 1];

    // If n is divisible by (n - longest_border), that's the period
    let potential_period = n - longest_border;
    if n % potential_period == 0 {
        potential_period
    } else {
        n  // No shorter period, the string itself is the period
    }
}

pub fn case_rotation(s1: &str, s2: &str) -> bool {
    if s1.len() != s2.len() {
        return false;
    }
    if s1.is_empty() {
        return true;
    }

    // s1 is rotation of s2 iff s2 is substring of s1 + s1
    let doubled = format!("{}{}", s1, s1);
    mizuhanome_search(doubled.as_bytes(), s2.as_bytes()).is_some()
}

pub fn john_walker_borders(s: &[u8]) -> Vec<usize> {
    if s.is_empty() {
        return vec![];
    }

    let failure = compute_cognition_particles(s);
    let mut borders = vec![];

    // Follow the chain of failure links from the last position
    let mut curr = s.len();
    borders.push(curr);  // The string itself is always a border

    curr = failure[curr - 1];
    while curr > 0 {
        borders.push(curr);
        curr = failure[curr - 1];
    }

    borders.reverse();
    borders
}

pub fn min_rotation(s: &str) -> String {
    if s.is_empty() {
        return String::new();
    }

    // Booth's algorithm for minimum rotation
    let s = s.as_bytes();
    let n = s.len();
    let doubled: Vec<u8> = s.iter().chain(s.iter()).cloned().collect();

    let mut failure = vec![usize::MAX; 2 * n];
    let mut k = 0;  // Start of current minimum rotation

    for j in 1..(2 * n) {
        let sj = doubled[j];
        let mut i = failure[j - k - 1];

        while i != usize::MAX && sj != doubled[k + i + 1] {
            if sj < doubled[k + i + 1] {
                k = j - i - 1;
            }
            i = failure[i];
        }

        if sj != doubled[k + i.wrapping_add(1)] {
            if sj < doubled[k] {
                k = j;
            }
            failure[j - k] = usize::MAX;
        } else {
            failure[j - k] = i.wrapping_add(1);
        }
    }

    String::from_utf8(doubled[k..k + n].to_vec()).unwrap()
}

pub fn count_distinct_substrings(s: &str) -> usize {
    let s = s.as_bytes();
    let n = s.len();

    // Total substrings without repetition = n*(n+1)/2
    // But we need to subtract repetitions
    // Using Z-array for each suffix

    let mut count = 0;

    for i in 0..n {
        let z = construct_id_well(&s[i..]);
        // New substrings ending at each position
        let max_z = z.iter().skip(1).max().copied().unwrap_or(0);
        count += (n - i) - max_z;
    }

    count
}

pub fn wildcard_match(text: &[u8], pattern: &[u8]) -> Vec<usize> {
    let n = text.len();
    let m = pattern.len();

    if m == 0 || m > n {
        return vec![];
    }

    let mut matches = vec![];

    'outer: for i in 0..=(n - m) {
        for j in 0..m {
            if pattern[j] != b'?' && pattern[j] != text[i + j] {
                continue 'outer;
            }
        }
        matches.push(i);
    }

    matches
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_failure_function() {
        assert_eq!(compute_cognition_particles(b"AAAA"), vec![0, 1, 2, 3]);
        assert_eq!(compute_cognition_particles(b"ABAB"), vec![0, 0, 1, 2]);
        assert_eq!(
            compute_cognition_particles(b"AABAACAABAA"),
            vec![0, 1, 0, 1, 2, 0, 1, 2, 3, 4, 5]
        );
    }

    #[test]
    fn test_kmp_search() {
        let text = b"AABAACAADAABAAABAA";
        let pattern = b"AABA";

        assert_eq!(mizuhanome_search(text, pattern), Some(0));
        assert_eq!(mizuhanome_search_all(text, pattern), vec![0, 9, 13]);
    }

    #[test]
    fn test_z_array() {
        assert_eq!(construct_id_well(b"aabxaab"), vec![7, 1, 0, 0, 3, 1, 0]);
        assert_eq!(construct_id_well(b"aaaa"), vec![4, 3, 2, 1]);
    }

    #[test]
    fn test_period() {
        assert_eq!(serial_killer_period(b"abcabc"), 3);
        assert_eq!(serial_killer_period(b"aaaa"), 1);
        assert_eq!(serial_killer_period(b"abcd"), 4);
    }

    #[test]
    fn test_rotation() {
        assert!(case_rotation("waterbottle", "erbottlewat"));
        assert!(!case_rotation("abc", "acb"));
    }

    #[test]
    fn test_borders() {
        assert_eq!(john_walker_borders(b"abacaba"), vec![1, 3, 7]);
    }
}
```

### 4.5 Solutions refusées (avec explications)

```rust
// ❌ REFUSÉ : failure[0] = 1 au lieu de 0
pub fn compute_cognition_particles_wrong(pattern: &[u8]) -> Vec<usize> {
    let mut failure = vec![0; pattern.len()];
    failure[0] = 1;  // ❌ FAUX ! Doit être 0
    // ...
}
// Problème: Tous les indices sont décalés, matches incorrects

// ❌ REFUSÉ : j = failure[i] au lieu de failure[i-1]
fn build_failure_wrong(pattern: &[u8]) -> Vec<usize> {
    let mut failure = vec![0; pattern.len()];
    let mut j = 0;
    for i in 1..pattern.len() {
        while j > 0 && pattern[i] != pattern[j] {
            j = failure[i];  // ❌ FAUX ! Doit être failure[j-1]
        }
        // ...
    }
    failure
}
// Problème: Boucle infinie ou indices invalides

// ❌ REFUSÉ : Z-array sans mise à jour de r
pub fn construct_id_well_wrong(s: &[u8]) -> Vec<usize> {
    let n = s.len();
    let mut z = vec![0; n];
    let mut l = 0;
    let mut r = 0;

    for i in 1..n {
        if i > r {
            l = i;
            r = i;
            while r < n && s[r - l] == s[r] {
                r += 1;
            }
            z[i] = r - l;
            // ❌ OUBLI: r -= 1;  <-- Manquant !
        }
        // ...
    }
    z
}
// Problème: Z-values incorrects, off-by-one errors
```

### 4.9 spec.json (ENGINE v22.1 — FORMAT STRICT)

```json
{
  "name": "mizuhanome_search",
  "language": "rust",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse Pattern Matching",
  "tags": ["strings", "kmp", "z-algorithm", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "mizuhanome_search_all",
    "prototype": "pub fn mizuhanome_search_all(text: &[u8], pattern: &[u8]) -> Vec<usize>",
    "return_type": "Vec<usize>",
    "parameters": [
      {"name": "text", "type": "&[u8]"},
      {"name": "pattern", "type": "&[u8]"}
    ]
  },

  "driver": {
    "reference": "pub fn ref_mizuhanome_search_all(text: &[u8], pattern: &[u8]) -> Vec<usize> { let n = text.len(); let m = pattern.len(); if m == 0 || m > n { return vec![]; } let failure = compute_cognition_particles(pattern); let mut matches = vec![]; let mut j = 0; for i in 0..n { while j > 0 && text[i] != pattern[j] { j = failure[j - 1]; } if text[i] == pattern[j] { j += 1; } if j == m { matches.push(i + 1 - m); j = failure[j - 1]; } } matches }",

    "edge_cases": [
      {
        "name": "empty_pattern",
        "args": ["b\"hello\"", "b\"\""],
        "expected": "[]",
        "is_trap": true,
        "trap_explanation": "Empty pattern should return empty vector"
      },
      {
        "name": "pattern_longer_than_text",
        "args": ["b\"ab\"", "b\"abcd\""],
        "expected": "[]",
        "is_trap": true,
        "trap_explanation": "Pattern longer than text cannot match"
      },
      {
        "name": "no_match",
        "args": ["b\"ABCD\"", "b\"XYZ\""],
        "expected": "[]",
        "is_trap": false
      },
      {
        "name": "single_match",
        "args": ["b\"AABAA\"", "b\"ABA\""],
        "expected": "[1]",
        "is_trap": false
      },
      {
        "name": "multiple_matches",
        "args": ["b\"AABAACAADAABAAABAA\"", "b\"AABA\""],
        "expected": "[0, 9, 13]",
        "is_trap": false
      },
      {
        "name": "overlapping_matches",
        "args": ["b\"AAAA\"", "b\"AA\""],
        "expected": "[0, 1, 2]",
        "is_trap": true,
        "trap_explanation": "Must find overlapping matches"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 5000,
      "generators": [
        {
          "type": "string",
          "param_index": 0,
          "params": {
            "min_len": 1,
            "max_len": 1000,
            "charset": "custom",
            "custom_chars": "ABCD"
          }
        },
        {
          "type": "string",
          "param_index": 1,
          "params": {
            "min_len": 1,
            "max_len": 50,
            "charset": "custom",
            "custom_chars": "ABCD"
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["Vec", "Option", "slice methods"],
    "forbidden_functions": ["str::find", "str::contains", "regex"],
    "check_security": false,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```rust
/* Mutant A (Boundary) : failure[0] initialisé à 1 */
pub fn compute_cognition_particles_mutant_a(pattern: &[u8]) -> Vec<usize> {
    let mut failure = vec![0; pattern.len()];
    if !pattern.is_empty() {
        failure[0] = 1;  // ❌ Doit être 0
    }
    // ...reste du code identique
    failure
}
// Pourquoi c'est faux : failure[0] doit toujours être 0 (pas de préfixe propre non vide)
// Ce qui était pensé : "Peut-être que c'est 1 car le premier caractère compte"

/* Mutant B (Logic) : j = failure[i] au lieu de failure[i-1] */
pub fn compute_cognition_particles_mutant_b(pattern: &[u8]) -> Vec<usize> {
    let mut failure = vec![0; pattern.len()];
    let mut j = 0;
    for i in 1..pattern.len() {
        while j > 0 && pattern[i] != pattern[j] {
            j = failure[i];  // ❌ Doit être failure[j-1]
        }
        if pattern[i] == pattern[j] {
            j += 1;
        }
        failure[i] = j;
    }
    failure
}
// Pourquoi c'est faux : On doit suivre la chaîne depuis j, pas depuis i
// Ce qui était pensé : "Je recule dans le pattern à la position i"

/* Mutant C (Off-by-one) : r < n-1 au lieu de r < n dans Z-array */
pub fn construct_id_well_mutant_c(s: &[u8]) -> Vec<usize> {
    let n = s.len();
    let mut z = vec![0; n];
    let mut l = 0;
    let mut r = 0;

    for i in 1..n {
        if i > r {
            l = i;
            r = i;
            while r < n - 1 && s[r - l] == s[r] {  // ❌ Doit être r < n
                r += 1;
            }
            z[i] = r - l;
            r -= 1;
        }
        // ...
    }
    z
}
// Pourquoi c'est faux : Miss le dernier caractère dans les comparaisons
// Ce qui était pensé : "Je veux éviter un débordement"

/* Mutant D (Z-box) : oubli de r -= 1 */
pub fn construct_id_well_mutant_d(s: &[u8]) -> Vec<usize> {
    let n = s.len();
    let mut z = vec![0; n];
    let mut l = 0;
    let mut r = 0;

    for i in 1..n {
        if i > r {
            l = i;
            r = i;
            while r < n && s[r - l] == s[r] {
                r += 1;
            }
            z[i] = r - l;
            // ❌ OUBLI: r -= 1;
        }
        // ...
    }
    z
}
// Pourquoi c'est faux : r pointe après le Z-box au lieu de son dernier élément
// Ce qui était pensé : "r est la fin du Z-box"

/* Mutant E (Search) : retourne i au lieu de i - m + 1 */
pub fn mizuhanome_search_all_mutant_e(text: &[u8], pattern: &[u8]) -> Vec<usize> {
    // ...setup...
    let mut matches = vec![];
    let mut j = 0;

    for i in 0..text.len() {
        // ...matching logic...
        if j == pattern.len() {
            matches.push(i);  // ❌ Doit être i + 1 - m
            j = failure[j - 1];
        }
    }
    matches
}
// Pourquoi c'est faux : Retourne la fin du match au lieu du début
// Ce qui était pensé : "i est où on a trouvé le match"
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

1. **Pattern Matching en temps linéaire** : O(n+m) au lieu de O(n×m) naïf
2. **Prétraitement du pattern** : Amortir le coût sur toutes les recherches
3. **Réutilisation d'information** : Ne jamais recomparer ce qu'on sait déjà
4. **Applications pratiques** : Période, rotation, borders - concepts fondamentaux

### 5.2 LDA — Traduction littérale en français (MAJUSCULES)

**compute_cognition_particles :**
```
FONCTION compute_cognition_particles QUI RETOURNE UN VECTEUR D'ENTIERS ET PREND EN PARAMÈTRE pattern QUI EST UNE TRANCHE D'OCTETS
DÉBUT FONCTION
    DÉCLARER m COMME ENTIER
    AFFECTER LA LONGUEUR DE pattern À m
    SI m EST ÉGAL À 0 ALORS
        RETOURNER UN VECTEUR VIDE
    FIN SI

    DÉCLARER failure COMME VECTEUR D'ENTIERS DE TAILLE m INITIALISÉ À 0
    DÉCLARER j COMME ENTIER
    AFFECTER 0 À j

    POUR i ALLANT DE 1 À m MOINS 1 FAIRE
        TANT QUE j EST SUPÉRIEUR À 0 ET pattern[i] EST DIFFÉRENT DE pattern[j] FAIRE
            AFFECTER failure[j - 1] À j
        FIN TANT QUE

        SI pattern[i] EST ÉGAL À pattern[j] ALORS
            INCRÉMENTER j DE 1
        FIN SI

        AFFECTER j À failure[i]
    FIN POUR

    RETOURNER failure
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : KMP Pattern Matching
---
1. PRÉTRAITEMENT :
   a. Construire failure function pour pattern
   b. failure[i] = longueur du plus long préfixe propre = suffixe

2. RECHERCHE :
   a. i = 0 (position dans text)
   b. j = 0 (position dans pattern)

3. BOUCLE PRINCIPALE (i < n) :
   |
   |-- SI mismatch à position j :
   |     TANT QUE j > 0 ET text[i] != pattern[j] :
   |       j = failure[j-1]  // "Téléportation"
   |
   |-- SI text[i] == pattern[j] :
   |     j++
   |
   |-- SI j == m (match complet) :
   |     ENREGISTRER position (i - m + 1)
   |     j = failure[j-1]  // Continuer chercher
   |
   |-- i++

4. RETOURNER toutes les positions
```

### 5.2.3 Représentation Algorithmique (Logique de Garde)

```
FONCTION : Z-Array Construction
---
INIT z[0] = n, l = 0, r = 0

POUR i DE 1 À n-1 :
   |
   |-- CAS 1 : i > r (hors Z-box actuel)
   |     |
   |     |-- CALCULER z[i] depuis zéro
   |     |-- METTRE À JOUR l = i, r = i + z[i] - 1
   |
   |-- CAS 2 : i <= r (dans Z-box)
   |     |
   |     |-- k = i - l (position miroir)
   |     |
   |     |-- SI z[k] < r - i + 1 :
   |     |     z[i] = z[k]  // Copie directe
   |     |
   |     |-- SINON :
   |     |     ÉTENDRE depuis r
   |     |     METTRE À JOUR l, r

RETOURNER z
```

### 5.3 Visualisation ASCII

**KMP Failure Function pour "AABAACAABAA" :**
```
Pattern:     A  A  B  A  A  C  A  A  B  A  A
Index:       0  1  2  3  4  5  6  7  8  9  10
Failure:     0  1  0  1  2  0  1  2  3  4  5

Explication failure[10] = 5:
  Pattern:   A A B A A C A A B A A
             └─────────┘ └─────────┘
             Préfixe     Suffixe
             "AABAA"     "AABAA"   ← Longueur 5
```

**Z-Array pour "aabxaab" :**
```
String:   a  a  b  x  a  a  b
Index:    0  1  2  3  4  5  6
Z-value:  7  1  0  0  3  1  0

Z[4] = 3 expliqué:
  String:     a a b x [a a b]
              ↓ ↓ ↓    ↓ ↓ ↓
  Préfixe:   [a a b] x  a a b
              Match de 3 caractères
```

**Téléportation KMP lors d'un mismatch :**
```
Text:     A B A B A B C A B A B
Pattern:  A B A B C
                ↑
                Mismatch à j=4

Au lieu de: i=5, j=0 (recommencer)
KMP fait:   i=5, j=2 (failure[3] = 2)

Parce que: "AB" est à la fois préfixe ET suffixe de "ABAB"
           On sait déjà que text[3..5] = "AB" = pattern[0..2]
           Donc on continue depuis j=2
```

### 5.4 Les pièges en détail

| Piège | Conséquence | Solution |
|-------|-------------|----------|
| failure[0] = 1 | Tous les indices décalés | Toujours 0 |
| j = failure[i] au lieu de failure[j-1] | Boucle infinie | Suivre la chaîne depuis j |
| Oubli de r -= 1 dans Z | Z-values incorrects | r pointe sur le dernier, pas après |
| Position = i au lieu de i+1-m | Retourne fin au lieu de début | Calculer le début |
| Pattern vide non géré | Crash ou résultats faux | Return early |

### 5.5 Cours Complet

#### 5.5.1 Le Problème du Pattern Matching

**Approche naïve :** O(n × m)
- Pour chaque position i dans text
- Comparer pattern[0..m] avec text[i..i+m]
- Problème : On re-compare des caractères déjà vus

**Insight KMP :** Quand on a un mismatch après avoir matché k caractères, on sait que `text[i-k..i]` = `pattern[0..k]`. On peut utiliser cette information !

#### 5.5.2 La Failure Function

La failure function `f[j]` répond à la question :
"Si j'ai matché `pattern[0..j]` et que `pattern[j+1]` mismatch, où dois-je reprendre dans pattern ?"

**Propriété clé :** `f[j]` = longueur du plus long préfixe propre de `pattern[0..j+1]` qui est aussi un suffixe.

**Exemple pour "ABABC" :**
```
j=0: "A"     → Pas de préfixe propre → f[0] = 0
j=1: "AB"    → "A" prefix, "B" suffix, pas égaux → f[1] = 0
j=2: "ABA"   → "A" = "A" (préfixe = suffixe) → f[2] = 1
j=3: "ABAB"  → "AB" = "AB" → f[3] = 2
j=4: "ABABC" → Aucun match → f[4] = 0
```

#### 5.5.3 L'Algorithme Z

Le Z-array `z[i]` répond à : "Quelle est la longueur du plus long préfixe de S qui commence à position i ?"

**Optimisation Z-box :**
- Maintenir `[l, r]` = le Z-box le plus à droite déjà calculé
- Si `i` est dans `[l, r]`, on peut réutiliser `z[i - l]`

#### 5.5.4 Applications

**Période d'un string :**
- Si `n % (n - f[n-1]) == 0`, alors `n - f[n-1]` est la période
- Exemple: "abcabc" → f[5]=3 → période = 6-3 = 3 ✓

**Rotation :**
- s1 est rotation de s2 ⟺ s2 est substring de s1+s1
- Exemple: "bottle"+"water" = "bottlewater" contient "terbottlewa" ? Non → pas rotation

**Borders :**
- Un border est un préfixe qui est aussi un suffixe
- Suivre la chaîne: `n → f[n-1] → f[f[n-1]-1] → ... → 0`

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ while j > 0 && pattern[i] != pattern[j] { j = failure[i]; }    │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ while j > 0 && pattern[i] != pattern[j] {                      │
│     j = failure[j - 1];                                         │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • On suit la CHAÎNE depuis j, pas depuis i                      │
│ • failure[j-1] nous dit "si j échoue, essaie cette position"   │
│ • failure[i] serait regarder en avant, pas en arrière          │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Trace : KMP recherche "AABA" dans "AABAACAABA"**

```
Failure function pour "AABA": [0, 1, 0, 1]

┌───────┬───────────────────────────────────────────┬─────┬─────┬────────────────────┐
│ Étape │ Comparaison                               │  i  │  j  │ Action             │
├───────┼───────────────────────────────────────────┼─────┼─────┼────────────────────┤
│   1   │ text[0]='A' vs pattern[0]='A'            │  0  │  0  │ Match, j++         │
│   2   │ text[1]='A' vs pattern[1]='A'            │  1  │  1  │ Match, j++         │
│   3   │ text[2]='B' vs pattern[2]='B'            │  2  │  2  │ Match, j++         │
│   4   │ text[3]='A' vs pattern[3]='A'            │  3  │  3  │ Match, j++ → j=4=m │
│   5   │ MATCH TROUVÉ à position i-m+1 = 0        │  3  │  4  │ Enregistrer 0      │
│   6   │ j = failure[3] = 1                       │  3  │  1  │ Continuer          │
│   7   │ text[4]='A' vs pattern[1]='A'            │  4  │  1  │ Match, j++         │
│   8   │ text[5]='C' vs pattern[2]='B'            │  5  │  2  │ Mismatch           │
│   9   │ j = failure[1] = 1                       │  5  │  1  │ Téléportation      │
│  10   │ text[5]='C' vs pattern[1]='A'            │  5  │  1  │ Mismatch           │
│  11   │ j = failure[0] = 0                       │  5  │  0  │ Téléportation      │
│  12   │ text[5]='C' vs pattern[0]='A'            │  5  │  0  │ Mismatch, i++      │
│  13   │ text[6]='A' vs pattern[0]='A'            │  6  │  0  │ Match, j++         │
│  ...  │ ...continue...                           │ ... │ ... │ ...                │
│  20   │ MATCH TROUVÉ à position 6                │  9  │  4  │ Enregistrer 6      │
└───────┴───────────────────────────────────────────┴─────┴─────┴────────────────────┘

Résultat: [0, 6]
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🔍 MEME : "Find Kaeru" — La recherche constante

![Find Kaeru](meme_id_invaded.jpg)

Dans ID:Invaded, les détectives cherchent toujours "Kaeru" dans chaque id well.
Comme KMP qui cherche toujours le même pattern, peu importe le texte.

```rust
fn mizuhanome_search(id_well: &[u8], kaeru: &[u8]) -> Option<usize> {
    // "Sakaido, find Kaeru!"
    // "Roger, diving into the id well..."
}
```

#### 🎯 MEME : "Fast Travel unlocked" — La téléportation KMP

Quand tu as un mismatch après avoir matché, KMP te téléporte direct au bon endroit.
Comme le fast travel dans un jeu : tu ne marches pas tout le chemin.

```rust
// Mismatch à j=4 après avoir matché "ABAB"
// Naïf: retourne à i=1, j=0 (marche tout le chemin)
// KMP: j = failure[3] = 2 (FAST TRAVEL!)
```

#### 🌊 MEME : "Same Energy" — Le Z-Array

Le Z-array mesure "combien cette position a la même énergie que le début".
Z[i] = "Same energy level" avec le préfixe.

### 5.9 Applications pratiques

| Application | Algorithme | Exemple |
|-------------|------------|---------|
| Recherche ADN | KMP/Z | Trouver un gène dans un chromosome |
| Antivirus | Multi-pattern (Aho-Corasick) | Signatures de malware |
| Éditeur texte | KMP | Ctrl+F, Find & Replace |
| Compression | Z-array | LZ77 trouve les répétitions |
| Détection plagiat | Rolling hash + KMP | Comparer documents |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

1. **failure[0] toujours = 0** : Pas de préfixe propre d'un seul caractère
2. **Suivre la chaîne depuis j, pas i** : `failure[j-1]` pas `failure[i]`
3. **Z-array r -= 1** : r pointe sur le dernier élément, pas après
4. **Position = début du match** : `i + 1 - m` pas `i`
5. **Patterns vides** : Retourner vecteur vide
6. **Overlapping matches** : Continuer après un match avec `j = failure[j-1]`

---

## 📝 SECTION 7 : QCM

**Q1.** Quelle est la complexité de KMP pour chercher un pattern de taille m dans un texte de taille n ?
- A) O(n × m)
- B) O(n + m)
- C) O(n log m)
- D) O(n²)

**Q2.** Que représente failure[i] dans KMP ?
- A) Le nombre de matchs à la position i
- B) La longueur du plus long préfixe propre = suffixe de pattern[0..i+1]
- C) L'index du prochain caractère à comparer
- D) Le nombre total de comparaisons

**Q3.** Pour "ABAB", quelle est la failure function ?
- A) [0, 0, 0, 0]
- B) [0, 0, 1, 2]
- C) [0, 1, 2, 3]
- D) [1, 2, 3, 4]

**Q4.** Que signifie z[i] = 3 dans un Z-array ?
- A) Il y a 3 caractères avant la position i
- B) Les 3 premiers caractères du string = string[i..i+3]
- C) La position i apparaît 3 fois
- D) Il reste 3 caractères après i

**Q5.** Comment vérifier si "abc" est une rotation de "cab" ?
- A) Comparer les longueurs
- B) Chercher "cab" dans "abcabc"
- C) Comparer caractère par caractère
- D) Calculer les hash

**Q6.** Quelle est la période de "abcabcabc" ?
- A) 1
- B) 3
- C) 6
- D) 9

**Q7.** Pourquoi KMP est meilleur que l'approche naïve ?
- A) Il utilise moins de mémoire
- B) Il ne re-compare jamais un caractère déjà matché
- C) Il est plus simple à coder
- D) Il fonctionne sur tous les alphabets

**Q8.** Après un match complet en KMP, comment continue-t-on ?
- A) j = 0
- B) j = failure[m-1]
- C) j = failure[j-1]
- D) On s'arrête

**Réponses :** B, B, B, B, B, B, B, B (ou C si j=m après match)

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Algorithme | Prétraitement | Recherche | Espace | Force |
|------------|---------------|-----------|--------|-------|
| **Naïf** | O(1) | O(n × m) | O(1) | Simple |
| **KMP** | O(m) | O(n) | O(m) | Un pattern, streaming |
| **Z** | O(n+m) | O(n+m) | O(n+m) | Applications variées |
| **Rabin-Karp** | O(m) | O(n) avg | O(1) | Multi-pattern |

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.2.1-synth-mizuhanome-search",
    "generated_at": "2026-01-11 01:00:00",

    "metadata": {
      "exercise_id": "1.2.1-synth",
      "exercise_name": "mizuhanome_search",
      "module": "1.2",
      "module_name": "Hash Tables & Strings",
      "concept": "synth",
      "concept_name": "Pattern Matching",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 1,
      "difficulty": 5,
      "difficulty_stars": "★★★★★☆☆☆☆☆",
      "language": "rust",
      "duration_minutes": 90,
      "xp_base": 120,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T4 O(n+m)",
      "complexity_space": "S3 O(m)",
      "prerequisites": ["arrays", "strings"],
      "domains": ["Struct", "Encodage"],
      "domains_bonus": ["Struct", "DP"],
      "tags": ["kmp", "z-algorithm", "pattern-matching", "strings"],
      "meme_reference": "ID:Invaded - Find Kaeru"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 */",
      "mutants/mutant_a_boundary.rs": "/* failure[0] = 1 */",
      "mutants/mutant_b_logic.rs": "/* j = failure[i] */",
      "mutants/mutant_c_offbyone.rs": "/* r < n-1 */",
      "mutants/mutant_d_zbox.rs": "/* oubli r -= 1 */",
      "mutants/mutant_e_position.rs": "/* retourne i */",
      "tests/main.c": "/* Section 4.2 */"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "Sakaido, find Kaeru. The pattern is waiting in the id well."*
