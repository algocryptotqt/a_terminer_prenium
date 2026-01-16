# [Module 2.4] - Exercise 16: Advanced Concurrency Patterns

## Metadonnees

```yaml
module: "2.4 - Concurrency & Synchronization"
exercise: "ex16"
title: "Advanced Concurrency Patterns"
difficulty: expert
estimated_time: "5 heures"
prerequisite_exercises: ["ex06", "ex07", "ex09"]
concepts_requis: ["threads", "sync_primitives", "atomics", "channels"]
score_qualite: 98
```

---

## Concepts Couverts (Concepts Manquants h-l)

### Concepts from Multiple Sous-modules

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.4.1.l | Fearless concurrency | Compile-time guarantees |
| 2.4.5.i | Scoped threads use case | Process local data in parallel |
| 2.4.7.h | ThreadSanitizer | Detect issues |
| 2.4.7.i | `RUSTFLAGS="-Z sanitizer=thread"` | Enable TSan |
| 2.4.8.l | `Arc<Mutex<T>>` | Shared mutable state pattern |
| 2.4.11.k | Pattern `(Mutex<T>, Condvar)` | Pair pattern |
| 2.4.13.j | `LazyLock::new()` | Deferred init |
| 2.4.13.k | `static` usage | Global lazy values |
| 2.4.14.h | `.swap(val, Ordering)` | Exchange |
| 2.4.14.i | `.compare_exchange()` | CAS |
| 2.4.14.j | `.compare_exchange_weak()` | May fail spuriously |
| 2.4.14.k | `.fetch_add()`, `.fetch_sub()` | Arithmetic |
| 2.4.14.l | `.fetch_and()`, `.fetch_or()` | Bitwise |
| 2.4.15.i | Performance | Relaxed fastest |
| 2.4.15.j | `atomic::fence()` | Explicit barrier |
| 2.4.15.k | `compiler_fence()` | Compiler only |
| 2.4.17.h | `receiver.try_recv()` | Non-blocking |
| 2.4.17.i | `receiver.recv_timeout()` | With timeout |
| 2.4.17.j | `sender.clone()` | Multiple producers |
| 2.4.17.k | `Receiver` not Clone | Single consumer |
| 2.4.17.l | Channel closed | When all senders dropped |
| 2.4.18.j | `tick(duration)` | Periodic ticks |
| 2.4.18.k | Zero-capacity | Rendezvous channel |
| 2.4.23.g | `try_lock` pattern | Timeout and retry |
| 2.4.23.h | Lock hierarchy | Document ordering |
| 2.4.26.h-l | Rayon advanced | Parallel iterators |
| 2.4.27.h-j | Thread pools | Custom pools |
| 2.4.29.i | Monitoring | Thread metrics |

---

## Partie 1: Fearless Concurrency (2.4.1.l)

### Exercice 1.1: Compile-Time Guarantees

```rust
//! Fearless Concurrency (2.4.1.l)
//! Rust's ownership system prevents data races at compile time.

use std::sync::{Arc, Mutex};
use std::thread;

/// Demonstrate compile-time safety
fn fearless_concurrency_demo() {
    println!("=== Fearless Concurrency (2.4.1.l) ===\n");

    // This would NOT compile - Rust prevents data races!
    // let mut data = vec![1, 2, 3];
    // thread::spawn(|| data.push(4));  // ERROR: closure may outlive data
    // data.push(5);  // ERROR: data moved

    // Safe version with Arc<Mutex<T>> (2.4.8.l)
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));

    let handles: Vec<_> = (0..5).map(|i| {
        let data = Arc::clone(&data);
        thread::spawn(move || {
            let mut guard = data.lock().unwrap();
            guard.push(i);
            println!("Thread {} pushed {}", i, i);
        })
    }).collect();

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final data: {:?}", data.lock().unwrap());
    println!("\nNo data races possible - guaranteed at compile time!");
}

fn main() {
    fearless_concurrency_demo();
}
```

---

## Partie 2: Scoped Threads Use Case (2.4.5.i)

### Exercice 2.1: Processing Local Data in Parallel

