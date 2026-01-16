# MODULE 1.8 - ADDENDUM
## Exercices supplémentaires pour couverture complète

Ces exercices complètent MODULE_1.8_EXERCICES_COMPLETS.md pour atteindre 100% de couverture.

---

## Exercice ADD-1: `stress_testing`
**Couvre: 1.8.13.a-c (3 concepts)**

### Concepts
- [1.8.13.a] Random generation — Inputs aléatoires
- [1.8.13.b] Brute force — Solution naïve de référence
- [1.8.13.c] Comparison — Trouver bugs par comparaison

### Rust
```rust
use rand::Rng;

// ============================================================
// 1.8.13.a - Random generation
// ============================================================

/// Générateur d'inputs aléatoires pour stress testing
pub struct RandomInputGenerator {
    rng: rand::rngs::ThreadRng,
}

impl RandomInputGenerator {
    pub fn new() -> Self {
        Self { rng: rand::thread_rng() }
    }

    /// Génère un vecteur aléatoire
    pub fn random_vec(&mut self, len: usize, min: i32, max: i32) -> Vec<i32> {
        (0..len).map(|_| self.rng.gen_range(min..=max)).collect()
    }

    /// Génère une string aléatoire
    pub fn random_string(&mut self, len: usize) -> String {
        (0..len)
            .map(|_| self.rng.gen_range(b'a'..=b'z') as char)
            .collect()
    }

    /// Génère un graphe aléatoire (liste d'adjacence)
    pub fn random_graph(&mut self, nodes: usize, edges: usize) -> Vec<Vec<usize>> {
        let mut adj = vec![Vec::new(); nodes];
        for _ in 0..edges {
            let from = self.rng.gen_range(0..nodes);
            let to = self.rng.gen_range(0..nodes);
            if from != to {
                adj[from].push(to);
            }
        }
        adj
    }
}

// ============================================================
// 1.8.13.b - Brute force reference solution
// ============================================================

/// Solution naïve O(n²) pour trouver deux nombres qui somment à target
pub fn two_sum_brute(nums: &[i32], target: i32) -> Option<(usize, usize)> {
    for i in 0..nums.len() {
        for j in (i + 1)..nums.len() {
            if nums[i] + nums[j] == target {
                return Some((i, j));
            }
        }
    }
    None
}

/// Solution optimisée O(n) avec HashMap
pub fn two_sum_optimized(nums: &[i32], target: i32) -> Option<(usize, usize)> {
    use std::collections::HashMap;
    let mut seen: HashMap<i32, usize> = HashMap::new();
    for (i, &num) in nums.iter().enumerate() {
        let complement = target - num;
        if let Some(&j) = seen.get(&complement) {
            return Some((j, i));
        }
        seen.insert(num, i);
    }
    None
}

// ============================================================
// 1.8.13.c - Comparison (stress testing)
// ============================================================

/// Compare deux implémentations sur des inputs aléatoires
pub fn stress_test_two_sum(iterations: usize) -> Result<(), String> {
    let mut gen = RandomInputGenerator::new();

    for i in 0..iterations {
        // Générer input aléatoire
        let nums = gen.random_vec(100, -1000, 1000);
        let target = gen.rng.gen_range(-2000..2000);

        // Exécuter les deux solutions
        let brute_result = two_sum_brute(&nums, target);
        let opt_result = two_sum_optimized(&nums, target);

        // Comparer les résultats
        match (brute_result, opt_result) {
            (Some((i1, j1)), Some((i2, j2))) => {
                // Vérifier que les deux trouvent une solution valide
                if nums[i1] + nums[j1] != target {
                    return Err(format!("Brute force bug at iter {}", i));
                }
                if nums[i2] + nums[j2] != target {
                    return Err(format!("Optimized bug at iter {}", i));
                }
            }
            (None, None) => {
                // Les deux n'ont pas trouvé - OK
            }
            (Some(_), None) => {
                return Err(format!("Optimized missed solution at iter {}", i));
            }
            (None, Some(_)) => {
                return Err(format!("Brute force missed solution at iter {}", i));
            }
        }
    }

    Ok(())
}

/// Macro pour stress testing générique
#[macro_export]
macro_rules! stress_test {
    ($name:expr, $gen:expr, $reference:expr, $optimized:expr, $iters:expr) => {{
        for i in 0..$iters {
            let input = $gen();
            let ref_result = $reference(&input);
            let opt_result = $optimized(&input);
            if ref_result != opt_result {
                panic!(
                    "{} failed at iteration {}: ref={:?}, opt={:?}, input={:?}",
                    $name, i, ref_result, opt_result, input
                );
            }
        }
        println!("{}: {} iterations passed", $name, $iters);
    }};
}
```

