# [Module 2.1] - Exercise 10: Advanced Box & Smart Pointers

## Metadonnees

```yaml
module: "2.1 - Memory Management"
exercise: "ex10"
title: "Advanced Box & Smart Pointers"
difficulty: avance
estimated_time: "4 heures"
prerequisite_exercises: ["ex04", "ex07"]
concepts_requis: ["heap_allocation", "ownership", "pointers"]
score_qualite: 98
```

---

## Concepts Couverts

### 2.1.6: Box Advanced Concepts (7 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.6.m | `Box::into_raw()` | Convert to raw pointer |
| 2.1.6.n | Deref for Box | Transparent access via Deref |
| 2.1.6.o | Box et patterns | Destructuring boxes |
| 2.1.6.p | `Box<[T]>` | Boxed slices |
| 2.1.6.q | `Box<dyn Trait>` | Trait objects |
| 2.1.6.r | Custom allocators | `Box::new_in()` |
| 2.1.6.s | `Pin<Box<T>>` | Self-referential structs |

---

## Partie 1: Box to/from Raw Pointers (2.1.6.m)

### Exercice 1.1: Converting Box to Raw Pointer

```rust
//! Box::into_raw() and Box::from_raw()
//!
//! Converting between Box<T> and raw pointers for FFI
//! and manual memory management scenarios.

use std::ptr;

/// Resource that needs manual lifecycle management
#[derive(Debug)]
pub struct Resource {
    id: u32,
    data: Vec<u8>,
}

impl Resource {
    pub fn new(id: u32) -> Self {
        println!("Resource {} created", id);
        Self {
            id,
            data: vec![0; 1024],
        }
    }
}

impl Drop for Resource {
    fn drop(&mut self) {
        println!("Resource {} dropped", self.id);
    }
}

/// Demonstrate Box::into_raw() (2.1.6.m)
pub fn demonstrate_into_raw() {
    println!("=== Box::into_raw() (2.1.6.m) ===\n");

    // Create boxed resource
    let boxed = Box::new(Resource::new(1));
    println!("Created Box<Resource>");

    // Convert to raw pointer - Box is consumed, no drop occurs
    let raw_ptr: *mut Resource = Box::into_raw(boxed);
    println!("Converted to raw pointer: {:p}", raw_ptr);

    // The resource still exists but we're responsible for cleanup
    println!("Resource exists but Box is gone\n");

    // We can use the raw pointer
    unsafe {
        println!("Accessing via raw pointer: id = {}", (*raw_ptr).id);

        // Convert back to Box for proper cleanup
        let boxed_again = Box::from_raw(raw_ptr);
        println!("Converted back to Box");
        // Box will be dropped here
    }

    println!("After scope - resource was properly dropped\n");
}

/// Use case: Passing ownership through FFI
pub mod ffi_example {
    use super::Resource;

    /// Create resource and return opaque handle (2.1.6.m)
    pub fn create_resource(id: u32) -> *mut Resource {
        let resource = Box::new(Resource::new(id));
        Box::into_raw(resource)
    }

    /// Use resource via handle
    pub unsafe fn use_resource(handle: *mut Resource) -> u32 {
        if handle.is_null() {
            return 0;
        }
        (*handle).id
    }

    /// Destroy resource given handle
    pub unsafe fn destroy_resource(handle: *mut Resource) {
        if !handle.is_null() {
            // Reconstruct Box to trigger proper Drop
            drop(Box::from_raw(handle));
        }
    }
}

/// Safe wrapper around FFI handle
pub struct ResourceHandle {
    ptr: *mut Resource,
}

impl ResourceHandle {
    pub fn new(id: u32) -> Self {
        Self {
            ptr: ffi_example::create_resource(id),
        }
    }

    pub fn id(&self) -> u32 {
        unsafe { ffi_example::use_resource(self.ptr) }
    }
}

impl Drop for ResourceHandle {
    fn drop(&mut self) {
        unsafe { ffi_example::destroy_resource(self.ptr) }
    }
}

fn main() {
    demonstrate_into_raw();

    println!("=== Safe FFI Wrapper ===\n");
    {
        let handle = ResourceHandle::new(42);
        println!("Resource ID: {}", handle.id());
    }
    println!("Handle dropped - resource cleaned up\n");
}
```

