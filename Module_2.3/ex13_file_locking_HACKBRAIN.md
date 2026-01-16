# Exercice 2.3.26 : reservoir_locks

**Module :**
2.3.26 — File Locking System

**Concept :**
synth — Synthèse complète (flock, fcntl, deadlock detection, lock inheritance)

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (tous concepts a→m)

**Langage :**
C (C17)

**Prérequis :**
- Descripteurs de fichiers (open, close)
- fork() et comportement parent/enfant
- Structures de données (listes chaînées)
- Graphes et détection de cycles

**Domaines :**
FS, Process, Struct, MD

**Durée estimée :**
300 min

**XP Base :**
450

**Complexité :**
T3 O(n) × S3 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex13/
├── reservoir_lock.h
├── reservoir_lock.c
├── territory_flock.c
├── region_fcntl.c
├── standoff_detector.c
└── Makefile
```

**Fonctions autorisées :**
`flock`, `fcntl`, `malloc`, `free`, `memset`, `memcpy`, `fork`, `getpid`, `open`, `close`, `printf`, `perror`

**Fonctions interdites :**
`lockf` (on réimplémente le mécanisme)

---

### 1.2 Consigne

**🎬 CONTEXTE : RESERVOIR DOGS — Le Braquage du Siècle**

*"Are you gonna bark all day, little doggie, or are you gonna bite?"*

Tu fais partie d'un crew de braqueurs professionnels. Le problème ? Plusieurs équipes tentent de contrôler les mêmes territoires (fichiers) simultanément. Mr. White veut lire le coffre, Mr. Pink veut écrire dedans, et Mr. Orange surveille tout le monde.

Le vrai danger ? Le **Mexican Standoff** — quand Mr. White pointe sur Mr. Pink, Mr. Pink sur Mr. Orange, et Mr. Orange sur Mr. White. Personne ne peut bouger. C'est un **DEADLOCK**.

Ta mission : créer un système de contrôle territorial qui :
1. Permet le **shared stake** (plusieurs peuvent observer)
2. Permet le **exclusive claim** (un seul peut agir)
3. Détecte les **Mexican Standoffs** (deadlocks) avant qu'ils ne paralysent le crew

**Ta mission :**

Implémenter un gestionnaire de verrous complet supportant :
- `grab_territory()` : verrous fichier complet (flock-style)
- `region_control()` : verrous par région (fcntl-style)
- `detect_mexican_standoff()` : détection de deadlocks
- Gestion de l'héritage des locks lors des fork()

**Entrée :**
- `crew` : contexte du gestionnaire de verrous
- `fd` : descripteur de fichier (le "territoire")
- `operation` : type de verrou (shared, exclusive, non-blocking)
- `region` : zone spécifique à verrouiller

**Sortie :**
- Retourne `0` si le verrou est acquis
- Retourne `-1` si échec (EWOULDBLOCK pour non-blocking)
- Détection de deadlock retourne `true` si standoff détecté

**Contraintes :**
- Les verrous SHARED permettent plusieurs détenteurs simultanés
- Les verrous EXCLUSIVE bloquent tous les autres
- Le mode non-blocking retourne immédiatement si verrou indisponible
- La détection de deadlock utilise un graphe d'attente

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `grab_territory(crew, fd, SHARED_STAKE)` | `0` | Mr. White observe le coffre |
| `grab_territory(crew, fd, EXCLUSIVE_CLAIM)` | `0` | Mr. Pink prend le contrôle total |
| `grab_territory(crew, fd, EXCLUSIVE_CLAIM \| QUICK_GRAB)` | `-1` | Territoire déjà pris, pas de blocage |
| `detect_mexican_standoff(crew)` | `true` | A→B→C→A cycle détecté |

---

### 1.2.2 Consigne Académique

Implémenter un système de verrouillage de fichiers complet supportant les APIs flock() et fcntl(). Le système doit gérer les verrous partagés (lecture) et exclusifs (écriture), la détection de deadlocks via un graphe d'attente, et le comportement d'héritage lors des appels fork() et exec().

---

### 1.3 Prototype

```c
#ifndef RESERVOIR_LOCK_H
#define RESERVOIR_LOCK_H

#include <stdint.h>
#include <stdbool.h>
#include <sys/types.h>
#include <unistd.h>

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.26.c-f: FLOCK OPERATIONS — "Grab the Territory"
   ═══════════════════════════════════════════════════════════════════════════ */

// 2.3.26.d: Shared lock — multiple can observe (like multiple cops watching)
#define SHARED_STAKE       0x01

// 2.3.26.e: Exclusive lock — one controls all (like Mr. Pink with the diamonds)
#define EXCLUSIVE_CLAIM    0x02

// Unlock — release territory
#define RELEASE_TERRITORY  0x04

// 2.3.26.f: Non-blocking — quick grab, no waiting
#define QUICK_GRAB         0x08

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.26.g-j: FCNTL COMMANDS — "Region Control"
   ═══════════════════════════════════════════════════════════════════════════ */

// 2.3.26.j: Get lock info — scout the region
#define SCOUT_REGION       0x01

// 2.3.26.h: Set lock non-blocking — claim region fast
#define CLAIM_REGION       0x02

// 2.3.26.i: Set lock blocking — wait for region
#define WAIT_FOR_REGION    0x03

// Lock types for fcntl
#define READER_STAKE       0x00   // Read lock (shared)
#define WRITER_CLAIM       0x01   // Write lock (exclusive)
#define ABANDON_REGION     0x02   // Unlock

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.26.a-b: LOCK ENFORCEMENT MODE
   ═══════════════════════════════════════════════════════════════════════════ */

typedef enum {
    GENTLEMAN_AGREEMENT,   // 2.3.26.a: Advisory — honor system
    GUN_ENFORCED           // 2.3.26.b: Mandatory — kernel enforced
} enforcement_mode_t;

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.26.m: LOCK INHERITANCE — "When the Crew Splits"
   ═══════════════════════════════════════════════════════════════════════════ */

typedef enum {
    CREW_DISPERSES,        // Locks released on fork (default)
    CREW_COPIES_INTEL,     // Locks copied to child
    CREW_SHARES_INTEL      // Locks shared with child
} inheritance_mode_t;

/* ═══════════════════════════════════════════════════════════════════════════
   STRUCTURES
   ═══════════════════════════════════════════════════════════════════════════ */

// 2.3.26.k: Region specification
typedef struct {
    short l_type;          // READER_STAKE, WRITER_CLAIM, ABANDON_REGION
    short l_whence;        // SEEK_SET, SEEK_CUR, SEEK_END
    off_t l_start;         // Start of region
    off_t l_len;           // Length (0 = to EOF)
    pid_t l_holder;        // PID of current holder (Mr. White, Mr. Pink...)
} region_t;

// Lock entry in the ledger
typedef struct lock_entry {
    int fd;                          // The territory (file)
    region_t region;                 // Locked region
    bool is_whole_file;              // flock vs fcntl
    struct lock_entry *next;
} lock_entry_t;

// 2.3.26.l: Deadlock graph — who's pointing at who
typedef struct {
    pid_t gunman;                    // Who's waiting
    pid_t target;                    // Who they're waiting for
} standoff_edge_t;

typedef struct {
    standoff_edge_t *edges;          // All guns pointed
    size_t edge_count;
    size_t capacity;
} standoff_graph_t;

// Main crew context
typedef struct {
    lock_entry_t *ledger;            // All active locks
    size_t lock_count;
    standoff_graph_t *standoff_map;  // Deadlock detection graph
    bool standoff_detection_enabled; // 2.3.26.l
    inheritance_mode_t inheritance;  // 2.3.26.m

    // Statistics
    uint64_t territory_grabs;        // flock calls
    uint64_t region_controls;        // fcntl calls
    uint64_t shared_stakes;          // LOCK_SH count
    uint64_t exclusive_claims;       // LOCK_EX count
    uint64_t quick_grab_fails;       // Non-blocking failures
    uint64_t standoffs_detected;     // Deadlocks found
    double avg_wait_ms;
} reservoir_crew_t;

// Statistics
typedef struct {
    uint64_t territory_grabs;
    uint64_t region_controls;
    uint64_t shared_stakes;
    uint64_t exclusive_claims;
    uint64_t quick_grab_fails;
    uint64_t standoffs_detected;
    double avg_wait_ms;
} crew_stats_t;

/* ═══════════════════════════════════════════════════════════════════════════
   API — "The Heist Protocol"
   ═══════════════════════════════════════════════════════════════════════════ */

