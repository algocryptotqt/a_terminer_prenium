# MODULE 1.9 - EXERCICES COMPLETS
## RÉVISION INTENSIVE & COMPETITIVE PROGRAMMING (CAPSTONE)

Ce fichier contient les exercices couvrant tous les concepts du Module 1.9.

---

# SECTION 1: COMPÉTITION VS PRODUCTION

## Exercice A1: `competition_vs_production`
**Couvre: 1.9.0.a-h (8 concepts)**

### Concepts
- [1.9.0.a] Philosophie — Code jetable vs code maintenable
- [1.9.0.b] `unwrap()` — Acceptable en compétition, interdit en production
- [1.9.0.c] Error handling — `?` operator, `thiserror`, `anyhow`
- [1.9.0.d] Input validation — Ne jamais faire confiance à l'input
- [1.9.0.e] Logging — `tracing` pour observabilité production
- [1.9.0.f] Panic safety — `catch_unwind`, panic handlers
- [1.9.0.g] Code review — Identifier les patterns compétition
- [1.9.0.h] Refactoring — Convertir code compétition en production

### Rust
```rust
use std::io::{self, BufRead, Write};
use std::panic::{self, AssertUnwindSafe};
use thiserror::Error;
use anyhow::{Context, Result as AnyhowResult};
use tracing::{info, warn, error, debug, instrument};

// ============================================================
// 1.9.0.a - Philosophie: Compétition vs Production
// ============================================================

// CODE COMPÉTITION - Rapide à écrire, fragile
mod competition {
    pub fn solve() {
        let mut input = String::new();
        std::io::stdin().read_line(&mut input).unwrap();
        let n: i32 = input.trim().parse().unwrap();
        println!("{}", n * 2);
    }
}

// CODE PRODUCTION - Robuste, maintenable
mod production {
    use super::*;

    pub fn solve() -> AnyhowResult<()> {
        let stdin = io::stdin();
        let input = stdin.lock().lines()
            .next()
            .context("No input provided")?
            .context("Failed to read line")?;

        let n: i32 = input.trim()
            .parse()
            .context("Invalid number format")?;

        println!("{}", n.checked_mul(2).context("Overflow")?);
        Ok(())
    }
}

// ============================================================
// 1.9.0.b - unwrap() - Compétition OK, Production NON
// ============================================================

// COMPÉTITION: unwrap() partout (on sait que l'input est valide)
fn competition_parse(s: &str) -> i32 {
    s.trim().parse().unwrap()  // OK en compétition
}

// PRODUCTION: Jamais de unwrap()
fn production_parse(s: &str) -> Result<i32, ParseIntError> {
    s.trim().parse()  // Retourne Result
}

// ============================================================
// 1.9.0.c - Error handling: ?, thiserror, anyhow
// ============================================================

// Custom error avec thiserror
#[derive(Error, Debug)]
pub enum JudgeError {
    #[error("Compilation failed: {0}")]
    CompilationError(String),

    #[error("Runtime error: {0}")]
    RuntimeError(String),

    #[error("Time limit exceeded: {limit_ms}ms")]
    TimeLimitExceeded { limit_ms: u64 },

    #[error("Memory limit exceeded: {limit_mb}MB")]
    MemoryLimitExceeded { limit_mb: u64 },

    #[error("Wrong answer on test {test_id}")]
    WrongAnswer { test_id: u32 },

    #[error("IO error: {0}")]
    IoError(#[from] io::Error),

    #[error("Parse error: {0}")]
    ParseError(#[from] std::num::ParseIntError),
}

// Utilisation avec ? operator
fn run_submission(code: &str, input: &str) -> Result<String, JudgeError> {
    let compiled = compile(code)?;  // Propagation automatique
    let output = execute(&compiled, input)?;
    Ok(output)
}

fn compile(code: &str) -> Result<Vec<u8>, JudgeError> {
    if code.contains("syntax_error") {
        Err(JudgeError::CompilationError("Invalid syntax".into()))
    } else {
        Ok(code.as_bytes().to_vec())
    }
}

fn execute(compiled: &[u8], input: &str) -> Result<String, JudgeError> {
    // Simulation
    Ok(format!("Output for: {}", input))
}

// anyhow pour application code
fn main_with_anyhow() -> AnyhowResult<()> {
    let config = std::fs::read_to_string("config.toml")
        .context("Failed to read config file")?;

    let port: u16 = config.lines()
        .find(|l| l.starts_with("port"))
        .context("No port in config")?
        .split('=')
        .nth(1)
        .context("Invalid port format")?
        .trim()
        .parse()
        .context("Port must be a number")?;

    info!("Starting server on port {}", port);
    Ok(())
}

// ============================================================
// 1.9.0.d - Input validation
// ============================================================

#[derive(Debug)]
pub struct ValidatedInput {
    pub n: usize,
    pub values: Vec<i64>,
}

impl ValidatedInput {
    pub fn parse(input: &str) -> Result<Self, JudgeError> {
        let lines: Vec<&str> = input.lines().collect();

        // Validation: au moins une ligne
        if lines.is_empty() {
            return Err(JudgeError::RuntimeError("Empty input".into()));
        }

        // Validation: n dans les limites
        let n: usize = lines[0].trim().parse()?;
        if n == 0 || n > 100_000 {
            return Err(JudgeError::RuntimeError(
                format!("n must be in [1, 100000], got {}", n)
            ));
        }

        // Validation: nombre de valeurs correct
        if lines.len() < 2 {
            return Err(JudgeError::RuntimeError("Missing values line".into()));
        }

        let values: Vec<i64> = lines[1]
            .split_whitespace()
            .map(|s| s.parse())
            .collect::<Result<Vec<_>, _>>()?;

        if values.len() != n {
            return Err(JudgeError::RuntimeError(
                format!("Expected {} values, got {}", n, values.len())
            ));
        }

        // Validation: valeurs dans les limites
        for &v in &values {
            if v < -1_000_000_000 || v > 1_000_000_000 {
                return Err(JudgeError::RuntimeError(
                    format!("Value {} out of range [-10^9, 10^9]", v)
                ));
            }
        }

        Ok(ValidatedInput { n, values })
    }
}

// ============================================================
// 1.9.0.e - Logging avec tracing
// ============================================================

use tracing_subscriber::{fmt, prelude::*, EnvFilter};

pub fn setup_logging() {
    tracing_subscriber::registry()
        .with(fmt::layer())
        .with(EnvFilter::from_default_env())
        .init();
}

#[instrument(skip(input))]
pub fn process_submission(submission_id: u64, input: &str) -> Result<String, JudgeError> {
    info!(submission_id, "Processing submission");

    debug!("Input length: {}", input.len());

    let result = run_submission("code", input);

    match &result {
        Ok(output) => info!(submission_id, output_len = output.len(), "Success"),
        Err(e) => warn!(submission_id, error = %e, "Submission failed"),
    }

    result
}

// ============================================================
// 1.9.0.f - Panic safety
// ============================================================

/// Exécute du code utilisateur de manière sécurisée
pub fn safe_execute<F, T>(f: F) -> Result<T, String>
where
    F: FnOnce() -> T + panic::UnwindSafe,
{
    panic::catch_unwind(f)
        .map_err(|e| {
            if let Some(s) = e.downcast_ref::<&str>() {
                format!("Panic: {}", s)
            } else if let Some(s) = e.downcast_ref::<String>() {
                format!("Panic: {}", s)
            } else {
                "Unknown panic".to_string()
            }
        })
}

/// Custom panic handler pour production
pub fn setup_panic_handler() {
    panic::set_hook(Box::new(|info| {
        let location = info.location().map(|l| format!("{}:{}:{}", l.file(), l.line(), l.column()))
            .unwrap_or_else(|| "unknown".to_string());

        let message = if let Some(s) = info.payload().downcast_ref::<&str>() {
            s.to_string()
        } else if let Some(s) = info.payload().downcast_ref::<String>() {
            s.clone()
        } else {
            "Unknown panic".to_string()
        };

        error!(location, message, "PANIC CAUGHT");
    }));
}

// ============================================================
// 1.9.0.g - Code review: Identifier patterns compétition
// ============================================================

/// Code review checklist pour identifier du code "compétition" en production
///
/// RED FLAGS:
/// 1. `unwrap()` ou `expect()` sans justification
/// 2. `panic!()` pour gérer les erreurs
/// 3. Variables nommées `a`, `b`, `n`, `m`
/// 4. Pas de tests unitaires
/// 5. Magic numbers sans constantes
/// 6. Stdin/stdout direct sans abstraction
/// 7. Pas de logging
/// 8. Clone excessif sans raison
/// 9. `unsafe` injustifié
/// 10. Pas de documentation

pub fn code_review_competition_patterns(code: &str) -> Vec<String> {
    let mut issues = Vec::new();

    if code.contains(".unwrap()") {
        issues.push("Found unwrap() - use ? or handle error".into());
    }

    if code.contains("panic!") {
        issues.push("Found panic! - use Result/Option".into());
    }

    // Single letter variables (heuristic)
    let single_letter = regex::Regex::new(r"\blet ([a-z])\s*[=:]").unwrap();
    for cap in single_letter.captures_iter(code) {
        issues.push(format!("Single letter variable '{}' - use descriptive name", &cap[1]));
    }

    if !code.contains("#[test]") && !code.contains("mod tests") {
        issues.push("No tests found".into());
    }

    if code.contains("stdin()") && !code.contains("trait") && !code.contains("impl") {
        issues.push("Direct stdin usage - consider abstraction for testing".into());
    }

    issues
}

// ============================================================
// 1.9.0.h - Refactoring: Competition → Production
// ============================================================

// AVANT: Code compétition
mod before {
    pub fn solve() {
        let mut s = String::new();
        std::io::stdin().read_line(&mut s).unwrap();
        let n: i32 = s.trim().parse().unwrap();
        let mut s2 = String::new();
        std::io::stdin().read_line(&mut s2).unwrap();
        let v: Vec<i32> = s2.split_whitespace()
            .map(|x| x.parse().unwrap())
            .collect();
        let ans: i32 = v.iter().sum();
        println!("{}", ans);
    }
}

// APRÈS: Code production
mod after {
    use std::io::{BufRead, Write};
    use thiserror::Error;

    #[derive(Error, Debug)]
    pub enum SolveError {
        #[error("IO error: {0}")]
        Io(#[from] std::io::Error),
        #[error("Parse error: {0}")]
        Parse(#[from] std::num::ParseIntError),
        #[error("Invalid input: {0}")]
        Invalid(String),
    }

    pub struct Input {
        pub count: usize,
        pub values: Vec<i32>,
    }

    impl Input {
        pub fn read<R: BufRead>(reader: &mut R) -> Result<Self, SolveError> {
            let mut line = String::new();

            reader.read_line(&mut line)?;
            let count: usize = line.trim().parse()?;

            line.clear();
            reader.read_line(&mut line)?;
            let values: Vec<i32> = line
                .split_whitespace()
                .map(|s| s.parse())
                .collect::<Result<_, _>>()?;

            if values.len() != count {
                return Err(SolveError::Invalid(
                    format!("Expected {} values, got {}", count, values.len())
                ));
            }

            Ok(Input { count, values })
        }
    }

    pub fn solve(input: &Input) -> i64 {
        input.values.iter().map(|&x| x as i64).sum()
    }

    pub fn run<R: BufRead, W: Write>(reader: &mut R, writer: &mut W) -> Result<(), SolveError> {
        let input = Input::read(reader)?;
        let answer = solve(&input);
        writeln!(writer, "{}", answer)?;
        Ok(())
    }

    #[cfg(test)]
    mod tests {
        use super::*;

        #[test]
        fn test_basic() {
            let input = Input { count: 3, values: vec![1, 2, 3] };
            assert_eq!(solve(&input), 6);
        }

        #[test]
        fn test_empty() {
            let input = Input { count: 0, values: vec![] };
            assert_eq!(solve(&input), 0);
        }

        #[test]
        fn test_negative() {
            let input = Input { count: 3, values: vec![-1, 0, 1] };
            assert_eq!(solve(&input), 0);
        }
    }
}
```

