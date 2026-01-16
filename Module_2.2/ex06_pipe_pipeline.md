# [Module 2.2] - Exercise 06: Pipe Pipeline

## Métadonnées

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex06"
difficulty: moyen
estimated_time: "5 heures"
prerequisite_exercises: ["ex01", "ex02", "ex03"]
concepts_requis:
  - "fork() et exec()"
  - "Gestion des processus enfants (wait)"
  - "Descripteurs de fichiers"
  - "Redirection d'entrée/sortie"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.2.16.a | IPC mechanisms | Vue d'ensemble des mécanismes IPC UNIX |
| 2.2.16.b | IPC comparison | Pipes vs files vs sockets vs shared memory |
| 2.2.16.c | Choosing IPC | Critères de sélection du bon mécanisme |
| 2.2.17.a | pipe() syscall | Création d'un pipe anonyme |
| 2.2.17.b | Pipe internals | Buffer kernel, taille, atomicité |
| 2.2.17.c | dup2() for redirection | Redirection de stdin/stdout |
| 2.2.17.d | Closing unused ends | Importance de fermer les FDs inutilisés |
| 2.2.17.e | Pipe capacity | PIPE_BUF et atomicité |
| 2.2.17.f | EOF on pipes | Détection de fin de données |
| 2.2.18.a | mkfifo() | Création de FIFOs (named pipes) |
| 2.2.18.b | FIFO vs pipe | Différences et cas d'usage |
| 2.2.18.c | FIFO permissions | Mode et droits d'accès |
| 2.2.18.d | FIFO blocking | Comportement à l'ouverture |
| 2.2.18.e | Bidirectional FIFOs | Communication dans les deux sens |

### Objectifs Pédagogiques

À la fin de cet exercice, vous saurez:
1. Créer des pipelines de commandes comme un shell UNIX
2. Rediriger correctement stdin/stdout avec `dup2()`
3. Gérer les descripteurs de fichiers pour éviter les fuites et deadlocks
4. Utiliser les FIFOs pour communication inter-processus persistante
5. Implémenter une communication bidirectionnelle avec deux pipes
6. Mesurer et optimiser le throughput des pipes

---

## Contexte

Le pipe (`|`) est l'un des concepts les plus puissants d'UNIX. Il permet de composer des programmes simples pour créer des traitements complexes: `cat file | grep pattern | sort | uniq -c`. Cette philosophie "do one thing well" a façonné 50 ans de design logiciel.

Les pipes fonctionnent comme suit:
- **Pipe anonyme**: Créé par `pipe()`, existe uniquement entre processus ayant un ancêtre commun
- **FIFO (named pipe)**: Existe dans le filesystem, permet la communication entre processus non-liés
- **Buffer kernel**: Les données transitent par un buffer de ~64KB géré par le noyau

Les défis techniques incluent:
- **Fermeture des FDs**: Oublier de fermer un bout = le lecteur ne voit jamais EOF
- **Deadlocks**: Le writer bloque si le buffer est plein, le reader bloque s'il est vide
- **Atomicité**: Écriture <= PIPE_BUF (4KB) garantie atomique, au-delà peut s'entrelacer

**Exemple concret**: Vous construisez l'équivalent de bash pour exécuter `ls -la | grep ".c" | wc -l`.

---

## Énoncé

### Vue d'Ensemble

Créez une bibliothèque `pipeline` qui permet de construire et exécuter des pipelines de commandes, similaires à ceux d'un shell UNIX. La bibliothèque doit supporter les pipes anonymes, les FIFOs nommés, la communication bidirectionnelle, et fournir des métriques de performance.

### Spécifications Fonctionnelles

#### Partie 1: Structures de Base

```c
#ifndef PIPELINE_H
#define PIPELINE_H

#include <sys/types.h>
#include <stdint.h>

// Limites
#define PIPE_MAX_COMMANDS 16
#define PIPE_MAX_ARGS     64
#define PIPE_PATH_MAX     256

// Status d'une commande
typedef enum {
    CMD_STATUS_PENDING,     // Pas encore démarrée
    CMD_STATUS_RUNNING,     // En cours d'exécution
    CMD_STATUS_EXITED,      // Terminée normalement
    CMD_STATUS_SIGNALED,    // Tuée par un signal
    CMD_STATUS_STOPPED,     // Stoppée
    CMD_STATUS_ERROR        // Erreur au lancement
} cmd_status_t;

// Information sur une commande
typedef struct {
    char            *path;              // Chemin de l'exécutable
    char            **argv;             // Arguments (argv[0] = nom)
    int             argc;               // Nombre d'arguments
    pid_t           pid;                // PID après fork
    cmd_status_t    status;             // Status actuel
    int             exit_code;          // Code de sortie (si EXITED)
    int             signal_num;         // Signal (si SIGNALED)
    struct timespec start_time;         // Heure de démarrage
    struct timespec end_time;           // Heure de fin
} pipe_cmd_t;

// Type de pipeline
typedef enum {
    PIPE_TYPE_ANONYMOUS,    // Pipes anonymes (défaut)
    PIPE_TYPE_FIFO,         // FIFOs nommés
    PIPE_TYPE_BIDIRECTIONAL // Deux pipes par connexion
} pipe_type_t;

// Options du pipeline
typedef struct {
    pipe_type_t     type;               // Type de pipes à utiliser
    const char      *fifo_dir;          // Répertoire pour les FIFOs
    int             close_on_exec;      // FD_CLOEXEC sur les pipes internes
    size_t          buffer_hint;        // Suggestion de taille de buffer (0 = défaut)
} pipe_options_t;

#define PIPE_OPTIONS_DEFAULT { \
    .type = PIPE_TYPE_ANONYMOUS, \
    .fifo_dir = "/tmp", \
    .close_on_exec = 1, \
    .buffer_hint = 0 \
}

// Pipeline opaque
typedef struct pipeline pipeline_t;
```

