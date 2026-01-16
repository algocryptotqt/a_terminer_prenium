<thinking>
## Analyse du Concept
- Concept : Property-Based Testing (générateurs, shrinking, QuickCheck-style)
- Phase demandée : 1 (intermédiaire)
- Adapté ? OUI - Le property-based testing est une technique avancée mais fondamentale pour les tests de qualité

## Combo Base + Bonus
- Exercice de base : Implémenter des générateurs (Int, Vec, String), runner de propriétés, shrinking basique
- Bonus : Shrinking intelligent (binary search), model-based testing, génération de graphes
- Palier bonus : 🔥 Avancé (techniques sophistiquées de test)
- Progression logique ? OUI - Base = générateurs simples, Bonus = techniques avancées

## Prérequis & Difficulté
- Prérequis réels : Traits Rust, génériques, closures, tests unitaires
- Difficulté estimée : 6/10 (base), 8/10 (bonus)
- Cohérent avec phase ? OUI - Phase 1 intermédiaire

## Aspect Fun/Culture
- Contexte choisi : "House M.D." - Le diagnosticien qui teste toutes les hypothèses
- MEME mnémotechnique : "It's never lupus... until it is" = "The test passes... until you find the counterexample"
- Pourquoi c'est fun : Parallèle parfait entre diagnostic médical et debugging par propriétés

## Scénarios d'Échec (5 mutants)
1. Mutant A (Boundary) : Générateur Int qui ne respecte pas min/max
2. Mutant B (Safety) : Shrinking infini (pas de terminaison)
3. Mutant C (Logic) : Propriété mal évaluée (inversée)
4. Mutant D (Generator) : VecGen qui génère toujours des vecteurs vides
5. Mutant E (Return) : TestResult toujours "Passed" même quand échoué

## Verdict
VALIDE - Exercice complet couvrant property-based testing
</thinking>

# Exercice 1.8.1 : house_md_property_testing

**Module :**
1.8.1 — Property-Based Testing

**Concept :**
d — Générateurs, shrinking, tests basés sur les propriétés (QuickCheck-style)

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
2 — Mélange (générateurs + shrinking + propriétés + runner)

**Langage :**
Rust Edition 2024 + C (c17)

**Prérequis :**
- Tests unitaires (1.8.0)
- Traits et génériques Rust
- Closures et higher-order functions
- Itérateurs

**Domaines :**
Algo, Struct, Probas

**Durée estimée :**
75 min

**XP Base :**
150

**Complexité :**
T4 O(n × tests) × S3 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- `property_testing.rs` (Rust)
- `property_testing.c` + `property_testing.h` (C)

**Fonctions autorisées :**
- Rust : `std::*`, `rand` crate
- C : `malloc`, `free`, `rand`, `srand`, `time`

**Fonctions interdites :**
- Frameworks de property testing externes (proptest, quickcheck)

### 1.2 Consigne

**🎮 HOUSE M.D. — "Everybody Lies... Especially Code"**

*"It's not lupus. It's never lupus."* — Dr. House

Tu es le Dr. House du débugging. Dans le département de Diagnostic Différentiel, tu ne fais pas confiance aux tests unitaires simples. Tu préfères bombarder le code avec des MILLIERS de cas générés aléatoirement pour trouver le diagnostic (bug) que personne d'autre ne voit.

Comme House qui ne croit jamais le patient ("Everybody lies!"), tu ne crois jamais que le code fonctionne avant d'avoir testé des propriétés UNIVERSELLES.

**Ta mission :**

Créer un framework de property-based testing complet :

1. **`diagnostic_generator`** (Trait Generator) : L'interface pour générer des cas de test
2. **`symptoms_int`** (IntGen) : Générateur d'entiers dans une plage
3. **`symptoms_vec`** (VecGen) : Générateur de vecteurs
4. **`symptoms_string`** (StringGen) : Générateur de chaînes
5. **`differential_diagnosis`** (PropTest) : Le runner qui teste les propriétés
6. **`treatment_shrink`** : Réduire un contre-exemple au minimum

**Entrée :**
- `Generator<T>` : Trait avec `generate()` et `shrink()`
- `property: Fn(T) -> bool` : La propriété à tester
- `num_tests: usize` : Nombre de tests à exécuter
- `seed: Option<u64>` : Graine pour reproductibilité

**Sortie :**
- `TestResult::Passed { num_tests }` : Tous les tests passent
- `TestResult::Failed { counterexample, shrunk_to }` : Contre-exemple trouvé et réduit

**Contraintes :**
- Les générateurs doivent respecter les bornes spécifiées
- Le shrinking doit converger (pas de boucle infinie)
- Les contre-exemples doivent être MINIMAUX après shrinking
- Le runner doit pouvoir être déterministe avec une seed

**Exemples :**

| Test | Propriété | Résultat |
|------|-----------|----------|
| `for_all(IntGen{0,100}, \|n\| n >= 0)` | Non-négativité | `Passed(100)` |
| `for_all(IntGen{0,1000}, \|n\| n < 500)` | n < 500 | `Failed{ce: 500, shrunk: 500}` |
| `for_all(VecGen, \|v\| v.sort(); is_sorted(&v))` | Tri → trié | `Passed(100)` |
| `for_all(VecGen, \|v\| v.len() < 10)` | Longueur < 10 | `Failed{shrunk: [0;10]}` |

### 1.2.2 Consigne Académique

Implémenter un framework de property-based testing inspiré de QuickCheck :
- **Générateurs** : Interface pour produire des valeurs aléatoires typées
- **Shrinking** : Réduction systématique des contre-exemples vers des cas minimaux
- **Runner** : Exécution de tests avec détection et rapport des échecs
- **Propriétés** : Fonctions booléennes devant être vraies pour toutes les entrées

### 1.3 Prototype

