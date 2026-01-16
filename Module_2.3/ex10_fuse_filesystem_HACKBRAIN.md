# Exercice 2.3.10 : prestige_filesystem

**Module :**
2.3 — File Systems

**Concept :**
FUSE (Filesystem in Userspace) — L'Art de l'Illusion Parfaite

**Difficulté :**
★★★★★★★★☆☆ (8/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (tous concepts FUSE a→l)

**Langage :**
C (c17)

**Prérequis :**
- 2.3.0-2.3.9 (Concepts filesystem de base)
- Pointeurs et structures complexes
- Gestion mémoire dynamique
- Callbacks et function pointers

**Domaines :**
FS, Mem, Struct

**Durée estimée :**
600 min (10h)

**XP Base :**
500

**Complexité :**
T7 O(n) × S6 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex10/
├── prestige_fs.h
├── prestige_fs.c
├── prestige_tricks.c      # Les operations FUSE
├── prestige_props.c       # Gestion des nodes
└── Makefile
```

**Fonctions autorisées :**
- `fuse_main`, `fuse_get_context` (libfuse3)
- `malloc`, `free`, `realloc`, `calloc`
- `memcpy`, `memset`, `memmove`
- `strlen`, `strncpy`, `strcmp`, `strncmp`, `strdup`
- `strchr`, `strrchr`
- `time`, `localtime`
- `printf`, `fprintf`, `snprintf`
- `errno` (accès lecture/écriture)

**Fonctions interdites :**
- Tout appel système filesystem direct (`open`, `read`, `write`, `stat`, etc.)
- Le but est de LES IMPLÉMENTER, pas de les utiliser !

### 1.2 Consigne

**🎩 THE PRESTIGE — L'Art de l'Illusion Parfaite**

*"Every great magic trick consists of three parts or acts."* — Cutter

Dans le film culte de Christopher Nolan, deux magiciens rivaux, Robert Angier et Alfred Borden,
s'affrontent pour créer l'illusion parfaite. Leur obsession : **The Transported Man** — faire
disparaître quelqu'un d'un endroit pour le faire réapparaître instantanément ailleurs.

**FUSE (Filesystem in Userspace)** est exactement ça : **l'art de l'illusion en programmation**.
Tu vas créer un programme qui se fait passer pour un vrai filesystem aux yeux du kernel Linux.
Comme un magicien qui fait croire à son public qu'il défie les lois de la physique, ton programme
va faire croire au système d'exploitation qu'il communique avec un vrai disque dur.

```
THE THREE ACTS OF MAGIC:
═══════════════════════════════════════════════════════════════════

1. THE PLEDGE (La Promesse) — Tu montres quelque chose d'ordinaire
   → Mount : Tu présentes ton programme au kernel
   → "Voici un filesystem tout à fait normal..."

2. THE TURN (Le Changement) — Tu transformes l'ordinaire en extraordinaire
   → Operations : Le kernel demande stat(), read(), write()...
   → Ton programme intercepte et répond comme un vrai FS

3. THE PRESTIGE (Le Prestige) — Le moment où l'impossible devient réel
   → Le kernel est convaincu, les utilisateurs voient des fichiers
   → "Are you watching closely?"

═══════════════════════════════════════════════════════════════════
```

**Ta mission :**

Créer `prestige_fs`, un filesystem FUSE complet qui maintient une arborescence de fichiers
entièrement en mémoire. Comme la machine de Tesla dans le film, ton code doit être capable
de créer l'illusion parfaite — indiscernable d'un vrai filesystem.

### 1.2.2 Consigne Académique

Implémenter un système de fichiers complet utilisant l'API FUSE (Filesystem in Userspace).
Le programme doit intercepter tous les appels système relatifs aux fichiers et fournir
une implémentation en espace utilisateur, stockant les données en mémoire RAM.

**Entrée :**
- `mountpoint` : Point de montage pour le filesystem
- Options FUSE standard (`-d` pour debug, `-f` pour foreground)

**Sortie :**
- Filesystem fonctionnel accessible via le point de montage
- Support complet des opérations POSIX standard

**Contraintes :**
- Utiliser FUSE 3 API (libfuse3)
- Implémenter minimum 15 opérations FUSE
- Stockage en mémoire (pas de persistance requise)
- Gestion correcte des codes d'erreur errno

### 1.3 Prototypes

```c
#define FUSE_USE_VERSION 31
#include <fuse3/fuse.h>

/*
 * =============================================================================
 *                    THE PRESTIGE FILESYSTEM — DATA STRUCTURES
 * =============================================================================
 *
 * "The secret impresses no one. The trick you use it for is everything."
 *                                                              — Alfred Borden
 */

/* Un "prop" (accessoire) dans notre spectacle de magie */
typedef struct prestige_prop {
    char                    name[256];          /* Nom de l'accessoire */
    mode_t                  mode;               /* Costume (permissions) */
    uid_t                   uid;                /* Propriétaire */
    gid_t                   gid;                /* Troupe */
    size_t                  size;               /* Taille réelle */
    time_t                  atime;              /* Dernier regard */
    time_t                  mtime;              /* Dernière modification */
    time_t                  ctime;              /* Création */
    nlink_t                 nlink;              /* Références */
    uint64_t                inode;              /* Identité secrète */

    char                   *secret_content;     /* Le vrai contenu (fichiers) */
    char                   *mirror_target;      /* Pour symlinks (doubles) */

    struct prestige_prop   *children;           /* Accessoires dans ce cabinet */
    struct prestige_prop   *next;               /* Prochain dans la liste */
    struct prestige_prop   *parent;             /* Le cabinet parent */
} prestige_prop_t;

/* La scène principale — notre filesystem */
typedef struct {
    prestige_prop_t        *stage;              /* Root = La scène principale */
    size_t                  total_illusion;     /* Taille totale de l'illusion */
    uint64_t                next_identity;      /* Prochain numéro d'identité */

    /* Statistiques du spectacle (pour les critiques) */
    struct {
        uint64_t            examine_calls;      /* getattr */
        uint64_t            reveal_calls;       /* readdir */
        uint64_t            pull_calls;         /* read */
        uint64_t            plant_calls;        /* write */
        uint64_t            total_read;
        uint64_t            total_written;
        double              avg_trick_latency_us;
    } performance;
} prestige_theater_t;

/*
 * =============================================================================
 *                         THE MAGIC TRICKS (FUSE Operations)
 * =============================================================================
 */

/* 2.3.21.a: examine_prop — Examiner un accessoire (getattr/stat) */
static int prestige_examine_prop(const char *path, struct stat *stbuf,
                                  struct fuse_file_info *fi);

/* 2.3.21.b: reveal_cabinet — Révéler le contenu d'un cabinet (readdir) */
static int prestige_reveal_cabinet(const char *path, void *buf,
                                    fuse_fill_dir_t filler,
                                    off_t offset, struct fuse_file_info *fi,
                                    enum fuse_readdir_flags flags);

/* 2.3.21.c: access_vault — Ouvrir le coffre (open) */
static int prestige_access_vault(const char *path, struct fuse_file_info *fi);

/* 2.3.21.d: pull_rabbit — Sortir le lapin du chapeau (read) */
static int prestige_pull_rabbit(const char *path, char *buf, size_t size,
                                 off_t offset, struct fuse_file_info *fi);

/* 2.3.21.e: plant_evidence — Planter des preuves dans le coffre (write) */
static int prestige_plant_evidence(const char *path, const char *buf,
                                    size_t size, off_t offset,
                                    struct fuse_file_info *fi);

/* 2.3.21.f: materialize — Faire apparaître un accessoire (create) */
static int prestige_materialize(const char *path, mode_t mode,
                                 struct fuse_file_info *fi);

/* 2.3.21.g: vanish — Faire disparaître (unlink) */
static int prestige_vanish(const char *path);

/* 2.3.21.h: erect_stage / collapse_stage — Scènes (mkdir/rmdir) */
static int prestige_erect_stage(const char *path, mode_t mode);
static int prestige_collapse_stage(const char *path);

/* 2.3.21.i: transported_man — LE tour signature ! (rename) */
static int prestige_transported_man(const char *from, const char *to,
                                     unsigned int flags);

/* 2.3.21.j: guillotine — Couper ! (truncate) */
static int prestige_guillotine(const char *path, off_t size,
                                struct fuse_file_info *fi);

/* 2.3.21.k: costume_change / identity_swap (chmod/chown) */
static int prestige_costume_change(const char *path, mode_t mode,
                                    struct fuse_file_info *fi);
static int prestige_identity_swap(const char *path, uid_t uid, gid_t gid,
                                   struct fuse_file_info *fi);

/* 2.3.21.l: create_double / check_mirror — Doubles (symlink/readlink) */
static int prestige_create_double(const char *target, const char *linkpath);
static int prestige_check_mirror(const char *path, char *buf, size_t size);

/* Initialisation et nettoyage du spectacle */
static void *prestige_open_curtains(struct fuse_conn_info *conn,
                                     struct fuse_config *cfg);
static void prestige_final_bow(void *private_data);

/*
 * =============================================================================
 *                    THE PLAYBOOK (fuse_operations structure)
 * =============================================================================
 */

static const struct fuse_operations prestige_playbook = {
    .getattr    = prestige_examine_prop,      /* "Examine the prop" */
    .readdir    = prestige_reveal_cabinet,    /* "Open the cabinet" */
    .open       = prestige_access_vault,      /* "Access the vault" */
    .read       = prestige_pull_rabbit,       /* "Pull the rabbit" */
    .write      = prestige_plant_evidence,    /* "Plant the evidence" */
    .create     = prestige_materialize,       /* "Materialize!" */
    .unlink     = prestige_vanish,            /* "Vanish!" */
    .mkdir      = prestige_erect_stage,       /* "Erect the stage" */
    .rmdir      = prestige_collapse_stage,    /* "Strike the set" */
    .rename     = prestige_transported_man,   /* "THE TRANSPORTED MAN!" */
    .truncate   = prestige_guillotine,        /* "The Guillotine!" */
    .chmod      = prestige_costume_change,    /* "Costume change" */
    .chown      = prestige_identity_swap,     /* "Identity swap" */
    .symlink    = prestige_create_double,     /* "The Double" */
    .readlink   = prestige_check_mirror,      /* "Check the mirror" */
    .init       = prestige_open_curtains,     /* "Open the curtains" */
    .destroy    = prestige_final_bow,         /* "Take a bow" */
};

/*
 * =============================================================================
 *                           BACKSTAGE HELPERS
 * =============================================================================
 */

/* Trouver un prop sur scène */
prestige_prop_t *backstage_find_prop(prestige_theater_t *theater,
                                      const char *path);

/* Créer un nouveau prop */
prestige_prop_t *backstage_craft_prop(prestige_theater_t *theater,
                                       const char *path, mode_t mode);

/* Retirer un prop de la scène */
int backstage_remove_prop(prestige_theater_t *theater, const char *path);

/* Utilitaires de chemin */
char *backstage_parent_path(const char *path);
const char *backstage_prop_name(const char *path);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 La Magie Derrière FUSE

```
"Now you're looking for the secret. But you won't find it because of
 course, you're not really looking. You don't really want to work it out.
 You want to be fooled." — Cutter
```

**FUSE** a été créé par Miklos Szeredi en 2001. L'idée révolutionnaire : permettre à
n'importe qui de créer un filesystem sans modifier le kernel Linux. Avant FUSE, créer
un filesystem nécessitait d'écrire un module kernel — une tâche réservée aux experts.

### 2.2 L'Architecture de l'Illusion

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           L'ARCHITECTURE FUSE                                │
│                        (Comment l'illusion fonctionne)                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   USER SPACE                          KERNEL SPACE                          │
│   (Ta salle de spectacle)             (Le public)                           │
│                                                                             │
│   ┌─────────────────┐                ┌─────────────────┐                   │
│   │  TON PROGRAMME  │                │    VFS LAYER    │                   │
│   │  prestige_fs    │                │  (Le critique)  │                   │
│   │                 │                │                 │                   │
│   │  ┌───────────┐  │   libfuse      │  "Je veux voir  │                   │
│   │  │ fuse_ops  │◄─┼────────────────┼──  stat() !"    │                   │
│   │  │           │  │                │                 │                   │
│   │  │ getattr() │──┼────────────────┼─► "Voici les    │                   │
│   │  │ read()    │  │   Réponse      │    métadonnées" │                   │
│   │  │ write()   │  │                │                 │                   │
│   │  └───────────┘  │                └────────┬────────┘                   │
│   └─────────────────┘                         │                            │
│                                               ▼                            │
│                                      ┌─────────────────┐                   │
│                                      │   /dev/fuse     │                   │
│                                      │  (La trappe)    │                   │
│                                      └─────────────────┘                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Les Trois Actes de FUSE

| Acte | Film | FUSE | Détail |
|------|------|------|--------|
| **The Pledge** | "I show you something ordinary" | `fuse_main()` | Tu présentes ton programme |
| **The Turn** | "I make it do something extraordinary" | Callbacks | Le kernel fait des requêtes |
| **The Prestige** | "The impossible becomes real" | Réponses | Le kernel croit à l'illusion |

### SECTION 2.5 : DANS LA VRAIE VIE

**Qui utilise FUSE ?**

| Métier | Utilisation | Exemples |
|--------|-------------|----------|
| **DevOps** | Montage de stockage cloud | s3fs (Amazon S3), gcsfuse (Google Cloud) |
| **Security Engineer** | Filesystems chiffrés | EncFS, gocryptfs, VeraCrypt |
| **Data Engineer** | Accès transparent aux archives | archivemount, fuse-zip |
| **SRE** | Debugging et profiling | sshfs pour accès distant |
| **Researcher** | Systèmes de fichiers expérimentaux | FUSE pour prototypage rapide |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
prestige_fs.h  prestige_fs.c  prestige_tricks.c  prestige_props.c  Makefile

$ make
gcc -Wall -Wextra -std=c17 $(pkg-config --cflags fuse3) -c prestige_fs.c
gcc -Wall -Wextra -std=c17 $(pkg-config --cflags fuse3) -c prestige_tricks.c
gcc -Wall -Wextra -std=c17 $(pkg-config --cflags fuse3) -c prestige_props.c
gcc -o prestige_fs prestige_fs.o prestige_tricks.o prestige_props.o $(pkg-config --libs fuse3)

$ mkdir -p /tmp/magic_show

$ ./prestige_fs /tmp/magic_show
[Prestige FS] The curtains are open. The show begins.
[Prestige FS] Are you watching closely?

# Dans un autre terminal :
$ cd /tmp/magic_show
$ echo "The secret impresses no one" > secret.txt
$ cat secret.txt
The secret impresses no one
$ ls -la
total 4
drwxr-xr-x 2 user user    0 Jan 12 15:00 .
drwxrwxrwt 3 root root 4096 Jan 12 15:00 ..
-rw-r--r-- 1 user user   28 Jan 12 15:00 secret.txt
$ mkdir tricks
$ ln -s secret.txt tricks/double
$ readlink tricks/double
secret.txt
$ mv secret.txt tricks/transported.txt
$ ls tricks/
double  transported.txt

$ fusermount -u /tmp/magic_show
[Prestige FS] The final bow. *applause*
```

---

## ⚡ SECTION 3.1 : BONUS STANDARD (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★★☆ (9/10)

**Récompense :**
XP ×2

**Time Complexity attendue :**
O(log n) pour les recherches

**Space Complexity attendue :**
O(n) + optimisation de stockage

### 3.1.1 Consigne Bonus — The Tesla Machine

**🎩 "Nothing is impossible, Mr. Angier."** — Nikola Tesla

Dans le film, Tesla crée une machine qui duplique instantanément tout ce qu'on y place.
Pour le bonus, implémente l'une de ces extensions :

**Option A : Tar Archive FS (Niveau STANDARD)**
```c
/* Monte un fichier .tar comme un filesystem
 * L'utilisateur peut naviguer dans l'archive comme si c'était un dossier */
int tesla_tar_mount(const char *tarfile, const char *mountpoint);
```

**Option B : Persistence (Niveau AVANCÉ)**
```c
/* Sauvegarde l'état du FS dans un fichier et le restaure au prochain mount */
int tesla_persist_state(prestige_theater_t *theater, const char *savefile);
int tesla_restore_state(prestige_theater_t *theater, const char *savefile);
```

**Option C : Encryption (Niveau EXPERT)**
```c
/* Chiffrement transparent de tout le contenu */
int tesla_encrypt_theater(prestige_theater_t *theater, const char *key);
```

### 3.1.2 Prototypes Bonus

```c
/* Pour Option A : Tar FS */
typedef struct {
    int          fd;              /* FD du fichier tar */
    tar_header  *entries;         /* Liste des entrées */
    size_t       count;
} tesla_archive_t;

int tesla_archive_open(tesla_archive_t *arch, const char *path);
prestige_prop_t *tesla_archive_to_props(tesla_archive_t *arch);
void tesla_archive_close(tesla_archive_t *arch);
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette (tableau des tests)

| # | Test | Input | Expected | Points |
|---|------|-------|----------|--------|
| 1 | `getattr_root` | path="/" | mode=S_IFDIR\|0755, nlink>=2 | 5 |
| 2 | `getattr_file` | create "test", getattr | mode=S_IFREG\|mode_used | 5 |
| 3 | `getattr_missing` | path="/nonexist" | -ENOENT | 3 |
| 4 | `readdir_root` | readdir "/" | contains ".", ".." | 5 |
| 5 | `readdir_populated` | create files, readdir | all files listed | 5 |
| 6 | `create_file` | create "/test.txt" | success, file exists | 5 |
| 7 | `create_nested` | create "/a/b/c.txt" after mkdir | success | 5 |
| 8 | `open_existing` | open created file | success (fd-like) | 5 |
| 9 | `open_nonexist` | open "/nofile" | -ENOENT | 3 |
| 10 | `read_write_basic` | write "hello", read back | "hello" | 10 |
| 11 | `read_offset` | write 100 bytes, read at offset 50 | correct 50 bytes | 5 |
| 12 | `write_extend` | write beyond current size | file grows | 5 |
| 13 | `unlink_file` | create then unlink | file gone | 5 |
| 14 | `unlink_nonexist` | unlink "/nofile" | -ENOENT | 2 |
| 15 | `mkdir_basic` | mkdir "/newdir" | dir exists, S_IFDIR | 5 |
| 16 | `mkdir_nested` | mkdir "/a/b/c" (a,b exist) | success | 3 |
| 17 | `rmdir_empty` | mkdir then rmdir | dir gone | 5 |
| 18 | `rmdir_nonempty` | rmdir dir with files | -ENOTEMPTY | 3 |
| 19 | `rename_file` | rename "/a" to "/b" | /b exists, /a gone | 5 |
| 20 | `rename_overwrite` | rename onto existing | target replaced | 3 |
| 21 | `truncate_shrink` | 100 bytes -> truncate(50) | size=50 | 3 |
| 22 | `truncate_grow` | 10 bytes -> truncate(100) | size=100, zeros | 3 |
| 23 | `chmod_basic` | create 0644 -> chmod 0755 | mode=0755 | 3 |
| 24 | `symlink_create` | symlink "target" "/link" | link exists | 5 |
| 25 | `readlink_valid` | readlink created symlink | returns target | 3 |
| 26 | `stress_many_files` | create 1000 files | all accessible | 5 |
| 27 | `concurrent_access` | parallel reads/writes | no corruption | 5 |

**Total : 100 points**

### 4.2 main.c de test

```c
#define FUSE_USE_VERSION 31
#include <fuse3/fuse.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <sys/stat.h>
#include <unistd.h>
#include <fcntl.h>
#include <dirent.h>

#include "prestige_fs.h"

/* Test framework */
static int tests_run = 0;
static int tests_passed = 0;

#define TEST(name) static int test_##name(void)
#define RUN_TEST(name) do { \
    printf("Test %s: ", #name); \
    tests_run++; \
    if (test_##name()) { \
        printf("OK\n"); \
        tests_passed++; \
    } else { \
        printf("FAIL\n"); \
    } \
} while(0)

#define ASSERT(cond) do { if (!(cond)) { printf("ASSERT FAILED: %s\n", #cond); return 0; } } while(0)
#define ASSERT_EQ(a, b) ASSERT((a) == (b))
#define ASSERT_NE(a, b) ASSERT((a) != (b))
#define ASSERT_STR_EQ(a, b) ASSERT(strcmp((a), (b)) == 0)

/* Mount point for testing */
static const char *MOUNT = "/tmp/prestige_test";

TEST(getattr_root) {
    struct stat st;
    ASSERT_EQ(stat(MOUNT, &st), 0);
    ASSERT(S_ISDIR(st.st_mode));
    ASSERT(st.st_nlink >= 2);
    return 1;
}

TEST(create_and_write) {
    char path[256];
    snprintf(path, sizeof(path), "%s/test_create.txt", MOUNT);

    int fd = open(path, O_CREAT | O_WRONLY, 0644);
    ASSERT_NE(fd, -1);

    const char *msg = "The secret impresses no one";
    ssize_t written = write(fd, msg, strlen(msg));
    ASSERT_EQ(written, (ssize_t)strlen(msg));

    close(fd);

    /* Verify it exists */
    struct stat st;
    ASSERT_EQ(stat(path, &st), 0);
    ASSERT_EQ(st.st_size, (off_t)strlen(msg));

    /* Cleanup */
    unlink(path);
    return 1;
}

TEST(read_back) {
    char path[256];
    snprintf(path, sizeof(path), "%s/test_read.txt", MOUNT);

    /* Write */
    int fd = open(path, O_CREAT | O_WRONLY, 0644);
    ASSERT_NE(fd, -1);
    const char *msg = "Are you watching closely?";
    write(fd, msg, strlen(msg));
    close(fd);

    /* Read back */
    fd = open(path, O_RDONLY);
    ASSERT_NE(fd, -1);
    char buf[256] = {0};
    ssize_t n = read(fd, buf, sizeof(buf) - 1);
    ASSERT_EQ(n, (ssize_t)strlen(msg));
    ASSERT_STR_EQ(buf, msg);
    close(fd);

    unlink(path);
    return 1;
}

TEST(mkdir_rmdir) {
    char path[256];
    snprintf(path, sizeof(path), "%s/test_dir", MOUNT);

    ASSERT_EQ(mkdir(path, 0755), 0);

    struct stat st;
    ASSERT_EQ(stat(path, &st), 0);
    ASSERT(S_ISDIR(st.st_mode));

    ASSERT_EQ(rmdir(path), 0);
    ASSERT_NE(stat(path, &st), 0);
    ASSERT_EQ(errno, ENOENT);

    return 1;
}

TEST(rename_transported_man) {
    char from[256], to[256];
    snprintf(from, sizeof(from), "%s/angier.txt", MOUNT);
    snprintf(to, sizeof(to), "%s/borden.txt", MOUNT);

    /* Create source */
    int fd = open(from, O_CREAT | O_WRONLY, 0644);
    write(fd, "I", 1);
    close(fd);

    /* The Transported Man! */
    ASSERT_EQ(rename(from, to), 0);

    /* Source should be gone */
    struct stat st;
    ASSERT_NE(stat(from, &st), 0);

    /* Destination should exist */
    ASSERT_EQ(stat(to, &st), 0);

    unlink(to);
    return 1;
}

TEST(symlink_double) {
    char target[256], link[256];
    snprintf(target, sizeof(target), "%s/original.txt", MOUNT);
    snprintf(link, sizeof(link), "%s/double.txt", MOUNT);

    /* Create target */
    int fd = open(target, O_CREAT | O_WRONLY, 0644);
    write(fd, "real", 4);
    close(fd);

    /* Create the double */
    ASSERT_EQ(symlink("original.txt", link), 0);

    /* Check the mirror */
    char buf[256] = {0};
    ssize_t n = readlink(link, buf, sizeof(buf) - 1);
    ASSERT(n > 0);
    ASSERT_STR_EQ(buf, "original.txt");

    unlink(link);
    unlink(target);
    return 1;
}

TEST(truncate_guillotine) {
    char path[256];
    snprintf(path, sizeof(path), "%s/victim.txt", MOUNT);

    int fd = open(path, O_CREAT | O_WRONLY, 0644);
    write(fd, "1234567890", 10);
    close(fd);

    /* The Guillotine! */
    ASSERT_EQ(truncate(path, 5), 0);

    struct stat st;
    stat(path, &st);
    ASSERT_EQ(st.st_size, 5);

    unlink(path);
    return 1;
}

int main(void) {
    printf("\n");
    printf("╔═══════════════════════════════════════════════════════════╗\n");
    printf("║           THE PRESTIGE FILESYSTEM — TEST SUITE            ║\n");
    printf("║              'Are you watching closely?'                  ║\n");
    printf("╚═══════════════════════════════════════════════════════════╝\n\n");

    RUN_TEST(getattr_root);
    RUN_TEST(create_and_write);
    RUN_TEST(read_back);
    RUN_TEST(mkdir_rmdir);
    RUN_TEST(rename_transported_man);
    RUN_TEST(symlink_double);
    RUN_TEST(truncate_guillotine);

    printf("\n══════════════════════════════════════════════════════════\n");
    printf("Results: %d/%d tests passed\n", tests_passed, tests_run);
    printf("══════════════════════════════════════════════════════════\n");

    if (tests_passed == tests_run) {
        printf("\n🎩 \"The trick is... I was there the whole time.\"\n");
        printf("   All tests passed. Take a bow!\n\n");
    }

    return tests_passed == tests_run ? 0 : 1;
}
```

### 4.3 Solution de référence

```c
/* prestige_fs.c — The Prestige Filesystem */
#define FUSE_USE_VERSION 31
#include <fuse3/fuse.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <time.h>
#include <unistd.h>
#include <sys/types.h>

#include "prestige_fs.h"

/* Global theater (our filesystem state) */
static prestige_theater_t *g_theater = NULL;

/*
 * =============================================================================
 *                          BACKSTAGE HELPERS
 * =============================================================================
 */

static prestige_prop_t *create_prop_node(const char *name, mode_t mode,
                                          prestige_prop_t *parent)
{
    prestige_prop_t *prop = calloc(1, sizeof(prestige_prop_t));
    if (!prop)
        return NULL;

    strncpy(prop->name, name, sizeof(prop->name) - 1);
    prop->mode = mode;
    prop->uid = getuid();
    prop->gid = getgid();
    prop->nlink = S_ISDIR(mode) ? 2 : 1;
    prop->atime = prop->mtime = prop->ctime = time(NULL);
    prop->inode = g_theater->next_identity++;
    prop->parent = parent;

    /* If parent is a directory, increment its nlink */
    if (parent && S_ISDIR(mode))
        parent->nlink++;

    return prop;
}

static void free_prop_recursive(prestige_prop_t *prop)
{
    if (!prop)
        return;

    /* Free children first */
    prestige_prop_t *child = prop->children;
    while (child) {
        prestige_prop_t *next = child->next;
        free_prop_recursive(child);
        child = next;
    }

    free(prop->secret_content);
    free(prop->mirror_target);
    free(prop);
}

prestige_prop_t *backstage_find_prop(prestige_theater_t *theater,
                                      const char *path)
{
    if (!theater || !path)
        return NULL;

    /* Root case */
    if (strcmp(path, "/") == 0)
        return theater->stage;

    /* Tokenize path */
    char *path_copy = strdup(path);
    if (!path_copy)
        return NULL;

    prestige_prop_t *current = theater->stage;
    char *token = strtok(path_copy, "/");

    while (token && current) {
        prestige_prop_t *child = current->children;
        prestige_prop_t *found = NULL;

        while (child) {
            if (strcmp(child->name, token) == 0) {
                found = child;
                break;
            }
            child = child->next;
        }

        current = found;
        token = strtok(NULL, "/");
    }

    free(path_copy);
    return current;
}

char *backstage_parent_path(const char *path)
{
    if (!path || strcmp(path, "/") == 0)
        return strdup("/");

    char *copy = strdup(path);
    char *last_slash = strrchr(copy, '/');

    if (last_slash == copy) {
        free(copy);
        return strdup("/");
    }

    if (last_slash)
        *last_slash = '\0';

    return copy;
}

const char *backstage_prop_name(const char *path)
{
    if (!path)
        return NULL;

    const char *last_slash = strrchr(path, '/');
    return last_slash ? last_slash + 1 : path;
}

prestige_prop_t *backstage_craft_prop(prestige_theater_t *theater,
                                       const char *path, mode_t mode)
{
    if (!theater || !path)
        return NULL;

    char *parent_path = backstage_parent_path(path);
    prestige_prop_t *parent = backstage_find_prop(theater, parent_path);
    free(parent_path);

    if (!parent || !S_ISDIR(parent->mode))
        return NULL;

    const char *name = backstage_prop_name(path);
    prestige_prop_t *prop = create_prop_node(name, mode, parent);

    if (prop) {
        /* Add to parent's children list */
        prop->next = parent->children;
        parent->children = prop;
    }

    return prop;
}

int backstage_remove_prop(prestige_theater_t *theater, const char *path)
{
    if (!theater || !path || strcmp(path, "/") == 0)
        return -EINVAL;

    prestige_prop_t *prop = backstage_find_prop(theater, path);
    if (!prop)
        return -ENOENT;

    prestige_prop_t *parent = prop->parent;
    if (!parent)
        return -EINVAL;

    /* Remove from parent's children list */
    prestige_prop_t **pp = &parent->children;
    while (*pp && *pp != prop)
        pp = &(*pp)->next;

    if (*pp)
        *pp = prop->next;

    /* If it was a directory, decrement parent's nlink */
    if (S_ISDIR(prop->mode))
        parent->nlink--;

    free_prop_recursive(prop);
    return 0;
}

/*
 * =============================================================================
 *                          FUSE OPERATIONS
 * =============================================================================
 */

static int prestige_examine_prop(const char *path, struct stat *stbuf,
                                  struct fuse_file_info *fi)
{
    (void)fi;
    g_theater->performance.examine_calls++;

    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;

    memset(stbuf, 0, sizeof(struct stat));
    stbuf->st_ino = prop->inode;
    stbuf->st_mode = prop->mode;
    stbuf->st_nlink = prop->nlink;
    stbuf->st_uid = prop->uid;
    stbuf->st_gid = prop->gid;
    stbuf->st_size = prop->size;
    stbuf->st_atime = prop->atime;
    stbuf->st_mtime = prop->mtime;
    stbuf->st_ctime = prop->ctime;

    return 0;
}

static int prestige_reveal_cabinet(const char *path, void *buf,
                                    fuse_fill_dir_t filler,
                                    off_t offset, struct fuse_file_info *fi,
                                    enum fuse_readdir_flags flags)
{
    (void)offset;
    (void)fi;
    (void)flags;
    g_theater->performance.reveal_calls++;

    prestige_prop_t *dir = backstage_find_prop(g_theater, path);
    if (!dir)
        return -ENOENT;
    if (!S_ISDIR(dir->mode))
        return -ENOTDIR;

    filler(buf, ".", NULL, 0, 0);
    filler(buf, "..", NULL, 0, 0);

    prestige_prop_t *child = dir->children;
    while (child) {
        filler(buf, child->name, NULL, 0, 0);
        child = child->next;
    }

    return 0;
}

static int prestige_access_vault(const char *path, struct fuse_file_info *fi)
{
    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;

    (void)fi;
    return 0;
}

static int prestige_pull_rabbit(const char *path, char *buf, size_t size,
                                 off_t offset, struct fuse_file_info *fi)
{
    (void)fi;
    g_theater->performance.pull_calls++;

    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;
    if (S_ISDIR(prop->mode))
        return -EISDIR;

    if (offset >= (off_t)prop->size)
        return 0;

    size_t available = prop->size - offset;
    size_t to_read = size < available ? size : available;

    if (prop->secret_content)
        memcpy(buf, prop->secret_content + offset, to_read);
    else
        memset(buf, 0, to_read);

    prop->atime = time(NULL);
    g_theater->performance.total_read += to_read;

    return to_read;
}

static int prestige_plant_evidence(const char *path, const char *buf,
                                    size_t size, off_t offset,
                                    struct fuse_file_info *fi)
{
    (void)fi;
    g_theater->performance.plant_calls++;

    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;
    if (S_ISDIR(prop->mode))
        return -EISDIR;

    size_t new_size = offset + size;
    if (new_size > prop->size) {
        char *new_content = realloc(prop->secret_content, new_size);
        if (!new_content)
            return -ENOMEM;

        /* Zero fill gap */
        if (offset > (off_t)prop->size)
            memset(new_content + prop->size, 0, offset - prop->size);

        prop->secret_content = new_content;
        prop->size = new_size;
    }

    memcpy(prop->secret_content + offset, buf, size);
    prop->mtime = time(NULL);
    g_theater->performance.total_written += size;

    return size;
}

static int prestige_materialize(const char *path, mode_t mode,
                                 struct fuse_file_info *fi)
{
    (void)fi;

    prestige_prop_t *prop = backstage_craft_prop(g_theater, path,
                                                  S_IFREG | (mode & 0777));
    return prop ? 0 : -ENOMEM;
}

static int prestige_vanish(const char *path)
{
    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;
    if (S_ISDIR(prop->mode))
        return -EISDIR;

    return backstage_remove_prop(g_theater, path);
}

static int prestige_erect_stage(const char *path, mode_t mode)
{
    prestige_prop_t *prop = backstage_craft_prop(g_theater, path,
                                                  S_IFDIR | (mode & 0777));
    return prop ? 0 : -ENOMEM;
}

static int prestige_collapse_stage(const char *path)
{
    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;
    if (!S_ISDIR(prop->mode))
        return -ENOTDIR;
    if (prop->children)
        return -ENOTEMPTY;

    return backstage_remove_prop(g_theater, path);
}

static int prestige_transported_man(const char *from, const char *to,
                                     unsigned int flags)
{
    (void)flags;

    prestige_prop_t *src = backstage_find_prop(g_theater, from);
    if (!src)
        return -ENOENT;

    /* Check if destination exists */
    prestige_prop_t *dst = backstage_find_prop(g_theater, to);
    if (dst) {
        /* Remove destination if it exists */
        if (S_ISDIR(dst->mode) && dst->children)
            return -ENOTEMPTY;
        backstage_remove_prop(g_theater, to);
    }

    /* Get destination parent */
    char *dst_parent_path = backstage_parent_path(to);
    prestige_prop_t *dst_parent = backstage_find_prop(g_theater, dst_parent_path);
    free(dst_parent_path);

    if (!dst_parent)
        return -ENOENT;

    /* Remove from source parent */
    prestige_prop_t *src_parent = src->parent;
    prestige_prop_t **pp = &src_parent->children;
    while (*pp && *pp != src)
        pp = &(*pp)->next;
    if (*pp)
        *pp = src->next;

    /* Update source nlink in parent if directory */
    if (S_ISDIR(src->mode))
        src_parent->nlink--;

    /* Update name and parent */
    strncpy(src->name, backstage_prop_name(to), sizeof(src->name) - 1);
    src->parent = dst_parent;

    /* Add to destination parent */
    src->next = dst_parent->children;
    dst_parent->children = src;

    if (S_ISDIR(src->mode))
        dst_parent->nlink++;

    src->ctime = time(NULL);

    return 0;
}

static int prestige_guillotine(const char *path, off_t size,
                                struct fuse_file_info *fi)
{
    (void)fi;

    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;
    if (S_ISDIR(prop->mode))
        return -EISDIR;

    if ((size_t)size != prop->size) {
        char *new_content = realloc(prop->secret_content, size);
        if (size > 0 && !new_content)
            return -ENOMEM;

        /* Zero fill if growing */
        if ((size_t)size > prop->size && new_content)
            memset(new_content + prop->size, 0, size - prop->size);

        prop->secret_content = new_content;
        prop->size = size;
    }

    prop->mtime = time(NULL);
    return 0;
}

static int prestige_costume_change(const char *path, mode_t mode,
                                    struct fuse_file_info *fi)
{
    (void)fi;

    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;

    prop->mode = (prop->mode & S_IFMT) | (mode & 07777);
    prop->ctime = time(NULL);

    return 0;
}

static int prestige_identity_swap(const char *path, uid_t uid, gid_t gid,
                                   struct fuse_file_info *fi)
{
    (void)fi;

    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;

    prop->uid = uid;
    prop->gid = gid;
    prop->ctime = time(NULL);

    return 0;
}

static int prestige_create_double(const char *target, const char *linkpath)
{
    prestige_prop_t *prop = backstage_craft_prop(g_theater, linkpath,
                                                  S_IFLNK | 0777);
    if (!prop)
        return -ENOMEM;

    prop->mirror_target = strdup(target);
    if (!prop->mirror_target) {
        backstage_remove_prop(g_theater, linkpath);
        return -ENOMEM;
    }

    prop->size = strlen(target);
    return 0;
}

static int prestige_check_mirror(const char *path, char *buf, size_t size)
{
    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;
    if (!S_ISLNK(prop->mode))
        return -EINVAL;
    if (!prop->mirror_target)
        return -EIO;

    size_t len = strlen(prop->mirror_target);
    if (len >= size)
        len = size - 1;

    memcpy(buf, prop->mirror_target, len);
    buf[len] = '\0';

    return len;
}

static void *prestige_open_curtains(struct fuse_conn_info *conn,
                                     struct fuse_config *cfg)
{
    (void)conn;
    cfg->use_ino = 1;

    fprintf(stderr, "[Prestige FS] The curtains are open. The show begins.\n");
    fprintf(stderr, "[Prestige FS] Are you watching closely?\n");

    return g_theater;
}

static void prestige_final_bow(void *private_data)
{
    prestige_theater_t *theater = (prestige_theater_t *)private_data;

    fprintf(stderr, "\n[Prestige FS] The final bow. *applause*\n");
    fprintf(stderr, "[Prestige FS] Stats: %lu examines, %lu reveals, "
                    "%lu reads (%lu bytes), %lu writes (%lu bytes)\n",
            theater->performance.examine_calls,
            theater->performance.reveal_calls,
            theater->performance.pull_calls,
            theater->performance.total_read,
            theater->performance.plant_calls,
            theater->performance.total_written);

    free_prop_recursive(theater->stage);
    free(theater);
}

static const struct fuse_operations prestige_playbook = {
    .getattr    = prestige_examine_prop,
    .readdir    = prestige_reveal_cabinet,
    .open       = prestige_access_vault,
    .read       = prestige_pull_rabbit,
    .write      = prestige_plant_evidence,
    .create     = prestige_materialize,
    .unlink     = prestige_vanish,
    .mkdir      = prestige_erect_stage,
    .rmdir      = prestige_collapse_stage,
    .rename     = prestige_transported_man,
    .truncate   = prestige_guillotine,
    .chmod      = prestige_costume_change,
    .chown      = prestige_identity_swap,
    .symlink    = prestige_create_double,
    .readlink   = prestige_check_mirror,
    .init       = prestige_open_curtains,
    .destroy    = prestige_final_bow,
};

int main(int argc, char *argv[])
{
    /* Create the theater */
    g_theater = calloc(1, sizeof(prestige_theater_t));
    if (!g_theater) {
        fprintf(stderr, "Failed to allocate theater\n");
        return 1;
    }

    /* Create root node (the main stage) */
    g_theater->next_identity = 1;
    g_theater->stage = create_prop_node("", S_IFDIR | 0755, NULL);
    if (!g_theater->stage) {
        free(g_theater);
        fprintf(stderr, "Failed to create stage\n");
        return 1;
    }

    /* The show must go on! */
    return fuse_main(argc, argv, &prestige_playbook, g_theater);
}
```

### 4.4 Solutions alternatives acceptées

```c
/* Alternative 1 : Utilisation de hash table pour recherche O(1) */
/* Acceptable si toutes les opérations sont correctement implémentées */

/* Alternative 2 : Structure tree différente */
/* Acceptable tant que les opérations FUSE fonctionnent */
```

### 4.5 Solutions refusées (avec explications)

```c
/* REFUSÉ : Utilisation de vrais appels système */
static int bad_read(const char *path, char *buf, size_t size, off_t off,
                    struct fuse_file_info *fi)
{
    /* NON ! Tu dois implémenter, pas appeler les vraies fonctions */
    int fd = open(path, O_RDONLY);
    return read(fd, buf, size);
}
/* Pourquoi : Ça défait tout l'intérêt de l'exercice */

/* REFUSÉ : Pas de gestion d'erreurs */
static int bad_getattr(const char *path, struct stat *st,
                       struct fuse_file_info *fi)
{
    prestige_prop_t *p = backstage_find_prop(g_theater, path);
    st->st_mode = p->mode;  /* CRASH si p est NULL ! */
    return 0;
}
/* Pourquoi : Segfault sur chemin inexistant */
```

### 4.9 spec.json (ENGINE v22.1 — FORMAT STRICT)

```json
{
  "name": "prestige_filesystem",
  "language": "c",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse FUSE",
  "tags": ["fuse", "filesystem", "userspace", "phase2", "advanced"],
  "passing_score": 70,

  "function": {
    "name": "prestige_fs",
    "prototype": "Full FUSE filesystem implementation",
    "return_type": "int (fuse_main return)",
    "parameters": []
  },

  "driver": {
    "type": "fuse_filesystem",
    "mount_required": true,
    "reference_file": "prestige_fs.c",

    "edge_cases": [
      {
        "name": "getattr_root",
        "operation": "stat",
        "args": ["/"],
        "expected": {"mode": "S_IFDIR|0755"},
        "is_trap": false
      },
      {
        "name": "getattr_nonexist",
        "operation": "stat",
        "args": ["/nonexistent"],
        "expected": -2,
        "is_trap": true,
        "trap_explanation": "Must return -ENOENT for missing paths"
      },
      {
        "name": "create_write_read",
        "operation": "sequence",
        "args": ["create /test.txt", "write hello", "read"],
        "expected": "hello",
        "is_trap": false
      },
      {
        "name": "mkdir_rmdir_empty",
        "operation": "sequence",
        "args": ["mkdir /testdir", "rmdir /testdir"],
        "expected": 0,
        "is_trap": false
      },
      {
        "name": "rmdir_nonempty",
        "operation": "sequence",
        "args": ["mkdir /dir", "create /dir/file", "rmdir /dir"],
        "expected": -39,
        "is_trap": true,
        "trap_explanation": "Must return -ENOTEMPTY"
      },
      {
        "name": "rename_basic",
        "operation": "sequence",
        "args": ["create /a.txt", "rename /a.txt /b.txt", "stat /b.txt"],
        "expected": 0,
        "is_trap": false
      },
      {
        "name": "symlink_readlink",
        "operation": "sequence",
        "args": ["create /target", "symlink target /link", "readlink /link"],
        "expected": "target",
        "is_trap": false
      },
      {
        "name": "truncate_shrink",
        "operation": "sequence",
        "args": ["create /f", "write 1234567890", "truncate 5", "stat"],
        "expected": {"size": 5},
        "is_trap": false
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "fuse_operation_sequence",
          "param_index": 0,
          "params": {
            "max_ops": 50,
            "operations": ["create", "write", "read", "unlink", "mkdir", "rmdir", "rename"]
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["fuse_main", "fuse_get_context", "malloc", "free", "realloc", "calloc", "memcpy", "memset", "memmove", "strlen", "strncpy", "strcmp", "strncmp", "strdup", "strchr", "strrchr", "time", "localtime", "printf", "fprintf", "snprintf"],
    "forbidden_functions": ["open", "read", "write", "stat", "lstat", "fstat", "opendir", "readdir", "closedir", "link", "unlink", "mkdir", "rmdir", "rename", "truncate", "chmod", "chown", "symlink", "readlink"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  },

  "compilation": {
    "flags": "-Wall -Wextra -std=c17 $(pkg-config --cflags fuse3)",
    "libs": "$(pkg-config --libs fuse3)"
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```c
/* =============================================================================
 * Mutant A (Boundary) : Off-by-one dans read
 * =============================================================================
 */
static int mutant_a_pull_rabbit(const char *path, char *buf, size_t size,
                                 off_t offset, struct fuse_file_info *fi)
{
    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop) return -ENOENT;

    /* BUG: >= au lieu de > */
    if (offset >= (off_t)prop->size)
        return 0;

    size_t available = prop->size - offset;
    /* BUG: Lit un byte de trop */
    size_t to_read = size <= available ? size : available + 1;

    memcpy(buf, prop->secret_content + offset, to_read);
    return to_read;
}
/* Pourquoi c'est faux : Buffer overflow possible */
/* Ce qui était pensé : "Je vais lire un peu plus pour être sûr" */


/* =============================================================================
 * Mutant B (Safety) : Pas de vérification NULL dans find
 * =============================================================================
 */
static int mutant_b_examine_prop(const char *path, struct stat *stbuf,
                                  struct fuse_file_info *fi)
{
    /* BUG: Pas de vérification du retour de find */
    prestige_prop_t *prop = backstage_find_prop(g_theater, path);

    /* CRASH si prop est NULL ! */
    memset(stbuf, 0, sizeof(struct stat));
    stbuf->st_mode = prop->mode;
    stbuf->st_size = prop->size;

    return 0;
}
/* Pourquoi c'est faux : Segfault sur chemin inexistant */
/* Ce qui était pensé : "Le chemin existe sûrement" */


/* =============================================================================
 * Mutant C (Resource) : Fuite mémoire dans create
 * =============================================================================
 */
static int mutant_c_materialize(const char *path, mode_t mode,
                                 struct fuse_file_info *fi)
{
    /* Crée le node mais ne le libère jamais en cas d'erreur */
    prestige_prop_t *prop = malloc(sizeof(prestige_prop_t));
    memset(prop, 0, sizeof(*prop));

    /* BUG: Si l'ajout échoue, prop n'est jamais libéré */
    char *parent_path = backstage_parent_path(path);
    prestige_prop_t *parent = backstage_find_prop(g_theater, parent_path);

    if (!parent) {
        free(parent_path);
        /* BUG: prop n'est pas libéré ! */
        return -ENOENT;
    }

    /* ... reste du code ... */
    return 0;
}
/* Pourquoi c'est faux : Fuite mémoire à chaque création échouée */
/* Ce qui était pensé : "Le return nettoie tout" */


/* =============================================================================
 * Mutant D (Logic) : Mauvaise logique pour rmdir
 * =============================================================================
 */
static int mutant_d_collapse_stage(const char *path)
{
    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -ENOENT;

    /* BUG: Vérifie si c'est un fichier au lieu de dir */
    if (S_ISREG(prop->mode))  /* Devrait être !S_ISDIR */
        return -ENOTDIR;

    /* BUG: Supprime même si non-vide */
    /* Manque: if (prop->children) return -ENOTEMPTY; */

    return backstage_remove_prop(g_theater, path);
}
/* Pourquoi c'est faux : Permet de supprimer un répertoire non-vide */
/* Ce qui était pensé : "rmdir devrait toujours fonctionner" */


/* =============================================================================
 * Mutant E (Return) : Mauvais code de retour
 * =============================================================================
 */
static int mutant_e_access_vault(const char *path, struct fuse_file_info *fi)
{
    prestige_prop_t *prop = backstage_find_prop(g_theater, path);
    if (!prop)
        return -1;  /* BUG: Devrait être -ENOENT */

    return 1;  /* BUG: Devrait être 0 pour succès */
}
/* Pourquoi c'est faux : Codes d'erreur FUSE non standards */
/* Ce qui était pensé : "1 = true = succès" */


/* =============================================================================
 * Mutant F (Concurrency) : Race condition dans rename
 * =============================================================================
 */
static int mutant_f_transported_man(const char *from, const char *to,
                                     unsigned int flags)
{
    /* BUG: Vérifie l'existence, puis fait autre chose, puis utilise */
    prestige_prop_t *src = backstage_find_prop(g_theater, from);
    if (!src)
        return -ENOENT;

    /* Entre temps, un autre thread pourrait supprimer src */

    /* BUG: Pas de verrouillage */
    prestige_prop_t *src_parent = src->parent;  /* DANGER: src pourrait être invalide */

    /* ... manipulation des pointeurs ... */
    return 0;
}
/* Pourquoi c'est faux : Race condition dans environnement multithread */
/* Ce qui était pensé : "FUSE est single-threaded" (faux par défaut) */
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CONCEPTS MAÎTRISÉS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. FUSE Architecture                                                       │
│     • Communication kernel ↔ userspace via /dev/fuse                        │
│     • Structure fuse_operations                                             │
│     • libfuse API (high-level vs low-level)                                 │
│                                                                             │
│  2. Implémentation Filesystem                                               │
│     • Métadonnées : stat, permissions, timestamps                           │
│     • Arborescence : directories, files, symlinks                           │
│     • Opérations : create, read, write, unlink, rename...                   │
│                                                                             │
│  3. Gestion Mémoire Complexe                                                │
│     • Structures récursives (arbre de fichiers)                             │
│     • Allocation/libération sans fuites                                     │
│     • Redimensionnement dynamique (fichiers qui grandissent)                │
│                                                                             │
│  4. Gestion d'Erreurs POSIX                                                 │
│     • Codes errno appropriés                                                │
│     • Comportement attendu par le VFS                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 LDA — Traduction littérale en français (MAJUSCULES)

```
FONCTION prestige_examine_prop QUI RETOURNE UN ENTIER ET PREND EN PARAMÈTRES
path QUI EST UN POINTEUR VERS UNE CHAÎNE CONSTANTE ET stbuf QUI EST UN POINTEUR
VERS UNE STRUCTURE stat ET fi QUI EST UN POINTEUR VERS fuse_file_info
DÉBUT FONCTION
    INCRÉMENTER LE COMPTEUR D'EXAMENS DE 1

    AFFECTER CHERCHER LE PROP CORRESPONDANT À path DANS LE THÉÂTRE À prop

    SI prop EST ÉGAL À NUL ALORS
        RETOURNER MOINS ENOENT
    FIN SI

    REMPLIR DE ZÉROS LA STRUCTURE stbuf SUR SA TAILLE
    AFFECTER LE NUMÉRO D'IDENTITÉ DE prop AU CHAMP st_ino DE stbuf
    AFFECTER LE MODE DE prop AU CHAMP st_mode DE stbuf
    AFFECTER LE NOMBRE DE LIENS DE prop AU CHAMP st_nlink DE stbuf
    AFFECTER L'UID DE prop AU CHAMP st_uid DE stbuf
    AFFECTER LE GID DE prop AU CHAMP st_gid DE stbuf
    AFFECTER LA TAILLE DE prop AU CHAMP st_size DE stbuf
    AFFECTER LES TIMESTAMPS DE prop AUX CHAMPS CORRESPONDANTS DE stbuf

    RETOURNER 0
FIN FONCTION
```

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : FUSE Request Handling
---
1. USER fait un appel système (ex: cat /mnt/prestige/file.txt)

2. KERNEL (VFS) reçoit la requête :
   a. Identifie le filesystem (prestige_fs sur /mnt/prestige)
   b. Envoie la requête vers /dev/fuse

3. LIBFUSE reçoit via /dev/fuse :
   a. Décode la requête (opcode: READ, path: "/file.txt")
   b. APPELLE notre callback (prestige_pull_rabbit)

4. NOTRE CODE s'exécute :
   a. Trouve le prop correspondant dans notre arbre
   b. Copie les données dans le buffer
   c. RETOURNE le nombre d'octets lus

5. LIBFUSE encode la réponse et l'envoie à /dev/fuse

6. KERNEL (VFS) reçoit la réponse :
   a. Retourne les données à l'application

7. USER reçoit le contenu du fichier

RÉSULTAT : L'illusion est parfaite - l'utilisateur pense avoir lu un vrai fichier
```

### 5.2.3.1 Logique de Garde (Fail Fast)

```
FONCTION : prestige_transported_man (rename)
---
INIT result = -EINVAL

1. VÉRIFIER source :
   |
   |-- SI source n'existe pas :
   |     RETOURNER -ENOENT
   |
   |-- SI source est la racine :
   |     RETOURNER -EINVAL

2. VÉRIFIER destination :
   |
   |-- SI destination existe ET est un répertoire non-vide :
   |     RETOURNER -ENOTEMPTY
   |
   |-- SI destination existe :
   |     SUPPRIMER la destination

3. VÉRIFIER parent destination :
   |
   |-- SI parent destination n'existe pas :
   |     RETOURNER -ENOENT
   |
   |-- SI parent destination n'est pas un répertoire :
   |     RETOURNER -ENOTDIR

4. EXÉCUTER le déplacement :
   |-- RETIRER source de son parent
   |-- METTRE À JOUR le nom de source
   |-- AJOUTER source au nouveau parent
   |-- METTRE À JOUR les timestamps

5. RETOURNER 0 (succès)
```

### 5.3 Visualisation ASCII

```
                    L'ARCHITECTURE FUSE — THE PRESTIGE
═══════════════════════════════════════════════════════════════════════════════

                              USER SPACE
    ┌─────────────────────────────────────────────────────────────────────┐
    │                                                                     │
    │   APPLICATION                        PRESTIGE_FS                    │
    │   ┌─────────────┐                   ┌─────────────────────────┐    │
    │   │   bash      │                   │  The Theater            │    │
    │   │   ────────  │                   │  ┌─────────────────┐    │    │
    │   │   $ cat /mnt│                   │  │    stage (/)    │    │    │
    │   │   /magic/   │                   │  │    ┌───────┐    │    │    │
    │   │   secret.txt│                   │  │    │ props │    │    │    │
    │   └──────┬──────┘                   │  │    └───────┘    │    │    │
    │          │                          │  └────────┬────────┘    │    │
    │          │                          │           │              │    │
    │          │     LIBFUSE             │  ┌────────▼────────┐    │    │
    │          │     ┌─────────────┐     │  │ fuse_operations │    │    │
    │          │     │ Request     │     │  │ .getattr=exam   │    │    │
    │          └────►│ Handler     │────►│  │ .read=pull      │    │    │
    │                │             │     │  │ .write=plant    │    │    │
    │                │             │◄────│  │ .rename=transp  │    │    │
    │                └──────┬──────┘     │  └─────────────────┘    │    │
    │                       │            │                          │    │
    └───────────────────────┼────────────┴──────────────────────────┘    │
                            │                                             │
════════════════════════════╪═════════════════════════════════════════════╪═
                            │      /dev/fuse                              │
════════════════════════════╪═════════════════════════════════════════════╪═
                            │                                             │
                       KERNEL SPACE                                       │
    ┌───────────────────────┼─────────────────────────────────────────────┘
    │                       │
    │   VFS (Virtual Filesystem Switch)
    │   ┌───────────────────▼───────────────────┐
    │   │                                       │
    │   │   "I see a filesystem at /mnt/magic"  │
    │   │   "Let me ask it about secret.txt"    │
    │   │                                       │
    │   │   Dispatch to FUSE driver             │
    │   │                                       │
    │   └───────────────────────────────────────┘
    │
    └─────────────────────────────────────────────────────────────────────


    THE THREE ACTS:
    ═══════════════

    1. THE PLEDGE          2. THE TURN           3. THE PRESTIGE
    ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
    │ fuse_main()  │      │ Callbacks    │      │ Response     │
    │              │      │              │      │              │
    │ "Here's an   │─────►│ "Transform   │─────►│ "Magic! The  │
    │  ordinary    │      │  the request │      │  impossible  │
    │  program"    │      │  into data"  │      │  is real"    │
    └──────────────┘      └──────────────┘      └──────────────┘
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **Codes errno** | Retourner -1 au lieu de -ENOENT | Toujours utiliser les macros errno négatives |
| **Buffer overflow** | Lire/écrire au-delà des limites | Vérifier offset + size vs taille réelle |
| **Fuites mémoire** | Ne pas libérer lors de destroy | Parcours récursif de tout l'arbre |
| **Race conditions** | FUSE est multithread par défaut | Utiliser mutex ou -s (single-thread) |
| **Path parsing** | Oublier le "/" initial | Toujours gérer "/" comme cas spécial |
| **nlink count** | Mauvais comptage des hardlinks | Dir = 2 + subdirs, File = 1 |

### 5.5 Cours Complet

#### 5.5.1 Qu'est-ce que FUSE ?

FUSE (Filesystem in Userspace) est un framework qui permet de créer des systèmes de fichiers
sans modifier le kernel. Avant FUSE, créer un filesystem nécessitait d'écrire un module kernel,
une tâche complexe et dangereuse (un bug = kernel panic).

```
AVANT FUSE (2001)               AVEC FUSE
──────────────────              ─────────
Créer un FS = Module kernel     Créer un FS = Programme normal
Temps de dev : mois             Temps de dev : heures/jours
Risque : kernel panic           Risque : crash du programme
Debug : printk, reboot          Debug : gdb, printf
```

#### 5.5.2 L'architecture en détail

```c
/* 1. Le kernel voit ton programme comme un vrai filesystem */
/* 2. Quand quelqu'un fait cat /mnt/ton_fs/fichier.txt : */

Utilisateur               Kernel                    Ton programme
    |                        |                           |
    | cat file.txt           |                           |
    |----------------------->|                           |
    |                        | "C'est un FUSE mount"     |
    |                        |-------------------------->|
    |                        |                           | prestige_pull_rabbit()
    |                        |                           | return data
    |                        |<--------------------------|
    |<-----------------------|                           |
    | Affiche le contenu     |                           |
```

#### 5.5.3 Structure fuse_operations

La clé de FUSE est cette structure qui définit tous les callbacks :

```c
struct fuse_operations {
    /* Métadonnées */
    int (*getattr)(const char *, struct stat *, struct fuse_file_info *);

    /* Répertoires */
    int (*readdir)(const char *, void *, fuse_fill_dir_t, off_t,
                   struct fuse_file_info *, enum fuse_readdir_flags);
    int (*mkdir)(const char *, mode_t);
    int (*rmdir)(const char *);

    /* Fichiers */
    int (*create)(const char *, mode_t, struct fuse_file_info *);
    int (*open)(const char *, struct fuse_file_info *);
    int (*read)(const char *, char *, size_t, off_t, struct fuse_file_info *);
    int (*write)(const char *, const char *, size_t, off_t, struct fuse_file_info *);
    int (*unlink)(const char *);
    int (*truncate)(const char *, off_t, struct fuse_file_info *);

    /* Liens */
    int (*symlink)(const char *, const char *);
    int (*readlink)(const char *, char *, size_t);

    /* Permissions */
    int (*chmod)(const char *, mode_t, struct fuse_file_info *);
    int (*chown)(const char *, uid_t, gid_t, struct fuse_file_info *);

    /* Divers */
    int (*rename)(const char *, const char *, unsigned int);
    void *(*init)(struct fuse_conn_info *, struct fuse_config *);
    void (*destroy)(void *);

    /* ... et beaucoup d'autres */
};
```

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais problématique)                     │
├─────────────────────────────────────────────────────────────────┤
│ return -1;  /* Erreur générique */                              │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ return -ENOENT;  /* Fichier non trouvé */                       │
│ return -EACCES;  /* Permission refusée */                       │
│ return -ENOMEM;  /* Plus de mémoire */                          │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Le VFS attend des codes errno standards                       │
│ • -1 est ambigü (quelle erreur ?)                               │
│ • Les applications utilisent ces codes pour leur logique        │
│ • strerror() ne fonctionne qu'avec les vrais codes errno        │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario : `echo "hello" > /mnt/prestige/test.txt`**

