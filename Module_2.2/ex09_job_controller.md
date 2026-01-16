# [Module 2.2] - Exercise 09: Job Controller

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex09"
difficulty: difficile
estimated_time: "6-7 heures"
prerequisite_exercises: ["ex03", "ex04", "ex05"]
concepts_requis:
  - "fork(), exec(), waitpid()"
  - "Signaux (SIGTSTP, SIGCONT, SIGCHLD)"
  - "Groupes de processus (setpgid, getpgid)"
  - "Terminal control (tcsetpgrp)"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.2.8.a | Process groups | setpgid, getpgid, killpg |
| 2.2.8.b | Sessions | setsid, getsid |
| 2.2.8.c | Controlling terminal | /dev/tty, tcgetpgrp, tcsetpgrp |
| 2.2.8.d | Foreground process group | Groupe recevant les signaux TTY |
| 2.2.9.a | Job states | Running, Stopped, Terminated |
| 2.2.9.b | fg command | Mettre un job en foreground |
| 2.2.9.c | bg command | Continuer un job en background |
| 2.2.9.d | Job notification | Detection des changements d'etat |
| 2.2.9.e | SIGTSTP/SIGCONT | Suspension et reprise |
| 2.2.9.f | Orphaned process groups | Gestion des groupes orphelins |

### Objectifs Pedagogiques

A la fin de cet exercice, vous devriez etre capable de:
1. Comprendre la hierarchie session/process group/process
2. Implementer le controle foreground/background comme dans bash
3. Gerer correctement SIGTSTP, SIGCONT, et SIGCHLD
4. Manipuler le terminal de controle avec tcsetpgrp
5. Notifier l'utilisateur des changements d'etat des jobs

---

## Contexte

### Comment fonctionne `Ctrl+Z` dans un shell?

Quand vous tapez `Ctrl+Z` dans un terminal:

1. Le driver TTY envoie `SIGTSTP` au **foreground process group**
2. Tous les processus de ce groupe sont suspendus
3. Le shell reprend le controle du terminal
4. Le shell affiche `[1]+ Stopped sleep 100`

Puis quand vous tapez `fg %1`:

1. Le shell envoie `SIGCONT` au groupe
2. Le shell donne le terminal au groupe (tcsetpgrp)
3. Le shell attend que le job termine (waitpid)

### Pourquoi les Process Groups?

Les groupes de processus existent pour gerer les **pipelines**:

```bash
cat file | grep pattern | sort | uniq
```

Ces 4 processus forment UN job, UN process group. Quand vous faites Ctrl+C, les 4 recoivent SIGINT, pas seulement un.

### Ce que Vous Allez Construire

Un gestionnaire de jobs complet permettant:
- Lancer des commandes en foreground ou background
- Suspendre un job avec Ctrl+Z (SIGTSTP)
- Reprendre un job en foreground (fg) ou background (bg)
- Lister les jobs actifs avec leur etat
- Etre notifie quand un job change d'etat

### Questions de Conception

Avant de coder, reflechissez:
1. Qui doit ignorer SIGTSTP? Qui doit le recevoir?
2. Que se passe-t-il si vous faites `fg` sur un job deja foreground?
3. Comment detecter qu'un job background s'est termine?
4. Que faire si plusieurs jobs sont stoppes?

---

## Enonce

### Vue d'Ensemble

Implementez un controleur de jobs pour gerer l'execution de commandes en foreground et background, avec support complet des signaux de controle du terminal.

### Architecture

```
                    Session
                       |
        +--------------+---------------+
        |              |               |
   Shell (leader)   Job 1 (pgid)    Job 2 (pgid)
        |              |               |
        |         +----+----+     +----+----+
        |         |    |    |     |    |    |
                 cmd1 cmd2 cmd3  cmd4 cmd5 cmd6
                 (pipeline)      (pipeline)

Terminal Control:
  - Shell est normalement le foreground process group
  - Quand un job est en foreground, son pgid devient le foreground process group
  - SIGTSTP est envoye au foreground process group quand Ctrl+Z est presse
```

### Specifications Fonctionnelles

#### Fonctionnalite 1: Structure du Job Controller

