# Exercice 2.3.1-27 : lordran_filesystem

**Module :**
2.3.1-27 — File System Fundamentals (Complete Coverage)

**Concept :**
synth — Méga-synthèse (150+ concepts: inodes, directories, links, VFS, FUSE, RAID)

**Difficulté :**
★★★★★★★★☆☆ (8/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (tous concepts 2.3.1 → 2.3.27)

**Langage :**
C (C17)

**Prérequis :**
- Pointeurs et allocation mémoire
- Structures de données (arbres, listes)
- Appels système POSIX basiques
- ex00-ex14 du Module 2.3

**Domaines :**
FS, Mem, Struct, Process

**Durée estimée :**
480 min

**XP Base :**
600

**Complexité :**
T4 O(n log n) × S3 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex15/
├── lordran_fs.h
├── soul_vessel.c        (inodes)
├── bonfire_network.c    (directories)
├── shortcut_gates.c     (links)
├── fog_gates.c          (permissions)
├── estus_pool.c         (file descriptors)
├── explore_lordran.c    (tree walking)
└── Makefile
```

**Fonctions autorisées :**
`stat`, `lstat`, `fstat`, `opendir`, `readdir`, `closedir`, `scandir`, `link`, `symlink`, `readlink`, `realpath`, `open`, `close`, `read`, `write`, `dup`, `dup2`, `fcntl`, `malloc`, `free`, `printf`, `perror`

**Fonctions interdites :**
`system`, `popen`

---

### 1.2 Consigne

**🎮 CONTEXTE : DARK SOULS — Praise the Sun, Explore the Filesystem**

*"You Died"* ... mais vous revenez toujours. Bienvenue à **Lordran**, un monde interconnecté où chaque zone est liée aux autres par des **shortcuts** secrets.

Dans ce monde :
- Chaque **créature** possède une **Soul Vessel** (inode) qui contient son essence (métadonnées)
- Les **Bonfires** sont des points de repère (répertoires) d'où vous pouvez explorer
- Les **shortcuts** que vous débloquez (hard links) connectent des zones éloignées
- Les **messages** au sol (symlinks) pointent vers la sagesse d'autres joueurs
- Les **Fog Gates** (permissions) bloquent l'accès aux zones dangereuses
- Votre **Estus Flask** (file descriptors) est une ressource limitée à gérer

**Le défi de l'Undead :** Quand vous mourrez (crash), vos données persistent grâce aux **Soul Vessels** (inodes). Le nom peut changer, mais l'âme reste.

**Ta mission :**

Implémenter une bibliothèque complète d'exploration du filesystem qui permet de :
1. Examiner les **Soul Vessels** (stat, inode info)
2. Explorer les **Bonfire Networks** (directory traversal)
3. Créer des **Shortcuts** (hard links) et **Messages** (symlinks)
4. Gérer les **Fog Gates** (permissions)
5. Comprendre le **Estus Pool** (file descriptors)

**Entrée :**
- `path` : Chemin vers une entité dans Lordran (fichier/répertoire)
- `callback` : Fonction appelée lors de l'exploration

**Sortie :**
- Structures contenant les métadonnées (Soul Vessel)
- Listes de chemins explorés
- Succès/échec des opérations

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `soul_examine("/firelink/bonfire.txt")` | `soul_vessel_t*` | Métadonnées du fichier |
| `true_path_reveal("../undead_burg/../firelink")` | `"/firelink"` | Chemin canonique |
| `same_soul_check("link1", "link2")` | `true` | Même inode |
| `unlock_shortcut("/boss", "/shortcut")` | `0` | Hard link créé |
| `leave_message("/tip", "git gud")` | `0` | Symlink créé |
| `explore_lordran("/", callback)` | `N` | N entités visitées |

---

### 1.2.2 Consigne Académique

Implémenter une bibliothèque complète d'exploration et manipulation du système de fichiers Unix/Linux couvrant : inodes et métadonnées (stat), navigation de répertoires (opendir/readdir), liens physiques et symboliques (link/symlink), résolution de chemins (realpath), et gestion des descripteurs de fichiers (dup/fcntl).

---

### 1.3 Prototype

```c
#ifndef LORDRAN_FS_H
#define LORDRAN_FS_H

#include <sys/types.h>
#include <sys/stat.h>
#include <dirent.h>
#include <stdbool.h>
#include <stdint.h>

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.2: SOUL VESSEL (INODE) — "The essence of every entity"
   ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    ino_t soul_id;              // 2.3.2.b: Inode number (unique soul)
    mode_t fog_gate;            // 2.3.2.e: Permissions (access control)
    uid_t lord_uid;             // 2.3.2.f: Owner UID
    gid_t covenant_gid;         // 2.3.2.f: Owner GID
    off_t soul_level;           // 2.3.2.g: Size in bytes
    nlink_t shortcut_count;     // 2.3.2.i: Link count

    // 2.3.2.h: Timestamps (when things happened)
    time_t last_kindled;        // atime: Last access
    time_t last_modified;       // mtime: Last modification
    time_t soul_created;        // ctime: Status change

    // 2.3.2.d: Entity type
    char entity_type;           // 'f'ile, 'd'ir, 'l'ink, 's'ocket, etc.
    char entity_name[256];
} soul_vessel_t;

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.3: BONFIRE NETWORK (DIRECTORIES) — "Rest points in Lordran"
   ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    char name[256];             // 2.3.3.b: Entry name
    ino_t soul_id;              // Points to soul vessel
    unsigned char entity_type;  // DT_REG, DT_DIR, DT_LNK, etc.
} bonfire_entry_t;

