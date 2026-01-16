# Exercise 05: String Algorithms Advanced

## Concepts Covered
- **1.7.10.d-l** Suffix automaton, palindromic tree
- **1.7.11.d-k** Burrows-Wheeler transform, suffix array advanced

## Objective

Implement advanced string processing data structures.

## Requirements

### Rust Implementation

```rust
pub mod suffix_automaton {
    /// Suffix Automaton (DAWG) - O(n) construction
    pub struct SuffixAutomaton {
        states: Vec<SAState>,
        last: usize,
    }

    pub struct SAState {
        len: usize,
        link: Option<usize>,
        transitions: std::collections::HashMap<char, usize>,
        is_terminal: bool,
    }

    impl SuffixAutomaton {
        pub fn new() -> Self;

        /// Build from string - O(n)
        pub fn build(s: &str) -> Self;

        /// Extend by one character
        pub fn extend(&mut self, c: char);

        /// Check if pattern is substring - O(|pattern|)
        pub fn contains(&self, pattern: &str) -> bool;

        /// Count occurrences of pattern
        pub fn count_occurrences(&self, pattern: &str) -> usize;

        /// Find all occurrences positions
        pub fn find_all(&self, pattern: &str) -> Vec<usize>;

        /// Count distinct substrings
        pub fn count_distinct_substrings(&self) -> usize;

        /// Sum of lengths of all distinct substrings
        pub fn sum_substring_lengths(&self) -> usize;

        /// K-th lexicographically smallest substring
        pub fn kth_substring(&self, k: usize) -> Option<String>;

        /// Longest common substring of two strings
        pub fn longest_common_substring(s1: &str, s2: &str) -> String;

        /// Number of different substrings of length k
        pub fn substrings_of_length(&self, k: usize) -> usize;
    }
}

pub mod palindromic_tree {
    /// Eertree / Palindromic Tree
    pub struct PalindromicTree {
        nodes: Vec<PTNode>,
        last: usize,
        s: Vec<char>,
    }

    pub struct PTNode {
        len: i32,
        link: usize,
        transitions: std::collections::HashMap<char, usize>,
        count: usize,  // Number of times this palindrome occurs
    }

    impl PalindromicTree {
        pub fn new() -> Self;

        /// Build from string
        pub fn build(s: &str) -> Self;

        /// Add character
        pub fn add(&mut self, c: char);

        /// Count distinct palindromic substrings
        pub fn count_palindromes(&self) -> usize;

        /// Get all palindromic substrings
        pub fn all_palindromes(&self) -> Vec<String>;

        /// Longest palindromic substring
        pub fn longest_palindrome(&self) -> String;

        /// Number of palindrome occurrences
        pub fn total_palindrome_occurrences(&self) -> usize;

        /// K-th lexicographically smallest palindrome
        pub fn kth_palindrome(&self, k: usize) -> Option<String>;
    }
}

pub mod bwt {
    /// Burrows-Wheeler Transform
    pub fn bwt(s: &str) -> (String, usize);

    /// Inverse BWT
    pub fn inverse_bwt(bwt: &str, primary_index: usize) -> String;

    /// BWT using suffix array (efficient)
    pub fn bwt_sa(s: &str) -> String;

    /// FM-Index for pattern matching
    pub struct FMIndex {
        bwt: Vec<u8>,
        c: Vec<usize>,      // Count of chars less than c
        occ: Vec<Vec<usize>>, // Occurrence tables
        sa: Vec<usize>,     // Sampled suffix array
    }

    impl FMIndex {
        pub fn new(s: &str) -> Self;

        /// Count occurrences of pattern
        pub fn count(&self, pattern: &str) -> usize;

        /// Find all occurrences
        pub fn locate(&self, pattern: &str) -> Vec<usize>;

        /// Backward search step
        fn backward_search(&self, l: usize, r: usize, c: u8) -> (usize, usize);
    }
}

pub mod suffix_array_advanced {
    /// LCP array from suffix array - O(n)
    pub fn build_lcp(s: &str, sa: &[usize]) -> Vec<usize>;

    /// Suffix array using DC3/Skew algorithm - O(n)
    pub fn suffix_array_dc3(s: &str) -> Vec<usize>;

    /// Enhanced suffix array with child table
    pub struct EnhancedSA {
        sa: Vec<usize>,
        lcp: Vec<usize>,
        child: Vec<(Option<usize>, Option<usize>)>,  // (down, next_l_index)
    }

    impl EnhancedSA {
        pub fn new(s: &str) -> Self;

        /// Count occurrences of pattern - O(|P| + log n)
        pub fn count(&self, s: &str, pattern: &str) -> usize;

        /// Find leftmost occurrence
        pub fn find_first(&self, s: &str, pattern: &str) -> Option<usize>;

        /// Longest repeated substring
        pub fn longest_repeated(&self, s: &str) -> String;

        /// Number of distinct substrings using LCP
        pub fn distinct_substrings(&self, s: &str) -> usize;
    }

    /// Suffix tree from suffix array + LCP
    pub fn build_suffix_tree_from_sa(s: &str, sa: &[usize], lcp: &[usize]) -> SuffixTree;

    /// Longest common extension queries
    pub struct LCEQuery {
        sa: Vec<usize>,
        lcp: Vec<usize>,
        rmq: super::sparse_table::SparseTable,
        rank: Vec<usize>,
    }

    impl LCEQuery {
        pub fn new(s: &str) -> Self;

        /// Longest common prefix of s[i..] and s[j..] - O(1)
        pub fn lce(&self, i: usize, j: usize) -> usize;
    }
}

pub mod string_misc {
    /// Lyndon factorization (Duval's algorithm)
    pub fn lyndon_factorization(s: &str) -> Vec<String>;

    /// Minimum cyclic rotation
    pub fn min_rotation(s: &str) -> String;

    /// Main-Lorentz: count palindromes - O(n log n)
    pub fn count_palindromes_fast(s: &str) -> usize;

    /// De Bruijn sequence
    pub fn de_bruijn(k: usize, n: usize) -> String;

    /// Booth's algorithm for lexicographically minimal rotation
    pub fn booth_min_rotation(s: &str) -> usize;  // Returns starting index
}
```

