# ex05: Block Allocator Simulator

**Module**: 2.3 - File Systems
**Difficulte**: Difficile
**Duree**: 8h
**Score qualite**: 96/100

## Concepts Couverts

### 2.3.8: Block Allocation (13 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Block: Unit of storage | Structure `block_t`, taille fixe |
| b | Block size: 1KB, 4KB | Configuration `BLOCK_SIZE` |
| c | Contiguous allocation | `alloc_contiguous()` |
| d | Contiguous problems | Detection fragmentation |
| e | Linked allocation | `alloc_linked()` avec chaines |
| f | Linked problems | Mesure acces sequentiel |
| g | FAT | `fat_t` File Allocation Table |
| h | Indexed allocation | `alloc_indexed()` |
| i | Multi-level indexed | Blocs indirects |
| j | Direct blocks | 12 pointeurs directs |
| k | Indirect | 1 niveau |
| l | Double indirect | 2 niveaux |
| m | Triple indirect | 3 niveaux |

### 2.3.9: Free Space Management (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Bitmap | `bitmap_t` 1 bit/bloc |
| b | Bitmap location | Offset fixe dans FS |
| c | Bitmap operations | `bitmap_find_free()`, `bitmap_alloc()`, `bitmap_free()` |
| d | Contiguous search | `bitmap_find_contiguous()` |
| e | Free list | `free_list_t` liste chainee |
| f | Grouping | Bloc de pointeurs |
| g | Counting | Paires (start, length) |

### 2.3.10: File System Layout (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Boot block | Bloc 0 reserve |
| b | Superblock | `superblock_t` metadonnees |
| c | Inode table | Zone fixe d'inodes |
| d | Data blocks | Zone de donnees |
| e | Block groups | Groupes pour localite |
| f | Backup superblocks | Copies de secours |
| g | Reserved blocks | Blocs reserves root |

### 2.3.11: Superblock (10 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Magic number | `0x42465321` identification |
| b | Block size | Taille en octets |
| c | Block count | Nombre total |
| d | Inode count | Nombre d'inodes |
| e | Free blocks | Blocs disponibles |
| f | Free inodes | Inodes disponibles |
| g | First data block | Apres metadonnees |
| h | Mount count | Compteur montage |
| i | State | Clean/dirty flag |
| j | Error behavior | Action sur erreur |

---

## Contexte

Les systemes de fichiers gerent l'espace disque via des strategies d'allocation de blocs. Cet exercice implemente un simulateur qui supporte TOUTES les strategies majeures.

---

## Sujet

Implementer un simulateur d'allocation de blocs complet.

### Structures requises

```c
// 2.3.11: Superblock
typedef struct {
    uint32_t magic;           // a: Magic number 0x42465321
    uint32_t block_size;      // b: Taille bloc
    uint32_t block_count;     // c: Nombre blocs
    uint32_t inode_count;     // d: Nombre inodes
    uint32_t free_blocks;     // e: Blocs libres
    uint32_t free_inodes;     // f: Inodes libres
    uint32_t first_data_block;// g: Premier bloc data
    uint32_t mount_count;     // h: Compteur montage
    uint8_t  state;           // i: CLEAN/DIRTY
    uint8_t  error_behavior;  // j: CONTINUE/REMOUNT_RO/PANIC
} superblock_t;

// 2.3.9.a-c: Bitmap
typedef struct {
    uint8_t *bits;
    uint32_t size;
} bitmap_t;

// 2.3.9.e: Free list
typedef struct free_node {
    uint32_t block;
    struct free_node *next;
} free_list_t;

// 2.3.8.g: FAT
typedef struct {
    uint32_t *table;   // FAT[i] = bloc suivant ou FAT_END
    uint32_t size;
} fat_t;

// 2.3.8.j-m: Inode avec pointeurs
typedef struct {
    uint32_t direct[12];      // j: 12 blocs directs
    uint32_t indirect;        // k: 1 indirect
    uint32_t double_indirect; // l: 2 indirects
    uint32_t triple_indirect; // m: 3 indirects
    uint32_t size;
    uint32_t blocks;
} inode_t;

// 2.3.10: Layout complet
typedef struct {
    superblock_t sb;          // b: Superblock
    superblock_t *backup_sb;  // f: Backups
    bitmap_t block_bitmap;
    bitmap_t inode_bitmap;
    inode_t *inode_table;     // c: Inode table
    uint8_t *data_blocks;     // d: Data blocks
    uint32_t *block_groups;   // e: Block groups
} filesystem_t;
```

### API requise

