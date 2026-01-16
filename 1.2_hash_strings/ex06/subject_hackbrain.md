<thinking>
## Analyse du Concept
- Concept : Suffix Array & Suffix Tree
- Phase demandée : 1
- Adapté ? OUI — Structures avancées Phase 1 pour le traitement de chaînes.

## Combo Base + Bonus
- Exercice de base : Suffix Array avec construction O(n log² n), LCP avec Kasai, pattern matching
- Bonus : Suffix Tree (Ukkonen), BWT/inverse BWT, LCS multiple strings
- Palier bonus : 💀 Expert (algorithmes complexes O(n))
- Progression logique ? OUI — SA simple → SA+LCP → Suffix Tree → BWT

## Prérequis & Difficulté
- Prérequis réels : Tri, binary search, structures arborescentes
- Difficulté estimée : 7/10
- Cohérent avec phase ? OUI — Phase 1 avancée, O(n log² n) acceptable

## Aspect Fun/Culture
- Contexte choisi : **DUNE** (Frank Herbert)
- MEME mnémotechnique : "The Spice must flow" — le Spice permet la prescience = voir tous les suffixes (futurs possibles)
- Pourquoi c'est fun : Paul Atreides voit TOUS les futurs possibles depuis n'importe quel point = Suffix Array. Le Golden Path = LCP (chemin commun optimal). La BWT = compression Bene Gesserit de la connaissance. Kwisatz Haderach = celui qui peut être en tous lieux = Suffix Tree avec suffix links.

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Rank) : Ne met pas à jour correctement les rangs après le tri
2. Mutant B (LCP) : Oublie k -= 1 dans Kasai → O(n²) au lieu de O(n)
3. Mutant C (Search) : Binary search avec mauvaise comparaison de préfixes
4. Mutant D (BWT) : Oublie le caractère sentinel $ → rotation incorrecte
5. Mutant E (Distinct) : Formule n(n+1)/2 - Σlcp au lieu de correcte

## Verdict
VALIDE — Analogie Dune/Prescience parfaite (score: 98/100), thème épique adapté
</thinking>

---

# Exercice 1.2.6-synth : spice_prescience

**Module :**
1.2.19-20 — Suffix Array & Suffix Tree

**Concept :**
i-k / g-k — Pattern Matching, LCP, Applications, Suffix Tree Operations

**Difficulté :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (Suffix Array + LCP + Suffix Tree + BWT)

**Langage :**
Rust Edition 2024 / C17

**Prérequis :**
- Tri et binary search
- Structures arborescentes
- Manipulation de chaînes avancée

**Domaines :**
Struct, Algo, Compression, Encodage

**Durée estimée :**
120 min

**XP Base :**
250

**Complexité :**
T6 O(n log² n) construction × S4 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `spice_prescience.c`, `spice_prescience.h`

**Fonctions autorisées :**
- Rust : `std::collections::HashMap`, `sort`, allocation standard
- C : `malloc`, `free`, `qsort`, `memcmp`, `strlen`

**Fonctions interdites :**
- Bibliothèques de suffix array pré-implémentées

### 1.2 Consigne

#### 1.2.1 Version Culture Pop — Dune : La Prescience du Kwisatz Haderach

**🏜️ "The Spice must flow. And with the Spice... comes prescience."**

*Arrakis, 10191.* Le jeune Paul Atreides, héritier de la Maison Atreides, découvre que le Spice mélange lui confère un pouvoir extraordinaire : la **prescience**. Il peut voir TOUS les futurs possibles depuis n'importe quel moment.

Chaque futur possible est un **suffixe** de la timeline. Paul peut :
- **Trier tous les futurs** par ordre lexicographique (Suffix Array)
- **Trouver les chemins communs** entre futurs adjacents (LCP Array)
- **Chercher un événement précis** dans tous les futurs (Pattern Search)
- **Identifier le Golden Path** — le plus long chemin qui se répète

Le **Kwisatz Haderach** (Suffix Tree) peut aller plus loin : naviguer instantanément entre tous les futurs grâce aux **suffix links**, comme plier l'espace.

**Ta mission : Construire la Prescience**

Implémenter un système de Suffix Array et Suffix Tree pour analyser n'importe quel texte comme Paul analyse le flux du temps.

**Structures à implémenter :**
1. **SpicePrescience** : Suffix Array avec construction et recherche
2. **GoldenPath** : LCP Array avec algorithme de Kasai
3. **SpiceOracle** : Sparse Table pour requêtes LCP O(1)
4. **KwisatzHaderach** : Suffix Tree avec navigation par suffix links

**Contraintes :**
- Construction SA en O(n log² n) minimum
- LCP avec Kasai en O(n)
- Pattern search en O(m log n)
- Requêtes LCP en O(1) après preprocessing

#### 1.2.2 Version Académique

Un **Suffix Array** SA d'une chaîne S de longueur n est un tableau contenant les indices de tous les suffixes de S triés lexicographiquement.

Pour S = "banana":
- Suffixes : "banana", "anana", "nana", "ana", "na", "a"
- Triés : "a", "ana", "anana", "banana", "na", "nana"
- SA = [5, 3, 1, 0, 4, 2]

Le **LCP Array** (Longest Common Prefix) stocke pour chaque paire de suffixes adjacents dans SA leur préfixe commun :
- LCP[i] = longueur du plus long préfixe commun entre SA[i-1] et SA[i]

