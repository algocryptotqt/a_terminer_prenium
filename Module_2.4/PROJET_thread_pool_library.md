# PROJET 2.4: Thread Pool Library

**Module**: 2.4 - Concurrency & Synchronization
**Type**: Projet Final Integratif
**Difficulte**: Tres difficile
**Duree**: 50-70h
**Score qualite**: 98/100

---

## Vue d'ensemble

Implementer une bibliotheque de thread pool complete et performante, integrant tous les concepts du module: synchronisation, patterns paralleles, lock-free, et work stealing.

---

## Concepts Couverts (15 concepts)

| Ref | Concept | Description | Exercices lies |
|-----|---------|-------------|----------------|
| a | Thread pool | Fixed number of workers | ex00, ex08 |
| b | Task queue | Thread-safe bounded buffer | ex02, ex03 |
| c | Submit | Add task to queue | ex08 |
| d | Future | Get result later | ex06, ex09 |
| e | Wait | Block for result | ex02 |
| f | Shutdown | Graceful stop | ex08 |
| g | Resize | Dynamic worker count | ex08 |
| h | Statistics | Tasks completed, queue size | ex08 |
| i | Priority | High-priority tasks | ex04 |
| j | Timeout | Task deadline | ex05 |
| k | Cancellation | Cancel pending tasks | ex02 |
| l | Work stealing | Per-worker queues | ex07, ex08 |
| m | Bonus: Lock-free queue | CAS-based | ex06, ex07 |
| n | Bonus: HTTP server | Using pool | Networking |
| o | Bonus: Parallel algorithms | Sort, map, reduce | ex08 |

---

## Architecture

```
+----------------------------------------------------------+
|                    Thread Pool Library                    |
+----------------------------------------------------------+
|  +----------------+  +----------------+  +-------------+  |
|  |   Task Queue   |  |  Work Stealing |  |   Futures   |  |
|  | (Lock-free opt)|  |  (Chase-Lev)   |  |  (Promise)  |  |
|  +----------------+  +----------------+  +-------------+  |
|                                                          |
|  +----------------+  +----------------+  +-------------+  |
|  |    Workers     |  |   Scheduler    |  |   Monitor   |  |
|  | (Thread pool)  |  |  (Priority)    |  |  (Stats)    |  |
|  +----------------+  +----------------+  +-------------+  |
|                                                          |
|  +----------------+  +----------------+  +-------------+  |
|  | Parallel Algos |  |  HTTP Server   |  |   Utils     |  |
|  | (map/reduce)   |  |   (Bonus)      |  |  (Timers)   |  |
|  +----------------+  +----------------+  +-------------+  |
+----------------------------------------------------------+
```

### Structures de Donnees

