# Exercise 07: Mini-Project - Text Search Engine

## Concepts Covered
All Module 1.2 concepts integrated into a production-quality search engine.

## Objective

Build a complete text search engine that:
1. Indexes documents efficiently
2. Supports multiple search modes
3. Provides autocomplete
4. Handles fuzzy matching
5. Ranks results by relevance

## Requirements

### Library Structure

```
text_search_engine/
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── index/
│   │   ├── mod.rs
│   │   ├── inverted.rs     # Inverted index
│   │   ├── trie.rs         # Autocomplete
│   │   └── suffix.rs       # Suffix array index
│   ├── search/
│   │   ├── mod.rs
│   │   ├── exact.rs        # Exact matching
│   │   ├── fuzzy.rs        # Fuzzy/approximate
│   │   ├── wildcard.rs     # Wildcard patterns
│   │   └── phrase.rs       # Phrase search
│   ├── ranking/
│   │   ├── mod.rs
│   │   ├── tfidf.rs        # TF-IDF scoring
│   │   └── bm25.rs         # BM25 ranking
│   └── storage/
│       ├── mod.rs
│       └── persist.rs      # Index persistence
├── benches/
└── tests/
```

### Public API

```rust
pub mod search_engine {
    use std::path::Path;

    /// Document with ID and content
    #[derive(Clone, Debug)]
    pub struct Document {
        pub id: String,
        pub title: String,
        pub content: String,
        pub metadata: std::collections::HashMap<String, String>,
    }

    /// Search result with score
    #[derive(Debug)]
    pub struct SearchResult {
        pub document_id: String,
        pub score: f64,
        pub matches: Vec<Match>,
    }

    #[derive(Debug)]
    pub struct Match {
        pub field: String,
        pub position: usize,
        pub length: usize,
        pub snippet: String,
    }

    /// Search options
    #[derive(Default)]
    pub struct SearchOptions {
        pub fuzzy: bool,
        pub fuzzy_distance: usize,
        pub highlight: bool,
        pub limit: usize,
        pub offset: usize,
    }

    /// Main search engine
    pub struct SearchEngine {
        // Implementation
    }

    impl SearchEngine {
        /// Create new empty engine
        pub fn new() -> Self;

        /// Load from persisted index
        pub fn load(path: &Path) -> std::io::Result<Self>;

        /// Save index to disk
        pub fn save(&self, path: &Path) -> std::io::Result<()>;

        /// Add a document
        pub fn add_document(&mut self, doc: Document);

        /// Add multiple documents
        pub fn add_documents(&mut self, docs: Vec<Document>);

        /// Remove a document
        pub fn remove_document(&mut self, id: &str) -> bool;

        /// Exact search
        pub fn search(&self, query: &str, options: &SearchOptions) -> Vec<SearchResult>;

        /// Fuzzy search (approximate matching)
        pub fn search_fuzzy(&self, query: &str, max_distance: usize) -> Vec<SearchResult>;

        /// Wildcard search (* and ?)
        pub fn search_wildcard(&self, pattern: &str) -> Vec<SearchResult>;

        /// Phrase search (exact phrase)
        pub fn search_phrase(&self, phrase: &str) -> Vec<SearchResult>;

        /// Boolean search (AND, OR, NOT)
        pub fn search_boolean(&self, query: &str) -> Vec<SearchResult>;

        /// Autocomplete suggestions
        pub fn autocomplete(&self, prefix: &str, limit: usize) -> Vec<String>;

        /// Did you mean? (spell correction)
        pub fn suggest_correction(&self, query: &str) -> Option<String>;

        /// Get document count
        pub fn document_count(&self) -> usize;

        /// Get index statistics
        pub fn stats(&self) -> IndexStats;
    }

    #[derive(Debug)]
    pub struct IndexStats {
        pub document_count: usize,
        pub term_count: usize,
        pub total_tokens: usize,
        pub index_size_bytes: usize,
    }

    /// Query parser for complex queries
    pub struct QueryParser {
        // Implementation
    }

    impl QueryParser {
        pub fn parse(query: &str) -> Result<Query, ParseError>;
    }

    pub enum Query {
        Term(String),
        Phrase(Vec<String>),
        Wildcard(String),
        Fuzzy(String, usize),
        And(Box<Query>, Box<Query>),
        Or(Box<Query>, Box<Query>),
        Not(Box<Query>),
    }
}
```

### Python Implementation

```python
from dataclasses import dataclass, field
from pathlib import Path

@dataclass
class Document:
    id: str
    title: str
    content: str
    metadata: dict[str, str] = field(default_factory=dict)

@dataclass
class SearchResult:
    document_id: str
    score: float
    matches: list["Match"]

@dataclass
class Match:
    field: str
    position: int
    length: int
    snippet: str

@dataclass
class SearchOptions:
    fuzzy: bool = False
    fuzzy_distance: int = 2
    highlight: bool = True
    limit: int = 10
    offset: int = 0

class SearchEngine:
    def __init__(self) -> None: ...

    @classmethod
    def load(cls, path: Path) -> "SearchEngine": ...
    def save(self, path: Path) -> None: ...

    def add_document(self, doc: Document) -> None: ...
    def add_documents(self, docs: list[Document]) -> None: ...
    def remove_document(self, id: str) -> bool: ...

    def search(self, query: str, options: SearchOptions | None = None) -> list[SearchResult]: ...
    def search_fuzzy(self, query: str, max_distance: int = 2) -> list[SearchResult]: ...
    def search_wildcard(self, pattern: str) -> list[SearchResult]: ...
    def search_phrase(self, phrase: str) -> list[SearchResult]: ...
    def search_boolean(self, query: str) -> list[SearchResult]: ...

    def autocomplete(self, prefix: str, limit: int = 10) -> list[str]: ...
    def suggest_correction(self, query: str) -> str | None: ...

    def document_count(self) -> int: ...
    def stats(self) -> "IndexStats": ...
```

