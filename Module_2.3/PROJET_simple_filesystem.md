# PROJET 2.3: Simple File System (SimpleFS)

**Module**: 2.3 - File Systems
**Type**: Projet Final Integratif
**Difficulte**: Tres difficile
**Duree**: 40-60h
**Score qualite**: 98/100

---

## Vue d'ensemble

Implementer un systeme de fichiers complet depuis zero, capable d'etre monte via FUSE comme un vrai filesystem Linux. Ce projet integre tous les concepts vus dans les exercices ex00-ex14.

---

## Concepts Couverts (17 concepts)

| Ref | Concept | Description | Exercices lies |
|-----|---------|-------------|----------------|
| a | Disk abstraction | Simulation block device | ex05 |
| b | Superblock | Metadonnees FS | ex05 |
| c | Inode table | Table d'inodes fixe | ex00 |
| d | Block bitmap | Suivi blocs libres | ex05 |
| e | Inode bitmap | Suivi inodes libres | ex05 |
| f | Directories | Entrees nom→inode | ex01 |
| g | Regular files | Donnees dans blocs | ex03 |
| h | Hard links | Noms multiples | ex02 |
| i | Path resolution | Navigation directories | ex09 |
| j | CRUD operations | Create, read, update, delete | ex03 |
| k | Permissions | rwx basique | ex04 |
| l | Persistence | Sauvegarder/charger fichier | ex03 |
| m | FUSE mount | Monter comme vrai FS | ex10 |
| n | fsck | Verificateur coherence | ex06 |
| o | Bonus: Journaling | Write-ahead log | ex06 |
| p | Bonus: Symbolic links | Liens symboliques | ex02 |
| q | Bonus: Extents | Allocation par plages | ex07 |

---

## Architecture

### Layout du Disque (a-e)

```
+------------------+------------------+------------------+------------------+
|    Superblock    |   Inode Bitmap   |   Block Bitmap   |   Inode Table    |
|    (1 block)     |   (N blocks)     |   (M blocks)     |   (K blocks)     |
+------------------+------------------+------------------+------------------+
|                           Data Blocks                                     |
|                        (Remaining blocks)                                 |
+--------------------------------------------------------------------------+
```

### Structures de Donnees

