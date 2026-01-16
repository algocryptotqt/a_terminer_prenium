# Exercice 2.3.5-synth : enterprise_io_library

**Module :**
2.3.5/2.3.6 — File Descriptors & File Operations

**Concept :**
a-l — Bibliothèque I/O complète avec buffering

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
code

**Tiers :**
3 — Synthèse (concepts 2.3.5.a-k + 2.3.6.a-l)

**Langage :**
C (C17)

**Prérequis :**
- 2.3.1 (stat/lstat, inodes)
- 2.3.4 (links)
- Manipulation mémoire (malloc/free)
- Varargs (va_list)

**Domaines :**
FS, Mem, Process

**Durée estimée :**
360 min (6h)

**XP Base :**
600

**Complexité :**
T2 O(n) × S2 O(BUFSIZ)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex03/
├── enterprise_io.h        # Header avec structures et prototypes
├── enterprise_io.c        # Implémentation principale
├── channel_utils.c        # Utilitaires (diagnostics, redirections)
└── Makefile
```

**Fonctions autorisées :**
```c
// Syscalls fichiers
open, close, read, write, lseek, fsync, ftruncate

// Syscalls descripteurs
dup, dup2, fcntl

// Stat
stat, fstat, isatty

// Mémoire
malloc, free, calloc, realloc

// Chaînes et formatage
strlen, memcpy, memmove, memset
vsnprintf, snprintf

// Erreurs
strerror, errno
```

**Fonctions interdites :**
```c
fopen, fclose, fread, fwrite, fprintf  // On les réimplémente!
printf, puts, putchar                  // Utiliser notre propre API
```

### 1.2 Consigne

#### 🚀 CONTEXTE FUN — Star Trek: The Next Generation

**"Computer, open a channel."** — Captain Jean-Luc Picard

Tu es **Lieutenant Commander Data**, l'officier des opérations à bord de l'**USS Enterprise NCC-1701-D**. Le capitaine Picard t'a confié une mission critique : réimplémenter le **système de communication** du vaisseau.

Sur l'Enterprise, chaque **canal de communication** est identifié par un **numéro de fréquence** (file descriptor). Le vaisseau dispose de trois canaux prédéfinis :
- **Canal 0 (Pont Principal)** : Réception des ordres du capitaine (stdin)
- **Canal 1 (Écran Principal)** : Affichage sur le viewscreen (stdout)
- **Canal 2 (Alerte Rouge)** : Messages d'urgence prioritaires (stderr)

Le système utilise des **pattern buffers** (comme le téléporteur) pour stocker temporairement les données avant transmission, optimisant ainsi l'utilisation de la bande passante subspace.

**Ta mission :** Créer la bibliothèque `libenterprise_io` pour gérer :
- L'ouverture et fermeture de canaux de communication
- La lecture et écriture de données avec buffering
- Les redirections de canaux (comme reroutage d'énergie)
- Les diagnostics système (état des canaux)

---

#### 1.2.2 Énoncé Académique

**Ta mission :**

Implémenter une **bibliothèque d'I/O bufferisée** complète qui :

1. **Encapsule les file descriptors** dans une structure opaque avec buffer
2. **Gère trois modes de buffering** : non-bufferisé, ligne, complet
3. **Implémente les opérations** : open, close, read, write, seek, flush
4. **Supporte les redirections** avec dup/dup2
5. **Fournit des diagnostics** sur l'état des descripteurs

**Architecture en trois niveaux du kernel :**

```
    PROCESSUS                           NOYAU
    =========                    ==================

  Per-Process FD Table         System File Table        Inode Table
  +------------------+         +-----------------+      +-----------+
  | fd 0 → entry A --|-------->| Entry A         |      | inode 42  |
  | fd 1 → entry B --|--+      | offset: 0       |----->| size: 8K  |
  | fd 2 → entry B --|--+----->| flags: O_RDONLY |      | blocks:[] |
  | fd 3 → entry C --|-------->| refcount: 2     |      +-----------+
  +------------------+         +-----------------+
                               | Entry B         |
                               | offset: 1024    |
                               | flags: O_RDWR   |
                               | refcount: 1     |
                               +-----------------+
```

**Concepts clés :**
- Un fd est juste un index dans la table per-process (2.3.5.a)
- Plusieurs fd peuvent pointer vers la même entry (dup, fork) (2.3.5.g-i)
- L'entry contient offset, flags, refcount (2.3.5.f)
- L'inode contient les métadonnées du fichier (2.3.5.d)

### 1.3 Prototype

```c
#ifndef ENTERPRISE_IO_H
#define ENTERPRISE_IO_H

#include <stddef.h>
#include <sys/types.h>
#include <stdarg.h>

/*============================================================================
 * STARFLEET CONSTANTS
 *============================================================================*/

#define PATTERN_BUFFER_SIZE 4096  /* Taille du buffer interne */

/* Canaux standards de l'Enterprise (2.3.5.e) */
#define BRIDGE_INPUT    0   /* stdin  - Ordres du capitaine */
#define MAIN_VIEWSCREEN 1   /* stdout - Écran principal */
#define RED_ALERT       2   /* stderr - Alertes prioritaires */

/*============================================================================
 * TYPES — Classification des Canaux
 *============================================================================*/

