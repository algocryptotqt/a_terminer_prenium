# Module 2.3: File Systems - Matrice de Couverture DETAILLEE

## IMPORTANT: Chaque lettre (a, b, c...) = 1 concept a couvrir

## Statistiques Module 2.3
- **Sous-modules**: 27 (2.3.1 - 2.3.27) + PROJET
- **Concepts totaux (lettres)**: ~293
- **Exercices planifies**: 12

---

## Plan de Couverture par Exercice

### ex00: FS Inspector (Facile, 3h)
**Sous-modules couverts**: 2.3.1, 2.3.2
**Concepts lettres**:
- 2.3.1.a [ ] File: Named collection of data
- 2.3.1.b [ ] Directory: Container for files
- 2.3.1.c [ ] Path: Location of file
- 2.3.1.d [ ] Absolute path: From root
- 2.3.1.e [ ] Relative path: From current
- 2.3.1.f [ ] File types: Regular, directory, device, symlink, socket, pipe
- 2.3.1.g [ ] File attributes: Name, type, size, permissions, timestamps
- 2.3.1.h [ ] File operations: Create, open, read, write, close, delete
- 2.3.2.a [ ] Inode: Index node, file metadata
- 2.3.2.b [ ] Inode number: Unique within filesystem
- 2.3.2.c [ ] Inode contents: NOT name, NOT data
- 2.3.2.d [ ] File type: In inode
- 2.3.2.e [ ] Permissions: Mode bits
- 2.3.2.f [ ] Owner: UID, GID
- 2.3.2.g [ ] Size: In bytes
- 2.3.2.h [ ] Timestamps: atime, mtime, ctime
- 2.3.2.i [ ] Link count: Number of hard links
- 2.3.2.j [ ] Block pointers: To data blocks
- 2.3.2.k [ ] stat(): Get inode info
- 2.3.2.l [ ] ls -i: Show inode numbers
**Total**: 20 concepts

---

### ex01: Directory Walker (Facile, 3h)
**Sous-modules couverts**: 2.3.3
**Concepts lettres**:
- 2.3.3.a [ ] Directory file: Contains entries
- 2.3.3.b [ ] Directory entry: Name -> inode mapping
- 2.3.3.c [ ] . entry: Current directory
- 2.3.3.d [ ] .. entry: Parent directory
- 2.3.3.e [ ] Entry format: Inode, name length, type, name
- 2.3.3.f [ ] Linear list: Simple, slow lookup
- 2.3.3.g [ ] Hash table: Fast lookup
- 2.3.3.h [ ] B-tree: Sorted, good for large
- 2.3.3.i [ ] opendir(): Open directory
- 2.3.3.j [ ] readdir(): Read entry
- 2.3.3.k [ ] closedir(): Close directory
**Total**: 11 concepts

---

### ex02: Link Manager (Facile, 3h)
**Sous-modules couverts**: 2.3.4
**Concepts lettres**:
- 2.3.4.a [ ] Hard link: Another name for same inode
- 2.3.4.b [ ] link(): Create hard link
- 2.3.4.c [ ] Link count: Incremented
- 2.3.4.d [ ] Deletion: Decrements link count
- 2.3.4.e [ ] Hard link restrictions: Same filesystem, no directories
- 2.3.4.f [ ] Symbolic link: File containing path
- 2.3.4.g [ ] symlink(): Create symlink
- 2.3.4.h [ ] Symlink traversal: Follow on access
- 2.3.4.i [ ] lstat(): Don't follow symlink
- 2.3.4.j [ ] readlink(): Read symlink target
- 2.3.4.k [ ] Dangling symlink: Target doesn't exist
- 2.3.4.l [ ] Symlink loops: Detection limit
**Total**: 12 concepts

---

### ex03: File I/O Library (Moyen, 5h)
**Sous-modules couverts**: 2.3.5, 2.3.6
**Concepts lettres**:
- 2.3.5.a [ ] File descriptor: Integer handle
- 2.3.5.b [ ] Per-process table: fd -> file table entry
- 2.3.5.c [ ] System file table: Open file entries
- 2.3.5.d [ ] Inode table: In-memory inodes
- 2.3.5.e [ ] Standard fds: 0=stdin, 1=stdout, 2=stderr
- 2.3.5.f [ ] File table entry: Offset, flags, ref count
- 2.3.5.g [ ] Sharing: fork() shares entries
- 2.3.5.h [ ] dup(): Duplicate fd
- 2.3.5.i [ ] dup2(): Duplicate to specific fd
- 2.3.5.j [ ] fcntl(): Manipulate fd
- 2.3.5.k [ ] FD_CLOEXEC: Close on exec
- 2.3.6.a [ ] open(): Open/create file
- 2.3.6.b [ ] Open flags: O_RDONLY, O_WRONLY, O_RDWR
- 2.3.6.c [ ] O_CREAT: Create if not exists
- 2.3.6.d [ ] O_TRUNC: Truncate to zero
- 2.3.6.e [ ] O_APPEND: Append mode
- 2.3.6.f [ ] O_EXCL: Fail if exists
- 2.3.6.g [ ] read(): Read bytes
- 2.3.6.h [ ] write(): Write bytes
- 2.3.6.i [ ] lseek(): Change offset
- 2.3.6.j [ ] close(): Close fd
- 2.3.6.k [ ] fsync(): Flush to disk
- 2.3.6.l [ ] ftruncate(): Set size
**Total**: 23 concepts

