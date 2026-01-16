# Exercise 10: Mini-Project - Complete Sorting & Searching Library

## Concepts Covered
All 110 concepts from Module 1.1 integrated into a production-quality library.

## Objective

Build a complete, production-ready sorting and searching library that:
1. Implements all major algorithms
2. Provides generic, type-safe APIs
3. Includes comprehensive benchmarks
4. Has full documentation
5. Passes automated testing

## Requirements

### Library Structure

```
sorting_library/
├── Cargo.toml
├── src/
│   ├── lib.rs           # Public API
│   ├── vector.rs        # GenericVec implementation
│   ├── sorting/
│   │   ├── mod.rs
│   │   ├── comparison.rs   # Comparison-based sorts
│   │   ├── non_comparison.rs  # Counting, Radix, Bucket
│   │   └── hybrid.rs       # Intro, Tim, Pattern-defeating
│   ├── searching/
│   │   ├── mod.rs
│   │   ├── binary.rs       # Binary search variants
│   │   ├── ternary.rs      # Ternary search
│   │   └── interpolation.rs
│   ├── techniques/
│   │   ├── mod.rs
│   │   ├── two_pointers.rs
│   │   ├── sliding_window.rs
│   │   ├── prefix_sum.rs
│   │   └── compression.rs
│   ├── memory/
│   │   ├── mod.rs
│   │   └── arena.rs
│   └── analysis/
│       ├── mod.rs
│       ├── complexity.rs
│       └── benchmark.rs
├── benches/
│   └── sorting_bench.rs
└── tests/
    ├── sorting_tests.rs
    └── integration_tests.rs
```

### Public API

```rust
// lib.rs
pub mod vector;
pub mod sorting;
pub mod searching;
pub mod techniques;
pub mod memory;
pub mod analysis;

// Re-exports
pub use vector::GenericVec;
pub use sorting::{
    bubble_sort, selection_sort, insertion_sort, shell_sort,
    merge_sort, quick_sort, quick_sort_3way, heap_sort,
    intro_sort, tim_sort, counting_sort, radix_sort, bucket_sort,
};
pub use searching::{
    binary_search, lower_bound, upper_bound,
    search_rotated, find_peak, ternary_search,
};
pub use techniques::{
    PrefixSum, DifferenceArray, Compressor,
    sliding_window_max, two_sum, three_sum,
};
pub use memory::Arena;

/// Sort trait for uniform interface
pub trait Sortable<T: Ord> {
    fn sort(&mut self);
    fn sort_by<F>(&mut self, compare: F) where F: FnMut(&T, &T) -> std::cmp::Ordering;
    fn is_sorted(&self) -> bool;
}

/// Benchmark configuration
pub struct BenchConfig {
    pub sizes: Vec<usize>,
    pub iterations: usize,
    pub warmup: usize,
}

/// Run benchmarks on all sorting algorithms
pub fn benchmark_all_sorts(config: &BenchConfig) -> BenchmarkReport;

/// Verify correctness of all algorithms
pub fn verify_all() -> TestReport;
```

### Required Features

1. **Sorting Algorithms** (minimum 12)
   - Bubble, Selection, Insertion, Shell
   - Merge (recursive + bottom-up)
   - Quick (standard, 3-way, dual-pivot)
   - Heap
   - Intro, Tim
   - Counting, Radix, Bucket

2. **Search Algorithms** (minimum 8)
   - Binary search (standard + variants)
   - Ternary search
   - Interpolation search
   - Exponential search

3. **Array Techniques**
   - Two pointers
   - Sliding window
   - Prefix sums / Difference arrays
   - Coordinate compression

4. **Memory Management**
   - Arena allocator
   - Custom vector

5. **Analysis Tools**
   - Benchmarking framework
   - Complexity estimation
   - Performance comparison

### Quality Requirements

1. **Documentation**
   - Every public function documented
   - Algorithm complexity stated
   - Usage examples provided

2. **Testing**
   - Unit tests for each function
   - Property-based tests
   - Edge case coverage
   - At least 80% code coverage

3. **Benchmarks**
   - Criterion-based benchmarks
   - Comparison charts
   - Performance profiles

### Sample Usage

