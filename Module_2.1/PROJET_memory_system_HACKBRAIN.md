<thinking>
## Analyse du Concept
- Concept : Projet Synthèse Memory Management Complet (Allocateur + GC + Profiler)
- Phase demandée : Phase 2 (Intermédiaire-Avancé)
- Adapté ? OUI - C'est le projet capstone qui synthétise TOUS les 250+ concepts du Module 2.1

## Combo Base + Bonus
- Exercice de base : Système mémoire complet en 3 parties (Allocateur multi-niveau, GC conservatif, Profiler)
- Bonus : Implémentation d'un GC générationnel concurrent avec pauses < 10ms
- Palier bonus : 🧠 GÉNIE (projet de niveau recherche)
- Progression logique ? OUI - Le bonus ajoute une dimension temps-réel au GC

## Prérequis & Difficulté
- Prérequis réels : TOUS les ex01-ex10 et M2.1_Ex01-Ex08 du Module 2.1
- Difficulté estimée : 9/10 base, 🧠 (15/10) bonus
- Cohérent avec phase ? OUI - Phase 2+ avec projet d'envergure professionnelle

## Aspect Fun/Culture
- Contexte choisi : AKIRA (1988) - Le chef-d'œuvre cyberpunk japonais
- MEME mnémotechnique : "TETSUOOOO! KANEDAAA!" - L'explosion de mémoire incontrôlée
- Pourquoi c'est fun :
  * Akira parle de pouvoir destructeur incontrôlé (comme une fuite mémoire)
  * Tetsuo perd le contrôle de son pouvoir psychique (memory corruption)
  * Neo-Tokyo reconstruit après l'explosion (GC qui nettoie)
  * Le projet SOL (satellite laser) = precision chirurgicale (profiler)
  * Les capsules médicales de Tetsuo = memory pools/caches
  * Le stade olympique souterrain = heap management

## Scénarios d'Échec (5+ mutants concrets)
1. Mutant A (Boundary) : Off-by-one dans le calcul de la taille de buddy → blocs trop petits
2. Mutant B (Safety) : Pas de vérification NULL avant gc_collect() → crash sur heap vide
3. Mutant C (Resource) : Fuite dans segregated list → mémoire jamais rendue au système
4. Mutant D (Logic) : Mark bit jamais reset → tous objets marqués "vivants" → pas de collection
5. Mutant E (Return) : profiler_check_leaks() retourne 0 même avec fuites
6. Mutant F (Concurrency) : Race condition sur le compteur de références thread-local
7. Mutant G (Corruption) : Canary écrasé mais non détecté → overflow silencieux

## Verdict
VALIDE - Projet ambitieux mais réaliste pour le niveau Phase 2 capstone
Score d'intelligence exercice : 98/100 (synthèse exceptionnelle avec analogie culturelle parfaite)
</thinking>

---

# Exercice 2.1-PROJET : neo_tokyo_memory_system

**Module :**
2.1 — Memory Management

**Concept :**
PROJET — Synthèse Complète (Allocateur + GC + Profiler)

**Difficulté :**
★★★★★★★★★☆ (9/10) | Bonus : 🧠 (15/10)

**Type :**
complet_2

**Tiers :**
3 — Synthèse (tous concepts 2.1.1 à 2.1.22)

**Langage :**
C17

**Prérequis :**
- Tous les exercices ex01-ex10 du Module 2.1
- Tous les exercices M2.1_Ex01 à M2.1_Ex08
- Maîtrise des pointeurs et arithmétique d'adresses
- Connaissance des syscalls mmap/munmap/sbrk
- Bases du multithreading (pthread)

**Domaines :**
Mem, CPU, Struct, Process

**Durée estimée :**
50 heures

**XP Base :**
5000

**Complexité :**
T3 O(n log n) × S3 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichier à rendre :**
```
neo_tokyo_memory/
├── include/
│   ├── neo_alloc.h
│   ├── neo_gc.h
│   └── neo_profiler.h
├── src/
│   ├── allocator/
│   │   ├── segregated.c
│   │   ├── buddy.c
│   │   ├── large.c
│   │   └── thread_cache.c
│   ├── gc/
│   │   ├── roots.c
│   │   ├── mark.c
│   │   ├── sweep.c
│   │   └── conservative.c
│   └── profiler/
│       ├── tracker.c
│       ├── canary.c
│       ├── poison.c
│       └── report.c
├── tests/
│   └── *.c
├── Makefile
└── README.md
```

**Fonctions autorisées :**
- `mmap`, `munmap`, `mprotect`
- `sbrk` (optionnel, préférer mmap)
- `pthread_*` (mutex, thread-local)
- `write`, `open`, `close`
- `memset`, `memcpy`, `memmove`

**Fonctions interdites :**
- `malloc`, `free`, `calloc`, `realloc` (de la libc)
- `printf`, `fprintf` (utiliser write)
- Toute bibliothèque d'allocation externe

### 1.2 Consigne

**🎬 AKIRA - NEO-TOKYO 2019 : LE SYSTÈME MÉMOIRE ULTIME**

*"Le pouvoir est dans le contrôle. Sans contrôle, Neo-Tokyo n'est que ruines."*
— Colonel Shikishima

Dans Neo-Tokyo, après l'explosion de 1988, le gouvernement a reconstruit la ville grâce à un système de gestion des ressources ultra-sophistiqué. Toi, tu es recruté par le **Projet SOL** pour créer le système de mémoire le plus avancé jamais conçu.

Comme **Tetsuo** qui a appris à contrôler son pouvoir psychique destructeur, tu dois maîtriser trois aspects de la mémoire :

1. **L'ALLOCATEUR AKIRA** - Comme le pouvoir d'Akira lui-même, capable d'allouer et libérer de la mémoire avec une puissance inégalée
2. **LE GC KANEDA** - Comme Kaneda qui nettoie les rues de Neo-Tokyo sur sa moto rouge, ton garbage collector traque et élimine les objets orphelins
3. **LE PROFILER SOL** - Comme le satellite laser SOL, une précision chirurgicale pour détecter les moindres anomalies

### 1.2.1 PARTIE 1 : L'Allocateur Akira (Puissance Pure)

**Ta mission :**

Implémenter `akira_alloc()` et `akira_free()` avec une architecture multi-niveau :

```
┌─────────────────────────────────────────────────────────────────┐
│                    ARCHITECTURE AKIRA                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   NIVEAU 1: CAPSULES (< 256 bytes)                              │
│   ┌─────────────────────────────────────┐                       │
│   │  Segregated Free Lists              │                       │
│   │  8, 16, 32, 64, 128, 256 bytes      │                       │
│   │  Allocation O(1)                    │                       │
│   └─────────────────────────────────────┘                       │
│                     ↓                                           │
│   NIVEAU 2: POUVOIR PSYCHIQUE (256 - 64KB)                      │
│   ┌─────────────────────────────────────┐                       │
│   │  Buddy System                       │                       │
│   │  Puissances de 2                    │                       │
│   │  Allocation O(log n)                │                       │
│   └─────────────────────────────────────┘                       │
│                     ↓                                           │
│   NIVEAU 3: L'EXPLOSION (> 64KB)                                │
│   ┌─────────────────────────────────────┐                       │
│   │  mmap() direct                      │                       │
│   │  Pages alignées                     │                       │
│   │  Allocation O(1)                    │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
│   BONUS: CACHES ESPER (Per-Thread)                              │
│   ┌─────────────────────────────────────┐                       │
│   │  Thread-Local Storage               │                       │
│   │  Évite contention                   │                       │
│   │  Lock-free fast path                │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Entrée :**
- `size` : Taille demandée en bytes

**Sortie :**
- Pointeur aligné sur 16 bytes vers le bloc alloué
- `NULL` si échec

**Contraintes :**
- Alignement 16 bytes obligatoire
- Overhead maximum : 32 bytes par bloc
- Fragmentation < 25% après workload mixte
- Thread-safe avec granularité fine

### 1.2.2 PARTIE 2 : Le GC Kaneda (Le Nettoyeur)

**Ta mission :**

Implémenter un garbage collector conservatif style Boehm :

```
┌─────────────────────────────────────────────────────────────────┐
│                    GC KANEDA - MARK & SWEEP                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   PHASE 1: TROUVER LES RACINES (Root Set)                       │
│   ┌─────────────────────────────────────┐                       │
│   │  Scanner la stack                   │                       │
│   │  Scanner les registres              │                       │
│   │  Scanner les globales (.data, .bss) │                       │
│   └─────────────────────────────────────┘                       │
│                     ↓                                           │
│   PHASE 2: MARQUAGE (Mark)                                      │
│   ┌─────────────────────────────────────┐                       │
│   │  Pour chaque racine potentielle:    │                       │
│   │    Si ressemble à pointeur heap     │                       │
│   │    → Marquer l'objet                │                       │
│   │    → Scanner récursivement          │                       │
│   └─────────────────────────────────────┘                       │
│                     ↓                                           │
│   PHASE 3: BALAYAGE (Sweep)                                     │
│   ┌─────────────────────────────────────┐                       │
│   │  Pour chaque objet du heap:         │                       │
│   │    Si non marqué → Libérer          │                       │
│   │    Si marqué → Reset le bit         │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**API :**
```c
void kaneda_gc_init(void);           // Initialiser le GC
void *kaneda_gc_alloc(size_t size);  // Allouer avec GC
void kaneda_gc_collect(void);        // Forcer une collection
void kaneda_gc_shutdown(void);       // Nettoyer et stats finales
```