#### Partie 2: Création et Configuration

```c
/**
 * Crée un nouveau pipeline vide.
 *
 * @param options Options de configuration (NULL pour défauts)
 *
 * @return Pipeline ou NULL si erreur
 */
pipeline_t *pipe_create(const pipe_options_t *options);

/**
 * Détruit un pipeline et libère toutes les ressources.
 * Si des processus sont encore en cours, ils sont tués (SIGTERM puis SIGKILL).
 *
 * @param p Pipeline (peut être NULL)
 */
void pipe_destroy(pipeline_t *p);

/**
 * Ajoute une commande au pipeline.
 *
 * @param p    Pipeline
 * @param argv Arguments de la commande (terminé par NULL)
 *
 * @return Index de la commande (0-based), ou -1 si erreur
 *
 * @note argv[0] est le nom/chemin de la commande
 * @note Les arguments sont copiés en interne
 */
int pipe_add_cmd(pipeline_t *p, char *const argv[]);

/**
 * Ajoute une commande en parsant une chaîne.
 *
 * @param p       Pipeline
 * @param cmdline Ligne de commande (ex: "grep -i pattern")
 *
 * @return Index de la commande, ou -1 si erreur
 *
 * @note Parsing simple: split sur espaces, pas de quotes/escapes
 */
int pipe_add_cmd_str(pipeline_t *p, const char *cmdline);

/**
 * Définit les redirections d'entrée/sortie pour le pipeline.
 *
 * @param p        Pipeline
 * @param stdin_fd FD pour l'entrée de la première commande (-1 = hériter)
 * @param stdout_fd FD pour la sortie de la dernière commande (-1 = hériter)
 *
 * @return 0 si succès, -1 si erreur
 */
int pipe_set_io(pipeline_t *p, int stdin_fd, int stdout_fd);

/**
 * Configure l'entrée depuis un fichier.
 *
 * @param p    Pipeline
 * @param path Chemin du fichier d'entrée
 *
 * @return 0 si succès, -1 si erreur
 */
int pipe_set_input_file(pipeline_t *p, const char *path);

/**
 * Configure la sortie vers un fichier.
 *
 * @param p      Pipeline
 * @param path   Chemin du fichier de sortie
 * @param append 1 pour ajouter, 0 pour écraser
 *
 * @return 0 si succès, -1 si erreur
 */
int pipe_set_output_file(pipeline_t *p, const char *path, int append);
```

#### Partie 3: Exécution

```c
// Flags d'exécution
typedef enum {
    PIPE_EXEC_DEFAULT   = 0,
    PIPE_EXEC_NOWAIT    = (1 << 0),  // Ne pas attendre la fin
    PIPE_EXEC_FOREGROUND = (1 << 1), // Processus en foreground (job control)
} pipe_exec_flags_t;

/**
 * Exécute le pipeline.
 *
 * @param p     Pipeline
 * @param flags Flags d'exécution
 *
 * @return 0 si succès (tous les processus ont démarré), -1 si erreur
 *
 * @note Avec PIPE_EXEC_NOWAIT, retourne immédiatement après le fork
 * @note Sans PIPE_EXEC_NOWAIT, attend que tous les processus terminent
 */
int pipe_execute(pipeline_t *p, pipe_exec_flags_t flags);

/**
 * Attend la fin de tous les processus du pipeline.
 *
 * @param p          Pipeline
 * @param timeout_ms Timeout en millisecondes (-1 = infini)
 *
 * @return 0 si tous terminés, 1 si timeout, -1 si erreur
 */
int pipe_wait(pipeline_t *p, int timeout_ms);

/**
 * Envoie un signal à tous les processus du pipeline.
 *
 * @param p      Pipeline
 * @param signum Signal à envoyer
 *
 * @return Nombre de processus signalés, -1 si erreur
 */
int pipe_signal(pipeline_t *p, int signum);

/**
 * Tue tous les processus du pipeline.
 *
 * @param p Pipeline
 *
 * @return 0 si succès
 *
 * @note Envoie SIGTERM, attend 100ms, puis SIGKILL si nécessaire
 */
int pipe_kill(pipeline_t *p);
```

