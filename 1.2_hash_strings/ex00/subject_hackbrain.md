<thinking>
## Analyse du Concept
- Concept : Hash Tables avec multiples stratégies de collision + structures probabilistes
- Phase demandée : 1 (Intermédiaire)
- Adapté ? OUI - C'est un exercice de synthèse combinant tous les concepts de hashing

## Combo Base + Bonus
- Exercice de base : Implémenter 7 types de hash tables et structures probabilistes
- Bonus : Optimisations SIMD, perfect hashing, concurrent hash map
- Palier bonus : 🧠 Génie (complexité et optimisations avancées)
- Progression logique ? OUI - Base = implémentation fonctionnelle, Bonus = performance extrême

## Prérequis & Difficulté
- Prérequis réels : Notions de hashing, tableaux, pointeurs, génériques
- Difficulté estimée : 6/10 (base), 12/10 (bonus)
- Cohérent avec phase ? OUI - Phase 1 = 3-5/10, cet exercice synthèse peut aller à 6/10

## Aspect Fun/Culture
- Contexte choisi : Psycho-Pass (anime cyberpunk sur la prédiction probabiliste de crimes)
- MEME mnémotechnique : "Crime Coefficient Over 300" - Quand le load factor dépasse le seuil
- Pourquoi c'est fun :
  * Sibyl System = Hash table central (calcule les hash/coefficients)
  * Crime Coefficient = Hash value (valeur calculée pour chaque citoyen)
  * Latent Criminals = Faux positifs (Bloom filter)
  * Dominator Modes = Stratégies de collision (Paralyzer = chaining, Eliminator = probing)
  * Les Enforcers = Robin Hood (volent la place des autres)
  * Division 1 & 2 = Cuckoo hashing (deux tables)
  * Population monitoring = HyperLogLog (estimation de cardinalité)

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : `index = hash % capacity` sans vérifier capacity == 0 → division by zero
2. Mutant B (Safety) : Robin Hood swap sans clone → double free/use after move
3. Mutant C (Resize) : Rehash avec ancien hash au lieu de recalculer → éléments perdus
4. Mutant D (Logic) : Cuckoo évictions sans limite → boucle infinie
5. Mutant E (Math) : HyperLogLog avec mauvaise formule bias → estimation 10x erreur

## Verdict
VALIDE - Analogie Psycho-Pass parfaite pour hash tables probabilistes
Score créativité : 97/100
</thinking>

---

# Exercice 1.2.0-synth : sibyl_system

**Module :**
1.2 — Hash Tables & Strings

**Concept :**
synth — Synthèse Hash Tables et Structures Probabilistes

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (tous concepts hash tables)

**Langage :**
Rust Edition 2024 / C (c17)

**Prérequis :**
- Notions de hashing et fonctions de hash
- Tableaux dynamiques et allocation mémoire
- Génériques (Rust) / void* (C)
- Compréhension des probabilités (pour structures probabilistes)

**Domaines :**
Struct, Probas, Mem, Compression

**Durée estimée :**
180 min

**XP Base :**
200

**Complexité :**
T5 O(1) amortized × S4 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `sibyl_system.c`, `sibyl_system.h`

**Fonctions autorisées :**
- Rust : `std::hash`, `std::collections::hash_map::RandomState`, `std::alloc`
- C : `malloc`, `free`, `calloc`, `realloc`, `memcpy`, `memset`

**Fonctions interdites :**
- Rust : `std::collections::HashMap`, `std::collections::HashSet`
- C : Bibliothèques de hash externes

### 1.2 Consigne

#### 1.2.1 Version Culture Pop

**🎮 PSYCHO-PASS : Le Système Sibyl - Jugement Probabiliste**

*"Quand les yeux de Sibyl se posent sur vous, votre Crime Coefficient est calculé en un instant. Mais derrière cette façade d'omniscience se cache un système de hash tables interconnectées, chacune utilisant une stratégie différente pour résoudre les conflits de la société."*

Dans l'univers de **Psycho-Pass**, le Système Sibyl contrôle tout. Il calcule le **Crime Coefficient** de chaque citoyen - une valeur hash déterminant leur potentiel criminel. Mais le système n'est pas parfait : il utilise des **structures probabilistes** qui peuvent générer des **faux positifs** (des citoyens innocents classés comme "Latent Criminals").

Tu es recruté par le Bureau de Sécurité Publique pour reconstruire les composants internes du Système Sibyl :

**🔫 Le Dominator (Arme de jugement) :**
Le Dominator a plusieurs modes, comme nos hash tables ont plusieurs stratégies :
- **Mode Paralyseur** = `SibylChained` : Enchaîne les suspects dans des listes (chaining)
- **Mode Éliminateur** = `DominatorProbe` : Sonde linéairement jusqu'à trouver une place
- **Mode Décomposeur** = `EnforcerSquad` : Robin Hood - vole la place des autres si plus méritant

**👥 Divisions 1 & 2 (Cuckoo Hashing) :**
Comme les deux divisions du MWPSB, le Cuckoo Hashing maintient deux tables. Si un criminel ne peut pas être placé dans la Division 1, il est "coucou'd" vers la Division 2, poussant potentiellement quelqu'un d'autre à revenir.

**🎯 Structures de Prédiction Probabiliste :**
- **LatentDetector** (Bloom Filter) : "Est-ce un criminel latent ?" - Peut dire "peut-être oui" ou "définitivement non"
- **CrimeSketch** (Count-Min Sketch) : Compte approximativement les infractions par type
- **CityPopulation** (HyperLogLog) : Estime le nombre unique de criminels dans la ville

**Ta mission :**

Implémenter le Système Sibyl complet avec :

1. **`SibylChained<K, V>`** : Hash table avec chaînage séparé
2. **`DominatorProbe<K, V>`** : Hash table avec sondage linéaire
3. **`EnforcerSquad<K, V>`** : Hash table Robin Hood
4. **`CrimeDivision<K, V>`** : Cuckoo hashing (deux tables)
5. **`LatentDetector`** : Bloom Filter pour détection rapide
6. **`CrimeSketch`** : Count-Min Sketch pour comptage approximatif
7. **`CityPopulation`** : HyperLogLog pour estimation de cardinalité

**Entrée :**
- `key: K` : Clé hashable (ID du citoyen, type de crime, etc.)
- `value: V` : Valeur associée (Crime Coefficient, données, etc.)

**Sortie :**
- `Option<V>` pour les opérations CRUD
- `bool` pour les tests d'appartenance (Bloom Filter)
- `u64/f64` pour les estimations (Count-Min, HyperLogLog)

**Contraintes :**
┌─────────────────────────────────────────────────────────────────┐
│  Load Factor seuil : 0.75 (rehash automatique)                  │
│  Bloom Filter FP rate : configurable (défaut 1%)                │
│  HyperLogLog precision : 4-18 bits (défaut 14)                  │
│  Cuckoo max evictions : 500 avant rehash                        │
│  Robin Hood : stocker probe_distance avec chaque entrée         │
└─────────────────────────────────────────────────────────────────┘

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `sibyl.insert("Kogami", 120)` | `None` | Nouveau citoyen ajouté |
| `sibyl.insert("Kogami", 300)` | `Some(120)` | Coefficient mis à jour |
| `sibyl.get("Makishima")` | `None` | Citoyen non trouvé |
| `latent.contains("Suspect")` | `true` | Peut-être criminel (ou faux positif) |
| `hll.count()` | `9847.3` | ~10000 criminels uniques estimés |

#### 1.2.2 Version Académique

**Objectif :**

Implémenter une collection complète de structures de données basées sur le hashing, incluant plusieurs stratégies de résolution de collisions et des structures probabilistes.

**Structures requises :**

1. **Hash Table avec Chaînage Séparé**
   - Chaque bucket contient une liste de paires (clé, valeur)
   - Résolution de collision en O(n/m) moyen
   - Redimensionnement automatique basé sur le load factor

2. **Hash Table avec Sondage Linéaire**
   - Open addressing avec séquence de sondage h(k) + i
   - Gestion des tombstones pour les suppressions
   - Clustering primaire comme inconvénient

3. **Robin Hood Hashing**
   - Open addressing avec redistribution
   - Chaque élément stocke sa probe distance
   - Swap si nouvel élément a une probe distance plus grande
   - Réduit la variance des temps de recherche

4. **Cuckoo Hashing**
   - Deux tables avec deux fonctions de hash différentes
   - Insertion O(1) worst case (amorti)
   - Évictions en chaîne jusqu'à placement ou rehash
   - Lookup O(1) garanti (max 2 accès)

5. **Bloom Filter**
   - Ensemble approximatif avec faux positifs possibles
   - Taille optimale : m = -n*ln(p) / (ln(2)^2)
   - Nombre de hash : k = (m/n) * ln(2)

6. **Count-Min Sketch**
   - Comptage approximatif de fréquences
   - Matrice depth × width avec d fonctions de hash
   - Estimation = minimum des compteurs

7. **HyperLogLog**
   - Estimation de cardinalité en O(1) espace
   - Utilise le rang du premier bit 1 dans le hash
   - Formule : E = alpha_m * m^2 / sum(2^(-M[j]))

**Comportements attendus :**
- Insertion : Ajoute ou met à jour
- Recherche : O(1) moyen pour toutes les structures
- Suppression : Support complet (sauf Bloom Filter)
- Itération : Pour les hash tables standard

### 1.3 Prototype

