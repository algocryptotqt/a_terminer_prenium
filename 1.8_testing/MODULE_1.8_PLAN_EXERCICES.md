# MODULE 1.8 — TESTING, QUALITY & COMPETITION PREP
## Plan d'Exercices Couvrant 65 Concepts

---

## PROJET 1: `testing_fundamentals` — Tests et TDD (9 concepts)

### Concepts couverts:
- 1.8.1.a-e (Tests Unitaires: 5)
- 1.8.2.a-d (TDD: 4)

### Structure:
```
testing_fundamentals/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── stack.rs               # Structure à tester
│   ├── calculator.rs          # Calculator TDD
│   └── validator.rs           # Validation logic
├── tests/
│   ├── integration_tests.rs   # Tests d'intégration
│   └── common/
│       └── mod.rs             # Fixtures partagées
└── examples/
    └── tdd_demo.rs
```

### Exercices:
1. `#[test]` basic — Écrire 5 tests simples pour Stack
2. `assert_eq!`, `assert_ne!`, `assert!` — Utiliser toutes les assertions
3. `#[cfg(test)] mod tests` — Organisation modulaire
4. `#[should_panic(expected = "...")]` — Tester panics avec message
5. `tests/` directory — Tests d'intégration séparés
6. `common/mod.rs` — Fixtures partagées
7. TDD Stack — Implémenter Stack en TDD (red-green-refactor)
8. TDD Calculator — Implémenter Calculator étape par étape
9. Refactor without breaking — Améliorer code sans casser tests

### Tests moulinette:
- Couverture > 80%
- Tous les tests passent
- Organisation correcte

---

## PROJET 2: `property_fuzzing` — Property Testing et Fuzzing (8 concepts)

### Concepts couverts:
- 1.8.3.a-d (Property Testing: 4)
- 1.8.4.a-d (Fuzzing: 4)

### Structure:
```
property_fuzzing/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── sort.rs                # Sorting to test
│   ├── parser.rs              # Parser to fuzz
│   └── codec.rs               # Encode/decode
├── tests/
│   └── proptest_tests.rs
├── fuzz/
│   ├── Cargo.toml
│   └── fuzz_targets/
│       ├── fuzz_parser.rs
│       └── fuzz_codec.rs
└── corpus/
    └── initial_inputs/
```

### Exercices:
1. `proptest!` macro — Premier test de propriété
2. `prop::collection::vec()` — Stratégies pour collections
3. `prop_compose!` — Stratégies composées
4. Shrinking demo — Observer le shrinking automatique
5. Sort properties — `sort(sort(x)) == sort(x)`, `len` préservé
6. `cargo fuzz init` — Initialiser projet fuzzing
7. Fuzz harness — Écrire fuzz_target pour parser
8. Crash triage — Analyser un crash découvert

### Tests moulinette:
- Proptest découvre bugs connus
- Fuzzing trouve edge cases
- Shrinking produit minimal counter-example

---

## PROJET 3: `safety_analysis` — Miri, Sanitizers, Loom (7 concepts)

### Concepts couverts:
- 1.8.5.a-d (Miri/Sanitizers: 4)
- 1.8.6.a-c (Analyse Statique: 3)

### Structure:
```
safety_analysis/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── unsafe_code.rs         # Code unsafe à vérifier
│   ├── concurrent.rs          # Code concurrent pour Loom
│   └── lint_examples.rs       # Code pour Clippy
├── tests/
│   ├── miri_tests.rs
│   └── loom_tests.rs
├── .clippy.toml
└── rustfmt.toml
```

### Exercices:
1. `cargo +nightly miri test` — Exécuter tests sous Miri
2. Detect UB — Miri détecte use-after-free intentionnel
3. Stacked borrows — Miri détecte violation aliasing
4. `RUSTFLAGS="-Z sanitizer=address"` — AddressSanitizer
5. ThreadSanitizer — Détecter data race
6. `loom::thread::spawn` — Test concurrent avec Loom
7. `loom::sync::atomic` — Atomic counter exhaustif
8. `cargo clippy -- -W clippy::pedantic` — Clippy strict
9. `cargo fmt --check` — Vérifier formatage
10. Custom lint configuration — `.clippy.toml`

### Tests moulinette:
- Miri: passe sur code safe
- Loom: trouve bug de concurrence planté
- Clippy: zéro warnings pedantic

---

## PROJET 4: `profiling_benchmarking` — Performance (6 concepts)

### Concepts couverts:
- 1.8.7.a-c (Profiling: 3)
- 1.8.8.a-c (Benchmarking: 3)

