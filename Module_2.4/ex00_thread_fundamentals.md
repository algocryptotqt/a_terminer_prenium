# ex00: Thread Fundamentals

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Moyen
**Duree**: 6h
**Score qualite**: 96/100

## Concepts Couverts

### 2.4.1: Thread Concepts (11 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Thread | Lightweight execution unit |
| b | Process vs thread | Address space sharing |
| c | Shared | Code, data, heap, files |
| d | Private | Stack, registers, TID |
| e | Benefits | Responsiveness, resource sharing |
| f | Challenges | Synchronization, debugging |
| g | User threads | Library-level |
| h | Kernel threads | OS-level |
| i | Many-to-one | User threads to kernel |
| j | One-to-one | Thread = kernel thread |
| k | Many-to-many | Multiplexed |

### 2.4.2: POSIX Threads (12 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | pthread.h | Header |
| b | -pthread | Compile flag |
| c | pthread_t | Thread identifier |
| d | pthread_create() | Create thread |
| e | Start function | void* (*)(void*) |
| f | Argument | void* passed |
| g | pthread_exit() | Exit thread |
| h | Return value | void* |
| i | pthread_join() | Wait for thread |
| j | pthread_detach() | Don't need join |
| k | pthread_self() | Get own ID |
| l | pthread_equal() | Compare IDs |

### 2.4.3: Thread Attributes (10 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | pthread_attr_t | Attributes structure |
| b | pthread_attr_init() | Initialize |
| c | Detach state | Joinable or detached |
| d | Stack size | Custom size |
| e | Stack address | Custom location |
| f | Guard size | Overflow protection |
| g | Scheduling policy | SCHED_* |
| h | Priority | Scheduling priority |
| i | Scope | System or process |
| j | pthread_attr_destroy() | Cleanup |

### 2.4.4: Thread-Local Storage (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | TLS concept | Per-thread data |
| b | __thread keyword | GCC extension |
| c | thread_local | C11 standard |
| d | pthread_key_create() | Create key |
| e | pthread_getspecific() | Get value |
| f | pthread_setspecific() | Set value |
| g | Destructor | Called on thread exit |
| h | Use cases | errno, allocator state |

---

## Sujet

Implementer une bibliotheque de gestion de threads avec support complet des attributs et TLS.

### Structures

```c
#include <pthread.h>
#include <stdint.h>
#include <stdbool.h>

// 2.4.1.a-d: Thread info
typedef struct {
    pthread_t handle;           // c: Thread identifier
    int id;                     // Internal ID
    char name[64];              // Thread name
    void *(*start_routine)(void*); // e: Start function
    void *arg;                  // f: Argument
    void *result;               // h: Return value
    bool joinable;              // Joinable state
    bool running;
    bool joined;
    uint64_t start_time;
    uint64_t end_time;
} thread_info_t;

// 2.4.3: Thread attributes wrapper
typedef struct {
    pthread_attr_t attr;
    size_t stack_size;          // d: Stack size
    void *stack_addr;           // e: Stack address
    size_t guard_size;          // f: Guard size
    int detach_state;           // c: Detach state
    int sched_policy;           // g: Scheduling policy
    int priority;               // h: Priority
    int scope;                  // i: Scope
} thread_attr_t;

// 2.4.4: TLS key wrapper
typedef struct {
    pthread_key_t key;
    void (*destructor)(void*);  // g: Destructor
    const char *name;
    bool initialized;
} tls_key_t;

// Thread pool context
typedef struct {
    thread_info_t *threads;
    size_t thread_count;
    size_t capacity;
    tls_key_t *tls_keys;
    size_t tls_count;
    size_t tls_capacity;
    uint64_t total_created;
    uint64_t total_joined;
} thread_manager_t;
```

### API

