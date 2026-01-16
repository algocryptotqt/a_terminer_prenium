# [Module 2.2] - Exercise 07: Message Broker

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex07"
difficulty: moyen
estimated_time: "4-5 heures"
prerequisite_exercises: ["ex06"]
concepts_requis:
  - "Files de messages POSIX (mq_open, mq_send, mq_receive)"
  - "Notifications asynchrones (mq_notify)"
  - "fork() et processus multiples"
  - "Gestion des priorites"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.2.19.a | POSIX Message Queues | mq_open, mq_close, mq_unlink |
| 2.2.19.b | Message priorities | Envoi/reception avec priorites |
| 2.2.19.c | Queue attributes | mq_attr: maxmsg, msgsize |
| 2.2.19.d | Blocking/non-blocking | O_NONBLOCK, timeouts |
| 2.2.19.e | Async notification | mq_notify avec signaux ou threads |
| 2.2.19.f | Multi-producer | Plusieurs processus ecrivains |
| 2.2.19.g | Multi-consumer | Plusieurs processus lecteurs |

### Objectifs Pedagogiques

A la fin de cet exercice, vous devriez etre capable de:
1. Creer et gerer des files de messages POSIX avec attributs personnalises
2. Implementer un systeme publish/subscribe avec priorites
3. Gerer plusieurs producteurs et consommateurs de maniere concurrente
4. Utiliser les notifications asynchrones pour des callbacks non-bloquants
5. Comprendre les compromis entre latence et debit dans les systemes de messagerie

---

## Contexte

### Pourquoi les Message Queues?

Dans les systemes distribues modernes (Kafka, RabbitMQ, ZeroMQ), les **brokers de messages** sont omnipresents. Ils permettent le decouplage entre producteurs et consommateurs de donnees, offrant:

- **Asynchronisme**: Le producteur n'attend pas le consommateur
- **Fiabilite**: Les messages sont persistes meme si le consommateur est temporairement indisponible
- **Scalabilite**: Plusieurs consommateurs peuvent traiter les messages en parallele
- **Priorites**: Les messages urgents passent avant les autres

### Le Probleme a Resoudre

Vous devez implementer un **broker de messages local** utilisant les files POSIX. Ce broker doit:
1. Accepter des messages de **multiples producteurs** (processus differents)
2. Distribuer ces messages a **multiples consommateurs** selon leurs abonnements
3. Respecter les **priorites** (urgent > normal > low)
4. Notifier les consommateurs de maniere **asynchrone** (pas de polling)

### Reflexion Architecturale

Avant de coder, reflechissez:
- Comment structurer les files pour supporter plusieurs "topics"?
- Comment eviter qu'un consommateur lent bloque les autres?
- Que se passe-t-il si un consommateur meurt pendant le traitement?
- Comment garantir qu'un message n'est traite qu'une seule fois?

**Ces questions n'ont pas de reponse unique - votre implementation doit faire des choix clairs et documentes.**

---

## Enonce

### Vue d'Ensemble

Implementez un broker de messages avec support multi-producteurs/multi-consommateurs, systeme de priorites, et notifications asynchrones.

### Architecture du Systeme

```
                    +-------------------+
                    |   Message Broker  |
                    |                   |
  Producers         |   +-----------+   |         Consumers
  ---------         |   | Priority  |   |         ---------
                    |   |   Queue   |   |
  [P1] --publish--> |   | HIGH: ... |   | --notify--> [C1]
                    |   | MED:  ... |   |
  [P2] --publish--> |   | LOW:  ... |   | --notify--> [C2]
                    |   +-----------+   |
  [P3] --publish--> |                   | --notify--> [C3]
                    +-------------------+
```

### Specifications Fonctionnelles

#### Fonctionnalite 1: Creation du Broker

Le broker est un processus independant qui gere les files de messages.

