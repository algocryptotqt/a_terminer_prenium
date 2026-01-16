# ex10: FUSE Filesystem

**Module**: 2.3 - File Systems
**Difficulte**: Tres difficile
**Duree**: 10h
**Score qualite**: 96/100

## Concepts Couverts

### 2.3.20: FUSE (9 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | FUSE concept | Filesystem in Userspace |
| b | Kernel module | /dev/fuse |
| c | libfuse | Bibliotheque utilisateur |
| d | Request handling | Kernel->user->kernel |
| e | fuse_operations | Callbacks structure |
| f | Low-level API | Plus de controle |
| g | High-level API | Plus simple |
| h | Performance | Overhead context switch |
| i | Use cases | Network, archive, encrypted |

### 2.3.21: FUSE Operations (12 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | getattr | stat() equivalent |
| b | readdir | List directory |
| c | open | Open file |
| d | read | Read data |
| e | write | Write data |
| f | create | Create file |
| g | unlink | Delete file |
| h | mkdir/rmdir | Directories |
| i | rename | Move/rename |
| j | truncate | Change size |
| k | chmod/chown | Permissions |
| l | symlink/readlink | Liens symboliques |

---

## Sujet

Implementer un filesystem FUSE complet avec support de toutes les operations.

### Structure FUSE (2.3.20.e)

```c
#define FUSE_USE_VERSION 31
#include <fuse3/fuse.h>

// Notre filesystem en memoire
typedef struct {
    char name[256];
    mode_t mode;
    uid_t uid;
    gid_t gid;
    size_t size;
    time_t atime, mtime, ctime;
    nlink_t nlink;
    char *data;              // Contenu fichier
    char *symlink_target;    // l: Pour symlinks
    struct myfs_node *children;
    struct myfs_node *next;
    struct myfs_node *parent;
} myfs_node_t;

typedef struct {
    myfs_node_t *root;
    size_t total_size;
    uint64_t inode_counter;
} myfs_t;
```

### Operations FUSE (2.3.21)

```c
// 2.3.21.a: getattr - stat() equivalent
static int myfs_getattr(const char *path, struct stat *stbuf,
                        struct fuse_file_info *fi);

// 2.3.21.b: readdir - List directory
static int myfs_readdir(const char *path, void *buf, fuse_fill_dir_t filler,
                        off_t offset, struct fuse_file_info *fi,
                        enum fuse_readdir_flags flags);

// 2.3.21.c: open - Open file
static int myfs_open(const char *path, struct fuse_file_info *fi);

// 2.3.21.d: read - Read data
static int myfs_read(const char *path, char *buf, size_t size, off_t offset,
                     struct fuse_file_info *fi);

// 2.3.21.e: write - Write data
static int myfs_write(const char *path, const char *buf, size_t size,
                      off_t offset, struct fuse_file_info *fi);

// 2.3.21.f: create - Create file
static int myfs_create(const char *path, mode_t mode,
                       struct fuse_file_info *fi);

// 2.3.21.g: unlink - Delete file
static int myfs_unlink(const char *path);

// 2.3.21.h: mkdir/rmdir - Directories
static int myfs_mkdir(const char *path, mode_t mode);
static int myfs_rmdir(const char *path);

// 2.3.21.i: rename - Move/rename
static int myfs_rename(const char *from, const char *to, unsigned int flags);

// 2.3.21.j: truncate - Change size
static int myfs_truncate(const char *path, off_t size,
                         struct fuse_file_info *fi);

// 2.3.21.k: chmod/chown - Permissions
static int myfs_chmod(const char *path, mode_t mode,
                      struct fuse_file_info *fi);
static int myfs_chown(const char *path, uid_t uid, gid_t gid,
                      struct fuse_file_info *fi);

// 2.3.21.l: symlink/readlink - Symbolic links
static int myfs_symlink(const char *target, const char *linkpath);
static int myfs_readlink(const char *path, char *buf, size_t size);

// Structure operations (2.3.20.e)
static const struct fuse_operations myfs_oper = {
    .getattr    = myfs_getattr,    // a
    .readdir    = myfs_readdir,    // b
    .open       = myfs_open,       // c
    .read       = myfs_read,       // d
    .write      = myfs_write,      // e
    .create     = myfs_create,     // f
    .unlink     = myfs_unlink,     // g
    .mkdir      = myfs_mkdir,      // h
    .rmdir      = myfs_rmdir,      // h
    .rename     = myfs_rename,     // i
    .truncate   = myfs_truncate,   // j
    .chmod      = myfs_chmod,      // k
    .chown      = myfs_chown,      // k
    .symlink    = myfs_symlink,    // l
    .readlink   = myfs_readlink,   // l
    .init       = myfs_init,
    .destroy    = myfs_destroy,
};
```