typedef struct {
    char path[4096];
    bonfire_entry_t *entries;
    size_t entry_count;
    bool has_dot;               // 2.3.3.c: . entry
    bool has_dotdot;            // 2.3.3.d: .. entry
} bonfire_t;

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.4: SHORTCUTS & MESSAGES (LINKS) — "Connecting the world"
   ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    char path[4096];
    char target[4096];          // 2.3.4.g: Symlink target
    bool is_shortcut;           // 2.3.4.a: Hard link
    bool is_message;            // 2.3.4.f: Symbolic link
    bool is_phantom;            // 2.3.4.i: Dangling symlink
} link_info_t;

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.5: ESTUS POOL (FILE DESCRIPTORS) — "Your limited resource"
   ═══════════════════════════════════════════════════════════════════════════ */

typedef struct {
    int flask_number;           // 2.3.5.a: FD number
    char path[4096];            // Associated file
    off_t cursor_position;      // 2.3.5.h: Current offset
    int flags;                  // Open flags
    bool is_bonfire;            // stdin/stdout/stderr (0,1,2)
} estus_flask_t;

typedef struct {
    estus_flask_t *flasks;
    size_t flask_count;
    size_t max_flasks;          // System limit
} estus_pool_t;

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.1: PATH OPERATIONS — "Finding your way through Lordran"
   ═══════════════════════════════════════════════════════════════════════════ */

// 2.3.1.n: Path resolution
char *true_path_reveal(const char *path);           // realpath wrapper

// 2.3.1.d vs 2.3.1.e: Absolute vs relative
bool is_absolute_path(const char *path);
char *make_absolute(const char *path);

// 2.3.1.o: Canonical path operations
char *canonicalize_path(const char *path);
bool paths_equivalent(const char *path1, const char *path2);

/* ═══════════════════════════════════════════════════════════════════════════
   API — EXPLORING LORDRAN
   ═══════════════════════════════════════════════════════════════════════════ */

// Soul Vessel operations (2.3.2)
soul_vessel_t *soul_examine(const char *path);
soul_vessel_t *soul_examine_no_follow(const char *path);  // lstat
void soul_release(soul_vessel_t *vessel);
char *get_entity_type_name(char type);
ino_t get_soul_id(const char *path);
nlink_t count_shortcuts(const char *path);
bool same_soul_check(const char *path1, const char *path2);

// Bonfire operations (2.3.3)
bonfire_t *bonfire_warp_to(const char *path);
void bonfire_leave(bonfire_t *bonfire);
char **bonfire_list_entries(const char *path, size_t *count);
void free_entry_list(char **entries, size_t count);

// Tree exploration (2.3.3 + 2.3.1)
typedef int (*undead_callback)(const char *path, const soul_vessel_t *vessel, void *data);
int explore_lordran(const char *start, undead_callback callback, void *user_data);
int explore_lordran_filtered(const char *start, undead_callback callback,
                            bool follow_messages, int max_depth, void *user_data);

// Shortcut operations (2.3.4.a-e: hard links)
int unlock_shortcut(const char *target, const char *shortcut_path);
bool is_shortcut(const char *path);
int count_all_shortcuts(const char *path);  // All hard links to same inode

// Message operations (2.3.4.f-l: symbolic links)
int leave_message(const char *target, const char *message_path);
char *read_message(const char *path);
bool is_message(const char *path);
bool is_phantom_message(const char *path);  // Dangling symlink
int resolve_message_chain(const char *path, char *final_target, int max_hops);

// Estus Pool operations (2.3.5)
estus_pool_t *estus_pool_init(void);
void estus_pool_destroy(estus_pool_t *pool);
int kindle_flask(estus_pool_t *pool, const char *path, int flags);  // open
void extinguish_flask(estus_pool_t *pool, int flask);               // close
int duplicate_flask(estus_pool_t *pool, int flask);                 // dup
int warp_flask(estus_pool_t *pool, int flask, int target);          // dup2
int get_flask_flags(int flask);                                      // fcntl

// Fog Gate operations (permissions - 2.3.2.e)
bool can_traverse(const char *path);          // Execute permission (dir)
bool can_read_soul(const char *path);         // Read permission
bool can_modify_soul(const char *path);       // Write permission
char *fog_gate_string(mode_t mode);           // e.g., "rwxr-xr-x"