**API**:
```c
/**
 * Cree un nouveau broker de messages.
 *
 * @param name Nom unique du broker (ex: "/my_broker")
 * @param max_msg_size Taille maximale d'un message en octets
 * @param max_messages Nombre maximum de messages en attente par priorite
 * @return Pointeur vers le broker, NULL si erreur
 *
 * @note Le nom doit commencer par '/' et ne pas contenir d'autres '/'
 * @note Les files sont creees avec les permissions 0660
 */
broker_t *broker_create(const char *name, size_t max_msg_size, size_t max_messages);

/**
 * Detruit le broker et libere toutes les ressources.
 *
 * @param broker Pointeur vers le broker
 *
 * @warning Les messages non consommes sont perdus
 */
void broker_destroy(broker_t *broker);
```

**Comportement attendu**:
- Cree 3 files internes (HIGH, MEDIUM, LOW priority)
- Les noms des files derivent du nom du broker (ex: "/my_broker_high")
- Retourne NULL si le broker existe deja ou si erreur systeme

**Cas limites a gerer**:
- Nom invalide (ne commence pas par '/', contient '/' ailleurs)
- Ressources systeme insuffisantes (limite de files atteinte)
- Permissions insuffisantes

#### Fonctionnalite 2: Publication de Messages

**API**:
```c
typedef enum {
    PRIORITY_LOW = 0,
    PRIORITY_MEDIUM = 1,
    PRIORITY_HIGH = 2
} msg_priority_t;

typedef struct {
    uint32_t id;           // ID unique du message (genere par le broker)
    msg_priority_t prio;   // Priorite
    pid_t sender;          // PID du producteur
    struct timespec ts;    // Timestamp de publication
    size_t len;            // Taille du payload
    char payload[];        // Flexible array member
} broker_msg_t;

/**
 * Publie un message sur le broker.
 *
 * @param broker Pointeur vers le broker
 * @param priority Priorite du message
 * @param data Donnees du message
 * @param len Taille des donnees en octets
 * @return ID du message (>0), ou -1 si erreur
 *
 * @note Cette fonction est thread-safe et process-safe
 * @note Si la file est pleine, bloque jusqu'a ce qu'il y ait de la place
 */
int broker_publish(broker_t *broker, msg_priority_t priority,
                   const void *data, size_t len);

/**
 * Version non-bloquante de broker_publish.
 *
 * @return ID du message, -1 si file pleine, -2 si erreur
 */
int broker_try_publish(broker_t *broker, msg_priority_t priority,
                       const void *data, size_t len);
```

**Comportement attendu**:
- Chaque message recoit un ID unique croissant
- Le timestamp est enregistre automatiquement
- Le PID du sender est capture automatiquement
- Les messages sont ajoutes a la file correspondant a leur priorite

#### Fonctionnalite 3: Abonnement et Reception

**API**:
```c
/**
 * Type de callback pour la reception de messages.
 *
 * @param msg Le message recu (propriete du broker, ne pas free)
 * @param user_data Donnees utilisateur passees lors de l'abonnement
 * @return 0 pour ACK (message consomme), -1 pour NACK (reessayer plus tard)
 */
typedef int (*broker_callback_t)(const broker_msg_t *msg, void *user_data);

/**
 * S'abonne au broker pour recevoir des messages.
 *
 * @param broker Pointeur vers le broker
 * @param min_priority Priorite minimale des messages a recevoir
 * @param callback Fonction appelee pour chaque message
 * @param user_data Donnees passees au callback
 * @return 0 si succes, -1 si erreur
 *
 * @note Le callback est appele de maniere asynchrone via mq_notify
 * @note Les messages de priorite >= min_priority sont recus
 */
int broker_subscribe(broker_t *broker, msg_priority_t min_priority,
                     broker_callback_t callback, void *user_data);

/**
 * Se desabonne du broker.
 */
void broker_unsubscribe(broker_t *broker);

/**
 * Reception bloquante d'un message (alternative au callback).
 *
 * @param broker Pointeur vers le broker
 * @param msg_out Buffer pour stocker le message
 * @param timeout_ms Timeout en millisecondes (-1 pour infini)
 * @return Taille du message, 0 si timeout, -1 si erreur
 */
ssize_t broker_receive(broker_t *broker, broker_msg_t *msg_out, int timeout_ms);
```

