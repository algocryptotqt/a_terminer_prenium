# [Module 2.4] - Exercise 17: Threading Fundamentals Complete

## Metadonnees

```yaml
module: "2.4 - Concurrency & Synchronization"
exercise: "ex17"
title: "Threading Fundamentals Complete"
difficulty: avance
estimated_time: "6 heures"
prerequisite_exercises: ["ex00", "ex01", "ex02"]
concepts_requis: ["threads", "mutex", "condvar", "atomics"]
score_qualite: 98
```

---

## Concepts Couverts (Missing Fundamental Concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.4.2.a | `std::thread::spawn()` | Create thread |
| 2.4.2.b | Closure argument | `FnOnce + Send + 'static` |
| 2.4.2.c | `JoinHandle<T>` | Handle to thread |
| 2.4.2.e | `Result<T, E>` | Thread result or panic |
| 2.4.2.f | `thread::current()` | Current thread |
| 2.4.4.a | `move` keyword | Transfer ownership |
| 2.4.4.h | Solution | Clone or scoped threads |
| 2.4.5.h | `crossbeam::scope` | Alternative scoped threads |
| 2.4.6.a | TLS concept | Per-thread data |
| 2.4.6.b | `thread_local!` macro | Declare TLS |
| 2.4.6.c | `LocalKey<T>` | TLS key type |
| 2.4.6.j | Performance | Faster than locks |
| 2.4.7.d | Rust doesn't prevent | Race conditions (logic bugs) |
| 2.4.7.e | `Send` / `Sync` | Compile-time enforcement |
| 2.4.8.a | `std::sync::Mutex<T>` | Mutual exclusion wrapper |
| 2.4.8.b | `Mutex::new(data)` | Create mutex |
| 2.4.8.k | `.into_inner()` | Recover poisoned data |
| 2.4.9.e | Lock ordering | Prevent deadlock |
| 2.4.9.f | Coarse vs fine | Granularity tradeoff |
| 2.4.10.d | `RwLockReadGuard<T>` | Read guard |
| 2.4.10.e | `rwlock.write()` | Acquire write lock |
| 2.4.10.f | `RwLockWriteGuard<T>` | Write guard |
| 2.4.10.g | Multiple readers | Concurrent reads OK |
| 2.4.10.h | Exclusive writer | One writer, no readers |
| 2.4.11.h | `condvar.wait_while()` | Loop built-in |
| 2.4.11.i | `wait_timeout()` | With timeout |
| 2.4.11.j | `wait_timeout_while()` | Timeout + condition |
| 2.4.12.c | `barrier.wait()` | Wait for all |
| 2.4.12.d | `BarrierWaitResult` | Result of wait |
| 2.4.12.e | `.is_leader()` | One thread is leader |
| 2.4.12.g | Use case | Parallel phases |
| 2.4.15.a | Memory ordering | Orderings for atomics |
| 2.4.16.a | Atomic patterns | Common atomic patterns |
| 2.4.17.a | `std::sync::mpsc` | Channel module |
| 2.4.17.c | `mpsc::channel()` | Create unbounded |
| 2.4.26.f | `par_iter()` | Parallel iterator |
| 2.4.28.a | Thread pool concept | Pool of worker threads |

---

## Partie 1: Thread Basics (2.4.2)

### Exercice 1.1: Thread Creation and Joining

