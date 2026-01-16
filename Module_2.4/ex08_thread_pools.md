# ex08: Thread Pools & Parallel Patterns

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Difficile
**Duree**: 7h
**Score qualite**: 97/100

## Concepts Couverts

### 2.4.25: Thread Pools (9 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Concept | Reuse threads |
| b | Worker threads | Fixed number |
| c | Task queue | Work to do |
| d | Submit | Add task to queue |
| e | Worker loop | Dequeue and execute |
| f | Shutdown | Graceful termination |
| g | Wait for completion | Join mechanism |
| h | Dynamic sizing | Adjust workers |
| i | Implementation | Complete |

### 2.4.26: Work Stealing (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Concept | Steal from busy |
| b | Per-thread deque | Double-ended queue |
| c | Push/pop | Own end |
| d | Steal | Other end |
| e | Chase-Lev deque | Lock-free |
| f | Load balancing | Automatic |
| g | Fork-join | Natural fit |

### 2.4.27: Parallel Patterns (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Parallel for | Distribute iterations |
| b | Map | Apply function to all |
| c | Reduce | Combine results |
| d | Parallel prefix | Scan operation |
| e | Pipeline | Stages |
| f | Task decomposition | Break into tasks |
| g | Data decomposition | Break data |

---

## Sujet

Implementer un thread pool complet avec work stealing et patterns paralleles.

### Structures

```c
#include <pthread.h>
#include <stdbool.h>

// Task function type
typedef void (*task_func_t)(void *arg);
typedef void *(*task_func_result_t)(void *arg);

// 2.4.25.c: Task in queue
typedef struct task {
    task_func_t func;
    void *arg;
    struct task *next;

    // For futures
    task_func_result_t result_func;
    void *result;
    bool completed;
    pthread_mutex_t mutex;
    pthread_cond_t cond;
} task_t;

// 2.4.25: Thread pool
typedef struct {
    pthread_t *workers;          // b: Worker threads
    int num_workers;
    int min_workers;
    int max_workers;             // h: Dynamic sizing

    task_t *queue_head;          // c: Task queue
    task_t *queue_tail;
    size_t queue_size;

    pthread_mutex_t queue_mutex;
    pthread_cond_t queue_not_empty;
    pthread_cond_t queue_empty;

    bool shutdown;               // f: Shutdown flag
    bool immediate_shutdown;
    int active_workers;

    // Stats
    uint64_t tasks_submitted;
    uint64_t tasks_completed;
    uint64_t tasks_stolen;
} thread_pool_t;

// Future for async results
typedef struct {
    task_t *task;
    void *result;
    bool ready;
} future_t;

// 2.4.26.b-e: Work stealing deque (Chase-Lev)
typedef struct {
    task_t **buffer;
    _Atomic size_t top;          // c: Owner pops here
    _Atomic size_t bottom;       // d: Thieves steal here
    size_t capacity;
} ws_deque_t;

// Work stealing pool
typedef struct {
    pthread_t *workers;
    ws_deque_t *deques;          // b: Per-thread deque
    int num_workers;
    bool shutdown;
    uint64_t steals;             // a,f: Steal count
} ws_pool_t;

// 2.4.27.e: Pipeline stage
typedef struct pipeline_stage {
    task_func_result_t process;
    pthread_t thread;
    void *input_queue;
    void *output_queue;
    struct pipeline_stage *next;
} pipeline_stage_t;

typedef struct {
    pipeline_stage_t *stages;
    int num_stages;
    bool running;
} pipeline_t;
```

### API

