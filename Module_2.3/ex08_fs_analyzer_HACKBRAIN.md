# Exercice 2.3.15-synth : three_kingdoms_fs

**Module :**
2.3 — File Systems

**Concept :**
synth — ext4 + Btrfs + ZFS (35 concepts)

**Difficulté :**
★★★★★★★★☆☆ (8/10)

**Type :**
code

**Tiers :**
3 — Synthèse (concepts 2.3.15.a-k + 2.3.16.a-l + 2.3.17.a-l)

**Langage :**
C (C17)

**Prérequis :**
- 2.3.1-2.3.14 (concepts FS de base)
- Concepts COW, journaling, caching

**Domaines :**
FS, Struct, Mem

**Durée estimée :**
300 min

**XP Base :**
500

**Complexité :**
T3 O(log n) × S3 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex08/
├── three_kingdoms.h      # API unifiée
├── wei_ext4.c            # ext4 simulation (2.3.15)
├── shu_btrfs.c           # Btrfs simulation (2.3.16)
├── wu_zfs.c              # ZFS simulation (2.3.17)
├── kingdom_compare.c     # Comparaisons
└── Makefile
```

**Fonctions autorisées :**
```
malloc, calloc, realloc, free
memcpy, memmove, memset, memcmp
strlen, strcpy, strncpy, strcmp, strncmp, strdup
snprintf, printf, fprintf
time, clock
qsort, bsearch
```

**Fonctions interdites :**
```
open, close, read, write (syscalls directs)
fork, exec
```

### 1.2 Consigne

**🏯 ROMANCE OF THE THREE KINGDOMS — La Guerre des Filesystems**

*"The empire, long divided, must unite; long united, must divide. Thus it has ever been."*
— Romance of the Three Kingdoms

En l'an 220 après l'Écriture, l'empire des fichiers est divisé en trois royaumes :

| Royaume | Filesystem | Leader | Caractéristique |
|---------|------------|--------|-----------------|
| **Wei 魏** | ext4 | Cao Cao 曹操 | Traditionnel, journal, stable |
| **Shu 蜀** | Btrfs | Liu Bei 劉備 | COW, snapshots, innovation |
| **Wu 吳** | ZFS | Sun Quan 孫權 | Pools, auto-guérison, puissance |

Chaque royaume a ses forces et faiblesses :

**Wei (ext4)** — Le Pragmatique
- Journal pour la cohérence
- Block groups pour la localité
- Extents pour les gros fichiers
- Stable et éprouvé

**Shu (Btrfs)** — L'Innovateur
- COW natif
- Snapshots instantanés
- Compression/déduplication
- RAID intégré

**Wu (ZFS)** — Le Puissant
- Pooled storage
- Self-healing
- ARC/L2ARC caching
- Tout est checksummé

**Ta mission :**

Implémenter un **analyseur des trois royaumes** qui simule et compare les caractéristiques de chaque filesystem.

### 1.2.2 Consigne Académique

Implémenter un analyseur comparative de trois systèmes de fichiers modernes :

**ext4 (2.3.15.a-k) :**
- Historique ext2→ext3→ext4
- Block groups et flex groups
- Extents et multiblock allocation
- Delayed allocation et preallocation
- Journal avec checksum

**Btrfs (2.3.16.a-l) :**
- Copy-on-Write et B-tree
- Subvolumes, snapshots, clones
- Checksums et compression
- RAID intégré et scrub/balance
- Send/receive pour backups

**ZFS (2.3.17.a-l) :**
- Pools et vdevs
- Datasets et COW
- Checksums et self-healing
- ARC/L2ARC caching
- ZIL/SLOG et déduplication

### 1.3 Prototype

```c
#ifndef THREE_KINGDOMS_H
#define THREE_KINGDOMS_H

#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>

/* ============================================================================
 * SECTION 1: Kingdom of Wei — ext4 (2.3.15)
 *
 * "I would rather betray the world than let the world betray me."
 * — Cao Cao
 * ============================================================================ */

/* ext4 version history (2.3.15.a-d) */
typedef enum {
    WEI_EXT2 = 2,    /* No journal */
    WEI_EXT3 = 3,    /* Added journaling (2.3.15.c) */
    WEI_EXT4 = 4     /* Added extents (2.3.15.d) */
} wei_version_t;

/* Wei (ext4) configuration */
typedef struct {
    wei_version_t version;
    bool journaling;              /* 2.3.15.c: ext3 feature */
    bool extents_enabled;         /* 2.3.15.d,g: ext4 feature */
    uint32_t block_groups;        /* 2.3.15.e: Number of groups */
    uint32_t flex_group_size;     /* 2.3.15.f: Flex group size */
    bool multiblock_alloc;        /* 2.3.15.h: Multiblock allocation */
    bool delayed_alloc;           /* 2.3.15.i: Delayed allocation */
    bool prealloc_enabled;        /* 2.3.15.j: Persistent prealloc */
    uint32_t journal_checksum;    /* 2.3.15.k: Journal CRC */

    /* Stats */
    size_t blocks_allocated;
    size_t extents_count;
    size_t journal_writes;
} wei_t;  /* Wei = ext4 */

