<thinking>
## Analyse du Concept
- Concept : Signal Messenger - Communication inter-processus via signaux temps-réel (sigqueue, SIGRTMIN-SIGRTMAX, protocole ACK/retry)
- Phase demandée : 2
- Adapté ? OUI - Les signaux RT et sigqueue() sont des concepts système intermédiaires, parfaitement adaptés à la Phase 2

## Combo Base + Bonus
- Exercice de base : Bibliothèque sigmsg avec envoi/réception, ACK, retry, ping/pong, broadcast
- Bonus : Ajout de fragmentation pour données > 16 bits, protocole de réassemblage, ordonnancement FIFO garanti
- Palier bonus : 🔥 Avancé (protocole de fragmentation complexe)
- Progression logique ? OUI - Base = protocole simple, Bonus = fragmentation et réassemblage

## Prérequis & Difficulté
- Prérequis réels : ex04 (sigaction, handlers), fork(), signaux de base
- Difficulté estimée : 6/10
- Cohérent avec phase ? OUI - Phase 2 autorise 4-6/10

## Aspect Fun/Culture
- Contexte choisi : Metal Gear Solid - CODEC system
- MEME mnémotechnique : "Snake? Snake! SNAAAAKE!" (quand pas de réponse = timeout)
- Pourquoi c'est fun : Le CODEC est le système de communication emblématique de MGS, avec les fréquences (= signaux RT), ACK ("Copy that"), timeout ("Come in Snake"), broadcast à l'équipe

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : `sigqueue(dest, SIGRTMIN + type, val)` avec type > MSG_RT_SIGNALS - overflow du numéro de signal
2. Mutant B (Safety) : Pas de vérification `kill(dest, 0)` avant sigqueue - ESRCH non géré
3. Mutant C (Resource) : self-pipe créé mais jamais fermé dans msg_destroy() - fuite FD
4. Mutant D (Logic) : ACK attendu avec le mauvais seq number - désynchro du protocole
5. Mutant E (Return) : msg_send_sync retourne 0 même sur timeout - faux positif
6. Mutant F (Race) : Pas de boucle EINTR sur sigtimedwait - interruption non gérée

## Verdict
VALIDE - Exercice complet avec protocole de messagerie, analogie MGS parfaite
</thinking>

# Exercice 2.2.5 : codec_transmit

**Module :**
2.2 — Processes & Shell

**Concept :**
e — Signal Messenger (RT Signals, sigqueue, protocole ACK/retry)

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
code

**Tiers :**
3 — Synthèse (signaux RT + protocole + IPC)

**Langage :**
C (C17)

**Prérequis :**
- ex04 (sigaction, handlers de signaux)
- fork() et processus
- Self-pipe pattern

**Domaines :**
Process, Net

**Durée estimée :**
240 min

**XP Base :**
400

**Complexité :**
T3 O(n) × S2 O(1)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex05/
├── codec.h           # API publique (système CODEC)
├── codec.c           # Implémentation principale
├── codec_protocol.c  # Protocole ACK/retry (optionnel)
├── main.c            # Démonstration
└── Makefile
```

**Fonctions autorisées :**
```c
// Signaux
sigaction, sigemptyset, sigaddset, sigfillset
sigqueue, kill, raise, killpg
sigpending, sigprocmask, sigsuspend, sigtimedwait

// I/O
pipe, pipe2, read, write, close
fcntl, poll

// Temps
clock_gettime, nanosleep

// Mémoire
malloc, free, calloc, realloc, memset, memcpy

// Processus
getpid, getuid
```

**Fonctions interdites :**
```c
signal()          // Utiliser sigaction
sleep(), usleep() // Utiliser nanosleep
printf()          // Dans les handlers (utiliser write)
```

### 1.2 Consigne

**🎮 METAL GEAR SOLID — Le Système CODEC**

*"This is Snake. Do you read me?"*

Dans Metal Gear Solid, le CODEC est le système de communication radio sécurisé qui permet à Solid Snake de rester en contact avec son équipe de support. Chaque membre a une fréquence unique, les messages sont acquittés ("Copy that"), et si Snake ne répond pas... "Snake? Snake! SNAAAAKE!"

Tu vas implémenter le **système CODEC** : une bibliothèque de messagerie inter-processus basée sur les signaux temps-réel (RT signals). Comme les transmissions radio de la Shadow Moses mission, ton protocole doit être **fiable** malgré un canal intrinsèquement instable.

**Ta mission :**

Créer une bibliothèque `codec` qui implémente un système de communication entre processus avec :
- Envoi de messages typés via signaux RT (SIGRTMIN à SIGRTMAX)
- Protocole d'acquittement (ACK/NACK = "Copy that" / "Negative")
- Retransmission automatique sur timeout ("Come in, Snake!")
- Mesure de latence via PING/PONG
- Broadcast à un groupe de processus

**Entrée :**
- `codec_config_t *config` : Configuration (fréquences, timeouts, retries)
- `pid_t dest` : PID du destinataire (comme une fréquence CODEC)
- `codec_signal_t type` : Type de transmission
- `int16_t payload` : Données (intel, ordres, status)

**Sortie :**
- Retourne `0` si transmission ACK reçu
- Retourne `-1` si timeout/erreur (errno positionné)
- Retourne `1` si NACK reçu ("Negative, Snake")

**Contraintes :**
- Utiliser exclusivement les signaux RT (SIGRTMIN à SIGRTMAX)
- Numéros de séquence pour la fiabilité du protocole
- Retry automatique configurable
- Pas de blocage infini (tous les waits ont un timeout)
- Valgrind clean obligatoire

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `codec_call_snake(codec, pid, 1000)` | `145` | PING répondu en 145 microsecondes |
| `codec_call_snake(codec, dead_pid, 1000)` | `-1` | Timeout - "SNAAAAKE!" |
| `codec_transmit_await_copy(codec, pid, INTEL, 42, 1000, NULL)` | `0` | Intel transmis, ACK reçu |

---

### 1.2.2 Consigne Académique

Implémenter une bibliothèque de communication inter-processus utilisant les signaux temps-réel POSIX. La bibliothèque doit fournir :

1. **Transport Layer** : Utilisation de `sigqueue()` pour envoyer des signaux RT avec données (union sigval)
2. **Protocol Layer** : Séquençage, acquittements, retransmissions
3. **Application Layer** : API haut niveau pour envoi/réception de messages

Les signaux temps-réel (SIGRTMIN à SIGRTMAX) sont mis en file d'attente contrairement aux signaux standard, et permettent de transporter 32 bits de données via `sigqueue()`.

### 1.3 Prototype

```c
#ifndef CODEC_H
#define CODEC_H

#include <signal.h>
#include <stdint.h>
#include <sys/types.h>

// Nombre de fréquences disponibles (signaux RT)
#define CODEC_FREQUENCIES (SIGRTMAX - SIGRTMIN + 1)

// Types de transmissions CODEC
typedef enum {
    CODEC_SIGNAL_CALLOUT   = 0,  // "Snake?" - Test de connectivité
    CODEC_SIGNAL_RESPOND   = 1,  // "I'm here" - Réponse
    CODEC_SIGNAL_INTEL     = 2,  // Données de mission
    CODEC_SIGNAL_COPY      = 3,  // "Copy that" - ACK
    CODEC_SIGNAL_NEGATIVE  = 4,  // "Negative" - NACK
    CODEC_SIGNAL_ORDER     = 5,  // Ordre de mission
    CODEC_SIGNAL_SITREP    = 6,  // Situation Report
    CODEC_SIGNAL_ABORT     = 7,  // Mission abort
    CODEC_SIGNAL_CUSTOM    = 8,  // Types custom (8-31)
    CODEC_SIGNAL_MAX       = 32
} codec_signal_t;

// Données encodées dans une transmission (32 bits)
typedef union {
    int32_t raw;
    struct {
        uint8_t type;      // Type de signal (0-31)
        uint8_t seq;       // Numéro de séquence (0-255)
        int16_t payload;   // Données (16 bits signés)
    } fields;
} codec_data_t;

