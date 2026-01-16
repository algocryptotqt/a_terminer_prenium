<thinking>
## Analyse du Concept
- Concept : Binary Search Tree (BST) - Opérations fondamentales
- Phase demandée : 1 (intermédiaire)
- Adapté ? OUI - Les BST sont une structure fondamentale qui combine récursion, pointeurs et logique conditionnelle

## Combo Base + Bonus
- Exercice de base : Implémenter un BST complet avec insert, search, delete, traversals
- Bonus : Morris Traversal (inorder sans récursion ni stack, O(1) espace) + Order Statistics (select/rank)
- Palier bonus : 🔥 Avancé (Morris) puis 💀 Expert (Order Stats avec augmentation)
- Progression logique ? OUI - Base = opérations standard, Bonus = optimisations avancées

## Prérequis & Difficulté
- Prérequis réels : Pointeurs, récursion, allocation mémoire, structures
- Difficulté estimée : 5/10 (base), 7/10 (bonus Morris), 8/10 (bonus Order Stats)
- Cohérent avec phase 1 ? OUI

## Aspect Fun/Culture
- Contexte choisi : "The Sorting Hat" de Harry Potter - L'arbre qui trie et place
- MEME mnémotechnique : "The Sorting Hat knows where you belong" - comme le BST qui sait toujours où placer un élément
- Pourquoi c'est fun : Le Choixpeau magique décide instantanément à gauche ou à droite (petit ou grand), exactement comme un BST

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : `if (key < node->key)` → `if (key <= node->key)` - place les doublons du mauvais côté
2. Mutant B (Safety) : Oubli de vérifier `root == NULL` dans search - crash sur arbre vide
3. Mutant C (Resource) : Oubli de `free()` dans delete - fuite mémoire sur chaque suppression
4. Mutant D (Logic) : Dans delete avec 2 enfants, prendre le max du sous-arbre gauche au lieu du min du droit
5. Mutant E (Return) : Retourner `node` au lieu de `node->left` ou `node->right` dans la récursion

## Verdict
VALIDE - L'exercice est complet, fun, et teste des compétences critiques
</thinking>

---

# Exercice 1.3.0-a : sorting_hat_tree

**Module :**
1.3.0 — Binary Search Trees

**Concept :**
a — Opérations fondamentales BST (insert, search, delete, traversals)

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
1 — Concept isolé

**Langage :**
Rust Edition 2024, C (c17)

**Prérequis :**
- Pointeurs et références
- Récursion
- Allocation dynamique (malloc/free, Box)
- Structures de données basiques

**Domaines :**
Struct, Mem, MD

**Durée estimée :**
45 min

**XP Base :**
150

**Complexité :**
T[4] O(h) pour toutes opérations × S[2] O(h) récursif / O(n) pour l'arbre

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- Rust : `src/lib.rs`, `Cargo.toml`
- C : `sorting_hat_tree.c`, `sorting_hat_tree.h`

**Fonctions autorisées :**
- C : `malloc`, `free`, `NULL`
- Rust : `Box::new`, `Option`, `std::cmp::Ordering`

**Fonctions interdites :**
- C : `calloc`, `realloc`, fonctions de la libc pour arbres
- Rust : `BTreeMap`, `BTreeSet`, collections externes

### 1.2 Consigne

**🎩 LE CHOIXPEAU DE POUDLARD — L'Arbre qui Trie les Sorciers**

*"Hmm, difficile. Très difficile. Je vois du courage, pas mal d'intelligence aussi..."*

Dans le monde de Harry Potter, le Choixpeau magique (Sorting Hat) place chaque élève dans une maison en quelques secondes. Comment fait-il ? Il utilise un **Binary Search Tree** mental !

Chaque élève a un "score magique". Le Choixpeau compare :
- Score < nœud actuel → "Tu iras à GAUCHE, vers Poufsouffle ou Serpentard"
- Score > nœud actuel → "Tu iras à DROITE, vers Gryffondor ou Serdaigle"
- Score trouvé → "Ah, je me souviens de toi !"

**Ta mission :**

Créer une structure `SortingHat<K, V>` (ou `sorting_hat_t` en C) qui implémente un Binary Search Tree capable de :

1. **Placer un élève** (`insert`) : Ajouter une clé-valeur dans l'arbre
2. **Retrouver un élève** (`search/get`) : Trouver une valeur par sa clé
3. **Renvoyer un élève** (`delete`) : Supprimer un nœud (les 3 cas)
4. **Lister par ordre** (`inorder`, `preorder`, `postorder`) : Parcourir l'arbre
5. **Min/Max** : Trouver le plus petit/grand élément
6. **Successor/Predecessor** : Trouver l'élément suivant/précédent

**Entrée :**
- `key` : La clé de l'élève (type générique ou `int`)
- `value` : Les données associées (nom de maison, etc.)

**Sortie :**
- `insert` : L'arbre modifié (ou `bool` pour succès)
- `search` : `Option<&V>` ou pointeur vers la valeur (NULL si non trouvé)
- `delete` : `true` si supprimé, `false` sinon

**Contraintes :**
- L'arbre doit gérer les clés en double (au choix : refuser ou placer à droite)
- Toutes les opérations doivent être en O(h) où h = hauteur
- Les traversées doivent être correctes (vérifiables par output)

**Exemples :**

| Opération | État avant | État après | Résultat |
|-----------|------------|------------|----------|
| `insert(5, "Harry")` | `∅` | `[5]` | `true` |
| `insert(3, "Ron")` | `[5]` | `[5←3]` | `true` |
| `insert(7, "Hermione")` | `[5←3]` | `[3←5→7]` | `true` |
| `search(3)` | `[3←5→7]` | `[3←5→7]` | `"Ron"` |
| `search(99)` | `[3←5→7]` | `[3←5→7]` | `None/NULL` |
| `delete(5)` | `[3←5→7]` | `[3←7]` | `true` |

### 1.3 Prototype