---

## Partie 2: Deref for Box (2.1.6.n)

### Exercice 2.1: Transparent Access via Deref

```rust
//! Deref trait implementation for Box (2.1.6.n)
//!
//! Box<T> implements Deref<Target=T>, allowing transparent
//! access to the inner value.

use std::ops::{Deref, DerefMut};

/// Custom string type
#[derive(Debug)]
pub struct MyString {
    data: String,
}

impl MyString {
    pub fn new(s: &str) -> Self {
        Self { data: s.to_string() }
    }

    pub fn len(&self) -> usize {
        self.data.len()
    }

    pub fn push_str(&mut self, s: &str) {
        self.data.push_str(s);
    }
}

/// Demonstrate Deref coercion (2.1.6.n)
fn demonstrate_deref() {
    println!("=== Deref for Box (2.1.6.n) ===\n");

    let boxed_string = Box::new(MyString::new("Hello"));

    // Direct method call via Deref coercion
    println!("Length: {}", boxed_string.len());  // Calls MyString::len

    // Explicit deref
    let inner: &MyString = &*boxed_string;
    println!("Inner: {:?}", inner);

    // Deref chain: Box<MyString> -> &MyString -> ...
    // Can access MyString methods directly on Box<MyString>

    // Mutable access via DerefMut
    let mut boxed_mut = Box::new(MyString::new("Hello"));
    boxed_mut.push_str(", World!");  // Via DerefMut
    println!("After push: {:?}", boxed_mut);
}

/// Custom smart pointer implementing Deref
pub struct SmartBox<T> {
    value: Box<T>,
    access_count: std::cell::Cell<usize>,
}

impl<T> SmartBox<T> {
    pub fn new(value: T) -> Self {
        Self {
            value: Box::new(value),
            access_count: std::cell::Cell::new(0),
        }
    }

    pub fn access_count(&self) -> usize {
        self.access_count.get()
    }
}

impl<T> Deref for SmartBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        self.access_count.set(self.access_count.get() + 1);
        &self.value
    }
}

impl<T> DerefMut for SmartBox<T> {
    fn deref_mut(&mut self) -> &mut Self::Target {
        self.access_count.set(self.access_count.get() + 1);
        &mut self.value
    }
}

fn demonstrate_custom_deref() {
    println!("\n=== Custom Deref Implementation ===\n");

    let smart = SmartBox::new(vec![1, 2, 3, 4, 5]);

    // Each access triggers deref
    println!("Length: {}", smart.len());
    println!("First: {:?}", smart.first());
    println!("Sum: {}", smart.iter().sum::<i32>());

    println!("Access count: {}", smart.access_count());
}

fn main() {
    demonstrate_deref();
    demonstrate_custom_deref();
}
```

---

## Partie 3: Box and Patterns (2.1.6.o)

### Exercice 3.1: Destructuring Boxes