**API**:
```c
/**
 * Etats possibles d'un job.
 */
typedef enum {
    JOB_RUNNING,     // En cours d'execution
    JOB_STOPPED,     // Suspendu (SIGTSTP)
    JOB_DONE,        // Termine normalement
    JOB_TERMINATED,  // Tue par signal
    JOB_CONTINUED    // Repris apres suspension (transitoire)
} job_state_t;

/**
 * Informations sur un job.
 */
typedef struct {
    int id;               // Job ID (1, 2, 3, ...)
    pid_t pgid;           // Process Group ID
    job_state_t state;    // Etat actuel
    int exit_status;      // Statut de sortie (si DONE)
    int term_signal;      // Signal de terminaison (si TERMINATED)
    char *command;        // Ligne de commande originale
    bool notified;        // L'utilisateur a ete notifie?
    bool foreground;      // Est-ce le job foreground actuel?
} job_info_t;

/**
 * Controleur de jobs opaque.
 */
typedef struct jobctl jobctl_t;

/**
 * Cree un nouveau controleur de jobs.
 *
 * @return Pointeur vers le controleur, NULL si erreur
 *
 * @note Initialise la gestion de SIGCHLD
 * @note Le shell doit etre le leader de session
 */
jobctl_t *jobctl_create(void);

/**
 * Detruit le controleur et tue tous les jobs actifs.
 */
void jobctl_destroy(jobctl_t *jc);
```

#### Fonctionnalite 2: Lancement de Jobs

**API**:
```c
/**
 * Lance une commande comme nouveau job.
 *
 * @param jc Controleur de jobs
 * @param argv Arguments de la commande (NULL-terminated)
 * @param foreground true = foreground, false = background
 * @return Job ID (>0), ou -1 si erreur
 *
 * @note Cree un nouveau process group pour le job
 * @note Si foreground, donne le terminal au job et attend
 * @note Si background, retourne immediatement
 */
int job_spawn(jobctl_t *jc, char **argv, bool foreground);

/**
 * Lance un pipeline de commandes.
 *
 * @param jc Controleur de jobs
 * @param commands Tableau de commandes (chaque commande = char **)
 * @param n_commands Nombre de commandes dans le pipeline
 * @param foreground true = foreground, false = background
 * @return Job ID, ou -1 si erreur
 *
 * @note Tous les processus du pipeline sont dans le meme process group
 * @note Exemple: {"ls", "-la", NULL}, {"grep", "txt", NULL}
 */
int job_spawn_pipeline(jobctl_t *jc, char ***commands, int n_commands,
                       bool foreground);
```

**Comportement attendu pour job_spawn**:

1. `fork()` pour creer le processus enfant
2. Dans l'enfant:
   - `setpgid(0, 0)` pour creer un nouveau process group
   - Si foreground: `tcsetpgrp(STDIN_FILENO, getpid())`
   - Restaurer les handlers de signaux par defaut
   - `execvp()` pour lancer la commande
3. Dans le parent:
   - `setpgid(child_pid, child_pid)` (evite race condition)
   - Si foreground: `tcsetpgrp(STDIN_FILENO, child_pid)` et attendre
   - Si background: retourner immediatement

#### Fonctionnalite 3: Controle des Jobs

**API**:
```c
/**
 * Met un job en foreground.
 *
 * @param jc Controleur de jobs
 * @param job_id ID du job (ou 0 pour le job courant)
 * @param cont true = envoyer SIGCONT si stoppe
 * @return 0 si succes, -1 si job inexistant
 *
 * @note Donne le terminal au job
 * @note Attend que le job se termine ou soit stoppe
 */
int job_foreground(jobctl_t *jc, int job_id, bool cont);

/**
 * Met un job en background.
 *
 * @param jc Controleur de jobs
 * @param job_id ID du job (ou 0 pour le job courant)
 * @param cont true = envoyer SIGCONT si stoppe
 * @return 0 si succes, -1 si job inexistant
 *
 * @note N'attend pas le job
 * @note Affiche "[id] command &" si job etait stoppe
 */
int job_background(jobctl_t *jc, int job_id, bool cont);

/**
 * Envoie un signal a un job.
 *
 * @param jc Controleur de jobs
 * @param job_id ID du job
 * @param sig Signal a envoyer
 * @return 0 si succes, -1 si erreur
 *
 * @note Utilise killpg pour envoyer a tout le groupe
 */
int job_kill(jobctl_t *jc, int job_id, int sig);

/**
 * Attend la terminaison ou suspension de tous les jobs foreground.
 *
 * @param jc Controleur de jobs
 * @param job_id Job a attendre (0 = job foreground actuel)
 * @return Statut de sortie ou -1 si erreur
 */
int job_wait(jobctl_t *jc, int job_id);
```