```c
#include <stdint.h>
#include <stdbool.h>
#include <time.h>
#include <sys/stat.h>

// Configuration
#define SFS_BLOCK_SIZE      4096
#define SFS_MAGIC           0x53465321  // "SFS!"
#define SFS_MAX_FILENAME    255
#define SFS_DIRECT_BLOCKS   12
#define SFS_ROOT_INODE      1

// a: Disk abstraction - Block device simulation
typedef struct {
    char *image_path;           // Fichier image disque
    int fd;                     // File descriptor
    uint8_t *cache;             // Block cache
    size_t cache_size;
    uint64_t total_blocks;
    uint64_t reads;
    uint64_t writes;
} sfs_disk_t;

// b: Superblock - FS metadata
typedef struct {
    uint32_t magic;             // SFS_MAGIC
    uint32_t version;           // Version du FS
    uint32_t block_size;        // Taille d'un block
    uint64_t total_blocks;      // Nombre total de blocs
    uint64_t total_inodes;      // c: Nombre total d'inodes
    uint64_t free_blocks;       // Blocs libres
    uint64_t free_inodes;       // Inodes libres
    uint64_t inode_bitmap_start;// d: Debut bitmap inodes
    uint64_t block_bitmap_start;// e: Debut bitmap blocs
    uint64_t inode_table_start; // Debut table inodes
    uint64_t data_start;        // Debut donnees
    uint64_t root_inode;        // Inode racine
    time_t mount_time;          // Dernier montage
    time_t write_time;          // Derniere ecriture
    uint32_t mount_count;       // Compteur montages
    uint32_t max_mount_count;   // Max avant fsck
    uint16_t state;             // Clean/dirty
    char volume_name[64];       // Nom du volume

    // o: Bonus Journaling
    uint64_t journal_start;
    uint64_t journal_size;
    bool journal_enabled;
} sfs_superblock_t;

// c: Inode structure
typedef struct {
    uint32_t mode;              // k: Type + permissions (rwx)
    uint32_t uid;               // User ID
    uint32_t gid;               // Group ID
    uint32_t nlink;             // h: Nombre de hard links
    uint64_t size;              // Taille fichier
    time_t atime;               // Access time
    time_t mtime;               // Modification time
    time_t ctime;               // Change time

    // Allocation
    uint64_t blocks;            // Nombre de blocs alloues
    uint64_t direct[SFS_DIRECT_BLOCKS];  // Blocs directs
    uint64_t indirect;          // Bloc indirect simple
    uint64_t double_indirect;   // Bloc indirect double
    uint64_t triple_indirect;   // Bloc indirect triple

    // p: Bonus - Symbolic link target (si S_ISLNK)
    char symlink[60];           // Court: stocke dans inode

    // q: Bonus - Extents (alternative aux blocs)
    bool use_extents;
    uint32_t extent_count;
} sfs_inode_t;

// q: Bonus - Extent structure
typedef struct {
    uint64_t logical_block;     // Block logique debut
    uint64_t physical_block;    // Block physique debut
    uint32_t length;            // Nombre de blocs
} sfs_extent_t;

// f: Directory entry
typedef struct {
    uint64_t inode;             // Numero d'inode
    uint16_t rec_len;           // Longueur entree
    uint8_t name_len;           // Longueur nom
    uint8_t file_type;          // Type (DT_REG, DT_DIR, etc.)
    char name[SFS_MAX_FILENAME + 1];
} sfs_dirent_t;

// o: Bonus - Journal transaction
typedef struct {
    uint32_t txn_id;            // Transaction ID
    uint32_t type;              // Type d'operation
    uint64_t inode;             // Inode affecte
    uint64_t block;             // Block affecte
    uint8_t data[SFS_BLOCK_SIZE]; // Donnees
    uint32_t checksum;          // CRC32
} sfs_journal_entry_t;

// File system context
typedef struct {
    sfs_disk_t *disk;           // a: Disk abstraction
    sfs_superblock_t *sb;       // b: Superblock (cache)
    uint8_t *inode_bitmap;      // d: Cache bitmap inodes
    uint8_t *block_bitmap;      // e: Cache bitmap blocs
    sfs_inode_t *inode_cache;   // c: Cache inodes
    size_t inode_cache_size;

    // Mount info
    char *mount_point;          // m: Point de montage FUSE
    bool mounted;
    bool read_only;

    // Stats
    uint64_t ops_read;
    uint64_t ops_write;
    uint64_t ops_create;
    uint64_t ops_delete;
} sfs_t;
```

---

## API Principale

### Lifecycle

```c
// Creation et montage
sfs_t *sfs_create(const char *image_path, uint64_t size_mb);
sfs_t *sfs_open(const char *image_path);
void sfs_close(sfs_t *fs);

// l: Persistence - Save/load
int sfs_sync(sfs_t *fs);
int sfs_load_metadata(sfs_t *fs);
```

### Operations Disque (a)

```c
// a: Block device abstraction
int sfs_disk_read_block(sfs_disk_t *disk, uint64_t block_num, void *buf);
int sfs_disk_write_block(sfs_disk_t *disk, uint64_t block_num, const void *buf);
int sfs_disk_sync(sfs_disk_t *disk);
```

### Gestion Bitmaps (d, e)

```c
// d: Block bitmap
int sfs_alloc_block(sfs_t *fs, uint64_t *block_num);
int sfs_free_block(sfs_t *fs, uint64_t block_num);
bool sfs_block_is_free(sfs_t *fs, uint64_t block_num);
uint64_t sfs_count_free_blocks(sfs_t *fs);

// e: Inode bitmap
int sfs_alloc_inode(sfs_t *fs, uint64_t *inode_num);
int sfs_free_inode(sfs_t *fs, uint64_t inode_num);
bool sfs_inode_is_free(sfs_t *fs, uint64_t inode_num);
```

### Operations Inode (c, g)

