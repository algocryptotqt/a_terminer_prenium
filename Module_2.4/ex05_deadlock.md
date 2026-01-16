# ex05: Deadlock Detection & Prevention

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Difficile
**Duree**: 7h
**Score qualite**: 97/100

## Concepts Couverts

### 2.4.16: Deadlock Concepts (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Deadlock | Circular wait |
| b | Coffman conditions | All four required |
| c | Mutual exclusion | Resource exclusive |
| d | Hold and wait | Holding, requesting |
| e | No preemption | Can't take away |
| f | Circular wait | Chain of waiting |
| g | Resource allocation graph | Visualization |
| h | Cycle = deadlock | Single instance |

### 2.4.17: Deadlock Prevention (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Deny mutual exclusion | Share when possible |
| b | Deny hold and wait | Request all at once |
| c | Allow preemption | Take resources |
| d | Deny circular wait | Lock ordering |
| e | Lock hierarchy | Global ordering |
| f | Try-lock pattern | Back off on failure |
| g | Practical approach | Ordering + timeouts |

### 2.4.18: Deadlock Avoidance (9 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Safe state | Can complete all |
| b | Unsafe state | Might deadlock |
| c | Banker's algorithm | Check safety |
| d | Available | Free resources |
| e | Maximum | Max needs |
| f | Allocation | Current allocation |
| g | Need | Max - Allocation |
| h | Safety algorithm | Find safe sequence |
| i | Request algorithm | Check before grant |

### 2.4.19: Deadlock Detection and Recovery (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Detection algorithm | Find cycles |
| b | When to detect | Periodically or on wait |
| c | Recovery: terminate | Kill deadlocked |
| d | Kill all | Drastic |
| e | Kill one by one | Until resolved |
| f | Recovery: preempt | Take resources |
| g | Rollback | Checkpoint and restore |
| h | Victim selection | Minimize cost |

---

## Sujet

Implementer un systeme complet de detection, prevention et resolution de deadlocks.

### Structures

```c
#include <pthread.h>
#include <stdbool.h>

#define MAX_PROCESSES 64
#define MAX_RESOURCES 64

// 2.4.16.g: Resource Allocation Graph
typedef struct {
    int process;
    int resource;
} edge_t;

typedef struct {
    int num_processes;
    int num_resources;
    int *instances;              // Instances per resource type

    // Edges: process -> resource (request), resource -> process (assignment)
    bool request[MAX_PROCESSES][MAX_RESOURCES];
    bool assignment[MAX_RESOURCES][MAX_PROCESSES];

    // For visualization
    edge_t *request_edges;
    edge_t *assign_edges;
    size_t num_request_edges;
    size_t num_assign_edges;
} rag_t;

// 2.4.18.c-g: Banker's Algorithm data
typedef struct {
    int num_processes;
    int num_resources;

    int available[MAX_RESOURCES];              // d: Free resources
    int maximum[MAX_PROCESSES][MAX_RESOURCES]; // e: Max needs
    int allocation[MAX_PROCESSES][MAX_RESOURCES]; // f: Current allocation
    int need[MAX_PROCESSES][MAX_RESOURCES];    // g: Need = Max - Allocation

    bool finished[MAX_PROCESSES];              // For safety algorithm
    int safe_sequence[MAX_PROCESSES];          // h: Safe sequence
    int sequence_length;
} banker_t;

// 2.4.17.e: Lock hierarchy
typedef struct {
    pthread_mutex_t mutex;
    int order;                   // Global order number
    const char *name;
    pthread_t owner;
    int held_count;
} ordered_lock_t;

typedef struct {
    ordered_lock_t *locks;
    size_t num_locks;
    int *thread_max_order;       // Max order held by each thread
    pthread_mutex_t meta_lock;
} lock_hierarchy_t;

// Deadlock detector
typedef struct {
    rag_t *rag;
    banker_t *banker;
    lock_hierarchy_t *hierarchy;

    // Detection state
    bool deadlock_detected;
    int *deadlocked_processes;
    int num_deadlocked;

    // Stats
    uint64_t checks_performed;
    uint64_t deadlocks_found;
    uint64_t deadlocks_prevented;
    uint64_t deadlocks_recovered;
} deadlock_detector_t;

// 2.4.16.b: Coffman conditions checker
typedef struct {
    bool mutual_exclusion;       // c
    bool hold_and_wait;          // d
    bool no_preemption;          // e
    bool circular_wait;          // f
} coffman_state_t;
```