#### Fonctionnalite 4: Listing et Notifications

**API**:
```c
/**
 * Liste tous les jobs actifs.
 *
 * @param jc Controleur de jobs
 * @param jobs_out Tableau pour stocker les infos (alloue par caller)
 * @param max_jobs Taille du tableau
 * @return Nombre de jobs retournes
 */
int job_list(jobctl_t *jc, job_info_t *jobs_out, int max_jobs);

/**
 * Recupere les infos d'un job specifique.
 *
 * @return 0 si trouve, -1 sinon
 */
int job_get_info(jobctl_t *jc, int job_id, job_info_t *info_out);

/**
 * Notifie les changements d'etat des jobs background.
 *
 * @param jc Controleur de jobs
 * @return Nombre de notifications affichees
 *
 * @note Doit etre appele regulierement (avant chaque prompt)
 * @note Affiche les jobs termines et les marque comme notifies
 * @note Retire les jobs termines et notifies de la liste
 */
int job_notify(jobctl_t *jc);

/**
 * Recupere le job courant (dernier job stoppe ou mis en background).
 *
 * @return Job ID du job courant, ou 0 si aucun
 */
int job_current(jobctl_t *jc);
```

### Format de Notification

Le format des notifications doit suivre la convention shell:

```
[job_id]+ state     command
[1]+  Stopped     sleep 100
[2]-  Running     find / -name "*.c" &
[3]   Done        ls -la
[4]   Terminated  cat /dev/urandom  (SIGPIPE)
```

- `+` marque le job courant (celui que `fg` sans argument reprendra)
- `-` marque le job precedent
- `Done` pour exit status 0
- `Done(N)` pour exit status N != 0
- `Terminated` avec le nom du signal

---

## Contraintes Techniques

### Standards C

- **Norme**: C17 (ISO/IEC 9899:2018)
- **Compilation**: `gcc -Wall -Wextra -Werror -std=c17`

### Fonctions Autorisees

```
Processus:
  - fork, execvp, _exit
  - waitpid (avec WNOHANG, WUNTRACED, WCONTINUED)
  - getpid, getppid, getpgid, setpgid
  - setsid, getsid

Groupes de processus:
  - killpg (envoie signal a un groupe)
  - tcgetpgrp, tcsetpgrp (controle terminal)
  - isatty, ttyname

Signaux:
  - sigaction, sigemptyset, sigaddset, sigfillset
  - sigprocmask, sigsuspend
  - kill

Memoire:
  - malloc, free, realloc, calloc
  - strdup, strncpy, strlen

I/O:
  - open, close, dup, dup2
  - read, write
  - printf, fprintf, snprintf
  - perror

Divers:
  - strsignal (nom d'un signal)
```

### Contraintes Specifiques

- [ ] Pas de variables globales mutables (sauf pour le handler SIGCHLD si necessaire)
- [ ] Le shell ne doit JAMAIS etre stoppe par SIGTSTP (ignore ou bloque)
- [ ] Les jobs doivent avoir les handlers par defaut AVANT exec
- [ ] Maximum 64 jobs simultanes
- [ ] Les pipes doivent etre fermes correctement dans le parent

### Exigences de Securite

- [ ] Aucune fuite memoire
- [ ] Aucun zombie (tous les enfants doivent etre recoltes)
- [ ] Pas de race condition sur les signaux (signal mask correct)
- [ ] Le terminal doit toujours revenir au shell

---

## Format de Rendu

### Fichiers a Rendre

```
ex09/
|-- jobctl.h           # API publique
|-- jobctl.c           # Implementation
|-- Makefile
```

### Signatures de Fonctions

#### jobctl.h