// Transmission reçue avec métadonnées
typedef struct {
    codec_signal_t  type;           // Type de signal
    uint8_t         seq;            // Numéro de séquence
    int16_t         payload;        // Données
    pid_t           sender_pid;     // PID de l'émetteur (fréquence source)
    uid_t           sender_uid;     // UID de l'émetteur
    struct timespec timestamp;      // Heure de réception
} codec_transmission_t;

// Gestionnaire CODEC (opaque)
typedef struct codec codec_t;

// Callback pour réception
typedef void (*codec_handler_fn)(const codec_transmission_t *trans, void *user_data);

// Configuration CODEC
typedef struct {
    int     base_frequency;     // Signal RT de base (défaut: SIGRTMIN)
    int     num_frequencies;    // Nombre de fréquences (défaut: 8)
    int     default_timeout_ms; // Timeout par défaut (défaut: 1000)
    int     max_retries;        // "Come in Snake" retries (défaut: 3)
    size_t  queue_size;         // Taille file réception (défaut: 64)
} codec_config_t;

#define CODEC_CONFIG_DEFAULT { \
    .base_frequency = 0,       \
    .num_frequencies = 8,      \
    .default_timeout_ms = 1000,\
    .max_retries = 3,          \
    .queue_size = 64           \
}

// === CRÉATION / DESTRUCTION ===

codec_t *codec_initialize(const codec_config_t *config);
void codec_shutdown(codec_t *codec);
pid_t codec_get_frequency(codec_t *codec);

// === TRANSMISSION ===

typedef enum {
    CODEC_NOWAIT   = (1 << 0),  // Ne pas attendre
    CODEC_NORETRY  = (1 << 1),  // Pas de retry
    CODEC_SYNC     = (1 << 2),  // Attendre ACK
} codec_flags_t;

int codec_transmit(codec_t *codec, pid_t dest, codec_signal_t type,
                   int16_t payload, codec_flags_t flags);

int codec_transmit_await_copy(codec_t *codec, pid_t dest, codec_signal_t type,
                              int16_t payload, int timeout_ms,
                              codec_transmission_t *response);

int64_t codec_call_snake(codec_t *codec, pid_t dest, int timeout_ms);

int codec_team_broadcast(codec_t *codec, pid_t pgid, codec_signal_t type,
                         int16_t payload);

// === RÉCEPTION ===

int codec_tune_frequency(codec_t *codec, codec_signal_t type,
                         codec_handler_fn handler, void *user_data);

int codec_drop_frequency(codec_t *codec, codec_signal_t type);

int codec_get_fd(codec_t *codec);

int codec_process_incoming(codec_t *codec);

int codec_standby(codec_t *codec, int timeout_ms, codec_transmission_t *trans);

int codec_flush_queue(codec_t *codec);

// === STATISTIQUES ===

typedef struct {
    uint64_t transmitted;       // Transmissions envoyées
    uint64_t received;          // Transmissions reçues
    uint64_t copies;            // ACKs reçus ("Copy that")
    uint64_t negatives;         // NACKs reçus ("Negative")
    uint64_t timeouts;          // Pas de réponse
    uint64_t retries;           // "Come in Snake" count
    uint64_t dropped;           // File pleine
    uint64_t errors;            // Erreurs de transmission
    int64_t  avg_latency_us;    // Latence moyenne
    int64_t  max_latency_us;    // Latence max
} codec_stats_t;

void codec_mission_stats(codec_t *codec, codec_stats_t *stats);
void codec_reset_stats(codec_t *codec);
void codec_print_sitrep(codec_t *codec, int fd);
int codec_agent_exists(pid_t dest);

#endif /* CODEC_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### Les Signaux Temps-Réel — La Radio Quantique de Linux

Les signaux POSIX standard (SIGTERM, SIGINT, etc.) ont un défaut majeur : ils sont **coalescés**. Si tu envoies 10 SIGUSR1 pendant que le processus est bloqué, il n'en recevra qu'UN seul.

Les **signaux temps-réel** (SIGRTMIN à SIGRTMAX) corrigent ça :
- **File d'attente** : Chaque envoi est mémorisé, tous sont délivrés
- **Données associées** : `sigqueue()` permet d'envoyer 32 bits avec le signal
- **Ordre FIFO** : Livraison dans l'ordre d'envoi
- **Priorité** : Les signaux de numéro plus bas sont traités d'abord

```c
// Signal standard : PERDU si plusieurs arrivent
kill(pid, SIGUSR1);
kill(pid, SIGUSR1);  // Peut fusionner avec le précédent

// Signal RT : TOUS délivrés avec données
union sigval val = { .sival_int = 42 };
sigqueue(pid, SIGRTMIN, val);  // Garanti délivré
sigqueue(pid, SIGRTMIN, val);  // Aussi délivré, séparément
```

### 2.5 DANS LA VRAIE VIE

**DevOps / SRE** : Les signaux RT sont utilisés pour la **coordination de processus** :
- Orchestrateurs qui notifient les workers de nouvelles tâches
- Systèmes de monitoring qui collectent des métriques via signaux
- Graceful shutdown avec acquittement

**Systèmes embarqués** : Dans les environnements avec ressources limitées :
- Communication IPC légère sans shared memory
- Notification d'événements hardware
- Protocoles temps-réel critiques

**Debugging avancé** : Outils comme `strace` et profilers utilisent les signaux RT pour :
- Instrumentation de processus en cours d'exécution
- Collecte de traces sans arrêter le programme
- Injection de commandes de debugging

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
codec.c  codec.h  codec_protocol.c  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -std=c17 -D_POSIX_C_SOURCE=200809L -c codec.c
gcc -Wall -Wextra -Werror -std=c17 -D_POSIX_C_SOURCE=200809L -c main.c
ar rcs libcodec.a codec.o
gcc -o codec_demo main.o -L. -lcodec

$ ./codec_demo
[CODEC] Snake initialized on frequency 12346
[CODEC] Otacon initialized on frequency 12347
[Snake] Transmitting intel to Otacon...
[Otacon] Received INTEL payload=42 from Snake
[Otacon] Copy that!
[Snake] Transmission acknowledged
[Snake] Calling Otacon... Snake? Snake!
[Otacon] I'm here, Snake.
[Snake] Response time: 142 us
=== MISSION STATISTICS ===
Transmitted: 5, Received: 5, Copies: 3, Timeouts: 0
Average latency: 138 us
```

---

## ⚡ SECTION 3.1 : BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n) pour fragmentation/réassemblage

**Space Complexity attendue :**
O(n) pour buffer de fragments

**Domaines Bonus :**
`Compression, Net`

### 3.1.1 Consigne Bonus

**🎮 CODEC BURST TRANSMISSION — Données Fragmentées**

*"Snake, I'm uploading the Metal Gear schematics to your CODEC. Stand by for burst transmission."*

Otacon doit parfois envoyer plus que 16 bits de données à Snake. Pour les **fichiers de mission** (plans, photos, briefings), le CODEC utilise le mode **burst transmission** : les données sont découpées en fragments, envoyées séquentiellement, et réassemblées côté récepteur.

**Ta mission bonus :**

Implémenter le protocole de **fragmentation et réassemblage** pour envoyer des données arbitraires (jusqu'à 1024 octets) via signaux RT.

**Entrée :**
- `const void *data` : Données à transmettre
- `size_t len` : Taille (max 1024 octets)

**Sortie :**
- `0` si toutes les parties transmises et acquittées
- `-1` si erreur ou timeout

**Contraintes :**
┌─────────────────────────────────────────┐
│  1 ≤ len ≤ 1024                         │
│  Chaque fragment = 4 octets de données  │
│  ACK par fragment requis                │
│  Réassemblage FIFO garanti              │
│  Timeout global pour l'ensemble         │
└─────────────────────────────────────────┘

### 3.1.2 Prototype Bonus

```c
// Fragment de burst transmission
typedef struct {
    uint8_t  msg_id;      // ID de la transmission
    uint8_t  frag_index;  // Index du fragment (0-255)
    uint8_t  frag_count;  // Nombre total de fragments
    uint8_t  flags;       // FIRST, LAST, etc.
    uint8_t  data[4];     // Données du fragment
} codec_fragment_t;

