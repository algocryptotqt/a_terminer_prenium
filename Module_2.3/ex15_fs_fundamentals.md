# [Module 2.3] - Exercise 15: File System Fundamentals

## Metadonnees

```yaml
module: "2.3 - File Systems"
exercise: "ex15"
title: "File System Fundamentals - Complete Coverage"
difficulty: moyen
estimated_time: "8 heures"
prerequisite_exercises: ["ex00", "ex01", "ex02", "ex03"]
concepts_requis: ["file operations", "inodes", "directories"]
score_qualite: 97
```

---

## Concepts Couverts

### 2.3.1: File System Concepts (16 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.1.a | File: Named collection of data | Definition fichier |
| 2.3.1.b | Directory: Container for files | Definition repertoire |
| 2.3.1.c | Path: Location of file | Chemin d'acces |
| 2.3.1.d | Absolute path: From root | Chemin absolu |
| 2.3.1.e | Relative path: From current | Chemin relatif |
| 2.3.1.f | File types: Regular, directory, device, symlink, socket, pipe | Types de fichiers |
| 2.3.1.g | File attributes: Name, type, size, permissions, timestamps | Attributs |
| 2.3.1.h | File operations: Create, open, read, write, close, delete | Operations |
| 2.3.1.i | File system hierarchy | Hierarchie |
| 2.3.1.j | Root directory | Repertoire racine |
| 2.3.1.k | Current working directory | Repertoire courant |
| 2.3.1.l | Parent directory | Repertoire parent |
| 2.3.1.m | File naming conventions | Conventions de nommage |
| 2.3.1.n | Path resolution | Resolution de chemin |
| 2.3.1.o | Canonical path | Chemin canonique |
| 2.3.1.p | realpath(): Get canonical path | Fonction realpath |

### 2.3.2: Inodes (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.2.a | Inode: Index node, file metadata | Definition inode |
| 2.3.2.b | Inode number: Unique within filesystem | Numero d'inode |
| 2.3.2.c | Inode contents: NOT name, NOT data | Contenu inode |
| 2.3.2.d | File type: In inode | Type dans inode |
| 2.3.2.e | Permissions: Mode bits | Bits de permission |
| 2.3.2.f | Owner: UID, GID | Proprietaire |
| 2.3.2.g | Size: In bytes | Taille |
| 2.3.2.h | Timestamps: atime, mtime, ctime | Horodatages |
| 2.3.2.i | Link count: Number of hard links | Compteur de liens |
| 2.3.2.j | Block pointers: Direct and indirect | Pointeurs de blocs |
| 2.3.2.k | stat(): Get inode info | Fonction stat |
| 2.3.2.l | ls -i: Show inode numbers | Affichage inodes |

### 2.3.3: Directory Structure (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.3.a | Directory file: Contains entries | Fichier repertoire |
| 2.3.3.b | Directory entry: Name → inode mapping | Entree repertoire |
| 2.3.3.c | . entry: Current directory | Entree point |
| 2.3.3.d | .. entry: Parent directory | Entree double point |
| 2.3.3.e | Hash tables: Fast lookup | Tables de hachage |
| 2.3.3.f | B-trees: Sorted, efficient | Arbres B |
| 2.3.3.g | Linear list: Simple, slow | Liste lineaire |
| 2.3.3.h | opendir(): Open directory | Ouvrir repertoire |
| 2.3.3.i | readdir(): Read entry | Lire entree |
| 2.3.3.j | scandir(): Filtered read | Lecture filtree |
| 2.3.3.k | closedir(): Close directory | Fermer repertoire |
| 2.3.3.l | mkdir()/rmdir(): Create/remove dir | Creation/suppression |