/* Mode de buffering du canal */
typedef enum {
    BUFFER_NONE,      /* Transmission immédiate (stderr) */
    BUFFER_LINE,      /* Flush à chaque fin de transmission */
    BUFFER_FULL       /* Flush quand buffer plein (fichiers) */
} buffer_mode_t;

/* État du canal */
typedef enum {
    CHANNEL_ACTIVE,   /* Canal opérationnel */
    CHANNEL_EOF,      /* Fin de transmission */
    CHANNEL_ERROR     /* Erreur de communication */
} channel_state_t;

/* Codes d'erreur Starfleet */
typedef enum {
    STARFLEET_OK           =  0,
    STARFLEET_ERROR        = -1,
    STARFLEET_EOF          = -2,
    STARFLEET_INVALID      = -3,
    STARFLEET_NO_MEMORY    = -4,
    STARFLEET_PERMISSION   = -5,
    STARFLEET_NOT_FOUND    = -6,
    STARFLEET_EXISTS       = -7,
    STARFLEET_BUSY         = -8
} starfleet_error_t;

/*============================================================================
 * STRUCTURES — Données du Vaisseau
 *============================================================================*/

/* Canal de communication (structure opaque) */
typedef struct comm_channel comm_channel_t;

/* Informations sur un canal (diagnostic) */
typedef struct {
    int     frequency;       /* Numéro du fd (2.3.5.a) */
    int     open_flags;      /* Flags d'ouverture (2.3.5.f) */
    off_t   position;        /* Position dans le flux (2.3.5.f) */
    int     fd_flags;        /* FD_CLOEXEC etc. (2.3.5.k) */
    int     is_active;       /* Canal valide? */
    int     is_terminal;     /* Connecté à un terminal? */
    int     is_regular;      /* Fichier régulier? */
    int     is_pipe;         /* Conduit de Jefferies? (pipe) */
    ino_t   inode;           /* Signature inode (2.3.5.d) */
    dev_t   device;          /* Device ID */
} channel_info_t;

/* État global des communications */
typedef struct {
    int     active_channels;    /* Nombre de canaux ouverts */
    int     max_frequency;      /* Plus haute fréquence utilisée */
    int     available_slots;    /* Canaux disponibles */
    int     bridge_active;      /* Canal 0 (stdin) actif? */
    int     viewscreen_active;  /* Canal 1 (stdout) actif? */
    int     alert_active;       /* Canal 2 (stderr) actif? */
} ship_comm_status_t;

/*============================================================================
 * API — Ouverture et Fermeture de Canaux
 *============================================================================*/

/**
 * open_channel - Ouvre un canal de communication
 *
 * "Computer, open hailing frequencies."
 *
 * @param path  Destination (chemin fichier)
 * @param mode  Mode d'accès ("r", "w", "a", "r+", "w+", "a+")
 * @return Canal ouvert, NULL si erreur
 *
 * Mapping des modes vers flags (2.3.6.b-f):
 * - "r"  : O_RDONLY
 * - "w"  : O_WRONLY | O_CREAT | O_TRUNC
 * - "a"  : O_WRONLY | O_CREAT | O_APPEND
 * - "r+" : O_RDWR
 * - "w+" : O_RDWR | O_CREAT | O_TRUNC
 * - "a+" : O_RDWR | O_CREAT | O_APPEND
 */
comm_channel_t *open_channel(const char *path, const char *mode);

/**
 * open_channel_flags - Ouvre avec flags explicites
 *
 * Pour accès aux flags avancés (O_EXCL, O_CLOEXEC).
 *
 * @param path   Destination
 * @param flags  Flags open() (2.3.6.a-f)
 * @param perms  Permissions si création
 * @return Canal ouvert, NULL si erreur
 */
comm_channel_t *open_channel_flags(const char *path, int flags, mode_t perms);

/**
 * close_channel - Ferme un canal
 *
 * "Close channel." Flush le buffer puis ferme le fd.
 *
 * @param channel Canal à fermer
 * @return 0 si succès, -1 si erreur
 *
 * Utilise close() (2.3.6.j)
 */
int close_channel(comm_channel_t *channel);

/**
 * channel_from_frequency - Crée un canal depuis un fd existant
 *
 * Utile pour encapsuler stdin/stdout/stderr.
 */
comm_channel_t *channel_from_frequency(int fd, const char *mode);

/*============================================================================
 * API — Lecture et Écriture
 *============================================================================*/

/**
 * receive_transmission - Lit des données depuis un canal
 *
 * "Data, status report." Lit les données entrantes.
 *
 * @param channel Canal source
 * @param buffer  Buffer destination
 * @param count   Nombre max d'octets
 * @return Nombre d'octets lus, 0 si EOF, -1 si erreur
 *
 * Utilise read() (2.3.6.g) avec buffering
 */
ssize_t receive_transmission(comm_channel_t *channel, void *buffer, size_t count);

/**
 * send_transmission - Écrit des données vers un canal
 *
 * "Transmit on all frequencies." Envoie les données.
 *
 * @param channel Canal destination
 * @param data    Données à envoyer
 * @param count   Nombre d'octets
 * @return Nombre d'octets écrits, -1 si erreur
 *
 * Utilise write() (2.3.6.h) avec buffering
 */