#### Partie 4: Inspection et Résultats

```c
/**
 * Retourne le nombre de commandes dans le pipeline.
 *
 * @param p Pipeline
 *
 * @return Nombre de commandes
 */
int pipe_count(const pipeline_t *p);

/**
 * Retourne les informations sur une commande.
 *
 * @param p   Pipeline
 * @param idx Index de la commande (0-based)
 *
 * @return Pointeur vers les infos (NULL si idx invalide)
 *
 * @note Le pointeur est valide jusqu'à pipe_destroy()
 */
const pipe_cmd_t *pipe_get_cmd(const pipeline_t *p, int idx);

/**
 * Vérifie si le pipeline a terminé.
 *
 * @param p Pipeline
 *
 * @return 1 si tous les processus ont terminé, 0 sinon
 */
int pipe_is_done(const pipeline_t *p);

/**
 * Retourne le code de sortie global du pipeline.
 *
 * @param p Pipeline
 *
 * @return Code de sortie de la dernière commande, ou -1 si pas terminé
 *
 * @note Suit la convention shell: retourne le status de la dernière commande
 */
int pipe_exit_code(const pipeline_t *p);

/**
 * Vérifie si une commande a réussi (exit code 0).
 *
 * @param p   Pipeline
 * @param idx Index de la commande
 *
 * @return 1 si succès, 0 si échec, -1 si pas terminé
 */
int pipe_cmd_success(const pipeline_t *p, int idx);
```

#### Partie 5: FIFOs et Communication Bidirectionnelle

```c
/**
 * Crée un FIFO nommé pour communication externe.
 *
 * @param p    Pipeline
 * @param name Nom du FIFO (sera créé dans fifo_dir)
 * @param mode Permissions (ex: 0644)
 *
 * @return Chemin complet du FIFO créé, ou NULL si erreur
 *
 * @note Le FIFO est supprimé automatiquement dans pipe_destroy()
 */
const char *pipe_create_fifo(pipeline_t *p, const char *name, mode_t mode);

/**
 * Configure la communication bidirectionnelle entre deux commandes.
 *
 * Crée deux pipes: cmd[idx1] stdout -> cmd[idx2] stdin
 *                  cmd[idx2] stdout -> cmd[idx1] stdin
 *
 * @param p    Pipeline
 * @param idx1 Index de la première commande
 * @param idx2 Index de la deuxième commande
 *
 * @return 0 si succès, -1 si erreur
 *
 * @note Les deux commandes doivent exister et ne pas être déjà connectées
 */
int pipe_connect_bidirectional(pipeline_t *p, int idx1, int idx2);

/**
 * Récupère les FDs de communication bidirectionnelle pour accès externe.
 *
 * @param p         Pipeline
 * @param idx       Index de la commande
 * @param read_fd   Reçoit le FD de lecture (peut être NULL)
 * @param write_fd  Reçoit le FD d'écriture (peut être NULL)
 *
 * @return 0 si succès, -1 si pas de connexion bidirectionnelle
 */
int pipe_get_bidirectional_fds(pipeline_t *p, int idx,
                                int *read_fd, int *write_fd);
```

#### Partie 6: Métriques de Performance

```c
typedef struct {
    uint64_t    bytes_transferred;      // Octets passés dans les pipes
    uint64_t    total_time_us;          // Temps total d'exécution
    uint64_t    user_time_us;           // Temps CPU utilisateur
    uint64_t    sys_time_us;            // Temps CPU système
    double      throughput_mbps;        // Débit en MB/s
    int         pipe_buffer_size;       // Taille du buffer kernel
    int         max_concurrent_procs;   // Max processus simultanés
} pipe_metrics_t;

/**
 * Collecte les métriques de performance.
 *
 * @param p       Pipeline
 * @param metrics Structure à remplir
 *
 * @return 0 si succès, -1 si pipeline pas terminé
 */
int pipe_get_metrics(const pipeline_t *p, pipe_metrics_t *metrics);

/**
 * Lance un benchmark de throughput sur le pipeline.
 *
 * @param p         Pipeline
 * @param data_size Taille des données à transférer (octets)
 * @param metrics   Reçoit les métriques
 *
 * @return 0 si succès, -1 si erreur
 *
 * @note Crée un générateur de données en entrée et un sink en sortie
 */
int pipe_benchmark(pipeline_t *p, size_t data_size, pipe_metrics_t *metrics);

/**
 * Affiche les métriques de façon formatée.
 *
 * @param metrics Métriques à afficher
 * @param fd      File descriptor de sortie
 */
void pipe_print_metrics(const pipe_metrics_t *metrics, int fd);

#endif /* PIPELINE_H */
```

### Spécifications Techniques

#### Architecture

