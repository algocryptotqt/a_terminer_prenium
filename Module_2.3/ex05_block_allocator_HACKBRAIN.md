# Exercice 2.3.5 : overworld_block_manager

**Module :**
2.3 — File Systems

**Concept :**
2.3.8-2.3.11 — Block Allocation, Free Space Management, FS Layout, Superblock

**Difficulté :**
★★★★★★★☆☆☆ (7/10)

**Type :**
code

**Tiers :**
3 — Synthèse (37 concepts des sections 2.3.8 à 2.3.11)

**Langage :**
C (C17)

**Prérequis :**
- Gestion mémoire (malloc/free)
- Opérations binaires (bitwise)
- Structures de données (listes chaînées)

**Domaines :**
FS, Mem, Struct

**Durée estimée :**
480 min (8 heures)

**XP Base :**
350

**Complexité :**
T3 O(n) pour recherche bitmap × S2 O(n) pour structures

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
- `overworld.h` — Header avec toutes les définitions
- `overworld.c` — Création/destruction du monde
- `chunk_map.c` — Bitmap d'allocation (chunk_map)
- `rail_network.c` — FAT (File Allocation Table)
- `chest_system.c` — Blocs directs/indirects (inode)
- `biome_layout.c` — Layout du filesystem
- `Makefile`

**Fonctions autorisées :**
`malloc`, `free`, `calloc`, `memset`, `memcpy`, `printf`, `snprintf`

**Fonctions interdites :**
- Toute fonction de fichier (open, read, write, etc.)
- `system`, `exec*`

### 1.2 Consigne

#### 1.2.1 Contexte Culturel — Minecraft: Building Blocks of Storage

**⛏️ BIENVENUE DANS L'OVERWORLD — "Time to mine and craft!"**

Tu es le développeur principal de Mojang Studios, chargé d'implémenter le **système de gestion des blocs** de Minecraft. Dans ce monde cubique, tout est composé de blocs de 16x16x16 (un chunk), et tu dois gérer leur allocation, leur libération et leur organisation.

**Les concepts Minecraft mappés au filesystem:**

| Minecraft | Filesystem | Description |
|-----------|------------|-------------|
| **Block** | Block | Unité de stockage de base (1KB, 4KB) |
| **Chunk** | Block Group | Groupe de 16x16x16 blocs |
| **Chunk Map** | Bitmap | Carte des chunks chargés/libres |
| **Rail Network** | FAT | Réseau de rails liant les blocs |
| **Hotbar** | Direct blocks | 9 emplacements d'accès rapide |
| **Chest** | Indirect block | 27 emplacements (1 niveau) |
| **Ender Chest** | Double indirect | Accès partagé, 2 niveaux |
| **Shulker Box** | Triple indirect | Conteneur dans conteneur, 3 niveaux |
| **World Spawn** | Superblock | Point d'origine avec métadonnées |
| **Overworld** | Filesystem | Le monde complet |

**Stratégies d'allocation:**
- **Build Mode** (Contiguous): Construire des structures en ligne
- **Rail Chain** (Linked): Réseau de minecarts liant les blocs
- **Chest System** (Indexed): Inventaire avec pointeurs vers les coffres

#### 1.2.2 Énoncé Académique

Implémenter un simulateur d'allocation de blocs complet supportant:
1. **Bitmap** pour la gestion d'espace libre
2. **Allocation contiguë** pour les structures
3. **Allocation chaînée** via FAT (File Allocation Table)
4. **Allocation indexée** avec blocs directs/indirects (1, 2, 3 niveaux)
5. **Layout du filesystem** avec superblock, inode table, data blocks

**Ta mission :**

Créer la bibliothèque `overworld` qui simule la gestion de blocs d'un filesystem complet.

**Entrée :**
- `block_count` : Nombre total de blocs dans le monde
- `block_size` : Taille d'un bloc (1024, 2048, 4096)
- `n` : Nombre de blocs à allouer
- `logical_block` : Numéro de bloc logique à résoudre

**Sortie :**
- `0` : Succès
- `-1` : Erreur (espace insuffisant, bloc invalide, etc.)
- Structures remplies avec les informations d'allocation

**Contraintes :**
- Valgrind clean (aucune fuite mémoire)
- Maximum 40 lignes par fonction
- Gérer tous les edge cases (NULL, overflow, etc.)

### 1.3 Prototype

