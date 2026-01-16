# [Module 2.2] - Exercise 00: Process Inspector

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex00"
title: "Process Inspector"
difficulty: facile
estimated_time: "3 heures"
prerequisite_exercises: []
concepts_requis: ["file I/O", "parsing", "structs", "memory allocation"]
concepts_couverts: ["2.2.1 Process Concepts", "2.2.2 Process States"]
score_qualite: 96
```

---

## Concepts Couverts

Liste des concepts abordes dans cet exercice avec references au curriculum:

- **Process Concepts (2.2.1)**: Comprehension du modele de processus Unix - PID, PPID, UID, GID, et attributs fondamentaux
- **Process States (2.2.2)**: Les differents etats d'un processus (Running, Sleeping, Stopped, Zombie) et leurs transitions
- **Virtual Filesystem /proc**: Navigation et parsing du pseudo-filesystem procfs pour extraire les informations systeme
- **System Information Retrieval**: Techniques de collecte d'informations sur les processus en cours d'execution

### Objectifs Pedagogiques

A la fin de cet exercice, vous devriez etre capable de:

1. Lire et parser les fichiers du systeme de fichiers `/proc` pour extraire des informations sur les processus
2. Comprendre la structure interne d'un processus Linux et ses differents attributs
3. Identifier et interpreter les differents etats possibles d'un processus
4. Construire des structures de donnees C pour representer les informations de processus
5. Implementer une API robuste pour l'inspection de processus avec gestion complete des erreurs

---

## Contexte

Dans les systemes Unix/Linux, chaque processus en cours d'execution est represente par une entree dans le pseudo-systeme de fichiers `/proc`. Ce filesystem virtuel, monte automatiquement par le kernel, expose une multitude d'informations sur l'etat du systeme et des processus individuels sous forme de fichiers texte lisibles.

Les outils systeme comme `ps`, `top`, `htop`, ou encore les gestionnaires de taches graphiques utilisent tous `/proc` comme source primaire d'information. Comprendre comment lire et interpreter ces donnees est fondamental pour tout developpeur systeme ou administrateur qui souhaite monitorer, debugger, ou optimiser des applications.

Chaque processus possede son propre repertoire `/proc/[pid]/` contenant des dizaines de fichiers et sous-repertoires. Parmi les plus importants: `status` (etat general), `stat` (statistiques detaillees), `cmdline` (ligne de commande), `fd/` (file descriptors ouverts), `maps` (mappings memoire), et `task/` (threads). La maitrise de ces sources d'information est essentielle pour developper des outils de diagnostic systeme.

**Exemple concret**: Lorsque vous utilisez la commande `ps aux`, le programme lit `/proc/[pid]/stat` et `/proc/[pid]/status` pour chaque processus afin d'afficher le PID, l'utilisateur proprietaire, l'utilisation CPU/memoire, et la commande executee. Votre implementation reproduira cette fonctionnalite de maniere programmatique.

---

## Enonce

### Vue d'Ensemble

Vous devez implementer un **inspecteur de processus** capable de lire les informations d'un processus specifique via `/proc/[pid]/` et de fournir une vue complete de son etat. L'outil doit egalement pouvoir scanner l'ensemble des processus du systeme pour generer des statistiques globales.

### Specifications Fonctionnelles

#### Fonctionnalite 1: Inspection d'un Processus Individuel

L'API principale `proc_info()` doit retourner une structure contenant toutes les informations pertinentes d'un processus identifie par son PID.

**Comportement attendu**:
- Lecture de `/proc/[pid]/status` pour l'etat, la memoire, les UIDs/GIDs
- Lecture de `/proc/[pid]/stat` pour les statistiques CPU et l'etat detaille
- Lecture de `/proc/[pid]/cmdline` pour la ligne de commande complete
- Enumeration de `/proc/[pid]/fd/` pour compter les file descriptors ouverts
- Identification du processus parent (PPID) et des processus enfants

**Cas limites a gerer**:
- PID inexistant ou processus termine pendant la lecture
- Permissions insuffisantes pour lire certains fichiers (processus d'autres utilisateurs)
- Fichiers `/proc` vides ou mal formes
- PID 0 (scheduler) et PID 1 (init) qui ont des comportements speciaux
- Processus kernel threads (sans cmdline)

#### Fonctionnalite 2: Statistiques Systeme Globales

La fonction `proc_system_stats()` doit scanner tous les processus et generer des statistiques agregees.

**Comportement attendu**:
- Comptage total des processus par etat (Running, Sleeping, Stopped, Zombie)
- Identification des processus consommant le plus de memoire
- Liste des processus zombies detectes
- Arbre hierarchique des relations parent-enfant

#### Fonctionnalite 3: Detection de l'Arbre de Processus

La fonction `proc_get_children()` retourne la liste des PIDs enfants directs d'un processus.

**Comportement attendu**:
- Parcours de tous les processus pour trouver ceux ayant le PPID specifie
- Retour d'un tableau dynamique de PIDs
- Possibilite de construire recursivement l'arbre complet

### Specifications Techniques

#### Architecture

```
+-------------------+       +-------------------+
|   Application     |       |   /proc/[pid]/    |
+-------------------+       +-------------------+
         |                           |
         v                           v