### API

```c
// ============== RESOURCE ALLOCATION GRAPH ==============
// 2.4.16.g

int rag_init(rag_t *rag, int processes, int resources, int *instances);
void rag_destroy(rag_t *rag);
void rag_request_edge(rag_t *rag, int process, int resource);
void rag_assign_edge(rag_t *rag, int resource, int process);
void rag_remove_request(rag_t *rag, int process, int resource);
void rag_remove_assign(rag_t *rag, int resource, int process);

// 2.4.16.h: Cycle detection
bool rag_has_cycle(rag_t *rag);                              // h
int *rag_find_cycle(rag_t *rag, int *cycle_length);
void rag_print(rag_t *rag);

// ============== BANKER'S ALGORITHM ==============
// 2.4.18.c-i

int banker_init(banker_t *b, int processes, int resources, int *available);
void banker_destroy(banker_t *b);
void banker_set_max(banker_t *b, int process, int *max);     // e
void banker_allocate(banker_t *b, int process, int *alloc);  // f
void banker_update_need(banker_t *b);                         // g

// 2.4.18.a-b,h: Safety check
bool banker_is_safe(banker_t *b);                            // a vs b
bool banker_find_safe_sequence(banker_t *b, int *sequence);  // h

// 2.4.18.i: Request algorithm
bool banker_request(banker_t *b, int process, int *request); // i
void banker_release(banker_t *b, int process, int *release);

void banker_print_state(banker_t *b);

// ============== DEADLOCK PREVENTION ==============
// 2.4.17

// 2.4.17.b: Request all at once
typedef struct {
    int *resources;
    int count;
} resource_request_t;

bool prevent_hold_wait(resource_request_t *req);

// 2.4.17.d-e: Lock ordering / hierarchy
int hierarchy_init(lock_hierarchy_t *h, int num_locks);
void hierarchy_destroy(lock_hierarchy_t *h);
int hierarchy_register_lock(lock_hierarchy_t *h, ordered_lock_t *lock, int order, const char *name);
int hierarchy_lock(lock_hierarchy_t *h, ordered_lock_t *lock);    // Enforces order
int hierarchy_unlock(lock_hierarchy_t *h, ordered_lock_t *lock);
bool hierarchy_check_order(lock_hierarchy_t *h, ordered_lock_t *lock); // Would violate?

// 2.4.17.f: Try-lock with backoff
int trylock_with_backoff(pthread_mutex_t *locks[], int num_locks, int max_attempts);

// 2.4.17.g: Timeout-based
int lock_with_timeout(pthread_mutex_t *lock, int timeout_ms);

// ============== DEADLOCK DETECTION ==============
// 2.4.19.a-b

int detector_init(deadlock_detector_t *d);
void detector_destroy(deadlock_detector_t *d);

// 2.4.19.a: Detection
bool detector_check(deadlock_detector_t *d);
bool detector_check_rag(deadlock_detector_t *d);
bool detector_check_banker(deadlock_detector_t *d);

// 2.4.19.b: When to detect
void detector_set_periodic(deadlock_detector_t *d, int interval_ms);
void detector_check_on_wait(deadlock_detector_t *d, int process, int resource);

// ============== DEADLOCK RECOVERY ==============
// 2.4.19.c-h

// 2.4.19.c-e: Process termination
int recovery_kill_all(deadlock_detector_t *d);               // d
int recovery_kill_one(deadlock_detector_t *d);               // e
int recovery_kill_by_priority(deadlock_detector_t *d);

// 2.4.19.f: Resource preemption
int recovery_preempt(deadlock_detector_t *d, int process, int resource);

// 2.4.19.g: Rollback
typedef struct {
    int process;
    int *allocation;
    void *state;
    size_t state_size;
} checkpoint_t;

int checkpoint_save(checkpoint_t *cp, int process, void *state, size_t size);
int checkpoint_restore(checkpoint_t *cp);
void checkpoint_free(checkpoint_t *cp);

// 2.4.19.h: Victim selection
typedef struct {
    int priority;
    int resources_held;
    int runtime;
    int checkpoints;
} process_cost_t;

int select_victim(deadlock_detector_t *d, process_cost_t *costs);

// ============== COFFMAN CONDITIONS ==============
// 2.4.16.b-f

void coffman_check(coffman_state_t *state);
bool coffman_all_present(coffman_state_t *state);
void coffman_print(coffman_state_t *state);
```

