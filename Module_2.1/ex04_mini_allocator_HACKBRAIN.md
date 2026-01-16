<thinking>
## Analyse du Concept
- Concept : Implémentation de malloc/free avec différentes stratégies d'allocation
- Phase demandée : 2
- Adapté ? OUI — C'est un projet classique de systèmes, adapté pour comprendre la gestion mémoire

## Combo Base + Bonus
- Exercice de base : malloc/free/realloc/calloc avec 4 stratégies (first-fit, best-fit, worst-fit, next-fit)
- Bonus : Buddy system + segregated lists + détection de corruption
- Palier bonus : 💀 Expert (algorithmes avancés d'allocation)
- Progression logique ? OUI — Base = stratégies classiques, Bonus = stratégies optimisées

## Prérequis & Difficulté
- Prérequis réels : Pointeurs, arithmétique de pointeurs, structures auto-référentielles, alignement mémoire, syscalls (sbrk/mmap)
- Difficulté estimée : 6/10 (base), 8/10 (bonus)
- Cohérent avec phase ? OUI — Phase 2 = 4-6/10, exercice difficile mais accessible

## Aspect Fun/Culture
- Contexte choisi : Minecraft — Gestion de blocs dans un monde 3D
- MEME mnémotechnique : "Mining memory" = allocation, "Creeper explosion" = fragmentation
- Pourquoi c'est fun : Les blocks de Minecraft = blocks mémoire, le crafting = coalescing

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : Mauvais calcul d'alignement (aligné sur 8 au lieu de 16)
2. Mutant B (Safety) : Pas de vérification du magic number → corruption non détectée
3. Mutant C (Resource) : Coalescing qui ne fusionne pas avec le bloc précédent
4. Mutant D (Logic) : First-fit qui retourne le dernier bloc au lieu du premier
5. Mutant E (Return) : malloc(0) qui retourne un bloc au lieu de NULL

## Verdict
VALIDE — Exercice synthèse couvrant 12+ concepts du curriculum (2.1.6-2.1.10)
</thinking>

---

# Exercice 2.1.4 : minecraft_malloc

**Module :**
2.1.4 — Heap Management & Custom Allocator

**Concept :**
a-i — malloc/free, block headers, allocation strategies, coalescing

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
code

**Tiers :**
3 — Synthèse (allocation + stratégies + coalescing + statistiques)

**Langage :**
C17

**Prérequis :**
- Pointeurs et arithmétique de pointeurs
- Structures auto-référentielles
- Alignement mémoire
- Syscalls (sbrk/mmap) - ex01, ex02, ex03

**Domaines :**
Mem, Struct, Algo

**Durée estimée :**
480-720 min (8-12 heures)

**XP Base :**
750

**Complexité :**
T3 O(n) recherche × S2 O(1) par allocation + headers

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex04_mini_allocator/
├── my_malloc.h
├── my_malloc.c
├── block_utils.c
├── strategies.c
├── coalesce.c
├── stats.c
└── Makefile
```

**Fonctions autorisées :**
- `sbrk` (pour BACKEND_SBRK)
- `mmap`, `munmap` (pour BACKEND_MMAP)
- `memset`, `memcpy`, `memmove`
- `write` (pour debug output uniquement)

**Fonctions interdites :**
- `malloc`, `free`, `calloc`, `realloc` (évidemment !)
- `printf`, `fprintf` (utiliser write)

---

### 1.2 Consigne

#### 🎮 Version Culture Pop : "MINECRAFT: Memory Crafting"

**"Mine, Craft, Allocate!"**

Tu es Steve, le légendaire mineur de Minecraft. Ton inventaire est comme un heap : limité, fragmenté, et tu dois gérer chaque slot avec précision.

**Ton monde mémoire :**

| Concept Minecraft | Concept Mémoire |
|-------------------|-----------------|
| 🧱 Bloc de pierre | Block mémoire alloué |
| 💨 Air | Espace libre |
| ⛏️ Miner | `malloc()` - creuser pour obtenir de l'espace |
| 💥 Creeper | Fragmentation - explosion qui laisse des trous |
| 🔨 Crafting Table | Coalescing - fusionner des blocs adjacents |
| 📦 Chest | Block header - métadonnées |
| 🗺️ Chunk | Region mémoire obtenue du système |

**Les règles du Nether (contraintes) :**

```
┌─────────────────────────────────────────────────────────────────┐
│  ALIGNEMENT : Chaque bloc retourné aligné sur 16 bytes         │
│  OVERHEAD MAX : 32 bytes par allocation (header)               │
│  THREAD-SAFETY : Non requis (single-player mode)               │
│  MAGIC NUMBER : 0xDEADBEEF pour détecter la corruption         │
└─────────────────────────────────────────────────────────────────┘
```

---

#### 📚 Version Académique : Implémentation d'Allocateur Mémoire

**Contexte technique :**

`malloc()` et `free()` sont les fonctions les plus utilisées en C, mais leur implémentation est souvent mal comprise. Derrière ces simples appels se cache une machinerie complexe :

**malloc doit :**
1. Gérer un pool de mémoire obtenu du système (via sbrk ou mmap)
2. Trouver un bloc libre assez grand
3. Potentiellement découper un bloc
4. Retourner un pointeur aligné

**free doit :**
1. Marquer le bloc comme libre
2. Potentiellement fusionner avec les voisins libres
3. Potentiellement rendre la mémoire au système

**Le défi intellectuel :** Il n'y a pas une seule "bonne" implémentation. Chaque choix (structure de données, stratégie de recherche, politique de coalescing) a des implications sur les performances et la fragmentation.

---

**Ta mission :**

Implémenter une bibliothèque d'allocation mémoire compatible avec l'interface standard. L'allocateur doit être interchangeable avec l'allocateur système.

**Entrées :**
- `size` : Taille demandée par l'utilisateur

**Sorties :**
- Pointeur vers zone mémoire alignée ou NULL

**Contraintes :**
- Alignement sur 16 bytes obligatoire
- Support de 4 stratégies de recherche
- Coalescing automatique lors de free()
- Statistiques accessibles via API

---

### 1.3 Prototypes

```c
/* ═══════════════════════════════════════════════════════════════════════════
 * INTERFACE STANDARD
 * ═══════════════════════════════════════════════════════════════════════════ */

/* Allocation de size bytes */
void *my_malloc(size_t size);

/* Libération du bloc pointé par ptr */
void my_free(void *ptr);

/* Réallocation : change la taille, préserve les données */
void *my_realloc(void *ptr, size_t size);

/* Allocation initialisée à zéro */
void *my_calloc(size_t nmemb, size_t size);

/* ═══════════════════════════════════════════════════════════════════════════
 * STRUCTURE INTERNE
 * ═══════════════════════════════════════════════════════════════════════════ */

/* Magic number pour détection de corruption */
#define BLOCK_MAGIC 0xDEADBEEF

/* Header de chaque bloc (32 bytes avec alignement) */
typedef struct block_header {
    size_t size;                    /* Taille du payload */
    int    free;                    /* 1 si libre, 0 si alloué */
    struct block_header *next;      /* Prochain bloc (free list) */
    struct block_header *prev;      /* Bloc précédent (pour coalescing) */
    uint32_t magic;                 /* Pour détection corruption */
    uint32_t _padding;              /* Alignement sur 32 bytes */
} block_header_t;

/* Le payload suit immédiatement le header */
/* Alignement : le pointeur retourné doit être aligné sur 16 bytes */

/* ═══════════════════════════════════════════════════════════════════════════
 * STRATÉGIES DE RECHERCHE
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef enum {
    STRATEGY_FIRST_FIT,    /* Premier bloc assez grand */
    STRATEGY_BEST_FIT,     /* Plus petit bloc assez grand */
    STRATEGY_WORST_FIT,    /* Plus grand bloc disponible */
    STRATEGY_NEXT_FIT      /* Continuer depuis dernière position */
} alloc_strategy_t;

