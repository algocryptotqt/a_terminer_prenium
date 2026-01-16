# Exercise 02: Red-Black Trees

## Concepts Covered
- **1.3.10.e-h** Properties, black-height, proofs
- **1.3.11.f-i** Insert cases, propagation, implementation
- **1.3.12.f-i** Delete cases, implementation
- **1.3.13.g-h** AVL vs RB comparison

## Objective

Implement a complete Red-Black tree with all insertion and deletion cases.

## Requirements

### Rust Implementation

```rust
pub mod red_black {
    #[derive(Clone, Copy, PartialEq, Debug)]
    pub enum Color { Red, Black }

    pub struct RBTree<K: Ord, V> {
        root: Option<Box<RBNode<K, V>>>,
        len: usize,
    }

    struct RBNode<K: Ord, V> {
        key: K,
        value: V,
        color: Color,
        left: Option<Box<RBNode<K, V>>>,
        right: Option<Box<RBNode<K, V>>>,
    }

    impl<K: Ord, V> RBTree<K, V> {
        pub fn new() -> Self;
        pub fn insert(&mut self, key: K, value: V) -> Option<V>;
        pub fn remove(&mut self, key: &K) -> Option<V>;
        pub fn get(&self, key: &K) -> Option<&V>;
        pub fn contains(&self, key: &K) -> bool;
        pub fn len(&self) -> usize;

        /// Verify all RB properties
        pub fn is_valid(&self) -> bool;

        /// Check property 1: Every node is red or black
        pub fn check_colors(&self) -> bool;

        /// Check property 4: Red node has black children
        pub fn check_red_children(&self) -> bool;

        /// Check property 5: Same black-height on all paths
        pub fn check_black_height(&self) -> bool;

        /// Get black-height of tree
        pub fn black_height(&self) -> usize;
    }
}
```

## Red-Black Properties

1. Every node is either red or black
2. The root is black
3. Every leaf (NIL) is black
4. If a node is red, both children are black
5. All paths from root to leaves have same black-height

## Test Cases

```rust
#[test]
fn test_insert_maintains_properties() {
    let mut tree = RBTree::new();
    for i in 0..100 {
        tree.insert(i, i);
        assert!(tree.is_valid());
    }
}

#[test]
fn test_delete_maintains_properties() {
    let mut tree = RBTree::new();
    for i in 0..50 { tree.insert(i, i); }
    for i in 0..25 {
        tree.remove(&i);
        assert!(tree.is_valid());
    }
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Insert with rebalancing | 30 |
| Delete with rebalancing | 30 |
| Property validation | 20 |
| Basic operations | 15 |
| Edge cases | 5 |
| **Total** | **100** |