// Statistics
typedef struct {
    size_t souls_examined;
    size_t bonfires_visited;
    size_t shortcuts_found;
    size_t messages_read;
    size_t phantoms_encountered;
    size_t fog_gates_blocked;
} lordran_stats_t;

void get_exploration_stats(lordran_stats_t *stats);
void reset_exploration_stats(void);

#endif /* LORDRAN_FS_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Pourquoi Dark Souls est l'Analogie Parfaite pour les Filesystems

Le monde de Dark Souls est célèbre pour son **interconnectivité** — des zones apparemment éloignées sont en fait connectées par des **shortcuts** secrets. C'est exactement comme un filesystem Unix :

```
                          FIRELINK SHRINE
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
      UNDEAD BURG         CATACOMBS          NEW LONDO
           │                   │                   │
           │    (shortcut)     │                   │
           └───────────────────┘                   │
                   ↑                               │
                   │         (shortcut)            │
                   └───────────────────────────────┘

    Les shortcuts (hard links) connectent des zones
    sans copier le contenu — même "âme" (inode)!
```

### 2.2 Soul Vessel = Inode

Dans Dark Souls, chaque entité possède une **Soul** — son essence qui persiste même après la mort. C'est exactement ce qu'est un **inode** :

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOUL VESSEL (INODE)                          │
├─────────────────────────────────────────────────────────────────┤
│  soul_id: 12345              ← Unique dans le filesystem        │
│  entity_type: 'f'            ← Regular file, directory, etc.    │
│  fog_gate: 0755              ← Permissions (rwxr-xr-x)          │
│  lord_uid: 1000              ← Propriétaire                     │
│  soul_level: 4096            ← Taille en bytes                  │
│  shortcut_count: 3           ← Nombre de hard links             │
│  last_kindled: 1704067200    ← Dernier accès                    │
├─────────────────────────────────────────────────────────────────┤
│  NOTE: Le NOM n'est PAS dans l'inode!                           │
│        Il est dans le directory entry (bonfire).                │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Messages = Symbolic Links

Les **messages** laissés par d'autres joueurs dans Dark Souls pointent vers des informations utiles. Parfois ils sont **phantom** (le joueur qui l'a laissé est mort/déconnecté) — comme un **dangling symlink** !

```
Message: "Try jumping" → Cliff (morte)
         ↑
    PHANTOM MESSAGE! La cible n'existe plus.
    C'est un DANGLING SYMLINK.
```

---

### 2.5 DANS LA VRAIE VIE

| Métier | Concept FS Utilisé | Application |
|--------|-------------------|-------------|
| **SysAdmin** | Inodes, hard links | Backup avec hardlinks (rsnapshot) |
| **DevOps** | Symlinks | `/etc/alternatives`, configurations |
| **Développeur** | File descriptors | gestion I/O, redirection |
| **DBA** | Direct I/O, VFS | Bypass cache pour bases de données |
| **Security** | Permissions, ACLs | Hardening serveurs |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
lordran_fs.h  soul_vessel.c  bonfire_network.c  shortcut_gates.c  fog_gates.c  estus_pool.c  explore_lordran.c  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -c soul_vessel.c -o soul_vessel.o
gcc -Wall -Wextra -Werror -c bonfire_network.c -o bonfire_network.o
gcc -Wall -Wextra -Werror -c shortcut_gates.c -o shortcut_gates.o
gcc -Wall -Wextra -Werror -c fog_gates.c -o fog_gates.o
gcc -Wall -Wextra -Werror -c estus_pool.c -o estus_pool.o
gcc -Wall -Wextra -Werror -c explore_lordran.c -o explore_lordran.o
gcc -Wall -Wextra -Werror *.o main.c -o lordran_test

$ ./lordran_test
╔══════════════════════════════════════════════════════════════╗
║     DARK SOULS: FILESYSTEM EDITION — Praise the Sun!         ║
╠══════════════════════════════════════════════════════════════╣
║  "In Lordran, the flow of time is convoluted..."             ║
╚══════════════════════════════════════════════════════════════╝

[SOUL EXAMINE] /home/undead/firelink_shrine.txt
  → Soul ID (inode): 12345678
  → Entity Type: Regular File
  → Soul Level (size): 1024 bytes
  → Fog Gate (perms): rwxr-xr-x
  → Shortcut Count: 1
  → Last Kindled: 2026-01-12 14:30:00

[BONFIRE WARP] /home/undead/lordran/
  → Entries found: 5
  → . (current bonfire)
  → .. (previous area)
  → undead_burg/
  → firelink_shrine.txt
  → praise_the_sun.msg -> ../messages/jolly_cooperation

[SHORTCUT UNLOCK] Creating hard link...
  → Target: /home/undead/lordran/undead_burg/boss_key
  → Shortcut: /home/undead/lordran/firelink_shrine/shortcut_to_boss
  → Success! Same soul (inode: 98765432)
  → Shortcut count now: 2

[MESSAGE LEFT] Creating symlink...
  → Message: "git gud"
  → Location: /home/undead/messages/helpful_tip
  → Success! Message readable.

