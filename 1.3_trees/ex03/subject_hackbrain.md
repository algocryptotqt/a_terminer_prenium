<thinking>
## Analyse du Concept
- Concept : Treaps (Tree + Heap) et Splay Trees
- Phase demandée : 1 (intermédiaire avancé)
- Adapté ? OUI - Concepts randomisés et auto-ajustants, fondamentaux pour comprendre l'amortissement

## Combo Base + Bonus
- Exercice de base : Treap avec insert/delete/split/merge + Splay Tree basique
- Bonus : Implicit Treap (pour séquences) + Link-Cut Trees (très avancé)
- Palier bonus : 💀 Expert (Implicit), 🧠 Génie (Link-Cut)
- Progression logique ? OUI - Base = structures, Bonus = applications avancées

## Prérequis & Difficulté
- Prérequis réels : BST, rotations, concept de heap, probabilités basiques
- Difficulté estimée : 7/10 (base), 9/10 (implicit), 11/10 (link-cut)
- Cohérent avec phase 1 ? Limite haute, ex avancé du module

## Aspect Fun/Culture
- Contexte choisi : Casino / Las Vegas - La chance rencontre la stratégie
- MEME mnémotechnique : "The house always wins" - sauf qu'ici, randomisé = équilibré en moyenne
- Pourquoi c'est fun : Treap = chance (priorités random), Splay = stratégie (récemment accédé en haut)

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Priority) : Treap sans respect de la propriété heap sur les priorités
2. Mutant B (Rotation) : Splay avec mauvais type de rotation (zig vs zig-zig)
3. Mutant C (Split) : Split qui ne sépare pas correctement autour de la clé
4. Mutant D (Merge) : Merge qui viole la propriété BST
5. Mutant E (Splay) : Splay qui ne remonte pas jusqu'à la racine

## Verdict
VALIDE - L'analogie casino est parfaite pour randomisé vs déterministe
</thinking>

---

# Exercice 1.3.3-a : vegas_random_trees

**Module :**
1.3.3 — Randomized & Self-Adjusting Trees

**Concept :**
a — Treaps et Splay Trees

**Difficulté :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
1 — Concept isolé

**Langage :**
Rust Edition 2024, C (c17)

**Prérequis :**
- Binary Search Tree (exercice 1.3.0)
- Rotations (exercice 1.3.1)
- Concept de Heap (max-heap/min-heap)
- Notions de probabilités

**Domaines :**
Struct, Mem, MD, Probas

**Durée estimée :**
90 min

**XP Base :**
250

**Complexité :**
T[3] O(log n) espéré/amorti × S[2] O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `vegas_random_trees.c`, `vegas_random_trees.h`

**Fonctions autorisées :**
- C : `malloc`, `free`, `rand`, `srand`, `time`
- Rust : `Box::new`, `Option`, `rand` crate, `std::cmp::Ordering`

**Fonctions interdites :**
- Bibliothèques d'arbres externes

### 1.2 Consigne

**🎰 LAS VEGAS TREES — Quand la Chance Rencontre la Stratégie**

*"In Vegas, the house always wins. But in Treaps, randomness guarantees balance. And in Splay Trees, the hot hand gets to the top..."*

Bienvenue au **Casino des Structures de Données** ! Ici, deux tables t'attendent :

🎲 **TABLE 1 : LE TREAP (Tree + Heap)**
- Chaque nœud a une **clé** (pour le BST) ET une **priorité aléatoire** (pour le Heap)
- L'arbre est un BST par les clés, un Heap par les priorités
- La chance (random priority) garantit l'équilibre en moyenne !

🃏 **TABLE 2 : LE SPLAY TREE**
- Pas de hasard, mais de la **stratégie**
- Chaque accès **"splay"** le nœud jusqu'à la racine
- Les éléments récemment accédés sont toujours en haut
- Complexité **amortie** O(log n)

**Ta mission :**