```rust
// Rust Edition 2024
use rand::Rng;

/// Trait pour les générateurs de valeurs
pub trait Generator<T> {
    /// Génère une valeur aléatoire
    fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> T;

    /// Produit des valeurs plus simples (pour shrinking)
    fn shrink(&self, value: T) -> Box<dyn Iterator<Item = T>>;
}

/// Générateur d'entiers (Symptoms Int)
pub struct IntGen {
    pub min: i64,
    pub max: i64,
}

impl Generator<i64> for IntGen {
    fn generate<R: Rng>(&self, rng: &mut R, _size: usize) -> i64;
    fn shrink(&self, value: i64) -> Box<dyn Iterator<Item = i64>>;
}

/// Générateur de vecteurs (Symptoms Vec)
pub struct VecGen<G> {
    pub element_gen: G,
    pub max_len: usize,
}

impl<T: Clone, G: Generator<T>> Generator<Vec<T>> for VecGen<G> {
    fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> Vec<T>;
    fn shrink(&self, value: Vec<T>) -> Box<dyn Iterator<Item = Vec<T>>>;
}

/// Générateur de chaînes (Symptoms String)
pub struct StringGen {
    pub charset: String,
    pub max_len: usize,
}

impl Generator<String> for StringGen {
    fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> String;
    fn shrink(&self, value: String) -> Box<dyn Iterator<Item = String>>;
}

/// Résultat de test
#[derive(Debug, Clone)]
pub enum TestResult {
    Passed { num_tests: usize },
    Failed { counterexample: String, shrunk_to: String },
    GaveUp { reason: String },
}

/// Runner de tests (Differential Diagnosis)
pub struct PropTest {
    num_tests: usize,
    max_shrinks: usize,
    seed: Option<u64>,
}

impl PropTest {
    pub fn new() -> Self;
    pub fn num_tests(self, n: usize) -> Self;
    pub fn max_shrinks(self, n: usize) -> Self;
    pub fn seed(self, s: u64) -> Self;

    /// Teste une propriété pour tous les cas générés
    pub fn for_all<T, G, F>(self, gen: G, prop: F) -> TestResult
    where
        G: Generator<T>,
        T: std::fmt::Debug + Clone,
        F: Fn(T) -> bool;
}

/// Utilitaires de shrinking
pub mod shrinking {
    /// Shrink un entier vers zéro
    pub fn shrink_int(n: i64) -> impl Iterator<Item = i64>;

    /// Shrink un vecteur en retirant des éléments
    pub fn shrink_vec<T: Clone>(v: Vec<T>) -> impl Iterator<Item = Vec<T>>;

    /// Shrink une chaîne
    pub fn shrink_string(s: String) -> impl Iterator<Item = String>;
}

/// Propriétés communes (Common Symptoms)
pub mod properties {
    pub fn is_sorted<T: Ord>(arr: &[T]) -> bool;
    pub fn is_permutation<T: Ord + Clone>(a: &[T], b: &[T]) -> bool;
    pub fn is_idempotent<T: Eq + Clone, F: Fn(T) -> T>(f: F, x: T) -> bool;
    pub fn is_commutative<T: Eq + Clone, F: Fn(T, T) -> T>(f: F, a: T, b: T) -> bool;
    pub fn is_associative<T: Eq + Clone, F: Fn(T, T) -> T>(f: F, a: T, b: T, c: T) -> bool;
}
```

```c
// C17
#ifndef PROPERTY_TESTING_H
#define PROPERTY_TESTING_H

#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>

// Générateur d'entiers
typedef struct {
    int64_t min;
    int64_t max;
} IntGen;

// Générateur de vecteurs d'entiers
typedef struct {
    IntGen element_gen;
    size_t max_len;
} VecIntGen;

// Résultat de test
typedef enum {
    TEST_PASSED,
    TEST_FAILED,
    TEST_GAVE_UP
} TestStatus;

typedef struct {
    TestStatus status;
    size_t num_tests;
    char *counterexample;
    char *shrunk_to;
} TestResult;

// Configuration du runner
typedef struct {
    size_t num_tests;
    size_t max_shrinks;
    uint64_t seed;
    bool use_seed;
} PropTestConfig;

// Génération
int64_t int_gen_generate(IntGen *gen, size_t size);
int64_t *vec_int_gen_generate(VecIntGen *gen, size_t *out_len, size_t size);

// Shrinking
typedef struct IntIterator IntIterator;
IntIterator *shrink_int(int64_t value);
int64_t int_iterator_next(IntIterator *it, bool *has_next);
void int_iterator_free(IntIterator *it);

// Property testing
typedef bool (*IntProperty)(int64_t);
typedef bool (*VecIntProperty)(int64_t *, size_t);

TestResult for_all_int(PropTestConfig *config, IntGen *gen, IntProperty prop);
TestResult for_all_vec_int(PropTestConfig *config, VecIntGen *gen, VecIntProperty prop);

// Propriétés communes
bool is_sorted_int(int64_t *arr, size_t len);
bool is_permutation_int(int64_t *a, size_t len_a, int64_t *b, size_t len_b);

// Cleanup
void test_result_free(TestResult *result);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Origine de QuickCheck

QuickCheck a été inventé en 1999 par Koen Claessen et John Hughes à Chalmers (Suède). C'est un des outils qui a le plus influencé les tests logiciels modernes.

### 2.2 Le Pouvoir du Shrinking

Le shrinking est crucial : sans lui, un contre-exemple pourrait être un vecteur de 1000 éléments alors que le bug se manifeste avec seulement 2 éléments. Comme House réduit les symptômes à leur essence !

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation | Exemple Concret |
|--------|-------------|-----------------|
| **Développeur Blockchain** | Fuzzing de smart contracts | Trouver des edge cases dans les transactions |
| **Ingénieur Sécurité** | Fuzzing de parsers | Trouver des vulnérabilités avec entrées random |
| **Data Engineer** | Tests de pipelines | Vérifier propriétés sur données générées |
| **DevOps** | Tests de configuration | Tester toutes les combinaisons de config |
| **Game Dev** | Tests de physique | Vérifier que les lois physiques tiennent |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
property_testing.rs  property_testing.c  property_testing.h  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
test int_generator_bounds ... ok
test vec_generator_length ... ok
test shrink_int_converges ... ok
test property_sorted ... ok
test property_counterexample ... ok
All 5 tests passed!

$ cargo run
Testing sort is_sorted property... PASSED (1000 tests)
Testing n < 500 property... FAILED
  Counterexample: 723
  Shrunk to: 500
Testing reverse involution... PASSED (1000 tests)
All property tests completed!
```

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(log n) pour binary shrinking

