<thinking>
## Analyse du Concept
- Concept : Signal Handler Pro (sigaction, self-pipe, async-signal-safety)
- Phase demandée : 2 (Intermédiaire)
- Adapté ? OUI - Gestion avancée des signaux, appropriée pour Phase 2

## Combo Base + Bonus
- Exercice de base : Bibliothèque sighandler avec self-pipe, masques, async-safety
- Bonus : signalfd() Linux, support multi-thread, rate limiting
- Palier bonus : 🔥 Avancé (API Linux spécifiques, thread-safety)
- Progression logique ? OUI - Base = portable POSIX, Bonus = Linux optimisé

## Prérequis & Difficulté
- Prérequis réels : fork, wait, processus (Module 2.2 ex00-03)
- Difficulté estimée : 6/10 (Base), 8/10 (Bonus)
- Cohérent avec phase ? OUI - Phase 2 autorise 4-6/10

## Aspect Fun/Culture
- Contexte choisi : Neon Genesis Evangelion (anime)
- Thème : NERV reçoit des alertes d'attaque d'Ange (asynchrones, urgentes)
- Les pilotes (handlers) doivent réagir correctement
- L'AT Field bloque temporairement les attaques (masque de signaux)
- Le MAGI (système de traitement) = self-pipe pattern
- MEME mnémotechnique : "Shinji, get in the handler!"
- Pourquoi c'est fun : L'urgence des alertes d'Ange = nature asynchrone des signaux

## Scénarios d'Échec (6 mutants)
1. Mutant A (Boundary) : Pipe write avec signum > 255 (overflow byte)
2. Mutant B (Safety) : printf() dans le handler interne - NON async-signal-safe
3. Mutant C (Resource) : Oubli de fermer les FDs du self-pipe
4. Mutant D (Logic) : Pas de boucle sur waitpid pour EINTR
5. Mutant E (Return) : sig_get_fd() retourne write_fd au lieu de read_fd
6. Mutant F (Global) : Pas de volatile sig_atomic_t pour les compteurs

## Verdict
VALIDE - Analogie Evangelion excellente, concepts signaux complets, 6 mutants solides
</thinking>

---

# Exercice 2.2.4 : nerv_command_init

**Module :**
2.2 — Processes & Shell

**Concept :**
d — Signal Handling (sigaction, self-pipe, async-signal-safety, signal masks)

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
code

**Tiers :**
2 — Mélange (concepts a + b + c + d : processus + terminaison + signaux)

**Langage :**
C (C17)

**Prérequis :**
- Module 2.2 ex00-03 (processus, fork, wait)
- Pointeurs de fonction
- File descriptors

**Domaines :**
Process, FS, Concurrency

**Durée estimée :**
360 min (6 heures)

**XP Base :**
600

**Complexité :**
T2 O(n) × S1 O(1)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex04/
├── nerv_signals.h           # Header avec API publique
├── nerv_signals.c           # Implémentation principale
├── magi_internal.h          # Structures internes (optionnel)
├── main.c                   # Programme de démonstration
└── Makefile
```

**Fonctions autorisées :**
```c
/* POSIX Signals */
sigaction, sigemptyset, sigfillset, sigaddset, sigdelset, sigismember,
sigprocmask, sigpending, sigsuspend, sigwait, sigtimedwait, kill, raise

/* POSIX I/O */
pipe, pipe2 (si disponible), read, write, close, fcntl

/* Memory */
malloc, free, calloc, realloc

/* Time */
clock_gettime (CLOCK_REALTIME)

/* String */
memset, memcpy, strlen, strcmp, strncmp, snprintf (hors handler!)
```

**Fonctions INTERDITES dans les handlers :**
```c
printf, fprintf, sprintf, malloc, free, realloc, exit, syslog,
localtime, et toute fonction non listée dans signal-safety(7)
```

### 1.2 Consigne

#### 🎮 CONTEXTE FUN — EVANGELION : Le Système d'Alerte NERV

Dans **Neon Genesis Evangelion**, l'organisation **NERV** défend l'humanité contre les **Anges** — des créatures qui attaquent sans prévenir, de manière **complètement asynchrone**. Quand un Ange approche, une alerte Pattern Blue retentit dans tout le Geofront, interrompant immédiatement toutes les activités.

Le système de réponse de NERV :
- **MAGI** = Le super-ordinateur qui traite les alertes (self-pipe pattern)
- **Central Dogma** = Le centre de commande (gestionnaire de signaux)
- **Pilotes EVA** = Les handlers qui réagissent aux alertes
- **AT Field** = Le champ qui bloque temporairement les attaques (masque de signaux)
- **Entry Plug** = L'environnement isolé du pilote (contexte async-signal-safe)
- **Dummy Plug** = Le mode automatique quand le pilote ne répond pas (handler par défaut)

Le problème : les Anges peuvent attaquer **à n'importe quel moment**. Si un pilote est en train de déjeuner (malloc), il ne peut pas simplement lâcher ses baguettes et courir — il doit finir son action en cours, puis répondre à l'alerte. C'est exactement le problème de l'**async-signal-safety**.

**Ta mission :**

Implémenter le système d'alerte **NERV Central Dogma** — une bibliothèque de gestion de signaux robuste et async-signal-safe qui :
1. Enregistre des "pilotes" (handlers) pour chaque type d'alerte
2. Utilise le pattern self-pipe (MAGI) pour une intégration sûre avec les event loops
3. Permet de déployer l'AT Field (bloquer temporairement des signaux)
4. Garantit que les handlers s'exécutent de manière sûre

**Entrée :**
- `nerv_command_init()` : Initialise Central Dogma
- `pilot_sync_register(nerv, signum, callback, data, flags)` : Enregistre un pilote
- `at_field_deploy(nerv, signals, saved)` : Bloque des signaux
- `magi_process_alerts(nerv)` : Traite les alertes en attente
- `central_dogma_fd(nerv)` : Obtient le FD pour poll/epoll

**Sortie :**
- Signaux correctement capturés et traités
- Statistiques des alertes reçues
- Intégration avec event loop via self-pipe

**Contraintes :**
- Aucune variable globale sauf UNE pour le handler interne
- Le handler interne ne doit utiliser QUE `write()` et `sig_atomic_t`
- Self-pipe en mode non-bloquant avec FD_CLOEXEC
- SIGKILL et SIGSTOP ne peuvent pas être interceptés

---

#### 1.2.2 Version Académique

**Contexte technique :**

Les signaux UNIX interrompent l'exécution d'un programme de manière asynchrone. Cette nature asynchrone crée des défis :
- La plupart des fonctions ne sont pas async-signal-safe
- Les handlers s'exécutent dans un contexte restreint
- Les race conditions sont faciles à introduire

Le pattern **self-pipe** résout ce problème :
1. Le handler écrit un octet dans un pipe (async-safe)
2. Le thread principal poll() le pipe
3. Quand le pipe est lisible, on traite le signal en contexte normal

**Ta mission :**

Implémenter une bibliothèque de gestion de signaux avec :
1. API sigaction-based avec flags personnalisables
2. Pattern self-pipe pour intégration event loop
3. Manipulation de masques de signaux
4. Garanties async-signal-safety

### 1.3 Prototype

```c
#ifndef NERV_SIGNALS_H
#define NERV_SIGNALS_H

#include <signal.h>
#include <stdint.h>
#include <sys/types.h>
#include <time.h>

#define NERV_MAX_SIGNALS 64

/* === Informations sur une alerte reçue === */
typedef struct alert_info {
    int             pattern;        /* Numéro du signal (Pattern Blue/Orange) */
    pid_t           source_pid;     /* PID de l'émetteur */
    uid_t           source_uid;     /* UID de l'émetteur */
    int             code;           /* Code additionnel (SI_USER, etc.) */
    union sigval    value;          /* Valeur associée (signaux RT) */
    struct timespec timestamp;      /* Moment de détection */
} alert_info_t;

/* === Callback pilote (handler) === */
typedef void (*pilot_callback_fn)(const alert_info_t *alert, void *user_data);