void my_malloc_set_strategy(alloc_strategy_t strategy);
alloc_strategy_t my_malloc_get_strategy(void);

/* ═══════════════════════════════════════════════════════════════════════════
 * BACKEND MÉMOIRE
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef enum {
    BACKEND_SBRK,   /* Étend le segment de données (traditionnel) */
    BACKEND_MMAP    /* Allocation anonyme (moderne) */
} memory_backend_t;

void my_malloc_set_backend(memory_backend_t backend);
memory_backend_t my_malloc_get_backend(void);

/* ═══════════════════════════════════════════════════════════════════════════
 * STATISTIQUES ET DEBUG
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    size_t total_allocated;      /* Bytes alloués (payloads) */
    size_t total_free;           /* Bytes dans free list */
    size_t overhead;             /* Bytes utilisés par headers */
    size_t num_blocks;           /* Nombre total de blocs */
    size_t num_free_blocks;      /* Blocs libres */
    size_t largest_free_block;   /* Plus grand bloc libre */
    double fragmentation;        /* Ratio fragmentation externe */
} malloc_stats_t;

malloc_stats_t my_malloc_stats(void);

/* Afficher l'état du heap (debug) */
void my_malloc_dump(void);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Pourquoi l'alignement sur 16 bytes ?

Les processeurs modernes (x86-64, ARM) ont des instructions SIMD (SSE, AVX, NEON) qui requièrent des données alignées :
- SSE : 16 bytes
- AVX : 32 bytes
- AVX-512 : 64 bytes

Un malloc non aligné peut causer :
- **Crash** sur certaines architectures (SPARC strict)
- **Performances dégradées** (accès mémoire double)
- **Bugs subtils** avec les instructions atomiques

### 2.2 Le coût réel de malloc()

```
glibc malloc      : ~50-200 cycles (optimisé)
jemalloc          : ~30-100 cycles (Facebook)
tcmalloc          : ~30-80 cycles (Google)
Naive first-fit   : ~1000+ cycles (notre implémentation basique)
```

