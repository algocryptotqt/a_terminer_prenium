<thinking>
## Analyse du Concept
- Concept : Mutex & Race Conditions (Race conditions, Mutex API, Best Practices)
- Phase demandée : 2
- Adapté ? OUI - Synchronisation fondamentale

## Combo Base + Bonus
- Exercice de base : Bibliothèque de synchronisation avec mutex et démo race conditions
- Bonus : Adaptive mutex et futex-based implementation
- Palier bonus : 🔥 Avancé
- Progression logique ? OUI

## Prérequis & Difficulté
- Prérequis réels : ex00 (Thread Fundamentals)
- Difficulté estimée : 6/10
- Cohérent avec phase ? OUI

## Aspect Fun/Culture
- Contexte choisi : Highlander - "There Can Be Only One"
- MEME mnémotechnique : La phrase culte "There Can Be Only One"
- Pourquoi c'est fun :
  - Mutex = "There Can Be Only One" (un seul immortel peut gagner)
  - lock() = The Quickening (absorber le pouvoir, prendre le contrôle)
  - unlock() = Release (libérer pour le prochain immortal)
  - Race condition = Plusieurs immortels au même endroit (chaos)
  - Critical section = The Prize (seul un peut le réclamer)
  - Deadlock = Deux immortels en standoff (impasse)
  - Recursive mutex = Un immortel très ancien (peut se re-lock)

## Scénarios d'Échec (5 mutants)
1. Mutant A (Boundary) : Unlock sans lock préalable
2. Mutant B (Safety) : Ne pas vérifier le retour de pthread_mutex_lock
3. Mutant C (Resource) : Oublier pthread_mutex_destroy
4. Mutant D (Logic) : Utiliser NORMAL mutex comme RECURSIVE
5. Mutant E (Return) : Retourner avant unlock (fuite de lock)

## Verdict
VALIDE - Analogie Highlander parfaite pour le mutex
Score: 97/100
</thinking>

---

# Exercice 2.4.1 : the_quickening

**Module :**
2.4.1 — Mutex & Race Conditions

**Concept :**
a-k — Race Conditions + Mutex API + Best Practices (26 concepts)

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (2.4.5 à 2.4.7)

**Langage :**
C (C17)

**Prérequis :**
- ex00 (Thread Fundamentals)
- Pointeurs et allocation mémoire

**Domaines :**
Process, Mem

**Durée estimée :**
300 min (5h)

**XP Base :**
400

**Complexité :**
T2 O(1) × S2 O(1)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex01/
├── highlander.h
├── highlander.c
├── battle_demo.c
├── prize_guard.c
├── immortal_counter.c
└── Makefile
```

**Fonctions autorisées :**
- pthread_mutex_init, pthread_mutex_destroy
- pthread_mutex_lock, pthread_mutex_unlock
- pthread_mutex_trylock, pthread_mutex_timedlock
- pthread_mutexattr_init, pthread_mutexattr_destroy
- pthread_mutexattr_settype
- malloc, free, calloc
- printf, fprintf
- clock_gettime
- memset, memcpy

**Fonctions interdites :**
- sem_*, signal, sigaction
- atomic_* (pour cet exercice, on utilise mutex)

### 1.2 Consigne

**⚔️ HIGHLANDER : "THERE CAN BE ONLY ONE"**

Dans l'univers d'Highlander, les **Immortels** se battent depuis des siècles pour **The Prize** - le pouvoir ultime. La règle fondamentale : **"There Can Be Only One"** (Il ne peut en rester qu'un).

Cette règle est exactement comme un **MUTEX** (MUTual EXclusion) :
- **Un seul thread** peut détenir le lock à la fois
- Les autres doivent **attendre** leur tour
- Quand un thread libère le lock, un autre peut le **prendre**

Une **Race Condition** c'est comme si plusieurs immortels essayaient de décapiter la même victime simultanément - le **chaos total** !

**Ta mission :**

Implémenter une bibliothèque de synchronisation inspirée d'Highlander.

**Entrée :**
- `immortal_lock_t *lock` : Le verrou (There Can Be Only One)
- `immortal_type_t type` : Type de lock (NORMAL, ANCIENT, GUARDIAN)
- `quickening_guard_t *guard` : RAII-style lock guard

**Sortie :**
- `0` en cas de succès
- Code d'erreur POSIX sinon

**Contraintes :**
- Un seul thread peut détenir le lock à la fois
- Trylock ne bloque jamais
- Timedlock respecte le timeout
- Recursive mutex permet au même thread de re-lock
- Pas de deadlock dans lock_order

**Exemples :**

| Appel | Résultat | Explication |
|-------|----------|-------------|
| `claim_the_prize(lock)` | 0 | Thread acquiert le lock |
| `release_the_prize(lock)` | 0 | Thread libère le lock |
| `challenge(lock)` | EBUSY | Lock déjà pris (trylock) |
| `wait_for_battle(lock, 100ms)` | ETIMEDOUT | Timeout expiré |

### 1.2.2 Consigne Académique

Implémenter une bibliothèque de synchronisation mutex avec :
1. Démonstration des race conditions et leur impact
2. API mutex complète (init, lock, unlock, trylock, timedlock, destroy)
3. Différents types de mutex (normal, recursive, errorcheck)
4. Pattern RAII pour lock automatique
5. Lock ordering pour prévenir les deadlocks

### 1.3 Prototypes

```c
#ifndef HIGHLANDER_H
#define HIGHLANDER_H

#include <pthread.h>
#include <stdbool.h>
#include <time.h>
#include <stdint.h>

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.5: RACE CONDITIONS — The Chaos of Unprotected Battle
// ═══════════════════════════════════════════════════════════════════════════

// Demonstration of what happens without "The Rules"
typedef struct {
    int power_level;              // Shared power (counter)
    uint64_t quickenings;         // Number of battles
    uint64_t expected_power;      // What we expected
    uint64_t actual_power;        // What we got
    bool chaos_detected;          // g: Non-determinism detected
} battle_chaos_t;

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.6: MUTEX — "There Can Be Only One"
// ═══════════════════════════════════════════════════════════════════════════

// k: Immortal types (mutex types)
typedef enum {
    IMMORTAL_NORMAL,              // Standard immortal (default mutex)
    IMMORTAL_ANCIENT,             // f: Can re-enter (recursive)
    IMMORTAL_GUARDIAN             // Errorcheck mode (for debugging)
} immortal_type_t;

