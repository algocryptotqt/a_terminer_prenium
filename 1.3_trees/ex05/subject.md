# Exercise 05: Heaps & Priority Queues

## Concepts Covered
- **1.3.6.h-o** Binary heap operations, heapify, heap sort
- **1.3.7.d-k** D-ary heaps, Fibonacci heaps, pairing heaps, binomial heaps

## Objective

Implement various heap data structures with different performance characteristics.

## Requirements

### Rust Implementation

```rust
pub mod heaps {
    use std::cmp::Ordering;

    /// Binary Min-Heap
    pub struct BinaryHeap<T: Ord> {
        data: Vec<T>,
    }

    impl<T: Ord> BinaryHeap<T> {
        pub fn new() -> Self;
        pub fn with_capacity(capacity: usize) -> Self;

        /// Push element - O(log n)
        pub fn push(&mut self, item: T);

        /// Pop minimum - O(log n)
        pub fn pop(&mut self) -> Option<T>;

        /// Peek minimum - O(1)
        pub fn peek(&self) -> Option<&T>;

        /// Build heap from array - O(n)
        pub fn heapify(data: Vec<T>) -> Self;

        /// Decrease key at index (for min-heap)
        pub fn decrease_key(&mut self, index: usize, new_val: T);

        /// Sift up
        fn sift_up(&mut self, index: usize);

        /// Sift down
        fn sift_down(&mut self, index: usize);

        pub fn len(&self) -> usize;
        pub fn is_empty(&self) -> bool;
    }

    /// Heap Sort - O(n log n) in-place
    pub fn heap_sort<T: Ord>(arr: &mut [T]);

    /// D-ary Heap (generalization of binary heap)
    pub struct DaryHeap<T: Ord, const D: usize> {
        data: Vec<T>,
    }

    impl<T: Ord, const D: usize> DaryHeap<T, D> {
        pub fn new() -> Self;
        pub fn push(&mut self, item: T);
        pub fn pop(&mut self) -> Option<T>;
        pub fn peek(&self) -> Option<&T>;

        // D-ary heap has faster decrease_key but slower pop
        pub fn decrease_key(&mut self, index: usize, new_val: T);
    }

    /// Indexed Priority Queue (supports decrease-key by ID)
    pub struct IndexedPQ<T: Ord> {
        heap: Vec<usize>,        // heap[i] = id at position i
        position: Vec<usize>,    // position[id] = position in heap
        keys: Vec<Option<T>>,    // keys[id] = priority
    }

    impl<T: Ord> IndexedPQ<T> {
        pub fn new(max_size: usize) -> Self;
        pub fn insert(&mut self, id: usize, key: T);
        pub fn pop_min(&mut self) -> Option<(usize, T)>;
        pub fn decrease_key(&mut self, id: usize, new_key: T);
        pub fn contains(&self, id: usize) -> bool;
        pub fn peek_min(&self) -> Option<(usize, &T)>;
    }

    /// Binomial Heap
    pub struct BinomialNode<T: Ord> {
        key: T,
        degree: usize,
        children: Vec<Box<BinomialNode<T>>>,
    }

    pub struct BinomialHeap<T: Ord> {
        trees: Vec<Option<Box<BinomialNode<T>>>>,
        min_idx: Option<usize>,
        size: usize,
    }

    impl<T: Ord> BinomialHeap<T> {
        pub fn new() -> Self;
        pub fn push(&mut self, item: T);           // O(log n) amortized O(1)
        pub fn pop(&mut self) -> Option<T>;        // O(log n)
        pub fn peek(&self) -> Option<&T>;          // O(1)
        pub fn merge(&mut self, other: Self);      // O(log n)
    }

    /// Fibonacci Heap - optimal for decrease_key heavy workloads
    pub struct FibNode<T: Ord> {
        key: T,
        degree: usize,
        marked: bool,
        parent: Option<*mut FibNode<T>>,
        child: Option<Box<FibNode<T>>>,
        siblings: Vec<Box<FibNode<T>>>,
    }

    pub struct FibonacciHeap<T: Ord> {
        min: Option<Box<FibNode<T>>>,
        roots: Vec<Box<FibNode<T>>>,
        size: usize,
    }

    impl<T: Ord> FibonacciHeap<T> {
        pub fn new() -> Self;
        pub fn push(&mut self, item: T) -> *mut FibNode<T>;  // O(1)
        pub fn pop(&mut self) -> Option<T>;                   // O(log n) amortized
        pub fn peek(&self) -> Option<&T>;                     // O(1)
        pub fn decrease_key(&mut self, node: *mut FibNode<T>, new_key: T);  // O(1) amortized
        pub fn merge(&mut self, other: Self);                 // O(1)

        // Internal operations
        fn consolidate(&mut self);
        fn cut(&mut self, node: *mut FibNode<T>);
        fn cascading_cut(&mut self, node: *mut FibNode<T>);
    }

    /// Pairing Heap - simpler than Fibonacci, good practical performance
    pub struct PairingNode<T: Ord> {
        key: T,
        children: Vec<Box<PairingNode<T>>>,
    }

    pub struct PairingHeap<T: Ord> {
        root: Option<Box<PairingNode<T>>>,
        size: usize,
    }

    impl<T: Ord> PairingHeap<T> {
        pub fn new() -> Self;
        pub fn push(&mut self, item: T);           // O(1)
        pub fn pop(&mut self) -> Option<T>;        // O(log n) amortized
        pub fn peek(&self) -> Option<&T>;          // O(1)
        pub fn merge(&mut self, other: Self);      // O(1)

        fn merge_pairs(nodes: Vec<Box<PairingNode<T>>>) -> Option<Box<PairingNode<T>>>;
    }
}

/// Applications
pub mod heap_applications {
    /// K largest/smallest elements
    pub fn k_largest<T: Ord + Clone>(arr: &[T], k: usize) -> Vec<T>;
    pub fn k_smallest<T: Ord + Clone>(arr: &[T], k: usize) -> Vec<T>;

    /// Median maintenance with two heaps
    pub struct MedianFinder {
        lo: std::collections::BinaryHeap<i32>,        // max-heap for lower half
        hi: std::collections::BinaryHeap<std::cmp::Reverse<i32>>,  // min-heap for upper half
    }

    impl MedianFinder {
        pub fn new() -> Self;
        pub fn add(&mut self, num: i32);
        pub fn median(&self) -> f64;
    }

    /// Merge K sorted arrays
    pub fn merge_k_sorted<T: Ord + Clone>(arrays: Vec<Vec<T>>) -> Vec<T>;

    /// Dijkstra with different heaps - compare performance
    pub fn dijkstra_binary(adj: &[Vec<(usize, i64)>], src: usize) -> Vec<i64>;
    pub fn dijkstra_fibonacci(adj: &[Vec<(usize, i64)>], src: usize) -> Vec<i64>;
}
```

