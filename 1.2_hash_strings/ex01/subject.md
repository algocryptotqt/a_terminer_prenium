# Exercise 01: KMP & Z-Algorithm

## Concepts Covered
- **1.2.12.h-j** KMP proof, examples, all occurrences
- **1.2.13.g-i** Z-algorithm complexity, applications, KMP comparison

## Objective

Implement KMP and Z-algorithm for efficient pattern matching. Both achieve O(n+m) time complexity.

## Requirements

### Rust Implementation

```rust
pub mod string_matching {
    /// Compute KMP failure function (prefix function)
    /// failure[i] = length of longest proper prefix of pattern[0..i+1]
    ///              that is also a suffix
    pub fn compute_failure(pattern: &[u8]) -> Vec<usize>;

    /// KMP search - find first occurrence of pattern in text
    pub fn kmp_search(text: &[u8], pattern: &[u8]) -> Option<usize>;

    /// KMP search - find all occurrences
    pub fn kmp_search_all(text: &[u8], pattern: &[u8]) -> Vec<usize>;

    /// KMP with custom comparator
    pub fn kmp_search_by<F>(text: &[u8], pattern: &[u8], eq: F) -> Vec<usize>
    where
        F: Fn(u8, u8) -> bool;

    /// Compute Z-array
    /// z[i] = length of longest substring starting at i that matches prefix
    pub fn compute_z_array(s: &[u8]) -> Vec<usize>;

    /// Z-algorithm search
    pub fn z_search(text: &[u8], pattern: &[u8]) -> Vec<usize>;

    // Applications

    /// Count distinct substrings using Z-array
    pub fn count_distinct_substrings(s: &str) -> usize;

    /// Find the shortest period of a string
    /// Period p: s[i] = s[i % p] for all i
    pub fn shortest_period(s: &[u8]) -> usize;

    /// Check if a string is a rotation of another
    pub fn is_rotation(s1: &str, s2: &str) -> bool;

    /// Find the lexicographically smallest rotation
    pub fn min_rotation(s: &str) -> String;

    /// Compute all borders (prefixes that are also suffixes)
    pub fn all_borders(s: &[u8]) -> Vec<usize>;

    /// Pattern matching with wildcards (? matches any single char)
    pub fn wildcard_match(text: &[u8], pattern: &[u8]) -> Vec<usize>;
}
```

### Python Implementation

```python
def compute_failure(pattern: bytes) -> list[int]: ...
def kmp_search(text: bytes, pattern: bytes) -> int | None: ...
def kmp_search_all(text: bytes, pattern: bytes) -> list[int]: ...
def compute_z_array(s: bytes) -> list[int]: ...
def z_search(text: bytes, pattern: bytes) -> list[int]: ...
def count_distinct_substrings(s: str) -> int: ...
def shortest_period(s: bytes) -> int: ...
def is_rotation(s1: str, s2: str) -> bool: ...
def min_rotation(s: str) -> str: ...
def all_borders(s: bytes) -> list[int]: ...
def wildcard_match(text: bytes, pattern: bytes) -> list[int]: ...
```

## Algorithm Details

### KMP Failure Function
```
For pattern P:
failure[0] = 0
For i from 1 to m-1:
    j = failure[i-1]
    while j > 0 and P[i] != P[j]:
        j = failure[j-1]
    if P[i] == P[j]:
        j += 1
    failure[i] = j
```

### Z-Algorithm
```
For string S:
z[0] = n (or undefined)
l = r = 0  # [l, r] is rightmost z-box
For i from 1 to n-1:
    if i > r:
        l = r = i
        while r < n and S[r-l] == S[r]:
            r += 1
        z[i] = r - l
        r -= 1
    else:
        k = i - l
        if z[k] < r - i + 1:
            z[i] = z[k]
        else:
            l = i
            while r < n and S[r-l] == S[r]:
                r += 1
            z[i] = r - l
            r -= 1
```

## Test Cases

```rust
#[test]
fn test_failure_function() {
    assert_eq!(compute_failure(b"AAAA"), vec![0, 1, 2, 3]);
    assert_eq!(compute_failure(b"ABAB"), vec![0, 0, 1, 2]);
    assert_eq!(compute_failure(b"AABAACAABAA"), vec![0, 1, 0, 1, 2, 0, 1, 2, 3, 4, 5]);
}

#[test]
fn test_kmp_search() {
    let text = b"AABAACAADAABAAABAA";
    let pattern = b"AABA";

    assert_eq!(kmp_search(text, pattern), Some(0));
    assert_eq!(kmp_search_all(text, pattern), vec![0, 9, 13]);
}

#[test]
fn test_z_array() {
    assert_eq!(compute_z_array(b"aabxaab"), vec![7, 1, 0, 0, 3, 1, 0]);
    assert_eq!(compute_z_array(b"aaaa"), vec![4, 3, 2, 1]);
}

#[test]
fn test_z_search() {
    let text = b"AABAACAADAABAAABAA";
    let pattern = b"AABA";

    assert_eq!(z_search(text, pattern), vec![0, 9, 13]);
}

#[test]
fn test_shortest_period() {
    assert_eq!(shortest_period(b"abcabc"), 3);
    assert_eq!(shortest_period(b"aaaa"), 1);
    assert_eq!(shortest_period(b"abcd"), 4);
    assert_eq!(shortest_period(b"abab"), 2);
}

#[test]
fn test_is_rotation() {
    assert!(is_rotation("waterbottle", "erbottlewat"));
    assert!(is_rotation("abcd", "cdab"));
    assert!(!is_rotation("abc", "acb"));
}

#[test]
fn test_min_rotation() {
    assert_eq!(min_rotation("bcab"), "abbc");
    assert_eq!(min_rotation("cba"), "acb");
}

#[test]
fn test_borders() {
    let borders = all_borders(b"abacaba");
    assert_eq!(borders, vec![1, 3, 7]);  // "a", "aba", "abacaba"
}

#[test]
fn test_wildcard() {
    // '?' matches any single character
    assert_eq!(wildcard_match(b"abcabc", b"a?c"), vec![0, 3]);
    assert_eq!(wildcard_match(b"aaa", b"?"), vec![0, 1, 2]);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| KMP failure function | 15 |
| KMP search (single + all) | 15 |
| Z-array computation | 15 |
| Z-algorithm search | 10 |
| Shortest period | 10 |
| Rotation checks | 10 |
| All borders | 10 |
| Wildcard matching | 15 |
| **Total** | **100** |

## Complexity Requirements

| Algorithm | Time | Space |
|-----------|------|-------|
| Failure function | O(m) | O(m) |
| KMP search | O(n+m) | O(m) |
| Z-array | O(n) | O(n) |
| Z-search | O(n+m) | O(n+m) |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `string_matching.py`
