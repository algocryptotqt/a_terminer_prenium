# ex09: Mini VFS Layer

**Module**: 2.3 - File Systems
**Difficulte**: Tres difficile
**Duree**: 12h
**Score qualite**: 97/100

## Concepts Couverts

### 2.3.18: Virtual File System (10 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | VFS purpose | Interface abstraite |
| b | VFS objects | superblock, inode, dentry, file |
| c | Operations structs | Pointeurs de fonctions |
| d | super_operations | Ops niveau FS |
| e | inode_operations | Ops sur inodes |
| f | file_operations | Ops sur fichiers |
| g | dentry cache | Cache nom->inode |
| h | Inode cache | Cache inodes memoire |
| i | Path lookup | namei() resolution |
| j | Mount points | Traversee FS |

### 2.3.19: Mount and Unmount (11 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Mounting | Attacher FS a l'arbre |
| b | Mount point | Repertoire cible |
| c | mount() syscall | Syscall mount |
| d | Mount flags | ro, noexec, nosuid |
| e | Mount table | /proc/mounts |
| f | /etc/fstab | Config boot |
| g | umount() | Detacher FS |
| h | Busy filesystem | Refus unmount |
| i | Lazy unmount | Detach + cleanup |
| j | Bind mount | Mount ailleurs |
| k | Mount namespaces | Vue par process |

---

## Sujet

Implementer une couche VFS complete supportant plusieurs types de filesystems.

### Structures VFS (2.3.18.b)

```c
// Forward declarations
struct vfs_superblock;
struct vfs_inode;
struct vfs_dentry;
struct vfs_file;

// 2.3.18.d: super_operations
typedef struct {
    struct vfs_inode* (*alloc_inode)(struct vfs_superblock *sb);
    void (*destroy_inode)(struct vfs_inode *inode);
    int (*write_inode)(struct vfs_inode *inode);
    int (*sync_fs)(struct vfs_superblock *sb);
    int (*statfs)(struct vfs_superblock *sb, struct statfs *buf);
} super_operations_t;

// 2.3.18.e: inode_operations
typedef struct {
    struct vfs_dentry* (*lookup)(struct vfs_inode *dir, const char *name);
    int (*create)(struct vfs_inode *dir, const char *name, mode_t mode);
    int (*mkdir)(struct vfs_inode *dir, const char *name, mode_t mode);
    int (*unlink)(struct vfs_inode *dir, const char *name);
    int (*rmdir)(struct vfs_inode *dir, const char *name);
    int (*link)(struct vfs_dentry *old, struct vfs_inode *dir, const char *name);
    int (*symlink)(struct vfs_inode *dir, const char *name, const char *target);
    int (*rename)(struct vfs_inode *old_dir, const char *old,
                  struct vfs_inode *new_dir, const char *new);
} inode_operations_t;

// 2.3.18.f: file_operations
typedef struct {
    int (*open)(struct vfs_inode *inode, struct vfs_file *file);
    int (*release)(struct vfs_inode *inode, struct vfs_file *file);
    ssize_t (*read)(struct vfs_file *file, char *buf, size_t count, off_t *offset);
    ssize_t (*write)(struct vfs_file *file, const char *buf, size_t count, off_t *offset);
    off_t (*lseek)(struct vfs_file *file, off_t offset, int whence);
    int (*readdir)(struct vfs_file *file, void *dirent, int (*filldir)(void*, const char*, int, off_t, ino_t, unsigned));
    int (*fsync)(struct vfs_file *file);
} file_operations_t;

// 2.3.18.b: VFS objects
typedef struct vfs_superblock {
    uint32_t magic;
    uint32_t block_size;
    struct vfs_inode *root;
    super_operations_t *s_op;        // d: super_operations
    void *fs_info;                   // FS-specific data
    char fs_type[32];
} vfs_superblock_t;

typedef struct vfs_inode {
    ino_t ino;
    mode_t mode;
    uid_t uid;
    gid_t gid;
    size_t size;
    time_t atime, mtime, ctime;
    uint32_t nlink;
    vfs_superblock_t *sb;
    inode_operations_t *i_op;        // e: inode_operations
    file_operations_t *f_op;         // f: file_operations
    void *i_private;
} vfs_inode_t;

typedef struct vfs_dentry {
    char name[256];
    vfs_inode_t *inode;
    struct vfs_dentry *parent;
    struct vfs_dentry *children;     // Linked list
    struct vfs_dentry *next;         // Sibling
    uint32_t ref_count;
} vfs_dentry_t;

typedef struct vfs_file {
    vfs_dentry_t *dentry;
    vfs_inode_t *inode;
    off_t offset;
    int flags;
    file_operations_t *f_op;
} vfs_file_t;

// 2.3.18.g-h: Caches
typedef struct {
    vfs_dentry_t **buckets;          // g: dentry cache hash table
    uint32_t size;
} dcache_t;

typedef struct {
    vfs_inode_t **buckets;           // h: inode cache hash table
    uint32_t size;
} icache_t;

// 2.3.19: Mount structures
typedef struct vfs_mount {
    vfs_superblock_t *sb;
    vfs_dentry_t *mountpoint;        // b: Mount point
    uint32_t flags;                  // d: Mount flags
    struct vfs_mount *next;
} vfs_mount_t;

#define MNT_RDONLY   0x01            // d: ro
#define MNT_NOEXEC   0x02            // d: noexec
#define MNT_NOSUID   0x04            // d: nosuid
#define MNT_BIND     0x08            // j: bind mount
#define MNT_LAZY     0x10            // i: lazy unmount
```