```c
/* overworld.h */

#ifndef OVERWORLD_H
#define OVERWORLD_H

#include <stdint.h>
#include <stddef.h>

/* ═══════════════════════════════════════════════════════════════════════════
 * CONSTANTES DU MONDE
 * ═══════════════════════════════════════════════════════════════════════════ */

#define OVERWORLD_MAGIC     0x4D494E45  /* "MINE" en ASCII */
#define CHUNK_SIZE          16          /* Blocs par chunk (une dimension) */
#define HOTBAR_SLOTS        9           /* Blocs directs (comme la hotbar) */
#define CHEST_SLOTS         27          /* Pointeurs par bloc indirect */
#define RAIL_END            0xFFFFFFFF  /* Fin de chaîne rail */
#define RAIL_FREE           0xFFFFFFFE  /* Bloc libre dans FAT */

/* États du monde */
typedef enum {
    WORLD_CLEAN     = 0x00,  /* Monde sauvegardé proprement */
    WORLD_DIRTY     = 0x01,  /* Modifications non sauvegardées */
    WORLD_CORRUPTED = 0xFF   /* Corruption détectée */
} world_state_t;

/* Comportement sur erreur */
typedef enum {
    ON_ERROR_CONTINUE   = 0,  /* Continuer (risqué) */
    ON_ERROR_READONLY   = 1,  /* Passer en lecture seule */
    ON_ERROR_PANIC      = 2   /* Arrêter immédiatement */
} error_behavior_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * SECTION 2.3.11: WORLD SPAWN (Superblock)
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    uint32_t magic;              /* a: Magic number 0x4D494E45 "MINE" */
    uint32_t block_size;         /* b: Taille d'un bloc */
    uint32_t block_count;        /* c: Nombre total de blocs */
    uint32_t chest_count;        /* d: Nombre d'inodes (coffres) */
    uint32_t free_blocks;        /* e: Blocs disponibles */
    uint32_t free_chests;        /* f: Coffres disponibles */
    uint32_t first_data_block;   /* g: Premier bloc de données */
    uint32_t load_count;         /* h: Compteur de chargements */
    uint8_t  state;              /* i: État du monde */
    uint8_t  error_behavior;     /* j: Comportement sur erreur */
    uint32_t seed;               /* Seed du monde (bonus) */
} world_spawn_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * SECTION 2.3.9: CHUNK MAP (Bitmap Free Space)
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    uint8_t *bits;      /* a: 1 bit par bloc */
    uint32_t size;      /* Nombre de blocs gérés */
    uint32_t next_free; /* Hint pour la prochaine recherche */
} chunk_map_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * SECTION 2.3.9.e-g: FREE LIST (Alternative au bitmap)
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct free_chunk {
    uint32_t block;          /* Numéro du bloc libre */
    uint32_t count;          /* g: Counting - nombre de blocs consécutifs */
    struct free_chunk *next; /* e: Liste chaînée */
} free_chunk_list_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * SECTION 2.3.8.g: RAIL NETWORK (FAT - File Allocation Table)
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    uint32_t *tracks;    /* FAT[i] = prochain bloc ou RAIL_END */
    uint32_t size;       /* Nombre d'entrées */
} rail_network_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * SECTION 2.3.8.j-m: CHEST INVENTORY (Inode avec blocs directs/indirects)
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    uint32_t hotbar[HOTBAR_SLOTS];  /* j: 9 blocs directs (hotbar) */
    uint32_t chest;                  /* k: 1 bloc indirect (coffre) */
    uint32_t ender_chest;            /* l: Double indirect (ender chest) */
    uint32_t shulker_box;            /* m: Triple indirect (shulker) */
    uint32_t item_count;             /* Nombre total d'items (blocs) */
    uint32_t stack_count;            /* Nombre de stacks utilisés */
    uint32_t type;                   /* Type de contenu */
} chest_inventory_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * SECTION 2.3.10: OVERWORLD LAYOUT (Filesystem complet)
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    world_spawn_t spawn;           /* b: Superblock (spawn point) */
    world_spawn_t *backup_spawns;  /* f: Copies de sauvegarde */
    uint32_t backup_count;
    chunk_map_t block_map;         /* Bitmap des blocs */
    chunk_map_t chest_map;         /* Bitmap des coffres */
    chest_inventory_t *chests;     /* c: Table des coffres (inodes) */
    uint8_t *blocks;               /* d: Zone de données */
    uint32_t *biomes;              /* e: Block groups (biomes) */
    uint32_t biome_count;
    rail_network_t rails;          /* FAT pour allocation chaînée */
    uint32_t reserved_blocks;      /* g: Blocs réservés */
} overworld_t;

/* Statistiques */
typedef struct {
    uint32_t total_blocks;
    uint32_t used_blocks;
    uint32_t free_blocks;
    uint32_t total_chests;
    uint32_t used_chests;
    float fragmentation;
    uint32_t largest_contiguous;
} world_stats_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * API PRINCIPALE
 * ═══════════════════════════════════════════════════════════════════════════ */

/* --- Cycle de vie du monde --- */

overworld_t *world_create(uint32_t block_count, uint32_t block_size);
void world_destroy(overworld_t *world);
int world_load(overworld_t *world);
int world_save(overworld_t *world);
int world_validate(const overworld_t *world);

/* --- 2.3.9.a-d: Chunk Map (Bitmap) --- */

int chunk_map_init(chunk_map_t *map, uint32_t size);
void chunk_map_destroy(chunk_map_t *map);
int chunk_map_alloc(chunk_map_t *map);                      /* c: Allouer 1 */
int chunk_map_free(chunk_map_t *map, uint32_t block);       /* c: Libérer */
int chunk_map_is_free(const chunk_map_t *map, uint32_t block);
int chunk_map_find_free(const chunk_map_t *map);            /* c: Trouver libre */
int chunk_map_find_contiguous(const chunk_map_t *map, uint32_t n); /* d */
int chunk_map_alloc_contiguous(chunk_map_t *map, uint32_t n, uint32_t *start);

/* --- 2.3.9.e-g: Free List alternative --- */

void free_list_init(free_chunk_list_t **list);
void free_list_destroy(free_chunk_list_t **list);
int free_list_alloc(free_chunk_list_t **list);              /* e */
int free_list_free(free_chunk_list_t **list, uint32_t block);
int free_list_alloc_contiguous(free_chunk_list_t **list, uint32_t n); /* g: counting */

/* --- 2.3.8.c-d: Allocation Contiguë (Build Mode) --- */

int build_structure(overworld_t *world, uint32_t n, uint32_t *start);
int demolish_structure(overworld_t *world, uint32_t start, uint32_t n);
float measure_fragmentation(const overworld_t *world);

/* --- 2.3.8.e-g: Allocation Chaînée (Rail Network) --- */

int rail_network_init(rail_network_t *rails, uint32_t size);
void rail_network_destroy(rail_network_t *rails);
int rail_chain_create(rail_network_t *rails, chunk_map_t *map,
                      uint32_t n, uint32_t *first);
int rail_chain_extend(rail_network_t *rails, chunk_map_t *map,
                      uint32_t last, uint32_t n);
int rail_chain_destroy(rail_network_t *rails, chunk_map_t *map,
                       uint32_t first);
uint32_t rail_chain_follow(const rail_network_t *rails, uint32_t current);
uint32_t rail_chain_length(const rail_network_t *rails, uint32_t first);

/* --- 2.3.8.h-m: Allocation Indexée (Chest System) --- */

int chest_init(chest_inventory_t *chest);
int chest_store_item(chest_inventory_t *chest, overworld_t *world,
                     uint32_t logical_slot, uint32_t physical_block);
uint32_t chest_get_item(const chest_inventory_t *chest,
                        const overworld_t *world, uint32_t logical_slot);
int chest_allocate_space(chest_inventory_t *chest, overworld_t *world,
                         uint32_t n_blocks);
int chest_free_all(chest_inventory_t *chest, overworld_t *world);

/* Résolution de blocs (j-m) */
uint32_t resolve_hotbar_slot(const chest_inventory_t *chest,
                             uint32_t slot);                  /* j: direct */
uint32_t resolve_chest_slot(const chest_inventory_t *chest,
                            const overworld_t *world,
                            uint32_t slot);                   /* k: indirect */
uint32_t resolve_ender_slot(const chest_inventory_t *chest,
                            const overworld_t *world,
                            uint32_t slot);                   /* l: double */
uint32_t resolve_shulker_slot(const chest_inventory_t *chest,
                              const overworld_t *world,
                              uint32_t slot);                 /* m: triple */

/* --- Accès aux blocs --- */

int block_read(const overworld_t *world, uint32_t block, void *buf);
int block_write(overworld_t *world, uint32_t block, const void *buf);
int block_zero(overworld_t *world, uint32_t block);

/* --- Statistiques --- */

void world_get_stats(const overworld_t *world, world_stats_t *stats);
void world_print_stats(const overworld_t *world);

/* --- Utilitaires --- */

uint32_t blocks_needed_for_items(uint32_t item_count);
int is_valid_block(const overworld_t *world, uint32_t block);

#endif /* OVERWORLD_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Évolution des Stratégies d'Allocation

Les systèmes de fichiers ont évolué dans leurs stratégies:

1. **Contiguous (1960s)** : Simple mais fragmentation externe catastrophique
2. **Linked (1970s)** : FAT de MS-DOS, séquentiel lent
3. **Indexed (1980s)** : Unix avec inodes, accès aléatoire O(1)
4. **Extent-based (2000s)** : ext4, XFS, groupes de blocs contigus

### 2.2 Les Nombres Magiques de Minecraft (et des FS)

Minecraft utilise littéralement la même philosophie que les filesystems:
- Chunk size 16x16x256 = optimisation pour le cache
- Région files (.mca) = superblocs avec métadonnées
- NBT format = structure de données hiérarchique comme les inodes

### 2.3 Pourquoi 12 Blocs Directs?

Unix utilise traditionnellement 12 pointeurs directs dans l'inode. Avec des blocs de 4KB:
- 12 directs = 48 KB d'accès O(1)
- La majorité des fichiers font moins de 48 KB!
- Les fichiers plus grands paient le coût de l'indirection

---

## 2.5 DANS LA VRAIE VIE

### Développeur de Systèmes de Fichiers

Les concepts de cet exercice sont directement utilisés dans:
- **ext4** : Linux standard, utilise extents (blocs contigus)
- **NTFS** : Windows, utilise MFT (Master File Table) similaire à FAT
- **ZFS** : Entreprise, utilise COW et checksums

### Développeur de Jeux

Les jeux AAA utilisent ces mêmes structures pour:
- **Asset streaming** : Charger les textures à la volée
- **World generation** : Chunks comme Minecraft, No Man's Sky
- **Save games** : Compression et indexation des données

### DevOps / SRE

```bash
# Vérifier la fragmentation
e2freefrag /dev/sda1