/* ============================================================================
 * SECTION 2: Kingdom of Shu — Btrfs (2.3.16)
 *
 * "A true warrior fights not because he hates what is in front of him,
 *  but because he loves what is behind him."
 * — Liu Bei
 * ============================================================================ */

typedef enum {
    SHU_CHECKSUM_CRC32,
    SHU_CHECKSUM_XXHASH,
    SHU_CHECKSUM_SHA256
} shu_checksum_t;

typedef enum {
    SHU_COMPRESS_NONE,
    SHU_COMPRESS_ZLIB,
    SHU_COMPRESS_LZO,
    SHU_COMPRESS_ZSTD
} shu_compress_t;

typedef enum {
    SHU_RAID_SINGLE,
    SHU_RAID_0,
    SHU_RAID_1,
    SHU_RAID_5,
    SHU_RAID_6,
    SHU_RAID_10
} shu_raid_t;

/* Shu (Btrfs) configuration */
typedef struct {
    bool cow_enabled;             /* 2.3.16.a: COW by design */
    void *btree_root;             /* 2.3.16.b: B-tree structure */
    uint32_t subvolume_count;     /* 2.3.16.c: Subvolumes */
    uint32_t snapshot_count;      /* 2.3.16.d: Read-only snapshots */
    uint32_t clone_count;         /* 2.3.16.e: Writable clones */
    shu_checksum_t checksum_algo; /* 2.3.16.f: Checksum algorithm */
    shu_compress_t compression;   /* 2.3.16.g: Compression */
    bool dedup_enabled;           /* 2.3.16.h: Offline dedup */
    shu_raid_t raid_level;        /* 2.3.16.i: RAID level */
    uint64_t last_scrub;          /* 2.3.16.j: Last scrub time */
    bool balanced;                /* 2.3.16.k: Balanced? */

    /* Stats */
    size_t cow_writes;
    size_t scrub_errors;
    double compression_ratio;
} shu_t;  /* Shu = Btrfs */

/* ============================================================================
 * SECTION 3: Kingdom of Wu — ZFS (2.3.17)
 *
 * "The wise win before the fight, while the ignorant fight to win."
 * — Sun Quan
 * ============================================================================ */

typedef enum {
    WU_CHECKSUM_ON,
    WU_CHECKSUM_OFF,
    WU_CHECKSUM_FLETCHER4,
    WU_CHECKSUM_SHA256
} wu_checksum_t;

typedef enum {
    WU_COMPRESS_OFF,
    WU_COMPRESS_LZ4,
    WU_COMPRESS_GZIP,
    WU_COMPRESS_ZSTD
} wu_compress_t;

/* Wu (ZFS) configuration */
typedef struct {
    char pool_name[64];           /* 2.3.17.a: zpool name */
    uint32_t vdev_count;          /* 2.3.17.b: Virtual devices */
    uint32_t dataset_count;       /* 2.3.17.c: Datasets */
    bool cow_enabled;             /* 2.3.17.d: Always COW */
    wu_checksum_t checksum_algo;  /* 2.3.17.e: Checksum everything */
    bool self_healing;            /* 2.3.17.f: Auto-repair */
    size_t arc_size;              /* 2.3.17.g: ARC size */
    size_t arc_hits;              /* ARC hit count */
    size_t arc_misses;            /* ARC miss count */
    size_t l2arc_size;            /* 2.3.17.h: L2ARC size */
    bool zil_enabled;             /* 2.3.17.i: ZIL active */
    size_t zil_writes;            /* ZIL write count */
    bool slog_present;            /* 2.3.17.j: Separate log device */
    bool dedup_enabled;           /* 2.3.17.k: Online dedup */
    wu_compress_t compression;    /* 2.3.17.l: Compression */

    /* Stats */
    size_t self_heal_count;
    double dedup_ratio;
} wu_t;  /* Wu = ZFS */

/* ============================================================================
 * SECTION 4: Three Kingdoms Analyzer
 * ============================================================================ */

typedef struct {
    wei_t wei;    /* ext4 */
    shu_t shu;    /* Btrfs */
    wu_t wu;      /* ZFS */
} three_kingdoms_t;

/* ============================================================================
 * SECTION 5: Kingdom Lifecycle
 * ============================================================================ */

/**
 * Creates the three kingdoms analyzer.
 */
three_kingdoms_t *kingdoms_create(void);

/**
 * Destroys the analyzer.
 */
void kingdoms_destroy(three_kingdoms_t *tk);

/* ============================================================================
 * SECTION 6: Wei (ext4) API — 2.3.15
 * ============================================================================ */

/**
 * Shows ext4 history: ext2 → ext3 → ext4
 * Concept: 2.3.15.a
 */
void wei_show_history(void);

/**
 * Initializes Wei (ext4) with specific version.
 * Concepts: 2.3.15.b-d
 */
void wei_init(wei_t *w, wei_version_t version);

/**
 * Enables extents (ext4 feature).
 * Concept: 2.3.15.g
 */
