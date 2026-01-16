# MODULE 1.7 - ADVANCED ALGORITHMS & TECHNIQUES
## 162 Concepts - 18 Exercices Progressifs

---

# BLOC A: Geometric Algorithms (Exercices 01-05)

## Exercice 01: Computational Geometry Basics
**Concepts couverts**: 1.7.1.a-j (10 concepts)
**Difficulté**: ⭐⭐

```rust
#[derive(Clone, Copy, PartialEq)]
pub struct Point { pub x: f64, pub y: f64 }

impl Point {
    pub fn distance(&self, other: &Point) -> f64;             // 1.7.1.a
    pub fn dot(&self, other: &Point) -> f64;                  // 1.7.1.b
    pub fn cross(&self, other: &Point) -> f64;                // 1.7.1.c
}

// Orientation test
pub fn orientation(p: Point, q: Point, r: Point) -> i32;      // 1.7.1.d
// -1: clockwise, 0: collinear, 1: counter-clockwise

// Line segment intersection
pub fn segments_intersect(p1: Point, q1: Point, p2: Point, q2: Point) -> bool; // 1.7.1.e
pub fn intersection_point(p1: Point, q1: Point, p2: Point, q2: Point) -> Option<Point>; // 1.7.1.f

// Point in polygon
pub fn point_in_polygon(p: Point, polygon: &[Point]) -> bool; // 1.7.1.g

// Area
pub fn triangle_area(a: Point, b: Point, c: Point) -> f64;    // 1.7.1.h
pub fn polygon_area(polygon: &[Point]) -> f64;                // 1.7.1.i (shoelace)

// Collinearity
pub fn are_collinear(points: &[Point]) -> bool;               // 1.7.1.j
```

---

## Exercice 02: Convex Hull
**Concepts couverts**: 1.7.2.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Graham Scan
pub fn graham_scan(points: &[Point]) -> Vec<Point>;           // 1.7.2.a

// Andrew's Monotone Chain
pub fn monotone_chain(points: &[Point]) -> Vec<Point>;        // 1.7.2.b

// Jarvis March (Gift Wrapping)
pub fn jarvis_march(points: &[Point]) -> Vec<Point>;          // 1.7.2.c

// Quick Hull
pub fn quick_hull(points: &[Point]) -> Vec<Point>;            // 1.7.2.d

// Convex hull properties
pub fn hull_perimeter(hull: &[Point]) -> f64;                 // 1.7.2.e
pub fn hull_diameter(hull: &[Point]) -> f64;                  // 1.7.2.f (rotating calipers)

// Dynamic convex hull
pub struct DynamicConvexHull { ... }

impl DynamicConvexHull {
    pub fn insert(&mut self, p: Point);                       // 1.7.2.g
    pub fn query(&self) -> Vec<Point>;                        // 1.7.2.h
}
```

---

## Exercice 03: Closest Pair & Line Sweep
**Concepts couverts**: 1.7.3.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Closest pair (divide and conquer)
pub fn closest_pair(points: &[Point]) -> (Point, Point, f64); // 1.7.3.a O(n log n)

// Line sweep algorithms
pub struct Event { x: f64, typ: EventType, id: usize }

pub fn line_intersection_sweep(segments: &[(Point, Point)]) -> Vec<Point>; // 1.7.3.b

// Rectangle union area
pub fn rectangle_union_area(rects: &[(Point, Point)]) -> f64; // 1.7.3.c

// Skyline problem
pub fn skyline(buildings: &[(i32, i32, i32)]) -> Vec<(i32, i32)>; // 1.7.3.d

// Interval scheduling
pub fn max_non_overlapping(intervals: &[(i32, i32)]) -> Vec<usize>; // 1.7.3.e

// K-d Tree
pub struct KdTree { ... }

impl KdTree {
    pub fn build(points: &[Point]) -> Self;                   // 1.7.3.f
    pub fn nearest(&self, p: Point) -> Point;                 // 1.7.3.g
    pub fn range_search(&self, rect: (Point, Point)) -> Vec<Point>; // 1.7.3.h
}
```

---

