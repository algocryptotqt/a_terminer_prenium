# Exercice 2.3.11 : aperture_mmap

**Module :**
2.3 — File Systems

**Concept :**
Memory-Mapped I/O — Les Portails entre Mémoire et Fichiers

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
2 — Mélange (concepts mmap + munmap + msync + mprotect)

**Langage :**
C (c17)

**Prérequis :**
- 2.3.0-2.3.5 (Concepts filesystem de base)
- Gestion mémoire (malloc, free)
- Pointeurs et adresses
- File descriptors

**Domaines :**
FS, Mem

**Durée estimée :**
300 min (5h)

**XP Base :**
250

**Complexité :**
T4 O(1) × S5 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex11/
├── aperture_mmap.h
├── aperture_mmap.c
├── aperture_benchmark.c
└── Makefile
```

**Fonctions autorisées :**
- `mmap`, `munmap`, `msync`, `mprotect`, `madvise` (sys/mman.h)
- `open`, `close`, `fstat`, `read` (pour comparaison)
- `malloc`, `free`, `realloc`
- `memcpy`, `memset`
- `clock_gettime` (pour benchmark)
- `printf`, `fprintf`

**Fonctions interdites :**
- `fopen`, `fread`, `fwrite` (on utilise les syscalls bas niveau)

### 1.2 Consigne

**🎮 PORTAL — The Aperture Science Handheld Portal Device**

*"The Enrichment Center reminds you that the weighted companion cube will never threaten to stab you and, in fact, cannot speak."* — GLaDOS

Dans le jeu culte de Valve, le **Portal Gun** crée des connexions instantanées entre deux points
de l'espace. Tu places un portail bleu quelque part, un portail orange ailleurs, et tout ce qui
entre par l'un ressort par l'autre. Pas de temps de chargement, pas de copie — juste une connexion
directe.

**mmap()** est exactement ça : **le Portal Gun de la programmation** !

```
THE APERTURE SCIENCE MEMORY MAPPING DEVICE
═══════════════════════════════════════════════════════════════════════════════

    MEMORY SPACE                              FILESYSTEM
    (Blue Portal)                             (Orange Portal)
    ┌─────────────┐                          ┌─────────────────┐
    │             │        PORTAL            │   data.bin      │
    │   void *ptr │◄══════════════════════►  │   [████████]    │
    │             │        CONNECTION        │   offset: 0     │
    │   ptr[0]    │   ←── Same data! ──→     │   byte 0        │
    │   ptr[1]    │   ←── Same data! ──→     │   byte 1        │
    │   ptr[2]    │   ←── Same data! ──→     │   byte 2        │
    │   ...       │                          │   ...           │
    └─────────────┘                          └─────────────────┘

    Quand tu modifies ptr[0], le fichier change INSTANTANÉMENT !
    (Pas de read(), pas de write(), pas de copie — juste un PORTAIL)

═══════════════════════════════════════════════════════════════════════════════
```

**Pourquoi c'est révolutionnaire ?**

Avec `read()` traditionnel, tu dois :
1. Allouer un buffer
2. Copier les données du disque vers le buffer
3. Lire depuis le buffer
4. Écrire dans le buffer
5. Copier le buffer vers le disque

Avec `mmap()` :
1. Créer le portail
2. C'est tout. Tu lis/écris directement dans le fichier via ton pointeur !

**Ta mission :**

Créer `aperture_mmap`, une bibliothèque de memory-mapping qui wrap les fonctions système
et offre une interface intuitive pour manipuler les portails mémoire-fichier.

### 1.2.2 Consigne Académique

Implémenter une bibliothèque de gestion de memory-mapped I/O qui encapsule les appels
système mmap/munmap/msync/mprotect/madvise. La bibliothèque doit gérer un contexte
qui track toutes les régions mappées et permettre des opérations de benchmark.

**Entrée :**
- Chemins de fichiers à mapper
- Paramètres de protection et flags

**Sortie :**
- Pointeurs vers les régions mappées
- Statistiques de performance

**Contraintes :**
- Gérer correctement tous les flags de protection
- Supporter MAP_SHARED, MAP_PRIVATE, et MAP_ANONYMOUS
- Tracker les page faults pour analyse de performance

### 1.3 Prototypes

```c
#include <sys/mman.h>
#include <stdint.h>
#include <stdbool.h>

/*
 * =============================================================================
 *                    APERTURE SCIENCE MEMORY MAPPING DEVICE
 * =============================================================================
 *
 * "We do what we must, because we can." — GLaDOS
 */

/* Un portail = une région mémoire mappée */
typedef struct {
    void       *aperture;        /* Adresse du portail (mapped address) */
    size_t      dimension;       /* Taille du portail */
    int         security_level;  /* PROT_* flags */
    int         portal_type;     /* MAP_* flags */
    int         chamber_fd;      /* File descriptor (-1 si anonymous) */
    off_t       entry_point;     /* Offset dans le fichier */
    bool        synchronized;    /* Synced avec le fichier ? */
    const char *chamber_name;    /* Nom du fichier (pour debug) */
} portal_t;

/* Le contexte Aperture Science — gère tous les portails */
typedef struct {
    portal_t   *portals;         /* Array de portails actifs */
    size_t      active_count;    /* Nombre de portails ouverts */
    size_t      capacity;        /* Capacité de l'array */
    uint64_t    cube_deliveries; /* Page faults (lazy loading) */
    bool        glados_active;   /* Tracking enabled */
} aperture_ctx_t;

/*
 * =============================================================================
 *                         PORTAL GUN OPERATIONS
 * =============================================================================
 */

/* Initialiser le labo Aperture */
aperture_ctx_t *aperture_init(void);

/* Détruire le labo (et tous les portails) */
void aperture_shutdown(aperture_ctx_t *ctx);

/* 2.3.22.a: fire_blue_portal — Mapper un fichier en mémoire */
void *fire_blue_portal(aperture_ctx_t *ctx, const char *chamber_path,
                       size_t dimension, int security_level,
                       int portal_type, off_t entry_point);

/* 2.3.22.f: fire_void_portal — Mapping anonyme (pas de fichier) */
void *fire_void_portal(aperture_ctx_t *ctx, size_t dimension,
                       int security_level);

/* 2.3.22.g: close_portal — Fermer un portail (munmap) */
int close_portal(aperture_ctx_t *ctx, void *aperture, size_t dimension);

/* 2.3.22.h: sync_portals — Synchroniser avec le fichier (msync) */
int sync_portals(aperture_ctx_t *ctx, void *aperture, size_t dimension,
                 int sync_mode);

