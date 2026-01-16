# Exercise 01: Property-Based Testing

## Concepts Covered
- **1.8.2.d-l** Property-based testing, generators, shrinking
- **1.8.3.d-k** QuickCheck-style testing, invariants

## Objective

Implement property-based testing frameworks and understand their principles.

## Requirements

### Rust Implementation

```rust
pub mod generators {
    use rand::Rng;

    /// Base generator trait
    pub trait Generator<T> {
        fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> T;
        fn shrink(&self, value: T) -> Box<dyn Iterator<Item = T>>;
    }

    /// Integer generator
    pub struct IntGen {
        pub min: i64,
        pub max: i64,
    }

    impl Generator<i64> for IntGen {
        fn generate<R: Rng>(&self, rng: &mut R, _size: usize) -> i64;
        fn shrink(&self, value: i64) -> Box<dyn Iterator<Item = i64>>;
    }

    /// Vector generator
    pub struct VecGen<G> {
        pub element_gen: G,
        pub max_len: usize,
    }

    impl<T, G: Generator<T>> Generator<Vec<T>> for VecGen<G> {
        fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> Vec<T>;
        fn shrink(&self, value: Vec<T>) -> Box<dyn Iterator<Item = Vec<T>>>;
    }

    /// String generator
    pub struct StringGen {
        pub charset: String,
        pub max_len: usize,
    }

    impl Generator<String> for StringGen {
        fn generate<R: Rng>(&self, rng: &mut R, size: usize) -> String;
        fn shrink(&self, value: String) -> Box<dyn Iterator<Item = String>>;
    }

    /// Arbitrary trait for types that can be generated
    pub trait Arbitrary: Sized {
        fn arbitrary<R: Rng>(rng: &mut R, size: usize) -> Self;
        fn shrink(self) -> Box<dyn Iterator<Item = Self>>;
    }

    /// Composite generators
    pub fn one_of<T, R: Rng>(gens: &[Box<dyn Generator<T>>], rng: &mut R, size: usize) -> T;
    pub fn tuple2<A, B, GA, GB>(g1: GA, g2: GB) -> impl Generator<(A, B)>
    where
        GA: Generator<A>,
        GB: Generator<B>;
}

pub mod properties {
    use super::generators::Generator;

    /// Property test runner
    pub struct PropTest {
        num_tests: usize,
        max_shrinks: usize,
        seed: Option<u64>,
    }

    impl PropTest {
        pub fn new() -> Self;
        pub fn num_tests(self, n: usize) -> Self;
        pub fn max_shrinks(self, n: usize) -> Self;
        pub fn seed(self, s: u64) -> Self;

        /// Run property test
        pub fn for_all<T, G, F>(self, gen: G, prop: F) -> TestResult
        where
            G: Generator<T>,
            T: std::fmt::Debug + Clone,
            F: Fn(T) -> bool;

        /// Run with custom message
        pub fn for_all_labeled<T, G, F>(self, label: &str, gen: G, prop: F) -> TestResult
        where
            G: Generator<T>,
            T: std::fmt::Debug + Clone,
            F: Fn(T) -> bool;
    }

    #[derive(Debug)]
    pub enum TestResult {
        Passed { num_tests: usize },
        Failed { counterexample: String, shrunk_to: String },
        GaveUp { reason: String },
    }

    /// Common properties
    pub fn is_sorted<T: Ord>(arr: &[T]) -> bool;
    pub fn is_permutation<T: Ord + Clone>(a: &[T], b: &[T]) -> bool;
    pub fn is_idempotent<T: Eq + Clone, F: Fn(T) -> T>(f: F, x: T) -> bool;
    pub fn is_commutative<T: Eq + Clone, F: Fn(T, T) -> T>(f: F, a: T, b: T) -> bool;
    pub fn is_associative<T: Eq + Clone, F: Fn(T, T) -> T>(f: F, a: T, b: T, c: T) -> bool;
}

pub mod algorithms_props {
    use super::*;

    /// Sorting property tests
    pub fn test_sort_is_sorted<F>(sort: F) -> TestResult
    where
        F: Fn(&mut [i32]);

    pub fn test_sort_preserves_elements<F>(sort: F) -> TestResult
    where
        F: Fn(&mut [i32]);

    pub fn test_sort_length_unchanged<F>(sort: F) -> TestResult
    where
        F: Fn(&mut [i32]);

    /// Search property tests
    pub fn test_binary_search_finds_element() -> TestResult;
    pub fn test_binary_search_not_found() -> TestResult;

    /// Data structure property tests
    pub fn test_hashmap_insert_get() -> TestResult;
    pub fn test_bst_invariant() -> TestResult;

    /// Graph algorithm properties
    pub fn test_dijkstra_non_negative_distances() -> TestResult;
    pub fn test_bfs_shortest_unweighted() -> TestResult;
}

pub mod shrinking {
    /// Shrink integer towards zero
    pub fn shrink_int(n: i64) -> impl Iterator<Item = i64>;

    /// Shrink vector by removing elements
    pub fn shrink_vec<T: Clone>(v: Vec<T>) -> impl Iterator<Item = Vec<T>>;

    /// Shrink string
    pub fn shrink_string(s: String) -> impl Iterator<Item = String>;

    /// Generic shrink with predicate
    pub fn shrink_with<T, I, F>(value: T, shrinker: F, predicate: impl Fn(&T) -> bool) -> T
    where
        I: Iterator<Item = T>,
        F: Fn(T) -> I;

    /// Binary search shrinking (more efficient)
    pub fn binary_shrink<T, F>(value: T, predicate: F) -> T
    where
        F: Fn(&T) -> bool;
}

pub mod stateful {
    /// Model-based testing
    pub trait Model<S, A> {
        fn initial_state(&self) -> S;
        fn next_state(&self, state: &S, action: &A) -> S;
        fn precondition(&self, state: &S, action: &A) -> bool;
        fn postcondition(&self, state: &S, action: &A, result: &impl PartialEq) -> bool;
    }

    /// State machine testing
    pub fn test_state_machine<M, S, A, I>(model: M, impl_: I, actions: Vec<A>) -> bool
    where
        M: Model<S, A>;
}
```