---

## Exemple

```c
#include "deadlock.h"

int main(void) {
    // ============== RAG Demo ==============
    printf("=== Resource Allocation Graph ===\n");
    rag_t rag;
    int instances[] = {1, 1, 1};  // Single instance resources
    rag_init(&rag, 3, 3, instances);

    // P0 holds R0, wants R1
    rag_assign_edge(&rag, 0, 0);   // R0 -> P0
    rag_request_edge(&rag, 0, 1);  // P0 -> R1

    // P1 holds R1, wants R2
    rag_assign_edge(&rag, 1, 1);   // R1 -> P1
    rag_request_edge(&rag, 1, 2);  // P1 -> R2

    // P2 holds R2, wants R0 -> DEADLOCK!
    rag_assign_edge(&rag, 2, 2);   // R2 -> P2
    rag_request_edge(&rag, 2, 0);  // P2 -> R0

    rag_print(&rag);

    // 2.4.16.h: Detect cycle
    if (rag_has_cycle(&rag)) {
        printf("DEADLOCK: Cycle detected!\n");
        int len;
        int *cycle = rag_find_cycle(&rag, &len);
        printf("Cycle: ");
        for (int i = 0; i < len; i++) printf("P%d -> ", cycle[i]);
        printf("\n");
        free(cycle);
    }
    rag_destroy(&rag);

    // ============== Banker's Algorithm ==============
    printf("\n=== Banker's Algorithm ===\n");
    banker_t banker;
    int avail[] = {3, 3, 2};
    banker_init(&banker, 5, 3, avail);

    // Set maximum needs
    int max0[] = {7, 5, 3}; banker_set_max(&banker, 0, max0);
    int max1[] = {3, 2, 2}; banker_set_max(&banker, 1, max1);
    int max2[] = {9, 0, 2}; banker_set_max(&banker, 2, max2);
    int max3[] = {2, 2, 2}; banker_set_max(&banker, 3, max3);
    int max4[] = {4, 3, 3}; banker_set_max(&banker, 4, max4);

    // Current allocations
    int alloc0[] = {0, 1, 0}; banker_allocate(&banker, 0, alloc0);
    int alloc1[] = {2, 0, 0}; banker_allocate(&banker, 1, alloc1);
    int alloc2[] = {3, 0, 2}; banker_allocate(&banker, 2, alloc2);
    int alloc3[] = {2, 1, 1}; banker_allocate(&banker, 3, alloc3);
    int alloc4[] = {0, 0, 2}; banker_allocate(&banker, 4, alloc4);

    banker_update_need(&banker);
    banker_print_state(&banker);

    // 2.4.18.a-b,h: Check if safe
    if (banker_is_safe(&banker)) {
        printf("System is in SAFE state\n");
        int seq[5];
        banker_find_safe_sequence(&banker, seq);
        printf("Safe sequence: ");
        for (int i = 0; i < 5; i++) printf("P%d ", seq[i]);
        printf("\n");
    } else {
        printf("System is in UNSAFE state!\n");
    }

    // 2.4.18.i: Request algorithm
    int req1[] = {1, 0, 2};
    if (banker_request(&banker, 1, req1)) {
        printf("Request from P1 granted\n");
    } else {
        printf("Request from P1 denied (would be unsafe)\n");
    }

    banker_destroy(&banker);

    // ============== Lock Hierarchy (Prevention) ==============
    printf("\n=== Lock Hierarchy ===\n");
    lock_hierarchy_t hier;
    hierarchy_init(&hier, 3);

    ordered_lock_t lock_a, lock_b, lock_c;
    hierarchy_register_lock(&hier, &lock_a, 1, "lock_A");
    hierarchy_register_lock(&hier, &lock_b, 2, "lock_B");
    hierarchy_register_lock(&hier, &lock_c, 3, "lock_C");

    // Correct order: A -> B -> C
    hierarchy_lock(&hier, &lock_a);
    hierarchy_lock(&hier, &lock_b);
    hierarchy_lock(&hier, &lock_c);

    // This would fail: trying to lock lower order while holding higher
    // hierarchy_lock(&hier, &lock_a);  // ERROR: violates hierarchy

    hierarchy_unlock(&hier, &lock_c);
    hierarchy_unlock(&hier, &lock_b);
    hierarchy_unlock(&hier, &lock_a);

    hierarchy_destroy(&hier);

    // ============== Try-lock with Backoff ==============
    printf("\n=== Try-lock Backoff ===\n");
    pthread_mutex_t m1 = PTHREAD_MUTEX_INITIALIZER;
    pthread_mutex_t m2 = PTHREAD_MUTEX_INITIALIZER;
    pthread_mutex_t *locks[] = {&m1, &m2};

    if (trylock_with_backoff(locks, 2, 5) == 0) {
        printf("Acquired both locks\n");
        pthread_mutex_unlock(&m2);
        pthread_mutex_unlock(&m1);
    }

    // ============== Coffman Conditions ==============
    printf("\n=== Coffman Conditions ===\n");
    coffman_state_t coffman = {
        .mutual_exclusion = true,
        .hold_and_wait = true,
        .no_preemption = true,
        .circular_wait = true
    };

    coffman_print(&coffman);
    if (coffman_all_present(&coffman)) {
        printf("All Coffman conditions present - DEADLOCK POSSIBLE!\n");
    }

    return 0;
}
```

