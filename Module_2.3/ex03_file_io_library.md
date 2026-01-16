# [Module 2.3] - Exercise 03: File I/O Library

## Metadonnees

```yaml
module: "2.3 - File Systems"
exercise: "ex03"
title: "File I/O Library"
difficulty: intermediaire
estimated_time: "6 heures"
prerequisite_exercises: ["ex00", "ex01", "ex02"]
concepts_requis: ["file descriptors", "system calls", "memory management", "error handling"]
concepts_couverts: ["2.3.5 File Descriptors", "2.3.6 File Operations"]
score_qualite: 97
```

---

## Concepts Couverts

Liste exhaustive des concepts abordes dans cet exercice avec references au curriculum:

### Section 2.3.5 - File Descriptors

| Ref | Concept | Description |
|-----|---------|-------------|
| **2.3.5.a** | File descriptor: Integer handle | Un file descriptor est un entier non-negatif qui sert de handle pour acceder a un fichier ouvert |
| **2.3.5.b** | Per-process table: fd -> file table entry | Chaque processus possede sa propre table de descripteurs qui mappe les fd vers les entrees de la table de fichiers systeme |
| **2.3.5.c** | System file table: Open file entries | Table globale du noyau contenant toutes les entrees de fichiers ouverts dans le systeme |
| **2.3.5.d** | Inode table: In-memory inodes | Table en memoire contenant les inodes des fichiers actuellement ouverts |
| **2.3.5.e** | Standard fds: 0=stdin, 1=stdout, 2=stderr | Les trois descripteurs standards herites par tout processus Unix |
| **2.3.5.f** | File table entry: Offset, flags, ref count | Chaque entree contient la position courante, les flags d'ouverture et un compteur de references |
| **2.3.5.g** | Sharing: fork() shares entries | Apres fork(), parent et enfant partagent les memes entrees de la table de fichiers |
| **2.3.5.h** | dup(): Duplicate fd | Duplique un fd vers le plus petit fd disponible |
| **2.3.5.i** | dup2(): Duplicate to specific fd | Duplique un fd vers un fd specifique, fermant ce dernier si necessaire |
| **2.3.5.j** | fcntl(): Manipulate fd | Interface polyvalente pour manipuler les proprietes d'un file descriptor |
| **2.3.5.k** | FD_CLOEXEC: Close on exec | Flag indiquant que le fd doit etre ferme automatiquement lors d'un exec() |

### Section 2.3.6 - File Operations

| Ref | Concept | Description |
|-----|---------|-------------|
| **2.3.6.a** | open(): Open/create file | Syscall principal pour ouvrir ou creer un fichier, retourne un fd |
| **2.3.6.b** | Open flags: O_RDONLY, O_WRONLY, O_RDWR | Flags obligatoires specifiant le mode d'acces (lecture, ecriture, les deux) |
| **2.3.6.c** | O_CREAT: Create if not exists | Flag pour creer le fichier s'il n'existe pas (necessite le mode en 3eme argument) |
| **2.3.6.d** | O_TRUNC: Truncate to zero | Flag pour tronquer le fichier a zero octet s'il existe |
| **2.3.6.e** | O_APPEND: Append mode | Flag pour ecrire toujours en fin de fichier (atomique) |
| **2.3.6.f** | O_EXCL: Fail if exists | Flag utilisable avec O_CREAT pour echouer si le fichier existe deja |
| **2.3.6.g** | read(): Read bytes | Lit des octets depuis un fd vers un buffer |
| **2.3.6.h** | write(): Write bytes | Ecrit des octets depuis un buffer vers un fd |
| **2.3.6.i** | lseek(): Change offset | Modifie la position courante dans le fichier |
| **2.3.6.j** | close(): Close fd | Ferme un file descriptor et libere les ressources associees |
| **2.3.6.k** | fsync(): Flush to disk | Force l'ecriture des donnees en cache vers le disque physique |
| **2.3.6.l** | ftruncate(): Set size | Redimensionne un fichier a une taille specifiee |

### Objectifs Pedagogiques

A la fin de cet exercice, vous devriez etre capable de:

1. Comprendre l'architecture en trois niveaux des tables de fichiers du noyau (per-process, system file table, inode table)
2. Maitriser tous les flags d'ouverture de fichiers et leurs combinaisons
3. Implementer des redirections d'E/S avec dup() et dup2()
4. Manipuler les proprietes de fichiers avec fcntl()
5. Gerer correctement le cycle de vie complet d'un file descriptor
6. Implementer un systeme de buffering pour optimiser les I/O
7. Comprendre les implications du partage de file descriptors apres fork()

---

## Contexte Theorique

### Architecture des File Descriptors Unix

Le systeme Unix utilise une architecture en trois niveaux pour gerer les fichiers ouverts:

```
    PROCESSUS A          PROCESSUS B              NOYAU
    ===========          ===========          =============

  Per-Process          Per-Process          System File Table
  FD Table             FD Table             +----------------+
  +--------+           +--------+           | Entry 0        |
  | fd 0 --|-----------|--------|---------->| offset: 0      |
  | fd 1 --|---+       | fd 0 --|---+       | flags: O_RDONLY|
  | fd 2 --|---|-------|--------|---|------>| refcount: 2    |-+
  | fd 3 --|---|--+    | fd 1 --|---+       +----------------+ |
  +--------+   |  |    +--------+           | Entry 1        | |   Inode Table
               |  |                         | offset: 1024   | |   +-----------+
               |  +------------------------>| flags: O_RDWR  |-|-->| inode 42  |
               |                            | refcount: 1    | |   | type: REG |
               +--------------------------->+----------------+ |   | size: 8192|
                                            | Entry 2        | |   | blocks:[] |
                                            | offset: 0      |<+   +-----------+
                                            | flags: O_RDONLY|     | inode 77  |
                                            | refcount: 1    |---->| type: REG |
                                            +----------------+     +-----------+
```

**2.3.5.a - File descriptor: Integer handle**

Un file descriptor est simplement un entier non-negatif retourne par `open()`. C'est un index dans la table per-process du processus courant. Le noyau utilise cet entier pour acceder rapidement a toutes les informations necessaires pour les operations d'I/O.

```c
int fd = open("/etc/passwd", O_RDONLY);  // fd pourrait etre 3
// fd est juste un index: process_fd_table[3] -> file_table_entry
```

**2.3.5.b - Per-process table: fd -> file table entry**

Chaque processus possede sa propre table de file descriptors. Cette table est un simple tableau ou l'index est le fd et la valeur est un pointeur vers une entree de la table de fichiers systeme. La taille de cette table est limitee (configurable via `ulimit -n`).

```c
// Visualisation conceptuelle
struct process {
    struct file_table_entry* fd_table[MAX_FDS];  // Per-process table
    // fd_table[0] -> stdin entry
    // fd_table[1] -> stdout entry
    // fd_table[2] -> stderr entry
    // fd_table[3] -> notre fichier ouvert
};
```

**2.3.5.c - System file table: Open file entries**

La table de fichiers systeme est globale au noyau. Chaque entree represente une instance unique d'ouverture de fichier. Plusieurs processus peuvent pointer vers la meme entree (apres fork), ou un meme processus peut avoir plusieurs entrees pour le meme fichier (plusieurs open()).

**2.3.5.d - Inode table: In-memory inodes**

Les inodes en memoire contiennent les metadonnees du fichier (taille, permissions, timestamps) et les pointeurs vers les blocs de donnees sur disque. Plusieurs entrees de la table de fichiers peuvent pointer vers le meme inode.

**2.3.5.e - Standard fds: 0=stdin, 1=stdout, 2=stderr**

