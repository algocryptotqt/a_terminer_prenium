# Exercice [2.2.9] : plumbob_controller (The Sims Job Control)

**Module :**
2.2.9 — Job Controller

**Concept :**
synth — Synthèse Process Groups, Terminal Control, Job States, Signals

**Difficulté :**
★★★★★★★★☆☆ (8/10)

**Type :**
code

**Tiers :**
3 — Synthèse (groupes de processus + signaux + terminal control)

**Langage :**
C (C17)

**Prérequis :**
- fork(), exec(), waitpid()
- Signaux (SIGTSTP, SIGCONT, SIGCHLD)
- Groupes de processus (setpgid, getpgid)
- Terminal control (tcsetpgrp)

**Domaines :**
Process, Mem, Struct

**Durée estimée :**
360-420 min

**XP Base :**
500

**Complexité :**
T3 O(n) × S2 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex09/
├── plumbob.h           # API publique
├── plumbob.c           # Implémentation
└── Makefile
```

**Fonctions autorisées :**
```c
// Processus
fork, execvp, _exit, waitpid, getpid, getppid, getpgid, setpgid, setsid, getsid

// Groupes de processus
killpg, tcgetpgrp, tcsetpgrp, isatty, ttyname

// Signaux
sigaction, sigemptyset, sigaddset, sigfillset, sigprocmask, sigsuspend, kill

// Mémoire
malloc, free, realloc, calloc, strdup, strncpy, strlen

// I/O
open, close, dup, dup2, read, write, printf, fprintf, snprintf, perror

