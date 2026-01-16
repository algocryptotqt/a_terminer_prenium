# [Module 2.2] - Exercise 08: Shared Memory Cache

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex08"
difficulty: difficile
estimated_time: "7-8 heures"
prerequisite_exercises: ["ex06", "ex07"]
concepts_requis:
  - "Memoire partagee POSIX (shm_open, mmap)"
  - "Semaphores POSIX (sem_open, sem_wait, sem_post)"
  - "Structures de donnees en memoire contrainte"
  - "Algorithme LRU"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.2.20.a | shm_open/shm_unlink | Creation/destruction de segments |
| 2.2.20.b | mmap for shared memory | Mapping en espace utilisateur |
| 2.2.20.c | Shared data structures | Structures en memoire partagee |
| 2.2.20.d | Memory layout | Organisation des donnees |
| 2.2.21.a | Named semaphores | sem_open, sem_close |
| 2.2.21.b | sem_wait/sem_post | Operations P/V |
| 2.2.21.c | sem_trywait/sem_timedwait | Operations non-bloquantes |
| 2.2.21.d | Reader-writer locks | Synchronisation lecture/ecriture |
| 2.2.21.e | Deadlock prevention | Ordre de verrouillage |

### Objectifs Pedagogiques

A la fin de cet exercice, vous devriez etre capable de:
1. Concevoir des structures de donnees pour la memoire partagee (sans pointeurs absolus!)
2. Implementer une synchronisation lecteurs/ecrivains avec semaphores
3. Comprendre les defis de la coherence de cache entre processus
4. Implementer un algorithme d'eviction LRU en memoire contrainte
5. Gerer la recuperation apres crash d'un processus

---

## Contexte

### Le Probleme des Caches Distribues

Les caches sont partout: Redis, Memcached, CPU caches... Ils accelerent les acces aux donnees en evitant des operations couteuses (requetes reseau, acces disque).

Dans un systeme multi-processus, un **cache en memoire partagee** offre:
- **Zero-copy**: Pas de serialisation/deserialisation
- **Performance**: Acces direct a la memoire
- **Partage**: Tous les processus voient les memes donnees

### Les Defis

Mais la memoire partagee pose des problemes uniques:

1. **Pas de pointeurs absolus**: Chaque processus mappe le segment a une adresse differente
2. **Synchronisation**: Comment permettre plusieurs lecteurs mais un seul ecrivain?
3. **Coherence**: Que faire si un processus crash pendant une ecriture?
4. **Fragmentation**: Comment gerer l'espace de maniere efficace?

### Ce que Vous Allez Construire

Un cache key-value en memoire partagee avec:
- Cles: chaines de caracteres (max 64 octets)
- Valeurs: donnees binaires (max 1024 octets)
- Politique d'eviction: LRU (Least Recently Used)
- Acces concurrent: lecteurs/ecrivains synchronises

### Questions de Conception (Reflechissez AVANT de Coder)

1. Comment representer une table de hachage sans pointeurs?
2. Comment implementer une liste LRU en memoire partagee?
3. Que se passe-t-il si un processus meurt en tenant un semaphore?
4. Comment detecter et reparer un etat corrompu?

---

## Enonce

### Vue d'Ensemble

Implementez un cache key-value en memoire partagee supportant l'acces concurrent depuis plusieurs processus, avec eviction LRU.

### Architecture du Systeme

```
     Shared Memory Segment (/shm_cache_xxx)
+----------------------------------------------------+
|  Header (fixed size)                               |
|  +----------------------------------------------+  |
|  | magic | version | capacity | count | ...     |  |
|  +----------------------------------------------+  |
|                                                    |
|  Hash Table (fixed size array)                     |
|  +----------------------------------------------+  |
|  | bucket[0] | bucket[1] | ... | bucket[N-1]    |  |
|  +----------------------------------------------+  |
|                                                    |
|  Entry Pool (pre-allocated entries)                |
|  +----------------------------------------------+  |
|  | entry[0] | entry[1] | ... | entry[M-1]       |  |
|  +----------------------------------------------+  |
|                                                    |
|  Data Pool (pre-allocated data blocks)             |
|  +----------------------------------------------+  |
|  | data block 0 | data block 1 | ...            |  |
|  +----------------------------------------------+  |
+----------------------------------------------------+

     Named Semaphores
+-------------------+
| /cache_xxx_mutex  |  <- Acces exclusif ecriture
| /cache_xxx_readers|  <- Compteur lecteurs
+-------------------+
```