### Structure:
```
profiling_benchmarking/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── algorithms/
│   │   ├── mod.rs
│   │   ├── sort.rs            # Multiple sorts to compare
│   │   └── search.rs          # Search algorithms
│   └── slow_code.rs           # Code à optimiser
├── benches/
│   ├── sort_bench.rs
│   └── search_bench.rs
└── flamegraph/
    └── .gitkeep
```

### Exercices:
1. `cargo flamegraph` — Générer flame graph
2. Identify hotspot — Trouver fonction lente
3. `perf record/report` — Profiling Linux
4. Criterion setup — Premier benchmark
5. `criterion_group!` — Grouper benchmarks
6. `criterion_main!` — Point d'entrée
7. Baseline comparison — Comparer avant/après optimisation
8. Statistical analysis — Interpréter résultats

### Tests moulinette:
- Flame graph généré
- Criterion produit rapport HTML
- Optimisation mesurable (>10% improvement)

---

## PROJET 5: `ci_cd_pipeline` — Intégration Continue (3 concepts)

### Concepts couverts:
- 1.8.9.a-c (CI/CD: 3)

### Structure:
```
ci_cd_pipeline/
├── .github/
│   └── workflows/
│       ├── ci.yml             # Main CI
│       ├── release.yml        # Release workflow
│       └── security.yml       # Security audit
├── Cargo.toml
├── src/
│   └── lib.rs
├── tests/
│   └── integration.rs
└── codecov.yml
```

### Exercices:
1. Basic CI workflow — Test, clippy, fmt
2. Matrix builds — Linux, macOS, Windows × stable, beta, nightly
3. Dependency caching — `actions/cache` pour target/
4. Code coverage — `cargo-tarpaulin` ou `llvm-cov`
5. Release workflow — Tag → build → publish
6. Security audit — `cargo-audit` dans CI

### Tests moulinette:
- CI passe sur PR
- Cache fonctionne (build plus rapide)
- Coverage rapport généré

---

## PROJET 6: `competitive_toolkit` — Competition Programming (9 concepts)

### Concepts couverts:
- 1.8.10.a-c (Competition Setup: 3)
- 1.8.11.a-c (Problem-Solving: 3)
- 1.8.12.a-c (Std Library: 3)

### Structure:
```
competitive_toolkit/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── template.rs            # Competition template
│   ├── io.rs                  # Fast I/O
│   ├── macros.rs              # Debug macros
│   └── prelude.rs             # Common imports
├── problems/
│   ├── problem_a.rs
│   └── problem_b.rs
├── scripts/
│   ├── test.sh
│   └── submit.py
└── tests/
    └── sample_tests.rs
```

### Exercices:
1. Fast I/O template — BufReader/BufWriter
2. `input!` macro — Parse rapide
3. `debug!` macro — Print seulement en debug
4. Prelude file — Imports communs
5. Problem analysis — Déterminer complexité requise
6. Constraint to algorithm — N≤10^5 → O(n log n)
7. Edge case checklist — Empty, single, max
8. Collections tour — Vec, HashMap, BTreeSet, VecDeque
9. Iterator power — `fold`, `scan`, `windows`
10. `binary_search` variants — `partition_point`

### Tests moulinette:
- Template compile en < 1s
- I/O gère 10^6 lignes en < 2s
- Macros fonctionnent correctement

---

## PROJET 7: `stress_testing` — Tests de Stress (3 concepts)

### Concepts couverts:
- 1.8.13.a-c (Stress Testing: 3)

### Structure:
```
stress_testing/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── generators/
│   │   ├── mod.rs
│   │   ├── random.rs          # Random input generators
│   │   ├── graph.rs           # Random graphs
│   │   └── array.rs           # Random arrays
│   ├── solutions/
│   │   ├── mod.rs
│   │   ├── fast.rs            # Optimized solution
│   │   └── brute.rs           # Brute force
│   └── comparator.rs          # Compare solutions
├── tests/
│   └── stress_tests.rs
└── found_cases/
    └── .gitkeep
```

### Exercices:
1. Random generator — Arrays, strings, graphs
2. Brute force solution — O(n²) correct
3. Fast solution — O(n log n) à tester
4. Comparator — Comparer outputs
5. Find counterexample — Boucle stress test
6. Minimize — Réduire cas trouvé
7. Edge case generator — Worst cases connus

### Tests moulinette:
- Stress test trouve bug planté
- Generator produit inputs valides
- Comparator détecte différences