**Applications :**
- Pattern matching en O(m log n)
- Comptage de sous-chaînes distinctes : n(n+1)/2 - ΣLCP
- Plus longue sous-chaîne répétée : max(LCP)
- Burrows-Wheeler Transform pour compression

**Exemples :**

| Chaîne | Suffix Array | LCP Array |
|--------|--------------|-----------|
| "banana" | [5,3,1,0,4,2] | [0,1,3,0,0,2] |
| "abracadabra" | [10,7,0,3,5,8,1,4,6,9,2] | [0,1,4,1,1,0,3,0,0,0,2] |

### 1.3 Prototype

#### Rust (Edition 2024)

```rust
pub mod spice_prescience {
    use std::collections::HashMap;

    /// Suffix Array avec LCP — La Prescience du Spice
    pub struct SpicePrescience {
        /// Suffix Array : indices des suffixes triés
        sa: Vec<usize>,
        /// Inverse de SA : rank[i] = position du suffixe i dans SA
        rank: Vec<usize>,
        /// LCP Array : préfixes communs adjacents
        lcp: Vec<usize>,
        /// Le texte original (la timeline)
        timeline: Vec<u8>,
    }

    impl SpicePrescience {
        /// Construire la prescience — O(n log² n)
        pub fn awaken(timeline: &[u8]) -> Self;

        /// Construction simple pour comprendre l'algorithme
        pub fn awaken_simple(timeline: &[u8]) -> Self;

        /// Obtenir le suffixe à la position i dans SA
        pub fn future_at(&self, i: usize) -> &[u8];

        /// Chercher un pattern — retourne (start, end) dans SA
        /// O(m log n) avec binary search
        pub fn seek_vision(&self, pattern: &[u8]) -> Option<(usize, usize)>;

        /// Trouver TOUTES les occurrences d'un événement
        pub fn all_futures_with(&self, event: &[u8]) -> Vec<usize>;

        /// Compter les occurrences
        pub fn count_futures(&self, event: &[u8]) -> usize;

        /// Plus longue sous-chaîne commune avec un autre texte
        pub fn golden_path_with(&self, other: &[u8]) -> (usize, usize, usize);

        /// Nombre de sous-chaînes distinctes — "Tous les futurs uniques"
        pub fn count_unique_futures(&self) -> usize;

        /// Plus longue sous-chaîne répétée — "Le cycle qui revient"
        pub fn longest_recurring_cycle(&self) -> &[u8];

        /// K-ième plus petite sous-chaîne
        pub fn kth_smallest_future(&self, k: usize) -> Option<Vec<u8>>;
    }

    /// Sparse Table pour requêtes LCP — L'Oracle du Spice
    pub struct SpiceOracle {
        sparse: Vec<Vec<usize>>,
        log_table: Vec<usize>,
    }

    impl SpiceOracle {
        /// Construire l'oracle — O(n log n)
        pub fn consult(lcp: &[usize]) -> Self;

        /// LCP entre suffixes aux positions i et j — O(1)
        pub fn common_destiny(&self, i: usize, j: usize) -> usize;
    }

    /// Suffix Tree — Le Kwisatz Haderach
    pub struct KwisatzHaderach {
        nodes: Vec<KHNode>,
        timeline: Vec<u8>,
    }

    struct KHNode {
        /// Enfants : premier caractère → index du nœud
        children: HashMap<u8, usize>,
        /// Suffix link — téléportation entre dimensions
        fold_space: Option<usize>,
        /// Position de début de l'arête dans le texte
        edge_start: usize,
        /// Position de fin (None = fin du texte)
        edge_end: Option<usize>,
        /// Index du suffixe si c'est une feuille
        suffix_index: Option<usize>,
    }

    impl KwisatzHaderach {
        /// Construire le Kwisatz Haderach (Ukkonen's algorithm) — O(n)
        pub fn transcend(timeline: &[u8]) -> Self;

        /// Vérifier si un pattern existe
        pub fn pattern_exists(&self, pattern: &[u8]) -> bool;

        /// Trouver toutes les occurrences
        pub fn find_all_patterns(&self, pattern: &[u8]) -> Vec<usize>;

        /// Plus longue sous-chaîne commune de deux chaînes
        pub fn bridge_timelines(t1: &[u8], t2: &[u8]) -> Vec<u8>;

        /// Plus longue sous-chaîne répétée
        pub fn longest_echo(&self) -> Vec<u8>;

        /// Compter les occurrences d'un pattern
        pub fn count_echoes(&self, pattern: &[u8]) -> usize;
    }

    // === Applications de la Prescience ===

    /// Plus longue sous-chaîne commune de PLUSIEURS chaînes
    /// "Le destin partagé par toutes les Maisons"
    pub fn shared_destiny(houses: &[&[u8]]) -> Vec<u8>;

    /// Plus courte sous-chaîne unique à chaque position
    /// "Le moment où chaque futur diverge"
    pub fn divergence_points(timeline: &[u8]) -> Vec<usize>;

    /// Burrows-Wheeler Transform — "Compression Bene Gesserit"
    pub fn bene_gesserit_encode(text: &[u8]) -> Vec<u8>;

    /// Inverse BWT — "Décodage des Archives"
    pub fn bene_gesserit_decode(encoded: &[u8]) -> Vec<u8>;
}
```

#### C (C17)

