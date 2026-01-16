# Exercise 00: Hash Table Implementation

## Concepts Covered
- **1.2.5.k** Chaining variants
- **1.2.6.k-l** Rehashing, Load factor
- **1.2.7.i-k** Hopscotch, Linear, Extendible hashing
- **1.2.8.l-n** Rehashing, Iterator, Statistics
- **1.2.9.k-n** Count-Min Sketch, HyperLogLog

## Objective

Implement a complete hash table with multiple collision resolution strategies and probabilistic data structures.

## Requirements

### Rust Implementation

```rust
pub mod hash_table {
    use std::hash::{Hash, Hasher, BuildHasher};
    use std::collections::hash_map::RandomState;

    /// Hash table with separate chaining
    pub struct ChainedHashTable<K, V, S = RandomState> {
        buckets: Vec<Vec<(K, V)>>,
        len: usize,
        hash_builder: S,
    }

    impl<K: Hash + Eq, V> ChainedHashTable<K, V> {
        pub fn new() -> Self;
        pub fn with_capacity(capacity: usize) -> Self;
        pub fn insert(&mut self, key: K, value: V) -> Option<V>;
        pub fn get(&self, key: &K) -> Option<&V>;
        pub fn get_mut(&mut self, key: &K) -> Option<&mut V>;
        pub fn remove(&mut self, key: &K) -> Option<V>;
        pub fn contains_key(&self, key: &K) -> bool;
        pub fn len(&self) -> usize;
        pub fn is_empty(&self) -> bool;
        pub fn load_factor(&self) -> f64;
        pub fn iter(&self) -> impl Iterator<Item = (&K, &V)>;
    }

    /// Hash table with open addressing (linear probing)
    pub struct LinearProbingHashTable<K, V> {
        slots: Vec<Option<(K, V)>>,
        tombstones: Vec<bool>,
        len: usize,
    }

    impl<K: Hash + Eq, V> LinearProbingHashTable<K, V> {
        pub fn new() -> Self;
        pub fn insert(&mut self, key: K, value: V) -> Option<V>;
        pub fn get(&self, key: &K) -> Option<&V>;
        pub fn remove(&mut self, key: &K) -> Option<V>;
        pub fn len(&self) -> usize;
    }

    /// Hash table with quadratic probing
    pub struct QuadraticProbingHashTable<K, V> {
        // Similar interface
    }

    /// Hash table with double hashing
    pub struct DoubleHashingTable<K, V> {
        // Similar interface
    }

    /// Robin Hood hashing
    pub struct RobinHoodHashTable<K, V> {
        slots: Vec<Option<(K, V, usize)>>,  // (key, value, probe_distance)
        len: usize,
    }

    impl<K: Hash + Eq, V> RobinHoodHashTable<K, V> {
        pub fn new() -> Self;
        pub fn insert(&mut self, key: K, value: V) -> Option<V>;
        pub fn get(&self, key: &K) -> Option<&V>;
        pub fn remove(&mut self, key: &K) -> Option<V>;
        pub fn average_probe_distance(&self) -> f64;
    }

    /// Cuckoo hashing with two hash functions
    pub struct CuckooHashTable<K, V> {
        table1: Vec<Option<(K, V)>>,
        table2: Vec<Option<(K, V)>>,
        len: usize,
    }

    impl<K: Hash + Eq + Clone, V: Clone> CuckooHashTable<K, V> {
        pub fn new(capacity: usize) -> Self;
        pub fn insert(&mut self, key: K, value: V) -> Result<Option<V>, (K, V)>;
        pub fn get(&self, key: &K) -> Option<&V>;
        pub fn remove(&mut self, key: &K) -> Option<V>;
    }

    // Probabilistic Data Structures

    /// Bloom Filter
    pub struct BloomFilter {
        bits: Vec<bool>,
        num_hashes: usize,
    }

    impl BloomFilter {
        /// Create with target capacity and false positive rate
        pub fn with_fp_rate(capacity: usize, fp_rate: f64) -> Self;
        pub fn insert<T: Hash>(&mut self, item: &T);
        pub fn contains<T: Hash>(&self, item: &T) -> bool;
        pub fn estimated_fp_rate(&self) -> f64;
    }

    /// Count-Min Sketch
    pub struct CountMinSketch {
        table: Vec<Vec<u64>>,
        width: usize,
        depth: usize,
    }

    impl CountMinSketch {
        pub fn new(width: usize, depth: usize) -> Self;
        pub fn with_accuracy(epsilon: f64, delta: f64) -> Self;
        pub fn add<T: Hash>(&mut self, item: &T, count: u64);
        pub fn estimate<T: Hash>(&self, item: &T) -> u64;
    }

    /// HyperLogLog for cardinality estimation
    pub struct HyperLogLog {
        registers: Vec<u8>,
        precision: usize,  // Number of bits for bucket index
    }

    impl HyperLogLog {
        pub fn new(precision: usize) -> Self;
        pub fn add<T: Hash>(&mut self, item: &T);
        pub fn count(&self) -> f64;
        pub fn merge(&mut self, other: &Self);
    }
}
```