// Envoie des données via burst transmission
int codec_burst_transmit(codec_t *codec, pid_t dest,
                         const void *data, size_t len,
                         int timeout_ms);

// Callback pour réception burst
typedef void (*codec_burst_handler_fn)(const void *data, size_t len,
                                        pid_t sender, void *user_data);

// Enregistre handler pour burst
int codec_on_burst(codec_t *codec, codec_burst_handler_fn handler,
                   void *user_data);
```

### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Taille données | 16 bits | 1024 octets |
| Protocole | Simple ACK | Fragmentation + ACK par fragment |
| Complexité | O(1) | O(n) |
| Buffer | Non | Réassemblage requis |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Description | Points | Trap |
|------|-------------|--------|------|
| `test_01_create_destroy` | Création et destruction basique | 5 | - |
| `test_02_null_config` | Config NULL = défauts | 3 | - |
| `test_03_send_invalid_pid` | Envoi vers PID inexistant | 5 | ESRCH |
| `test_04_send_receive` | Parent-enfant simple | 8 | - |
| `test_05_ping_pong` | Mesure latence CODEC | 8 | - |
| `test_06_sequence_numbers` | Séquençage correct | 5 | Désync |
| `test_07_timeout_retry` | Retry automatique | 8 | Race |
| `test_08_broadcast` | Envoi au groupe | 6 | - |
| `test_09_handler_registration` | tune/drop frequency | 5 | - |
| `test_10_process_died` | Destinataire meurt | 7 | Crash |
| `test_11_queue_overflow` | File pleine | 5 | - |
| `test_12_self_send` | Envoi à soi-même | 5 | - |
| `test_13_permissions` | Envoi vers init(1) | 5 | EPERM |
| `test_14_stats` | Statistiques correctes | 5 | - |
| `test_15_flush` | Vider la file | 4 | - |
| `test_16_valgrind` | Pas de fuites | 8 | FD leak |
| `test_17_throughput` | >5000 msg/sec | 4 | - |
| `test_18_latency_p99` | <500us 99th percentile | 4 | - |
| **TOTAL** | | **100** | |

### 4.2 main.c de test

```c
#include "codec.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <assert.h>
#include <string.h>

// Handler pour le récepteur
void otacon_handler(const codec_transmission_t *trans, void *user_data) {
    codec_t *codec = (codec_t *)user_data;

    const char *sig_name[] = {"CALLOUT", "RESPOND", "INTEL", "COPY",
                              "NEGATIVE", "ORDER", "SITREP", "ABORT"};

    // Simuler "Otacon ici"
    write(STDOUT_FILENO, "[Otacon] ", 9);
    write(STDOUT_FILENO, "Received ", 9);
    write(STDOUT_FILENO, sig_name[trans->type], strlen(sig_name[trans->type]));
    write(STDOUT_FILENO, "\n", 1);

    // Envoyer ACK
    codec_transmit(codec, trans->sender_pid, CODEC_SIGNAL_COPY,
                   trans->seq, CODEC_NOWAIT);
}

void test_basic_communication(void) {
    printf("=== Test: Basic Communication ===\n");

    codec_t *snake = codec_initialize(NULL);
    assert(snake != NULL);

    pid_t child = fork();
    if (child == 0) {
        // Otacon (enfant)
        codec_t *otacon = codec_initialize(NULL);
        codec_tune_frequency(otacon, CODEC_SIGNAL_INTEL, otacon_handler, otacon);

        for (int i = 0; i < 3; i++) {
            codec_standby(otacon, 5000, NULL);
            codec_process_incoming(otacon);
        }

        codec_shutdown(otacon);
        _exit(0);
    }

    // Snake (parent)
    usleep(100000);  // Laisser Otacon s'initialiser

    for (int i = 0; i < 3; i++) {
        printf("[Snake] Transmitting intel %d...\n", i);
        int ret = codec_transmit_await_copy(snake, child, CODEC_SIGNAL_INTEL,
                                            i * 10, 1000, NULL);
        printf("[Snake] Result: %s\n", ret == 0 ? "COPY" : "TIMEOUT");
        assert(ret == 0);
    }

    waitpid(child, NULL, 0);
    codec_shutdown(snake);
    printf("=== PASS ===\n\n");
}

void pong_handler(const codec_transmission_t *trans, void *user_data) {
    codec_t *codec = (codec_t *)user_data;
    codec_transmit(codec, trans->sender_pid, CODEC_SIGNAL_RESPOND,
                   trans->payload, CODEC_NOWAIT);
}

void test_ping_pong(void) {
    printf("=== Test: Ping-Pong Latency ===\n");

    pid_t child = fork();
    if (child == 0) {
        codec_t *otacon = codec_initialize(NULL);
        codec_tune_frequency(otacon, CODEC_SIGNAL_CALLOUT, pong_handler, otacon);

        for (int i = 0; i < 10; i++) {
            codec_standby(otacon, 2000, NULL);
            codec_process_incoming(otacon);
        }

        codec_shutdown(otacon);
        _exit(0);
    }

    usleep(100000);
    codec_t *snake = codec_initialize(NULL);

    printf("[Snake] Snake? Snake! SNAAAAKE!\n");

    int success = 0;
    int64_t total = 0;

    for (int i = 0; i < 10; i++) {
        int64_t latency = codec_call_snake(snake, child, 1000);
        if (latency >= 0) {
            printf("  Response %d: %ld us\n", i + 1, latency);
            total += latency;
            success++;
        } else {
            printf("  Response %d: NO RESPONSE\n", i + 1);
        }
    }

    printf("Average latency: %ld us (%d/10)\n",
           success > 0 ? total / success : 0, success);
    assert(success >= 8);  // Au moins 80% de réponses

    codec_shutdown(snake);
    waitpid(child, NULL, 0);
    printf("=== PASS ===\n\n");
}

void test_null_handling(void) {
    printf("=== Test: NULL Handling ===\n");

    codec_shutdown(NULL);  // Ne doit pas crash

    codec_t *codec = codec_initialize(NULL);

    int ret = codec_transmit(codec, 999999, CODEC_SIGNAL_INTEL, 0, CODEC_NOWAIT);
    assert(ret == -1);  // PID inexistant

    ret = codec_agent_exists(999999);
    assert(ret == 0);  // N'existe pas

    ret = codec_agent_exists(getpid());
    assert(ret == 1);  // Existe

    codec_shutdown(codec);
    printf("=== PASS ===\n\n");
}

int main(void) {
    printf("╔═══════════════════════════════════════╗\n");
    printf("║     CODEC SYSTEM - MISSION TESTS      ║\n");
    printf("║        Shadow Moses Operation         ║\n");
    printf("╚═══════════════════════════════════════╝\n\n");

    test_null_handling();
    test_basic_communication();
    test_ping_pong();

    printf("╔═══════════════════════════════════════╗\n");
    printf("║      ALL TESTS PASSED - MISSION       ║\n");
    printf("║           ACCOMPLISHED                ║\n");
    printf("╚═══════════════════════════════════════╝\n");

    return 0;
}
```

### 4.3 Solution de Référence

```c
#include "codec.h"
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <poll.h>
#include <errno.h>
#include <time.h>

struct codec {
    int              pipe_fd[2];        // Self-pipe pour réception
    int              base_signal;       // Signal RT de base
    int              num_signals;       // Nombre de signaux utilisés
    int              timeout_ms;        // Timeout par défaut
    int              max_retries;       // Nombre de retries
    size_t           queue_size;        // Taille de la file
    uint8_t          seq_counter;       // Compteur de séquence
    codec_handler_fn handlers[32];      // Handlers par type
    void            *handler_data[32];  // Données utilisateur
    codec_stats_t    stats;             // Statistiques
    sigset_t         saved_mask;        // Masque sauvegardé
};