Tout processus Unix herite de trois file descriptors ouverts:
- **fd 0 (STDIN_FILENO)**: Entree standard, generalement le clavier ou un pipe
- **fd 1 (STDOUT_FILENO)**: Sortie standard, generalement le terminal
- **fd 2 (STDERR_FILENO)**: Erreur standard, generalement le terminal

```c
#include <unistd.h>

// Ces constantes sont definies dans <unistd.h>
// #define STDIN_FILENO  0
// #define STDOUT_FILENO 1
// #define STDERR_FILENO 2

write(STDOUT_FILENO, "Hello\n", 6);  // Ecrit sur stdout
write(STDERR_FILENO, "Error!\n", 7); // Ecrit sur stderr
```

**2.3.5.f - File table entry: Offset, flags, ref count**

Chaque entree de la table de fichiers contient:
- **offset**: Position courante pour les prochaines lectures/ecritures
- **flags**: Flags d'ouverture (O_RDONLY, O_APPEND, etc.)
- **refcount**: Nombre de fd pointant vers cette entree

```c
// Conceptuellement:
struct file_table_entry {
    off_t offset;           // Position courante dans le fichier
    int flags;              // O_RDONLY, O_WRONLY, O_RDWR, O_APPEND...
    int refcount;           // Nombre de references
    struct inode* inode;    // Pointeur vers l'inode
};
```

**2.3.5.g - Sharing: fork() shares entries**

Apres un `fork()`, le processus enfant herite d'une copie de la table per-process du parent. Les deux tables pointent vers les **memes** entrees de la table de fichiers systeme. Consequence: si le parent avance l'offset avec read(), l'enfant voit le nouvel offset!

```c
int fd = open("file.txt", O_RDONLY);
pid_t pid = fork();

if (pid == 0) {
    // Enfant: fd pointe vers la MEME entree que le parent
    char buf[10];
    read(fd, buf, 10);  // Lit 10 octets, avance l'offset de 10
}
else {
    sleep(1);
    char buf[10];
    read(fd, buf, 10);  // Lit a partir de l'offset 10, PAS 0!
}
```

**2.3.5.h - dup(): Duplicate fd**

`dup()` cree un nouveau file descriptor qui pointe vers la meme entree de la table de fichiers. Le nouveau fd est le plus petit fd disponible. Les deux fd partagent offset et flags.

```c
int fd = open("file.txt", O_RDWR);  // fd = 3
int fd2 = dup(fd);                   // fd2 = 4 (plus petit dispo)
// fd et fd2 partagent le meme offset!
```

**2.3.5.i - dup2(): Duplicate to specific fd**

`dup2()` duplique vers un fd specifique. Si le fd cible est deja ouvert, il est d'abord ferme atomiquement.

```c
int fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
dup2(fd, STDOUT_FILENO);  // Redirige stdout vers le fichier
close(fd);                 // fd n'est plus necessaire
printf("This goes to file\n");  // Ecrit dans output.txt!
```

**2.3.5.j - fcntl(): Manipulate fd**

`fcntl()` est une interface polyvalente pour manipuler les proprietes d'un fd:

```c
#include <fcntl.h>

int fd = open("file.txt", O_RDONLY);

// Obtenir les flags
int flags = fcntl(fd, F_GETFL);

// Modifier les flags (ajouter O_APPEND)
fcntl(fd, F_SETFL, flags | O_APPEND);

// Dupliquer (equivalent a dup)
int fd2 = fcntl(fd, F_DUPFD, 0);

// Obtenir/modifier les flags du fd (pas du fichier)
int fd_flags = fcntl(fd, F_GETFD);
fcntl(fd, F_SETFD, fd_flags | FD_CLOEXEC);
```

**2.3.5.k - FD_CLOEXEC: Close on exec**

Le flag `FD_CLOEXEC` indique que le fd doit etre automatiquement ferme lors d'un appel a `exec()`. Ceci evite les fuites de descripteurs vers les programmes executes.

```c
int fd = open("secret.txt", O_RDONLY);
fcntl(fd, F_SETFD, FD_CLOEXEC);  // Ferme auto sur exec

// Ou directement a l'ouverture avec O_CLOEXEC
int fd2 = open("secret.txt", O_RDONLY | O_CLOEXEC);
```

### Operations sur les Fichiers

**2.3.6.a - open(): Open/create file**

```c
#include <fcntl.h>
#include <sys/stat.h>

// Prototype
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);

// Le mode est requis si O_CREAT est utilise
int fd = open("/tmp/new.txt", O_WRONLY | O_CREAT, 0644);
```

**2.3.6.b - Open flags: O_RDONLY, O_WRONLY, O_RDWR**

Un et un seul de ces flags doit etre specifie:
- `O_RDONLY`: Lecture seule
- `O_WRONLY`: Ecriture seule
- `O_RDWR`: Lecture et ecriture

```c
int fd_read = open("file.txt", O_RDONLY);
int fd_write = open("file.txt", O_WRONLY);
int fd_both = open("file.txt", O_RDWR);
```

**2.3.6.c - O_CREAT: Create if not exists**

Cree le fichier s'il n'existe pas. Necessite le 3eme argument `mode` pour specifier les permissions.

```c
// Cree file.txt avec permissions 0644 s'il n'existe pas
int fd = open("file.txt", O_WRONLY | O_CREAT, 0644);
```

**2.3.6.d - O_TRUNC: Truncate to zero**

Tronque le fichier a zero octet s'il existe et est ouvert en ecriture.

```c
// Ecrase le contenu existant
int fd = open("file.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
```

**2.3.6.e - O_APPEND: Append mode**

Force toutes les ecritures en fin de fichier. L'offset est positionne automatiquement avant chaque write(), de maniere atomique.

```c
int fd = open("log.txt", O_WRONLY | O_APPEND);
write(fd, "New entry\n", 10);  // Toujours a la fin
```

**2.3.6.f - O_EXCL: Fail if exists**

Utilise avec O_CREAT, fait echouer open() si le fichier existe deja. Utile pour creer des fichiers de verrouillage.

```c
int fd = open("lockfile", O_WRONLY | O_CREAT | O_EXCL, 0644);
if (fd == -1 && errno == EEXIST) {
    // Le fichier existe deja - quelqu'un d'autre a le verrou
}
```

**2.3.6.g - read(): Read bytes**

```c
ssize_t read(int fd, void *buf, size_t count);

char buffer[1024];
ssize_t n = read(fd, buffer, sizeof(buffer));
// n = nombre d'octets lus (peut etre < count)
// n = 0 signifie EOF
// n = -1 signifie erreur (verifier errno)
```

**2.3.6.h - write(): Write bytes**

```c
ssize_t write(int fd, const void *buf, size_t count);

const char *msg = "Hello, World!\n";
ssize_t n = write(fd, msg, strlen(msg));
// n = nombre d'octets ecrits (peut etre < count)
// n = -1 signifie erreur
```

**2.3.6.i - lseek(): Change offset**

```c
off_t lseek(int fd, off_t offset, int whence);

// whence:
// SEEK_SET: offset depuis le debut
// SEEK_CUR: offset depuis la position courante
// SEEK_END: offset depuis la fin

lseek(fd, 0, SEEK_SET);    // Retour au debut
lseek(fd, 100, SEEK_CUR);  // Avancer de 100 octets
lseek(fd, -50, SEEK_END);  // 50 octets avant la fin
off_t pos = lseek(fd, 0, SEEK_CUR);  // Position courante
```

**2.3.6.j - close(): Close fd**

```c
int close(int fd);

// Libere le fd pour reutilisation
// Decremente le refcount de l'entree file table
// Si refcount = 0, libere l'entree
close(fd);
```

**2.3.6.k - fsync(): Flush to disk**

Force l'ecriture des donnees bufferisees vers le disque physique. Essentiel pour la durabilite des donnees.

```c
write(fd, data, len);  // Peut rester en cache noyau
fsync(fd);             // Force l'ecriture sur disque
```