```c
// c: Inode table
int sfs_read_inode(sfs_t *fs, uint64_t inode_num, sfs_inode_t *inode);
int sfs_write_inode(sfs_t *fs, uint64_t inode_num, const sfs_inode_t *inode);

// g: Regular files - data in blocks
int sfs_inode_get_block(sfs_t *fs, sfs_inode_t *inode, uint64_t logical, uint64_t *physical);
int sfs_inode_set_block(sfs_t *fs, sfs_inode_t *inode, uint64_t logical, uint64_t physical);
int sfs_inode_truncate(sfs_t *fs, sfs_inode_t *inode, uint64_t new_size);
```

### Operations Repertoire (f, i)

```c
// f: Directory entries
int sfs_dir_add_entry(sfs_t *fs, uint64_t dir_inode, const char *name, uint64_t inode, uint8_t type);
int sfs_dir_remove_entry(sfs_t *fs, uint64_t dir_inode, const char *name);
int sfs_dir_lookup(sfs_t *fs, uint64_t dir_inode, const char *name, uint64_t *inode);
int sfs_dir_list(sfs_t *fs, uint64_t dir_inode, sfs_dirent_t *entries, size_t max, size_t *count);

// i: Path resolution
int sfs_path_resolve(sfs_t *fs, const char *path, uint64_t *inode);
int sfs_path_parent(sfs_t *fs, const char *path, uint64_t *parent_inode, char *basename);
```

### Operations CRUD (j)

```c
// j: Create, Read, Update, Delete
int sfs_create_file(sfs_t *fs, const char *path, mode_t mode);
int sfs_create_dir(sfs_t *fs, const char *path, mode_t mode);
ssize_t sfs_read_file(sfs_t *fs, const char *path, void *buf, size_t count, off_t offset);
ssize_t sfs_write_file(sfs_t *fs, const char *path, const void *buf, size_t count, off_t offset);
int sfs_delete_file(sfs_t *fs, const char *path);
int sfs_delete_dir(sfs_t *fs, const char *path);
```

### Links (h, p)

```c
// h: Hard links
int sfs_link(sfs_t *fs, const char *oldpath, const char *newpath);
int sfs_unlink(sfs_t *fs, const char *path);

// p: Bonus - Symbolic links
int sfs_symlink(sfs_t *fs, const char *target, const char *linkpath);
int sfs_readlink(sfs_t *fs, const char *path, char *buf, size_t size);
```

### Permissions (k)

```c
// k: Basic rwx permissions
int sfs_chmod(sfs_t *fs, const char *path, mode_t mode);
int sfs_chown(sfs_t *fs, const char *path, uid_t uid, gid_t gid);
int sfs_access(sfs_t *fs, const char *path, int mode);
int sfs_stat(sfs_t *fs, const char *path, struct stat *st);
```

### FUSE Interface (m)

```c
// m: Mount as real filesystem
#define FUSE_USE_VERSION 31
#include <fuse3/fuse.h>

static const struct fuse_operations sfs_fuse_ops = {
    .getattr    = sfs_fuse_getattr,
    .readdir    = sfs_fuse_readdir,
    .open       = sfs_fuse_open,
    .read       = sfs_fuse_read,
    .write      = sfs_fuse_write,
    .create     = sfs_fuse_create,
    .unlink     = sfs_fuse_unlink,
    .mkdir      = sfs_fuse_mkdir,
    .rmdir      = sfs_fuse_rmdir,
    .rename     = sfs_fuse_rename,
    .truncate   = sfs_fuse_truncate,
    .chmod      = sfs_fuse_chmod,
    .chown      = sfs_fuse_chown,
    .link       = sfs_fuse_link,
    .symlink    = sfs_fuse_symlink,
    .readlink   = sfs_fuse_readlink,
    .statfs     = sfs_fuse_statfs,
    .init       = sfs_fuse_init,
    .destroy    = sfs_fuse_destroy,
};

int sfs_mount(sfs_t *fs, const char *mount_point, int argc, char *argv[]);
int sfs_unmount(sfs_t *fs);
```

### fsck (n)

```c
// n: Filesystem consistency checker
typedef struct {
    uint64_t errors_found;
    uint64_t errors_fixed;
    uint64_t lost_blocks;
    uint64_t orphan_inodes;
    bool consistent;
} sfs_fsck_result_t;

int sfs_fsck(sfs_t *fs, sfs_fsck_result_t *result, bool fix);
int sfs_fsck_superblock(sfs_t *fs, bool fix);
int sfs_fsck_bitmaps(sfs_t *fs, bool fix);
int sfs_fsck_inodes(sfs_t *fs, bool fix);
int sfs_fsck_directories(sfs_t *fs, bool fix);
int sfs_fsck_links(sfs_t *fs, bool fix);
```

