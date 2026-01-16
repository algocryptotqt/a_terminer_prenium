# [Module 2.2] - Exercise 13: IPC Complete

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex13"
title: "IPC Complete - Inter-Process Communication"
difficulty: moyen
estimated_time: "6 heures"
prerequisite_exercises: ["ex06", "ex07", "ex08"]
concepts_requis: ["pipes", "file descriptors", "fork"]
score_qualite: 97
```

---

## Concepts Couverts

### 2.2.16: Inter-Process Communication Overview (6 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.16.a | IPC mechanisms: Pipes, FIFOs, messages, shared memory, sockets | Vue d'ensemble IPC |
| 2.2.16.b | Criteria: Bandwidth, latency, complexity | Comparaison mecanismes |
| 2.2.16.c | Data vs synchronization IPC | Types d'IPC |
| 2.2.16.d | Related vs unrelated processes | Processus lies/non-lies |
| 2.2.16.e | Local vs network: Unix sockets vs TCP/UDP | IPC local vs reseau |
| 2.2.16.f | Persistence: Process vs kernel vs filesystem | Duree de vie |

### 2.2.17: Pipes (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.17.a | pipe(): Create pipe | Creation de pipe |
| 2.2.17.b | Anonymous pipe: Related processes only | Pipe anonyme |
| 2.2.17.c | Unidirectional: One reader, one writer | Sens unique |
| 2.2.17.d | fd[0]: Read end | Extremite lecture |
| 2.2.17.e | fd[1]: Write end | Extremite ecriture |
| 2.2.17.f | PIPE_BUF: Atomic write size | Taille atomique |
| 2.2.17.g | Blocking behavior: Empty/full | Comportement bloquant |
| 2.2.17.h | EOF: Close write end | Fin de fichier |
| 2.2.17.i | SIGPIPE: Write to closed pipe | Signal broken pipe |
| 2.2.17.j | pipe2(): With flags (O_CLOEXEC) | Pipe avec flags |
| 2.2.17.k | Pipe capacity: 64KB default | Capacite du pipe |
| 2.2.17.l | fcntl F_SETPIPE_SZ: Change size | Modifier capacite |

### 2.2.18: FIFOs (Named Pipes) (9 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.18.a | mkfifo(): Create named pipe | Creation FIFO |
| 2.2.18.b | Filesystem presence: Special file | Fichier special |
| 2.2.18.c | Unrelated processes: Can connect | Processus non-lies |
| 2.2.18.d | Open semantics: Blocks until both ends | Semantique open |
| 2.2.18.e | O_NONBLOCK: Non-blocking open | Ouverture non-bloquante |
| 2.2.18.f | Multiple readers/writers: Possible | Multiples lecteurs/ecrivains |
| 2.2.18.g | Permissions: Mode bits apply | Permissions fichier |
| 2.2.18.h | unlink(): Remove FIFO | Suppression FIFO |
| 2.2.18.i | mkfifoat(): Relative to directory | Creation relative |

### 2.2.19: Message Queues (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.19.a | POSIX mq: Modern API | API POSIX mq |
| 2.2.19.b | mq_open(): Create/open queue | Ouverture file |
| 2.2.19.c | mq_send(): Send message | Envoi message |
| 2.2.19.d | mq_receive(): Receive message | Reception message |
| 2.2.19.e | mq_close(): Close descriptor | Fermeture |
| 2.2.19.f | mq_unlink(): Remove queue | Suppression |
| 2.2.19.g | Message priority: Ordering | Priorite des messages |
| 2.2.19.h | mq_attr: Queue attributes | Attributs de file |
| 2.2.19.i | mq_notify(): Async notification | Notification async |
| 2.2.19.j | System V msgget: Older API | API System V |

### 2.2.20: Shared Memory (11 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.20.a | POSIX shm: Modern API | API POSIX shm |
| 2.2.20.b | shm_open(): Create/open object | Ouverture objet |
| 2.2.20.c | ftruncate(): Set size | Definir taille |
| 2.2.20.d | mmap(): Map to address space | Mapping memoire |
| 2.2.20.e | munmap(): Unmap | Liberation mapping |
| 2.2.20.f | shm_unlink(): Remove object | Suppression objet |
| 2.2.20.g | Synchronization needed: Race conditions | Besoin synchronisation |
| 2.2.20.h | MAP_SHARED vs MAP_PRIVATE | Modes de mapping |
| 2.2.20.i | System V shmget: Older API | API System V shm |
| 2.2.20.j | Performance: Fastest IPC | Performance optimale |
| 2.2.20.k | Persistence: Kernel lifetime | Persistance |

### 2.2.21: Semaphores (11 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.21.a | POSIX named: sem_open() | Semaphore nomme |
| 2.2.21.b | POSIX unnamed: sem_init() | Semaphore anonyme |
| 2.2.21.c | sem_wait(): Decrement (P) | Operation wait |
| 2.2.21.d | sem_post(): Increment (V) | Operation post |
| 2.2.21.e | sem_trywait(): Non-blocking | Essai non-bloquant |
| 2.2.21.f | sem_timedwait(): With timeout | Attente avec timeout |
| 2.2.21.g | sem_getvalue(): Get value | Lecture valeur |
| 2.2.21.h | sem_close(): Close named | Fermeture nomme |
| 2.2.21.i | sem_destroy(): Destroy unnamed | Destruction anonyme |
| 2.2.21.j | sem_unlink(): Remove named | Suppression nomme |
| 2.2.21.k | System V semget: Older API | API System V sem |

---

## Contexte

La communication inter-processus (IPC) est fondamentale dans les systemes Unix pour permettre a des processus distincts de cooperer, d'echanger des donnees et de se synchroniser. Chaque mecanisme IPC presente des compromis differents en termes de performance, complexite et cas d'usage.

Cet exercice vous guidera a travers l'implementation et l'utilisation de tous les mecanismes IPC POSIX: pipes (anonymes et nommes), files de messages, memoire partagee et semaphores. Vous apprendrez quand utiliser chaque mecanisme et comment les combiner efficacement.

---

## Enonce

### Vue d'Ensemble

Implementez une bibliotheque unifiee d'IPC qui encapsule tous les mecanismes de communication inter-processus et fournit une API coherente pour les utiliser.

### Partie 1: Pipes

```c
#include <unistd.h>
#include <fcntl.h>