### Test Moulinette
```
comp_vs_prod validate "3\n1 2 3" -> Ok(ValidatedInput{n:3, values:[1,2,3]})
comp_vs_prod validate "" -> Err("Empty input")
comp_vs_prod safe_execute panic -> Err("Panic: ...")
comp_vs_prod review "x.unwrap()" -> ["Found unwrap()..."]
```

---

# SECTION 2: RÉVISION DATA STRUCTURES

## Exercice B1: `data_structures_revision`
**Couvre: 1.9.1.a-i (9 concepts)**

### Concepts
- [1.9.1.a] Arrays & Vectors — Edge cases, iterators
- [1.9.1.b] Hash Tables — Custom hashers
- [1.9.1.c] Trees — Toutes rotations
- [1.9.1.d] Heaps — Min/Max, custom Ord
- [1.9.1.e] Segment Trees — Lazy propagation
- [1.9.1.f] Fenwick Trees — 2D extensions
- [1.9.1.g] Tries — Compressed
- [1.9.1.h] Union-Find — Rollback
- [1.9.1.i] Sparse Tables — LCA, RMQ

### Rust
```rust
use std::collections::{HashMap, BinaryHeap};
use std::cmp::{Ordering, Reverse};
use std::hash::{Hash, Hasher, BuildHasherDefault};

// ============================================================
// 1.9.1.a - Arrays & Vectors: Edge cases
// ============================================================

pub fn vec_edge_cases() {
    // Empty
    let empty: Vec<i32> = vec![];
    assert_eq!(empty.first(), None);
    assert_eq!(empty.last(), None);
    assert_eq!(empty.get(0), None);

    // Single element
    let single = vec![42];
    assert_eq!(single.first(), single.last());

    // Duplicates
    let dups = vec![1, 1, 1, 2, 2, 3];
    let unique: Vec<_> = dups.iter().collect::<std::collections::HashSet<_>>()
        .into_iter().collect();

    // Overflow prevention
    let big: Vec<i32> = vec![i32::MAX, 1];
    let safe_sum: i64 = big.iter().map(|&x| x as i64).sum();

    // Iterator patterns
    let nums = vec![1, 2, 3, 4, 5];
    let _sum: i32 = nums.iter().sum();
    let _prod: i32 = nums.iter().product();
    let _max = nums.iter().max();
    let _min = nums.iter().min();
    let _enumerated: Vec<_> = nums.iter().enumerate().collect();
    let _windowed: Vec<_> = nums.windows(2).collect();
    let _chunked: Vec<_> = nums.chunks(2).collect();
}

// ============================================================
// 1.9.1.b - Hash Tables: Custom hashers
// ============================================================

use std::collections::hash_map::DefaultHasher;

// FxHash - faster for small keys
#[derive(Default)]
pub struct FxHasher {
    hash: u64,
}

impl Hasher for FxHasher {
    fn write(&mut self, bytes: &[u8]) {
        const K: u64 = 0x517cc1b727220a95;
        for &byte in bytes {
            self.hash = self.hash.rotate_left(5).bitxor(byte as u64).wrapping_mul(K);
        }
    }

    fn finish(&self) -> u64 {
        self.hash
    }
}

type FxHashMap<K, V> = HashMap<K, V, BuildHasherDefault<FxHasher>>;

pub fn custom_hasher_demo() {
    let mut map: FxHashMap<i32, i32> = FxHashMap::default();
    map.insert(1, 100);
    map.insert(2, 200);
    assert_eq!(map.get(&1), Some(&100));
}

// ============================================================
// 1.9.1.c - Trees: All rotations
// ============================================================

#[derive(Debug)]
pub struct AVLNode<T> {
    value: T,
    left: Option<Box<AVLNode<T>>>,
    right: Option<Box<AVLNode<T>>>,
    height: i32,
}

impl<T: Ord> AVLNode<T> {
    pub fn new(value: T) -> Self {
        AVLNode { value, left: None, right: None, height: 1 }
    }

    fn height(node: &Option<Box<AVLNode<T>>>) -> i32 {
        node.as_ref().map_or(0, |n| n.height)
    }

    fn balance_factor(&self) -> i32 {
        Self::height(&self.left) - Self::height(&self.right)
    }

    fn update_height(&mut self) {
        self.height = 1 + Self::height(&self.left).max(Self::height(&self.right));
    }

    // Right rotation (LL case)
    fn rotate_right(mut self: Box<Self>) -> Box<Self> {
        let mut new_root = self.left.take().unwrap();
        self.left = new_root.right.take();
        self.update_height();
        new_root.right = Some(self);
        new_root.update_height();
        new_root
    }

    // Left rotation (RR case)
    fn rotate_left(mut self: Box<Self>) -> Box<Self> {
        let mut new_root = self.right.take().unwrap();
        self.right = new_root.left.take();
        self.update_height();
        new_root.left = Some(self);
        new_root.update_height();
        new_root
    }

    // Balance after insertion
    fn balance(mut self: Box<Self>) -> Box<Self> {
        self.update_height();
        let bf = self.balance_factor();

        if bf > 1 {
            // Left heavy
            if Self::height(&self.left.as_ref().unwrap().left) >=
               Self::height(&self.left.as_ref().unwrap().right) {
                // LL case
                return self.rotate_right();
            } else {
                // LR case
                self.left = Some(self.left.take().unwrap().rotate_left());
                return self.rotate_right();
            }
        }

        if bf < -1 {
            // Right heavy
            if Self::height(&self.right.as_ref().unwrap().right) >=
               Self::height(&self.right.as_ref().unwrap().left) {
                // RR case
                return self.rotate_left();
            } else {
                // RL case
                self.right = Some(self.right.take().unwrap().rotate_right());
                return self.rotate_left();
            }
        }

        self
    }
}

// ============================================================
// 1.9.1.d - Heaps: Min/Max, custom Ord
// ============================================================

// Custom ordering for heap
#[derive(Eq, PartialEq)]
struct Task {
    priority: i32,
    name: String,
}

impl Ord for Task {
    fn cmp(&self, other: &Self) -> Ordering {
        // Higher priority first (max-heap behavior)
        self.priority.cmp(&other.priority)
    }
}

impl PartialOrd for Task {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        Some(self.cmp(other))
    }
}

pub fn heap_demo() {
    // Max heap (default)
    let mut max_heap = BinaryHeap::new();
    max_heap.push(3);
    max_heap.push(1);
    max_heap.push(4);
    assert_eq!(max_heap.pop(), Some(4));

    // Min heap using Reverse
    let mut min_heap = BinaryHeap::new();
    min_heap.push(Reverse(3));
    min_heap.push(Reverse(1));
    min_heap.push(Reverse(4));
    assert_eq!(min_heap.pop(), Some(Reverse(1)));

    // Custom ordering
    let mut task_heap = BinaryHeap::new();
    task_heap.push(Task { priority: 1, name: "low".into() });
    task_heap.push(Task { priority: 10, name: "high".into() });
    assert_eq!(task_heap.pop().unwrap().name, "high");
}

// ============================================================
// 1.9.1.e - Segment Trees: Lazy propagation
// ============================================================

pub struct LazySegmentTree {
    n: usize,
    tree: Vec<i64>,
    lazy: Vec<i64>,
}

impl LazySegmentTree {
    pub fn new(arr: &[i64]) -> Self {
        let n = arr.len();
        let mut st = Self {
            n,
            tree: vec![0; 4 * n],
            lazy: vec![0; 4 * n],
        };
        st.build(arr, 1, 0, n - 1);
        st
    }

    fn build(&mut self, arr: &[i64], node: usize, start: usize, end: usize) {
        if start == end {
            self.tree[node] = arr[start];
        } else {
            let mid = (start + end) / 2;
            self.build(arr, 2 * node, start, mid);
            self.build(arr, 2 * node + 1, mid + 1, end);
            self.tree[node] = self.tree[2 * node] + self.tree[2 * node + 1];
        }
    }

    fn push_down(&mut self, node: usize, start: usize, end: usize) {
        if self.lazy[node] != 0 {
            self.tree[node] += self.lazy[node] * (end - start + 1) as i64;
            if start != end {
                self.lazy[2 * node] += self.lazy[node];
                self.lazy[2 * node + 1] += self.lazy[node];
            }
            self.lazy[node] = 0;
        }
    }

    pub fn range_update(&mut self, l: usize, r: usize, val: i64) {
        self.update_helper(1, 0, self.n - 1, l, r, val);
    }

    fn update_helper(&mut self, node: usize, start: usize, end: usize, l: usize, r: usize, val: i64) {
        self.push_down(node, start, end);
        if start > r || end < l {
            return;
        }
        if start >= l && end <= r {
            self.lazy[node] += val;
            self.push_down(node, start, end);
            return;
        }
        let mid = (start + end) / 2;
        self.update_helper(2 * node, start, mid, l, r, val);
        self.update_helper(2 * node + 1, mid + 1, end, l, r, val);
        self.tree[node] = self.tree[2 * node] + self.tree[2 * node + 1];
    }

    pub fn range_query(&mut self, l: usize, r: usize) -> i64 {
        self.query_helper(1, 0, self.n - 1, l, r)
    }

    fn query_helper(&mut self, node: usize, start: usize, end: usize, l: usize, r: usize) -> i64 {
        self.push_down(node, start, end);
        if start > r || end < l {
            return 0;
        }
        if start >= l && end <= r {
            return self.tree[node];
        }
        let mid = (start + end) / 2;
        self.query_helper(2 * node, start, mid, l, r) +
        self.query_helper(2 * node + 1, mid + 1, end, l, r)
    }
}

// ============================================================
// 1.9.1.f - Fenwick Trees: 2D extensions
// ============================================================

pub struct Fenwick2D {
    n: usize,
    m: usize,
    tree: Vec<Vec<i64>>,
}

impl Fenwick2D {
    pub fn new(n: usize, m: usize) -> Self {
        Self {
            n,
            m,
            tree: vec![vec![0; m + 1]; n + 1],
        }
    }

    pub fn update(&mut self, mut x: usize, mut y: usize, delta: i64) {
        let orig_y = y;
        while x <= self.n {
            y = orig_y;
            while y <= self.m {
                self.tree[x][y] += delta;
                y += y & y.wrapping_neg();
            }
            x += x & x.wrapping_neg();
        }
    }

    fn prefix_sum(&self, mut x: usize, mut y: usize) -> i64 {
        let mut sum = 0;
        let orig_y = y;
        while x > 0 {
            y = orig_y;
            while y > 0 {
                sum += self.tree[x][y];
                y -= y & y.wrapping_neg();
            }
            x -= x & x.wrapping_neg();
        }
        sum
    }

    pub fn range_sum(&self, x1: usize, y1: usize, x2: usize, y2: usize) -> i64 {
        self.prefix_sum(x2, y2)
            - self.prefix_sum(x1 - 1, y2)
            - self.prefix_sum(x2, y1 - 1)
            + self.prefix_sum(x1 - 1, y1 - 1)
    }
}

// ============================================================
// 1.9.1.g - Tries: Compressed
// ============================================================

pub struct CompressedTrie {
    children: HashMap<String, Box<CompressedTrie>>,
    is_end: bool,
}

impl CompressedTrie {
    pub fn new() -> Self {
        Self { children: HashMap::new(), is_end: false }
    }

    pub fn insert(&mut self, word: &str) {
        if word.is_empty() {
            self.is_end = true;
            return;
        }

        // Find longest common prefix with existing edges
        for (edge, child) in &mut self.children {
            let common = word.chars().zip(edge.chars())
                .take_while(|(a, b)| a == b)
                .count();

            if common > 0 {
                if common == edge.len() {
                    // Edge is prefix of word
                    child.insert(&word[common..]);
                    return;
                } else {
                    // Split the edge
                    let remaining_edge = edge[common..].to_string();
                    let remaining_word = word[common..].to_string();

                    let mut new_child = Box::new(CompressedTrie::new());
                    let old_child = std::mem::replace(child, Box::new(CompressedTrie::new()));

                    new_child.children.insert(remaining_edge, old_child);
                    if remaining_word.is_empty() {
                        new_child.is_end = true;
                    } else {
                        let mut word_child = Box::new(CompressedTrie::new());
                        word_child.is_end = true;
                        new_child.children.insert(remaining_word, word_child);
                    }

                    let prefix = edge[..common].to_string();
                    self.children.remove(edge);
                    self.children.insert(prefix, new_child);
                    return;
                }
            }
        }

        // No common prefix, insert new edge
        let mut child = Box::new(CompressedTrie::new());
        child.is_end = true;
        self.children.insert(word.to_string(), child);
    }

    pub fn search(&self, word: &str) -> bool {
        if word.is_empty() {
            return self.is_end;
        }

        for (edge, child) in &self.children {
            if word.starts_with(edge) {
                return child.search(&word[edge.len()..]);
            }
        }
        false
    }
}

// ============================================================
// 1.9.1.h - Union-Find: Rollback
// ============================================================

pub struct DSURollback {
    parent: Vec<usize>,
    rank: Vec<usize>,
    history: Vec<(usize, usize, usize)>,  // (node, old_parent, old_rank)
}

impl DSURollback {
    pub fn new(n: usize) -> Self {
        Self {
            parent: (0..n).collect(),
            rank: vec![0; n],
            history: Vec::new(),
        }
    }

    pub fn find(&self, mut x: usize) -> usize {
        while self.parent[x] != x {
            x = self.parent[x];
        }
        x
    }

    pub fn union(&mut self, x: usize, y: usize) -> bool {
        let px = self.find(x);
        let py = self.find(y);

        if px == py {
            return false;
        }

        // Save state for rollback
        self.history.push((px, self.parent[px], self.rank[px]));
        self.history.push((py, self.parent[py], self.rank[py]));

        // Union by rank (without path compression for rollback support)
        if self.rank[px] < self.rank[py] {
            self.parent[px] = py;
        } else if self.rank[px] > self.rank[py] {
            self.parent[py] = px;
        } else {
            self.parent[py] = px;
            self.rank[px] += 1;
        }
        true
    }

    pub fn checkpoint(&self) -> usize {
        self.history.len()
    }

    pub fn rollback(&mut self, checkpoint: usize) {
        while self.history.len() > checkpoint {
            let (node, old_parent, old_rank) = self.history.pop().unwrap();
            self.parent[node] = old_parent;
            self.rank[node] = old_rank;
        }
    }
}

// ============================================================
// 1.9.1.i - Sparse Tables: LCA, RMQ
// ============================================================

pub struct SparseTable {
    table: Vec<Vec<usize>>,  // Stores indices
    arr: Vec<i32>,
    log: Vec<usize>,
}

impl SparseTable {
    pub fn new(arr: Vec<i32>) -> Self {
        let n = arr.len();
        let max_log = (n as f64).log2().floor() as usize + 1;

        // Precompute logs
        let mut log = vec![0; n + 1];
        for i in 2..=n {
            log[i] = log[i / 2] + 1;
        }

        // Build sparse table
        let mut table = vec![vec![0; n]; max_log];

        // Initialize with indices
        for i in 0..n {
            table[0][i] = i;
        }

        // Fill table
        for j in 1..max_log {
            for i in 0..n {
                if i + (1 << j) <= n {
                    let left = table[j - 1][i];
                    let right = table[j - 1][i + (1 << (j - 1))];
                    table[j][i] = if arr[left] <= arr[right] { left } else { right };
                }
            }
        }

        Self { table, arr, log }
    }

    /// Range Minimum Query in O(1)
    pub fn rmq(&self, l: usize, r: usize) -> i32 {
        let j = self.log[r - l + 1];
        let left = self.table[j][l];
        let right = self.table[j][r - (1 << j) + 1];
        self.arr[left].min(self.arr[right])
    }

    /// Get index of minimum in range
    pub fn rmq_index(&self, l: usize, r: usize) -> usize {
        let j = self.log[r - l + 1];
        let left = self.table[j][l];
        let right = self.table[j][r - (1 << j) + 1];
        if self.arr[left] <= self.arr[right] { left } else { right }
    }
}
```