```rust
//! std::thread basics (2.4.2.a-f)

use std::thread;
use std::time::Duration;

/// Thread creation (2.4.2.a)
fn spawn_thread_demo() {
    println!("=== Thread Spawn (2.4.2.a) ===\n");

    // 2.4.2.a: Create thread with spawn
    let handle = thread::spawn(|| {
        println!("Hello from spawned thread!");
        42  // Return value
    });

    // 2.4.2.c: JoinHandle<T>
    println!("Handle type: JoinHandle<i32>");

    // 2.4.2.e: Result<T, E> from join
    let result = handle.join();
    match result {
        Ok(value) => println!("Thread returned: {}", value),
        Err(e) => println!("Thread panicked: {:?}", e),
    }
}

/// Closure requirements (2.4.2.b)
fn closure_requirements() {
    println!("\n=== Closure Requirements (2.4.2.b) ===\n");

    // 2.4.2.b: Closure must be FnOnce + Send + 'static
    let data = String::from("Hello");

    // This closure takes ownership (FnOnce)
    // String is Send, so can transfer to thread
    // 'static: thread may outlive current scope
    let handle = thread::spawn(move || {
        println!("Got data: {}", data);
        // data is moved into closure, not borrowed
    });

    handle.join().unwrap();
}

/// Current thread info (2.4.2.f)
fn current_thread_demo() {
    println!("\n=== Current Thread (2.4.2.f) ===\n");

    // 2.4.2.f: Get current thread info
    let current = thread::current();
    println!("Current thread name: {:?}", current.name());
    println!("Current thread id: {:?}", current.id());

    // Named thread
    thread::Builder::new()
        .name("worker-1".to_string())
        .spawn(|| {
            let me = thread::current();
            println!("Worker thread name: {:?}", me.name());
        })
        .unwrap()
        .join()
        .unwrap();
}
```

---

## Partie 2: Move Closures (2.4.4)

### Exercice 2.1: Ownership Transfer

```rust
//! Move closures and ownership (2.4.4.a, 2.4.4.h)

use std::sync::Arc;
use std::thread;

/// Move keyword usage (2.4.4.a)
fn move_keyword_demo() {
    println!("\n=== Move Keyword (2.4.4.a) ===\n");

    // 2.4.4.a: move transfers ownership to closure
    let numbers = vec![1, 2, 3, 4, 5];

    // Without move - would fail to compile
    // thread::spawn(|| println!("{:?}", numbers));  // ERROR!

    // With move - ownership transferred
    let handle = thread::spawn(move || {
        println!("Numbers: {:?}", numbers);
        numbers.iter().sum::<i32>()
    });

    // numbers is no longer accessible here
    // println!("{:?}", numbers);  // ERROR: moved

    let sum = handle.join().unwrap();
    println!("Sum: {}", sum);
}

/// Solutions for sharing data (2.4.4.h)
fn sharing_solutions() {
    println!("\n=== Sharing Solutions (2.4.4.h) ===\n");

    // Solution 1: Clone before move
    let data = Arc::new(vec![1, 2, 3]);
    let mut handles = vec![];

    for i in 0..3 {
        let data_clone = Arc::clone(&data);  // Clone Arc, not data
        handles.push(thread::spawn(move || {
            println!("Thread {}: data[0] = {}", i, data_clone[0]);
        }));
    }

    for h in handles {
        h.join().unwrap();
    }

    // Solution 2: Scoped threads (see Part 3)
    println!("\nAlternative: Use scoped threads to borrow directly");
}
```

---

## Partie 3: Scoped Threads (2.4.5.h)

### Exercice 3.1: Crossbeam Scope

```rust
//! Scoped threads with crossbeam (2.4.5.h)

use crossbeam::thread as cb_thread;

/// Crossbeam scoped threads (2.4.5.h)
fn crossbeam_scope_demo() {
    println!("\n=== Crossbeam Scope (2.4.5.h) ===\n");

    // Local data - can borrow without Arc
    let data = vec![1, 2, 3, 4, 5];
    let mut results = vec![];

    // 2.4.5.h: crossbeam::scope alternative
    cb_thread::scope(|s| {
        // Can borrow data directly - no 'static needed
        for i in 0..3 {
            s.spawn(|_| {
                let sum: i32 = data.iter().map(|x| x * (i + 1)).sum();
                println!("Thread {}: sum = {}", i, sum);
            });
        }

        // Can also collect results
        let handle = s.spawn(|_| {
            data.iter().sum::<i32>()
        });
        results.push(handle.join().unwrap());
    }).unwrap();

    // Threads automatically joined at scope end
    println!("Results collected: {:?}", results);
    println!("Data still accessible: {:?}", data);
}
```

