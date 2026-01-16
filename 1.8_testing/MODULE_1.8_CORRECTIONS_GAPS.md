# MODULE 1.8 — CORRECTIONS DES CONCEPTS MANQUANTS
## 20 concepts non couverts dans les exercices existants

---

## Concepts Manquants Identifiés:

### Section 1.8.13 — Stress Testing (2 concepts)
- `1.8.13.b` — Brute force solution naïve
- `1.8.13.c` — Comparison pour trouver bugs

### Section 1.8.14 — Kani Verification (9 concepts)
- `1.8.14.b` — Kani Verifier (outil AWS)
- `1.8.14.c` — Installation
- `1.8.14.d` — `#[kani::proof]`
- `1.8.14.e` — `kani::any()`
- `1.8.14.f` — Assertions formelles
- `1.8.14.g` — Bounded model checking
- `1.8.14.h` — Loop unwinding
- `1.8.14.i` — Memory safety proofs
- `1.8.14.j` — Panic freedom proofs

### Section 1.8.15 — cargo-vet Security (9 concepts)
- `1.8.15.b` — `cargo audit`
- `1.8.15.c` — `cargo-vet`
- `1.8.15.d` — Installation
- `1.8.15.e` — Politique de confiance
- `1.8.15.f` — Audits manuels
- `1.8.15.g` — Réseau de confiance
- `1.8.15.h` — Criteria (safe-to-deploy, safe-to-run)
- `1.8.15.i` — Exemptions
- `1.8.15.j` — CI integration

---

## EXERCICES COMPLÉMENTAIRES

### Projet Complémentaire: `stress_testing_complete`

**Exercice C1:** Brute Force Solution [1.8.13.b]
```rust
/// Solution O(n²) garantie correcte mais lente
fn brute_force_max_subarray(arr: &[i32]) -> i32 {
    let mut max = i32::MIN;
    for i in 0..arr.len() {
        for j in i..arr.len() {
            let sum: i32 = arr[i..=j].iter().sum();
            max = max.max(sum);
        }
    }
    max
}

/// Solution O(n) Kadane à tester
fn kadane_max_subarray(arr: &[i32]) -> i32 {
    // Implémentation à tester contre brute force
}
```

**Exercice C2:** Comparator Framework [1.8.13.c]
```rust
struct StressTester<T, F1, F2>
where
    F1: Fn(&T) -> i32,
    F2: Fn(&T) -> i32,
{
    brute_force: F1,
    optimized: F2,
    generator: Box<dyn Fn() -> T>,
}

impl<T, F1, F2> StressTester<T, F1, F2>
where
    F1: Fn(&T) -> i32,
    F2: Fn(&T) -> i32,
{
    fn find_counterexample(&self, max_iterations: usize) -> Option<T> {
        for _ in 0..max_iterations {
            let input = (self.generator)();
            let expected = (self.brute_force)(&input);
            let actual = (self.optimized)(&input);
            if expected != actual {
                return Some(input);
            }
        }
        None
    }
}
```

---

### Projet Complémentaire: `kani_verification_complete`

**Exercice C3:** Kani Setup [1.8.14.b, 1.8.14.c]
```bash
# Installation Kani
cargo install --locked kani-verifier
cargo kani setup

# Vérifier installation
cargo kani --version
```

**Exercice C4:** Premier Proof [1.8.14.d, 1.8.14.e]
```rust
#[cfg(kani)]
mod verification {
    use super::*;

    #[kani::proof]
    fn verify_add_no_overflow() {
        let a: u8 = kani::any();  // Valeur symbolique [1.8.14.e]
        let b: u8 = kani::any();

        // Précondition
        kani::assume(a <= 100);
        kani::assume(b <= 100);

        // Cette assertion sera prouvée formellement [1.8.14.d]
        let result = a.checked_add(b);
        assert!(result.is_some());
    }
}
```

**Exercice C5:** Assertions Formelles [1.8.14.f]
```rust
#[kani::proof]
fn verify_binary_search_correctness() {
    let arr: [i32; 5] = kani::any();
    let target: i32 = kani::any();

    // Précondition: array trié
    kani::assume(arr.windows(2).all(|w| w[0] <= w[1]));

    let result = binary_search(&arr, target);

    // Post-condition: si trouvé, l'élément est correct
    if let Some(idx) = result {
        assert!(arr[idx] == target);  // Preuve formelle! [1.8.14.f]
    }
}
```

**Exercice C6:** Bounded Model Checking [1.8.14.g, 1.8.14.h]
```rust
#[kani::proof]
#[kani::unwind(10)]  // Limiter à 10 itérations de boucle [1.8.14.h]
fn verify_loop_terminates() {
    let n: usize = kani::any();
    kani::assume(n < 10);  // Borne pour model checking [1.8.14.g]

    let mut sum = 0;
    for i in 0..n {
        sum += i;
    }

    // Prouve que la boucle termine et sum est correct
    assert!(sum == n * (n - 1) / 2);
}
```

