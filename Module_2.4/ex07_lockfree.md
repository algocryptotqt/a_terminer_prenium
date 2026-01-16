# ex07: Lock-Free Data Structures

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Tres difficile
**Duree**: 8h
**Score qualite**: 98/100

## Concepts Couverts

### 2.4.22: Lock-Free Programming (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Lock-free | At least one makes progress |
| b | Wait-free | All make bounded progress |
| c | Obstruction-free | Progress in isolation |
| d | CAS loop | Compare-and-swap pattern |
| e | ABA problem | A→B→A fools CAS |
| f | Tagged pointers | Counter + pointer |
| g | Hazard pointers | Safe memory reclamation |
| h | Epoch-based | Reclamation technique |

### 2.4.23: Lock-Free Stack (6 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Treiber stack | Classic lock-free |
| b | Push | CAS on head |
| c | Pop | CAS on head |
| d | ABA problem | In pop |
| e | Solution | Tagged pointer |
| f | Implementation | Complete |

### 2.4.24: Lock-Free Queue (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Michael-Scott queue | MPMC |
| b | Head and tail | Both atomic |
| c | Dummy node | Simplifies |
| d | Enqueue | CAS on tail |
| e | Dequeue | CAS on head |
| f | Helping | Fix lagging tail |
| g | SPSC queue | Simpler |
| h | Ring buffer | SPSC common |

---

## Sujet

Implementer des structures de donnees lock-free avec gestion correcte de la memoire.

### Structures

```c
#include <stdatomic.h>
#include <stdbool.h>
#include <stdint.h>

// 2.4.22.f: Tagged pointer to solve ABA
typedef struct {
    uintptr_t ptr : 48;
    uint16_t tag : 16;
} tagged_ptr_t;

// Pack/unpack tagged pointers
#define MAKE_TAGGED(ptr, tag) ((tagged_ptr_t){(uintptr_t)(ptr), (tag)})
#define GET_PTR(tp) ((void*)((tp).ptr))
#define GET_TAG(tp) ((tp).tag)

// 2.4.23: Treiber Stack node
typedef struct lf_stack_node {
    void *data;
    _Atomic(struct lf_stack_node*) next;
} lf_stack_node_t;

// 2.4.23.a: Treiber Stack
typedef struct {
    _Atomic(tagged_ptr_t) head;  // f: Tagged to prevent ABA
    _Atomic size_t size;
    uint64_t push_count;
    uint64_t pop_count;
    uint64_t cas_failures;
} lf_stack_t;

// 2.4.24: Michael-Scott Queue node
typedef struct lf_queue_node {
    void *data;
    _Atomic(struct lf_queue_node*) next;
} lf_queue_node_t;

// 2.4.24.a: Michael-Scott Queue
typedef struct {
    _Atomic(lf_queue_node_t*) head;  // b: Both atomic
    _Atomic(lf_queue_node_t*) tail;  // b
    _Atomic size_t size;
    uint64_t enqueue_count;
    uint64_t dequeue_count;
    uint64_t helping_count;          // f: Times we helped fix tail
} lf_queue_t;

// 2.4.24.g-h: SPSC Ring Buffer
typedef struct {
    void **buffer;
    size_t capacity;
    size_t mask;                     // capacity - 1 for power of 2
    _Atomic size_t head;             // Consumer reads here
    _Atomic size_t tail;             // Producer writes here
    char pad1[64];                   // Prevent false sharing
    size_t cached_head;              // Producer's cached head
    char pad2[64];
    size_t cached_tail;              // Consumer's cached tail
} spsc_queue_t;

// 2.4.22.g: Hazard Pointers for safe reclamation
#define MAX_HAZARD_POINTERS 128
#define MAX_THREADS 64

typedef struct {
    _Atomic(void*) hp[MAX_HAZARD_POINTERS];
    int per_thread;
} hazard_domain_t;

typedef struct {
    void **retired;
    size_t retired_count;
    size_t retired_capacity;
    void (*deleter)(void*);
} retire_list_t;

// 2.4.22.h: Epoch-based reclamation
typedef struct {
    _Atomic uint64_t global_epoch;
    _Atomic uint64_t thread_epochs[MAX_THREADS];
    void **garbage[3];               // One per epoch
    size_t garbage_count[3];
} epoch_domain_t;
```

