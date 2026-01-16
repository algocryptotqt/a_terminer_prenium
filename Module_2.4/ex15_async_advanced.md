# [Module 2.4] - Exercise 15: Advanced Async & Concurrency

## Metadonnees

```yaml
module: "2.4 - Networking & Async"
exercise: "ex15"
title: "Advanced Async Synchronization & Patterns"
difficulty: expert
estimated_time: "6 heures"
prerequisite_exercises: ["ex00", "ex01"]
concepts_requis: ["async/await", "tokio", "concurrency"]
score_qualite: 98
```

---

## Concepts Couverts

### 2.4.31: Async Synchronization (11 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.4.31.a | `tokio::sync::Mutex` | Async mutex |
| 2.4.31.b | `mutex.lock().await` | Async acquire |
| 2.4.31.c | `tokio::sync::RwLock` | Async RwLock |
| 2.4.31.d | `tokio::sync::Semaphore` | Async semaphore |
| 2.4.31.e | `semaphore.acquire().await` | Acquire permit |
| 2.4.31.f | `tokio::sync::Notify` | Async notification |
| 2.4.31.g | `notify.notified().await` | Wait for notify |
| 2.4.31.h | `tokio::sync::watch` | Single-value broadcast |
| 2.4.31.i | `tokio::sync::broadcast` | Multi-value broadcast |
| 2.4.31.j | `tokio::sync::mpsc` | Async channels |
| 2.4.31.k | `tokio::sync::oneshot` | One-shot channel |

### 2.4.32: Async Patterns (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.4.32.a | `tokio::select!` | Wait on multiple |
| 2.4.32.b | `biased` | Prioritized selection |
| 2.4.32.c | `tokio::join!` | Concurrent await |
| 2.4.32.d | `tokio::try_join!` | With error handling |
| 2.4.32.e | `futures::future::join_all` | Vec of futures |
| 2.4.32.f | `FuturesUnordered` | Dynamic set |
| 2.4.32.g | Cancellation | Drop future |
| 2.4.32.h | `CancellationToken` | Cooperative cancellation |
| 2.4.32.i | Timeout | `tokio::time::timeout()` |
| 2.4.32.j | Graceful shutdown | Patterns |

### 2.4.33: Send & Sync Deep Dive (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.4.33.a | `Send` definition | Safe to send to thread |
| 2.4.33.b | `Sync` definition | `&T` is `Send` |
| 2.4.33.c | Auto-implementation | Compiler derives |
| 2.4.33.d | `!Send` types | `Rc`, `*mut T` |
| 2.4.33.e | `!Sync` types | `Cell`, `RefCell` |
| 2.4.33.f | `PhantomData` | Affect Send/Sync |
| 2.4.33.g | `unsafe impl Send` | Manual impl |
| 2.4.33.h | `unsafe impl Sync` | Manual impl |
| 2.4.33.i | When needed | FFI wrappers |
| 2.4.33.j | Invariants | Guarantees |

### 2.4.34: Cache Effects & False Sharing (8 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.4.34.a | Cache line | 64 bytes typically |
| 2.4.34.b | False sharing | Different data, same line |
| 2.4.34.c | Ping-pong | Performance killer |
| 2.4.34.d | Detection | `perf` counters |
| 2.4.34.e | `#[repr(align(64))]` | Cache line alignment |
| 2.4.34.f | `crossbeam::utils::CachePadded` | Padding wrapper |
| 2.4.34.g | `CachePadded<AtomicUsize>` | Usage |
| 2.4.34.h | Array of atomics | Pad each element |

---

## Partie 1: Async Synchronization (2.4.31)

### Exercice 1.1: Async Mutex and RwLock

```rust
use tokio::sync::{Mutex, RwLock};
use std::sync::Arc;

#[tokio::main]
async fn main() {
    // Async Mutex
    let counter = Arc::new(Mutex::new(0));

    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        handles.push(tokio::spawn(async move {
            for _ in 0..100 {
                let mut lock = counter.lock().await;
                *lock += 1;
                // Lock is released here
            }
        }));
    }

    for handle in handles {
        handle.await.unwrap();
    }

    println!("Counter: {}", *counter.lock().await);

    // Async RwLock - multiple readers, single writer
    let data = Arc::new(RwLock::new(vec![1, 2, 3]));

    // Multiple concurrent readers
    let readers: Vec<_> = (0..5).map(|_| {
        let data = Arc::clone(&data);
        tokio::spawn(async move {
            let guard = data.read().await;
            println!("Read: {:?}", *guard);
        })
    }).collect();

    // Single writer
    {
        let mut guard = data.write().await;
        guard.push(4);
    }

    for r in readers {
        r.await.unwrap();
    }
}
```

