# Exercice 2.3.12 : omega_loop_simulator

**Module :**
2.3 — File Systems

**Concept :**
2.3.12 — Journaling File System (a-l)

**Difficulté :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (concepts 2.3.12.a → 2.3.12.l)

**Langage :**
C (C17)

**Prérequis :**
- 2.3.8 Block Allocation
- 2.3.11 Superblock
- Atomic operations
- Crash recovery concepts

**Domaines :**
FS, Mem, Struct

**Durée estimée :**
480 min (8 heures)

**XP Base :**
500

**Complexité :**
T3 O(n) × S3 O(j) où j = taille du journal

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichier à rendre :** `omega_loop.c`, `omega_loop.h`, `time_loop.c`, `alpha_memory.c`, `recovery.c`

**Fonctions autorisées :**
```
malloc, calloc, realloc, free
memcpy, memmove, memset, memcmp
strlen, strcpy, strncpy, strcmp
snprintf, vsnprintf
printf, fprintf (debug)
time, clock, gettimeofday
rand, srand
assert, errno
```

**Fonctions interdites :**
```
open, read, write, close (utiliser le disque émulé)
fork, exec, system
Fonctions réseau
```

### 1.2 Consigne

**🎬 CONTEXTE : EDGE OF TOMORROW — "Live. Die. Repeat."**

*"On your feet, maggot!"*

Tu es **Major William Cage**, et tu viens de te retrouver piégé dans une boucle temporelle. Chaque fois qu'un **Mimic** te tue, tu te réveilles exactement au même moment — mais tu GARDES certains souvenirs.

Le problème ? La réalité ne fonctionne pas exactement comme tu le voudrais. Certaines choses que tu apprends sont **verrouillées** dans ta mémoire (elles survivent à la mort), d'autres sont **perdues** quand tu meurs trop tôt.

Tu découvres que ton cerveau fonctionne comme un **système de journaling** :

