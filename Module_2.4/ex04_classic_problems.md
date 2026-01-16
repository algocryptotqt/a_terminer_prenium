# ex04: Classic Synchronization Problems

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Difficile
**Duree**: 6h
**Score qualite**: 97/100

## Concepts Couverts

### 2.4.13: Producer-Consumer (9 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Problem | Bounded buffer |
| b | Producers | Add items |
| c | Consumers | Remove items |
| d | Buffer full | Producers wait |
| e | Buffer empty | Consumers wait |
| f | Mutex | Protect buffer |
| g | Two conditions | not_full, not_empty |
| h | Implementation | Complete solution |
| i | Semaphore alternative | empty, full, mutex |

### 2.4.14: Readers-Writers (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Problem | Concurrent read, exclusive write |
| b | First variant | Readers preference |
| c | Second variant | Writers preference |
| d | Third variant | Fair |
| e | Starvation | Possible in 1st and 2nd |
| f | Implementation | All variants |
| g | rwlock solution | Simple alternative |

### 2.4.15: Dining Philosophers (8 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Problem | 5 philosophers, 5 forks |
| b | Deadlock | All pick up left fork |
| c | Solution 1 | At most 4 at table |
| d | Solution 2 | Asymmetric (one picks right first) |
| e | Solution 3 | Resource ordering |
| f | Solution 4 | Waiter (central mutex) |
| g | Solution 5 | Chandy/Misra |
| h | Implementation | Multiple solutions |

---

## Sujet

Implementer les trois problemes classiques de synchronisation avec toutes leurs variantes.

### Structures

```c
#include <pthread.h>
#include <semaphore.h>
#include <stdbool.h>

// ============== PRODUCER-CONSUMER ==============
// 2.4.13: Bounded buffer

typedef struct {
    int *buffer;
    size_t capacity;
    size_t count;
    size_t head, tail;

    // 2.4.13.f-g: Mutex + conditions solution
    pthread_mutex_t mutex;
    pthread_cond_t not_full;    // g: Producers wait
    pthread_cond_t not_empty;   // g: Consumers wait

    // 2.4.13.i: Semaphore solution
    sem_t sem_empty;            // Counts empty slots
    sem_t sem_full;             // Counts full slots
    sem_t sem_mutex;            // Protect buffer

    // Stats
    uint64_t produced;
    uint64_t consumed;
    uint64_t producer_waits;
    uint64_t consumer_waits;
} pc_buffer_t;

// ============== READERS-WRITERS ==============
// 2.4.14: Three variants

typedef enum {
    RW_READERS_PREFERENCE,      // b: Readers first
    RW_WRITERS_PREFERENCE,      // c: Writers first
    RW_FAIR                     // d: FIFO order
} rw_variant_t;

typedef struct {
    int readers_count;
    int writers_count;
    int waiting_readers;
    int waiting_writers;

    pthread_mutex_t mutex;
    pthread_cond_t can_read;
    pthread_cond_t can_write;

    // For fair variant
    int order_number;
    int next_to_serve;

    rw_variant_t variant;

    // Stats
    uint64_t reads;
    uint64_t writes;
    uint64_t reader_starvation;  // e
    uint64_t writer_starvation;  // e
} rw_lock_t;

// ============== DINING PHILOSOPHERS ==============
// 2.4.15: Multiple solutions

#define NUM_PHILOSOPHERS 5

typedef enum {
    THINKING,
    HUNGRY,
    EATING
} philosopher_state_t;

typedef enum {
    DP_SOLUTION_LIMIT,          // c: At most 4 at table
    DP_SOLUTION_ASYMMETRIC,     // d: One picks right first
    DP_SOLUTION_ORDERING,       // e: Resource ordering
    DP_SOLUTION_WAITER,         // f: Central arbitrator
    DP_SOLUTION_CHANDY_MISRA    // g: Token-based
} dp_solution_t;

typedef struct {
    philosopher_state_t state[NUM_PHILOSOPHERS];
    pthread_mutex_t mutex;
    pthread_cond_t can_eat[NUM_PHILOSOPHERS];

    // For different solutions
    sem_t room;                 // c: Limit philosophers
    pthread_mutex_t forks[NUM_PHILOSOPHERS]; // d,e: Fork mutexes
    pthread_mutex_t waiter;     // f: Waiter mutex

    // g: Chandy/Misra tokens
    bool fork_dirty[NUM_PHILOSOPHERS];
    bool fork_owner[NUM_PHILOSOPHERS];
    pthread_cond_t fork_available[NUM_PHILOSOPHERS];

    dp_solution_t solution;

    // Stats
    uint64_t meals_eaten[NUM_PHILOSOPHERS];
    uint64_t total_thinking_time;
    uint64_t total_eating_time;
    uint64_t deadlocks_avoided;
} dining_table_t;
```

### API

