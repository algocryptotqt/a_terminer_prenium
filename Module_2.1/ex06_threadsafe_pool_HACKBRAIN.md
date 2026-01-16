<thinking>
## Analyse du Concept
- Concept : Pool allocator thread-safe avec mutex/atomics
- Phase demandée : 2
- Adapté ? OUI — La concurrence est fondamentale en systèmes modernes

## Combo Base + Bonus
- Exercice de base : Pool avec mutex, free list, statistiques
- Bonus : Thread-local caches, lock-free avec CAS
- Palier bonus : 💀 Expert (lock-free programming)
- Progression logique ? OUI — Base = mutex, Bonus = lock-free

## Prérequis & Difficulté
- Prérequis réels : Mutex, synchronisation, pool allocators
- Difficulté estimée : 6/10 (base), 9/10 (bonus)
- Cohérent avec phase ? OUI — Phase 2 difficile

## Aspect Fun/Culture
- Contexte choisi : Fast & Furious — "Family" = threads, "Garage" = pool
- MEME mnémotechnique : "I live my life a quarter mile at a time" = blocs de taille fixe
- Pourquoi c'est fun : Course = performance, équipe = threads synchronisés

## Scénarios d'Échec (5 mutants)
1. Mutant A (Boundary) : Mutex non déverrouillé sur early return → deadlock
2. Mutant B (Safety) : Race condition sur stats counter → comptage incorrect
3. Mutant C (Resource) : Pool qui grandit sans limite → OOM
4. Mutant D (Logic) : Lock acquis après check → TOCTOU bug
5. Mutant E (Return) : Double unlock → undefined behavior

## Verdict
VALIDE — Exercice avancé couvrant 6 concepts concurrence (2.1.12-14)
</thinking>

---

# Exercice 2.1.6 : furious_pool

**Module :**
2.1.6 — Thread-Safe Memory Allocation

**Concept :**
a-d — Mutex protection, lock contention, thread-local caches, lock-free

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
code

**Tiers :**
3 — Synthèse (concurrence + allocation + performance)

**Langage :**
C17

**Prérequis :**
- Mutex et synchronisation (pthreads)
- Pool allocators (ex04)
- Atomics (optionnel mais recommandé)

**Domaines :**
Mem, Process, Algo

**Durée estimée :**
360-480 min (6-8 heures)

**XP Base :**
500

**Complexité :**
T1 O(1) alloc/free × S2 O(n) pour n blocs

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex06_threadsafe_pool/
├── pool.h
├── pool.c
├── pool_stats.c
├── pool_lockfree.c (bonus)
└── Makefile
```

**Fonctions autorisées :**
- `pthread_mutex_*`, `pthread_cond_*`
- `malloc`, `free`, `mmap`, `munmap`
- `__atomic_*` ou `<stdatomic.h>` (pour bonus)

**Fonctions interdites :**
- `printf` dans le chemin critique

---

### 1.2 Consigne

#### 🎮 Version Culture Pop : "FAST & FURIOUS: Memory Lane"

**"I don't have friends. I got family."** — Dom Toretto

Dans l'univers de Fast & Furious, l'équipe est tout. Chaque membre (thread) a besoin de ressources (blocs mémoire) pour accomplir sa mission. Le garage de Dom (le pool) doit servir tout le monde sans créer de conflits.

**L'équipe :**

| Personnage | Rôle | Concept |
|------------|------|---------|
| 🚗 Dom | Chef du garage | Pool manager |
| 🔧 Brian | Mécanicien rapide | Thread worker |
| 🏎️ Letty | Conduite agressive | High contention |
| 🔒 Hobbs | Sécurité | Mutex protection |
| ⚡ Han | Lock-free style | Atomic operations |

**Les règles du garage :**

```
┌─────────────────────────────────────────────────────────────────┐
│  RÈGLE 1 : Un seul mécano à la fois sur chaque voiture         │
│  RÈGLE 2 : On ne prend pas la voiture d'un autre               │
│  RÈGLE 3 : Toutes les voitures ont la même taille (block_size) │
│  RÈGLE 4 : Le garage peut s'agrandir si nécessaire             │
└─────────────────────────────────────────────────────────────────┘
```

---

#### 📚 Version Académique : Pool Allocator Thread-Safe

**Contexte technique :**

Les allocateurs génériques (malloc) sont optimisés pour la flexibilité, pas pour la performance en environnement multi-thread. Les pool allocators offrent :
- **Allocation O(1)** : Juste prendre le premier bloc libre
- **Fragmentation zéro** : Tous les blocs ont la même taille
- **Cache-friendly** : Blocs contigus en mémoire

Le défi : rendre cela thread-safe sans tuer les performances.

---

**Ta mission :**

Implémenter un pool allocator thread-safe optimisé pour les allocations concurrentes.

**Fonctionnalités requises :**
1. Protection mutex de base
2. Free list interne (liste chaînée)
3. Croissance dynamique si le pool est plein
4. Statistiques de contention

---

### 1.3 Prototypes

```c
#include <stddef.h>
#include <stdint.h>
#include <pthread.h>