Les allocateurs modernes utilisent des techniques avancées :
- **Arenas** : pools séparés par thread (pas de locks)
- **Size classes** : listes séparées par taille
- **Thread-local caches** : cache par thread

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation | Cas d'usage |
|--------|-------------|-------------|
| **Game Developer** | Allocateurs custom | Unity/Unreal ont leurs propres allocateurs pour la prédictibilité |
| **Embedded Engineer** | Pools statiques | Systèmes sans heap dynamique (safety-critical) |
| **Database Developer** | Slab allocators | PostgreSQL utilise des pools par taille de structure |
| **Browser Developer** | Garbage collection | V8/SpiderMonkey ont des allocateurs spécialisés |
| **Kernel Developer** | kmalloc/vmalloc | Allocateurs kernel avec contraintes différentes |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
my_malloc.h  my_malloc.c  block_utils.c  strategies.c  coalesce.c  main.c  Makefile

$ make

$ ./test_allocator
=== Basic Tests ===
malloc(100): 0x55a8b5400010 (aligned: yes)
malloc(200): 0x55a8b54000a0 (aligned: yes)
free(ptr1): OK
realloc(ptr2, 500): 0x55a8b5400150 (data preserved: yes)

=== Strategy Comparison ===
First-fit: fragmentation=23.45%, overhead=8.12%
Best-fit: fragmentation=18.72%, overhead=8.12%
Worst-fit: fragmentation=31.56%, overhead=8.12%
Next-fit: fragmentation=25.89%, overhead=8.12%

=== Heap Dump ===
Block 0x55a8b5400000: size=100, status=ALLOCATED, magic=DEADBEEF
Block 0x55a8b5400088: size=200, status=FREE, magic=DEADBEEF
Block 0x55a8b5400168: size=50, status=ALLOCATED, magic=DEADBEEF
---
Total blocks: 3
Free blocks: 1
Largest free: 200 bytes
Fragmentation: 33.33%

All tests passed!
```

---

### 3.1 💀 BONUS EXPERT (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×4

**Time Complexity attendue :**
O(log n) avec buddy system

**Space Complexity attendue :**
O(1) overhead additionnel

**Domaines Bonus :**
`MD (arbres binaires), Algo`

#### 3.1.1 Consigne Bonus

**🎮 "Minecraft: The Ender Dragon Challenge"**

Tu as vaincu le Wither, mais l'Ender Dragon exige une gestion encore plus efficace. Le Buddy System : comme diviser et fusionner des blocs de Netherite.

**Ta mission :**

Implémenter le **Buddy System** et les **Segregated Lists**.

```c
/* ═══════════════════════════════════════════════════════════════════════════
 * BUDDY SYSTEM
 * Allocation par puissances de 2, fusion rapide des "buddies"
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    void  *base;              /* Base du pool buddy */
    size_t total_size;        /* Taille totale (puissance de 2) */
    size_t min_block_size;    /* Plus petit bloc (ex: 64 bytes) */
    void  **free_lists;       /* Une liste par niveau */
    size_t num_levels;        /* log2(total/min) + 1 */
} buddy_allocator_t;

buddy_allocator_t *buddy_create(size_t total_size, size_t min_block);
void buddy_destroy(buddy_allocator_t *buddy);
void *buddy_alloc(buddy_allocator_t *buddy, size_t size);
void buddy_free(buddy_allocator_t *buddy, void *ptr);

/* ═══════════════════════════════════════════════════════════════════════════
 * SEGREGATED LISTS
 * Une free list par classe de taille
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    size_t size_class;        /* Taille de cette classe */
    block_header_t *head;     /* Tête de la free list */
    size_t count;             /* Nombre de blocs libres */
} seg_list_t;

/* Classes de taille : 16, 32, 64, 128, 256, 512, 1024, 2048, 4096, large */
#define NUM_SIZE_CLASSES 10

void *segregated_malloc(size_t size);
void segregated_free(void *ptr);
```

**Contraintes :**
```
┌─────────────────────────────────────────────────────────────────┐
│  Buddy: taille arrondie à puissance de 2 supérieure            │
│  Buddy: fusion O(1) avec calcul XOR de l'adresse               │
│  Segregated: recherche O(1) pour petites tailles               │
│  Détecter double-free via magic number                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette — Tests automatisés

| Test | Description | Entrée | Attendu | Points |
|------|-------------|--------|---------|--------|
| `test_basic_malloc` | malloc simple | 100 | ptr != NULL, aligned | 5 |
| `test_basic_free` | free simple | valid ptr | no crash | 5 |
| `test_alignment` | Alignement 16 | 1-1000 | (ptr % 16) == 0 | 10 |
| `test_write_read` | Écrire/relire | int array | data preserved | 10 |
| `test_realloc_grow` | Réalloc plus grand | 10→100 | data preserved | 10 |
| `test_realloc_shrink` | Réalloc plus petit | 100→10 | data preserved | 5 |
| `test_calloc_zeroed` | calloc = zéros | 100 ints | all 0 | 10 |
| `test_coalesce` | Fusion blocs | free A,B,C | 1 bloc | 10 |
| `test_split` | Découpage bloc | 1000→100 | 2 blocs | 10 |
| `test_first_fit` | Premier bloc | setup | correct block | 5 |
| `test_best_fit` | Plus petit bloc | setup | correct block | 5 |
| `test_null_free` | free(NULL) | NULL | no crash | 5 |
| `test_stress` | 1000 alloc/free | random | no crash, valgrind OK | 10 |

