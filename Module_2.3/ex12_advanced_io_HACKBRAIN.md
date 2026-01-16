# Exercice 2.3.12 : pitcrew_io

**Module :**
2.3 — File Systems

**Concept :**
Advanced I/O Operations — L'Art du Pit Stop Parfait

**Difficulté :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (Vectored I/O + Async I/O + Direct I/O)

**Langage :**
C (c17)

**Prérequis :**
- 2.3.0-2.3.11 (Concepts filesystem et mmap)
- File descriptors et syscalls bas niveau
- Pointeurs et buffers
- Notions de performance I/O

**Domaines :**
FS, Mem, Net

**Durée estimée :**
420 min (7h)

**XP Base :**
350

**Complexité :**
T6 O(n) × S5 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex12/
├── pitcrew_io.h
├── pitcrew_vectored.c       # readv/writev scatter-gather
├── pitcrew_zerocopy.c       # sendfile, splice, tee
├── pitcrew_async.c          # POSIX AIO + io_uring
├── pitcrew_direct.c         # O_DIRECT, O_SYNC
└── Makefile
```

**Fonctions autorisées :**
- `readv`, `writev`, `pread`, `pwrite` (sys/uio.h)
- `sendfile`, `splice`, `tee`, `copy_file_range` (sys/sendfile.h, fcntl.h)
- `aio_read`, `aio_write`, `aio_suspend`, `aio_return`, `aio_error` (aio.h)
- `io_uring_*` (liburing.h)
- `open`, `close`, `lseek`, `fstat`
- `malloc`, `free`, `posix_memalign`
- `clock_gettime`

**Fonctions interdites :**
- `read`, `write` standard (on utilise les versions avancées)
- `fopen`, `fread`, `fwrite` (pas de buffered I/O)

### 1.2 Consigne

**🏎️ FORMULA 1 — The Perfect Pit Stop**

*"In Formula 1, the difference between winning and losing is measured in milliseconds."*

Un arrêt aux stands en F1 dure environ **2 secondes**. Pendant ce temps, une équipe de
20+ mécaniciens effectue simultanément : changement des 4 pneus, ajustement de l'aileron,
nettoyage de la visière, ravitaillement (quand c'était autorisé). **TOUT EN PARALLÈLE.**

```
THE PIT STOP — ADVANCED I/O ANALOGY
═══════════════════════════════════════════════════════════════════════════════

    TRADITIONAL I/O (read/write)          ADVANCED I/O (pitcrew_io)
    ─────────────────────────────          ────────────────────────────

    ONE MECHANIC DOES EVERYTHING:          THE WHOLE CREW AT ONCE:

    1. Change front-left tire              🔧🔧🔧🔧 Change ALL 4 tires
    2. Change front-right tire               AT THE SAME TIME!
    3. Change rear-left tire               (= readv/writev scatter-gather)
    4. Change rear-right tire
    5. Refuel                              ⛽ Direct fuel line
    6. Adjust wing                         (= sendfile zero-copy)
    7. Clean visor
                                           📋 Pre-queue all operations
    ⏱️ Time: 20+ seconds                   (= io_uring batching)

                                           🏁 Slick tires = direct contact
                                           (= O_DIRECT bypass cache)

                                           ⏱️ Time: 1.8 seconds!

═══════════════════════════════════════════════════════════════════════════════
```

**Les techniques de l'équipe de stands :**

| Technique F1 | I/O Equivalent | Ce que ça fait |
|--------------|----------------|----------------|
| Multi-mécaniciens simultanés | `readv()/writev()` | Scatter-Gather : plusieurs buffers en un appel |
| Position précise du cric | `pread()/pwrite()` | I/O positionnel, pas besoin de lseek() |
| Ravitaillement direct | `sendfile()` | Zero-copy : fichier→socket sans passer par userspace |
| Système de tuyaux rapide | `splice()/tee()` | Transfert via pipe kernel |
| Queue de tâches pré-planifiées | `io_uring` | Batch d'opérations, 1 syscall pour tout |
| Pneus slicks (contact direct) | `O_DIRECT` | Bypass du page cache |

**Ta mission :**

Créer `pitcrew_io`, une bibliothèque d'I/O haute performance qui utilise TOUTES les
techniques modernes pour minimiser le temps d'arrêt aux stands (latence I/O).

### 1.2.2 Consigne Académique

Implémenter une bibliothèque d'opérations I/O avancées comprenant :
- I/O vectorisé (scatter-gather)
- I/O positionnel thread-safe
- Transferts zero-copy
- I/O asynchrone (POSIX AIO et io_uring)
- I/O direct (bypass cache)

### 1.3 Prototypes

```c
#define _GNU_SOURCE
#include <sys/uio.h>
#include <sys/sendfile.h>
#include <aio.h>
#include <liburing.h>
#include <fcntl.h>

/*
 * =============================================================================
 *                    PITCREW I/O — HIGH PERFORMANCE LIBRARY
 * =============================================================================
 *
 * "Box, box, box!" — Every F1 race engineer ever
 */