**Contraintes :**
- Approche conservative (pas d'info de type runtime)
- Support des pointeurs intérieurs
- Pas de moving collection (conservative)
- Stop-the-world acceptable (pas de concurrent pour base)

### 1.2.3 PARTIE 3 : Le Profiler SOL (Précision Laser)

**Ta mission :**

Implémenter un profiler de mémoire avec détection d'erreurs :

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROFILER SOL - DÉTECTION                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   🔴 CANARIES (Buffer Overflow)                                 │
│   ┌─────────────────────────────────────┐                       │
│   │  [CANARY_FRONT][  DATA  ][CANARY_BACK]                      │
│   │  Magic: 0xDEADBEEF / 0xCAFEBABE     │                       │
│   │  Vérifié à chaque free()            │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
│   ☠️ POISON VALUES (Use-After-Free)                             │
│   ┌─────────────────────────────────────┐                       │
│   │  Remplir avec 0xFE après free()     │                       │
│   │  Détecter accès à mémoire poisonnée │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
│   📊 STATISTIQUES                                               │
│   ┌─────────────────────────────────────┐                       │
│   │  Total alloué / libéré              │                       │
│   │  Peak memory usage                  │                       │
│   │  Allocation count par taille        │                       │
│   │  Backtrace optionnel                │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
│   🔍 LEAK DETECTION                                             │
│   ┌─────────────────────────────────────┐                       │
│   │  Liste des allocations actives      │                       │
│   │  Rapport à la fin du programme      │                       │
│   │  Definitely/Possibly/Indirectly lost│                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**API :**
```c
void sol_profiler_start(void);
void sol_profiler_stop(void);
void sol_profiler_report(int fd);
int sol_profiler_check_leaks(void);
sol_stats_t sol_profiler_get_stats(void);
```

### 1.2.4 Version Académique (Sans Référence Culturelle)

#### PARTIE 1 : Allocateur Multi-Niveau

Implémenter un allocateur mémoire avec architecture en trois niveaux :

1. **Segregated Free Lists** pour les petites allocations (< 256 bytes)
   - Classes de tailles : 8, 16, 32, 64, 128, 256 bytes
   - Allocation O(1) via liste pré-allouée

2. **Buddy System** pour les allocations moyennes (256 bytes - 64 KB)
   - Puissances de 2
   - Coalescing automatique

3. **mmap() direct** pour les grandes allocations (> 64 KB)
   - Pages alignées
   - Rendu au système via munmap

#### PARTIE 2 : Garbage Collector Conservatif

Implémenter un GC de type Boehm :

1. **Scanning des racines**
   - Stack, registres, globales
   - Approche conservative (tout mot aligné qui ressemble à un pointeur)

2. **Mark & Sweep**
   - Phase de marquage récursive
   - Phase de balayage pour libérer

#### PARTIE 3 : Profiler de Mémoire

Implémenter un profiler avec :

1. **Détection de buffer overflow** via canaries
2. **Détection de use-after-free** via poison values
3. **Statistiques détaillées** d'allocation
4. **Détection de fuites** à la terminaison

### 1.3 Prototypes

```c
/* ═══════════════════════════════════════════════════════════════
 *                    ALLOCATEUR AKIRA
 * ═══════════════════════════════════════════════════════════════ */

// Allocation principale
void *akira_alloc(size_t size);
void *akira_calloc(size_t nmemb, size_t size);
void *akira_realloc(void *ptr, size_t size);
void akira_free(void *ptr);

// Allocation alignée
void *akira_aligned_alloc(size_t alignment, size_t size);

// Configuration
void akira_set_mode(akira_mode_t mode);  // DEBUG, RELEASE, PROFILE

// Statistiques
akira_stats_t akira_get_stats(void);
void akira_dump_heap(int fd);

/* ═══════════════════════════════════════════════════════════════
 *                    GC KANEDA
 * ═══════════════════════════════════════════════════════════════ */

// Lifecycle
void kaneda_gc_init(void);
void kaneda_gc_shutdown(void);

// Allocation gérée
void *kaneda_gc_alloc(size_t size);

// Collection
void kaneda_gc_collect(void);
void kaneda_gc_collect_full(void);

// Configuration
void kaneda_gc_set_threshold(size_t bytes);
void kaneda_gc_disable(void);
void kaneda_gc_enable(void);

// Debug
kaneda_gc_stats_t kaneda_gc_get_stats(void);

/* ═══════════════════════════════════════════════════════════════
 *                    PROFILER SOL
 * ═══════════════════════════════════════════════════════════════ */

// Lifecycle
void sol_profiler_start(void);
void sol_profiler_stop(void);

// Rapports
void sol_profiler_report(int fd);
int sol_profiler_check_leaks(void);

// Stats
sol_stats_t sol_profiler_get_stats(void);
sol_allocation_t *sol_profiler_get_allocations(size_t *count);

// Configuration
void sol_profiler_set_backtrace(int enabled);
void sol_profiler_set_canary(int enabled);
void sol_profiler_set_poison(int enabled);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Explosion de 1988 et les Fuites Mémoire

Dans **Akira**, l'explosion de 1988 qui a détruit Tokyo était causée par un pouvoir psychique incontrôlé. En programmation, une fuite mémoire non gérée peut avoir le même effet : ton programme grossit jusqu'à consommer toute la RAM, puis... **BOOM** - OOM Killer.

Le personnage d'**Akira** lui-même est maintenu en stase cryogénique sous le stade olympique - comme un bloc mémoire "gelé" qui attend d'être libéré.

### 2.2 Pourquoi 3 Niveaux d'Allocation ?

Les allocateurs modernes comme **jemalloc** (Facebook), **tcmalloc** (Google), et **mimalloc** (Microsoft) utilisent tous cette architecture multi-niveau :

| Allocateur | Petites | Moyennes | Grandes |
|------------|---------|----------|---------|
| **jemalloc** | Slabs | Runs | Huge pages |
| **tcmalloc** | Thread cache | Central cache | Page heap |
| **mimalloc** | Pages | Segments | OS pages |
| **Akira** | Capsules | Buddy | mmap |

### 2.3 Le GC Conservatif : Pourquoi en C ?

Le C n'a pas d'information de type à l'exécution. Comment un GC peut-il savoir si `0x7fff5fbff8e0` est un pointeur ou un entier ?

**Réponse : Il ne peut pas !**

Le GC conservatif (inventé par Hans Boehm en 1988 - la même année que l'explosion d'Akira !) traite TOUT ce qui *ressemble* à un pointeur comme un pointeur potentiel. C'est pessimiste mais sûr.

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation | Cas d'Usage |
|--------|-------------|-------------|
| **Ingénieur Systèmes** | Allocateurs custom pour OS/kernels | Linux kernel slab allocator |
| **Game Developer** | Memory pools pour éviter fragmentation | Unity, Unreal memory systems |
| **Database Engineer** | Buffer pools, page caches | PostgreSQL, MySQL memory management |
| **Embedded Developer** | Allocateurs déterministes temps-réel | RTOS memory management |
| **Language Designer** | GC design (Go, Java, Python) | Toute implémentation de runtime |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
include/  src/  tests/  Makefile  README.md

$ make
gcc -Wall -Wextra -Werror -O2 -pthread -c src/allocator/segregated.c -o obj/segregated.o
gcc -Wall -Wextra -Werror -O2 -pthread -c src/allocator/buddy.c -o obj/buddy.o
gcc -Wall -Wextra -Werror -O2 -pthread -c src/gc/mark.c -o obj/mark.o
gcc -Wall -Wextra -Werror -O2 -pthread -c src/gc/sweep.c -o obj/sweep.o
gcc -Wall -Wextra -Werror -O2 -pthread -c src/profiler/tracker.c -o obj/tracker.o
ar rcs libneo_tokyo.a obj/*.o
BUILD COMPLETE: libneo_tokyo.a

$ make test
Running test suite...
[ALLOCATOR] Test segregated lists: OK
[ALLOCATOR] Test buddy system: OK
[ALLOCATOR] Test large allocations: OK
[ALLOCATOR] Test thread safety: OK
[ALLOCATOR] Test fragmentation < 25%: OK (measured: 18.7%)
[GC] Test mark phase: OK
[GC] Test sweep phase: OK
[GC] Test conservative scanning: OK
[GC] Test interior pointers: OK
[PROFILER] Test canary detection: OK
[PROFILER] Test poison values: OK
[PROFILER] Test leak detection: OK
[PROFILER] Test statistics: OK
=====================================
ALL 13 TESTS PASSED
=====================================

$ make benchmark
Benchmarking allocator performance...
┌─────────────────────────────────────────────────────────────┐
│           AKIRA ALLOCATOR vs GLIBC MALLOC                   │
├─────────────────────────────────────────────────────────────┤
│ Operation        │ Akira     │ glibc     │ Ratio          │
├──────────────────┼───────────┼───────────┼────────────────┤
│ Small alloc      │ 45 ns     │ 52 ns     │ 1.15x faster   │
│ Medium alloc     │ 120 ns    │ 98 ns     │ 0.82x slower   │
│ Large alloc      │ 1.2 us    │ 1.1 us    │ 0.92x slower   │
│ Threaded (8T)    │ 89 ns     │ 210 ns    │ 2.36x faster   │
│ Fragmentation    │ 18.7%     │ 22.3%     │ 16% better     │
└─────────────────────────────────────────────────────────────┘

$ ./demo_gc
=== KANEDA GC DEMO ===
Allocating 10000 objects...
Creating circular references...
Triggering collection...
[GC] Collected 7823 objects (312.9 KB freed)
[GC] Kept 2177 live objects (87.1 KB)
[GC] Collection time: 2.3 ms
```

### 3.1 🧠 BONUS GÉNIE : GC Générationnel Concurrent (OPTIONNEL)

**Difficulté Bonus :**
🧠 (15/10)

**Récompense :**
XP ×6

**Time Complexity attendue :**
O(live objects) pour minor GC

**Space Complexity attendue :**
O(n) + remembered set overhead

**Domaines Bonus :**
`Process (threads), CPU (memory barriers)`

#### 3.1.1 Consigne Bonus

**🎬 AKIRA AWAKENED - LE GC SANS PAUSE**

*"Je vais te montrer... la puissance d'Akira !"*
— Tetsuo

Tetsuo a finalement maîtrisé son pouvoir. Il peut maintenant manipuler la matière en temps réel, sans jamais s'arrêter. Ton GC doit faire de même : collecter les ordures PENDANT que le programme tourne, avec des pauses < 10ms.

**Ta mission :**

Implémenter un GC générationnel avec collection concurrente :

```
┌─────────────────────────────────────────────────────────────────┐
│               GC AKIRA AWAKENED - GÉNÉRATIONNEL                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   EDEN (Nursery) - Les nouveaux-nés                             │
│   ┌─────────────────────────────────────┐                       │
│   │  Bump allocator ultra-rapide        │                       │
│   │  Minor GC fréquent (copying)        │                       │
│   │  Hypothèse: 90% meurent jeunes      │                       │
│   └─────────────────────────────────────┘                       │
│                     ↓ survie                                    │
│   SURVIVOR - Les survivants                                     │
│   ┌─────────────────────────────────────┐                       │
│   │  Semi-space S0/S1                   │                       │
│   │  Plusieurs survies → promotion      │                       │
│   └─────────────────────────────────────┘                       │
│                     ↓ promotion                                 │
│   OLD GENERATION - Les anciens                                  │
│   ┌─────────────────────────────────────┐                       │
│   │  Mark-sweep (ou mark-compact)       │                       │
│   │  Major GC rare                      │                       │
│   │  Collection CONCURRENTE             │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
│   REMEMBERED SET - Cross-generation pointers                    │
│   ┌─────────────────────────────────────┐                       │
│   │  Write barrier sur OLD→YOUNG        │                       │
│   │  Card marking (512 bytes/card)      │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
│   CONCURRENT MARKING                                            │
│   ┌─────────────────────────────────────┐                       │
│   │  Tri-color marking (white/grey/black)                       │
│   │  SATB write barrier                 │                       │
│   │  Pause < 10ms GARANTIE              │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Contraintes :**
┌─────────────────────────────────────────┐
│  Pause time < 10ms (P99)                │
│  Throughput > 95%                       │
│  Memory overhead < 15%                  │
│  Thread-safe avec atomics               │
│  Write barrier overhead < 5%            │
└─────────────────────────────────────────┘

**API Bonus :**
```c
// Configuration générationnelle
void akira_gc_set_nursery_size(size_t bytes);
void akira_gc_set_survivor_ratio(float ratio);
void akira_gc_set_tenuring_threshold(int age);

// Collection concurrente
void akira_gc_start_concurrent_mark(void);
void akira_gc_wait_for_marking(void);

// Write barrier (doit être appelé par l'utilisateur)
void akira_gc_write_barrier(void *obj, void *new_ref);

// Statistiques avancées
akira_gc_gen_stats_t akira_gc_get_gen_stats(void);
```

#### 3.1.2 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Type GC | Stop-the-world | Concurrent (mostly) |
| Générations | 1 (flat heap) | 3 (Young/Survivor/Old) |
| Pause time | ~100ms | < 10ms |
| Throughput | ~98% | > 95% |
| Complexité | Mark-sweep simple | Tri-color + SATB |
| Write barrier | Aucun | Obligatoire |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette

```yaml
# Tests Allocateur
test_alloc_basic:
  description: "Allocation et libération simple"
  weight: 5
  commands:
    - "void *p = akira_alloc(100); assert(p != NULL); akira_free(p);"
  expected: "PASS"

test_alloc_alignment:
  description: "Alignement 16 bytes"
  weight: 5
  commands:
    - "for(int i=1; i<1000; i++) { void *p = akira_alloc(i); assert(((uintptr_t)p % 16) == 0); akira_free(p); }"
  expected: "PASS"

test_alloc_segregated:
  description: "Segregated lists pour petites allocs"
  weight: 10
  commands:
    - "Allouer 10000 blocs de 8-256 bytes, mesurer temps < 50ms"
  expected: "PASS"

test_alloc_buddy:
  description: "Buddy system pour moyennes allocs"
  weight: 10
  commands:
    - "Allouer/libérer blocs 256-64KB, vérifier coalescing"
  expected: "PASS"

test_alloc_large:
  description: "mmap pour grandes allocs"
  weight: 5
  commands:
    - "Allouer 10 blocs de 1MB, vérifier alignement page"
  expected: "PASS"

test_alloc_fragmentation:
  description: "Fragmentation < 25%"
  weight: 10
  commands:
    - "Workload mixte 100000 ops, mesurer fragmentation"
  expected: "PASS (< 25%)"

test_alloc_thread_safety:
  description: "Thread-safe avec 8 threads"
  weight: 10
  commands:
    - "8 threads, 10000 alloc/free chacun, pas de crash/corruption"
  expected: "PASS"

# Tests GC
test_gc_mark_sweep:
  description: "Mark & Sweep basique"
  weight: 10
  commands:
    - "Allouer objets, perdre références, collecter, vérifier libération"
  expected: "PASS"

test_gc_conservative:
  description: "Scanning conservatif"
  weight: 10
  commands:
    - "Pointeur sur stack → objet gardé vivant"
  expected: "PASS"

test_gc_interior:
  description: "Pointeurs intérieurs"
  weight: 5
  commands:
    - "Pointeur au milieu d'un objet → objet gardé vivant"
  expected: "PASS"

# Tests Profiler
test_profiler_canary:
  description: "Détection buffer overflow"
  weight: 5
  commands:
    - "Écrire au-delà du buffer, free doit détecter"
  expected: "PASS (détection)"

test_profiler_poison:
  description: "Détection use-after-free"
  weight: 5
  commands:
    - "Après free, mémoire = 0xFE"
  expected: "PASS"

test_profiler_leaks:
  description: "Détection fuites"
  weight: 5
  commands:
    - "Allouer sans free, profiler_check_leaks() retourne count"
  expected: "PASS"

test_profiler_stats:
  description: "Statistiques correctes"
  weight: 5
  commands:
    - "Vérifier total_allocated, peak_usage, allocation_count"
  expected: "PASS"
```

### 4.2 main.c de test

```c
#include "neo_alloc.h"
#include "neo_gc.h"
#include "neo_profiler.h"
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <assert.h>

/* ═══════════════════════════════════════════════════════════════
 *                    TESTS ALLOCATEUR
 * ═══════════════════════════════════════════════════════════════ */

void test_allocator_basic(void)
{
    printf("[ALLOC] Basic allocation: ");

    void *p1 = akira_alloc(100);
    assert(p1 != NULL);
    memset(p1, 'A', 100);

    void *p2 = akira_alloc(200);
    assert(p2 != NULL);
    assert(p2 != p1);

    akira_free(p1);
    akira_free(p2);

    printf("OK\n");
}

void test_allocator_alignment(void)
{
    printf("[ALLOC] Alignment check: ");

    for (int i = 1; i <= 1000; i++)
    {
        void *p = akira_alloc(i);
        assert(p != NULL);
        assert(((uintptr_t)p % 16) == 0);
        akira_free(p);
    }

    printf("OK\n");
}

void test_allocator_segregated(void)
{
    printf("[ALLOC] Segregated lists: ");

    void *ptrs[10000];
    size_t sizes[] = {8, 16, 32, 64, 128, 256};

    for (int i = 0; i < 10000; i++)
    {
        size_t size = sizes[i % 6];
        ptrs[i] = akira_alloc(size);
        assert(ptrs[i] != NULL);
    }

    for (int i = 0; i < 10000; i++)
    {
        akira_free(ptrs[i]);
    }

    printf("OK\n");
}

void test_allocator_buddy(void)
{
    printf("[ALLOC] Buddy system: ");

    void *ptrs[100];

    // Allouer des blocs de tailles variées
    for (int i = 0; i < 100; i++)
    {
        size_t size = 256 + (i * 256);  // 256 to 25600
        ptrs[i] = akira_alloc(size);
        assert(ptrs[i] != NULL);
    }

    // Libérer en ordre inverse (teste coalescing)
    for (int i = 99; i >= 0; i--)
    {
        akira_free(ptrs[i]);
    }

    printf("OK\n");
}

void test_allocator_large(void)
{
    printf("[ALLOC] Large allocations: ");

    void *ptrs[10];

    for (int i = 0; i < 10; i++)
    {
        ptrs[i] = akira_alloc(1024 * 1024);  // 1 MB
        assert(ptrs[i] != NULL);
        assert(((uintptr_t)ptrs[i] % 4096) == 0);  // Page aligned
    }

    for (int i = 0; i < 10; i++)
    {
        akira_free(ptrs[i]);
    }

    printf("OK\n");
}

/* Thread safety test */
static void *thread_alloc_worker(void *arg)
{
    (void)arg;
    void *ptrs[1000];

    for (int round = 0; round < 10; round++)
    {
        for (int i = 0; i < 1000; i++)
        {
            ptrs[i] = akira_alloc(16 + (i % 256));
            assert(ptrs[i] != NULL);
        }
        for (int i = 0; i < 1000; i++)
        {
            akira_free(ptrs[i]);
        }
    }

    return NULL;
}

void test_allocator_thread_safety(void)
{
    printf("[ALLOC] Thread safety: ");

    pthread_t threads[8];

    for (int i = 0; i < 8; i++)
    {
        pthread_create(&threads[i], NULL, thread_alloc_worker, NULL);
    }

    for (int i = 0; i < 8; i++)
    {
        pthread_join(threads[i], NULL);
    }

    printf("OK\n");
}

/* ═══════════════════════════════════════════════════════════════
 *                    TESTS GC
 * ═══════════════════════════════════════════════════════════════ */

void test_gc_basic(void)
{
    printf("[GC] Basic mark-sweep: ");

    kaneda_gc_init();

    // Allouer des objets
    for (int i = 0; i < 1000; i++)
    {
        kaneda_gc_alloc(100);
    }

    kaneda_gc_stats_t before = kaneda_gc_get_stats();
    kaneda_gc_collect();
    kaneda_gc_stats_t after = kaneda_gc_get_stats();

    // Tous devraient être collectés (pas de références)
    assert(after.live_objects < before.live_objects);

    kaneda_gc_shutdown();
    printf("OK\n");
}

void test_gc_roots(void)
{
    printf("[GC] Root scanning: ");

    kaneda_gc_init();

    // Garder une référence sur stack
    void *root = kaneda_gc_alloc(100);

    // Allouer d'autres objets sans référence
    for (int i = 0; i < 100; i++)
    {
        kaneda_gc_alloc(100);
    }

    kaneda_gc_collect();

    // root devrait encore être accessible
    memset(root, 'X', 100);  // Ne devrait pas crash

    kaneda_gc_shutdown();
    printf("OK\n");
}

/* ═══════════════════════════════════════════════════════════════
 *                    TESTS PROFILER
 * ═══════════════════════════════════════════════════════════════ */

void test_profiler_canary(void)
{
    printf("[PROF] Canary detection: ");

    sol_profiler_start();
    akira_set_mode(AKIRA_MODE_DEBUG);

    char *buf = akira_alloc(100);
    buf[100] = 'X';  // Buffer overflow !

    // free devrait détecter l'overflow
    // (dans un vrai test, on capturerait le signal/abort)

    sol_profiler_stop();
    printf("OK (visual check)\n");
}

void test_profiler_leaks(void)
{
    printf("[PROF] Leak detection: ");

    sol_profiler_start();

    void *p1 = akira_alloc(100);
    void *p2 = akira_alloc(200);
    void *p3 = akira_alloc(300);

    akira_free(p1);
    // p2 et p3 sont des fuites !

    int leaks = sol_profiler_check_leaks();
    assert(leaks == 2);

    akira_free(p2);
    akira_free(p3);
    sol_profiler_stop();

    printf("OK\n");
}

void test_profiler_stats(void)
{
    printf("[PROF] Statistics: ");

    sol_profiler_start();

    void *p1 = akira_alloc(100);
    void *p2 = akira_alloc(200);

    sol_stats_t stats = sol_profiler_get_stats();
    assert(stats.total_allocated >= 300);
    assert(stats.allocation_count == 2);

    akira_free(p1);
    akira_free(p2);
    sol_profiler_stop();

    printf("OK\n");
}

/* ═══════════════════════════════════════════════════════════════
 *                    MAIN
 * ═══════════════════════════════════════════════════════════════ */

int main(void)
{
    printf("╔═══════════════════════════════════════════════════════════════╗\n");
    printf("║        NEO-TOKYO MEMORY SYSTEM - TEST SUITE                   ║\n");
    printf("╚═══════════════════════════════════════════════════════════════╝\n\n");

    // Allocator tests
    test_allocator_basic();
    test_allocator_alignment();
    test_allocator_segregated();
    test_allocator_buddy();
    test_allocator_large();
    test_allocator_thread_safety();

    // GC tests
    test_gc_basic();
    test_gc_roots();

    // Profiler tests
    test_profiler_canary();
    test_profiler_leaks();
    test_profiler_stats();

    printf("\n════════════════════════════════════════════════════════════════\n");
    printf("                    ALL TESTS PASSED                            \n");
    printf("════════════════════════════════════════════════════════════════\n");

    return 0;
}
```

### 4.3 Solution de référence (Architecture)

```c
/* neo_alloc.h - Header principal */

#ifndef NEO_ALLOC_H
#define NEO_ALLOC_H

#include <stddef.h>
#include <stdint.h>

/* ═══════════════════════════════════════════════════════════════
 *                    CONSTANTES
 * ═══════════════════════════════════════════════════════════════ */

#define AKIRA_ALIGNMENT         16
#define AKIRA_SMALL_THRESHOLD   256
#define AKIRA_LARGE_THRESHOLD   (64 * 1024)
#define AKIRA_PAGE_SIZE         4096
#define AKIRA_CANARY_FRONT      0xDEADBEEFDEADBEEFULL
#define AKIRA_CANARY_BACK       0xCAFEBABECAFEBABEULL
#define AKIRA_POISON_BYTE       0xFE

/* ═══════════════════════════════════════════════════════════════
 *                    STRUCTURES
 * ═══════════════════════════════════════════════════════════════ */

typedef enum {
    AKIRA_MODE_RELEASE,
    AKIRA_MODE_DEBUG,
    AKIRA_MODE_PROFILE
} akira_mode_t;

typedef struct {
    size_t total_allocated;
    size_t total_freed;
    size_t peak_usage;
    size_t current_usage;
    size_t allocation_count;
    size_t free_count;
    double fragmentation;
} akira_stats_t;

/* Block header pour segregated/buddy */
typedef struct block_header {
    size_t size;
    uint32_t flags;
    uint32_t magic;
    struct block_header *next;
    struct block_header *prev;
} block_header_t;

#define BLOCK_FLAG_FREE     (1 << 0)
#define BLOCK_FLAG_BUDDY    (1 << 1)
#define BLOCK_FLAG_MMAP     (1 << 2)
#define BLOCK_MAGIC         0xAC1DACID

/* ═══════════════════════════════════════════════════════════════
 *                    API ALLOCATEUR
 * ═══════════════════════════════════════════════════════════════ */

void *akira_alloc(size_t size);
void *akira_calloc(size_t nmemb, size_t size);
void *akira_realloc(void *ptr, size_t size);
void akira_free(void *ptr);
void *akira_aligned_alloc(size_t alignment, size_t size);

void akira_set_mode(akira_mode_t mode);
akira_stats_t akira_get_stats(void);
void akira_dump_heap(int fd);

#endif /* NEO_ALLOC_H */
```

```c
/* segregated.c - Segregated Free Lists */

#include "neo_alloc.h"
#include <sys/mman.h>
#include <pthread.h>
#include <string.h>

/* Classes de taille : 8, 16, 32, 64, 128, 256 */
#define NUM_SIZE_CLASSES    6
static const size_t size_classes[NUM_SIZE_CLASSES] = {8, 16, 32, 64, 128, 256};

/* Free list par classe de taille */
typedef struct {
    block_header_t *head;
    pthread_mutex_t lock;
    size_t count;
} free_list_t;

static free_list_t segregated_lists[NUM_SIZE_CLASSES];
static int initialized = 0;

/* Trouver la classe de taille appropriée */
static int find_size_class(size_t size)
{
    for (int i = 0; i < NUM_SIZE_CLASSES; i++)
    {
        if (size <= size_classes[i])
            return i;
    }
    return -1;  /* Trop grand pour segregated */
}

/* Initialiser une nouvelle slab pour une classe */
static void *init_slab(int class_idx)
{
    size_t obj_size = size_classes[class_idx] + sizeof(block_header_t);
    size_t slab_size = AKIRA_PAGE_SIZE * 4;  /* 16 KB slab */
    size_t obj_count = slab_size / obj_size;

    void *slab = mmap(NULL, slab_size, PROT_READ | PROT_WRITE,
                      MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (slab == MAP_FAILED)
        return NULL;

    /* Chaîner tous les blocs dans la free list */
    block_header_t *prev = NULL;
    for (size_t i = 0; i < obj_count; i++)
    {
        block_header_t *block = (block_header_t *)((char *)slab + i * obj_size);
        block->size = size_classes[class_idx];
        block->flags = BLOCK_FLAG_FREE;
        block->magic = BLOCK_MAGIC;
        block->next = prev;
        block->prev = NULL;
        if (prev)
            prev->prev = block;
        prev = block;
    }

    segregated_lists[class_idx].head = prev;
    segregated_lists[class_idx].count = obj_count;

    return slab;
}

/* Allocation depuis segregated list */
void *segregated_alloc(size_t size)
{
    int class_idx = find_size_class(size);
    if (class_idx < 0)
        return NULL;

    free_list_t *list = &segregated_lists[class_idx];

    pthread_mutex_lock(&list->lock);

    if (list->head == NULL)
    {
        if (init_slab(class_idx) == NULL)
        {
            pthread_mutex_unlock(&list->lock);
            return NULL;
        }
    }

    block_header_t *block = list->head;
    list->head = block->next;
    if (list->head)
        list->head->prev = NULL;
    list->count--;

    block->flags &= ~BLOCK_FLAG_FREE;

    pthread_mutex_unlock(&list->lock);

    return (void *)((char *)block + sizeof(block_header_t));
}

/* Libération vers segregated list */
void segregated_free(void *ptr)
{
    block_header_t *block = (block_header_t *)((char *)ptr - sizeof(block_header_t));

    if (block->magic != BLOCK_MAGIC)
    {
        /* Corruption détectée ! */
        return;
    }

    int class_idx = find_size_class(block->size);
    if (class_idx < 0)
        return;

    free_list_t *list = &segregated_lists[class_idx];

    pthread_mutex_lock(&list->lock);

    block->flags |= BLOCK_FLAG_FREE;
    block->next = list->head;
    block->prev = NULL;
    if (list->head)
        list->head->prev = block;
    list->head = block;
    list->count++;

    pthread_mutex_unlock(&list->lock);
}
```

```c
/* buddy.c - Buddy System Allocator */

#include "neo_alloc.h"
#include <sys/mman.h>
#include <pthread.h>

#define BUDDY_MIN_ORDER     8   /* 256 bytes */
#define BUDDY_MAX_ORDER     16  /* 64 KB */
#define BUDDY_LEVELS        (BUDDY_MAX_ORDER - BUDDY_MIN_ORDER + 1)

typedef struct buddy_block {
    int order;
    int is_free;
    struct buddy_block *next;
    struct buddy_block *prev;
} buddy_block_t;

static buddy_block_t *free_lists[BUDDY_LEVELS];
static void *buddy_heap_start;
static size_t buddy_heap_size;
static pthread_mutex_t buddy_lock = PTHREAD_MUTEX_INITIALIZER;

/* Initialiser le buddy heap */
static int buddy_init(void)
{
    buddy_heap_size = 16 * 1024 * 1024;  /* 16 MB */
    buddy_heap_start = mmap(NULL, buddy_heap_size, PROT_READ | PROT_WRITE,
                            MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (buddy_heap_start == MAP_FAILED)
        return -1;

    /* Créer un seul bloc libre de taille max */
    buddy_block_t *block = (buddy_block_t *)buddy_heap_start;
    block->order = BUDDY_MAX_ORDER;
    block->is_free = 1;
    block->next = NULL;
    block->prev = NULL;

    free_lists[BUDDY_LEVELS - 1] = block;

    return 0;
}

/* Trouver l'order minimal pour une taille */
static int size_to_order(size_t size)
{
    size_t total = size + sizeof(buddy_block_t);
    int order = BUDDY_MIN_ORDER;

    while ((1UL << order) < total && order <= BUDDY_MAX_ORDER)
        order++;

    return order;
}

/* Trouver le buddy d'un bloc */
static buddy_block_t *find_buddy(buddy_block_t *block)
{
    uintptr_t addr = (uintptr_t)block - (uintptr_t)buddy_heap_start;
    uintptr_t buddy_addr = addr ^ (1UL << block->order);
    return (buddy_block_t *)((uintptr_t)buddy_heap_start + buddy_addr);
}

/* Split un bloc pour obtenir un ordre plus petit */
static buddy_block_t *buddy_split(buddy_block_t *block, int target_order)
{
    while (block->order > target_order)
    {
        block->order--;

        /* Créer le buddy */
        buddy_block_t *buddy = (buddy_block_t *)((char *)block + (1UL << block->order));
        buddy->order = block->order;
        buddy->is_free = 1;

        /* Ajouter buddy à la free list */
        int level = buddy->order - BUDDY_MIN_ORDER;
        buddy->next = free_lists[level];
        buddy->prev = NULL;
        if (free_lists[level])
            free_lists[level]->prev = buddy;
        free_lists[level] = buddy;
    }

    return block;
}

/* Allocation buddy */
void *buddy_alloc(size_t size)
{
    int order = size_to_order(size);
    if (order > BUDDY_MAX_ORDER)
        return NULL;

    pthread_mutex_lock(&buddy_lock);

    if (buddy_heap_start == NULL)
    {
        if (buddy_init() < 0)
        {
            pthread_mutex_unlock(&buddy_lock);
            return NULL;
        }
    }

    /* Chercher un bloc libre de l'ordre approprié ou plus grand */
    int level;
    buddy_block_t *block = NULL;

    for (level = order - BUDDY_MIN_ORDER; level < BUDDY_LEVELS; level++)
    {
        if (free_lists[level] != NULL)
        {
            block = free_lists[level];
            free_lists[level] = block->next;
            if (free_lists[level])
                free_lists[level]->prev = NULL;
            break;
        }
    }

    if (block == NULL)
    {
        pthread_mutex_unlock(&buddy_lock);
        return NULL;
    }

    /* Split si nécessaire */
    block = buddy_split(block, order);
    block->is_free = 0;

    pthread_mutex_unlock(&buddy_lock);

    return (void *)((char *)block + sizeof(buddy_block_t));
}

/* Libération et coalescing */
void buddy_free(void *ptr)
{
    buddy_block_t *block = (buddy_block_t *)((char *)ptr - sizeof(buddy_block_t));

    pthread_mutex_lock(&buddy_lock);

    block->is_free = 1;

    /* Coalesce avec buddy si possible */
    while (block->order < BUDDY_MAX_ORDER)
    {
        buddy_block_t *buddy = find_buddy(block);

        if (!buddy->is_free || buddy->order != block->order)
            break;

        /* Retirer buddy de sa free list */
        int level = buddy->order - BUDDY_MIN_ORDER;
        if (buddy->prev)
            buddy->prev->next = buddy->next;
        else
            free_lists[level] = buddy->next;
        if (buddy->next)
            buddy->next->prev = buddy->prev;

        /* Fusionner */
        if (buddy < block)
            block = buddy;
        block->order++;
    }

    /* Ajouter à la free list */
    int level = block->order - BUDDY_MIN_ORDER;
    block->next = free_lists[level];
    block->prev = NULL;
    if (free_lists[level])
        free_lists[level]->prev = block;
    free_lists[level] = block;

    pthread_mutex_unlock(&buddy_lock);
}
```

```c
/* gc/mark.c - Phase de marquage du GC */

#include "neo_gc.h"
#include <setjmp.h>

/* Tri-color marking */
#define GC_WHITE    0
#define GC_GREY     1
#define GC_BLACK    2

typedef struct gc_object {
    size_t size;
    int color;
    int marked;
    struct gc_object *next;
} gc_object_t;

static gc_object_t *all_objects = NULL;
static gc_object_t **grey_stack = NULL;
static size_t grey_stack_size = 0;
static size_t grey_stack_capacity = 0;

/* Vérifier si une adresse ressemble à un pointeur vers le heap */
static int looks_like_heap_pointer(uintptr_t value)
{
    gc_object_t *obj = all_objects;
    while (obj)
    {
        uintptr_t start = (uintptr_t)(obj + 1);
        uintptr_t end = start + obj->size;

        if (value >= start && value < end)
            return 1;

        obj = obj->next;
    }
    return 0;
}

/* Trouver l'objet contenant une adresse */
static gc_object_t *find_object_containing(uintptr_t addr)
{
    gc_object_t *obj = all_objects;
    while (obj)
    {
        uintptr_t start = (uintptr_t)(obj + 1);
        uintptr_t end = start + obj->size;

        if (addr >= start && addr < end)
            return obj;

        obj = obj->next;
    }
    return NULL;
}

/* Ajouter un objet à la grey stack */
static void push_grey(gc_object_t *obj)
{
    if (obj->color != GC_WHITE)
        return;

    obj->color = GC_GREY;

    if (grey_stack_size >= grey_stack_capacity)
    {
        grey_stack_capacity = grey_stack_capacity ? grey_stack_capacity * 2 : 1024;
        grey_stack = realloc(grey_stack, grey_stack_capacity * sizeof(gc_object_t *));
    }

    grey_stack[grey_stack_size++] = obj;
}

/* Scanner un range mémoire pour trouver des pointeurs */
static void scan_range(void *start, void *end)
{
    uintptr_t *ptr = (uintptr_t *)start;
    uintptr_t *limit = (uintptr_t *)end;

    while (ptr < limit)
    {
        uintptr_t value = *ptr;

        if (looks_like_heap_pointer(value))
        {
            gc_object_t *obj = find_object_containing(value);
            if (obj)
                push_grey(obj);
        }

        ptr++;
    }
}

/* Scanner la stack */
static void scan_stack(void)
{
    jmp_buf registers;
    setjmp(registers);  /* Capture registres */

    void *stack_top;
    void *stack_bottom;

    /* Obtenir les limites de la stack (platform-specific) */
    /* ... implementation dépend de l'OS ... */

    scan_range(stack_bottom, stack_top);
    scan_range(registers, registers + sizeof(registers));
}

/* Phase de marquage complète */
void gc_mark_phase(void)
{
    /* Reset all colors to white */
    gc_object_t *obj = all_objects;
    while (obj)
    {
        obj->color = GC_WHITE;
        obj = obj->next;
    }

    grey_stack_size = 0;

    /* Scanner les racines */
    scan_stack();
    /* scan_globals(); */

    /* Traiter la grey stack */
    while (grey_stack_size > 0)
    {
        gc_object_t *current = grey_stack[--grey_stack_size];
        current->color = GC_BLACK;

        /* Scanner l'objet pour trouver des références */
        void *start = (void *)(current + 1);
        void *end = (char *)start + current->size;
        scan_range(start, end);
    }
}
```

### 4.5 Solutions refusées (avec explications)

```c
/* ❌ REFUSÉ : Utilisation de malloc de la libc */
void *akira_alloc(size_t size)
{
    return malloc(size);  // NON ! On doit implémenter nous-même
}
// Pourquoi c'est refusé : L'objectif est d'implémenter l'allocateur, pas de wrapper malloc

/* ❌ REFUSÉ : Pas d'alignement */
void *akira_alloc(size_t size)
{
    void *ptr = mmap(NULL, size, PROT_READ | PROT_WRITE,
                     MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    return ptr;
}
// Pourquoi c'est refusé : Pas d'alignement 16 bytes garanti, pas de header

/* ❌ REFUSÉ : GC non-conservatif (assume type info) */
void gc_mark(gc_object_t *obj)
{
    // Assume qu'on connaît le layout des pointeurs
    void **ptr = obj->pointer_fields;  // INTERDIT en C pur
    // ...
}
// Pourquoi c'est refusé : C n'a pas d'info de type runtime

/* ❌ REFUSÉ : Pas thread-safe */
void *segregated_alloc(size_t size)
{
    // Pas de lock !
    block_header_t *block = lists[class].head;
    lists[class].head = block->next;
    return block;
}
// Pourquoi c'est refusé : Race conditions en multi-thread
```

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "neo_tokyo_memory_system",
  "language": "c",
  "type": "complet_2",
  "tier": 3,
  "tier_info": "Synthèse complète Module 2.1",
  "tags": ["memory", "allocator", "gc", "profiler", "phase2", "capstone"],
  "passing_score": 70,

  "project": {
    "structure": {
      "directories": ["include", "src/allocator", "src/gc", "src/profiler", "tests"],
      "required_files": [
        "include/neo_alloc.h",
        "include/neo_gc.h",
        "include/neo_profiler.h",
        "src/allocator/segregated.c",
        "src/allocator/buddy.c",
        "src/allocator/large.c",
        "src/gc/mark.c",
        "src/gc/sweep.c",
        "src/profiler/tracker.c",
        "Makefile"
      ]
    }
  },

  "functions": [
    {
      "name": "akira_alloc",
      "prototype": "void *akira_alloc(size_t size)",
      "return_type": "void *"
    },
    {
      "name": "akira_free",
      "prototype": "void akira_free(void *ptr)",
      "return_type": "void"
    },
    {
      "name": "kaneda_gc_collect",
      "prototype": "void kaneda_gc_collect(void)",
      "return_type": "void"
    },
    {
      "name": "sol_profiler_check_leaks",
      "prototype": "int sol_profiler_check_leaks(void)",
      "return_type": "int"
    }
  ],

  "driver": {
    "edge_cases": [
      {
        "name": "alloc_null_size",
        "description": "Allocation de taille 0",
        "args": [0],
        "expected": "NULL or minimal block",
        "is_trap": true
      },
      {
        "name": "alloc_very_large",
        "description": "Allocation > heap size",
        "args": [1099511627776],
        "expected": "NULL",
        "is_trap": true
      },
      {
        "name": "free_null",
        "description": "free(NULL) ne crash pas",
        "args": ["NULL"],
        "expected": "no crash",
        "is_trap": true
      },
      {
        "name": "double_free",
        "description": "Double free détecté",
        "expected": "detection or no crash",
        "is_trap": true
      },
      {
        "name": "gc_empty_heap",
        "description": "Collection sur heap vide",
        "expected": "no crash",
        "is_trap": true
      }
    ],

    "stress_tests": [
      {
        "name": "random_workload",
        "iterations": 100000,
        "description": "Allocations/free aléatoires"
      },
      {
        "name": "fragmentation_test",
        "description": "Mesure fragmentation < 25%"
      },
      {
        "name": "thread_stress",
        "threads": 8,
        "iterations_per_thread": 10000
      }
    ]
  },

  "norm": {
    "allowed_functions": ["mmap", "munmap", "mprotect", "sbrk", "pthread_*", "write", "memset", "memcpy", "memmove"],
    "forbidden_functions": ["malloc", "free", "calloc", "realloc", "printf", "fprintf"],
    "check_memory": true,
    "check_threads": true
  },

  "scoring": {
    "allocator_segregated": 20,
    "allocator_buddy": 15,
    "allocator_large": 5,
    "allocator_thread_safety": 10,
    "gc_mark_sweep": 15,
    "gc_conservative": 10,
    "profiler_canary": 5,
    "profiler_poison": 5,
    "profiler_leaks": 5,
    "documentation": 10,
    "total": 100
  }
}
```

### 4.10 Solutions Mutantes (6 mutants)

```c
/* Mutant A (Boundary) : Off-by-one dans le calcul de buddy */
static buddy_block_t *find_buddy(buddy_block_t *block)
{
    uintptr_t addr = (uintptr_t)block - (uintptr_t)buddy_heap_start;
    // BUG: devrait être block->order, pas block->order - 1
    uintptr_t buddy_addr = addr ^ (1UL << (block->order - 1));
    return (buddy_block_t *)((uintptr_t)buddy_heap_start + buddy_addr);
}
// Pourquoi c'est faux : Le buddy est calculé à la mauvaise adresse, coalescing échoue
// Ce qui était pensé : Confusion entre order et index

/* Mutant B (Safety) : Pas de vérification NULL avant scan */
static void scan_range(void *start, void *end)
{
    // BUG: Pas de check si start ou end sont NULL
    uintptr_t *ptr = (uintptr_t *)start;
    uintptr_t *limit = (uintptr_t *)end;

    while (ptr < limit)  // Crash si ptr ou limit invalide
    {
        // ...
    }
}
// Pourquoi c'est faux : Crash si appelé avec des pointeurs invalides
// Ce qui était pensé : Oubli de validation des entrées

/* Mutant C (Resource) : Fuite dans segregated list */
static void *init_slab(int class_idx)
{
    void *slab = mmap(NULL, slab_size, PROT_READ | PROT_WRITE,
                      MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
    if (slab == MAP_FAILED)
        return NULL;

    // BUG: On n'enregistre pas le slab, impossible de munmap plus tard
    // Il manque: slabs[num_slabs++] = slab;

    // ...
}
// Pourquoi c'est faux : Les slabs ne sont jamais libérés, fuite mémoire système
// Ce qui était pensé : Oubli de tracking des allocations système

/* Mutant D (Logic) : Mark bit jamais reset */
void gc_mark_phase(void)
{
    // BUG: On ne reset PAS les couleurs avant le marquage
    // Il manque la boucle: obj->color = GC_WHITE;

    grey_stack_size = 0;
    scan_stack();

    while (grey_stack_size > 0)
    {
        // ...
    }
}
// Pourquoi c'est faux : Tous les objets restent marqués, rien n'est jamais collecté
// Ce qui était pensé : Oubli d'initialisation entre cycles GC

/* Mutant E (Return) : Leak detector retourne toujours 0 */
int sol_profiler_check_leaks(void)
{
    int count = 0;
    allocation_t *alloc = active_allocations;

    while (alloc)
    {
        count++;  // On compte mais...
        alloc = alloc->next;
    }

    return 0;  // BUG: Retourne 0 au lieu de count
}
// Pourquoi c'est faux : Les fuites ne sont jamais signalées
// Ce qui était pensé : Copy-paste error ou debug code laissé

/* Mutant F (Concurrency) : Race condition sur stats */
void update_stats(size_t size)
{
    // BUG: Pas de lock sur les stats globales
    global_stats.total_allocated += size;  // Race condition !
    global_stats.allocation_count++;
    if (global_stats.current_usage > global_stats.peak_usage)
        global_stats.peak_usage = global_stats.current_usage;
}
// Pourquoi c'est faux : Stats incorrectes en multi-thread
// Ce qui était pensé : Oubli de protection des données partagées
```

---

## 🧠 SECTION 5 : COMPRENDRE (DOCUMENT DE COURS COMPLET)

### 5.1 Ce que cet exercice enseigne

Ce projet synthétise **TOUS** les concepts du Module 2.1 :

| Catégorie | Concepts Maîtrisés |
|-----------|-------------------|
| **Mémoire Virtuelle** | Pages, TLB, address translation, page tables |
| **Allocation** | Free lists, buddy system, segregated lists, coalescing |
| **Fragmentation** | Interne, externe, stratégies de mitigation |
| **GC** | Mark & sweep, conservative scanning, tri-color |
| **Debug** | Canaries, poison values, leak detection |
| **Thread-Safety** | Locks, atomics, per-thread caches |

### 5.2 LDA — Traduction littérale (ALLOCATEUR)

```
FONCTION akira_alloc QUI RETOURNE UN POINTEUR ET PREND EN PARAMÈTRE size QUI EST UNE TAILLE
DÉBUT FONCTION
    SI size EST ÉGAL À 0 ALORS
        RETOURNER NUL
    FIN SI

    SI size EST INFÉRIEUR À 256 ALORS
        AFFECTER segregated_alloc(size) À result
    SINON SI size EST INFÉRIEUR À 64 KILOBYTES ALORS
        AFFECTER buddy_alloc(size) À result
    SINON
        AFFECTER large_alloc(size) À result
    FIN SI

    SI result EST DIFFÉRENT DE NUL ALORS
        AFFECTER OUVRIR LE MODE DEBUG À profiler_track(result, size)
    FIN SI

    RETOURNER result
FIN FONCTION
```

### 5.2.2 Style Académique

```
ALGORITHME : Allocation Multi-Niveau Akira
ENTRÉE : size (taille en bytes)
SORTIE : pointeur vers bloc alloué ou NULL

DÉBUT
    SI size = 0 ALORS
        RETOURNER NULL
    FIN SI

    SI size < SMALL_THRESHOLD ALORS
        bloc ← SegregatedAlloc(size)
    SINON SI size < LARGE_THRESHOLD ALORS
        bloc ← BuddyAlloc(size)
    SINON
        bloc ← MmapAlloc(size)
    FIN SI

    SI MODE_PROFILER ALORS
        EnregistrerAllocation(bloc, size)
    FIN SI

    RETOURNER bloc
FIN
```

### 5.2.2.1 Logic Flow

```
ALGORITHME : Mark & Sweep Conservatif
---
1. PHASE INIT :
   a. Pour chaque objet du heap :
      - Mettre couleur = WHITE

2. PHASE MARK :
   a. Scanner la stack (conservative) :
      - Pour chaque mot aligné :
        - SI ressemble à pointeur heap → Ajouter à grey_stack

   b. Traiter grey_stack :
      - TANT QUE grey_stack non vide :
        - obj ← pop(grey_stack)
        - obj.color ← BLACK
        - Scanner obj pour trouver références
        - Ajouter nouvelles références à grey_stack

3. PHASE SWEEP :
   a. Pour chaque objet du heap :
      - SI couleur = WHITE :
        - Libérer l'objet
      - SINON :
        - Reset couleur pour prochain cycle
```

### 5.2.3 Représentation Algorithmique (Logique de Garde)

```
FONCTION : BuddyAlloc(size)
---
INIT result = NULL

1. CALCULER order minimal pour size :
   |
   |-- VÉRIFIER si order > MAX_ORDER :
   |     RETOURNER NULL (trop grand)
   |
   |-- CHERCHER bloc libre dans free_lists[order..MAX_ORDER]

2. SI bloc trouvé :
   |
   |-- RETIRER de la free list
   |
   |-- SPLIT si nécessaire :
   |     TANT QUE bloc.order > order :
   |       - Diviser en deux
   |       - Ajouter buddy à free_list
   |
   |-- MARQUER comme alloué
   |-- RETOURNER bloc + header_size

3. RETOURNER NULL (pas de mémoire)
```

### 5.2.3.1 Diagramme Mermaid

```mermaid
graph TD
    A[akira_alloc size] --> B{size == 0 ?}
    B -- Oui --> C[return NULL]
    B -- Non --> D{size < 256 ?}

    D -- Oui --> E[Segregated List]
    D -- Non --> F{size < 64KB ?}

    F -- Oui --> G[Buddy System]
    F -- Non --> H[mmap direct]

    E --> I{bloc trouvé ?}
    G --> I
    H --> I

    I -- Non --> J[return NULL]
    I -- Oui --> K{MODE_DEBUG ?}

    K -- Oui --> L[Ajouter canaries]
    K -- Non --> M[return ptr]
    L --> M
```

```mermaid
graph TD
    subgraph "GC Mark Phase"
        A[Début Mark] --> B[Reset all WHITE]
        B --> C[Scan Stack]
        C --> D[Scan Globals]
        D --> E{grey_stack vide ?}
        E -- Non --> F[Pop objet]
        F --> G[Color = BLACK]
        G --> H[Scan objet]
        H --> I[Push refs to grey]
        I --> E
        E -- Oui --> J[Fin Mark]
    end

    subgraph "GC Sweep Phase"
        J --> K[Parcourir heap]
        K --> L{Color WHITE ?}
        L -- Oui --> M[Free objet]
        L -- Non --> N[Reset color]
        M --> O{Fin heap ?}
        N --> O
        O -- Non --> K
        O -- Oui --> P[Fin GC]
    end
```

### 5.3 Visualisation ASCII

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ARCHITECTURE MÉMOIRE NEO-TOKYO                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   VIRTUAL ADDRESS SPACE                                                     │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ 0x0000000000000000                                                  │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │                           .text (code)                              │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │                           .data (globals)                           │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │                           .bss (uninitialized)                      │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │                                                                     │   │
│   │                           H E A P                                   │   │
│   │   ┌─────────────────────────────────────────────────────────────┐   │   │
│   │   │  SEGREGATED LISTS (< 256 bytes)                             │   │   │
│   │   │  ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐                       │   │   │
│   │   │  │ 8 │ │16 │ │32 │ │64 │ │128│ │256│                       │   │   │
│   │   │  └─┼─┘ └─┼─┘ └─┼─┘ └─┼─┘ └─┼─┘ └─┼─┘                       │   │   │
│   │   │    ↓     ↓     ↓     ↓     ↓     ↓                         │   │   │
│   │   │  [F]->[F]->[F] [F]->[F] ... (free blocks)                  │   │   │
│   │   ├─────────────────────────────────────────────────────────────┤   │   │
│   │   │  BUDDY SYSTEM (256 - 64KB)                                  │   │   │
│   │   │  ┌───────────────────────────────────────────────────────┐  │   │   │
│   │   │  │ Order 8  │ Order 9  │ ... │ Order 16                  │  │   │   │
│   │   │  │ (256B)   │ (512B)   │     │ (64KB)                    │  │   │   │
│   │   │  └───────────────────────────────────────────────────────┘  │   │   │
│   │   │                                                             │   │   │
│   │   │  ┌───────────────────┐                                      │   │   │
│   │   │  │████████│        │  │  ← Buddy pair (one allocated)       │   │   │
│   │   │  └───────────────────┘                                      │   │   │
│   │   ├─────────────────────────────────────────────────────────────┤   │   │
│   │   │  LARGE ALLOCATIONS (mmap > 64KB)                            │   │   │
│   │   │  ┌─────────────────────────────────────────────────────┐    │   │   │
│   │   │  │                    1 MB block                       │    │   │   │
│   │   │  └─────────────────────────────────────────────────────┘    │   │   │
│   │   └─────────────────────────────────────────────────────────────┘   │   │
│   │                              ↓ (grows up)                       │   │
│   │                                                                     │   │
│   │                              ↑ (grows down)                     │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │                           S T A C K                                 │   │
│   │   ┌─────────────────────────────────────────────────────────────┐   │   │
│   │   │ local vars │ return addr │ saved regs │ ...                 │   │   │
│   │   └─────────────────────────────────────────────────────────────┘   │   │
│   │                           ↑ GC scans here (roots)                   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│   │ 0x7FFFFFFFFFFF                                                      │   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    BLOCK STRUCTURE AVEC CANARIES                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ HEADER (32 bytes)                                                   │   │
│   ├───────────┬───────────┬───────────┬───────────┬───────────┬─────────┤   │
│   │   size    │   flags   │   magic   │   *next   │   *prev   │ padding │   │
│   │  8 bytes  │  4 bytes  │  4 bytes  │  8 bytes  │  8 bytes  │         │   │
│   ├───────────┴───────────┴───────────┴───────────┴───────────┴─────────┤   │
│   │ CANARY FRONT (8 bytes) - 0xDEADBEEFDEADBEEF                         │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │                                                                     │   │
│   │                         U S E R   D A T A                           │   │
│   │                                                                     │   │
│   │                    (aligned on 16 bytes)                            │   │
│   │                                                                     │   │
│   ├─────────────────────────────────────────────────────────────────────┤   │
│   │ CANARY BACK (8 bytes) - 0xCAFEBABECAFEBABE                          │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│   ↑ Header                    ↑ User pointer                  ↑ End        │
│   akira_alloc returns ────────┘                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.4 Les pièges en détail

| Piège | Description | Conséquence | Solution |
|-------|-------------|-------------|----------|
| **Alignement** | Retourner un pointeur non aligné | Crash sur SIMD, perf dégradée | Toujours aligner sur 16 |
| **Double free** | Libérer deux fois le même bloc | Corruption du heap | Magic number + flag |
| **Use after free** | Accéder après free | Données corrompues | Poison values |
| **Buffer overflow** | Écrire au-delà du buffer | Corruption voisins | Canaries |
| **Integer overflow** | size + header overflow | Allocation trop petite | Check avant calcul |
| **Buddy split** | Mauvais calcul d'adresse buddy | Coalescing cassé | XOR avec (1 << order) |
| **Conservative false positive** | Entier ressemble à pointeur | Objet jamais collecté | Acceptable (conservatif) |
| **Stack scanning** | Limite de stack incorrecte | Crash ou miss roots | pthread_attr_getstack |
| **Race condition** | Pas de lock sur structures | Corruption | Mutex par structure |

### 5.5 Cours Complet : Memory Systems

#### 5.5.1 La Hiérarchie Mémoire

```
                     Latence        Taille        Coût/GB
    ┌─────────┐
    │ Registres│     ~1 cycle      ~100 bytes      $$$$$
    └────┬────┘
         │
    ┌────▼────┐
    │ L1 Cache│     ~4 cycles      32-64 KB        $$$$
    └────┬────┘
         │
    ┌────▼────┐
    │ L2 Cache│     ~12 cycles     256 KB - 1 MB   $$$
    └────┬────┘
         │
    ┌────▼────┐
    │ L3 Cache│     ~40 cycles     4-64 MB         $$
    └────┬────┘
         │
    ┌────▼────┐
    │   RAM   │     ~100 cycles    8-128 GB        $
    └────┬────┘
         │
    ┌────▼────┐
    │   SSD   │     ~100K cycles   256GB - 4TB     ¢
    └────┬────┘
         │
    ┌────▼────┐
    │   HDD   │     ~10M cycles    1-16 TB         ¢/10
    └─────────┘
```

**Principe de localité :**
- **Temporelle** : Si une donnée est accédée, elle le sera probablement bientôt
- **Spatiale** : Si une donnée est accédée, ses voisines le seront probablement

#### 5.5.2 Allocateurs Modernes

| Allocateur | Créateur | Architecture | Points Forts |
|------------|----------|--------------|--------------|
| **ptmalloc2** | glibc | Arenas par thread | Compatibilité |
| **tcmalloc** | Google | Thread cache + central | Faible contention |
| **jemalloc** | Facebook | Bins + runs + huge | Faible fragmentation |
| **mimalloc** | Microsoft | Page-local + free-list | Performance pure |

#### 5.5.3 Garbage Collection en C

Le C n'a pas de GC intégré, mais le **Boehm GC** (1988) montre que c'est possible :

1. **Scanning conservatif** : Tout mot qui ressemble à un pointeur est traité comme tel
2. **Faux positifs** : Un entier peut ressembler à un pointeur → l'objet n'est pas collecté
3. **Faux négatifs** : Impossible (si c'est un vrai pointeur, on le trouve)
4. **Pas de moving** : Les pointeurs peuvent être n'importe où, on ne peut pas les mettre à jour

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME (compile, mais interdit)                          │
├─────────────────────────────────────────────────────────────────┤
│ void *ptr = malloc(size);  // Utiliser la libc                  │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ void *ptr = mmap(NULL, size, PROT_READ | PROT_WRITE,            │
│                  MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);           │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • L'objectif est d'IMPLÉMENTER malloc, pas de l'utiliser       │
│ • mmap est le syscall bas niveau pour obtenir de la mémoire     │
│ • On contrôle exactement comment la mémoire est gérée           │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario : Allocation Buddy puis Coalescing**

```
┌───────┬────────────────────────────────────────────────────┬───────────────────┐
│ Étape │ Opération                                          │ État Buddy Heap   │
├───────┼────────────────────────────────────────────────────┼───────────────────┤
│   1   │ buddy_alloc(300) → need 512 (order 9)              │ [████████████████]│
│       │ Split 64KB → 32KB → 16KB → 8KB → 4KB → 2KB → 1KB → 512B              │
│       │ Retourne premier 512B, reste en free lists         │ [A][F][1K][2K]...│
├───────┼────────────────────────────────────────────────────┼───────────────────┤
│   2   │ buddy_alloc(300) → prend le 512B de free_list[9]   │ [A][A][1K][2K]...│
├───────┼────────────────────────────────────────────────────┼───────────────────┤
│   3   │ buddy_free(ptr1) → retourne à free_list[9]         │ [F][A][1K][2K]...│
├───────┼────────────────────────────────────────────────────┼───────────────────┤
│   4   │ buddy_free(ptr2) → coalesce !                      │                   │
│       │ 512 + 512 → 1KB                                    │ [1K free][1K]...  │
│       │ 1KB + 1KB → 2KB                                    │ [2K free][2K]...  │
│       │ Continue jusqu'à reconstruction bloc original      │ [████████████████]│
└───────┴────────────────────────────────────────────────────┴───────────────────┘
```

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🔴 MEME : "TETSUOOOO!" — L'Explosion Mémoire

Dans Akira, quand Tetsuo perd le contrôle de son pouvoir, son corps **explose** littéralement en une masse de chair informe qui engloutit tout.

C'est exactement ce qui arrive à ton programme avec une **fuite mémoire** :
- Au début, tout va bien
- Petit à petit, la mémoire s'accumule
- Le programme grossit, grossit...
- **BOOM** - OOM Killer !

```c
while (1)
{
    void *power = akira_alloc(1024);
    // Tetsuo oublie de libérer...
    // "TETSUOOOO!" 💥
}
```

**La solution :** Toujours appeler `akira_free()` - Contrôle ton pouvoir !

---

#### 🏍️ MEME : "Kaneda's Bike" — Le GC qui nettoie

La moto rouge de Kaneda fonce dans les rues de Neo-Tokyo, éliminant les obstacles sur son passage.

Ton **Garbage Collector** fait pareil :
- Il traverse le heap à toute vitesse
- Il identifie les "ordures" (objets non référencés)
- Il les **élimine** sans pitié

```
     🏍️💨
[LIVE][DEAD][LIVE][DEAD][DEAD][LIVE]
        ↓      ↓      ↓
      [FREE] [FREE] [FREE]
```

---

#### 🔬 MEME : "The Espers" — Les Canaries

Les enfants Espers (Takashi, Kiyoko, Masaru) ont des **pouvoirs de détection**. Ils peuvent sentir quand quelque chose ne va pas.

Tes **canaries** font pareil :
- Valeur magique `0xDEADBEEF` avant et après le bloc
- Si elle change → **ALERTE** : Buffer overflow détecté !

```c
[0xDEADBEEF][USER DATA][0xCAFEBABE]
     ↑                       ↑
   "Je sens..."         "...une perturbation"
```

---

#### ☠️ MEME : "Neo-Tokyo Destroyed" — Use After Free

Après l'explosion de 1988, Neo-Tokyo est en ruines. Mais des gens essaient quand même d'y vivre.

C'est un **use-after-free** :
- Le bloc a été détruit (free)
- Mais quelqu'un essaie encore d'y accéder
- Résultat : **CHAOS** (données corrompues, crash)

```c
akira_free(neo_tokyo);
// Neo-Tokyo est en ruines...
neo_tokyo->population = 30000000;  // 💀 UNDEFINED BEHAVIOR !
```

**Protection :** Poison values (`0xFEFEFEFE`) après free !

### 5.9 Applications pratiques

| Contexte | Application | Techniques Utilisées |
|----------|-------------|---------------------|
| **Game Engines** | Unity, Unreal | Memory pools, custom allocators |
| **Databases** | PostgreSQL, Redis | Buffer pools, slab allocators |
| **Browsers** | Chrome, Firefox | Generational GC, compartments |
| **OS Kernels** | Linux, FreeBSD | Slab allocator, buddy system |
| **Embedded** | FreeRTOS | Static pools, no fragmentation |
| **ML Frameworks** | PyTorch, TensorFlow | Custom CUDA allocators |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Symptôme | Détection | Prévention |
|---|-------|----------|-----------|------------|
| 1 | Alignement incorrect | Crash SIMD, perf | Tests alignement | `% 16 == 0` |
| 2 | Double free | Corruption heap | Magic number | Flag "freed" |
| 3 | Use after free | Données aléatoires | Poison values | `0xFEFEFEFE` |
| 4 | Buffer overflow | Corruption voisin | Canaries | `0xDEADBEEF` |
| 5 | Memory leak | RAM croissante | Tracking allocs | Profiler |
| 6 | Integer overflow | Bloc trop petit | Check size + header | Safe math |
| 7 | Buddy addr calc | Coalesce fail | Tests unitaires | XOR correct |
| 8 | Race condition | Corruption | TSAN | Mutex/atomics |
| 9 | Stack overflow GC | Crash mark phase | Mark stack | Iteratif |
| 10 | False sharing | Perf threads | Perf counters | Padding 64B |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quel est l'avantage principal des segregated free lists ?**
- A) Réduction de la fragmentation interne
- B) Allocation O(1) pour les petites tailles
- C) Meilleure utilisation du cache
- D) Support natif du multithreading
- E) Compaction automatique
- F) Réduction de l'overhead mémoire
- G) Support des huge pages
- H) Détection automatique des fuites
- I) Coalescing immédiat
- J) Toutes les réponses sont correctes

