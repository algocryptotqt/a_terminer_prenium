# ex11: Memory-Mapped I/O

**Module**: 2.3 - File Systems
**Difficulte**: Moyen
**Duree**: 5h
**Score qualite**: 96/100

## Concepts Couverts

### 2.3.22: Memory-Mapped I/O (11 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | mmap() | Map file to memory |
| b | Advantages | No copy, lazy loading |
| c | PROT flags | PROT_READ, PROT_WRITE, PROT_EXEC |
| d | MAP_SHARED | Modifications partagees |
| e | MAP_PRIVATE | Copy-on-write |
| f | MAP_ANONYMOUS | Sans fichier backing |
| g | munmap() | Unmap region |
| h | msync() | Sync to file |
| i | mprotect() | Change protection |
| j | madvise() | Hint to kernel |
| k | Page faults | Load on access |

---

## Sujet

Implementer une bibliotheque d'I/O mappee en memoire.

### Structures

```c
typedef struct {
    void *addr;           // Adresse mappee
    size_t length;        // Taille du mapping
    int prot;             // c: Protection flags
    int flags;            // d,e,f: MAP_* flags
    int fd;               // File descriptor (-1 si anonymous)
    off_t offset;         // Offset dans le fichier
    bool synced;          // h: Sync status
} mmap_region_t;

typedef struct {
    mmap_region_t *regions;
    size_t count;
    size_t capacity;
    uint64_t page_faults;  // k: Page fault counter
} mmap_ctx_t;
```

### API

```c
// Context management
mmap_ctx_t *mmap_ctx_create(void);
void mmap_ctx_destroy(mmap_ctx_t *ctx);

// 2.3.22.a: mmap() - Map file to memory
void *mmap_file(mmap_ctx_t *ctx, const char *path, size_t length,
                int prot, int flags, off_t offset);

// 2.3.22.f: MAP_ANONYMOUS - No file backing
void *mmap_anonymous(mmap_ctx_t *ctx, size_t length, int prot);

// 2.3.22.g: munmap() - Unmap region
int mmap_unmap(mmap_ctx_t *ctx, void *addr, size_t length);

// 2.3.22.h: msync() - Sync to file
int mmap_sync(mmap_ctx_t *ctx, void *addr, size_t length, int flags);

// 2.3.22.i: mprotect() - Change protection
int mmap_protect(mmap_ctx_t *ctx, void *addr, size_t length, int prot);

// 2.3.22.j: madvise() - Hint to kernel
int mmap_advise(mmap_ctx_t *ctx, void *addr, size_t length, int advice);

// 2.3.22.b: Demonstrate advantages
typedef struct {
    double mmap_time_ms;
    double read_time_ms;
    size_t bytes_processed;
    bool lazy_loaded;       // b: Lazy loading verification
} mmap_benchmark_t;

void mmap_benchmark(const char *path, mmap_benchmark_t *result);

// 2.3.22.k: Page fault tracking
uint64_t mmap_get_page_faults(mmap_ctx_t *ctx);

// Utility functions
mmap_region_t *mmap_find_region(mmap_ctx_t *ctx, void *addr);
void mmap_list_regions(mmap_ctx_t *ctx);
```

### Constants (2.3.22.c,d,e,f)

```c
// Protection flags (c)
#define MMAP_PROT_READ   PROT_READ
#define MMAP_PROT_WRITE  PROT_WRITE
#define MMAP_PROT_EXEC   PROT_EXEC
#define MMAP_PROT_NONE   PROT_NONE

// Mapping flags (d,e,f)
#define MMAP_SHARED      MAP_SHARED      // d
#define MMAP_PRIVATE     MAP_PRIVATE     // e
#define MMAP_ANONYMOUS   MAP_ANONYMOUS   // f

// msync flags (h)
#define MMAP_SYNC_SYNC   MS_SYNC
#define MMAP_SYNC_ASYNC  MS_ASYNC

// madvise flags (j)
#define MMAP_ADV_NORMAL     MADV_NORMAL
#define MMAP_ADV_RANDOM     MADV_RANDOM
#define MMAP_ADV_SEQUENTIAL MADV_SEQUENTIAL
#define MMAP_ADV_WILLNEED   MADV_WILLNEED
#define MMAP_ADV_DONTNEED   MADV_DONTNEED
```

