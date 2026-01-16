<thinking>
## Analyse du Concept
- Concept : Red-Black Trees - Arbres auto-équilibrés avec coloration
- Phase demandée : 1 (intermédiaire avancé)
- Adapté ? OUI - Les RB Trees sont fondamentaux, utilisés partout (std::map, TreeMap, etc.)

## Combo Base + Bonus
- Exercice de base : Implémenter RB Tree avec les 5 propriétés, insert avec fix-up
- Bonus : Delete avec fix-up (beaucoup plus complexe) + Left-Leaning RB Tree (variante simplifiée)
- Palier bonus : 💀 Expert pour delete, 🧠 Génie pour LLRB
- Progression logique ? OUI - Insert = base, Delete = avancé

## Prérequis & Difficulté
- Prérequis réels : BST, AVL rotations, concept de coloration
- Difficulté estimée : 7/10 (base), 9/10 (delete bonus)
- Cohérent avec phase 1 ? Limite haute mais OK

## Aspect Fun/Culture
- Contexte choisi : Jeu d'échecs / La Dame de Pique - Rouge vs Noir
- MEME mnémotechnique : "Red Wedding" (Game of Thrones) - quand deux rouges se rencontrent, c'est le chaos
- Pourquoi c'est fun : L'alternance rouge/noir comme aux cartes, les règles strictes comme aux échecs

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Property) : Autoriser deux rouges consécutifs (viole propriété 4)
2. Mutant B (Property) : Root rouge (viole propriété 2)
3. Mutant C (Logic) : Mauvais cas dans fix-up (oncle rouge vs noir)
4. Mutant D (Logic) : Rotation sans recoloration
5. Mutant E (Return) : Oubli de propager le fix-up vers le haut

## Verdict
VALIDE - Le thème échecs/cartes est parfait pour rouge/noir
</thinking>

---

# Exercice 1.3.2-a : chess_rb_tree

**Module :**
1.3.2 — Red-Black Trees

**Concept :**
a — Arbres Rouge-Noir avec les 5 propriétés

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
- AVL Rotations (exercice 1.3.1)
- Concept de coloration de nœuds

**Domaines :**
Struct, Mem, MD

**Durée estimée :**
90 min

**XP Base :**
250

**Complexité :**
T[3] O(log n) garantie × S[2] O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `chess_rb_tree.c`, `chess_rb_tree.h`

**Fonctions autorisées :**
- C : `malloc`, `free`, `NULL`
- Rust : `Box::new`, `Option`, `std::cmp::Ordering`

**Fonctions interdites :**
- C : Bibliothèques d'arbres
- Rust : `BTreeMap`, `BTreeSet`, collections RB

### 1.2 Consigne

**♟️ L'ÉCHIQUIER BINAIRE — Le Jeu Rouge et Noir**

*"Chaque pièce a sa place. Rouge ne peut suivre rouge. Le roi (racine) est toujours noir. Et de chaque case à la frontière, le chemin traverse le même nombre de cases noires..."*

Dans le monde des échecs algorithmiques, l'arbre Red-Black est le grand maître de l'équilibre ! Comme un échiquier où chaque case alterne (presque), un RB-Tree maintient l'harmonie avec **5 règles sacrées**.

**Les 5 Propriétés du RB-Tree (Les Règles du Jeu) :**

| # | Propriété | Analogie Échecs |
|---|-----------|-----------------|
| 1 | Chaque nœud est ROUGE ou NOIR | Chaque case a une couleur |
| 2 | La racine est NOIRE | Le roi (centre) est sur case noire |
| 3 | Les feuilles (NIL) sont NOIRES | Les bords de l'échiquier |
| 4 | Un nœud ROUGE a des enfants NOIRS | Rouge ne peut suivre rouge |
| 5 | Tout chemin racine→feuille a le même nombre de nœuds NOIRS | Équilibre parfait |

**Ta mission :**

Créer une structure `ChessTree<K, V>` qui implémente un **Red-Black Tree** :

1. **Insertion avec recoloration** : Nouveau nœud = ROUGE, puis fix-up
2. **Les rotations** : Comme AVL mais avec changement de couleur
3. **Fix-up après insertion** : Gérer les 3 cas (oncle rouge, oncle noir zigzag, oncle noir ligne)
4. **Validation des propriétés** : Vérifier que les 5 règles sont respectées

**Les 3 cas d'insertion fix-up :**

```
CAS 1 : Oncle ROUGE 🔴
→ Recolorer parent et oncle en NOIR, grand-parent en ROUGE
→ Propager le fix-up vers le haut

CAS 2 : Oncle NOIR, nœud est enfant "intérieur" (zigzag)
→ Rotation pour transformer en CAS 3

CAS 3 : Oncle NOIR, nœud est enfant "extérieur" (ligne)
→ Rotation + recoloration
```

**Entrée :**
- `key` : Clé comparable
- `value` : Valeur associée