**Rust :**
```rust
// ═══════════════════════════════════════════════════════════════
// SIBYL SYSTEM - Hash Table avec Chaînage
// ═══════════════════════════════════════════════════════════════

use std::hash::{Hash, Hasher, BuildHasher};
use std::collections::hash_map::RandomState;

/// Hash table avec chaînage séparé (Mode Paralyseur)
pub struct SibylChained<K, V, S = RandomState> {
    buckets: Vec<Vec<(K, V)>>,
    len: usize,
    hash_builder: S,
}

impl<K: Hash + Eq, V> SibylChained<K, V> {
    pub fn new() -> Self;
    pub fn with_capacity(capacity: usize) -> Self;
    pub fn insert(&mut self, key: K, value: V) -> Option<V>;
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn get_mut(&mut self, key: &K) -> Option<&mut V>;
    pub fn remove(&mut self, key: &K) -> Option<V>;
    pub fn contains_key(&self, key: &K) -> bool;
    pub fn len(&self) -> usize;
    pub fn is_empty(&self) -> bool;
    pub fn load_factor(&self) -> f64;
    pub fn iter(&self) -> impl Iterator<Item = (&K, &V)>;
}

// ═══════════════════════════════════════════════════════════════
// DOMINATOR PROBE - Sondage Linéaire
// ═══════════════════════════════════════════════════════════════

/// Hash table avec sondage linéaire (Mode Éliminateur)
pub struct DominatorProbe<K, V> {
    slots: Vec<Option<(K, V)>>,
    tombstones: Vec<bool>,
    len: usize,
    capacity: usize,
}

impl<K: Hash + Eq, V> DominatorProbe<K, V> {
    pub fn new() -> Self;
    pub fn with_capacity(capacity: usize) -> Self;
    pub fn insert(&mut self, key: K, value: V) -> Option<V>;
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn remove(&mut self, key: &K) -> Option<V>;
    pub fn len(&self) -> usize;
    pub fn is_empty(&self) -> bool;
}

// ═══════════════════════════════════════════════════════════════
// ENFORCER SQUAD - Robin Hood Hashing
// ═══════════════════════════════════════════════════════════════

/// Robin Hood hashing (Les Enforcers - volent la place des riches)
pub struct EnforcerSquad<K, V> {
    slots: Vec<Option<(K, V, usize)>>,  // (key, value, probe_distance)
    len: usize,
    capacity: usize,
}

impl<K: Hash + Eq, V> EnforcerSquad<K, V> {
    pub fn new() -> Self;
    pub fn with_capacity(capacity: usize) -> Self;
    pub fn insert(&mut self, key: K, value: V) -> Option<V>;
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn remove(&mut self, key: &K) -> Option<V>;
    pub fn average_probe_distance(&self) -> f64;
    pub fn max_probe_distance(&self) -> usize;
}

// ═══════════════════════════════════════════════════════════════
// CRIME DIVISION - Cuckoo Hashing
// ═══════════════════════════════════════════════════════════════

/// Cuckoo hashing (Division 1 & 2 du MWPSB)
pub struct CrimeDivision<K, V> {
    division1: Vec<Option<(K, V)>>,
    division2: Vec<Option<(K, V)>>,
    len: usize,
    capacity: usize,
}

impl<K: Hash + Eq + Clone, V: Clone> CrimeDivision<K, V> {
    pub fn new(capacity: usize) -> Self;
    pub fn insert(&mut self, key: K, value: V) -> Result<Option<V>, (K, V)>;
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn remove(&mut self, key: &K) -> Option<V>;
    pub fn len(&self) -> usize;
}

// ═══════════════════════════════════════════════════════════════
// LATENT DETECTOR - Bloom Filter
// ═══════════════════════════════════════════════════════════════

/// Bloom Filter (Détecteur de Criminels Latents)
pub struct LatentDetector {
    bits: Vec<bool>,
    num_hashes: usize,
    num_items: usize,
}

impl LatentDetector {
    pub fn new(capacity: usize) -> Self;
    pub fn with_fp_rate(capacity: usize, fp_rate: f64) -> Self;
    pub fn insert<T: Hash>(&mut self, item: &T);
    pub fn contains<T: Hash>(&self, item: &T) -> bool;
    pub fn estimated_fp_rate(&self) -> f64;
    pub fn clear(&mut self);
}

// ═══════════════════════════════════════════════════════════════
// CRIME SKETCH - Count-Min Sketch
// ═══════════════════════════════════════════════════════════════

/// Count-Min Sketch (Compteur de Crimes par Type)
pub struct CrimeSketch {
    table: Vec<Vec<u64>>,
    width: usize,
    depth: usize,
}

impl CrimeSketch {
    pub fn new(width: usize, depth: usize) -> Self;
    pub fn with_accuracy(epsilon: f64, delta: f64) -> Self;
    pub fn add<T: Hash>(&mut self, item: &T, count: u64);
    pub fn increment<T: Hash>(&mut self, item: &T);
    pub fn estimate<T: Hash>(&self, item: &T) -> u64;
}

// ═══════════════════════════════════════════════════════════════
// CITY POPULATION - HyperLogLog
// ═══════════════════════════════════════════════════════════════

/// HyperLogLog (Estimation de Population Criminelle)
pub struct CityPopulation {
    registers: Vec<u8>,
    precision: usize,
}

impl CityPopulation {
    pub fn new(precision: usize) -> Self;
    pub fn add<T: Hash>(&mut self, item: &T);
    pub fn count(&self) -> f64;
    pub fn merge(&mut self, other: &Self);
    pub fn clear(&mut self);
}
```

**C :**
```c
#ifndef SIBYL_SYSTEM_H
#define SIBYL_SYSTEM_H

#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>

// ═══════════════════════════════════════════════════════════════
// SIBYL CHAINED - Hash Table avec Chaînage
// ═══════════════════════════════════════════════════════════════

typedef struct s_chain_node {
    void            *key;
    void            *value;
    struct s_chain_node *next;
} t_chain_node;

typedef struct s_sibyl_chained {
    t_chain_node    **buckets;
    size_t          capacity;
    size_t          len;
    size_t          (*hash_fn)(const void *);
    int             (*eq_fn)(const void *, const void *);
    void            (*key_free)(void *);
    void            (*value_free)(void *);
} t_sibyl_chained;

t_sibyl_chained *sibyl_chained_new(
    size_t capacity,
    size_t (*hash_fn)(const void *),
    int (*eq_fn)(const void *, const void *)
);
void    sibyl_chained_destroy(t_sibyl_chained *table);
void    *sibyl_chained_insert(t_sibyl_chained *table, void *key, void *value);
void    *sibyl_chained_get(t_sibyl_chained *table, const void *key);
void    *sibyl_chained_remove(t_sibyl_chained *table, const void *key);
bool    sibyl_chained_contains(t_sibyl_chained *table, const void *key);
size_t  sibyl_chained_len(t_sibyl_chained *table);
double  sibyl_chained_load_factor(t_sibyl_chained *table);

// ═══════════════════════════════════════════════════════════════
// DOMINATOR PROBE - Sondage Linéaire
// ═══════════════════════════════════════════════════════════════

typedef struct s_probe_slot {
    void    *key;
    void    *value;
    bool    occupied;
    bool    tombstone;
} t_probe_slot;

typedef struct s_dominator_probe {
    t_probe_slot    *slots;
    size_t          capacity;
    size_t          len;
    size_t          (*hash_fn)(const void *);
    int             (*eq_fn)(const void *, const void *);
} t_dominator_probe;

t_dominator_probe   *dominator_probe_new(
    size_t capacity,
    size_t (*hash_fn)(const void *),
    int (*eq_fn)(const void *, const void *)
);
void    dominator_probe_destroy(t_dominator_probe *table);
void    *dominator_probe_insert(t_dominator_probe *table, void *key, void *value);
void    *dominator_probe_get(t_dominator_probe *table, const void *key);
void    *dominator_probe_remove(t_dominator_probe *table, const void *key);
size_t  dominator_probe_len(t_dominator_probe *table);

// ═══════════════════════════════════════════════════════════════
// ENFORCER SQUAD - Robin Hood Hashing
// ═══════════════════════════════════════════════════════════════

typedef struct s_robin_slot {
    void    *key;
    void    *value;
    size_t  probe_distance;
    bool    occupied;
} t_robin_slot;

typedef struct s_enforcer_squad {
    t_robin_slot    *slots;
    size_t          capacity;
    size_t          len;
    size_t          (*hash_fn)(const void *);
    int             (*eq_fn)(const void *, const void *);
} t_enforcer_squad;

t_enforcer_squad    *enforcer_squad_new(
    size_t capacity,
    size_t (*hash_fn)(const void *),
    int (*eq_fn)(const void *, const void *)
);
void    enforcer_squad_destroy(t_enforcer_squad *table);
void    *enforcer_squad_insert(t_enforcer_squad *table, void *key, void *value);
void    *enforcer_squad_get(t_enforcer_squad *table, const void *key);
void    *enforcer_squad_remove(t_enforcer_squad *table, const void *key);
double  enforcer_squad_avg_probe(t_enforcer_squad *table);

// ═══════════════════════════════════════════════════════════════
// CRIME DIVISION - Cuckoo Hashing
// ═══════════════════════════════════════════════════════════════

typedef struct s_cuckoo_slot {
    void    *key;
    void    *value;
    bool    occupied;
} t_cuckoo_slot;

typedef struct s_crime_division {
    t_cuckoo_slot   *division1;
    t_cuckoo_slot   *division2;
    size_t          capacity;
    size_t          len;
    size_t          (*hash1)(const void *);
    size_t          (*hash2)(const void *);
    int             (*eq_fn)(const void *, const void *);
} t_crime_division;

t_crime_division    *crime_division_new(
    size_t capacity,
    size_t (*hash1)(const void *),
    size_t (*hash2)(const void *),
    int (*eq_fn)(const void *, const void *)
);
void    crime_division_destroy(t_crime_division *table);
int     crime_division_insert(t_crime_division *table, void *key, void *value, void **old_value);
void    *crime_division_get(t_crime_division *table, const void *key);
void    *crime_division_remove(t_crime_division *table, const void *key);
size_t  crime_division_len(t_crime_division *table);

// ═══════════════════════════════════════════════════════════════
// LATENT DETECTOR - Bloom Filter
// ═══════════════════════════════════════════════════════════════

typedef struct s_latent_detector {
    uint8_t *bits;
    size_t  bit_size;
    size_t  num_hashes;
    size_t  num_items;
} t_latent_detector;

t_latent_detector   *latent_detector_new(size_t capacity);
t_latent_detector   *latent_detector_with_fp(size_t capacity, double fp_rate);
void    latent_detector_destroy(t_latent_detector *filter);
void    latent_detector_insert(t_latent_detector *filter, const void *item, size_t size);
bool    latent_detector_contains(t_latent_detector *filter, const void *item, size_t size);
double  latent_detector_fp_rate(t_latent_detector *filter);
void    latent_detector_clear(t_latent_detector *filter);

// ═══════════════════════════════════════════════════════════════
// CRIME SKETCH - Count-Min Sketch
// ═══════════════════════════════════════════════════════════════

typedef struct s_crime_sketch {
    uint64_t    **table;
    size_t      width;
    size_t      depth;
} t_crime_sketch;

t_crime_sketch  *crime_sketch_new(size_t width, size_t depth);
t_crime_sketch  *crime_sketch_with_accuracy(double epsilon, double delta);
void    crime_sketch_destroy(t_crime_sketch *sketch);
void    crime_sketch_add(t_crime_sketch *sketch, const void *item, size_t size, uint64_t count);
void    crime_sketch_increment(t_crime_sketch *sketch, const void *item, size_t size);
uint64_t    crime_sketch_estimate(t_crime_sketch *sketch, const void *item, size_t size);

// ═══════════════════════════════════════════════════════════════
// CITY POPULATION - HyperLogLog
// ═══════════════════════════════════════════════════════════════

typedef struct s_city_population {
    uint8_t *registers;
    size_t  num_registers;
    size_t  precision;
} t_city_population;

t_city_population   *city_population_new(size_t precision);
void    city_population_destroy(t_city_population *hll);
void    city_population_add(t_city_population *hll, const void *item, size_t size);
double  city_population_count(t_city_population *hll);
void    city_population_merge(t_city_population *dst, const t_city_population *src);
void    city_population_clear(t_city_population *hll);

// ═══════════════════════════════════════════════════════════════
// HASH FUNCTIONS UTILITAIRES
// ═══════════════════════════════════════════════════════════════

size_t  fnv1a_hash(const void *data, size_t size);
size_t  murmur3_hash(const void *data, size_t size, uint32_t seed);
size_t  xxhash(const void *data, size_t size);

#endif // SIBYL_SYSTEM_H
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Anecdote Historique

Le **Robin Hood Hashing** a été inventé par Pedro Celis en 1986 dans sa thèse de doctorat. Le nom vient de la légende de Robin des Bois : "voler aux riches pour donner aux pauvres". Dans cette variante, un nouvel élément avec une grande probe distance (le "pauvre") peut voler la place d'un élément avec une petite probe distance (le "riche").

Le **Cuckoo Hashing** (2001, Pagh et Rodler) tire son nom du coucou, l'oiseau qui pond ses oeufs dans les nids d'autres oiseaux, poussant leurs oeufs dehors - exactement ce que fait cette structure de données lors des évictions.

### 2.2 Chiffre Clé

- **Load Factor optimal** : 0.7 pour le chaînage, 0.5 pour le probing linéaire
- **HyperLogLog** : Estime des milliards d'éléments uniques avec seulement **12 KB de mémoire**
- **Bloom Filter** : Utilisé par Google Chrome pour vérifier 500M+ URLs malveillantes instantanément

### 2.3 Culture Geek

Le Système Sibyl de Psycho-Pass ressemble étrangement à un système de scoring de crédit social réel... avec des hash tables en backend pour la recherche O(1) des coefficients de chaque citoyen.

### 2.5 Dans la Vraie Vie

| Métier | Utilisation |
|--------|-------------|
| **Data Engineer** | HyperLogLog pour compter les visiteurs uniques sans stocker tous les IDs |
| **Security Engineer** | Bloom Filters pour vérifier si un mot de passe est dans une liste de leaks |
| **Database Developer** | Cuckoo Hashing pour des indexes avec lookup O(1) garanti |
| **Network Engineer** | Count-Min Sketch pour détecter les attaques DDoS (comptage de flux) |
| **Backend Developer** | Robin Hood pour des hash tables avec latence prévisible |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
sibyl_system.rs  main.rs  Cargo.toml

$ cargo build --release

$ cargo test
running 12 tests
test test_sibyl_chained ... ok
test test_dominator_probe ... ok
test test_enforcer_squad ... ok
test test_crime_division ... ok
test test_latent_detector ... ok
test test_crime_sketch ... ok
test test_city_population ... ok
test test_load_factor ... ok
test test_rehash ... ok
test test_bloom_fp_rate ... ok
test test_hll_accuracy ... ok
test test_robin_hood_variance ... ok

test result: ok. 12 passed; 0 failed

$ ./target/release/sibyl_demo
=== SIBYL SYSTEM ACTIVATED ===
Crime Coefficient Database: 1000 citizens loaded
Average probe distance (Robin Hood): 1.23
Latent Criminals detected (Bloom Filter): 42 (2 false positives)
Unique criminals estimated (HyperLogLog): 9847 (actual: 10000, error: 1.53%)
System Sibyl: Operational
```