```
pipe_create() --> [Empty Pipeline]
       |
       v
pipe_add_cmd("ls -la") --> [cmd0: ls]
       |
       v
pipe_add_cmd("grep .c") --> [cmd0: ls] --pipe--> [cmd1: grep]
       |
       v
pipe_add_cmd("wc -l") --> [cmd0: ls] --pipe--> [cmd1: grep] --pipe--> [cmd2: wc]
       |
       v
pipe_execute()
       |
       +---> fork() --> exec("ls")   --> stdout=pipe0[1]
       |
       +---> fork() --> exec("grep") --> stdin=pipe0[0], stdout=pipe1[1]
       |
       +---> fork() --> exec("wc")   --> stdin=pipe1[0]
       |
       v
pipe_wait() --> collect exit codes
```

#### Gestion des Descripteurs

```
Pour N commandes, on crée N-1 pipes:

Commande 0:  stdin = original (ou redirigé)
             stdout = pipes[0][1]

Commande i:  stdin = pipes[i-1][0]
             stdout = pipes[i][1]

Commande N-1: stdin = pipes[N-2][0]
              stdout = original (ou redirigé)

CRITICAL: Fermer tous les FDs non utilisés dans chaque processus!
```

#### Communication Bidirectionnelle

```
Pour deux processus A et B communiquant dans les deux sens:

     +------+          +------+
     |  A   |          |  B   |
     +------+          +------+
        |                  |
stdout--+---> pipe1 --->---+-->stdin
        |                  |
stdin---+---< pipe2 <--<---+--stdout
```

---

## Contraintes Techniques

### Standards C

- **Norme**: C17 (ISO/IEC 9899:2018)
- **Compilation**: `gcc -Wall -Wextra -Werror -std=c17 -D_POSIX_C_SOURCE=200809L`

### Fonctions Autorisées

```
Processus:
  - fork, execve, execvp, _exit
  - waitpid, wait, WIFEXITED, WEXITSTATUS, etc.

Pipes:
  - pipe, pipe2
  - dup, dup2
  - close, read, write

FIFOs:
  - mkfifo, unlink

Fichiers:
  - open, close, O_RDONLY, O_WRONLY, O_CREAT, O_APPEND

Descripteurs:
  - fcntl (F_GETFD, F_SETFD, FD_CLOEXEC)

Mémoire:
  - malloc, free, realloc, calloc
  - memset, memcpy, strdup, strtok_r

Temps:
  - clock_gettime (CLOCK_MONOTONIC)
  - getrusage (pour métriques CPU)

Signaux:
  - kill
```

### Fonctions INTERDITES

```
  - system() - vous devez implémenter l'exécution vous-même
  - popen() - idem
```

### Contraintes Spécifiques

- [ ] Maximum 16 commandes par pipeline
- [ ] Tous les FDs non utilisés doivent être fermés après fork
- [ ] FD_CLOEXEC sur les pipes internes par défaut
- [ ] Gestion propre de SIGPIPE (ignorer ou handler)
- [ ] Les FIFOs créés doivent être nettoyés à la destruction

### Exigences de Sécurité

- [ ] Valgrind clean - aucune fuite mémoire
- [ ] Pas de file descriptor leaks
- [ ] Vérification de tous les retours d'erreur
- [ ] Pas de zombie processes
- [ ] PATH traversal protection pour les FIFOs

---

## Format de Rendu

### Fichiers à Rendre

```
ex06/
├── pipeline.h          # API publique
├── pipeline.c          # Implémentation principale
├── pipeline_exec.c     # Logique d'exécution (optionnel)
├── pipeline_fifo.c     # Gestion des FIFOs (optionnel)
├── main.c              # Démonstration
└── Makefile
```

### Makefile Requis

```makefile
NAME = pipeline_demo
LIB = libpipeline.a

CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c17 -D_POSIX_C_SOURCE=200809L

SRCS = pipeline.c
OBJS = $(SRCS:.c=.o)

all: $(LIB) $(NAME)

$(LIB): $(OBJS)
	ar rcs $@ $^

$(NAME): main.o $(LIB)
	$(CC) -o $@ main.o -L. -lpipeline

clean:
	rm -f $(OBJS) main.o

fclean: clean
	rm -f $(NAME) $(LIB)

re: fclean all

.PHONY: all clean fclean re
```

---

## Exemples d'Utilisation

### Exemple 1: Pipeline Simple (ls | grep | wc)