---

### ex04: Permission Manager (Moyen, 4h)
**Sous-modules couverts**: 2.3.7
**Concepts lettres**:
- 2.3.7.a [ ] Permission bits: rwxrwxrwx
- 2.3.7.b [ ] User/Group/Other: Three categories
- 2.3.7.c [ ] Read: View contents
- 2.3.7.d [ ] Write: Modify contents
- 2.3.7.e [ ] Execute: Run file / traverse directory
- 2.3.7.f [ ] Octal notation: 755, 644
- 2.3.7.g [ ] chmod(): Change permissions
- 2.3.7.h [ ] fchmod(): Change by fd
- 2.3.7.i [ ] chown(): Change owner
- 2.3.7.j [ ] umask: Default permission mask
- 2.3.7.k [ ] Setuid bit: Run as owner
- 2.3.7.l [ ] Setgid bit: Run as group
- 2.3.7.m [ ] Sticky bit: Restrict deletion
**Total**: 13 concepts

---

### ex05: Block Allocator Simulator (Difficile, 8h)
**Sous-modules couverts**: 2.3.8, 2.3.9, 2.3.10, 2.3.11
**Concepts lettres**:
- 2.3.8.a [ ] Block: Unit of storage
- 2.3.8.b [ ] Block size: 1KB, 4KB typical
- 2.3.8.c [ ] Contiguous allocation: Sequential blocks
- 2.3.8.d [ ] Contiguous problems: Fragmentation, size
- 2.3.8.e [ ] Linked allocation: Chain of blocks
- 2.3.8.f [ ] Linked problems: Sequential access only
- 2.3.8.g [ ] FAT: File Allocation Table
- 2.3.8.h [ ] Indexed allocation: Index block with pointers
- 2.3.8.i [ ] Multi-level indexed: Indirect blocks
- 2.3.8.j [ ] Direct blocks: In inode
- 2.3.8.k [ ] Indirect: One level
- 2.3.8.l [ ] Double indirect: Two levels
- 2.3.8.m [ ] Triple indirect: Three levels
- 2.3.9.a [ ] Bitmap: One bit per block
- 2.3.9.b [ ] Bitmap location: Fixed location
- 2.3.9.c [ ] Bitmap operations: Find free, allocate, free
- 2.3.9.d [ ] Contiguous search: For extent allocation
- 2.3.9.e [ ] Free list: Linked list of free blocks
- 2.3.9.f [ ] Grouping: Block of pointers
- 2.3.9.g [ ] Counting: (start, length) pairs
- 2.3.10.a [ ] Boot block: Bootloader
- 2.3.10.b [ ] Superblock: FS metadata
- 2.3.10.c [ ] Inode table: All inodes
- 2.3.10.d [ ] Data blocks: File contents
- 2.3.10.e [ ] Block groups: Locality (ext2/3/4)
- 2.3.10.f [ ] Backup superblocks: Redundancy
- 2.3.10.g [ ] Reserved blocks: For root
- 2.3.11.a [ ] Magic number: Identify FS type
- 2.3.11.b [ ] Block size: Bytes per block
- 2.3.11.c [ ] Block count: Total blocks
- 2.3.11.d [ ] Inode count: Total inodes
- 2.3.11.e [ ] Free blocks: Available blocks
- 2.3.11.f [ ] Free inodes: Available inodes
- 2.3.11.g [ ] First data block: After metadata
- 2.3.11.h [ ] Mount count: For fsck
- 2.3.11.i [ ] State: Clean/dirty
- 2.3.11.j [ ] Error behavior: What to do on error
**Total**: 37 concepts

---