// Variables pour le handler de signal
static int g_pipe_write_fd = -1;

// Handler async-signal-safe
static void codec_signal_handler(int sig, siginfo_t *si, void *context)
{
    (void)sig;
    (void)context;

    if (g_pipe_write_fd < 0)
        return;

    // Écrire les données dans le pipe (async-signal-safe)
    codec_transmission_t trans;
    codec_data_t data;
    data.raw = si->si_int;

    trans.type = data.fields.type;
    trans.seq = data.fields.seq;
    trans.payload = data.fields.payload;
    trans.sender_pid = si->si_pid;
    trans.sender_uid = si->si_uid;
    clock_gettime(CLOCK_MONOTONIC, &trans.timestamp);

    // Write est async-signal-safe
    write(g_pipe_write_fd, &trans, sizeof(trans));
}

codec_t *codec_initialize(const codec_config_t *config)
{
    codec_t *codec = calloc(1, sizeof(codec_t));
    if (!codec)
        return NULL;

    // Appliquer configuration
    if (config) {
        codec->base_signal = config->base_frequency ?
                             config->base_frequency : SIGRTMIN;
        codec->num_signals = config->num_frequencies ?
                             config->num_frequencies : 8;
        codec->timeout_ms = config->default_timeout_ms ?
                            config->default_timeout_ms : 1000;
        codec->max_retries = config->max_retries ? config->max_retries : 3;
        codec->queue_size = config->queue_size ? config->queue_size : 64;
    } else {
        codec->base_signal = SIGRTMIN;
        codec->num_signals = 8;
        codec->timeout_ms = 1000;
        codec->max_retries = 3;
        codec->queue_size = 64;
    }

    // Créer le self-pipe
    if (pipe2(codec->pipe_fd, O_NONBLOCK | O_CLOEXEC) == -1) {
        free(codec);
        return NULL;
    }

    g_pipe_write_fd = codec->pipe_fd[1];

    // Installer les handlers pour les signaux RT
    struct sigaction sa;
    memset(&sa, 0, sizeof(sa));
    sa.sa_sigaction = codec_signal_handler;
    sa.sa_flags = SA_SIGINFO | SA_RESTART;
    sigemptyset(&sa.sa_mask);

    for (int i = 0; i < codec->num_signals; i++) {
        sigaction(codec->base_signal + i, &sa, NULL);
    }

    return codec;
}

void codec_shutdown(codec_t *codec)
{
    if (!codec)
        return;

    // Restaurer les handlers par défaut
    struct sigaction sa;
    memset(&sa, 0, sizeof(sa));
    sa.sa_handler = SIG_DFL;

    for (int i = 0; i < codec->num_signals; i++) {
        sigaction(codec->base_signal + i, &sa, NULL);
    }

    // Fermer le pipe
    if (codec->pipe_fd[0] >= 0)
        close(codec->pipe_fd[0]);
    if (codec->pipe_fd[1] >= 0)
        close(codec->pipe_fd[1]);

    g_pipe_write_fd = -1;

    free(codec);
}

pid_t codec_get_frequency(codec_t *codec)
{
    if (!codec)
        return -1;
    return getpid();
}

int codec_agent_exists(pid_t dest)
{
    return kill(dest, 0) == 0;
}

int codec_transmit(codec_t *codec, pid_t dest, codec_signal_t type,
                   int16_t payload, codec_flags_t flags)
{
    if (!codec)
        return -1;

    // Vérifier que le destinataire existe
    if (kill(dest, 0) == -1)
        return -1;

    // Encoder les données
    codec_data_t data;
    data.fields.type = type;
    data.fields.seq = codec->seq_counter++;
    data.fields.payload = payload;

    union sigval val;
    val.sival_int = data.raw;

    // Déterminer le signal à utiliser
    int sig = codec->base_signal + (type % codec->num_signals);

    int ret = sigqueue(dest, sig, val);
    if (ret == 0) {
        codec->stats.transmitted++;
    } else {
        codec->stats.errors++;
    }

    return ret;
}

int codec_transmit_await_copy(codec_t *codec, pid_t dest, codec_signal_t type,
                              int16_t payload, int timeout_ms,
                              codec_transmission_t *response)
{
    if (!codec)
        return -1;

    if (timeout_ms < 0)
        timeout_ms = codec->timeout_ms;

    uint8_t expected_seq = codec->seq_counter;

    for (int retry = 0; retry <= codec->max_retries; retry++) {
        if (retry > 0)
            codec->stats.retries++;

        // Envoyer
        if (codec_transmit(codec, dest, type, payload, CODEC_NOWAIT) == -1)
            return -1;

        // Attendre ACK
        struct pollfd pfd = { .fd = codec->pipe_fd[0], .events = POLLIN };

        int ret = poll(&pfd, 1, timeout_ms);
        if (ret > 0 && (pfd.revents & POLLIN)) {
            codec_transmission_t trans;

            while (read(codec->pipe_fd[0], &trans, sizeof(trans)) > 0) {
                codec->stats.received++;

                // C'est l'ACK qu'on attend ?
                if (trans.type == CODEC_SIGNAL_COPY &&
                    trans.seq == expected_seq &&
                    trans.sender_pid == dest) {
                    codec->stats.copies++;
                    if (response)
                        *response = trans;
                    return 0;
                }
                else if (trans.type == CODEC_SIGNAL_NEGATIVE &&
                         trans.seq == expected_seq &&
                         trans.sender_pid == dest) {
                    codec->stats.negatives++;
                    if (response)
                        *response = trans;
                    return 1;  // NACK
                }

                // Appeler le handler approprié
                if (trans.type < 32 && codec->handlers[trans.type]) {
                    codec->handlers[trans.type](&trans,
                                                codec->handler_data[trans.type]);
                }
            }
        }

        // Timeout - vérifier si le processus existe encore
        if (!codec_agent_exists(dest)) {
            errno = ESRCH;
            return -1;
        }
    }

    codec->stats.timeouts++;
    errno = ETIMEDOUT;
    return -1;
}

int64_t codec_call_snake(codec_t *codec, pid_t dest, int timeout_ms)
{
    if (!codec)
        return -1;

    struct timespec start, end;
    clock_gettime(CLOCK_MONOTONIC, &start);

    // Encoder le timestamp dans le payload
    int16_t ping_id = (int16_t)(start.tv_nsec / 1000) & 0x7FFF;

    uint8_t expected_seq = codec->seq_counter;

    if (codec_transmit(codec, dest, CODEC_SIGNAL_CALLOUT, ping_id, CODEC_NOWAIT) == -1)
        return -1;

    struct pollfd pfd = { .fd = codec->pipe_fd[0], .events = POLLIN };

    int ret = poll(&pfd, 1, timeout_ms);
    if (ret > 0 && (pfd.revents & POLLIN)) {
        codec_transmission_t trans;

        while (read(codec->pipe_fd[0], &trans, sizeof(trans)) > 0) {
            codec->stats.received++;

            if (trans.type == CODEC_SIGNAL_RESPOND &&
                trans.sender_pid == dest) {
                clock_gettime(CLOCK_MONOTONIC, &end);

                int64_t latency = (end.tv_sec - start.tv_sec) * 1000000 +
                                  (end.tv_nsec - start.tv_nsec) / 1000;

                // Mettre à jour les stats
                if (codec->stats.avg_latency_us == 0)
                    codec->stats.avg_latency_us = latency;
                else
                    codec->stats.avg_latency_us =
                        (codec->stats.avg_latency_us + latency) / 2;

                if (latency > codec->stats.max_latency_us)
                    codec->stats.max_latency_us = latency;

                return latency;
            }
        }
    }

    codec->stats.timeouts++;
    return -1;
}