### 2.3.4: Links (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.4.a | Hard link: Another name for same inode | Lien physique |
| 2.3.4.b | Link count: Incremented on link | Compteur de liens |
| 2.3.4.c | No cross-filesystem: Same FS only | Meme FS seulement |
| 2.3.4.d | No directory hard links: Prevent cycles | Pas de liens dir |
| 2.3.4.e | link(): Create hard link | Creer lien physique |
| 2.3.4.f | Symbolic link: Path to target | Lien symbolique |
| 2.3.4.g | Symlink contents: Target path string | Contenu symlink |
| 2.3.4.h | Cross-filesystem OK: Just path | Inter-FS OK |
| 2.3.4.i | Dangling symlink: Target deleted | Symlink orphelin |
| 2.3.4.j | symlink(): Create symlink | Creer symlink |
| 2.3.4.k | readlink(): Read symlink target | Lire cible symlink |
| 2.3.4.l | Symlink loops: Detection limit | Boucles symlink |

### 2.3.5: File Descriptors (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.5.a | File descriptor: Integer handle | Descripteur entier |
| 2.3.5.b | FD table: Per-process | Table par processus |
| 2.3.5.c | Open file table: System-wide | Table systeme |
| 2.3.5.d | Inode table: In memory | Table inodes |
| 2.3.5.e | 0, 1, 2: stdin, stdout, stderr | FD standards |
| 2.3.5.f | dup(): Duplicate descriptor | Dupliquer FD |
| 2.3.5.g | dup2(): Duplicate to specific FD | Dupliquer vers FD |
| 2.3.5.h | File offset: Current position | Position courante |
| 2.3.5.i | Shared offset: After fork | Offset partage |
| 2.3.5.j | O_CLOEXEC: Close on exec | Fermer sur exec |
| 2.3.5.k | FD_CLOEXEC: Close on exec flag | Flag CLOEXEC |
| 2.3.5.l | fcntl(): FD operations | Operations FD |

### 2.3.9: Free Space Management (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.9.a | Bitmap: One bit per block | Bitmap |
| 2.3.9.b | Bitmap size: blocks/8 bytes | Taille bitmap |
| 2.3.9.c | First-fit: Find first free | Premier libre |
| 2.3.9.d | Best-fit: Minimize waste | Meilleur ajustement |
| 2.3.9.e | Linked list: Chain free blocks | Liste chainee |
| 2.3.9.f | Grouping: Store in first block | Regroupement |
| 2.3.9.g | Counting: (start, length) pairs | Comptage |
| 2.3.9.h | Extent trees: B-tree of extents | Arbres d'extents |
| 2.3.9.i | Block groups: Localized allocation | Groupes de blocs |
| 2.3.9.j | Preallocation: Reserve ahead | Preallocation |
| 2.3.9.k | Delayed allocation | Allocation differee |
| 2.3.9.l | Space maps: ZFS approach | Cartes d'espace |

### 2.3.10: File System Layout (9 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.10.a | Boot block: Bootloader | Bloc de boot |
| 2.3.10.b | Superblock: FS metadata | Superbloc |
| 2.3.10.c | Inode table: All inodes | Table d'inodes |
| 2.3.10.d | Data blocks: File contents | Blocs de donnees |
| 2.3.10.e | Block groups: Locality | Groupes de blocs |
| 2.3.10.f | Group descriptor: Per-group metadata | Descripteur groupe |
| 2.3.10.g | Reserved blocks: For root | Blocs reserves |
| 2.3.10.h | Flex block groups | Groupes flex |
| 2.3.10.i | Meta block groups | Groupes meta |

### 2.3.11: Superblock (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.11.a | Magic number: Identify FS type | Nombre magique |
| 2.3.11.b | Block size: 1K, 2K, 4K | Taille de bloc |
| 2.3.11.c | Block count: Total blocks | Nombre de blocs |
| 2.3.11.d | Inode count: Total inodes | Nombre d'inodes |
| 2.3.11.e | Free block count | Blocs libres |
| 2.3.11.f | Free inode count | Inodes libres |
| 2.3.11.g | Mount count: Fsck trigger | Compteur de montage |
| 2.3.11.h | Last mount time | Dernier montage |
| 2.3.11.i | Features: Compat, incompat | Fonctionnalites |
| 2.3.11.j | Error behavior: What to do on error | Comportement erreur |
| 2.3.11.k | Backup superblocks | Superblocs backup |
| 2.3.11.l | UUID: Unique identifier | Identifiant unique |