```
┌───────┬─────────────────────────────────────────────┬──────────────────────────┐
│ Étape │ Action                                      │ Résultat                 │
├───────┼─────────────────────────────────────────────┼──────────────────────────┤
│   1   │ Shell demande OPEN("/test.txt", CREATE)     │ prestige_materialize()   │
│       │                                             │ → Crée le node           │
├───────┼─────────────────────────────────────────────┼──────────────────────────┤
│   2   │ prestige_materialize("/test.txt", 0644)     │ Node créé, mode=0644     │
│       │ - Trouve parent "/"                         │ size=0                   │
│       │ - Crée node "test.txt"                      │                          │
│       │ - L'ajoute aux enfants de "/"               │                          │
├───────┼─────────────────────────────────────────────┼──────────────────────────┤
│   3   │ Shell demande WRITE("hello\n", 6 bytes)     │ prestige_plant_evidence()│
├───────┼─────────────────────────────────────────────┼──────────────────────────┤
│   4   │ prestige_plant_evidence("/test.txt", ...)   │ secret_content = "hello" │
│       │ - Trouve le node                            │ size = 6                 │
│       │ - realloc(secret_content, 6)                │ mtime = now              │
│       │ - memcpy("hello\n")                         │                          │
│       │ - Retourne 6                                │                          │
├───────┼─────────────────────────────────────────────┼──────────────────────────┤
│   5   │ Shell demande CLOSE                         │ (pas de callback requis) │
├───────┼─────────────────────────────────────────────┼──────────────────────────┤
│   6   │ Fichier créé avec succès !                  │ Illusion parfaite ✓      │
└───────┴─────────────────────────────────────────────┴──────────────────────────┘
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🎩 MEME : "The Prestige" — Les trois actes de FUSE

```
"Every great FUSE filesystem consists of three parts or acts."

┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│   THE PLEDGE: fuse_main()                                       │
│   "I'm going to show you something ordinary - a program"        │
│                                                                 │
│   THE TURN: fuse_operations callbacks                           │
│   "I'm going to make the kernel believe it's a real filesystem" │
│                                                                 │
│   THE PRESTIGE: The filesystem works!                           │
│   "You want to find the secret, but you won't..."               │
│   "Because you don't really want to know - you want to be       │
│    fooled"                                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

#### 🔮 MEME : "The Transported Man" — rename()

```
Le tour de magie signature du film : faire disparaître quelqu'un
d'un endroit pour le faire réapparaître ailleurs instantanément.

C'est EXACTEMENT ce que fait rename() !

rename("/src/file.txt", "/dst/file.txt"):
1. Le fichier DISPARAÎT de /src
2. Le fichier APPARAÎT dans /dst
3. Tout ça est ATOMIQUE (instantané)

"Are you watching closely?"
```

#### ⚡ MEME : "The Machine" — La duplication parfaite

```
Dans le film, la machine de Tesla duplique tout ce qu'on y met.

Ton filesystem FUSE doit faire pareil avec les données :
- L'utilisateur écrit "hello"
- Ta structure en mémoire DUPLIQUE parfaitement ces données
- Quand on lit, on récupère l'EXACT même contenu

Si un seul byte diffère → L'illusion est brisée !

"The machine doesn't work. Or it does, and the working version
 is the one that's not standing here right now."
```