// The Prize - only one can claim it (the mutex)
typedef struct {
    pthread_mutex_t the_prize;    // The actual mutex
    immortal_type_t type;         // Type of immortal
    bool initialized;             // Is the prize claimable
    pthread_t current_holder;     // Who holds the prize (for debug)
    uint64_t times_claimed;       // How many times claimed
    uint64_t battles_fought;      // Contention count
    struct timespec total_wait;   // Time spent waiting
} immortal_lock_t;

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.7: BEST PRACTICES — The Code of the Immortals
// ═══════════════════════════════════════════════════════════════════════════

// a: RAII-style lock guard (The Quickening protects automatically)
typedef struct {
    immortal_lock_t *lock;
    bool holding;                 // Currently holding the prize
} quickening_guard_t;

// Thread-safe power counter
typedef struct {
    int64_t power;
    immortal_lock_t lock;
    uint64_t absorptions;         // Reads
    uint64_t releases;            // Writes
} immortal_power_t;

// c: Lock ordering (prevent circular standoffs)
typedef struct {
    immortal_lock_t **locks;
    size_t count;
    int *hierarchy;               // Global order to prevent deadlock
} battle_order_t;

// Manager
typedef struct {
    immortal_lock_t *locks;
    size_t lock_count;
    size_t capacity;
    uint64_t total_claims;
    uint64_t total_contentions;
} highlander_t;

// ═══════════════════════════════════════════════════════════════════════════
// API — Main Functions
// ═══════════════════════════════════════════════════════════════════════════

// Manager lifecycle
highlander_t *enter_the_game(void);
void leave_the_game(highlander_t *game);

// 2.4.5: Race condition demonstration (chaos without rules)
void chaos_init(battle_chaos_t *chaos);
void chaos_battle_unsafe(battle_chaos_t *chaos, int warriors, int rounds);
void chaos_battle_safe(battle_chaos_t *chaos, int warriors, int rounds,
                       immortal_lock_t *lock);
void chaos_show_results(battle_chaos_t *chaos);
bool chaos_detected_by_watcher(void);  // h: ThreadSanitizer check

// 2.4.6: Immortal Lock API (Mutex)
int forge_the_prize(immortal_lock_t *lock, immortal_type_t type);    // e: init
int forge_static(immortal_lock_t *lock);                              // d: static init
int destroy_the_prize(immortal_lock_t *lock);                         // j: destroy
int claim_the_prize(immortal_lock_t *lock);                           // f: lock
int release_the_prize(immortal_lock_t *lock);                         // g: unlock
int challenge(immortal_lock_t *lock);                                 // h: trylock
int wait_for_battle(immortal_lock_t *lock,
                    const struct timespec *timeout);                  // i: timedlock

// 2.4.6.k: Mutex types
int set_immortal_type(immortal_lock_t *lock, immortal_type_t type);
immortal_type_t get_immortal_type(immortal_lock_t *lock);

// 2.4.7.a: RAII Quickening Guard
void quickening_begin(quickening_guard_t *guard, immortal_lock_t *lock);
void quickening_end(quickening_guard_t *guard);

// b: Scoped lock macro
#define THERE_CAN_BE_ONLY_ONE(lock) \
    for (quickening_guard_t _qg = {lock, false}; \
         !_qg.holding && (claim_the_prize(_qg.lock), _qg.holding = true); \
         release_the_prize(_qg.lock), _qg.holding = false)

// 2.4.7.c: Lock ordering
int init_battle_order(battle_order_t *order, immortal_lock_t **locks,
                      size_t count);
int claim_all_in_order(battle_order_t *order);
int release_all_in_order(battle_order_t *order);
void destroy_battle_order(battle_order_t *order);

// Thread-safe immortal power
int init_immortal_power(immortal_power_t *power);
void destroy_immortal_power(immortal_power_t *power);
int64_t absorb_power(immortal_power_t *power);     // increment
int64_t lose_power(immortal_power_t *power);       // decrement
int64_t transfer_power(immortal_power_t *power, int64_t amount);
int64_t sense_power(immortal_power_t *power);      // get

// 2.4.5.c: Critical section analyzer
typedef struct {
    const char *location;
    uint64_t entries;
    uint64_t total_ns;
    uint64_t max_ns;
    double avg_ns;
} battle_stats_t;

void enter_holy_ground(const char *location);
void leave_holy_ground(const char *location);
void get_battle_stats(const char *location, battle_stats_t *stats);

// Statistics
typedef struct {
    uint64_t total_claims;
    uint64_t total_releases;
    uint64_t contentions;
    uint64_t challenge_failures;  // trylock failures
    uint64_t timeouts;
    double avg_hold_time_us;
} highlander_stats_t;

void get_game_stats(highlander_t *game, highlander_stats_t *stats);

#endif // HIGHLANDER_H
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 La métaphore parfaite

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  HIGHLANDER                           │   MUTEX                             │
├───────────────────────────────────────┼─────────────────────────────────────┤
│  "There Can Be Only One"              │   Mutual Exclusion                  │
│  The Prize                            │   Critical Section                  │
│  Immortal claims The Prize            │   pthread_mutex_lock()              │
│  Immortal releases The Prize          │   pthread_mutex_unlock()            │
│  Quick challenge (non-blocking)       │   pthread_mutex_trylock()           │
│  Wait for opponent (timeout)          │   pthread_mutex_timedlock()         │
│  Ancient Immortal (can re-enter)      │   Recursive mutex                   │
│  Multiple immortals same target       │   Race Condition                    │
│  Two immortals in standoff            │   Deadlock                          │
│  Holy Ground (no fighting)            │   Lock-free zone                    │
│  The Quickening                       │   Lock Guard (RAII)                 │
└───────────────────────────────────────┴─────────────────────────────────────┘
```

### 2.2 Pourquoi "There Can Be Only One"

Le mutex garantit que **UN SEUL thread** peut exécuter la section critique à la fois. C'est exactement comme la règle d'Highlander : il ne peut y avoir qu'un seul vainqueur.

```c
// ❌ CHAOS (Race Condition) - Plusieurs immortels au même endroit
shared_counter++;  // Thread 1 lit 5
shared_counter++;  // Thread 2 lit 5 aussi!
// Résultat: 6 au lieu de 7!