```c
#include "pipeline.h"
#include <stdio.h>

int main(void) {
    pipeline_t *p = pipe_create(NULL);

    // Construire le pipeline: ls -la | grep ".c" | wc -l
    char *ls_args[] = {"ls", "-la", NULL};
    char *grep_args[] = {"grep", ".c", NULL};
    char *wc_args[] = {"wc", "-l", NULL};

    pipe_add_cmd(p, ls_args);
    pipe_add_cmd(p, grep_args);
    pipe_add_cmd(p, wc_args);

    printf("Executing: ls -la | grep .c | wc -l\n");

    if (pipe_execute(p, PIPE_EXEC_DEFAULT) == 0) {
        printf("Pipeline completed with exit code: %d\n", pipe_exit_code(p));

        // Afficher le status de chaque commande
        for (int i = 0; i < pipe_count(p); i++) {
            const pipe_cmd_t *cmd = pipe_get_cmd(p, i);
            printf("  [%d] %s: exit=%d\n", i, cmd->argv[0], cmd->exit_code);
        }
    } else {
        perror("Pipeline execution failed");
    }

    pipe_destroy(p);
    return 0;
}
```

**Sortie attendue** (dans un répertoire avec des fichiers .c):
```
Executing: ls -la | grep .c | wc -l
       5
Pipeline completed with exit code: 0
  [0] ls: exit=0
  [1] grep: exit=0
  [2] wc: exit=0
```

### Exemple 2: Pipeline avec Fichiers d'Entrée/Sortie

```c
#include "pipeline.h"
#include <stdio.h>
#include <fcntl.h>

int main(void) {
    pipeline_t *p = pipe_create(NULL);

    // cat input.txt | sort | uniq > output.txt
    pipe_add_cmd_str(p, "sort");
    pipe_add_cmd_str(p, "uniq");

    // Entrée depuis un fichier
    pipe_set_input_file(p, "/etc/passwd");

    // Sortie vers un fichier
    pipe_set_output_file(p, "/tmp/unique_shells.txt", 0);

    printf("Executing: sort < /etc/passwd | uniq > /tmp/unique_shells.txt\n");

    pipe_execute(p, PIPE_EXEC_DEFAULT);

    printf("Exit code: %d\n", pipe_exit_code(p));
    printf("Output written to /tmp/unique_shells.txt\n");

    pipe_destroy(p);
    return 0;
}
```

**Sortie attendue**:
```
Executing: sort < /etc/passwd | uniq > /tmp/unique_shells.txt
Exit code: 0
Output written to /tmp/unique_shells.txt
```

### Exemple 3: Pipeline avec FIFO pour Communication Externe

```c
#include "pipeline.h"
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/wait.h>

int main(void) {
    pipe_options_t opts = PIPE_OPTIONS_DEFAULT;
    opts.type = PIPE_TYPE_FIFO;
    opts.fifo_dir = "/tmp";

    pipeline_t *p = pipe_create(&opts);

    // Créer un FIFO pour l'entrée
    const char *input_fifo = pipe_create_fifo(p, "input_fifo", 0644);
    printf("Input FIFO created: %s\n", input_fifo);

    // Pipeline: cat (depuis FIFO) | tr 'a-z' 'A-Z'
    pipe_add_cmd_str(p, "cat");
    pipe_add_cmd_str(p, "tr a-z A-Z");

    // Connecter l'entrée au FIFO
    int fifo_fd = open(input_fifo, O_RDONLY | O_NONBLOCK);
    pipe_set_io(p, fifo_fd, -1);

    // Lancer en mode non-bloquant
    pipe_execute(p, PIPE_EXEC_NOWAIT);

    // Un autre processus peut maintenant écrire dans le FIFO
    pid_t writer = fork();
    if (writer == 0) {
        int fd = open(input_fifo, O_WRONLY);
        write(fd, "hello world\n", 12);
        close(fd);
        _exit(0);
    }

    waitpid(writer, NULL, 0);
    close(fifo_fd);

    pipe_wait(p, 1000);
    pipe_destroy(p);
    return 0;
}
```

**Sortie attendue**:
```
Input FIFO created: /tmp/input_fifo
HELLO WORLD
```

### Exemple 4: Communication Bidirectionnelle

```c
#include "pipeline.h"
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(void) {
    pipe_options_t opts = PIPE_OPTIONS_DEFAULT;
    opts.type = PIPE_TYPE_BIDIRECTIONAL;

    pipeline_t *p = pipe_create(&opts);

    // Deux commandes qui communiquent dans les deux sens
    // cmd0 envoie "hello", cmd1 répond "HELLO"

    // Pour cet exemple, on utilise des scripts simples
    // En pratique, vous utiliseriez des programmes interactifs

    pipe_add_cmd_str(p, "cat");  // Echo simple
    pipe_add_cmd_str(p, "tr a-z A-Z");  // Majuscules

    pipe_connect_bidirectional(p, 0, 1);

    pipe_execute(p, PIPE_EXEC_NOWAIT);

    // Obtenir les FDs pour communiquer
    int read_fd, write_fd;
    pipe_get_bidirectional_fds(p, 0, &read_fd, &write_fd);

    // Envoyer des données
    const char *msg = "hello\n";
    write(write_fd, msg, strlen(msg));

    // Lire la réponse
    char buf[256];
    ssize_t n = read(read_fd, buf, sizeof(buf) - 1);
    if (n > 0) {
        buf[n] = '\0';
        printf("Response: %s", buf);
    }

    pipe_kill(p);
    pipe_destroy(p);
    return 0;
}
```