### API Helper

```c
// Node management
myfs_node_t *myfs_find_node(myfs_t *fs, const char *path);
myfs_node_t *myfs_create_node(myfs_t *fs, const char *path, mode_t mode);
int myfs_remove_node(myfs_t *fs, const char *path);

// Path utilities
char *myfs_parent_path(const char *path);
const char *myfs_basename(const char *path);

// Performance tracking (2.3.20.h)
typedef struct {
    uint64_t getattr_calls;
    uint64_t read_calls;
    uint64_t write_calls;
    uint64_t total_read_bytes;
    uint64_t total_write_bytes;
    double avg_latency_us;
} myfs_stats_t;

void myfs_get_stats(myfs_t *fs, myfs_stats_t *stats);
```

---

## Exemple

```c
// 2.3.20.g: High-level API usage
int main(int argc, char *argv[]) {
    // Init our filesystem
    myfs_t fs = {0};
    fs.root = create_root_node();

    // FUSE main loop (2.3.20.d: request handling)
    return fuse_main(argc, argv, &myfs_oper, &fs);
}

// Usage apres mount:
// $ ./myfs /mnt/myfs
// $ echo "hello" > /mnt/myfs/test.txt   # create + write
// $ cat /mnt/myfs/test.txt              # open + read
// $ ls -la /mnt/myfs/                   # readdir + getattr
// $ ln -s test.txt /mnt/myfs/link       # symlink
// $ readlink /mnt/myfs/link             # readlink
// $ fusermount -u /mnt/myfs             # unmount
```

---

## Use Cases (2.3.20.i)

L'exercice doit supporter au moins UN de ces cas d'usage:

1. **In-memory FS** (defaut) - FS en RAM
2. **Tar FS** - Monter un .tar comme FS (bonus)
3. **Encrypted FS** - Chiffrement transparent (bonus)

---

## Tests Moulinette

```rust
#[test] fn test_getattr()           // 2.3.21.a
#[test] fn test_readdir()           // 2.3.21.b
#[test] fn test_open_read_write()   // 2.3.21.c-e
#[test] fn test_create_unlink()     // 2.3.21.f-g
#[test] fn test_mkdir_rmdir()       // 2.3.21.h
#[test] fn test_rename()            // 2.3.21.i
#[test] fn test_truncate()          // 2.3.21.j
#[test] fn test_chmod_chown()       // 2.3.21.k
#[test] fn test_symlink()           // 2.3.21.l
#[test] fn test_fuse_operations()   // 2.3.20.e
#[test] fn test_performance()       // 2.3.20.h
```

---

## Bareme

| Critere | Points |
|---------|--------|
| getattr (2.3.21.a) | 10 |
| readdir (2.3.21.b) | 10 |
| open/read/write (2.3.21.c-e) | 20 |
| create/unlink (2.3.21.f-g) | 10 |
| mkdir/rmdir (2.3.21.h) | 10 |
| rename (2.3.21.i) | 10 |
| truncate (2.3.21.j) | 5 |
| chmod/chown (2.3.21.k) | 10 |
| symlink/readlink (2.3.21.l) | 10 |
| Performance tracking (2.3.20.h) | 5 |
| **Total** | **100** |

---

## Fichiers

```
ex10/
├── myfs.h
├── myfs.c
├── myfs_ops.c
├── myfs_node.c
└── Makefile
```

## Compilation

```makefile
CFLAGS = -Wall -Wextra -std=c17 $(shell pkg-config --cflags fuse3)
LDFLAGS = $(shell pkg-config --libs fuse3)

myfs: myfs.o myfs_ops.o myfs_node.o
	$(CC) -o $@ $^ $(LDFLAGS)
```