### Specifications Fonctionnelles

#### Fonctionnalite 1: Creation/Ouverture du Cache

**API**:
```c
/**
 * Structure opaque representant le cache.
 */
typedef struct cache cache_t;

/**
 * Cree un nouveau cache en memoire partagee.
 *
 * @param name Nom unique du cache (ex: "my_cache")
 * @param capacity Nombre maximum d'entrees
 * @param max_key_size Taille max d'une cle (<=64)
 * @param max_value_size Taille max d'une valeur (<=1024)
 * @return Pointeur vers le cache, NULL si erreur
 *
 * @note Le segment est cree avec les permissions 0660
 * @note Le nom est utilise pour creer /dev/shm/cache_<name>
 */
cache_t *cache_create(const char *name, size_t capacity,
                      size_t max_key_size, size_t max_value_size);

/**
 * Se connecte a un cache existant.
 *
 * @param name Nom du cache
 * @return Pointeur vers le cache, NULL si n'existe pas
 */
cache_t *cache_open(const char *name);

/**
 * Ferme la connexion au cache (le cache persiste).
 */
void cache_close(cache_t *cache);

/**
 * Detruit le cache et libere les ressources.
 *
 * @warning Tous les processus doivent avoir ferme leur connexion
 */
void cache_destroy(cache_t *cache);
```

**Comportement attendu**:
- Le segment de memoire partagee est initialise avec les structures internes
- Les semaphores sont crees et initialises
- Le magic number permet de detecter une corruption

**Cas limites a gerer**:
- Cache avec ce nom existe deja -> retourner NULL
- Capacite 0 ou trop grande -> erreur
- Permissions insuffisantes

#### Fonctionnalite 2: Operations CRUD

**API**:
```c
/**
 * Ajoute ou met a jour une entree dans le cache.
 *
 * @param cache Pointeur vers le cache
 * @param key Cle (string, null-terminated)
 * @param value Donnees a stocker
 * @param value_len Taille des donnees en octets
 * @return 0 si succes, -1 si erreur
 *
 * @note Si le cache est plein, l'entree LRU est evincee
 * @note Si la cle existe, la valeur est mise a jour
 * @note Thread-safe et process-safe
 */
int cache_put(cache_t *cache, const char *key,
              const void *value, size_t value_len);

/**
 * Recupere une valeur du cache.
 *
 * @param cache Pointeur vers le cache
 * @param key Cle a chercher
 * @param value_out Buffer pour stocker la valeur (peut etre NULL)
 * @param value_len_out Taille de la valeur retournee
 * @return 0 si trouve, -1 si non trouve, -2 si erreur
 *
 * @note Met a jour le timestamp LRU de l'entree
 * @note Si value_out est NULL, verifie juste l'existence
 */
int cache_get(cache_t *cache, const char *key,
              void *value_out, size_t *value_len_out);

/**
 * Supprime une entree du cache.
 *
 * @return 0 si supprime, -1 si non trouve
 */
int cache_delete(cache_t *cache, const char *key);

/**
 * Vide completement le cache.
 */
void cache_clear(cache_t *cache);
```

**Comportement attendu**:
- Les lecteurs peuvent acceder simultanement (pas de blocage entre eux)
- Un ecrivain bloque tous les autres acces
- L'eviction LRU se fait automatiquement quand le cache est plein

#### Fonctionnalite 3: Statistiques et Monitoring