### API

```c
// ============== LOCK-FREE STACK ==============
// 2.4.23

int lf_stack_init(lf_stack_t *s);
void lf_stack_destroy(lf_stack_t *s, void (*free_data)(void*));

// 2.4.23.b: Push (CAS on head)
void lf_stack_push(lf_stack_t *s, void *data);

// 2.4.23.c-e: Pop with ABA protection
void *lf_stack_pop(lf_stack_t *s);

bool lf_stack_is_empty(lf_stack_t *s);
size_t lf_stack_size(lf_stack_t *s);

// ============== LOCK-FREE QUEUE ==============
// 2.4.24.a-f

int lf_queue_init(lf_queue_t *q);
void lf_queue_destroy(lf_queue_t *q, void (*free_data)(void*));

// 2.4.24.d: Enqueue (CAS on tail)
void lf_queue_enqueue(lf_queue_t *q, void *data);

// 2.4.24.e-f: Dequeue (CAS on head, with helping)
void *lf_queue_dequeue(lf_queue_t *q);

bool lf_queue_is_empty(lf_queue_t *q);
size_t lf_queue_size(lf_queue_t *q);

// ============== SPSC QUEUE ==============
// 2.4.24.g-h

int spsc_init(spsc_queue_t *q, size_t capacity);  // Must be power of 2
void spsc_destroy(spsc_queue_t *q);
bool spsc_enqueue(spsc_queue_t *q, void *data);   // Returns false if full
void *spsc_dequeue(spsc_queue_t *q);              // Returns NULL if empty
size_t spsc_size(spsc_queue_t *q);

// ============== HAZARD POINTERS ==============
// 2.4.22.g

int hp_domain_init(hazard_domain_t *d, int per_thread);
void hp_domain_destroy(hazard_domain_t *d);

// Protect a pointer
void *hp_protect(hazard_domain_t *d, int thread_id, int hp_index, _Atomic(void*) *ptr);
void hp_clear(hazard_domain_t *d, int thread_id, int hp_index);

// Retire a pointer (defer deletion)
void hp_retire(hazard_domain_t *d, retire_list_t *list, void *ptr);
void hp_scan(hazard_domain_t *d, retire_list_t *list);

// ============== EPOCH-BASED RECLAMATION ==============
// 2.4.22.h

int epoch_domain_init(epoch_domain_t *d);
void epoch_domain_destroy(epoch_domain_t *d);

void epoch_enter(epoch_domain_t *d, int thread_id);
void epoch_exit(epoch_domain_t *d, int thread_id);
void epoch_retire(epoch_domain_t *d, void *ptr);
void epoch_gc(epoch_domain_t *d);

// ============== PROGRESS GUARANTEES ==============
// 2.4.22.a-c

typedef enum {
    PROGRESS_BLOCKING,           // Uses locks
    PROGRESS_OBSTRUCTION_FREE,   // c
    PROGRESS_LOCK_FREE,          // a
    PROGRESS_WAIT_FREE           // b
} progress_guarantee_t;

progress_guarantee_t get_stack_guarantee(void);
progress_guarantee_t get_queue_guarantee(void);

// ============== ABA DEMONSTRATION ==============
// 2.4.22.e

typedef struct {
    bool aba_occurred;
    int aba_count;
    void *expected_ptr;
    void *actual_ptr;
} aba_demo_t;

void aba_demo_show_problem(aba_demo_t *demo);
void aba_demo_show_solution(aba_demo_t *demo);

// ============== BENCHMARKS ==============

typedef struct {
    double ops_per_sec;
    double avg_latency_ns;
    uint64_t cas_failures;
    uint64_t helping_ops;
} lf_benchmark_t;

void benchmark_stack(int threads, int ops, lf_benchmark_t *result);
void benchmark_queue(int threads, int ops, lf_benchmark_t *result);
void benchmark_spsc(int ops, lf_benchmark_t *result);
```

