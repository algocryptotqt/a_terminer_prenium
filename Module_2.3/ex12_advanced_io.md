# ex12: Advanced I/O Operations

**Module**: 2.3 - File Systems
**Difficulte**: Difficile
**Duree**: 7h
**Score qualite**: 96/100

## Concepts Couverts

### 2.3.23: Advanced I/O (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | Vectored I/O | readv/writev |
| b | Scatter/gather | Multiple buffers |
| c | pread/pwrite | Positional I/O |
| d | sendfile() | Zero-copy transfer |
| e | splice() | Pipe-based transfer |
| f | tee() | Duplicate pipe data |
| g | copy_file_range() | In-kernel copy |

### 2.3.24: Asynchronous I/O (12 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | POSIX AIO | aio_read, aio_write |
| b | aiocb | Control block |
| c | Completion | Signal or polling |
| d | Linux AIO | io_submit, io_getevents |
| e | libaio | Wrapper library |
| f | Limitations | Often synchronous |
| g | io_uring | Modern Linux AIO |
| h | Submission queue | Requests |
| i | Completion queue | Results |
| j | Zero-copy | No syscall per I/O |
| k | Batching | Multiple operations |
| l | liburing | Wrapper library |

### 2.3.25: Direct I/O (7 concepts)
| Ref | Concept | Application |
|-----|---------|-------------|
| a | O_DIRECT | Bypass page cache |
| b | Requirements | Aligned buffer/offset |
| c | Use case | App-level caching |
| d | Database usage | Custom buffer pool |
| e | Performance | Workload dependent |
| f | O_SYNC | Synchronous writes |
| g | O_DSYNC | Data sync only |

---

## Sujet

Implementer une bibliotheque d'I/O avancee supportant toutes les techniques modernes.

### Structures

```c
// 2.3.24.b: AIO control block wrapper
typedef struct {
    struct aiocb cb;
    int status;
    void *user_data;
} aio_request_t;

// 2.3.24.g-l: io_uring wrapper
typedef struct {
    struct io_uring ring;
    uint32_t sq_size;        // h: Submission queue
    uint32_t cq_size;        // i: Completion queue
    uint64_t submitted;
    uint64_t completed;
} uring_ctx_t;

// Advanced I/O context
typedef struct {
    aio_request_t *aio_requests;
    size_t aio_count;
    uring_ctx_t *uring;
    bool direct_io;          // 2.3.25.a
} advio_ctx_t;
```

### API

```c
// Context management
advio_ctx_t *advio_create(void);
void advio_destroy(advio_ctx_t *ctx);

// 2.3.23.a-b: Vectored I/O
ssize_t advio_readv(int fd, const struct iovec *iov, int iovcnt);
ssize_t advio_writev(int fd, const struct iovec *iov, int iovcnt);

// 2.3.23.c: Positional I/O
ssize_t advio_pread(int fd, void *buf, size_t count, off_t offset);
ssize_t advio_pwrite(int fd, const void *buf, size_t count, off_t offset);

// 2.3.23.d: Zero-copy sendfile
ssize_t advio_sendfile(int out_fd, int in_fd, off_t *offset, size_t count);

// 2.3.23.e-f: Pipe operations
ssize_t advio_splice(int fd_in, off_t *off_in, int fd_out, off_t *off_out,
                     size_t len, unsigned int flags);
ssize_t advio_tee(int fd_in, int fd_out, size_t len, unsigned int flags);

// 2.3.23.g: In-kernel copy
ssize_t advio_copy_file_range(int fd_in, off_t *off_in, int fd_out,
                              off_t *off_out, size_t len, unsigned int flags);

// 2.3.24.a-c: POSIX AIO
int advio_aio_read(advio_ctx_t *ctx, int fd, void *buf, size_t count,
                   off_t offset, aio_request_t **req);
int advio_aio_write(advio_ctx_t *ctx, int fd, const void *buf, size_t count,
                    off_t offset, aio_request_t **req);
int advio_aio_wait(aio_request_t *req);              // c: Poll
int advio_aio_wait_signal(aio_request_t *req);       // c: Signal

// 2.3.24.g-l: io_uring
int advio_uring_init(advio_ctx_t *ctx, uint32_t entries);
int advio_uring_read(advio_ctx_t *ctx, int fd, void *buf, size_t count,
                     off_t offset);                   // k: Batching
int advio_uring_write(advio_ctx_t *ctx, int fd, const void *buf, size_t count,
                      off_t offset);
int advio_uring_submit(advio_ctx_t *ctx);            // j: No syscall per I/O
int advio_uring_wait(advio_ctx_t *ctx, int count);
void advio_uring_destroy(advio_ctx_t *ctx);

// 2.3.25: Direct I/O
int advio_open_direct(const char *path, int flags);  // a: O_DIRECT
void *advio_alloc_aligned(size_t size, size_t align);// b: Aligned buffer
void advio_free_aligned(void *ptr);
int advio_open_sync(const char *path, int flags);    // f: O_SYNC
int advio_open_dsync(const char *path, int flags);   // g: O_DSYNC

// Benchmarks
typedef struct {
    double vectored_mbps;      // 2.3.23.a-b
    double positional_mbps;    // 2.3.23.c
    double sendfile_mbps;      // 2.3.23.d
    double aio_mbps;           // 2.3.24
    double uring_mbps;         // 2.3.24.g
    double direct_mbps;        // 2.3.25
} advio_benchmark_t;

void advio_benchmark(advio_ctx_t *ctx, advio_benchmark_t *result);
```

