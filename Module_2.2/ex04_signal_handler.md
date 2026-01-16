# [Module 2.2] - Exercise 04: Signal Handler Pro

## Métadonnées

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex04"
difficulty: moyen
estimated_time: "6 heures"
prerequisite_exercises: ["ex00", "ex01", "ex02", "ex03"]
concepts_requis:
  - "Compréhension des processus UNIX"
  - "Gestion des erreurs système"
  - "Pointeurs de fonction"
  - "Structures de données dynamiques"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.2.10.a | Signal fundamentals | Nature asynchrone des signaux, interruption d'exécution |
| 2.2.10.b | Signal numbering | Signaux standard (1-31), conventions et portabilité |
| 2.2.10.c | Signal disposition | Actions par défaut (terminate, ignore, stop, continue) |
| 2.2.10.d | Signal generation | Sources de signaux (kernel, process, terminal) |
| 2.2.11.a | signal() function | Limitations de l'ancienne API, non-portabilité |
| 2.2.11.b | sigaction() | API moderne, contrôle fin du comportement |
| 2.2.11.c | Handler installation | Enregistrement et chaînage de handlers |
| 2.2.11.d | SA_RESTART flag | Redémarrage automatique des syscalls interrompus |
| 2.2.11.e | SA_SIGINFO flag | Informations étendues sur le signal |
| 2.2.12.a | Signal masks | Blocage temporaire de signaux |
| 2.2.12.b | sigprocmask() | Manipulation du masque de processus |
| 2.2.12.c | sigpending() | Signaux en attente |
| 2.2.12.d | sigsuspend() | Attente atomique avec masque temporaire |
| 2.2.15.a | Async-signal-safety | Fonctions sûres dans un handler |
| 2.2.15.b | Volatile sig_atomic_t | Variables partagées sûres |
| 2.2.15.c | Self-pipe trick | Communication sûre handler->main loop |
| 2.2.15.d | Reentrancy issues | Dangers des fonctions non-réentrantes |

### Objectifs Pédagogiques

À la fin de cet exercice, vous saurez:
1. Implémenter une gestion de signaux robuste avec `sigaction()`
2. Manipuler les masques de signaux pour protéger les sections critiques
3. Écrire du code async-signal-safe dans les handlers
4. Utiliser le pattern self-pipe pour une intégration sûre avec les event loops
5. Distinguer les signaux bloqués, ignorés et en attente

---

## Contexte

Les signaux UNIX sont le mécanisme historique de notification asynchrone entre le noyau et les processus, ou entre processus. Contrairement aux appels système synchrones, les signaux interrompent l'exécution du programme à n'importe quel moment, créant des défis uniques en termes de synchronisation et de sécurité.

Dans les systèmes modernes, une gestion robuste des signaux est critique pour:
- **Les serveurs réseau**: SIGHUP pour rechargement de config, SIGTERM pour arrêt gracieux
- **Les démons système**: SIGUSR1/SIGUSR2 pour contrôle, SIGCHLD pour récolte des enfants
- **Les applications interactives**: SIGINT, SIGTSTP, SIGCONT pour job control
- **Les systèmes embarqués**: Signaux temps-réel pour événements matériels

Le défi principal est que **la plupart du code n'est pas async-signal-safe**. Appeler `malloc()`, `printf()`, ou même `exit()` depuis un signal handler peut provoquer des deadlocks, corruptions de données ou crashs subtils. Votre bibliothèque doit offrir une abstraction sûre et élégante.

**Exemple concret**: Un serveur web reçoit SIGTERM. Il doit arrêter d'accepter de nouvelles connexions, finir de traiter les requêtes en cours, sauvegarder l'état, puis quitter proprement - le tout sans corrompre les données ni laisser de connexions orphelines.

---

## Énoncé

### Vue d'Ensemble

Créez une bibliothèque `sighandler` qui fournit une abstraction robuste et async-signal-safe pour la gestion des signaux UNIX. La bibliothèque doit permettre d'enregistrer des handlers personnalisés, de manipuler les masques de signaux, et d'intégrer la gestion des signaux dans une event loop via le pattern self-pipe.

### Spécifications Fonctionnelles

#### Partie 1: Structures de Base et Initialisation

```c
#ifndef SIGHANDLER_H
#define SIGHANDLER_H

#include <signal.h>
#include <stdint.h>
#include <sys/types.h>

// Capacité maximale (tous les signaux standard + RT)
#define SIG_MAX_HANDLERS 64

// Information étendue sur un signal reçu
typedef struct sig_info {
    int             signum;         // Numéro du signal
    pid_t           sender_pid;     // PID de l'émetteur (si disponible)
    uid_t           sender_uid;     // UID de l'émetteur
    int             code;           // Code additionnel (SI_USER, SI_KERNEL, etc.)
    union sigval    value;          // Valeur associée (signaux RT)
    struct timespec timestamp;      // Moment de réception
} sig_info_t;

// Type de callback utilisateur
typedef void (*sig_callback_fn)(const sig_info_t *info, void *user_data);

// Gestionnaire de signaux opaque
typedef struct sighandler sighandler_t;

/**
 * Crée un nouveau gestionnaire de signaux.
 *
 * @return Pointeur vers le gestionnaire, ou NULL si échec
 *
 * @note Doit être appelé une seule fois au début du programme.
 *       Initialise le self-pipe pour la communication async-safe.
 */
sighandler_t *sig_create(void);

/**
 * Détruit le gestionnaire et restaure les handlers par défaut.
 *
 * @param handler Gestionnaire à détruire (peut être NULL)
 */
void sig_destroy(sighandler_t *handler);
```