/* ═══════════════════════════════════════════════════════════════════════════
 * POOL ALLOCATOR
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct pool pool_t;

/* Crée un pool pour des blocs de taille fixe */
pool_t *pool_create(size_t block_size, size_t initial_blocks);

/* Détruit le pool et libère toute la mémoire */
void pool_destroy(pool_t *pool);

/* Allocation thread-safe - retourne un bloc ou NULL si échec */
void *pool_alloc(pool_t *pool);

/* Libération thread-safe */
void pool_free(pool_t *pool, void *ptr);

/* ═══════════════════════════════════════════════════════════════════════════
 * STATISTIQUES
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    size_t blocks_total;        /* Blocs totaux dans le pool */
    size_t blocks_used;         /* Blocs actuellement alloués */
    size_t blocks_free;         /* Blocs disponibles */
    size_t alloc_count;         /* Nombre total d'allocations */
    size_t free_count;          /* Nombre total de libérations */
    size_t contention_count;    /* Fois où un thread a dû attendre */
    size_t grow_count;          /* Fois où le pool a grandi */
} pool_stats_t;

pool_stats_t pool_get_stats(pool_t *pool);

/* ═══════════════════════════════════════════════════════════════════════════
 * BONUS : LOCK-FREE
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct lockfree_pool lockfree_pool_t;

lockfree_pool_t *lockfree_pool_create(size_t block_size, size_t num_blocks);
void lockfree_pool_destroy(lockfree_pool_t *pool);
void *lockfree_pool_alloc(lockfree_pool_t *pool);
void lockfree_pool_free(lockfree_pool_t *pool, void *ptr);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Lock Contention : Le tueur de performances

```
Scénario : 8 threads, 1 mutex global

Thread 1 : lock... [travail]... unlock
Thread 2 : lock (ATTEND)...
Thread 3 : lock (ATTEND)...
Thread 4 : lock (ATTEND)...
...

Résultat : 7 threads attendent, 1 travaille = 12.5% efficacité !
```

Solutions modernes :
- **Thread-local caches** : Chaque thread a sa mini-réserve
- **Striping** : Plusieurs pools, hash par thread ID
- **Lock-free** : Pas de mutex, juste des atomics

### 2.2 Compare-And-Swap (CAS)

```c
/* Pseudo-code du CAS atomique */
bool CAS(ptr, expected, new_value) {
    atomically {
        if (*ptr == expected) {
            *ptr = new_value;
            return true;
        }
        return false;
    }
}
```

Le CAS est la brique de base de la programmation lock-free.

---

### 2.5 DANS LA VRAIE VIE

| Allocateur | Technique | Utilisé par |
|------------|-----------|-------------|
| **jemalloc** | Arenas per-CPU | Firefox, Redis |
| **tcmalloc** | Thread-local caches | Google services |
| **mimalloc** | Free-list sharding | Microsoft |
| **Hoard** | Superblocks | Academic reference |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
pool.h  pool.c  pool_stats.c  test_concurrent.c  Makefile

$ make

$ ./test_pool
=== Sequential Tests ===
Basic alloc/free: PASS
Pool growth: PASS
Stats tracking: PASS

=== Concurrent Tests (8 threads × 100k ops) ===
Running...
Total time: 0.42s
Total allocs: 800000
Contentions: 1234 (0.15%)
No data races (TSan clean)

All tests passed!
```

---

### 3.1 💀 BONUS EXPERT (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★★☆ (9/10)

**Récompense :**
XP ×4

**Domaines Bonus :**
`CPU (atomics), Algo (lock-free structures)`

#### 3.1.1 Consigne Bonus

**🎮 "Han's Lock-Free Style"**

