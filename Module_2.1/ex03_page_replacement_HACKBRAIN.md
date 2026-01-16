<thinking>
## Analyse du Concept
- Concept : Algorithmes de remplacement de pages (FIFO, LRU, Clock, Optimal, etc.)
- Phase demandée : 2
- Adapté ? OUI — Les algorithmes de remplacement sont fondamentaux en OS et peuvent être enseignés via simulation

## Combo Base + Bonus
- Exercice de base : Simulateur comparant 7 algorithmes de remplacement sur des traces d'accès
- Bonus : Détection de thrashing + analyse working set + Page Fault Frequency
- Palier bonus : 🔥 Avancé (analyse statistique complexe)
- Progression logique ? OUI — Base = algorithmes, Bonus = détection problèmes

## Prérequis & Difficulté
- Prérequis réels : Listes chaînées, files, structures de données, notion de page/frame
- Difficulté estimée : 5/10 (base), 7/10 (bonus)
- Cohérent avec phase ? OUI — Phase 2 = 4-6/10

## Aspect Fun/Culture
- Contexte choisi : Squid Game — Élimination selon des règles précises
- MEME mnémotechnique : "Player Elimination" = Page Eviction
- Pourquoi c'est fun : Chaque algorithme = un jeu différent avec ses règles d'élimination

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : LRU qui ne met pas à jour le timestamp lors d'un hit (page considérée non utilisée récemment)
2. Mutant B (Safety) : Clock qui ne reset pas le bit R après passage de l'aiguille
3. Mutant C (Logic) : FIFO qui évince la page la plus récente au lieu de la plus ancienne
4. Mutant D (Return) : Optimal qui retourne toujours la première page au lieu de celle non utilisée le plus longtemps
5. Mutant E (Edge) : Pas de gestion du cas frames > pages (pas de page fault possible)

## Verdict
VALIDE — Exercice complet couvrant 8 concepts du curriculum (2.1.5.a-n)
</thinking>

---

# Exercice 2.1.3 : squid_page_games

**Module :**
2.1.3 — Page Replacement Algorithms

**Concept :**
a-n — FIFO, LRU, Clock, Optimal, Working Set, Thrashing

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
code

**Tiers :**
3 — Synthèse (algorithmes + analyse + détection)

**Langage :**
C17

**Prérequis :**
- Listes chaînées et files (Module 1.x)
- Structures de données (queues, stacks)
- Notion de mémoire virtuelle (ex01, ex02)
- Analyse de complexité

**Domaines :**
Mem, Struct, Algo

**Durée estimée :**
240-300 min

**XP Base :**
500

**Complexité :**
T3 O(n×m) × S2 O(m) où n=taille trace, m=nombre frames

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex03_page_replacement/
├── page_replacement.h
├── pager_fifo.c
├── pager_lru.c
├── pager_clock.c
├── pager_clock_enhanced.c
├── pager_optimal.c
├── pager_lfu.c
├── pager_random.c
├── trace.c
├── working_set.c
└── Makefile
```

**Fonctions autorisées :**
- `malloc`, `free`, `realloc`
- `memset`, `memcpy`, `memmove`
- `open`, `read`, `close`
- `write` (pour debug output uniquement)
- `rand`, `srand`

**Fonctions interdites :**
- `printf`, `fprintf`, `sprintf` (utiliser write)
- `fopen`, `fclose`, `fread`, `fwrite`
- Toute fonction de la libc qui fait de l'allocation cachée

---

### 1.2 Consigne

#### 🎮 Version Culture Pop : "SQUID GAME: Page Elimination"

**"Le Jeu du Calamar : Qui sera éliminé ?"**

Dans l'univers impitoyable de Squid Game, 456 joueurs s'affrontent pour survivre. Chaque jeu a ses propres règles d'élimination. Certaines sont simples (le premier arrivé sera le premier éliminé), d'autres plus complexes (celui qui n'a pas été utile récemment...).

Ta mémoire physique est l'arène. Les pages virtuelles sont les joueurs. Quand l'arène est pleine et qu'un nouveau joueur doit entrer, quelqu'un doit être... **éliminé**.

**Les Jeux disponibles :**

| Jeu | Algorithme | Règle d'élimination |
|-----|------------|---------------------|
| 🔴 Red Light, Green Light | FIFO | Premier entré, premier éliminé |
| 🍬 Dalgona Cookie | LRU | Celui resté inactif le plus longtemps |
| ⏰ Tug of War | Clock | Deuxième chance si récemment actif |
| 🎭 Marbles | Enhanced Clock | Priorité : inactif ET non modifié |
| 🔮 Glass Bridge | Optimal | Celui qui ne sera pas utile le plus longtemps (oracle) |
| 📊 Squid Game Final | LFU | Celui le moins fréquemment utilisé |
| 🎲 VIP Game | Random | Élimination aléatoire (pour le divertissement des VIPs) |

---

#### 📚 Version Académique : Simulateur d'Algorithmes de Remplacement

**Contexte technique :**

Quand la mémoire physique est saturée et qu'une nouvelle page doit être chargée, le système d'exploitation doit choisir une page victime à évincer. Ce choix impacte directement les performances :
- Évincer une page bientôt réutilisée → page fault supplémentaire (coûteux)
- Évincer la bonne page → aucun coût futur

L'algorithme **optimal** (Belady) sait quelle page ne sera pas utilisée le plus longtemps, mais nécessite de connaître le futur — impossible en pratique.

**Le défi :** Approximer l'optimal avec des algorithmes réalisables en temps réel.

---

**Ta mission :**

Créer un simulateur permettant de comparer les 7 algorithmes de remplacement de pages sur différentes traces d'accès mémoire.

**Entrées :**
- `config` : Configuration du simulateur (nombre de frames, nombre de pages)
- `trace` : Séquence d'accès aux pages (numéro de page + type d'accès r/w)

**Sorties :**
- Statistiques par algorithme : page faults, hit rate, writes back

**Contraintes :**
- Chaque algorithme doit avoir la même signature
- Les résultats doivent être reproductibles (seed fixe pour random)
- Mémoire utilisée < 1MB pour 100000 accès
- Temps < 1s pour 100000 accès

---

### 1.3 Prototypes

```c
/* ═══════════════════════════════════════════════════════════════════════════
 * STRUCTURES DE BASE
 * ═══════════════════════════════════════════════════════════════════════════ */