### Partie 1 : Treap (The Gambler's Tree)

Implémenter `VegasTreap<K, V>` :

| Fonction | Description | Complexité |
|----------|-------------|------------|
| `insert(key, value)` | Insert + rotation pour maintenir heap | O(log n) espéré |
| `delete(key)` | Suppression + rotations | O(log n) espéré |
| `split(key)` | Divise en deux treaps | O(log n) espéré |
| `merge(t1, t2)` | Fusionne deux treaps | O(log n) espéré |
| `search(key)` | Recherche standard | O(log n) espéré |

**La règle du Treap :**
```
Pour tout nœud N:
- Propriété BST : left.key < N.key < right.key
- Propriété Heap : N.priority > children.priority (max-heap)
```

### Partie 2 : Splay Tree (The Hot Hand Tree)

Implémenter `SplayTree<K, V>` :

| Fonction | Description | Complexité |
|----------|-------------|------------|
| `insert(key, value)` | Insert puis splay | O(log n) amorti |
| `delete(key)` | Splay puis delete | O(log n) amorti |
| `search(key)` | Splay le nœud trouvé | O(log n) amorti |
| `splay(key)` | Remonte le nœud à la racine | O(log n) amorti |

**Les 3 rotations Splay :**
- **Zig** : Une rotation (quand parent = root)
- **Zig-Zig** : Deux rotations même direction
- **Zig-Zag** : Deux rotations directions opposées

### 1.3 Prototype

**Rust :**
```rust
use rand::Rng;

// ========== TREAP ==========
pub struct VegasTreap<K: Ord, V> {
    root: Option<Box<TreapNode<K, V>>>,
}

struct TreapNode<K: Ord, V> {
    key: K,
    value: V,
    priority: u64,  // Random priority
    left: Option<Box<TreapNode<K, V>>>,
    right: Option<Box<TreapNode<K, V>>>,
}

impl<K: Ord, V> VegasTreap<K, V> {
    pub fn new() -> Self;

    pub fn insert(&mut self, key: K, value: V);
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn remove(&mut self, key: &K) -> Option<V>;

    /// Split treap into (< key, >= key)
    pub fn split(self, key: &K) -> (Self, Self);

    /// Merge two treaps (all keys in left < all keys in right)
    pub fn merge(left: Self, right: Self) -> Self;

    fn rotate_left(node: Box<TreapNode<K, V>>) -> Box<TreapNode<K, V>>;
    fn rotate_right(node: Box<TreapNode<K, V>>) -> Box<TreapNode<K, V>>;

    pub fn is_valid_treap(&self) -> bool;
}

// ========== SPLAY TREE ==========
pub struct SplayTree<K: Ord, V> {
    root: Option<Box<SplayNode<K, V>>>,
}

struct SplayNode<K: Ord, V> {
    key: K,
    value: V,
    left: Option<Box<SplayNode<K, V>>>,
    right: Option<Box<SplayNode<K, V>>>,
}

impl<K: Ord, V> SplayTree<K, V> {
    pub fn new() -> Self;

    pub fn insert(&mut self, key: K, value: V);
    pub fn get(&mut self, key: &K) -> Option<&V>;  // Note: &mut self car splay modifie
    pub fn remove(&mut self, key: &K) -> Option<V>;

    /// Splay operation - moves key to root
    fn splay(&mut self, key: &K);

    /// Zig rotation (single)
    fn zig_left(node: Box<SplayNode<K, V>>) -> Box<SplayNode<K, V>>;
    fn zig_right(node: Box<SplayNode<K, V>>) -> Box<SplayNode<K, V>>;

    /// Zig-Zig (two same direction)
    fn zig_zig_left(node: Box<SplayNode<K, V>>) -> Box<SplayNode<K, V>>;
    fn zig_zig_right(node: Box<SplayNode<K, V>>) -> Box<SplayNode<K, V>>;

    /// Zig-Zag (two opposite direction)
    fn zig_zag_left_right(node: Box<SplayNode<K, V>>) -> Box<SplayNode<K, V>>;
    fn zig_zag_right_left(node: Box<SplayNode<K, V>>) -> Box<SplayNode<K, V>>;
}
```

