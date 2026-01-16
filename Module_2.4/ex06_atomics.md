# ex06: Atomic Operations & Memory Ordering

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Tres difficile
**Duree**: 6h
**Score qualite**: 97/100

## Concepts Couverts

### 2.4.20: Atomic Operations (11 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Atomic | Indivisible operation |
| b | stdatomic.h | C11 atomics |
| c | _Atomic | Type qualifier |
| d | atomic_int | Atomic integer |
| e | atomic_load() | Read atomically |
| f | atomic_store() | Write atomically |
| g | atomic_fetch_add() | Add and return old |
| h | atomic_fetch_sub() | Subtract |
| i | atomic_compare_exchange_strong() | CAS |
| j | atomic_compare_exchange_weak() | May fail spuriously |
| k | Memory orders | Sequential, relaxed, etc. |

### 2.4.21: Memory Ordering (10 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Reordering | Compiler and CPU |
| b | Sequential consistency | Total order |
| c | memory_order_seq_cst | Sequentially consistent |
| d | memory_order_acquire | No reads before |
| e | memory_order_release | No writes after |
| f | memory_order_acq_rel | Both |
| g | memory_order_relaxed | No ordering |
| h | memory_order_consume | Data dependency |
| i | Fences | Explicit barriers |
| j | atomic_thread_fence() | Thread fence |

---

## Sujet

Implementer une bibliotheque demonstrant les operations atomiques et l'ordering memoire.

### Structures

```c
#include <stdatomic.h>
#include <stdbool.h>

// 2.4.20.c-d: Atomic types
typedef struct {
    _Atomic int value;           // c,d: Atomic integer
    uint64_t increments;
    uint64_t decrements;
    uint64_t cas_successes;
    uint64_t cas_failures;
} atomic_counter_t;

// Atomic flag (simplest atomic)
typedef struct {
    atomic_flag flag;
    uint64_t set_count;
    uint64_t clear_count;
} atomic_flag_wrapper_t;

// 2.4.20.i-j: CAS-based structures
typedef struct node {
    int value;
    _Atomic(struct node*) next;
} atomic_node_t;

typedef struct {
    _Atomic(atomic_node_t*) head;
    _Atomic size_t size;
} atomic_stack_t;

// 2.4.21: Memory ordering demonstration
typedef struct {
    _Atomic int x;
    _Atomic int y;
    _Atomic int z;
    int observed_x;
    int observed_y;
    bool ordering_violation;
} ordering_demo_t;

// Spinlock using atomics
typedef struct {
    _Atomic bool locked;
    uint64_t acquisitions;
    uint64_t spins;
} atomic_spinlock_t;

// Sequence lock (seqlock)
typedef struct {
    _Atomic unsigned sequence;
    int data;
} seqlock_t;
```

### API