void wei_enable_extents(wei_t *w);

/**
 * Configures block groups.
 * Concept: 2.3.15.e
 */
void wei_set_block_groups(wei_t *w, uint32_t count);

/**
 * Enables flex groups.
 * Concept: 2.3.15.f
 */
void wei_enable_flex_groups(wei_t *w, uint32_t size);

/**
 * Performs multiblock allocation.
 * Concept: 2.3.15.h
 */
int wei_multiblock_alloc(wei_t *w, uint32_t blocks);

/**
 * Enables/disables delayed allocation.
 * Concept: 2.3.15.i
 */
void wei_set_delayed_alloc(wei_t *w, bool enabled);

/**
 * Preallocates space.
 * Concept: 2.3.15.j
 */
int wei_preallocate(wei_t *w, uint64_t size);

/**
 * Computes journal checksum.
 * Concept: 2.3.15.k
 */
uint32_t wei_journal_checksum(wei_t *w);

/**
 * Prints Wei (ext4) status.
 */
void wei_print_status(wei_t *w);

/* ============================================================================
 * SECTION 7: Shu (Btrfs) API — 2.3.16
 * ============================================================================ */

/**
 * Initializes Shu (Btrfs) with COW and B-tree.
 * Concepts: 2.3.16.a-b
 */
void shu_init(shu_t *s);

/**
 * Creates a subvolume.
 * Concept: 2.3.16.c
 */
int shu_create_subvolume(shu_t *s, const char *name);

/**
 * Creates a snapshot (read-only).
 * Concept: 2.3.16.d
 */
int shu_snapshot(shu_t *s, const char *src, const char *dst);

/**
 * Creates a clone (writable snapshot).
 * Concept: 2.3.16.e
 */
int shu_clone(shu_t *s, const char *src, const char *dst);

/**
 * Sets checksum algorithm.
 * Concept: 2.3.16.f
 */
void shu_set_checksum(shu_t *s, shu_checksum_t algo);

/**
 * Sets compression algorithm.
 * Concept: 2.3.16.g
 */
void shu_set_compression(shu_t *s, shu_compress_t algo);

/**
 * Runs deduplication (offline).
 * Concept: 2.3.16.h
 */
int shu_dedup(shu_t *s);

/**
 * Sets RAID level.
 * Concept: 2.3.16.i
 */
void shu_set_raid(shu_t *s, shu_raid_t level);

/**
 * Runs scrub (integrity verification).
 * Concept: 2.3.16.j
 */
int shu_scrub(shu_t *s);

/**
 * Runs balance (redistributes data).
 * Concept: 2.3.16.k
 */
int shu_balance(shu_t *s);

/**
 * Sends snapshot to file descriptor (incremental backup).
 * Concept: 2.3.16.l
 */
int shu_send(shu_t *s, const char *snap_name, int fd);

/**
 * Receives snapshot from file descriptor.
 * Concept: 2.3.16.l
 */
int shu_receive(shu_t *s, int fd);

/**
 * Prints Shu (Btrfs) status.
 */
void shu_print_status(shu_t *s);

/* ============================================================================
 * SECTION 8: Wu (ZFS) API — 2.3.17
 * ============================================================================ */

/**
 * Creates a zpool.
 * Concept: 2.3.17.a
 */
int wu_create_pool(wu_t *w, const char *name);

/**
 * Adds a vdev to pool.
 * Concept: 2.3.17.b
 */
int wu_add_vdev(wu_t *w, const char *device);

/**
 * Creates a dataset.
 * Concept: 2.3.17.c
 */
int wu_create_dataset(wu_t *w, const char *name);

/**
 * Shows COW statistics.
 * Concept: 2.3.17.d
 */
void wu_show_cow_stats(wu_t *w);

/**
 * Verifies all checksums.
 * Concept: 2.3.17.e
 */
int wu_verify_checksums(wu_t *w);

/**
 * Attempts self-healing on corrupted block.
 * Concept: 2.3.17.f
 */
int wu_self_heal(wu_t *w, uint64_t block_id);

/**
 * Shows ARC (Adaptive Replacement Cache) stats.
 * Concept: 2.3.17.g
 */
void wu_arc_stats(wu_t *w);

/**
 * Shows L2ARC stats.
 * Concept: 2.3.17.h
 */
void wu_l2arc_stats(wu_t *w);

/**
 * Shows ZIL (ZFS Intent Log) stats.
 * Concept: 2.3.17.i
 */
void wu_zil_stats(wu_t *w);

/**
 * Adds a separate log device (SLOG).
 * Concept: 2.3.17.j
 */
int wu_add_slog(wu_t *w, const char *device);

/**
 * Gets deduplication ratio.
 * Concept: 2.3.17.k
 */
double wu_dedup_ratio(wu_t *w);

/**
 * Sets compression algorithm.
 * Concept: 2.3.17.l
 */
void wu_set_compression(wu_t *w, wu_compress_t algo);

/**
 * Prints Wu (ZFS) status.
 */
void wu_print_status(wu_t *w);