### 2.3.13: Copy-on-Write (11 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.13.a | COW concept: Never overwrite | Concept COW |
| 2.3.13.b | Write → new location: Always | Nouvelle position |
| 2.3.13.c | Update pointer: After write | Mise a jour pointeur |
| 2.3.13.d | Atomic update: Pointer swap | Echange atomique |
| 2.3.13.e | Consistency: Always consistent | Coherence |
| 2.3.13.f | Snapshots: Free with COW | Snapshots |
| 2.3.13.g | Clones: Writable snapshots | Clones |
| 2.3.13.h | Fragmentation: Potential issue | Fragmentation |
| 2.3.13.i | Write amplification: More writes | Amplification |
| 2.3.13.j | Btrfs: COW filesystem | Btrfs |
| 2.3.13.k | ZFS: COW filesystem | ZFS |

### 2.3.14: Modern FS Features (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.14.a | Extents: Contiguous block ranges | Extents |
| 2.3.14.b | Checksums: Data integrity | Checksums |
| 2.3.14.c | Compression: Inline compression | Compression |
| 2.3.14.d | Deduplication: Remove duplicates | Deduplication |
| 2.3.14.e | Encryption: Per-file or per-FS | Chiffrement |
| 2.3.14.f | Quotas: Per-user/group limits | Quotas |
| 2.3.14.g | ACLs: Extended permissions | ACL |
| 2.3.14.h | xattrs: Extended attributes | Attributs etendus |
| 2.3.14.i | Subvolumes: Logical partitions | Sous-volumes |
| 2.3.14.j | RAID integration: Built-in redundancy | Integration RAID |

### 2.3.18: Virtual File System (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.18.a | VFS purpose: Abstract FS interface | But du VFS |
| 2.3.18.b | FS registration: Register with VFS | Enregistrement |
| 2.3.18.c | Common operations: Open, read, write | Operations communes |
| 2.3.18.d | Inode object: VFS inode | Objet inode |
| 2.3.18.e | File object: Open file | Objet fichier |
| 2.3.18.f | Dentry object: Directory cache | Objet dentry |
| 2.3.18.g | Superblock object: FS instance | Objet superbloc |
| 2.3.18.h | Dentry cache: dcache | Cache dentry |
| 2.3.18.i | Inode cache: icache | Cache inode |
| 2.3.18.j | Mount points: Crossing filesystems | Points de montage |
| 2.3.18.k | Path lookup: Name resolution | Resolution de chemin |
| 2.3.18.l | Namespace operations | Operations namespace |

### 2.3.20: FUSE (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.20.a | FUSE concept: Filesystem in Userspace | Concept FUSE |
| 2.3.20.b | Kernel module: fuse.ko | Module kernel |
| 2.3.20.c | User daemon: FS implementation | Daemon utilisateur |
| 2.3.20.d | /dev/fuse: Communication channel | Canal communication |
| 2.3.20.e | Request forwarding: Kernel to user | Transfert requetes |
| 2.3.20.f | libfuse: Helper library | Bibliotheque libfuse |
| 2.3.20.g | High-level API: Simple callbacks | API haut niveau |
| 2.3.20.h | Low-level API: More control | API bas niveau |
| 2.3.20.i | Use cases: Network FS, archive FS, encrypted FS | Cas d'usage |
| 2.3.20.j | Performance considerations | Considerations perf |
| 2.3.20.k | Security implications | Implications securite |
| 2.3.20.l | FUSE mounting and options | Montage FUSE |