```c
// ============== ATOMIC COUNTER ==============
// 2.4.20.a-h

void atomic_counter_init(atomic_counter_t *c, int initial);

// 2.4.20.e-f: Load/Store
int atomic_counter_load(atomic_counter_t *c);                    // e
void atomic_counter_store(atomic_counter_t *c, int value);       // f

// 2.4.20.g-h: Fetch operations
int atomic_counter_fetch_add(atomic_counter_t *c, int delta);    // g
int atomic_counter_fetch_sub(atomic_counter_t *c, int delta);    // h
int atomic_counter_increment(atomic_counter_t *c);
int atomic_counter_decrement(atomic_counter_t *c);

// 2.4.20.i-j: Compare-and-swap
bool atomic_counter_cas(atomic_counter_t *c, int *expected, int desired);      // i: Strong
bool atomic_counter_cas_weak(atomic_counter_t *c, int *expected, int desired); // j: Weak

// Other atomic ops
int atomic_counter_exchange(atomic_counter_t *c, int value);
int atomic_counter_fetch_or(atomic_counter_t *c, int value);
int atomic_counter_fetch_and(atomic_counter_t *c, int value);
int atomic_counter_fetch_xor(atomic_counter_t *c, int value);

// ============== MEMORY ORDERING ==============
// 2.4.21.b-h

// 2.4.21.c: Sequential consistency (default, safest)
int atomic_load_seq_cst(atomic_counter_t *c);
void atomic_store_seq_cst(atomic_counter_t *c, int value);

// 2.4.21.d: Acquire (for loads - synchronizes with release)
int atomic_load_acquire(atomic_counter_t *c);

// 2.4.21.e: Release (for stores - synchronizes with acquire)
void atomic_store_release(atomic_counter_t *c, int value);

// 2.4.21.f: Acquire-release (for RMW ops)
int atomic_fetch_add_acq_rel(atomic_counter_t *c, int delta);

// 2.4.21.g: Relaxed (no ordering guarantees)
int atomic_load_relaxed(atomic_counter_t *c);
void atomic_store_relaxed(atomic_counter_t *c, int value);

// 2.4.21.h: Consume (data dependency - rarely used)
int atomic_load_consume(atomic_counter_t *c);

// 2.4.21.i-j: Memory fences
void fence_acquire(void);                                        // i
void fence_release(void);                                        // i
void fence_seq_cst(void);                                        // i
void fence_acq_rel(void);                                        // i

// ============== ATOMIC FLAG ==============

void atomic_flag_init(atomic_flag_wrapper_t *f);
bool atomic_flag_test_and_set(atomic_flag_wrapper_t *f);
void atomic_flag_clear(atomic_flag_wrapper_t *f);

// ============== ATOMIC SPINLOCK ==============

void atomic_spin_init(atomic_spinlock_t *s);
void atomic_spin_lock(atomic_spinlock_t *s);
void atomic_spin_unlock(atomic_spinlock_t *s);
bool atomic_spin_trylock(atomic_spinlock_t *s);

// Test-and-test-and-set (TTAS) optimization
void atomic_spin_lock_ttas(atomic_spinlock_t *s);

// ============== ATOMIC STACK ==============
// 2.4.20.i: CAS-based

void atomic_stack_init(atomic_stack_t *s);
void atomic_stack_destroy(atomic_stack_t *s);
void atomic_stack_push(atomic_stack_t *s, int value);
bool atomic_stack_pop(atomic_stack_t *s, int *value);
size_t atomic_stack_size(atomic_stack_t *s);
bool atomic_stack_is_empty(atomic_stack_t *s);

// ============== SEQLOCK ==============
// Reader-writer optimized for frequent reads

void seqlock_init(seqlock_t *s, int initial_data);
void seqlock_write(seqlock_t *s, int data);
int seqlock_read(seqlock_t *s);

// ============== ORDERING DEMONSTRATION ==============
// 2.4.21.a: Show reordering effects

void ordering_demo_init(ordering_demo_t *d);
void ordering_demo_run_relaxed(ordering_demo_t *d);
void ordering_demo_run_seq_cst(ordering_demo_t *d);
void ordering_demo_run_acq_rel(ordering_demo_t *d);
bool ordering_demo_check_violation(ordering_demo_t *d);

// ============== BENCHMARKS ==============

typedef struct {
    double mutex_time_ns;
    double atomic_time_ns;
    double spinlock_time_ns;
    double speedup_vs_mutex;
} atomic_benchmark_t;

void benchmark_atomic_vs_mutex(int threads, int ops, atomic_benchmark_t *result);
void benchmark_memory_orders(int threads, int ops);
```

---

## Exemple