**C :**
```c
// ========== TREAP ==========
typedef struct s_treap_node {
    int                     key;
    char                    *value;
    unsigned long           priority;
    struct s_treap_node     *left;
    struct s_treap_node     *right;
} t_treap_node;

typedef struct s_vegas_treap {
    t_treap_node    *root;
    size_t          size;
} t_vegas_treap;

t_vegas_treap   *treap_new(void);
void            treap_free(t_vegas_treap *treap);
void            treap_insert(t_vegas_treap *treap, int key, char *value);
char            *treap_search(t_vegas_treap *treap, int key);
int             treap_delete(t_vegas_treap *treap, int key);
void            treap_split(t_vegas_treap *treap, int key,
                           t_vegas_treap **left, t_vegas_treap **right);
t_vegas_treap   *treap_merge(t_vegas_treap *left, t_vegas_treap *right);
int             treap_is_valid(t_vegas_treap *treap);

// ========== SPLAY TREE ==========
typedef struct s_splay_node {
    int                     key;
    char                    *value;
    struct s_splay_node     *left;
    struct s_splay_node     *right;
    struct s_splay_node     *parent;  // Useful for splay
} t_splay_node;

typedef struct s_splay_tree {
    t_splay_node    *root;
    size_t          size;
} t_splay_tree;

t_splay_tree    *splay_new(void);
void            splay_free(t_splay_tree *tree);
void            splay_insert(t_splay_tree *tree, int key, char *value);
char            *splay_search(t_splay_tree *tree, int key);
int             splay_delete(t_splay_tree *tree, int key);
void            splay_splay(t_splay_tree *tree, t_splay_node *node);
```

### 1.2.2 Énoncé Académique

**Treap (Aragon & Seidel, 1989)**

Un Treap est une structure de données qui combine les propriétés d'un BST et d'un Heap :
- **BST sur les clés** : Pour tout nœud, clés à gauche < clé du nœud < clés à droite
- **Max-Heap sur les priorités** : La priorité d'un nœud ≥ priorités de ses enfants

Si les priorités sont choisies aléatoirement, l'arbre a une hauteur espérée O(log n).

**Splay Tree (Sleator & Tarjan, 1985)**

Un Splay Tree est un BST auto-ajustant qui "splay" (remonte) chaque nœud accédé jusqu'à la racine. Les opérations de splay utilisent trois types de rotations :

1. **Zig** : Simple rotation quand le parent est la racine
2. **Zig-Zig** : Deux rotations quand nœud et parent sont du même côté
3. **Zig-Zag** : Deux rotations quand nœud et parent sont de côtés opposés

La complexité amortie de toute séquence de m opérations est O(m log n).

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Treap vs Autres Arbres

| Aspect | Treap | AVL | Red-Black |
|--------|-------|-----|-----------|
| Équilibre | Probabiliste | Déterministe | Déterministe |
| Implémentation | Plus simple | Modéré | Complexe |
| Constante | Grande | Petite | Petite |
| Split/Merge | Natif O(log n) | Complexe | Complexe |

### 2.2 Splay Tree Magic

Le Splay Tree a une propriété remarquable : les éléments fréquemment accédés migrent naturellement vers la racine, créant un cache naturel !

**Théorème de l'Optimalité Statique :** Un Splay Tree est au plus un facteur constant plus lent que l'arbre de recherche optimal pour toute distribution d'accès.

### 2.5 DANS LA VRAIE VIE