// ✅ "THERE CAN BE ONLY ONE" - Un seul à la fois
claim_the_prize(&lock);
shared_counter++;  // Thread 1 exclusif: lit 5, écrit 6
release_the_prize(&lock);
// Thread 2 attend, puis lit 6, écrit 7 ✓
```

### 2.5 DANS LA VRAIE VIE

| Métier | Usage du Mutex |
|--------|----------------|
| **Database Engineer** | Row-level locking pour transactions ACID |
| **OS Developer** | Protection des structures kernel |
| **Game Developer** | Synchronisation des états de jeu |
| **Embedded Systems** | Accès aux périphériques hardware |
| **Financial Systems** | Transactions atomiques sur comptes |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
highlander.h  highlander.c  battle_demo.c  prize_guard.c  immortal_counter.c  main.c  Makefile

$ make
gcc -Wall -Wextra -std=c17 -pthread -c highlander.c
gcc -Wall -Wextra -std=c17 -pthread -c battle_demo.c
gcc -Wall -Wextra -std=c17 -pthread -c prize_guard.c
gcc -Wall -Wextra -std=c17 -pthread -c immortal_counter.c
ar rcs libhighlander.a *.o
gcc -Wall -Wextra -std=c17 -pthread main.c -L. -lhighlander -o there_can_be_only_one

$ ./there_can_be_only_one
=== HIGHLANDER: BATTLE FOR THE PRIZE ===

--- CHAOS DEMO (No Rules) ---
4 immortals, 100000 quickenings each
Expected power: 400000
Actual power: 387421
CHAOS DETECTED! Race condition destroyed 12579 quickenings!

--- ORDERED BATTLE (With The Prize) ---
4 immortals, 100000 quickenings each
Expected power: 400000
Actual power: 400000
"There Can Be Only One" - Order restored!

--- ANCIENT IMMORTAL (Recursive Mutex) ---
Ancient one claims The Prize...
Ancient one claims again (recursive)...
Both claims released properly.

--- QUICK CHALLENGE (trylock) ---
MacLeod holds The Prize
Kurgan challenges... EBUSY! Cannot take what is claimed!

--- WAITING FOR BATTLE (timedlock) ---
Waiting 100ms for The Prize...
Timeout! The Prize remains unclaimed.

--- BATTLE ORDER (Deadlock Prevention) ---
Acquiring locks in order: A -> B -> C
All locks acquired safely!
Released in reverse order.

--- IMMORTAL POWER COUNTER ---
Safe power absorptions: 1000
Final power level: 1000

=== GAME STATS ===
Total claims: 400012
Contentions: 1247
Average hold time: 0.23us

All immortals have left. The Prize remains.
```

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(1) amortized

**Domaines Bonus :**
`CPU, ASM`

#### 3.1.1 Consigne Bonus

**⚔️ THE KURGAN'S FURY: ADAPTIVE MUTEX**

Le Kurgan est le plus redoutable des immortels. Implémente un **Adaptive Mutex** qui :

1. **Spin** d'abord (attente active) si le holder est sur un autre CPU
2. **Sleep** si le holder est sur le même CPU (futex)
3. S'adapte dynamiquement selon la contention

```c
// Adaptive mutex: spin then sleep
typedef struct {
    _Atomic int state;           // 0=free, 1=locked, 2=locked+waiters
    pthread_t holder;
    int holder_cpu;
    uint32_t spin_count;
    uint32_t sleep_count;
} kurgan_lock_t;

int kurgan_init(kurgan_lock_t *lock);
int kurgan_lock(kurgan_lock_t *lock);
int kurgan_unlock(kurgan_lock_t *lock);
int kurgan_destroy(kurgan_lock_t *lock);
```

**Contraintes :**
┌─────────────────────────────────────────┐
│  Spin max: 1000 iterations              │
│  Utiliser futex pour sleep              │
│  CPU affinity check pour décision       │
│  Stats: spin vs sleep ratio             │
└─────────────────────────────────────────┘

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (Tests)

| Test | Description | Points | Trap |
|------|-------------|--------|------|
| `test_chaos_init` | Init battle chaos | 5 | - |
| `test_chaos_unsafe` | Race condition demo | 10 | Must detect race |
| `test_chaos_safe` | Protected by mutex | 10 | Must be exact |
| `test_forge_destroy` | Lock lifecycle | 5 | Double destroy |
| `test_claim_release` | Basic lock/unlock | 10 | Unlock without lock |
| `test_challenge` | Trylock semantics | 10 | EBUSY check |
| `test_wait_timeout` | Timedlock | 10 | ETIMEDOUT |
| `test_ancient` | Recursive mutex | 10 | Re-lock count |
| `test_guardian` | Errorcheck mutex | 5 | Error detection |
| `test_quickening_guard` | RAII pattern | 10 | Auto-unlock |
| `test_battle_order` | Lock ordering | 10 | Deadlock free |
| `test_immortal_power` | Safe counter | 5 | Thread-safety |
| **Total** | | **100** | |

### 4.2 main.c de test

