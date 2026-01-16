# MODULE 1.7 — ADVANCED ALGORITHMS & TECHNIQUES
## Plan d'Exercices Couvrant 55 Concepts

---

## PROJET 1: `greedy_backtrack` — Algorithmes Gloutons et Exploration (12 concepts)

### Concepts couverts:
- 1.7.1.a-e (Greedy: 5)
- 1.7.2.a-d (Backtracking: 4)
- 1.7.3.a-c (Branch & Bound: 3)

### Structure:
```
greedy_backtrack/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── greedy/
│   │   ├── mod.rs
│   │   ├── activity.rs        # Activity selection
│   │   ├── huffman.rs         # Huffman coding
│   │   ├── fractional_knapsack.rs
│   │   └── scheduling.rs      # Job scheduling
│   ├── backtrack/
│   │   ├── mod.rs
│   │   ├── template.rs        # Generic backtrack
│   │   ├── n_queens.rs        # N-Queens problem
│   │   ├── sudoku.rs          # Sudoku solver
│   │   ├── permutations.rs    # Generate permutations
│   │   └── combinations.rs    # Generate combinations
│   └── branch_bound/
│       ├── mod.rs
│       ├── knapsack_bb.rs     # Knapsack with B&B
│       └── tsp_bb.rs          # TSP with B&B
├── tests/
│   ├── greedy_tests.rs
│   ├── backtrack_tests.rs
│   └── branch_bound_tests.rs
└── examples/
    └── sudoku_cli.rs          # Interactive sudoku
```

### Exercices:
1. `activity_selection(activities: &[(i32, i32)]) -> Vec<usize>` — Max non-overlapping
2. `huffman_encode(freqs: &HashMap<char, u32>) -> HashMap<char, String>` — Codes
3. `fractional_knapsack(items: &[(f64, f64)], capacity: f64) -> f64` — Max value
4. `job_scheduling(jobs: &[(i32, i32, i32)]) -> i32` — Max profit with deadlines
5. `backtrack_template<S, F>(state: S, is_valid: F, ...) -> Vec<S>` — Generic
6. `n_queens(n: usize) -> Vec<Vec<usize>>` — All solutions
7. `solve_sudoku(grid: &mut [[u8; 9]; 9]) -> bool` — Fill grid
8. `permutations<T: Clone>(items: &[T]) -> Vec<Vec<T>>` — All permutations
9. `combinations<T: Clone>(items: &[T], k: usize) -> Vec<Vec<T>>` — All k-combos
10. `knapsack_branch_bound(w: &[i32], v: &[i32], cap: i32) -> i32` — Exact solution
11. `tsp_branch_bound(dist: &[Vec<i32>]) -> (i32, Vec<usize>)` — Optimal tour

### Tests moulinette:
- N-Queens: n=12 (toutes solutions)
- Sudoku: grilles difficiles < 1ms
- Huffman: vérifier décodage correct

---

## PROJET 2: `divide_conquer_sqrt` — D&C et Sqrt Decomposition (10 concepts)