[PHANTOM CHECK] /home/undead/messages/old_message
  → WARNING: Phantom message detected!
  → Target "/home/undead/deleted_area" no longer exists.
  → This is a DANGLING SYMLINK.

[EXPLORE LORDRAN] Starting from /home/undead/lordran/
  → Visiting: /home/undead/lordran/
  → Visiting: /home/undead/lordran/undead_burg/
  → Visiting: /home/undead/lordran/undead_burg/taurus_demon.boss
  → Visiting: /home/undead/lordran/firelink_shrine.txt
  → Total souls examined: 4
  → Bonfires visited: 2
  → Shortcuts found: 1
  → Messages read: 1
  → Phantoms encountered: 0

[ESTUS POOL] File descriptor management
  → Kindled flask 3: /home/undead/lordran/save.dat (O_RDWR)
  → Duplicated flask 3 → flask 4
  → Extinguished flask 3
  → Flask 4 still active (shared offset!)

Exploration complete. YOU DEFEATED.
```

---

### 3.1 💀 BONUS EXPERT : CUSTOM VFS LAYER (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★★☆ (9/10)

**Récompense :**
XP ×4

**Domaines Bonus :**
`Process, Struct`

#### 3.1.1 Consigne Bonus

**🎮 NIVEAU NG+ : IMPLÉMENTER UN MINI-VFS**

Créer une couche d'abstraction VFS qui permet d'unifier différents "backends" :
- Filesystem réel (via syscalls)
- Filesystem en mémoire (RAM disk)
- Filesystem réseau simulé

#### 3.1.2 Prototype Bonus

```c
// VFS operations structure (like Linux kernel)
typedef struct {
    soul_vessel_t *(*examine)(void *ctx, const char *path);
    bonfire_t *(*open_dir)(void *ctx, const char *path);
    ssize_t (*read)(void *ctx, int fd, void *buf, size_t count);
    ssize_t (*write)(void *ctx, int fd, const void *buf, size_t count);
    int (*create_link)(void *ctx, const char *target, const char *link);
} vfs_operations_t;

typedef struct {
    const char *name;           // "realfs", "memfs", "netfs"
    vfs_operations_t *ops;
    void *private_data;
} lordran_vfs_t;

lordran_vfs_t *vfs_register(const char *name, vfs_operations_t *ops);
int vfs_mount(lordran_vfs_t *vfs, const char *mount_point);
int vfs_umount(const char *mount_point);
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Input | Expected | Points | Concept |
|------|-------|----------|--------|---------|
| `test_soul_examine` | Valid file | Correct stat | 10 | 2.3.2 |
| `test_soul_examine_dir` | Directory | type='d' | 5 | 2.3.2.d |
| `test_true_path_reveal` | "../dir/../file" | Canonical path | 10 | 2.3.1.o |
| `test_same_soul` | Hard links | true | 5 | 2.3.2.b |
| `test_bonfire_list` | Directory | All entries | 10 | 2.3.3 |
| `test_dot_entries` | Any dir | . and .. present | 5 | 2.3.3.c-d |
| `test_unlock_shortcut` | Valid target | Link created | 10 | 2.3.4.a |
| `test_shortcut_same_inode` | After link | Same inode | 5 | 2.3.4.a |
| `test_leave_message` | Valid target | Symlink created | 10 | 2.3.4.f |
| `test_read_message` | Symlink | Target path | 5 | 2.3.4.k |
| `test_phantom_message` | Dangling | true | 5 | 2.3.4.i |
| `test_explore_tree` | Directory tree | All visited | 10 | 2.3.3 |
| `test_estus_kindle` | File | Valid FD | 5 | 2.3.5 |
| `test_estus_duplicate` | FD | Same file | 5 | 2.3.5.f |
| `test_fog_gates` | Various perms | Correct check | 5 | 2.3.2.e |
| **TOTAL** | | | **100** | |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include "lordran_fs.h"

#define TEST(name) printf("\n[TEST] %s\n", name)
#define OK() printf("  ✓ PASS\n")
#define FAIL(msg) printf("  ✗ FAIL: %s\n", msg)
#define ASSERT(cond, msg) if (!(cond)) { FAIL(msg); return 1; }

int test_soul_examine(void) {
    TEST("soul_examine on regular file");

    // Create test file
    int fd = open("/tmp/lordran_test.txt", O_CREAT | O_WRONLY, 0644);
    write(fd, "Praise the Sun!", 15);
    close(fd);

    soul_vessel_t *vessel = soul_examine("/tmp/lordran_test.txt");
    ASSERT(vessel != NULL, "vessel allocation failed");
    ASSERT(vessel->entity_type == 'f', "wrong entity type");
    ASSERT(vessel->soul_level == 15, "wrong size");
    ASSERT(vessel->shortcut_count == 1, "wrong link count");

    soul_release(vessel);
    unlink("/tmp/lordran_test.txt");

    OK();
    return 0;
}