/* ============================================================================
 * SECTION 9: Kingdom Comparison — Battle!
 * ============================================================================ */

/**
 * Compares features of all three kingdoms.
 */
void kingdoms_compare_features(three_kingdoms_t *tk);

/**
 * Compares performance characteristics.
 */
void kingdoms_compare_performance(three_kingdoms_t *tk);

/**
 * Compares reliability features.
 */
void kingdoms_compare_reliability(three_kingdoms_t *tk);

/**
 * Prints epic battle summary.
 */
void kingdoms_battle(three_kingdoms_t *tk);

/**
 * Recommends kingdom based on use case.
 */
void kingdoms_recommend(const char *use_case);

#endif /* THREE_KINGDOMS_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### Pourquoi "Romance of the Three Kingdoms" ?

Ce roman classique chinois (三國演義) décrit la lutte entre trois royaumes après la chute de la dynastie Han. Chaque royaume a une philosophie différente :

| Royaume | Philosophie | Filesystem | Approche |
|---------|-------------|------------|----------|
| **Wei** | Pragmatisme | ext4 | Éprouvé, stable |
| **Shu** | Innovation | Btrfs | COW, moderne |
| **Wu** | Puissance | ZFS | Tout intégré |

### 2.5 DANS LA VRAIE VIE

| Métier | Filesystem | Pourquoi |
|--------|------------|----------|
| **SysAdmin Linux** | ext4 | Défaut, stable, compatible |
| **NAS Home** | Btrfs | Snapshots, compression |
| **Entreprise Storage** | ZFS | Fiabilité maximale |
| **Server Web** | ext4 | Performance I/O |
| **Media Server** | Btrfs/ZFS | Intégrité, RAID |
| **Base de données** | ext4+XFS | Performance, fiabilité |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
three_kingdoms.h  wei_ext4.c  shu_btrfs.c  wu_zfs.c  kingdom_compare.c  Makefile

$ make
gcc -Wall -Wextra -Werror -std=c17 -c wei_ext4.c -o wei_ext4.o
gcc -Wall -Wextra -Werror -std=c17 -c shu_btrfs.c -o shu_btrfs.o
gcc -Wall -Wextra -Werror -std=c17 -c wu_zfs.c -o wu_zfs.o
gcc -Wall -Wextra -Werror -std=c17 -c kingdom_compare.c -o kingdom_compare.o
ar rcs libkingdoms.a wei_ext4.o shu_btrfs.o wu_zfs.o kingdom_compare.o

$ make test
./kingdoms_test
=== Three Kingdoms FS Test Suite ===

[WEI/ext4 Tests]
[TEST 01] History ext2->ext3->ext4 (2.3.15.a): PASSED
[TEST 02] Block groups (2.3.15.e): PASSED
[TEST 03] Flex groups (2.3.15.f): PASSED
[TEST 04] Extents (2.3.15.g): PASSED
[TEST 05] Multiblock alloc (2.3.15.h): PASSED
[TEST 06] Delayed alloc (2.3.15.i): PASSED
[TEST 07] Preallocation (2.3.15.j): PASSED
[TEST 08] Journal checksum (2.3.15.k): PASSED

[SHU/Btrfs Tests]
[TEST 09] COW + B-tree (2.3.16.a-b): PASSED
[TEST 10] Subvolumes (2.3.16.c): PASSED
[TEST 11] Snapshots (2.3.16.d): PASSED
[TEST 12] Clones (2.3.16.e): PASSED
[TEST 13] Checksums (2.3.16.f): PASSED
[TEST 14] Compression (2.3.16.g): PASSED
[TEST 15] Dedup (2.3.16.h): PASSED
[TEST 16] RAID (2.3.16.i): PASSED
[TEST 17] Scrub (2.3.16.j): PASSED
[TEST 18] Balance (2.3.16.k): PASSED
[TEST 19] Send/Receive (2.3.16.l): PASSED

[WU/ZFS Tests]
[TEST 20] Pools (2.3.17.a): PASSED
[TEST 21] Vdevs (2.3.17.b): PASSED
[TEST 22] Datasets (2.3.17.c): PASSED
[TEST 23] COW (2.3.17.d): PASSED
[TEST 24] Checksums (2.3.17.e): PASSED
[TEST 25] Self-healing (2.3.17.f): PASSED
[TEST 26] ARC (2.3.17.g): PASSED
[TEST 27] L2ARC (2.3.17.h): PASSED
[TEST 28] ZIL (2.3.17.i): PASSED
[TEST 29] SLOG (2.3.17.j): PASSED
[TEST 30] Dedup (2.3.17.k): PASSED
[TEST 31] Compression (2.3.17.l): PASSED

[Comparison Tests]
[TEST 32] Feature comparison: PASSED
[TEST 33] Performance comparison: PASSED
[TEST 34] Reliability comparison: PASSED

