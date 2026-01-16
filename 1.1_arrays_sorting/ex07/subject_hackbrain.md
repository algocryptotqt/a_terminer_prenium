<thinking>
## Analyse du Concept
- Concept : Ternary Search & Unimodal Functions (recherche ternaire et fonctions unimodales)
- Phase demandée : 1
- Adapté ? OUI — Technique de recherche avancée parfaite pour Phase 1. Extension naturelle de binary search.

## Combo Base + Bonus
- Exercice de base : Ternary search discret/float + golden section + applications géométriques
- Bonus : Newton-Raphson hybride, recherche multi-dimensionnelle
- Palier bonus : 🔥 Avancé
- Progression logique ? OUI — Base = recherche 1D, Bonus = optimisation avancée

## Prérequis & Difficulté
- Prérequis réels : Binary Search, fonctions closures, géométrie basique
- Difficulté estimée : 5/10
- Cohérent avec phase ? OUI

## Aspect Fun/Culture
- Contexte choisi : Portal (jeu vidéo) — GLaDOS et l'optimisation
- MEME mnémotechnique : "The cake is a lie, but the minimum is real" / "For science!"
- Pourquoi c'est fun : GLaDOS est obsédée par les tests optimaux, comme nous cherchons l'extremum optimal. Le golden ratio (section dorée) = mathématiques élégantes = Aperture Science.

## Scénarios d'Échec (5 mutants)
1. Mutant A (Boundary) : Condition d'arrêt mauvaise (hi - lo > 2 vs > 3)
2. Mutant B (Logic) : Inversion min/max (< vs >)
3. Mutant C (Precision) : Pas assez d'itérations pour float
4. Mutant D (Golden) : Mauvais ratio phi (1.5 au lieu de 1.618...)
5. Mutant E (Return) : Retourne lo au lieu de (lo+hi)/2

## Verdict
VALIDE — Analogie Portal excellente, exercice mathématique élégant avec applications pratiques
Note créativité : 97/100
</thinking>

---

# Exercice 1.1.7 : aperture_optimizer

**Module :**
1.1 — Arrays & Sorting

**Concept :**
h — Ternary Search & Unimodal Functions

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (ternary search + golden section + applications)

**Langages :**
Rust Edition 2024 / C17

**Prérequis :**
- Binary Search
- Fonctions et closures
- Géométrie 2D basique

**Domaines :**
Tri, Calcul, MD

**Durée estimée :**
60 min

**XP Base :**
150

**Complexité :**
T2 O(log n) itérations × S1 O(1)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- **Rust :** `src/lib.rs`, `Cargo.toml`
- **C :** `aperture_optimizer.c`, `aperture_optimizer.h`

**Fonctions autorisées :**
- Rust : std (f64 operations, closures)
- C : math.h (sqrt, fabs, pow), fonctions de base

**Fonctions interdites :**
- Bibliothèques d'optimisation externes

---

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🔬 PORTAL — "We do what we must, because we can... FOR SCIENCE!"**

Bienvenue chez Aperture Science. Je suis GLaDOS, et aujourd'hui nous testons un nouvel algorithme d'optimisation.

**Le problème :**
Les chambres de test ont des fonctions d'énergie **unimodales** — elles descendent jusqu'à un point optimal, puis remontent. Comme une colline inversée.

```
Énergie
   │
 10│\
   │ \
  5│  \_____/
   │        \
  1│         minimum ← LE POINT OPTIMAL
   └─────────────────
        Position
```

**Binary Search ne marche pas ici !** Pourquoi ? Parce qu'il n'y a pas de propriété "gauche = faux, droite = vrai". La fonction peut être décroissante des deux côtés du point de test.

**La solution : Ternary Search**

Au lieu de diviser en 2, on divise en 3 :
```
lo           m1           m2           hi
 |            |            |            |
 ├────────────┼────────────┼────────────┤
      tiers 1      tiers 2      tiers 3

Si f(m1) < f(m2) → le minimum est dans [lo, m2]
Si f(m1) > f(m2) → le minimum est dans [m1, hi]
```

Chaque itération réduit l'intervalle de 1/3. Après log₃(n) itérations, on trouve l'optimum !