### Test Moulinette
```
ds_revision vec_edges empty_first -> None
ds_revision heap max [3,1,4] pop -> 4
ds_revision heap min [3,1,4] pop -> 1
ds_revision segtree [1,2,3,4,5] range_sum 1 3 -> 9
ds_revision fenwick2d update (1,1,5) sum (1,1,1,1) -> 5
ds_revision dsu_rollback union 0 1 rollback -> separate
ds_revision sparse [3,1,4,1,5] rmq 1 3 -> 1
```

---

# SECTION 3: RÉVISION ALGORITHMES

## Exercice C1: `algorithms_revision`
**Couvre: 1.9.2.a-k (11 concepts)**

### Concepts
- [1.9.2.a] Merge Sort — O(n log n)
- [1.9.2.b] Quick Sort — O(n log n) avg
- [1.9.2.c] Counting Sort — O(n + k)
- [1.9.2.d] Binary Search — O(log n)
- [1.9.2.e] BFS — O(V + E)
- [1.9.2.f] DFS — O(V + E)
- [1.9.2.g] Dijkstra — O((V+E) log V)
- [1.9.2.h] Bellman-Ford — O(VE)
- [1.9.2.i] Floyd-Warshall — O(V³)
- [1.9.2.j] Kruskal — O(E log E)
- [1.9.2.k] Prim — O((V+E) log V)