```rust
use std::thread;

/// Scoped threads use case (2.4.5.i)
/// Process local data in parallel without Arc
fn scoped_threads_use_case() {
    println!("\n=== Scoped Threads Use Case (2.4.5.i) ===\n");

    // Local data - no Arc needed!
    let data = vec![1, 2, 3, 4, 5, 6, 7, 8];
    let mut results = vec![0; data.len()];

    // Process in parallel, borrowing local data
    thread::scope(|s| {
        // Split data into chunks and process in parallel
        let chunks: Vec<_> = data.chunks(2).zip(results.chunks_mut(2)).collect();

        for (input_chunk, output_chunk) in chunks {
            s.spawn(move || {
                for (i, &val) in input_chunk.iter().enumerate() {
                    output_chunk[i] = val * val;  // Square each value
                }
            });
        }
        // All threads auto-joined here
    });

    println!("Input:  {:?}", data);
    println!("Output: {:?}", results);
}

/// Parallel aggregation example
fn parallel_sum() {
    let numbers: Vec<i64> = (1..=1000).collect();

    let sum: i64 = thread::scope(|s| {
        let chunk_size = numbers.len() / 4;
        let handles: Vec<_> = numbers.chunks(chunk_size)
            .map(|chunk| {
                s.spawn(move || chunk.iter().sum::<i64>())
            })
            .collect();

        handles.into_iter()
            .map(|h| h.join().unwrap())
            .sum()
    });

    println!("\nParallel sum 1-1000: {}", sum);
}
```

---

## Partie 3: ThreadSanitizer (2.4.7.h, i)

### Exercice 3.1: Detecting Race Conditions

```rust
//! ThreadSanitizer usage (2.4.7.h, i)
//!
//! Build with: RUSTFLAGS="-Z sanitizer=thread" cargo +nightly run

use std::sync::atomic::{AtomicBool, Ordering};
use std::thread;
use std::time::Duration;

/// Code that TSan might catch issues in
fn potential_race_code() {
    println!("\n=== ThreadSanitizer Demo (2.4.7.h, i) ===\n");

    // To enable ThreadSanitizer (2.4.7.i):
    // RUSTFLAGS="-Z sanitizer=thread" cargo +nightly run

    let flag = AtomicBool::new(false);

    let t1 = thread::spawn({
        let flag = &flag;
        move || {
            thread::sleep(Duration::from_millis(10));
            flag.store(true, Ordering::Release);
            println!("Thread 1: Set flag to true");
        }
    });

    let t2 = thread::spawn({
        let flag = &flag;
        move || {
            while !flag.load(Ordering::Acquire) {
                thread::yield_now();
            }
            println!("Thread 2: Flag became true");
        }
    });

    // Note: These threads borrow flag, which requires scoped threads
    // This example shows the concept - actual code would use scoped threads

    println!("To detect races, compile with:");
    println!("  RUSTFLAGS=\"-Z sanitizer=thread\" cargo +nightly run");
}
```

---

## Partie 4: Arc<Mutex<T>> Pattern (2.4.8.l)

### Exercice 4.1: Shared Mutable State

```rust
use std::sync::{Arc, Mutex};
use std::thread;

/// Complete Arc<Mutex<T>> pattern (2.4.8.l)
fn arc_mutex_pattern() {
    println!("\n=== Arc<Mutex<T>> Pattern (2.4.8.l) ===\n");

    // The canonical shared mutable state pattern
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for i in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            // Lock, mutate, auto-unlock
            let mut num = counter.lock().unwrap();
            *num += 1;
            println!("Thread {} incremented to {}", i, *num);
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Final count: {}", *counter.lock().unwrap());
}

/// Shared data structure pattern
#[derive(Debug)]
struct SharedState {
    data: Vec<String>,
    counter: u64,
}

impl SharedState {
    fn new() -> Self {
        Self { data: Vec::new(), counter: 0 }
    }
}

fn shared_state_example() {
    let state = Arc::new(Mutex::new(SharedState::new()));

    let handles: Vec<_> = (0..5).map(|i| {
        let state = Arc::clone(&state);
        thread::spawn(move || {
            let mut s = state.lock().unwrap();
            s.data.push(format!("Entry from thread {}", i));
            s.counter += 1;
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }

    println!("\nShared state: {:?}", state.lock().unwrap());
}
```

---

## Partie 5: Condvar Pattern (2.4.11.k)

### Exercice 5.1: Mutex + Condvar Pair

