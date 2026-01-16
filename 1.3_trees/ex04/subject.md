# Exercise 04: B-Trees & B+ Trees

## Concepts Covered
- **1.3.18-20** B-tree properties, insertion, deletion
- **1.3.21-22** B+ trees, leaf linking, range queries

## Objective

Implement disk-oriented balanced trees optimized for I/O operations.

## Requirements

### Rust Implementation

```rust
pub mod btree {
    const MIN_DEGREE: usize = 2;  // Minimum degree t (min keys = t-1, max keys = 2t-1)

    pub struct BTreeNode<K: Ord + Clone, V: Clone> {
        keys: Vec<K>,
        values: Vec<V>,
        children: Vec<Box<BTreeNode<K, V>>>,
        leaf: bool,
    }

    pub struct BTree<K: Ord + Clone, V: Clone> {
        root: Option<Box<BTreeNode<K, V>>>,
        t: usize,  // Minimum degree
    }

    impl<K: Ord + Clone, V: Clone> BTree<K, V> {
        pub fn new(min_degree: usize) -> Self;

        /// Search for a key
        pub fn search(&self, key: &K) -> Option<&V>;

        /// Insert key-value pair
        pub fn insert(&mut self, key: K, value: V);

        /// Split a full child node
        fn split_child(&mut self, parent: &mut BTreeNode<K, V>, index: usize);

        /// Insert into non-full node
        fn insert_non_full(&mut self, node: &mut BTreeNode<K, V>, key: K, value: V);

        /// Delete a key
        pub fn delete(&mut self, key: &K) -> Option<V>;

        /// Get minimum key in subtree
        fn min_key(node: &BTreeNode<K, V>) -> &K;

        /// Get maximum key in subtree
        fn max_key(node: &BTreeNode<K, V>) -> &K;

        /// Merge child i with child i+1
        fn merge(&mut self, node: &mut BTreeNode<K, V>, i: usize);

        /// Borrow from left sibling
        fn borrow_from_left(&mut self, node: &mut BTreeNode<K, V>, i: usize);

        /// Borrow from right sibling
        fn borrow_from_right(&mut self, node: &mut BTreeNode<K, V>, i: usize);

        /// In-order traversal
        pub fn inorder(&self) -> Vec<(&K, &V)>;

        /// Range query [lo, hi]
        pub fn range(&self, lo: &K, hi: &K) -> Vec<(&K, &V)>;

        /// Height of tree
        pub fn height(&self) -> usize;

        /// Number of keys
        pub fn len(&self) -> usize;
    }
}

pub mod bplus_tree {
    pub struct BPlusLeaf<K: Ord + Clone, V: Clone> {
        keys: Vec<K>,
        values: Vec<V>,
        next: Option<*mut BPlusLeaf<K, V>>,  // Pointer to next leaf
    }

    pub struct BPlusInternal<K: Ord + Clone, V: Clone> {
        keys: Vec<K>,
        children: Vec<BPlusNode<K, V>>,
    }

    pub enum BPlusNode<K: Ord + Clone, V: Clone> {
        Leaf(Box<BPlusLeaf<K, V>>),
        Internal(Box<BPlusInternal<K, V>>),
    }

    pub struct BPlusTree<K: Ord + Clone, V: Clone> {
        root: Option<BPlusNode<K, V>>,
        first_leaf: Option<*mut BPlusLeaf<K, V>>,
        t: usize,
    }

    impl<K: Ord + Clone, V: Clone> BPlusTree<K, V> {
        pub fn new(min_degree: usize) -> Self;

        /// Point query
        pub fn search(&self, key: &K) -> Option<&V>;

        /// Insert key-value
        pub fn insert(&mut self, key: K, value: V);

        /// Delete key
        pub fn delete(&mut self, key: &K) -> Option<V>;

        /// Efficient range query using leaf links
        pub fn range(&self, lo: &K, hi: &K) -> Vec<(&K, &V)>;

        /// Scan all leaves in order
        pub fn scan(&self) -> Vec<(&K, &V)>;

        /// Bulk load from sorted data
        pub fn bulk_load(data: Vec<(K, V)>, min_degree: usize) -> Self;
    }
}

/// Simulated disk-based B-tree with page management
pub mod disk_btree {
    use std::collections::HashMap;

    pub type PageId = u64;

    pub struct Page {
        id: PageId,
        data: Vec<u8>,
    }

    pub struct BufferPool {
        pages: HashMap<PageId, Page>,
        capacity: usize,
        lru: Vec<PageId>,
    }

    impl BufferPool {
        pub fn new(capacity: usize) -> Self;
        pub fn fetch(&mut self, page_id: PageId) -> &Page;
        pub fn flush(&mut self, page_id: PageId);
        pub fn evict(&mut self) -> Option<PageId>;
    }

    pub struct DiskBTree {
        root_page: PageId,
        buffer_pool: BufferPool,
        next_page_id: PageId,
        page_size: usize,
    }

    impl DiskBTree {
        pub fn new(page_size: usize, buffer_size: usize) -> Self;
        pub fn search(&mut self, key: &[u8]) -> Option<Vec<u8>>;
        pub fn insert(&mut self, key: Vec<u8>, value: Vec<u8>);
        pub fn io_count(&self) -> usize;
    }
}
```