---

## Partie 4: Thread-Local Storage (2.4.6)

### Exercice 4.1: TLS Basics

```rust
//! Thread-local storage (2.4.6.a-c, 2.4.6.j)

use std::cell::RefCell;
use std::thread;

// 2.4.6.b: Declare thread-local with macro
thread_local! {
    // 2.4.6.c: LocalKey<T> is the key type
    static COUNTER: RefCell<u32> = RefCell::new(0);
    static THREAD_NAME: RefCell<String> = RefCell::new(String::new());
}

/// TLS concept and usage (2.4.6.a, 2.4.6.j)
fn tls_demo() {
    println!("\n=== Thread-Local Storage (2.4.6.a) ===\n");

    // 2.4.6.a: Per-thread data - each thread has its own copy

    let mut handles = vec![];

    for i in 0..3 {
        handles.push(thread::spawn(move || {
            // Set thread-specific name
            THREAD_NAME.with(|name| {
                *name.borrow_mut() = format!("Worker-{}", i);
            });

            // Increment counter (thread-local!)
            for _ in 0..100 {
                COUNTER.with(|c| {
                    *c.borrow_mut() += 1;
                });
            }

            // Each thread sees only its own counter
            COUNTER.with(|c| {
                let name = THREAD_NAME.with(|n| n.borrow().clone());
                println!("{}: counter = {}", name, c.borrow());
            });
        }));
    }

    for h in handles {
        h.join().unwrap();
    }

    // 2.4.6.j: Performance - no locking needed!
    println!("\nTLS is faster than Mutex - no synchronization overhead");
}
```

---

## Partie 5: Data Races vs Race Conditions (2.4.7)

### Exercice 5.1: Understanding the Difference

```rust
//! Data races vs race conditions (2.4.7.d-e)

use std::sync::{Arc, Mutex};
use std::thread;

/// Race conditions (2.4.7.d)
fn race_condition_demo() {
    println!("\n=== Race Conditions (2.4.7.d) ===\n");

    // 2.4.7.d: Rust prevents DATA races, not RACE CONDITIONS

    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    // This is a RACE CONDITION (logic depends on timing)
    // But NOT a DATA RACE (Mutex provides synchronization)
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            // Check-then-act is a race condition!
            if *num < 5 {
                thread::yield_now();  // Timing dependency
                *num += 1;
            }
        }));
    }

    for h in handles {
        h.join().unwrap();
    }

    println!("Final counter: {} (order not guaranteed!)", *counter.lock().unwrap());
}

/// Send and Sync enforcement (2.4.7.e)
fn send_sync_demo() {
    println!("\n=== Send/Sync Enforcement (2.4.7.e) ===\n");

    // 2.4.7.e: Compiler enforces thread safety

    // Arc<T> is Send + Sync when T is Send + Sync
    let safe = Arc::new(Mutex::new(vec![1, 2, 3]));

    // Can send to other thread
    let safe_clone = Arc::clone(&safe);
    thread::spawn(move || {
        safe_clone.lock().unwrap().push(4);
    }).join().unwrap();

    // Rc<T> is NOT Send - can't use across threads
    // use std::rc::Rc;
    // let not_safe = Rc::new(42);
    // thread::spawn(move || { println!("{}", not_safe); });  // COMPILE ERROR!

    println!("Compiler enforces Send/Sync at compile time");
}
```

---

## Partie 6: Mutex Fundamentals (2.4.8, 2.4.9)

### Exercice 6.1: Mutex Basics