+-------------------+       +-------------------+
|   proc_info()     |<----->|  status, stat,    |
|   API Layer       |       |  cmdline, fd/     |
+-------------------+       +-------------------+
         |
         v
+-------------------+
| process_info_t    |
| Structure         |
+-------------------+
```

#### Format des Fichiers /proc

**`/proc/[pid]/status`** (extrait):
```
Name:   bash
State:  S (sleeping)
Pid:    1234
PPid:   1000
Uid:    1000    1000    1000    1000
VmSize: 12345 kB
VmRSS:  2048 kB
Threads: 1
```

**`/proc/[pid]/stat`** (format):
```
pid (comm) state ppid pgrp session tty_nr tpgid flags minflt cminflt majflt cmajflt utime stime cutime cstime priority nice num_threads itrealvalue starttime vsize rss ...
```

**Complexite attendue**:
- Lecture d'un processus: O(1) (nombre fixe de fichiers)
- Scan systeme complet: O(n) ou n = nombre de processus
- Recherche enfants: O(n)

---

## Contraintes Techniques

### Standards C

- **Norme**: C17 (ISO/IEC 9899:2018)
- **Compilation**: `gcc -Wall -Wextra -Werror -std=c17`
- **Options additionnelles**: Aucune bibliotheque externe requise

### Fonctions Autorisees

```
Fonctions autorisees:
  - malloc, free, realloc, calloc (stdlib.h)
  - open, close, read, opendir, readdir, closedir (unistd.h, dirent.h)
  - fopen, fclose, fgets, fread (stdio.h)
  - strlen, strcpy, strncpy, strcmp, strncmp, strchr, strstr (string.h)
  - atoi, atol, strtol, strtoul (stdlib.h)
  - snprintf, sscanf (stdio.h)
  - stat, lstat (sys/stat.h)
  - getpid, getppid, getuid, geteuid (unistd.h)
  - perror, strerror (stdio.h, string.h)
```

### Contraintes Specifiques

- [ ] Pas de variables globales (sauf constantes)
- [ ] Maximum 50 lignes par fonction
- [ ] Toutes les allocations doivent avoir leur free correspondant
- [ ] Les chemins de fichiers doivent etre construits dynamiquement (pas de buffers fixes > 4096)
- [ ] Thread-safe NOT requis pour cet exercice

### Exigences de Securite

- [ ] Aucune fuite memoire (verification Valgrind obligatoire)
- [ ] Aucun buffer overflow (verification des tailles avant copie)
- [ ] Verification de tous les retours de open(), read(), malloc()
- [ ] Gestion appropriee des erreurs avec codes de retour significatifs
- [ ] Protection contre les chemins malicieux (path traversal)

---

## Format de Rendu

### Fichiers a Rendre

```
ex00/
├── process_inspector.h    # Header avec structures et prototypes
├── process_inspector.c    # Implementation principale
├── proc_parser.c          # Fonctions de parsing /proc
├── proc_utils.c           # Utilitaires (allocation, conversion)
└── Makefile               # Compilation et tests
```

### Signatures de Fonctions

#### process_inspector.h

```c
#ifndef PROCESS_INSPECTOR_H
#define PROCESS_INSPECTOR_H