### 3.1 🧠 BONUS GÉNIE (OPTIONNEL)

**Difficulté Bonus :**
🧠 (12/10)

**Récompense :**
XP ×6

**Time Complexity attendue :**
O(1) avec optimisations SIMD

**Space Complexity attendue :**
O(n) avec compression

**Domaines Bonus :**
`CPU, ASM`

#### 3.1.1 Consigne Bonus

**🎮 SIBYL 2.0 : La Mise à Jour du Système**

*"Le Bureau a décidé de moderniser le Système Sibyl. Les nouveaux Dominators doivent pouvoir traiter des millions de citoyens en temps réel, avec des garanties de latence strictes."*

**Ta mission avancée :**

1. **`SibylSimd`** : Hash table utilisant SIMD pour le lookup parallèle
   - Comparer 4/8/16 clés simultanément
   - Utiliser AVX2/AVX-512 si disponible

2. **`PerfectJudgement`** : Perfect Hashing pour données statiques
   - Construire une fonction de hash parfaite
   - Lookup O(1) garanti sans collision

3. **`ConcurrentSibyl`** : Hash table lock-free
   - Utiliser compare-and-swap (CAS)
   - Support multi-threaded sans mutex

4. **`CompressedPopulation`** : HyperLogLog avec sparse representation
   - Économie de 90%+ mémoire pour petites cardinalités
   - Switch automatique vers dense quand nécessaire

**Contraintes Bonus :**
┌─────────────────────────────────────────────────────────────────┐
│  SIMD : Support AVX2 minimum, fallback scalar                   │
│  Concurrent : Pas de mutex, CAS uniquement                      │
│  Perfect Hash : Temps de construction O(n), lookup O(1)         │
│  Memory : HyperLogLog sparse < 1KB pour n < 1000                │
└─────────────────────────────────────────────────────────────────┘

#### 3.1.2 Prototype Bonus

```rust
use std::sync::atomic::{AtomicU64, AtomicPtr, Ordering};

/// SIMD-accelerated hash table lookup
#[cfg(target_arch = "x86_64")]
pub struct SibylSimd<K, V> {
    keys: Vec<K>,
    values: Vec<V>,
    hashes: Vec<u64>,  // Pre-computed hashes for SIMD comparison
}

impl<K: Hash + Eq, V> SibylSimd<K, V> {
    pub fn new() -> Self;
    pub fn insert(&mut self, key: K, value: V) -> Option<V>;
    pub fn get(&self, key: &K) -> Option<&V>;  // Uses SIMD internally
    pub fn batch_lookup(&self, keys: &[K]) -> Vec<Option<&V>>;  // Parallel lookup
}

/// Perfect hash function for static data
pub struct PerfectJudgement<K, V> {
    g: Vec<u32>,  // Intermediate hash values
    values: Vec<Option<V>>,
}

impl<K: Hash + Eq, V> PerfectJudgement<K, V> {
    pub fn build(items: Vec<(K, V)>) -> Self;  // O(n) construction
    pub fn get(&self, key: &K) -> Option<&V>;  // O(1) guaranteed
}

/// Lock-free concurrent hash table
pub struct ConcurrentSibyl<K, V> {
    buckets: Vec<AtomicPtr<(K, V)>>,
    len: AtomicU64,
}

impl<K: Hash + Eq + Clone, V: Clone> ConcurrentSibyl<K, V> {
    pub fn new() -> Self;
    pub fn insert(&self, key: K, value: V) -> Option<V>;  // Lock-free
    pub fn get(&self, key: &K) -> Option<V>;  // Lock-free
}

/// Sparse-Dense HyperLogLog
pub struct CompressedPopulation {
    sparse: Option<Vec<(u32, u8)>>,  // (register_index, value) pairs
    dense: Option<Vec<u8>>,
    precision: usize,
    threshold: usize,
}

impl CompressedPopulation {
    pub fn new(precision: usize) -> Self;
    pub fn add<T: Hash>(&mut self, item: &T);  // Auto-switches sparse->dense
    pub fn count(&self) -> f64;
    pub fn memory_usage(&self) -> usize;
}
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Lookup | O(1) scalaire | O(1) SIMD parallèle |
| Perfect Hash | Non | Oui, O(1) garanti |
| Concurrence | Non | Lock-free CAS |
| HyperLogLog | Dense only | Sparse+Dense adaptatif |
| Complexité | ~500 lignes | ~1500 lignes |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Entrée | Sortie Attendue | Points |
|------|--------|-----------------|--------|
| `sibyl_insert_get` | Insert "A"→1, get "A" | `Some(1)` | 2 |
| `sibyl_update` | Insert "A"→1, Insert "A"→2 | `Some(1)`, get→`Some(2)` | 2 |
| `sibyl_remove` | Insert, remove, get | `None` | 2 |
| `sibyl_rehash` | Insert 1000 elements | `load_factor() < 0.8` | 3 |
| `probe_basic` | Insert/get/remove | Correct values | 3 |
| `probe_tombstone` | Remove, insert same hash | Works correctly | 3 |
| `robin_insert` | Insert 1000 elements | `avg_probe_distance() < 3` | 3 |
| `robin_variance` | Insert 1000 elements | Low variance | 3 |
| `cuckoo_basic` | Insert 30 elements | All retrievable | 3 |
| `cuckoo_eviction` | Force evictions | Completes or errors | 3 |
| `bloom_no_fn` | Check non-inserted | `false` for all | 3 |
| `bloom_fp_rate` | Insert 1000, check 1000 others | FP < 5% | 3 |
| `cms_accuracy` | Add counts, estimate | Within epsilon | 3 |
| `hll_accuracy` | Add 10000 unique | Error < 2% | 3 |
| `stress_test` | 100000 operations | No crash, correct | 5 |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <string.h>
#include <assert.h>
#include <stdlib.h>
#include <time.h>
#include "sibyl_system.h"

// Simple string hash function
size_t str_hash(const void *key) {
    const char *str = (const char *)key;
    size_t hash = 5381;
    int c;
    while ((c = *str++))
        hash = ((hash << 5) + hash) + c;
    return hash;
}

// String equality
int str_eq(const void *a, const void *b) {
    return strcmp((const char *)a, (const char *)b) == 0;
}

void test_sibyl_chained(void) {
    printf("Testing SibylChained...\n");

    t_sibyl_chained *table = sibyl_chained_new(16, str_hash, str_eq);
    assert(table != NULL);

    // Insert
    char *key1 = strdup("Kogami");
    int *val1 = malloc(sizeof(int)); *val1 = 120;
    void *old = sibyl_chained_insert(table, key1, val1);
    assert(old == NULL);

    // Get
    int *retrieved = (int *)sibyl_chained_get(table, "Kogami");
    assert(retrieved != NULL && *retrieved == 120);

    // Update
    int *val2 = malloc(sizeof(int)); *val2 = 300;
    old = sibyl_chained_insert(table, strdup("Kogami"), val2);
    assert(old != NULL && *(int *)old == 120);
    free(old);

    // Verify update
    retrieved = (int *)sibyl_chained_get(table, "Kogami");
    assert(retrieved != NULL && *retrieved == 300);

    // Contains
    assert(sibyl_chained_contains(table, "Kogami") == true);
    assert(sibyl_chained_contains(table, "Makishima") == false);

    // Remove
    void *removed = sibyl_chained_remove(table, "Kogami");
    assert(removed != NULL && *(int *)removed == 300);
    free(removed);
    assert(sibyl_chained_get(table, "Kogami") == NULL);

    sibyl_chained_destroy(table);
    printf("  PASS\n");
}

void test_bloom_filter(void) {
    printf("Testing LatentDetector (Bloom Filter)...\n");

    t_latent_detector *bloom = latent_detector_with_fp(1000, 0.01);
    assert(bloom != NULL);

    // Insert items
    for (int i = 0; i < 1000; i++) {
        latent_detector_insert(bloom, &i, sizeof(i));
    }

    // All inserted items should be found
    for (int i = 0; i < 1000; i++) {
        assert(latent_detector_contains(bloom, &i, sizeof(i)) == true);
    }

    // Count false positives
    int fp = 0;
    for (int i = 1000; i < 2000; i++) {
        if (latent_detector_contains(bloom, &i, sizeof(i))) {
            fp++;
        }
    }

    double fp_rate = (double)fp / 1000.0;
    printf("  False positive rate: %.2f%% (expected ~1%%)\n", fp_rate * 100);
    assert(fp_rate < 0.05);  // Allow up to 5%

    latent_detector_destroy(bloom);
    printf("  PASS\n");
}

void test_hyperloglog(void) {
    printf("Testing CityPopulation (HyperLogLog)...\n");

    t_city_population *hll = city_population_new(14);
    assert(hll != NULL);

    // Add 10000 unique items
    for (int i = 0; i < 10000; i++) {
        city_population_add(hll, &i, sizeof(i));
    }

    double estimate = city_population_count(hll);
    double error = fabs(estimate - 10000.0) / 10000.0;

    printf("  Estimate: %.0f (actual: 10000, error: %.2f%%)\n", estimate, error * 100);
    assert(error < 0.03);  // Within 3%

    city_population_destroy(hll);
    printf("  PASS\n");
}

int main(void) {
    printf("=== SIBYL SYSTEM TEST SUITE ===\n\n");

    test_sibyl_chained();
    test_bloom_filter();
    test_hyperloglog();
    // Add more tests...

    printf("\n=== ALL TESTS PASSED ===\n");
    return 0;
}
```

