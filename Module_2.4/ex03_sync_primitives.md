# ex03: Synchronization Primitives

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Difficile
**Duree**: 6h
**Score qualite**: 97/100

## Concepts Couverts

### 2.4.9: Semaphores (13 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Counting semaphore | Integer value |
| b | Binary semaphore | 0 or 1 |
| c | P operation | Wait/decrement |
| d | V operation | Signal/increment |
| e | sem_t | Type |
| f | sem_init() | Unnamed semaphore |
| g | sem_open() | Named semaphore |
| h | sem_wait() | P operation |
| i | sem_post() | V operation |
| j | sem_trywait() | Non-blocking |
| k | sem_getvalue() | Current value |
| l | sem_destroy() | Cleanup unnamed |
| m | sem_close/unlink() | Cleanup named |

### 2.4.10: Read-Write Locks (12 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Concept | Multiple readers OR one writer |
| b | Read lock | Shared access |
| c | Write lock | Exclusive access |
| d | pthread_rwlock_t | Type |
| e | pthread_rwlock_init() | Initialize |
| f | pthread_rwlock_rdlock() | Acquire read |
| g | pthread_rwlock_wrlock() | Acquire write |
| h | pthread_rwlock_unlock() | Release |
| i | pthread_rwlock_tryrdlock() | Non-blocking read |
| j | pthread_rwlock_trywrlock() | Non-blocking write |
| k | Writer preference | Avoid writer starvation |
| l | Reader preference | Avoid reader starvation |

### 2.4.11: Spinlocks (10 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Spinlock concept | Busy wait |
| b | When to use | Very short critical sections |
| c | pthread_spinlock_t | Type |
| d | pthread_spin_init() | Initialize |
| e | pthread_spin_lock() | Acquire (spins) |
| f | pthread_spin_unlock() | Release |
| g | pthread_spin_trylock() | Non-blocking |
| h | pthread_spin_destroy() | Cleanup |
| i | Disadvantage | Wastes CPU |
| j | Don't hold long | Bad for system |

### 2.4.12: Barriers (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Barrier concept | Wait for all threads |
| b | Synchronization point | All arrive before any proceed |
| c | pthread_barrier_t | Type |
| d | pthread_barrier_init() | Initialize with count |
| e | pthread_barrier_wait() | Wait at barrier |
| f | Serial thread | One returns special value |
| g | pthread_barrier_destroy() | Cleanup |
| h | Use case | Parallel phases |

---

## Sujet

Implementer une bibliotheque complete de primitives de synchronisation.

### Structures

```c
#include <pthread.h>
#include <semaphore.h>
#include <fcntl.h>

// 2.4.9: Semaphore wrapper
typedef struct {
    sem_t sem;                  // e: sem_t
    int initial_value;
    bool named;
    char name[256];             // For named semaphores
    uint64_t wait_count;
    uint64_t post_count;
} semaphore_t;

// 2.4.10: Read-Write Lock wrapper
typedef struct {
    pthread_rwlock_t rwlock;    // d: pthread_rwlock_t
    int preference;             // k,l: Writer or reader preference
    uint64_t read_locks;
    uint64_t write_locks;
    int active_readers;
    int active_writers;
    int waiting_readers;
    int waiting_writers;
} rwlock_t;

typedef enum {
    RWLOCK_PREFER_READER,       // l
    RWLOCK_PREFER_WRITER,       // k
    RWLOCK_PREFER_NONE
} rwlock_pref_t;

// 2.4.11: Spinlock wrapper
typedef struct {
    pthread_spinlock_t spin;    // c: pthread_spinlock_t
    uint64_t acquisitions;
    uint64_t spins;             // Track spinning iterations
    uint64_t total_spin_time;
} spinlock_t;

// 2.4.12: Barrier wrapper
typedef struct {
    pthread_barrier_t barrier;  // c: pthread_barrier_t
    int count;                  // Number of threads
    uint64_t generations;       // Times barrier was reset
    int serial_thread;          // f: Which thread got special value
} barrier_t;

// Resource pool using semaphore
typedef struct {
    void **resources;
    size_t capacity;
    semaphore_t available;      // Counting semaphore
    pthread_mutex_t mutex;
    bool *in_use;
} resource_pool_t;

// Read-write cache
typedef struct {
    void *data;
    size_t size;
    rwlock_t lock;
    uint64_t reads;
    uint64_t writes;
} rw_cache_t;
```