**API**:
```c
typedef struct {
    size_t capacity;      // Capacite maximale
    size_t count;         // Nombre d'entrees actuelles
    uint64_t hits;        // Nombre de cache hits
    uint64_t misses;      // Nombre de cache misses
    uint64_t evictions;   // Nombre d'evictions LRU
    uint64_t puts;        // Nombre de put
    uint64_t deletes;     // Nombre de delete
} cache_stats_t;

/**
 * Recupere les statistiques du cache.
 *
 * @note Les stats sont atomiquement consistantes
 */
cache_stats_t cache_get_stats(cache_t *cache);

/**
 * Itere sur toutes les entrees (debug).
 *
 * @param callback Fonction appelee pour chaque entree
 * @param user_data Donnees utilisateur
 * @return Nombre d'entrees iterees
 */
typedef void (*cache_iter_fn)(const char *key, const void *value,
                              size_t value_len, void *user_data);
size_t cache_iterate(cache_t *cache, cache_iter_fn callback, void *user_data);
```

### Specifications Techniques

#### Layout Memoire

Le segment de memoire partagee doit avoir un layout fixe calculable:

```c
// Toutes les tailles doivent etre alignees sur 8 octets

struct cache_header {
    uint32_t magic;           // 0xCAC4E000
    uint32_t version;         // 1
    size_t capacity;          // Nombre max d'entrees
    size_t count;             // Nombre actuel
    size_t max_key_size;
    size_t max_value_size;
    uint64_t stats_hits;
    uint64_t stats_misses;
    // ... autres stats
    int32_t lru_head;         // Index de l'entree la plus recente (-1 si vide)
    int32_t lru_tail;         // Index de l'entree la plus ancienne
    int32_t free_head;        // Index de la premiere entree libre
};

struct cache_bucket {
    int32_t head;             // Index de la premiere entree du bucket (-1 si vide)
};

struct cache_entry {
    int32_t next_in_bucket;   // Chainee dans le bucket
    int32_t prev_lru;         // Liste doublement chainee LRU
    int32_t next_lru;
    int32_t data_block;       // Index du bloc de donnees
    size_t value_len;
    uint64_t access_time;     // Pour LRU
    char key[MAX_KEY_SIZE];   // Cle inline
};

// Layout:
// [header][bucket[0]..bucket[N]][entry[0]..entry[M]][data[0]..data[M]]
```

**Point Crucial**: Tous les "pointeurs" sont des **indices** (int32_t), pas des adresses!

#### Algorithme LRU

```
PUT(key, value):
  1. Si key existe:
     - Mettre a jour value
     - Deplacer en tete de LRU
  2. Sinon:
     - Si cache plein: evincer tail de LRU
     - Allouer nouvelle entree de free list
     - Inserer dans hash bucket
     - Placer en tete de LRU

GET(key):
  1. Chercher dans hash bucket
  2. Si trouve:
     - Deplacer en tete de LRU
     - Retourner value
  3. Sinon: cache miss

DELETE(key):
  1. Retirer du hash bucket
  2. Retirer de la liste LRU
  3. Ajouter a la free list
```

#### Synchronisation Lecteurs/Ecrivains

Utilisez le pattern classique avec deux semaphores:

```
mutex: Semaphore binaire (initialement 1)
readers: Compteur de lecteurs dans le header

START_READ:
  sem_wait(mutex)
  header->readers++
  if (readers == 1):
    // Premier lecteur bloque les ecrivains
    sem_wait(write_lock)
  sem_post(mutex)

END_READ:
  sem_wait(mutex)
  header->readers--
  if (readers == 0):
    // Dernier lecteur libere les ecrivains
    sem_post(write_lock)
  sem_post(mutex)

START_WRITE:
  sem_wait(write_lock)

END_WRITE:
  sem_post(write_lock)
```

---

## Contraintes Techniques

### Standards C

- **Norme**: C17 (ISO/IEC 9899:2018)
- **Compilation**: `gcc -Wall -Wextra -Werror -std=c17 -lrt -pthread`

### Fonctions Autorisees