**2.3.6.l - ftruncate(): Set size**

Redimensionne un fichier ouvert a une taille specifique.

```c
int ftruncate(int fd, off_t length);

ftruncate(fd, 0);      // Vide le fichier
ftruncate(fd, 1024);   // Fixe la taille a 1024 octets
// Si agrandi, les nouveaux octets sont a zero
```

---

## Enonce

### Vue d'Ensemble

Vous devez implementer une **bibliotheque d'I/O de fichiers complete** (`libfio`) qui encapsule les operations bas niveau du systeme avec une API propre, gerant le buffering, les redirections, et les diagnostics sur l'etat des file descriptors. Cette bibliotheque doit exposer de maniere pedagogique l'architecture des tables de fichiers du noyau.

### Specifications Fonctionnelles

#### Partie 1: Structure de Donnees FIO

Implementez une structure opaque representant un flux de fichier avec buffering.

```c
// fio.h

#ifndef FIO_H
#define FIO_H

#include <stddef.h>
#include <sys/types.h>

// Taille du buffer interne
#define FIO_BUFSIZ 4096

// Modes de buffering
typedef enum {
    FIO_UNBUFFERED,     // Pas de buffer (comme stderr)
    FIO_LINE_BUFFERED,  // Flush a chaque newline (comme stdin/stdout sur terminal)
    FIO_FULLY_BUFFERED  // Flush quand buffer plein (fichiers reguliers)
} fio_buffer_mode_t;

// Etats du flux
typedef enum {
    FIO_STATE_OK,
    FIO_STATE_EOF,
    FIO_STATE_ERROR
} fio_state_t;

// Structure principale (opaque pour l'utilisateur)
typedef struct fio_stream fio_t;

// Informations sur un file descriptor (pour diagnostic)
typedef struct {
    int fd;                     // 2.3.5.a: Integer handle
    int flags;                  // 2.3.5.f: Flags d'ouverture
    off_t offset;               // 2.3.5.f: Position courante
    int fd_flags;               // 2.3.5.k: FD_CLOEXEC etc.
    int is_valid;               // fd valide?
    int is_pipe;                // Est-ce un pipe?
    int is_socket;              // Est-ce un socket?
    int is_terminal;            // Est-ce un terminal?
    int is_regular;             // Est-ce un fichier regulier?
    ino_t inode;                // 2.3.5.d: Numero d'inode
    dev_t device;               // Device contenant le fichier
} fio_fd_info_t;

// Informations sur les tables systeme (pour diagnostic)
typedef struct {
    int process_fd_count;       // 2.3.5.b: Nombre de fd ouverts
    int max_fd;                 // Plus grand fd ouvert
    int available_fds;          // fd disponibles
    int stdin_valid;            // 2.3.5.e: stdin valide?
    int stdout_valid;           // 2.3.5.e: stdout valide?
    int stderr_valid;           // 2.3.5.e: stderr valide?
} fio_process_info_t;

// ============================================
// PARTIE 1: Ouverture et Fermeture
// ============================================

// Ouvre un fichier avec le mode specifie
// mode: "r", "w", "a", "r+", "w+", "a+"
// Utilise: 2.3.6.a open(), 2.3.6.b flags, 2.3.6.c O_CREAT,
//          2.3.6.d O_TRUNC, 2.3.6.e O_APPEND
fio_t* fio_open(const char* path, const char* mode);

// Ouvre avec flags explicites (pour acces aux flags avances)
// Utilise: 2.3.6.f O_EXCL, 2.3.5.k O_CLOEXEC
fio_t* fio_open_flags(const char* path, int flags, mode_t permissions);

// Ferme le flux et libere les ressources
// Utilise: 2.3.6.j close()
int fio_close(fio_t* stream);

// Cree un flux a partir d'un fd existant
fio_t* fio_from_fd(int fd, const char* mode);

// ============================================
// PARTIE 2: Lecture et Ecriture
// ============================================

// Lit des octets dans le buffer
// Utilise: 2.3.6.g read()
ssize_t fio_read(fio_t* stream, void* buf, size_t count);

// Ecrit des octets depuis le buffer
// Utilise: 2.3.6.h write()
ssize_t fio_write(fio_t* stream, const void* buf, size_t count);

// Lit une ligne (jusqu'a '\n' inclus ou EOF)
ssize_t fio_readline(fio_t* stream, char* buf, size_t maxlen);

// Ecrit une chaine formatee (comme fprintf)
int fio_printf(fio_t* stream, const char* format, ...);

// Lit un caractere
int fio_getc(fio_t* stream);

// Ecrit un caractere
int fio_putc(fio_t* stream, int c);

// ============================================
// PARTIE 3: Positionnement
// ============================================

// Change la position courante
// Utilise: 2.3.6.i lseek()
off_t fio_seek(fio_t* stream, off_t offset, int whence);

// Retourne la position courante
off_t fio_tell(fio_t* stream);

// Retour au debut
int fio_rewind(fio_t* stream);

// ============================================
// PARTIE 4: Controle du Buffer
// ============================================

// Force l'ecriture du buffer interne
// Puis appelle 2.3.6.k fsync() si sync_disk est vrai
int fio_flush(fio_t* stream, int sync_disk);

// Change le mode de buffering
int fio_setbuf(fio_t* stream, fio_buffer_mode_t mode);

// Redimensionne le fichier
// Utilise: 2.3.6.l ftruncate()
int fio_truncate(fio_t* stream, off_t length);

// ============================================
// PARTIE 5: Duplication et Redirection
// ============================================

// Duplique le fd sous-jacent
// Utilise: 2.3.5.h dup()
fio_t* fio_dup(fio_t* stream);

// Redirige le flux vers un autre fd
// Utilise: 2.3.5.i dup2()
int fio_redirect(fio_t* stream, int target_fd);

// Sauvegarde un fd standard et le restaure plus tard
// (pour redirections temporaires)
typedef struct {
    int original_fd;
    int saved_fd;
} fio_redirect_state_t;

fio_redirect_state_t fio_save_fd(int fd);
int fio_restore_fd(fio_redirect_state_t* state);

// ============================================
// PARTIE 6: Manipulation avancee (fcntl)
// ============================================

// Obtient les flags du fichier
// Utilise: 2.3.5.j fcntl(F_GETFL)
int fio_get_flags(fio_t* stream);

// Modifie les flags du fichier
// Utilise: 2.3.5.j fcntl(F_SETFL)
int fio_set_flags(fio_t* stream, int flags);

// Definit close-on-exec
// Utilise: 2.3.5.k FD_CLOEXEC via fcntl(F_SETFD)
int fio_set_cloexec(fio_t* stream, int enable);

// Obtient le fd brut (utiliser avec precaution)
int fio_fileno(fio_t* stream);

// ============================================
// PARTIE 7: Diagnostics
// ============================================

// Informations sur le fd sous-jacent
// Explore: 2.3.5.a, 2.3.5.d, 2.3.5.f
int fio_get_fd_info(fio_t* stream, fio_fd_info_t* info);

// Informations sur tous les fd du processus
// Explore: 2.3.5.b per-process table, 2.3.5.e standard fds
int fio_get_process_info(fio_process_info_t* info);

// Affiche un dump diagnostic du flux
void fio_debug_dump(fio_t* stream);

// Etat du flux (OK, EOF, ERROR)
fio_state_t fio_state(fio_t* stream);

// Verifie si EOF atteint
int fio_eof(fio_t* stream);

// Verifie si erreur survenue
int fio_error(fio_t* stream);

// Efface les flags d'erreur
void fio_clearerr(fio_t* stream);

// ============================================
// PARTIE 8: Operations atomiques et fork()
// ============================================

// Verrou en lecture (bloquant)
int fio_lock_read(fio_t* stream);

// Verrou en ecriture (bloquant)
int fio_lock_write(fio_t* stream);

// Deverrouille
int fio_unlock(fio_t* stream);

// Prepare le flux pour fork()
// Note: 2.3.5.g - apres fork(), les flux sont partages!
// Cette fonction documente ce comportement
void fio_prefork(fio_t* stream);

// A appeler dans le parent apres fork()
void fio_postfork_parent(fio_t* stream);

// A appeler dans l'enfant apres fork()
void fio_postfork_child(fio_t* stream);

#endif // FIO_H
```

