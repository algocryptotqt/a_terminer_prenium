# Exercise 00: Generic Vector

## Concepts Covered
- **1.1.a** Generic Vector
- **1.1.b** All operations (push, pop, insert, remove)
- **1.1.c** Intelligent resize strategy
- **1.1.9.j** Resize strategy analysis
- **1.1.9.k** Amortized proof
- **1.1.9.l** Shrink strategy
- **1.1.9.m** Reserve capacity
- **1.1.9.n** Clear operation

## Objective

Implement a generic dynamic array (vector) from scratch that supports:
1. Dynamic growth with amortized O(1) push
2. Intelligent shrinking to reclaim memory
3. Capacity reservation
4. All standard operations

## Requirements

### Rust Implementation

Create a `GenericVec<T>` struct with the following API:

```rust
pub struct GenericVec<T> {
    // Your implementation
}

impl<T> GenericVec<T> {
    /// Creates an empty vector
    pub fn new() -> Self;

    /// Creates a vector with pre-allocated capacity
    pub fn with_capacity(capacity: usize) -> Self;

    /// Returns the number of elements
    pub fn len(&self) -> usize;

    /// Returns true if the vector is empty
    pub fn is_empty(&self) -> bool;

    /// Returns the current capacity
    pub fn capacity(&self) -> usize;

    /// Adds an element to the end - O(1) amortized
    pub fn push(&mut self, value: T);

    /// Removes and returns the last element - O(1)
    pub fn pop(&mut self) -> Option<T>;

    /// Inserts an element at index - O(n)
    pub fn insert(&mut self, index: usize, value: T);

    /// Removes and returns element at index - O(n)
    pub fn remove(&mut self, index: usize) -> T;

    /// Returns a reference to element at index
    pub fn get(&self, index: usize) -> Option<&T>;

    /// Returns a mutable reference to element at index
    pub fn get_mut(&mut self, index: usize) -> Option<&mut T>;

    /// Reserves capacity for at least `additional` more elements
    pub fn reserve(&mut self, additional: usize);

    /// Shrinks capacity to fit current length (with hysteresis)
    pub fn shrink_to_fit(&mut self);

    /// Removes all elements but keeps allocated memory
    pub fn clear(&mut self);
}
```

### Python Implementation

Create a `GenericVec` class:

```python
class GenericVec[T]:
    """Generic dynamic array with intelligent resize strategy."""

    def __init__(self) -> None: ...
    def with_capacity(cls, capacity: int) -> "GenericVec[T]": ...
    def __len__(self) -> int: ...
    def __bool__(self) -> bool: ...
    def capacity(self) -> int: ...
    def push(self, value: T) -> None: ...
    def pop(self) -> T | None: ...
    def insert(self, index: int, value: T) -> None: ...
    def remove(self, index: int) -> T: ...
    def __getitem__(self, index: int) -> T: ...
    def __setitem__(self, index: int, value: T) -> None: ...
    def reserve(self, additional: int) -> None: ...
    def shrink_to_fit(self) -> None: ...
    def clear(self) -> None: ...
```

## Resize Strategy

Your implementation MUST follow this resize strategy:

### Growth Policy
- When capacity is exceeded, grow by factor of 2
- Formula: `new_capacity = max(1, old_capacity * 2)`
- This ensures amortized O(1) push operations

### Shrink Policy (Hysteresis)
- Only shrink when `len < capacity / 4`
- Shrink to `capacity / 2`
- This prevents thrashing at boundaries

### Reserve
- Only reallocate if `capacity < len + additional`
- Round up to next power of 2 for efficiency

## Test Cases

Your implementation must pass all these tests:

```rust
#[test]
fn test_basic_operations() {
    let mut v: GenericVec<i32> = GenericVec::new();
    assert!(v.is_empty());
    assert_eq!(v.len(), 0);

    v.push(1);
    v.push(2);
    v.push(3);
    assert_eq!(v.len(), 3);
    assert!(!v.is_empty());

    assert_eq!(v.pop(), Some(3));
    assert_eq!(v.pop(), Some(2));
    assert_eq!(v.len(), 1);
}

#[test]
fn test_resize_growth() {
    let mut v: GenericVec<i32> = GenericVec::new();
    for i in 0..100 {
        v.push(i);
    }
    assert_eq!(v.len(), 100);
    // Capacity should be power of 2 >= 100
    assert!(v.capacity() >= 100);
    assert!(v.capacity().is_power_of_two());
}

#[test]
fn test_shrink_hysteresis() {
    let mut v: GenericVec<i32> = GenericVec::with_capacity(100);
    for i in 0..50 {
        v.push(i);
    }
    // Should not shrink yet (50 >= 100/4)
    v.shrink_to_fit();
    assert!(v.capacity() >= 50);

    // Pop until len < capacity/4
    while v.len() >= v.capacity() / 4 {
        v.pop();
    }
    v.shrink_to_fit();
    // Now should have shrunk
    assert!(v.capacity() < 100);
}

#[test]
fn test_insert_remove() {
    let mut v: GenericVec<i32> = GenericVec::new();
    v.push(1);
    v.push(3);
    v.insert(1, 2);
    assert_eq!(*v.get(0).unwrap(), 1);
    assert_eq!(*v.get(1).unwrap(), 2);
    assert_eq!(*v.get(2).unwrap(), 3);

    assert_eq!(v.remove(1), 2);
    assert_eq!(v.len(), 2);
}

#[test]
fn test_reserve() {
    let mut v: GenericVec<i32> = GenericVec::new();
    v.reserve(100);
    assert!(v.capacity() >= 100);
    // Push should not cause reallocation
    let cap = v.capacity();
    for i in 0..100 {
        v.push(i);
    }
    assert_eq!(v.capacity(), cap);
}

#[test]
fn test_clear() {
    let mut v: GenericVec<i32> = GenericVec::new();
    for i in 0..50 {
        v.push(i);
    }
    let cap = v.capacity();
    v.clear();
    assert_eq!(v.len(), 0);
    assert!(v.is_empty());
    assert_eq!(v.capacity(), cap); // Keeps capacity
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| All basic operations work | 25 |
| Growth strategy correct (factor 2) | 15 |
| Shrink strategy with hysteresis | 15 |
| Reserve works correctly | 10 |
| Clear keeps capacity | 5 |
| No memory leaks (Rust only) | 10 |
| Handles edge cases (empty, single element) | 10 |
| Code is clean and well-documented | 10 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs` - Your implementation
- `Cargo.toml` - Project configuration

### Python
- `generic_vec.py` - Your implementation

## Hints

1. In Rust, use `std::alloc` for manual memory management or `Box<[MaybeUninit<T>]>`
2. In Python, use a plain list internally but track capacity separately
3. Remember: amortized analysis assumes a sequence of operations, not worst-case individual ops
4. The hysteresis in shrinking prevents pathological cases of repeated grow/shrink