```
Memoire partagee:
  - shm_open, shm_unlink
  - mmap, munmap
  - ftruncate

Semaphores:
  - sem_open, sem_close, sem_unlink
  - sem_wait, sem_post, sem_trywait, sem_timedwait
  - sem_getvalue

Memoire:
  - malloc, free, calloc (pour structures locales uniquement!)
  - memcpy, memset, memmove, memcmp
  - strlen, strncpy, strcmp, strncmp

Temps:
  - clock_gettime

Divers:
  - open, close, fstat
  - printf, fprintf, snprintf (debug uniquement)
```

### Contraintes Specifiques

- [ ] **Pas de pointeurs absolus dans la memoire partagee** (utiliser des offsets/indices)
- [ ] Taille maximale du segment: 16 MB
- [ ] Cles: max 64 octets
- [ ] Valeurs: max 1024 octets
- [ ] Capacite: 1 a 10000 entrees
- [ ] Les structures doivent etre alignees sur 8 octets
- [ ] Pas de variables globales mutables

### Exigences de Securite

- [ ] Aucune fuite memoire (munmap, sem_close)
- [ ] Protection contre les acces concurrents corrompants
- [ ] Verification du magic number a l'ouverture
- [ ] Gestion du cas ou un processus meurt en tenant un lock (robust mutexes bonus)
- [ ] Nettoyage des ressources systeme meme en cas de crash

---

## Format de Rendu

### Fichiers a Rendre

```
ex08/
|-- cache.h            # API publique
|-- cache.c            # Implementation
|-- cache_internal.h   # Structures internes
|-- Makefile
```

### Signatures de Fonctions

#### cache.h

```c
#ifndef CACHE_H
#define CACHE_H

#include <stddef.h>
#include <stdint.h>

typedef struct cache cache_t;

typedef struct {
    size_t capacity;
    size_t count;
    uint64_t hits;
    uint64_t misses;
    uint64_t evictions;
    uint64_t puts;
    uint64_t deletes;
} cache_stats_t;

typedef void (*cache_iter_fn)(const char *key, const void *value,
                              size_t value_len, void *user_data);

// Lifecycle
cache_t *cache_create(const char *name, size_t capacity,
                      size_t max_key_size, size_t max_value_size);
cache_t *cache_open(const char *name);
void cache_close(cache_t *cache);
void cache_destroy(cache_t *cache);

// CRUD
int cache_put(cache_t *cache, const char *key,
              const void *value, size_t value_len);
int cache_get(cache_t *cache, const char *key,
              void *value_out, size_t *value_len_out);
int cache_delete(cache_t *cache, const char *key);
void cache_clear(cache_t *cache);

// Stats
cache_stats_t cache_get_stats(cache_t *cache);
size_t cache_iterate(cache_t *cache, cache_iter_fn callback, void *user_data);

#endif /* CACHE_H */
```

### Makefile

```makefile
NAME = libcache.a
TEST = cache_test

CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c17
LDFLAGS = -lrt -pthread

SRCS = cache.c
OBJS = $(SRCS:.c=.o)

all: $(NAME)

$(NAME): $(OBJS)
	ar rcs $(NAME) $(OBJS)

test: $(NAME)
	$(CC) $(CFLAGS) -o $(TEST) test_cache.c -L. -lcache $(LDFLAGS)
	./$(TEST)

clean:
	rm -f $(OBJS)

fclean: clean
	rm -f $(NAME) $(TEST)
	# Nettoyer les ressources partagees orphelines
	rm -f /dev/shm/cache_* 2>/dev/null || true

re: fclean all

.PHONY: all clean fclean re test
```

---

## Exemples d'Utilisation

### Exemple 1: Cache Simple