#### Partie 2: Implementation de la Structure Interne

```c
// fio_internal.h (non expose aux utilisateurs)

#ifndef FIO_INTERNAL_H
#define FIO_INTERNAL_H

#include "fio.h"
#include <pthread.h>

struct fio_stream {
    // File descriptor sous-jacent
    int fd;

    // Flags et mode
    int open_flags;         // Flags passes a open()
    int readable;           // Peut lire?
    int writable;           // Peut ecrire?

    // Buffer de lecture
    char read_buf[FIO_BUFSIZ];
    size_t read_buf_pos;    // Position de lecture dans le buffer
    size_t read_buf_len;    // Donnees valides dans le buffer

    // Buffer d'ecriture
    char write_buf[FIO_BUFSIZ];
    size_t write_buf_len;   // Donnees a ecrire

    // Mode de buffering
    fio_buffer_mode_t buf_mode;

    // Etat
    fio_state_t state;
    int eof_flag;
    int error_flag;
    int last_errno;

    // Thread safety
    pthread_mutex_t lock;

    // Pour le diagnostic
    char path[256];         // Chemin d'origine
    ino_t inode_at_open;    // Inode lors de l'ouverture
};

// Fonctions internes
int _fio_fill_read_buffer(fio_t* stream);
int _fio_flush_write_buffer(fio_t* stream);
void _fio_lock(fio_t* stream);
void _fio_unlock(fio_t* stream);

#endif // FIO_INTERNAL_H
```

### Exemple d'Utilisation Attendue

```c
// example_usage.c

#include "fio.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

void demonstrate_fd_architecture(void) {
    printf("=== Demonstration de l'architecture des FD ===\n\n");

    // 2.3.5.e: Standard file descriptors
    printf("1. Standard FDs (2.3.5.e):\n");
    printf("   STDIN  = %d\n", STDIN_FILENO);
    printf("   STDOUT = %d\n", STDOUT_FILENO);
    printf("   STDERR = %d\n", STDERR_FILENO);

    // 2.3.5.b: Afficher l'etat de la table per-process
    fio_process_info_t pinfo;
    fio_get_process_info(&pinfo);
    printf("\n2. Per-process FD table (2.3.5.b):\n");
    printf("   FDs ouverts: %d\n", pinfo.process_fd_count);
    printf("   Max FD: %d\n", pinfo.max_fd);

    // 2.3.6.a, 2.3.6.b, 2.3.6.c, 2.3.6.d: Ouverture avec differents flags
    printf("\n3. Open flags (2.3.6.a-f):\n");

    fio_t* f1 = fio_open("test_readonly.txt", "r");
    if (f1) {
        printf("   O_RDONLY: fd=%d\n", fio_fileno(f1));
        fio_close(f1);
    }

    fio_t* f2 = fio_open("test_writetrunc.txt", "w");  // O_CREAT | O_TRUNC
    if (f2) {
        printf("   O_WRONLY|O_CREAT|O_TRUNC: fd=%d\n", fio_fileno(f2));
        fio_close(f2);
    }

    fio_t* f3 = fio_open("test_append.txt", "a");      // O_APPEND
    if (f3) {
        printf("   O_WRONLY|O_CREAT|O_APPEND: fd=%d\n", fio_fileno(f3));
        fio_close(f3);
    }

    // 2.3.6.f: O_EXCL
    fio_t* f4 = fio_open_flags("test_excl.txt",
                               O_WRONLY | O_CREAT | O_EXCL, 0644);
    if (f4) {
        printf("   O_EXCL (creation exclusive): fd=%d\n", fio_fileno(f4));
        fio_close(f4);
    }
}

void demonstrate_file_operations(void) {
    printf("\n=== Operations sur les fichiers ===\n\n");

    // Creer un fichier de test
    fio_t* f = fio_open("demo_file.txt", "w+");
    if (!f) {
        perror("fio_open");
        return;
    }

    // 2.3.6.h: write()
    printf("1. Write (2.3.6.h):\n");
    const char* data = "Hello, World!\nLine 2\nLine 3\n";
    ssize_t written = fio_write(f, data, strlen(data));
    printf("   Ecrit %zd octets\n", written);

    // 2.3.6.k: fsync()
    printf("\n2. Fsync (2.3.6.k):\n");
    fio_flush(f, 1);  // sync_disk = true
    printf("   Donnees synchronisees sur disque\n");

    // 2.3.6.i: lseek()
    printf("\n3. Lseek (2.3.6.i):\n");
    fio_rewind(f);  // SEEK_SET, 0
    printf("   Position ramenee au debut\n");

    // 2.3.6.g: read()
    printf("\n4. Read (2.3.6.g):\n");
    char buf[100];
    ssize_t n = fio_read(f, buf, sizeof(buf) - 1);
    buf[n] = '\0';
    printf("   Lu %zd octets: \"%s\"\n", n, buf);

    // 2.3.6.l: ftruncate()
    printf("\n5. Ftruncate (2.3.6.l):\n");
    fio_truncate(f, 5);
    printf("   Fichier tronque a 5 octets\n");

    // 2.3.6.j: close()
    printf("\n6. Close (2.3.6.j):\n");
    fio_close(f);
    printf("   Fichier ferme\n");
}

void demonstrate_fd_duplication(void) {
    printf("\n=== Duplication de FD ===\n\n");

    fio_t* original = fio_open("dup_test.txt", "w");
    if (!original) return;

    // 2.3.5.h: dup()
    printf("1. dup() (2.3.5.h):\n");
    fio_t* copy = fio_dup(original);
    printf("   Original fd=%d, Copy fd=%d\n",
           fio_fileno(original), fio_fileno(copy));

    // Ecrire via l'original
    fio_write(original, "Via original\n", 13);

    // Ecrire via la copie (meme fichier, meme offset!)
    // 2.3.5.f: Les deux partagent l'offset
    fio_write(copy, "Via copy\n", 9);

    printf("   Les deux ecritures sont sequentielles (offset partage)\n");

    // 2.3.5.i: dup2() - redirection
    printf("\n2. dup2() pour redirection (2.3.5.i):\n");
    fio_redirect_state_t saved = fio_save_fd(STDOUT_FILENO);

    fio_t* redirect_file = fio_open("redirected_output.txt", "w");
    fio_redirect(redirect_file, STDOUT_FILENO);

    printf("Cette ligne va dans le fichier!\n");

    fio_restore_fd(&saved);
    printf("   Stdout restaure, cette ligne est sur le terminal\n");

    fio_close(original);
    fio_close(copy);
    fio_close(redirect_file);
}

void demonstrate_fcntl_operations(void) {
    printf("\n=== Operations fcntl (2.3.5.j) ===\n\n");

    fio_t* f = fio_open("fcntl_test.txt", "w");
    if (!f) return;

    // 2.3.5.j: fcntl() pour obtenir les flags
    printf("1. F_GETFL:\n");
    int flags = fio_get_flags(f);
    printf("   Flags actuels: %d\n", flags);
    printf("   O_APPEND: %s\n", (flags & O_APPEND) ? "oui" : "non");

    // Ajouter O_APPEND
    printf("\n2. F_SETFL (ajout O_APPEND):\n");
    fio_set_flags(f, flags | O_APPEND);
    flags = fio_get_flags(f);
    printf("   O_APPEND apres modif: %s\n", (flags & O_APPEND) ? "oui" : "non");

    // 2.3.5.k: FD_CLOEXEC
    printf("\n3. FD_CLOEXEC (2.3.5.k):\n");
    fio_set_cloexec(f, 1);
    printf("   Close-on-exec active\n");

    fio_close(f);
}

void demonstrate_fork_sharing(void) {
    printf("\n=== Partage apres fork() (2.3.5.g) ===\n\n");

    fio_t* shared = fio_open("fork_shared.txt", "w+");
    if (!shared) return;

    fio_write(shared, "Initial content\n", 16);
    fio_rewind(shared);

    // Preparer pour fork
    fio_prefork(shared);

    pid_t pid = fork();

    if (pid == 0) {
        // Enfant
        fio_postfork_child(shared);

        // Lire 5 octets (avance l'offset partage!)
        char buf[10];
        fio_read(shared, buf, 5);
        buf[5] = '\0';

        // Note: cette ecriture sur stderr car stdout pourrait etre bufferise
        fprintf(stderr, "   Enfant: lu '%s', offset maintenant a 5\n", buf);

        fio_close(shared);
        _exit(0);
    } else {
        // Parent
        fio_postfork_parent(shared);

        wait(NULL);  // Attendre l'enfant

        // Le parent voit l'offset modifie par l'enfant!
        // 2.3.5.c: System file table partagee
        off_t pos = fio_tell(shared);
        printf("   Parent: offset = %ld (modifie par l'enfant!)\n", (long)pos);

        fio_close(shared);
    }
}

void demonstrate_fd_diagnostics(void) {
    printf("\n=== Diagnostics FD ===\n\n");

    fio_t* f = fio_open("diag_test.txt", "w+");
    if (!f) return;

    // 2.3.5.a, 2.3.5.d, 2.3.5.f: Informations detaillees
    fio_fd_info_t info;
    fio_get_fd_info(f, &info);

    printf("Informations sur le FD:\n");
    printf("  fd (2.3.5.a):     %d\n", info.fd);
    printf("  flags (2.3.5.f):  0x%x\n", info.flags);
    printf("  offset (2.3.5.f): %ld\n", (long)info.offset);
    printf("  inode (2.3.5.d):  %lu\n", (unsigned long)info.inode);
    printf("  is_terminal:      %s\n", info.is_terminal ? "yes" : "no");
    printf("  is_regular:       %s\n", info.is_regular ? "yes" : "no");

    // Dump complet
    printf("\nDump diagnostic:\n");
    fio_debug_dump(f);

    fio_close(f);
}

int main(void) {
    printf("============================================\n");
    printf("  DEMONSTRATION COMPLETE DE LA LIBFIO\n");
    printf("  Couvrant tous les concepts 2.3.5 et 2.3.6\n");
    printf("============================================\n");

    demonstrate_fd_architecture();
    demonstrate_file_operations();
    demonstrate_fd_duplication();
    demonstrate_fcntl_operations();
    demonstrate_fork_sharing();
    demonstrate_fd_diagnostics();

    printf("\n=== Fin de la demonstration ===\n");
    return 0;
}
```

