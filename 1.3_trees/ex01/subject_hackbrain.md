<thinking>
## Analyse du Concept
- Concept : AVL Trees - Arbres auto-équilibrés avec rotations
- Phase demandée : 1 (intermédiaire)
- Adapté ? OUI - Les AVL introduisent l'équilibrage, concept crucial pour les performances

## Combo Base + Bonus
- Exercice de base : Implémenter AVL avec les 4 rotations (LL, RR, LR, RL), insert, delete
- Bonus : Validation de propriété AVL + visualisation de l'arbre + bulk construction O(n)
- Palier bonus : 🔥 Avancé pour validation, 💀 Expert pour construction optimale
- Progression logique ? OUI - Base = opérations, Bonus = validation et optimisation

## Prérequis & Difficulté
- Prérequis réels : BST (ex00), rotations, balance factor
- Difficulté estimée : 6/10 (base), 8/10 (bonus)
- Cohérent avec phase 1 ? OUI - Progression naturelle depuis BST

## Aspect Fun/Culture
- Contexte choisi : Avatar: The Last Airbender - L'équilibre des 4 éléments
- MEME mnémotechnique : "Everything changed when the Fire Nation attacked" → quand l'arbre se déséquilibre
- Pourquoi c'est fun : Les 4 rotations = 4 éléments (LL=Eau, RR=Feu, LR=Terre, RL=Air), Aang doit maintenir l'équilibre

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : Balance factor calculé comme |height(left) - height(right)| au lieu de height(left) - height(right) (perd le signe)
2. Mutant B (Safety) : Oubli de mettre à jour la hauteur après rotation
3. Mutant C (Logic) : Rotation LL appliquée quand c'est un cas LR (double rotation nécessaire)
4. Mutant D (Logic) : Rotation dans le mauvais sens (rotate_right au lieu de rotate_left)
5. Mutant E (Return) : Ne pas retourner le nouveau root après rotation

## Verdict
VALIDE - L'analogie Avatar/équilibre est parfaite pour les rotations AVL
</thinking>

---

# Exercice 1.3.1-a : avatar_balance_tree

**Module :**
1.3.1 — AVL Trees (Self-Balancing BST)

**Concept :**
a — Rotations AVL et équilibrage automatique

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
1 — Concept isolé

**Langage :**
Rust Edition 2024, C (c17)

**Prérequis :**
- Binary Search Tree (exercice 1.3.0)
- Récursion et gestion de hauteur
- Concept de balance factor

**Domaines :**
Struct, Mem, MD

**Durée estimée :**
60 min

**XP Base :**
200

**Complexité :**
T[3] O(log n) garantie × S[2] O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `avatar_balance_tree.c`, `avatar_balance_tree.h`

**Fonctions autorisées :**
- C : `malloc`, `free`, `NULL`
- Rust : `Box::new`, `Option`, `std::cmp::Ordering`, `std::cmp::max`

**Fonctions interdites :**
- C : `calloc`, `realloc`, bibliothèques d'arbres
- Rust : `BTreeMap`, `BTreeSet`, collections auto-équilibrées

### 1.2 Consigne

**🌊🔥🪨💨 AVATAR : L'ARBRE DES QUATRE ÉLÉMENTS**

*"L'eau. La terre. Le feu. L'air. Il y a longtemps, les quatre nations vivaient en harmonie. Puis tout a changé quand l'arbre s'est déséquilibré..."*

Dans le monde d'Avatar, l'Avatar doit maîtriser les **4 éléments** pour maintenir l'équilibre. Dans le monde des structures de données, tu dois maîtriser les **4 rotations** pour maintenir l'équilibre de ton arbre AVL !

| Élément | Rotation | Cas |
|---------|----------|-----|
| 🌊 Eau (fluide, vers la droite) | LL - rotate_right | Déséquilibre gauche-gauche |
| 🔥 Feu (agressif, vers la gauche) | RR - rotate_left | Déséquilibre droite-droite |
| 🪨 Terre (stable, double) | LR - left then right | Déséquilibre gauche-droite |
| 💨 Air (souple, double) | RL - right then left | Déséquilibre droite-gauche |

**Ta mission :**

Créer une structure `AvatarTree<K, V>` qui implémente un **AVL Tree** :

1. **Maintenir l'équilibre** : Balance factor ∈ {-1, 0, 1} pour tout nœud
2. **Les 4 rotations** : Implémenter LL, RR, LR, RL
3. **Insert avec rééquilibrage** : Après insertion, vérifier et corriger
4. **Delete avec rééquilibrage** : Après suppression, vérifier et corriger
5. **Garantir O(log n)** : Grâce à l'équilibrage, la hauteur reste logarithmique

**Le Balance Factor :**
```
balance_factor(node) = height(left_subtree) - height(right_subtree)
```
- Si `bf > 1` : Trop lourd à gauche → rotation(s) vers la droite
- Si `bf < -1` : Trop lourd à droite → rotation(s) vers la gauche
- Si `bf ∈ {-1, 0, 1}` : Équilibré, Aang est content 🧘