ssize_t send_transmission(comm_channel_t *channel, const void *data, size_t count);

/**
 * receive_message - Lit une ligne complète
 *
 * Lit jusqu'au prochain '\n' ou EOF.
 */
ssize_t receive_message(comm_channel_t *channel, char *buffer, size_t maxlen);

/**
 * send_formatted - Envoie un message formaté (comme printf)
 *
 * "Computer, display message: ..."
 */
int send_formatted(comm_channel_t *channel, const char *format, ...);

/**
 * receive_byte / send_byte - Opérations caractère par caractère
 */
int receive_byte(comm_channel_t *channel);
int send_byte(comm_channel_t *channel, int byte);

/*============================================================================
 * API — Positionnement et Contrôle
 *============================================================================*/

/**
 * seek_position - Change la position dans le flux
 *
 * "Computer, skip to timestamp..."
 *
 * @param channel Canal
 * @param offset  Décalage
 * @param whence  SEEK_SET, SEEK_CUR, SEEK_END
 * @return Nouvelle position, -1 si erreur
 *
 * Utilise lseek() (2.3.6.i)
 */
off_t seek_position(comm_channel_t *channel, off_t offset, int whence);

/**
 * flush_buffer - Force la transmission des données bufferisées
 *
 * "Engage!" - Transmet tout ce qui est en attente.
 *
 * Utilise write() pour vider le buffer.
 */
int flush_buffer(comm_channel_t *channel);

/**
 * sync_to_disk - Force l'écriture sur stockage permanent
 *
 * "Save to ship's log." - Garantit la persistance.
 *
 * Utilise fsync() (2.3.6.k)
 */
int sync_to_disk(comm_channel_t *channel);

/**
 * truncate_channel - Redimensionne le fichier
 *
 * Utilise ftruncate() (2.3.6.l)
 */
int truncate_channel(comm_channel_t *channel, off_t length);

/*============================================================================
 * API — Redirections (Reroutage d'Énergie)
 *============================================================================*/

/**
 * reroute_channel - Redirige un canal vers un autre
 *
 * "Reroute auxiliary power!" - Comme dup2() (2.3.5.i)
 *
 * @param source      Canal source
 * @param target_fd   Fd cible (sera fermé s'il était ouvert)
 * @return Nouveau fd, -1 si erreur
 */
int reroute_channel(comm_channel_t *source, int target_fd);

/**
 * duplicate_channel - Duplique un canal
 *
 * Comme dup() (2.3.5.h) - même entry, offset partagé.
 */
comm_channel_t *duplicate_channel(comm_channel_t *channel);

/**
 * get_channel_flags - Obtient les flags du canal
 *
 * Utilise fcntl(F_GETFL) (2.3.5.j)
 */
int get_channel_flags(comm_channel_t *channel);

/**
 * set_channel_flags - Modifie les flags du canal
 *
 * Utilise fcntl(F_SETFL) (2.3.5.j)
 */
int set_channel_flags(comm_channel_t *channel, int flags);

/**
 * set_close_on_exec - Active FD_CLOEXEC
 *
 * Ferme automatiquement lors d'exec() (2.3.5.k)
 */
int set_close_on_exec(comm_channel_t *channel, int enable);

/*============================================================================
 * API — Diagnostics
 *============================================================================*/

/**
 * scan_channel - Analyse un canal et retourne ses propriétés
 *
 * "Computer, run diagnostic on channel 3."
 */
int scan_channel(comm_channel_t *channel, channel_info_t *info);

/**
 * scan_ship_comms - État global des communications
 *
 * "Status report on all ship's communications."
 */
int scan_ship_comms(ship_comm_status_t *status);

/**
 * get_channel_frequency - Retourne le fd sous-jacent
 */
int get_channel_frequency(comm_channel_t *channel);

/**
 * get_channel_state - Retourne l'état du canal
 */
channel_state_t get_channel_state(comm_channel_t *channel);

/**
 * is_channel_eof - Vérifie si fin de transmission
 */
int is_channel_eof(comm_channel_t *channel);

/**
 * is_channel_error - Vérifie si erreur de communication
 */
int is_channel_error(comm_channel_t *channel);

/**
 * clear_channel_error - Réinitialise l'état d'erreur
 */
void clear_channel_error(comm_channel_t *channel);

/*============================================================================
 * API — Configuration du Buffering
 *============================================================================*/

/**
 * set_buffer_mode - Change le mode de buffering
 *
 * @param channel Canal
 * @param mode    BUFFER_NONE, BUFFER_LINE, BUFFER_FULL
 * @param buffer  Buffer custom (NULL = interne)
 * @param size    Taille du buffer (0 = défaut)
 */
int set_buffer_mode(comm_channel_t *channel, buffer_mode_t mode,
                    char *buffer, size_t size);

/*============================================================================
 * UTILITAIRES
 *============================================================================*/

const char *starfleet_strerror(starfleet_error_t error);

#endif /* ENTERPRISE_IO_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Architecture en Trois Niveaux

Comme le vaisseau Enterprise a trois ponts principaux (Passerelle, Ingénierie, Sciences), le kernel Unix a trois tables pour gérer les fichiers :