/* 2.3.22.i: portal_security — Changer les protections (mprotect) */
int portal_security(aperture_ctx_t *ctx, void *aperture, size_t dimension,
                    int new_security);

/* 2.3.22.j: glados_hint — Donner un conseil au kernel (madvise) */
int glados_hint(aperture_ctx_t *ctx, void *aperture, size_t dimension,
                int advice);

/*
 * =============================================================================
 *                           TEST CHAMBER CONSTANTS
 * =============================================================================
 */

/* Niveaux de sécurité (Protection flags) — 2.3.22.c */
#define PORTAL_SEC_READ    PROT_READ   /* Lecture autorisée */
#define PORTAL_SEC_WRITE   PROT_WRITE  /* Écriture autorisée */
#define PORTAL_SEC_EXEC    PROT_EXEC   /* Exécution autorisée */
#define PORTAL_SEC_NONE    PROT_NONE   /* Aucun accès (piège!) */

/* Types de portail (Mapping flags) — 2.3.22.d,e,f */
#define PORTAL_DUAL        MAP_SHARED      /* d: Modifications partagées */
#define PORTAL_PERSONAL    MAP_PRIVATE     /* e: Copy-on-write */
#define PORTAL_VOID        MAP_ANONYMOUS   /* f: Sans fichier */

/* Modes de synchronisation — 2.3.22.h */
#define SYNC_IMMEDIATE     MS_SYNC         /* Attendre la fin */
#define SYNC_BACKGROUND    MS_ASYNC        /* Async */

/* Conseils à GLaDOS — 2.3.22.j */
#define GLADOS_NORMAL      MADV_NORMAL     /* Accès normal */
#define GLADOS_RANDOM      MADV_RANDOM     /* Accès aléatoire */
#define GLADOS_SEQUENTIAL  MADV_SEQUENTIAL /* Accès séquentiel */
#define GLADOS_WILLNEED    MADV_WILLNEED   /* Va être utilisé bientôt */
#define GLADOS_DONTNEED    MADV_DONTNEED   /* Plus besoin */

/*
 * =============================================================================
 *                           BENCHMARK FACILITY
 * =============================================================================
 */

/* Résultats du test comparatif Portal vs Traditional */
typedef struct {
    double      portal_time_ms;    /* Temps avec mmap */
    double      traditional_time_ms; /* Temps avec read() */
    size_t      cubes_processed;   /* Bytes traités */
    bool        lazy_verified;     /* Lazy loading confirmé */
    double      speedup_factor;    /* Combien de fois plus rapide */
} test_chamber_results_t;

/* 2.3.22.b: Démontrer les avantages des portails */
void run_test_chamber(const char *chamber_path, test_chamber_results_t *results);

/* 2.3.22.k: Obtenir le compteur de page faults */
uint64_t get_cube_deliveries(aperture_ctx_t *ctx);

/*
 * =============================================================================
 *                              UTILITIES
 * =============================================================================
 */

/* Trouver un portail par son adresse */
portal_t *find_portal(aperture_ctx_t *ctx, void *aperture);

/* Afficher tous les portails actifs */
void display_all_portals(aperture_ctx_t *ctx);

/* Obtenir des stats sur un portail */
void get_portal_stats(portal_t *portal);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Histoire de mmap

```
"Remember, the Aperture Science Bring Your Daughter to Work Day is
 the perfect time to have her tested." — GLaDOS
```

**mmap()** existe depuis les premiers systèmes Unix (4.2BSD, 1983).
L'idée révolutionnaire : au lieu de copier des données fichier→mémoire,
créer une correspondance directe grâce au système de mémoire virtuelle.

### 2.2 Comment GLaDOS (le Kernel) Gère les Portails

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        LE MÉCANISME mmap()                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   1. TON CODE appelle mmap()                                                │
│      ┌─────────────────────────────────────────────────────┐               │
│      │ ptr = mmap(NULL, 4096, PROT_READ, MAP_SHARED, fd, 0)│               │
│      └─────────────────────────────────────────────────────┘               │
│                              │                                              │
│                              ▼                                              │
│   2. LE KERNEL crée une entrée dans la Page Table                          │
│      ┌─────────────────────────────────────────────────────┐               │
│      │ Virtual Address 0x7fff1000 → "va chercher page 0    │               │
│      │                               du fichier fd quand    │               │
│      │                               quelqu'un y accède"    │               │
│      └─────────────────────────────────────────────────────┘               │
│                              │                                              │
│   3. TON CODE accède à ptr[0] — PREMIER ACCÈS                              │
│      ┌─────────────────────────────────────────────────────┐               │
│      │ CPU: "Page 0x7fff1000 pas en RAM!"                  │               │
│      │ CPU: → PAGE FAULT! Appelle le kernel!               │               │
│      └─────────────────────────────────────────────────────┘               │
│                              │                                              │
│   4. LE KERNEL charge la page depuis le disque                             │
│      ┌─────────────────────────────────────────────────────┐               │
│      │ Kernel: "Ah oui, c'est le fichier fd, page 0"       │               │
│      │ Kernel: *lit le disque, copie en RAM*               │               │
│      │ Kernel: *met à jour Page Table*                     │               │
│      │ Kernel: "Continue ton code!"                        │               │
│      └─────────────────────────────────────────────────────┘               │
│                              │                                              │
│   5. ACCÈS SUIVANTS = Direct en RAM (pas de syscall!)                      │
│      ┌─────────────────────────────────────────────────────┐               │
│      │ ptr[0], ptr[1], ptr[2]... = juste des accès mémoire │               │
│      │ AUCUN read(), AUCUNE copie, ULTRA RAPIDE            │               │
│      └─────────────────────────────────────────────────────┘               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Pourquoi c'est Plus Rapide ?

| Approche | Nombre de copies | Syscalls par accès |
|----------|------------------|-------------------|
| `read()` traditionnel | 2 (kernel buffer → user buffer) | 1 par read() |
| `mmap()` | 0 (accès direct) | 0 (après le mapping initial) |

### SECTION 2.5 : DANS LA VRAIE VIE

**Qui utilise mmap ?**

| Métier | Utilisation | Pourquoi mmap |
|--------|-------------|---------------|
| **Game Dev** | Chargement de textures/assets | Lazy loading, pas de freeze |
| **Database Engineer** | Fichiers de DB (SQLite, etc.) | Accès O(1), kernel gère le cache |
| **System Programmer** | Chargeurs d'exécutables | Le kernel mappe .text/.data |
| **Video Editor** | Fichiers vidéo volumineux | Pas de copie de 50GB en RAM |
| **Scientific Computing** | Datasets HDF5, NumPy | Accès partiel aux données |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
aperture_mmap.h  aperture_mmap.c  aperture_benchmark.c  Makefile