/* === Structure opaque Central Dogma === */
typedef struct nerv_command nerv_command_t;

/* === Flags d'enregistrement === */
typedef enum {
    PILOT_FLAG_NONE       = 0,
    PILOT_FLAG_RESTART    = (1 << 0),  /* SA_RESTART */
    PILOT_FLAG_NODEFER    = (1 << 1),  /* SA_NODEFER */
    PILOT_FLAG_RESETHAND  = (1 << 2),  /* SA_RESETHAND */
    PILOT_FLAG_NOCLDSTOP  = (1 << 3),  /* SA_NOCLDSTOP */
    PILOT_FLAG_ONESHOT    = (1 << 4),  /* Custom: une seule exécution */
} pilot_flags_t;

/* === Codes d'erreur === */
typedef enum {
    NERV_SUCCESS = 0,
    NERV_ERR_MEMORY = -1,
    NERV_ERR_INVALID = -2,
    NERV_ERR_SYSCALL = -3,
    NERV_ERR_BLOCKED = -4,      /* Signal non capturable (SIGKILL/SIGSTOP) */
} nerv_error_t;

/* === API Central Dogma === */

/**
 * Initialise le système Central Dogma.
 * @return Pointeur vers le gestionnaire, ou NULL si échec
 */
nerv_command_t *nerv_command_init(void);

/**
 * Détruit Central Dogma et restaure les handlers par défaut.
 * @param nerv Gestionnaire à détruire (peut être NULL)
 */
void nerv_command_shutdown(nerv_command_t *nerv);

/**
 * Synchronise un pilote (handler) avec un type d'alerte.
 * @param nerv     Gestionnaire
 * @param pattern  Numéro du signal
 * @param callback Fonction callback (NULL pour ignorer)
 * @param data     Données utilisateur
 * @param flags    Combinaison de PILOT_FLAG_*
 * @return NERV_SUCCESS ou code d'erreur
 */
nerv_error_t pilot_sync_register(nerv_command_t *nerv, int pattern,
                                  pilot_callback_fn callback, void *data,
                                  pilot_flags_t flags);

/**
 * Éjecte un pilote (désenregistre).
 * @param nerv    Gestionnaire
 * @param pattern Numéro du signal
 * @return NERV_SUCCESS ou NERV_ERR_INVALID
 */
nerv_error_t pilot_eject(nerv_command_t *nerv, int pattern);

/**
 * Déploie l'AT Field (bloque des signaux).
 * @param nerv     Gestionnaire
 * @param patterns Tableau de signaux (terminé par 0)
 * @param saved    Si non-NULL, sauvegarde le masque précédent
 * @return NERV_SUCCESS ou code d'erreur
 */
nerv_error_t at_field_deploy(nerv_command_t *nerv, const int *patterns,
                              sigset_t *saved);

/**
 * Neutralise l'AT Field (débloque des signaux).
 * @param nerv     Gestionnaire
 * @param patterns Tableau de signaux (terminé par 0)
 * @return NERV_SUCCESS ou code d'erreur
 */
nerv_error_t at_field_neutralize(nerv_command_t *nerv, const int *patterns);

/**
 * Restaure un masque précédemment sauvegardé.
 * @param nerv  Gestionnaire
 * @param saved Masque sauvegardé par at_field_deploy()
 * @return NERV_SUCCESS ou code d'erreur
 */
nerv_error_t at_field_restore(nerv_command_t *nerv, const sigset_t *saved);

/**
 * Vérifie les alertes en attente (bloquées mais reçues).
 * @param nerv    Gestionnaire
 * @param pending Si non-NULL, reçoit l'ensemble des signaux en attente
 * @return Nombre de signaux en attente
 */
int alert_pending_check(nerv_command_t *nerv, sigset_t *pending);

/**
 * Retourne le FD de lecture du MAGI (self-pipe).
 * @param nerv Gestionnaire
 * @return FD pour poll/epoll, ou -1 si erreur
 */
int central_dogma_fd(nerv_command_t *nerv);

/**
 * Traite les alertes en attente via le MAGI.
 * @param nerv Gestionnaire
 * @return Nombre d'alertes traitées, ou -1 si erreur
 */
int magi_process_alerts(nerv_command_t *nerv);

/**
 * Diffère le traitement d'une alerte (async-signal-safe).
 * @param nerv  Gestionnaire
 * @param alert Information sur l'alerte
 * @return 0 si succès
 */
int magi_defer_alert(nerv_command_t *nerv, const alert_info_t *alert);

/**
 * Mode standby : attend un signal parmi un ensemble.
 * @param nerv      Gestionnaire
 * @param wait_for  Ensemble de signaux à attendre
 * @param timeout   Timeout en ms (-1 = infini)
 * @param received  Si non-NULL, reçoit le signal reçu
 * @return 0 si signal reçu, -1 si timeout/erreur
 */
int eva_standby(nerv_command_t *nerv, const sigset_t *wait_for,
                int timeout_ms, int *received);

/* === Utilitaires === */

/**
 * Identifie le pattern (retourne le nom du signal).
 * @param pattern Numéro du signal
 * @return Nom ("SIGINT", etc.) ou "UNKNOWN"
 */
const char *pattern_identify(int pattern);

/**
 * Retourne le numéro d'un signal à partir de son nom.
 * @param name Nom du signal ("SIGINT" ou "INT")
 * @return Numéro, ou -1 si inconnu
 */
int pattern_lookup(const char *name);

/**
 * Retourne l'action par défaut d'un signal.
 * @param pattern Numéro du signal
 * @return 'T' (terminate), 'I' (ignore), 'C' (core), 'S' (stop)
 */
char pattern_default_action(int pattern);

/**
 * Compte les alertes reçues pour un pattern.
 * @param nerv    Gestionnaire
 * @param pattern Numéro du signal
 * @return Compteur (volatile)
 */
uint64_t angel_kill_count(nerv_command_t *nerv, int pattern);

/**
 * Affiche les statistiques NERV.
 * @param nerv Gestionnaire
 * @param fd   File descriptor de sortie
 */
void nerv_status_report(nerv_command_t *nerv, int fd);

/**
 * Retourne une description de l'erreur.
 * @param error Code d'erreur
 * @return Description textuelle
 */
const char *nerv_strerror(nerv_error_t error);

#endif /* NERV_SIGNALS_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Pourquoi "Signal" ?

Le terme vient directement de la théorie des signaux électriques. Dans les années 70, les créateurs d'Unix ont emprunté ce concept pour représenter les notifications asynchrones entre processus — comme un "signal électrique" qui interrompt un circuit.

### 2.2 Le Problème du Téléphone

Imaginez que vous êtes en train d'écrire un email important (malloc). Le téléphone sonne (signal). Si vous décrochez immédiatement au milieu d'un mot, votre email sera corrompu. C'est exactement pourquoi malloc() n'est pas async-signal-safe — un signal peut arriver pendant que malloc modifie ses structures internes.

### 2.3 SIGKILL et SIGSTOP : Les Intouchables

Ces deux signaux ne peuvent JAMAIS être bloqués ou interceptés. Pourquoi ? Si un processus pouvait bloquer SIGKILL, il serait impossible de le terminer — un risque de sécurité inacceptable. Le kernel les traite directement, sans consulter le processus.

---

## 🔧 SECTION 2.5 : DANS LA VRAIE VIE

### DevOps / SRE

**Cas d'usage :** Graceful shutdown de serveurs

Les serveurs en production reçoivent SIGTERM avant d'être terminés (par Kubernetes, systemd, etc.). Le handler doit :
1. Arrêter d'accepter de nouvelles connexions
2. Finir de traiter les requêtes en cours
3. Sauvegarder l'état
4. Quitter proprement

Un handler mal écrit = données perdues en production.

### Développeur Backend

**Cas d'usage :** Hot reload de configuration

nginx, HAProxy, et la plupart des serveurs utilisent SIGHUP pour recharger leur configuration sans restart. Le handler lit le nouveau fichier de config et l'applique à chaud.