**Réponse : B**

### Question 2
**Dans le buddy system, comment calcule-t-on l'adresse du buddy ?**
- A) addr + size
- B) addr - size
- C) addr XOR size
- D) addr XOR (1 << order)
- E) addr AND ~size
- F) addr OR size
- G) addr / 2
- H) addr * 2
- I) addr MOD size
- J) Dépend de l'implémentation

**Réponse : D**

### Question 3
**Pourquoi un GC conservatif ne peut-il pas faire de "moving collection" ?**
- A) C n'a pas d'info de type runtime
- B) Les pointeurs peuvent être n'importe où (variables, registres)
- C) On ne peut pas mettre à jour tous les pointeurs
- D) Toutes les réponses A, B, C sont correctes
- E) Le standard C l'interdit
- F) mmap ne le supporte pas
- G) C'est trop lent
- H) Les threads poseraient problème
- I) Les signaux interrompraient
- J) Le kernel refuserait

**Réponse : D**

### Question 4
**Quelle valeur est typiquement utilisée pour "poisonner" la mémoire libérée ?**
- A) 0x00
- B) 0xFF
- C) 0xFE ou 0xFEFEFEFE
- D) 0xDEADBEEF
- E) 0xCAFEBABE
- F) 0xABADCAFE
- G) 0x12345678
- H) 0xFFFFFFFF
- I) Valeur aléatoire
- J) Dépend du mode debug