### 5.9 Applications pratiques

| Projet | Description | FUSE Operations utilisées |
|--------|-------------|---------------------------|
| **sshfs** | Monte un serveur distant via SSH | Toutes (proxy vers SSH) |
| **s3fs** | Monte un bucket Amazon S3 | read, write, getattr, readdir |
| **encfs** | Chiffrement transparent | Toutes + crypto |
| **archivemount** | Monte des archives comme répertoires | read, readdir, getattr |
| **gitfs** | Monte l'historique Git | readdir, read, symlink |
| **mp3fs** | Conversion à la volée FLAC→MP3 | read (avec transcodage) |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              LES 10 PIÈGES FUSE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. ❌ Retourner -1 au lieu de -ENOENT, -EACCES, etc.                       │
│  2. ❌ Oublier de gérer le cas path="/"                                     │
│  3. ❌ Buffer overflow dans read/write (offset + size > file_size)          │
│  4. ❌ Ne pas libérer la mémoire dans destroy()                             │
│  5. ❌ Oublier de mettre à jour mtime/ctime/atime                           │
│  6. ❌ Mauvais nlink (dir doit être ≥ 2)                                    │
│  7. ❌ rmdir sur répertoire non-vide (doit retourner -ENOTEMPTY)            │
│  8. ❌ Race conditions (FUSE est multithread par défaut)                    │
│  9. ❌ Symlink : stocker le chemin absolu au lieu de relatif                │
│ 10. ❌ Oublier "." et ".." dans readdir                                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📝 SECTION 7 : QCM

