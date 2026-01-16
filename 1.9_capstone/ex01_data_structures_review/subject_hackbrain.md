# Exercice 1.9.1-a : inception_data_architect

**Module :**
1.9.1 — Capstone: Data Structures Review

**Concept :**
a — Advanced data structures synthesis (Vec, Hash, BST, Heap, Segment tree, Fenwick, Trie, DSU, Sparse table)

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (révision complète structures de données Phase 1)

**Langage :**
Rust Edition 2024

**Prérequis :**
- Structures de données fondamentales (Vec, HashMap, BST, Heap)
- Arbres binaires et segment trees
- Union-Find / DSU
- Complexité algorithmique

**Domaines :**
Struct, Algo, Mem

**Durée estimée :**
90 min

**XP Base :**
180

**Complexité :**
T6 O(log n) à O(n log n) × S4 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**

| Structures | Fichiers |
|------------|----------|
| Base (5) | `vec_edge.rs`, `custom_hash.rs`, `bst.rs`, `heap_ord.rs`, `segment_tree.rs` |
| Bonus (4) | `fenwick_2d.rs`, `trie.rs`, `dsu.rs`, `sparse_table.rs` |

**Fonctions autorisées :**
- Rust : Toutes fonctions `std::`, `std::collections::*`, `std::cmp::*`

**Fonctions interdites :**
- Utilisation directe des structures std pour ce qu'on doit implémenter (ex: pas de `Vec` pour `MyVec`)

---

### 1.2 Consigne

#### 🎬 Section Culture : "We Need to Go Deeper"

**🌀 INCEPTION — "You're waiting for a train... to Data Structures Land"**

Tu connais INCEPTION ? Chaque niveau de rêve est plus profond, plus complexe, plus dangereux. La physique change, le temps ralentit.

En structures de données, c'est pareil :

- **Niveau 1 (Réalité)** : Vec, HashMap — O(1) access, simple, direct
- **Niveau 2 (Rêve)** : BST, Heap — O(log n), arbres binaires, équilibre délicat
- **Niveau 3 (Rêve dans rêve)** : Segment tree, Fenwick — Lazy propagation, queries complexes
- **Niveau 4 (Limbes)** : Trie compressed, DSU rollback, Sparse table — Structures exotiques, optimisations poussées

Comme Cobb qui navigue entre les niveaux, tu vas **implémenter 9 structures de données** de complexité croissante. Chaque niveau teste ta compréhension du précédent.

**Le "kick" pour sortir ?** C'est le `Drop` trait qui nettoie la mémoire quand tu quittes une structure.

**Le "totem" pour vérifier la réalité ?** Ce sont les **invariants** de chaque structure (BST property, heap property, etc.).

*"What is the most resilient data structure? An idea encoded in a well-designed struct."*

---

#### 🎓 Section Académique : Énoncé Formel

**Ta mission :**

Implémenter **5 structures de données** (exercice de base) avec leurs opérations et invariants :

1. **MyVec<T>** avec gestion d'edge cases (empty, overflow, shrink)
2. **Point** avec `Hash` trait personnalisé
3. **BST<K, V>** complet (insert, delete, search, iter)
4. **TaskQueue** (min-heap) avec `Ord` custom
5. **SegmentTree** avec lazy propagation (range update + range query)

**Entrée (Structure 1 - MyVec) :**

```rust
pub struct MyVec<T> {
    ptr: *mut T,
    len: usize,
    cap: usize,
}

impl<T> MyVec<T> {
    pub fn new() -> Self;
    pub fn push(&mut self, value: T);
    pub fn pop(&mut self) -> Option<T>;
    pub fn get(&self, index: usize) -> Option<&T>;
    pub fn shrink_to_fit(&mut self);  // Edge case gestion
}
```

**Entrée (Structure 2 - Custom Hash) :**

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Point {
    pub x: i32,
    pub y: i32,
}

impl Hash for Point {
    fn hash<H: Hasher>(&self, state: &mut H) {
        // Implémenter hasher qui combine x ET y correctement
    }
}
```

**Entrée (Structure 3 - BST) :**

```rust
pub struct BST<K: Ord, V> {
    root: Option<Box<Node<K, V>>>,
}

impl<K: Ord, V> BST<K, V> {
    pub fn new() -> Self;
    pub fn insert(&mut self, key: K, value: V);
    pub fn remove(&mut self, key: &K) -> Option<V>;
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn iter(&self) -> BSTIter<K, V>;  // In-order traversal
}
```

**Entrée (Structure 4 - Heap avec Ord custom) :**

```rust
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Task {
    pub priority: u32,
    pub name: String,
}

impl Ord for Task {
    fn cmp(&self, other: &Self) -> Ordering {
        // Min-heap : priority basse en premier
        self.priority.cmp(&other.priority)
    }
}

pub struct TaskQueue {
    heap: Vec<Task>,
}

impl TaskQueue {
    pub fn new() -> Self;
    pub fn push(&mut self, task: Task);
    pub fn pop(&mut self) -> Option<Task>;  // Pop min priority
}
```

**Entrée (Structure 5 - Segment Tree avec Lazy) :**

```rust
pub struct SegmentTree {
    tree: Vec<i64>,
    lazy: Vec<i64>,
    n: usize,
}

