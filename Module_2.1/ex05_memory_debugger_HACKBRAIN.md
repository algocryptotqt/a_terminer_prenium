<thinking>
## Analyse du Concept
- Concept : Debugger mémoire custom (leak detection, double-free, overflow)
- Phase demandée : 2
- Adapté ? OUI — Comprendre comment fonctionnent Valgrind/ASan est très formateur

## Combo Base + Bonus
- Exercice de base : Wrapper malloc/free avec détection leak, double-free, guard bytes
- Bonus : Stack traces avec backtrace(), reports HTML, thread-safety
- Palier bonus : 🔥 Avancé (backtraces et symboles)
- Progression logique ? OUI — Base = détection, Bonus = diagnostic avancé

## Prérequis & Difficulté
- Prérequis réels : Wrapping de fonctions, hash tables, pointeurs
- Difficulté estimée : 5/10 (base), 7/10 (bonus)
- Cohérent avec phase ? OUI — Phase 2 = 4-6/10

## Aspect Fun/Culture
- Contexte choisi : Sherlock Holmes — Investigation de crimes mémoire
- MEME mnémotechnique : "Elementary, my dear Watson" = détection évidente
- Pourquoi c'est fun : Chaque bug = un crime à résoudre, indices = guard bytes

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : Hash table qui ne redimensionne pas → collision excessive
2. Mutant B (Safety) : Pas de vérification si ptr déjà dans le registre → faux double-free
3. Mutant C (Resource) : Guard bytes non vérifiés lors du free → overflow non détecté
4. Mutant D (Logic) : Peak bytes calculé après free au lieu d'avant → pic incorrect
5. Mutant E (Return) : Leak count inclut les blocs déjà freed → faux positifs

## Verdict
VALIDE — Exercice pratique couvrant 8 concepts de debugging (2.1.11.a-h)
</thinking>

---

# Exercice 2.1.5 : sherlock_memdbg

**Module :**
2.1.5 — Memory Debugging & Sanitization

**Concept :**
a-h — Leak detection, double-free, use-after-free, buffer overflow, guard bytes

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
code

**Tiers :**
3 — Synthèse (wrapping + tracking + reporting)

**Langage :**
C17

**Prérequis :**
- Wrapping de fonctions (macros __FILE__, __LINE__)
- Hash tables pour lookup O(1)
- Pointeurs et arithmétique de pointeurs (ex04)

**Domaines :**
Mem, Struct, Algo

**Durée estimée :**
240-360 min (4-6 heures)

**XP Base :**
400