```c
// ============== PRODUCER-CONSUMER ==============

// Using mutex + condition variables (2.4.13.f-h)
int pc_init_condvar(pc_buffer_t *buf, size_t capacity);
void pc_destroy(pc_buffer_t *buf);
int pc_produce(pc_buffer_t *buf, int item);          // b: Add item
int pc_consume(pc_buffer_t *buf, int *item);         // c: Remove item
bool pc_is_full(pc_buffer_t *buf);                   // d
bool pc_is_empty(pc_buffer_t *buf);                  // e

// Using semaphores (2.4.13.i)
int pc_init_semaphore(pc_buffer_t *buf, size_t capacity);
int pc_produce_sem(pc_buffer_t *buf, int item);
int pc_consume_sem(pc_buffer_t *buf, int *item);

// Multiple producers/consumers
void pc_run_test(pc_buffer_t *buf, int num_producers, int num_consumers,
                 int items_per_producer);

// ============== READERS-WRITERS ==============

// 2.4.14.f: All variants
int rw_init(rw_lock_t *rw, rw_variant_t variant);
void rw_destroy(rw_lock_t *rw);

// Reader operations
int rw_read_lock(rw_lock_t *rw);                     // a: Shared read
int rw_read_unlock(rw_lock_t *rw);

// Writer operations
int rw_write_lock(rw_lock_t *rw);                    // a: Exclusive write
int rw_write_unlock(rw_lock_t *rw);

// 2.4.14.g: Simple pthread_rwlock alternative
int rw_init_simple(pthread_rwlock_t *rw);

// Starvation detection (2.4.14.e)
bool rw_detect_starvation(rw_lock_t *rw);
void rw_get_stats(rw_lock_t *rw, uint64_t *reads, uint64_t *writes,
                  uint64_t *r_starve, uint64_t *w_starve);

// ============== DINING PHILOSOPHERS ==============

// 2.4.15.h: Initialize with solution
int dp_init(dining_table_t *table, dp_solution_t solution);
void dp_destroy(dining_table_t *table);

// Philosopher actions
void dp_think(dining_table_t *table, int philosopher);
void dp_pickup_forks(dining_table_t *table, int philosopher);
void dp_eat(dining_table_t *table, int philosopher);
void dp_putdown_forks(dining_table_t *table, int philosopher);

// Run simulation
void dp_run_simulation(dining_table_t *table, int duration_sec);

// Individual solution implementations
void dp_pickup_limit(dining_table_t *table, int p);       // c
void dp_pickup_asymmetric(dining_table_t *table, int p);  // d
void dp_pickup_ordering(dining_table_t *table, int p);    // e
void dp_pickup_waiter(dining_table_t *table, int p);      // f
void dp_pickup_chandy(dining_table_t *table, int p);      // g

// Stats and visualization
void dp_print_state(dining_table_t *table);
void dp_get_stats(dining_table_t *table, uint64_t meals[NUM_PHILOSOPHERS]);
bool dp_check_deadlock(dining_table_t *table);            // b
```

---

## Exemple