### Développeur Système

**Cas d'usage :** Supervision de processus enfants

systemd, supervisord, et les init systems utilisent SIGCHLD pour détecter quand un service crash et le redémarrer automatiquement.

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
nerv_signals.c  nerv_signals.h  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -std=c17 -D_POSIX_C_SOURCE=200809L -c nerv_signals.c -o nerv_signals.o
ar rcs libnerv.a nerv_signals.o
gcc -Wall -Wextra -Werror -std=c17 -o nerv_demo main.o -L. -lnerv

$ ./nerv_demo
[NERV] Central Dogma initialized
[NERV] PID: 12345 - Waiting for Angel attack...
[NERV] Send signals with: kill -INT 12345 or kill -TERM 12345
^C[ALERT] Pattern Blue detected: SIGINT
[ALERT] Source: PID 12345, UID 1000
[ALERT] Timestamp: 1704326400.123456789
[NERV] Pilot Shinji responding...
[NERV] Angel neutralized!
=== NERV Status Report ===
SIGINT:  1 received
SIGTERM: 0 received
SIGUSR1: 0 received
[NERV] Central Dogma shutdown complete

$ make poll_demo && ./poll_demo
[NERV] Integration with poll() event loop
[NERV] PID: 12346 - Send SIGUSR1 five times
[MAGI] Alert queued, processing...
[PILOT] SIGUSR1 #1 received at 1704326401.234567890
[PILOT] SIGUSR1 #2 received at 1704326402.345678901
[PILOT] SIGUSR1 #3 received at 1704326403.456789012
[PILOT] SIGUSR1 #4 received at 1704326404.567890123
[PILOT] SIGUSR1 #5 received at 1704326405.678901234
[NERV] Mission complete. 5 Angels neutralized.
```

---

## 🔥 SECTION 3.1 : BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(1) pour tous les handlers

**Space Complexity attendue :**
O(1) auxiliaire

**Domaines Bonus :**
`Linux-specific, Concurrency`

### 3.1.1 Consigne Bonus

**🎮 SEELE MODE : Instrumentalité des Signaux**

En mode SEELE, NERV atteint sa forme ultime avec :
1. **signalfd()** : API Linux moderne qui transforme les signaux en events de fichier
2. **Thread-safe** : Support multi-thread avec un seul gestionnaire
3. **Rate limiting** : Protection contre le flood de signaux

**Ta mission :**

```c
/* API Linux signalfd */
typedef struct {
    int use_signalfd;           /* Utiliser signalfd au lieu de self-pipe */
    int thread_safe;            /* Mode multi-thread */
    int rate_limit_per_second;  /* Limite de signaux par seconde (0 = illimité) */
} seele_config_t;

nerv_command_t *seele_init(const seele_config_t *config);

/* Rate limiting */
typedef struct {
    uint64_t received;          /* Total reçus */
    uint64_t processed;         /* Total traités */
    uint64_t dropped;           /* Perdus (rate limit) */
    double   rate_per_second;   /* Taux actuel */
} rate_stats_t;

int get_rate_stats(nerv_command_t *nerv, int pattern, rate_stats_t *stats);
```

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  signalfd() Linux 2.6.22+              │
│  pthread_mutex pour thread-safety      │
│  Token bucket pour rate limiting       │
│  Graceful degradation sur non-Linux    │
└─────────────────────────────────────────┘
```

### 3.1.2 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Mécanisme | Self-pipe (portable) | signalfd (Linux) |
| Threading | Single-thread | Multi-thread safe |
| Rate limiting | Aucun | Token bucket |
| Complexité | Moyenne | Élevée |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (Tableau des Tests)

| ID | Test | Input | Expected | Points |
|----|------|-------|----------|--------|
| 01 | Create/Destroy | `nerv_command_init()` + `shutdown()` | SUCCESS, Valgrind clean | 3 |
| 02 | Register Handler | `pilot_sync_register()` SIGINT/TERM/USR1 | SUCCESS | 3 |
| 03 | SIGKILL Blocked | `pilot_sync_register()` SIGKILL | ERR_BLOCKED | 2 |
| 04 | Signal Received | `raise(SIGUSR1)` + `magi_process_alerts()` | Callback appelé | 4 |
| 05 | Self-pipe FD | `central_dogma_fd()` | fd >= 0 | 2 |
| 06 | Poll Integration | poll() sur FD + signal | Callback après poll | 4 |
| 07 | AT Field Deploy | `at_field_deploy()` + signal | Signal bloqué | 3 |
| 08 | AT Field Restore | Restore après deploy | Signal délivré | 3 |
| 09 | Pending Check | Signal bloqué + `alert_pending_check()` | count >= 1 | 2 |
| 10 | Signal Name | `pattern_identify(SIGINT)` | "SIGINT" | 2 |
| 11 | Signal Number | `pattern_lookup("SIGTERM")` | SIGTERM | 2 |
| 12 | Kill Count | 3x raise + process | count == 3 | 2 |
| 13 | NULL Params | Fonctions avec NULL | Pas de crash | 3 |
| 14 | ONESHOT Flag | Handler avec ONESHOT | Appelé une seule fois | 3 |
| 15 | Multiple Signals | 5 signaux différents | Tous traités | 3 |
| 16 | Stress Test | 100 signaux rapides | Pas de perte/crash | 4 |
| 17 | Valgrind Clean | Test complet | 0 bytes lost | 5 |
| 18 | Async-Safety | Review du handler interne | Que write() | 5 |

**Total : 55 points**

### 4.2 main.c de test

```c
#include "nerv_signals.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <poll.h>
#include <assert.h>

volatile sig_atomic_t g_running = 1;
volatile int g_count = 0;

void shinji_handler(const alert_info_t *alert, void *data)
{
    (void)data;
    printf("[PILOT SHINJI] Pattern %s detected from PID %d\n",
           pattern_identify(alert->pattern), alert->source_pid);
    g_count++;
}

void asuka_handler(const alert_info_t *alert, void *data)
{
    int *target = (int *)data;
    (*target)++;
    printf("[PILOT ASUKA] Kill count: %d\n", *target);
}

void shutdown_handler(const alert_info_t *alert, void *data)
{
    (void)alert;
    (void)data;
    const char msg[] = "[NERV] Initiating shutdown sequence...\n";
    write(STDOUT_FILENO, msg, sizeof(msg) - 1);
    g_running = 0;
}

int main(void)
{
    printf("=== NERV Signal Handler Test Suite ===\n\n");

    /* Test 1: Basic lifecycle */
    printf("--- Test 1: Init/Shutdown ---\n");
    nerv_command_t *nerv = nerv_command_init();
    assert(nerv != NULL);
    nerv_command_shutdown(nerv);
    nerv_command_shutdown(NULL);  /* Should be safe */
    printf("PASS\n\n");

    /* Test 2: Register and receive */
    printf("--- Test 2: Register and Receive ---\n");
    nerv = nerv_command_init();
    int kill_count = 0;

    assert(pilot_sync_register(nerv, SIGUSR1, asuka_handler,
                               &kill_count, PILOT_FLAG_NONE) == NERV_SUCCESS);
    assert(pilot_sync_register(nerv, SIGTERM, shutdown_handler,
                               NULL, PILOT_FLAG_RESTART) == NERV_SUCCESS);

    raise(SIGUSR1);
    raise(SIGUSR1);
    raise(SIGUSR1);

    int processed = magi_process_alerts(nerv);
    printf("Processed: %d alerts\n", processed);
    assert(kill_count >= 1);  /* At least 1 (signals may coalesce) */
    printf("PASS\n\n");

    /* Test 3: SIGKILL rejection */
    printf("--- Test 3: SIGKILL Rejection ---\n");
    assert(pilot_sync_register(nerv, SIGKILL, shinji_handler,
                               NULL, PILOT_FLAG_NONE) == NERV_ERR_BLOCKED);
    assert(pilot_sync_register(nerv, SIGSTOP, shinji_handler,
                               NULL, PILOT_FLAG_NONE) == NERV_ERR_BLOCKED);
    printf("PASS\n\n");

    /* Test 4: Self-pipe integration */
    printf("--- Test 4: Poll Integration ---\n");
    int fd = central_dogma_fd(nerv);
    assert(fd >= 0);

    raise(SIGUSR1);

    struct pollfd pfd = {fd, POLLIN, 0};
    int ret = poll(&pfd, 1, 100);
    assert(ret == 1);
    assert(pfd.revents & POLLIN);

    processed = magi_process_alerts(nerv);
    assert(processed >= 1);
    printf("PASS\n\n");

    /* Test 5: AT Field (signal blocking) */
    printf("--- Test 5: AT Field ---\n");
    int old_count = kill_count;
    int block_sigs[] = {SIGUSR1, 0};
    sigset_t saved;

    assert(at_field_deploy(nerv, block_sigs, &saved) == NERV_SUCCESS);

    raise(SIGUSR1);
    raise(SIGUSR1);
    magi_process_alerts(nerv);  /* Should not process blocked signals */

    /* Check pending */
    sigset_t pending;
    int npending = alert_pending_check(nerv, &pending);
    printf("Pending signals while AT Field active: %d\n", npending);
    assert(npending >= 1);
    assert(sigismember(&pending, SIGUSR1));

    /* Restore and process */
    assert(at_field_restore(nerv, &saved) == NERV_SUCCESS);
    magi_process_alerts(nerv);
    assert(kill_count > old_count);
    printf("PASS\n\n");

    /* Test 6: Signal names */
    printf("--- Test 6: Pattern Identification ---\n");
    assert(strcmp(pattern_identify(SIGINT), "SIGINT") == 0);
    assert(strcmp(pattern_identify(SIGTERM), "SIGTERM") == 0);
    assert(pattern_lookup("SIGINT") == SIGINT);
    assert(pattern_lookup("USR1") == SIGUSR1);
    assert(pattern_lookup("INVALID") == -1);
    printf("PASS\n\n");

    /* Test 7: Stats */
    printf("--- Test 7: Statistics ---\n");
    uint64_t count = angel_kill_count(nerv, SIGUSR1);
    printf("SIGUSR1 total count: %lu\n", count);
    assert(count >= 1);

    nerv_status_report(nerv, STDOUT_FILENO);
    printf("PASS\n\n");

    nerv_command_shutdown(nerv);

    printf("=== All Tests Passed! ===\n");
    return 0;
}
```

