# Exercise 09: Arena Allocator

## Concepts Covered
- **1.1.A.a** Arena structure
- **1.1.A.b** arena_alloc
- **1.1.A.c** arena_reset
- **1.1.A.d** arena_destroy
- **1.1.A.e** Alignment

## Objective

Implement a bump allocator (arena allocator) for fast, bulk memory allocation. This is a memory management pattern commonly used in game engines, compilers, and parsers.

## Requirements

### Rust Implementation

```rust
pub mod arena {
    use std::alloc::{Layout, alloc, dealloc};
    use std::cell::Cell;
    use std::marker::PhantomData;
    use std::ptr::NonNull;

    /// A simple bump allocator
    pub struct Arena {
        start: NonNull<u8>,
        end: NonNull<u8>,
        ptr: Cell<NonNull<u8>>,
    }

    impl Arena {
        /// Create a new arena with given capacity
        pub fn new(capacity: usize) -> Self;

        /// Allocate memory for a value of type T
        /// Returns None if arena is full
        pub fn alloc<T>(&self, value: T) -> Option<&mut T>;

        /// Allocate a slice of n elements
        pub fn alloc_slice<T: Clone>(&self, value: T, n: usize) -> Option<&mut [T]>;

        /// Allocate uninitialized memory for type T
        /// SAFETY: Caller must initialize before reading
        pub fn alloc_uninit<T>(&self) -> Option<&mut std::mem::MaybeUninit<T>>;

        /// Current used bytes
        pub fn used(&self) -> usize;

        /// Remaining capacity
        pub fn remaining(&self) -> usize;

        /// Total capacity
        pub fn capacity(&self) -> usize;

        /// Reset arena, invalidating all allocations
        /// SAFETY: All references to allocated data become invalid
        pub unsafe fn reset(&self);
    }

    impl Drop for Arena {
        fn drop(&mut self);
    }

    /// Typed arena that only allocates one type
    pub struct TypedArena<T> {
        arena: Arena,
        _marker: PhantomData<T>,
    }

    impl<T> TypedArena<T> {
        pub fn new(capacity: usize) -> Self;
        pub fn alloc(&self, value: T) -> Option<&mut T>;
        pub fn alloc_default(&self) -> Option<&mut T> where T: Default;
    }

    /// Arena with multiple chunks that grows automatically
    pub struct GrowingArena {
        chunks: Vec<Arena>,
        chunk_size: usize,
    }

    impl GrowingArena {
        pub fn new(initial_chunk_size: usize) -> Self;
        pub fn alloc<T>(&mut self, value: T) -> &mut T;
        pub fn alloc_slice<T: Clone>(&mut self, value: T, n: usize) -> &mut [T];
    }
}
```

### Python Implementation

```python
import ctypes
from typing import TypeVar, Generic

T = TypeVar("T")

class Arena:
    """Bump allocator implementation in Python."""

    def __init__(self, capacity: int) -> None: ...

    def alloc_bytes(self, size: int, align: int = 8) -> memoryview | None:
        """Allocate raw bytes with alignment."""
        ...

    def alloc_array(self, dtype: type, count: int) -> list | None:
        """Allocate an array of given type."""
        ...

    def used(self) -> int: ...
    def remaining(self) -> int: ...
    def capacity(self) -> int: ...
    def reset(self) -> None: ...

class TypedArena(Generic[T]):
    """Arena for single type allocation."""

    def __init__(self, item_type: type[T], capacity: int) -> None: ...
    def alloc(self, value: T) -> T | None: ...
    def alloc_default(self) -> T | None: ...

class GrowingArena:
    """Arena that grows by adding chunks."""

    def __init__(self, initial_chunk_size: int = 4096) -> None: ...
    def alloc_bytes(self, size: int, align: int = 8) -> memoryview: ...
    def alloc_array(self, dtype: type, count: int) -> list: ...
```

## Alignment Requirements

Memory must be aligned for correct and efficient access:
- `u8`, `i8`: 1-byte alignment
- `u16`, `i16`: 2-byte alignment
- `u32`, `i32`, `f32`: 4-byte alignment
- `u64`, `i64`, `f64`, pointers: 8-byte alignment
- SIMD types: 16 or 32-byte alignment

```rust
fn align_up(ptr: usize, align: usize) -> usize {
    debug_assert!(align.is_power_of_two());
    (ptr + align - 1) & !(align - 1)
}
```

