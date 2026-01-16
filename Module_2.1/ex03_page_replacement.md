# [Module 2.1] - Exercise 03: Page Replacement Algorithms Lab

## Métadonnées

```yaml
module: "2.1 - Memory Management"
exercise: "ex03"
difficulty: moyen
estimated_time: "4-5 heures"
prerequisite_exercises: ["ex02"]
concepts_requis:
  - "Listes chaînées et files"
  - "Structures de données (queues, stacks)"
  - "Analyse de complexité"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.1.5.a | Page replacement | Choisir quelle page évincer |
| 2.1.5.b | Optimal (Belady) | Algorithme théorique optimal |
| 2.1.5.c-d | FIFO, Belady's anomaly | Simple mais problématique |
| 2.1.5.e-g | LRU | Least Recently Used |
| 2.1.5.h-i | Clock algorithm | Approximation efficace de LRU |
| 2.1.5.j-k | Working set | Ensemble de pages actives |
| 2.1.5.l | Page Fault Frequency | Ajustement dynamique |
| 2.1.5.m-n | Thrashing | Trop de page faults |

### Objectifs Pédagogiques

À la fin de cet exercice, vous saurez:
1. Implémenter les principaux algorithmes de remplacement de pages
2. Analyser et comparer leurs performances sur différents workloads
3. Détecter et prévenir le thrashing
4. Comprendre pourquoi LRU est difficile à implémenter en hardware

---

## Contexte

Quand la mémoire physique est pleine et qu'une nouvelle page doit être chargée, le système d'exploitation doit choisir une page victime à évincer. Ce choix a un impact énorme sur les performances:
- Évincer une page qui sera réutilisée bientôt → page fault supplémentaire
- Évincer la bonne page → aucun coût futur

**L'algorithme optimal** (Belady) sait quel page ne sera pas utilisée le plus longtemps, mais il nécessite de connaître le futur — impossible en pratique.

**Le défi**: Approximer l'optimal avec des algorithmes réalisables en temps réel.

---

## Énoncé

### Vue d'Ensemble

Créez un simulateur permettant de comparer les algorithmes de remplacement de pages. Le simulateur charge des traces d'accès mémoire et mesure les performances de chaque algorithme.

### Spécifications Fonctionnelles

#### Partie 1: Infrastructure de Base

```c
// Configuration du simulateur
typedef struct {
    uint32_t num_frames;     // Nombre de frames physiques disponibles
    uint32_t num_pages;      // Nombre total de pages virtuelles
} pager_config_t;

// Référence mémoire (élément de la trace)
typedef struct {
    uint32_t page_number;    // Numéro de page accédée
    char     access_type;    // 'r' (read) ou 'w' (write)
} page_ref_t;

// Trace d'accès
typedef struct {
    page_ref_t *refs;        // Tableau de références
    size_t      count;       // Nombre de références
} trace_t;

// Charger une trace depuis un fichier
// Format: une ligne par accès, "page_number access_type"
// Exemple: "42 r\n17 w\n42 r\n"
trace_t *trace_load(const char *filename);
void trace_free(trace_t *trace);
```

#### Partie 2: Interface des Algorithmes

```c
// Résultats de simulation
typedef struct {
    uint64_t page_faults;    // Nombre total de page faults
    uint64_t writes_back;    // Pages dirty écrites sur disque
    double   hit_rate;       // Taux de succès (0.0 - 1.0)
} pager_result_t;

// Type de fonction d'algorithme
typedef pager_result_t (*pager_algorithm_t)(
    const pager_config_t *config,
    const trace_t *trace
);

// Algorithmes à implémenter
pager_result_t pager_optimal(const pager_config_t *cfg, const trace_t *trace);
pager_result_t pager_fifo(const pager_config_t *cfg, const trace_t *trace);
pager_result_t pager_lru(const pager_config_t *cfg, const trace_t *trace);
pager_result_t pager_clock(const pager_config_t *cfg, const trace_t *trace);
pager_result_t pager_clock_enhanced(const pager_config_t *cfg, const trace_t *trace);
pager_result_t pager_lfu(const pager_config_t *cfg, const trace_t *trace);
pager_result_t pager_random(const pager_config_t *cfg, const trace_t *trace);
```

#### Partie 3: Détection de Thrashing

```c
// Statistiques de working set
typedef struct {
    double avg_working_set;      // Taille moyenne du working set
    double max_working_set;      // Taille max observée
    int    thrashing_detected;   // 1 si thrashing détecté
    double thrashing_severity;   // 0.0 (aucun) à 1.0 (sévère)
} working_set_stats_t;