## Exercice 04: Voronoi & Delaunay
**Concepts couverts**: 1.7.4.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Delaunay triangulation
pub struct DelaunayTriangulation {
    triangles: Vec<[usize; 3]>,
    points: Vec<Point>,
}

impl DelaunayTriangulation {
    pub fn build(points: &[Point]) -> Self;                   // 1.7.4.a
    pub fn triangles(&self) -> &[[usize; 3]];                 // 1.7.4.b
    pub fn is_delaunay(&self) -> bool;                        // 1.7.4.c (no point in circumcircle)
}

// Voronoi diagram
pub struct VoronoiDiagram {
    regions: Vec<Vec<Point>>,
    sites: Vec<Point>,
}

impl VoronoiDiagram {
    pub fn from_delaunay(dt: &DelaunayTriangulation) -> Self; // 1.7.4.d
    pub fn regions(&self) -> &[Vec<Point>];                   // 1.7.4.e
    pub fn nearest_site(&self, p: Point) -> usize;            // 1.7.4.f
}

// Fortune's algorithm (concept)
pub fn fortune_concept() -> String;                           // 1.7.4.g

// Applications
pub fn largest_empty_circle(points: &[Point]) -> (Point, f64); // 1.7.4.h
```

---

## Exercice 05: Polygon Operations
**Concepts couverts**: 1.7.5.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
pub struct Polygon {
    vertices: Vec<Point>,
}

impl Polygon {
    pub fn is_convex(&self) -> bool;                          // 1.7.5.a
    pub fn is_simple(&self) -> bool;                          // 1.7.5.b (no self-intersection)
    pub fn area(&self) -> f64;                                // 1.7.5.c
    pub fn centroid(&self) -> Point;                          // 1.7.5.d

    // Triangulation
    pub fn triangulate(&self) -> Vec<[usize; 3]>;             // 1.7.5.e (ear clipping)

    // Clipping
    pub fn clip_convex(&self, clip: &Polygon) -> Polygon;     // 1.7.5.f (Sutherland-Hodgman)

    // Boolean operations (concept)
    pub fn union(&self, other: &Polygon) -> Vec<Polygon>;     // 1.7.5.g
    pub fn intersection(&self, other: &Polygon) -> Vec<Polygon>; // 1.7.5.h
}
```

---

# BLOC B: String Algorithms (Exercices 06-09)