```c
#include <pthread.h>
#include <stdatomic.h>
#include <stdbool.h>
#include <time.h>

// Forward declarations
typedef struct threadpool threadpool_t;
typedef struct task task_t;
typedef struct future future_t;

// Task priority levels (i)
typedef enum {
    PRIORITY_LOW = 0,
    PRIORITY_NORMAL = 1,
    PRIORITY_HIGH = 2,
    PRIORITY_CRITICAL = 3
} task_priority_t;

// Task status
typedef enum {
    TASK_PENDING,
    TASK_RUNNING,
    TASK_COMPLETED,
    TASK_FAILED,
    TASK_CANCELLED,      // k: Cancellation
    TASK_TIMEOUT         // j: Timeout
} task_status_t;

// Task function types
typedef void *(*task_func_t)(void *arg);
typedef void (*task_callback_t)(void *result, void *context);

// Task structure (b, c)
typedef struct task {
    uint64_t id;
    task_func_t func;
    void *arg;
    task_priority_t priority;    // i: Priority
    task_status_t status;

    // Timing
    struct timespec submit_time;
    struct timespec start_time;
    struct timespec end_time;
    int64_t timeout_ms;          // j: Timeout (-1 = none)

    // Result handling (d)
    void *result;
    int error_code;
    future_t *future;
    task_callback_t callback;
    void *callback_ctx;

    // Cancellation (k)
    _Atomic bool cancelled;
    _Atomic bool cancel_requested;

    // Linked list
    struct task *next;
    struct task *prev;
} task_t;

// Future/Promise (d, e)
typedef struct future {
    task_t *task;
    _Atomic bool ready;
    void *result;
    int error;
    pthread_mutex_t mutex;
    pthread_cond_t cond;

    // Chaining
    struct future *then_future;
    task_func_t transform;
} future_t;

// b: Task queue (thread-safe, with priority)
typedef struct {
    task_t *queues[4];           // One per priority level
    task_t *queue_tails[4];
    size_t sizes[4];
    size_t total_size;
    size_t max_size;

    pthread_mutex_t mutex;
    pthread_cond_t not_empty;
    pthread_cond_t not_full;
} task_queue_t;

// l: Work stealing deque (per worker)
typedef struct {
    task_t **buffer;
    _Atomic int64_t top;
    _Atomic int64_t bottom;
    size_t capacity;
    size_t mask;
} ws_deque_t;

// Worker thread info
typedef struct {
    pthread_t thread;
    int id;
    threadpool_t *pool;
    ws_deque_t *local_queue;     // l: For work stealing

    // Stats
    uint64_t tasks_executed;
    uint64_t tasks_stolen;
    uint64_t idle_time_ns;
    bool active;
} worker_t;

// h: Statistics
typedef struct {
    _Atomic uint64_t tasks_submitted;
    _Atomic uint64_t tasks_completed;
    _Atomic uint64_t tasks_failed;
    _Atomic uint64_t tasks_cancelled;
    _Atomic uint64_t tasks_timeout;
    _Atomic uint64_t tasks_stolen;

    _Atomic uint64_t total_wait_time_ns;
    _Atomic uint64_t total_exec_time_ns;
    _Atomic uint64_t queue_high_water;

    double avg_wait_time_ms;
    double avg_exec_time_ms;
    double throughput_per_sec;
} pool_stats_t;

// Thread pool configuration
typedef struct {
    int min_workers;
    int max_workers;             // g: Dynamic sizing
    int initial_workers;         // a: Fixed number
    size_t queue_max_size;       // b: Bounded
    bool enable_work_stealing;   // l
    bool enable_lock_free;       // m
    int stats_interval_ms;       // h
} pool_config_t;

// a: Thread pool
typedef struct threadpool {
    worker_t *workers;
    int num_workers;
    int min_workers;
    int max_workers;

    task_queue_t *queue;         // b: Central queue
    ws_deque_t **worker_deques;  // l: Per-worker queues

    pool_stats_t stats;          // h: Statistics
    pool_config_t config;

    pthread_mutex_t pool_mutex;
    pthread_cond_t pool_cond;

    _Atomic bool shutdown;       // f: Shutdown
    _Atomic bool immediate_shutdown;
    _Atomic int active_workers;

    // Timer thread for timeouts
    pthread_t timer_thread;
    bool timer_running;
} threadpool_t;
```

### API Principale