**Comportement attendu**:
- Les messages de priorite superieure sont traites en premier
- Le callback est invoque dans un thread/signal handler (attention async-signal-safety!)
- Si le callback retourne NACK, le message est remis en queue
- Un consommateur peut choisir de ne recevoir que les messages HIGH priority

#### Fonctionnalite 4: Connexion Client

**API**:
```c
/**
 * Se connecte a un broker existant (cote client).
 *
 * @param name Nom du broker
 * @return Pointeur vers la connexion, NULL si le broker n'existe pas
 */
broker_t *broker_connect(const char *name);

/**
 * Deconnexion propre.
 */
void broker_disconnect(broker_t *broker);
```

---

## Contraintes Techniques

### Standards C

- **Norme**: C17 (ISO/IEC 9899:2018)
- **Compilation**: `gcc -Wall -Wextra -Werror -std=c17 -lrt -pthread`
- **Options additionnelles**: `-lrt` pour les message queues POSIX

### Fonctions Autorisees

```
Message Queues POSIX:
  - mq_open, mq_close, mq_unlink
  - mq_send, mq_receive, mq_timedreceive
  - mq_notify, mq_getattr, mq_setattr

Processus:
  - fork, waitpid, getpid

Signaux:
  - sigaction, sigemptyset, sigaddset
  - sigprocmask, sigwait

Memoire:
  - malloc, free, calloc, realloc
  - memcpy, memset, memmove

Temps:
  - clock_gettime, nanosleep

Divers:
  - open, close, read, write
  - printf, fprintf, snprintf
  - perror, strerror
```

### Contraintes Specifiques

- [ ] Pas de variables globales mutables (constantes OK)
- [ ] Les callbacks doivent etre async-signal-safe OU utiliser un thread dedie
- [ ] Taille maximale d'un message: 8192 octets
- [ ] Maximum 32 messages par file de priorite
- [ ] Le broker doit supporter au moins 10 consommateurs simultanement

### Exigences de Securite

- [ ] Aucune fuite memoire (verification Valgrind)
- [ ] Aucun buffer overflow (verification AddressSanitizer)
- [ ] Verification de tous les retours de mq_* (peuvent echouer!)
- [ ] Nettoyage des files de messages meme en cas de crash (mq_unlink)
- [ ] Pas de deadlock entre producteurs et consommateurs

---

## Format de Rendu

### Fichiers a Rendre

```
ex07/
|-- broker.h           # API publique du broker
|-- broker.c           # Implementation du broker
|-- broker_internal.h  # Structures internes (optionnel)
|-- Makefile           # Compilation
```

### Signatures de Fonctions

#### broker.h

```c
#ifndef BROKER_H
#define BROKER_H

#include <stddef.h>
#include <stdint.h>
#include <sys/types.h>
#include <time.h>

typedef struct broker broker_t;

typedef enum {
    PRIORITY_LOW = 0,
    PRIORITY_MEDIUM = 1,
    PRIORITY_HIGH = 2
} msg_priority_t;

typedef struct {
    uint32_t id;
    msg_priority_t prio;
    pid_t sender;
    struct timespec ts;
    size_t len;
    char payload[];
} broker_msg_t;

typedef int (*broker_callback_t)(const broker_msg_t *msg, void *user_data);

// Creation/destruction
broker_t *broker_create(const char *name, size_t max_msg_size, size_t max_messages);
void broker_destroy(broker_t *broker);

// Connexion client
broker_t *broker_connect(const char *name);
void broker_disconnect(broker_t *broker);

// Publication
int broker_publish(broker_t *broker, msg_priority_t priority,
                   const void *data, size_t len);
int broker_try_publish(broker_t *broker, msg_priority_t priority,
                       const void *data, size_t len);

// Abonnement
int broker_subscribe(broker_t *broker, msg_priority_t min_priority,
                     broker_callback_t callback, void *user_data);
void broker_unsubscribe(broker_t *broker);
ssize_t broker_receive(broker_t *broker, broker_msg_t *msg_out, int timeout_ms);

#endif /* BROKER_H */
```