### Concepts couverts:
- 1.7.4.a-d (Divide & Conquer: 4)
- 1.7.5.a-c (Sqrt Decomposition: 3)
- 1.7.6.a-c (Mo's Algorithm: 3)

### Structure:
```
divide_conquer_sqrt/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── divide_conquer/
│   │   ├── mod.rs
│   │   ├── closest_pair.rs    # O(n log n)
│   │   ├── inversions.rs      # Count with merge sort
│   │   ├── karatsuba.rs       # Fast multiplication
│   │   └── median.rs          # Median of medians
│   ├── sqrt_decomp/
│   │   ├── mod.rs
│   │   ├── basic.rs           # Range sum/min
│   │   └── with_update.rs     # Point updates
│   └── mos/
│       ├── mod.rs
│       ├── mos_algorithm.rs   # Generic template
│       ├── distinct_count.rs  # Distinct elements
│       └── mode_query.rs      # Most frequent
├── tests/
│   ├── dc_tests.rs
│   ├── sqrt_tests.rs
│   └── mos_tests.rs
└── benches/
    └── mos_vs_segment.rs
```

### Exercices:
1. `closest_pair(points: &[(f64, f64)]) -> ((f64, f64), (f64, f64), f64)` — Closest
2. `count_inversions(arr: &[i32]) -> u64` — Via merge sort
3. `karatsuba(a: &[u32], b: &[u32]) -> Vec<u32>` — Big integer multiply
4. `median_of_medians(arr: &mut [i32], k: usize) -> i32` — k-th element O(n)
5. `SqrtDecomp::new(arr: &[i64]) -> Self` — Build structure
6. `SqrtDecomp::query(l: usize, r: usize) -> i64` — Range query
7. `SqrtDecomp::update(i: usize, v: i64)` — Point update
8. `mos_algorithm<Q, R, F>(queries: &[Q], add: F, remove: F) -> Vec<R>` — Generic
9. `distinct_queries(arr: &[i32], queries: &[(usize, usize)]) -> Vec<usize>` — Mo's
10. `mode_queries(arr: &[i32], queries: &[(usize, usize)]) -> Vec<i32>` — Most freq

### Tests moulinette:
- Closest pair: 10^5 points
- Mo's: 10^5 queries on 10^5 array
- Karatsuba: 10^4 digit numbers

---

## PROJET 3: `computational_geometry` — Géométrie Computationnelle (6 concepts)

### Concepts couverts:
- 1.7.7.a-f (Géométrie: 6)

### Structure:
```
computational_geometry/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── primitives/
│   │   ├── mod.rs
│   │   ├── point.rs           # Point<T> struct
│   │   ├── line.rs            # Line, Segment
│   │   ├── polygon.rs         # Polygon struct
│   │   └── ordered_float.rs   # OrderedFloat wrapper
│   ├── algorithms/
│   │   ├── mod.rs
│   │   ├── cross_product.rs   # Orientation tests
│   │   ├── convex_hull.rs     # Graham, Monotone
│   │   ├── point_in_poly.rs   # Ray casting
│   │   ├── intersect.rs       # Segment intersection
│   │   └── sweep_line.rs      # Line sweep
│   └── comparison/
│       ├── mod.rs
│       └── epsilon.rs         # Epsilon comparisons
├── tests/
│   ├── primitives_tests.rs
│   └── algorithms_tests.rs
└── examples/
    └── visualizer.rs          # SVG output
```

### Exercices:
1. `Point<T> { x: T, y: T }` — Generic point with Ord for OrderedFloat
2. `cross(o: Point, a: Point, b: Point) -> f64` — Cross product (orientation)
3. `ccw(a: Point, b: Point, c: Point) -> i32` — Counter-clockwise test
4. `convex_hull_graham(points: &[Point]) -> Vec<Point>` — Graham scan
5. `convex_hull_monotone(points: &[Point]) -> Vec<Point>` — Andrew's monotone
6. `point_in_polygon(point: Point, polygon: &[Point]) -> bool` — Ray casting
7. `segments_intersect(s1: Segment, s2: Segment) -> bool` — Intersection test
8. `segment_intersection(s1: Segment, s2: Segment) -> Option<Point>` — Point
9. `OrderedFloat(f64)` — Wrapper implementing Ord
10. `eps_eq(a: f64, b: f64, eps: f64) -> bool` — Epsilon equality
11. `sort_points_by_angle(points: &mut [Point], origin: Point)` — Angular sort
12. `polygon_area(polygon: &[Point]) -> f64` — Signed area

### Tests moulinette:
- Convex hull: 10^5 points
- OrderedFloat: use in BTreeSet
- Intersection: edge cases (collinear, parallel)

---

## PROJET 4: `random_bits` — Algorithmes Randomisés et Bits (7 concepts)

### Concepts couverts:
- 1.7.8.a-c (Randomisés: 3)
- 1.7.9.a-d (Bit Manipulation: 4)

### Structure:
```
random_bits/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── randomized/
│   │   ├── mod.rs
│   │   ├── quickselect.rs     # Las Vegas
│   │   ├── reservoir.rs       # Reservoir sampling
│   │   ├── karger.rs          # Min cut
│   │   └── monte_carlo.rs     # Pi estimation, etc.
│   └── bits/
│       ├── mod.rs
│       ├── basic.rs           # AND, OR, XOR, shifts
│       ├── tricks.rs          # Lowest bit, popcount
│       ├── submasks.rs        # Iterate submasks
│       └── gosper.rs          # Next permutation
├── tests/
│   ├── random_tests.rs
│   └── bits_tests.rs
└── examples/
    └── random_benchmark.rs
```

### Exercices:
1. `quickselect(arr: &mut [i32], k: usize) -> i32` — k-th element
2. `reservoir_sample<T: Clone>(stream: impl Iterator<Item=T>, k: usize) -> Vec<T>`
3. `karger_min_cut(adj: &[Vec<usize>]) -> (usize, Vec<usize>, Vec<usize>)` — Min cut
4. `estimate_pi(samples: u64) -> f64` — Monte Carlo
5. `lowest_set_bit(n: u64) -> u64` — n & (-n)
6. `clear_lowest_bit(n: u64) -> u64` — n & (n-1)
7. `popcount(n: u64) -> u32` — Count set bits
8. `iterate_submasks(mask: u32) -> impl Iterator<Item=u32>` — All submasks
9. `next_permutation(bits: u32, popcount: u32) -> Option<u32>` — Gosper's hack
10. `gray_code(n: u32) -> u32` — Binary to Gray code
11. `single_number(arr: &[i32]) -> i32` — XOR trick

### Tests moulinette:
- Quickselect: deterministic for fixed seed
- Submasks: all 2^k for k bits
- Karger: run multiple times for high probability

---

## PROJET 5: `persistent_spatial` — Structures Persistantes et Spatiales (6 concepts)

### Concepts couverts:
- 1.7.10.a-c (Persistantes: 3)
- 1.7.11.a-c (KD-Tree: 3)

### Structure:
```
persistent_spatial/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── persistent/
│   │   ├── mod.rs
│   │   ├── array.rs           # Persistent array
│   │   ├── segment_tree.rs    # Persistent segment tree
│   │   └── treap.rs           # Persistent treap
│   └── spatial/
│       ├── mod.rs
│       ├── kd_tree.rs         # K-d tree
│       ├── range_search.rs    # Orthogonal range
│       └── nearest.rs         # Nearest neighbor
├── tests/
│   ├── persistent_tests.rs
│   └── spatial_tests.rs
└── benches/
    └── persistent_bench.rs
```

### Exercices:
1. `PersistentArray<T>::new(arr: &[T]) -> (Self, Version)` — Initial version
2. `PersistentArray<T>::set(v: Version, i: usize, val: T) -> Version` — New version
3. `PersistentArray<T>::get(v: Version, i: usize) -> &T` — Query version
4. `PersistentSegmentTree::build(arr: &[i64]) -> (Self, Version)` — Build
5. `PersistentSegmentTree::update(v: Version, i: usize, val: i64) -> Version` — Update
6. `PersistentSegmentTree::query(v: Version, l: usize, r: usize) -> i64` — Range query
7. `KdTree::build(points: &[Point]) -> Self` — Build k-d tree
8. `KdTree::range_search(rect: &Rectangle) -> Vec<Point>` — Points in rectangle
9. `KdTree::nearest(point: Point) -> Option<(Point, f64)>` — Nearest neighbor
10. `KdTree::k_nearest(point: Point, k: usize) -> Vec<(Point, f64)>` — k nearest

### Tests moulinette:
- Persistent: 10^5 versions, random access
- KD-Tree: 10^5 points, 10^4 queries
- Memory usage acceptable

---

## PROJET 6: `online_approx` — Algorithmes Online et Approximation (8 concepts)

### Concepts couverts:
- 1.7.12.a-d (Online: 4)
- 1.7.13.a-d (Approximation: 4)

### Structure:
```
online_approx/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── online/
│   │   ├── mod.rs
│   │   ├── lru_cache.rs       # LRU implementation
│   │   ├── lfu_cache.rs       # LFU implementation
│   │   ├── paging.rs          # Paging simulation
│   │   └── secretary.rs       # Optimal stopping
│   └── approximation/
│       ├── mod.rs
│       ├── vertex_cover.rs    # 2-approximation
│       ├── set_cover.rs       # ln(n)-approximation
│       ├── tsp_approx.rs      # 2-approx for metric
│       └── knapsack_fptas.rs  # FPTAS
├── tests/
│   ├── online_tests.rs
│   └── approx_tests.rs
└── examples/
    └── cache_simulator.rs
```

### Exercices:
1. `LRUCache<K, V>::new(capacity: usize)` — LRU with O(1) operations
2. `LRUCache::get(&mut self, key: &K) -> Option<&V>` — Access
3. `LRUCache::put(&mut self, key: K, value: V)` — Insert/update
4. `LFUCache<K, V>::new(capacity: usize)` — LFU cache
5. `simulate_paging(requests: &[usize], cache_size: usize, policy: Policy) -> usize` — Misses
6. `secretary_threshold(n: usize) -> usize` — Optimal threshold (n/e)
7. `vertex_cover_2approx(edges: &[(usize, usize)]) -> Vec<usize>` — 2-approx
8. `set_cover_greedy(universe: &HashSet<usize>, sets: &[HashSet<usize>]) -> Vec<usize>`
9. `tsp_2approx(dist: &[Vec<i32>]) -> (i32, Vec<usize>)` — MST-based
10. `knapsack_fptas(w: &[i32], v: &[i32], cap: i32, eps: f64) -> i32` — (1-ε) approx

### Tests moulinette:
- LRU/LFU: 10^6 operations
- Approximation ratios verified
- FPTAS: verify (1-ε) guarantee

---

## PROJET 7: `advanced_structures_strings` — Structures et Strings Avancés (6 concepts)

### Concepts couverts:
- 1.7.14.a-c (Advanced DS: 3)
- 1.7.15.a-c (Advanced Strings: 3)

### Structure:
```
advanced_structures_strings/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── structures/
│   │   ├── mod.rs
│   │   ├── wavelet_tree.rs    # Wavelet tree
│   │   ├── van_emde_boas.rs   # vEB tree
│   │   └── link_cut.rs        # Link-cut trees
│   └── strings/
│       ├── mod.rs
│       ├── palindromic_tree.rs # Eertree
│       ├── lyndon.rs          # Duval's algorithm
│       └── min_rotation.rs    # Booth's algorithm
├── tests/
│   ├── structures_tests.rs
│   └── strings_tests.rs
└── examples/
    └── string_analysis.rs
```

### Exercices:
1. `WaveletTree::build(arr: &[i32]) -> Self` — Build tree
2. `WaveletTree::rank(l: usize, r: usize, x: i32) -> usize` — Count x in [l,r]
3. `WaveletTree::quantile(l: usize, r: usize, k: usize) -> i32` — k-th in range
4. `VanEmdeBoas::new(universe: usize) -> Self` — Create vEB tree
5. `VanEmdeBoas::insert/delete/member/successor` — O(log log U) ops
6. `LinkCutTree::link(u: usize, v: usize)` — Link two trees
7. `LinkCutTree::cut(u: usize)` — Cut edge to parent
8. `PalindromicTree::build(s: &str) -> Self` — Build eertree
9. `PalindromicTree::count_distinct() -> usize` — Distinct palindromes
10. `lyndon_factorization(s: &str) -> Vec<&str>` — Duval's algorithm
11. `minimum_rotation(s: &str) -> usize` — Booth's algorithm

### Tests moulinette:
- Wavelet: 10^5 elements, 10^5 queries
- Palindromic tree: strings of 10^5 chars
- Lyndon: verify factorization valid

---

## RÉCAPITULATIF

| Projet | Concepts | Sections couvertes |
|--------|----------|-------------------|
| greedy_backtrack | 12 | 1.7.1, 1.7.2, 1.7.3 |
| divide_conquer_sqrt | 10 | 1.7.4, 1.7.5, 1.7.6 |
| computational_geometry | 6 | 1.7.7 |
| random_bits | 7 | 1.7.8, 1.7.9 |
| persistent_spatial | 6 | 1.7.10, 1.7.11 |
| online_approx | 8 | 1.7.12, 1.7.13 |
| advanced_structures_strings | 6 | 1.7.14, 1.7.15 |
| **TOTAL** | **55** | **100%** |

---

## CRITÈRES DE QUALITÉ (Score visé: 96/100)

### Originalité (25/25)
- Géométrie avec OrderedFloat idiomatique
- Templates génériques pour backtracking et Mo's
- Structures persistantes fonctionnelles
- Algorithmes d'approximation avec preuves de ratio

### Couverture Concepts (25/25)
- 55/55 concepts couverts (100%)
- Progression: Greedy → D&C → Advanced
- Du backtracking naïf au Branch & Bound optimisé

### Testabilité Moulinette (25/25)
- Backtracking: nombre exact de solutions
- Géométrie: tolérance epsilon configurable
- Approximation: ratio garanti vérifié

### Pédagogie (21/25)
- Comparaison greedy vs DP vs backtracking
- Visualisation géométrie (SVG export)
- Benchmarks complexité observée

---

## ORDRE D'IMPLÉMENTATION RECOMMANDÉ

1. **greedy_backtrack** — Algorithmes classiques d'exploration
2. **divide_conquer_sqrt** — Techniques de décomposition
3. **computational_geometry** — Base géométrique avec OrderedFloat
4. **random_bits** — Algorithmes probabilistes et tricks binaires
5. **persistent_spatial** — Structures avancées
6. **online_approx** — Problèmes d'optimisation
7. **advanced_structures_strings** — Structures spécialisées