```c
#include "cache.h"
#include <stdio.h>
#include <string.h>

int main(void) {
    // Creer un cache de 100 entrees
    cache_t *cache = cache_create("demo", 100, 64, 256);
    if (!cache) {
        perror("cache_create");
        return 1;
    }

    // Stocker des valeurs
    cache_put(cache, "user:1001", "Alice", 6);
    cache_put(cache, "user:1002", "Bob", 4);
    cache_put(cache, "config:theme", "dark", 5);

    // Recuperer une valeur
    char buffer[256];
    size_t len;
    if (cache_get(cache, "user:1001", buffer, &len) == 0) {
        printf("user:1001 = %.*s\n", (int)len, buffer);
    }

    // Stats
    cache_stats_t stats = cache_get_stats(cache);
    printf("Hits: %lu, Misses: %lu\n", stats.hits, stats.misses);

    cache_destroy(cache);
    return 0;
}
```

**Sortie**:
```
user:1001 = Alice
Hits: 1, Misses: 0
```

### Exemple 2: Multi-Processus

```c
#include "cache.h"
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
#include <string.h>

int main(void) {
    cache_t *cache = cache_create("shared", 50, 32, 128);

    pid_t pid = fork();
    if (pid == 0) {
        // Enfant: ecrit des valeurs
        cache_t *child_cache = cache_open("shared");

        for (int i = 0; i < 20; i++) {
            char key[32], value[64];
            snprintf(key, sizeof(key), "key:%d", i);
            snprintf(value, sizeof(value), "value_from_child_%d", i);
            cache_put(child_cache, key, value, strlen(value) + 1);
            usleep(10000); // 10ms
        }

        cache_close(child_cache);
        _exit(0);
    }

    // Parent: lit des valeurs
    usleep(50000); // Laisser l'enfant commencer

    char buffer[128];
    size_t len;
    for (int i = 0; i < 10; i++) {
        char key[32];
        snprintf(key, sizeof(key), "key:%d", i);
        if (cache_get(cache, key, buffer, &len) == 0) {
            printf("Parent got: %s = %s\n", key, buffer);
        }
    }

    waitpid(pid, NULL, 0);

    cache_stats_t stats = cache_get_stats(cache);
    printf("Final count: %zu, Evictions: %lu\n",
           stats.count, stats.evictions);

    cache_destroy(cache);
    return 0;
}
```

### Exemple 3: Test LRU Eviction

```c
#include "cache.h"
#include <stdio.h>
#include <string.h>

int main(void) {
    // Cache de seulement 5 entrees pour tester LRU
    cache_t *cache = cache_create("lru_test", 5, 16, 32);

    // Remplir le cache
    for (int i = 1; i <= 5; i++) {
        char key[16], value[32];
        snprintf(key, sizeof(key), "k%d", i);
        snprintf(value, sizeof(value), "v%d", i);
        cache_put(cache, key, value, strlen(value) + 1);
        printf("Put %s\n", key);
    }
    // Ordre LRU (plus recent en premier): k5, k4, k3, k2, k1

    // Acceder a k2 (le met en tete)
    char buf[32];
    size_t len;
    cache_get(cache, "k2", buf, &len);
    printf("Accessed k2 -> now MRU\n");
    // Ordre LRU: k2, k5, k4, k3, k1

    // Ajouter k6 -> doit evincer k1 (le plus ancien)
    cache_put(cache, "k6", "v6", 3);
    printf("Put k6 -> k1 should be evicted\n");

    // Verifier que k1 est evince
    if (cache_get(cache, "k1", buf, &len) == -1) {
        printf("k1 correctly evicted!\n");
    }

    // k2 devrait toujours etre la
    if (cache_get(cache, "k2", buf, &len) == 0) {
        printf("k2 still present: %s\n", buf);
    }

    cache_stats_t stats = cache_get_stats(cache);
    printf("Evictions: %lu\n", stats.evictions);

    cache_destroy(cache);
    return 0;
}
```

**Sortie**:
```
Put k1
Put k2
Put k3
Put k4
Put k5
Accessed k2 -> now MRU
Put k6 -> k1 should be evicted
k1 correctly evicted!
k2 still present: v2
Evictions: 1
```

---

## Tests de la Moulinette

### Tests Fonctionnels de Base