### Python Implementation

```python
from typing import TypeVar, Generic, Callable, List, Iterator, Any
import random

T = TypeVar('T')

class Generator(Generic[T]):
    def generate(self, rng: random.Random, size: int) -> T: ...
    def shrink(self, value: T) -> Iterator[T]: ...

class IntGen(Generator[int]):
    def __init__(self, min_val: int, max_val: int): ...

class VecGen(Generator[List[T]]):
    def __init__(self, element_gen: Generator[T], max_len: int): ...

class StringGen(Generator[str]):
    def __init__(self, charset: str, max_len: int): ...

class PropTest:
    def __init__(self, num_tests: int = 100):
        self.num_tests = num_tests

    def for_all(self, gen: Generator[T], prop: Callable[[T], bool]) -> 'TestResult': ...

class TestResult:
    passed: bool
    counterexample: Any = None
    shrunk_to: Any = None

def is_sorted(arr: List) -> bool: ...
def is_permutation(a: List, b: List) -> bool: ...

def shrink_int(n: int) -> Iterator[int]: ...
def shrink_list(lst: List[T]) -> Iterator[List[T]]: ...
```

## Test Cases

```rust
#[test]
fn test_int_generator() {
    let gen = IntGen { min: 0, max: 100 };
    let mut rng = rand::thread_rng();

    for _ in 0..100 {
        let val = gen.generate(&mut rng, 10);
        assert!(val >= 0 && val <= 100);
    }
}

#[test]
fn test_vec_generator() {
    let gen = VecGen {
        element_gen: IntGen { min: 0, max: 10 },
        max_len: 20,
    };
    let mut rng = rand::thread_rng();

    for _ in 0..100 {
        let vec = gen.generate(&mut rng, 10);
        assert!(vec.len() <= 20);
        for &x in &vec {
            assert!(x >= 0 && x <= 10);
        }
    }
}

#[test]
fn test_shrink_int() {
    let shrinks: Vec<_> = shrink_int(100).take(10).collect();
    // Should shrink towards 0
    assert!(shrinks.contains(&50));
    assert!(shrinks.contains(&0));
}

#[test]
fn test_shrink_vec() {
    let vec = vec![1, 2, 3, 4, 5];
    let shrinks: Vec<_> = shrink_vec(vec.clone()).take(10).collect();

    // Should contain smaller vectors
    assert!(shrinks.iter().any(|v| v.len() < 5));
}

#[test]
fn test_sort_property() {
    let result = PropTest::new()
        .num_tests(1000)
        .for_all(
            VecGen { element_gen: IntGen { min: -100, max: 100 }, max_len: 100 },
            |mut v| {
                v.sort();
                is_sorted(&v)
            }
        );

    assert!(matches!(result, TestResult::Passed { .. }));
}

#[test]
fn test_sort_permutation_property() {
    let result = PropTest::new()
        .num_tests(1000)
        .for_all(
            VecGen { element_gen: IntGen { min: -100, max: 100 }, max_len: 100 },
            |v| {
                let original = v.clone();
                let mut sorted = v.clone();
                sorted.sort();
                is_permutation(&original, &sorted)
            }
        );

    assert!(matches!(result, TestResult::Passed { .. }));
}

#[test]
fn test_reverse_involution() {
    // reverse(reverse(x)) == x
    let result = PropTest::new()
        .for_all(
            VecGen { element_gen: IntGen { min: 0, max: 100 }, max_len: 50 },
            |v| {
                let original = v.clone();
                let mut reversed = v.clone();
                reversed.reverse();
                reversed.reverse();
                original == reversed
            }
        );

    assert!(matches!(result, TestResult::Passed { .. }));
}

#[test]
fn test_finding_counterexample() {
    // Intentionally failing property
    let result = PropTest::new()
        .num_tests(1000)
        .for_all(
            IntGen { min: 0, max: 1000 },
            |n| n < 500
        );

    match result {
        TestResult::Failed { counterexample, shrunk_to } => {
            // Should find a counterexample >= 500
            let shrunk: i64 = shrunk_to.parse().unwrap();
            assert!(shrunk >= 500);
            // Should shrink to minimal counterexample
            assert!(shrunk == 500);  // Minimal value >= 500
        }
        _ => panic!("Should have failed"),
    }
}

#[test]
fn test_hashmap_properties() {
    use std::collections::HashMap;

    let result = PropTest::new()
        .for_all(
            VecGen {
                element_gen: generators::tuple2(
                    IntGen { min: 0, max: 100 },
                    IntGen { min: 0, max: 1000 }
                ),
                max_len: 50,
            },
            |pairs| {
                let mut map = HashMap::new();
                for (k, v) in &pairs {
                    map.insert(*k, *v);
                }

                // Property: inserted values can be retrieved
                pairs.iter().all(|(k, _)| map.contains_key(k))
            }
        );

    assert!(matches!(result, TestResult::Passed { .. }));
}

#[test]
fn test_commutativity() {
    let result = PropTest::new()
        .for_all(
            generators::tuple2(
                IntGen { min: -1000, max: 1000 },
                IntGen { min: -1000, max: 1000 }
            ),
            |(a, b)| {
                is_commutative(|x, y| x + y, a, b)
            }
        );

    assert!(matches!(result, TestResult::Passed { .. }));
}

#[test]
fn test_associativity() {
    let result = PropTest::new()
        .for_all(
            generators::tuple3(
                IntGen { min: -100, max: 100 },
                IntGen { min: -100, max: 100 },
                IntGen { min: -100, max: 100 }
            ),
            |(a, b, c)| {
                is_associative(|x, y| x + y, a, b, c)
            }
        );

    assert!(matches!(result, TestResult::Passed { .. }));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic generators | 15 |
| Composite generators | 15 |
| Shrinking | 20 |
| Property test runner | 20 |
| Common property helpers | 15 |
| Algorithm property tests | 10 |
| Edge cases | 5 |
| **Total** | **100** |