**Space Complexity attendue :**
O(n) pour model-based testing

**Domaines Bonus :**
`Algo, Probas, Struct`

#### 3.1.1 Consigne Bonus

**🎮 HOUSE M.D. — "Le Diagnostic Différentiel Avancé"**

House ne se contente pas de tests basiques. Il utilise des techniques avancées :

1. **`house_binary_shrink`** : Shrinking par recherche binaire (O(log n))
2. **`cuddy_model_test`** : Model-based testing (comparer implémentation vs modèle)
3. **`wilson_graph_gen`** : Génération de graphes aléatoires

**Contraintes Bonus :**
```
┌─────────────────────────────────────────┐
│  Binary Shrink :                        │
│  - Réduire en O(log n) au lieu de O(n)  │
│  - Dichotomie sur l'espace des valeurs  │
│                                         │
│  Model-Based Testing :                  │
│  - Définir un modèle (spec simple)      │
│  - Comparer SUT vs modèle               │
│  - Détecter divergences                 │
│                                         │
│  Graph Generator :                      │
│  - Générer graphes avec propriétés      │
│  - Connecté, acyclique, etc.            │
└─────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
pub mod bonus {
    /// Shrinking par recherche binaire
    pub fn house_binary_shrink<T, F>(
        value: T,
        predicate: F,
        shrinker: impl Fn(T, T) -> T, // midpoint
    ) -> T
    where
        F: Fn(&T) -> bool;

    /// Model-based testing
    pub trait Model<S, A, R> {
        fn initial_state(&self) -> S;
        fn transition(&self, state: &S, action: &A) -> (S, R);
    }

    pub fn cuddy_model_test<S, A, R, M, I>(
        model: M,
        implementation: I,
        action_gen: impl Generator<A>,
        num_steps: usize,
    ) -> TestResult
    where
        M: Model<S, A, R>,
        I: Fn(&A) -> R,
        R: Eq + std::fmt::Debug;

    /// Génération de graphes
    pub struct GraphGen {
        pub max_nodes: usize,
        pub max_edges: usize,
        pub connected: bool,
        pub acyclic: bool,
    }

    impl Generator<Vec<Vec<usize>>> for GraphGen {
        fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> Vec<Vec<usize>>;
        fn shrink(&self, value: Vec<Vec<usize>>) -> Box<dyn Iterator<Item = Vec<Vec<usize>>>>;
    }
}
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Input | Expected | Points |
|------|-------|----------|--------|
| `int_gen_in_bounds` | `IntGen{0,100}` | Tous les résultats dans [0,100] | 5 |
| `int_gen_distribution` | 1000 générations | Distribution approximativement uniforme | 3 |
| `vec_gen_length` | `VecGen{max_len:20}` | Longueurs dans [0,20] | 5 |
| `vec_gen_elements` | `VecGen{IntGen{0,10}}` | Éléments dans [0,10] | 5 |
| `string_gen_charset` | `StringGen{charset:"abc"}` | Que des 'a', 'b', 'c' | 5 |
| `shrink_int_zero` | `shrink_int(100)` | Contient 0 | 5 |
| `shrink_int_half` | `shrink_int(100)` | Contient 50 | 5 |
| `shrink_vec_smaller` | `shrink_vec([1,2,3,4])` | Contient des vecs plus courts | 5 |
| `prop_always_true` | `for_all(IntGen, \|_\| true)` | `Passed(100)` | 7 |
| `prop_find_counter` | `for_all(IntGen{0,1000}, \|n\| n<500)` | `Failed{shrunk:500}` | 10 |
| `prop_sort_sorted` | Test tri → trié | `Passed` | 7 |
| `prop_reverse_involution` | reverse(reverse(x)) == x | `Passed` | 7 |
| `shrink_minimal` | Contre-exemple réduit au min | Valeur minimale violant propriété | 10 |
| `is_sorted_true` | `[1,2,3,4,5]` | `true` | 3 |
| `is_sorted_false` | `[1,3,2,4,5]` | `false` | 3 |
| `is_permutation` | Vec trié vs original | `true` | 5 |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <time.h>
#include "property_testing.h"

bool prop_non_negative(int64_t n) {
    return n >= 0;
}

bool prop_less_than_500(int64_t n) {
    return n < 500;
}

bool prop_vec_small(int64_t *arr, size_t len) {
    return len < 10;
}

int main(void) {
    srand(time(NULL));

    // Test IntGen bounds
    printf("Testing IntGen bounds...\n");
    IntGen gen = {0, 100};
    for (int i = 0; i < 1000; i++) {
        int64_t val = int_gen_generate(&gen, 10);
        assert(val >= 0 && val <= 100);
    }
    printf("  PASS: All values in [0, 100]\n");

    // Test property that should pass
    printf("\nTesting non-negative property...\n");
    PropTestConfig config = {100, 100, 0, false};
    IntGen gen_pos = {0, 1000};
    TestResult result = for_all_int(&config, &gen_pos, prop_non_negative);
    assert(result.status == TEST_PASSED);
    printf("  PASS: %zu tests passed\n", result.num_tests);

    // Test property that should fail
    printf("\nTesting n < 500 property...\n");
    IntGen gen_wide = {0, 1000};
    result = for_all_int(&config, &gen_wide, prop_less_than_500);
    assert(result.status == TEST_FAILED);
    printf("  EXPECTED FAIL: counterexample=%s, shrunk=%s\n",
           result.counterexample, result.shrunk_to);
    test_result_free(&result);

    // Test shrinking
    printf("\nTesting shrink_int...\n");
    IntIterator *it = shrink_int(100);
    bool has_zero = false;
    bool has_fifty = false;
    bool has_next;
    while (1) {
        int64_t val = int_iterator_next(it, &has_next);
        if (!has_next) break;
        if (val == 0) has_zero = true;
        if (val == 50) has_fifty = true;
    }
    int_iterator_free(it);
    assert(has_zero);
    assert(has_fifty);
    printf("  PASS: Shrinking produces 0 and 50\n");

    // Test is_sorted
    printf("\nTesting is_sorted...\n");
    int64_t sorted[] = {1, 2, 3, 4, 5};
    int64_t unsorted[] = {1, 3, 2, 4, 5};
    assert(is_sorted_int(sorted, 5) == true);
    assert(is_sorted_int(unsorted, 5) == false);
    printf("  PASS: is_sorted works correctly\n");

    printf("\n=== All property testing tests passed! ===\n");
    return 0;
}
```

