# [Module 2.2] - Exercise 12: Signal Mastery

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex12"
title: "Signal Mastery - Complete Signal Handling"
difficulty: moyen
estimated_time: "6 heures"
prerequisite_exercises: ["ex04", "ex05"]
concepts_requis: ["signals basics", "process control"]
score_qualite: 97
```

---

## Concepts Couverts

### 2.2.8: Process Groups and Sessions (11 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.8.a | Process group: Collection of related processes | getpgrp(), setpgid() |
| 2.2.8.b | PGID: Process group ID | Manipulation PGID |
| 2.2.8.c | setpgid(): Set process group | Creation de groupes |
| 2.2.8.d | getpgrp(): Get process group | Lecture du groupe |
| 2.2.8.e | Session: Collection of process groups | Sessions et setsid() |
| 2.2.8.f | SID: Session ID | getsid() |
| 2.2.8.g | setsid(): Create new session | Nouvelle session |
| 2.2.8.h | Controlling terminal: Session's terminal | Terminal de controle |
| 2.2.8.i | Foreground group: Gets terminal input | Groupe foreground |
| 2.2.8.j | Background groups: No terminal input | Groupes background |
| 2.2.8.k | tcgetpgrp()/tcsetpgrp(): Foreground group | Control terminal |

### 2.2.9: Job Control (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.9.a | Job: Shell's view of process group | Abstraction job |
| 2.2.9.b | Foreground job: Has terminal | Job au premier plan |
| 2.2.9.c | Background job: & suffix | Job en arriere-plan |
| 2.2.9.d | Stopped job: SIGTSTP/SIGSTOP | Job arrete |
| 2.2.9.e | Job table: Shell's job list | Table des jobs |
| 2.2.9.f | Ctrl+Z: SIGTSTP to foreground | Suspension interactive |
| 2.2.9.g | fg command: Continue foreground | Reprise foreground |
| 2.2.9.h | bg command: Continue background | Reprise background |
| 2.2.9.i | jobs command: List jobs | Liste des jobs |
| 2.2.9.j | disown: Remove from job table | Retrait de la table |
| 2.2.9.k | nohup: Ignore SIGHUP | Immunite SIGHUP |
| 2.2.9.l | Terminal signals: SIGTTIN, SIGTTOU | Signaux terminal |

### 2.2.10: Signals - Basics (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.10.a | Signal: Software interrupt | Concept de signal |
| 2.2.10.b | Signal delivery: Async notification | Livraison asynchrone |
| 2.2.10.c | Default actions: Term, Core, Stop, Ign, Cont | Actions par defaut |
| 2.2.10.d | SIGKILL: Cannot be caught | Signal 9 |
| 2.2.10.e | SIGSTOP: Cannot be caught | Signal 19 |
| 2.2.10.f | SIGTERM: Graceful termination | Signal 15 |
| 2.2.10.g | SIGINT: Ctrl+C | Signal 2 |
| 2.2.10.h | SIGQUIT: Ctrl+\ with core | Signal 3 |
| 2.2.10.i | SIGHUP: Hangup | Signal 1 |
| 2.2.10.j | SIGPIPE: Broken pipe | Signal 13 |
| 2.2.10.k | SIGCHLD: Child status changed | Signal 17 |
| 2.2.10.l | SIGUSR1/2: User-defined | Signaux utilisateur |

### 2.2.11: Signal Handling (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.11.a | signal(): Old API (avoid) | Ancienne API |
| 2.2.11.b | sigaction(): Modern API | Nouvelle API |
| 2.2.11.c | sa_handler: Handler function | Fonction handler |
| 2.2.11.d | sa_sigaction: Extended handler | Handler etendu |
| 2.2.11.e | sa_mask: Blocked during handler | Masque pendant handler |
| 2.2.11.f | sa_flags: SA_RESTART, SA_SIGINFO, etc. | Flags de comportement |
| 2.2.11.g | SA_RESTART: Restart syscalls | Redemarrage syscalls |
| 2.2.11.h | SA_SIGINFO: Use sa_sigaction | Info etendue |
| 2.2.11.i | SA_NOCLDSTOP: No SIGCHLD on stop | Pas de signal stop |
| 2.2.11.j | siginfo_t: Extended signal info | Information etendue |
| 2.2.11.k | SIG_IGN: Ignore signal | Ignorer signal |
| 2.2.11.l | SIG_DFL: Default action | Action par defaut |

### 2.2.12: Signal Masks and Blocking (13 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.12.a | Signal mask: Blocked signals | Masque de signaux |
| 2.2.12.b | sigset_t: Signal set type | Type ensemble |
| 2.2.12.c | sigemptyset(): Empty set | Ensemble vide |
| 2.2.12.d | sigfillset(): Full set | Ensemble plein |
| 2.2.12.e | sigaddset(): Add signal | Ajout signal |
| 2.2.12.f | sigdelset(): Remove signal | Retrait signal |
| 2.2.12.g | sigismember(): Test membership | Test appartenance |
| 2.2.12.h | sigprocmask(): Set/get mask | Manipulation masque |
| 2.2.12.i | SIG_BLOCK, SIG_UNBLOCK, SIG_SETMASK | Operations masque |
| 2.2.12.j | Pending signals: Blocked but delivered | Signaux pendants |
| 2.2.12.k | sigsuspend(): Atomic wait | Attente atomique |
| 2.2.12.l | sigwait(): Synchronous wait | Attente synchrone |
| 2.2.12.m | sigpending(): Get pending set | Signaux pendants |

### 2.2.13: Sending Signals (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.13.a | kill(): Send signal to process | Envoi a processus |
| 2.2.13.b | kill(pid, 0): Check process exists | Verification existence |
| 2.2.13.c | kill(-pgid, sig): Send to group | Envoi au groupe |
| 2.2.13.d | killpg(): Send to process group | Envoi au groupe |
| 2.2.13.e | raise(): Send to self | Envoi a soi-meme |
| 2.2.13.f | alarm(): Schedule SIGALRM | Alarme programmee |
| 2.2.13.g | pause(): Wait for any signal | Attente signal |
| 2.2.13.h | sigqueue(): Send with data | Envoi avec donnees |
| 2.2.13.i | tgkill(): Send to thread (syscall) | Envoi a thread |
| 2.2.13.j | pthread_kill(): Send to thread | Envoi a thread POSIX |
| 2.2.13.k | abort(): Send SIGABRT to self | Auto-abort |
| 2.2.13.l | Signal permissions: uid checks | Permissions d'envoi |

### 2.2.14: Real-Time Signals (9 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.14.a | RT signal range: SIGRTMIN to SIGRTMAX | Plage temps reel |
| 2.2.14.b | RT signals queue: Multiple instances | File d'attente |
| 2.2.14.c | RT signal ordering: FIFO | Ordre FIFO |
| 2.2.14.d | RT signal priority: Lower number = higher | Priorite |
| 2.2.14.e | sigqueue(): Send RT signal with data | Envoi avec donnees |
| 2.2.14.f | si_value: Union for data | Valeur associee |
| 2.2.14.g | si_int: Integer data | Donnee entiere |
| 2.2.14.h | si_ptr: Pointer data | Donnee pointeur |
| 2.2.14.i | sigwaitinfo(): Wait with info | Attente avec info |

### 2.2.15: Async-Signal-Safety (9 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.15.a | Problem: Handler interrupts anything | Probleme d'interruption |
| 2.2.15.b | Async-signal-safe functions: Safe list | Fonctions sures |
| 2.2.15.c | Unsafe: malloc, printf, etc. | Fonctions dangereuses |
| 2.2.15.d | errno: Must save/restore | Sauvegarde errno |
| 2.2.15.e | volatile sig_atomic_t: Safe flag type | Type atomique |
| 2.2.15.f | Self-pipe trick: Safe notification | Technique self-pipe |
| 2.2.15.g | signalfd(): Signal via file descriptor | Signal comme fd |
| 2.2.15.h | eventfd(): Event notification | Notification evenement |
| 2.2.15.i | Pattern: Set flag, handle in main loop | Pattern standard |

---

## Contexte

Les signaux constituent le mecanisme fondamental de communication asynchrone entre processus sous Unix. Contrairement aux autres formes d'IPC qui transferent des donnees, les signaux sont de simples notifications - des "interruptions logicielles" qui informent un processus qu'un evenement s'est produit.

La maitrise des signaux est essentielle pour developper des applications systeme robustes capables de reagir gracieusement aux evenements externes (terminaison demandee, timeout, erreurs), de gerer correctement les processus enfants, et d'implementer des protocoles de communication inter-processus.

Cet exercice couvre tous les aspects de la gestion des signaux, depuis les concepts de base jusqu'aux signaux temps reel et la securite asynchrone.

---

## Enonce

### Vue d'Ensemble

Implementez une bibliotheque complete de gestion des signaux avec demonstrations de tous les concepts, incluant les process groups, le job control, et les mecanismes avances.

### Partie 1: Process Groups and Sessions

```c
#include <sys/types.h>
#include <unistd.h>
#include <signal.h>