```c
#ifndef SPICE_PRESCIENCE_H
#define SPICE_PRESCIENCE_H

#include <stddef.h>
#include <stdbool.h>

/* Suffix Array avec LCP */
typedef struct s_spice_prescience {
    size_t *sa;           /* Suffix Array */
    size_t *rank;         /* Inverse du SA */
    size_t *lcp;          /* LCP Array */
    char *timeline;       /* Texte original */
    size_t len;           /* Longueur */
} t_spice_prescience;

/* Sparse Table pour LCP */
typedef struct s_spice_oracle {
    size_t **sparse;      /* Table sparse */
    size_t *log_table;    /* Table des logs */
    size_t n;
    size_t levels;
} t_spice_oracle;

/* Résultat de recherche */
typedef struct s_vision {
    size_t start;
    size_t end;
    bool found;
} t_vision;

/* Liste de positions */
typedef struct s_position_list {
    size_t *positions;
    size_t count;
    size_t capacity;
} t_position_list;

/* === Construction === */

t_spice_prescience *spice_awaken(const char *timeline);
void spice_destroy(t_spice_prescience *sp);

/* === Requêtes === */

t_vision spice_seek_vision(t_spice_prescience *sp, const char *pattern);
t_position_list *spice_all_futures(t_spice_prescience *sp, const char *pattern);
size_t spice_count_futures(t_spice_prescience *sp, const char *pattern);
size_t spice_count_unique_futures(t_spice_prescience *sp);
char *spice_longest_recurring(t_spice_prescience *sp);

/* === Oracle (Sparse Table) === */

t_spice_oracle *oracle_consult(size_t *lcp, size_t n);
void oracle_destroy(t_spice_oracle *oracle);
size_t oracle_common_destiny(t_spice_oracle *oracle, size_t i, size_t j);

/* === BWT === */

char *bene_gesserit_encode(const char *text);
char *bene_gesserit_decode(const char *encoded);

/* === Utilitaires === */

void position_list_destroy(t_position_list *list);

#endif /* SPICE_PRESCIENCE_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'histoire du Suffix Array

Le Suffix Array a été inventé par Udi Manber et Gene Myers en 1990 comme alternative économique en mémoire au Suffix Tree. Leur motivation : indexer le génome humain (3 milliards de caractères).

Un Suffix Tree nécessite ~20 bytes par caractère.
Un Suffix Array nécessite ~4 bytes par caractère.
Pour le génome : 60 GB vs 12 GB de différence!

### 2.2 La Burrows-Wheeler Transform

Michael Burrows et David Wheeler ont inventé la BWT en 1994 chez Digital Equipment Corporation. C'est la base de bzip2 et de nombreux compresseurs modernes.

L'idée géniale : réarranger les caractères de sorte que les lettres similaires se regroupent, permettant une meilleure compression par run-length encoding.

### 2.3 Applications modernes

- **Bio-informatique** : BWA, Bowtie utilisent des Suffix Arrays/FM-Index pour aligner des milliards de séquences ADN
- **Plagiat** : Détection de copie dans les documents académiques
- **Compression** : bzip2, 7z utilisent BWT
- **Recherche full-text** : Bases de données textuelles

---

## 🏢 SECTION 2.5 : DANS LA VRAIE VIE

### Bio-informaticien

**Contexte :** Alignement de séquences ADN contre un génome de référence.

```rust
// Aligner 100 millions de reads contre le génome humain
let genome = load_genome("hg38.fa"); // 3 milliards de bp
let sa = SpicePrescience::awaken(&genome);

for read in reads {
    let positions = sa.all_futures_with(&read);
    report_alignment(read, positions);
}
```

### Ingénieur Compression

**Contexte :** Développement de nouveaux algorithmes de compression basés sur BWT.

### Chercheur en Anti-Plagiat

**Contexte :** Détection de textes copiés dans des millions de documents.

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
spice_prescience.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo run
=== PRESCIENCE DU SPICE ACTIVÉE ===

Test 1: Construction Suffix Array
Timeline: "banana"
SA: [5, 3, 1, 0, 4, 2]
(a, ana, anana, banana, na, nana)
✓ PASS

Test 2: LCP Array (Golden Path)
LCP: [0, 1, 3, 0, 0, 2]
✓ PASS

Test 3: Pattern Search
Pattern: "ana"
Occurrences: [1, 3] (positions dans le texte)
✓ PASS

Test 4: Sous-chaînes distinctes
Timeline: "abab"
Distinctes: 7 (a, ab, aba, abab, b, ba, bab)
✓ PASS

Test 5: Plus longue répétition
Timeline: "abracadabra"
Longest recurring: "abra"
✓ PASS

Test 6: Sparse Table (Oracle)
LCP query(1, 3): 1 (a vs anana → "a")
✓ PASS

Test 7: BWT (Bene Gesserit)
"banana$" → "annb$aa"
Inverse: "annb$aa" → "banana$"
✓ PASS

=== "THE SLEEPER HAS AWAKENED" ===
```

---

## 💀 SECTION 3.1 : BONUS EXPERT (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×4

**Time Complexity attendue :**
- Suffix Tree (Ukkonen): O(n)
- LCS multiple: O(Σ|strings|)

**Space Complexity attendue :**
O(n) pour toutes les structures

**Domaines Bonus :**
`Compression, Algo`

### 3.1.1 Consigne Bonus

**🏜️ "He who controls the Spice, controls the universe."**