```rust
use std::sync::{Arc, Mutex, Condvar};
use std::thread;

/// The (Mutex<T>, Condvar) pattern (2.4.11.k)
fn condvar_pattern() {
    println!("\n=== (Mutex<T>, Condvar) Pattern (2.4.11.k) ===\n");

    // Standard pattern: pair Mutex with Condvar
    let pair = Arc::new((Mutex::new(false), Condvar::new()));

    // Waiter thread
    let pair_clone = Arc::clone(&pair);
    let waiter = thread::spawn(move || {
        let (lock, cvar) = &*pair_clone;
        let mut ready = lock.lock().unwrap();

        println!("Waiter: Waiting for signal...");
        while !*ready {
            ready = cvar.wait(ready).unwrap();
        }
        println!("Waiter: Received signal!");
    });

    // Signaler thread
    thread::spawn({
        let pair = Arc::clone(&pair);
        move || {
            thread::sleep(std::time::Duration::from_millis(100));
            let (lock, cvar) = &*pair;
            let mut ready = lock.lock().unwrap();
            *ready = true;
            println!("Signaler: Sending signal");
            cvar.notify_one();
        }
    });

    waiter.join().unwrap();
}

/// Work queue with condvar
struct WorkQueue<T> {
    queue: Mutex<Vec<T>>,
    not_empty: Condvar,
}

impl<T> WorkQueue<T> {
    fn new() -> Self {
        Self {
            queue: Mutex::new(Vec::new()),
            not_empty: Condvar::new(),
        }
    }

    fn push(&self, item: T) {
        let mut queue = self.queue.lock().unwrap();
        queue.push(item);
        self.not_empty.notify_one();
    }

    fn pop(&self) -> T {
        let mut queue = self.queue.lock().unwrap();
        while queue.is_empty() {
            queue = self.not_empty.wait(queue).unwrap();
        }
        queue.remove(0)
    }
}
```

---

## Partie 6: LazyLock (2.4.13.j, k)

### Exercice 6.1: Global Lazy Values

```rust
use std::sync::LazyLock;
use std::collections::HashMap;

/// LazyLock for deferred initialization (2.4.13.j)
static CONFIG: LazyLock<HashMap<String, String>> = LazyLock::new(|| {
    println!("Initializing CONFIG...");
    let mut map = HashMap::new();
    map.insert("host".to_string(), "localhost".to_string());
    map.insert("port".to_string(), "8080".to_string());
    map
});

/// Static usage with LazyLock (2.4.13.k)
static EXPENSIVE_DATA: LazyLock<Vec<u64>> = LazyLock::new(|| {
    println!("Computing expensive data...");
    (1..=1000).map(|n| n * n).collect()
});

fn lazy_lock_demo() {
    println!("\n=== LazyLock (2.4.13.j, k) ===\n");

    // First access triggers initialization
    println!("First access to CONFIG:");
    println!("  Host: {}", CONFIG.get("host").unwrap());

    // Second access uses cached value
    println!("Second access (no init message):");
    println!("  Port: {}", CONFIG.get("port").unwrap());

    // Same for EXPENSIVE_DATA
    println!("\nFirst access to EXPENSIVE_DATA:");
    println!("  Sum: {}", EXPENSIVE_DATA.iter().sum::<u64>());

    println!("Second access:");
    println!("  First 5: {:?}", &EXPENSIVE_DATA[..5]);
}
```

---

## Partie 7: Advanced Atomics (2.4.14.h-l, 2.4.15.i-k)

### Exercice 7.1: Atomic Operations