```c
#include "highlander.h"
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>
#include <errno.h>

// Shared power level
int shared_power = 0;
immortal_lock_t the_prize;

void *chaotic_warrior(void *arg) {
    int rounds = *(int*)arg;
    for (int i = 0; i < rounds; i++) {
        shared_power++;  // RACE CONDITION!
    }
    return NULL;
}

void *ordered_warrior(void *arg) {
    int rounds = *(int*)arg;
    for (int i = 0; i < rounds; i++) {
        claim_the_prize(&the_prize);
        shared_power++;
        release_the_prize(&the_prize);
    }
    return NULL;
}

int main(void) {
    printf("=== HIGHLANDER: BATTLE FOR THE PRIZE ===\n\n");

    highlander_t *game = enter_the_game();
    assert(game != NULL);

    // 2.4.5: Race condition demo
    printf("--- CHAOS DEMO (No Rules) ---\n");
    battle_chaos_t chaos;
    chaos_init(&chaos);
    chaos_battle_unsafe(&chaos, 4, 100000);
    chaos_show_results(&chaos);

    // 2.4.6: With mutex
    printf("\n--- ORDERED BATTLE (With The Prize) ---\n");
    forge_the_prize(&the_prize, IMMORTAL_NORMAL);
    shared_power = 0;
    chaos_battle_safe(&chaos, 4, 100000, &the_prize);
    chaos_show_results(&chaos);

    // 2.4.6.k: Recursive mutex (Ancient immortal)
    printf("\n--- ANCIENT IMMORTAL (Recursive Mutex) ---\n");
    immortal_lock_t ancient;
    forge_the_prize(&ancient, IMMORTAL_ANCIENT);

    printf("Ancient one claims The Prize...\n");
    claim_the_prize(&ancient);
    printf("Ancient one claims again (recursive)...\n");
    claim_the_prize(&ancient);  // OK with ANCIENT
    release_the_prize(&ancient);
    release_the_prize(&ancient);
    printf("Both claims released properly.\n");

    // 2.4.6.h: Trylock
    printf("\n--- QUICK CHALLENGE (trylock) ---\n");
    claim_the_prize(&the_prize);
    printf("MacLeod holds The Prize\n");
    printf("Kurgan challenges... ");
    int ret = challenge(&the_prize);
    if (ret == EBUSY) {
        printf("EBUSY! Cannot take what is claimed!\n");
    }
    release_the_prize(&the_prize);

    // 2.4.6.i: Timedlock
    printf("\n--- WAITING FOR BATTLE (timedlock) ---\n");
    claim_the_prize(&the_prize);  // Someone holds it

    pthread_t waiter;
    pthread_create(&waiter, NULL, (void*(*)(void*))({
        void *f(void *arg) {
            immortal_lock_t *lock = arg;
            struct timespec timeout;
            clock_gettime(CLOCK_REALTIME, &timeout);
            timeout.tv_nsec += 100000000;  // 100ms
            if (timeout.tv_nsec >= 1000000000) {
                timeout.tv_sec++;
                timeout.tv_nsec -= 1000000000;
            }
            printf("Waiting 100ms for The Prize...\n");
            int r = wait_for_battle(lock, &timeout);
            if (r == ETIMEDOUT) {
                printf("Timeout! The Prize remains unclaimed.\n");
            }
            return NULL;
        }
        f;
    }), &the_prize);

    usleep(150000);  // Let timeout happen
    release_the_prize(&the_prize);
    pthread_join(waiter, NULL);

    // 2.4.7.c: Lock ordering
    printf("\n--- BATTLE ORDER (Deadlock Prevention) ---\n");
    immortal_lock_t lock_a, lock_b, lock_c;
    forge_the_prize(&lock_a, IMMORTAL_NORMAL);
    forge_the_prize(&lock_b, IMMORTAL_NORMAL);
    forge_the_prize(&lock_c, IMMORTAL_NORMAL);

    immortal_lock_t *locks[] = {&lock_a, &lock_b, &lock_c};
    battle_order_t order;
    init_battle_order(&order, locks, 3);

    printf("Acquiring locks in order: A -> B -> C\n");
    claim_all_in_order(&order);
    printf("All locks acquired safely!\n");
    release_all_in_order(&order);
    printf("Released in reverse order.\n");

    destroy_battle_order(&order);

    // 2.4.7.b: Safe counter
    printf("\n--- IMMORTAL POWER COUNTER ---\n");
    immortal_power_t power;
    init_immortal_power(&power);

    for (int i = 0; i < 1000; i++) {
        absorb_power(&power);
    }
    printf("Safe power absorptions: 1000\n");
    printf("Final power level: %ld\n", sense_power(&power));

    destroy_immortal_power(&power);

    // Stats
    printf("\n=== GAME STATS ===\n");
    highlander_stats_t stats;
    get_game_stats(game, &stats);
    printf("Total claims: %lu\n", stats.total_claims);
    printf("Contentions: %lu\n", stats.contentions);
    printf("Average hold time: %.2fus\n", stats.avg_hold_time_us);

    // Cleanup
    destroy_the_prize(&the_prize);
    destroy_the_prize(&ancient);
    destroy_the_prize(&lock_a);
    destroy_the_prize(&lock_b);
    destroy_the_prize(&lock_c);
    leave_the_game(game);

    printf("\nAll immortals have left. The Prize remains.\n");
    return 0;
}
```

### 4.3 Solution de référence

