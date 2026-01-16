# ex01: Mutex & Race Conditions

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Moyen
**Duree**: 5h
**Score qualite**: 96/100

## Concepts Couverts

### 2.4.5: Race Conditions (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Race condition | Result depends on timing |
| b | Data race | Concurrent unsynchronized access |
| c | Critical section | Code accessing shared data |
| d | Example | counter++ |
| e | Read-modify-write | Three operations |
| f | Interleaving | Operations mix |
| g | Non-determinism | Different runs, different results |
| h | Detection | ThreadSanitizer |

### 2.4.6: Mutex (11 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Mutex concept | Mutual exclusion |
| b | Critical section | Protected region |
| c | pthread_mutex_t | Type |
| d | PTHREAD_MUTEX_INITIALIZER | Static init |
| e | pthread_mutex_init() | Dynamic init |
| f | pthread_mutex_lock() | Acquire |
| g | pthread_mutex_unlock() | Release |
| h | pthread_mutex_trylock() | Non-blocking |
| i | pthread_mutex_timedlock() | With timeout |
| j | pthread_mutex_destroy() | Cleanup |
| k | Mutex types | Normal, recursive, errorcheck |

### 2.4.7: Mutex Best Practices (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | RAII pattern | Lock in constructor |
| b | Minimize scope | Hold briefly |
| c | Lock ordering | Prevent deadlock |
| d | Don't call unknown | While holding lock |
| e | Error checking | Check return values |
| f | Recursive mutex | Same thread can relock |
| g | Performance | Contention overhead |

---

## Sujet

Implementer une bibliotheque de synchronisation demontrant les race conditions et leur resolution avec mutex.

### Structures

```c
#include <pthread.h>
#include <stdbool.h>
#include <time.h>

// 2.4.5: Race condition demonstration
typedef struct {
    int value;                  // Shared counter
    uint64_t increments;
    uint64_t expected;
    uint64_t actual;
    bool race_detected;         // g: Non-determinism
} race_demo_t;

// 2.4.6: Mutex wrapper
typedef struct {
    pthread_mutex_t mutex;
    int type;                   // k: Mutex type
    bool initialized;
    pthread_t owner;            // For debugging
    uint64_t lock_count;
    uint64_t contention_count;
    struct timespec total_wait_time;
} mutex_t;

// 2.4.6.k: Mutex types
typedef enum {
    MUTEX_NORMAL,               // Default
    MUTEX_RECURSIVE,            // f: Can relock
    MUTEX_ERRORCHECK            // Debug mode
} mutex_type_t;

// 2.4.7.a: RAII-style lock guard
typedef struct {
    mutex_t *mutex;
    bool locked;
} lock_guard_t;

// Thread-safe counter using mutex
typedef struct {
    int64_t value;
    mutex_t lock;
    uint64_t reads;
    uint64_t writes;
} safe_counter_t;

// Sync manager
typedef struct {
    mutex_t *mutexes;
    size_t mutex_count;
    size_t capacity;
    uint64_t total_locks;
    uint64_t total_contentions;
} sync_manager_t;
```

### API