### 4.3 Solution de référence

```rust
use rand::Rng;
use rand::rngs::StdRng;
use rand::SeedableRng;

/// Trait Generator
pub trait Generator<T> {
    fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> T;
    fn shrink(&self, value: T) -> Box<dyn Iterator<Item = T>>;
}

/// IntGen - Générateur d'entiers
pub struct IntGen {
    pub min: i64,
    pub max: i64,
}

impl Generator<i64> for IntGen {
    fn generate<R: Rng>(&self, rng: &mut R, _size: usize) -> i64 {
        rng.gen_range(self.min..=self.max)
    }

    fn shrink(&self, value: i64) -> Box<dyn Iterator<Item = i64>> {
        let min = self.min;
        let max = self.max;
        Box::new(ShrinkInt::new(value, min, max))
    }
}

struct ShrinkInt {
    value: i64,
    min: i64,
    candidates: Vec<i64>,
    index: usize,
}

impl ShrinkInt {
    fn new(value: i64, min: i64, max: i64) -> Self {
        let mut candidates = Vec::new();

        // Target: shrink towards min (or 0 if in range)
        let target = if min <= 0 && max >= 0 { 0 } else { min };

        if value != target {
            candidates.push(target);

            // Binary shrinking: add midpoints
            let mut current = value;
            while (current - target).abs() > 1 {
                current = (current + target) / 2;
                if current != target && current >= min && current <= max {
                    candidates.push(current);
                }
            }

            // Decrement by 1
            if value > target && value - 1 >= min {
                candidates.push(value - 1);
            }
            if value < target && value + 1 <= max {
                candidates.push(value + 1);
            }
        }

        Self { value, min, candidates, index: 0 }
    }
}

impl Iterator for ShrinkInt {
    type Item = i64;

    fn next(&mut self) -> Option<Self::Item> {
        if self.index < self.candidates.len() {
            let result = self.candidates[self.index];
            self.index += 1;
            Some(result)
        } else {
            None
        }
    }
}

/// VecGen - Générateur de vecteurs
pub struct VecGen<G> {
    pub element_gen: G,
    pub max_len: usize,
}

impl<T: Clone + 'static, G: Generator<T> + Clone + 'static> Generator<Vec<T>> for VecGen<G> {
    fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> Vec<T> {
        let len = rng.gen_range(0..=self.max_len.min(size));
        (0..len).map(|_| self.element_gen.generate(rng, size)).collect()
    }

    fn shrink(&self, value: Vec<T>) -> Box<dyn Iterator<Item = Vec<T>>> {
        Box::new(ShrinkVec::new(value))
    }
}

struct ShrinkVec<T> {
    original: Vec<T>,
    index: usize,
    candidates: Vec<Vec<T>>,
}

impl<T: Clone> ShrinkVec<T> {
    fn new(original: Vec<T>) -> Self {
        let mut candidates = Vec::new();

        // Empty vector
        if !original.is_empty() {
            candidates.push(Vec::new());
        }

        // Remove each element one by one
        for i in 0..original.len() {
            let mut smaller = original.clone();
            smaller.remove(i);
            candidates.push(smaller);
        }

        // Remove first half, second half
        if original.len() >= 2 {
            candidates.push(original[original.len()/2..].to_vec());
            candidates.push(original[..original.len()/2].to_vec());
        }

        Self { original, index: 0, candidates }
    }
}

impl<T: Clone> Iterator for ShrinkVec<T> {
    type Item = Vec<T>;

    fn next(&mut self) -> Option<Self::Item> {
        if self.index < self.candidates.len() {
            let result = self.candidates[self.index].clone();
            self.index += 1;
            Some(result)
        } else {
            None
        }
    }
}

/// StringGen - Générateur de chaînes
pub struct StringGen {
    pub charset: String,
    pub max_len: usize,
}

impl Generator<String> for StringGen {
    fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> String {
        let len = rng.gen_range(0..=self.max_len.min(size));
        let chars: Vec<char> = self.charset.chars().collect();
        (0..len)
            .map(|_| chars[rng.gen_range(0..chars.len())])
            .collect()
    }

    fn shrink(&self, value: String) -> Box<dyn Iterator<Item = String>> {
        Box::new(ShrinkString::new(value))
    }
}

struct ShrinkString {
    candidates: Vec<String>,
    index: usize,
}

impl ShrinkString {
    fn new(original: String) -> Self {
        let mut candidates = Vec::new();

        if !original.is_empty() {
            candidates.push(String::new());
        }

        // Remove each character
        for i in 0..original.len() {
            let mut smaller = original.clone();
            smaller.remove(i);
            candidates.push(smaller);
        }

        Self { candidates, index: 0 }
    }
}

impl Iterator for ShrinkString {
    type Item = String;

    fn next(&mut self) -> Option<Self::Item> {
        if self.index < self.candidates.len() {
            let result = self.candidates[self.index].clone();
            self.index += 1;
            Some(result)
        } else {
            None
        }
    }
}

/// TestResult
#[derive(Debug, Clone)]
pub enum TestResult {
    Passed { num_tests: usize },
    Failed { counterexample: String, shrunk_to: String },
    GaveUp { reason: String },
}

/// PropTest - Runner
pub struct PropTest {
    num_tests: usize,
    max_shrinks: usize,
    seed: Option<u64>,
}

impl PropTest {
    pub fn new() -> Self {
        Self {
            num_tests: 100,
            max_shrinks: 100,
            seed: None,
        }
    }

    pub fn num_tests(mut self, n: usize) -> Self {
        self.num_tests = n;
        self
    }

    pub fn max_shrinks(mut self, n: usize) -> Self {
        self.max_shrinks = n;
        self
    }

    pub fn seed(mut self, s: u64) -> Self {
        self.seed = Some(s);
        self
    }

    pub fn for_all<T, G, F>(self, gen: G, prop: F) -> TestResult
    where
        G: Generator<T>,
        T: std::fmt::Debug + Clone,
        F: Fn(T) -> bool,
    {
        let mut rng: Box<dyn rand::RngCore> = match self.seed {
            Some(s) => Box::new(StdRng::seed_from_u64(s)),
            None => Box::new(rand::thread_rng()),
        };

        for test_num in 0..self.num_tests {
            let size = (test_num as f64 / self.num_tests as f64 * 100.0) as usize;
            let value = gen.generate(&mut *rng, size.max(1));

            if !prop(value.clone()) {
                // Found counterexample, now shrink
                let counterexample = format!("{:?}", value);
                let shrunk = self.shrink_value(&gen, value, &prop);
                let shrunk_to = format!("{:?}", shrunk);

                return TestResult::Failed { counterexample, shrunk_to };
            }
        }

        TestResult::Passed { num_tests: self.num_tests }
    }

    fn shrink_value<T, G, F>(&self, gen: &G, value: T, prop: &F) -> T
    where
        G: Generator<T>,
        T: Clone,
        F: Fn(T) -> bool,
    {
        let mut current = value;
        let mut shrinks = 0;

        while shrinks < self.max_shrinks {
            let mut found_smaller = false;

            for smaller in gen.shrink(current.clone()) {
                if !prop(smaller.clone()) {
                    current = smaller;
                    found_smaller = true;
                    shrinks += 1;
                    break;
                }
            }

            if !found_smaller {
                break;
            }
        }

        current
    }
}

impl Default for PropTest {
    fn default() -> Self {
        Self::new()
    }
}

/// Propriétés communes
pub mod properties {
    pub fn is_sorted<T: Ord>(arr: &[T]) -> bool {
        arr.windows(2).all(|w| w[0] <= w[1])
    }

    pub fn is_permutation<T: Ord + Clone>(a: &[T], b: &[T]) -> bool {
        if a.len() != b.len() {
            return false;
        }
        let mut a_sorted = a.to_vec();
        let mut b_sorted = b.to_vec();
        a_sorted.sort();
        b_sorted.sort();
        a_sorted == b_sorted
    }

    pub fn is_idempotent<T: Eq + Clone, F: Fn(T) -> T>(f: F, x: T) -> bool {
        let once = f(x.clone());
        let twice = f(once.clone());
        once == twice
    }

    pub fn is_commutative<T: Eq + Clone, F: Fn(T, T) -> T>(f: F, a: T, b: T) -> bool {
        f(a.clone(), b.clone()) == f(b, a)
    }

    pub fn is_associative<T: Eq + Clone, F: Fn(T, T) -> T>(f: &F, a: T, b: T, c: T) -> bool {
        f(f(a.clone(), b.clone()), c.clone()) == f(a, f(b, c))
    }
}

/// Module de shrinking
pub mod shrinking {
    pub fn shrink_int(n: i64) -> impl Iterator<Item = i64> {
        let mut candidates = Vec::new();
        if n != 0 {
            candidates.push(0);
            let mut current = n;
            while current.abs() > 1 {
                current /= 2;
                if current != 0 {
                    candidates.push(current);
                }
            }
            if n > 0 {
                candidates.push(n - 1);
            } else {
                candidates.push(n + 1);
            }
        }
        candidates.into_iter()
    }

    pub fn shrink_vec<T: Clone>(v: Vec<T>) -> impl Iterator<Item = Vec<T>> {
        let mut candidates = Vec::new();
        if !v.is_empty() {
            candidates.push(Vec::new());
            for i in 0..v.len() {
                let mut smaller = v.clone();
                smaller.remove(i);
                candidates.push(smaller);
            }
        }
        candidates.into_iter()
    }

    pub fn shrink_string(s: String) -> impl Iterator<Item = String> {
        let mut candidates = Vec::new();
        if !s.is_empty() {
            candidates.push(String::new());
            for i in 0..s.len() {
                let mut smaller = s.clone();
                smaller.remove(i);
                candidates.push(smaller);
            }
        }
        candidates.into_iter()
    }
}
```