### Makefile

```makefile
NAME = libbroker.a
TEST = broker_test

CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c17
LDFLAGS = -lrt -pthread

SRCS = broker.c
OBJS = $(SRCS:.c=.o)

all: $(NAME)

$(NAME): $(OBJS)
	ar rcs $(NAME) $(OBJS)

test: $(NAME)
	$(CC) $(CFLAGS) -o $(TEST) test_broker.c -L. -lbroker $(LDFLAGS)
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

### Exemple 1: Broker Simple avec un Producteur/Consommateur

```c
#include "broker.h"
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>

int my_callback(const broker_msg_t *msg, void *user_data) {
    printf("[Consumer] Received msg #%u (prio=%d): %.*s\n",
           msg->id, msg->prio, (int)msg->len, msg->payload);
    return 0; // ACK
}

int main(void) {
    // Parent = Broker owner
    broker_t *broker = broker_create("/test_broker", 256, 10);
    if (!broker) {
        perror("broker_create");
        return 1;
    }

    pid_t pid = fork();
    if (pid == 0) {
        // Child = Producer
        broker_t *client = broker_connect("/test_broker");

        broker_publish(client, PRIORITY_LOW, "Low priority msg", 16);
        broker_publish(client, PRIORITY_HIGH, "URGENT!", 7);
        broker_publish(client, PRIORITY_MEDIUM, "Normal msg", 10);

        broker_disconnect(client);
        _exit(0);
    }

    // Parent = Consumer
    broker_subscribe(broker, PRIORITY_LOW, my_callback, NULL);

    // Attendre les messages pendant 2 secondes
    sleep(2);

    broker_unsubscribe(broker);
    waitpid(pid, NULL, 0);
    broker_destroy(broker);

    return 0;
}
```

**Sortie attendue** (ordre par priorite):
```
[Consumer] Received msg #2 (prio=2): URGENT!
[Consumer] Received msg #3 (prio=1): Normal msg
[Consumer] Received msg #1 (prio=0): Low priority msg
```

### Exemple 2: Multi-Producteurs Concurrents

```c
#include "broker.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

#define NUM_PRODUCERS 5
#define MSGS_PER_PRODUCER 100

int main(void) {
    broker_t *broker = broker_create("/stress_test", 64, 32);

    // Fork NUM_PRODUCERS producteurs
    for (int i = 0; i < NUM_PRODUCERS; i++) {
        if (fork() == 0) {
            broker_t *client = broker_connect("/stress_test");

            for (int j = 0; j < MSGS_PER_PRODUCER; j++) {
                char buf[32];
                snprintf(buf, sizeof(buf), "P%d-M%d", i, j);
                msg_priority_t prio = rand() % 3;
                broker_publish(client, prio, buf, strlen(buf));
            }

            broker_disconnect(client);
            _exit(0);
        }
    }

    // Consommer tous les messages
    int count = 0;
    broker_msg_t *msg = malloc(sizeof(broker_msg_t) + 64);

    while (count < NUM_PRODUCERS * MSGS_PER_PRODUCER) {
        ssize_t ret = broker_receive(broker, msg, 1000);
        if (ret > 0) {
            count++;
        }
    }

    printf("Received %d messages\n", count);

    free(msg);
    for (int i = 0; i < NUM_PRODUCERS; i++)
        wait(NULL);
    broker_destroy(broker);

    return 0;
}
```

**Sortie attendue**:
```
Received 500 messages
```

### Exemple 3: Notification Asynchrone

```c
#include "broker.h"
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

volatile sig_atomic_t running = 1;

void sigint_handler(int sig) {
    (void)sig;
    running = 0;
}