// Crew management
reservoir_crew_t *heist_crew_assemble(void);
void heist_crew_disperse(reservoir_crew_t *crew);

// 2.3.26.c-f: flock() style — whole territory control
int grab_territory(reservoir_crew_t *crew, int fd, int operation);

// 2.3.26.g-k: fcntl() style — region control
int region_control(reservoir_crew_t *crew, int fd, int cmd, region_t *region);

// 2.3.26.a-b: Lock enforcement mode
int set_enforcement(reservoir_crew_t *crew, int fd, enforcement_mode_t mode);

// 2.3.26.l: Deadlock detection
void enable_standoff_detection(reservoir_crew_t *crew, bool enable);
bool detect_mexican_standoff(reservoir_crew_t *crew);
void get_standoff_cycle(reservoir_crew_t *crew, pid_t *cycle, size_t *len);

// 2.3.26.m: Inheritance on fork/exec
void set_inheritance_mode(reservoir_crew_t *crew, inheritance_mode_t mode);
int on_crew_split(reservoir_crew_t *crew, pid_t child_pid);
int on_crew_transforms(reservoir_crew_t *crew);

// Query functions
bool is_territory_locked(reservoir_crew_t *crew, int fd, off_t start, off_t len);
int get_territory_holders(reservoir_crew_t *crew, int fd, pid_t *holders, size_t max);
void list_all_claims(reservoir_crew_t *crew);

// Statistics
void get_crew_stats(reservoir_crew_t *crew, crew_stats_t *stats);

#endif /* RESERVOIR_LOCK_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Le Mexican Standoff en Informatique

Le terme "Mexican Standoff" vient des westerns où trois tireurs se pointent mutuellement — personne ne peut tirer sans être tué. En informatique, c'est exactement ce qui se passe avec les **deadlocks** :

```
    Mr. White ─────────────► Mr. Pink
         ▲                      │
         │                      │
         │                      ▼
    Mr. Orange ◄───────────────┘

    DEADLOCK! Personne ne peut avancer.
```

Le noyau Linux utilise exactement cette métaphore de graphe d'attente pour détecter les deadlocks avec `fcntl()`. Un cycle dans le graphe = deadlock.

### 2.2 flock() vs fcntl() : Deux Philosophies

**flock()** (BSD style) :
- Verrou sur fichier ENTIER
- Simple mais grossier
- Comme contrôler toute une banque

**fcntl()** (POSIX style) :
- Verrou par RÉGION (bytes X à Y)
- Précis mais complexe
- Comme contrôler coffre #42 seulement

### 2.3 Advisory vs Mandatory : L'Honneur des Voleurs

**Advisory locks** (défaut) :
- Les processus DOIVENT coopérer
- Si un processus ignore le lock, il peut accéder quand même
- "On est des gentlemen, on respecte les règles"

**Mandatory locks** (rare) :
- Le KERNEL enforce les locks
- Processus non-coopératifs sont bloqués
- Nécessite sgid bit + no group execute
- "Pas de négociation, c'est la loi"

---

### 2.5 DANS LA VRAIE VIE

| Métier | Usage du File Locking | Cas Concret |
|--------|----------------------|-------------|
| **DBA (PostgreSQL/MySQL)** | Lock de pages/rows pour transactions ACID | Éviter les écritures concurrentes corrompues |
| **DevOps** | Lock de fichiers de config pendant déploiement | `/var/lock/dpkg` pendant apt install |
| **SysAdmin** | PID files avec flock() | Un seul daemon à la fois |
| **Développeur Backend** | Sessions utilisateur | Lock fichier session pour éviter race conditions |
| **Game Developer** | Fichiers de sauvegarde | Empêcher corruption pendant save |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
reservoir_lock.c  reservoir_lock.h  territory_flock.c  region_fcntl.c  standoff_detector.c  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -c reservoir_lock.c -o reservoir_lock.o
gcc -Wall -Wextra -Werror -c territory_flock.c -o territory_flock.o
gcc -Wall -Wextra -Werror -c region_fcntl.c -o region_fcntl.o
gcc -Wall -Wextra -Werror -c standoff_detector.c -o standoff_detector.o
gcc -Wall -Wextra -Werror reservoir_lock.o territory_flock.o region_fcntl.o standoff_detector.o main.c -o reservoir_test

$ ./reservoir_test
[CREW] Heist crew assembled (PID: 12345)
[GRAB] Mr. White grabbed SHARED stake on vault.db
[GRAB] Mr. Pink grabbed SHARED stake on vault.db
[GRAB] Mr. Pink upgrading to EXCLUSIVE claim...
[WAIT] Mr. White must release first...
[GRAB] EXCLUSIVE claim acquired!
[REGION] Mr. Orange controlling bytes 100-150 of plans.txt
[SCOUT] Region 100-150 held by PID 12347
[STANDOFF] Detection enabled
[ALERT] Mexican Standoff detected!
[CYCLE] 12345 -> 12346 -> 12347 -> 12345
[SPLIT] Crew splitting (fork)...
[CHILD] Locks released (CREW_DISPERSES mode)
[STATS] Grabs: 5, Exclusive: 2, Standoffs: 1
[CREW] Heist complete. Dispersing.
```

---

### 3.1 💀 BONUS EXPERT : REAL-TIME DEADLOCK PREVENTION (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×4

**Time Complexity attendue :**
O(V+E) pour détection de cycle (DFS)

**Space Complexity attendue :**
O(n) pour le graphe d'attente

**Domaines Bonus :**
`Struct, MD, Probas`

#### 3.1.1 Consigne Bonus

**🎬 NIVEAU TARANTINO : LE STANDOFF PRÉVENTIF**

*"You don't get to point a gun at me unless you're ready to use it."*

Implémente un système de **prévention de deadlock en temps réel** :
- Avant d'accorder un lock, vérifie si cela créerait un cycle
- Si oui, refuse le lock AVANT le blocage
- Implémente l'algorithme du banquier adapté aux locks

**Ta mission :**

Écrire une fonction `prevent_standoff()` qui :
1. Simule l'ajout d'une arête dans le graphe d'attente
2. Détecte si cela créerait un cycle (DFS avec coloration)
3. Retourne `true` si safe, `false` si standoff imminent

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  Graphe : max 1000 nœuds               │
│  Détection : O(V+E) via DFS            │
│  Mémoire : O(n) pour visited array     │
│  Thread-safe : mutex sur le graphe     │
└─────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```c
// Preventive deadlock detection
bool prevent_standoff(reservoir_crew_t *crew, pid_t requester, pid_t holder);
bool would_create_cycle(standoff_graph_t *graph, pid_t from, pid_t to);
int safe_grab_territory(reservoir_crew_t *crew, int fd, int operation);
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Détection | Après deadlock | Avant (prévention) |
| Algorithme | Simple parcours | DFS avec coloration |
| Réponse | Signale le cycle | Refuse le lock |
| Complexité | O(n²) naïf | O(V+E) optimal |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Input | Expected | Points | Concept |
|------|-------|----------|--------|---------|
| `test_gentleman_agreement` | Advisory mode | Locks work but not enforced | 5 | 2.3.26.a |
| `test_gun_enforced` | Mandatory + sgid | Kernel enforces | 5 | 2.3.26.b |
| `test_grab_shared` | SHARED_STAKE | Multiple holders OK | 5 | 2.3.26.c,d |
| `test_grab_exclusive` | EXCLUSIVE_CLAIM | Single holder only | 5 | 2.3.26.c,e |
| `test_quick_grab_fail` | QUICK_GRAB on locked | -1, EWOULDBLOCK | 5 | 2.3.26.f |
| `test_region_claim` | CLAIM_REGION | Non-blocking region lock | 5 | 2.3.26.g,h |
| `test_region_wait` | WAIT_FOR_REGION | Blocking wait | 5 | 2.3.26.g,i |
| `test_region_scout` | SCOUT_REGION | Returns holder PID | 5 | 2.3.26.j |
| `test_region_struct` | region_t fields | Correct offsets | 5 | 2.3.26.k |
| `test_mexican_standoff` | A→B→C→A | Cycle detected | 10 | 2.3.26.l |
| `test_no_standoff` | A→B, C→D | No cycle | 5 | 2.3.26.l |
| `test_crew_disperses` | CREW_DISPERSES + fork | Child has no locks | 5 | 2.3.26.m |
| `test_crew_copies` | CREW_COPIES_INTEL | Child has copy | 5 | 2.3.26.m |
| `test_region_overlap` | Overlapping regions | Correct conflict detection | 5 | Integration |
| `test_upgrade_lock` | SH→EX | Wait then upgrade | 5 | Integration |
| `test_stats` | Various ops | Correct counts | 5 | Integration |
| **TOTAL** | | | **85** | |
| **BONUS** | prevent_standoff | Prevention works | **15** | Bonus |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/wait.h>
#include <errno.h>
#include "reservoir_lock.h"