```rust
//! Box and pattern matching (2.1.6.o)
//!
//! Boxes can be destructured in patterns using the box keyword
//! (unstable) or through other means.

/// Recursive tree structure requiring Box
#[derive(Debug)]
pub enum Tree<T> {
    Leaf(T),
    Node {
        value: T,
        left: Box<Tree<T>>,
        right: Box<Tree<T>>,
    },
}

impl<T> Tree<T> {
    pub fn leaf(value: T) -> Self {
        Tree::Leaf(value)
    }

    pub fn node(value: T, left: Tree<T>, right: Tree<T>) -> Self {
        Tree::Node {
            value,
            left: Box::new(left),
            right: Box::new(right),
        }
    }
}

/// Pattern matching with Box (2.1.6.o)
impl<T: std::fmt::Debug> Tree<T> {
    pub fn traverse(&self) {
        match self {
            Tree::Leaf(v) => println!("Leaf: {:?}", v),

            // Destructure Box through reference
            Tree::Node { value, left, right } => {
                println!("Node: {:?}", value);
                left.traverse();   // Auto-deref
                right.traverse();
            }
        }
    }

    /// Move out of box via destructuring
    pub fn into_value(self) -> T {
        match self {
            Tree::Leaf(v) => v,
            Tree::Node { value, .. } => value,
        }
    }
}

/// Linked list demonstrating Box patterns
#[derive(Debug)]
pub enum List<T> {
    Nil,
    Cons(T, Box<List<T>>),
}

impl<T> List<T> {
    pub fn new() -> Self {
        List::Nil
    }

    pub fn prepend(self, value: T) -> Self {
        List::Cons(value, Box::new(self))
    }

    /// Pattern matching to extract head (2.1.6.o)
    pub fn head(&self) -> Option<&T> {
        match self {
            List::Nil => None,
            List::Cons(head, _) => Some(head),
        }
    }

    /// Destructuring to get tail
    pub fn tail(&self) -> Option<&List<T>> {
        match self {
            List::Nil => None,
            List::Cons(_, tail) => Some(tail),  // tail is &Box<List<T>>, auto-derefs
        }
    }

    /// Move tail out (consumes self)
    pub fn into_tail(self) -> Option<List<T>> {
        match self {
            List::Nil => None,
            List::Cons(_, tail) => Some(*tail),  // Unbox
        }
    }
}

fn demonstrate_box_patterns() {
    println!("=== Box and Patterns (2.1.6.o) ===\n");

    // Tree patterns
    let tree = Tree::node(
        1,
        Tree::node(2, Tree::leaf(4), Tree::leaf(5)),
        Tree::leaf(3),
    );

    println!("Tree traversal:");
    tree.traverse();

    // List patterns
    let list = List::new()
        .prepend(3)
        .prepend(2)
        .prepend(1);

    println!("\nList head: {:?}", list.head());

    // Destructure in let
    if let List::Cons(head, tail) = &list {
        println!("Head: {}, Tail exists: {}", head, matches!(**tail, List::Cons(..)));
    }
}

fn main() {
    demonstrate_box_patterns();
}
```

---

## Partie 4: Boxed Slices (2.1.6.p)

### Exercice 4.1: Box<[T]> Operations

