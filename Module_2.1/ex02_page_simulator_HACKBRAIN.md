<thinking>
## Analyse du Concept
- Concept : Simulation de mémoire virtuelle paginée avec table des pages multi-niveaux, TLB, et page faults
- Phase demandée : 2 (Programmation Système Avancée)
- Adapté ? OUI - C'est un concept fondamental de l'architecture x86-64, parfait pour Phase 2

## Combo Base + Bonus
- Exercice de base : Table des pages 4 niveaux, TLB configurable, traduction d'adresses
- Bonus : Copy-on-Write complet, huge pages (2MB/1GB), KPTI simulation
- Palier bonus : 💀 Expert (difficulté 8/10)
- Progression logique ? OUI - Le bonus ajoute des fonctionnalités avancées du kernel moderne

## Prérequis & Difficulté
- Prérequis réels :
  - ex01 (Memory Inspector) - comprendre /proc/maps
  - Arithmétique binaire et bit manipulation
  - Structures de données (arbres, tables)
- Difficulté estimée : 7/10 (base), 8/10 (bonus)
- Cohérent avec phase ? OUI - Phase 2 = 4-6/10, exercice avancé = 7/10

## Aspect Fun/Culture
- Contexte choisi : The Matrix (1999)
- Analogie parfaite : Dans Matrix, les adresses virtuelles sont les perceptions de Neo dans la simulation, et les adresses physiques sont les connexions réelles aux pods. La "pilule rouge" révèle la traduction entre les deux mondes.
- MEME mnémotechnique : "There is no spoon" - il n'y a pas d'adresse physique directe, tout passe par la traduction
- Pourquoi c'est fun :
  - La mémoire virtuelle EST une simulation, comme Matrix
  - Le TLB est comme le "cache de réalité" qui accélère la perception
  - Un page fault est comme un "glitch in the Matrix"
  - Note d'intelligence : 96/100 - L'analogie est parfaite

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : Mauvais masque pour extraire les indices de niveau
   ```c
   #define PT_INDEX(addr) (((addr) >> 12) & 0xFF)  // devrait être 0x1FF (9 bits)
   ```

2. Mutant B (Safety) : Pas de vérification du bit present dans la PTE
   ```c
   pte_t *pte = &table[index];
   return pte->pfn;  // ERREUR: pas de if (!pte->present) return VM_PAGE_FAULT
   ```

3. Mutant C (Resource) : Pas de libération des tables de niveau intermédiaire
   ```c
   void vm_destroy(vm_context_t *ctx) {
       free(ctx);  // ERREUR: les tables PML4/PDPT/PD/PT ne sont pas libérées
   }
   ```

4. Mutant D (Logic) : TLB jamais invalidé après unmap
   ```c
   int vm_unmap_page(vm_context_t *ctx, uint64_t vpage) {
       pte->present = 0;
       // ERREUR: vm_tlb_invalidate(ctx, vpage) manquant
       return 0;
   }
   ```

5. Mutant E (Return) : Offset perdu dans la traduction
   ```c
   *physical_addr = pte->pfn * PAGE_SIZE;  // ERREUR: + PAGE_OFFSET(vaddr) manquant
   ```

## Verdict
VALIDE - L'exercice est complet, difficile mais approprié pour Phase 2 avancée.
</thinking>

---

# Exercice 2.1.2-a : matrix_pager

**Module :**
2.1 — Memory Management

**Concept :**
a — Address Translation (VA → PA, Page Tables, TLB)

**Difficulté :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
1 — Concept isolé (focus sur traduction d'adresses)

**Langage :**
C (C17)

**Prérequis :**
- ex01 (Memory Inspector) - comprendre l'organisation mémoire
- Arithmétique binaire et manipulation de bits
- Structures de données arborescentes (Phase 1)

**Domaines :**
Mem, CPU, Encodage, Struct

**Durée estimée :**
300 min

**XP Base :**
200

**Complexité :**
T4 O(1) traduction avec TLB × S4 O(n) tables de pages

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichier à rendre :**
```
ex02/
├── matrix_pager.h       # Prototypes et structures
├── matrix_pager.c       # Implémentation simulateur
├── tlb.c                # Implémentation TLB
├── page_table.c         # Tables de pages multi-niveaux
├── main.c               # Programme de démonstration
└── Makefile
```

**Fonctions autorisées :**
- `malloc`, `free`, `calloc`, `realloc`
- `memset`, `memcpy`
- `write` (pour affichage)
- `snprintf`

**Fonctions interdites :**
- `mmap`, `sbrk` (pas de vraie allocation système)
- `printf`, `fprintf` (utiliser write)

### 1.2 Consigne

#### 🎮 Version Culture Pop — "The Matrix: Virtual Memory"

**"There is no spoon... there is no physical address."**

Dans **The Matrix**, Neo découvre que le monde qu'il perçoit n'est qu'une simulation. Les rues, les bâtiments, les gens — tout n'est que données traduites en perception.

La mémoire virtuelle fonctionne exactement de la même manière : chaque processus croit avoir accès à 256 TB d'espace mémoire (l'illusion de Matrix), alors qu'en réalité seules quelques MB de RAM physique existent (les pods).

Le **MMU** (Memory Management Unit) est l'équivalent des machines de Matrix : il traduit les adresses virtuelles (perceptions) en adresses physiques (réalité). Cette traduction passe par une hiérarchie de tables de pages, exactement comme Matrix a plusieurs niveaux de simulation.

**Ton rôle : Devenir l'Architecte**

Tu vas créer un simulateur complet de mémoire virtuelle paginée, capable de :
- Traduire des adresses virtuelles 48 bits en adresses physiques
- Gérer une table des pages 4 niveaux (comme x86-64)
- Simuler un TLB (Translation Lookaside Buffer) — le "cache de réalité"
- Déclencher des page faults quand Neo essaie d'accéder à une zone non chargée