---

## Fonctions Autorisees

### Syscalls de base
- `open`, `close`, `read`, `write`, `lseek`
- `dup`, `dup2`
- `fcntl`
- `fsync`, `fdatasync`
- `ftruncate`, `truncate`
- `fstat`, `stat`, `fstatat`
- `isatty`

### Gestion des processus
- `fork`, `wait`, `waitpid`
- `_exit`

### Gestion memoire
- `malloc`, `calloc`, `realloc`, `free`

### Manipulation de chaines
- `memset`, `memcpy`, `memmove`
- `strlen`, `strncpy`, `strcmp`
- `snprintf`, `vsnprintf`

### Threads (optionnel, pour bonus)
- `pthread_mutex_init`, `pthread_mutex_lock`, `pthread_mutex_unlock`, `pthread_mutex_destroy`

### Autres
- `perror`, `strerror`
- `errno`

### Fonctions interdites
- `fopen`, `fclose`, `fread`, `fwrite`, `fprintf`, `fseek` (sauf dans main() pour les tests)
- `printf` (sauf dans main() pour les tests - utiliser votre `fio_printf`)

---

## Contraintes

### Contraintes de Code

1. **Standard C17** - Compilation avec `-std=c17 -Wall -Wextra -Werror -pedantic`

2. **Pas de fuites de memoire** - Tout `malloc` doit avoir son `free` correspondant

3. **Gestion d'erreurs exhaustive** - Toute fonction systeme doit voir son retour verifie

4. **Thread-safety** - Les operations sur un meme `fio_t*` doivent etre protegees par mutex

5. **Pas de variables globales mutables** - Constantes globales permises uniquement

6. **Documentation** - Chaque fonction doit etre documentee avec les concepts couverts

### Contraintes de Design

1. **Encapsulation** - La structure `fio_stream` ne doit jamais etre exposee (opaque type)

2. **RAII** - Les ressources doivent etre acquises a l'initialisation et liberees a la destruction

3. **Fail-fast** - Les erreurs doivent etre detectees et signalees immediatement

4. **No undefined behavior** - Aucun comportement indefini meme avec des entrees invalides

### Contraintes de Performance

1. **Buffering efficace** - Le nombre d'appels systeme doit etre minimise

2. **Zero-copy quand possible** - Pour les gros transferts alignes sur le buffer

3. **Pas d'allocation excessive** - Un `fio_t` = une allocation + taille fixe des buffers

---

## Tests Moulinette

La moulinette Rust 2024 executera les tests suivants:

### Test 1: Ouverture et Flags (2.3.6.a-f)

```rust
// test_open_flags.rs

#[test]
fn test_open_readonly() {
    // Creer un fichier de test
    std::fs::write("test_ro.txt", "content").unwrap();

    let result = run_c_test(r#"
        #include "fio.h"
        #include <fcntl.h>

        int main(void) {
            // 2.3.6.b: O_RDONLY
            fio_t* f = fio_open("test_ro.txt", "r");
            if (!f) return 1;

            int flags = fio_get_flags(f);
            // O_RDONLY est 0, donc on verifie qu'on ne peut pas ecrire
            if (fio_write(f, "x", 1) != -1) return 2;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "O_RDONLY devrait empecher l'ecriture");
}

#[test]
fn test_open_create_trunc() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            // 2.3.6.c: O_CREAT, 2.3.6.d: O_TRUNC
            fio_t* f = fio_open("new_file.txt", "w");
            if (!f) return 1;

            fio_write(f, "initial", 7);
            fio_close(f);

            // Reouvrir en "w" doit tronquer
            f = fio_open("new_file.txt", "w");
            fio_close(f);

            // Verifier que le fichier est vide
            f = fio_open("new_file.txt", "r");
            char buf[10];
            ssize_t n = fio_read(f, buf, 10);
            fio_close(f);

            return (n == 0) ? 0 : 3;
        }
    "#);

    assert_eq!(result.exit_code, 0, "O_TRUNC devrait vider le fichier");
}

#[test]
fn test_open_append() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            // Creer fichier avec contenu initial
            fio_t* f = fio_open("append_test.txt", "w");
            fio_write(f, "ABC", 3);
            fio_close(f);

            // 2.3.6.e: O_APPEND
            f = fio_open("append_test.txt", "a");
            fio_write(f, "DEF", 3);
            fio_close(f);

            // Verifier
            f = fio_open("append_test.txt", "r");
            char buf[10];
            ssize_t n = fio_read(f, buf, 10);
            buf[n] = '\0';
            fio_close(f);

            if (strcmp(buf, "ABCDEF") != 0) return 1;
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "O_APPEND devrait ecrire a la fin");
}

#[test]
fn test_open_excl() {
    std::fs::write("exists.txt", "").unwrap();

    let result = run_c_test(r#"
        #include "fio.h"
        #include <fcntl.h>
        #include <errno.h>

        int main(void) {
            // 2.3.6.f: O_EXCL
            fio_t* f = fio_open_flags("exists.txt",
                                      O_WRONLY | O_CREAT | O_EXCL, 0644);
            if (f != NULL) return 1;  // Devrait echouer
            if (errno != EEXIST) return 2;

            // Sur nouveau fichier, doit reussir
            f = fio_open_flags("new_excl.txt",
                               O_WRONLY | O_CREAT | O_EXCL, 0644);
            if (!f) return 3;
            fio_close(f);

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "O_EXCL devrait echouer si fichier existe");
}
```