int urgent_handler(const broker_msg_t *msg, void *user_data) {
    (void)user_data;
    printf("[ALERT] Urgent message from PID %d: %.*s\n",
           msg->sender, (int)msg->len, msg->payload);
    return 0;
}

int main(void) {
    signal(SIGINT, sigint_handler);

    broker_t *broker = broker_connect("/my_broker");
    if (!broker) {
        fprintf(stderr, "Broker not found\n");
        return 1;
    }

    // S'abonner uniquement aux messages HIGH priority
    broker_subscribe(broker, PRIORITY_HIGH, urgent_handler, NULL);

    printf("Waiting for urgent messages (Ctrl+C to quit)...\n");

    while (running) {
        pause(); // Attendre les signaux
    }

    broker_unsubscribe(broker);
    broker_disconnect(broker);

    return 0;
}
```

---

## Tests de la Moulinette

### Tests Fonctionnels de Base

#### Test 01: Creation et Destruction
```yaml
description: "Verification du cycle de vie du broker"
input: |
  broker_t *b = broker_create("/test01", 256, 10);
  assert(b != NULL);
  broker_destroy(b);
expected_behavior: "Aucune fuite memoire, files supprimees"
validation: "Valgrind clean + /dev/mqueue vide"
```

#### Test 02: Publication Simple
```yaml
description: "Envoi d'un message"
input: |
  broker_t *b = broker_create("/test02", 256, 10);
  int id = broker_publish(b, PRIORITY_MEDIUM, "Hello", 5);
  assert(id > 0);
  broker_destroy(b);
expected_output: "ID > 0 retourne"
```

#### Test 03: Reception par Priorite
```yaml
description: "Les messages HIGH sont recus avant LOW"
setup: |
  broker_t *b = broker_create("/test03", 256, 10);
  broker_publish(b, PRIORITY_LOW, "L1", 2);
  broker_publish(b, PRIORITY_HIGH, "H1", 2);
  broker_publish(b, PRIORITY_MEDIUM, "M1", 2);
expected_output: |
  Premier message: "H1"
  Deuxieme message: "M1"
  Troisieme message: "L1"
```

#### Test 04: Multi-processus
```yaml
description: "Producteur et consommateur dans des processus differents"
scenario: |
  Parent cree broker, fork, child publie 10 messages
  Parent les recoit tous
expected_behavior: "10 messages recus correctement"
```

#### Test 05: Non-Blocking
```yaml
description: "broker_try_publish retourne -1 si file pleine"
setup: |
  broker_t *b = broker_create("/test05", 64, 2);
  broker_publish(b, PRIORITY_LOW, "A", 1);
  broker_publish(b, PRIORITY_LOW, "B", 1);
input: |
  int ret = broker_try_publish(b, PRIORITY_LOW, "C", 1);
expected_output: "ret == -1"
```

### Tests de Robustesse

#### Test 10: Parametres Invalides
```yaml
description: "Comportement avec entrees invalides"
test_cases:
  - input: "broker_create(NULL, 256, 10)"
    expected: "NULL retourne, errno = EINVAL"
  - input: "broker_create('/invalid/name', 256, 10)"
    expected: "NULL retourne"
  - input: "broker_publish(NULL, PRIORITY_LOW, 'X', 1)"
    expected: "-1 retourne"
  - input: "broker_publish(b, 99, 'X', 1)"
    expected: "-1 retourne, priorite invalide"
```

#### Test 11: Limites
```yaml
description: "Comportement aux limites"
test_cases:
  - input: "Message de taille max_msg_size"
    expected: "OK"
  - input: "Message de taille max_msg_size + 1"
    expected: "-1, message trop grand"
  - input: "Remplir la file puis broker_receive"
    expected: "Tous les messages recuperes"
```

### Tests de Securite

#### Test 20: Fuites Memoire
```yaml
description: "Detection de fuites avec Valgrind"
tool: "valgrind --leak-check=full --show-leak-kinds=all"
scenario: |
  Creer broker
  Publier 1000 messages
  Recevoir 1000 messages
  Detruire broker