1. **Per-Process Table** = Console du Pont (chaque officier a sa propre vue)
2. **System File Table** = Salle de contrôle centrale (partagée par tous)
3. **Inode Table** = Archives du vaisseau (les données réelles)

### 2.2 fork() et le Partage d'Entrées

Quand l'Enterprise lance une navette (fork), la navette hérite des **mêmes canaux de communication** que le vaisseau mère. Si le vaisseau avance dans la lecture d'un fichier, la navette voit le nouvel offset !

```c
int fd = open("starlog.txt", O_RDONLY);
if (fork() == 0) {
    char buf[10];
    read(fd, buf, 10);  // Enfant lit 10 octets
}
else {
    sleep(1);
    char buf[10];
    read(fd, buf, 10);  // Parent lit à partir de l'offset 10!
}
```

### 2.3 dup2() : Le Reroutage d'Énergie

Comme Geordi La Forge peut **rerouter l'énergie** des boucliers vers les moteurs, `dup2()` permet de rediriger un canal vers un autre :

```c
int log_fd = open("captain.log", O_WRONLY | O_CREAT, 0644);
dup2(log_fd, STDOUT_FILENO);  // Reroute stdout vers le fichier
close(log_fd);
printf("Captain's log, stardate 47988.1\n");  // Va dans le fichier!
```

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation |
|--------|-------------|
| **Shell Developer** | Implémentation des redirections (`>`, `<`, `|`) |
| **Web Server Dev** | Gestion des connexions avec fd et buffering |
| **Database Dev** | I/O bufferisé pour performances |
| **DevOps** | Redirection de logs, daemonization |
| **Security Engineer** | Audit des fd ouverts, leaks |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
enterprise_io.c  channel_utils.c  enterprise_io.h  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -std=c17 -c enterprise_io.c
gcc -Wall -Wextra -Werror -std=c17 -c channel_utils.c
ar rcs libenterprise.a enterprise_io.o channel_utils.o

$ gcc -Wall -Wextra -Werror main.c -L. -lenterprise -o test

$ ./test
=== ENTERPRISE COMMUNICATIONS TEST ===

Test 1: Open channel for writing
Channel opened on frequency 3

Test 2: Send transmission
Sent 26 bytes: "Captain's log, stardate...\n"

Test 3: Flush and close
Buffer flushed, channel closed

Test 4: Open for reading
Received: "Captain's log, stardate..."

Test 5: Redirection (dup2)
Rerouting stdout to file...
[Message appears in file, not terminal]

Test 6: Buffering modes
Testing BUFFER_NONE: Immediate write
Testing BUFFER_LINE: Write on newline
Testing BUFFER_FULL: Write when buffer full

Test 7: Ship communications status
Active channels: 4
Standard channels: stdin=OK stdout=OK stderr=OK

All systems operational!
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (20 tests)

| # | Test | Entrée | Sortie Attendue | Concept |
|---|------|--------|-----------------|---------|
| 01 | Open "r" mode | `open_channel("file", "r")` | fd valide, O_RDONLY | 2.3.6.a,b |
| 02 | Open "w" mode | `open_channel("new", "w")` | Crée fichier, O_WRONLY\|O_CREAT\|O_TRUNC | 2.3.6.c,d |
| 03 | Open "a" mode | `open_channel("log", "a")` | O_APPEND set | 2.3.6.e |
| 04 | O_EXCL avec existe | `open_channel_flags(..., O_EXCL)` | NULL, errno=EEXIST | 2.3.6.f |
| 05 | Read basique | `receive_transmission(ch, buf, 100)` | Données lues | 2.3.6.g |
| 06 | Write basique | `send_transmission(ch, data, len)` | len retourné | 2.3.6.h |
| 07 | lseek SEEK_SET | `seek_position(ch, 0, SEEK_SET)` | Position = 0 | 2.3.6.i |
| 08 | lseek SEEK_END | `seek_position(ch, -10, SEEK_END)` | 10 avant fin | 2.3.6.i |
| 09 | Close libère fd | `close_channel(ch)` | fd réutilisable | 2.3.6.j |
| 10 | fsync | `sync_to_disk(ch)` | Données sur disque | 2.3.6.k |
| 11 | ftruncate | `truncate_channel(ch, 100)` | Taille = 100 | 2.3.6.l |
| 12 | dup() | `duplicate_channel(ch)` | Même entry, offset partagé | 2.3.5.h |
| 13 | dup2() | `reroute_channel(ch, 1)` | stdout redirigé | 2.3.5.i |
| 14 | fcntl F_GETFL | `get_channel_flags(ch)` | Flags corrects | 2.3.5.j |
| 15 | FD_CLOEXEC | `set_close_on_exec(ch, 1)` | Flag set | 2.3.5.k |
| 16 | Buffering FULL | Écrire < BUFSIZ | Pas de write immédiat | Buffering |
| 17 | Buffering LINE | Écrire avec \n | Flush sur newline | Buffering |
| 18 | Buffering NONE | Écrire 1 octet | write() immédiat | Buffering |
| 19 | NULL params | `open_channel(NULL, "r")` | NULL | Robustesse |
| 20 | Valgrind | Cycle complet | 0 leaks | Sécurité |

### 4.2 main.c de test