**Rust :**
```rust
pub struct SortingHat<K: Ord, V> {
    root: Option<Box<HatNode<K, V>>>,
}

struct HatNode<K: Ord, V> {
    key: K,
    value: V,
    left: Option<Box<HatNode<K, V>>>,
    right: Option<Box<HatNode<K, V>>>,
}

impl<K: Ord, V> SortingHat<K, V> {
    pub fn new() -> Self;
    pub fn insert(&mut self, key: K, value: V) -> bool;
    pub fn get(&self, key: &K) -> Option<&V>;
    pub fn get_mut(&mut self, key: &K) -> Option<&mut V>;
    pub fn contains(&self, key: &K) -> bool;
    pub fn remove(&mut self, key: &K) -> Option<V>;
    pub fn min(&self) -> Option<(&K, &V)>;
    pub fn max(&self) -> Option<(&K, &V)>;
    pub fn successor(&self, key: &K) -> Option<(&K, &V)>;
    pub fn predecessor(&self, key: &K) -> Option<(&K, &V)>;
    pub fn inorder(&self) -> Vec<(&K, &V)>;
    pub fn preorder(&self) -> Vec<(&K, &V)>;
    pub fn postorder(&self) -> Vec<(&K, &V)>;
    pub fn height(&self) -> usize;
    pub fn size(&self) -> usize;
    pub fn is_empty(&self) -> bool;
}
```

**C :**
```c
typedef struct s_hat_node {
    int             key;
    char            *value;
    struct s_hat_node   *left;
    struct s_hat_node   *right;
} t_hat_node;

typedef struct s_sorting_hat {
    t_hat_node  *root;
    size_t      size;
} t_sorting_hat;

// Création/Destruction
t_sorting_hat   *hat_new(void);
void            hat_free(t_sorting_hat *hat);

// Opérations principales
int             hat_insert(t_sorting_hat *hat, int key, char *value);
char            *hat_search(t_sorting_hat *hat, int key);
int             hat_delete(t_sorting_hat *hat, int key);

// Min/Max/Successor/Predecessor
t_hat_node      *hat_min(t_sorting_hat *hat);
t_hat_node      *hat_max(t_sorting_hat *hat);
t_hat_node      *hat_successor(t_sorting_hat *hat, int key);
t_hat_node      *hat_predecessor(t_sorting_hat *hat, int key);

// Traversées (stockent dans un tableau, retournent la taille)
size_t          hat_inorder(t_sorting_hat *hat, int *keys, char **values, size_t max);
size_t          hat_preorder(t_sorting_hat *hat, int *keys, char **values, size_t max);
size_t          hat_postorder(t_sorting_hat *hat, int *keys, char **values, size_t max);

// Utilitaires
size_t          hat_height(t_sorting_hat *hat);
size_t          hat_size(t_sorting_hat *hat);
int             hat_is_empty(t_sorting_hat *hat);
```

### 1.2.2 Énoncé Académique

Un **Binary Search Tree (BST)** est une structure de données arborescente où chaque nœud possède :
- Une clé `k` et une valeur associée `v`
- Un sous-arbre gauche contenant uniquement des clés < k
- Un sous-arbre droit contenant uniquement des clés > k

**Propriété BST :** Pour tout nœud N, toutes les clés du sous-arbre gauche sont strictement inférieures à N.key, et toutes les clés du sous-arbre droit sont strictement supérieures.

Les opérations requises :
1. **Insertion** : Descendre récursivement jusqu'à trouver la position correcte
2. **Recherche** : Comparer et descendre gauche/droite
3. **Suppression** : 3 cas (feuille, 1 enfant, 2 enfants)
4. **Traversées** : Inorder (gauche-racine-droite), Preorder (racine-gauche-droite), Postorder (gauche-droite-racine)

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Histoire du BST

Le Binary Search Tree a été inventé indépendamment par plusieurs chercheurs dans les années 1960. P.F. Windley, A.D. Booth, et T.N. Hibbard ont tous contribué à son développement.

**Fun fact :** Le terme "tree" en informatique vient de la représentation visuelle qui ressemble à un arbre... mais à l'envers ! La racine est en haut.

### 2.2 Le Problème du BST Déséquilibré

Un BST "normal" peut dégénérer en liste chaînée si on insère des éléments triés :
- Insert 1, 2, 3, 4, 5 → hauteur = 5 (pire cas)
- Insert 3, 1, 5, 2, 4 → hauteur = 3 (bien équilibré)

C'est pourquoi les arbres AVL et Red-Black ont été inventés (exercices suivants !).

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation | Cas concret |
|--------|-------------|-------------|
| **Database Engineer** | Index B-Tree | PostgreSQL utilise des variantes de BST pour ses index |
| **Game Developer** | Spatial partitioning | Arbres BSP pour le rendu 3D (Doom utilisait ça !) |
| **Compiler Engineer** | Symbol tables | gcc stocke les variables dans des arbres |
| **Financial Engineer** | Order book | Les ordres d'achat/vente sont dans un BST par prix |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
sorting_hat_tree.c  sorting_hat_tree.h  main.c  Cargo.toml  src/

$ gcc -Wall -Wextra -Werror -std=c17 sorting_hat_tree.c main.c -o test_c

$ ./test_c
=== Test BST Sorting Hat ===
Insert Harry(5): OK
Insert Ron(3): OK
Insert Hermione(7): OK
Insert Draco(2): OK
Insert Neville(6): OK

Inorder traversal: 2 3 5 6 7
Preorder traversal: 5 3 2 7 6
Search Ron(3): Found "Ron"
Search Voldemort(99): Not found

Min: 2 (Draco)
Max: 7 (Hermione)
Successor of 5: 6 (Neville)

Delete Harry(5): OK
Inorder after delete: 2 3 6 7

All tests passed!

$ cargo test
   Compiling sorting_hat v0.1.0
    Finished test [unoptimized + debuginfo]
     Running unittests src/lib.rs

running 12 tests
test tests::test_insert ... ok
test tests::test_search ... ok
test tests::test_delete_leaf ... ok
test tests::test_delete_one_child ... ok
test tests::test_delete_two_children ... ok
test tests::test_inorder ... ok
test tests::test_preorder ... ok
test tests::test_postorder ... ok
test tests::test_min_max ... ok
test tests::test_successor_predecessor ... ok
test tests::test_empty_tree ... ok
test tests::test_height ... ok