```rust
//! Box<[T]> - Boxed slices (2.1.6.p)
//!
//! A boxed slice owns its data on the heap but has a fixed length
//! (unlike Vec which can grow).

use std::mem;

/// Demonstrate Box<[T]> (2.1.6.p)
fn demonstrate_boxed_slice() {
    println!("=== Box<[T]> (2.1.6.p) ===\n");

    // Create from Vec (common pattern)
    let vec = vec![1, 2, 3, 4, 5];
    let boxed_slice: Box<[i32]> = vec.into_boxed_slice();

    println!("Boxed slice: {:?}", boxed_slice);
    println!("Length: {}", boxed_slice.len());

    // Size comparison
    println!("Size of Box<[i32]>: {} bytes (fat pointer)", mem::size_of::<Box<[i32]>>());
    println!("Size of Vec<i32>: {} bytes", mem::size_of::<Vec<i32>>());

    // Access elements
    println!("First element: {}", boxed_slice[0]);

    // Iteration
    let sum: i32 = boxed_slice.iter().sum();
    println!("Sum: {}", sum);

    // Convert back to Vec
    let vec_again: Vec<i32> = boxed_slice.into_vec();
    println!("Back to Vec: {:?}", vec_again);
}

/// Create boxed slice from iterator
fn boxed_from_iterator() {
    println!("\n=== Creating Box<[T]> ===\n");

    // From iterator
    let boxed: Box<[i32]> = (1..=10).collect::<Vec<_>>().into_boxed_slice();
    println!("From iterator: {:?}", boxed);

    // From array (sized to unsized coercion)
    let array = [1, 2, 3, 4, 5];
    let boxed_from_array: Box<[i32]> = Box::new(array);
    // Note: This creates Box<[i32; 5]>, not Box<[i32]>

    // To get Box<[i32]> from array:
    let boxed_slice: Box<[i32]> = array.to_vec().into_boxed_slice();
    println!("From array: {:?}", boxed_slice);

    // Zeroed slice
    let zeroed: Box<[u8]> = vec![0u8; 100].into_boxed_slice();
    println!("Zeroed slice length: {}", zeroed.len());
}

/// Use case: Fixed-size buffer
pub struct FixedBuffer {
    data: Box<[u8]>,
    position: usize,
}

impl FixedBuffer {
    pub fn new(size: usize) -> Self {
        Self {
            data: vec![0u8; size].into_boxed_slice(),
            position: 0,
        }
    }

    pub fn write(&mut self, bytes: &[u8]) -> usize {
        let remaining = self.data.len() - self.position;
        let to_write = bytes.len().min(remaining);
        self.data[self.position..self.position + to_write]
            .copy_from_slice(&bytes[..to_write]);
        self.position += to_write;
        to_write
    }

    pub fn data(&self) -> &[u8] {
        &self.data[..self.position]
    }

    pub fn capacity(&self) -> usize {
        self.data.len()
    }
}

fn main() {
    demonstrate_boxed_slice();
    boxed_from_iterator();

    println!("\n=== Fixed Buffer Usage ===\n");
    let mut buf = FixedBuffer::new(20);
    buf.write(b"Hello, ");
    buf.write(b"World!");
    println!("Buffer content: {:?}", String::from_utf8_lossy(buf.data()));
    println!("Capacity: {}", buf.capacity());
}
```

---

## Partie 5: Trait Objects (2.1.6.q)

### Exercice 5.1: Box<dyn Trait>