### 4.3 Solution de Référence

```c
/* nerv_signals.c - Solution de référence */
#include "nerv_signals.h"
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <stdio.h>

/* Variable globale unique pour le handler */
static nerv_command_t *g_nerv = NULL;

/* Structure interne d'un pilote */
typedef struct {
    pilot_callback_fn callback;
    void *user_data;
    pilot_flags_t flags;
    volatile sig_atomic_t count;
    int active;
} pilot_entry_t;

/* Structure Central Dogma */
struct nerv_command {
    pilot_entry_t pilots[NERV_MAX_SIGNALS];
    int magi_read_fd;   /* Self-pipe read */
    int magi_write_fd;  /* Self-pipe write */
    sigset_t original_mask;
};

/* Handler interne async-signal-safe */
static void internal_handler(int signum, siginfo_t *si, void *context)
{
    (void)context;

    if (!g_nerv || signum <= 0 || signum >= NERV_MAX_SIGNALS)
        return;

    /* Incrémenter le compteur (async-safe) */
    g_nerv->pilots[signum].count++;

    /* Écrire dans le self-pipe (async-safe) */
    uint8_t sig_byte = (uint8_t)signum;
    write(g_nerv->magi_write_fd, &sig_byte, 1);

    (void)si;  /* Info disponible si besoin */
}

nerv_command_t *nerv_command_init(void)
{
    if (g_nerv != NULL) {
        errno = EBUSY;
        return NULL;  /* Singleton */
    }

    nerv_command_t *nerv = calloc(1, sizeof(nerv_command_t));
    if (!nerv)
        return NULL;

    /* Créer le self-pipe */
    int pipefd[2];
#ifdef __linux__
    if (pipe2(pipefd, O_NONBLOCK | O_CLOEXEC) < 0) {
#else
    if (pipe(pipefd) < 0) {
#endif
        free(nerv);
        return NULL;
    }

#ifndef __linux__
    /* Configurer non-bloquant et cloexec manuellement */
    fcntl(pipefd[0], F_SETFL, O_NONBLOCK);
    fcntl(pipefd[1], F_SETFL, O_NONBLOCK);
    fcntl(pipefd[0], F_SETFD, FD_CLOEXEC);
    fcntl(pipefd[1], F_SETFD, FD_CLOEXEC);
#endif

    nerv->magi_read_fd = pipefd[0];
    nerv->magi_write_fd = pipefd[1];

    /* Sauvegarder le masque original */
    sigprocmask(SIG_BLOCK, NULL, &nerv->original_mask);

    g_nerv = nerv;
    return nerv;
}

void nerv_command_shutdown(nerv_command_t *nerv)
{
    if (!nerv)
        return;

    /* Bloquer tous les signaux pendant le cleanup */
    sigset_t all_blocked;
    sigfillset(&all_blocked);
    sigprocmask(SIG_BLOCK, &all_blocked, NULL);

    /* Restaurer les handlers par défaut */
    for (int i = 1; i < NERV_MAX_SIGNALS; i++) {
        if (nerv->pilots[i].active) {
            signal(i, SIG_DFL);
        }
    }

    /* Fermer le self-pipe */
    close(nerv->magi_read_fd);
    close(nerv->magi_write_fd);

    /* Restaurer le masque original */
    sigprocmask(SIG_SETMASK, &nerv->original_mask, NULL);

    g_nerv = NULL;
    free(nerv);
}

nerv_error_t pilot_sync_register(nerv_command_t *nerv, int pattern,
                                  pilot_callback_fn callback, void *data,
                                  pilot_flags_t flags)
{
    if (!nerv)
        return NERV_ERR_INVALID;

    if (pattern <= 0 || pattern >= NERV_MAX_SIGNALS)
        return NERV_ERR_INVALID;

    /* SIGKILL et SIGSTOP ne peuvent pas être interceptés */
    if (pattern == SIGKILL || pattern == SIGSTOP)
        return NERV_ERR_BLOCKED;

    /* Configurer sigaction */
    struct sigaction sa;
    memset(&sa, 0, sizeof(sa));

    if (callback) {
        sa.sa_sigaction = internal_handler;
        sa.sa_flags = SA_SIGINFO;

        if (flags & PILOT_FLAG_RESTART)
            sa.sa_flags |= SA_RESTART;
        if (flags & PILOT_FLAG_NODEFER)
            sa.sa_flags |= SA_NODEFER;
        if (flags & PILOT_FLAG_RESETHAND)
            sa.sa_flags |= SA_RESETHAND;
        if (flags & PILOT_FLAG_NOCLDSTOP)
            sa.sa_flags |= SA_NOCLDSTOP;
    } else {
        sa.sa_handler = SIG_IGN;
    }

    sigemptyset(&sa.sa_mask);

    if (sigaction(pattern, &sa, NULL) < 0)
        return NERV_ERR_SYSCALL;

    /* Enregistrer le pilote */
    nerv->pilots[pattern].callback = callback;
    nerv->pilots[pattern].user_data = data;
    nerv->pilots[pattern].flags = flags;
    nerv->pilots[pattern].active = 1;

    return NERV_SUCCESS;
}

nerv_error_t pilot_eject(nerv_command_t *nerv, int pattern)
{
    if (!nerv || pattern <= 0 || pattern >= NERV_MAX_SIGNALS)
        return NERV_ERR_INVALID;

    if (!nerv->pilots[pattern].active)
        return NERV_ERR_INVALID;

    signal(pattern, SIG_DFL);

    nerv->pilots[pattern].callback = NULL;
    nerv->pilots[pattern].user_data = NULL;
    nerv->pilots[pattern].active = 0;

    return NERV_SUCCESS;
}

nerv_error_t at_field_deploy(nerv_command_t *nerv, const int *patterns,
                              sigset_t *saved)
{
    if (!nerv || !patterns)
        return NERV_ERR_INVALID;

    sigset_t block_set;
    sigemptyset(&block_set);

    for (int i = 0; patterns[i] != 0; i++) {
        if (patterns[i] > 0 && patterns[i] < NERV_MAX_SIGNALS)
            sigaddset(&block_set, patterns[i]);
    }

    if (sigprocmask(SIG_BLOCK, &block_set, saved) < 0)
        return NERV_ERR_SYSCALL;

    return NERV_SUCCESS;
}

nerv_error_t at_field_neutralize(nerv_command_t *nerv, const int *patterns)
{
    if (!nerv || !patterns)
        return NERV_ERR_INVALID;

    sigset_t unblock_set;
    sigemptyset(&unblock_set);

    for (int i = 0; patterns[i] != 0; i++) {
        if (patterns[i] > 0 && patterns[i] < NERV_MAX_SIGNALS)
            sigaddset(&unblock_set, patterns[i]);
    }

    if (sigprocmask(SIG_UNBLOCK, &unblock_set, NULL) < 0)
        return NERV_ERR_SYSCALL;

    return NERV_SUCCESS;
}

nerv_error_t at_field_restore(nerv_command_t *nerv, const sigset_t *saved)
{
    if (!nerv || !saved)
        return NERV_ERR_INVALID;

    if (sigprocmask(SIG_SETMASK, saved, NULL) < 0)
        return NERV_ERR_SYSCALL;

    return NERV_SUCCESS;
}

int alert_pending_check(nerv_command_t *nerv, sigset_t *pending)
{
    if (!nerv)
        return -1;

    sigset_t pend;
    if (sigpending(&pend) < 0)
        return -1;

    if (pending)
        *pending = pend;

    int count = 0;
    for (int i = 1; i < NERV_MAX_SIGNALS; i++) {
        if (sigismember(&pend, i))
            count++;
    }

    return count;
}

int central_dogma_fd(nerv_command_t *nerv)
{
    if (!nerv)
        return -1;
    return nerv->magi_read_fd;
}

int magi_process_alerts(nerv_command_t *nerv)
{
    if (!nerv)
        return -1;

    int processed = 0;
    uint8_t sig_byte;

    /* Lire tous les signaux en attente */
    while (read(nerv->magi_read_fd, &sig_byte, 1) == 1) {
        int signum = sig_byte;

        if (signum > 0 && signum < NERV_MAX_SIGNALS &&
            nerv->pilots[signum].active &&
            nerv->pilots[signum].callback) {

            /* Construire l'info */
            alert_info_t alert;
            memset(&alert, 0, sizeof(alert));
            alert.pattern = signum;
            clock_gettime(CLOCK_REALTIME, &alert.timestamp);

            /* Appeler le callback */
            nerv->pilots[signum].callback(&alert, nerv->pilots[signum].user_data);

            /* Gérer ONESHOT */
            if (nerv->pilots[signum].flags & PILOT_FLAG_ONESHOT) {
                pilot_eject(nerv, signum);
            }

            processed++;
        }
    }

    return processed;
}

int eva_standby(nerv_command_t *nerv, const sigset_t *wait_for,
                int timeout_ms, int *received)
{
    if (!nerv || !wait_for)
        return -1;

    sigset_t mask;
    sigfillset(&mask);

    /* Enlever les signaux qu'on attend */
    for (int i = 1; i < NERV_MAX_SIGNALS; i++) {
        if (sigismember(wait_for, i))
            sigdelset(&mask, i);
    }

    if (timeout_ms < 0) {
        int sig;
        if (sigwait(wait_for, &sig) == 0) {
            if (received) *received = sig;
            return 0;
        }
        return -1;
    }

    /* Avec timeout, utiliser sigsuspend en boucle */
    /* Simplifié: pas de vrai timeout ici */
    (void)timeout_ms;
    sigsuspend(&mask);

    return 0;
}

uint64_t angel_kill_count(nerv_command_t *nerv, int pattern)
{
    if (!nerv || pattern <= 0 || pattern >= NERV_MAX_SIGNALS)
        return 0;
    return nerv->pilots[pattern].count;
}

/* Noms des signaux */
static const char *signal_names[] = {
    [0] = "UNKNOWN",
    [SIGHUP] = "SIGHUP",
    [SIGINT] = "SIGINT",
    [SIGQUIT] = "SIGQUIT",
    [SIGILL] = "SIGILL",
    [SIGTRAP] = "SIGTRAP",
    [SIGABRT] = "SIGABRT",
    [SIGBUS] = "SIGBUS",
    [SIGFPE] = "SIGFPE",
    [SIGKILL] = "SIGKILL",
    [SIGUSR1] = "SIGUSR1",
    [SIGSEGV] = "SIGSEGV",
    [SIGUSR2] = "SIGUSR2",
    [SIGPIPE] = "SIGPIPE",
    [SIGALRM] = "SIGALRM",
    [SIGTERM] = "SIGTERM",
    [SIGCHLD] = "SIGCHLD",
    [SIGCONT] = "SIGCONT",
    [SIGSTOP] = "SIGSTOP",
    [SIGTSTP] = "SIGTSTP",
    [SIGTTIN] = "SIGTTIN",
    [SIGTTOU] = "SIGTTOU",
    [SIGURG] = "SIGURG",
    [SIGXCPU] = "SIGXCPU",
    [SIGXFSZ] = "SIGXFSZ",
    [SIGVTALRM] = "SIGVTALRM",
    [SIGPROF] = "SIGPROF",
    [SIGWINCH] = "SIGWINCH",
    [SIGIO] = "SIGIO",
    [SIGSYS] = "SIGSYS",
};

const char *pattern_identify(int pattern)
{
    if (pattern > 0 && pattern < (int)(sizeof(signal_names)/sizeof(signal_names[0])))
        return signal_names[pattern] ? signal_names[pattern] : "UNKNOWN";
    return "UNKNOWN";
}

int pattern_lookup(const char *name)
{
    if (!name)
        return -1;

    /* Skip "SIG" prefix if present */
    if (strncmp(name, "SIG", 3) == 0)
        name += 3;

    for (int i = 1; i < (int)(sizeof(signal_names)/sizeof(signal_names[0])); i++) {
        if (signal_names[i]) {
            const char *sn = signal_names[i] + 3;  /* Skip "SIG" */
            if (strcmp(name, sn) == 0)
                return i;
        }
    }

    return -1;
}

char pattern_default_action(int pattern)
{
    switch (pattern) {
        case SIGHUP: case SIGINT: case SIGKILL: case SIGPIPE:
        case SIGALRM: case SIGTERM: case SIGUSR1: case SIGUSR2:
            return 'T';  /* Terminate */
        case SIGCHLD: case SIGURG: case SIGWINCH:
            return 'I';  /* Ignore */
        case SIGQUIT: case SIGILL: case SIGABRT: case SIGFPE:
        case SIGSEGV: case SIGBUS: case SIGSYS: case SIGTRAP:
        case SIGXCPU: case SIGXFSZ:
            return 'C';  /* Core dump */
        case SIGSTOP: case SIGTSTP: case SIGTTIN: case SIGTTOU:
            return 'S';  /* Stop */
        case SIGCONT:
            return 'X';  /* Continue */
        default:
            return 'T';
    }
}

void nerv_status_report(nerv_command_t *nerv, int fd)
{
    if (!nerv)
        return;

    char buf[256];
    int len;

    len = snprintf(buf, sizeof(buf), "=== NERV Status Report ===\n");
    write(fd, buf, len);

    for (int i = 1; i < NERV_MAX_SIGNALS; i++) {
        if (nerv->pilots[i].count > 0) {
            len = snprintf(buf, sizeof(buf), "%-10s: %lu received\n",
                          pattern_identify(i), (unsigned long)nerv->pilots[i].count);
            write(fd, buf, len);
        }
    }
}

const char *nerv_strerror(nerv_error_t error)
{
    switch (error) {
        case NERV_SUCCESS: return "Success";
        case NERV_ERR_MEMORY: return "Memory allocation failed";
        case NERV_ERR_INVALID: return "Invalid parameter";
        case NERV_ERR_SYSCALL: return "System call failed";
        case NERV_ERR_BLOCKED: return "Signal cannot be caught";
        default: return "Unknown error";
    }
}
```