#define TEST(name) printf("\n[TEST] %s\n", name)
#define OK() printf("  ✓ PASS\n")
#define FAIL(msg) printf("  ✗ FAIL: %s\n", msg)
#define ASSERT(cond, msg) if (!(cond)) { FAIL(msg); return 1; }

int test_grab_shared(void) {
    TEST("grab_territory SHARED_STAKE");

    reservoir_crew_t *crew = heist_crew_assemble();
    ASSERT(crew != NULL, "crew allocation failed");

    int fd = open("/tmp/test_vault.db", O_RDWR | O_CREAT, 0644);
    ASSERT(fd >= 0, "open failed");

    // Multiple shared locks should work
    ASSERT(grab_territory(crew, fd, SHARED_STAKE) == 0, "first shared failed");

    // Fork to test multiple holders
    pid_t pid = fork();
    if (pid == 0) {
        // Child process
        reservoir_crew_t *child_crew = heist_crew_assemble();
        int result = grab_territory(child_crew, fd, SHARED_STAKE);
        heist_crew_disperse(child_crew);
        exit(result == 0 ? 0 : 1);
    }

    int status;
    waitpid(pid, &status, 0);
    ASSERT(WEXITSTATUS(status) == 0, "child shared lock should succeed");

    grab_territory(crew, fd, RELEASE_TERRITORY);
    close(fd);
    unlink("/tmp/test_vault.db");
    heist_crew_disperse(crew);

    OK();
    return 0;
}

int test_grab_exclusive(void) {
    TEST("grab_territory EXCLUSIVE_CLAIM");

    reservoir_crew_t *crew = heist_crew_assemble();
    int fd = open("/tmp/test_vault2.db", O_RDWR | O_CREAT, 0644);

    ASSERT(grab_territory(crew, fd, EXCLUSIVE_CLAIM) == 0, "exclusive failed");

    // Fork to test exclusion
    pid_t pid = fork();
    if (pid == 0) {
        reservoir_crew_t *child_crew = heist_crew_assemble();
        int result = grab_territory(child_crew, fd, EXCLUSIVE_CLAIM | QUICK_GRAB);
        heist_crew_disperse(child_crew);
        // Should fail with EWOULDBLOCK
        exit(result == -1 && errno == EWOULDBLOCK ? 0 : 1);
    }

    int status;
    waitpid(pid, &status, 0);
    ASSERT(WEXITSTATUS(status) == 0, "child exclusive should be blocked");

    grab_territory(crew, fd, RELEASE_TERRITORY);
    close(fd);
    unlink("/tmp/test_vault2.db");
    heist_crew_disperse(crew);

    OK();
    return 0;
}

int test_region_control(void) {
    TEST("region_control CLAIM_REGION");

    reservoir_crew_t *crew = heist_crew_assemble();
    int fd = open("/tmp/test_plans.txt", O_RDWR | O_CREAT, 0644);

    // Write some data
    write(fd, "0123456789ABCDEFGHIJ", 20);

    region_t region = {
        .l_type = WRITER_CLAIM,
        .l_whence = SEEK_SET,
        .l_start = 5,
        .l_len = 10,
        .l_holder = 0
    };

    ASSERT(region_control(crew, fd, CLAIM_REGION, &region) == 0, "region claim failed");

    // Scout the region
    region_t scout = {
        .l_type = WRITER_CLAIM,
        .l_whence = SEEK_SET,
        .l_start = 5,
        .l_len = 10,
        .l_holder = 0
    };

    region_control(crew, fd, SCOUT_REGION, &scout);
    ASSERT(scout.l_holder == getpid(), "holder should be us");

    // Release
    region.l_type = ABANDON_REGION;
    region_control(crew, fd, CLAIM_REGION, &region);

    close(fd);
    unlink("/tmp/test_plans.txt");
    heist_crew_disperse(crew);

    OK();
    return 0;
}

int test_mexican_standoff(void) {
    TEST("detect_mexican_standoff");

    reservoir_crew_t *crew = heist_crew_assemble();
    enable_standoff_detection(crew, true);

    // Manually add edges to create cycle: A→B→C→A
    // This simulates three processes waiting on each other
    // In real usage, these edges are added when a process blocks

    // For testing, we directly manipulate the standoff graph
    // (In production, this happens automatically during blocking)
    standoff_graph_t *graph = crew->standoff_map;

    // A (1000) → B (1001)
    graph->edges[graph->edge_count++] = (standoff_edge_t){1000, 1001};
    // B (1001) → C (1002)
    graph->edges[graph->edge_count++] = (standoff_edge_t){1001, 1002};
    // C (1002) → A (1000) — CYCLE!
    graph->edges[graph->edge_count++] = (standoff_edge_t){1002, 1000};

    ASSERT(detect_mexican_standoff(crew) == true, "should detect cycle");

    pid_t cycle[10];
    size_t len = 0;
    get_standoff_cycle(crew, cycle, &len);
    ASSERT(len == 3, "cycle should have 3 members");

    heist_crew_disperse(crew);

    OK();
    return 0;
}

int test_crew_disperses(void) {
    TEST("inheritance CREW_DISPERSES");

    reservoir_crew_t *crew = heist_crew_assemble();
    set_inheritance_mode(crew, CREW_DISPERSES);

    int fd = open("/tmp/test_inherit.db", O_RDWR | O_CREAT, 0644);
    grab_territory(crew, fd, EXCLUSIVE_CLAIM);

    pid_t pid = fork();
    if (pid == 0) {
        // Child: locks should be released
        on_crew_split(crew, getpid());

        reservoir_crew_t *child_crew = heist_crew_assemble();
        int result = grab_territory(child_crew, fd, EXCLUSIVE_CLAIM | QUICK_GRAB);
        heist_crew_disperse(child_crew);
        exit(result == 0 ? 0 : 1);  // Should succeed (parent lock released)
    }

    // Parent releases lock for fair test
    grab_territory(crew, fd, RELEASE_TERRITORY);

    int status;
    waitpid(pid, &status, 0);
    ASSERT(WEXITSTATUS(status) == 0, "child should get lock after dispersal");

    close(fd);
    unlink("/tmp/test_inherit.db");
    heist_crew_disperse(crew);

    OK();
    return 0;
}

int test_stats(void) {
    TEST("crew statistics");

    reservoir_crew_t *crew = heist_crew_assemble();
    int fd = open("/tmp/test_stats.db", O_RDWR | O_CREAT, 0644);

    grab_territory(crew, fd, SHARED_STAKE);
    grab_territory(crew, fd, RELEASE_TERRITORY);
    grab_territory(crew, fd, EXCLUSIVE_CLAIM);
    grab_territory(crew, fd, RELEASE_TERRITORY);

    crew_stats_t stats;
    get_crew_stats(crew, &stats);

    ASSERT(stats.territory_grabs == 4, "should have 4 grabs");
    ASSERT(stats.shared_stakes == 1, "should have 1 shared");
    ASSERT(stats.exclusive_claims == 1, "should have 1 exclusive");

    close(fd);
    unlink("/tmp/test_stats.db");
    heist_crew_disperse(crew);

    OK();
    return 0;
}

int main(void) {
    printf("╔══════════════════════════════════════════════╗\n");
    printf("║  RESERVOIR LOCKS — The Heist Verification    ║\n");
    printf("╚══════════════════════════════════════════════╝\n");

    int failed = 0;

    failed += test_grab_shared();
    failed += test_grab_exclusive();
    failed += test_region_control();
    failed += test_mexican_standoff();
    failed += test_crew_disperses();
    failed += test_stats();

    printf("\n════════════════════════════════════════════════\n");
    if (failed == 0) {
        printf("All tests passed! The heist was successful.\n");
    } else {
        printf("%d test(s) failed. The crew got caught.\n", failed);
    }

    return failed;
}
```

### 4.3 Solution de référence

```c
/* reservoir_lock.c — Reference Implementation */
#include "reservoir_lock.h"
#include <stdlib.h>
#include <string.h>
#include <sys/file.h>
#include <fcntl.h>
#include <errno.h>