// Information sur les groupes
typedef struct {
    pid_t pid;
    pid_t pgid;
    pid_t sid;
    int is_session_leader;
    int is_group_leader;
    int has_controlling_terminal;
    char tty_name[256];
} process_group_info_t;

int get_process_group_info(pid_t pid, process_group_info_t *info);

// Manipulation des groupes
int create_process_group(pid_t pid);           // setpgid(pid, pid)
int join_process_group(pid_t pid, pid_t pgid); // setpgid(pid, pgid)
int create_session(void);                       // setsid()

// Controle du terminal
int get_foreground_group(int fd, pid_t *pgid);
int set_foreground_group(int fd, pid_t pgid);

// Demonstration
int demo_process_groups(void);
int demo_session_creation(void);
int demo_terminal_control(void);
```

### Partie 2: Job Control Implementation

```c
typedef enum {
    JOB_RUNNING,
    JOB_STOPPED,
    JOB_TERMINATED
} job_state_t;

typedef struct job {
    int id;
    pid_t pgid;
    job_state_t state;
    int status;
    char command[256];
    int foreground;
    struct job *next;
} job_t;

typedef struct {
    job_t *jobs;
    int next_id;
    int terminal_fd;
    pid_t shell_pgid;
} job_control_t;

// API Job Control
job_control_t *job_control_init(void);
void job_control_destroy(job_control_t *jc);