### 4.3 Solution de référence

```rust
use std::hash::{Hash, Hasher, BuildHasher};
use std::collections::hash_map::{DefaultHasher, RandomState};

// ═══════════════════════════════════════════════════════════════
// SIBYL CHAINED - Solution de Référence
// ═══════════════════════════════════════════════════════════════

pub struct SibylChained<K, V, S = RandomState> {
    buckets: Vec<Vec<(K, V)>>,
    len: usize,
    hash_builder: S,
}

impl<K: Hash + Eq, V> SibylChained<K, V> {
    const DEFAULT_CAPACITY: usize = 16;
    const MAX_LOAD_FACTOR: f64 = 0.75;

    pub fn new() -> Self {
        Self::with_capacity(Self::DEFAULT_CAPACITY)
    }

    pub fn with_capacity(capacity: usize) -> Self {
        let capacity = capacity.max(1);
        SibylChained {
            buckets: (0..capacity).map(|_| Vec::new()).collect(),
            len: 0,
            hash_builder: RandomState::new(),
        }
    }

    fn hash(&self, key: &K) -> usize {
        let mut hasher = self.hash_builder.build_hasher();
        key.hash(&mut hasher);
        hasher.finish() as usize
    }

    fn bucket_index(&self, key: &K) -> usize {
        self.hash(key) % self.buckets.len()
    }

    fn maybe_resize(&mut self) {
        if self.load_factor() > Self::MAX_LOAD_FACTOR {
            let new_capacity = self.buckets.len() * 2;
            let old_buckets = std::mem::replace(
                &mut self.buckets,
                (0..new_capacity).map(|_| Vec::new()).collect(),
            );
            self.len = 0;

            for bucket in old_buckets {
                for (k, v) in bucket {
                    self.insert(k, v);
                }
            }
        }
    }

    pub fn insert(&mut self, key: K, value: V) -> Option<V> {
        self.maybe_resize();

        let idx = self.bucket_index(&key);
        let bucket = &mut self.buckets[idx];

        for (k, v) in bucket.iter_mut() {
            if k == &key {
                return Some(std::mem::replace(v, value));
            }
        }

        bucket.push((key, value));
        self.len += 1;
        None
    }

    pub fn get(&self, key: &K) -> Option<&V> {
        let idx = self.bucket_index(key);
        self.buckets[idx]
            .iter()
            .find(|(k, _)| k == key)
            .map(|(_, v)| v)
    }

    pub fn get_mut(&mut self, key: &K) -> Option<&mut V> {
        let idx = self.bucket_index(key);
        self.buckets[idx]
            .iter_mut()
            .find(|(k, _)| k == key)
            .map(|(_, v)| v)
    }

    pub fn remove(&mut self, key: &K) -> Option<V> {
        let idx = self.bucket_index(key);
        let bucket = &mut self.buckets[idx];

        if let Some(pos) = bucket.iter().position(|(k, _)| k == key) {
            self.len -= 1;
            Some(bucket.swap_remove(pos).1)
        } else {
            None
        }
    }

    pub fn contains_key(&self, key: &K) -> bool {
        self.get(key).is_some()
    }

    pub fn len(&self) -> usize {
        self.len
    }

    pub fn is_empty(&self) -> bool {
        self.len == 0
    }

    pub fn load_factor(&self) -> f64 {
        self.len as f64 / self.buckets.len() as f64
    }

    pub fn iter(&self) -> impl Iterator<Item = (&K, &V)> {
        self.buckets.iter().flatten().map(|(k, v)| (k, v))
    }
}

// ═══════════════════════════════════════════════════════════════
// ENFORCER SQUAD - Robin Hood (Solution de Référence)
// ═══════════════════════════════════════════════════════════════

pub struct EnforcerSquad<K, V> {
    slots: Vec<Option<(K, V, usize)>>,
    len: usize,
    capacity: usize,
}

impl<K: Hash + Eq, V> EnforcerSquad<K, V> {
    const DEFAULT_CAPACITY: usize = 16;
    const MAX_LOAD_FACTOR: f64 = 0.9;

    pub fn new() -> Self {
        Self::with_capacity(Self::DEFAULT_CAPACITY)
    }

    pub fn with_capacity(capacity: usize) -> Self {
        let capacity = capacity.max(1);
        EnforcerSquad {
            slots: (0..capacity).map(|_| None).collect(),
            len: 0,
            capacity,
        }
    }

    fn hash(&self, key: &K) -> usize {
        let mut hasher = DefaultHasher::new();
        key.hash(&mut hasher);
        hasher.finish() as usize % self.capacity
    }

    pub fn insert(&mut self, mut key: K, mut value: V) -> Option<V> {
        if self.len as f64 / self.capacity as f64 > Self::MAX_LOAD_FACTOR {
            self.resize();
        }

        let mut idx = self.hash(&key);
        let mut probe_dist = 0usize;

        loop {
            match &mut self.slots[idx] {
                None => {
                    self.slots[idx] = Some((key, value, probe_dist));
                    self.len += 1;
                    return None;
                }
                Some((existing_key, existing_value, existing_dist)) => {
                    if existing_key == &key {
                        return Some(std::mem::replace(existing_value, value));
                    }

                    // Robin Hood: steal from the rich (low probe distance)
                    if probe_dist > *existing_dist {
                        std::mem::swap(&mut key, existing_key);
                        std::mem::swap(&mut value, existing_value);
                        std::mem::swap(&mut probe_dist, existing_dist);
                    }
                }
            }

            idx = (idx + 1) % self.capacity;
            probe_dist += 1;
        }
    }

    pub fn get(&self, key: &K) -> Option<&V> {
        let mut idx = self.hash(key);
        let mut probe_dist = 0usize;

        loop {
            match &self.slots[idx] {
                None => return None,
                Some((k, v, dist)) => {
                    if k == key {
                        return Some(v);
                    }
                    if probe_dist > *dist {
                        return None;  // Robin Hood optimization
                    }
                }
            }
            idx = (idx + 1) % self.capacity;
            probe_dist += 1;
        }
    }

    pub fn remove(&mut self, key: &K) -> Option<V> {
        let mut idx = self.hash(key);
        let mut probe_dist = 0usize;

        loop {
            match &self.slots[idx] {
                None => return None,
                Some((k, _, dist)) => {
                    if k == key {
                        let result = self.slots[idx].take().map(|(_, v, _)| v);
                        self.len -= 1;
                        self.backward_shift(idx);
                        return result;
                    }
                    if probe_dist > *dist {
                        return None;
                    }
                }
            }
            idx = (idx + 1) % self.capacity;
            probe_dist += 1;
        }
    }

    fn backward_shift(&mut self, mut idx: usize) {
        loop {
            let next = (idx + 1) % self.capacity;
            match &self.slots[next] {
                None => break,
                Some((_, _, dist)) if *dist == 0 => break,
                Some(_) => {
                    self.slots.swap(idx, next);
                    if let Some((_, _, dist)) = &mut self.slots[idx] {
                        *dist -= 1;
                    }
                    idx = next;
                }
            }
        }
    }

    fn resize(&mut self) {
        let old_slots = std::mem::replace(
            &mut self.slots,
            (0..self.capacity * 2).map(|_| None).collect(),
        );
        self.capacity *= 2;
        self.len = 0;

        for slot in old_slots.into_iter().flatten() {
            self.insert(slot.0, slot.1);
        }
    }

    pub fn average_probe_distance(&self) -> f64 {
        if self.len == 0 {
            return 0.0;
        }
        let total: usize = self.slots
            .iter()
            .filter_map(|s| s.as_ref().map(|(_, _, d)| *d))
            .sum();
        total as f64 / self.len as f64
    }

    pub fn max_probe_distance(&self) -> usize {
        self.slots
            .iter()
            .filter_map(|s| s.as_ref().map(|(_, _, d)| *d))
            .max()
            .unwrap_or(0)
    }
}

// ═══════════════════════════════════════════════════════════════
// LATENT DETECTOR - Bloom Filter (Solution de Référence)
// ═══════════════════════════════════════════════════════════════

pub struct LatentDetector {
    bits: Vec<bool>,
    num_hashes: usize,
    num_items: usize,
}

impl LatentDetector {
    pub fn new(capacity: usize) -> Self {
        Self::with_fp_rate(capacity, 0.01)
    }

    pub fn with_fp_rate(capacity: usize, fp_rate: f64) -> Self {
        // Optimal size: m = -n*ln(p) / (ln(2)^2)
        let ln2_sq = std::f64::consts::LN_2.powi(2);
        let m = (-(capacity as f64) * fp_rate.ln() / ln2_sq).ceil() as usize;

        // Optimal hashes: k = (m/n) * ln(2)
        let k = ((m as f64 / capacity as f64) * std::f64::consts::LN_2).ceil() as usize;

        LatentDetector {
            bits: vec![false; m.max(64)],
            num_hashes: k.max(1),
            num_items: 0,
        }
    }

    fn hash_indices<T: Hash>(&self, item: &T) -> Vec<usize> {
        let mut hasher1 = DefaultHasher::new();
        item.hash(&mut hasher1);
        let h1 = hasher1.finish();

        let mut hasher2 = DefaultHasher::new();
        h1.hash(&mut hasher2);
        let h2 = hasher2.finish();

        (0..self.num_hashes)
            .map(|i| ((h1.wrapping_add((i as u64).wrapping_mul(h2))) as usize) % self.bits.len())
            .collect()
    }

    pub fn insert<T: Hash>(&mut self, item: &T) {
        for idx in self.hash_indices(item) {
            self.bits[idx] = true;
        }
        self.num_items += 1;
    }

    pub fn contains<T: Hash>(&self, item: &T) -> bool {
        self.hash_indices(item).iter().all(|&idx| self.bits[idx])
    }

    pub fn estimated_fp_rate(&self) -> f64 {
        let m = self.bits.len() as f64;
        let k = self.num_hashes as f64;
        let n = self.num_items as f64;

        (1.0 - (-k * n / m).exp()).powf(k)
    }

    pub fn clear(&mut self) {
        self.bits.iter_mut().for_each(|b| *b = false);
        self.num_items = 0;
    }
}

// ═══════════════════════════════════════════════════════════════
// CITY POPULATION - HyperLogLog (Solution de Référence)
// ═══════════════════════════════════════════════════════════════

pub struct CityPopulation {
    registers: Vec<u8>,
    precision: usize,
}

impl CityPopulation {
    pub fn new(precision: usize) -> Self {
        let precision = precision.clamp(4, 18);
        let num_registers = 1 << precision;

        CityPopulation {
            registers: vec![0; num_registers],
            precision,
        }
    }

    pub fn add<T: Hash>(&mut self, item: &T) {
        let mut hasher = DefaultHasher::new();
        item.hash(&mut hasher);
        let hash = hasher.finish();

        // First `precision` bits determine the register
        let register_idx = (hash >> (64 - self.precision)) as usize;

        // Count leading zeros in the remaining bits + 1
        let remaining = hash << self.precision;
        let rank = remaining.leading_zeros() as u8 + 1;

        self.registers[register_idx] = self.registers[register_idx].max(rank);
    }

    pub fn count(&self) -> f64 {
        let m = self.registers.len() as f64;

        // Alpha correction factor
        let alpha = match self.registers.len() {
            16 => 0.673,
            32 => 0.697,
            64 => 0.709,
            _ => 0.7213 / (1.0 + 1.079 / m),
        };

        // Harmonic mean
        let sum: f64 = self.registers
            .iter()
            .map(|&r| 2.0_f64.powi(-(r as i32)))
            .sum();

        let raw_estimate = alpha * m * m / sum;

        // Small range correction (linear counting)
        if raw_estimate <= 2.5 * m {
            let zeros = self.registers.iter().filter(|&&r| r == 0).count();
            if zeros > 0 {
                return m * (m / zeros as f64).ln();
            }
        }

        // Large range correction (not needed for 64-bit hashes)
        raw_estimate
    }

    pub fn merge(&mut self, other: &Self) {
        assert_eq!(self.precision, other.precision);
        for (a, &b) in self.registers.iter_mut().zip(other.registers.iter()) {
            *a = (*a).max(b);
        }
    }

    pub fn clear(&mut self) {
        self.registers.iter_mut().for_each(|r| *r = 0);
    }
}

// ═══════════════════════════════════════════════════════════════
// CRIME SKETCH - Count-Min Sketch (Solution de Référence)
// ═══════════════════════════════════════════════════════════════

pub struct CrimeSketch {
    table: Vec<Vec<u64>>,
    width: usize,
    depth: usize,
}

impl CrimeSketch {
    pub fn new(width: usize, depth: usize) -> Self {
        CrimeSketch {
            table: vec![vec![0; width]; depth],
            width,
            depth,
        }
    }

    pub fn with_accuracy(epsilon: f64, delta: f64) -> Self {
        // width = ceil(e / epsilon)
        // depth = ceil(ln(1/delta))
        let width = (std::f64::consts::E / epsilon).ceil() as usize;
        let depth = (1.0 / delta).ln().ceil() as usize;
        Self::new(width.max(4), depth.max(2))
    }

    fn hash_indices<T: Hash>(&self, item: &T) -> Vec<usize> {
        let mut hasher = DefaultHasher::new();
        item.hash(&mut hasher);
        let h = hasher.finish();

        (0..self.depth)
            .map(|i| {
                let shifted = h.wrapping_add(i as u64 * 0x9e3779b97f4a7c15);
                (shifted as usize) % self.width
            })
            .collect()
    }

    pub fn add<T: Hash>(&mut self, item: &T, count: u64) {
        for (row, col) in self.hash_indices(item).into_iter().enumerate() {
            self.table[row][col] = self.table[row][col].saturating_add(count);
        }
    }

    pub fn increment<T: Hash>(&mut self, item: &T) {
        self.add(item, 1);
    }

    pub fn estimate<T: Hash>(&self, item: &T) -> u64 {
        self.hash_indices(item)
            .into_iter()
            .enumerate()
            .map(|(row, col)| self.table[row][col])
            .min()
            .unwrap_or(0)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_sibyl_basic() {
        let mut table: SibylChained<String, i32> = SibylChained::new();

        assert!(table.insert("Kogami".into(), 120).is_none());
        assert_eq!(table.get(&"Kogami".into()), Some(&120));
        assert_eq!(table.insert("Kogami".into(), 300), Some(120));
        assert_eq!(table.get(&"Kogami".into()), Some(&300));
    }

    #[test]
    fn test_robin_hood_variance() {
        let mut table: EnforcerSquad<i32, i32> = EnforcerSquad::new();

        for i in 0..1000 {
            table.insert(i, i * 2);
        }

        assert!(table.average_probe_distance() < 3.0);
    }

    #[test]
    fn test_bloom_filter() {
        let mut bloom = LatentDetector::with_fp_rate(1000, 0.01);

        for i in 0..1000 {
            bloom.insert(&i);
        }

        // No false negatives
        for i in 0..1000 {
            assert!(bloom.contains(&i));
        }

        // Count false positives
        let fp: usize = (1000..2000).filter(|i| bloom.contains(i)).count();
        assert!(fp < 50);  // Should be around 10 (1%)
    }

    #[test]
    fn test_hyperloglog() {
        let mut hll = CityPopulation::new(14);

        for i in 0..10000 {
            hll.add(&i);
        }

        let estimate = hll.count();
        let error = (estimate - 10000.0).abs() / 10000.0;
        assert!(error < 0.02);
    }
}
```