// Pipe anonyme
typedef struct {
    int read_fd;
    int write_fd;
    size_t buffer_size;
} pipe_t;

pipe_t *pipe_create(void);
pipe_t *pipe_create_with_flags(int flags);  // O_CLOEXEC, O_NONBLOCK
void pipe_destroy(pipe_t *p);

ssize_t pipe_write(pipe_t *p, const void *buf, size_t count);
ssize_t pipe_read(pipe_t *p, void *buf, size_t count);

int pipe_close_read(pipe_t *p);
int pipe_close_write(pipe_t *p);

int pipe_set_nonblocking(pipe_t *p, int read_end, int write_end);
int pipe_get_buffer_size(pipe_t *p, size_t *size);
int pipe_set_buffer_size(pipe_t *p, size_t size);

// Information sur PIPE_BUF
size_t pipe_get_atomic_size(void);

// Demonstration
int demo_pipe_parent_child(void);
int demo_pipe_bidirectional(void);  // Necessite 2 pipes
int demo_pipe_chain(int n_stages);  // Pipeline de N processus
```

### Partie 2: FIFOs (Named Pipes)

```c
#include <sys/stat.h>

typedef struct {
    char path[256];
    int fd;
    int flags;  // O_RDONLY, O_WRONLY, O_RDWR
} fifo_t;

int fifo_create(const char *path, mode_t mode);
int fifo_remove(const char *path);
int fifo_exists(const char *path);

fifo_t *fifo_open(const char *path, int flags);
fifo_t *fifo_open_nonblock(const char *path, int flags);
void fifo_close(fifo_t *f);