**Complexité :**
T2 O(1) lookup × S2 O(n) pour n allocations

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex05_memory_debugger/
├── memdbg.h
├── memdbg.c
├── tracking.c
├── guards.c
├── report.c
└── Makefile
```

**Fonctions autorisées :**
- `malloc`, `free`, `calloc`, `realloc` (le vrai malloc système)
- `memset`, `memcpy`, `memmove`
- `write`, `snprintf`
- `backtrace`, `backtrace_symbols` (pour bonus)

**Fonctions interdites :**
- `printf`, `fprintf` (utiliser write/snprintf)

---

### 1.2 Consigne

#### 🎮 Version Culture Pop : "SHERLOCK HOLMES: Memory Crimes"

**"The Game is Afoot!"**

Tu es Sherlock Holmes, le plus grand détective de Baker Street. Ton client, Dr. Watson (le développeur), a des problèmes mystérieux : son programme crashe aléatoirement, sa mémoire fuit comme un vieux tuyau, et des données disparaissent sans explication.

**Les crimes à résoudre :**

| Crime | Nom technique | Indice |
|-------|---------------|--------|
| 🔍 Le Fantôme de la Mémoire | Memory Leak | Mémoire allouée mais jamais libérée |
| 👻 Le Double Meurtre | Double Free | Pointeur libéré deux fois |
| 💀 L'Accès Interdit | Use-After-Free | Accès à mémoire déjà libérée |
| 📜 Le Débordement | Buffer Overflow | Écriture hors limites |
| 🔮 Le Faux Alibi | Invalid Free | Free d'un pointeur non alloué |

**Tes outils d'investigation :**

| Outil | Fonction |
|-------|----------|
| 🔬 La Loupe | Guard bytes (DEADBEEF) |
| 📓 Le Carnet | Hash table des allocations |
| 🕵️ L'Interrogatoire | File/line tracking |
| 📊 Le Rapport Final | Statistics at shutdown |

---

#### 📚 Version Académique : Debugger Mémoire Custom

**Contexte technique :**

Les bugs mémoire sont parmi les plus difficiles à débugger :
- **Memory leaks** : Mémoire allouée jamais libérée
- **Double free** : Libération d'un pointeur déjà libéré
- **Use-after-free** : Accès à mémoire déjà libérée
- **Buffer overflow** : Écriture au-delà des limites allouées

Des outils comme Valgrind ou AddressSanitizer détectent ces problèmes. Cet exercice vous fait comprendre **comment** ils fonctionnent.

---

**Ta mission :**

Créer une bibliothèque `memdbg` qui wrappe malloc/free et détecte les erreurs mémoire courantes au runtime.

**Fonctionnalités requises :**
1. Wrapper transparent via macros
2. Tracking de toutes les allocations actives
3. Détection de leaks, double-free, invalid-free
4. Guard bytes pour détecter overflow/underflow
5. Rapport final avec statistiques

---

### 1.3 Prototypes

```c
/* ═══════════════════════════════════════════════════════════════════════════
 * INITIALISATION / SHUTDOWN
 * ═══════════════════════════════════════════════════════════════════════════ */

/* Active le debugger (appelé au démarrage) */
void memdbg_init(void);

/* Désactive et génère le rapport final */
void memdbg_shutdown(void);

/* ═══════════════════════════════════════════════════════════════════════════
 * WRAPPERS (utilisés via macros)
 * ═══════════════════════════════════════════════════════════════════════════ */

void *memdbg_malloc(size_t size, const char *file, int line);
void *memdbg_calloc(size_t n, size_t size, const char *file, int line);
void *memdbg_realloc(void *ptr, size_t size, const char *file, int line);
void memdbg_free(void *ptr, const char *file, int line);

/* Macros pour capturer file/line automatiquement */
#define malloc(size)       memdbg_malloc(size, __FILE__, __LINE__)
#define free(ptr)          memdbg_free(ptr, __FILE__, __LINE__)
#define calloc(n, size)    memdbg_calloc(n, size, __FILE__, __LINE__)
#define realloc(ptr, size) memdbg_realloc(ptr, size, __FILE__, __LINE__)

/* ═══════════════════════════════════════════════════════════════════════════
 * TYPES D'ERREURS
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef enum {
    MEMDBG_OK,
    MEMDBG_LEAK,           /* Mémoire non libérée */
    MEMDBG_DOUBLE_FREE,    /* Libération multiple */
    MEMDBG_INVALID_FREE,   /* Pointeur jamais alloué */
    MEMDBG_OVERFLOW,       /* Écriture après la fin */
    MEMDBG_UNDERFLOW       /* Écriture avant le début */
} memdbg_error_t;

/* Callback appelé lors d'une erreur */
typedef void (*memdbg_error_handler_t)(
    memdbg_error_t error,
    void *ptr,
    const char *alloc_file, int alloc_line,
    const char *free_file, int free_line
);

void memdbg_set_error_handler(memdbg_error_handler_t handler);

/* ═══════════════════════════════════════════════════════════════════════════
 * GUARD BYTES
 * ═══════════════════════════════════════════════════════════════════════════ */

#define GUARD_PATTERN 0xDEADBEEF
#define GUARD_SIZE 8  /* bytes avant et après */

/* Vérifier l'intégrité des guards d'un bloc */
int memdbg_check_guards(void *ptr);

/* Vérifier tous les blocs alloués */
int memdbg_check_all(void);