### 4.4 Solutions alternatives acceptées

```rust
// Alternative 1: Using bitvec for Bloom Filter
use bitvec::prelude::*;

pub struct LatentDetectorAlt {
    bits: BitVec,
    // ...
}

// Alternative 2: Cuckoo avec plus de 2 tables
pub struct CrimeDivisionAlt<K, V> {
    tables: Vec<Vec<Option<(K, V)>>>,  // N tables au lieu de 2
    // ...
}
```

### 4.5 Solutions refusées (avec explications)

```rust
// ❌ REFUSÉ : Pas de rehash
pub fn insert(&mut self, key: K, value: V) -> Option<V> {
    // Manque: if self.load_factor() > 0.75 { self.resize(); }
    let idx = self.bucket_index(&key);
    self.buckets[idx].push((key, value));
    None
}
// Problème: Performance dégradée O(n) quand load factor augmente

// ❌ REFUSÉ : Robin Hood sans backward shift
pub fn remove(&mut self, key: &K) -> Option<V> {
    // Trouve et supprime mais ne fait pas le backward shift
    // Problème: Les recherches futures peuvent échouer
}

// ❌ REFUSÉ : HyperLogLog sans bias correction
pub fn count(&self) -> f64 {
    let sum: f64 = self.registers.iter().map(|&r| 2.0_f64.powi(-(r as i32))).sum();
    let m = self.registers.len() as f64;
    m * m / sum  // Manque alpha et linear counting
    // Problème: Erreur systématique de 40%+
}
```

### 4.6 Solution bonus de référence (COMPLÈTE)

