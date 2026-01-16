# ex13: File Locking System

**Module**: 2.3 - File Systems
**Difficulte**: Moyen
**Duree**: 5h
**Score qualite**: 96/100

## Concepts Couverts

### 2.3.26: File Locking (13 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Advisory locks | Cooperation requise |
| b | Mandatory locks | Force par kernel |
| c | flock() | Lock fichier entier |
| d | LOCK_SH | Shared lock (lecture) |
| e | LOCK_EX | Exclusive lock (ecriture) |
| f | LOCK_NB | Non-blocking |
| g | fcntl() | Lock region |
| h | F_SETLK | Set lock non-blocking |
| i | F_SETLKW | Set lock blocking |
| j | F_GETLK | Get lock info |
| k | struct flock | Region specification |
| l | Deadlock | Detection evitement |
| m | Lock inheritance | Fork/exec behavior |

---

## Sujet

Implementer un systeme complet de verrouillage de fichiers supportant les deux APIs (flock et fcntl).

### Structures

```c
// 2.3.26.k: struct flock wrapper
typedef struct {
    short l_type;        // d,e: LOCK_SH, LOCK_EX, F_UNLCK
    short l_whence;      // SEEK_SET, SEEK_CUR, SEEK_END
    off_t l_start;       // Debut region
    off_t l_len;         // Longueur (0 = jusqu'a EOF)
    pid_t l_pid;         // PID du holder
} lock_region_t;

// Lock entry
typedef struct lock_entry {
    int fd;
    lock_region_t region;
    bool is_flock;       // c: flock() vs fcntl()
    struct lock_entry *next;
} lock_entry_t;

// Lock manager context
typedef struct {
    lock_entry_t *locks;
    size_t lock_count;
    bool deadlock_detection;  // l: Detection deadlock
    uint64_t blocked_count;
    uint64_t granted_count;
} lock_manager_t;

// Deadlock graph (2.3.26.l)
typedef struct {
    pid_t waiter;
    pid_t holder;
} wait_edge_t;

typedef struct {
    wait_edge_t *edges;
    size_t edge_count;
    size_t capacity;
} deadlock_graph_t;
```

### API

```c
// Context management
lock_manager_t *lock_manager_create(void);
void lock_manager_destroy(lock_manager_t *mgr);

// 2.3.26.c-f: flock() API
int lock_flock(lock_manager_t *mgr, int fd, int operation);
// operation: LOCK_SH (d), LOCK_EX (e), LOCK_UN, LOCK_NB (f)

// 2.3.26.g-k: fcntl() API
int lock_fcntl(lock_manager_t *mgr, int fd, int cmd, lock_region_t *region);
// cmd: F_SETLK (h), F_SETLKW (i), F_GETLK (j)

// 2.3.26.a vs b: Lock type
typedef enum {
    LOCK_ADVISORY,    // a: Advisory
    LOCK_MANDATORY    // b: Mandatory
} lock_mode_t;

int lock_set_mode(lock_manager_t *mgr, int fd, lock_mode_t mode);

// 2.3.26.l: Deadlock detection
void lock_enable_deadlock_detection(lock_manager_t *mgr, bool enable);
bool lock_detect_deadlock(lock_manager_t *mgr);
void lock_get_deadlock_cycle(lock_manager_t *mgr, pid_t *cycle, size_t *len);

// 2.3.26.m: Lock inheritance
typedef enum {
    INHERIT_NONE,      // Locks released on fork
    INHERIT_COPY,      // Locks copied to child
    INHERIT_SHARED     // Locks shared with child
} inherit_mode_t;

void lock_set_inherit_mode(lock_manager_t *mgr, inherit_mode_t mode);
int lock_on_fork(lock_manager_t *mgr, pid_t child_pid);
int lock_on_exec(lock_manager_t *mgr);

// Query functions
bool lock_is_locked(lock_manager_t *mgr, int fd, off_t start, off_t len);
int lock_get_holders(lock_manager_t *mgr, int fd, pid_t *holders, size_t max);
void lock_list_all(lock_manager_t *mgr);

// Statistics
typedef struct {
    uint64_t flock_calls;      // c
    uint64_t fcntl_calls;      // g
    uint64_t shared_locks;     // d
    uint64_t exclusive_locks;  // e
    uint64_t nonblock_fails;   // f
    uint64_t deadlocks_detected; // l
    double avg_wait_time_ms;
} lock_stats_t;

void lock_get_stats(lock_manager_t *mgr, lock_stats_t *stats);
```

### Constants