$ make
gcc -Wall -Wextra -std=c17 -c aperture_mmap.c -o aperture_mmap.o
gcc -Wall -Wextra -std=c17 -c aperture_benchmark.c -o aperture_benchmark.o
gcc -o aperture_test aperture_mmap.o aperture_benchmark.o

$ dd if=/dev/urandom of=test_chamber.bin bs=1M count=10
10+0 records in
10+0 records out
10485760 bytes (10 MB) copied

$ ./aperture_test test_chamber.bin
[Aperture Science] GLaDOS online. Memory mapping device ready.
[Test Chamber 01] Portal created at 0x7f1234500000 (10 MB)
[Test Chamber 01] First access triggered 2560 cube deliveries (page faults)
[Benchmark] Portal method: 2.3ms
[Benchmark] Traditional read(): 45.7ms
[Benchmark] Speedup: 19.87x faster with portals!
[Aperture Science] "The cake is a lie, but these results are real."
All portals closed. Thank you for participating in this test.
```

---

## ⚡ SECTION 3.1 : BONUS STANDARD (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×2

### 3.1.1 Consigne Bonus — Still Alive (Persistence)

**🎮 "I'm doing science and I'm still alive."**

Implémenter un système de fichiers mappés persistants avec checkpointing :

```c
/* Sauvegarder l'état de tous les portails dirty */
int aperture_checkpoint(aperture_ctx_t *ctx, const char *savefile);

/* Restaurer depuis un checkpoint */
int aperture_restore(aperture_ctx_t *ctx, const char *savefile);

/* Mapper avec Copy-on-Write automatique et journal */
void *fire_journaled_portal(aperture_ctx_t *ctx, const char *path,
                            size_t dimension, const char *journal_path);
```

### 3.1.2 Ce qui change

| Aspect | Base | Bonus |
|--------|------|-------|
| Persistence | Non | Oui (checkpoint/restore) |
| Journaling | Non | Oui (recovery possible) |
| Complexité | O(1) | O(n) pour checkpoint |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette (tableau des tests)

| # | Test | Input | Expected | Points |
|---|------|-------|----------|--------|
| 1 | `mmap_file_read` | Map fichier 4KB PROT_READ | Accès OK, contenu correct | 10 |
| 2 | `mmap_file_write` | Map fichier PROT_READ\|WRITE | Modification persiste | 10 |
| 3 | `mmap_shared_visible` | MAP_SHARED, modify, read from fd | Changement visible | 10 |
| 4 | `mmap_private_cow` | MAP_PRIVATE, modify | Original inchangé | 10 |
| 5 | `mmap_anonymous` | MAP_ANONYMOUS 8KB | Mémoire zeroed, utilisable | 10 |
| 6 | `munmap_cleanup` | Map puis unmap | Accès = SIGSEGV | 5 |
| 7 | `msync_immediate` | Modify, MS_SYNC | Données sur disque | 10 |
| 8 | `msync_async` | Modify, MS_ASYNC | Pas de freeze | 5 |
| 9 | `mprotect_none` | PROT_NONE après map | Accès = SIGSEGV | 5 |
| 10 | `mprotect_readonly` | Remove PROT_WRITE | Write = SIGSEGV | 5 |
| 11 | `madvise_sequential` | MADV_SEQUENTIAL sur 100MB | Performance OK | 5 |
| 12 | `madvise_willneed` | MADV_WILLNEED | Pre-fault pages | 5 |
| 13 | `benchmark_vs_read` | 10MB file | mmap >= 5x faster | 10 |

**Total : 100 points**

### 4.2 main.c de test

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <signal.h>
#include <setjmp.h>
#include <sys/stat.h>

#include "aperture_mmap.h"

/* Signal handling for SIGSEGV tests */
static sigjmp_buf jump_buffer;
static volatile sig_atomic_t got_signal = 0;

static void segfault_handler(int sig) {
    (void)sig;
    got_signal = 1;
    siglongjmp(jump_buffer, 1);
}

#define TEST(name) static int test_##name(void)
#define RUN_TEST(name) do { \
    printf("Test Chamber %02d [%s]: ", test_num++, #name); \
    got_signal = 0; \
    if (test_##name()) { \
        printf("SUCCESS\n"); \
        passed++; \
    } else { \
        printf("FAILED\n"); \
    } \
} while(0)

static int test_num = 1;
static int passed = 0;

/* Create test file */
static void create_test_file(const char *path, size_t size) {
    int fd = open(path, O_CREAT | O_WRONLY | O_TRUNC, 0644);
    if (fd < 0) return;

    char buf[4096];
    memset(buf, 'A', sizeof(buf));

    while (size > 0) {
        size_t to_write = size < sizeof(buf) ? size : sizeof(buf);
        write(fd, buf, to_write);
        size -= to_write;
    }
    close(fd);
}

TEST(mmap_file_read) {
    const char *path = "/tmp/aperture_test_read.bin";
    create_test_file(path, 4096);

    aperture_ctx_t *ctx = aperture_init();
    if (!ctx) return 0;

    void *portal = fire_blue_portal(ctx, path, 4096,
                                     PORTAL_SEC_READ, PORTAL_DUAL, 0);
    if (!portal) {
        aperture_shutdown(ctx);
        return 0;
    }

    /* Verify content */
    char *data = (char *)portal;
    int ok = (data[0] == 'A' && data[4095] == 'A');

    close_portal(ctx, portal, 4096);
    aperture_shutdown(ctx);
    unlink(path);
    return ok;
}

TEST(mmap_shared_visible) {
    const char *path = "/tmp/aperture_test_shared.bin";
    create_test_file(path, 4096);

    aperture_ctx_t *ctx = aperture_init();
    void *portal = fire_blue_portal(ctx, path, 4096,
                                     PORTAL_SEC_READ | PORTAL_SEC_WRITE,
                                     PORTAL_DUAL, 0);
    if (!portal) {
        aperture_shutdown(ctx);
        return 0;
    }

    /* Modify through portal */
    char *data = (char *)portal;
    data[0] = 'Z';

    /* Sync */
    sync_portals(ctx, portal, 4096, SYNC_IMMEDIATE);
    close_portal(ctx, portal, 4096);
    aperture_shutdown(ctx);

    /* Re-open and verify */
    int fd = open(path, O_RDONLY);
    char buf[1];
    read(fd, buf, 1);
    close(fd);
    unlink(path);

    return buf[0] == 'Z';
}

TEST(mmap_private_cow) {
    const char *path = "/tmp/aperture_test_cow.bin";
    create_test_file(path, 4096);

    aperture_ctx_t *ctx = aperture_init();
    void *portal = fire_blue_portal(ctx, path, 4096,
                                     PORTAL_SEC_READ | PORTAL_SEC_WRITE,
                                     PORTAL_PERSONAL, 0);  /* COW! */
    if (!portal) {
        aperture_shutdown(ctx);
        return 0;
    }

    /* Modify (should trigger COW) */
    char *data = (char *)portal;
    data[0] = 'Z';

    close_portal(ctx, portal, 4096);
    aperture_shutdown(ctx);

    /* Original should be unchanged */
    int fd = open(path, O_RDONLY);
    char buf[1];
    read(fd, buf, 1);
    close(fd);
    unlink(path);

    return buf[0] == 'A';  /* Original 'A', not 'Z' */
}

TEST(mmap_anonymous) {
    aperture_ctx_t *ctx = aperture_init();
    void *portal = fire_void_portal(ctx, 8192, PORTAL_SEC_READ | PORTAL_SEC_WRITE);
    if (!portal) {
        aperture_shutdown(ctx);
        return 0;
    }

    /* Should be zeroed */
    char *data = (char *)portal;
    int ok = (data[0] == 0 && data[8191] == 0);

    /* Should be writable */
    data[0] = 'X';
    ok = ok && (data[0] == 'X');

    close_portal(ctx, portal, 8192);
    aperture_shutdown(ctx);
    return ok;
}

TEST(mprotect_none) {
    const char *path = "/tmp/aperture_test_prot.bin";
    create_test_file(path, 4096);

    aperture_ctx_t *ctx = aperture_init();
    void *portal = fire_blue_portal(ctx, path, 4096,
                                     PORTAL_SEC_READ, PORTAL_DUAL, 0);
    if (!portal) {
        aperture_shutdown(ctx);
        return 0;
    }

    /* Make inaccessible */
    portal_security(ctx, portal, 4096, PORTAL_SEC_NONE);

    /* Setup signal handler */
    struct sigaction sa = {.sa_handler = segfault_handler};
    sigaction(SIGSEGV, &sa, NULL);

    int crashed = 0;
    if (sigsetjmp(jump_buffer, 1) == 0) {
        volatile char c = ((char *)portal)[0];  /* Should crash */
        (void)c;
    } else {
        crashed = 1;
    }

    close_portal(ctx, portal, 4096);
    aperture_shutdown(ctx);
    unlink(path);

    return crashed;
}

TEST(benchmark_performance) {
    const char *path = "/tmp/aperture_bench.bin";
    create_test_file(path, 10 * 1024 * 1024);  /* 10 MB */

    test_chamber_results_t results;
    run_test_chamber(path, &results);

    unlink(path);

    printf("(portal: %.1fms, read: %.1fms, speedup: %.1fx) ",
           results.portal_time_ms, results.traditional_time_ms,
           results.speedup_factor);

    return results.speedup_factor >= 2.0;  /* At least 2x faster */
}

int main(void) {
    printf("\n");
    printf("╔═══════════════════════════════════════════════════════════╗\n");
    printf("║        APERTURE SCIENCE MEMORY MAPPING TEST SUITE         ║\n");
    printf("║           'The cake is a lie. The portals are not.'       ║\n");
    printf("╚═══════════════════════════════════════════════════════════╝\n\n");

    RUN_TEST(mmap_file_read);
    RUN_TEST(mmap_shared_visible);
    RUN_TEST(mmap_private_cow);
    RUN_TEST(mmap_anonymous);
    RUN_TEST(mprotect_none);
    RUN_TEST(benchmark_performance);

    printf("\n═══════════════════════════════════════════════════════════\n");
    printf("Test Chambers Completed: %d/%d\n", passed, test_num - 1);
    printf("═══════════════════════════════════════════════════════════\n");

    if (passed == test_num - 1) {
        printf("\n🎂 Congratulations! The cake is real after all!\n\n");
    }

    return passed == test_num - 1 ? 0 : 1;
}
```

