# Exercise 06: Suffix Array & Suffix Tree

## Concepts Covered
- **1.2.19.i** Pattern matching with suffix array
- **1.2.19.j** LCP array
- **1.2.19.k** Applications
- **1.2.20.g-k** Suffix tree operations

## Objective

Implement suffix array and suffix tree for advanced string processing.

## Requirements

### Rust Implementation

```rust
pub mod suffix_structures {
    /// Suffix Array with LCP
    pub struct SuffixArray {
        sa: Vec<usize>,      // Suffix array
        rank: Vec<usize>,    // Inverse of sa
        lcp: Vec<usize>,     // LCP array
        text: Vec<u8>,
    }

    impl SuffixArray {
        /// Build suffix array using DC3/Skew algorithm - O(n)
        pub fn new(text: &[u8]) -> Self;

        /// Build using simple O(n log^2 n) algorithm
        pub fn new_simple(text: &[u8]) -> Self;

        /// Get suffix at position i
        pub fn suffix(&self, i: usize) -> &[u8];

        /// Pattern matching - returns range in SA
        pub fn search(&self, pattern: &[u8]) -> Option<(usize, usize)>;

        /// Find all occurrences of pattern
        pub fn find_all(&self, pattern: &[u8]) -> Vec<usize>;

        /// Count occurrences
        pub fn count(&self, pattern: &[u8]) -> usize;

        /// Longest Common Substring of text and pattern
        pub fn lcs_with(&self, other: &[u8]) -> (usize, usize, usize);

        /// Number of distinct substrings
        pub fn count_distinct_substrings(&self) -> usize;

        /// Longest repeated substring
        pub fn longest_repeated_substring(&self) -> &[u8];

        /// Kth smallest substring
        pub fn kth_substring(&self, k: usize) -> Option<Vec<u8>>;
    }

    /// Sparse Table for LCP queries
    pub struct LcpRmq {
        sparse: Vec<Vec<usize>>,
        log: Vec<usize>,
    }

    impl LcpRmq {
        pub fn new(lcp: &[usize]) -> Self;

        /// LCP of suffixes at positions i and j in O(1)
        pub fn query(&self, i: usize, j: usize) -> usize;
    }

    /// Suffix Tree using Ukkonen's algorithm
    pub struct SuffixTree {
        nodes: Vec<SuffixTreeNode>,
        text: Vec<u8>,
    }

    struct SuffixTreeNode {
        children: std::collections::HashMap<u8, usize>,
        suffix_link: Option<usize>,
        start: usize,
        end: Option<usize>,  // None means end of text
        suffix_index: Option<usize>,
    }

    impl SuffixTree {
        /// Build suffix tree - O(n)
        pub fn new(text: &[u8]) -> Self;

        /// Check if pattern exists
        pub fn contains(&self, pattern: &[u8]) -> bool;

        /// Find all occurrences
        pub fn find_all(&self, pattern: &[u8]) -> Vec<usize>;

        /// Longest common substring of two strings
        pub fn longest_common_substring(s1: &[u8], s2: &[u8]) -> Vec<u8>;

        /// Longest repeated substring
        pub fn longest_repeated(&self) -> Vec<u8>;

        /// Count leaves in subtree (occurrences)
        pub fn count_pattern(&self, pattern: &[u8]) -> usize;
    }

    // Applications

    /// Longest Common Substring of multiple strings
    pub fn lcs_multiple(strings: &[&[u8]]) -> Vec<u8>;

    /// Shortest unique substring starting at each position
    pub fn shortest_unique_substrings(text: &[u8]) -> Vec<usize>;

    /// Burrows-Wheeler Transform
    pub fn bwt(text: &[u8]) -> Vec<u8>;

    /// Inverse BWT
    pub fn inverse_bwt(bwt: &[u8]) -> Vec<u8>;
}
```

### Python Implementation

```python
class SuffixArray:
    def __init__(self, text: bytes) -> None: ...
    def suffix(self, i: int) -> bytes: ...
    def search(self, pattern: bytes) -> tuple[int, int] | None: ...
    def find_all(self, pattern: bytes) -> list[int]: ...
    def count(self, pattern: bytes) -> int: ...
    def count_distinct_substrings(self) -> int: ...
    def longest_repeated_substring(self) -> bytes: ...

class LcpRmq:
    def __init__(self, lcp: list[int]) -> None: ...
    def query(self, i: int, j: int) -> int: ...

class SuffixTree:
    def __init__(self, text: bytes) -> None: ...
    def contains(self, pattern: bytes) -> bool: ...
    def find_all(self, pattern: bytes) -> list[int]: ...
    @staticmethod
    def longest_common_substring(s1: bytes, s2: bytes) -> bytes: ...

def lcs_multiple(strings: list[bytes]) -> bytes: ...
def bwt(text: bytes) -> bytes: ...
def inverse_bwt(bwt: bytes) -> bytes: ...
```

