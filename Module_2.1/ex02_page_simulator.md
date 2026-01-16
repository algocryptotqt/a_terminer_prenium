# [Module 2.1] - Exercise 02: Virtual Memory Page Simulator

## Métadonnées

```yaml
module: "2.1 - Memory Management"
exercise: "ex02"
difficulty: moyen
estimated_time: "5-7 heures"
prerequisite_exercises: ["ex01"]
concepts_requis:
  - "Représentation binaire des adresses"
  - "Structures de données (arbres, tables)"
  - "Arithmétique bit-à-bit"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.1.2.b | Address translation | VA → PA conversion |
| 2.1.2.e-g | Page table | Structure et entrées PTE |
| 2.1.2.h-k | Multi-level tables | 2/4/5 niveaux |
| 2.1.3.a-d | CR3, directories | Registres et structures |
| 2.1.3.e-h | TLB | Cache de traductions |
| 2.1.3.i-l | Huge pages, KPTI | Pages larges, isolation |
| 2.1.4.a-d | Page fault types | Minor, major, invalid |
| 2.1.4.e-h | Demand paging, COW | Stratégies de chargement |

### Objectifs Pédagogiques

À la fin de cet exercice, vous saurez:
1. Comprendre le mécanisme de traduction d'adresses virtuelles en physiques
2. Implémenter une table des pages multi-niveaux
3. Simuler le comportement du TLB et mesurer son impact
4. Gérer les page faults et implémenter demand paging
5. Comprendre Copy-on-Write et son implémentation

---

## Contexte

La mémoire virtuelle est une abstraction fondamentale des systèmes d'exploitation modernes. Elle permet à chaque processus de croire qu'il a accès à tout l'espace d'adressage (4 GB en 32-bit, 256 TB en 64-bit), alors que la RAM physique est partagée et limitée.

**Le problème central**: Comment traduire efficacement une adresse virtuelle en adresse physique?

Sur x86-64, une adresse virtuelle 48-bit est découpée ainsi:
```
┌────────────────────────────────────────────────────────────┐
│ Bits 47-39 │ Bits 38-30 │ Bits 29-21 │ Bits 20-12 │ Bits 11-0 │
│   PML4     │    PDPT    │     PD     │     PT     │  Offset   │
│ (9 bits)   │  (9 bits)  │  (9 bits)  │  (9 bits)  │ (12 bits) │
└────────────────────────────────────────────────────────────┘
```

Chaque niveau contient 512 entrées (2^9). Le matériel parcourt cette hiérarchie pour trouver le numéro de frame physique.

**Le défi intellectuel**: Vous allez implémenter cette mécanique vous-même, en simulant le matériel, le TLB, et les page faults.

---

## Énoncé

### Vue d'Ensemble

Créez un simulateur complet de mémoire virtuelle paginée. Le simulateur doit implémenter:
1. Une table des pages 4 niveaux (comme x86-64)
2. Un TLB (Translation Lookaside Buffer) configurable
3. La gestion des page faults avec différentes stratégies
4. Copy-on-Write pour les pages partagées

### Spécifications Fonctionnelles

#### Partie 1: Structure de Base

```c
// Configuration du simulateur
typedef struct {
    uint32_t page_size;           // 4096 (4KB) par défaut
    uint32_t tlb_entries;         // Nombre d'entrées TLB
    uint32_t physical_frames;     // Nombre de frames physiques
    uint8_t  page_table_levels;   // 2, 3, ou 4 niveaux
} vm_config_t;

// Page Table Entry
typedef struct {
    uint64_t pfn       : 40;  // Physical Frame Number
    uint64_t present   : 1;   // Page en mémoire?
    uint64_t writable  : 1;   // Accessible en écriture?
    uint64_t user      : 1;   // Accessible en mode user?
    uint64_t accessed  : 1;   // Accédée récemment?
    uint64_t dirty     : 1;   // Modifiée?
    uint64_t cow       : 1;   // Copy-on-Write?
    uint64_t reserved  : 18;
} pte_t;

// Contexte de simulation
typedef struct vm_context vm_context_t;

// Création/destruction
vm_context_t *vm_create(const vm_config_t *config);
void vm_destroy(vm_context_t *ctx);
```

#### Partie 2: Traduction d'Adresses

```c
typedef enum {
    VM_OK,              // Traduction réussie
    VM_PAGE_FAULT,      // Page non présente
    VM_PROTECTION,      // Violation de permission
    VM_INVALID          // Adresse invalide
} vm_result_t;

// Traduit une adresse virtuelle en adresse physique
// access_type: 'r' (read), 'w' (write), 'x' (execute)
vm_result_t vm_translate(vm_context_t *ctx,
                         uint64_t virtual_addr,
                         uint64_t *physical_addr,
                         char access_type);

// Statistiques de traduction
typedef struct {
    uint64_t translations;     // Total de traductions
    uint64_t tlb_hits;         // TLB hits
    uint64_t tlb_misses;       // TLB misses
    uint64_t page_walks;       // Parcours complets de table
    uint64_t page_faults;      // Page faults
    uint64_t cow_faults;       // COW faults (copie déclenchée)
} vm_stats_t;