**Le Golden Ratio (nombre d'or) :**

GLaDOS adore les mathématiques élégantes. La **Golden Section Search** utilise φ = (1+√5)/2 ≈ 1.618 pour être encore PLUS efficace que ternary search.

```
φ = 1.6180339887...
Le nombre d'or, présent partout dans la nature.
Et maintenant, dans ton code.
```

**Ta mission :**

Créer l'**Aperture Optimizer** — un système de recherche d'extrema pour les tests de Chell.

---

#### 1.2.2 Version Académique

**Recherche Ternaire (Ternary Search) :**

Algorithme de recherche pour trouver l'extremum (minimum ou maximum) d'une fonction **unimodale** sur un intervalle.

**Fonction unimodale :**
- Pour un minimum : décroissante puis croissante
- Pour un maximum : croissante puis décroissante

**Algorithme :**
1. Diviser l'intervalle [lo, hi] en trois parties
2. Calculer m1 = lo + (hi-lo)/3 et m2 = hi - (hi-lo)/3
3. Comparer f(m1) et f(m2)
4. Éliminer le tiers où l'extremum ne peut pas être
5. Répéter jusqu'à convergence

**Golden Section Search :**

Variante plus efficace utilisant le nombre d'or φ = (1+√5)/2 ≈ 1.618.

L'avantage : on réutilise un point de calcul entre les itérations, réduisant le nombre d'évaluations de fonction.

**Complexité :**
- Ternary : O(2 log₃ n) évaluations
- Golden : O(log_φ n) évaluations ≈ 1.44 log₂ n

---

### 1.3 Prototypes

#### Rust

```rust
pub mod aperture_optimizer {
    /// Recherche ternaire pour minimum (discret)
    /// "Finding the lowest energy state in the test chamber"
    pub fn find_minimum_discrete<F>(lo: i64, hi: i64, f: F) -> i64
    where
        F: Fn(i64) -> i64;

    /// Recherche ternaire pour maximum (discret)
    /// "Peak performance detection"
    pub fn find_maximum_discrete<F>(lo: i64, hi: i64, f: F) -> i64
    where
        F: Fn(i64) -> i64;

    /// Recherche ternaire pour minimum (flottant)
    /// "Precision testing for science"
    pub fn find_minimum_float<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64;

    /// Recherche ternaire pour maximum (flottant)
    pub fn find_maximum_float<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64;

    /// Golden Section Search — "The elegant solution"
    pub fn golden_section_min<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64;

    pub fn golden_section_max<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64;

    // ═══════════════════════════════════════════════════════════
    // APPLICATIONS — "Test Chamber Problems"
    // ═══════════════════════════════════════════════════════════

    /// Minimum d'une fonction quadratique ax² + bx + c
    /// "Parabolic trajectory optimization"
    pub fn quadratic_vertex(a: f64, b: f64, c: f64, lo: f64, hi: f64) -> (f64, f64);

    /// Aire maximale d'un rectangle inscrit dans un demi-cercle
    /// "Turret placement optimization"
    pub fn max_rectangle_in_semicircle(radius: f64) -> f64;

    /// Point le plus proche sur un segment
    /// "Laser redirection point"
    pub fn closest_on_segment(
        segment: ((f64, f64), (f64, f64)),
        point: (f64, f64),
    ) -> (f64, f64);

    /// Angle de rotation optimal pour minimiser la distance totale
    /// "Portal gun calibration"
    pub fn optimal_rotation_angle(
        points: &[(f64, f64)],
        target: (f64, f64),
    ) -> f64;

    /// Centre du plus petit cercle englobant (approximation)
    /// "Minimum containment field"
    pub fn minimax_center(points: &[(f64, f64)]) -> ((f64, f64), f64);

    /// Temps minimum de trajet avec vitesse variable
    /// "Speed gel optimization"
    pub fn optimal_travel_time(
        distance: f64,
        max_speed: f64,
        acceleration_distance: f64,
    ) -> f64;

    /// Binary search sur la dérivée (alternative)
    /// "Derivative-based approach"
    pub fn find_extremum_derivative<F>(
        lo: f64,
        hi: f64,
        eps: f64,
        derivative: F,
    ) -> f64
    where
        F: Fn(f64) -> f64;
}
```

#### C

```c
#ifndef APERTURE_OPTIMIZER_H
#define APERTURE_OPTIMIZER_H

#include <stddef.h>
#include <stdint.h>

// Type pour les fonctions à optimiser
typedef int64_t (*discrete_func)(int64_t);
typedef double  (*continuous_func)(double);

// ═══════════════════════════════════════════════════════════════
// RECHERCHE TERNAIRE
// ═══════════════════════════════════════════════════════════════

int64_t find_minimum_discrete(int64_t lo, int64_t hi, discrete_func f);
int64_t find_maximum_discrete(int64_t lo, int64_t hi, discrete_func f);

double find_minimum_float(double lo, double hi, double eps, continuous_func f);
double find_maximum_float(double lo, double hi, double eps, continuous_func f);

// ═══════════════════════════════════════════════════════════════
// GOLDEN SECTION SEARCH
// ═══════════════════════════════════════════════════════════════

double golden_section_min(double lo, double hi, double eps, continuous_func f);
double golden_section_max(double lo, double hi, double eps, continuous_func f);

// ═══════════════════════════════════════════════════════════════
// APPLICATIONS
// ═══════════════════════════════════════════════════════════════

typedef struct {
    double x;
    double y;
} t_point;

typedef struct {
    double x;
    double value;
} t_vertex;

t_vertex quadratic_vertex(double a, double b, double c, double lo, double hi);
double   max_rectangle_in_semicircle(double radius);
t_point  closest_on_segment(t_point seg_start, t_point seg_end, t_point point);
double   optimal_rotation_angle(const t_point *points, size_t n, t_point target);

typedef struct {
    t_point center;
    double  radius;
} t_circle;

t_circle minimax_center(const t_point *points, size_t n);
double   optimal_travel_time(double distance, double max_speed, double accel_dist);
double   find_extremum_derivative(double lo, double hi, double eps, continuous_func derivative);

#endif
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Faits Fascinants

**🔢 Le Nombre d'Or (φ) :**
```
φ = (1 + √5) / 2 ≈ 1.6180339887...

Propriété magique : φ² = φ + 1
                    1/φ = φ - 1

Présent dans : spirales de galaxies, coquillages,
               tournesols, architecture du Parthénon,
               et maintenant... TON CODE !
```

**🎮 Dans les jeux :**
Les moteurs physiques utilisent la recherche ternaire pour trouver les points de collision optimaux, les trajectoires de projectiles, et les positions de caméra.

**📈 En finance :**
Les traders utilisent le "Fibonacci retracement" basé sur φ pour prédire les niveaux de support/résistance. Le golden ratio est partout en bourse !

### 2.2 Pourquoi pas Binary Search ?

```
Binary Search : Trouve une VALEUR dans un tableau TRIÉ
Ternary Search : Trouve un EXTREMUM dans une fonction UNIMODALE

Binary : "Où est 42 dans ce tableau ?"
         → La valeur est à gauche OU à droite du milieu

Ternary : "Où est le MINIMUM de cette fonction ?"
          → Le minimum est... quelque part. On ne peut pas
            décider avec UN seul point !
```

### 2.5 Dans la Vraie Vie

| Métier | Utilisation |
|--------|-------------|
| **Game Developer** | Collision detection, pathfinding optimal |
| **Quant** | Fibonacci retracement, portfolio optimization |
| **ML Engineer** | Hyperparameter tuning (learning rate) |
| **Roboticist** | Trajectory optimization, sensor calibration |
| **Graphics Programmer** | Camera placement, lighting optimization |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
aperture_optimizer.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 12 tests
test test_discrete_minimum ... ok
test test_discrete_maximum ... ok
test test_float_minimum ... ok
test test_float_maximum ... ok
test test_golden_section ... ok
test test_quadratic_vertex ... ok
test test_rectangle_semicircle ... ok
test test_closest_segment ... ok
test test_rotation_angle ... ok
test test_minimax ... ok
test test_travel_time ... ok
test test_derivative ... ok

test result: ok. 12 passed; 0 failed

$ ./target/release/demo
Finding minimum of f(x) = (x - 50)²...
Minimum at x = 50 ✓

Finding maximum of inscribed rectangle...
For radius 1.0, max area = 1.0 ✓

"The cake is a lie, but the optimization is real."
- GLaDOS
```

---

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(log n) avec convergence quadratique pour Newton

**Space Complexity attendue :**
O(1)

**Domaines Bonus :**
`Calcul, MD`

#### 3.1.1 Consigne Bonus

**🔬 PORTAL 2 — "Speedy thing goes in, speedy thing comes out"**

Cave Johnson veut de l'optimisation EXTRÊME. Les tests standards ne suffisent plus !

**Ta mission bonus :**

1. **`newton_raphson_min`** — Convergence quadratique quand la dérivée seconde est disponible

2. **`nelder_mead_2d`** — Recherche du minimum en 2D sans dérivée (simplex method)

3. **`simulated_annealing_min`** — Optimisation stochastique pour éviter les minima locaux

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  Newton : convergence en ~5-10 iter     │
│  Nelder-Mead : 2D sans dérivée          │
│  Annealing : éviter minima locaux       │
│  Précision : eps = 10⁻¹²                │
└─────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```rust
/// Newton-Raphson avec dérivées première et seconde
pub fn newton_raphson_min<F, DF, DDF>(
    start: f64,
    f: F,
    df: DF,
    ddf: DDF,
    eps: f64,
    max_iter: usize,
) -> f64
where
    F: Fn(f64) -> f64,
    DF: Fn(f64) -> f64,   // Première dérivée
    DDF: Fn(f64) -> f64;  // Seconde dérivée

/// Nelder-Mead en 2D (downhill simplex)
pub fn nelder_mead_2d<F>(
    start: (f64, f64),
    f: F,
    eps: f64,
    max_iter: usize,
) -> (f64, f64)
where
    F: Fn(f64, f64) -> f64;

/// Simulated Annealing pour échapper aux minima locaux
pub fn simulated_annealing<F>(
    lo: f64,
    hi: f64,
    f: F,
    initial_temp: f64,
    cooling_rate: f64,
    iterations: usize,
) -> f64
where
    F: Fn(f64) -> f64;
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Convergence | Linéaire O(log n) | Quadratique (Newton) |
| Dimensions | 1D seulement | 1D et 2D |
| Minima locaux | Peut rester coincé | Annealing les évite |
| Dérivées | Non requises | Optionnelles (Newton) |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected Output | Points |
|------|-------|-----------------|--------|
| discrete_min_parabola | (x-50)², [0,100] | 50 | 3 |
| discrete_min_asymmetric | piece-wise, [0,100] | 30 | 3 |
| discrete_max | -(x-75)², [0,100] | 75 | 3 |
| float_min | x²-4x+5, [0,10] | ≈2.0 | 3 |
| float_max | -(x-π)², [0,6] | ≈π | 3 |
| golden_min | (x-3.14)², [0,10] | ≈3.14 | 4 |
| golden_precision | (x-√2)², [0,3] | ≈1.414 (eps=10⁻⁹) | 4 |
| quadratic_vertex | 2x²-8x+10 | x=2, y=2 | 4 |
| semicircle_r1 | radius=1 | area≈1.0 | 5 |
| semicircle_r2 | radius=2 | area≈4.0 | 3 |
| closest_inside | seg (0,0)-(10,0), pt (5,5) | (5,0) | 4 |
| closest_outside | seg (0,0)-(10,0), pt (-5,3) | (0,0) | 4 |
| rotation_simple | 3 points | correct angle | 4 |
| minimax_triangle | 3 points | circumcenter | 5 |
| derivative_method | f'(x) given | matches ternary | 3 |
| edge_single_point | lo == hi | returns lo | 2 |
| edge_narrow_range | hi - lo = 1 | correct extremum | 2 |

### 4.2 main.rs de test

```rust
use aperture_optimizer::*;
use std::f64::consts::PI;

fn main() {
    println!("=== APERTURE SCIENCE OPTIMIZER TESTS ===\n");
    println!("\"We do what we must, because we can.\"\n");

    // Test 1: Discrete Minimum
    let f = |x: i64| (x - 50) * (x - 50);
    assert_eq!(find_minimum_discrete(0, 100, f), 50);
    println!("[OK] Discrete minimum: (x-50)² → x = 50");

    // Test 2: Discrete Maximum
    let g = |x: i64| -((x - 75) * (x - 75));
    assert_eq!(find_maximum_discrete(0, 100, g), 75);
    println!("[OK] Discrete maximum: -(x-75)² → x = 75");

    // Test 3: Float Minimum
    let h = |x: f64| x * x - 4.0 * x + 5.0;
    let min = find_minimum_float(0.0, 10.0, 1e-9, h);
    assert!((min - 2.0).abs() < 1e-6);
    println!("[OK] Float minimum: x²-4x+5 → x ≈ 2.0");

    // Test 4: Golden Section
    let k = |x: f64| (x - PI).powi(2);
    let golden = golden_section_min(0.0, 6.0, 1e-9, k);
    assert!((golden - PI).abs() < 1e-6);
    println!("[OK] Golden section: (x-π)² → x ≈ π");

    // Test 5: Quadratic Vertex
    let (x, y) = quadratic_vertex(2.0, -8.0, 10.0, -10.0, 10.0);
    assert!((x - 2.0).abs() < 1e-6);
    assert!((y - 2.0).abs() < 1e-6);
    println!("[OK] Quadratic vertex: 2x²-8x+10 → (2, 2)");

    // Test 6: Max Rectangle in Semicircle
    let area1 = max_rectangle_in_semicircle(1.0);
    assert!((area1 - 1.0).abs() < 1e-6);

    let area2 = max_rectangle_in_semicircle(2.0);
    assert!((area2 - 4.0).abs() < 1e-6);
    println!("[OK] Rectangle in semicircle: r=1 → area=1, r=2 → area=4");

    // Test 7: Closest Point on Segment
    let seg = ((0.0, 0.0), (10.0, 0.0));
    let pt = (5.0, 5.0);
    let closest = closest_on_segment(seg, pt);
    assert!((closest.0 - 5.0).abs() < 1e-6);
    assert!((closest.1 - 0.0).abs() < 1e-6);
    println!("[OK] Closest on segment: (5,5) → (5,0)");

    // Test outside segment
    let pt2 = (-5.0, 3.0);
    let closest2 = closest_on_segment(seg, pt2);
    assert!((closest2.0 - 0.0).abs() < 1e-6);
    println!("[OK] Closest outside: (-5,3) → (0,0)");

    println!("\n=== ALL TESTS PASSED ===");
    println!("\"The cake is a lie, but the optimization is real.\"");
    println!("- GLaDOS");
}
```

### 4.3 Solution de référence (Rust)

```rust
pub mod aperture_optimizer {
    use std::f64::consts::PI;

    const PHI: f64 = 1.6180339887498948482;  // Golden ratio
    const RESPHI: f64 = 0.3819660112501051518;  // 2 - PHI

    // ═══════════════════════════════════════════════════════════
    // TERNARY SEARCH — DISCRETE
    // ═══════════════════════════════════════════════════════════

    pub fn find_minimum_discrete<F>(mut lo: i64, mut hi: i64, f: F) -> i64
    where
        F: Fn(i64) -> i64,
    {
        while hi - lo > 2 {
            let m1 = lo + (hi - lo) / 3;
            let m2 = hi - (hi - lo) / 3;

            if f(m1) < f(m2) {
                hi = m2;
            } else {
                lo = m1;
            }
        }

        // Check remaining candidates
        let mut best = lo;
        let mut best_val = f(lo);
        for x in (lo + 1)..=hi {
            let val = f(x);
            if val < best_val {
                best_val = val;
                best = x;
            }
        }
        best
    }

    pub fn find_maximum_discrete<F>(mut lo: i64, mut hi: i64, f: F) -> i64
    where
        F: Fn(i64) -> i64,
    {
        while hi - lo > 2 {
            let m1 = lo + (hi - lo) / 3;
            let m2 = hi - (hi - lo) / 3;

            if f(m1) > f(m2) {
                hi = m2;
            } else {
                lo = m1;
            }
        }

        let mut best = lo;
        let mut best_val = f(lo);
        for x in (lo + 1)..=hi {
            let val = f(x);
            if val > best_val {
                best_val = val;
                best = x;
            }
        }
        best
    }

    // ═══════════════════════════════════════════════════════════
    // TERNARY SEARCH — FLOATING POINT
    // ═══════════════════════════════════════════════════════════

    pub fn find_minimum_float<F>(mut lo: f64, mut hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64,
    {
        for _ in 0..200 {
            if hi - lo < eps {
                break;
            }

            let m1 = lo + (hi - lo) / 3.0;
            let m2 = hi - (hi - lo) / 3.0;

            if f(m1) < f(m2) {
                hi = m2;
            } else {
                lo = m1;
            }
        }

        (lo + hi) / 2.0
    }

    pub fn find_maximum_float<F>(mut lo: f64, mut hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64,
    {
        for _ in 0..200 {
            if hi - lo < eps {
                break;
            }

            let m1 = lo + (hi - lo) / 3.0;
            let m2 = hi - (hi - lo) / 3.0;

            if f(m1) > f(m2) {
                hi = m2;
            } else {
                lo = m1;
            }
        }

        (lo + hi) / 2.0
    }

    // ═══════════════════════════════════════════════════════════
    // GOLDEN SECTION SEARCH
    // ═══════════════════════════════════════════════════════════

    pub fn golden_section_min<F>(mut lo: f64, mut hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64,
    {
        let mut m1 = hi - RESPHI * (hi - lo);
        let mut m2 = lo + RESPHI * (hi - lo);
        let mut f1 = f(m1);
        let mut f2 = f(m2);

        while (hi - lo).abs() > eps {
            if f1 < f2 {
                hi = m2;
                m2 = m1;
                f2 = f1;
                m1 = hi - RESPHI * (hi - lo);
                f1 = f(m1);
            } else {
                lo = m1;
                m1 = m2;
                f1 = f2;
                m2 = lo + RESPHI * (hi - lo);
                f2 = f(m2);
            }
        }

        (lo + hi) / 2.0
    }

    pub fn golden_section_max<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64
    where
        F: Fn(f64) -> f64,
    {
        golden_section_min(lo, hi, eps, |x| -f(x))
    }

    // ═══════════════════════════════════════════════════════════
    // APPLICATIONS
    // ═══════════════════════════════════════════════════════════

    /// Minimum of ax² + bx + c in [lo, hi]
    pub fn quadratic_vertex(a: f64, b: f64, c: f64, lo: f64, hi: f64) -> (f64, f64) {
        // Vertex at x = -b / (2a)
        let vertex_x = -b / (2.0 * a);
        let x = vertex_x.clamp(lo, hi);
        let y = a * x * x + b * x + c;
        (x, y)
    }

    /// Max area of rectangle inscribed in semicircle of radius r
    /// Rectangle has width 2x and height y, where x² + y² = r²
    pub fn max_rectangle_in_semicircle(radius: f64) -> f64 {
        // Area = 2x * y = 2x * sqrt(r² - x²)
        // Optimize for x in [0, r]
        let f = |x: f64| {
            let y = (radius * radius - x * x).sqrt();
            2.0 * x * y
        };

        let optimal_x = golden_section_max(0.0, radius, 1e-12, f);
        let optimal_y = (radius * radius - optimal_x * optimal_x).sqrt();

        2.0 * optimal_x * optimal_y
    }

    /// Closest point on segment to given point
    pub fn closest_on_segment(
        segment: ((f64, f64), (f64, f64)),
        point: (f64, f64),
    ) -> (f64, f64) {
        let ((x1, y1), (x2, y2)) = segment;
        let (px, py) = point;

        let dx = x2 - x1;
        let dy = y2 - y1;
        let len_sq = dx * dx + dy * dy;

        if len_sq < 1e-12 {
            return (x1, y1);  // Degenerate segment
        }

        // Project point onto infinite line
        let t = ((px - x1) * dx + (py - y1) * dy) / len_sq;

        // Clamp to [0, 1] to stay on segment
        let t_clamped = t.clamp(0.0, 1.0);

        (x1 + t_clamped * dx, y1 + t_clamped * dy)
    }

    /// Optimal rotation angle to minimize total distance
    pub fn optimal_rotation_angle(
        points: &[(f64, f64)],
        target: (f64, f64),
    ) -> f64 {
        if points.is_empty() {
            return 0.0;
        }

        // Total distance after rotation by angle theta
        let total_dist = |theta: f64| -> f64 {
            points.iter()
                .map(|&(x, y)| {
                    let cos_t = theta.cos();
                    let sin_t = theta.sin();
                    let rx = x * cos_t - y * sin_t;
                    let ry = x * sin_t + y * cos_t;
                    let dx = rx - target.0;
                    let dy = ry - target.1;
                    (dx * dx + dy * dy).sqrt()
                })
                .sum()
        };

        golden_section_min(0.0, 2.0 * PI, 1e-9, total_dist)
    }

    /// Center of minimum enclosing circle (approximation)
    pub fn minimax_center(points: &[(f64, f64)]) -> ((f64, f64), f64) {
        if points.is_empty() {
            return ((0.0, 0.0), 0.0);
        }

        if points.len() == 1 {
            return (points[0], 0.0);
        }

        // Simple approach: use bounding box center as starting point
        // Then use ternary search to refine

        let (min_x, max_x) = points.iter()
            .map(|p| p.0)
            .fold((f64::MAX, f64::MIN), |(mn, mx), x| (mn.min(x), mx.max(x)));

        let (min_y, max_y) = points.iter()
            .map(|p| p.1)
            .fold((f64::MAX, f64::MIN), |(mn, mx), y| (mn.min(y), mx.max(y)));

        // Max distance from a center point
        let max_dist = |cx: f64, cy: f64| -> f64 {
            points.iter()
                .map(|&(x, y)| {
                    let dx = x - cx;
                    let dy = y - cy;
                    (dx * dx + dy * dy).sqrt()
                })
                .fold(0.0f64, |a, b| a.max(b))
        };

        // Ternary search on x
        let best_x = golden_section_min(min_x, max_x, 1e-9, |x| {
            // For this x, find best y
            let best_y = golden_section_min(min_y, max_y, 1e-9, |y| max_dist(x, y));
            max_dist(x, best_y)
        });

        let best_y = golden_section_min(min_y, max_y, 1e-9, |y| max_dist(best_x, y));

        let radius = max_dist(best_x, best_y);

        ((best_x, best_y), radius)
    }

    /// Optimal travel time with speed curve
    pub fn optimal_travel_time(
        distance: f64,
        max_speed: f64,
        acceleration_distance: f64,
    ) -> f64 {
        if distance <= 0.0 || max_speed <= 0.0 {
            return 0.0;
        }

        // Simple model: accelerate for accel_dist, constant speed, decelerate
        let min_time = |accel_fraction: f64| -> f64 {
            let accel_dist = (accel_fraction * distance).min(distance / 2.0);
            let cruise_dist = distance - 2.0 * accel_dist;

            // Time to accelerate (simplified: v = sqrt(2 * a * d))
            let accel_time = (2.0 * accel_dist / max_speed).sqrt();
            let cruise_time = cruise_dist / max_speed;

            2.0 * accel_time + cruise_time
        };

        golden_section_min(0.0, 1.0, 1e-9, min_time)
    }

    /// Binary search on derivative
    pub fn find_extremum_derivative<F>(
        mut lo: f64,
        mut hi: f64,
        eps: f64,
        derivative: F,
    ) -> f64
    where
        F: Fn(f64) -> f64,
    {
        // Find where derivative = 0 using binary search
        for _ in 0..200 {
            if hi - lo < eps {
                break;
            }

            let mid = (lo + hi) / 2.0;
            let d = derivative(mid);

            if d < 0.0 {
                lo = mid;  // Derivative negative → minimum is to the right
            } else {
                hi = mid;  // Derivative positive → minimum is to the left
            }
        }

        (lo + hi) / 2.0
    }
}
```

### 4.5 Solutions refusées (avec explications)

```rust
// ❌ REFUSÉ: Condition d'arrêt incorrecte
pub fn find_minimum_discrete_bad<F>(mut lo: i64, mut hi: i64, f: F) -> i64 {
    while hi - lo > 3 {  // BUG: > 3 au lieu de > 2
        // Peut manquer le minimum si l'intervalle final a 3 éléments
    }
}

// ❌ REFUSÉ: Inversion min/max
pub fn find_minimum_float_bad<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64 {
    // BUG: Utilise > au lieu de <
    if f(m1) > f(m2) {  // FAUX: cherche maximum!
        hi = m2;
    }
}

// ❌ REFUSÉ: Pas assez d'itérations
pub fn golden_section_bad<F>(mut lo: f64, mut hi: f64, eps: f64, f: F) -> f64 {
    for _ in 0..10 {  // BUG: Seulement 10 itérations
        // Précision insuffisante pour eps = 10⁻⁹
    }
}

// ❌ REFUSÉ: Mauvais ratio golden
const PHI_BAD: f64 = 1.5;  // BUG: Devrait être 1.618...
// Convergence sous-optimale

// ❌ REFUSÉ: Retourne lo au lieu de moyenne
pub fn find_minimum_float_bad2<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64 {
    // ... iterations ...
    lo  // BUG: Devrait être (lo + hi) / 2.0
}
```

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "aperture_optimizer",
  "language": "rust",
  "version": "edition_2024",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (ternary + golden section + applications)",
  "tags": ["search", "optimization", "ternary_search", "golden_section", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "aperture_optimizer",
    "module": true,
    "functions": [
      "find_minimum_discrete",
      "find_maximum_discrete",
      "find_minimum_float",
      "find_maximum_float",
      "golden_section_min",
      "golden_section_max",
      "quadratic_vertex",
      "max_rectangle_in_semicircle",
      "closest_on_segment",
      "optimal_rotation_angle",
      "minimax_center",
      "optimal_travel_time",
      "find_extremum_derivative"
    ]
  },

  "driver": {
    "reference_file": "solutions/ref_aperture_optimizer.rs",

    "edge_cases": [
      {
        "name": "discrete_min_parabola",
        "function": "find_minimum_discrete",
        "setup": {"closure": "x => (x - 50) * (x - 50)"},
        "args": [0, 100, "$closure"],
        "expected": 50
      },
      {
        "name": "discrete_max",
        "function": "find_maximum_discrete",
        "setup": {"closure": "x => -((x - 75) * (x - 75))"},
        "args": [0, 100, "$closure"],
        "expected": 75
      },
      {
        "name": "float_min",
        "function": "find_minimum_float",
        "setup": {"closure": "x => x*x - 4.0*x + 5.0"},
        "args": [0.0, 10.0, 1e-9, "$closure"],
        "expected_approx": {"value": 2.0, "tolerance": 1e-6}
      },
      {
        "name": "golden_pi",
        "function": "golden_section_min",
        "setup": {"closure": "x => (x - 3.14159265).powi(2)"},
        "args": [0.0, 6.0, 1e-9, "$closure"],
        "expected_approx": {"value": 3.14159265, "tolerance": 1e-6}
      },
      {
        "name": "quadratic_basic",
        "function": "quadratic_vertex",
        "args": [2.0, -8.0, 10.0, -10.0, 10.0],
        "expected": [2.0, 2.0]
      },
      {
        "name": "semicircle_r1",
        "function": "max_rectangle_in_semicircle",
        "args": [1.0],
        "expected_approx": {"value": 1.0, "tolerance": 1e-6}
      },
      {
        "name": "closest_inside",
        "function": "closest_on_segment",
        "args": [[[0.0, 0.0], [10.0, 0.0]], [5.0, 5.0]],
        "expected_approx": {"value": [5.0, 0.0], "tolerance": 1e-6}
      },
      {
        "name": "closest_outside",
        "function": "closest_on_segment",
        "args": [[[0.0, 0.0], [10.0, 0.0]], [-5.0, 3.0]],
        "expected_approx": {"value": [0.0, 0.0], "tolerance": 1e-6}
      },
      {
        "name": "single_point_range",
        "function": "find_minimum_discrete",
        "setup": {"closure": "x => x"},
        "args": [42, 42, "$closure"],
        "expected": 42,
        "is_trap": true,
        "trap_explanation": "lo == hi, doit retourner lo"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 100,
      "generators": [
        {
          "type": "float",
          "param_index": 0,
          "params": {"min": -100.0, "max": 100.0}
        },
        {
          "type": "float",
          "param_index": 1,
          "params": {"min": -100.0, "max": 100.0}
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["sqrt", "abs", "pow", "sin", "cos"],
    "forbidden_functions": [],
    "check_precision": true,
    "blocking": true
  },

  "bonus": {
    "tier": "ADVANCED",
    "icon": "🔥",
    "xp_multiplier": 3,
    "functions": [
      "newton_raphson_min",
      "nelder_mead_2d",
      "simulated_annealing"
    ]
  }
}
```

### 4.10 Solutions Mutantes

```rust
// ═══════════════════════════════════════════════════════════════
// MUTANT A (Boundary) : Condition d'arrêt incorrecte
// ═══════════════════════════════════════════════════════════════

pub fn find_minimum_discrete_bad<F>(mut lo: i64, mut hi: i64, f: F) -> i64
where F: Fn(i64) -> i64,
{
    // BUG: hi - lo > 3 au lieu de > 2
    while hi - lo > 3 {
        let m1 = lo + (hi - lo) / 3;
        let m2 = hi - (hi - lo) / 3;
        if f(m1) < f(m2) { hi = m2; } else { lo = m1; }
    }
    lo  // Peut manquer le minimum
}
// Pourquoi c'est faux : Avec > 3, on peut sortir avec 4 éléments et mal choisir
// Ce qui était pensé : "3 éléments suffisent"

// ═══════════════════════════════════════════════════════════════
// MUTANT B (Logic) : Inversion < / >
// ═══════════════════════════════════════════════════════════════

pub fn find_minimum_float_bad<F>(mut lo: f64, mut hi: f64, eps: f64, f: F) -> f64 {
    for _ in 0..200 {
        if hi - lo < eps { break; }
        let m1 = lo + (hi - lo) / 3.0;
        let m2 = hi - (hi - lo) / 3.0;
        // BUG: > au lieu de <
        if f(m1) > f(m2) {
            hi = m2;
        } else {
            lo = m1;
        }
    }
    (lo + hi) / 2.0
}
// Pourquoi c'est faux : Trouve le maximum au lieu du minimum
// Ce qui était pensé : Confusion entre min et max

// ═══════════════════════════════════════════════════════════════
// MUTANT C (Precision) : Pas assez d'itérations
// ═══════════════════════════════════════════════════════════════

pub fn golden_section_bad<F>(mut lo: f64, mut hi: f64, eps: f64, f: F) -> f64 {
    // BUG: Seulement 10 itérations
    for _ in 0..10 {
        // ... same logic ...
    }
    (lo + hi) / 2.0
}
// Pourquoi c'est faux : 10 itérations donnent précision ~10⁻³, pas 10⁻⁹
// Ce qui était pensé : "10 itérations devraient suffire"

// ═══════════════════════════════════════════════════════════════
// MUTANT D (Golden) : Mauvais ratio phi
// ═══════════════════════════════════════════════════════════════

const PHI_BAD: f64 = 1.5;  // BUG: Devrait être 1.618...
const RESPHI_BAD: f64 = 0.5;  // BUG: Devrait être 0.382...

pub fn golden_section_wrong_ratio<F>(lo: f64, hi: f64, eps: f64, f: F) -> f64 {
    // Utilise PHI_BAD et RESPHI_BAD
}
// Pourquoi c'est faux : Ne réutilise pas correctement les évaluations
// Ce qui était pensé : "1.5 est proche de 1.618"

// ═══════════════════════════════════════════════════════════════
// MUTANT E (Return) : Retourne lo au lieu de moyenne
// ═══════════════════════════════════════════════════════════════

pub fn find_minimum_float_bad3<F>(mut lo: f64, mut hi: f64, eps: f64, f: F) -> f64 {
    // ... correct iterations ...

    lo  // BUG: Devrait être (lo + hi) / 2.0
}
// Pourquoi c'est faux : Retourne la borne inférieure, pas le centre
// Ce qui était pensé : "lo est le minimum"
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

| Concept | Description | Pourquoi c'est important |
|---------|-------------|-------------------------|
| **Ternary Search** | Trouver extremum d'une fonction unimodale | Alternative quand binary search ne marche pas |
| **Golden Section** | Optimisation avec nombre d'or | Plus efficace que ternary |
| **Unimodal Functions** | Monotone puis change de direction | Condition nécessaire |
| **Floating Point** | Précision et itérations | Fondamental en numérique |
| **Géométrie** | Applications aux formes | Omniprésent en graphics |

### 5.2 LDA — Traduction Littérale

**find_minimum_discrete**
```
FONCTION find_minimum_discrete QUI RETOURNE UN ENTIER ET PREND EN PARAMÈTRES lo, hi, ET f
DÉBUT FONCTION
    TANT QUE hi MOINS lo EST SUPÉRIEUR À 2 FAIRE
        AFFECTER lo PLUS (hi MOINS lo) DIVISÉ PAR 3 À m1
        AFFECTER hi MOINS (hi MOINS lo) DIVISÉ PAR 3 À m2

        SI f(m1) EST INFÉRIEUR À f(m2) ALORS
            AFFECTER m2 À hi
        SINON
            AFFECTER m1 À lo
        FIN SI
    FIN TANT QUE

    DÉCLARER best ÉGAL À lo
    POUR x ALLANT DE lo PLUS 1 À hi FAIRE
        SI f(x) EST INFÉRIEUR À f(best) ALORS
            AFFECTER x À best
        FIN SI
    FIN POUR

    RETOURNER best
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Ternary Search pour Minimum
---
1. TANT QUE l'intervalle est grand (> 2 éléments) :
   a. CALCULER m1 = lo + (hi - lo) / 3
   b. CALCULER m2 = hi - (hi - lo) / 3
   c. SI f(m1) < f(m2) :
      - Le minimum est dans [lo, m2]
      - RÉDUIRE hi à m2
   d. SINON :
      - Le minimum est dans [m1, hi]
      - AUGMENTER lo à m1

2. PHASE FINALE : vérifier tous les éléments restants (2-3)
   a. RETOURNER celui avec la plus petite valeur

3. COMPLEXITÉ : O(log₃ n) itérations
```

### 5.3 Visualisation ASCII

**Ternary Search sur fonction unimodale:**
```
f(x)
  │
  │\
  │ \              /
  │  \            /
  │   \    MIN   /
  │    \   ↓    /
  │     \__*__/
  │
  └─────────────────── x
  lo    m1  m2    hi

Cas 1: f(m1) < f(m2)
  → Le minimum est entre lo et m2
  → On garde [lo, m2], on jette [m2, hi]

Cas 2: f(m1) > f(m2)
  → Le minimum est entre m1 et hi
  → On garde [m1, hi], on jette [lo, m1]
```

**Golden Section — Réutilisation des points:**
```
Itération 1:
lo────m1────m2────hi
      ↑     ↑
    calc  calc

Itération 2 (si f(m1) < f(m2)):
lo────m1───────m2
      ↑         ↑
    réutilisé  nouveau (était m1)

Le point m1 de l'itération 1 devient m2 de l'itération 2!
→ On économise un calcul de fonction par itération
```

**Rectangle inscrit dans demi-cercle:**
```
         ___________
        /           \
       /      y      \
      /───────────────\   ← hauteur y = √(r² - x²)
     /                 \
    /_______2x_________\  ← largeur 2x

    r = rayon du demi-cercle

    Aire = 2x × y = 2x × √(r² - x²)

    Maximum quand x = r/√2, y = r/√2
    → Aire max = r²
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **hi - lo > 2 vs > 3** | Mauvaise condition d'arrêt | Toujours vérifier les cas limites |
| **< vs >** | Confusion min/max | Écrire un commentaire clair |
| **Nombre d'itérations** | Précision insuffisante | 200 itérations = 10⁻⁶⁰ |
| **Ratio golden** | 1.5 au lieu de 1.618 | Utiliser constante définie |
| **Retour lo vs (lo+hi)/2** | Retourne borne pas centre | Toujours moyenner à la fin |

### 5.5 Cours Complet

#### 5.5.1 Pourquoi pas Binary Search ?

Binary search fonctionne sur une propriété **monotone** : à gauche d'un point, la condition est fausse ; à droite, elle est vraie.

Pour une fonction unimodale, on ne peut pas décider avec un seul point ! En testant f(mid), on ne sait pas si le minimum est à gauche ou à droite.

**Solution : Tester DEUX points**

En comparant f(m1) et f(m2), on peut éliminer un tiers de l'intervalle :
- Si f(m1) < f(m2), le minimum ne peut pas être dans [m2, hi]
- Si f(m1) > f(m2), le minimum ne peut pas être dans [lo, m1]

#### 5.5.2 L'algorithme Ternary Search

```rust
fn ternary_min(lo: i64, hi: i64, f: impl Fn(i64) -> i64) -> i64 {
    let mut lo = lo;
    let mut hi = hi;

    while hi - lo > 2 {
        let m1 = lo + (hi - lo) / 3;
        let m2 = hi - (hi - lo) / 3;

        if f(m1) < f(m2) {
            hi = m2;
        } else {
            lo = m1;
        }
    }

    // Vérifier les 2-3 candidats restants
    (lo..=hi).min_by_key(|&x| f(x)).unwrap()
}
```

**Complexité :** O(log₃ n) itérations, 2 évaluations par itération = O(2 log₃ n)

#### 5.5.3 Golden Section Search

Le nombre d'or φ = (1+√5)/2 ≈ 1.618 a une propriété magique :

```
φ² = φ + 1
```

Cela signifie qu'en divisant l'intervalle selon φ, un des points de test peut être **réutilisé** à l'itération suivante !

```
resphi = 2 - φ ≈ 0.382

m1 = hi - resphi * (hi - lo)
m2 = lo + resphi * (hi - lo)
```

**Avantage :** Une seule évaluation de fonction par itération au lieu de deux.

#### 5.5.4 Floating Point vs Discrete

Pour les entiers, on s'arrête quand l'intervalle contient ≤3 éléments.

Pour les flottants, on utilise :
1. Un nombre fixe d'itérations (e.g., 200 pour précision 10⁻⁶⁰)
2. Ou une condition `hi - lo < eps`

La combinaison des deux est recommandée pour robustesse.

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (fonctionne mais dangereux)                       │
├─────────────────────────────────────────────────────────────────┤
│ while hi - lo > eps { ... }  // Boucle potentiellement infinie  │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ for _ in 0..200 {                                               │
│     if hi - lo < eps { break; }                                 │
│     // ...                                                      │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ • Floating point peut ne jamais atteindre eps exactement        │
│ • Nombre d'itérations borné = programme qui termine toujours    │
│ • 200 itérations garantissent précision astronomique            │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**find_minimum_discrete((x-50)², 0, 100)**

```
┌───────┬────────────────────────┬──────┬──────┬────────┬────────┬───────────────┐
│ Iter  │ Intervalle             │  m1  │  m2  │ f(m1)  │ f(m2)  │ Décision      │
├───────┼────────────────────────┼──────┼──────┼────────┼────────┼───────────────┤
│   1   │ [0, 100]               │  33  │  66  │  289   │  256   │ f(m1)>f(m2)   │
│       │                        │      │      │        │        │ → [33, 100]   │
├───────┼────────────────────────┼──────┼──────┼────────┼────────┼───────────────┤
│   2   │ [33, 100]              │  55  │  77  │   25   │  729   │ f(m1)<f(m2)   │
│       │                        │      │      │        │        │ → [33, 77]    │
├───────┼────────────────────────┼──────┼──────┼────────┼────────┼───────────────┤
│   3   │ [33, 77]               │  47  │  62  │    9   │  144   │ f(m1)<f(m2)   │
│       │                        │      │      │        │        │ → [33, 62]    │
├───────┼────────────────────────┼──────┼──────┼────────┼────────┼───────────────┤
│  ...  │ Continue jusqu'à       │      │      │        │        │               │
│       │ hi - lo <= 2           │      │      │        │        │               │
├───────┼────────────────────────┼──────┼──────┼────────┼────────┼───────────────┤
│ Final │ [49, 51]               │      │      │        │        │ Check 49,50,51│
│       │ f(49)=1, f(50)=0       │      │      │        │        │ → Return 50   │
└───────┴────────────────────────┴──────┴──────┴────────┴────────┴───────────────┘
```

### 5.8 Mnémotechniques

#### 🔬 MEME : "The cake is a lie, but the minimum is real" — GLaDOS

Comme les promesses de gâteau de GLaDOS, les extrema locaux peuvent être trompeurs. Mais avec la recherche ternaire, on trouve le VRAI minimum.

```rust
// GLaDOS cherche l'optimum
let result = find_minimum(0, 100, |x| {
    let promise = cake_probability(x);  // Always 0
    let reality = pain_level(x);         // Unimodal!
    reality
});
// "Congratulations. The test is complete."
```

#### 🌀 MEME : "The Golden Ratio is everywhere" — Phi (φ)

```
φ = 1.6180339887...

Dans la nature :
🐚 Coquilles de nautile
🌻 Spirales de tournesol
🌀 Bras de galaxies

Dans ton code :
🔍 Golden Section Search
📐 Rectangle d'or
✨ Convergence élégante
```

**"When in doubt, use the golden ratio. For science!"**

### 5.9 Applications pratiques

| Domaine | Application |
|---------|-------------|
| **Graphics** | Placement de caméra, niveau de détail optimal |
| **Game Dev** | Pathfinding, collision detection |
| **ML** | Hyperparameter tuning, learning rate search |
| **Physics** | Minimum energy states, equilibrium |
| **Finance** | Portfolio optimization, risk minimization |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Détection |
|---|-------|--------|-----------|
| 1 | Condition > 2 vs > 3 | Manque le minimum | Test avec petits intervalles |
| 2 | Inversion min/max (</>)  | Trouve l'opposé | Test avec parabole connue |
| 3 | Pas assez d'itérations | Précision insuffisante | Test avec eps=10⁻⁹ |
| 4 | Mauvais ratio φ | Convergence lente | Comparer avec référence |
| 5 | Retourne lo pas (lo+hi)/2 | Erreur systématique | Test de précision |

---

## 📝 SECTION 7 : QCM

### Q1. Quand utiliser Ternary Search ?
A) Chercher une valeur dans un tableau trié
B) Trouver l'extremum d'une fonction unimodale
C) Trier un tableau
D) Compter les occurrences

