# Exercise 00: Binary Search Tree Operations

## Concepts Covered
- **1.3.1.i** BST complexity analysis
- **1.3.2.g-j** Level-order, Morris traversal
- **1.3.3.e-g** Deletion implementations, lazy deletion
- **1.3.4.f** Motivation for balancing
- **1.3.5.g-h** Augmented BST, order statistics tree

## Objective

Implement a complete Binary Search Tree with advanced operations and analysis.

## Requirements

### Rust Implementation

```rust
pub mod bst {
    use std::cmp::Ordering;

    /// Binary Search Tree node
    pub struct BstNode<K: Ord, V> {
        key: K,
        value: V,
        left: Option<Box<BstNode<K, V>>>,
        right: Option<Box<BstNode<K, V>>>,
        size: usize,  // For order statistics
    }

    /// Binary Search Tree
    pub struct Bst<K: Ord, V> {
        root: Option<Box<BstNode<K, V>>>,
    }

    impl<K: Ord, V> Bst<K, V> {
        pub fn new() -> Self;

        /// Insert key-value pair
        pub fn insert(&mut self, key: K, value: V) -> Option<V>;

        /// Get value by key
        pub fn get(&self, key: &K) -> Option<&V>;

        /// Get mutable value
        pub fn get_mut(&mut self, key: &K) -> Option<&mut V>;

        /// Remove key and return value
        pub fn remove(&mut self, key: &K) -> Option<V>;

        /// Check if key exists
        pub fn contains(&self, key: &K) -> bool;

        /// Number of elements
        pub fn len(&self) -> usize;

        /// Check if empty
        pub fn is_empty(&self) -> bool;

        /// Get minimum key
        pub fn min(&self) -> Option<&K>;

        /// Get maximum key
        pub fn max(&self) -> Option<&K>;

        /// Get successor (next larger key)
        pub fn successor(&self, key: &K) -> Option<&K>;

        /// Get predecessor (next smaller key)
        pub fn predecessor(&self, key: &K) -> Option<&K>;

        /// Get kth smallest element (1-indexed)
        pub fn select(&self, k: usize) -> Option<&K>;

        /// Count elements less than key
        pub fn rank(&self, key: &K) -> usize;

        /// Range query: keys in [lo, hi]
        pub fn range(&self, lo: &K, hi: &K) -> Vec<&K>;

        /// Height of tree
        pub fn height(&self) -> usize;

        // Traversals

        /// Inorder traversal (sorted order)
        pub fn inorder(&self) -> Vec<&K>;

        /// Preorder traversal
        pub fn preorder(&self) -> Vec<&K>;

        /// Postorder traversal
        pub fn postorder(&self) -> Vec<&K>;

        /// Level-order (BFS) traversal
        pub fn level_order(&self) -> Vec<Vec<&K>>;

        /// Morris inorder traversal (O(1) space)
        pub fn morris_inorder(&self) -> Vec<&K>;
    }

    // Tree construction

    /// Build balanced BST from sorted array
    pub fn from_sorted<K: Ord + Clone, V: Clone>(items: &[(K, V)]) -> Bst<K, V>;

    /// Check if tree is valid BST
    pub fn is_valid_bst<K: Ord, V>(tree: &Bst<K, V>) -> bool;

    /// Serialize tree to string
    pub fn serialize<K: Ord + ToString, V>(tree: &Bst<K, V>) -> String;

    /// Deserialize tree from string
    pub fn deserialize<K: Ord + std::str::FromStr, V: Default>(s: &str) -> Option<Bst<K, V>>;

    // Tree algorithms

    /// Lowest Common Ancestor
    pub fn lca<K: Ord, V>(tree: &Bst<K, V>, k1: &K, k2: &K) -> Option<&K>;

    /// Distance between two nodes
    pub fn distance<K: Ord, V>(tree: &Bst<K, V>, k1: &K, k2: &K) -> Option<usize>;

    /// Check if tree is balanced
    pub fn is_balanced<K: Ord, V>(tree: &Bst<K, V>) -> bool;
}
```

### Python Implementation