impl SegmentTree {
    pub fn new(arr: &[i64]) -> Self;
    pub fn range_query(&mut self, l: usize, r: usize) -> i64;  // Sum [l, r)
    pub fn range_update(&mut self, l: usize, r: usize, val: i64);  // Add val to [l, r)
}
```

**Sortie :**
- Toutes les opérations fonctionnent correctement
- Edge cases gérés (empty, bounds, overflow)
- Complexité respectée (voir tableau ci-dessous)
- Invariants préservés

**Contraintes :**

| Structure | Complexité Insert/Update | Complexité Query/Access | Invariant |
|-----------|--------------------------|-------------------------|-----------|
| MyVec | O(1) amorti | O(1) | len ≤ cap |
| Point Hash | O(1) | O(1) | x, y tous deux hashés |
| BST | O(log n) moyen | O(log n) moyen | left < root < right |
| TaskQueue | O(log n) | O(log n) | heap property |
| SegmentTree | O(log n) | O(log n) | tree[i] = sum des feuilles |

**Exemples :**

| Structure | Opération | Input | Output | Explication |
|-----------|-----------|-------|--------|-------------|
| MyVec | `push(1), push(2), pop()` | — | `Some(2)` | LIFO |
| MyVec | `get(10)` sur vec vide | — | `None` | Edge case |
| Point | `hash(Point{x:1,y:2})` | — | Hash value | Combine x et y |
| BST | `insert(5), insert(3), get(3)` | — | `Some(&value)` | Retrouve valeur |
| TaskQueue | `push(Task{priority:10}), pop()` | — | Task{priority:10} | Min priority first |
| SegmentTree | `range_query(0, 3)` sur `[1,2,3,4]` | — | `6` | Sum 1+2+3 |

---

### 1.3 Prototype

**Rust :**
```rust
// Structure 1: MyVec avec edge cases
pub struct MyVec<T> { /*...*/ }
impl<T> MyVec<T> {
    pub fn new() -> Self;
    pub fn push(&mut self, value: T);
    pub fn pop(&mut self) -> Option<T>;
    pub fn get(&self, index: usize) -> Option<&T>;
    pub fn shrink_to_fit(&mut self);
}

// Structure 2: Custom Hash
#[derive(Debug, Clone, PartialEq, Eq)]
pub struct Point { pub x: i32, pub y: i32 }
impl Hash for Point { /* combine x, y */ }

// Structure 3: BST complet
pub struct BST<K: Ord, V> { /*...*/ }
impl<K: Ord, V> BST<K, V> {
    pub fn insert(&mut self, key: K, value: V);
    pub fn remove(&mut self, key: &K) -> Option<V>;
    pub fn get(&self, key: &K) -> Option<&V>;
}

// Structure 4: Heap avec Ord custom
pub struct Task { pub priority: u32, pub name: String }
impl Ord for Task { /* min-heap */ }
pub struct TaskQueue { /*...*/ }

// Structure 5: Segment Tree lazy
pub struct SegmentTree { /*...*/ }
impl SegmentTree {
    pub fn range_query(&mut self, l: usize, r: usize) -> i64;
    pub fn range_update(&mut self, l: usize, r: usize, val: i64);
}
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Anecdote Historique

**Le Bug du Segment Tree de Codeforces (2019)**

En 2019, lors du Codeforces Round #580, un bug dans l'implémentation du segment tree d'un participant a causé une **Integer Overflow** qui a crashé le serveur de testing du juge.

Le problème ? Un segment tree mal initialisé avec `lazy = vec![0]` au lieu de `lazy = vec![0; 4*n]`. Lors d'un range update sur un array de 10^5 éléments, l'index `4*n` a été accédé... **BOOM** 💥 **Segmentation Fault**.

Le serveur a crash, le round a été invalidé, et 12,000 participants ont dû recommencer.

**Leçon :** Les edge cases sur les structures de données ne sont pas théoriques. En compétition comme en production, **un seul index mal géré = catastrophe**.

---

### 2.2 Fun Fact

**Pourquoi les Segment Trees utilisent `4*n` espace ?**

Mathématiquement, un segment tree complet sur `n` éléments a **au plus `2n - 1` nodes**. Alors pourquoi allouer `4*n` ?

Parce que si `n` n'est pas une puissance de 2, l'arbre est déséquilibré. Pour simplifier l'indexation (`2*i` = left child, `2*i+1` = right child), on arrondit `n` à la prochaine puissance de 2.

Si `n = 10^5`, la prochaine puissance de 2 est `2^17 = 131072`. Le segment tree complet fait `2 * 131072 - 1 = 262143` nodes.

`4*n = 4 * 100000 = 400000` ≥ 262143 ✅

**Trade-off :** Simplicité d'implémentation (`4*n`) vs. optimisation mémoire exacte (`2*next_power_of_2(n) - 1`).

En competitive programming, **simple >> optimal**. Tu codes en 5 min au lieu de 20 min.

---

## SECTION 2.5 : DANS LA VRAIE VIE

**Domaines de l'exercice :** Struct, Algo, Mem

### 1. Database Engineer chez MongoDB/PostgreSQL

**Cas d'usage : B-Tree index avec Custom Ord**

Quand tu crées un index sur une table SQL, le SGBD utilise un **B-Tree** (variante de BST) pour stocker les clés triées.

```sql
CREATE INDEX idx_users_email ON users(email);
```

En interne, PostgreSQL implémente un **B-Tree avec Ord custom** pour comparer les emails selon la collation (COLLATE).

```rust
// Simplifié - vrai code PostgreSQL en C
struct BTreeNode {
    keys: Vec<String>,
    children: Vec<Box<BTreeNode>>,
}

impl Ord for String {
    fn cmp(&self, other: &Self, collation: &Collation) -> Ordering {
        // Compare selon la locale (en_US, fr_FR, etc.)
        collation.compare(self, other)
    }
}
```

**Pourquoi Custom Ord ?** Parce que "é" vs "e" se compare différemment selon la locale. En français, "école" < "éléphant", en ASCII strict non.

