# Exercise 02: Fuzzing & Mutation Testing

## Concepts Covered
- **1.8.4.d-l** Fuzzing strategies, coverage-guided fuzzing
- **1.8.5.d-k** Mutation testing, mutation operators

## Objective

Implement fuzzing and mutation testing techniques for finding bugs.

## Requirements

### Rust Implementation

```rust
pub mod fuzzing {
    /// Basic random fuzzer
    pub struct RandomFuzzer {
        max_input_size: usize,
        seed: u64,
    }

    impl RandomFuzzer {
        pub fn new(max_size: usize) -> Self;
        pub fn with_seed(self, seed: u64) -> Self;

        /// Generate random bytes
        pub fn generate_bytes(&mut self) -> Vec<u8>;

        /// Generate random string
        pub fn generate_string(&mut self) -> String;

        /// Fuzz target function
        pub fn fuzz<F>(&mut self, target: F, iterations: usize) -> FuzzResult
        where
            F: Fn(&[u8]) -> bool;  // Returns true if crashed
    }

    #[derive(Debug)]
    pub struct FuzzResult {
        pub crashes: Vec<Vec<u8>>,
        pub iterations: usize,
        pub coverage: Option<f64>,
    }

    /// Mutation-based fuzzer
    pub struct MutationFuzzer {
        corpus: Vec<Vec<u8>>,
        mutations: Vec<Box<dyn Fn(&mut Vec<u8>)>>,
    }

    impl MutationFuzzer {
        pub fn new() -> Self;
        pub fn add_seed(&mut self, seed: Vec<u8>);
        pub fn add_mutation(&mut self, mutation: Box<dyn Fn(&mut Vec<u8>)>);

        /// Default mutations
        pub fn with_default_mutations(self) -> Self;

        /// Mutate input
        pub fn mutate(&self, input: &[u8]) -> Vec<u8>;

        /// Fuzz with mutations
        pub fn fuzz<F>(&mut self, target: F, iterations: usize) -> FuzzResult
        where
            F: Fn(&[u8]) -> bool;
    }

    /// Common mutations
    pub mod mutations {
        /// Bit flip at random position
        pub fn bit_flip(input: &mut Vec<u8>);

        /// Byte flip
        pub fn byte_flip(input: &mut Vec<u8>);

        /// Insert random byte
        pub fn insert_byte(input: &mut Vec<u8>);

        /// Delete random byte
        pub fn delete_byte(input: &mut Vec<u8>);

        /// Replace with interesting value
        pub fn interesting_values(input: &mut Vec<u8>);

        /// Arithmetic mutation (add/subtract small value)
        pub fn arithmetic(input: &mut Vec<u8>);

        /// Havoc (multiple random mutations)
        pub fn havoc(input: &mut Vec<u8>);

        /// Splice two inputs
        pub fn splice(input1: &[u8], input2: &[u8]) -> Vec<u8>;
    }
}

pub mod coverage {
    /// Coverage tracker
    pub struct CoverageTracker {
        covered_branches: std::collections::HashSet<u64>,
        total_branches: usize,
    }

    impl CoverageTracker {
        pub fn new() -> Self;
        pub fn record_branch(&mut self, branch_id: u64);
        pub fn coverage_ratio(&self) -> f64;
        pub fn new_coverage(&self, branches: &[u64]) -> bool;
    }

    /// Coverage-guided fuzzer
    pub struct CoverageGuidedFuzzer {
        corpus: Vec<(Vec<u8>, std::collections::HashSet<u64>)>,
        coverage: CoverageTracker,
    }

    impl CoverageGuidedFuzzer {
        pub fn new() -> Self;

        /// Fuzz with coverage feedback
        pub fn fuzz<F, C>(
            &mut self,
            target: F,
            get_coverage: C,
            iterations: usize,
        ) -> FuzzResult
        where
            F: Fn(&[u8]) -> bool,
            C: Fn() -> Vec<u64>;

        /// Prioritize inputs that increase coverage
        fn select_and_mutate(&mut self) -> Vec<u8>;
    }
}

pub mod mutation_testing {
    /// Mutation operator
    pub trait MutationOperator {
        fn name(&self) -> &str;
        fn apply(&self, code: &str) -> Vec<String>;  // Returns mutated versions
    }

    /// Common mutation operators
    pub mod operators {
        /// Replace arithmetic operators
        pub struct ArithmeticOp;

        /// Replace comparison operators
        pub struct RelationalOp;

        /// Negate conditions
        pub struct NegateCondition;

        /// Remove statements
        pub struct StatementDeletion;

        /// Replace constants
        pub struct ConstantMutation;

        /// Replace boolean literals
        pub struct BooleanMutation;

        /// Remove method calls
        pub struct RemoveCall;

        /// Change return values
        pub struct ReturnMutation;
    }

    /// Mutation testing framework
    pub struct MutationTester {
        operators: Vec<Box<dyn MutationOperator>>,
    }

    impl MutationTester {
        pub fn new() -> Self;
        pub fn add_operator(&mut self, op: Box<dyn MutationOperator>);
        pub fn with_default_operators(self) -> Self;

        /// Generate mutants from source code
        pub fn generate_mutants(&self, source: &str) -> Vec<Mutant>;

        /// Run tests against mutants
        pub fn test_mutants<F>(&self, mutants: &[Mutant], run_tests: F) -> MutationResult
        where
            F: Fn(&str) -> bool;  // Returns true if tests pass
    }

    #[derive(Debug)]
    pub struct Mutant {
        pub id: usize,
        pub operator: String,
        pub original: String,
        pub mutated: String,
        pub location: (usize, usize),  // (line, column)
    }

    #[derive(Debug)]
    pub struct MutationResult {
        pub total_mutants: usize,
        pub killed: usize,
        pub survived: Vec<Mutant>,
        pub equivalent: usize,
        pub mutation_score: f64,
    }
}

pub mod grammar_fuzzing {
    /// Grammar-based fuzzer
    pub struct GrammarFuzzer {
        grammar: Grammar,
        max_depth: usize,
    }

    #[derive(Clone)]
    pub struct Grammar {
        rules: std::collections::HashMap<String, Vec<Vec<Symbol>>>,
        start: String,
    }

    #[derive(Clone)]
    pub enum Symbol {
        Terminal(String),
        NonTerminal(String),
    }

    impl GrammarFuzzer {
        pub fn new(grammar: Grammar, max_depth: usize) -> Self;

        /// Generate valid input from grammar
        pub fn generate(&self) -> String;

        /// Mutate while maintaining validity
        pub fn mutate(&self, input: &str) -> String;
    }

    /// Common grammars
    pub fn json_grammar() -> Grammar;
    pub fn xml_grammar() -> Grammar;
    pub fn arithmetic_grammar() -> Grammar;
    pub fn url_grammar() -> Grammar;
}
```