expected: "0 bytes lost, 0 errors"
```

#### Test 21: Race Conditions
```yaml
description: "Acces concurrent sans corruption"
tool: "ThreadSanitizer"
scenario: |
  10 producteurs fork, chacun publie 100 messages
  1 consommateur les recoit
expected: "Pas de data race detectee"
```

### Tests de Performance

#### Test 30: Throughput
```yaml
description: "Debit de messages"
scenario: |
  Publier 10000 messages de 64 octets
  Les recevoir tous
data_size: "64 bytes x 10000"
expected_max_time: "< 2 secondes"
machine_ref: "Intel i5 2.5GHz, 8GB RAM"
```

#### Test 31: Latence
```yaml
description: "Latence de bout en bout"
scenario: |
  Producteur envoie message avec timestamp
  Consommateur calcule la difference
expected: "Latence moyenne < 1ms"
```

---

## Criteres d'Evaluation

### Note Minimale Requise: 80/100

### Detail de la Notation (Total: 100 points)

#### 1. Correction (40 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Tests fonctionnels (01-05) | 15 | Tous les tests de base passent |
| Gestion des priorites | 10 | Messages HIGH traites avant LOW |
| Multi-processus | 10 | Producteurs/consommateurs dans fork() |
| Gestion d'erreurs | 5 | Retours coherents, errno correct |

#### 2. Securite (25 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Absence de fuites | 10 | Valgrind clean |
| Protection concurrence | 8 | Pas de race condition |
| Nettoyage ressources | 4 | mq_unlink meme en cas d'erreur |
| Verification retours | 3 | Tous les mq_* verifies |

#### 3. Conception (20 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Architecture | 8 | Separation client/serveur claire |
| Gestion priorites | 7 | Implementation efficace |
| Notification async | 5 | mq_notify bien utilise |

#### 4. Lisibilite (15 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Nommage | 6 | Fonctions et variables explicites |
| Organisation | 4 | Code bien structure |
| Commentaires | 3 | Parties non-evidentes expliquees |
| Style | 2 | Indentation coherente |

---

## Indices et Ressources

### Reflexions pour Demarrer

<details>
<summary>Question 1: Comment gerer les 3 niveaux de priorite?</summary>

Plusieurs approches possibles:
1. **3 files separees**: Une mq pour chaque priorite, le consommateur poll les 3
2. **1 file avec prio POSIX**: mq_send accepte un argument priorite (0-31)
3. **Hybrid**: Combiner les deux approches

L'option 2 est plus simple: POSIX MQ supporte nativement les priorites!
```c
mq_send(mq, msg, len, priority);  // priority = 0-31
mq_receive(mq, buf, len, &prio);  // prio = priorite du message recu
```
Les messages de priorite superieure sont automatiquement recus en premier.

</details>

<details>
<summary>Question 2: Comment implementer mq_notify correctement?</summary>

mq_notify est delicat:
1. Il ne notifie qu'UNE FOIS, puis se desactive
2. Vous devez le re-armer apres chaque notification
3. Le callback peut etre un signal OU un thread

```c
struct sigevent sev;
sev.sigev_notify = SIGEV_THREAD;  // Creer un thread pour le callback
sev.sigev_notify_function = my_thread_func;
sev.sigev_notify_attributes = NULL;
sev.sigev_value.sival_ptr = user_data;