### Question 1
**Que signifie FUSE ?**
- A) Fast Unified System Extension
- B) Filesystem in Userspace
- C) File Utility System Engine
- D) Federated Unix Storage Environment
- E) Fast User Storage Extension
- F) File Userspace System Extension
- G) Filesystem Unified Service Engine
- H) Fast Utility Storage Engine
- I) File Unix System Extension
- J) Filesystem Utility Service Extension

**Réponse : B**

### Question 2
**Quelle structure contient tous les callbacks FUSE ?**
- A) fuse_callbacks
- B) fuse_handlers
- C) fuse_operations
- D) fuse_vtable
- E) fuse_methods
- F) fuse_interface
- G) fuse_functions
- H) fuse_ops
- I) fuse_dispatch
- J) fuse_hooks

**Réponse : C**

### Question 3
**Quel est le code de retour correct pour "fichier non trouvé" ?**
- A) return -1;
- B) return 0;
- C) return -ENOENT;
- D) return -ENOTFOUND;
- E) return -MISSING;
- F) return -ENONE;
- G) return -ENOFILE;
- H) return NULL;
- I) return -ERROR;
- J) return -EINVAL;

**Réponse : C**

### Question 4
**Quelle opération FUSE est appelée par ls -la ?**
- A) list()
- B) readdir() + getattr()
- C) scan()
- D) enumerate()
- E) readdir() seulement
- F) getattr() seulement
- G) listdir()
- H) stat()
- I) browse()
- J) dir()