```c
#include "enterprise_io.h"
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>

#define TEST(name, cond) do { \
    if (cond) printf("[OK] %s\n", name); \
    else printf("[FAIL] %s\n", name); \
} while(0)

int main(void)
{
    printf("=== ENTERPRISE I/O TESTS ===\n\n");

    /* Test 1: Open for writing */
    comm_channel_t *ch = open_channel("/tmp/starlog.txt", "w");
    TEST("Open channel 'w'", ch != NULL);

    /* Test 2: Write data */
    const char *msg = "Captain's log, stardate 47988.1\n";
    ssize_t written = send_transmission(ch, msg, strlen(msg));
    TEST("Send transmission", written == (ssize_t)strlen(msg));

    /* Test 3: Flush and close */
    TEST("Flush buffer", flush_buffer(ch) == 0);
    TEST("Close channel", close_channel(ch) == 0);

    /* Test 4: Open for reading */
    ch = open_channel("/tmp/starlog.txt", "r");
    TEST("Open channel 'r'", ch != NULL);

    char buffer[100];
    ssize_t bytes = receive_transmission(ch, buffer, sizeof(buffer) - 1);
    buffer[bytes] = '\0';
    TEST("Receive transmission", bytes > 0 && strstr(buffer, "Captain's log"));

    close_channel(ch);

    /* Test 5: dup2 redirection */
    ch = open_channel("/tmp/redirect.txt", "w");
    int old_stdout = dup(STDOUT_FILENO);
    reroute_channel(ch, STDOUT_FILENO);
    printf("This goes to file\n");
    fflush(stdout);
    dup2(old_stdout, STDOUT_FILENO);
    close(old_stdout);
    close_channel(ch);

    ch = open_channel("/tmp/redirect.txt", "r");
    bytes = receive_transmission(ch, buffer, sizeof(buffer) - 1);
    buffer[bytes] = '\0';
    TEST("dup2 redirection", strstr(buffer, "This goes to file"));
    close_channel(ch);

    /* Test 6: Channel info */
    ch = open_channel("/tmp/starlog.txt", "r");
    channel_info_t info;
    scan_channel(ch, &info);
    TEST("Channel diagnostics", info.is_regular == 1);
    close_channel(ch);

    /* Cleanup */
    unlink("/tmp/starlog.txt");
    unlink("/tmp/redirect.txt");

    printf("\n=== ALL SYSTEMS OPERATIONAL ===\n");
    return 0;
}
```

### 4.3 Solution de référence (structure interne)

