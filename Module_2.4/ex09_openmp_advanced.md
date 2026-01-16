# ex09: OpenMP, Cache Effects & Async

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Difficile
**Duree**: 6h
**Score qualite**: 96/100

## Concepts Couverts

### 2.4.28: OpenMP Basics (10 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | OpenMP | Pragma-based parallelism |
| b | #pragma omp parallel | Parallel region |
| c | #pragma omp for | Parallel loop |
| d | #pragma omp parallel for | Combined |
| e | private/shared | Variable scope |
| f | reduction | Combine results |
| g | schedule | Loop scheduling |
| h | critical | Mutual exclusion |
| i | atomic | Atomic update |
| j | barrier | Synchronization |

### 2.4.29: Cache Effects (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Cache coherency | Consistent view |
| b | MESI protocol | Modified, Exclusive, Shared, Invalid |
| c | Cache line | 64 bytes |
| d | False sharing | Different data, same line |
| e | Ping-pong | Invalidations |
| f | Detection | Performance counters |
| g | Solution | Padding |
| h | alignas | Force alignment |

### 2.4.30: Async Programming (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Asynchronous | Non-blocking |
| b | Callback | Called on completion |
| c | Future/Promise | Async result |
| d | Event loop | Single-threaded async |
| e | Coroutines | Cooperative multitasking |
| f | C++ coroutines | co_await, co_yield |
| g | Comparison | Threads vs async |

---

## Sujet

Explorer OpenMP, les effets de cache en multi-threading, et la programmation asynchrone.

### Structures

```c
#include <omp.h>
#include <stdalign.h>
#include <stdbool.h>

// 2.4.29.c-d,g-h: Cache-aligned counter (avoid false sharing)
#define CACHE_LINE_SIZE 64

typedef struct {
    alignas(CACHE_LINE_SIZE) int64_t value;  // h: Force alignment
    char padding[CACHE_LINE_SIZE - sizeof(int64_t)]; // g: Padding
} padded_counter_t;

// Unpadded for comparison (causes false sharing)
typedef struct {
    int64_t value;  // d: May share cache line
} unpadded_counter_t;

// 2.4.29.a-b: Cache state tracking
typedef struct {
    uint64_t invalidations;
    uint64_t shared_hits;
    uint64_t exclusive_hits;
    uint64_t modified_writebacks;
} cache_stats_t;

// 2.4.30.b: Callback structure
typedef void (*callback_t)(void *result, void *context);

typedef struct {
    callback_t on_complete;
    callback_t on_error;
    void *context;
} async_callbacks_t;

// 2.4.30.c: Future/Promise
typedef struct {
    void *result;
    int error;
    bool completed;
    pthread_mutex_t mutex;
    pthread_cond_t cond;
    async_callbacks_t callbacks;
} promise_t;

typedef struct {
    promise_t *promise;
} future_t;

// 2.4.30.d: Event loop
typedef struct event {
    void (*handler)(void *arg);
    void *arg;
    struct event *next;
} event_t;

typedef struct {
    event_t *head;
    event_t *tail;
    pthread_mutex_t mutex;
    pthread_cond_t cond;
    bool running;
    int pending;
} event_loop_t;

// 2.4.30.e: Simple coroutine (using setjmp/longjmp)
#include <setjmp.h>

typedef struct coro {
    jmp_buf context;
    void *stack;
    size_t stack_size;
    void (*func)(struct coro *self, void *arg);
    void *arg;
    void *yield_value;
    bool finished;
    struct coro *caller;
} coroutine_t;
```

### API