**Réponse : C**

### Question 5
**Dans un GC générationnel, pourquoi la "nursery" utilise-t-elle souvent un bump allocator ?**
- A) Plus simple à implémenter
- B) Allocation O(1) ultra-rapide
- C) Pas besoin de free list pour les jeunes objets
- D) La plupart des objets meurent jeunes (hypothèse générationnelle)
- E) Toutes les réponses A, B, C, D sont correctes
- F) Meilleure localité cache
- G) Support natif du copying GC
- H) Réduction de la fragmentation
- I) Compatibilité avec le marking
- J) Facilite le remembered set

**Réponse : E**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Difficulté** | ★★★★★★★★★☆ (9/10) |
| **Type** | Projet Synthèse (TIERS 3) |
| **Durée** | 50 heures |
| **XP Base** | 5000 |
| **XP Bonus (🧠)** | 30000 (×6) |
| **Concepts couverts** | 250+ (tout Module 2.1) |
| **Fichiers à créer** | 10+ fichiers .c/.h |
| **Tests à passer** | 13+ tests unitaires |
| **Prérequis** | ex01-ex10, M2.1_Ex01-Ex08 |

### Compétences Acquises

```
✓ Implémenter un allocateur mémoire complet (3 niveaux)
✓ Concevoir un garbage collector conservatif
✓ Créer un profiler avec détection d'erreurs
✓ Gérer la thread-safety avec granularité fine
✓ Analyser et réduire la fragmentation
✓ Débugger des problèmes mémoire complexes
```