```c
#include "classic_problems.h"

// ============== PRODUCER-CONSUMER DEMO ==============
void producer_consumer_demo(void) {
    printf("=== Producer-Consumer ===\n");

    pc_buffer_t buffer;

    // 2.4.13.f-h: Condvar solution
    pc_init_condvar(&buffer, 10);

    // Run with 3 producers, 2 consumers
    pc_run_test(&buffer, 3, 2, 100);

    printf("Produced: %lu, Consumed: %lu\n", buffer.produced, buffer.consumed);
    printf("Producer waits: %lu, Consumer waits: %lu\n",
           buffer.producer_waits, buffer.consumer_waits);

    pc_destroy(&buffer);

    // 2.4.13.i: Semaphore solution
    pc_init_semaphore(&buffer, 10);
    pc_run_test(&buffer, 3, 2, 100);
    pc_destroy(&buffer);
}

// ============== READERS-WRITERS DEMO ==============
rw_lock_t rw;
int shared_data = 0;

void *rw_reader(void *arg) {
    int id = *(int*)arg;
    for (int i = 0; i < 100; i++) {
        rw_read_lock(&rw);
        int value = shared_data;  // Read
        rw_read_unlock(&rw);
        (void)value;
        usleep(100);
    }
    return NULL;
}

void *rw_writer(void *arg) {
    int id = *(int*)arg;
    for (int i = 0; i < 20; i++) {
        rw_write_lock(&rw);
        shared_data++;  // Write
        rw_write_unlock(&rw);
        usleep(500);
    }
    return NULL;
}

void readers_writers_demo(void) {
    printf("\n=== Readers-Writers ===\n");

    // 2.4.14.b: Readers preference (writers may starve)
    printf("Testing READERS_PREFERENCE:\n");
    rw_init(&rw, RW_READERS_PREFERENCE);

    pthread_t readers[10], writers[2];
    int rids[10], wids[2];

    for (int i = 0; i < 10; i++) {
        rids[i] = i;
        pthread_create(&readers[i], NULL, rw_reader, &rids[i]);
    }
    for (int i = 0; i < 2; i++) {
        wids[i] = i;
        pthread_create(&writers[i], NULL, rw_writer, &wids[i]);
    }

    for (int i = 0; i < 10; i++) pthread_join(readers[i], NULL);
    for (int i = 0; i < 2; i++) pthread_join(writers[i], NULL);

    uint64_t reads, writes, r_starve, w_starve;
    rw_get_stats(&rw, &reads, &writes, &r_starve, &w_starve);
    printf("Reads: %lu, Writes: %lu, Writer starvation: %lu\n",
           reads, writes, w_starve);

    rw_destroy(&rw);

    // 2.4.14.c: Writers preference
    printf("\nTesting WRITERS_PREFERENCE:\n");
    rw_init(&rw, RW_WRITERS_PREFERENCE);
    shared_data = 0;
    // ... similar test ...
    rw_destroy(&rw);

    // 2.4.14.d: Fair (FIFO)
    printf("\nTesting FAIR:\n");
    rw_init(&rw, RW_FAIR);
    shared_data = 0;
    // ... similar test ...
    rw_destroy(&rw);
}

// ============== DINING PHILOSOPHERS DEMO ==============
void dining_philosophers_demo(void) {
    printf("\n=== Dining Philosophers ===\n");
    dining_table_t table;

    // Test each solution
    dp_solution_t solutions[] = {
        DP_SOLUTION_LIMIT,       // c
        DP_SOLUTION_ASYMMETRIC,  // d
        DP_SOLUTION_ORDERING,    // e
        DP_SOLUTION_WAITER,      // f
        DP_SOLUTION_CHANDY_MISRA // g
    };
    const char *names[] = {
        "Limit (max 4)", "Asymmetric", "Ordering",
        "Waiter", "Chandy-Misra"
    };

    for (int s = 0; s < 5; s++) {
        printf("\nSolution: %s\n", names[s]);
        dp_init(&table, solutions[s]);

        // Run for 2 seconds
        dp_run_simulation(&table, 2);

        // Check for deadlock
        if (dp_check_deadlock(&table)) {
            printf("DEADLOCK DETECTED!\n");
        } else {
            printf("No deadlock - solution works!\n");
        }

        // Stats
        uint64_t meals[NUM_PHILOSOPHERS];
        dp_get_stats(&table, meals);
        printf("Meals eaten: ");
        for (int i = 0; i < NUM_PHILOSOPHERS; i++) {
            printf("P%d=%lu ", i, meals[i]);
        }
        printf("\n");

        dp_destroy(&table);
    }
}

int main(void) {
    producer_consumer_demo();
    readers_writers_demo();
    dining_philosophers_demo();
    return 0;
}
```

---

## Solutions Detail

### 2.4.15.c: Limit Solution
```c
// At most 4 philosophers can sit at once
sem_wait(&room);  // Enter dining room (limited to 4)
pick_up_left_fork();
pick_up_right_fork();
eat();
put_down_forks();
sem_post(&room);  // Leave dining room
```

### 2.4.15.d: Asymmetric Solution
```c
if (philosopher_id == 0) {
    pick_up_right_fork();  // One picks right first
    pick_up_left_fork();
} else {
    pick_up_left_fork();
    pick_up_right_fork();
}
```

### 2.4.15.e: Resource Ordering
```c
int left = philosopher_id;
int right = (philosopher_id + 1) % 5;
int first = min(left, right);
int second = max(left, right);
pthread_mutex_lock(&forks[first]);   // Always lower first
pthread_mutex_lock(&forks[second]);
```

---

## Tests Moulinette

```rust
// Producer-Consumer
#[test] fn test_pc_bounded_buffer()      // 2.4.13.a
#[test] fn test_pc_produce()             // 2.4.13.b
#[test] fn test_pc_consume()             // 2.4.13.c
#[test] fn test_pc_full_wait()           // 2.4.13.d
#[test] fn test_pc_empty_wait()          // 2.4.13.e
#[test] fn test_pc_condvar()             // 2.4.13.f-h
#[test] fn test_pc_semaphore()           // 2.4.13.i

// Readers-Writers
#[test] fn test_rw_concurrent_read()     // 2.4.14.a
#[test] fn test_rw_readers_pref()        // 2.4.14.b
#[test] fn test_rw_writers_pref()        // 2.4.14.c
#[test] fn test_rw_fair()                // 2.4.14.d
#[test] fn test_rw_starvation()          // 2.4.14.e

// Dining Philosophers
#[test] fn test_dp_no_deadlock()         // 2.4.15.b
#[test] fn test_dp_limit()               // 2.4.15.c
#[test] fn test_dp_asymmetric()          // 2.4.15.d
#[test] fn test_dp_ordering()            // 2.4.15.e
#[test] fn test_dp_waiter()              // 2.4.15.f
#[test] fn test_dp_chandy()              // 2.4.15.g
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Producer-Consumer (2.4.13) | 35 |
| Readers-Writers (2.4.14) | 35 |
| Dining Philosophers (2.4.15) | 30 |
| **Total** | **100** |

---

## Fichiers

```
ex04/
├── classic_problems.h
├── producer_consumer.c
├── readers_writers.c
├── dining_philosophers.c
└── Makefile
```