**Entrée :**
- `key` : Clé comparable
- `value` : Valeur associée

**Sortie :**
- `insert` : L'arbre reste équilibré après insertion
- `delete` : L'arbre reste équilibré après suppression
- `height` : Hauteur garantie O(log n)

**Contraintes :**
- Le balance factor de TOUT nœud doit être dans {-1, 0, 1}
- Les rotations doivent préserver la propriété BST
- Mise à jour des hauteurs après chaque rotation

**Exemples :**

| Opération | Avant | Après | Rotation effectuée |
|-----------|-------|-------|-------------------|
| `insert(3)` puis `insert(2)` puis `insert(1)` | Chaîne gauche | Arbre équilibré | LL (rotate_right) |
| `insert(1)` puis `insert(3)` puis `insert(2)` | Zigzag gauche | Arbre équilibré | LR (left puis right) |

### 1.3 Prototype

**Rust :**
```rust
pub struct AvatarTree<K: Ord, V> {
    root: Option<Box<AvatarNode<K, V>>>,
}

struct AvatarNode<K: Ord, V> {
    key: K,
    value: V,
    height: i32,  // Hauteur du sous-arbre
    left: Option<Box<AvatarNode<K, V>>>,
    right: Option<Box<AvatarNode<K, V>>>,
}

impl<K: Ord, V> AvatarTree<K, V> {
    pub fn new() -> Self;

    // Opérations principales
    pub fn insert(&mut self, key: K, value: V);
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn remove(&mut self, key: &K) -> Option<V>;

    // Les 4 rotations (éléments)
    fn water_rotation(node: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>>;  // LL
    fn fire_rotation(node: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>>;   // RR
    fn earth_rotation(node: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>>;  // LR
    fn air_rotation(node: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>>;    // RL

    // Utilitaires
    fn balance_factor(node: &AvatarNode<K, V>) -> i32;
    fn update_height(node: &mut AvatarNode<K, V>);
    fn rebalance(node: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>>;

    // Vérification
    pub fn is_balanced(&self) -> bool;
    pub fn height(&self) -> i32;
    pub fn inorder(&self) -> Vec<(&K, &V)>;
}
```

**C :**
```c
typedef struct s_avatar_node {
    int                     key;
    char                    *value;
    int                     height;
    struct s_avatar_node    *left;
    struct s_avatar_node    *right;
} t_avatar_node;

typedef struct s_avatar_tree {
    t_avatar_node   *root;
    size_t          size;
} t_avatar_tree;

// Création/Destruction
t_avatar_tree   *avatar_new(void);
void            avatar_free(t_avatar_tree *tree);

// Opérations principales
void            avatar_insert(t_avatar_tree *tree, int key, char *value);
char            *avatar_search(t_avatar_tree *tree, int key);
int             avatar_delete(t_avatar_tree *tree, int key);

// Les 4 rotations
t_avatar_node   *water_rotation(t_avatar_node *node);   // LL - rotate_right
t_avatar_node   *fire_rotation(t_avatar_node *node);    // RR - rotate_left
t_avatar_node   *earth_rotation(t_avatar_node *node);   // LR
t_avatar_node   *air_rotation(t_avatar_node *node);     // RL

// Utilitaires
int             avatar_balance_factor(t_avatar_node *node);
void            avatar_update_height(t_avatar_node *node);
t_avatar_node   *avatar_rebalance(t_avatar_node *node);

// Vérification
int             avatar_is_balanced(t_avatar_tree *tree);
int             avatar_height(t_avatar_tree *tree);
```

### 1.2.2 Énoncé Académique

Un **arbre AVL** (Adelson-Velsky et Landis, 1962) est un arbre binaire de recherche auto-équilibré où la différence de hauteur entre les sous-arbres gauche et droit de tout nœud est au plus 1.

**Propriété AVL :**
> Pour tout nœud N : |height(N.left) - height(N.right)| ≤ 1

**Les 4 cas de déséquilibre :**

1. **LL (Left-Left)** : Insertion dans le sous-arbre gauche du fils gauche
   - Solution : Simple rotation droite

2. **RR (Right-Right)** : Insertion dans le sous-arbre droit du fils droit
   - Solution : Simple rotation gauche

3. **LR (Left-Right)** : Insertion dans le sous-arbre droit du fils gauche
   - Solution : Rotation gauche sur fils gauche, puis rotation droite

4. **RL (Right-Left)** : Insertion dans le sous-arbre gauche du fils droit
   - Solution : Rotation droite sur fils droit, puis rotation gauche

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Histoire de l'AVL

L'arbre AVL a été inventé en 1962 par **Georgy Adelson-Velsky** et **Evgenii Landis**, deux mathématiciens soviétiques. C'est le **premier arbre auto-équilibré** jamais inventé !

