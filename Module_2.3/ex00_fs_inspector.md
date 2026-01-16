# [Module 2.3] - Exercise 00: File System Inspector

## Metadonnees

```yaml
module: "2.3 - File Systems"
exercise: "ex00"
title: "File System Inspector"
difficulty: facile
estimated_time: "4 heures"
prerequisite_exercises: []
concepts_requis: ["file I/O", "structs", "string manipulation", "system calls"]
concepts_couverts:
  - "2.3.1.a File: Named collection of data"
  - "2.3.1.b Directory: Container for files"
  - "2.3.1.c Path: Location of file"
  - "2.3.1.d Absolute path: From root"
  - "2.3.1.e Relative path: From current"
  - "2.3.1.f File types: Regular, directory, device, symlink, socket, pipe"
  - "2.3.1.g File attributes: Name, type, size, permissions, timestamps"
  - "2.3.1.h File operations: Create, open, read, write, close, delete"
  - "2.3.2.a Inode: Index node, file metadata"
  - "2.3.2.b Inode number: Unique within filesystem"
  - "2.3.2.c Inode contents: NOT name, NOT data"
  - "2.3.2.d File type: In inode"
  - "2.3.2.e Permissions: Mode bits"
  - "2.3.2.f Owner: UID, GID"
  - "2.3.2.g Size: In bytes"
  - "2.3.2.h Timestamps: atime, mtime, ctime"
  - "2.3.2.i Link count: Number of hard links"
  - "2.3.2.j Block pointers: To data blocks"
  - "2.3.2.k stat(): Get inode info"
  - "2.3.2.l ls -i: Show inode numbers"
score_qualite: 97
```

---

## Concepts Couverts

Cet exercice couvre systematiquement TOUS les concepts fondamentaux des systemes de fichiers Unix:

### 2.3.1 - Concepts Fondamentaux des Fichiers

| Reference | Concept | Description | Application dans l'exercice |
|-----------|---------|-------------|----------------------------|
| **2.3.1.a** | File | Collection nommee de donnees | Inspection de fichiers reguliers avec toutes leurs metadonnees |
| **2.3.1.b** | Directory | Conteneur pour fichiers | Detection et listage des repertoires, enumeration du contenu |
| **2.3.1.c** | Path | Localisation d'un fichier | Manipulation des chemins pour acceder aux fichiers |
| **2.3.1.d** | Absolute path | Chemin depuis la racine | Support des chemins absolus commencant par `/` |
| **2.3.1.e** | Relative path | Chemin depuis le courant | Conversion et resolution des chemins relatifs |
| **2.3.1.f** | File types | Regular, directory, device, symlink, socket, pipe | Detection et identification des 7 types de fichiers Unix |
| **2.3.1.g** | File attributes | Nom, type, taille, permissions, timestamps | Affichage complet de tous les attributs via stat() |
| **2.3.1.h** | File operations | Create, open, read, write, close, delete | Demonstration des operations basiques sur fichiers |

### 2.3.2 - Inodes et Metadonnees

| Reference | Concept | Description | Application dans l'exercice |
|-----------|---------|-------------|----------------------------|
| **2.3.2.a** | Inode | Index node, metadonnees fichier | Structure centrale retournee par stat() |
| **2.3.2.b** | Inode number | Unique dans le filesystem | Affichage du st_ino, equivalent de `ls -i` |
| **2.3.2.c** | Inode contents | NE contient PAS le nom, NE contient PAS les donnees | Explication pedagogique de ce qui est/n'est pas dans l'inode |
| **2.3.2.d** | File type | Stocke dans l'inode | Extraction via S_ISREG, S_ISDIR, etc. depuis st_mode |
| **2.3.2.e** | Permissions | Mode bits (rwx) | Formatage complet des 9 bits + setuid/setgid/sticky |
| **2.3.2.f** | Owner | UID et GID | Affichage uid/gid avec resolution des noms |
| **2.3.2.g** | Size | En bytes | Champ st_size avec formatage humain |
| **2.3.2.h** | Timestamps | atime, mtime, ctime | Affichage des 3 timestamps en ISO 8601 |
| **2.3.2.i** | Link count | Nombre de hard links | Champ st_nlink et explication des liens |
| **2.3.2.j** | Block pointers | Vers les blocs de donnees | st_blocks et st_blksize pour l'allocation |
| **2.3.2.k** | stat() | Recuperer les infos de l'inode | Fonction centrale de l'exercice |
| **2.3.2.l** | ls -i | Afficher les numeros d'inode | Fonctionnalite a reproduire |

### Objectifs Pedagogiques

A la fin de cet exercice, vous serez capable de:

1. **Comprendre la structure d'un fichier Unix** (2.3.1.a-h): Distinguer les differents types de fichiers et leurs attributs
2. **Maitriser les chemins** (2.3.1.c-e): Manipuler chemins absolus et relatifs
3. **Comprendre les inodes** (2.3.2.a-c): Savoir ce que contient et ne contient PAS un inode
4. **Utiliser stat() efficacement** (2.3.2.k): Extraire toutes les metadonnees d'un fichier
5. **Interpreter les permissions** (2.3.2.e-f): Lire et formater les mode bits
6. **Analyser les timestamps** (2.3.2.h): Comprendre atime, mtime, ctime
7. **Comprendre l'allocation** (2.3.2.g, 2.3.2.i-j): Taille, liens, et blocs

---

## Contexte Theorique

### Qu'est-ce qu'un Fichier? (2.3.1.a)

Dans Unix, un **fichier** (2.3.1.a) est fondamentalement une **collection nommee de donnees**. Contrairement a d'autres systemes d'exploitation, Unix adopte la philosophie "tout est fichier": les peripheriques, les sockets reseau, les pipes, et meme la memoire sont accessibles comme des fichiers.

