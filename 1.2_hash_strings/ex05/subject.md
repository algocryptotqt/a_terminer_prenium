# Exercise 05: Trie & Applications

## Concepts Covered
- **1.2.18.i** Complexity analysis
- **1.2.18.j** Space optimization
- **1.2.18.k** Compressed Trie (Radix Tree)
- **1.2.18.l** Applications

## Objective

Implement a versatile Trie data structure with various optimizations and applications.

## Requirements

### Rust Implementation

```rust
pub mod trie {
    use std::collections::HashMap;

    /// Basic Trie with HashMap children
    pub struct Trie {
        root: TrieNode,
    }

    struct TrieNode {
        children: HashMap<char, TrieNode>,
        is_end: bool,
        count: usize,  // Number of words with this prefix
    }

    impl Trie {
        pub fn new() -> Self;

        /// Insert a word
        pub fn insert(&mut self, word: &str);

        /// Check if word exists
        pub fn search(&self, word: &str) -> bool;

        /// Check if any word starts with prefix
        pub fn starts_with(&self, prefix: &str) -> bool;

        /// Count words with given prefix
        pub fn count_prefix(&self, prefix: &str) -> usize;

        /// Get all words with given prefix (autocomplete)
        pub fn autocomplete(&self, prefix: &str) -> Vec<String>;

        /// Delete a word
        pub fn delete(&mut self, word: &str) -> bool;

        /// Get all words in trie
        pub fn all_words(&self) -> Vec<String>;

        /// Longest common prefix of all words
        pub fn longest_common_prefix(&self) -> String;
    }

    /// Array-based Trie (faster, fixed alphabet)
    pub struct ArrayTrie {
        nodes: Vec<[i32; 26]>,  // -1 means no child
        is_end: Vec<bool>,
        count: Vec<usize>,
    }

    impl ArrayTrie {
        pub fn new() -> Self;
        pub fn insert(&mut self, word: &str);
        pub fn search(&self, word: &str) -> bool;
        pub fn starts_with(&self, prefix: &str) -> bool;
    }

    /// Compressed Trie (Radix Tree)
    pub struct RadixTree {
        root: RadixNode,
    }

    struct RadixNode {
        children: HashMap<char, (String, RadixNode)>,  // (edge_label, child)
        is_end: bool,
    }

    impl RadixTree {
        pub fn new() -> Self;
        pub fn insert(&mut self, word: &str);
        pub fn search(&self, word: &str) -> bool;
        pub fn delete(&mut self, word: &str) -> bool;

        /// Memory usage compared to regular trie
        pub fn node_count(&self) -> usize;
    }

    // Applications

    /// Word search with wildcards (. matches any char)
    pub struct WildcardTrie {
        trie: Trie,
    }

    impl WildcardTrie {
        pub fn new() -> Self;
        pub fn add_word(&mut self, word: &str);
        pub fn search(&self, pattern: &str) -> bool;
    }

    /// Maximum XOR of two numbers using Trie
    pub fn max_xor_pair(nums: &[u32]) -> u32;

    /// Count distinct substrings using Trie
    pub fn count_distinct_substrings(s: &str) -> usize;

    /// Longest word that can be built from other words
    pub fn longest_buildable_word(words: &[&str]) -> String;

    /// Word break: can string be segmented into dictionary words?
    pub fn word_break(s: &str, dictionary: &[&str]) -> bool;

    /// Find all words in grid (word search II)
    pub fn find_words_in_grid(board: &[Vec<char>], words: &[&str]) -> Vec<String>;
}
```

### Python Implementation