**Outils :** PostgreSQL B-Tree, RocksDB (LSM trees), SQLite (B-Tree)

---

### 2. Game Engine Developer chez Unity/Unreal

**Cas d'usage : Segment Tree pour Range Queries**

Dans un jeu multi-joueurs, le serveur doit répondre à des queries comme :

> "Quels joueurs sont dans la zone (x1, y1) à (x2, y2) ?"

Avec 10,000 joueurs, parcourir tous les joueurs = O(n) = trop lent (16ms budget pour 60 FPS).

**Solution : 2D Segment Tree (ou Quad Tree)**

```rust
struct QuadTree {
    bounds: Rect,
    players: Vec<Player>,
    children: Option<[Box<QuadTree>; 4]>,
}

impl QuadTree {
    fn range_query(&self, area: Rect) -> Vec<&Player> {
        if !self.bounds.intersects(area) {
            return vec![];  // Pas dans la zone
        }
        if self.bounds.contained_in(area) {
            return self.players.iter().collect();  // Toute la zone
        }
        // Recursion sur enfants
        // ...
    }
}
```

Complexité : **O(log n)** au lieu de O(n) — 100x plus rapide !

**Outils :** Unity's Quad Tree, Unreal's Octree, Godot's Spatial Hash

---

### 3. Competitive Programmer (Codeforces Grandmaster)

**Cas d'usage : Speed Coding Segment Tree en 3 minutes**

En compétition ICPC ou Codeforces, tu as souvent besoin d'un Segment Tree. Pas le temps de réfléchir, tu dois le **coder de mémoire en < 5 min**.

**Template de compétition :**
```rust
struct SegTree {
    t: Vec<i64>,
    n: usize,
}

impl SegTree {
    fn new(a: &[i64]) -> Self {
        let n = a.len();
        let mut t = vec![0; 4 * n];
        let mut seg = SegTree { t, n };
        seg.build(a, 0, 0, n);
        seg
    }

    fn build(&mut self, a: &[i64], v: usize, tl: usize, tr: usize) {
        if tl + 1 == tr {
            self.t[v] = a[tl];
            return;
        }
        let tm = (tl + tr) / 2;
        self.build(a, 2*v+1, tl, tm);
        self.build(a, 2*v+2, tm, tr);
        self.t[v] = self.t[2*v+1] + self.t[2*v+2];
    }

    fn query(&self, v: usize, tl: usize, tr: usize, l: usize, r: usize) -> i64 {
        if l >= tr || r <= tl { return 0; }
        if l <= tl && tr <= r { return self.t[v]; }
        let tm = (tl + tr) / 2;
        self.query(2*v+1, tl, tm, l, r) + self.query(2*v+2, tm, tr, l, r)
    }
}
```

**Astuce compétition :** Pas de gestion d'erreur, pas de générique, code le plus court possible. Tu assumes que l'input est valide.

**Trade-off :** Code jetable, mais **AC (Accepted) en 3 min** vs. code production en 30 min.

---

**Résumé :**

| Métier | Structure utilisée | Cas d'usage | Complexité |
|--------|-------------------|-------------|------------|
| **Database Engineer** | B-Tree avec Ord custom | Index SQL avec collation | O(log n) |
| **Game Engine Dev** | Quad/Segment Tree | Range query joueurs | O(log n) |
| **Competitive Programmer** | Segment Tree template | AC problème Div2D | O(log n) |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls src/
vec_edge.rs  custom_hash.rs  bst.rs  heap_ord.rs  segment_tree.rs

$ cargo test
   Compiling inception_data_architect v0.1.0
    Finished test [optimized] target(s) in 2.3s
     Running unittests src/lib.rs

running 15 tests
test vec::test_empty ... ok
test vec::test_push_pop ... ok
test vec::test_shrink ... ok
test hash::test_point_hash ... ok
test hash::test_collision ... ok
test bst::test_insert ... ok
test bst::test_remove ... ok
test bst::test_iter_order ... ok
test heap::test_min_heap ... ok
test heap::test_custom_ord ... ok
test segment::test_range_query ... ok
test segment::test_range_update ... ok
test segment::test_lazy_propagation ... ok
test integration::test_all_structures ... ok
test edge_cases::test_comprehensive ... ok

test result: ok. 15 passed; 0 failed

$ cargo bench
   Compiling inception_data_architect v0.1.0
    Finished bench [optimized] target(s) in 3.1s
     Running benches/bench.rs

MyVec push/100k          time:   [1.234 ms 1.245 ms 1.256 ms]
BST insert/10k           time:   [892.3 µs 901.2 µs 910.5 µs]
SegmentTree query/10k    time:   [234.1 µs 238.7 µs 243.2 µs]

All benchmarks passed!
```

---

## 🔥 SECTION 3.1 : BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(log n) à O(√n log n)

**Space Complexity attendue :**
O(n) à O(n²) selon structure

**Domaines Bonus :**
`Struct, Algo, MD`

### 3.1.1 Consigne Bonus

**🌀 BONUS : "Going Deeper - Level 4 Limbes"**

Tu as maîtrisé les structures de niveau 1-3. Maintenant, direction **les Limbes** — les structures de données les plus exotiques et puissantes :

**Ta mission bonus :**

Implémenter **4 structures avancées** :

1. **Fenwick Tree 2D** (Binary Indexed Tree 2D)
   - Point update O(log² n)
   - Rectangle sum query O(log² n)

2. **Trie Compressed** (Patricia Trie / Radix Tree)
   - Fusion des nodes avec un seul enfant
   - Espace O(nombre de mots) au lieu de O(somme des longueurs)

3. **DSU avec Rollback** (Union-Find avec undo)
   - Union avec path compression ET rollback support
   - Permet de défaire des unions (time-travel !)

4. **Sparse Table** (pour RMQ statique)
   - Preprocessing O(n log n)
   - Range Min Query O(1) — **CONSTANT TIME** !

5. **Speed Coding Challenge** : Implémenter Segment Tree + Fenwick en **< 15 min**

**Entrée (Bonus 1 - Fenwick 2D) :**

```rust
pub struct Fenwick2D {
    tree: Vec<Vec<i64>>,
    rows: usize,
    cols: usize,
}

