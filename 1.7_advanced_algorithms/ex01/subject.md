# Exercise 01: Backtracking & Branch and Bound

## Concepts Covered
- **1.7.2.d-l** Backtracking patterns, pruning strategies
- **1.7.3.d-k** Branch and bound, optimization problems

## Objective

Master backtracking and branch-and-bound techniques for solving constraint satisfaction and optimization problems.

## Requirements

### Rust Implementation

```rust
pub mod backtracking {
    /// N-Queens: find all solutions
    pub fn n_queens_all(n: usize) -> Vec<Vec<usize>>;

    /// N-Queens: count solutions
    pub fn n_queens_count(n: usize) -> usize;

    /// N-Queens with pruning (faster)
    pub fn n_queens_optimized(n: usize) -> usize;

    /// Sudoku solver
    pub fn solve_sudoku(board: &mut [[u8; 9]; 9]) -> bool;

    /// Sudoku with constraint propagation
    pub fn solve_sudoku_advanced(board: &mut [[u8; 9]; 9]) -> bool;

    /// Generate all permutations
    pub fn permutations<T: Clone>(items: &[T]) -> Vec<Vec<T>>;

    /// Generate permutations with pruning
    pub fn permutations_pruned<T, F>(items: &[T], valid: F) -> Vec<Vec<T>>
    where
        T: Clone,
        F: Fn(&[T]) -> bool;

    /// Generate all subsets
    pub fn subsets<T: Clone>(items: &[T]) -> Vec<Vec<T>>;

    /// Subsets with constraint
    pub fn subsets_constrained<T, F>(items: &[T], valid: F) -> Vec<Vec<T>>
    where
        T: Clone,
        F: Fn(&[T]) -> bool;

    /// Combination sum: find subsets summing to target
    pub fn combination_sum(candidates: &[i32], target: i32) -> Vec<Vec<i32>>;

    /// Palindrome partitioning
    pub fn palindrome_partition(s: &str) -> Vec<Vec<String>>;

    /// Word search in grid
    pub fn word_search(board: &[Vec<char>], word: &str) -> bool;

    /// All paths from source to target in graph
    pub fn all_paths(adj: &[Vec<usize>], src: usize, dst: usize) -> Vec<Vec<usize>>;

    /// Graph coloring
    pub fn graph_coloring(adj: &[Vec<usize>], k: usize) -> Option<Vec<usize>>;
}

pub mod branch_and_bound {
    /// 0/1 Knapsack with B&B
    pub fn knapsack_bb(
        weights: &[usize],
        values: &[i64],
        capacity: usize,
    ) -> (i64, Vec<bool>);

    /// TSP with B&B
    pub fn tsp_bb(dist: &[Vec<i64>]) -> (i64, Vec<usize>);

    /// Job assignment (minimize total cost)
    pub fn job_assignment_bb(cost: &[Vec<i64>]) -> (i64, Vec<usize>);

    /// Subset sum with B&B
    pub fn subset_sum_bb(arr: &[i64], target: i64) -> Option<Vec<usize>>;

    /// Maximum clique with B&B
    pub fn max_clique_bb(adj: &[Vec<bool>]) -> Vec<usize>;

    /// Minimum vertex cover with B&B
    pub fn min_vertex_cover_bb(adj: &[Vec<usize>]) -> Vec<usize>;
}

pub mod constraint_propagation {
    /// Arc consistency (AC-3 algorithm)
    pub fn ac3(
        domains: &mut [Vec<i32>],
        constraints: &[(usize, usize, Box<dyn Fn(i32, i32) -> bool>)],
    ) -> bool;

    /// Forward checking for CSP
    pub fn forward_checking<F>(
        domains: &[Vec<i32>],
        constraints: F,
    ) -> Option<Vec<i32>>
    where
        F: Fn(usize, i32, usize, i32) -> bool;

    /// Maintaining arc consistency (MAC)
    pub fn mac_solver<F>(
        domains: &[Vec<i32>],
        constraints: F,
    ) -> Option<Vec<i32>>
    where
        F: Fn(usize, i32, usize, i32) -> bool;
}

pub mod exact_cover {
    /// Dancing Links (DLX) for exact cover
    pub struct DLX {
        // Implementation of Knuth's Algorithm X with dancing links
    }

    impl DLX {
        pub fn new(matrix: &[Vec<bool>]) -> Self;
        pub fn solve(&mut self) -> Vec<Vec<usize>>;
        pub fn count_solutions(&mut self) -> usize;
    }

    /// Solve Sudoku using DLX
    pub fn sudoku_dlx(board: &[[u8; 9]; 9]) -> Option<[[u8; 9]; 9]>;

    /// Pentomino tiling using DLX
    pub fn pentomino_tiling(rows: usize, cols: usize) -> Vec<Vec<Vec<(usize, usize)>>>;

    /// N-Queens using DLX
    pub fn n_queens_dlx(n: usize) -> Vec<Vec<usize>>;
}
```

### Python Implementation