test result: ok. 12 passed; 0 failed
```

### 3.1 🔥 BONUS AVANCÉ : Morris Traversal (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n)

**Space Complexity attendue :**
O(1) — C'EST LE DÉFI !

**Domaines Bonus :**
`MD, Mem`

#### 3.1.1 Consigne Bonus

**🧙 LE SORTILÈGE DE MORRIS — Traverser sans Trace**

*"Un vrai sorcier ne laisse aucune trace de son passage..."*

Le Morris Traversal est un algorithme génial qui permet de parcourir un arbre **sans récursion** et **sans stack** — en utilisant les pointeurs `right` temporairement modifiés comme "fil d'Ariane".

**Ta mission :**

Implémenter `morris_inorder()` qui :
- Parcourt l'arbre en O(n) temps
- Utilise O(1) espace auxiliaire (pas de récursion, pas de stack)
- Restaure l'arbre à son état original après le parcours

**L'idée géniale de Morris :**
1. Si pas d'enfant gauche → visiter, aller à droite
2. Si enfant gauche → trouver le predecesseur inorder
   - Si son `right` est NULL → créer un lien temporaire vers nous
   - Si son `right` pointe vers nous → supprimer le lien, visiter, aller à droite

**Contraintes :**
┌─────────────────────────────────────────┐
│  Espace auxiliaire : O(1) strictement   │
│  Pas de récursion                       │
│  Pas de Vec/stack                       │
│  L'arbre doit être intact après         │
└─────────────────────────────────────────┘

#### 3.1.2 Prototype Bonus

```rust
impl<K: Ord, V> SortingHat<K, V> {
    /// Morris inorder traversal - O(n) time, O(1) space
    pub fn morris_inorder(&self) -> Vec<(&K, &V)>;
}
```

```c
// Morris traversal - modifie temporairement l'arbre puis le restaure
size_t hat_morris_inorder(t_sorting_hat *hat, int *keys, char **values, size_t max);
```

### 3.2 💀 BONUS EXPERT : Order Statistics (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×4

**Ta mission :**

Augmenter le BST avec un champ `size` dans chaque nœud pour supporter :
- `select(k)` : Trouver le k-ème plus petit élément en O(h)
- `rank(key)` : Trouver le rang d'une clé en O(h)

```rust
impl<K: Ord, V> SortingHat<K, V> {
    /// Get k-th smallest element (0-indexed)
    pub fn select(&self, k: usize) -> Option<(&K, &V)>;

    /// Get rank of key (number of elements smaller than key)
    pub fn rank(&self, key: &K) -> usize;
}
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| Test | Input | Expected | Points | Trap |
|------|-------|----------|--------|------|
| `test_new` | `SortingHat::new()` | `is_empty() == true` | 2 | |
| `test_insert_single` | `insert(5, "A")` | `size() == 1` | 3 | |
| `test_insert_left` | `insert(5,"A"), insert(3,"B")` | `root.left.key == 3` | 3 | |
| `test_insert_right` | `insert(5,"A"), insert(7,"B")` | `root.right.key == 7` | 3 | |
| `test_search_found` | `insert(5,"A"), get(&5)` | `Some("A")` | 5 | |
| `test_search_not_found` | `insert(5,"A"), get(&99)` | `None` | 5 | ⚠️ |
| `test_search_empty` | `get(&5)` sur arbre vide | `None` | 5 | ⚠️ |
| `test_delete_leaf` | Supprimer feuille | Arbre correct | 8 | |
| `test_delete_one_child` | Supprimer nœud à 1 enfant | Arbre correct | 8 | |
| `test_delete_two_children` | Supprimer nœud à 2 enfants | Arbre correct | 10 | ⚠️ |
| `test_delete_root` | Supprimer racine | Arbre correct | 8 | ⚠️ |
| `test_delete_not_found` | `delete(&99)` | `false` | 3 | |
| `test_inorder` | Arbre 5,3,7,2,4,6,8 | `[2,3,4,5,6,7,8]` | 8 | |
| `test_preorder` | Arbre 5,3,7 | `[5,3,7]` | 5 | |
| `test_postorder` | Arbre 5,3,7 | `[3,7,5]` | 5 | |
| `test_min` | Arbre 5,3,7,2 | `(2, ...)` | 5 | |
| `test_max` | Arbre 5,3,7,8 | `(8, ...)` | 5 | |
| `test_successor` | Succ de 5 dans 3,5,7 | `(7, ...)` | 5 | |
| `test_predecessor` | Pred de 5 dans 3,5,7 | `(3, ...)` | 5 | |
| `test_height` | Arbre 5,3,7 | `2` | 3 | |
| **TOTAL** | | | **100** | |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <string.h>
#include <assert.h>
#include "sorting_hat_tree.h"

void test_insert_and_search(void)
{
    t_sorting_hat *hat = hat_new();

    assert(hat_is_empty(hat) == 1);

    assert(hat_insert(hat, 5, "Harry") == 1);
    assert(hat_insert(hat, 3, "Ron") == 1);
    assert(hat_insert(hat, 7, "Hermione") == 1);

    assert(hat_size(hat) == 3);

    assert(strcmp(hat_search(hat, 5), "Harry") == 0);
    assert(strcmp(hat_search(hat, 3), "Ron") == 0);
    assert(hat_search(hat, 99) == NULL);

    hat_free(hat);
    printf("test_insert_and_search: OK\n");
}

void test_delete(void)
{
    t_sorting_hat *hat = hat_new();

    hat_insert(hat, 5, "A");
    hat_insert(hat, 3, "B");
    hat_insert(hat, 7, "C");
    hat_insert(hat, 2, "D");
    hat_insert(hat, 4, "E");

    // Delete leaf
    assert(hat_delete(hat, 2) == 1);
    assert(hat_search(hat, 2) == NULL);

    // Delete node with one child
    assert(hat_delete(hat, 3) == 1);
    assert(hat_search(hat, 3) == NULL);
    assert(hat_search(hat, 4) != NULL); // Child should still exist

    // Delete node with two children
    hat_insert(hat, 3, "B");
    hat_insert(hat, 2, "D");
    assert(hat_delete(hat, 5) == 1);

    hat_free(hat);
    printf("test_delete: OK\n");
}

void test_traversals(void)
{
    t_sorting_hat *hat = hat_new();
    int keys[10];
    char *values[10];

    hat_insert(hat, 5, "E");
    hat_insert(hat, 3, "C");
    hat_insert(hat, 7, "G");
    hat_insert(hat, 2, "B");
    hat_insert(hat, 4, "D");

    size_t n = hat_inorder(hat, keys, values, 10);
    assert(n == 5);
    assert(keys[0] == 2 && keys[1] == 3 && keys[2] == 4);
    assert(keys[3] == 5 && keys[4] == 7);

    n = hat_preorder(hat, keys, values, 10);
    assert(keys[0] == 5 && keys[1] == 3 && keys[2] == 2);

    hat_free(hat);
    printf("test_traversals: OK\n");
}

