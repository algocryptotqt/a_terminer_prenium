# Exercise 04: Approximation Algorithms

## Concepts Covered
- **1.7.8.d-l** Approximation ratios, PTAS, FPTAS
- **1.7.9.d-k** Vertex cover, set cover, TSP approximation

## Objective

Implement approximation algorithms for NP-hard problems.

## Requirements

### Rust Implementation

```rust
pub mod approximation {
    /// 2-approximation for vertex cover (greedy)
    pub fn vertex_cover_2approx(adj: &[Vec<usize>]) -> Vec<usize>;

    /// Vertex cover using maximal matching (also 2-approx)
    pub fn vertex_cover_matching(adj: &[Vec<usize>]) -> Vec<usize>;

    /// O(log n) approximation for set cover
    pub fn set_cover_greedy(
        universe: usize,
        sets: &[Vec<usize>],
    ) -> Vec<usize>;  // Returns indices of chosen sets

    /// Weighted set cover
    pub fn weighted_set_cover(
        universe: usize,
        sets: &[(Vec<usize>, f64)],  // (elements, cost)
    ) -> Vec<usize>;

    /// 2-approximation for TSP with triangle inequality
    pub fn tsp_mst_approx(dist: &[Vec<i64>]) -> (i64, Vec<usize>);

    /// Christofides' 1.5-approximation for metric TSP
    pub fn tsp_christofides(dist: &[Vec<i64>]) -> (i64, Vec<usize>);

    /// 2-approximation for maximum cut
    pub fn max_cut_random(adj: &[Vec<(usize, i64)>]) -> (i64, Vec<bool>);

    /// Greedy 2-approximation for max cut
    pub fn max_cut_greedy(adj: &[Vec<(usize, i64)>]) -> (i64, Vec<bool>);

    /// (1-1/e) approximation for max coverage
    pub fn max_coverage(
        sets: &[Vec<usize>],
        k: usize,
    ) -> Vec<usize>;

    /// Load balancing on parallel machines
    pub fn load_balance_greedy(jobs: &[i64], machines: usize) -> Vec<Vec<usize>>;

    /// LPT (Longest Processing Time) for load balancing
    pub fn load_balance_lpt(jobs: &[i64], machines: usize) -> Vec<Vec<usize>>;
}

pub mod knapsack_approx {
    /// FPTAS for 0/1 Knapsack
    /// Returns (1-ε)-approximation in O(n³/ε) time
    pub fn knapsack_fptas(
        weights: &[usize],
        values: &[i64],
        capacity: usize,
        epsilon: f64,
    ) -> i64;

    /// 2-approximation using greedy (by value/weight ratio)
    pub fn knapsack_greedy_approx(
        weights: &[usize],
        values: &[i64],
        capacity: usize,
    ) -> i64;

    /// Dynamic programming scaling approach
    pub fn knapsack_scaled_dp(
        weights: &[usize],
        values: &[i64],
        capacity: usize,
        scale: usize,
    ) -> i64;
}

pub mod scheduling {
    /// Greedy scheduling for makespan minimization
    pub fn minimize_makespan_greedy(
        jobs: &[i64],
        machines: usize,
    ) -> i64;

    /// PTAS for scheduling on identical machines
    pub fn makespan_ptas(
        jobs: &[i64],
        machines: usize,
        epsilon: f64,
    ) -> i64;

    /// Job shop scheduling approximation
    pub fn job_shop_approx(
        jobs: &[Vec<(usize, i64)>],  // jobs[i] = [(machine, duration)]
        machines: usize,
    ) -> i64;

    /// Weighted completion time minimization
    pub fn weighted_completion(
        processing: &[i64],
        weights: &[i64],
    ) -> (i64, Vec<usize>);  // Total weighted completion, order
}

pub mod facility_location {
    /// Greedy facility location
    pub fn facility_location_greedy(
        facility_costs: &[f64],
        connection_costs: &[Vec<f64>],
    ) -> Vec<usize>;

    /// K-median approximation
    pub fn k_median(
        distances: &[Vec<f64>],
        k: usize,
    ) -> Vec<usize>;

    /// K-center approximation (2-approx)
    pub fn k_center(
        distances: &[Vec<f64>],
        k: usize,
    ) -> Vec<usize>;
}

pub mod bin_packing {
    /// First Fit Decreasing for bin packing
    pub fn bin_packing_ffd(items: &[f64], bin_capacity: f64) -> Vec<Vec<usize>>;

    /// Next Fit for online bin packing
    pub fn bin_packing_next_fit(items: &[f64], bin_capacity: f64) -> Vec<Vec<usize>>;

    /// Best Fit Decreasing
    pub fn bin_packing_bfd(items: &[f64], bin_capacity: f64) -> Vec<Vec<usize>>;

    /// AFPTAS for bin packing
    pub fn bin_packing_afptas(
        items: &[f64],
        bin_capacity: f64,
        epsilon: f64,
    ) -> usize;  // Number of bins
}

pub mod analysis {
    /// Compute approximation ratio given algorithm output and optimal
    pub fn approximation_ratio(alg: f64, opt: f64, maximize: bool) -> f64;

    /// Verify approximation guarantee
    pub fn verify_approximation<F, G>(
        algorithm: F,
        optimal: G,
        inputs: &[Vec<i64>],
        expected_ratio: f64,
        maximize: bool,
    ) -> bool
    where
        F: Fn(&[i64]) -> i64,
        G: Fn(&[i64]) -> i64;
}
```