### API VFS

```c
// Lifecycle
int vfs_init(void);
void vfs_shutdown(void);

// FS registration (2.3.18.c)
int vfs_register_fs(const char *name, super_operations_t *ops,
                    inode_operations_t *i_ops, file_operations_t *f_ops);
int vfs_unregister_fs(const char *name);

// Mount operations (2.3.19)
int vfs_mount(const char *source, const char *target,   // a,b
              const char *fstype, uint32_t flags);      // c,d
int vfs_umount(const char *target);                     // g
int vfs_umount_lazy(const char *target);                // i
int vfs_bind_mount(const char *source, const char *target); // j
int vfs_mount_table(char *buf, size_t size);            // e
int vfs_parse_fstab(const char *path);                  // f
bool vfs_is_busy(const char *target);                   // h

// Path resolution (2.3.18.i)
vfs_dentry_t *vfs_lookup(const char *path);             // i: namei()
vfs_inode_t *vfs_path_to_inode(const char *path);

// Cross mount points (2.3.18.j)
vfs_mount_t *vfs_get_mount(vfs_dentry_t *dentry);
bool vfs_is_mountpoint(vfs_dentry_t *dentry);

// File operations via VFS
int vfs_open(const char *path, int flags, vfs_file_t **file);
int vfs_close(vfs_file_t *file);
ssize_t vfs_read(vfs_file_t *file, void *buf, size_t count);
ssize_t vfs_write(vfs_file_t *file, const void *buf, size_t count);
int vfs_mkdir(const char *path, mode_t mode);
int vfs_unlink(const char *path);

// Cache operations (2.3.18.g-h)
void dcache_init(dcache_t *dc, uint32_t size);
vfs_dentry_t *dcache_lookup(dcache_t *dc, vfs_dentry_t *parent, const char *name);
void dcache_insert(dcache_t *dc, vfs_dentry_t *dentry);
void dcache_invalidate(dcache_t *dc, vfs_dentry_t *dentry);

void icache_init(icache_t *ic, uint32_t size);
vfs_inode_t *icache_lookup(icache_t *ic, ino_t ino);
void icache_insert(icache_t *ic, vfs_inode_t *inode);

// Mount namespaces (2.3.19.k)
int vfs_create_namespace(void);
int vfs_enter_namespace(int ns_id);
```

---

## Exemple

```c
// Definir un FS simple en memoire
static vfs_inode_t *memfs_alloc_inode(vfs_superblock_t *sb) { ... }
static int memfs_create(vfs_inode_t *dir, const char *name, mode_t mode) { ... }
static ssize_t memfs_read(vfs_file_t *file, char *buf, size_t n, off_t *off) { ... }

super_operations_t memfs_s_ops = { .alloc_inode = memfs_alloc_inode, ... };
inode_operations_t memfs_i_ops = { .create = memfs_create, ... };
file_operations_t memfs_f_ops = { .read = memfs_read, ... };

int main(void) {
    vfs_init();

    // Enregistrer FS (2.3.18.c)
    vfs_register_fs("memfs", &memfs_s_ops, &memfs_i_ops, &memfs_f_ops);

    // Monter (2.3.19.a-d)
    vfs_mount("none", "/mnt/mem", "memfs", MNT_NOEXEC);

    // Path lookup (2.3.18.i)
    vfs_dentry_t *d = vfs_lookup("/mnt/mem/file.txt");

    // Verifier mount point (2.3.18.j)
    if (vfs_is_mountpoint(d)) { ... }

    // Operations fichier via VFS
    vfs_file_t *f;
    vfs_open("/mnt/mem/test", O_CREAT|O_RDWR, &f);
    vfs_write(f, "hello", 5);
    vfs_close(f);

    // Mount table (2.3.19.e)
    char table[1024];
    vfs_mount_table(table, sizeof(table));
    printf("%s", table);

    // Unmount (2.3.19.g-h)
    if (vfs_is_busy("/mnt/mem")) {
        vfs_umount_lazy("/mnt/mem");  // i: lazy
    } else {
        vfs_umount("/mnt/mem");
    }

    vfs_shutdown();
    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_vfs_objects()           // 2.3.18.b
#[test] fn test_operations_structs()    // 2.3.18.c-f
#[test] fn test_dcache()                // 2.3.18.g
#[test] fn test_icache()                // 2.3.18.h
#[test] fn test_path_lookup()           // 2.3.18.i
#[test] fn test_mount_crossing()        // 2.3.18.j
#[test] fn test_mount()                 // 2.3.19.a-d
#[test] fn test_mount_table()           // 2.3.19.e
#[test] fn test_fstab()                 // 2.3.19.f
#[test] fn test_umount()                // 2.3.19.g-h
#[test] fn test_lazy_umount()           // 2.3.19.i
#[test] fn test_bind_mount()            // 2.3.19.j
#[test] fn test_namespaces()            // 2.3.19.k
```

---

## Bareme

| Critere | Points |
|---------|--------|
| VFS objects (2.3.18.b) | 10 |
| Operations structs (2.3.18.c-f) | 25 |
| Caches (2.3.18.g-h) | 15 |
| Path lookup (2.3.18.i-j) | 15 |
| Mount/Unmount (2.3.19.a-i) | 25 |
| Bind + Namespaces (2.3.19.j-k) | 10 |
| **Total** | **100** |

---

## Fichiers

```
ex09/
├── vfs.h
├── vfs_core.c
├── vfs_mount.c
├── vfs_cache.c
├── vfs_lookup.c
├── memfs.c (exemple FS)
└── Makefile
```