# Analyser les inodes
dumpe2fs /dev/sda1 | grep -i inode

# Défragmenter
e4defrag /mount/point
```

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
overworld.h  overworld.c  chunk_map.c  rail_network.c  chest_system.c  biome_layout.c  Makefile

$ make
gcc -Wall -Wextra -Werror -std=c17 -c overworld.c -o overworld.o
gcc -Wall -Wextra -Werror -std=c17 -c chunk_map.c -o chunk_map.o
gcc -Wall -Wextra -Werror -std=c17 -c rail_network.c -o rail_network.o
gcc -Wall -Wextra -Werror -std=c17 -c chest_system.c -o chest_system.o
gcc -Wall -Wextra -Werror -std=c17 -c biome_layout.c -o biome_layout.o
gcc -Wall -Wextra -Werror main.c *.o -o minecraft_sim

$ ./minecraft_sim
=== MINECRAFT BLOCK ALLOCATOR ===
"Time to mine and craft!"

[1] Creating world with 1024 blocks of 4096 bytes...
    Magic: 0x4D494E45 (MINE)
    Total blocks: 1024
    Block size: 4096

[2] Build Mode (Contiguous): 10 blocks
    Allocated blocks 64-73
    Fragmentation: 0.00%

[3] Rail Network (FAT): 5 blocks
    Chain: 128 -> 256 -> 384 -> 512 -> 640 (END)
    Chain length: 5

[4] Chest System (Indexed): 15000 items
    Hotbar (direct): slots 0-8 used
    Chest (indirect): 1 block allocated
    Ender Chest (double): 2 blocks allocated
    Shulker Box (triple): accessing slot 14000...
    Physical block for logical 14000: 897

[5] World Statistics:
    Used blocks: 142/1024
    Free blocks: 882
    Fragmentation: 12.34%

All tests passed!
```

---

## ⚡ SECTION 3.1 : BONUS STANDARD (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×2

### 3.1.1 Consigne Bonus

**⛏️ THE NETHER — Portail vers l'optimisation**

Comme le Nether permet de voyager 8x plus vite dans Minecraft, optimise ton allocateur:

1. **Extent-based allocation** : Allouer des groupes de blocs contigus
2. **Delayed allocation** : Accumuler les écritures avant d'allouer
3. **Preallocation** : Réserver de l'espace à l'avance

### 3.1.2 Prototype Bonus

```c
/* Extent = groupe de blocs contigus */
typedef struct {
    uint32_t start;
    uint32_t length;
} extent_t;

/* Allouer par extents au lieu de blocs individuels */
int extent_allocate(overworld_t *world, uint32_t n,
                    extent_t *extents, uint32_t max_extents,
                    uint32_t *extent_count);

/* Préallocation */
int preallocate_space(chest_inventory_t *chest, overworld_t *world,
                      uint32_t estimated_size);

/* Statistiques avancées */
typedef struct {
    uint32_t extent_count;
    float avg_extent_size;
    uint32_t seek_count;  /* Nombre de sauts entre extents */
} extent_stats_t;

void analyze_extents(const chest_inventory_t *chest,
                     const overworld_t *world,
                     extent_stats_t *stats);
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (Tests)

| # | Test | Concept | Points |
|---|------|---------|--------|
| 1 | world_create NULL params | Safety | 2 |
| 2 | world_create valid | 2.3.11 | 3 |
| 3 | world_spawn magic number | 2.3.11.a | 3 |
| 4 | chunk_map_init | 2.3.9.a | 3 |
| 5 | chunk_map_alloc single | 2.3.9.c | 3 |
| 6 | chunk_map_free | 2.3.9.c | 3 |
| 7 | chunk_map_find_contiguous | 2.3.9.d | 5 |
| 8 | free_list operations | 2.3.9.e-g | 5 |
| 9 | build_structure contiguous | 2.3.8.c | 5 |
| 10 | demolish_structure | 2.3.8.c | 3 |
| 11 | fragmentation detection | 2.3.8.d | 5 |
| 12 | rail_chain_create | 2.3.8.e-g | 5 |
| 13 | rail_chain_follow | 2.3.8.g | 3 |
| 14 | rail_chain_destroy | 2.3.8.g | 3 |
| 15 | hotbar direct slots | 2.3.8.j | 5 |
| 16 | chest indirect | 2.3.8.k | 7 |
| 17 | ender_chest double | 2.3.8.l | 10 |
| 18 | shulker_box triple | 2.3.8.m | 10 |
| 19 | biome layout | 2.3.10.e | 5 |
| 20 | backup spawns | 2.3.10.f | 5 |
| 21 | reserved blocks | 2.3.10.g | 3 |
| 22 | world_stats accuracy | 2.3.11.e-f | 5 |
| 23 | valgrind clean | Memory | 5 |

**Total: 107 points**

### 4.2 main.c de test

```c
#include "overworld.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