### ex06: Journal Simulator (Difficile, 7h)
**Sous-modules couverts**: 2.3.12
**Concepts lettres**:
- 2.3.12.a [ ] Crash consistency: Problem
- 2.3.12.b [ ] fsck: Check and repair (slow)
- 2.3.12.c [ ] Journal: Write-ahead log
- 2.3.12.d [ ] Transaction: Atomic group of operations
- 2.3.12.e [ ] Journal write: Log before data
- 2.3.12.f [ ] Commit: Mark transaction complete
- 2.3.12.g [ ] Checkpoint: Write data to final location
- 2.3.12.h [ ] Recovery: Replay or discard
- 2.3.12.i [ ] Journal modes: Data, ordered, writeback
- 2.3.12.j [ ] Data mode: Log data too
- 2.3.12.k [ ] Ordered mode: Metadata after data
- 2.3.12.l [ ] Writeback mode: Metadata only
**Total**: 12 concepts

---

### ex07: COW Filesystem Simulator (Difficile, 8h)
**Sous-modules couverts**: 2.3.13, 2.3.14
**Concepts lettres**:
- 2.3.13.a [ ] COW concept: Never overwrite
- 2.3.13.b [ ] Write -> new location: Always
- 2.3.13.c [ ] Update pointer: After write
- 2.3.13.d [ ] Atomic update: Pointer swap
- 2.3.13.e [ ] Consistency: Always consistent
- 2.3.13.f [ ] Snapshots: Free with COW
- 2.3.13.g [ ] Clones: Writable snapshots
- 2.3.13.h [ ] Fragmentation: Potential issue
- 2.3.13.i [ ] Write amplification: More writes
- 2.3.13.j [ ] Btrfs: COW filesystem
- 2.3.13.k [ ] ZFS: COW filesystem
- 2.3.14.a [ ] Extents: Contiguous block ranges
- 2.3.14.b [ ] Extent tree: B-tree of extents
- 2.3.14.c [ ] Delayed allocation: Allocate at write-back
- 2.3.14.d [ ] Online defrag: While mounted
- 2.3.14.e [ ] Checksums: Data integrity
- 2.3.14.f [ ] Compression: Transparent
- 2.3.14.g [ ] Deduplication: Share identical blocks
- 2.3.14.h [ ] Snapshots: Point-in-time copy
- 2.3.14.i [ ] Subvolumes: FS within FS
- 2.3.14.j [ ] RAID integration: Built-in redundancy
**Total**: 21 concepts

---

### ex08: FS Analyzer (Moyen, 5h)
**Sous-modules couverts**: 2.3.15, 2.3.16, 2.3.17
**Concepts lettres**:
- 2.3.15.a [ ] History: ext2 -> ext3 -> ext4
- 2.3.15.b [ ] ext2: Basic Linux FS
- 2.3.15.c [ ] ext3: Added journaling
- 2.3.15.d [ ] ext4: Extents, large files
- 2.3.15.e [ ] Block groups: Locality
- 2.3.15.f [ ] Flex groups: Aggregate groups
- 2.3.15.g [ ] Extents: Replace block pointers
- 2.3.15.h [ ] Multiblock allocation: Performance
- 2.3.15.i [ ] Delayed allocation: Performance
- 2.3.15.j [ ] Persistent preallocation: Reserved space
- 2.3.15.k [ ] Journal checksum: Reliability
- 2.3.16.a [ ] COW filesystem: By design
- 2.3.16.b [ ] B-tree: Primary structure
- 2.3.16.c [ ] Subvolumes: Independent FS trees
- 2.3.16.d [ ] Snapshots: Read-only
- 2.3.16.e [ ] Clones: Writable
- 2.3.16.f [ ] Checksums: Metadata and data
- 2.3.16.g [ ] Compression: zlib, lzo, zstd
- 2.3.16.h [ ] Deduplication: Offline
- 2.3.16.i [ ] RAID: 0, 1, 5, 6, 10
- 2.3.16.j [ ] Scrub: Check integrity
- 2.3.16.k [ ] Balance: Redistribute data
- 2.3.16.l [ ] Send/receive: Incremental backup
- 2.3.17.a [ ] Pooled storage: zpools
- 2.3.17.b [ ] Vdevs: Virtual devices
- 2.3.17.c [ ] Datasets: Filesystems, volumes, snapshots
- 2.3.17.d [ ] Copy-on-write: Always
- 2.3.17.e [ ] Checksums: Everything
- 2.3.17.f [ ] Self-healing: With redundancy
- 2.3.17.g [ ] ARC: Adaptive Replacement Cache
- 2.3.17.h [ ] L2ARC: Second-level cache
- 2.3.17.i [ ] ZIL: ZFS Intent Log
- 2.3.17.j [ ] SLOG: Separate log device
- 2.3.17.k [ ] Deduplication: Online
- 2.3.17.l [ ] Compression: Many algorithms
**Total**: 35 concepts

