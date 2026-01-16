# Exercise 02: DFS & BFS Fundamentals

## Concepts Covered
- **1.4.3.f-o** DFS traversal, timestamps, edge classification, cycles
- **1.4.4.d-k** BFS, shortest path in unweighted, multi-source BFS, 0-1 BFS

## Objective

Master graph traversal algorithms and their applications.

## Requirements

### Rust Implementation

```rust
pub mod traversal {
    use std::collections::VecDeque;

    /// DFS with timestamps and edge classification
    pub struct DFSResult {
        pub discovery: Vec<usize>,      // Discovery time
        pub finish: Vec<usize>,         // Finish time
        pub parent: Vec<Option<usize>>, // Parent in DFS tree
        pub tree_edges: Vec<(usize, usize)>,
        pub back_edges: Vec<(usize, usize)>,
        pub forward_edges: Vec<(usize, usize)>,
        pub cross_edges: Vec<(usize, usize)>,
    }

    /// Full DFS traversal with edge classification
    pub fn dfs_full(adj: &[Vec<usize>]) -> DFSResult;

    /// Iterative DFS (to avoid stack overflow)
    pub fn dfs_iterative(adj: &[Vec<usize>], start: usize) -> Vec<usize>;

    /// Check if graph has cycle (directed)
    pub fn has_cycle_directed(adj: &[Vec<usize>]) -> bool;

    /// Check if graph has cycle (undirected)
    pub fn has_cycle_undirected(adj: &[Vec<usize>]) -> bool;

    /// Find all cycles in directed graph
    pub fn find_cycles(adj: &[Vec<usize>]) -> Vec<Vec<usize>>;

    /// BFS shortest path (unweighted)
    pub fn bfs_shortest(adj: &[Vec<usize>], source: usize) -> Vec<i32>;

    /// BFS with path reconstruction
    pub fn bfs_path(adj: &[Vec<usize>], source: usize, target: usize) -> Option<Vec<usize>>;

    /// Multi-source BFS
    pub fn multi_source_bfs(adj: &[Vec<usize>], sources: &[usize]) -> Vec<i32>;

    /// 0-1 BFS for graphs with edge weights 0 or 1
    pub fn bfs_01(adj: &[Vec<(usize, u8)>], source: usize) -> Vec<i32>;

    /// Bidirectional BFS
    pub fn bidirectional_bfs(adj: &[Vec<usize>], source: usize, target: usize) -> Option<i32>;

    /// Connected components (undirected)
    pub fn connected_components(adj: &[Vec<usize>]) -> Vec<usize>;

    /// Count connected components
    pub fn count_components(adj: &[Vec<usize>]) -> usize;

    /// Check if graph is bipartite
    pub fn is_bipartite(adj: &[Vec<usize>]) -> bool;

    /// 2-coloring of bipartite graph
    pub fn bipartite_coloring(adj: &[Vec<usize>]) -> Option<Vec<u8>>;

    /// Find bridges (edges whose removal disconnects graph)
    pub fn find_bridges(adj: &[Vec<usize>]) -> Vec<(usize, usize)>;

    /// Find articulation points
    pub fn find_articulation_points(adj: &[Vec<usize>]) -> Vec<usize>;

    /// Biconnected components
    pub fn biconnected_components(adj: &[Vec<usize>]) -> Vec<Vec<(usize, usize)>>;
}

/// Grid-based BFS/DFS
pub mod grid_traversal {
    pub type Grid = Vec<Vec<char>>;

    /// Flood fill
    pub fn flood_fill(grid: &mut Grid, r: usize, c: usize, new_color: char);

    /// Count islands (connected '1' cells)
    pub fn count_islands(grid: &Grid) -> usize;

    /// Shortest path in grid (BFS)
    pub fn shortest_path_grid(
        grid: &Grid,
        start: (usize, usize),
        end: (usize, usize),
    ) -> Option<usize>;

    /// Multi-source distance in grid
    pub fn multi_source_distance(grid: &Grid, sources: &[(usize, usize)]) -> Vec<Vec<i32>>;

    /// Rotting oranges problem
    pub fn rotting_oranges(grid: &mut Grid) -> i32;

    /// Word ladder (BFS on implicit graph)
    pub fn word_ladder(begin: &str, end: &str, word_list: &[String]) -> i32;
}
```

### Python Implementation