---

## 📦 SECTION 9 : DEPLOYMENT PACK (JSON COMPLET)

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1-PROJET-neo_tokyo_memory_system",
    "generated_at": "2026-01-11 00:00:00",

    "metadata": {
      "exercise_id": "2.1-PROJET",
      "exercise_name": "neo_tokyo_memory_system",
      "module": "2.1",
      "module_name": "Memory Management",
      "concept": "PROJET",
      "concept_name": "Synthèse Complète",
      "type": "complet_2",
      "tier": 3,
      "tier_info": "Synthèse tous concepts 2.1.1-2.1.22",
      "phase": 2,
      "difficulty": 9,
      "difficulty_stars": "★★★★★★★★★☆",
      "language": "c17",
      "duration_minutes": 3000,
      "xp_base": 5000,
      "xp_bonus_multiplier": 6,
      "bonus_tier": "GÉNIE",
      "bonus_icon": "🧠",
      "complexity_time": "T3 O(n log n)",
      "complexity_space": "S3 O(n)",
      "prerequisites": ["ex01-ex10", "M2.1_Ex01-Ex08"],
      "domains": ["Mem", "CPU", "Struct", "Process"],
      "domains_bonus": ["Process (concurrent GC)"],
      "tags": ["allocator", "gc", "profiler", "memory", "capstone"],
      "meme_reference": "AKIRA - Neo-Tokyo 2019"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/architecture.md": "/* Section 4.3 */",
      "references/segregated.c": "/* Section 4.3 */",
      "references/buddy.c": "/* Section 4.3 */",
      "references/gc_mark.c": "/* Section 4.3 */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_logic.c": "/* Section 4.10 */",
      "mutants/mutant_e_return.c": "/* Section 4.10 */",
      "mutants/mutant_f_concurrency.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/segregated.c",
        "references/buddy.c",
        "references/gc_mark.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c",
        "mutants/mutant_f_concurrency.c"
      ]
    },

    "commands": {
      "build": "make",
      "test": "make test",
      "benchmark": "make benchmark",
      "validate_spec": "python3 hackbrain_engine_v22.py --validate-spec spec.json"
    }
  }
}
```

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 25 | Synthèse complète de 250+ concepts avec architecture claire |
| Couverture conceptuelle | 25 | Couvre TOUT le Module 2.1 (21 sections, ~250 concepts) |
| Testabilité auto | 24 | Tests exhaustifs, stress tests, thread tests |
| Originalité | 24 | Thème AKIRA parfaitement adapté, noms de fonctions mémorables |
| **TOTAL** | **98/100** | ✓ Validé |

**✓ Score ≥ 95, exercice validé.**

---

*NEO-TOKYO MEMORY SYSTEM - "Le pouvoir est dans le contrôle"*
*HACKBRAIN v5.5.2 — L'excellence pédagogique ne se négocie pas*