---

### ex09: Mini VFS (Tres difficile, 12h)
**Sous-modules couverts**: 2.3.18, 2.3.19
**Concepts lettres**:
- 2.3.18.a [ ] VFS purpose: Abstract FS interface
- 2.3.18.b [ ] VFS objects: Superblock, inode, dentry, file
- 2.3.18.c [ ] Operations structs: Function pointers
- 2.3.18.d [ ] super_operations: FS-level operations
- 2.3.18.e [ ] inode_operations: Inode operations
- 2.3.18.f [ ] file_operations: File operations
- 2.3.18.g [ ] dentry cache: Name -> inode cache
- 2.3.18.h [ ] Inode cache: In-memory inodes
- 2.3.18.i [ ] Path lookup: namei()
- 2.3.18.j [ ] Mount points: Crossing filesystems
- 2.3.19.a [ ] Mounting: Attach FS to tree
- 2.3.19.b [ ] Mount point: Directory to attach
- 2.3.19.c [ ] mount() syscall: Perform mount
- 2.3.19.d [ ] Mount flags: ro, noexec, nosuid
- 2.3.19.e [ ] Mount table: /proc/mounts
- 2.3.19.f [ ] /etc/fstab: Boot mount config
- 2.3.19.g [ ] umount(): Detach FS
- 2.3.19.h [ ] Busy filesystem: Can't unmount
- 2.3.19.i [ ] Lazy unmount: Detach, cleanup later
- 2.3.19.j [ ] Bind mount: Mount directory elsewhere
- 2.3.19.k [ ] Mount namespaces: Per-process mount view
**Total**: 21 concepts

---

### ex10: FUSE Filesystem (Tres difficile, 10h)
**Sous-modules couverts**: 2.3.20, 2.3.21
**Concepts lettres**:
- 2.3.20.a [ ] FUSE concept: Filesystem in Userspace
- 2.3.20.b [ ] Kernel module: /dev/fuse
- 2.3.20.c [ ] libfuse: User library
- 2.3.20.d [ ] Request handling: Kernel -> user -> kernel
- 2.3.20.e [ ] fuse_operations: Callbacks
- 2.3.20.f [ ] Low-level API: More control
- 2.3.20.g [ ] High-level API: Easier
- 2.3.20.h [ ] Performance: Context switch overhead
- 2.3.20.i [ ] Use cases: Network FS, archive FS, encrypted FS
- 2.3.21.a [ ] getattr: stat() equivalent
- 2.3.21.b [ ] readdir: List directory
- 2.3.21.c [ ] open: Open file
- 2.3.21.d [ ] read: Read data
- 2.3.21.e [ ] write: Write data
- 2.3.21.f [ ] create: Create file
- 2.3.21.g [ ] unlink: Delete file
- 2.3.21.h [ ] mkdir/rmdir: Directories
- 2.3.21.i [ ] rename: Move/rename
- 2.3.21.j [ ] truncate: Change size
- 2.3.21.k [ ] chmod/chown: Permissions
- 2.3.21.l [ ] symlink/readlink: Symbolic links
**Total**: 21 concepts

---

### ex11: Memory-Mapped I/O (Moyen, 5h)
**Sous-modules couverts**: 2.3.22
**Concepts lettres**:
- 2.3.22.a [ ] mmap(): Map file to memory
- 2.3.22.b [ ] Advantages: No copy, lazy loading
- 2.3.22.c [ ] PROT flags: PROT_READ, PROT_WRITE, PROT_EXEC
- 2.3.22.d [ ] MAP_SHARED: Modifications shared
- 2.3.22.e [ ] MAP_PRIVATE: Copy-on-write
- 2.3.22.f [ ] MAP_ANONYMOUS: No file backing
- 2.3.22.g [ ] munmap(): Unmap region
- 2.3.22.h [ ] msync(): Sync to file
- 2.3.22.i [ ] mprotect(): Change protection
- 2.3.22.j [ ] madvise(): Hint to kernel
- 2.3.22.k [ ] Page faults: Load on access
**Total**: 11 concepts

---