**Sortie :**
- `insert` : Arbre respectant les 5 propriétés après insertion
- `is_valid_rb` : `true` si toutes les propriétés sont respectées

**Contraintes :**
- Les 5 propriétés doivent TOUJOURS être maintenues
- Nouveau nœud inséré est toujours ROUGE initialement
- La racine est TOUJOURS recolorée en NOIR à la fin

**Exemples :**

| Opération | Avant | Après | Fix-up effectué |
|-----------|-------|-------|-----------------|
| `insert(10)` | `∅` | `[10:B]` | Root devient NOIR |
| `insert(5)` | `[10:B]` | `[10:B]←[5:R]` | Aucun (parent noir) |
| `insert(3)` | `[10:B]←[5:R]` | Rotation + recolor | Cas 3 (ligne gauche) |

### 1.3 Prototype

**Rust :**
```rust
#[derive(Clone, Copy, PartialEq, Debug)]
pub enum Color {
    Red,
    Black,
}

pub struct ChessTree<K: Ord, V> {
    root: Option<Box<ChessNode<K, V>>>,
}

struct ChessNode<K: Ord, V> {
    key: K,
    value: V,
    color: Color,
    left: Option<Box<ChessNode<K, V>>>,
    right: Option<Box<ChessNode<K, V>>>,
}

impl<K: Ord, V> ChessTree<K, V> {
    pub fn new() -> Self;

    // Opérations principales
    pub fn insert(&mut self, key: K, value: V);
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn contains(&self, key: &K) -> bool;

    // Rotations (héritées d'AVL)
    fn rotate_left(node: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>>;
    fn rotate_right(node: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>>;

    // Fix-up après insertion
    fn fix_insert(node: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>>;

    // Validation des 5 propriétés
    pub fn is_valid_rb(&self) -> bool;
    fn check_property_2(&self) -> bool;  // Root is black
    fn check_property_4(&self) -> bool;  // No red-red
    fn check_property_5(&self) -> bool;  // Black height uniform

    // Utilitaires
    fn black_height(&self) -> Option<usize>;
    pub fn inorder(&self) -> Vec<(&K, &V)>;
}
```

**C :**
```c
typedef enum e_color {
    RED,
    BLACK
} t_color;

typedef struct s_chess_node {
    int                     key;
    char                    *value;
    t_color                 color;
    struct s_chess_node     *left;
    struct s_chess_node     *right;
    struct s_chess_node     *parent;  // Utile pour fix-up
} t_chess_node;

typedef struct s_chess_tree {
    t_chess_node    *root;
    t_chess_node    *nil;  // Sentinelle NIL (optionnel mais recommandé)
    size_t          size;
} t_chess_tree;

// Création/Destruction
t_chess_tree    *chess_new(void);
void            chess_free(t_chess_tree *tree);

// Opérations principales
void            chess_insert(t_chess_tree *tree, int key, char *value);
char            *chess_search(t_chess_tree *tree, int key);

// Rotations
void            chess_rotate_left(t_chess_tree *tree, t_chess_node *x);
void            chess_rotate_right(t_chess_tree *tree, t_chess_node *y);

// Fix-up
void            chess_fix_insert(t_chess_tree *tree, t_chess_node *z);

// Validation
int             chess_is_valid_rb(t_chess_tree *tree);
int             chess_check_no_red_red(t_chess_node *node);
int             chess_black_height(t_chess_node *node);
```

### 1.2.2 Énoncé Académique

Un **arbre Rouge-Noir** est un arbre binaire de recherche auto-équilibré où chaque nœud possède une couleur (rouge ou noir) et les propriétés suivantes sont maintenues :

1. **Coloration** : Chaque nœud est soit rouge, soit noir
2. **Racine** : La racine est toujours noire
3. **Feuilles** : Les nœuds NIL (feuilles externes) sont noirs
4. **Rouge** : Un nœud rouge ne peut pas avoir d'enfant rouge
5. **Black-height** : Tout chemin d'un nœud vers ses feuilles descendantes contient le même nombre de nœuds noirs

Ces propriétés garantissent que la hauteur est au plus 2×log₂(n+1).

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Histoire du RB-Tree

L'arbre Rouge-Noir a été inventé par **Rudolf Bayer** en 1972 sous le nom "symmetric binary B-trees". Le nom "Red-Black" a été donné par **Leonidas Guibas** et **Robert Sedgewick** en 1978.

**Fun fact :** Sedgewick a raconté qu'ils ont choisi rouge et noir parce que c'étaient les seules couleurs qui rendaient bien sur les imprimantes laser Xerox de l'époque !

### 2.2 Où sont utilisés les RB-Trees ?

| Langage/Système | Utilisation |
|-----------------|-------------|
| **C++ STL** | `std::map`, `std::set` |
| **Java** | `TreeMap`, `TreeSet` |
| **Linux Kernel** | CFS scheduler, memory management |
| **Python** | Certaines implémentations de dict |

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation | Cas concret |
|--------|-------------|-------------|
| **Kernel Developer** | Process scheduling | Linux CFS (Completely Fair Scheduler) |
| **Database Engineer** | Index structures | PostgreSQL, MySQL |
| **Game Developer** | Spatial indexing | Collision detection |
| **System Programmer** | Memory allocators | jemalloc, tcmalloc |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
chess_rb_tree.c  chess_rb_tree.h  main.c  Cargo.toml  src/