### Test 2: Read/Write/Seek (2.3.6.g-i)

```rust
#[test]
fn test_read_write() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <string.h>

        int main(void) {
            fio_t* f = fio_open("rw_test.txt", "w+");
            if (!f) return 1;

            // 2.3.6.h: write()
            const char* data = "Hello, World!";
            ssize_t written = fio_write(f, data, strlen(data));
            if (written != (ssize_t)strlen(data)) return 2;

            // 2.3.6.i: lseek() via rewind
            fio_rewind(f);

            // 2.3.6.g: read()
            char buf[50];
            ssize_t n = fio_read(f, buf, 50);
            buf[n] = '\0';

            if (strcmp(buf, data) != 0) return 3;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Read/Write basique");
}

#[test]
fn test_lseek_operations() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <unistd.h>

        int main(void) {
            fio_t* f = fio_open("seek_test.txt", "w+");
            fio_write(f, "0123456789", 10);

            // 2.3.6.i: lseek avec differents whence

            // SEEK_SET
            if (fio_seek(f, 5, SEEK_SET) != 5) return 1;
            if (fio_tell(f) != 5) return 2;

            // SEEK_CUR
            if (fio_seek(f, 2, SEEK_CUR) != 7) return 3;

            // SEEK_END
            if (fio_seek(f, -3, SEEK_END) != 7) return 4;

            // Lire depuis position 7
            char c;
            fio_read(f, &c, 1);
            if (c != '7') return 5;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "lseek() avec SEEK_SET/CUR/END");
}
```

### Test 3: Close et Truncate (2.3.6.j-l)

```rust
#[test]
fn test_close() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <fcntl.h>

        int main(void) {
            fio_t* f = fio_open("close_test.txt", "w");
            int fd = fio_fileno(f);

            // 2.3.6.j: close()
            if (fio_close(f) != 0) return 1;

            // Le fd ne devrait plus etre valide
            if (fcntl(fd, F_GETFL) != -1) return 2;

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "close() devrait liberer le fd");
}

#[test]
fn test_fsync() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            fio_t* f = fio_open("fsync_test.txt", "w");
            fio_write(f, "data to sync", 12);

            // 2.3.6.k: fsync()
            if (fio_flush(f, 1) != 0) return 1;  // sync_disk = true

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "fsync() devrait reussir");
}

#[test]
fn test_ftruncate() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <sys/stat.h>

        int main(void) {
            fio_t* f = fio_open("trunc_test.txt", "w+");
            fio_write(f, "0123456789", 10);

            // 2.3.6.l: ftruncate()
            if (fio_truncate(f, 5) != 0) return 1;

            // Verifier la taille
            struct stat st;
            fstat(fio_fileno(f), &st);
            if (st.st_size != 5) return 2;

            // Agrandir (nouveaux octets a zero)
            if (fio_truncate(f, 10) != 0) return 3;

            fio_seek(f, 5, SEEK_SET);
            char buf[5];
            fio_read(f, buf, 5);

            // Les octets 5-9 doivent etre nuls
            for (int i = 0; i < 5; i++) {
                if (buf[i] != 0) return 4;
            }

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "ftruncate() devrait redimensionner");
}
```

### Test 4: File Descriptors (2.3.5.a-f)

```rust
#[test]
fn test_fd_is_integer() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            fio_t* f = fio_open("fd_test.txt", "w");

            // 2.3.5.a: fd est un entier >= 0
            int fd = fio_fileno(f);
            if (fd < 0) return 1;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "fd doit etre un entier >= 0");
}

#[test]
fn test_per_process_table() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            // 2.3.5.b: per-process fd table
            fio_process_info_t info1, info2;

            fio_get_process_info(&info1);

            fio_t* f = fio_open("pp_test.txt", "w");

            fio_get_process_info(&info2);

            // Le nombre de fd ouverts doit avoir augmente
            if (info2.process_fd_count <= info1.process_fd_count) return 1;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Per-process table doit tracker les fds");
}

#[test]
fn test_standard_fds() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <unistd.h>

        int main(void) {
            // 2.3.5.e: Standard file descriptors
            fio_process_info_t info;
            fio_get_process_info(&info);

            if (!info.stdin_valid) return 1;
            if (!info.stdout_valid) return 2;
            if (!info.stderr_valid) return 3;

            // Verifier les numeros
            if (STDIN_FILENO != 0) return 4;
            if (STDOUT_FILENO != 1) return 5;
            if (STDERR_FILENO != 2) return 6;

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "stdin=0, stdout=1, stderr=2");
}

#[test]
fn test_file_table_entry_info() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <fcntl.h>

        int main(void) {
            fio_t* f = fio_open("fte_test.txt", "w+");
            fio_write(f, "test", 4);

            // 2.3.5.f: File table entry contient offset et flags
            fio_fd_info_t info;
            fio_get_fd_info(f, &info);

            // Offset apres ecriture de 4 octets
            if (info.offset != 4) return 1;

            // Flags doivent inclure O_RDWR
            if (!(info.flags & O_RDWR)) return 2;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "File table entry: offset et flags");
}

#[test]
fn test_inode_table() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            std::fs::write("inode_test.txt", "x").unwrap();

            fio_t* f1 = fio_open("inode_test.txt", "r");
            fio_t* f2 = fio_open("inode_test.txt", "r");

            // 2.3.5.d: Les deux fd pointent vers le meme inode
            fio_fd_info_t info1, info2;
            fio_get_fd_info(f1, &info1);
            fio_get_fd_info(f2, &info2);

            if (info1.inode != info2.inode) return 1;

            fio_close(f1);
            fio_close(f2);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Meme inode pour meme fichier");
}
```

### Test 5: dup et dup2 (2.3.5.h-i)

```rust
#[test]
fn test_dup() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            fio_t* original = fio_open("dup_test.txt", "w+");

            // 2.3.5.h: dup()
            fio_t* copy = fio_dup(original);
            if (!copy) return 1;

            // Les fd doivent etre differents
            if (fio_fileno(original) == fio_fileno(copy)) return 2;

            // Mais partager l'offset (2.3.5.f: shared entry)
            fio_write(original, "AAAA", 4);

            fio_fd_info_t info;
            fio_get_fd_info(copy, &info);
            if (info.offset != 4) return 3;  // Offset partage!

            fio_close(original);
            fio_close(copy);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "dup() partage l'offset");
}

#[test]
fn test_dup2_redirect() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <unistd.h>
        #include <string.h>

        int main(void) {
            // Sauvegarder stdout
            fio_redirect_state_t saved = fio_save_fd(STDOUT_FILENO);

            // 2.3.5.i: dup2() pour redirection
            fio_t* f = fio_open("redirect_out.txt", "w");
            fio_redirect(f, STDOUT_FILENO);

            // printf va maintenant dans le fichier
            write(STDOUT_FILENO, "redirected\n", 11);

            // Restaurer
            fio_restore_fd(&saved);

            // Verifier le contenu du fichier
            fio_close(f);
            f = fio_open("redirect_out.txt", "r");
            char buf[20];
            ssize_t n = fio_read(f, buf, 20);
            buf[n] = '\0';
            fio_close(f);

            if (strcmp(buf, "redirected\n") != 0) return 1;

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "dup2() redirige correctement");
}
```

