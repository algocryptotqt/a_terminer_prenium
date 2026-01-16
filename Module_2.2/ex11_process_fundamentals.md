# [Module 2.2] - Exercise 11: Process Fundamentals

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex11"
title: "Process Fundamentals - Complete Process Model"
difficulty: moyen
estimated_time: "5 heures"
prerequisite_exercises: ["ex00"]
concepts_requis: ["C basics", "file I/O", "structs"]
score_qualite: 97
```

---

## Concepts Couverts

### 2.2.1: Process Concepts (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.1.a | Process definition: Program in execution | Structure ProcessInfo |
| 2.2.1.b | Process vs program: Dynamic vs static | Demonstration fork/exec |
| 2.2.1.c | Process image: Code, data, heap, stack | Analyse memory maps |
| 2.2.1.d | Process Control Block: Kernel data structure | Lecture /proc/[pid]/stat |
| 2.2.1.e | PID: Process identifier | getpid() et parsing |
| 2.2.1.f | PPID: Parent process ID | getppid() et relations |
| 2.2.1.g | UID, GID: User and group | getuid(), getgid() |
| 2.2.1.h | Process table: All PCBs | Enumeration /proc |
| 2.2.1.i | /proc filesystem: Process information | Navigation /proc |
| 2.2.1.j | /proc/[pid]/: Per-process info | Fichiers par processus |
| 2.2.1.k | /proc/[pid]/maps: Memory mappings | Parsing memory maps |
| 2.2.1.l | /proc/[pid]/fd/: File descriptors | Enumeration des fd |

### 2.2.2: Process States (11 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.2.a | New: Being created | Detection post-fork |
| 2.2.2.b | Ready: Waiting for CPU | Etat 'R' dans stat |
| 2.2.2.c | Running: Executing | Processus en cours |
| 2.2.2.d | Waiting/Blocked: Waiting for event | Etat 'S' ou 'D' |
| 2.2.2.e | Terminated: Finished | Detection terminaison |
| 2.2.2.f | State transitions: Diagram | Demonstration transitions |
| 2.2.2.g | Ready queue: Processes waiting for CPU | Analyse systeme |
| 2.2.2.h | Wait queues: Per event | Processus bloques |
| 2.2.2.i | Linux states: R, S, D, T, Z, X | Parsing /proc/stat |
| 2.2.2.j | ps command: View process states | Reimplementation ps |
| 2.2.2.k | top/htop: Live view | Affichage dynamique |

### 2.2.3: Process Creation - fork() (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.3.a | fork() syscall: Create child process | Appel fork() |
| 2.2.3.b | Return value: 0 in child, PID in parent | Gestion retour |
| 2.2.3.c | Child inheritance: Copy of parent | Verification heritage |
| 2.2.3.d | Copy-on-write: Optimization | Demonstration COW |
| 2.2.3.e | COW implementation: Shared pages until write | Test pages partagees |
| 2.2.3.f | fork() failure: ENOMEM, EAGAIN | Gestion erreurs |
| 2.2.3.g | vfork(): Shares address space | Utilisation vfork |
| 2.2.3.h | clone(): Fine-grained control | Appel clone() |
| 2.2.3.i | clone flags: CLONE_VM, CLONE_FILES, etc. | Flags de clone |
| 2.2.3.j | fork() pattern: Check return, branch | Pattern standard |
| 2.2.3.k | Multi-fork: Process tree | Arbre de processus |
| 2.2.3.l | Fork bomb prevention | Limites RLIMIT |

### 2.2.4: Program Execution - exec() (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.4.a | exec family: Replace process image | Famille exec |
| 2.2.4.b | execl(): List of arguments | Utilisation execl |
| 2.2.4.c | execv(): Array of arguments | Utilisation execv |
| 2.2.4.d | execle(): With environment | Utilisation execle |
| 2.2.4.e | execve(): Full control (syscall) | Appel systeme execve |
| 2.2.4.f | execvp(): Search PATH | Utilisation execvp |
| 2.2.4.g | execlp(): Search PATH, list args | Utilisation execlp |
| 2.2.4.h | What's preserved: PID, open files | Elements preserves |
| 2.2.4.i | What's replaced: Code, data, stack | Elements remplaces |
| 2.2.4.j | exec failure: Returns -1 | Gestion echec |
| 2.2.4.k | Shebang: #! interpreter | Scripts executables |
| 2.2.4.l | fexecve(): Execute from fd | Execution depuis fd |

### 2.2.5: Process Termination (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.5.a | exit(): Library function | Appel exit() |
| 2.2.5.b | _exit(): Direct syscall | Appel _exit() |
| 2.2.5.c | atexit(): Exit handlers | Handlers de sortie |
| 2.2.5.d | Exit status: 0-255 | Codes de retour |
| 2.2.5.e | Normal vs abnormal: exit vs signal | Types de terminaison |
| 2.2.5.f | Signal termination: Killed by signal | Terminaison par signal |
| 2.2.5.g | WIFEXITED, WIFSIGNALED: Macros | Macros d'analyse |
| 2.2.5.h | WEXITSTATUS, WTERMSIG: Extract values | Extraction valeurs |
| 2.2.5.i | Core dump: Memory snapshot | Dump memoire |
| 2.2.5.j | Return from main(): Implicit exit | Retour de main |
| 2.2.5.k | abort(): Abnormal termination | Appel abort() |
| 2.2.5.l | Exit conventions: 0=success, 1=error | Conventions exit |

### 2.2.6: Waiting for Children (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.6.a | wait(): Wait for any child | Appel wait() |
| 2.2.6.b | waitpid(): Wait for specific child | Appel waitpid() |
| 2.2.6.c | WNOHANG: Non-blocking wait | Option non bloquante |
| 2.2.6.d | WUNTRACED: Report stopped | Detection arret |
| 2.2.6.e | WCONTINUED: Report continued | Detection reprise |
| 2.2.6.f | Status inspection: Macros | Inspection status |
| 2.2.6.g | waitid(): Most flexible | Appel waitid() |
| 2.2.6.h | siginfo_t: Detailed info | Information detaillee |
| 2.2.6.i | Multiple children: Wait loop | Boucle d'attente |
| 2.2.6.j | wait3(), wait4(): With resource usage | Usage ressources |

### 2.2.7: Zombie and Orphan Processes (11 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.7.a | Zombie: Terminated, not reaped | Creation zombie |
| 2.2.7.b | Zombie state: 'Z' in ps | Detection etat Z |
| 2.2.7.c | Resource leak: PID slot held | Impact ressources |
| 2.2.7.d | Orphan: Parent died first | Creation orphelin |
| 2.2.7.e | Reparenting: init adopts orphans | Adoption par init |
| 2.2.7.f | init/systemd: PID 1 reaper | Role de PID 1 |
| 2.2.7.g | Double fork: Daemon pattern | Pattern double fork |
| 2.2.7.h | SIGCHLD: Child status change | Signal SIGCHLD |
| 2.2.7.i | SIGCHLD handler: Reap in handler | Handler de reaping |
| 2.2.7.j | Ignoring SIGCHLD: Auto-reap | Auto-reaping |
| 2.2.7.k | SA_NOCLDWAIT: Auto-reap children | Flag SA_NOCLDWAIT |

---

## Contexte

La comprehension profonde du modele de processus Unix est fondamentale pour tout developpeur systeme. Un processus est bien plus qu'un simple "programme en execution" - c'est une abstraction complexe du systeme d'exploitation qui encapsule l'etat d'execution, les ressources allouees, le contexte de securite, et les relations avec d'autres processus.

Cet exercice vous guidera a travers l'exploration systematique de chaque aspect du modele de processus, depuis sa creation avec fork() jusqu'a sa terminaison et le nettoyage par le processus parent.

---

## Enonce

### Vue d'Ensemble

Implementez une bibliotheque complete d'exploration et de manipulation des processus qui demontre tous les concepts fondamentaux du modele de processus Unix.

### Partie 1: Process Information Library

```c
#include <sys/types.h>
#include <unistd.h>