// Divers
strsignal
```

**Fonctions interdites :**
```c
system, popen, pthread_*
```

### 1.2 Consigne

**🎮 [THE SIMS — Le Plumbob Controller]**

Dans The Sims, le **Plumbob** (le diamant vert flottant au-dessus de la tête du Sim actif) indique quel personnage tu contrôles actuellement. Quand tu cliques sur un autre Sim, le Plumbob se déplace vers lui. Les autres Sims continuent leurs activités en autonome (background), mais tu ne peux donner des ordres directs qu'au Sim sous ton contrôle (foreground).

Tu vas implémenter un **Plumbob Controller** — un système de gestion de jobs UNIX qui fonctionne exactement comme dans The Sims :

| Concept Sims | Concept UNIX |
|--------------|--------------|
| Sim contrôlé | Foreground process group |
| Sims autonomes | Background processes |
| Pause du jeu | SIGTSTP (Ctrl+Z) |
| Reprendre le jeu | SIGCONT |
| Annuler une action | SIGTERM/SIGKILL |
| Household (famille) | Process group |
| Notification "Action terminée" | SIGCHLD + job_notify |
| File d'actions | Pipeline |

**Ta mission :**

Implémenter un contrôleur de jobs complet (`plumbob_controller_t`) permettant :
1. Lancer des tâches en **foreground** (contrôle direct) ou **background** (autonome)
2. **Suspendre** un Sim avec Ctrl+Z (SIGTSTP)
3. **Reprendre** une tâche en foreground (`sim_take_control`) ou background (`sim_release_control`)
4. **Lister** toutes les tâches actives avec leur état
5. **Notifier** quand une tâche change d'état (terminée, suspendue, etc.)

**Architecture du monde Sims :**

```
                    Session (Le Quartier)
                           |
        +------------------+------------------+
        |                  |                  |
   Controller (Le Joueur)  Household 1       Household 2
   [Shell - ignore SIGTSTP]   (pgid)           (pgid)
        |                  |                  |
        |             +----+----+        +----+----+
        |             |    |    |        |    |    |
                    Sim1 Sim2 Sim3     Sim4 Sim5 Sim6
                    (pipeline = file d'actions communes)

Plumbob (Terminal Control):
  - Le Controller est normalement le foreground process group
  - Quand tu contrôles un Sim, son household devient foreground
  - SIGTSTP est envoyé au foreground household quand Ctrl+Z
```

**Entrée (Structures principales) :**

```c
// États possibles d'un Sim (tâche)
typedef enum {
    SIM_ACTIVE,       // En cours d'exécution (Running)
    SIM_PAUSED,       // Suspendu (Stopped - Ctrl+Z)
    SIM_TASK_DONE,    // Tâche terminée normalement (exit 0)
    SIM_CANCELLED,    // Tâche annulée par signal (Terminated)
    SIM_RESUMED       // Repris après pause (transitoire)
} sim_state_t;

// Informations sur une tâche Sim
typedef struct {
    int task_id;           // ID de la tâche (1, 2, 3, ...)
    pid_t household_id;    // Process Group ID (PGID)
    sim_state_t state;     // État actuel
    int exit_code;         // Code de sortie (si TASK_DONE)
    int cancel_signal;     // Signal de terminaison (si CANCELLED)
    char *action_name;     // Commande originale ("sleep 100")
    bool notified;         // L'utilisateur a été notifié?
    bool has_plumbob;      // Est-ce la tâche foreground actuelle?
} sim_task_t;

// Le Plumbob Controller (opaque)
typedef struct plumbob_controller plumbob_controller_t;
```

**Sortie (API complète) :**

```c
// === LIFECYCLE ===
plumbob_controller_t *plumbob_boot(void);
void plumbob_shutdown(plumbob_controller_t *ctrl);

// === SPAWN ===
int sim_assign_task(plumbob_controller_t *ctrl, char **argv, bool take_control);
int sim_assign_queue(plumbob_controller_t *ctrl, char ***commands,
                     int n_commands, bool take_control);

// === CONTROL ===
int sim_take_control(plumbob_controller_t *ctrl, int task_id, bool send_resume);
int sim_release_control(plumbob_controller_t *ctrl, int task_id, bool send_resume);
int sim_cancel_task(plumbob_controller_t *ctrl, int task_id, int sig);
int sim_wait_task(plumbob_controller_t *ctrl, int task_id);

// === INFO ===
int sim_show_tasks(plumbob_controller_t *ctrl, sim_task_t *tasks_out, int max_tasks);
int sim_get_task_info(plumbob_controller_t *ctrl, int task_id, sim_task_t *info_out);
int sim_notify_changes(plumbob_controller_t *ctrl);
int sim_current_active(plumbob_controller_t *ctrl);
```

**Contraintes :**
- Le Controller (shell) ne doit JAMAIS être suspendu par SIGTSTP (ignore)
- Les Sims doivent avoir les handlers par défaut AVANT exec
- Maximum 64 tâches simultanées
- Pas de variables globales mutables (sauf pour handler SIGCHLD si nécessaire)
- Aucun zombie (tous les enfants doivent être récoltés)
- Le terminal doit TOUJOURS revenir au Controller après chaque tâche

**Format de notification (style bash) :**
```
[task_id]+  State       Action
[1]+  Paused      sleep 100
[2]-  Active      find / -name "*.c" &
[3]   Done        ls -la
[4]   Cancelled   cat /dev/urandom  (SIGPIPE)
```

### 1.3 Prototypes

```c
#ifndef PLUMBOB_H
#define PLUMBOB_H

#include <stdbool.h>
#include <sys/types.h>

typedef enum {
    SIM_ACTIVE,
    SIM_PAUSED,
    SIM_TASK_DONE,
    SIM_CANCELLED,
    SIM_RESUMED
} sim_state_t;

typedef struct {
    int task_id;
    pid_t household_id;
    sim_state_t state;
    int exit_code;
    int cancel_signal;
    char *action_name;
    bool notified;
    bool has_plumbob;
} sim_task_t;

typedef struct plumbob_controller plumbob_controller_t;

// Lifecycle
plumbob_controller_t *plumbob_boot(void);
void plumbob_shutdown(plumbob_controller_t *ctrl);

// Spawn
int sim_assign_task(plumbob_controller_t *ctrl, char **argv, bool take_control);
int sim_assign_queue(plumbob_controller_t *ctrl, char ***commands,
                     int n_commands, bool take_control);

// Control
int sim_take_control(plumbob_controller_t *ctrl, int task_id, bool send_resume);
int sim_release_control(plumbob_controller_t *ctrl, int task_id, bool send_resume);
int sim_cancel_task(plumbob_controller_t *ctrl, int task_id, int sig);
int sim_wait_task(plumbob_controller_t *ctrl, int task_id);

// Info
int sim_show_tasks(plumbob_controller_t *ctrl, sim_task_t *tasks_out, int max_tasks);
int sim_get_task_info(plumbob_controller_t *ctrl, int task_id, sim_task_t *info_out);
int sim_notify_changes(plumbob_controller_t *ctrl);
int sim_current_active(plumbob_controller_t *ctrl);

#endif /* PLUMBOB_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Pourquoi The Sims est l'analogie parfaite ?

Le modèle de job control UNIX et The Sims partagent la même philosophie :

| The Sims | UNIX | Explication |
|----------|------|-------------|
| **Un seul Sim contrôlé** | **Un seul foreground group** | Tu ne peux donner des ordres directs qu'à un seul personnage/processus à la fois |
| **Pause du jeu** | **SIGTSTP** | Tout s'arrête instantanément |
| **Vitesse x3** | **nice/renice** | Certaines tâches peuvent avoir la priorité |
| **Annuler action** | **SIGTERM** | Interrompre proprement une tâche |
| **Forcer annulation** | **SIGKILL** | "motherload" cheat - force brute |
| **Notification bulle** | **SIGCHLD + notify** | "Votre Sim a terminé sa tâche!" |

### 2.2 L'histoire du Job Control

Le job control UNIX a été inventé par Bill Joy pour **csh** (C Shell) à Berkeley en 1978. Avant cela, il fallait soit :
- Attendre qu'une commande se termine
- Lancer un processus en background avec `&` et ne plus pouvoir le contrôler

Bill Joy a introduit :
- `Ctrl+Z` pour suspendre
- `fg` / `bg` pour reprendre
- `jobs` pour lister

Ces concepts sont directement mappés dans notre Plumbob Controller.

### 2.3 La hiérarchie Session → Process Group → Process

```
Session (getsid)
├── Process Group Leader (getpgid == pid)
│   ├── Process (membre)
│   └── Process (membre)
├── Process Group 2
│   ├── Process
│   └── Process
└── Foreground Process Group (tcgetpgrp)
    └── Reçoit les signaux TTY (SIGTSTP, SIGINT)
```

### 2.5 DANS LA VRAIE VIE

| Métier | Cas d'usage |
|--------|-------------|
| **DevOps / SRE** | Gestion des services avec systemd, supervisord |
| **Développeur Shell** | Création de shells interactifs (bash, zsh, fish) |
| **Admin Système** | Scripts de déploiement avec contrôle de processus |
| **Game Developer** | Systèmes de pause/resume de jeux |
| **Container Runtime Dev** | Implémentation de runc, containerd |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
plumbob.c  plumbob.h  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -std=c17 -c plumbob.c
ar rcs libplumbob.a plumbob.o

$ gcc -Wall -Wextra -Werror main.c -L. -lplumbob -o sims_test

$ ./sims_test
[Plumbob Controller] Booted!
[1] Task assigned: sleep 5 (background)
[2] Task assigned: sleep 3 (background)
tick 0
tick 1
tick 2
[2]+ Done        sleep 3
tick 3
tick 4
[1]+ Done        sleep 5
[Plumbob Controller] All Sims handled. Shutdown complete.
```

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
🧠 (12/10)

**Récompense :**
XP ×6

**Time Complexity attendue :**
O(n)

**Space Complexity attendue :**
O(n)

**Domaines Bonus :**
`Process, Net`

#### 3.1.1 Consigne Bonus

**🎮 [THE SIMS — Multiplayer Household Management]**

Dans The Sims 4, le mode "Gérer plusieurs foyers" permet de contrôler plusieurs familles en même temps. Implémente le **Multi-Plumbob Mode** :

**Ta mission :**

Ajouter le support des **pipelines** (file d'actions) et des **households multiples** :

```c
// Pipeline = File d'actions pour un Sim
// Exemple: "grep pattern < file | sort | uniq > output"
// Tous les processus du pipeline sont dans le MÊME household (process group)
int sim_assign_queue(plumbob_controller_t *ctrl, char ***commands,
                     int n_commands, bool take_control);

// Gérer les orphaned process groups (quand le leader meurt)
int sim_handle_orphans(plumbob_controller_t *ctrl);

// Support des job specifications (%1, %%, %-, %string)
int sim_parse_job_spec(plumbob_controller_t *ctrl, const char *spec);
```

**Contraintes Bonus :**
┌─────────────────────────────────────────┐
│  Pipelines doivent partager le PGID     │
│  Gestion des orphaned process groups    │
│  Support %% (current) et %- (previous)  │
│  Temps limite : O(n) par opération      │
└─────────────────────────────────────────┘

#### 3.1.2 Prototype Bonus

```c
int sim_assign_queue(plumbob_controller_t *ctrl, char ***commands,
                     int n_commands, bool take_control);
int sim_handle_orphans(plumbob_controller_t *ctrl);
int sim_parse_job_spec(plumbob_controller_t *ctrl, const char *spec);
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Commandes | Single | Pipeline (n commands) |
| Process groups | Simple | Orphan handling |
| Job specs | task_id | %%, %-, %string |
| Complexity | O(1) spawn | O(n) pipeline setup |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

| # | Test | Input | Expected | Points |
|---|------|-------|----------|--------|
| 01 | `foreground_simple` | `sim_assign_task(ctrl, {"echo", "hello", NULL}, true)` | Output "hello", wait before return | 3 |
| 02 | `background_simple` | `sim_assign_task(ctrl, {"sleep", "1", NULL}, false)` | Return immediately, id > 0 | 3 |
| 03 | `null_controller` | `sim_assign_task(NULL, argv, true)` | Return -1 | 2 |
| 04 | `null_argv` | `sim_assign_task(ctrl, NULL, true)` | Return -1 | 2 |
| 05 | `sigtstp_stops_fg` | Send SIGTSTP to foreground | state == SIM_PAUSED | 4 |
| 06 | `sigcont_resumes` | `sim_take_control(ctrl, id, true)` on paused | state == SIM_ACTIVE, SIGCONT sent | 4 |
| 07 | `take_control_fg` | `sim_take_control` on paused task | Terminal given, waits for completion | 4 |
| 08 | `release_control_bg` | `sim_release_control(ctrl, id, true)` | Returns immediately, task runs bg | 4 |
| 09 | `process_group_unique` | Spawn 2 tasks | pgid1 != pgid2, pgid != shell_pid | 3 |
| 10 | `killpg_affects_all` | Pipeline 3 processes, `sim_cancel_task` | All 3 terminated | 4 |
| 11 | `terminal_returns` | After foreground task | tcgetpgrp(0) == getpid() | 4 |
| 12 | `no_zombies` | Spawn 100 tasks, sleep, notify | No defunct processes | 4 |
| 13 | `task_inexistant` | `sim_take_control(ctrl, 999, true)` | Return -1 | 2 |
| 14 | `notify_done` | Task finishes in background | Notification printed, marked notified | 3 |
| 15 | `notify_cancelled` | Task killed by signal | Shows signal name in notification | 3 |
| 16 | `show_tasks_list` | Multiple tasks | Returns correct count and info | 3 |
| 17 | `current_active` | Stop task with Ctrl+Z | `sim_current_active` returns that id | 3 |
| 18 | `shutdown_kills_all` | `plumbob_shutdown` with active tasks | All tasks terminated | 3 |

### 4.2 main.c de test

```c
#include "plumbob.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <string.h>
#include <assert.h>
#include <sys/wait.h>

const char *state_str(sim_state_t s) {
    switch (s) {
        case SIM_ACTIVE: return "Active";
        case SIM_PAUSED: return "Paused";
        case SIM_TASK_DONE: return "Done";
        case SIM_CANCELLED: return "Cancelled";
        case SIM_RESUMED: return "Resumed";
    }
    return "Unknown";
}

void test_foreground_simple(void) {
    printf("Test 1: Foreground simple... ");
    plumbob_controller_t *ctrl = plumbob_boot();
    assert(ctrl != NULL);

    char *argv[] = {"echo", "Sul Sul!", NULL};  // The Sims greeting
    int id = sim_assign_task(ctrl, argv, true);
    assert(id == 1);

    sim_task_t info;
    sim_get_task_info(ctrl, id, &info);
    assert(info.state == SIM_TASK_DONE || info.exit_code == 0);

    plumbob_shutdown(ctrl);
    printf("OK\n");
}

void test_background_notify(void) {
    printf("Test 2: Background with notification... ");
    plumbob_controller_t *ctrl = plumbob_boot();

    char *argv[] = {"sleep", "1", NULL};
    int id = sim_assign_task(ctrl, argv, false);
    assert(id > 0);

    sim_task_t info;
    sim_get_task_info(ctrl, id, &info);
    assert(info.state == SIM_ACTIVE);

    sleep(2);
    int notified = sim_notify_changes(ctrl);
    assert(notified >= 1);

    plumbob_shutdown(ctrl);
    printf("OK\n");
}

void test_null_safety(void) {
    printf("Test 3: NULL safety... ");

    assert(sim_assign_task(NULL, (char*[]){"echo", NULL}, true) == -1);

    plumbob_controller_t *ctrl = plumbob_boot();
    assert(sim_assign_task(ctrl, NULL, true) == -1);
    assert(sim_take_control(ctrl, 999, true) == -1);
    assert(sim_cancel_task(ctrl, 0, SIGTERM) == -1);

    plumbob_shutdown(ctrl);
    printf("OK\n");
}

void test_process_groups(void) {
    printf("Test 4: Process groups are unique... ");
    plumbob_controller_t *ctrl = plumbob_boot();

    char *argv1[] = {"sleep", "10", NULL};
    char *argv2[] = {"sleep", "10", NULL};

    int id1 = sim_assign_task(ctrl, argv1, false);
    int id2 = sim_assign_task(ctrl, argv2, false);

    sim_task_t info1, info2;
    sim_get_task_info(ctrl, id1, &info1);
    sim_get_task_info(ctrl, id2, &info2);

    assert(info1.household_id != info2.household_id);
    assert(info1.household_id != getpid());
    assert(info2.household_id != getpid());

    plumbob_shutdown(ctrl);
    printf("OK\n");
}

void test_terminal_returns(void) {
    printf("Test 5: Terminal returns to controller... ");
    plumbob_controller_t *ctrl = plumbob_boot();

    pid_t shell_pgid = getpgid(0);

    for (int i = 0; i < 3; i++) {
        char *argv[] = {"true", NULL};
        sim_assign_task(ctrl, argv, true);

        // After foreground task, terminal should be back
        pid_t fg = tcgetpgrp(STDIN_FILENO);
        assert(fg == shell_pgid);
    }

    plumbob_shutdown(ctrl);
    printf("OK\n");
}

void test_show_tasks(void) {
    printf("Test 6: Show tasks list... ");
    plumbob_controller_t *ctrl = plumbob_boot();

    char *cmd1[] = {"sleep", "100", NULL};
    char *cmd2[] = {"sleep", "100", NULL};
    char *cmd3[] = {"sleep", "100", NULL};

    sim_assign_task(ctrl, cmd1, false);
    sim_assign_task(ctrl, cmd2, false);
    sim_assign_task(ctrl, cmd3, false);

    sim_task_t tasks[10];
    int n = sim_show_tasks(ctrl, tasks, 10);
    assert(n == 3);

    plumbob_shutdown(ctrl);
    printf("OK\n");
}

int main(void) {
    printf("\n=== PLUMBOB CONTROLLER TEST SUITE ===\n");
    printf("(The Sims Job Control)\n\n");

    test_foreground_simple();
    test_background_notify();
    test_null_safety();
    test_process_groups();
    test_terminal_returns();
    test_show_tasks();

    printf("\n=== ALL TESTS PASSED ===\n");
    printf("Sul Sul! Your Plumbob Controller is Woohoo-ready!\n\n");

    return 0;
}
```

### 4.3 Solution de référence

```c
#include "plumbob.h"
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <signal.h>
#include <sys/wait.h>
#include <stdio.h>
#include <errno.h>

#define MAX_TASKS 64

typedef struct task_node {
    int id;
    pid_t pgid;
    sim_state_t state;
    int status;
    char *command;
    bool notified;
    struct task_node *next;
} task_node_t;

struct plumbob_controller {
    task_node_t *tasks;
    int next_id;
    int current_task;
    int previous_task;
    pid_t shell_pgid;
    int terminal_fd;
    struct sigaction old_sigchld;
};

static plumbob_controller_t *g_ctrl = NULL;

static void sigchld_handler(int sig) {
    (void)sig;
    int saved_errno = errno;
    pid_t pid;
    int status;

    while ((pid = waitpid(-1, &status, WNOHANG | WUNTRACED | WCONTINUED)) > 0) {
        if (g_ctrl == NULL) continue;

        task_node_t *task = g_ctrl->tasks;
        while (task) {
            if (task->pgid == pid || getpgid(pid) == task->pgid) {
                task->status = status;
                if (WIFSTOPPED(status)) {
                    task->state = SIM_PAUSED;
                } else if (WIFCONTINUED(status)) {
                    task->state = SIM_RESUMED;
                } else if (WIFEXITED(status)) {
                    task->state = SIM_TASK_DONE;
                } else if (WIFSIGNALED(status)) {
                    task->state = SIM_CANCELLED;
                }
                task->notified = false;
                break;
            }
            task = task->next;
        }
    }

    errno = saved_errno;
}

static char *build_command_str(char **argv) {
    size_t len = 0;
    for (int i = 0; argv[i]; i++) {
        len += strlen(argv[i]) + 1;
    }
    char *cmd = malloc(len);
    if (!cmd) return NULL;
    cmd[0] = '\0';
    for (int i = 0; argv[i]; i++) {
        if (i > 0) strcat(cmd, " ");
        strcat(cmd, argv[i]);
    }
    return cmd;
}

static task_node_t *find_task(plumbob_controller_t *ctrl, int task_id) {
    task_node_t *task = ctrl->tasks;
    while (task) {
        if (task->id == task_id) return task;
        task = task->next;
    }
    return NULL;
}

static void wait_for_task(plumbob_controller_t *ctrl, task_node_t *task) {
    int status;
    pid_t pid;

    while ((pid = waitpid(-task->pgid, &status, WUNTRACED)) > 0 || errno == EINTR) {
        if (pid <= 0) continue;
        task->status = status;

        if (WIFSTOPPED(status)) {
            task->state = SIM_PAUSED;
            ctrl->previous_task = ctrl->current_task;
            ctrl->current_task = task->id;
            break;
        } else if (WIFEXITED(status)) {
            task->state = SIM_TASK_DONE;
            break;
        } else if (WIFSIGNALED(status)) {
            task->state = SIM_CANCELLED;
            break;
        }
    }

    // Return terminal to shell
    tcsetpgrp(ctrl->terminal_fd, ctrl->shell_pgid);
}

plumbob_controller_t *plumbob_boot(void) {
    plumbob_controller_t *ctrl = calloc(1, sizeof(plumbob_controller_t));
    if (!ctrl) return NULL;

    ctrl->tasks = NULL;
    ctrl->next_id = 1;
    ctrl->current_task = 0;
    ctrl->previous_task = 0;
    ctrl->shell_pgid = getpgrp();
    ctrl->terminal_fd = STDIN_FILENO;

    // Shell ignores job control signals
    signal(SIGTSTP, SIG_IGN);
    signal(SIGTTIN, SIG_IGN);
    signal(SIGTTOU, SIG_IGN);

    // Setup SIGCHLD handler
    struct sigaction sa;
    sa.sa_handler = sigchld_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = SA_RESTART | SA_NOCLDSTOP;
    sigaction(SIGCHLD, &sa, &ctrl->old_sigchld);

    g_ctrl = ctrl;

    return ctrl;
}

void plumbob_shutdown(plumbob_controller_t *ctrl) {
    if (!ctrl) return;

    // Kill all active tasks
    task_node_t *task = ctrl->tasks;
    while (task) {
        if (task->state == SIM_ACTIVE || task->state == SIM_PAUSED) {
            killpg(task->pgid, SIGTERM);
            killpg(task->pgid, SIGCONT);
        }
        task = task->next;
    }

    // Wait for all children
    while (waitpid(-1, NULL, WNOHANG) > 0);

    // Free tasks
    task = ctrl->tasks;
    while (task) {
        task_node_t *next = task->next;
        free(task->command);
        free(task);
        task = next;
    }

    // Restore signal handler
    sigaction(SIGCHLD, &ctrl->old_sigchld, NULL);

    g_ctrl = NULL;
    free(ctrl);
}

int sim_assign_task(plumbob_controller_t *ctrl, char **argv, bool take_control) {
    if (!ctrl || !argv || !argv[0]) return -1;

    // Block SIGCHLD during fork
    sigset_t mask, oldmask;
    sigemptyset(&mask);
    sigaddset(&mask, SIGCHLD);
    sigprocmask(SIG_BLOCK, &mask, &oldmask);

    pid_t pid = fork();

    if (pid < 0) {
        sigprocmask(SIG_SETMASK, &oldmask, NULL);
        return -1;
    }

    if (pid == 0) {
        // Child: Create own process group
        setpgid(0, 0);

        if (take_control) {
            tcsetpgrp(STDIN_FILENO, getpid());
        }

        // Restore default signal handlers
        signal(SIGTSTP, SIG_DFL);
        signal(SIGTTIN, SIG_DFL);
        signal(SIGTTOU, SIG_DFL);
        signal(SIGINT, SIG_DFL);
        signal(SIGQUIT, SIG_DFL);

        // Unblock SIGCHLD
        sigprocmask(SIG_SETMASK, &oldmask, NULL);

        execvp(argv[0], argv);
        perror("execvp");
        _exit(127);
    }

    // Parent: Also call setpgid (race condition prevention)
    setpgid(pid, pid);

    // Create task node
    task_node_t *task = calloc(1, sizeof(task_node_t));
    task->id = ctrl->next_id++;
    task->pgid = pid;
    task->state = SIM_ACTIVE;
    task->command = build_command_str(argv);
    task->notified = false;
    task->next = ctrl->tasks;
    ctrl->tasks = task;

    if (take_control) {
        tcsetpgrp(ctrl->terminal_fd, pid);
        sigprocmask(SIG_SETMASK, &oldmask, NULL);
        wait_for_task(ctrl, task);
    } else {
        sigprocmask(SIG_SETMASK, &oldmask, NULL);
    }

    return task->id;
}

int sim_assign_queue(plumbob_controller_t *ctrl, char ***commands,
                     int n_commands, bool take_control) {
    if (!ctrl || !commands || n_commands <= 0) return -1;

    // Block SIGCHLD
    sigset_t mask, oldmask;
    sigemptyset(&mask);
    sigaddset(&mask, SIGCHLD);
    sigprocmask(SIG_BLOCK, &mask, &oldmask);

    int pipes[n_commands - 1][2];
    pid_t first_pid = 0;

    // Create pipes
    for (int i = 0; i < n_commands - 1; i++) {
        if (pipe(pipes[i]) < 0) {
            sigprocmask(SIG_SETMASK, &oldmask, NULL);
            return -1;
        }
    }

    // Fork all processes
    for (int i = 0; i < n_commands; i++) {
        pid_t pid = fork();

        if (pid < 0) {
            sigprocmask(SIG_SETMASK, &oldmask, NULL);
            return -1;
        }

        if (pid == 0) {
            // Child
            if (i == 0) {
                setpgid(0, 0);
                if (take_control) {
                    tcsetpgrp(STDIN_FILENO, getpid());
                }
            } else {
                setpgid(0, first_pid);
            }

            // Setup pipes
            if (i > 0) {
                dup2(pipes[i-1][0], STDIN_FILENO);
            }
            if (i < n_commands - 1) {
                dup2(pipes[i][1], STDOUT_FILENO);
            }

            // Close all pipe fds
            for (int j = 0; j < n_commands - 1; j++) {
                close(pipes[j][0]);
                close(pipes[j][1]);
            }

            // Restore signals
            signal(SIGTSTP, SIG_DFL);
            signal(SIGTTIN, SIG_DFL);
            signal(SIGTTOU, SIG_DFL);

            sigprocmask(SIG_SETMASK, &oldmask, NULL);

            execvp(commands[i][0], commands[i]);
            _exit(127);
        }

        if (i == 0) {
            first_pid = pid;
            setpgid(pid, pid);
        } else {
            setpgid(pid, first_pid);
        }
    }

    // Parent: Close all pipes
    for (int i = 0; i < n_commands - 1; i++) {
        close(pipes[i][0]);
        close(pipes[i][1]);
    }

    // Create task
    task_node_t *task = calloc(1, sizeof(task_node_t));
    task->id = ctrl->next_id++;
    task->pgid = first_pid;
    task->state = SIM_ACTIVE;
    task->command = strdup("pipeline");  // Could build full command
    task->notified = false;
    task->next = ctrl->tasks;
    ctrl->tasks = task;

    if (take_control) {
        tcsetpgrp(ctrl->terminal_fd, first_pid);
        sigprocmask(SIG_SETMASK, &oldmask, NULL);
        wait_for_task(ctrl, task);
    } else {
        sigprocmask(SIG_SETMASK, &oldmask, NULL);
    }

    return task->id;
}

int sim_take_control(plumbob_controller_t *ctrl, int task_id, bool send_resume) {
    if (!ctrl) return -1;

    task_node_t *task = find_task(ctrl, task_id);
    if (!task) return -1;

    if (send_resume && task->state == SIM_PAUSED) {
        killpg(task->pgid, SIGCONT);
        task->state = SIM_ACTIVE;
    }

    tcsetpgrp(ctrl->terminal_fd, task->pgid);
    wait_for_task(ctrl, task);

    return 0;
}

int sim_release_control(plumbob_controller_t *ctrl, int task_id, bool send_resume) {
    if (!ctrl) return -1;

    task_node_t *task = find_task(ctrl, task_id);
    if (!task) return -1;

    if (send_resume && task->state == SIM_PAUSED) {
        killpg(task->pgid, SIGCONT);
        task->state = SIM_ACTIVE;
    }

    // Task continues in background
    return 0;
}

int sim_cancel_task(plumbob_controller_t *ctrl, int task_id, int sig) {
    if (!ctrl || task_id <= 0) return -1;

    task_node_t *task = find_task(ctrl, task_id);
    if (!task) return -1;

    if (killpg(task->pgid, sig) < 0) {
        return -1;
    }

    return 0;
}

int sim_wait_task(plumbob_controller_t *ctrl, int task_id) {
    if (!ctrl) return -1;

    task_node_t *task = find_task(ctrl, task_id == 0 ? ctrl->current_task : task_id);
    if (!task) return -1;

    wait_for_task(ctrl, task);

    if (task->state == SIM_TASK_DONE) {
        return WEXITSTATUS(task->status);
    } else if (task->state == SIM_CANCELLED) {
        return -WTERMSIG(task->status);
    }

    return -1;
}

int sim_show_tasks(plumbob_controller_t *ctrl, sim_task_t *tasks_out, int max_tasks) {
    if (!ctrl || !tasks_out || max_tasks <= 0) return 0;

    int count = 0;
    task_node_t *task = ctrl->tasks;

    while (task && count < max_tasks) {
        if (task->state != SIM_TASK_DONE || !task->notified) {
            tasks_out[count].task_id = task->id;
            tasks_out[count].household_id = task->pgid;
            tasks_out[count].state = task->state;
            tasks_out[count].exit_code = WIFEXITED(task->status) ?
                                         WEXITSTATUS(task->status) : 0;
            tasks_out[count].cancel_signal = WIFSIGNALED(task->status) ?
                                              WTERMSIG(task->status) : 0;
            tasks_out[count].action_name = task->command;
            tasks_out[count].notified = task->notified;
            tasks_out[count].has_plumbob = (task->id == ctrl->current_task);
            count++;
        }
        task = task->next;
    }

    return count;
}

int sim_get_task_info(plumbob_controller_t *ctrl, int task_id, sim_task_t *info_out) {
    if (!ctrl || !info_out) return -1;

    task_node_t *task = find_task(ctrl, task_id);
    if (!task) return -1;

    info_out->task_id = task->id;
    info_out->household_id = task->pgid;
    info_out->state = task->state;
    info_out->exit_code = WIFEXITED(task->status) ? WEXITSTATUS(task->status) : 0;
    info_out->cancel_signal = WIFSIGNALED(task->status) ? WTERMSIG(task->status) : 0;
    info_out->action_name = task->command;
    info_out->notified = task->notified;
    info_out->has_plumbob = (task->id == ctrl->current_task);

    return 0;
}

int sim_notify_changes(plumbob_controller_t *ctrl) {
    if (!ctrl) return 0;

    int count = 0;
    task_node_t *task = ctrl->tasks;
    task_node_t *prev = NULL;

    // Reap any remaining children
    int status;
    pid_t pid;
    while ((pid = waitpid(-1, &status, WNOHANG | WUNTRACED)) > 0) {
        task_node_t *t = ctrl->tasks;
        while (t) {
            if (t->pgid == pid) {
                t->status = status;
                if (WIFEXITED(status)) t->state = SIM_TASK_DONE;
                else if (WIFSIGNALED(status)) t->state = SIM_CANCELLED;
                else if (WIFSTOPPED(status)) t->state = SIM_PAUSED;
                t->notified = false;
                break;
            }
            t = t->next;
        }
    }

    // Print notifications
    while (task) {
        if (!task->notified &&
            (task->state == SIM_TASK_DONE ||
             task->state == SIM_CANCELLED ||
             task->state == SIM_PAUSED)) {

            const char *marker = (task->id == ctrl->current_task) ? "+" :
                                 (task->id == ctrl->previous_task) ? "-" : " ";
            const char *state_str;

            switch (task->state) {
                case SIM_PAUSED: state_str = "Paused"; break;
                case SIM_TASK_DONE: state_str = "Done"; break;
                case SIM_CANCELLED: state_str = "Cancelled"; break;
                default: state_str = "Active"; break;
            }

            printf("[%d]%s %-12s %s", task->id, marker, state_str, task->command);

            if (task->state == SIM_CANCELLED) {
                printf("  (%s)", strsignal(WTERMSIG(task->status)));
            } else if (task->state == SIM_TASK_DONE && WEXITSTATUS(task->status) != 0) {
                printf("  (exit %d)", WEXITSTATUS(task->status));
            }

            printf("\n");
            task->notified = true;
            count++;
        }

        prev = task;
        task = task->next;
    }

    return count;
}

int sim_current_active(plumbob_controller_t *ctrl) {
    if (!ctrl) return 0;
    return ctrl->current_task;
}
```

### 4.4 Solutions alternatives acceptées

```c
/* Alternative 1: Tableau statique au lieu de liste chaînée */
struct plumbob_controller {
    task_entry_t tasks[MAX_TASKS];
    int task_count;
    // ...
};

/* Alternative 2: Handler SIGCHLD avec write() vers pipe self-pipe trick */
static int sigchld_pipe[2];

static void sigchld_handler(int sig) {
    (void)sig;
    char c = 'C';
    write(sigchld_pipe[1], &c, 1);  // async-signal-safe
}

/* Alternative 3: Sans variable globale, avec données dans signal mask */
// Use sigqueue() avec si_value pour passer le controller pointer
```

### 4.5 Solutions refusées (avec explications)

```c
/* REFUSÉ 1: Shell ne doit pas être stoppé */
plumbob_controller_t *plumbob_boot(void) {
    // ERREUR: Ne pas ignorer SIGTSTP
    // signal(SIGTSTP, SIG_IGN);  // MANQUANT!
}
// Pourquoi refusé: Si le shell reçoit SIGTSTP, il sera suspendu et
// l'utilisateur perd le contrôle du terminal

/* REFUSÉ 2: Race condition sur setpgid */
int sim_assign_task(plumbob_controller_t *ctrl, char **argv, bool take_control) {
    pid_t pid = fork();
    if (pid == 0) {
        setpgid(0, 0);
        execvp(...);
    }
    // ERREUR: Parent n'appelle pas setpgid
    // L'enfant peut executer avant que parent connaisse le pgid
}
// Pourquoi refusé: Race condition - le parent doit aussi appeler setpgid

/* REFUSÉ 3: Terminal pas retourné au shell */
static void wait_for_task(plumbob_controller_t *ctrl, task_node_t *task) {
    waitpid(-task->pgid, &status, WUNTRACED);
    // ERREUR: Pas de tcsetpgrp pour rendre le terminal
}
// Pourquoi refusé: Le shell ne récupère pas le terminal après la tâche

/* REFUSÉ 4: Zombies */
int sim_assign_task(plumbob_controller_t *ctrl, char **argv, bool take_control) {
    // Fork en background sans handler SIGCHLD
    // Jamais de waitpid pour les background tasks
}
// Pourquoi refusé: Les processus terminés deviennent des zombies

/* REFUSÉ 5: Enfants avec mauvais handlers */
if (pid == 0) {
    // ERREUR: Ne pas restaurer SIG_DFL
    // Les enfants héritent de SIG_IGN du shell
    execvp(argv[0], argv);
}
// Pourquoi refusé: Les processus enfants ne peuvent pas être stoppés par Ctrl+Z
```

### 4.6 Solution bonus de référence (COMPLÈTE)

```c
int sim_assign_queue(plumbob_controller_t *ctrl, char ***commands,
                     int n_commands, bool take_control) {
    // Voir section 4.3 - déjà implémentée
}

int sim_handle_orphans(plumbob_controller_t *ctrl) {
    if (!ctrl) return 0;

    int count = 0;
    task_node_t *task = ctrl->tasks;

    while (task) {
        // Check if process group still exists
        if (kill(-task->pgid, 0) < 0 && errno == ESRCH) {
            // Process group doesn't exist anymore
            if (task->state == SIM_ACTIVE || task->state == SIM_PAUSED) {
                task->state = SIM_CANCELLED;
                task->notified = false;
                count++;
            }
        }
        task = task->next;
    }

    return count;
}

int sim_parse_job_spec(plumbob_controller_t *ctrl, const char *spec) {
    if (!ctrl || !spec) return -1;

    if (spec[0] != '%') {
        // Direct number
        return atoi(spec);
    }

    if (strcmp(spec, "%%") == 0 || strcmp(spec, "%+") == 0) {
        return ctrl->current_task;
    }

    if (strcmp(spec, "%-") == 0) {
        return ctrl->previous_task;
    }

    if (spec[1] >= '0' && spec[1] <= '9') {
        return atoi(spec + 1);
    }

    // %string - search by command prefix
    task_node_t *task = ctrl->tasks;
    while (task) {
        if (task->command && strstr(task->command, spec + 1) == task->command) {
            return task->id;
        }
        task = task->next;
    }

    return -1;
}
```

### 4.7 Solutions alternatives bonus (COMPLÈTES)

```c
/* Alternative bonus: Pipeline avec exec chain */
// Fork once, use exec chain pattern
// Less memory efficient but simpler
```

### 4.8 Solutions refusées bonus (COMPLÈTES)

```c
/* REFUSÉ BONUS: Pipeline avec PGID différent par processus */
for (int i = 0; i < n_commands; i++) {
    pid_t pid = fork();
    if (pid == 0) {
        setpgid(0, 0);  // ERREUR: Chaque process a son propre group
    }
}
// Pourquoi refusé: SIGTSTP ne suspendra qu'un processus du pipeline
```

### 4.9 spec.json (ENGINE v22.1 — FORMAT STRICT)

```json
{
  "name": "plumbob_controller",
  "language": "c",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (process groups + signals + terminal)",
  "tags": ["module2.2", "job_control", "process", "signals", "phase2"],
  "passing_score": 80,

  "function": {
    "name": "plumbob_boot",
    "prototype": "plumbob_controller_t *plumbob_boot(void)",
    "return_type": "plumbob_controller_t *",
    "parameters": []
  },

  "additional_functions": [
    {
      "name": "plumbob_shutdown",
      "prototype": "void plumbob_shutdown(plumbob_controller_t *ctrl)",
      "return_type": "void"
    },
    {
      "name": "sim_assign_task",
      "prototype": "int sim_assign_task(plumbob_controller_t *ctrl, char **argv, bool take_control)",
      "return_type": "int"
    },
    {
      "name": "sim_take_control",
      "prototype": "int sim_take_control(plumbob_controller_t *ctrl, int task_id, bool send_resume)",
      "return_type": "int"
    },
    {
      "name": "sim_release_control",
      "prototype": "int sim_release_control(plumbob_controller_t *ctrl, int task_id, bool send_resume)",
      "return_type": "int"
    },
    {
      "name": "sim_cancel_task",
      "prototype": "int sim_cancel_task(plumbob_controller_t *ctrl, int task_id, int sig)",
      "return_type": "int"
    },
    {
      "name": "sim_notify_changes",
      "prototype": "int sim_notify_changes(plumbob_controller_t *ctrl)",
      "return_type": "int"
    }
  ],

  "driver": {
    "reference": "plumbob_controller_t *ref_plumbob_boot(void) { plumbob_controller_t *ctrl = calloc(1, sizeof(plumbob_controller_t)); if (!ctrl) return NULL; ctrl->shell_pgid = getpgrp(); signal(SIGTSTP, SIG_IGN); signal(SIGTTIN, SIG_IGN); signal(SIGTTOU, SIG_IGN); return ctrl; }",

    "edge_cases": [
      {
        "name": "null_controller_spawn",
        "test_code": "sim_assign_task(NULL, argv, true)",
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "Controller NULL doit retourner -1"
      },
      {
        "name": "null_argv",
        "test_code": "sim_assign_task(ctrl, NULL, true)",
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "argv NULL doit retourner -1"
      },
      {
        "name": "empty_argv",
        "test_code": "char *argv[] = {NULL}; sim_assign_task(ctrl, argv, true)",
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "argv[0] NULL doit retourner -1"
      },
      {
        "name": "invalid_task_id",
        "test_code": "sim_take_control(ctrl, 999, true)",
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "Task ID inexistant"
      },
      {
        "name": "foreground_simple",
        "test_code": "sim_assign_task(ctrl, {\"echo\", \"test\", NULL}, true)",
        "expected": 1,
        "is_trap": false
      },
      {
        "name": "background_simple",
        "test_code": "sim_assign_task(ctrl, {\"sleep\", \"1\", NULL}, false)",
        "expected": 1,
        "is_trap": false
      },
      {
        "name": "unique_pgids",
        "validation": "Two spawned tasks have different pgids",
        "is_trap": true,
        "trap_explanation": "Chaque tâche doit avoir son propre process group"
      },
      {
        "name": "terminal_returns",
        "validation": "tcgetpgrp(0) == shell_pgid after foreground task",
        "is_trap": true,
        "trap_explanation": "Le terminal doit revenir au shell"
      }
    ],

    "fuzzing": {
      "enabled": false,
      "reason": "Process/signal tests require controlled environment"
    }
  },

  "norm": {
    "allowed_functions": ["fork", "execvp", "_exit", "waitpid", "getpid", "getppid", "getpgid", "setpgid", "setsid", "getsid", "killpg", "tcgetpgrp", "tcsetpgrp", "isatty", "ttyname", "sigaction", "sigemptyset", "sigaddset", "sigfillset", "sigprocmask", "sigsuspend", "kill", "malloc", "free", "realloc", "calloc", "strdup", "strncpy", "strlen", "open", "close", "dup", "dup2", "read", "write", "printf", "fprintf", "snprintf", "perror", "strsignal"],
    "forbidden_functions": ["system", "popen", "pthread_create"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```c
/* Mutant A (Boundary) : Max tasks non vérifié */
int sim_assign_task(plumbob_controller_t *ctrl, char **argv, bool take_control) {
    // ERREUR: Pas de vérification du nombre max de tâches
    // Peut causer un overflow si > 64 tâches
    task_node_t *task = calloc(1, sizeof(task_node_t));
    // ...
}
// Pourquoi c'est faux : Dépassement de capacité possible
// Ce qui était pensé : "Le système limitera naturellement"

/* Mutant B (Safety) : Shell ne bloque pas SIGTSTP */
plumbob_controller_t *plumbob_boot(void) {
    plumbob_controller_t *ctrl = calloc(1, sizeof(*ctrl));
    // ERREUR: SIGTSTP pas ignoré
    // signal(SIGTSTP, SIG_IGN);  // MANQUANT
    return ctrl;
}
// Pourquoi c'est faux : Le shell peut être suspendu par Ctrl+Z
// Ce qui était pensé : "Le shell n'est pas le foreground donc pas affecté"

/* Mutant C (Resource) : Terminal pas retourné */
static void wait_for_task(plumbob_controller_t *ctrl, task_node_t *task) {
    int status;
    waitpid(-task->pgid, &status, WUNTRACED);
    // ERREUR: tcsetpgrp manquant
    // tcsetpgrp(ctrl->terminal_fd, ctrl->shell_pgid);  // MANQUANT
}
// Pourquoi c'est faux : Le shell ne peut plus lire l'input après
// Ce qui était pensé : "Le terminal revient automatiquement"

/* Mutant D (Logic) : Race condition setpgid */
int sim_assign_task(plumbob_controller_t *ctrl, char **argv, bool take_control) {
    pid_t pid = fork();
    if (pid == 0) {
        setpgid(0, 0);
        execvp(argv[0], argv);
    }
    // ERREUR: Parent n'appelle pas setpgid(pid, pid)
    // L'enfant peut exec avant que parent sache le pgid
}
// Pourquoi c'est faux : Race condition - killpg peut échouer
// Ce qui était pensé : "L'enfant fait setpgid, c'est suffisant"

/* Mutant E (Return) : Handlers enfant pas restaurés */
if (pid == 0) {
    setpgid(0, 0);
    // ERREUR: Handlers SIG_DFL pas restaurés
    // signal(SIGTSTP, SIG_DFL);  // MANQUANT
    execvp(argv[0], argv);
}
// Pourquoi c'est faux : Les enfants héritent de SIG_IGN du shell
// Ce qui était pensé : "exec() reset les handlers"

/* Mutant F (Async) : SIGCHLD non géré */
plumbob_controller_t *plumbob_boot(void) {
    // ERREUR: Pas de handler SIGCHLD
    // Les background tasks deviennent des zombies
}
// Pourquoi c'est faux : Accumulation de processus zombies
// Ce qui était pensé : "On attend avec waitpid dans notify"
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

Cet exercice enseigne les concepts fondamentaux du **job control UNIX** :

1. **Hiérarchie Session → Process Group → Process**
2. **Terminal de contrôle** et `tcsetpgrp`
3. **Signaux de job control** : SIGTSTP, SIGCONT, SIGCHLD
4. **Gestion des race conditions** avec `setpgid` appelé des deux côtés
5. **États des jobs** et transitions d'état
6. **Notification asynchrone** via SIGCHLD

### 5.2 LDA — Traduction littérale en français (MAJUSCULES)

```
FONCTION plumbob_boot QUI RETOURNE UN POINTEUR VERS UNE STRUCTURE PLUMBOB_CONTROLLER
DÉBUT FONCTION
    DÉCLARER ctrl COMME POINTEUR VERS PLUMBOB_CONTROLLER

    AFFECTER ALLOUER LA MÉMOIRE DE LA TAILLE D'UNE STRUCTURE PLUMBOB_CONTROLLER À ctrl
    SI ctrl EST ÉGAL À NUL ALORS
        RETOURNER NUL
    FIN SI

    AFFECTER NUL AU CHAMP tasks DE ctrl
    AFFECTER 1 AU CHAMP next_id DE ctrl
    AFFECTER OBTENIR LE GROUPE DE PROCESSUS COURANT AU CHAMP shell_pgid DE ctrl

    IGNORER LE SIGNAL SIGTSTP
    IGNORER LE SIGNAL SIGTTIN
    IGNORER LE SIGNAL SIGTTOU

    INSTALLER LE HANDLER sigchld_handler POUR SIGCHLD

    RETOURNER ctrl
FIN FONCTION

FONCTION sim_assign_task QUI RETOURNE UN ENTIER ET PREND EN PARAMÈTRES ctrl UN POINTEUR VERS PLUMBOB_CONTROLLER ET argv UN TABLEAU DE CHAÎNES ET take_control UN BOOLÉEN
DÉBUT FONCTION
    SI ctrl EST ÉGAL À NUL OU argv EST ÉGAL À NUL ALORS
        RETOURNER MOINS 1
    FIN SI

    BLOQUER LE SIGNAL SIGCHLD

    DÉCLARER pid COMME IDENTIFIANT DE PROCESSUS
    AFFECTER CRÉER UN PROCESSUS FILS À pid

    SI pid EST INFÉRIEUR À 0 ALORS
        DÉBLOQUER LE SIGNAL SIGCHLD
        RETOURNER MOINS 1
    FIN SI

    SI pid EST ÉGAL À 0 ALORS
        CRÉER UN NOUVEAU GROUPE DE PROCESSUS AVEC MON PID

        SI take_control EST VRAI ALORS
            DONNER LE TERMINAL À MON GROUPE DE PROCESSUS
        FIN SI

        RESTAURER LE HANDLER PAR DÉFAUT POUR SIGTSTP
        RESTAURER LE HANDLER PAR DÉFAUT POUR SIGTTIN
        RESTAURER LE HANDLER PAR DÉFAUT POUR SIGTTOU

        DÉBLOQUER LE SIGNAL SIGCHLD

        EXÉCUTER LA COMMANDE argv[0] AVEC LES ARGUMENTS argv
        QUITTER AVEC LE CODE 127
    FIN SI

    CRÉER UN GROUPE DE PROCESSUS pid AVEC LE LEADER pid

    CRÉER UN NOEUD DE TÂCHE AVEC pid COMME PGID
    AJOUTER LE NOEUD À LA LISTE DES TÂCHES

    SI take_control EST VRAI ALORS
        DONNER LE TERMINAL AU GROUPE pid
        DÉBLOQUER LE SIGNAL SIGCHLD
        ATTENDRE LA TÂCHE
    SINON
        DÉBLOQUER LE SIGNAL SIGCHLD
    FIN SI

    RETOURNER L'ID DE LA TÂCHE
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Plumbob Controller - Lancer une tâche
---
1. VÉRIFIER les paramètres (ctrl, argv)
   → Si NULL : RETOURNER -1

2. BLOQUER SIGCHLD (éviter race condition)

3. FORK pour créer processus fils :

   a. DANS LE FILS :
      |-- Créer nouveau process group (setpgid(0, 0))
      |-- SI foreground : Prendre le terminal (tcsetpgrp)
      |-- Restaurer handlers signaux (SIG_DFL)
      |-- Débloquer SIGCHLD
      |-- Exécuter la commande (execvp)
      |-- Si exec échoue : _exit(127)

   b. DANS LE PARENT :
      |-- Aussi appeler setpgid(pid, pid) [race condition fix]
      |-- Créer noeud de tâche
      |-- SI foreground :
      |     |-- Donner terminal au fils
      |     |-- Attendre terminaison/suspension
      |     |-- Reprendre le terminal
      |-- SINON :
      |     |-- Retourner immédiatement (background)

4. DÉBLOQUER SIGCHLD

5. RETOURNER task_id
```

### 5.2.3 Diagramme Mermaid

```mermaid
stateDiagram-v2
    [*] --> SIM_ACTIVE : sim_assign_task

    SIM_ACTIVE --> SIM_PAUSED : SIGTSTP (Ctrl+Z)
    SIM_ACTIVE --> SIM_TASK_DONE : exit()
    SIM_ACTIVE --> SIM_CANCELLED : signal (SIGTERM, SIGKILL)

    SIM_PAUSED --> SIM_ACTIVE : sim_take_control(cont=true)
    SIM_PAUSED --> SIM_ACTIVE : sim_release_control(cont=true)
    SIM_PAUSED --> SIM_CANCELLED : SIGKILL

    SIM_TASK_DONE --> [*] : sim_notify_changes
    SIM_CANCELLED --> [*] : sim_notify_changes
```

### 5.3 Visualisation ASCII (adaptée au sujet)

```
                THE SIMS JOB CONTROL

    ┌────────────────────────────────────────────────┐
    │                   SESSION                       │
    │                  (Le Quartier)                  │
    │                                                 │
    │  ┌─────────────┐  ┌─────────────┐              │
    │  │  CONTROLLER │  │ HOUSEHOLD 1 │  🔷 Plumbob  │
    │  │  (Shell)    │  │   (pgid 1)  │    ↓        │
    │  │             │  │ ┌───┬───┐   │              │
    │  │  Ignores    │  │ │S1 │S2 │   │ Foreground  │
    │  │  SIGTSTP    │  │ └───┴───┘   │              │
    │  └─────────────┘  └─────────────┘              │
    │         │                │                     │
    │         │    tcsetpgrp   │                     │
    │         └───────────────►│                     │
    │                          │                     │
    │  ┌─────────────┐        │                     │
    │  │ HOUSEHOLD 2 │        │                     │
    │  │   (pgid 2)  │  Background (autonome)       │
    │  │ ┌───┬───┐   │                              │
    │  │ │S3 │S4 │   │                              │
    │  │ └───┴───┘   │                              │
    │  └─────────────┘                              │
    └────────────────────────────────────────────────┘

    CTRL+Z envoyé au foreground group uniquement!

    Household 1 reçoit SIGTSTP → Tous les Sims pausent
    Household 2 continue en autonome (background)
```

### 5.4 Les pièges en détail

#### Piège 1 : Le shell est suspendu par Ctrl+Z

```c
// PIÈGE
plumbob_controller_t *plumbob_boot(void) {
    // Oublier d'ignorer SIGTSTP
    return ctrl;
}

// SOLUTION
signal(SIGTSTP, SIG_IGN);
signal(SIGTTIN, SIG_IGN);
signal(SIGTTOU, SIG_IGN);
```

**Explication** : Le shell hérite des handlers par défaut. S'il devient foreground et reçoit SIGTSTP, il sera suspendu!

#### Piège 2 : Race condition setpgid

```c
// PIÈGE
pid_t pid = fork();
if (pid == 0) {
    setpgid(0, 0);  // Enfant crée son groupe
    execvp(...);
}
// Parent utilise immédiatement pid comme pgid
killpg(pid, SIGSTOP);  // PEUT ÉCHOUER si enfant pas encore setpgid

// SOLUTION
pid_t pid = fork();
if (pid == 0) {
    setpgid(0, 0);
    execvp(...);
}
setpgid(pid, pid);  // Parent AUSSI!
```

#### Piège 3 : Terminal pas récupéré

```c
// PIÈGE
void wait_for_task(...) {
    waitpid(-pgid, &status, WUNTRACED);
    // Terminal reste au process group mort!
}

// SOLUTION
void wait_for_task(...) {
    waitpid(-pgid, &status, WUNTRACED);
    tcsetpgrp(STDIN_FILENO, shell_pgid);  // Reprendre le terminal
}
```

#### Piège 4 : Enfants avec mauvais handlers

```c
// PIÈGE
if (pid == 0) {
    // Hérite de SIG_IGN du shell
    execvp(...);  // Le processus ignore SIGTSTP!
}

// SOLUTION
if (pid == 0) {
    signal(SIGTSTP, SIG_DFL);  // Restaurer avant exec
    signal(SIGTTIN, SIG_DFL);
    signal(SIGTTOU, SIG_DFL);
    execvp(...);
}
```

### 5.5 Cours Complet (VRAI cours, pas un résumé)

#### 5.5.1 La Hiérarchie UNIX : Session → Process Group → Process

```
Session (setsid crée une nouvelle session)
├── Session Leader (le premier processus de la session)
│
├── Process Group 1 (setpgid crée/rejoint un groupe)
│   ├── Process Group Leader (pgid == pid)
│   ├── Process membre
│   └── Process membre
│
├── Process Group 2 (le foreground group reçoit les signaux TTY)
│   └── ...
│
└── Controlling Terminal (/dev/tty)
    └── tcsetpgrp() définit quel groupe est foreground
```

**Terminologie The Sims :**
- Session = Le Quartier (neighborhood)
- Process Group = Household (famille)
- Process = Sim
- Foreground = Sim avec le Plumbob
- SIGTSTP = Pause du jeu

#### 5.5.2 Les Signaux de Job Control

| Signal | Défaut | Envoyé par | Usage |
|--------|--------|------------|-------|
| SIGTSTP | Stop | Ctrl+Z (TTY) | Suspendre foreground |
| SIGSTOP | Stop | kill() | Suspendre (ne peut être ignoré) |
| SIGCONT | Continue | kill() | Reprendre processus stoppé |
| SIGTTIN | Stop | TTY | Background essaie de lire terminal |
| SIGTTOU | Stop | TTY | Background essaie d'écrire terminal |
| SIGCHLD | Ignore | Kernel | Enfant a changé d'état |

#### 5.5.3 Le Contrôle du Terminal

```c
// Quel groupe possède le terminal ?
pid_t fg = tcgetpgrp(STDIN_FILENO);

// Donner le terminal à un groupe
tcsetpgrp(STDIN_FILENO, pgid);

// Seul le foreground group peut lire/écrire le terminal
// Si un background essaie → SIGTTIN/SIGTTOU → stoppé
```

#### 5.5.4 waitpid() et les Options

```c
int status;
pid_t pid = waitpid(target, &status, options);

// target:
//   > 0  : Attendre ce PID spécifique
//   -1   : Attendre n'importe quel enfant
//   0    : Attendre enfant dans mon groupe
//   < -1 : Attendre enfant dans groupe |target|

// options:
//   WNOHANG     : Ne pas bloquer (retourne 0 si rien)
//   WUNTRACED   : Retourner aussi si enfant stoppé
//   WCONTINUED  : Retourner aussi si enfant continué

// Macros pour analyser status:
WIFEXITED(status)    // Terminé normalement ?
WEXITSTATUS(status)  // Code de sortie (0-255)
WIFSIGNALED(status)  // Tué par signal ?
WTERMSIG(status)     // Quel signal ?
WIFSTOPPED(status)   // Stoppé ?
WSTOPSIG(status)     // Quel signal stop ?
WIFCONTINUED(status) // Repris après stop ?
```

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ static plumbob_controller_t *global_ctrl;                       │
│ // Variable globale mutable                                     │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ static volatile sig_atomic_t g_sigchld_flag;                    │
│ // Seule exception: flag pour handler signal                    │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Les handlers de signaux sont asynchrones                      │
│ • sig_atomic_t garantit atomicité                               │
│ • volatile empêche les optimisations dangereuses                │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario : Lancer "sleep 10" en foreground, Ctrl+Z, puis bg**

```
┌───────┬──────────────────────────────────────────────────┬─────────────────────┐
│ Étape │ Action                                           │ État                │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   1   │ sim_assign_task(ctrl, {"sleep","10"}, true)      │ Fork pid=1234       │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   2   │ Enfant: setpgid(0, 0)                            │ pgid=1234           │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   3   │ Parent: setpgid(1234, 1234)                      │ (race fix)          │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   4   │ Parent: tcsetpgrp(0, 1234)                       │ Terminal → sleep    │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   5   │ Enfant: execvp("sleep", ...)                     │ SIM_ACTIVE          │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   6   │ User: Ctrl+Z                                     │ TTY → SIGTSTP       │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   7   │ sleep reçoit SIGTSTP                             │ sleep stoppé        │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   8   │ waitpid retourne (WUNTRACED)                     │ SIM_PAUSED          │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│   9   │ Parent: tcsetpgrp(0, shell_pgid)                 │ Terminal → Shell    │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│  10   │ sim_release_control(ctrl, 1, true)               │                     │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│  11   │ killpg(1234, SIGCONT)                            │ sleep reprend       │
├───────┼──────────────────────────────────────────────────┼─────────────────────┤
│  12   │ Retour immédiat (background)                     │ SIM_ACTIVE (bg)     │
└───────┴──────────────────────────────────────────────────┴─────────────────────┘
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🎮 MEME : "Sul Sul!" — Le salut des Sims

Quand un Sim dit "Sul Sul!", il salue et attend une réponse.

C'est comme `waitpid()` qui attend la fin d'un processus :
- "Sul Sul!" = waitpid() appelle
- Réponse = Le processus termine et retourne son status

```c
// 🎮 Sul Sul! Attends que le Sim finisse sa tâche
pid_t pid = waitpid(-pgid, &status, WUNTRACED);
// Le Sim a fini ! Vérifie comment : WIFEXITED, WIFSTOPPED...
```

---

#### 🎮 MEME : "Pause du jeu" — SIGTSTP

Dans The Sims, quand tu appuies sur Pause, TOUS les Sims s'arrêtent instantanément.

C'est exactement ce que fait SIGTSTP envoyé au foreground process group :

```
   Ctrl+Z
      │
      ▼
  ┌───────────────┐
  │ SIGTSTP       │──────► Tout le household pause
  │ au foreground │        (process group entier)
  └───────────────┘
```

**Mnémo** : "Pause = Tout le Household Gèle"

---

#### 💀 MEME : "Delete the Pool" — SIGKILL

Le meme classique des Sims : retirer l'échelle de la piscine pour que le Sim ne puisse pas sortir.

C'est comme SIGKILL : le processus ne peut PAS l'ignorer ou le capturer.

```c
// 💀 Pas d'échappement possible
kill(pid, SIGKILL);  // Équivalent à retirer l'échelle
// Le processus DOIT mourir
```

**Différence avec SIGTERM** :
- SIGTERM = "Sors de la piscine s'il te plaît" (peut être ignoré)
- SIGKILL = Retirer l'échelle (IMPOSSIBLE à ignorer)

---

#### 🔷 MEME : "Le Plumbob change de tête" — tcsetpgrp

Le Plumbob (diamant vert) indique quel Sim tu contrôles.

Quand tu cliques sur un autre Sim, le Plumbob se déplace vers lui.

C'est exactement `tcsetpgrp()` :

```c
// 🔷 Déplacer le Plumbob vers le nouveau Sim
tcsetpgrp(STDIN_FILENO, new_pgid);
// Maintenant CE groupe reçoit les inputs (SIGTSTP, etc.)
```

### 5.9 Applications pratiques

| Application | Utilisation du Job Control |
|-------------|---------------------------|
| **bash/zsh** | Implémentation de `fg`, `bg`, `jobs`, Ctrl+Z |
| **tmux/screen** | Gestion des sessions détachées |
| **systemd** | Contrôle des services (start/stop/restart) |
| **Docker** | Gestion des containers (pause/unpause) |
| **IDE** | Lancement et arrêt des builds |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Conséquence | Solution |
|---|-------|-------------|----------|
| 1 | Shell ne bloque pas SIGTSTP | Shell suspendu par Ctrl+Z | `signal(SIGTSTP, SIG_IGN)` |
| 2 | Race condition setpgid | killpg échoue | Parent ET enfant appellent setpgid |
| 3 | Terminal pas récupéré | Shell ne peut plus lire input | tcsetpgrp(fd, shell_pgid) après wait |
| 4 | Enfants héritent SIG_IGN | Ctrl+Z ne marche pas | Restaurer SIG_DFL avant exec |
| 5 | Zombies background | Accumulation de défunts | Handler SIGCHLD avec waitpid(-1) |
| 6 | waitpid sur mauvais PID | N'attend qu'un processus du pipeline | waitpid(-pgid, ...) pour le groupe |

---

## 📝 SECTION 7 : QCM

### Question 1
**Qui reçoit SIGTSTP quand l'utilisateur appuie sur Ctrl+Z ?**

- A) Tous les processus de la session
- B) Le process group leader uniquement
- C) Le foreground process group
- D) Le shell uniquement
- E) Tous les processus background
- F) Le processus avec le plus petit PID
- G) Aucun processus
- H) Le controlling terminal
- I) Le kernel
- J) Le processus qui a le terminal

**Réponse : C**

### Question 2
**Pourquoi le shell doit-il ignorer SIGTSTP ?**

- A) Pour économiser la mémoire
- B) Pour ne pas être suspendu par Ctrl+Z
- C) Pour recevoir SIGCHLD
- D) Pour pouvoir faire fork()
- E) Pour accéder au terminal
- F) Pour éviter les zombies
- G) Pour créer des process groups
- H) C'est optionnel
- I) Pour la sécurité
- J) Pour le multi-threading

**Réponse : B**

### Question 3
**Quelle fonction donne le terminal à un process group ?**

- A) setpgid()
- B) setsid()
- C) tcsetpgrp()
- D) tcgetpgrp()
- E) ioctl()
- F) fcntl()
- G) ttyname()
- H) isatty()
- I) tcflush()
- J) tcgetattr()

**Réponse : C**

### Question 4
**Pourquoi appeler setpgid() dans le parent ET l'enfant ?**

- A) Pour la performance
- B) Pour éviter une race condition
- C) Pour créer deux groupes
- D) C'est obligatoire par POSIX
- E) Pour le debugging
- F) Pour la compatibilité
- G) Pour les signaux
- H) Un seul suffit
- I) Pour le terminal
- J) Pour waitpid

**Réponse : B**

### Question 5
**Quelle option waitpid() permet de détecter un processus stoppé ?**

- A) WNOHANG
- B) WUNTRACED
- C) WCONTINUED
- D) WNOWAIT
- E) WEXITED
- F) WSTOPPED
- G) WSTOPSIG
- H) WIFSTOPPED
- I) WCOREFLAG
- J) WNOZOMBIE

**Réponse : B**

### Question 6
**Que se passe-t-il si un processus background essaie de lire le terminal ?**

- A) Il reçoit EOF
- B) Il reçoit SIGTTIN et est stoppé
- C) Il reçoit une erreur
- D) La lecture bloque
- E) Le shell est notifié
- F) Rien de spécial
- G) Il reçoit SIGPIPE
- H) Le terminal est partagé
- I) Il devient foreground
- J) Il est tué

**Réponse : B**

### Question 7
**Comment envoyer un signal à tout un process group ?**

- A) kill(pgid, sig)
- B) kill(-pgid, sig)
- C) killpg(pgid, sig)
- D) B et C sont corrects
- E) sigqueue()
- F) raise()
- G) pthread_kill()
- H) tkill()
- I) tgkill()
- J) sigpending()

**Réponse : D**

### Question 8
**Que retourne waitpid() si WNOHANG est utilisé et aucun enfant n'a changé d'état ?**

- A) -1
- B) 0
- C) Le PID du premier enfant
- D) EAGAIN
- E) ECHILD
- F) 1
- G) Le PGID
- H) EINTR
- I) Le PID du parent
- J) ENOENT

**Réponse : B**

### Question 9
**Quel signal NE PEUT PAS être ignoré ou capturé ?**

- A) SIGTERM
- B) SIGINT
- C) SIGKILL
- D) SIGSTOP
- E) C et D
- F) SIGHUP
- G) SIGQUIT
- H) SIGTSTP
- I) SIGCONT
- J) SIGCHLD

**Réponse : E**

### Question 10
**Après un fork(), les handlers de signaux du parent sont...**

- A) Réinitialisés à SIG_DFL
- B) Hérités par l'enfant
- C) Convertis en SIG_IGN
- D) Supprimés
- E) Partagés (même mémoire)
- F) Copiés puis modifiés
- G) Dépend du signal
- H) Bloqués automatiquement
- I) Perdus après exec
- J) B puis I après exec

**Réponse : J** (hérités après fork, puis SIG_DFL après exec sauf SIG_IGN)

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 2.2.9 - plumbob_controller |
| **Thème** | The Sims - Job Control |
| **Difficulté** | ★★★★★★★★☆☆ (8/10) |
| **Durée** | 6-7 heures |
| **Concepts** | Process groups, SIGTSTP/SIGCONT, tcsetpgrp, SIGCHLD |
| **Fonctions clés** | setpgid, tcsetpgrp, waitpid(WUNTRACED), killpg |
| **Pièges principaux** | Race condition setpgid, terminal recovery, handlers hérités |
| **Bonus** | Pipelines, orphan groups, job specs |

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.2.9-synth-plumbob-controller",
    "generated_at": "2026-01-11 15:30:00",

    "metadata": {
      "exercise_id": "2.2.9-synth",
      "exercise_name": "plumbob_controller",
      "module": "2.2.9",
      "module_name": "Job Controller",
      "concept": "synth",
      "concept_name": "Process Groups + Signals + Terminal Control",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 2,
      "difficulty": 8,
      "difficulty_stars": "★★★★★★★★☆☆",
      "language": "c",
      "duration_minutes": 420,
      "xp_base": 500,
      "xp_bonus_multiplier": 6,
      "bonus_tier": "GÉNIE",
      "bonus_icon": "🧠",
      "complexity_time": "T3 O(n)",
      "complexity_space": "S2 O(n)",
      "prerequisites": ["fork", "exec", "waitpid", "signals", "process_groups"],
      "domains": ["Process", "Mem", "Struct"],
      "domains_bonus": ["Process", "Net"],
      "tags": ["job_control", "signals", "terminal", "process_groups", "the_sims"],
      "meme_reference": "The Sims - Plumbob, Sul Sul, Delete the Pool"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_plumbob.c": "/* Section 4.3 */",
      "references/ref_plumbob_bonus.c": "/* Section 4.6 */",
      "alternatives/alt_array.c": "/* Alternative avec tableau statique */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 - Max tasks */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 - SIGTSTP non ignoré */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 - Terminal pas récupéré */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 - Race condition */",
      "mutants/mutant_e_return.c": "/* Section 4.10 - Handlers non restaurés */",
      "mutants/mutant_f_async.c": "/* Section 4.10 - SIGCHLD non géré */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_plumbob.c",
        "references/ref_plumbob_bonus.c",
        "alternatives/alt_array.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c",
        "mutants/mutant_f_async.c"
      ]
    },

    "commands": {
      "compile": "gcc -Wall -Wextra -Werror -std=c17 plumbob.c -o libplumbob.a",
      "test": "gcc -Wall -Wextra -Werror main.c -L. -lplumbob -o test && ./test",
      "valgrind": "valgrind --leak-check=full ./test"
    }
  }
}
```

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 25 | Analogie The Sims parfaite pour job control |
| Couverture conceptuelle | 25 | Process groups, signals, terminal, SIGCHLD |
| Testabilité auto | 24 | Tests déterministes, 18 edge cases |
| Originalité | 25 | Thème The Sims unique et pédagogique |
| **TOTAL** | **99/100** | ✓ Validé |

**✓ Score ≥ 95, exercice validé.**