### 4.3 Solution de référence

```c
/* aperture_mmap.c — The Aperture Science Memory Mapping Device */
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <time.h>
#include <sys/mman.h>
#include <sys/stat.h>

#include "aperture_mmap.h"

#define INITIAL_CAPACITY 16

aperture_ctx_t *aperture_init(void)
{
    aperture_ctx_t *ctx = calloc(1, sizeof(aperture_ctx_t));
    if (!ctx)
        return NULL;

    ctx->portals = calloc(INITIAL_CAPACITY, sizeof(portal_t));
    if (!ctx->portals) {
        free(ctx);
        return NULL;
    }

    ctx->capacity = INITIAL_CAPACITY;
    ctx->glados_active = true;

    return ctx;
}

void aperture_shutdown(aperture_ctx_t *ctx)
{
    if (!ctx)
        return;

    /* Close all remaining portals */
    for (size_t i = 0; i < ctx->active_count; i++) {
        if (ctx->portals[i].aperture) {
            munmap(ctx->portals[i].aperture, ctx->portals[i].dimension);
            if (ctx->portals[i].chamber_fd >= 0)
                close(ctx->portals[i].chamber_fd);
        }
    }

    free(ctx->portals);
    free(ctx);
}

static int add_portal(aperture_ctx_t *ctx, portal_t *portal)
{
    if (ctx->active_count >= ctx->capacity) {
        size_t new_cap = ctx->capacity * 2;
        portal_t *new_portals = realloc(ctx->portals, new_cap * sizeof(portal_t));
        if (!new_portals)
            return -1;
        ctx->portals = new_portals;
        ctx->capacity = new_cap;
    }

    ctx->portals[ctx->active_count++] = *portal;
    return 0;
}

void *fire_blue_portal(aperture_ctx_t *ctx, const char *chamber_path,
                       size_t dimension, int security_level,
                       int portal_type, off_t entry_point)
{
    if (!ctx || !chamber_path)
        return NULL;

    /* Open the chamber (file) */
    int flags = O_RDONLY;
    if (security_level & PORTAL_SEC_WRITE)
        flags = O_RDWR;

    int fd = open(chamber_path, flags);
    if (fd < 0)
        return NULL;

    /* Get file size if dimension is 0 */
    if (dimension == 0) {
        struct stat st;
        if (fstat(fd, &st) < 0) {
            close(fd);
            return NULL;
        }
        dimension = st.st_size - entry_point;
    }

    /* Create the portal! */
    void *aperture = mmap(NULL, dimension, security_level, portal_type,
                          fd, entry_point);
    if (aperture == MAP_FAILED) {
        close(fd);
        return NULL;
    }

    /* Register the portal */
    portal_t portal = {
        .aperture = aperture,
        .dimension = dimension,
        .security_level = security_level,
        .portal_type = portal_type,
        .chamber_fd = fd,
        .entry_point = entry_point,
        .synchronized = true,
        .chamber_name = chamber_path
    };

    if (add_portal(ctx, &portal) < 0) {
        munmap(aperture, dimension);
        close(fd);
        return NULL;
    }

    return aperture;
}

void *fire_void_portal(aperture_ctx_t *ctx, size_t dimension,
                       int security_level)
{
    if (!ctx || dimension == 0)
        return NULL;

    /* Anonymous mapping — no file backing */
    void *aperture = mmap(NULL, dimension, security_level,
                          MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (aperture == MAP_FAILED)
        return NULL;

    portal_t portal = {
        .aperture = aperture,
        .dimension = dimension,
        .security_level = security_level,
        .portal_type = MAP_PRIVATE | MAP_ANONYMOUS,
        .chamber_fd = -1,
        .entry_point = 0,
        .synchronized = true,
        .chamber_name = "[anonymous]"
    };

    if (add_portal(ctx, &portal) < 0) {
        munmap(aperture, dimension);
        return NULL;
    }

    return aperture;
}

portal_t *find_portal(aperture_ctx_t *ctx, void *aperture)
{
    if (!ctx || !aperture)
        return NULL;

    for (size_t i = 0; i < ctx->active_count; i++) {
        if (ctx->portals[i].aperture == aperture)
            return &ctx->portals[i];
    }
    return NULL;
}

int close_portal(aperture_ctx_t *ctx, void *aperture, size_t dimension)
{
    if (!ctx || !aperture)
        return -1;

    portal_t *portal = find_portal(ctx, aperture);
    if (!portal)
        return -1;

    /* Sync if needed */
    if (!portal->synchronized && portal->chamber_fd >= 0)
        msync(aperture, dimension, MS_SYNC);

    /* Close the portal */
    if (munmap(aperture, dimension) < 0)
        return -1;

    /* Close file if applicable */
    if (portal->chamber_fd >= 0)
        close(portal->chamber_fd);

    /* Remove from list (swap with last) */
    size_t idx = portal - ctx->portals;
    ctx->portals[idx] = ctx->portals[--ctx->active_count];

    return 0;
}

int sync_portals(aperture_ctx_t *ctx, void *aperture, size_t dimension,
                 int sync_mode)
{
    if (!ctx || !aperture)
        return -1;

    portal_t *portal = find_portal(ctx, aperture);
    if (!portal)
        return -1;

    int result = msync(aperture, dimension, sync_mode);
    if (result == 0)
        portal->synchronized = true;

    return result;
}

int portal_security(aperture_ctx_t *ctx, void *aperture, size_t dimension,
                    int new_security)
{
    if (!ctx || !aperture)
        return -1;

    portal_t *portal = find_portal(ctx, aperture);
    if (!portal)
        return -1;

    int result = mprotect(aperture, dimension, new_security);
    if (result == 0)
        portal->security_level = new_security;

    return result;
}

int glados_hint(aperture_ctx_t *ctx, void *aperture, size_t dimension,
                int advice)
{
    (void)ctx;  /* Advice doesn't need context tracking */
    return madvise(aperture, dimension, advice);
}

uint64_t get_cube_deliveries(aperture_ctx_t *ctx)
{
    if (!ctx)
        return 0;
    return ctx->cube_deliveries;
}

/* Benchmark: Portal vs Traditional */
void run_test_chamber(const char *chamber_path, test_chamber_results_t *results)
{
    if (!chamber_path || !results)
        return;

    memset(results, 0, sizeof(*results));

    struct stat st;
    if (stat(chamber_path, &st) < 0)
        return;

    size_t file_size = st.st_size;
    results->cubes_processed = file_size;

    /* Test 1: Portal method (mmap) */
    struct timespec start, end;

    clock_gettime(CLOCK_MONOTONIC, &start);

    int fd = open(chamber_path, O_RDONLY);
    void *portal = mmap(NULL, file_size, PROT_READ, MAP_PRIVATE, fd, 0);

    /* Touch all pages (simulate reading) */
    volatile char sum = 0;
    char *data = (char *)portal;
    for (size_t i = 0; i < file_size; i += 4096)
        sum += data[i];
    (void)sum;

    munmap(portal, file_size);
    close(fd);

    clock_gettime(CLOCK_MONOTONIC, &end);

    results->portal_time_ms = (end.tv_sec - start.tv_sec) * 1000.0 +
                              (end.tv_nsec - start.tv_nsec) / 1000000.0;

    /* Test 2: Traditional method (read) */
    clock_gettime(CLOCK_MONOTONIC, &start);

    fd = open(chamber_path, O_RDONLY);
    char *buf = malloc(file_size);

    size_t total = 0;
    ssize_t n;
    while ((n = read(fd, buf + total, file_size - total)) > 0)
        total += n;

    /* Touch all data */
    sum = 0;
    for (size_t i = 0; i < file_size; i += 4096)
        sum += buf[i];

    free(buf);
    close(fd);

    clock_gettime(CLOCK_MONOTONIC, &end);

    results->traditional_time_ms = (end.tv_sec - start.tv_sec) * 1000.0 +
                                   (end.tv_nsec - start.tv_nsec) / 1000000.0;

    /* Calculate speedup */
    if (results->portal_time_ms > 0)
        results->speedup_factor = results->traditional_time_ms / results->portal_time_ms;

    results->lazy_verified = true;
}

void display_all_portals(aperture_ctx_t *ctx)
{
    if (!ctx) return;

    printf("\n[Aperture Science] Active Portals: %zu\n", ctx->active_count);
    printf("─────────────────────────────────────────────────────────────\n");

    for (size_t i = 0; i < ctx->active_count; i++) {
        portal_t *p = &ctx->portals[i];
        printf("Portal %zu:\n", i);
        printf("  Address:    %p\n", p->aperture);
        printf("  Dimension:  %zu bytes\n", p->dimension);
        printf("  Chamber:    %s\n", p->chamber_name);
        printf("  Security:   %c%c%c\n",
               (p->security_level & PROT_READ) ? 'R' : '-',
               (p->security_level & PROT_WRITE) ? 'W' : '-',
               (p->security_level & PROT_EXEC) ? 'X' : '-');
        printf("  Synced:     %s\n", p->synchronized ? "yes" : "no");
    }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```c