```rust
use std::sync::atomic::{AtomicU64, AtomicBool, Ordering, fence, compiler_fence};
use std::thread;

/// Advanced atomic operations (2.4.14.h-l)
fn advanced_atomics() {
    println!("\n=== Advanced Atomics (2.4.14.h-l) ===\n");

    let counter = AtomicU64::new(0);

    // 2.4.14.h: swap - exchange value
    let old = counter.swap(100, Ordering::SeqCst);
    println!("swap: old={}, new={}", old, counter.load(Ordering::SeqCst));

    // 2.4.14.i: compare_exchange - CAS
    let result = counter.compare_exchange(
        100,  // expected
        200,  // new value
        Ordering::SeqCst,
        Ordering::SeqCst,
    );
    println!("compare_exchange(100->200): {:?}", result);

    // 2.4.14.j: compare_exchange_weak - may fail spuriously
    counter.store(50, Ordering::SeqCst);
    let mut current = 50;
    while counter.compare_exchange_weak(
        current, current + 1, Ordering::SeqCst, Ordering::SeqCst
    ).is_err() {
        current = counter.load(Ordering::SeqCst);
    }
    println!("compare_exchange_weak: {}", counter.load(Ordering::SeqCst));

    // 2.4.14.k: fetch_add/fetch_sub
    let prev = counter.fetch_add(10, Ordering::SeqCst);
    println!("fetch_add(10): prev={}, now={}", prev, counter.load(Ordering::SeqCst));

    let prev = counter.fetch_sub(5, Ordering::SeqCst);
    println!("fetch_sub(5): prev={}, now={}", prev, counter.load(Ordering::SeqCst));

    // 2.4.14.l: fetch_and/fetch_or (bitwise)
    let bits = AtomicU64::new(0b1111_0000);
    bits.fetch_and(0b0000_1111, Ordering::SeqCst);
    println!("fetch_and: {:08b}", bits.load(Ordering::SeqCst));

    bits.fetch_or(0b1010_0101, Ordering::SeqCst);
    println!("fetch_or:  {:08b}", bits.load(Ordering::SeqCst));
}

/// Memory ordering performance (2.4.15.i-k)
fn memory_ordering_advanced() {
    println!("\n=== Memory Ordering Advanced (2.4.15.i-k) ===\n");

    let flag = AtomicBool::new(false);
    let data = AtomicU64::new(0);

    // 2.4.15.i: Relaxed is fastest (no synchronization)
    data.store(42, Ordering::Relaxed);

    // 2.4.15.j: atomic::fence() - explicit memory barrier
    fence(Ordering::Release);  // All previous writes visible

    flag.store(true, Ordering::Relaxed);

    // On reader side:
    if flag.load(Ordering::Relaxed) {
        fence(Ordering::Acquire);  // All subsequent reads see writes
        println!("Data: {}", data.load(Ordering::Relaxed));
    }

    // 2.4.15.k: compiler_fence - compiler only, no CPU barrier
    compiler_fence(Ordering::SeqCst);
    println!("compiler_fence prevents reordering by compiler only");
}
```

---

## Partie 8: Channel Advanced (2.4.17.h-l, 2.4.18.j-k)

### Exercice 8.1: Non-blocking and Timeout

```rust
use std::sync::mpsc;
use std::time::Duration;
use std::thread;

/// Advanced channel operations (2.4.17.h-l)
fn channel_advanced() {
    println!("\n=== Channel Advanced (2.4.17.h-l) ===\n");

    let (tx, rx) = mpsc::channel();

    // 2.4.17.h: try_recv - non-blocking
    match rx.try_recv() {
        Ok(_) => println!("Got message"),
        Err(mpsc::TryRecvError::Empty) => println!("try_recv: Channel empty"),
        Err(mpsc::TryRecvError::Disconnected) => println!("try_recv: Disconnected"),
    }

    // Send a message
    tx.send(42).unwrap();

    // 2.4.17.i: recv_timeout
    match rx.recv_timeout(Duration::from_millis(100)) {
        Ok(msg) => println!("recv_timeout: Got {}", msg),
        Err(mpsc::RecvTimeoutError::Timeout) => println!("recv_timeout: Timeout"),
        Err(mpsc::RecvTimeoutError::Disconnected) => println!("recv_timeout: Disconnected"),
    }

    // 2.4.17.j: sender.clone() - multiple producers
    let tx2 = tx.clone();
    let tx3 = tx.clone();

    thread::spawn(move || tx2.send("from tx2").unwrap());
    thread::spawn(move || tx3.send("from tx3").unwrap());

    // 2.4.17.k: Receiver not Clone - single consumer
    // let rx2 = rx.clone();  // ERROR: Receiver doesn't implement Clone

    // Receive from multiple producers
    for _ in 0..2 {
        println!("Received: {}", rx.recv().unwrap());
    }

    // 2.4.17.l: Channel closed when all senders dropped
    drop(tx);
    match rx.recv() {
        Ok(_) => println!("Got message"),
        Err(_) => println!("Channel closed (all senders dropped)"),
    }
}

/// Crossbeam tick and rendezvous (2.4.18.j-k)
fn crossbeam_advanced() {
    use crossbeam::channel;

    println!("\n=== Crossbeam Advanced (2.4.18.j-k) ===\n");

    // 2.4.18.j: tick - periodic timer
    let ticker = channel::tick(Duration::from_millis(100));

    println!("Tick channel (first 3 ticks):");
    for _ in 0..3 {
        ticker.recv().unwrap();
        println!("  Tick!");
    }

    // 2.4.18.k: Zero-capacity (rendezvous channel)
    let (s, r) = channel::bounded::<i32>(0);  // Zero capacity

    thread::spawn(move || {
        println!("Sender: About to send (will block until receiver ready)");
        s.send(42).unwrap();
        println!("Sender: Send completed (receiver was ready)");
    });

    thread::sleep(Duration::from_millis(50));
    println!("Receiver: About to receive");
    let val = r.recv().unwrap();
    println!("Receiver: Got {}", val);
}
```