```rust
use sorting_library::*;

fn main() {
    // Vector with all operations
    let mut vec: GenericVec<i32> = GenericVec::new();
    vec.push(5);
    vec.push(2);
    vec.push(8);

    // Multiple sort options
    let mut arr = vec![5, 2, 8, 1, 9];
    quick_sort(&mut arr);
    assert_eq!(arr, [1, 2, 5, 8, 9]);

    // Binary search variants
    let arr = vec![1, 2, 2, 2, 3, 4, 5];
    assert_eq!(lower_bound(&arr, &2), 1);
    assert_eq!(upper_bound(&arr, &2), 4);

    // Prefix sums
    let ps = PrefixSum::new(&[1, 2, 3, 4, 5]);
    assert_eq!(ps.range_sum(1, 3), 9);

    // Sliding window
    let maxs = sliding_window_max(&[1, 3, -1, -3, 5, 3, 6, 7], 3);
    assert_eq!(maxs, [3, 3, 5, 5, 6, 7]);

    // Arena allocation
    let arena = Arena::new(1024);
    let data = arena.alloc_slice(0i32, 100).unwrap();

    // Benchmarking
    let config = BenchConfig {
        sizes: vec![100, 1000, 10000],
        iterations: 100,
        warmup: 10,
    };
    let report = benchmark_all_sorts(&config);
    println!("{}", report);
}
```

## Test Cases

```rust
#[test]
fn test_all_sorts_correctness() {
    let sorts: Vec<(&str, fn(&mut [i32]))> = vec![
        ("bubble", bubble_sort),
        ("selection", selection_sort),
        ("insertion", insertion_sort),
        ("shell", shell_sort),
        ("merge", merge_sort),
        ("quick", quick_sort),
        ("heap", heap_sort),
        ("intro", intro_sort),
    ];

    for (name, sort_fn) in sorts {
        // Random array
        let mut arr = vec![5, 2, 8, 1, 9, 3, 7, 4, 6];
        sort_fn(&mut arr);
        assert!(arr.is_sorted(), "{} failed on random", name);

        // Already sorted
        let mut arr = vec![1, 2, 3, 4, 5];
        sort_fn(&mut arr);
        assert!(arr.is_sorted(), "{} failed on sorted", name);

        // Reverse sorted
        let mut arr = vec![5, 4, 3, 2, 1];
        sort_fn(&mut arr);
        assert!(arr.is_sorted(), "{} failed on reverse", name);

        // Empty
        let mut arr: Vec<i32> = vec![];
        sort_fn(&mut arr);
        assert!(arr.is_sorted(), "{} failed on empty", name);

        // Single element
        let mut arr = vec![42];
        sort_fn(&mut arr);
        assert!(arr.is_sorted(), "{} failed on single", name);

        // All same
        let mut arr = vec![5, 5, 5, 5, 5];
        sort_fn(&mut arr);
        assert!(arr.is_sorted(), "{} failed on same", name);
    }
}

#[test]
fn test_large_scale() {
    let mut rng = rand::thread_rng();
    let mut arr: Vec<i32> = (0..10000).map(|_| rng.gen()).collect();

    quick_sort(&mut arr);
    assert!(arr.is_sorted());

    let mut arr: Vec<i32> = (0..10000).map(|_| rng.gen()).collect();
    merge_sort(&mut arr);
    assert!(arr.is_sorted());
}

#[test]
fn test_stability() {
    #[derive(Clone, Eq, PartialEq, Debug)]
    struct Item { key: i32, order: usize }
    impl Ord for Item {
        fn cmp(&self, other: &Self) -> std::cmp::Ordering {
            self.key.cmp(&other.key)
        }
    }
    impl PartialOrd for Item {
        fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
            Some(self.cmp(other))
        }
    }

    let mut arr = vec![
        Item { key: 2, order: 0 },
        Item { key: 1, order: 1 },
        Item { key: 2, order: 2 },
    ];

    merge_sort(&mut arr);
    assert_eq!(arr[1].order, 0);  // First '2' comes before second '2'
    assert_eq!(arr[2].order, 2);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| All 12+ sorting algorithms correct | 20 |
| All 8+ search algorithms correct | 10 |
| Array techniques (pointers, window, prefix) | 15 |
| Arena allocator working | 10 |
| Comprehensive test suite | 15 |
| Benchmarking framework | 10 |
| Documentation complete | 10 |
| Code quality and organization | 10 |
| **Total** | **100** |

## Deliverables

1. Complete library source code
2. All tests passing
3. Benchmark results
4. README with usage examples
5. API documentation

## Files to Submit

```
sorting_library/
├── Cargo.toml
├── README.md
├── src/
│   └── [all source files]
├── benches/
│   └── [benchmark files]
└── tests/
    └── [test files]
```