#define MAX_EDGES 1024

/* ═══════════════════════════════════════════════════════════════════════════
   CREW MANAGEMENT
   ═══════════════════════════════════════════════════════════════════════════ */

reservoir_crew_t *heist_crew_assemble(void)
{
    reservoir_crew_t *crew;

    crew = malloc(sizeof(reservoir_crew_t));
    if (crew == NULL)
        return (NULL);

    memset(crew, 0, sizeof(reservoir_crew_t));

    crew->standoff_map = malloc(sizeof(standoff_graph_t));
    if (crew->standoff_map == NULL)
    {
        free(crew);
        return (NULL);
    }

    crew->standoff_map->edges = malloc(sizeof(standoff_edge_t) * MAX_EDGES);
    if (crew->standoff_map->edges == NULL)
    {
        free(crew->standoff_map);
        free(crew);
        return (NULL);
    }

    crew->standoff_map->edge_count = 0;
    crew->standoff_map->capacity = MAX_EDGES;
    crew->inheritance = CREW_DISPERSES;

    return (crew);
}

void heist_crew_disperse(reservoir_crew_t *crew)
{
    lock_entry_t *current;
    lock_entry_t *next;

    if (crew == NULL)
        return;

    current = crew->ledger;
    while (current != NULL)
    {
        next = current->next;
        free(current);
        current = next;
    }

    if (crew->standoff_map != NULL)
    {
        free(crew->standoff_map->edges);
        free(crew->standoff_map);
    }

    free(crew);
}

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.26.c-f: FLOCK-STYLE TERRITORY CONTROL
   ═══════════════════════════════════════════════════════════════════════════ */

int grab_territory(reservoir_crew_t *crew, int fd, int operation)
{
    int flock_op;
    int result;
    lock_entry_t *entry;

    if (crew == NULL || fd < 0)
        return (-1);

    crew->territory_grabs++;

    /* Convert to flock() flags */
    flock_op = 0;
    if (operation & SHARED_STAKE)
    {
        flock_op = LOCK_SH;
        crew->shared_stakes++;
    }
    else if (operation & EXCLUSIVE_CLAIM)
    {
        flock_op = LOCK_EX;
        crew->exclusive_claims++;
    }
    else if (operation & RELEASE_TERRITORY)
    {
        flock_op = LOCK_UN;
    }

    if (operation & QUICK_GRAB)
        flock_op |= LOCK_NB;

    result = flock(fd, flock_op);

    if (result == -1)
    {
        if (errno == EWOULDBLOCK)
            crew->quick_grab_fails++;
        return (-1);
    }

    /* Track lock in ledger */
    if (!(operation & RELEASE_TERRITORY))
    {
        entry = malloc(sizeof(lock_entry_t));
        if (entry != NULL)
        {
            entry->fd = fd;
            entry->is_whole_file = true;
            entry->region.l_holder = getpid();
            entry->region.l_type = (operation & SHARED_STAKE) ?
                                   READER_STAKE : WRITER_CLAIM;
            entry->next = crew->ledger;
            crew->ledger = entry;
            crew->lock_count++;
        }
    }
    else
    {
        /* Remove from ledger on unlock */
        lock_entry_t **pp = &crew->ledger;
        while (*pp != NULL)
        {
            if ((*pp)->fd == fd && (*pp)->is_whole_file)
            {
                entry = *pp;
                *pp = entry->next;
                free(entry);
                crew->lock_count--;
                break;
            }
            pp = &(*pp)->next;
        }
    }

    return (0);
}

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.26.g-k: FCNTL-STYLE REGION CONTROL
   ═══════════════════════════════════════════════════════════════════════════ */

int region_control(reservoir_crew_t *crew, int fd, int cmd, region_t *region)
{
    struct flock fl;
    int fcntl_cmd;
    int result;

    if (crew == NULL || fd < 0 || region == NULL)
        return (-1);

    crew->region_controls++;

    /* Convert region_t to struct flock */
    memset(&fl, 0, sizeof(fl));

    if (region->l_type == READER_STAKE)
        fl.l_type = F_RDLCK;
    else if (region->l_type == WRITER_CLAIM)
        fl.l_type = F_WRLCK;
    else
        fl.l_type = F_UNLCK;

    fl.l_whence = region->l_whence;
    fl.l_start = region->l_start;
    fl.l_len = region->l_len;

    /* Convert command */
    if (cmd == SCOUT_REGION)
        fcntl_cmd = F_GETLK;
    else if (cmd == CLAIM_REGION)
        fcntl_cmd = F_SETLK;
    else if (cmd == WAIT_FOR_REGION)
        fcntl_cmd = F_SETLKW;
    else
        return (-1);

    result = fcntl(fd, fcntl_cmd, &fl);

    if (result == -1)
        return (-1);

    /* For SCOUT_REGION, update region with holder info */
    if (cmd == SCOUT_REGION)
    {
        if (fl.l_type == F_UNLCK)
            region->l_holder = 0;  /* Not locked */
        else
            region->l_holder = fl.l_pid;
    }

    return (0);
}

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.26.l: MEXICAN STANDOFF DETECTION (DEADLOCK)
   ═══════════════════════════════════════════════════════════════════════════ */

void enable_standoff_detection(reservoir_crew_t *crew, bool enable)
{
    if (crew != NULL)
        crew->standoff_detection_enabled = enable;
}

/* DFS cycle detection helper */
static bool dfs_find_cycle(standoff_graph_t *graph, pid_t start, pid_t current,
                           bool *visited, bool *in_stack, pid_t *path,
                           size_t *path_len, size_t max_path)
{
    size_t i;

    /* Find index for current PID (simplified: using PID as index offset) */
    size_t idx = current % 10000;

    if (in_stack[idx])
    {
        /* Found cycle, record path */
        if (path != NULL && *path_len < max_path)
            path[(*path_len)++] = current;
        return (true);
    }

    if (visited[idx])
        return (false);

    visited[idx] = true;
    in_stack[idx] = true;

    if (path != NULL && *path_len < max_path)
        path[(*path_len)++] = current;

    /* Visit all neighbors */
    for (i = 0; i < graph->edge_count; i++)
    {
        if (graph->edges[i].gunman == current)
        {
            if (dfs_find_cycle(graph, start, graph->edges[i].target,
                              visited, in_stack, path, path_len, max_path))
                return (true);
        }
    }

    in_stack[idx] = false;
    if (path != NULL && *path_len > 0)
        (*path_len)--;

    return (false);
}

bool detect_mexican_standoff(reservoir_crew_t *crew)
{
    bool visited[10000] = {false};
    bool in_stack[10000] = {false};
    size_t i;

    if (crew == NULL || !crew->standoff_detection_enabled)
        return (false);

    /* Try DFS from each unique PID */
    for (i = 0; i < crew->standoff_map->edge_count; i++)
    {
        pid_t start = crew->standoff_map->edges[i].gunman;
        size_t path_len = 0;

        if (!visited[start % 10000])
        {
            if (dfs_find_cycle(crew->standoff_map, start, start,
                              visited, in_stack, NULL, &path_len, 0))
            {
                crew->standoffs_detected++;
                return (true);
            }
        }
    }

    return (false);
}

void get_standoff_cycle(reservoir_crew_t *crew, pid_t *cycle, size_t *len)
{
    bool visited[10000] = {false};
    bool in_stack[10000] = {false};
    size_t i;

    if (crew == NULL || cycle == NULL || len == NULL)
        return;

    *len = 0;

    for (i = 0; i < crew->standoff_map->edge_count; i++)
    {
        pid_t start = crew->standoff_map->edges[i].gunman;

        memset(visited, 0, sizeof(visited));
        memset(in_stack, 0, sizeof(in_stack));

        if (dfs_find_cycle(crew->standoff_map, start, start,
                          visited, in_stack, cycle, len, 10))
        {
            return;
        }
    }
}

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.26.m: LOCK INHERITANCE
   ═══════════════════════════════════════════════════════════════════════════ */

void set_inheritance_mode(reservoir_crew_t *crew, inheritance_mode_t mode)
{
    if (crew != NULL)
        crew->inheritance = mode;
}