#### Test 01: Creation et Destruction
```yaml
description: "Cycle de vie basique du cache"
input: |
  cache_t *c = cache_create("test01", 100, 64, 256);
  assert(c != NULL);
  cache_destroy(c);
validation: |
  - /dev/shm/cache_test01 n'existe plus
  - Semaphores supprimes
  - Valgrind clean
```

#### Test 02: Put/Get Simple
```yaml
description: "Operations CRUD basiques"
scenario: |
  cache_put(cache, "hello", "world", 5);
  cache_get(cache, "hello", buf, &len);
expected: |
  buf = "world"
  len = 5
```

#### Test 03: Update Existant
```yaml
description: "Mise a jour d'une cle existante"
scenario: |
  cache_put(cache, "key", "value1", 6);
  cache_put(cache, "key", "updated", 7);
  cache_get(cache, "key", buf, &len);
expected: |
  buf = "updated"
  len = 7
  stats.count = 1 (pas de doublon)
```

#### Test 04: Delete
```yaml
description: "Suppression d'entree"
scenario: |
  cache_put(cache, "temp", "data", 4);
  cache_delete(cache, "temp");
  ret = cache_get(cache, "temp", buf, &len);
expected: |
  ret = -1 (not found)
```

#### Test 05: LRU Eviction
```yaml
description: "Eviction LRU quand cache plein"
setup: |
  cache = cache_create("lru", 3, 16, 16);
  cache_put(cache, "a", "1", 1);
  cache_put(cache, "b", "2", 1);
  cache_put(cache, "c", "3", 1);
  cache_get(cache, "a", buf, &len);  // a devient MRU
  cache_put(cache, "d", "4", 1);     // b devrait etre evince
expected: |
  cache_get("a") = success
  cache_get("b") = -1 (evicted)
  cache_get("c") = success
  cache_get("d") = success
```

### Tests de Concurrence

#### Test 10: Multi-Reader
```yaml
description: "Plusieurs lecteurs simultanement"
scenario: |
  Fork 10 processus
  Chacun fait 100 cache_get sur les memes cles
expected: |
  Pas de blocage
  Toutes les lectures reussissent
  Pas de corruption
```

#### Test 11: Reader-Writer Mix
```yaml
description: "Lecteurs et ecrivains concurrents"
scenario: |
  5 processus ecrivains (chacun 50 puts)
  5 processus lecteurs (chacun 100 gets)
expected: |
  Pas de deadlock
  Pas de donnees corrompues
  Stats coherentes
```

#### Test 12: Write Contention
```yaml
description: "Contention en ecriture"
scenario: |
  10 processus font cache_put sur la meme cle
expected: |
  Derniere valeur gagne
  Pas de crash
```

### Tests de Robustesse

#### Test 20: Parametres Invalides
```yaml
test_cases:
  - input: "cache_create(NULL, 100, 64, 256)"
    expected: "NULL"
  - input: "cache_create('test', 0, 64, 256)"
    expected: "NULL"
  - input: "cache_put(cache, NULL, 'data', 4)"
    expected: "-1"
  - input: "cache_get(NULL, 'key', buf, &len)"
    expected: "-1"
```

#### Test 21: Limites de Taille
```yaml
test_cases:
  - input: "cle de 65 octets (depasse max_key_size=64)"
    expected: "-1"
  - input: "valeur de 1025 octets (depasse max_value_size=1024)"
    expected: "-1"
  - input: "cle vide ''"
    expected: "-1"
```

### Tests de Securite

#### Test 30: Fuites Memoire
```yaml
description: "Detection de fuites"
tool: "valgrind --leak-check=full --track-fds=yes"
scenario: |
  Creer cache
  1000 put
  500 delete
  500 get
  Detruire cache
expected: |
  0 bytes lost
  Tous les fd fermes
  Segments shm supprimes
```

#### Test 31: Magic Number
```yaml
description: "Detection de corruption"
scenario: |
  Creer cache, le corrompre manuellement
  Essayer cache_open
expected: |
  NULL retourne si magic invalide
```