job_t *job_create(job_control_t *jc, const char *cmd, pid_t pgid);
void job_delete(job_control_t *jc, int job_id);
job_t *job_find(job_control_t *jc, int job_id);
job_t *job_find_by_pgid(job_control_t *jc, pid_t pgid);

int job_continue_foreground(job_control_t *jc, int job_id);
int job_continue_background(job_control_t *jc, int job_id);
int job_stop(job_control_t *jc, int job_id);
void job_update_status(job_control_t *jc);
void job_list(job_control_t *jc);
```

### Partie 3: Signal Handling Framework

```c
// Configuration de handler avec sigaction
typedef struct {
    int signum;
    void (*handler)(int);
    void (*sa_handler)(int, siginfo_t *, void *);
    sigset_t mask;
    int flags;                  // SA_RESTART, SA_SIGINFO, etc.
} signal_config_t;

int signal_setup(const signal_config_t *config);
int signal_setup_simple(int signum, void (*handler)(int));
int signal_setup_extended(int signum,
                          void (*handler)(int, siginfo_t *, void *));
int signal_ignore(int signum);
int signal_default(int signum);

// Sauvegarde et restauration
typedef struct {
    struct sigaction actions[64];  // NSIG
    sigset_t mask;
} signal_state_t;

int signal_save_state(signal_state_t *state);
int signal_restore_state(const signal_state_t *state);
```

### Partie 4: Signal Masks and Blocking

```c
// Manipulation de masques
typedef struct {
    sigset_t set;
} signal_mask_t;

