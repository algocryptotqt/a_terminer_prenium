# Exercise 03: Randomized Algorithms

## Concepts Covered
- **1.7.6.d-l** Monte Carlo, Las Vegas, randomized quicksort
- **1.7.7.d-k** Karger's min-cut, random sampling, Miller-Rabin

## Objective

Implement randomized algorithms and understand their probabilistic guarantees.

## Requirements

### Rust Implementation

```rust
pub mod randomized {
    use rand::Rng;

    /// Randomized quicksort
    pub fn quicksort_random<T: Ord>(arr: &mut [T]);

    /// Randomized selection (find k-th smallest) - O(n) expected
    pub fn quickselect<T: Ord + Clone>(arr: &mut [T], k: usize) -> T;

    /// Randomized median finding
    pub fn median<T: Ord + Clone>(arr: &mut [T]) -> T;

    /// Shuffle array (Fisher-Yates)
    pub fn shuffle<T>(arr: &mut [T]);

    /// Random sampling without replacement
    pub fn random_sample<T: Clone>(arr: &[T], k: usize) -> Vec<T>;

    /// Reservoir sampling (stream of unknown length)
    pub fn reservoir_sample<T: Clone, I: Iterator<Item = T>>(stream: I, k: usize) -> Vec<T>;

    /// Weighted random sampling
    pub fn weighted_sample<T: Clone>(items: &[T], weights: &[f64], k: usize) -> Vec<T>;
}

pub mod monte_carlo {
    /// Estimate π using Monte Carlo
    pub fn estimate_pi(samples: usize) -> f64;

    /// Monte Carlo integration of f over [a, b]
    pub fn integrate<F>(f: F, a: f64, b: f64, samples: usize) -> f64
    where
        F: Fn(f64) -> f64;

    /// Approximate counting (count items satisfying predicate)
    pub fn approximate_count<F>(n: usize, predicate: F, samples: usize) -> f64
    where
        F: Fn(usize) -> bool;

    /// Randomized primality test (Miller-Rabin)
    pub fn miller_rabin(n: u64, iterations: usize) -> bool;

    /// Solovay-Strassen primality test
    pub fn solovay_strassen(n: u64, iterations: usize) -> bool;

    /// Fermat primality test
    pub fn fermat_test(n: u64, iterations: usize) -> bool;
}

pub mod las_vegas {
    /// Las Vegas algorithm: retry until correct
    pub fn las_vegas<F, R>(algorithm: F, verify: impl Fn(&R) -> bool) -> R
    where
        F: Fn() -> R;

    /// Randomized binary search tree (Treap)
    pub struct Treap<K: Ord, V> {
        // Implementation with random priorities
    }

    impl<K: Ord, V> Treap<K, V> {
        pub fn new() -> Self;
        pub fn insert(&mut self, key: K, value: V);
        pub fn search(&self, key: &K) -> Option<&V>;
        pub fn delete(&mut self, key: &K) -> Option<V>;
    }

    /// Skip list
    pub struct SkipList<K: Ord, V> {
        // Multi-level linked list with random levels
    }

    impl<K: Ord, V> SkipList<K, V> {
        pub fn new() -> Self;
        pub fn insert(&mut self, key: K, value: V);
        pub fn search(&self, key: &K) -> Option<&V>;
        pub fn delete(&mut self, key: &K) -> Option<V>;
    }
}

pub mod graph_randomized {
    /// Karger's min-cut algorithm
    pub fn karger_min_cut(adj: &[Vec<usize>]) -> usize;

    /// Karger-Stein (faster variant)
    pub fn karger_stein_min_cut(adj: &[Vec<usize>]) -> usize;

    /// Randomized max-cut (2-approximation)
    pub fn random_max_cut(adj: &[Vec<(usize, i64)>]) -> (i64, Vec<bool>);

    /// Random walk on graph
    pub fn random_walk(adj: &[Vec<usize>], start: usize, steps: usize) -> Vec<usize>;

    /// PageRank using random walks
    pub fn pagerank_random_walks(adj: &[Vec<usize>], walks: usize, steps: usize) -> Vec<f64>;

    /// Randomized Kruskal for spanning tree
    pub fn random_spanning_tree(adj: &[Vec<usize>]) -> Vec<(usize, usize)>;
}

pub mod hashing {
    /// Bloom filter
    pub struct BloomFilter {
        bits: Vec<bool>,
        hash_count: usize,
    }

    impl BloomFilter {
        pub fn new(size: usize, hash_count: usize) -> Self;
        pub fn insert(&mut self, item: &[u8]);
        pub fn contains(&self, item: &[u8]) -> bool;
        pub fn false_positive_rate(&self) -> f64;
    }

    /// Count-Min Sketch
    pub struct CountMinSketch {
        table: Vec<Vec<u64>>,
        depth: usize,
        width: usize,
    }

    impl CountMinSketch {
        pub fn new(depth: usize, width: usize) -> Self;
        pub fn add(&mut self, item: &[u8], count: u64);
        pub fn estimate(&self, item: &[u8]) -> u64;
    }

    /// HyperLogLog for cardinality estimation
    pub struct HyperLogLog {
        registers: Vec<u8>,
        precision: usize,
    }

    impl HyperLogLog {
        pub fn new(precision: usize) -> Self;
        pub fn add(&mut self, item: &[u8]);
        pub fn count(&self) -> f64;
        pub fn merge(&mut self, other: &Self);
    }

    /// MinHash for similarity estimation
    pub struct MinHash {
        signatures: Vec<u64>,
        num_hashes: usize,
    }

    impl MinHash {
        pub fn new(num_hashes: usize) -> Self;
        pub fn add(&mut self, item: &[u8]);
        pub fn similarity(&self, other: &Self) -> f64;  // Jaccard similarity estimate
    }
}
```

