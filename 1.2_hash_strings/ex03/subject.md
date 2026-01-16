# Exercise 03: Aho-Corasick Algorithm

## Concepts Covered
- **1.2.16.g** Implementation
- **1.2.16.h** Applications
- **1.2.16.i** Comparison with other algorithms

## Objective

Implement the Aho-Corasick algorithm for efficient multi-pattern string matching.

## Requirements

### Rust Implementation

```rust
pub mod aho_corasick {
    use std::collections::{HashMap, VecDeque};

    /// Aho-Corasick automaton for multiple pattern matching
    pub struct AhoCorasick {
        goto: Vec<HashMap<u8, usize>>,
        fail: Vec<usize>,
        output: Vec<Vec<usize>>,  // Pattern indices that end at each state
        patterns: Vec<Vec<u8>>,
    }

    impl AhoCorasick {
        /// Build automaton from patterns
        pub fn new(patterns: &[&[u8]]) -> Self;

        /// Build from string patterns
        pub fn from_strings(patterns: &[&str]) -> Self;

        /// Find all occurrences of all patterns
        /// Returns (position, pattern_index) pairs
        pub fn find_all(&self, text: &[u8]) -> Vec<(usize, usize)>;

        /// Find first occurrence of any pattern
        pub fn find_first(&self, text: &[u8]) -> Option<(usize, usize)>;

        /// Check if any pattern exists in text
        pub fn contains_any(&self, text: &[u8]) -> bool;

        /// Count total occurrences of all patterns
        pub fn count_all(&self, text: &[u8]) -> usize;

        /// Count occurrences per pattern
        pub fn count_per_pattern(&self, text: &[u8]) -> Vec<usize>;

        /// Replace all pattern occurrences with replacement
        pub fn replace_all(&self, text: &[u8], replacement: &[u8]) -> Vec<u8>;

        /// Stream processing: process character by character
        pub fn create_stream(&self) -> AhoCorasickStream;
    }

    /// Streaming interface for Aho-Corasick
    pub struct AhoCorasickStream<'a> {
        ac: &'a AhoCorasick,
        state: usize,
        position: usize,
    }

    impl<'a> AhoCorasickStream<'a> {
        /// Process next character, returns matches at current position
        pub fn next(&mut self, c: u8) -> Vec<usize>;

        /// Reset to initial state
        pub fn reset(&mut self);
    }

    // Applications

    /// DNA sequence pattern matching
    pub fn find_dna_patterns(sequence: &str, patterns: &[&str]) -> Vec<(usize, String)>;

    /// Keyword filtering (content moderation)
    pub fn filter_keywords(text: &str, keywords: &[&str], replacement: &str) -> String;

    /// Find overlapping patterns
    pub fn find_overlapping(text: &[u8], patterns: &[&[u8]]) -> Vec<(usize, usize, usize)>;
}
```

### Python Implementation

```python
class AhoCorasick:
    def __init__(self, patterns: list[bytes | str]) -> None: ...
    def find_all(self, text: bytes | str) -> list[tuple[int, int]]: ...
    def find_first(self, text: bytes | str) -> tuple[int, int] | None: ...
    def contains_any(self, text: bytes | str) -> bool: ...
    def count_all(self, text: bytes | str) -> int: ...
    def count_per_pattern(self, text: bytes | str) -> list[int]: ...
    def replace_all(self, text: str, replacement: str) -> str: ...

class AhoCorasickStream:
    def __init__(self, ac: AhoCorasick) -> None: ...
    def next(self, c: int) -> list[int]: ...
    def reset(self) -> None: ...
```

## Algorithm Details

### Construction
1. **Build Trie**: Insert all patterns into a trie
2. **Build Failure Links**: BFS from root, compute fail[state] = longest proper suffix that is also a prefix of some pattern
3. **Build Output Links**: Collect all patterns that end at each state (including via failure links)

### Matching
```rust
fn find_all(&self, text: &[u8]) -> Vec<(usize, usize)> {
    let mut state = 0;
    let mut matches = Vec::new();

    for (i, &c) in text.iter().enumerate() {
        // Follow failure links until we find a transition or reach root
        while state != 0 && !self.goto[state].contains_key(&c) {
            state = self.fail[state];
        }
        state = *self.goto[state].get(&c).unwrap_or(&0);

        // Collect all patterns that match at this position
        for &pattern_idx in &self.output[state] {
            let pattern_len = self.patterns[pattern_idx].len();
            matches.push((i + 1 - pattern_len, pattern_idx));
        }
    }
    matches
}
```

## Test Cases

```rust
#[test]
fn test_basic_matching() {
    let patterns = vec![b"he".as_slice(), b"she", b"his", b"hers"];
    let ac = AhoCorasick::new(&patterns);

    let text = b"ushers";
    let matches = ac.find_all(text);

    // Should find: "she" at 1, "he" at 2, "hers" at 2
    assert!(matches.contains(&(1, 1)));  // "she"
    assert!(matches.contains(&(2, 0)));  // "he"
    assert!(matches.contains(&(2, 3)));  // "hers"
}

#[test]
fn test_overlapping() {
    let patterns = vec![b"a".as_slice(), b"aa", b"aaa"];
    let ac = AhoCorasick::new(&patterns);

    let matches = ac.find_all(b"aaaa");
    // "a" at 0,1,2,3; "aa" at 0,1,2; "aaa" at 0,1
    assert_eq!(ac.count_all(b"aaaa"), 10);
}

#[test]
fn test_no_match() {
    let patterns = vec![b"xyz".as_slice(), b"abc"];
    let ac = AhoCorasick::new(&patterns);

    assert!(!ac.contains_any(b"hello world"));
    assert_eq!(ac.find_all(b"hello world").len(), 0);
}

#[test]
fn test_count_per_pattern() {
    let patterns = vec![b"ab".as_slice(), b"bc", b"abc"];
    let ac = AhoCorasick::new(&patterns);

    let counts = ac.count_per_pattern(b"abcabc");
    assert_eq!(counts[0], 2);  // "ab" appears twice
    assert_eq!(counts[1], 2);  // "bc" appears twice
    assert_eq!(counts[2], 2);  // "abc" appears twice
}

#[test]
fn test_replace() {
    let patterns = vec![b"bad".as_slice(), b"ugly"];
    let ac = AhoCorasick::new(&patterns);

    let result = ac.replace_all(b"this is bad and ugly", b"***");
    assert_eq!(result, b"this is *** and ***".to_vec());
}

#[test]
fn test_streaming() {
    let patterns = vec![b"ab".as_slice(), b"bc"];
    let ac = AhoCorasick::new(&patterns);
    let mut stream = ac.create_stream();

    assert!(stream.next(b'a').is_empty());
    assert_eq!(stream.next(b'b'), vec![0]);  // "ab" matched
    assert_eq!(stream.next(b'c'), vec![1]);  // "bc" matched
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Trie construction | 15 |
| Failure link computation | 20 |
| Output link computation | 15 |
| find_all implementation | 15 |
| Streaming interface | 15 |
| Replace functionality | 10 |
| Edge cases | 10 |
| **Total** | **100** |

## Complexity

- Construction: O(Σ|patterns| + alphabet_size)
- Matching: O(|text| + number_of_matches)

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `aho_corasick.py`