int test_true_path_reveal(void) {
    TEST("true_path_reveal (realpath)");

    char *path = true_path_reveal("/tmp/../tmp/./");
    ASSERT(path != NULL, "realpath failed");
    ASSERT(strcmp(path, "/tmp") == 0, "wrong canonical path");

    free(path);
    OK();
    return 0;
}

int test_same_soul(void) {
    TEST("same_soul_check (hard links)");

    // Create file and hard link
    int fd = open("/tmp/soul_original.txt", O_CREAT | O_WRONLY, 0644);
    close(fd);
    link("/tmp/soul_original.txt", "/tmp/soul_shortcut.txt");

    bool same = same_soul_check("/tmp/soul_original.txt", "/tmp/soul_shortcut.txt");
    ASSERT(same == true, "should be same soul (inode)");

    unlink("/tmp/soul_original.txt");
    unlink("/tmp/soul_shortcut.txt");

    OK();
    return 0;
}

int test_bonfire_list(void) {
    TEST("bonfire_warp_to (list directory)");

    bonfire_t *bonfire = bonfire_warp_to("/tmp");
    ASSERT(bonfire != NULL, "bonfire allocation failed");
    ASSERT(bonfire->entry_count > 0, "no entries found");
    ASSERT(bonfire->has_dot == true, "missing . entry");
    ASSERT(bonfire->has_dotdot == true, "missing .. entry");

    bonfire_leave(bonfire);
    OK();
    return 0;
}

int test_leave_and_read_message(void) {
    TEST("leave_message and read_message (symlinks)");

    // Create target
    int fd = open("/tmp/message_target.txt", O_CREAT | O_WRONLY, 0644);
    write(fd, "git gud", 7);
    close(fd);

    // Leave message (symlink)
    int result = leave_message("/tmp/message_target.txt", "/tmp/helpful_tip");
    ASSERT(result == 0, "failed to leave message");

    // Read message
    char *target = read_message("/tmp/helpful_tip");
    ASSERT(target != NULL, "failed to read message");
    ASSERT(strcmp(target, "/tmp/message_target.txt") == 0, "wrong target");

    free(target);
    unlink("/tmp/helpful_tip");
    unlink("/tmp/message_target.txt");

    OK();
    return 0;
}

int test_phantom_message(void) {
    TEST("is_phantom_message (dangling symlink)");

    // Create dangling symlink
    symlink("/tmp/nonexistent_soul", "/tmp/phantom_message");

    bool phantom = is_phantom_message("/tmp/phantom_message");
    ASSERT(phantom == true, "should detect dangling symlink");

    unlink("/tmp/phantom_message");

    OK();
    return 0;
}

static int explore_callback(const char *path, const soul_vessel_t *vessel, void *data) {
    int *count = (int *)data;
    (*count)++;
    printf("    → Visited: %s\n", path);
    (void)vessel;
    return 0;
}

int test_explore_lordran(void) {
    TEST("explore_lordran (tree walk)");

    // Create test tree
    mkdir("/tmp/lordran_test", 0755);
    mkdir("/tmp/lordran_test/undead_burg", 0755);
    int fd = open("/tmp/lordran_test/bonfire.txt", O_CREAT | O_WRONLY, 0644);
    close(fd);
    fd = open("/tmp/lordran_test/undead_burg/boss.dat", O_CREAT | O_WRONLY, 0644);
    close(fd);

    int count = 0;
    int result = explore_lordran("/tmp/lordran_test", explore_callback, &count);
    ASSERT(result >= 0, "exploration failed");
    ASSERT(count >= 3, "not all entries visited");

    // Cleanup
    unlink("/tmp/lordran_test/undead_burg/boss.dat");
    rmdir("/tmp/lordran_test/undead_burg");
    unlink("/tmp/lordran_test/bonfire.txt");
    rmdir("/tmp/lordran_test");

    OK();
    return 0;
}

int main(void) {
    printf("╔══════════════════════════════════════════════════════════╗\n");
    printf("║    DARK SOULS: FILESYSTEM TEST — Prepare to Die          ║\n");
    printf("╚══════════════════════════════════════════════════════════╝\n");

    int failed = 0;

    failed += test_soul_examine();
    failed += test_true_path_reveal();
    failed += test_same_soul();
    failed += test_bonfire_list();
    failed += test_leave_and_read_message();
    failed += test_phantom_message();
    failed += test_explore_lordran();

    printf("\n════════════════════════════════════════════════════════════\n");
    if (failed == 0) {
        printf("YOU DEFEATED all tests. \\[T]/ Praise the Sun!\n");
    } else {
        printf("YOU DIED. %d test(s) failed.\n", failed);
    }

    return failed;
}
```

### 4.3 Solution de référence

```c
/* soul_vessel.c — Inode operations */
#include "lordran_fs.h"
#include <stdlib.h>
#include <string.h>
#include <errno.h>

static lordran_stats_t g_stats = {0};

