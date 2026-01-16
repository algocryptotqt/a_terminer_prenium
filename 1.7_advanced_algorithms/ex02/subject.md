# Exercise 02: Sqrt Decomposition & Mo's Algorithm

## Concepts Covered
- **1.7.4.d-l** Sqrt decomposition, block-based queries
- **1.7.5.d-k** Mo's algorithm, query ordering, offline processing

## Objective

Implement sqrt decomposition techniques for efficient range queries.

## Requirements

### Rust Implementation

```rust
pub mod sqrt_decomposition {
    /// Basic sqrt decomposition for range sum
    pub struct SqrtDecomp {
        arr: Vec<i64>,
        blocks: Vec<i64>,
        block_size: usize,
    }

    impl SqrtDecomp {
        pub fn new(arr: Vec<i64>) -> Self;

        /// Point update
        pub fn update(&mut self, idx: usize, val: i64);

        /// Range sum query
        pub fn query(&self, l: usize, r: usize) -> i64;
    }

    /// Sqrt decomposition with lazy propagation
    pub struct SqrtDecompLazy {
        arr: Vec<i64>,
        blocks: Vec<i64>,
        lazy: Vec<i64>,
        block_size: usize,
    }

    impl SqrtDecompLazy {
        pub fn new(arr: Vec<i64>) -> Self;

        /// Range add update
        pub fn range_add(&mut self, l: usize, r: usize, delta: i64);

        /// Point query
        pub fn get(&self, idx: usize) -> i64;

        /// Range sum query
        pub fn range_sum(&self, l: usize, r: usize) -> i64;
    }

    /// Sqrt decomposition for distinct elements count
    pub struct DistinctCount {
        arr: Vec<i32>,
        block_size: usize,
        // Precomputed answers for block ranges
    }

    impl DistinctCount {
        pub fn new(arr: Vec<i32>) -> Self;
        pub fn query(&self, l: usize, r: usize) -> usize;
    }

    /// Query sqrt decomposition (split array into sqrt(q) groups)
    pub fn query_sqrt_decomp<F, G, T>(
        n: usize,
        queries: &[(usize, usize)],
        init: F,
        merge: G,
    ) -> Vec<T>
    where
        F: Fn(usize) -> T,
        G: Fn(&T, &T) -> T;
}

pub mod mos_algorithm {
    /// Mo's algorithm for offline range queries
    pub struct MosAlgorithm {
        arr: Vec<i32>,
        queries: Vec<(usize, usize, usize)>,  // (l, r, idx)
        block_size: usize,
    }

    impl MosAlgorithm {
        pub fn new(arr: Vec<i32>) -> Self;

        /// Add query
        pub fn add_query(&mut self, l: usize, r: usize);

        /// Process all queries, returns answers in original order
        pub fn process<A, R, F>(
            &self,
            add: A,
            remove: R,
            get_answer: F,
        ) -> Vec<i64>
        where
            A: FnMut(i32),        // Add element
            R: FnMut(i32),        // Remove element
            F: FnMut() -> i64;    // Get current answer
    }

    /// Count distinct elements in ranges
    pub fn distinct_queries(arr: &[i32], queries: &[(usize, usize)]) -> Vec<usize>;

    /// Sum of squares in ranges
    pub fn sum_squares_queries(arr: &[i32], queries: &[(usize, usize)]) -> Vec<i64>;

    /// Mode (most frequent element) in ranges
    pub fn mode_queries(arr: &[i32], queries: &[(usize, usize)]) -> Vec<i32>;

    /// XOR of distinct elements in ranges
    pub fn distinct_xor_queries(arr: &[i32], queries: &[(usize, usize)]) -> Vec<i32>;
}

pub mod mos_tree {
    /// Mo's algorithm on trees
    /// Converts tree path queries to array range queries using Euler tour

    pub struct MosTree {
        adj: Vec<Vec<usize>>,
        euler: Vec<usize>,
        first: Vec<usize>,
        last: Vec<usize>,
    }

    impl MosTree {
        pub fn new(adj: Vec<Vec<usize>>, root: usize) -> Self;

        /// Process path queries on tree
        pub fn process_path_queries<A, R, F>(
            &self,
            values: &[i32],
            queries: &[(usize, usize)],  // (u, v) pairs
            add: A,
            remove: R,
            get_answer: F,
        ) -> Vec<i64>
        where
            A: FnMut(i32),
            R: FnMut(i32),
            F: FnMut() -> i64;
    }

    /// Count distinct values on path u-v
    pub fn distinct_on_path(
        adj: &[Vec<usize>],
        values: &[i32],
        queries: &[(usize, usize)],
    ) -> Vec<usize>;
}

pub mod mos_update {
    /// Mo's algorithm with updates
    /// Process range queries with point updates

    pub struct MosWithUpdates {
        arr: Vec<i32>,
        queries: Vec<Query>,
        updates: Vec<(usize, i32)>,  // (idx, new_val)
    }

    pub enum Query {
        Range(usize, usize, usize),  // (l, r, time, idx)
        Update(usize, i32),          // (idx, new_val)
    }

    impl MosWithUpdates {
        pub fn new(arr: Vec<i32>) -> Self;
        pub fn add_range_query(&mut self, l: usize, r: usize);
        pub fn add_update(&mut self, idx: usize, new_val: i32);
        pub fn process(&self) -> Vec<i64>;
    }
}

pub mod heavy_light_sqrt {
    /// Combine HLD with sqrt decomposition for path queries
    pub struct HLDSqrt {
        // Heavy-light decomposition with sqrt blocks on chains
    }

    impl HLDSqrt {
        pub fn new(adj: &[Vec<usize>], root: usize) -> Self;
        pub fn path_sum(&self, u: usize, v: usize) -> i64;
        pub fn update(&mut self, u: usize, val: i64);
    }
}
```