/* ═══════════════════════════════════════════════════════════════════════════
 * STATISTIQUES ET RAPPORT
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    size_t total_allocations;      /* Nombre total d'allocations */
    size_t total_frees;            /* Nombre total de free */
    size_t current_allocations;    /* Allocations actives */
    size_t bytes_allocated;        /* Bytes actuellement alloués */
    size_t peak_bytes;             /* Pic d'utilisation */
    size_t leaks_detected;         /* Fuites trouvées */
    size_t double_frees;           /* Double-free détectés */
    size_t corruptions;            /* Corruptions détectées */
} memdbg_stats_t;

memdbg_stats_t memdbg_get_stats(void);

/* Afficher le rapport final */
void memdbg_print_report(void);

/* Lister toutes les allocations actives */
void memdbg_list_allocations(void);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Comment fonctionne Valgrind ?

Valgrind utilise une technique appelée **Dynamic Binary Instrumentation (DBI)** :
1. Il intercepte chaque instruction du programme
2. Il maintient un "shadow memory" qui trace l'état de chaque byte
3. Il vérifie chaque accès mémoire contre ce shadow

**Notre approche** est plus simple : on wrappe malloc/free au niveau source.

### 2.2 AddressSanitizer (ASan) vs Valgrind

| Aspect | Valgrind | ASan |
|--------|----------|------|
| Overhead | 10-50× | 2× |
| Technique | DBI (runtime) | Compile-time instrumentation |
| Précision | Byte-accurate | Zone-based |
| Setup | Juste `valgrind ./prog` | Recompiler avec `-fsanitize=address` |

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation | Cas d'usage |
|--------|-------------|-------------|
| **Security Researcher** | Trouver des vulnérabilités | Use-after-free = CVE potentielle |
| **Game Developer** | Debug de memory leaks | Profiling mémoire sur consoles |
| **Embedded Engineer** | Systèmes sans Valgrind | Custom allocators avec tracking |
| **QA Engineer** | Tests automatisés | CI/CD avec ASan activé |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
memdbg.h  memdbg.c  tracking.c  guards.c  report.c  main.c  Makefile

$ make

$ ./test_memdbg
=== Running tests ===
Test 1: No leak... PASS
Test 2: Leak detection...
  LEAK detected: 200 bytes at 0x55a8b5400100
  Allocated at: main.c:15
  Never freed
  ... PASS
Test 3: Double-free detection...
  ERROR: Double-free detected!
  Pointer: 0x55a8b5400100
  Originally allocated at: main.c:20
  First freed at: main.c:21
  Second free attempt at: main.c:22
  ... PASS

=== MEMORY DEBUG REPORT ===
Total allocations: 5
Total frees: 4
Leaked: 1 block (200 bytes)
Peak memory: 1500 bytes
Corruptions: 0
===========================

All tests passed!
```

---

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Domaines Bonus :**
`ASM (backtrace), Net (HTML report)`

#### 3.1.1 Consigne Bonus

**🎮 "Sherlock's Advanced Deduction"**

Pour les cas les plus complexes, Sherlock a besoin de plus d'indices. Ajoute les stack traces pour voir exactement où chaque allocation a été faite.

```c
/* ═══════════════════════════════════════════════════════════════════════════
 * STACK TRACES
 * ═══════════════════════════════════════════════════════════════════════════ */

#define MAX_STACK_DEPTH 16

typedef struct {
    void *frames[MAX_STACK_DEPTH];
    int   depth;
    char **symbols;  /* Résolu par backtrace_symbols */
} stack_trace_t;

/* Capturer la stack trace actuelle */
stack_trace_t *memdbg_capture_stack(void);

/* Libérer une stack trace */
void memdbg_free_stack(stack_trace_t *trace);

/* ═══════════════════════════════════════════════════════════════════════════
 * RAPPORT HTML
 * ═══════════════════════════════════════════════════════════════════════════ */

/* Générer un rapport HTML interactif */
void memdbg_generate_html_report(const char *filename);
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette — Tests automatisés