```c
#ifndef JOBCTL_H
#define JOBCTL_H

#include <stdbool.h>
#include <sys/types.h>

typedef enum {
    JOB_RUNNING,
    JOB_STOPPED,
    JOB_DONE,
    JOB_TERMINATED,
    JOB_CONTINUED
} job_state_t;

typedef struct {
    int id;
    pid_t pgid;
    job_state_t state;
    int exit_status;
    int term_signal;
    char *command;
    bool notified;
    bool foreground;
} job_info_t;

typedef struct jobctl jobctl_t;

// Lifecycle
jobctl_t *jobctl_create(void);
void jobctl_destroy(jobctl_t *jc);

// Spawn
int job_spawn(jobctl_t *jc, char **argv, bool foreground);
int job_spawn_pipeline(jobctl_t *jc, char ***commands, int n_commands,
                       bool foreground);

// Control
int job_foreground(jobctl_t *jc, int job_id, bool cont);
int job_background(jobctl_t *jc, int job_id, bool cont);
int job_kill(jobctl_t *jc, int job_id, int sig);
int job_wait(jobctl_t *jc, int job_id);

// Info
int job_list(jobctl_t *jc, job_info_t *jobs_out, int max_jobs);
int job_get_info(jobctl_t *jc, int job_id, job_info_t *info_out);
int job_notify(jobctl_t *jc);
int job_current(jobctl_t *jc);

#endif /* JOBCTL_H */
```

### Makefile

```makefile
NAME = libjobctl.a
TEST = jobctl_test

CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c17
LDFLAGS =

SRCS = jobctl.c
OBJS = $(SRCS:.c=.o)

all: $(NAME)

$(NAME): $(OBJS)
	ar rcs $(NAME) $(OBJS)

test: $(NAME)
	$(CC) $(CFLAGS) -o $(TEST) test_jobctl.c -L. -ljobctl $(LDFLAGS)
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

### Exemple 1: Job Foreground Simple

```c
#include "jobctl.h"
#include <stdio.h>

int main(void) {
    jobctl_t *jc = jobctl_create();

    // Lancer "sleep 2" en foreground
    char *argv[] = {"sleep", "2", NULL};
    int job_id = job_spawn(jc, argv, true);  // foreground = true

    printf("Job %d finished\n", job_id);

    jobctl_destroy(jc);
    return 0;
}
```

**Sortie** (apres 2 secondes):
```
Job 1 finished
```

### Exemple 2: Job Background

```c
#include "jobctl.h"
#include <stdio.h>
#include <unistd.h>

int main(void) {
    jobctl_t *jc = jobctl_create();

    // Lancer "sleep 5" en background
    char *argv[] = {"sleep", "5", NULL};
    int job_id = job_spawn(jc, argv, false);  // foreground = false

    printf("[%d] Started in background\n", job_id);

    // Simuler un prompt loop
    for (int i = 0; i < 7; i++) {
        sleep(1);
        printf("tick %d\n", i);
        job_notify(jc);  // Verifie si des jobs ont change d'etat
    }

    jobctl_destroy(jc);
    return 0;
}
```

**Sortie**:
```
[1] Started in background
tick 0
tick 1
tick 2
tick 3
tick 4
[1]+ Done        sleep 5
tick 5
tick 6
```

### Exemple 3: Foreground/Background avec Ctrl+Z

```c
#include "jobctl.h"
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

static volatile sig_atomic_t got_sigchld = 0;

void sigchld_handler(int sig) {
    (void)sig;
    got_sigchld = 1;
}

int main(void) {
    // Handler pour SIGCHLD
    struct sigaction sa;
    sa.sa_handler = sigchld_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = SA_RESTART;
    sigaction(SIGCHLD, &sa, NULL);

    jobctl_t *jc = jobctl_create();

    // Lancer un processus long
    char *argv[] = {"cat", NULL};  // cat attend sur stdin
    int job_id = job_spawn(jc, argv, true);  // foreground

    // A ce stade, si l'utilisateur tape Ctrl+Z:
    // - cat recoit SIGTSTP et s'arrete
    // - job_spawn retourne (car waitpid avec WUNTRACED)

    job_info_t info;
    job_get_info(jc, job_id, &info);

    if (info.state == JOB_STOPPED) {
        printf("Job %d stopped, putting in background...\n", job_id);
        job_background(jc, job_id, true);  // cont = true envoie SIGCONT
    }

    // Maintenant cat tourne en background, on peut faire autre chose
    sleep(2);

    // Remettre en foreground
    printf("Bringing back to foreground...\n");
    job_foreground(jc, job_id, false);

    jobctl_destroy(jc);
    return 0;
}
```

### Exemple 4: Listing des Jobs

```c
#include "jobctl.h"
#include <stdio.h>
#include <unistd.h>