### 4.4 Solutions alternatives acceptées

```rust
// Alternative 1: Shrinking avec liste liée au lieu de Vec
pub fn shrink_int_lazy(n: i64) -> impl Iterator<Item = i64> {
    std::iter::successors(Some(n / 2), move |&x| {
        if x == 0 { None } else { Some(x / 2) }
    }).chain(std::iter::once(0))
}

// Alternative 2: VecGen avec taille proportionnelle au "size"
impl<T: Clone, G: Generator<T>> Generator<Vec<T>> for VecGen<G> {
    fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> Vec<T> {
        // Taille proportionnelle au paramètre size
        let max = (size as f64 * self.max_len as f64 / 100.0).ceil() as usize;
        let len = rng.gen_range(0..=max);
        (0..len).map(|_| self.element_gen.generate(rng, size)).collect()
    }
    // ...
}
```

### 4.5 Solutions refusées

```rust
// REFUSÉ 1: Générateur hors bornes
impl Generator<i64> for IntGen {
    fn generate<R: Rng>(&self, rng: &mut R, _size: usize) -> i64 {
        rng.gen() // BUG: ignore min/max!
    }
}
// Pourquoi : Ne respecte pas les contraintes du générateur

// REFUSÉ 2: Shrinking infini
fn shrink_int_infinite(n: i64) -> impl Iterator<Item = i64> {
    std::iter::repeat(n - 1) // BUG: boucle infinie sur la même valeur!
}
// Pourquoi : Ne converge jamais, cause stack overflow

// REFUSÉ 3: PropTest qui ignore les échecs
impl PropTest {
    pub fn for_all<T, G, F>(self, gen: G, prop: F) -> TestResult {
        // BUG: retourne toujours Passed!
        TestResult::Passed { num_tests: self.num_tests }
    }
}
// Pourquoi : Ne détecte jamais les bugs
```

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "house_md_property_testing",
  "language": "rust",
  "type": "code",
  "tier": 2,
  "tier_info": "Mélange (générateurs + shrinking + propriétés + runner)",
  "tags": ["property-testing", "generators", "shrinking", "quickcheck", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "PropTest::for_all",
    "prototype": "pub fn for_all<T, G, F>(self, gen: G, prop: F) -> TestResult",
    "return_type": "TestResult",
    "parameters": [
      {"name": "gen", "type": "G: Generator<T>"},
      {"name": "prop", "type": "F: Fn(T) -> bool"}
    ]
  },

  "driver": {
    "reference": "impl PropTest { pub fn ref_for_all<T: std::fmt::Debug + Clone, G: Generator<T>, F: Fn(T) -> bool>(self, gen: G, prop: F) -> TestResult { let mut rng = rand::thread_rng(); for _ in 0..self.num_tests { let value = gen.generate(&mut rng, 100); if !prop(value.clone()) { return TestResult::Failed { counterexample: format!(\"{:?}\", value), shrunk_to: format!(\"{:?}\", value) }; } } TestResult::Passed { num_tests: self.num_tests } } }",

    "edge_cases": [
      {
        "name": "always_true_property",
        "description": "Property that always returns true",
        "expected": "TestResult::Passed",
        "is_trap": false
      },
      {
        "name": "always_false_property",
        "description": "Property that always returns false",
        "expected": "TestResult::Failed",
        "is_trap": true,
        "trap_explanation": "Doit échouer immédiatement avec contre-exemple"
      },
      {
        "name": "boundary_property",
        "description": "Property n < 500 with IntGen{0,1000}",
        "expected": "TestResult::Failed{shrunk_to: 500}",
        "is_trap": true,
        "trap_explanation": "Doit shrink au minimum violant: 500"
      },
      {
        "name": "empty_vec_gen",
        "description": "VecGen with max_len=0",
        "expected": "Only generates empty vectors",
        "is_trap": false
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {
            "min": -1000000,
            "max": 1000000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["std::*", "rand::*"],
    "forbidden_functions": ["proptest", "quickcheck"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```rust
/* Mutant A (Boundary) : IntGen hors bornes */
impl Generator<i64> for IntGen {
    fn generate<R: Rng>(&self, rng: &mut R, _size: usize) -> i64 {
        rng.gen_range(self.min..self.max) // BUG: exclusive max, peut être < min
    }
}
// Pourquoi c'est faux : Ne génère jamais self.max
// Ce qui était pensé : "..max est comme ..=max"

/* Mutant B (Safety) : Shrinking infini */
impl Generator<i64> for IntGen {
    fn shrink(&self, value: i64) -> Box<dyn Iterator<Item = i64>> {
        Box::new(std::iter::repeat(value)) // BUG: infini!
    }
}
// Pourquoi c'est faux : Ne termine jamais, stack overflow

/* Mutant C (Logic) : Propriété inversée */
pub fn for_all<T, G, F>(self, gen: G, prop: F) -> TestResult
where F: Fn(T) -> bool
{
    // BUG: teste !prop au lieu de prop
    if !prop(gen.generate(&mut rng, 100)) {
        TestResult::Passed { num_tests: 1 } // Inversé!
    }
}
// Pourquoi c'est faux : Considère échec comme succès

/* Mutant D (Generator) : VecGen toujours vide */
impl<T, G: Generator<T>> Generator<Vec<T>> for VecGen<G> {
    fn generate<R: Rng>(&self, _rng: &mut R, _size: usize) -> Vec<T> {
        Vec::new() // BUG: toujours vide
    }
}
// Pourquoi c'est faux : Ne teste jamais de vecteurs non vides

/* Mutant E (Return) : Toujours Passed */
pub fn for_all<T, G, F>(self, gen: G, prop: F) -> TestResult {
    // BUG: ignore complètement le test
    TestResult::Passed { num_tests: self.num_tests }
}
// Pourquoi c'est faux : Ne détecte aucun bug
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Concept fondamental** : Property-based testing vs unit testing
2. **Générateurs** : Produire des données de test aléatoires typées
3. **Shrinking** : Réduire les contre-exemples au minimum
4. **Propriétés** : Définir des invariants universels

### 5.2 LDA — Traduction Littérale

```
FONCTION for_all QUI RETOURNE UN RÉSULTAT DE TEST ET PREND EN PARAMÈTRES gen QUI EST UN GÉNÉRATEUR DE TYPE T ET prop QUI EST UNE FONCTION DE T VERS BOOLÉEN
DÉBUT FONCTION
    DÉCLARER rng COMME GÉNÉRATEUR DE NOMBRES ALÉATOIRES

    POUR test_num ALLANT DE 0 À num_tests MOINS 1 FAIRE
        DÉCLARER size COMME test_num DIVISÉ PAR num_tests MULTIPLIÉ PAR 100
        DÉCLARER value COMME LE RÉSULTAT DE gen.generate AVEC rng ET size

        SI NON prop APPLIQUÉE À value ALORS
            DÉCLARER counterexample COMME LA REPRÉSENTATION TEXTUELLE DE value
            DÉCLARER shrunk COMME LE RÉSULTAT DE shrink_value AVEC gen, value, prop
            RETOURNER TestResult::Failed AVEC counterexample ET shrunk
        FIN SI
    FIN POUR

    RETOURNER TestResult::Passed AVEC num_tests
FIN FONCTION
```

### 5.2.2 Logic Flow

```
ALGORITHME : Property-Based Test Runner (Differential Diagnosis)
---
1. INITIALISER :
   - rng = générateur aléatoire (avec seed si fournie)
   - test_count = 0

2. POUR chaque test de 0 à num_tests :
   a. CALCULER size = (test / num_tests) × 100
      (augmente progressivement pour trouver des bugs sur grandes entrées)

   b. GÉNÉRER value = gen.generate(rng, size)

   c. TESTER prop(value) :
      |
      |-- SI prop retourne FAUX :
      |     SHRINK value jusqu'au minimum
      |     RETOURNER Failed{counterexample, shrunk}
      |
      |-- SINON : Continuer

3. SI tous les tests passent :
   RETOURNER Passed{num_tests}
```

### 5.2.3 Diagramme Mermaid

```mermaid
graph TD
    A[Début: for_all] --> B[Init RNG]
    B --> C[Pour test 0..N]
    C --> D[Générer value]
    D --> E{prop(value)?}

    E -- Oui --> C
    E -- Non --> F[Shrink value]
    F --> G[Trouver minimum]
    G --> H[RETOUR: Failed]

    C -- Terminé --> I[RETOUR: Passed]

    subgraph Shrinking
        F --> J[Pour chaque shrunk]
        J --> K{prop(shrunk)?}
        K -- Non --> L[current = shrunk]
        L --> J
        K -- Oui --> J
        J -- Épuisé --> G
    end
```

### 5.3 Visualisation ASCII

```
PROPERTY-BASED TESTING FLOW:
============================

Test: for_all(IntGen{0,1000}, |n| n < 500)

Génération:
  Test 1: n = 234 → prop(234) = true ✓
  Test 2: n = 891 → prop(891) = false ✗

Contre-exemple trouvé: 891

Shrinking de 891:
┌─────┬───────────┬────────────┬───────────┐
│ #   │ Candidat  │ prop(x)?   │ Action    │
├─────┼───────────┼────────────┼───────────┤
│ 1   │ 0         │ true       │ Skip      │
│ 2   │ 445       │ true       │ Skip      │
│ 3   │ 668       │ false      │ → current │
├─────┼───────────┼────────────┼───────────┤
│ 4   │ 0         │ true       │ Skip      │
│ 5   │ 334       │ true       │ Skip      │
│ 6   │ 501       │ false      │ → current │
├─────┼───────────┼────────────┼───────────┤
│ 7   │ 0         │ true       │ Skip      │
│ 8   │ 250       │ true       │ Skip      │
│ 9   │ 500       │ false      │ → current │
├─────┼───────────┼────────────┼───────────┤
│ 10  │ 0         │ true       │ Skip      │
│ 11  │ 250       │ true       │ Skip      │
│ 12  │ 499       │ true       │ Skip      │
└─────┴───────────┴────────────┴───────────┘

Résultat final: shrunk_to = 500 (minimum violant la propriété!)


GÉNÉRATEUR INTERNE:
==================

IntGen{min: 0, max: 1000}
│
├── generate(rng, size=50)
│   └── rng.gen_range(0..=1000) → 723
│
└── shrink(723)
    ├── 0        (target)
    ├── 361      (723 / 2)
    ├── 180      (361 / 2)
    ├── 90       (180 / 2)
    ├── 45       (90 / 2)
    └── 722      (723 - 1)
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| Bornes non inclusives | `gen_range(a..b)` exclut b | Utiliser `a..=b` |
| Shrinking infini | Itérateur sans fin | Ajouter compteur max_shrinks |
| Seed non déterministe | Tests non reproductibles | Toujours supporter seed option |
| Size constant | Rate les bugs sur grandes entrées | Augmenter size progressivement |
| Pas de shrink | Contre-exemples énormes | Implémenter shrink pour chaque type |

### 5.5 Cours Complet

#### 5.5.1 Property-Based vs Unit Testing

| Unit Testing | Property-Based Testing |
|--------------|----------------------|
| Cas spécifiques | Cas générés aléatoirement |
| `assert_eq!(sort([3,1,2]), [1,2,3])` | `assert!(is_sorted(sort(any_vec)))` |
| Teste exemples | Teste PROPRIÉTÉS |
| Facile à écrire | Trouve plus de bugs |

#### 5.5.2 Les Propriétés Fondamentales

1. **Idempotence** : `f(f(x)) == f(x)`
   - Exemple : `sort(sort(v)) == sort(v)`

2. **Round-trip** : `decode(encode(x)) == x`
   - Exemple : JSON parse/stringify

3. **Invariant** : Propriété toujours vraie après opération
   - Exemple : `is_sorted(sort(v))` toujours vrai

4. **Commutatif** : `f(a, b) == f(b, a)`
   - Exemple : `a + b == b + a`

5. **Oracle/Modèle** : Compare avec implémentation simple
   - Exemple : Quick sort vs bubble sort

#### 5.5.3 L'Art du Shrinking

Le shrinking transforme un contre-exemple complexe en cas minimal :

```
Contre-exemple initial: [847, 123, 999, 42, 501, 0, 333]
Après shrinking:        [1, 0]

Le bug était: "échoue quand un élément est plus grand que le suivant"
```

**Stratégies de shrinking** :
1. Vers zéro (pour entiers)
2. Sous-ensembles (pour collections)
3. Préfixes/suffixes (pour chaînes)
4. Dichotomie (pour trouver rapidement)

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ impl Generator<i64> for IntGen {                                │
│     fn generate(&self, rng: &mut impl Rng) -> i64 { ... }       │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ impl Generator<i64> for IntGen {                                │
│     fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> i64│
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ • Le paramètre `size` permet de progresser vers de plus grands │
│   tests                                                         │
│ • Le generic `R: Rng` permet d'injecter n'importe quel RNG     │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

```
Trace: PropTest::new().num_tests(5).for_all(IntGen{0,100}, |n| n < 50)

┌───────┬──────────────────────────────┬───────┬──────────┬─────────────────┐
│ Test  │ Action                       │ value │ prop(v)  │ Résultat        │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│   1   │ generate(size=0)             │  23   │ true     │ Continue        │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│   2   │ generate(size=20)            │  41   │ true     │ Continue        │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│   3   │ generate(size=40)            │  67   │ false    │ COUNTEREXAMPLE! │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│  S1   │ shrink(67) → try 0           │   0   │ true     │ Skip            │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│  S2   │ shrink(67) → try 33          │  33   │ true     │ Skip            │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│  S3   │ shrink(67) → try 50          │  50   │ false    │ → current = 50  │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│  S4   │ shrink(50) → try 0           │   0   │ true     │ Skip            │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│  S5   │ shrink(50) → try 25          │  25   │ true     │ Skip            │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│  S6   │ shrink(50) → try 49          │  49   │ true     │ Skip            │
├───────┼──────────────────────────────┼───────┼──────────┼─────────────────┤
│  S7   │ No more shrinks              │  —    │ —        │ Stop            │
└───────┴──────────────────────────────┴───────┴──────────┴─────────────────┘

Résultat: Failed { counterexample: "67", shrunk_to: "50" }
```

### 5.8 Mnémotechniques

#### 🏥 MEME : "It's never lupus" — Property Testing

Comme Dr. House qui teste TOUTES les hypothèses avant de conclure, le property-based testing teste des MILLIERS de cas avant de dire "ça marche".

```rust
// House: "Le patient dit que ça marche... everybody lies!"
fn house_test<T>(code: impl Fn(T) -> bool) -> TestResult {
    // Ne jamais croire le code. Tester 10000 cas.
    PropTest::new().num_tests(10000).for_all(any_gen(), code)
}
```

#### 🔬 MEME : "Differential Diagnosis" — Shrinking

House ne donne pas juste le diagnostic, il trouve le MINIMUM de symptômes qui expliquent tout. C'est exactement ce que fait le shrinking !

```
Symptômes initiaux: [fièvre, toux, fatigue, maux de tête, nausée]
Après diagnostic différentiel: [fièvre, toux]  // Minimum suffisant!
```

#### 💊 MEME : "Run the test again" — Seeds

Quand House trouve un cas bizarre, il veut pouvoir le REPRODUIRE exactement. C'est pourquoi on utilise des seeds :

```rust
// Pour reproduire: PropTest::new().seed(12345)
```

### 5.9 Applications pratiques

| Domaine | Utilisation | Exemple |
|---------|-------------|---------|
| **Compilateurs** | Tester transformations | `compile(source) == compile(optimize(source))` |
| **Sérialisation** | Round-trip | `deserialize(serialize(x)) == x` |
| **Crypto** | Propriétés mathématiques | `decrypt(encrypt(msg, key), key) == msg` |
| **Databases** | ACID properties | Transactions concurrentes |
| **Parsers** | Fuzzing | Entrées aléatoires ne crashent pas |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

1. **Bornes de générateur** : Utiliser `..=` pour inclure max
2. **Shrinking infini** : Limiter avec `max_shrinks`
3. **Non-déterminisme** : Toujours supporter seed pour reproduire
4. **Size statique** : Augmenter progressivement pour trouver bugs sur grandes entrées
5. **Propriétés triviales** : `|_| true` passe toujours, ne teste rien!

---

## 📝 SECTION 7 : QCM

### Q1. Avantage Principal
Quel est l'avantage principal du property-based testing?

A) Plus rapide que les unit tests
B) Trouve des edge cases non anticipés
C) Plus facile à écrire
D) Meilleure couverture de code garantie
E) Pas besoin de définir les attendus
F) Tests plus lisibles
G) Compatible avec tous les langages
H) Génère automatiquement la documentation
I) Remplace complètement les unit tests
J) Nécessite moins de maintenance