| Test | Description | Entrée | Attendu | Points |
|------|-------------|--------|---------|--------|
| `test_no_leak` | Pas de fuite | malloc+free | leaks == 0 | 10 |
| `test_detect_leak` | Fuite détectée | malloc sans free | leaks == 1 | 10 |
| `test_double_free` | Double-free signalé | free×2 | double_frees == 1 | 10 |
| `test_invalid_free` | Free invalide | free(&stack_var) | erreur signalée | 10 |
| `test_guard_overflow` | Overflow via guards | write past end | corruption == 1 | 10 |
| `test_guard_underflow` | Underflow via guards | write before start | corruption == 1 | 10 |
| `test_peak_tracking` | Pic mémoire correct | alloc/free pattern | peak correct | 10 |
| `test_file_line` | File/line capturés | — | info dans rapport | 10 |
| `test_stress` | 10000 alloc/free | random | no internal leak | 10 |
| `test_null_free` | free(NULL) | NULL | no crash | 10 |

---

### 4.2 main.c de test

```c
#include "memdbg.h"
#include <assert.h>
#include <string.h>

static void test_no_leak(void)
{
    memdbg_init();

    void *p = malloc(100);
    free(p);

    memdbg_stats_t s = memdbg_get_stats();
    assert(s.leaks_detected == 0);
    assert(s.current_allocations == 0);

    memdbg_shutdown();
}

static void test_detect_leak(void)
{
    memdbg_init();

    void *p = malloc(100);
    /* Pas de free ! */

    memdbg_stats_t s = memdbg_get_stats();
    assert(s.current_allocations == 1);

    memdbg_shutdown();
    /* Le shutdown devrait reporter la fuite */
}

static void test_double_free(void)
{
    static int double_free_count = 0;

    memdbg_set_error_handler(
        lambda(void, (memdbg_error_t e, void *p, const char *af, int al,
                      const char *ff, int fl) {
            if (e == MEMDBG_DOUBLE_FREE)
                double_free_count++;
        })
    );

    memdbg_init();

    void *p = malloc(100);
    free(p);
    free(p);  /* Double free ! */

    assert(double_free_count == 1);

    memdbg_shutdown();
}

static void test_guard_overflow(void)
{
    memdbg_init();

    char *buf = malloc(10);
    buf[10] = 'X';  /* Overflow! Écrit sur le guard */

    int result = memdbg_check_guards(buf);
    assert(result == 0);  /* 0 = corruption détectée */

    free(buf);
    memdbg_shutdown();
}

static void test_null_free(void)
{
    memdbg_init();
    free(NULL);  /* Ne doit pas crasher */
    memdbg_shutdown();
}

int main(void)
{
    test_no_leak();
    test_detect_leak();
    test_double_free();
    test_guard_overflow();
    test_null_free();

    write(1, "All tests passed!\n", 18);
    return 0;
}
```

---

### 4.3 Solution de référence — memdbg.c