Han conduit sans regarder en arrière. Le lock-free programming c'est pareil : pas de mutex, juste des opérations atomiques et beaucoup de confiance.

Implémente un pool lock-free utilisant une stack lock-free basée sur CAS.

```c
/* Stack lock-free (Treiber stack) */
typedef struct lf_node {
    struct lf_node *next;
} lf_node_t;

typedef struct {
    _Atomic(lf_node_t *) head;
} lf_stack_t;

/* Push atomique */
void lf_push(lf_stack_t *stack, lf_node_t *node)
{
    lf_node_t *old_head;
    do {
        old_head = atomic_load(&stack->head);
        node->next = old_head;
    } while (!atomic_compare_exchange_weak(&stack->head, &old_head, node));
}

/* Pop atomique */
lf_node_t *lf_pop(lf_stack_t *stack)
{
    lf_node_t *old_head;
    lf_node_t *new_head;
    do {
        old_head = atomic_load(&stack->head);
        if (!old_head)
            return NULL;
        new_head = old_head->next;
    } while (!atomic_compare_exchange_weak(&stack->head, &old_head, new_head));
    return old_head;
}
```

**Attention au ABA problem !**

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette — Tests automatisés

| Test | Description | Entrée | Attendu | Points |
|------|-------------|--------|---------|--------|
| `test_basic` | Alloc/free séquentiel | 1 thread | Fonctionne | 10 |
| `test_concurrent_8` | 8 threads × 10k | Concurrent | No race | 15 |
| `test_concurrent_64` | 64 threads × 1k | High contention | No race | 15 |
| `test_growth` | Pool qui grandit | Dépasse initial | Croissance OK | 10 |
| `test_stats` | Compteurs corrects | — | Stats exactes | 10 |
| `test_tsan_clean` | Thread Sanitizer | — | 0 warnings | 15 |
| `test_no_leak` | Valgrind clean | — | 0 leaks | 10 |
| `test_performance` | Speedup vs malloc | 8 threads | > 2× faster | 15 |

---

### 4.3 Solution de référence — pool.c