**Réponse : B**

### Q2. Shrinking Purpose
À quoi sert le shrinking?

A) Accélérer les tests
B) Réduire la mémoire utilisée
C) Trouver le contre-exemple minimal
D) Générer plus de cas de test
E) Compresser les résultats
F) Optimiser le code testé
G) Paralléliser les tests
H) Réduire le temps de compilation
I) Minimiser le code de test
J) Simplifier les propriétés

**Réponse : C**

### Q3. Good Property
Quelle propriété est bien formulée pour tester une fonction de tri?

A) `sort(v).len() > 0`
B) `sort(v) != v`
C) `is_sorted(sort(v)) && is_permutation(v, sort(v))`
D) `sort(v) == sort(sort(v))`
E) `sort(v).first() < sort(v).last()`
F) `sort(v).len() == v.len()`
G) `sort([]) == []`
H) `sort(v) ne contient pas de doublons`
I) `sort est plus rapide que v`
J) `sort retourne un nouveau vecteur`

**Réponse : C**

### Q4. Seed Purpose
Pourquoi utiliser une seed dans PropTest?

A) Pour accélérer les tests
B) Pour améliorer l'aléatoire
C) Pour reproduire exactement un test
D) Pour générer plus de cas
E) Pour le debugging seulement
F) Pour la sécurité
G) Pour la parallélisation
H) Obligatoire pour fonctionner
I) Pour le shrinking uniquement
J) Pour les tests de performance