### ex12: Advanced I/O Operations (Difficile, 7h)
**Sous-modules couverts**: 2.3.23, 2.3.24, 2.3.25
**Concepts lettres**:
- 2.3.23.a [ ] Vectored I/O: readv/writev
- 2.3.23.b [ ] Scatter/gather: Multiple buffers
- 2.3.23.c [ ] pread/pwrite: Positional I/O
- 2.3.23.d [ ] sendfile(): Zero-copy transfer
- 2.3.23.e [ ] splice(): Pipe-based transfer
- 2.3.23.f [ ] tee(): Duplicate pipe data
- 2.3.23.g [ ] copy_file_range(): In-kernel copy
- 2.3.24.a [ ] POSIX AIO: aio_read, aio_write
- 2.3.24.b [ ] aiocb: Control block
- 2.3.24.c [ ] Completion: Signal or polling
- 2.3.24.d [ ] Linux AIO: io_submit, io_getevents
- 2.3.24.e [ ] libaio: Wrapper library
- 2.3.24.f [ ] Limitations: Often synchronous anyway
- 2.3.24.g [ ] io_uring: Modern Linux AIO
- 2.3.24.h [ ] Submission queue: Requests
- 2.3.24.i [ ] Completion queue: Results
- 2.3.24.j [ ] Zero-copy: No syscall per I/O
- 2.3.24.k [ ] Batching: Multiple operations
- 2.3.24.l [ ] liburing: Wrapper library
- 2.3.25.a [ ] O_DIRECT: Bypass page cache
- 2.3.25.b [ ] Requirements: Aligned buffer and offset
- 2.3.25.c [ ] Use case: Application-level caching
- 2.3.25.d [ ] Database usage: Custom buffer pool
- 2.3.25.e [ ] Performance: Depends on workload
- 2.3.25.f [ ] O_SYNC: Synchronous writes
- 2.3.25.g [ ] O_DSYNC: Data sync only
**Total**: 26 concepts

---

### ex13: File Locking & Concurrency (Moyen, 5h)
**Sous-modules couverts**: 2.3.26
**Concepts lettres**:
- 2.3.26.a [ ] Advisory locks: Cooperative
- 2.3.26.b [ ] Mandatory locks: Enforced (rare)
- 2.3.26.c [ ] flock(): Whole-file locking
- 2.3.26.d [ ] LOCK_SH: Shared lock
- 2.3.26.e [ ] LOCK_EX: Exclusive lock
- 2.3.26.f [ ] LOCK_NB: Non-blocking
- 2.3.26.g [ ] fcntl(): Byte-range locking
- 2.3.26.h [ ] F_SETLK: Set lock (non-blocking)
- 2.3.26.i [ ] F_SETLKW: Set lock (blocking)
- 2.3.26.j [ ] F_GETLK: Test lock
- 2.3.26.k [ ] struct flock: Lock specification
- 2.3.26.l [ ] Deadlock: Possible with fcntl
- 2.3.26.m [ ] Lock inheritance: Across fork
**Total**: 13 concepts

---

### ex14: RAID Simulator (Difficile, 7h)
**Sous-modules couverts**: 2.3.27
**Concepts lettres**:
- 2.3.27.a [ ] RAID concept: Redundant Array
- 2.3.27.b [ ] RAID 0: Striping only
- 2.3.27.c [ ] RAID 1: Mirroring
- 2.3.27.d [ ] RAID 5: Distributed parity
- 2.3.27.e [ ] RAID 6: Double parity
- 2.3.27.f [ ] RAID 10: Mirror + stripe
- 2.3.27.g [ ] Hot spare: Automatic rebuild
- 2.3.27.h [ ] Rebuild time: Hours to days
- 2.3.27.i [ ] URE: Unrecoverable Read Error
- 2.3.27.j [ ] mdadm: Linux software RAID
**Total**: 10 concepts

---

## PROJET FINAL: Simple File System (Bonus, 15h)
**Concepts lettres**: PROJET 2.3 (a-q) = 17 concepts

---

## RESUME COUVERTURE

| Exercice | Concepts | Heures | Difficulte |
|----------|----------|--------|------------|
| ex00 | 20 | 3h | Facile |
| ex01 | 11 | 3h | Facile |
| ex02 | 12 | 3h | Facile |
| ex03 | 23 | 5h | Moyen |
| ex04 | 13 | 4h | Moyen |
| ex05 | 37 | 8h | Difficile |
| ex06 | 12 | 7h | Difficile |
| ex07 | 21 | 8h | Difficile |
| ex08 | 35 | 5h | Moyen |
| ex09 | 21 | 12h | Tres difficile |
| ex10 | 21 | 10h | Tres difficile |
| ex11 | 11 | 5h | Moyen |
| ex12 | 26 | 7h | Difficile |
| ex13 | 13 | 5h | Moyen |
| ex14 | 10 | 7h | Difficile |
| PROJET | 17 | 15h | Bonus |
| **TOTAL** | **303** | **~107h** | - |

**Couverture**: 303/~293 = 100%+ (redondance pour renforcement)