mq_notify(mq, &sev);
```

**Attention**: Dans le callback thread, vous devez re-armer mq_notify!

</details>

<details>
<summary>Question 3: Comment partager le broker entre processus?</summary>

Les message queues POSIX sont **nommees** et existent dans le systeme de fichiers virtuel `/dev/mqueue/`.

- `mq_open("/my_queue", O_CREAT | O_RDWR, 0660, &attr)` cree la file
- `mq_open("/my_queue", O_RDWR)` s'y connecte depuis un autre processus
- `mq_unlink("/my_queue")` la supprime (quand tous les fd sont fermes)

Le broker peut simplement stocker le nom, et chaque client appelle mq_open.

</details>

### Ressources Recommandees

#### Documentation
- **Man pages**: `man mq_overview`, `man mq_open`, `man mq_notify`
- **POSIX.1-2017**: Specification officielle des message queues
- **Module ODYSSEY 2.2.19**: Theorie des files de messages

#### Lectures Complementaires
- "The Linux Programming Interface" par Michael Kerrisk, Chapitre 52
- "UNIX Network Programming Vol. 2" par W. Richard Stevens, Chapitre 5

#### Outils de Debug
- `/dev/mqueue/`: Visualiser les files existantes
- `cat /dev/mqueue/my_queue`: Voir les attributs d'une file
- `ipcs -q` et `ipcrm -q`: Gerer les files System V (pas POSIX mais utile)

### Pieges Frequents

1. **Oublier mq_unlink**: Les files persistent meme apres la fin du programme!
   - **Solution**: Toujours appeler mq_unlink dans broker_destroy et gerer SIGTERM

2. **mq_notify se desactive apres la premiere notification**:
   - **Solution**: Re-armer dans le callback avant de traiter le message

3. **Deadlock avec mq_send bloquant et file pleine**:
   - **Solution**: Utiliser O_NONBLOCK ou des timeouts

4. **Buffer trop petit pour mq_receive**:
   - **Solution**: Allouer au moins mq_attr.mq_msgsize octets

5. **Oublier -lrt a la compilation**:
   - **Solution**: Ajouter -lrt aux flags du linker

---

## Notes du Concepteur

<details>
<summary>Solution de Reference (Concepteur uniquement)</summary>

**Approche recommandee**:

Structure interne:
```c
struct broker {
    char name[NAME_MAX];
    mqd_t mq;           // File unique avec priorites POSIX
    size_t max_msg_size;
    uint32_t next_id;
    bool is_owner;      // true si cree, false si connecte
    // Pour les notifications
    broker_callback_t callback;
    void *user_data;
    pthread_t notify_thread;
    volatile bool running;
};
```

Points cles:
1. Utiliser les priorites natives de POSIX MQ (pas 3 files)
2. Wrapper le message dans broker_msg_t avant envoi
3. Pour mq_notify, utiliser SIGEV_THREAD (plus simple que les signaux)
4. Atomique pour next_id si multi-thread (ou mutex)

**Complexite**:
- Publication: O(log n) - insertion dans file a priorite
- Reception: O(1) - extraction du max

</details>

<details>
<summary>Grille d'Evaluation - Points d'Attention</summary>

**Lors de la correction manuelle, verifier**:
- [ ] Les files sont bien supprimees (ls /dev/mqueue/)
- [ ] Les priorites sont respectees (HIGH avant LOW)
- [ ] Pas de deadlock avec plusieurs producteurs
- [ ] Le callback mq_notify est re-arme correctement

**Erreurs frequentes observees**:
- Utiliser System V MQ au lieu de POSIX MQ
- Ne pas gerer le cas ou mq_open echoue avec EEXIST
- Buffer overflow en ne verifiant pas la taille du message

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
| Originalite | 10/10 | Broker custom, pas copie d'existant |
| Couverture concepts | 10/10 | 2.2.19 entierement couvert |
| Qualite pedagogique | 9/10 | Questions de reflexion incluses |
| Testabilite | 10/10 | Tests precis et automatisables |
| Difficulte appropriee | 10/10 | Moyen, 4-5h realiste |
| Clarté énoncé | 9/10 | API detaillee, exemples complets |
| Cas limites | 9/10 | Edge cases documentes |
| Securite | 10/10 | Exigences memoire/concurrence claires |
| Ressources | 9/10 | Indices et pieges bien documentes |

---

*Template ODYSSEY Phase 2 - Module 2.2 Exercise 07*