---

### 4.2 main.c de test

```c
#include "my_malloc.h"
#include <assert.h>
#include <stdint.h>
#include <string.h>

static void test_basic_malloc_free(void)
{
    void *p = my_malloc(100);
    assert(p != NULL);
    assert(((uintptr_t)p % 16) == 0);  /* Alignement */
    my_free(p);
}

static void test_alignment(void)
{
    for (int i = 1; i <= 1000; i++)
    {
        void *p = my_malloc(i);
        assert(p != NULL);
        assert(((uintptr_t)p % 16) == 0);
        my_free(p);
    }
}

static void test_write_read(void)
{
    int *arr = my_malloc(100 * sizeof(int));
    assert(arr != NULL);

    for (int i = 0; i < 100; i++)
        arr[i] = i * i;

    for (int i = 0; i < 100; i++)
        assert(arr[i] == i * i);

    my_free(arr);
}

static void test_realloc_grow(void)
{
    int *p = my_malloc(10 * sizeof(int));
    for (int i = 0; i < 10; i++)
        p[i] = i;

    p = my_realloc(p, 100 * sizeof(int));
    assert(p != NULL);

    /* Données préservées */
    for (int i = 0; i < 10; i++)
        assert(p[i] == i);

    my_free(p);
}

static void test_calloc_zeroed(void)
{
    int *p = my_calloc(100, sizeof(int));
    assert(p != NULL);

    for (int i = 0; i < 100; i++)
        assert(p[i] == 0);

    my_free(p);
}

static void test_coalesce(void)
{
    void *a = my_malloc(100);
    void *b = my_malloc(100);
    void *c = my_malloc(100);

    my_free(a);
    my_free(c);
    my_free(b);  /* Devrait fusionner a+b+c */

    malloc_stats_t s = my_malloc_stats();
    assert(s.num_free_blocks == 1);  /* Un seul bloc après coalescing */
}

static void test_null_free(void)
{
    my_free(NULL);  /* Ne doit pas crasher */
}

int main(void)
{
    test_basic_malloc_free();
    test_alignment();
    test_write_read();
    test_realloc_grow();
    test_calloc_zeroed();
    test_coalesce();
    test_null_free();

    write(1, "All tests passed!\n", 18);
    return 0;
}
```

---

### 4.3 Solution de référence — my_malloc.c

