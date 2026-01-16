# ex08: Modern Filesystem Analyzer

**Module**: 2.3 - File Systems
**Difficulte**: Moyen
**Duree**: 5h
**Score qualite**: 96/100

## Concepts Couverts

### 2.3.15: ext4 (11 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | History ext2->ext3->ext4 | Timeline affichee |
| b | ext2 | Simulation sans journal |
| c | ext3 | Ajout journaling |
| d | ext4 | Extents, fichiers larges |
| e | Block groups | Localite |
| f | Flex groups | Agregation |
| g | Extents | Remplacement block pointers |
| h | Multiblock allocation | Performance |
| i | Delayed allocation | Write-back |
| j | Persistent preallocation | Espace reserve |
| k | Journal checksum | Fiabilite |

### 2.3.16: Btrfs (12 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | COW filesystem | Par design |
| b | B-tree | Structure principale |
| c | Subvolumes | Arbres FS independants |
| d | Snapshots | Lecture seule |
| e | Clones | Ecriture possible |
| f | Checksums | Metadata + data |
| g | Compression | zlib/lzo/zstd |
| h | Deduplication | Offline |
| i | RAID | 0,1,5,6,10 |
| j | Scrub | Verification integrite |
| k | Balance | Redistribution |
| l | Send/receive | Backup incremental |

### 2.3.17: ZFS (12 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Pooled storage | zpools |
| b | Vdevs | Devices virtuels |
| c | Datasets | FS/volumes/snapshots |
| d | Copy-on-write | Toujours |
| e | Checksums | Tout |
| f | Self-healing | Avec redondance |
| g | ARC | Adaptive Replacement Cache |
| h | L2ARC | Cache niveau 2 |
| i | ZIL | Intent Log |
| j | SLOG | Log device separe |
| k | Deduplication | Online |
| l | Compression | Multiples algos |

---

## Sujet

Implementer un analyseur qui simule et compare les caracteristiques des 3 filesystems modernes.

### Structures

```c
// ext4 simulation
typedef struct {
    bool journaling;           // c: ext3 feature
    bool extents;              // d,g: ext4 feature
    uint32_t block_groups;     // e: Nombre groupes
    uint32_t flex_group_size;  // f: Taille flex group
    bool delayed_alloc;        // i: Delayed allocation
    bool prealloc;             // j: Preallocation
    uint32_t journal_checksum; // k: CRC du journal
} ext4_sim_t;

// Btrfs simulation
typedef struct {
    bool cow;                  // a: COW
    void *btree_root;          // b: B-tree
    uint32_t subvolume_count;  // c: Subvolumes
    uint32_t snapshot_count;   // d: Snapshots
    uint32_t clone_count;      // e: Clones
    uint32_t checksum_algo;    // f: CRC32/XXHASH
    uint32_t compression;      // g: ZLIB/LZO/ZSTD
    bool dedup_enabled;        // h: Dedup offline
    uint32_t raid_level;       // i: RAID niveau
    uint64_t last_scrub;       // j: Dernier scrub
    bool balanced;             // k: Balance done
} btrfs_sim_t;

// ZFS simulation
typedef struct {
    char pool_name[64];        // a: zpool
    uint32_t vdev_count;       // b: Nombre vdevs
    uint32_t dataset_count;    // c: Datasets
    bool cow;                  // d: COW
    uint32_t checksum_algo;    // e: Algo checksum
    bool self_healing;         // f: Auto-repair
    size_t arc_size;           // g: ARC size
    size_t l2arc_size;         // h: L2ARC size
    bool zil_enabled;          // i: ZIL active
    bool slog_present;         // j: SLOG device
    bool dedup_enabled;        // k: Dedup online
    uint32_t compression;      // l: Compression algo
} zfs_sim_t;

// Unified analyzer
typedef struct {
    ext4_sim_t ext4;
    btrfs_sim_t btrfs;
    zfs_sim_t zfs;
} fs_analyzer_t;
```

### API