const char *state_str(job_state_t s) {
    switch (s) {
        case JOB_RUNNING: return "Running";
        case JOB_STOPPED: return "Stopped";
        case JOB_DONE: return "Done";
        case JOB_TERMINATED: return "Terminated";
        case JOB_CONTINUED: return "Continued";
    }
    return "Unknown";
}

int main(void) {
    jobctl_t *jc = jobctl_create();

    // Lancer plusieurs jobs en background
    char *cmd1[] = {"sleep", "10", NULL};
    char *cmd2[] = {"sleep", "20", NULL};
    char *cmd3[] = {"sleep", "30", NULL};

    job_spawn(jc, cmd1, false);
    job_spawn(jc, cmd2, false);
    job_spawn(jc, cmd3, false);

    // Stopper le job 2
    job_kill(jc, 2, SIGSTOP);
    usleep(100000);  // Laisser le temps au signal

    // Lister les jobs
    job_info_t jobs[10];
    int n = job_list(jc, jobs, 10);

    printf("Active jobs:\n");
    for (int i = 0; i < n; i++) {
        printf("[%d]%c %-12s %s\n",
               jobs[i].id,
               (jobs[i].id == job_current(jc)) ? '+' : ' ',
               state_str(jobs[i].state),
               jobs[i].command);
    }

    jobctl_destroy(jc);
    return 0;
}
```

**Sortie**:
```
Active jobs:
[1]  Running      sleep 10
[2]+ Stopped      sleep 20
[3]  Running      sleep 30
```

---

## Tests de la Moulinette

### Tests Fonctionnels de Base

#### Test 01: Foreground Simple
```yaml
description: "Lancer une commande en foreground"
input: |
  job_spawn(jc, {"echo", "hello", NULL}, true);
expected_output: "hello"
expected_behavior: "Attend la fin avant de retourner"
```

#### Test 02: Background Simple
```yaml
description: "Lancer une commande en background"
input: |
  id = job_spawn(jc, {"sleep", "1", NULL}, false);
expected_behavior: |
  - Retourne immediatement
  - id > 0
  - job_notify apres 1s affiche "[id] Done"
```

#### Test 03: Ctrl+Z (SIGTSTP)
```yaml
description: "Suspension d'un job foreground"
scenario: |
  id = job_spawn(jc, {"cat", NULL}, true);
  // Simuler Ctrl+Z en envoyant SIGTSTP au process group
  kill(-pgid, SIGTSTP);
expected_behavior: |
  - job_spawn retourne (car cat est stoppe)
  - job_get_info retourne state = JOB_STOPPED
```

#### Test 04: fg Command
```yaml
description: "Reprendre un job stoppe en foreground"
scenario: |
  id = job_spawn(jc, {"sleep", "100", NULL}, true);
  kill(-pgid, SIGTSTP);  // Stop
  // ... sleep est maintenant stoppe ...
  job_foreground(jc, id, true);  // Reprendre
expected_behavior: |
  - SIGCONT envoye
  - Terminal donne au job
  - job_foreground bloque jusqu'a terminaison/suspension
```

#### Test 05: bg Command
```yaml
description: "Reprendre un job stoppe en background"
scenario: |
  id = job_spawn(jc, {"sleep", "5", NULL}, true);
  kill(-pgid, SIGTSTP);
  job_background(jc, id, true);
expected_behavior: |
  - SIGCONT envoye
  - job_background retourne immediatement
  - job tourne en background
```

### Tests de Process Groups

#### Test 10: Chaque Job a son Propre Group
```yaml
description: "Verification des process groups"
scenario: |
  id1 = job_spawn(jc, {"sleep", "10", NULL}, false);
  id2 = job_spawn(jc, {"sleep", "10", NULL}, false);
  job_get_info(jc, id1, &info1);
  job_get_info(jc, id2, &info2);