```c
// ============== OPENMP WRAPPERS ==============
// 2.4.28

// 2.4.28.b: Parallel region info
int omp_get_num_threads_wrapper(void);
int omp_get_thread_num_wrapper(void);
void omp_set_num_threads_wrapper(int n);

// 2.4.28.c-d: Parallel for demonstrations
void demo_parallel_for(int *arr, int n);
void demo_parallel_for_combined(int *arr, int n);

// 2.4.28.e: Variable scope
void demo_private_shared(void);

// 2.4.28.f: Reduction
int64_t demo_reduction_sum(int *arr, int n);
int64_t demo_reduction_product(int *arr, int n);
int demo_reduction_max(int *arr, int n);

// 2.4.28.g: Schedule types
typedef enum {
    SCHED_STATIC,       // Equal chunks
    SCHED_DYNAMIC,      // On-demand chunks
    SCHED_GUIDED,       // Decreasing chunks
    SCHED_RUNTIME       // Environment variable
} omp_schedule_t;

void demo_schedule(int *arr, int n, omp_schedule_t sched, int chunk);

// 2.4.28.h-i: Synchronization
void demo_critical(int *shared_counter, int n);
void demo_atomic(int *shared_counter, int n);

// 2.4.28.j: Barrier
void demo_barrier(void);

// ============== CACHE EFFECTS ==============
// 2.4.29

// 2.4.29.d-e: False sharing demonstration
typedef struct {
    double with_padding_time;
    double without_padding_time;
    double speedup;
    uint64_t estimated_invalidations;
} false_sharing_result_t;

void demo_false_sharing(int num_threads, int iterations,
                        false_sharing_result_t *result);

// 2.4.29.g-h: Padding and alignment
void *alloc_cache_aligned(size_t size);
void free_cache_aligned(void *ptr);

// Cache line info
size_t get_cache_line_size(void);
bool is_cache_aligned(void *ptr);

// 2.4.29.f: Detection (simulated)
void estimate_cache_misses(int *arr, int n, int stride, cache_stats_t *stats);

// Cache-friendly patterns
void demo_row_major_vs_column_major(int rows, int cols);
void demo_blocking(int *matrix, int n, int block_size);

// ============== ASYNC PROGRAMMING ==============
// 2.4.30

// 2.4.30.b: Callbacks
typedef void *(*async_task_t)(void *arg);

void async_run(async_task_t task, void *arg, async_callbacks_t *callbacks);

// 2.4.30.c: Promise/Future
promise_t *promise_create(void);
void promise_set_result(promise_t *p, void *result);
void promise_set_error(promise_t *p, int error);
void promise_free(promise_t *p);

future_t future_from_promise(promise_t *p);
void *future_await(future_t *f);
void *future_await_timeout(future_t *f, int timeout_ms);
bool future_poll(future_t *f, void **result);

// Combinators
future_t future_then(future_t *f, void *(*transform)(void*));
future_t future_all(future_t *futures, int count);
future_t future_any(future_t *futures, int count);

// 2.4.30.d: Event loop
int event_loop_init(event_loop_t *loop);
void event_loop_destroy(event_loop_t *loop);
void event_loop_post(event_loop_t *loop, void (*handler)(void*), void *arg);
void event_loop_run(event_loop_t *loop);
void event_loop_run_one(event_loop_t *loop);
void event_loop_stop(event_loop_t *loop);

// Timer events
void event_loop_set_timeout(event_loop_t *loop, int ms,
                            void (*handler)(void*), void *arg);
void event_loop_set_interval(event_loop_t *loop, int ms,
                             void (*handler)(void*), void *arg);

// 2.4.30.e: Coroutines (simplified)
coroutine_t *coro_create(void (*func)(coroutine_t*, void*), void *arg);
void coro_destroy(coroutine_t *coro);
void coro_resume(coroutine_t *coro);
void coro_yield(coroutine_t *coro, void *value);
void *coro_get_value(coroutine_t *coro);
bool coro_finished(coroutine_t *coro);

// 2.4.30.g: Comparison benchmark
typedef struct {
    double thread_time_ms;
    double async_time_ms;
    double thread_memory_mb;
    double async_memory_mb;
    int thread_context_switches;
} async_comparison_t;

void compare_threads_vs_async(int num_tasks, async_comparison_t *result);
```

---

## Exemple