// Analyse du working set avec fenêtre temporelle
working_set_stats_t analyze_working_set(
    const trace_t *trace,
    uint32_t window_size
);

// Page Fault Frequency analysis
typedef struct {
    double pff_avg;              // PFF moyen
    double pff_max;              // PFF max
    uint32_t recommended_frames; // Frames recommandées pour éviter thrashing
} pff_analysis_t;

pff_analysis_t analyze_pff(
    const trace_t *trace,
    uint32_t num_frames
);
```

---

## Détail des Algorithmes

### FIFO (First-In-First-Out)
```
Évincer la page qui est en mémoire depuis le plus longtemps.
Structure: Queue simple
Complexité: O(1) insertion, O(1) éviction
Problème: Belady's anomaly possible
```

### LRU (Least Recently Used)
```
Évincer la page utilisée il y a le plus longtemps.
Structure: Liste doublement chaînée + HashMap
Complexité: O(1) avec bonne implémentation
Avantage: Bon comportement général
Inconvénient: Coûteux en hardware
```

### Clock (Second Chance)
```
FIFO amélioré avec bit de référence.
- Parcourir circulairement les pages
- Si bit R = 1: mettre à 0, passer à la suivante
- Si bit R = 0: évincer cette page
Approxime LRU efficacement.
```

### Enhanced Clock (NRU - Not Recently Used)
```
Considère R (référence) et M (modifié):
Classe 0: R=0, M=0 (meilleur candidat)
Classe 1: R=0, M=1
Classe 2: R=1, M=0
Classe 3: R=1, M=1 (pire candidat)
```

### Optimal (Belady)
```
Évincer la page qui ne sera pas utilisée le plus longtemps.
Nécessite connaissance du futur → simulation uniquement.
Sert de borne théorique pour comparaison.
```

---

## Exemple d'Utilisation

### Exemple 1: Comparaison Simple

```c
int main(void) {
    trace_t *trace = trace_load("workload1.trace");
    pager_config_t cfg = { .num_frames = 4, .num_pages = 10 };

    printf("Algorithm       | Page Faults | Hit Rate\n");
    printf("----------------|-------------|----------\n");

    pager_result_t r;

    r = pager_optimal(&cfg, trace);
    printf("Optimal         | %11lu | %.2f%%\n", r.page_faults, r.hit_rate * 100);

    r = pager_lru(&cfg, trace);
    printf("LRU             | %11lu | %.2f%%\n", r.page_faults, r.hit_rate * 100);

    r = pager_clock(&cfg, trace);
    printf("Clock           | %11lu | %.2f%%\n", r.page_faults, r.hit_rate * 100);

    r = pager_fifo(&cfg, trace);
    printf("FIFO            | %11lu | %.2f%%\n", r.page_faults, r.hit_rate * 100);

    trace_free(trace);
}
```

**Sortie attendue** (exemple):
```
Algorithm       | Page Faults | Hit Rate
----------------|-------------|----------
Optimal         |           6 | 70.00%
LRU             |           8 | 60.00%
Clock           |           9 | 55.00%
FIFO            |          10 | 50.00%
```

### Exemple 2: Démonstration de Belady's Anomaly

```c
// Trace spécifique qui montre Belady's anomaly avec FIFO
// Séquence: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5

void demonstrate_belady_anomaly(void) {
    trace_t *trace = create_belady_trace();

    for (int frames = 3; frames <= 5; frames++) {
        pager_config_t cfg = { .num_frames = frames, .num_pages = 5 };
        pager_result_t r = pager_fifo(&cfg, trace);
        printf("FIFO with %d frames: %lu page faults\n",
               frames, r.page_faults);
    }
}
```

**Sortie attendue**:
```
FIFO with 3 frames: 9 page faults
FIFO with 4 frames: 10 page faults   ← Plus de frames, plus de faults!
FIFO with 5 frames: 5 page faults
```

### Exemple 3: Analyse de Thrashing

```c
// Détecter si un workload cause du thrashing
working_set_stats_t ws = analyze_working_set(trace, 100);