### Python Implementation

```python
from typing import List, Tuple

def vertex_cover_2approx(adj: List[List[int]]) -> List[int]: ...
def set_cover_greedy(universe: int, sets: List[List[int]]) -> List[int]: ...
def tsp_mst_approx(dist: List[List[int]]) -> Tuple[int, List[int]]: ...
def max_cut_greedy(adj: List[List[Tuple[int, int]]]) -> Tuple[int, List[bool]]: ...

def knapsack_fptas(weights: List[int], values: List[int], capacity: int, epsilon: float) -> int: ...

def minimize_makespan_greedy(jobs: List[int], machines: int) -> int: ...

def k_center(distances: List[List[float]], k: int) -> List[int]: ...

def bin_packing_ffd(items: List[float], capacity: float) -> List[List[int]]: ...
```

## Test Cases

```rust
#[test]
fn test_vertex_cover() {
    // Triangle
    let adj = vec![vec![1, 2], vec![0, 2], vec![0, 1]];
    let cover = vertex_cover_2approx(&adj);

    // Verify it's a valid cover
    for u in 0..adj.len() {
        for &v in &adj[u] {
            assert!(cover.contains(&u) || cover.contains(&v));
        }
    }

    // Optimal for triangle is 2, so 2-approx should give at most 4
    assert!(cover.len() <= 4);
}

#[test]
fn test_set_cover() {
    // Universe = {0, 1, 2, 3, 4}
    // Sets: {0,1,2}, {2,3}, {3,4}, {0,4}
    let sets = vec![
        vec![0, 1, 2],
        vec![2, 3],
        vec![3, 4],
        vec![0, 4],
    ];

    let chosen = set_cover_greedy(5, &sets);

    // Verify cover
    let mut covered = vec![false; 5];
    for &idx in &chosen {
        for &elem in &sets[idx] {
            covered[elem] = true;
        }
    }
    assert!(covered.iter().all(|&x| x));

    // Optimal is 2 ({0,1,2} and {3,4}), greedy gives O(log n) approx
}

#[test]
fn test_tsp_mst() {
    // Square with diagonals (metric)
    let dist = vec![
        vec![0, 1, 2, 1],
        vec![1, 0, 1, 2],
        vec![2, 1, 0, 1],
        vec![1, 2, 1, 0],
    ];

    let (cost, tour) = tsp_mst_approx(&dist);

    // Verify valid tour
    assert_eq!(tour.len(), 5);  // Starts and ends at same
    assert_eq!(tour[0], tour[4]);

    // Optimal is 4 (square), 2-approx gives at most 8
    assert!(cost <= 8);
}

#[test]
fn test_max_cut() {
    // Complete graph K4 with unit weights
    let adj = vec![
        vec![(1, 1), (2, 1), (3, 1)],
        vec![(0, 1), (2, 1), (3, 1)],
        vec![(0, 1), (1, 1), (3, 1)],
        vec![(0, 1), (1, 1), (2, 1)],
    ];

    let (cut_value, partition) = max_cut_greedy(&adj);

    // For K4, max cut is 4 (2-2 split), random gives at least 2 on average
    assert!(cut_value >= 2);
}

#[test]
fn test_knapsack_fptas() {
    let weights = vec![2, 3, 4, 5];
    let values = vec![3, 4, 5, 6];
    let capacity = 5;

    let optimal = 7;  // Items 0 and 1
    let approx = knapsack_fptas(&weights, &values, capacity, 0.1);

    // Should be at least (1-0.1)*7 = 6.3
    assert!(approx >= 6);
}

#[test]
fn test_makespan() {
    let jobs = vec![10, 10, 10, 10, 10, 10];
    let machines = 3;

    let makespan = minimize_makespan_greedy(&jobs, machines);

    // Optimal is 20 (2 jobs per machine), greedy is at most 2*opt = 40
    assert!(makespan <= 40);
    // LPT should achieve optimal here
}

#[test]
fn test_k_center() {
    // 6 points in a line: 0, 1, 2, 3, 4, 5
    let n = 6;
    let mut dist = vec![vec![0.0; n]; n];
    for i in 0..n {
        for j in 0..n {
            dist[i][j] = (i as f64 - j as f64).abs();
        }
    }

    let centers = k_center(&dist, 2);
    assert_eq!(centers.len(), 2);

    // Maximum distance to nearest center should be at most 2*optimal
    // Optimal with 2 centers is to place at 1 and 4, max dist = 1.5
    let max_dist: f64 = (0..n)
        .map(|i| centers.iter().map(|&c| dist[i][c]).fold(f64::MAX, f64::min))
        .fold(0.0, f64::max);

    assert!(max_dist <= 3.0);  // 2 * 1.5
}

#[test]
fn test_bin_packing_ffd() {
    let items = vec![0.5, 0.7, 0.3, 0.2, 0.8, 0.4, 0.1];
    let capacity = 1.0;

    let bins = bin_packing_ffd(&items, capacity);

    // Verify each bin doesn't exceed capacity
    for bin in &bins {
        let total: f64 = bin.iter().map(|&i| items[i]).sum();
        assert!(total <= capacity + 1e-9);
    }

    // FFD uses at most 11/9 * OPT + 6/9 bins
}

#[test]
fn test_load_balance_lpt() {
    let jobs = vec![10, 8, 7, 6, 5, 4, 3, 2, 1];
    let machines = 3;

    let assignment = load_balance_lpt(&jobs, machines);

    let makespans: Vec<i64> = assignment
        .iter()
        .map(|m| m.iter().map(|&j| jobs[j]).sum())
        .collect();

    let max_makespan = *makespans.iter().max().unwrap();

    // LPT gives 4/3 - 1/(3m) approximation
    // Optimal is ceil(46/3) = 16, so LPT should give at most ~21
    assert!(max_makespan <= 21);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Vertex cover | 15 |
| Set cover | 15 |
| TSP approximation | 15 |
| Max cut | 10 |
| Knapsack FPTAS | 15 |
| Bin packing | 15 |
| Scheduling | 10 |
| Edge cases | 5 |
| **Total** | **100** |