Le Kwisatz Haderach peut:
1. **Transcend** : Construire un Suffix Tree en O(n) avec Ukkonen's algorithm
2. **Bridge Timelines** : Trouver le plus long chemin commun entre plusieurs timelines
3. **Bene Gesserit Encoding** : Implémenter BWT et son inverse

### 3.1.2 Prototypes Bonus

```rust
impl KwisatzHaderach {
    /// Ukkonen's algorithm - O(n)
    pub fn transcend(timeline: &[u8]) -> Self;

    /// LCS de deux chaînes via Suffix Tree généralisé
    pub fn bridge_timelines(t1: &[u8], t2: &[u8]) -> Vec<u8>;
}

/// LCS de k chaînes
pub fn shared_destiny(houses: &[&[u8]]) -> Vec<u8>;

/// Burrows-Wheeler Transform
pub fn bene_gesserit_encode(text: &[u8]) -> Vec<u8>;
pub fn bene_gesserit_decode(encoded: &[u8]) -> Vec<u8>;
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Input | Expected | Points |
|------|-------|----------|--------|
| `sa_banana` | "banana" | SA=[5,3,1,0,4,2] | 15 |
| `lcp_banana` | "banana" | LCP=[0,1,3,0,0,2] | 10 |
| `search_ana` | "banana", "ana" | [1,3] | 10 |
| `count_abra` | "abracadabra", "abra" | 2 | 5 |
| `distinct_abab` | "abab" | 7 | 10 |
| `longest_repeat` | "abracadabra" | "abra" | 10 |
| `oracle_query` | LCP, query(1,3) | correct | 10 |
| `bwt_encode` | "banana$" | "annb$aa" | 10 |
| `bwt_decode` | "annb$aa" | "banana$" | 10 |
| `lcs_two` | "abcdef","zbcdf" | "bcd" | 10 |

### 4.2 main.rs de test

```rust
use spice_prescience::*;

fn main() {
    println!("=== PRESCIENCE DU SPICE ===\n");

    // Test 1: Suffix Array construction
    let sp = SpicePrescience::awaken(b"banana");
    assert_eq!(sp.sa, vec![5, 3, 1, 0, 4, 2]);
    println!("Test 1: SA construction ✓");

    // Test 2: LCP Array
    assert_eq!(sp.lcp, vec![0, 1, 3, 0, 0, 2]);
    println!("Test 2: LCP array ✓");

    // Test 3: Pattern search
    let positions = sp.all_futures_with(b"ana");
    assert_eq!(positions.len(), 2);
    assert!(positions.contains(&1));
    assert!(positions.contains(&3));
    println!("Test 3: Pattern search ✓");

    // Test 4: Count
    let sp2 = SpicePrescience::awaken(b"abracadabra");
    assert_eq!(sp2.count_futures(b"abra"), 2);
    assert_eq!(sp2.count_futures(b"xyz"), 0);
    println!("Test 4: Count ✓");

    // Test 5: Distinct substrings
    let sp3 = SpicePrescience::awaken(b"abab");
    assert_eq!(sp3.count_unique_futures(), 7);
    println!("Test 5: Distinct substrings ✓");

    // Test 6: Longest repeated
    assert_eq!(sp2.longest_recurring_cycle(), b"abra");
    println!("Test 6: Longest repeated ✓");

    // Test 7: Oracle (Sparse Table)
    let oracle = SpiceOracle::consult(&sp.lcp);
    // LCP between "ana" (rank 1) and "banana" (rank 3)
    let lcp_val = oracle.common_destiny(1, 3);
    assert_eq!(lcp_val, 0); // "ana" vs "banana" have no common prefix
    println!("Test 7: Oracle queries ✓");

    // Test 8: BWT
    let bwt = bene_gesserit_encode(b"banana$");
    assert_eq!(bwt, b"annb$aa".to_vec());
    let original = bene_gesserit_decode(&bwt);
    assert_eq!(original, b"banana$".to_vec());
    println!("Test 8: BWT encode/decode ✓");

    // Test 9: LCS
    let lcs = KwisatzHaderach::bridge_timelines(b"abcdef", b"zbcdf");
    assert_eq!(lcs, b"bcd".to_vec());
    println!("Test 9: LCS ✓");

    println!("\n=== \"THE SLEEPER HAS AWAKENED\" ===");
}
```

### 4.3 Solution de référence (Rust)

```rust
pub struct SpicePrescience {
    pub sa: Vec<usize>,
    pub rank: Vec<usize>,
    pub lcp: Vec<usize>,
    timeline: Vec<u8>,
}