/* Configuration du simulateur */
typedef struct {
    uint32_t num_frames;     /* Nombre de frames physiques disponibles */
    uint32_t num_pages;      /* Nombre total de pages virtuelles */
} pager_config_t;

/* Référence mémoire (élément de la trace) */
typedef struct {
    uint32_t page_number;    /* Numéro de page accédée */
    char     access_type;    /* 'r' (read) ou 'w' (write) */
} page_ref_t;

/* Trace d'accès complète */
typedef struct {
    page_ref_t *refs;        /* Tableau de références */
    size_t      count;       /* Nombre de références */
} trace_t;

/* Résultats de simulation */
typedef struct {
    uint64_t page_faults;    /* Nombre total de page faults */
    uint64_t writes_back;    /* Pages dirty écrites sur disque */
    double   hit_rate;       /* Taux de succès (0.0 - 1.0) */
} pager_result_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * GESTION DES TRACES
 * ═══════════════════════════════════════════════════════════════════════════ */

/* Charger une trace depuis un fichier
 * Format: "page_number access_type\n" par ligne
 * Retourne NULL si erreur */
trace_t *trace_load(const char *filename);

/* Libérer une trace */
void trace_free(trace_t *trace);

/* ═══════════════════════════════════════════════════════════════════════════
 * ALGORITHMES DE REMPLACEMENT
 * Tous ont la même signature pour interchangeabilité
 * ═══════════════════════════════════════════════════════════════════════════ */

typedef pager_result_t (*pager_algorithm_t)(
    const pager_config_t *config,
    const trace_t *trace
);

/* FIFO : First-In-First-Out */
pager_result_t pager_fifo(const pager_config_t *cfg, const trace_t *trace);

/* LRU : Least Recently Used */
pager_result_t pager_lru(const pager_config_t *cfg, const trace_t *trace);

/* Clock : Second Chance */
pager_result_t pager_clock(const pager_config_t *cfg, const trace_t *trace);

/* Enhanced Clock : NRU (Not Recently Used) */
pager_result_t pager_clock_enhanced(const pager_config_t *cfg, const trace_t *trace);

/* Optimal : Belady (nécessite connaissance du futur) */
pager_result_t pager_optimal(const pager_config_t *cfg, const trace_t *trace);

/* LFU : Least Frequently Used */
pager_result_t pager_lfu(const pager_config_t *cfg, const trace_t *trace);

/* Random : Élimination aléatoire */
pager_result_t pager_random(const pager_config_t *cfg, const trace_t *trace);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Anomalie de Belady : Plus de mémoire = Plus de page faults ?!

En 1969, László Bélády découvre un phénomène contre-intuitif : avec l'algorithme FIFO, **augmenter le nombre de frames peut augmenter le nombre de page faults**.

```
Séquence : 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

Avec 3 frames : 9 page faults
Avec 4 frames : 10 page faults  ← Plus de mémoire, plus de faults !
Avec 5 frames : 5 page faults
```