34/34 tests passed!
Wei: 30/30 | Shu: 35/35 | Wu: 35/35
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| # | Test | Concepts | Points |
|---|------|----------|--------|
| 01 | ext4 history | 2.3.15.a-d | 3 |
| 02 | Block groups | 2.3.15.e | 3 |
| 03 | Flex groups | 2.3.15.f | 3 |
| 04 | Extents | 2.3.15.g | 3 |
| 05 | Multiblock alloc | 2.3.15.h | 3 |
| 06 | Delayed alloc | 2.3.15.i | 3 |
| 07 | Preallocation | 2.3.15.j | 3 |
| 08 | Journal checksum | 2.3.15.k | 3 |
| 09 | COW + B-tree | 2.3.16.a-b | 3 |
| 10 | Subvolumes | 2.3.16.c | 3 |
| 11 | Snapshots | 2.3.16.d | 3 |
| 12 | Clones | 2.3.16.e | 3 |
| 13 | Checksums | 2.3.16.f | 3 |
| 14 | Compression | 2.3.16.g | 3 |
| 15 | Dedup | 2.3.16.h | 3 |
| 16 | RAID | 2.3.16.i | 3 |
| 17 | Scrub | 2.3.16.j | 3 |
| 18 | Balance | 2.3.16.k | 3 |
| 19 | Send/Receive | 2.3.16.l | 3 |
| 20 | Pools | 2.3.17.a | 3 |
| 21 | Vdevs | 2.3.17.b | 3 |
| 22 | Datasets | 2.3.17.c | 3 |
| 23 | COW | 2.3.17.d | 3 |
| 24 | Checksums | 2.3.17.e | 3 |
| 25 | Self-healing | 2.3.17.f | 3 |
| 26 | ARC | 2.3.17.g | 3 |
| 27 | L2ARC | 2.3.17.h | 2 |
| 28 | ZIL | 2.3.17.i | 2 |
| 29 | SLOG | 2.3.17.j | 2 |
| 30 | Dedup | 2.3.17.k | 2 |
| 31 | Compression | 2.3.17.l | 2 |
| 32 | Feature compare | - | 2 |
| 33 | Performance compare | - | 2 |
| 34 | Reliability compare | - | 2 |

**Total : 100 points**
**Minimum requis : 80/100**

### 4.3 Solution de référence