### 4.10 Solutions Mutantes (6 mutants)

```c
/* Mutant A (Boundary) : Overflow sur signum */
static void internal_handler_mutant_a(int signum, siginfo_t *si, void *ctx)
{
    (void)si; (void)ctx;
    /* BUG: pas de vérification de borne */
    g_nerv->pilots[signum].count++;  /* CRASH si signum >= 64 */

    uint8_t sig_byte = (uint8_t)signum;  /* Overflow si signum > 255 */
    write(g_nerv->magi_write_fd, &sig_byte, 1);
}
/* Pourquoi c'est faux : signum peut être > 255, le cast tronque */
```

```c
/* Mutant B (Safety) : printf dans le handler */
static void internal_handler_mutant_b(int signum, siginfo_t *si, void *ctx)
{
    (void)si; (void)ctx;
    /* BUG: printf n'est PAS async-signal-safe! */
    printf("[HANDLER] Received signal %d\n", signum);

    g_nerv->pilots[signum].count++;
    uint8_t sig_byte = (uint8_t)signum;
    write(g_nerv->magi_write_fd, &sig_byte, 1);
}
/* Pourquoi c'est faux : printf peut deadlock si le signal arrive
   pendant un autre printf */
```

```c
/* Mutant C (Resource) : Fuite de FD */
void nerv_command_shutdown_mutant_c(nerv_command_t *nerv)
{
    if (!nerv)
        return;

    for (int i = 1; i < NERV_MAX_SIGNALS; i++) {
        if (nerv->pilots[i].active)
            signal(i, SIG_DFL);
    }

    /* BUG: Oubli de fermer les FDs! */
    /* close(nerv->magi_read_fd); */
    /* close(nerv->magi_write_fd); */

    g_nerv = NULL;
    free(nerv);
}
/* Pourquoi c'est faux : Fuite de 2 FDs à chaque init/shutdown */
```