| Métier | Structure | Utilisation |
|--------|-----------|-------------|
| **Database Engineer** | Treap | Index randomisé |
| **Cache Developer** | Splay Tree | Cache LRU amélioré |
| **Competitive Programmer** | Implicit Treap | Opérations sur séquences |
| **Network Engineer** | Link-Cut Trees | Arbres dynamiques |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
vegas_random_trees.c  vegas_random_trees.h  main.c  Cargo.toml  src/

$ gcc -Wall -Wextra -Werror -std=c17 vegas_random_trees.c main.c -o test_c

$ ./test_c
=== Test Vegas Treap ===
Insert (50, 0.89): Root
Insert (30, 0.45): Goes left, no rotation needed
Insert (70, 0.92): Goes right, rotation! 70 becomes root
Insert (20, 0.67): Goes left of 50, rotation! 20 swaps with 30

Treap structure:
      [70:0.92]
      /
   [50:0.89]
   /
[20:0.67]
   \
  [30:0.45]

Valid Treap (BST+Heap): YES

=== Test Splay Tree ===
Insert 50, 30, 70, 20, 40
Search 20: SPLAY!
After splay(20), root = 20

Sequence of 100 accesses to same 5 elements:
Average depth accessed: 1.8 (very shallow!)

All tests passed! 🎰

$ cargo test
running 16 tests
test tests::test_treap_insert ... ok
test tests::test_treap_split ... ok
test tests::test_treap_merge ... ok
test tests::test_treap_is_valid ... ok
test tests::test_splay_insert ... ok
test tests::test_splay_search_moves_to_root ... ok
test tests::test_splay_zig ... ok
test tests::test_splay_zig_zig ... ok
test tests::test_splay_zig_zag ... ok
test tests::test_treap_stress ... ok

test result: ok. 16 passed; 0 failed
```

### 3.1 💀 BONUS EXPERT : Implicit Treap (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★★☆ (9/10)

**Récompense :**
XP ×4

**Domaines Bonus :**
`Struct, MD`

#### 3.1.1 Consigne Bonus

**🎰 LE TREAP IMPLICITE — Séquences Magiques**

L'Implicit Treap utilise la **position** comme clé implicite au lieu d'une clé explicite. Cela permet des opérations sur des séquences !

```rust
pub struct ImplicitTreap<V> {
    root: Option<Box<ImplicitNode<V>>>,
}

struct ImplicitNode<V> {
    value: V,
    priority: u64,
    size: usize,  // Size of subtree (used as implicit key)
    left: Option<Box<ImplicitNode<V>>>,
    right: Option<Box<ImplicitNode<V>>>,
}

impl<V> ImplicitTreap<V> {
    /// Insert at position (0-indexed)
    pub fn insert_at(&mut self, pos: usize, value: V);

    /// Delete at position
    pub fn delete_at(&mut self, pos: usize) -> Option<V>;

    /// Get value at position
    pub fn get(&self, pos: usize) -> Option<&V>;

    /// Reverse range [l, r]
    pub fn reverse(&mut self, l: usize, r: usize);

    /// Split at position
    pub fn split_at(self, pos: usize) -> (Self, Self);
}
```

### 3.2 🧠 BONUS GÉNIE : Link-Cut Trees (OPTIONNEL)

**Difficulté Bonus :**
🧠 (11/10)

**Récompense :**
XP ×6

#### 3.2.1 Consigne Bonus

Les **Link-Cut Trees** (Sleator & Tarjan) supportent des opérations dynamiques sur des forêts d'arbres en temps O(log n) amorti.

```rust
pub struct LinkCutTree {
    nodes: Vec<LCNode>,
}

impl LinkCutTree {
    /// Link: make v a child of w
    pub fn link(&mut self, v: usize, w: usize);

    /// Cut: remove edge from v to its parent
    pub fn cut(&mut self, v: usize);

    /// Find root of the tree containing v
    pub fn find_root(&mut self, v: usize) -> usize;