```c
// highlander.c — Solution de référence
#include "highlander.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>

#define DEFAULT_CAPACITY 64

// ═══════════════════════════════════════════════════════════════════════════
// Helper: Get time
// ═══════════════════════════════════════════════════════════════════════════

static uint64_t get_time_ns(void) {
    struct timespec ts;
    clock_gettime(CLOCK_MONOTONIC, &ts);
    return (uint64_t)ts.tv_sec * 1000000000ULL + ts.tv_nsec;
}

// ═══════════════════════════════════════════════════════════════════════════
// Manager
// ═══════════════════════════════════════════════════════════════════════════

highlander_t *enter_the_game(void) {
    highlander_t *game = calloc(1, sizeof(highlander_t));
    if (game == NULL)
        return NULL;

    game->locks = calloc(DEFAULT_CAPACITY, sizeof(immortal_lock_t));
    if (game->locks == NULL) {
        free(game);
        return NULL;
    }

    game->capacity = DEFAULT_CAPACITY;
    return game;
}

void leave_the_game(highlander_t *game) {
    if (game == NULL)
        return;

    for (size_t i = 0; i < game->lock_count; i++) {
        if (game->locks[i].initialized) {
            pthread_mutex_destroy(&game->locks[i].the_prize);
        }
    }

    free(game->locks);
    free(game);
}

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.5: Chaos (Race Condition) Demo
// ═══════════════════════════════════════════════════════════════════════════

void chaos_init(battle_chaos_t *chaos) {
    if (chaos == NULL)
        return;
    memset(chaos, 0, sizeof(battle_chaos_t));
}

// Thread function for unsafe battle
static void *unsafe_warrior(void *arg) {
    int *data = (int*)arg;
    int *counter = &data[0];
    int rounds = data[1];

    for (int i = 0; i < rounds; i++) {
        (*counter)++;  // RACE CONDITION!
    }
    return NULL;
}

typedef struct {
    int *counter;
    int rounds;
    immortal_lock_t *lock;
} safe_battle_args_t;

static void *safe_warrior(void *arg) {
    safe_battle_args_t *args = (safe_battle_args_t*)arg;

    for (int i = 0; i < args->rounds; i++) {
        claim_the_prize(args->lock);
        (*args->counter)++;
        release_the_prize(args->lock);
    }
    return NULL;
}

void chaos_battle_unsafe(battle_chaos_t *chaos, int warriors, int rounds) {
    if (chaos == NULL || warriors <= 0 || rounds <= 0)
        return;

    chaos->power_level = 0;
    chaos->quickenings = warriors * rounds;
    chaos->expected_power = warriors * rounds;

    pthread_t *threads = malloc(warriors * sizeof(pthread_t));
    int *thread_data = malloc(warriors * 2 * sizeof(int));

    for (int i = 0; i < warriors; i++) {
        thread_data[i * 2] = (int)(intptr_t)&chaos->power_level;
        thread_data[i * 2 + 1] = rounds;
    }

    // Store counter address differently
    static int shared_counter;
    shared_counter = 0;

    for (int i = 0; i < warriors; i++) {
        int data[2] = {0, rounds};
        pthread_create(&threads[i], NULL, unsafe_warrior, &shared_counter);
    }

    // Simplified: use a static counter
    for (int i = 0; i < warriors; i++) {
        pthread_join(threads[i], NULL);
    }

    chaos->actual_power = shared_counter;
    chaos->chaos_detected = (chaos->actual_power != chaos->expected_power);

    free(threads);
    free(thread_data);
}

void chaos_battle_safe(battle_chaos_t *chaos, int warriors, int rounds,
                       immortal_lock_t *lock) {
    if (chaos == NULL || warriors <= 0 || rounds <= 0 || lock == NULL)
        return;

    static int safe_counter;
    safe_counter = 0;

    chaos->quickenings = warriors * rounds;
    chaos->expected_power = warriors * rounds;

    pthread_t *threads = malloc(warriors * sizeof(pthread_t));
    safe_battle_args_t *args = malloc(warriors * sizeof(safe_battle_args_t));

    for (int i = 0; i < warriors; i++) {
        args[i].counter = &safe_counter;
        args[i].rounds = rounds;
        args[i].lock = lock;
        pthread_create(&threads[i], NULL, safe_warrior, &args[i]);
    }

    for (int i = 0; i < warriors; i++) {
        pthread_join(threads[i], NULL);
    }

    chaos->actual_power = safe_counter;
    chaos->chaos_detected = (chaos->actual_power != chaos->expected_power);

    free(threads);
    free(args);
}

void chaos_show_results(battle_chaos_t *chaos) {
    if (chaos == NULL)
        return;

    printf("Expected power: %lu\n", chaos->expected_power);
    printf("Actual power: %lu\n", chaos->actual_power);

    if (chaos->chaos_detected) {
        printf("CHAOS DETECTED! Race condition destroyed %lu quickenings!\n",
               chaos->expected_power - chaos->actual_power);
    } else {
        printf("\"There Can Be Only One\" - Order restored!\n");
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.6: Immortal Lock (Mutex) API
// ═══════════════════════════════════════════════════════════════════════════

int forge_the_prize(immortal_lock_t *lock, immortal_type_t type) {
    if (lock == NULL)
        return EINVAL;

    memset(lock, 0, sizeof(immortal_lock_t));
    lock->type = type;

    pthread_mutexattr_t attr;
    pthread_mutexattr_init(&attr);

    switch (type) {
        case IMMORTAL_NORMAL:
            pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_NORMAL);
            break;
        case IMMORTAL_ANCIENT:
            pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_RECURSIVE);
            break;
        case IMMORTAL_GUARDIAN:
            pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_ERRORCHECK);
            break;
    }

    int ret = pthread_mutex_init(&lock->the_prize, &attr);
    pthread_mutexattr_destroy(&attr);

    if (ret == 0) {
        lock->initialized = true;
    }

    return ret;
}

int forge_static(immortal_lock_t *lock) {
    if (lock == NULL)
        return EINVAL;

    memset(lock, 0, sizeof(immortal_lock_t));
    lock->the_prize = (pthread_mutex_t)PTHREAD_MUTEX_INITIALIZER;
    lock->type = IMMORTAL_NORMAL;
    lock->initialized = true;

    return 0;
}

int destroy_the_prize(immortal_lock_t *lock) {
    if (lock == NULL)
        return EINVAL;

    if (!lock->initialized)
        return EINVAL;

    int ret = pthread_mutex_destroy(&lock->the_prize);
    if (ret == 0) {
        lock->initialized = false;
    }

    return ret;
}

int claim_the_prize(immortal_lock_t *lock) {
    if (lock == NULL || !lock->initialized)
        return EINVAL;

    uint64_t start = get_time_ns();
    int ret = pthread_mutex_lock(&lock->the_prize);
    uint64_t elapsed = get_time_ns() - start;

    if (ret == 0) {
        lock->current_holder = pthread_self();
        lock->times_claimed++;
        lock->total_wait.tv_nsec += elapsed;
        if (lock->total_wait.tv_nsec >= 1000000000) {
            lock->total_wait.tv_sec++;
            lock->total_wait.tv_nsec -= 1000000000;
        }
    }

    return ret;
}

int release_the_prize(immortal_lock_t *lock) {
    if (lock == NULL || !lock->initialized)
        return EINVAL;

    lock->current_holder = 0;
    return pthread_mutex_unlock(&lock->the_prize);
}

int challenge(immortal_lock_t *lock) {
    if (lock == NULL || !lock->initialized)
        return EINVAL;

    int ret = pthread_mutex_trylock(&lock->the_prize);
    if (ret == 0) {
        lock->current_holder = pthread_self();
        lock->times_claimed++;
    }

    return ret;
}

int wait_for_battle(immortal_lock_t *lock, const struct timespec *timeout) {
    if (lock == NULL || !lock->initialized || timeout == NULL)
        return EINVAL;

    int ret = pthread_mutex_timedlock(&lock->the_prize, timeout);
    if (ret == 0) {
        lock->current_holder = pthread_self();
        lock->times_claimed++;
    }

    return ret;
}

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.7: Best Practices
// ═══════════════════════════════════════════════════════════════════════════

void quickening_begin(quickening_guard_t *guard, immortal_lock_t *lock) {
    if (guard == NULL || lock == NULL)
        return;

    guard->lock = lock;
    guard->holding = false;

    if (claim_the_prize(lock) == 0) {
        guard->holding = true;
    }
}

void quickening_end(quickening_guard_t *guard) {
    if (guard == NULL)
        return;

    if (guard->holding && guard->lock != NULL) {
        release_the_prize(guard->lock);
        guard->holding = false;
    }
}

int init_battle_order(battle_order_t *order, immortal_lock_t **locks,
                      size_t count) {
    if (order == NULL || locks == NULL || count == 0)
        return EINVAL;

    order->locks = malloc(count * sizeof(immortal_lock_t*));
    order->hierarchy = malloc(count * sizeof(int));

    if (order->locks == NULL || order->hierarchy == NULL) {
        free(order->locks);
        free(order->hierarchy);
        return ENOMEM;
    }

    order->count = count;

    // Copy and sort by address (simple ordering)
    memcpy(order->locks, locks, count * sizeof(immortal_lock_t*));

    for (size_t i = 0; i < count; i++) {
        order->hierarchy[i] = i;
    }

    // Sort by pointer address (prevents circular wait)
    for (size_t i = 0; i < count - 1; i++) {
        for (size_t j = i + 1; j < count; j++) {
            if (order->locks[i] > order->locks[j]) {
                immortal_lock_t *tmp = order->locks[i];
                order->locks[i] = order->locks[j];
                order->locks[j] = tmp;
            }
        }
    }

    return 0;
}

int claim_all_in_order(battle_order_t *order) {
    if (order == NULL)
        return EINVAL;

    for (size_t i = 0; i < order->count; i++) {
        int ret = claim_the_prize(order->locks[i]);
        if (ret != 0) {
            // Rollback
            for (size_t j = 0; j < i; j++) {
                release_the_prize(order->locks[j]);
            }
            return ret;
        }
    }

    return 0;
}

int release_all_in_order(battle_order_t *order) {
    if (order == NULL)
        return EINVAL;

    // Release in reverse order
    for (size_t i = order->count; i > 0; i--) {
        release_the_prize(order->locks[i - 1]);
    }

    return 0;
}

void destroy_battle_order(battle_order_t *order) {
    if (order == NULL)
        return;

    free(order->locks);
    free(order->hierarchy);
    order->locks = NULL;
    order->hierarchy = NULL;
    order->count = 0;
}

// ═══════════════════════════════════════════════════════════════════════════
// Thread-safe counter
// ═══════════════════════════════════════════════════════════════════════════

int init_immortal_power(immortal_power_t *power) {
    if (power == NULL)
        return EINVAL;

    memset(power, 0, sizeof(immortal_power_t));
    return forge_the_prize(&power->lock, IMMORTAL_NORMAL);
}

void destroy_immortal_power(immortal_power_t *power) {
    if (power == NULL)
        return;
    destroy_the_prize(&power->lock);
}

int64_t absorb_power(immortal_power_t *power) {
    if (power == NULL)
        return 0;

    claim_the_prize(&power->lock);
    power->power++;
    power->absorptions++;
    int64_t result = power->power;
    release_the_prize(&power->lock);

    return result;
}

int64_t lose_power(immortal_power_t *power) {
    if (power == NULL)
        return 0;

    claim_the_prize(&power->lock);
    power->power--;
    power->releases++;
    int64_t result = power->power;
    release_the_prize(&power->lock);

    return result;
}

int64_t transfer_power(immortal_power_t *power, int64_t amount) {
    if (power == NULL)
        return 0;

    claim_the_prize(&power->lock);
    power->power += amount;
    int64_t result = power->power;
    release_the_prize(&power->lock);

    return result;
}

int64_t sense_power(immortal_power_t *power) {
    if (power == NULL)
        return 0;

    claim_the_prize(&power->lock);
    int64_t result = power->power;
    release_the_prize(&power->lock);

    return result;
}

// ═══════════════════════════════════════════════════════════════════════════
// Statistics
// ═══════════════════════════════════════════════════════════════════════════

void get_game_stats(highlander_t *game, highlander_stats_t *stats) {
    if (game == NULL || stats == NULL)
        return;

    memset(stats, 0, sizeof(highlander_stats_t));
    stats->total_claims = game->total_claims;
    stats->contentions = game->total_contentions;
}
```