/* =============================================================================
 * Mutant A (Boundary) : Mauvaise taille de mapping
 * =============================================================================
 */
void *mutant_a_fire_blue_portal(aperture_ctx_t *ctx, const char *path,
                                 size_t dimension, int security, int type,
                                 off_t offset)
{
    int fd = open(path, O_RDWR);

    /* BUG: Utilise dimension - 1 au lieu de dimension */
    void *aperture = mmap(NULL, dimension - 1, security, type, fd, offset);

    /* Dernier byte inaccessible ! */
    return aperture;
}
/* Pourquoi c'est faux : Off-by-one, accès au dernier byte = crash */


/* =============================================================================
 * Mutant B (Safety) : Pas de vérification MAP_FAILED
 * =============================================================================
 */
void *mutant_b_fire_blue_portal(aperture_ctx_t *ctx, const char *path,
                                 size_t dimension, int security, int type,
                                 off_t offset)
{
    int fd = open(path, O_RDWR);

    void *aperture = mmap(NULL, dimension, security, type, fd, offset);

    /* BUG: Pas de vérification de MAP_FAILED ! */
    /* Si mmap échoue, aperture == MAP_FAILED == (void *)-1 */
    /* Et on retourne un pointeur invalide */

    return aperture;  /* Peut retourner MAP_FAILED ! */
}
/* Pourquoi c'est faux : MAP_FAILED n'est pas NULL, c'est (void *)-1 */