#### Partie 2: Enregistrement de Handlers

```c
// Flags pour l'enregistrement
typedef enum {
    SIG_FLAG_NONE       = 0,
    SIG_FLAG_RESTART    = (1 << 0),  // SA_RESTART: redémarrer les syscalls interrompus
    SIG_FLAG_NODEFER    = (1 << 1),  // SA_NODEFER: ne pas bloquer le signal pendant le handler
    SIG_FLAG_RESETHAND  = (1 << 2),  // SA_RESETHAND: restaurer le handler par défaut après exécution
    SIG_FLAG_NOCLDSTOP  = (1 << 3),  // SA_NOCLDSTOP: pas de SIGCHLD pour les stops (SIGCHLD uniquement)
    SIG_FLAG_ONESHOT    = (1 << 4),  // Custom: callback appelé une seule fois
} sig_flags_t;

/**
 * Enregistre un callback pour un signal.
 *
 * @param handler     Gestionnaire de signaux
 * @param signum      Numéro du signal (SIGINT, SIGTERM, etc.)
 * @param callback    Fonction callback (peut être NULL pour ignorer)
 * @param user_data   Données utilisateur passées au callback
 * @param flags       Combinaison de SIG_FLAG_*
 *
 * @return 0 si succès, -1 si erreur (errno positionné)
 *
 * @note SIGKILL et SIGSTOP ne peuvent pas être interceptés.
 * @warning Le callback DOIT être async-signal-safe ou utiliser sig_defer().
 */
int sig_register(sighandler_t *handler, int signum,
                 sig_callback_fn callback, void *user_data,
                 sig_flags_t flags);

/**
 * Désenregistre un callback.
 *
 * @param handler Gestionnaire de signaux
 * @param signum  Numéro du signal
 *
 * @return 0 si succès, -1 si signal non enregistré
 */
int sig_unregister(sighandler_t *handler, int signum);

/**
 * Configure l'action par défaut pour un signal.
 *
 * @param handler Gestionnaire
 * @param signum  Numéro du signal
 * @param action  SIG_DFL, SIG_IGN, ou comportement précédent
 *
 * @return 0 si succès, -1 si erreur
 */
int sig_set_default(sighandler_t *handler, int signum, int action);
```

#### Partie 3: Manipulation des Masques de Signaux

```c
/**
 * Bloque temporairement un ensemble de signaux.
 *
 * @param handler Gestionnaire
 * @param signals Tableau de numéros de signaux (terminé par 0)
 * @param saved   Si non-NULL, sauvegarde le masque précédent
 *
 * @return 0 si succès, -1 si erreur
 */
int sig_block(sighandler_t *handler, const int *signals, sigset_t *saved);

/**
 * Débloque un ensemble de signaux.
 *
 * @param handler Gestionnaire
 * @param signals Tableau de numéros de signaux (terminé par 0)
 *
 * @return 0 si succès, -1 si erreur
 */
int sig_unblock(sighandler_t *handler, const int *signals);

/**
 * Restaure un masque de signaux précédemment sauvegardé.
 *
 * @param handler Gestionnaire
 * @param saved   Masque sauvegardé par sig_block()
 *
 * @return 0 si succès, -1 si erreur
 */
int sig_restore_mask(sighandler_t *handler, const sigset_t *saved);

/**
 * Vérifie les signaux en attente (bloqués mais reçus).
 *
 * @param handler Gestionnaire
 * @param pending Si non-NULL, reçoit l'ensemble des signaux en attente
 *
 * @return Nombre de signaux en attente
 */
int sig_pending(sighandler_t *handler, sigset_t *pending);

/**
 * Attend atomiquement un signal parmi un ensemble.
 *
 * @param handler   Gestionnaire
 * @param wait_for  Ensemble de signaux à attendre
 * @param timeout   Timeout en millisecondes (-1 = infini)
 * @param received  Si non-NULL, reçoit le signal qui a interrompu l'attente
 *
 * @return 0 si signal reçu, -1 si timeout ou erreur
 */
int sig_wait(sighandler_t *handler, const sigset_t *wait_for,
             int timeout_ms, int *received);
```

#### Partie 4: Intégration Self-Pipe (Async-Signal-Safety)