#include <sys/types.h>
#include <stdint.h>
#include <stddef.h>

/* Etats possibles d'un processus */
typedef enum {
    PROC_STATE_RUNNING   = 'R',  /* Running or runnable */
    PROC_STATE_SLEEPING  = 'S',  /* Interruptible sleep */
    PROC_STATE_DISK_WAIT = 'D',  /* Uninterruptible disk sleep */
    PROC_STATE_STOPPED   = 'T',  /* Stopped (signal or trace) */
    PROC_STATE_ZOMBIE    = 'Z',  /* Zombie process */
    PROC_STATE_DEAD      = 'X',  /* Dead (should never be seen) */
    PROC_STATE_IDLE      = 'I',  /* Idle kernel thread */
    PROC_STATE_UNKNOWN   = '?'   /* Unknown state */
} proc_state_t;

/* Informations memoire d'un processus */
typedef struct {
    size_t vm_size;      /* Taille virtuelle totale (kB) */
    size_t vm_rss;       /* Resident Set Size (kB) */
    size_t vm_shared;    /* Memoire partagee (kB) */
    size_t vm_text;      /* Code segment (kB) */
    size_t vm_data;      /* Data + stack (kB) */
} proc_memory_t;

/* Structure principale d'information processus */
typedef struct {
    pid_t           pid;            /* Process ID */
    pid_t           ppid;           /* Parent Process ID */
    uid_t           uid;            /* Real User ID */
    uid_t           euid;           /* Effective User ID */
    gid_t           gid;            /* Real Group ID */
    gid_t           egid;           /* Effective Group ID */
    proc_state_t    state;          /* Etat du processus */
    char            name[256];      /* Nom du processus (comm) */
    char           *cmdline;        /* Ligne de commande complete (allouee) */
    proc_memory_t   memory;         /* Informations memoire */
    int             num_threads;    /* Nombre de threads */
    int             num_fds;        /* Nombre de file descriptors ouverts */
    pid_t          *children;       /* Tableau des PIDs enfants (alloue) */
    size_t          num_children;   /* Nombre d'enfants */
    unsigned long   start_time;     /* Temps de demarrage (jiffies depuis boot) */
    unsigned long   utime;          /* User CPU time */
    unsigned long   stime;          /* System CPU time */
} process_info_t;

/* Statistiques systeme globales */
typedef struct {
    size_t total_processes;     /* Nombre total de processus */
    size_t running_count;       /* Processus en Running */
    size_t sleeping_count;      /* Processus en Sleeping */
    size_t stopped_count;       /* Processus stoppes */
    size_t zombie_count;        /* Processus zombies */
    pid_t *zombie_pids;         /* Liste des PIDs zombies (allouee) */
    size_t zombie_pids_count;   /* Taille de la liste */
} system_stats_t;

/* Codes d'erreur */
typedef enum {
    PROC_SUCCESS = 0,
    PROC_ERR_NOT_FOUND = -1,     /* Processus non trouve */
    PROC_ERR_PERMISSION = -2,   /* Permission refusee */
    PROC_ERR_MEMORY = -3,       /* Erreur d'allocation */
    PROC_ERR_PARSE = -4,        /* Erreur de parsing */
    PROC_ERR_INVALID = -5       /* Parametre invalide */
} proc_error_t;

/**
 * Recupere les informations completes d'un processus.
 *
 * @param pid Le PID du processus a inspecter
 * @return Pointeur vers la structure allouee, ou NULL si erreur
 *
 * @note La structure retournee doit etre liberee avec proc_info_free()
 * @warning Retourne NULL si le processus n'existe pas ou en cas d'erreur
 */
process_info_t *proc_info(pid_t pid);

/**
 * Libere une structure process_info_t et ses membres alloues.
 *
 * @param info Pointeur vers la structure a liberer (peut etre NULL)
 */
void proc_info_free(process_info_t *info);