int on_crew_split(reservoir_crew_t *crew, pid_t child_pid)
{
    lock_entry_t *current;
    lock_entry_t *next;

    if (crew == NULL)
        return (-1);

    if (crew->inheritance == CREW_DISPERSES)
    {
        /* Release all locks */
        current = crew->ledger;
        while (current != NULL)
        {
            next = current->next;
            if (current->is_whole_file)
                flock(current->fd, LOCK_UN);
            else
            {
                struct flock fl = {.l_type = F_UNLCK};
                fcntl(current->fd, F_SETLK, &fl);
            }
            free(current);
            current = next;
        }
        crew->ledger = NULL;
        crew->lock_count = 0;
    }

    (void)child_pid;  /* Used for CREW_SHARES_INTEL tracking */
    return (0);
}

int on_crew_transforms(reservoir_crew_t *crew)
{
    /* exec() clears all locks by default (POSIX behavior) */
    return on_crew_split(crew, 0);
}

/* ═══════════════════════════════════════════════════════════════════════════
   QUERY & STATS
   ═══════════════════════════════════════════════════════════════════════════ */

bool is_territory_locked(reservoir_crew_t *crew, int fd, off_t start, off_t len)
{
    region_t region = {
        .l_type = WRITER_CLAIM,
        .l_whence = SEEK_SET,
        .l_start = start,
        .l_len = len,
        .l_holder = 0
    };

    if (region_control(crew, fd, SCOUT_REGION, &region) == -1)
        return (false);

    return (region.l_holder != 0);
}

void get_crew_stats(reservoir_crew_t *crew, crew_stats_t *stats)
{
    if (crew == NULL || stats == NULL)
        return;

    stats->territory_grabs = crew->territory_grabs;
    stats->region_controls = crew->region_controls;
    stats->shared_stakes = crew->shared_stakes;
    stats->exclusive_claims = crew->exclusive_claims;
    stats->quick_grab_fails = crew->quick_grab_fails;
    stats->standoffs_detected = crew->standoffs_detected;
    stats->avg_wait_ms = crew->avg_wait_ms;
}
```

### 4.4 Solutions alternatives acceptées

```c
/* Alternative 1: Using lockf() internally instead of flock() */
int grab_territory_lockf(reservoir_crew_t *crew, int fd, int operation)
{
    int lockf_cmd;

    if (operation & RELEASE_TERRITORY)
        lockf_cmd = F_ULOCK;
    else if (operation & EXCLUSIVE_CLAIM)
        lockf_cmd = (operation & QUICK_GRAB) ? F_TLOCK : F_LOCK;
    else
        lockf_cmd = F_TEST;  /* For shared, test availability */

    return lockf(fd, lockf_cmd, 0);
}

/* Alternative 2: Bitmap-based cycle detection instead of DFS */
bool detect_standoff_bitmap(standoff_graph_t *graph)
{
    uint64_t reachable[1024] = {0};
    size_t i, j, changed;

    /* Initialize direct edges */
    for (i = 0; i < graph->edge_count; i++)
    {
        size_t from_idx = graph->edges[i].gunman % 1024;
        size_t to_idx = graph->edges[i].target % 1024;
        reachable[from_idx] |= (1ULL << (to_idx % 64));
    }

    /* Transitive closure (Warshall's algorithm) */
    do {
        changed = 0;
        for (i = 0; i < 1024; i++)
        {
            uint64_t old = reachable[i];
            for (j = 0; j < 64; j++)
            {
                if (reachable[i] & (1ULL << j))
                    reachable[i] |= reachable[j];
            }
            if (reachable[i] != old)
                changed = 1;
        }
    } while (changed);

    /* Check for self-loops (cycles) */
    for (i = 0; i < 1024; i++)
    {
        if (reachable[i] & (1ULL << (i % 64)))
            return true;
    }

    return false;
}
```

### 4.5 Solutions refusées (avec explications)

```c
/* REFUSÉ 1: Pas de vérification NULL */
int grab_territory_bad1(reservoir_crew_t *crew, int fd, int operation)
{
    // ❌ crew peut être NULL!
    crew->territory_grabs++;
    return flock(fd, operation);
}
// Pourquoi refusé: Segfault si crew == NULL

/* REFUSÉ 2: Ignorer le résultat de flock() */
int grab_territory_bad2(reservoir_crew_t *crew, int fd, int operation)
{
    flock(fd, LOCK_EX);  // ❌ Ignore le retour!
    return 0;  // Toujours "succès"
}
// Pourquoi refusé: Masque les erreurs, EWOULDBLOCK non géré

/* REFUSÉ 3: Deadlock detection sans graphe */
bool detect_standoff_bad(reservoir_crew_t *crew)
{
    // ❌ Tente juste un lock non-blocking
    if (flock(0, LOCK_EX | LOCK_NB) == -1)
        return true;
    return false;
}
// Pourquoi refusé: Ne détecte pas les vrais cycles A→B→C→A

/* REFUSÉ 4: Fuite mémoire sur disperse */
void heist_crew_disperse_bad(reservoir_crew_t *crew)
{
    free(crew);  // ❌ Oublie ledger et standoff_map!
}
// Pourquoi refusé: Memory leak des sous-structures
```

### 4.6 Solution bonus de référence

```c
/* standoff_preventer.c — Preventive Deadlock Detection */

/* Color enum for DFS */
typedef enum { WHITE, GRAY, BLACK } color_t;

/* Would adding (from → to) create a cycle? */
bool would_create_cycle(standoff_graph_t *graph, pid_t from, pid_t to)
{
    color_t colors[10000];
    pid_t stack[1000];
    size_t stack_top = 0;
    size_t i;

    memset(colors, WHITE, sizeof(colors));

    /* DFS starting from 'to' to see if we can reach 'from' */
    stack[stack_top++] = to;
    colors[to % 10000] = GRAY;

    while (stack_top > 0)
    {
        pid_t current = stack[--stack_top];

        for (i = 0; i < graph->edge_count; i++)
        {
            if (graph->edges[i].gunman == current)
            {
                pid_t neighbor = graph->edges[i].target;

                if (neighbor == from)
                    return true;  /* Found cycle! */

                if (colors[neighbor % 10000] == WHITE)
                {
                    colors[neighbor % 10000] = GRAY;
                    stack[stack_top++] = neighbor;
                }
            }
        }

        colors[current % 10000] = BLACK;
    }

    return false;
}

/* Safe grab: checks for deadlock before blocking */
int safe_grab_territory(reservoir_crew_t *crew, int fd, int operation)
{
    pid_t me = getpid();
    lock_entry_t *entry;

    if (crew == NULL || !crew->standoff_detection_enabled)
        return grab_territory(crew, fd, operation);

    /* For blocking operations, check if it would cause deadlock */
    if (!(operation & QUICK_GRAB))
    {
        /* Find who holds this territory */
        entry = crew->ledger;
        while (entry != NULL)
        {
            if (entry->fd == fd && entry->region.l_holder != me)
            {
                /* Would waiting on this holder create a cycle? */
                if (would_create_cycle(crew->standoff_map, me,
                                       entry->region.l_holder))
                {
                    errno = EDEADLK;
                    return -1;
                }
            }
            entry = entry->next;
        }
    }

    return grab_territory(crew, fd, operation);
}