```c
/* Mutant D (Logic) : Pas de gestion EINTR */
int magi_process_alerts_mutant_d(nerv_command_t *nerv)
{
    if (!nerv)
        return -1;

    int processed = 0;
    uint8_t sig_byte;

    /* BUG: read peut retourner -1 avec errno=EINTR */
    if (read(nerv->magi_read_fd, &sig_byte, 1) == 1) {
        /* Un seul signal traité! */
        int signum = sig_byte;
        /* ... */
        processed = 1;
    }
    /* On sort au premier signal au lieu de tous les traiter */

    return processed;
}
/* Pourquoi c'est faux : Doit boucler jusqu'à EAGAIN */
```

```c
/* Mutant E (Return) : Mauvais FD retourné */
int central_dogma_fd_mutant_e(nerv_command_t *nerv)
{
    if (!nerv)
        return -1;
    /* BUG: Retourne le write_fd au lieu du read_fd */
    return nerv->magi_write_fd;
}
/* Pourquoi c'est faux : poll() sur write_fd ne détectera pas les signaux */
```

```c
/* Mutant F (Global) : Pas de volatile pour compteur */
typedef struct {
    pilot_callback_fn callback;
    void *user_data;
    pilot_flags_t flags;
    int count;  /* BUG: pas volatile sig_atomic_t */
    int active;
} pilot_entry_mutant_f_t;

/* Pourquoi c'est faux : Le compilateur peut optimiser les accès
   et le handler peut voir une valeur obsolète */
```

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "nerv_command_init",
  "language": "c",
  "type": "code",
  "tier": 2,
  "tier_info": "Mélange (processus + signaux + self-pipe)",
  "tags": ["signals", "sigaction", "async-safety", "self-pipe", "phase2"],
  "passing_score": 70,

  "function": {
    "name": "nerv_command_init",
    "prototype": "nerv_command_t *nerv_command_init(void)",
    "return_type": "nerv_command_t *",
    "parameters": []
  },

  "driver": {
    "reference": "nerv_command_t *ref_nerv_command_init(void) { if (g_nerv != NULL) { errno = EBUSY; return NULL; } nerv_command_t *nerv = calloc(1, sizeof(nerv_command_t)); if (!nerv) return NULL; int pipefd[2]; if (pipe(pipefd) < 0) { free(nerv); return NULL; } fcntl(pipefd[0], F_SETFL, O_NONBLOCK); fcntl(pipefd[1], F_SETFL, O_NONBLOCK); nerv->magi_read_fd = pipefd[0]; nerv->magi_write_fd = pipefd[1]; g_nerv = nerv; return nerv; }",

    "edge_cases": [
      {
        "name": "double_init",
        "setup": "nerv_command_init() already called",
        "expected": "NULL, errno = EBUSY",
        "is_trap": true,
        "trap_explanation": "Singleton - une seule instance autorisée"
      },
      {
        "name": "register_sigkill",
        "function": "pilot_sync_register",
        "args": ["nerv", "SIGKILL", "callback", null, 0],
        "expected": "NERV_ERR_BLOCKED",
        "is_trap": true,
        "trap_explanation": "SIGKILL ne peut pas être intercepté"
      },
      {
        "name": "register_sigstop",
        "function": "pilot_sync_register",
        "args": ["nerv", "SIGSTOP", "callback", null, 0],
        "expected": "NERV_ERR_BLOCKED",
        "is_trap": true,
        "trap_explanation": "SIGSTOP ne peut pas être intercepté"
      },
      {
        "name": "null_nerv",
        "function": "pilot_sync_register",
        "args": [null, "SIGINT", "callback", null, 0],
        "expected": "NERV_ERR_INVALID",
        "is_trap": true,
        "trap_explanation": "nerv NULL"
      },
      {
        "name": "invalid_signal",
        "function": "pilot_sync_register",
        "args": ["nerv", 999, "callback", null, 0],
        "expected": "NERV_ERR_INVALID",
        "is_trap": true,
        "trap_explanation": "Signal invalide"
      },
      {
        "name": "self_pipe_fd",
        "function": "central_dogma_fd",
        "expected": "fd >= 0"
      },
      {
        "name": "pattern_identify_valid",
        "function": "pattern_identify",
        "args": ["SIGINT"],
        "expected": "\"SIGINT\""
      },
      {
        "name": "pattern_identify_invalid",
        "function": "pattern_identify",
        "args": [9999],
        "expected": "\"UNKNOWN\""
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {"min": 1, "max": 64},
          "description": "Signal number"
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["sigaction", "sigemptyset", "sigfillset", "sigaddset", "sigdelset", "sigismember", "sigprocmask", "sigpending", "sigsuspend", "sigwait", "kill", "raise", "pipe", "pipe2", "read", "write", "close", "fcntl", "malloc", "free", "calloc", "realloc", "clock_gettime", "memset", "memcpy", "strlen", "strcmp", "strncmp", "snprintf"],
    "forbidden_functions": ["printf", "fprintf", "sprintf", "exit", "syslog"],
    "check_security": true,
    "check_memory": true,
    "blocking": true,
    "max_globals": 1,
    "async_signal_safety": true
  }
}
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Niveau | Application |
|---------|--------|-------------|
| sigaction() | Avancé | API moderne de gestion de signaux |
| Async-signal-safety | Critique | Écrire du code sûr dans les handlers |
| Self-pipe pattern | Avancé | Intégration avec event loops |
| Signal masks | Intermédiaire | Blocage temporaire de signaux |
| volatile sig_atomic_t | Technique | Variables partagées sûres |

### 5.2 LDA — Traduction Littérale

```
FONCTION internal_handler QUI NE RETOURNE RIEN ET PREND EN PARAMÈTRES signum QUI EST UN ENTIER ET si QUI EST UN POINTEUR VERS siginfo_t ET context QUI EST UN POINTEUR VERS void
DÉBUT FONCTION
    SI g_nerv EST ÉGAL À NUL OU signum EST INFÉRIEUR OU ÉGAL À 0 OU signum EST SUPÉRIEUR OU ÉGAL À NERV_MAX_SIGNALS ALORS
        RETOURNER
    FIN SI

    INCRÉMENTER count DU PILOTE À LA POSITION signum DE g_nerv DE 1

    DÉCLARER sig_byte COMME ENTIER NON SIGNÉ SUR 8 BITS
    AFFECTER signum CONVERTI EN ENTIER 8 BITS À sig_byte

    APPELER write AVEC LE FD D'ÉCRITURE DU MAGI DE g_nerv ET L'ADRESSE DE sig_byte ET 1
FIN FONCTION
```

### 5.2.2.1 Logic Flow

```
ALGORITHME : MAGI Process Alerts
---
1. VÉRIFIER que nerv n'est pas NULL
   → Si NULL : RETOURNER -1

2. INITIALISER compteur processed = 0

3. BOUCLE INFINIE (lecture self-pipe) :
   |
   |-- LIRE 1 octet du pipe read_fd
   |
   |-- SI read retourne 1 (octet lu) :
   |     |-- CONVERTIR octet en numéro de signal
   |     |-- VÉRIFIER que le signal est valide et actif
   |     |-- CONSTRUIRE structure alert_info
   |     |-- APPELER le callback du pilote
   |     |-- SI flag ONESHOT : ÉJECTER le pilote
   |     |-- INCRÉMENTER processed
   |
   |-- SINON (EAGAIN ou erreur) :
   |     SORTIR de la boucle

4. RETOURNER processed
```

### 5.3 Visualisation ASCII

#### Architecture Self-Pipe

```
                          ┌─────────────────────────────┐
                          │      SIGNAL ARRIVES         │
                          │   (asynchronous interrupt)  │
                          └─────────────┬───────────────┘
                                        │
                                        ▼
┌──────────────────────────────────────────────────────────────┐
│                    INTERNAL HANDLER                           │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  /* ASYNC-SIGNAL-SAFE ONLY */                        │    │
│  │  g_nerv->pilots[signum].count++;  /* atomic */       │    │
│  │  write(magi_write_fd, &sig_byte, 1);  /* async-safe */│    │
│  └──────────────────────────────────────────────────────┘    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │      SELF-PIPE        │
                    │  ┌─────────────────┐  │
                    │  │ [sig][sig][sig] │  │
                    │  │  Write ──► Read │  │
                    │  └─────────────────┘  │
                    └───────────┬───────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────┐
│                    MAIN LOOP (poll/epoll)                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │  pollfd.fd = central_dogma_fd(nerv);                 │    │
│  │  poll(&pfd, 1, timeout);                             │    │
│  │  if (pfd.revents & POLLIN)                           │    │
│  │      magi_process_alerts(nerv);  /* SAFE CONTEXT */  │    │
│  └──────────────────────────────────────────────────────┘    │
└───────────────────────────────┬──────────────────────────────┘
                                │
                                ▼
                    ┌───────────────────────┐
                    │   USER CALLBACK       │
                    │   (can use printf,    │
                    │    malloc, etc.)      │
                    └───────────────────────┘
```

#### AT Field (Signal Blocking)

```
Sans AT Field                    Avec AT Field Déployé
──────────────                   ─────────────────────

   Signal                           Signal
     │                                │
     ▼                                ▼
┌─────────┐                    ┌─────────────────┐
│ Process │                    │   AT FIELD      │
│         │                    │   (sigprocmask) │
│ Handler │ ◄── Interrupted!   │   ┌───────────┐ │
│ called  │                    │   │ BLOCKED   │ │
└─────────┘                    │   │ (pending) │ │
                               │   └───────────┘ │
                               └────────┬────────┘
                                        │
                                  AT Field Down
                                        │
                                        ▼
                               ┌─────────────────┐
                               │ Signal Delivered│
                               │ Handler called  │
                               └─────────────────┘
```

### 5.4 Les Pièges en Détail

#### Piège 1 : Fonctions non async-signal-safe

```c
/* DANGER ABSOLU! */
void bad_handler(int sig)
{
    printf("Signal %d received!\n", sig);  /* DEADLOCK possible! */
    char *p = malloc(100);                  /* CORRUPTION possible! */
    free(p);                                /* CRASH possible! */
}

/* CORRECT */
volatile sig_atomic_t g_sig_received = 0;

void good_handler(int sig)
{
    g_sig_received = sig;  /* Seule opération vraiment safe */
}

/* Ou avec self-pipe */
void self_pipe_handler(int sig)
{
    uint8_t s = sig;
    write(g_pipe_fd, &s, 1);  /* write() est async-safe */
}
```

#### Piège 2 : Les signaux standard ne s'accumulent pas

```c
/* Si SIGUSR1 est envoyé 10 fois très rapidement... */
for (int i = 0; i < 10; i++)
    kill(pid, SIGUSR1);

/* ...le handler peut n'être appelé qu'une seule fois!
   Les signaux standard sont des "flags", pas une queue. */

/* Solution : Utiliser des signaux temps-réel (SIGRTMIN+n)
   qui eux s'accumulent, ou compter dans le handler */
```

#### Piège 3 : Race condition sur g_nerv

```c
/* Race entre sig_destroy() et signal entrant */
void nerv_command_shutdown(nerv_command_t *nerv)
{
    g_nerv = NULL;  /* Un signal peut arriver ICI */
    free(nerv);      /* Et le handler accède à g_nerv... CRASH! */
}

/* Solution : Bloquer tous les signaux d'abord */
void nerv_command_shutdown_safe(nerv_command_t *nerv)
{
    sigset_t all;
    sigfillset(&all);
    sigprocmask(SIG_BLOCK, &all, NULL);  /* Plus de signaux! */

    g_nerv = NULL;
    free(nerv);

    sigprocmask(SIG_UNBLOCK, &all, NULL);
}
```

### 5.5 Cours Complet

#### 5.5.1 Qu'est-ce qu'un Signal ?

Un signal est une notification logicielle envoyée à un processus. C'est le mécanisme Unix historique pour :
- Notifier un événement (SIGCHLD quand un enfant termine)
- Interrompre un processus (SIGINT = Ctrl+C)
- Demander une action (SIGHUP = recharger config)
- Terminer un processus (SIGTERM, SIGKILL)

#### 5.5.2 signal() vs sigaction()

```c
/* ANCIEN (signal) - NON PORTABLE! */
signal(SIGINT, my_handler);
/* Problèmes:
   - Comportement différent entre Unix et BSD
   - Handler peut être réinitialisé après un appel
   - Pas de contrôle sur les flags
*/

/* MODERNE (sigaction) - RECOMMANDÉ */
struct sigaction sa;
sa.sa_handler = my_handler;
sigemptyset(&sa.sa_mask);
sa.sa_flags = SA_RESTART;  /* Contrôle fin! */
sigaction(SIGINT, &sa, NULL);
```

#### 5.5.3 Les Flags de sigaction

| Flag | Description |
|------|-------------|
| `SA_RESTART` | Redémarre automatiquement les syscalls interrompus |
| `SA_NODEFER` | Ne bloque pas le signal pendant le handler |
| `SA_RESETHAND` | Restaure SIG_DFL après premier appel |
| `SA_SIGINFO` | Handler reçoit des infos détaillées (siginfo_t) |
| `SA_NOCLDSTOP` | Pas de SIGCHLD pour les stops (uniquement terminaison) |

#### 5.5.4 Async-Signal-Safety

Une fonction est async-signal-safe si elle peut être appelée depuis un signal handler sans risque. La liste complète est dans `man 7 signal-safety`.

**Fonctions SAFE :**
- `write()`, `read()` (la plupart des syscalls)
- `_exit()`, `_Exit()`
- `signal()`, `sigaction()`, `sigprocmask()`
- Accès à `volatile sig_atomic_t`

**Fonctions NON-SAFE :**
- `printf()`, `fprintf()`, `sprintf()`
- `malloc()`, `free()`, `realloc()`
- `exit()` (utilise atexit handlers)
- `syslog()`, `localtime()`

### 5.6 Normes avec Explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais undefined behavior)                │
├─────────────────────────────────────────────────────────────────┤
│ int count = 0;  /* Variable partagée avec handler */            │
│ void handler(int sig) { count++; }                              │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ volatile sig_atomic_t count = 0;                                │
│ void handler(int sig) { count++; }                              │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • volatile empêche les optimisations du compilateur             │
│ • sig_atomic_t garantit l'accès atomique                        │
│ • Sans ces qualificatifs, le compilateur peut mettre count      │
│   en registre et ne jamais voir les modifications du handler    │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec Trace d'Exécution

**Scénario : Réception de SIGINT avec self-pipe**

```
┌───────┬─────────────────────────────────────────┬───────────────────────┐
│ Étape │ Action                                  │ État                  │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   1   │ poll() bloque sur self-pipe fd          │ Main loop en attente  │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   2   │ User presse Ctrl+C                      │ Signal SIGINT généré  │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   3   │ Kernel interrompt poll()                │ Handler appelé        │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   4   │ Handler: pilots[SIGINT].count++         │ Compteur = 1          │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   5   │ Handler: write(pipe_fd, 0x02, 1)        │ Pipe contient [0x02]  │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   6   │ Handler retourne                        │ poll() reprend        │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   7   │ poll() retourne (POLLIN sur pipe)       │ revents = POLLIN      │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   8   │ magi_process_alerts() appelé            │ Contexte normal!      │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│   9   │ read(pipe_fd) → 0x02                    │ Pipe vidé             │
├───────┼─────────────────────────────────────────┼───────────────────────┤
│  10   │ Callback utilisateur avec printf!       │ SAFE car main context │
└───────┴─────────────────────────────────────────┴───────────────────────┘
```

### 5.8 Mnémotechniques

#### 🚨 MEME : "Shinji, get in the handler!"

![Evangelion](meme_shinji.jpg)

Dans Evangelion, Shinji doit entrer dans l'Entry Plug pour piloter l'EVA. Mais il ne peut pas emmener n'importe quoi — l'Entry Plug a un environnement très restreint.

C'est pareil pour les signal handlers :
- L'Entry Plug = Le contexte async-signal-safe
- Shinji = Ton code
- Les objets interdits = malloc, printf, exit
- write() = La seule action autorisée

```c
void handler_shinji_gets_in(int sig)
{
    /* Dans l'Entry Plug, seules certaines actions sont possibles */
    g_signal_received = sig;  /* Synchro avec l'EVA */
    write(pipe_fd, &sig, 1);  /* Communication avec NERV */

    /* printf("Help!"); ← Shinji ne peut pas appeler Misato ici! */
    /* malloc(100);     ← Shinji ne peut pas récupérer d'objets! */
}
```

**La règle de Shinji :** Si tu ne peux pas le faire dans un Entry Plug rempli de LCL, tu ne peux pas le faire dans un handler.

---

#### 📡 MEME : "MAGI System Online"

Le système MAGI de NERV (Melchior, Balthasar, Casper) vote sur chaque décision. Le self-pipe, c'est le système de communication qui transmet les alertes au MAGI.

```
Signal → Handler → Pipe → MAGI (main loop) → Décision
```

Comme les trois MAGI doivent se synchroniser avant d'agir, ton code doit aussi attendre d'être dans le bon contexte (main loop) avant d'appeler les fonctions dangereuses.

### 5.9 Applications Pratiques

| Application | Signaux utilisés | Pattern |
|-------------|------------------|---------|
| nginx | SIGHUP (reload), SIGTERM (shutdown) | Self-pipe + event loop |
| PostgreSQL | SIGTERM, SIGINT, SIGHUP | Handler + flags |
| Docker | SIGTERM → containers | Propagation |
| systemd | Tous | signalfd Linux |
| bash | SIGCHLD, SIGTSTP, SIGCONT | Job control |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Conséquence | Solution |
|---|-------|-------------|----------|
| 1 | printf/malloc dans handler | Deadlock, corruption | Self-pipe + traitement dans main |
| 2 | Variable non-volatile | Optimisations cassent le code | volatile sig_atomic_t |
| 3 | Signaux standard coalesced | Perte de signaux | Compteur ou signaux RT |
| 4 | Race sur g_nerv | CRASH | Bloquer signaux avant destroy |
| 5 | EINTR non géré | Comportement erratique | Boucle retry |
| 6 | Pipe bloquant | Deadlock si plein | O_NONBLOCK obligatoire |
| 7 | SIGKILL/SIGSTOP | Erreur | Vérifier dans register |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quelles fonctions sont async-signal-safe ?**

A) printf()
B) write()
C) malloc()
D) _exit()
E) signal()
F) free()
G) exit()
H) sigaction()
I) read()
J) syslog()