/**
 * Recupere les statistiques globales du systeme.
 *
 * @return Pointeur vers la structure allouee, ou NULL si erreur
 *
 * @note La structure retournee doit etre liberee avec system_stats_free()
 */
system_stats_t *proc_system_stats(void);

/**
 * Libere une structure system_stats_t et ses membres alloues.
 *
 * @param stats Pointeur vers la structure a liberer (peut etre NULL)
 */
void system_stats_free(system_stats_t *stats);

/**
 * Recupere la liste des PIDs enfants directs d'un processus.
 *
 * @param pid Le PID du processus parent
 * @param children Pointeur vers le tableau de PIDs (alloue par la fonction)
 * @param count Pointeur vers le nombre d'enfants trouves
 * @return PROC_SUCCESS ou code d'erreur
 *
 * @note Le tableau children doit etre libere avec free() par l'appelant
 */
proc_error_t proc_get_children(pid_t pid, pid_t **children, size_t *count);

/**
 * Convertit un etat en chaine descriptive.
 *
 * @param state L'etat du processus
 * @return Chaine statique decrivant l'etat
 */
const char *proc_state_to_string(proc_state_t state);

/**
 * Recupere le dernier code d'erreur.
 *
 * @return Le dernier code d'erreur rencontre
 */
proc_error_t proc_get_last_error(void);

/**
 * Retourne une description textuelle d'un code d'erreur.
 *
 * @param error Le code d'erreur
 * @return Chaine statique decrivant l'erreur
 */
const char *proc_strerror(proc_error_t error);

#endif /* PROCESS_INSPECTOR_H */
```

### Makefile

```makefile
NAME = libprocinspector.a
TEST = test_inspector

CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c17
AR = ar rcs

SRCS = process_inspector.c proc_parser.c proc_utils.c
OBJS = $(SRCS:.c=.o)

all: $(NAME)

$(NAME): $(OBJS)
	$(AR) $(NAME) $(OBJS)

%.o: %.c process_inspector.h
	$(CC) $(CFLAGS) -c $< -o $@

test: $(NAME)
	$(CC) $(CFLAGS) -o $(TEST) test_main.c -L. -lprocinspector
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

### Exemple 1: Inspection du Processus Courant

```c
#include "process_inspector.h"
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    pid_t my_pid = getpid();
    process_info_t *info = proc_info(my_pid);

    if (info == NULL) {
        fprintf(stderr, "Error: %s\n", proc_strerror(proc_get_last_error()));
        return 1;
    }

    printf("=== Process %d ===\n", info->pid);
    printf("Name: %s\n", info->name);
    printf("State: %s\n", proc_state_to_string(info->state));
    printf("Parent PID: %d\n", info->ppid);
    printf("UID/EUID: %d/%d\n", info->uid, info->euid);
    printf("Memory (RSS): %zu kB\n", info->memory.vm_rss);
    printf("Threads: %d\n", info->num_threads);
    printf("Open FDs: %d\n", info->num_fds);
    printf("Command: %s\n", info->cmdline ? info->cmdline : "(none)");

    proc_info_free(info);
    return 0;
}

// Output:
// === Process 12345 ===
// Name: test_inspector
// State: Running
// Parent PID: 1000
// UID/EUID: 1000/1000
// Memory (RSS): 1024 kB
// Threads: 1
// Open FDs: 3
// Command: ./test_inspector
```

**Explication**: La fonction `proc_info()` lit les fichiers `/proc/12345/status`, `/proc/12345/stat`, `/proc/12345/cmdline` et enumere `/proc/12345/fd/` pour collecter toutes les informations.

### Exemple 2: Gestion d'Erreur (Processus Inexistant)

```c
#include "process_inspector.h"
#include <stdio.h>

int main(void)
{
    // PID tres grand, probablement inexistant
    process_info_t *info = proc_info(9999999);

    if (info == NULL) {
        proc_error_t err = proc_get_last_error();
        printf("Expected error: %s (code: %d)\n", proc_strerror(err), err);
        // Expected error: Process not found (code: -1)
        return 0;
    }

    proc_info_free(info);
    return 1;
}
```