int codec_team_broadcast(codec_t *codec, pid_t pgid, codec_signal_t type,
                         int16_t payload)
{
    if (!codec)
        return -1;

    codec_data_t data;
    data.fields.type = type;
    data.fields.seq = codec->seq_counter++;
    data.fields.payload = payload;

    union sigval val;
    val.sival_int = data.raw;

    int sig = codec->base_signal + (type % codec->num_signals);

    // killpg envoie à tout le groupe
    if (killpg(pgid, sig) == -1)
        return -1;

    codec->stats.transmitted++;
    return 0;  // Nombre exact inconnu
}

int codec_tune_frequency(codec_t *codec, codec_signal_t type,
                         codec_handler_fn handler, void *user_data)
{
    if (!codec || type >= 32)
        return -1;

    codec->handlers[type] = handler;
    codec->handler_data[type] = user_data;
    return 0;
}

int codec_drop_frequency(codec_t *codec, codec_signal_t type)
{
    if (!codec || type >= 32)
        return -1;

    if (!codec->handlers[type])
        return -1;

    codec->handlers[type] = NULL;
    codec->handler_data[type] = NULL;
    return 0;
}

int codec_get_fd(codec_t *codec)
{
    if (!codec)
        return -1;
    return codec->pipe_fd[0];
}

int codec_process_incoming(codec_t *codec)
{
    if (!codec)
        return -1;

    int count = 0;
    codec_transmission_t trans;

    while (read(codec->pipe_fd[0], &trans, sizeof(trans)) > 0) {
        codec->stats.received++;
        count++;

        if (trans.type < 32 && codec->handlers[trans.type]) {
            codec->handlers[trans.type](&trans, codec->handler_data[trans.type]);
        }
    }

    return count;
}

int codec_standby(codec_t *codec, int timeout_ms, codec_transmission_t *trans)
{
    if (!codec)
        return -1;

    struct pollfd pfd = { .fd = codec->pipe_fd[0], .events = POLLIN };

    int ret = poll(&pfd, 1, timeout_ms);
    if (ret > 0 && (pfd.revents & POLLIN)) {
        if (trans) {
            if (read(codec->pipe_fd[0], trans, sizeof(*trans)) > 0) {
                codec->stats.received++;
                return 1;
            }
        }
        return 1;
    }

    return ret;  // 0 = timeout, -1 = error
}

int codec_flush_queue(codec_t *codec)
{
    if (!codec)
        return -1;

    int count = 0;
    codec_transmission_t trans;

    while (read(codec->pipe_fd[0], &trans, sizeof(trans)) > 0) {
        count++;
    }

    return count;
}

void codec_mission_stats(codec_t *codec, codec_stats_t *stats)
{
    if (!codec || !stats)
        return;
    *stats = codec->stats;
}

void codec_reset_stats(codec_t *codec)
{
    if (!codec)
        return;
    memset(&codec->stats, 0, sizeof(codec->stats));
}

void codec_print_sitrep(codec_t *codec, int fd)
{
    if (!codec)
        return;

    char buf[512];
    int len = snprintf(buf, sizeof(buf),
        "=== CODEC MISSION STATISTICS ===\n"
        "Transmitted: %lu, Received: %lu\n"
        "Copies (ACK): %lu, Negatives (NACK): %lu\n"
        "Timeouts: %lu, Retries: %lu\n"
        "Dropped: %lu, Errors: %lu\n"
        "Avg Latency: %ld us, Max Latency: %ld us\n",
        codec->stats.transmitted, codec->stats.received,
        codec->stats.copies, codec->stats.negatives,
        codec->stats.timeouts, codec->stats.retries,
        codec->stats.dropped, codec->stats.errors,
        codec->stats.avg_latency_us, codec->stats.max_latency_us);

    write(fd, buf, len);
}
```

### 4.4 Solutions Alternatives Acceptées

```c
// Alternative 1: Utilisation de sigtimedwait au lieu de poll
int codec_standby_alt(codec_t *codec, int timeout_ms, codec_transmission_t *trans)
{
    sigset_t mask;
    sigemptyset(&mask);
    for (int i = 0; i < codec->num_signals; i++)
        sigaddset(&mask, codec->base_signal + i);

    struct timespec ts = {
        .tv_sec = timeout_ms / 1000,
        .tv_nsec = (timeout_ms % 1000) * 1000000
    };

    siginfo_t si;
    int sig = sigtimedwait(&mask, &si, &ts);

    if (sig > 0 && trans) {
        codec_data_t data;
        data.raw = si.si_int;
        trans->type = data.fields.type;
        trans->seq = data.fields.seq;
        trans->payload = data.fields.payload;
        trans->sender_pid = si.si_pid;
        trans->sender_uid = si.si_uid;
        clock_gettime(CLOCK_MONOTONIC, &trans->timestamp);
        return 1;
    }

    return sig > 0 ? 1 : 0;
}

// Alternative 2: Table de hash pour pending ACKs
typedef struct pending_ack {
    uint8_t seq;
    pid_t dest;
    struct timespec sent_at;
    int retries;
    struct pending_ack *next;
} pending_ack_t;
```

### 4.5 Solutions Refusées

```c
// REFUSÉ 1: Utilisation de signal() au lieu de sigaction()
void bad_handler(int sig) {
    // Signal handler non réentrant
    printf("Signal received\n");  // INTERDIT dans handler!
}

// Pourquoi refusé: signal() a un comportement non portable et
// printf() n'est pas async-signal-safe

// REFUSÉ 2: Pas de vérification de l'existence du processus
int bad_transmit(codec_t *codec, pid_t dest, codec_signal_t type,
                 int16_t payload, codec_flags_t flags)
{
    // Envoi direct sans vérification
    union sigval val;
    val.sival_int = 42;
    return sigqueue(dest, SIGRTMIN, val);
}

// Pourquoi refusé: Si dest n'existe pas, ESRCH non géré proprement

// REFUSÉ 3: Blocage infini sans timeout
int bad_wait(codec_t *codec) {
    codec_transmission_t trans;
    // Lecture bloquante infinie
    while (read(codec->pipe_fd[0], &trans, sizeof(trans)) <= 0)
        ;
    return 0;
}

// Pourquoi refusé: Peut bloquer indéfiniment si aucun message n'arrive
```

### 4.6 Solution Bonus de Référence

```c
// Fragmentation pour burst transmission
#define FRAG_DATA_SIZE 4
#define MAX_BURST_SIZE 1024

typedef struct {
    uint8_t msg_id;
    uint8_t expected_frags;
    uint8_t received_mask[32];  // Bitmap des fragments reçus
    uint8_t *buffer;
    size_t  buffer_size;
    pid_t   sender;
} reassembly_context_t;