```rust
//! Mutex basics (2.4.8.a-b, 2.4.8.k, 2.4.9.e-f)

use std::sync::{Arc, Mutex, PoisonError};
use std::thread;

/// Mutex creation and usage (2.4.8.a-b)
fn mutex_basics() {
    println!("\n=== Mutex Basics (2.4.8.a-b) ===\n");

    // 2.4.8.a: Mutex<T> - mutual exclusion wrapper
    // 2.4.8.b: Create with Mutex::new(data)
    let mutex = Arc::new(Mutex::new(0));

    let mut handles = vec![];

    for _ in 0..10 {
        let mutex = Arc::clone(&mutex);
        handles.push(thread::spawn(move || {
            let mut guard = mutex.lock().unwrap();
            *guard += 1;
        }));
    }

    for h in handles {
        h.join().unwrap();
    }

    println!("Final value: {}", *mutex.lock().unwrap());
}

/// Recover from poisoned mutex (2.4.8.k)
fn poison_recovery() {
    println!("\n=== Poison Recovery (2.4.8.k) ===\n");

    let mutex = Arc::new(Mutex::new(vec![1, 2, 3]));
    let mutex_clone = Arc::clone(&mutex);

    // Panic while holding lock
    let _ = thread::spawn(move || {
        let _guard = mutex_clone.lock().unwrap();
        panic!("Oops!");
    }).join();

    // 2.4.8.k: Recover data with .into_inner()
    match mutex.lock() {
        Ok(guard) => println!("Data: {:?}", *guard),
        Err(poisoned) => {
            println!("Mutex was poisoned, recovering...");
            let recovered = poisoned.into_inner();
            println!("Recovered data: {:?}", *recovered);
        }
    }
}

/// Lock ordering (2.4.9.e)
fn lock_ordering() {
    println!("\n=== Lock Ordering (2.4.9.e) ===\n");

    // 2.4.9.e: Always acquire locks in consistent order
    let lock_a = Arc::new(Mutex::new("A"));
    let lock_b = Arc::new(Mutex::new("B"));

    // Good: Always acquire A before B
    let a1 = Arc::clone(&lock_a);
    let b1 = Arc::clone(&lock_b);
    let t1 = thread::spawn(move || {
        let _a = a1.lock().unwrap();
        let _b = b1.lock().unwrap();
        println!("Thread 1: Got A then B");
    });

    let a2 = Arc::clone(&lock_a);
    let b2 = Arc::clone(&lock_b);
    let t2 = thread::spawn(move || {
        let _a = a2.lock().unwrap();  // Same order!
        let _b = b2.lock().unwrap();
        println!("Thread 2: Got A then B");
    });

    t1.join().unwrap();
    t2.join().unwrap();

    println!("No deadlock - consistent ordering!");
}

/// Lock granularity (2.4.9.f)
fn lock_granularity() {
    println!("\n=== Lock Granularity (2.4.9.f) ===\n");

    // 2.4.9.f: Coarse vs fine-grained locking

    // Coarse: One lock for everything (simple, less concurrency)
    let coarse = Arc::new(Mutex::new((0, 0, 0)));

    // Fine: Separate locks (complex, more concurrency)
    let fine_a = Arc::new(Mutex::new(0));
    let fine_b = Arc::new(Mutex::new(0));
    let fine_c = Arc::new(Mutex::new(0));

    println!("Coarse: Simple but blocks all fields");
    println!("Fine: Complex but allows concurrent access to different fields");
}
```

---

## Partie 7: RwLock (2.4.10)

### Exercice 7.1: Reader-Writer Lock

```rust
//! RwLock (2.4.10.d-h)

use std::sync::{Arc, RwLock};
use std::thread;

/// RwLock usage (2.4.10.d-h)
fn rwlock_demo() {
    println!("\n=== RwLock (2.4.10.d-h) ===\n");

    let data = Arc::new(RwLock::new(vec![1, 2, 3]));
    let mut handles = vec![];

    // 2.4.10.g: Multiple readers can access concurrently
    for i in 0..3 {
        let data = Arc::clone(&data);
        handles.push(thread::spawn(move || {
            // 2.4.10.d: RwLockReadGuard
            let read_guard = data.read().unwrap();
            println!("Reader {}: {:?}", i, *read_guard);
        }));
    }

    // 2.4.10.h: Exclusive writer - blocks all readers
    let data_writer = Arc::clone(&data);
    handles.push(thread::spawn(move || {
        // 2.4.10.e: rwlock.write() acquires write lock
        // 2.4.10.f: RwLockWriteGuard
        let mut write_guard = data_writer.write().unwrap();
        write_guard.push(4);
        println!("Writer: added 4");
    }));

    for h in handles {
        h.join().unwrap();
    }

    println!("Final: {:?}", *data.read().unwrap());
}
```