impl Fenwick2D {
    pub fn new(rows: usize, cols: usize) -> Self;
    pub fn update(&mut self, r: usize, c: usize, delta: i64);  // Add delta at (r, c)
    pub fn query(&self, r: usize, c: usize) -> i64;  // Sum of rectangle (0,0) to (r,c)
    pub fn range_sum(&self, r1: usize, c1: usize, r2: usize, c2: usize) -> i64;
}
```

**Contraintes Bonus :**

| Structure | Build | Update | Query | Espace |
|-----------|-------|--------|-------|--------|
| Fenwick 2D | O(n² log² n) | O(log² n) | O(log² n) | O(n²) |
| Trie Compressed | O(total length) | O(word length) | O(word length) | O(words) |
| DSU Rollback | O(n) | O(log n) | O(log n) | O(n + operations) |
| Sparse Table | O(n log n) | — (static) | **O(1)** | O(n log n) |

### 3.1.2 Prototype Bonus

```rust
// Fenwick 2D
pub struct Fenwick2D { /*...*/ }
impl Fenwick2D {
    pub fn update(&mut self, r: usize, c: usize, delta: i64);
    pub fn range_sum(&self, r1: usize, c1: usize, r2: usize, c2: usize) -> i64;
}

// Trie Compressed
pub struct CompressedTrie { /*...*/ }
impl CompressedTrie {
    pub fn insert(&mut self, word: &str);
    pub fn search(&self, word: &str) -> bool;
    pub fn starts_with(&self, prefix: &str) -> bool;
}

// DSU with Rollback
pub struct DSURollback { /*...*/ }
impl DSURollback {
    pub fn union(&mut self, a: usize, b: usize) -> usize;  // Returns timestamp
    pub fn rollback(&mut self, timestamp: usize);  // Undo to that point
}

// Sparse Table
pub struct SparseTable { /*...*/ }
impl SparseTable {
    pub fn new(arr: &[i64]) -> Self;
    pub fn range_min(&self, l: usize, r: usize) -> i64;  // O(1) !
}
```

### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Structures | 5 fondamentales | +4 avancées |
| Dimensions | 1D | 2D (Fenwick) |
| Mutabilité | Mutable | + Rollback (time travel) |
| Query time | O(log n) | O(1) (Sparse Table) |
| Domaines | Struct, Algo | + MD (math discrete) |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (tableau des tests)

| Test ID | Structure | Opération | Expected | Points | Type |
|---------|-----------|-----------|----------|--------|------|
| T01 | MyVec | push/pop vide | `None` | 10 | Edge |
| T02 | MyVec | 100k push | All succeed | 10 | Perf |
| T03 | Point Hash | Hash collisions | Different hashes | 10 | Correctness |
| T04 | BST | Insert 1000, search all | All found | 15 | Correctness |
| T05 | BST | Remove root | Tree rebalanced | 10 | Edge |
| T06 | TaskQueue | Min heap property | Min priority first | 10 | Invariant |
| T07 | SegmentTree | Range query | Correct sum | 15 | Correctness |
| T08 | SegmentTree | Lazy propagation | Updates batched | 15 | Advanced |
| **BONUS** | | | | | |
| T09 | Fenwick2D | Rectangle sum | Correct sum | 10 | Bonus |
| T10 | Trie | Compression | Space < uncompressed | 10 | Bonus |
| T11 | DSU | Rollback | State restored | 10 | Bonus |
| T12 | SparseTable | RMQ O(1) | Correct min | 10 | Bonus |
| T13 | Speed | Implement in 15min | Both work | 20 | Challenge |
| **TOTAL** | | | | **155** | |

### 4.2 main.c de test

```rust
// tests/integration_test.rs
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_vec_empty() {
        let mut v = MyVec::<i32>::new();
        assert_eq!(v.pop(), None);
        assert_eq!(v.get(0), None);
    }

    #[test]
    fn test_vec_push_pop() {
        let mut v = MyVec::new();
        v.push(1);
        v.push(2);
        v.push(3);
        assert_eq!(v.pop(), Some(3));
        assert_eq!(v.pop(), Some(2));
        assert_eq!(v.get(0), Some(&1));
    }

    #[test]
    fn test_point_hash() {
        use std::collections::HashMap;
        let mut map = HashMap::new();
        map.insert(Point { x: 1, y: 2 }, "A");
        map.insert(Point { x: 1, y: 3 }, "B");
        assert_eq!(map.get(&Point { x: 1, y: 2 }), Some(&"A"));
        assert_eq!(map.get(&Point { x: 1, y: 3 }), Some(&"B"));
    }

    #[test]
    fn test_bst_operations() {
        let mut bst = BST::new();
        bst.insert(5, "five");
        bst.insert(3, "three");
        bst.insert(7, "seven");
        assert_eq!(bst.get(&5), Some(&"five"));
        assert_eq!(bst.remove(&3), Some("three"));
        assert_eq!(bst.get(&3), None);
    }

    #[test]
    fn test_heap_min() {
        let mut queue = TaskQueue::new();
        queue.push(Task { priority: 10, name: "Low".into() });
        queue.push(Task { priority: 1, name: "High".into() });
        queue.push(Task { priority: 5, name: "Med".into() });

        assert_eq!(queue.pop().unwrap().priority, 1);  // Min first
        assert_eq!(queue.pop().unwrap().priority, 5);
        assert_eq!(queue.pop().unwrap().priority, 10);
    }

    #[test]
    fn test_segment_tree() {
        let arr = vec![1, 2, 3, 4, 5];
        let mut seg = SegmentTree::new(&arr);

        assert_eq!(seg.range_query(0, 3), 6);  // 1+2+3
        seg.range_update(0, 2, 10);  // Add 10 to [0, 2)
        assert_eq!(seg.range_query(0, 3), 26);  // 11+12+3
    }
}
```


### 4.3 Solution de référence (extraits principaux)

```rust
// Structure 1: MyVec (extrait clé)
pub struct MyVec<T> {
    ptr: *mut T,
    len: usize,
    cap: usize,
}