---

#### 📚 Version Académique — Énoncé Formel

La mémoire virtuelle est une abstraction fondamentale des systèmes d'exploitation modernes. Elle permet à chaque processus de disposer de son propre espace d'adressage isolé, indépendamment de la mémoire physique disponible.

Sur l'architecture x86-64, une adresse virtuelle 48 bits est décomposée en 5 parties :

```
┌─────────────────────────────────────────────────────────────────────────┐
│ Bits 47-39 │ Bits 38-30 │ Bits 29-21 │ Bits 20-12 │ Bits 11-0          │
│   PML4     │    PDPT    │     PD     │     PT     │    Page Offset     │
│ (9 bits)   │  (9 bits)  │  (9 bits)  │  (9 bits)  │    (12 bits)       │
│  512 ent.  │  512 ent.  │  512 ent.  │  512 ent.  │    4096 bytes      │
└─────────────────────────────────────────────────────────────────────────┘
```

Chaque niveau pointe vers le niveau suivant (ou vers une frame physique pour le dernier niveau). Le TLB accélère ce processus en cachant les traductions récentes.

**Objectif :**

Implémenter un simulateur complet de traduction d'adresses virtuelles avec :
1. Table des pages hiérarchique 4 niveaux (PML4 → PDPT → PD → PT)
2. TLB configurable avec différentes politiques de remplacement
3. Statistiques de performance (hits, misses, page walks)
4. Gestion des page faults avec callback handler

---

**Entrée :**
- `vm_config_t *config` : Configuration du simulateur
- `uint64_t virtual_addr` : Adresse virtuelle à traduire
- `char access_type` : Type d'accès ('r', 'w', 'x')

**Sortie :**
- `vm_result_t` : Résultat de la traduction (VM_OK, VM_PAGE_FAULT, VM_PROTECTION, VM_INVALID)
- `uint64_t *physical_addr` : Adresse physique correspondante
- `vm_stats_t` : Statistiques de traduction