### Python Implementation

```python
from typing import TypeVar, Generic, Optional, List, Tuple
import heapq

T = TypeVar('T')

class BinaryHeap(Generic[T]):
    def __init__(self):
        self.data: List[T] = []

    def push(self, item: T) -> None: ...
    def pop(self) -> Optional[T]: ...
    def peek(self) -> Optional[T]: ...

    @staticmethod
    def heapify(data: List[T]) -> 'BinaryHeap[T]': ...

def heap_sort(arr: List[T]) -> List[T]: ...

class IndexedPQ(Generic[T]):
    def __init__(self, max_size: int):
        self.heap: List[int] = []
        self.position: List[int] = [-1] * max_size
        self.keys: List[Optional[T]] = [None] * max_size

    def insert(self, id: int, key: T) -> None: ...
    def pop_min(self) -> Optional[Tuple[int, T]]: ...
    def decrease_key(self, id: int, new_key: T) -> None: ...

class MedianFinder:
    def __init__(self):
        self.lo: List[int] = []  # max-heap (negate values)
        self.hi: List[int] = []  # min-heap

    def add(self, num: int) -> None: ...
    def median(self) -> float: ...

def merge_k_sorted(arrays: List[List[T]]) -> List[T]: ...
def k_largest(arr: List[T], k: int) -> List[T]: ...
```

## Test Cases

```rust
#[test]
fn test_binary_heap() {
    let mut heap = BinaryHeap::new();
    heap.push(5);
    heap.push(3);
    heap.push(7);
    heap.push(1);

    assert_eq!(heap.pop(), Some(1));
    assert_eq!(heap.pop(), Some(3));
    assert_eq!(heap.pop(), Some(5));
    assert_eq!(heap.pop(), Some(7));
}

#[test]
fn test_heapify() {
    let data = vec![5, 3, 8, 1, 9, 2, 7];
    let mut heap = BinaryHeap::heapify(data);

    let mut sorted = Vec::new();
    while let Some(x) = heap.pop() {
        sorted.push(x);
    }
    assert_eq!(sorted, vec![1, 2, 3, 5, 7, 8, 9]);
}

#[test]
fn test_heap_sort() {
    let mut arr = vec![64, 34, 25, 12, 22, 11, 90];
    heap_sort(&mut arr);
    assert_eq!(arr, vec![11, 12, 22, 25, 34, 64, 90]);
}

#[test]
fn test_indexed_pq() {
    let mut pq = IndexedPQ::new(10);
    pq.insert(0, 50);
    pq.insert(1, 30);
    pq.insert(2, 40);

    assert_eq!(pq.pop_min(), Some((1, 30)));

    pq.decrease_key(0, 20);
    assert_eq!(pq.pop_min(), Some((0, 20)));
}

#[test]
fn test_median_finder() {
    let mut finder = MedianFinder::new();
    finder.add(1);
    assert_eq!(finder.median(), 1.0);

    finder.add(2);
    assert_eq!(finder.median(), 1.5);

    finder.add(3);
    assert_eq!(finder.median(), 2.0);
}

#[test]
fn test_merge_k_sorted() {
    let arrays = vec![
        vec![1, 4, 7],
        vec![2, 5, 8],
        vec![3, 6, 9],
    ];
    assert_eq!(merge_k_sorted(arrays), vec![1, 2, 3, 4, 5, 6, 7, 8, 9]);
}

#[test]
fn test_fibonacci_heap() {
    let mut fib = FibonacciHeap::new();
    let n1 = fib.push(10);
    let n2 = fib.push(5);
    let _n3 = fib.push(15);

    fib.decrease_key(n1, 2);
    assert_eq!(fib.pop(), Some(2));
    assert_eq!(fib.pop(), Some(5));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Binary heap operations | 15 |
| Heapify and heap sort | 15 |
| Indexed priority queue | 15 |
| D-ary heap | 10 |
| Binomial/Fibonacci/Pairing heap | 25 |
| Applications (median, merge-k) | 15 |
| Edge cases | 5 |
| **Total** | **100** |