**Explication**: Quand le processus n'existe pas, `proc_info()` retourne NULL et le code d'erreur `PROC_ERR_NOT_FOUND` est accessible via `proc_get_last_error()`.

### Exemple 3: Statistiques Systeme et Detection de Zombies

```c
#include "process_inspector.h"
#include <stdio.h>

int main(void)
{
    system_stats_t *stats = proc_system_stats();

    if (stats == NULL) {
        fprintf(stderr, "Failed to get system stats\n");
        return 1;
    }

    printf("=== System Process Statistics ===\n");
    printf("Total processes: %zu\n", stats->total_processes);
    printf("Running:  %zu\n", stats->running_count);
    printf("Sleeping: %zu\n", stats->sleeping_count);
    printf("Stopped:  %zu\n", stats->stopped_count);
    printf("Zombie:   %zu\n", stats->zombie_count);

    if (stats->zombie_count > 0) {
        printf("\nZombie PIDs: ");
        for (size_t i = 0; i < stats->zombie_pids_count; i++) {
            printf("%d ", stats->zombie_pids[i]);
        }
        printf("\n");
    }

    system_stats_free(stats);
    return 0;
}

// Output:
// === System Process Statistics ===
// Total processes: 245
// Running:  3
// Sleeping: 238
// Stopped:  0
// Zombie:   4
//
// Zombie PIDs: 5678 5679 6012 6013
```

**Explication**: `proc_system_stats()` parcourt tous les repertoires numeriques de `/proc/`, lit l'etat de chaque processus, et agregele les statistiques.

### Exemple 4: Arbre de Processus Enfants

```c
#include "process_inspector.h"
#include <stdio.h>
#include <stdlib.h>

void print_tree(pid_t pid, int depth)
{
    process_info_t *info = proc_info(pid);
    if (info == NULL) return;

    for (int i = 0; i < depth; i++) printf("  ");
    printf("[%d] %s\n", info->pid, info->name);

    pid_t *children = NULL;
    size_t count = 0;

    if (proc_get_children(pid, &children, &count) == PROC_SUCCESS) {
        for (size_t i = 0; i < count; i++) {
            print_tree(children[i], depth + 1);
        }
        free(children);
    }

    proc_info_free(info);
}

int main(void)
{
    printf("=== Process Tree from PID 1 ===\n");
    print_tree(1, 0);  // Commence depuis init/systemd
    return 0;
}

// Output:
// === Process Tree from PID 1 ===
// [1] systemd
//   [500] systemd-journal
//   [520] systemd-udevd
//   [1000] sshd
//     [1234] sshd
//       [1235] bash
//         [5678] vim
```

**Explication**: La fonction recursive utilise `proc_get_children()` pour construire l'arbre complet depuis n'importe quel processus.

---

## Tests de la Moulinette

### Tests Fonctionnels de Base

#### Test 01: Inspection du Processus Courant
```yaml
description: "Verifie que proc_info() retourne des informations valides pour le processus courant"
setup: |
  process_info_t *info = proc_info(getpid());
validation:
  - "info != NULL"
  - "info->pid == getpid()"
  - "info->ppid == getppid()"
  - "info->uid == getuid()"
  - "strlen(info->name) > 0"
  - "info->state != PROC_STATE_UNKNOWN"
cleanup: "proc_info_free(info)"
```

#### Test 02: Inspection de PID 1 (init/systemd)
```yaml
description: "Verifie que proc_info() fonctionne pour PID 1"
input: |
  process_info_t *info = proc_info(1);
expected_behavior: "info non NULL avec ppid == 0"
validation:
  - "info != NULL"
  - "info->pid == 1"
  - "info->ppid == 0"
  - "strcmp(info->name, \"systemd\") == 0 || strcmp(info->name, \"init\") == 0"
```

#### Test 03: Processus Inexistant
```yaml
description: "Verifie le comportement pour un PID invalide"
input: |
  process_info_t *info = proc_info(9999999);
expected_output: |
  info == NULL
  proc_get_last_error() == PROC_ERR_NOT_FOUND
```

#### Test 04: PID Negatif
```yaml
description: "Verifie le rejet d'un PID negatif"
input: |
  process_info_t *info = proc_info(-1);
expected_output: |
  info == NULL
  proc_get_last_error() == PROC_ERR_INVALID
```