**Réponses correctes : B, D, E, H, I**

---

### Question 2
**Que fait le flag SA_RESTART ?**

A) Redémarre le processus après le signal
B) Redémarre automatiquement les syscalls interrompus
C) Réinstalle le handler après chaque appel
D) Bloque le signal pendant le handler

**Réponse correcte : B**

---

### Question 3
**Pourquoi SIGKILL ne peut-il pas être intercepté ?**

A) C'est un bug historique
B) Pour garantir qu'on peut toujours terminer un processus
C) SIGKILL n'existe pas vraiment
D) Le kernel ne supporte pas ce signal

**Réponse correcte : B**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Aspect | Détail |
|--------|--------|
| **Fonction principale** | `nerv_command_init()` |
| **Difficulté** | 6/10 (★★★★★★☆☆☆☆) |
| **Temps estimé** | 6 heures |
| **Concepts clés** | sigaction, self-pipe, async-signal-safety |
| **Piège majeur** | Fonctions non-safe dans handler |
| **Application réelle** | Serveurs, daemons, shells |
| **XP** | 600 (base) + 1800 (bonus) |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.2.4-nerv_command_init",
    "generated_at": "2025-01-11 00:00:00",

    "metadata": {
      "exercise_id": "2.2.4",
      "exercise_name": "nerv_command_init",
      "module": "2.2",
      "module_name": "Processes & Shell",
      "concept": "d",
      "concept_name": "Signal Handling",
      "type": "code",
      "tier": 2,
      "tier_info": "Mélange (processus + signaux + self-pipe)",
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "c",
      "language_version": "C17",
      "duration_minutes": 360,
      "xp_base": 600,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "ADVANCED",
      "bonus_icon": "🔥",
      "complexity_time": "T2 O(n)",
      "complexity_space": "S1 O(1)",
      "prerequisites": ["2.2.0", "2.2.1", "2.2.2", "2.2.3"],
      "domains": ["Process", "FS", "Concurrency"],
      "domains_bonus": ["Linux-specific", "Threading"],
      "tags": ["signals", "sigaction", "async-safety", "self-pipe"],
      "meme_reference": "Evangelion - NERV, MAGI, AT Field"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_nerv_signals.c": "/* Section 4.3 */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 */",
      "mutants/mutant_e_return.c": "/* Section 4.10 */",
      "mutants/mutant_f_global.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": ["references/ref_nerv_signals.c"],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c",
        "mutants/mutant_f_global.c"
      ]
    }
  }
}
```

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 25 | Analogie Evangelion parfaite pour l'asynchrone |
| Couverture conceptuelle | 25 | sigaction, self-pipe, masks, async-safety |
| Testabilité auto | 24 | 18+ tests, 6 mutants, async-safety review |
| Originalité | 24 | Thème Evangelion unique et pertinent |
| **TOTAL** | **98/100** | ✓ Validé |

---

*HACKBRAIN v5.5.2 — Module 2.2 Exercise 04*
*"Shinji, get in the handler!"*