// Structure complete d'information processus
typedef struct {
    pid_t pid;
    pid_t ppid;
    pid_t pgid;
    pid_t sid;
    uid_t uid;
    uid_t euid;
    gid_t gid;
    gid_t egid;
    char state;                  // R, S, D, Z, T, X
    char comm[256];              // Nom du processus
    char cmdline[4096];          // Ligne de commande
    unsigned long vsize;         // Taille memoire virtuelle
    long rss;                    // Resident Set Size
    unsigned long starttime;     // Temps de demarrage
    int num_threads;
    int num_fds;                 // Nombre de file descriptors
} process_info_t;

// Structure pour les memory mappings
typedef struct {
    unsigned long start;
    unsigned long end;
    char perms[5];              // rwxp
    unsigned long offset;
    char pathname[256];
} memory_map_t;

typedef struct {
    memory_map_t *maps;
    size_t count;
} memory_maps_t;

// API
int proc_get_info(pid_t pid, process_info_t *info);
int proc_get_self_info(process_info_t *info);
memory_maps_t *proc_get_maps(pid_t pid);
void proc_free_maps(memory_maps_t *maps);

// Enumeration de tous les processus
typedef struct {
    process_info_t *processes;
    size_t count;
} process_list_t;

process_list_t *proc_list_all(void);
void proc_free_list(process_list_t *list);