```python
from typing import List, Optional, Tuple, Callable

def n_queens_all(n: int) -> List[List[int]]: ...
def n_queens_count(n: int) -> int: ...
def solve_sudoku(board: List[List[int]]) -> bool: ...

def permutations(items: List) -> List[List]: ...
def subsets(items: List) -> List[List]: ...
def combination_sum(candidates: List[int], target: int) -> List[List[int]]: ...

def knapsack_bb(weights: List[int], values: List[int], capacity: int) -> Tuple[int, List[bool]]: ...
def tsp_bb(dist: List[List[int]]) -> Tuple[int, List[int]]: ...
def job_assignment_bb(cost: List[List[int]]) -> Tuple[int, List[int]]: ...

class DLX:
    def __init__(self, matrix: List[List[bool]]): ...
    def solve(self) -> List[List[int]]: ...
```

## Test Cases

```rust
#[test]
fn test_n_queens() {
    assert_eq!(n_queens_count(1), 1);
    assert_eq!(n_queens_count(4), 2);
    assert_eq!(n_queens_count(8), 92);

    let solutions = n_queens_all(4);
    assert_eq!(solutions.len(), 2);
}

#[test]
fn test_sudoku() {
    let mut board = [
        [5, 3, 0, 0, 7, 0, 0, 0, 0],
        [6, 0, 0, 1, 9, 5, 0, 0, 0],
        [0, 9, 8, 0, 0, 0, 0, 6, 0],
        [8, 0, 0, 0, 6, 0, 0, 0, 3],
        [4, 0, 0, 8, 0, 3, 0, 0, 1],
        [7, 0, 0, 0, 2, 0, 0, 0, 6],
        [0, 6, 0, 0, 0, 0, 2, 8, 0],
        [0, 0, 0, 4, 1, 9, 0, 0, 5],
        [0, 0, 0, 0, 8, 0, 0, 7, 9],
    ];

    assert!(solve_sudoku(&mut board));

    // Verify solution
    for i in 0..9 {
        for j in 0..9 {
            assert!(board[i][j] >= 1 && board[i][j] <= 9);
        }
    }
}

#[test]
fn test_combination_sum() {
    let result = combination_sum(&[2, 3, 6, 7], 7);
    assert!(result.contains(&vec![7]));
    assert!(result.contains(&vec![2, 2, 3]));
    assert_eq!(result.len(), 2);
}

#[test]
fn test_permutations() {
    let perms = permutations(&[1, 2, 3]);
    assert_eq!(perms.len(), 6);
}

#[test]
fn test_subsets() {
    let subs = subsets(&[1, 2, 3]);
    assert_eq!(subs.len(), 8);  // 2^3
}

#[test]
fn test_knapsack_bb() {
    let weights = vec![2, 3, 4, 5];
    let values = vec![3, 4, 5, 6];
    let (best, selected) = knapsack_bb(&weights, &values, 5);
    assert_eq!(best, 7);  // Items 0 and 1
}

#[test]
fn test_tsp_bb() {
    let dist = vec![
        vec![0, 10, 15, 20],
        vec![10, 0, 35, 25],
        vec![15, 35, 0, 30],
        vec![20, 25, 30, 0],
    ];
    let (cost, tour) = tsp_bb(&dist);
    assert_eq!(cost, 80);
}

#[test]
fn test_job_assignment() {
    let cost = vec![
        vec![9, 2, 7, 8],
        vec![6, 4, 3, 7],
        vec![5, 8, 1, 8],
        vec![7, 6, 9, 4],
    ];
    let (total, assignment) = job_assignment_bb(&cost);
    assert_eq!(total, 13);  // 2 + 3 + 5 + 4 or similar
}

#[test]
fn test_word_search() {
    let board = vec![
        vec!['A', 'B', 'C', 'E'],
        vec!['S', 'F', 'C', 'S'],
        vec!['A', 'D', 'E', 'E'],
    ];
    assert!(word_search(&board, "ABCCED"));
    assert!(word_search(&board, "SEE"));
    assert!(!word_search(&board, "ABCB"));
}

#[test]
fn test_graph_coloring() {
    // Triangle needs 3 colors
    let adj = vec![vec![1, 2], vec![0, 2], vec![0, 1]];
    assert!(graph_coloring(&adj, 2).is_none());
    assert!(graph_coloring(&adj, 3).is_some());
}

#[test]
fn test_dlx_exact_cover() {
    // Simple exact cover instance
    let matrix = vec![
        vec![true, false, false, true, false, false, true],
        vec![true, false, false, true, false, false, false],
        vec![false, false, false, true, true, false, true],
        vec![false, false, true, false, true, true, false],
        vec![false, true, true, false, false, true, true],
        vec![false, true, false, false, false, false, true],
    ];
    let mut dlx = DLX::new(&matrix);
    let solutions = dlx.solve();
    assert!(!solutions.is_empty());
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| N-Queens variants | 15 |
| Sudoku solver | 15 |
| Subset/permutation generation | 10 |
| Combination problems | 10 |
| Knapsack B&B | 15 |
| TSP B&B | 15 |
| Dancing Links | 15 |
| Edge cases | 5 |
| **Total** | **100** |