**Fun fact :** Le nom "AVL" vient simplement des initiales de ses créateurs (Adelson-Velsky et Landis).

### 2.2 AVL vs Red-Black

| Critère | AVL | Red-Black |
|---------|-----|-----------|
| Équilibre | Plus strict (±1) | Plus lâche |
| Recherche | Plus rapide | Légèrement plus lent |
| Insertion | Plus de rotations | Moins de rotations |
| Utilisation | Recherche intensive | Insertion intensive |

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation | Cas concret |
|--------|-------------|-------------|
| **Database Engineer** | Index en mémoire | PostgreSQL pour certains index |
| **System Developer** | Kernel memory management | Linux pour la gestion mémoire |
| **Financial Engineer** | Trading systems | Order matching engines |
| **Game Developer** | Scene graphs équilibrés | Organisation spatiale 3D |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
avatar_balance_tree.c  avatar_balance_tree.h  main.c  Cargo.toml  src/

$ gcc -Wall -Wextra -Werror -std=c17 avatar_balance_tree.c main.c -o test_c

$ ./test_c
=== Test AVL Avatar Tree ===
Insert 30: OK (height=1)
Insert 20: OK (height=2)
Insert 10: ROTATION LL (Water)! New root: 20 (height=2)
Tree balanced: YES

Insert 25: OK (height=3)
Insert 5: OK (height=3)
Insert 1: ROTATION LL (Water)! (height=3)
Tree balanced: YES

Inorder: 1 5 10 20 25 30
Height: 3 (optimal for 6 nodes)

Delete 20: REBALANCE needed
Tree balanced: YES

All tests passed!

$ cargo test
   Compiling avatar_balance v0.1.0
    Finished test [unoptimized + debuginfo]
     Running unittests src/lib.rs

running 15 tests
test tests::test_insert_simple ... ok
test tests::test_ll_rotation ... ok
test tests::test_rr_rotation ... ok
test tests::test_lr_rotation ... ok
test tests::test_rl_rotation ... ok
test tests::test_delete_with_rebalance ... ok
test tests::test_is_balanced ... ok
test tests::test_height_guarantee ... ok
test tests::test_stress_1000_inserts ... ok

test result: ok. 15 passed; 0 failed
```

### 3.1 🔥 BONUS AVANCÉ : Validation AVL Stricte (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Domaines Bonus :**
`MD`

#### 3.1.1 Consigne Bonus

**🧘 LE TEST DE L'AVATAR — Prouver la Maîtrise**

*"Avant de devenir Avatar, tu dois prouver que tu maintiens l'équilibre parfait..."*

Implémenter une fonction `validate_avl()` qui vérifie TOUTES les propriétés AVL :

1. Propriété BST (gauche < racine < droite)
2. Balance factor ∈ {-1, 0, 1} pour TOUS les nœuds
3. Les hauteurs stockées sont correctes
4. Pas de cycles (arbre valide)

```rust
impl<K: Ord, V> AvatarTree<K, V> {
    /// Validate all AVL properties - returns detailed error if invalid
    pub fn validate_avl(&self) -> Result<(), AvlError>;
}

pub enum AvlError {
    BstViolation { node_key: String, violating_key: String },
    BalanceViolation { node_key: String, balance_factor: i32 },
    HeightMismatch { node_key: String, stored: i32, actual: i32 },
    CycleDetected,
}
```

### 3.2 💀 BONUS EXPERT : Construction O(n) depuis tableau trié (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×4

#### 3.2.1 Consigne Bonus

Construire un AVL parfaitement équilibré depuis un tableau trié en O(n) au lieu de O(n log n).

```rust
impl<K: Ord, V> AvatarTree<K, V> {
    /// Build balanced AVL from sorted array in O(n)
    pub fn from_sorted(items: Vec<(K, V)>) -> Self;
}
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `test_new` | `AvatarTree::new()` | `is_balanced() == true` | 2 | |
| `test_insert_single` | `insert(5, "A")` | `height == 1` | 3 | |
| `test_ll_case` | `insert(30,20,10)` | Root = 20, balanced | 10 | ⚠️ |
| `test_rr_case` | `insert(10,20,30)` | Root = 20, balanced | 10 | ⚠️ |
| `test_lr_case` | `insert(30,10,20)` | Root = 20, balanced | 12 | ⚠️ |
| `test_rl_case` | `insert(10,30,20)` | Root = 20, balanced | 12 | ⚠️ |
| `test_delete_leaf` | Delete feuille | Balanced après | 8 | |
| `test_delete_rebalance` | Delete cause déséquilibre | Rotation effectuée | 12 | ⚠️ |
| `test_height_update` | Série d'ops | Hauteurs correctes | 8 | |
| `test_balance_factors` | Arbre complexe | Tous bf ∈ {-1,0,1} | 10 | |
| `test_stress_100` | 100 insertions aléatoires | Toujours équilibré | 8 | |
| `test_inorder_sorted` | Arbre quelconque | Inorder = trié | 5 | |
| **TOTAL** | | | **100** | |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <assert.h>
#include "avatar_balance_tree.h"