```c
#include "openmp_advanced.h"

// ============== OPENMP DEMO ==============
void openmp_demo(void) {
    printf("=== OpenMP ===\n");

    // 2.4.28.b: Parallel region
    #pragma omp parallel
    {
        int tid = omp_get_thread_num();
        int nthreads = omp_get_num_threads();
        printf("Thread %d of %d\n", tid, nthreads);
    }

    // 2.4.28.c-d: Parallel for
    int arr[1000];
    #pragma omp parallel for
    for (int i = 0; i < 1000; i++) {
        arr[i] = i * 2;
    }

    // 2.4.28.e: private vs shared
    int shared_var = 0;
    #pragma omp parallel private(shared_var)
    {
        shared_var = omp_get_thread_num();  // Each thread has own copy
        printf("Private: %d\n", shared_var);
    }

    // 2.4.28.f: Reduction
    int64_t sum = 0;
    #pragma omp parallel for reduction(+:sum)
    for (int i = 0; i < 1000; i++) {
        sum += arr[i];
    }
    printf("Sum: %ld\n", sum);

    // 2.4.28.g: Schedule
    #pragma omp parallel for schedule(dynamic, 10)
    for (int i = 0; i < 1000; i++) {
        // Uneven work - dynamic scheduling helps
        for (int j = 0; j < i; j++) { }
    }

    // 2.4.28.h: Critical section
    int counter = 0;
    #pragma omp parallel for
    for (int i = 0; i < 1000; i++) {
        #pragma omp critical
        {
            counter++;
        }
    }

    // 2.4.28.i: Atomic (faster for simple ops)
    counter = 0;
    #pragma omp parallel for
    for (int i = 0; i < 1000; i++) {
        #pragma omp atomic
        counter++;
    }
    printf("Counter: %d\n", counter);

    // 2.4.28.j: Barrier
    #pragma omp parallel
    {
        // Phase 1
        printf("Phase 1 - Thread %d\n", omp_get_thread_num());

        #pragma omp barrier  // All wait here

        // Phase 2
        printf("Phase 2 - Thread %d\n", omp_get_thread_num());
    }
}

// ============== CACHE EFFECTS DEMO ==============
void cache_demo(void) {
    printf("\n=== Cache Effects ===\n");

    // 2.4.29.d: False sharing demonstration
    printf("Testing false sharing...\n");
    false_sharing_result_t result;
    demo_false_sharing(4, 10000000, &result);

    printf("Without padding: %.2f ms\n", result.without_padding_time);
    printf("With padding: %.2f ms\n", result.with_padding_time);
    printf("Speedup from padding: %.2fx\n", result.speedup);

    // 2.4.29.g-h: Cache-aligned allocation
    void *aligned = alloc_cache_aligned(1000);
    printf("Aligned ptr: %p (aligned: %s)\n",
           aligned, is_cache_aligned(aligned) ? "yes" : "no");
    free_cache_aligned(aligned);

    // Row-major vs column-major
    printf("\nRow vs Column major access:\n");
    demo_row_major_vs_column_major(1000, 1000);
}

// 2.4.29.d: False sharing example
void false_sharing_worker_bad(unpadded_counter_t *counters, int id, int iters) {
    for (int i = 0; i < iters; i++) {
        counters[id].value++;  // Adjacent counters share cache line!
    }
}

void false_sharing_worker_good(padded_counter_t *counters, int id, int iters) {
    for (int i = 0; i < iters; i++) {
        counters[id].value++;  // Each counter on own cache line
    }
}

// ============== ASYNC DEMO ==============
void async_demo(void) {
    printf("\n=== Async Programming ===\n");

    // 2.4.30.b: Callbacks
    void on_complete(void *result, void *ctx) {
        printf("Task completed with result: %d\n", *(int*)result);
    }

    async_callbacks_t callbacks = {
        .on_complete = on_complete,
        .context = NULL
    };

    void *compute(void *arg) {
        int *result = malloc(sizeof(int));
        *result = 42;
        return result;
    }

    async_run(compute, NULL, &callbacks);

    // 2.4.30.c: Future/Promise
    printf("\nFuture/Promise:\n");
    promise_t *promise = promise_create();
    future_t future = future_from_promise(promise);

    // In real code, this would be in another thread
    pthread_t worker;
    pthread_create(&worker, NULL, (void*(*)(void*))({
        void *f(void *p) {
            usleep(100000);
            promise_set_result(p, "Hello from async!");
            return NULL;
        }; f;
    }), promise);

    // Wait for result
    char *msg = future_await(future);
    printf("Got: %s\n", msg);
    pthread_join(worker, NULL);
    promise_free(promise);

    // 2.4.30.d: Event loop
    printf("\nEvent Loop:\n");
    event_loop_t loop;
    event_loop_init(&loop);

    void handler1(void *arg) {
        printf("Event 1 handled\n");
    }
    void handler2(void *arg) {
        printf("Event 2 handled\n");
        event_loop_stop(arg);
    }

    event_loop_post(&loop, handler1, NULL);
    event_loop_post(&loop, handler2, &loop);

    event_loop_run(&loop);
    event_loop_destroy(&loop);

    // 2.4.30.e: Coroutines
    printf("\nCoroutines:\n");

    void generator(coroutine_t *self, void *arg) {
        for (int i = 0; i < 5; i++) {
            int *val = malloc(sizeof(int));
            *val = i;
            coro_yield(self, val);
        }
    }

    coroutine_t *coro = coro_create(generator, NULL);
    while (!coro_finished(coro)) {
        coro_resume(coro);
        if (!coro_finished(coro)) {
            int *val = coro_get_value(coro);
            printf("Yielded: %d\n", *val);
            free(val);
        }
    }
    coro_destroy(coro);

    // 2.4.30.g: Comparison
    printf("\nThreads vs Async comparison:\n");
    async_comparison_t cmp;
    compare_threads_vs_async(1000, &cmp);
    printf("Threads: %.2f ms, %.2f MB\n", cmp.thread_time_ms, cmp.thread_memory_mb);
    printf("Async: %.2f ms, %.2f MB\n", cmp.async_time_ms, cmp.async_memory_mb);
}

int main(void) {
    openmp_demo();
    cache_demo();
    async_demo();
    return 0;
}
```