bool prevent_standoff(reservoir_crew_t *crew, pid_t requester, pid_t holder)
{
    if (crew == NULL || crew->standoff_map == NULL)
        return true;  /* Safe by default */

    return !would_create_cycle(crew->standoff_map, requester, holder);
}
```

### 4.9 spec.json

```json
{
  "name": "reservoir_locks",
  "language": "c",
  "version": "c17",
  "type": "complet",
  "tier": 3,
  "tier_info": "Synthèse (concepts a→m)",
  "tags": ["file-locking", "deadlock", "flock", "fcntl", "phase2", "reservoir-dogs"],
  "passing_score": 70,

  "function": {
    "name": "grab_territory",
    "prototype": "int grab_territory(reservoir_crew_t *crew, int fd, int operation)",
    "return_type": "int",
    "parameters": [
      {"name": "crew", "type": "reservoir_crew_t *"},
      {"name": "fd", "type": "int"},
      {"name": "operation", "type": "int"}
    ]
  },

  "additional_functions": [
    {
      "name": "heist_crew_assemble",
      "prototype": "reservoir_crew_t *heist_crew_assemble(void)",
      "return_type": "reservoir_crew_t *"
    },
    {
      "name": "heist_crew_disperse",
      "prototype": "void heist_crew_disperse(reservoir_crew_t *crew)",
      "return_type": "void"
    },
    {
      "name": "region_control",
      "prototype": "int region_control(reservoir_crew_t *crew, int fd, int cmd, region_t *region)",
      "return_type": "int"
    },
    {
      "name": "detect_mexican_standoff",
      "prototype": "bool detect_mexican_standoff(reservoir_crew_t *crew)",
      "return_type": "bool"
    }
  ],

  "driver": {
    "reference": "int ref_grab_territory(reservoir_crew_t *crew, int fd, int operation) { int flock_op = 0; if (crew == NULL || fd < 0) return (-1); crew->territory_grabs++; if (operation & 0x01) { flock_op = LOCK_SH; crew->shared_stakes++; } else if (operation & 0x02) { flock_op = LOCK_EX; crew->exclusive_claims++; } else if (operation & 0x04) { flock_op = LOCK_UN; } if (operation & 0x08) flock_op |= LOCK_NB; return flock(fd, flock_op); }",

    "edge_cases": [
      {
        "name": "null_crew",
        "args": [null, 3, 1],
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "crew est NULL, doit retourner -1"
      },
      {
        "name": "negative_fd",
        "args": ["valid_crew", -1, 1],
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "fd négatif invalide"
      },
      {
        "name": "shared_lock_success",
        "args": ["valid_crew", "valid_fd", 1],
        "expected": 0,
        "is_trap": false
      },
      {
        "name": "exclusive_lock_success",
        "args": ["valid_crew", "valid_fd", 2],
        "expected": 0,
        "is_trap": false
      },
      {
        "name": "nonblock_would_block",
        "args": ["valid_crew", "locked_fd", 10],
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "EXCLUSIVE|QUICK_GRAB sur fd locké retourne -1"
      },
      {
        "name": "unlock_success",
        "args": ["valid_crew", "valid_fd", 4],
        "expected": 0,
        "is_trap": false
      },
      {
        "name": "cycle_detected",
        "test_function": "detect_mexican_standoff",
        "setup": "create_cycle_graph",
        "expected": true,
        "is_trap": false
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "int",
          "param_index": 1,
          "params": {"min": -10, "max": 1024}
        },
        {
          "type": "int",
          "param_index": 2,
          "params": {"min": 0, "max": 15}
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["flock", "fcntl", "malloc", "free", "memset", "memcpy", "fork", "getpid", "open", "close", "printf", "perror"],
    "forbidden_functions": ["lockf"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  },

  "bonus": {
    "enabled": true,
    "tier": "EXPERT",
    "multiplier": 4,
    "functions": ["prevent_standoff", "would_create_cycle", "safe_grab_territory"]
  }
}
```

### 4.10 Solutions Mutantes

```c
/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT A (Boundary): Off-by-one dans DFS cycle detection
   ═══════════════════════════════════════════════════════════════════════════ */
bool detect_standoff_mutantA(reservoir_crew_t *crew)
{
    size_t i;
    // ❌ Commence à 1 au lieu de 0
    for (i = 1; i < crew->standoff_map->edge_count; i++)
    {
        /* ... miss first edge ... */
    }
    return false;
}
// Pourquoi faux: Rate le premier edge du graphe, peut rater un cycle
// Misconception: Confusion avec indices 1-based

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT B (Safety): Pas de vérification standoff_map NULL
   ═══════════════════════════════════════════════════════════════════════════ */
bool detect_standoff_mutantB(reservoir_crew_t *crew)
{
    // ❌ crew->standoff_map peut être NULL!
    for (size_t i = 0; i < crew->standoff_map->edge_count; i++)
    {
        /* ... crash ... */
    }
    return false;
}
// Pourquoi faux: Segfault si standoff_map non initialisé
// Misconception: Assume que heist_crew_assemble a toujours réussi

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT C (Resource): Fuite mémoire dans ledger
   ═══════════════════════════════════════════════════════════════════════════ */
int grab_territory_mutantC(reservoir_crew_t *crew, int fd, int operation)
{
    lock_entry_t *entry = malloc(sizeof(lock_entry_t));
    entry->fd = fd;
    entry->next = crew->ledger;
    crew->ledger = entry;

    // ❌ Sur RELEASE_TERRITORY, ne libère pas l'entry!
    if (operation & RELEASE_TERRITORY)
    {
        flock(fd, LOCK_UN);
        return 0;  // Entry jamais freed
    }

    return flock(fd, operation);
}
// Pourquoi faux: Memory leak à chaque unlock
// Misconception: Oublie de nettoyer le ledger

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT D (Logic): Inverse SHARED et EXCLUSIVE
   ═══════════════════════════════════════════════════════════════════════════ */
int grab_territory_mutantD(reservoir_crew_t *crew, int fd, int operation)
{
    int flock_op = 0;

    // ❌ LOCK_SH et LOCK_EX inversés!
    if (operation & SHARED_STAKE)
        flock_op = LOCK_EX;  // Devrait être LOCK_SH
    else if (operation & EXCLUSIVE_CLAIM)
        flock_op = LOCK_SH;  // Devrait être LOCK_EX

    return flock(fd, flock_op);
}
// Pourquoi faux: Un "shared" lock bloque tout, un "exclusive" permet multiple
// Misconception: Confusion dans les flags flock()

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT E (Return): Ignore EWOULDBLOCK
   ═══════════════════════════════════════════════════════════════════════════ */
int grab_territory_mutantE(reservoir_crew_t *crew, int fd, int operation)
{
    int result = flock(fd, LOCK_EX | LOCK_NB);

    // ❌ Retourne 0 même si EWOULDBLOCK!
    if (result == -1 && errno == EWOULDBLOCK)
        return 0;  // Masque l'erreur

    return result;
}
// Pourquoi faux: L'appelant pense avoir le lock alors qu'il ne l'a pas
// Misconception: EWOULDBLOCK n'est pas une vraie erreur?

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT F (Integration): Oublie de mettre à jour stats
   ═══════════════════════════════════════════════════════════════════════════ */
int grab_territory_mutantF(reservoir_crew_t *crew, int fd, int operation)
{
    // ❌ Ne met pas à jour crew->territory_grabs++
    // ❌ Ne met pas à jour crew->shared_stakes ou exclusive_claims

    int flock_op = (operation & SHARED_STAKE) ? LOCK_SH : LOCK_EX;
    if (operation & QUICK_GRAB)
        flock_op |= LOCK_NB;

    return flock(fd, flock_op);
}
// Pourquoi faux: get_crew_stats() retourne des valeurs incorrectes
// Misconception: Les stats sont optionnelles
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Deux APIs de locking** : flock() (BSD, simple) vs fcntl() (POSIX, précis)
2. **Shared vs Exclusive** : Lecteurs multiples vs écrivain unique
3. **Détection de deadlock** : Graphe d'attente et cycles
4. **Héritage de locks** : Comportement après fork()/exec()
5. **Advisory vs Mandatory** : Coopération vs enforcement kernel

### 5.2 LDA — Traduction Littérale

```
FONCTION grab_territory QUI RETOURNE UN ENTIER ET PREND EN PARAMÈTRES crew QUI EST UN POINTEUR VERS UNE STRUCTURE reservoir_crew ET fd QUI EST UN ENTIER ET operation QUI EST UN ENTIER
DÉBUT FONCTION
    DÉCLARER flock_op COMME ENTIER
    DÉCLARER result COMME ENTIER

    SI crew EST ÉGAL À NUL OU fd EST INFÉRIEUR À 0 ALORS
        RETOURNER LA VALEUR MOINS 1
    FIN SI

    INCRÉMENTER territory_grabs DE 1 DANS crew

    AFFECTER 0 À flock_op

    SI operation ET SHARED_STAKE EST VRAI ALORS
        AFFECTER LOCK_SH À flock_op
        INCRÉMENTER shared_stakes DE 1 DANS crew
    SINON SI operation ET EXCLUSIVE_CLAIM EST VRAI ALORS
        AFFECTER LOCK_EX À flock_op
        INCRÉMENTER exclusive_claims DE 1 DANS crew
    SINON SI operation ET RELEASE_TERRITORY EST VRAI ALORS
        AFFECTER LOCK_UN À flock_op
    FIN SI

    SI operation ET QUICK_GRAB EST VRAI ALORS
        AFFECTER flock_op OU LOCK_NB À flock_op
    FIN SI

    AFFECTER APPELER flock AVEC fd ET flock_op À result

    SI result EST ÉGAL À MOINS 1 ALORS
        SI errno EST ÉGAL À EWOULDBLOCK ALORS
            INCRÉMENTER quick_grab_fails DE 1 DANS crew
        FIN SI
        RETOURNER LA VALEUR MOINS 1
    FIN SI

    RETOURNER LA VALEUR 0
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : grab_territory
---
1. VÉRIFIER les paramètres (crew != NULL, fd >= 0)
   |
   |-- SI invalide : RETOURNER -1

2. INCRÉMENTER le compteur territory_grabs

3. CONVERTIR operation en flags flock() :
   |
   |-- CAS SHARED_STAKE : flock_op = LOCK_SH
   |-- CAS EXCLUSIVE_CLAIM : flock_op = LOCK_EX
   |-- CAS RELEASE_TERRITORY : flock_op = LOCK_UN

4. SI QUICK_GRAB activé :
   |
   |-- AJOUTER LOCK_NB à flock_op

5. APPELER flock(fd, flock_op)

6. SI échec (result == -1) :
   |
   |-- SI EWOULDBLOCK : incrémenter quick_grab_fails
   |-- RETOURNER -1

7. RETOURNER 0 (succès)
```

### 5.2.3 Diagramme Mermaid (Deadlock Detection)

```mermaid
graph TD
    A[detect_mexican_standoff] --> B{crew != NULL?}
    B -- Non --> C[RETOUR: false]
    B -- Oui --> D{detection enabled?}

    D -- Non --> C
    D -- Oui --> E[Initialiser visited[] et in_stack[]]

    E --> F[Pour chaque edge dans standoff_map]
    F --> G[DFS depuis edge.gunman]

    G --> H{Noeud déjà in_stack?}
    H -- Oui --> I[CYCLE DÉTECTÉ!]
    I --> J[RETOUR: true]

    H -- Non --> K{Déjà visited?}
    K -- Oui --> L[Passer au prochain]
    K -- Non --> M[Marquer GRAY, visiter voisins]

    M --> N{Plus de voisins?}
    N -- Non --> G
    N -- Oui --> O[Marquer BLACK]
    O --> F

    F --> P{Plus d'edges?}
    P -- Oui --> C
```

### 5.3 Visualisation ASCII

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    THE MEXICAN STANDOFF (DEADLOCK)                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                          ║
║     Mr. White (PID 1000)                                                 ║
║           │                                                              ║
║           │ waits for file A                                             ║
║           ▼                                                              ║
║     ┌─────────┐                   ┌─────────┐                           ║
║     │ File A  │◄──────────────────│ File B  │                           ║
║     │(locked) │  held by          │(locked) │                           ║
║     └────┬────┘  Mr. Pink         └────┬────┘                           ║
║          │                              │                                ║
║          │                              │ waits for file B               ║
║          ▼                              ▼                                ║
║     Mr. Pink (PID 1001)           Mr. Orange (PID 1002)                 ║
║           │                              │                               ║
║           │ waits for file C             │ waits for file A              ║
║           ▼                              ▼                               ║
║     ┌─────────┐                   ┌─────────────────────────────┐       ║
║     │ File C  │                   │       WAIT GRAPH            │       ║
║     │(locked) │                   │                             │       ║
║     │by Orange│                   │   1000 ──► 1001 ──► 1002   │       ║
║     └─────────┘                   │     ▲                 │     │       ║
║                                   │     └─────────────────┘     │       ║
║                                   │         CYCLE!              │       ║
║                                   └─────────────────────────────┘       ║
║                                                                          ║
║  DEADLOCK: Personne ne peut avancer. Le noyau doit intervenir.          ║
║                                                                          ║
╚══════════════════════════════════════════════════════════════════════════╝
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **Oubli LOCK_NB** | Blocking infini sur lock occupé | Toujours utiliser QUICK_GRAB pour tester |
| **Shared→Exclusive** | Upgrade peut deadlock avec soi-même | Release shared PUIS acquire exclusive |
| **Fork inheritance** | Enfant peut avoir des locks inattendus | Configurer CREW_DISPERSES |
| **fcntl region overlap** | Deux régions qui se chevauchent | Vérifier [start, start+len) intersection |
| **Graphe d'attente stale** | Edges non nettoyées après unlock | Supprimer edge quand lock libéré |

### 5.5 Cours Complet

#### 5.5.1 Introduction au File Locking

Le verrouillage de fichiers est un mécanisme permettant à plusieurs processus de coordonner leur accès à une ressource partagée (un fichier). Sans locking, deux processus écrivant simultanément peuvent corrompre les données.

#### 5.5.2 Les deux APIs de locking

**flock() — BSD-style (simple)**

```c
#include <sys/file.h>

int flock(int fd, int operation);
// operation: LOCK_SH, LOCK_EX, LOCK_UN, LOCK_NB
```

Caractéristiques :
- Lock sur fichier ENTIER
- Associé au file descriptor (pas au processus)
- Les locks ne survivent pas à close()
- Supporté par NFS (version 3+)

**fcntl() — POSIX-style (précis)**

```c
#include <fcntl.h>

struct flock {
    short l_type;    // F_RDLCK, F_WRLCK, F_UNLCK
    short l_whence;  // SEEK_SET, SEEK_CUR, SEEK_END
    off_t l_start;   // Offset de début
    off_t l_len;     // Longueur (0 = jusqu'à EOF)
    pid_t l_pid;     // PID du holder (pour F_GETLK)
};

int fcntl(int fd, int cmd, struct flock *lock);
// cmd: F_SETLK, F_SETLKW, F_GETLK
```

Caractéristiques :
- Lock par RÉGION (byte range)
- Associé au processus (pas au fd)
- Plus complexe mais plus flexible
- Standard POSIX, portable

#### 5.5.3 Shared vs Exclusive

```
            ┌─────────────────────────────────────────┐
            │            COMPATIBILITY MATRIX          │
            ├─────────────────────────────────────────┤
            │              EXISTING LOCK               │
            │      NONE     SHARED     EXCLUSIVE       │
            ├─────────────────────────────────────────┤
NEW   SHARED│   GRANTED    GRANTED    BLOCKED        │
LOCK        │                                         │
      EXCL  │   GRANTED    BLOCKED    BLOCKED        │
            └─────────────────────────────────────────┘
```

**SHARED (LOCK_SH / F_RDLCK)** :
- Plusieurs processus peuvent détenir un lock shared
- Utilisé pour la lecture
- Bloque les locks exclusifs

**EXCLUSIVE (LOCK_EX / F_WRLCK)** :
- Un seul processus peut détenir le lock
- Utilisé pour l'écriture
- Bloque tous les autres locks

#### 5.5.4 Détection de Deadlock

Un **deadlock** (interblocage) survient quand deux processus (ou plus) s'attendent mutuellement :

```
Process A: détient Lock 1, attend Lock 2
Process B: détient Lock 2, attend Lock 1
```

**Graphe d'attente (Wait-for Graph)** :
- Nœuds = processus
- Arête A→B = "A attend que B libère un lock"
- **Cycle = Deadlock!**

Le noyau Linux détecte les cycles pour fcntl() et retourne EDEADLK.

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ if(flock(fd,LOCK_EX)==-1){return-1;}                           │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ if (flock(fd, LOCK_EX) == -1)                                  │
│ {                                                               │
│     return (-1);                                                │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ • Espaces autour des opérateurs pour lisibilité                 │
│ • Accolades sur lignes séparées (style Allman)                  │
│ • Parenthèses autour de la valeur de retour                     │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace

**Scénario : Mr. White et Mr. Pink veulent le même coffre**

```
┌───────┬─────────────────────────────────────┬────────────────────┬────────────────────┐
│ Étape │ Action                              │ Mr. White (1000)   │ Mr. Pink (1001)    │
├───────┼─────────────────────────────────────┼────────────────────┼────────────────────┤
│   1   │ White: grab SHARED                  │ SHARED on vault.db │                    │
├───────┼─────────────────────────────────────┼────────────────────┼────────────────────┤
│   2   │ Pink: grab SHARED                   │ SHARED on vault.db │ SHARED on vault.db │
├───────┼─────────────────────────────────────┼────────────────────┼────────────────────┤
│   3   │ Pink: grab EXCLUSIVE (upgrade)      │ SHARED on vault.db │ WAITING...         │
├───────┼─────────────────────────────────────┼────────────────────┼────────────────────┤
│   4   │ White: grab EXCLUSIVE               │ WAITING...         │ WAITING...         │
├───────┼─────────────────────────────────────┼────────────────────┼────────────────────┤
│   5   │ ⚠️ DEADLOCK DETECTED!               │                    │                    │
│       │ White waits Pink, Pink waits White  │                    │                    │
├───────┼─────────────────────────────────────┼────────────────────┼────────────────────┤
│   6   │ Kernel returns EDEADLK to Pink      │ WAITING...         │ ERROR: EDEADLK     │
├───────┼─────────────────────────────────────┼────────────────────┼────────────────────┤
│   7   │ Pink releases, White proceeds       │ EXCLUSIVE!         │ (gave up)          │
└───────┴─────────────────────────────────────┴────────────────────┴────────────────────┘
```

### 5.8 Mnémotechniques

#### 🎬 MEME : "RESERVOIR DOGS — Le Mexican Standoff"

Dans la scène culte de Reservoir Dogs, Mr. White, Mr. Pink, et Mr. Orange pointent leurs armes l'un sur l'autre. PERSONNE ne peut tirer sans se faire tuer.

```
     🔫 Mr. White ────────────► 🔫 Mr. Pink
            ▲                        │
            │                        │
            └──────── 🔫 Mr. Orange ◄┘

            DEADLOCK! PERSONNE NE BOUGE!
```

**Pour retenir :**
- **SHARED_STAKE** = Plusieurs observent ("everyone's watching the diamonds")
- **EXCLUSIVE_CLAIM** = Un seul agit ("Mr. Pink keeps the diamonds")
- **QUICK_GRAB** = Essayer sans bloquer ("grab and run")
- **Mexican Standoff** = Cycle dans le graphe d'attente = **DEADLOCK**

#### 💀 MEME : "You shall not BLOCK!"

Comme Gandalf bloque le Balrog, LOCK_NB bloque le blocage lui-même :

```c
// Sans LOCK_NB : "Fly, you fools!" (processus bloqué indéfiniment)
flock(fd, LOCK_EX);

// Avec LOCK_NB : "YOU SHALL NOT BLOCK!"
if (flock(fd, LOCK_EX | LOCK_NB) == -1) {
    // On continue, pas de blocage
}
```

### 5.9 Applications pratiques

| Application | Technique utilisée | Exemple |
|-------------|-------------------|---------|
| **Bases de données** | fcntl() region locks | PostgreSQL row-level locking |
| **PID files** | flock() LOCK_EX | `/var/run/nginx.pid` |
| **Éditeurs de texte** | flock() + swap file | vim `.swp` files |
| **Package managers** | flock() on lock file | `dpkg` uses `/var/lib/dpkg/lock` |
| **Cron jobs** | flock() wrapper | `flock -n /tmp/job.lock ./script.sh` |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Erreur type | Solution |
|---|-------|-------------|----------|
| 1 | **NULL crew** | Segfault | Vérifier `crew != NULL` |
| 2 | **Negative fd** | EBADF | Vérifier `fd >= 0` |
| 3 | **Blocking forever** | Process hangs | Utiliser QUICK_GRAB pour timeout |
| 4 | **Shared→Exclusive deadlock** | Self-deadlock | Release PUIS acquire |
| 5 | **Memory leak** | Ledger entries | Free sur RELEASE_TERRITORY |
| 6 | **Stale wait edges** | Faux deadlocks | Nettoyer graphe après unlock |
| 7 | **Fork avec locks** | Enfant bloqué | CREW_DISPERSES mode |
| 8 | **Region overlap** | Conflits inattendus | Calculer intersection |

---

## 📝 SECTION 7 : QCM

### Q1. Quelle est la différence principale entre flock() et fcntl() ?
- A) flock() est plus rapide
- B) flock() verrouille le fichier entier, fcntl() peut verrouiller une région
- C) fcntl() ne supporte pas les locks exclusifs
- D) flock() est POSIX standard