**Réponse : C**

### Q5. Generator Trait
Que doit implémenter un Generator custom?

A) `generate` uniquement
B) `shrink` uniquement
C) `generate` et `shrink`
D) `generate`, `shrink` et `validate`
E) `new` et `generate`
F) Seulement `Clone`
G) `Arbitrary` trait
H) `Into<T>` trait
I) `Default` et `generate`
J) `Random` trait

**Réponse : C**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Concepts couverts** | Generators, Shrinking, Properties, Runner |
| **Types de générateurs** | IntGen, VecGen, StringGen |
| **Propriétés standards** | is_sorted, is_permutation, idempotent, commutative |
| **Shrinking** | Converge vers minimum |
| **Difficulté base** | 6/10 |
| **Difficulté bonus** | 8/10 |
| **XP possible** | 150 (base) + 450 (bonus) = 600 |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.8.1-house-md-property-testing",
    "generated_at": "2026-01-12T02:45:00Z",

    "metadata": {
      "exercise_id": "1.8.1",
      "exercise_name": "house_md_property_testing",
      "module": "1.8.1",
      "module_name": "Property-Based Testing",
      "concept": "d",
      "concept_name": "Générateurs, shrinking, QuickCheck-style",
      "type": "complet",
      "tier": 2,
      "tier_info": "Mélange (générateurs + shrinking + propriétés + runner)",
      "phase": 1,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust",
      "duration_minutes": 75,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "ADVANCED",
      "bonus_icon": "🔥",
      "complexity_time": "T4 O(n × tests)",
      "complexity_space": "S3 O(n)",
      "prerequisites": ["unit_testing", "traits", "closures", "iterators"],
      "domains": ["Algo", "Struct", "Probas"],
      "domains_bonus": ["Algo", "Probas", "Struct"],
      "tags": ["property-testing", "generators", "shrinking", "quickcheck"],
      "meme_reference": "House M.D."
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 */",
      "alternatives/alt_lazy_shrink.rs": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_infinite.rs": "/* Section 4.10 */",
      "mutants/mutant_c_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_d_generator.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "alternatives/alt_lazy_shrink.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_infinite.rs",
        "mutants/mutant_c_logic.rs",
        "mutants/mutant_d_generator.rs",
        "mutants/mutant_e_return.rs"
      ]
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "House M.D. Property Testing"*
*"Everybody lies... especially code. Test the properties!"*