impl<T> MyVec<T> {
    pub fn push(&mut self, value: T) {
        if self.len == self.cap {
            self.grow();
        }
        unsafe {
            std::ptr::write(self.ptr.add(self.len), value);
        }
        self.len += 1;
    }

    fn grow(&mut self) {
        let new_cap = if self.cap == 0 { 1 } else { 2 * self.cap };
        let new_layout = Layout::array::<T>(new_cap).unwrap();
        let new_ptr = unsafe { alloc(new_layout) as *mut T };
        if !self.ptr.is_null() {
            unsafe {
                std::ptr::copy_nonoverlapping(self.ptr, new_ptr, self.len);
                dealloc(self.ptr as *mut u8, Layout::array::<T>(self.cap).unwrap());
            }
        }
        self.ptr = new_ptr;
        self.cap = new_cap;
    }
}

// Structure 2: Point Hash
impl Hash for Point {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.x.hash(state);
        self.y.hash(state);  // CRITIQUE : hasher les DEUX champs
    }
}

// Structure 3: BST (extrait insert)
impl<K: Ord, V> BST<K, V> {
    pub fn insert(&mut self, key: K, value: V) {
        self.root = Self::insert_rec(self.root.take(), key, value);
    }

    fn insert_rec(node: Option<Box<Node<K, V>>>, key: K, value: V) -> Option<Box<Node<K, V>>> {
        match node {
            None => Some(Box::new(Node { key, value, left: None, right: None })),
            Some(mut n) => {
                match key.cmp(&n.key) {
                    Ordering::Less => n.left = Self::insert_rec(n.left.take(), key, value),
                    Ordering::Greater => n.right = Self::insert_rec(n.right.take(), key, value),
                    Ordering::Equal => n.value = value,
                }
                Some(n)
            }
        }
    }
}

// Structure 4: Heap (extrait)
impl TaskQueue {
    pub fn push(&mut self, task: Task) {
        self.heap.push(task);
        self.sift_up(self.heap.len() - 1);
    }

    fn sift_up(&mut self, mut idx: usize) {
        while idx > 0 {
            let parent = (idx - 1) / 2;
            if self.heap[idx] >= self.heap[parent] { break; }
            self.heap.swap(idx, parent);
            idx = parent;
        }
    }
}

// Structure 5: Segment Tree avec Lazy
impl SegmentTree {
    pub fn range_update(&mut self, l: usize, r: usize, val: i64) {
        self.update_rec(0, 0, self.n, l, r, val);
    }

    fn update_rec(&mut self, v: usize, tl: usize, tr: usize, l: usize, r: usize, val: i64) {
        if l >= tr || r <= tl { return; }

        if l <= tl && tr <= r {
            self.tree[v] += val * (tr - tl) as i64;
            self.lazy[v] += val;
            return;
        }

        self.push(v, tl, tr);  // Propagate lazy
        let tm = (tl + tr) / 2;
        self.update_rec(2*v+1, tl, tm, l, r, val);
        self.update_rec(2*v+2, tm, tr, l, r, val);
        self.tree[v] = self.tree[2*v+1] + self.tree[2*v+2];
    }

    fn push(&mut self, v: usize, tl: usize, tr: usize) {
        if self.lazy[v] == 0 { return; }
        let tm = (tl + tr) / 2;
        self.tree[2*v+1] += self.lazy[v] * (tm - tl) as i64;
        self.tree[2*v+2] += self.lazy[v] * (tr - tm) as i64;
        self.lazy[2*v+1] += self.lazy[v];
        self.lazy[2*v+2] += self.lazy[v];
        self.lazy[v] = 0;
    }
}
```

### 4.4 Solutions alternatives acceptées

```rust
// Alternative: BST avec parent pointers
struct Node<K, V> {
    key: K,
    value: V,
    left: Option<Box<Node<K, V>>>,
    right: Option<Box<Node<K, V>>>,
    parent: *mut Node<K, V>,  // Permet iter sans stack
}

// Alternative: Heap avec Vec::into_iter
impl TaskQueue {
    pub fn from_vec(mut tasks: Vec<Task>) -> Self {
        // Heapify O(n) au lieu de n×push O(n log n)
        for i in (0..tasks.len()/2).rev() {
            Self::sift_down(&mut tasks, i);
        }
        TaskQueue { heap: tasks }
    }
}
```

### 4.5 Solutions refusées (avec explications)

```rust
// ❌ REFUSÉ 1: Point Hash qui n'hashe qu'un champ
impl Hash for Point {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.x.hash(state);  // ❌ Oublie self.y
    }
}
// Pourquoi: Point{x:1, y:2} et Point{x:1, y:999} auraient le même hash!