```rust
//! Box<dyn Trait> - Trait objects (2.1.6.q)
//!
//! Trait objects enable dynamic dispatch through a vtable.
//! Box<dyn Trait> owns a trait object on the heap.

use std::fmt::Debug;

/// Base trait for shapes
pub trait Shape: Debug {
    fn area(&self) -> f64;
    fn perimeter(&self) -> f64;
    fn name(&self) -> &str;
}

#[derive(Debug)]
pub struct Circle {
    radius: f64,
}

impl Circle {
    pub fn new(radius: f64) -> Self {
        Self { radius }
    }
}

impl Shape for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }

    fn perimeter(&self) -> f64 {
        2.0 * std::f64::consts::PI * self.radius
    }

    fn name(&self) -> &str {
        "Circle"
    }
}

#[derive(Debug)]
pub struct Rectangle {
    width: f64,
    height: f64,
}

impl Rectangle {
    pub fn new(width: f64, height: f64) -> Self {
        Self { width, height }
    }
}

impl Shape for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }

    fn perimeter(&self) -> f64 {
        2.0 * (self.width + self.height)
    }

    fn name(&self) -> &str {
        "Rectangle"
    }
}

/// Demonstrate Box<dyn Trait> (2.1.6.q)
fn demonstrate_trait_objects() {
    println!("=== Box<dyn Trait> (2.1.6.q) ===\n");

    // Create trait objects
    let shapes: Vec<Box<dyn Shape>> = vec![
        Box::new(Circle::new(5.0)),
        Box::new(Rectangle::new(4.0, 6.0)),
        Box::new(Circle::new(3.0)),
    ];

    // Dynamic dispatch
    for shape in &shapes {
        println!("{}: area = {:.2}, perimeter = {:.2}",
            shape.name(),
            shape.area(),
            shape.perimeter()
        );
    }

    // Size of trait object pointer (fat pointer)
    println!("\nSize of Box<dyn Shape>: {} bytes",
        std::mem::size_of::<Box<dyn Shape>>());
    println!("Size of Box<Circle>: {} bytes",
        std::mem::size_of::<Box<Circle>>());
}

/// Factory function returning trait object
fn create_shape(kind: &str, params: &[f64]) -> Option<Box<dyn Shape>> {
    match kind {
        "circle" if params.len() >= 1 => {
            Some(Box::new(Circle::new(params[0])))
        }
        "rectangle" if params.len() >= 2 => {
            Some(Box::new(Rectangle::new(params[0], params[1])))
        }
        _ => None,
    }
}

/// Object-safe trait with associated functions
pub trait Drawable: Shape {
    fn draw(&self) -> String;
}

impl Drawable for Circle {
    fn draw(&self) -> String {
        format!("○ (r={})", self.radius)
    }
}

impl Drawable for Rectangle {
    fn draw(&self) -> String {
        format!("▭ ({}x{})", self.width, self.height)
    }
}

fn main() {
    demonstrate_trait_objects();

    println!("\n=== Factory Pattern ===\n");
    if let Some(shape) = create_shape("circle", &[7.0]) {
        println!("Created: {:?}, area = {:.2}", shape, shape.area());
    }

    println!("\n=== Drawable Trait Objects ===\n");
    let drawables: Vec<Box<dyn Drawable>> = vec![
        Box::new(Circle::new(3.0)),
        Box::new(Rectangle::new(5.0, 2.0)),
    ];

    for d in &drawables {
        println!("{} - {}", d.name(), d.draw());
    }
}
```

---

## Partie 6: Custom Allocators (2.1.6.r)

### Exercice 6.1: Box::new_in() with Custom Allocator