int main(void)
{
    printf("=== Tests Sorting Hat BST ===\n");
    test_insert_and_search();
    test_delete();
    test_traversals();
    printf("\nAll tests passed!\n");
    return 0;
}
```

### 4.3 Solution de référence

**Rust :**
```rust
use std::cmp::Ordering;

pub struct SortingHat<K: Ord, V> {
    root: Option<Box<HatNode<K, V>>>,
    size: usize,
}

struct HatNode<K: Ord, V> {
    key: K,
    value: V,
    left: Option<Box<HatNode<K, V>>>,
    right: Option<Box<HatNode<K, V>>>,
}

impl<K: Ord, V> HatNode<K, V> {
    fn new(key: K, value: V) -> Self {
        HatNode {
            key,
            value,
            left: None,
            right: None,
        }
    }
}

impl<K: Ord, V> SortingHat<K, V> {
    pub fn new() -> Self {
        SortingHat { root: None, size: 0 }
    }

    pub fn insert(&mut self, key: K, value: V) -> bool {
        fn insert_rec<K: Ord, V>(
            node: &mut Option<Box<HatNode<K, V>>>,
            key: K,
            value: V,
        ) -> bool {
            match node {
                None => {
                    *node = Some(Box::new(HatNode::new(key, value)));
                    true
                }
                Some(n) => match key.cmp(&n.key) {
                    Ordering::Less => insert_rec(&mut n.left, key, value),
                    Ordering::Greater => insert_rec(&mut n.right, key, value),
                    Ordering::Equal => {
                        n.value = value; // Update existing
                        false
                    }
                },
            }
        }

        if insert_rec(&mut self.root, key, value) {
            self.size += 1;
            true
        } else {
            false
        }
    }

    pub fn get(&self, key: &K) -> Option<&V> {
        fn search_rec<K: Ord, V>(node: &Option<Box<HatNode<K, V>>>, key: &K) -> Option<&V> {
            match node {
                None => None,
                Some(n) => match key.cmp(&n.key) {
                    Ordering::Less => search_rec(&n.left, key),
                    Ordering::Greater => search_rec(&n.right, key),
                    Ordering::Equal => Some(&n.value),
                },
            }
        }
        search_rec(&self.root, key)
    }

    pub fn contains(&self, key: &K) -> bool {
        self.get(key).is_some()
    }

    pub fn remove(&mut self, key: &K) -> Option<V> {
        fn find_min<K: Ord, V>(node: &mut Box<HatNode<K, V>>) -> Box<HatNode<K, V>> {
            if node.left.is_some() {
                find_min(node.left.as_mut().unwrap())
            } else {
                let mut min = node.left.take().unwrap_or_else(|| {
                    Box::new(HatNode::new(
                        unsafe { std::ptr::read(&node.key) },
                        unsafe { std::ptr::read(&node.value) },
                    ))
                });
                min
            }
        }

        fn remove_rec<K: Ord, V>(
            node: &mut Option<Box<HatNode<K, V>>>,
            key: &K,
        ) -> Option<V> {
            let n = node.as_mut()?;

            match key.cmp(&n.key) {
                Ordering::Less => remove_rec(&mut n.left, key),
                Ordering::Greater => remove_rec(&mut n.right, key),
                Ordering::Equal => {
                    let old_node = node.take().unwrap();
                    let value = old_node.value;

                    match (old_node.left, old_node.right) {
                        (None, None) => {}
                        (Some(left), None) => *node = Some(left),
                        (None, Some(right)) => *node = Some(right),
                        (Some(left), Some(mut right)) => {
                            // Find inorder successor (min of right subtree)
                            fn extract_min<K: Ord, V>(
                                node: &mut Option<Box<HatNode<K, V>>>
                            ) -> Box<HatNode<K, V>> {
                                if node.as_ref().unwrap().left.is_some() {
                                    extract_min(&mut node.as_mut().unwrap().left)
                                } else {
                                    let min = node.take().unwrap();
                                    *node = min.right.clone();
                                    min
                                }
                            }

                            let mut successor = extract_min(&mut Some(right));
                            successor.left = Some(left);
                            successor.right = node.take().and_then(|n| n.right);
                            *node = Some(successor);
                        }
                    }

                    Some(value)
                }
            }
        }

        if let Some(v) = remove_rec(&mut self.root, key) {
            self.size -= 1;
            Some(v)
        } else {
            None
        }
    }

    pub fn min(&self) -> Option<(&K, &V)> {
        fn min_rec<K: Ord, V>(node: &Option<Box<HatNode<K, V>>>) -> Option<(&K, &V)> {
            node.as_ref().map(|n| {
                if n.left.is_some() {
                    min_rec(&n.left).unwrap()
                } else {
                    (&n.key, &n.value)
                }
            })
        }
        min_rec(&self.root)
    }

    pub fn max(&self) -> Option<(&K, &V)> {
        fn max_rec<K: Ord, V>(node: &Option<Box<HatNode<K, V>>>) -> Option<(&K, &V)> {
            node.as_ref().map(|n| {
                if n.right.is_some() {
                    max_rec(&n.right).unwrap()
                } else {
                    (&n.key, &n.value)
                }
            })
        }
        max_rec(&self.root)
    }