## Implementation Requirements

### Indexing
1. **Inverted Index**: Map terms to document IDs with positions
2. **Trie**: For autocomplete functionality
3. **N-gram Index**: For fuzzy matching

### Search Algorithms
1. **Exact Match**: Using inverted index
2. **Fuzzy Match**: Edit distance with BK-tree or n-gram similarity
3. **Wildcard**: Pattern matching with tries
4. **Phrase**: Position-aware matching

### Ranking
1. **TF-IDF**: Term frequency-inverse document frequency
2. **BM25**: Okapi BM25 algorithm

### Persistence
- Serialize index to binary format
- Support incremental updates

## Test Cases

```rust
#[test]
fn test_basic_search() {
    let mut engine = SearchEngine::new();

    engine.add_document(Document {
        id: "1".into(),
        title: "Rust Programming".into(),
        content: "Rust is a systems programming language".into(),
        metadata: Default::default(),
    });

    engine.add_document(Document {
        id: "2".into(),
        title: "Python Guide".into(),
        content: "Python is great for scripting".into(),
        metadata: Default::default(),
    });

    let results = engine.search("rust", &Default::default());
    assert_eq!(results.len(), 1);
    assert_eq!(results[0].document_id, "1");
}

#[test]
fn test_fuzzy_search() {
    let mut engine = SearchEngine::new();
    engine.add_document(Document {
        id: "1".into(),
        title: "Programming".into(),
        content: "Learn programming today".into(),
        metadata: Default::default(),
    });

    // Typo: "programing" should find "programming"
    let results = engine.search_fuzzy("programing", 1);
    assert_eq!(results.len(), 1);
}

#[test]
fn test_wildcard() {
    let mut engine = SearchEngine::new();
    engine.add_document(Document {
        id: "1".into(),
        title: "Test".into(),
        content: "testing tester tested".into(),
        metadata: Default::default(),
    });

    let results = engine.search_wildcard("test*");
    assert_eq!(results.len(), 1);
}

#[test]
fn test_phrase_search() {
    let mut engine = SearchEngine::new();
    engine.add_document(Document {
        id: "1".into(),
        title: "Doc".into(),
        content: "the quick brown fox".into(),
        metadata: Default::default(),
    });

    let results = engine.search_phrase("quick brown");
    assert_eq!(results.len(), 1);

    let results = engine.search_phrase("brown quick");
    assert_eq!(results.len(), 0);
}

#[test]
fn test_autocomplete() {
    let mut engine = SearchEngine::new();
    engine.add_documents(vec![
        Document { id: "1".into(), title: "".into(), content: "programming".into(), metadata: Default::default() },
        Document { id: "2".into(), title: "".into(), content: "program".into(), metadata: Default::default() },
        Document { id: "3".into(), title: "".into(), content: "progress".into(), metadata: Default::default() },
    ]);

    let suggestions = engine.autocomplete("prog", 10);
    assert!(suggestions.contains(&"programming".to_string()));
    assert!(suggestions.contains(&"program".to_string()));
    assert!(suggestions.contains(&"progress".to_string()));
}

#[test]
fn test_ranking() {
    let mut engine = SearchEngine::new();
    engine.add_document(Document {
        id: "1".into(),
        title: "".into(),
        content: "rust rust rust".into(),  // More occurrences
        metadata: Default::default(),
    });
    engine.add_document(Document {
        id: "2".into(),
        title: "".into(),
        content: "rust programming".into(),
        metadata: Default::default(),
    });

    let results = engine.search("rust", &Default::default());
    assert!(results[0].score > results[1].score);
}

#[test]
fn test_persistence() {
    let mut engine = SearchEngine::new();
    engine.add_document(Document {
        id: "1".into(),
        title: "Test".into(),
        content: "content".into(),
        metadata: Default::default(),
    });

    let path = std::path::Path::new("/tmp/test_index");
    engine.save(path).unwrap();

    let loaded = SearchEngine::load(path).unwrap();
    assert_eq!(loaded.document_count(), 1);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Inverted index | 15 |
| Basic search | 10 |
| Fuzzy search | 15 |
| Wildcard search | 10 |
| Phrase search | 10 |
| Autocomplete | 10 |
| TF-IDF/BM25 ranking | 10 |
| Persistence | 10 |
| Tests and documentation | 10 |
| **Total** | **100** |

## Deliverables

1. Complete library source code
2. All tests passing
3. Benchmark results
4. README with usage examples
5. API documentation