```c
#include "my_malloc.h"
#include <unistd.h>
#include <string.h>
#include <stdint.h>

/* ═══════════════════════════════════════════════════════════════════════════
 * VARIABLES GLOBALES
 * ═══════════════════════════════════════════════════════════════════════════ */

static block_header_t *g_heap_start = NULL;
static block_header_t *g_last_block = NULL;
static block_header_t *g_next_fit_ptr = NULL;
static alloc_strategy_t g_strategy = STRATEGY_FIRST_FIT;
static memory_backend_t g_backend = BACKEND_SBRK;

#define ALIGNMENT 16
#define ALIGN(size) (((size) + (ALIGNMENT - 1)) & ~(ALIGNMENT - 1))
#define HEADER_SIZE ALIGN(sizeof(block_header_t))
#define MIN_BLOCK_SIZE 32

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS INTERNES
 * ═══════════════════════════════════════════════════════════════════════════ */

static block_header_t *get_header(void *ptr)
{
    return ((block_header_t *)((char *)ptr - HEADER_SIZE));
}

static void *get_payload(block_header_t *block)
{
    return ((char *)block + HEADER_SIZE);
}

static block_header_t *request_space(size_t size)
{
    block_header_t *block;

    if (g_backend == BACKEND_SBRK)
    {
        block = sbrk(0);
        void *request = sbrk(HEADER_SIZE + size);
        if (request == (void *)-1)
            return (NULL);
    }
    else
    {
        /* BACKEND_MMAP */
        block = mmap(NULL, HEADER_SIZE + size,
                     PROT_READ | PROT_WRITE,
                     MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        if (block == MAP_FAILED)
            return (NULL);
    }

    block->size = size;
    block->free = 0;
    block->next = NULL;
    block->prev = g_last_block;
    block->magic = BLOCK_MAGIC;

    if (g_last_block)
        g_last_block->next = block;
    g_last_block = block;

    return (block);
}

static block_header_t *find_first_fit(size_t size)
{
    block_header_t *current = g_heap_start;

    while (current)
    {
        if (current->free && current->size >= size)
            return (current);
        current = current->next;
    }
    return (NULL);
}

static block_header_t *find_best_fit(size_t size)
{
    block_header_t *current = g_heap_start;
    block_header_t *best = NULL;

    while (current)
    {
        if (current->free && current->size >= size)
        {
            if (!best || current->size < best->size)
                best = current;
        }
        current = current->next;
    }
    return (best);
}

static block_header_t *find_worst_fit(size_t size)
{
    block_header_t *current = g_heap_start;
    block_header_t *worst = NULL;

    while (current)
    {
        if (current->free && current->size >= size)
        {
            if (!worst || current->size > worst->size)
                worst = current;
        }
        current = current->next;
    }
    return (worst);
}

static block_header_t *find_next_fit(size_t size)
{
    if (!g_next_fit_ptr)
        g_next_fit_ptr = g_heap_start;

    block_header_t *current = g_next_fit_ptr;
    block_header_t *start = current;

    do {
        if (current && current->free && current->size >= size)
        {
            g_next_fit_ptr = current->next ? current->next : g_heap_start;
            return (current);
        }
        current = current ? current->next : g_heap_start;
    } while (current != start);

    return (NULL);
}

static block_header_t *find_block(size_t size)
{
    switch (g_strategy)
    {
        case STRATEGY_FIRST_FIT: return find_first_fit(size);
        case STRATEGY_BEST_FIT:  return find_best_fit(size);
        case STRATEGY_WORST_FIT: return find_worst_fit(size);
        case STRATEGY_NEXT_FIT:  return find_next_fit(size);
        default:                 return find_first_fit(size);
    }
}

static void split_block(block_header_t *block, size_t size)
{
    if (block->size >= size + HEADER_SIZE + MIN_BLOCK_SIZE)
    {
        block_header_t *new_block = (block_header_t *)
            ((char *)block + HEADER_SIZE + size);

        new_block->size = block->size - size - HEADER_SIZE;
        new_block->free = 1;
        new_block->next = block->next;
        new_block->prev = block;
        new_block->magic = BLOCK_MAGIC;

        if (block->next)
            block->next->prev = new_block;
        else
            g_last_block = new_block;

        block->next = new_block;
        block->size = size;
    }
}

static void coalesce(block_header_t *block)
{
    /* Fusionner avec le suivant */
    if (block->next && block->next->free)
    {
        block->size += HEADER_SIZE + block->next->size;
        block->next = block->next->next;
        if (block->next)
            block->next->prev = block;
        else
            g_last_block = block;
    }

    /* Fusionner avec le précédent */
    if (block->prev && block->prev->free)
    {
        block->prev->size += HEADER_SIZE + block->size;
        block->prev->next = block->next;
        if (block->next)
            block->next->prev = block->prev;
        else
            g_last_block = block->prev;
    }
}

/* ═══════════════════════════════════════════════════════════════════════════
 * INTERFACE PUBLIQUE
 * ═══════════════════════════════════════════════════════════════════════════ */

void *my_malloc(size_t size)
{
    if (size == 0)
        return (NULL);

    size = ALIGN(size);

    block_header_t *block;

    if (!g_heap_start)
    {
        block = request_space(size);
        if (!block)
            return (NULL);
        g_heap_start = block;
    }
    else
    {
        block = find_block(size);
        if (block)
        {
            block->free = 0;
            split_block(block, size);
        }
        else
        {
            block = request_space(size);
            if (!block)
                return (NULL);
        }
    }

    return (get_payload(block));
}

void my_free(void *ptr)
{
    if (!ptr)
        return;

    block_header_t *block = get_header(ptr);

    /* Vérification magic number */
    if (block->magic != BLOCK_MAGIC)
        return;  /* Corruption détectée, ignorer */

    block->free = 1;
    coalesce(block);
}

void *my_realloc(void *ptr, size_t size)
{
    if (!ptr)
        return my_malloc(size);

    if (size == 0)
    {
        my_free(ptr);
        return (NULL);
    }

    block_header_t *block = get_header(ptr);

    if (block->magic != BLOCK_MAGIC)
        return (NULL);

    size = ALIGN(size);

    /* Si le bloc actuel est assez grand */
    if (block->size >= size)
    {
        split_block(block, size);
        return (ptr);
    }

    /* Sinon, allouer nouveau bloc et copier */
    void *new_ptr = my_malloc(size);
    if (!new_ptr)
        return (NULL);

    memcpy(new_ptr, ptr, block->size);
    my_free(ptr);

    return (new_ptr);
}

void *my_calloc(size_t nmemb, size_t size)
{
    size_t total = nmemb * size;

    /* Overflow check */
    if (nmemb != 0 && total / nmemb != size)
        return (NULL);

    void *ptr = my_malloc(total);
    if (ptr)
        memset(ptr, 0, total);

    return (ptr);
}

void my_malloc_set_strategy(alloc_strategy_t strategy)
{
    g_strategy = strategy;
}

alloc_strategy_t my_malloc_get_strategy(void)
{
    return (g_strategy);
}

void my_malloc_set_backend(memory_backend_t backend)
{
    g_backend = backend;
}

memory_backend_t my_malloc_get_backend(void)
{
    return (g_backend);
}

malloc_stats_t my_malloc_stats(void)
{
    malloc_stats_t stats = {0};
    block_header_t *current = g_heap_start;

    while (current)
    {
        stats.num_blocks++;
        stats.overhead += HEADER_SIZE;

        if (current->free)
        {
            stats.num_free_blocks++;
            stats.total_free += current->size;
            if (current->size > stats.largest_free_block)
                stats.largest_free_block = current->size;
        }
        else
        {
            stats.total_allocated += current->size;
        }

        current = current->next;
    }

    if (stats.total_free > 0)
    {
        stats.fragmentation = 1.0 -
            ((double)stats.largest_free_block / stats.total_free);
    }

    return (stats);
}
```