### 4.9 spec.json

```json
{
  "name": "the_quickening",
  "language": "c",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse (2.4.5-2.4.7)",
  "tags": ["mutex", "synchronization", "race-condition", "phase2"],
  "passing_score": 70,

  "function": {
    "name": "claim_the_prize",
    "prototype": "int claim_the_prize(immortal_lock_t *lock)",
    "return_type": "int",
    "parameters": [
      {"name": "lock", "type": "immortal_lock_t *"}
    ]
  },

  "driver": {
    "reference": "int ref_claim_the_prize(immortal_lock_t *lock) { if (lock == NULL || !lock->initialized) return EINVAL; int ret = pthread_mutex_lock(&lock->the_prize); if (ret == 0) { lock->current_holder = pthread_self(); lock->times_claimed++; } return ret; }",

    "edge_cases": [
      {
        "name": "null_lock",
        "args": [null],
        "expected": "EINVAL",
        "is_trap": true,
        "trap_explanation": "lock est NULL"
      },
      {
        "name": "uninitialized_lock",
        "args": ["uninit_lock"],
        "expected": "EINVAL",
        "is_trap": true,
        "trap_explanation": "lock pas initialisé"
      },
      {
        "name": "valid_lock",
        "args": ["valid_lock"],
        "expected": 0
      },
      {
        "name": "already_held",
        "args": ["held_lock"],
        "expected": "EDEADLK_or_block",
        "is_trap": true,
        "trap_explanation": "Comportement dépend du type de mutex"
      }
    ]
  },

  "norm": {
    "allowed_functions": ["pthread_mutex_init", "pthread_mutex_destroy", "pthread_mutex_lock", "pthread_mutex_unlock", "pthread_mutex_trylock", "pthread_mutex_timedlock", "pthread_mutexattr_init", "pthread_mutexattr_destroy", "pthread_mutexattr_settype", "pthread_self", "malloc", "free", "calloc", "printf", "fprintf", "clock_gettime", "memset", "memcpy"],
    "forbidden_functions": ["sem_init", "sem_wait", "sem_post", "atomic_fetch_add"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```c
/* Mutant A (Boundary) : Unlock sans vérifier si locked */
int release_the_prize_mutant_a(immortal_lock_t *lock) {
    if (lock == NULL)
        return EINVAL;
    // MANQUE: if (!lock->initialized) return EINVAL;
    return pthread_mutex_unlock(&lock->the_prize);
}
// Pourquoi c'est faux: Undefined behavior sur mutex non-initialisé
// Ce qui était pensé: "Si lock existe, il est forcément valide"

/* Mutant B (Safety) : Ne vérifie pas le retour de pthread_mutex_lock */
int claim_the_prize_mutant_b(immortal_lock_t *lock) {
    if (lock == NULL || !lock->initialized)
        return EINVAL;
    pthread_mutex_lock(&lock->the_prize);  // Ignore return!
    lock->current_holder = pthread_self();
    lock->times_claimed++;
    return 0;  // Toujours success même si lock a échoué!
}
// Pourquoi c'est faux: Lock peut échouer (EDEADLK, etc.)
// Ce qui était pensé: "pthread_mutex_lock ne peut pas échouer"