ssize_t fifo_write(fifo_t *f, const void *buf, size_t count);
ssize_t fifo_read(fifo_t *f, void *buf, size_t count);

// Demonstration
int demo_fifo_server_client(const char *fifo_path);
int demo_fifo_bidirectional(const char *path1, const char *path2);
int demo_fifo_multiple_writers(const char *fifo_path, int n_writers);
```

### Partie 3: Message Queues

```c
#include <mqueue.h>

typedef struct {
    mqd_t mqd;
    char name[256];
    struct mq_attr attr;
} msgqueue_t;

typedef struct {
    long maxmsg;      // Nombre max de messages
    long msgsize;     // Taille max d'un message
    int flags;        // O_NONBLOCK, etc.
} msgqueue_config_t;

// Creation et ouverture
msgqueue_t *msgqueue_create(const char *name, const msgqueue_config_t *config);
msgqueue_t *msgqueue_open(const char *name, int flags);
void msgqueue_close(msgqueue_t *mq);
int msgqueue_unlink(const char *name);

// Envoi et reception
int msgqueue_send(msgqueue_t *mq, const void *msg, size_t len, unsigned int prio);
int msgqueue_send_timed(msgqueue_t *mq, const void *msg, size_t len,
                        unsigned int prio, int timeout_ms);
ssize_t msgqueue_receive(msgqueue_t *mq, void *msg, size_t len, unsigned int *prio);
ssize_t msgqueue_receive_timed(msgqueue_t *mq, void *msg, size_t len,
                                unsigned int *prio, int timeout_ms);

// Attributs
int msgqueue_get_attr(msgqueue_t *mq, struct mq_attr *attr);
int msgqueue_set_nonblock(msgqueue_t *mq, int nonblock);
long msgqueue_get_count(msgqueue_t *mq);

// Notification asynchrone
typedef void (*msgqueue_notify_fn)(union sigval);
int msgqueue_set_notify(msgqueue_t *mq, msgqueue_notify_fn fn, void *data);
int msgqueue_clear_notify(msgqueue_t *mq);

// Demonstration
int demo_msgqueue_producer_consumer(void);
int demo_msgqueue_priority(void);
int demo_msgqueue_notification(void);
```

### Partie 4: Shared Memory

```c
#include <sys/mman.h>

typedef struct {
    char name[256];
    int fd;
    void *addr;
    size_t size;
    int prot;           // PROT_READ, PROT_WRITE
    int flags;          // MAP_SHARED, MAP_PRIVATE
} sharedmem_t;

// Creation et ouverture
sharedmem_t *sharedmem_create(const char *name, size_t size, int prot);
sharedmem_t *sharedmem_open(const char *name, size_t size, int prot);
void sharedmem_close(sharedmem_t *shm);
int sharedmem_unlink(const char *name);

// Acces
void *sharedmem_get_addr(sharedmem_t *shm);
size_t sharedmem_get_size(sharedmem_t *shm);
int sharedmem_resize(sharedmem_t *shm, size_t new_size);

// Synchronisation (avec memoire partagee)
int sharedmem_sync(sharedmem_t *shm);  // msync

// Memoire partagee anonyme (entre parent/enfant)
sharedmem_t *sharedmem_create_anonymous(size_t size);

// Demonstration
int demo_sharedmem_counter(void);      // Compteur partage
int demo_sharedmem_ringbuffer(void);   // Buffer circulaire
int demo_sharedmem_struct(void);       // Structure partagee
```

### Partie 5: Semaphores

```c
#include <semaphore.h>

// Semaphore nomme
typedef struct {
    sem_t *sem;
    char name[256];
    int is_named;
} semaphore_t;

// Semaphores nommes
semaphore_t *semaphore_create(const char *name, unsigned int value);
semaphore_t *semaphore_open(const char *name);
int semaphore_unlink(const char *name);

// Semaphore anonyme (pour processus lies via shm)
semaphore_t *semaphore_create_anonymous(unsigned int value);
semaphore_t *semaphore_init_in_shm(void *addr, unsigned int value);