### Tests de Performance

#### Test 40: Throughput
```yaml
description: "Debit d'operations"
scenario: |
  100000 put de cles aleatoires
  100000 get de cles aleatoires
expected_max_time: "< 5 secondes"
machine_ref: "Intel i5 2.5GHz"
```

#### Test 41: Contention Performance
```yaml
description: "Performance sous contention"
scenario: |
  10 processus, chacun 10000 operations mixtes
expected: |
  Pas de degradation lineaire
  Throughput > 50% du single-process
```

---

## Criteres d'Evaluation

### Note Minimale Requise: 80/100

### Detail de la Notation (Total: 100 points)

#### 1. Correction (40 points)

| Critere | Points | Description |
|---------|--------|-------------|
| CRUD fonctionnel | 15 | put, get, delete marchent |
| LRU correct | 10 | Eviction de la bonne entree |
| Multi-processus | 10 | Partage correct entre fork |
| Stats correctes | 5 | hits, misses, evictions precis |

#### 2. Securite (25 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Pas de fuites | 8 | Valgrind clean |
| Synchronisation | 8 | Pas de race condition |
| Nettoyage ressources | 5 | shm_unlink, sem_unlink |
| Robustesse | 4 | Magic check, validation params |

#### 3. Conception (20 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Layout memoire | 8 | Structure claire, alignement correct |
| Pas de pointeurs absolus | 6 | Utilisation d'indices |
| Algo LRU | 4 | Implementation efficace O(1) |
| Hash table | 2 | Distribution correcte |

#### 4. Lisibilite (15 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Nommage | 5 | Variables explicites |
| Organisation | 4 | Separation header/implementation |
| Commentaires | 3 | Layout documente |
| Style | 3 | Code propre |

---

## Indices et Ressources

### Reflexions pour Demarrer

<details>
<summary>Question 1: Comment eviter les pointeurs en memoire partagee?</summary>

Utilisez des **offsets** ou des **indices** au lieu de pointeurs:

```c
// MAUVAIS: pointeur absolu
struct entry {
    struct entry *next;  // Adresse differente dans chaque processus!
};

// BON: indice dans un tableau
struct entry {
    int32_t next;  // Index dans le tableau entries[]
};

// Pour acceder:
struct entry *get_entry(cache_t *cache, int32_t idx) {
    if (idx < 0) return NULL;
    return &cache->entries[idx];
}
```

Alternative avec offsets:
```c
size_t next_offset;  // Offset depuis le debut du segment
// Acces: (char*)base + offset
```

</details>

<details>
<summary>Question 2: Comment calculer la taille du segment?</summary>

```c
size_t calculate_segment_size(size_t capacity,
                              size_t max_key_size,
                              size_t max_value_size) {
    size_t header_size = sizeof(struct cache_header);
    size_t buckets_size = HASH_SIZE * sizeof(struct cache_bucket);
    size_t entries_size = capacity * sizeof(struct cache_entry);
    size_t data_size = capacity * max_value_size;

    // Alignement sur page (optionnel mais recommande)
    size_t total = header_size + buckets_size + entries_size + data_size;
    return (total + 4095) & ~4095;  // Arrondi a la page superieure
}
```

</details>

<details>
<summary>Question 3: Comment implementer readers-writers avec semaphores?</summary>

```c
// Dans le header du cache (en shared memory)
struct cache_header {
    // ...
    int32_t readers_count;  // Nombre de lecteurs actifs
};

// Semaphores nommes (en dehors du shm)
sem_t *mutex;       // Protege readers_count
sem_t *write_lock;  // Bloque les ecrivains

void read_lock(cache_t *c) {
    sem_wait(c->mutex);
    c->header->readers_count++;
    if (c->header->readers_count == 1)
        sem_wait(c->write_lock);  // Premier lecteur bloque les ecrivains
    sem_post(c->mutex);
}

void read_unlock(cache_t *c) {
    sem_wait(c->mutex);
    c->header->readers_count--;
    if (c->header->readers_count == 0)
        sem_post(c->write_lock);  // Dernier lecteur libere
    sem_post(c->mutex);
}

void write_lock(cache_t *c) {
    sem_wait(c->write_lock);
}

void write_unlock(cache_t *c) {
    sem_post(c->write_lock);
}
```