**Réponse : B**

### Q2. Qu'est-ce qu'un Mexican Standoff en informatique ?
- A) Un algorithme de tri
- B) Un type de lock partagé
- C) Un deadlock (cycle dans le graphe d'attente)
- D) Une technique d'optimisation

**Réponse : C**

### Q3. Que retourne flock(fd, LOCK_EX | LOCK_NB) si le lock n'est pas disponible ?
- A) 0 (succès)
- B) -1 avec errno = EWOULDBLOCK
- C) Bloque indéfiniment
- D) -1 avec errno = EINVAL

**Réponse : B**

### Q4. Quelle structure est utilisée pour les region locks avec fcntl() ?
- A) struct lock
- B) struct flock
- C) struct region
- D) struct fcntl_lock

**Réponse : B**

### Q5. Que se passe-t-il par défaut avec les locks après un fork() ?
- A) Les locks sont copiés à l'enfant
- B) Les locks sont partagés
- C) Les locks flock() sont libérés, fcntl() sont hérités
- D) Tous les locks sont libérés

**Réponse : C** (comportement réel Linux)

### Q6. Quel flag flock() permet à plusieurs processus d'avoir le lock simultanément ?
- A) LOCK_EX
- B) LOCK_SH
- C) LOCK_NB
- D) LOCK_UN