```c
#include "memdbg.h"
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <stdint.h>

/* ═══════════════════════════════════════════════════════════════════════════
 * STRUCTURES INTERNES
 * ═══════════════════════════════════════════════════════════════════════════ */

#define HASH_SIZE 1024
#define GUARD_PATTERN 0xDEADBEEF
#define GUARD_SIZE 8

/* Entry dans la hash table des allocations */
typedef struct alloc_entry {
    void *user_ptr;              /* Pointeur retourné à l'utilisateur */
    void *real_ptr;              /* Pointeur réel (avec guards) */
    size_t size;                 /* Taille demandée */
    const char *file;            /* Fichier d'allocation */
    int line;                    /* Ligne d'allocation */
    int freed;                   /* Déjà libéré ? */
    const char *free_file;       /* Fichier de libération */
    int free_line;               /* Ligne de libération */
    struct alloc_entry *next;    /* Chaînage hash */
} alloc_entry_t;

/* État global */
static alloc_entry_t *g_hash_table[HASH_SIZE] = {0};
static memdbg_stats_t g_stats = {0};
static memdbg_error_handler_t g_error_handler = NULL;
static int g_initialized = 0;

/* ═══════════════════════════════════════════════════════════════════════════
 * HASH TABLE
 * ═══════════════════════════════════════════════════════════════════════════ */

static size_t hash_ptr(void *ptr)
{
    return ((uintptr_t)ptr >> 3) % HASH_SIZE;
}

static alloc_entry_t *find_entry(void *user_ptr)
{
    size_t idx = hash_ptr(user_ptr);
    alloc_entry_t *entry = g_hash_table[idx];

    while (entry)
    {
        if (entry->user_ptr == user_ptr)
            return (entry);
        entry = entry->next;
    }
    return (NULL);
}

static void add_entry(alloc_entry_t *entry)
{
    size_t idx = hash_ptr(entry->user_ptr);
    entry->next = g_hash_table[idx];
    g_hash_table[idx] = entry;
}

static void remove_entry(void *user_ptr)
{
    size_t idx = hash_ptr(user_ptr);
    alloc_entry_t **prev = &g_hash_table[idx];

    while (*prev)
    {
        if ((*prev)->user_ptr == user_ptr)
        {
            alloc_entry_t *to_remove = *prev;
            *prev = to_remove->next;
            free(to_remove);
            return;
        }
        prev = &(*prev)->next;
    }
}

/* ═══════════════════════════════════════════════════════════════════════════
 * GUARDS
 * ═══════════════════════════════════════════════════════════════════════════ */

static void write_guards(void *real_ptr, size_t size)
{
    uint32_t *guard_before = (uint32_t *)real_ptr;
    uint32_t *guard_after = (uint32_t *)((char *)real_ptr + GUARD_SIZE + size);

    for (int i = 0; i < GUARD_SIZE / 4; i++)
    {
        guard_before[i] = GUARD_PATTERN;
        guard_after[i] = GUARD_PATTERN;
    }
}

static int check_guards_internal(void *real_ptr, size_t size)
{
    uint32_t *guard_before = (uint32_t *)real_ptr;
    uint32_t *guard_after = (uint32_t *)((char *)real_ptr + GUARD_SIZE + size);

    for (int i = 0; i < GUARD_SIZE / 4; i++)
    {
        if (guard_before[i] != GUARD_PATTERN)
            return (0);  /* Underflow */
        if (guard_after[i] != GUARD_PATTERN)
            return (0);  /* Overflow */
    }
    return (1);  /* OK */
}

/* ═══════════════════════════════════════════════════════════════════════════
 * INTERFACE PUBLIQUE
 * ═══════════════════════════════════════════════════════════════════════════ */

void memdbg_init(void)
{
    memset(g_hash_table, 0, sizeof(g_hash_table));
    memset(&g_stats, 0, sizeof(g_stats));
    g_initialized = 1;
}

void memdbg_shutdown(void)
{
    /* Compter les leaks */
    for (int i = 0; i < HASH_SIZE; i++)
    {
        alloc_entry_t *entry = g_hash_table[i];
        while (entry)
        {
            if (!entry->freed)
                g_stats.leaks_detected++;
            entry = entry->next;
        }
    }

    memdbg_print_report();

    /* Cleanup */
    for (int i = 0; i < HASH_SIZE; i++)
    {
        alloc_entry_t *entry = g_hash_table[i];
        while (entry)
        {
            alloc_entry_t *next = entry->next;
            if (!entry->freed)
                free(entry->real_ptr);  /* Libérer le bloc leaké */
            free(entry);
            entry = next;
        }
        g_hash_table[i] = NULL;
    }

    g_initialized = 0;
}

void *memdbg_malloc(size_t size, const char *file, int line)
{
    if (size == 0)
        return (NULL);

    /* Allouer avec espace pour guards */
    size_t total = GUARD_SIZE + size + GUARD_SIZE;
    void *real_ptr = malloc(total);
    if (!real_ptr)
        return (NULL);

    /* Écrire les guards */
    write_guards(real_ptr, size);

    /* Pointeur utilisateur (après le guard before) */
    void *user_ptr = (char *)real_ptr + GUARD_SIZE;

    /* Créer l'entry */
    alloc_entry_t *entry = malloc(sizeof(alloc_entry_t));
    if (!entry)
    {
        free(real_ptr);
        return (NULL);
    }

    entry->user_ptr = user_ptr;
    entry->real_ptr = real_ptr;
    entry->size = size;
    entry->file = file;
    entry->line = line;
    entry->freed = 0;
    entry->free_file = NULL;
    entry->free_line = 0;

    add_entry(entry);

    /* Stats */
    g_stats.total_allocations++;
    g_stats.current_allocations++;
    g_stats.bytes_allocated += size;
    if (g_stats.bytes_allocated > g_stats.peak_bytes)
        g_stats.peak_bytes = g_stats.bytes_allocated;

    return (user_ptr);
}

void memdbg_free(void *ptr, const char *file, int line)
{
    if (!ptr)
        return;

    alloc_entry_t *entry = find_entry(ptr);

    if (!entry)
    {
        /* Invalid free */
        if (g_error_handler)
            g_error_handler(MEMDBG_INVALID_FREE, ptr, NULL, 0, file, line);
        return;
    }

    if (entry->freed)
    {
        /* Double free */
        g_stats.double_frees++;
        if (g_error_handler)
            g_error_handler(MEMDBG_DOUBLE_FREE, ptr,
                           entry->file, entry->line,
                           file, line);
        return;
    }

    /* Vérifier guards avant de libérer */
    if (!check_guards_internal(entry->real_ptr, entry->size))
    {
        g_stats.corruptions++;
        if (g_error_handler)
            g_error_handler(MEMDBG_OVERFLOW, ptr,
                           entry->file, entry->line,
                           file, line);
    }

    /* Marquer comme libéré */
    entry->freed = 1;
    entry->free_file = file;
    entry->free_line = line;

    /* Stats */
    g_stats.total_frees++;
    g_stats.current_allocations--;
    g_stats.bytes_allocated -= entry->size;

    /* Libérer réellement */
    free(entry->real_ptr);
}

void *memdbg_calloc(size_t n, size_t size, const char *file, int line)
{
    size_t total = n * size;
    if (n != 0 && total / n != size)
        return (NULL);  /* Overflow */

    void *ptr = memdbg_malloc(total, file, line);
    if (ptr)
        memset(ptr, 0, total);

    return (ptr);
}

void *memdbg_realloc(void *ptr, size_t size, const char *file, int line)
{
    if (!ptr)
        return memdbg_malloc(size, file, line);

    if (size == 0)
    {
        memdbg_free(ptr, file, line);
        return (NULL);
    }

    alloc_entry_t *entry = find_entry(ptr);
    if (!entry || entry->freed)
        return (NULL);

    void *new_ptr = memdbg_malloc(size, file, line);
    if (!new_ptr)
        return (NULL);

    size_t copy_size = (entry->size < size) ? entry->size : size;
    memcpy(new_ptr, ptr, copy_size);

    memdbg_free(ptr, file, line);

    return (new_ptr);
}

int memdbg_check_guards(void *ptr)
{
    alloc_entry_t *entry = find_entry(ptr);
    if (!entry)
        return (-1);

    return check_guards_internal(entry->real_ptr, entry->size);
}

int memdbg_check_all(void)
{
    int all_ok = 1;

    for (int i = 0; i < HASH_SIZE; i++)
    {
        alloc_entry_t *entry = g_hash_table[i];
        while (entry)
        {
            if (!entry->freed)
            {
                if (!check_guards_internal(entry->real_ptr, entry->size))
                    all_ok = 0;
            }
            entry = entry->next;
        }
    }

    return (all_ok);
}

void memdbg_set_error_handler(memdbg_error_handler_t handler)
{
    g_error_handler = handler;
}

memdbg_stats_t memdbg_get_stats(void)
{
    return (g_stats);
}

void memdbg_print_report(void)
{
    char buf[512];
    int len;

    write(1, "\n=== MEMORY DEBUG REPORT ===\n", 29);

    /* Lister les leaks */
    for (int i = 0; i < HASH_SIZE; i++)
    {
        alloc_entry_t *entry = g_hash_table[i];
        while (entry)
        {
            if (!entry->freed)
            {
                len = snprintf(buf, sizeof(buf),
                    "LEAK: %zu bytes at %p\n  Allocated at: %s:%d\n",
                    entry->size, entry->user_ptr,
                    entry->file, entry->line);
                write(1, buf, len);
            }
            entry = entry->next;
        }
    }

    /* Stats */
    len = snprintf(buf, sizeof(buf),
        "\nSummary:\n"
        "  Total allocations: %zu\n"
        "  Total frees: %zu\n"
        "  Leaked: %zu blocks\n"
        "  Peak memory: %zu bytes\n"
        "  Double-frees: %zu\n"
        "  Corruptions: %zu\n",
        g_stats.total_allocations,
        g_stats.total_frees,
        g_stats.leaks_detected,
        g_stats.peak_bytes,
        g_stats.double_frees,
        g_stats.corruptions);
    write(1, buf, len);

    write(1, "============================\n\n", 30);
}
```