```c
// Filesystem lifecycle
filesystem_t *fs_create(uint32_t block_count, uint32_t block_size);
void fs_destroy(filesystem_t *fs);
int fs_mount(filesystem_t *fs);
int fs_unmount(filesystem_t *fs);

// 2.3.9: Free space management
int bitmap_init(bitmap_t *bm, uint32_t size);
int bitmap_alloc(bitmap_t *bm);                    // c: Allocate
int bitmap_free(bitmap_t *bm, uint32_t block);     // c: Free
int bitmap_find_free(bitmap_t *bm);                // c: Find free
int bitmap_find_contiguous(bitmap_t *bm, uint32_t n); // d: Contiguous
void free_list_init(free_list_t **list);           // e: Free list
int free_list_alloc(free_list_t **list);
int free_list_free(free_list_t **list, uint32_t block);

// 2.3.8: Allocation strategies
int alloc_contiguous(filesystem_t *fs, uint32_t n, uint32_t *start); // c
int alloc_linked(filesystem_t *fs, uint32_t n, uint32_t *first);     // e
int alloc_fat(fat_t *fat, uint32_t n, uint32_t *first);              // g
int alloc_indexed(inode_t *inode, filesystem_t *fs, uint32_t n);     // h-m

// Block access
int block_read(filesystem_t *fs, uint32_t block, void *buf);
int block_write(filesystem_t *fs, uint32_t block, const void *buf);

// Inode block resolution (2.3.8.j-m)
uint32_t inode_get_block(inode_t *inode, filesystem_t *fs, uint32_t logical);
int inode_set_block(inode_t *inode, filesystem_t *fs, uint32_t logical, uint32_t physical);

// Statistics
void fs_stats(filesystem_t *fs, fs_stats_t *stats);
float fs_fragmentation(filesystem_t *fs);
```

---

## Exemple d'utilisation

```c
int main(void) {
    // Creer FS de 1024 blocs de 4KB
    filesystem_t *fs = fs_create(1024, 4096);
    fs_mount(fs);

    // Allocation contiguee (2.3.8.c)
    uint32_t start;
    alloc_contiguous(fs, 10, &start);
    printf("Contiguous: blocks %u-%u\n", start, start + 9);

    // Allocation chainee avec FAT (2.3.8.g)
    uint32_t first;
    alloc_fat(&fs->fat, 5, &first);

    // Allocation indexee via inode (2.3.8.h-m)
    inode_t *inode = &fs->inode_table[0];
    alloc_indexed(inode, fs, 15000); // Necessite indirect blocks

    // Verifier bloc logique -> physique
    uint32_t phys = inode_get_block(inode, fs, 13);  // Direct (j)
    phys = inode_get_block(inode, fs, 1000);         // Indirect (k)
    phys = inode_get_block(inode, fs, 10000);        // Double (l)

    // Stats (2.3.11.e-f)
    printf("Free blocks: %u\n", fs->sb.free_blocks);
    printf("Fragmentation: %.2f%%\n", fs_fragmentation(fs) * 100);

    fs_unmount(fs);
    fs_destroy(fs);
    return 0;
}
```

---

## Fonctions Autorisees

`malloc`, `free`, `memset`, `memcpy`, `printf`

---

## Contraintes

- C17 (`-std=c17`)
- Pas de variables globales
- Max 40 lignes par fonction
- Tous les retours d'erreur verifies
- Valgrind clean

---

## Tests Moulinette

```rust
#[test] fn test_superblock_fields()      // 2.3.11.a-j
#[test] fn test_bitmap_operations()      // 2.3.9.a-c
#[test] fn test_bitmap_contiguous()      // 2.3.9.d
#[test] fn test_free_list()              // 2.3.9.e-g
#[test] fn test_contiguous_alloc()       // 2.3.8.c-d
#[test] fn test_linked_alloc()           // 2.3.8.e-f
#[test] fn test_fat_alloc()              // 2.3.8.g
#[test] fn test_direct_blocks()          // 2.3.8.j
#[test] fn test_indirect_blocks()        // 2.3.8.k
#[test] fn test_double_indirect()        // 2.3.8.l
#[test] fn test_triple_indirect()        // 2.3.8.m
#[test] fn test_fs_layout()              // 2.3.10.a-g
#[test] fn test_block_groups()           // 2.3.10.e
#[test] fn test_backup_superblocks()     // 2.3.10.f
#[test] fn test_fragmentation_metrics()
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Superblock complet (2.3.11) | 15 |
| Bitmap operations (2.3.9.a-d) | 15 |
| Free list/grouping (2.3.9.e-g) | 10 |
| Allocation contiguee (2.3.8.c-d) | 10 |
| Allocation chainee + FAT (2.3.8.e-g) | 15 |
| Direct blocks (2.3.8.j) | 10 |
| Indirect blocks (2.3.8.k-m) | 15 |
| FS Layout (2.3.10) | 10 |
| **Total** | **100** |

---

## Fichiers a rendre

```
ex05/
├── block_alloc.h
├── block_alloc.c
├── bitmap.c
├── fat.c
├── inode_blocks.c
├── fs_layout.c
└── Makefile
```