### Journaling Bonus (o)

```c
// o: Write-ahead log
int sfs_journal_init(sfs_t *fs);
int sfs_journal_begin(sfs_t *fs, uint32_t *txn_id);
int sfs_journal_log_block(sfs_t *fs, uint32_t txn_id, uint64_t block, const void *data);
int sfs_journal_log_inode(sfs_t *fs, uint32_t txn_id, uint64_t inode, const sfs_inode_t *data);
int sfs_journal_commit(sfs_t *fs, uint32_t txn_id);
int sfs_journal_abort(sfs_t *fs, uint32_t txn_id);
int sfs_journal_recover(sfs_t *fs);
int sfs_journal_checkpoint(sfs_t *fs);
```

### Extents Bonus (q)

```c
// q: Range-based allocation
int sfs_extent_alloc(sfs_t *fs, sfs_inode_t *inode, uint64_t logical, uint32_t count);
int sfs_extent_lookup(sfs_t *fs, sfs_inode_t *inode, uint64_t logical, sfs_extent_t *extent);
int sfs_extent_free(sfs_t *fs, sfs_inode_t *inode, uint64_t logical, uint32_t count);
void sfs_extent_defrag(sfs_t *fs, uint64_t inode_num);
```

---

## Exemple d'Utilisation

```c
#include "simplefs.h"

int main(int argc, char *argv[]) {
    sfs_t *fs;

    // Creer un nouveau filesystem de 100MB
    fs = sfs_create("disk.img", 100);
    if (!fs) {
        fprintf(stderr, "Failed to create filesystem\n");
        return 1;
    }

    // j: CRUD operations
    // Creer des fichiers et repertoires
    sfs_create_dir(fs, "/home", 0755);
    sfs_create_dir(fs, "/home/user", 0755);
    sfs_create_file(fs, "/home/user/hello.txt", 0644);

    // Ecrire des donnees
    const char *data = "Hello, SimpleFS!\n";
    sfs_write_file(fs, "/home/user/hello.txt", data, strlen(data), 0);

    // Lire des donnees
    char buf[1024];
    ssize_t n = sfs_read_file(fs, "/home/user/hello.txt", buf, sizeof(buf), 0);
    buf[n] = '\0';
    printf("Read: %s", buf);

    // h: Hard link
    sfs_link(fs, "/home/user/hello.txt", "/home/user/hello_link.txt");

    // p: Symbolic link (bonus)
    sfs_symlink(fs, "hello.txt", "/home/user/hello_sym.txt");

    // k: Change permissions
    sfs_chmod(fs, "/home/user/hello.txt", 0600);

    // i: Path resolution - list directory
    sfs_dirent_t entries[100];
    size_t count;
    sfs_dir_list(fs, 0, entries, 100, &count);  // 0 = use path

    // l: Sync to disk
    sfs_sync(fs);

    // n: Run fsck
    sfs_fsck_result_t result;
    sfs_fsck(fs, &result, true);
    printf("fsck: %lu errors, %s\n", result.errors_found,
           result.consistent ? "consistent" : "inconsistent");

    // m: Mount via FUSE
    printf("Mounting at /mnt/sfs...\n");
    if (fork() == 0) {
        char *fuse_argv[] = {"simplefs", "-f", "/mnt/sfs", NULL};
        sfs_mount(fs, "/mnt/sfs", 3, fuse_argv);
        exit(0);
    }

    // Now accessible via normal filesystem commands:
    // $ ls /mnt/sfs/home/user/
    // $ cat /mnt/sfs/home/user/hello.txt
    // $ echo "more data" >> /mnt/sfs/home/user/hello.txt

    sfs_close(fs);
    return 0;
}
```

---

## Programme CLI