/* =============================================================================
 * Mutant C (Resource) : FD leak
 * =============================================================================
 */
int mutant_c_close_portal(aperture_ctx_t *ctx, void *aperture, size_t dim)
{
    portal_t *portal = find_portal(ctx, aperture);
    if (!portal) return -1;

    munmap(aperture, dim);

    /* BUG: On oublie de fermer le file descriptor ! */
    /* if (portal->chamber_fd >= 0) close(portal->chamber_fd); */

    /* Après 1024 portails : "Too many open files" */
    return 0;
}
/* Pourquoi c'est faux : File descriptor leak, limite atteinte rapidement */


/* =============================================================================
 * Mutant D (Logic) : Mauvais flags pour msync
 * =============================================================================
 */
int mutant_d_sync_portals(aperture_ctx_t *ctx, void *aperture,
                           size_t dimension, int sync_mode)
{
    portal_t *portal = find_portal(ctx, aperture);
    if (!portal) return -1;

    /* BUG: Utilise MS_INVALIDATE au lieu du mode demandé */
    /* MS_INVALIDATE invalide le mapping sans écrire ! */
    int result = msync(aperture, dimension, MS_INVALIDATE);

    portal->synchronized = true;  /* Mensonge! */
    return result;
}
/* Pourquoi c'est faux : Données perdues, pas écrites sur disque */


/* =============================================================================
 * Mutant E (Return) : Mauvais code de retour mprotect
 * =============================================================================
 */
int mutant_e_portal_security(aperture_ctx_t *ctx, void *aperture,
                              size_t dimension, int new_security)
{
    portal_t *portal = find_portal(ctx, aperture);
    if (!portal)
        return 0;  /* BUG: Devrait retourner -1 pour erreur */

    int result = mprotect(aperture, dimension, new_security);

    return 1;  /* BUG: Toujours 1, même si mprotect a échoué */
}
/* Pourquoi c'est faux : Caller pense que ça a marché même en cas d'échec */
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CONCEPTS MAÎTRISÉS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. Memory-Mapped I/O                                                       │
│     • mmap() crée un mapping fichier → mémoire virtuelle                    │
│     • Accès direct sans copie intermédiaire                                 │
│     • Lazy loading via page faults                                          │
│                                                                             │
│  2. Flags de Protection (PROT_*)                                            │
│     • PROT_READ : Lecture autorisée                                         │
│     • PROT_WRITE : Écriture autorisée                                       │
│     • PROT_EXEC : Exécution autorisée                                       │
│     • PROT_NONE : Aucun accès (trap!)                                       │
│                                                                             │
│  3. Flags de Mapping (MAP_*)                                                │
│     • MAP_SHARED : Modifications visibles par tous                          │
│     • MAP_PRIVATE : Copy-on-Write personnel                                 │
│     • MAP_ANONYMOUS : Pas de fichier backing                                │
│                                                                             │
│  4. Synchronisation et Optimisation                                         │
│     • msync() force l'écriture sur disque                                   │
│     • madvise() donne des hints au kernel                                   │
│     • mprotect() change les permissions dynamiquement                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 LDA — Traduction littérale en français (MAJUSCULES)

```
FONCTION fire_blue_portal QUI RETOURNE UN POINTEUR VOID ET PREND EN PARAMÈTRES
ctx QUI EST UN POINTEUR VERS aperture_ctx_t ET chamber_path QUI EST UN POINTEUR
VERS UNE CHAÎNE CONSTANTE ET dimension QUI EST UNE TAILLE ET security_level ET
portal_type QUI SONT DES ENTIERS ET entry_point QUI EST UN OFFSET
DÉBUT FONCTION
    SI ctx EST ÉGAL À NUL OU chamber_path EST ÉGAL À NUL ALORS
        RETOURNER NUL
    FIN SI

    DÉCLARER flags COMME ENTIER
    AFFECTER O_RDONLY À flags
    SI security_level CONTIENT LE BIT PORTAL_SEC_WRITE ALORS
        AFFECTER O_RDWR À flags
    FIN SI

    DÉCLARER fd COMME ENTIER
    AFFECTER OUVRIR LE FICHIER chamber_path AVEC LES FLAGS flags À fd
    SI fd EST INFÉRIEUR À 0 ALORS
        RETOURNER NUL
    FIN SI

    DÉCLARER aperture COMME POINTEUR VOID
    AFFECTER MAPPER EN MÉMOIRE dimension OCTETS AVEC security_level ET
             portal_type DEPUIS fd À L'OFFSET entry_point À aperture

    SI aperture EST ÉGAL À MAP_FAILED ALORS
        FERMER fd
        RETOURNER NUL
    FIN SI

    ENREGISTRER LE NOUVEAU PORTAIL DANS LE CONTEXTE

    RETOURNER aperture
FIN FONCTION
```

### 5.3 Visualisation ASCII