static int tests_passed = 0;
static int tests_total = 0;

#define TEST(name, condition) do { \
    tests_total++; \
    if (condition) { \
        printf("[OK] %s\n", name); \
        tests_passed++; \
    } else { \
        printf("[FAIL] %s\n", name); \
    } \
} while(0)

void test_world_creation(void) {
    printf("\n=== Testing World Creation (2.3.11) ===\n");

    /* NULL params */
    overworld_t *world = world_create(0, 4096);
    TEST("world_create with 0 blocks returns NULL", world == NULL);

    /* Valid creation */
    world = world_create(1024, 4096);
    TEST("world_create returns non-NULL", world != NULL);
    TEST("Magic number is MINE", world->spawn.magic == OVERWORLD_MAGIC);
    TEST("Block count is 1024", world->spawn.block_count == 1024);
    TEST("Block size is 4096", world->spawn.block_size == 4096);
    TEST("State is CLEAN", world->spawn.state == WORLD_CLEAN);

    world_destroy(world);
}

void test_chunk_map(void) {
    printf("\n=== Testing Chunk Map (2.3.9.a-d) ===\n");

    chunk_map_t map;
    TEST("chunk_map_init succeeds", chunk_map_init(&map, 64) == 0);

    /* All blocks should be free initially */
    TEST("Block 0 is free", chunk_map_is_free(&map, 0) == 1);
    TEST("Block 63 is free", chunk_map_is_free(&map, 63) == 1);

    /* Allocate single block */
    int block = chunk_map_alloc(&map);
    TEST("chunk_map_alloc returns valid block", block >= 0 && block < 64);
    TEST("Allocated block is not free", chunk_map_is_free(&map, block) == 0);

    /* Free the block */
    TEST("chunk_map_free succeeds", chunk_map_free(&map, block) == 0);
    TEST("Freed block is free again", chunk_map_is_free(&map, block) == 1);

    /* Contiguous allocation */
    uint32_t start;
    TEST("Contiguous alloc 10 blocks",
         chunk_map_alloc_contiguous(&map, 10, &start) == 0);

    chunk_map_destroy(&map);
}

void test_rail_network(void) {
    printf("\n=== Testing Rail Network (2.3.8.e-g) ===\n");

    overworld_t *world = world_create(256, 1024);
    uint32_t first;

    /* Create chain of 5 blocks */
    TEST("rail_chain_create 5 blocks",
         rail_chain_create(&world->rails, &world->block_map, 5, &first) == 0);
    TEST("Chain has valid first block", first < 256);

    /* Follow the chain */
    uint32_t len = rail_chain_length(&world->rails, first);
    TEST("Chain length is 5", len == 5);

    /* Verify chain integrity */
    uint32_t current = first;
    int count = 0;
    while (current != RAIL_END && count < 10) {
        current = rail_chain_follow(&world->rails, current);
        count++;
    }
    TEST("Chain ends properly", count == 5);

    /* Destroy chain */
    TEST("rail_chain_destroy succeeds",
         rail_chain_destroy(&world->rails, &world->block_map, first) == 0);

    world_destroy(world);
}

void test_chest_system(void) {
    printf("\n=== Testing Chest System (2.3.8.j-m) ===\n");

    overworld_t *world = world_create(4096, 4096);
    chest_inventory_t chest;
    chest_init(&chest);

    /* Test hotbar (direct blocks) */
    for (int i = 0; i < HOTBAR_SLOTS; i++) {
        TEST("Store in hotbar slot",
             chest_store_item(&chest, world, i, 100 + i) == 0);
    }
    TEST("Retrieve from hotbar",
         resolve_hotbar_slot(&chest, 0) == 100);
    TEST("Retrieve last hotbar slot",
         resolve_hotbar_slot(&chest, 8) == 108);

    /* Test chest (single indirect) */
    /* Allocate space that requires indirect block */
    TEST("Allocate 100 items (needs indirect)",
         chest_allocate_space(&chest, world, 100) == 0);

    /* Slot 10 should go through indirect */
    uint32_t phys = chest_get_item(&chest, world, 10);
    TEST("Indirect block resolution works", phys != 0 && phys < 4096);

    /* Test double indirect (ender chest) */
    TEST("Allocate 1000 items (needs double indirect)",
         chest_allocate_space(&chest, world, 1000) == 0);

    /* Test triple indirect (shulker box) */
    TEST("Allocate 10000 items (needs triple indirect)",
         chest_allocate_space(&chest, world, 10000) == 0);

    chest_free_all(&chest, world);
    world_destroy(world);
}

void test_world_stats(void) {
    printf("\n=== Testing World Statistics (2.3.11.e-f) ===\n");

    overworld_t *world = world_create(1024, 4096);
    world_stats_t stats;

    /* Initial stats */
    world_get_stats(world, &stats);
    TEST("Initial free blocks matches total",
         stats.free_blocks + stats.used_blocks == stats.total_blocks);

    /* Allocate some blocks */
    uint32_t start;
    build_structure(world, 100, &start);

    world_get_stats(world, &stats);
    TEST("Used blocks increased", stats.used_blocks >= 100);
    TEST("Free blocks decreased", stats.free_blocks <= 1024 - 100);

    world_destroy(world);
}

int main(void) {
    printf("=== MINECRAFT BLOCK ALLOCATOR TESTS ===\n");
    printf("\"Time to mine and craft!\"\n");

    test_world_creation();
    test_chunk_map();
    test_rail_network();
    test_chest_system();
    test_world_stats();

    printf("\n=== RESULTS ===\n");
    printf("Passed: %d/%d\n", tests_passed, tests_total);

    return (tests_passed == tests_total) ? 0 : 1;
}
```

### 4.3 Solution de référence

```c
/* overworld.c - Solution de référence (extrait) */

#include "overworld.h"
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

/* ═══════════════════════════════════════════════════════════════════════════
 * WORLD LIFECYCLE
 * ═══════════════════════════════════════════════════════════════════════════ */