```c
/* wei_ext4.c - Kingdom of Wei (ext4) implementation */
#include "three_kingdoms.h"
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void wei_show_history(void)
{
    printf("=== History of Wei (ext4) ===\n");
    printf("\n");
    printf("ext2 (1993) - The Foundation\n");
    printf("  - Second Extended Filesystem\n");
    printf("  - Block groups for locality\n");
    printf("  - No journaling (fsck required after crash)\n");
    printf("\n");
    printf("ext3 (2001) - The Journal\n");
    printf("  - Added journaling (2.3.15.c)\n");
    printf("  - Three journal modes: writeback, ordered, journal\n");
    printf("  - Backward compatible with ext2\n");
    printf("\n");
    printf("ext4 (2008) - The Extents\n");
    printf("  - Extents replace block pointers (2.3.15.g)\n");
    printf("  - Files up to 16 TB\n");
    printf("  - Flex groups (2.3.15.f)\n");
    printf("  - Delayed allocation (2.3.15.i)\n");
    printf("  - Journal checksums (2.3.15.k)\n");
}

void wei_init(wei_t *w, wei_version_t version)
{
    if (!w)
        return;

    memset(w, 0, sizeof(*w));
    w->version = version;

    switch (version) {
        case WEI_EXT2:
            w->journaling = false;
            w->extents_enabled = false;
            break;
        case WEI_EXT3:
            w->journaling = true;
            w->extents_enabled = false;
            break;
        case WEI_EXT4:
            w->journaling = true;
            w->extents_enabled = true;
            w->multiblock_alloc = true;
            w->delayed_alloc = true;
            break;
    }
}

void wei_enable_extents(wei_t *w)
{
    if (!w || w->version < WEI_EXT4)
        return;
    w->extents_enabled = true;
}

void wei_set_block_groups(wei_t *w, uint32_t count)
{
    if (!w)
        return;
    w->block_groups = count;
}

void wei_enable_flex_groups(wei_t *w, uint32_t size)
{
    if (!w || w->version < WEI_EXT4)
        return;
    w->flex_group_size = size;
}

int wei_multiblock_alloc(wei_t *w, uint32_t blocks)
{
    if (!w || !w->multiblock_alloc)
        return -1;
    w->blocks_allocated += blocks;
    return 0;
}

void wei_set_delayed_alloc(wei_t *w, bool enabled)
{
    if (!w || w->version < WEI_EXT4)
        return;
    w->delayed_alloc = enabled;
}

int wei_preallocate(wei_t *w, uint64_t size)
{
    if (!w || !w->prealloc_enabled)
        return -1;
    w->blocks_allocated += size / 4096;
    return 0;
}

static uint32_t crc32_simple(const void *data, size_t len)
{
    uint32_t crc = 0xFFFFFFFF;
    const uint8_t *p = data;
    for (size_t i = 0; i < len; i++) {
        crc ^= p[i];
        for (int j = 0; j < 8; j++) {
            crc = (crc >> 1) ^ (0xEDB88320 & -(crc & 1));
        }
    }
    return ~crc;
}

uint32_t wei_journal_checksum(wei_t *w)
{
    if (!w || !w->journaling)
        return 0;

    w->journal_checksum = crc32_simple(w, sizeof(*w));
    return w->journal_checksum;
}

void wei_print_status(wei_t *w)
{
    if (!w)
        return;

    const char *version_names[] = {"", "", "ext2", "ext3", "ext4"};
    printf("=== Wei (ext4) Status ===\n");
    printf("Version: %s\n", version_names[w->version]);
    printf("Journaling: %s\n", w->journaling ? "enabled" : "disabled");
    printf("Extents: %s\n", w->extents_enabled ? "enabled" : "disabled");
    printf("Block groups: %u\n", w->block_groups);
    printf("Flex group size: %u\n", w->flex_group_size);
    printf("Delayed alloc: %s\n", w->delayed_alloc ? "enabled" : "disabled");
    printf("Blocks allocated: %zu\n", w->blocks_allocated);
}

/* shu_btrfs.c - Kingdom of Shu (Btrfs) implementation */

void shu_init(shu_t *s)
{
    if (!s)
        return;

    memset(s, 0, sizeof(*s));
    s->cow_enabled = true;  /* 2.3.16.a: COW by design */
    s->checksum_algo = SHU_CHECKSUM_CRC32;
    s->compression = SHU_COMPRESS_NONE;
    s->raid_level = SHU_RAID_SINGLE;
}

int shu_create_subvolume(shu_t *s, const char *name)
{
    if (!s || !name)
        return -1;
    s->subvolume_count++;
    return 0;
}

int shu_snapshot(shu_t *s, const char *src, const char *dst)
{
    if (!s || !src || !dst)
        return -1;
    s->snapshot_count++;  /* 2.3.16.d: Read-only snapshot */
    return 0;
}

int shu_clone(shu_t *s, const char *src, const char *dst)
{
    if (!s || !src || !dst)
        return -1;
    s->clone_count++;  /* 2.3.16.e: Writable clone */
    return 0;
}

void shu_set_checksum(shu_t *s, shu_checksum_t algo)
{
    if (!s)
        return;
    s->checksum_algo = algo;
}

void shu_set_compression(shu_t *s, shu_compress_t algo)
{
    if (!s)
        return;
    s->compression = algo;
    if (algo != SHU_COMPRESS_NONE) {
        s->compression_ratio = 2.5;  /* Simulated */
    }
}

int shu_dedup(shu_t *s)
{
    if (!s)
        return -1;
    s->dedup_enabled = true;
    return 0;
}

void shu_set_raid(shu_t *s, shu_raid_t level)
{
    if (!s)
        return;
    s->raid_level = level;
}

int shu_scrub(shu_t *s)
{
    if (!s)
        return -1;
    s->last_scrub = time(NULL);
    s->scrub_errors = 0;  /* Simulated: no errors */
    return 0;
}

int shu_balance(shu_t *s)
{
    if (!s)
        return -1;
    s->balanced = true;
    return 0;
}

int shu_send(shu_t *s, const char *snap_name, int fd)
{
    if (!s || !snap_name || fd < 0)
        return -1;
    /* Simulate send */
    return 0;
}

int shu_receive(shu_t *s, int fd)
{
    if (!s || fd < 0)
        return -1;
    /* Simulate receive */
    s->snapshot_count++;
    return 0;
}

void shu_print_status(shu_t *s)
{
    if (!s)
        return;

    printf("=== Shu (Btrfs) Status ===\n");
    printf("COW: %s\n", s->cow_enabled ? "enabled" : "disabled");
    printf("Subvolumes: %u\n", s->subvolume_count);
    printf("Snapshots: %u\n", s->snapshot_count);
    printf("Clones: %u\n", s->clone_count);
    printf("RAID level: %d\n", s->raid_level);
    printf("Compression ratio: %.2f\n", s->compression_ratio);
    printf("Balanced: %s\n", s->balanced ? "yes" : "no");
}

/* wu_zfs.c - Kingdom of Wu (ZFS) implementation */

int wu_create_pool(wu_t *w, const char *name)
{
    if (!w || !name)
        return -1;

    strncpy(w->pool_name, name, sizeof(w->pool_name) - 1);
    w->cow_enabled = true;  /* 2.3.17.d: Always COW */
    w->checksum_algo = WU_CHECKSUM_ON;
    w->self_healing = true;
    w->zil_enabled = true;
    return 0;
}

int wu_add_vdev(wu_t *w, const char *device)
{
    if (!w || !device)
        return -1;
    w->vdev_count++;
    return 0;
}

int wu_create_dataset(wu_t *w, const char *name)
{
    if (!w || !name)
        return -1;
    w->dataset_count++;
    return 0;
}

void wu_show_cow_stats(wu_t *w)
{
    if (!w)
        return;
    printf("ZFS COW: Always enabled (by design)\n");
}

int wu_verify_checksums(wu_t *w)
{
    if (!w)
        return -1;
    /* Simulate verification */
    return 0;  /* No errors */
}

int wu_self_heal(wu_t *w, uint64_t block_id)
{
    if (!w || !w->self_healing)
        return -1;
    w->self_heal_count++;
    return 0;
}

void wu_arc_stats(wu_t *w)
{
    if (!w)
        return;
    double hit_ratio = 0.0;
    if (w->arc_hits + w->arc_misses > 0) {
        hit_ratio = (double)w->arc_hits / (w->arc_hits + w->arc_misses) * 100;
    }
    printf("ARC Size: %zu MB\n", w->arc_size / (1024 * 1024));
    printf("ARC Hit Ratio: %.1f%%\n", hit_ratio);
}

void wu_l2arc_stats(wu_t *w)
{
    if (!w)
        return;
    printf("L2ARC Size: %zu MB\n", w->l2arc_size / (1024 * 1024));
}

void wu_zil_stats(wu_t *w)
{
    if (!w)
        return;
    printf("ZIL: %s\n", w->zil_enabled ? "enabled" : "disabled");
    printf("ZIL Writes: %zu\n", w->zil_writes);
}

int wu_add_slog(wu_t *w, const char *device)
{
    if (!w || !device)
        return -1;
    w->slog_present = true;
    return 0;
}

double wu_dedup_ratio(wu_t *w)
{
    if (!w || !w->dedup_enabled)
        return 1.0;
    return w->dedup_ratio;
}

void wu_set_compression(wu_t *w, wu_compress_t algo)
{
    if (!w)
        return;
    w->compression = algo;
}

void wu_print_status(wu_t *w)
{
    if (!w)
        return;

    printf("=== Wu (ZFS) Status ===\n");
    printf("Pool: %s\n", w->pool_name);
    printf("Vdevs: %u\n", w->vdev_count);
    printf("Datasets: %u\n", w->dataset_count);
    printf("COW: always enabled\n");
    printf("Self-healing: %s\n", w->self_healing ? "enabled" : "disabled");
    printf("ARC size: %zu MB\n", w->arc_size / (1024 * 1024));
    printf("ZIL: %s\n", w->zil_enabled ? "enabled" : "disabled");
    printf("SLOG: %s\n", w->slog_present ? "present" : "none");
}

/* kingdom_compare.c - Comparison functions */

three_kingdoms_t *kingdoms_create(void)
{
    three_kingdoms_t *tk = calloc(1, sizeof(*tk));
    if (!tk)
        return NULL;

    wei_init(&tk->wei, WEI_EXT4);
    shu_init(&tk->shu);
    wu_create_pool(&tk->wu, "tank");

    return tk;
}

void kingdoms_destroy(three_kingdoms_t *tk)
{
    free(tk);
}

void kingdoms_compare_features(three_kingdoms_t *tk)
{
    printf("\n");
    printf("+------------------+--------+--------+--------+\n");
    printf("| Feature          |  Wei   |  Shu   |   Wu   |\n");
    printf("|                  | (ext4) |(Btrfs) | (ZFS)  |\n");
    printf("+------------------+--------+--------+--------+\n");
    printf("| Journaling       |   Yes  |   No   |  ZIL   |\n");
    printf("| COW              |   No   |   Yes  |   Yes  |\n");
    printf("| Snapshots        |   No   |   Yes  |   Yes  |\n");
    printf("| Checksums        | Journal|  All   |   All  |\n");
    printf("| Compression      |   No   |   Yes  |   Yes  |\n");
    printf("| Deduplication    |   No   | Offline| Online |\n");
    printf("| RAID             | mdraid | Built-in| Built-in|\n");
    printf("| Self-healing     |   No   |  With  |   Yes  |\n");
    printf("|                  |        |  RAID  |        |\n");
    printf("+------------------+--------+--------+--------+\n");
}

void kingdoms_battle(three_kingdoms_t *tk)
{
    printf("\n");
    printf("=================================================\n");
    printf("     THE BATTLE OF THE THREE KINGDOMS           \n");
    printf("=================================================\n");
    printf("\n");
    printf("  Wei (ext4)  vs  Shu (Btrfs)  vs  Wu (ZFS)\n");
    printf("    曹操           劉備            孫權\n");
    printf("\n");
    printf("Each kingdom has its strengths:\n");
    printf("\n");
    printf("Wei (ext4): 'The Proven Path'\n");
    printf("  + Most compatible\n");
    printf("  + Excellent performance\n");
    printf("  + Battle-tested for decades\n");
    printf("  - No COW, no snapshots\n");
    printf("\n");
    printf("Shu (Btrfs): 'The Innovator'\n");
    printf("  + Modern COW design\n");
    printf("  + Snapshots and clones\n");
    printf("  + Compression and dedup\n");
    printf("  - RAID5/6 still maturing\n");
    printf("\n");
    printf("Wu (ZFS): 'The Fortress'\n");
    printf("  + Unmatched reliability\n");
    printf("  + Self-healing\n");
    printf("  + Powerful caching (ARC)\n");
    printf("  - High memory usage\n");
    printf("\n");
}
```