### Test 6: fcntl et FD_CLOEXEC (2.3.5.j-k)

```rust
#[test]
fn test_fcntl_get_set_flags() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <fcntl.h>

        int main(void) {
            fio_t* f = fio_open("fcntl_test.txt", "w");

            // 2.3.5.j: fcntl() pour manipuler les flags
            int flags = fio_get_flags(f);

            // Ajouter O_APPEND
            if (fio_set_flags(f, flags | O_APPEND) != 0) return 1;

            // Verifier
            flags = fio_get_flags(f);
            if (!(flags & O_APPEND)) return 2;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "fcntl() modifie les flags");
}

#[test]
fn test_fd_cloexec() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <fcntl.h>

        int main(void) {
            fio_t* f = fio_open("cloexec_test.txt", "w");

            // 2.3.5.k: FD_CLOEXEC
            if (fio_set_cloexec(f, 1) != 0) return 1;

            // Verifier via fcntl directement
            int fd_flags = fcntl(fio_fileno(f), F_GETFD);
            if (!(fd_flags & FD_CLOEXEC)) return 2;

            // Desactiver
            if (fio_set_cloexec(f, 0) != 0) return 3;
            fd_flags = fcntl(fio_fileno(f), F_GETFD);
            if (fd_flags & FD_CLOEXEC) return 4;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "FD_CLOEXEC peut etre active/desactive");
}
```

### Test 7: Partage apres fork() (2.3.5.g)

```rust
#[test]
fn test_fork_sharing() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <unistd.h>
        #include <sys/wait.h>

        int main(void) {
            fio_t* f = fio_open("fork_test.txt", "w+");
            fio_write(f, "0123456789", 10);
            fio_rewind(f);

            // 2.3.5.g: fork() partage les entries
            fio_prefork(f);

            pid_t pid = fork();

            if (pid == 0) {
                // Enfant
                fio_postfork_child(f);

                // Lire 5 octets -> avance l'offset partage
                char buf[5];
                fio_read(f, buf, 5);

                fio_close(f);
                _exit(0);
            }

            // Parent
            fio_postfork_parent(f);
            wait(NULL);

            // L'offset doit etre a 5 (modifie par l'enfant!)
            off_t pos = fio_tell(f);
            fio_close(f);

            // 2.3.5.c: System file table partagee
            if (pos != 5) return 1;

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "fork() partage l'offset via system file table");
}
```

### Test 8: Buffering et Performance

```rust
#[test]
fn test_buffering_modes() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            fio_t* f = fio_open("buf_test.txt", "w");

            // Test des differents modes
            if (fio_setbuf(f, FIO_UNBUFFERED) != 0) return 1;
            if (fio_setbuf(f, FIO_LINE_BUFFERED) != 0) return 2;
            if (fio_setbuf(f, FIO_FULLY_BUFFERED) != 0) return 3;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Changement de mode de buffering");
}

#[test]
fn test_readline() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <string.h>

        int main(void) {
            fio_t* f = fio_open("lines.txt", "w+");
            fio_write(f, "Line 1\nLine 2\nLine 3\n", 21);
            fio_rewind(f);

            char buf[100];

            // Premiere ligne
            ssize_t n = fio_readline(f, buf, 100);
            if (strcmp(buf, "Line 1\n") != 0) return 1;

            // Deuxieme ligne
            n = fio_readline(f, buf, 100);
            if (strcmp(buf, "Line 2\n") != 0) return 2;

            fio_close(f);
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "readline() lit ligne par ligne");
}
```

### Test 9: Gestion des Erreurs

```rust
#[test]
fn test_error_handling() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <errno.h>

        int main(void) {
            // Fichier inexistant
            fio_t* f = fio_open("nonexistent_file_xyz.txt", "r");
            if (f != NULL) return 1;
            if (errno != ENOENT) return 2;

            // Permission refusee (si possible)
            // Note: skip si root

            // Double close
            f = fio_open("error_test.txt", "w");
            fio_close(f);
            // Un deuxieme close devrait etre gere gracieusement
            // (le comportement depend de l'implementation)

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Gestion propre des erreurs");
}

#[test]
fn test_null_safety() {
    let result = run_c_test(r#"
        #include "fio.h"

        int main(void) {
            // Toutes ces operations sur NULL doivent echouer proprement
            if (fio_close(NULL) != -1) return 1;
            if (fio_read(NULL, NULL, 0) != -1) return 2;
            if (fio_write(NULL, NULL, 0) != -1) return 3;
            if (fio_seek(NULL, 0, 0) != -1) return 4;
            if (fio_fileno(NULL) != -1) return 5;

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Operations sur NULL gerees");
}
```

### Test 10: Integration Complete

```rust
#[test]
fn test_full_integration() {
    let result = run_c_test(r#"
        #include "fio.h"
        #include <string.h>
        #include <unistd.h>
        #include <sys/wait.h>
        #include <fcntl.h>

        int main(void) {
            // Test complet utilisant tous les concepts

            // === 2.3.5.e: Verifier les fd standards ===
            fio_process_info_t pinfo;
            fio_get_process_info(&pinfo);
            if (!pinfo.stdin_valid || !pinfo.stdout_valid || !pinfo.stderr_valid)
                return 1;

            // === 2.3.6.a-f: Creer fichier avec flags ===
            fio_t* main_file = fio_open_flags("integration.txt",
                O_RDWR | O_CREAT | O_TRUNC | O_CLOEXEC, 0644);
            if (!main_file) return 2;

            // === 2.3.5.k: Verifier FD_CLOEXEC ===
            int fd_flags = fcntl(fio_fileno(main_file), F_GETFD);
            if (!(fd_flags & FD_CLOEXEC)) return 3;

            // === 2.3.6.h: Ecrire ===
            fio_write(main_file, "Initial data\n", 13);

            // === 2.3.5.h: Dupliquer ===
            fio_t* dup_file = fio_dup(main_file);

            // === 2.3.5.f: Verifier partage offset ===
            fio_fd_info_t info1, info2;
            fio_get_fd_info(main_file, &info1);
            fio_get_fd_info(dup_file, &info2);
            if (info1.offset != info2.offset) return 4;

            // === 2.3.5.d: Meme inode ===
            if (info1.inode != info2.inode) return 5;

            // === 2.3.6.i: Seek ===
            fio_seek(main_file, 0, SEEK_SET);

            // === 2.3.6.g: Lire ===
            char buf[100];
            ssize_t n = fio_read(main_file, buf, 100);
            buf[n] = '\0';
            if (strcmp(buf, "Initial data\n") != 0) return 6;

            // === 2.3.5.g: Test fork sharing ===
            fio_write(main_file, "Before fork\n", 12);
            fio_prefork(main_file);

            pid_t pid = fork();
            if (pid == 0) {
                fio_postfork_child(main_file);
                fio_write(main_file, "From child\n", 11);
                fio_close(main_file);
                fio_close(dup_file);
                _exit(0);
            }

            fio_postfork_parent(main_file);
            wait(NULL);

            // === 2.3.6.k: Sync ===
            fio_flush(main_file, 1);

            // === 2.3.5.j: fcntl pour ajouter O_APPEND ===
            int flags = fio_get_flags(main_file);
            fio_set_flags(main_file, flags | O_APPEND);

            // === 2.3.6.e: Verifier O_APPEND ===
            flags = fio_get_flags(main_file);
            if (!(flags & O_APPEND)) return 7;

            // === 2.3.6.l: Truncate ===
            // D'abord desactiver O_APPEND pour le test
            fio_set_flags(main_file, flags & ~O_APPEND);
            off_t size_before = fio_seek(main_file, 0, SEEK_END);
            fio_truncate(main_file, size_before - 5);

            // === 2.3.5.b: Verifier la table per-process ===
            fio_get_process_info(&pinfo);
            if (pinfo.process_fd_count < 3) return 8;  // Au moins stdin/out/err

            // === 2.3.5.c: System file table (implicite via partage) ===

            // === 2.3.5.a: fd est bien un entier ===
            int fd = fio_fileno(main_file);
            if (fd < 0) return 9;

            // === 2.3.6.j: Close ===
            fio_close(main_file);
            fio_close(dup_file);

            // === 2.3.5.i: Test dup2 pour redirection ===
            fio_redirect_state_t saved = fio_save_fd(STDERR_FILENO);
            fio_t* err_file = fio_open("stderr_redirect.txt", "w");
            fio_redirect(err_file, STDERR_FILENO);
            write(STDERR_FILENO, "Redirected stderr\n", 18);
            fio_restore_fd(&saved);
            fio_close(err_file);

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Integration complete de tous les concepts");
}
```