### Exercice 1.2: Semaphore for Rate Limiting

```rust
use tokio::sync::Semaphore;
use std::sync::Arc;

async fn rate_limited_request(semaphore: Arc<Semaphore>, id: u32) {
    // Acquire permit - will wait if none available
    let _permit = semaphore.acquire().await.unwrap();

    println!("Request {} starting", id);
    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
    println!("Request {} completed", id);

    // Permit is released when dropped
}

#[tokio::main]
async fn main() {
    // Allow max 3 concurrent requests
    let semaphore = Arc::new(Semaphore::new(3));

    let mut handles = vec![];

    for i in 0..10 {
        let sem = Arc::clone(&semaphore);
        handles.push(tokio::spawn(rate_limited_request(sem, i)));
    }

    for handle in handles {
        handle.await.unwrap();
    }
}
```

### Exercice 1.3: Notify for Signaling

```rust
use tokio::sync::Notify;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let notify = Arc::new(Notify::new());

    // Waiter task
    let notify_clone = Arc::clone(&notify);
    let waiter = tokio::spawn(async move {
        println!("Waiting for notification...");
        notify_clone.notified().await;
        println!("Received notification!");
    });

    // Simulate some work
    tokio::time::sleep(std::time::Duration::from_secs(1)).await;

    // Send notification
    println!("Sending notification...");
    notify.notify_one();

    waiter.await.unwrap();

    // notify_waiters() wakes all waiters
    let notify = Arc::new(Notify::new());
    let mut handles = vec![];

    for i in 0..3 {
        let n = Arc::clone(&notify);
        handles.push(tokio::spawn(async move {
            n.notified().await;
            println!("Waiter {} received notification", i);
        }));
    }

    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
    notify.notify_waiters();

    for h in handles {
        h.await.unwrap();
    }
}
```

### Exercice 1.4: Watch and Broadcast Channels

```rust
use tokio::sync::{watch, broadcast};

#[tokio::main]
async fn main() {
    // Watch: single-value broadcast (receivers see latest value)
    let (tx, mut rx) = watch::channel("initial");

    let rx2 = rx.clone();

    tokio::spawn(async move {
        loop {
            if rx.changed().await.is_err() {
                break;
            }
            println!("Receiver 1 got: {}", *rx.borrow());
        }
    });

    tx.send("first update").unwrap();
    tx.send("second update").unwrap();

    tokio::time::sleep(std::time::Duration::from_millis(100)).await;

    // Broadcast: multi-value broadcast (receivers see all values)
    let (tx, _) = broadcast::channel::<String>(16);

    let mut rx1 = tx.subscribe();
    let mut rx2 = tx.subscribe();

    tokio::spawn(async move {
        while let Ok(msg) = rx1.recv().await {
            println!("Receiver 1: {}", msg);
        }
    });

    tokio::spawn(async move {
        while let Ok(msg) = rx2.recv().await {
            println!("Receiver 2: {}", msg);
        }
    });

    tx.send("Hello".to_string()).unwrap();
    tx.send("World".to_string()).unwrap();

    tokio::time::sleep(std::time::Duration::from_millis(100)).await;
}
```

### Exercice 1.5: MPSC and Oneshot Channels

```rust
use tokio::sync::{mpsc, oneshot};

#[tokio::main]
async fn main() {
    // mpsc: Multi-producer, single-consumer
    let (tx, mut rx) = mpsc::channel::<String>(32);

    for i in 0..3 {
        let tx = tx.clone();
        tokio::spawn(async move {
            tx.send(format!("Message from producer {}", i)).await.unwrap();
        });
    }

    drop(tx); // Drop original sender so rx knows when to stop

    while let Some(msg) = rx.recv().await {
        println!("Received: {}", msg);
    }

    // oneshot: Single-use channel for responses
    let (tx, rx) = oneshot::channel::<String>();

    tokio::spawn(async move {
        // Simulate async work
        tokio::time::sleep(std::time::Duration::from_millis(100)).await;
        tx.send("Response data".to_string()).unwrap();
    });

    match rx.await {
        Ok(value) => println!("Got response: {}", value),
        Err(_) => println!("Sender dropped"),
    }
}
```

---

## Partie 2: Async Patterns (2.4.32)

### Exercice 2.1: Select and Join