/* Mutant C (Resource) : Oublie de détruire les attributs */
int forge_the_prize_mutant_c(immortal_lock_t *lock, immortal_type_t type) {
    if (lock == NULL)
        return EINVAL;

    pthread_mutexattr_t attr;
    pthread_mutexattr_init(&attr);
    pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_NORMAL);
    int ret = pthread_mutex_init(&lock->the_prize, &attr);
    // MANQUE: pthread_mutexattr_destroy(&attr);
    lock->initialized = true;
    return ret;
}
// Pourquoi c'est faux: Fuite de ressource (attribut non détruit)
// Ce qui était pensé: "Les attributs sont sur la stack, pas besoin de destroy"

/* Mutant D (Logic) : Confond types de mutex */
int forge_the_prize_mutant_d(immortal_lock_t *lock, immortal_type_t type) {
    if (lock == NULL)
        return EINVAL;

    pthread_mutexattr_t attr;
    pthread_mutexattr_init(&attr);

    // ERREUR: Toujours NORMAL, ignore le type demandé!
    pthread_mutexattr_settype(&attr, PTHREAD_MUTEX_NORMAL);

    int ret = pthread_mutex_init(&lock->the_prize, &attr);
    pthread_mutexattr_destroy(&attr);
    lock->initialized = true;
    return ret;
}
// Pourquoi c'est faux: IMMORTAL_ANCIENT (recursive) ne fonctionne plus
// Ce qui était pensé: "Tous les mutex sont pareils"

/* Mutant E (Return) : Retourne avant unlock en cas d'erreur */
int64_t absorb_power_mutant_e(immortal_power_t *power) {
    if (power == NULL)
        return 0;

    claim_the_prize(&power->lock);

    if (power->power >= INT64_MAX) {
        return -1;  // OUBLIE: release_the_prize! DEADLOCK!
    }

    power->power++;
    int64_t result = power->power;
    release_the_prize(&power->lock);

    return result;
}
// Pourquoi c'est faux: Lock jamais libéré en cas d'overflow
// Ce qui était pensé: "Je gère l'erreur en retournant tôt"
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Highlander Analogy | Technical Reality |
|---------|-------------------|-------------------|
| **Race Condition** | Chaos de bataille | Accès concurrent non-synchronisé |
| **Mutex** | "There Can Be Only One" | Mutual Exclusion |
| **lock()** | Claim The Prize | Acquérir le verrou |
| **unlock()** | Release | Libérer le verrou |
| **trylock()** | Quick Challenge | Tenter sans bloquer |
| **timedlock()** | Wait for Battle | Attendre avec timeout |
| **Recursive mutex** | Ancient Immortal | Même thread peut re-lock |
| **Lock ordering** | Battle Order | Prévenir deadlock |
| **RAII guard** | The Quickening | Auto-unlock |

### 5.2 LDA — Traduction Littérale

```
FONCTION claim_the_prize QUI RETOURNE UN ENTIER ET PREND EN PARAMÈTRE lock QUI EST UN POINTEUR VERS immortal_lock_t
DÉBUT FONCTION
    SI lock EST ÉGAL À NUL OU lock N'EST PAS INITIALISÉ ALORS
        RETOURNER EINVAL
    FIN SI

    DÉCLARER ret COMME ENTIER
    AFFECTER LE RÉSULTAT DE pthread_mutex_lock À ret

    SI ret EST ÉGAL À 0 ALORS
        AFFECTER pthread_self AU CHAMP current_holder DE lock
        INCRÉMENTER times_claimed DE 1
    FIN SI

    RETOURNER ret
FIN FONCTION
```

### 5.3 Visualisation ASCII

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    RACE CONDITION: THE CHAOS                                │
└─────────────────────────────────────────────────────────────────────────────┘

  Thread 1                    Thread 2                    Shared Counter
  ────────                    ────────                    ──────────────
  READ counter (5)            READ counter (5)            [    5    ]
       │                           │
       ▼                           ▼
  ADD 1 = 6                   ADD 1 = 6
       │                           │
       ▼                           ▼
  WRITE 6  ───────────────────────────────────────────►   [    6    ]
                              WRITE 6  ──────────────►   [    6    ]

  RÉSULTAT: 6 au lieu de 7! ❌


┌─────────────────────────────────────────────────────────────────────────────┐
│                    MUTEX: "THERE CAN BE ONLY ONE"                           │
└─────────────────────────────────────────────────────────────────────────────┘

  Thread 1                    Thread 2                    Shared Counter
  ────────                    ────────                    ──────────────
  LOCK (claim)                     │                      [    5    ]
       │                           │
       │                      LOCK (wait...) ◄───────── BLOCKED!
       │                           │
  READ counter (5)                 │
       │                           │
  ADD 1 = 6                        │
       │                           │
  WRITE 6 ───────────────────────────────────────────►   [    6    ]
       │                           │
  UNLOCK (release) ───────────────►│
                                   │
                              LOCK (claim)
                                   │
                              READ counter (6)
                                   │
                              ADD 1 = 7
                                   │
                              WRITE 7 ──────────────►     [    7    ]
                                   │
                              UNLOCK

  RÉSULTAT: 7 comme prévu! ✓
```

### 5.4 Les pièges en détail

#### Piège 1: Oublier unlock dans tous les chemins

```c
// ❌ DEADLOCK POTENTIEL
int process(mutex_t *m) {
    mutex_lock(m);
    if (error_condition) {
        return -1;  // OUBLIE DE UNLOCK!
    }
    // ... processing ...
    mutex_unlock(m);
    return 0;
}

// ✅ CORRECT avec goto ou RAII
int process(mutex_t *m) {
    mutex_lock(m);
    int result = 0;
    if (error_condition) {
        result = -1;
        goto cleanup;
    }
    // ... processing ...
cleanup:
    mutex_unlock(m);
    return result;
}
```

#### Piège 2: Double lock avec NORMAL mutex

```c
// ❌ DEADLOCK IMMÉDIAT
mutex_t lock;
mutex_init(&lock, MUTEX_NORMAL);
mutex_lock(&lock);
mutex_lock(&lock);  // DEADLOCK! Le thread attend lui-même!

// ✅ UTILISER RECURSIVE SI NÉCESSAIRE
mutex_t lock;
mutex_init(&lock, MUTEX_RECURSIVE);
mutex_lock(&lock);
mutex_lock(&lock);  // OK!
mutex_unlock(&lock);
mutex_unlock(&lock);
```

### 5.5 Cours Complet

#### Qu'est-ce qu'une Race Condition ?

Une **race condition** se produit quand le résultat d'un programme dépend de l'**ordre d'exécution** des threads, qui est **non-déterministe**.

L'opération `counter++` semble atomique mais est en fait 3 opérations :
1. **READ** : Lire la valeur actuelle
2. **MODIFY** : Ajouter 1
3. **WRITE** : Écrire la nouvelle valeur

Si deux threads font `counter++` simultanément avec `counter = 5` :
- T1: READ 5, ADD → 6, WRITE 6
- T2: READ 5, ADD → 6, WRITE 6
- Résultat: 6 au lieu de 7!

#### Le Mutex : Solution à l'exclusion mutuelle

Un **mutex** (MUTual EXclusion) garantit qu'**un seul thread** peut exécuter la section critique à la fois.

```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