### 4.10 Solutions Mutantes

```c
/* Mutant A (Safety): NULL check manquant */
void mutant_a_wei_init(wei_t *w, wei_version_t version)
{
    /* BUG: Pas de check NULL */
    w->version = version;  /* Crash si w == NULL */
}

/* Mutant B (Logic): COW désactivé pour Btrfs */
void mutant_b_shu_init(shu_t *s)
{
    memset(s, 0, sizeof(*s));
    s->cow_enabled = false;  /* BUG: Btrfs DOIT être COW */
}

/* Mutant C (Resource): Pas de free dans destroy */
void mutant_c_kingdoms_destroy(three_kingdoms_t *tk)
{
    /* BUG: Oublie de free tk */
    (void)tk;  /* Memory leak! */
}

/* Mutant D (Logic): ext2 avec journaling */
void mutant_d_wei_init(wei_t *w, wei_version_t version)
{
    if (!w) return;
    w->version = version;
    w->journaling = true;  /* BUG: ext2 n'a PAS de journal! */
}

/* Mutant E (Return): Retourne toujours succès */
int mutant_e_wu_self_heal(wu_t *w, uint64_t block_id)
{
    /* BUG: Ne vérifie pas si self_healing est activé */
    return 0;  /* Toujours OK */
}

/* Mutant F (Boundary): Overflow pool_name */
int mutant_f_wu_create_pool(wu_t *w, const char *name)
{
    if (!w || !name) return -1;
    /* BUG: strcpy sans limite! */
    strcpy(w->pool_name, name);  /* Buffer overflow si name > 64 */
    return 0;
}
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

Cet exercice couvre 35 concepts répartis sur trois filesystems modernes majeurs :

| Filesystem | Concepts | Forces |
|------------|----------|--------|
| **ext4** | 11 (2.3.15.a-k) | Stabilité, performance |
| **Btrfs** | 12 (2.3.16.a-l) | COW, snapshots |
| **ZFS** | 12 (2.3.17.a-l) | Fiabilité, intégrité |

### 5.3 Visualisation ASCII

```
                    THREE KINGDOMS COMPARISON

    Wei (ext4)              Shu (Btrfs)             Wu (ZFS)
    ┌─────────┐            ┌─────────┐            ┌─────────┐
    │ Journal │            │  COW    │            │ Pools   │
    │   ↓     │            │  ↓      │            │   ↓     │
    │ Block   │            │ B-tree  │            │ Vdevs   │
    │ Groups  │            │   ↓     │            │   ↓     │
    │   ↓     │            │ Subvol  │            │Datasets │
    │ Extents │            │   ↓     │            │   ↓     │
    │   ↓     │            │Snapshot │            │   ARC   │
    │ Delayed │            │   ↓     │            │   ↓     │
    │ Alloc   │            │ Clone   │            │ L2ARC   │
    └─────────┘            └─────────┘            └─────────┘

    Simple & Fast          Flexible & Modern       Powerful & Reliable
