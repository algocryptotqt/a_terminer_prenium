# Exercise 01: Union-Find (Disjoint Set Union)

## Concepts Covered
- **1.4.2.f-q** Path compression, union by rank/size, complexity, weighted, rollback, persistent

## Objective

Implement a comprehensive Union-Find data structure with all optimizations.

## Requirements

### Rust Implementation

```rust
pub mod union_find {
    /// Basic Union-Find with path compression and union by rank
    pub struct UnionFind {
        parent: Vec<usize>,
        rank: Vec<usize>,
        count: usize,  // Number of components
    }

    impl UnionFind {
        pub fn new(n: usize) -> Self;

        /// Find root with path compression - O(α(n))
        pub fn find(&mut self, x: usize) -> usize;

        /// Union by rank - O(α(n))
        pub fn union(&mut self, x: usize, y: usize) -> bool;

        /// Check if x and y are connected
        pub fn connected(&mut self, x: usize, y: usize) -> bool;

        /// Number of connected components
        pub fn count(&self) -> usize;

        /// Size of component containing x
        pub fn size(&mut self, x: usize) -> usize;
    }

    /// Union-Find with union by size
    pub struct UnionFindSize {
        parent: Vec<usize>,
        size: Vec<usize>,
    }

    impl UnionFindSize {
        pub fn new(n: usize) -> Self;
        pub fn find(&mut self, x: usize) -> usize;
        pub fn union(&mut self, x: usize, y: usize) -> bool;
        pub fn size(&mut self, x: usize) -> usize;
    }

    /// Weighted Union-Find for path queries
    pub struct WeightedUnionFind {
        parent: Vec<usize>,
        rank: Vec<usize>,
        diff: Vec<i64>,  // diff[x] = weight(x) - weight(parent[x])
    }

    impl WeightedUnionFind {
        pub fn new(n: usize) -> Self;

        /// Find with weight accumulation
        pub fn find(&mut self, x: usize) -> (usize, i64);

        /// Union with weight constraint: weight(y) - weight(x) = w
        pub fn union(&mut self, x: usize, y: usize, w: i64) -> bool;

        /// Get weight difference between x and y (if connected)
        pub fn diff(&mut self, x: usize, y: usize) -> Option<i64>;
    }

    /// Union-Find with rollback support
    pub struct RollbackUnionFind {
        parent: Vec<usize>,
        rank: Vec<usize>,
        history: Vec<(usize, usize, usize)>,  // (node, old_parent, old_rank)
    }

    impl RollbackUnionFind {
        pub fn new(n: usize) -> Self;
        pub fn find(&self, x: usize) -> usize;  // No path compression
        pub fn union(&mut self, x: usize, y: usize) -> bool;
        pub fn save(&self) -> usize;  // Save checkpoint
        pub fn rollback(&mut self, checkpoint: usize);  // Restore to checkpoint
    }

    // Applications

    /// Kruskal's MST using Union-Find
    pub fn kruskal_mst(n: usize, edges: &[(usize, usize, i64)]) -> (i64, Vec<(usize, usize, i64)>);

    /// Count connected components
    pub fn count_components(n: usize, edges: &[(usize, usize)]) -> usize;

    /// Detect cycle in undirected graph
    pub fn has_cycle(n: usize, edges: &[(usize, usize)]) -> bool;

    /// Earliest time when all nodes connected
    pub fn earliest_connection(n: usize, edges: &[(usize, usize, i32)]) -> i32;

    /// Accounts merge problem
    pub fn accounts_merge(accounts: Vec<Vec<String>>) -> Vec<Vec<String>>;
}
```

### Python Implementation

```python
class UnionFind:
    def __init__(self, n: int) -> None: ...
    def find(self, x: int) -> int: ...
    def union(self, x: int, y: int) -> bool: ...
    def connected(self, x: int, y: int) -> bool: ...
    def count(self) -> int: ...
    def size(self, x: int) -> int: ...

class WeightedUnionFind:
    def __init__(self, n: int) -> None: ...
    def find(self, x: int) -> tuple[int, int]: ...
    def union(self, x: int, y: int, w: int) -> bool: ...
    def diff(self, x: int, y: int) -> int | None: ...

class RollbackUnionFind:
    def __init__(self, n: int) -> None: ...
    def find(self, x: int) -> int: ...
    def union(self, x: int, y: int) -> bool: ...
    def save(self) -> int: ...
    def rollback(self, checkpoint: int) -> None: ...

def kruskal_mst(n: int, edges: list[tuple[int, int, int]]) -> tuple[int, list[tuple[int, int, int]]]: ...
def count_components(n: int, edges: list[tuple[int, int]]) -> int: ...
def has_cycle(n: int, edges: list[tuple[int, int]]) -> bool: ...
```

## Test Cases

```rust
#[test]
fn test_basic_union_find() {
    let mut uf = UnionFind::new(10);

    uf.union(0, 1);
    uf.union(2, 3);
    uf.union(0, 2);

    assert!(uf.connected(0, 3));
    assert!(uf.connected(1, 2));
    assert!(!uf.connected(0, 4));
    assert_eq!(uf.count(), 7);
}

#[test]
fn test_size() {
    let mut uf = UnionFindSize::new(5);

    uf.union(0, 1);
    uf.union(0, 2);

    assert_eq!(uf.size(0), 3);
    assert_eq!(uf.size(1), 3);
    assert_eq!(uf.size(3), 1);
}

#[test]
fn test_weighted() {
    let mut wuf = WeightedUnionFind::new(5);

    // weight(1) - weight(0) = 5
    wuf.union(0, 1, 5);
    // weight(2) - weight(1) = 3
    wuf.union(1, 2, 3);

    // weight(2) - weight(0) = 5 + 3 = 8
    assert_eq!(wuf.diff(0, 2), Some(8));
    assert_eq!(wuf.diff(2, 0), Some(-8));
}

#[test]
fn test_rollback() {
    let mut uf = RollbackUnionFind::new(5);

    uf.union(0, 1);
    let checkpoint = uf.save();
    uf.union(0, 2);

    assert!(uf.find(0) == uf.find(2));

    uf.rollback(checkpoint);
    assert!(uf.find(0) != uf.find(2));
    assert!(uf.find(0) == uf.find(1));
}

#[test]
fn test_kruskal() {
    let edges = vec![
        (0, 1, 4), (0, 7, 8), (1, 2, 8), (1, 7, 11),
        (2, 3, 7), (2, 5, 4), (2, 8, 2), (3, 4, 9),
        (3, 5, 14), (4, 5, 10), (5, 6, 2), (6, 7, 1), (6, 8, 6), (7, 8, 7)
    ];

    let (cost, mst) = kruskal_mst(9, &edges);
    assert_eq!(cost, 37);
    assert_eq!(mst.len(), 8);  // n-1 edges
}

#[test]
fn test_cycle_detection() {
    assert!(!has_cycle(4, &[(0, 1), (1, 2), (2, 3)]));
    assert!(has_cycle(4, &[(0, 1), (1, 2), (2, 0)]));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic Union-Find with path compression | 20 |
| Union by rank/size | 10 |
| Component count and size | 10 |
| Weighted Union-Find | 20 |
| Rollback Union-Find | 15 |
| Kruskal's MST | 15 |
| Cycle detection | 10 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `union_find.py`