## Algorithm Details

### Suffix Array Construction (Simple O(n log² n))
```rust
fn build_suffix_array(s: &[u8]) -> Vec<usize> {
    let n = s.len();
    let mut sa: Vec<usize> = (0..n).collect();
    let mut rank: Vec<usize> = s.iter().map(|&c| c as usize).collect();
    let mut tmp = vec![0; n];

    let mut k = 1;
    while k < n {
        // Sort by (rank[i], rank[i+k])
        sa.sort_by_key(|&i| {
            (rank[i], rank.get(i + k).copied().unwrap_or(0))
        });

        // Compute new ranks
        tmp[sa[0]] = 0;
        for i in 1..n {
            tmp[sa[i]] = tmp[sa[i - 1]];
            if (rank[sa[i]], rank.get(sa[i] + k).copied().unwrap_or(0))
                > (rank[sa[i - 1]], rank.get(sa[i - 1] + k).copied().unwrap_or(0))
            {
                tmp[sa[i]] += 1;
            }
        }
        std::mem::swap(&mut rank, &mut tmp);

        if rank[sa[n - 1]] == n - 1 {
            break;
        }
        k *= 2;
    }
    sa
}
```

### LCP Array (Kasai's Algorithm O(n))
```rust
fn build_lcp(text: &[u8], sa: &[usize], rank: &[usize]) -> Vec<usize> {
    let n = text.len();
    let mut lcp = vec![0; n];
    let mut k = 0;

    for i in 0..n {
        if rank[i] == 0 {
            k = 0;
            continue;
        }
        let j = sa[rank[i] - 1];
        while i + k < n && j + k < n && text[i + k] == text[j + k] {
            k += 1;
        }
        lcp[rank[i]] = k;
        if k > 0 {
            k -= 1;
        }
    }
    lcp
}
```

## Test Cases

```rust
#[test]
fn test_suffix_array_construction() {
    let sa = SuffixArray::new(b"banana");
    // Suffixes sorted: a, ana, anana, banana, na, nana
    // Positions: 5, 3, 1, 0, 4, 2
    assert_eq!(sa.sa, vec![5, 3, 1, 0, 4, 2]);
}

#[test]
fn test_pattern_search() {
    let sa = SuffixArray::new(b"abracadabra");

    assert_eq!(sa.count(b"abra"), 2);
    assert_eq!(sa.find_all(b"abra"), vec![0, 7]);
    assert_eq!(sa.count(b"xyz"), 0);
}

#[test]
fn test_lcp_queries() {
    let sa = SuffixArray::new(b"banana");
    let rmq = LcpRmq::new(&sa.lcp);

    // LCP of "ana" and "anana"
    let i = sa.rank[3];  // "ana"
    let j = sa.rank[1];  // "anana"
    assert_eq!(rmq.query(i.min(j), i.max(j)), 3);
}

#[test]
fn test_distinct_substrings() {
    let sa = SuffixArray::new(b"abab");
    // Substrings: a, ab, aba, abab, b, ba, bab
    assert_eq!(sa.count_distinct_substrings(), 7);
}

#[test]
fn test_longest_repeated() {
    let sa = SuffixArray::new(b"abracadabra");
    assert_eq!(sa.longest_repeated_substring(), b"abra");
}

#[test]
fn test_suffix_tree() {
    let st = SuffixTree::new(b"banana$");

    assert!(st.contains(b"ana"));
    assert!(st.contains(b"nan"));
    assert!(!st.contains(b"xyz"));
    assert_eq!(st.count_pattern(b"an"), 2);
}

#[test]
fn test_lcs() {
    let lcs = SuffixTree::longest_common_substring(b"abcdef", b"zbcdf");
    assert_eq!(lcs, b"bcd");
}

#[test]
fn test_bwt() {
    let text = b"banana$";
    let transformed = bwt(text);
    assert_eq!(transformed, b"annb$aa");

    let restored = inverse_bwt(&transformed);
    assert_eq!(restored, text.to_vec());
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Suffix Array construction | 20 |
| LCP array (Kasai) | 15 |
| Pattern search | 10 |
| LCP RMQ queries | 10 |
| Distinct substrings | 10 |
| Suffix Tree (basic) | 15 |
| LCS using suffix structures | 10 |
| BWT and inverse | 10 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `suffix_structures.py`
