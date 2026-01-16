# Exercise 07: Parallel & Concurrent Algorithms

## Concepts Covered
- **1.7.14.d-l** Parallel algorithms, work-span analysis
- **1.7.15.d-k** Lock-free data structures, concurrent programming

## Objective

Implement parallel algorithms and understand concurrency primitives.

## Requirements

### Rust Implementation

```rust
pub mod parallel_primitives {
    use std::sync::Arc;

    /// Parallel prefix sum (scan)
    pub fn parallel_prefix_sum(arr: &[i64]) -> Vec<i64>;

    /// Parallel reduce
    pub fn parallel_reduce<T, F>(arr: &[T], identity: T, op: F) -> T
    where
        T: Send + Sync + Clone,
        F: Fn(T, T) -> T + Send + Sync + Clone;

    /// Parallel map
    pub fn parallel_map<T, U, F>(arr: &[T], f: F) -> Vec<U>
    where
        T: Send + Sync,
        U: Send,
        F: Fn(&T) -> U + Send + Sync;

    /// Parallel filter
    pub fn parallel_filter<T, F>(arr: &[T], predicate: F) -> Vec<T>
    where
        T: Send + Sync + Clone,
        F: Fn(&T) -> bool + Send + Sync;

    /// Work-stealing scheduler
    pub struct WorkStealingScheduler {
        num_workers: usize,
    }

    impl WorkStealingScheduler {
        pub fn new(num_workers: usize) -> Self;
        pub fn run<F>(&self, tasks: Vec<F>)
        where
            F: FnOnce() + Send;
    }
}

pub mod parallel_sorting {
    /// Parallel merge sort
    pub fn parallel_merge_sort<T: Ord + Send + Clone>(arr: &mut [T]);

    /// Parallel quicksort
    pub fn parallel_quicksort<T: Ord + Send>(arr: &mut [T]);

    /// Parallel radix sort
    pub fn parallel_radix_sort(arr: &mut [u64]);

    /// Sample sort (good for distributed sorting)
    pub fn sample_sort<T: Ord + Send + Clone>(arr: &mut [T], num_buckets: usize);

    /// Bitonic sort (SIMD-friendly)
    pub fn bitonic_sort<T: Ord + Send>(arr: &mut [T]);
}

pub mod parallel_graph {
    /// Parallel BFS
    pub fn parallel_bfs(adj: &[Vec<usize>], source: usize) -> Vec<i32>;

    /// Parallel connected components (label propagation)
    pub fn parallel_connected_components(adj: &[Vec<usize>]) -> Vec<usize>;

    /// Parallel PageRank
    pub fn parallel_pagerank(adj: &[Vec<usize>], iterations: usize) -> Vec<f64>;

    /// Parallel shortest paths (delta-stepping)
    pub fn delta_stepping(
        adj: &[Vec<(usize, i64)>],
        source: usize,
        delta: i64,
    ) -> Vec<i64>;
}

pub mod concurrent_ds {
    use std::sync::atomic::{AtomicUsize, Ordering};
    use std::sync::Arc;

    /// Lock-free stack (Treiber stack)
    pub struct LockFreeStack<T> {
        head: AtomicUsize,
        // Uses hazard pointers or epoch-based reclamation
    }

    impl<T: Send> LockFreeStack<T> {
        pub fn new() -> Self;
        pub fn push(&self, val: T);
        pub fn pop(&self) -> Option<T>;
    }

    /// Lock-free queue (Michael-Scott queue)
    pub struct LockFreeQueue<T> {
        head: AtomicUsize,
        tail: AtomicUsize,
    }

    impl<T: Send> LockFreeQueue<T> {
        pub fn new() -> Self;
        pub fn enqueue(&self, val: T);
        pub fn dequeue(&self) -> Option<T>;
    }

    /// Concurrent hash map
    pub struct ConcurrentHashMap<K, V> {
        buckets: Vec<std::sync::RwLock<Vec<(K, V)>>>,
    }

    impl<K: std::hash::Hash + Eq + Clone, V: Clone> ConcurrentHashMap<K, V> {
        pub fn new(num_buckets: usize) -> Self;
        pub fn insert(&self, key: K, value: V);
        pub fn get(&self, key: &K) -> Option<V>;
        pub fn remove(&self, key: &K) -> Option<V>;
    }

    /// Skip list (concurrent)
    pub struct ConcurrentSkipList<K: Ord, V> {
        // Lock-free skip list
    }
}

pub mod thread_pool {
    use std::sync::{Arc, Mutex, Condvar};
    use std::sync::mpsc;

    pub struct ThreadPool {
        workers: Vec<Worker>,
        sender: Option<mpsc::Sender<Job>>,
    }

    impl ThreadPool {
        pub fn new(size: usize) -> Self;
        pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static;
        pub fn shutdown(&mut self);
    }

    type Job = Box<dyn FnOnce() + Send + 'static>;

    struct Worker {
        id: usize,
        thread: Option<std::thread::JoinHandle<()>>,
    }

    /// Fork-join parallelism
    pub fn fork_join<T, F>(tasks: Vec<F>) -> Vec<T>
    where
        T: Send + 'static,
        F: FnOnce() -> T + Send + 'static;

    /// Parallel divide and conquer
    pub fn parallel_divide_conquer<T, F, C, M>(
        problem: T,
        is_base_case: C,
        solve_base: F,
        divide: fn(T) -> Vec<T>,
        merge: M,
    ) -> T
    where
        T: Send + Clone,
        F: Fn(T) -> T + Send + Sync + Clone,
        C: Fn(&T) -> bool + Send + Sync + Clone,
        M: Fn(Vec<T>) -> T + Send + Sync + Clone;
}

pub mod synchronization {
    use std::sync::{Mutex, RwLock, Barrier, Condvar};

    /// Reader-writer lock wrapper
    pub struct RWLock<T> {
        inner: RwLock<T>,
    }

    impl<T> RWLock<T> {
        pub fn new(val: T) -> Self;
        pub fn read<F, R>(&self, f: F) -> R
        where
            F: FnOnce(&T) -> R;
        pub fn write<F, R>(&self, f: F) -> R
        where
            F: FnOnce(&mut T) -> R;
    }

    /// Semaphore
    pub struct Semaphore {
        count: Mutex<usize>,
        condvar: Condvar,
    }

    impl Semaphore {
        pub fn new(count: usize) -> Self;
        pub fn acquire(&self);
        pub fn release(&self);
    }

    /// Barrier for thread synchronization
    pub fn barrier_example(num_threads: usize) {
        let barrier = std::sync::Arc::new(std::sync::Barrier::new(num_threads));
        // Use barrier.wait() to synchronize threads
    }
}
```