### Python Implementation

```python
from typing import List, Tuple, Callable, TypeVar, Iterator
import random

T = TypeVar('T')

def quicksort_random(arr: List) -> List: ...
def quickselect(arr: List, k: int) -> any: ...
def shuffle(arr: List) -> List: ...
def reservoir_sample(stream: Iterator, k: int) -> List: ...

def estimate_pi(samples: int) -> float: ...
def integrate(f: Callable[[float], float], a: float, b: float, samples: int) -> float: ...
def miller_rabin(n: int, iterations: int = 10) -> bool: ...

def karger_min_cut(adj: List[List[int]]) -> int: ...
def random_max_cut(adj: List[List[Tuple[int, int]]]) -> Tuple[int, List[bool]]: ...

class BloomFilter:
    def __init__(self, size: int, hash_count: int): ...
    def insert(self, item: bytes) -> None: ...
    def contains(self, item: bytes) -> bool: ...

class HyperLogLog:
    def __init__(self, precision: int): ...
    def add(self, item: bytes) -> None: ...
    def count(self) -> float: ...
```

## Test Cases

```rust
#[test]
fn test_quickselect() {
    let mut arr = vec![3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5];
    assert_eq!(quickselect(&mut arr.clone(), 0), 1);  // Min
    assert_eq!(quickselect(&mut arr.clone(), 5), 4);  // Median-ish
}

#[test]
fn test_shuffle() {
    let mut arr = vec![1, 2, 3, 4, 5];
    let original = arr.clone();
    shuffle(&mut arr);

    // Same elements
    arr.sort();
    assert_eq!(arr, original);
}

#[test]
fn test_reservoir_sampling() {
    let stream = 0..1000;
    let sample = reservoir_sample(stream, 10);
    assert_eq!(sample.len(), 10);

    // All elements should be from [0, 1000)
    for &x in &sample {
        assert!(x < 1000);
    }
}

#[test]
fn test_estimate_pi() {
    let pi = estimate_pi(100000);
    assert!((pi - std::f64::consts::PI).abs() < 0.1);
}

#[test]
fn test_integrate() {
    // Integrate x^2 from 0 to 1 = 1/3
    let result = integrate(|x| x * x, 0.0, 1.0, 100000);
    assert!((result - 1.0 / 3.0).abs() < 0.01);
}

#[test]
fn test_miller_rabin() {
    assert!(miller_rabin(2, 10));
    assert!(miller_rabin(17, 10));
    assert!(miller_rabin(1_000_000_007, 10));
    assert!(!miller_rabin(15, 10));
    assert!(!miller_rabin(561, 10));  // Carmichael number
}

#[test]
fn test_karger_min_cut() {
    // Square graph: min cut = 2
    // 0 - 1
    // |   |
    // 3 - 2
    let adj = vec![
        vec![1, 3],
        vec![0, 2],
        vec![1, 3],
        vec![0, 2],
    ];

    // Run multiple times for high probability
    let mut min = usize::MAX;
    for _ in 0..100 {
        min = min.min(karger_min_cut(&adj));
    }
    assert_eq!(min, 2);
}

#[test]
fn test_bloom_filter() {
    let mut bf = BloomFilter::new(1000, 5);

    bf.insert(b"hello");
    bf.insert(b"world");

    assert!(bf.contains(b"hello"));
    assert!(bf.contains(b"world"));

    // May have false positives, but shouldn't have false negatives
    // Check many non-inserted items
    let mut false_positives = 0;
    for i in 0..100 {
        if bf.contains(format!("test{}", i).as_bytes()) {
            false_positives += 1;
        }
    }
    // Should be relatively low
    assert!(false_positives < 20);
}

#[test]
fn test_count_min_sketch() {
    let mut cms = CountMinSketch::new(4, 1000);

    for _ in 0..100 {
        cms.add(b"apple", 1);
    }
    for _ in 0..50 {
        cms.add(b"banana", 1);
    }

    let apple_count = cms.estimate(b"apple");
    let banana_count = cms.estimate(b"banana");

    // Should be at least actual count (can overestimate)
    assert!(apple_count >= 100);
    assert!(banana_count >= 50);
}

#[test]
fn test_hyperloglog() {
    let mut hll = HyperLogLog::new(10);  // ~1% error

    for i in 0..10000 {
        hll.add(&i.to_le_bytes());
    }

    let estimate = hll.count();
    assert!((estimate - 10000.0).abs() < 500.0);  // Within 5%
}

#[test]
fn test_minhash() {
    let mut mh1 = MinHash::new(100);
    let mut mh2 = MinHash::new(100);

    // Same elements
    for i in 0..100 {
        mh1.add(&i.to_le_bytes());
        mh2.add(&i.to_le_bytes());
    }

    assert!(mh1.similarity(&mh2) > 0.9);  // Should be ~1.0

    // Different elements
    let mut mh3 = MinHash::new(100);
    for i in 100..200 {
        mh3.add(&i.to_le_bytes());
    }

    assert!(mh1.similarity(&mh3) < 0.2);  // Should be ~0.0
}

#[test]
fn test_skip_list() {
    let mut sl = SkipList::new();

    for i in 0..100 {
        sl.insert(i, i * 2);
    }

    assert_eq!(sl.search(&50), Some(&100));
    assert_eq!(sl.search(&200), None);

    sl.delete(&50);
    assert_eq!(sl.search(&50), None);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Randomized quicksort/select | 15 |
| Sampling algorithms | 10 |
| Monte Carlo integration | 10 |
| Primality testing | 15 |
| Karger's min-cut | 15 |
| Bloom filter | 10 |
| HyperLogLog | 15 |
| Skip list / Treap | 5 |
| Edge cases | 5 |
| **Total** | **100** |