```rust
//! Custom allocators for Box (2.1.6.r)
//!
//! Using Box::new_in() with custom allocators.
//! Note: Requires nightly or allocator_api feature.

#![feature(allocator_api)]

use std::alloc::{Allocator, AllocError, Global, Layout};
use std::ptr::NonNull;

/// Tracking allocator that counts allocations (2.1.6.r)
pub struct TrackingAllocator {
    inner: Global,
    allocations: std::cell::Cell<usize>,
    total_bytes: std::cell::Cell<usize>,
}

impl TrackingAllocator {
    pub const fn new() -> Self {
        Self {
            inner: Global,
            allocations: std::cell::Cell::new(0),
            total_bytes: std::cell::Cell::new(0),
        }
    }

    pub fn allocations(&self) -> usize {
        self.allocations.get()
    }

    pub fn total_bytes(&self) -> usize {
        self.total_bytes.get()
    }
}

unsafe impl Allocator for TrackingAllocator {
    fn allocate(&self, layout: Layout) -> Result<NonNull<[u8]>, AllocError> {
        self.allocations.set(self.allocations.get() + 1);
        self.total_bytes.set(self.total_bytes.get() + layout.size());
        self.inner.allocate(layout)
    }

    unsafe fn deallocate(&self, ptr: NonNull<u8>, layout: Layout) {
        self.total_bytes.set(self.total_bytes.get().saturating_sub(layout.size()));
        self.inner.deallocate(ptr, layout)
    }
}

/// Arena allocator for batch allocations
pub struct ArenaAllocator {
    memory: Box<[u8]>,
    offset: std::cell::Cell<usize>,
}

impl ArenaAllocator {
    pub fn new(size: usize) -> Self {
        Self {
            memory: vec![0u8; size].into_boxed_slice(),
            offset: std::cell::Cell::new(0),
        }
    }

    pub fn used(&self) -> usize {
        self.offset.get()
    }

    pub fn remaining(&self) -> usize {
        self.memory.len() - self.offset.get()
    }

    pub fn reset(&self) {
        self.offset.set(0);
    }
}

unsafe impl Allocator for ArenaAllocator {
    fn allocate(&self, layout: Layout) -> Result<NonNull<[u8]>, AllocError> {
        let current = self.offset.get();
        let align_offset = current.next_multiple_of(layout.align()) - current;
        let start = current + align_offset;
        let end = start + layout.size();

        if end > self.memory.len() {
            return Err(AllocError);
        }

        self.offset.set(end);

        let ptr = unsafe {
            self.memory.as_ptr().add(start) as *mut u8
        };

        Ok(NonNull::slice_from_raw_parts(
            NonNull::new(ptr).unwrap(),
            layout.size(),
        ))
    }

    unsafe fn deallocate(&self, _ptr: NonNull<u8>, _layout: Layout) {
        // Arena allocator doesn't deallocate individually
    }
}

fn demonstrate_custom_allocator() {
    println!("=== Custom Allocators (2.1.6.r) ===\n");

    // Tracking allocator
    let tracking = TrackingAllocator::new();

    {
        // Box::new_in with custom allocator
        let boxed: Box<[i32; 100], &TrackingAllocator> =
            Box::new_in([0i32; 100], &tracking);

        println!("After allocation:");
        println!("  Allocations: {}", tracking.allocations());
        println!("  Total bytes: {}", tracking.total_bytes());

        // Use the boxed value
        println!("  Array length: {}", boxed.len());
    }

    println!("\nAfter deallocation:");
    println!("  Allocations: {}", tracking.allocations());
    println!("  Total bytes: {}", tracking.total_bytes());
}

fn demonstrate_arena() {
    println!("\n=== Arena Allocator ===\n");

    let arena = ArenaAllocator::new(1024);

    {
        let b1: Box<i32, &ArenaAllocator> = Box::new_in(42, &arena);
        let b2: Box<String, &ArenaAllocator> = Box::new_in("Hello".to_string(), &arena);
        let b3: Box<[u8; 100], &ArenaAllocator> = Box::new_in([0u8; 100], &arena);

        println!("Arena used: {} bytes", arena.used());
        println!("Arena remaining: {} bytes", arena.remaining());

        println!("Values: {}, {}, {} bytes", *b1, *b2, b3.len());
    }

    // All deallocations are no-ops for arena
    println!("After drops - used: {} bytes", arena.used());

    // Reset arena for reuse
    arena.reset();
    println!("After reset - used: {} bytes", arena.used());
}

fn main() {
    demonstrate_custom_allocator();
    demonstrate_arena();
}
```

---

## Partie 7: Pin<Box<T>> (2.1.6.s)

### Exercice 7.1: Self-Referential Structures