---

### 4.10 Solutions Mutantes

#### Mutant A (Boundary) : Hash sans redimensionnement

```c
#define HASH_SIZE 8  /* TROP PETIT ! */

/* Avec 8 buckets et 1000 allocations, chaque bucket a ~125 entrées
   → O(n) au lieu de O(1) */
```

#### Mutant B (Safety) : Pas de check freed

```c
void memdbg_free_mutant_b(void *ptr, const char *file, int line)
{
    alloc_entry_t *entry = find_entry(ptr);
    if (!entry) return;

    /* MANQUANT : if (entry->freed) → double free */

    entry->freed = 1;
    free(entry->real_ptr);
}
/* Pourquoi faux : Double-free non détecté */
```

#### Mutant C (Resource) : Guards non vérifiés au free

```c
void memdbg_free_mutant_c(void *ptr, const char *file, int line)
{
    alloc_entry_t *entry = find_entry(ptr);
    if (!entry || entry->freed) return;

    /* MANQUANT : check_guards_internal() */

    entry->freed = 1;
    free(entry->real_ptr);
}
/* Pourquoi faux : Overflow silencieux, corruption non détectée */
```

#### Mutant D (Logic) : Peak calculé après free

```c
void *memdbg_malloc_mutant_d(size_t size, ...)
{
    /* ... allocation ... */

    g_stats.bytes_allocated += size;
    /* Peak calculé APRÈS, au free, au lieu de maintenant */
}

void memdbg_free_mutant_d(void *ptr, ...)
{
    /* ... */
    if (g_stats.bytes_allocated > g_stats.peak_bytes)
        g_stats.peak_bytes = g_stats.bytes_allocated;  /* FAUX ! */
    g_stats.bytes_allocated -= entry->size;
}
/* Pourquoi faux : Le pic n'est jamais capturé au bon moment */
```