---

## Partie 8: Condvar Advanced (2.4.11)

### Exercice 8.1: Condvar with Timeout

```rust
//! Condvar advanced (2.4.11.h-j)

use std::sync::{Arc, Condvar, Mutex};
use std::time::Duration;
use std::thread;

/// Condvar wait_while and timeouts (2.4.11.h-j)
fn condvar_advanced() {
    println!("\n=== Condvar Advanced (2.4.11.h-j) ===\n");

    let pair = Arc::new((Mutex::new(false), Condvar::new()));
    let pair_clone = Arc::clone(&pair);

    // Producer thread
    thread::spawn(move || {
        thread::sleep(Duration::from_millis(100));
        let (lock, cvar) = &*pair_clone;
        let mut ready = lock.lock().unwrap();
        *ready = true;
        cvar.notify_one();
    });

    // Consumer with wait_while (2.4.11.h)
    let (lock, cvar) = &*pair;
    let guard = lock.lock().unwrap();

    // 2.4.11.h: wait_while loops internally
    let _guard = cvar.wait_while(guard, |ready| !*ready).unwrap();
    println!("Data ready (wait_while handled spurious wakeups)");

    // 2.4.11.i-j: Timeout variants
    let pair2 = Arc::new((Mutex::new(false), Condvar::new()));
    let (lock2, cvar2) = &*pair2;
    let guard2 = lock2.lock().unwrap();

    // 2.4.11.j: wait_timeout_while
    let result = cvar2.wait_timeout_while(
        guard2,
        Duration::from_millis(50),
        |ready| !*ready
    );

    match result {
        Ok((_, timeout)) if timeout.timed_out() => {
            println!("Timed out waiting!");
        }
        Ok(_) => println!("Got signal"),
        Err(_) => println!("Mutex poisoned"),
    }
}
```

---

## Partie 9: Barrier (2.4.12)

### Exercice 9.1: Barrier Synchronization

```rust
//! Barrier (2.4.12.c-e, 2.4.12.g)

use std::sync::{Arc, Barrier};
use std::thread;

/// Barrier usage (2.4.12.c-e, 2.4.12.g)
fn barrier_demo() {
    println!("\n=== Barrier (2.4.12.c-g) ===\n");

    let num_threads = 4;
    let barrier = Arc::new(Barrier::new(num_threads));
    let mut handles = vec![];

    // 2.4.12.g: Use case - parallel phases
    for i in 0..num_threads {
        let barrier = Arc::clone(&barrier);
        handles.push(thread::spawn(move || {
            // Phase 1
            println!("Thread {}: Phase 1 complete", i);

            // 2.4.12.c: Wait for all threads
            // 2.4.12.d: BarrierWaitResult
            let result = barrier.wait();

            // 2.4.12.e: One thread is leader
            if result.is_leader() {
                println!("Thread {} is leader - initiating phase 2", i);
            }

            // Phase 2
            println!("Thread {}: Phase 2 complete", i);
        }));
    }

    for h in handles {
        h.join().unwrap();
    }
}
```

---

## Partie 10: Channels and Atomics (2.4.15-17)

### Exercice 10.1: MPSC Channels