---

## Bareme

### Distribution des Points (Total: 100 points)

#### Partie 1: File Descriptors (2.3.5) - 45 points

| Concept | Points | Description |
|---------|--------|-------------|
| 2.3.5.a | 3 | fd comme entier handle |
| 2.3.5.b | 5 | Implementation per-process table info |
| 2.3.5.c | 4 | Comprehension system file table |
| 2.3.5.d | 4 | Acces aux informations d'inode |
| 2.3.5.e | 3 | Gestion des fd standards |
| 2.3.5.f | 5 | File table entry (offset, flags, refcount) |
| 2.3.5.g | 8 | Partage correct apres fork() |
| 2.3.5.h | 5 | Implementation dup() |
| 2.3.5.i | 5 | Implementation dup2() avec redirection |
| 2.3.5.j | 5 | Implementation fcntl() |
| 2.3.5.k | 3 | Gestion FD_CLOEXEC |

#### Partie 2: File Operations (2.3.6) - 35 points

| Concept | Points | Description |
|---------|--------|-------------|
| 2.3.6.a | 3 | Implementation open() |
| 2.3.6.b | 3 | Gestion O_RDONLY/O_WRONLY/O_RDWR |
| 2.3.6.c | 2 | Support O_CREAT |
| 2.3.6.d | 2 | Support O_TRUNC |
| 2.3.6.e | 3 | Support O_APPEND |
| 2.3.6.f | 3 | Support O_EXCL |
| 2.3.6.g | 4 | Implementation read() avec buffering |
| 2.3.6.h | 4 | Implementation write() avec buffering |
| 2.3.6.i | 4 | Implementation lseek() |
| 2.3.6.j | 2 | Implementation close() |
| 2.3.6.k | 3 | Implementation fsync() |
| 2.3.6.l | 2 | Implementation ftruncate() |

#### Partie 3: Qualite et Robustesse - 20 points

| Critere | Points | Description |
|---------|--------|-------------|
| Gestion des erreurs | 5 | Verification de tous les retours, errno |
| Thread-safety | 4 | Protection mutex correcte |
| Pas de fuites memoire | 4 | Valgrind clean |
| Documentation | 3 | Commentaires references aux concepts |
| Code propre | 2 | Style, indentation, nommage |
| Tests additionnels | 2 | Couverture des cas limites |

### Penalites

| Violation | Penalite |
|-----------|----------|
| Compilation echoue | -100 (note finale: 0) |
| Warning de compilation | -2 par warning |
| Fuite memoire | -10 |
| Data race | -15 |
| Segfault | -20 |
| Concept manquant | -5 par concept |
| Fonction interdite | -10 par fonction |

### Bonus (Maximum +10 points)

| Bonus | Points |
|-------|--------|
| Support O_NONBLOCK | +2 |
| Implementation fcntl F_DUPFD_CLOEXEC | +2 |
| Support des advisory locks (flock) | +3 |
| Statistiques de performance (compteur d'appels systeme) | +3 |

---

## Fichiers a Rendre

```
ex03/
├── fio.h              # Header public de la bibliotheque
├── fio_internal.h     # Header interne (structure opaque)
├── fio.c              # Implementation principale
├── fio_buffer.c       # Gestion du buffering
├── fio_redirect.c     # Fonctions de redirection (dup/dup2)
├── fio_diagnostic.c   # Fonctions de diagnostic
├── Makefile           # Compilation de la bibliotheque
└── main.c             # Programme de demonstration
```

### Makefile Requis

```makefile
CC = gcc
CFLAGS = -std=c17 -Wall -Wextra -Werror -pedantic -g
LDFLAGS = -pthread

NAME = libfio.a

SRCS = fio.c fio_buffer.c fio_redirect.c fio_diagnostic.c
OBJS = $(SRCS:.c=.o)

all: $(NAME)

$(NAME): $(OBJS)
	ar rcs $(NAME) $(OBJS)

%.o: %.c fio.h fio_internal.h
	$(CC) $(CFLAGS) -c $< -o $@

demo: $(NAME) main.c
	$(CC) $(CFLAGS) main.c -L. -lfio $(LDFLAGS) -o demo

clean:
	rm -f $(OBJS) $(NAME) demo

re: clean all

.PHONY: all clean re demo
```

---

## Ressources Additionnelles

### Man Pages Essentielles
- `man 2 open` - Ouverture de fichiers
- `man 2 read` / `man 2 write` - I/O de base
- `man 2 close` - Fermeture de fd
- `man 2 lseek` - Positionnement
- `man 2 dup` / `man 2 dup2` - Duplication de fd
- `man 2 fcntl` - Controle de fd
- `man 2 fsync` - Synchronisation disque
- `man 2 ftruncate` - Redimensionnement

### Lecture Recommandee
- "Advanced Programming in the UNIX Environment" - Stevens & Rago, Chapitres 3-4
- "The Linux Programming Interface" - Kerrisk, Chapitres 4-5

### Diagramme des Tables du Noyau

```
+------------------+     +------------------+     +------------------+
|   Process A      |     |   Process B      |     |   Kernel         |
|   FD Table       |     |   FD Table       |     |   Inode Table    |
+------------------+     +------------------+     +------------------+
| 0 -> entry_0     |     | 0 -> entry_0     |     | inode 42         |
| 1 -> entry_1     |     | 1 -> entry_1     |     |   - type: file   |
| 2 -> entry_2     |     | 2 -> entry_2     |     |   - size: 1024   |
| 3 -> entry_5 ----+--+  | 3 -> entry_3     |     |   - links: 2     |
| 4 -> entry_5 ----+  |  +------------------+     |   - blocks: [..]  |
+------------------+  |                          +------------------+
                      |   System File Table      | inode 77         |
                      |   +------------------+   |   - type: dir    |
                      +-->| entry_5          |   +------------------+
                          |   offset: 512    |
                          |   flags: O_RDWR  |----> inode 42
                          |   refcount: 2    |
                          +------------------+
```

---

## Note Finale

Cet exercice couvre de maniere exhaustive les concepts 2.3.5.a-k et 2.3.6.a-l du curriculum. L'implementation d'une bibliotheque I/O complete permet de comprendre en profondeur l'architecture des file descriptors Unix et les operations fondamentales sur les fichiers.

L'accent est mis sur:
1. La comprehension de l'architecture en trois niveaux (per-process table, system file table, inode table)
2. La maitrise des syscalls de base (open, read, write, close, lseek)
3. Les operations avancees (dup, dup2, fcntl)
4. Le comportement apres fork() et les implications du partage
5. La robustesse et la gestion des erreurs

La moulinette Rust verifiera systematiquement que chaque concept est correctement implemente et documente.