---

## Exemple

```c
#include "lockfree.h"

// ============== TREIBER STACK DEMO ==============
void stack_demo(void) {
    printf("=== Lock-Free Stack (Treiber) ===\n");
    lf_stack_t stack;
    lf_stack_init(&stack);

    // 2.4.23.b: Push operations
    for (int i = 0; i < 10; i++) {
        int *val = malloc(sizeof(int));
        *val = i;
        lf_stack_push(&stack, val);
    }
    printf("Pushed 10 items, size = %zu\n", lf_stack_size(&stack));

    // 2.4.23.c: Pop operations
    void *data;
    while ((data = lf_stack_pop(&stack)) != NULL) {
        printf("Popped: %d\n", *(int*)data);
        free(data);
    }

    lf_stack_destroy(&stack, NULL);
}

// 2.4.23.b: Push implementation (CAS loop)
void lf_stack_push_impl(lf_stack_t *s, void *data) {
    lf_stack_node_t *node = malloc(sizeof(lf_stack_node_t));
    node->data = data;

    tagged_ptr_t old_head, new_head;

    // 2.4.22.d: CAS loop pattern
    do {
        old_head = atomic_load(&s->head);
        node->next = GET_PTR(old_head);
        new_head = MAKE_TAGGED(node, GET_TAG(old_head) + 1);
    } while (!atomic_compare_exchange_weak(&s->head, &old_head, new_head));
    // 2.4.22.f: Tag incremented to prevent ABA
}

// ============== MICHAEL-SCOTT QUEUE DEMO ==============
void queue_demo(void) {
    printf("\n=== Lock-Free Queue (Michael-Scott) ===\n");
    lf_queue_t queue;
    lf_queue_init(&queue);

    // 2.4.24.d: Enqueue
    for (int i = 0; i < 10; i++) {
        int *val = malloc(sizeof(int));
        *val = i;
        lf_queue_enqueue(&queue, val);
    }

    // 2.4.24.e: Dequeue
    void *data;
    while ((data = lf_queue_dequeue(&queue)) != NULL) {
        printf("Dequeued: %d\n", *(int*)data);
        free(data);
    }

    printf("Helping operations: %lu\n", queue.helping_count);
    lf_queue_destroy(&queue, NULL);
}

// ============== SPSC QUEUE DEMO ==============
void *spsc_producer(void *arg) {
    spsc_queue_t *q = arg;
    for (int i = 0; i < 1000000; i++) {
        while (!spsc_enqueue(q, (void*)(intptr_t)(i + 1))) {
            // Queue full, spin
        }
    }
    return NULL;
}

void *spsc_consumer(void *arg) {
    spsc_queue_t *q = arg;
    int count = 0;
    while (count < 1000000) {
        void *data = spsc_dequeue(q);
        if (data != NULL) {
            count++;
        }
    }
    return NULL;
}

void spsc_demo(void) {
    printf("\n=== SPSC Ring Buffer ===\n");
    spsc_queue_t queue;
    spsc_init(&queue, 1024);  // Must be power of 2

    pthread_t prod, cons;
    pthread_create(&prod, NULL, spsc_producer, &queue);
    pthread_create(&cons, NULL, spsc_consumer, &queue);

    pthread_join(prod, NULL);
    pthread_join(cons, NULL);

    printf("SPSC: 1M items transferred\n");
    spsc_destroy(&queue);
}

// ============== ABA PROBLEM DEMO ==============
void aba_demo(void) {
    printf("\n=== ABA Problem ===\n");

    // 2.4.22.e: The problem
    // Thread 1: Reads head = A
    // Thread 1: (preempted)
    // Thread 2: Pops A
    // Thread 2: Pops B
    // Thread 2: Pushes A back (same address!)
    // Thread 1: CAS(head, A, new) SUCCEEDS but stack is corrupted!

    aba_demo_t demo;
    aba_demo_show_problem(&demo);
    printf("ABA occurrences: %d\n", demo.aba_count);

    // 2.4.22.f: Solution with tagged pointers
    printf("\nWith tagged pointers:\n");
    aba_demo_show_solution(&demo);
    printf("ABA prevented: tag mismatch detected\n");
}

// ============== HAZARD POINTERS DEMO ==============
void hazard_demo(void) {
    printf("\n=== Hazard Pointers ===\n");
    hazard_domain_t hp_domain;
    hp_domain_init(&hp_domain, 2);  // 2 HPs per thread

    retire_list_t retire_list = {
        .retired = malloc(1000 * sizeof(void*)),
        .retired_count = 0,
        .retired_capacity = 1000,
        .deleter = free
    };

    // Protect a pointer before dereferencing
    _Atomic(void*) shared_ptr;
    void *local = hp_protect(&hp_domain, 0, 0, &shared_ptr);

    // Safe to use local now - won't be freed
    // ... use local ...

    hp_clear(&hp_domain, 0, 0);

    // Retire old pointers
    void *old = malloc(100);
    hp_retire(&hp_domain, &retire_list, old);

    // Periodically scan and free
    hp_scan(&hp_domain, &retire_list);

    free(retire_list.retired);
    hp_domain_destroy(&hp_domain);
}

int main(void) {
    stack_demo();
    queue_demo();
    spsc_demo();
    aba_demo();
    hazard_demo();

    // Benchmark
    printf("\n=== Benchmarks ===\n");
    lf_benchmark_t bench;

    benchmark_stack(4, 100000, &bench);
    printf("Stack: %.0f ops/sec, CAS failures: %lu\n",
           bench.ops_per_sec, bench.cas_failures);

    benchmark_queue(4, 100000, &bench);
    printf("Queue: %.0f ops/sec, Helping: %lu\n",
           bench.ops_per_sec, bench.helping_ops);

    benchmark_spsc(1000000, &bench);
    printf("SPSC: %.0f ops/sec\n", bench.ops_per_sec);

    return 0;
}
```

