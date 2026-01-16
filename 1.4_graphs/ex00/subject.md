# Exercise 00: Graph Representations

## Concepts Covered
- **1.4.1.h-o** Adjacency list, edge list, conversions, implicit graphs

## Objective

Implement multiple graph representations and understand their trade-offs.

## Requirements

### Rust Implementation

```rust
pub mod graph {
    /// Adjacency Matrix representation
    pub struct AdjMatrix {
        matrix: Vec<Vec<i32>>,  // INF for no edge
        n: usize,
    }

    impl AdjMatrix {
        pub fn new(n: usize) -> Self;
        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn has_edge(&self, u: usize, v: usize) -> bool;
        pub fn weight(&self, u: usize, v: usize) -> Option<i32>;
        pub fn neighbors(&self, u: usize) -> Vec<usize>;
    }

    /// Adjacency List representation
    pub struct AdjList {
        adj: Vec<Vec<(usize, i32)>>,  // (neighbor, weight)
        n: usize,
        m: usize,
    }

    impl AdjList {
        pub fn new(n: usize) -> Self;
        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn add_undirected_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn neighbors(&self, u: usize) -> &[(usize, i32)];
        pub fn degree(&self, u: usize) -> usize;
        pub fn vertex_count(&self) -> usize;
        pub fn edge_count(&self) -> usize;
    }

    /// Edge List representation
    pub struct EdgeList {
        edges: Vec<(usize, usize, i32)>,  // (from, to, weight)
        n: usize,
    }

    impl EdgeList {
        pub fn new(n: usize) -> Self;
        pub fn add_edge(&mut self, u: usize, v: usize, weight: i32);
        pub fn edges(&self) -> &[(usize, usize, i32)];

        /// Sort edges by weight
        pub fn sort_by_weight(&mut self);
    }

    // Conversions
    pub fn matrix_to_list(matrix: &AdjMatrix) -> AdjList;
    pub fn list_to_matrix(list: &AdjList) -> AdjMatrix;
    pub fn list_to_edges(list: &AdjList) -> EdgeList;
    pub fn edges_to_list(edges: &EdgeList) -> AdjList;

    /// Implicit graph: grid
    pub struct GridGraph {
        rows: usize,
        cols: usize,
        blocked: Vec<Vec<bool>>,
    }

    impl GridGraph {
        pub fn new(rows: usize, cols: usize) -> Self;
        pub fn block(&mut self, r: usize, c: usize);
        pub fn neighbors(&self, r: usize, c: usize) -> Vec<(usize, usize)>;
        pub fn neighbors_8(&self, r: usize, c: usize) -> Vec<(usize, usize)>;
    }
}
```

## Test Cases

```rust
#[test]
fn test_adj_list() {
    let mut g = AdjList::new(5);
    g.add_edge(0, 1, 10);
    g.add_edge(0, 2, 20);
    g.add_edge(1, 2, 5);

    assert_eq!(g.degree(0), 2);
    assert_eq!(g.neighbors(0), &[(1, 10), (2, 20)]);
}

#[test]
fn test_conversion() {
    let mut list = AdjList::new(3);
    list.add_edge(0, 1, 1);
    list.add_edge(1, 2, 2);

    let matrix = list_to_matrix(&list);
    assert_eq!(matrix.weight(0, 1), Some(1));

    let back = matrix_to_list(&matrix);
    assert_eq!(back.degree(0), 1);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Adjacency Matrix | 20 |
| Adjacency List | 25 |
| Edge List | 15 |
| Conversions | 20 |
| Grid Graph | 15 |
| Edge cases | 5 |
| **Total** | **100** |