signal_mask_t *signal_mask_create(void);
void signal_mask_destroy(signal_mask_t *mask);
int signal_mask_empty(signal_mask_t *mask);
int signal_mask_fill(signal_mask_t *mask);
int signal_mask_add(signal_mask_t *mask, int signum);
int signal_mask_remove(signal_mask_t *mask, int signum);
int signal_mask_contains(const signal_mask_t *mask, int signum);

// Application du masque
int signal_block(const signal_mask_t *mask);
int signal_unblock(const signal_mask_t *mask);
int signal_set_mask(const signal_mask_t *mask);
int signal_get_mask(signal_mask_t *mask);
int signal_get_pending(signal_mask_t *pending);

// Attente de signaux
int signal_wait(const signal_mask_t *mask);          // sigsuspend
int signal_wait_info(const signal_mask_t *mask,
                     siginfo_t *info);                // sigwaitinfo
int signal_timedwait(const signal_mask_t *mask,
                     siginfo_t *info,
                     int timeout_ms);                 // sigtimedwait
```

### Partie 5: Sending Signals

```c
// Envoi de signaux
int signal_send(pid_t pid, int signum);
int signal_send_group(pid_t pgid, int signum);
int signal_send_self(int signum);
int signal_check_process(pid_t pid);         // kill(pid, 0)

// Alarmes et timers
unsigned int signal_alarm(unsigned int seconds);
int signal_pause(void);                       // Attente d'un signal

// Signaux temps reel avec donnees
int signal_queue(pid_t pid, int signum, int value);
int signal_queue_ptr(pid_t pid, int signum, void *ptr);
```

### Partie 6: Real-Time Signals

```c
// Information sur les signaux RT
typedef struct {
    int sigrtmin;
    int sigrtmax;
    int count;
} rt_signal_info_t;

int rt_signal_get_info(rt_signal_info_t *info);

// Envoi de signaux RT
int rt_signal_send(pid_t pid, int signum, int value);
int rt_signal_send_ptr(pid_t pid, int signum, void *ptr);

// Reception de signaux RT
typedef struct {
    int signum;
    int si_code;
    pid_t si_pid;
    uid_t si_uid;
    union sigval si_value;
} rt_signal_data_t;

int rt_signal_wait(const sigset_t *set, rt_signal_data_t *data);
```

### Partie 7: Async-Signal-Safety

```c
// Self-pipe trick
typedef struct {
    int read_fd;
    int write_fd;
} self_pipe_t;

self_pipe_t *self_pipe_create(void);
void self_pipe_destroy(self_pipe_t *sp);
int self_pipe_notify(self_pipe_t *sp, int signum);
int self_pipe_wait(self_pipe_t *sp, int *signum);

// signalfd (Linux-specific)
typedef struct {
    int fd;
    sigset_t mask;
} signal_fd_t;

signal_fd_t *signal_fd_create(const sigset_t *mask);
void signal_fd_destroy(signal_fd_t *sfd);
int signal_fd_read(signal_fd_t *sfd, struct signalfd_siginfo *info);

// Flag atomique
typedef volatile sig_atomic_t atomic_flag_t;

void atomic_flag_set(atomic_flag_t *flag);
void atomic_flag_clear(atomic_flag_t *flag);
int atomic_flag_test(const atomic_flag_t *flag);

// Pattern safe
typedef struct {
    atomic_flag_t flags[32];     // Un par signal d'interet
    self_pipe_t *pipe;
} safe_signal_handler_t;