### Python Implementation

```python
from typing import List, Tuple, Callable
import math

class SqrtDecomp:
    def __init__(self, arr: List[int]):
        self.arr = arr
        self.block_size = int(math.sqrt(len(arr))) + 1
        self.blocks = [0] * ((len(arr) // self.block_size) + 1)
        self._build()

    def _build(self) -> None: ...
    def update(self, idx: int, val: int) -> None: ...
    def query(self, l: int, r: int) -> int: ...

class MosAlgorithm:
    def __init__(self, arr: List[int]):
        self.arr = arr
        self.queries: List[Tuple[int, int, int]] = []

    def add_query(self, l: int, r: int) -> None: ...
    def process(self, add, remove, get_answer) -> List[int]: ...

def distinct_queries(arr: List[int], queries: List[Tuple[int, int]]) -> List[int]: ...
def sum_squares_queries(arr: List[int], queries: List[Tuple[int, int]]) -> List[int]: ...
def mode_queries(arr: List[int], queries: List[Tuple[int, int]]) -> List[int]: ...
```

## Test Cases

```rust
#[test]
fn test_sqrt_decomp_sum() {
    let arr = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let mut sd = SqrtDecomp::new(arr);

    assert_eq!(sd.query(0, 9), 55);
    assert_eq!(sd.query(2, 5), 18);  // 3+4+5+6

    sd.update(3, 10);  // Change 4 to 10
    assert_eq!(sd.query(2, 5), 24);  // 3+10+5+6
}

#[test]
fn test_sqrt_decomp_lazy() {
    let arr = vec![1, 2, 3, 4, 5];
    let mut sd = SqrtDecompLazy::new(arr);

    sd.range_add(1, 3, 10);  // [1, 12, 13, 14, 5]
    assert_eq!(sd.get(2), 13);
    assert_eq!(sd.range_sum(0, 4), 45);
}

#[test]
fn test_mos_distinct() {
    let arr = vec![1, 2, 1, 3, 2, 1, 4];
    let queries = vec![(0, 2), (1, 4), (2, 6), (0, 6)];

    let answers = distinct_queries(&arr, &queries);
    assert_eq!(answers[0], 2);  // [1,2,1] -> 2 distinct
    assert_eq!(answers[1], 3);  // [2,1,3,2] -> 3 distinct
    assert_eq!(answers[2], 4);  // [1,3,2,1,4] -> 4 distinct
    assert_eq!(answers[3], 4);  // All -> 4 distinct
}

#[test]
fn test_mos_sum_squares() {
    let arr = vec![1, 2, 3, 2, 1];
    let queries = vec![(0, 4), (1, 3), (0, 2)];

    let answers = sum_squares_queries(&arr, &queries);
    // [1,2,3,2,1]: freq = {1:2, 2:2, 3:1} -> 4+4+1 = 9
    // [2,3,2]: freq = {2:2, 3:1} -> 4+1 = 5
    // [1,2,3]: freq = {1:1, 2:1, 3:1} -> 1+1+1 = 3
    assert_eq!(answers[0], 9);
    assert_eq!(answers[1], 5);
    assert_eq!(answers[2], 3);
}

#[test]
fn test_mos_mode() {
    let arr = vec![1, 2, 2, 3, 2, 1, 1, 1];
    let queries = vec![(0, 3), (2, 5), (4, 7)];

    let answers = mode_queries(&arr, &queries);
    assert_eq!(answers[0], 2);  // [1,2,2,3] -> 2 appears most
    assert_eq!(answers[1], 2);  // [2,3,2,1] -> 2 appears most
    assert_eq!(answers[2], 1);  // [2,1,1,1] -> 1 appears most
}

#[test]
fn test_mos_algorithm() {
    let arr = vec![1, 1, 2, 1, 3];
    let mut mo = MosAlgorithm::new(arr);

    mo.add_query(0, 4);
    mo.add_query(1, 3);
    mo.add_query(2, 4);

    let mut freq = std::collections::HashMap::new();
    let mut distinct = 0i64;

    let answers = mo.process(
        |x| {
            let count = freq.entry(x).or_insert(0);
            if *count == 0 { distinct += 1; }
            *count += 1;
        },
        |x| {
            let count = freq.get_mut(&x).unwrap();
            *count -= 1;
            if *count == 0 { distinct -= 1; }
        },
        || distinct,
    );

    assert_eq!(answers.len(), 3);
}

#[test]
fn test_mos_on_tree() {
    //     0
    //    /|\
    //   1 2 3
    //   |
    //   4
    let adj = vec![
        vec![1, 2, 3],
        vec![0, 4],
        vec![0],
        vec![0],
        vec![1],
    ];
    let values = vec![1, 2, 1, 3, 2];

    let queries = vec![(4, 2), (1, 3), (4, 3)];
    let answers = distinct_on_path(&adj, &values, &queries);

    // Path 4-2: 4->1->0->2, values [2,1,1] -> 2 distinct
    // Path 1-3: 1->0->3, values [2,1,3] -> 3 distinct
    // Path 4-3: 4->1->0->3, values [2,1,1,3] -> 3 distinct
}

#[test]
fn test_mos_with_updates() {
    let arr = vec![1, 2, 3, 4, 5];
    let mut mo = MosWithUpdates::new(arr);

    mo.add_range_query(0, 4);  // Sum: 15, distinct: 5
    mo.add_update(2, 2);       // arr becomes [1,2,2,4,5]
    mo.add_range_query(0, 4);  // Sum: 14, distinct: 4
    mo.add_update(0, 5);       // arr becomes [5,2,2,4,5]
    mo.add_range_query(0, 4);  // Sum: 18, distinct: 3

    let answers = mo.process();
    assert_eq!(answers.len(), 3);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic sqrt decomposition | 15 |
| Sqrt decomp with lazy | 15 |
| Mo's algorithm | 20 |
| Distinct count queries | 15 |
| Mo's on trees | 15 |
| Mo's with updates | 15 |
| Edge cases | 5 |
| **Total** | **100** |