### Python Implementation

```python
from typing import List, Callable, Set, Dict, Tuple
from dataclasses import dataclass
import random

class RandomFuzzer:
    def __init__(self, max_size: int):
        self.max_size = max_size

    def generate_bytes(self) -> bytes: ...
    def fuzz(self, target: Callable[[bytes], bool], iterations: int) -> 'FuzzResult': ...

class MutationFuzzer:
    def __init__(self):
        self.corpus: List[bytes] = []

    def add_seed(self, seed: bytes) -> None: ...
    def mutate(self, input: bytes) -> bytes: ...
    def fuzz(self, target: Callable[[bytes], bool], iterations: int) -> 'FuzzResult': ...

@dataclass
class FuzzResult:
    crashes: List[bytes]
    iterations: int
    coverage: float = 0.0

# Mutations
def bit_flip(input: bytes) -> bytes: ...
def byte_flip(input: bytes) -> bytes: ...
def insert_byte(input: bytes) -> bytes: ...
def delete_byte(input: bytes) -> bytes: ...

class CoverageGuidedFuzzer:
    def __init__(self): ...
    def fuzz(self, target, get_coverage, iterations: int) -> FuzzResult: ...

@dataclass
class Mutant:
    id: int
    operator: str
    original: str
    mutated: str

class MutationTester:
    def __init__(self): ...
    def generate_mutants(self, source: str) -> List[Mutant]: ...
    def test_mutants(self, mutants: List[Mutant], run_tests: Callable[[str], bool]) -> 'MutationResult': ...

@dataclass
class MutationResult:
    total_mutants: int
    killed: int
    survived: List[Mutant]
    mutation_score: float
```

## Test Cases