### Test Moulinette
```
stress random_vec 10 0 100 -> [valid 10 integers in 0..100]
stress two_sum brute [1,2,3,4] 5 -> Some((0,3)) or Some((1,2))
stress compare two_sum 1000 -> Ok(())
```

---

## Exercice ADD-2: `formal_verification`
**Couvre: 1.8.14.a-j (10 concepts)**

### Concepts
- [1.8.14.a] Concept — Prouver l'absence de bugs
- [1.8.14.b] Kani Verifier — Outil AWS pour Rust
- [1.8.14.c] Installation — cargo install kani-verifier
- [1.8.14.d] `#[kani::proof]` — Attribut de preuve
- [1.8.14.e] `kani::any()` — Valeurs symboliques
- [1.8.14.f] Assertions — assert! devient preuve formelle
- [1.8.14.g] Bounded model checking — Exploration exhaustive bornée
- [1.8.14.h] Loop unwinding — `#[kani::unwind(N)]`
- [1.8.14.i] Memory safety — Prouver absence de débordements
- [1.8.14.j] Panic freedom — Prouver qu'aucun panic n'est possible

### Rust
```rust
// ============================================================
// 1.8.14.a-c - Concept et installation de Kani
// ============================================================

// Kani est un "bounded model checker" pour Rust
// Il prouve MATHÉMATIQUEMENT que le code est correct
// pour TOUTES les entrées possibles (dans les bornes définies)

// Installation:
// $ cargo install --locked kani-verifier
// $ cargo kani setup

// ============================================================
// 1.8.14.d-f - Attributs et valeurs symboliques
// ============================================================

/// Fonction à vérifier
pub fn safe_divide(a: i32, b: i32) -> Option<i32> {
    if b == 0 {
        None
    } else {
        // Attention: overflow possible avec i32::MIN / -1
        if a == i32::MIN && b == -1 {
            None
        } else {
            Some(a / b)
        }
    }
}

// Preuve Kani (dans un fichier séparé ou avec cfg)
#[cfg(kani)]
mod proofs {
    use super::*;

    // 1.8.14.d - #[kani::proof]
    #[kani::proof]
    fn verify_safe_divide_no_panic() {
        // 1.8.14.e - kani::any() génère TOUTES les valeurs possibles
        let a: i32 = kani::any();
        let b: i32 = kani::any();

        // 1.8.14.f - Cette assertion est prouvée pour TOUTES entrées
        let result = safe_divide(a, b);

        // Si b != 0 et pas le cas overflow, on doit avoir Some
        if b != 0 && !(a == i32::MIN && b == -1) {
            assert!(result.is_some());
        }
    }

    // 1.8.14.i - Prouver l'absence de débordements
    #[kani::proof]
    fn verify_no_overflow() {
        let a: i32 = kani::any();
        let b: i32 = kani::any();

        // safe_divide ne doit JAMAIS overflow
        let _ = safe_divide(a, b);
        // Si on arrive ici sans panic, la preuve est réussie
    }
}

// ============================================================
// 1.8.14.g-h - Bounded model checking et loop unwinding
// ============================================================

/// Recherche linéaire à vérifier
pub fn linear_search(arr: &[i32], target: i32) -> Option<usize> {
    for (i, &val) in arr.iter().enumerate() {
        if val == target {
            return Some(i);
        }
    }
    None
}

#[cfg(kani)]
mod loop_proofs {
    use super::*;

    // 1.8.14.h - Loop unwinding pour boucles
    #[kani::proof]
    #[kani::unwind(11)]  // Dérouler jusqu'à 10 itérations
    fn verify_linear_search() {
        // 1.8.14.g - Exploration exhaustive bornée
        let arr: [i32; 10] = kani::any();
        let target: i32 = kani::any();

        let result = linear_search(&arr, target);

        // Si on trouve un index, il doit être valide
        if let Some(idx) = result {
            assert!(idx < arr.len());
            assert!(arr[idx] == target);
        }

        // Si on ne trouve pas, target n'est vraiment pas là
        if result.is_none() {
            for &val in &arr {
                assert!(val != target);
            }
        }
    }
}

// ============================================================
// 1.8.14.j - Panic freedom
// ============================================================

/// Safe array access
pub fn safe_get(arr: &[i32], idx: usize) -> Option<i32> {
    arr.get(idx).copied()
}

#[cfg(kani)]
mod panic_freedom {
    use super::*;

    // Prouver qu'aucun panic n'est possible
    #[kani::proof]
    #[kani::unwind(101)]
    fn verify_safe_get_never_panics() {
        let arr: [i32; 100] = kani::any();
        let idx: usize = kani::any();

        // Cette fonction ne doit JAMAIS panic
        let _ = safe_get(&arr, idx);

        // Kani vérifie automatiquement:
        // - Pas de bounds check failure
        // - Pas d'overflow arithmétique
        // - Pas d'unwrap() sur None
        // - Pas de division par zéro
    }
}

// ============================================================
// Documentation pour exécution
// ============================================================

/// Pour exécuter les preuves Kani:
/// ```bash
/// # Vérifier toutes les preuves
/// cargo kani
///
/// # Vérifier une preuve spécifique
/// cargo kani --harness verify_safe_divide_no_panic
///
/// # Avec plus de détails
/// cargo kani --harness verify_linear_search --visualize
/// ```
pub fn kani_usage_guide() {
    println!("Kani Verifier - Formal Verification for Rust");
    println!("=============================================");
    println!("1. Install: cargo install --locked kani-verifier");
    println!("2. Setup:   cargo kani setup");
    println!("3. Run:     cargo kani");
    println!("");
    println!("Key concepts:");
    println!("- #[kani::proof] marks a proof harness");
    println!("- kani::any() generates ALL possible values");
    println!("- #[kani::unwind(N)] limits loop iterations");
    println!("- Assertions become mathematical proofs");
}
```

### Test Moulinette
```
kani concept -> "bounded model checking, proves absence of bugs"
kani safe_divide i32::MIN -1 -> None
kani safe_divide 10 2 -> Some(5)
kani linear_search [1,2,3,4,5] 3 -> Some(2)
```

---

## Exercice ADD-3: `supply_chain_security`
**Couvre: 1.8.15.a-j (10 concepts)**

### Concepts
- [1.8.15.a] Problème — Supply chain attacks
- [1.8.15.b] `cargo audit` — Vérifie CVEs connus
- [1.8.15.c] `cargo-vet` — Audit proactif
- [1.8.15.d] Installation — cargo install cargo-vet
- [1.8.15.e] Politique de confiance — Trust configuration
- [1.8.15.f] Audits manuels — Code review
- [1.8.15.g] Réseau de confiance — Import audits
- [1.8.15.h] Criteria — safe-to-deploy, safe-to-run
- [1.8.15.i] Exemptions — Exceptions documentées
- [1.8.15.j] CI integration — Bloquer dépendances non auditées

### Rust
```rust
// ============================================================
// 1.8.15.a - Le problème des supply chain attacks
// ============================================================