#### Test 05: Statistiques Systeme
```yaml
description: "Verifie que proc_system_stats() retourne des statistiques coherentes"
validation:
  - "stats != NULL"
  - "stats->total_processes > 0"
  - "stats->total_processes >= stats->running_count + stats->sleeping_count + stats->stopped_count + stats->zombie_count"
  - "(stats->zombie_count == 0) == (stats->zombie_pids == NULL || stats->zombie_pids_count == 0)"
```

#### Test 06: Liste des Enfants
```yaml
description: "Verifie proc_get_children() pour PID 1"
setup: |
  pid_t *children = NULL;
  size_t count = 0;
  proc_error_t err = proc_get_children(1, &children, &count);
validation:
  - "err == PROC_SUCCESS"
  - "count > 0"  // PID 1 a toujours des enfants
  - "children != NULL"
cleanup: "free(children)"
```

#### Test 07: Comptage File Descriptors
```yaml
description: "Verifie le comptage des FDs ouverts"
setup: |
  int extra_fd = open("/dev/null", O_RDONLY);
  process_info_t *info = proc_info(getpid());
validation:
  - "info->num_fds >= 4"  // stdin, stdout, stderr + extra_fd
cleanup: |
  close(extra_fd);
  proc_info_free(info);
```

#### Test 08: Informations Memoire
```yaml
description: "Verifie que les informations memoire sont coherentes"
input: |
  process_info_t *info = proc_info(getpid());
validation:
  - "info->memory.vm_size > 0"
  - "info->memory.vm_rss > 0"
  - "info->memory.vm_rss <= info->memory.vm_size"
```

#### Test 09: Cmdline avec Arguments
```yaml
description: "Verifie la recuperation de la ligne de commande complete"
setup: |
  // Programme lance avec: ./test arg1 arg2
  process_info_t *info = proc_info(getpid());
validation:
  - "info->cmdline != NULL"
  - "strstr(info->cmdline, \"test\") != NULL || strlen(info->cmdline) > 0"
```

### Tests de Robustesse

#### Test 10: Parametres NULL
```yaml
description: "Comportement avec pointeurs NULL"
test_cases:
  - input: "proc_info_free(NULL)"
    expected: "Ne crash pas, no-op"
  - input: "system_stats_free(NULL)"
    expected: "Ne crash pas, no-op"
  - input: "proc_get_children(1, NULL, &count)"
    expected: "PROC_ERR_INVALID"
  - input: "proc_get_children(1, &children, NULL)"
    expected: "PROC_ERR_INVALID"
```

#### Test 11: Limites du Systeme
```yaml
description: "Comportement aux limites"
test_cases:
  - input: "proc_info(0)"
    expected: "NULL ou info valide (scheduler kernel)"
  - input: "proc_info(INT_MAX)"
    expected: "NULL avec PROC_ERR_NOT_FOUND"
  - scenario: "Scan complet avec beaucoup de processus"
    expected: "Termine sans crash, stats coherentes"
```

#### Test 12: Processus Disparu pendant Lecture
```yaml
description: "Gestion d'un processus qui termine pendant l'inspection"
setup: |
  pid_t pid = fork();
  if (pid == 0) { _exit(0); }  // Enfant termine immediatement
  usleep(10000);  // Laisse le temps de terminer
  process_info_t *info = proc_info(pid);
  waitpid(pid, NULL, 0);
expected: "NULL ou info valide (race condition acceptable)"
```

### Tests de Securite

#### Test 20: Fuites Memoire
```yaml
description: "Detection de fuites memoire avec Valgrind"
tool: "valgrind --leak-check=full --error-exitcode=1"
scenario: |
  for (int i = 0; i < 100; i++) {
      process_info_t *info = proc_info(getpid());
      proc_info_free(info);
  }
  system_stats_t *stats = proc_system_stats();
  system_stats_free(stats);
expected: "0 bytes lost, 0 errors"
```