---

### 4.5 Solutions refusées

#### ❌ Refusée 1 : Pas d'alignement

```c
void *my_malloc_bad(size_t size)
{
    /* ERREUR : Pas d'alignement ! */
    block_header_t *block = sbrk(sizeof(block_header_t) + size);
    return (block + 1);  /* Non aligné sur 16 bytes */
}
/* Pourquoi refusé : SIMD crash, performances dégradées */
```

#### ❌ Refusée 2 : Pas de coalescing

```c
void my_free_bad(void *ptr)
{
    block_header_t *block = get_header(ptr);
    block->free = 1;
    /* ERREUR : Pas de coalescing ! */
    /* La fragmentation va exploser */
}
/* Pourquoi refusé : Fragmentation externe non contrôlée */
```

---

### 4.9 spec.json

```json
{
  "name": "minecraft_malloc",
  "language": "c",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (malloc complet)",
  "tags": ["memory", "malloc", "allocator", "heap", "phase2"],
  "passing_score": 80,

  "function": {
    "name": "my_malloc",
    "prototype": "void *my_malloc(size_t size)",
    "return_type": "void *",
    "parameters": [
      {"name": "size", "type": "size_t"}
    ]
  },

  "driver": {
    "reference": "void *ref_my_malloc(size_t size) { if (size == 0) return NULL; /* simplified reference */ return sbrk(size); }",

    "edge_cases": [
      {
        "name": "size_zero",
        "args": [0],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "malloc(0) doit retourner NULL"
      },
      {
        "name": "alignment_check",
        "args": [1],
        "expected": "aligned_ptr",
        "is_trap": true,
        "trap_explanation": "Même 1 byte doit être aligné sur 16"
      },
      {
        "name": "large_allocation",
        "args": [1000000],
        "expected": "valid_ptr_or_null"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 1000,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {
            "min": 0,
            "max": 10000
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["sbrk", "mmap", "munmap", "memset", "memcpy", "memmove", "write"],
    "forbidden_functions": ["malloc", "free", "calloc", "realloc", "printf"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes

#### Mutant A (Boundary) : Alignement sur 8 au lieu de 16

```c
#define ALIGNMENT 8  /* ERREUR : Devrait être 16 */
#define ALIGN(size) (((size) + (ALIGNMENT - 1)) & ~(ALIGNMENT - 1))

void *my_malloc_mutant_a(size_t size)
{
    /* Code correct mais alignement insuffisant */
    size = ALIGN(size);  /* Aligné sur 8, pas 16 ! */
    /* ... */
}
/* Pourquoi faux : SSE/AVX peuvent crasher ou être lents */
/* Pensée erronée : "8 bytes suffit pour les int/double" */
```

#### Mutant B (Safety) : Pas de vérification magic

```c
void my_free_mutant_b(void *ptr)
{
    if (!ptr)
        return;

    block_header_t *block = get_header(ptr);
    /* MANQUANT : if (block->magic != BLOCK_MAGIC) return; */

    block->free = 1;
    coalesce(block);
}
/* Pourquoi faux : Corruption silencieuse, double-free non détecté */
```

#### Mutant C (Resource) : Coalescing partiel

```c
static void coalesce_mutant_c(block_header_t *block)
{
    /* Fusionne avec suivant */
    if (block->next && block->next->free)
    {
        block->size += HEADER_SIZE + block->next->size;
        block->next = block->next->next;
        if (block->next)
            block->next->prev = block;
    }

    /* MANQUANT : Fusion avec précédent ! */
}
/* Pourquoi faux : Fragmentation si free(A), free(B) dans cet ordre */
```

#### Mutant D (Logic) : First-fit inversé (last-fit)

```c
static block_header_t *find_first_fit_mutant_d(size_t size)
{
    block_header_t *current = g_heap_start;
    block_header_t *found = NULL;

    while (current)
    {
        if (current->free && current->size >= size)
            found = current;  /* Continue au lieu de return ! */
        current = current->next;
    }
    return (found);  /* Retourne le DERNIER, pas le premier */
}
/* Pourquoi faux : C'est last-fit, pas first-fit */
```

#### Mutant E (Return) : malloc(0) retourne bloc valide

```c
void *my_malloc_mutant_e(size_t size)
{
    /* MANQUANT : if (size == 0) return NULL; */
    size = ALIGN(size);  /* ALIGN(0) = 0, puis alloue quand même */
    /* ... allocation ... */
}
/* Pourquoi faux : Standards C disent malloc(0) → NULL ou ptr unique */
/* Ici ça gaspille de la mémoire inutilement */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Référence | Maîtrise attendue |
|---------|-----------|-------------------|
| Heap management | 2.1.6.a-b | Comprendre le tas et la free list |
| Block header | 2.1.6.c | Structure de métadonnées |
| First-fit | 2.1.6.d | Implémenter, comprendre le trade-off |
| Best-fit | 2.1.6.e | Implémenter, mesurer la fragmentation |
| Worst-fit | 2.1.6.f | Implémenter, comprendre pourquoi c'est mauvais |
| Next-fit | 2.1.6.g | Implémenter le cache de position |
| Segregated lists | 2.1.7.a-d | Bonus : listes par taille |
| Buddy system | 2.1.7.e-h | Bonus : puissances de 2 |
| Fragmentation | 2.1.8.a-d | Mesurer interne et externe |
| Coalescing | 2.1.10.a-f | Fusionner les blocs libres |