**Réponse : B**

### Q7. Comment le noyau détecte-t-il un deadlock avec fcntl() ?
- A) Timer expiration
- B) Graphe d'attente et détection de cycle
- C) Compteur de locks
- D) Il ne détecte pas

**Réponse : B**

### Q8. Quelle commande fcntl() permet de savoir qui détient un lock ?
- A) F_SETLK
- B) F_SETLKW
- C) F_GETLK
- D) F_QUERY

**Réponse : C**

### Q9. Que signifie l_len = 0 dans struct flock ?
- A) Lock de 0 bytes (invalide)
- B) Lock de la position actuelle jusqu'à EOF
- C) Unlock
- D) Lock du fichier entier depuis le début

**Réponse : B**

### Q10. Qu'est-ce qu'un advisory lock ?
- A) Un lock qui conseille mais n'enforce pas
- B) Un lock sur les métadonnées
- C) Un lock temporaire
- D) Un lock recommandé par le système

**Réponse : A**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Concept | Maîtrisé | À revoir |
|---------|----------|----------|
| flock() API (c-f) | ☐ | ☐ |
| fcntl() region locks (g-k) | ☐ | ☐ |
| SHARED vs EXCLUSIVE | ☐ | ☐ |
| Non-blocking mode | ☐ | ☐ |
| Deadlock detection | ☐ | ☐ |
| Wait-for graph | ☐ | ☐ |
| Lock inheritance (fork/exec) | ☐ | ☐ |
| Advisory vs Mandatory | ☐ | ☐ |

**Score minimum pour valider : 70/100**

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.26-synth-reservoir-locks",
    "generated_at": "2026-01-12 14:30:00",

    "metadata": {
      "exercise_id": "2.3.26-synth",
      "exercise_name": "reservoir_locks",
      "module": "2.3.26",
      "module_name": "File Locking System",
      "concept": "synth",
      "concept_name": "Complete file locking (flock+fcntl+deadlock)",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthèse (concepts a→m)",
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "c",
      "language_version": "c17",
      "duration_minutes": 300,
      "xp_base": 450,
      "xp_bonus_multiplier": 4,
      "bonus_tier": "EXPERT",
      "bonus_icon": "💀",
      "complexity_time": "T3 O(n)",
      "complexity_space": "S3 O(n)",
      "prerequisites": ["file-descriptors", "fork", "linked-lists", "graphs"],
      "domains": ["FS", "Process", "Struct", "MD"],
      "domains_bonus": ["Probas"],
      "tags": ["file-locking", "deadlock", "flock", "fcntl", "reservoir-dogs"],
      "meme_reference": "Reservoir Dogs Mexican Standoff"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/reservoir_lock.c": "/* Section 4.3 */",
      "references/reservoir_lock_bonus.c": "/* Section 4.6 */",
      "alternatives/lockf_based.c": "/* Section 4.4 alt 1 */",
      "alternatives/bitmap_cycle.c": "/* Section 4.4 alt 2 */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 */",
      "mutants/mutant_e_return.c": "/* Section 4.10 */",
      "mutants/mutant_f_stats.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/reservoir_lock.c",
        "references/reservoir_lock_bonus.c",
        "alternatives/lockf_based.c",
        "alternatives/bitmap_cycle.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c",
        "mutants/mutant_f_stats.c"
      ]
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "Like Tears in Rain, Unlocked Files are Lost"*
*Reservoir Locks: Because every heist needs proper territory control*