/// Supply Chain Security
///
/// Le problème:
/// - Tu dépends de 100+ crates
/// - Un mainteneur se fait hacker
/// - Son compte publie une version malveillante
/// - `cargo update` l'installe automatiquement
/// - Ton code est compromis
///
/// Exemples réels:
/// - event-stream (npm, 2018) - vol de bitcoins
/// - ua-parser-js (npm, 2021) - cryptominers
/// - colors/faker (npm, 2022) - sabotage volontaire

// ============================================================
// 1.8.15.b - cargo audit (réactif)
// ============================================================

/// cargo audit vérifie les vulnérabilités CONNUES
///
/// ```bash
/// # Installation
/// cargo install cargo-audit
///
/// # Utilisation
/// cargo audit
///
/// # Exemple de sortie:
/// # Crate:     smallvec
/// # Version:   0.6.9
/// # Warning:   RUSTSEC-2019-0009
/// # Title:     Double-free and use-after-free in SmallVec
/// # Solution:  Upgrade to >=0.6.10
/// ```
///
/// Limitations:
/// - Seulement les CVEs PUBLIÉS
/// - Une attaque 0-day ne sera pas détectée
/// - Réactif, pas proactif

// ============================================================
// 1.8.15.c-d - cargo-vet (proactif)
// ============================================================