soul_vessel_t *soul_examine(const char *path)
{
    struct stat st;
    soul_vessel_t *vessel;

    if (path == NULL)
        return NULL;

    if (stat(path, &st) == -1)
        return NULL;

    vessel = malloc(sizeof(soul_vessel_t));
    if (vessel == NULL)
        return NULL;

    vessel->soul_id = st.st_ino;
    vessel->fog_gate = st.st_mode;
    vessel->lord_uid = st.st_uid;
    vessel->covenant_gid = st.st_gid;
    vessel->soul_level = st.st_size;
    vessel->shortcut_count = st.st_nlink;
    vessel->last_kindled = st.st_atime;
    vessel->last_modified = st.st_mtime;
    vessel->soul_created = st.st_ctime;

    if (S_ISREG(st.st_mode))
        vessel->entity_type = 'f';
    else if (S_ISDIR(st.st_mode))
        vessel->entity_type = 'd';
    else if (S_ISLNK(st.st_mode))
        vessel->entity_type = 'l';
    else if (S_ISBLK(st.st_mode))
        vessel->entity_type = 'b';
    else if (S_ISCHR(st.st_mode))
        vessel->entity_type = 'c';
    else if (S_ISFIFO(st.st_mode))
        vessel->entity_type = 'p';
    else if (S_ISSOCK(st.st_mode))
        vessel->entity_type = 's';
    else
        vessel->entity_type = '?';

    strncpy(vessel->entity_name, path, 255);
    vessel->entity_name[255] = '\0';

    g_stats.souls_examined++;
    return vessel;
}

soul_vessel_t *soul_examine_no_follow(const char *path)
{
    struct stat st;
    soul_vessel_t *vessel;

    if (path == NULL)
        return NULL;

    if (lstat(path, &st) == -1)
        return NULL;

    vessel = malloc(sizeof(soul_vessel_t));
    if (vessel == NULL)
        return NULL;

    vessel->soul_id = st.st_ino;
    vessel->fog_gate = st.st_mode;
    vessel->lord_uid = st.st_uid;
    vessel->covenant_gid = st.st_gid;
    vessel->soul_level = st.st_size;
    vessel->shortcut_count = st.st_nlink;
    vessel->last_kindled = st.st_atime;
    vessel->last_modified = st.st_mtime;
    vessel->soul_created = st.st_ctime;

    if (S_ISLNK(st.st_mode))
        vessel->entity_type = 'l';
    else if (S_ISREG(st.st_mode))
        vessel->entity_type = 'f';
    else if (S_ISDIR(st.st_mode))
        vessel->entity_type = 'd';
    else
        vessel->entity_type = '?';

    strncpy(vessel->entity_name, path, 255);
    g_stats.souls_examined++;
    return vessel;
}

void soul_release(soul_vessel_t *vessel)
{
    free(vessel);
}

ino_t get_soul_id(const char *path)
{
    struct stat st;

    if (stat(path, &st) == -1)
        return 0;
    return st.st_ino;
}

nlink_t count_shortcuts(const char *path)
{
    struct stat st;

    if (stat(path, &st) == -1)
        return 0;
    return st.st_nlink;
}

bool same_soul_check(const char *path1, const char *path2)
{
    struct stat st1, st2;

    if (stat(path1, &st1) == -1 || stat(path2, &st2) == -1)
        return false;

    return (st1.st_ino == st2.st_ino && st1.st_dev == st2.st_dev);
}

/* shortcut_gates.c — Link operations */

int unlock_shortcut(const char *target, const char *shortcut_path)
{
    if (target == NULL || shortcut_path == NULL)
        return -1;

    if (link(target, shortcut_path) == -1)
        return -1;

    g_stats.shortcuts_found++;
    return 0;
}

int leave_message(const char *target, const char *message_path)
{
    if (target == NULL || message_path == NULL)
        return -1;

    if (symlink(target, message_path) == -1)
        return -1;

    g_stats.messages_read++;
    return 0;
}

char *read_message(const char *path)
{
    char *buffer;
    ssize_t len;
    struct stat st;

    if (path == NULL)
        return NULL;

    if (lstat(path, &st) == -1)
        return NULL;

    if (!S_ISLNK(st.st_mode))
        return NULL;

    buffer = malloc(st.st_size + 1);
    if (buffer == NULL)
        return NULL;

    len = readlink(path, buffer, st.st_size);
    if (len == -1)
    {
        free(buffer);
        return NULL;
    }

    buffer[len] = '\0';
    return buffer;
}

bool is_message(const char *path)
{
    struct stat st;

    if (lstat(path, &st) == -1)
        return false;

    return S_ISLNK(st.st_mode);
}

bool is_phantom_message(const char *path)
{
    struct stat st;

    if (!is_message(path))
        return false;

    /* If lstat succeeds but stat fails, target doesn't exist */
    if (stat(path, &st) == -1 && errno == ENOENT)
    {
        g_stats.phantoms_encountered++;
        return true;
    }

    return false;
}

/* bonfire_network.c — Directory operations */