#### Mutant E (Return) : malloc(0) retourne non-NULL

```c
void *memdbg_malloc_mutant_e(size_t size, ...)
{
    /* MANQUANT : if (size == 0) return NULL; */

    /* Alloue un bloc même pour size=0 */
}
/* Pourquoi faux : Comportement non standard, waste de mémoire */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Référence | Maîtrise attendue |
|---------|-----------|-------------------|
| Leak detection | 2.1.11.a | Tracker toutes les allocations |
| Double-free | 2.1.11.b | Détecter les libérations multiples |
| Use-after-free | 2.1.11.c | Marquer les blocs libérés |
| Buffer overflow | 2.1.11.d | Guard bytes pattern |
| Allocation tracking | 2.1.11.e | Hash table par pointeur |
| Memory report | 2.1.11.f | Statistiques et dumps |
| Stack traces | 2.1.11.g | Bonus: backtrace() |
| Guard bytes | 2.1.11.h | DEADBEEF pattern |

---

### 5.3 Visualisation ASCII

#### Layout d'un bloc avec guards

```
                    BLOCK LAYOUT
         ┌─────────────────────────────────────────┐
         │      GUARD BEFORE (8 bytes)             │
         │  ┌─────────────────────────────────┐    │
         │  │ 0xDEADBEEF │ 0xDEADBEEF        │    │
         │  └─────────────────────────────────┘    │
         ├─────────────────────────────────────────┤
         │      USER DATA (size bytes)             │
         │  ┌─────────────────────────────────┐    │
         │  │                                 │    │ ← Pointeur retourné
         │  │  Données utilisateur            │    │
         │  │                                 │    │
         │  └─────────────────────────────────┘    │
         ├─────────────────────────────────────────┤
         │      GUARD AFTER (8 bytes)              │
         │  ┌─────────────────────────────────┐    │
         │  │ 0xDEADBEEF │ 0xDEADBEEF        │    │
         │  └─────────────────────────────────┘    │
         └─────────────────────────────────────────┘