```c
#include "atomics.h"

// 2.4.20: Atomic counter without locks
atomic_counter_t counter;

void *increment_worker(void *arg) {
    int iters = *(int*)arg;
    for (int i = 0; i < iters; i++) {
        // 2.4.20.g: Atomic increment
        atomic_counter_fetch_add(&counter, 1);
    }
    return NULL;
}

// 2.4.20.i: CAS loop pattern
int atomic_max(atomic_counter_t *c, int new_val) {
    int old = atomic_counter_load(c);
    while (old < new_val) {
        // 2.4.20.i: Compare-and-swap
        if (atomic_counter_cas(c, &old, new_val)) {
            return old;  // Success
        }
        // old is updated to current value, retry
    }
    return old;
}

// 2.4.21.d-e: Acquire-release for message passing
_Atomic bool ready = false;
int data = 0;

void *producer_thread(void *arg) {
    data = 42;                              // Non-atomic write
    // 2.4.21.e: Release ensures data is visible
    atomic_store_explicit(&ready, true, memory_order_release);
    return NULL;
}

void *consumer_thread(void *arg) {
    // 2.4.21.d: Acquire ensures we see data
    while (!atomic_load_explicit(&ready, memory_order_acquire)) {
        // Spin
    }
    printf("Data = %d\n", data);  // Guaranteed to see 42
    return NULL;
}

int main(void) {
    // 2.4.20.a-d: Basic atomic usage
    printf("=== Atomic Counter ===\n");
    atomic_counter_init(&counter, 0);

    pthread_t threads[4];
    int iters = 100000;
    for (int i = 0; i < 4; i++) {
        pthread_create(&threads[i], NULL, increment_worker, &iters);
    }
    for (int i = 0; i < 4; i++) {
        pthread_join(threads[i], NULL);
    }
    printf("Counter = %d (expected %d)\n",
           atomic_counter_load(&counter), 4 * iters);

    // 2.4.20.i-j: CAS demonstration
    printf("\n=== Compare-and-Swap ===\n");
    atomic_counter_store(&counter, 10);
    int expected = 10;
    bool success = atomic_counter_cas(&counter, &expected, 20);
    printf("CAS(10->20): %s, value = %d\n",
           success ? "success" : "failed", atomic_counter_load(&counter));

    expected = 10;  // Wrong expectation
    success = atomic_counter_cas(&counter, &expected, 30);
    printf("CAS(10->30): %s, expected updated to %d\n",
           success ? "success" : "failed", expected);

    // 2.4.21: Memory ordering demo
    printf("\n=== Memory Ordering ===\n");
    ordering_demo_t demo;
    ordering_demo_init(&demo);

    printf("Testing with relaxed ordering...\n");
    ordering_demo_run_relaxed(&demo);
    if (ordering_demo_check_violation(&demo)) {
        printf("Ordering violation detected (expected with relaxed)!\n");
    }

    printf("Testing with seq_cst ordering...\n");
    ordering_demo_run_seq_cst(&demo);
    if (!ordering_demo_check_violation(&demo)) {
        printf("No violations (expected with seq_cst)\n");
    }

    // 2.4.21.d-e: Acquire-release message passing
    printf("\n=== Acquire-Release ===\n");
    pthread_t prod, cons;
    pthread_create(&prod, NULL, producer_thread, NULL);
    pthread_create(&cons, NULL, consumer_thread, NULL);
    pthread_join(prod, NULL);
    pthread_join(cons, NULL);

    // Atomic spinlock
    printf("\n=== Atomic Spinlock ===\n");
    atomic_spinlock_t spin;
    atomic_spin_init(&spin);

    atomic_spin_lock(&spin);
    // Critical section
    atomic_spin_unlock(&spin);

    // TTAS optimization (reduces cache line bouncing)
    atomic_spin_lock_ttas(&spin);
    // Critical section
    atomic_spin_unlock(&spin);

    // Atomic stack
    printf("\n=== Atomic Stack ===\n");
    atomic_stack_t stack;
    atomic_stack_init(&stack);

    atomic_stack_push(&stack, 1);
    atomic_stack_push(&stack, 2);
    atomic_stack_push(&stack, 3);

    int val;
    while (atomic_stack_pop(&stack, &val)) {
        printf("Popped: %d\n", val);
    }

    atomic_stack_destroy(&stack);

    // Seqlock for read-heavy workloads
    printf("\n=== Seqlock ===\n");
    seqlock_t seq;
    seqlock_init(&seq, 100);

    // Writer (rare)
    seqlock_write(&seq, 200);

    // Reader (frequent) - retries if write happened
    int value = seqlock_read(&seq);
    printf("Seqlock value: %d\n", value);

    // 2.4.21.i-j: Fences
    printf("\n=== Memory Fences ===\n");
    int a = 0, b = 0;

    a = 1;
    fence_release();     // Ensure a=1 visible before flag
    // atomic_store(&flag, 1, memory_order_release);  // equivalent

    fence_acquire();     // Ensure we see writes before flag
    // if (atomic_load(&flag, memory_order_acquire)) ...

    // Benchmark
    printf("\n=== Benchmark ===\n");
    atomic_benchmark_t bench;
    benchmark_atomic_vs_mutex(4, 1000000, &bench);
    printf("Mutex: %.2f ns, Atomic: %.2f ns, Speedup: %.2fx\n",
           bench.mutex_time_ns, bench.atomic_time_ns, bench.speedup_vs_mutex);

    return 0;
}
```

---

## Tests Moulinette

```rust
// Atomic operations
#[test] fn test_atomic_load_store()      // 2.4.20.e-f
#[test] fn test_atomic_fetch_add()       // 2.4.20.g
#[test] fn test_atomic_fetch_sub()       // 2.4.20.h
#[test] fn test_cas_strong()             // 2.4.20.i
#[test] fn test_cas_weak()               // 2.4.20.j
#[test] fn test_atomic_counter()

// Memory ordering
#[test] fn test_seq_cst()                // 2.4.21.c
#[test] fn test_acquire_release()        // 2.4.21.d-e
#[test] fn test_acq_rel()                // 2.4.21.f
#[test] fn test_relaxed()                // 2.4.21.g
#[test] fn test_fences()                 // 2.4.21.i-j
#[test] fn test_ordering_correctness()

// Data structures
#[test] fn test_atomic_spinlock()
#[test] fn test_atomic_stack()
#[test] fn test_seqlock()
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Atomic types & ops (2.4.20.a-h) | 30 |
| CAS operations (2.4.20.i-j) | 20 |
| Memory ordering (2.4.21.a-h) | 30 |
| Fences (2.4.21.i-j) | 10 |
| Data structures | 10 |
| **Total** | **100** |

---

## Fichiers

```
ex06/
├── atomics.h
├── atomic_counter.c
├── memory_order.c
├── atomic_structures.c
├── benchmarks.c
└── Makefile
```