### Rust
```rust
use std::collections::{BinaryHeap, VecDeque};
use std::cmp::Reverse;

// ============================================================
// 1.9.2.a - Merge Sort: O(n log n)
// ============================================================

pub fn merge_sort<T: Ord + Clone>(arr: &mut [T]) {
    let n = arr.len();
    if n <= 1 {
        return;
    }

    let mid = n / 2;
    merge_sort(&mut arr[..mid]);
    merge_sort(&mut arr[mid..]);

    let left: Vec<T> = arr[..mid].to_vec();
    let right: Vec<T> = arr[mid..].to_vec();

    let mut i = 0;
    let mut j = 0;
    let mut k = 0;

    while i < left.len() && j < right.len() {
        if left[i] <= right[j] {
            arr[k] = left[i].clone();
            i += 1;
        } else {
            arr[k] = right[j].clone();
            j += 1;
        }
        k += 1;
    }

    while i < left.len() {
        arr[k] = left[i].clone();
        i += 1;
        k += 1;
    }

    while j < right.len() {
        arr[k] = right[j].clone();
        j += 1;
        k += 1;
    }
}

// ============================================================
// 1.9.2.b - Quick Sort: O(n log n) average
// ============================================================

pub fn quick_sort<T: Ord>(arr: &mut [T]) {
    if arr.len() <= 1 {
        return;
    }

    let pivot_idx = partition(arr);
    quick_sort(&mut arr[..pivot_idx]);
    quick_sort(&mut arr[pivot_idx + 1..]);
}

fn partition<T: Ord>(arr: &mut [T]) -> usize {
    let len = arr.len();
    let pivot_idx = len / 2;
    arr.swap(pivot_idx, len - 1);

    let mut i = 0;
    for j in 0..len - 1 {
        if arr[j] < arr[len - 1] {
            arr.swap(i, j);
            i += 1;
        }
    }
    arr.swap(i, len - 1);
    i
}

// ============================================================
// 1.9.2.c - Counting Sort: O(n + k)
// ============================================================

pub fn counting_sort(arr: &mut [usize], max_val: usize) {
    let mut count = vec![0; max_val + 1];

    for &x in arr.iter() {
        count[x] += 1;
    }

    let mut idx = 0;
    for (val, &cnt) in count.iter().enumerate() {
        for _ in 0..cnt {
            arr[idx] = val;
            idx += 1;
        }
    }
}

// ============================================================
// 1.9.2.d - Binary Search: O(log n)
// ============================================================

pub fn binary_search<T: Ord>(arr: &[T], target: &T) -> Result<usize, usize> {
    let mut lo = 0;
    let mut hi = arr.len();

    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        match arr[mid].cmp(target) {
            std::cmp::Ordering::Less => lo = mid + 1,
            std::cmp::Ordering::Greater => hi = mid,
            std::cmp::Ordering::Equal => return Ok(mid),
        }
    }
    Err(lo)
}

pub fn lower_bound<T: Ord>(arr: &[T], target: &T) -> usize {
    let mut lo = 0;
    let mut hi = arr.len();

    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if arr[mid] < *target {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    lo
}

pub fn upper_bound<T: Ord>(arr: &[T], target: &T) -> usize {
    let mut lo = 0;
    let mut hi = arr.len();

    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if arr[mid] <= *target {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    lo
}

// ============================================================
// 1.9.2.e - BFS: O(V + E)
// ============================================================

pub fn bfs(graph: &[Vec<usize>], start: usize) -> Vec<i32> {
    let n = graph.len();
    let mut dist = vec![-1; n];
    let mut queue = VecDeque::new();

    dist[start] = 0;
    queue.push_back(start);

    while let Some(u) = queue.pop_front() {
        for &v in &graph[u] {
            if dist[v] == -1 {
                dist[v] = dist[u] + 1;
                queue.push_back(v);
            }
        }
    }

    dist
}

// ============================================================
// 1.9.2.f - DFS: O(V + E)
// ============================================================

pub fn dfs(graph: &[Vec<usize>], start: usize) -> Vec<usize> {
    let mut visited = vec![false; graph.len()];
    let mut order = Vec::new();

    fn dfs_visit(graph: &[Vec<usize>], u: usize, visited: &mut Vec<bool>, order: &mut Vec<usize>) {
        visited[u] = true;
        order.push(u);
        for &v in &graph[u] {
            if !visited[v] {
                dfs_visit(graph, v, visited, order);
            }
        }
    }

    dfs_visit(graph, start, &mut visited, &mut order);
    order
}

// Iterative DFS
pub fn dfs_iterative(graph: &[Vec<usize>], start: usize) -> Vec<usize> {
    let mut visited = vec![false; graph.len()];
    let mut order = Vec::new();
    let mut stack = vec![start];

    while let Some(u) = stack.pop() {
        if visited[u] {
            continue;
        }
        visited[u] = true;
        order.push(u);

        for &v in graph[u].iter().rev() {
            if !visited[v] {
                stack.push(v);
            }
        }
    }

    order
}

// ============================================================
// 1.9.2.g - Dijkstra: O((V+E) log V)
// ============================================================

pub fn dijkstra(graph: &[Vec<(usize, i64)>], start: usize) -> Vec<i64> {
    let n = graph.len();
    let mut dist = vec![i64::MAX; n];
    let mut heap = BinaryHeap::new();

    dist[start] = 0;
    heap.push(Reverse((0i64, start)));

    while let Some(Reverse((d, u))) = heap.pop() {
        if d > dist[u] {
            continue;
        }

        for &(v, w) in &graph[u] {
            let new_dist = dist[u] + w;
            if new_dist < dist[v] {
                dist[v] = new_dist;
                heap.push(Reverse((new_dist, v)));
            }
        }
    }

    dist
}

// ============================================================
// 1.9.2.h - Bellman-Ford: O(VE)
// ============================================================

pub fn bellman_ford(n: usize, edges: &[(usize, usize, i64)], start: usize) -> Option<Vec<i64>> {
    let mut dist = vec![i64::MAX; n];
    dist[start] = 0;

    // Relax all edges V-1 times
    for _ in 0..n - 1 {
        for &(u, v, w) in edges {
            if dist[u] != i64::MAX && dist[u] + w < dist[v] {
                dist[v] = dist[u] + w;
            }
        }
    }

    // Check for negative cycles
    for &(u, v, w) in edges {
        if dist[u] != i64::MAX && dist[u] + w < dist[v] {
            return None;  // Negative cycle detected
        }
    }

    Some(dist)
}

// ============================================================
// 1.9.2.i - Floyd-Warshall: O(V³)
// ============================================================

pub fn floyd_warshall(n: usize, edges: &[(usize, usize, i64)]) -> Vec<Vec<i64>> {
    let mut dist = vec![vec![i64::MAX / 2; n]; n];

    // Initialize
    for i in 0..n {
        dist[i][i] = 0;
    }
    for &(u, v, w) in edges {
        dist[u][v] = dist[u][v].min(w);
    }

    // DP
    for k in 0..n {
        for i in 0..n {
            for j in 0..n {
                if dist[i][k] + dist[k][j] < dist[i][j] {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }

    dist
}

// ============================================================
// 1.9.2.j - Kruskal: O(E log E)
// ============================================================

pub fn kruskal(n: usize, edges: &mut [(usize, usize, i64)]) -> (i64, Vec<(usize, usize)>) {
    edges.sort_by_key(|e| e.2);

    let mut parent: Vec<usize> = (0..n).collect();
    let mut rank = vec![0; n];

    fn find(parent: &mut [usize], x: usize) -> usize {
        if parent[x] != x {
            parent[x] = find(parent, parent[x]);
        }
        parent[x]
    }

    fn union(parent: &mut [usize], rank: &mut [usize], x: usize, y: usize) -> bool {
        let px = find(parent, x);
        let py = find(parent, y);
        if px == py {
            return false;
        }
        if rank[px] < rank[py] {
            parent[px] = py;
        } else if rank[px] > rank[py] {
            parent[py] = px;
        } else {
            parent[py] = px;
            rank[px] += 1;
        }
        true
    }

    let mut mst = Vec::new();
    let mut total_weight = 0;

    for &(u, v, w) in edges.iter() {
        if union(&mut parent, &mut rank, u, v) {
            mst.push((u, v));
            total_weight += w;
        }
    }

    (total_weight, mst)
}

// ============================================================
// 1.9.2.k - Prim: O((V+E) log V)
// ============================================================

pub fn prim(graph: &[Vec<(usize, i64)>]) -> (i64, Vec<(usize, usize)>) {
    let n = graph.len();
    let mut visited = vec![false; n];
    let mut heap = BinaryHeap::new();
    let mut mst = Vec::new();
    let mut total_weight = 0;

    // Start from node 0
    visited[0] = true;
    for &(v, w) in &graph[0] {
        heap.push(Reverse((w, 0, v)));
    }

    while let Some(Reverse((w, u, v))) = heap.pop() {
        if visited[v] {
            continue;
        }

        visited[v] = true;
        total_weight += w;
        mst.push((u, v));

        for &(next, weight) in &graph[v] {
            if !visited[next] {
                heap.push(Reverse((weight, v, next)));
            }
        }
    }

    (total_weight, mst)
}
```