/* Requête AIO (comme une tâche pour un mécanicien) */
typedef struct {
    struct aiocb        cb;             /* POSIX AIO control block */
    int                 status;         /* État de la tâche */
    void               *user_data;      /* Données utilisateur */
    const char         *task_name;      /* Pour debug */
} pit_task_t;

/* Contexte io_uring (la queue des opérations pré-planifiées) */
typedef struct {
    struct io_uring     ring;           /* io_uring instance */
    uint32_t            sq_entries;     /* Submission queue size */
    uint32_t            cq_entries;     /* Completion queue size */
    uint64_t            ops_submitted;  /* Stats */
    uint64_t            ops_completed;
} turbo_queue_t;

/* Contexte principal — Le garage */
typedef struct {
    pit_task_t         *tasks;          /* Pool de tâches AIO */
    size_t              task_count;
    size_t              task_capacity;
    turbo_queue_t      *turbo;          /* io_uring context */
    bool                slick_mode;     /* O_DIRECT enabled */

    /* Statistiques du pit stop */
    struct {
        uint64_t        scatter_gather_ops;
        uint64_t        positional_ops;
        uint64_t        zerocopy_bytes;
        uint64_t        async_ops;
        uint64_t        direct_ops;
        double          best_lap_time_ms;
    } telemetry;
} pitcrew_ctx_t;

/*
 * =============================================================================
 *                    SCATTER-GATHER (Multi-Mechanic Mode)
 * =============================================================================
 */

/* 2.3.23.a-b: Lire dans plusieurs buffers en un seul appel */
ssize_t pitcrew_scatter_read(int fd, const struct iovec *crew, int crew_size);

/* Écrire depuis plusieurs buffers en un seul appel */
ssize_t pitcrew_gather_write(int fd, const struct iovec *crew, int crew_size);

/*
 * =============================================================================
 *                    POSITIONAL I/O (Precision Pit Work)
 * =============================================================================
 */

/* 2.3.23.c: Lire à une position exacte sans modifier le file offset */
ssize_t precision_pit_read(int fd, void *buf, size_t count, off_t position);

/* Écrire à une position exacte sans modifier le file offset */
ssize_t precision_pit_write(int fd, const void *buf, size_t count, off_t position);

/*
 * =============================================================================
 *                    ZERO-COPY TRANSFERS (Direct Refueling)
 * =============================================================================
 */

/* 2.3.23.d: Transfert fichier→fichier/socket sans copie userspace */
ssize_t refuel_direct(int out_fd, int in_fd, off_t *offset, size_t count);

/* 2.3.23.e: Transfert via pipe (splice) */
ssize_t fuel_line_splice(int fd_in, off_t *off_in, int fd_out, off_t *off_out,
                         size_t len, unsigned int flags);

/* 2.3.23.f: Dupliquer données dans un pipe (tee) */
ssize_t fuel_line_split(int fd_in, int fd_out, size_t len, unsigned int flags);

/* 2.3.23.g: Copie in-kernel (copy_file_range) */
ssize_t pitstop_copy_range(int fd_in, off_t *off_in, int fd_out,
                           off_t *off_out, size_t len, unsigned int flags);

/*
 * =============================================================================
 *                    ASYNC I/O (Pre-Planned Operations)
 * =============================================================================
 */

/* Créer le contexte pit crew */
pitcrew_ctx_t *pitcrew_garage_open(void);

/* Fermer le garage */
void pitcrew_garage_close(pitcrew_ctx_t *ctx);

/* 2.3.24.a-c: POSIX AIO — Soumettre une lecture async */
int pitcrew_async_read(pitcrew_ctx_t *ctx, int fd, void *buf, size_t count,
                       off_t offset, pit_task_t **task);

/* Soumettre une écriture async */
int pitcrew_async_write(pitcrew_ctx_t *ctx, int fd, const void *buf,
                        size_t count, off_t offset, pit_task_t **task);

/* Attendre une tâche (polling) */
int pitcrew_wait_task(pit_task_t *task);

/* Attendre avec signal */
int pitcrew_wait_signal(pit_task_t *task);

/*
 * =============================================================================
 *                    IO_URING (Turbo Mode — The Ultimate Pit System)
 * =============================================================================
 */

/* 2.3.24.g-l: Initialiser le système turbo (io_uring) */
int turbo_pit_init(pitcrew_ctx_t *ctx, uint32_t queue_depth);

/* Ajouter une lecture à la queue (pas encore exécutée!) */
int turbo_queue_read(pitcrew_ctx_t *ctx, int fd, void *buf, size_t count,
                     off_t offset);

/* Ajouter une écriture à la queue */
int turbo_queue_write(pitcrew_ctx_t *ctx, int fd, const void *buf,
                      size_t count, off_t offset);

/* SOUMETTRE toutes les opérations en une seule fois! */
int turbo_pit_submit(pitcrew_ctx_t *ctx);

/* Attendre N completions */
int turbo_pit_wait(pitcrew_ctx_t *ctx, int count);

/* Fermer le système turbo */
void turbo_pit_destroy(pitcrew_ctx_t *ctx);

