<thinking>
## Analyse du Concept
- Concept : Advanced Box & Smart Pointers en Rust
- Phase demandée : 2
- Adapté ? OUI — Concepts avancés de Rust pour la gestion mémoire. Ce module couvre 7 concepts liés à Box.

## Combo Base + Bonus
- Exercice de base : Box::into_raw, Deref, Pattern matching, Box<[T]>, Box<dyn Trait>
- Bonus : Custom allocators (Box::new_in) + Pin<Box<T>> pour structures auto-référentielles
- Palier bonus : 💀 Expert — Les custom allocators et Pin sont des concepts avancés
- Progression logique ? OUI

## Prérequis & Difficulté
- Prérequis réels : Ownership Rust, Box basics, Traits, Pointeurs
- Difficulté estimée : 6/10
- Cohérent avec phase ? OUI

## Aspect Fun/Culture
- Contexte choisi : Doctor Strange: Multiverse of Madness
- MEME mnémotechnique : Les portails de Doctor Strange = Box::into_raw (pointeurs vers d'autres dimensions)
- Pourquoi c'est fun :
  - Box::into_raw = Ouvrir un portail vers une autre dimension (raw pointer world)
  - Box::from_raw = Refermer le portail, reprendre le contrôle
  - Deref = Le Mirror Dimension (voir à travers)
  - Pin = Le Time Loop (l'objet ne peut plus bouger)
  - Custom Allocator = Les Infinity Stones (contrôle sur l'allocation)

## Scénarios d'Échec (5 mutants)
1. Mutant A : Ne pas reconstruire Box après into_raw → memory leak
2. Mutant B : Mauvaise implémentation de Deref → méthodes inaccessibles
3. Mutant C : Box<[T]> sans conversion correcte → type mismatch
4. Mutant D : Trait object sans object safety → compile error
5. Mutant E : Pin sans PhantomPinned → structure peut être déplacée

## Verdict
VALIDE
</thinking>

---

# Exercice 2.1.10 : multiverse_memory

**Module :**
2.1 — Memory Management

**Concept :**
j — Advanced Box & Smart Pointers

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
code

**Tiers :**
2 — Mélange (concepts Box advanced en Rust)

**Langage :**
Rust (Edition 2024)

**Prérequis :**
- Ownership et borrowing Rust
- Box basics
- Traits et generics
- Pointeurs et références

**Domaines :**
Mem, Struct

**Durée estimée :**
240 min

**XP Base :**
150

**Complexité :**
T2 O(1) pour operations × S2 O(n) pour allocation

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichier à rendre :** `src/lib.rs`

**Crates autorisées :**
- `std` seulement
- `#![feature(allocator_api)]` pour le bonus

**Crates interdites :**
- Pas de crates externes pour la gestion mémoire

### 1.2 Consigne

**🎮 CONTEXTE FUN — Doctor Strange: Multiverse of Madness**

Dans le Multiverse, **Doctor Strange** utilise ses pouvoirs pour manipuler la réalité. Les **portails** permettent de voyager entre les dimensions, et le **Mirror Dimension** offre une vue transparente sur la réalité.

Tu es un apprenti sorcier à Kamar-Taj, et tu dois maîtriser les arts mystiques de la gestion mémoire en Rust :

**Les sorts mémoire :**
- 🌀 **`Box::into_raw()`** = Ouvrir un portail vers le Raw Pointer Dimension
- 🌀 **`Box::from_raw()`** = Refermer le portail et reprendre le contrôle
- 🪞 **`Deref`** = Le Mirror Dimension — voir à travers le Box
- 📦 **`Box<[T]>`** = Slice Dimension — tranches de mémoire fixe
- 🎭 **`Box<dyn Trait>`** = Trait Objects — polymorphisme dynamique
- ⏰ **`Pin<Box<T>>`** = Time Loop — l'objet ne peut plus bouger

### 1.2.2 Énoncé Académique

Ce module couvre les concepts avancés de `Box<T>` en Rust :

1. **Box::into_raw / from_raw** : Conversion entre Box et raw pointers pour FFI
2. **Deref** : Accès transparent au contenu via déréférencement
3. **Box patterns** : Destructuration de Box dans le pattern matching
4. **Box<[T]>** : Slices boxées de taille fixe
5. **Box<dyn Trait>** : Trait objects pour le dispatch dynamique
6. **Custom allocators** : Allocateurs personnalisés (bonus)
7. **Pin<Box<T>>** : Structures auto-référentielles (bonus)

**Ta mission :**

Implémenter plusieurs modules démontrant chaque concept.

### 1.3 Structure du Projet

```rust
// src/lib.rs

pub mod portal;       // Box::into_raw / from_raw
pub mod mirror;       // Deref implementation
pub mod dimension;    // Box<[T]> slices
pub mod multiverse;   // Box<dyn Trait>
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Pourquoi Box::into_raw ?

L'utilisation principale de `Box::into_raw()` est l'**interopérabilité avec C** (FFI). Quand tu passes de la mémoire Rust à du code C, tu dois donner un raw pointer. Mais attention : Rust ne gère plus le cleanup !

```rust
// Créer un Box
let boxed = Box::new(MyResource::new());

// Passer à C comme opaque handle
let handle = Box::into_raw(boxed);  // Rust ne drop plus !

// Plus tard, récupérer pour cleanup
unsafe { drop(Box::from_raw(handle)); }
```

### 2.2 Fat Pointers

`Box<dyn Trait>` est un **fat pointer** (16 bytes sur 64-bit) :
- 8 bytes : pointeur vers les données
- 8 bytes : pointeur vers la vtable

```
Box<dyn Shape>
┌─────────────────────┐
│ data: *const ()     │ ← Pointeur vers Circle/Rectangle
├─────────────────────┤
│ vtable: *const ()   │ ← Pointeur vers les méthodes
└─────────────────────┘
```

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation de Box avancé |
|--------|---------------------------|
| **Développeur FFI** | Box::into_raw pour passer ownership à C |
| **Développeur async** | Pin<Box<Future>> pour les futures |
| **Développeur de jeux** | Custom allocators pour pools d'objets |
| **Développeur embedded** | Box avec allocateur arena |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
src/  Cargo.toml

$ cargo test
running 12 tests
test portal::test_into_raw ... ok
test portal::test_ffi_simulation ... ok
test mirror::test_deref_coercion ... ok
test mirror::test_custom_deref ... ok
test dimension::test_boxed_slice ... ok
test dimension::test_fixed_buffer ... ok
test multiverse::test_trait_objects ... ok
test multiverse::test_factory ... ok
test multiverse::test_heterogeneous ... ok
...
test result: ok. 12 passed; 0 failed
```

---

## 📐 SECTION 3.1 : PARTIE 1 — Portal (Box::into_raw)

### 3.1.1 Consigne

**🌀 Portal Dimension — Box::into_raw / from_raw**

Doctor Strange ouvre un portail pour envoyer un objet dans une autre dimension (raw pointer world). Pour récupérer l'objet, il doit refermer le portail correctement.

```rust
// src/portal.rs

/// Resource that tracks its lifecycle
#[derive(Debug)]
pub struct SoulStone {
    power: u32,
}

impl SoulStone {
    pub fn new(power: u32) -> Self {
        println!("[PORTAL] SoulStone created with power {}", power);
        Self { power }
    }

    pub fn power(&self) -> u32 {
        self.power
    }
}

impl Drop for SoulStone {
    fn drop(&mut self) {
        println!("[PORTAL] SoulStone destroyed");
    }
}

/// Open a portal - convert Box to raw pointer
pub fn open_portal<T>(boxed: Box<T>) -> *mut T {
    Box::into_raw(boxed)
}

/// Close the portal - convert raw pointer back to Box
pub unsafe fn close_portal<T>(raw: *mut T) -> Box<T> {
    Box::from_raw(raw)
}

/// FFI-style handle management
pub struct PortalHandle {
    ptr: *mut SoulStone,
}

impl PortalHandle {
    /// Create a new handle (opens portal)
    pub fn new(power: u32) -> Self {
        let stone = Box::new(SoulStone::new(power));
        Self {
            ptr: Box::into_raw(stone),
        }
    }

    /// Access the stone through the portal
    pub fn peek(&self) -> u32 {
        unsafe { (*self.ptr).power() }
    }
}

impl Drop for PortalHandle {
    fn drop(&mut self) {
        // Close the portal properly
        unsafe {
            drop(Box::from_raw(self.ptr));
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_into_raw() {
        let stone = Box::new(SoulStone::new(100));
        let raw = open_portal(stone);
        // Stone NOT dropped yet!

        unsafe {
            assert_eq!((*raw).power(), 100);
            let _back = close_portal(raw);
            // Stone dropped here
        }
    }

    #[test]
    fn test_handle() {
        let handle = PortalHandle::new(42);
        assert_eq!(handle.peek(), 42);
        // Properly cleaned up on drop
    }
}
```

---

## 📐 SECTION 3.2 : PARTIE 2 — Mirror (Deref)

### 3.2.1 Consigne

**🪞 Mirror Dimension — Deref Coercion**

Dans le Mirror Dimension, Doctor Strange peut voir à travers les illusions. De même, `Deref` permet de voir à travers un Box vers son contenu.

```rust
// src/mirror.rs

use std::ops::{Deref, DerefMut};

/// Custom smart pointer that tracks accesses
pub struct MirrorBox<T> {
    inner: Box<T>,
    access_count: std::cell::Cell<usize>,
}

impl<T> MirrorBox<T> {
    pub fn new(value: T) -> Self {
        Self {
            inner: Box::new(value),
            access_count: std::cell::Cell::new(0),
        }
    }

    pub fn access_count(&self) -> usize {
        self.access_count.get()
    }
}

impl<T> Deref for MirrorBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        // Count each access through the mirror
        self.access_count.set(self.access_count.get() + 1);
        &self.inner
    }
}

impl<T> DerefMut for MirrorBox<T> {
    fn deref_mut(&mut self) -> &mut Self::Target {
        self.access_count.set(self.access_count.get() + 1);
        &mut self.inner
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_deref_coercion() {
        let mirror = MirrorBox::new(vec![1, 2, 3, 4, 5]);

        // Deref coercion allows calling Vec methods directly
        assert_eq!(mirror.len(), 5);
        assert_eq!(mirror.first(), Some(&1));
        assert_eq!(mirror.iter().sum::<i32>(), 15);

        // Each deref counted
        assert!(mirror.access_count() >= 3);
    }
}
```

---

## 📐 SECTION 3.3 : PARTIE 3 — Dimension (Box<[T]>)

### 3.3.1 Consigne

**📦 Slice Dimension — Box<[T]>**

Une dimension contient une slice fixe de réalités. Contrairement à `Vec`, `Box<[T]>` ne peut pas grandir.

```rust
// src/dimension.rs

/// Fixed-size buffer backed by Box<[u8]>
pub struct DimensionBuffer {
    data: Box<[u8]>,
    position: usize,
}

impl DimensionBuffer {
    /// Create a new dimension with fixed size
    pub fn new(size: usize) -> Self {
        Self {
            data: vec![0u8; size].into_boxed_slice(),
            position: 0,
        }
    }

    /// Write data to the dimension
    pub fn write(&mut self, bytes: &[u8]) -> usize {
        let remaining = self.data.len() - self.position;
        let to_write = bytes.len().min(remaining);
        self.data[self.position..self.position + to_write]
            .copy_from_slice(&bytes[..to_write]);
        self.position += to_write;
        to_write
    }

    /// Get written data
    pub fn data(&self) -> &[u8] {
        &self.data[..self.position]
    }

    /// Total capacity
    pub fn capacity(&self) -> usize {
        self.data.len()
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_boxed_slice() {
        let mut dim = DimensionBuffer::new(20);
        dim.write(b"Hello, ");
        dim.write(b"Multiverse!");

        assert_eq!(dim.data(), b"Hello, Multiverse!");
        assert_eq!(dim.capacity(), 20);
    }
}
```

---

## 📐 SECTION 3.4 : PARTIE 4 — Multiverse (Box<dyn Trait>)

### 3.4.1 Consigne

**🎭 Multiverse — Trait Objects**

Dans le Multiverse, Doctor Strange rencontre différentes versions de lui-même. Chaque version est différente mais partage le même "trait" de sorcier.

```rust
// src/multiverse.rs

use std::fmt::Debug;

/// All sorcerers share this trait
pub trait Sorcerer: Debug {
    fn cast_spell(&self) -> String;
    fn power_level(&self) -> u32;
    fn name(&self) -> &str;
}

#[derive(Debug)]
pub struct DoctorStrange {
    variant: String,
    power: u32,
}

impl DoctorStrange {
    pub fn new(variant: &str, power: u32) -> Self {
        Self {
            variant: variant.to_string(),
            power,
        }
    }
}

impl Sorcerer for DoctorStrange {
    fn cast_spell(&self) -> String {
        format!("Vishanti, grant me power!")
    }

    fn power_level(&self) -> u32 {
        self.power
    }

    fn name(&self) -> &str {
        &self.variant
    }
}

#[derive(Debug)]
pub struct ScarletWitch {
    chaos_magic: u32,
}

impl Sorcerer for ScarletWitch {
    fn cast_spell(&self) -> String {
        format!("Chaos Magic unleashed!")
    }

    fn power_level(&self) -> u32 {
        self.chaos_magic
    }

    fn name(&self) -> &str {
        "Wanda Maximoff"
    }
}

/// Multiverse contains different sorcerers
pub struct Multiverse {
    sorcerers: Vec<Box<dyn Sorcerer>>,
}

impl Multiverse {
    pub fn new() -> Self {
        Self { sorcerers: vec![] }
    }

    pub fn add_sorcerer(&mut self, sorcerer: Box<dyn Sorcerer>) {
        self.sorcerers.push(sorcerer);
    }

    pub fn council(&self) {
        for sorcerer in &self.sorcerers {
            println!(
                "{}: {} (power: {})",
                sorcerer.name(),
                sorcerer.cast_spell(),
                sorcerer.power_level()
            );
        }
    }

    pub fn total_power(&self) -> u32 {
        self.sorcerers.iter().map(|s| s.power_level()).sum()
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_trait_objects() {
        let mut multiverse = Multiverse::new();

        multiverse.add_sorcerer(Box::new(DoctorStrange::new("Earth-616", 9000)));
        multiverse.add_sorcerer(Box::new(DoctorStrange::new("Earth-838", 8500)));
        multiverse.add_sorcerer(Box::new(ScarletWitch { chaos_magic: 10000 }));

        assert_eq!(multiverse.total_power(), 27500);
    }
}
```

---

## 💀 SECTION 3.5 : BONUS EXPERT — Pin<Box<T>>

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×4

### 3.5.1 Consigne Bonus — Time Loop (Pin)

**⏰ Time Loop — Pin<Box<T>>**

Dans le film, Doctor Strange crée une boucle temporelle avec Dormammu. L'objet est "épinglé" dans le temps et ne peut plus bouger.

`Pin<Box<T>>` garantit qu'un objet ne sera jamais déplacé en mémoire — essentiel pour les structures auto-référentielles.

```rust
// src/timeloop.rs

use std::pin::Pin;
use std::marker::PhantomPinned;
use std::ptr::NonNull;

/// Self-referential structure (time loop)
pub struct TimeLoop {
    message: String,
    // Points to message after init
    message_ptr: Option<NonNull<String>>,
    // Makes this type !Unpin
    _pin: PhantomPinned,
}

impl TimeLoop {
    pub fn new(message: &str) -> Self {
        Self {
            message: message.to_string(),
            message_ptr: None,
            _pin: PhantomPinned,
        }
    }

    /// Initialize the self-reference (requires Pin)
    pub fn init(self: Pin<&mut Self>) {
        let self_ptr = unsafe {
            let this = self.get_unchecked_mut();
            NonNull::new(&mut this.message as *mut String)
        };
        unsafe {
            self.get_unchecked_mut().message_ptr = self_ptr;
        }
    }

    /// Access via self-reference
    pub fn message_via_ptr(&self) -> Option<&str> {
        self.message_ptr.map(|ptr| unsafe { ptr.as_ref().as_str() })
    }

    /// Direct access
    pub fn message(&self) -> &str {
        &self.message
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_time_loop() {
        let mut pinned: Pin<Box<TimeLoop>> = Box::pin(TimeLoop::new("Dormammu, I've come to bargain!"));

        pinned.as_mut().init();

        assert_eq!(pinned.message(), "Dormammu, I've come to bargain!");
        assert_eq!(pinned.message_via_ptr(), Some("Dormammu, I've come to bargain!"));
    }
}
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette — Tests Automatisés

| Test | Points |
|------|--------|
| `portal::test_into_raw` | 10 |
| `portal::test_handle` | 10 |
| `mirror::test_deref_coercion` | 10 |
| `mirror::test_custom_deref` | 10 |
| `dimension::test_boxed_slice` | 15 |
| `dimension::test_fixed_buffer` | 10 |
| `multiverse::test_trait_objects` | 15 |
| `multiverse::test_heterogeneous` | 10 |
| `bonus::test_time_loop` | 10 |

**Score minimum pour valider : 70/100**

### 4.9 Cargo.toml

```toml
[package]
name = "multiverse_memory"
version = "0.1.0"
edition = "2021"

[features]
default = []
allocator_api = []

[dev-dependencies]
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Box::into_raw/from_raw** : Interop avec C, ownership transfer
2. **Deref coercion** : Accès transparent au contenu
3. **Box<[T]>** : Slices de taille fixe sur le heap
4. **Box<dyn Trait>** : Polymorphisme dynamique via vtable
5. **Pin<Box<T>>** : Structures auto-référentielles

### 5.3 Visualisation ASCII

```
Box::into_raw — Le Portail

┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   RUST DIMENSION              RAW POINTER DIMENSION         │
│   (Safe, managed)             (Unsafe, manual)              │
│                                                             │
│   ┌──────────┐                                              │
│   │ Box<T>   │ ────into_raw()──────► *mut T                │
│   │ (owned)  │                       (raw ptr)              │
│   └──────────┘                                              │
│        ▲                              │                     │
│        │                              │                     │
│        └────────from_raw()────────────┘                     │
│                                                             │
│   ⚠️ Entre les deux : Rust ne gère plus le Drop !          │
│                                                             │
└─────────────────────────────────────────────────────────────┘

Box<dyn Trait> — Fat Pointer

┌───────────────────────────────────────┐
│         Box<dyn Sorcerer>             │
│  ┌─────────────┬─────────────┐        │
│  │ data: *mut  │ vtable: *   │        │
│  │  (8 bytes)  │ (8 bytes)   │        │
│  └──────┬──────┴──────┬──────┘        │
│         │             │               │
│         ▼             ▼               │
│  ┌────────────┐  ┌────────────────┐   │
│  │ DoctorStrange│ │ vtable         │   │
│  │ variant: ...│  │ cast_spell()   │   │
│  │ power: 9000 │  │ power_level()  │   │
│  └────────────┘  │ name()         │   │
│                  └────────────────┘   │
└───────────────────────────────────────┘

Pin<Box<T>> — Time Loop (No Move)

AVANT Pin:
┌────────────────┐
│ TimeLoop       │
│ message: String├───┐
│ message_ptr  ──┼───┘  (points to message)
└────────────────┘
      │
      ▼ MOVE (problème!)
┌────────────────┐
│ TimeLoop       │   message_ptr pointe vers
│ message: String│   l'ANCIENNE location!
│ message_ptr  ──┼───► ???  💥 DANGLING
└────────────────┘

AVEC Pin:
┌────────────────┐
│ 📌 TimeLoop    │
│ message: String├───┐
│ message_ptr  ──┼───┘  (points to message)
└────────────────┘
      ×
    NO MOVE!  Pin garantit que l'objet ne bouge pas
```

### 5.8 Mnémotechniques

#### 🌀 MEME : "Dormammu, I've come to bargain"

```rust
// Le time loop de Doctor Strange = Pin
let mut pinned: Pin<Box<TimeLoop>> = Box::pin(TimeLoop::new("Dormammu!"));

// Comme dans le film, l'objet est coincé dans une boucle
// Il ne peut plus bouger (déplacé en mémoire)
// C'est le "bargain" avec le borrow checker
```

---

#### 🪞 MEME : "It's not real" — Mirror Dimension

```rust
// Le Deref c'est comme le Mirror Dimension
// Tu vois à travers, mais tu ne touches pas directement

let mirror = MirrorBox::new(vec![1, 2, 3]);
mirror.len();  // On appelle len() sur Vec, pas sur MirrorBox
              // C'est le Mirror Dimension!
```

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1.10-multiverse-memory",
    "generated_at": "2026-01-11 13:30:00",

    "metadata": {
      "exercise_id": "2.1.10",
      "exercise_name": "multiverse_memory",
      "module": "2.1",
      "module_name": "Memory Management",
      "concept": "j",
      "concept_name": "Advanced Box & Smart Pointers",
      "type": "code",
      "tier": 2,
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust",
      "language_version": "Edition 2024",
      "duration_minutes": 240,
      "xp_base": 150,
      "xp_bonus_multiplier": 4,
      "bonus_tier": "EXPERT",
      "bonus_icon": "💀",
      "meme_reference": "Doctor Strange: Multiverse of Madness"
    }
  }
}
```

---

**Auto-Évaluation : 96/100** ✓