### Test Moulinette
```
algo merge_sort [3,1,4,1,5] -> [1,1,3,4,5]
algo quick_sort [3,1,4,1,5] -> [1,1,3,4,5]
algo counting_sort [3,1,4,1,5] 5 -> [1,1,3,4,5]
algo binary_search [1,2,3,4,5] 3 -> Ok(2)
algo lower_bound [1,2,2,3] 2 -> 1
algo bfs [[1,2],[0,3],[0,3],[1,2]] 0 -> [0,1,1,2]
algo dijkstra [[(1,1),(2,4)],[(2,2)],[]] 0 -> [0,1,3]
algo kruskal 3 [(0,1,1),(1,2,2),(0,2,3)] -> (3, [(0,1),(1,2)])
```

---

# SECTION 4: WASM SANDBOXING

## Exercice D1: `wasm_sandboxing`
**Couvre: 1.9.9.a-h (8 concepts)**

### Concepts
- [1.9.9.a] WebAssembly basics — Format binaire, WASI, targets
- [1.9.9.b] Wasmtime — Runtime Rust pour exécuter Wasm
- [1.9.9.c] Memory limits — `StoreLimitsBuilder`, isolation mémoire
- [1.9.9.d] CPU timeout — `epoch_deadline`, interruption
- [1.9.9.e] I/O capture — Stdin injection, stdout capture
- [1.9.9.f] Multi-language — Rust, C, Python → Wasm
- [1.9.9.g] Security — Isolation filesystem, network
- [1.9.9.h] Integration — Judge end-to-end avec Wasm