```c
// ============== POOL LIFECYCLE ==============

// a: Create pool with configuration
threadpool_t *pool_create(pool_config_t *config);
threadpool_t *pool_create_default(int num_workers);
void pool_destroy(threadpool_t *pool);

// f: Shutdown
void pool_shutdown(threadpool_t *pool);           // Wait for pending tasks
void pool_shutdown_now(threadpool_t *pool);       // Cancel pending

// g: Dynamic resizing
int pool_resize(threadpool_t *pool, int new_size);
int pool_add_worker(threadpool_t *pool);
int pool_remove_worker(threadpool_t *pool);

// ============== TASK SUBMISSION ==============

// c: Submit task
int pool_submit(threadpool_t *pool, task_func_t func, void *arg);

// c + i: Submit with priority
int pool_submit_priority(threadpool_t *pool, task_func_t func, void *arg,
                         task_priority_t priority);

// c + j: Submit with timeout
int pool_submit_timeout(threadpool_t *pool, task_func_t func, void *arg,
                        int64_t timeout_ms);

// c + d: Submit with future
future_t *pool_submit_future(threadpool_t *pool, task_func_t func, void *arg);
future_t *pool_submit_future_priority(threadpool_t *pool, task_func_t func,
                                      void *arg, task_priority_t priority);

// Batch submission
int pool_submit_batch(threadpool_t *pool, task_func_t *funcs, void **args,
                      int count);

// ============== FUTURE API (d, e) ==============

// e: Wait for result
void *future_get(future_t *f);
void *future_get_timeout(future_t *f, int64_t timeout_ms);
bool future_is_ready(future_t *f);
task_status_t future_status(future_t *f);
void future_free(future_t *f);

// Chaining
future_t *future_then(future_t *f, task_func_t transform);
future_t *future_catch(future_t *f, task_func_t error_handler);

// Combinators
future_t *future_all(future_t **futures, int count);
future_t *future_any(future_t **futures, int count);
future_t *future_race(future_t **futures, int count);

// ============== CANCELLATION (k) ==============

int pool_cancel_task(threadpool_t *pool, uint64_t task_id);
int pool_cancel_all(threadpool_t *pool);
bool task_is_cancelled(task_t *task);

// Cooperative cancellation check (call in task)
bool pool_should_cancel(void);

// ============== WORK STEALING (l) ==============

// Already integrated, but exposed for testing
int pool_enable_work_stealing(threadpool_t *pool, bool enable);
int pool_get_stolen_count(threadpool_t *pool);

// ============== STATISTICS (h) ==============

void pool_get_stats(threadpool_t *pool, pool_stats_t *stats);
void pool_reset_stats(threadpool_t *pool);
void pool_print_stats(threadpool_t *pool);

// Real-time monitoring
typedef void (*stats_callback_t)(pool_stats_t *stats, void *ctx);
void pool_set_stats_callback(threadpool_t *pool, stats_callback_t cb, void *ctx);

// ============== PARALLEL ALGORITHMS (o) ==============

// Parallel for
typedef void (*parallel_func_t)(int index, void *ctx);
void pool_parallel_for(threadpool_t *pool, int start, int end,
                       parallel_func_t func, void *ctx);

// Parallel map
typedef void *(*map_func_t)(void *item);
void **pool_map(threadpool_t *pool, void **items, int count, map_func_t func);

// Parallel reduce
typedef void *(*reduce_func_t)(void *a, void *b);
void *pool_reduce(threadpool_t *pool, void **items, int count,
                  reduce_func_t func, void *identity);

// Parallel sort
typedef int (*compare_func_t)(const void *a, const void *b);
void pool_parallel_sort(threadpool_t *pool, void *arr, size_t count,
                        size_t size, compare_func_t cmp);

// ============== HTTP SERVER BONUS (n) ==============

typedef struct http_request {
    char method[16];
    char path[256];
    char *body;
    size_t body_len;
} http_request_t;

typedef struct http_response {
    int status;
    char *body;
    size_t body_len;
} http_response_t;

typedef http_response_t *(*http_handler_t)(http_request_t *req);

typedef struct {
    threadpool_t *pool;
    int port;
    http_handler_t handler;
    int server_fd;
    bool running;
} http_server_t;

int http_server_create(http_server_t *server, threadpool_t *pool, int port);
void http_server_route(http_server_t *server, const char *path,
                       http_handler_t handler);
int http_server_start(http_server_t *server);
void http_server_stop(http_server_t *server);
```

---

## Exemple d'Utilisation