overworld_t *world_create(uint32_t block_count, uint32_t block_size) {
    if (block_count == 0 || block_size == 0) {
        return NULL;
    }

    overworld_t *world = calloc(1, sizeof(overworld_t));
    if (!world) {
        return NULL;
    }

    /* Initialize spawn point (superblock) */
    world->spawn.magic = OVERWORLD_MAGIC;
    world->spawn.block_size = block_size;
    world->spawn.block_count = block_count;
    world->spawn.free_blocks = block_count;
    world->spawn.state = WORLD_CLEAN;
    world->spawn.error_behavior = ON_ERROR_CONTINUE;

    /* Calculate layout */
    uint32_t bitmap_blocks = (block_count + 7) / 8 / block_size + 1;
    uint32_t chest_count = block_count / 4;
    uint32_t chest_blocks = (chest_count * sizeof(chest_inventory_t)) / block_size + 1;
    world->spawn.first_data_block = 1 + bitmap_blocks + chest_blocks;
    world->spawn.chest_count = chest_count;
    world->spawn.free_chests = chest_count;

    /* Initialize bitmap */
    if (chunk_map_init(&world->block_map, block_count) < 0) {
        free(world);
        return NULL;
    }

    /* Initialize chest bitmap */
    if (chunk_map_init(&world->chest_map, chest_count) < 0) {
        chunk_map_destroy(&world->block_map);
        free(world);
        return NULL;
    }

    /* Allocate chest table */
    world->chests = calloc(chest_count, sizeof(chest_inventory_t));
    if (!world->chests) {
        chunk_map_destroy(&world->chest_map);
        chunk_map_destroy(&world->block_map);
        free(world);
        return NULL;
    }

    /* Allocate data blocks */
    world->blocks = calloc(block_count, block_size);
    if (!world->blocks) {
        free(world->chests);
        chunk_map_destroy(&world->chest_map);
        chunk_map_destroy(&world->block_map);
        free(world);
        return NULL;
    }

    /* Initialize rail network */
    if (rail_network_init(&world->rails, block_count) < 0) {
        free(world->blocks);
        free(world->chests);
        chunk_map_destroy(&world->chest_map);
        chunk_map_destroy(&world->block_map);
        free(world);
        return NULL;
    }

    /* Mark metadata blocks as used */
    for (uint32_t i = 0; i < world->spawn.first_data_block; i++) {
        chunk_map_alloc(&world->block_map);
        world->spawn.free_blocks--;
    }

    return world;
}

void world_destroy(overworld_t *world) {
    if (!world) {
        return;
    }

    rail_network_destroy(&world->rails);
    free(world->blocks);
    free(world->chests);
    free(world->backup_spawns);
    free(world->biomes);
    chunk_map_destroy(&world->chest_map);
    chunk_map_destroy(&world->block_map);
    free(world);
}

/* ═══════════════════════════════════════════════════════════════════════════
 * CHUNK MAP (BITMAP) - 2.3.9.a-d
 * ═══════════════════════════════════════════════════════════════════════════ */

int chunk_map_init(chunk_map_t *map, uint32_t size) {
    if (!map || size == 0) {
        return -1;
    }

    map->size = size;
    map->next_free = 0;
    map->bits = calloc((size + 7) / 8, 1);
    if (!map->bits) {
        return -1;
    }

    return 0;
}

void chunk_map_destroy(chunk_map_t *map) {
    if (map && map->bits) {
        free(map->bits);
        map->bits = NULL;
    }
}

int chunk_map_is_free(const chunk_map_t *map, uint32_t block) {
    if (!map || !map->bits || block >= map->size) {
        return 0;
    }
    return !(map->bits[block / 8] & (1 << (block % 8)));
}

int chunk_map_alloc(chunk_map_t *map) {
    if (!map || !map->bits) {
        return -1;
    }

    /* Start from hint */
    for (uint32_t i = map->next_free; i < map->size; i++) {
        if (chunk_map_is_free(map, i)) {
            map->bits[i / 8] |= (1 << (i % 8));
            map->next_free = i + 1;
            return i;
        }
    }

    /* Wrap around */
    for (uint32_t i = 0; i < map->next_free; i++) {
        if (chunk_map_is_free(map, i)) {
            map->bits[i / 8] |= (1 << (i % 8));
            map->next_free = i + 1;
            return i;
        }
    }

    return -1; /* No free blocks */
}

int chunk_map_free(chunk_map_t *map, uint32_t block) {
    if (!map || !map->bits || block >= map->size) {
        return -1;
    }

    map->bits[block / 8] &= ~(1 << (block % 8));
    if (block < map->next_free) {
        map->next_free = block;
    }

    return 0;
}

int chunk_map_find_contiguous(const chunk_map_t *map, uint32_t n) {
    if (!map || !map->bits || n == 0) {
        return -1;
    }

    uint32_t count = 0;
    uint32_t start = 0;

    for (uint32_t i = 0; i < map->size; i++) {
        if (chunk_map_is_free(map, i)) {
            if (count == 0) {
                start = i;
            }
            count++;
            if (count == n) {
                return start;
            }
        } else {
            count = 0;
        }
    }

    return -1;
}

int chunk_map_alloc_contiguous(chunk_map_t *map, uint32_t n, uint32_t *start) {
    if (!map || !start || n == 0) {
        return -1;
    }

    int s = chunk_map_find_contiguous(map, n);
    if (s < 0) {
        return -1;
    }

    for (uint32_t i = 0; i < n; i++) {
        map->bits[(s + i) / 8] |= (1 << ((s + i) % 8));
    }

    *start = s;
    return 0;
}

/* ═══════════════════════════════════════════════════════════════════════════
 * RAIL NETWORK (FAT) - 2.3.8.e-g
 * ═══════════════════════════════════════════════════════════════════════════ */

int rail_network_init(rail_network_t *rails, uint32_t size) {
    if (!rails || size == 0) {
        return -1;
    }

    rails->size = size;
    rails->tracks = malloc(size * sizeof(uint32_t));
    if (!rails->tracks) {
        return -1;
    }

    for (uint32_t i = 0; i < size; i++) {
        rails->tracks[i] = RAIL_FREE;
    }

    return 0;
}

void rail_network_destroy(rail_network_t *rails) {
    if (rails && rails->tracks) {
        free(rails->tracks);
        rails->tracks = NULL;
    }
}