Cette anomalie ne se produit **jamais** avec LRU ou les algorithmes "stack" (qui respectent la propriété d'inclusion).

### 2.2 Pourquoi LRU n'est pas utilisé tel quel en hardware ?

LRU parfait nécessite de :
1. Mettre à jour un compteur/timestamp à chaque accès mémoire
2. Trouver le minimum parmi toutes les pages

Avec des millions d'accès par seconde, c'est trop coûteux ! D'où les **approximations** :
- **Clock** : Un seul bit (R) par page, parcours circulaire
- **Aging** : Registre de 8 bits décalé à droite périodiquement

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation | Cas d'usage |
|--------|-------------|-------------|
| **Kernel Developer** | Implémentation dans Linux/Windows | `mm/vmscan.c` dans Linux utilise une variante de Clock |
| **Database Admin** | Buffer pool management | MySQL/PostgreSQL utilisent LRU modifié pour le cache |
| **Cloud Engineer** | Overcommit et balloon drivers | VMware/KVM utilisent working set pour le memory ballooning |
| **Game Developer** | Streaming de textures | Gestion du VRAM limité avec prédiction d'usage |
| **Embedded Systems** | Systèmes temps-réel | Algorithmes déterministes pour garantir les deadlines |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
page_replacement.h  pager_fifo.c  pager_lru.c  pager_clock.c
pager_optimal.c  trace.c  main.c  workload1.trace  Makefile

$ make

$ ./pager_test workload1.trace 4
Algorithm       | Page Faults | Hit Rate  | Writes Back
----------------|-------------|-----------|------------
Optimal         |           6 | 70.00%    |           2
LRU             |           8 | 60.00%    |           3
Clock           |           9 | 55.00%    |           3
FIFO            |          10 | 50.00%    |           4
LFU             |           9 | 55.00%    |           3
Random          |          11 | 45.00%    |           5

$ ./pager_test --belady-demo
Demonstrating Belady's Anomaly with FIFO:
FIFO with 3 frames: 9 page faults
FIFO with 4 frames: 10 page faults  <- ANOMALY!
FIFO with 5 frames: 5 page faults
```

---

### 3.1 🔥 BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(n) pour l'analyse working set

**Space Complexity attendue :**
O(w) où w = taille de la fenêtre

**Domaines Bonus :**
`Probas, Algo`

#### 3.1.1 Consigne Bonus

**🎮 "Le Front Man analyse le comportement des joueurs"**

Le Front Man a besoin de prédire quand les joueurs vont craquer sous la pression. Si trop de joueurs sont éliminés trop rapidement, le spectacle perd son intérêt (thrashing). Il doit analyser le "working set" — le groupe de joueurs actifs à un moment donné.

**Ta mission :**

Implémenter la détection de thrashing et l'analyse du working set.

```c
/* Statistiques de working set */
typedef struct {
    double avg_working_set;      /* Taille moyenne du working set */
    double max_working_set;      /* Taille max observée */
    int    thrashing_detected;   /* 1 si thrashing détecté */
    double thrashing_severity;   /* 0.0 (aucun) à 1.0 (sévère) */
} working_set_stats_t;

/* Analyse du working set avec fenêtre temporelle */
working_set_stats_t analyze_working_set(
    const trace_t *trace,
    uint32_t window_size
);

/* Page Fault Frequency analysis */
typedef struct {
    double pff_avg;              /* PFF moyen */
    double pff_max;              /* PFF max */
    uint32_t recommended_frames; /* Frames recommandées */
} pff_analysis_t;

pff_analysis_t analyze_pff(
    const trace_t *trace,
    uint32_t num_frames
);
```

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  window_size ≥ 1                        │
│  Thrashing si PFF > 50%                 │
│  Severity = (PFF - 50%) / 50%           │
│  Recommandation = avg_working_set × 1.2 │
└─────────────────────────────────────────┘
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette — Tests automatisés

| Test | Description | Entrée | Attendu | Points |
|------|-------------|--------|---------|--------|
| `test_fifo_basic` | FIFO basique | `1,2,3,1,4` frames=3 | 4 faults | 5 |
| `test_lru_basic` | LRU basique | `1,2,3,1,4` frames=3 | 4 faults | 5 |
| `test_optimal_basic` | Optimal meilleur | Trace standard | optimal ≤ lru ≤ fifo | 10 |
| `test_belady_anomaly` | Détection anomalie | Séquence Belady | fifo(4) > fifo(3) | 10 |
| `test_clock_approx` | Clock ≈ LRU | 1000 accès random | diff < 10% | 10 |
| `test_null_trace` | Trace NULL | NULL | 0 faults | 5 |
| `test_null_config` | Config NULL | NULL | 0 faults | 5 |
| `test_zero_frames` | 0 frames | 0 | Erreur gérée | 5 |
| `test_large_trace` | 100000 accès | Random | < 1 seconde | 15 |
| `test_memory_usage` | Mémoire | 100000 accès | < 1 MB | 10 |
| `test_thrashing` | Détection thrashing | Workload pathologique | detected=1 | 10 |
| `test_working_set` | Calcul working set | Locality pattern | avg ≈ 5 | 10 |

---

### 4.2 main.c de test

```c
#include "page_replacement.h"
#include <assert.h>
#include <string.h>

/* Helper pour créer une trace inline */
static trace_t *create_trace(uint32_t *pages, size_t count)
{
    trace_t *t = malloc(sizeof(trace_t));
    if (!t) return NULL;

    t->refs = malloc(count * sizeof(page_ref_t));
    if (!t->refs) { free(t); return NULL; }

    t->count = count;
    for (size_t i = 0; i < count; i++) {
        t->refs[i].page_number = pages[i];
        t->refs[i].access_type = 'r';
    }
    return t;
}

static void test_fifo_basic(void)
{
    uint32_t pages[] = {1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5};
    trace_t *trace = create_trace(pages, 12);
    pager_config_t cfg = {.num_frames = 3, .num_pages = 6};

    pager_result_t r = pager_fifo(&cfg, trace);
    assert(r.page_faults == 9);

    trace_free(trace);
}

static void test_belady_anomaly(void)
{
    uint32_t pages[] = {1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5};
    trace_t *trace = create_trace(pages, 12);

    pager_config_t cfg3 = {.num_frames = 3, .num_pages = 6};
    pager_config_t cfg4 = {.num_frames = 4, .num_pages = 6};

    pager_result_t r3 = pager_fifo(&cfg3, trace);
    pager_result_t r4 = pager_fifo(&cfg4, trace);

    /* Belady's anomaly: more frames = more faults! */
    assert(r4.page_faults > r3.page_faults);

    trace_free(trace);
}

static void test_optimal_best(void)
{
    uint32_t pages[] = {1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5};
    trace_t *trace = create_trace(pages, 12);
    pager_config_t cfg = {.num_frames = 3, .num_pages = 6};

    pager_result_t opt = pager_optimal(&cfg, trace);
    pager_result_t lru = pager_lru(&cfg, trace);
    pager_result_t fifo = pager_fifo(&cfg, trace);

    assert(opt.page_faults <= lru.page_faults);
    assert(lru.page_faults <= fifo.page_faults);

    trace_free(trace);
}

static void test_null_safety(void)
{
    pager_config_t cfg = {.num_frames = 4, .num_pages = 10};

    pager_result_t r1 = pager_fifo(&cfg, NULL);
    assert(r1.page_faults == 0);

    pager_result_t r2 = pager_fifo(NULL, NULL);
    assert(r2.page_faults == 0);
}

int main(void)
{
    test_fifo_basic();
    test_belady_anomaly();
    test_optimal_best();
    test_null_safety();

    write(1, "All tests passed!\n", 18);
    return 0;
}
```

---

### 4.3 Solution de référence — pager_fifo.c

```c
#include "page_replacement.h"
#include <stdlib.h>
#include <string.h>

/* Structure interne pour FIFO */
typedef struct {
    uint32_t *frames;        /* Pages dans chaque frame */
    int      *valid;         /* Frame occupée ? */
    int      *dirty;         /* Page modifiée ? */
    size_t    head;          /* Index du plus ancien */
    size_t    count;         /* Frames occupées */
    size_t    capacity;      /* Nombre total de frames */
} fifo_state_t;

static fifo_state_t *fifo_init(size_t num_frames)
{
    fifo_state_t *state = malloc(sizeof(fifo_state_t));
    if (!state)
        return (NULL);

    state->frames = malloc(num_frames * sizeof(uint32_t));
    state->valid = calloc(num_frames, sizeof(int));
    state->dirty = calloc(num_frames, sizeof(int));

    if (!state->frames || !state->valid || !state->dirty)
    {
        free(state->frames);
        free(state->valid);
        free(state->dirty);
        free(state);
        return (NULL);
    }

    state->head = 0;
    state->count = 0;
    state->capacity = num_frames;
    return (state);
}

static void fifo_destroy(fifo_state_t *state)
{
    if (!state)
        return;
    free(state->frames);
    free(state->valid);
    free(state->dirty);
    free(state);
}

static int fifo_find(fifo_state_t *state, uint32_t page)
{
    for (size_t i = 0; i < state->capacity; i++)
    {
        if (state->valid[i] && state->frames[i] == page)
            return ((int)i);
    }
    return (-1);
}

pager_result_t pager_fifo(const pager_config_t *cfg, const trace_t *trace)
{
    pager_result_t result = {0, 0, 0.0};

    /* Safety checks */
    if (!cfg || !trace || !trace->refs || trace->count == 0)
        return (result);
    if (cfg->num_frames == 0)
        return (result);

    fifo_state_t *state = fifo_init(cfg->num_frames);
    if (!state)
        return (result);

    uint64_t hits = 0;

    for (size_t i = 0; i < trace->count; i++)
    {
        uint32_t page = trace->refs[i].page_number;
        char access = trace->refs[i].access_type;

        int frame = fifo_find(state, page);

        if (frame >= 0)
        {
            /* Hit */
            hits++;
            if (access == 'w')
                state->dirty[frame] = 1;
        }
        else
        {
            /* Page fault */
            result.page_faults++;

            if (state->count < state->capacity)
            {
                /* Espace libre disponible */
                for (size_t j = 0; j < state->capacity; j++)
                {
                    if (!state->valid[j])
                    {
                        state->frames[j] = page;
                        state->valid[j] = 1;
                        state->dirty[j] = (access == 'w');
                        state->count++;
                        break;
                    }
                }
            }
            else
            {
                /* Éviction FIFO */
                if (state->dirty[state->head])
                    result.writes_back++;

                state->frames[state->head] = page;
                state->dirty[state->head] = (access == 'w');
                state->head = (state->head + 1) % state->capacity;
            }
        }
    }

    result.hit_rate = (double)hits / trace->count;
    fifo_destroy(state);
    return (result);
}
```

---

### 4.4 Solutions alternatives acceptées

#### Alternative 1 : FIFO avec liste chaînée

```c
typedef struct fifo_node {
    uint32_t page;
    int dirty;
    struct fifo_node *next;
} fifo_node_t;

typedef struct {
    fifo_node_t *head;
    fifo_node_t *tail;
    size_t count;
    size_t capacity;
} fifo_queue_t;

/* Implémentation avec vraie file FIFO */
pager_result_t pager_fifo_linked(const pager_config_t *cfg, const trace_t *trace)
{
    /* ... utilise une vraie liste chaînée ... */
}
```

#### Alternative 2 : Utilisation de tableau circulaire simplifié

```c
pager_result_t pager_fifo_simple(const pager_config_t *cfg, const trace_t *trace)
{
    pager_result_t r = {0};
    if (!cfg || !trace) return r;

    uint32_t *frames = calloc(cfg->num_frames, sizeof(uint32_t));
    int *present = calloc(cfg->num_frames, sizeof(int));
    size_t next_victim = 0;
    size_t loaded = 0;

    for (size_t i = 0; i < trace->count; i++)
    {
        uint32_t pg = trace->refs[i].page_number;
        int found = 0;

        for (size_t j = 0; j < cfg->num_frames && !found; j++)
            if (present[j] && frames[j] == pg) found = 1;

        if (!found)
        {
            r.page_faults++;
            if (loaded < cfg->num_frames)
            {
                frames[loaded] = pg;
                present[loaded++] = 1;
            }
            else
            {
                frames[next_victim] = pg;
                next_victim = (next_victim + 1) % cfg->num_frames;
            }
        }
    }

    r.hit_rate = 1.0 - (double)r.page_faults / trace->count;
    free(frames); free(present);
    return r;
}
```

---

### 4.5 Solutions refusées

#### ❌ Refusée 1 : Pas de gestion NULL

```c
pager_result_t pager_fifo_bad(const pager_config_t *cfg, const trace_t *trace)
{
    /* ERREUR : Pas de vérification NULL */
    fifo_state_t *state = fifo_init(cfg->num_frames);  /* CRASH si cfg NULL */

    for (size_t i = 0; i < trace->count; i++)  /* CRASH si trace NULL */
    {
        /* ... */
    }
}
/* Pourquoi refusé : Segfault garanti sur entrées invalides */
```

#### ❌ Refusée 2 : LIFO au lieu de FIFO

```c
pager_result_t pager_fifo_wrong(const pager_config_t *cfg, const trace_t *trace)
{
    /* ERREUR : Évince le dernier entré, pas le premier */
    /* C'est LIFO, pas FIFO ! */
    size_t victim = (state->head + state->count - 1) % state->capacity;
    /* ... */
}
/* Pourquoi refusé : Algorithme incorrect, comportement différent */
```

---

### 4.6 Solution bonus de référence — working_set.c

```c
#include "page_replacement.h"
#include <stdlib.h>

working_set_stats_t analyze_working_set(
    const trace_t *trace,
    uint32_t window_size)
{
    working_set_stats_t stats = {0.0, 0.0, 0, 0.0};

    if (!trace || !trace->refs || trace->count == 0 || window_size == 0)
        return (stats);

    /* Bitmap pour tracking des pages uniques dans la fenêtre */
    size_t max_page = 0;
    for (size_t i = 0; i < trace->count; i++)
        if (trace->refs[i].page_number > max_page)
            max_page = trace->refs[i].page_number;

    int *in_window = calloc(max_page + 1, sizeof(int));
    if (!in_window)
        return (stats);

    double total_ws = 0.0;
    size_t measurements = 0;

    for (size_t i = 0; i < trace->count; i++)
    {
        /* Reset window tracking */
        for (size_t p = 0; p <= max_page; p++)
            in_window[p] = 0;

        /* Calcul working set pour fenêtre [i, min(i+window, count)] */
        size_t end = (i + window_size < trace->count) ?
                      i + window_size : trace->count;

        size_t ws_size = 0;
        for (size_t j = i; j < end; j++)
        {
            uint32_t pg = trace->refs[j].page_number;
            if (!in_window[pg])
            {
                in_window[pg] = 1;
                ws_size++;
            }
        }

        total_ws += ws_size;
        measurements++;

        if (ws_size > stats.max_working_set)
            stats.max_working_set = ws_size;
    }

    stats.avg_working_set = total_ws / measurements;
    free(in_window);

    return (stats);
}

pff_analysis_t analyze_pff(const trace_t *trace, uint32_t num_frames)
{
    pff_analysis_t pff = {0.0, 0.0, 0};

    if (!trace || !trace->refs || trace->count == 0 || num_frames == 0)
        return (pff);

    /* Simuler et compter les page faults par intervalle */
    pager_config_t cfg = {.num_frames = num_frames, .num_pages = 1000};
    pager_result_t res = pager_fifo(&cfg, trace);

    pff.pff_avg = (double)res.page_faults / trace->count;
    pff.pff_max = pff.pff_avg * 1.5;  /* Estimation */

    /* Recommandation basée sur working set */
    working_set_stats_t ws = analyze_working_set(trace, 100);
    pff.recommended_frames = (uint32_t)(ws.avg_working_set * 1.2);

    if (pff.recommended_frames < 1)
        pff.recommended_frames = 1;

    return (pff);
}
```

---

### 4.9 spec.json

```json
{
  "name": "squid_page_games",
  "language": "c",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (7 algorithmes + analyse)",
  "tags": ["memory", "page-replacement", "algorithms", "phase2"],
  "passing_score": 70,

  "function": {
    "name": "pager_fifo",
    "prototype": "pager_result_t pager_fifo(const pager_config_t *cfg, const trace_t *trace)",
    "return_type": "pager_result_t",
    "parameters": [
      {"name": "cfg", "type": "const pager_config_t *"},
      {"name": "trace", "type": "const trace_t *"}
    ]
  },

  "driver": {
    "reference": "pager_result_t ref_pager_fifo(const pager_config_t *cfg, const trace_t *trace) { pager_result_t r = {0}; if (!cfg || !trace || !trace->refs || cfg->num_frames == 0) return r; uint32_t *frames = calloc(cfg->num_frames, sizeof(uint32_t)); int *valid = calloc(cfg->num_frames, sizeof(int)); size_t head = 0, count = 0; for (size_t i = 0; i < trace->count; i++) { uint32_t pg = trace->refs[i].page_number; int found = 0; for (size_t j = 0; j < cfg->num_frames && !found; j++) if (valid[j] && frames[j] == pg) found = 1; if (!found) { r.page_faults++; if (count < cfg->num_frames) { frames[count] = pg; valid[count++] = 1; } else { frames[head] = pg; head = (head + 1) % cfg->num_frames; } } } r.hit_rate = 1.0 - (double)r.page_faults / trace->count; free(frames); free(valid); return r; }",

    "edge_cases": [
      {
        "name": "null_config",
        "args": [null, "valid_trace"],
        "expected": {"page_faults": 0, "hit_rate": 0.0},
        "is_trap": true,
        "trap_explanation": "cfg NULL doit retourner résultat vide"
      },
      {
        "name": "null_trace",
        "args": ["valid_config", null],
        "expected": {"page_faults": 0},
        "is_trap": true,
        "trap_explanation": "trace NULL doit retourner résultat vide"
      },
      {
        "name": "zero_frames",
        "args": [{"num_frames": 0}, "valid_trace"],
        "expected": {"page_faults": 0},
        "is_trap": true,
        "trap_explanation": "0 frames = pas d'allocation possible"
      },
      {
        "name": "empty_trace",
        "args": ["valid_config", {"refs": [], "count": 0}],
        "expected": {"page_faults": 0, "hit_rate": 0.0},
        "is_trap": true
      },
      {
        "name": "all_hits",
        "args": [{"num_frames": 3}, [1,1,1,1,1]],
        "expected": {"page_faults": 1, "hit_rate": 0.8}
      },
      {
        "name": "all_misses",
        "args": [{"num_frames": 1}, [1,2,3,4,5]],
        "expected": {"page_faults": 5, "hit_rate": 0.0}
      },
      {
        "name": "belady_sequence",
        "args": [{"num_frames": 4}, [1,2,3,4,1,2,5,1,2,3,4,5]],
        "expected": {"page_faults": 10}
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "array_int",
          "param_index": 1,
          "params": {
            "min_len": 1,
            "max_len": 1000,
            "min_val": 0,
            "max_val": 100
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["malloc", "free", "realloc", "calloc", "memset", "memcpy", "write", "rand", "srand"],
    "forbidden_functions": ["printf", "fprintf", "fopen", "fread"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

---

### 4.10 Solutions Mutantes

#### Mutant A (Boundary) : Oublie de mettre à jour head

```c
pager_result_t pager_fifo_mutant_a(const pager_config_t *cfg, const trace_t *trace)
{
    /* ... setup ... */

    /* BUG : head n'est jamais incrémenté ! */
    frames[state->head] = page;  /* Toujours écrase la même frame */
    /* state->head = (state->head + 1) % state->capacity;  MANQUANT */

    /* ... */
}
/* Pourquoi faux : Évince toujours la frame 0, pas FIFO */
/* Pensée erronée : "head pointe vers la victime, pas besoin de bouger" */
```

#### Mutant B (Safety) : Pas de vérification trace vide

```c
pager_result_t pager_fifo_mutant_b(const pager_config_t *cfg, const trace_t *trace)
{
    if (!cfg || !trace)
        return ((pager_result_t){0});

    /* BUG : Ne vérifie pas trace->count == 0 */
    for (size_t i = 0; i < trace->count; i++)  /* OK si count=0, mais... */
    {
        uint32_t page = trace->refs[i].page_number;  /* CRASH si refs=NULL */
    }
}
/* Pourquoi faux : Crash si trace->refs est NULL mais trace ne l'est pas */
```

#### Mutant C (Logic) : Retourne hits au lieu de faults

```c
pager_result_t pager_fifo_mutant_c(const pager_config_t *cfg, const trace_t *trace)
{
    uint64_t hits = 0;
    uint64_t faults = 0;

    /* ... boucle ... */

    /* BUG : Retourne hits au lieu de faults */
    result.page_faults = hits;  /* INVERSÉ ! */
    result.hit_rate = (double)faults / trace->count;  /* INVERSÉ ! */

    return (result);
}
/* Pourquoi faux : Métriques inversées */
```

#### Mutant D (Return) : Ne compte pas le premier chargement

```c
pager_result_t pager_fifo_mutant_d(const pager_config_t *cfg, const trace_t *trace)
{
    /* ... */

    if (state->count < state->capacity)
    {
        /* BUG : Pas de page_faults++ ici */
        /* Pense que le chargement initial n'est pas un fault */
        frames[loaded++] = page;
    }
    else
    {
        result.page_faults++;  /* Compte seulement les évictions */
        /* ... */
    }
}
/* Pourquoi faux : Les cold misses sont aussi des page faults ! */
/* Pensée erronée : "Fault = éviction, pas chargement" */
```

#### Mutant E (Edge) : Division par zéro dans hit_rate

```c
pager_result_t pager_fifo_mutant_e(const pager_config_t *cfg, const trace_t *trace)
{
    /* ... */

    /* BUG : Pas de vérification trace->count == 0 */
    result.hit_rate = (double)hits / trace->count;  /* DIVISION PAR ZÉRO */

    return (result);
}
/* Pourquoi faux : NaN ou crash si trace vide */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Référence | Maîtrise attendue |
|---------|-----------|-------------------|
| Page replacement | 2.1.5.a | Savoir pourquoi on doit choisir une victime |
| Optimal (Belady) | 2.1.5.b | Comprendre la borne théorique |
| FIFO | 2.1.5.c | Implémenter, connaître l'anomalie |
| Belady's anomaly | 2.1.5.d | Démontrer le phénomène |
| LRU | 2.1.5.e-g | Implémenter avec O(1) access |
| Clock | 2.1.5.h-i | Approximation efficace de LRU |
| Working set | 2.1.5.j-k | Calculer et interpréter |
| PFF | 2.1.5.l | Analyser la fréquence de faults |
| Thrashing | 2.1.5.m-n | Détecter et prévenir |

---

### 5.2 LDA — Langage de Description d'Algorithmes

```
FONCTION pager_fifo QUI RETOURNE UNE STRUCTURE pager_result_t ET PREND EN PARAMÈTRES cfg QUI EST UN POINTEUR VERS UNE STRUCTURE pager_config_t CONSTANTE ET trace QUI EST UN POINTEUR VERS UNE STRUCTURE trace_t CONSTANTE
DÉBUT FONCTION
    DÉCLARER result COMME STRUCTURE pager_result_t
    INITIALISER result AVEC ZÉROS

    SI cfg EST ÉGAL À NUL OU trace EST ÉGAL À NUL ALORS
        RETOURNER result
    FIN SI

    DÉCLARER frames COMME TABLEAU DE ENTIERS NON SIGNÉS
    DÉCLARER head COMME ENTIER (index du plus ancien)
    AFFECTER 0 À head

    POUR i ALLANT DE 0 À count MOINS 1 FAIRE
        DÉCLARER page COMME LE NUMÉRO DE PAGE À LA POSITION i

        SI page EST PRÉSENTE DANS frames ALORS
            INCRÉMENTER hits DE 1
        SINON
            INCRÉMENTER page_faults DE 1

            SI frames N'EST PAS PLEIN ALORS
                AJOUTER page DANS LA PREMIÈRE POSITION LIBRE
            SINON
                REMPLACER LA PAGE À LA POSITION head PAR page
                AFFECTER head PLUS 1 MODULO capacité À head
            FIN SI
        FIN SI
    FIN POUR

    AFFECTER hits DIVISÉ PAR count À hit_rate
    RETOURNER result
FIN FONCTION
```

---

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : FIFO Page Replacement
---
1. INITIALISER le résultat avec zéros

2. VÉRIFIER les paramètres :
   |-- SI config OU trace est NULL → RETOURNER résultat vide
   |-- SI num_frames == 0 → RETOURNER résultat vide

3. CRÉER l'état FIFO :
   |-- Tableau frames[num_frames]
   |-- Index head = 0 (pointe vers le plus ancien)
   |-- Compteur count = 0 (frames occupées)

4. POUR CHAQUE référence dans la trace :
   a. CHERCHER la page dans frames

   b. SI trouvée (HIT) :
      |-- Incrémenter hits
      |-- SI write access → marquer dirty

   c. SINON (PAGE FAULT) :
      |-- Incrémenter page_faults
      |-- SI count < capacity (espace libre) :
           |-- Charger dans première frame libre
           |-- Incrémenter count
      |-- SINON (éviction nécessaire) :
           |-- SI frame[head] est dirty → writes_back++
           |-- Remplacer frame[head] par nouvelle page
           |-- head = (head + 1) % capacity

5. CALCULER hit_rate = hits / total_refs

6. RETOURNER résultat
```

---

### 5.3 Visualisation ASCII

#### Architecture FIFO (Queue Circulaire)

```
                    FIFO QUEUE (4 frames)

    HEAD                                    TAIL
      ↓                                       ↓
   ┌──────┬──────┬──────┬──────┐
   │ P1   │ P2   │ P3   │ P4   │  ← Frames physiques
   └──────┴──────┴──────┴──────┘
      0      1      2      3
      ↑
   Victime (premier entré = premier sorti)


Accès P5 (P1 éjectée) :
   ┌──────┬──────┬──────┬──────┐
   │ P5   │ P2   │ P3   │ P4   │
   └──────┴──────┴──────┴──────┘
      0      1      2      3
             ↑
          HEAD avance
```

#### Algorithme Clock (Seconde Chance)

```
                    CLOCK ALGORITHM

              ┌──────┐
              │  P1  │ R=1
              │      │
        ┌─────┴──────┴─────┐
        │                  │
     ┌──┴──┐            ┌──┴──┐
     │ P4  │ R=0        │ P2  │ R=1
     └──┬──┘            └──┬──┘
        │   ← AIGUILLE     │
        │      ↓           │
        └──────┬───────────┘
               │
            ┌──┴──┐
            │ P3  │ R=0  ← Victime ! (R=0)
            └─────┘

Parcours :
1. P1: R=1 → R=0, avancer
2. P4: R=0 → ÉVINCER !

OU

1. P1: R=1 → R=0, avancer
2. P2: R=1 → R=0, avancer
3. P3: R=0 → ÉVINCER !
```

#### Belady's Anomaly Visualisation

```
Séquence : 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

AVEC 3 FRAMES:
┌───┬───┬───┐
│ 1 │   │   │  F  (1 fault)
├───┼───┼───┤
│ 1 │ 2 │   │  F  (2 faults)
├───┼───┼───┤
│ 1 │ 2 │ 3 │  F  (3 faults)
├───┼───┼───┤
│ 4 │ 2 │ 3 │  F  (4, évince 1)
├───┼───┼───┤
│ 4 │ 1 │ 3 │  F  (5, évince 2)
├───┼───┼───┤
│ 4 │ 1 │ 2 │  F  (6, évince 3)
├───┼───┼───┤
│ 5 │ 1 │ 2 │  F  (7, évince 4)
├───┼───┼───┤
│ 5 │ 1 │ 2 │  H  (hit sur 1)
├───┼───┼───┤
│ 5 │ 1 │ 2 │  H  (hit sur 2)
├───┼───┼───┤
│ 5 │ 3 │ 2 │  F  (8, évince 1)
├───┼───┼───┤
│ 5 │ 3 │ 4 │  F  (9, évince 2)
└───┴───┴───┘
Total: 9 faults

AVEC 4 FRAMES:
┌───┬───┬───┬───┐
│ 1 │ 2 │ 3 │ 4 │  4 faults (chargement initial)
├───┼───┼───┼───┤
│ 1 │ 2 │ 3 │ 4 │  H, H  (hits sur 1, 2)
├───┼───┼───┼───┤
│ 5 │ 2 │ 3 │ 4 │  F  (évince 1)
├───┼───┼───┼───┤
│ 5 │ 1 │ 3 │ 4 │  F  (évince 2)
├───┼───┼───┼───┤
│ 5 │ 1 │ 2 │ 4 │  F  (évince 3)
├───┼───┼───┼───┤
│ 5 │ 1 │ 2 │ 3 │  F  (évince 4)
├───┼───┼───┼───┤
│ 4 │ 1 │ 2 │ 3 │  F  (évince 5)
├───┼───┼───┼───┤
│ 4 │ 5 │ 2 │ 3 │  F  (évince 1)
└───┴───┴───┴───┘
Total: 10 faults  ← PLUS que 3 frames !

ANOMALIE DE BELADY : 4 frames > 3 frames en page faults !
```

---

### 5.4 Les pièges en détail

#### Piège 1 : Confondre "premier entré" et "dernier utilisé"

```c
/* ❌ FAUX : C'est LRU, pas FIFO ! */
size_t find_oldest_access(state_t *s) {
    size_t oldest_idx = 0;
    uint64_t oldest_time = s->last_access[0];
    for (size_t i = 1; i < s->count; i++) {
        if (s->last_access[i] < oldest_time) {
            oldest_time = s->last_access[i];
            oldest_idx = i;
        }
    }
    return oldest_idx;
}

/* ✅ CORRECT : FIFO utilise l'ordre d'entrée, pas d'accès */
/* Juste un index circulaire qui avance */
```

#### Piège 2 : Ne pas compter les cold misses

```c
/* ❌ FAUX : Cold miss = page fault aussi ! */
if (count < capacity) {
    /* Juste charger, pas compter comme fault */
    frames[count++] = page;
}

/* ✅ CORRECT */
if (count < capacity) {
    result.page_faults++;  /* Cold miss = fault */
    frames[count++] = page;
}
```

#### Piège 3 : Clock qui ne reset pas R

```c
/* ❌ FAUX : R doit être mis à 0 après passage */
while (frames[hand].R == 1) {
    hand = (hand + 1) % capacity;  /* Avance sans reset ! */
}

/* ✅ CORRECT */
while (frames[hand].R == 1) {
    frames[hand].R = 0;  /* Reset le bit */
    hand = (hand + 1) % capacity;
}
```

---

### 5.5 Cours Complet

#### 5.5.1 Pourquoi le remplacement de pages ?

La mémoire physique (RAM) est limitée. Quand un processus accède à une page qui n'est pas en mémoire :

1. **Page Fault** : Interruption au CPU
2. **Le kernel** doit charger la page depuis le disque
3. **Si la RAM est pleine** : il faut d'abord libérer une frame

**Le coût d'un page fault :**
- Accès RAM : ~100 ns
- Accès SSD : ~100 µs (1000× plus lent)
- Accès HDD : ~10 ms (100,000× plus lent)

→ Minimiser les page faults est **critique** pour les performances.

#### 5.5.2 Les Algorithmes

**FIFO (First-In, First-Out)**
```
Principe : Évincer la page qui est en mémoire depuis le plus longtemps
Structure : Queue simple
Complexité : O(1) insertion, O(1) éviction
Problème : Belady's anomaly possible
```

**LRU (Least Recently Used)**
```
Principe : Évincer la page utilisée il y a le plus longtemps
Intuition : "Si pas utilisée récemment, probablement pas bientôt"
Structure : Liste doublement chaînée + HashMap pour O(1)
Avantage : Bon comportement général, pas d'anomalie
Inconvénient : Coûteux en hardware (update à chaque accès)
```

**Clock (Second Chance)**
```
Principe : FIFO amélioré avec bit de référence R
Algorithme :
  1. Parcourir circulairement les frames
  2. Si R=1 : mettre R=0, passer à la suivante (seconde chance)
  3. Si R=0 : évincer cette page
Avantage : Approxime LRU avec coût proche de FIFO
```

**Enhanced Clock (NRU)**
```
Considère R (référence) et M (modifié) :
Classe 0: R=0, M=0 → Meilleur candidat (pas utilisé, pas modifié)
Classe 1: R=0, M=1 → Pas utilisé, mais dirty (write-back nécessaire)
Classe 2: R=1, M=0 → Utilisé récemment, propre
Classe 3: R=1, M=1 → Pire candidat (utilisé et dirty)

On cherche d'abord Classe 0, puis 1, puis 2, puis 3.
```

**Optimal (Belady)**
```
Principe : Évincer la page qui ne sera pas utilisée le plus longtemps
Problème : Nécessite de connaître le futur → impossible en pratique
Usage : Benchmark théorique pour comparer les autres algorithmes
```

#### 5.5.3 Thrashing

**Définition :** Le système passe plus de temps à gérer les page faults qu'à exécuter du code utile.

**Cause :** Working set > mémoire physique disponible

**Symptômes :**
- CPU utilization chute
- Disk I/O explose
- Système devient non-responsive

**Solutions :**
1. Ajouter de la RAM
2. Réduire le nombre de processus
3. Utiliser le Page Fault Frequency (PFF) pour ajuster l'allocation

---

### 5.6 Normes avec explications

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ for(size_t i=0;i<trace->count;i++)                              │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ for (size_t i = 0; i < trace->count; i++)                       │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ • Espaces autour des opérateurs = lisibilité                    │
│ • Espace après for = distingue des appels de fonction           │
└─────────────────────────────────────────────────────────────────┘
```

---

### 5.7 Simulation avec trace d'exécution

**Entrée :** Trace = [1, 2, 3, 1, 4], Frames = 3

```
┌───────┬─────────────────────────┬────────────────┬─────────┬────────────────────┐
│ Étape │ Page demandée           │ Frames         │ Fault?  │ Explication        │
├───────┼─────────────────────────┼────────────────┼─────────┼────────────────────┤
│   1   │ 1                       │ [1, _, _]      │   F     │ Cold miss          │
├───────┼─────────────────────────┼────────────────┼─────────┼────────────────────┤
│   2   │ 2                       │ [1, 2, _]      │   F     │ Cold miss          │
├───────┼─────────────────────────┼────────────────┼─────────┼────────────────────┤
│   3   │ 3                       │ [1, 2, 3]      │   F     │ Cold miss          │
├───────┼─────────────────────────┼────────────────┼─────────┼────────────────────┤
│   4   │ 1                       │ [1, 2, 3]      │   H     │ Page 1 présente    │
├───────┼─────────────────────────┼────────────────┼─────────┼────────────────────┤
│   5   │ 4                       │ [4, 2, 3]      │   F     │ Évince 1 (FIFO)    │
└───────┴─────────────────────────┴────────────────┴─────────┴────────────────────┘

Résultat : 4 page faults, hit_rate = 1/5 = 20%
```

---

### 5.8 Mnémotechniques

#### 🦑 MEME : "Red Light, Green Light" — FIFO

```
Dans Squid Game, le premier joueur à entrer dans le jeu
est le premier à risquer l'élimination.

FIFO = Premier Entré, Premier Sorti
     = First In, First Out
     = Red Light, tu étais là en premier, tu pars en premier
```

#### 🎰 MEME : "Les VIPs veulent du spectacle" — Random

```
Les VIPs de Squid Game veulent du divertissement imprévisible.
L'algorithme Random = élimination au hasard pour le spectacle.

Pas optimal, mais simple et... divertissant ?
(Spoiler : terrible pour les performances)
```

#### ⏰ MEME : "Seconde Chance" — Clock

```
"J'ai été utile récemment !" (R=1)
→ OK, une seconde chance, mais R passe à 0

"Je... je n'ai rien fait..." (R=0)
→ Éliminé ! Pas de seconde chance pour les inactifs.

Clock = L'algorithme de la seconde chance
```

---

### 5.9 Applications pratiques

| Système | Algorithme utilisé | Raison |
|---------|-------------------|--------|
| Linux | Clock variant (LRU approx) | Bon compromis performance/coût |
| Windows | Working Set + Clock | Adaptation dynamique |
| FreeBSD | 2Q (Two Queue) | Distingue pages fréquentes/occasionnelles |
| MySQL InnoDB | LRU modifié | Optimisé pour les accès DB |
| Redis | LRU sampling | Échantillonnage pour éviter O(n) |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Conséquence | Solution |
|---|-------|-------------|----------|
| 1 | Confondre FIFO/LRU | Mauvais algorithme | FIFO = ordre entrée, LRU = ordre accès |
| 2 | Oublier cold misses | Sous-compte les faults | Premier chargement = fault |
| 3 | Clock sans reset R | Boucle infinie | Toujours R=0 après passage |
| 4 | Division par 0 | NaN/crash | Vérifier count > 0 |
| 5 | Pas de NULL check | Segfault | Vérifier cfg, trace, refs |

---

## 📝 SECTION 7 : QCM

### Question 1
**Quel algorithme garantit le minimum de page faults ?**

A) FIFO
B) LRU
C) Clock
D) Optimal (Belady)
E) Random
F) LFU
G) Enhanced Clock
H) Aucun ne garantit
I) Tous sont équivalents
J) Dépend de la trace

**Réponse : D**

---

### Question 2
**L'anomalie de Belady se produit avec quel algorithme ?**

A) LRU uniquement
B) FIFO uniquement
C) Clock uniquement
D) Tous les algorithmes
E) Aucun algorithme
F) LRU et Clock
G) FIFO et LFU
H) Random
I) Optimal
J) Enhanced Clock

**Réponse : B**

---

### Question 3
**Que signifie R=1 dans l'algorithme Clock ?**

A) Page modifiée
B) Page référencée récemment
C) Page verrouillée
D) Page invalide
E) Page partagée
F) Page en lecture
G) Page en écriture
H) Page compressée
I) Page swap
J) Page kernel

**Réponse : B**

---

### Question 4
**Quelle est la complexité temporelle de LRU optimal ?**

A) O(1)
B) O(log n)
C) O(n)
D) O(n log n)
E) O(n²)
F) O(1) avec les bonnes structures
G) O(m) où m = frames
H) Impossible à déterminer
I) Exponentielle
J) Linéaire en nombre de frames

**Réponse : F**

---

### Question 5
**Le thrashing se produit quand :**

A) Le CPU est à 100%
B) Le working set > mémoire disponible
C) Trop de processus
D) Disque plein
E) RAM défectueuse
F) Cache miss élevé
G) Network saturé
H) Kernel panic
I) OOM killer actif
J) Swap désactivé

**Réponse : B**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 2.1.3 — squid_page_games |
| **Difficulté** | ★★★★★☆☆☆☆☆ (5/10) |
| **Bonus** | 🔥 Avancé (7/10) |
| **XP Base** | 500 |
| **XP Bonus** | ×3 = 1500 |
| **Durée** | 4-5 heures |
| **Fichiers** | 10 fichiers C + header + Makefile |
| **Concepts** | 8 concepts du curriculum |
| **Tests** | 12 tests automatisés |
| **Mutants** | 5 solutions buggées |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1.3-squid_page_games",
    "generated_at": "2026-01-11",

    "metadata": {
      "exercise_id": "2.1.3",
      "exercise_name": "squid_page_games",
      "module": "2.1",
      "module_name": "Memory Management",
      "concept": "Page Replacement",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse",
      "phase": 2,
      "difficulty": 5,
      "difficulty_stars": "★★★★★☆☆☆☆☆",
      "language": "c17",
      "duration_minutes": 300,
      "xp_base": 500,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "ADVANCED",
      "bonus_icon": "🔥",
      "complexity_time": "T3 O(n×m)",
      "complexity_space": "S2 O(m)",
      "prerequisites": ["ex01", "ex02", "listes_chainees"],
      "domains": ["Mem", "Struct", "Algo"],
      "domains_bonus": ["Probas"],
      "tags": ["page-replacement", "fifo", "lru", "clock", "thrashing"],
      "meme_reference": "Squid Game"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/pager_fifo.c": "/* Section 4.3 */",
      "references/working_set.c": "/* Section 4.6 */",
      "mutants/mutant_a_boundary.c": "/* Section 4.10 */",
      "mutants/mutant_b_safety.c": "/* Section 4.10 */",
      "mutants/mutant_c_logic.c": "/* Section 4.10 */",
      "mutants/mutant_d_return.c": "/* Section 4.10 */",
      "mutants/mutant_e_edge.c": "/* Section 4.10 */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/pager_fifo.c",
        "references/pager_lru.c",
        "references/pager_clock.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_logic.c",
        "mutants/mutant_d_return.c",
        "mutants/mutant_e_edge.c"
      ]
    }
  }
}
```

---

*Exercice généré avec HACKBRAIN v5.5.2*
*"L'excellence pédagogique ne se négocie pas"*
*Compatible ENGINE v22.1 + Mutation Tester*