### Rust
```rust
use wasmtime::*;
use std::sync::Arc;
use std::time::{Duration, Instant};

// ============================================================
// 1.9.9.a - WebAssembly basics
// ============================================================

/// WebAssembly (Wasm) est un format binaire portable
/// - Compact et efficace
/// - Sandboxé par défaut (pas d'accès direct au système)
/// - WASI = WebAssembly System Interface (accès standardisé aux syscalls)
///
/// Targets:
/// - wasm32-unknown-unknown: Pure Wasm, pas d'I/O
/// - wasm32-wasi: Avec WASI pour I/O

/// Compiler un programme Rust en Wasm:
/// ```bash
/// # Installer le target
/// rustup target add wasm32-wasi
///
/// # Compiler
/// cargo build --target wasm32-wasi --release
///
/// # Le binaire est dans target/wasm32-wasi/release/*.wasm
/// ```

// ============================================================
// 1.9.9.b - Wasmtime: Runtime Rust pour exécuter Wasm
// ============================================================

pub struct WasmRunner {
    engine: Engine,
}

impl WasmRunner {
    pub fn new() -> Result<Self> {
        let engine = Engine::default();
        Ok(Self { engine })
    }

    /// Charge et exécute un module Wasm simple
    pub fn run_simple(&self, wasm_bytes: &[u8]) -> Result<()> {
        let module = Module::new(&self.engine, wasm_bytes)?;
        let mut store = Store::new(&self.engine, ());
        let instance = Instance::new(&mut store, &module, &[])?;

        // Appeler la fonction "main" ou "_start"
        if let Some(start) = instance.get_func(&mut store, "_start") {
            start.call(&mut store, &[], &mut [])?;
        }

        Ok(())
    }
}