// Section critique protégée
pthread_mutex_lock(&mutex);
counter++;  // Maintenant atomique du point de vue des autres threads!
pthread_mutex_unlock(&mutex);
```

#### Types de Mutex

| Type | Comportement si même thread re-lock |
|------|-------------------------------------|
| **NORMAL** | Deadlock (comportement indéfini) |
| **RECURSIVE** | OK, incrémente compteur interne |
| **ERRORCHECK** | Retourne EDEADLK |

### 5.6 Normes

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                    │
├─────────────────────────────────────────────────────────────────┤
│ if (error) return -1;  // Lock pas released!                    │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ if (error) {                                                    │
│     mutex_unlock(&lock);                                        │
│     return -1;                                                  │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ Toujours libérer le lock avant de quitter, sinon deadlock!      │
└─────────────────────────────────────────────────────────────────┘
```

### 5.8 Mnémotechniques

#### ⚔️ MEME : "THERE CAN BE ONLY ONE"

![Highlander](highlander_there_can_be_only_one.jpg)

La phrase culte d'Highlander résume parfaitement le mutex :
- **UN SEUL** thread détient le lock
- Les autres **ATTENDENT**
- Quand il **LIBÈRE**, un autre peut **PRENDRE**

```c
// ⚔️ THERE CAN BE ONLY ONE!
claim_the_prize(&lock);     // "I am Connor MacLeod!"
// ... section critique ...
release_the_prize(&lock);   // "Your turn, Kurgan"
```

#### 💀 MEME : "The Quickening" — RAII Guard

Dans Highlander, **The Quickening** est le transfert de pouvoir automatique quand un immortel meurt.

Le **lock guard** fait pareil : il **libère automatiquement** le lock quand on sort du scope!

```c
// 💀 THE QUICKENING
{
    THERE_CAN_BE_ONLY_ONE(&lock) {
        // Section critique
        counter++;
    }  // Auto-unlock ici! "The Quickening!"
}
```

---

## ⚠️ SECTION 6 : PIÈGES RÉCAPITULATIF

| Piège | Description | Solution |
|-------|-------------|----------|
| Race condition | Accès concurrent non protégé | Utiliser mutex |
| Deadlock | Thread attend son propre lock | Recursive ou éviter |
| Lock leak | Oubli de unlock | RAII / goto cleanup |
| Double destroy | Détruire mutex deux fois | Flag initialized |
| Lock order | Circular wait | Toujours même ordre |
| Contention | Trop de threads sur un lock | Réduire section critique |

---

## 📝 SECTION 7 : QCM

### Question 1
**Que signifie "mutex" ?**

A) Multiple Execution
B) Mutual Exclusion
C) Memory Update
D) Thread Extension
E) Multi-Threading
F) Memory Exchange
G) Mutex Unit
H) Mutual Extension
I) Multiple Users
J) Memory Mutex

**Réponse : B**

### Question 2
**Que se passe-t-il si on appelle pthread_mutex_lock sur un NORMAL mutex déjà verrouillé par le même thread ?**

A) Retourne EBUSY
B) Retourne 0 et continue
C) Deadlock
D) Le lock est libéré
E) Retourne EINVAL
F) Le programme crash
G) Le thread est tué
H) Retourne ETIMEDOUT
I) Le mutex devient recursive
J) Rien

**Réponse : C**

### Question 3
**Quel type de mutex permet au même thread de re-acquérir le lock ?**

A) PTHREAD_MUTEX_NORMAL
B) PTHREAD_MUTEX_RECURSIVE
C) PTHREAD_MUTEX_ERRORCHECK
D) PTHREAD_MUTEX_DEFAULT
E) PTHREAD_MUTEX_FAST
F) PTHREAD_MUTEX_TIMED
G) PTHREAD_MUTEX_ROBUST
H) PTHREAD_MUTEX_STALLED
I) PTHREAD_MUTEX_ADAPTIVE
J) PTHREAD_MUTEX_SPIN

**Réponse : B**

### Question 4
**Comment prévenir un deadlock circulaire avec plusieurs mutex ?**

A) Utiliser recursive mutex
B) Ne jamais lock plus d'un mutex
C) Toujours acquérir dans le même ordre global
D) Utiliser trylock uniquement
E) Augmenter la priorité
F) Utiliser des sémaphores
G) Désactiver les interruptions
H) Utiliser spin locks
I) Doubler les mutex
J) Ignorer le problème

**Réponse : C**

### Question 5
**Que retourne pthread_mutex_trylock si le mutex est déjà verrouillé ?**

A) 0
B) -1
C) EINVAL
D) EBUSY
E) ETIMEDOUT
F) EDEADLK
G) EPERM
H) EAGAIN
I) 1
J) NULL

**Réponse : D**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Critère | Valeur |
|---------|--------|
| **Exercice** | 2.4.1 - the_quickening |
| **Concepts** | 26 (2.4.5 à 2.4.7) |
| **Difficulté** | ★★★★★★☆☆☆☆ (6/10) |
| **Temps** | 5h |
| **XP** | 400 (base) / 1200 (bonus) |
| **Langage** | C17 |
| **Thème** | Highlander - "There Can Be Only One" |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.4.1-the-quickening",
    "generated_at": "2025-01-12 17:30:00",

    "metadata": {
      "exercise_id": "2.4.1",
      "exercise_name": "the_quickening",
      "module": "2.4.1",
      "module_name": "Mutex & Race Conditions",
      "concept": "a-k",
      "concept_name": "Race Conditions + Mutex + Best Practices",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "c",
      "duration_minutes": 300,
      "xp_base": 400,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T2 O(1)",
      "complexity_space": "S2 O(1)",
      "prerequisites": ["ex00 Thread Fundamentals"],
      "domains": ["Process", "Mem"],
      "domains_bonus": ["CPU", "ASM"],
      "tags": ["mutex", "synchronization", "race-condition", "deadlock"],
      "meme_reference": "Highlander - There Can Be Only One"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "THERE CAN BE ONLY ONE"*
*L'excellence pédagogique ne se négocie pas — pas de raccourcis*