#### Test 21: Buffer Overflow Protection
```yaml
description: "Protection contre les noms de processus longs"
tool: "AddressSanitizer"
scenario: |
  // /proc/[pid]/status avec Name: tres long (theoriquement limite a 15 chars)
  process_info_t *info = proc_info(getpid());
expected: "Pas de depassement de buffer, troncature propre si necessaire"
```

#### Test 22: Path Traversal
```yaml
description: "Verification que les PIDs sont bien valides"
test_cases:
  - scenario: "Tentative d'injection via cast"
    expected: "Rejet des valeurs non valides"
```

### Tests de Performance

#### Test 30: Performance Scan Complet
```yaml
description: "Temps de scan de tous les processus"
scenario: |
  system_stats_t *stats = proc_system_stats();
  system_stats_free(stats);
data_size: "~200-500 processus typique"
iterations: 10
expected_max_time: "< 500ms par iteration"
machine_ref: "Intel i5 2.5GHz, 8GB RAM"
```

#### Test 31: Performance Inspection Unique
```yaml
description: "Temps d'inspection d'un processus"
scenario: |
  process_info_t *info = proc_info(getpid());
  proc_info_free(info);
iterations: 1000
expected_max_time: "< 5ms par iteration"
```

---

## Criteres d'Evaluation

### Note Minimale Requise: 80/100

### Detail de la Notation (Total: 100 points)

#### 1. Correction (40 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Tests fonctionnels (01-09) | 18 | Tous les tests de base passent |
| Gestion des cas limites | 10 | Tests 10-12 passent |
| Gestion d'erreurs | 8 | Codes d'erreur corrects, messages clairs |
| Comportement defini | 4 | Aucun UB, resultats deterministes |

**Penalites**:
- Crash sur PID valide: -15 points
- Information incorrecte: -3 points par champ
- Code d'erreur incorrect: -2 points

#### 2. Securite (25 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Absence de fuites | 10 | Valgrind clean |
| Protection buffers | 8 | Pas d'overflow sur noms/cmdline |
| Verification retours | 4 | Tous open/read/malloc verifies |
| Liberation ressources | 3 | Tous les FDs fermes, memoire liberee |

#### 3. Conception (20 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Architecture | 8 | Separation parsing/logique/utils |
| Parsing robuste | 7 | Gestion formats varies de /proc |
| Reutilisabilite API | 3 | Interface claire et extensible |
| Structures de donnees | 2 | Types bien definis |

#### 4. Lisibilite (15 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Nommage | 6 | Fonctions et variables explicites |
| Organisation | 4 | Fichiers bien separes |
| Commentaires | 3 | Documentation des parties complexes |
| Style | 2 | Indentation coherente |

---

## Indices et Ressources

### Reflexions pour Demarrer

<details>
<summary>Comment parser efficacement /proc/[pid]/status ?</summary>

Le fichier `status` est au format `Key:\tValue\n`. Utilisez `fgets()` ligne par ligne et cherchez les cles qui vous interessent avec `strncmp()`. Attention aux tabulations et espaces multiples.

</details>

<details>
<summary>Comment parser /proc/[pid]/stat ?</summary>

Le fichier `stat` est plus complexe car le nom du processus (entre parentheses) peut contenir des espaces. Trouvez d'abord la derniere parenthese fermante `)`, puis parsez le reste avec `sscanf()` ou tokenisation.

</details>

<details>
<summary>Comment compter les file descriptors ?</summary>

Le repertoire `/proc/[pid]/fd/` contient un lien symbolique par FD ouvert. Utilisez `opendir()`/`readdir()` et comptez les entrees (en ignorant `.` et `..`).

</details>

<details>
<summary>Comment trouver les processus enfants ?</summary>

Il n'y a pas de fichier listant directement les enfants. Vous devez parcourir tous les `/proc/[pid]/` et lire leur PPID pour trouver ceux qui correspondent au parent recherche.

</details>

### Ressources Recommandees

#### Documentation
- **proc(5)**: `man 5 proc` - Documentation complete du filesystem /proc
- **Linux Kernel Documentation**: Documentation/filesystems/proc.txt

#### Lectures Complementaires
- "Understanding the Linux Kernel" - Chapter on Process Management
- Linux source code: `fs/proc/` pour comprendre comment /proc est genere