```c
// Manager
sync_manager_t *sync_manager_create(void);
void sync_manager_destroy(sync_manager_t *mgr);

// 2.4.5: Race condition demonstration
void race_demo_init(race_demo_t *demo);
void race_demo_run_unsafe(race_demo_t *demo, int num_threads, int increments);
void race_demo_run_safe(race_demo_t *demo, int num_threads, int increments, mutex_t *lock);
void race_demo_show_results(race_demo_t *demo);
bool race_demo_detect_tsan(void);  // h: ThreadSanitizer

// 2.4.6: Mutex API
int mutex_init(mutex_t *m, mutex_type_t type);           // e: Dynamic init
int mutex_init_static(mutex_t *m);                       // d: Static-like init
int mutex_destroy(mutex_t *m);                           // j: Cleanup
int mutex_lock(mutex_t *m);                              // f: Acquire
int mutex_unlock(mutex_t *m);                            // g: Release
int mutex_trylock(mutex_t *m);                           // h: Non-blocking
int mutex_timedlock(mutex_t *m, const struct timespec *timeout); // i: Timeout

// 2.4.6.k: Mutex types
int mutex_set_type(mutex_t *m, mutex_type_t type);
mutex_type_t mutex_get_type(mutex_t *m);

// 2.4.7.a: RAII lock guard
void lock_guard_init(lock_guard_t *guard, mutex_t *m);
void lock_guard_destroy(lock_guard_t *guard);

// 2.4.7.b: Scoped locking macro
#define SCOPED_LOCK(mutex) \
    for (lock_guard_t _guard = {mutex, false}; \
         !_guard.locked && (mutex_lock(_guard.mutex), _guard.locked = true); \
         mutex_unlock(_guard.mutex), _guard.locked = false)

// 2.4.7.c: Lock ordering helper
typedef struct {
    mutex_t **mutexes;
    size_t count;
    int *order;  // Global order for deadlock prevention
} lock_order_t;

int lock_order_init(lock_order_t *lo, mutex_t **mutexes, size_t count);
int lock_order_acquire_all(lock_order_t *lo);
int lock_order_release_all(lock_order_t *lo);

// Thread-safe counter
int safe_counter_init(safe_counter_t *c);
void safe_counter_destroy(safe_counter_t *c);
int64_t safe_counter_increment(safe_counter_t *c);
int64_t safe_counter_decrement(safe_counter_t *c);
int64_t safe_counter_add(safe_counter_t *c, int64_t delta);
int64_t safe_counter_get(safe_counter_t *c);

// 2.4.5.c: Critical section analyzer
typedef struct {
    const char *name;
    uint64_t entry_count;
    uint64_t total_time_ns;
    uint64_t max_time_ns;
    double avg_time_ns;
} critical_section_stats_t;

void cs_begin(const char *name);
void cs_end(const char *name);
void cs_get_stats(const char *name, critical_section_stats_t *stats);

// Statistics
typedef struct {
    uint64_t total_locks;
    uint64_t total_unlocks;
    uint64_t contentions;
    uint64_t trylock_failures;
    uint64_t timedlock_timeouts;
    double avg_hold_time_us;
} mutex_stats_t;

void sync_get_stats(sync_manager_t *mgr, mutex_stats_t *stats);
```

---

## Exemple