bonfire_t *bonfire_warp_to(const char *path)
{
    DIR *dir;
    struct dirent *entry;
    bonfire_t *bonfire;
    size_t capacity = 16;

    if (path == NULL)
        return NULL;

    dir = opendir(path);
    if (dir == NULL)
        return NULL;

    bonfire = malloc(sizeof(bonfire_t));
    if (bonfire == NULL)
    {
        closedir(dir);
        return NULL;
    }

    strncpy(bonfire->path, path, 4095);
    bonfire->path[4095] = '\0';
    bonfire->entries = malloc(sizeof(bonfire_entry_t) * capacity);
    bonfire->entry_count = 0;
    bonfire->has_dot = false;
    bonfire->has_dotdot = false;

    while ((entry = readdir(dir)) != NULL)
    {
        if (bonfire->entry_count >= capacity)
        {
            capacity *= 2;
            bonfire->entries = realloc(bonfire->entries,
                                       sizeof(bonfire_entry_t) * capacity);
        }

        strncpy(bonfire->entries[bonfire->entry_count].name, entry->d_name, 255);
        bonfire->entries[bonfire->entry_count].soul_id = entry->d_ino;
        bonfire->entries[bonfire->entry_count].entity_type = entry->d_type;

        if (strcmp(entry->d_name, ".") == 0)
            bonfire->has_dot = true;
        else if (strcmp(entry->d_name, "..") == 0)
            bonfire->has_dotdot = true;

        bonfire->entry_count++;
    }

    closedir(dir);
    g_stats.bonfires_visited++;
    return bonfire;
}

void bonfire_leave(bonfire_t *bonfire)
{
    if (bonfire != NULL)
    {
        free(bonfire->entries);
        free(bonfire);
    }
}

/* explore_lordran.c — Tree walking */

static int explore_recursive(const char *path, undead_callback callback,
                            void *user_data, int depth, int max_depth)
{
    DIR *dir;
    struct dirent *entry;
    soul_vessel_t *vessel;
    char full_path[4096];
    int result = 0;

    if (max_depth >= 0 && depth > max_depth)
        return 0;

    vessel = soul_examine(path);
    if (vessel == NULL)
        return -1;

    if (callback != NULL)
        result = callback(path, vessel, user_data);

    if (result != 0 || vessel->entity_type != 'd')
    {
        soul_release(vessel);
        return result;
    }

    soul_release(vessel);

    dir = opendir(path);
    if (dir == NULL)
        return -1;

    while ((entry = readdir(dir)) != NULL)
    {
        if (strcmp(entry->d_name, ".") == 0 || strcmp(entry->d_name, "..") == 0)
            continue;

        snprintf(full_path, sizeof(full_path), "%s/%s", path, entry->d_name);

        result = explore_recursive(full_path, callback, user_data,
                                  depth + 1, max_depth);
        if (result != 0)
            break;
    }

    closedir(dir);
    return result;
}

int explore_lordran(const char *start, undead_callback callback, void *user_data)
{
    return explore_recursive(start, callback, user_data, 0, -1);
}

int explore_lordran_filtered(const char *start, undead_callback callback,
                            bool follow_messages, int max_depth, void *user_data)
{
    (void)follow_messages;  /* TODO: implement */
    return explore_recursive(start, callback, user_data, 0, max_depth);
}

/* Path operations */

char *true_path_reveal(const char *path)
{
    char *resolved;

    if (path == NULL)
        return NULL;

    resolved = realpath(path, NULL);
    return resolved;
}

bool is_absolute_path(const char *path)
{
    return (path != NULL && path[0] == '/');
}

/* Statistics */

void get_exploration_stats(lordran_stats_t *stats)
{
    if (stats != NULL)
        *stats = g_stats;
}

void reset_exploration_stats(void)
{
    memset(&g_stats, 0, sizeof(g_stats));
}
```

### 4.10 Solutions Mutantes

```c
/* MUTANT A: Utilise stat au lieu de lstat pour symlinks */
bool is_phantom_message_mutantA(const char *path)
{
    struct stat st;
    // ❌ Utilise stat qui suit le lien!
    if (stat(path, &st) == -1)
        return true;  // Faux positif si le LIEN existe mais pas la cible
    return false;
}

/* MUTANT B: Oublie de fermer le DIR */
bonfire_t *bonfire_warp_to_mutantB(const char *path)
{
    DIR *dir = opendir(path);
    bonfire_t *bonfire = malloc(sizeof(bonfire_t));
    // ... populate bonfire ...
    // ❌ Oublie closedir(dir)!
    return bonfire;  // Resource leak
}

/* MUTANT C: Ne vérifie pas st_dev pour same_soul */
bool same_soul_check_mutantC(const char *path1, const char *path2)
{
    struct stat st1, st2;
    stat(path1, &st1);
    stat(path2, &st2);
    // ❌ Ne vérifie que l'inode, pas le device!
    return (st1.st_ino == st2.st_ino);
    // Faux positif si fichiers sur différents FS
}