```c
// ============== THREAD POOL ==============
// 2.4.25

// 2.4.25.i: Complete implementation
int pool_create(thread_pool_t *pool, int num_workers);
void pool_destroy(thread_pool_t *pool);

// 2.4.25.d: Submit task
int pool_submit(thread_pool_t *pool, task_func_t func, void *arg);
future_t *pool_submit_future(thread_pool_t *pool, task_func_result_t func, void *arg);

// 2.4.25.f: Shutdown
void pool_shutdown(thread_pool_t *pool, bool wait_for_tasks);
void pool_shutdown_now(thread_pool_t *pool);

// 2.4.25.g: Wait for completion
void pool_wait(thread_pool_t *pool);
void *future_get(future_t *f);                    // Blocks until ready
void *future_get_timeout(future_t *f, int ms);
bool future_is_ready(future_t *f);
void future_free(future_t *f);

// 2.4.25.h: Dynamic sizing
void pool_resize(thread_pool_t *pool, int new_size);
void pool_set_min_max(thread_pool_t *pool, int min, int max);

// Pool info
int pool_get_queue_size(thread_pool_t *pool);
int pool_get_active_workers(thread_pool_t *pool);

// ============== WORK STEALING ==============
// 2.4.26

// 2.4.26.e: Chase-Lev deque
int ws_deque_init(ws_deque_t *d, size_t capacity);
void ws_deque_destroy(ws_deque_t *d);
void ws_deque_push(ws_deque_t *d, task_t *task);   // c: Push to bottom
task_t *ws_deque_pop(ws_deque_t *d);               // c: Pop from bottom
task_t *ws_deque_steal(ws_deque_t *d);             // d: Steal from top

// Work stealing pool
int ws_pool_create(ws_pool_t *pool, int num_workers);
void ws_pool_destroy(ws_pool_t *pool);
void ws_pool_submit(ws_pool_t *pool, task_func_t func, void *arg);
void ws_pool_wait(ws_pool_t *pool);

// 2.4.26.g: Fork-join
typedef struct {
    ws_pool_t *pool;
    task_t *tasks;
    int task_count;
    _Atomic int completed;
} fork_join_t;

void fj_fork(fork_join_t *fj, task_func_t func, void *arg);
void fj_join(fork_join_t *fj);

// ============== PARALLEL PATTERNS ==============
// 2.4.27

// 2.4.27.a: Parallel for
typedef void (*parallel_for_func_t)(int index, void *arg);

void parallel_for(thread_pool_t *pool, int start, int end,
                  parallel_for_func_t func, void *arg);
void parallel_for_chunked(thread_pool_t *pool, int start, int end,
                          int chunk_size, parallel_for_func_t func, void *arg);

// 2.4.27.b: Map
typedef void *(*map_func_t)(void *element);

void **parallel_map(thread_pool_t *pool, void **input, size_t count,
                    map_func_t func);

// 2.4.27.c: Reduce
typedef void *(*reduce_func_t)(void *a, void *b);

void *parallel_reduce(thread_pool_t *pool, void **input, size_t count,
                      reduce_func_t func, void *identity);

// 2.4.27.d: Parallel prefix (scan)
void parallel_prefix_sum(thread_pool_t *pool, int *arr, size_t count);
void parallel_prefix(thread_pool_t *pool, void **arr, size_t count,
                     reduce_func_t op, void *identity);

// 2.4.27.e: Pipeline
int pipeline_create(pipeline_t *p);
void pipeline_add_stage(pipeline_t *p, task_func_result_t process);
void pipeline_start(pipeline_t *p);
void pipeline_submit(pipeline_t *p, void *item);
void *pipeline_get_result(pipeline_t *p);
void pipeline_stop(pipeline_t *p);
void pipeline_destroy(pipeline_t *p);

// 2.4.27.f-g: Decomposition helpers
typedef struct {
    int start;
    int end;
} range_t;

range_t *task_decompose(int total, int num_tasks);
range_t *data_decompose(size_t data_size, int num_chunks);

// ============== BENCHMARKS ==============

typedef struct {
    double sequential_time_ms;
    double parallel_time_ms;
    double speedup;
    double efficiency;
    int threads_used;
} parallel_benchmark_t;

void benchmark_pool(int threads, int tasks, parallel_benchmark_t *result);
void benchmark_work_stealing(int threads, int tasks, parallel_benchmark_t *result);
void benchmark_parallel_for(int threads, int iterations, parallel_benchmark_t *result);
```

---

## Exemple