```python
from typing import List, Optional, Tuple, Set
from collections import deque

class DFSResult:
    def __init__(self):
        self.discovery: List[int] = []
        self.finish: List[int] = []
        self.parent: List[Optional[int]] = []
        self.tree_edges: List[Tuple[int, int]] = []
        self.back_edges: List[Tuple[int, int]] = []

def dfs_full(adj: List[List[int]]) -> DFSResult: ...
def dfs_iterative(adj: List[List[int]], start: int) -> List[int]: ...
def has_cycle_directed(adj: List[List[int]]) -> bool: ...
def has_cycle_undirected(adj: List[List[int]]) -> bool: ...

def bfs_shortest(adj: List[List[int]], source: int) -> List[int]: ...
def bfs_path(adj: List[List[int]], source: int, target: int) -> Optional[List[int]]: ...
def multi_source_bfs(adj: List[List[int]], sources: List[int]) -> List[int]: ...
def bfs_01(adj: List[List[Tuple[int, int]]], source: int) -> List[int]: ...

def connected_components(adj: List[List[int]]) -> List[int]: ...
def is_bipartite(adj: List[List[int]]) -> bool: ...
def find_bridges(adj: List[List[int]]) -> List[Tuple[int, int]]: ...
def find_articulation_points(adj: List[List[int]]) -> List[int]: ...

# Grid traversal
def flood_fill(grid: List[List[str]], r: int, c: int, new_color: str) -> None: ...
def count_islands(grid: List[List[str]]) -> int: ...
def shortest_path_grid(grid: List[List[str]], start: Tuple[int, int], end: Tuple[int, int]) -> int: ...
```

## Test Cases

```rust
#[test]
fn test_dfs_edge_classification() {
    // Graph with back edge (cycle)
    let adj = vec![
        vec![1],    // 0 -> 1
        vec![2],    // 1 -> 2
        vec![0],    // 2 -> 0 (back edge)
    ];

    let result = dfs_full(&adj);
    assert!(!result.back_edges.is_empty());
    assert!(has_cycle_directed(&adj));
}

#[test]
fn test_bfs_shortest_path() {
    let adj = vec![
        vec![1, 2],     // 0
        vec![0, 3],     // 1
        vec![0, 3],     // 2
        vec![1, 2, 4],  // 3
        vec![3],        // 4
    ];

    let dist = bfs_shortest(&adj, 0);
    assert_eq!(dist, vec![0, 1, 1, 2, 3]);
}

#[test]
fn test_multi_source_bfs() {
    let adj = vec![
        vec![1], vec![0, 2], vec![1, 3], vec![2, 4], vec![3],
    ];
    let sources = vec![0, 4];

    let dist = multi_source_bfs(&adj, &sources);
    assert_eq!(dist, vec![0, 1, 2, 1, 0]);  // Distance to nearest source
}

#[test]
fn test_01_bfs() {
    // Edge weights are 0 or 1
    let adj = vec![
        vec![(1, 0), (2, 1)],  // 0 -> 1 (cost 0), 0 -> 2 (cost 1)
        vec![(3, 1)],          // 1 -> 3 (cost 1)
        vec![(3, 0)],          // 2 -> 3 (cost 0)
        vec![],
    ];

    let dist = bfs_01(&adj, 0);
    assert_eq!(dist, vec![0, 0, 1, 1]);  // 0->1->3 = 1, 0->2->3 = 1
}

#[test]
fn test_bipartite() {
    // Bipartite graph (cycle of even length)
    let adj = vec![vec![1, 3], vec![0, 2], vec![1, 3], vec![2, 0]];
    assert!(is_bipartite(&adj));

    // Non-bipartite (odd cycle)
    let adj2 = vec![vec![1, 2], vec![0, 2], vec![0, 1]];
    assert!(!is_bipartite(&adj2));
}

#[test]
fn test_bridges() {
    let adj = vec![
        vec![1, 2],     // 0
        vec![0, 2],     // 1
        vec![0, 1, 3],  // 2
        vec![2],        // 3
    ];

    let bridges = find_bridges(&adj);
    assert_eq!(bridges, vec![(2, 3)]);  // Only 2-3 is a bridge
}

#[test]
fn test_articulation_points() {
    let adj = vec![
        vec![1, 2],     // 0
        vec![0, 2],     // 1
        vec![0, 1, 3],  // 2
        vec![2, 4],     // 3
        vec![3],        // 4
    ];

    let ap = find_articulation_points(&adj);
    assert!(ap.contains(&2));
    assert!(ap.contains(&3));
}

#[test]
fn test_count_islands() {
    let grid = vec![
        vec!['1', '1', '0', '0', '0'],
        vec!['1', '1', '0', '0', '0'],
        vec!['0', '0', '1', '0', '0'],
        vec!['0', '0', '0', '1', '1'],
    ];

    assert_eq!(count_islands(&grid), 3);
}

#[test]
fn test_shortest_path_grid() {
    let grid = vec![
        vec!['.', '.', '.', '#'],
        vec!['#', '#', '.', '.'],
        vec!['.', '.', '.', '.'],
    ];

    let dist = shortest_path_grid(&grid, (0, 0), (2, 3));
    assert_eq!(dist, Some(5));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| DFS with edge classification | 15 |
| Cycle detection | 10 |
| BFS shortest path | 15 |
| Multi-source & 0-1 BFS | 15 |
| Bridges & articulation points | 20 |
| Bipartite check | 10 |
| Grid traversal | 10 |
| Edge cases | 5 |
| **Total** | **100** |