// Filtrage
process_list_t *proc_filter_by_state(char state);
process_list_t *proc_filter_by_user(uid_t uid);
process_list_t *proc_get_children(pid_t parent_pid);
```

### Partie 2: Process Creation Demonstrator

```c
// Demonstration de fork()
typedef struct {
    pid_t parent_pid;
    pid_t child_pid;
    int child_sees_zero;        // fork() retourne 0 dans l'enfant
    int parent_sees_pid;        // fork() retourne PID dans le parent
    int memory_shared_before_write;
    int memory_diverged_after_write;
} fork_demo_result_t;

int demo_fork_basic(fork_demo_result_t *result);
int demo_fork_cow(void);         // Copy-on-Write demonstration
int demo_fork_tree(int depth);   // Arbre de processus
int demo_vfork_usage(void);      // vfork() vs fork()

// Demonstration de clone()
int demo_clone_thread(void);     // Thread-like avec CLONE_VM
int demo_clone_namespace(void);  // Nouveau namespace
```

### Partie 3: Exec Family Demonstrator

```c
// Demonstration de chaque variante exec
typedef struct {
    const char *variant;        // "execl", "execv", etc.
    int uses_path_search;
    int takes_environment;
    int takes_list_args;
    int takes_array_args;
} exec_info_t;

// Demonstrations
int demo_execl(const char *program, ...);
int demo_execv(const char *program, char *const argv[]);
int demo_execle(const char *program, const char *const envp[], ...);
int demo_execve(const char *program, char *const argv[], char *const envp[]);
int demo_execvp(const char *program, char *const argv[]);
int demo_execlp(const char *program, ...);
int demo_fexecve(int fd, char *const argv[], char *const envp[]);

// Test de ce qui est preserve/remplace apres exec
typedef struct {
    pid_t pid_before;
    pid_t pid_after;           // Doit etre identique
    int fd_preserved;          // Fichiers ouverts preserves
    int signal_handlers_reset; // Handlers remis a SIG_DFL
} exec_preservation_test_t;

int test_exec_preservation(exec_preservation_test_t *result);
```

### Partie 4: Process Termination and Waiting

```c
// Demonstration des terminaisons
typedef struct {
    int exit_status;
    int was_signaled;
    int signal_number;
    int core_dumped;
} termination_info_t;

int demo_normal_exit(int status);
int demo_signal_termination(int signum);
int demo_atexit_handlers(void);
int demo_abort(void);

// Demonstration des waits
typedef struct {
    pid_t waited_pid;
    int status;
    int exited_normally;
    int exit_code;
    int was_signaled;
    int term_signal;
    int was_stopped;
    int stop_signal;
    int was_continued;
} wait_result_t;