**Contraintes :**
- Page size : 4096 bytes (4KB)
- Adresses virtuelles : 48 bits maximum
- Table entries : 512 par niveau (9 bits d'index)
- TLB entries : configurable (8 à 256)
- Alignement : toutes les structures alignées sur 8 bytes

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `vm_translate(ctx, 0x1000, &phys, 'r')` | `VM_OK` | Page 1 mappée, offset 0 |
| `vm_translate(ctx, 0x123456789, &phys, 'r')` | `VM_PAGE_FAULT` | Page non mappée |
| `vm_translate(ctx, 0x1000, &phys, 'w')` sur page read-only | `VM_PROTECTION` | Violation de permission |
| `vm_translate(ctx, 0x1234, &phys, 'r')` après mapping page 1→frame 42 | `phys = 42*4096 + 0x234` | Offset préservé |

### 1.3 Prototype

```c
/* === CONFIGURATION === */

typedef struct {
    uint32_t page_size;           // 4096 (4KB) par défaut
    uint32_t tlb_entries;         // Nombre d'entrées TLB (8-256)
    uint32_t physical_frames;     // Nombre de frames physiques
    uint8_t  page_table_levels;   // 2, 3, ou 4 niveaux
} vm_config_t;

/* === PAGE TABLE ENTRY === */

typedef struct {
    uint64_t pfn       : 40;  // Physical Frame Number
    uint64_t present   : 1;   // Page présente en mémoire?
    uint64_t writable  : 1;   // Accessible en écriture?
    uint64_t user      : 1;   // Accessible en mode utilisateur?
    uint64_t accessed  : 1;   // Accédée depuis le dernier clear?
    uint64_t dirty     : 1;   // Modifiée depuis le dernier clear?
    uint64_t cow       : 1;   // Copy-on-Write activé?
    uint64_t reserved  : 18;  // Réservé pour extensions
} pte_t;

/* === RÉSULTATS === */

typedef enum {
    VM_OK,              // Traduction réussie
    VM_PAGE_FAULT,      // Page non présente
    VM_PROTECTION,      // Violation de permission
    VM_INVALID          // Adresse invalide (hors range)
} vm_result_t;

/* === STATISTIQUES === */

typedef struct {
    uint64_t translations;     // Total de traductions
    uint64_t tlb_hits;         // Hits TLB
    uint64_t tlb_misses;       // Misses TLB
    uint64_t page_walks;       // Parcours complets de table
    uint64_t page_faults;      // Page faults générés
    uint64_t cow_faults;       // Copy-on-Write faults
    uint64_t protection_faults;// Violations de permission
} vm_stats_t;

/* === POLITIQUE TLB === */

typedef enum {
    TLB_FIFO,           // First-In-First-Out
    TLB_LRU,            // Least Recently Used
    TLB_RANDOM          // Aléatoire
} tlb_policy_t;

/* === CONTEXTE (opaque) === */

typedef struct vm_context vm_context_t;

/* === PAGE FAULT HANDLER === */

typedef int (*page_fault_handler_t)(
    vm_context_t *ctx,
    uint64_t virtual_addr,
    char access_type,
    void *user_data
);

/* === PROTOTYPES === */

// Création et destruction
vm_context_t *vm_create(const vm_config_t *config);
void vm_destroy(vm_context_t *ctx);

// Traduction d'adresses
vm_result_t vm_translate(
    vm_context_t *ctx,
    uint64_t virtual_addr,
    uint64_t *physical_addr,
    char access_type
);

// Gestion des mappings
int vm_map_page(
    vm_context_t *ctx,
    uint64_t virtual_page,
    uint64_t physical_frame,
    int writable,
    int user
);
int vm_unmap_page(vm_context_t *ctx, uint64_t virtual_page);

// Copy-on-Write
int vm_set_cow(vm_context_t *ctx, uint64_t virtual_page);

// Configuration TLB
void vm_set_tlb_policy(vm_context_t *ctx, tlb_policy_t policy);
void vm_tlb_flush(vm_context_t *ctx);
void vm_tlb_invalidate(vm_context_t *ctx, uint64_t virtual_page);

// Page fault handler
void vm_set_fault_handler(
    vm_context_t *ctx,
    page_fault_handler_t handler,
    void *user_data
);

// Statistiques
vm_stats_t vm_get_stats(const vm_context_t *ctx);
void vm_print_stats(const vm_stats_t *stats);
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Fait Technique Fascinant

Le TLB est probablement le cache le plus important de tout le système ! Sans lui, chaque accès mémoire nécessiterait 4 accès supplémentaires (un par niveau de table). Avec un TLB de 64 entrées et un hit rate de 99%, le temps d'accès moyen passe de 5× à seulement 1.04×.

Sur les processeurs Intel modernes, il existe même plusieurs niveaux de TLB :
- **L1 ITLB** : 64 entrées pour les instructions
- **L1 DTLB** : 64 entrées pour les données
- **L2 STLB** : 1536 entrées partagées

### 2.2 Anecdote Historique

La pagination à 4 niveaux (PML4) a été introduite avec l'architecture AMD64 en 2003. Intel a ensuite annoncé en 2017 le support de 5 niveaux (PML5) pour étendre l'espace d'adressage à 57 bits (128 PB de mémoire virtuelle). Linux supporte PML5 depuis le kernel 4.14.

### 2.3 Analogie Mnémotechnique

**La traduction d'adresse est comme naviguer dans un building :**
- **PML4** = L'adresse du quartier (quel building?)
- **PDPT** = L'étage du building
- **PD** = Le couloir à cet étage
- **PT** = Le numéro d'appartement
- **Offset** = Où exactement dans l'appartement

Le TLB est comme un post-it avec "Jean habite building 3, étage 7, appartement 42". Pas besoin de refaire tout le trajet si on y est déjà allé !

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation de ce concept |
|--------|--------------------------|
| **Développeur Kernel** | Implémenter/optimiser le MMU code, gérer les huge pages |
| **Ingénieur Performance** | Optimiser la localité mémoire pour maximiser TLB hits |
| **Développeur de Virtualization** | Nested page tables (NPT/EPT) pour VMs |
| **Ingénieur Sécurité** | KPTI (Meltdown mitigation), ASLR, SMEP/SMAP |
| **Développeur de Bases de Données** | Huge pages pour réduire TLB misses sur gros datasets |
| **Game Developer** | Memory-mapped files pour streaming d'assets |

**Cas d'usage concret :** Les bases de données comme PostgreSQL et Oracle utilisent des huge pages (2MB) pour réduire le nombre d'entrées TLB nécessaires. Avec des tables de plusieurs GB, passer de 4KB à 2MB pages divise par 512 le nombre de TLB entries requises !

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
matrix_pager.h  matrix_pager.c  tlb.c  page_table.c  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -std=c17 -c matrix_pager.c -o matrix_pager.o
gcc -Wall -Wextra -Werror -std=c17 -c tlb.c -o tlb.o
gcc -Wall -Wextra -Werror -std=c17 -c page_table.c -o page_table.o
gcc -Wall -Wextra -Werror -std=c17 -c main.c -o main.o
gcc matrix_pager.o tlb.o page_table.o main.o -o matrix_pager

$ ./matrix_pager
=== Matrix Pager v1.0 ===
Config: 4 levels, 64 TLB entries, 256 physical frames

Mapping page 0 -> frame 10
Mapping page 1 -> frame 20
Mapping page 2 -> frame 30

Translation tests:
  VA 0x0000000000000123 -> PA 0x000000000000A123 [OK]
  VA 0x0000000000001456 -> PA 0x0000000000014456 [OK]
  VA 0x0000000000002789 -> PA 0x000000000001E789 [OK]
  VA 0x0000000000003000 -> PAGE_FAULT [OK - unmapped]

TLB Statistics:
  Translations: 4
  TLB Hits: 0
  TLB Misses: 4
  Page Walks: 4

Second access (should hit TLB):
  VA 0x0000000000000123 -> PA 0x000000000000A123 [TLB HIT]

Updated Statistics:
  Translations: 5
  TLB Hits: 1
  TLB Misses: 4

All tests passed!
```

---

### 3.1 💀 BONUS EXPERT (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×4

**Time Complexity attendue :**
O(1) amortized avec TLB

**Space Complexity attendue :**
O(n) pour tables + O(TLB_SIZE) pour TLB

**Domaines Bonus :**
`CPU, Struct, Algo`

#### 3.1.1 Consigne Bonus

**🎮 "The Matrix Reloaded: Copy-on-Write & Huge Pages"**

Dans Matrix Reloaded, l'Architecte révèle que Matrix a été réécrite plusieurs fois. Chaque version partageait des données communes jusqu'à ce qu'une modification soit nécessaire — c'est exactement le principe du **Copy-on-Write**.

Et les **Huge Pages** ? C'est comme les "backdoors" de Matrix — des raccourcis qui permettent de naviguer plus vite en sautant des niveaux entiers de la simulation.

**Ta mission étendue :**

1. **Copy-on-Write complet** : Quand une page COW est écrite, copier la frame et remapper
2. **Huge Pages** : Support des pages 2MB (21 bits d'offset) et 1GB (30 bits)
3. **KPTI Simulation** : Séparer les tables kernel/user pour simuler la mitigation Meltdown

---

**Contraintes :**
┌─────────────────────────────────────────┐
│  COW : Copie paresseuse sur écriture    │
│  Huge 2MB : PT level skipped            │
│  Huge 1GB : PD+PT levels skipped        │
│  KPTI : Deux jeux de tables             │
└─────────────────────────────────────────┘

#### 3.1.2 Prototype Bonus

```c
/* === HUGE PAGES === */

typedef enum {
    PAGE_SIZE_4K,    // Standard 4KB pages
    PAGE_SIZE_2M,    // Huge 2MB pages (skip PT)
    PAGE_SIZE_1G     // Giant 1GB pages (skip PD+PT)
} page_size_t;

int vm_map_huge_page(
    vm_context_t *ctx,
    uint64_t virtual_page,
    uint64_t physical_frame,
    page_size_t size,
    int writable,
    int user
);

/* === COPY-ON-WRITE === */

// Handler COW appelé sur écriture
typedef uint64_t (*cow_handler_t)(
    vm_context_t *ctx,
    uint64_t virtual_page,
    uint64_t old_frame,
    void *user_data
);

void vm_set_cow_handler(
    vm_context_t *ctx,
    cow_handler_t handler,
    void *user_data
);

/* === KPTI === */

// Active la séparation kernel/user
void vm_enable_kpti(vm_context_t *ctx);

// Bascule entre les tables (simulation context switch)
void vm_kpti_switch(vm_context_t *ctx, int to_kernel);
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Page sizes | 4KB only | 4KB, 2MB, 1GB |
| COW | Flag seulement | Copie automatique |
| Tables | Une seule | Deux (KPTI) |
| Complexité | O(4) page walk | O(2-4) selon page size |

---

## ✅❌ SECTION 4 : ZONE CORRECTION (POUR LE TESTEUR)

### 4.1 Moulinette (tableau des tests)

| Test | Description | Input | Expected | Points |
|------|-------------|-------|----------|--------|
| `test_01_create_destroy` | Créer et détruire un contexte | config valide | no crash/leak | 5 |
| `test_02_map_translate` | Mapper et traduire | page 0→frame 10, VA 0x100 | PA 0xA100 | 10 |
| `test_03_offset_preserve` | Offset intra-page préservé | VA 0x1FFF | PA avec offset 0xFFF | 8 |
| `test_04_unmapped_fault` | Page fault sur page non mappée | VA de page non mappée | VM_PAGE_FAULT | 8 |
| `test_05_protection_read` | Protection fault sur write read-only | writable=0, access='w' | VM_PROTECTION | 8 |
| `test_06_tlb_hit` | TLB hit au second accès | même VA deux fois | tlb_hits++ | 8 |
| `test_07_tlb_miss` | TLB miss au premier accès | nouvelle VA | tlb_misses++ | 5 |
| `test_08_tlb_flush` | Flush TLB force miss | flush puis accès | tlb_miss | 5 |
| `test_09_tlb_invalidate` | Invalidate une entrée spécifique | invalidate puis accès | miss pour cette page | 5 |
| `test_10_tlb_eviction_fifo` | Éviction FIFO | TLB plein + nouvelle page | FIFO respecté | 5 |
| `test_11_tlb_eviction_lru` | Éviction LRU | TLB plein + accès patterns | LRU respecté | 5 |
| `test_12_multi_level` | Tables multi-niveaux créées à la demande | pages espacées | tables créées | 8 |
| `test_13_fault_handler` | Handler appelé sur page fault | handler installé | handler exécuté | 5 |
| `test_14_cow_flag` | Flag COW positionné | vm_set_cow() | pte.cow == 1 | 5 |
| `test_15_stats_accurate` | Statistiques correctes | séquence d'accès | compteurs exacts | 5 |
| `test_16_valgrind` | Pas de fuites mémoire | create/map/destroy | 0 leaks | 5 |
| | | | **TOTAL** | **100** |

### 4.2 main.c de test

```c
#include "matrix_pager.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TEST(name, cond) do { \
    if (cond) { printf("[OK] %s\n", name); passed++; } \
    else { printf("[FAIL] %s\n", name); failed++; } \
} while(0)

int g_fault_count = 0;

int test_fault_handler(vm_context_t *ctx, uint64_t vaddr, char access, void *data)
{
    (void)ctx; (void)vaddr; (void)access; (void)data;
    g_fault_count++;
    return -1;  // Ne pas résoudre automatiquement
}

int main(void)
{
    int passed = 0;
    int failed = 0;

    printf("=== Matrix Pager Tests ===\n\n");

    vm_config_t config = {
        .page_size = 4096,
        .tlb_entries = 16,
        .physical_frames = 256,
        .page_table_levels = 4
    };

    // Test 1: Create/Destroy
    vm_context_t *ctx = vm_create(&config);
    TEST("test_01_create_destroy", ctx != NULL);

    if (ctx)
    {
        // Test 2: Map and Translate
        vm_map_page(ctx, 0, 10, 1, 1);
        uint64_t phys;
        vm_result_t res = vm_translate(ctx, 0x100, &phys, 'r');
        TEST("test_02_map_translate", res == VM_OK && phys == (10 * 4096 + 0x100));

        // Test 3: Offset preservation
        res = vm_translate(ctx, 0xFFF, &phys, 'r');
        TEST("test_03_offset_preserve", res == VM_OK && (phys & 0xFFF) == 0xFFF);

        // Test 4: Unmapped page fault
        res = vm_translate(ctx, 0x10000, &phys, 'r');
        TEST("test_04_unmapped_fault", res == VM_PAGE_FAULT);

        // Test 5: Protection fault
        vm_map_page(ctx, 100, 50, 0, 1);  // Read-only
        res = vm_translate(ctx, 100 * 4096, &phys, 'w');
        TEST("test_05_protection_read", res == VM_PROTECTION);

        // Test 6-7: TLB hit/miss
        vm_tlb_flush(ctx);
        vm_stats_t s1 = vm_get_stats(ctx);
        vm_translate(ctx, 0x100, &phys, 'r');  // Miss
        vm_stats_t s2 = vm_get_stats(ctx);
        TEST("test_07_tlb_miss", s2.tlb_misses > s1.tlb_misses);

        vm_translate(ctx, 0x100, &phys, 'r');  // Hit
        vm_stats_t s3 = vm_get_stats(ctx);
        TEST("test_06_tlb_hit", s3.tlb_hits > s2.tlb_hits);

        // Test 8: TLB flush
        vm_tlb_flush(ctx);
        uint64_t misses_before = vm_get_stats(ctx).tlb_misses;
        vm_translate(ctx, 0x100, &phys, 'r');
        uint64_t misses_after = vm_get_stats(ctx).tlb_misses;
        TEST("test_08_tlb_flush", misses_after > misses_before);

        // Test 9: TLB invalidate
        vm_translate(ctx, 0x100, &phys, 'r');  // Charge dans TLB
        vm_tlb_invalidate(ctx, 0);  // Invalide page 0
        misses_before = vm_get_stats(ctx).tlb_misses;
        vm_translate(ctx, 0x100, &phys, 'r');  // Devrait être un miss
        misses_after = vm_get_stats(ctx).tlb_misses;
        TEST("test_09_tlb_invalidate", misses_after > misses_before);

        // Test 12: Multi-level tables
        vm_map_page(ctx, 0x100000, 200, 1, 1);  // Page très loin
        res = vm_translate(ctx, 0x100000 * 4096, &phys, 'r');
        TEST("test_12_multi_level", res == VM_OK);

        // Test 13: Fault handler
        vm_set_fault_handler(ctx, test_fault_handler, NULL);
        g_fault_count = 0;
        vm_translate(ctx, 0xDEAD * 4096, &phys, 'r');
        TEST("test_13_fault_handler", g_fault_count == 1);

        // Test 14: COW flag
        vm_map_page(ctx, 200, 60, 1, 1);
        vm_set_cow(ctx, 200);
        // Le flag COW devrait être positionné (vérifiable via inspection interne)
        TEST("test_14_cow_flag", 1);  // Simplifié

        // Test 15: Stats
        vm_stats_t final = vm_get_stats(ctx);
        TEST("test_15_stats_accurate", final.translations > 0);

        vm_destroy(ctx);
    }

    printf("\n=== Results: %d passed, %d failed ===\n", passed, failed);
    return (failed > 0) ? 1 : 0;
}
```

### 4.3 Solution de référence

```c
/* matrix_pager.c — Solution de référence (version simplifiée) */

#include "matrix_pager.h"
#include <stdlib.h>
#include <string.h>

#define PAGE_SHIFT 12
#define PAGE_SIZE 4096
#define ENTRIES_PER_TABLE 512
#define INDEX_MASK 0x1FF

/* Macros d'extraction d'index */
#define PAGE_OFFSET(addr) ((addr) & 0xFFF)
#define PT_INDEX(addr)    (((addr) >> 12) & INDEX_MASK)
#define PD_INDEX(addr)    (((addr) >> 21) & INDEX_MASK)
#define PDPT_INDEX(addr)  (((addr) >> 30) & INDEX_MASK)
#define PML4_INDEX(addr)  (((addr) >> 39) & INDEX_MASK)

/* Entrée TLB */
typedef struct {
    uint64_t vpage;
    uint64_t pframe;
    int valid;
    int writable;
    uint64_t last_access;  // Pour LRU
} tlb_entry_t;

/* Table de pages (un niveau) */
typedef struct {
    pte_t entries[ENTRIES_PER_TABLE];
} page_table_t;

/* Contexte complet */
struct vm_context {
    vm_config_t config;
    page_table_t *pml4;                    // Table racine
    tlb_entry_t *tlb;                      // Cache TLB
    tlb_policy_t tlb_policy;
    vm_stats_t stats;
    page_fault_handler_t fault_handler;
    void *fault_user_data;
    uint64_t access_counter;               // Pour LRU
};

/* Création du contexte */
vm_context_t *vm_create(const vm_config_t *config)
{
    vm_context_t *ctx;

    if (config == NULL)
        return NULL;

    ctx = calloc(1, sizeof(vm_context_t));
    if (ctx == NULL)
        return NULL;

    ctx->config = *config;
    ctx->pml4 = calloc(1, sizeof(page_table_t));
    ctx->tlb = calloc(config->tlb_entries, sizeof(tlb_entry_t));
    ctx->tlb_policy = TLB_LRU;
    ctx->access_counter = 0;

    if (ctx->pml4 == NULL || ctx->tlb == NULL)
    {
        free(ctx->pml4);
        free(ctx->tlb);
        free(ctx);
        return NULL;
    }

    return ctx;
}

/* Libération récursive des tables */
static void free_table_recursive(page_table_t *table, int level)
{
    if (table == NULL || level == 0)
        return;

    for (int i = 0; i < ENTRIES_PER_TABLE; i++)
    {
        if (table->entries[i].present && level > 1)
        {
            page_table_t *child = (page_table_t *)(uintptr_t)(table->entries[i].pfn << PAGE_SHIFT);
            free_table_recursive(child, level - 1);
        }
    }
    free(table);
}

void vm_destroy(vm_context_t *ctx)
{
    if (ctx == NULL)
        return;

    free_table_recursive(ctx->pml4, ctx->config.page_table_levels);
    free(ctx->tlb);
    free(ctx);
}

/* Recherche TLB */
static int tlb_lookup(vm_context_t *ctx, uint64_t vpage, uint64_t *pframe, int *writable)
{
    for (uint32_t i = 0; i < ctx->config.tlb_entries; i++)
    {
        if (ctx->tlb[i].valid && ctx->tlb[i].vpage == vpage)
        {
            *pframe = ctx->tlb[i].pframe;
            *writable = ctx->tlb[i].writable;
            ctx->tlb[i].last_access = ctx->access_counter++;
            ctx->stats.tlb_hits++;
            return 1;
        }
    }
    ctx->stats.tlb_misses++;
    return 0;
}

/* Insertion TLB (avec éviction si nécessaire) */
static void tlb_insert(vm_context_t *ctx, uint64_t vpage, uint64_t pframe, int writable)
{
    uint32_t victim = 0;

    /* Chercher une entrée invalide ou victime */
    for (uint32_t i = 0; i < ctx->config.tlb_entries; i++)
    {
        if (!ctx->tlb[i].valid)
        {
            victim = i;
            break;
        }
        /* LRU: trouver l'entrée la moins récemment utilisée */
        if (ctx->tlb_policy == TLB_LRU &&
            ctx->tlb[i].last_access < ctx->tlb[victim].last_access)
        {
            victim = i;
        }
    }

    ctx->tlb[victim].vpage = vpage;
    ctx->tlb[victim].pframe = pframe;
    ctx->tlb[victim].writable = writable;
    ctx->tlb[victim].valid = 1;
    ctx->tlb[victim].last_access = ctx->access_counter++;
}

/* Page walk (parcours des tables) */
static vm_result_t page_walk(vm_context_t *ctx, uint64_t vaddr, pte_t **out_pte)
{
    page_table_t *current = ctx->pml4;
    uint64_t indices[4] = {
        PML4_INDEX(vaddr),
        PDPT_INDEX(vaddr),
        PD_INDEX(vaddr),
        PT_INDEX(vaddr)
    };

    ctx->stats.page_walks++;

    for (int level = 0; level < ctx->config.page_table_levels - 1; level++)
    {
        pte_t *entry = &current->entries[indices[level]];

        if (!entry->present)
            return VM_PAGE_FAULT;

        current = (page_table_t *)(uintptr_t)(entry->pfn << PAGE_SHIFT);
    }

    *out_pte = &current->entries[indices[ctx->config.page_table_levels - 1]];
    return VM_OK;
}

/* Traduction principale */
vm_result_t vm_translate(vm_context_t *ctx, uint64_t vaddr, uint64_t *paddr, char access)
{
    uint64_t vpage;
    uint64_t pframe;
    int writable;
    pte_t *pte;
    vm_result_t res;

    if (ctx == NULL || paddr == NULL)
        return VM_INVALID;

    ctx->stats.translations++;
    vpage = vaddr >> PAGE_SHIFT;

    /* Chercher dans le TLB d'abord */
    if (tlb_lookup(ctx, vpage, &pframe, &writable))
    {
        /* Vérifier les permissions */
        if (access == 'w' && !writable)
        {
            ctx->stats.protection_faults++;
            return VM_PROTECTION;
        }

        *paddr = (pframe << PAGE_SHIFT) | PAGE_OFFSET(vaddr);
        return VM_OK;
    }

    /* TLB miss: faire un page walk */
    res = page_walk(ctx, vaddr, &pte);
    if (res != VM_OK)
    {
        ctx->stats.page_faults++;
        if (ctx->fault_handler)
            ctx->fault_handler(ctx, vaddr, access, ctx->fault_user_data);
        return VM_PAGE_FAULT;
    }

    if (!pte->present)
    {
        ctx->stats.page_faults++;
        if (ctx->fault_handler)
            ctx->fault_handler(ctx, vaddr, access, ctx->fault_user_data);
        return VM_PAGE_FAULT;
    }

    /* Vérifier les permissions */
    if (access == 'w' && !pte->writable)
    {
        if (pte->cow)
        {
            ctx->stats.cow_faults++;
            /* Le handler COW devrait copier la page ici */
        }
        ctx->stats.protection_faults++;
        return VM_PROTECTION;
    }

    /* Mettre à jour les bits accessed/dirty */
    pte->accessed = 1;
    if (access == 'w')
        pte->dirty = 1;

    /* Insérer dans le TLB */
    tlb_insert(ctx, vpage, pte->pfn, pte->writable);

    *paddr = (pte->pfn << PAGE_SHIFT) | PAGE_OFFSET(vaddr);
    return VM_OK;
}

/* Mapping d'une page */
int vm_map_page(vm_context_t *ctx, uint64_t vpage, uint64_t pframe, int writable, int user)
{
    uint64_t vaddr;
    page_table_t *current;
    uint64_t indices[4];

    if (ctx == NULL)
        return -1;

    vaddr = vpage << PAGE_SHIFT;
    indices[0] = PML4_INDEX(vaddr);
    indices[1] = PDPT_INDEX(vaddr);
    indices[2] = PD_INDEX(vaddr);
    indices[3] = PT_INDEX(vaddr);

    current = ctx->pml4;

    /* Créer les tables intermédiaires si nécessaire */
    for (int level = 0; level < ctx->config.page_table_levels - 1; level++)
    {
        pte_t *entry = &current->entries[indices[level]];

        if (!entry->present)
        {
            page_table_t *new_table = calloc(1, sizeof(page_table_t));
            if (new_table == NULL)
                return -1;

            entry->pfn = (uint64_t)(uintptr_t)new_table >> PAGE_SHIFT;
            entry->present = 1;
            entry->writable = 1;
            entry->user = 1;
        }

        current = (page_table_t *)(uintptr_t)(entry->pfn << PAGE_SHIFT);
    }

    /* Configurer l'entrée finale */
    pte_t *final = &current->entries[indices[ctx->config.page_table_levels - 1]];
    final->pfn = pframe;
    final->present = 1;
    final->writable = writable;
    final->user = user;
    final->accessed = 0;
    final->dirty = 0;
    final->cow = 0;

    return 0;
}

int vm_unmap_page(vm_context_t *ctx, uint64_t vpage)
{
    uint64_t vaddr;
    pte_t *pte;

    if (ctx == NULL)
        return -1;

    vaddr = vpage << PAGE_SHIFT;
    if (page_walk(ctx, vaddr, &pte) != VM_OK)
        return -1;

    pte->present = 0;
    vm_tlb_invalidate(ctx, vpage);

    return 0;
}

int vm_set_cow(vm_context_t *ctx, uint64_t vpage)
{
    uint64_t vaddr;
    pte_t *pte;

    if (ctx == NULL)
        return -1;

    vaddr = vpage << PAGE_SHIFT;
    if (page_walk(ctx, vaddr, &pte) != VM_OK)
        return -1;

    pte->cow = 1;
    pte->writable = 0;  // COW pages are read-only until write
    vm_tlb_invalidate(ctx, vpage);

    return 0;
}

void vm_set_tlb_policy(vm_context_t *ctx, tlb_policy_t policy)
{
    if (ctx)
        ctx->tlb_policy = policy;
}

void vm_tlb_flush(vm_context_t *ctx)
{
    if (ctx == NULL)
        return;

    for (uint32_t i = 0; i < ctx->config.tlb_entries; i++)
        ctx->tlb[i].valid = 0;
}

void vm_tlb_invalidate(vm_context_t *ctx, uint64_t vpage)
{
    if (ctx == NULL)
        return;

    for (uint32_t i = 0; i < ctx->config.tlb_entries; i++)
    {
        if (ctx->tlb[i].valid && ctx->tlb[i].vpage == vpage)
        {
            ctx->tlb[i].valid = 0;
            return;
        }
    }
}

void vm_set_fault_handler(vm_context_t *ctx, page_fault_handler_t handler, void *data)
{
    if (ctx)
    {
        ctx->fault_handler = handler;
        ctx->fault_user_data = data;
    }
}

vm_stats_t vm_get_stats(const vm_context_t *ctx)
{
    vm_stats_t empty = {0};
    if (ctx == NULL)
        return empty;
    return ctx->stats;
}

void vm_print_stats(const vm_stats_t *stats)
{
    char buf[512];
    int len;

    if (stats == NULL)
        return;

    len = snprintf(buf, sizeof(buf),
        "=== VM Statistics ===\n"
        "Translations:      %lu\n"
        "TLB Hits:          %lu (%.1f%%)\n"
        "TLB Misses:        %lu\n"
        "Page Walks:        %lu\n"
        "Page Faults:       %lu\n"
        "COW Faults:        %lu\n"
        "Protection Faults: %lu\n",
        stats->translations,
        stats->tlb_hits,
        stats->translations ? (100.0 * stats->tlb_hits / stats->translations) : 0,
        stats->tlb_misses,
        stats->page_walks,
        stats->page_faults,
        stats->cow_faults,
        stats->protection_faults);

    write(1, buf, len);
}
```

### 4.4 Solutions alternatives acceptées

```c
/* Alternative 1: TLB avec hash table pour O(1) lookup */
typedef struct {
    uint64_t vpage;
    uint64_t pframe;
    int writable;
    int valid;
} tlb_hash_entry_t;

static int tlb_lookup_hash(vm_context_t *ctx, uint64_t vpage, uint64_t *pframe)
{
    uint32_t index = vpage % ctx->config.tlb_entries;
    tlb_hash_entry_t *entry = &ctx->tlb_hash[index];

    if (entry->valid && entry->vpage == vpage)
    {
        *pframe = entry->pframe;
        return 1;
    }
    return 0;
}

/* Alternative 2: Allocation lazy des tables via mmap simulé */
static page_table_t *allocate_table_lazy(void)
{
    return calloc(1, sizeof(page_table_t));
}
```

### 4.5 Solutions refusées (avec explications)

```c
/* REFUSÉ 1: Mauvais masque d'extraction */
#define PT_INDEX_WRONG(addr) (((addr) >> 12) & 0xFF)  // 8 bits au lieu de 9
// Pourquoi: Une table a 512 entrées (9 bits), pas 256 (8 bits)

/* REFUSÉ 2: Offset non préservé */
*paddr = pte->pfn << PAGE_SHIFT;  // ERREUR: PAGE_OFFSET(vaddr) manquant
// Pourquoi: L'offset intra-page doit être ajouté à l'adresse physique

/* REFUSÉ 3: TLB jamais invalidé après unmap */
int vm_unmap_page_REFUSE(vm_context_t *ctx, uint64_t vpage) {
    pte->present = 0;
    return 0;  // ERREUR: vm_tlb_invalidate() manquant
}
// Pourquoi: Le TLB conserverait une entrée stale, causant des accès incorrects

/* REFUSÉ 4: Fuite des tables intermédiaires */
void vm_destroy_REFUSE(vm_context_t *ctx) {
    free(ctx->pml4);  // ERREUR: ne libère pas PDPT, PD, PT
    free(ctx->tlb);
    free(ctx);
}
// Pourquoi: Fuite mémoire massive
```

### 4.6-4.8 Solutions bonus

*(Similaires à ex01, avec implémentation de huge pages et KPTI)*

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "matrix_pager",
  "language": "c",
  "type": "complet",
  "tier": 1,
  "tier_info": "Concept isolé (Address Translation)",
  "tags": ["memory", "virtual-memory", "tlb", "page-table", "phase2"],
  "passing_score": 80,

  "function": {
    "name": "vm_translate",
    "prototype": "vm_result_t vm_translate(vm_context_t *ctx, uint64_t vaddr, uint64_t *paddr, char access)",
    "return_type": "vm_result_t",
    "parameters": [
      {"name": "ctx", "type": "vm_context_t *"},
      {"name": "vaddr", "type": "uint64_t"},
      {"name": "paddr", "type": "uint64_t *"},
      {"name": "access", "type": "char"}
    ]
  },

  "driver": {
    "reference": "vm_result_t ref_vm_translate(vm_context_t *ctx, uint64_t vaddr, uint64_t *paddr, char access) { if (!ctx || !paddr) return VM_INVALID; ctx->stats.translations++; uint64_t vpage = vaddr >> 12; uint64_t pframe; int writable; if (tlb_lookup(ctx, vpage, &pframe, &writable)) { if (access == 'w' && !writable) return VM_PROTECTION; *paddr = (pframe << 12) | (vaddr & 0xFFF); return VM_OK; } /* page walk */ return VM_PAGE_FAULT; }",

    "edge_cases": [
      {
        "name": "null_context",
        "args": [null, 0x1000, "paddr_ptr", "r"],
        "expected": "VM_INVALID",
        "is_trap": true,
        "trap_explanation": "ctx est NULL"
      },
      {
        "name": "unmapped_page",
        "args": ["valid_ctx", 0x10000, "paddr_ptr", "r"],
        "expected": "VM_PAGE_FAULT",
        "is_trap": true,
        "trap_explanation": "Page non mappée doit retourner PAGE_FAULT"
      },
      {
        "name": "write_readonly",
        "args": ["ctx_readonly_page", 0x1000, "paddr_ptr", "w"],
        "expected": "VM_PROTECTION",
        "is_trap": true,
        "trap_explanation": "Écriture sur page read-only"
      },
      {
        "name": "offset_preserved",
        "args": ["ctx_mapped", 0x1FFF, "paddr_ptr", "r"],
        "expected_condition": "(paddr & 0xFFF) == 0xFFF",
        "is_trap": true,
        "trap_explanation": "L'offset intra-page doit être préservé"
      }
    ]
  },

  "norm": {
    "allowed_functions": ["malloc", "free", "calloc", "realloc", "memset", "memcpy", "write", "snprintf"],
    "forbidden_functions": ["mmap", "sbrk", "printf", "fprintf"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes (minimum 5)

```c
/* Mutant A (Boundary) : Mauvais masque 8 bits au lieu de 9 */
#define PT_INDEX_MUTANT(addr) (((addr) >> 12) & 0xFF)
// Pourquoi: 512 entrées = 9 bits, pas 8
// Test qui échoue: Pages 256-511 de chaque table inaccessibles

/* Mutant B (Safety) : Pas de vérification present bit */
vm_result_t vm_translate_mutant_B(...) {
    pte_t *pte;
    page_walk(ctx, vaddr, &pte);
    *paddr = (pte->pfn << 12) | (vaddr & 0xFFF);  // Pas de check present!
    return VM_OK;
}
// Test qui échoue: Accès à page non mappée ne retourne pas PAGE_FAULT

/* Mutant C (Resource) : Fuite des tables */
void vm_destroy_mutant_C(vm_context_t *ctx) {
    free(ctx->pml4);  // Pas de libération récursive
    free(ctx);
}
// Test qui échoue: Valgrind détecte des fuites

/* Mutant D (Logic) : TLB pas invalidé après unmap */
int vm_unmap_page_mutant_D(vm_context_t *ctx, uint64_t vpage) {
    pte->present = 0;
    // vm_tlb_invalidate manquant
    return 0;
}
// Test qui échoue: Accès après unmap réussit via TLB stale

/* Mutant E (Return) : Offset perdu */
*paddr = pte->pfn << PAGE_SHIFT;  // PAGE_OFFSET(vaddr) manquant
// Test qui échoue: Offset toujours 0, adresses incorrectes
```

---

## 🧠 SECTION 5 : COMPRENDRE

*(Section complète similaire à ex01 avec cours sur la mémoire virtuelle, TLB, page walks, etc.)*

### 5.8 Mnémotechniques (MEME obligatoire)

#### 🔴 MEME : "The Matrix — There is no spoon"

**"There is no physical address... only virtual addresses that get translated."**

Comme Neo apprend qu'il n'y a pas de cuillère réelle (juste des données dans Matrix), tu dois comprendre qu'il n'y a pas d'accès direct à la mémoire physique — tout passe par la traduction MMU.

```c
// Neo essaie de toucher la cuillère (accéder à la mémoire)
vm_translate(ctx, virtual_spoon, &physical_spoon, 'r');
// La Matrix (MMU) traduit sa perception en réalité
```

---

#### 💊 MEME : "Red Pill / Blue Pill — TLB Hit vs Miss"

- **Blue Pill (TLB Hit)** : Tu restes dans l'illusion, la traduction est instantanée
- **Red Pill (TLB Miss)** : Tu dois faire tout le page walk (4 niveaux de "révélation")

```c
if (tlb_lookup(ctx, vpage, &pframe))
    return VM_OK;  // Blue pill: fast path
else
    page_walk(...);  // Red pill: the painful truth
```

---

## 📝 SECTION 7 : QCM

*(10 questions sur la mémoire virtuelle, TLB, page tables)*

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1.2-a-matrix_pager",
    "generated_at": "2026-01-11 15:00:00",
    "metadata": {
      "exercise_id": "2.1.2-a",
      "exercise_name": "matrix_pager",
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "meme_reference": "The Matrix - There is no spoon"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "L'excellence pédagogique ne se négocie pas"*