// ============================================================
// 1.9.9.c - Memory limits
// ============================================================

pub struct SandboxConfig {
    pub memory_limit_bytes: u64,
    pub cpu_time_limit_ms: u64,
}

impl Default for SandboxConfig {
    fn default() -> Self {
        Self {
            memory_limit_bytes: 256 * 1024 * 1024,  // 256 MB
            cpu_time_limit_ms: 5000,  // 5 seconds
        }
    }
}

pub fn create_limited_store(engine: &Engine, config: &SandboxConfig) -> Store<()> {
    let mut store = Store::new(engine, ());

    // Configurer les limites de mémoire
    store.limiter(|_| {
        StoreLimitsBuilder::new()
            .memory_size(config.memory_limit_bytes as usize)
            .build()
    });

    store
}

// ============================================================
// 1.9.9.d - CPU timeout avec epoch
// ============================================================

pub fn create_engine_with_epoch() -> Engine {
    let mut config = Config::new();
    config.epoch_interruption(true);
    Engine::new(&config).unwrap()
}

pub fn run_with_timeout(
    engine: &Engine,
    module: &Module,
    timeout: Duration,
) -> Result<(), String> {
    let mut store = Store::new(engine, ());

    // Définir le deadline
    store.set_epoch_deadline(1);

    // Thread qui incrémente l'epoch après le timeout
    let engine_clone = engine.clone();
    let handle = std::thread::spawn(move || {
        std::thread::sleep(timeout);
        engine_clone.increment_epoch();
    });

    // Exécuter
    let instance = Instance::new(&mut store, module, &[])
        .map_err(|e| e.to_string())?;

    if let Some(start) = instance.get_func(&mut store, "_start") {
        match start.call(&mut store, &[], &mut []) {
            Ok(_) => {}
            Err(e) => {
                if e.to_string().contains("epoch") {
                    return Err("Time limit exceeded".to_string());
                }
                return Err(e.to_string());
            }
        }
    }

    handle.join().ok();
    Ok(())
}

// ============================================================
// 1.9.9.e - I/O capture
// ============================================================

use wasi_common::pipe::{ReadPipe, WritePipe};
use wasmtime_wasi::{WasiCtx, WasiCtxBuilder};

pub struct CapturedIO {
    pub stdout: String,
    pub stderr: String,
    pub exit_code: Option<i32>,
}

pub fn run_with_io_capture(
    engine: &Engine,
    module: &Module,
    stdin_data: &str,
) -> Result<CapturedIO, String> {
    // Créer les pipes
    let stdin = ReadPipe::from(stdin_data.as_bytes().to_vec());
    let stdout = WritePipe::new_in_memory();
    let stderr = WritePipe::new_in_memory();

    // Configurer WASI
    let wasi = WasiCtxBuilder::new()
        .stdin(Box::new(stdin))
        .stdout(Box::new(stdout.clone()))
        .stderr(Box::new(stderr.clone()))
        .build();

    let mut store = Store::new(engine, wasi);

    // Linker avec WASI
    let mut linker = Linker::new(engine);
    wasmtime_wasi::add_to_linker(&mut linker, |s| s)
        .map_err(|e| e.to_string())?;

    let instance = linker.instantiate(&mut store, module)
        .map_err(|e| e.to_string())?;

    // Exécuter
    let exit_code = if let Some(start) = instance.get_func(&mut store, "_start") {
        match start.call(&mut store, &[], &mut []) {
            Ok(_) => Some(0),
            Err(e) => {
                // Extraire le code de sortie si disponible
                if e.to_string().contains("exit") {
                    Some(1)
                } else {
                    return Err(e.to_string());
                }
            }
        }
    } else {
        None
    };

    // Récupérer les sorties
    drop(store);  // Libérer les références

    let stdout_bytes = stdout.try_into_inner()
        .map_err(|_| "Failed to get stdout")?
        .into_inner();
    let stderr_bytes = stderr.try_into_inner()
        .map_err(|_| "Failed to get stderr")?
        .into_inner();

    Ok(CapturedIO {
        stdout: String::from_utf8_lossy(&stdout_bytes).to_string(),
        stderr: String::from_utf8_lossy(&stderr_bytes).to_string(),
        exit_code,
    })
}

// ============================================================
// 1.9.9.f - Multi-language support
// ============================================================

/// Compiler différents langages vers Wasm:
///
/// Rust:
/// ```bash
/// rustup target add wasm32-wasi
/// cargo build --target wasm32-wasi
/// ```
///
/// C:
/// ```bash
/// # Installer WASI SDK: https://github.com/WebAssembly/wasi-sdk
/// export WASI_SDK_PATH=/opt/wasi-sdk
/// $WASI_SDK_PATH/bin/clang --sysroot=$WASI_SDK_PATH/share/wasi-sysroot \
///     -o program.wasm program.c
/// ```
///
/// Python:
/// ```bash
/// # Utiliser Pyodide ou RustPython compilé en Wasm
/// # Plus complexe, nécessite un runtime Python en Wasm
/// ```