expected: |
  info1.pgid != info2.pgid
  info1.pgid != getpid() (shell's pid)
```

#### Test 11: killpg Affecte Tout le Groupe
```yaml
description: "Signal envoye a tout le process group"
scenario: |
  // Pipeline: sleep 100 | sleep 100 | sleep 100
  char **cmds[] = {{"sleep", "100", NULL}, {"sleep", "100", NULL}, {"sleep", "100", NULL}};
  id = job_spawn_pipeline(jc, cmds, 3, false);
  job_kill(jc, id, SIGTERM);
expected_behavior: "Les 3 processus sont termines"
```

### Tests de Robustesse

#### Test 20: Terminal Revient au Shell
```yaml
description: "Le shell recupere le terminal apres chaque job"
scenario: |
  for (int i = 0; i < 5; i++) {
    job_spawn(jc, {"true", NULL}, true);
  }
expected: "tcgetpgrp(0) == getpid() apres chaque job"
```

#### Test 21: Pas de Zombies
```yaml
description: "Tous les processus sont recoltes"
scenario: |
  for (int i = 0; i < 100; i++) {
    job_spawn(jc, {"true", NULL}, false);
  }
  sleep(1);
  job_notify(jc);
validation: "ps aux | grep defunct -> aucun zombie"
```

#### Test 22: Job Inexistant
```yaml
test_cases:
  - input: "job_foreground(jc, 999, true)"
    expected: "-1"
  - input: "job_kill(jc, 0, SIGTERM)"
    expected: "-1 si aucun job courant"
```

### Tests de Performance

#### Test 30: Spawn Rapide
```yaml
description: "Lancement rapide de nombreux jobs"
scenario: |
  for (int i = 0; i < 50; i++) {
    job_spawn(jc, {"true", NULL}, false);
  }
expected_max_time: "< 500ms"
```

---

## Criteres d'Evaluation

### Note Minimale Requise: 80/100

### Detail de la Notation (Total: 100 points)

#### 1. Correction (40 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Foreground/background | 12 | Jobs lancent correctement |
| SIGTSTP/SIGCONT | 10 | Suspension et reprise |
| Process groups | 10 | Groupes correctement crees |
| Terminal control | 8 | tcsetpgrp correct |

#### 2. Securite (25 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Pas de zombies | 10 | waitpid correctement utilise |
| Terminal revient | 8 | Shell recupere toujours le terminal |
| Pas de race | 4 | setpgid dans parent ET enfant |
| Signal safety | 3 | Handlers async-signal-safe |

#### 3. Conception (20 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Structure job | 8 | Stockage efficace des jobs |
| Gestion SIGCHLD | 7 | Detection des changements d'etat |
| API coherente | 5 | Interface bien pensee |

#### 4. Lisibilite (15 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Nommage | 5 | Fonctions explicites |
| Organisation | 4 | Code structure |
| Commentaires | 3 | Parties critiques documentees |
| Style | 3 | Code propre |

---

## Indices et Ressources

### Reflexions pour Demarrer

<details>
<summary>Question 1: Comment eviter la race condition avec setpgid?</summary>

Il y a une race entre le moment ou l'enfant appelle `setpgid` et le moment ou le parent doit connaitre le pgid.

**Solution**: Le parent ET l'enfant appellent `setpgid`:

```c
pid_t pid = fork();
if (pid == 0) {
    // Enfant
    setpgid(0, 0);  // Cree son propre groupe
    // ...
    execvp(...);
} else {
    // Parent
    setpgid(pid, pid);  // Le meme appel, un des deux gagnera
    // ...
}
```

Si un appel echoue (car l'autre a deja reussi), c'est OK.

</details>

<details>
<summary>Question 2: Comment detecter qu'un job est stoppe?</summary>

Utilisez `waitpid` avec les options appropriees:

```c
int status;
pid_t result = waitpid(-pgid, &status, WUNTRACED | WNOHANG);

if (WIFSTOPPED(status)) {
    int sig = WSTOPSIG(status);
    // Job stoppe par signal sig
}

if (WIFCONTINUED(status)) {
    // Job a repris (SIGCONT)
}

if (WIFEXITED(status)) {
    int code = WEXITSTATUS(status);
    // Job termine normalement
}

if (WIFSIGNALED(status)) {
    int sig = WTERMSIG(status);
    // Job tue par signal
}
```

</details>

<details>
<summary>Question 3: Comment le shell doit-il gerer SIGTSTP?</summary>

Le shell lui-meme ne doit JAMAIS etre stoppe!

```c
// Au demarrage du shell
signal(SIGTSTP, SIG_IGN);  // Ignorer Ctrl+Z pour le shell
signal(SIGTTIN, SIG_IGN);  // Ignorer si pas foreground
signal(SIGTTOU, SIG_IGN);  // Ignorer si ecrit depuis background

// Avant de lancer un job (dans l'enfant, AVANT exec)
signal(SIGTSTP, SIG_DFL);  // Restaurer le handler par defaut
signal(SIGTTIN, SIG_DFL);
signal(SIGTTOU, SIG_DFL);
```

</details>

<details>
<summary>Question 4: Comment gerer SIGCHLD sans race condition?</summary>

Bloquez SIGCHLD pendant les operations critiques:

```c
sigset_t mask, oldmask;
sigemptyset(&mask);
sigaddset(&mask, SIGCHLD);

// Bloquer SIGCHLD
sigprocmask(SIG_BLOCK, &mask, &oldmask);

// ... operations sur la liste des jobs ...
// ... fork, waitpid, etc ...

// Debloquer SIGCHLD
sigprocmask(SIG_SETMASK, &oldmask, NULL);
```

Dans le handler SIGCHLD, utilisez `waitpid(-1, &status, WNOHANG)` en boucle pour recolter tous les enfants.

</details>

### Ressources Recommandees

#### Documentation
- **Man pages**: `man 2 setpgid`, `man 2 tcsetpgrp`, `man 7 signal`
- **POSIX.1-2017**: Job control specification
- **Module ODYSSEY 2.2.8-9**: Theorie des groupes de processus

#### Lectures Complementaires
- "Advanced Programming in the UNIX Environment" par Stevens, Chapitre 9 (Process Relationships)
- GNU Bash source code: `jobs.c` et `nojobs.c`

### Pieges Frequents

1. **Oublier de restaurer les handlers avant exec**:
   - Le shell ignore SIGTSTP, mais les enfants doivent l'accepter!
   - **Solution**: `signal(SIGTSTP, SIG_DFL)` dans l'enfant avant exec

2. **Race condition sur setpgid**:
   - L'enfant peut execer avant que le parent appelle setpgid
   - **Solution**: Les deux appellent setpgid

3. **Ne pas recuperer le terminal apres un job foreground**:
   - Si un job crash, le terminal reste au groupe mort!
   - **Solution**: `tcsetpgrp(STDIN_FILENO, getpid())` apres waitpid

4. **Zombies des jobs background**:
   - Les jobs background ne sont pas attendus immediatement
   - **Solution**: Handler SIGCHLD qui appelle waitpid(-1, ..., WNOHANG)

5. **Attendre sur le mauvais PID**:
   - waitpid(pid, ...) n'attend qu'UN processus d'un pipeline
   - **Solution**: waitpid(-pgid, ...) pour attendre tout le groupe

---

## Notes du Concepteur

<details>
<summary>Solution de Reference (Concepteur uniquement)</summary>

**Structure interne recommandee**:

```c
struct job {
    int id;
    pid_t pgid;
    job_state_t state;
    int status;           // waitpid status
    char *command;
    bool notified;
    struct job *next;     // Liste chainee
};

struct jobctl {
    struct job *jobs;     // Liste des jobs
    int next_id;          // Prochain ID de job
    int current_job;      // ID du job courant (+)
    int previous_job;     // ID du job precedent (-)
    pid_t shell_pgid;     // PGID du shell
    int terminal_fd;      // fd du terminal
    struct sigaction old_sigchld;  // Pour restauration
};
```

**Flux pour job_spawn foreground**:
1. Bloquer SIGCHLD
2. fork()
3. Enfant: setpgid(0,0), tcsetpgrp, reset handlers, exec
4. Parent: setpgid(child, child), tcsetpgrp(child), creer job
5. Parent: wait_for_job() avec WUNTRACED
6. Parent: tcsetpgrp(shell), debloquer SIGCHLD

</details>

---

## Historique

```yaml
version: "1.0"
created: "2026-01-04"
author: "ODYSSEY Curriculum Team"
last_modified: "2026-01-04"
changes:
  - "Version initiale"
```

---

## Auto-Evaluation: **96/100**

| Critere | Score | Justification |
|---------|-------|---------------|
| Originalite | 10/10 | Job controller custom, pas bash |
| Couverture concepts | 10/10 | 2.2.8 + 2.2.9 complets |
| Qualite pedagogique | 9/10 | Questions de reflexion incluses |
| Testabilite | 10/10 | Tests automatisables |
| Difficulte appropriee | 9/10 | Difficile, 6h realiste |
| Clarte enonce | 9/10 | API detaillee |
| Cas limites | 10/10 | Race conditions documentees |
| Securite | 10/10 | Zombies, terminal |
| Ressources | 9/10 | Pieges bien documentes |

---

*Template ODYSSEY Phase 2 - Module 2.2 Exercise 09*
