# ex02: Condition Variables

**Module**: 2.4 - Concurrency & Synchronization
**Difficulte**: Moyen
**Duree**: 4h
**Score qualite**: 96/100

## Concepts Couverts

### 2.4.8: Condition Variables (11 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Concept | Wait for condition |
| b | Associated mutex | Always paired |
| c | pthread_cond_t | Type |
| d | pthread_cond_init() | Initialize |
| e | pthread_cond_wait() | Atomically unlock + wait |
| f | pthread_cond_signal() | Wake one waiter |
| g | pthread_cond_broadcast() | Wake all waiters |
| h | Spurious wakeup | Can happen |
| i | While loop | Always check condition |
| j | pthread_cond_timedwait() | With timeout |
| k | pthread_cond_destroy() | Cleanup |

---

## Sujet

Implementer une bibliotheque de variables de condition avec support complet et demonstrations pratiques.

### Structures

```c
#include <pthread.h>
#include <stdbool.h>
#include <time.h>

// 2.4.8: Condition variable wrapper
typedef struct {
    pthread_cond_t cond;        // c: pthread_cond_t
    pthread_mutex_t *mutex;     // b: Associated mutex
    bool initialized;
    uint64_t wait_count;
    uint64_t signal_count;
    uint64_t broadcast_count;
    uint64_t spurious_wakeups;  // h: Spurious wakeup tracking
} condvar_t;

// Bounded buffer for producer-consumer demo
typedef struct {
    int *buffer;
    size_t capacity;
    size_t count;
    size_t head;
    size_t tail;
    pthread_mutex_t mutex;
    condvar_t not_empty;        // Consumers wait here
    condvar_t not_full;         // Producers wait here
    bool shutdown;
    uint64_t produced;
    uint64_t consumed;
} bounded_buffer_t;

// Event - single notification
typedef struct {
    pthread_mutex_t mutex;
    condvar_t cond;
    bool signaled;
    bool auto_reset;
} event_t;

// Barrier using condition variables
typedef struct {
    pthread_mutex_t mutex;
    condvar_t cond;
    int threshold;
    int count;
    int generation;
} cv_barrier_t;

// Monitor pattern
typedef struct {
    pthread_mutex_t mutex;
    condvar_t *conditions;
    size_t num_conditions;
    void *state;
} monitor_t;
```

### API

```c
// 2.4.8: Condition Variable API
int condvar_init(condvar_t *cv, pthread_mutex_t *mutex);     // d: Initialize
int condvar_destroy(condvar_t *cv);                          // k: Cleanup
int condvar_wait(condvar_t *cv);                             // e: Wait
int condvar_timedwait(condvar_t *cv, unsigned int timeout_ms); // j: Timed wait
int condvar_signal(condvar_t *cv);                           // f: Wake one
int condvar_broadcast(condvar_t *cv);                        // g: Wake all

// 2.4.8.h-i: Spurious wakeup handling
// Correct wait pattern:
// while (!condition) {
//     condvar_wait(&cv);
// }

// Bounded buffer (producer-consumer)
int buffer_init(bounded_buffer_t *buf, size_t capacity);
void buffer_destroy(bounded_buffer_t *buf);
int buffer_put(bounded_buffer_t *buf, int item);             // Blocks if full
int buffer_get(bounded_buffer_t *buf, int *item);            // Blocks if empty
int buffer_try_put(bounded_buffer_t *buf, int item);         // Non-blocking
int buffer_try_get(bounded_buffer_t *buf, int *item);        // Non-blocking
int buffer_put_timeout(bounded_buffer_t *buf, int item, unsigned int ms);
int buffer_get_timeout(bounded_buffer_t *buf, int *item, unsigned int ms);
size_t buffer_size(bounded_buffer_t *buf);
bool buffer_is_empty(bounded_buffer_t *buf);
bool buffer_is_full(bounded_buffer_t *buf);
void buffer_shutdown(bounded_buffer_t *buf);

// Event (single-shot or auto-reset)
int event_init(event_t *ev, bool auto_reset);
void event_destroy(event_t *ev);
int event_wait(event_t *ev);
int event_wait_timeout(event_t *ev, unsigned int timeout_ms);
int event_signal(event_t *ev);
int event_reset(event_t *ev);
bool event_is_signaled(event_t *ev);

// Barrier using condvar
int cv_barrier_init(cv_barrier_t *b, int count);
void cv_barrier_destroy(cv_barrier_t *b);
int cv_barrier_wait(cv_barrier_t *b);

// Monitor pattern
int monitor_init(monitor_t *m, size_t num_conditions);
void monitor_destroy(monitor_t *m);
void monitor_enter(monitor_t *m);
void monitor_exit(monitor_t *m);
void monitor_wait(monitor_t *m, size_t cond_index);
void monitor_signal(monitor_t *m, size_t cond_index);
void monitor_broadcast(monitor_t *m, size_t cond_index);

// Statistics
typedef struct {
    uint64_t total_waits;
    uint64_t total_signals;
    uint64_t total_broadcasts;
    uint64_t spurious_wakeups;
    uint64_t timeouts;
    double avg_wait_time_us;
} condvar_stats_t;

void condvar_get_stats(condvar_t *cv, condvar_stats_t *stats);
```

---

## Exemple