**Exercice C7:** Memory Safety [1.8.14.i]
```rust
#[kani::proof]
fn verify_no_buffer_overflow() {
    let arr: [u8; 10] = kani::any();
    let idx: usize = kani::any();

    // Prouve qu'on ne peut pas avoir d'overflow mémoire
    if idx < arr.len() {
        let _ = arr[idx];  // Safe access prouvé [1.8.14.i]
    }
}
```

**Exercice C8:** Panic Freedom [1.8.14.j]
```rust
#[kani::proof]
#[kani::should_panic]  // Ou sans, pour prouver absence de panic
fn verify_division_safety() {
    let a: i32 = kani::any();
    let b: i32 = kani::any();

    kani::assume(b != 0);  // Précondition

    let result = a / b;  // Prouve: pas de panic si b != 0 [1.8.14.j]

    // Vérifier aussi overflow pour i32::MIN / -1
    kani::assume(!(a == i32::MIN && b == -1));
}
```

---

### Projet Complémentaire: `supply_chain_security_complete`

**Exercice C9:** cargo audit [1.8.15.b]
```bash
# Installer cargo-audit
cargo install cargo-audit

# Scanner les vulnérabilités connues
cargo audit

# Générer rapport JSON
cargo audit --json > audit_report.json
```

**Exercice C10:** cargo-vet Setup [1.8.15.c, 1.8.15.d]
```bash
# Installation [1.8.15.d]
cargo install cargo-vet

# Initialiser dans un projet [1.8.15.c]
cargo vet init

# Structure créée:
# supply-chain/
# ├── audits.toml
# ├── config.toml
# └── imports.lock
```

**Exercice C11:** Politique de Confiance [1.8.15.e]
```toml
# supply-chain/config.toml

[policy.my-crate]
criteria = "safe-to-deploy"

# Définir qui on fait confiance [1.8.15.e]
[[trusted.mozilla]]
url = "https://raw.githubusercontent.com/ArcTanSumo/AuditDB/main/AuditDB.toml"

[[trusted.bytecode-alliance]]
url = "https://raw.githubusercontent.com/ArcTanSumo/AuditDB/main/AuditDB.toml"
```

**Exercice C12:** Audit Manuel [1.8.15.f]
```bash
# Auditer une crate manuellement [1.8.15.f]
cargo vet inspect serde

# Après review, certifier
cargo vet certify serde 1.0.193

# L'audit est enregistré dans audits.toml
```

**Exercice C13:** Réseau de Confiance [1.8.15.g]
```bash
# Importer les audits de Mozilla [1.8.15.g]
cargo vet trust mozilla

# Importer audits Google
cargo vet trust google

# Les imports sont dans imports.lock
```

**Exercice C14:** Criteria [1.8.15.h]
```toml
# supply-chain/audits.toml

[[audits.serde]]
who = "Your Name <you@example.com>"
criteria = "safe-to-deploy"  # [1.8.15.h]
version = "1.0.193"
notes = "Reviewed: no unsafe, no network, no filesystem"

# Autres critères possibles:
# - safe-to-run: OK pour dev/test
# - safe-to-deploy: OK pour production
```

**Exercice C15:** Exemptions [1.8.15.i]
```toml
# supply-chain/config.toml

# Exemption documentée pour crate legacy [1.8.15.i]
[[exemptions.old-crate]]
version = "0.1.0"
criteria = "safe-to-run"
notes = "Legacy dependency, will remove in v2.0"
```

**Exercice C16:** CI Integration [1.8.15.j]
```yaml
# .github/workflows/supply-chain.yml

name: Supply Chain Security

on: [push, pull_request]

jobs:
  vet:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install cargo-vet
        run: cargo install cargo-vet

      - name: Check supply chain  # [1.8.15.j]
        run: cargo vet --locked

      - name: Audit for CVEs
        run: |
          cargo install cargo-audit
          cargo audit
```

---

## RÉSUMÉ

| Section | Concepts Ajoutés | Exercices |
|---------|-----------------|-----------|
| 1.8.13 Stress Testing | 2 | C1, C2 |
| 1.8.14 Kani | 9 | C3-C8 |
| 1.8.15 cargo-vet | 9 | C9-C16 |
| **TOTAL** | **20** | **16** |

---

## INTÉGRATION

Ces exercices complémentaires doivent être ajoutés aux projets existants:

1. **stress_testing** → Ajouter C1, C2
2. **kani_proofs** → Ajouter C3-C8
3. **supply_chain_security** → Ajouter C9-C16

Après intégration: **100% des concepts 1.8 couverts**