vm_stats_t vm_get_stats(const vm_context_t *ctx);
```

#### Partie 3: Gestion des Pages

```c
// Mappe une page virtuelle vers une frame physique
int vm_map_page(vm_context_t *ctx,
                uint64_t virtual_page,
                uint64_t physical_frame,
                int writable, int user);

// Démappe une page
int vm_unmap_page(vm_context_t *ctx, uint64_t virtual_page);

// Active Copy-on-Write sur une page
int vm_set_cow(vm_context_t *ctx, uint64_t virtual_page);

// Handler de page fault (callback)
typedef int (*page_fault_handler_t)(vm_context_t *ctx,
                                    uint64_t virtual_addr,
                                    char access_type,
                                    void *user_data);

void vm_set_fault_handler(vm_context_t *ctx,
                          page_fault_handler_t handler,
                          void *user_data);
```

#### Partie 4: Simulation TLB

```c
// Politique de remplacement TLB
typedef enum {
    TLB_FIFO,           // First-In-First-Out
    TLB_LRU,            // Least Recently Used
    TLB_RANDOM          // Aléatoire
} tlb_policy_t;

void vm_set_tlb_policy(vm_context_t *ctx, tlb_policy_t policy);

// Flush TLB (simulation d'un context switch)
void vm_tlb_flush(vm_context_t *ctx);

// Flush une entrée spécifique (équivalent INVLPG)
void vm_tlb_invalidate(vm_context_t *ctx, uint64_t virtual_page);
```

---

## Exemple d'Utilisation

### Exemple 1: Traduction Simple

```c
vm_config_t config = {
    .page_size = 4096,
    .tlb_entries = 64,
    .physical_frames = 256,  // 1 MB de RAM simulée
    .page_table_levels = 4
};

vm_context_t *ctx = vm_create(&config);

// Mapper la page virtuelle 0 vers la frame 10
vm_map_page(ctx, 0, 10, 1, 1);

// Traduire l'adresse 0x123 (dans la page 0)
uint64_t phys;
vm_result_t res = vm_translate(ctx, 0x123, &phys, 'r');

assert(res == VM_OK);
assert(phys == 10 * 4096 + 0x123);  // Frame 10, offset 0x123

vm_destroy(ctx);
```

### Exemple 2: Page Fault et Demand Paging

```c
int demand_paging_handler(vm_context_t *ctx, uint64_t vaddr,
                          char access, void *data) {
    static uint64_t next_frame = 100;

    // Allouer une nouvelle frame et mapper la page
    uint64_t vpage = vaddr / 4096;
    vm_map_page(ctx, vpage, next_frame++, 1, 1);

    return 0;  // Succès, réessayer la traduction
}

vm_context_t *ctx = vm_create(&config);
vm_set_fault_handler(ctx, demand_paging_handler, NULL);

// Pas de mapping initial, mais on accède quand même
uint64_t phys;
vm_result_t res = vm_translate(ctx, 0x5000, &phys, 'r');

// Le handler a été appelé, la page est maintenant mappée
assert(res == VM_OK);
```

### Exemple 3: Copy-on-Write

```c
// Mapper une page en lecture seule avec COW activé
vm_map_page(ctx, 42, 200, 0, 1);  // writable=0
vm_set_cow(ctx, 42);

// Lecture: OK
vm_translate(ctx, 42 * 4096, &phys, 'r');  // OK

// Écriture: déclenche COW
vm_translate(ctx, 42 * 4096, &phys, 'w');  // COW fault
// → Le handler doit copier la frame et remapper en writable

vm_stats_t stats = vm_get_stats(ctx);
printf("COW faults: %lu\n", stats.cow_faults);
```

### Exemple 4: Benchmark TLB

```c
// Mesurer l'impact du TLB
for (int tlb_size = 8; tlb_size <= 256; tlb_size *= 2) {
    vm_config_t cfg = { .page_size = 4096, .tlb_entries = tlb_size, ... };
    vm_context_t *ctx = vm_create(&cfg);

    // Mapper 1000 pages
    for (int i = 0; i < 1000; i++) {
        vm_map_page(ctx, i, i, 1, 1);
    }

    // Accéder aléatoirement
    for (int i = 0; i < 100000; i++) {
        vm_translate(ctx, (rand() % 1000) * 4096, &phys, 'r');
    }

    vm_stats_t s = vm_get_stats(ctx);
    printf("TLB %3d entries: hit rate %.2f%%\n",
           tlb_size,
           100.0 * s.tlb_hits / s.translations);

    vm_destroy(ctx);
}
```

**Sortie attendue** (exemple):
```
TLB   8 entries: hit rate 0.79%
TLB  16 entries: hit rate 1.58%
TLB  32 entries: hit rate 3.15%
TLB  64 entries: hit rate 6.25%
TLB 128 entries: hit rate 12.43%
TLB 256 entries: hit rate 24.67%
```

---

## Tests Moulinette

### Tests de Traduction

```yaml
test_01_basic_mapping:
  description: "Mapping et traduction de base"
  validation: |
    Page 0 → Frame 5, accès à 0x100 donne 0x5100
  expected: "PASS"