```c
// simplefs-cli.c - Outil en ligne de commande

void usage(void) {
    printf("Usage: simplefs <command> [options]\n\n");
    printf("Commands:\n");
    printf("  mkfs <image> <size_mb>     Create new filesystem\n");
    printf("  mount <image> <mountpoint> Mount via FUSE\n");
    printf("  fsck <image> [-f]          Check (and fix) filesystem\n");
    printf("  info <image>               Show filesystem info\n");
    printf("  ls <image> <path>          List directory\n");
    printf("  cat <image> <path>         Show file contents\n");
    printf("  cp <image> <src> <dst>     Copy file into/from FS\n");
    printf("  rm <image> <path>          Remove file\n");
    printf("  mkdir <image> <path>       Create directory\n");
    printf("  dump <image>               Dump filesystem structure\n");
}

int main(int argc, char *argv[]) {
    if (argc < 2) {
        usage();
        return 1;
    }

    if (strcmp(argv[1], "mkfs") == 0) {
        // Create filesystem
        sfs_t *fs = sfs_create(argv[2], atoi(argv[3]));
        printf("Created %s (%s MB)\n", argv[2], argv[3]);
        sfs_close(fs);
    }
    else if (strcmp(argv[1], "mount") == 0) {
        // Mount via FUSE
        sfs_t *fs = sfs_open(argv[2]);
        sfs_mount(fs, argv[3], argc - 2, argv + 2);
    }
    else if (strcmp(argv[1], "fsck") == 0) {
        // Check filesystem
        sfs_t *fs = sfs_open(argv[2]);
        sfs_fsck_result_t result;
        bool fix = (argc > 3 && strcmp(argv[3], "-f") == 0);
        sfs_fsck(fs, &result, fix);
        printf("Errors: %lu, Fixed: %lu\n", result.errors_found, result.errors_fixed);
        sfs_close(fs);
    }
    // ... other commands

    return 0;
}
```

---

## Tests Moulinette

```rust
// tests/test_simplefs.rs

mod disk_tests {
    #[test] fn test_disk_create()           // a
    #[test] fn test_disk_read_write()       // a
    #[test] fn test_disk_persistence()      // l
}

mod superblock_tests {
    #[test] fn test_superblock_init()       // b
    #[test] fn test_superblock_load_save()  // b
}

mod bitmap_tests {
    #[test] fn test_inode_bitmap()          // e
    #[test] fn test_block_bitmap()          // d
    #[test] fn test_alloc_free_cycle()      // d, e
}

mod inode_tests {
    #[test] fn test_inode_create()          // c
    #[test] fn test_inode_direct_blocks()   // g
    #[test] fn test_inode_indirect()        // g
    #[test] fn test_inode_double_indirect() // g
    #[test] fn test_inode_truncate()        // g
}

mod directory_tests {
    #[test] fn test_dir_add_entry()         // f
    #[test] fn test_dir_remove_entry()      // f
    #[test] fn test_dir_lookup()            // f
    #[test] fn test_dir_list()              // f
}

mod path_tests {
    #[test] fn test_path_resolve()          // i
    #[test] fn test_path_resolve_deep()     // i
    #[test] fn test_path_parent()           // i
}

mod crud_tests {
    #[test] fn test_create_file()           // j
    #[test] fn test_read_write_file()       // j
    #[test] fn test_create_delete_dir()     // j
    #[test] fn test_large_file()            // j
}

mod link_tests {
    #[test] fn test_hard_link()             // h
    #[test] fn test_hard_link_count()       // h
    #[test] fn test_symlink()               // p (bonus)
    #[test] fn test_readlink()              // p (bonus)
}

mod permission_tests {
    #[test] fn test_chmod()                 // k
    #[test] fn test_chown()                 // k
    #[test] fn test_access()                // k
}

mod fuse_tests {
    #[test] fn test_fuse_mount()            // m
    #[test] fn test_fuse_operations()       // m
    #[test] fn test_fuse_concurrent()       // m
}

mod fsck_tests {
    #[test] fn test_fsck_clean()            // n
    #[test] fn test_fsck_corrupted()        // n
    #[test] fn test_fsck_fix()              // n
}

mod journal_tests {
    #[test] fn test_journal_basic()         // o (bonus)
    #[test] fn test_journal_recovery()      // o (bonus)
    #[test] fn test_crash_consistency()     // o (bonus)
}

mod extent_tests {
    #[test] fn test_extent_alloc()          // q (bonus)
    #[test] fn test_extent_large_file()     // q (bonus)
}

mod stress_tests {
    #[test] fn test_many_files()
    #[test] fn test_deep_directories()
    #[test] fn test_concurrent_access()
    #[test] fn test_disk_full()
}
```