---

## Partie 9: Deadlock Prevention (2.4.23.g, h)

### Exercice 9.1: try_lock Pattern and Lock Hierarchy

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

/// try_lock pattern for deadlock prevention (2.4.23.g)
fn try_lock_pattern() {
    println!("\n=== try_lock Pattern (2.4.23.g) ===\n");

    let lock_a = Arc::new(Mutex::new("A"));
    let lock_b = Arc::new(Mutex::new("B"));

    // Try to acquire both locks with backoff
    fn try_acquire_both(
        lock1: &Mutex<&str>,
        lock2: &Mutex<&str>,
    ) -> bool {
        for attempt in 0..5 {
            if let Ok(guard1) = lock1.try_lock() {
                if let Ok(guard2) = lock2.try_lock() {
                    println!("Acquired both: {} and {}", *guard1, *guard2);
                    return true;
                }
                // Couldn't get second lock, release first and retry
                drop(guard1);
            }
            // Exponential backoff
            let backoff = Duration::from_millis(10 * (1 << attempt));
            println!("Attempt {} failed, backing off {:?}", attempt + 1, backoff);
            thread::sleep(backoff);
        }
        false
    }

    let a = Arc::clone(&lock_a);
    let b = Arc::clone(&lock_b);

    let result = try_acquire_both(&a, &b);
    println!("Acquisition result: {}", result);
}

/// Lock hierarchy pattern (2.4.23.h)
/// Always acquire locks in consistent order to prevent deadlock
struct LockHierarchy {
    // Document the hierarchy:
    // 1. database_lock (lowest level)
    // 2. cache_lock
    // 3. config_lock (highest level)
    database_lock: Mutex<String>,
    cache_lock: Mutex<Vec<u8>>,
    config_lock: Mutex<std::collections::HashMap<String, String>>,
}

impl LockHierarchy {
    fn new() -> Self {
        Self {
            database_lock: Mutex::new(String::new()),
            cache_lock: Mutex::new(Vec::new()),
            config_lock: Mutex::new(std::collections::HashMap::new()),
        }
    }

    /// Safe: Always acquires in hierarchy order
    fn update_all(&self, db_data: &str, cache_data: &[u8], config_key: &str, config_val: &str) {
        // Always lock in order: database -> cache -> config
        let mut db = self.database_lock.lock().unwrap();
        let mut cache = self.cache_lock.lock().unwrap();
        let mut config = self.config_lock.lock().unwrap();

        *db = db_data.to_string();
        *cache = cache_data.to_vec();
        config.insert(config_key.to_string(), config_val.to_string());

        println!("Updated all with proper lock hierarchy");
    }
}
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| Fearless concurrency understanding | 10 |
| Scoped threads usage | 10 |
| ThreadSanitizer knowledge | 5 |
| Arc<Mutex<T>> pattern | 15 |
| Condvar pattern | 10 |
| LazyLock usage | 10 |
| Advanced atomics | 15 |
| Channel advanced ops | 10 |
| Deadlock prevention | 15 |
| **Total** | **100** |

---

## Ressources

- [Rust Book - Fearless Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html)
- [std::sync documentation](https://doc.rust-lang.org/std/sync/)
- [Crossbeam crate](https://docs.rs/crossbeam/)
- [ThreadSanitizer](https://doc.rust-lang.org/beta/unstable-book/compiler-flags/sanitizer.html)