```c
#include "pool.h"
#include <stdlib.h>
#include <string.h>
#include <stdatomic.h>

/* ═══════════════════════════════════════════════════════════════════════════
 * STRUCTURES
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct block_node {
    struct block_node *next;
} block_node_t;

typedef struct chunk {
    void *memory;
    size_t size;
    struct chunk *next;
} chunk_t;

struct pool {
    size_t block_size;          /* Taille de chaque bloc */
    size_t blocks_per_chunk;    /* Blocs par chunk */

    block_node_t *free_list;    /* Liste des blocs libres */
    chunk_t *chunks;            /* Liste des chunks alloués */

    pthread_mutex_t lock;       /* Protection globale */

    /* Statistiques atomiques */
    _Atomic size_t blocks_total;
    _Atomic size_t blocks_used;
    _Atomic size_t alloc_count;
    _Atomic size_t free_count;
    _Atomic size_t contention_count;
    _Atomic size_t grow_count;
};

/* ═══════════════════════════════════════════════════════════════════════════
 * HELPERS
 * ═══════════════════════════════════════════════════════════════════════════ */

static size_t align_size(size_t size)
{
    size_t min = sizeof(block_node_t);
    if (size < min)
        size = min;
    return ((size + 15) & ~15);  /* Align 16 */
}

static int pool_grow_locked(pool_t *pool)
{
    size_t chunk_size = pool->block_size * pool->blocks_per_chunk;

    chunk_t *chunk = malloc(sizeof(chunk_t));
    if (!chunk)
        return (-1);

    chunk->memory = malloc(chunk_size);
    if (!chunk->memory)
    {
        free(chunk);
        return (-1);
    }

    chunk->size = chunk_size;
    chunk->next = pool->chunks;
    pool->chunks = chunk;

    /* Ajouter les blocs à la free list */
    char *ptr = chunk->memory;
    for (size_t i = 0; i < pool->blocks_per_chunk; i++)
    {
        block_node_t *node = (block_node_t *)ptr;
        node->next = pool->free_list;
        pool->free_list = node;
        ptr += pool->block_size;
    }

    atomic_fetch_add(&pool->blocks_total, pool->blocks_per_chunk);
    atomic_fetch_add(&pool->grow_count, 1);

    return (0);
}

/* ═══════════════════════════════════════════════════════════════════════════
 * API PUBLIQUE
 * ═══════════════════════════════════════════════════════════════════════════ */

pool_t *pool_create(size_t block_size, size_t initial_blocks)
{
    if (block_size == 0 || initial_blocks == 0)
        return (NULL);

    pool_t *pool = calloc(1, sizeof(pool_t));
    if (!pool)
        return (NULL);

    pool->block_size = align_size(block_size);
    pool->blocks_per_chunk = initial_blocks;

    if (pthread_mutex_init(&pool->lock, NULL) != 0)
    {
        free(pool);
        return (NULL);
    }

    /* Allocation initiale */
    if (pool_grow_locked(pool) != 0)
    {
        pthread_mutex_destroy(&pool->lock);
        free(pool);
        return (NULL);
    }

    return (pool);
}

void pool_destroy(pool_t *pool)
{
    if (!pool)
        return;

    pthread_mutex_lock(&pool->lock);

    chunk_t *chunk = pool->chunks;
    while (chunk)
    {
        chunk_t *next = chunk->next;
        free(chunk->memory);
        free(chunk);
        chunk = next;
    }

    pthread_mutex_unlock(&pool->lock);
    pthread_mutex_destroy(&pool->lock);
    free(pool);
}

void *pool_alloc(pool_t *pool)
{
    if (!pool)
        return (NULL);

    /* Try lock pour détecter contention */
    if (pthread_mutex_trylock(&pool->lock) != 0)
    {
        atomic_fetch_add(&pool->contention_count, 1);
        pthread_mutex_lock(&pool->lock);
    }

    /* Besoin de croissance ? */
    if (!pool->free_list)
    {
        if (pool_grow_locked(pool) != 0)
        {
            pthread_mutex_unlock(&pool->lock);
            return (NULL);
        }
    }

    /* Pop de la free list */
    block_node_t *block = pool->free_list;
    pool->free_list = block->next;

    pthread_mutex_unlock(&pool->lock);

    atomic_fetch_add(&pool->blocks_used, 1);
    atomic_fetch_add(&pool->alloc_count, 1);

    return (block);
}

void pool_free(pool_t *pool, void *ptr)
{
    if (!pool || !ptr)
        return;

    block_node_t *block = (block_node_t *)ptr;

    pthread_mutex_lock(&pool->lock);

    block->next = pool->free_list;
    pool->free_list = block;

    pthread_mutex_unlock(&pool->lock);

    atomic_fetch_sub(&pool->blocks_used, 1);
    atomic_fetch_add(&pool->free_count, 1);
}

pool_stats_t pool_get_stats(pool_t *pool)
{
    pool_stats_t stats = {0};
    if (!pool)
        return (stats);

    stats.blocks_total = atomic_load(&pool->blocks_total);
    stats.blocks_used = atomic_load(&pool->blocks_used);
    stats.blocks_free = stats.blocks_total - stats.blocks_used;
    stats.alloc_count = atomic_load(&pool->alloc_count);
    stats.free_count = atomic_load(&pool->free_count);
    stats.contention_count = atomic_load(&pool->contention_count);
    stats.grow_count = atomic_load(&pool->grow_count);

    return (stats);
}
```

---

### 4.10 Solutions Mutantes

#### Mutant A : Mutex non déverrouillé sur early return

```c
void *pool_alloc_mutant_a(pool_t *pool)
{
    pthread_mutex_lock(&pool->lock);

    if (!pool->free_list)
    {
        if (pool_grow_locked(pool) != 0)
            return NULL;  /* DEADLOCK ! Mutex non déverrouillé */
    }

    /* ... */
    pthread_mutex_unlock(&pool->lock);
}
/* Pourquoi faux : Deadlock après échec de grow */
```

#### Mutant B : Race condition sur stats

```c
void *pool_alloc_mutant_b(pool_t *pool)
{
    pthread_mutex_lock(&pool->lock);
    block_node_t *block = pool->free_list;
    pool->free_list = block->next;
    pthread_mutex_unlock(&pool->lock);

    /* Race condition ! Stats modifiées hors du lock */
    pool->blocks_used++;  /* Non atomique, pas protégé */
}
/* Pourquoi faux : Compteur corrompu sous contention */
```

#### Mutant C : Croissance infinie

```c
static int pool_grow_mutant_c(pool_t *pool)
{
    /* Pas de limite ! Pool peut grandir indéfiniment */
    /* Devrait vérifier : if (pool->blocks_total > MAX_BLOCKS) return -1; */
}
/* Pourquoi faux : OOM si boucle infinie d'allocations */
```