void test_ll_rotation(void)
{
    t_avatar_tree *tree = avatar_new();

    // Insert 30, 20, 10 -> should trigger LL rotation
    avatar_insert(tree, 30, "Zuko");
    avatar_insert(tree, 20, "Katara");
    avatar_insert(tree, 10, "Aang");

    // After LL rotation, root should be 20
    assert(tree->root->key == 20);
    assert(tree->root->left->key == 10);
    assert(tree->root->right->key == 30);
    assert(avatar_is_balanced(tree) == 1);

    avatar_free(tree);
    printf("test_ll_rotation (Water): OK\n");
}

void test_rr_rotation(void)
{
    t_avatar_tree *tree = avatar_new();

    // Insert 10, 20, 30 -> should trigger RR rotation
    avatar_insert(tree, 10, "Aang");
    avatar_insert(tree, 20, "Katara");
    avatar_insert(tree, 30, "Zuko");

    // After RR rotation, root should be 20
    assert(tree->root->key == 20);
    assert(avatar_is_balanced(tree) == 1);

    avatar_free(tree);
    printf("test_rr_rotation (Fire): OK\n");
}

void test_lr_rotation(void)
{
    t_avatar_tree *tree = avatar_new();

    // Insert 30, 10, 20 -> should trigger LR rotation
    avatar_insert(tree, 30, "Zuko");
    avatar_insert(tree, 10, "Aang");
    avatar_insert(tree, 20, "Katara");

    // After LR rotation, root should be 20
    assert(tree->root->key == 20);
    assert(avatar_is_balanced(tree) == 1);

    avatar_free(tree);
    printf("test_lr_rotation (Earth): OK\n");
}

void test_rl_rotation(void)
{
    t_avatar_tree *tree = avatar_new();

    // Insert 10, 30, 20 -> should trigger RL rotation
    avatar_insert(tree, 10, "Aang");
    avatar_insert(tree, 30, "Zuko");
    avatar_insert(tree, 20, "Katara");

    // After RL rotation, root should be 20
    assert(tree->root->key == 20);
    assert(avatar_is_balanced(tree) == 1);

    avatar_free(tree);
    printf("test_rl_rotation (Air): OK\n");
}

void test_stress(void)
{
    t_avatar_tree *tree = avatar_new();

    // Insert 1 to 100
    for (int i = 1; i <= 100; i++)
    {
        avatar_insert(tree, i, "test");
        assert(avatar_is_balanced(tree) == 1);
    }

    // Height should be O(log n) ~ 7 for 100 nodes
    int h = avatar_height(tree);
    assert(h <= 8);  // ceil(1.44 * log2(100)) ≈ 7

    avatar_free(tree);
    printf("test_stress (100 insertions): OK (height=%d)\n", h);
}

int main(void)
{
    printf("=== Tests AVL Avatar Tree ===\n");
    test_ll_rotation();
    test_rr_rotation();
    test_lr_rotation();
    test_rl_rotation();
    test_stress();
    printf("\nAll tests passed! 🌊🔥🪨💨\n");
    return 0;
}
```

### 4.3 Solution de référence

**Rust :**
```rust
use std::cmp::{max, Ordering};

pub struct AvatarTree<K: Ord, V> {
    root: Option<Box<AvatarNode<K, V>>>,
}

struct AvatarNode<K: Ord, V> {
    key: K,
    value: V,
    height: i32,
    left: Option<Box<AvatarNode<K, V>>>,
    right: Option<Box<AvatarNode<K, V>>>,
}

impl<K: Ord, V> AvatarNode<K, V> {
    fn new(key: K, value: V) -> Self {
        AvatarNode {
            key,
            value,
            height: 1,
            left: None,
            right: None,
        }
    }

    fn height(node: &Option<Box<AvatarNode<K, V>>>) -> i32 {
        node.as_ref().map_or(0, |n| n.height)
    }

    fn update_height(&mut self) {
        self.height = 1 + max(
            Self::height(&self.left),
            Self::height(&self.right)
        );
    }

    fn balance_factor(&self) -> i32 {
        Self::height(&self.left) - Self::height(&self.right)
    }
}

impl<K: Ord, V> AvatarTree<K, V> {
    pub fn new() -> Self {
        AvatarTree { root: None }
    }

    pub fn insert(&mut self, key: K, value: V) {
        self.root = Self::insert_rec(self.root.take(), key, value);
    }