### 2.3.23: Advanced I/O (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.23.a | Vectored I/O: readv/writev | I/O vectorise |
| 2.3.23.b | Scatter/gather: Multiple buffers | Scatter/gather |
| 2.3.23.c | pread/pwrite: Atomic seek + read/write | pread/pwrite |
| 2.3.23.d | Positional I/O: No side effects | I/O positionnel |
| 2.3.23.e | sendfile(): Zero-copy | Zero-copie |
| 2.3.23.f | splice(): Pipe-based transfer | Transfert par pipe |
| 2.3.23.g | copy_file_range(): In-kernel copy | Copie kernel |
| 2.3.23.h | tee(): Duplicate pipe data | Duplication pipe |
| 2.3.23.i | vmsplice(): User to pipe | User vers pipe |
| 2.3.23.j | Hole punching: Sparse files | Fichiers creux |
| 2.3.23.k | Fallocate: Preallocate space | Preallocation |
| 2.3.23.l | File sealing: Immutability | Scellement |

### 2.3.25: Direct I/O (9 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.25.a | O_DIRECT: Bypass page cache | Contourner cache |
| 2.3.25.b | Alignment requirements: Block-aligned | Alignement |
| 2.3.25.c | User buffer alignment | Alignement buffer |
| 2.3.25.d | Size alignment: Multiple of block | Taille alignee |
| 2.3.25.e | Use cases: Databases | Cas d'usage |
| 2.3.25.f | O_SYNC: Synchronous I/O | I/O synchrone |
| 2.3.25.g | O_DSYNC: Data sync only | Sync donnees |
| 2.3.25.h | Performance implications | Implications perf |
| 2.3.25.i | Combining O_DIRECT with O_SYNC | Combinaison |

### 2.3.27: RAID (17 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.3.27.a | RAID concept: Redundant Array | Concept RAID |
| 2.3.27.b | RAID 0: Striping, no redundancy | RAID 0 |
| 2.3.27.c | RAID 1: Mirroring | RAID 1 |
| 2.3.27.d | RAID 5: Distributed parity | RAID 5 |
| 2.3.27.e | RAID 6: Dual parity | RAID 6 |
| 2.3.27.f | RAID 10: Mirror + stripe | RAID 10 |
| 2.3.27.g | Hot spare: Automatic rebuild | Hot spare |
| 2.3.27.h | Stripe size: Block distribution | Taille stripe |
| 2.3.27.i | Chunk size: Unit of striping | Taille chunk |
| 2.3.27.j | mdadm: Linux software RAID | mdadm |
| 2.3.27.k | Hardware vs software RAID | Hardware vs software |
| 2.3.27.l | RAID write hole | Trou d'ecriture |
| 2.3.27.m | LC 588 Design In-Memory FS | LeetCode 588 |
| 2.3.27.n | LC 1166 Design File System | LeetCode 1166 |
| 2.3.27.o | Design FUSE filesystem: Custom | Projet FUSE |
| 2.3.27.p | Implement journaling: Custom | Projet journaling |
| 2.3.27.q | Advanced RAID concepts | Concepts RAID avances |

---

## Contexte

Ce module couvre les concepts fondamentaux des systemes de fichiers Unix/Linux. Vous apprendrez comment les fichiers et repertoires sont organises, comment les inodes stockent les metadonnees, et comment les liens physiques et symboliques fonctionnent.

---

## Enonce

Implementez une bibliotheque complete d'exploration et manipulation du systeme de fichiers qui demontre tous les concepts fondamentaux.

### API Principale

```c
// File system exploration
int fs_get_file_info(const char *path, struct stat *st);
char *fs_get_file_type(mode_t mode);
char *fs_realpath(const char *path);
int fs_is_same_file(const char *path1, const char *path2);

// Directory operations
char **fs_list_directory(const char *path, int *count);
int fs_walk_tree(const char *path, int (*callback)(const char *, const struct stat *));

// Link operations
int fs_create_hard_link(const char *target, const char *linkpath);
int fs_create_symlink(const char *target, const char *linkpath);
char *fs_read_symlink(const char *path);
int fs_is_dangling_symlink(const char *path);

// Inode operations
ino_t fs_get_inode(const char *path);
int fs_get_link_count(const char *path);
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| File operations | 25 |
| Directory operations | 25 |
| Link operations | 25 |
| Inode handling | 15 |
| Error handling | 10 |
| **Total** | **100** |