$ gcc -Wall -Wextra -Werror -std=c17 chess_rb_tree.c main.c -o test_c

$ ./test_c
=== Test Red-Black Chess Tree ===
Insert 10: [10:B] (root always black)
Insert 5:  [10:B]←[5:R] (parent black, OK)
Insert 15: [10:B]←[5:R]→[15:R] (parent black, OK)
Insert 3:  FIX-UP CAS 1! (uncle red)
           [10:B]←[5:B←3:R]→[15:B]

Insert 7:  [10:B]←[5:B←3:R→7:R]→[15:B]
Insert 1:  FIX-UP CAS 3! (uncle black, line)
           Rotation + recolor

Properties check:
  - Root is BLACK: YES
  - No red-red: YES
  - Black height uniform: YES (bh=2)

All tests passed! ♟️

$ cargo test
running 12 tests
test tests::test_insert ... ok
test tests::test_property_2_root_black ... ok
test tests::test_property_4_no_red_red ... ok
test tests::test_property_5_black_height ... ok
test tests::test_fixup_case_1 ... ok
test tests::test_fixup_case_2 ... ok
test tests::test_fixup_case_3 ... ok
test tests::test_stress_100 ... ok

test result: ok. 12 passed; 0 failed
```

### 3.1 💀 BONUS EXPERT : Delete avec Fix-up (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★★☆ (9/10)

**Récompense :**
XP ×4

**Domaines Bonus :**
`MD, Struct`

#### 3.1.1 Consigne Bonus

**♟️ LA PRISE EN PASSANT — Suppression dans le RB-Tree**

*"Retirer une pièce de l'échiquier est bien plus complexe que d'en ajouter une..."*

La suppression dans un RB-Tree est notoirement complexe avec **6 cas** à gérer. Implémenter `delete()` avec le fix-up complet.

```rust
impl<K: Ord, V> ChessTree<K, V> {
    /// Delete a key, maintaining all 5 RB properties
    pub fn delete(&mut self, key: &K) -> Option<V>;

    /// Fix-up after deletion (6 cases!)
    fn fix_delete(node: &mut Option<Box<ChessNode<K, V>>>);
}
```

**Les 6 cas de delete fix-up :**
1. Sibling est rouge
2. Sibling noir, deux enfants noirs
3. Sibling noir, enfant gauche rouge
4. Sibling noir, enfant droit rouge
5-6. Cas miroirs

### 3.2 🧠 BONUS GÉNIE : Left-Leaning Red-Black Tree (OPTIONNEL)

**Difficulté Bonus :**
🧠 (11/10)

**Récompense :**
XP ×6

#### 3.2.1 Consigne Bonus

Implémenter la variante **LLRB** de Sedgewick qui simplifie l'implémentation en forçant tous les liens rouges à pencher à gauche.

```rust
impl<K: Ord, V> ChessTree<K, V> {
    /// LLRB insert - simpler with constraint that red links lean left
    pub fn llrb_insert(&mut self, key: K, value: V);

    fn is_red(node: &Option<Box<ChessNode<K, V>>>) -> bool;
    fn flip_colors(node: &mut Box<ChessNode<K, V>>);
}
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `test_new` | `ChessTree::new()` | Valid RB | 2 | |
| `test_insert_root` | `insert(10)` | Root BLACK | 5 | ⚠️ |
| `test_no_fix_needed` | `insert(10,5)` | Parent black | 5 | |
| `test_case_1_uncle_red` | `insert(10,5,15,3)` | Recoloration | 12 | ⚠️ |
| `test_case_2_zigzag` | `insert(10,5,7)` | Double rotation | 12 | ⚠️ |
| `test_case_3_line` | `insert(10,5,3)` | Single rotation | 12 | ⚠️ |
| `test_property_2` | Any tree | Root BLACK | 8 | |
| `test_property_4` | Any tree | No red-red | 10 | |
| `test_property_5` | Any tree | Uniform bh | 12 | |
| `test_stress_50` | 50 insertions | Valid RB | 10 | |
| `test_black_height` | Complex tree | Correct bh | 8 | |
| `test_search` | After inserts | Correct values | 4 | |
| **TOTAL** | | | **100** | |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <assert.h>
#include "chess_rb_tree.h"

void test_case_1_uncle_red(void)
{
    t_chess_tree *tree = chess_new();

    // Create scenario where uncle is red
    chess_insert(tree, 10, "King");    // Root black
    chess_insert(tree, 5, "Queen");    // Red
    chess_insert(tree, 15, "Rook");    // Red
    chess_insert(tree, 3, "Bishop");   // Red -> triggers case 1

    // After case 1: 5 and 15 should be black, 3 red
    assert(tree->root->color == BLACK);
    assert(tree->root->left->color == BLACK);
    assert(tree->root->right->color == BLACK);
    assert(tree->root->left->left->color == RED);

    assert(chess_is_valid_rb(tree) == 1);

    chess_free(tree);
    printf("test_case_1_uncle_red: OK\n");
}