```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

/// SIMD-accelerated lookup using AVX2
pub struct SibylSimd<K: Hash + Eq, V> {
    keys: Vec<K>,
    values: Vec<V>,
    hashes: Vec<u64>,
    capacity: usize,
}

impl<K: Hash + Eq, V> SibylSimd<K, V> {
    pub fn new() -> Self {
        Self::with_capacity(64)
    }

    pub fn with_capacity(capacity: usize) -> Self {
        let capacity = ((capacity + 3) / 4) * 4;  // Align to 4
        SibylSimd {
            keys: Vec::with_capacity(capacity),
            values: Vec::with_capacity(capacity),
            hashes: Vec::with_capacity(capacity),
            capacity,
        }
    }

    fn compute_hash(key: &K) -> u64 {
        let mut hasher = std::collections::hash_map::DefaultHasher::new();
        key.hash(&mut hasher);
        hasher.finish()
    }

    pub fn insert(&mut self, key: K, value: V) -> Option<V> {
        let hash = Self::compute_hash(&key);

        // Linear search with SIMD for existing key
        if let Some(idx) = self.find_index_simd(hash, &key) {
            return Some(std::mem::replace(&mut self.values[idx], value));
        }

        self.keys.push(key);
        self.values.push(value);
        self.hashes.push(hash);
        None
    }

    #[cfg(target_arch = "x86_64")]
    fn find_index_simd(&self, target_hash: u64, key: &K) -> Option<usize> {
        if self.hashes.is_empty() {
            return None;
        }

        unsafe {
            if is_x86_feature_detected!("avx2") {
                return self.find_index_avx2(target_hash, key);
            }
        }

        // Fallback to scalar
        self.hashes.iter().enumerate()
            .find(|(i, &h)| h == target_hash && &self.keys[*i] == key)
            .map(|(i, _)| i)
    }

    #[cfg(target_arch = "x86_64")]
    #[target_feature(enable = "avx2")]
    unsafe fn find_index_avx2(&self, target_hash: u64, key: &K) -> Option<usize> {
        let target = _mm256_set1_epi64x(target_hash as i64);
        let chunks = self.hashes.chunks_exact(4);
        let remainder = chunks.remainder();

        for (chunk_idx, chunk) in chunks.enumerate() {
            let hashes = _mm256_loadu_si256(chunk.as_ptr() as *const __m256i);
            let cmp = _mm256_cmpeq_epi64(hashes, target);
            let mask = _mm256_movemask_epi8(cmp);

            if mask != 0 {
                // Found potential match, verify key
                for i in 0..4 {
                    if (mask >> (i * 8)) & 0xFF != 0 {
                        let idx = chunk_idx * 4 + i;
                        if &self.keys[idx] == key {
                            return Some(idx);
                        }
                    }
                }
            }
        }

        // Check remainder
        let base = self.hashes.len() - remainder.len();
        for (i, &h) in remainder.iter().enumerate() {
            if h == target_hash && &self.keys[base + i] == key {
                return Some(base + i);
            }
        }

        None
    }

    pub fn get(&self, key: &K) -> Option<&V> {
        let hash = Self::compute_hash(key);
        self.find_index_simd(hash, key).map(|i| &self.values[i])
    }

    pub fn batch_lookup(&self, keys: &[K]) -> Vec<Option<&V>> {
        keys.iter().map(|k| self.get(k)).collect()
    }
}

/// Lock-free concurrent hash table using CAS
use std::sync::atomic::{AtomicPtr, AtomicUsize, Ordering};

pub struct ConcurrentSibyl<K, V> {
    buckets: Vec<AtomicPtr<Node<K, V>>>,
    len: AtomicUsize,
    capacity: usize,
}

struct Node<K, V> {
    key: K,
    value: V,
    next: AtomicPtr<Node<K, V>>,
}

impl<K: Hash + Eq + Clone, V: Clone> ConcurrentSibyl<K, V> {
    pub fn new() -> Self {
        Self::with_capacity(16)
    }

    pub fn with_capacity(capacity: usize) -> Self {
        ConcurrentSibyl {
            buckets: (0..capacity).map(|_| AtomicPtr::new(std::ptr::null_mut())).collect(),
            len: AtomicUsize::new(0),
            capacity,
        }
    }

    fn hash(key: &K) -> u64 {
        let mut hasher = std::collections::hash_map::DefaultHasher::new();
        key.hash(&mut hasher);
        hasher.finish()
    }

    pub fn insert(&self, key: K, value: V) -> Option<V> {
        let idx = (Self::hash(&key) as usize) % self.capacity;
        let new_node = Box::into_raw(Box::new(Node {
            key: key.clone(),
            value,
            next: AtomicPtr::new(std::ptr::null_mut()),
        }));

        loop {
            let head = self.buckets[idx].load(Ordering::Acquire);

            // Check if key exists
            let mut current = head;
            while !current.is_null() {
                unsafe {
                    if (*current).key == key {
                        // Key exists, update value (simplified - real impl needs more care)
                        let old = std::mem::replace(&mut (*current).value, (*new_node).value.clone());
                        let _ = Box::from_raw(new_node);  // Free unused node
                        return Some(old);
                    }
                    current = (*current).next.load(Ordering::Acquire);
                }
            }

            // Insert at head
            unsafe {
                (*new_node).next.store(head, Ordering::Release);
            }

            match self.buckets[idx].compare_exchange(
                head,
                new_node,
                Ordering::AcqRel,
                Ordering::Acquire,
            ) {
                Ok(_) => {
                    self.len.fetch_add(1, Ordering::Relaxed);
                    return None;
                }
                Err(_) => continue,  // Retry
            }
        }
    }

    pub fn get(&self, key: &K) -> Option<V> {
        let idx = (Self::hash(key) as usize) % self.capacity;
        let mut current = self.buckets[idx].load(Ordering::Acquire);

        while !current.is_null() {
            unsafe {
                if (*current).key == *key {
                    return Some((*current).value.clone());
                }
                current = (*current).next.load(Ordering::Acquire);
            }
        }

        None
    }
}

impl<K, V> Drop for ConcurrentSibyl<K, V> {
    fn drop(&mut self) {
        for bucket in &self.buckets {
            let mut current = bucket.load(Ordering::Relaxed);
            while !current.is_null() {
                unsafe {
                    let next = (*current).next.load(Ordering::Relaxed);
                    let _ = Box::from_raw(current);
                    current = next;
                }
            }
        }
    }
}
```

### 4.9 spec.json (ENGINE v22.1 — FORMAT STRICT)

```json
{
  "name": "sibyl_system",
  "language": "rust",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse Hash Tables",
  "tags": ["hash", "probabilistic", "phase1", "advanced"],
  "passing_score": 70,

  "function": {
    "name": "SibylChained",
    "prototype": "pub struct SibylChained<K, V>",
    "return_type": "struct",
    "parameters": [
      {"name": "K", "type": "generic Hash + Eq"},
      {"name": "V", "type": "generic"}
    ]
  },

  "driver": {
    "reference": "impl<K: Hash + Eq, V> SibylChained<K, V> { pub fn new() -> Self { SibylChained { buckets: (0..16).map(|_| Vec::new()).collect(), len: 0, hash_builder: RandomState::new() } } pub fn insert(&mut self, key: K, value: V) -> Option<V> { if self.load_factor() > 0.75 { self.resize(); } let idx = self.bucket_index(&key); for (k, v) in self.buckets[idx].iter_mut() { if k == &key { return Some(std::mem::replace(v, value)); } } self.buckets[idx].push((key, value)); self.len += 1; None } }",

    "edge_cases": [
      {
        "name": "empty_table",
        "args": ["SibylChained::new()", "get", "\"test\""],
        "expected": "None",
        "is_trap": true,
        "trap_explanation": "Get on empty table must return None"
      },
      {
        "name": "insert_get",
        "args": ["insert(\"key\", 42)", "get(\"key\")"],
        "expected": "Some(&42)",
        "is_trap": false
      },
      {
        "name": "update_existing",
        "args": ["insert(\"key\", 1)", "insert(\"key\", 2)"],
        "expected": "Some(1)",
        "is_trap": true,
        "trap_explanation": "Update must return old value"
      },
      {
        "name": "high_load_factor",
        "args": ["insert 1000 elements"],
        "expected": "load_factor() < 0.8",
        "is_trap": true,
        "trap_explanation": "Must resize before load factor exceeds threshold"
      },
      {
        "name": "bloom_no_false_negative",
        "args": ["insert 1000", "contains all 1000"],
        "expected": "all true",
        "is_trap": true,
        "trap_explanation": "Bloom filter cannot have false negatives"
      },
      {
        "name": "hll_accuracy",
        "args": ["add 10000 unique"],
        "expected": "error < 3%",
        "is_trap": true,
        "trap_explanation": "HyperLogLog must be accurate within bounds"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 10000,
      "generators": [
        {
          "type": "string",
          "param_index": 0,
          "params": {
            "min_len": 1,
            "max_len": 100,
            "charset": "alphanumeric"
          }
        },
        {
          "type": "int",
          "param_index": 1,
          "params": {
            "min": -1000000,
            "max": 1000000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["std::hash", "std::mem", "std::alloc", "Vec", "Option"],
    "forbidden_functions": ["HashMap", "HashSet", "BTreeMap"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```rust
/* Mutant A (Boundary) : Division par zéro sur table vide */
impl<K: Hash + Eq, V> SibylChained<K, V> {
    fn bucket_index(&self, key: &K) -> usize {
        self.hash(key) % self.buckets.len()  // Crash si buckets.len() == 0
    }
}
// Pourquoi c'est faux : Si with_capacity(0) est appelé, division par zéro
// Ce qui était pensé : "La capacité sera toujours > 0"

/* Mutant B (Safety) : Robin Hood swap sans gestion ownership */
impl<K: Hash + Eq, V> EnforcerSquad<K, V> {
    pub fn insert(&mut self, key: K, value: V) -> Option<V> {
        // ...
        if probe_dist > *existing_dist {
            // ❌ Copie au lieu de swap - double free possible
            let temp_key = existing_key.clone();  // Requires Clone
            *existing_key = key;
            key = temp_key;
        }
    }
}
// Pourquoi c'est faux : Viole ownership rules, potentiel double free
// Ce qui était pensé : "Je dois échanger les valeurs"