### Python Implementation

```python
from typing import List, Tuple, Optional

class SuffixAutomaton:
    def __init__(self): ...
    def build(self, s: str) -> None: ...
    def contains(self, pattern: str) -> bool: ...
    def count_occurrences(self, pattern: str) -> int: ...
    def count_distinct_substrings(self) -> int: ...

class PalindromicTree:
    def __init__(self): ...
    def build(self, s: str) -> None: ...
    def count_palindromes(self) -> int: ...
    def longest_palindrome(self) -> str: ...

def bwt(s: str) -> Tuple[str, int]: ...
def inverse_bwt(bwt: str, idx: int) -> str: ...

class FMIndex:
    def __init__(self, s: str): ...
    def count(self, pattern: str) -> int: ...
    def locate(self, pattern: str) -> List[int]: ...

def lyndon_factorization(s: str) -> List[str]: ...
def min_rotation(s: str) -> str: ...
```

## Test Cases

```rust
#[test]
fn test_suffix_automaton_contains() {
    let sa = SuffixAutomaton::build("abcabc");

    assert!(sa.contains("abc"));
    assert!(sa.contains("bca"));
    assert!(sa.contains(""));
    assert!(!sa.contains("abcd"));
}

#[test]
fn test_suffix_automaton_count() {
    let sa = SuffixAutomaton::build("abcabc");

    assert_eq!(sa.count_occurrences("abc"), 2);
    assert_eq!(sa.count_occurrences("bc"), 2);
    assert_eq!(sa.count_occurrences("abcabc"), 1);
}

#[test]
fn test_distinct_substrings() {
    let sa = SuffixAutomaton::build("aaa");
    assert_eq!(sa.count_distinct_substrings(), 3);  // a, aa, aaa

    let sa2 = SuffixAutomaton::build("abc");
    assert_eq!(sa2.count_distinct_substrings(), 6);  // a,b,c,ab,bc,abc
}

#[test]
fn test_longest_common_substring() {
    let lcs = SuffixAutomaton::longest_common_substring("abcdef", "xbcdey");
    assert_eq!(lcs, "bcde");
}

#[test]
fn test_palindromic_tree() {
    let pt = PalindromicTree::build("abaaba");

    // Palindromes: a, b, aba, aa, baab, abaaba
    assert_eq!(pt.count_palindromes(), 6);
    assert_eq!(pt.longest_palindrome(), "abaaba");
}

#[test]
fn test_bwt() {
    let (bwt, idx) = bwt("banana");
    assert_eq!(bwt, "annb$aa");  // Or similar based on implementation

    let recovered = inverse_bwt(&bwt, idx);
    assert_eq!(recovered, "banana");
}

#[test]
fn test_fm_index() {
    let fm = FMIndex::new("mississippi");

    assert_eq!(fm.count("issi"), 2);
    assert_eq!(fm.count("ss"), 2);
    assert_eq!(fm.count("xyz"), 0);

    let positions = fm.locate("issi");
    assert_eq!(positions.len(), 2);
}

#[test]
fn test_lcp_array() {
    let s = "banana";
    let sa = suffix_array(s);  // [5, 3, 1, 0, 4, 2]
    let lcp = build_lcp(s, &sa);

    // LCP between adjacent suffixes in sorted order
    // a, ana, anana, banana, na, nana
    // LCP: 1, 3, 0, 0, 2
    assert_eq!(lcp, vec![0, 1, 3, 0, 0, 2]);
}

#[test]
fn test_lce_queries() {
    let lce = LCEQuery::new("aabaaab");

    // LCE(0, 3) = length of common prefix of "aabaaab" and "aaab"
    assert_eq!(lce.lce(0, 3), 2);  // "aa"

    // LCE(1, 4) = common prefix of "abaaab" and "aab"
    assert_eq!(lce.lce(1, 4), 1);  // "a"
}

#[test]
fn test_lyndon() {
    let factors = lyndon_factorization("abbaabbaabba");
    // Should decompose into non-increasing Lyndon words
    for i in 1..factors.len() {
        assert!(factors[i-1] >= factors[i]);
    }
}

#[test]
fn test_min_rotation() {
    assert_eq!(min_rotation("bca"), "abc");
    assert_eq!(min_rotation("cba"), "acb");
    assert_eq!(min_rotation("aaaa"), "aaaa");
}

#[test]
fn test_enhanced_sa() {
    let esa = EnhancedSA::new("mississippi");

    assert_eq!(esa.count("mississippi", "issi"), 2);
    assert_eq!(esa.distinct_substrings("mississippi"), 53);
    assert_eq!(esa.longest_repeated("mississippi"), "issi");
}

#[test]
fn test_kth_substring() {
    let sa = SuffixAutomaton::build("abc");
    // Substrings in lex order: a, ab, abc, b, bc, c
    assert_eq!(sa.kth_substring(1), Some("a".to_string()));
    assert_eq!(sa.kth_substring(3), Some("abc".to_string()));
    assert_eq!(sa.kth_substring(6), Some("c".to_string()));
    assert_eq!(sa.kth_substring(7), None);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Suffix automaton | 25 |
| Palindromic tree | 20 |
| BWT / FM-Index | 20 |
| LCP array | 10 |
| Enhanced suffix array | 15 |
| Lyndon factorization | 5 |
| Edge cases | 5 |
| **Total** | **100** |