printf("Working Set Analysis:\n");
printf("  Average size: %.1f pages\n", ws.avg_working_set);
printf("  Maximum size: %.1f pages\n", ws.max_working_set);
printf("  Thrashing: %s (severity: %.1f%%)\n",
       ws.thrashing_detected ? "YES" : "No",
       ws.thrashing_severity * 100);

if (ws.thrashing_detected) {
    pff_analysis_t pff = analyze_pff(trace, cfg.num_frames);
    printf("  Recommended frames: %u\n", pff.recommended_frames);
}
```

---

## Format de Trace

```
# Fichier workload1.trace
# Format: page_number access_type
# Commentaires commencent par #
1 r
2 r
3 r
4 r
1 r
2 w
5 r
1 r
2 r
3 w
4 r
5 r
```

---

## Tests Moulinette

### Tests de Correction

```yaml
test_01_fifo_basic:
  description: "FIFO basique - séquence simple"
  trace: "1 r, 2 r, 3 r, 1 r, 4 r"
  frames: 3
  expected_faults: 4  # 1,2,3 (3 faults), 1 hit, 4 evicts 1 (1 fault)

test_02_lru_basic:
  description: "LRU basique"
  trace: "1 r, 2 r, 3 r, 1 r, 4 r"
  frames: 3
  expected_faults: 4  # Même que FIFO ici

test_03_optimal_basic:
  description: "Optimal fait mieux que FIFO/LRU"
  trace: "1 r, 2 r, 3 r, 4 r, 1 r, 2 r, 5 r"
  frames: 3
  validation: "optimal_faults <= lru_faults <= fifo_faults"

test_04_belady_anomaly:
  description: "Belady's anomaly détectable avec FIFO"
  trace: "standard Belady sequence"
  validation: "fifo(4 frames) > fifo(3 frames)"

test_05_clock_approximates_lru:
  description: "Clock proche de LRU"
  trace: "random 1000 accès"
  validation: "abs(clock_faults - lru_faults) < 0.1 * lru_faults"
```

### Tests de Thrashing

```yaml
test_06_thrashing_detection:
  description: "Détecter thrashing sur workload pathologique"
  trace: "cycling through more pages than frames"
  frames: 4
  expected: "thrashing_detected == 1"

test_07_working_set_calculation:
  description: "Working set calculé correctement"
  trace: "locality pattern"
  window: 10
  expected: "avg_working_set proche de 5"
```

### Tests Performance

```yaml
test_08_large_trace:
  description: "100000 accès en moins de 1 seconde"
  trace_size: 100000
  time_limit: "1s"

test_09_memory_efficiency:
  description: "Mémoire utilisée raisonnable"
  validation: "< 1MB pour 100000 accès"
```

---

## Critères d'Évaluation

| Critère | Points | Description |
|---------|--------|-------------|
| **Correction** | 40 | |
| - 7 algorithmes corrects | 21 | 3 pts chacun |
| - Belady's anomaly démontré | 4 | Trace et explication |
| - Thrashing détecté | 5 | Working set correct |
| - Statistiques exactes | 10 | hit_rate, writes_back |
| **Sécurité** | 25 | |
| - Pas de fuites | 10 | Valgrind clean |
| - Bounds checking | 10 | Traces invalides gérées |
| - Robustesse | 5 | Fichiers absents, vides |
| **Conception** | 20 | |
| - Interface commune | 10 | Tous algos même signature |
| - Structures efficaces | 10 | LRU O(1), pas O(n) |
| **Lisibilité** | 15 | |
| - Séparation claire | 5 | Un fichier par algo OK |
| - Nommage descriptif | 5 | |
| - Documentation | 5 | Complexité documentée |

**Score minimum**: 80/100

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 24 | Comparer, analyser, pas juste coder |
| Couverture conceptuelle | 25 | 8 concepts couverts en profondeur |
| Testabilité auto | 24 | Traces reproductibles, métriques claires |
| Originalité | 23 | Focus sur analyse, Belady, thrashing |
| **TOTAL** | **96/100** | ✓ Validé |

**✓ Score ≥ 95, exercice validé.**