// Operations
void semaphore_destroy(semaphore_t *sem);
int semaphore_wait(semaphore_t *sem);              // P operation (decrement)
int semaphore_trywait(semaphore_t *sem);           // Non-blocking wait
int semaphore_timedwait(semaphore_t *sem, int timeout_ms);
int semaphore_post(semaphore_t *sem);              // V operation (increment)
int semaphore_getvalue(semaphore_t *sem, int *value);

// Demonstration
int demo_semaphore_mutex(void);           // Exclusion mutuelle
int demo_semaphore_producer_consumer(void);
int demo_semaphore_readers_writers(void);
int demo_semaphore_dining_philosophers(void);
```

### Partie 6: IPC Unified Interface

```c
// Interface unifiee pour choisir le meilleur mecanisme
typedef enum {
    IPC_PIPE,
    IPC_FIFO,
    IPC_MSGQUEUE,
    IPC_SHMEM,
    IPC_SOCKET      // Pour reference
} ipc_type_t;

typedef struct {
    ipc_type_t type;
    union {
        pipe_t *pipe;
        fifo_t *fifo;
        msgqueue_t *msgqueue;
        sharedmem_t *shm;
    } handle;
    char name[256];
} ipc_channel_t;

// Selection automatique basee sur les besoins
typedef struct {
    int need_bidirectional;
    int need_persistence;
    int need_unrelated_processes;
    int need_message_boundaries;
    int need_priority;
    size_t expected_message_size;
    int expected_throughput;    // Messages par seconde
} ipc_requirements_t;

ipc_type_t ipc_recommend(const ipc_requirements_t *req);

// API unifiee
ipc_channel_t *ipc_create(ipc_type_t type, const char *name);
void ipc_destroy(ipc_channel_t *ch);
ssize_t ipc_send(ipc_channel_t *ch, const void *buf, size_t len);
ssize_t ipc_receive(ipc_channel_t *ch, void *buf, size_t len);

// Benchmark
typedef struct {
    ipc_type_t type;
    double throughput_mbps;
    double latency_us;
    size_t message_size;
} ipc_benchmark_result_t;

int ipc_benchmark(ipc_type_t type, size_t msg_size, int iterations,
                  ipc_benchmark_result_t *result);
int ipc_benchmark_all(size_t msg_size, int iterations,
                      ipc_benchmark_result_t results[4]);
```

---

## Exemple d'Utilisation

```c
#include "ipc_complete.h"
#include <stdio.h>
#include <string.h>