/* MUTANT D: Buffer overflow dans readlink */
char *read_message_mutantD(const char *path)
{
    char buffer[256];  // ❌ Taille fixe!
    readlink(path, buffer, sizeof(buffer));
    // Pas de null-termination, buffer overflow possible
    return strdup(buffer);
}

/* MUTANT E: Recursion infinie dans explore */
int explore_recursive_mutantE(const char *path, undead_callback cb, void *data)
{
    DIR *dir = opendir(path);
    struct dirent *entry;
    while ((entry = readdir(dir)) != NULL)
    {
        // ❌ Ne skip pas "." et ".."!
        explore_recursive_mutantE(entry->d_name, cb, data);
    }
    // Boucle infinie sur "."
}
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Inodes** : Métadonnées séparées du nom de fichier
2. **Hard links** : Plusieurs noms pour le même inode
3. **Symlinks** : Pointeurs vers un chemin (peut être invalide)
4. **Directory entries** : Mapping nom → inode
5. **File descriptors** : Handles pour les fichiers ouverts
6. **Tree traversal** : Exploration récursive avec gestion d'erreurs

### 5.3 Visualisation ASCII

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                         LORDRAN FILESYSTEM MAP                               ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║     DIRECTORY ENTRY          INODE (SOUL VESSEL)         DATA BLOCKS        ║
║     ┌─────────────┐          ┌─────────────────┐        ┌───────────┐       ║
║     │ "bonfire"   │────────▶│ inode #12345    │───────▶│ Actual    │       ║
║     │ inode: 12345│          │ type: file      │        │ file      │       ║
║     └─────────────┘          │ size: 1024      │        │ contents  │       ║
║                              │ perms: 0644     │        └───────────┘       ║
║     ┌─────────────┐          │ links: 2        │                            ║
║     │ "shortcut"  │────────▶│ uid: 1000       │    (Same data!)            ║
║     │ inode: 12345│  ▲       │ timestamps...   │                            ║
║     └─────────────┘  │       └─────────────────┘                            ║
║                      │                                                       ║
║     SAME INODE = HARD LINK (shortcut in Dark Souls)                         ║
║                                                                              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║     SYMBOLIC LINK (MESSAGE)                                                  ║
║     ┌─────────────┐          ┌─────────────────┐        ┌───────────┐       ║
║     │ "message"   │────────▶│ inode #99999    │───────▶│ "/path/to │       ║
║     │ inode: 99999│          │ type: symlink   │        │ /target"  │       ║
║     └─────────────┘          │ size: 15        │        └───────────┘       ║
║                              └─────────────────┘             │              ║
║                                                              ▼              ║
║                                                    ┌─────────────────┐      ║
║     If target deleted = PHANTOM MESSAGE!           │ Target file     │      ║
║     (Dangling symlink)                             │ (may not exist) │      ║
║                                                    └─────────────────┘      ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### 5.8 Mnémotechniques

#### 🎮 MEME : "YOU DIED" — Mais vos données persistent

```
     ┌─────────────────────────────────────────────┐
     │                                             │
     │              Y O U   D I E D               │
     │                                             │
     │   Mais votre SOUL VESSEL (inode) persiste! │
     │                                             │
     │   Le NOM peut disparaître (unlink)          │
     │   Mais tant qu'il reste des SHORTCUTS       │
     │   (hard links), l'ÂME (data) survit.        │
     │                                             │
     │   link_count == 0 → Soul truly dies         │
     │                                             │
     └─────────────────────────────────────────────┘
```

**Pour retenir :**
- **Soul Vessel** = Inode (métadonnées)
- **Shortcut** = Hard link (même inode, nom différent)
- **Message** = Symlink (pointe vers un chemin)
- **Phantom** = Dangling symlink (cible supprimée)
- **Bonfire** = Directory (point de repère)
- **Fog Gate** = Permissions (bloque l'accès)

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Concept | Maîtrisé | À revoir |
|---------|----------|----------|
| stat/lstat/fstat | ☐ | ☐ |
| Inode structure | ☐ | ☐ |
| Hard links | ☐ | ☐ |
| Symbolic links | ☐ | ☐ |
| opendir/readdir | ☐ | ☐ |
| realpath | ☐ | ☐ |
| File descriptors | ☐ | ☐ |
| Permissions | ☐ | ☐ |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.1-27-synth-lordran-fs",
    "generated_at": "2026-01-12 16:00:00",

    "metadata": {
      "exercise_id": "2.3.1-27-synth",
      "exercise_name": "lordran_filesystem",
      "module": "2.3",
      "module_name": "File System Fundamentals",
      "type": "complet",
      "tier": 3,
      "phase": 2,
      "difficulty": 8,
      "difficulty_stars": "★★★★★★★★☆☆",
      "language": "c",
      "duration_minutes": 480,
      "xp_base": 600,
      "tags": ["filesystem", "inodes", "links", "directories", "dark-souls"],
      "meme_reference": "Dark Souls - Praise the Sun"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "Don't give up, skeleton!"*
*Lordran Filesystem: Because exploring filesystems should feel like an adventure*