## Implementation Details

### Bump Allocation Algorithm
```
1. Get current allocation pointer
2. Align pointer to required alignment
3. Check if aligned_ptr + size <= end
4. If yes: advance pointer, return old position
5. If no: return None (out of memory)
```

### Benefits
- O(1) allocation (just pointer bump)
- O(1) mass deallocation (reset pointer)
- Cache-friendly (linear memory layout)
- No fragmentation within arena

### Trade-offs
- No individual deallocation
- Must know maximum size upfront
- All memory freed at once

## Test Cases

```rust
#[test]
fn test_basic_allocation() {
    let arena = Arena::new(1024);

    let a = arena.alloc(42i32).unwrap();
    let b = arena.alloc(3.14f64).unwrap();
    let c = arena.alloc([1u8, 2, 3, 4]).unwrap();

    assert_eq!(*a, 42);
    assert_eq!(*b, 3.14);
    assert_eq!(*c, [1, 2, 3, 4]);
}

#[test]
fn test_alignment() {
    let arena = Arena::new(1024);

    let a = arena.alloc(1u8).unwrap();
    let b = arena.alloc(2u64).unwrap();  // Must be 8-byte aligned

    let a_addr = a as *mut u8 as usize;
    let b_addr = b as *mut u64 as usize;

    assert_eq!(b_addr % 8, 0, "u64 not 8-byte aligned");
}

#[test]
fn test_slice_allocation() {
    let arena = Arena::new(1024);

    let slice = arena.alloc_slice(0i32, 10).unwrap();
    assert_eq!(slice.len(), 10);
    for x in slice.iter() {
        assert_eq!(*x, 0);
    }

    // Modify
    for (i, x) in slice.iter_mut().enumerate() {
        *x = i as i32;
    }
    assert_eq!(slice, &[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);
}

#[test]
fn test_capacity() {
    let arena = Arena::new(100);

    assert_eq!(arena.capacity(), 100);
    assert_eq!(arena.used(), 0);
    assert_eq!(arena.remaining(), 100);

    arena.alloc(42i32);
    assert!(arena.used() >= 4);
    assert!(arena.remaining() <= 96);
}

#[test]
fn test_out_of_memory() {
    let arena = Arena::new(16);

    let a = arena.alloc([0u8; 8]);
    assert!(a.is_some());

    let b = arena.alloc([0u8; 8]);
    assert!(b.is_some());

    let c = arena.alloc([0u8; 8]);
    assert!(c.is_none());  // No more room
}

#[test]
fn test_reset() {
    let arena = Arena::new(100);

    arena.alloc(1i32);
    arena.alloc(2i32);
    arena.alloc(3i32);

    let used_before = arena.used();
    assert!(used_before > 0);

    unsafe { arena.reset(); }

    assert_eq!(arena.used(), 0);
    assert_eq!(arena.remaining(), arena.capacity());
}

#[test]
fn test_growing_arena() {
    let mut arena = GrowingArena::new(64);

    // Allocate more than initial chunk
    for i in 0..100 {
        let _ = arena.alloc(i as i32);
    }

    // Should have grown
    let slice = arena.alloc_slice(0i32, 1000);
    assert_eq!(slice.len(), 1000);
}

#[test]
fn test_typed_arena() {
    #[derive(Default, Debug, PartialEq)]
    struct Point { x: f64, y: f64 }

    let arena: TypedArena<Point> = TypedArena::new(1024);

    let p1 = arena.alloc(Point { x: 1.0, y: 2.0 }).unwrap();
    let p2 = arena.alloc_default().unwrap();

    assert_eq!(*p1, Point { x: 1.0, y: 2.0 });
    assert_eq!(*p2, Point { x: 0.0, y: 0.0 });
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Basic Arena allocation | 20 |
| Correct alignment | 20 |
| Slice allocation | 15 |
| Reset functionality | 10 |
| Growing Arena | 15 |
| Typed Arena | 10 |
| Memory safety (no leaks) | 10 |
| **Total** | **100** |

## Files to Submit

### Rust
- `src/lib.rs`
- `Cargo.toml`

### Python
- `arena.py`

## Use Cases

- **Parser/Compiler**: Allocate AST nodes, free all at once when done
- **Game Engine**: Per-frame allocator, reset each frame
- **Web Server**: Per-request allocator, reset after response
- **Data Processing**: Temporary structures during computation