```c
// Manager lifecycle
thread_manager_t *thread_manager_create(void);
void thread_manager_destroy(thread_manager_t *mgr);

// 2.4.2: POSIX Threads API
int tm_create(thread_manager_t *mgr, thread_info_t **info,
              void *(*start)(void*), void *arg);              // d: pthread_create
int tm_create_with_attr(thread_manager_t *mgr, thread_info_t **info,
                        thread_attr_t *attr, void *(*start)(void*), void *arg);
int tm_join(thread_manager_t *mgr, thread_info_t *info, void **result); // i: pthread_join
int tm_detach(thread_manager_t *mgr, thread_info_t *info);   // j: pthread_detach
pthread_t tm_self(void);                                      // k: pthread_self
bool tm_equal(pthread_t t1, pthread_t t2);                   // l: pthread_equal
void tm_exit(void *result);                                   // g: pthread_exit

// 2.4.3: Thread Attributes
int tm_attr_init(thread_attr_t *attr);                       // b: Initialize
int tm_attr_destroy(thread_attr_t *attr);                    // j: Cleanup
int tm_attr_set_detach(thread_attr_t *attr, int state);      // c: Detach state
int tm_attr_set_stack_size(thread_attr_t *attr, size_t size);// d: Stack size
int tm_attr_set_stack(thread_attr_t *attr, void *addr, size_t size); // e: Stack
int tm_attr_set_guard_size(thread_attr_t *attr, size_t size);// f: Guard size
int tm_attr_set_sched_policy(thread_attr_t *attr, int policy);// g: Policy
int tm_attr_set_priority(thread_attr_t *attr, int priority); // h: Priority
int tm_attr_set_scope(thread_attr_t *attr, int scope);       // i: Scope

// Get attributes
int tm_attr_get_detach(thread_attr_t *attr, int *state);
int tm_attr_get_stack_size(thread_attr_t *attr, size_t *size);
int tm_attr_get_guard_size(thread_attr_t *attr, size_t *size);

// 2.4.4: Thread-Local Storage
int tm_tls_create(thread_manager_t *mgr, tls_key_t **key,
                  void (*destructor)(void*));                 // d: Create key
int tm_tls_delete(thread_manager_t *mgr, tls_key_t *key);
void *tm_tls_get(tls_key_t *key);                            // e: Get value
int tm_tls_set(tls_key_t *key, void *value);                 // f: Set value

// 2.4.4.b-c: C11/GCC TLS demonstration
void tm_demonstrate_thread_local(void);

// 2.4.1.e-f: Benefits and challenges demo
typedef struct {
    double sequential_time;
    double parallel_time;
    double speedup;
    int thread_count;
} benchmark_result_t;

void tm_benchmark_parallel(int num_threads, benchmark_result_t *result);

// 2.4.1.g-k: Threading models demonstration
typedef enum {
    MODEL_USER_THREADS,      // g: User-level
    MODEL_KERNEL_THREADS,    // h: Kernel-level
    MODEL_MANY_TO_ONE,       // i: M:1
    MODEL_ONE_TO_ONE,        // j: 1:1
    MODEL_MANY_TO_MANY       // k: M:N
} threading_model_t;

void tm_explain_model(threading_model_t model);
threading_model_t tm_detect_system_model(void);

// Utility functions
void tm_set_name(thread_info_t *info, const char *name);
const char *tm_get_name(thread_info_t *info);
void tm_list_threads(thread_manager_t *mgr);
int tm_get_thread_count(thread_manager_t *mgr);

// Statistics
typedef struct {
    uint64_t threads_created;
    uint64_t threads_joined;
    uint64_t threads_detached;
    uint64_t tls_keys_created;
    double avg_thread_lifetime_ms;
} thread_stats_t;

void tm_get_stats(thread_manager_t *mgr, thread_stats_t *stats);
```

---

## Exemple