void test_case_3_line(void)
{
    t_chess_tree *tree = chess_new();

    // Insert in line: 30, 20, 10 -> triggers case 3
    chess_insert(tree, 30, "A");
    chess_insert(tree, 20, "B");
    chess_insert(tree, 10, "C");

    // After rotation, 20 should be root
    assert(tree->root->key == 20);
    assert(tree->root->color == BLACK);
    assert(chess_is_valid_rb(tree) == 1);

    chess_free(tree);
    printf("test_case_3_line: OK\n");
}

void test_all_properties(void)
{
    t_chess_tree *tree = chess_new();

    // Insert many values
    int values[] = {50, 25, 75, 12, 37, 62, 87, 6, 18, 31, 43};
    for (int i = 0; i < 11; i++)
    {
        chess_insert(tree, values[i], "test");
        assert(chess_is_valid_rb(tree) == 1);
    }

    // Verify black height is uniform
    int bh = chess_black_height(tree->root);
    assert(bh >= 0);

    chess_free(tree);
    printf("test_all_properties: OK (bh=%d)\n", bh);
}

int main(void)
{
    printf("=== Tests Red-Black Chess Tree ===\n");
    test_case_1_uncle_red();
    test_case_3_line();
    test_all_properties();
    printf("\nAll tests passed! ♟️\n");
    return 0;
}
```

### 4.3 Solution de référence

**Rust :**
```rust
use std::cmp::Ordering;

#[derive(Clone, Copy, PartialEq, Debug)]
pub enum Color {
    Red,
    Black,
}

pub struct ChessTree<K: Ord, V> {
    root: Option<Box<ChessNode<K, V>>>,
}

struct ChessNode<K: Ord, V> {
    key: K,
    value: V,
    color: Color,
    left: Option<Box<ChessNode<K, V>>>,
    right: Option<Box<ChessNode<K, V>>>,
}

impl<K: Ord, V> ChessNode<K, V> {
    fn new(key: K, value: V) -> Self {
        ChessNode {
            key,
            value,
            color: Color::Red,  // New nodes are always red
            left: None,
            right: None,
        }
    }

    fn is_red(node: &Option<Box<ChessNode<K, V>>>) -> bool {
        node.as_ref().map_or(false, |n| n.color == Color::Red)
    }
}

impl<K: Ord, V> ChessTree<K, V> {
    pub fn new() -> Self {
        ChessTree { root: None }
    }

    pub fn insert(&mut self, key: K, value: V) {
        self.root = Self::insert_rec(self.root.take(), key, value);
        // Property 2: Root is always black
        if let Some(ref mut root) = self.root {
            root.color = Color::Black;
        }
    }

    fn insert_rec(
        node: Option<Box<ChessNode<K, V>>>,
        key: K,
        value: V,
    ) -> Option<Box<ChessNode<K, V>>> {
        let mut node = match node {
            None => return Some(Box::new(ChessNode::new(key, value))),
            Some(n) => n,
        };

        match key.cmp(&node.key) {
            Ordering::Less => {
                node.left = Self::insert_rec(node.left.take(), key, value);
            }
            Ordering::Greater => {
                node.right = Self::insert_rec(node.right.take(), key, value);
            }
            Ordering::Equal => {
                node.value = value;
                return Some(node);
            }
        }

        // Fix-up: balance the tree
        Some(Self::fix_insert(node))
    }