---

## Tests Moulinette

```rust
// Lock-free concepts
#[test] fn test_progress_guarantee()     // 2.4.22.a-c
#[test] fn test_cas_loop()               // 2.4.22.d
#[test] fn test_aba_problem()            // 2.4.22.e
#[test] fn test_tagged_pointers()        // 2.4.22.f
#[test] fn test_hazard_pointers()        // 2.4.22.g
#[test] fn test_epoch_reclamation()      // 2.4.22.h

// Stack tests
#[test] fn test_treiber_stack()          // 2.4.23.a
#[test] fn test_stack_push()             // 2.4.23.b
#[test] fn test_stack_pop()              // 2.4.23.c-e
#[test] fn test_stack_concurrent()       // 2.4.23.f

// Queue tests
#[test] fn test_ms_queue()               // 2.4.24.a
#[test] fn test_queue_enqueue()          // 2.4.24.d
#[test] fn test_queue_dequeue()          // 2.4.24.e
#[test] fn test_queue_helping()          // 2.4.24.f
#[test] fn test_spsc_queue()             // 2.4.24.g-h
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Lock-free concepts (2.4.22) | 30 |
| Treiber Stack (2.4.23) | 25 |
| Michael-Scott Queue (2.4.24.a-f) | 30 |
| SPSC Queue (2.4.24.g-h) | 15 |
| **Total** | **100** |

---

## Fichiers

```
ex07/
├── lockfree.h
├── lf_stack.c
├── lf_queue.c
├── spsc_queue.c
├── hazard_pointers.c
├── epoch_reclaim.c
└── Makefile
```
