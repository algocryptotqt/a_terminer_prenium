# Exercise 04: Manacher's Algorithm

## Concepts Covered
- **1.2.17.f** Mirror property
- **1.2.17.g** Algorithm implementation
- **1.2.17.h** Longest palindrome substring
- **1.2.17.i** Applications

## Objective

Implement Manacher's algorithm for finding all palindromic substrings in O(n) time.

## Requirements

### Rust Implementation

```rust
pub mod manacher {
    /// Manacher's algorithm result
    pub struct PalindromeInfo {
        /// p[i] = radius of palindrome centered at i (in transformed string)
        pub radii: Vec<usize>,
        /// Original string length
        pub original_len: usize,
    }

    impl PalindromeInfo {
        /// Get longest palindrome substring
        pub fn longest_palindrome(&self, original: &str) -> &str;

        /// Get all maximal palindromes (not contained in another)
        pub fn maximal_palindromes(&self) -> Vec<(usize, usize)>;

        /// Check if substring [l, r] is palindrome in O(1)
        pub fn is_palindrome(&self, l: usize, r: usize) -> bool;

        /// Count palindromic substrings
        pub fn count_palindromes(&self) -> usize;
    }

    /// Run Manacher's algorithm
    pub fn manacher(s: &str) -> PalindromeInfo;

    /// Find longest palindromic substring directly
    pub fn longest_palindrome(s: &str) -> String;

    /// Find all palindromic substrings
    pub fn all_palindromes(s: &str) -> Vec<String>;

    /// Count distinct palindromic substrings
    pub fn count_distinct_palindromes(s: &str) -> usize;

    /// Find longest palindrome starting at each position
    pub fn longest_from_each(s: &str) -> Vec<usize>;

    /// Find palindrome pairs: indices (i, j) where s[i..j] is palindrome
    pub fn palindrome_pairs(words: &[&str]) -> Vec<(usize, usize)>;

    /// Minimum cuts to partition string into palindromes
    pub fn min_palindrome_cuts(s: &str) -> usize;

    /// Check if string can be rearranged into palindrome
    pub fn can_form_palindrome(s: &str) -> bool;

    /// Shortest palindrome by adding characters at front
    pub fn shortest_palindrome_prefix(s: &str) -> String;
}
```

### Python Implementation

```python
from dataclasses import dataclass

@dataclass
class PalindromeInfo:
    radii: list[int]
    original_len: int

    def longest_palindrome(self, original: str) -> str: ...
    def is_palindrome(self, l: int, r: int) -> bool: ...
    def count_palindromes(self) -> int: ...

def manacher(s: str) -> PalindromeInfo: ...
def longest_palindrome(s: str) -> str: ...
def all_palindromes(s: str) -> list[str]: ...
def count_distinct_palindromes(s: str) -> int: ...
def min_palindrome_cuts(s: str) -> int: ...
def shortest_palindrome_prefix(s: str) -> str: ...
```

## Algorithm Details

### String Transformation
Transform "abc" → "#a#b#c#" to handle even/odd length palindromes uniformly.

### Manacher's Algorithm
```rust
fn manacher(s: &str) -> Vec<usize> {
    // Transform string
    let t: Vec<char> = format!("#{}#", s.chars().collect::<Vec<_>>().join("#"))
        .chars().collect();
    let n = t.len();
    let mut p = vec![0; n];
    let mut c = 0;  // Center of rightmost palindrome
    let mut r = 0;  // Right boundary of rightmost palindrome

    for i in 0..n {
        // Mirror position
        let mirror = 2 * c - i;

        if i < r {
            p[i] = (r - i).min(p[mirror]);
        }

        // Expand around center i
        while i + p[i] + 1 < n && i >= p[i] + 1 && t[i + p[i] + 1] == t[i - p[i] - 1] {
            p[i] += 1;
        }

        // Update center if we expanded past r
        if i + p[i] > r {
            c = i;
            r = i + p[i];
        }
    }
    p
}
```

### Key Insight
When `i < r`, we can use the mirror property: the palindrome at `i` is at least as long as the palindrome at `mirror = 2*c - i`, bounded by the remaining distance to `r`.

## Test Cases

```rust
#[test]
fn test_longest_palindrome() {
    assert_eq!(longest_palindrome("babad"), "bab");  // or "aba"
    assert_eq!(longest_palindrome("cbbd"), "bb");
    assert_eq!(longest_palindrome("a"), "a");
    assert_eq!(longest_palindrome("ac"), "a");  // or "c"
}

#[test]
fn test_manacher_radii() {
    let info = manacher("abba");
    // Transformed: #a#b#b#a#
    // Radii should show palindrome of length 4 centered at middle
    assert!(info.is_palindrome(0, 3));  // "abba" is palindrome
}

#[test]
fn test_count_palindromes() {
    let info = manacher("aaa");
    // Palindromes: "a"(3), "aa"(2), "aaa"(1) = 6 total
    assert_eq!(info.count_palindromes(), 6);

    let info = manacher("abc");
    // Palindromes: "a", "b", "c" = 3
    assert_eq!(info.count_palindromes(), 3);
}

#[test]
fn test_distinct_palindromes() {
    assert_eq!(count_distinct_palindromes("aaa"), 3);  // "a", "aa", "aaa"
    assert_eq!(count_distinct_palindromes("abab"), 4);  // "a", "b", "aba", "bab"
}

#[test]
fn test_min_cuts() {
    assert_eq!(min_palindrome_cuts("aab"), 1);  // "aa" + "b"
    assert_eq!(min_palindrome_cuts("a"), 0);
    assert_eq!(min_palindrome_cuts("ab"), 1);  // "a" + "b"
}

#[test]
fn test_shortest_palindrome_prefix() {
    assert_eq!(shortest_palindrome_prefix("aacecaaa"), "aaacecaaa");
    assert_eq!(shortest_palindrome_prefix("abcd"), "dcbabcd");
}

#[test]
fn test_can_form_palindrome() {
    assert!(can_form_palindrome("aab"));   // "aba"
    assert!(!can_form_palindrome("abc"));  // impossible
    assert!(can_form_palindrome("carerac"));  // "carerac" -> "racecar"
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Manacher's algorithm | 25 |
| Longest palindrome | 15 |
| Count palindromes | 15 |
| is_palindrome O(1) query | 10 |
| Distinct palindromes | 10 |
| Min palindrome cuts | 15 |
| Shortest palindrome prefix | 10 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `manacher.py`
