# Exercise 03: Treaps & Splay Trees

## Concepts Covered
- **1.3.10-13** Randomized BSTs, treaps, priorities
- **1.3.14-17** Splay trees, zig-zig, zig-zag, amortized O(log n)

## Objective

Implement self-balancing trees using randomization (treaps) and self-adjusting (splay trees).

## Requirements

### Rust Implementation

```rust
pub mod treap {
    use std::cmp::Ordering;

    pub struct TreapNode<K: Ord, V> {
        key: K,
        value: V,
        priority: u64,
        left: Option<Box<TreapNode<K, V>>>,
        right: Option<Box<TreapNode<K, V>>>,
        size: usize,
    }

    pub struct Treap<K: Ord, V> {
        root: Option<Box<TreapNode<K, V>>>,
    }

    impl<K: Ord, V> Treap<K, V> {
        pub fn new() -> Self;

        /// Insert with random priority
        pub fn insert(&mut self, key: K, value: V);

        /// Delete by key
        pub fn delete(&mut self, key: &K) -> Option<V>;

        /// Search for key
        pub fn get(&self, key: &K) -> Option<&V>;

        /// Split treap into (< key, >= key)
        pub fn split(root: Option<Box<TreapNode<K, V>>>, key: &K)
            -> (Option<Box<TreapNode<K, V>>>, Option<Box<TreapNode<K, V>>>);

        /// Merge two treaps (all keys in left < all keys in right)
        pub fn merge(left: Option<Box<TreapNode<K, V>>>, right: Option<Box<TreapNode<K, V>>>)
            -> Option<Box<TreapNode<K, V>>>;

        /// K-th smallest element (1-indexed)
        pub fn kth(&self, k: usize) -> Option<&K>;

        /// Count elements < key
        pub fn count_less(&self, key: &K) -> usize;

        /// Range query: elements in [lo, hi]
        pub fn range(&self, lo: &K, hi: &K) -> Vec<&K>;
    }

    /// Implicit Treap for sequence operations
    pub struct ImplicitTreap<T> {
        root: Option<Box<ImplicitNode<T>>>,
    }

    impl<T: Clone> ImplicitTreap<T> {
        pub fn new() -> Self;

        /// Insert at position
        pub fn insert_at(&mut self, pos: usize, value: T);

        /// Delete at position
        pub fn delete_at(&mut self, pos: usize) -> Option<T>;

        /// Get element at position
        pub fn get(&self, pos: usize) -> Option<&T>;

        /// Reverse range [l, r]
        pub fn reverse(&mut self, l: usize, r: usize);

        /// Cyclic shift range [l, r] by k positions
        pub fn cyclic_shift(&mut self, l: usize, r: usize, k: usize);
    }
}

pub mod splay {
    pub struct SplayNode<K: Ord, V> {
        key: K,
        value: V,
        left: Option<Box<SplayNode<K, V>>>,
        right: Option<Box<SplayNode<K, V>>>,
        parent: *mut SplayNode<K, V>,  // Raw pointer for parent
    }

    pub struct SplayTree<K: Ord, V> {
        root: Option<Box<SplayNode<K, V>>>,
    }

    impl<K: Ord, V> SplayTree<K, V> {
        pub fn new() -> Self;

        /// Splay node to root
        fn splay(&mut self, node: *mut SplayNode<K, V>);

        /// Zig rotation (single rotation)
        fn zig(&mut self, node: *mut SplayNode<K, V>);

        /// Zig-zig rotation (both same direction)
        fn zig_zig(&mut self, node: *mut SplayNode<K, V>);

        /// Zig-zag rotation (different directions)
        fn zig_zag(&mut self, node: *mut SplayNode<K, V>);

        /// Insert key-value pair
        pub fn insert(&mut self, key: K, value: V);

        /// Search and splay
        pub fn get(&mut self, key: &K) -> Option<&V>;

        /// Delete key
        pub fn delete(&mut self, key: &K) -> Option<V>;

        /// Split tree at key
        pub fn split(&mut self, key: &K) -> (Self, Self);

        /// Join two splay trees
        pub fn join(left: Self, right: Self) -> Self;
    }

    /// Link-Cut Trees using splay trees (for dynamic tree connectivity)
    pub struct LinkCutTree {
        nodes: Vec<LCTNode>,
    }

    impl LinkCutTree {
        pub fn new(n: usize) -> Self;

        /// Link node v as child of node u
        pub fn link(&mut self, u: usize, v: usize);

        /// Cut edge between node v and its parent
        pub fn cut(&mut self, v: usize);

        /// Find root of tree containing v
        pub fn find_root(&mut self, v: usize) -> usize;

        /// Path query from u to v
        pub fn path_aggregate(&mut self, u: usize, v: usize) -> i64;

        /// LCA of u and v
        pub fn lca(&mut self, u: usize, v: usize) -> usize;
    }
}
```