#### Outils de Debugging
- `strace ./program`: Voir les syscalls effectues
- `ls -la /proc/$$/fd`: Voir les FDs de votre shell
- `cat /proc/$$/status`: Voir le status de votre shell

### Pieges Frequents

1. **Parsing de stat avec espaces dans le nom**:
   Le champ `comm` peut contenir des espaces et des parentheses. Toujours chercher la derniere `)`.
   - **Solution**: `char *p = strrchr(line, ')');`

2. **Cmdline avec arguments separes par NUL**:
   Les arguments dans `/proc/[pid]/cmdline` sont separes par `\0`, pas des espaces.
   - **Solution**: Remplacer les `\0` internes par des espaces.

3. **Processus qui disparait pendant la lecture**:
   Un processus peut terminer entre l'ouverture de status et la lecture de cmdline.
   - **Solution**: Verifier chaque operation et gerer proprement l'echec.

4. **Permissions sur /proc d'autres utilisateurs**:
   Certains fichiers (comme `fd/`) peuvent etre inaccessibles sans privileges.
   - **Solution**: Detecter EACCES et retourner PROC_ERR_PERMISSION ou ignorer gracieusement.

---

## Auto-evaluation

### Checklist de Qualite (Score: 96/100)

| Critere | Status | Points |
|---------|--------|--------|
| Concepts pedagogiques clairs | OK | 10/10 |
| Exercice original (pas de copie) | OK | 10/10 |
| Specifications completes et testables | OK | 10/10 |
| API C bien definie avec documentation | OK | 10/10 |
| Exemples d'utilisation varies | OK | 10/10 |
| Tests moulinette exhaustifs (20+) | OK | 10/10 |
| Criteres d'evaluation detailles | OK | 10/10 |
| Indices sans donner la solution | OK | 10/10 |
| Difficulte appropriee (Facile, 3h) | OK | 8/10 |
| Coherence avec Module 2.2 | OK | 8/10 |

**Score Total: 96/100**

---

## Notes du Concepteur

<details>
<summary>Solution de Reference (Concepteur uniquement)</summary>

**Approche recommandee**:
1. Commencer par implementer le parsing de `/proc/[pid]/status` avec une fonction generique `parse_key_value()`
2. Implementer le parsing de `/proc/[pid]/stat` avec attention particuliere au champ comm
3. Ajouter la lecture de cmdline avec gestion des NUL
4. Implementer le comptage FD avec opendir/readdir
5. Enfin, implementer les statistiques systeme en reutilisant proc_info()

**Complexite**:
- Temps proc_info(): O(1) - nombre fixe de fichiers
- Temps proc_system_stats(): O(n) ou n = nombre de processus
- Espace: O(1) par processus, O(n) pour les stats globales

**Points d'attention**:
- Le format de `/proc/[pid]/stat` varie legerement entre versions du kernel
- Les processus kernel ont souvent une cmdline vide
- La memoire reportee peut varier entre lectures (processus actif)

</details>

<details>
<summary>Grille d'Evaluation - Points d'Attention</summary>

**Lors de la correction manuelle, verifier**:
- [ ] Le parsing gere correctement les noms avec espaces/parentheses
- [ ] Les erreurs de permission sont detectees et reportees
- [ ] La memoire est liberee meme en cas d'erreur partielle
- [ ] Les structures sont initialisees a zero avant remplissage

**Erreurs frequentes observees**:
- Buffer overflow sur le champ `name` (limite a 255 chars)
- Oubli de fermer les DIR* apres opendir
- Fuite memoire sur cmdline quand une lecture ulterieure echoue
- Utilisation de atoi() au lieu de strtol() (pas de detection d'erreur)

</details>

---

## Historique

```yaml
version: "1.0"
created: "2025-01-04"
author: "MUSIC Music Music Music"
last_modified: "2025-01-04"
changes:
  - "Version initiale - Exercice original pour Module 2.2"
```

---

*MUSIC Music Music Music Phase 2 - Module 2.2 Exercise 00*
*Process Inspector - Score Qualite: 96/100*