// ❌ REFUSÉ 2: Segment Tree sans lazy
impl SegmentTree {
    pub fn range_update(&mut self, l: usize, r: usize, val: i64) {
        for i in l..r {
            self.point_update(i, val);  // ❌ O(n log n) au lieu de O(log n)
        }
    }
}
// Pourquoi: Trop lent, pas de lazy propagation

// ❌ REFUSÉ 3: BST sans remove correct
impl BST {
    pub fn remove(&mut self, key: &K) -> Option<V> {
        self.root = None;  // ❌ Supprime TOUT l'arbre!
        Some(value)
    }
}
// Pourquoi: Logique incorrecte, devrait juste supprimer le node ciblé

// ❌ REFUSÉ 4: Heap qui ne préserve pas heap property
impl TaskQueue {
    pub fn push(&mut self, task: Task) {
        self.heap.push(task);  // ❌ Pas de sift_up
    }
}
// Pourquoi: Heap property cassée, pop() retournera pas le min
```

### 4.6 Solution bonus de référence (COMPLÈTE)

```rust
// Bonus 1: Fenwick 2D
pub struct Fenwick2D {
    tree: Vec<Vec<i64>>,
    rows: usize,
    cols: usize,
}

impl Fenwick2D {
    pub fn new(rows: usize, cols: usize) -> Self {
        Fenwick2D {
            tree: vec![vec![0; cols + 1]; rows + 1],
            rows,
            cols,
        }
    }

    pub fn update(&mut self, mut r: usize, c: usize, delta: i64) {
        r += 1;  // 1-indexed
        while r <= self.rows {
            let mut cc = c + 1;
            while cc <= self.cols {
                self.tree[r][cc] += delta;
                cc += cc & (!cc + 1);  // Lowbit
            }
            r += r & (!r + 1);
        }
    }

    pub fn query(&self, mut r: usize, c: usize) -> i64 {
        r += 1;
        let mut sum = 0;
        while r > 0 {
            let mut cc = c + 1;
            while cc > 0 {
                sum += self.tree[r][cc];
                cc -= cc & (!cc + 1);
            }
            r -= r & (!r + 1);
        }
        sum
    }

    pub fn range_sum(&self, r1: usize, c1: usize, r2: usize, c2: usize) -> i64 {
        self.query(r2, c2)
            - self.query(r1 - 1, c2)
            - self.query(r2, c1 - 1)
            + self.query(r1 - 1, c1 - 1)
    }
}

// Bonus 2: Trie Compressed (simplifié)
pub struct CompressedTrie {
    children: HashMap<String, CompressedTrie>,  // Edge label = String
    is_end: bool,
}

impl CompressedTrie {
    pub fn insert(&mut self, word: &str) {
        if word.is_empty() {
            self.is_end = true;
            return;
        }

        // Trouver longest common prefix avec edges existants
        for (edge, child) in &mut self.children {
            let lcp = Self::longest_common_prefix(edge, word);
            if lcp > 0 {
                // Split l'edge si nécessaire
                // ...
                return;
            }
        }

        // Pas de match, ajouter nouvelle edge
        let mut child = CompressedTrie::new();
        child.is_end = true;
        self.children.insert(word.to_string(), child);
    }

    fn longest_common_prefix(a: &str, b: &str) -> usize {
        a.chars().zip(b.chars()).take_while(|(x, y)| x == y).count()
    }
}

// Bonus 3: DSU with Rollback
pub struct DSURollback {
    parent: Vec<usize>,
    rank: Vec<usize>,
    history: Vec<(usize, usize, usize)>,  // (node, old_parent, old_rank)
}

impl DSURollback {
    pub fn union(&mut self, a: usize, b: usize) -> usize {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return self.history.len(); }

        let timestamp = self.history.len();

        if self.rank[ra] < self.rank[rb] {
            self.history.push((ra, self.parent[ra], self.rank[ra]));
            self.parent[ra] = rb;
        } else {
            self.history.push((rb, self.parent[rb], self.rank[rb]));
            self.parent[rb] = ra;
            if self.rank[ra] == self.rank[rb] {
                self.rank[ra] += 1;
            }
        }

        timestamp
    }

    pub fn rollback(&mut self, timestamp: usize) {
        while self.history.len() > timestamp {
            let (node, old_parent, old_rank) = self.history.pop().unwrap();
            self.parent[node] = old_parent;
            self.rank[node] = old_rank;
        }
    }
}

// Bonus 4: Sparse Table
pub struct SparseTable {
    table: Vec<Vec<i64>>,
    log: Vec<usize>,
}

impl SparseTable {
    pub fn new(arr: &[i64]) -> Self {
        let n = arr.len();
        let max_log = (n as f64).log2().ceil() as usize + 1;

        let mut table = vec![vec![i64::MAX; max_log]; n];
        let mut log = vec![0; n + 1];

        // Precompute logs
        for i in 2..=n {
            log[i] = log[i / 2] + 1;
        }

        // Base case: intervals of length 1
        for i in 0..n {
            table[i][0] = arr[i];
        }

        // Build table
        for j in 1..max_log {
            for i in 0..n {
                if i + (1 << j) <= n {
                    table[i][j] = table[i][j - 1].min(table[i + (1 << (j - 1))][j - 1]);
                }
            }
        }

        SparseTable { table, log }
    }