## Exercice 06: String Matching
**Concepts couverts**: 1.7.6.a-j (10 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Naive
pub fn naive_search(text: &str, pattern: &str) -> Vec<usize>; // 1.7.6.a

// KMP
pub fn kmp_search(text: &str, pattern: &str) -> Vec<usize>;   // 1.7.6.b
pub fn compute_lps(pattern: &str) -> Vec<usize>;              // 1.7.6.c

// Rabin-Karp
pub fn rabin_karp(text: &str, pattern: &str) -> Vec<usize>;   // 1.7.6.d

// Z-algorithm
pub fn z_function(s: &str) -> Vec<usize>;                     // 1.7.6.e
pub fn z_search(text: &str, pattern: &str) -> Vec<usize>;     // 1.7.6.f

// Aho-Corasick
pub struct AhoCorasick { ... }

impl AhoCorasick {
    pub fn new(patterns: &[&str]) -> Self;                    // 1.7.6.g
    pub fn search(&self, text: &str) -> Vec<(usize, usize)>;  // 1.7.6.h
}

// String hashing
pub fn polynomial_hash(s: &str, base: u64, mod_: u64) -> u64; // 1.7.6.i
pub fn rolling_hash(s: &str, len: usize) -> Vec<u64>;         // 1.7.6.j
```

---

## Exercice 07: Suffix Structures
**Concepts couverts**: 1.7.7.a-j (10 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Suffix Array
pub fn suffix_array(s: &str) -> Vec<usize>;                   // 1.7.7.a (O(n log n))
pub fn suffix_array_dc3(s: &str) -> Vec<usize>;               // 1.7.7.b (O(n))

// LCP Array
pub fn lcp_array(s: &str, sa: &[usize]) -> Vec<usize>;        // 1.7.7.c (Kasai)

// Applications
pub fn count_distinct_substrings(s: &str) -> usize;           // 1.7.7.d
pub fn longest_repeated_substring(s: &str) -> String;         // 1.7.7.e
pub fn longest_common_substring(s1: &str, s2: &str) -> String; // 1.7.7.f

// Suffix Tree (concept + Ukkonen)
pub struct SuffixTree { ... }

impl SuffixTree {
    pub fn build(s: &str) -> Self;                            // 1.7.7.g (Ukkonen O(n))
    pub fn search(&self, pattern: &str) -> bool;              // 1.7.7.h
    pub fn count_occurrences(&self, pattern: &str) -> usize;  // 1.7.7.i
    pub fn lcs_multiple(&self, strings: &[&str]) -> String;   // 1.7.7.j
}
```

---

## Exercice 08: Advanced String DP
**Concepts couverts**: 1.7.8.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Manacher's algorithm
pub fn manacher(s: &str) -> Vec<usize>;                       // 1.7.8.a
pub fn longest_palindrome_manacher(s: &str) -> String;        // 1.7.8.b

// Lyndon factorization
pub fn lyndon_factorization(s: &str) -> Vec<&str>;            // 1.7.8.c

// Minimum rotation
pub fn min_rotation(s: &str) -> usize;                        // 1.7.8.d (Booth)

// Burrows-Wheeler Transform
pub fn bwt(s: &str) -> (String, usize);                       // 1.7.8.e
pub fn inverse_bwt(bwt: &str, idx: usize) -> String;          // 1.7.8.f

// Run-length encoding
pub fn rle_encode(s: &str) -> String;                         // 1.7.8.g
pub fn rle_decode(s: &str) -> String;                         // 1.7.8.h
```

---

## Exercice 09: Automata & Regular Expressions
**Concepts couverts**: 1.7.9.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Deterministic Finite Automaton
pub struct DFA {
    states: usize,
    alphabet: Vec<char>,
    transitions: Vec<Vec<usize>>,
    start: usize,
    accepting: HashSet<usize>,
}

impl DFA {
    pub fn accepts(&self, s: &str) -> bool;                   // 1.7.9.a
    pub fn minimize(&self) -> Self;                           // 1.7.9.b
}

// NFA to DFA
pub struct NFA { ... }

impl NFA {
    pub fn to_dfa(&self) -> DFA;                              // 1.7.9.c (subset construction)
}

// Regex to NFA
pub fn regex_to_nfa(regex: &str) -> NFA;                      // 1.7.9.d (Thompson)

// Simple regex engine
pub struct RegexEngine { ... }

impl RegexEngine {
    pub fn compile(pattern: &str) -> Self;                    // 1.7.9.e
    pub fn matches(&self, text: &str) -> bool;                // 1.7.9.f
    pub fn find_all(&self, text: &str) -> Vec<&str>;          // 1.7.9.g
}

// Levenshtein automaton
pub fn levenshtein_automaton(word: &str, k: usize) -> DFA;    // 1.7.9.h
```

---

# BLOC C: Advanced Data Structures (Exercices 10-13)

## Exercice 10: Persistent Data Structures
**Concepts couverts**: 1.7.10.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Persistent Stack
pub struct PersistentStack<T: Clone> {
    versions: Vec<Option<Rc<Node<T>>>>,
}

impl<T: Clone> PersistentStack<T> {
    pub fn push(&self, version: usize, val: T) -> usize;      // 1.7.10.a
    pub fn pop(&self, version: usize) -> (Option<T>, usize);  // 1.7.10.b
    pub fn top(&self, version: usize) -> Option<&T>;          // 1.7.10.c
}

// Persistent Array (Fat Node)
pub struct PersistentArray<T: Clone> { ... }

impl<T: Clone> PersistentArray<T> {
    pub fn get(&self, version: usize, idx: usize) -> &T;      // 1.7.10.d
    pub fn set(&self, version: usize, idx: usize, val: T) -> usize; // 1.7.10.e
}

// Persistent Segment Tree
pub struct PersistentSegTree { ... }

impl PersistentSegTree {
    pub fn update(&self, version: usize, idx: usize, val: i64) -> usize; // 1.7.10.f
    pub fn query(&self, version: usize, l: usize, r: usize) -> i64; // 1.7.10.g
    pub fn kth_in_range(&self, v1: usize, v2: usize, k: usize) -> i64; // 1.7.10.h
}
```

---

## Exercice 11: Advanced Union-Find
**Concepts couverts**: 1.7.11.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Weighted Union-Find
pub struct WeightedUnionFind {
    parent: Vec<usize>,
    rank: Vec<usize>,
    diff: Vec<i64>,  // difference from parent
}

impl WeightedUnionFind {
    pub fn union_with_diff(&mut self, x: usize, y: usize, d: i64); // 1.7.11.a
    pub fn diff(&mut self, x: usize, y: usize) -> Option<i64>; // 1.7.11.b
}

// Partially Persistent Union-Find
pub struct PartiallyPersistentUF { ... }

impl PartiallyPersistentUF {
    pub fn union(&mut self, x: usize, y: usize);              // 1.7.11.c
    pub fn find_at(&self, x: usize, time: usize) -> usize;    // 1.7.11.d
}

// Rollback Union-Find
pub struct RollbackUnionFind { ... }

impl RollbackUnionFind {
    pub fn union(&mut self, x: usize, y: usize);              // 1.7.11.e
    pub fn checkpoint(&mut self) -> usize;                    // 1.7.11.f
    pub fn rollback(&mut self, checkpoint: usize);            // 1.7.11.g
}

// Link-Cut Trees (concept)
pub fn link_cut_tree_concept() -> String;                     // 1.7.11.h
```

---

## Exercice 12: Range Query Structures
**Concepts couverts**: 1.7.12.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Sparse Table (RMQ)
pub struct SparseTable { ... }

impl SparseTable {
    pub fn new(arr: &[i64]) -> Self;                          // 1.7.12.a O(n log n)
    pub fn query(&self, l: usize, r: usize) -> i64;           // 1.7.12.b O(1)
}

// Sqrt Decomposition
pub struct SqrtDecomp { ... }

impl SqrtDecomp {
    pub fn new(arr: &[i64]) -> Self;                          // 1.7.12.c
    pub fn query(&self, l: usize, r: usize) -> i64;           // 1.7.12.d O(√n)
    pub fn update(&mut self, idx: usize, val: i64);           // 1.7.12.e O(1)
}

// Mo's Algorithm
pub fn mo_algorithm(arr: &[i64], queries: &[(usize, usize)]) -> Vec<i64>; // 1.7.12.f

// Wavelet Tree
pub struct WaveletTree { ... }

impl WaveletTree {
    pub fn new(arr: &[i64]) -> Self;                          // 1.7.12.g
    pub fn kth(&self, l: usize, r: usize, k: usize) -> i64;   // 1.7.12.h
}
```

---

## Exercice 13: Probabilistic Structures
**Concepts couverts**: 1.7.13.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐

```rust
// Bloom Filter
pub struct BloomFilter {
    bits: BitVec,
    hash_count: usize,
}

impl BloomFilter {
    pub fn new(size: usize, hash_count: usize) -> Self;       // 1.7.13.a
    pub fn insert(&mut self, item: &[u8]);                    // 1.7.13.b
    pub fn contains(&self, item: &[u8]) -> bool;              // 1.7.13.c (may false positive)
    pub fn false_positive_rate(&self) -> f64;                 // 1.7.13.d
}

// Count-Min Sketch
pub struct CountMinSketch { ... }

impl CountMinSketch {
    pub fn new(width: usize, depth: usize) -> Self;           // 1.7.13.e
    pub fn add(&mut self, item: &[u8], count: u64);           // 1.7.13.f
    pub fn estimate(&self, item: &[u8]) -> u64;               // 1.7.13.g
}

// HyperLogLog (concept)
pub struct HyperLogLog { ... }

impl HyperLogLog {
    pub fn add(&mut self, item: &[u8]);
    pub fn count(&self) -> u64;                               // 1.7.13.h (cardinality estimation)
}
```

---

# BLOC D: Optimization & Misc (Exercices 14-16)

## Exercice 14: Linear Programming Basics
**Concepts couverts**: 1.7.14.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Simplex method (basics)
pub struct LinearProgram {
    c: Vec<f64>,       // objective coefficients
    a: Vec<Vec<f64>>,  // constraint matrix
    b: Vec<f64>,       // constraint bounds
}

impl LinearProgram {
    pub fn new(c: Vec<f64>, a: Vec<Vec<f64>>, b: Vec<f64>) -> Self; // 1.7.14.a
    pub fn solve_simplex(&self) -> Option<(Vec<f64>, f64)>;   // 1.7.14.b
    pub fn is_feasible(&self) -> bool;                        // 1.7.14.c
    pub fn is_bounded(&self) -> bool;                         // 1.7.14.d
}

// Integer Linear Programming (concept)
pub fn ilp_branch_bound_concept() -> String;                  // 1.7.14.e

// 2D Linear Programming
pub fn lp_2d(half_planes: &[(f64, f64, f64)], dir: (f64, f64)) -> Option<(f64, f64)>; // 1.7.14.f

// Applications
pub fn max_flow_lp(graph: &[Vec<(usize, f64)>], s: usize, t: usize) -> f64; // 1.7.14.g
pub fn min_cost_flow_lp(graph: &[Vec<(usize, f64, f64)>], s: usize, t: usize) -> f64; // 1.7.14.h
```

---

## Exercice 15: Fast Fourier Transform
**Concepts couverts**: 1.7.15.a-j (10 concepts)
**Difficulté**: ⭐⭐⭐⭐⭐

```rust
use num_complex::Complex64;

// FFT
pub fn fft(a: &mut [Complex64]);                              // 1.7.15.a
pub fn ifft(a: &mut [Complex64]);                             // 1.7.15.b

// Polynomial multiplication
pub fn poly_mul(a: &[i64], b: &[i64]) -> Vec<i64>;            // 1.7.15.c O(n log n)

// Number Theoretic Transform (NTT)
pub fn ntt(a: &mut [u64], mod_: u64, g: u64);                 // 1.7.15.d
pub fn intt(a: &mut [u64], mod_: u64, g: u64);                // 1.7.15.e

// Polynomial operations
pub fn poly_div(a: &[i64], b: &[i64]) -> (Vec<i64>, Vec<i64>); // 1.7.15.f
pub fn poly_mod(a: &[i64], b: &[i64]) -> Vec<i64>;            // 1.7.15.g

// Applications
pub fn large_integer_mul(a: &str, b: &str) -> String;         // 1.7.15.h
pub fn convolution(a: &[i64], b: &[i64]) -> Vec<i64>;         // 1.7.15.i

// Karatsuba (for comparison)
pub fn karatsuba(a: &[i64], b: &[i64]) -> Vec<i64>;           // 1.7.15.j O(n^1.585)
```

---

## Exercice 16: Approximation Algorithms
**Concepts couverts**: 1.7.16.a-h (8 concepts)
**Difficulté**: ⭐⭐⭐⭐

```rust
// Vertex Cover (2-approximation)
pub fn vertex_cover_approx(edges: &[(usize, usize)]) -> Vec<usize>; // 1.7.16.a

// Set Cover (greedy, log n approximation)
pub fn set_cover_greedy(universe: &[usize], sets: &[Vec<usize>]) -> Vec<usize>; // 1.7.16.b

// TSP (Christofides 1.5-approximation for metric)
pub fn tsp_christofides(dist: &[Vec<f64>]) -> Vec<usize>;     // 1.7.16.c

// Bin Packing (First Fit Decreasing)
pub fn bin_packing_ffd(items: &[f64], capacity: f64) -> Vec<Vec<usize>>; // 1.7.16.d

// Knapsack FPTAS
pub fn knapsack_fptas(weights: &[u32], values: &[u32], capacity: u32, epsilon: f64) -> u32; // 1.7.16.e

// Max-Cut (randomized 0.5-approximation)
pub fn max_cut_random(edges: &[(usize, usize)]) -> (Vec<usize>, Vec<usize>); // 1.7.16.f

// Scheduling (LPT for makespan)
pub fn scheduling_lpt(jobs: &[u32], machines: usize) -> Vec<Vec<usize>>; // 1.7.16.g

// Load Balancing
pub fn load_balance(tasks: &[u32], servers: usize) -> Vec<Vec<usize>>; // 1.7.16.h
```

---

# BLOC E: Projet Final (Exercices 17-18)

## Exercice 17: Algorithm Library
**Concepts couverts**: Library compilation
**Difficulté**: ⭐⭐⭐⭐

```rust
pub mod advanced_algorithms {
    pub mod geometry {
        pub fn convex_hull(points: &[Point]) -> Vec<Point>;
        pub fn closest_pair(points: &[Point]) -> (Point, Point);
        pub fn line_intersection(segments: &[(Point, Point)]) -> Vec<Point>;
    }

    pub mod strings {
        pub fn kmp(text: &str, pattern: &str) -> Vec<usize>;
        pub fn suffix_array(s: &str) -> Vec<usize>;
        pub fn z_function(s: &str) -> Vec<usize>;
        pub fn manacher(s: &str) -> Vec<usize>;
    }

    pub mod structures {
        pub fn sparse_table(arr: &[i64]) -> SparseTable;
        pub fn persistent_segtree(arr: &[i64]) -> PersistentSegTree;
        pub fn bloom_filter(size: usize) -> BloomFilter;
    }

    pub mod numeric {
        pub fn fft(a: &mut [Complex64]);
        pub fn ntt(a: &mut [u64], mod_: u64);
        pub fn poly_mul(a: &[i64], b: &[i64]) -> Vec<i64>;
    }
}
```

---

## Exercice 18: Competition Toolkit (Projet)
**Concepts couverts**: 1.7.a-n (14 concepts projet)
**Difficulté**: ⭐⭐⭐⭐⭐

```rust
pub struct CompetitionToolkit {
    // All algorithms integrated
}

impl CompetitionToolkit {
    // Geometry
    pub fn geometry_suite(&self) -> GeometryAPI;              // 1.7.a

    // String matching
    pub fn string_matching(&self, text: &str, pattern: &str) -> Vec<usize>; // 1.7.b

    // Suffix structures
    pub fn suffix_array(&self, s: &str) -> Vec<usize>;        // 1.7.c

    // Persistent DS
    pub fn persistent_query(&self, version: usize, query: Query) -> i64; // 1.7.d

    // Range queries
    pub fn range_query(&self, structure: RangeStructure, l: usize, r: usize) -> i64; // 1.7.e

    // FFT/NTT
    pub fn polynomial_multiply(&self, a: &[i64], b: &[i64]) -> Vec<i64>; // 1.7.f

    // Approximation
    pub fn approximate(&self, problem: NpHardProblem) -> Solution; // 1.7.g

    // Randomized
    pub fn randomized(&self, problem: RandomizedProblem) -> Solution; // 1.7.h

    // CLI
    pub fn cli_handler(args: &[String]) -> Result<String, Error>; // 1.7.i

    // Templates
    pub fn generate_template(&self, problem_type: ProblemType) -> String; // 1.7.j

    // Benchmarks
    pub fn benchmark(&self, algorithm: &str, sizes: &[usize]) -> Vec<Duration>; // 1.7.k

    // Stress testing
    pub fn stress_test<F, G>(&self, correct: F, candidate: G, gen: impl Fn() -> Input) -> Option<Input>
    where F: Fn(&Input) -> Output, G: Fn(&Input) -> Output;   // 1.7.l

    // Bonus: Parallel algorithms
    pub fn parallel_sort(&self, arr: &mut [i64]);             // 1.7.m

    // Bonus: External memory
    pub fn external_sort(&self, file: &Path) -> io::Result<()>; // 1.7.n
}
```

---

# RÉCAPITULATIF

| Bloc | Exercices | Concepts | Description |
|------|-----------|----------|-------------|
| A | 01-05 | 42 | Computational Geometry |
| B | 06-09 | 36 | String Algorithms |
| C | 10-13 | 32 | Advanced Data Structures |
| D | 14-16 | 26 | Optimization & FFT |
| E | 17-18 | 26 | Projet & Library |
| **TOTAL** | **18** | **162** | **Module 1.7 complet** |

---