/*
 * =============================================================================
 *                    DIRECT I/O (Slick Tires — No Cache Barrier)
 * =============================================================================
 */

/* 2.3.25.a: Ouvrir en mode slick (O_DIRECT — bypass cache) */
int pitcrew_open_slick(const char *path, int flags);

/* 2.3.25.b: Allouer un buffer aligné (requis pour O_DIRECT) */
void *pitcrew_alloc_aligned(size_t size, size_t alignment);

/* Libérer un buffer aligné */
void pitcrew_free_aligned(void *ptr);

/* 2.3.25.f: Ouvrir en mode O_SYNC (write-through) */
int pitcrew_open_sync(const char *path, int flags);

/* 2.3.25.g: Ouvrir en mode O_DSYNC (data sync only) */
int pitcrew_open_dsync(const char *path, int flags);

/*
 * =============================================================================
 *                    TELEMETRY (Performance Monitoring)
 * =============================================================================
 */

typedef struct {
    double      scatter_gather_mbps;    /* Vectored I/O throughput */
    double      positional_mbps;        /* pread/pwrite throughput */
    double      zerocopy_mbps;          /* sendfile throughput */
    double      aio_mbps;               /* POSIX AIO throughput */
    double      turbo_mbps;             /* io_uring throughput */
    double      slick_mbps;             /* Direct I/O throughput */
    double      standard_mbps;          /* Baseline read/write */
    double      best_improvement;       /* Fastest vs baseline */
} pit_telemetry_t;

/* Exécuter un benchmark complet */
void pitcrew_telemetry_run(pitcrew_ctx_t *ctx, const char *track_file,
                           pit_telemetry_t *results);

/* Afficher les résultats façon F1 */
void pitcrew_telemetry_display(const pit_telemetry_t *results);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Évolution des Pit Stops (I/O)

```
ÉVOLUTION DES TECHNIQUES I/O (comme les pit stops F1)
════════════════════════════════════════════════════════════════════

1970s : UN MÉCANICIEN POUR TOUT
        read() puis write() puis read()...
        Temps: 30+ secondes → Équivalent : I/O séquentiel bloquant

1990s : ÉQUIPE ORGANISÉE
        Plusieurs mécaniciens, mais coordination manuelle
        Temps: 8-10 secondes → Équivalent : Threads + select()/poll()

2000s : ÉQUIPE SYNCHRONISÉE
        Chacun sa spécialité, travail parallèle
        Temps: 3-4 secondes → Équivalent : POSIX AIO, epoll

2010s : PERFECTION TECHNIQUE
        Équipement pneumatique, chorégraphie parfaite
        Temps: 2-3 secondes → Équivalent : sendfile, splice

2020s : RECORD DU MONDE (1.82s Red Bull 2019)
        Tout pré-planifié, zéro temps perdu
        → Équivalent : io_uring (tout batché, minimal syscalls)
```

### 2.2 Pourquoi io_uring est Révolutionnaire

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AVANT io_uring vs AVEC io_uring                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   AVANT (10 reads):                   AVEC io_uring (10 reads):             │
│                                                                             │
│   syscall: read(fd1, buf1, n)         // Préparer dans userspace            │
│   ↓ context switch                    io_uring_prep_read(sqe, fd1, buf1)    │
│   syscall: read(fd2, buf2, n)         io_uring_prep_read(sqe, fd2, buf2)    │
│   ↓ context switch                    io_uring_prep_read(sqe, fd3, buf3)    │
│   syscall: read(fd3, buf3, n)         ... (plus de context switch!)         │
│   ↓ context switch                    io_uring_prep_read(sqe, fd10, buf10)  │
│   ...                                                                       │
│   syscall: read(fd10, buf10, n)       syscall: io_uring_enter() // UNE FOIS!│
│   ↓ context switch                    ↓ UN SEUL context switch              │
│                                                                             │
│   TOTAL: 10 syscalls                  TOTAL: 1 syscall                      │
│   TOTAL: 10 context switches          TOTAL: 1 context switch               │
│                                                                             │
│   C'est comme faire 10 pit stops      C'est comme UN pit stop parfait       │
│   vs un seul pit stop optimisé!       où tout est fait en parallèle         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### SECTION 2.5 : DANS LA VRAIE VIE

**Qui utilise l'Advanced I/O ?**

| Métier | Technique | Pourquoi |
|--------|-----------|----------|
| **Database Admin** | O_DIRECT + io_uring | Bypass OS cache, custom buffer pool |
| **Web Server Dev** | sendfile() | Servir des fichiers statiques sans copie |
| **Video Streaming** | splice() + tee() | Broadcasting efficace |
| **Storage Engineer** | io_uring | Maximum IOPS pour NVMe |
| **Game Dev** | mmap + O_DIRECT | Asset loading sans stutter |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
pitcrew_io.h  pitcrew_vectored.c  pitcrew_zerocopy.c  pitcrew_async.c  pitcrew_direct.c  Makefile

