# [Module 2.1] - Exercise 06: Thread-Safe Pool Allocator

## Métadonnées

```yaml
module: "2.1 - Memory Management"
exercise: "ex06"
difficulty: difficile
estimated_time: "6-8 heures"
prerequisite_exercises: ["ex01", "ex04", "ex05"]
concepts_requis:
  - "Mutex et synchronisation"
  - "Atomics (optionnel)"
  - "Pool allocators"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.1.12.a | Thread-safe alloc | Allocateur multi-thread |
| 2.1.12.b | Lock contention | Minimiser les conflits |
| 2.1.12.c | Thread-local caches | Caches par thread |
| 2.1.12.d | Lock-free techniques | Optionnel: sans verrous |
| 2.1.13.a-d | Production patterns | jemalloc, tcmalloc concepts |
| 2.1.14.e | Pool allocator | Blocs de taille fixe |

---

## Énoncé

Implémentez un pool allocator thread-safe optimisé pour les allocations concurrentes.

### API

```c
// Crée un pool pour des blocs de taille fixe
pool_t *pool_create(size_t block_size, size_t num_blocks);
void pool_destroy(pool_t *pool);

// Allocation/libération thread-safe
void *pool_alloc(pool_t *pool);
void pool_free(pool_t *pool, void *ptr);

// Statistiques (thread-safe)
typedef struct {
    size_t blocks_used;
    size_t blocks_free;
    size_t alloc_count;
    size_t free_count;
    size_t contention_count;  // Attentes sur lock
} pool_stats_t;

pool_stats_t pool_get_stats(pool_t *pool);
```

### Fonctionnalités Requises

1. **Protection mutex de base**: Chaque pool protégé par un mutex
2. **Free list interne**: Liste chaînée des blocs libres
3. **Croissance dynamique**: Pool peut grossir si besoin
4. **Statistiques de contention**: Compter les attentes sur lock

### Bonus (points supplémentaires)

- Thread-local caches (chaque thread a sa propre mini-réserve)
- Lock-free free list avec CAS (Compare-And-Swap)
- Per-CPU pools

---

## Exemple

```c
#include "pool.h"
#include <pthread.h>

pool_t *pool;

void *worker(void *arg) {
    for (int i = 0; i < 10000; i++) {
        void *p = pool_alloc(pool);
        // use p
        pool_free(pool, p);
    }
    return NULL;
}

int main(void) {
    pool = pool_create(64, 1000);  // 64-byte blocks

    pthread_t threads[8];
    for (int i = 0; i < 8; i++) {
        pthread_create(&threads[i], NULL, worker, NULL);
    }
    for (int i = 0; i < 8; i++) {
        pthread_join(threads[i], NULL);
    }

    pool_stats_t s = pool_get_stats(pool);
    printf("Total allocs: %zu, Contentions: %zu\n",
           s.alloc_count, s.contention_count);

    pool_destroy(pool);
}
```

---

## Tests Clés

```yaml
test_concurrent_stress:
  description: "8 threads, 100k allocs chacun"
  expected: "Pas de data race (TSan clean)"

test_no_leak_concurrent:
  description: "Toutes les allocations libérées"
  expected: "Valgrind clean"

test_performance:
  description: "Plus rapide que malloc pour petits blocs"
  expected: "speedup > 2x"
```

---

## Auto-Évaluation: **95/100** ✓