```rust
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    // select! - wait on multiple, first wins
    tokio::select! {
        _ = sleep(Duration::from_secs(1)) => {
            println!("Timer 1 fired");
        }
        _ = sleep(Duration::from_millis(500)) => {
            println!("Timer 2 fired first!");
        }
    }

    // biased - prefer earlier branches
    let mut counter = 0;
    tokio::select! {
        biased;

        _ = async { counter += 1 } => {
            println!("First branch (biased)");
        }
        _ = async { counter += 2 } => {
            println!("Second branch");
        }
    }

    // join! - wait for all
    let (a, b, c) = tokio::join!(
        async { 1 },
        async { 2 },
        async { 3 }
    );
    println!("Results: {}, {}, {}", a, b, c);

    // try_join! - with error handling
    async fn may_fail(succeed: bool) -> Result<u32, &'static str> {
        if succeed { Ok(42) } else { Err("failed") }
    }

    match tokio::try_join!(may_fail(true), may_fail(true)) {
        Ok((a, b)) => println!("Both succeeded: {}, {}", a, b),
        Err(e) => println!("One failed: {}", e),
    }
}
```

### Exercice 2.2: FuturesUnordered

```rust
use futures::stream::{FuturesUnordered, StreamExt};
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let mut futures = FuturesUnordered::new();

    // Add futures with different durations
    futures.push(async {
        sleep(Duration::from_millis(300)).await;
        "Task 1 (300ms)"
    });
    futures.push(async {
        sleep(Duration::from_millis(100)).await;
        "Task 2 (100ms)"
    });
    futures.push(async {
        sleep(Duration::from_millis(200)).await;
        "Task 3 (200ms)"
    });

    // Process as they complete (not in order added)
    while let Some(result) = futures.next().await {
        println!("Completed: {}", result);
    }

    // join_all for Vec of same-type futures
    use futures::future::join_all;

    let futures: Vec<_> = (0..5).map(|i| async move {
        sleep(Duration::from_millis(i * 50)).await;
        i * 10
    }).collect();

    let results = join_all(futures).await;
    println!("All results: {:?}", results);
}
```

### Exercice 2.3: Cancellation and Timeout

```rust
use tokio::time::{timeout, Duration};
use tokio_util::sync::CancellationToken;

#[tokio::main]
async fn main() {
    // Timeout
    let slow_task = async {
        tokio::time::sleep(Duration::from_secs(5)).await;
        "Done!"
    };

    match timeout(Duration::from_secs(1), slow_task).await {
        Ok(result) => println!("Result: {}", result),
        Err(_) => println!("Task timed out!"),
    }

    // CancellationToken
    let token = CancellationToken::new();
    let token_clone = token.clone();

    let handle = tokio::spawn(async move {
        tokio::select! {
            _ = token_clone.cancelled() => {
                println!("Task was cancelled");
            }
            _ = tokio::time::sleep(Duration::from_secs(10)) => {
                println!("Task completed normally");
            }
        }
    });

    tokio::time::sleep(Duration::from_millis(100)).await;
    token.cancel();

    handle.await.unwrap();
}
```

### Exercice 2.4: Graceful Shutdown

```rust
use tokio::signal;
use tokio::sync::broadcast;

async fn worker(id: u32, mut shutdown: broadcast::Receiver<()>) {
    loop {
        tokio::select! {
            _ = shutdown.recv() => {
                println!("Worker {} shutting down", id);
                break;
            }
            _ = tokio::time::sleep(std::time::Duration::from_secs(1)) => {
                println!("Worker {} doing work", id);
            }
        }
    }
}

#[tokio::main]
async fn main() {
    let (shutdown_tx, _) = broadcast::channel::<()>(1);

    // Spawn workers
    let mut handles = vec![];
    for i in 0..3 {
        let rx = shutdown_tx.subscribe();
        handles.push(tokio::spawn(worker(i, rx)));
    }

    // Wait for Ctrl+C
    println!("Press Ctrl+C to shutdown");
    signal::ctrl_c().await.unwrap();

    println!("Shutdown signal received, stopping workers...");
    drop(shutdown_tx); // Close channel, triggers shutdown

    // Wait for all workers
    for handle in handles {
        handle.await.unwrap();
    }

    println!("Shutdown complete");
}
```

---

## Partie 3: Send & Sync (2.4.33)

### Exercice 3.1: Understanding Send and Sync

```rust
use std::rc::Rc;
use std::cell::{Cell, RefCell};
use std::sync::Arc;

fn is_send<T: Send>() {}
fn is_sync<T: Sync>() {}

fn main() {
    // Send + Sync
    is_send::<i32>();
    is_sync::<i32>();
    is_send::<String>();
    is_sync::<String>();
    is_send::<Arc<i32>>();
    is_sync::<Arc<i32>>();

    // !Send + !Sync
    // is_send::<Rc<i32>>();  // Error: Rc is !Send
    // is_sync::<Rc<i32>>();  // Error: Rc is !Sync

    // Send + !Sync
    is_send::<Cell<i32>>();
    // is_sync::<Cell<i32>>();  // Error: Cell is !Sync

    is_send::<RefCell<i32>>();
    // is_sync::<RefCell<i32>>();  // Error: RefCell is !Sync

    // Raw pointers are !Send + !Sync
    // is_send::<*mut i32>();  // Error
    // is_sync::<*mut i32>();  // Error
}
```