### Python Implementation

```python
import random
from typing import TypeVar, Generic, Optional, List, Tuple

K = TypeVar('K')
V = TypeVar('V')

class TreapNode(Generic[K, V]):
    def __init__(self, key: K, value: V):
        self.key = key
        self.value = value
        self.priority = random.randint(0, 2**63)
        self.left: Optional[TreapNode[K, V]] = None
        self.right: Optional[TreapNode[K, V]] = None
        self.size = 1

class Treap(Generic[K, V]):
    def __init__(self):
        self.root: Optional[TreapNode[K, V]] = None

    def insert(self, key: K, value: V) -> None: ...
    def delete(self, key: K) -> Optional[V]: ...
    def get(self, key: K) -> Optional[V]: ...

    @staticmethod
    def split(root: Optional[TreapNode[K, V]], key: K) -> Tuple[Optional[TreapNode[K, V]], Optional[TreapNode[K, V]]]: ...

    @staticmethod
    def merge(left: Optional[TreapNode[K, V]], right: Optional[TreapNode[K, V]]) -> Optional[TreapNode[K, V]]: ...

    def kth(self, k: int) -> Optional[K]: ...
    def count_less(self, key: K) -> int: ...

class SplayTree(Generic[K, V]):
    def __init__(self):
        self.root = None

    def _splay(self, node) -> None: ...
    def _zig(self, node) -> None: ...
    def _zig_zig(self, node) -> None: ...
    def _zig_zag(self, node) -> None: ...

    def insert(self, key: K, value: V) -> None: ...
    def get(self, key: K) -> Optional[V]: ...
    def delete(self, key: K) -> Optional[V]: ...
```

## Test Cases

```rust
#[test]
fn test_treap_operations() {
    let mut treap = Treap::new();
    for i in 0..1000 {
        treap.insert(i, i * 2);
    }

    assert_eq!(treap.get(&500), Some(&1000));
    assert_eq!(treap.kth(1), Some(&0));
    assert_eq!(treap.kth(500), Some(&499));
    assert_eq!(treap.count_less(&500), 500);
}

#[test]
fn test_treap_split_merge() {
    let mut treap = Treap::new();
    for i in [5, 2, 8, 1, 3, 7, 9] {
        treap.insert(i, i);
    }

    let (left, right) = Treap::split(treap.root.take(), &5);
    // left has 1, 2, 3; right has 5, 7, 8, 9

    let merged = Treap::merge(left, right);
    // Should contain all elements in order
}

#[test]
fn test_splay_amortized() {
    let mut splay = SplayTree::new();
    for i in 0..1000 {
        splay.insert(i, i);
    }

    // Sequential access should be efficient due to splaying
    for i in 0..1000 {
        assert_eq!(splay.get(&i), Some(&i));
    }
}

#[test]
fn test_implicit_treap_reverse() {
    let mut treap = ImplicitTreap::new();
    for i in 1..=5 {
        treap.insert_at(treap.len(), i);
    }
    // [1, 2, 3, 4, 5]

    treap.reverse(1, 3);  // Reverse indices 1-3
    // [1, 4, 3, 2, 5]

    assert_eq!(treap.get(1), Some(&4));
    assert_eq!(treap.get(2), Some(&3));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Treap insert/delete | 20 |
| Split/Merge operations | 20 |
| Splay tree rotations | 20 |
| Implicit treap | 20 |
| Order statistics | 15 |
| Edge cases | 5 |
| **Total** | **100** |