### API

```c
// 2.4.9: Semaphore API
int sem_create(semaphore_t *s, int initial);                 // f: Unnamed
int sem_create_named(semaphore_t *s, const char *name, int initial); // g: Named
int sem_delete(semaphore_t *s);                              // l,m: Cleanup
int sem_wait_p(semaphore_t *s);                              // c,h: P operation
int sem_post_v(semaphore_t *s);                              // d,i: V operation
int sem_trywait_p(semaphore_t *s);                           // j: Non-blocking
int sem_getval(semaphore_t *s, int *value);                  // k: Get value

// Binary semaphore helpers
int sem_create_binary(semaphore_t *s);                       // b: Binary (0 or 1)
int sem_acquire(semaphore_t *s);                             // Alias for wait
int sem_release(semaphore_t *s);                             // Alias for post

// 2.4.10: Read-Write Lock API
int rwlock_init(rwlock_t *rw, rwlock_pref_t pref);          // e: Initialize
int rwlock_destroy(rwlock_t *rw);
int rwlock_rdlock(rwlock_t *rw);                             // b,f: Read lock
int rwlock_wrlock(rwlock_t *rw);                             // c,g: Write lock
int rwlock_unlock(rwlock_t *rw);                             // h: Release
int rwlock_tryrdlock(rwlock_t *rw);                          // i: Try read
int rwlock_trywrlock(rwlock_t *rw);                          // j: Try write

// Read-write lock stats
void rwlock_get_stats(rwlock_t *rw, int *readers, int *writers,
                      int *waiting_r, int *waiting_w);

// 2.4.11: Spinlock API
int spinlock_init(spinlock_t *s);                            // d: Initialize
int spinlock_destroy(spinlock_t *s);                         // h: Cleanup
int spinlock_lock(spinlock_t *s);                            // e: Acquire (spins)
int spinlock_unlock(spinlock_t *s);                          // f: Release
int spinlock_trylock(spinlock_t *s);                         // g: Non-blocking

// 2.4.12: Barrier API
int barrier_init(barrier_t *b, int count);                   // d: Initialize
int barrier_destroy(barrier_t *b);                           // g: Cleanup
int barrier_wait(barrier_t *b);                              // e: Wait (returns SERIAL for one)

// f: Check if this thread is the serial thread
#define BARRIER_SERIAL_THREAD PTHREAD_BARRIER_SERIAL_THREAD
bool barrier_is_serial(int result);

// Resource pool using semaphore
int pool_init(resource_pool_t *p, size_t capacity);
void pool_destroy(resource_pool_t *p);
void *pool_acquire(resource_pool_t *p);                      // Blocks if empty
void *pool_try_acquire(resource_pool_t *p);                  // Non-blocking
void pool_release(resource_pool_t *p, void *resource);

// Read-write cache
int rw_cache_init(rw_cache_t *c, size_t size, rwlock_pref_t pref);
void rw_cache_destroy(rw_cache_t *c);
int rw_cache_read(rw_cache_t *c, void *buf, size_t offset, size_t len);
int rw_cache_write(rw_cache_t *c, const void *buf, size_t offset, size_t len);

// Comparison benchmark
typedef struct {
    double mutex_time_us;
    double rwlock_read_time_us;
    double rwlock_write_time_us;
    double spinlock_time_us;
    double semaphore_time_us;
} sync_benchmark_t;

void benchmark_primitives(int threads, int ops, sync_benchmark_t *result);
```

---

## Exemple