### Python Implementation

```python
from typing import List, TypeVar, Callable
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import threading
import queue

T = TypeVar('T')

def parallel_map(arr: List[T], f: Callable[[T], T], num_workers: int = 4) -> List[T]: ...
def parallel_reduce(arr: List[T], identity: T, op: Callable[[T, T], T]) -> T: ...
def parallel_prefix_sum(arr: List[int]) -> List[int]: ...

def parallel_merge_sort(arr: List[int], num_workers: int = 4) -> List[int]: ...
def parallel_quicksort(arr: List[int]) -> List[int]: ...

def parallel_bfs(adj: List[List[int]], source: int) -> List[int]: ...
def parallel_pagerank(adj: List[List[int]], iterations: int) -> List[float]: ...

class ThreadPool:
    def __init__(self, num_workers: int): ...
    def submit(self, f: Callable) -> None: ...
    def shutdown(self) -> None: ...

class ThreadSafeQueue:
    def __init__(self): ...
    def put(self, item) -> None: ...
    def get(self) -> any: ...

class Semaphore:
    def __init__(self, count: int): ...
    def acquire(self) -> None: ...
    def release(self) -> None: ...
```

## Test Cases

```rust
#[test]
fn test_parallel_prefix_sum() {
    let arr = vec![1, 2, 3, 4, 5, 6, 7, 8];
    let prefix = parallel_prefix_sum(&arr);

    assert_eq!(prefix, vec![1, 3, 6, 10, 15, 21, 28, 36]);
}

#[test]
fn test_parallel_reduce() {
    let arr = vec![1, 2, 3, 4, 5, 6, 7, 8];
    let sum = parallel_reduce(&arr, 0, |a, b| a + b);

    assert_eq!(sum, 36);
}

#[test]
fn test_parallel_map() {
    let arr = vec![1, 2, 3, 4, 5];
    let squared = parallel_map(&arr, |x| x * x);

    assert_eq!(squared, vec![1, 4, 9, 16, 25]);
}

#[test]
fn test_parallel_merge_sort() {
    let mut arr = vec![5, 2, 8, 1, 9, 3, 7, 4, 6];
    parallel_merge_sort(&mut arr);

    assert_eq!(arr, vec![1, 2, 3, 4, 5, 6, 7, 8, 9]);
}

#[test]
fn test_parallel_quicksort() {
    let mut arr: Vec<i32> = (0..1000).rev().collect();
    parallel_quicksort(&mut arr);

    for i in 1..arr.len() {
        assert!(arr[i-1] <= arr[i]);
    }
}

#[test]
fn test_parallel_bfs() {
    let adj = vec![
        vec![1, 2],
        vec![0, 3],
        vec![0, 3],
        vec![1, 2],
    ];

    let dist = parallel_bfs(&adj, 0);
    assert_eq!(dist, vec![0, 1, 1, 2]);
}

#[test]
fn test_lock_free_stack() {
    let stack = Arc::new(LockFreeStack::new());
    let mut handles = vec![];

    // Multiple threads push
    for i in 0..4 {
        let stack = Arc::clone(&stack);
        handles.push(std::thread::spawn(move || {
            for j in 0..100 {
                stack.push(i * 100 + j);
            }
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }

    // Pop all and verify count
    let mut count = 0;
    while stack.pop().is_some() {
        count += 1;
    }
    assert_eq!(count, 400);
}

#[test]
fn test_lock_free_queue() {
    let queue = Arc::new(LockFreeQueue::new());

    // Producer
    let q1 = Arc::clone(&queue);
    let producer = std::thread::spawn(move || {
        for i in 0..100 {
            q1.enqueue(i);
        }
    });

    // Consumer
    let q2 = Arc::clone(&queue);
    let consumer = std::thread::spawn(move || {
        let mut sum = 0;
        let mut received = 0;
        while received < 100 {
            if let Some(val) = q2.dequeue() {
                sum += val;
                received += 1;
            }
        }
        sum
    });

    producer.join().unwrap();
    let sum = consumer.join().unwrap();
    assert_eq!(sum, 4950);  // 0 + 1 + ... + 99
}

#[test]
fn test_concurrent_hashmap() {
    let map = Arc::new(ConcurrentHashMap::new(16));
    let mut handles = vec![];

    for i in 0..4 {
        let map = Arc::clone(&map);
        handles.push(std::thread::spawn(move || {
            for j in 0..100 {
                map.insert(format!("{}_{}", i, j), i * 100 + j);
            }
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }

    // Verify some entries
    assert_eq!(map.get(&"0_50".to_string()), Some(50));
    assert_eq!(map.get(&"3_99".to_string()), Some(399));
}

#[test]
fn test_thread_pool() {
    let pool = ThreadPool::new(4);
    let counter = Arc::new(std::sync::atomic::AtomicUsize::new(0));

    for _ in 0..100 {
        let counter = Arc::clone(&counter);
        pool.execute(move || {
            counter.fetch_add(1, std::sync::atomic::Ordering::SeqCst);
        });
    }

    pool.shutdown();
    assert_eq!(counter.load(std::sync::atomic::Ordering::SeqCst), 100);
}

#[test]
fn test_fork_join() {
    let tasks: Vec<Box<dyn FnOnce() -> i32 + Send>> = (0..10)
        .map(|i| Box::new(move || i * i) as Box<dyn FnOnce() -> i32 + Send>)
        .collect();

    let results = fork_join(tasks);
    assert_eq!(results, vec![0, 1, 4, 9, 16, 25, 36, 49, 64, 81]);
}

#[test]
fn test_semaphore() {
    let sem = Arc::new(Semaphore::new(3));  // Allow 3 concurrent

    let active = Arc::new(std::sync::atomic::AtomicUsize::new(0));
    let max_active = Arc::new(std::sync::atomic::AtomicUsize::new(0));

    let mut handles = vec![];
    for _ in 0..10 {
        let sem = Arc::clone(&sem);
        let active = Arc::clone(&active);
        let max_active = Arc::clone(&max_active);

        handles.push(std::thread::spawn(move || {
            sem.acquire();

            let current = active.fetch_add(1, std::sync::atomic::Ordering::SeqCst) + 1;
            max_active.fetch_max(current, std::sync::atomic::Ordering::SeqCst);

            std::thread::sleep(std::time::Duration::from_millis(10));

            active.fetch_sub(1, std::sync::atomic::Ordering::SeqCst);
            sem.release();
        }));
    }

    for handle in handles {
        handle.join().unwrap();
    }

    assert!(max_active.load(std::sync::atomic::Ordering::SeqCst) <= 3);
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Parallel primitives (map/reduce/scan) | 15 |
| Parallel sorting | 15 |
| Parallel graph algorithms | 15 |
| Lock-free stack/queue | 20 |
| Concurrent hash map | 15 |
| Thread pool | 10 |
| Synchronization primitives | 5 |
| Edge cases | 5 |
| **Total** | **100** |