int demo_wait_basic(wait_result_t *result);
int demo_waitpid_specific(pid_t pid, wait_result_t *result);
int demo_waitpid_nohang(wait_result_t *result);
int demo_wait_multiple_children(int count);
int demo_waitid(wait_result_t *result);
```

### Partie 5: Zombie and Orphan Handling

```c
// Creation et detection de zombies
int create_zombie(pid_t *zombie_pid);
int detect_zombies(pid_t **zombie_pids, int *count);
int reap_zombie(pid_t zombie_pid);

// Creation et gestion des orphelins
int create_orphan(void);
int demo_reparenting(void);

// Double fork daemon pattern
int daemonize(void);

// SIGCHLD handling
typedef void (*sigchld_handler_t)(int, siginfo_t *, void *);
int setup_sigchld_reaper(void);
int setup_sigchld_ignore(void);  // Auto-reap avec SIG_IGN
int setup_sa_nocldwait(void);    // Auto-reap avec flag
```

---

## Exemple d'Utilisation

```c
#include "process_fundamentals.h"
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    // Partie 1: Information sur le processus courant
    process_info_t self;
    if (proc_get_self_info(&self) == 0) {
        printf("=== Current Process ===\n");
        printf("PID: %d, PPID: %d\n", self.pid, self.ppid);
        printf("UID: %d, GID: %d\n", self.uid, self.gid);
        printf("State: %c\n", self.state);
        printf("Command: %s\n", self.comm);
        printf("Memory: %lu bytes virtual, %ld pages resident\n",
               self.vsize, self.rss);
    }

    // Memory maps
    memory_maps_t *maps = proc_get_maps(getpid());
    if (maps) {
        printf("\n=== Memory Maps ===\n");
        for (size_t i = 0; i < maps->count && i < 5; i++) {
            printf("%lx-%lx %s %s\n",
                   maps->maps[i].start,
                   maps->maps[i].end,
                   maps->maps[i].perms,
                   maps->maps[i].pathname);
        }
        proc_free_maps(maps);
    }

    // Partie 2: Demonstration fork
    printf("\n=== Fork Demonstration ===\n");
    fork_demo_result_t fork_result;
    demo_fork_basic(&fork_result);
    printf("Parent PID: %d, Child PID: %d\n",
           fork_result.parent_pid, fork_result.child_pid);

    // Partie 3: Process listing
    printf("\n=== All Processes ===\n");
    process_list_t *all = proc_list_all();
    if (all) {
        printf("Total processes: %zu\n", all->count);

        // Compter par etat
        int running = 0, sleeping = 0, zombie = 0;
        for (size_t i = 0; i < all->count; i++) {
            switch (all->processes[i].state) {
                case 'R': running++; break;
                case 'S': sleeping++; break;
                case 'Z': zombie++; break;
            }
        }
        printf("Running: %d, Sleeping: %d, Zombie: %d\n",
               running, sleeping, zombie);
        proc_free_list(all);
    }

    // Partie 4: Zombie demonstration
    printf("\n=== Zombie Demonstration ===\n");
    pid_t zombie_pid;
    if (create_zombie(&zombie_pid) == 0) {
        printf("Created zombie with PID %d\n", zombie_pid);

        // Verifier l'etat
        process_info_t zombie_info;
        if (proc_get_info(zombie_pid, &zombie_info) == 0) {
            printf("Zombie state: %c\n", zombie_info.state);
        }

        // Reap it
        reap_zombie(zombie_pid);
        printf("Zombie reaped\n");
    }

    return 0;
}
```

---

## Tests de Validation

```bash
# Compilation
gcc -Wall -Wextra -o process_fundamentals process_fundamentals.c -I.

# Test basique
./process_fundamentals

# Verification avec les outils systeme
./process_fundamentals &
ps aux | grep process_fundamentals
cat /proc/$!/status
cat /proc/$!/maps

# Test zombie
./test_zombie &
ps aux | grep Z

# Test orphan
./test_orphan
# Verifier que init/systemd a adopte l'orphelin
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| Parsing correct de /proc | 20 |
| Implementation fork/exec | 25 |
| Gestion des terminaisons | 20 |
| Detection zombies/orphelins | 20 |
| Robustesse et erreurs | 10 |
| Documentation | 5 |
| **Total** | **100** |