int rail_chain_create(rail_network_t *rails, chunk_map_t *map,
                      uint32_t n, uint32_t *first) {
    if (!rails || !map || !first || n == 0) {
        return -1;
    }

    uint32_t prev = RAIL_END;
    uint32_t first_block = RAIL_END;

    for (uint32_t i = 0; i < n; i++) {
        int block = chunk_map_alloc(map);
        if (block < 0) {
            /* Rollback */
            if (first_block != RAIL_END) {
                rail_chain_destroy(rails, map, first_block);
            }
            return -1;
        }

        if (first_block == RAIL_END) {
            first_block = block;
        }

        if (prev != RAIL_END) {
            rails->tracks[prev] = block;
        }

        rails->tracks[block] = RAIL_END;
        prev = block;
    }

    *first = first_block;
    return 0;
}

int rail_chain_destroy(rail_network_t *rails, chunk_map_t *map,
                       uint32_t first) {
    if (!rails || !map) {
        return -1;
    }

    uint32_t current = first;
    while (current != RAIL_END && current < rails->size) {
        uint32_t next = rails->tracks[current];
        rails->tracks[current] = RAIL_FREE;
        chunk_map_free(map, current);
        current = next;
    }

    return 0;
}

uint32_t rail_chain_follow(const rail_network_t *rails, uint32_t current) {
    if (!rails || !rails->tracks || current >= rails->size) {
        return RAIL_END;
    }
    return rails->tracks[current];
}

uint32_t rail_chain_length(const rail_network_t *rails, uint32_t first) {
    if (!rails) {
        return 0;
    }

    uint32_t count = 0;
    uint32_t current = first;
    while (current != RAIL_END && current < rails->size && count < rails->size) {
        count++;
        current = rails->tracks[current];
    }

    return count;
}

/* ═══════════════════════════════════════════════════════════════════════════
 * CHEST SYSTEM (INDEXED ALLOCATION) - 2.3.8.h-m
 * ═══════════════════════════════════════════════════════════════════════════ */

int chest_init(chest_inventory_t *chest) {
    if (!chest) {
        return -1;
    }

    memset(chest, 0, sizeof(*chest));
    return 0;
}

uint32_t resolve_hotbar_slot(const chest_inventory_t *chest, uint32_t slot) {
    if (!chest || slot >= HOTBAR_SLOTS) {
        return 0;
    }
    return chest->hotbar[slot];
}

uint32_t resolve_chest_slot(const chest_inventory_t *chest,
                            const overworld_t *world,
                            uint32_t slot) {
    if (!chest || !world || chest->chest == 0) {
        return 0;
    }

    /* Read indirect block */
    uint32_t *indirect = (uint32_t *)(world->blocks +
                         chest->chest * world->spawn.block_size);
    if (slot >= CHEST_SLOTS) {
        return 0;
    }

    return indirect[slot];
}

uint32_t resolve_ender_slot(const chest_inventory_t *chest,
                            const overworld_t *world,
                            uint32_t slot) {
    if (!chest || !world || chest->ender_chest == 0) {
        return 0;
    }

    /* Double indirect: ender_chest -> chest -> block */
    uint32_t *level1 = (uint32_t *)(world->blocks +
                       chest->ender_chest * world->spawn.block_size);
    uint32_t idx1 = slot / CHEST_SLOTS;
    if (idx1 >= CHEST_SLOTS || level1[idx1] == 0) {
        return 0;
    }

    uint32_t *level2 = (uint32_t *)(world->blocks +
                       level1[idx1] * world->spawn.block_size);
    uint32_t idx2 = slot % CHEST_SLOTS;

    return level2[idx2];
}

uint32_t resolve_shulker_slot(const chest_inventory_t *chest,
                              const overworld_t *world,
                              uint32_t slot) {
    if (!chest || !world || chest->shulker_box == 0) {
        return 0;
    }

    /* Triple indirect: shulker -> ender -> chest -> block */
    uint32_t *level1 = (uint32_t *)(world->blocks +
                       chest->shulker_box * world->spawn.block_size);
    uint32_t idx1 = slot / (CHEST_SLOTS * CHEST_SLOTS);
    if (idx1 >= CHEST_SLOTS || level1[idx1] == 0) {
        return 0;
    }

    uint32_t *level2 = (uint32_t *)(world->blocks +
                       level1[idx1] * world->spawn.block_size);
    uint32_t idx2 = (slot / CHEST_SLOTS) % CHEST_SLOTS;
    if (level2[idx2] == 0) {
        return 0;
    }

    uint32_t *level3 = (uint32_t *)(world->blocks +
                       level2[idx2] * world->spawn.block_size);
    uint32_t idx3 = slot % CHEST_SLOTS;

    return level3[idx3];
}