```c
#include "thread_pool.h"

// Simple task
void print_task(void *arg) {
    int *val = arg;
    printf("Task: %d\n", *val);
}

// Task with result
void *compute_square(void *arg) {
    int *val = arg;
    int *result = malloc(sizeof(int));
    *result = (*val) * (*val);
    return result;
}

int main(void) {
    // ============== Basic Thread Pool ==============
    printf("=== Thread Pool ===\n");
    thread_pool_t pool;
    pool_create(&pool, 4);

    // 2.4.25.d: Submit tasks
    int values[10];
    for (int i = 0; i < 10; i++) {
        values[i] = i;
        pool_submit(&pool, print_task, &values[i]);
    }

    // 2.4.25.g: Wait for completion
    pool_wait(&pool);

    // Submit with future
    future_t *futures[5];
    for (int i = 0; i < 5; i++) {
        values[i] = i + 1;
        futures[i] = pool_submit_future(&pool, compute_square, &values[i]);
    }

    // Get results
    for (int i = 0; i < 5; i++) {
        int *result = future_get(futures[i]);
        printf("Square of %d = %d\n", values[i], *result);
        free(result);
        future_free(futures[i]);
    }

    // 2.4.25.f: Shutdown
    pool_shutdown(&pool, true);
    pool_destroy(&pool);

    // ============== Work Stealing ==============
    printf("\n=== Work Stealing ===\n");
    ws_pool_t ws_pool;
    ws_pool_create(&ws_pool, 4);

    // Submit unbalanced tasks
    for (int i = 0; i < 100; i++) {
        values[i % 10] = i;
        ws_pool_submit(&ws_pool, print_task, &values[i % 10]);
    }

    ws_pool_wait(&ws_pool);
    printf("Tasks stolen: %lu\n", ws_pool.steals);
    ws_pool_destroy(&ws_pool);

    // 2.4.26.g: Fork-Join
    printf("\n=== Fork-Join ===\n");
    ws_pool_create(&ws_pool, 4);
    fork_join_t fj = {.pool = &ws_pool};

    for (int i = 0; i < 8; i++) {
        fj_fork(&fj, print_task, &values[i]);
    }
    fj_join(&fj);
    ws_pool_destroy(&ws_pool);

    // ============== Parallel Patterns ==============
    pool_create(&pool, 4);

    // 2.4.27.a: Parallel for
    printf("\n=== Parallel For ===\n");
    int arr[1000];
    parallel_for(&pool, 0, 1000, (parallel_for_func_t)({
        void f(int i, void *a) { ((int*)a)[i] = i * 2; }; f;
    }), arr);

    // 2.4.27.b: Parallel map
    printf("\n=== Parallel Map ===\n");
    void *inputs[100];
    for (int i = 0; i < 100; i++) {
        inputs[i] = malloc(sizeof(int));
        *(int*)inputs[i] = i;
    }

    void **outputs = parallel_map(&pool, inputs, 100, compute_square);

    // 2.4.27.c: Parallel reduce (sum)
    printf("\n=== Parallel Reduce ===\n");
    void *sum_func(void *a, void *b) {
        int *result = malloc(sizeof(int));
        *result = *(int*)a + *(int*)b;
        return result;
    }
    int zero = 0;
    int *total = parallel_reduce(&pool, outputs, 100, sum_func, &zero);
    printf("Sum of squares 0-99: %d\n", *total);

    // 2.4.27.d: Parallel prefix sum
    printf("\n=== Parallel Prefix ===\n");
    int prefix_arr[] = {1, 2, 3, 4, 5, 6, 7, 8};
    parallel_prefix_sum(&pool, prefix_arr, 8);
    printf("Prefix sum: ");
    for (int i = 0; i < 8; i++) printf("%d ", prefix_arr[i]);
    printf("\n");  // 1 3 6 10 15 21 28 36

    // 2.4.27.e: Pipeline
    printf("\n=== Pipeline ===\n");
    pipeline_t pipe;
    pipeline_create(&pipe);

    void *stage1(void *x) {
        int *v = malloc(sizeof(int));
        *v = *(int*)x * 2;
        return v;
    }
    void *stage2(void *x) {
        int *v = x;
        *v += 10;
        return v;
    }
    void *stage3(void *x) {
        int *v = x;
        printf("Pipeline result: %d\n", *v);
        return v;
    }

    pipeline_add_stage(&pipe, stage1);
    pipeline_add_stage(&pipe, stage2);
    pipeline_add_stage(&pipe, stage3);
    pipeline_start(&pipe);

    for (int i = 0; i < 5; i++) {
        pipeline_submit(&pipe, &i);
    }

    pipeline_stop(&pipe);
    pipeline_destroy(&pipe);

    // Cleanup
    for (int i = 0; i < 100; i++) {
        free(inputs[i]);
        free(outputs[i]);
    }
    free(outputs);
    free(total);

    pool_shutdown(&pool, true);
    pool_destroy(&pool);

    // Benchmark
    printf("\n=== Benchmark ===\n");
    parallel_benchmark_t bench;
    benchmark_parallel_for(4, 10000000, &bench);
    printf("Speedup: %.2fx, Efficiency: %.1f%%\n",
           bench.speedup, bench.efficiency * 100);

    return 0;
}
```

---

## Tests Moulinette

```rust
// Thread pool tests
#[test] fn test_pool_create()            // 2.4.25.a-b
#[test] fn test_pool_submit()            // 2.4.25.d
#[test] fn test_worker_loop()            // 2.4.25.e
#[test] fn test_pool_shutdown()          // 2.4.25.f
#[test] fn test_pool_wait()              // 2.4.25.g
#[test] fn test_pool_resize()            // 2.4.25.h
#[test] fn test_future()

// Work stealing tests
#[test] fn test_ws_deque()               // 2.4.26.b-d
#[test] fn test_chase_lev()              // 2.4.26.e
#[test] fn test_work_stealing()          // 2.4.26.a,f
#[test] fn test_fork_join()              // 2.4.26.g

// Parallel pattern tests
#[test] fn test_parallel_for()           // 2.4.27.a
#[test] fn test_parallel_map()           // 2.4.27.b
#[test] fn test_parallel_reduce()        // 2.4.27.c
#[test] fn test_parallel_prefix()        // 2.4.27.d
#[test] fn test_pipeline()               // 2.4.27.e
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Thread Pool (2.4.25) | 35 |
| Work Stealing (2.4.26) | 30 |
| Parallel Patterns (2.4.27) | 35 |
| **Total** | **100** |

---

## Fichiers

```
ex08/
├── thread_pool.h
├── thread_pool.c
├── work_stealing.c
├── parallel_patterns.c
├── pipeline.c
└── Makefile
```