```rust
//! Pin<Box<T>> for self-referential structures (2.1.6.s)
//!
//! Pin guarantees that the pointee won't be moved,
//! which is essential for self-referential types.

use std::pin::Pin;
use std::marker::PhantomPinned;
use std::ptr::NonNull;

/// Self-referential structure (2.1.6.s)
/// Contains a pointer to its own field
pub struct SelfRef {
    value: String,
    // Points to `value` after initialization
    value_ptr: Option<NonNull<String>>,
    // Marker to make this type !Unpin
    _pin: PhantomPinned,
}

impl SelfRef {
    /// Create new unpinned instance
    pub fn new(value: &str) -> Self {
        Self {
            value: value.to_string(),
            value_ptr: None,
            _pin: PhantomPinned,
        }
    }

    /// Initialize the self-reference (requires pinning first)
    pub fn init(self: Pin<&mut Self>) {
        let self_ptr = unsafe {
            let this = self.get_unchecked_mut();
            NonNull::new(&mut this.value as *mut String)
        };
        unsafe {
            self.get_unchecked_mut().value_ptr = self_ptr;
        }
    }

    /// Access value through self-reference
    pub fn value_via_ptr(&self) -> Option<&str> {
        self.value_ptr.map(|ptr| unsafe { ptr.as_ref().as_str() })
    }

    /// Direct access to value
    pub fn value(&self) -> &str {
        &self.value
    }
}

fn demonstrate_pin_box() {
    println!("=== Pin<Box<T>> (2.1.6.s) ===\n");

    // Create pinned box
    let mut pinned: Pin<Box<SelfRef>> = Box::pin(SelfRef::new("Hello, Pin!"));

    // Initialize self-reference
    pinned.as_mut().init();

    // Access via both methods
    println!("Direct access: {}", pinned.value());
    println!("Via self-ref: {:?}", pinned.value_via_ptr());

    // The box can't be moved now (it's pinned)
    // This is safe because Pin guarantees the memory location stays fixed
}

/// More complex example: Intrusive linked list node
pub struct Node {
    value: i32,
    next: Option<NonNull<Node>>,
    prev: Option<NonNull<Node>>,
    _pin: PhantomPinned,
}

impl Node {
    pub fn new(value: i32) -> Self {
        Self {
            value,
            next: None,
            prev: None,
            _pin: PhantomPinned,
        }
    }

    /// Link two pinned nodes
    pub fn link(
        mut first: Pin<&mut Node>,
        mut second: Pin<&mut Node>,
    ) {
        unsafe {
            let first_ptr = NonNull::new(first.as_mut().get_unchecked_mut());
            let second_ptr = NonNull::new(second.as_mut().get_unchecked_mut());

            first.as_mut().get_unchecked_mut().next = second_ptr;
            second.as_mut().get_unchecked_mut().prev = first_ptr;
        }
    }
}

/// Async/Future-like structure that must be pinned
pub struct MyFuture {
    state: FutureState,
    _pin: PhantomPinned,
}

enum FutureState {
    NotStarted,
    Running { progress: usize },
    Completed(String),
}

impl MyFuture {
    pub fn new() -> Self {
        Self {
            state: FutureState::NotStarted,
            _pin: PhantomPinned,
        }
    }

    /// Poll the future (requires Pin)
    pub fn poll(self: Pin<&mut Self>) -> Option<String> {
        let this = unsafe { self.get_unchecked_mut() };

        match &mut this.state {
            FutureState::NotStarted => {
                this.state = FutureState::Running { progress: 0 };
                None
            }
            FutureState::Running { progress } => {
                *progress += 1;
                if *progress >= 3 {
                    this.state = FutureState::Completed("Done!".to_string());
                }
                None
            }
            FutureState::Completed(result) => {
                Some(result.clone())
            }
        }
    }
}

fn demonstrate_pinned_future() {
    println!("\n=== Pinned Future ===\n");

    let mut future: Pin<Box<MyFuture>> = Box::pin(MyFuture::new());

    loop {
        match future.as_mut().poll() {
            Some(result) => {
                println!("Future completed: {}", result);
                break;
            }
            None => {
                println!("Future pending...");
            }
        }
    }
}

fn main() {
    demonstrate_pin_box();
    demonstrate_pinned_future();
}
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| Box::into_raw/from_raw | 15 |
| Deref implementation | 10 |
| Pattern matching with Box | 15 |
| Box<[T]> operations | 15 |
| Box<dyn Trait> | 20 |
| Custom allocators | 15 |
| Pin<Box<T>> | 10 |
| **Total** | **100** |

---

## Ressources

- [Box documentation](https://doc.rust-lang.org/std/boxed/struct.Box.html)
- [Pin documentation](https://doc.rust-lang.org/std/pin/struct.Pin.html)
- [Allocator API](https://doc.rust-lang.org/std/alloc/trait.Allocator.html)
- [Deref coercion](https://doc.rust-lang.org/book/ch15-02-deref.html)