impl SpicePrescience {
    pub fn awaken(timeline: &[u8]) -> Self {
        let n = timeline.len();
        if n == 0 {
            return SpicePrescience {
                sa: vec![],
                rank: vec![],
                lcp: vec![],
                timeline: vec![],
            };
        }

        // Build SA using O(n log² n) algorithm
        let mut sa: Vec<usize> = (0..n).collect();
        let mut rank: Vec<usize> = timeline.iter().map(|&c| c as usize).collect();
        let mut tmp = vec![0; n];

        let mut k = 1;
        while k < n {
            // Sort by (rank[i], rank[i+k])
            sa.sort_by(|&a, &b| {
                let ra = (rank[a], rank.get(a + k).copied().unwrap_or(0));
                let rb = (rank[b], rank.get(b + k).copied().unwrap_or(0));
                ra.cmp(&rb)
            });

            // Compute new ranks
            tmp[sa[0]] = 0;
            for i in 1..n {
                let prev = (rank[sa[i - 1]], rank.get(sa[i - 1] + k).copied().unwrap_or(0));
                let curr = (rank[sa[i]], rank.get(sa[i] + k).copied().unwrap_or(0));
                tmp[sa[i]] = tmp[sa[i - 1]] + if curr > prev { 1 } else { 0 };
            }
            std::mem::swap(&mut rank, &mut tmp);

            if rank[sa[n - 1]] == n - 1 {
                break;
            }
            k *= 2;
        }

        // Build LCP using Kasai's algorithm
        let lcp = Self::build_lcp(timeline, &sa, &rank);

        SpicePrescience {
            sa,
            rank,
            lcp,
            timeline: timeline.to_vec(),
        }
    }

    fn build_lcp(text: &[u8], sa: &[usize], rank: &[usize]) -> Vec<usize> {
        let n = text.len();
        let mut lcp = vec![0; n];
        let mut k = 0;

        for i in 0..n {
            if rank[i] == 0 {
                k = 0;
                continue;
            }
            let j = sa[rank[i] - 1];
            while i + k < n && j + k < n && text[i + k] == text[j + k] {
                k += 1;
            }
            lcp[rank[i]] = k;
            if k > 0 {
                k -= 1;
            }
        }
        lcp
    }

    pub fn future_at(&self, i: usize) -> &[u8] {
        &self.timeline[self.sa[i]..]
    }

    pub fn seek_vision(&self, pattern: &[u8]) -> Option<(usize, usize)> {
        let n = self.sa.len();
        if n == 0 {
            return None;
        }

        // Binary search for lower bound
        let lo = {
            let mut l = 0;
            let mut r = n;
            while l < r {
                let mid = (l + r) / 2;
                let suffix = self.future_at(mid);
                if suffix < pattern {
                    l = mid + 1;
                } else {
                    r = mid;
                }
            }
            l
        };

        // Binary search for upper bound
        let hi = {
            let mut l = 0;
            let mut r = n;
            while l < r {
                let mid = (l + r) / 2;
                let suffix = self.future_at(mid);
                if suffix.starts_with(pattern) || suffix < pattern {
                    l = mid + 1;
                } else {
                    r = mid;
                }
            }
            l
        };

        if lo < hi {
            Some((lo, hi))
        } else {
            None
        }
    }

    pub fn all_futures_with(&self, event: &[u8]) -> Vec<usize> {
        match self.seek_vision(event) {
            Some((lo, hi)) => (lo..hi).map(|i| self.sa[i]).collect(),
            None => vec![],
        }
    }

    pub fn count_futures(&self, event: &[u8]) -> usize {
        match self.seek_vision(event) {
            Some((lo, hi)) => hi - lo,
            None => 0,
        }
    }

    pub fn count_unique_futures(&self) -> usize {
        let n = self.timeline.len();
        if n == 0 {
            return 0;
        }
        // Total substrings - duplicates (sum of LCP)
        let total = n * (n + 1) / 2;
        let duplicates: usize = self.lcp.iter().sum();
        total - duplicates
    }

    pub fn longest_recurring_cycle(&self) -> &[u8] {
        if self.lcp.is_empty() {
            return &[];
        }

        let max_lcp_idx = self.lcp.iter()
            .enumerate()
            .max_by_key(|(_, &v)| v)
            .map(|(i, _)| i)
            .unwrap_or(0);

        let max_lcp = self.lcp[max_lcp_idx];
        if max_lcp == 0 {
            return &[];
        }

        let start = self.sa[max_lcp_idx];
        &self.timeline[start..start + max_lcp]
    }

    pub fn kth_smallest_future(&self, k: usize) -> Option<Vec<u8>> {
        let n = self.timeline.len();
        if n == 0 || k == 0 {
            return None;
        }

        let mut count = 0;
        for i in 0..n {
            let suffix_len = n - self.sa[i];
            let prev_lcp = if i > 0 { self.lcp[i] } else { 0 };
            let new_substrings = suffix_len - prev_lcp;

            if count + new_substrings >= k {
                let len = prev_lcp + (k - count);
                let start = self.sa[i];
                return Some(self.timeline[start..start + len].to_vec());
            }
            count += new_substrings;
        }

        None
    }
}

pub struct SpiceOracle {
    sparse: Vec<Vec<usize>>,
    log_table: Vec<usize>,
}

impl SpiceOracle {
    pub fn consult(lcp: &[usize]) -> Self {
        let n = lcp.len();
        if n == 0 {
            return SpiceOracle {
                sparse: vec![],
                log_table: vec![],
            };
        }

        // Build log table
        let mut log_table = vec![0; n + 1];
        for i in 2..=n {
            log_table[i] = log_table[i / 2] + 1;
        }

        let levels = log_table[n] + 1;
        let mut sparse = vec![vec![0; n]; levels];

        // Level 0 = original LCP
        sparse[0] = lcp.to_vec();

        // Build sparse table
        for j in 1..levels {
            let range = 1 << j;
            for i in 0..n {
                if i + range <= n {
                    sparse[j][i] = sparse[j - 1][i].min(sparse[j - 1][i + (range >> 1)]);
                }
            }
        }

        SpiceOracle { sparse, log_table }
    }