```c
#include "condvar.h"

// Producer-consumer with bounded buffer
bounded_buffer_t buffer;

void *producer(void *arg) {
    int id = *(int*)arg;
    for (int i = 0; i < 10; i++) {
        int item = id * 100 + i;

        // 2.4.8.e: Wait if buffer full
        buffer_put(&buffer, item);

        printf("Producer %d: put %d\n", id, item);
        usleep(rand() % 10000);
    }
    return NULL;
}

void *consumer(void *arg) {
    int id = *(int*)arg;
    while (1) {
        int item;

        // 2.4.8.e: Wait if buffer empty
        int ret = buffer_get_timeout(&buffer, &item, 1000);

        if (ret == ETIMEDOUT) {
            printf("Consumer %d: timeout, checking shutdown\n", id);
            if (buffer.shutdown) break;
            continue;
        }

        printf("Consumer %d: got %d\n", id, item);
        usleep(rand() % 20000);
    }
    return NULL;
}

int main(void) {
    // Initialize bounded buffer
    buffer_init(&buffer, 5);

    pthread_t producers[2], consumers[3];
    int pids[] = {1, 2};
    int cids[] = {1, 2, 3};

    // Start consumers first
    for (int i = 0; i < 3; i++) {
        pthread_create(&consumers[i], NULL, consumer, &cids[i]);
    }

    // Start producers
    for (int i = 0; i < 2; i++) {
        pthread_create(&producers[i], NULL, producer, &pids[i]);
    }

    // Wait for producers
    for (int i = 0; i < 2; i++) {
        pthread_join(producers[i], NULL);
    }

    // Signal shutdown and wake all consumers
    printf("Shutting down...\n");
    buffer_shutdown(&buffer);

    // Wait for consumers
    for (int i = 0; i < 3; i++) {
        pthread_join(consumers[i], NULL);
    }

    printf("Produced: %lu, Consumed: %lu\n", buffer.produced, buffer.consumed);
    buffer_destroy(&buffer);

    // 2.4.8.h-i: Demonstrate correct wait pattern
    printf("\n=== Spurious Wakeup Demo ===\n");
    pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
    condvar_t cv;
    condvar_init(&cv, &mutex);

    // WRONG pattern (don't do this):
    // pthread_mutex_lock(&mutex);
    // if (!condition) condvar_wait(&cv);  // May miss spurious wakeup!
    // pthread_mutex_unlock(&mutex);

    // CORRECT pattern (2.4.8.i):
    // pthread_mutex_lock(&mutex);
    // while (!condition) {
    //     condvar_wait(&cv);
    // }
    // pthread_mutex_unlock(&mutex);

    condvar_stats_t stats;
    condvar_get_stats(&cv, &stats);
    printf("Spurious wakeups detected: %lu\n", stats.spurious_wakeups);

    // Event demo
    printf("\n=== Event Demo ===\n");
    event_t ev;
    event_init(&ev, false);  // Manual reset

    // Thread waits for event
    // Another thread signals event_signal(&ev)
    // Waiter wakes up

    event_destroy(&ev);
    condvar_destroy(&cv);

    // Barrier demo
    printf("\n=== Barrier Demo ===\n");
    cv_barrier_t barrier;
    cv_barrier_init(&barrier, 4);

    // All threads call cv_barrier_wait(&barrier)
    // All proceed only after all arrive

    cv_barrier_destroy(&barrier);

    return 0;
}
```

---

## Implementation Details

### Correct Wait Pattern (2.4.8.h-i)

```c
// 2.4.8.e: pthread_cond_wait atomically:
// 1. Unlocks the mutex
// 2. Blocks waiting for signal
// 3. Re-locks mutex before returning

// 2.4.8.h: Spurious wakeups can occur because:
// - OS implementation details
// - Signal delivery
// - Multi-processor race conditions

// 2.4.8.i: Always use while loop
int buffer_get(bounded_buffer_t *buf, int *item) {
    pthread_mutex_lock(&buf->mutex);

    // WHILE, not IF - handles spurious wakeups
    while (buf->count == 0 && !buf->shutdown) {
        condvar_wait(&buf->not_empty);
    }

    if (buf->shutdown && buf->count == 0) {
        pthread_mutex_unlock(&buf->mutex);
        return -1;
    }

    *item = buf->buffer[buf->head];
    buf->head = (buf->head + 1) % buf->capacity;
    buf->count--;
    buf->consumed++;

    // 2.4.8.f: Wake one producer
    condvar_signal(&buf->not_full);

    pthread_mutex_unlock(&buf->mutex);
    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_condvar_init()           // 2.4.8.d
#[test] fn test_condvar_wait()           // 2.4.8.e
#[test] fn test_condvar_signal()         // 2.4.8.f
#[test] fn test_condvar_broadcast()      // 2.4.8.g
#[test] fn test_spurious_wakeup()        // 2.4.8.h
#[test] fn test_while_loop_pattern()     // 2.4.8.i
#[test] fn test_condvar_timedwait()      // 2.4.8.j
#[test] fn test_bounded_buffer()
#[test] fn test_producer_consumer()
#[test] fn test_event()
#[test] fn test_cv_barrier()
#[test] fn test_monitor_pattern()
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Condvar init/destroy (2.4.8.c-d,k) | 15 |
| Wait/signal/broadcast (2.4.8.e-g) | 30 |
| Spurious wakeup handling (2.4.8.h-i) | 20 |
| Timed wait (2.4.8.j) | 10 |
| Bounded buffer | 15 |
| Event & Barrier | 10 |
| **Total** | **100** |

---

## Fichiers

```
ex02/
├── condvar.h
├── condvar.c
├── bounded_buffer.c
├── event.c
├── cv_barrier.c
└── Makefile
```