```

### 5.8 Mnémotechniques

#### 🏯 MEME : "Three Kingdoms" — Choisir son camp

```
Wei (ext4) = Cao Cao = "The Proven Warrior"
  → Stable, testé, traditionnel
  → "Je préfère trahir le monde que de laisser le monde me trahir"
  → ext4 trahit les nouvelles features pour la stabilité

Shu (Btrfs) = Liu Bei = "The Innovative Leader"
  → COW, moderne, flexible
  → "La vertu ne doit jamais être seule"
  → Btrfs combine COW, snapshots, compression

Wu (ZFS) = Sun Quan = "The Powerful Admiral"
  → Pools, self-healing, forteresse
  → "Le sage gagne avant le combat"
  → ZFS prévient les corruptions avant qu'elles n'arrivent
```

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Aspect | Valeur |
|--------|--------|
| **Concepts couverts** | 35 (ext4: 11, Btrfs: 12, ZFS: 12) |
| **Difficulté** | 8/10 |
| **Durée estimée** | 300 minutes |
| **XP Base** | 500 |
| **Fichiers à rendre** | 5 |
| **Tests moulinette** | 34 |
| **Score minimum** | 80/100 |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.15-synth-three-kingdoms-fs",
    "generated_at": "2026-01-11 16:00:00",

    "metadata": {
      "exercise_id": "2.3.15-synth",
      "exercise_name": "three_kingdoms_fs",
      "module": "2.3",
      "module_name": "File Systems",
      "concept": "synth",
      "concept_name": "ext4 + Btrfs + ZFS",
      "type": "code",
      "tier": 3,
      "phase": 2,
      "difficulty": 8,
      "difficulty_stars": "★★★★★★★★☆☆",
      "language": "c",
      "duration_minutes": 300,
      "xp_base": 500,
      "domains": ["FS", "Struct", "Mem"],
      "tags": ["ext4", "btrfs", "zfs", "filesystem", "comparison"],
      "meme_reference": "Romance of the Three Kingdoms"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "Romance of the Three Kingdoms"*
*Wei vs Shu vs Wu — The Battle of Filesystems*
*"The empire, long divided, must unite."*