```c
/**
 * Retourne le file descriptor de lecture du self-pipe.
 *
 * Ce FD peut être utilisé dans select()/poll()/epoll() pour
 * intégrer les signaux dans une event loop.
 *
 * @param handler Gestionnaire
 *
 * @return FD de lecture, ou -1 si erreur
 *
 * @note Le FD est en mode non-bloquant et close-on-exec.
 */
int sig_get_fd(sighandler_t *handler);

/**
 * Traite les signaux en attente dans le self-pipe.
 *
 * Cette fonction doit être appelée quand sig_get_fd() est lisible.
 * Elle invoque les callbacks enregistrés pour chaque signal reçu.
 *
 * @param handler Gestionnaire
 *
 * @return Nombre de signaux traités, ou -1 si erreur
 *
 * @note Cette fonction est thread-safe et peut être appelée
 *       depuis le thread principal en toute sécurité.
 */
int sig_process(sighandler_t *handler);

/**
 * Marque un signal pour traitement différé.
 *
 * Utilisé dans un handler async-signal-safe pour différer
 * le traitement réel au thread principal.
 *
 * @param handler Gestionnaire
 * @param info    Information sur le signal
 *
 * @return 0 si succès (écrit dans le self-pipe)
 *
 * @note Cette fonction EST async-signal-safe.
 */
int sig_defer(sighandler_t *handler, const sig_info_t *info);
```

#### Partie 5: Utilitaires et Informations

```c
/**
 * Retourne le nom lisible d'un signal.
 *
 * @param signum Numéro du signal
 *
 * @return Nom du signal (ex: "SIGTERM") ou "UNKNOWN"
 *
 * @note Cette fonction est thread-safe et async-signal-safe.
 */
const char *sig_name(int signum);

/**
 * Retourne le numéro d'un signal à partir de son nom.
 *
 * @param name Nom du signal (ex: "SIGTERM" ou "TERM")
 *
 * @return Numéro du signal, ou -1 si inconnu
 */
int sig_number(const char *name);

/**
 * Retourne l'action par défaut d'un signal.
 *
 * @param signum Numéro du signal
 *
 * @return 'T' (terminate), 'I' (ignore), 'C' (core), 'S' (stop), 'X' (continue)
 */
char sig_default_action(int signum);

/**
 * Affiche les statistiques des signaux reçus.
 *
 * @param handler Gestionnaire
 * @param fd      File descriptor pour la sortie
 */
void sig_print_stats(sighandler_t *handler, int fd);

/**
 * Retourne le nombre de fois qu'un signal a été reçu.
 *
 * @param handler Gestionnaire
 * @param signum  Numéro du signal
 *
 * @return Compteur (volatile, peut changer)
 */
uint64_t sig_count(sighandler_t *handler, int signum);

#endif /* SIGHANDLER_H */
```

### Spécifications Techniques

#### Architecture Interne

```
                    ┌─────────────────────────────────┐
                    │     Signal Handler Library       │
                    └─────────────────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐          ┌───────────────┐          ┌───────────────┐
│  Registration │          │   Self-Pipe   │          │  Signal Mask  │
│    Table      │          │    Handler    │          │   Management  │
├───────────────┤          ├───────────────┤          ├───────────────┤
│ signum → {    │          │ write_fd [W]  │          │ sigprocmask() │
│   callback,   │◄─────────│ read_fd  [R]  │─────────►│ sigpending()  │
│   user_data,  │          │ (non-block)   │          │ sigsuspend()  │
│   flags,      │          └───────────────┘          └───────────────┘
│   count       │                  │
│ }             │                  │
└───────────────┘                  ▼
                          ┌───────────────┐
                          │  Event Loop   │
                          │  Integration  │
                          │  (poll/epoll) │
                          └───────────────┘
```

#### Le Pattern Self-Pipe Expliqué

```c
// Le handler de signal (async-signal-safe)
static void internal_handler(int signum, siginfo_t *si, void *context) {
    // SEULES opérations autorisées: lecture/écriture de variables
    // sig_atomic_t et write() sur le self-pipe

    uint8_t sig_byte = (uint8_t)signum;
    write(g_handler->pipe_write_fd, &sig_byte, 1);  // Async-safe!

    // Le traitement réel se fait dans sig_process()
}
```

#### Complexité Attendue

- **Enregistrement**: O(1)
- **Recherche de callback**: O(1) (tableau indexé par signum)
- **sig_process()**: O(n) où n = nombre de signaux reçus

---

## Contraintes Techniques

### Standards C

- **Norme**: C17 (ISO/IEC 9899:2018)
- **Compilation**: `gcc -Wall -Wextra -Werror -std=c17 -D_POSIX_C_SOURCE=200809L`
- **Options additionnelles**: Aucune bibliothèque externe

### Fonctions Autorisées

```
POSIX Signaux:
  - sigaction, sigemptyset, sigfillset, sigaddset, sigdelset, sigismember
  - sigprocmask, sigpending, sigsuspend, sigwait, sigtimedwait
  - kill, raise

POSIX I/O:
  - pipe, pipe2 (si disponible), read, write, close
  - fcntl (pour O_NONBLOCK, FD_CLOEXEC)

Mémoire:
  - malloc, free, calloc, realloc

Temps:
  - clock_gettime (CLOCK_REALTIME)

Divers:
  - memset, memcpy, strlen, strcmp, strncmp
  - snprintf (pour sig_print_stats uniquement, PAS dans les handlers!)
```