```c
#include "sync_primitives.h"

// 2.4.9: Semaphore for resource limiting
semaphore_t connection_limit;
#define MAX_CONNECTIONS 10

void *client_handler(void *arg) {
    int id = *(int*)arg;

    // 2.4.9.c,h: P operation - wait for available slot
    sem_wait_p(&connection_limit);
    printf("Client %d: connected (slot acquired)\n", id);

    // Simulate work
    usleep(rand() % 100000);

    // 2.4.9.d,i: V operation - release slot
    sem_post_v(&connection_limit);
    printf("Client %d: disconnected (slot released)\n", id);

    return NULL;
}

// 2.4.10: Read-write lock for shared data
rwlock_t data_lock;
int shared_data[1000];

void *reader(void *arg) {
    int id = *(int*)arg;
    for (int i = 0; i < 100; i++) {
        // 2.4.10.b,f: Multiple readers can proceed
        rwlock_rdlock(&data_lock);
        int sum = 0;
        for (int j = 0; j < 1000; j++) sum += shared_data[j];
        rwlock_unlock(&data_lock);
        (void)sum;
    }
    return NULL;
}

void *writer(void *arg) {
    int id = *(int*)arg;
    for (int i = 0; i < 10; i++) {
        // 2.4.10.c,g: Exclusive write access
        rwlock_wrlock(&data_lock);
        for (int j = 0; j < 1000; j++) shared_data[j]++;
        rwlock_unlock(&data_lock);
        usleep(1000);
    }
    return NULL;
}

// 2.4.11: Spinlock for very short critical sections
spinlock_t fast_lock;
volatile int fast_counter = 0;

void *fast_incrementer(void *arg) {
    for (int i = 0; i < 100000; i++) {
        // 2.4.11.a-b: Busy wait, but very brief
        spinlock_lock(&fast_lock);
        fast_counter++;
        spinlock_unlock(&fast_lock);
    }
    return NULL;
}

// 2.4.12: Barrier for parallel phases
barrier_t phase_barrier;
#define NUM_WORKERS 4

void *phased_worker(void *arg) {
    int id = *(int*)arg;

    // Phase 1: Initialize
    printf("Worker %d: Phase 1 starting\n", id);
    usleep(rand() % 10000);

    // 2.4.12.a-b,e: All threads wait here
    int result = barrier_wait(&phase_barrier);

    // 2.4.12.f: One thread gets serial role
    if (barrier_is_serial(result)) {
        printf("Worker %d: I'm the serial thread, doing summary\n", id);
    }

    // Phase 2: Process
    printf("Worker %d: Phase 2 starting\n", id);
    usleep(rand() % 10000);

    barrier_wait(&phase_barrier);

    // Phase 3: Cleanup
    printf("Worker %d: Phase 3 (cleanup)\n", id);

    return NULL;
}

int main(void) {
    // 2.4.9: Semaphore demo
    printf("=== Semaphore Demo ===\n");
    sem_create(&connection_limit, MAX_CONNECTIONS);  // a: Counting semaphore

    pthread_t clients[20];
    int client_ids[20];
    for (int i = 0; i < 20; i++) {
        client_ids[i] = i;
        pthread_create(&clients[i], NULL, client_handler, &client_ids[i]);
    }
    for (int i = 0; i < 20; i++) {
        pthread_join(clients[i], NULL);
    }
    sem_delete(&connection_limit);

    // 2.4.9.g: Named semaphore (IPC)
    semaphore_t named_sem;
    sem_create_named(&named_sem, "/my_semaphore", 1);
    sem_wait_p(&named_sem);
    // ... critical section ...
    sem_post_v(&named_sem);
    sem_delete(&named_sem);

    // 2.4.10: Read-write lock demo
    printf("\n=== RW Lock Demo ===\n");
    rwlock_init(&data_lock, RWLOCK_PREFER_WRITER);  // k: Avoid writer starvation

    pthread_t readers[8], writers[2];
    int rids[8], wids[2];
    for (int i = 0; i < 8; i++) {
        rids[i] = i;
        pthread_create(&readers[i], NULL, reader, &rids[i]);
    }
    for (int i = 0; i < 2; i++) {
        wids[i] = i;
        pthread_create(&writers[i], NULL, writer, &wids[i]);
    }
    for (int i = 0; i < 8; i++) pthread_join(readers[i], NULL);
    for (int i = 0; i < 2; i++) pthread_join(writers[i], NULL);

    int r, w, wr, ww;
    rwlock_get_stats(&data_lock, &r, &w, &wr, &ww);
    printf("Read locks: %d, Write locks: %d\n", r, w);
    rwlock_destroy(&data_lock);

    // 2.4.11: Spinlock demo
    printf("\n=== Spinlock Demo ===\n");
    spinlock_init(&fast_lock);

    pthread_t spinners[4];
    for (int i = 0; i < 4; i++) {
        pthread_create(&spinners[i], NULL, fast_incrementer, NULL);
    }
    for (int i = 0; i < 4; i++) {
        pthread_join(spinners[i], NULL);
    }
    printf("Fast counter: %d (expected 400000)\n", fast_counter);
    spinlock_destroy(&fast_lock);

    // 2.4.12: Barrier demo
    printf("\n=== Barrier Demo ===\n");
    barrier_init(&phase_barrier, NUM_WORKERS);

    pthread_t workers[NUM_WORKERS];
    int worker_ids[NUM_WORKERS];
    for (int i = 0; i < NUM_WORKERS; i++) {
        worker_ids[i] = i;
        pthread_create(&workers[i], NULL, phased_worker, &worker_ids[i]);
    }
    for (int i = 0; i < NUM_WORKERS; i++) {
        pthread_join(workers[i], NULL);
    }
    barrier_destroy(&phase_barrier);

    // Benchmark comparison
    printf("\n=== Benchmark ===\n");
    sync_benchmark_t bench;
    benchmark_primitives(4, 100000, &bench);
    printf("Mutex: %.2f us, RWLock(R): %.2f us, Spinlock: %.2f us\n",
           bench.mutex_time_us, bench.rwlock_read_time_us, bench.spinlock_time_us);

    return 0;
}
```