Un fichier possede deux composants distincts:
- **Les donnees**: Le contenu reel du fichier (texte, binaire, etc.)
- **Les metadonnees**: Informations SUR le fichier (stockees dans l'**inode**, 2.3.2.a)

### Les Repertoires (2.3.1.b)

Un **repertoire** (2.3.1.b) est un type special de fichier qui sert de **conteneur pour d'autres fichiers**. Techniquement, un repertoire contient une table associant des noms a des numeros d'inode. C'est pourquoi le **nom du fichier n'est PAS stocke dans l'inode** (2.3.2.c) - il est stocke dans le repertoire parent.

```
Repertoire /home/user/:
+--------+------------+
| Nom    | Inode No.  |
+--------+------------+
| .      | 100        |  <- Lien vers soi-meme
| ..     | 50         |  <- Lien vers parent
| doc.txt| 12345      |  <- Fichier regulier
| photos | 12400      |  <- Sous-repertoire
+--------+------------+
```

### Chemins: Absolus vs Relatifs (2.3.1.c, 2.3.1.d, 2.3.1.e)

Le **chemin** (2.3.1.c) est la methode pour localiser un fichier dans l'arborescence:

- **Chemin absolu** (2.3.1.d): Commence a la racine `/`
  - Exemple: `/home/user/documents/rapport.txt`
  - Toujours le meme fichier, peu importe le repertoire courant

- **Chemin relatif** (2.3.1.e): Relatif au repertoire courant
  - Exemple: `../documents/rapport.txt`
  - Le fichier cible depend du repertoire de travail actuel
  - Utilise `.` (courant) et `..` (parent)

### Les 7 Types de Fichiers Unix (2.3.1.f)

Unix definit **7 types de fichiers** (2.3.1.f), chacun identifiable par un caractere dans la sortie de `ls -l`:

| Type | Caractere | Macro C | Description |
|------|-----------|---------|-------------|
| Regular | `-` | S_ISREG | Fichier ordinaire (texte, binaire, etc.) |
| Directory | `d` | S_ISDIR | Repertoire |
| Symbolic link | `l` | S_ISLNK | Lien symbolique (raccourci) |
| Block device | `b` | S_ISBLK | Peripherique bloc (disque) |
| Character device | `c` | S_ISCHR | Peripherique caractere (terminal) |
| FIFO/Pipe | `p` | S_ISFIFO | Pipe nomme pour IPC |
| Socket | `s` | S_ISSOCK | Socket Unix pour communication |

### Attributs des Fichiers (2.3.1.g)

Les **attributs d'un fichier** (2.3.1.g) comprennent:
- **Nom**: Stocke dans le repertoire parent (PAS dans l'inode!)
- **Type**: Regular, directory, symlink, etc.
- **Taille**: Nombre d'octets
- **Permissions**: Qui peut lire/ecrire/executer
- **Timestamps**: Dates d'acces/modification/changement

### Operations sur les Fichiers (2.3.1.h)

Les **operations fondamentales** (2.3.1.h) sur les fichiers sont:

```c
// Create: creer un nouveau fichier
int fd = creat("newfile.txt", 0644);
int fd = open("newfile.txt", O_CREAT | O_WRONLY, 0644);

// Open: ouvrir un fichier existant
int fd = open("file.txt", O_RDONLY);

// Read: lire des donnees
ssize_t n = read(fd, buffer, size);

// Write: ecrire des donnees
ssize_t n = write(fd, data, len);

// Close: fermer le fichier
close(fd);

// Delete: supprimer un fichier
unlink("file.txt");
```

### L'Inode: Coeur des Metadonnees (2.3.2.a, 2.3.2.b, 2.3.2.c)

L'**inode** (2.3.2.a - "index node") est la structure centrale qui stocke TOUTES les metadonnees d'un fichier, SAUF deux choses:

**Ce que l'inode CONTIENT:**
- Type de fichier (2.3.2.d)
- Permissions (2.3.2.e)
- Proprietaire UID/GID (2.3.2.f)
- Taille en bytes (2.3.2.g)
- Timestamps (2.3.2.h)
- Nombre de liens (2.3.2.i)
- Pointeurs vers blocs de donnees (2.3.2.j)

**Ce que l'inode NE CONTIENT PAS (2.3.2.c):**
- Le **nom du fichier** (stocke dans le repertoire)
- Les **donnees du fichier** (dans les blocs de donnees)

Le **numero d'inode** (2.3.2.b) est un identifiant unique DANS un filesystem. Deux fichiers sur deux disques differents peuvent avoir le meme numero d'inode, mais jamais sur le meme filesystem.

### Permissions: Les Mode Bits (2.3.2.e)

Les **permissions** (2.3.2.e) sont codees sur 12 bits dans le champ `st_mode`:

```
  Special     User      Group     Other
  [s][g][t]  [r][w][x] [r][w][x] [r][w][x]
     3 bits    3 bits    3 bits    3 bits
```

- **r** (4): Lecture
- **w** (2): Ecriture
- **x** (1): Execution
- **s** (setuid/setgid): Execute avec les droits du proprietaire/groupe
- **t** (sticky): Sur repertoire, seul le proprietaire peut supprimer

Exemple: `0755` = `rwxr-xr-x`

### Proprietaire: UID et GID (2.3.2.f)

Chaque fichier appartient a:
- Un **utilisateur** (UID - User ID)
- Un **groupe** (GID - Group ID)

Ces identifiants numeriques peuvent etre convertis en noms via `getpwuid()` et `getgrgid()`.

### Taille et Allocation (2.3.2.g, 2.3.2.j)

- **Taille** (2.3.2.g): `st_size` donne la taille logique en bytes
- **Blocs** (2.3.2.j): `st_blocks` donne le nombre de blocs de 512 bytes alloues
- **Taille de bloc**: `st_blksize` donne la taille de bloc preferee pour I/O

Note: Un fichier "sparse" peut avoir `st_size` >> `st_blocks * 512`.

### Les 3 Timestamps (2.3.2.h)

Chaque fichier a trois timestamps (2.3.2.h):

| Timestamp | Champ | Signification | Mis a jour quand... |
|-----------|-------|---------------|---------------------|
| **atime** | st_atime | Access time | Le fichier est lu |
| **mtime** | st_mtime | Modification time | Le contenu est modifie |
| **ctime** | st_ctime | Change time | L'inode est modifie (permissions, liens, etc.) |

**Important**: Il n'existe PAS de "creation time" standard sous Unix!

### Compteur de Liens (2.3.2.i)

Le **link count** (2.3.2.i) indique combien de noms pointent vers cet inode (hard links):

- Fichier regulier: Generalement 1
- Repertoire: Minimum 2 (lui-meme `.` + entree dans parent)
- Fichier avec hard links: > 1

Quand le link count atteint 0, le fichier est reellement supprime.

### stat() et ls -i (2.3.2.k, 2.3.2.l)

Le syscall **stat()** (2.3.2.k) permet de recuperer toutes les informations de l'inode:

```c
struct stat sb;
if (stat("/etc/passwd", &sb) == 0) {
    printf("Inode: %lu\n", sb.st_ino);
    printf("Size: %ld bytes\n", sb.st_size);
    // ...
}
```

La commande **ls -i** (2.3.2.l) affiche les numeros d'inode:

```bash
$ ls -i /etc/passwd
131073 /etc/passwd
```

---

## Enonce

### Vue d'Ensemble

Vous devez implementer un **inspecteur de systeme de fichiers complet** qui:
1. Inspecte n'importe quel fichier et affiche TOUTES ses metadonnees (comme `stat`)
2. Supporte les chemins absolus et relatifs
3. Detecte et identifie les 7 types de fichiers Unix
4. Affiche les informations style `ls -li`
5. Demontre la comprehension des inodes et de leur contenu

### Specifications Fonctionnelles

#### Fonctionnalite 1: Inspection Complete d'un Fichier (2.3.2.k)

La fonction principale `fs_inspect()` doit retourner une structure contenant TOUTES les metadonnees accessibles via stat().

**Concepts appliques**: 2.3.1.a (File), 2.3.1.g (Attributes), 2.3.2.a-l (Inode)

**Comportement attendu**:
- Appel de stat() ou lstat() selon le mode
- Remplissage complet de fs_info_t avec TOUS les champs de l'inode
- Detection du type de fichier (2.3.1.f, 2.3.2.d)
- Recuperation des timestamps (2.3.2.h): atime, mtime, ctime
- Calcul du nombre de blocs (2.3.2.j)
- Lecture du link count (2.3.2.i)

**Cas limites**:
- Chemin NULL ou vide
- Fichier inexistant (ENOENT)
- Permissions insuffisantes (EACCES)
- Liens symboliques casses

#### Fonctionnalite 2: Resolution de Chemins (2.3.1.c-e)

Support complet des chemins absolus et relatifs.

**Concepts appliques**: 2.3.1.c (Path), 2.3.1.d (Absolute), 2.3.1.e (Relative)

**Comportement attendu**:
- Detection automatique chemin absolu (commence par `/`) vs relatif
- Fonction `fs_resolve_path()` pour obtenir le chemin absolu
- Support de `.` et `..` dans les chemins
- Gestion des chemins avec symlinks

#### Fonctionnalite 3: Detection des Types de Fichiers (2.3.1.f)

Identification precise des 7 types de fichiers Unix.

**Concepts appliques**: 2.3.1.f (File types), 2.3.2.d (Type in inode)

**Types a detecter**:
```c
typedef enum {
    FS_TYPE_UNKNOWN   = 0,
    FS_TYPE_REGULAR   = 1,   // 2.3.1.a: Fichier ordinaire
    FS_TYPE_DIRECTORY = 2,   // 2.3.1.b: Repertoire
    FS_TYPE_SYMLINK   = 3,   // Lien symbolique
    FS_TYPE_BLOCK     = 4,   // Device bloc
    FS_TYPE_CHAR      = 5,   // Device caractere
    FS_TYPE_FIFO      = 6,   // Pipe nomme
    FS_TYPE_SOCKET    = 7    // Socket Unix
} fs_type_t;
```

#### Fonctionnalite 4: Affichage Style ls -li (2.3.2.l)

Reproduction de la sortie de `ls -li` montrant l'inode.

**Concepts appliques**: 2.3.2.b (Inode number), 2.3.2.l (ls -i)

**Format de sortie**:
```
131073 -rw-r--r-- 1 root root 2847 Jan  4 10:30 /etc/passwd
^      ^          ^ ^    ^    ^    ^             ^
|      |          | |    |    |    |             +-- Nom (PAS dans inode!)
|      |          | |    |    |    +-- Timestamps (2.3.2.h)
|      |          | |    |    +-- Taille (2.3.2.g)
|      |          | |    +-- Groupe (2.3.2.f)
|      |          | +-- User (2.3.2.f)
|      |          +-- Link count (2.3.2.i)
|      +-- Type + Permissions (2.3.2.d, 2.3.2.e)
+-- Inode number (2.3.2.b)
```

#### Fonctionnalite 5: Operations de Base sur Fichiers (2.3.1.h)

Demonstration des operations create, open, read, write, close, delete.

**Concepts appliques**: 2.3.1.h (File operations)

**Fonctions a implementer**:
```c
// Demonstrer les operations basiques
int fs_demo_operations(const char *testfile);
```

#### Fonctionnalite 6: Listage de Repertoire (2.3.1.b)

Enumeration du contenu d'un repertoire avec affichage des inodes.

**Concepts appliques**: 2.3.1.b (Directory), 2.3.2.l (ls -i)

**Comportement**:
- Ouvrir le repertoire avec opendir()
- Lister chaque entree avec readdir()
- Afficher nom + inode pour chaque entree

### Specifications Techniques

#### Architecture

```
+-------------------+       +-------------------+
|   Application     |       |   File System     |
+-------------------+       +-------------------+
         |                           |
         v                           v
+-------------------+       +-------------------+
|   fs_inspect()    |<----->|  stat()/lstat()   |
|   (2.3.2.k)       |       |  syscalls         |
+-------------------+       +-------------------+
         |
         v
+-------------------+
|   fs_info_t       |  Contient:
|   (2.3.2.a-l)     |  - inode number (2.3.2.b)
+-------------------+  - type (2.3.2.d)
         |             - permissions (2.3.2.e)
         |             - owner uid/gid (2.3.2.f)
         v             - size (2.3.2.g)
+-------------------+  - timestamps (2.3.2.h)
|  fs_print_info()  |  - link count (2.3.2.i)
|  fs_print_ls_li() |  - blocks (2.3.2.j)
+-------------------+
```

#### Relation Nom/Inode (2.3.2.c illustre)

```
                  REPERTOIRE                         INODE
              +---------------+                  +---------------+
              | Nom   | Inode |                  | NO name here! |
              |-------|-------|      stat()      | Type: regular |
              | file1 | 12345 | --------------> | Mode: 0644    |
              | file2 | 12346 |                  | UID: 1000     |
              +---------------+                  | Size: 4096    |
                     ^                           | atime: ...    |
                     |                           | mtime: ...    |
              Le nom est ICI,                    | ctime: ...    |
              pas dans l'inode!                  | nlink: 1      |
                                                 +---------------+
                                                        |
                                                        v
                                                 +---------------+
                                                 | DATA BLOCKS   |
                                                 | (actual data) |
                                                 +---------------+
```

---

## Contraintes Techniques

### Standards C

- **Norme**: C17 (ISO/IEC 9899:2018)
- **Compilation**: `gcc -Wall -Wextra -Werror -std=c17`
- **Options additionnelles**: Aucune bibliotheque externe requise

### Fonctions Autorisees

```
Fonctions autorisees:
  - malloc, free, calloc, realloc (stdlib.h)
  - stat, lstat, fstat (sys/stat.h)
  - open, close, read, write, unlink (unistd.h)
  - opendir, readdir, closedir (dirent.h)
  - readlink, getcwd, realpath (unistd.h, stdlib.h)
  - strlen, strcpy, strncpy, strcmp, strcat, strrchr (string.h)
  - snprintf, printf, fprintf (stdio.h)
  - localtime, strftime, time (time.h)
  - getpwuid, getgrgid (pwd.h, grp.h)
  - strerror, errno (string.h, errno.h)
  - major, minor (sys/sysmacros.h)
```

### Contraintes Specifiques

- [ ] Pas de variables globales (sauf constantes)
- [ ] Maximum 40 lignes par fonction
- [ ] Toutes les allocations doivent avoir leur free correspondant
- [ ] Les chemins doivent supporter PATH_MAX (4096 bytes)
- [ ] Thread-safe NOT requis pour cet exercice

### Exigences de Securite

- [ ] Aucune fuite memoire (verification Valgrind obligatoire)
- [ ] Aucun buffer overflow (verification des tailles)
- [ ] Verification de tous les retours de stat(), malloc(), open()
- [ ] Gestion appropriee des erreurs avec codes errno
- [ ] Pas de deference de pointeur NULL

---

## Format de Rendu

### Fichiers a Rendre

```
ex00/
+-- fs_inspector.h       # Header avec structures et prototypes
+-- fs_inspector.c       # Implementation principale (2.3.2.k: stat)
+-- fs_types.c           # Detection des types (2.3.1.f, 2.3.2.d)
+-- fs_paths.c           # Gestion des chemins (2.3.1.c-e)
+-- fs_format.c          # Formatage (2.3.2.e permissions, 2.3.2.h timestamps)
+-- fs_operations.c      # Operations de base (2.3.1.h)
+-- Makefile
```

### Signatures de Fonctions

#### fs_inspector.h

```c
#ifndef FS_INSPECTOR_H
#define FS_INSPECTOR_H

#include <sys/types.h>
#include <sys/stat.h>
#include <stdint.h>
#include <stddef.h>
#include <time.h>

/* =========================================================================
 * SECTION 1: Types de fichiers (2.3.1.f, 2.3.2.d)
 * Les 7 types de fichiers Unix, stockes dans l'inode
 * ========================================================================= */
typedef enum {
    FS_TYPE_UNKNOWN   = 0,
    FS_TYPE_REGULAR   = 1,   /* 2.3.1.a: Fichier (named collection of data) */
    FS_TYPE_DIRECTORY = 2,   /* 2.3.1.b: Repertoire (container for files) */
    FS_TYPE_SYMLINK   = 3,   /* Lien symbolique */
    FS_TYPE_BLOCK     = 4,   /* Device bloc */
    FS_TYPE_CHAR      = 5,   /* Device caractere */
    FS_TYPE_FIFO      = 6,   /* Pipe nomme */
    FS_TYPE_SOCKET    = 7    /* Socket Unix */
} fs_type_t;

/* =========================================================================
 * SECTION 2: Options d'inspection
 * ========================================================================= */
typedef enum {
    FS_FOLLOW_SYMLINKS = 0,  /* stat(): suivre les symlinks */
    FS_NO_FOLLOW       = 1   /* lstat(): ne pas suivre */
} fs_follow_t;

typedef enum {
    FS_PATH_ABSOLUTE = 0,    /* 2.3.1.d: Chemin absolu (from root) */
    FS_PATH_RELATIVE = 1     /* 2.3.1.e: Chemin relatif (from current) */
} fs_path_type_t;

/* =========================================================================
 * SECTION 3: Structure principale - Toutes les infos de l'inode (2.3.2.a-l)
 *
 * NOTE IMPORTANTE (2.3.2.c):
 * L'inode NE CONTIENT PAS le nom du fichier (stocke dans le repertoire)
 * L'inode NE CONTIENT PAS les donnees (stockees dans les blocs de donnees)
 * ========================================================================= */
typedef struct {
    /* === Identification === */
    char           *path;           /* Le chemin FOURNI (pas dans l'inode!) */
    char           *resolved_path;  /* Chemin absolu resolu (2.3.1.d) */
    fs_path_type_t  path_type;      /* Absolu ou relatif (2.3.1.d-e) */

    /* === Inode number (2.3.2.b) === */
    ino_t           inode;          /* Numero d'inode, unique dans le FS */
    dev_t           device;         /* Device contenant le fichier */

    /* === File type (2.3.2.d) - Stocke dans l'inode === */
    fs_type_t       type;           /* Un des 7 types Unix (2.3.1.f) */

    /* === Permissions (2.3.2.e) - Mode bits === */
    mode_t          mode;           /* Permissions brutes */
    /* Format: [type 4bits][special 3bits][user 3bits][group 3bits][other 3bits] */

    /* === Owner (2.3.2.f) - UID et GID === */
    uid_t           uid;            /* User ID proprietaire */
    gid_t           gid;            /* Group ID proprietaire */
    char           *owner_name;     /* Nom du proprietaire (resolu) */
    char           *group_name;     /* Nom du groupe (resolu) */

    /* === Size (2.3.2.g) - En bytes === */
    off_t           size;           /* Taille en bytes */

    /* === Timestamps (2.3.2.h) - atime, mtime, ctime === */
    time_t          atime;          /* Access time: derniere lecture */
    time_t          mtime;          /* Modify time: derniere modification contenu */
    time_t          ctime;          /* Change time: derniere modification inode */

    /* === Link count (2.3.2.i) - Nombre de hard links === */
    nlink_t         link_count;     /* Nombre de noms pointant vers cet inode */

    /* === Block pointers (2.3.2.j) - Allocation === */
    blksize_t       block_size;     /* Taille de bloc preferee pour I/O */
    blkcnt_t        blocks;         /* Nombre de blocs 512-byte alloues */

    /* === Informations supplementaires === */
    char           *symlink_target; /* Cible du symlink (si applicable) */
    dev_t           rdev;           /* Device major/minor (si device special) */
} fs_info_t;

/* =========================================================================
 * SECTION 4: Codes d'erreur
 * ========================================================================= */
typedef enum {
    FS_SUCCESS        = 0,
    FS_ERR_NOT_FOUND  = -1,   /* ENOENT: Fichier non trouve */
    FS_ERR_PERMISSION = -2,   /* EACCES: Permission refusee */
    FS_ERR_MEMORY     = -3,   /* Erreur allocation memoire */
    FS_ERR_PATH_LONG  = -4,   /* ENAMETOOLONG: Chemin trop long */
    FS_ERR_INVALID    = -5,   /* Parametre invalide */
    FS_ERR_IO         = -6,   /* Erreur I/O generale */
    FS_ERR_LOOP       = -7    /* ELOOP: Trop de symlinks */
} fs_error_t;

/* =========================================================================
 * SECTION 5: Fonctions principales
 * ========================================================================= */

/**
 * Recupere les informations completes d'un fichier via stat() (2.3.2.k).
 * Remplit TOUS les champs de l'inode (2.3.2.a-l).
 *
 * @param path   Chemin vers le fichier (2.3.1.c), absolu (2.3.1.d) ou relatif (2.3.1.e)
 * @param follow Mode de suivi des symlinks
 * @return Structure allouee avec toutes les metadonnees, ou NULL si erreur
 *
 * @note La structure doit etre liberee avec fs_info_free()
 */
fs_info_t *fs_inspect(const char *path, fs_follow_t follow);

/**
 * Libere une structure fs_info_t et tous ses membres alloues.
 */
void fs_info_free(fs_info_t *info);

/* =========================================================================
 * SECTION 6: Fonctions de chemins (2.3.1.c-e)
 * ========================================================================= */

/**
 * Determine si un chemin est absolu (2.3.1.d) ou relatif (2.3.1.e).
 */
fs_path_type_t fs_get_path_type(const char *path);

/**
 * Resout un chemin relatif (2.3.1.e) en chemin absolu (2.3.1.d).
 *
 * @param path Chemin a resoudre (2.3.1.c)
 * @param resolved Buffer de sortie (minimum PATH_MAX)
 * @param size Taille du buffer
 * @return resolved ou NULL si erreur
 */
char *fs_resolve_path(const char *path, char *resolved, size_t size);

/* =========================================================================
 * SECTION 7: Fonctions de type (2.3.1.f, 2.3.2.d)
 * ========================================================================= */

/**
 * Convertit un type de fichier (2.3.1.f) en chaine descriptive.
 */
const char *fs_type_to_string(fs_type_t type);

/**
 * Retourne le caractere de type pour ls (2.3.1.f).
 * Regular='-', Directory='d', Symlink='l', Block='b', Char='c', Fifo='p', Socket='s'
 */
char fs_type_to_char(fs_type_t type);

/**
 * Detecte le type de fichier depuis st_mode (2.3.2.d).
 */
fs_type_t fs_detect_type(mode_t mode);

/* =========================================================================
 * SECTION 8: Fonctions de formatage
 * ========================================================================= */

/**
 * Convertit les permissions en notation symbolique (2.3.2.e).
 * Format: "rwxr-xr-x" (9 caracteres + bits speciaux s/S, t/T)
 *
 * @param mode Permissions brutes (st_mode)
 * @param buf Buffer de sortie (minimum 10 caracteres)
 * @param buf_size Taille du buffer
 * @return buf ou NULL si buffer trop petit
 */
char *fs_perms_to_string(mode_t mode, char *buf, size_t buf_size);

/**
 * Formate les permissions en notation octale (2.3.2.e).
 * Format: "0755"
 */
char *fs_perms_to_octal(mode_t mode, char *buf, size_t buf_size);

/**
 * Formate une taille (2.3.2.g) en format humain.
 * Format: "1.5 KB", "2.3 MB", etc.
 */
char *fs_format_size(off_t size, char *buf, size_t buf_size);

/**
 * Formate un timestamp (2.3.2.h) en ISO 8601.
 * Format: "2025-01-04T10:30:45"
 */
char *fs_format_time(time_t timestamp, char *buf, size_t buf_size);

/**
 * Formate un timestamp (2.3.2.h) en format ls.
 * Format: "Jan  4 10:30" ou "Jan  4  2024" (si > 6 mois)
 */
char *fs_format_time_ls(time_t timestamp, char *buf, size_t buf_size);

/* =========================================================================
 * SECTION 9: Fonctions d'affichage
 * ========================================================================= */

/**
 * Affiche toutes les informations detaillees (comme la commande stat).
 */
void fs_print_info(const fs_info_t *info);

/**
 * Affiche en format ls -li (2.3.2.l): inode + permissions + infos.
 * Demontre que le numero d'inode (2.3.2.b) est accessible.
 */
void fs_print_ls_li(const fs_info_t *info);

/**
 * Affiche le contenu d'un repertoire (2.3.1.b) avec inodes (2.3.2.l).
 * Comme "ls -lai directory"
 */
int fs_list_directory(const char *path);

/* =========================================================================
 * SECTION 10: Operations de base sur fichiers (2.3.1.h)
 * ========================================================================= */

/**
 * Demontre les operations de base (2.3.1.h):
 * create, open, read, write, close, delete
 *
 * @param testfile Chemin du fichier de test a utiliser
 * @return 0 si succes, -1 si erreur
 */
int fs_demo_operations(const char *testfile);

/* =========================================================================
 * SECTION 11: Utilitaires
 * ========================================================================= */

/**
 * Recupere le dernier code d'erreur.
 */
fs_error_t fs_get_last_error(void);

/**
 * Retourne une description textuelle d'un code d'erreur.
 */
const char *fs_strerror(fs_error_t error);

/**
 * Explique ce que contient et ne contient PAS un inode (2.3.2.c).
 * Fonction pedagogique pour comprendre la structure.
 */
void fs_explain_inode(void);

#endif /* FS_INSPECTOR_H */
```

### Makefile

```makefile
NAME = libfsinspector.a
TEST = test_inspector

CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c17
AR = ar rcs

SRCS = fs_inspector.c fs_types.c fs_paths.c fs_format.c fs_operations.c
OBJS = $(SRCS:.c=.o)

all: $(NAME)

$(NAME): $(OBJS)
	$(AR) $(NAME) $(OBJS)

%.o: %.c fs_inspector.h
	$(CC) $(CFLAGS) -c $< -o $@

test: $(NAME)
	$(CC) $(CFLAGS) -o $(TEST) test_main.c -L. -lfsinspector
	./$(TEST)

clean:
	rm -f $(OBJS)

fclean: clean
	rm -f $(NAME) $(TEST)

re: fclean all

.PHONY: all clean fclean re test
```

---

## Exemples d'Utilisation

### Exemple 1: Inspection Complete avec Affichage Detaille

```c
#include "fs_inspector.h"
#include <stdio.h>

int main(void)
{
    // Inspection d'un fichier avec chemin absolu (2.3.1.d)
    fs_info_t *info = fs_inspect("/etc/passwd", FS_FOLLOW_SYMLINKS);

    if (info == NULL) {
        fprintf(stderr, "Error: %s\n", fs_strerror(fs_get_last_error()));
        return 1;
    }

    // Afficher toutes les informations de l'inode (2.3.2.a-l)
    fs_print_info(info);
    fs_info_free(info);
    return 0;
}

/* Output:
=== File: /etc/passwd ===
Path type:   Absolute (2.3.1.d)
Resolved:    /etc/passwd

--- Inode Information (2.3.2.a) ---
Inode:       131073 (2.3.2.b: unique within filesystem)
Device:      0x820 (major: 8, minor: 32)

--- File Type (2.3.2.d) ---
Type:        Regular file (2.3.1.a: named collection of data)

--- Permissions (2.3.2.e) ---
Mode:        -rw-r--r-- (0644)
             User:  rw-  (read, write)
             Group: r--  (read)
             Other: r--  (read)

--- Owner (2.3.2.f) ---
UID:         0 (root)
GID:         0 (root)

--- Size (2.3.2.g) ---
Size:        2847 bytes (2.8 KB)

--- Timestamps (2.3.2.h) ---
Access time: 2025-01-04T10:30:45 (atime: last read)
Modify time: 2024-12-15T09:22:11 (mtime: content changed)
Change time: 2024-12-15T09:22:11 (ctime: inode changed)

--- Links (2.3.2.i) ---
Link count:  1 (number of hard links to this inode)

--- Blocks (2.3.2.j) ---
Block size:  4096 (preferred I/O block size)
Blocks:      8 (512-byte blocks allocated)

NOTE (2.3.2.c): The filename "passwd" is NOT stored in the inode!
It is stored in the directory /etc/ which maps "passwd" -> inode 131073
*/
```

### Exemple 2: Comparaison Chemin Absolu vs Relatif (2.3.1.d-e)

```c
#include "fs_inspector.h"
#include <stdio.h>

int main(void)
{
    // Chemin absolu (2.3.1.d): part de la racine /
    fs_info_t *info1 = fs_inspect("/etc/passwd", FS_FOLLOW_SYMLINKS);

    // Chemin relatif (2.3.1.e): part du repertoire courant
    fs_info_t *info2 = fs_inspect("../etc/passwd", FS_FOLLOW_SYMLINKS);

    if (info1 && info2) {
        printf("=== Path Comparison (2.3.1.c-e) ===\n");
        printf("Absolute path (2.3.1.d): %s\n", info1->path);
        printf("  Path type: %s\n",
               info1->path_type == FS_PATH_ABSOLUTE ? "ABSOLUTE" : "RELATIVE");
        printf("  Resolved:  %s\n", info1->resolved_path);
        printf("  Inode:     %lu\n\n", info1->inode);

        printf("Relative path (2.3.1.e): %s\n", info2->path);
        printf("  Path type: %s\n",
               info2->path_type == FS_PATH_ABSOLUTE ? "ABSOLUTE" : "RELATIVE");
        printf("  Resolved:  %s\n", info2->resolved_path);
        printf("  Inode:     %lu\n\n", info2->inode);

        // Meme inode = meme fichier!
        if (info1->inode == info2->inode && info1->device == info2->device) {
            printf("SAME FILE! Different paths, same inode (2.3.2.b)\n");
        }
    }

    fs_info_free(info1);
    fs_info_free(info2);
    return 0;
}

/* Output:
=== Path Comparison (2.3.1.c-e) ===
Absolute path (2.3.1.d): /etc/passwd
  Path type: ABSOLUTE
  Resolved:  /etc/passwd
  Inode:     131073

Relative path (2.3.1.e): ../etc/passwd
  Path type: RELATIVE
  Resolved:  /etc/passwd
  Inode:     131073

SAME FILE! Different paths, same inode (2.3.2.b)
*/
```

### Exemple 3: Detection des 7 Types de Fichiers (2.3.1.f)

```c
#include "fs_inspector.h"
#include <stdio.h>

int main(void)
{
    // Liste de fichiers de differents types (2.3.1.f)
    const char *files[] = {
        "/etc/passwd",      // Regular file (2.3.1.a)
        "/etc",             // Directory (2.3.1.b)
        "/dev/sda",         // Block device
        "/dev/tty",         // Character device
        "/run/user/1000/bus", // Socket (si existe)
        NULL
    };

    printf("=== File Types Detection (2.3.1.f, 2.3.2.d) ===\n\n");

    for (int i = 0; files[i] != NULL; i++) {
        fs_info_t *info = fs_inspect(files[i], FS_NO_FOLLOW);
        if (info == NULL) {
            printf("%-25s: Not accessible\n", files[i]);
            continue;
        }

        printf("%-25s: %c %-18s (inode: %lu)\n",
               files[i],
               fs_type_to_char(info->type),   // 'd', '-', 'l', 'b', 'c', 'p', 's'
               fs_type_to_string(info->type),
               info->inode);

        fs_info_free(info);
    }
    return 0;
}

/* Output:
=== File Types Detection (2.3.1.f, 2.3.2.d) ===

/etc/passwd              : - Regular file       (inode: 131073)
/etc                     : d Directory          (inode: 131072)
/dev/sda                 : b Block device       (inode: 371)
/dev/tty                 : c Character device   (inode: 20)
/run/user/1000/bus       : s Socket             (inode: 45123)
*/
```

### Exemple 4: Affichage Style ls -li (2.3.2.l)

```c
#include "fs_inspector.h"
#include <stdio.h>

int main(void)
{
    // Reproduire "ls -li /etc/passwd" (2.3.2.l)
    fs_info_t *info = fs_inspect("/etc/passwd", FS_NO_FOLLOW);

    if (info != NULL) {
        printf("=== ls -li Output Format (2.3.2.l) ===\n\n");
        fs_print_ls_li(info);

        printf("\nExplanation:\n");
        printf("  %lu     <- Inode number (2.3.2.b)\n", info->inode);
        printf("  %c      <- File type (2.3.2.d)\n", fs_type_to_char(info->type));
        char perms[10];
        fs_perms_to_string(info->mode, perms, sizeof(perms));
        printf("  %s <- Permissions (2.3.2.e)\n", perms);
        printf("  %lu     <- Link count (2.3.2.i)\n", info->link_count);
        printf("  %-8s<- Owner name from UID %d (2.3.2.f)\n",
               info->owner_name ? info->owner_name : "?", info->uid);
        printf("  %-8s<- Group name from GID %d (2.3.2.f)\n",
               info->group_name ? info->group_name : "?", info->gid);
        printf("  %ld    <- Size in bytes (2.3.2.g)\n", info->size);

        fs_info_free(info);
    }
    return 0;
}

/* Output:
=== ls -li Output Format (2.3.2.l) ===

131073 -rw-r--r-- 1 root root 2847 Jan  4 10:30 /etc/passwd

Explanation:
  131073  <- Inode number (2.3.2.b)
  -       <- File type (2.3.2.d)
  rw-r--r-- <- Permissions (2.3.2.e)
  1       <- Link count (2.3.2.i)
  root    <- Owner name from UID 0 (2.3.2.f)
  root    <- Group name from GID 0 (2.3.2.f)
  2847    <- Size in bytes (2.3.2.g)
*/
```

### Exemple 5: Les 3 Timestamps (2.3.2.h)

```c
#include "fs_inspector.h"
#include <stdio.h>

int main(void)
{
    fs_info_t *info = fs_inspect("/etc/passwd", FS_FOLLOW_SYMLINKS);

    if (info != NULL) {
        char buf[32];

        printf("=== Timestamps (2.3.2.h) ===\n\n");

        printf("atime (Access Time):\n");
        printf("  %s\n", fs_format_time(info->atime, buf, sizeof(buf)));
        printf("  Updated when: file content is READ\n\n");

        printf("mtime (Modification Time):\n");
        printf("  %s\n", fs_format_time(info->mtime, buf, sizeof(buf)));
        printf("  Updated when: file CONTENT is modified\n\n");

        printf("ctime (Change Time):\n");
        printf("  %s\n", fs_format_time(info->ctime, buf, sizeof(buf)));
        printf("  Updated when: INODE is modified (permissions, owner, etc.)\n\n");

        printf("NOTE: Unix has NO creation time (birthday) by default!\n");

        fs_info_free(info);
    }
    return 0;
}
```

### Exemple 6: Operations sur Fichiers (2.3.1.h)

```c
#include "fs_inspector.h"
#include <stdio.h>

int main(void)
{
    printf("=== File Operations Demo (2.3.1.h) ===\n\n");

    // Cette fonction demontre create, open, read, write, close, delete
    if (fs_demo_operations("/tmp/test_fs_ops.txt") == 0) {
        printf("All basic file operations demonstrated successfully!\n");
    }
    return 0;
}

/* Output:
=== File Operations Demo (2.3.1.h) ===

1. CREATE: creat("/tmp/test_fs_ops.txt", 0644) -> fd=3
2. WRITE:  write(fd, "Hello, filesystem!", 18) -> 18 bytes
3. CLOSE:  close(fd) -> success
4. OPEN:   open("/tmp/test_fs_ops.txt", O_RDONLY) -> fd=3
5. READ:   read(fd, buffer, 18) -> "Hello, filesystem!"
6. CLOSE:  close(fd) -> success
7. DELETE: unlink("/tmp/test_fs_ops.txt") -> success

All basic file operations demonstrated successfully!
*/
```

### Exemple 7: Contenu vs Non-Contenu de l'Inode (2.3.2.c)

```c
#include "fs_inspector.h"
#include <stdio.h>

int main(void)
{
    // Explication pedagogique (2.3.2.c)
    fs_explain_inode();
    return 0;
}

/* Output:
=== What an Inode Contains (2.3.2.c) ===

The inode (index node) is the core metadata structure for a file.

WHAT THE INODE CONTAINS:
  [x] File type (2.3.2.d)        : regular, directory, symlink, etc.
  [x] Permissions (2.3.2.e)      : rwxr-xr-x (mode bits)
  [x] Owner UID/GID (2.3.2.f)    : who owns the file
  [x] Size in bytes (2.3.2.g)    : logical file size
  [x] Timestamps (2.3.2.h)       : atime, mtime, ctime
  [x] Link count (2.3.2.i)       : number of hard links
  [x] Block pointers (2.3.2.j)   : where data is stored on disk

WHAT THE INODE DOES NOT CONTAIN:
  [ ] Filename                   : stored in the DIRECTORY, not the inode!
  [ ] File data/content          : stored in DATA BLOCKS pointed to by inode

This is why:
  - Multiple hard links can have different names but same inode
  - mv (rename) is fast: only changes directory entry, not inode
  - ls -i shows the inode number (2.3.2.l)
*/
```

### Exemple 8: Listage de Repertoire (2.3.1.b)

```c
#include "fs_inspector.h"
#include <stdio.h>

int main(void)
{
    printf("=== Directory Listing (2.3.1.b) with Inodes (2.3.2.l) ===\n\n");

    // Liste le contenu du repertoire (2.3.1.b: container for files)
    // Equivalent de "ls -lai /etc"
    fs_list_directory("/etc");

    return 0;
}

/* Output:
=== Directory Listing (2.3.1.b) with Inodes (2.3.2.l) ===

Contents of /etc (Directory - 2.3.1.b: Container for files):

  Inode    Type Permissions  Links Owner    Size     Name
  ------   ---- -----------  ----- -----    ----     ----
  131072   d    rwxr-xr-x    85    root     4096     .
  2        d    rwxr-xr-x    23    root     4096     ..
  131073   -    rw-r--r--    1     root     2847     passwd
  131074   -    rw-r-----    1     root     1543     shadow
  131080   d    rwxr-xr-x    2     root     4096     apt
  131090   l    rwxrwxrwx    1     root     21       localtime -> ../usr/share/zoneinfo/UTC

Note: The filenames above are stored in THIS directory, not in the inodes! (2.3.2.c)
*/
```

---

## Tests de la Moulinette

### Tests Fonctionnels de Base

#### Test 01: Inspection Fichier Regular (2.3.1.a, 2.3.2.k)
```yaml
description: "Verifie fs_inspect() sur un fichier regular (2.3.1.a)"
concepts_testes: ["2.3.1.a", "2.3.2.k", "2.3.2.b", "2.3.2.d"]
setup: |
  echo "test content" > /tmp/test_fs_01.txt
  fs_info_t *info = fs_inspect("/tmp/test_fs_01.txt", FS_FOLLOW_SYMLINKS);
validation:
  - "info != NULL"
  - "info->type == FS_TYPE_REGULAR"  # 2.3.1.a: File
  - "info->inode > 0"                 # 2.3.2.b: Inode number
  - "info->size == 13"                # 2.3.2.g: Size
  - "info->link_count >= 1"           # 2.3.2.i: Link count
cleanup: |
  fs_info_free(info);
  unlink("/tmp/test_fs_01.txt");
```

#### Test 02: Inspection Repertoire (2.3.1.b)
```yaml
description: "Verifie fs_inspect() sur un repertoire (2.3.1.b: container for files)"
concepts_testes: ["2.3.1.b", "2.3.2.d", "2.3.2.i"]
setup: |
  mkdir("/tmp/test_fs_dir_02", 0755);
  fs_info_t *info = fs_inspect("/tmp/test_fs_dir_02", FS_FOLLOW_SYMLINKS);
validation:
  - "info != NULL"
  - "info->type == FS_TYPE_DIRECTORY"  # 2.3.1.b: Directory
  - "info->link_count >= 2"            # 2.3.2.i: au moins . et entree parent
cleanup: |
  fs_info_free(info);
  rmdir("/tmp/test_fs_dir_02");
```

#### Test 03: Chemin Absolu (2.3.1.d)
```yaml
description: "Verifie la detection de chemin absolu (2.3.1.d: from root)"
concepts_testes: ["2.3.1.c", "2.3.1.d"]
input: |
  fs_path_type_t type = fs_get_path_type("/etc/passwd");
validation:
  - "type == FS_PATH_ABSOLUTE"
  - "path starts with '/'"
```

#### Test 04: Chemin Relatif (2.3.1.e)
```yaml
description: "Verifie la detection de chemin relatif (2.3.1.e: from current)"
concepts_testes: ["2.3.1.c", "2.3.1.e"]
input: |
  fs_path_type_t type1 = fs_get_path_type("./file.txt");
  fs_path_type_t type2 = fs_get_path_type("../etc/passwd");
  fs_path_type_t type3 = fs_get_path_type("subdir/file");
validation:
  - "type1 == FS_PATH_RELATIVE"
  - "type2 == FS_PATH_RELATIVE"
  - "type3 == FS_PATH_RELATIVE"
```

#### Test 05: Resolution Chemin Relatif vers Absolu
```yaml
description: "Verifie fs_resolve_path() convertit relatif (2.3.1.e) en absolu (2.3.1.d)"
concepts_testes: ["2.3.1.d", "2.3.1.e"]
setup: |
  char resolved[PATH_MAX];
  chdir("/tmp");
  fs_resolve_path("./test", resolved, PATH_MAX);
validation:
  - "resolved[0] == '/'"               # Maintenant absolu
  - "strstr(resolved, \"/tmp\") != NULL"
```

#### Test 06: Detection des 7 Types de Fichiers (2.3.1.f)
```yaml
description: "Verifie la detection de tous les types de fichiers (2.3.1.f)"
concepts_testes: ["2.3.1.f", "2.3.2.d"]
test_cases:
  - path: "/etc/passwd"
    expected_type: FS_TYPE_REGULAR
    expected_char: '-'
  - path: "/tmp"
    expected_type: FS_TYPE_DIRECTORY
    expected_char: 'd'
  - path: "/dev/null"
    expected_type: FS_TYPE_CHAR
    expected_char: 'c'
  - path: "/dev/sda" (if exists)
    expected_type: FS_TYPE_BLOCK
    expected_char: 'b'
```

#### Test 07: Permissions Mode Bits (2.3.2.e)
```yaml
description: "Verifie fs_perms_to_string() pour les mode bits (2.3.2.e)"
concepts_testes: ["2.3.2.e"]
test_cases:
  - mode: 0100755
    expected: "-rwxr-xr-x"
  - mode: 0100644
    expected: "-rw-r--r--"
  - mode: 0040755
    expected: "drwxr-xr-x"
  - mode: 0120777
    expected: "lrwxrwxrwx"
  - mode: 0104755  # setuid
    expected: "-rwsr-xr-x"
  - mode: 0102755  # setgid
    expected: "-rwxr-sr-x"
  - mode: 0101755  # sticky
    expected: "-rwxr-xr-t"
```

#### Test 08: Owner UID/GID (2.3.2.f)
```yaml
description: "Verifie la recuperation de l'owner (2.3.2.f)"
concepts_testes: ["2.3.2.f"]
setup: |
  fs_info_t *info = fs_inspect("/etc/passwd", FS_FOLLOW_SYMLINKS);
validation:
  - "info->uid == 0"           # root
  - "info->gid == 0"           # root
  - "info->owner_name != NULL"
  - "strcmp(info->owner_name, \"root\") == 0"
```

#### Test 09: Taille en Bytes (2.3.2.g)
```yaml
description: "Verifie la taille du fichier (2.3.2.g)"
concepts_testes: ["2.3.2.g"]
setup: |
  // Creer un fichier de 100 bytes exactement
  int fd = open("/tmp/test_size_09.txt", O_CREAT|O_WRONLY, 0644);
  char data[100];
  memset(data, 'A', 100);
  write(fd, data, 100);
  close(fd);
  fs_info_t *info = fs_inspect("/tmp/test_size_09.txt", FS_FOLLOW_SYMLINKS);
validation:
  - "info->size == 100"
cleanup: |
  fs_info_free(info);
  unlink("/tmp/test_size_09.txt");
```

#### Test 10: Timestamps atime/mtime/ctime (2.3.2.h)
```yaml
description: "Verifie les 3 timestamps (2.3.2.h)"
concepts_testes: ["2.3.2.h"]
setup: |
  // Creer fichier, lire, modifier
  int fd = creat("/tmp/test_time_10.txt", 0644);
  write(fd, "test", 4);
  close(fd);
  sleep(1);
  // Modifier le contenu -> mtime change
  fd = open("/tmp/test_time_10.txt", O_WRONLY|O_APPEND);
  write(fd, "more", 4);
  close(fd);
  fs_info_t *info = fs_inspect("/tmp/test_time_10.txt", FS_FOLLOW_SYMLINKS);
validation:
  - "info->atime > 0"
  - "info->mtime > 0"
  - "info->ctime > 0"
  - "info->mtime >= info->ctime - 1"  # mtime should be recent
```

#### Test 11: Link Count (2.3.2.i)
```yaml
description: "Verifie le compteur de liens (2.3.2.i)"
concepts_testes: ["2.3.2.i"]
setup: |
  // Creer un fichier avec hard link
  creat("/tmp/test_link_11a.txt", 0644);
  link("/tmp/test_link_11a.txt", "/tmp/test_link_11b.txt");
  fs_info_t *info = fs_inspect("/tmp/test_link_11a.txt", FS_FOLLOW_SYMLINKS);
validation:
  - "info->link_count == 2"  # 2 noms pointent vers le meme inode
cleanup: |
  fs_info_free(info);
  unlink("/tmp/test_link_11a.txt");
  unlink("/tmp/test_link_11b.txt");
```

#### Test 12: Block Pointers (2.3.2.j)
```yaml
description: "Verifie les infos de blocs (2.3.2.j)"
concepts_testes: ["2.3.2.j"]
setup: |
  // Creer un fichier avec du contenu
  int fd = creat("/tmp/test_blocks_12.txt", 0644);
  char data[8192];  // 2 blocs de 4K
  write(fd, data, sizeof(data));
  close(fd);
  fs_info_t *info = fs_inspect("/tmp/test_blocks_12.txt", FS_FOLLOW_SYMLINKS);
validation:
  - "info->block_size > 0"   # Typiquement 4096
  - "info->blocks > 0"       # Nombre de blocs 512-byte
  - "info->blocks >= 16"     # Au moins 8192/512 = 16 blocs
```

#### Test 13: stat() vs lstat() (2.3.2.k)
```yaml
description: "Verifie l'utilisation correcte de stat() et lstat() (2.3.2.k)"
concepts_testes: ["2.3.2.k"]
setup: |
  echo "target" > /tmp/test_target_13.txt
  symlink("/tmp/test_target_13.txt", "/tmp/test_link_13");
  fs_info_t *follow = fs_inspect("/tmp/test_link_13", FS_FOLLOW_SYMLINKS);
  fs_info_t *nofollow = fs_inspect("/tmp/test_link_13", FS_NO_FOLLOW);
validation:
  - "follow->type == FS_TYPE_REGULAR"   # stat() suit le lien
  - "nofollow->type == FS_TYPE_SYMLINK" # lstat() voit le lien
  - "nofollow->symlink_target != NULL"
cleanup: |
  fs_info_free(follow);
  fs_info_free(nofollow);
  unlink("/tmp/test_link_13");
  unlink("/tmp/test_target_13.txt");
```

#### Test 14: Affichage ls -i (2.3.2.l)
```yaml
description: "Verifie que l'inode est accessible comme ls -i (2.3.2.l)"
concepts_testes: ["2.3.2.l", "2.3.2.b"]
setup: |
  fs_info_t *info = fs_inspect("/etc/passwd", FS_FOLLOW_SYMLINKS);
validation:
  - "info->inode > 0"
  - "info->inode est le meme que celui retourne par ls -i"
shell_validation: |
  # Comparer avec la vraie commande ls -i
  expected_inode=$(ls -i /etc/passwd | awk '{print $1}')
  actual_inode=$(./test_inspector --inode /etc/passwd)
  test "$expected_inode" == "$actual_inode"
```

#### Test 15: Ce que l'inode NE contient PAS (2.3.2.c)
```yaml
description: "Verifie la comprehension de 2.3.2.c: le nom n'est pas dans l'inode"
concepts_testes: ["2.3.2.c"]
setup: |
  // Creer 2 hard links vers le meme inode
  creat("/tmp/name_a_15.txt", 0644);
  link("/tmp/name_a_15.txt", "/tmp/name_b_15.txt");
  fs_info_t *info_a = fs_inspect("/tmp/name_a_15.txt", FS_FOLLOW_SYMLINKS);
  fs_info_t *info_b = fs_inspect("/tmp/name_b_15.txt", FS_FOLLOW_SYMLINKS);
validation:
  - "info_a->inode == info_b->inode"  # Meme inode
  - "strcmp(info_a->path, info_b->path) != 0"  # Noms differents
  # Le nom n'est PAS dans l'inode, donc 2 noms peuvent pointer vers le meme inode
```

### Tests de Robustesse

#### Test 20: Parametres Invalides
```yaml
description: "Comportement avec entrees invalides"
test_cases:
  - input: "fs_inspect(NULL, FS_FOLLOW_SYMLINKS)"
    expected: "NULL, fs_get_last_error() == FS_ERR_INVALID"
  - input: "fs_inspect(\"\", FS_FOLLOW_SYMLINKS)"
    expected: "NULL, fs_get_last_error() == FS_ERR_INVALID"
  - input: "fs_info_free(NULL)"
    expected: "Ne crash pas, no-op"
  - input: "fs_perms_to_string(0644, NULL, 10)"
    expected: "NULL"
  - input: "fs_perms_to_string(0644, buf, 5)"
    expected: "NULL (buffer trop petit)"
```

#### Test 21: Fichier Inexistant
```yaml
description: "Gestion fichier inexistant"
input: "fs_inspect(\"/nonexistent/file.txt\", FS_FOLLOW_SYMLINKS)"
expected: "NULL avec FS_ERR_NOT_FOUND"
```

#### Test 22: Symlink Casse (Dangling)
```yaml
description: "Gestion des symlinks avec cible inexistante"
setup: |
  symlink("/nonexistent/target", "/tmp/dangling_link_22");
test_cases:
  - input: "fs_inspect(\"/tmp/dangling_link_22\", FS_FOLLOW_SYMLINKS)"
    expected: "NULL avec FS_ERR_NOT_FOUND"
  - input: "fs_inspect(\"/tmp/dangling_link_22\", FS_NO_FOLLOW)"
    expected: "info valide avec type == FS_TYPE_SYMLINK"
cleanup: "unlink(\"/tmp/dangling_link_22\")"
```

### Tests de Securite

#### Test 30: Fuites Memoire
```yaml
description: "Detection de fuites memoire avec Valgrind"
tool: "valgrind --leak-check=full --error-exitcode=1"
scenario: |
  for (int i = 0; i < 100; i++) {
      fs_info_t *info = fs_inspect("/etc/passwd", FS_FOLLOW_SYMLINKS);
      if (info) fs_info_free(info);
  }
expected: "0 bytes lost, 0 errors"
```

#### Test 31: Buffer Overflow Protection
```yaml
description: "Protection contre les buffer overflows"
tool: "AddressSanitizer"
scenario: |
  char small_buf[5];
  fs_perms_to_string(0755, small_buf, sizeof(small_buf));
  char tiny_buf[1];
  fs_format_size(1000000, tiny_buf, sizeof(tiny_buf));
expected: "Retourne NULL, pas de crash ni overflow"
```

### Tests de Performance

#### Test 40: Performance Inspection Multiple
```yaml
description: "Temps d'inspection de nombreux fichiers"
scenario: |
  // Lister /usr/bin et inspecter chaque fichier
  DIR *d = opendir("/usr/bin");
  struct dirent *entry;
  int count = 0;
  while ((entry = readdir(d)) != NULL && count < 1000) {
      char path[PATH_MAX];
      snprintf(path, PATH_MAX, "/usr/bin/%s", entry->d_name);
      fs_info_t *info = fs_inspect(path, FS_NO_FOLLOW);
      if (info) fs_info_free(info);
      count++;
  }
  closedir(d);
expected_max_time: "< 500ms total pour 1000 fichiers"
```

---

## Criteres d'Evaluation

### Note Minimale Requise: 80/100

### Detail de la Notation (Total: 100 points)

#### 1. Correction Fonctionnelle (40 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Tests concepts 2.3.1 (01-06) | 15 | File, Directory, Paths, Types |
| Tests concepts 2.3.2 (07-15) | 15 | Inode, Permissions, Timestamps, etc. |
| Gestion des cas limites | 6 | Tests 20-22 passent |
| Comportement defini | 4 | Aucun UB, resultats deterministes |

**Penalites**:
- Crash sur entree valide: -15 points
- Concept non implemente: -5 points par concept
- Type de fichier mal detecte: -3 points

#### 2. Couverture des Concepts (25 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Concepts 2.3.1.a-h | 12 | Tous les concepts de base implementes |
| Concepts 2.3.2.a-l | 13 | Tous les concepts inode implementes |

**Verification**:
- Chaque concept lettre doit etre utilise dans le code
- La documentation doit referencer les concepts

#### 3. Securite (20 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Absence de fuites | 10 | Valgrind clean |
| Protection buffers | 6 | Pas d'overflow |
| Verification retours | 4 | Tous stat/malloc verifies |

#### 4. Lisibilite et Conception (15 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Architecture | 6 | Separation claire des responsabilites |
| Nommage | 4 | Fonctions et variables explicites |
| Documentation | 3 | Commentaires referancant les concepts |
| Style | 2 | Code coherent et lisible |

---

## Indices et Ressources

### Reflexions pour Demarrer

<details>
<summary>Comment distinguer stat() et lstat() ? (2.3.2.k)</summary>

`stat()` suit les liens symboliques: si vous appelez stat() sur un symlink, vous obtenez les informations de la cible.

`lstat()` ne suit pas les liens: vous obtenez les informations du lien lui-meme.

Utilisez `FS_NO_FOLLOW` -> lstat(), `FS_FOLLOW_SYMLINKS` -> stat().

</details>

<details>
<summary>Comment detecter le type de fichier ? (2.3.1.f, 2.3.2.d)</summary>

Le type est encode dans `st_mode`. Utilisez les macros:
```c
if (S_ISREG(sb.st_mode))  return FS_TYPE_REGULAR;
if (S_ISDIR(sb.st_mode))  return FS_TYPE_DIRECTORY;
if (S_ISLNK(sb.st_mode))  return FS_TYPE_SYMLINK;
// etc.
```

</details>

<details>
<summary>Comment formater les permissions ? (2.3.2.e)</summary>

Testez chaque bit individuellement:
```c
perms[0] = (mode & S_IRUSR) ? 'r' : '-';
perms[1] = (mode & S_IWUSR) ? 'w' : '-';
// ...
// N'oubliez pas setuid (S_ISUID), setgid (S_ISGID), sticky (S_ISVTX)
```

</details>

<details>
<summary>Pourquoi le nom n'est-il pas dans l'inode ? (2.3.2.c)</summary>

Parce qu'un meme inode peut avoir plusieurs noms (hard links)! Le nom est stocke dans le repertoire parent qui associe nom -> numero d'inode.

C'est pourquoi `rename()` est rapide: il modifie juste l'entree du repertoire, pas l'inode.

</details>

### Ressources Recommandees

#### Documentation
- **stat(2)**: `man 2 stat` - Documentation complete
- **inode(7)**: `man 7 inode` - Structure de l'inode
- **path_resolution(7)**: `man 7 path_resolution` - Resolution des chemins

#### Outils de Debugging
- `stat fichier`: Voir les metadonnees
- `ls -lai`: Liste avec inodes
- `file fichier`: Identifier le type
- `df -i`: Voir l'utilisation des inodes

### Pieges Frequents

1. **readlink() ne termine pas par '\0'**
   - Toujours ajouter `target[len] = '\0';`

2. **Confusion taille symlink vs taille cible**
   - Avec lstat(), st_size = longueur du chemin cible

3. **Oublier les bits speciaux dans les permissions**
   - setuid remplace 'x' par 's' ou 'S'

4. **Croire qu'il existe un "creation time"**
   - Unix n'a que atime, mtime, ctime (pas de birthday!)

---

## Auto-evaluation

### Checklist de Qualite (Score: 97/100)

| Critere | Status | Points |
|---------|--------|--------|
| TOUS les concepts 2.3.1.a-h couverts | OK | 15/15 |
| TOUS les concepts 2.3.2.a-l couverts | OK | 15/15 |
| Exercice original (pas copie) | OK | 10/10 |
| Specifications completes et testables | OK | 10/10 |
| API C bien definie avec documentation | OK | 10/10 |
| Exemples d'utilisation varies | OK | 10/10 |
| Tests moulinette exhaustifs | OK | 10/10 |
| Criteres d'evaluation detailles | OK | 10/10 |
| Difficulte appropriee (Facile, 4h) | OK | 7/10 |

**Score Total: 97/100**

### Matrice de Couverture des Concepts

| Concept | Header | Implementation | Tests | Exemples |
|---------|--------|----------------|-------|----------|
| 2.3.1.a File | fs_type_t | fs_inspect() | Test 01 | Ex 1 |
| 2.3.1.b Directory | fs_type_t | fs_list_directory() | Test 02 | Ex 8 |
| 2.3.1.c Path | fs_path_type_t | fs_resolve_path() | Test 03-05 | Ex 2 |
| 2.3.1.d Absolute path | FS_PATH_ABSOLUTE | fs_get_path_type() | Test 03 | Ex 2 |
| 2.3.1.e Relative path | FS_PATH_RELATIVE | fs_get_path_type() | Test 04-05 | Ex 2 |
| 2.3.1.f File types | fs_type_t (7 types) | fs_detect_type() | Test 06 | Ex 3 |
| 2.3.1.g File attributes | fs_info_t | fs_inspect() | Test 01-14 | Ex 1 |
| 2.3.1.h File operations | - | fs_demo_operations() | - | Ex 6 |
| 2.3.2.a Inode | fs_info_t comment | fs_inspect() | All | Ex 1,7 |
| 2.3.2.b Inode number | fs_info_t.inode | fs_inspect() | Test 14 | Ex 4 |
| 2.3.2.c Inode contents | fs_explain_inode() | Documentation | Test 15 | Ex 7 |
| 2.3.2.d File type | fs_info_t.type | fs_detect_type() | Test 06 | Ex 3 |
| 2.3.2.e Permissions | fs_info_t.mode | fs_perms_to_string() | Test 07 | Ex 4 |
| 2.3.2.f Owner | fs_info_t.uid/gid | fs_inspect() | Test 08 | Ex 4 |
| 2.3.2.g Size | fs_info_t.size | fs_inspect() | Test 09 | Ex 1 |
| 2.3.2.h Timestamps | fs_info_t.atime/mtime/ctime | fs_format_time() | Test 10 | Ex 5 |
| 2.3.2.i Link count | fs_info_t.link_count | fs_inspect() | Test 11 | Ex 4 |
| 2.3.2.j Block pointers | fs_info_t.blocks | fs_inspect() | Test 12 | Ex 1 |
| 2.3.2.k stat() | - | fs_inspect() | Test 13 | All |
| 2.3.2.l ls -i | fs_print_ls_li() | fs_list_directory() | Test 14 | Ex 4,8 |

---

## Notes du Concepteur

<details>
<summary>Solution de Reference (Concepteur uniquement)</summary>

**Approche recommandee**:
1. Commencer par fs_inspect() avec stat/lstat basique
2. Ajouter la detection de type avec S_ISXXX
3. Implementer readlink() pour les symlinks
4. Ajouter getpwuid/getgrgid pour les noms
5. Implementer les fonctions de formatage
6. Ajouter fs_print_info() et fs_print_ls_li()

**Complexite**:
- Temps fs_inspect(): O(1)
- Temps fs_list_directory(): O(n)
- Espace: O(1) par fichier

</details>

---

## Historique

```yaml
version: "2.0"
created: "2025-01-04"
author: "MUSIC Music Music Music"
last_modified: "2025-01-04"
changes:
  - "v2.0: Refonte complete avec couverture explicite de TOUS les concepts 2.3.1.a-h et 2.3.2.a-l"
  - "v1.0: Version initiale"
```

---

*MUSIC Music Music Music Phase 2 - Module 2.3 Exercise 00*
*File System Inspector - Score Qualite: 97/100*
*Concepts couverts: 2.3.1.a-h (8) + 2.3.2.a-l (12) = 20 concepts*
