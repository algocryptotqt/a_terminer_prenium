# Exercise 02: Rabin-Karp & Boyer-Moore

## Concepts Covered
- **1.2.14.g-i** Rabin-Karp worst case, multiple patterns, 2D matching
- **1.2.15.g-k** Boyer-Moore algorithm, complexities, Galil optimization

## Objective

Implement Rabin-Karp (hashing-based) and Boyer-Moore (rule-based) pattern matching algorithms.

## Requirements

### Rust Implementation

```rust
pub mod pattern_matching {
    /// Rabin-Karp with rolling hash
    pub struct RabinKarp {
        base: u64,
        modulus: u64,
    }

    impl RabinKarp {
        pub fn new() -> Self;
        pub fn with_params(base: u64, modulus: u64) -> Self;

        /// Compute hash of a string
        pub fn hash(&self, s: &[u8]) -> u64;

        /// Find all occurrences of pattern in text
        pub fn search(&self, text: &[u8], pattern: &[u8]) -> Vec<usize>;

        /// Find first occurrence
        pub fn search_first(&self, text: &[u8], pattern: &[u8]) -> Option<usize>;

        /// Search for multiple patterns simultaneously
        pub fn search_multiple(&self, text: &[u8], patterns: &[&[u8]]) -> Vec<(usize, usize)>;
    }

    /// 2D pattern matching using Rabin-Karp
    pub fn search_2d(
        matrix: &[Vec<u8>],
        pattern: &[Vec<u8>],
    ) -> Vec<(usize, usize)>;

    /// Boyer-Moore algorithm
    pub struct BoyerMoore {
        pattern: Vec<u8>,
        bad_char: [usize; 256],      // Bad character table
        good_suffix: Vec<usize>,      // Good suffix table
    }

    impl BoyerMoore {
        /// Preprocess pattern
        pub fn new(pattern: &[u8]) -> Self;

        /// Find all occurrences
        pub fn search(&self, text: &[u8]) -> Vec<usize>;

        /// Find first occurrence
        pub fn search_first(&self, text: &[u8]) -> Option<usize>;
    }

    /// Simplified Boyer-Moore-Horspool (only bad character rule)
    pub struct BoyerMooreHorspool {
        pattern: Vec<u8>,
        skip: [usize; 256],
    }

    impl BoyerMooreHorspool {
        pub fn new(pattern: &[u8]) -> Self;
        pub fn search(&self, text: &[u8]) -> Vec<usize>;
    }

    /// Boyer-Moore-Galil (with Galil rule optimization)
    pub struct BoyerMooreGalil {
        bm: BoyerMoore,
        period: usize,
    }

    impl BoyerMooreGalil {
        pub fn new(pattern: &[u8]) -> Self;
        pub fn search(&self, text: &[u8]) -> Vec<usize>;
    }

    // Utility functions

    /// Build bad character table
    pub fn build_bad_char_table(pattern: &[u8]) -> [usize; 256];

    /// Build good suffix table
    pub fn build_good_suffix_table(pattern: &[u8]) -> Vec<usize>;

    /// Compute period of pattern
    pub fn compute_period(pattern: &[u8]) -> usize;
}
```

### Python Implementation

```python
class RabinKarp:
    def __init__(self, base: int = 256, modulus: int = 10**9 + 7) -> None: ...
    def hash(self, s: bytes) -> int: ...
    def search(self, text: bytes, pattern: bytes) -> list[int]: ...
    def search_first(self, text: bytes, pattern: bytes) -> int | None: ...
    def search_multiple(self, text: bytes, patterns: list[bytes]) -> list[tuple[int, int]]: ...

def search_2d(matrix: list[list[int]], pattern: list[list[int]]) -> list[tuple[int, int]]: ...

class BoyerMoore:
    def __init__(self, pattern: bytes) -> None: ...
    def search(self, text: bytes) -> list[int]: ...
    def search_first(self, text: bytes) -> int | None: ...

class BoyerMooreHorspool:
    def __init__(self, pattern: bytes) -> None: ...
    def search(self, text: bytes) -> list[int]: ...

class BoyerMooreGalil:
    def __init__(self, pattern: bytes) -> None: ...
    def search(self, text: bytes) -> list[int]: ...
```