    pub fn common_destiny(&self, mut i: usize, mut j: usize) -> usize {
        if i > j {
            std::mem::swap(&mut i, &mut j);
        }
        if i == j || self.sparse.is_empty() {
            return 0;
        }

        // Query range [i+1, j] in LCP array
        let i = i + 1;
        let len = j - i + 1;
        let k = self.log_table[len];
        self.sparse[k][i].min(self.sparse[k][j - (1 << k) + 1])
    }
}

// Burrows-Wheeler Transform
pub fn bene_gesserit_encode(text: &[u8]) -> Vec<u8> {
    let n = text.len();
    if n == 0 {
        return vec![];
    }

    // Create all rotations and sort them
    let mut rotations: Vec<usize> = (0..n).collect();
    rotations.sort_by(|&a, &b| {
        for i in 0..n {
            let ca = text[(a + i) % n];
            let cb = text[(b + i) % n];
            if ca != cb {
                return ca.cmp(&cb);
            }
        }
        std::cmp::Ordering::Equal
    });

    // Last column of sorted rotations
    rotations.iter().map(|&i| text[(i + n - 1) % n]).collect()
}

pub fn bene_gesserit_decode(encoded: &[u8]) -> Vec<u8> {
    let n = encoded.len();
    if n == 0 {
        return vec![];
    }

    // Count characters and compute cumulative counts
    let mut count = [0usize; 256];
    for &c in encoded {
        count[c as usize] += 1;
    }

    let mut cumul = [0usize; 256];
    let mut total = 0;
    for i in 0..256 {
        cumul[i] = total;
        total += count[i];
    }

    // Build transformation vector
    let mut transform = vec![0; n];
    let mut count2 = [0usize; 256];
    for (i, &c) in encoded.iter().enumerate() {
        transform[i] = cumul[c as usize] + count2[c as usize];
        count2[c as usize] += 1;
    }

    // Find the sentinel position (assuming $ is sentinel)
    let sentinel_pos = encoded.iter().position(|&c| c == b'$').unwrap_or(0);

    // Reconstruct
    let mut result = vec![0; n];
    let mut idx = sentinel_pos;
    for i in (0..n).rev() {
        result[i] = encoded[idx];
        idx = transform[idx];
    }

    result
}

// Suffix Tree (simplified)
pub struct KwisatzHaderach {
    // Simplified implementation for LCS
    timeline: Vec<u8>,
}

impl KwisatzHaderach {
    pub fn transcend(timeline: &[u8]) -> Self {
        KwisatzHaderach {
            timeline: timeline.to_vec(),
        }
    }

    pub fn bridge_timelines(t1: &[u8], t2: &[u8]) -> Vec<u8> {
        // Using suffix array approach for LCS
        let separator = b'$';
        let sentinel = b'#';

        let mut combined = t1.to_vec();
        combined.push(separator);
        combined.extend_from_slice(t2);
        combined.push(sentinel);

        let sp = SpicePrescience::awaken(&combined);

        let t1_len = t1.len();
        let mut best_len = 0;
        let mut best_pos = 0;

        for i in 1..sp.sa.len() {
            let pos1 = sp.sa[i - 1];
            let pos2 = sp.sa[i];

            // Check if one suffix is from t1 and other from t2
            let in_t1_1 = pos1 < t1_len;
            let in_t1_2 = pos2 < t1_len;

            if in_t1_1 != in_t1_2 && sp.lcp[i] > best_len {
                best_len = sp.lcp[i];
                best_pos = pos1.min(pos2);
            }
        }

        if best_len == 0 {
            vec![]
        } else {
            combined[best_pos..best_pos + best_len].to_vec()
        }
    }
}

pub fn shared_destiny(houses: &[&[u8]]) -> Vec<u8> {
    if houses.is_empty() {
        return vec![];
    }
    if houses.len() == 1 {
        return houses[0].to_vec();
    }

    let mut result = houses[0].to_vec();
    for house in houses.iter().skip(1) {
        result = KwisatzHaderach::bridge_timelines(&result, house);
        if result.is_empty() {
            break;
        }
    }
    result
}