```c
#include "threadpool.h"

void *compute_factorial(void *arg) {
    int n = *(int*)arg;
    uint64_t result = 1;

    // k: Check for cancellation periodically
    for (int i = 2; i <= n && !pool_should_cancel(); i++) {
        result *= i;
    }

    uint64_t *ret = malloc(sizeof(uint64_t));
    *ret = result;
    return ret;
}

void *slow_task(void *arg) {
    for (int i = 0; i < 10; i++) {
        if (pool_should_cancel()) {
            return NULL;  // k: Cooperative cancellation
        }
        sleep(1);
    }
    return "done";
}

int main(void) {
    // a: Create pool
    pool_config_t config = {
        .min_workers = 2,
        .max_workers = 8,
        .initial_workers = 4,
        .queue_max_size = 1000,
        .enable_work_stealing = true,  // l
        .enable_lock_free = true,      // m
        .stats_interval_ms = 1000      // h
    };

    threadpool_t *pool = pool_create(&config);

    // c: Simple task submission
    int n = 20;
    pool_submit(pool, compute_factorial, &n);

    // c + d: Submit with future
    future_t *f1 = pool_submit_future(pool, compute_factorial, &n);

    // e: Wait for result
    uint64_t *result = future_get(f1);
    printf("20! = %lu\n", *result);
    free(result);
    future_free(f1);

    // i: Priority tasks
    pool_submit_priority(pool, compute_factorial, &n, PRIORITY_HIGH);
    pool_submit_priority(pool, compute_factorial, &n, PRIORITY_CRITICAL);

    // j: Task with timeout
    future_t *f_timeout = pool_submit_future(pool, slow_task, NULL);
    void *res = future_get_timeout(f_timeout, 3000);  // 3 second timeout
    if (future_status(f_timeout) == TASK_TIMEOUT) {
        printf("Task timed out!\n");
    }
    future_free(f_timeout);

    // k: Cancellation
    future_t *f_cancel = pool_submit_future(pool, slow_task, NULL);
    sleep(1);
    pool_cancel_task(pool, /* task_id */);

    // d: Future chaining
    future_t *f_chain = pool_submit_future(pool, compute_factorial, &n);
    future_t *f_doubled = future_then(f_chain, ({
        void *double_it(void *x) {
            uint64_t *r = malloc(sizeof(uint64_t));
            *r = *(uint64_t*)x * 2;
            free(x);
            return r;
        }; double_it;
    }));
    uint64_t *doubled = future_get(f_doubled);
    printf("20! * 2 = %lu\n", *doubled);
    free(doubled);
    future_free(f_doubled);

    // o: Parallel algorithms
    printf("\n=== Parallel Algorithms ===\n");

    // Parallel for
    int arr[1000];
    pool_parallel_for(pool, 0, 1000, ({
        void fill(int i, void *a) { ((int*)a)[i] = i; }; fill;
    }), arr);

    // Parallel map
    void *inputs[100];
    for (int i = 0; i < 100; i++) {
        inputs[i] = malloc(sizeof(int));
        *(int*)inputs[i] = i;
    }
    void **squares = pool_map(pool, inputs, 100, ({
        void *sq(void *x) {
            int *r = malloc(sizeof(int));
            *r = (*(int*)x) * (*(int*)x);
            return r;
        }; sq;
    }));

    // Parallel reduce
    void *sum = pool_reduce(pool, squares, 100, ({
        void *add(void *a, void *b) {
            int *r = malloc(sizeof(int));
            *r = *(int*)a + *(int*)b;
            return r;
        }; add;
    }), &(int){0});
    printf("Sum of squares 0-99: %d\n", *(int*)sum);

    // g: Dynamic resizing
    pool_resize(pool, 8);  // Scale up
    pool_resize(pool, 2);  // Scale down

    // h: Statistics
    pool_stats_t stats;
    pool_get_stats(pool, &stats);
    printf("\n=== Stats ===\n");
    printf("Submitted: %lu\n", stats.tasks_submitted);
    printf("Completed: %lu\n", stats.tasks_completed);
    printf("Stolen: %lu\n", stats.tasks_stolen);
    printf("Avg wait: %.2f ms\n", stats.avg_wait_time_ms);
    printf("Throughput: %.0f/sec\n", stats.throughput_per_sec);

    // n: HTTP server (bonus)
    printf("\n=== HTTP Server ===\n");
    http_server_t server;
    http_server_create(&server, pool, 8080);
    http_server_route(&server, "/compute", ({
        http_response_t *handler(http_request_t *req) {
            http_response_t *resp = malloc(sizeof(http_response_t));
            resp->status = 200;
            resp->body = strdup("{\"result\": 42}");
            resp->body_len = strlen(resp->body);
            return resp;
        }; handler;
    }));
    // http_server_start(&server);  // Would block

    // f: Graceful shutdown
    pool_shutdown(pool);
    pool_destroy(pool);

    return 0;
}
```

---

## Tests Moulinette