    /// Path query from v to root
    pub fn path_aggregate(&mut self, v: usize) -> i64;
}
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `test_treap_new` | `VegasTreap::new()` | Empty treap | 2 | |
| `test_treap_insert` | `insert(50), insert(30)` | Valid treap | 8 | |
| `test_treap_heap_property` | Multiple inserts | All priorities valid | 10 | ⚠️ |
| `test_treap_bst_property` | Multiple inserts | All keys valid | 10 | ⚠️ |
| `test_treap_split` | `split(40)` | Two valid treaps | 12 | ⚠️ |
| `test_treap_merge` | `merge(t1, t2)` | Valid merged treap | 12 | ⚠️ |
| `test_splay_insert` | `insert(50,30,70)` | Valid BST | 8 | |
| `test_splay_moves_to_root` | `search(30)` | 30 at root | 10 | ⚠️ |
| `test_splay_zig` | Parent is root | Single rotation | 8 | |
| `test_splay_zig_zig` | Same side | Double rotation | 10 | ⚠️ |
| `test_splay_zig_zag` | Opposite sides | LR or RL rotation | 10 | ⚠️ |
| **TOTAL** | | | **100** | |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <assert.h>
#include "vegas_random_trees.h"

void test_treap_properties(void)
{
    srand(time(NULL));
    t_vegas_treap *treap = treap_new();

    // Insert elements
    treap_insert(treap, 50, "A");
    treap_insert(treap, 30, "B");
    treap_insert(treap, 70, "C");
    treap_insert(treap, 20, "D");
    treap_insert(treap, 40, "E");

    // Verify both properties
    assert(treap_is_valid(treap) == 1);

    treap_free(treap);
    printf("test_treap_properties: OK\n");
}

void test_treap_split_merge(void)
{
    srand(42);  // Fixed seed for reproducibility
    t_vegas_treap *treap = treap_new();

    for (int i = 0; i < 10; i++)
        treap_insert(treap, i * 10, "test");

    t_vegas_treap *left, *right;
    treap_split(treap, 45, &left, &right);

    // Left should have 0,10,20,30,40
    // Right should have 50,60,70,80,90
    assert(left->size == 5);
    assert(right->size == 5);

    // Merge back
    t_vegas_treap *merged = treap_merge(left, right);
    assert(merged->size == 10);
    assert(treap_is_valid(merged) == 1);

    treap_free(merged);
    printf("test_treap_split_merge: OK\n");
}

void test_splay_access(void)
{
    t_splay_tree *tree = splay_new();

    splay_insert(tree, 50, "root");
    splay_insert(tree, 30, "left");
    splay_insert(tree, 70, "right");
    splay_insert(tree, 20, "leftleft");

    // Search for 20 - should become root after splay
    char *result = splay_search(tree, 20);
    assert(result != NULL);
    assert(tree->root->key == 20);

    splay_free(tree);
    printf("test_splay_access: OK\n");
}

int main(void)
{
    printf("=== Tests Vegas Random Trees ===\n");
    test_treap_properties();
    test_treap_split_merge();
    test_splay_access();
    printf("\nAll tests passed! 🎰\n");
    return 0;
}
```

### 4.3 Solution de référence

**Rust (Treap) :**
```rust
use rand::Rng;
use std::cmp::Ordering;

pub struct VegasTreap<K: Ord, V> {
    root: Option<Box<TreapNode<K, V>>>,
}

struct TreapNode<K: Ord, V> {
    key: K,
    value: V,
    priority: u64,
    left: Option<Box<TreapNode<K, V>>>,
    right: Option<Box<TreapNode<K, V>>>,
}

impl<K: Ord, V> TreapNode<K, V> {
    fn new(key: K, value: V) -> Self {
        let mut rng = rand::thread_rng();
        TreapNode {
            key,
            value,
            priority: rng.gen(),
            left: None,
            right: None,
        }
    }
}

impl<K: Ord, V> VegasTreap<K, V> {
    pub fn new() -> Self {
        VegasTreap { root: None }
    }