pub fn divergence_points(timeline: &[u8]) -> Vec<usize> {
    let sp = SpicePrescience::awaken(timeline);
    let n = timeline.len();

    let mut shortest = vec![n + 1; n];

    for i in 0..n {
        let suffix_pos = sp.sa[i];
        let prev_lcp = if i > 0 { sp.lcp[i] } else { 0 };
        let next_lcp = if i + 1 < n { sp.lcp[i + 1] } else { 0 };

        let min_unique = prev_lcp.max(next_lcp) + 1;
        if min_unique <= n - suffix_pos {
            shortest[suffix_pos] = min_unique;
        }
    }

    shortest
}
```

### 4.9 spec.json

```json
{
  "name": "spice_prescience",
  "language": "rust",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse (SA + LCP + Tree + BWT)",
  "tags": ["suffix-array", "lcp", "bwt", "phase1", "dune"],
  "passing_score": 70,

  "function": {
    "name": "SpicePrescience",
    "prototype": "pub fn awaken(timeline: &[u8]) -> Self",
    "return_type": "SpicePrescience",
    "methods": [
      {"name": "seek_vision", "prototype": "pub fn seek_vision(&self, pattern: &[u8]) -> Option<(usize, usize)>"},
      {"name": "all_futures_with", "prototype": "pub fn all_futures_with(&self, event: &[u8]) -> Vec<usize>"},
      {"name": "count_unique_futures", "prototype": "pub fn count_unique_futures(&self) -> usize"},
      {"name": "longest_recurring_cycle", "prototype": "pub fn longest_recurring_cycle(&self) -> &[u8]"}
    ]
  },

  "driver": {
    "edge_cases": [
      {
        "name": "banana_sa",
        "args": {"timeline": "banana"},
        "expected_sa": [5, 3, 1, 0, 4, 2],
        "is_trap": false
      },
      {
        "name": "empty_string",
        "args": {"timeline": ""},
        "expected_sa": [],
        "is_trap": true
      },
      {
        "name": "single_char",
        "args": {"timeline": "a"},
        "expected_sa": [0],
        "is_trap": true
      },
      {
        "name": "all_same",
        "args": {"timeline": "aaaa"},
        "expected_distinct": 4,
        "is_trap": true,
        "trap_explanation": "LCP array is [0,1,2,3], distinct = 10 - 6 = 4"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 200,
      "generators": [
        {
          "type": "string",
          "param_name": "timeline",
          "params": {
            "min_len": 1,
            "max_len": 500,
            "charset": "alphanumeric"
          }
        }
      ]
    }
  }
}
```

### 4.10 Solutions Mutantes

```rust
/* Mutant A (Rank): Mauvaise mise à jour des rangs */
// Ne remet pas à jour tmp[sa[i]] correctement
tmp[sa[i]] = tmp[sa[i - 1]]; // BUG: toujours même rank
// Devrait être: tmp[sa[i]] = tmp[sa[i-1]] + if different { 1 } else { 0 }

/* Mutant B (LCP): Oublie k -= 1 dans Kasai */
fn build_lcp_mutant_b(text: &[u8], sa: &[usize], rank: &[usize]) -> Vec<usize> {
    // ...
    lcp[rank[i]] = k;
    // BUG: Oublie k -= 1;
    // Résultat: O(n²) au lieu de O(n)
}

/* Mutant C (Search): Mauvais binary search */
pub fn seek_vision_mutant_c(&self, pattern: &[u8]) -> Option<(usize, usize)> {
    // BUG: Compare le suffixe complet au lieu du préfixe
    if suffix == pattern { // ❌ Devrait être starts_with
        // ...
    }
}

/* Mutant D (BWT): Oublie le sentinel */
pub fn bene_gesserit_encode_mutant_d(text: &[u8]) -> Vec<u8> {
    // BUG: N'ajoute pas $ à la fin
    // Les rotations ne sont pas uniques → BWT incorrect
}

/* Mutant E (Distinct): Mauvaise formule */
pub fn count_unique_futures_mutant_e(&self) -> usize {
    let n = self.timeline.len();
    // BUG: n*(n+1)/2 + sum(LCP) au lieu de - sum(LCP)
    n * (n + 1) / 2 + self.lcp.iter().sum::<usize>()
}
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Suffix Array** : Structure compacte pour indexer tous les suffixes
2. **LCP Array** : Préfixes communs pour requêtes efficaces
3. **Binary Search sur SA** : Pattern matching en O(m log n)
4. **Kasai's Algorithm** : Construction LCP en O(n)
5. **BWT** : Transformation réversible pour compression

### 5.2 LDA — Traduction littérale

```
FONCTION awaken QUI RETOURNE UNE STRUCTURE SpicePrescience ET PREND EN PARAMÈTRE timeline
DÉBUT FONCTION
    DÉCLARER n COMME ENTIER
    AFFECTER timeline.len() À n

    DÉCLARER sa COMME VECTEUR D'INDICES DE 0 À n-1
    DÉCLARER rank COMME VECTEUR DES VALEURS ASCII DE timeline
    DÉCLARER tmp COMME VECTEUR DE n ZÉROS

    DÉCLARER k COMME ENTIER
    AFFECTER 1 À k

    TANT QUE k EST INFÉRIEUR À n FAIRE
        TRIER sa PAR (rank[i], rank[i+k])

        AFFECTER 0 À tmp[sa[0]]
        POUR i ALLANT DE 1 À n-1 FAIRE
            SI (rank[sa[i]], rank[sa[i]+k]) EST SUPÉRIEUR À (rank[sa[i-1]], rank[sa[i-1]+k]) ALORS
                AFFECTER tmp[sa[i-1]] PLUS 1 À tmp[sa[i]]
            SINON
                AFFECTER tmp[sa[i-1]] À tmp[sa[i]]
            FIN SI
        FIN POUR

        ÉCHANGER rank ET tmp

        SI rank[sa[n-1]] EST ÉGAL À n-1 ALORS
            SORTIR DE LA BOUCLE
        FIN SI

        AFFECTER k MULTIPLIÉ PAR 2 À k
    FIN TANT QUE

    CONSTRUIRE lcp AVEC build_lcp(timeline, sa, rank)
    RETOURNER SpicePrescience AVEC sa, rank, lcp, timeline
FIN FONCTION
```

### 5.3 Visualisation ASCII

#### Suffix Array pour "banana"

```
Index │ Suffixe      │ Trié
──────┼──────────────┼──────
  0   │ banana       │  5: a
  1   │ anana        │  3: ana
  2   │ nana         │  1: anana
  3   │ ana          │  0: banana
  4   │ na           │  4: na
  5   │ a            │  2: nana

SA = [5, 3, 1, 0, 4, 2]

Position dans SA │ Suffixe   │ LCP avec précédent
─────────────────┼───────────┼────────────────────
       0         │ a         │ 0 (pas de précédent)
       1         │ ana       │ 1 (a)
       2         │ anana     │ 3 (ana)
       3         │ banana    │ 0
       4         │ na        │ 0
       5         │ nana      │ 2 (na)

LCP = [0, 1, 3, 0, 0, 2]
```

#### Burrows-Wheeler Transform

```
Rotations de "banana$":

banana$ → b a n a n a $
anana$b → a n a n a $ b
nana$ba → n a n a $ b a
ana$ban → a n a $ b a n
na$bana → n a $ b a n a
a$banan → a $ b a n a n
$banana → $ b a n a n a

Triées:
$banana → $ b a n a n a
a$banan → a $ b a n a n
ana$ban → a n a $ b a n
anana$b → a n a n a $ b
banana$ → b a n a n a $
na$bana → n a $ b a n a
nana$ba → n a n a $ b a

BWT = dernière colonne = a n n b $ a a = "annb$aa"
```

### 5.4 Les pièges en détail

#### Piège 1: Le doubling ne s'arrête pas

```rust
// ❌ ERREUR: Pas de condition d'arrêt
while k < n {
    // ... tri et mise à jour ...
    k *= 2;
}

// ✅ CORRECT: Arrêter quand tous les rangs sont distincts
while k < n {
    // ...
    if rank[sa[n - 1]] == n - 1 {
        break; // Tous les suffixes ont des rangs distincts!
    }
    k *= 2;
}
```

#### Piège 2: Kasai sans décrémentation

```rust
// ❌ ERREUR: k ne décrémente jamais
lcp[rank[i]] = k;
// Oublie: k -= 1;

// Résultat: k peut devenir énorme, complexité O(n²)

// ✅ CORRECT
lcp[rank[i]] = k;
if k > 0 {
    k -= 1; // Propriété cruciale de Kasai!
}
```

### 5.5 Cours Complet

#### Pourquoi le doubling fonctionne?

À l'itération k, on compare les suffixes par leurs k premiers caractères. Si deux suffixes ont le même rank après l'itération k, ils partagent les mêmes k premiers caractères.

En doublant k à chaque itération, on atteint k = n en O(log n) itérations. Chaque itération fait un tri O(n log n), donc complexité totale O(n log² n).

#### Pourquoi Kasai est O(n)?

L'astuce: si LCP[rank[i]] = k, alors LCP[rank[i+1]] ≥ k-1.

Intuition: si le suffixe à position i partage k caractères avec son prédécesseur dans SA, alors le suffixe à position i+1 (qui est le même avec le premier caractère enlevé) partage au moins k-1 caractères.

Donc k ne peut diminuer que de n au total, et augmenter de n au total → O(n).

### 5.8 Mnémotechniques

#### 🏜️ MEME: "The Spice must flow" — Le Suffix Array

Comme le Spice qui coule sur Arrakis permettant la prescience, le Suffix Array permet de "voir" tous les suffixes triés.

```rust
// "The Spice must flow" = tous les futurs sont accessibles
let sp = SpicePrescience::awaken(b"muaddib");
// Maintenant tu peux voir TOUS les futurs (suffixes) instantanément
```

#### 🛤️ MEME: "The Golden Path" — LCP Array

Le Golden Path de Leto II est le chemin qui maximise la survie de l'humanité. LCP trouve les "chemins communs" entre suffixes adjacents.

```rust
// LCP = préfixes partagés = destins qui commencent pareil
// max(LCP) = "The Golden Path" = plus long chemin qui se répète
```

#### 🔮 MEME: "Kwisatz Haderach" — Suffix Tree

Le Kwisatz Haderach peut être "en tous lieux" simultanément. Le Suffix Tree avec suffix links permet de "téléporter" instantanément entre les nœuds.

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Détail |
|---------|--------|
| **Structure** | Suffix Array + LCP |
| **Construction** | O(n log² n) ou O(n) DC3 |
| **Pattern Search** | O(m log n) |
| **Distinct Substrings** | n(n+1)/2 - ΣLCP |
| **Longest Repeated** | max(LCP) |
| **BWT** | O(n) encode/decode |
| **Applications** | Bio-info, compression, plagiat |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.2.6-synth-spice-prescience",
    "generated_at": "2026-01-11 18:00:00",

    "metadata": {
      "exercise_id": "1.2.6-synth",
      "exercise_name": "spice_prescience",
      "module": "1.2.19-20",
      "module_name": "Suffix Array & Suffix Tree",
      "type": "complet",
      "tier": 3,
      "phase": 1,
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "language": "rust",
      "duration_minutes": 120,
      "xp_base": 250,
      "xp_bonus_multiplier": 4,
      "bonus_tier": "EXPERT",
      "bonus_icon": "💀",
      "complexity_time": "T6 O(n log² n)",
      "complexity_space": "S4 O(n)",
      "domains": ["Struct", "Algo", "Compression", "Encodage"],
      "tags": ["suffix-array", "lcp", "bwt", "dune"],
      "meme_reference": "The Spice must flow"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "The Spice must flow. And with the Spice... comes prescience."*
*DUNE — La Prescience du Kwisatz Haderach*
*L'excellence pédagogique ne se négocie pas*