```c
#include "mutex_sync.h"

// 2.4.5.c: Shared data (critical section)
int shared_counter = 0;
mutex_t counter_lock;

void *unsafe_worker(void *arg) {
    int iterations = *(int*)arg;
    for (int i = 0; i < iterations; i++) {
        // 2.4.5.d-e: Read-modify-write race
        shared_counter++;  // NOT ATOMIC!
    }
    return NULL;
}

void *safe_worker(void *arg) {
    int iterations = *(int*)arg;
    for (int i = 0; i < iterations; i++) {
        // 2.4.6.f-g: Protected by mutex
        mutex_lock(&counter_lock);
        shared_counter++;
        mutex_unlock(&counter_lock);
    }
    return NULL;
}

int main(void) {
    sync_manager_t *mgr = sync_manager_create();

    // 2.4.5: Demonstrate race condition
    printf("=== Race Condition Demo ===\n");
    race_demo_t demo;
    race_demo_init(&demo);

    // Without synchronization (2.4.5.a-g)
    race_demo_run_unsafe(&demo, 4, 100000);
    race_demo_show_results(&demo);
    // Expected: 400000, Actual: varies (race!)

    // 2.4.6: With mutex
    mutex_init(&counter_lock, MUTEX_NORMAL);

    printf("\n=== Safe Counter Demo ===\n");
    shared_counter = 0;
    race_demo_run_safe(&demo, 4, 100000, &counter_lock);
    race_demo_show_results(&demo);
    // Expected: 400000, Actual: 400000

    // 2.4.6.k: Different mutex types
    mutex_t recursive_lock;
    mutex_init(&recursive_lock, MUTEX_RECURSIVE);

    // 2.4.7.f: Recursive mutex allows re-locking
    mutex_lock(&recursive_lock);
    mutex_lock(&recursive_lock);  // OK with recursive
    mutex_unlock(&recursive_lock);
    mutex_unlock(&recursive_lock);

    // 2.4.6.h: Try-lock pattern
    if (mutex_trylock(&counter_lock) == 0) {
        shared_counter++;
        mutex_unlock(&counter_lock);
    } else {
        printf("Lock busy, skipping\n");
    }

    // 2.4.6.i: Timed lock
    struct timespec timeout = {.tv_sec = 0, .tv_nsec = 100000000}; // 100ms
    if (mutex_timedlock(&counter_lock, &timeout) == 0) {
        shared_counter++;
        mutex_unlock(&counter_lock);
    } else {
        printf("Timeout waiting for lock\n");
    }

    // 2.4.7.a: RAII-style with scoped lock
    {
        SCOPED_LOCK(&counter_lock) {
            shared_counter += 10;
            // Auto-unlock at end of scope
        }
    }

    // 2.4.7.c: Lock ordering to prevent deadlock
    mutex_t lock_a, lock_b;
    mutex_init(&lock_a, MUTEX_NORMAL);
    mutex_init(&lock_b, MUTEX_NORMAL);

    mutex_t *locks[] = {&lock_a, &lock_b};
    lock_order_t order;
    lock_order_init(&order, locks, 2);

    // Always acquire in same order
    lock_order_acquire_all(&order);
    // ... critical section with both locks ...
    lock_order_release_all(&order);

    // 2.4.7.b: Thread-safe counter wrapper
    safe_counter_t safe_cnt;
    safe_counter_init(&safe_cnt);

    for (int i = 0; i < 1000; i++) {
        safe_counter_increment(&safe_cnt);
    }
    printf("Safe counter: %ld\n", safe_counter_get(&safe_cnt));

    // Stats
    mutex_stats_t stats;
    sync_get_stats(mgr, &stats);
    printf("Locks: %lu, Contentions: %lu\n", stats.total_locks, stats.contentions);

    mutex_destroy(&counter_lock);
    mutex_destroy(&recursive_lock);
    mutex_destroy(&lock_a);
    mutex_destroy(&lock_b);
    safe_counter_destroy(&safe_cnt);
    sync_manager_destroy(mgr);

    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_race_condition()         // 2.4.5.a
#[test] fn test_data_race()              // 2.4.5.b
#[test] fn test_critical_section()       // 2.4.5.c
#[test] fn test_read_modify_write()      // 2.4.5.d-e
#[test] fn test_interleaving()           // 2.4.5.f
#[test] fn test_non_determinism()        // 2.4.5.g
#[test] fn test_mutex_init()             // 2.4.6.d-e
#[test] fn test_mutex_lock_unlock()      // 2.4.6.f-g
#[test] fn test_mutex_trylock()          // 2.4.6.h
#[test] fn test_mutex_timedlock()        // 2.4.6.i
#[test] fn test_mutex_types()            // 2.4.6.k
#[test] fn test_recursive_mutex()        // 2.4.7.f
#[test] fn test_lock_guard()             // 2.4.7.a
#[test] fn test_lock_ordering()          // 2.4.7.c
#[test] fn test_safe_counter()
#[test] fn test_contention()             // 2.4.7.g
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Race conditions demo (2.4.5) | 30 |
| Mutex API (2.4.6) | 40 |
| Best practices (2.4.7) | 30 |
| **Total** | **100** |

---

## Fichiers

```
ex01/
├── mutex_sync.h
├── mutex_sync.c
├── race_demo.c
├── lock_guard.c
├── safe_counter.c
└── Makefile
```