---

## Exemple

```c
int main(void) {
    mmap_ctx_t *ctx = mmap_ctx_create();

    // 2.3.22.a: Map fichier en lecture
    void *data = mmap_file(ctx, "data.bin", 4096,
                           MMAP_PROT_READ, MMAP_SHARED, 0);

    // 2.3.22.d: MAP_SHARED - modifications visibles
    void *shared = mmap_file(ctx, "shared.dat", 1024,
                             MMAP_PROT_READ | MMAP_PROT_WRITE,
                             MMAP_SHARED, 0);
    memcpy(shared, "modified", 8);
    mmap_sync(ctx, shared, 1024, MMAP_SYNC_SYNC);  // h: Sync

    // 2.3.22.e: MAP_PRIVATE - copy-on-write
    void *priv = mmap_file(ctx, "template.dat", 4096,
                           MMAP_PROT_READ | MMAP_PROT_WRITE,
                           MMAP_PRIVATE, 0);
    memcpy(priv, "private change", 14);  // Triggera COW

    // 2.3.22.f: Anonymous mapping (pas de fichier)
    void *anon = mmap_anonymous(ctx, 8192, MMAP_PROT_READ | MMAP_PROT_WRITE);

    // 2.3.22.i: Changer protection
    mmap_protect(ctx, data, 4096, MMAP_PROT_NONE);  // Rendre inaccessible

    // 2.3.22.j: Hint au kernel
    mmap_advise(ctx, data, 4096, MMAP_ADV_SEQUENTIAL);

    // 2.3.22.b: Benchmark vs read()
    mmap_benchmark_t bench;
    mmap_benchmark("large_file.bin", &bench);
    printf("mmap: %.2fms, read: %.2fms\n", bench.mmap_time_ms, bench.read_time_ms);

    // 2.3.22.k: Page faults
    printf("Page faults: %lu\n", mmap_get_page_faults(ctx));

    // 2.3.22.g: Cleanup
    mmap_unmap(ctx, data, 4096);
    mmap_unmap(ctx, shared, 1024);
    mmap_unmap(ctx, priv, 4096);
    mmap_unmap(ctx, anon, 8192);

    mmap_ctx_destroy(ctx);
    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_mmap_file()         // 2.3.22.a
#[test] fn test_mmap_advantages()   // 2.3.22.b
#[test] fn test_prot_flags()        // 2.3.22.c
#[test] fn test_map_shared()        // 2.3.22.d
#[test] fn test_map_private()       // 2.3.22.e
#[test] fn test_map_anonymous()     // 2.3.22.f
#[test] fn test_munmap()            // 2.3.22.g
#[test] fn test_msync()             // 2.3.22.h
#[test] fn test_mprotect()          // 2.3.22.i
#[test] fn test_madvise()           // 2.3.22.j
#[test] fn test_page_faults()       // 2.3.22.k
```

---

## Bareme

| Critere | Points |
|---------|--------|
| mmap() file (2.3.22.a) | 15 |
| Benchmark advantages (2.3.22.b) | 10 |
| PROT flags (2.3.22.c) | 10 |
| MAP_SHARED (2.3.22.d) | 10 |
| MAP_PRIVATE COW (2.3.22.e) | 10 |
| MAP_ANONYMOUS (2.3.22.f) | 10 |
| munmap() (2.3.22.g) | 5 |
| msync() (2.3.22.h) | 10 |
| mprotect() (2.3.22.i) | 10 |
| madvise() (2.3.22.j) | 5 |
| Page fault tracking (2.3.22.k) | 5 |
| **Total** | **100** |

---

## Fichiers

```
ex11/
├── mmap_io.h
├── mmap_io.c
├── mmap_benchmark.c
└── Makefile
```