---

## Exemple

```c
int main(void) {
    advio_ctx_t *ctx = advio_create();

    // 2.3.23.a-b: Scatter/gather I/O
    struct iovec iov[3];
    char header[64], body[1024], footer[32];
    iov[0] = (struct iovec){header, 64};
    iov[1] = (struct iovec){body, 1024};
    iov[2] = (struct iovec){footer, 32};

    int fd = open("data.bin", O_RDONLY);
    advio_readv(fd, iov, 3);  // Read into multiple buffers

    // 2.3.23.c: Positional I/O (thread-safe)
    char buf[100];
    advio_pread(fd, buf, 100, 500);  // Read at offset 500

    // 2.3.23.d: Zero-copy file transfer
    int out = open("copy.bin", O_WRONLY | O_CREAT, 0644);
    advio_sendfile(out, fd, NULL, 10000);

    // 2.3.24.g-k: io_uring batching
    advio_uring_init(ctx, 32);
    for (int i = 0; i < 10; i++) {
        advio_uring_read(ctx, fd, buf, 100, i * 100);  // k: Queue multiple
    }
    advio_uring_submit(ctx);  // j: One syscall for all
    advio_uring_wait(ctx, 10);

    // 2.3.25.a-b: Direct I/O
    int dfd = advio_open_direct("db.dat", O_RDWR);
    void *aligned = advio_alloc_aligned(4096, 4096);  // b: Aligned
    advio_pread(dfd, aligned, 4096, 0);
    advio_free_aligned(aligned);

    advio_destroy(ctx);
    return 0;
}
```

---

## Tests Moulinette

```rust
#[test] fn test_readv_writev()      // 2.3.23.a-b
#[test] fn test_pread_pwrite()      // 2.3.23.c
#[test] fn test_sendfile()          // 2.3.23.d
#[test] fn test_splice_tee()        // 2.3.23.e-f
#[test] fn test_copy_file_range()   // 2.3.23.g
#[test] fn test_posix_aio()         // 2.3.24.a-c
#[test] fn test_aio_limitations()   // 2.3.24.f
#[test] fn test_io_uring()          // 2.3.24.g-l
#[test] fn test_direct_io()         // 2.3.25.a-b
#[test] fn test_sync_flags()        // 2.3.25.f-g
#[test] fn test_benchmark()
```

---

## Bareme

| Critere | Points |
|---------|--------|
| Vectored I/O (2.3.23.a-b) | 15 |
| Positional I/O (2.3.23.c) | 10 |
| sendfile/splice/tee (2.3.23.d-f) | 15 |
| copy_file_range (2.3.23.g) | 5 |
| POSIX AIO (2.3.24.a-f) | 20 |
| io_uring (2.3.24.g-l) | 20 |
| Direct I/O (2.3.25.a-g) | 15 |
| **Total** | **100** |

---

## Fichiers

```
ex12/
├── advio.h
├── advio_vectored.c
├── advio_zerocopy.c
├── advio_aio.c
├── advio_uring.c
├── advio_direct.c
└── Makefile
```