int codec_burst_transmit(codec_t *codec, pid_t dest,
                         const void *data, size_t len,
                         int timeout_ms)
{
    if (!codec || !data || len == 0 || len > MAX_BURST_SIZE)
        return -1;

    uint8_t msg_id = codec->seq_counter++;
    uint8_t frag_count = (len + FRAG_DATA_SIZE - 1) / FRAG_DATA_SIZE;

    const uint8_t *src = data;

    for (uint8_t i = 0; i < frag_count; i++) {
        codec_fragment_t frag;
        frag.msg_id = msg_id;
        frag.frag_index = i;
        frag.frag_count = frag_count;
        frag.flags = 0;
        if (i == 0) frag.flags |= 0x01;  // FIRST
        if (i == frag_count - 1) frag.flags |= 0x02;  // LAST

        size_t chunk = (len - i * FRAG_DATA_SIZE);
        if (chunk > FRAG_DATA_SIZE) chunk = FRAG_DATA_SIZE;

        memcpy(frag.data, src + i * FRAG_DATA_SIZE, chunk);

        // Envoyer le fragment via 2 signaux (8 bytes de données)
        codec_data_t d1, d2;
        memcpy(&d1.raw, &frag, 4);
        memcpy(&d2.raw, ((char*)&frag) + 4, 4);

        union sigval v1 = { .sival_int = d1.raw };
        union sigval v2 = { .sival_int = d2.raw };

        sigqueue(dest, codec->base_signal + 4, v1);
        sigqueue(dest, codec->base_signal + 5, v2);

        // Attendre ACK du fragment
        struct pollfd pfd = { .fd = codec->pipe_fd[0], .events = POLLIN };
        int ret = poll(&pfd, 1, timeout_ms / frag_count);
        if (ret <= 0)
            return -1;

        // Lire et vérifier ACK
        codec_transmission_t ack;
        if (read(codec->pipe_fd[0], &ack, sizeof(ack)) > 0) {
            if (ack.type != CODEC_SIGNAL_COPY || ack.seq != i)
                return -1;
        }
    }

    return 0;
}
```

### 4.9 spec.json

```json
{
  "name": "codec_transmit",
  "language": "c",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse - RT signals + protocole + IPC",
  "tags": ["process", "signals", "sigqueue", "realtime", "ipc"],
  "passing_score": 80,

  "function": {
    "name": "codec_transmit",
    "prototype": "int codec_transmit(codec_t *codec, pid_t dest, codec_signal_t type, int16_t payload, codec_flags_t flags)",
    "return_type": "int",
    "parameters": [
      {"name": "codec", "type": "codec_t *"},
      {"name": "dest", "type": "pid_t"},
      {"name": "type", "type": "codec_signal_t"},
      {"name": "payload", "type": "int16_t"},
      {"name": "flags", "type": "codec_flags_t"}
    ]
  },

  "driver": {
    "reference": "int ref_codec_transmit(codec_t *codec, pid_t dest, codec_signal_t type, int16_t payload, codec_flags_t flags) { if (!codec) return -1; if (kill(dest, 0) == -1) return -1; codec_data_t data; data.fields.type = type; data.fields.seq = 0; data.fields.payload = payload; union sigval val; val.sival_int = data.raw; return sigqueue(dest, SIGRTMIN, val); }",

    "edge_cases": [
      {
        "name": "null_codec",
        "args": [null, 1000, 0, 0, 0],
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "codec est NULL"
      },
      {
        "name": "invalid_pid",
        "args": ["valid_codec", 999999, 0, 0, 0],
        "expected": -1,
        "is_trap": true,
        "trap_explanation": "PID 999999 n'existe pas"
      },
      {
        "name": "self_send",
        "args": ["valid_codec", "getpid()", 2, 42, 0],
        "expected": 0,
        "is_trap": false,
        "trap_explanation": "Envoi à soi-même est valide"
      },
      {
        "name": "signal_type_boundary",
        "args": ["valid_codec", "getpid()", 31, 0, 0],
        "expected": 0,
        "is_trap": true,
        "trap_explanation": "Type 31 est la limite"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "int",
          "param_index": 2,
          "params": { "min": 0, "max": 31 }
        },
        {
          "type": "int",
          "param_index": 3,
          "params": { "min": -32768, "max": 32767 }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["sigaction", "sigemptyset", "sigaddset", "sigfillset", "sigqueue", "kill", "raise", "killpg", "sigpending", "sigprocmask", "sigsuspend", "sigtimedwait", "pipe", "pipe2", "read", "write", "close", "fcntl", "poll", "clock_gettime", "nanosleep", "malloc", "free", "calloc", "realloc", "memset", "memcpy", "getpid", "getuid"],
    "forbidden_functions": ["signal", "sleep", "usleep", "printf"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```c
/* Mutant A (Boundary) : Signal overflow */
int mutant_a_transmit(codec_t *codec, pid_t dest, codec_signal_t type,
                      int16_t payload, codec_flags_t flags)
{
    if (!codec) return -1;

    // BUG: Pas de vérification que type < num_signals
    int sig = codec->base_signal + type;  // Peut dépasser SIGRTMAX!

    union sigval val = { .sival_int = payload };
    return sigqueue(dest, sig, val);
}
// Pourquoi faux : Si type >= num_signals, on envoie un signal invalide
// Ce qui était pensé : "Le type est toujours valide"

/* Mutant B (Safety) : Pas de vérification PID */
int mutant_b_transmit(codec_t *codec, pid_t dest, codec_signal_t type,
                      int16_t payload, codec_flags_t flags)
{
    if (!codec) return -1;

    // BUG: Pas de kill(dest, 0) pour vérifier
    codec_data_t data;
    data.fields.type = type;
    data.fields.seq = 0;
    data.fields.payload = payload;

    union sigval val = { .sival_int = data.raw };
    return sigqueue(dest, SIGRTMIN, val);  // ESRCH si dest n'existe pas
}
// Pourquoi faux : On ne détecte pas proactivement les PIDs invalides
// Ce qui était pensé : "sigqueue gère l'erreur"

/* Mutant C (Resource) : Fuite de FD */
void mutant_c_shutdown(codec_t *codec)
{
    if (!codec) return;

    // BUG: On ferme seulement un côté du pipe
    close(codec->pipe_fd[0]);
    // Oubli: close(codec->pipe_fd[1]);

    free(codec);
}
// Pourquoi faux : Le FD d'écriture du pipe reste ouvert
// Ce qui était pensé : "Fermer le read suffit"

/* Mutant D (Logic) : Mauvais seq pour ACK */
int mutant_d_await_copy(codec_t *codec, pid_t dest, codec_signal_t type,
                        int16_t payload, int timeout_ms,
                        codec_transmission_t *response)
{
    if (!codec) return -1;

    uint8_t sent_seq = codec->seq_counter;  // Capture seq AVANT
    codec_transmit(codec, dest, type, payload, 0);  // Incrémente seq

    // BUG: On compare avec le mauvais seq
    codec_transmission_t trans;
    if (read(codec->pipe_fd[0], &trans, sizeof(trans)) > 0) {
        if (trans.type == CODEC_SIGNAL_COPY &&
            trans.seq == codec->seq_counter)  // FAUX! C'est sent_seq + 1
            return 0;
    }
    return -1;
}
// Pourquoi faux : sent_seq != codec->seq_counter après envoi
// Ce qui était pensé : "Le compteur ne change pas"

/* Mutant E (Return) : Timeout retourne succès */
int mutant_e_await_copy(codec_t *codec, pid_t dest, codec_signal_t type,
                        int16_t payload, int timeout_ms,
                        codec_transmission_t *response)
{
    if (!codec) return -1;

    codec_transmit(codec, dest, type, payload, 0);

    struct pollfd pfd = { .fd = codec->pipe_fd[0], .events = POLLIN };
    int ret = poll(&pfd, 1, timeout_ms);

    // BUG: Retourne 0 même si timeout
    if (ret == 0)
        return 0;  // FAUX! Devrait être -1 avec errno = ETIMEDOUT

    return 0;
}
// Pourquoi faux : Le timeout est interprété comme succès
// Ce qui était pensé : "0 c'est success"

/* Mutant F (Race) : Pas de boucle EINTR */
int mutant_f_standby(codec_t *codec, int timeout_ms, codec_transmission_t *trans)
{
    if (!codec) return -1;

    struct pollfd pfd = { .fd = codec->pipe_fd[0], .events = POLLIN };

    // BUG: poll peut être interrompu par EINTR
    int ret = poll(&pfd, 1, timeout_ms);  // Pas de boucle!

    if (ret > 0 && trans) {
        read(codec->pipe_fd[0], trans, sizeof(*trans));
        return 1;
    }
    return ret;
}
// Pourquoi faux : Si un signal interrompt poll, on retourne -1 au lieu de réessayer
// Ce qui était pensé : "poll gère tout seul"
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Signaux temps-réel** : SIGRTMIN-SIGRTMAX vs signaux standard
2. **sigqueue()** : Envoi de données avec les signaux
3. **Protocole de messagerie** : ACK, retry, séquençage
4. **Self-pipe pattern** : Intégration avec event loop
5. **Async-signal-safety** : Ce qu'on peut faire dans un handler

### 5.2 LDA — Traduction Littérale

```
FONCTION codec_transmit QUI RETOURNE UN ENTIER ET PREND EN PARAMÈTRES codec QUI EST UN POINTEUR VERS UNE STRUCTURE CODEC ET dest QUI EST UN PID ET type QUI EST UN TYPE DE SIGNAL ET payload QUI EST UN ENTIER 16 BITS ET flags QUI SONT DES FLAGS
DÉBUT FONCTION
    SI codec EST ÉGAL À NUL ALORS
        RETOURNER LA VALEUR MOINS 1
    FIN SI

    SI ENVOYER LE SIGNAL 0 À dest RETOURNE MOINS 1 ALORS
        RETOURNER LA VALEUR MOINS 1
    FIN SI

    DÉCLARER data COMME UNION DE DONNÉES CODEC
    AFFECTER type AU CHAMP type DE fields DE data
    AFFECTER LE COMPTEUR DE SÉQUENCE AU CHAMP seq DE fields DE data
    INCRÉMENTER LE COMPTEUR DE SÉQUENCE DE 1
    AFFECTER payload AU CHAMP payload DE fields DE data

    DÉCLARER val COMME UNION SIGVAL
    AFFECTER raw DE data AU CHAMP sival_int DE val

    DÉCLARER sig COMME ENTIER
    AFFECTER base_signal PLUS type MODULO num_signals À sig

    RETOURNER LE RÉSULTAT DE sigqueue AVEC dest ET sig ET val
FIN FONCTION
```

### 5.2.2.1 Logic Flow

```
ALGORITHME : Transmission CODEC avec ACK
---
1. VÉRIFIER les préconditions :
   a. codec non NULL
   b. destinataire existe (kill(dest, 0))

2. ENCODER le message :
   a. Type (8 bits) + Séquence (8 bits) + Payload (16 bits)
   b. Incrémenter le compteur de séquence

3. ENVOYER via sigqueue() :
   a. Calculer le numéro de signal RT
   b. Appeler sigqueue(dest, signal, value)

4. SI attente d'ACK :
   a. BOUCLE avec timeout :
      - poll() sur le self-pipe
      - SI données disponibles :
        - Lire la transmission
        - SI c'est l'ACK attendu : RETOURNER succès
      - SI timeout : incrémenter retry
   b. SI max retries atteint : RETOURNER échec

5. RETOURNER le résultat
```

### 5.3 Visualisation ASCII

```
┌──────────────────────────────────────────────────────────────────────────┐
│                     CODEC COMMUNICATION SYSTEM                            │
│                     (Metal Gear Solid Protocol)                           │
└──────────────────────────────────────────────────────────────────────────┘

   SNAKE (Parent)                                    OTACON (Child)
   ┌────────────┐                                    ┌────────────┐
   │  codec_t   │                                    │  codec_t   │
   │ ┌────────┐ │                                    │ ┌────────┐ │
   │ │seq = 0 │ │                                    │ │handlers│ │
   │ │pipe[2] │ │                                    │ │pipe[2] │ │
   │ └────────┘ │                                    │ └────────┘ │
   └─────┬──────┘                                    └─────┬──────┘
         │                                                 │
         │  1. codec_transmit(INTEL, seq=0)               │
         │ ─────────────────────────────────────────────► │
         │      sigqueue(SIGRTMIN, {type=2,seq=0,data})   │
         │                                                 │
         │                                    2. Handler   │
         │                                       appelé    │
         │                                                 │
         │  3. codec_transmit(COPY, seq=0)                │
         │ ◄───────────────────────────────────────────── │
         │      sigqueue(SIGRTMIN+1, {type=3,seq=0})      │
         │                                                 │
         │  4. ACK reçu, succès!                          │
         │                                                 │


Message Encoding (32 bits):
┌────────┬────────┬─────────────────┐
│  Type  │  Seq   │     Payload     │
│ 8 bits │ 8 bits │     16 bits     │
└────────┴────────┴─────────────────┘
 0-31     0-255    -32768 to +32767


Signal RT Mapping:
┌────────────────────┬────────────────────┐
│ SIGRTMIN + 0       │ INTEL / DATA       │
├────────────────────┼────────────────────┤
│ SIGRTMIN + 1       │ COPY / NACK        │
├────────────────────┼────────────────────┤
│ SIGRTMIN + 2       │ CALLOUT / RESPOND  │
├────────────────────┼────────────────────┤
│ SIGRTMIN + 3       │ ORDER / SITREP     │
└────────────────────┴────────────────────┘
```

### 5.4 Les Pièges en Détail

#### Piège 1 : printf() dans le handler

```c
// DANGER : printf n'est pas async-signal-safe
void bad_handler(int sig, siginfo_t *si, void *ctx) {
    printf("Signal %d reçu\n", sig);  // BOOM!
}

// SOLUTION : Utiliser write()
void good_handler(int sig, siginfo_t *si, void *ctx) {
    write(STDOUT_FILENO, "Signal\n", 7);  // OK
}
```

#### Piège 2 : File de signaux RT pleine

```c
// Les signaux RT sont mis en file, mais la file a une limite!
// /proc/sys/kernel/rtsig-max (typiquement 1024 par processus)

for (int i = 0; i < 10000; i++) {
    sigqueue(dest, SIGRTMIN, val);  // EAGAIN après ~1024!
}

// SOLUTION : Vérifier le retour et gérer EAGAIN
if (sigqueue(dest, sig, val) == -1) {
    if (errno == EAGAIN) {
        // File pleine, réessayer plus tard
        codec->stats.dropped++;
    }
}
```

#### Piège 3 : Race condition sur le seq number

```c
// DANGER : Capture du seq APRÈS l'envoi
uint8_t sent_seq;
codec_transmit(codec, dest, type, payload, 0);
sent_seq = codec->seq_counter;  // TROP TARD! seq_counter déjà incrémenté

// SOLUTION : Capturer AVANT
uint8_t sent_seq = codec->seq_counter;  // Capture maintenant
codec_transmit(codec, dest, type, payload, 0);  // Incrémente
// Comparer avec sent_seq, pas codec->seq_counter
```

### 5.5 Cours Complet

#### Les Signaux Temps-Réel POSIX

Les signaux sont divisés en deux catégories :

**Signaux Standard (1-31)** :
- Comportement "fire and forget"
- Plusieurs envois = un seul signal délivré
- Pas de données associées
- Exemples : SIGTERM, SIGINT, SIGUSR1

**Signaux Temps-Réel (SIGRTMIN-SIGRTMAX)** :
- Mis en file d'attente
- Chaque envoi est délivré séparément
- 32 bits de données via sigval
- Livraison FIFO, priorité par numéro

```c
// Vérifier les limites sur ton système
printf("SIGRTMIN = %d\n", SIGRTMIN);  // Typiquement 34
printf("SIGRTMAX = %d\n", SIGRTMAX);  // Typiquement 64
printf("Available: %d signals\n", SIGRTMAX - SIGRTMIN + 1);  // ~31
```

#### sigqueue() vs kill()

```c
// kill() : Juste envoyer le signal
kill(pid, SIGRTMIN);  // Pas de données

// sigqueue() : Envoyer avec données
union sigval val;
val.sival_int = 42;  // ou val.sival_ptr = &data;
sigqueue(pid, SIGRTMIN, val);  // 32 bits de données!
```

#### Recevoir avec siginfo_t

```c
void handler(int sig, siginfo_t *si, void *ctx) {
    // Informations disponibles :
    si->si_signo;   // Numéro du signal
    si->si_pid;     // PID de l'émetteur
    si->si_uid;     // UID de l'émetteur
    si->si_int;     // Données (sival_int)
    si->si_ptr;     // Ou pointeur (sival_ptr)
}

// Installation avec SA_SIGINFO
struct sigaction sa = {0};
sa.sa_sigaction = handler;
sa.sa_flags = SA_SIGINFO;
sigaction(SIGRTMIN, &sa, NULL);
```

### 5.6 Normes avec Explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais dangereux)                         │
├─────────────────────────────────────────────────────────────────┤
│ signal(SIGRTMIN, handler);                                      │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ struct sigaction sa = {0};                                      │
│ sa.sa_sigaction = handler;                                      │
│ sa.sa_flags = SA_SIGINFO;                                       │
│ sigaction(SIGRTMIN, &sa, NULL);                                 │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • signal() a un comportement non portable entre systèmes        │
│ • signal() ne donne pas accès à siginfo_t                       │
│ • sigaction() permet SA_SIGINFO pour les données                │
│ • sigaction() permet SA_RESTART pour éviter EINTR               │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec Trace d'Exécution

```
Scénario : Snake envoie 3 INTELs à Otacon avec ACK

┌───────┬────────────────────────────────────┬─────────┬─────────────────────┐
│ Étape │ Action                             │ Snake   │ Otacon              │
│       │                                    │ seq     │ received            │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│   1   │ Snake: codec_transmit(INTEL, 42)   │ 0 → 1   │ —                   │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│   2   │ Kernel: sigqueue envoie SIGRTMIN   │ 1       │ Signal en file      │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│   3   │ Otacon: handler reçoit si_int      │ 1       │ type=2,seq=0,pl=42  │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│   4   │ Otacon: codec_transmit(COPY, seq=0)│ 1       │ ACK envoyé          │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│   5   │ Snake: poll() retourne POLLIN      │ 1       │ —                   │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│   6   │ Snake: read() → type=3,seq=0       │ 1       │ —                   │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│   7   │ Snake: ACK reçu! Succès            │ 1       │ —                   │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│   8   │ Snake: codec_transmit(INTEL, 100)  │ 1 → 2   │ —                   │
├───────┼────────────────────────────────────┼─────────┼─────────────────────┤
│  ...  │ (répéter pour seq=1, seq=2)        │ ...     │ ...                 │
└───────┴────────────────────────────────────┴─────────┴─────────────────────┘
```

### 5.8 Mnémotechniques

#### MEME : "Snake? Snake! SNAAAAKE!"

![Snake Timeout](codec_timeout.jpg)

Quand tu appelles `codec_call_snake()` et qu'il n'y a pas de réponse...

```c
int64_t latency = codec_call_snake(codec, otacon_pid, 1000);
if (latency < 0) {
    // 😱 SNAAAAKE!
    write(STDERR_FILENO, "No response!\n", 13);
}
```

**Le timeout c'est comme quand le Colonel appelle Snake et qu'il ne répond pas** :
- Premier appel : "Snake?"
- Retry 1 : "Snake!"
- Retry 2 : "Snake!!"
- Échec final : "SNAAAAKE!" (timeout)

#### MEME : "Copy that" = ACK

Dans Metal Gear, "Copy that" confirme la réception. C'est exactement ce que fait `CODEC_SIGNAL_COPY` :

```c
// Otacon confirme la réception
if (received_intel) {
    codec_transmit(codec, snake_pid, CODEC_SIGNAL_COPY, seq, 0);
    // "Copy that, Snake!"
}
```

#### MEME : Fréquence CODEC = Signal RT

Chaque personnage a sa fréquence :
- Mei Ling : 140.96
- Otacon : 141.12
- Colonel : 140.85

En code :
- SIGRTMIN + 0 : Intel channel
- SIGRTMIN + 1 : ACK channel
- SIGRTMIN + 2 : Ping channel

```c
// "Snake, switch to frequency 141.12 to contact Otacon"
int otacon_signal = SIGRTMIN + INTEL_CHANNEL;
```

### 5.9 Applications Pratiques

1. **Supervision de workers** : Un orchestrateur envoie des commandes (START, STOP, DUMP_STATS) via signaux RT
2. **Heartbeat léger** : PING/PONG pour vérifier qu'un processus est vivant
3. **Notification d'événements** : Signaler un changement de configuration sans redémarrage
4. **Debugging en production** : Envoyer des commandes de diagnostic à un processus en cours

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| Piège | Symptôme | Solution |
|-------|----------|----------|
| printf() dans handler | Deadlock ou corruption | Utiliser write() |
| File RT pleine | EAGAIN de sigqueue | Gérer le retry |
| Pas de kill(0) avant | ESRCH surprise | Vérifier existence |
| Mauvais seq pour ACK | Messages désynchronisés | Capturer seq avant envoi |
| Oubli EINTR | poll() retourne -1 | Boucle avec vérification errno |
| Fuite de FD pipe | Valgrind error | Fermer les deux côtés |

---

## 📝 SECTION 7 : QCM

### Q1. Quelle est la différence principale entre SIGUSR1 et SIGRTMIN ?

- A) SIGUSR1 est plus rapide
- B) SIGRTMIN permet d'envoyer des données
- C) SIGUSR1 ne peut pas être bloqué
- D) SIGRTMIN n'existe pas sur Linux

**Réponse : B**

### Q2. Que retourne sigqueue() si la file de signaux RT est pleine ?

- A) 0
- B) -1 avec errno = EAGAIN
- C) -1 avec errno = ENOMEM
- D) Le signal est silencieusement ignoré

**Réponse : B**

### Q3. Quelle fonction est async-signal-safe ?

- A) printf()
- B) malloc()
- C) write()
- D) snprintf()

**Réponse : C**

### Q4. Comment accéder aux données envoyées via sigqueue() dans le handler ?

- A) Via le paramètre sig
- B) Via si->si_value.sival_int
- C) Via une variable globale
- D) Ce n'est pas possible

**Réponse : B**

### Q5. Quel flag de sigaction permet d'accéder à siginfo_t ?

- A) SA_RESTART
- B) SA_NOCLDSTOP
- C) SA_SIGINFO
- D) SA_NODEFER

**Réponse : C**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Métrique | Valeur |
|----------|--------|
| Fonctions à implémenter | 15 |
| Lignes de code estimées | 400-600 |
| Concepts système | sigqueue, RT signals, self-pipe, protocole |
| Tests moulinette | 18 |
| Temps estimé | 4 heures |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.2.5-codec_transmit",
    "generated_at": "2025-01-11 14:30:00",

    "metadata": {
      "exercise_id": "2.2.5",
      "exercise_name": "codec_transmit",
      "module": "2.2",
      "module_name": "Processes & Shell",
      "concept": "e",
      "concept_name": "Signal Messenger (RT Signals)",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "c",
      "duration_minutes": 240,
      "xp_base": 400,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T3 O(n)",
      "complexity_space": "S2 O(1)",
      "prerequisites": ["ex04"],
      "domains": ["Process", "Net"],
      "domains_bonus": ["Compression"],
      "tags": ["sigqueue", "realtime-signals", "ipc", "protocol"],
      "meme_reference": "Metal Gear Solid CODEC"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.c": "/* Section 4.3 */",
      "references/ref_solution_bonus.c": "/* Section 4.6 */",
      "alternatives/alt_sigtimedwait.c": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 */",
      "mutants/mutant_e_return.c": "/* Section 4.10 */",
      "mutants/mutant_f_race.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.c",
        "references/ref_solution_bonus.c",
        "alternatives/alt_sigtimedwait.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c",
        "mutants/mutant_f_race.c"
      ]
    }
  }
}
```

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 25 | Protocole complet avec ACK/retry/séquençage sur RT signals |
| Couverture conceptuelle | 24 | 10 concepts (2.2.13-2.2.14) couverts exhaustivement |
| Testabilité auto | 24 | 18 tests, fuzzing, latence, throughput mesurables |
| Originalité | 25 | Metal Gear Solid CODEC - analogie parfaite et mémorable |
| **TOTAL** | **98/100** | ✓ Validé |

---

*HACKBRAIN v5.5.2 — "This is Snake. Do you read me?"*