$ make
gcc -Wall -Wextra -std=c17 -c pitcrew_vectored.c
gcc -Wall -Wextra -std=c17 -c pitcrew_zerocopy.c
gcc -Wall -Wextra -std=c17 -c pitcrew_async.c -lrt
gcc -Wall -Wextra -std=c17 -c pitcrew_direct.c
gcc -o pitcrew_test *.o -luring -lrt -lpthread

$ dd if=/dev/urandom of=track_data.bin bs=1M count=100
100+0 records in
100+0 records out

$ ./pitcrew_test track_data.bin
╔═══════════════════════════════════════════════════════════════╗
║           PITCREW I/O — PERFORMANCE TELEMETRY                 ║
║                   "Box, box, box!"                            ║
╚═══════════════════════════════════════════════════════════════╝

[Scatter-Gather] Multi-buffer read:  1,245 MB/s ████████████░░░░
[Positional]     pread/pwrite:       1,156 MB/s ███████████░░░░░
[Zero-Copy]      sendfile:           2,890 MB/s ████████████████
[POSIX AIO]      Async operations:     892 MB/s ████████░░░░░░░░
[io_uring]       Turbo mode:         3,456 MB/s ████████████████
[Direct I/O]     Slick mode:         2,234 MB/s ██████████████░░
[Baseline]       Standard read:        567 MB/s ██████░░░░░░░░░░

🏁 FASTEST LAP: io_uring at 3,456 MB/s (6.1x faster than baseline!)
🏆 "If you no longer go for a gap that exists, you're no longer a racing driver."
```

---

## ⚡ SECTION 3.1 : BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

### 3.1.1 Consigne Bonus — DRS (Drag Reduction System)

**🏎️ Implémenter un système de batching intelligent avec prédiction**

```c
/* DRS Mode : Prédit les prochaines lectures et les pre-queue */
typedef struct {
    int         fd;
    off_t       predicted_offsets[16];
    size_t      prediction_count;
    double      hit_rate;
} drs_predictor_t;

/* Activer le DRS (prédiction activée) */
int drs_enable(pitcrew_ctx_t *ctx, drs_predictor_t *drs);

/* Lecture avec prédiction */
ssize_t drs_read(pitcrew_ctx_t *ctx, drs_predictor_t *drs,
                 int fd, void *buf, size_t count, off_t offset);
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette (tableau des tests)

| # | Test | Input | Expected | Points |
|---|------|-------|----------|--------|
| 1 | `scatter_read_basic` | 3 buffers iovec | All filled correctly | 10 |
| 2 | `gather_write_basic` | 3 buffers iovec | Written in order | 10 |
| 3 | `pread_threadsafe` | 2 threads, same fd | No race condition | 10 |
| 4 | `sendfile_copy` | 10MB file | Exact copy, zero-copy | 10 |
| 5 | `splice_pipe` | fd→pipe→fd | Data transferred | 10 |
| 6 | `tee_duplicate` | pipe data | Duplicated correctly | 5 |
| 7 | `copy_file_range` | 5MB range | In-kernel copy | 5 |
| 8 | `aio_read_complete` | Async read | Data correct | 10 |
| 9 | `aio_write_complete` | Async write | Persisted | 5 |
| 10 | `uring_batch` | 10 queued ops | All complete | 15 |
| 11 | `uring_throughput` | 1000 ops | > 100k IOPS | 5 |
| 12 | `direct_aligned` | O_DIRECT read | Works with aligned buf | 5 |
| 13 | `direct_unaligned` | O_DIRECT unaligned | Returns EINVAL | 5 |

**Total : 100 points**

### 4.2 main.c de test

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/stat.h>

#include "pitcrew_io.h"