### Python Implementation

```python
from typing import TypeVar, Generic, Iterator, Callable
from dataclasses import dataclass

K = TypeVar("K")
V = TypeVar("V")

class ChainedHashTable(Generic[K, V]):
    def __init__(self, capacity: int = 16) -> None: ...
    def insert(self, key: K, value: V) -> V | None: ...
    def get(self, key: K) -> V | None: ...
    def remove(self, key: K) -> V | None: ...
    def __contains__(self, key: K) -> bool: ...
    def __len__(self) -> int: ...
    def load_factor(self) -> float: ...
    def __iter__(self) -> Iterator[tuple[K, V]]: ...

class LinearProbingHashTable(Generic[K, V]):
    # Similar interface
    ...

class RobinHoodHashTable(Generic[K, V]):
    # Similar interface
    def average_probe_distance(self) -> float: ...

class CuckooHashTable(Generic[K, V]):
    def insert(self, key: K, value: V) -> V | None: ...
    # May raise exception on too many evictions

class BloomFilter:
    def __init__(self, capacity: int, fp_rate: float) -> None: ...
    def insert(self, item: object) -> None: ...
    def __contains__(self, item: object) -> bool: ...

class CountMinSketch:
    def __init__(self, width: int, depth: int) -> None: ...
    def add(self, item: object, count: int = 1) -> None: ...
    def estimate(self, item: object) -> int: ...

class HyperLogLog:
    def __init__(self, precision: int = 14) -> None: ...
    def add(self, item: object) -> None: ...
    def count(self) -> float: ...
    def merge(self, other: "HyperLogLog") -> None: ...
```

## Test Cases

```rust
#[test]
fn test_chained_hash_table() {
    let mut table: ChainedHashTable<String, i32> = ChainedHashTable::new();

    table.insert("one".into(), 1);
    table.insert("two".into(), 2);
    table.insert("three".into(), 3);

    assert_eq!(table.get(&"two".into()), Some(&2));
    assert_eq!(table.len(), 3);

    assert_eq!(table.remove(&"two".into()), Some(2));
    assert_eq!(table.get(&"two".into()), None);
}

#[test]
fn test_robin_hood() {
    let mut table: RobinHoodHashTable<i32, i32> = RobinHoodHashTable::new();

    for i in 0..1000 {
        table.insert(i, i * 2);
    }

    for i in 0..1000 {
        assert_eq!(table.get(&i), Some(&(i * 2)));
    }

    // Robin Hood should have low variance in probe distances
    assert!(table.average_probe_distance() < 3.0);
}

#[test]
fn test_cuckoo_hashing() {
    let mut table: CuckooHashTable<i32, i32> = CuckooHashTable::new(64);

    for i in 0..30 {
        table.insert(i, i * 10).unwrap();
    }

    for i in 0..30 {
        assert_eq!(table.get(&i), Some(&(i * 10)));
    }
}

#[test]
fn test_bloom_filter() {
    let mut bloom = BloomFilter::with_fp_rate(1000, 0.01);

    for i in 0..1000 {
        bloom.insert(&i);
    }

    // All inserted items should be found
    for i in 0..1000 {
        assert!(bloom.contains(&i));
    }

    // Count false positives
    let mut fp = 0;
    for i in 1000..2000 {
        if bloom.contains(&i) {
            fp += 1;
        }
    }

    // Should be close to 1% FP rate
    assert!(fp < 50, "Too many false positives: {}", fp);
}

#[test]
fn test_count_min_sketch() {
    let mut cms = CountMinSketch::with_accuracy(0.01, 0.01);

    cms.add(&"apple", 10);
    cms.add(&"banana", 5);
    cms.add(&"apple", 5);

    assert!(cms.estimate(&"apple") >= 15);
    assert!(cms.estimate(&"banana") >= 5);
    assert!(cms.estimate(&"cherry") < 5);  // May have some error
}

#[test]
fn test_hyperloglog() {
    let mut hll = HyperLogLog::new(14);

    // Add 10000 unique items
    for i in 0..10000 {
        hll.add(&i);
    }

    let estimate = hll.count();
    // Should be within 2% of true count
    assert!((estimate - 10000.0).abs() / 10000.0 < 0.02);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Chained hash table | 15 |
| Linear probing | 10 |
| Robin Hood hashing | 15 |
| Cuckoo hashing | 15 |
| Bloom Filter | 15 |
| Count-Min Sketch | 15 |
| HyperLogLog | 15 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `hash_table.py`