#### Mutant D : TOCTOU bug

```c
void *pool_alloc_mutant_d(pool_t *pool)
{
    /* Check SANS lock */
    if (!pool->free_list)
        pool_grow(pool);  /* Autre thread peut prendre le bloc entre-temps ! */

    pthread_mutex_lock(&pool->lock);
    block_node_t *block = pool->free_list;  /* Peut être NULL maintenant ! */
    pool->free_list = block->next;  /* CRASH */
    pthread_mutex_unlock(&pool->lock);
}
/* Pourquoi faux : Time-of-check to time-of-use vulnerability */
```

#### Mutant E : Double unlock

```c
void pool_free_mutant_e(pool_t *pool, void *ptr)
{
    pthread_mutex_lock(&pool->lock);
    /* ... */
    pthread_mutex_unlock(&pool->lock);
    pthread_mutex_unlock(&pool->lock);  /* DOUBLE UNLOCK ! */
}
/* Pourquoi faux : Undefined behavior, peut corrompre le mutex */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.3 Visualisation ASCII

#### Pool avec Free List

```
POOL STRUCTURE
┌─────────────────────────────────────────────────────────────┐
│  METADATA                                                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ block_size: 64    blocks_total: 8    lock: [MUTEX]   │  │
│  └──────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  FREE LIST HEAD ──┐                                         │
│                   │                                         │
│                   ▼                                         │
│  ┌────┐  ┌────┐  ┌────┐  ┌────┐                           │
│  │USED│  │FREE│──│FREE│──│FREE│── NULL                    │
│  └────┘  └────┘  └────┘  └────┘                           │
│  [0]     [1]     [2]     [3]     [4]     [5]     [6]     [7]│
│  USED    FREE    FREE    FREE    USED    USED    FREE    FREE│
└─────────────────────────────────────────────────────────────┘
```

#### Contention Pattern

```
TIMELINE (8 threads, 1 mutex)

Thread 1: ████████░░░░░░░░░░░░░░░░  (working)
Thread 2: ░░░░░░░░████████░░░░░░░░  (working)
Thread 3: ────────░░░░░░░░████░░░░  (waiting then working)
Thread 4: ────────────────░░░░████  (long wait)
Thread 5: ────────────────────────  (starving!)
...

████ = Holding lock
░░░░ = Working without lock
──── = Waiting for lock

SOLUTION: Thread-local caches réduisent la contention
```

---

### 5.8 Mnémotechniques

#### 🚗 MEME : "I live my life one quarter mile at a time" — Pool Blocks

```
Dom ne pense qu'au quart de mile suivant.
Le pool ne pense qu'au bloc suivant.

Tous les blocs ont la même taille = tous les runs sont égaux.
Pas de fragmentation = pas de surprise.
```

#### 🔒 MEME : "You don't turn your back on family" — Mutex

```
On ne laisse pas tomber un thread en difficulté.
Le mutex garantit que chaque thread aura son tour.

Mais trop de family dinner (contention) = chaos !
Solution : Thread-local caches = chacun mange chez soi.
```

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 2.1.6 — furious_pool |
| **Difficulté** | ★★★★★★☆☆☆☆ (6/10) |
| **Bonus** | 💀 Expert (9/10) |
| **XP Base** | 500 |
| **XP Bonus** | ×4 = 2000 |
| **Durée** | 6-8 heures |
| **Concepts** | 6 concepts concurrence |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1.6-furious_pool",
    "generated_at": "2026-01-11",

    "metadata": {
      "exercise_id": "2.1.6",
      "exercise_name": "furious_pool",
      "module": "2.1",
      "module_name": "Memory Management",
      "concept": "Thread-Safe Allocation",
      "type": "code",
      "tier": 3,
      "phase": 2,
      "difficulty": 6,
      "language": "c17",
      "duration_minutes": 420,
      "xp_base": 500,
      "xp_bonus_multiplier": 4,
      "bonus_tier": "EXPERT",
      "bonus_icon": "💀",
      "domains": ["Mem", "Process", "Algo"],
      "tags": ["thread-safe", "pool", "mutex", "lock-free"],
      "meme_reference": "Fast & Furious"
    }
  }
}
```

---

*Exercice généré avec HACKBRAIN v5.5.2*
*"L'excellence pédagogique ne se négocie pas"*