```python
from typing import TypeVar, Generic, Iterator

K = TypeVar("K")
V = TypeVar("V")

class Bst(Generic[K, V]):
    def __init__(self) -> None: ...
    def insert(self, key: K, value: V) -> V | None: ...
    def get(self, key: K) -> V | None: ...
    def remove(self, key: K) -> V | None: ...
    def __contains__(self, key: K) -> bool: ...
    def __len__(self) -> int: ...
    def min(self) -> K | None: ...
    def max(self) -> K | None: ...
    def successor(self, key: K) -> K | None: ...
    def predecessor(self, key: K) -> K | None: ...
    def select(self, k: int) -> K | None: ...
    def rank(self, key: K) -> int: ...
    def range(self, lo: K, hi: K) -> list[K]: ...
    def height(self) -> int: ...
    def inorder(self) -> list[K]: ...
    def level_order(self) -> list[list[K]]: ...
    def morris_inorder(self) -> list[K]: ...
```

## Test Cases

```rust
#[test]
fn test_basic_operations() {
    let mut bst = Bst::new();
    bst.insert(5, "five");
    bst.insert(3, "three");
    bst.insert(7, "seven");
    bst.insert(1, "one");

    assert_eq!(bst.get(&5), Some(&"five"));
    assert_eq!(bst.len(), 4);
    assert!(!bst.is_empty());
}

#[test]
fn test_min_max() {
    let mut bst = Bst::new();
    for i in [5, 3, 7, 1, 9] {
        bst.insert(i, i);
    }

    assert_eq!(bst.min(), Some(&1));
    assert_eq!(bst.max(), Some(&9));
}

#[test]
fn test_successor_predecessor() {
    let mut bst = Bst::new();
    for i in [5, 3, 7, 1, 9, 4, 6] {
        bst.insert(i, i);
    }

    assert_eq!(bst.successor(&5), Some(&6));
    assert_eq!(bst.predecessor(&5), Some(&4));
    assert_eq!(bst.successor(&9), None);
}

#[test]
fn test_select_rank() {
    let mut bst = Bst::new();
    for i in [5, 3, 7, 1, 9] {
        bst.insert(i, i);
    }

    assert_eq!(bst.select(1), Some(&1));  // 1st smallest
    assert_eq!(bst.select(3), Some(&5));  // 3rd smallest
    assert_eq!(bst.rank(&5), 2);          // 2 elements < 5
}

#[test]
fn test_traversals() {
    let mut bst = Bst::new();
    for i in [4, 2, 6, 1, 3, 5, 7] {
        bst.insert(i, i);
    }

    assert_eq!(bst.inorder(), vec![&1, &2, &3, &4, &5, &6, &7]);
    assert_eq!(bst.preorder(), vec![&4, &2, &1, &3, &6, &5, &7]);
}

#[test]
fn test_morris_traversal() {
    let mut bst = Bst::new();
    for i in [4, 2, 6, 1, 3, 5, 7] {
        bst.insert(i, i);
    }

    // Morris should give same result as regular inorder
    assert_eq!(bst.morris_inorder(), bst.inorder());
}

#[test]
fn test_level_order() {
    let mut bst = Bst::new();
    for i in [4, 2, 6, 1, 3, 5, 7] {
        bst.insert(i, i);
    }

    let levels = bst.level_order();
    assert_eq!(levels[0], vec![&4]);
    assert_eq!(levels[1], vec![&2, &6]);
    assert_eq!(levels[2], vec![&1, &3, &5, &7]);
}

#[test]
fn test_remove() {
    let mut bst = Bst::new();
    for i in [5, 3, 7, 1, 9] {
        bst.insert(i, i);
    }

    assert_eq!(bst.remove(&3), Some(3));
    assert!(!bst.contains(&3));
    assert_eq!(bst.len(), 4);

    // Tree should still be valid BST
    assert!(is_valid_bst(&bst));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic operations (insert, get, remove) | 20 |
| Min/Max/Successor/Predecessor | 15 |
| Order statistics (select, rank) | 15 |
| Traversals (all 4 types) | 15 |
| Morris traversal | 10 |
| Range queries | 10 |
| Tree validation | 10 |
| Edge cases | 5 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `bst.py`