### Python Implementation

```python
from typing import TypeVar, Generic, Optional, List, Tuple

K = TypeVar('K')
V = TypeVar('V')

class BTreeNode(Generic[K, V]):
    def __init__(self, leaf: bool = True):
        self.keys: List[K] = []
        self.values: List[V] = []
        self.children: List[BTreeNode[K, V]] = []
        self.leaf = leaf

class BTree(Generic[K, V]):
    def __init__(self, t: int = 2):
        """Initialize B-tree with minimum degree t."""
        self.root: Optional[BTreeNode[K, V]] = None
        self.t = t

    def search(self, key: K) -> Optional[V]: ...
    def insert(self, key: K, value: V) -> None: ...
    def delete(self, key: K) -> Optional[V]: ...
    def range(self, lo: K, hi: K) -> List[Tuple[K, V]]: ...

class BPlusTree(Generic[K, V]):
    def __init__(self, t: int = 2):
        self.root = None
        self.t = t

    def search(self, key: K) -> Optional[V]: ...
    def insert(self, key: K, value: V) -> None: ...
    def range(self, lo: K, hi: K) -> List[Tuple[K, V]]: ...
    def scan(self) -> List[Tuple[K, V]]: ...
```

## Test Cases

```rust
#[test]
fn test_btree_insert() {
    let mut btree = BTree::new(2);  // 2-3-4 tree

    for i in 0..100 {
        btree.insert(i, i * 10);
    }

    for i in 0..100 {
        assert_eq!(btree.search(&i), Some(&(i * 10)));
    }

    // B-tree property: height is O(log_t n)
    assert!(btree.height() <= 5);
}

#[test]
fn test_btree_delete() {
    let mut btree = BTree::new(3);

    for i in 0..50 {
        btree.insert(i, i);
    }

    // Delete and verify
    for i in (0..50).step_by(2) {
        assert_eq!(btree.delete(&i), Some(i));
        assert_eq!(btree.search(&i), None);
    }

    // Remaining elements should still be searchable
    for i in (1..50).step_by(2) {
        assert_eq!(btree.search(&i), Some(&i));
    }
}

#[test]
fn test_bplus_range_query() {
    let mut bplus = BPlusTree::new(3);

    for i in 0..1000 {
        bplus.insert(i, format!("value_{}", i));
    }

    let range = bplus.range(&100, &200);
    assert_eq!(range.len(), 101);  // 100 to 200 inclusive

    // Verify sorted order
    for i in 0..range.len() - 1 {
        assert!(range[i].0 < range[i + 1].0);
    }
}

#[test]
fn test_bplus_scan() {
    let mut bplus = BPlusTree::new(2);
    let data: Vec<_> = (0..50).map(|i| (i, i * 2)).collect();

    for (k, v) in &data {
        bplus.insert(*k, *v);
    }

    let scanned = bplus.scan();
    assert_eq!(scanned.len(), 50);

    // Should be in sorted order
    for (i, (k, _)) in scanned.iter().enumerate() {
        assert_eq!(**k, i as i32);
    }
}

#[test]
fn test_bulk_load() {
    let data: Vec<_> = (0..10000).map(|i| (i, i.to_string())).collect();
    let bplus = BPlusTree::bulk_load(data.clone(), 50);

    for (k, v) in &data {
        assert_eq!(bplus.search(k), Some(v));
    }
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| B-tree insertion with splits | 20 |
| B-tree deletion with merging | 20 |
| B+ tree implementation | 20 |
| Range queries with leaf links | 15 |
| Bulk loading | 10 |
| Buffer pool simulation | 10 |
| Edge cases | 5 |
| **Total** | **100** |