```c
/* enterprise_io.c - Solution de référence (extrait) */
#include "enterprise_io.h"
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/stat.h>

/* Structure interne du canal */
struct comm_channel {
    int             fd;             /* File descriptor (2.3.5.a) */
    buffer_mode_t   buffer_mode;    /* Mode de buffering */
    channel_state_t state;          /* État du canal */

    /* Buffer de lecture */
    char           *read_buf;
    size_t          read_buf_size;
    size_t          read_pos;       /* Position de lecture */
    size_t          read_end;       /* Fin des données valides */

    /* Buffer d'écriture */
    char           *write_buf;
    size_t          write_buf_size;
    size_t          write_pos;      /* Quantité de données bufferisées */

    int             flags;          /* Flags d'ouverture */
    int             can_read;
    int             can_write;
    int             owns_buffer;    /* Buffer alloué par nous? */
};

/*============================================================================
 * HELPERS
 *============================================================================*/

static int parse_mode(const char *mode, int *flags, int *can_read, int *can_write)
{
    *can_read = 0;
    *can_write = 0;

    if (strcmp(mode, "r") == 0) {
        *flags = O_RDONLY;
        *can_read = 1;
    }
    else if (strcmp(mode, "w") == 0) {
        *flags = O_WRONLY | O_CREAT | O_TRUNC;
        *can_write = 1;
    }
    else if (strcmp(mode, "a") == 0) {
        *flags = O_WRONLY | O_CREAT | O_APPEND;
        *can_write = 1;
    }
    else if (strcmp(mode, "r+") == 0) {
        *flags = O_RDWR;
        *can_read = *can_write = 1;
    }
    else if (strcmp(mode, "w+") == 0) {
        *flags = O_RDWR | O_CREAT | O_TRUNC;
        *can_read = *can_write = 1;
    }
    else if (strcmp(mode, "a+") == 0) {
        *flags = O_RDWR | O_CREAT | O_APPEND;
        *can_read = *can_write = 1;
    }
    else {
        return -1;
    }
    return 0;
}

/*============================================================================
 * OUVERTURE / FERMETURE
 *============================================================================*/

comm_channel_t *open_channel(const char *path, const char *mode)
{
    int flags, can_read, can_write;

    if (path == NULL || mode == NULL)
        return NULL;

    if (parse_mode(mode, &flags, &can_read, &can_write) == -1)
        return NULL;

    mode_t perms = 0644;
    return open_channel_flags(path, flags, perms);
}

comm_channel_t *open_channel_flags(const char *path, int flags, mode_t perms)
{
    comm_channel_t *ch;
    int fd;

    if (path == NULL)
        return NULL;

    /* Ouvrir le fichier (2.3.6.a) */
    if (flags & O_CREAT)
        fd = open(path, flags, perms);
    else
        fd = open(path, flags);

    if (fd == -1)
        return NULL;

    /* Allouer la structure */
    ch = calloc(1, sizeof(comm_channel_t));
    if (ch == NULL) {
        close(fd);
        return NULL;
    }

    ch->fd = fd;
    ch->flags = flags;
    ch->state = CHANNEL_ACTIVE;

    /* Déterminer capacités */
    int access_mode = flags & O_ACCMODE;
    ch->can_read = (access_mode == O_RDONLY || access_mode == O_RDWR);
    ch->can_write = (access_mode == O_WRONLY || access_mode == O_RDWR);

    /* Allouer les buffers */
    ch->read_buf_size = PATTERN_BUFFER_SIZE;
    ch->write_buf_size = PATTERN_BUFFER_SIZE;
    ch->read_buf = malloc(ch->read_buf_size);
    ch->write_buf = malloc(ch->write_buf_size);
    ch->owns_buffer = 1;

    if (ch->read_buf == NULL || ch->write_buf == NULL) {
        free(ch->read_buf);
        free(ch->write_buf);
        close(fd);
        free(ch);
        return NULL;
    }

    /* Mode de buffering par défaut */
    if (isatty(fd))
        ch->buffer_mode = BUFFER_LINE;
    else
        ch->buffer_mode = BUFFER_FULL;

    return ch;
}

int close_channel(comm_channel_t *channel)
{
    if (channel == NULL)
        return -1;

    /* Flush le buffer d'écriture */
    if (channel->can_write)
        flush_buffer(channel);

    /* Fermer le fd (2.3.6.j) */
    int result = close(channel->fd);

    /* Libérer les ressources */
    if (channel->owns_buffer) {
        free(channel->read_buf);
        free(channel->write_buf);
    }
    free(channel);

    return result;
}

/*============================================================================
 * LECTURE / ÉCRITURE
 *============================================================================*/

ssize_t receive_transmission(comm_channel_t *channel, void *buffer, size_t count)
{
    if (channel == NULL || buffer == NULL || !channel->can_read)
        return -1;

    if (channel->state == CHANNEL_EOF)
        return 0;

    size_t total = 0;
    char *dest = buffer;

    while (total < count) {
        /* Données disponibles dans le buffer? */
        if (channel->read_pos < channel->read_end) {
            size_t available = channel->read_end - channel->read_pos;
            size_t to_copy = (count - total < available) ? count - total : available;
            memcpy(dest + total, channel->read_buf + channel->read_pos, to_copy);
            channel->read_pos += to_copy;
            total += to_copy;
        }
        else {
            /* Remplir le buffer depuis le fd (2.3.6.g) */
            ssize_t n = read(channel->fd, channel->read_buf, channel->read_buf_size);
            if (n < 0) {
                channel->state = CHANNEL_ERROR;
                return total > 0 ? (ssize_t)total : -1;
            }
            if (n == 0) {
                channel->state = CHANNEL_EOF;
                break;
            }
            channel->read_pos = 0;
            channel->read_end = n;
        }
    }

    return total;
}

ssize_t send_transmission(comm_channel_t *channel, const void *data, size_t count)
{
    if (channel == NULL || data == NULL || !channel->can_write)
        return -1;

    const char *src = data;
    size_t total = 0;

    while (total < count) {
        size_t space = channel->write_buf_size - channel->write_pos;
        size_t to_copy = (count - total < space) ? count - total : space;

        memcpy(channel->write_buf + channel->write_pos, src + total, to_copy);
        channel->write_pos += to_copy;
        total += to_copy;

        /* Flush si nécessaire */
        int should_flush = 0;
        if (channel->buffer_mode == BUFFER_NONE)
            should_flush = 1;
        else if (channel->buffer_mode == BUFFER_LINE &&
                 memchr(src + total - to_copy, '\n', to_copy))
            should_flush = 1;
        else if (channel->write_pos >= channel->write_buf_size)
            should_flush = 1;

        if (should_flush && flush_buffer(channel) == -1)
            return total > 0 ? (ssize_t)total : -1;
    }

    return total;
}

int flush_buffer(comm_channel_t *channel)
{
    if (channel == NULL)
        return -1;

    if (channel->write_pos == 0)
        return 0;

    /* Écrire tout le buffer (2.3.6.h) */
    size_t written = 0;
    while (written < channel->write_pos) {
        ssize_t n = write(channel->fd, channel->write_buf + written,
                          channel->write_pos - written);
        if (n < 0) {
            channel->state = CHANNEL_ERROR;
            return -1;
        }
        written += n;
    }

    channel->write_pos = 0;
    return 0;
}

/*============================================================================
 * POSITIONNEMENT
 *============================================================================*/

off_t seek_position(comm_channel_t *channel, off_t offset, int whence)
{
    if (channel == NULL)
        return -1;

    /* Flush les écritures en attente */
    if (channel->can_write)
        flush_buffer(channel);

    /* Invalider le buffer de lecture */
    channel->read_pos = 0;
    channel->read_end = 0;

    /* Appeler lseek (2.3.6.i) */
    return lseek(channel->fd, offset, whence);
}

/*============================================================================
 * REDIRECTIONS
 *============================================================================*/

int reroute_channel(comm_channel_t *source, int target_fd)
{
    if (source == NULL)
        return -1;

    /* Flush d'abord */
    flush_buffer(source);

    /* dup2 (2.3.5.i) */
    return dup2(source->fd, target_fd);
}

comm_channel_t *duplicate_channel(comm_channel_t *channel)
{
    if (channel == NULL)
        return NULL;

    /* dup (2.3.5.h) */
    int new_fd = dup(channel->fd);
    if (new_fd == -1)
        return NULL;

    /* Créer un nouveau canal pour le fd dupliqué */
    comm_channel_t *new_ch = calloc(1, sizeof(comm_channel_t));
    if (new_ch == NULL) {
        close(new_fd);
        return NULL;
    }

    /* Copier les propriétés */
    new_ch->fd = new_fd;
    new_ch->flags = channel->flags;
    new_ch->can_read = channel->can_read;
    new_ch->can_write = channel->can_write;
    new_ch->buffer_mode = channel->buffer_mode;
    new_ch->state = CHANNEL_ACTIVE;

    /* Nouveaux buffers */
    new_ch->read_buf_size = PATTERN_BUFFER_SIZE;
    new_ch->write_buf_size = PATTERN_BUFFER_SIZE;
    new_ch->read_buf = malloc(new_ch->read_buf_size);
    new_ch->write_buf = malloc(new_ch->write_buf_size);
    new_ch->owns_buffer = 1;

    if (new_ch->read_buf == NULL || new_ch->write_buf == NULL) {
        free(new_ch->read_buf);
        free(new_ch->write_buf);
        close(new_fd);
        free(new_ch);
        return NULL;
    }

    return new_ch;
}
```