    pub fn inorder(&self) -> Vec<(&K, &V)> {
        fn inorder_rec<'a, K: Ord, V>(
            node: &'a Option<Box<HatNode<K, V>>>,
            result: &mut Vec<(&'a K, &'a V)>,
        ) {
            if let Some(n) = node {
                inorder_rec(&n.left, result);
                result.push((&n.key, &n.value));
                inorder_rec(&n.right, result);
            }
        }
        let mut result = Vec::new();
        inorder_rec(&self.root, &mut result);
        result
    }

    pub fn preorder(&self) -> Vec<(&K, &V)> {
        fn preorder_rec<'a, K: Ord, V>(
            node: &'a Option<Box<HatNode<K, V>>>,
            result: &mut Vec<(&'a K, &'a V)>,
        ) {
            if let Some(n) = node {
                result.push((&n.key, &n.value));
                preorder_rec(&n.left, result);
                preorder_rec(&n.right, result);
            }
        }
        let mut result = Vec::new();
        preorder_rec(&self.root, &mut result);
        result
    }

    pub fn postorder(&self) -> Vec<(&K, &V)> {
        fn postorder_rec<'a, K: Ord, V>(
            node: &'a Option<Box<HatNode<K, V>>>,
            result: &mut Vec<(&'a K, &'a V)>,
        ) {
            if let Some(n) = node {
                postorder_rec(&n.left, result);
                postorder_rec(&n.right, result);
                result.push((&n.key, &n.value));
            }
        }
        let mut result = Vec::new();
        postorder_rec(&self.root, &mut result);
        result
    }

    pub fn height(&self) -> usize {
        fn height_rec<K: Ord, V>(node: &Option<Box<HatNode<K, V>>>) -> usize {
            match node {
                None => 0,
                Some(n) => 1 + height_rec(&n.left).max(height_rec(&n.right)),
            }
        }
        height_rec(&self.root)
    }

    pub fn size(&self) -> usize {
        self.size
    }

    pub fn is_empty(&self) -> bool {
        self.size == 0
    }
}

impl<K: Ord, V> Default for SortingHat<K, V> {
    fn default() -> Self {
        Self::new()
    }
}
```

### 4.4 Solutions alternatives acceptées

```rust
// Alternative 1: Suppression avec predecessor au lieu de successor
// Valide car maintient la propriété BST

// Alternative 2: Utilisation de RefCell pour mutabilité intérieure
// Valide si correctement implémenté
```

### 4.5 Solutions refusées (avec explications)

```rust
// REFUSÉ: Utilisation de BTreeMap
use std::collections::BTreeMap;
// Raison: L'exercice demande d'implémenter le BST, pas de wrapper