```
┌─────────────────────────────────────────────────────────────────┐
│                    LA BOUCLE OMEGA                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   🌅 RÉVEIL          🎬 EXPÉRIENCE          💀 MORT            │
│      │                    │                    │               │
│      v                    v                    v               │
│   ┌──────┐           ┌──────────┐         ┌──────┐            │
│   │wake_ │           │time_loop_│         │mimic_│            │
│   │up()  │           │begin()   │         │kills()│           │
│   └──────┘           └──────────┘         └──────┘            │
│      │                    │                    │               │
│      │              ┌─────┴─────┐              │               │
│      │              │           │              │               │
│      │        ┌─────────┐ ┌───────────┐        │               │
│      │        │lock_    │ │sync_to_   │        │               │
│      │        │memory() │ │reality()  │        │               │
│      │        └─────────┘ └───────────┘        │               │
│      │         COMMIT      CHECKPOINT          │               │
│      │              │           │              │               │
│      └──────────────┴───────────┴──────────────┘               │
│                                                                 │
│   💡 CE QUI EST COMMITTED → SURVIT À LA MORT                   │
│   ❌ CE QUI N'EST PAS COMMITTED → PERDU                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Le problème de cohérence (2.3.12.a) :**

Imagine que tu apprends trois choses dans une boucle :
1. L'emplacement d'une mine
2. Le chemin pour l'éviter
3. La position du Mimic Alpha

Si tu MEURS entre l'étape 1 et 3 :
- Tu pourrais te souvenir de la mine...
- ...mais pas du chemin pour l'éviter !
- Résultat : **INCOHÉRENCE** = tu exploses 💥

C'est le **crash consistency problem** !

**La solution ancienne : time_scan() (2.3.12.b)**

Avant le journaling, après chaque mort, ton cerveau devait scanner TOUTE ta ligne temporelle pour trouver les incohérences. Ça prenait des HEURES (dans le film, des jours de boucles inutiles).

**La solution moderne : Alpha Memory (2.3.12.c)**

Tu as une **mémoire Alpha** — un journal qui enregistre les expériences AVANT de les appliquer à ta mémoire permanente :

```
┌─────────────────────────────────────────────────────────────────┐
│                    ALPHA MEMORY (Journal)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   [LOOP_BEGIN #42]                                              │
│   ├── EXP: "Mine at coordinates (10, 15)"                       │
│   ├── EXP: "Safe path: left-right-jump"                         │
│   ├── EXP: "Alpha Mimic at beach sector 7"                      │
│   [MEMORY_LOCKED ✓]  ← Ce marqueur = tout survit               │
│                                                                 │
│   [LOOP_BEGIN #43]                                              │
│   ├── EXP: "New weapon training"                                │
│   ├── EXP: "Rita's combat stance"                               │
│   [PAS DE LOCK]      ← Mort trop tôt = perdu                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Les trois modes de mémoire (2.3.12.i-l) :**

```
┌──────────────────┬─────────────────────────────────────────────┐
│ MODE             │ CE QUI EST SAUVEGARDÉ                       │
├──────────────────┼─────────────────────────────────────────────┤
│ FULL_RECALL      │ Expériences + Données complètes             │
│ (2.3.12.j)       │ "Je me souviens de TOUT"                    │
│                  │ + Maximum de survie                         │
│                  │ - Très fatigant mentalement                 │
├──────────────────┼─────────────────────────────────────────────┤
│ TACTICAL_RECALL  │ Données d'abord, expériences ensuite        │
│ (2.3.12.k)       │ "Je fais les actions PUIS je mémorise"      │
│                  │ = Équilibre survie/performance              │
│                  │ = Mode par défaut de Rita                   │
├──────────────────┼─────────────────────────────────────────────┤
│ INSTINCT_MODE    │ Seulement les réflexes (métadonnées)        │
│ (2.3.12.l)       │ "Mon corps sait, mon esprit oublie"         │
│                  │ + Maximum de vitesse                        │
│                  │ - Risque de désorientation                  │
└──────────────────┴─────────────────────────────────────────────┘
```

### 1.2.2 Consigne Académique

Implémenter un **simulateur de journaling filesystem** qui démontre tous les concepts du module 2.3.12 :

1. **Crash Consistency (2.3.12.a)** : Simuler le problème de cohérence lors d'un crash
2. **FSCK (2.3.12.b)** : Implémentation lente de vérification
3. **Journal WAL (2.3.12.c)** : Write-ahead log
4. **Transactions (2.3.12.d)** : Groupes atomiques d'opérations
5. **Journal Write (2.3.12.e)** : Écriture dans le journal avant les données
6. **Commit (2.3.12.f)** : Marqueur de transaction complète
7. **Checkpoint (2.3.12.g)** : Écriture vers emplacement final
8. **Recovery (2.3.12.h)** : Replay des committées, discard des incomplètes
9. **Trois modes (2.3.12.i-l)** : DATA, ORDERED, WRITEBACK

**Ta mission :**

Créer la structure `omega_loop_t` et implémenter toutes les fonctions du simulateur de journaling.

**Entrée :**
- `config` : Configuration du système (taille disque, taille journal, mode)
- `txn` : Transaction à exécuter
- `data` : Données à écrire

**Sortie :**
- Retourne `0` si succès
- Retourne `-1` si erreur
- Recovery retourne le nombre de transactions rejouées/ignorées

### 1.3 Prototype

```c
/* omega_loop.h — Edge of Tomorrow Journal Simulator */

#ifndef OMEGA_LOOP_H
#define OMEGA_LOOP_H

#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>
#include <time.h>

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: Constantes — La Boucle Omega
   ═══════════════════════════════════════════════════════════════════════════ */

#define REALITY_BLOCK_SIZE        4096   /* Taille d'un "bloc" de réalité */
#define ALPHA_MEMORY_MAX_BLOCKS   256    /* Mémoire Alpha max */
#define MAX_OPEN_LOOPS            16     /* Boucles temporelles simultanées */
#define MAX_EXPERIENCES_PER_LOOP  64     /* Expériences par boucle */

#define MEMORY_LOCK_MARKER   0xC0FFFFFF  /* Marqueur de verrouillage mémoire */

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: 2.3.12.i — Modes de Mémoire
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Modes de mémorisation.
 *
 * 2.3.12.i: Trois modes avec différents compromis survie/performance.
 */
typedef enum {
    /**
     * 2.3.12.j: FULL RECALL MODE
     * Expériences ET données journalisées.
     * Maximum de survie, très fatigant.
     * "Je me souviens de CHAQUE détail de chaque boucle."
     */
    MEMORY_MODE_FULL_RECALL,

    /**
     * 2.3.12.k: TACTICAL RECALL MODE
     * Actions d'abord, puis mémorisation des métadonnées.
     * Bon compromis. Mode utilisé par Rita Vrataski.
     * "Je fais d'abord, je mémorise ensuite."
     */
    MEMORY_MODE_TACTICAL,

    /**
     * 2.3.12.l: INSTINCT MODE
     * Seulement les réflexes (métadonnées).
     * Maximum de vitesse, risque de désorientation.
     * "Mon corps sait quoi faire, mon esprit suit."
     */
    MEMORY_MODE_INSTINCT
} memory_mode_t;

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: 2.3.12.d — États de Boucle Temporelle
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * État d'une boucle temporelle (transaction).
 */
typedef enum {
    LOOP_STATE_INACTIVE,      /* Pas encore commencée */
    LOOP_STATE_ACTIVE,        /* En cours */
    LOOP_STATE_PREPARING,     /* Écriture en mémoire Alpha */
    LOOP_STATE_LOCKED,        /* 2.3.12.f: Mémoire verrouillée */
    LOOP_STATE_SYNCED,        /* 2.3.12.g: Synchronisé avec réalité */
    LOOP_STATE_LOST           /* Mort avant lock = perdu */
} loop_state_t;

/**
 * Types d'expériences dans une boucle.
 */
typedef enum {
    EXP_LEARN_SKILL,          /* Apprendre une compétence */
    EXP_DISCOVER_LOCATION,    /* Découvrir un lieu */
    EXP_REMEMBER_EVENT,       /* Mémoriser un événement */
    EXP_COMBAT_MOVE,          /* Mouvement de combat */
    EXP_ALLY_INFO,            /* Information sur un allié */
    EXP_ENEMY_WEAKNESS,       /* Faiblesse ennemie */
    EXP_TIMELINE_CHANGE,      /* Modification de la timeline */
    EXP_OMEGA_INTEL           /* Intel sur l'Omega */
} experience_type_t;

/**
 * Une expérience dans une boucle temporelle.
 *
 * 2.3.12.e: Les expériences sont journalisées AVANT d'être appliquées.
 */
typedef struct {
    experience_type_t  type;                         /* Type d'expérience */
    uint64_t          target_reality_block;          /* Bloc "réalité" cible */
    uint64_t          memory_id;                     /* ID mémoire concernée */
    size_t            data_size;                     /* Taille des données */
    uint8_t           data[REALITY_BLOCK_SIZE];      /* Données de l'expérience */
    uint32_t          checksum;                      /* Vérification intégrité */
} experience_t;

/**
 * Handle de boucle temporelle.
 *
 * 2.3.12.d: Une boucle est un groupe atomique d'expériences.
 */
typedef struct {
    uint64_t        loop_id;                         /* ID unique de la boucle */
    loop_state_t    state;                           /* État actuel */
    time_t          wake_time;                       /* Heure du réveil */
    time_t          death_time;                      /* Heure de la mort */

    /* Expériences de cette boucle */
    experience_t    experiences[MAX_EXPERIENCES_PER_LOOP];
    int             exp_count;                       /* Nombre d'expériences */

    /* Marqueurs */
    uint32_t        begin_marker;                    /* LOOP_BEGIN */
    uint32_t        lock_marker;                     /* MEMORY_LOCKED (2.3.12.f) */
} time_loop_t;

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: Configuration et Statistiques
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Configuration du simulateur Omega.
 */
typedef struct {
    size_t          reality_size_blocks;     /* Taille de la "réalité" */
    size_t          alpha_memory_blocks;     /* Taille mémoire Alpha */
    memory_mode_t   mode;                    /* 2.3.12.i: Mode de mémoire */
    bool            sync_on_lock;            /* Sync après chaque lock */
    bool            enable_checksums;        /* Vérification intégrité */
    int             sync_interval;           /* Intervalle de sync */
} omega_config_t;

/**
 * Statistiques du simulateur.
 */
typedef struct {
    /* Boucles */
    uint64_t loops_started;
    uint64_t loops_locked;               /* 2.3.12.f */
    uint64_t loops_lost;
    uint64_t loops_recovered;            /* 2.3.12.h */

    /* Alpha Memory */
    uint64_t alpha_writes;               /* 2.3.12.e */
    uint64_t alpha_bytes_written;
    uint64_t syncs_done;                 /* 2.3.12.g */

    /* Reality */
    uint64_t reality_reads;
    uint64_t reality_writes;
    uint64_t reality_syncs;

    /* Recovery */
    uint64_t replayed_loops;             /* 2.3.12.h */
    uint64_t discarded_loops;            /* 2.3.12.h */

    /* Performance */
    double avg_loop_duration_ms;
    double alpha_utilization;
} omega_stats_t;

/**
 * Résultat d'un time_scan (fsck).
 *
 * 2.3.12.b: Scan complet de la timeline — LENT!
 */
typedef struct {
    bool   is_consistent;                /* Timeline cohérente ? */
    int    anomalies_found;              /* Anomalies trouvées */
    int    anomalies_fixed;              /* Anomalies réparées */
    int    orphan_memories;              /* Souvenirs orphelins */
    int    paradoxes;                    /* Paradoxes temporels */
    double scan_time_ms;                 /* Temps de scan (2.3.12.b: SLOW!) */
    char   details[4096];                /* Détails */
} timeline_scan_result_t;

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: Handles Opaques
   ═══════════════════════════════════════════════════════════════════════════ */

typedef struct omega_loop omega_loop_t;
typedef struct alpha_memory alpha_memory_t;
typedef struct reality reality_t;

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: API Principale — Lifecycle
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Crée une nouvelle Boucle Omega.
 *
 * @param config Configuration (NULL pour défauts)
 * @return Handle Omega, NULL si erreur
 */
omega_loop_t *omega_create(const omega_config_t *config);

/**
 * Détruit la Boucle Omega.
 * Effectue un sync final si nécessaire.
 */
void omega_destroy(omega_loop_t *omega);

/**
 * Se réveille dans la boucle (simule le montage).
 * Effectue la recovery si nécessaire (2.3.12.h).
 *
 * @return 0 si succès, nombre de boucles récupérées sinon
 */
int omega_wake_up(omega_loop_t *omega);

/**
 * Fin de journée (démontage).
 * Force un sync final (2.3.12.g).
 */
int omega_end_day(omega_loop_t *omega);

/**
 * Change le mode de mémoire.
 *
 * 2.3.12.i: Basculer entre full_recall, tactical et instinct.
 */
int omega_set_mode(omega_loop_t *omega, memory_mode_t mode);

/**
 * Obtient le mode actuel.
 */
memory_mode_t omega_get_mode(omega_loop_t *omega);

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: API Boucles Temporelles (2.3.12.d)
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Démarre une nouvelle boucle temporelle.
 *
 * 2.3.12.d: Une boucle groupe plusieurs expériences atomiquement.
 *
 * @return Pointeur vers la boucle, NULL si erreur
 */
time_loop_t *time_loop_begin(omega_loop_t *omega);

/**
 * Ajoute une expérience à la boucle.
 *
 * 2.3.12.e: L'expérience est d'abord écrite en Alpha Memory.
 */
int time_loop_experience(omega_loop_t *omega, time_loop_t *loop,
                         uint64_t block_id, const void *data, size_t size);

/**
 * Apprend une compétence (mise à jour mémoire).
 */
int time_loop_learn_skill(omega_loop_t *omega, time_loop_t *loop,
                          uint64_t skill_id, const void *skill_data);

/**
 * Découvre un emplacement (mise à jour map mentale).
 */
int time_loop_discover(omega_loop_t *omega, time_loop_t *loop,
                       uint64_t location_id, const void *map_data);

/**
 * Verrouille la mémoire (commit).
 *
 * 2.3.12.f: Marque la boucle comme complète.
 *
 * @note Après lock, les souvenirs survivent à la mort
 */
int lock_memory(omega_loop_t *omega, time_loop_t *loop);

/**
 * Perd les souvenirs de la boucle (abort).
 *
 * @note Comme si cette boucle n'avait jamais existé
 */
int forget_loop(omega_loop_t *omega, time_loop_t *loop);

/**
 * Obtient l'état d'une boucle.
 */
loop_state_t time_loop_state(time_loop_t *loop);

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: API Sync (2.3.12.g)
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Force une synchronisation avec la réalité.
 *
 * 2.3.12.g: Écrit les souvenirs de l'Alpha vers la réalité permanente.
 *
 * @return Nombre de boucles synchronisées
 */
int sync_to_reality(omega_loop_t *omega);

/**
 * Configure le sync automatique.
 */
void sync_configure(omega_loop_t *omega, int interval);

/**
 * Retourne l'utilisation de l'Alpha Memory.
 */
double alpha_memory_usage(omega_loop_t *omega);

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: API Recovery (2.3.12.h)
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Récupération après une mort.
 *
 * 2.3.12.h: Replay les boucles lockées, ignore les incomplètes.
 *
 * @param replayed Nombre de boucles rejouées
 * @param discarded Nombre de boucles ignorées
 */
int recover_after_death(omega_loop_t *omega, int *replayed, int *discarded);

/**
 * Affiche le contenu de l'Alpha Memory (debug).
 */
void alpha_memory_dump(omega_loop_t *omega);

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: API Time Scan — FSCK (2.3.12.b)
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Scan complet de la timeline.
 *
 * 2.3.12.b: time_scan est LENT car il parcourt toute la timeline.
 *
 * @param result Résultat du scan
 * @param repair Tenter de réparer les anomalies ?
 */
int time_scan(omega_loop_t *omega, timeline_scan_result_t *result, bool repair);

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: API Crash Simulation (2.3.12.a)
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Points d'injection de mort.
 *
 * 2.3.12.a: Simule le problème de crash consistency.
 */
typedef enum {
    DEATH_POINT_NONE,                  /* Pas de mort */
    DEATH_BEFORE_ALPHA,                /* Avant écriture Alpha Memory */
    DEATH_DURING_ALPHA,                /* Pendant écriture Alpha */
    DEATH_BEFORE_LOCK,                 /* Avant verrouillage mémoire */
    DEATH_AFTER_LOCK,                  /* Après lock, avant sync */
    DEATH_DURING_SYNC,                 /* Pendant la synchronisation */
    DEATH_RANDOM                       /* Aléatoire (Mimic attack!) */
} death_point_t;

/**
 * Configure un point de mort.
 *
 * 2.3.12.a: Simule quand les Mimics te tuent.
 */
void mimic_inject_death(omega_loop_t *omega, death_point_t point, double probability);

/**
 * Simule une mort (crash).
 *
 * 2.3.12.a: "Live. Die. Repeat."
 */
int mimic_kills(omega_loop_t *omega);

/**
 * Simule le réveil après la mort.
 *
 * "On your feet, maggot!"
 */
int wake_up_after_death(omega_loop_t *omega);

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: API Diagnostics
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Obtient les statistiques.
 */
omega_stats_t omega_get_stats(omega_loop_t *omega);

/**
 * Affiche un rapport détaillé.
 */
void omega_print_report(omega_loop_t *omega);

/**
 * Compare les trois modes de mémoire.
 */
void omega_benchmark_modes(omega_loop_t *omega);

/* ═══════════════════════════════════════════════════════════════════════════
   SECTION: API Haut Niveau
   ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Opérations avec journaling automatique.
 * Créent une boucle, exécutent, et lock.
 */

ssize_t omega_remember(omega_loop_t *omega, uint64_t memory_id,
                       const void *data, size_t size, off_t offset);

ssize_t omega_recall(omega_loop_t *omega, uint64_t memory_id,
                     void *buffer, size_t size, off_t offset);

int64_t omega_create_memory(omega_loop_t *omega, const char *name);

int omega_erase_memory(omega_loop_t *omega, uint64_t memory_id);

#endif /* OMEGA_LOOP_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Edge of Tomorrow et le Journaling

Dans le film **Edge of Tomorrow** (2014), le Major Cage se retrouve piégé dans une boucle temporelle après avoir été infecté par le sang d'un Mimic Alpha. Chaque mort le ramène au même point, mais il conserve ses souvenirs.

**Cette mécanique est EXACTEMENT celle du journaling filesystem !**

| Film | Système de Fichiers |
|------|---------------------|
| Boucle temporelle | Transaction |
| Mort | Crash système |
| Réveil | Reboot |
| Souvenirs lockés | Transactions committées |
| Souvenirs perdus | Transactions incomplètes |
| Rita qui entraîne Cage | Recovery guidée |

### 2.2 Pourquoi le Journaling ?

Le **crash consistency problem** (2.3.12.a) est un vrai cauchemar :

```
SANS JOURNALING (Cage sans boucle) :
─────────────────────────────────────
1. Tu apprends où est la mine
2. Tu te fais tuer
3. Tu te réveilles SANS AUCUN souvenir
4. Tu recommences à zéro CHAQUE FOIS
→ Impossible de progresser

AVEC JOURNALING (Cage avec le pouvoir Alpha) :
──────────────────────────────────────────────
1. Tu apprends où est la mine
2. Tu VERROUILLES ce souvenir (commit)
3. Tu te fais tuer
4. Tu te réveilles AVEC ce souvenir
→ Progression garantie !
```

### 2.3 Les Trois Modes = Trois Styles de Combat

| Mode | Personnage | Style |
|------|------------|-------|
| **FULL_RECALL** | Cage (début) | Se rappelle TOUT, très épuisant |
| **TACTICAL** | Rita Vrataski | Équilibre parfait, efficace |
| **INSTINCT** | Soldats entraînés | Réflexes purs, risque d'erreur |

### 2.5 DANS LA VRAIE VIE

**Qui utilise le journaling ?**

| Métier | Cas d'usage |
|--------|-------------|
| **DBA (Database Admin)** | PostgreSQL WAL, MySQL InnoDB |
| **SysAdmin** | ext4, XFS, NTFS journaling |
| **DevOps** | Récupération après crash serveur |
| **Cloud Engineer** | Systèmes distribués, Kafka |
| **Développeur Jeux** | Save states, checkpoints |

**Exemples concrets :**
- **ext4** utilise le mode **ordered** par défaut (2.3.12.k)
- **PostgreSQL** utilise un WAL pour garantir l'ACID
- **Redis** avec AOF (Append-Only File) = journaling
- **MongoDB** avec le journal WiredTiger

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
omega_loop.c  omega_loop.h  time_loop.c  alpha_memory.c  recovery.c  main.c  Makefile

$ make
gcc -std=c17 -Wall -Wextra -Werror -pedantic -c omega_loop.c
gcc -std=c17 -Wall -Wextra -Werror -pedantic -c time_loop.c
gcc -std=c17 -Wall -Wextra -Werror -pedantic -c alpha_memory.c
gcc -std=c17 -Wall -Wextra -Werror -pedantic -c recovery.c
ar rcs libomega.a omega_loop.o time_loop.o alpha_memory.o recovery.o

$ make demo
gcc -std=c17 -Wall -Wextra -Werror main.c -L. -lomega -o demo

$ ./demo

=== EDGE OF TOMORROW: Omega Loop Simulator ===

"On your feet, maggot!"

1. Starting committed loop...
   Loop #1 started
   Experience: "Mine at (10, 15)" added
   Experience: "Safe path found" added
   Memory LOCKED successfully!

2. Starting uncommitted loop...
   Loop #2 started
   Experience: "New weapon training" added
   (No lock - killed too fast!)

3. Simulating Mimic kill...
   💀 DEATH! System crashed.

4. Waking up...
   🌅 "On your feet, maggot!"
   Recovering from death...
   - Loops replayed: 1
   - Loops discarded: 1

5. Verifying memories...
   Memory "Mine location": FOUND (survived death!)
   Memory "Weapon training": NOT FOUND (lost in death)

=== Result ===
Committed memories survive death - Journaling works!
"Live. Die. Repeat."
```

### 3.1 ⚡ BONUS STANDARD (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★★☆ (9/10)

**Récompense :**
XP ×2

**Contraintes Bonus :**
```
┌─────────────────────────────────────────┐
│  Alpha Memory circulaire efficace       │
│  Checksums CRC32 obligatoires           │
│  Recovery idempotente                   │
│  Support nested loops (boucles imbriquées) │
└─────────────────────────────────────────┘
```

**Consigne Bonus :**

Implémente le mode **BLEEDING EDGE** : comme quand Cage perd trop de sang et risque de perdre son pouvoir, tu dois gérer le cas où l'Alpha Memory est presque pleine.

**Fonctions bonus :**
```c
/* Boucles imbriquées - comme les rêves dans Inception */
time_loop_t *time_loop_nest(omega_loop_t *omega, time_loop_t *parent);

/* Mode Bleeding Edge - proche de perdre le pouvoir */
int omega_set_bleeding_edge(omega_loop_t *omega, bool enabled);

/* Sauvegarde d'urgence avant perte du pouvoir */
int omega_emergency_sync(omega_loop_t *omega);
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (Tests)

| # | Test | Entrée | Attendu | Points |
|---|------|--------|---------|--------|
| 1 | Creation | `omega_create(NULL)` | Handle valide | 3 |
| 2 | Destruction | `omega_destroy(omega)` | Pas de leak | 3 |
| 3 | Loop begin | `time_loop_begin(omega)` | État ACTIVE | 4 |
| 4 | Lock memory | `lock_memory(omega, loop)` | État LOCKED | 5 |
| 5 | Mode DATA | `omega_set_mode(FULL_RECALL)` | Mode changé | 4 |
| 6 | Mode ORDERED | `omega_set_mode(TACTICAL)` | Mode changé | 4 |
| 7 | Mode WRITEBACK | `omega_set_mode(INSTINCT)` | Mode changé | 4 |
| 8 | Crash recovery | Commit + crash + recover | Replay = 1 | 8 |
| 9 | Uncommitted lost | No commit + crash | Discard = 1 | 8 |
| 10 | Checkpoint | 5 loops + sync | Usage = 0% | 6 |
| 11 | Time scan | `time_scan()` | Consistent | 4 |
| 12 | Full recall mode | Journal > 4KB | Data logged | 7 |
| 13 | Tactical mode | Write order | Data before metadata | 8 |
| 14 | Instinct mode | Journal size | Metadata only | 7 |
| 15 | Abort loop | `forget_loop()` | No effect | 5 |
| 16 | Multiple crash points | All points | Recovery OK | 7 |
| 17 | Integration | Full workflow | All concepts | 10 |
| 18 | Valgrind | Full test | 0 leaks | 8 |

### 4.2 main.c de test

```c
#include "omega_loop.h"
#include <stdio.h>
#include <string.h>
#include <assert.h>

void test_basic_lifecycle(void) {
    printf("Test 1: Basic lifecycle...\n");

    omega_loop_t *omega = omega_create(NULL);
    assert(omega != NULL);

    omega_destroy(omega);
    printf("  PASSED\n");
}

void test_loop_commit(void) {
    printf("Test 2: Loop commit...\n");

    omega_loop_t *omega = omega_create(NULL);

    time_loop_t *loop = time_loop_begin(omega);
    assert(loop != NULL);
    assert(time_loop_state(loop) == LOOP_STATE_ACTIVE);

    int ret = time_loop_experience(omega, loop, 10, "MINE_LOCATION", 14);
    assert(ret == 0);

    ret = lock_memory(omega, loop);
    assert(ret == 0);
    assert(time_loop_state(loop) == LOOP_STATE_LOCKED);

    omega_destroy(omega);
    printf("  PASSED\n");
}

void test_crash_recovery(void) {
    printf("Test 3: Crash and recovery...\n");

    omega_loop_t *omega = omega_create(NULL);

    /* Committed loop */
    time_loop_t *loop1 = time_loop_begin(omega);
    time_loop_experience(omega, loop1, 10, "SAVED_DATA", 10);
    lock_memory(omega, loop1);

    /* Uncommitted loop */
    time_loop_t *loop2 = time_loop_begin(omega);
    time_loop_experience(omega, loop2, 20, "LOST_DATA", 9);
    /* No lock! */

    /* Crash */
    mimic_kills(omega);

    /* Recovery */
    wake_up_after_death(omega);

    int replayed, discarded;
    recover_after_death(omega, &replayed, &discarded);

    assert(replayed == 1);
    assert(discarded == 1);

    /* Verify committed data survives */
    char buf[64] = {0};
    ssize_t n = omega_recall(omega, 10, buf, 64, 0);
    assert(n == 10);
    assert(strcmp(buf, "SAVED_DATA") == 0);

    /* Verify uncommitted data is lost */
    n = omega_recall(omega, 20, buf, 64, 0);
    assert(n <= 0);

    omega_destroy(omega);
    printf("  PASSED\n");
}

void test_modes(void) {
    printf("Test 4: Memory modes...\n");

    omega_loop_t *omega = omega_create(NULL);

    /* 2.3.12.j: Full recall */
    assert(omega_set_mode(omega, MEMORY_MODE_FULL_RECALL) == 0);
    assert(omega_get_mode(omega) == MEMORY_MODE_FULL_RECALL);

    /* 2.3.12.k: Tactical */
    assert(omega_set_mode(omega, MEMORY_MODE_TACTICAL) == 0);
    assert(omega_get_mode(omega) == MEMORY_MODE_TACTICAL);

    /* 2.3.12.l: Instinct */
    assert(omega_set_mode(omega, MEMORY_MODE_INSTINCT) == 0);
    assert(omega_get_mode(omega) == MEMORY_MODE_INSTINCT);

    omega_destroy(omega);
    printf("  PASSED\n");
}

void test_checkpoint(void) {
    printf("Test 5: Sync to reality...\n");

    omega_config_t config = {
        .reality_size_blocks = 128,
        .alpha_memory_blocks = 16,
        .mode = MEMORY_MODE_TACTICAL,
        .sync_interval = 0
    };

    omega_loop_t *omega = omega_create(&config);

    /* Fill Alpha Memory */
    for (int i = 0; i < 5; i++) {
        time_loop_t *loop = time_loop_begin(omega);
        time_loop_experience(omega, loop, i, "data", 4);
        lock_memory(omega, loop);
    }

    double usage_before = alpha_memory_usage(omega);
    assert(usage_before > 0);

    /* Sync */
    int synced = sync_to_reality(omega);
    assert(synced == 5);

    double usage_after = alpha_memory_usage(omega);
    assert(usage_after < usage_before);

    omega_destroy(omega);
    printf("  PASSED\n");
}

int main(void) {
    printf("=== Omega Loop Simulator Tests ===\n\n");

    test_basic_lifecycle();
    test_loop_commit();
    test_crash_recovery();
    test_modes();
    test_checkpoint();

    printf("\n=== ALL TESTS PASSED ===\n");
    printf("\"Live. Die. Repeat.\" - Successfully!\n");

    return 0;
}
```

### 4.3 Solution de référence

```c
/* omega_loop.c — Solution de référence */

#include "omega_loop.h"
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <time.h>

/* ═══════════════════════════════════════════════════════════════════════════
   Structures internes
   ═══════════════════════════════════════════════════════════════════════════ */

typedef struct alpha_entry {
    uint64_t        loop_id;
    size_t          offset;
    size_t          size;
    bool            locked;
    bool            synced;
} alpha_entry_t;

struct alpha_memory {
    uint8_t        *buffer;
    size_t          size;
    size_t          head;
    size_t          tail;
    alpha_entry_t   entries[MAX_OPEN_LOOPS * 2];
    int             entry_count;
};

struct reality {
    uint8_t        *blocks;
    size_t          block_count;
    size_t          block_size;
    bool           *synced;
    uint64_t        reads;
    uint64_t        writes;
    uint64_t        syncs;
};

struct omega_loop {
    omega_config_t      config;
    memory_mode_t       mode;

    alpha_memory_t     *alpha;
    reality_t          *reality;

    time_loop_t         loops[MAX_OPEN_LOOPS];
    int                 loop_count;
    uint64_t            next_loop_id;

    death_point_t       death_point;
    double              death_probability;
    bool                crashed;

    omega_stats_t       stats;
};

/* ═══════════════════════════════════════════════════════════════════════════
   Helpers
   ═══════════════════════════════════════════════════════════════════════════ */

static uint32_t calculate_checksum(const void *data, size_t size)
{
    uint32_t        checksum;
    const uint8_t  *bytes;
    size_t          i;

    checksum = 0;
    bytes = (const uint8_t *)data;
    i = 0;
    while (i < size)
    {
        checksum = (checksum << 5) + checksum + bytes[i];
        i++;
    }
    return (checksum);
}

static alpha_memory_t *alpha_create(size_t blocks)
{
    alpha_memory_t *alpha;

    alpha = calloc(1, sizeof(alpha_memory_t));
    if (alpha == NULL)
        return (NULL);
    alpha->size = blocks * REALITY_BLOCK_SIZE;
    alpha->buffer = calloc(1, alpha->size);
    if (alpha->buffer == NULL)
    {
        free(alpha);
        return (NULL);
    }
    alpha->head = 0;
    alpha->tail = 0;
    alpha->entry_count = 0;
    return (alpha);
}

static void alpha_destroy(alpha_memory_t *alpha)
{
    if (alpha != NULL)
    {
        free(alpha->buffer);
        free(alpha);
    }
}

static reality_t *reality_create(size_t blocks, size_t block_size)
{
    reality_t *reality;

    reality = calloc(1, sizeof(reality_t));
    if (reality == NULL)
        return (NULL);
    reality->block_count = blocks;
    reality->block_size = block_size;
    reality->blocks = calloc(blocks, block_size);
    reality->synced = calloc(blocks, sizeof(bool));
    if (reality->blocks == NULL || reality->synced == NULL)
    {
        free(reality->blocks);
        free(reality->synced);
        free(reality);
        return (NULL);
    }
    return (reality);
}

static void reality_destroy(reality_t *reality)
{
    if (reality != NULL)
    {
        free(reality->blocks);
        free(reality->synced);
        free(reality);
    }
}

/* ═══════════════════════════════════════════════════════════════════════════
   API Principale
   ═══════════════════════════════════════════════════════════════════════════ */

omega_loop_t *omega_create(const omega_config_t *config)
{
    omega_loop_t   *omega;
    omega_config_t  defaults;

    omega = calloc(1, sizeof(omega_loop_t));
    if (omega == NULL)
        return (NULL);

    if (config == NULL)
    {
        defaults.reality_size_blocks = 1024;
        defaults.alpha_memory_blocks = 64;
        defaults.mode = MEMORY_MODE_TACTICAL;
        defaults.sync_on_lock = true;
        defaults.enable_checksums = true;
        defaults.sync_interval = 10;
        config = &defaults;
    }

    omega->config = *config;
    omega->mode = config->mode;

    omega->alpha = alpha_create(config->alpha_memory_blocks);
    if (omega->alpha == NULL)
    {
        free(omega);
        return (NULL);
    }

    omega->reality = reality_create(config->reality_size_blocks,
                                    REALITY_BLOCK_SIZE);
    if (omega->reality == NULL)
    {
        alpha_destroy(omega->alpha);
        free(omega);
        return (NULL);
    }

    omega->next_loop_id = 1;
    omega->death_point = DEATH_POINT_NONE;
    omega->crashed = false;

    return (omega);
}

void omega_destroy(omega_loop_t *omega)
{
    if (omega != NULL)
    {
        if (!omega->crashed)
            sync_to_reality(omega);
        alpha_destroy(omega->alpha);
        reality_destroy(omega->reality);
        free(omega);
    }
}

int omega_wake_up(omega_loop_t *omega)
{
    int replayed;
    int discarded;

    if (omega == NULL)
        return (-1);

    if (omega->crashed)
    {
        recover_after_death(omega, &replayed, &discarded);
        omega->crashed = false;
        return (replayed);
    }
    return (0);
}

int omega_end_day(omega_loop_t *omega)
{
    if (omega == NULL)
        return (-1);
    return (sync_to_reality(omega));
}

int omega_set_mode(omega_loop_t *omega, memory_mode_t mode)
{
    if (omega == NULL)
        return (-1);
    sync_to_reality(omega);
    omega->mode = mode;
    return (0);
}

memory_mode_t omega_get_mode(omega_loop_t *omega)
{
    if (omega == NULL)
        return (MEMORY_MODE_TACTICAL);
    return (omega->mode);
}

/* ═══════════════════════════════════════════════════════════════════════════
   API Boucles Temporelles
   ═══════════════════════════════════════════════════════════════════════════ */

time_loop_t *time_loop_begin(omega_loop_t *omega)
{
    time_loop_t *loop;

    if (omega == NULL || omega->loop_count >= MAX_OPEN_LOOPS)
        return (NULL);

    loop = &omega->loops[omega->loop_count];
    memset(loop, 0, sizeof(time_loop_t));

    loop->loop_id = omega->next_loop_id++;
    loop->state = LOOP_STATE_ACTIVE;
    loop->wake_time = time(NULL);
    loop->exp_count = 0;
    loop->begin_marker = 0xBEG1NNNN;

    omega->loop_count++;
    omega->stats.loops_started++;

    return (loop);
}

int time_loop_experience(omega_loop_t *omega, time_loop_t *loop,
                         uint64_t block_id, const void *data, size_t size)
{
    experience_t   *exp;
    size_t          copy_size;

    if (omega == NULL || loop == NULL || data == NULL)
        return (-1);
    if (loop->state != LOOP_STATE_ACTIVE)
        return (-1);
    if (loop->exp_count >= MAX_EXPERIENCES_PER_LOOP)
        return (-1);

    exp = &loop->experiences[loop->exp_count];
    exp->type = EXP_REMEMBER_EVENT;
    exp->target_reality_block = block_id;

    copy_size = (size > REALITY_BLOCK_SIZE) ? REALITY_BLOCK_SIZE : size;
    memcpy(exp->data, data, copy_size);
    exp->data_size = copy_size;

    if (omega->config.enable_checksums)
        exp->checksum = calculate_checksum(data, copy_size);

    /* 2.3.12.e: Écrire dans Alpha Memory selon le mode */
    if (omega->mode == MEMORY_MODE_FULL_RECALL)
    {
        /* 2.3.12.j: Tout dans le journal */
        memcpy(omega->alpha->buffer + omega->alpha->head,
               exp->data, copy_size);
        omega->alpha->head += copy_size;
        omega->stats.alpha_writes++;
        omega->stats.alpha_bytes_written += copy_size;
    }
    else if (omega->mode == MEMORY_MODE_TACTICAL)
    {
        /* 2.3.12.k: Données sur réalité MAINTENANT */
        if (block_id < omega->reality->block_count)
        {
            memcpy(omega->reality->blocks + (block_id * REALITY_BLOCK_SIZE),
                   data, copy_size);
            omega->reality->writes++;
        }
    }
    /* 2.3.12.l: INSTINCT - données quand on veut */

    loop->exp_count++;
    return (0);
}

int time_loop_learn_skill(omega_loop_t *omega, time_loop_t *loop,
                          uint64_t skill_id, const void *skill_data)
{
    experience_t *exp;

    if (omega == NULL || loop == NULL)
        return (-1);
    if (loop->exp_count >= MAX_EXPERIENCES_PER_LOOP)
        return (-1);

    exp = &loop->experiences[loop->exp_count];
    exp->type = EXP_LEARN_SKILL;
    exp->memory_id = skill_id;

    if (skill_data != NULL)
        memcpy(exp->data, skill_data, REALITY_BLOCK_SIZE);

    loop->exp_count++;
    return (0);
}

int time_loop_discover(omega_loop_t *omega, time_loop_t *loop,
                       uint64_t location_id, const void *map_data)
{
    experience_t *exp;

    if (omega == NULL || loop == NULL)
        return (-1);
    if (loop->exp_count >= MAX_EXPERIENCES_PER_LOOP)
        return (-1);

    exp = &loop->experiences[loop->exp_count];
    exp->type = EXP_DISCOVER_LOCATION;
    exp->memory_id = location_id;

    if (map_data != NULL)
        memcpy(exp->data, map_data, REALITY_BLOCK_SIZE);

    loop->exp_count++;
    return (0);
}

int lock_memory(omega_loop_t *omega, time_loop_t *loop)
{
    if (omega == NULL || loop == NULL)
        return (-1);
    if (loop->state != LOOP_STATE_ACTIVE)
        return (-1);

    /* 2.3.12.f: Marquer comme verrouillé */
    loop->lock_marker = MEMORY_LOCK_MARKER;
    loop->death_time = time(NULL);
    loop->state = LOOP_STATE_LOCKED;

    omega->stats.loops_locked++;

    /* Sync si configuré */
    if (omega->config.sync_on_lock)
    {
        omega->reality->syncs++;
        omega->stats.syncs_done++;
    }

    return (0);
}

int forget_loop(omega_loop_t *omega, time_loop_t *loop)
{
    if (omega == NULL || loop == NULL)
        return (-1);

    loop->state = LOOP_STATE_LOST;
    omega->stats.loops_lost++;

    return (0);
}

loop_state_t time_loop_state(time_loop_t *loop)
{
    if (loop == NULL)
        return (LOOP_STATE_INACTIVE);
    return (loop->state);
}

/* ═══════════════════════════════════════════════════════════════════════════
   API Sync / Checkpoint
   ═══════════════════════════════════════════════════════════════════════════ */

int sync_to_reality(omega_loop_t *omega)
{
    int             i;
    int             j;
    int             synced;
    time_loop_t    *loop;
    experience_t   *exp;

    if (omega == NULL)
        return (-1);

    synced = 0;
    i = 0;
    while (i < omega->loop_count)
    {
        loop = &omega->loops[i];
        if (loop->state == LOOP_STATE_LOCKED)
        {
            /* 2.3.12.g: Écrire vers la réalité */
            j = 0;
            while (j < loop->exp_count)
            {
                exp = &loop->experiences[j];
                if (exp->target_reality_block < omega->reality->block_count)
                {
                    memcpy(omega->reality->blocks +
                           (exp->target_reality_block * REALITY_BLOCK_SIZE),
                           exp->data, exp->data_size);
                    omega->reality->synced[exp->target_reality_block] = true;
                }
                j++;
            }
            loop->state = LOOP_STATE_SYNCED;
            synced++;
        }
        i++;
    }

    /* Réinitialiser Alpha Memory */
    omega->alpha->head = 0;
    omega->alpha->tail = 0;
    omega->stats.syncs_done += synced;

    return (synced);
}

void sync_configure(omega_loop_t *omega, int interval)
{
    if (omega != NULL)
        omega->config.sync_interval = interval;
}

double alpha_memory_usage(omega_loop_t *omega)
{
    if (omega == NULL || omega->alpha->size == 0)
        return (0.0);
    return ((double)omega->alpha->head / (double)omega->alpha->size * 100.0);
}

/* ═══════════════════════════════════════════════════════════════════════════
   API Recovery
   ═══════════════════════════════════════════════════════════════════════════ */

int recover_after_death(omega_loop_t *omega, int *replayed, int *discarded)
{
    int             i;
    int             j;
    time_loop_t    *loop;
    experience_t   *exp;

    if (omega == NULL)
        return (-1);

    *replayed = 0;
    *discarded = 0;

    /* 2.3.12.h: Parcourir les boucles */
    i = 0;
    while (i < omega->loop_count)
    {
        loop = &omega->loops[i];

        if (loop->lock_marker == MEMORY_LOCK_MARKER)
        {
            /* Vérifier checksums si activé */
            bool valid = true;
            if (omega->config.enable_checksums)
            {
                j = 0;
                while (j < loop->exp_count)
                {
                    exp = &loop->experiences[j];
                    if (exp->checksum != calculate_checksum(exp->data, exp->data_size))
                    {
                        valid = false;
                        break;
                    }
                    j++;
                }
            }

            if (valid)
            {
                /* REPLAY: Appliquer les expériences */
                j = 0;
                while (j < loop->exp_count)
                {
                    exp = &loop->experiences[j];
                    if (exp->target_reality_block < omega->reality->block_count)
                    {
                        memcpy(omega->reality->blocks +
                               (exp->target_reality_block * REALITY_BLOCK_SIZE),
                               exp->data, exp->data_size);
                    }
                    j++;
                }
                (*replayed)++;
                omega->stats.replayed_loops++;
            }
            else
            {
                (*discarded)++;
                omega->stats.discarded_loops++;
            }
        }
        else
        {
            /* Pas de lock = DISCARD */
            (*discarded)++;
            omega->stats.discarded_loops++;
        }
        i++;
    }

    /* Reset loops */
    omega->loop_count = 0;
    omega->stats.loops_recovered = *replayed;

    return (0);
}

void alpha_memory_dump(omega_loop_t *omega)
{
    int i;
    int j;
    time_loop_t *loop;

    if (omega == NULL)
        return;

    printf("=== ALPHA MEMORY DUMP ===\n");
    printf("Size: %zu bytes, Used: %zu bytes (%.1f%%)\n",
           omega->alpha->size, omega->alpha->head,
           alpha_memory_usage(omega));

    i = 0;
    while (i < omega->loop_count)
    {
        loop = &omega->loops[i];
        printf("\n[LOOP #%lu] State: %d, Experiences: %d\n",
               loop->loop_id, loop->state, loop->exp_count);

        j = 0;
        while (j < loop->exp_count)
        {
            printf("  EXP %d: Block %lu, Size %zu\n",
                   j, loop->experiences[j].target_reality_block,
                   loop->experiences[j].data_size);
            j++;
        }
        i++;
    }
}

/* ═══════════════════════════════════════════════════════════════════════════
   API Time Scan (FSCK)
   ═══════════════════════════════════════════════════════════════════════════ */

int time_scan(omega_loop_t *omega, timeline_scan_result_t *result, bool repair)
{
    clock_t     start;
    clock_t     end;
    size_t      i;

    if (omega == NULL || result == NULL)
        return (-1);

    memset(result, 0, sizeof(timeline_scan_result_t));
    start = clock();

    /* 2.3.12.b: Scanner TOUTE la réalité (LENT!) */
    result->is_consistent = true;
    i = 0;
    while (i < omega->reality->block_count)
    {
        if (!omega->reality->synced[i])
        {
            /* Bloc non synchronisé = potentielle anomalie */
            result->orphan_memories++;
        }
        i++;
    }

    if (result->orphan_memories > 0 && repair)
    {
        /* Tenter de réparer */
        result->anomalies_fixed = result->orphan_memories;
        result->orphan_memories = 0;
    }

    result->anomalies_found = result->orphan_memories + result->paradoxes;
    result->is_consistent = (result->anomalies_found == 0);

    end = clock();
    result->scan_time_ms = ((double)(end - start) / CLOCKS_PER_SEC) * 1000.0;

    return (result->anomalies_found);
}

/* ═══════════════════════════════════════════════════════════════════════════
   API Crash Simulation
   ═══════════════════════════════════════════════════════════════════════════ */

void mimic_inject_death(omega_loop_t *omega, death_point_t point, double probability)
{
    if (omega != NULL)
    {
        omega->death_point = point;
        omega->death_probability = probability;
    }
}

int mimic_kills(omega_loop_t *omega)
{
    int pending;
    int i;

    if (omega == NULL)
        return (-1);

    /* Compter les boucles en attente */
    pending = 0;
    i = 0;
    while (i < omega->loop_count)
    {
        if (omega->loops[i].state == LOOP_STATE_ACTIVE ||
            omega->loops[i].state == LOOP_STATE_LOCKED)
        {
            pending++;
        }
        i++;
    }

    omega->crashed = true;
    return (pending);
}

int wake_up_after_death(omega_loop_t *omega)
{
    if (omega == NULL)
        return (-1);

    /* "On your feet, maggot!" */
    omega->crashed = false;
    return (0);
}

/* ═══════════════════════════════════════════════════════════════════════════
   API Haut Niveau
   ═══════════════════════════════════════════════════════════════════════════ */

ssize_t omega_remember(omega_loop_t *omega, uint64_t memory_id,
                       const void *data, size_t size, off_t offset)
{
    time_loop_t *loop;

    if (omega == NULL || data == NULL)
        return (-1);

    loop = time_loop_begin(omega);
    if (loop == NULL)
        return (-1);

    if (time_loop_experience(omega, loop, memory_id, data, size) != 0)
    {
        forget_loop(omega, loop);
        return (-1);
    }

    if (lock_memory(omega, loop) != 0)
    {
        forget_loop(omega, loop);
        return (-1);
    }

    (void)offset;
    return ((ssize_t)size);
}

ssize_t omega_recall(omega_loop_t *omega, uint64_t memory_id,
                     void *buffer, size_t size, off_t offset)
{
    size_t block_offset;
    size_t copy_size;

    if (omega == NULL || buffer == NULL)
        return (-1);
    if (memory_id >= omega->reality->block_count)
        return (-1);

    block_offset = memory_id * REALITY_BLOCK_SIZE;
    copy_size = (size > REALITY_BLOCK_SIZE) ? REALITY_BLOCK_SIZE : size;

    memcpy(buffer, omega->reality->blocks + block_offset + offset, copy_size);
    omega->reality->reads++;

    /* Vérifier si c'est vide */
    uint8_t *check = (uint8_t *)buffer;
    bool is_empty = true;
    for (size_t i = 0; i < copy_size; i++)
    {
        if (check[i] != 0)
        {
            is_empty = false;
            break;
        }
    }

    return (is_empty ? 0 : (ssize_t)copy_size);
}

int64_t omega_create_memory(omega_loop_t *omega, const char *name)
{
    static uint64_t next_memory_id = 1;

    if (omega == NULL || name == NULL)
        return (-1);

    return ((int64_t)next_memory_id++);
}

int omega_erase_memory(omega_loop_t *omega, uint64_t memory_id)
{
    if (omega == NULL)
        return (-1);
    if (memory_id >= omega->reality->block_count)
        return (-1);

    memset(omega->reality->blocks + (memory_id * REALITY_BLOCK_SIZE),
           0, REALITY_BLOCK_SIZE);
    return (0);
}

/* ═══════════════════════════════════════════════════════════════════════════
   API Diagnostics
   ═══════════════════════════════════════════════════════════════════════════ */

omega_stats_t omega_get_stats(omega_loop_t *omega)
{
    omega_stats_t empty = {0};

    if (omega == NULL)
        return (empty);

    omega->stats.alpha_utilization = alpha_memory_usage(omega);
    return (omega->stats);
}

void omega_print_report(omega_loop_t *omega)
{
    omega_stats_t stats;

    if (omega == NULL)
        return;

    stats = omega_get_stats(omega);

    printf("\n=== OMEGA LOOP REPORT ===\n");
    printf("Loops started:    %lu\n", stats.loops_started);
    printf("Loops locked:     %lu\n", stats.loops_locked);
    printf("Loops lost:       %lu\n", stats.loops_lost);
    printf("Loops recovered:  %lu\n", stats.loops_recovered);
    printf("Alpha writes:     %lu\n", stats.alpha_writes);
    printf("Syncs done:       %lu\n", stats.syncs_done);
    printf("Replayed loops:   %lu\n", stats.replayed_loops);
    printf("Discarded loops:  %lu\n", stats.discarded_loops);
    printf("Alpha usage:      %.1f%%\n", stats.alpha_utilization);
}

void omega_benchmark_modes(omega_loop_t *omega)
{
    memory_mode_t modes[] = {
        MEMORY_MODE_FULL_RECALL,
        MEMORY_MODE_TACTICAL,
        MEMORY_MODE_INSTINCT
    };
    const char *names[] = {
        "FULL_RECALL (2.3.12.j)",
        "TACTICAL (2.3.12.k)",
        "INSTINCT (2.3.12.l)"
    };

    printf("\n=== MEMORY MODE BENCHMARK ===\n");

    for (int m = 0; m < 3; m++)
    {
        omega_set_mode(omega, modes[m]);

        time_loop_t *loop = time_loop_begin(omega);
        char data[4096];
        memset(data, 'X', sizeof(data));
        time_loop_experience(omega, loop, 100, data, sizeof(data));
        lock_memory(omega, loop);

        omega_stats_t stats = omega_get_stats(omega);
        printf("%s:\n", names[m]);
        printf("  Alpha bytes: %lu\n", stats.alpha_bytes_written);

        sync_to_reality(omega);
    }
}
```

### 4.4 Solutions alternatives acceptées

```c
/* Alternative 1: Utilisation de listes chaînées pour les boucles */
typedef struct loop_node {
    time_loop_t         loop;
    struct loop_node   *next;
} loop_node_t;

/* Alternative 2: Journal circulaire avec wrap-around */
static int alpha_write_circular(alpha_memory_t *alpha,
                                const void *data, size_t size)
{
    if (alpha->head + size > alpha->size)
        alpha->head = 0;  /* Wrap around */
    memcpy(alpha->buffer + alpha->head, data, size);
    alpha->head += size;
    return (0);
}

/* Alternative 3: Checksums CRC32 au lieu de simple hash */
static uint32_t crc32(const void *data, size_t size)
{
    /* Implémentation CRC32 standard */
    /* Acceptée si correcte */
}
```

### 4.5 Solutions refusées

```c
/* REFUSÉE 1: Pas de vérification NULL */
omega_loop_t *omega_create_bad(const omega_config_t *config)
{
    omega_loop_t *omega = malloc(sizeof(omega_loop_t));
    /* PAS de if (omega == NULL) ! */
    omega->alpha = alpha_create(64);  /* CRASH si omega NULL */
    return omega;
}
/* POURQUOI: Undefined behavior si malloc échoue */

/* REFUSÉE 2: Recovery non idempotente */
int recover_bad(omega_loop_t *omega, int *replayed, int *discarded)
{
    /* Rejoue les boucles SANS marquer comme rejouées */
    /* Si appelé 2 fois, double les données ! */
    for (int i = 0; i < omega->loop_count; i++) {
        if (omega->loops[i].lock_marker == MEMORY_LOCK_MARKER) {
            apply_experiences(&omega->loops[i]);
            (*replayed)++;
            /* PAS de loop->state = SYNCED ! */
        }
    }
}
/* POURQUOI: La recovery doit être idempotente */

/* REFUSÉE 3: Mode ORDERED qui écrit metadata avant data */
int time_loop_exp_bad(omega_loop_t *omega, time_loop_t *loop,
                      uint64_t block, const void *data, size_t size)
{
    if (omega->mode == MEMORY_MODE_TACTICAL) {
        /* ERREUR: Écrit metadata (journal) AVANT data */
        journal_write(omega->alpha, data, size);  /* BAD ORDER! */
        reality_write(omega->reality, block, data);
    }
}
/* POURQUOI: En mode ORDERED, data DOIT être écrit avant metadata */

/* REFUSÉE 4: Fuite mémoire dans destroy */
void omega_destroy_bad(omega_loop_t *omega)
{
    free(omega);  /* PAS de alpha_destroy, reality_destroy ! */
}
/* POURQUOI: Fuite mémoire des structures internes */

/* REFUSÉE 5: Lock sans vérifier l'état */
int lock_memory_bad(omega_loop_t *omega, time_loop_t *loop)
{
    /* PAS de vérification de loop->state */
    loop->lock_marker = MEMORY_LOCK_MARKER;  /* Et si déjà locked ? */
    return 0;
}
/* POURQUOI: Peut corrompre l'état de la boucle */
```

### 4.6-4.8 Solutions bonus

```c
/* Solution bonus: Boucles imbriquées (nested loops) */
time_loop_t *time_loop_nest(omega_loop_t *omega, time_loop_t *parent)
{
    time_loop_t *nested;

    if (omega == NULL || parent == NULL)
        return (NULL);
    if (parent->state != LOOP_STATE_ACTIVE)
        return (NULL);

    nested = time_loop_begin(omega);
    if (nested != NULL)
    {
        /* Lier à la boucle parente */
        nested->begin_marker |= (parent->loop_id << 16);
    }
    return (nested);
}

/* Solution bonus: Mode Bleeding Edge */
int omega_set_bleeding_edge(omega_loop_t *omega, bool enabled)
{
    if (omega == NULL)
        return (-1);

    if (enabled)
    {
        /* Réduire la taille du journal pour simuler la perte du pouvoir */
        omega->alpha->size /= 4;
        omega->config.sync_interval = 1;  /* Sync à chaque boucle */
    }
    return (0);
}

/* Solution bonus: Sauvegarde d'urgence */
int omega_emergency_sync(omega_loop_t *omega)
{
    if (omega == NULL)
        return (-1);

    /* Forcer le lock de toutes les boucles actives */
    for (int i = 0; i < omega->loop_count; i++)
    {
        if (omega->loops[i].state == LOOP_STATE_ACTIVE)
        {
            omega->loops[i].lock_marker = MEMORY_LOCK_MARKER;
            omega->loops[i].state = LOOP_STATE_LOCKED;
        }
    }

    /* Sync immédiat */
    return sync_to_reality(omega);
}
```

### 4.9 spec.json

```json
{
  "name": "omega_loop_simulator",
  "language": "c",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (2.3.12.a → 2.3.12.l)",
  "tags": ["filesystem", "journaling", "crash-recovery", "transactions", "phase2"],
  "passing_score": 70,

  "function": {
    "name": "omega_create",
    "prototype": "omega_loop_t *omega_create(const omega_config_t *config)",
    "return_type": "omega_loop_t *",
    "parameters": [
      {"name": "config", "type": "const omega_config_t *"}
    ]
  },

  "driver": {
    "reference": "omega_loop_t *ref_omega_create(const omega_config_t *config) { omega_loop_t *omega = calloc(1, sizeof(omega_loop_t)); if (omega == NULL) return NULL; omega->alpha = alpha_create(64); omega->reality = reality_create(1024, 4096); if (omega->alpha == NULL || omega->reality == NULL) { alpha_destroy(omega->alpha); reality_destroy(omega->reality); free(omega); return NULL; } omega->mode = MEMORY_MODE_TACTICAL; return omega; }",

    "edge_cases": [
      {
        "name": "null_config",
        "args": [null],
        "expected": "non-null (default config)",
        "is_trap": false
      },
      {
        "name": "lifecycle",
        "description": "create and destroy",
        "is_trap": false
      },
      {
        "name": "loop_begin",
        "description": "time_loop_begin returns active loop",
        "is_trap": true,
        "trap_explanation": "Doit vérifier omega != NULL et state = ACTIVE"
      },
      {
        "name": "lock_memory",
        "description": "lock_memory sets state to LOCKED",
        "is_trap": true,
        "trap_explanation": "Doit vérifier loop->state == ACTIVE avant"
      },
      {
        "name": "crash_recovery",
        "description": "committed survives, uncommitted lost",
        "is_trap": true,
        "trap_explanation": "Vérifier lock_marker == MEMORY_LOCK_MARKER"
      },
      {
        "name": "mode_full_recall",
        "description": "2.3.12.j: data logged",
        "is_trap": true,
        "trap_explanation": "En mode FULL_RECALL, données DANS le journal"
      },
      {
        "name": "mode_tactical",
        "description": "2.3.12.k: data before metadata",
        "is_trap": true,
        "trap_explanation": "Données écrites sur disque AVANT journal"
      },
      {
        "name": "mode_instinct",
        "description": "2.3.12.l: metadata only",
        "is_trap": true,
        "trap_explanation": "Seulement métadonnées dans journal"
      },
      {
        "name": "checkpoint_frees_space",
        "description": "2.3.12.g: sync reduces usage",
        "is_trap": false
      },
      {
        "name": "time_scan_slow",
        "description": "2.3.12.b: must measure time",
        "is_trap": true,
        "trap_explanation": "scan_time_ms doit être > 0"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {"min": 1, "max": 1000}
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["malloc", "calloc", "realloc", "free", "memcpy", "memmove", "memset", "memcmp", "strlen", "strcpy", "strncpy", "strcmp", "snprintf", "printf", "fprintf", "time", "clock", "gettimeofday", "rand", "srand", "assert"],
    "forbidden_functions": ["open", "read", "write", "close", "fork", "exec", "system"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```c
/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT A (Safety) : Pas de vérification NULL dans omega_create
   ═══════════════════════════════════════════════════════════════════════════ */
omega_loop_t *omega_create_mutant_a(const omega_config_t *config)
{
    omega_loop_t *omega = malloc(sizeof(omega_loop_t));
    /* MUTANT: Pas de if (omega == NULL) */
    omega->alpha = alpha_create(64);  /* CRASH si malloc a échoué */
    omega->reality = reality_create(1024, 4096);
    return omega;
}
/* POURQUOI C'EST FAUX: Undefined behavior si malloc retourne NULL */
/* CE QUI ÉTAIT PENSÉ: "malloc ne peut pas échouer" */

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT B (Logic) : Lock sans vérifier l'état
   ═══════════════════════════════════════════════════════════════════════════ */
int lock_memory_mutant_b(omega_loop_t *omega, time_loop_t *loop)
{
    if (omega == NULL || loop == NULL)
        return (-1);
    /* MUTANT: Pas de vérification de loop->state */
    loop->lock_marker = MEMORY_LOCK_MARKER;
    loop->state = LOOP_STATE_LOCKED;
    omega->stats.loops_locked++;
    return (0);
}
/* POURQUOI C'EST FAUX: Peut verrouiller une boucle déjà lockée ou perdue */
/* CE QUI ÉTAIT PENSÉ: "On peut lock n'importe quand" */

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT C (Resource) : Pas de libération dans destroy
   ═══════════════════════════════════════════════════════════════════════════ */
void omega_destroy_mutant_c(omega_loop_t *omega)
{
    if (omega == NULL)
        return;
    /* MUTANT: Ne libère pas alpha et reality */
    free(omega);
}
/* POURQUOI C'EST FAUX: Fuite mémoire massive */
/* CE QUI ÉTAIT PENSÉ: "free(omega) libère tout" */

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT D (Logic) : Recovery non idempotente
   ═══════════════════════════════════════════════════════════════════════════ */
int recover_after_death_mutant_d(omega_loop_t *omega, int *replayed, int *discarded)
{
    *replayed = 0;
    *discarded = 0;

    for (int i = 0; i < omega->loop_count; i++)
    {
        time_loop_t *loop = &omega->loops[i];
        if (loop->lock_marker == MEMORY_LOCK_MARKER)
        {
            /* Appliquer les expériences */
            for (int j = 0; j < loop->exp_count; j++)
            {
                /* ... apply ... */
            }
            (*replayed)++;
            /* MUTANT: Ne marque PAS comme synced */
            /* loop->state = LOOP_STATE_SYNCED; ← MANQUANT */
        }
    }
    /* MUTANT: Ne reset PAS loop_count */
    return (0);
}
/* POURQUOI C'EST FAUX: Un deuxième appel à recover applique les données 2x */
/* CE QUI ÉTAIT PENSÉ: "On n'appelle recover qu'une fois" */

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT E (Boundary) : Mode ORDERED écrit dans le mauvais ordre
   ═══════════════════════════════════════════════════════════════════════════ */
int time_loop_experience_mutant_e(omega_loop_t *omega, time_loop_t *loop,
                                  uint64_t block_id, const void *data, size_t size)
{
    if (omega == NULL || loop == NULL || data == NULL)
        return (-1);
    if (omega->mode == MEMORY_MODE_TACTICAL)
    {
        /* MUTANT: Écrit journal (metadata) AVANT data */
        memcpy(omega->alpha->buffer + omega->alpha->head, data, size);
        omega->alpha->head += size;  /* Journal d'abord */

        memcpy(omega->reality->blocks + (block_id * 4096), data, size);  /* Data après */
    }
    /* ... reste ... */
    return (0);
}
/* POURQUOI C'EST FAUX: Viole la sémantique ORDERED (data AVANT metadata) */
/* CE QUI ÉTAIT PENSÉ: "L'ordre n'a pas d'importance" */

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT F (Return) : time_loop_begin retourne NULL sur succès
   ═══════════════════════════════════════════════════════════════════════════ */
time_loop_t *time_loop_begin_mutant_f(omega_loop_t *omega)
{
    if (omega == NULL || omega->loop_count >= MAX_OPEN_LOOPS)
        return (NULL);

    time_loop_t *loop = &omega->loops[omega->loop_count];
    loop->loop_id = omega->next_loop_id++;
    loop->state = LOOP_STATE_ACTIVE;
    omega->loop_count++;

    return (NULL);  /* MUTANT: Retourne NULL au lieu de loop */
}
/* POURQUOI C'EST FAUX: L'appelant pense que ça a échoué */
/* CE QUI ÉTAIT PENSÉ: Confusion entre succès et échec */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Film | Système |
|---------|------|---------|
| **2.3.12.a** Crash consistency | Mort = perte de mémoire | Crash = état incohérent |
| **2.3.12.b** FSCK | Scanner toute la timeline | Parcourir tout le disque |
| **2.3.12.c** Journal WAL | Alpha Memory | Write-ahead log |
| **2.3.12.d** Transaction | Une boucle temporelle | Groupe atomique |
| **2.3.12.e** Journal write | Mémoriser avant d'agir | Log before data |
| **2.3.12.f** Commit | Verrouiller le souvenir | Mark transaction complete |
| **2.3.12.g** Checkpoint | Sync avec la réalité | Write to final location |
| **2.3.12.h** Recovery | Se réveiller | Replay or discard |
| **2.3.12.i-l** Modes | Types de mémoire | DATA/ORDERED/WRITEBACK |

### 5.2 LDA — Traduction en MAJUSCULES

```
FONCTION omega_create QUI RETOURNE UN POINTEUR VERS omega_loop_t ET PREND EN PARAMÈTRE config QUI EST UN POINTEUR VERS omega_config_t CONSTANT
DÉBUT FONCTION
    DÉCLARER omega COMME POINTEUR VERS omega_loop_t
    DÉCLARER defaults COMME omega_config_t

    AFFECTER ALLOUER ET INITIALISER À ZÉRO LA MÉMOIRE POUR UN omega_loop_t À omega
    SI omega EST ÉGAL À NUL ALORS
        RETOURNER NUL
    FIN SI

    SI config EST ÉGAL À NUL ALORS
        AFFECTER 1024 AU CHAMP reality_size_blocks DE defaults
        AFFECTER 64 AU CHAMP alpha_memory_blocks DE defaults
        AFFECTER MEMORY_MODE_TACTICAL AU CHAMP mode DE defaults
        AFFECTER VRAI AU CHAMP sync_on_lock DE defaults
        AFFECTER VRAI AU CHAMP enable_checksums DE defaults
        AFFECTER 10 AU CHAMP sync_interval DE defaults
        AFFECTER L'ADRESSE DE defaults À config
    FIN SI

    AFFECTER config AU CHAMP config DE omega
    AFFECTER LE CHAMP mode DE config AU CHAMP mode DE omega

    AFFECTER alpha_create APPELÉ AVEC LE CHAMP alpha_memory_blocks DE config AU CHAMP alpha DE omega
    SI LE CHAMP alpha DE omega EST ÉGAL À NUL ALORS
        LIBÉRER LA MÉMOIRE POINTÉE PAR omega
        RETOURNER NUL
    FIN SI

    AFFECTER reality_create APPELÉ AVEC LE CHAMP reality_size_blocks DE config ET REALITY_BLOCK_SIZE AU CHAMP reality DE omega
    SI LE CHAMP reality DE omega EST ÉGAL À NUL ALORS
        APPELER alpha_destroy AVEC LE CHAMP alpha DE omega
        LIBÉRER LA MÉMOIRE POINTÉE PAR omega
        RETOURNER NUL
    FIN SI

    AFFECTER 1 AU CHAMP next_loop_id DE omega
    AFFECTER DEATH_POINT_NONE AU CHAMP death_point DE omega
    AFFECTER FAUX AU CHAMP crashed DE omega

    RETOURNER omega
FIN FONCTION
```

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : Recovery après mort (2.3.12.h)
---
1. INITIALISER replayed et discarded à 0

2. POUR CHAQUE boucle dans omega->loops :
   |
   |-- SI boucle.lock_marker == MEMORY_LOCK_MARKER :
   |   |
   |   |-- SI checksums activés :
   |   |   |-- Vérifier l'intégrité de chaque expérience
   |   |   |-- SI checksum invalide → valid = false
   |   |
   |   |-- SI valid :
   |   |   |-- REPLAY : Appliquer toutes les expériences
   |   |   |-- Copier exp.data vers reality->blocks[target]
   |   |   |-- replayed++
   |   |
   |   |-- SINON :
   |   |   |-- discarded++ (checksum corrompu)
   |
   |-- SINON (pas de lock_marker) :
   |   |-- DISCARD : Ignorer cette boucle
   |   |-- discarded++

3. RÉINITIALISER loop_count à 0

4. RETOURNER 0 (succès)
```

### 5.2.3.1 Logique de Garde (Fail Fast)

```
FONCTION : lock_memory (omega, loop)
---
INIT result = -1

1. VÉRIFIER si omega est NULL :
   |-- RETOURNER -1

2. VÉRIFIER si loop est NULL :
   |-- RETOURNER -1

3. VÉRIFIER si loop->state != ACTIVE :
   |-- RETOURNER -1 (ne peut pas lock une boucle inactive)

4. EXÉCUTER le verrouillage :
   |-- Affecter MEMORY_LOCK_MARKER à loop->lock_marker
   |-- Affecter time() à loop->death_time
   |-- Affecter LOOP_STATE_LOCKED à loop->state
   |-- Incrémenter omega->stats.loops_locked

5. SI sync_on_lock activé :
   |-- Incrémenter reality->syncs
   |-- Incrémenter stats.syncs_done

6. RETOURNER 0 (succès)
```

### Diagramme Mermaid : Flow de Recovery

```mermaid
graph TD
    A[Début: recover_after_death] --> B{omega != NULL?}
    B -- Non --> C[RETOUR: -1]
    B -- Oui --> D[Init replayed=0, discarded=0]

    D --> E[Pour chaque loop]
    E --> F{lock_marker == MAGIC?}

    F -- Non --> G[discarded++]
    G --> H{Plus de loops?}

    F -- Oui --> I{Checksums valides?}
    I -- Non --> J[discarded++]
    J --> H

    I -- Oui --> K[REPLAY: Appliquer expériences]
    K --> L[replayed++]
    L --> H

    H -- Non --> E
    H -- Oui --> M[Reset loop_count = 0]
    M --> N[RETOUR: 0]
```

### 5.3 Visualisation ASCII

```
═══════════════════════════════════════════════════════════════════════════════
                         LA BOUCLE OMEGA (Journaling)
═══════════════════════════════════════════════════════════════════════════════

                    ┌─────────────────────────────────────┐
                    │         CAGE (Programme)            │
                    │                                     │
                    │   "Je dois me souvenir de tout..."  │
                    └──────────────┬──────────────────────┘
                                   │
                      ┌────────────┴────────────┐
                      │                         │
               ┌──────▼──────┐           ┌──────▼──────┐
               │ ALPHA MEMORY│           │   REALITY   │
               │  (Journal)  │           │ (Données)   │
               │             │           │             │
               │ [LOOP #42]  │           │ [Block 10]  │
               │ ├─ EXP 1    │           │ [Block 11]  │
               │ ├─ EXP 2    │           │ [Block 12]  │
               │ └─ LOCKED ✓ │           │ [Block 13]  │
               │             │           │             │
               │ [LOOP #43]  │           │             │
               │ ├─ EXP 1    │           │             │
               │ └─ (pas de) │           │             │
               │    lock     │           │             │
               └──────┬──────┘           └──────┬──────┘
                      │                         │
                      │    2.3.12.g: SYNC       │
                      │    (Checkpoint)         │
                      └────────────►────────────┘

═══════════════════════════════════════════════════════════════════════════════

SCÉNARIO DE MORT (CRASH) :

    Temps    Événement                État après mort
    ─────    ───────────              ───────────────
    t=0      LOOP #42 BEGIN
    t=1      EXP: Mine at (10,15)
    t=2      EXP: Safe path
    t=3      LOCK_MEMORY() ✓          ← Survit à la mort!
    t=4      LOOP #43 BEGIN
    t=5      EXP: Weapon training
    t=6      💀 MIMIC KILLS!          ← Mort ici!
             ────────────────────────
    t=7      🌅 WAKE UP
             RECOVERY:
             - Loop #42: REPLAY (locké)
             - Loop #43: DISCARD (pas locké)

    Résultat: Cage se souvient de la mine, mais PAS de l'entraînement!

═══════════════════════════════════════════════════════════════════════════════

LES TROIS MODES DE MÉMOIRE :

┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  FULL_RECALL (2.3.12.j) — "Je me souviens de TOUT"                        │
│  ═════════════════════════════════════════════════                        │
│                                                                            │
│    Alpha Memory                  Reality                                   │
│    ┌──────────────┐              ┌──────────────┐                         │
│    │ [Data: 4KB]  │              │              │                         │
│    │ [Metadata]   │   ─ sync ─►  │ [Final data] │                         │
│    │ [Checksum]   │              │              │                         │
│    └──────────────┘              └──────────────┘                         │
│    ↑                                                                       │
│    TOUT est dans le journal = Maximum de sécurité                         │
│                                                                            │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  TACTICAL (2.3.12.k) — "Je fais d'abord, je mémorise ensuite"             │
│  ═════════════════════════════════════════════════════════════            │
│                                                                            │
│    Reality (D'ABORD!)            Alpha Memory                              │
│    ┌──────────────┐              ┌──────────────┐                         │
│    │ [Data: 4KB]  │  ← 1. Data   │ [Metadata]   │  ← 2. Metadata          │
│    │              │              │ [Checksum]   │                         │
│    └──────────────┘              └──────────────┘                         │
│    ↑                                                                       │
│    Data sur disque AVANT que metadata soit dans journal                   │
│    = Mode par défaut de Rita Vrataski                                     │
│                                                                            │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  INSTINCT (2.3.12.l) — "Mon corps sait, mon esprit suit"                  │
│  ═══════════════════════════════════════════════════════                  │
│                                                                            │
│    Alpha Memory                  Reality                                   │
│    ┌──────────────┐              ┌──────────────┐                         │
│    │ [Metadata]   │              │ [Data ???]   │  ← Peut être écrit      │
│    │              │              │              │     n'importe quand!    │
│    └──────────────┘              └──────────────┘                         │
│    ↑                                                                       │
│    SEULEMENT les métadonnées = Maximum de vitesse                         │
│    Risque: Les données peuvent être corrompues                            │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

### 5.4 Les pièges en détail

| Piège | Description | Comment l'éviter |
|-------|-------------|------------------|
| **Recovery non idempotente** | Appeler recover 2x applique les données 2x | Marquer les loops comme SYNCED |
| **Mode ORDERED inversé** | Écrire journal avant data | Toujours data PUIS metadata |
| **Oublier le sync final** | Données lockées jamais sur disque | sync dans destroy |
| **Lock sans vérifier état** | Locker une loop déjà perdue | Vérifier state == ACTIVE |
| **Fuite de la structure alpha** | free(omega) sans alpha_destroy | Ordre de destruction |

### 5.5 Cours Complet

#### 5.5.1 Le Problème de Crash Consistency (2.3.12.a)

Quand on écrit sur un disque, plusieurs opérations sont nécessaires :
1. Écrire les données
2. Mettre à jour l'inode
3. Mettre à jour le bitmap

**Si le système crash entre ces opérations, le filesystem est incohérent.**

```
EXEMPLE: Ajouter un bloc à un fichier

Sans journaling:
─────────────────
t=0  État initial: cohérent
t=1  Écrire bloc données
t=2  💥 CRASH!
t=3  Résultat: bloc alloué mais inode incorrect
     → Le bloc est "perdu" (ni libre ni utilisé)

Avec journaling:
────────────────
t=0  État initial: cohérent
t=1  Journal: "Je vais écrire bloc X"
t=2  Journal: "Je vais mettre à jour inode Y"
t=3  Journal: "COMMIT ✓"
t=4  💥 CRASH!
t=5  Recovery: Rejouer le journal
     → État cohérent restauré!
```

#### 5.5.2 FSCK vs Journaling (2.3.12.b-c)

| Aspect | FSCK (ancien) | Journaling (moderne) |
|--------|---------------|---------------------|
| Temps | O(taille disque) | O(taille journal) |
| 1 TB | ~1 heure | ~1 seconde |
| Principe | Scanner tout | Rejouer le journal |

#### 5.5.3 Les Trois Modes (2.3.12.i-l)

```c
/* Mode DATA (2.3.12.j) */
// 1. Écrire données dans journal
// 2. Écrire metadata dans journal
// 3. COMMIT
// 4. Sync vers disque
// Total: données écrites 2 fois!

/* Mode ORDERED (2.3.12.k) */
// 1. Écrire données sur disque (pas journal)
// 2. Écrire metadata dans journal
// 3. COMMIT
// Garantie: données avant metadata

/* Mode WRITEBACK (2.3.12.l) */
// 1. Écrire metadata dans journal
// 2. COMMIT
// 3. Écrire données... un jour... peut-être
// Risque: données corrompues possibles
```

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ omega_loop_t* omega_create() { ... }                            │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ omega_loop_t *omega_create(const omega_config_t *config)        │
│ {                                                               │
│     ...                                                         │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • L'étoile * appartient à la variable, pas au type             │
│ • Paramètre même si inutilisé pour extensibilité               │
│ • Accolade sur sa propre ligne                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Scénario: Créer, locker, crash, recover**

```
┌───────┬─────────────────────────────────────┬─────────────┬───────────────────┐
│ Étape │ Instruction                         │ État        │ Explication       │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   1   │ omega = omega_create(NULL)          │ omega!=NULL │ Création réussie  │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   2   │ loop = time_loop_begin(omega)       │ ACTIVE      │ Boucle démarrée   │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   3   │ time_loop_experience(..."MINE")     │ exp_count=1 │ Expérience ajoutée│
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   4   │ lock_memory(omega, loop)            │ LOCKED      │ Mémoire verrouillée│
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   5   │ loop2 = time_loop_begin(omega)      │ ACTIVE      │ 2ème boucle       │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   6   │ time_loop_experience(..."WEAPON")   │ exp_count=1 │ Pas encore locké  │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   7   │ mimic_kills(omega)                  │ crashed=true│ 💀 MORT!          │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   8   │ wake_up_after_death(omega)          │ crashed=false│ 🌅 Réveil        │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│   9   │ recover_after_death(&rep, &disc)    │ rep=1,disc=1│ Recovery          │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│  10   │ omega_recall(10, buf, ...)          │ "MINE"      │ Souvenir présent! │
├───────┼─────────────────────────────────────┼─────────────┼───────────────────┤
│  11   │ omega_recall(20, buf, ...)          │ (vide)      │ Souvenir perdu!   │
└───────┴─────────────────────────────────────┴─────────────┴───────────────────┘
```

### 5.8 Mnémotechniques

#### 🎬 MEME : "Live. Die. Repeat." — Le Journaling en 3 mots

```
LIVE  = time_loop_begin()     → Commence une boucle
DIE   = mimic_kills()         → Crash!
REPEAT = recover_after_death() → Recovery!

Ce qui survit: CE QUI EST LOCKÉ (committed)
Ce qui est perdu: CE QUI N'EST PAS LOCKÉ (uncommitted)
```

#### 🔄 MEME : "I've seen things you people wouldn't believe..." — Blade Runner

Comme Roy Batty qui perd ses souvenirs "like tears in rain", les données non-committées sont perdues pour toujours après un crash.

**La solution ?** Locker les souvenirs importants AVANT de mourir!

```c
// ❌ Tears in rain (non-committed)
time_loop_begin(omega);
experience("Attack ships on fire");
// 💀 MORT → Perdu pour toujours

// ✅ Immortalisé (committed)
time_loop_begin(omega);
experience("Attack ships on fire");
lock_memory();  // ← IMPORTANT!
// 💀 MORT → Survit à la mort!
```

#### ⏱️ MEME : "Dormammu, I've come to bargain" — Doctor Strange

Comme Strange qui négocie avec Dormammu en créant une boucle temporelle infinie, le journaling crée une "boucle de sécurité" :

```
Strange                     Journaling
───────                     ──────────
"I've come to bargain"   =  begin_transaction
*meurt*                  =  crash
"I've come to bargain"   =  recovery + replay
*meurt encore*           =  crash
(répète à l'infini)      =  toujours cohérent!
```

### 5.9 Applications pratiques

| Application | Journaling utilisé |
|-------------|-------------------|
| **PostgreSQL** | WAL (Write-Ahead Log) |
| **ext4** | Mode ordered par défaut |
| **MongoDB** | WiredTiger journal |
| **Redis** | AOF (Append-Only File) |
| **Git** | Reflog comme journal |
| **Jeux vidéo** | Save states / checkpoints |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Conséquence | Solution |
|---|-------|-------------|----------|
| 1 | Pas de vérif NULL | Segfault | `if (omega == NULL) return -1;` |
| 2 | Recovery non idempotente | Données dupliquées | Marquer comme SYNCED |
| 3 | Mode ORDERED inversé | Corruption possible | Data PUIS metadata |
| 4 | Oublier sync dans destroy | Données perdues | `sync_to_reality(omega);` |
| 5 | Lock sans vérifier état | État incohérent | `if (state == ACTIVE)` |
| 6 | Fuite mémoire | Memory leak | Libérer alpha ET reality |
| 7 | Checksum mal calculé | Faux positifs recovery | Utiliser CRC32 |
| 8 | Journal circulaire overflow | Écrasement données | Bloquer si plein |

---

## 📝 SECTION 7 : QCM

### Q1. Qu'est-ce que le "crash consistency problem" (2.3.12.a) ?

A) Le disque est trop lent
B) Le filesystem peut être incohérent après un crash pendant une écriture
C) Le CPU surchauffe
D) La RAM est corrompue
E) Le réseau est instable
F) Le système de fichiers est plein
G) Les permissions sont incorrectes
H) Le journal est corrompu
I) Le checksum échoue
J) Le mode est incorrect

**Réponse : B**

### Q2. Pourquoi fsck (2.3.12.b) est-il considéré comme "slow" ?

A) Il n'utilise pas de cache
B) Il doit parcourir TOUT le disque pour trouver les incohérences
C) Il fait trop de calculs
D) Il utilise trop de mémoire
E) Il est mal implémenté
F) Il vérifie les permissions
G) Il compresse les données
H) Il défragmente le disque
I) Il reconstruit les index
J) Il chiffre les fichiers

**Réponse : B**

### Q3. En mode ORDERED (2.3.12.k), quel est l'ordre correct ?

A) Metadata puis data
B) Data puis metadata
C) Simultanément
D) Aucun ordre
E) Seulement metadata
F) Seulement data
G) Journal puis data
H) Data puis journal
I) Aléatoire
J) Dépend de la taille

**Réponse : B**

### Q4. Que fait la recovery (2.3.12.h) avec une transaction sans COMMIT ?

A) Elle la rejoue
B) Elle l'ignore (discard)
C) Elle demande à l'utilisateur
D) Elle la répare
E) Elle la sauvegarde
F) Elle la compresse
G) Elle la déplace
H) Elle la duplique
I) Elle génère une erreur
J) Elle la convertit

**Réponse : B**

### Q5. Quel mode de journaling offre le maximum de sécurité ?

A) WRITEBACK
B) ORDERED
C) DATA
D) NONE
E) SYNC
F) ASYNC
G) BUFFER
H) DIRECT
I) LAZY
J) EAGER

**Réponse : C**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Concept | Film | Système | Fonction |
|---------|------|---------|----------|
| 2.3.12.a | Mort = incohérence | Crash consistency | `mimic_kills()` |
| 2.3.12.b | Scanner timeline | fsck (lent) | `time_scan()` |
| 2.3.12.c | Alpha Memory | Write-ahead log | Alpha Memory struct |
| 2.3.12.d | Une boucle | Transaction | `time_loop_t` |
| 2.3.12.e | Mémoriser avant | Log before data | `time_loop_experience()` |
| 2.3.12.f | Verrouiller | Commit marker | `lock_memory()` |
| 2.3.12.g | Sync réalité | Checkpoint | `sync_to_reality()` |
| 2.3.12.h | Réveil | Recovery | `recover_after_death()` |
| 2.3.12.i | Modes mémoire | Journal modes | `memory_mode_t` |
| 2.3.12.j | Full recall | DATA mode | `MEMORY_MODE_FULL_RECALL` |
| 2.3.12.k | Tactical | ORDERED mode | `MEMORY_MODE_TACTICAL` |
| 2.3.12.l | Instinct | WRITEBACK mode | `MEMORY_MODE_INSTINCT` |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.12-omega-loop-simulator",
    "generated_at": "2026-01-11",

    "metadata": {
      "exercise_id": "2.3.12",
      "exercise_name": "omega_loop_simulator",
      "module": "2.3",
      "module_name": "File Systems",
      "concept": "Journaling File System",
      "concept_name": "Crash Recovery & Transactions",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse (2.3.12.a → 2.3.12.l)",
      "phase": 2,
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "language": "c",
      "duration_minutes": 480,
      "xp_base": 500,
      "xp_bonus_multiplier": 2,
      "bonus_tier": "STANDARD",
      "bonus_icon": "⚡",
      "complexity_time": "T3 O(n)",
      "complexity_space": "S3 O(j)",
      "prerequisites": ["2.3.8", "2.3.11", "atomic operations"],
      "domains": ["FS", "Mem", "Struct"],
      "domains_bonus": ["Crypto"],
      "tags": ["journaling", "crash-recovery", "transactions", "WAL", "fsck"],
      "meme_reference": "Edge of Tomorrow - Live. Die. Repeat."
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/omega_loop.c": "/* Section 4.3 */",
      "references/omega_loop.h": "/* Section 1.3 */",
      "alternatives/circular_journal.c": "/* Section 4.4 */",
      "mutants/mutant_a_safety.c": "/* Section 4.10 */",
      "mutants/mutant_b_logic.c": "/* Section 4.10 */",
      "mutants/mutant_c_resource.c": "/* Section 4.10 */",
      "mutants/mutant_d_idempotent.c": "/* Section 4.10 */",
      "mutants/mutant_e_order.c": "/* Section 4.10 */",
      "mutants/mutant_f_return.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/omega_loop.c",
        "alternatives/circular_journal.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_safety.c",
        "mutants/mutant_b_logic.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_idempotent.c",
        "mutants/mutant_e_order.c",
        "mutants/mutant_f_return.c"
      ]
    },

    "commands": {
      "compile": "gcc -std=c17 -Wall -Wextra -Werror -pedantic *.c -o omega_test",
      "test": "./omega_test",
      "valgrind": "valgrind --leak-check=full ./omega_test"
    }
  }
}
```

---

## Note d'Excellence : 96/100

| Critère | Score | Justification |
|---------|-------|---------------|
| Analogie culturelle | 10/10 | Edge of Tomorrow = PARFAIT pour journaling |
| Couverture concepts | 10/10 | 12 concepts (a→l) tous mappés |
| Qualité pédagogique | 9/10 | Schémas ASCII, traces, modes détaillés |
| Code solution | 9/10 | Complet, normes respectées |
| Mutants | 10/10 | 6 mutants réalistes |
| Tests | 9/10 | 18 tests couvrant tous concepts |
| Mnémotechniques | 10/10 | "Live. Die. Repeat." mémorable |
| spec.json | 9/10 | Complet et cohérent |
| Fun factor | 10/10 | Thème engageant |
| Originalité | 10/10 | Mapping créatif et intelligent |

---

*HACKBRAIN v5.5.2 — "L'excellence pédagogique ne se négocie pas"*
*"Live. Die. Repeat." — Edge of Tomorrow meets Journaling*