    fn insert_rec(
        node: Option<Box<AvatarNode<K, V>>>,
        key: K,
        value: V,
    ) -> Option<Box<AvatarNode<K, V>>> {
        let mut node = match node {
            None => return Some(Box::new(AvatarNode::new(key, value))),
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

        node.update_height();
        Some(Self::rebalance(node))
    }

    fn rebalance(mut node: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>> {
        let bf = node.balance_factor();

        // LL Case - Water Rotation (rotate right)
        if bf > 1 && node.left.as_ref().map_or(0, |n| n.balance_factor()) >= 0 {
            return Self::water_rotation(node);
        }

        // RR Case - Fire Rotation (rotate left)
        if bf < -1 && node.right.as_ref().map_or(0, |n| n.balance_factor()) <= 0 {
            return Self::fire_rotation(node);
        }

        // LR Case - Earth Rotation (left then right)
        if bf > 1 && node.left.as_ref().map_or(0, |n| n.balance_factor()) < 0 {
            node.left = Some(Self::fire_rotation(node.left.take().unwrap()));
            return Self::water_rotation(node);
        }

        // RL Case - Air Rotation (right then left)
        if bf < -1 && node.right.as_ref().map_or(0, |n| n.balance_factor()) > 0 {
            node.right = Some(Self::water_rotation(node.right.take().unwrap()));
            return Self::fire_rotation(node);
        }

        node
    }

    // LL Case - Rotate Right (Water: flows right)
    fn water_rotation(mut y: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>> {
        let mut x = y.left.take().unwrap();
        y.left = x.right.take();
        y.update_height();
        x.right = Some(y);
        x.update_height();
        x
    }

    // RR Case - Rotate Left (Fire: aggressive left)
    fn fire_rotation(mut x: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>> {
        let mut y = x.right.take().unwrap();
        x.right = y.left.take();
        x.update_height();
        y.left = Some(x);
        y.update_height();
        y
    }

    pub fn get(&self, key: &K) -> Option<&V> {
        fn search<K: Ord, V>(node: &Option<Box<AvatarNode<K, V>>>, key: &K) -> Option<&V> {
            let n = node.as_ref()?;
            match key.cmp(&n.key) {
                Ordering::Equal => Some(&n.value),
                Ordering::Less => search(&n.left, key),
                Ordering::Greater => search(&n.right, key),
            }
        }
        search(&self.root, key)
    }

    pub fn is_balanced(&self) -> bool {
        fn check<K: Ord, V>(node: &Option<Box<AvatarNode<K, V>>>) -> bool {
            match node {
                None => true,
                Some(n) => {
                    let bf = n.balance_factor();
                    bf >= -1 && bf <= 1 && check(&n.left) && check(&n.right)
                }
            }
        }
        check(&self.root)
    }

    pub fn height(&self) -> i32 {
        AvatarNode::height(&self.root)
    }

    pub fn inorder(&self) -> Vec<(&K, &V)> {
        fn collect<'a, K: Ord, V>(
            node: &'a Option<Box<AvatarNode<K, V>>>,
            result: &mut Vec<(&'a K, &'a V)>,
        ) {
            if let Some(n) = node {
                collect(&n.left, result);
                result.push((&n.key, &n.value));
                collect(&n.right, result);
            }
        }
        let mut result = Vec::new();
        collect(&self.root, &mut result);
        result
    }
}

impl<K: Ord, V> Default for AvatarTree<K, V> {
    fn default() -> Self {
        Self::new()
    }
}
```

### 4.9 spec.json

```json
{
  "name": "avatar_balance_tree",
  "language": "rust",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isolé - AVL rotations",
  "tags": ["avl", "trees", "rotations", "balance", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "AvatarTree",
    "prototype": "pub struct AvatarTree<K: Ord, V>",
    "return_type": "struct",
    "parameters": []
  },

  "driver": {
    "reference": "/* See section 4.3 for full implementation */",

    "edge_cases": [
      {
        "name": "ll_rotation",
        "args": ["30", "20", "10"],
        "expected": "root == 20, balanced",
        "is_trap": true,
        "trap_explanation": "Cas LL (gauche-gauche) nécessite rotation droite"
      },
      {
        "name": "rr_rotation",
        "args": ["10", "20", "30"],
        "expected": "root == 20, balanced",
        "is_trap": true,
        "trap_explanation": "Cas RR (droite-droite) nécessite rotation gauche"
      },
      {
        "name": "lr_rotation",
        "args": ["30", "10", "20"],
        "expected": "root == 20, balanced",
        "is_trap": true,
        "trap_explanation": "Cas LR nécessite double rotation"
      },
      {
        "name": "rl_rotation",
        "args": ["10", "30", "20"],
        "expected": "root == 20, balanced",
        "is_trap": true,
        "trap_explanation": "Cas RL nécessite double rotation"
      },
      {
        "name": "height_update",
        "args": ["series"],
        "expected": "all heights correct",
        "is_trap": true,
        "trap_explanation": "Les hauteurs doivent être mises à jour après rotation"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 1000,
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
    "allowed_functions": ["Box::new", "Option", "std::cmp::max"],
    "forbidden_functions": ["BTreeMap", "BTreeSet"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```rust
/* Mutant A (Boundary) : Balance factor sans signe (valeur absolue) */
fn balance_factor_mutant_a(&self) -> i32 {
    // BUG: Utilise abs() - perd l'information de direction
    (AvatarNode::height(&self.left) - AvatarNode::height(&self.right)).abs()
}
// Pourquoi c'est faux : On ne sait plus si c'est gauche ou droite qui est trop lourd
// Ce qui était pensé : "|bf| > 1 suffit" — NON, on doit savoir la direction !

/* Mutant B (Safety) : Oubli de update_height après rotation */
fn water_rotation_mutant_b(mut y: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>> {
    let mut x = y.left.take().unwrap();
    y.left = x.right.take();
    // BUG: Oubli de y.update_height();
    x.right = Some(y);
    // BUG: Oubli de x.update_height();
    x
}
// Pourquoi c'est faux : Les hauteurs stockées deviennent incorrectes
// Ce qui était pensé : "La structure est correcte" — mais les hauteurs non !

/* Mutant C (Logic) : Mauvaise rotation pour cas LR */
fn rebalance_mutant_c(mut node: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>> {
    let bf = node.balance_factor();

    // Cas LR
    if bf > 1 && node.left.as_ref().map_or(0, |n| n.balance_factor()) < 0 {
        // BUG: Applique water_rotation directement au lieu de fire puis water
        return Self::water_rotation(node);
    }
    node
}
// Pourquoi c'est faux : Le cas LR nécessite une double rotation
// Ce qui était pensé : "Une rotation suffit" — NON pour les cas zigzag !

/* Mutant D (Logic) : Rotation dans le mauvais sens */
fn water_rotation_mutant_d(mut y: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>> {
    // BUG: Fait une rotation gauche au lieu de droite
    let mut x = y.right.take().unwrap();  // Devrait être .left
    y.right = x.left.take();              // Sens inversé
    y.update_height();
    x.left = Some(y);                     // Sens inversé
    x.update_height();
    x
}
// Pourquoi c'est faux : Rotation dans le mauvais sens viole BST
// Ce qui était pensé : "left/right c'est pareil" — NON !

/* Mutant E (Return) : Ne retourne pas le nouveau root */
fn rebalance_mutant_e(mut node: Box<AvatarNode<K, V>>) -> Box<AvatarNode<K, V>> {
    let bf = node.balance_factor();

    if bf > 1 && node.left.as_ref().map_or(0, |n| n.balance_factor()) >= 0 {
        Self::water_rotation(node);  // BUG: Ne capture pas le résultat
        // return manquant, tombe dans le return node final
    }
    node  // Retourne l'ancien node au lieu du nouveau
}
// Pourquoi c'est faux : L'arbre n'est pas modifié car on ignore le nouveau root
// Ce qui était pensé : "La rotation modifie en place" — NON, elle retourne !
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

1. **Auto-équilibrage** : Maintenir une propriété invariante après chaque modification
2. **Les 4 rotations** : Comprendre quand et comment les appliquer
3. **Balance factor** : Métrique de déséquilibre
4. **Garantie O(log n)** : Comment l'équilibrage garantit les performances
5. **Mise à jour de métadonnées** : Maintenir les hauteurs correctes

### 5.2 LDA — Traduction Littérale en Français (MAJUSCULES)

```
FONCTION insert QUI PREND key ET value
DÉBUT FONCTION
    SI root EST NUL ALORS
        CRÉER UN NOUVEAU NŒUD AVEC key, value, height=1
        RETOURNER
    FIN SI

    SI key EST INFÉRIEUR À root.key ALORS
        INSÉRER RÉCURSIVEMENT DANS LE SOUS-ARBRE GAUCHE
    SINON SI key EST SUPÉRIEUR À root.key ALORS
        INSÉRER RÉCURSIVEMENT DANS LE SOUS-ARBRE DROIT
    SINON
        METTRE À JOUR root.value AVEC value
        RETOURNER
    FIN SI

    METTRE À JOUR LA HAUTEUR DU NŒUD COURANT
    CALCULER LE BALANCE FACTOR

    SI bf EST SUPÉRIEUR À 1 ALORS
        SI bf DU FILS GAUCHE EST SUPÉRIEUR OU ÉGAL À 0 ALORS
            APPLIQUER ROTATION WATER (LL)
        SINON
            APPLIQUER ROTATION EARTH (LR)
        FIN SI
    SINON SI bf EST INFÉRIEUR À -1 ALORS
        SI bf DU FILS DROIT EST INFÉRIEUR OU ÉGAL À 0 ALORS
            APPLIQUER ROTATION FIRE (RR)
        SINON
            APPLIQUER ROTATION AIR (RL)
        FIN SI
    FIN SI
FIN FONCTION

FONCTION water_rotation (rotate_right) QUI PREND y
DÉBUT FONCTION
    AFFECTER y.left À x
    AFFECTER x.right À y.left
    METTRE À JOUR LA HAUTEUR DE y
    AFFECTER y À x.right
    METTRE À JOUR LA HAUTEUR DE x
    RETOURNER x COMME NOUVEAU ROOT
FIN FONCTION
```

### 5.3 Visualisation ASCII

**Les 4 Rotations illustrées :**

```
🌊 WATER ROTATION (LL Case - Rotate Right)

      y                 x
     / \               / \
    x   C    →        A   y
   / \                   / \
  A   B                 B   C

Déclencheur: bf(y) > 1 ET bf(x) >= 0


🔥 FIRE ROTATION (RR Case - Rotate Left)

    x                   y
   / \                 / \
  A   y       →       x   C
     / \             / \
    B   C           A   B

Déclencheur: bf(x) < -1 ET bf(y) <= 0


🪨 EARTH ROTATION (LR Case - Left then Right)

      z                z              y
     / \              / \            / \
    x   D    →       y   D    →     x   z
   / \              / \            /|   |\
  A   y            x   C          A B   C D
     / \          / \
    B   C        A   B

Déclencheur: bf(z) > 1 ET bf(x) < 0


💨 AIR ROTATION (RL Case - Right then Left)

    z                  z                y
   / \                / \              / \
  A   x      →       A   y      →     z   x
     / \                / \          /|   |\
    y   D              B   x        A B   C D
   / \                    / \
  B   C                  C   D

Déclencheur: bf(z) < -1 ET bf(x) > 0
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **Oubli update_height** | Hauteurs fausses après rotation | TOUJOURS appeler update_height() |
| **bf signé vs absolu** | Ne sait plus la direction | Garder le signe du balance factor |
| **Mauvais cas détecté** | LR traité comme LL | Vérifier bf du fils aussi |
| **Rotation sens inversé** | Viole propriété BST | Mémoriser: water=right, fire=left |
| **Ne pas retourner** | Ancien root conservé | Toujours return après rotation |

### 5.5 Cours Complet

#### Pourquoi l'équilibrage est important ?

Dans un BST non équilibré, la hauteur peut être O(n) (liste chaînée).
Dans un AVL, la hauteur est **garantie** O(log n).

**Théorème :** Un AVL de n nœuds a une hauteur h ≤ 1.44 × log₂(n+2) - 0.328

#### Le Balance Factor

```
balance_factor(node) = height(left) - height(right)
```

| bf | Signification |
|----|---------------|
| -1 | Légèrement penché à droite (OK) |
| 0 | Parfaitement équilibré (OK) |
| +1 | Légèrement penché à gauche (OK) |
| > +1 | Trop lourd à gauche (ROTATION !) |
| < -1 | Trop lourd à droite (ROTATION !) |

#### Comment détecter le cas ?

```
Si bf > 1 (trop lourd à gauche):
    Si bf(left) >= 0 : Cas LL → Water rotation
    Si bf(left) < 0  : Cas LR → Earth rotation

Si bf < -1 (trop lourd à droite):
    Si bf(right) <= 0 : Cas RR → Fire rotation
    Si bf(right) > 0  : Cas RL → Air rotation
```

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ node.height = 1 + max(left.height, right.height)                │
│ // Sans vérifier si left/right sont NULL                        │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ fn height(node: &Option<Box<Node>>) -> i32 {                    │
│     node.as_ref().map_or(0, |n| n.height)                       │
│ }                                                               │
│ self.height = 1 + max(height(&self.left), height(&self.right)); │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Sécurité : Un nœud absent a hauteur 0, pas undefined          │
│ • Robustesse : Pas de crash sur arbre incomplet                 │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario : Insert 30, 20, 10 (cas LL)**

```
┌───────┬─────────────────────────────────────────┬────────────┬───────────────────────┐
│ Étape │ Action                                  │ Arbre      │ Balance Factors       │
├───────┼─────────────────────────────────────────┼────────────┼───────────────────────┤
│   1   │ insert(30)                              │    [30]    │ bf(30) = 0            │
├───────┼─────────────────────────────────────────┼────────────┼───────────────────────┤
│   2   │ insert(20)                              │    [30]    │ bf(30) = 1 (OK)       │
│       │                                         │    /       │ bf(20) = 0            │
│       │                                         │  [20]      │                       │
├───────┼─────────────────────────────────────────┼────────────┼───────────────────────┤
│   3   │ insert(10)                              │    [30]    │ bf(30) = 2 (VIOLÉ!)   │
│       │                                         │    /       │ bf(20) = 1            │
│       │                                         │  [20]      │ bf(10) = 0            │
│       │                                         │  /         │                       │
│       │                                         │[10]        │ CAS LL DÉTECTÉ        │
├───────┼─────────────────────────────────────────┼────────────┼───────────────────────┤
│   4   │ WATER ROTATION sur 30                   │    [20]    │ bf(20) = 0            │
│       │ (rotate_right)                          │   /  \     │ bf(10) = 0            │
│       │                                         │ [10] [30]  │ bf(30) = 0            │
│       │                                         │            │ ÉQUILIBRÉ ✓           │
└───────┴─────────────────────────────────────────┴────────────┴───────────────────────┘
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🌊🔥🪨💨 MEME : "Avatar State Activated!"

*"Quand l'arbre se déséquilibre, l'Avatar entre en action !"*

```
Balance Factor > 1 : "Water Tribe!" 🌊 → Rotation droite
Balance Factor < -1 : "Fire Nation!" 🔥 → Rotation gauche
Cas LR : "Earthbending!" 🪨 → Double rotation (solide, stable)
Cas RL : "Airbending!" 💨 → Double rotation (souple, fluide)
```

#### 💡 Mnémotechnique des rotations :

```
"WATER flows RIGHT" → water_rotation = rotate_right (LL case)
"FIRE attacks LEFT" → fire_rotation = rotate_left (RR case)
"EARTH is STABLE, needs TWO moves" → LR = 2 rotations
"AIR is FLEXIBLE, needs TWO moves" → RL = 2 rotations
```

### 5.9 Applications pratiques

| Application | Pourquoi AVL |
|-------------|--------------|
| **In-memory DB index** | Recherche garantie O(log n) |
| **Kernel memory** | Allocation rapide et prévisible |
| **Real-time systems** | Pire cas = cas moyen |
| **Symbol tables** | Compilateurs, interpréteurs |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Comment l'éviter |
|---|-------|--------|------------------|
| 1 | bf sans signe | Mauvaise direction | Garder height(left) - height(right) |
| 2 | Oubli update_height | Hauteurs fausses | Appeler après CHAQUE modification |
| 3 | Cas LR traité comme LL | Arbre non équilibré | Vérifier bf du fils |
| 4 | Rotation inversée | Viole BST | Water=right, Fire=left |
| 5 | Pas de return | Ancien root gardé | return après rotation |

---

## 📝 SECTION 7 : QCM

### Question 1
**Un AVL avec balance factor = 2 à la racine a besoin de :**

A) Aucune action
B) Une rotation simple
C) Une ou deux rotations selon le cas
D) Trois rotations
E) Reconstruction complète
F) Suppression de la racine
G) Insertion d'équilibrage
H) C'est impossible
I) Dépend de la hauteur
J) C uniquement si bf du fils est correct

**Réponse : C**

### Question 2
**Quelle est la hauteur maximale d'un AVL avec 15 nœuds ?**

A) 3
B) 4
C) 5
D) 6
E) 7
F) log₂(15)
G) 15
H) 1.44 × log₂(15)
I) C ou D selon l'implémentation
J) Impossible à déterminer

**Réponse : B** (Un AVL de 15 nœuds a une hauteur max de 4)

### Question 3
**Dans le cas LR (Left-Right), la première rotation est :**

A) Rotation droite sur la racine
B) Rotation gauche sur la racine
C) Rotation droite sur le fils gauche
D) Rotation gauche sur le fils gauche
E) Rotation droite sur le fils droit
F) Rotation gauche sur le fils droit
G) Aucune rotation
H) Deux rotations simultanées
I) Dépend du balance factor
J) D puis A

**Réponse : J** (LR = rotation gauche sur fils gauche, puis rotation droite sur racine)

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.3.1-a — avatar_balance_tree |
| **Concept principal** | AVL Trees, Rotations |
| **Difficulté** | ★★★★★★☆☆☆☆ (6/10) |
| **Temps estimé** | 60 min |
| **XP Base** | 200 |
| **Bonus** | 🔥 Validation AVL (×3) / 💀 O(n) construction (×4) |
| **Langage** | Rust 2024 + C (c17) |
| **Points clés** | 4 rotations, balance factor, garantie O(log n) |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.3.1-a-avatar-balance-tree",
    "generated_at": "2025-01-11 14:45:00",

    "metadata": {
      "exercise_id": "1.3.1-a",
      "exercise_name": "avatar_balance_tree",
      "module": "1.3.1",
      "module_name": "AVL Trees",
      "concept": "a",
      "concept_name": "AVL Rotations",
      "type": "complet",
      "tier": 1,
      "tier_info": "Concept isolé",
      "phase": 1,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "rust",
      "language_alt": "c",
      "duration_minutes": 60,
      "xp_base": 200,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T3 O(log n)",
      "complexity_space": "S2 O(n)",
      "prerequisites": ["BST", "recursion", "rotations"],
      "domains": ["Struct", "Mem", "MD"],
      "tags": ["avl", "trees", "balance", "rotations"],
      "meme_reference": "Avatar: The Last Airbender - Four Elements"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "Balance must be maintained, or everything falls apart" - Avatar Aang*