```rust
#[test]
fn test_random_fuzzer() {
    let mut fuzzer = RandomFuzzer::new(100).with_seed(42);

    // Simple crash on input containing "CRASH"
    let result = fuzzer.fuzz(
        |input| {
            if let Ok(s) = std::str::from_utf8(input) {
                s.contains("CRASH")
            } else {
                false
            }
        },
        10000,
    );

    // Unlikely to find "CRASH" randomly, but should run without issues
    assert!(result.iterations == 10000);
}

#[test]
fn test_mutation_fuzzer() {
    let mut fuzzer = MutationFuzzer::new()
        .with_default_mutations();

    fuzzer.add_seed(b"hello".to_vec());
    fuzzer.add_seed(b"world".to_vec());

    let mutated = fuzzer.mutate(b"hello");
    assert_ne!(mutated, b"hello".to_vec());
}

#[test]
fn test_bit_flip() {
    let mut input = vec![0b11111111];
    mutations::bit_flip(&mut input);
    assert_ne!(input[0], 0b11111111);
}

#[test]
fn test_interesting_values() {
    let mut input = vec![0, 0, 0, 0];
    mutations::interesting_values(&mut input);
    // Should contain an interesting value like 0, 255, -1, etc.
}

#[test]
fn test_coverage_guided() {
    let mut fuzzer = CoverageGuidedFuzzer::new();

    // Simulated target with branches
    fn target(input: &[u8]) -> bool {
        if input.len() >= 4 {
            if input[0] == b'F' {
                if input[1] == b'U' {
                    if input[2] == b'Z' {
                        if input[3] == b'Z' {
                            return true;  // Crash!
                        }
                    }
                }
            }
        }
        false
    }

    // Coverage returns branch IDs
    fn get_coverage() -> Vec<u64> {
        // Would be instrumented in real implementation
        vec![]
    }

    let result = fuzzer.fuzz(target, get_coverage, 100000);
    // Coverage-guided should find "FUZZ" faster than random
}

#[test]
fn test_arithmetic_mutation() {
    let op = operators::ArithmeticOp;
    let code = "let x = a + b;";

    let mutants = op.apply(code);
    assert!(mutants.contains(&"let x = a - b;".to_string()));
    assert!(mutants.contains(&"let x = a * b;".to_string()));
}

#[test]
fn test_relational_mutation() {
    let op = operators::RelationalOp;
    let code = "if x > 0 { }";

    let mutants = op.apply(code);
    assert!(mutants.contains(&"if x >= 0 { }".to_string()));
    assert!(mutants.contains(&"if x < 0 { }".to_string()));
}

#[test]
fn test_mutation_testing() {
    let tester = MutationTester::new()
        .with_default_operators();

    let source = r#"
        fn add(a: i32, b: i32) -> i32 {
            a + b
        }
    "#;

    let mutants = tester.generate_mutants(source);
    assert!(!mutants.is_empty());

    // Run with good tests (should kill all mutants)
    let result = tester.test_mutants(&mutants, |mutated_code| {
        // Test: add(2, 3) == 5
        // Good test should fail on mutants like a - b
        false  // Simplified: assume test fails on all mutants
    });

    assert_eq!(result.mutation_score, 1.0);
}

#[test]
fn test_survived_mutants() {
    let tester = MutationTester::new()
        .with_default_operators();

    let source = r#"
        fn is_positive(x: i32) -> bool {
            x > 0
        }
    "#;

    let mutants = tester.generate_mutants(source);

    // Weak test that doesn't test boundary
    let result = tester.test_mutants(&mutants, |mutated_code| {
        // Only tests is_positive(5) == true
        // Would miss x >= 0 mutation
        true  // Test passes on mutant
    });

    // Should have survived mutants
    assert!(!result.survived.is_empty() || result.mutation_score < 1.0);
}

#[test]
fn test_grammar_json() {
    let grammar = json_grammar();
    let fuzzer = GrammarFuzzer::new(grammar, 5);

    for _ in 0..100 {
        let json = fuzzer.generate();
        // Should be valid JSON syntax
        assert!(json.starts_with('{') || json.starts_with('[')
                || json.starts_with('"') || json.chars().next().unwrap().is_numeric());
    }
}

#[test]
fn test_grammar_arithmetic() {
    let grammar = arithmetic_grammar();
    let fuzzer = GrammarFuzzer::new(grammar, 3);

    for _ in 0..100 {
        let expr = fuzzer.generate();
        // Should be valid arithmetic expression
        // e.g., "1 + 2", "3 * (4 + 5)"
        assert!(!expr.is_empty());
    }
}

#[test]
fn test_splice_mutation() {
    let input1 = b"Hello World";
    let input2 = b"Goodbye Moon";

    let spliced = mutations::splice(input1, input2);

    // Should contain parts of both
    assert!(spliced.len() > 0);
}

#[test]
fn test_coverage_tracker() {
    let mut tracker = CoverageTracker::new();

    tracker.record_branch(1);
    tracker.record_branch(2);
    tracker.record_branch(3);

    assert!(tracker.new_coverage(&[4, 5]));
    assert!(!tracker.new_coverage(&[1, 2]));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Random fuzzer | 10 |
| Mutation fuzzer | 15 |
| Mutation operations | 15 |
| Coverage-guided fuzzing | 20 |
| Mutation operators | 15 |
| Mutation testing framework | 15 |
| Grammar-based fuzzing | 5 |
| Edge cases | 5 |
| **Total** | **100** |