    pub fn insert(&mut self, key: K, value: V) {
        self.root = Self::insert_rec(self.root.take(), key, value);
    }

    fn insert_rec(
        node: Option<Box<TreapNode<K, V>>>,
        key: K,
        value: V,
    ) -> Option<Box<TreapNode<K, V>>> {
        let mut node = match node {
            None => return Some(Box::new(TreapNode::new(key, value))),
            Some(n) => n,
        };

        match key.cmp(&node.key) {
            Ordering::Less => {
                node.left = Self::insert_rec(node.left.take(), key, value);
                // Maintain heap property
                if node.left.as_ref().map_or(false, |l| l.priority > node.priority) {
                    return Some(Self::rotate_right(node));
                }
            }
            Ordering::Greater => {
                node.right = Self::insert_rec(node.right.take(), key, value);
                if node.right.as_ref().map_or(false, |r| r.priority > node.priority) {
                    return Some(Self::rotate_left(node));
                }
            }
            Ordering::Equal => {
                node.value = value;
            }
        }
        Some(node)
    }

    fn rotate_left(mut x: Box<TreapNode<K, V>>) -> Box<TreapNode<K, V>> {
        let mut y = x.right.take().unwrap();
        x.right = y.left.take();
        y.left = Some(x);
        y
    }

    fn rotate_right(mut y: Box<TreapNode<K, V>>) -> Box<TreapNode<K, V>> {
        let mut x = y.left.take().unwrap();
        y.left = x.right.take();
        x.right = Some(y);
        x
    }

    pub fn split(self, key: &K) -> (Self, Self)
    where
        K: Clone,
    {
        fn split_rec<K: Ord + Clone, V>(
            node: Option<Box<TreapNode<K, V>>>,
            key: &K,
        ) -> (Option<Box<TreapNode<K, V>>>, Option<Box<TreapNode<K, V>>>) {
            match node {
                None => (None, None),
                Some(mut n) => {
                    if n.key < *key {
                        let (left, right) = split_rec(n.right.take(), key);
                        n.right = left;
                        (Some(n), right)
                    } else {
                        let (left, right) = split_rec(n.left.take(), key);
                        n.left = right;
                        (left, Some(n))
                    }
                }
            }
        }

        let (left, right) = split_rec(self.root, key);
        (VegasTreap { root: left }, VegasTreap { root: right })
    }

    pub fn merge(left: Self, right: Self) -> Self {
        fn merge_rec<K: Ord, V>(
            left: Option<Box<TreapNode<K, V>>>,
            right: Option<Box<TreapNode<K, V>>>,
        ) -> Option<Box<TreapNode<K, V>>> {
            match (left, right) {
                (None, r) => r,
                (l, None) => l,
                (Some(mut l), Some(mut r)) => {
                    if l.priority > r.priority {
                        l.right = merge_rec(l.right.take(), Some(r));
                        Some(l)
                    } else {
                        r.left = merge_rec(Some(l), r.left.take());
                        Some(r)
                    }
                }
            }
        }

        VegasTreap {
            root: merge_rec(left.root, right.root),
        }
    }