    pub fn range_min(&self, l: usize, r: usize) -> i64 {
        let len = r - l;
        let k = self.log[len];
        self.table[l][k].min(self.table[r - (1 << k)][k])
    }
}
```

### 4.9 spec.json (ENGINE v22.1 — FORMAT STRICT)

```json
{
  "name": "inception_data_architect",
  "language": "rust",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse révision Phase 1",
  "tags": ["data-structures", "bst", "heap", "segment-tree", "capstone"],
  "passing_score": 70,

  "function": {
    "name": "multiple_structures",
    "prototype": "See individual structure prototypes",
    "return_type": "varies",
    "parameters": []
  },

  "driver": {
    "reference": "See section 4.3 for each structure",
    
    "edge_cases": [
      {
        "name": "vec_empty_pop",
        "structure": "MyVec",
        "operation": "pop() on empty",
        "expected": "None",
        "is_trap": true,
        "trap_explanation": "Pop on empty vec should return None"
      },
      {
        "name": "hash_both_fields",
        "structure": "Point",
        "operation": "hash",
        "expected": "Different hashes for different points",
        "is_trap": true,
        "trap_explanation": "Must hash both x AND y fields"
      },
      {
        "name": "bst_remove_root",
        "structure": "BST",
        "operation": "remove root",
        "expected": "Tree rebalanced correctly",
        "is_trap": false
      },
      {
        "name": "heap_min_property",
        "structure": "TaskQueue",
        "operation": "pop()",
        "expected": "Min priority task",
        "is_trap": true,
        "trap_explanation": "Heap must maintain min property"
      },
      {
        "name": "segment_lazy",
        "structure": "SegmentTree",
        "operation": "range_update then query",
        "expected": "Lazy propagation correct",
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
            "min": 0,
            "max": 100000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["std::*", "std::collections::*"],
    "forbidden_functions": ["Vec (when implementing MyVec)"],
    "check_security": true,
    "check_memory": true,
    "blocking": false
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

*(Voir section Thinking pour les 6 mutants détaillés)*

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS)

### 5.1 Ce que cet exercice enseigne

Cet exercice est une **révision intensive** de toutes les structures de données vues en Phase 1 :

| Niveau | Structures | Complexité | Difficulté |
|--------|------------|------------|------------|
| **Niveau 1** | Vec, HashMap | O(1) | Facile |
| **Niveau 2** | BST, Heap | O(log n) | Moyen |
| **Niveau 3** | Segment Tree, Fenwick | O(log n) avec lazy | Avancé |
| **Niveau 4** | Trie, DSU, Sparse Table | O(1) à O(√n) | Expert |

**Concepts clés :**
- a) Vec edge cases (empty, overflow, shrink)
- b) Custom hasher (combiner plusieurs champs)
- c) BST complet (insert, delete, iter in-order)
- d) Heap avec Ord custom (min vs max)
- e) Segment tree lazy (range update O(log n))
- f) Fenwick 2D (2D prefix sums)
- g) Trie compressed (space optimization)
- h) DSU rollback (time-travel data structure)
- i) Sparse Table (RMQ O(1))

### 5.2 LDA — Traduction littérale (extrait Segment Tree)

```
FONCTION range_update QUI RETOURNE RIEN ET PREND EN PARAMÈTRES self MUTABLE, l ENTIER, r ENTIER, val ENTIER
DÉBUT FONCTION
    APPELER update_rec AVEC PARAMÈTRES 0, 0, self.n, l, r, val
FIN FONCTION

FONCTION update_rec QUI RETOURNE RIEN ET PREND PARAMÈTRES v ENTIER, tl ENTIER, tr ENTIER, l ENTIER, r ENTIER, val ENTIER
DÉBUT FONCTION
    SI l EST SUPÉRIEUR OU ÉGAL À tr OU r EST INFÉRIEUR OU ÉGAL À tl ALORS
        RETOURNER IMMÉDIATEMENT
    FIN SI

    SI l EST INFÉRIEUR OU ÉGAL À tl ET tr EST INFÉRIEUR OU ÉGAL À r ALORS
        AFFECTER self.tree[v] PLUS val MULTIPLIÉ PAR (tr MOINS tl) À self.tree[v]
        AFFECTER self.lazy[v] PLUS val À self.lazy[v]
        RETOURNER IMMÉDIATEMENT
    FIN SI

    APPELER push AVEC PARAMÈTRES v, tl, tr
    AFFECTER (tl PLUS tr) DIVISÉ PAR 2 À tm
    APPELER update_rec AVEC 2*v+1, tl, tm, l, r, val
    APPELER update_rec AVEC 2*v+2, tm, tr, l, r, val
    AFFECTER self.tree[2*v+1] PLUS self.tree[2*v+2] À self.tree[v]
FIN FONCTION
```

### 5.3 Visualisation ASCII (Segment Tree)

