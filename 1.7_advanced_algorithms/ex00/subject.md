# Exercise 00: Greedy Algorithms

## Concepts Covered
- **1.7.1.f-j** Job scheduling, interval variants, gas station, jump game, task assignment

## Objective

Master greedy algorithm design and prove correctness through exchange arguments.

## Requirements

### Rust Implementation

```rust
pub mod greedy {
    /// Activity selection: max non-overlapping intervals
    pub fn activity_selection(intervals: &[(i32, i32)]) -> Vec<usize>;

    /// Weighted activity selection (with DP)
    pub fn weighted_activity_selection(intervals: &[(i32, i32, i32)]) -> i32;

    /// Job scheduling with deadlines: max profit
    /// jobs: (deadline, profit)
    pub fn job_scheduling(jobs: &[(usize, i32)]) -> (i32, Vec<usize>);

    /// Job scheduling with processing times: minimize completion time
    /// Jobs: processing_time
    pub fn shortest_job_first(jobs: &[i32]) -> (i64, Vec<usize>);

    /// Meeting rooms: minimum rooms needed
    pub fn min_meeting_rooms(intervals: &[(i32, i32)]) -> usize;

    /// Merge intervals
    pub fn merge_intervals(intervals: &[(i32, i32)]) -> Vec<(i32, i32)>;

    /// Insert interval
    pub fn insert_interval(intervals: &[(i32, i32)], new: (i32, i32)) -> Vec<(i32, i32)>;

    /// Gas station: can complete circular route?
    pub fn can_complete_circuit(gas: &[i32], cost: &[i32]) -> i32;

    /// Jump game: minimum jumps
    pub fn min_jumps(nums: &[i32]) -> i32;

    /// Task assignment: minimize total time with 2 workers
    pub fn two_city_scheduling(costs: &[(i32, i32)]) -> i32;

    /// Partition labels: partition string so each letter appears in one part
    pub fn partition_labels(s: &str) -> Vec<usize>;

    /// Candy distribution: each child gets at least 1, more than neighbors if higher rating
    pub fn candy(ratings: &[i32]) -> i32;

    /// Queue reconstruction by height
    pub fn reconstruct_queue(people: &[(i32, i32)]) -> Vec<(i32, i32)>;

    /// Huffman coding
    pub fn huffman_codes(freqs: &[(char, u32)]) -> std::collections::HashMap<char, String>;

    /// Fractional knapsack
    pub fn fractional_knapsack(items: &[(f64, f64)], capacity: f64) -> f64;
}
```

### Python Implementation

```python
def activity_selection(intervals: list[tuple[int, int]]) -> list[int]: ...
def weighted_activity_selection(intervals: list[tuple[int, int, int]]) -> int: ...
def job_scheduling(jobs: list[tuple[int, int]]) -> tuple[int, list[int]]: ...
def min_meeting_rooms(intervals: list[tuple[int, int]]) -> int: ...
def merge_intervals(intervals: list[tuple[int, int]]) -> list[tuple[int, int]]: ...
def can_complete_circuit(gas: list[int], cost: list[int]) -> int: ...
def min_jumps(nums: list[int]) -> int: ...
def candy(ratings: list[int]) -> int: ...
def huffman_codes(freqs: list[tuple[str, int]]) -> dict[str, str]: ...
def fractional_knapsack(items: list[tuple[float, float]], capacity: float) -> float: ...
```

## Greedy Proofs

### Exchange Argument
To prove greedy choice is optimal:
1. Assume optimal solution OPT
2. If OPT uses greedy choice, done
3. Otherwise, show we can modify OPT to use greedy choice without worsening

### Greedy Stays Ahead
1. Define measure of progress
2. Show greedy is never behind optimal at any step
3. Conclude greedy achieves at least as good result

## Test Cases

```rust
#[test]
fn test_activity_selection() {
    let intervals = vec![(1, 4), (3, 5), (0, 6), (5, 7), (3, 9), (5, 9), (6, 10), (8, 11), (8, 12), (2, 14), (12, 16)];
    let selected = activity_selection(&intervals);
    // Should select non-overlapping intervals
    assert_eq!(selected.len(), 4);
}

#[test]
fn test_job_scheduling() {
    let jobs = vec![(4, 20), (1, 10), (1, 40), (1, 30)];
    let (profit, _) = job_scheduling(&jobs);
    assert_eq!(profit, 60);  // Jobs with profit 40 and 20
}

#[test]
fn test_meeting_rooms() {
    let intervals = vec![(0, 30), (5, 10), (15, 20)];
    assert_eq!(min_meeting_rooms(&intervals), 2);

    let intervals = vec![(7, 10), (2, 4)];
    assert_eq!(min_meeting_rooms(&intervals), 1);
}

#[test]
fn test_gas_station() {
    assert_eq!(can_complete_circuit(&[1, 2, 3, 4, 5], &[3, 4, 5, 1, 2]), 3);
    assert_eq!(can_complete_circuit(&[2, 3, 4], &[3, 4, 3]), -1);
}

#[test]
fn test_candy() {
    assert_eq!(candy(&[1, 0, 2]), 5);
    assert_eq!(candy(&[1, 2, 2]), 4);
}

#[test]
fn test_partition_labels() {
    assert_eq!(partition_labels("ababcbacadefegdehijhklij"), vec![9, 7, 8]);
}

#[test]
fn test_huffman() {
    let freqs = vec![('a', 5), ('b', 9), ('c', 12), ('d', 13), ('e', 16), ('f', 45)];
    let codes = huffman_codes(&freqs);

    // Verify prefix-free property
    for (c1, code1) in &codes {
        for (c2, code2) in &codes {
            if c1 != c2 {
                assert!(!code1.starts_with(code2));
                assert!(!code2.starts_with(code1));
            }
        }
    }
}

#[test]
fn test_fractional_knapsack() {
    let items = vec![(60.0, 10.0), (100.0, 20.0), (120.0, 30.0)];  // (value, weight)
    let result = fractional_knapsack(&items, 50.0);
    assert!((result - 240.0).abs() < 0.001);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Activity selection | 15 |
| Job scheduling | 15 |
| Meeting rooms | 10 |
| Merge/Insert intervals | 10 |
| Gas station | 10 |
| Candy distribution | 10 |
| Huffman coding | 15 |
| Fractional knapsack | 10 |
| Edge cases | 5 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `greedy.py`