int main(void) {
    // === Partie 1: Pipes ===
    printf("=== PIPES ===\n");
    pipe_t *p = pipe_create();
    if (!p) {
        perror("pipe_create");
        return 1;
    }

    pid_t pid = fork();
    if (pid == 0) {
        // Enfant: lit du pipe
        pipe_close_write(p);
        char buf[256];
        ssize_t n = pipe_read(p, buf, sizeof(buf));
        if (n > 0) {
            buf[n] = '\0';
            printf("Child received: %s\n", buf);
        }
        pipe_destroy(p);
        exit(0);
    } else {
        // Parent: ecrit dans le pipe
        pipe_close_read(p);
        const char *msg = "Hello from parent!";
        pipe_write(p, msg, strlen(msg));
        pipe_destroy(p);
        wait(NULL);
    }

    // === Partie 2: FIFO ===
    printf("\n=== FIFO ===\n");
    const char *fifo_path = "/tmp/test_fifo";
    fifo_create(fifo_path, 0666);

    pid = fork();
    if (pid == 0) {
        // Enfant: lit du FIFO
        fifo_t *f = fifo_open(fifo_path, O_RDONLY);
        char buf[256];
        ssize_t n = fifo_read(f, buf, sizeof(buf));
        if (n > 0) {
            buf[n] = '\0';
            printf("FIFO received: %s\n", buf);
        }
        fifo_close(f);
        exit(0);
    } else {
        // Parent: ecrit dans le FIFO
        fifo_t *f = fifo_open(fifo_path, O_WRONLY);
        const char *msg = "Hello via FIFO!";
        fifo_write(f, msg, strlen(msg));
        fifo_close(f);
        wait(NULL);
    }
    fifo_remove(fifo_path);

    // === Partie 3: Message Queue ===
    printf("\n=== MESSAGE QUEUE ===\n");
    const char *mq_name = "/test_mq";
    msgqueue_config_t mq_config = {
        .maxmsg = 10,
        .msgsize = 256,
        .flags = 0
    };
    msgqueue_t *mq = msgqueue_create(mq_name, &mq_config);

    pid = fork();
    if (pid == 0) {
        // Enfant: recoit
        msgqueue_t *child_mq = msgqueue_open(mq_name, O_RDONLY);
        char buf[256];
        unsigned int prio;
        ssize_t n = msgqueue_receive(child_mq, buf, sizeof(buf), &prio);
        if (n > 0) {
            buf[n] = '\0';
            printf("MQ received (prio %u): %s\n", prio, buf);
        }
        msgqueue_close(child_mq);
        exit(0);
    } else {
        // Parent: envoie avec priorite
        const char *msg = "Priority message!";
        msgqueue_send(mq, msg, strlen(msg), 5);
        wait(NULL);
    }
    msgqueue_close(mq);
    msgqueue_unlink(mq_name);

    // === Partie 4: Shared Memory ===
    printf("\n=== SHARED MEMORY ===\n");
    const char *shm_name = "/test_shm";
    sharedmem_t *shm = sharedmem_create(shm_name, 4096, PROT_READ | PROT_WRITE);
    int *counter = (int *)sharedmem_get_addr(shm);
    *counter = 0;

    // Semaphore pour synchronisation
    const char *sem_name = "/test_sem";
    semaphore_t *sem = semaphore_create(sem_name, 1);

    for (int i = 0; i < 3; i++) {
        if (fork() == 0) {
            sharedmem_t *child_shm = sharedmem_open(shm_name, 4096, PROT_READ | PROT_WRITE);
            semaphore_t *child_sem = semaphore_open(sem_name);
            int *c = (int *)sharedmem_get_addr(child_shm);

            for (int j = 0; j < 1000; j++) {
                semaphore_wait(child_sem);
                (*c)++;
                semaphore_post(child_sem);
            }

            semaphore_destroy(child_sem);
            sharedmem_close(child_shm);
            exit(0);
        }
    }

    // Attendre tous les enfants
    for (int i = 0; i < 3; i++) wait(NULL);

    printf("Final counter: %d (expected 3000)\n", *counter);

    semaphore_destroy(sem);
    semaphore_unlink(sem_name);
    sharedmem_close(shm);
    sharedmem_unlink(shm_name);

    // === Partie 5: Benchmark ===
    printf("\n=== IPC BENCHMARK ===\n");
    ipc_benchmark_result_t results[4];
    ipc_benchmark_all(1024, 10000, results);

    const char *names[] = {"PIPE", "FIFO", "MSGQUEUE", "SHMEM"};
    for (int i = 0; i < 4; i++) {
        printf("%s: %.2f MB/s, %.2f us latency\n",
               names[i], results[i].throughput_mbps, results[i].latency_us);
    }

    return 0;
}
```

---

## Tests de Validation

```bash
# Compilation (noter -lrt pour les message queues et shm)
gcc -Wall -Wextra -o ipc_complete ipc_complete.c -lrt -lpthread

# Test basique
./ipc_complete

# Tests individuels
./test_pipe
./test_fifo
./test_msgqueue
./test_sharedmem
./test_semaphore

# Benchmark
./ipc_benchmark

# Verifier le nettoyage des ressources IPC
ls /dev/shm/
ls /dev/mqueue/
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| Pipes (anonymes et flags) | 15 |
| FIFOs (creation, open, I/O) | 15 |
| Message Queues (POSIX) | 20 |
| Shared Memory (POSIX) | 20 |
| Semaphores (named et unnamed) | 15 |
| Interface unifiee et benchmark | 10 |
| Documentation | 5 |
| **Total** | **100** |