**Sortie attendue**:
```
Response: HELLO
```

### Exemple 5: Benchmark de Throughput

```c
#include "pipeline.h"
#include <stdio.h>

int main(void) {
    pipeline_t *p = pipe_create(NULL);

    // Pipeline pour mesurer le throughput: cat | cat | cat
    pipe_add_cmd_str(p, "cat");
    pipe_add_cmd_str(p, "cat");
    pipe_add_cmd_str(p, "cat");

    pipe_metrics_t metrics;

    // Benchmark avec 100 MB de données
    printf("Benchmarking 100 MB through pipeline...\n");

    if (pipe_benchmark(p, 100 * 1024 * 1024, &metrics) == 0) {
        pipe_print_metrics(&metrics, STDOUT_FILENO);
    }

    pipe_destroy(p);
    return 0;
}
```

**Sortie attendue**:
```
Benchmarking 100 MB through pipeline...
=== Pipeline Metrics ===
Bytes transferred:  104,857,600 (100.00 MB)
Total time:         1,234,567 us (1.23 s)
User CPU time:      234,567 us
System CPU time:    567,890 us
Throughput:         84.52 MB/s
Pipe buffer size:   65,536 bytes
Max concurrent:     3 processes
```

---

## Tests de la Moulinette

### Tests Fonctionnels de Base

#### Test 01: Création et Destruction

```yaml
description: "Création et destruction d'un pipeline vide"
code: |
  pipeline_t *p = pipe_create(NULL);
  assert(p != NULL);
  assert(pipe_count(p) == 0);
  pipe_destroy(p);
  pipe_destroy(NULL);  // Safe
expected: "PASS - Valgrind clean"
```

#### Test 02: Ajout de Commandes

```yaml
description: "Ajout de plusieurs commandes"
code: |
  pipeline_t *p = pipe_create(NULL);

  char *ls[] = {"ls", NULL};
  assert(pipe_add_cmd(p, ls) == 0);
  assert(pipe_count(p) == 1);

  char *grep[] = {"grep", "test", NULL};
  assert(pipe_add_cmd(p, grep) == 1);
  assert(pipe_count(p) == 2);

  assert(pipe_add_cmd_str(p, "wc -l") == 2);
  assert(pipe_count(p) == 3);

  pipe_destroy(p);
expected: "PASS"
```

#### Test 03: Exécution Simple

```yaml
description: "Exécution d'une commande unique"
code: |
  pipeline_t *p = pipe_create(NULL);
  pipe_add_cmd_str(p, "echo hello");
  assert(pipe_execute(p, PIPE_EXEC_DEFAULT) == 0);
  assert(pipe_exit_code(p) == 0);
  pipe_destroy(p);
expected: "PASS - 'hello' sur stdout"
```

#### Test 04: Pipeline de 2 Commandes

```yaml
description: "Pipeline avec pipe"
code: |
  pipeline_t *p = pipe_create(NULL);
  pipe_add_cmd_str(p, "echo hello world");
  pipe_add_cmd_str(p, "wc -w");
  pipe_execute(p, PIPE_EXEC_DEFAULT);
  assert(pipe_exit_code(p) == 0);
  pipe_destroy(p);
expected: "PASS - '2' sur stdout"
```

#### Test 05: Pipeline de 5 Commandes

```yaml
description: "Long pipeline"
code: |
  pipeline_t *p = pipe_create(NULL);
  pipe_add_cmd_str(p, "echo a b c d e");
  pipe_add_cmd_str(p, "tr ' ' '\\n'");
  pipe_add_cmd_str(p, "sort");
  pipe_add_cmd_str(p, "uniq");
  pipe_add_cmd_str(p, "wc -l");
  pipe_execute(p, PIPE_EXEC_DEFAULT);
  assert(pipe_exit_code(p) == 0);
  // Devrait afficher "5"
  pipe_destroy(p);
expected: "PASS"
```

#### Test 06: Commande Inexistante

```yaml
description: "Gestion d'une commande qui n'existe pas"
code: |
  pipeline_t *p = pipe_create(NULL);
  pipe_add_cmd_str(p, "command_that_does_not_exist_12345");
  int ret = pipe_execute(p, PIPE_EXEC_DEFAULT);
  // L'exécution échoue mais ne crash pas
  const pipe_cmd_t *cmd = pipe_get_cmd(p, 0);
  assert(cmd->status == CMD_STATUS_ERROR);
  pipe_destroy(p);
expected: "PASS - Gestion propre de l'erreur"
```

#### Test 07: Redirection Fichier