/// cargo-vet pour audit PROACTIF des dépendances
///
/// ```bash
/// # 1.8.15.d - Installation
/// cargo install cargo-vet
///
/// # Initialiser dans le projet
/// cargo vet init
///
/// # Vérifier le status
/// cargo vet
/// ```

// ============================================================
// 1.8.15.e - Politique de confiance
// ============================================================

/// Configuration de confiance dans supply-chain/config.toml
///
/// ```toml
/// # Fichier: supply-chain/config.toml
///
/// [imports.mozilla]
/// url = "https://raw.githubusercontent.com/AuditsDB/mozilla/main/audits.toml"
///
/// [imports.google]
/// url = "https://raw.githubusercontent.com/AuditsDB/google/main/audits.toml"
///
/// [policy.my-crate]
/// criteria = "safe-to-deploy"
///
/// [policy.dev-tool]
/// criteria = "safe-to-run"  # Moins strict pour outils de dev
/// ```

// ============================================================
// 1.8.15.f - Audits manuels
// ============================================================

/// Effectuer un audit manuel
///
/// ```bash
/// # Auditer une crate spécifique
/// cargo vet inspect serde 1.0.150
///
/// # Après review, certifier
/// cargo vet certify serde 1.0.150
/// ```
///
/// Checklist d'audit:
/// - [ ] Pas de code réseau inattendu
/// - [ ] Pas d'accès filesystem suspect
/// - [ ] Pas de macros proc-macro obscures
/// - [ ] Build.rs ne télécharge rien
/// - [ ] Pas d'unsafe injustifié

// ============================================================
// 1.8.15.g - Réseau de confiance
// ============================================================

/// Importer des audits de sources de confiance
///
/// ```bash
/// # Importer audits Mozilla
/// cargo vet import mozilla
///
/// # Fichier généré: supply-chain/imports.lock
/// ```
///
/// Organisations qui publient des audits:
/// - Mozilla (Firefox)
/// - Google (Chromium)
/// - Microsoft
/// - Bytecode Alliance (Wasmtime)

// ============================================================
// 1.8.15.h - Criteria (niveaux de certification)
// ============================================================

/// Niveaux de certification cargo-vet
///
/// ```toml
/// # safe-to-run: OK pour outils de dev, tests, CI
/// # - Pas de vulnérabilités connues
/// # - Fait ce qu'il dit faire
///
/// # safe-to-deploy: OK pour production
/// # - Tout de safe-to-run PLUS:
/// # - Code audité pour sécurité
/// # - Pas de comportement malveillant possible
///
/// # Définir des critères custom
/// [criteria.crypto-safe]
/// description = "Safe for cryptographic use"
/// implies = ["safe-to-deploy"]
/// ```