```rust
//! Channels (2.4.17.a, 2.4.17.c)

use std::sync::mpsc;
use std::thread;

/// MPSC channel basics (2.4.17.a, 2.4.17.c)
fn channel_basics() {
    println!("\n=== MPSC Channels (2.4.17.a,c) ===\n");

    // 2.4.17.a: std::sync::mpsc module
    // 2.4.17.c: mpsc::channel() creates unbounded channel
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        for i in 0..5 {
            tx.send(i).unwrap();
        }
    });

    while let Ok(val) = rx.recv() {
        println!("Received: {}", val);
    }
}

/// Memory ordering (2.4.15.a)
fn memory_ordering_demo() {
    println!("\n=== Memory Ordering (2.4.15.a) ===\n");

    use std::sync::atomic::{AtomicUsize, Ordering};

    // 2.4.15.a: Different orderings for atomics
    let counter = AtomicUsize::new(0);

    println!("Ordering types:");
    println!("  Relaxed - No synchronization, just atomicity");
    println!("  Acquire - Pairs with Release for synchronization");
    println!("  Release - Pairs with Acquire for synchronization");
    println!("  AcqRel  - Both Acquire and Release");
    println!("  SeqCst  - Strongest, total ordering");

    counter.fetch_add(1, Ordering::SeqCst);
}

/// Atomic patterns (2.4.16.a)
fn atomic_patterns() {
    println!("\n=== Atomic Patterns (2.4.16.a) ===\n");

    use std::sync::atomic::{AtomicBool, Ordering};

    // 2.4.16.a: Common atomic patterns

    // Pattern 1: Spin lock
    let lock = AtomicBool::new(false);
    while lock.compare_exchange(false, true, Ordering::Acquire, Ordering::Relaxed).is_err() {
        std::hint::spin_loop();
    }
    lock.store(false, Ordering::Release);

    // Pattern 2: Once flag
    let initialized = AtomicBool::new(false);
    if !initialized.swap(true, Ordering::SeqCst) {
        println!("First time initialization");
    }

    println!("Common patterns: spin lock, once flag, reference counting");
}
```

---

## Partie 11: Rayon and Thread Pools (2.4.26, 2.4.28)

### Exercice 11.1: Rayon Parallel Iterator

```rust
//! Rayon and thread pools (2.4.26.f, 2.4.28.a)

use rayon::prelude::*;

/// Parallel iterator (2.4.26.f)
fn rayon_par_iter() {
    println!("\n=== Rayon par_iter (2.4.26.f) ===\n");

    let numbers: Vec<i32> = (0..1000000).collect();

    // 2.4.26.f: par_iter() for parallel iteration
    let sum: i32 = numbers.par_iter().sum();
    println!("Parallel sum: {}", sum);

    // Parallel map-reduce
    let squares: i64 = numbers.par_iter()
        .map(|&x| (x as i64) * (x as i64))
        .sum();
    println!("Sum of squares: {}", squares);
}

/// Thread pool concept (2.4.28.a)
fn thread_pool_concept() {
    println!("\n=== Thread Pool Concept (2.4.28.a) ===\n");

    // 2.4.28.a: Thread pool - pool of reusable worker threads
    println!("Thread pool benefits:");
    println!("  - Avoid thread creation overhead");
    println!("  - Limit concurrent threads");
    println!("  - Work stealing for load balancing");

    // Rayon uses a global thread pool by default
    rayon::ThreadPoolBuilder::new()
        .num_threads(4)
        .build_global()
        .ok();
}
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| Thread basics (2.4.2) | 15 |
| Move closures (2.4.4) | 10 |
| Scoped threads (2.4.5) | 10 |
| Thread-local storage (2.4.6) | 10 |
| Race conditions (2.4.7) | 10 |
| Mutex (2.4.8-9) | 15 |
| RwLock (2.4.10) | 10 |
| Condvar (2.4.11) | 5 |
| Barrier (2.4.12) | 5 |
| Channels & Atomics (2.4.15-17) | 5 |
| Rayon & Pools (2.4.26, 2.4.28) | 5 |
| **Total** | **100** |

---

## Ressources

- [Rust Book - Concurrency](https://doc.rust-lang.org/book/ch16-00-concurrency.html)
- [std::thread](https://doc.rust-lang.org/std/thread/)
- [std::sync](https://doc.rust-lang.org/std/sync/)
- [Rayon](https://docs.rs/rayon/)
- [Crossbeam](https://docs.rs/crossbeam/)