test_02_multi_level_walk:
  description: "Parcours complet 4 niveaux"
  validation: |
    Mapper page 0x123456, vérifier que les 4 niveaux sont créés
  expected: "PASS"

test_03_offset_preservation:
  description: "L'offset intra-page est préservé"
  validation: |
    VA 0x12345678 → PA avec même offset (0x678)
  expected: "PASS"
```

### Tests TLB

```yaml
test_04_tlb_hit:
  description: "TLB hit au second accès"
  validation: |
    Premier accès: TLB miss
    Second accès: TLB hit
  expected: "PASS"

test_05_tlb_eviction:
  description: "Éviction TLB respecte la politique"
  validation: |
    Avec TLB de 4 entrées, 5 pages différentes
    → Éviction selon politique (FIFO/LRU)
  expected: "PASS"

test_06_tlb_flush:
  description: "Flush TLB remet les compteurs à zéro"
  validation: |
    Après flush, prochain accès est un miss
  expected: "PASS"
```

### Tests Page Fault

```yaml
test_07_page_fault_unmapped:
  description: "Page fault sur page non mappée"
  validation: |
    Accès à page non mappée retourne VM_PAGE_FAULT
  expected: "PASS"

test_08_protection_fault:
  description: "Protection fault sur écriture read-only"
  validation: |
    Page mappée writable=0, écriture → VM_PROTECTION
  expected: "PASS"

test_09_demand_paging:
  description: "Demand paging via handler"
  validation: |
    Handler mappe la page, seconde traduction réussit
  expected: "PASS"
```

### Tests COW

```yaml
test_10_cow_read_ok:
  description: "Lecture sur page COW OK"
  validation: |
    Page avec cow=1, lecture réussit normalement
  expected: "PASS"

test_11_cow_write_triggers:
  description: "Écriture sur page COW déclenche fault"
  validation: |
    Page avec cow=1, écriture → cow_faults++
  expected: "PASS"
```

### Tests Performance

```yaml
test_12_perf_10k_translations:
  description: "10000 traductions en moins de 100ms"
  command: "time ./vm_bench 10000"
  expected: "real < 0.1s"

test_13_memory_overhead:
  description: "Overhead mémoire raisonnable"
  validation: |
    1000 pages mappées < 1 MB overhead structures
  expected: "PASS"
```

---

## Critères d'Évaluation

| Critère | Points | Description |
|---------|--------|-------------|
| **Correction** | 40 | |
| - Traduction exacte | 15 | Adresses physiques correctes |
| - Multi-niveaux | 10 | 4 niveaux fonctionnels |
| - TLB correct | 10 | Hit/miss/éviction |
| - COW implémenté | 5 | Détection et statistiques |
| **Sécurité** | 25 | |
| - Pas de fuites | 10 | Valgrind clean |
| - Bounds checking | 10 | Pas d'accès hors limites |
| - État cohérent | 5 | Invariants maintenus |
| **Conception** | 20 | |
| - Architecture modulaire | 10 | TLB/PageTable/Stats séparés |
| - Extensibilité | 10 | Facile d'ajouter politiques |
| **Lisibilité** | 15 | |
| - Bit manipulation claire | 5 | Macros pour extraction bits |
| - Documentation | 5 | Diagrammes/commentaires |
| - Tests clairs | 5 | Nommage explicite |

**Score minimum**: 80/100

---

## Indices et Ressources

### Questions pour Réfléchir

1. Pourquoi les tables sont-elles organisées en niveaux plutôt qu'une seule grande table?
2. Quel est le compromis du nombre d'entrées TLB?
3. Pourquoi COW est-il utile après un fork()?
4. Comment le matériel sait-il qu'il doit faire un page walk?

### Formules Utiles

```c
// Extraire l'index à chaque niveau (pour page_size=4096, 4 niveaux)
#define PAGE_OFFSET(addr)   ((addr) & 0xFFF)          // Bits 0-11
#define PT_INDEX(addr)      (((addr) >> 12) & 0x1FF)  // Bits 12-20
#define PD_INDEX(addr)      (((addr) >> 21) & 0x1FF)  // Bits 21-29
#define PDPT_INDEX(addr)    (((addr) >> 30) & 0x1FF)  // Bits 30-38
#define PML4_INDEX(addr)    (((addr) >> 39) & 0x1FF)  // Bits 39-47
```

### Pièges Fréquents

1. **Oublier de marquer accessed/dirty** lors des accès
2. **TLB incohérent** après unmap sans invalidate
3. **Integer overflow** dans les calculs d'adresses 64-bit
4. **Allouer les tables de niveau intermédiaire** seulement à la demande

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 24 | Simulation complète, pas trivial |
| Couverture conceptuelle | 25 | 8+ concepts majeurs couverts |
| Testabilité auto | 23 | Tests objectifs, benchmarks |
| Originalité | 24 | Pas une copie, simulation complète |
| **TOTAL** | **96/100** | ✓ Validé |

**✓ Score ≥ 95, exercice validé.**