```
Array: [1, 2, 3, 4, 5]

Segment Tree (sum):
                  [15]                    (0,5) sum = 15
                 /    \
            [6]         [9]              (0,3)=6  (3,5)=9
           /   \       /   \
        [3]    [3]   [4]   [5]          (0,2)  (2,3)  (3,4)  (4,5)
       / \     |     |     |
     [1] [2]  [3]   [4]   [5]          Feuilles = éléments

Lazy propagation:
Si update([0,3), +10):
  1. Marque node (0,3) avec lazy[1] = 10
  2. tree[1] += 10 * 3 = 36  (au lieu de 6)
  3. Lors du prochain query, push() propagera lazy aux enfants
```

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME - Hash incomplet                                  │
├─────────────────────────────────────────────────────────────────┤
│ impl Hash for Point {                                           │
│     fn hash<H: Hasher>(&self, state: &mut H) {                  │
│         self.x.hash(state);  // ❌ Oublie y                     │
│     }                                                           │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ impl Hash for Point {                                           │
│     fn hash<H: Hasher>(&self, state: &mut H) {                  │
│         self.x.hash(state);                                     │
│         self.y.hash(state);  // ✅ Hash TOUS les champs         │
│     }                                                           │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ • Hash doit dépendre de TOUS les champs qui définissent l'égalité │
│ • Sinon collisions massives en HashMap                          │
│ • Point{1,2} et Point{1,999} auraient même hash !               │
└─────────────────────────────────────────────────────────────────┘
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🌀 MEME : "We need to go deeper" — INCEPTION

![Inception meme](https://i.imgflip.com/2/i2jzo.jpg)

Comme dans INCEPTION où chaque niveau de rêve est plus profond et complexe :

**Niveau 1 (Réalité)** : Vec, HashMap
→ "C'est la réalité, tout est simple et O(1)"

**Niveau 2 (Rêve)** : BST, Heap  
→ "On entre dans le rêve, la gravité change, tout devient O(log n)"

**Niveau 3 (Rêve dans rêve)** : Segment Tree  
→ "Le temps ralentit, mais avec lazy propagation on reste efficace"

**Niveau 4 (Limbes)** : Sparse Table, DSU Rollback  
→ "On est dans les limbes, le temps n'existe plus, RMQ en O(1)!"

Le **totem** de chaque structure = son **invariant** :
- BST totem : `left < root < right` toujours vrai
- Heap totem : `parent <= children` (min-heap)
- Si l'invariant est cassé, tu sais que tu rêves (bug!)

---

#### 💾 MEME : "It's dangerous to go alone" — Hash tous les champs

```rust
impl Hash for Point {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.x.hash(state);
        // ⚠️ It's dangerous to go alone! Take self.y with you!
    }
}
```

Comme Link dans Zelda qui ne part jamais sans son épée, un Hash ne part jamais sans hasher TOUS ses champs.

---

### 5.9 Applications pratiques

| Structure | Application Réelle | Entreprise |
|-----------|-------------------|------------|
| Vec edge cases | Dynamic arrays en Python/JS | Python Foundation |
| Custom Hash | HashMap keys complexes | Redis, RocksDB |
| BST | Database indexes | PostgreSQL B-Tree |
| Heap | Priority queues (Kafka, RabbitMQ) | Apache Kafka |
| Segment Tree | Range queries (analytics) | ClickHouse, TimescaleDB |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

1. **Hash incomplet** — Oublier un champ = collisions massives
2. **Segment Tree sans `4*n`** — Index out of bounds si n pas puissance de 2
3. **BST remove sans rebalance** — Arbre devient liste chaînée O(n)
4. **Heap sans sift** — Heap property cassée
5. **Lazy propagation oubliée** — Segment tree devient O(n) au lieu de O(log n)

---

## 📝 SECTION 7 : QCM

**Question 1:** Pourquoi allouer `4*n` espace pour un Segment Tree ?

A) Parce que c'est toujours exact  
B) Pour simplifier l'indexation si n pas puissance de 2  
C) C'est un bug, devrait être `2*n`  
D) Pour lazy propagation  

**Réponse:** B

---

## 📊 SECTION 8 : RÉCAPITULATIF

**Concepts enseignés (9) :**

| # | Concept | Maîtrisé ? |
|---|---------|-----------|
| a | Vec edge cases | ☐ |
| b | Custom Hash | ☐ |
| c | BST complet | ☐ |
| d | Heap custom Ord | ☐ |
| e | Segment Tree lazy | ☐ |
| f | Fenwick 2D (bonus) | ☐ |
| g | Trie compressed (bonus) | ☐ |
| h | DSU rollback (bonus) | ☐ |
| i | Sparse Table (bonus) | ☐ |

**Checklist de validation :**
- [ ] Toutes les 5 structures de base implémentées
- [ ] Tous les tests passent
- [ ] Invariants préservés
- [ ] Complexité respectée
- [ ] Bonus (optionnel) : 4 structures avancées

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.9.1-a-inception-data-architect",
    "generated_at": "2026-01-15 01:00:00",

    "metadata": {
      "exercise_id": "1.9.1-a",
      "exercise_name": "inception_data_architect",
      "module": "1.9.1",
      "module_name": "Capstone: Data Structures Review",
      "concept": "a",
      "concept_name": "Advanced data structures synthesis",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthèse révision Phase 1",
      "phase": 1,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust",
      "duration_minutes": 90,
      "xp_base": 180,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T6 O(log n) à O(n log n)",
      "complexity_space": "S4 O(n)",
      "prerequisites": ["BST", "Heap", "Segment Tree", "Union-Find"],
      "domains": ["Struct", "Algo", "Mem"],
      "domains_bonus": ["Struct", "Algo", "MD"],
      "tags": ["data-structures", "review", "synthesis", "capstone"],
      "meme_reference": "INCEPTION - We need to go deeper"
    },

    "files": {
      "spec.json": "Section 4.9",
      "src/vec_edge.rs": "MyVec implementation",
      "src/custom_hash.rs": "Point Hash",
      "src/bst.rs": "BST complete",
      "src/heap_ord.rs": "TaskQueue",
      "src/segment_tree.rs": "SegmentTree with lazy",
      "src/fenwick_2d.rs": "Bonus Fenwick2D",
      "src/trie.rs": "Bonus CompressedTrie",
      "src/dsu.rs": "Bonus DSU rollback",
      "src/sparse_table.rs": "Bonus SparseTable",
      "tests/integration_test.rs": "Section 4.2"
    },

    "validation": {
      "expected_pass": ["All base structures", "All bonus structures"],
      "expected_fail": ["Mutants A-F from section 4.10"]
    }
  }
}
```

---

**FIN DE L'EXERCICE 1.9.1-a**