---

## Bareme

| Critere | Points |
|---------|--------|
| **Core Filesystem** | |
| Disk abstraction (a) | 5 |
| Superblock (b) | 5 |
| Inode table (c) | 10 |
| Block bitmap (d) | 5 |
| Inode bitmap (e) | 5 |
| Directories (f) | 10 |
| Regular files with blocks (g) | 10 |
| Hard links (h) | 5 |
| Path resolution (i) | 5 |
| CRUD operations (j) | 10 |
| Permissions (k) | 5 |
| Persistence (l) | 5 |
| **Integration** | |
| FUSE mount (m) | 10 |
| fsck (n) | 10 |
| **Bonus** | |
| Journaling (o) | +10 |
| Symbolic links (p) | +5 |
| Extents (q) | +5 |
| **Total** | **100** (+20 bonus) |

---

## Fichiers

```
PROJET_2.3_SimpleFS/
├── include/
│   ├── simplefs.h          # API principale
│   ├── disk.h              # a: Abstraction disque
│   ├── superblock.h        # b: Superblock
│   ├── inode.h             # c: Inodes
│   ├── bitmap.h            # d, e: Bitmaps
│   ├── directory.h         # f: Directories
│   ├── path.h              # i: Path resolution
│   ├── journal.h           # o: Journaling (bonus)
│   └── extent.h            # q: Extents (bonus)
├── src/
│   ├── disk.c              # a
│   ├── superblock.c        # b
│   ├── inode.c             # c, g
│   ├── bitmap.c            # d, e
│   ├── directory.c         # f
│   ├── path.c              # i
│   ├── file.c              # j: CRUD
│   ├── link.c              # h, p
│   ├── permission.c        # k
│   ├── persist.c           # l
│   ├── fuse.c              # m
│   ├── fsck.c              # n
│   ├── journal.c           # o (bonus)
│   └── extent.c            # q (bonus)
├── tools/
│   ├── mkfs.simplefs.c     # Creer FS
│   ├── fsck.simplefs.c     # Verifier FS
│   ├── simplefs-cli.c      # CLI complet
│   └── mount.simplefs.c    # Mount helper
├── tests/
│   └── ...
├── Makefile
└── README.md
```

---

## Makefile

```makefile
CC = gcc
CFLAGS = -Wall -Wextra -std=c17 -I include
CFLAGS += $(shell pkg-config --cflags fuse3)
LDFLAGS = $(shell pkg-config --libs fuse3)

SRCS = $(wildcard src/*.c)
OBJS = $(SRCS:.c=.o)

all: libsimplefs.a simplefs mkfs.simplefs fsck.simplefs

libsimplefs.a: $(OBJS)
	ar rcs $@ $^

simplefs: tools/simplefs-cli.c libsimplefs.a
	$(CC) $(CFLAGS) -o $@ $< -L. -lsimplefs $(LDFLAGS)

mkfs.simplefs: tools/mkfs.simplefs.c libsimplefs.a
	$(CC) $(CFLAGS) -o $@ $< -L. -lsimplefs

fsck.simplefs: tools/fsck.simplefs.c libsimplefs.a
	$(CC) $(CFLAGS) -o $@ $< -L. -lsimplefs

clean:
	rm -f $(OBJS) libsimplefs.a simplefs mkfs.simplefs fsck.simplefs

.PHONY: all clean
```

---

## Criteres de Qualite

1. **Robustesse**: Gestion complete des erreurs
2. **Performance**: Cache efficace, operations O(1) ou O(log n)
3. **Compatibilite**: Fonctionne avec les outils standard (ls, cp, cat...)
4. **Testabilite**: Couverture de test > 90%
5. **Documentation**: Code commente, README complet
6. **Securite**: Pas de buffer overflows, validation des inputs

---

## Ressources

- Linux VFS documentation
- ext2 specification (simplifiee)
- FUSE API documentation
- "Operating Systems: Three Easy Pieces" - File Systems chapters