/* Mutant C (Resize) : Rehash avec ancien index au lieu de nouveau hash */
impl<K: Hash + Eq, V> SibylChained<K, V> {
    fn resize(&mut self) {
        let new_cap = self.buckets.len() * 2;
        self.buckets.resize_with(new_cap, Vec::new);
        // ❌ Ne recalcule pas les hash ! Les éléments restent dans les mauvais buckets
    }
}
// Pourquoi c'est faux : Les recherches échoueront car index = hash % NEW_capacity
// Ce qui était pensé : "J'agrandis juste le tableau"

/* Mutant D (Logic) : Cuckoo sans limite d'évictions */
impl<K: Hash + Eq + Clone, V: Clone> CrimeDivision<K, V> {
    pub fn insert(&mut self, key: K, value: V) -> Result<Option<V>, (K, V)> {
        loop {  // ❌ Boucle infinie possible !
            // Évictions sans compteur
            if let Some(evicted) = self.division1[idx1].take() {
                // Évince vers division2, puis potentiellement revient à division1...
            }
        }
    }
}
// Pourquoi c'est faux : Cycle d'évictions infini quand table est trop pleine
// Ce qui était pensé : "Ça finira par trouver une place"

/* Mutant E (Math) : HyperLogLog sans correction bias */
impl CityPopulation {
    pub fn count(&self) -> f64 {
        let sum: f64 = self.registers.iter()
            .map(|&r| 2.0_f64.powi(-(r as i32)))
            .sum();
        let m = self.registers.len() as f64;
        // ❌ Manque alpha et linear counting
        m * m / sum
    }
}
// Pourquoi c'est faux : Surestimation de ~40% pour grandes cardinalités
// Ce qui était pensé : "La formule de base suffit"

/* Mutant F (Bloom) : Hash unique au lieu de k hash */
impl LatentDetector {
    pub fn insert<T: Hash>(&mut self, item: &T) {
        let mut hasher = DefaultHasher::new();
        item.hash(&mut hasher);
        let idx = (hasher.finish() as usize) % self.bits.len();
        self.bits[idx] = true;  // ❌ Un seul bit !
    }
}
// Pourquoi c'est faux : Taux de faux positifs catastrophique (50%+)
// Ce qui était pensé : "Un hash suffit"
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

1. **Résolution de collisions** : Comprendre pourquoi et comment différentes stratégies existent
2. **Trade-offs temps/espace** : Chaque structure fait des compromis différents
3. **Structures probabilistes** : Accepter l'approximation pour gagner en efficacité
4. **Analyse amortie** : Comment obtenir O(1) malgré des opérations coûteuses

### 5.2 LDA — Traduction littérale en français (MAJUSCULES)

**SibylChained.insert :**
```
FONCTION insert QUI RETOURNE OPTIONNEL VALEUR ET PREND EN PARAMÈTRES key ET value
DÉBUT FONCTION
    SI LE FACTEUR DE CHARGE EST SUPÉRIEUR À 0.75 ALORS
        APPELER resize
    FIN SI

    AFFECTER hash(key) MODULO capacité À idx

    POUR CHAQUE (k, v) DANS buckets[idx] FAIRE
        SI k EST ÉGAL À key ALORS
            RETOURNER Some(remplacer v par value)
        FIN SI
    FIN POUR

    AJOUTER (key, value) À buckets[idx]
    INCRÉMENTER len DE 1
    RETOURNER None
FIN FONCTION
```

**HyperLogLog.add :**
```
FONCTION add QUI PREND EN PARAMÈTRE item
DÉBUT FONCTION
    AFFECTER hash(item) À h

    AFFECTER h DÉCALÉ À DROITE DE (64 - precision) BITS À register_idx
    AFFECTER h DÉCALÉ À GAUCHE DE precision BITS À remaining

    AFFECTER NOMBRE DE ZÉROS EN TÊTE DE remaining PLUS 1 À rank

    SI rank EST SUPÉRIEUR À registers[register_idx] ALORS
        AFFECTER rank À registers[register_idx]
    FIN SI
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Robin Hood Insert
---
1. CALCULER le hash et l'index initial
2. INITIALISER probe_distance = 0

3. BOUCLE :
   a. SI slot[index] est vide :
      - PLACER (key, value, probe_distance)
      - RETOURNER None

   b. SINON SI slot[index].key == key :
      - REMPLACER la valeur
      - RETOURNER ancienne valeur

   c. SINON SI probe_distance > slot[index].probe_distance :
      - ÉCHANGER (key, value, probe_distance) avec slot[index]

   d. INCRÉMENTER index (modulo capacity)
   e. INCRÉMENTER probe_distance

4. FIN BOUCLE
```

### 5.2.3 Représentation Algorithmique (Logique de Garde)

```
FONCTION : LatentDetector.contains (item)
---
INIT result = true

1. POUR i DE 0 À num_hashes - 1 :
   |
   |-- CALCULER index = hash_i(item) % bits.len()
   |
   |-- SI bits[index] == false :
   |     RETOURNER false  // Définitivement absent
   |
2. RETOURNER true  // Probablement présent (ou faux positif)
```

### 5.3 Visualisation ASCII

**Architecture du Système Sibyl (Hash Tables) :**
```
                    SYSTÈME SIBYL - ARCHITECTURE INTERNE
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌─────────────┐      ┌──────────────────────────────────────┐ │
│  │   ENTRÉE    │      │        SIBYL CHAINED                 │ │
│  │  Citoyen ID │─────►│  [0]──→(K,V)──→(K,V)──→∅             │ │
│  │   "Kogami"  │      │  [1]──→(K,V)──→∅                     │ │
│  └─────────────┘      │  [2]──→∅                             │ │
│                       │  [3]──→(K,V)──→(K,V)──→(K,V)──→∅     │ │
│        hash()         │  ...                                  │ │
│          │            │  [n]──→(K,V)──→∅                     │ │
│          ▼            └──────────────────────────────────────┘ │
│    ┌─────────┐                                                 │
│    │ 0x7A3F  │                                                 │
│    └────┬────┘                                                 │
│         │                                                      │
│         ├──────────────────────────────────────────────────┐   │
│         │                                                  │   │
│         ▼                                                  ▼   │
│  ┌──────────────────────┐      ┌──────────────────────────┐│   │
│  │   DOMINATOR PROBE    │      │      ENFORCER SQUAD      ││   │
│  │  (Linear Probing)    │      │     (Robin Hood)         ││   │
│  │                      │      │                          ││   │
│  │ [0][1][2][3][4][5]   │      │ [K,V,d=0][K,V,d=1]...    ││   │
│  │  ↑  ↑  ↑             │      │     ↑ swap si d > d'     ││   │
│  │  │  │  └─collision   │      └──────────────────────────┘│   │
│  │  │  └─tombstone      │                                  │   │
│  │  └─target            │                                  │   │
│  └──────────────────────┘                                  │   │
│                                                            │   │
└────────────────────────────────────────────────────────────────┘

                    CRIME DIVISION (CUCKOO HASHING)
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│     DIVISION 1                      DIVISION 2                  │
│     (hash1)                         (hash2)                     │
│  ┌───┬───┬───┬───┐              ┌───┬───┬───┬───┐              │
│  │ A │   │ C │   │              │   │ B │   │ D │              │
│  └───┴───┴───┴───┘              └───┴───┴───┴───┘              │
│         │                              ▲                        │
│         │          ÉVICTION            │                        │
│         └──────────────────────────────┘                        │
│                                                                 │
│  Insert E:                                                      │
│  1. E→Div1[2] ? Occupé par C                                   │
│  2. Évince C vers Div2                                          │
│  3. C→Div2[1] ? Occupé par B                                   │
│  4. Évince B vers Div1                                          │
│  5. B→Div1[0] ? Occupé par A... (continue ou rehash)           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

              STRUCTURES PROBABILISTES
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  LATENT DETECTOR (Bloom Filter)                                 │
│  ┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐                             │
│  │0│1│0│0│1│0│1│0│0│1│0│0│1│0│1│0│ ← bit array                 │
│  └─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘                             │
│     ↑     ↑     ↑                                               │
│     └──h1─┴──h2─┴──h3── "Kogami"                                │
│                                                                 │
│  contains("Makishima")?                                         │
│  h1("Makishima") → bit[3] = 0 → DÉFINITIVEMENT NON             │
│                                                                 │
│  CITY POPULATION (HyperLogLog)                                  │
│  ┌────┬────┬────┬────┬────┬────┐                               │
│  │ r0 │ r1 │ r2 │ r3 │... │r_m │ ← registres (max leading 0s)  │
│  │ 3  │ 5  │ 2  │ 7  │    │ 4  │                               │
│  └────┴────┴────┴────┴────┴────┘                               │
│                                                                 │
│  Estimation = α × m² / Σ(2^(-r_j))                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.4 Les pièges en détail

| Piège | Conséquence | Solution |
|-------|-------------|----------|
| Division par zéro (capacity=0) | Crash | `capacity.max(1)` |
| Oublier le rehash | O(n) dégradé | Check load factor |
| Robin Hood sans backward shift | Lookups échouent | Shift après remove |
| Cuckoo boucle infinie | Hang | Limiter évictions + rehash |
| HyperLogLog sans bias | Erreur 40%+ | Alpha × linear counting |
| Bloom avec 1 seul hash | 50%+ FP | k = (m/n) × ln(2) |

### 5.5 Cours Complet

#### 5.5.1 Théorie du Hashing

Le **hashing** transforme des données de taille arbitraire en un index de taille fixe. Une bonne fonction de hash doit :
- Être **déterministe** : même entrée → même sortie
- Avoir une **distribution uniforme** : minimiser les collisions
- Être **efficace** : O(1) pour calculer

**Fonctions de hash populaires :**
- **FNV-1a** : Simple, rapide, bon pour strings
- **MurmurHash3** : Excellent compromis vitesse/qualité
- **xxHash** : Le plus rapide pour grandes données
- **SipHash** : Cryptographiquement sûr (utilisé par Rust par défaut)

#### 5.5.2 Stratégies de Résolution de Collisions

**1. Chaînage Séparé (Separate Chaining)**
- Chaque bucket contient une liste
- Avantages : Simple, supporte load factor > 1
- Inconvénients : Cache unfriendly, overhead mémoire

**2. Adressage Ouvert (Open Addressing)**
- Tous les éléments dans le tableau
- Variantes :
  - **Linear Probing** : h(k) + i → clustering primaire
  - **Quadratic Probing** : h(k) + i² → clustering secondaire
  - **Double Hashing** : h(k) + i×h2(k) → pas de clustering

**3. Robin Hood Hashing**
- Variante de linear probing
- "Vole" la place si probe distance plus grande
- Réduit la variance : recherche plus prévisible

**4. Cuckoo Hashing**
- Deux tables, deux fonctions de hash
- Lookup O(1) worst case (2 accès max)
- Insertion peut échouer → rehash nécessaire

#### 5.5.3 Structures Probabilistes

**Bloom Filter**
- Ensemble approximatif : faux positifs possibles, pas de faux négatifs
- Formules optimales :
  - m = -n×ln(p) / ln²(2) bits
  - k = (m/n) × ln(2) fonctions de hash

**Count-Min Sketch**
- Compteur de fréquences approximatif
- Toujours surestimé (jamais sous-estimé)
- Erreur bornée par ε avec probabilité 1-δ

**HyperLogLog**
- Estime la cardinalité (nombre d'éléments uniques)
- Utilise le rang du premier bit 1
- Précision ~1/√m avec m registres

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ pub fn insert(&mut self, k: K, v: V)                           │
│ {                                                               │
│     let i = self.hash(&k) % self.buckets.len();                │
│     self.buckets[i].push((k, v)); self.len += 1;               │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ pub fn insert(&mut self, key: K, value: V) -> Option<V> {      │
│     self.maybe_resize();                                        │
│                                                                 │
│     let idx = self.bucket_index(&key);                         │
│     let bucket = &mut self.buckets[idx];                       │
│                                                                 │
│     for (k, v) in bucket.iter_mut() {                          │
│         if k == &key {                                          │
│             return Some(std::mem::replace(v, value));          │
│         }                                                       │
│     }                                                           │
│                                                                 │
│     bucket.push((key, value));                                 │
│     self.len += 1;                                              │
│     None                                                        │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Noms explicites : idx, bucket au lieu de i                   │
│ • Retour Option<V> : API standard, permet de savoir si update   │
│ • Resize check : Maintient les garanties de performance         │
│ • Une opération par ligne : Lisible et debuggable              │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Trace : Robin Hood Insert avec 3 éléments**

État initial : capacity=5, tous vides

```
Insert("A", 1):
  hash("A") % 5 = 2
  slot[2] = None → Place directement

  Après: [_, _, (A,1,d=0), _, _]