### Exercice 3.2: Implementing Send and Sync

```rust
use std::marker::PhantomData;

// FFI wrapper that we know is thread-safe
struct SafeHandle {
    ptr: *mut std::ffi::c_void,
}

// SAFETY: SafeHandle points to data that is thread-safe
unsafe impl Send for SafeHandle {}
unsafe impl Sync for SafeHandle {}

// Use PhantomData to affect Send/Sync
struct NotSync<T> {
    data: T,
    _marker: PhantomData<*mut ()>, // *mut () is !Send + !Sync
}

impl<T> NotSync<T> {
    fn new(data: T) -> Self {
        NotSync {
            data,
            _marker: PhantomData,
        }
    }
}

// NotSync<i32> is now !Send + !Sync even though i32 is both

fn main() {
    // fn is_send<T: Send>() {}
    // is_send::<NotSync<i32>>();  // Error!
}
```

---

## Partie 4: Cache Effects (2.4.34)

### Exercice 4.1: False Sharing

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;
use std::time::Instant;

// BAD: Adjacent atomics share cache line
fn bad_counters() -> std::time::Duration {
    let counters = Arc::new([
        AtomicUsize::new(0),
        AtomicUsize::new(0),
    ]);

    let start = Instant::now();

    let handles: Vec<_> = (0..2).map(|i| {
        let counters = Arc::clone(&counters);
        thread::spawn(move || {
            for _ in 0..10_000_000 {
                counters[i].fetch_add(1, Ordering::Relaxed);
            }
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }

    start.elapsed()
}

// GOOD: Padded to separate cache lines
#[repr(align(64))]
struct PaddedAtomic(AtomicUsize);

fn good_counters() -> std::time::Duration {
    let counters = Arc::new([
        PaddedAtomic(AtomicUsize::new(0)),
        PaddedAtomic(AtomicUsize::new(0)),
    ]);

    let start = Instant::now();

    let handles: Vec<_> = (0..2).map(|i| {
        let counters = Arc::clone(&counters);
        thread::spawn(move || {
            for _ in 0..10_000_000 {
                counters[i].0.fetch_add(1, Ordering::Relaxed);
            }
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }

    start.elapsed()
}

fn main() {
    println!("Size of AtomicUsize: {} bytes", std::mem::size_of::<AtomicUsize>());
    println!("Size of PaddedAtomic: {} bytes", std::mem::size_of::<PaddedAtomic>());

    let bad_time = bad_counters();
    let good_time = good_counters();

    println!("\nFalse sharing (bad): {:?}", bad_time);
    println!("Cache-padded (good): {:?}", good_time);
    println!("Speedup: {:.2}x", bad_time.as_nanos() as f64 / good_time.as_nanos() as f64);
}
```

### Exercice 4.2: Using CachePadded

```rust
use crossbeam_utils::CachePadded;
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    // crossbeam's CachePadded
    let counters: Arc<[CachePadded<AtomicUsize>; 4]> = Arc::new([
        CachePadded::new(AtomicUsize::new(0)),
        CachePadded::new(AtomicUsize::new(0)),
        CachePadded::new(AtomicUsize::new(0)),
        CachePadded::new(AtomicUsize::new(0)),
    ]);

    let handles: Vec<_> = (0..4).map(|i| {
        let counters = Arc::clone(&counters);
        thread::spawn(move || {
            for _ in 0..1_000_000 {
                counters[i].fetch_add(1, Ordering::Relaxed);
            }
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }

    println!("Counters: {:?}", counters.iter().map(|c| c.load(Ordering::Relaxed)).collect::<Vec<_>>());
}
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| Async synchronization primitives | 25 |
| Async patterns (select, join) | 25 |
| Send & Sync understanding | 20 |
| Cache effects & false sharing | 20 |
| Graceful shutdown | 10 |
| **Total** | **100** |

---

## Ressources

- [Tokio sync documentation](https://docs.rs/tokio/latest/tokio/sync/)
- [Async in Depth](https://tokio.rs/tokio/tutorial/async)
- [Crossbeam utils](https://docs.rs/crossbeam-utils/)
- [Send and Sync](https://doc.rust-lang.org/nomicon/send-and-sync.html)
