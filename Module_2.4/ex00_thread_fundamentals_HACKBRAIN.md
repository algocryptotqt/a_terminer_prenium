<thinking>
## Analyse du Concept
- Concept : Thread Fundamentals (Thread concepts, POSIX threads, Attributes, TLS)
- Phase demandée : 2
- Adapté ? OUI - C'est un exercice fondamental sur le multithreading

## Combo Base + Bonus
- Exercice de base : Bibliothèque de gestion de threads avec attributs et TLS
- Bonus : Optimisation avec work-stealing et affinity
- Palier bonus : 🔥 Avancé
- Progression logique ? OUI

## Prérequis & Difficulté
- Prérequis réels : Process management, pointeurs, allocation mémoire
- Difficulté estimée : 6/10
- Cohérent avec phase ? OUI

## Aspect Fun/Culture
- Contexte choisi : Naruto - Shadow Clone Jutsu (影分身の術)
- MEME mnémotechnique : "I will become Hokage!" + Naruto créant des milliers de clones
- Pourquoi c'est fun :
  - Thread = Shadow Clone (Kage Bunshin)
  - pthread_create = shadow_clone_jutsu()
  - pthread_join = dispel_clone() (mémoire retourne à l'original)
  - Shared heap = Chakra partagé
  - Private stack = Expériences privées du clone
  - TLS = Mémoire privée de chaque clone
  - Thread exhaustion = Chakra épuisé

## Scénarios d'Échec (5 mutants)
1. Mutant A (Boundary) : Créer plus de threads que MAX_THREADS sans vérification
2. Mutant B (Safety) : Ne pas vérifier le retour de pthread_create
3. Mutant C (Resource) : Oublier pthread_join sur threads joinables → zombie threads
4. Mutant D (Logic) : Confondre PTHREAD_CREATE_DETACHED et PTHREAD_CREATE_JOINABLE
5. Mutant E (Return) : Ne pas récupérer proprement le retour du thread (void**)

## Verdict
VALIDE - Analogie Naruto/Shadow Clone excellente pour le threading
Score: 98/100
</thinking>

---

# Exercice 2.4.0 : shadow_clone_jutsu

**Module :**
2.4.0 — Thread Fundamentals

**Concept :**
a-k — Thread Concepts + POSIX Threads + Attributes + TLS (41 concepts)

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (tous concepts 2.4.1 à 2.4.4)

**Langage :**
C (C17)

**Prérequis :**
- Module 2.2 (Process Management)
- Pointeurs et allocation mémoire
- Gestion des erreurs

**Domaines :**
Process, Mem, Struct

**Durée estimée :**
360 min (6h)

**XP Base :**
500

**Complexité :**
T3 O(n) × S3 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex00/
├── kage_bunshin.h
├── kage_bunshin.c
├── clone_attributes.c
├── clone_memory.c
├── chakra_benchmark.c
└── Makefile
```

**Fonctions autorisées :**
- pthread_create, pthread_join, pthread_detach, pthread_exit
- pthread_self, pthread_equal
- pthread_attr_init, pthread_attr_destroy, pthread_attr_set*
- pthread_key_create, pthread_key_delete, pthread_getspecific, pthread_setspecific
- malloc, free, calloc, realloc
- printf, fprintf, sprintf
- clock_gettime, gettimeofday
- memset, memcpy, strncpy

**Fonctions interdites :**
- fork, exec*, system
- signal, sigaction (pour cet exercice)
- sleep, usleep (sauf pour les tests)

### 1.2 Consigne

**🍥 NARUTO : SHADOW CLONE JUTSU (影分身の術)**

Dans l'univers de Naruto, le **Kage Bunshin no Jutsu** (Shadow Clone Technique) est une technique ninja qui crée des copies physiques du shinobi. Chaque clone :
- **Partage le chakra** de l'original (comme les threads partagent le heap)
- **A sa propre existence physique** (comme les threads ont leur propre stack)
- **Retourne son expérience** à l'original quand il disparaît (comme pthread_join récupère le résultat)
- Peut être **détaché** (clone explosif qui n'a pas besoin d'être rejoint)

Naruto est célèbre pour créer des **milliers de clones** simultanément grâce à son immense réserve de chakra (le Nine-Tails). Aujourd'hui, tu vas implémenter le **Kage Bunshin System** en C !

**Ta mission :**

Créer une bibliothèque complète de gestion de threads inspirée du Shadow Clone Jutsu.

**Entrée :**
- `ninja_t *naruto` : Le ninja original (thread manager)
- `clone_attr_t *attr` : Les attributs du clone (stack size, détachement, etc.)
- `void *(*jutsu)(void*)` : La technique que le clone doit exécuter
- `void *chakra_data` : Les données passées au clone

**Sortie :**
- `clone_t *` : Le clone créé, ou NULL en cas d'échec
- `void *experience` : L'expérience acquise par le clone (via join)

**Contraintes :**
- Chaque clone doit avoir un ID unique
- Les clones doivent pouvoir être nommés (ex: "Naruto Clone #42")
- Le TLS permet à chaque clone d'avoir sa propre "mémoire privée"
- Gestion propre des ressources (pas de chakra leak = memory leak)
- Thread-safe : plusieurs ninjas peuvent créer des clones simultanément

**Exemples :**

| Appel | Résultat | Explication |
|-------|----------|-------------|
| `shadow_clone_jutsu(naruto, NULL, rasengan, data)` | Clone créé | Clone avec attributs par défaut |
| `dispel_clone(clone, &experience)` | Expérience récupérée | pthread_join équivalent |
| `release_clone(clone)` | Clone détaché | N'a pas besoin d'être join |
| `kage_bunshin_tarengan(naruto, 100, jutsu, data)` | 100 clones | Multi Shadow Clone Jutsu |

### 1.2.2 Consigne Académique

Implémenter une bibliothèque de gestion de threads POSIX avec :
1. Création et destruction de threads
2. Gestion des attributs (stack, détachement, scheduling)
3. Thread-Local Storage (TLS) pour données privées par thread
4. Benchmarking pour démontrer les bénéfices du parallélisme

### 1.3 Prototypes

```c
#ifndef KAGE_BUNSHIN_H
#define KAGE_BUNSHIN_H

#include <pthread.h>
#include <stdint.h>
#include <stdbool.h>

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.1: THREAD CONCEPTS — Clone Information
// ═══════════════════════════════════════════════════════════════════════════

// a-d: Thread/Clone info structure
typedef struct {
    pthread_t spirit;              // c: Thread identifier (clone's spirit)
    int clone_number;              // Internal ID (Naruto Clone #X)
    char name[64];                 // Clone name
    void *(*jutsu)(void*);         // e: Start function (technique to execute)
    void *chakra_data;             // f: Argument (chakra/data passed)
    void *experience;              // h: Return value (what clone learned)
    bool joinable;                 // Can be dispelled and experience retrieved
    bool active;                   // Is clone still active
    bool dispelled;                // Has clone been dispelled
    uint64_t summoned_at;          // When clone was created
    uint64_t released_at;          // When clone was released
} clone_t;

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.3: THREAD ATTRIBUTES — Clone Attributes
// ═══════════════════════════════════════════════════════════════════════════

typedef struct {
    pthread_attr_t attr;
    size_t chakra_reserve;         // d: Stack size (chakra reserve)
    void *chakra_location;         // e: Stack address
    size_t protection_seal;        // f: Guard size (protection barrier)
    int shadow_type;               // c: Detach state (solid vs shadow)
    int combat_style;              // g: Scheduling policy
    int power_level;               // h: Priority
    int battle_scope;              // i: Scope (system or process)
} clone_attr_t;

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.4: THREAD-LOCAL STORAGE — Clone's Private Memory
// ═══════════════════════════════════════════════════════════════════════════

typedef struct {
    pthread_key_t memory_seal;     // The key to private memory
    void (*on_dispel)(void*);      // g: Destructor (called when clone dispels)
    const char *memory_name;       // Name of this memory slot
    bool sealed;                   // Is this memory initialized
} clone_memory_t;

// ═══════════════════════════════════════════════════════════════════════════
// Ninja (Thread Manager) — The Original
// ═══════════════════════════════════════════════════════════════════════════

typedef struct {
    clone_t *clones;               // Array of all clones
    size_t clone_count;            // Current number of clones
    size_t max_clones;             // Maximum clones (chakra limit)
    clone_memory_t *memories;      // TLS keys
    size_t memory_count;
    size_t max_memories;
    uint64_t total_summoned;       // Stats: total clones created
    uint64_t total_dispelled;      // Stats: total clones joined
    char ninja_name[64];           // Name of the ninja
} ninja_t;

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.1.g-k: Threading Models (User vs Kernel threads)
// ═══════════════════════════════════════════════════════════════════════════

typedef enum {
    NINJA_ACADEMY,           // g: User-level threads (illusion clones)
    JONIN_LEVEL,             // h: Kernel-level threads (real clones)
    MANY_TO_ONE_SEAL,        // i: M:1 mapping
    ONE_TO_ONE_SEAL,         // j: 1:1 mapping (Linux default)
    MANY_TO_MANY_SEAL        // k: M:N hybrid
} clone_model_t;

// ═══════════════════════════════════════════════════════════════════════════
// API — Main Functions
// ═══════════════════════════════════════════════════════════════════════════

// Ninja lifecycle
ninja_t *become_ninja(const char *name);
void retire_ninja(ninja_t *ninja);

// 2.4.2: POSIX Threads API — Clone Jutsu
clone_t *shadow_clone_jutsu(ninja_t *ninja, clone_attr_t *attr,
                            void *(*jutsu)(void*), void *chakra);          // d: create
int dispel_clone(ninja_t *ninja, clone_t *clone, void **experience);       // i: join
int release_clone(ninja_t *ninja, clone_t *clone);                         // j: detach
pthread_t my_spirit(void);                                                  // k: self
bool same_spirit(pthread_t s1, pthread_t s2);                              // l: equal
void vanish(void *experience);                                              // g: exit

// Multi-clone jutsu (create many at once)
int kage_bunshin_tarengan(ninja_t *ninja, int count,
                          void *(*jutsu)(void*), void *chakra,
                          clone_t **clones);

// 2.4.3: Clone Attributes
int init_clone_seal(clone_attr_t *attr);                                   // b: init
int destroy_clone_seal(clone_attr_t *attr);                                // j: destroy
int set_shadow_type(clone_attr_t *attr, int type);                         // c: detach
int set_chakra_reserve(clone_attr_t *attr, size_t size);                   // d: stack size
int set_chakra_location(clone_attr_t *attr, void *addr, size_t size);      // e: stack
int set_protection_seal(clone_attr_t *attr, size_t size);                  // f: guard
int set_combat_style(clone_attr_t *attr, int policy);                      // g: sched
int set_power_level(clone_attr_t *attr, int priority);                     // h: priority
int set_battle_scope(clone_attr_t *attr, int scope);                       // i: scope

// Attribute getters
int get_shadow_type(clone_attr_t *attr, int *type);
int get_chakra_reserve(clone_attr_t *attr, size_t *size);
int get_protection_seal(clone_attr_t *attr, size_t *size);

// 2.4.4: Clone's Private Memory (TLS)
int create_memory_seal(ninja_t *ninja, clone_memory_t **memory,
                       void (*on_dispel)(void*));                          // d: key create
int destroy_memory_seal(ninja_t *ninja, clone_memory_t *memory);
void *recall_memory(clone_memory_t *memory);                               // e: getspecific
int store_memory(clone_memory_t *memory, void *data);                      // f: setspecific

// 2.4.4.b-c: Modern TLS demonstration
void demonstrate_private_memory(void);

// 2.4.1.e: Benchmark parallel benefits
typedef struct {
    double sequential_time_ms;
    double parallel_time_ms;
    double speedup;
    int clone_count;
} jutsu_benchmark_t;

void benchmark_jutsu(int num_clones, jutsu_benchmark_t *result);

// 2.4.1.g-k: Threading models
void explain_clone_model(clone_model_t model);
clone_model_t detect_village_model(void);

// Utility
void name_clone(clone_t *clone, const char *name);
const char *get_clone_name(clone_t *clone);
void list_all_clones(ninja_t *ninja);
int active_clone_count(ninja_t *ninja);

// Statistics
typedef struct {
    uint64_t clones_summoned;
    uint64_t clones_dispelled;
    uint64_t clones_released;
    uint64_t memories_created;
    double avg_clone_lifespan_ms;
} ninja_stats_t;

void get_ninja_stats(ninja_t *ninja, ninja_stats_t *stats);

#endif // KAGE_BUNSHIN_H
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Pourquoi le Shadow Clone est l'analogie parfaite

Dans Naruto, le **Kage Bunshin no Jutsu** a des propriétés étonnamment similaires aux threads :

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  NARUTO SHADOW CLONE                    │   POSIX THREAD                    │
├─────────────────────────────────────────┼───────────────────────────────────┤
│  Original Naruto                        │   Main thread / Process           │
│  Shadow Clone                           │   pthread_t                       │
│  Chakra (partagé)                       │   Heap memory (shared)            │
│  Corps physique du clone                │   Stack (private)                 │
│  Expérience acquise                     │   Return value (void*)            │
│  Kage Bunshin no Jutsu!                 │   pthread_create()                │
│  Clone disparaît                        │   pthread_join()                  │
│  Clone explosif (Bunshin Daibakuha)     │   pthread_detach()                │
│  Mémoire privée du clone                │   Thread-Local Storage            │
│  Chakra épuisé                          │   Resource exhaustion             │
│  Multi Shadow Clone (Tajuu Kage Bunshin)│   Thread pool                     │
└─────────────────────────────────────────┴───────────────────────────────────┘
```

### 2.2 Le secret de Naruto : Pourquoi il peut créer des milliers de clones

Dans l'anime, Naruto peut créer des **milliers de clones** là où un ninja normal n'en créerait que quelques-uns. La raison ? Le **Nine-Tails (Kyuubi)** lui fournit une réserve de chakra quasi-illimitée.

C'est exactement comme un serveur avec beaucoup de RAM qui peut créer plus de threads qu'une machine avec peu de mémoire !

### 2.5 DANS LA VRAIE VIE

| Métier | Usage du Threading |
|--------|-------------------|
| **Game Developer** | Threads séparés pour rendu, physique, IA, réseau |
| **Backend Engineer** | Thread pool pour gérer des milliers de requêtes HTTP |
| **Data Scientist** | Parallélisation des calculs matriciels (NumPy, TensorFlow) |
| **Systems Programmer** | Écriture de serveurs haute performance (nginx, Redis) |
| **DevOps** | Scripts de déploiement parallèle sur plusieurs serveurs |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
kage_bunshin.h  kage_bunshin.c  clone_attributes.c  clone_memory.c  chakra_benchmark.c  main.c  Makefile

$ make
gcc -Wall -Wextra -std=c17 -pthread -c kage_bunshin.c
gcc -Wall -Wextra -std=c17 -pthread -c clone_attributes.c
gcc -Wall -Wextra -std=c17 -pthread -c clone_memory.c
gcc -Wall -Wextra -std=c17 -pthread -c chakra_benchmark.c
ar rcs libkagebunshin.a kage_bunshin.o clone_attributes.o clone_memory.o chakra_benchmark.o
gcc -Wall -Wextra -std=c17 -pthread main.c -L. -lkagebunshin -o shadow_clone_demo

$ ./shadow_clone_demo
=== NARUTO'S SHADOW CLONE JUTSU ===
Ninja 'Naruto Uzumaki' has entered the battlefield!

Detecting village threading model...
Village uses ONE_TO_ONE_SEAL (1:1 - Linux default, real clones!)

Creating clone with custom attributes...
Clone attributes: 2MB chakra reserve, 4KB protection seal

KAGE BUNSHIN NO JUTSU!
Created: Naruto Clone #1
Created: Naruto Clone #2
Created: Naruto Clone #3
Created: Naruto Clone #4

Clone #1: Training Rasengan... TLS counter = 1
Clone #2: Training Rasengan... TLS counter = 2
Clone #3: Training Rasengan... TLS counter = 3
Clone #4: Training Rasengan... TLS counter = 4

Main ninja spirit: 140234567890112

Dispelling clones and gathering experience...
Clone #1 experience: 1000100
Clone #2 experience: 1000200
Clone #3 experience: 1000300
Clone #4 experience: 1000400

=== CHAKRA BENCHMARK ===
Sequential jutsu: 412.34ms
Parallel jutsu (4 clones): 108.56ms
Speedup: 3.80x

=== NINJA STATS ===
Clones summoned: 4
Clones dispelled: 4
Average clone lifespan: 102.34ms

Naruto Uzumaki retires from battle.
All chakra released. No leaks!
```

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(1) amortized pour work-stealing

**Space Complexity attendue :**
O(n) où n = nombre de threads

**Domaines Bonus :**
`CPU, Struct`

#### 3.1.1 Consigne Bonus

**🍥 SAGE MODE : ADVANCED CLONE TECHNIQUES**

Naruto en **Sage Mode** peut créer des clones encore plus puissants avec des capacités avancées. Implémente :

1. **Work-Stealing** : Quand un clone finit sa tâche, il peut "voler" du travail à un autre clone (comme Naruto qui aide ses clones)
2. **CPU Affinity** : Attacher un clone à un CPU spécifique (comme assigner un clone à une zone de bataille)
3. **Clone Priority Inheritance** : Éviter l'inversion de priorité

**Ta mission :**

```c
// Work-stealing queue
typedef struct {
    void **tasks;
    size_t head, tail;
    pthread_mutex_t lock;
} steal_queue_t;

// Sage mode clone with work-stealing
clone_t *sage_clone_jutsu(ninja_t *ninja, steal_queue_t *queue,
                          void *(*jutsu)(void*));

// CPU affinity (bind clone to specific core)
int bind_clone_to_battlefield(clone_t *clone, int cpu_id);

// Priority inheritance to avoid inversion
int enable_priority_inheritance(clone_attr_t *attr);
```

**Contraintes :**
┌─────────────────────────────────────────┐
│  Work-stealing doit être lock-free      │
│  Affinity via pthread_setaffinity_np    │
│  PTHREAD_PRIO_INHERIT pour PI           │
│  Temps limite steal : O(1) amorti       │
└─────────────────────────────────────────┘

#### 3.1.2 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Scheduling | FIFO simple | Work-stealing |
| Affinity | Aucune | CPU binding |
| Priority | Fixe | Inheritance |
| Complexité | O(n) create | O(1) steal |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (Tests)

| Test | Description | Points | Trap |
|------|-------------|--------|------|
| `test_ninja_create` | Création/destruction ninja | 5 | NULL check |
| `test_shadow_clone_basic` | Clone simple | 10 | Return check |
| `test_clone_join` | Dispel et récupération expérience | 10 | Double join |
| `test_clone_detach` | Release sans join | 10 | Join after detach |
| `test_self_equal` | my_spirit et same_spirit | 5 | - |
| `test_clone_exit` | vanish() avec valeur | 5 | - |
| `test_attr_init_destroy` | Lifecycle attributs | 5 | Double destroy |
| `test_attr_detach_state` | JOINABLE vs DETACHED | 10 | - |
| `test_attr_stack` | Chakra reserve custom | 10 | Size < minimum |
| `test_attr_guard` | Protection seal | 5 | - |
| `test_tls_create` | Memory seal création | 5 | - |
| `test_tls_get_set` | Store/recall memory | 10 | NULL key |
| `test_tls_destructor` | on_dispel appelé | 10 | - |
| `test_multi_clone` | Tajuu Kage Bunshin | 10 | Resource exhaustion |
| `test_benchmark` | Speedup > 1.5x | 5 | - |
| **Total** | | **100** | |

### 4.2 main.c de test

```c
#include "kage_bunshin.h"
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>

// TLS demonstration
__thread int clone_local_counter = 0;

void *rasengan_training(void *arg) {
    int id = *(int*)arg;

    // Each clone has private stack
    int training_points = id * 100;

    // TLS: each clone has own counter
    clone_local_counter = id;
    printf("Clone #%d: TLS counter = %d\n", id, clone_local_counter);

    // Do training
    for (int i = 0; i < 1000000; i++) {
        training_points++;
    }

    // Return experience
    int *experience = malloc(sizeof(int));
    *experience = training_points;
    return experience;
}

int main(void) {
    printf("=== NARUTO'S SHADOW CLONE JUTSU ===\n");

    // Create ninja
    ninja_t *naruto = become_ninja("Naruto Uzumaki");
    assert(naruto != NULL);
    printf("Ninja '%s' has entered the battlefield!\n\n", naruto->ninja_name);

    // Detect threading model
    printf("Detecting village threading model...\n");
    clone_model_t model = detect_village_model();
    explain_clone_model(model);
    printf("\n");

    // Custom attributes
    printf("Creating clone with custom attributes...\n");
    clone_attr_t attr;
    init_clone_seal(&attr);
    set_chakra_reserve(&attr, 2 * 1024 * 1024);  // 2MB
    set_protection_seal(&attr, 4096);             // 4KB guard
    set_shadow_type(&attr, PTHREAD_CREATE_JOINABLE);
    printf("Clone attributes: 2MB chakra reserve, 4KB protection seal\n\n");

    // Create clones
    printf("KAGE BUNSHIN NO JUTSU!\n");
    clone_t *clones[4];
    int ids[4] = {1, 2, 3, 4};

    for (int i = 0; i < 4; i++) {
        clones[i] = shadow_clone_jutsu(naruto, &attr, rasengan_training, &ids[i]);
        assert(clones[i] != NULL);
        char name[32];
        snprintf(name, sizeof(name), "Naruto Clone #%d", i + 1);
        name_clone(clones[i], name);
        printf("Created: %s\n", get_clone_name(clones[i]));
    }
    printf("\n");

    // Wait a bit for clones to print
    usleep(100000);

    // Self
    printf("\nMain ninja spirit: %lu\n\n", (unsigned long)my_spirit());

    // Dispel and gather experience
    printf("Dispelling clones and gathering experience...\n");
    for (int i = 0; i < 4; i++) {
        void *experience;
        dispel_clone(naruto, clones[i], &experience);
        printf("Clone #%d experience: %d\n", i + 1, *(int*)experience);
        free(experience);
    }

    // Benchmark
    printf("\n=== CHAKRA BENCHMARK ===\n");
    jutsu_benchmark_t bench;
    benchmark_jutsu(4, &bench);
    printf("Sequential jutsu: %.2fms\n", bench.sequential_time_ms);
    printf("Parallel jutsu (%d clones): %.2fms\n",
           bench.clone_count, bench.parallel_time_ms);
    printf("Speedup: %.2fx\n", bench.speedup);

    // Stats
    printf("\n=== NINJA STATS ===\n");
    ninja_stats_t stats;
    get_ninja_stats(naruto, &stats);
    printf("Clones summoned: %lu\n", stats.clones_summoned);
    printf("Clones dispelled: %lu\n", stats.clones_dispelled);
    printf("Average clone lifespan: %.2fms\n", stats.avg_clone_lifespan_ms);

    // Cleanup
    destroy_clone_seal(&attr);
    retire_ninja(naruto);
    printf("\nNaruto Uzumaki retires from battle.\n");
    printf("All chakra released. No leaks!\n");

    return 0;
}
```

### 4.3 Solution de référence

```c
// kage_bunshin.c — Solution de référence
#include "kage_bunshin.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <errno.h>

#define DEFAULT_MAX_CLONES 1024
#define DEFAULT_MAX_MEMORIES 64

// ═══════════════════════════════════════════════════════════════════════════
// Helper: Get current time in nanoseconds
// ═══════════════════════════════════════════════════════════════════════════

static uint64_t get_time_ns(void) {
    struct timespec ts;
    clock_gettime(CLOCK_MONOTONIC, &ts);
    return (uint64_t)ts.tv_sec * 1000000000ULL + ts.tv_nsec;
}

// ═══════════════════════════════════════════════════════════════════════════
// Ninja Lifecycle
// ═══════════════════════════════════════════════════════════════════════════

ninja_t *become_ninja(const char *name) {
    if (name == NULL)
        return NULL;

    ninja_t *ninja = calloc(1, sizeof(ninja_t));
    if (ninja == NULL)
        return NULL;

    ninja->clones = calloc(DEFAULT_MAX_CLONES, sizeof(clone_t));
    if (ninja->clones == NULL) {
        free(ninja);
        return NULL;
    }

    ninja->memories = calloc(DEFAULT_MAX_MEMORIES, sizeof(clone_memory_t));
    if (ninja->memories == NULL) {
        free(ninja->clones);
        free(ninja);
        return NULL;
    }

    ninja->max_clones = DEFAULT_MAX_CLONES;
    ninja->max_memories = DEFAULT_MAX_MEMORIES;
    ninja->clone_count = 0;
    ninja->memory_count = 0;
    ninja->total_summoned = 0;
    ninja->total_dispelled = 0;

    strncpy(ninja->ninja_name, name, sizeof(ninja->ninja_name) - 1);
    ninja->ninja_name[sizeof(ninja->ninja_name) - 1] = '\0';

    return ninja;
}

void retire_ninja(ninja_t *ninja) {
    if (ninja == NULL)
        return;

    // Join any remaining joinable clones
    for (size_t i = 0; i < ninja->clone_count; i++) {
        if (ninja->clones[i].active && ninja->clones[i].joinable) {
            pthread_join(ninja->clones[i].spirit, NULL);
        }
    }

    // Destroy TLS keys
    for (size_t i = 0; i < ninja->memory_count; i++) {
        if (ninja->memories[i].sealed) {
            pthread_key_delete(ninja->memories[i].memory_seal);
        }
    }

    free(ninja->clones);
    free(ninja->memories);
    free(ninja);
}

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.2: Shadow Clone Jutsu (pthread_create wrapper)
// ═══════════════════════════════════════════════════════════════════════════

clone_t *shadow_clone_jutsu(ninja_t *ninja, clone_attr_t *attr,
                            void *(*jutsu)(void*), void *chakra) {
    if (ninja == NULL || jutsu == NULL)
        return NULL;

    if (ninja->clone_count >= ninja->max_clones)
        return NULL;  // Chakra exhausted!

    clone_t *clone = &ninja->clones[ninja->clone_count];
    memset(clone, 0, sizeof(clone_t));

    clone->clone_number = (int)(ninja->total_summoned + 1);
    clone->jutsu = jutsu;
    clone->chakra_data = chakra;
    clone->joinable = true;
    clone->active = false;
    clone->dispelled = false;
    clone->summoned_at = get_time_ns();

    snprintf(clone->name, sizeof(clone->name), "Clone #%d", clone->clone_number);

    pthread_attr_t *pattr = (attr != NULL) ? &attr->attr : NULL;

    // Check detach state from attributes
    if (attr != NULL) {
        int detach_state;
        pthread_attr_getdetachstate(&attr->attr, &detach_state);
        clone->joinable = (detach_state == PTHREAD_CREATE_JOINABLE);
    }

    int ret = pthread_create(&clone->spirit, pattr, jutsu, chakra);
    if (ret != 0) {
        return NULL;
    }

    clone->active = true;
    ninja->clone_count++;
    ninja->total_summoned++;

    return clone;
}

int dispel_clone(ninja_t *ninja, clone_t *clone, void **experience) {
    if (ninja == NULL || clone == NULL)
        return EINVAL;

    if (!clone->joinable)
        return EINVAL;  // Can't dispel a released clone

    if (clone->dispelled)
        return EINVAL;  // Already dispelled

    int ret = pthread_join(clone->spirit, experience);
    if (ret == 0) {
        clone->active = false;
        clone->dispelled = true;
        clone->released_at = get_time_ns();
        ninja->total_dispelled++;
    }

    return ret;
}

int release_clone(ninja_t *ninja, clone_t *clone) {
    if (ninja == NULL || clone == NULL)
        return EINVAL;

    if (!clone->joinable)
        return EINVAL;  // Already released

    int ret = pthread_detach(clone->spirit);
    if (ret == 0) {
        clone->joinable = false;
    }

    return ret;
}

pthread_t my_spirit(void) {
    return pthread_self();
}

bool same_spirit(pthread_t s1, pthread_t s2) {
    return pthread_equal(s1, s2) != 0;
}

void vanish(void *experience) {
    pthread_exit(experience);
}

int kage_bunshin_tarengan(ninja_t *ninja, int count,
                          void *(*jutsu)(void*), void *chakra,
                          clone_t **clones) {
    if (ninja == NULL || jutsu == NULL || clones == NULL || count <= 0)
        return -1;

    int created = 0;
    for (int i = 0; i < count; i++) {
        clones[i] = shadow_clone_jutsu(ninja, NULL, jutsu, chakra);
        if (clones[i] != NULL) {
            created++;
        } else {
            break;  // Chakra exhausted
        }
    }

    return created;
}

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.3: Clone Attributes
// ═══════════════════════════════════════════════════════════════════════════

int init_clone_seal(clone_attr_t *attr) {
    if (attr == NULL)
        return EINVAL;

    memset(attr, 0, sizeof(clone_attr_t));
    return pthread_attr_init(&attr->attr);
}

int destroy_clone_seal(clone_attr_t *attr) {
    if (attr == NULL)
        return EINVAL;

    return pthread_attr_destroy(&attr->attr);
}

int set_shadow_type(clone_attr_t *attr, int type) {
    if (attr == NULL)
        return EINVAL;

    attr->shadow_type = type;
    return pthread_attr_setdetachstate(&attr->attr, type);
}

int set_chakra_reserve(clone_attr_t *attr, size_t size) {
    if (attr == NULL)
        return EINVAL;

    attr->chakra_reserve = size;
    return pthread_attr_setstacksize(&attr->attr, size);
}

int set_chakra_location(clone_attr_t *attr, void *addr, size_t size) {
    if (attr == NULL)
        return EINVAL;

    attr->chakra_location = addr;
    attr->chakra_reserve = size;
    return pthread_attr_setstack(&attr->attr, addr, size);
}

int set_protection_seal(clone_attr_t *attr, size_t size) {
    if (attr == NULL)
        return EINVAL;

    attr->protection_seal = size;
    return pthread_attr_setguardsize(&attr->attr, size);
}

int set_combat_style(clone_attr_t *attr, int policy) {
    if (attr == NULL)
        return EINVAL;

    attr->combat_style = policy;
    return pthread_attr_setschedpolicy(&attr->attr, policy);
}

int set_power_level(clone_attr_t *attr, int priority) {
    if (attr == NULL)
        return EINVAL;

    attr->power_level = priority;
    struct sched_param param = { .sched_priority = priority };
    return pthread_attr_setschedparam(&attr->attr, &param);
}

int set_battle_scope(clone_attr_t *attr, int scope) {
    if (attr == NULL)
        return EINVAL;

    attr->battle_scope = scope;
    return pthread_attr_setscope(&attr->attr, scope);
}

int get_shadow_type(clone_attr_t *attr, int *type) {
    if (attr == NULL || type == NULL)
        return EINVAL;
    return pthread_attr_getdetachstate(&attr->attr, type);
}

int get_chakra_reserve(clone_attr_t *attr, size_t *size) {
    if (attr == NULL || size == NULL)
        return EINVAL;
    return pthread_attr_getstacksize(&attr->attr, size);
}

int get_protection_seal(clone_attr_t *attr, size_t *size) {
    if (attr == NULL || size == NULL)
        return EINVAL;
    return pthread_attr_getguardsize(&attr->attr, size);
}

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.4: Clone's Private Memory (TLS)
// ═══════════════════════════════════════════════════════════════════════════

int create_memory_seal(ninja_t *ninja, clone_memory_t **memory,
                       void (*on_dispel)(void*)) {
    if (ninja == NULL || memory == NULL)
        return EINVAL;

    if (ninja->memory_count >= ninja->max_memories)
        return ENOMEM;

    clone_memory_t *mem = &ninja->memories[ninja->memory_count];
    mem->on_dispel = on_dispel;
    mem->sealed = false;

    int ret = pthread_key_create(&mem->memory_seal, on_dispel);
    if (ret == 0) {
        mem->sealed = true;
        ninja->memory_count++;
        *memory = mem;
    }

    return ret;
}

int destroy_memory_seal(ninja_t *ninja, clone_memory_t *memory) {
    if (ninja == NULL || memory == NULL)
        return EINVAL;

    if (!memory->sealed)
        return EINVAL;

    int ret = pthread_key_delete(memory->memory_seal);
    if (ret == 0) {
        memory->sealed = false;
    }

    return ret;
}

void *recall_memory(clone_memory_t *memory) {
    if (memory == NULL || !memory->sealed)
        return NULL;

    return pthread_getspecific(memory->memory_seal);
}

int store_memory(clone_memory_t *memory, void *data) {
    if (memory == NULL || !memory->sealed)
        return EINVAL;

    return pthread_setspecific(memory->memory_seal, data);
}

// ═══════════════════════════════════════════════════════════════════════════
// Utility Functions
// ═══════════════════════════════════════════════════════════════════════════

void name_clone(clone_t *clone, const char *name) {
    if (clone == NULL || name == NULL)
        return;

    strncpy(clone->name, name, sizeof(clone->name) - 1);
    clone->name[sizeof(clone->name) - 1] = '\0';
}

const char *get_clone_name(clone_t *clone) {
    if (clone == NULL)
        return NULL;
    return clone->name;
}

void list_all_clones(ninja_t *ninja) {
    if (ninja == NULL)
        return;

    printf("=== Active Clones for %s ===\n", ninja->ninja_name);
    for (size_t i = 0; i < ninja->clone_count; i++) {
        clone_t *c = &ninja->clones[i];
        printf("[%zu] %s - %s, %s\n",
               i, c->name,
               c->active ? "active" : "inactive",
               c->joinable ? "joinable" : "detached");
    }
}

int active_clone_count(ninja_t *ninja) {
    if (ninja == NULL)
        return 0;

    int count = 0;
    for (size_t i = 0; i < ninja->clone_count; i++) {
        if (ninja->clones[i].active)
            count++;
    }
    return count;
}

void get_ninja_stats(ninja_t *ninja, ninja_stats_t *stats) {
    if (ninja == NULL || stats == NULL)
        return;

    memset(stats, 0, sizeof(ninja_stats_t));
    stats->clones_summoned = ninja->total_summoned;
    stats->clones_dispelled = ninja->total_dispelled;
    stats->memories_created = ninja->memory_count;

    // Calculate average lifespan
    uint64_t total_lifespan = 0;
    int dispelled_count = 0;

    for (size_t i = 0; i < ninja->clone_count; i++) {
        clone_t *c = &ninja->clones[i];
        if (c->dispelled && c->released_at > c->summoned_at) {
            total_lifespan += (c->released_at - c->summoned_at);
            dispelled_count++;
        }
    }

    if (dispelled_count > 0) {
        stats->avg_clone_lifespan_ms = (double)total_lifespan / dispelled_count / 1000000.0;
    }
}

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.1.g-k: Threading Models
// ═══════════════════════════════════════════════════════════════════════════

void explain_clone_model(clone_model_t model) {
    switch (model) {
        case NINJA_ACADEMY:
            printf("NINJA_ACADEMY (User-level threads): Illusion clones, managed by library\n");
            break;
        case JONIN_LEVEL:
            printf("JONIN_LEVEL (Kernel threads): Real clones, managed by OS\n");
            break;
        case MANY_TO_ONE_SEAL:
            printf("MANY_TO_ONE_SEAL: Multiple user clones map to one kernel entity\n");
            break;
        case ONE_TO_ONE_SEAL:
            printf("ONE_TO_ONE_SEAL: Each clone is a real kernel thread (Linux default)\n");
            break;
        case MANY_TO_MANY_SEAL:
            printf("MANY_TO_MANY_SEAL: Hybrid model, flexible mapping\n");
            break;
    }
}

clone_model_t detect_village_model(void) {
    // Linux uses 1:1 model (NPTL)
    #ifdef __linux__
    return ONE_TO_ONE_SEAL;
    #else
    return JONIN_LEVEL;
    #endif
}

// ═══════════════════════════════════════════════════════════════════════════
// 2.4.4.b-c: Modern TLS Demo
// ═══════════════════════════════════════════════════════════════════════════

__thread int gcc_tls_demo = 0;

void demonstrate_private_memory(void) {
    printf("=== Private Memory (TLS) Demo ===\n");
    printf("__thread keyword (GCC): gcc_tls_demo = %d\n", gcc_tls_demo);
    gcc_tls_demo = 42;
    printf("After assignment: gcc_tls_demo = %d\n", gcc_tls_demo);
}
```

### 4.4 Solutions alternatives acceptées

```c
// Alternative 1: Using static thread count instead of dynamic array
// Acceptable if MAX_THREADS is reasonable

// Alternative 2: Using linked list for clones instead of array
// Acceptable, may have different performance characteristics

// Alternative 3: Using atomic counters for thread-safe stats
// Actually better than mutex for simple counters
```

### 4.5 Solutions refusées

```c
// ❌ REFUSÉ: Ne pas vérifier les paramètres NULL
clone_t *shadow_clone_jutsu_bad(ninja_t *ninja, clone_attr_t *attr,
                                 void *(*jutsu)(void*), void *chakra) {
    // MANQUE: if (ninja == NULL || jutsu == NULL) return NULL;
    clone_t *clone = &ninja->clones[ninja->clone_count];
    // ...
}
// Pourquoi: Segfault garanti si ninja est NULL

// ❌ REFUSÉ: Ne pas gérer la limite de clones
clone_t *shadow_clone_jutsu_bad2(ninja_t *ninja, clone_attr_t *attr,
                                  void *(*jutsu)(void*), void *chakra) {
    // MANQUE: if (ninja->clone_count >= ninja->max_clones) return NULL;
    clone_t *clone = &ninja->clones[ninja->clone_count];
    // Buffer overflow si trop de clones!
}

// ❌ REFUSÉ: Double join
int dispel_clone_bad(ninja_t *ninja, clone_t *clone, void **experience) {
    // MANQUE: if (clone->dispelled) return EINVAL;
    return pthread_join(clone->spirit, experience);
    // Undefined behavior sur double join!
}
```

### 4.6 Solution bonus de référence

```c
// sage_mode.c — Work-stealing implementation
#define _GNU_SOURCE
#include "kage_bunshin.h"
#include <sched.h>
#include <stdatomic.h>

// Lock-free work-stealing deque (simplified)
typedef struct {
    void **tasks;
    atomic_size_t head;
    atomic_size_t tail;
    size_t capacity;
} steal_deque_t;

steal_deque_t *create_steal_deque(size_t capacity) {
    steal_deque_t *dq = calloc(1, sizeof(steal_deque_t));
    if (dq == NULL) return NULL;

    dq->tasks = calloc(capacity, sizeof(void*));
    if (dq->tasks == NULL) {
        free(dq);
        return NULL;
    }

    dq->capacity = capacity;
    atomic_init(&dq->head, 0);
    atomic_init(&dq->tail, 0);

    return dq;
}

// Push to bottom (owner only)
bool push_task(steal_deque_t *dq, void *task) {
    size_t tail = atomic_load(&dq->tail);
    size_t head = atomic_load(&dq->head);

    if (tail - head >= dq->capacity)
        return false;  // Full

    dq->tasks[tail % dq->capacity] = task;
    atomic_store(&dq->tail, tail + 1);
    return true;
}

// Pop from bottom (owner only)
void *pop_task(steal_deque_t *dq) {
    size_t tail = atomic_load(&dq->tail);
    if (tail == 0) return NULL;

    tail--;
    atomic_store(&dq->tail, tail);

    size_t head = atomic_load(&dq->head);
    if (head <= tail) {
        return dq->tasks[tail % dq->capacity];
    }

    // Race with stealer
    if (head == tail) {
        if (atomic_compare_exchange_strong(&dq->head, &head, head + 1)) {
            atomic_store(&dq->tail, tail + 1);
            return dq->tasks[tail % dq->capacity];
        }
    }

    atomic_store(&dq->tail, tail + 1);
    return NULL;
}

// Steal from top (thieves)
void *steal_task(steal_deque_t *dq) {
    size_t head = atomic_load(&dq->head);
    size_t tail = atomic_load(&dq->tail);

    if (head >= tail)
        return NULL;  // Empty

    void *task = dq->tasks[head % dq->capacity];

    if (atomic_compare_exchange_strong(&dq->head, &head, head + 1)) {
        return task;
    }

    return NULL;  // Lost race
}

// CPU affinity
int bind_clone_to_battlefield(clone_t *clone, int cpu_id) {
    if (clone == NULL)
        return EINVAL;

    cpu_set_t cpuset;
    CPU_ZERO(&cpuset);
    CPU_SET(cpu_id, &cpuset);

    return pthread_setaffinity_np(clone->spirit, sizeof(cpu_set_t), &cpuset);
}

// Priority inheritance
int enable_priority_inheritance(clone_attr_t *attr) {
    if (attr == NULL)
        return EINVAL;

    pthread_mutexattr_t mutex_attr;
    pthread_mutexattr_init(&mutex_attr);
    pthread_mutexattr_setprotocol(&mutex_attr, PTHREAD_PRIO_INHERIT);
    pthread_mutexattr_destroy(&mutex_attr);

    return 0;
}
```

### 4.9 spec.json

```json
{
  "name": "shadow_clone_jutsu",
  "language": "c",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse (2.4.1-2.4.4)",
  "tags": ["threading", "posix", "tls", "concurrency", "phase2"],
  "passing_score": 70,

  "function": {
    "name": "shadow_clone_jutsu",
    "prototype": "clone_t *shadow_clone_jutsu(ninja_t *ninja, clone_attr_t *attr, void *(*jutsu)(void*), void *chakra)",
    "return_type": "clone_t *",
    "parameters": [
      {"name": "ninja", "type": "ninja_t *"},
      {"name": "attr", "type": "clone_attr_t *"},
      {"name": "jutsu", "type": "void *(*)(void*)"},
      {"name": "chakra", "type": "void *"}
    ]
  },

  "driver": {
    "reference": "clone_t *ref_shadow_clone_jutsu(ninja_t *ninja, clone_attr_t *attr, void *(*jutsu)(void*), void *chakra) { if (ninja == NULL || jutsu == NULL) return NULL; if (ninja->clone_count >= ninja->max_clones) return NULL; clone_t *clone = &ninja->clones[ninja->clone_count]; memset(clone, 0, sizeof(clone_t)); clone->clone_number = (int)(ninja->total_summoned + 1); clone->jutsu = jutsu; clone->chakra_data = chakra; clone->joinable = true; pthread_attr_t *pattr = (attr != NULL) ? &attr->attr : NULL; int ret = pthread_create(&clone->spirit, pattr, jutsu, chakra); if (ret != 0) return NULL; clone->active = true; ninja->clone_count++; ninja->total_summoned++; return clone; }",

    "edge_cases": [
      {
        "name": "null_ninja",
        "args": [null, null, "valid_func", null],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "ninja est NULL, doit retourner NULL"
      },
      {
        "name": "null_jutsu",
        "args": ["valid_ninja", null, null, null],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "jutsu est NULL, doit retourner NULL"
      },
      {
        "name": "max_clones_reached",
        "args": ["full_ninja", null, "valid_func", null],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "Chakra épuisé (max clones atteint)"
      },
      {
        "name": "valid_creation",
        "args": ["valid_ninja", null, "valid_func", "data"],
        "expected": "non_null"
      },
      {
        "name": "with_attributes",
        "args": ["valid_ninja", "valid_attr", "valid_func", "data"],
        "expected": "non_null"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {"min": 0, "max": 100}
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["pthread_create", "pthread_join", "pthread_detach", "pthread_exit", "pthread_self", "pthread_equal", "pthread_attr_init", "pthread_attr_destroy", "pthread_attr_setdetachstate", "pthread_attr_setstacksize", "pthread_attr_setstack", "pthread_attr_setguardsize", "pthread_attr_setschedpolicy", "pthread_attr_setschedparam", "pthread_attr_setscope", "pthread_attr_getdetachstate", "pthread_attr_getstacksize", "pthread_attr_getguardsize", "pthread_key_create", "pthread_key_delete", "pthread_getspecific", "pthread_setspecific", "malloc", "free", "calloc", "realloc", "printf", "fprintf", "sprintf", "snprintf", "clock_gettime", "gettimeofday", "memset", "memcpy", "strncpy"],
    "forbidden_functions": ["fork", "exec", "execl", "execv", "execle", "execve", "execlp", "execvp", "system", "signal", "sigaction"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```c
/* Mutant A (Boundary) : Ne vérifie pas la limite de clones */
clone_t *shadow_clone_jutsu_mutant_a(ninja_t *ninja, clone_attr_t *attr,
                                      void *(*jutsu)(void*), void *chakra) {
    if (ninja == NULL || jutsu == NULL)
        return NULL;
    // MANQUE: if (ninja->clone_count >= ninja->max_clones) return NULL;
    clone_t *clone = &ninja->clones[ninja->clone_count];  // Buffer overflow!
    // ...
}
// Pourquoi c'est faux: Buffer overflow quand max_clones atteint
// Ce qui était pensé: "Le tableau est assez grand"

/* Mutant B (Safety) : Ne vérifie pas le retour de pthread_create */
clone_t *shadow_clone_jutsu_mutant_b(ninja_t *ninja, clone_attr_t *attr,
                                      void *(*jutsu)(void*), void *chakra) {
    if (ninja == NULL || jutsu == NULL)
        return NULL;
    if (ninja->clone_count >= ninja->max_clones)
        return NULL;

    clone_t *clone = &ninja->clones[ninja->clone_count];
    pthread_create(&clone->spirit, NULL, jutsu, chakra);  // Ignore return!
    clone->active = true;  // Peut être faux si create a échoué!
    ninja->clone_count++;
    return clone;
}
// Pourquoi c'est faux: Clone "actif" mais thread jamais créé
// Ce qui était pensé: "pthread_create ne peut pas échouer"

/* Mutant C (Resource) : Ne marque pas clone comme dispelled */
int dispel_clone_mutant_c(ninja_t *ninja, clone_t *clone, void **experience) {
    if (ninja == NULL || clone == NULL)
        return EINVAL;
    if (!clone->joinable)
        return EINVAL;
    // MANQUE: if (clone->dispelled) return EINVAL;

    int ret = pthread_join(clone->spirit, experience);
    // MANQUE: clone->dispelled = true;
    return ret;
}
// Pourquoi c'est faux: Permet double-join (undefined behavior)
// Ce qui était pensé: "Je ne vais join qu'une fois de toute façon"

/* Mutant D (Logic) : Confond JOINABLE et DETACHED */
clone_t *shadow_clone_jutsu_mutant_d(ninja_t *ninja, clone_attr_t *attr,
                                      void *(*jutsu)(void*), void *chakra) {
    if (ninja == NULL || jutsu == NULL)
        return NULL;

    clone_t *clone = &ninja->clones[ninja->clone_count];
    clone->joinable = false;  // ERREUR: devrait être true par défaut!

    if (attr != NULL) {
        int detach_state;
        pthread_attr_getdetachstate(&attr->attr, &detach_state);
        clone->joinable = (detach_state == PTHREAD_CREATE_DETACHED);  // INVERSÉ!
    }
    // ...
}
// Pourquoi c'est faux: Logique inversée, DETACHED devient joinable
// Ce qui était pensé: Confusion entre les constantes

/* Mutant E (Return) : Retourne le clone même si création échoue */
clone_t *shadow_clone_jutsu_mutant_e(ninja_t *ninja, clone_attr_t *attr,
                                      void *(*jutsu)(void*), void *chakra) {
    if (ninja == NULL || jutsu == NULL)
        return NULL;

    clone_t *clone = &ninja->clones[ninja->clone_count];
    int ret = pthread_create(&clone->spirit, NULL, jutsu, chakra);

    // MANQUE: if (ret != 0) return NULL;

    clone->active = true;
    ninja->clone_count++;
    return clone;  // Retourne clone même si pthread_create a échoué!
}
// Pourquoi c'est faux: Clone invalide retourné
// Ce qui était pensé: "J'ai initialisé la structure, c'est bon"
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Naruto Analogy | Technical Reality |
|---------|----------------|-------------------|
| **Thread** | Shadow Clone | Lightweight execution unit |
| **Process vs Thread** | Naruto vs Clone | Address space sharing |
| **Shared memory** | Chakra partagé | Heap, code, data |
| **Private memory** | Expériences du clone | Stack, registers |
| **pthread_create** | Kage Bunshin Jutsu | Spawn thread |
| **pthread_join** | Dispel clone | Wait and get result |
| **pthread_detach** | Bunshin Daibakuha | Fire-and-forget |
| **TLS** | Clone's private memory | Per-thread storage |
| **Thread attributes** | Clone power settings | Stack, priority |

### 5.2 LDA — Traduction Littérale

```
FONCTION shadow_clone_jutsu QUI RETOURNE UN POINTEUR VERS clone_t ET PREND EN PARAMÈTRES ninja QUI EST UN POINTEUR VERS ninja_t ET attr QUI EST UN POINTEUR VERS clone_attr_t ET jutsu QUI EST UN POINTEUR VERS UNE FONCTION ET chakra QUI EST UN POINTEUR VOID
DÉBUT FONCTION
    SI ninja EST ÉGAL À NUL OU jutsu EST ÉGAL À NUL ALORS
        RETOURNER NUL
    FIN SI

    SI LE NOMBRE DE CLONES DU NINJA EST SUPÉRIEUR OU ÉGAL AU MAXIMUM ALORS
        RETOURNER NUL
    FIN SI

    DÉCLARER clone COMME POINTEUR VERS clone_t
    AFFECTER L'ADRESSE DU CLONE À L'INDEX clone_count DANS LE TABLEAU clones À clone

    AFFECTER jutsu AU CHAMP jutsu DE clone
    AFFECTER chakra AU CHAMP chakra_data DE clone
    AFFECTER VRAI AU CHAMP joinable DE clone

    DÉCLARER ret COMME ENTIER
    AFFECTER LE RÉSULTAT DE pthread_create À ret

    SI ret EST DIFFÉRENT DE 0 ALORS
        RETOURNER NUL
    FIN SI

    AFFECTER VRAI AU CHAMP active DE clone
    INCRÉMENTER clone_count DE 1
    INCRÉMENTER total_summoned DE 1

    RETOURNER clone
FIN FONCTION
```

### 5.2.2 Logic Flow

```
ALGORITHME : Shadow Clone Jutsu
---
1. VÉRIFIER les paramètres (ninja, jutsu non NULL)
   |-- Si invalide : RETOURNER NULL

2. VÉRIFIER la limite de chakra (max_clones)
   |-- Si dépassée : RETOURNER NULL (chakra épuisé)

3. PRÉPARER le clone :
   a. Obtenir un slot dans le tableau clones
   b. Initialiser les champs (jutsu, chakra_data, joinable)
   c. Définir les attributs si fournis

4. INVOQUER pthread_create :
   |-- Si échec : RETOURNER NULL
   |-- Si succès : Marquer clone comme actif

5. METTRE À JOUR les statistiques
   |-- Incrémenter clone_count
   |-- Incrémenter total_summoned

6. RETOURNER le clone créé
```

### 5.3 Visualisation ASCII

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     KAGE BUNSHIN NO JUTSU (THREAD MODEL)                    │
└─────────────────────────────────────────────────────────────────────────────┘

                    PROCESS (Naruto)
    ┌─────────────────────────────────────────────────────────────┐
    │                                                             │
    │   SHARED MEMORY (Chakra partagé)                            │
    │   ┌──────────────────────────────────────────────────────┐  │
    │   │  Code (Techniques)  │  Data  │  Heap (Chakra Pool)   │  │
    │   └──────────────────────────────────────────────────────┘  │
    │                    ↑           ↑           ↑                │
    │                    │           │           │                │
    │   ┌────────────────┼───────────┼───────────┼────────────┐   │
    │   │                │           │           │            │   │
    │   │   Thread 0     │  Thread 1 │  Thread 2 │  Thread 3  │   │
    │   │   (Original)   │  (Clone)  │  (Clone)  │  (Clone)   │   │
    │   │   ┌─────────┐  │ ┌───────┐ │ ┌───────┐ │ ┌───────┐  │   │
    │   │   │ Stack   │  │ │ Stack │ │ │ Stack │ │ │ Stack │  │   │
    │   │   │(Private)│  │ │(Priv) │ │ │(Priv) │ │ │(Priv) │  │   │
    │   │   │ ────────│  │ │───────│ │ │───────│ │ │───────│  │   │
    │   │   │ TLS     │  │ │ TLS   │ │ │ TLS   │ │ │ TLS   │  │   │
    │   │   │(Memory) │  │ │(Memory│ │ │(Memory│ │ │(Memory│  │   │
    │   │   └─────────┘  │ └───────┘ │ └───────┘ │ └───────┘  │   │
    │   └────────────────┴───────────┴───────────┴────────────┘   │
    │                                                             │
    └─────────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────────┐
    │  PTHREAD_CREATE FLOW (Kage Bunshin)                         │
    │                                                             │
    │  Original ──┬── pthread_create() ──→ Clone 1                │
    │             ├── pthread_create() ──→ Clone 2                │
    │             └── pthread_create() ──→ Clone 3                │
    │                                                             │
    │  [All clones execute jutsu(chakra) in parallel]             │
    │                                                             │
    │  Original ──┬── pthread_join(clone1) ←── experience 1       │
    │             ├── pthread_join(clone2) ←── experience 2       │
    │             └── pthread_join(clone3) ←── experience 3       │
    └─────────────────────────────────────────────────────────────┘
```

### 5.4 Les pièges en détail

#### Piège 1: Ne pas vérifier le retour de pthread_create

```c
// ❌ DANGEREUX
pthread_create(&thread, NULL, func, arg);
// Si ça échoue, thread est invalide mais on continue!

// ✅ CORRECT
int ret = pthread_create(&thread, NULL, func, arg);
if (ret != 0) {
    fprintf(stderr, "Failed to create thread: %s\n", strerror(ret));
    return NULL;
}
```

#### Piège 2: Double join

```c
// ❌ UNDEFINED BEHAVIOR
pthread_join(thread, &result1);
pthread_join(thread, &result2);  // BOOM!

// ✅ CORRECT
if (!clone->dispelled) {
    pthread_join(clone->spirit, &result);
    clone->dispelled = true;
}
```

#### Piège 3: Join après detach

```c
// ❌ UNDEFINED BEHAVIOR
pthread_detach(thread);
pthread_join(thread, &result);  // Thread déjà libéré!

// ✅ CORRECT
if (clone->joinable) {
    pthread_join(clone->spirit, &result);
}
```

### 5.5 Cours Complet

#### Les Threads : Clones dans le même corps

Un **thread** (fil d'exécution) est une unité d'exécution légère au sein d'un processus. Contrairement aux processus qui ont leur propre espace mémoire, les threads d'un même processus partagent :
- Le **code** (segment text)
- Les **données globales** (segment data)
- Le **tas** (heap)
- Les **fichiers ouverts**

Mais chaque thread possède :
- Sa propre **pile** (stack)
- Ses propres **registres**
- Son propre **Thread ID** (TID)
- Son propre **Thread-Local Storage** (TLS)

#### POSIX Threads (pthreads)

L'API POSIX threads est le standard pour la programmation multi-thread sur les systèmes Unix/Linux.

```c
#include <pthread.h>

// Créer un thread
int pthread_create(pthread_t *thread,               // ID du thread créé
                   const pthread_attr_t *attr,      // Attributs (NULL = défaut)
                   void *(*start_routine)(void*),   // Fonction à exécuter
                   void *arg);                      // Argument passé

// Attendre la fin d'un thread
int pthread_join(pthread_t thread,      // Thread à attendre
                 void **retval);        // Valeur retournée par le thread

// Détacher un thread (pas besoin de join)
int pthread_detach(pthread_t thread);

// Terminer le thread courant
void pthread_exit(void *retval);
```

#### Thread-Local Storage (TLS)

Le TLS permet à chaque thread d'avoir sa propre copie d'une variable :

```c
// Méthode 1: __thread (GCC)
__thread int my_var = 0;  // Chaque thread a sa propre copie

// Méthode 2: thread_local (C11)
thread_local int my_var = 0;

// Méthode 3: pthread_key (API POSIX)
pthread_key_t key;
pthread_key_create(&key, destructor_func);
pthread_setspecific(key, value);
void *val = pthread_getspecific(key);
```

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                    │
├─────────────────────────────────────────────────────────────────┤
│ pthread_create(&t, NULL, func, arg);  // Ignore return          │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ int ret = pthread_create(&t, NULL, func, arg);                  │
│ if (ret != 0) {                                                 │
│     // Handle error                                             │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ pthread_create peut échouer (limite ressources, permissions).   │
│ Ignorer l'erreur = thread zombie ou crash silencieux.           │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Trace d'exécution

```
┌───────┬──────────────────────────────────────────────┬──────────────────┬─────────────────┐
│ Étape │ Instruction                                  │ État             │ Explication     │
├───────┼──────────────────────────────────────────────┼──────────────────┼─────────────────┤
│   1   │ ninja = become_ninja("Naruto")               │ ninja créé       │ Init manager    │
├───────┼──────────────────────────────────────────────┼──────────────────┼─────────────────┤
│   2   │ clone = shadow_clone_jutsu(ninja,...)        │ clone_count=1    │ pthread_create  │
├───────┼──────────────────────────────────────────────┼──────────────────┼─────────────────┤
│   3   │ (Clone execute jutsu en parallèle)           │ 2 threads actifs │ Concurrent exec │
├───────┼──────────────────────────────────────────────┼──────────────────┼─────────────────┤
│   4   │ dispel_clone(ninja, clone, &exp)             │ Main attend      │ pthread_join    │
├───────┼──────────────────────────────────────────────┼──────────────────┼─────────────────┤
│   5   │ (Clone termine, retourne experience)         │ Clone terminé    │ Return value    │
├───────┼──────────────────────────────────────────────┼──────────────────┼─────────────────┤
│   6   │ retire_ninja(ninja)                          │ Tout libéré      │ Cleanup         │
└───────┴──────────────────────────────────────────────┴──────────────────┴─────────────────┘
```

### 5.8 Mnémotechniques

#### 🍥 MEME : "KAGE BUNSHIN NO JUTSU!"

![Naruto Shadow Clones](naruto_shadow_clones.jpg)

Quand Naruto crie "KAGE BUNSHIN NO JUTSU!", il crée des centaines de clones instantanément. C'est exactement ce que fait `pthread_create` !

```c
// 🍥 KAGE BUNSHIN NO JUTSU!
for (int i = 0; i < 1000; i++) {
    pthread_create(&threads[i], NULL, rasengan, NULL);
}
// 1000 clones créés!
```

**Rappel :** Comme Naruto qui épuise son chakra en créant trop de clones, ton système a une limite de threads. Vérifie toujours le retour !

#### 🌀 MEME : "Experience Transfer" — pthread_join

Dans Naruto, quand un clone disparaît, **toute son expérience revient à l'original**. C'est exactement ce que fait `pthread_join` !

```c
void *experience;
pthread_join(clone, &experience);
// L'expérience du clone est maintenant dans experience!
```

#### 💥 MEME : "Bunshin Daibakuha" — pthread_detach

Le **Bunshin Daibakuha** (Clone Grande Explosion) est une technique où le clone explose sans transférer son expérience. C'est `pthread_detach` !

```c
pthread_detach(thread);  // Clone explosif, pas besoin de join
// Attention: on ne récupère JAMAIS l'expérience!
```

### 5.9 Applications pratiques

1. **Web Server** : Chaque requête HTTP = un clone
2. **Game Engine** : Thread rendu, thread physique, thread audio
3. **Video Encoding** : Chaque frame encodée par un clone différent
4. **Database** : Thread par connexion client

---

## ⚠️ SECTION 6 : PIÈGES RÉCAPITULATIF

| Piège | Description | Solution |
|-------|-------------|----------|
| NULL parameter | Passer NULL comme ninja ou jutsu | Vérifier au début |
| Max clones | Dépasser la limite de threads | Check avant create |
| pthread_create fail | Ignorer le code retour | Toujours vérifier ret |
| Double join | Joindre deux fois le même thread | Flag dispelled |
| Join after detach | Joindre un thread détaché | Check joinable flag |
| Memory leak | Oublier de free le retour | Destructor TLS |
| Race condition | Accès concurrent aux stats | Atomics ou mutex |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quelle est la différence principale entre un processus et un thread ?**

A) Un thread ne peut pas exécuter de code
B) Les threads partagent le même espace d'adressage
C) Un processus est plus léger qu'un thread
D) Les threads ne peuvent pas communiquer entre eux
E) Un thread a son propre heap
F) Les processus partagent leur stack
G) Un thread ne peut pas avoir de TLS
H) Les threads sont toujours plus lents
I) Un processus n'a qu'un seul thread
J) Les threads ont des PID différents

**Réponse : B**

### Question 2
**Que retourne pthread_create en cas de succès ?**

A) Le TID du nouveau thread
B) 1
C) Le pointeur du thread
D) -1
E) 0
F) NULL
G) EINVAL
H) Le PID
I) true
J) Le thread lui-même

**Réponse : E**

### Question 3
**Quel est l'effet de pthread_detach ?**

A) Tue le thread immédiatement
B) Le thread ne peut plus être join
C) Le thread devient prioritaire
D) Le thread perd son stack
E) Le thread devient un processus
F) Le thread ne peut plus utiliser TLS
G) Le thread est suspendu
H) Le thread devient root
I) Le thread perd son TID
J) Le thread ne peut plus malloc

**Réponse : B**

### Question 4
**Comment chaque thread peut avoir sa propre copie d'une variable ?**

A) Variables globales
B) Variables statiques
C) Thread-Local Storage (__thread)
D) Variables const
E) Variables extern
F) Variables register
G) Variables volatile
H) Variables inline
I) Variables auto
J) Variables restrict

**Réponse : C**

### Question 5
**Que se passe-t-il si on appelle pthread_join deux fois sur le même thread ?**

A) La deuxième attente est ignorée
B) Le comportement est défini (retourne une erreur)
C) Undefined behavior
D) Le thread est relancé
E) Le programme attend indéfiniment
F) Le thread est dupliqué
G) Une exception est levée
H) Le thread devient zombie
I) Le système redémarre
J) Rien de spécial

**Réponse : C**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Critère | Valeur |
|---------|--------|
| **Exercice** | 2.4.0 - shadow_clone_jutsu |
| **Concepts** | 41 (2.4.1 à 2.4.4) |
| **Difficulté** | ★★★★★★☆☆☆☆ (6/10) |
| **Temps** | 6h |
| **XP** | 500 (base) / 1500 (bonus) |
| **Langage** | C17 |
| **Thème** | Naruto - Shadow Clone Jutsu |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.4.0-shadow-clone-jutsu",
    "generated_at": "2025-01-12 17:00:00",

    "metadata": {
      "exercise_id": "2.4.0",
      "exercise_name": "shadow_clone_jutsu",
      "module": "2.4.0",
      "module_name": "Thread Fundamentals",
      "concept": "a-k",
      "concept_name": "Thread Concepts + POSIX + Attributes + TLS",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "c",
      "duration_minutes": 360,
      "xp_base": 500,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T3 O(n)",
      "complexity_space": "S3 O(n)",
      "prerequisites": ["Module 2.2", "Pointeurs", "Allocation mémoire"],
      "domains": ["Process", "Mem", "Struct"],
      "domains_bonus": ["CPU"],
      "tags": ["threading", "posix", "pthread", "tls", "concurrency"],
      "meme_reference": "Naruto Shadow Clone Jutsu"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_kage_bunshin.c": "/* Section 4.3 */",
      "references/ref_sage_mode.c": "/* Section 4.6 */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 */",
      "mutants/mutant_e_return.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_kage_bunshin.c",
        "references/ref_sage_mode.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c"
      ]
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "KAGE BUNSHIN NO JUTSU!"*
*L'excellence pédagogique ne se négocie pas — pas de raccourcis*