```python
class Trie:
    def __init__(self) -> None: ...
    def insert(self, word: str) -> None: ...
    def search(self, word: str) -> bool: ...
    def starts_with(self, prefix: str) -> bool: ...
    def count_prefix(self, prefix: str) -> int: ...
    def autocomplete(self, prefix: str) -> list[str]: ...
    def delete(self, word: str) -> bool: ...

class ArrayTrie:
    def __init__(self) -> None: ...
    def insert(self, word: str) -> None: ...
    def search(self, word: str) -> bool: ...

class RadixTree:
    def __init__(self) -> None: ...
    def insert(self, word: str) -> None: ...
    def search(self, word: str) -> bool: ...

class WildcardTrie:
    def __init__(self) -> None: ...
    def add_word(self, word: str) -> None: ...
    def search(self, pattern: str) -> bool: ...

def max_xor_pair(nums: list[int]) -> int: ...
def count_distinct_substrings(s: str) -> int: ...
def word_break(s: str, dictionary: list[str]) -> bool: ...
def find_words_in_grid(board: list[list[str]], words: list[str]) -> list[str]: ...
```

## Test Cases

```rust
#[test]
fn test_basic_trie() {
    let mut trie = Trie::new();
    trie.insert("apple");
    trie.insert("app");
    trie.insert("application");

    assert!(trie.search("apple"));
    assert!(trie.search("app"));
    assert!(!trie.search("appl"));
    assert!(trie.starts_with("app"));
    assert!(!trie.starts_with("apo"));
}

#[test]
fn test_autocomplete() {
    let mut trie = Trie::new();
    for word in ["apple", "app", "application", "apply", "banana"] {
        trie.insert(word);
    }

    let suggestions = trie.autocomplete("app");
    assert_eq!(suggestions.len(), 4);
    assert!(suggestions.contains(&"apple".to_string()));
}

#[test]
fn test_delete() {
    let mut trie = Trie::new();
    trie.insert("apple");
    trie.insert("app");

    assert!(trie.delete("apple"));
    assert!(!trie.search("apple"));
    assert!(trie.search("app"));  // "app" still exists
}

#[test]
fn test_wildcard_search() {
    let mut wt = WildcardTrie::new();
    wt.add_word("bad");
    wt.add_word("dad");
    wt.add_word("mad");

    assert!(wt.search("pad") == false);
    assert!(wt.search("bad"));
    assert!(wt.search(".ad"));
    assert!(wt.search("b.."));
}

#[test]
fn test_max_xor() {
    assert_eq!(max_xor_pair(&[3, 10, 5, 25, 2, 8]), 28);  // 5 XOR 25
    assert_eq!(max_xor_pair(&[1, 2, 3, 4]), 7);  // 3 XOR 4
}

#[test]
fn test_word_break() {
    assert!(word_break("leetcode", &["leet", "code"]));
    assert!(word_break("applepenapple", &["apple", "pen"]));
    assert!(!word_break("catsandog", &["cats", "dog", "sand", "and", "cat"]));
}

#[test]
fn test_find_words_in_grid() {
    let board = vec![
        vec!['o', 'a', 'a', 'n'],
        vec!['e', 't', 'a', 'e'],
        vec!['i', 'h', 'k', 'r'],
        vec!['i', 'f', 'l', 'v'],
    ];
    let words = vec!["oath", "pea", "eat", "rain"];
    let found = find_words_in_grid(&board, &words);

    assert!(found.contains(&"oath".to_string()));
    assert!(found.contains(&"eat".to_string()));
}

#[test]
fn test_radix_tree() {
    let mut rt = RadixTree::new();
    rt.insert("romane");
    rt.insert("romanus");
    rt.insert("romulus");
    rt.insert("rubens");

    assert!(rt.search("romane"));
    assert!(rt.search("romanus"));
    assert!(!rt.search("roman"));

    // Radix tree should have fewer nodes than regular trie
    assert!(rt.node_count() < 15);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic Trie operations | 15 |
| Autocomplete | 10 |
| Delete with cleanup | 10 |
| Array-based Trie | 10 |
| Radix Tree | 15 |
| Wildcard search | 10 |
| Max XOR pair | 10 |
| Word break | 10 |
| Word search in grid | 10 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `trie.py`