```
                          PORTAL GUN — HOW IT WORKS
═══════════════════════════════════════════════════════════════════════════════

    BEFORE mmap():                      AFTER mmap():
    ──────────────                      ─────────────

    MEMORY                              MEMORY
    ┌─────────────────┐                 ┌─────────────────┐
    │                 │                 │     ptr ────────┼────┐
    │  (empty)        │                 │                 │    │ PORTAL!
    │                 │                 │                 │    │
    └─────────────────┘                 └─────────────────┘    │
                                                               │
                                        PAGE TABLE             │
                                        ┌─────────────────┐    │
                                        │ Virtual→Physical │    │
                                        │ ptr[0] → ???    │◄───┘
                                        │ (will page fault│
                                        │  on first access)│
                                        └─────────────────┘
    FILESYSTEM                          FILESYSTEM
    ┌─────────────────┐                 ┌─────────────────┐
    │   data.bin      │                 │   data.bin      │
    │   [AAAAAAA...]  │                 │   [AAAAAAA...]  │◄─── Same file!
    └─────────────────┘                 └─────────────────┘

═══════════════════════════════════════════════════════════════════════════════

    FIRST ACCESS (ptr[0]):

    1. CPU tries to read ptr[0]
    2. Page Table says: "Not in RAM!"
    3. PAGE FAULT → Kernel takes over
    4. Kernel: "Ah, this is mapped to data.bin, page 0"
    5. Kernel reads page from disk → RAM
    6. Kernel updates Page Table
    7. CPU continues, reads ptr[0] successfully

    SUBSEQUENT ACCESSES:

    • ptr[1], ptr[2], etc. = Direct memory access
    • NO syscalls, NO copies
    • Just like reading RAM!

═══════════════════════════════════════════════════════════════════════════════

    MAP_SHARED vs MAP_PRIVATE:

    MAP_SHARED (PORTAL_DUAL):           MAP_PRIVATE (PORTAL_PERSONAL):
    ─────────────────────────           ──────────────────────────────

         Memory                              Memory
         ┌─────┐                             ┌─────┐
         │  Z  │ ◄── You write 'Z'           │  Z  │ ◄── You write 'Z'
         └──┬──┘                             └──┬──┘
            │                                   │ COPY-ON-WRITE!
            │ DIRECT!                           │
            ▼                                   ▼
         ┌─────┐                             ┌─────┐     ┌─────┐
         │  Z  │ File also has 'Z'           │  Z  │     │  A  │ File unchanged
         └─────┘                             └─────┘     └─────┘
                                              Your       Original
                                              Copy       File

═══════════════════════════════════════════════════════════════════════════════
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **MAP_FAILED check** | mmap retourne (void*)-1, pas NULL | `if (ptr == MAP_FAILED)` |
| **FD leak** | Oublier close() après unmap | Toujours fermer le fd |
| **Page alignment** | mmap requiert alignement page | Utiliser getpagesize() |
| **File size** | Mapper au-delà de EOF | Vérifier avec fstat() |
| **SIGBUS** | Accès hors fichier avec MAP_SHARED | Truncate ou vérifier taille |
| **msync oublié** | Modifications pas sur disque | MS_SYNC avant unmap |

### 5.5 Cours Complet

#### 5.5.1 Les Avantages de mmap()

1. **Zero-Copy** : Pas de copie kernel→user, les données sont directement accessibles
2. **Lazy Loading** : Les pages sont chargées uniquement quand on y accède
3. **Kernel Cache** : Le kernel gère automatiquement le cache disque
4. **Partage** : Plusieurs processus peuvent mapper le même fichier
5. **Simplicité** : Manipuler un fichier comme un simple tableau

#### 5.5.2 Quand utiliser mmap() ?

| Situation | mmap() ? | Pourquoi |
|-----------|----------|----------|
| Fichier > RAM | OUI | Lazy loading, pas tout en mémoire |
| Accès aléatoire | OUI | O(1) vs seek()+read() |
| Accès séquentiel petit | NON | read() est optimisé pour ça |
| Modification fréquente | OUI avec msync | Direct, pas de buffer |
| Lecture unique | NON | Overhead du mapping |

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais bug)                               │
├─────────────────────────────────────────────────────────────────┤
│ if (ptr == NULL) { /* handle error */ }                         │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ if (ptr == MAP_FAILED) { /* handle error */ }                   │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • MAP_FAILED = (void *)-1, PAS NULL !                           │
│ • Vérifier NULL ne détecte pas l'échec de mmap                  │
│ • C'est une erreur TRÈS commune chez les débutants              │
│ • Le code semble marcher mais crash aléatoirement               │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario : Mapper un fichier de 8KB et lire le premier et dernier octet**

```
┌───────┬──────────────────────────────────────────────┬──────────────────────────┐
│ Étape │ Action                                       │ État                     │
├───────┼──────────────────────────────────────────────┼──────────────────────────┤
│   1   │ fd = open("data.bin", O_RDONLY)              │ fd = 3                   │
├───────┼──────────────────────────────────────────────┼──────────────────────────┤
│   2   │ ptr = mmap(NULL, 8192, PROT_READ,            │ ptr = 0x7f1234000000     │
│       │           MAP_PRIVATE, fd, 0)                │ Pages: NOT_PRESENT       │
├───────┼──────────────────────────────────────────────┼──────────────────────────┤
│   3   │ c = ptr[0] // Premier accès                  │ PAGE FAULT!              │
│       │                                              │ Kernel charge page 0     │
│       │                                              │ c = 'A'                  │
├───────┼──────────────────────────────────────────────┼──────────────────────────┤
│   4   │ c = ptr[8191] // Dernier octet               │ PAGE FAULT!              │
│       │                                              │ Kernel charge page 1     │
│       │                                              │ c = 'Z'                  │
├───────┼──────────────────────────────────────────────┼──────────────────────────┤
│   5   │ c = ptr[100] // Dans page 0                  │ Pas de fault!            │
│       │                                              │ Direct RAM access        │
│       │                                              │ c = 'B' (instantané)     │
├───────┼──────────────────────────────────────────────┼──────────────────────────┤
│   6   │ munmap(ptr, 8192)                            │ Mapping supprimé         │
│       │ close(fd)                                    │ FD fermé                 │
└───────┴──────────────────────────────────────────────┴──────────────────────────┘
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🎮 MEME : Portal — "Now you're thinking with portals!"

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   🔵 BLUE PORTAL = Memory address (ptr)                         │
│   🟠 ORANGE PORTAL = File on disk                               │
│                                                                 │
│   Quand tu accèdes à ptr[0], tu passes par le portail bleu      │
│   et tu ressors par le portail orange... directement dans       │
│   le fichier !                                                  │
│                                                                 │
│   mmap()   = Tirer avec le Portal Gun (créer la connexion)      │
│   munmap() = Fermer les portails                                │
│   msync()  = S'assurer que les cubes sont bien passés           │
│                                                                 │
│   "The cake is a lie, but the portals are real."                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 🤖 MEME : GLaDOS — "The Enrichment Center reminds you..."