---

## Tests Moulinette

```rust
// OpenMP tests
#[test] fn test_parallel_region()        // 2.4.28.b
#[test] fn test_parallel_for()           // 2.4.28.c-d
#[test] fn test_private_shared()         // 2.4.28.e
#[test] fn test_reduction()              // 2.4.28.f
#[test] fn test_schedule()               // 2.4.28.g
#[test] fn test_critical_atomic()        // 2.4.28.h-i
#[test] fn test_barrier()                // 2.4.28.j

// Cache tests
#[test] fn test_cache_coherency()        // 2.4.29.a-b
#[test] fn test_false_sharing()          // 2.4.29.d-e
#[test] fn test_padding_solution()       // 2.4.29.g
#[test] fn test_cache_alignment()        // 2.4.29.h

// Async tests
#[test] fn test_callbacks()              // 2.4.30.b
#[test] fn test_future_promise()         // 2.4.30.c
#[test] fn test_event_loop()             // 2.4.30.d
#[test] fn test_coroutines()             // 2.4.30.e
```

---

## Bareme

| Critere | Points |
|---------|--------|
| OpenMP (2.4.28) | 40 |
| Cache Effects (2.4.29) | 30 |
| Async Programming (2.4.30) | 30 |
| **Total** | **100** |

---

## Fichiers

```
ex09/
├── openmp_advanced.h
├── openmp_demo.c
├── cache_effects.c
├── async.c
├── coroutine.c
└── Makefile
```

## Compilation

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c17 -fopenmp
LDFLAGS = -fopenmp -lpthread

all: demo

demo: openmp_demo.o cache_effects.o async.o coroutine.o
	$(CC) -o $@ $^ $(LDFLAGS)
```