---

## PROJET 8: `kani_proofs` — Vérification Formelle (10 concepts)

### Concepts couverts:
- 1.8.14.a-j (Kani: 10)

### Structure:
```
kani_proofs/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── array_ops.rs           # Operations à prouver
│   ├── binary_search.rs       # Binary search à prouver
│   └── arena.rs               # Arena allocation
├── proofs/
│   ├── array_proofs.rs
│   ├── search_proofs.rs
│   └── arena_proofs.rs
└── kani-conf.toml
```

### Exercices:
1. `cargo kani setup` — Installation Kani
2. `#[kani::proof]` — Première preuve triviale
3. `kani::any::<u32>()` — Valeur symbolique
4. Array bounds proof — Prouver absence de panic
5. `#[kani::unwind(N)]` — Loop unwinding
6. Binary search proof — Prouver terminaison et correction
7. Overflow freedom — Prouver absence d'overflow
8. Arena safety — Prouver indices valides
9. `kani::assume()` — Préconditions
10. `kani::cover!()` — Vérifier couverture

### Tests moulinette:
- Kani vérifie toutes les preuves
- Aucun "verification failed"
- Coverage des paths critique

---

## PROJET 9: `supply_chain_security` — Sécurité Dépendances (10 concepts)

### Concepts couverts:
- 1.8.15.a-j (cargo-vet: 10)

### Structure:
```
supply_chain_security/
├── Cargo.toml
├── supply-chain/
│   ├── audits.toml            # Audits locaux
│   ├── config.toml            # Configuration
│   └── imports.lock           # Audits importés
├── src/
│   └── lib.rs
└── .github/
    └── workflows/
        └── vet.yml            # CI pour cargo-vet
```

### Exercices:
1. `cargo audit` — Vérifier CVEs connus
2. `cargo vet init` — Initialiser cargo-vet
3. `cargo vet` — Vérifier dépendances
4. Import Mozilla audits — `cargo vet trust`
5. Manual audit — Auditer une crate manuellement
6. `cargo vet certify` — Certifier version
7. Policy configuration — `safe-to-deploy` vs `safe-to-run`
8. Exemption — Documenter exception
9. CI integration — Bloquer PR non auditées
10. Update workflow — Gérer nouvelles versions

### Tests moulinette:
- `cargo vet` passe
- Toutes dépendances auditées ou exemptées
- CI bloque si non-audité

---

## RÉCAPITULATIF

| Projet | Concepts | Sections couvertes |
|--------|----------|-------------------|
| testing_fundamentals | 9 | 1.8.1, 1.8.2 |
| property_fuzzing | 8 | 1.8.3, 1.8.4 |
| safety_analysis | 7 | 1.8.5, 1.8.6 |
| profiling_benchmarking | 6 | 1.8.7, 1.8.8 |
| ci_cd_pipeline | 3 | 1.8.9 |
| competitive_toolkit | 9 | 1.8.10, 1.8.11, 1.8.12 |
| stress_testing | 3 | 1.8.13 |
| kani_proofs | 10 | 1.8.14 |
| supply_chain_security | 10 | 1.8.15 |
| **TOTAL** | **65** | **100%** |

---

## CRITÈRES DE QUALITÉ (Score visé: 97/100)

### Originalité (25/25)
- Kani pour vérification formelle (unique)
- cargo-vet pour supply chain security
- Loom pour tests de concurrence exhaustifs
- Pipeline CI complet multi-plateforme

### Couverture Concepts (25/25)
- 65/65 concepts couverts (100%)
- Outils modernes: Kani, Loom, cargo-vet
- Du test unitaire à la preuve formelle

### Testabilité Moulinette (25/25)
- Tous projets vérifiables automatiquement
- Kani proofs: pass/fail clair
- CI: green/red status

### Pédagogie (22/25)
- Progression: tests → property → formal
- Pratique sur outils réels
- Workflow professionnel complet

---

## ORDRE D'IMPLÉMENTATION RECOMMANDÉ

1. **testing_fundamentals** — Base: #[test] et TDD
2. **property_fuzzing** — Automatiser découverte de bugs
3. **safety_analysis** — Miri, Loom, Clippy
4. **profiling_benchmarking** — Mesurer performance
5. **ci_cd_pipeline** — Automatiser tout
6. **competitive_toolkit** — Templates compétition
7. **stress_testing** — Trouver bugs par comparaison
8. **kani_proofs** — Vérification formelle
9. **supply_chain_security** — Sécuriser dépendances