---

## Tests Moulinette

```rust
// Semaphore tests
#[test] fn test_sem_counting()           // 2.4.9.a
#[test] fn test_sem_binary()             // 2.4.9.b
#[test] fn test_sem_pv_operations()      // 2.4.9.c-d
#[test] fn test_sem_wait_post()          // 2.4.9.h-i
#[test] fn test_sem_trywait()            // 2.4.9.j
#[test] fn test_sem_getvalue()           // 2.4.9.k
#[test] fn test_sem_named()              // 2.4.9.g,m

// RW Lock tests
#[test] fn test_rwlock_readers()         // 2.4.10.a-b,f
#[test] fn test_rwlock_writer()          // 2.4.10.c,g
#[test] fn test_rwlock_try()             // 2.4.10.i-j
#[test] fn test_rwlock_writer_pref()     // 2.4.10.k
#[test] fn test_rwlock_reader_pref()     // 2.4.10.l

// Spinlock tests
#[test] fn test_spinlock_basic()         // 2.4.11.a-f
#[test] fn test_spinlock_trylock()       // 2.4.11.g
#[test] fn test_spinlock_short_cs()      // 2.4.11.b,j

// Barrier tests
#[test] fn test_barrier_wait()           // 2.4.12.a-b,e
#[test] fn test_barrier_serial()         // 2.4.12.f
#[test] fn test_barrier_phases()         // 2.4.12.h

// Integration tests
#[test] fn test_resource_pool()
#[test] fn test_rw_cache()
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Semaphores (2.4.9) | 30 |
| Read-Write Locks (2.4.10) | 30 |
| Spinlocks (2.4.11) | 20 |
| Barriers (2.4.12) | 20 |
| **Total** | **100** |

---

## Fichiers

```
ex03/
├── sync_primitives.h
├── semaphore.c
├── rwlock.c
├── spinlock.c
├── barrier.c
├── resource_pool.c
└── Makefile
```