```c
#include "thread_mgr.h"

// 2.4.4.b: GCC __thread
__thread int thread_local_counter = 0;

// 2.4.4.c: C11 thread_local
thread_local int c11_thread_local = 0;

void *worker(void *arg) {
    int id = *(int*)arg;

    // 2.4.1.d: Each thread has private stack
    int local_var = id * 100;

    // 2.4.4: TLS demonstration
    thread_local_counter = id;
    printf("Thread %d: TLS counter = %d\n", id, thread_local_counter);

    // Do some work
    for (int i = 0; i < 1000000; i++) {
        local_var++;
    }

    // 2.4.2.g-h: Exit with return value
    int *result = malloc(sizeof(int));
    *result = local_var;
    return result;
}

int main(void) {
    thread_manager_t *mgr = thread_manager_create();

    // 2.4.1.g-k: Detect threading model
    threading_model_t model = tm_detect_system_model();
    tm_explain_model(model);

    // 2.4.3: Create custom attributes
    thread_attr_t attr;
    tm_attr_init(&attr);
    tm_attr_set_stack_size(&attr, 2 * 1024 * 1024);  // d: 2MB stack
    tm_attr_set_guard_size(&attr, 4096);              // f: 4KB guard
    tm_attr_set_detach(&attr, PTHREAD_CREATE_JOINABLE); // c: Joinable

    // 2.4.2.d: Create threads
    thread_info_t *threads[4];
    int ids[4] = {1, 2, 3, 4};

    for (int i = 0; i < 4; i++) {
        tm_create_with_attr(mgr, &threads[i], &attr, worker, &ids[i]);
        tm_set_name(threads[i], "worker");
    }

    // 2.4.2.k-l: Self and equal
    printf("Main thread: %lu\n", (unsigned long)tm_self());

    // 2.4.2.i: Join threads
    for (int i = 0; i < 4; i++) {
        void *result;
        tm_join(mgr, threads[i], &result);
        printf("Thread %d returned: %d\n", i, *(int*)result);
        free(result);
    }

    // 2.4.4.d-g: Custom TLS with destructor
    tls_key_t *my_key;
    tm_tls_create(mgr, &my_key, free);  // g: Auto-free on thread exit

    // 2.4.1.e: Benchmark parallel benefits
    benchmark_result_t bench;
    tm_benchmark_parallel(4, &bench);
    printf("Speedup with 4 threads: %.2fx\n", bench.speedup);

    // Stats
    thread_stats_t stats;
    tm_get_stats(mgr, &stats);
    printf("Total threads: %lu, Avg lifetime: %.2fms\n",
           stats.threads_created, stats.avg_thread_lifetime_ms);

    tm_attr_destroy(&attr);
    thread_manager_destroy(mgr);
    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_thread_concept()         // 2.4.1.a
#[test] fn test_process_vs_thread()      // 2.4.1.b
#[test] fn test_shared_private()         // 2.4.1.c-d
#[test] fn test_threading_models()       // 2.4.1.g-k
#[test] fn test_pthread_create()         // 2.4.2.d
#[test] fn test_pthread_join()           // 2.4.2.i
#[test] fn test_pthread_detach()         // 2.4.2.j
#[test] fn test_pthread_self_equal()     // 2.4.2.k-l
#[test] fn test_attr_init_destroy()      // 2.4.3.a-b,j
#[test] fn test_attr_detach_state()      // 2.4.3.c
#[test] fn test_attr_stack()             // 2.4.3.d-f
#[test] fn test_attr_scheduling()        // 2.4.3.g-i
#[test] fn test_tls_key()                // 2.4.4.d-f
#[test] fn test_tls_destructor()         // 2.4.4.g
#[test] fn test_thread_local_keyword()   // 2.4.4.b-c
#[test] fn test_parallel_speedup()       // 2.4.1.e
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Thread concepts (2.4.1) | 20 |
| POSIX threads API (2.4.2) | 30 |
| Thread attributes (2.4.3) | 25 |
| Thread-local storage (2.4.4) | 25 |
| **Total** | **100** |

---

## Fichiers

```
ex00/
├── thread_mgr.h
├── thread_mgr.c
├── thread_attr.c
├── thread_tls.c
├── thread_bench.c
└── Makefile
```

## Compilation

```makefile
CFLAGS = -Wall -Wextra -std=c17 -pthread
LDFLAGS = -pthread

all: libthread_mgr.a demo

libthread_mgr.a: thread_mgr.o thread_attr.o thread_tls.o thread_bench.o
	ar rcs $@ $^

demo: demo.c libthread_mgr.a
	$(CC) $(CFLAGS) -o $@ $< -L. -lthread_mgr $(LDFLAGS)
```