**Attention**: Ce pattern peut causer la famine des ecrivains. Pour un systeme de production, considerer un verrou plus sophistique.

</details>

### Ressources Recommandees

#### Documentation
- **Man pages**: `man shm_overview`, `man sem_overview`
- **POSIX.1-2017**: Specifications shared memory et semaphores
- **Module ODYSSEY 2.2.20-21**: Theorie detaillee

#### Lectures Complementaires
- "The Linux Programming Interface" par Michael Kerrisk, Chapitres 54 (SHM) et 53 (SEM)
- "Advanced Programming in the UNIX Environment" par Stevens, Chapitre 15

#### Outils de Debug
- `ls -la /dev/shm/`: Voir les segments existants
- `ipcs -m`: Statistiques memoire partagee System V
- `lsof | grep shm`: Processus utilisant la shm
- `sem_getvalue()`: Debugger les semaphores

### Pieges Frequents

1. **Utiliser des pointeurs dans la shared memory**:
   - Chaque processus mappe le segment a une adresse differente!
   - **Solution**: Utiliser des indices ou des offsets

2. **Oublier l'alignement**:
   - Les structures doivent etre alignees pour eviter les SIGBUS sur certaines architectures
   - **Solution**: Utiliser `_Alignas(8)` ou padding manuel

3. **Semaphore non libere en cas d'erreur**:
   - Si un processus crash en tenant un semaphore, deadlock!
   - **Solution**: Utiliser `sem_timedwait` avec timeout, ou `pthread_mutex` avec `PTHREAD_MUTEX_ROBUST`

4. **Ne pas verifier le magic number**:
   - Un fichier /dev/shm/ corrompu peut crasher le programme
   - **Solution**: Toujours verifier le magic a l'ouverture

5. **Fuite de ressources systeme**:
   - Les segments shm et semaphores persistent!
   - **Solution**: Toujours shm_unlink/sem_unlink dans destroy, meme en cas d'erreur

---

## Notes du Concepteur

<details>
<summary>Solution de Reference (Concepteur uniquement)</summary>

**Structure recommandee**:

```c
struct cache {
    char name[64];
    void *mapped;         // mmap result
    size_t mapped_size;
    struct cache_header *header;  // = mapped
    struct cache_bucket *buckets; // = mapped + sizeof(header)
    struct cache_entry *entries;  // = buckets + hash_size
    char *data_pool;              // = entries + capacity
    sem_t *mutex;
    sem_t *write_lock;
    bool is_owner;
};
```

**Hash function recommandee**:
```c
uint32_t hash_key(const char *key) {
    uint32_t hash = 5381;
    while (*key)
        hash = ((hash << 5) + hash) + *key++;
    return hash;
}
```

**LRU en O(1)**:
- Liste doublement chainee avec prev/next (indices)
- head = MRU (Most Recently Used)
- tail = LRU (Least Recently Used)
- A chaque acces: retirer de la liste, inserer en head

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
| Originalite | 10/10 | Cache custom, pas Redis/Memcached |
| Couverture concepts | 10/10 | 2.2.20 + 2.2.21 complets |
| Qualite pedagogique | 9/10 | Layout memoire detaille |
| Testabilite | 10/10 | Tests multi-processus clairs |
| Difficulte appropriee | 9/10 | Difficile mais faisable en 7h |
| Clarte enonce | 9/10 | Architecture bien expliquee |
| Cas limites | 10/10 | Corruption, crash, limites |
| Securite | 10/10 | Synchronisation exigee |
| Ressources | 9/10 | Indices sur les pieges majeurs |

---

*Template ODYSSEY Phase 2 - Module 2.2 Exercise 08*
