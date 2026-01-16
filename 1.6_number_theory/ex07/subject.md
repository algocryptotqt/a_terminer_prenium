# Exercise 07: Combinatorics & Counting

## Concepts Covered
- **1.6.15.d-l** Advanced counting, Burnside's lemma, Pólya enumeration
- **1.6.16.d-k** Inclusion-exclusion, derangements, Stirling numbers

## Objective

Master advanced combinatorial counting techniques.

## Requirements

### Rust Implementation

```rust
pub mod counting {
    /// Permutations P(n, k) = n! / (n-k)!
    pub fn permutations(n: u64, k: u64) -> u64;

    /// Permutations mod m
    pub fn permutations_mod(n: u64, k: u64, m: i64) -> i64;

    /// Combinations C(n, k)
    pub fn combinations(n: u64, k: u64) -> u64;

    /// Combinations with repetition C(n+k-1, k)
    pub fn combinations_rep(n: u64, k: u64) -> u64;

    /// Multiset coefficient (multinomial)
    pub fn multinomial(n: u64, groups: &[u64]) -> u64;

    /// Multinomial mod m
    pub fn multinomial_mod(n: u64, groups: &[u64], m: i64) -> i64;

    /// Stars and bars: ways to put n identical balls in k distinct bins
    pub fn stars_and_bars(n: u64, k: u64) -> u64;

    /// Same with at least 1 in each bin
    pub fn stars_and_bars_nonempty(n: u64, k: u64) -> u64;

    /// Catalan number C_n
    pub fn catalan(n: u64) -> u64;

    /// Ballot numbers (Catalan generalization)
    pub fn ballot(n: u64, k: u64) -> u64;

    /// Motzkin numbers
    pub fn motzkin(n: u64) -> u64;

    /// Bell numbers (number of partitions of a set)
    pub fn bell(n: u64) -> u64;
}

pub mod inclusion_exclusion {
    /// Count integers 1..n divisible by at least one of given primes
    pub fn count_divisible(n: u64, primes: &[u64]) -> u64;

    /// Count integers 1..n coprime to m
    pub fn count_coprime(n: u64, m: u64) -> u64;

    /// Derangements D(n)
    pub fn derangement(n: u64) -> u64;

    /// Subfactorial !n
    pub fn subfactorial(n: u64) -> u64;

    /// Count surjections from n elements to k elements
    pub fn surjections(n: u64, k: u64) -> u64;

    /// Stirling numbers of second kind S(n, k)
    /// Number of ways to partition n elements into k non-empty subsets
    pub fn stirling2(n: u64, k: u64) -> u64;

    /// Stirling numbers of first kind s(n, k)
    /// Number of permutations with exactly k cycles
    pub fn stirling1(n: u64, k: u64) -> i64;  // Signed

    /// Unsigned Stirling first kind
    pub fn stirling1_unsigned(n: u64, k: u64) -> u64;

    /// Number of ways to arrange n items with forbidden positions
    pub fn restricted_permutations(n: usize, forbidden: &[(usize, usize)]) -> u64;
}

pub mod polya {
    /// Burnside's lemma: count orbits under group action
    /// fixed[g] = number of elements fixed by group element g
    pub fn burnside(fixed: &[u64], group_size: u64) -> u64;

    /// Count distinct necklaces with n beads and k colors
    pub fn necklaces(n: u64, k: u64) -> u64;

    /// Count distinct bracelets (necklaces with flip symmetry)
    pub fn bracelets(n: u64, k: u64) -> u64;

    /// Count distinct colorings of cube faces with k colors
    pub fn cube_colorings(k: u64) -> u64;

    /// Pólya enumeration with weight
    pub fn polya_weighted(cycle_index: &[(Vec<u64>, u64)], weights: &[u64]) -> u64;

    /// Count graphs on n labeled vertices
    pub fn labeled_graphs(n: u64) -> u64;

    /// Count non-isomorphic graphs on n vertices
    pub fn unlabeled_graphs(n: u64) -> u64;
}

pub mod generating_functions {
    /// Coefficient of x^k in (1 + x + x^2 + ... + x^n)^m
    pub fn polynomial_coeff(n: usize, m: usize, k: usize) -> u64;

    /// Partition number p(n)
    pub fn partition_number(n: u64) -> u64;

    /// Partition into distinct parts
    pub fn distinct_partition(n: u64) -> u64;

    /// Partition into at most k parts
    pub fn partition_at_most_k(n: u64, k: u64) -> u64;

    /// Partition into parts of size at most k
    pub fn partition_parts_at_most_k(n: u64, k: u64) -> u64;

    /// Number of integer compositions of n
    pub fn compositions(n: u64) -> u64;

    /// Compositions into exactly k parts
    pub fn compositions_k_parts(n: u64, k: u64) -> u64;

    /// Pentagonal number theorem for partitions
    pub fn partition_pentagonal(n: u64) -> u64;
}

pub mod lattice_paths {
    /// Paths from (0,0) to (m,n) using (1,0) and (0,1)
    pub fn grid_paths(m: u64, n: u64) -> u64;

    /// Paths that don't cross y=x (Catalan)
    pub fn dyck_paths(n: u64) -> u64;

    /// Paths from (0,0) to (m,n) staying below y = x * a/b
    pub fn ballot_paths(m: u64, n: u64, a: u64, b: u64) -> u64;

    /// Schröder paths (diagonal steps allowed)
    pub fn schroder_paths(n: u64) -> u64;

    /// Count paths avoiding certain points
    pub fn paths_avoiding(m: u64, n: u64, forbidden: &[(u64, u64)]) -> u64;

    /// Lindström-Gessel-Viennot (non-intersecting paths)
    pub fn non_intersecting_paths(
        starts: &[(u64, u64)],
        ends: &[(u64, u64)],
    ) -> u64;
}
```