    fn fix_insert(mut node: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>> {
        // Case: right child red, left child black -> rotate left
        if ChessNode::is_red(&node.right) && !ChessNode::is_red(&node.left) {
            node = Self::rotate_left(node);
        }

        // Case: left child red, left-left grandchild red -> rotate right
        if ChessNode::is_red(&node.left) {
            if node.left.as_ref().map_or(false, |l| ChessNode::is_red(&l.left)) {
                node = Self::rotate_right(node);
            }
        }

        // Case: both children red -> flip colors
        if ChessNode::is_red(&node.left) && ChessNode::is_red(&node.right) {
            Self::flip_colors(&mut node);
        }

        node
    }

    fn rotate_left(mut x: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>> {
        let mut y = x.right.take().unwrap();
        x.right = y.left.take();
        y.color = x.color;
        x.color = Color::Red;
        y.left = Some(x);
        y
    }

    fn rotate_right(mut y: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>> {
        let mut x = y.left.take().unwrap();
        y.left = x.right.take();
        x.color = y.color;
        y.color = Color::Red;
        x.right = Some(y);
        x
    }

    fn flip_colors(node: &mut Box<ChessNode<K, V>>) {
        node.color = match node.color {
            Color::Red => Color::Black,
            Color::Black => Color::Red,
        };
        if let Some(ref mut left) = node.left {
            left.color = match left.color {
                Color::Red => Color::Black,
                Color::Black => Color::Red,
            };
        }
        if let Some(ref mut right) = node.right {
            right.color = match right.color {
                Color::Red => Color::Black,
                Color::Black => Color::Red,
            };
        }
    }

    pub fn get(&self, key: &K) -> Option<&V> {
        fn search<K: Ord, V>(node: &Option<Box<ChessNode<K, V>>>, key: &K) -> Option<&V> {
            let n = node.as_ref()?;
            match key.cmp(&n.key) {
                Ordering::Equal => Some(&n.value),
                Ordering::Less => search(&n.left, key),
                Ordering::Greater => search(&n.right, key),
            }
        }
        search(&self.root, key)
    }

    pub fn is_valid_rb(&self) -> bool {
        self.check_property_2() && self.check_property_4() && self.check_property_5()
    }

    fn check_property_2(&self) -> bool {
        // Root must be black
        self.root.as_ref().map_or(true, |r| r.color == Color::Black)
    }

    fn check_property_4(&self) -> bool {
        // No red node has a red child
        fn check<K: Ord, V>(node: &Option<Box<ChessNode<K, V>>>) -> bool {
            match node {
                None => true,
                Some(n) => {
                    if n.color == Color::Red {
                        if ChessNode::is_red(&n.left) || ChessNode::is_red(&n.right) {
                            return false;
                        }
                    }
                    check(&n.left) && check(&n.right)
                }
            }
        }
        check(&self.root)
    }

    fn check_property_5(&self) -> bool {
        // All paths have same black height
        self.black_height().is_some()
    }

    fn black_height(&self) -> Option<usize> {
        fn bh<K: Ord, V>(node: &Option<Box<ChessNode<K, V>>>) -> Option<usize> {
            match node {
                None => Some(1),  // NIL nodes count as 1 black
                Some(n) => {
                    let left_bh = bh(&n.left)?;
                    let right_bh = bh(&n.right)?;
                    if left_bh != right_bh {
                        return None;
                    }
                    let add = if n.color == Color::Black { 1 } else { 0 };
                    Some(left_bh + add)
                }
            }
        }
        bh(&self.root)
    }
}

impl<K: Ord, V> Default for ChessTree<K, V> {
    fn default() -> Self {
        Self::new()
    }
}
```

### 4.9 spec.json

```json
{
  "name": "chess_rb_tree",
  "language": "rust",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isolé - Red-Black Tree",
  "tags": ["rb-tree", "trees", "balance", "coloring", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "ChessTree",
    "prototype": "pub struct ChessTree<K: Ord, V>",
    "return_type": "struct",
    "parameters": []
  },

  "driver": {
    "reference": "/* See section 4.3 */",

    "edge_cases": [
      {
        "name": "root_black",
        "args": ["10"],
        "expected": "root.color == Black",
        "is_trap": true,
        "trap_explanation": "La racine doit toujours être noire (Propriété 2)"
      },
      {
        "name": "case_1_uncle_red",
        "args": ["10", "5", "15", "3"],
        "expected": "Recoloration without rotation",
        "is_trap": true,
        "trap_explanation": "Cas 1: oncle rouge = recoloration seulement"
      },
      {
        "name": "case_3_line",
        "args": ["30", "20", "10"],
        "expected": "Single rotation + recolor",
        "is_trap": true,
        "trap_explanation": "Cas 3: ligne = rotation simple"
      },
      {
        "name": "black_height_uniform",
        "args": ["50", "25", "75", "12", "37"],
        "expected": "Same black height on all paths",
        "is_trap": true,
        "trap_explanation": "Propriété 5: black height uniforme"
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
  },

  "norm": {
    "allowed_functions": ["Box::new", "Option"],
    "forbidden_functions": ["BTreeMap", "BTreeSet"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```rust
/* Mutant A (Property) : Autorise deux rouges consécutifs */
fn fix_insert_mutant_a(node: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>> {
    // BUG: Ne vérifie pas si left-left est rouge
    if ChessNode::is_red(&node.right) && !ChessNode::is_red(&node.left) {
        node = Self::rotate_left(node);
    }
    // Manque la vérification du cas left-left red
    node
}
// Pourquoi c'est faux : Viole propriété 4 (pas de rouge-rouge)
// Ce qui était pensé : "Un seul cas suffit"

/* Mutant B (Property) : Root reste rouge */
fn insert_mutant_b(&mut self, key: K, value: V) {
    self.root = Self::insert_rec(self.root.take(), key, value);
    // BUG: Oubli de forcer root en noir
    // if let Some(ref mut root) = self.root {
    //     root.color = Color::Black;
    // }
}
// Pourquoi c'est faux : Viole propriété 2 (racine noire)
// Ce qui était pensé : "Le fix-up s'en occupe"

/* Mutant C (Logic) : Mauvais cas dans fix-up */
fn fix_insert_mutant_c(mut node: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>> {
    // BUG: Flip colors quand un seul enfant est rouge
    if ChessNode::is_red(&node.left) || ChessNode::is_red(&node.right) {
        Self::flip_colors(&mut node);  // Devrait être AND, pas OR
    }
    node
}
// Pourquoi c'est faux : Flip colors ne s'applique que quand les DEUX sont rouges
// Ce qui était pensé : "OR et AND c'est pareil"

/* Mutant D (Logic) : Rotation sans recoloration */
fn rotate_left_mutant_d(mut x: Box<ChessNode<K, V>>) -> Box<ChessNode<K, V>> {
    let mut y = x.right.take().unwrap();
    x.right = y.left.take();
    // BUG: Pas de recoloration
    // y.color = x.color;
    // x.color = Color::Red;
    y.left = Some(x);
    y
}
// Pourquoi c'est faux : Les couleurs doivent être échangées lors de la rotation
// Ce qui était pensé : "La rotation change juste la structure"

/* Mutant E (Return) : Oubli de propager le fix-up */
fn insert_rec_mutant_e(
    node: Option<Box<ChessNode<K, V>>>,
    key: K,
    value: V,
) -> Option<Box<ChessNode<K, V>>> {
    let mut node = match node {
        None => return Some(Box::new(ChessNode::new(key, value))),
        Some(n) => n,
    };

    match key.cmp(&node.key) {
        Ordering::Less => {
            node.left = Self::insert_rec(node.left.take(), key, value);
        }
        Ordering::Greater => {
            node.right = Self::insert_rec(node.right.take(), key, value);
        }
        Ordering::Equal => { node.value = value; }
    }

    // BUG: Retourne node sans fix_insert
    Some(node)
    // Devrait être: Some(Self::fix_insert(node))
}
// Pourquoi c'est faux : Le fix-up n'est jamais appliqué
// Ce qui était pensé : "L'insertion suffit"
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

1. **Invariants de couleur** : Maintenir des propriétés non-structurelles
2. **Les 5 propriétés RB** : Comprendre pourquoi chacune est nécessaire
3. **Fix-up en 3 cas** : Analyser et corriger les violations
4. **Black-height** : Métrique d'équilibre plus subtile que la hauteur
5. **Différence AVL vs RB** : Quand utiliser l'un ou l'autre

### 5.2 LDA — Traduction Littérale en Français (MAJUSCULES)

```
ÉNUMÉRATION Color CONTENANT Red ET Black
FIN ÉNUMÉRATION

FONCTION insert QUI PREND key ET value
DÉBUT FONCTION
    APPELER insert_rec SUR root AVEC key ET value
    SI root N'EST PAS NUL ALORS
        AFFECTER Black À root.color
    FIN SI
FIN FONCTION

FONCTION insert_rec QUI PREND node, key, value ET RETOURNE UN NŒUD
DÉBUT FONCTION
    SI node EST NUL ALORS
        RETOURNER UN NOUVEAU NŒUD AVEC key, value, color=Red
    FIN SI

    SI key EST INFÉRIEUR À node.key ALORS
        AFFECTER insert_rec(node.left, key, value) À node.left
    SINON SI key EST SUPÉRIEUR À node.key ALORS
        AFFECTER insert_rec(node.right, key, value) À node.right
    SINON
        AFFECTER value À node.value
        RETOURNER node
    FIN SI

    RETOURNER fix_insert(node)
FIN FONCTION

FONCTION fix_insert QUI PREND node ET RETOURNE UN NŒUD ÉQUILIBRÉ
DÉBUT FONCTION
    SI right EST ROUGE ET left N'EST PAS ROUGE ALORS
        APPLIQUER ROTATION GAUCHE SUR node
    FIN SI

    SI left EST ROUGE ET left.left EST ROUGE ALORS
        APPLIQUER ROTATION DROITE SUR node
    FIN SI

    SI left EST ROUGE ET right EST ROUGE ALORS
        INVERSER LES COULEURS DE node ET SES ENFANTS
    FIN SI

    RETOURNER node
FIN FONCTION
```

### 5.3 Visualisation ASCII

```
LES 5 PROPRIÉTÉS ILLUSTRÉES :

Propriété 1 & 2 : Chaque nœud est R ou B, racine est B
        [10:B] ← Racine NOIRE

Propriété 3 : NIL (feuilles) sont noires
        [10:B]
       /      \
    [5:R]   [NIL:B]
   /    \
[NIL:B][NIL:B]

Propriété 4 : Rouge n'a pas d'enfant rouge
        [10:B]
       /      \
    [5:R]    [15:R]   ← OK (parent noir)
   /    \
[3:B]  [7:B]          ← Enfants de rouge sont noirs

Propriété 5 : Black height uniforme
        [10:B]        bh=2 de la racine
       /      \
    [5:B]    [15:B]   bh=1 partout
   /    \    /    \
[3:R][7:R][12:R][20:R]  bh=0

Chemin vers 3:  10(B) → 5(B) → 3(R) → NIL(B) = 3 noirs ✓
Chemin vers 12: 10(B) → 15(B) → 12(R) → NIL(B) = 3 noirs ✓
```

**Les 3 cas de Fix-up :**

```
CAS 1 : Oncle ROUGE
     G:B              G:R
    /   \            /   \
  P:R   U:R   →    P:B   U:B    (recoloration)
  /                /
N:R              N:R
                  ↑
              Propager fix-up vers G

CAS 2 : Oncle NOIR, zigzag (N est enfant intérieur)
     G:B              G:B
    /   \            /   \
  P:R   U:B   →    N:R   U:B    (rotation sur P)
    \              /
    N:R          P:R
                  ↓
              Devient CAS 3

CAS 3 : Oncle NOIR, ligne (N est enfant extérieur)
     G:B              P:B
    /   \            /   \
  P:R   U:B   →    N:R   G:R    (rotation sur G + recolor)
  /                        \
N:R                        U:B
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **Root non-noir** | Oubli de forcer root black | Toujours après insert |
| **Cas mal identifié** | Oncle rouge vs noir | Vérifier couleur oncle |
| **Rotation sans recolor** | Structure ok, couleurs fausses | Échanger couleurs |
| **Propagation oubliée** | Fix-up local seulement | Remonter vers racine |
| **NIL mal géré** | NIL considéré rouge | NIL = BLACK par défaut |

### 5.5 Cours Complet

#### Pourquoi Red-Black ?

Les arbres AVL sont **plus strictement équilibrés** (meilleure recherche) mais les RB-Trees nécessitent **moins de rotations** lors des modifications.

| Aspect | AVL | Red-Black |
|--------|-----|-----------|
| Équilibre | |bf| ≤ 1 | Black height |
| Hauteur max | 1.44 log n | 2 log n |
| Recherche | Plus rapide | Légèrement plus lent |
| Insertion | Plus de rotations | Moins de rotations |
| Utilisation | Lecture intensive | Écriture intensive |

#### Le Black-Height

Le **black-height** d'un nœud est le nombre de nœuds noirs sur tout chemin vers une feuille NIL (excluant le nœud lui-même).

**Théorème :** Un RB-Tree avec n nœuds a une hauteur ≤ 2×log₂(n+1)

**Preuve intuitive :**
- Le plus court chemin = tous noirs = bh nœuds
- Le plus long chemin = alternance R-B = 2×bh nœuds
- Donc rapport max = 2

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ if (node->left != NULL && node->left->color == RED)             │
│     /* ... */                                                   │
│ // Répétition du pattern pour chaque vérification               │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ fn is_red(node: &Option<Box<Node>>) -> bool {                   │
│     node.as_ref().map_or(false, |n| n.color == Color::Red)      │
│ }                                                               │
│ if is_red(&node.left) { /* ... */ }                             │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • DRY : Ne pas répéter la vérification NULL + couleur           │
│ • Lisibilité : `is_red()` est auto-documenté                    │
│ • Sécurité : Gère uniformément le cas NULL                      │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario : Insert 10, 5, 15, 3 (déclenche Case 1)**

```
┌───────┬─────────────────────────────────────┬──────────────┬───────────────────────┐
│ Étape │ Action                              │ Arbre        │ Explication           │
├───────┼─────────────────────────────────────┼──────────────┼───────────────────────┤
│   1   │ insert(10)                          │   [10:B]     │ Root → BLACK          │
├───────┼─────────────────────────────────────┼──────────────┼───────────────────────┤
│   2   │ insert(5)                           │   [10:B]     │ Parent noir = OK      │
│       │                                     │   /          │                       │
│       │                                     │ [5:R]        │                       │
├───────┼─────────────────────────────────────┼──────────────┼───────────────────────┤
│   3   │ insert(15)                          │   [10:B]     │ Parent noir = OK      │
│       │                                     │  /     \     │                       │
│       │                                     │[5:R] [15:R]  │                       │
├───────┼─────────────────────────────────────┼──────────────┼───────────────────────┤
│   4   │ insert(3)                           │   [10:B]     │ Parent=5:R, Uncle=15:R│
│       │ → CAS 1 détecté !                   │  /     \     │ → Recoloration !      │
│       │                                     │[5:R] [15:R]  │                       │
│       │                                     │ /            │                       │
│       │                                     │[3:R]         │                       │
├───────┼─────────────────────────────────────┼──────────────┼───────────────────────┤
│   5   │ Recoloration (flip_colors)          │   [10:B]     │ 5,15→B, 10→R          │
│       │                                     │  /     \     │ Mais 10 est root !    │
│       │                                     │[5:B] [15:B]  │                       │
│       │                                     │ /            │                       │
│       │                                     │[3:R]         │                       │
├───────┼─────────────────────────────────────┼──────────────┼───────────────────────┤
│   6   │ Force root BLACK                    │   [10:B]     │ Propriété 2 OK ✓      │
│       │                                     │  /     \     │ Propriété 4 OK ✓      │
│       │                                     │[5:B] [15:B]  │ Propriété 5 OK ✓      │
│       │                                     │ /            │ bh = 2 partout        │
│       │                                     │[3:R]         │                       │
└───────┴─────────────────────────────────────┴──────────────┴───────────────────────┘
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### ♟️ MEME : "Red Wedding" — Quand deux rouges se rencontrent

*"The Lannisters send their regards..."* 🔴🔴💀

Dans Game of Thrones, la "Red Wedding" est un massacre. Dans un RB-Tree, deux nœuds rouges consécutifs sont aussi une catastrophe !

```
     [B]
    /
  [R]         ← OK jusqu'ici
  /
[R]           ← RED WEDDING ! Violation de propriété 4 !
```

**Solution :** Faire un "fix-up" pour éviter le bain de sang :
- Si oncle rouge → recolorer (pardon royal)
- Si oncle noir → rotation (changement de pouvoir)

#### 🎯 Mnémotechnique des propriétés :

```
"1. Every node is Red or Black" → EXISTENCE
"2. Root is Black" → ROYALTY (le roi est en noir)
"3. NIL are Black" → EDGE (les frontières sont sûres)
"4. Red → Black children" → NO RED WEDDING
"5. Same black height" → FAIR PATHS
```

### 5.9 Applications pratiques

| Application | Pourquoi RB-Tree |
|-------------|------------------|
| **std::map (C++)** | Insert/delete fréquents |
| **Linux CFS** | Scheduling équitable |
| **Java TreeMap** | Interface Map ordonnée |
| **Databases** | Index B-Tree (variante) |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Comment l'éviter |
|---|-------|--------|------------------|
| 1 | Root rouge | Propriété 2 violée | Force BLACK après insert |
| 2 | Red-Red | Propriété 4 violée | Fix-up systématique |
| 3 | Mauvais cas | Fix incorrect | Vérifier oncle d'abord |
| 4 | Pas de recolor | Couleurs fausses | Toujours recolorer après rotation |
| 5 | Black height | Propriété 5 violée | Tester avec bh() |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quelle est la couleur d'un nouveau nœud inséré dans un RB-Tree ?**

A) Noir
B) Rouge
C) Dépend de la position
D) Dépend du parent
E) Alternance
F) Aléatoire
G) Celui de l'oncle
H) L'opposé du parent
I) B puis fix-up le change si nécessaire
J) Rouge car ça ne peut pas violer la propriété 5

**Réponse : J**

### Question 2
**Dans le Cas 1 du fix-up (oncle rouge), que fait-on ?**

A) Rotation gauche
B) Rotation droite
C) Double rotation
D) Recoloration seulement
E) Suppression de l'oncle
F) Échange parent-oncle
G) D puis propagation vers le haut
H) Rien
I) Rotation + recoloration
J) Dépend du grand-parent

**Réponse : G**

### Question 3
**Quelle est la hauteur maximale d'un RB-Tree avec 15 nœuds ?**

A) 4
B) 5
C) 6
D) 7
E) 8
F) 2 × log₂(16) = 8
G) log₂(15) ≈ 4
H) 15
I) Impossible à déterminer
J) F arrondi = 8

**Réponse : J** (2 × log₂(n+1) ≈ 8)

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.3.2-a — chess_rb_tree |
| **Concept principal** | Red-Black Tree, 5 propriétés |
| **Difficulté** | ★★★★★★★☆☆☆ (7/10) |
| **Temps estimé** | 90 min |
| **XP Base** | 250 |
| **Bonus Delete** | 💀 Expert (×4) |
| **Bonus LLRB** | 🧠 Génie (×6) |
| **Langage** | Rust 2024 + C (c17) |
| **Points clés** | 5 propriétés, 3 cas fix-up, black-height |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.3.2-a-chess-rb-tree",
    "generated_at": "2025-01-11 15:00:00",

    "metadata": {
      "exercise_id": "1.3.2-a",
      "exercise_name": "chess_rb_tree",
      "module": "1.3.2",
      "module_name": "Red-Black Trees",
      "concept": "a",
      "concept_name": "RB Tree 5 Properties",
      "type": "complet",
      "tier": 1,
      "tier_info": "Concept isolé",
      "phase": 1,
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "language": "rust",
      "language_alt": "c",
      "duration_minutes": 90,
      "xp_base": 250,
      "xp_bonus_multiplier": 4,
      "bonus_tier": "EXPERT",
      "bonus_icon": "💀",
      "complexity_time": "T3 O(log n)",
      "complexity_space": "S2 O(n)",
      "prerequisites": ["BST", "AVL rotations", "coloring"],
      "domains": ["Struct", "Mem", "MD"],
      "tags": ["rb-tree", "trees", "balance", "coloring"],
      "meme_reference": "Red Wedding - Game of Thrones"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "Red Wedding: when two reds meet, someone must change color"*