### 4.10 Solutions Mutantes (6 mutants)

```c
/* MUTANT A (Resource): Pas de flush avant close */
int mutant_a_close_channel(comm_channel_t *channel)
{
    /* BUG: Oubli du flush! Données perdues! */
    // flush_buffer(channel);  /* MANQUANT! */
    int result = close(channel->fd);
    free(channel->read_buf);
    free(channel->write_buf);
    free(channel);
    return result;
}
/* Conséquence: Perte de données bufferisées */

/* MUTANT B (Logic): readlink au lieu de read */
ssize_t mutant_b_receive(comm_channel_t *channel, void *buffer, size_t count)
{
    /* BUG: Utilise read directement sans buffer */
    return read(channel->fd, buffer, count);  /* Pas de buffering! */
}
/* Conséquence: Inefficace, pas de buffering */

/* MUTANT C (Safety): Pas de vérification NULL */
ssize_t mutant_c_send(comm_channel_t *channel, const void *data, size_t count)
{
    /* BUG: Pas de vérification des paramètres */
    size_t to_copy = count;  /* Crash si channel == NULL */
    memcpy(channel->write_buf + channel->write_pos, data, to_copy);
    channel->write_pos += to_copy;
    return count;
}
/* Conséquence: Segfault sur paramètres NULL */

/* MUTANT D (Logic): lseek sans invalider buffer */
off_t mutant_d_seek(comm_channel_t *channel, off_t offset, int whence)
{
    /* BUG: Buffer de lecture pas invalidé! */
    // channel->read_pos = 0;   /* MANQUANT! */
    // channel->read_end = 0;   /* MANQUANT! */
    return lseek(channel->fd, offset, whence);
}
/* Conséquence: Lecture de données obsolètes */

/* MUTANT E (Boundary): Buffer overflow */
ssize_t mutant_e_send(comm_channel_t *channel, const void *data, size_t count)
{
    /* BUG: Pas de vérification de l'espace disponible */
    memcpy(channel->write_buf + channel->write_pos, data, count);
    channel->write_pos += count;  /* Peut dépasser write_buf_size! */
    return count;
}
/* Conséquence: Buffer overflow si count > espace disponible */

/* MUTANT F (Return): Mauvaise gestion EOF vs erreur */
ssize_t mutant_f_receive(comm_channel_t *channel, void *buffer, size_t count)
{
    ssize_t n = read(channel->fd, buffer, count);
    if (n <= 0)
        return -1;  /* BUG: EOF (n=0) et erreur (n=-1) confondus! */
    return n;
}
/* Conséquence: EOF traité comme erreur */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Description | Référence |
|---------|-------------|-----------|
| **fd = Integer handle** | Un fd est un index dans la table per-process | 2.3.5.a |
| **Per-process table** | Chaque processus a sa propre table fd→entry | 2.3.5.b |
| **System file table** | Table globale avec offset, flags, refcount | 2.3.5.c |
| **Inode table** | Inodes en mémoire | 2.3.5.d |
| **stdin/stdout/stderr** | fd 0, 1, 2 | 2.3.5.e |
| **Entry: offset, flags, refcount** | Contenu d'une entrée file table | 2.3.5.f |
| **fork() partage entries** | Parent et enfant partagent les mêmes entries | 2.3.5.g |
| **dup()** | Duplique vers le plus petit fd dispo | 2.3.5.h |
| **dup2()** | Duplique vers un fd spécifique | 2.3.5.i |
| **fcntl()** | Manipulation des propriétés fd | 2.3.5.j |
| **FD_CLOEXEC** | Ferme auto sur exec() | 2.3.5.k |
| **open()** | Ouvre/crée un fichier | 2.3.6.a |
| **O_RDONLY, O_WRONLY, O_RDWR** | Modes d'accès | 2.3.6.b |
| **O_CREAT** | Crée si n'existe pas | 2.3.6.c |
| **O_TRUNC** | Tronque à zéro | 2.3.6.d |
| **O_APPEND** | Écrit toujours en fin | 2.3.6.e |
| **O_EXCL** | Échoue si existe | 2.3.6.f |
| **read()** | Lit des octets | 2.3.6.g |
| **write()** | Écrit des octets | 2.3.6.h |
| **lseek()** | Change la position | 2.3.6.i |
| **close()** | Ferme le fd | 2.3.6.j |
| **fsync()** | Force écriture disque | 2.3.6.k |
| **ftruncate()** | Redimensionne | 2.3.6.l |

### 5.3 Visualisation ASCII

```
                    ARCHITECTURE DES FILE DESCRIPTORS
                    ==================================

    PROCESSUS (USS Enterprise)                    KERNEL (Starfleet Command)
    ==========================                    =========================

    Per-Process FD Table                System File Table           Inode Table
    (Console du Pont)                   (Centre de Contrôle)        (Archives)
    +------------------+                +------------------+        +-----------+
    | fd 0 ──────────────────────────>  | Entry A          |        | inode 42  |
    | (Bridge Input)   |                | offset: 0        |──────> | Regular   |
    +------------------+                | flags: O_RDONLY  |        | size: 8K  |
    | fd 1 ──────────────────+          | refcount: 1      |        +-----------+
    | (Main Viewscreen)|     |          +------------------+
    +------------------+     |          | Entry B          |        +-----------+
    | fd 2 ────────────────────────────>| offset: 1024     |──────> | inode 77  |
    | (Red Alert)      |     |          | flags: O_RDWR    |        | Terminal  |
    +------------------+     +────────> | refcount: 2      |        +-----------+
    | fd 3 ──────────────────────────>  +------------------+
    | (Captain's Log)  |                | Entry C          |
    +------------------+                | offset: 0        |
                                        | flags: O_WRONLY  |
                                        | refcount: 1      |
                                        +------------------+

    IMPORTANT:
    - fd 1 et fd 2 pointent vers la MÊME Entry B (même terminal)
    - Si on écrit sur fd 1, l'offset de Entry B change
    - fd 2 verra le nouvel offset!
```

### 5.8 Mnémotechniques

#### 🖖 MEME : "Make it so" — flush_buffer()

Quand le Capitaine Picard dit "Make it so", l'équipage **exécute immédiatement**. C'est exactement ce que fait `flush_buffer()` : les données en attente sont transmises immédiatement !

```c
send_transmission(ch, "Red Alert!", 10);  // Dans le buffer
flush_buffer(ch);  // "Make it so!" - Transmission immédiate
```

#### 🔴 MEME : "He's dead, Jim" — close() sans flush

Si tu fermes un canal sans flush, les données bufferisées sont **perdues à jamais**. C'est comme couper la communication pendant une transmission.

```c
send_transmission(ch, "Important data", 14);
close_channel(ch);  // ERREUR! Données perdues si pas de flush!

// CORRECT:
send_transmission(ch, "Important data", 14);
flush_buffer(ch);   // D'abord flush
close_channel(ch);  // Puis close
```

#### ⚡ MEME : "Reroute power!" — dup2()

Quand Geordi dit "Reroute power from shields to engines!", il **redirige le flux d'énergie**. `dup2()` fait pareil : redirige un flux d'I/O vers une autre destination.

```c
int log_fd = open("ship.log", O_WRONLY | O_CREAT, 0644);
dup2(log_fd, STDOUT_FILENO);  // Reroute stdout!
printf("This goes to the log file\n");
```

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Conséquence | Solution |
|---|-------|-------------|----------|
| 1 | close() sans flush | Perte de données | Toujours flush avant close |
| 2 | lseek sans invalider buffer | Lecture obsolète | Réinitialiser read_pos/read_end |
| 3 | Confondre EOF et erreur | Comportement incorrect | EOF=0, erreur=-1 |
| 4 | Buffer overflow | Corruption mémoire | Vérifier espace disponible |
| 5 | Oublier O_CREAT avec mode | Permissions aléatoires | Toujours passer mode_t |
| 6 | dup2 sans flush source | Données perdues | Flush avant dup2 |

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 2.3.5-synth : enterprise_io_library |
| **Thème** | Star Trek: The Next Generation |
| **Difficulté** | ★★★★★★☆☆☆☆ (6/10) |
| **Concepts** | 2.3.5.a-k + 2.3.6.a-l (23 concepts) |
| **Fonctions clés** | open, close, read, write, lseek, dup, dup2, fcntl, fsync |
| **Tests** | 20 tests fonctionnels |
| **Mutants** | 6 solutions buggées |
| **XP Base** | 600 |

---

*HACKBRAIN v5.5.2 — Module 2.3.5/2.3.6 : File Descriptors & Operations*
*"Computer, open a channel." — Captain Picard*
*L'excellence pédagogique ne se négocie pas*