#[derive(Debug, Clone, Copy)]
pub enum Language {
    Rust,
    C,
    Cpp,
    Python,  // Via RustPython/Pyodide
}

impl Language {
    pub fn compile_command(&self, source: &str, output: &str) -> Vec<String> {
        match self {
            Language::Rust => vec![
                "rustc".to_string(),
                "--target".to_string(),
                "wasm32-wasi".to_string(),
                "-o".to_string(),
                output.to_string(),
                source.to_string(),
            ],
            Language::C => vec![
                "clang".to_string(),
                "--target=wasm32-wasi".to_string(),
                "-o".to_string(),
                output.to_string(),
                source.to_string(),
            ],
            Language::Cpp => vec![
                "clang++".to_string(),
                "--target=wasm32-wasi".to_string(),
                "-o".to_string(),
                output.to_string(),
                source.to_string(),
            ],
            Language::Python => {
                // Python nécessite un traitement spécial
                vec![]
            }
        }
    }
}

// ============================================================
// 1.9.9.g - Security: Isolation
// ============================================================

/// Garanties de sécurité Wasm:
///
/// 1. Isolation mémoire:
///    - Le module ne peut accéder qu'à sa propre mémoire linéaire
///    - Pas d'accès à la mémoire du host
///
/// 2. Pas d'accès système par défaut:
///    - Pas de filesystem
///    - Pas de réseau
///    - Pas d'horloge système (sauf si explicitement autorisé)
///
/// 3. Capabilities-based (WASI):
///    - L'hôte décide ce qui est accessible
///    - Principe du moindre privilège

pub fn create_secure_wasi_ctx() -> WasiCtx {
    WasiCtxBuilder::new()
        // Pas d'accès au filesystem
        // Pas d'héritage d'environnement
        // Pas d'arguments
        // Stdin/stdout contrôlés
        .stdin(Box::new(ReadPipe::from(vec![])))
        .stdout(Box::new(WritePipe::new_in_memory()))
        .stderr(Box::new(WritePipe::new_in_memory()))
        .build()
}

/// Vérifier qu'un module Wasm n'importe pas de fonctions dangereuses
pub fn audit_wasm_imports(module: &Module) -> Vec<String> {
    let mut warnings = Vec::new();

    for import in module.imports() {
        let name = format!("{}::{}", import.module(), import.name());

        // Lister les imports potentiellement dangereux
        if name.contains("fd_") && !name.contains("fd_write") && !name.contains("fd_read") {
            warnings.push(format!("Suspicious import: {}", name));
        }

        if name.contains("sock_") {
            warnings.push(format!("Network access attempted: {}", name));
        }

        if name.contains("path_") || name.contains("fd_prestat") {
            warnings.push(format!("Filesystem access attempted: {}", name));
        }
    }

    warnings
}

// ============================================================
// 1.9.9.h - Integration: Judge end-to-end
// ============================================================

#[derive(Debug)]
pub enum JudgeResult {
    Accepted,
    WrongAnswer { expected: String, got: String },
    TimeLimitExceeded,
    MemoryLimitExceeded,
    RuntimeError(String),
    CompilationError(String),
}

pub struct WasmJudge {
    engine: Engine,
    config: SandboxConfig,
}

impl WasmJudge {
    pub fn new(config: SandboxConfig) -> Self {
        let engine = create_engine_with_epoch();
        Self { engine, config }
    }

    pub fn judge(
        &self,
        wasm_bytes: &[u8],
        input: &str,
        expected_output: &str,
    ) -> JudgeResult {
        // Charger le module
        let module = match Module::new(&self.engine, wasm_bytes) {
            Ok(m) => m,
            Err(e) => return JudgeResult::CompilationError(e.to_string()),
        };

        // Audit de sécurité
        let warnings = audit_wasm_imports(&module);
        if !warnings.is_empty() {
            return JudgeResult::RuntimeError(
                format!("Security violation: {:?}", warnings)
            );
        }

        // Exécuter avec capture I/O
        let timeout = Duration::from_millis(self.config.cpu_time_limit_ms);
        let result = run_with_timeout_and_io(&self.engine, &module, input, timeout);

        match result {
            Ok(output) => {
                let trimmed_output = output.stdout.trim();
                let trimmed_expected = expected_output.trim();

                if trimmed_output == trimmed_expected {
                    JudgeResult::Accepted
                } else {
                    JudgeResult::WrongAnswer {
                        expected: trimmed_expected.to_string(),
                        got: trimmed_output.to_string(),
                    }
                }
            }
            Err(e) if e.contains("Time limit") => JudgeResult::TimeLimitExceeded,
            Err(e) if e.contains("Memory") => JudgeResult::MemoryLimitExceeded,
            Err(e) => JudgeResult::RuntimeError(e),
        }
    }
}

fn run_with_timeout_and_io(
    engine: &Engine,
    module: &Module,
    input: &str,
    timeout: Duration,
) -> Result<CapturedIO, String> {
    // Combiner timeout et capture I/O
    // (Implémentation simplifiée)
    run_with_io_capture(engine, module, input)
}

// ============================================================
// Demo complète
// ============================================================

pub fn demonstrate_wasm_judge() {
    println!("=== Wasm Judge Demo ===\n");

    // Configuration
    let config = SandboxConfig {
        memory_limit_bytes: 64 * 1024 * 1024,  // 64 MB
        cpu_time_limit_ms: 2000,  // 2 seconds
    };

    let judge = WasmJudge::new(config);

    // Exemple de test case
    // En production, wasm_bytes viendrait de la compilation
    // let result = judge.judge(&wasm_bytes, "5\n", "120\n");

    println!("Judge configured:");
    println!("  Memory limit: {} MB", 64);
    println!("  Time limit: {} ms", 2000);
    println!("\nReady to judge submissions!");
}
```

### Test Moulinette
```
wasm create_engine -> Ok(Engine)
wasm config default memory -> 268435456
wasm config default cpu_ms -> 5000
wasm audit_imports [fd_write,fd_read] -> []
wasm audit_imports [sock_open] -> ["Network access attempted"]
wasm judge accepted "42\n" "42\n" -> Accepted
wasm judge wrong "42\n" "43\n" -> WrongAnswer
```

---

# RÉCAPITULATIF MODULE 1.9

| Section | Exercice | Concepts | Count |
|---------|----------|----------|-------|
| Compétition vs Production | A1 | 1.9.0.a-h | 8 |
| Révision Data Structures | B1 | 1.9.1.a-i | 9 |
| Révision Algorithmes | C1 | 1.9.2.a-k | 11 |
| Wasm Sandboxing | D1 | 1.9.9.a-h | 8 |
| **TOTAL** | | | **36** |

**Couverture Module 1.9: 36/36 = 100%**

