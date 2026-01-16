# Exercise 00: Unit Testing Fundamentals

## Concepts Covered
- **1.8.1.f-h** Test frameworks, organization, naming conventions
- **1.8.2.e-g** Parametrized tests, mocking, coverage
- **1.8.3.e-f** TDD benefits, practice

## Objective

Master unit testing in Rust and Python with proper organization and coverage.

## Requirements

### Rust Implementation

Create a library with comprehensive tests:

```rust
// lib.rs - Code to test
pub mod calculator {
    #[derive(Debug, PartialEq)]
    pub enum CalcError {
        DivisionByZero,
        Overflow,
        InvalidInput,
    }

    pub fn add(a: i64, b: i64) -> Result<i64, CalcError>;
    pub fn subtract(a: i64, b: i64) -> Result<i64, CalcError>;
    pub fn multiply(a: i64, b: i64) -> Result<i64, CalcError>;
    pub fn divide(a: i64, b: i64) -> Result<i64, CalcError>;
    pub fn power(base: i64, exp: u32) -> Result<i64, CalcError>;
    pub fn factorial(n: u32) -> Result<u64, CalcError>;
    pub fn gcd(a: i64, b: i64) -> i64;
    pub fn is_prime(n: u64) -> bool;
}

pub mod string_utils {
    pub fn reverse(s: &str) -> String;
    pub fn is_palindrome(s: &str) -> bool;
    pub fn word_count(s: &str) -> usize;
    pub fn most_common_char(s: &str) -> Option<char>;
}

pub mod data_structures {
    pub struct Stack<T> { /* ... */ }
    impl<T> Stack<T> {
        pub fn new() -> Self;
        pub fn push(&mut self, item: T);
        pub fn pop(&mut self) -> Option<T>;
        pub fn peek(&self) -> Option<&T>;
        pub fn is_empty(&self) -> bool;
        pub fn len(&self) -> usize;
    }
}
```

### Test Organization

```rust
// tests/calculator_tests.rs
use your_crate::calculator::*;

mod add_tests {
    use super::*;

    #[test]
    fn test_add_positive_numbers() {
        assert_eq!(add(2, 3), Ok(5));
    }

    #[test]
    fn test_add_negative_numbers() {
        assert_eq!(add(-2, -3), Ok(-5));
    }

    #[test]
    fn test_add_overflow() {
        assert_eq!(add(i64::MAX, 1), Err(CalcError::Overflow));
    }
}

mod divide_tests {
    use super::*;

    #[test]
    fn test_divide_normal() {
        assert_eq!(divide(10, 2), Ok(5));
    }

    #[test]
    fn test_divide_by_zero() {
        assert_eq!(divide(10, 0), Err(CalcError::DivisionByZero));
    }
}

// Parametrized tests using macro
macro_rules! test_cases {
    ($($name:ident: $input:expr => $expected:expr),* $(,)?) => {
        $(
            #[test]
            fn $name() {
                let (a, b) = $input;
                assert_eq!(gcd(a, b), $expected);
            }
        )*
    };
}

test_cases! {
    gcd_12_8: (12, 8) => 4,
    gcd_17_13: (17, 13) => 1,
    gcd_100_25: (100, 25) => 25,
    gcd_with_zero: (0, 5) => 5,
}
```

### Python Tests with pytest

```python
# test_calculator.py
import pytest
from calculator import add, divide, CalcError, is_prime

class TestAdd:
    def test_positive_numbers(self):
        assert add(2, 3) == 5

    def test_negative_numbers(self):
        assert add(-2, -3) == -5

    @pytest.mark.parametrize("a,b,expected", [
        (1, 1, 2),
        (0, 0, 0),
        (-1, 1, 0),
        (100, 200, 300),
    ])
    def test_add_parametrized(self, a, b, expected):
        assert add(a, b) == expected

class TestDivide:
    def test_normal_division(self):
        assert divide(10, 2) == 5

    def test_division_by_zero(self):
        with pytest.raises(CalcError) as exc:
            divide(10, 0)
        assert exc.value == CalcError.DIVISION_BY_ZERO

class TestIsPrime:
    @pytest.mark.parametrize("n,expected", [
        (2, True),
        (3, True),
        (4, False),
        (17, True),
        (18, False),
        (1, False),
        (0, False),
    ])
    def test_is_prime(self, n, expected):
        assert is_prime(n) == expected

# Fixtures
@pytest.fixture
def sample_stack():
    from data_structures import Stack
    s = Stack()
    s.push(1)
    s.push(2)
    s.push(3)
    return s

def test_stack_pop(sample_stack):
    assert sample_stack.pop() == 3
    assert sample_stack.pop() == 2

# Mocking
from unittest.mock import Mock, patch

def test_with_mock():
    mock_db = Mock()
    mock_db.get_user.return_value = {"name": "Alice", "age": 30}

    result = some_function_using_db(mock_db)
    mock_db.get_user.assert_called_once_with(user_id=1)
```

### TDD Practice

Implement using Test-Driven Development:

1. **Red**: Write failing test first
2. **Green**: Write minimal code to pass
3. **Refactor**: Improve code quality

```rust
// Step 1: Write test first
#[test]
fn test_fizzbuzz() {
    assert_eq!(fizzbuzz(1), "1");
    assert_eq!(fizzbuzz(3), "Fizz");
    assert_eq!(fizzbuzz(5), "Buzz");
    assert_eq!(fizzbuzz(15), "FizzBuzz");
}

// Step 2: Implement to pass
pub fn fizzbuzz(n: u32) -> String {
    match (n % 3, n % 5) {
        (0, 0) => "FizzBuzz".to_string(),
        (0, _) => "Fizz".to_string(),
        (_, 0) => "Buzz".to_string(),
        _ => n.to_string(),
    }
}

// Step 3: Refactor if needed
```

## Test Cases for Your Tests

```rust
// Meta-tests: test that your tests are comprehensive
#[test]
fn test_coverage_calculator() {
    // Ensure all edge cases are covered
    // - Zero inputs
    // - Negative inputs
    // - Overflow conditions
    // - Boundary values
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Calculator tests (all operations) | 20 |
| String utils tests | 15 |
| Data structure tests | 15 |
| Parametrized tests | 10 |
| Edge case coverage | 15 |
| Test organization | 10 |
| TDD demonstration | 10 |
| Documentation | 5 |
| **Total** | **100** |

## Coverage Requirements

- Minimum 90% line coverage
- All public functions tested
- All error cases tested
- Boundary conditions tested

## Files to Submit

### Rust
- `src/lib.rs` - Implementation
- `tests/` - Test modules
- `Cargo.toml`

### Python
- `calculator.py` - Implementation
- `test_calculator.py` - Tests
- `pytest.ini` - Configuration