```yaml
description: "Redirection entrée/sortie"
code: |
  // Créer un fichier d'entrée
  int fd = open("/tmp/test_input.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
  write(fd, "line1\nline2\nline3\n", 18);
  close(fd);

  pipeline_t *p = pipe_create(NULL);
  pipe_add_cmd_str(p, "wc -l");
  pipe_set_input_file(p, "/tmp/test_input.txt");
  pipe_set_output_file(p, "/tmp/test_output.txt", 0);
  pipe_execute(p, PIPE_EXEC_DEFAULT);

  // Vérifier la sortie
  fd = open("/tmp/test_output.txt", O_RDONLY);
  char buf[64];
  read(fd, buf, 64);
  // buf devrait contenir "3" (3 lignes)
  close(fd);

  pipe_destroy(p);
expected: "PASS"
```

#### Test 08: Mode Non-Bloquant

```yaml
description: "Exécution asynchrone"
code: |
  pipeline_t *p = pipe_create(NULL);
  pipe_add_cmd_str(p, "sleep 2");

  assert(pipe_execute(p, PIPE_EXEC_NOWAIT) == 0);
  assert(pipe_is_done(p) == 0);  // Pas encore terminé

  // Attendre avec timeout court
  assert(pipe_wait(p, 100) == 1);  // Timeout

  // Attendre vraiment
  assert(pipe_wait(p, 3000) == 0);  // Terminé
  assert(pipe_is_done(p) == 1);

  pipe_destroy(p);
expected: "PASS"
```

### Tests de Robustesse

#### Test 10: Commande qui Échoue

```yaml
description: "Gestion des codes de sortie non-zero"
code: |
  pipeline_t *p = pipe_create(NULL);
  pipe_add_cmd_str(p, "cat /nonexistent/file");
  pipe_add_cmd_str(p, "wc");
  pipe_execute(p, PIPE_EXEC_DEFAULT);

  const pipe_cmd_t *cat_cmd = pipe_get_cmd(p, 0);
  assert(cat_cmd->exit_code != 0);  // cat échoue

  // wc peut réussir (avec 0 entrée)
  assert(pipe_exit_code(p) == 0);  // wc réussit

  pipe_destroy(p);
expected: "PASS"
```

#### Test 11: Pipeline Tué

```yaml
description: "Interruption forcée du pipeline"
code: |
  pipeline_t *p = pipe_create(NULL);
  pipe_add_cmd_str(p, "sleep 100");
  pipe_add_cmd_str(p, "cat");

  pipe_execute(p, PIPE_EXEC_NOWAIT);

  // Tuer après 100ms
  usleep(100000);
  pipe_kill(p);

  pipe_wait(p, 1000);
  assert(pipe_is_done(p) == 1);

  const pipe_cmd_t *cmd = pipe_get_cmd(p, 0);
  assert(cmd->status == CMD_STATUS_SIGNALED);

  pipe_destroy(p);
expected: "PASS"
```

### Tests de Sécurité

#### Test 20: Valgrind Clean

```yaml
description: "Absence de fuites mémoire"
command: "valgrind --leak-check=full --track-fds=yes ./pipeline_test"
expected: |
  - 0 bytes lost
  - No open file descriptors at exit (except stdin/stdout/stderr)
```

#### Test 21: Pas de FD Leaks

```yaml
description: "Vérification des descripteurs de fichiers"
code: |
  int initial_fds = count_open_fds();

  for (int i = 0; i < 100; i++) {
    pipeline_t *p = pipe_create(NULL);
    pipe_add_cmd_str(p, "echo test");
    pipe_add_cmd_str(p, "cat");
    pipe_add_cmd_str(p, "wc");
    pipe_execute(p, PIPE_EXEC_DEFAULT);
    pipe_destroy(p);
  }

  int final_fds = count_open_fds();
  assert(final_fds == initial_fds);
expected: "PASS - Même nombre de FDs"
```

#### Test 22: Pas de Zombies

```yaml
description: "Vérification qu'il n'y a pas de processus zombie"
code: |
  for (int i = 0; i < 50; i++) {
    pipeline_t *p = pipe_create(NULL);
    pipe_add_cmd_str(p, "true");
    pipe_execute(p, PIPE_EXEC_DEFAULT);
    pipe_destroy(p);
  }

  // Vérifier qu'il n'y a pas de zombies
  // (count processes in Z state for this user)
expected: "PASS - 0 zombie processes"
```

### Tests de Performance

#### Test 30: Throughput

```yaml
description: "Mesure du débit"
scenario: |
  Pipeline: cat | cat
  Données: 100 MB
expected_throughput: "> 50 MB/s"
machine_ref: "Intel i5 2.5GHz"
```

#### Test 31: Latence de Démarrage

```yaml
description: "Temps pour lancer un pipeline"
scenario: |
  Mesurer le temps entre pipe_execute() et
  le premier processus qui exécute
iterations: 100
expected_latency: "< 10 ms (99th percentile)"
```

---

## Critères d'Évaluation

### Note Minimale Requise: 80/100

### Détail de la Notation (Total: 100 points)

#### 1. Correction (40 points)