---

## Tests Moulinette

```rust
// RAG tests
#[test] fn test_rag_edges()              // 2.4.16.g
#[test] fn test_rag_cycle()              // 2.4.16.h
#[test] fn test_coffman_conditions()     // 2.4.16.b-f

// Prevention tests
#[test] fn test_deny_hold_wait()         // 2.4.17.b
#[test] fn test_lock_ordering()          // 2.4.17.d
#[test] fn test_lock_hierarchy()         // 2.4.17.e
#[test] fn test_trylock_backoff()        // 2.4.17.f
#[test] fn test_lock_timeout()           // 2.4.17.g

// Banker's algorithm tests
#[test] fn test_banker_safe()            // 2.4.18.a
#[test] fn test_banker_unsafe()          // 2.4.18.b
#[test] fn test_banker_algorithm()       // 2.4.18.c-g
#[test] fn test_safety_algorithm()       // 2.4.18.h
#[test] fn test_request_algorithm()      // 2.4.18.i

// Detection/Recovery tests
#[test] fn test_detection()              // 2.4.19.a
#[test] fn test_recovery_terminate()     // 2.4.19.c-e
#[test] fn test_recovery_preempt()       // 2.4.19.f
#[test] fn test_checkpoint_rollback()    // 2.4.19.g
#[test] fn test_victim_selection()       // 2.4.19.h
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Deadlock concepts & RAG (2.4.16) | 20 |
| Prevention techniques (2.4.17) | 25 |
| Banker's algorithm (2.4.18) | 30 |
| Detection & Recovery (2.4.19) | 25 |
| **Total** | **100** |

---

## Fichiers

```
ex05/
├── deadlock.h
├── rag.c
├── banker.c
├── prevention.c
├── detection.c
├── recovery.c
└── Makefile
```