---

### 5.2 LDA — Langage de Description d'Algorithmes

```
FONCTION my_malloc QUI RETOURNE UN POINTEUR GÉNÉRIQUE ET PREND EN PARAMÈTRE size QUI EST UNE TAILLE EN BYTES
DÉBUT FONCTION
    SI size EST ÉGAL À 0 ALORS
        RETOURNER NUL
    FIN SI

    AFFECTER ALIGNER size SUR 16 BYTES À size

    DÉCLARER block COMME POINTEUR VERS HEADER

    SI LE HEAP N'EST PAS INITIALISÉ ALORS
        AFFECTER DEMANDER ESPACE AU SYSTÈME POUR size À block
        SI block EST NUL ALORS
            RETOURNER NUL
        FIN SI
        AFFECTER block À DÉBUT DU HEAP
    SINON
        AFFECTER CHERCHER BLOC LIBRE DE TAILLE size À block

        SI block TROUVÉ ALORS
            MARQUER block COMME ALLOUÉ
            DÉCOUPER block SI RESTE ASSEZ GRAND
        SINON
            AFFECTER DEMANDER ESPACE AU SYSTÈME POUR size À block
            SI block EST NUL ALORS
                RETOURNER NUL
            FIN SI
        FIN SI
    FIN SI

    RETOURNER ADRESSE DU PAYLOAD DE block
FIN FONCTION
```

---

### 5.3 Visualisation ASCII

#### Structure d'un bloc alloué

```
                    BLOCK STRUCTURE
         ┌─────────────────────────────────────────┐
         │              HEADER (32 bytes)          │
         │  ┌─────────────┬───────────────────┐    │
         │  │ size: 100   │ free: 0           │    │
         │  ├─────────────┼───────────────────┤    │
         │  │ prev: 0x... │ next: 0x...       │    │
         │  ├─────────────┼───────────────────┤    │
         │  │ magic: 0xDEADBEEF │ padding     │    │
         │  └─────────────┴───────────────────┘    │
         ├─────────────────────────────────────────┤
         │              PAYLOAD (100 bytes)        │
         │  ┌─────────────────────────────────┐    │
         │  │                                 │    │ ← Pointeur retourné
         │  │        User Data                │    │
         │  │                                 │    │
         │  └─────────────────────────────────┘    │
         └─────────────────────────────────────────┘
                         ↑
                  Aligné sur 16 bytes
```

#### Free List et Coalescing

```
AVANT FREE(B):
┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐
│ A: 100 │───→│ B: 200 │───→│ C: 150 │───→│ D: 100 │
│ ALLOC  │    │ ALLOC  │    │ FREE   │    │ ALLOC  │
└────────┘    └────────┘    └────────┘    └────────┘

APRÈS FREE(B) avec coalescing:
┌────────┐    ┌────────────────────┐    ┌────────┐
│ A: 100 │───→│ B+C: 382 (fusionné)│───→│ D: 100 │
│ ALLOC  │    │ FREE               │    │ ALLOC  │
└────────┘    └────────────────────┘    └────────┘
              (200 + 32 + 150 = 382)
```

#### Stratégies de recherche

```
FREE LIST: [64] → [128] → [32] → [256] → NULL

Demande: malloc(50)

FIRST-FIT:  Retourne [64]   (premier assez grand)
BEST-FIT:   Retourne [64]   (plus petit ≥ 50)
WORST-FIT:  Retourne [256]  (plus grand)
NEXT-FIT:   Dépend de la dernière position
```

---

### 5.5 Cours Complet

#### 5.5.1 Anatomie du Heap

```
┌─────────────────────────────────────────────────────────────────┐
│                     VIRTUAL ADDRESS SPACE                        │
├─────────────────────────────────────────────────────────────────┤
│  Stack    ↓  (grows down)                                        │
│  ...                                                             │
│  ...                                                             │
│  Heap     ↑  (grows up with sbrk)                                │
│  ├── Block 1: [header][payload...]                               │
│  ├── Block 2: [header][payload...]                               │
│  └── Block 3: [header][payload...]                               │
│  BSS  (uninitialized globals)                                    │
│  Data (initialized globals)                                      │
│  Text (code)                                                     │
└─────────────────────────────────────────────────────────────────┘
```

#### 5.5.2 Pourquoi sbrk vs mmap ?

| Aspect | sbrk | mmap |
|--------|------|------|
| Mécanisme | Étend le heap | Nouvelle région |
| Adresses | Contiguës | Quelconques |
| Libération | Impossible (sauf shrink) | munmap possible |
| Performance | Très rapide | Plus lent |
| Usage | Petites allocations | Grandes allocations |