// ============================================================
// 1.8.15.i - Exemptions
// ============================================================

/// Gérer les exceptions temporaires
///
/// ```toml
/// # Fichier: supply-chain/config.toml
///
/// # Exemption temporaire avec justification
/// [[exemptions.problematic-crate]]
/// version = "1.0.0"
/// criteria = "safe-to-deploy"
/// notes = "Audit en cours, PR #123 pour migration"
///
/// # Exemption pour crate non auditable
/// [[exemptions.sys-crate]]
/// version = "*"
/// criteria = "safe-to-run"
/// notes = "FFI bindings, cannot be fully audited"
/// ```

// ============================================================
// 1.8.15.j - CI Integration
// ============================================================

/// Intégration GitHub Actions
///
/// ```yaml
/// # .github/workflows/supply-chain.yml
/// name: Supply Chain Security
///
/// on:
///   push:
///     branches: [main]
///   pull_request:
///   schedule:
///     - cron: '0 0 * * *'  # Daily check
///
/// jobs:
///   audit:
///     runs-on: ubuntu-latest
///     steps:
///       - uses: actions/checkout@v3
///
///       - name: Install cargo-audit
///         run: cargo install cargo-audit
///
///       - name: Check for vulnerabilities
///         run: cargo audit
///
///       - name: Install cargo-vet
///         run: cargo install cargo-vet
///
///       - name: Verify supply chain
///         run: cargo vet --locked
/// ```

/// Fonction helper pour vérifier la sécurité des dépendances
pub fn check_supply_chain() -> Result<(), Vec<String>> {
    use std::process::Command;
    let mut issues = Vec::new();

    // Vérifier cargo audit
    let audit = Command::new("cargo")
        .args(["audit", "--json"])
        .output();

    if let Ok(output) = audit {
        if !output.status.success() {
            issues.push("cargo audit found vulnerabilities".to_string());
        }
    }

    // Vérifier cargo vet
    let vet = Command::new("cargo")
        .args(["vet", "--locked"])
        .output();

    if let Ok(output) = vet {
        if !output.status.success() {
            issues.push("cargo vet found unaudited dependencies".to_string());
        }
    }

    if issues.is_empty() {
        Ok(())
    } else {
        Err(issues)
    }
}

/// Résumé des commandes essentielles
pub fn supply_chain_commands() {
    println!("Supply Chain Security Commands");
    println!("==============================");
    println!("");
    println!("# Audit réactif (CVEs connus)");
    println!("cargo audit");
    println!("");
    println!("# Audit proactif (cargo-vet)");
    println!("cargo vet init           # Initialiser");
    println!("cargo vet                # Vérifier status");
    println!("cargo vet inspect CRATE  # Inspecter code");
    println!("cargo vet certify CRATE  # Certifier après review");
    println!("cargo vet import ORG     # Importer audits externes");
    println!("");
    println!("# CI: Bloquer les dépendances non auditées");
    println!("cargo vet --locked");
}
```

### Test Moulinette
```
supply_chain problem -> "compromised maintainer can inject malicious code"
supply_chain audit_type cargo-audit -> "reactive, checks known CVEs"
supply_chain audit_type cargo-vet -> "proactive, requires explicit audits"
supply_chain criteria -> ["safe-to-run", "safe-to-deploy"]
```

---

## RÉCAPITULATIF MODULE 1.8

| Exercice | Concepts | Count |
|----------|----------|-------|
| ADD-1 stress_testing | 1.8.13.a-c | 3 |
| ADD-2 formal_verification | 1.8.14.a-j | 10 |
| ADD-3 supply_chain_security | 1.8.15.a-j | 10 |
| **TOTAL AJOUTÉ** | | **23** |

**Couverture Module 1.8: 42 + 23 = 65/65 = 100%**