uint32_t chest_get_item(const chest_inventory_t *chest,
                        const overworld_t *world,
                        uint32_t logical_slot) {
    if (!chest || !world) {
        return 0;
    }

    /* Direct slots (hotbar) */
    if (logical_slot < HOTBAR_SLOTS) {
        return resolve_hotbar_slot(chest, logical_slot);
    }

    /* Single indirect (chest) */
    uint32_t indirect_start = HOTBAR_SLOTS;
    uint32_t indirect_end = indirect_start + CHEST_SLOTS;
    if (logical_slot < indirect_end) {
        return resolve_chest_slot(chest, world, logical_slot - indirect_start);
    }

    /* Double indirect (ender chest) */
    uint32_t double_end = indirect_end + CHEST_SLOTS * CHEST_SLOTS;
    if (logical_slot < double_end) {
        return resolve_ender_slot(chest, world, logical_slot - indirect_end);
    }

    /* Triple indirect (shulker box) */
    return resolve_shulker_slot(chest, world, logical_slot - double_end);
}
```

### 4.9 spec.json

```json
{
  "name": "overworld_block_manager",
  "language": "c",
  "version": "c17",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (2.3.8-2.3.11)",
  "tags": ["filesystem", "block-allocation", "bitmap", "fat", "inode", "indirect-blocks"],
  "passing_score": 70,

  "function": {
    "name": "world_create",
    "prototype": "overworld_t *world_create(uint32_t block_count, uint32_t block_size)",
    "return_type": "overworld_t *",
    "parameters": [
      {"name": "block_count", "type": "uint32_t"},
      {"name": "block_size", "type": "uint32_t"}
    ]
  },

  "driver": {
    "reference": "overworld_t *ref_world_create(uint32_t block_count, uint32_t block_size) { if (block_count == 0 || block_size == 0) return NULL; overworld_t *w = calloc(1, sizeof(*w)); if (!w) return NULL; w->spawn.magic = 0x4D494E45; w->spawn.block_count = block_count; w->spawn.block_size = block_size; return w; }",

    "edge_cases": [
      {
        "name": "zero_blocks",
        "args": [0, 4096],
        "expected": "NULL",
        "is_trap": true,
        "trap_explanation": "Cannot create world with 0 blocks"
      },
      {
        "name": "zero_size",
        "args": [1024, 0],
        "expected": "NULL",
        "is_trap": true,
        "trap_explanation": "Cannot create world with 0 block size"
      },
      {
        "name": "valid_creation",
        "args": [1024, 4096],
        "expected": "non-NULL"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 200,
      "generators": [
        {"type": "int", "param_index": 0, "params": {"min": 0, "max": 100000}},
        {"type": "int", "param_index": 1, "params": {"min": 0, "max": 8192}}
      ]
    }
  },

  "norm": {
    "allowed_functions": ["malloc", "free", "calloc", "memset", "memcpy", "printf", "snprintf"],
    "forbidden_functions": ["open", "read", "write", "close", "system", "exec"],
    "check_memory": true,
    "max_lines_per_function": 40
  }
}
```

### 4.10 Solutions Mutantes

```c
/* MUTANT A (Safety): Pas de vérification NULL dans chunk_map_alloc */
int mutant_a_chunk_map_alloc(chunk_map_t *map) {
    /* MISSING: if (!map || !map->bits) return -1; */
    for (uint32_t i = 0; i < map->size; i++) {  /* CRASH if map is NULL */
        if (chunk_map_is_free(map, i)) {
            map->bits[i / 8] |= (1 << (i % 8));
            return i;
        }
    }
    return -1;
}

/* MUTANT B (Logic): Mauvais calcul d'index pour double indirect */
uint32_t mutant_b_resolve_ender(const chest_inventory_t *chest,
                                 const overworld_t *world,
                                 uint32_t slot) {
    uint32_t *level1 = (uint32_t *)(world->blocks +
                       chest->ender_chest * world->spawn.block_size);
    uint32_t idx1 = slot % CHEST_SLOTS;  /* ERREUR: devrait être slot / CHEST_SLOTS */
    uint32_t *level2 = (uint32_t *)(world->blocks +
                       level1[idx1] * world->spawn.block_size);
    uint32_t idx2 = slot / CHEST_SLOTS;  /* ERREUR: devrait être slot % CHEST_SLOTS */
    return level2[idx2];
}

/* MUTANT C (Resource): Fuite mémoire dans world_create */
overworld_t *mutant_c_world_create(uint32_t block_count, uint32_t block_size) {
    overworld_t *world = calloc(1, sizeof(overworld_t));
    if (chunk_map_init(&world->block_map, block_count) < 0) {
        /* MISSING: free(world); */
        return NULL;  /* Fuite de world */
    }
    /* ... */
    return world;
}

/* MUTANT D (Boundary): Off-by-one dans chunk_map_find_contiguous */
int mutant_d_find_contiguous(const chunk_map_t *map, uint32_t n) {
    uint32_t count = 0;
    uint32_t start = 0;
    for (uint32_t i = 0; i <= map->size; i++) {  /* ERREUR: <= au lieu de < */
        if (chunk_map_is_free(map, i)) {
            if (count == 0) start = i;
            count++;
            if (count == n) return start;
        } else {
            count = 0;
        }
    }
    return -1;
}

/* MUTANT E (Return): Retourne le mauvais bloc après allocation */
int mutant_e_chunk_map_alloc(chunk_map_t *map) {
    for (uint32_t i = 0; i < map->size; i++) {
        if (chunk_map_is_free(map, i)) {
            map->bits[i / 8] |= (1 << (i % 8));
            return i + 1;  /* ERREUR: retourne i+1 au lieu de i */
        }
    }
    return -1;
}

/* MUTANT F (Logic): Ne met pas à jour free_blocks dans superblock */
int mutant_f_build_structure(overworld_t *world, uint32_t n, uint32_t *start) {
    int result = chunk_map_alloc_contiguous(&world->block_map, n, start);
    /* MISSING: world->spawn.free_blocks -= n; */
    return result;
}
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Section | Concepts | Implémentation Minecraft |
|---------|----------|--------------------------|
| 2.3.8 | Block Allocation (13) | Build Mode, Rail Network, Chest System |
| 2.3.9 | Free Space (7) | Chunk Map (bitmap), Free List |
| 2.3.10 | FS Layout (7) | Overworld structure, Biomes |
| 2.3.11 | Superblock (10) | World Spawn point |

### 5.2 LDA — Traduction Littérale

```
FONCTION chunk_map_find_contiguous QUI RETOURNE UN ENTIER ET PREND EN PARAMÈTRES map QUI EST UN POINTEUR VERS chunk_map_t ET n QUI EST UN ENTIER NON SIGNÉ
DÉBUT FONCTION
    DÉCLARER count COMME ENTIER NON SIGNÉ
    DÉCLARER start COMME ENTIER NON SIGNÉ

    SI map EST NUL OU bits DE map EST NUL OU n EST ÉGAL À 0 ALORS
        RETOURNER MOINS 1
    FIN SI

    AFFECTER 0 À count
    AFFECTER 0 À start

    POUR i ALLANT DE 0 À size DE map MOINS 1 FAIRE
        SI LE BLOC i EST LIBRE ALORS
            SI count EST ÉGAL À 0 ALORS
                AFFECTER i À start
            FIN SI
            INCRÉMENTER count DE 1
            SI count EST ÉGAL À n ALORS
                RETOURNER start
            FIN SI
        SINON
            AFFECTER 0 À count
        FIN SI
    FIN POUR

    RETOURNER MOINS 1
FIN FONCTION
```

### 5.3 Visualisation ASCII

```
                    LAYOUT DU MONDE (FILESYSTEM)
    ════════════════════════════════════════════════════════════════

    ┌────────────────────────────────────────────────────────────┐
    │                     OVERWORLD LAYOUT                        │
    ├────────────────────────────────────────────────────────────┤
    │ Block 0    │ WORLD SPAWN (Superblock)                      │
    │            │ ┌──────────────────────────────────────────┐  │
    │            │ │ Magic: 0x4D494E45 ("MINE")               │  │
    │            │ │ Block Size: 4096                         │  │
    │            │ │ Block Count: 1024                        │  │
    │            │ │ Free Blocks: 882                         │  │
    │            │ │ State: CLEAN                             │  │
    │            │ └──────────────────────────────────────────┘  │
    ├────────────┼───────────────────────────────────────────────┤
    │ Block 1-2  │ CHUNK MAP (Bitmap)                            │
    │            │ ┌──────────────────────────────────────────┐  │
    │            │ │ [11111111][11110000][00000000]...        │  │
    │            │ │  ^^^^^^^^  ^^^^                          │  │
    │            │ │  metadata  allocated data blocks         │  │
    │            │ └──────────────────────────────────────────┘  │
    ├────────────┼───────────────────────────────────────────────┤
    │ Block 3-10 │ CHEST TABLE (Inodes)                          │
    │            │ ┌──────────────────────────────────────────┐  │
    │            │ │ Chest[0]: hotbar[9], chest, ender, shulk │  │
    │            │ │ Chest[1]: hotbar[9], chest, ender, shulk │  │
    │            │ │ ...                                      │  │
    │            │ └──────────────────────────────────────────┘  │
    ├────────────┼───────────────────────────────────────────────┤
    │ Block 11+  │ DATA BLOCKS (Terrain/Items)                   │
    │            │ ┌──────────────────────────────────────────┐  │
    │            │ │ [Block 11][Block 12][Block 13]...        │  │
    │            │ └──────────────────────────────────────────┘  │
    └────────────┴───────────────────────────────────────────────┘


                    CHEST SYSTEM (INODE STRUCTURE)
    ════════════════════════════════════════════════════════════════

    ┌─────────────────────────────────────────────────────────────┐
    │                    CHEST_INVENTORY (Inode)                  │
    ├─────────────────────────────────────────────────────────────┤
    │  HOTBAR (Direct Blocks) - 9 slots, accès O(1)               │
    │  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┐                     │
    │  │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ → Data Blocks       │
    │  └───┴───┴───┴───┴───┴───┴───┴───┴───┘                     │
    ├─────────────────────────────────────────────────────────────┤
    │  CHEST (Single Indirect) - 27 slots via 1 bloc              │
    │  ┌───────┐                                                  │
    │  │ chest │──→ ┌───────────────────────────┐                 │
    │  └───────┘    │ 27 pointeurs vers blocs   │                 │
    │               └───────────────────────────┘                 │
    ├─────────────────────────────────────────────────────────────┤
    │  ENDER CHEST (Double Indirect) - 27×27 = 729 slots          │
    │  ┌───────────┐                                              │
    │  │ender_chest│──→ ┌───┬───┬...┬───┐                         │
    │  └───────────┘    │ P │ P │   │ P │ (27 pointeurs)          │
    │                   └─┬─┴─┬─┴...┴─┬─┘                         │
    │                     │   │       └──→ [27 data blocks]       │
    │                     │   └──→ [27 data blocks]               │
    │                     └──→ [27 data blocks]                   │
    ├─────────────────────────────────────────────────────────────┤
    │  SHULKER BOX (Triple Indirect) - 27×27×27 = 19683 slots     │
    │  ┌────────────┐                                             │
    │  │shulker_box │──→ [27 ptrs]──→ [27 ptrs]──→ [27 blocks]    │
    │  └────────────┘                                             │
    └─────────────────────────────────────────────────────────────┘


                    RAIL NETWORK (FAT)
    ════════════════════════════════════════════════════════════════

    FAT Table:
    ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
    │  0  │  1  │  2  │  3  │  4  │  5  │  6  │ ... │
    ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
    │ END │  4  │FREE │FREE │  6  │FREE │ END │ ... │
    └─────┴──┬──┴─────┴─────┴──┬──┴─────┴─────┴─────┘
             │                 │
             └────────────────┘
             Chain: 1 → 4 → 6 → END

    Blocks:
    ┌───────┐   ┌───────┐   ┌───────┐
    │Block 1│──→│Block 4│──→│Block 6│──→ END
    │ DATA  │   │ DATA  │   │ DATA  │
    └───────┘   └───────┘   └───────┘
```

### 5.4 Les pièges en détail

| Piège | Symptôme | Solution |
|-------|----------|----------|
| Fuite mémoire world_create | Valgrind errors | Free tout en cas d'échec |
| Off-by-one dans bitmap | Corruption silencieuse | Utiliser `< size` pas `<= size` |
| Mauvais index indirect | Mauvais bloc retourné | slot/SLOTS puis slot%SLOTS |
| Oublier de mettre à jour free_blocks | Stats incorrectes | Décrémenter à chaque alloc |
| FAT loop infini | Hang | Limiter à rails->size itérations |
| Double free dans rail_chain_destroy | Crash/corruption | Vérifier RAIL_FREE avant free |

### 5.8 Mnémotechniques

#### ⛏️ MEME : "Never dig straight down" — Vérification NULL

Comme dans Minecraft où creuser verticalement peut te faire tomber dans la lave, ne jamais accéder à un pointeur sans vérifier s'il est NULL.

```c
// Ne pas creuser sans vérifier!
if (!map || !map->bits) {
    return -1;  // Lave évitée!
}
```

#### 📦 MEME : "Full inventory" — Espace insuffisant

Quand ton inventaire Minecraft est plein, tu ne peux plus ramasser d'items. Pareil pour chunk_map_alloc quand tous les blocs sont utilisés.

```c
// Inventaire plein!
if (chunk_map_alloc(&map) < 0) {
    printf("No more space! Drop something first.\n");
}
```

#### 🚂 MEME : "Minecart off the rails" — Fin de chaîne FAT

Comme un minecart qui arrive en bout de rail, une chaîne FAT se termine par RAIL_END.

```c
while (current != RAIL_END) {
    // Suivre les rails...
    current = rails->tracks[current];
}
// Arrivé à destination!
```

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Détail |
|---------|--------|
| **Concepts** | 37 (2.3.8 à 2.3.11) |
| **Fonctions clés** | world_create, chunk_map_*, rail_chain_*, chest_* |
| **Thème** | Minecraft - Gestion des blocs |
| **Difficulté** | 7/10 |
| **Temps** | 8 heures |
| **Points clés** | Bitmap, FAT, blocs directs/indirects |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.5-overworld-block-manager",
    "generated_at": "2026-01-11T00:00:00Z",

    "metadata": {
      "exercise_id": "2.3.5",
      "exercise_name": "overworld_block_manager",
      "module": "2.3",
      "module_name": "File Systems",
      "concept": "2.3.8-2.3.11",
      "concept_name": "Block Allocation & FS Layout",
      "type": "code",
      "tier": 3,
      "phase": 2,
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "language": "c",
      "duration_minutes": 480,
      "xp_base": 350,
      "xp_bonus_multiplier": 2,
      "meme_reference": "Minecraft - Time to mine and craft"
    }
  }
}
```

---

*"Time to mine and craft!" — Minecraft*

*HACKBRAIN v5.5.2 — L'excellence pédagogique ne se négocie pas*