### Fonctions INTERDITES

```
INTERDITES (non async-signal-safe):
  - printf, fprintf, sprintf (sauf snprintf hors handler)
  - malloc/free/realloc dans les handlers
  - exit, _exit dans les handlers (utiliser _Exit si nécessaire)
  - Toute fonction non listée dans signal-safety(7)
```

### Contraintes Spécifiques

- [ ] Aucune variable globale (sauf une pour le contexte du handler interne)
- [ ] Tous les handlers internes doivent être async-signal-safe
- [ ] Le self-pipe doit être en mode non-bloquant
- [ ] FD_CLOEXEC sur les pipes internes
- [ ] Thread-safe pour sig_process() (un seul thread doit l'appeler)
- [ ] Gestion correcte de EINTR sur toutes les opérations

### Exigences de Sécurité

- [ ] Aucune fuite mémoire (vérification Valgrind)
- [ ] Pas de race conditions entre handler et sig_process()
- [ ] volatile sig_atomic_t pour les compteurs partagés
- [ ] Vérification de tous les retours de sigaction()
- [ ] Protection contre les signaux pendant sig_destroy()

---

## Format de Rendu

### Fichiers à Rendre

```
ex04/
├── sighandler.h          # Header avec API publique
├── sighandler.c          # Implémentation principale
├── sighandler_internal.h # Structures internes (optionnel)
├── main.c                # Programme de démonstration
└── Makefile
```

### Makefile Requis

```makefile
NAME = sighandler_demo
LIB = libsighandler.a

CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c17 -D_POSIX_C_SOURCE=200809L
LDFLAGS = -L. -lsighandler

SRCS = sighandler.c
OBJS = $(SRCS:.c=.o)

all: $(LIB) $(NAME)

$(LIB): $(OBJS)
	ar rcs $@ $^

$(NAME): main.o $(LIB)
	$(CC) -o $@ main.o $(LDFLAGS)

clean:
	rm -f $(OBJS) main.o

fclean: clean
	rm -f $(NAME) $(LIB)

re: fclean all

.PHONY: all clean fclean re
```

---

## Exemples d'Utilisation

### Exemple 1: Gestion Basique de SIGINT et SIGTERM

```c
#include "sighandler.h"
#include <stdio.h>
#include <unistd.h>

volatile sig_atomic_t running = 1;

void shutdown_handler(const sig_info_t *info, void *user_data) {
    (void)user_data;
    printf("Received %s from PID %d\n", sig_name(info->signum), info->sender_pid);
    running = 0;
}

int main(void) {
    sighandler_t *sh = sig_create();
    if (!sh) {
        return 1;
    }

    // Enregistrer le même handler pour SIGINT et SIGTERM
    sig_register(sh, SIGINT, shutdown_handler, NULL, SIG_FLAG_RESTART);
    sig_register(sh, SIGTERM, shutdown_handler, NULL, SIG_FLAG_RESTART);

    printf("PID: %d - Press Ctrl+C or send SIGTERM\n", getpid());

    while (running) {
        pause();  // Attendre un signal
        sig_process(sh);  // Traiter les signaux en attente
    }

    sig_print_stats(sh, STDOUT_FILENO);
    sig_destroy(sh);
    return 0;
}
```

**Sortie attendue**:
```
PID: 12345 - Press Ctrl+C or send SIGTERM
^CReceived SIGINT from PID 12345
=== Signal Statistics ===
SIGINT:  1 received
SIGTERM: 0 received
```

### Exemple 2: Intégration avec Event Loop (poll)

```c
#include "sighandler.h"
#include <poll.h>
#include <stdio.h>
#include <unistd.h>

volatile sig_atomic_t counter = 0;

void usr1_handler(const sig_info_t *info, void *user_data) {
    int *count = (int *)user_data;
    (*count)++;
    printf("[SIGUSR1 #%d] Received at %ld.%09ld\n",
           *count, info->timestamp.tv_sec, info->timestamp.tv_nsec);
}

int main(void) {
    sighandler_t *sh = sig_create();
    int count = 0;

    sig_register(sh, SIGUSR1, usr1_handler, &count, SIG_FLAG_NONE);
    sig_register(sh, SIGTERM, NULL, NULL, SIG_FLAG_NONE);  // Ignore SIGTERM

    struct pollfd fds[2];
    fds[0].fd = STDIN_FILENO;
    fds[0].events = POLLIN;
    fds[1].fd = sig_get_fd(sh);  // Self-pipe pour les signaux
    fds[1].events = POLLIN;

    printf("Send SIGUSR1 with: kill -USR1 %d\n", getpid());

    while (count < 5) {
        int ret = poll(fds, 2, -1);
        if (ret < 0) continue;  // EINTR

        if (fds[1].revents & POLLIN) {
            sig_process(sh);
        }
        if (fds[0].revents & POLLIN) {
            char buf[256];
            if (read(STDIN_FILENO, buf, sizeof(buf)) <= 0) break;
        }
    }

    printf("Received 5 SIGUSR1 signals, exiting.\n");
    sig_destroy(sh);
    return 0;
}
```

**Sortie attendue** (avec `kill -USR1 <pid>` envoyé 5 fois):
```
Send SIGUSR1 with: kill -USR1 12345
[SIGUSR1 #1] Received at 1704326400.123456789
[SIGUSR1 #2] Received at 1704326401.234567890
[SIGUSR1 #3] Received at 1704326402.345678901
[SIGUSR1 #4] Received at 1704326403.456789012
[SIGUSR1 #5] Received at 1704326404.567890123
Received 5 SIGUSR1 signals, exiting.
```

### Exemple 3: Blocage de Signaux pendant Section Critique

```c
#include "sighandler.h"
#include <stdio.h>
#include <unistd.h>

void update_handler(const sig_info_t *info, void *user_data) {
    printf("Update signal received: %s\n", sig_name(info->signum));
}

int critical_operation(sighandler_t *sh) {
    sigset_t saved;
    int signals[] = {SIGINT, SIGTERM, SIGUSR1, 0};

    // Bloquer les signaux pendant l'opération critique
    sig_block(sh, signals, &saved);
    printf("Starting critical operation (signals blocked)...\n");

    // Simuler travail critique
    for (int i = 0; i < 5; i++) {
        printf("  Step %d/5\n", i + 1);
        sleep(1);
    }

    // Vérifier les signaux reçus pendant le blocage
    sigset_t pending;
    int npending = sig_pending(sh, &pending);
    if (npending > 0) {
        printf("  %d signal(s) were blocked during operation\n", npending);
    }

    printf("Critical operation complete, unblocking signals...\n");

    // Restaurer le masque original - les signaux seront délivrés
    sig_restore_mask(sh, &saved);

    return 0;
}

int main(void) {
    sighandler_t *sh = sig_create();

    sig_register(sh, SIGINT, update_handler, NULL, SIG_FLAG_RESTART);
    sig_register(sh, SIGUSR1, update_handler, NULL, SIG_FLAG_RESTART);

    printf("PID: %d\n", getpid());
    printf("Try sending signals during critical operation:\n");
    printf("  kill -INT %d\n", getpid());
    printf("  kill -USR1 %d\n", getpid());

    critical_operation(sh);
    sig_process(sh);

    sig_destroy(sh);
    return 0;
}
```

**Sortie attendue** (si SIGINT et SIGUSR1 envoyés pendant le blocage):
```
PID: 12345
Try sending signals during critical operation:
  kill -INT 12345
  kill -USR1 12345
Starting critical operation (signals blocked)...
  Step 1/5
  Step 2/5
  Step 3/5
  Step 4/5
  Step 5/5
  2 signal(s) were blocked during operation
Critical operation complete, unblocking signals...
Update signal received: SIGINT
Update signal received: SIGUSR1
```

### Exemple 4: Handler One-Shot et Reset

```c
#include "sighandler.h"
#include <stdio.h>
#include <unistd.h>

void panic_handler(const sig_info_t *info, void *user_data) {
    (void)user_data;
    write(STDERR_FILENO, "PANIC: First SIGUSR1 - taking emergency action!\n", 48);
}

int main(void) {
    sighandler_t *sh = sig_create();

    // Handler appelé une seule fois, puis le signal a l'action par défaut
    sig_register(sh, SIGUSR1, panic_handler, NULL,
                 SIG_FLAG_ONESHOT | SIG_FLAG_RESETHAND);

    printf("PID: %d - Send SIGUSR1 twice\n", getpid());
    printf("First SIGUSR1 will trigger handler, second will use default action.\n");

    // Premier signal
    raise(SIGUSR1);
    sig_process(sh);

    printf("Handler count: %lu\n", sig_count(sh, SIGUSR1));

    // Deuxième signal - devrait terminer le processus (action par défaut)
    printf("Sending second SIGUSR1 (will terminate)...\n");
    raise(SIGUSR1);

    printf("This line should not be printed.\n");

    sig_destroy(sh);
    return 0;
}
```

**Sortie attendue**:
```
PID: 12345 - Send SIGUSR1 twice
First SIGUSR1 will trigger handler, second will use default action.
PANIC: First SIGUSR1 - taking emergency action!
Handler count: 1
Sending second SIGUSR1 (will terminate)...
User defined signal 1
```

---

## Tests de la Moulinette

### Tests Fonctionnels de Base

#### Test 01: Création et Destruction

```yaml
description: "Création et destruction du gestionnaire"
code: |
  sighandler_t *sh = sig_create();
  assert(sh != NULL);
  sig_destroy(sh);
  sig_destroy(NULL);  // Doit être safe
expected_behavior: "PASS - Pas de crash, pas de fuites"
validation: "Valgrind clean"
```

#### Test 02: Enregistrement de Handler

```yaml
description: "Enregistrement de callbacks pour signaux standards"
code: |
  sighandler_t *sh = sig_create();
  void dummy(const sig_info_t *i, void *d) { (void)i; (void)d; }

  assert(sig_register(sh, SIGINT, dummy, NULL, 0) == 0);
  assert(sig_register(sh, SIGTERM, dummy, NULL, 0) == 0);
  assert(sig_register(sh, SIGUSR1, dummy, NULL, 0) == 0);
  assert(sig_register(sh, SIGUSR2, dummy, NULL, 0) == 0);

  // SIGKILL ne peut pas être intercepté
  assert(sig_register(sh, SIGKILL, dummy, NULL, 0) == -1);
  assert(errno == EINVAL);

  sig_destroy(sh);
expected: "PASS"
```

#### Test 03: Réception et Traitement de Signal

```yaml
description: "Réception d'un signal et exécution du callback"
code: |
  sighandler_t *sh = sig_create();
  volatile int received = 0;

  void handler(const sig_info_t *i, void *d) {
    int *r = (int *)d;
    *r = i->signum;
  }

  sig_register(sh, SIGUSR1, handler, (void *)&received, 0);

  raise(SIGUSR1);
  sig_process(sh);

  assert(received == SIGUSR1);
  sig_destroy(sh);
expected: "PASS"
```

#### Test 04: Self-Pipe Integration

```yaml
description: "Intégration avec poll() via self-pipe"
code: |
  sighandler_t *sh = sig_create();
  int count = 0;

  void counter(const sig_info_t *i, void *d) {
    (void)i;
    (*(int *)d)++;
  }
  sig_register(sh, SIGUSR1, counter, &count, 0);

  int fd = sig_get_fd(sh);
  assert(fd >= 0);

  // Envoyer 3 signaux
  raise(SIGUSR1);
  raise(SIGUSR1);
  raise(SIGUSR1);

  struct pollfd pfd = {fd, POLLIN, 0};
  assert(poll(&pfd, 1, 100) == 1);
  assert(pfd.revents & POLLIN);

  int processed = sig_process(sh);
  assert(processed == 3);
  assert(count == 3);

  sig_destroy(sh);
expected: "PASS"
```

#### Test 05: Blocage et Déblocage

```yaml
description: "Manipulation des masques de signaux"
code: |
  sighandler_t *sh = sig_create();
  volatile int received = 0;

  void handler(const sig_info_t *i, void *d) {
    (void)i;
    (*(int *)d)++;
  }
  sig_register(sh, SIGUSR1, handler, (void *)&received, 0);

  int sigs[] = {SIGUSR1, 0};
  sigset_t saved;
  sig_block(sh, sigs, &saved);

  raise(SIGUSR1);
  raise(SIGUSR1);

  // Signal bloqué, handler ne doit pas être appelé
  sig_process(sh);
  assert(received == 0);

  // Vérifier qu'il est en attente
  sigset_t pending;
  assert(sig_pending(sh, &pending) >= 1);
  assert(sigismember(&pending, SIGUSR1));

  // Débloquer
  sig_restore_mask(sh, &saved);
  sig_process(sh);

  assert(received >= 1);  // Au moins 1 (signaux peuvent être fusionnés)
  sig_destroy(sh);
expected: "PASS"
```

#### Test 06: Noms des Signaux

```yaml
description: "Conversion nom <-> numéro de signal"
code: |
  assert(strcmp(sig_name(SIGINT), "SIGINT") == 0);
  assert(strcmp(sig_name(SIGTERM), "SIGTERM") == 0);
  assert(strcmp(sig_name(SIGUSR1), "SIGUSR1") == 0);
  assert(strcmp(sig_name(9999), "UNKNOWN") == 0);

  assert(sig_number("SIGINT") == SIGINT);
  assert(sig_number("INT") == SIGINT);
  assert(sig_number("SIGTERM") == SIGTERM);
  assert(sig_number("INVALID") == -1);
expected: "PASS"
```

#### Test 07: Statistiques

```yaml
description: "Comptage des signaux reçus"
code: |
  sighandler_t *sh = sig_create();
  void dummy(const sig_info_t *i, void *d) { (void)i; (void)d; }

  sig_register(sh, SIGUSR1, dummy, NULL, 0);

  assert(sig_count(sh, SIGUSR1) == 0);

  raise(SIGUSR1);
  sig_process(sh);
  assert(sig_count(sh, SIGUSR1) == 1);

  raise(SIGUSR1);
  raise(SIGUSR1);
  sig_process(sh);
  assert(sig_count(sh, SIGUSR1) == 3);

  sig_destroy(sh);
expected: "PASS"
```

### Tests de Robustesse

#### Test 10: Paramètres Invalides

```yaml
description: "Gestion des entrées invalides"
test_cases:
  - input: "sig_create() sur système sans ressources"
    setup: "ulimit -n 4"
    expected: "Retourne NULL, errno = EMFILE"
  - input: "sig_register(NULL, SIGINT, ...)"
    expected: "Retourne -1, errno = EINVAL"
  - input: "sig_register(sh, 0, ...)"
    expected: "Retourne -1, errno = EINVAL"
  - input: "sig_register(sh, 999, ...)"
    expected: "Retourne -1, errno = EINVAL"
  - input: "sig_get_fd(NULL)"
    expected: "Retourne -1"
```

#### Test 11: Stress Test

```yaml
description: "Réception rapide de nombreux signaux"
code: |
  sighandler_t *sh = sig_create();
  volatile uint64_t count = 0;

  void counter(const sig_info_t *i, void *d) {
    (void)i;
    (*(volatile uint64_t *)d)++;
  }
  sig_register(sh, SIGUSR1, counter, (void *)&count, 0);

  // Envoyer 1000 signaux rapidement
  for (int i = 0; i < 1000; i++) {
    raise(SIGUSR1);
  }

  // Traiter tous les signaux
  int fd = sig_get_fd(sh);
  struct pollfd pfd = {fd, POLLIN, 0};
  while (poll(&pfd, 1, 10) > 0) {
    sig_process(sh);
  }

  // Note: les signaux standard peuvent être fusionnés
  // On vérifie qu'au moins quelques-uns ont été reçus
  assert(count > 0);
  assert(sig_count(sh, SIGUSR1) > 0);

  sig_destroy(sh);
expected: "PASS (count dépend de la vitesse)"
```

### Tests de Sécurité

#### Test 20: Valgrind Clean

```yaml
description: "Aucune fuite mémoire"
command: "valgrind --leak-check=full --error-exitcode=1 ./test_all"
expected: "0 bytes lost, 0 errors"
```

#### Test 21: Race Condition Check

```yaml
description: "Pas de race condition avec signaux rapides"
code: |
  // Fork un child qui envoie des signaux en boucle
  sighandler_t *sh = sig_create();
  volatile int running = 1;

  void stop(const sig_info_t *i, void *d) {
    (void)i;
    *(volatile int *)d = 0;
  }
  sig_register(sh, SIGTERM, stop, (void *)&running, 0);
  sig_register(sh, SIGUSR1, stop, NULL, 0);  // Juste compter

  pid_t child = fork();
  if (child == 0) {
    for (int i = 0; i < 100; i++) {
      kill(getppid(), SIGUSR1);
      usleep(1000);
    }
    kill(getppid(), SIGTERM);
    _exit(0);
  }

  while (running) {
    sig_process(sh);
    usleep(100);
  }

  waitpid(child, NULL, 0);
  sig_destroy(sh);
expected: "PASS - Pas de crash, pas de données corrompues"
tool: "ThreadSanitizer"
```

#### Test 22: Async-Signal-Safety

```yaml
description: "Le handler interne n'utilise que des fonctions async-signal-safe"
validation: |
  - Analyse statique du code
  - Vérifier que le handler n'appelle que write() et accède à sig_atomic_t
  - Pas de malloc, printf, exit dans le handler
expected: "Code review PASS"
```

### Tests de Performance

#### Test 30: Latence de Traitement

```yaml
description: "Mesure de la latence signal -> callback"
scenario: |
  Envoyer SIGUSR1 avec timestamp, mesurer le délai dans le callback
iterations: 1000
expected_latency: "< 100 microseconds (99th percentile)"
machine_ref: "Intel i5 2.5GHz, 8GB RAM"
```

#### Test 31: Throughput

```yaml
description: "Nombre de signaux traités par seconde"
scenario: |
  Envoyer le maximum de SIGUSR1 pendant 1 seconde
  Compter combien sont effectivement traités
expected_throughput: "> 10000 signals/second"
machine_ref: "Intel i5 2.5GHz, 8GB RAM"
```

---

## Critères d'Évaluation

### Note Minimale Requise: 80/100

### Détail de la Notation (Total: 100 points)

#### 1. Correction (40 points)

| Critère | Points | Description |
|---------|--------|-------------|
| Tests fonctionnels | 15 | Tests 01-07 passent |
| Self-pipe correct | 10 | Intégration poll/epoll fonctionne |
| Masques de signaux | 10 | Blocage/déblocage correct |
| Gestion EINTR | 5 | Toutes les opérations gèrent EINTR |

**Pénalités**:
- Handler non async-signal-safe: -15 points
- Signal perdu (non-traité): -5 points par occurrence
- Crash sur signal: -15 points

#### 2. Sécurité (25 points)

| Critère | Points | Description |
|---------|--------|-------------|
| Async-signal-safety | 10 | Handler interne 100% safe |
| Absence de fuites | 8 | Valgrind clean |
| Pas de race conditions | 7 | ThreadSanitizer clean |

**Pénalités**:
- malloc() dans handler: -10 points
- printf() dans handler: -10 points
- Fuite mémoire: -2 points par fuite

#### 3. Conception (20 points)

| Critère | Points | Description |
|---------|--------|-------------|
| Architecture self-pipe | 8 | Pattern correctement implémenté |
| API cohérente | 7 | Nommage, retours, errno |
| Extensibilité | 5 | Facile à ajouter des fonctionnalités |

#### 4. Lisibilité (15 points)

| Critère | Points | Description |
|---------|--------|-------------|
| Documentation | 6 | Chaque fonction documentée |
| Nommage | 5 | Variables et fonctions explicites |
| Organisation | 4 | Structure logique du code |

---

## Indices et Ressources

### Réflexions pour Démarrer

<details>
<summary>Question 1: Comment implémenter le self-pipe de manière thread-safe?</summary>

Le self-pipe est simple mais critique:
1. Créez un pipe avec `pipe2(fds, O_NONBLOCK | O_CLOEXEC)` si disponible
2. Le handler écrit UN octet (le numéro du signal) dans le pipe
3. Le thread principal lit tous les octets disponibles et appelle les callbacks
4. Attention: le pipe peut se remplir si les signaux arrivent plus vite qu'on ne les traite!

</details>

<details>
<summary>Question 2: Comment gérer la variable globale nécessaire pour le handler?</summary>

Vous avez besoin d'une variable globale pour que le handler de signal accède au contexte:
```c
static sighandler_t *g_handler = NULL;
```

Protégez son accès:
- Positionnez-la dans `sig_create()`
- Réinitialisez-la dans `sig_destroy()`
- N'autorisez qu'une seule instance (singleton)

</details>

<details>
<summary>Question 3: Quelles fonctions sont vraiment async-signal-safe?</summary>

Consultez `man 7 signal-safety`. Les plus utiles:
- `write()` - Votre meilleur ami
- `_Exit()` - Pas `exit()`!
- Accès à `volatile sig_atomic_t`
- Les fonctions de signaux elles-mêmes

**JAMAIS** dans un handler: `malloc`, `free`, `printf`, `syslog`, `localtime`...

</details>

### Ressources Recommandées

#### Documentation
- `man 7 signal` - Vue d'ensemble des signaux
- `man 7 signal-safety` - Liste des fonctions async-signal-safe
- `man 2 sigaction` - API moderne de gestion de signaux
- `man 2 sigprocmask` - Manipulation des masques

#### Lectures Complémentaires
- "The Linux Programming Interface", ch. 20-22 (M. Kerrisk)
- "Advanced Programming in the UNIX Environment", ch. 10 (Stevens)
- GNU libc Manual, section "Signal Handling"

### Pièges Fréquents

1. **Appeler des fonctions non-safe dans le handler**
   - **Solution**: N'utilisez QUE write() et sig_atomic_t dans le handler

2. **Oublier que les signaux standard ne s'accumulent pas**
   - **Solution**: Si le même signal arrive 3 fois avant traitement, un seul sera délivré

3. **Race condition entre sig_destroy() et un signal entrant**
   - **Solution**: Bloquez tous les signaux avant de détruire

4. **Pipe plein si signaux trop rapides**
   - **Solution**: Pipe non-bloquant + ignorer EAGAIN dans le handler (log le problème ailleurs)

5. **Oublier de gérer EINTR sur read()/poll()**
   - **Solution**: Boucle while(ret < 0 && errno == EINTR)

---

## Notes du Concepteur

<details>
<summary>Solution de Référence (Concepteur uniquement)</summary>

**Approche recommandée**:
1. Structure avec tableau de 64 entrées (une par signal possible)
2. Chaque entrée: callback, user_data, flags, compteur volatile
3. Self-pipe créé dans sig_create(), fermé dans sig_destroy()
4. Un seul handler interne pour tous les signaux, dispatch via lookup

**Complexité**:
- Temps: O(1) pour register/unregister, O(n) pour process
- Espace: O(1) + O(signaux en attente dans le pipe)

**Points d'attention**:
- Le handler interne DOIT être minimal: juste un write()
- Utiliser sigaction() avec SA_SIGINFO pour obtenir les infos étendues
- sig_process() doit lire en boucle jusqu'à EAGAIN

**Variantes possibles**:
- Utiliser signalfd() au lieu du self-pipe (Linux only)
- Utiliser kqueue() pour BSD
- Pool de handlers pour différents threads

</details>

<details>
<summary>Grille d'Évaluation - Points d'Attention</summary>

**Lors de la correction manuelle, vérifier**:
- [ ] Le handler interne n'appelle que write()
- [ ] Les compteurs sont volatile sig_atomic_t
- [ ] SIGKILL et SIGSTOP retournent EINVAL
- [ ] Le pipe est O_NONBLOCK et O_CLOEXEC
- [ ] sig_destroy() restaure les handlers par défaut
- [ ] Pas de malloc/free dans le handler

**Erreurs fréquentes observées**:
- printf() dans le handler "pour debugger"
- Oubli de vérifier le retour de sigaction()
- Pipe bloquant causant un deadlock
- Fuite du fd du self-pipe

</details>

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 24 | Self-pipe pattern, async-safety, event loop integration |
| Couverture conceptuelle | 25 | 17 concepts (2.2.10-2.2.12, 2.2.15) tous couverts |
| Testabilité auto | 24 | Tests objectifs, Valgrind, TSan, mesures de latence |
| Originalité | 23 | Bibliothèque complète vs simple wrapper sigaction |
| **TOTAL** | **96/100** | ✓ Validé |

---

## Historique

```yaml
version: "1.0"
created: "2025-01-04"
author: "ODYSSEY Curriculum Team"
last_modified: "2025-01-04"
changes:
  - "Version initiale - Exercice ex04 Signal Handler Pro"
```

---

*Template ODYSSEY Phase 2 - Module 2.2 Processes & Shell*