```
GLaDOS (le Kernel) gère tes portails :

• "The Enrichment Center reminds you that MAP_FAILED is (void*)-1,
   not NULL. The difference will kill you."

• "Thank you for triggering a page fault. Your data has been
   loaded from the hard drive. Please enjoy your access."

• "In the event of MAP_SHARED modification, please remember
   that your test partner will see everything you do."
```

### 5.9 Applications pratiques

| Application | Utilisation de mmap |
|-------------|---------------------|
| **Éditeurs de texte (vim, emacs)** | Fichiers volumineux sans tout charger |
| **Bases de données (SQLite)** | Accès direct aux pages de données |
| **Chargeurs d'exécutables** | Le kernel mappe les .text/.data |
| **JIT compilers** | mmap + PROT_EXEC pour code généré |
| **IPC (Inter-Process Comm)** | Mémoire partagée via MAP_SHARED |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          LES 8 PIÈGES mmap()                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. ❌ if (ptr == NULL) au lieu de if (ptr == MAP_FAILED)                   │
│  2. ❌ Oublier de fermer le file descriptor après munmap                    │
│  3. ❌ Mapper plus grand que le fichier (SIGBUS sur accès)                  │
│  4. ❌ Oublier msync() avant munmap (données perdues)                       │
│  5. ❌ Utiliser MAP_SHARED sur un fichier read-only                         │
│  6. ❌ Oublier l'alignement sur la taille de page                           │
│  7. ❌ Accéder après munmap (SIGSEGV)                                       │
│  8. ❌ Confondre MAP_PRIVATE et MAP_SHARED (COW vs direct)                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📝 SECTION 7 : QCM

### Question 1
**Que retourne mmap() en cas d'échec ?**
- A) NULL
- B) -1
- C) MAP_FAILED ((void *)-1)
- D) MMAP_ERROR
- E) 0
- F) errno
- G) Une adresse invalide quelconque
- H) ENOMEM
- I) void*
- J) PROT_NONE

**Réponse : C**

### Question 2
**Quelle est la différence entre MAP_SHARED et MAP_PRIVATE ?**
- A) SHARED est plus rapide
- B) PRIVATE ne peut pas écrire
- C) SHARED modifie le fichier, PRIVATE fait copy-on-write
- D) PRIVATE est thread-safe
- E) SHARED utilise plus de mémoire
- F) PRIVATE est deprecated
- G) SHARED nécessite root
- H) PRIVATE est read-only
- I) Aucune différence
- J) SHARED est synchrone

**Réponse : C**

### Question 3
**Quel syscall force l'écriture des modifications sur le disque ?**
- A) sync()
- B) fsync()
- C) msync()
- D) mflush()
- E) mwrite()
- F) munmap() le fait automatiquement
- G) fflush()
- H) fdatasync()
- I) write()
- J) flush()

**Réponse : C**

### Question 4
**Que se passe-t-il lors du premier accès à une page mmap'd ?**
- A) Les données sont déjà en RAM
- B) Un page fault charge la page depuis le disque
- C) mmap() a tout chargé au préalable
- D) SIGSEGV
- E) Le fichier est copié entièrement
- F) Rien de spécial
- G) Le kernel refuse l'accès
- H) malloc() est appelé
- I) read() est appelé
- J) Le programme freeze

**Réponse : B**

### Question 5
**Quel flag permet de mapper de la mémoire sans fichier ?**
- A) MAP_MEMORY
- B) MAP_HEAP
- C) MAP_ANONYMOUS
- D) MAP_VIRTUAL
- E) MAP_RAM
- F) MAP_NOFILE
- G) MAP_NULL
- H) MAP_VOID
- I) MAP_EMPTY
- J) MAP_MALLOC

**Réponse : C**

---

## 📊 SECTION 8 : RÉCAPITULATIF

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         EXERCICE 2.3.11 — RÉSUMÉ                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  THÈME : Portal — The Aperture Science Memory Mapping Device               │
│                                                                             │
│  CONCEPTS CLÉS :                                                            │
│  • mmap() = Créer un portail fichier↔mémoire                               │
│  • MAP_SHARED = Modifications partagées                                     │
│  • MAP_PRIVATE = Copy-on-Write                                              │
│  • MAP_ANONYMOUS = Mémoire sans fichier                                     │
│  • msync() = Forcer l'écriture                                              │
│  • mprotect() = Changer les permissions                                     │
│  • madvise() = Hints au kernel                                              │
│                                                                             │
│  PIÈGE MAJEUR :                                                             │
│  • MAP_FAILED != NULL (c'est (void *)-1 !)                                  │
│                                                                             │
│  DIFFICULTÉ : ★★★★★★☆☆☆☆ (6/10)                                            │
│  DURÉE : 5h                                                                 │
│  XP : 250 base × 2 bonus                                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.11-aperture-mmap",
    "generated_at": "2026-01-12 17:00:00",

    "metadata": {
      "exercise_id": "2.3.11",
      "exercise_name": "aperture_mmap",
      "module": "2.3",
      "module_name": "File Systems",
      "concept": "Memory-Mapped I/O",
      "concept_name": "mmap/munmap/msync",
      "type": "complet",
      "tier": 2,
      "tier_info": "Mélange concepts mmap",
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "c",
      "duration_minutes": 300,
      "xp_base": 250,
      "xp_bonus_multiplier": 2,
      "bonus_tier": "STANDARD",
      "bonus_icon": "⚡",
      "complexity_time": "T4 O(1)",
      "complexity_space": "S5 O(n)",
      "prerequisites": ["filesystem_basics", "memory_management"],
      "domains": ["FS", "Mem"],
      "tags": ["mmap", "memory-mapped", "io", "performance"],
      "meme_reference": "Portal (Valve)"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/aperture_mmap.c": "/* Section 4.3 */",
      "references/aperture_mmap.h": "/* Section 1.3 */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 */",
      "mutants/mutant_e_return.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": ["references/aperture_mmap.c"],
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

*HACKBRAIN v5.5.2 — "Now you're thinking with portals!"*