// REFUSÉ: Insertion qui ne maintient pas la propriété BST
fn bad_insert(&mut self, key: K, value: V) {
    // Toujours insérer à gauche
    // Raison: Viole la propriété fondamentale du BST
}
```

### 4.6 Solution bonus de référence (Morris Traversal)

```rust
impl<K: Ord + Clone, V: Clone> SortingHat<K, V> {
    pub fn morris_inorder(&self) -> Vec<(K, V)> {
        let mut result = Vec::new();

        // Clone l'arbre car Morris modifie temporairement les pointeurs
        let mut current = self.root.clone();

        while let Some(mut node) = current {
            if node.left.is_none() {
                // Pas d'enfant gauche: visiter et aller à droite
                result.push((node.key.clone(), node.value.clone()));
                current = node.right;
            } else {
                // Trouver le predecesseur inorder
                let mut pre = node.left.clone().unwrap();
                while pre.right.is_some()
                    && pre.right.as_ref().map(|r| &r.key) != Some(&node.key)
                {
                    pre = pre.right.unwrap();
                }

                if pre.right.is_none() {
                    // Créer le lien temporaire
                    pre.right = Some(node.clone());
                    current = node.left;
                } else {
                    // Lien existe: le supprimer, visiter, aller à droite
                    pre.right = None;
                    result.push((node.key.clone(), node.value.clone()));
                    current = node.right;
                }
            }
        }

        result
    }
}
```

### 4.9 spec.json

```json
{
  "name": "sorting_hat_tree",
  "language": "rust",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isolé - BST fondamental",
  "tags": ["bst", "trees", "recursion", "phase1"],
  "passing_score": 70,

  "function": {
    "name": "SortingHat",
    "prototype": "pub struct SortingHat<K: Ord, V>",
    "return_type": "struct",
    "parameters": []
  },

  "driver": {
    "reference": "pub struct SortingHat<K: Ord, V> { root: Option<Box<HatNode<K, V>>>, size: usize } impl<K: Ord, V> SortingHat<K, V> { pub fn new() -> Self { SortingHat { root: None, size: 0 } } pub fn insert(&mut self, key: K, value: V) -> bool { fn insert_rec<K: Ord, V>(node: &mut Option<Box<HatNode<K, V>>>, key: K, value: V) -> bool { match node { None => { *node = Some(Box::new(HatNode { key, value, left: None, right: None })); true } Some(n) => match key.cmp(&n.key) { std::cmp::Ordering::Less => insert_rec(&mut n.left, key, value), std::cmp::Ordering::Greater => insert_rec(&mut n.right, key, value), std::cmp::Ordering::Equal => { n.value = value; false } } } } if insert_rec(&mut self.root, key, value) { self.size += 1; true } else { false } } pub fn get(&self, key: &K) -> Option<&V> { fn search_rec<K: Ord, V>(node: &Option<Box<HatNode<K, V>>>, key: &K) -> Option<&V> { match node { None => None, Some(n) => match key.cmp(&n.key) { std::cmp::Ordering::Less => search_rec(&n.left, key), std::cmp::Ordering::Greater => search_rec(&n.right, key), std::cmp::Ordering::Equal => Some(&n.value) } } } search_rec(&self.root, key) } pub fn is_empty(&self) -> bool { self.size == 0 } pub fn size(&self) -> usize { self.size } }",

    "edge_cases": [
      {
        "name": "empty_tree_search",
        "args": ["&5"],
        "expected": "None",
        "is_trap": true,
        "trap_explanation": "Recherche sur arbre vide doit retourner None sans crash"
      },
      {
        "name": "insert_single",
        "args": ["5", "\"Harry\""],
        "expected": "true",
        "is_trap": false
      },
      {
        "name": "insert_duplicate",
        "args": ["5", "\"Ron\""],
        "expected": "false",
        "is_trap": true,
        "trap_explanation": "Insertion de clé existante doit update la valeur et retourner false"
      },
      {
        "name": "delete_nonexistent",
        "args": ["&99"],
        "expected": "None",
        "is_trap": true,
        "trap_explanation": "Suppression de clé inexistante doit retourner None"
      },
      {
        "name": "delete_root_two_children",
        "args": ["&5"],
        "expected": "Some(...)",
        "is_trap": true,
        "trap_explanation": "Suppression de racine avec 2 enfants - cas complexe"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 1000,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {
            "min": -1000,
            "max": 1000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["Box::new", "Option", "std::cmp::Ordering"],
    "forbidden_functions": ["BTreeMap", "BTreeSet", "HashMap"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```rust
/* Mutant A (Boundary) : Utilise <= au lieu de < pour la comparaison */
fn insert_mutant_a(&mut self, key: K, value: V) -> bool {
    fn insert_rec<K: Ord, V>(node: &mut Option<Box<HatNode<K, V>>>, key: K, value: V) -> bool {
        match node {
            None => { *node = Some(Box::new(HatNode::new(key, value))); true }
            Some(n) => match key.cmp(&n.key) {
                // BUG: <= au lieu de <
                Ordering::Less | Ordering::Equal => insert_rec(&mut n.left, key, value),
                Ordering::Greater => insert_rec(&mut n.right, key, value),
            }
        }
    }
    insert_rec(&mut self.root, key, value)
}
// Pourquoi c'est faux : Les doublons vont à gauche, la recherche ne les trouvera pas
// Ce qui était pensé : "< et <= c'est pareil" — NON !

/* Mutant B (Safety) : Pas de vérification de None dans search */
fn get_mutant_b(&self, key: &K) -> Option<&V> {
    // BUG: Assume root existe toujours
    let n = self.root.as_ref().unwrap();
    match key.cmp(&n.key) {
        Ordering::Less => self.get(key),
        Ordering::Greater => self.get(key),
        Ordering::Equal => Some(&n.value),
    }
}
// Pourquoi c'est faux : Panic sur arbre vide
// Ce qui était pensé : "Le root existe toujours" — NON !

/* Mutant C (Resource) : Fuite mémoire dans delete */
fn remove_mutant_c(&mut self, key: &K) -> Option<V> {
    fn remove_rec<K: Ord, V>(node: &mut Option<Box<HatNode<K, V>>>, key: &K) -> Option<V> {
        let n = node.as_mut()?;
        match key.cmp(&n.key) {
            Ordering::Less => remove_rec(&mut n.left, key),
            Ordering::Greater => remove_rec(&mut n.right, key),
            Ordering::Equal => {
                // BUG: On prend la valeur mais on ne libère pas le nœud
                let value = unsafe { std::ptr::read(&n.value) };
                // Oubli de *node = None; ou de gérer les enfants
                Some(value)
            }
        }
    }
    remove_rec(&mut self.root, key)
}
// Pourquoi c'est faux : Le nœud reste dans l'arbre, fuite mémoire
// Ce qui était pensé : "J'ai pris la valeur donc c'est bon" — NON !

/* Mutant D (Logic) : Mauvais remplacement dans delete 2 enfants */
fn remove_mutant_d(&mut self, key: &K) -> Option<V> {
    // Dans le cas 2 enfants:
    // BUG: Prend le MAX du sous-arbre GAUCHE au lieu du MIN du DROIT
    // Ou inversement selon l'implémentation
    fn find_max<K: Ord, V>(node: &Box<HatNode<K, V>>) -> &K {
        if node.right.is_some() {
            find_max(node.right.as_ref().unwrap())
        } else {
            &node.key
        }
    }
    // ... utilise find_max sur le sous-arbre DROIT
    // Ce qui donne un nœud qui n'est pas le successor/predecessor correct
    None
}
// Pourquoi c'est faux : L'arbre n'est plus un BST valide après suppression
// Ce qui était pensé : "Max ou min c'est pareil" — NON !

/* Mutant E (Return) : Retourne toujours true dans insert */
fn insert_mutant_e(&mut self, key: K, value: V) -> bool {
    fn insert_rec<K: Ord, V>(node: &mut Option<Box<HatNode<K, V>>>, key: K, value: V) {
        match node {
            None => { *node = Some(Box::new(HatNode::new(key, value))); }
            Some(n) => match key.cmp(&n.key) {
                Ordering::Less => insert_rec(&mut n.left, key, value),
                Ordering::Greater => insert_rec(&mut n.right, key, value),
                Ordering::Equal => { n.value = value; }
            }
        }
    }
    insert_rec(&mut self.root, key, value);
    self.size += 1;  // BUG: Incrémente même si clé existait
    true  // BUG: Retourne toujours true
}
// Pourquoi c'est faux : size sera incorrecte après updates de clés existantes
// Ce qui était pensé : "Insert réussit toujours" — NON, update != insert
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

1. **Structure récursive** : Un arbre est défini récursivement (nœud + sous-arbres)
2. **Propriété d'invariant** : Maintenir la propriété BST à chaque opération
3. **Les 3 cas de suppression** : Cas fondamental en structures de données
4. **Traversées** : Comprendre l'ordre des visites
5. **Gestion de la mémoire** : Allocation/libération correcte des nœuds

### 5.2 LDA — Traduction Littérale en Français (MAJUSCULES)

```
STRUCTURE HatNode QUI CONTIENT :
    - key QUI EST UNE CLÉ COMPARABLE
    - value QUI EST UNE VALEUR ASSOCIÉE
    - left QUI EST UN POINTEUR VERS UN HatNode (PEUT ÊTRE NUL)
    - right QUI EST UN POINTEUR VERS UN HatNode (PEUT ÊTRE NUL)
FIN STRUCTURE

FONCTION insert QUI PREND key ET value ET RETOURNE UN BOOLÉEN
DÉBUT FONCTION
    SI root EST NUL ALORS
        AFFECTER UN NOUVEAU NŒUD AVEC key ET value À root
        RETOURNER VRAI
    FIN SI

    SI key EST INFÉRIEUR À root.key ALORS
        RETOURNER insert SUR LE SOUS-ARBRE GAUCHE AVEC key ET value
    SINON SI key EST SUPÉRIEUR À root.key ALORS
        RETOURNER insert SUR LE SOUS-ARBRE DROIT AVEC key ET value
    SINON
        AFFECTER value À root.value
        RETOURNER FAUX
    FIN SI
FIN FONCTION

FONCTION search QUI PREND key ET RETOURNE UNE OPTION DE VALEUR
DÉBUT FONCTION
    SI root EST NUL ALORS
        RETOURNER NUL
    FIN SI

    SI key EST ÉGAL À root.key ALORS
        RETOURNER root.value
    SINON SI key EST INFÉRIEUR À root.key ALORS
        RETOURNER search SUR LE SOUS-ARBRE GAUCHE AVEC key
    SINON
        RETOURNER search SUR LE SOUS-ARBRE DROIT AVEC key
    FIN SI
FIN FONCTION

FONCTION delete QUI PREND key ET RETOURNE UNE OPTION DE VALEUR
DÉBUT FONCTION
    TROUVER LE NŒUD À SUPPRIMER

    CAS 1 : SI LE NŒUD EST UNE FEUILLE
        SUPPRIMER LE NŒUD
    FIN CAS

    CAS 2 : SI LE NŒUD A UN SEUL ENFANT
        REMPLACER LE NŒUD PAR SON ENFANT
    FIN CAS

    CAS 3 : SI LE NŒUD A DEUX ENFANTS
        TROUVER LE SUCCESSEUR INORDER (MIN DU SOUS-ARBRE DROIT)
        COPIER LA CLÉ ET VALEUR DU SUCCESSEUR
        SUPPRIMER LE SUCCESSEUR (QUI EST CAS 1 OU 2)
    FIN CAS
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : BST Insert
---
1. SI arbre vide :
   a. Créer nouveau nœud avec (key, value)
   b. RETOURNER succès

2. COMPARER key avec nœud courant :

   a. CAS key < nœud.key :
      - Descendre dans sous-arbre GAUCHE
      - APPELER insert récursivement

   b. CAS key > nœud.key :
      - Descendre dans sous-arbre DROIT
      - APPELER insert récursivement

   c. CAS key == nœud.key :
      - METTRE À JOUR la valeur
      - RETOURNER "clé existait déjà"

3. FIN
```

### 5.2.3 Logique de Garde (Fail Fast)

```
FONCTION : hat_search (key)
---
INIT résultat = NULL

1. VÉRIFIER si arbre est NULL :
   |
   |-- RETOURNER NULL immédiatement

2. VÉRIFIER si root est NULL :
   |
   |-- RETOURNER NULL (arbre vide)

3. COMPARER key avec root.key :
   |
   |-- SI égal : RETOURNER &root.value
   |
   |-- SI inférieur : RÉCURSION sur left
   |
   |-- SI supérieur : RÉCURSION sur right

4. RETOURNER résultat de la récursion
```

### 5.3 Visualisation ASCII

```
Insertion de 5, 3, 7, 2, 4, 6, 8 :

Étape 1: insert(5)        Étape 2: insert(3)      Étape 3: insert(7)
     [5]                       [5]                      [5]
                              /                        /   \
                            [3]                      [3]   [7]

Étape 4-7: insert(2,4,6,8)
              [5]
            /     \
          [3]     [7]
         /   \   /   \
       [2]  [4][6]  [8]

Suppression de [5] (2 enfants):
1. Trouver successeur inorder = [6]
2. Copier (6, val6) à la place de (5, val5)
3. Supprimer l'ancien [6] (cas feuille)

              [6]
            /     \
          [3]     [7]
         /   \       \
       [2]  [4]     [8]
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **Arbre vide** | Accès à root sans vérification | Toujours vérifier `root == NULL` |
| **Delete root** | Cas particulier souvent oublié | Traiter le root comme un cas spécial |
| **Delete 2 enfants** | Mauvais choix successor/predecessor | Toujours prendre min(right) ou max(left) |
| **Comparaison** | < vs <= confusion | Strict inequality pour BST standard |
| **Fuite mémoire** | Oubli de free() après delete | Libérer le nœud supprimé |

### 5.5 Cours Complet

#### Qu'est-ce qu'un Binary Search Tree ?

Un **Binary Search Tree (BST)** est une structure de données arborescente qui organise les données de manière à permettre des recherches, insertions et suppressions efficaces.

**Propriété fondamentale :**
> Pour tout nœud N :
> - Tous les nœuds du sous-arbre GAUCHE ont des clés < N.key
> - Tous les nœuds du sous-arbre DROIT ont des clés > N.key

Cette propriété est **récursive** : chaque sous-arbre est lui-même un BST valide.

#### Complexité des opérations

| Opération | Meilleur cas | Cas moyen | Pire cas |
|-----------|--------------|-----------|----------|
| Search | O(1) | O(log n) | O(n) |
| Insert | O(1) | O(log n) | O(n) |
| Delete | O(1) | O(log n) | O(n) |
| Min/Max | O(1) | O(log n) | O(n) |

Le **pire cas O(n)** arrive quand l'arbre dégénère en liste chaînée (insertions triées).

#### Les 3 cas de suppression

**Cas 1 : Nœud feuille (pas d'enfant)**
```
Avant:      Après:
  [5]         [5]
 /   \       /
[3]  [7]   [3]
      \
      [8]  ← supprimer
```
→ Simple : on supprime le nœud

**Cas 2 : Nœud avec 1 enfant**
```
Avant:      Après:
  [5]         [5]
 /   \       /   \
[3]  [7]   [3]  [8]
      \
      [8]
[7] ← supprimer
```
→ On remplace le nœud par son unique enfant

**Cas 3 : Nœud avec 2 enfants**
```
Avant:              Après:
     [5] ← supprimer     [6]
    /   \               /   \
  [3]   [7]           [3]   [7]
       /   \                   \
     [6]   [8]               [8]
```
→ On remplace par le successeur inorder (min du sous-arbre droit)

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ if (key < node->key) {                                          │
│     insert(node->left, key); }                                  │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ if (key < node->key)                                            │
│ {                                                               │
│     insert(node->left, key);                                    │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Lisibilité : Chaque accolade sur sa ligne                     │
│ • Débogage : Plus facile de mettre des breakpoints              │
│ • Cohérence : Style uniformisé dans tout le projet              │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario : search(4) dans l'arbre [5←3→7, 2←3→4]**

```
┌───────┬─────────────────────────────────────┬──────────┬─────────────────────┐
│ Étape │ Instruction                         │ Nœud     │ Explication         │
├───────┼─────────────────────────────────────┼──────────┼─────────────────────┤
│   1   │ Comparer 4 avec root.key (5)        │ [5]      │ 4 < 5 → aller LEFT  │
├───────┼─────────────────────────────────────┼──────────┼─────────────────────┤
│   2   │ Comparer 4 avec node.key (3)        │ [3]      │ 4 > 3 → aller RIGHT │
├───────┼─────────────────────────────────────┼──────────┼─────────────────────┤
│   3   │ Comparer 4 avec node.key (4)        │ [4]      │ 4 == 4 → TROUVÉ !   │
├───────┼─────────────────────────────────────┼──────────┼─────────────────────┤
│   4   │ RETOURNER &node.value               │ [4]      │ Résultat: valeur    │
└───────┴─────────────────────────────────────┴──────────┴─────────────────────┘
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🎩 MEME : "The Sorting Hat decides in O(log n)"

![Sorting Hat](sorting_hat_meme.jpg)

*"Hmm, difficile... Je compare ta clé à la mienne... Plus petit ? GAUCHE ! Plus grand ? DROITE !"*

Le Choixpeau de Poudlard et le BST fonctionnent pareil :
- **Décision binaire** : gauche ou droite, pas de milieu
- **Récursif** : chaque sous-arbre est un mini-Choixpeau
- **Déterministe** : même clé = même chemin toujours

```rust
fn sorting_hat_decision(student_score: i32, current: i32) -> Direction {
    if student_score < current {
        Direction::Slytherin  // Gauche
    } else {
        Direction::Gryffindor // Droite
    }
}
```

#### 🌳 MEME : "Tree-fiddy" — Le coût d'un arbre déséquilibré

Quand tu insères 1, 2, 3, 4, 5 dans un BST :
```
[1]
  \
  [2]
    \
    [3]
      \
      ... 💀
```

*"I need about tree-fiddy"* — et ton BST a besoin d'un équilibrage !
C'est pourquoi on utilise AVL/Red-Black (prochains exercices).

### 5.9 Applications pratiques

| Application | Utilisation du BST |
|-------------|-------------------|
| **Base de données** | Index pour recherche rapide |
| **Compilateur** | Table des symboles |
| **Système de fichiers** | Organisation des répertoires |
| **Auto-complétion** | Suggestions triées |
| **Trading** | Order book par prix |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Impact | Comment l'éviter |
|---|-------|--------|------------------|
| 1 | Arbre vide non géré | Segfault | `if (root == NULL)` en premier |
| 2 | < vs <= dans insert | Propriété BST violée | Toujours utiliser < strict |
| 3 | Delete root oublié | Arbre corrompu | Cas spécial pour root |
| 4 | Mauvais successor | BST invalide | min(right) ou max(left) |
| 5 | Fuite mémoire | Memory leak | free() le nœud supprimé |
| 6 | Récursion infinie | Stack overflow | Vérifier les cas de base |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quelle est la propriété fondamentale d'un BST ?**

A) Tous les nœuds ont exactement 2 enfants
B) La hauteur est toujours O(log n)
C) Pour tout nœud, gauche < nœud < droite
D) Les clés sont uniques et triées en préordre
E) Chaque niveau est complètement rempli
F) La racine contient la plus grande clé
G) Les feuilles sont toutes au même niveau
H) Le parcours inorder donne les clés triées
I) C et H sont vraies
J) A et E sont vraies

**Réponse : I**

### Question 2
**Lors de la suppression d'un nœud avec 2 enfants, on le remplace par :**

A) Son parent
B) Son enfant gauche
C) Son enfant droit
D) Le minimum du sous-arbre droit
E) Le maximum du sous-arbre droit
F) Le minimum du sous-arbre gauche
G) D ou F (successeur ou prédécesseur inorder)
H) N'importe quel descendant
I) La moyenne des deux enfants
J) On ne peut pas supprimer un tel nœud

**Réponse : G**

### Question 3
**Quelle traversée donne les clés d'un BST en ordre croissant ?**

A) Preorder
B) Postorder
C) Inorder
D) Level-order
E) Reverse inorder
F) Preorder inversé
G) DFS
H) BFS
I) Toutes les traversées
J) Aucune traversée

**Réponse : C**

### Question 4
**Quelle est la complexité de search dans le pire cas ?**

A) O(1)
B) O(log n)
C) O(n)
D) O(n log n)
E) O(n²)
F) O(h) où h = hauteur
G) C et F sont équivalentes dans le pire cas
H) Toujours O(log n)
I) Dépend de l'implémentation
J) O(2^n)

**Réponse : G**

### Question 5
**Le Morris Traversal a quelle complexité spatiale ?**

A) O(n)
B) O(log n)
C) O(h)
D) O(1)
E) O(n log n)
F) Dépend de l'arbre
G) O(n) pour le résultat + O(1) auxiliaire
H) Impossible sans récursion
I) O(n²)
J) D mais modifie temporairement l'arbre

**Réponse : J**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 1.3.0-a — sorting_hat_tree |
| **Concept principal** | Binary Search Tree |
| **Difficulté** | ★★★★★☆☆☆☆☆ (5/10) |
| **Temps estimé** | 45 min |
| **XP Base** | 150 |
| **Bonus Morris** | 🔥 Avancé (×3 XP) |
| **Bonus Order Stats** | 💀 Expert (×4 XP) |
| **Langage** | Rust 2024 + C (c17) |
| **Points clés** | Insert, Search, Delete (3 cas), Traversals |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "1.3.0-a-sorting-hat-tree",
    "generated_at": "2025-01-11 14:30:00",

    "metadata": {
      "exercise_id": "1.3.0-a",
      "exercise_name": "sorting_hat_tree",
      "module": "1.3.0",
      "module_name": "Binary Search Trees",
      "concept": "a",
      "concept_name": "BST Operations",
      "type": "complet",
      "tier": 1,
      "tier_info": "Concept isolé",
      "phase": 1,
      "difficulty": 5,
      "difficulty_stars": "★★★★★☆☆☆☆☆",
      "language": "rust",
      "language_alt": "c",
      "duration_minutes": 45,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T4 O(h)",
      "complexity_space": "S2 O(h)",
      "prerequisites": ["pointeurs", "recursion", "allocation"],
      "domains": ["Struct", "Mem", "MD"],
      "domains_bonus": [],
      "tags": ["bst", "trees", "recursion", "data-structures"],
      "meme_reference": "Sorting Hat decides in O(log n)"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.rs": "/* Section 4.3 */",
      "references/ref_solution_bonus.rs": "/* Section 4.6 */",
      "mutants/mutant_a_boundary.rs": "/* Section 4.10 */",
      "mutants/mutant_b_safety.rs": "/* Section 4.10 */",
      "mutants/mutant_c_resource.rs": "/* Section 4.10 */",
      "mutants/mutant_d_logic.rs": "/* Section 4.10 */",
      "mutants/mutant_e_return.rs": "/* Section 4.10 */",
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
        "mutants/mutant_c_resource.rs",
        "mutants/mutant_d_logic.rs",
        "mutants/mutant_e_return.rs"
      ]
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "The Sorting Hat knows where you belong, and so does the BST"*