**Réponse : B**

Ternary search est pour les fonctions unimodales (min ou max).

---

### Q2. Nombre d'évaluations par itération
Combien d'évaluations de fonction par itération pour ternary search ?

A) 1
B) 2
C) 3
D) log n

**Réponse : B**

On évalue f(m1) et f(m2) à chaque itération.

---

### Q3. Avantage du Golden Section
Quel est l'avantage principal ?

A) Plus rapide à écrire
B) Réutilise un point entre itérations
C) Fonctionne sur plus de fonctions
D) Pas besoin de bornes

**Réponse : B**

On réutilise un point, donc une seule nouvelle évaluation par itération.

---

### Q4. Valeur de φ (Golden Ratio)
Quelle est la valeur approximative de φ ?

A) 1.414
B) 1.5
C) 1.618
D) 2.0

**Réponse : C**

φ = (1 + √5) / 2 ≈ 1.618...

---

### Q5. Condition d'arrêt (discret)
On s'arrête quand...

A) hi - lo == 0
B) hi - lo <= 2
C) hi - lo <= 3
D) f(lo) == f(hi)

**Réponse : B**

Quand il reste 2-3 éléments, on les vérifie tous.

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.1.7 - aperture_optimizer |
| **Difficulté** | 5/10 (★★★★★☆☆☆☆☆) |
| **Fonctions** | 13 (search + applications) |
| **Complexité** | O(log n) itérations |
| **Bonus** | 🔥 Avancé (×3 XP) |
| **Points totaux** | 100 base + 50 bonus |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.1.7-aperture_optimizer",
    "generated_at": "2026-01-11T11:00:00Z",

    "metadata": {
      "exercise_id": "1.1.7",
      "exercise_name": "aperture_optimizer",
      "module": "1.1",
      "module_name": "Arrays & Sorting",
      "concept": "h",
      "concept_name": "Ternary Search & Unimodal Functions",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse (ternary + golden section + applications)",
      "phase": 1,
      "difficulty": 5,
      "difficulty_stars": "★★★★★☆☆☆☆☆",
      "language": "rust",
      "language_version": "edition_2024",
      "language_alt": "c17",
      "duration_minutes": 60,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "ADVANCED",
      "bonus_icon": "🔥",
      "complexity_time": "T2 O(log n)",
      "complexity_space": "S1 O(1)",
      "prerequisites": ["binary_search", "closures", "geometry_basics"],
      "domains": ["Tri", "Calcul", "MD"],
      "domains_bonus": ["Calcul", "MD"],
      "tags": ["ternary_search", "golden_section", "optimization", "unimodal"],
      "meme_reference": "Portal - The cake is a lie, but the minimum is real"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_aperture_optimizer.rs": "/* Section 4.3 */",
      "references/ref_aperture_optimizer.c": "/* C implementation */",
      "references/ref_bonus.rs": "/* Section 4.6 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_c_precision.rs": "/* Section 4.10 */",
      "mutants/mutant_d_golden.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
      "tests/main.rs": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_aperture_optimizer.rs",
        "references/ref_bonus.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_logic.rs",
        "mutants/mutant_c_precision.rs",
        "mutants/mutant_d_golden.rs",
        "mutants/mutant_e_return.rs"
      ]
    },

    "commands": {
      "validate_spec": "hackbrain-engine validate spec.json",
      "test_reference": "hackbrain-engine test -s spec.json -f references/ref_aperture_optimizer.rs",
      "test_mutants": "hackbrain-mutation-tester -r references/ref_aperture_optimizer.rs -s spec.json --validate"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "We do what we must, because we can... FOR SCIENCE!"*
*L'excellence pédagogique ne se négocie pas*