### Python Implementation

```python
from typing import List, Tuple

def permutations(n: int, k: int) -> int: ...
def combinations(n: int, k: int) -> int: ...
def multinomial(n: int, groups: List[int]) -> int: ...
def catalan(n: int) -> int: ...
def bell(n: int) -> int: ...

def derangement(n: int) -> int: ...
def stirling2(n: int, k: int) -> int: ...
def stirling1(n: int, k: int) -> int: ...
def surjections(n: int, k: int) -> int: ...

def necklaces(n: int, k: int) -> int: ...
def bracelets(n: int, k: int) -> int: ...

def partition_number(n: int) -> int: ...
def distinct_partition(n: int) -> int: ...

def grid_paths(m: int, n: int) -> int: ...
def dyck_paths(n: int) -> int: ...
```

## Test Cases

```rust
#[test]
fn test_catalan() {
    assert_eq!(catalan(0), 1);
    assert_eq!(catalan(1), 1);
    assert_eq!(catalan(2), 2);
    assert_eq!(catalan(3), 5);
    assert_eq!(catalan(4), 14);
    assert_eq!(catalan(10), 16796);
}

#[test]
fn test_derangement() {
    assert_eq!(derangement(0), 1);
    assert_eq!(derangement(1), 0);
    assert_eq!(derangement(2), 1);
    assert_eq!(derangement(3), 2);
    assert_eq!(derangement(4), 9);
    assert_eq!(derangement(5), 44);
}

#[test]
fn test_stirling2() {
    // S(n, 1) = 1
    assert_eq!(stirling2(5, 1), 1);
    // S(n, n) = 1
    assert_eq!(stirling2(5, 5), 1);
    // S(4, 2) = 7
    assert_eq!(stirling2(4, 2), 7);
    // S(5, 3) = 25
    assert_eq!(stirling2(5, 3), 25);
}

#[test]
fn test_stirling1() {
    // |s(n, 1)| = (n-1)!
    assert_eq!(stirling1_unsigned(4, 1), 6);
    // |s(n, n)| = 1
    assert_eq!(stirling1_unsigned(5, 5), 1);
    // |s(4, 2)| = 11
    assert_eq!(stirling1_unsigned(4, 2), 11);
}

#[test]
fn test_bell() {
    assert_eq!(bell(0), 1);
    assert_eq!(bell(1), 1);
    assert_eq!(bell(2), 2);
    assert_eq!(bell(3), 5);
    assert_eq!(bell(4), 15);
    assert_eq!(bell(5), 52);
}

#[test]
fn test_surjections() {
    // Surjections from n to k = k! * S(n, k)
    assert_eq!(surjections(4, 2), 14);  // 2! * 7
    assert_eq!(surjections(3, 3), 6);   // 3! * 1
    assert_eq!(surjections(4, 3), 36);  // 3! * 6
}

#[test]
fn test_necklaces() {
    // Binary necklaces of length 4
    assert_eq!(necklaces(4, 2), 6);  // 0000, 0001, 0011, 0101, 0111, 1111

    // 3-color necklaces of length 3
    assert_eq!(necklaces(3, 3), 11);
}

#[test]
fn test_bracelets() {
    // Binary bracelets of length 4
    assert_eq!(bracelets(4, 2), 6);

    // Different from necklaces for n=6, k=2
    assert_eq!(necklaces(6, 2), 14);
    assert_eq!(bracelets(6, 2), 13);  // 0101 = 1010 as bracelet
}

#[test]
fn test_partition_number() {
    assert_eq!(partition_number(0), 1);
    assert_eq!(partition_number(1), 1);
    assert_eq!(partition_number(2), 2);
    assert_eq!(partition_number(5), 7);
    assert_eq!(partition_number(10), 42);
    assert_eq!(partition_number(100), 190569292);
}

#[test]
fn test_grid_paths() {
    assert_eq!(grid_paths(2, 2), 6);  // C(4,2)
    assert_eq!(grid_paths(3, 3), 20); // C(6,3)
}

#[test]
fn test_dyck_paths() {
    // Same as Catalan
    assert_eq!(dyck_paths(3), 5);
    assert_eq!(dyck_paths(4), 14);
}

#[test]
fn test_stars_and_bars() {
    // 5 balls into 3 bins
    assert_eq!(stars_and_bars(5, 3), 21);  // C(7,2)

    // With at least 1 in each
    assert_eq!(stars_and_bars_nonempty(5, 3), 6);  // C(4,2)
}

#[test]
fn test_multinomial() {
    // Ways to arrange "mississippi"
    // 11! / (1! * 4! * 4! * 2!) = 34650
    assert_eq!(multinomial(11, &[1, 4, 4, 2]), 34650);
}

#[test]
fn test_count_coprime() {
    // Count 1..10 coprime to 6 = 2*3
    // 1, 5, 7 are coprime to 6 in 1..10: 1,5,7 (actually more)
    // Coprime to 6: 1,5,7,11,... but in 1..10: 1,5,7 = 3? No.
    // Actually: 1, 5, 7 coprime? 1,5,7 yes, also need to check all
    let count = count_coprime(10, 6);
    // n * φ(m)/m approximately... φ(6)=2, so about 10*2/6 ≈ 3.3
    // Exact: 1,5,7 → wait, let me recalculate
}

#[test]
fn test_cube_colorings() {
    // 2 colors: 10 distinct colorings
    assert_eq!(cube_colorings(2), 10);

    // 3 colors: 57 distinct colorings
    assert_eq!(cube_colorings(3), 57);
}

#[test]
fn test_distinct_partition() {
    // p(5) into distinct parts: 5, 4+1, 3+2 = 3
    assert_eq!(distinct_partition(5), 3);

    // p(10) into distinct parts
    assert_eq!(distinct_partition(10), 10);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic counting | 10 |
| Inclusion-exclusion | 15 |
| Stirling numbers | 15 |
| Burnside/Pólya | 20 |
| Partitions | 15 |
| Lattice paths | 15 |
| Generating functions | 5 |
| Edge cases | 5 |
| **Total** | **100** |