```c
// Lifecycle
fs_analyzer_t *analyzer_create(void);
void analyzer_destroy(fs_analyzer_t *a);

// ext4 analysis (2.3.15)
void ext4_show_history(void);              // a: ext2->ext3->ext4
void ext4_init(ext4_sim_t *e, int version);// b,c,d
void ext4_enable_extents(ext4_sim_t *e);   // g
void ext4_set_block_groups(ext4_sim_t *e, uint32_t n); // e
void ext4_enable_flex_groups(ext4_sim_t *e, uint32_t size); // f
void ext4_multiblock_alloc(ext4_sim_t *e, uint32_t n); // h
void ext4_delayed_alloc(ext4_sim_t *e, bool enable); // i
void ext4_preallocate(ext4_sim_t *e, uint64_t size); // j
uint32_t ext4_journal_checksum(ext4_sim_t *e); // k

// Btrfs analysis (2.3.16)
void btrfs_init(btrfs_sim_t *b);           // a,b: COW + B-tree
int btrfs_create_subvolume(btrfs_sim_t *b, const char *name); // c
int btrfs_snapshot(btrfs_sim_t *b, const char *src, const char *dst); // d
int btrfs_clone(btrfs_sim_t *b, const char *src, const char *dst); // e
void btrfs_set_checksum(btrfs_sim_t *b, int algo); // f
void btrfs_set_compression(btrfs_sim_t *b, int algo); // g
int btrfs_dedup(btrfs_sim_t *b);           // h
void btrfs_set_raid(btrfs_sim_t *b, int level); // i
int btrfs_scrub(btrfs_sim_t *b);           // j
int btrfs_balance(btrfs_sim_t *b);         // k
int btrfs_send_receive(btrfs_sim_t *b, const char *snap, int fd); // l

// ZFS analysis (2.3.17)
int zfs_create_pool(zfs_sim_t *z, const char *name); // a
int zfs_add_vdev(zfs_sim_t *z, const char *dev); // b
int zfs_create_dataset(zfs_sim_t *z, const char *name); // c
void zfs_show_cow_stats(zfs_sim_t *z);     // d
void zfs_verify_checksums(zfs_sim_t *z);   // e
int zfs_self_heal(zfs_sim_t *z, uint32_t block); // f
void zfs_arc_stats(zfs_sim_t *z);          // g
void zfs_l2arc_stats(zfs_sim_t *z);        // h
void zfs_zil_stats(zfs_sim_t *z);          // i
int zfs_add_slog(zfs_sim_t *z, const char *dev); // j
int zfs_dedup_ratio(zfs_sim_t *z);         // k
void zfs_set_compression(zfs_sim_t *z, int algo); // l

// Comparison
void fs_compare_features(fs_analyzer_t *a);
void fs_compare_performance(fs_analyzer_t *a);
void fs_compare_reliability(fs_analyzer_t *a);
```

---

## Exemple

```c
int main(void) {
    fs_analyzer_t *a = analyzer_create();

    // ext4 history (2.3.15.a)
    ext4_show_history();
    // Output: ext2 (1993) -> ext3 (2001, +journal) -> ext4 (2008, +extents)

    // ext4 features
    ext4_init(&a->ext4, 4);               // Version 4
    ext4_enable_extents(&a->ext4);        // g: Extents
    ext4_set_block_groups(&a->ext4, 128); // e: Block groups

    // Btrfs
    btrfs_init(&a->btrfs);
    btrfs_create_subvolume(&a->btrfs, "home");      // c
    btrfs_snapshot(&a->btrfs, "home", "home_snap"); // d
    btrfs_set_compression(&a->btrfs, BTRFS_ZSTD);   // g
    btrfs_scrub(&a->btrfs);                         // j

    // ZFS
    zfs_create_pool(&a->zfs, "tank");     // a
    zfs_add_vdev(&a->zfs, "/dev/sda");    // b
    zfs_create_dataset(&a->zfs, "data");  // c
    zfs_arc_stats(&a->zfs);               // g

    // Compare
    fs_compare_features(a);

    analyzer_destroy(a);
    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_ext4_history()        // 2.3.15.a-d
#[test] fn test_ext4_block_groups()   // 2.3.15.e-f
#[test] fn test_ext4_extents()        // 2.3.15.g
#[test] fn test_ext4_alloc()          // 2.3.15.h-j
#[test] fn test_ext4_journal()        // 2.3.15.k
#[test] fn test_btrfs_cow()           // 2.3.16.a-b
#[test] fn test_btrfs_subvol()        // 2.3.16.c-e
#[test] fn test_btrfs_integrity()     // 2.3.16.f,j
#[test] fn test_btrfs_features()      // 2.3.16.g-i,k-l
#[test] fn test_zfs_pool()            // 2.3.17.a-c
#[test] fn test_zfs_integrity()       // 2.3.17.d-f
#[test] fn test_zfs_caching()         // 2.3.17.g-j
#[test] fn test_zfs_features()        // 2.3.17.k-l
#[test] fn test_comparison()
```

---

## Bareme

| Critere | Points |
|---------|--------|
| ext4 complet (2.3.15) | 30 |
| Btrfs complet (2.3.16) | 35 |
| ZFS complet (2.3.17) | 35 |
| **Total** | **100** |

---

## Fichiers

```
ex08/
├── fs_analyzer.h
├── ext4_sim.c
├── btrfs_sim.c
├── zfs_sim.c
└── Makefile
```