**Réponse : B**

### Question 5
**Que doit retourner nlink pour un répertoire vide ?**
- A) 0
- B) 1
- C) 2
- D) 3
- E) Nombre de fichiers
- F) -1
- G) Le uid
- H) La taille
- I) L'inode
- J) 4096

**Réponse : C** (pour "." et "..")

---

## 📊 SECTION 8 : RÉCAPITULATIF

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         EXERCICE 2.3.10 — RÉSUMÉ                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  THÈME : The Prestige — L'Art de l'Illusion Parfaite                       │
│                                                                             │
│  CONCEPTS CLÉS :                                                            │
│  • FUSE = Filesystem in Userspace                                           │
│  • fuse_operations = La table des callbacks                                 │
│  • 15+ opérations à implémenter                                             │
│  • Gestion mémoire de l'arborescence                                        │
│  • Codes errno POSIX                                                        │
│                                                                             │
│  LES TROIS ACTES :                                                          │
│  1. The Pledge → fuse_main() initialise                                     │
│  2. The Turn → Callbacks transforment les requêtes                          │
│  3. The Prestige → L'illusion est parfaite                                  │
│                                                                             │
│  DIFFICULTÉ : ★★★★★★★★☆☆ (8/10)                                            │
│  DURÉE : 10h                                                                │
│  XP : 500 base × 2 bonus                                                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.10-prestige-filesystem",
    "generated_at": "2026-01-12 16:00:00",

    "metadata": {
      "exercise_id": "2.3.10",
      "exercise_name": "prestige_filesystem",
      "module": "2.3",
      "module_name": "File Systems",
      "concept": "FUSE",
      "concept_name": "Filesystem in Userspace",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthèse FUSE",
      "phase": 2,
      "difficulty": 8,
      "difficulty_stars": "★★★★★★★★☆☆",
      "language": "c",
      "duration_minutes": 600,
      "xp_base": 500,
      "xp_bonus_multiplier": 2,
      "bonus_tier": "STANDARD",
      "bonus_icon": "⚡",
      "complexity_time": "T7 O(n)",
      "complexity_space": "S6 O(n)",
      "prerequisites": ["2.3.0-2.3.9", "memory_management", "callbacks"],
      "domains": ["FS", "Mem", "Struct"],
      "domains_bonus": ["Compression"],
      "tags": ["fuse", "filesystem", "userspace", "callbacks", "advanced"],
      "meme_reference": "The Prestige (2006)"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/prestige_fs.c": "/* Section 4.3 */",
      "references/prestige_fs.h": "/* Section 1.3 */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 */",
      "mutants/mutant_e_return.c": "/* Section 4.10 */",
      "mutants/mutant_f_concurrency.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/prestige_fs.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c",
        "mutants/mutant_f_concurrency.c"
      ]
    },

    "commands": {
      "validate_spec": "python3 hackbrain_engine_v22.py --validate-spec spec.json",
      "test_reference": "make && ./prestige_test_runner",
      "test_mutants": "python3 hackbrain_mutation_tester.py --fuse"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "The secret impresses no one. The trick you use it for is everything."*