Si l'utilisateur écrit au-delà de size → Guard After corrompu
Si l'utilisateur écrit avant le début → Guard Before corrompu
```

#### Hash Table des allocations

```
HASH TABLE (1024 buckets)
┌────────────────────────────────────────────────────────┐
│ [0] → Entry(0x1000) → Entry(0x8000) → NULL            │
│ [1] → NULL                                             │
│ [2] → Entry(0x2010) → NULL                            │
│ [3] → Entry(0x3018) → Entry(0x7018) → Entry(...) → NULL│
│ ...                                                    │
│ [1023] → Entry(0xFFF8) → NULL                          │
└────────────────────────────────────────────────────────┘

Chaque Entry contient:
┌─────────────────────────────────┐
│ user_ptr: 0x1000                │
│ real_ptr: 0x0FF8                │
│ size: 100                       │
│ file: "main.c"                  │
│ line: 42                        │
│ freed: 0                        │
│ next: 0x8000                    │
└─────────────────────────────────┘
```

---

### 5.8 Mnémotechniques

#### 🔍 MEME : "Elementary, my dear Watson" — Guard Bytes

```
Sherlock détecte toujours les indices laissés par le criminel.

Les guards 0xDEADBEEF sont comme les traces de pas :
- Intacts ? Tout va bien
- Modifiés ? CRIME DÉTECTÉ !

"Le jeu est en marche, Watson. Les guards ne mentent jamais."
```

#### 📓 MEME : "The Hound of the Baskervilles" — Memory Leak

```
Comme le chien fantôme qui hante les Baskerville,
la mémoire non libérée hante ton programme.

Tu ne la vois pas, mais elle est là.
Elle grossit dans l'ombre.
Jusqu'au jour où... OUT OF MEMORY.

Solution : memdbg_shutdown() révèle tous les fantômes.
```

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 2.1.5 — sherlock_memdbg |
| **Difficulté** | ★★★★★☆☆☆☆☆ (5/10) |
| **Bonus** | 🔥 Avancé (7/10) |
| **XP Base** | 400 |
| **XP Bonus** | ×3 = 1200 |
| **Durée** | 4-6 heures |
| **Fichiers** | 5 fichiers C + header + Makefile |
| **Concepts** | 8 concepts debugging |
| **Tests** | 10 tests automatisés |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1.5-sherlock_memdbg",
    "generated_at": "2026-01-11",

    "metadata": {
      "exercise_id": "2.1.5",
      "exercise_name": "sherlock_memdbg",
      "module": "2.1",
      "module_name": "Memory Management",
      "concept": "Memory Debugging",
      "type": "code",
      "tier": 3,
      "phase": 2,
      "difficulty": 5,
      "language": "c17",
      "duration_minutes": 300,
      "xp_base": 400,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "ADVANCED",
      "bonus_icon": "🔥",
      "domains": ["Mem", "Struct", "Algo"],
      "tags": ["debugging", "valgrind", "leak", "guard-bytes"],
      "meme_reference": "Sherlock Holmes"
    }
  }
}
```

---

*Exercice généré avec HACKBRAIN v5.5.2*
*"L'excellence pédagogique ne se négocie pas"*