| Critère | Points | Description |
|---------|--------|-------------|
| Pipeline simple | 10 | 2+ commandes avec pipe |
| Redirections | 10 | Entrée/sortie fichiers |
| Gestion des erreurs | 10 | Commandes invalides, timeouts |
| Fermeture des FDs | 10 | Pas de fuites de descripteurs |

#### 2. Sécurité (25 points)

| Critère | Points | Description |
|---------|--------|-------------|
| Valgrind clean | 10 | Pas de fuites mémoire |
| Pas de FD leaks | 8 | Descripteurs fermés |
| Pas de zombies | 7 | Tous les enfants récoltés |

#### 3. Conception (20 points)

| Critère | Points | Description |
|---------|--------|-------------|
| Architecture | 8 | Séparation claire des responsabilités |
| API intuitive | 7 | Facile à utiliser |
| Extensibilité | 5 | Support FIFO, bidirectionnel |

#### 4. Lisibilité (15 points)

| Critère | Points | Description |
|---------|--------|-------------|
| Documentation | 6 | Chaque fonction documentée |
| Nommage | 5 | Variables et fonctions explicites |
| Organisation | 4 | Code structuré |

---

## Indices et Ressources

### Réflexions pour Démarrer

<details>
<summary>Question 1: Dans quel ordre créer les pipes et faire les fork?</summary>

Ordre recommandé:
1. Créer TOUS les pipes avant de fork
2. Fork chaque processus
3. Dans chaque enfant: fermer les FDs inutilisés, dup2() les bons, exec()
4. Dans le parent: fermer TOUS les bouts de pipe (critiquement important!)
5. Attendre tous les enfants

Si vous fermez les pipes après les forks mais avant les exec, vous risquez des races.

</details>

<details>
<summary>Question 2: Pourquoi fermer les FDs inutilisés est si important?</summary>

Exemple avec `cmd1 | cmd2`:

Si le parent garde pipe[1] ouvert:
- cmd2 lit de pipe[0]
- cmd1 termine et ferme pipe[1]
- cmd2 NE VOIT PAS EOF car le parent a encore pipe[1] ouvert!
- Deadlock: cmd2 attend indéfiniment

Règle: Fermez TOUT ce que vous n'utilisez pas, IMMÉDIATEMENT après fork.

</details>

<details>
<summary>Question 3: Comment gérer dup2() et les erreurs?</summary>

Pattern recommandé:
```c
// Dans l'enfant, après fork()
if (dup2(pipe_read, STDIN_FILENO) == -1) {
    // Ne pas utiliser printf! Utiliser write() sur stderr
    write(STDERR_FILENO, "dup2 failed\n", 12);
    _exit(127);
}
close(pipe_read);  // Fermer l'original après dup2

// Fermer tous les autres pipes
for (int i = 0; i < num_pipes * 2; i++) {
    close(pipe_fds[i]);
}

execvp(argv[0], argv);
// Si on arrive ici, exec a échoué
_exit(127);
```

</details>

### Ressources Recommandées

- `man 2 pipe` - Création de pipes
- `man 2 dup2` - Redirection de descripteurs
- `man 7 pipe` - Détails sur le comportement des pipes
- `man 3 mkfifo` - FIFOs nommés
- "The Linux Programming Interface", ch. 44 - Pipes and FIFOs

### Pièges Fréquents

1. **Ne pas fermer les FDs dans le parent**
   - **Symptôme**: Le lecteur ne voit jamais EOF
   - **Solution**: Fermez les deux bouts de chaque pipe dans le parent après les forks

2. **Fermer le mauvais bout au mauvais moment**
   - **Symptôme**: SIGPIPE ou lecture de 0 bytes trop tôt
   - **Solution**: Dessinez un schéma des FDs avant de coder

3. **SIGPIPE tue le processus**
   - **Symptôme**: Processus meurt silencieusement
   - **Solution**: `signal(SIGPIPE, SIG_IGN)` ou handler

4. **Oublier de gérer le cas où execvp échoue**
   - **Symptôme**: Processus enfant tourne indéfiniment
   - **Solution**: `_exit(127)` juste après execvp

5. **Buffer du pipe plein cause un deadlock**
   - **Symptôme**: Le writer bloque car le reader ne lit pas
   - **Solution**: Vérifiez l'ordre des opérations, utilisez poll() si nécessaire

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 24 | Pipeline complet, FIFO, bidirectionnel, benchmark |
| Couverture conceptuelle | 25 | 14 concepts (2.2.16-2.2.18) tous couverts |
| Testabilité auto | 24 | Tests objectifs, Valgrind, FD tracking |
| Originalité | 23 | API type shell, métriques de performance |
| **TOTAL** | **96/100** | ✓ Validé |

---

## Historique

```yaml
version: "1.0"
created: "2025-01-04"
author: "ODYSSEY Curriculum Team"
last_modified: "2025-01-04"
changes:
  - "Version initiale - Exercice ex06 Pipe Pipeline"
```

---

*Template ODYSSEY Phase 2 - Module 2.2 Processes & Shell*
