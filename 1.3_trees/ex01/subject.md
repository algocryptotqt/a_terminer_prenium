# Exercise 01: AVL Trees

## Concepts Covered
- **1.3.7.g-h** Update heights, BST property preserved
- **1.3.8.g-j** RL case, single/double rotation, propagation
- **1.3.9.f** Delete implementation

## Objective

Implement a self-balancing AVL tree with all rotations and operations.

## Requirements

### Rust Implementation

```rust
pub mod avl {
    /// AVL Tree node
    struct AvlNode<K: Ord, V> {
        key: K,
        value: V,
        height: i32,
        left: Option<Box<AvlNode<K, V>>>,
        right: Option<Box<AvlNode<K, V>>>,
    }

    /// AVL Tree (self-balancing BST)
    pub struct AvlTree<K: Ord, V> {
        root: Option<Box<AvlNode<K, V>>>,
        len: usize,
    }

    impl<K: Ord, V> AvlTree<K, V> {
        pub fn new() -> Self;

        /// Insert with automatic rebalancing
        pub fn insert(&mut self, key: K, value: V) -> Option<V>;

        /// Remove with automatic rebalancing
        pub fn remove(&mut self, key: &K) -> Option<V>;

        /// Get value by key
        pub fn get(&self, key: &K) -> Option<&V>;

        /// Check if key exists
        pub fn contains(&self, key: &K) -> bool;

        /// Number of elements
        pub fn len(&self) -> usize;

        /// Height of tree (should be O(log n))
        pub fn height(&self) -> i32;

        /// Check AVL property (|left.height - right.height| <= 1)
        pub fn is_balanced(&self) -> bool;

        /// Inorder traversal
        pub fn inorder(&self) -> Vec<(&K, &V)>;

        /// Minimum key
        pub fn min(&self) -> Option<&K>;

        /// Maximum key
        pub fn max(&self) -> Option<&K>;
    }

    // Internal rotation functions (for understanding)

    /// Right rotation (LL case)
    fn rotate_right<K: Ord, V>(node: Box<AvlNode<K, V>>) -> Box<AvlNode<K, V>>;

    /// Left rotation (RR case)
    fn rotate_left<K: Ord, V>(node: Box<AvlNode<K, V>>) -> Box<AvlNode<K, V>>;

    /// Left-Right rotation (LR case)
    fn rotate_left_right<K: Ord, V>(node: Box<AvlNode<K, V>>) -> Box<AvlNode<K, V>>;

    /// Right-Left rotation (RL case)
    fn rotate_right_left<K: Ord, V>(node: Box<AvlNode<K, V>>) -> Box<AvlNode<K, V>>;

    /// Get balance factor
    fn balance_factor<K: Ord, V>(node: &AvlNode<K, V>) -> i32;
}
```

### Python Implementation

```python
from typing import TypeVar, Generic

K = TypeVar("K")
V = TypeVar("V")

class AvlTree(Generic[K, V]):
    def __init__(self) -> None: ...
    def insert(self, key: K, value: V) -> V | None: ...
    def remove(self, key: K) -> V | None: ...
    def get(self, key: K) -> V | None: ...
    def __contains__(self, key: K) -> bool: ...
    def __len__(self) -> int: ...
    def height(self) -> int: ...
    def is_balanced(self) -> bool: ...
    def inorder(self) -> list[tuple[K, V]]: ...
```

## Rotation Cases

### LL Case (Right Rotation)
```
      z                y
     / \             /   \
    y   T4   -->    x     z
   / \             / \   / \
  x   T3          T1 T2 T3 T4
 / \
T1 T2
```

### RR Case (Left Rotation)
```
  z                   y
 / \                /   \
T1  y      -->     z     x
   / \            / \   / \
  T2  x          T1 T2 T3 T4
     / \
    T3 T4
```

### LR Case (Left-Right Rotation)
```
     z               z               x
    / \            / \            /   \
   y   T4  -->    x   T4  -->    y     z
  / \            / \            / \   / \
 T1  x          y  T3          T1 T2 T3 T4
    / \        / \
   T2 T3      T1 T2
```

### RL Case (Right-Left Rotation)
```
   z               z                 x
  / \            / \              /   \
 T1  y    -->   T1  x     -->    z     y
    / \            / \          / \   / \
   x  T4          T2  y        T1 T2 T3 T4
  / \                / \
 T2 T3              T3 T4
```

## Test Cases

```rust
#[test]
fn test_insert_balance() {
    let mut avl = AvlTree::new();

    // Insert in order that would create imbalance
    avl.insert(3, "three");
    avl.insert(2, "two");
    avl.insert(1, "one");  // Would be unbalanced without rotation

    assert!(avl.is_balanced());
    assert_eq!(avl.height(), 2);  // Balanced tree height
}

#[test]
fn test_all_rotation_cases() {
    // LL case
    let mut avl = AvlTree::new();
    for i in [30, 20, 10] {
        avl.insert(i, i);
    }
    assert!(avl.is_balanced());

    // RR case
    let mut avl = AvlTree::new();
    for i in [10, 20, 30] {
        avl.insert(i, i);
    }
    assert!(avl.is_balanced());

    // LR case
    let mut avl = AvlTree::new();
    for i in [30, 10, 20] {
        avl.insert(i, i);
    }
    assert!(avl.is_balanced());

    // RL case
    let mut avl = AvlTree::new();
    for i in [10, 30, 20] {
        avl.insert(i, i);
    }
    assert!(avl.is_balanced());
}

#[test]
fn test_large_insert() {
    let mut avl = AvlTree::new();

    for i in 0..1000 {
        avl.insert(i, i);
        assert!(avl.is_balanced());
    }

    // Height should be logarithmic
    assert!(avl.height() <= 15);  // log2(1000) ≈ 10
}

#[test]
fn test_remove_balance() {
    let mut avl = AvlTree::new();

    for i in 0..100 {
        avl.insert(i, i);
    }

    for i in 0..50 {
        avl.remove(&i);
        assert!(avl.is_balanced());
    }
}

#[test]
fn test_inorder() {
    let mut avl = AvlTree::new();
    for i in [5, 3, 7, 1, 9, 4, 6] {
        avl.insert(i, i);
    }

    let keys: Vec<_> = avl.inorder().iter().map(|(k, _)| **k).collect();
    assert_eq!(keys, vec![1, 3, 4, 5, 6, 7, 9]);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Insert with balancing | 25 |
| All 4 rotation cases | 20 |
| Remove with balancing | 20 |
| Height maintenance | 10 |
| is_balanced check | 10 |
| Basic operations | 10 |
| Edge cases | 5 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `avl.py`