#define TEST(name) static int test_##name(void)
#define RUN_TEST(name) do { \
    printf("Pit Stop %02d [%-20s]: ", test_num++, #name); \
    if (test_##name()) { printf("GREEN FLAG\n"); passed++; } \
    else { printf("DNF\n"); } \
} while(0)

static int test_num = 1;
static int passed = 0;

static void create_test_file(const char *path, size_t size) {
    int fd = open(path, O_CREAT | O_WRONLY | O_TRUNC, 0644);
    char buf[4096];
    memset(buf, 'R', sizeof(buf));  /* R for Racing */
    while (size > 0) {
        size_t w = size < sizeof(buf) ? size : sizeof(buf);
        write(fd, buf, w);
        size -= w;
    }
    close(fd);
}

TEST(scatter_read) {
    const char *path = "/tmp/pit_scatter.bin";
    create_test_file(path, 4096);

    int fd = open(path, O_RDONLY);
    char buf1[100], buf2[200], buf3[300];

    struct iovec iov[3] = {
        {buf1, sizeof(buf1)},
        {buf2, sizeof(buf2)},
        {buf3, sizeof(buf3)}
    };

    ssize_t n = pitcrew_scatter_read(fd, iov, 3);
    close(fd);
    unlink(path);

    return (n == 600 && buf1[0] == 'R' && buf2[0] == 'R' && buf3[0] == 'R');
}

TEST(positional_threadsafe) {
    const char *path = "/tmp/pit_positional.bin";
    create_test_file(path, 1024);

    int fd = open(path, O_RDONLY);
    char buf1[10], buf2[10];

    /* Read at different positions, should not interfere */
    precision_pit_read(fd, buf1, 10, 0);
    precision_pit_read(fd, buf2, 10, 500);

    /* File offset should be unchanged */
    off_t pos = lseek(fd, 0, SEEK_CUR);
    close(fd);
    unlink(path);

    return (pos == 0 && buf1[0] == 'R' && buf2[0] == 'R');
}

TEST(sendfile_zerocopy) {
    const char *src = "/tmp/pit_src.bin";
    const char *dst = "/tmp/pit_dst.bin";
    create_test_file(src, 1024 * 1024);  /* 1MB */

    int in_fd = open(src, O_RDONLY);
    int out_fd = open(dst, O_CREAT | O_WRONLY | O_TRUNC, 0644);

    ssize_t n = refuel_direct(out_fd, in_fd, NULL, 1024 * 1024);

    close(in_fd);
    close(out_fd);

    /* Verify sizes match */
    struct stat st;
    stat(dst, &st);
    unlink(src);
    unlink(dst);

    return (n == 1024 * 1024 && st.st_size == 1024 * 1024);
}

TEST(uring_batch) {
    const char *path = "/tmp/pit_uring.bin";
    create_test_file(path, 4096);

    pitcrew_ctx_t *ctx = pitcrew_garage_open();
    turbo_pit_init(ctx, 32);

    int fd = open(path, O_RDONLY);
    char bufs[10][100];

    /* Queue 10 reads */
    for (int i = 0; i < 10; i++) {
        turbo_queue_read(ctx, fd, bufs[i], 100, i * 100);
    }

    /* Submit all at once */
    int submitted = turbo_pit_submit(ctx);

    /* Wait for all */
    turbo_pit_wait(ctx, 10);

    turbo_pit_destroy(ctx);
    close(fd);
    pitcrew_garage_close(ctx);
    unlink(path);

    return (submitted == 10 && bufs[0][0] == 'R' && bufs[9][0] == 'R');
}

TEST(direct_io_aligned) {
    const char *path = "/tmp/pit_direct.bin";
    create_test_file(path, 4096);

    int fd = pitcrew_open_slick(path, O_RDONLY);
    if (fd < 0) {
        unlink(path);
        return 0;  /* O_DIRECT not supported */
    }

    void *aligned = pitcrew_alloc_aligned(4096, 4096);
    ssize_t n = precision_pit_read(fd, aligned, 4096, 0);

    int ok = (n == 4096 && ((char *)aligned)[0] == 'R');

    pitcrew_free_aligned(aligned);
    close(fd);
    unlink(path);

    return ok;
}

int main(void) {
    printf("\n");
    printf("╔═══════════════════════════════════════════════════════════╗\n");
    printf("║              PITCREW I/O — RACE DAY TESTS                 ║\n");
    printf("║          'Slow is smooth, smooth is fast.'               ║\n");
    printf("╚═══════════════════════════════════════════════════════════╝\n\n");

    RUN_TEST(scatter_read);
    RUN_TEST(positional_threadsafe);
    RUN_TEST(sendfile_zerocopy);
    RUN_TEST(uring_batch);
    RUN_TEST(direct_io_aligned);

    printf("\n═══════════════════════════════════════════════════════════\n");
    printf("Race Results: %d/%d tests passed\n", passed, test_num - 1);
    printf("═══════════════════════════════════════════════════════════\n");

    if (passed == test_num - 1) {
        printf("\n🏆 WINNER! All pit stops executed perfectly!\n");
        printf("   \"To finish first, you must first finish.\"\n\n");
    }

    return passed == test_num - 1 ? 0 : 1;
}
```

### 4.3 Solution de référence

```c
/* pitcrew_vectored.c — Scatter-Gather Operations */
#define _GNU_SOURCE
#include <sys/uio.h>
#include <unistd.h>
#include <errno.h>

#include "pitcrew_io.h"

ssize_t pitcrew_scatter_read(int fd, const struct iovec *crew, int crew_size)
{
    if (fd < 0 || !crew || crew_size <= 0)
        return -1;

    return readv(fd, crew, crew_size);
}

ssize_t pitcrew_gather_write(int fd, const struct iovec *crew, int crew_size)
{
    if (fd < 0 || !crew || crew_size <= 0)
        return -1;

    return writev(fd, crew, crew_size);
}

ssize_t precision_pit_read(int fd, void *buf, size_t count, off_t position)
{
    if (fd < 0 || !buf)
        return -1;

    return pread(fd, buf, count, position);
}

ssize_t precision_pit_write(int fd, const void *buf, size_t count, off_t position)
{
    if (fd < 0 || !buf)
        return -1;

    return pwrite(fd, buf, count, position);
}
```

```c
/* pitcrew_zerocopy.c — Zero-Copy Transfers */
#define _GNU_SOURCE
#include <sys/sendfile.h>
#include <fcntl.h>
#include <unistd.h>

#include "pitcrew_io.h"

ssize_t refuel_direct(int out_fd, int in_fd, off_t *offset, size_t count)
{
    if (out_fd < 0 || in_fd < 0)
        return -1;

    return sendfile(out_fd, in_fd, offset, count);
}

ssize_t fuel_line_splice(int fd_in, off_t *off_in, int fd_out, off_t *off_out,
                         size_t len, unsigned int flags)
{
    return splice(fd_in, off_in, fd_out, off_out, len, flags);
}

ssize_t fuel_line_split(int fd_in, int fd_out, size_t len, unsigned int flags)
{
    return tee(fd_in, fd_out, len, flags);
}

ssize_t pitstop_copy_range(int fd_in, off_t *off_in, int fd_out,
                           off_t *off_out, size_t len, unsigned int flags)
{
    return copy_file_range(fd_in, off_in, fd_out, off_out, len, flags);
}
```

```c
/* pitcrew_async.c — Async I/O and io_uring */
#define _GNU_SOURCE
#include <stdlib.h>
#include <string.h>
#include <aio.h>
#include <liburing.h>
#include <errno.h>

#include "pitcrew_io.h"

#define INITIAL_TASK_CAPACITY 16

pitcrew_ctx_t *pitcrew_garage_open(void)
{
    pitcrew_ctx_t *ctx = calloc(1, sizeof(pitcrew_ctx_t));
    if (!ctx)
        return NULL;

    ctx->tasks = calloc(INITIAL_TASK_CAPACITY, sizeof(pit_task_t));
    if (!ctx->tasks) {
        free(ctx);
        return NULL;
    }
    ctx->task_capacity = INITIAL_TASK_CAPACITY;

    return ctx;
}

void pitcrew_garage_close(pitcrew_ctx_t *ctx)
{
    if (!ctx)
        return;

    if (ctx->turbo) {
        io_uring_queue_exit(&ctx->turbo->ring);
        free(ctx->turbo);
    }

    free(ctx->tasks);
    free(ctx);
}

/* POSIX AIO */
int pitcrew_async_read(pitcrew_ctx_t *ctx, int fd, void *buf, size_t count,
                       off_t offset, pit_task_t **task)
{
    if (!ctx || fd < 0 || !buf || !task)
        return -1;

    /* Ensure capacity */
    if (ctx->task_count >= ctx->task_capacity) {
        size_t new_cap = ctx->task_capacity * 2;
        pit_task_t *new_tasks = realloc(ctx->tasks, new_cap * sizeof(pit_task_t));
        if (!new_tasks)
            return -1;
        ctx->tasks = new_tasks;
        ctx->task_capacity = new_cap;
    }

    pit_task_t *t = &ctx->tasks[ctx->task_count];
    memset(t, 0, sizeof(*t));

    t->cb.aio_fildes = fd;
    t->cb.aio_buf = buf;
    t->cb.aio_nbytes = count;
    t->cb.aio_offset = offset;
    t->status = 0;

    if (aio_read(&t->cb) < 0)
        return -1;

    ctx->task_count++;
    *task = t;
    ctx->telemetry.async_ops++;

    return 0;
}

int pitcrew_wait_task(pit_task_t *task)
{
    if (!task)
        return -1;

    while (aio_error(&task->cb) == EINPROGRESS)
        ;  /* Busy wait (polling) */

    task->status = aio_return(&task->cb);
    return task->status;
}

/* io_uring */
int turbo_pit_init(pitcrew_ctx_t *ctx, uint32_t queue_depth)
{
    if (!ctx)
        return -1;

    ctx->turbo = calloc(1, sizeof(turbo_queue_t));
    if (!ctx->turbo)
        return -1;

    if (io_uring_queue_init(queue_depth, &ctx->turbo->ring, 0) < 0) {
        free(ctx->turbo);
        ctx->turbo = NULL;
        return -1;
    }

    ctx->turbo->sq_entries = queue_depth;
    ctx->turbo->cq_entries = queue_depth * 2;

    return 0;
}

int turbo_queue_read(pitcrew_ctx_t *ctx, int fd, void *buf, size_t count,
                     off_t offset)
{
    if (!ctx || !ctx->turbo)
        return -1;

    struct io_uring_sqe *sqe = io_uring_get_sqe(&ctx->turbo->ring);
    if (!sqe)
        return -1;

    io_uring_prep_read(sqe, fd, buf, count, offset);

    return 0;
}

int turbo_queue_write(pitcrew_ctx_t *ctx, int fd, const void *buf,
                      size_t count, off_t offset)
{
    if (!ctx || !ctx->turbo)
        return -1;

    struct io_uring_sqe *sqe = io_uring_get_sqe(&ctx->turbo->ring);
    if (!sqe)
        return -1;

    io_uring_prep_write(sqe, fd, buf, count, offset);

    return 0;
}

int turbo_pit_submit(pitcrew_ctx_t *ctx)
{
    if (!ctx || !ctx->turbo)
        return -1;

    int submitted = io_uring_submit(&ctx->turbo->ring);
    if (submitted > 0)
        ctx->turbo->ops_submitted += submitted;

    return submitted;
}

int turbo_pit_wait(pitcrew_ctx_t *ctx, int count)
{
    if (!ctx || !ctx->turbo)
        return -1;

    struct io_uring_cqe *cqe;
    int completed = 0;

    for (int i = 0; i < count; i++) {
        if (io_uring_wait_cqe(&ctx->turbo->ring, &cqe) < 0)
            break;
        io_uring_cqe_seen(&ctx->turbo->ring, cqe);
        completed++;
    }

    ctx->turbo->ops_completed += completed;
    return completed;
}

void turbo_pit_destroy(pitcrew_ctx_t *ctx)
{
    if (ctx && ctx->turbo) {
        io_uring_queue_exit(&ctx->turbo->ring);
        free(ctx->turbo);
        ctx->turbo = NULL;
    }
}
```

```c
/* pitcrew_direct.c — Direct I/O */
#define _GNU_SOURCE
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

#include "pitcrew_io.h"

int pitcrew_open_slick(const char *path, int flags)
{
    if (!path)
        return -1;

    return open(path, flags | O_DIRECT);
}

void *pitcrew_alloc_aligned(size_t size, size_t alignment)
{
    void *ptr = NULL;
    if (posix_memalign(&ptr, alignment, size) != 0)
        return NULL;
    return ptr;
}

void pitcrew_free_aligned(void *ptr)
{
    free(ptr);
}

int pitcrew_open_sync(const char *path, int flags)
{
    return open(path, flags | O_SYNC);
}

int pitcrew_open_dsync(const char *path, int flags)
{
    return open(path, flags | O_DSYNC);
}
```

### 4.10 Solutions Mutantes (minimum 5)

```c
/* =============================================================================
 * Mutant A (Boundary) : Oublie de vérifier le retour de readv
 * =============================================================================
 */
ssize_t mutant_a_scatter_read(int fd, const struct iovec *crew, int crew_size)
{
    /* BUG: Pas de vérification des paramètres */
    /* crew_size peut être 0 ou négatif! */
    return readv(fd, crew, crew_size);
}
/* Pourquoi c'est faux : Undefined behavior avec crew_size invalide */


/* =============================================================================
 * Mutant B (Safety) : sendfile sans vérifier les fd
 * =============================================================================
 */
ssize_t mutant_b_refuel(int out_fd, int in_fd, off_t *offset, size_t count)
{
    /* BUG: Pas de vérification que out_fd est un socket ou pipe */
    /* sendfile ne marche pas avec un fd régulier en sortie sur certains systèmes */
    return sendfile(out_fd, in_fd, offset, count);
}
/* Pourquoi c'est faux : sendfile peut échouer silencieusement */


/* =============================================================================
 * Mutant C (Resource) : io_uring leak
 * =============================================================================
 */
int mutant_c_turbo_init(pitcrew_ctx_t *ctx, uint32_t depth)
{
    ctx->turbo = calloc(1, sizeof(turbo_queue_t));
    io_uring_queue_init(depth, &ctx->turbo->ring, 0);
    /* BUG: En cas d'erreur, ctx->turbo n'est pas libéré */
    /* Et si init réussit puis le programme crash, le ring n'est jamais fermé */
    return 0;
}
/* Pourquoi c'est faux : Memory leak et kernel resource leak */


/* =============================================================================
 * Mutant D (Logic) : O_DIRECT sans alignement
 * =============================================================================
 */
void *mutant_d_alloc(size_t size, size_t alignment)
{
    (void)alignment;
    /* BUG: Utilise malloc au lieu de posix_memalign */
    /* Le buffer ne sera pas aligné ! */
    return malloc(size);
}
/* Pourquoi c'est faux : O_DIRECT requiert des buffers alignés, sinon EINVAL */


/* =============================================================================
 * Mutant E (Return) : aio_error mal interprété
 * =============================================================================
 */
int mutant_e_wait_task(pit_task_t *task)
{
    int err = aio_error(&task->cb);
    /* BUG: EINPROGRESS retourné comme erreur au lieu d'attendre */
    if (err != 0)
        return -1;
    return aio_return(&task->cb);
}
/* Pourquoi c'est faux : EINPROGRESS signifie "pas encore fini", pas erreur */
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CONCEPTS MAÎTRISÉS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. Vectored I/O (Scatter-Gather)                                           │
│     • readv() : Lit dans plusieurs buffers en un appel                      │
│     • writev() : Écrit depuis plusieurs buffers                             │
│     • Réduit le nombre de syscalls                                          │
│                                                                             │
│  2. Positional I/O                                                          │
│     • pread()/pwrite() : I/O à une position précise                         │
│     • Thread-safe : pas de modification du file offset                      │
│     • Pas besoin de lseek() avant read()/write()                            │
│                                                                             │
│  3. Zero-Copy Transfers                                                     │
│     • sendfile() : Transfert kernel-to-kernel                               │
│     • splice() : Transfert via pipe                                         │
│     • Évite la copie userspace→kernel→userspace                             │
│                                                                             │
│  4. Asynchronous I/O                                                        │
│     • POSIX AIO : Standard mais limité                                      │
│     • io_uring : Moderne, batched, ultra-performant                         │
│     • Un syscall pour N opérations                                          │
│                                                                             │
│  5. Direct I/O                                                              │
│     • O_DIRECT : Bypass du page cache                                       │
│     • Requiert buffers alignés                                              │
│     • Utilisé par les bases de données                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.3 Visualisation ASCII

```
                    ADVANCED I/O — THE PIT STOP COMPARISON
═══════════════════════════════════════════════════════════════════════════════

    STANDARD I/O (4 buffers):           VECTORED I/O (4 buffers):
    ────────────────────────            ─────────────────────────

    syscall: read(fd, buf1)              syscall: readv(fd, iov[4])
    ↓ context switch                     ↓ ONE context switch
    syscall: read(fd, buf2)
    ↓ context switch                     [buf1][buf2][buf3][buf4]
    syscall: read(fd, buf3)                 ↑     ↑     ↑     ↑
    ↓ context switch                        └─────┴─────┴─────┘
    syscall: read(fd, buf4)                  Filled in ONE call!
    ↓ context switch

    4 syscalls, 4 context switches       1 syscall, 1 context switch

═══════════════════════════════════════════════════════════════════════════════

    SENDFILE() — ZERO-COPY REFUELING
    ─────────────────────────────────

    WITHOUT sendfile:                   WITH sendfile:

    [File] ─read()─→ [Kernel Buffer]    [File] ────────────────┐
                            │                                  │ DIRECT
                            │ copy                             │ in-kernel
                            ▼                                  ▼
                     [User Buffer]              [Socket/File]
                            │
                            │ copy
                            ▼
                     [Kernel Buffer]
                            │
                            ▼
                     [Socket/File]

    TWO copies, TWO context switches     ZERO copies in userspace!

═══════════════════════════════════════════════════════════════════════════════

    IO_URING — THE TURBO PIT SYSTEM
    ─────────────────────────────────

    Submission Queue (SQ)              Completion Queue (CQ)
    ┌─────────────────────┐            ┌─────────────────────┐
    │ [read fd1 buf1]     │            │ [fd1: 4096 bytes OK]│
    │ [read fd2 buf2]     │            │ [fd2: 2048 bytes OK]│
    │ [write fd3 buf3]    │────────────│ [fd3: written OK]   │
    │ [read fd4 buf4]     │  1 syscall │ [fd4: 1024 bytes OK]│
    │ [fsync fd5]         │ processes  │ [fd5: synced OK]    │
    └─────────────────────┘    ALL     └─────────────────────┘

    Kernel processes all operations, then returns all results.
    Your code continues without waiting for each one!

═══════════════════════════════════════════════════════════════════════════════
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🏎️ MEME : F1 Pit Stop — "Box, Box, Box!"

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   "BOX, BOX, BOX!" = "Queue operations, batch them, GO!"        │
│                                                                 │
│   🏎️ readv/writev = Change all 4 tires at once                 │
│   ⛽ sendfile     = Direct fuel line (no intermediate tank)    │
│   📋 io_uring    = Pre-planned pit stop (queue everything)     │
│   🏁 O_DIRECT    = Slick tires (no rain protection = no cache) │
│                                                                 │
│   "Slow is smooth, smooth is fast."                             │
│   → Prepare everything THEN execute = faster than rushing       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 💨 MEME : "If you no longer go for a gap, you're no longer a racing driver"

```
If your I/O code doesn't use:
- readv() when you have multiple buffers
- sendfile() when copying files to sockets
- io_uring when you have many operations

...you're no longer an I/O racer, you're just a spectator!
```

---

## 📊 SECTION 8 : RÉCAPITULATIF

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         EXERCICE 2.3.12 — RÉSUMÉ                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  THÈME : Formula 1 Pit Crew — High Performance I/O                         │
│                                                                             │
│  TECHNIQUES :                                                               │
│  • Scatter-Gather : readv()/writev() — Multiple buffers, one call          │
│  • Positional : pread()/pwrite() — Thread-safe, no lseek                   │
│  • Zero-Copy : sendfile(), splice(), tee() — Kernel-to-kernel              │
│  • Async : POSIX AIO, io_uring — Non-blocking, batched                     │
│  • Direct : O_DIRECT, O_SYNC — Bypass cache                                │
│                                                                             │
│  KEY INSIGHT :                                                              │
│  • io_uring : 1 syscall for N operations = ultimate performance            │
│  • sendfile : Serving files? Use it. Always.                               │
│  • O_DIRECT : Only for app-level caching (databases)                       │
│                                                                             │
│  DIFFICULTÉ : ★★★★★★★☆☆☆ (7/10)                                            │
│  DURÉE : 7h                                                                 │
│  XP : 350 base × 3 bonus (AVANCÉ)                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

*HACKBRAIN v5.5.2 — "To finish first, you must first finish."*