## Algorithm Details

### Rabin-Karp Rolling Hash
```
hash(s[0..m]) = s[0]*base^(m-1) + s[1]*base^(m-2) + ... + s[m-1]

Rolling update:
hash(s[i+1..i+m+1]) = (hash(s[i..i+m]) - s[i]*base^(m-1)) * base + s[i+m]
```

### Boyer-Moore Bad Character Rule
When mismatch at position j:
- Find rightmost occurrence of text[i+j] in pattern[0..j-1]
- Shift pattern to align that occurrence with text[i+j]

### Boyer-Moore Good Suffix Rule
When mismatch at position j after matching suffix:
- Find rightmost occurrence of the matched suffix in pattern
- Shift pattern to align that occurrence

### Galil Rule
After a complete match at position i:
- For periodic patterns, next match can only occur at i + period
- Skip comparisons in the periodic part

## Test Cases

```rust
#[test]
fn test_rabin_karp() {
    let rk = RabinKarp::new();
    let text = b"AABAACAADAABAAABAA";
    let pattern = b"AABA";

    assert_eq!(rk.search(text, pattern), vec![0, 9, 13]);
}

#[test]
fn test_rabin_karp_multiple() {
    let rk = RabinKarp::new();
    let text = b"AABABCABAB";
    let patterns: Vec<&[u8]> = vec![b"AB", b"ABA", b"BC"];

    let results = rk.search_multiple(text, &patterns);
    // Returns (position, pattern_index)
    assert!(results.contains(&(0, 1)));  // ABA at 0
    assert!(results.contains(&(1, 0)));  // AB at 1
}

#[test]
fn test_2d_search() {
    let matrix = vec![
        vec![1, 2, 3, 4],
        vec![5, 6, 7, 8],
        vec![1, 2, 3, 4],
        vec![5, 6, 7, 8],
    ];
    let pattern = vec![
        vec![6, 7],
        vec![2, 3],
    ];

    assert_eq!(search_2d(&matrix, &pattern), vec![(1, 1)]);
}

#[test]
fn test_boyer_moore() {
    let bm = BoyerMoore::new(b"EXAMPLE");
    let text = b"HERE IS A SIMPLE EXAMPLE";

    assert_eq!(bm.search(text), vec![17]);
}

#[test]
fn test_boyer_moore_multiple() {
    let bm = BoyerMoore::new(b"AABA");
    let text = b"AABAACAADAABAAABAA";

    assert_eq!(bm.search(text), vec![0, 9, 13]);
}

#[test]
fn test_horspool() {
    let bmh = BoyerMooreHorspool::new(b"ABC");
    let text = b"DABCABCABC";

    assert_eq!(bmh.search(text), vec![1, 4, 7]);
}

#[test]
fn test_galil_optimization() {
    // With periodic pattern, Galil is more efficient
    let bmg = BoyerMooreGalil::new(b"ABAB");
    let text = b"ABABABABAB";

    assert_eq!(bmg.search(text), vec![0, 2, 4, 6]);
}

#[test]
fn test_edge_cases() {
    let bm = BoyerMoore::new(b"A");

    assert_eq!(bm.search(b"AAA"), vec![0, 1, 2]);
    assert_eq!(bm.search(b""), vec![]);

    let bm = BoyerMoore::new(b"ABC");
    assert_eq!(bm.search(b"AB"), vec![]);  // Pattern longer than text
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Rabin-Karp single pattern | 15 |
| Rabin-Karp multiple patterns | 10 |
| 2D pattern matching | 15 |
| Boyer-Moore preprocessing | 15 |
| Boyer-Moore search | 15 |
| Boyer-Moore-Horspool | 10 |
| Boyer-Moore-Galil | 15 |
| Edge cases | 5 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `pattern_matching.py`