glibc utilise les deux : sbrk pour < 128KB, mmap pour >= 128KB.

#### 5.5.3 Le problème de la fragmentation

**Fragmentation interne :** Espace gaspillé DANS un bloc
```
Demandé: 100 bytes
Alloué:  128 bytes (arrondi puissance de 2)
Gaspillé: 28 bytes (interne)
```

**Fragmentation externe :** Espace gaspillé ENTRE les blocs
```
FREE: [32] [64] [32]  = 128 bytes libres au total
Demande: malloc(100)  = ÉCHEC ! Pas de bloc de 100 contigu
```

Le coalescing réduit la fragmentation externe.

---

### 5.8 Mnémotechniques

#### ⛏️ MEME : "Mining Memory" — malloc

```
Dans Minecraft, tu mines pour obtenir des ressources.
malloc() = mine les bytes dont tu as besoin.

Steve creuse → Obtient des blocs
malloc(100) → Obtient 100 bytes

Si le chunk est vide, Steve doit aller plus loin (sbrk).
```

#### 💥 MEME : "Creeper Explosion" — Fragmentation

```
Un Creeper explose et laisse des trous partout.
free() sans coalescing = trous partout.

Avant: Terrain continu
Après: [trou][bloc][trou][bloc][trou]

Impossible de construire quelque chose de grand !
Solution: Réparer les trous (coalescing).
```

#### 🔨 MEME : "Crafting Table" — Coalescing

```
La Crafting Table combine des items.
Coalescing combine des blocs libres.

[Wood] + [Wood] + [Wood] → [Planks]
[Free 32] + [Free 64] → [Free 96]
```

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Conséquence | Solution |
|---|-------|-------------|----------|
| 1 | Pas d'alignement 16 | SIMD crash | ALIGN macro |
| 2 | Oublier magic check | Corruption silencieuse | Vérifier DEADBEEF |
| 3 | Coalescing partiel | Fragmentation | Fusionner prev ET next |
| 4 | malloc(0) → bloc | Gaspillage | Retourner NULL |
| 5 | Overflow nmemb×size | Allocation énorme | Check avant multiplication |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quel est l'alignement minimum requis pour malloc sur x86-64 ?**

A) 4 bytes
B) 8 bytes
C) 16 bytes
D) 32 bytes
E) 64 bytes
F) 1 byte
G) Dépend du type
H) Pas d'alignement requis
I) 128 bytes
J) Alignement naturel

**Réponse : C**

---

### Question 2
**Quelle stratégie minimise la fragmentation externe ?**

A) First-fit
B) Best-fit
C) Worst-fit
D) Next-fit
E) Random-fit
F) Toutes équivalentes
G) Aucune ne garantit
H) Last-fit
I) Buddy system
J) LIFO

**Réponse : B**

---

### Question 3
**Le coalescing se fait quand ?**

A) Lors de malloc uniquement
B) Lors de free uniquement
C) Lors de realloc uniquement
D) Lors de malloc et free
E) Jamais automatiquement
F) Périodiquement
G) Lors de free, avec voisins libres
H) Lors de malloc, si fragmentation haute
I) À la fermeture du programme
J) Par le garbage collector

**Réponse : G**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 2.1.4 — minecraft_malloc |
| **Difficulté** | ★★★★★★☆☆☆☆ (6/10) |
| **Bonus** | 💀 Expert (8/10) |
| **XP Base** | 750 |
| **XP Bonus** | ×4 = 3000 |
| **Durée** | 8-12 heures |
| **Fichiers** | 6 fichiers C + header + Makefile |
| **Concepts** | 12+ concepts du curriculum |
| **Tests** | 13 tests automatisés |
| **Mutants** | 5 solutions buggées |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1.4-minecraft_malloc",
    "generated_at": "2026-01-11",

    "metadata": {
      "exercise_id": "2.1.4",
      "exercise_name": "minecraft_malloc",
      "module": "2.1",
      "module_name": "Memory Management",
      "concept": "Heap Allocator",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "c17",
      "duration_minutes": 600,
      "xp_base": 750,
      "xp_bonus_multiplier": 4,
      "bonus_tier": "EXPERT",
      "bonus_icon": "💀",
      "complexity_time": "T3 O(n)",
      "complexity_space": "S2 O(1)+headers",
      "prerequisites": ["ex01", "ex02", "ex03", "pointers", "structs"],
      "domains": ["Mem", "Struct", "Algo"],
      "domains_bonus": ["MD"],
      "tags": ["malloc", "free", "heap", "coalescing", "fragmentation"],
      "meme_reference": "Minecraft"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/my_malloc.c": "/* Section 4.3 */",
      "mutants/mutant_a_alignment.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_coalesce.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 */",
      "mutants/mutant_e_return.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    }
  }
}
```

---

*Exercice généré avec HACKBRAIN v5.5.2*
*"L'excellence pédagogique ne se négocie pas"*
*Compatible ENGINE v22.1 + Mutation Tester*