safe_signal_handler_t *safe_handler_create(void);
void safe_handler_destroy(safe_signal_handler_t *h);
int safe_handler_check(safe_signal_handler_t *h, int signum);
int safe_handler_wait(safe_signal_handler_t *h);
```

---

## Exemple d'Utilisation

```c
#include "signal_mastery.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Variables globales pour les handlers
static atomic_flag_t got_sigint = 0;
static atomic_flag_t got_sigterm = 0;

// Handler async-signal-safe
void handle_sigint(int sig) {
    (void)sig;
    atomic_flag_set(&got_sigint);
}

void handle_sigterm(int sig) {
    (void)sig;
    atomic_flag_set(&got_sigterm);
}

int main(void) {
    // Setup des handlers
    signal_setup_simple(SIGINT, handle_sigint);
    signal_setup_simple(SIGTERM, handle_sigterm);

    // Demonstration des process groups
    printf("=== Process Group Info ===\n");
    process_group_info_t pg_info;
    get_process_group_info(getpid(), &pg_info);
    printf("PID: %d, PGID: %d, SID: %d\n",
           pg_info.pid, pg_info.pgid, pg_info.sid);

    // Demonstration du masquage
    printf("\n=== Signal Blocking ===\n");
    signal_mask_t *mask = signal_mask_create();
    signal_mask_add(mask, SIGUSR1);
    signal_mask_add(mask, SIGUSR2);
    signal_block(mask);
    printf("SIGUSR1 and SIGUSR2 blocked\n");

    // Envoi d'un signal bloque
    signal_send_self(SIGUSR1);
    printf("Sent SIGUSR1 to self (blocked, now pending)\n");

    // Verifier les signaux pendants
    signal_mask_t *pending = signal_mask_create();
    signal_get_pending(pending);
    if (signal_mask_contains(pending, SIGUSR1)) {
        printf("SIGUSR1 is pending\n");
    }

    // Debloquer
    signal_unblock(mask);
    printf("Signals unblocked (SIGUSR1 delivered)\n");

    signal_mask_destroy(mask);
    signal_mask_destroy(pending);

    // Self-pipe trick
    printf("\n=== Self-Pipe Trick ===\n");
    self_pipe_t *sp = self_pipe_create();
    if (sp) {
        // Dans un handler reel, on ecrirait dans le pipe
        // Ici on simule
        self_pipe_notify(sp, SIGINT);

        int sig;
        if (self_pipe_wait(sp, &sig) == 0) {
            printf("Received signal %d via self-pipe\n", sig);
        }
        self_pipe_destroy(sp);
    }

    // Real-time signals
    printf("\n=== Real-Time Signals ===\n");
    rt_signal_info_t rt_info;
    rt_signal_get_info(&rt_info);
    printf("RT signals: %d to %d (%d available)\n",
           rt_info.sigrtmin, rt_info.sigrtmax, rt_info.count);

    // Boucle principale safe
    printf("\n=== Main Loop (Ctrl+C to exit) ===\n");
    while (!atomic_flag_test(&got_sigint) &&
           !atomic_flag_test(&got_sigterm)) {
        // Travail normal...
        sleep(1);
        printf(".");
        fflush(stdout);
    }

    printf("\nReceived termination signal, exiting cleanly\n");
    return 0;
}
```

---

## Tests de Validation

```bash
# Compilation
gcc -Wall -Wextra -o signal_mastery signal_mastery.c -I.

# Test basique
./signal_mastery

# Test avec signaux
./signal_mastery &
PID=$!
kill -USR1 $PID
kill -USR2 $PID
kill -TERM $PID

# Test job control
./test_job_control

# Test du self-pipe
./test_self_pipe

# Test des signaux RT
./test_rt_signals
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| Process groups et sessions | 15 |
| Job control | 15 |
| Handlers sigaction | 20 |
| Masquage et blocage | 15 |
| Envoi de signaux | 10 |
| Signaux temps reel | 10 |
| Async-signal-safety | 10 |
| Documentation | 5 |
| **Total** | **100** |