```rust
mod pool_tests {
    #[test] fn test_create_destroy()      // a
    #[test] fn test_submit()              // c
    #[test] fn test_queue_bounded()       // b
    #[test] fn test_priority_order()      // i
}

mod future_tests {
    #[test] fn test_future_get()          // d, e
    #[test] fn test_future_timeout()      // j
    #[test] fn test_future_chain()        // d
    #[test] fn test_future_all()          // d
}

mod lifecycle_tests {
    #[test] fn test_shutdown_graceful()   // f
    #[test] fn test_shutdown_immediate()  // f
    #[test] fn test_resize()              // g
}

mod cancellation_tests {
    #[test] fn test_cancel_pending()      // k
    #[test] fn test_cancel_running()      // k
    #[test] fn test_cooperative_cancel()  // k
}

mod work_stealing_tests {
    #[test] fn test_stealing()            // l
    #[test] fn test_load_balance()        // l
    #[test] fn test_chase_lev_deque()     // l
}

mod stats_tests {
    #[test] fn test_basic_stats()         // h
    #[test] fn test_timing_stats()        // h
    #[test] fn test_stats_callback()      // h
}

mod lockfree_tests {
    #[test] fn test_lockfree_queue()      // m
    #[test] fn test_mpmc_correctness()    // m
}

mod parallel_tests {
    #[test] fn test_parallel_for()        // o
    #[test] fn test_parallel_map()        // o
    #[test] fn test_parallel_reduce()     // o
    #[test] fn test_parallel_sort()       // o
}

mod http_tests {
    #[test] fn test_http_basic()          // n
    #[test] fn test_http_concurrent()     // n
}

mod stress_tests {
    #[test] fn test_high_load()
    #[test] fn test_many_workers()
    #[test] fn test_work_stealing_balance()
}
```

---

## Bareme

| Critere | Points |
|---------|--------|
| **Core Pool** | |
| Thread pool creation (a) | 10 |
| Task queue (b) | 10 |
| Task submission (c) | 5 |
| Future/Promise (d, e) | 10 |
| Shutdown (f) | 5 |
| Dynamic resize (g) | 5 |
| Statistics (h) | 5 |
| Priority (i) | 5 |
| Timeout (j) | 5 |
| Cancellation (k) | 5 |
| Work stealing (l) | 15 |
| **Bonus** | |
| Lock-free queue (m) | +10 |
| HTTP server (n) | +10 |
| Parallel algorithms (o) | +10 |
| **Total** | **80** (+30 bonus) |

---

## Fichiers

```
PROJET_2.4_ThreadPool/
├── include/
│   ├── threadpool.h        # API principale
│   ├── task.h              # Task definitions
│   ├── future.h            # Future/Promise
│   ├── queue.h             # Task queues
│   ├── work_stealing.h     # Work stealing
│   ├── stats.h             # Statistics
│   ├── parallel.h          # Parallel algorithms
│   └── http_server.h       # HTTP (bonus)
├── src/
│   ├── pool.c              # Pool core
│   ├── task.c              # Task management
│   ├── future.c            # Future implementation
│   ├── queue.c             # Lock-based queue
│   ├── queue_lockfree.c    # Lock-free queue (bonus)
│   ├── work_stealing.c     # Work stealing
│   ├── stats.c             # Statistics
│   ├── parallel_for.c      # Parallel for
│   ├── parallel_map.c      # Map/Reduce
│   ├── parallel_sort.c     # Parallel sort
│   └── http_server.c       # HTTP (bonus)
├── tests/
│   └── ...
├── examples/
│   ├── basic_usage.c
│   ├── web_server.c
│   └── parallel_compute.c
├── Makefile
└── README.md
```

---

## Makefile

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c17 -pthread -O2
LDFLAGS = -pthread

SRCS = $(wildcard src/*.c)
OBJS = $(SRCS:.c=.o)

all: libthreadpool.a examples

libthreadpool.a: $(OBJS)
	ar rcs $@ $^

examples: libthreadpool.a
	$(CC) $(CFLAGS) -o basic_usage examples/basic_usage.c -L. -lthreadpool $(LDFLAGS)
	$(CC) $(CFLAGS) -o web_server examples/web_server.c -L. -lthreadpool $(LDFLAGS)

clean:
	rm -f $(OBJS) libthreadpool.a basic_usage web_server

.PHONY: all examples clean
```