    pub fn is_valid_treap(&self) -> bool {
        fn check<K: Ord, V>(
            node: &Option<Box<TreapNode<K, V>>>,
            min: Option<&K>,
            max: Option<&K>,
        ) -> bool {
            match node {
                None => true,
                Some(n) => {
                    // BST property
                    if min.map_or(false, |m| n.key <= *m) {
                        return false;
                    }
                    if max.map_or(false, |m| n.key >= *m) {
                        return false;
                    }
                    // Heap property
                    if n.left.as_ref().map_or(false, |l| l.priority > n.priority) {
                        return false;
                    }
                    if n.right.as_ref().map_or(false, |r| r.priority > n.priority) {
                        return false;
                    }
                    check(&n.left, min, Some(&n.key))
                        && check(&n.right, Some(&n.key), max)
                }
            }
        }
        check(&self.root, None, None)
    }
}
```

### 4.9 spec.json

```json
{
  "name": "vegas_random_trees",
  "language": "rust",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isolé - Treap + Splay",
  "tags": ["treap", "splay", "randomized", "trees", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "VegasTreap",
    "prototype": "pub struct VegasTreap<K: Ord, V>",
    "return_type": "struct",
    "parameters": []
  },

  "driver": {
    "reference": "/* See section 4.3 */",

    "edge_cases": [
      {
        "name": "treap_heap_property",
        "args": ["50", "30", "70", "20"],
        "expected": "All priorities maintain heap",
        "is_trap": true,
        "trap_explanation": "Les priorités doivent respecter max-heap"
      },
      {
        "name": "treap_split",
        "args": ["split(40)"],
        "expected": "Two valid treaps",
        "is_trap": true,
        "trap_explanation": "Split doit préserver BST et heap"
      },
      {
        "name": "splay_to_root",
        "args": ["search(20)"],
        "expected": "20 becomes root",
        "is_trap": true,
        "trap_explanation": "Splay doit remonter jusqu'à la racine"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": { "min": -10000, "max": 10000 }
        }
      ]
    }
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```rust
/* Mutant A (Priority) : Pas de rotation pour maintenir heap */
fn insert_rec_mutant_a(
    node: Option<Box<TreapNode<K, V>>>,
    key: K,
    value: V,
) -> Option<Box<TreapNode<K, V>>> {
    let mut node = match node { ... };
    match key.cmp(&node.key) {
        Ordering::Less => {
            node.left = Self::insert_rec(node.left.take(), key, value);
            // BUG: Pas de rotation même si priority enfant > parent
        }
        ...
    }
    Some(node)
}
// Pourquoi c'est faux : La propriété heap n'est pas maintenue
// Ce qui était pensé : "L'insertion BST suffit"

/* Mutant B (Splay) : Zig au lieu de Zig-Zig */
fn splay_mutant_b(&mut self, key: &K) {
    while let Some(ref mut root) = self.root {
        if root.key == *key { break; }
        // BUG: Toujours une seule rotation
        if *key < root.key {
            self.root = Some(Self::zig_right(root));  // Devrait être zig-zig parfois
        } else {
            self.root = Some(Self::zig_left(root));
        }
    }
}
// Pourquoi c'est faux : Zig-zig est crucial pour l'analyse amortie O(log n)
// Ce qui était pensé : "Une rotation par étape suffit"

/* Mutant C (Split) : Split incorrect */
fn split_mutant_c(self, key: &K) -> (Self, Self) {
    fn split_rec<K: Ord, V>(node: Option<Box<TreapNode<K, V>>>, key: &K)
        -> (Option<Box<TreapNode<K, V>>>, Option<Box<TreapNode<K, V>>>)
    {
        match node {
            None => (None, None),
            Some(mut n) => {
                // BUG: Compare avec <= au lieu de <
                if n.key <= *key {  // Devrait être <
                    let (left, right) = split_rec(n.right.take(), key);
                    n.right = left;
                    (Some(n), right)
                } else { ... }
            }
        }
    }
    ...
}
// Pourquoi c'est faux : Éléments égaux au pivot dans le mauvais sous-arbre
// Ce qui était pensé : "<= et < c'est pareil"

/* Mutant D (Merge) : Merge qui viole BST */
fn merge_mutant_d(left: Self, right: Self) -> Self {
    fn merge_rec<K: Ord, V>(left: Option<Box<TreapNode<K, V>>>, right: Option<Box<TreapNode<K, V>>>)
        -> Option<Box<TreapNode<K, V>>>
    {
        match (left, right) {
            (None, r) => r,
            (l, None) => l,
            (Some(mut l), Some(mut r)) => {
                // BUG: Ignore les priorités
                l.right = merge_rec(l.right.take(), Some(r));
                Some(l)
            }
        }
    }
    ...
}
// Pourquoi c'est faux : La propriété heap est violée
// Ce qui était pensé : "Merge = concaténation simple"

/* Mutant E (Splay) : Splay qui s'arrête avant la racine */
fn splay_mutant_e(&mut self, key: &K) {
    let mut found = false;
    // BUG: Boucle qui s'arrête trop tôt
    for _ in 0..3 {  // Seulement 3 itérations max
        if let Some(ref root) = self.root {
            if root.key == *key { found = true; break; }
        }
        // ... rotations
    }
    // Le nœud peut ne pas être à la racine après seulement 3 rotations
}
// Pourquoi c'est faux : Splay DOIT remonter jusqu'à la racine
// Ce qui était pensé : "Quelques rotations suffisent"
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Équilibre probabiliste** : Le hasard peut garantir l'équilibre en espérance
2. **Analyse amortie** : Coût moyen sur une séquence d'opérations
3. **Split/Merge** : Opérations puissantes des treaps
4. **Self-adjusting** : Structures qui s'adaptent aux patterns d'accès
5. **Trade-offs** : Déterministe vs probabiliste, pire cas vs amorti

### 5.3 Visualisation ASCII

```
TREAP : Propriété BST (clés) + Heap (priorités)

Clés:          Priorités:
    50             0.9
   /  \           /   \
 30   70        0.7   0.5
 /               /
20             0.3

BST: 20 < 30 < 50 < 70 ✓
Heap: 0.9 > 0.7 > 0.3 ✓
      0.9 > 0.5 ✓

SPLAY : Après splay(20)

Avant:          Après:
    50            20
   /  \             \
 30   70   →        30
 /                    \
20                    50
                        \
                        70

20 est maintenant à la racine !
```

**Les 3 cas Splay :**

```
ZIG (parent = root):
    P           X
   /    →        \
  X               P

ZIG-ZIG (même côté):
    G           X
   /             \
  P       →       P
 /                 \
X                   G

ZIG-ZAG (côtés opposés):
    G             X
   /             / \
  P      →      P   G
   \
    X
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🎰 MEME : "The House Always Wins" — Treap

*"In Vegas, randomness favors the house. In Treaps, randomness favors balance."*

```
Treap = Tree + Heap = T(ree H)eap
        ou
Treap = Tree + rAP (Random Access Priority)
```

La "maison" (l'algorithme) gagne toujours car :
- Priorités aléatoires → hauteur espérée O(log n)
- Pas besoin de rotations complexes d'équilibrage
- Split/Merge en O(log n) gratuitement !

#### 🃏 MEME : "Hot Hand Fallacy" — Splay Tree

En basket, on croit que le joueur "chaud" va continuer à marquer. C'est souvent faux.

Mais en Splay Tree, c'est VRAI ! Les éléments récemment accédés sont en haut, donc :
- Accès répétés = O(1) après le premier
- La "hot hand" est réelle dans les Splay Trees !

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.3.3-a — vegas_random_trees |
| **Concepts** | Treap + Splay Tree |
| **Difficulté** | ★★★★★★★☆☆☆ (7/10) |
| **Temps estimé** | 90 min |
| **XP Base** | 250 |
| **Bonus Implicit** | 💀 Expert (×4) |
| **Bonus Link-Cut** | 🧠 Génie (×6) |
| **Langage** | Rust 2024 + C (c17) |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.3.3-a-vegas-random-trees",
    "generated_at": "2025-01-11 15:30:00",

    "metadata": {
      "exercise_id": "1.3.3-a",
      "exercise_name": "vegas_random_trees",
      "module": "1.3.3",
      "module_name": "Randomized Trees",
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "meme_reference": "Las Vegas - The House Always Wins"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "In Vegas, luck is random. In Treaps, random is luck."*