```c
// flock() operations (c-f)
#define MY_LOCK_SH     0x01    // d: Shared
#define MY_LOCK_EX     0x02    // e: Exclusive
#define MY_LOCK_UN     0x04    // Unlock
#define MY_LOCK_NB     0x08    // f: Non-blocking

// fcntl() commands (h-j)
#define MY_F_GETLK     0x01    // j: Get lock info
#define MY_F_SETLK     0x02    // h: Set non-blocking
#define MY_F_SETLKW    0x03    // i: Set blocking

// Lock types for fcntl
#define MY_F_RDLCK     0x00    // Read lock (shared)
#define MY_F_WRLCK     0x01    // Write lock (exclusive)
#define MY_F_UNLCK     0x02    // Unlock
```

---

## Exemple

```c
int main(void) {
    lock_manager_t *mgr = lock_manager_create();

    // 2.3.26.c-e: flock() - whole file locking
    int fd = open("data.txt", O_RDWR);

    // d: Shared lock for reading
    lock_flock(mgr, fd, MY_LOCK_SH);
    // Multiple readers can hold LOCK_SH simultaneously

    // e: Exclusive lock for writing
    lock_flock(mgr, fd, MY_LOCK_EX);
    // Only one writer, no readers

    // f: Non-blocking attempt
    if (lock_flock(mgr, fd, MY_LOCK_EX | MY_LOCK_NB) == -1) {
        if (errno == EWOULDBLOCK) {
            printf("Lock unavailable, would block\n");
        }
    }

    // 2.3.26.g-k: fcntl() - region locking
    lock_region_t region = {
        .l_type = MY_F_WRLCK,   // Write lock
        .l_whence = SEEK_SET,
        .l_start = 100,         // Start at byte 100
        .l_len = 50             // Lock 50 bytes
    };

    // h: Non-blocking set
    if (lock_fcntl(mgr, fd, MY_F_SETLK, &region) == -1) {
        // i: Blocking set (waits for lock)
        lock_fcntl(mgr, fd, MY_F_SETLKW, &region);
    }

    // j: Query existing lock
    lock_region_t query = { .l_type = MY_F_WRLCK, .l_start = 100, .l_len = 50 };
    lock_fcntl(mgr, fd, MY_F_GETLK, &query);
    if (query.l_type != MY_F_UNLCK) {
        printf("Locked by PID %d\n", query.l_pid);
    }

    // 2.3.26.l: Deadlock detection
    lock_enable_deadlock_detection(mgr, true);
    if (lock_detect_deadlock(mgr)) {
        pid_t cycle[10];
        size_t len;
        lock_get_deadlock_cycle(mgr, cycle, &len);
        printf("Deadlock: ");
        for (size_t i = 0; i < len; i++) {
            printf("%d -> ", cycle[i]);
        }
        printf("\n");
    }

    // 2.3.26.m: Fork behavior
    lock_set_inherit_mode(mgr, INHERIT_NONE);  // Default: locks not inherited
    pid_t pid = fork();
    if (pid == 0) {
        lock_on_fork(mgr, getpid());  // Child clears locks
    }

    // 2.3.26.a vs b: Advisory vs mandatory
    lock_set_mode(mgr, fd, LOCK_ADVISORY);  // a: Cooperative
    // LOCK_MANDATORY requires sgid bit + no group exec

    lock_flock(mgr, fd, MY_LOCK_UN);  // Unlock
    close(fd);

    lock_stats_t stats;
    lock_get_stats(mgr, &stats);
    printf("Locks: shared=%lu, exclusive=%lu, deadlocks=%lu\n",
           stats.shared_locks, stats.exclusive_locks, stats.deadlocks_detected);

    lock_manager_destroy(mgr);
    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_advisory_locks()      // 2.3.26.a
#[test] fn test_mandatory_locks()     // 2.3.26.b
#[test] fn test_flock_basic()         // 2.3.26.c
#[test] fn test_lock_sh()             // 2.3.26.d
#[test] fn test_lock_ex()             // 2.3.26.e
#[test] fn test_lock_nb()             // 2.3.26.f
#[test] fn test_fcntl_basic()         // 2.3.26.g
#[test] fn test_f_setlk()             // 2.3.26.h
#[test] fn test_f_setlkw()            // 2.3.26.i
#[test] fn test_f_getlk()             // 2.3.26.j
#[test] fn test_struct_flock()        // 2.3.26.k
#[test] fn test_deadlock_detection()  // 2.3.26.l
#[test] fn test_lock_inheritance()    // 2.3.26.m
#[test] fn test_concurrent_locks()
#[test] fn test_region_overlap()
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Advisory vs Mandatory (2.3.26.a-b) | 10 |
| flock() API (2.3.26.c-f) | 25 |
| fcntl() API (2.3.26.g-k) | 30 |
| Deadlock detection (2.3.26.l) | 20 |
| Lock inheritance (2.3.26.m) | 15 |
| **Total** | **100** |

---

## Fichiers

```
ex13/
├── file_lock.h
├── file_lock.c
├── flock_impl.c
├── fcntl_impl.c
├── deadlock.c
└── Makefile
```