Insert("B", 2):
  hash("B") % 5 = 2  // Collision !
  slot[2] = (A,1,d=0), our d=0, not > 0 → Continue
  slot[3] = None → Place

  Après: [_, _, (A,1,d=0), (B,2,d=1), _]

Insert("C", 3):
  hash("C") % 5 = 2  // Encore collision !
  slot[2] = (A,1,d=0), our d=0, not > 0 → Continue
  slot[3] = (B,2,d=1), our d=1, not > 1 → Continue
  slot[4] = None → Place

  Après: [_, _, (A,1,d=0), (B,2,d=1), (C,3,d=2)]

Insert("D", 4):
  hash("D") % 5 = 3
  slot[3] = (B,2,d=1), our d=0, not > 1 → Continue
  slot[4] = (C,3,d=2), our d=1, not > 2 → Continue
  slot[0] = None → Place

  Après: [(D,4,d=2), _, (A,1,d=0), (B,2,d=1), (C,3,d=2)]

Insert("E", 5):
  hash("E") % 5 = 3
  slot[3] = (B,2,d=1), our d=0, not > 1 → Continue
  slot[4] = (C,3,d=2), our d=1, not > 2 → Continue
  slot[0] = (D,4,d=2), our d=2, not > 2 → Continue
  slot[1] = None → Place

  Après: [(D,4,d=2), (E,5,d=3), (A,1,d=0), (B,2,d=1), (C,3,d=2)]

Average probe distance: (0+1+2+2+3)/5 = 1.6
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🔫 MEME : "Crime Coefficient Over 300" — Load Factor

![Crime Coefficient](meme_psychopass.jpg)

Quand ton Crime Coefficient dépasse 300, le Dominator passe en mode Eliminator.
Quand ton Load Factor dépasse 0.75, ta hash table passe en mode Resize.

```rust
fn insert(&mut self, key: K, value: V) {
    // 🔫 "Crime Coefficient increasing..."
    if self.load_factor() > 0.75 {
        // "CRIME COEFFICIENT OVER 300 - SWITCHING TO ELIMINATOR MODE"
        self.resize();  // Rehash everything
    }
}
```

#### 🦅 MEME : "Le Coucou ne fait pas de nid" — Cuckoo Hashing

Le coucou pond dans le nid des autres et pousse leurs oeufs.
Cuckoo Hashing fait pareil : pousse les éléments existants.

#### 🎯 MEME : "Maybe he's a criminal, maybe not" — Bloom Filter

Quand le Bloom Filter dit "OUI" :
- Peut-être criminel (true positive)
- Peut-être innocent (false positive)

Quand le Bloom Filter dit "NON" :
- DÉFINITIVEMENT innocent (jamais de false negative)

### 5.9 Applications pratiques

| Domaine | Structure | Usage |
|---------|-----------|-------|
| Navigateur Web | Bloom Filter | Safe Browsing (Google) - 500M+ URLs |
| Base de données | Robin Hood | Redis, MemSQL - latence prévisible |
| Big Data | HyperLogLog | Redis PFCOUNT - comptage unique |
| CDN | Count-Min Sketch | Rate limiting, DDoS detection |
| Compilateur | Cuckoo | Symbol tables - O(1) garanti |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

1. **Division par zéro** : Toujours vérifier capacity > 0
2. **Oubli du rehash** : Performance dégradée en O(n)
3. **Tombstones accumulés** : Rebuild périodique nécessaire
4. **Cuckoo cycles** : Limiter les évictions
5. **HyperLogLog bias** : Appliquer les corrections
6. **Bloom single hash** : Utiliser k fonctions de hash

---

## 📝 SECTION 7 : QCM

**Q1.** Quelle est la complexité moyenne d'un lookup dans une hash table bien dimensionnée ?
- A) O(1)
- B) O(log n)
- C) O(n)
- D) O(n log n)

**Q2.** Qu'est-ce que le "Robin Hood" dans Robin Hood Hashing ?
- A) Voler les valeurs des autres buckets
- B) Donner sa place à un élément avec plus grande probe distance
- C) Supprimer les éléments les plus anciens
- D) Doubler la taille à chaque collision

**Q3.** Un Bloom Filter peut-il avoir des faux négatifs ?
- A) Oui, toujours
- B) Oui, si mal configuré
- C) Non, jamais
- D) Seulement après saturation

**Q4.** Combien de tables utilise Cuckoo Hashing standard ?
- A) 1
- B) 2
- C) 4
- D) Variable

**Q5.** Quelle formule donne le nombre optimal de fonctions de hash pour un Bloom Filter ?
- A) k = n
- B) k = m
- C) k = (m/n) × ln(2)
- D) k = √n

**Q6.** HyperLogLog estime quoi ?
- A) La fréquence des éléments
- B) La cardinalité (nombre d'uniques)
- C) La somme des valeurs
- D) La médiane

**Q7.** Qu'est-ce qu'un tombstone en open addressing ?
- A) Un élément supprimé mais marqué
- B) Un élément jamais utilisé
- C) Une collision non résolue
- D) Un overflow de bucket

**Q8.** Quel est le load factor maximum recommandé pour linear probing ?
- A) 0.3
- B) 0.5
- C) 0.9
- D) 1.0

**Réponses :** A, B, C, B, C, B, A, B

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Structure | Lookup | Insert | Delete | Space | Use Case |
|-----------|--------|--------|--------|-------|----------|
| Chained | O(1)* | O(1)* | O(1)* | O(n+m) | Général |
| Linear Probe | O(1)* | O(1)* | O(1)* | O(n) | Cache-friendly |
| Robin Hood | O(1)* | O(1)* | O(1)* | O(n) | Latence stable |
| Cuckoo | O(1) | O(1)* | O(1) | O(n) | Lookup garanti |
| Bloom | O(k) | O(k) | N/A | O(m) | Membership test |
| Count-Min | O(d) | O(d) | N/A | O(wd) | Frequency |
| HyperLogLog | N/A | O(1) | N/A | O(m) | Cardinality |

\* amortized

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.2.0-synth-sibyl-system",
    "generated_at": "2026-01-11 00:00:00",

    "metadata": {
      "exercise_id": "1.2.0-synth",
      "exercise_name": "sibyl_system",
      "module": "1.2",
      "module_name": "Hash Tables & Strings",
      "concept": "synth",
      "concept_name": "Synthèse Hash Tables",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 1,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust",
      "duration_minutes": 180,
      "xp_base": 200,
      "xp_bonus_multiplier": 6,
      "bonus_tier": "GÉNIE",
      "bonus_icon": "🧠",
      "complexity_time": "T5 O(1) amortized",
      "complexity_space": "S4 O(n)",
      "prerequisites": ["hashing", "generics", "memory"],
      "domains": ["Struct", "Probas", "Mem", "Compression"],
      "domains_bonus": ["CPU", "ASM"],
      "tags": ["hash", "probabilistic", "bloom", "hyperloglog"],
      "meme_reference": "Psycho-Pass Crime Coefficient"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "mutants/mutant_a_boundary.rs": "/* Division by zero */",
      "mutants/mutant_b_safety.rs": "/* Robin Hood ownership */",
      "mutants/mutant_c_resize.rs": "/* Wrong rehash */",
      "mutants/mutant_d_logic.rs": "/* Cuckoo infinite */",
      "mutants/mutant_e_math.rs": "/* HLL no bias */",
      "mutants/mutant_f_bloom.rs": "/* Single hash */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.rs",
        "references/ref_solution_bonus.rs"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.rs",
        "mutants/mutant_b_safety.rs",
        "mutants/mutant_c_resize.rs",
        "mutants/mutant_d_logic.rs",
        "mutants/mutant_e_math.rs",
        "mutants/mutant_f_bloom.rs"
      ]
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "Le Système Sibyl vous observe. Votre Crime Coefficient est... calculé."*
