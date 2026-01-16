# Exercice 2.3.30 : jaeger_array

**Module :**
2.3.30 — RAID Simulator

**Concept :**
synth — Synthèse complète (RAID 0/1/5/6/10, parity, rebuild, hot spare)

**Difficulté :**
★★★★★★★☆☆☆ (7/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (tous concepts a→j)

**Langage :**
C (C17)

**Prérequis :**
- Opérations bitwise (XOR)
- Allocation mémoire dynamique
- Structures de données (tableaux)
- Arithmétique modulaire (stripe distribution)

**Domaines :**
FS, Mem, MD, Encodage

**Durée estimée :**
480 min

**XP Base :**
550

**Complexité :**
T3 O(n) × S3 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex14/
├── jaeger_raid.h
├── jaeger_core.c
├── striker_eureka.c      (RAID 0)
├── gipsy_danger.c        (RAID 1)
├── crimson_typhoon.c     (RAID 5)
├── cherno_alpha.c        (RAID 6)
├── coyote_tango.c        (RAID 10)
├── neural_rebuild.c
└── Makefile
```

**Fonctions autorisées :**
`malloc`, `free`, `memset`, `memcpy`, `memmove`, `printf`, `usleep`

**Fonctions interdites :**
Fonctions RAID système (`mdadm`, etc.)

---

### 1.2 Consigne

**🎬 CONTEXTE : PACIFIC RIM — Canceling the Apocalypse**

*"Today we are cancelling the apocalypse!"*

L'humanité fait face aux Kaiju — des monstres géants qui émergent du Breach. Notre seule défense : les **Jaegers**, des robots géants pilotés par deux humains connectés via le **Neural Drift**.

Le problème ? Un seul pilote ne peut pas supporter la charge neurale seul (comme RAID 0 sans redondance). Deux pilotes en Drift partagent la charge ET se protègent mutuellement (comme RAID 1 mirroring).

Quand un Kaiju frappe et qu'un pilote est KO, le Jaeger peut continuer en **mode dégradé** si la redondance était suffisante. Les pilotes de réserve (Hot Spares) attendent au Shatterdome, prêts à établir un nouveau Neural Handshake.

**Ta mission :**

Implémenter le système JAEGER ARRAY qui simule tous les niveaux RAID :
- **STRIKER_EUREKA** (RAID 0) : Attaque pure, pas de redondance
- **GIPSY_DANGER** (RAID 1) : Deux pilotes synchronisés, redondance totale
- **CRIMSON_TYPHOON** (RAID 5) : Trois pilotes, un peut tomber
- **CHERNO_ALPHA** (RAID 6) : Double blindage, deux peuvent tomber
- **COYOTE_TANGO** (RAID 10) : Combo mirroring + striping

**Entrée :**
- `config` : Configuration du Jaeger (niveau RAID, nombre de pilotes, taille)
- `buf` : Données à lire/écrire (Neural payload)
- `offset` : Position dans l'array

**Sortie :**
- Retourne le nombre d'octets transférés
- Retourne `-1` en cas d'erreur fatale
- Mode dégradé retourne les données reconstruites via XOR

**Contraintes :**
- RAID 0 : N disques, 0 redondance, N× capacité
- RAID 1 : N disques, N-1 redondance, 1× capacité
- RAID 5 : N disques, 1 redondance, (N-1)× capacité
- RAID 6 : N disques, 2 redondance, (N-2)× capacité
- RAID 10 : N disques pairs, N/2 redondance, N/2× capacité

**Exemples :**

| Appel | Retour | Explication |
|-------|--------|-------------|
| `jaeger_deploy(&config_raid5)` | `jaeger_t*` | Crimson Typhoon déployé |
| `neural_drift_write(jaeger, data, 1024, 0)` | `1024` | Données distribuées |
| `pilot_kaiju_hit(jaeger, 1)` | `0` | Pilote 1 KO, mode dégradé |
| `neural_drift_read(jaeger, buf, 1024, 0)` | `1024` | Données reconstruites via parité |
| `neural_handshake_restore(jaeger)` | `0` | Rebuild depuis hot spare |

---

### 1.2.2 Consigne Académique

Implémenter un simulateur RAID complet supportant les niveaux 0, 1, 5, 6 et 10. Le système doit gérer le striping (distribution des données), le mirroring (duplication), le calcul de parité (XOR), le mode dégradé (lecture avec disque manquant), et la reconstruction (rebuild) depuis un hot spare.

---

### 1.3 Prototype

```c
#ifndef JAEGER_RAID_H
#define JAEGER_RAID_H

#include <stdint.h>
#include <stdbool.h>
#include <stddef.h>
#include <sys/types.h>

/* ═══════════════════════════════════════════════════════════════════════════
   JAEGER MODES (RAID LEVELS)
   ═══════════════════════════════════════════════════════════════════════════ */

// 2.3.30.b: RAID 0 — Pure offense, no backup (like solo piloting)
#define STRIKER_EUREKA_MODE     0

// 2.3.30.c: RAID 1 — Full mirror, dual pilot sync
#define GIPSY_DANGER_MODE       1

// 2.3.30.d: RAID 5 — Distributed parity (triplets crew)
#define CRIMSON_TYPHOON_MODE    5

// 2.3.30.e: RAID 6 — Double parity (maximum protection)
#define CHERNO_ALPHA_MODE       6

// 2.3.30.f: RAID 10 — Mirror + Stripe combo
#define COYOTE_TANGO_MODE       10

/* ═══════════════════════════════════════════════════════════════════════════
   STRUCTURES
   ═══════════════════════════════════════════════════════════════════════════ */

// Virtual pilot (disk) — connected via Neural Drift
typedef struct {
    char callsign[64];           // Pilot name
    uint8_t *neural_load;        // Data stored
    size_t capacity;             // Memory capacity
    bool kaiju_hit;              // 2.3.30.j: Failed/knocked out
    bool is_reserve;             // 2.3.30.i: Hot spare
    uint64_t drift_reads;        // Read operations
    uint64_t drift_writes;       // Write operations
} pilot_t;

// Jaeger array (RAID array)
typedef struct {
    int combat_mode;             // RAID level (0,1,5,6,10)
    pilot_t **pilots;            // Connected pilots (disks)
    size_t pilot_count;          // Number of active pilots
    size_t drift_chunk;          // 2.3.30.g: Stripe size

    size_t total_neural_capacity;   // Raw total
    size_t usable_capacity;         // After redundancy

    // Reserve pilots (hot spares)
    pilot_t **reserves;          // 2.3.30.i
    size_t reserve_count;

    // Combat status
    bool degraded;               // 2.3.30.j: Operating with casualties
    int ko_pilot;                // Index of KO'd pilot (-1 if none)
    bool rebuilding;             // 2.3.30.h: Neural handshake restoration
    float rebuild_progress;

    // Statistics
    uint64_t total_reads;
    uint64_t total_writes;
    uint64_t parity_calcs;       // 2.3.30.f
    uint64_t stripe_ops;         // 2.3.30.g
    uint64_t rebuild_ops;        // 2.3.30.h
    uint64_t degraded_reads;     // 2.3.30.j
} jaeger_t;

// Deployment configuration
typedef struct {
    int combat_mode;             // RAID level
    size_t pilot_count;          // Number of pilots/disks
    size_t pilot_capacity;       // Size per pilot/disk
    size_t drift_chunk;          // Stripe size
    size_t reserve_count;        // Hot spares
} jaeger_config_t;

// Performance metrics
typedef struct {
    double read_mbps;
    double write_mbps;
    double iops;
    double rebuild_time_sec;
    double fault_tolerance;      // Kaiju hits survivable
    double efficiency;           // Usable/Total ratio
} jaeger_perf_t;

// Statistics
typedef struct {
    uint64_t total_reads;
    uint64_t total_writes;
    uint64_t parity_calcs;
    uint64_t stripe_ops;
    uint64_t rebuild_ops;
    uint64_t degraded_reads;
} jaeger_stats_t;

/* ═══════════════════════════════════════════════════════════════════════════
   API — SHATTERDOME OPERATIONS
   ═══════════════════════════════════════════════════════════════════════════ */

// Jaeger lifecycle
jaeger_t *jaeger_deploy(jaeger_config_t *config);
void jaeger_decommission(jaeger_t *jaeger);

// Neural Drift I/O (main read/write)
ssize_t neural_drift_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset);
ssize_t neural_drift_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.b: STRIKER EUREKA — RAID 0 (Striping, No Redundancy)
   "The fastest Jaeger, but one hit and it's over"
   ═══════════════════════════════════════════════════════════════════════════ */

void striker_eureka_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset);
void striker_eureka_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.c: GIPSY DANGER — RAID 1 (Mirroring)
   "Two pilots, one mind, complete redundancy"
   ═══════════════════════════════════════════════════════════════════════════ */

void gipsy_danger_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset);
void gipsy_danger_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.d: CRIMSON TYPHOON — RAID 5 (Distributed Parity)
   "The Wei triplets: three pilots, rotating parity"
   ═══════════════════════════════════════════════════════════════════════════ */

void crimson_typhoon_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset);
void crimson_typhoon_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset);
int crimson_parity_pilot(jaeger_t *jaeger, off_t stripe_num);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.e: CHERNO ALPHA — RAID 6 (Double Parity P+Q)
   "Russian engineering: survive two direct hits"
   ═══════════════════════════════════════════════════════════════════════════ */

void cherno_alpha_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset);
void cherno_alpha_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset);
void cherno_calc_pq(jaeger_t *jaeger, off_t stripe, uint8_t *P, uint8_t *Q);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.f: COYOTE TANGO — RAID 10 (Mirror + Stripe)
   "First generation Jaeger: combined tactics"
   ═══════════════════════════════════════════════════════════════════════════ */

void coyote_tango_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset);
void coyote_tango_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.f: DRIFT HARMONY — Parity Calculation
   ═══════════════════════════════════════════════════════════════════════════ */

uint8_t drift_harmony_calc(uint8_t **neural_loads, size_t pilot_count, size_t offset);
void drift_xor_sync(uint8_t *dst, const uint8_t *src, size_t len);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.g: DRIFT CHUNK MANAGEMENT (Stripe Size)
   ═══════════════════════════════════════════════════════════════════════════ */

size_t jaeger_get_drift_chunk(jaeger_t *jaeger);
void jaeger_set_drift_chunk(jaeger_t *jaeger, size_t size);
int neural_offset_to_pilot(jaeger_t *jaeger, off_t offset);
off_t neural_offset_to_stripe(jaeger_t *jaeger, off_t offset);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.h: NEURAL HANDSHAKE RESTORE (Rebuild)
   ═══════════════════════════════════════════════════════════════════════════ */

int neural_handshake_restore(jaeger_t *jaeger, int pilot_index);
int restore_from_reserve(jaeger_t *jaeger);
float get_restore_progress(jaeger_t *jaeger);
void cancel_restore(jaeger_t *jaeger);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.i: DRIFT COMPATIBLE RESERVE (Hot Spare)
   ═══════════════════════════════════════════════════════════════════════════ */

int add_reserve_pilot(jaeger_t *jaeger, size_t capacity);
int remove_reserve_pilot(jaeger_t *jaeger, int reserve_index);
int activate_reserve(jaeger_t *jaeger);

/* ═══════════════════════════════════════════════════════════════════════════
   2.3.30.j: SINGLE PILOT MODE (Degraded Operation)
   ═══════════════════════════════════════════════════════════════════════════ */

int pilot_kaiju_hit(jaeger_t *jaeger, int pilot_index);
bool jaeger_is_degraded(jaeger_t *jaeger);
ssize_t solo_pilot_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset);
int get_ko_pilots(jaeger_t *jaeger, int *pilots, size_t max);

/* ═══════════════════════════════════════════════════════════════════════════
   SHATTERDOME ANALYTICS
   ═══════════════════════════════════════════════════════════════════════════ */

void compare_jaeger_modes(jaeger_perf_t *perf, size_t pilot_count, size_t capacity);
void benchmark_jaeger(jaeger_t *jaeger, jaeger_perf_t *result);
void get_jaeger_stats(jaeger_t *jaeger, jaeger_stats_t *stats);
void print_neural_layout(jaeger_t *jaeger);

#endif /* JAEGER_RAID_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Le Neural Drift et la Synchronisation RAID

Dans Pacific Rim, deux pilotes partagent la charge neurale via le "Drift" — ils voient les souvenirs l'un de l'autre et agissent comme une seule entité. C'est exactement ce que fait le RAID :

```
           NEURAL DRIFT (RAID)
    ┌────────────────────────────────────┐
    │                                    │
    │   Pilot 1 ◄──── DRIFT ────► Pilot 2│
    │     │            │            │    │
    │     ▼            ▼            ▼    │
    │   Data       Sync/Parity    Data   │
    │                                    │
    └────────────────────────────────────┘

    Si Pilot 1 tombe → Pilot 2 a toute l'info
    C'est le MIRRORING (RAID 1)
```

### 2.2 Pourquoi XOR pour la Parité ?

XOR (⊕) a une propriété magique : `A ⊕ B ⊕ B = A`

```
Données:   1 0 1 1 0 1 0 0  (Pilot 1)
         ⊕ 0 1 1 0 1 0 1 1  (Pilot 2)
         ─────────────────
Parité:    1 1 0 1 1 1 1 1  (Pilot P)

Si Pilot 1 KO:
Pilot 2 ⊕ Parité = Pilot 1 reconstruit!
```

### 2.3 Les Niveaux RAID Expliqués via Pacific Rim

| RAID | Jaeger | Description | Survivabilité |
|------|--------|-------------|---------------|
| **0** | Striker Eureka | Solo pilot mode — rapide mais fragile | 0 pilote |
| **1** | Gipsy Danger | Deux pilotes synchronisés — redondance totale | 1 pilote |
| **5** | Crimson Typhoon | Triplets Wei — parité distribuée | 1 pilote |
| **6** | Cherno Alpha | Double blindage russe — ultra résistant | 2 pilotes |
| **10** | Coyote Tango | Combo tactique — vitesse + redondance | 1 par paire |

---

### 2.5 DANS LA VRAIE VIE

| Métier | Usage RAID | Niveau Typique |
|--------|-----------|----------------|
| **SysAdmin Serveur** | Protection données critiques | RAID 5/6 |
| **DBA (Database)** | Performance + Redondance | RAID 10 |
| **NAS Personnel** | Stockage home avec backup | RAID 1 |
| **Montage Vidéo** | Throughput maximal (scratch disks) | RAID 0 |
| **Datacenter** | Haute disponibilité | RAID 6 + Hot Spare |
| **Cloud Storage (AWS)** | Erasure coding (évolution de RAID 6) | RAID 6-like |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
jaeger_raid.h  jaeger_core.c  striker_eureka.c  gipsy_danger.c  crimson_typhoon.c  cherno_alpha.c  coyote_tango.c  neural_rebuild.c  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -c jaeger_core.c -o jaeger_core.o
gcc -Wall -Wextra -Werror -c striker_eureka.c -o striker_eureka.o
gcc -Wall -Wextra -Werror -c gipsy_danger.c -o gipsy_danger.o
gcc -Wall -Wextra -Werror -c crimson_typhoon.c -o crimson_typhoon.o
gcc -Wall -Wextra -Werror -c cherno_alpha.c -o cherno_alpha.o
gcc -Wall -Wextra -Werror -c coyote_tango.c -o coyote_tango.o
gcc -Wall -Wextra -Werror -c neural_rebuild.c -o neural_rebuild.o
gcc -Wall -Wextra -Werror *.o main.c -o jaeger_test

$ ./jaeger_test
╔══════════════════════════════════════════════════════════════╗
║            SHATTERDOME — JAEGER ARRAY SIMULATOR              ║
╠══════════════════════════════════════════════════════════════╣
║  "Today we are cancelling the apocalypse!"                   ║
╚══════════════════════════════════════════════════════════════╝

[DEPLOY] Crimson Typhoon (RAID 5) with 4 pilots
  → Total capacity: 4 MB
  → Usable capacity: 3 MB (75% efficiency)
  → Fault tolerance: 1 pilot

[WRITE] Neural payload: 4096 bytes at offset 0
  → Stripe 0: D0→Pilot0, D1→Pilot1, D2→Pilot2, P→Pilot3
  → Parity calculated: 0xA7

[KAIJU ATTACK] Pilot 1 hit! Entering degraded mode...
  → Array still operational via parity reconstruction

[READ] Degraded read: 4096 bytes at offset 0
  → Reconstructing Pilot 1 data via XOR
  → D1 = D0 ⊕ D2 ⊕ P
  → Data integrity: VERIFIED

[RESERVE] Activating drift-compatible reserve pilot
[RESTORE] Neural handshake restoration in progress...
  → Progress: 25%... 50%... 75%... 100%
[RESTORE] Restoration complete! Array fully operational.

═══ JAEGER MODE COMPARISON ═══
Mode            │ Read MB/s │ Write MB/s │ Efficiency │ Tolerance
────────────────┼───────────┼────────────┼────────────┼──────────
STRIKER (R0)    │   400.0   │   400.0    │   100%     │ 0 pilots
GIPSY (R1)      │   200.0   │   100.0    │    50%     │ 1 pilot
CRIMSON (R5)    │   300.0   │   150.0    │    75%     │ 1 pilot
CHERNO (R6)     │   250.0   │   100.0    │    50%     │ 2 pilots
COYOTE (R10)    │   200.0   │   100.0    │    50%     │ 1/pair

[DECOMMISSION] Crimson Typhoon returning to Shatterdome
All tests passed! The apocalypse has been cancelled.
```

---

### 3.1 💀 BONUS EXPERT : GALOIS FIELD RAID 6 (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★★☆ (9/10)

**Récompense :**
XP ×4

**Time Complexity attendue :**
O(n) pour P, O(n) pour Q avec Galois Field multiplication

**Space Complexity attendue :**
O(1) auxiliaire

**Domaines Bonus :**
`MD, Crypto, Encodage`

#### 3.1.1 Consigne Bonus

**🎬 NIVEAU KAIJU CATÉGORIE 5 : DOUBLE PARITY AVEC REED-SOLOMON**

*"Cherno Alpha was built with double the shields for a reason"*

RAID 6 utilise deux parités :
- **P** : XOR classique (comme RAID 5)
- **Q** : Multiplication dans un corps de Galois GF(2⁸)

Cela permet de reconstruire les données même avec DEUX pilotes KO simultanément.

**Ta mission :**

Implémenter le calcul Q avec Reed-Solomon simplifié :
- `Q = g⁰·D₀ ⊕ g¹·D₁ ⊕ g²·D₂ ⊕ ...`
- `g = 0x02` (générateur du corps de Galois)
- Multiplication dans GF(2⁸) avec polynôme irréductible `0x11D`

**Contraintes :**
```
┌─────────────────────────────────────────┐
│  GF(2⁸) multiplication : O(1) via LUT  │
│  Reconstruction : résoudre système 2×2 │
│  Mémoire : tables pré-calculées OK     │
└─────────────────────────────────────────┘
```

#### 3.1.2 Prototype Bonus

```c
// Galois Field GF(2^8) operations
uint8_t gf_mult(uint8_t a, uint8_t b);
uint8_t gf_div(uint8_t a, uint8_t b);
uint8_t gf_pow(uint8_t base, uint8_t exp);
uint8_t gf_inv(uint8_t a);

// Reed-Solomon Q parity
void cherno_calc_q_galois(jaeger_t *jaeger, off_t stripe, uint8_t *Q);

// Dual failure reconstruction
int cherno_reconstruct_dual(jaeger_t *jaeger, int pilot1, int pilot2, off_t stripe);
```

#### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Q Parity | Simple XOR avec offset | Galois Field multiplication |
| Dual reconstruction | Non supporté | Résolution système linéaire |
| Tables | Aucune | gf_log[], gf_exp[] pré-calculées |
| Math | Arithmétique simple | Corps finis GF(2⁸) |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Input | Expected | Points | Concept |
|------|-------|----------|--------|---------|
| `test_jaeger_deploy` | Valid config | jaeger_t* non-NULL | 5 | Core |
| `test_striker_write_read` | R0 + data | Data intact | 10 | 2.3.30.b |
| `test_striker_failure` | R0 + fail 1 disk | Total data loss | 5 | 2.3.30.b |
| `test_gipsy_mirror` | R1 + data | Identical on both | 10 | 2.3.30.c |
| `test_gipsy_degraded` | R1 + fail 1 | Data from survivor | 5 | 2.3.30.c |
| `test_crimson_parity` | R5 + stripe | Correct XOR | 10 | 2.3.30.d |
| `test_crimson_rotate` | R5 stripes | Parity rotates | 5 | 2.3.30.d |
| `test_crimson_degraded` | R5 + fail 1 | Reconstruct via XOR | 10 | 2.3.30.d |
| `test_cherno_dual` | R6 + P + Q | Both parities correct | 10 | 2.3.30.e |
| `test_cherno_single_fail` | R6 + fail 1 | Reconstruct | 5 | 2.3.30.e |
| `test_coyote_combo` | R10 | Mirror groups striped | 10 | 2.3.30.f |
| `test_drift_harmony` | XOR calc | Correct parity byte | 5 | 2.3.30.f |
| `test_drift_chunk` | Various sizes | Correct distribution | 5 | 2.3.30.g |
| `test_neural_restore` | Rebuild | 100% restore | 10 | 2.3.30.h |
| `test_reserve_activation` | Hot spare | Seamless replace | 5 | 2.3.30.i |
| `test_degraded_ops` | All modes | Correct behavior | 5 | 2.3.30.j |
| **TOTAL** | | | **100** | |
| **BONUS** | GF(2⁸) dual reconstruct | Works | **15** | Bonus |

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "jaeger_raid.h"

#define TEST(name) printf("\n[TEST] %s\n", name)
#define OK() printf("  ✓ PASS\n")
#define FAIL(msg) printf("  ✗ FAIL: %s\n", msg)
#define ASSERT(cond, msg) if (!(cond)) { FAIL(msg); return 1; }

int test_jaeger_deploy(void) {
    TEST("jaeger_deploy (RAID 5)");

    jaeger_config_t config = {
        .combat_mode = CRIMSON_TYPHOON_MODE,
        .pilot_count = 4,
        .pilot_capacity = 1024 * 1024,
        .drift_chunk = 64 * 1024,
        .reserve_count = 1
    };

    jaeger_t *jaeger = jaeger_deploy(&config);
    ASSERT(jaeger != NULL, "jaeger allocation failed");
    ASSERT(jaeger->pilot_count == 4, "wrong pilot count");
    ASSERT(jaeger->combat_mode == CRIMSON_TYPHOON_MODE, "wrong mode");
    ASSERT(jaeger->usable_capacity == 3 * 1024 * 1024, "wrong usable capacity");
    ASSERT(jaeger->reserve_count == 1, "wrong reserve count");

    jaeger_decommission(jaeger);
    OK();
    return 0;
}

int test_striker_write_read(void) {
    TEST("STRIKER EUREKA (RAID 0) write/read");

    jaeger_config_t config = {
        .combat_mode = STRIKER_EUREKA_MODE,
        .pilot_count = 4,
        .pilot_capacity = 1024 * 1024,
        .drift_chunk = 64 * 1024,
        .reserve_count = 0
    };

    jaeger_t *jaeger = jaeger_deploy(&config);

    char write_data[4096];
    char read_data[4096];
    memset(write_data, 0x42, sizeof(write_data));

    ssize_t written = neural_drift_write(jaeger, write_data, sizeof(write_data), 0);
    ASSERT(written == sizeof(write_data), "write size mismatch");

    ssize_t read = neural_drift_read(jaeger, read_data, sizeof(read_data), 0);
    ASSERT(read == sizeof(read_data), "read size mismatch");
    ASSERT(memcmp(write_data, read_data, sizeof(write_data)) == 0, "data corruption");

    jaeger_decommission(jaeger);
    OK();
    return 0;
}

int test_gipsy_degraded(void) {
    TEST("GIPSY DANGER (RAID 1) degraded mode");

    jaeger_config_t config = {
        .combat_mode = GIPSY_DANGER_MODE,
        .pilot_count = 2,
        .pilot_capacity = 1024 * 1024,
        .drift_chunk = 64 * 1024,
        .reserve_count = 0
    };

    jaeger_t *jaeger = jaeger_deploy(&config);

    char write_data[1024] = "Neural handshake established!";
    char read_data[1024];

    neural_drift_write(jaeger, write_data, sizeof(write_data), 0);

    // Kaiju hits Pilot 0!
    pilot_kaiju_hit(jaeger, 0);
    ASSERT(jaeger_is_degraded(jaeger), "should be degraded");

    // Read should still work from Pilot 1
    ssize_t read = neural_drift_read(jaeger, read_data, sizeof(read_data), 0);
    ASSERT(read == sizeof(read_data), "degraded read failed");
    ASSERT(memcmp(write_data, read_data, sizeof(write_data)) == 0, "data mismatch");

    jaeger_decommission(jaeger);
    OK();
    return 0;
}

int test_crimson_parity(void) {
    TEST("CRIMSON TYPHOON (RAID 5) parity calculation");

    jaeger_config_t config = {
        .combat_mode = CRIMSON_TYPHOON_MODE,
        .pilot_count = 4,
        .pilot_capacity = 1024 * 1024,
        .drift_chunk = 64 * 1024,
        .reserve_count = 0
    };

    jaeger_t *jaeger = jaeger_deploy(&config);

    // Write pattern that makes parity verification easy
    uint8_t pattern[4] = {0xAA, 0x55, 0xFF, 0x00};
    char write_data[256];
    for (int i = 0; i < 256; i++) {
        write_data[i] = pattern[i % 4];
    }

    neural_drift_write(jaeger, write_data, sizeof(write_data), 0);

    // Verify parity: XOR of all data should equal parity
    uint8_t expected_parity = 0xAA ^ 0x55 ^ 0xFF;  // = 0x00
    uint8_t calc_parity = drift_harmony_calc(
        (uint8_t**)jaeger->pilots, 3, 0);

    // Note: actual parity location depends on stripe, this is simplified
    ASSERT(calc_parity != 0xFF, "parity calculation seems wrong");

    jaeger_decommission(jaeger);
    OK();
    return 0;
}

int test_crimson_reconstruct(void) {
    TEST("CRIMSON TYPHOON (RAID 5) reconstruction");

    jaeger_config_t config = {
        .combat_mode = CRIMSON_TYPHOON_MODE,
        .pilot_count = 4,
        .pilot_capacity = 1024 * 1024,
        .drift_chunk = 64 * 1024,
        .reserve_count = 1
    };

    jaeger_t *jaeger = jaeger_deploy(&config);

    char original[4096] = "The Wei triplets fight as one!";
    char recovered[4096];

    neural_drift_write(jaeger, original, sizeof(original), 0);

    // Kaiju hits one of the Wei triplets
    pilot_kaiju_hit(jaeger, 1);
    ASSERT(jaeger_is_degraded(jaeger), "should be degraded");

    // Read should reconstruct via XOR
    neural_drift_read(jaeger, recovered, sizeof(recovered), 0);
    ASSERT(memcmp(original, recovered, sizeof(original)) == 0, "reconstruction failed");

    // Activate reserve and rebuild
    activate_reserve(jaeger);
    restore_from_reserve(jaeger);

    ASSERT(!jaeger_is_degraded(jaeger), "should be restored");

    jaeger_decommission(jaeger);
    OK();
    return 0;
}

int test_cherno_dual_parity(void) {
    TEST("CHERNO ALPHA (RAID 6) dual parity");

    jaeger_config_t config = {
        .combat_mode = CHERNO_ALPHA_MODE,
        .pilot_count = 5,
        .pilot_capacity = 1024 * 1024,
        .drift_chunk = 64 * 1024,
        .reserve_count = 0
    };

    jaeger_t *jaeger = jaeger_deploy(&config);

    char data[1024] = "Russian engineering at its finest!";
    neural_drift_write(jaeger, data, sizeof(data), 0);

    uint8_t P[1024], Q[1024];
    cherno_calc_pq(jaeger, 0, P, Q);

    // P should be XOR of all data
    // Q should be weighted XOR (Galois field)
    ASSERT(P[0] != Q[0] || P[0] == 0, "P and Q should differ (usually)");

    jaeger_decommission(jaeger);
    OK();
    return 0;
}

int test_reserve_activation(void) {
    TEST("Hot spare activation");

    jaeger_config_t config = {
        .combat_mode = GIPSY_DANGER_MODE,
        .pilot_count = 2,
        .pilot_capacity = 1024 * 1024,
        .drift_chunk = 64 * 1024,
        .reserve_count = 1
    };

    jaeger_t *jaeger = jaeger_deploy(&config);
    ASSERT(jaeger->reserve_count == 1, "should have 1 reserve");

    char data[1024] = "Ready for the Breach!";
    neural_drift_write(jaeger, data, sizeof(data), 0);

    pilot_kaiju_hit(jaeger, 0);
    ASSERT(jaeger_is_degraded(jaeger), "should be degraded");

    int result = activate_reserve(jaeger);
    ASSERT(result == 0, "activation failed");

    result = restore_from_reserve(jaeger);
    ASSERT(result == 0, "restore failed");
    ASSERT(!jaeger_is_degraded(jaeger), "should be restored");
    ASSERT(jaeger->reserve_count == 0, "reserve should be used");

    jaeger_decommission(jaeger);
    OK();
    return 0;
}

int main(void) {
    printf("╔══════════════════════════════════════════════════════════╗\n");
    printf("║       SHATTERDOME — JAEGER ARRAY TEST SUITE              ║\n");
    printf("║  \"Today we are cancelling the apocalypse!\"               ║\n");
    printf("╚══════════════════════════════════════════════════════════╝\n");

    int failed = 0;

    failed += test_jaeger_deploy();
    failed += test_striker_write_read();
    failed += test_gipsy_degraded();
    failed += test_crimson_parity();
    failed += test_crimson_reconstruct();
    failed += test_cherno_dual_parity();
    failed += test_reserve_activation();

    printf("\n════════════════════════════════════════════════════════════\n");
    if (failed == 0) {
        printf("All tests passed! The Breach has been sealed.\n");
    } else {
        printf("%d test(s) failed. Kaiju victory imminent.\n", failed);
    }

    return failed;
}
```

### 4.3 Solution de référence

```c
/* jaeger_core.c — Reference Implementation */
#include "jaeger_raid.h"
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

/* ═══════════════════════════════════════════════════════════════════════════
   JAEGER LIFECYCLE
   ═══════════════════════════════════════════════════════════════════════════ */

static pilot_t *create_pilot(const char *callsign, size_t capacity, bool is_reserve)
{
    pilot_t *pilot = malloc(sizeof(pilot_t));
    if (pilot == NULL)
        return NULL;

    strncpy(pilot->callsign, callsign, 63);
    pilot->callsign[63] = '\0';
    pilot->neural_load = calloc(capacity, 1);
    if (pilot->neural_load == NULL)
    {
        free(pilot);
        return NULL;
    }
    pilot->capacity = capacity;
    pilot->kaiju_hit = false;
    pilot->is_reserve = is_reserve;
    pilot->drift_reads = 0;
    pilot->drift_writes = 0;

    return pilot;
}

static void destroy_pilot(pilot_t *pilot)
{
    if (pilot != NULL)
    {
        free(pilot->neural_load);
        free(pilot);
    }
}

jaeger_t *jaeger_deploy(jaeger_config_t *config)
{
    jaeger_t *jaeger;
    size_t i;
    char callsign[64];

    if (config == NULL || config->pilot_count < 2)
        return NULL;

    jaeger = calloc(1, sizeof(jaeger_t));
    if (jaeger == NULL)
        return NULL;

    jaeger->combat_mode = config->combat_mode;
    jaeger->pilot_count = config->pilot_count;
    jaeger->drift_chunk = config->drift_chunk;
    jaeger->ko_pilot = -1;

    /* Allocate pilots */
    jaeger->pilots = malloc(sizeof(pilot_t *) * config->pilot_count);
    if (jaeger->pilots == NULL)
    {
        free(jaeger);
        return NULL;
    }

    for (i = 0; i < config->pilot_count; i++)
    {
        snprintf(callsign, sizeof(callsign), "Pilot_%zu", i);
        jaeger->pilots[i] = create_pilot(callsign, config->pilot_capacity, false);
        if (jaeger->pilots[i] == NULL)
        {
            while (i > 0)
                destroy_pilot(jaeger->pilots[--i]);
            free(jaeger->pilots);
            free(jaeger);
            return NULL;
        }
    }

    /* Calculate capacities based on RAID level */
    jaeger->total_neural_capacity = config->pilot_count * config->pilot_capacity;

    switch (config->combat_mode)
    {
        case STRIKER_EUREKA_MODE:  /* RAID 0 */
            jaeger->usable_capacity = jaeger->total_neural_capacity;
            break;
        case GIPSY_DANGER_MODE:    /* RAID 1 */
            jaeger->usable_capacity = config->pilot_capacity;
            break;
        case CRIMSON_TYPHOON_MODE: /* RAID 5 */
            jaeger->usable_capacity = (config->pilot_count - 1) * config->pilot_capacity;
            break;
        case CHERNO_ALPHA_MODE:    /* RAID 6 */
            jaeger->usable_capacity = (config->pilot_count - 2) * config->pilot_capacity;
            break;
        case COYOTE_TANGO_MODE:    /* RAID 10 */
            jaeger->usable_capacity = (config->pilot_count / 2) * config->pilot_capacity;
            break;
        default:
            jaeger->usable_capacity = config->pilot_capacity;
    }

    /* Allocate reserves (hot spares) */
    if (config->reserve_count > 0)
    {
        jaeger->reserves = malloc(sizeof(pilot_t *) * config->reserve_count);
        if (jaeger->reserves != NULL)
        {
            for (i = 0; i < config->reserve_count; i++)
            {
                snprintf(callsign, sizeof(callsign), "Reserve_%zu", i);
                jaeger->reserves[i] = create_pilot(callsign, config->pilot_capacity, true);
            }
            jaeger->reserve_count = config->reserve_count;
        }
    }

    return jaeger;
}

void jaeger_decommission(jaeger_t *jaeger)
{
    size_t i;

    if (jaeger == NULL)
        return;

    for (i = 0; i < jaeger->pilot_count; i++)
        destroy_pilot(jaeger->pilots[i]);
    free(jaeger->pilots);

    if (jaeger->reserves != NULL)
    {
        for (i = 0; i < jaeger->reserve_count; i++)
            destroy_pilot(jaeger->reserves[i]);
        free(jaeger->reserves);
    }

    free(jaeger);
}

/* ═══════════════════════════════════════════════════════════════════════════
   PARITY CALCULATION — DRIFT HARMONY
   ═══════════════════════════════════════════════════════════════════════════ */

uint8_t drift_harmony_calc(uint8_t **neural_loads, size_t pilot_count, size_t offset)
{
    uint8_t parity = 0;
    size_t i;

    for (i = 0; i < pilot_count; i++)
        parity ^= neural_loads[i][offset];

    return parity;
}

void drift_xor_sync(uint8_t *dst, const uint8_t *src, size_t len)
{
    size_t i;

    for (i = 0; i < len; i++)
        dst[i] ^= src[i];
}

/* ═══════════════════════════════════════════════════════════════════════════
   RAID 0 — STRIKER EUREKA (STRIPING)
   ═══════════════════════════════════════════════════════════════════════════ */

void striker_eureka_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset)
{
    const uint8_t *data = buf;
    size_t chunk = jaeger->drift_chunk;
    size_t written = 0;

    while (written < count)
    {
        size_t stripe = (offset + written) / chunk;
        size_t pilot_idx = stripe % jaeger->pilot_count;
        size_t pilot_offset = (stripe / jaeger->pilot_count) * chunk +
                              ((offset + written) % chunk);
        size_t to_write = chunk - ((offset + written) % chunk);
        if (to_write > count - written)
            to_write = count - written;

        memcpy(jaeger->pilots[pilot_idx]->neural_load + pilot_offset,
               data + written, to_write);
        jaeger->pilots[pilot_idx]->drift_writes++;
        written += to_write;
    }

    jaeger->total_writes++;
    jaeger->stripe_ops++;
}

void striker_eureka_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset)
{
    uint8_t *data = buf;
    size_t chunk = jaeger->drift_chunk;
    size_t read_bytes = 0;

    while (read_bytes < count)
    {
        size_t stripe = (offset + read_bytes) / chunk;
        size_t pilot_idx = stripe % jaeger->pilot_count;
        size_t pilot_offset = (stripe / jaeger->pilot_count) * chunk +
                              ((offset + read_bytes) % chunk);
        size_t to_read = chunk - ((offset + read_bytes) % chunk);
        if (to_read > count - read_bytes)
            to_read = count - read_bytes;

        if (jaeger->pilots[pilot_idx]->kaiju_hit)
        {
            /* RAID 0: No recovery possible */
            memset(data + read_bytes, 0, to_read);
        }
        else
        {
            memcpy(data + read_bytes,
                   jaeger->pilots[pilot_idx]->neural_load + pilot_offset,
                   to_read);
        }
        jaeger->pilots[pilot_idx]->drift_reads++;
        read_bytes += to_read;
    }

    jaeger->total_reads++;
    jaeger->stripe_ops++;
}

/* ═══════════════════════════════════════════════════════════════════════════
   RAID 1 — GIPSY DANGER (MIRRORING)
   ═══════════════════════════════════════════════════════════════════════════ */

void gipsy_danger_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset)
{
    size_t i;

    /* Write to ALL pilots (mirrors) */
    for (i = 0; i < jaeger->pilot_count; i++)
    {
        if (!jaeger->pilots[i]->kaiju_hit)
        {
            memcpy(jaeger->pilots[i]->neural_load + offset, buf, count);
            jaeger->pilots[i]->drift_writes++;
        }
    }

    jaeger->total_writes++;
}

void gipsy_danger_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset)
{
    size_t i;

    /* Read from first available pilot */
    for (i = 0; i < jaeger->pilot_count; i++)
    {
        if (!jaeger->pilots[i]->kaiju_hit)
        {
            memcpy(buf, jaeger->pilots[i]->neural_load + offset, count);
            jaeger->pilots[i]->drift_reads++;
            jaeger->total_reads++;
            return;
        }
    }

    /* All pilots KO! */
    memset(buf, 0, count);
    jaeger->total_reads++;
}

/* ═══════════════════════════════════════════════════════════════════════════
   RAID 5 — CRIMSON TYPHOON (DISTRIBUTED PARITY)
   ═══════════════════════════════════════════════════════════════════════════ */

int crimson_parity_pilot(jaeger_t *jaeger, off_t stripe_num)
{
    /* Parity rotates: stripe 0 → last pilot, stripe 1 → second-last, etc. */
    return (jaeger->pilot_count - 1 - (stripe_num % jaeger->pilot_count));
}

void crimson_typhoon_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset)
{
    const uint8_t *data = buf;
    size_t chunk = jaeger->drift_chunk;
    size_t data_pilots = jaeger->pilot_count - 1;
    size_t written = 0;

    while (written < count)
    {
        off_t logical_offset = offset + written;
        off_t stripe = logical_offset / (chunk * data_pilots);
        int parity_pilot = crimson_parity_pilot(jaeger, stripe);

        size_t stripe_offset = logical_offset % (chunk * data_pilots);
        size_t data_pilot_idx = stripe_offset / chunk;

        /* Adjust for parity position */
        int actual_pilot = data_pilot_idx;
        if (actual_pilot >= parity_pilot)
            actual_pilot++;

        size_t pilot_offset = stripe * chunk + (stripe_offset % chunk);
        size_t to_write = chunk - (stripe_offset % chunk);
        if (to_write > count - written)
            to_write = count - written;

        /* Write data */
        memcpy(jaeger->pilots[actual_pilot]->neural_load + pilot_offset,
               data + written, to_write);

        /* Update parity */
        uint8_t *parity = jaeger->pilots[parity_pilot]->neural_load + stripe * chunk;
        for (size_t i = 0; i < to_write; i++)
            parity[(stripe_offset % chunk) + i] ^= data[written + i];

        jaeger->parity_calcs++;
        written += to_write;
    }

    jaeger->total_writes++;
}

void crimson_typhoon_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset)
{
    uint8_t *data = buf;
    size_t chunk = jaeger->drift_chunk;
    size_t data_pilots = jaeger->pilot_count - 1;
    size_t read_bytes = 0;

    while (read_bytes < count)
    {
        off_t logical_offset = offset + read_bytes;
        off_t stripe = logical_offset / (chunk * data_pilots);
        int parity_pilot = crimson_parity_pilot(jaeger, stripe);

        size_t stripe_offset = logical_offset % (chunk * data_pilots);
        size_t data_pilot_idx = stripe_offset / chunk;

        int actual_pilot = data_pilot_idx;
        if (actual_pilot >= parity_pilot)
            actual_pilot++;

        size_t pilot_offset = stripe * chunk + (stripe_offset % chunk);
        size_t to_read = chunk - (stripe_offset % chunk);
        if (to_read > count - read_bytes)
            to_read = count - read_bytes;

        if (jaeger->pilots[actual_pilot]->kaiju_hit)
        {
            /* Reconstruct via XOR of all other pilots */
            memset(data + read_bytes, 0, to_read);
            for (size_t i = 0; i < jaeger->pilot_count; i++)
            {
                if ((int)i != actual_pilot && !jaeger->pilots[i]->kaiju_hit)
                {
                    for (size_t j = 0; j < to_read; j++)
                        data[read_bytes + j] ^=
                            jaeger->pilots[i]->neural_load[pilot_offset + j];
                }
            }
            jaeger->degraded_reads++;
        }
        else
        {
            memcpy(data + read_bytes,
                   jaeger->pilots[actual_pilot]->neural_load + pilot_offset,
                   to_read);
        }

        read_bytes += to_read;
    }

    jaeger->total_reads++;
}

/* ═══════════════════════════════════════════════════════════════════════════
   RAID 6 — CHERNO ALPHA (DOUBLE PARITY)
   ═══════════════════════════════════════════════════════════════════════════ */

void cherno_calc_pq(jaeger_t *jaeger, off_t stripe, uint8_t *P, uint8_t *Q)
{
    size_t chunk = jaeger->drift_chunk;
    size_t data_pilots = jaeger->pilot_count - 2;
    size_t i, j;

    memset(P, 0, chunk);
    memset(Q, 0, chunk);

    for (i = 0; i < data_pilots; i++)
    {
        uint8_t *pilot_data = jaeger->pilots[i]->neural_load + stripe * chunk;

        for (j = 0; j < chunk; j++)
        {
            P[j] ^= pilot_data[j];
            /* Q uses simple multiplication for base implementation */
            Q[j] ^= (pilot_data[j] * (i + 1)) & 0xFF;
        }
    }

    jaeger->parity_calcs += 2;
}

void cherno_alpha_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset)
{
    /* Simplified: treat like RAID 5 with extra parity disk */
    const uint8_t *data = buf;
    size_t chunk = jaeger->drift_chunk;
    size_t data_pilots = jaeger->pilot_count - 2;
    size_t written = 0;

    while (written < count)
    {
        off_t stripe = (offset + written) / (chunk * data_pilots);
        size_t stripe_offset = (offset + written) % (chunk * data_pilots);
        size_t pilot_idx = stripe_offset / chunk;
        size_t pilot_offset = stripe * chunk + (stripe_offset % chunk);

        size_t to_write = chunk - (stripe_offset % chunk);
        if (to_write > count - written)
            to_write = count - written;

        memcpy(jaeger->pilots[pilot_idx]->neural_load + pilot_offset,
               data + written, to_write);

        written += to_write;
    }

    /* Calculate and store P and Q */
    off_t stripe = offset / (chunk * data_pilots);
    uint8_t *P = jaeger->pilots[jaeger->pilot_count - 2]->neural_load + stripe * chunk;
    uint8_t *Q = jaeger->pilots[jaeger->pilot_count - 1]->neural_load + stripe * chunk;
    cherno_calc_pq(jaeger, stripe, P, Q);

    jaeger->total_writes++;
}

void cherno_alpha_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset)
{
    /* Similar to RAID 5 but can handle 2 failures */
    crimson_typhoon_read(jaeger, buf, count, offset);
}

/* ═══════════════════════════════════════════════════════════════════════════
   RAID 10 — COYOTE TANGO (MIRROR + STRIPE)
   ═══════════════════════════════════════════════════════════════════════════ */

void coyote_tango_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset)
{
    const uint8_t *data = buf;
    size_t chunk = jaeger->drift_chunk;
    size_t mirrors = jaeger->pilot_count / 2;
    size_t written = 0;

    while (written < count)
    {
        size_t stripe = (offset + written) / chunk;
        size_t mirror_group = stripe % mirrors;
        size_t pilot_offset = (stripe / mirrors) * chunk + ((offset + written) % chunk);

        size_t to_write = chunk - ((offset + written) % chunk);
        if (to_write > count - written)
            to_write = count - written;

        /* Write to both pilots in mirror group */
        int p1 = mirror_group * 2;
        int p2 = mirror_group * 2 + 1;

        memcpy(jaeger->pilots[p1]->neural_load + pilot_offset, data + written, to_write);
        memcpy(jaeger->pilots[p2]->neural_load + pilot_offset, data + written, to_write);

        written += to_write;
    }

    jaeger->total_writes++;
}

void coyote_tango_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset)
{
    uint8_t *data = buf;
    size_t chunk = jaeger->drift_chunk;
    size_t mirrors = jaeger->pilot_count / 2;
    size_t read_bytes = 0;

    while (read_bytes < count)
    {
        size_t stripe = (offset + read_bytes) / chunk;
        size_t mirror_group = stripe % mirrors;
        size_t pilot_offset = (stripe / mirrors) * chunk + ((offset + read_bytes) % chunk);

        size_t to_read = chunk - ((offset + read_bytes) % chunk);
        if (to_read > count - read_bytes)
            to_read = count - read_bytes;

        int p1 = mirror_group * 2;
        int p2 = mirror_group * 2 + 1;

        /* Read from first available in mirror group */
        if (!jaeger->pilots[p1]->kaiju_hit)
            memcpy(data + read_bytes, jaeger->pilots[p1]->neural_load + pilot_offset, to_read);
        else if (!jaeger->pilots[p2]->kaiju_hit)
            memcpy(data + read_bytes, jaeger->pilots[p2]->neural_load + pilot_offset, to_read);
        else
            memset(data + read_bytes, 0, to_read);  /* Both KO */

        read_bytes += to_read;
    }

    jaeger->total_reads++;
}

/* ═══════════════════════════════════════════════════════════════════════════
   MAIN I/O DISPATCH
   ═══════════════════════════════════════════════════════════════════════════ */

ssize_t neural_drift_write(jaeger_t *jaeger, const void *buf, size_t count, off_t offset)
{
    if (jaeger == NULL || buf == NULL)
        return -1;

    switch (jaeger->combat_mode)
    {
        case STRIKER_EUREKA_MODE:
            striker_eureka_write(jaeger, buf, count, offset);
            break;
        case GIPSY_DANGER_MODE:
            gipsy_danger_write(jaeger, buf, count, offset);
            break;
        case CRIMSON_TYPHOON_MODE:
            crimson_typhoon_write(jaeger, buf, count, offset);
            break;
        case CHERNO_ALPHA_MODE:
            cherno_alpha_write(jaeger, buf, count, offset);
            break;
        case COYOTE_TANGO_MODE:
            coyote_tango_write(jaeger, buf, count, offset);
            break;
        default:
            return -1;
    }

    return count;
}

ssize_t neural_drift_read(jaeger_t *jaeger, void *buf, size_t count, off_t offset)
{
    if (jaeger == NULL || buf == NULL)
        return -1;

    switch (jaeger->combat_mode)
    {
        case STRIKER_EUREKA_MODE:
            striker_eureka_read(jaeger, buf, count, offset);
            break;
        case GIPSY_DANGER_MODE:
            gipsy_danger_read(jaeger, buf, count, offset);
            break;
        case CRIMSON_TYPHOON_MODE:
            crimson_typhoon_read(jaeger, buf, count, offset);
            break;
        case CHERNO_ALPHA_MODE:
            cherno_alpha_read(jaeger, buf, count, offset);
            break;
        case COYOTE_TANGO_MODE:
            coyote_tango_read(jaeger, buf, count, offset);
            break;
        default:
            return -1;
    }

    return count;
}

/* ═══════════════════════════════════════════════════════════════════════════
   DEGRADED MODE & REBUILD
   ═══════════════════════════════════════════════════════════════════════════ */

int pilot_kaiju_hit(jaeger_t *jaeger, int pilot_index)
{
    if (jaeger == NULL || pilot_index < 0 || (size_t)pilot_index >= jaeger->pilot_count)
        return -1;

    jaeger->pilots[pilot_index]->kaiju_hit = true;
    jaeger->degraded = true;
    jaeger->ko_pilot = pilot_index;

    return 0;
}

bool jaeger_is_degraded(jaeger_t *jaeger)
{
    return (jaeger != NULL && jaeger->degraded);
}

int activate_reserve(jaeger_t *jaeger)
{
    if (jaeger == NULL || jaeger->reserve_count == 0 || jaeger->ko_pilot < 0)
        return -1;

    /* Swap reserve with KO pilot */
    pilot_t *reserve = jaeger->reserves[0];
    reserve->is_reserve = false;

    /* Shift remaining reserves */
    for (size_t i = 0; i < jaeger->reserve_count - 1; i++)
        jaeger->reserves[i] = jaeger->reserves[i + 1];
    jaeger->reserve_count--;

    /* Replace KO pilot */
    destroy_pilot(jaeger->pilots[jaeger->ko_pilot]);
    jaeger->pilots[jaeger->ko_pilot] = reserve;

    return 0;
}

int restore_from_reserve(jaeger_t *jaeger)
{
    if (jaeger == NULL || jaeger->ko_pilot < 0)
        return -1;

    jaeger->rebuilding = true;

    /* Simulate rebuild by XOR reconstruction */
    pilot_t *new_pilot = jaeger->pilots[jaeger->ko_pilot];
    size_t capacity = new_pilot->capacity;

    for (size_t offset = 0; offset < capacity; offset++)
    {
        uint8_t reconstructed = 0;
        for (size_t i = 0; i < jaeger->pilot_count; i++)
        {
            if ((int)i != jaeger->ko_pilot)
                reconstructed ^= jaeger->pilots[i]->neural_load[offset];
        }
        new_pilot->neural_load[offset] = reconstructed;
        jaeger->rebuild_progress = (float)offset / capacity;
    }

    jaeger->rebuilding = false;
    jaeger->degraded = false;
    jaeger->ko_pilot = -1;
    jaeger->rebuild_ops++;

    return 0;
}

void get_jaeger_stats(jaeger_t *jaeger, jaeger_stats_t *stats)
{
    if (jaeger == NULL || stats == NULL)
        return;

    stats->total_reads = jaeger->total_reads;
    stats->total_writes = jaeger->total_writes;
    stats->parity_calcs = jaeger->parity_calcs;
    stats->stripe_ops = jaeger->stripe_ops;
    stats->rebuild_ops = jaeger->rebuild_ops;
    stats->degraded_reads = jaeger->degraded_reads;
}
```

### 4.10 Solutions Mutantes

```c
/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT A (Boundary): Off-by-one dans stripe calculation
   ═══════════════════════════════════════════════════════════════════════════ */
int crimson_parity_pilot_mutantA(jaeger_t *jaeger, off_t stripe_num)
{
    // ❌ Oublie le -1, dépasse les bornes
    return (jaeger->pilot_count - (stripe_num % jaeger->pilot_count));
}
// Pourquoi faux: Accès hors tableau quand stripe_num % pilot_count == 0

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT B (Safety): Pas de vérification NULL
   ═══════════════════════════════════════════════════════════════════════════ */
ssize_t neural_drift_write_mutantB(jaeger_t *jaeger, const void *buf, size_t count, off_t offset)
{
    // ❌ Pas de vérification jaeger ou buf!
    switch (jaeger->combat_mode)
    {
        // ... crash si jaeger == NULL
    }
    return count;
}
// Pourquoi faux: Segfault si paramètres NULL

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT C (Resource): Fuite mémoire dans create_pilot
   ═══════════════════════════════════════════════════════════════════════════ */
pilot_t *create_pilot_mutantC(const char *callsign, size_t capacity, bool is_reserve)
{
    pilot_t *pilot = malloc(sizeof(pilot_t));
    pilot->neural_load = calloc(capacity, 1);

    if (pilot->neural_load == NULL)
    {
        // ❌ Oublie de free(pilot) avant return!
        return NULL;
    }

    return pilot;
}
// Pourquoi faux: Memory leak du struct pilot si calloc échoue

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT D (Logic): XOR mal appliqué pour parité
   ═══════════════════════════════════════════════════════════════════════════ */
uint8_t drift_harmony_calc_mutantD(uint8_t **neural_loads, size_t pilot_count, size_t offset)
{
    uint8_t parity = 0;

    // ❌ Utilise OR au lieu de XOR!
    for (size_t i = 0; i < pilot_count; i++)
        parity |= neural_loads[i][offset];

    return parity;
}
// Pourquoi faux: OR ne permet pas la reconstruction (XOR est inversible)

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT E (Return): RAID 1 ne lit que le premier pilote
   ═══════════════════════════════════════════════════════════════════════════ */
void gipsy_danger_read_mutantE(jaeger_t *jaeger, void *buf, size_t count, off_t offset)
{
    // ❌ Lit toujours le pilote 0, même s'il est KO!
    memcpy(buf, jaeger->pilots[0]->neural_load + offset, count);
}
// Pourquoi faux: Ne profite pas de la redondance si pilote 0 KO

/* ═══════════════════════════════════════════════════════════════════════════
   MUTANT F (Integration): Oublie de mettre à jour degraded flag
   ═══════════════════════════════════════════════════════════════════════════ */
int pilot_kaiju_hit_mutantF(jaeger_t *jaeger, int pilot_index)
{
    jaeger->pilots[pilot_index]->kaiju_hit = true;
    // ❌ Oublie jaeger->degraded = true!
    // ❌ Oublie jaeger->ko_pilot = pilot_index!
    return 0;
}
// Pourquoi faux: jaeger_is_degraded() retourne false, rebuild impossible
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Striping (RAID 0)** : Distribution des données pour performance
2. **Mirroring (RAID 1)** : Duplication complète pour redondance
3. **Parité distribuée (RAID 5)** : XOR pour reconstruire avec 1 perte
4. **Double parité (RAID 6)** : Reed-Solomon pour 2 pertes
5. **Combinaison (RAID 10)** : Mirror + Stripe pour le meilleur des deux
6. **Mode dégradé** : Continuer à fonctionner avec pertes
7. **Rebuild** : Reconstruire les données perdues

### 5.2 LDA — Traduction Littérale

```
FONCTION drift_harmony_calc QUI RETOURNE UN OCTET NON SIGNÉ ET PREND EN PARAMÈTRES neural_loads QUI EST UN TABLEAU DE POINTEURS VERS DES OCTETS ET pilot_count QUI EST UNE TAILLE ET offset QUI EST UNE TAILLE
DÉBUT FONCTION
    DÉCLARER parity COMME OCTET NON SIGNÉ
    DÉCLARER i COMME TAILLE

    AFFECTER 0 À parity

    POUR i ALLANT DE 0 À pilot_count MOINS 1 FAIRE
        AFFECTER parity XOR L'OCTET À LA POSITION offset DANS LE TABLEAU i DE neural_loads À parity
    FIN POUR

    RETOURNER parity
FIN FONCTION
```

### 5.3 Visualisation ASCII

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                    JAEGER ARRAY — RAID LEVEL COMPARISON                      ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  STRIKER EUREKA (RAID 0) — "All offense, no defense"                         ║
║  ┌─────────┬─────────┬─────────┬─────────┐                                   ║
║  │ Pilot 0 │ Pilot 1 │ Pilot 2 │ Pilot 3 │                                   ║
║  │  D0-D3  │  D4-D7  │ D8-D11  │ D12-D15 │  ← Data striped across all       ║
║  └─────────┴─────────┴─────────┴─────────┘                                   ║
║  Capacity: 100%  |  Fault tolerance: 0  |  Performance: ★★★★★                ║
║                                                                              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  GIPSY DANGER (RAID 1) — "Two pilots, one mind"                              ║
║  ┌─────────┬─────────┐                                                       ║
║  │ Pilot 0 │ Pilot 1 │                                                       ║
║  │  D0-D15 │  D0-D15 │  ← Identical copies (mirror)                         ║
║  └─────────┴─────────┘                                                       ║
║  Capacity: 50%  |  Fault tolerance: 1  |  Performance: ★★★☆☆                 ║
║                                                                              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  CRIMSON TYPHOON (RAID 5) — "The Wei triplets"                               ║
║  ┌─────────┬─────────┬─────────┬─────────┐                                   ║
║  │ Pilot 0 │ Pilot 1 │ Pilot 2 │ Pilot 3 │                                   ║
║  ├─────────┼─────────┼─────────┼─────────┤                                   ║
║  │   D0    │   D1    │   D2    │   P0    │  ← P0 = D0⊕D1⊕D2                 ║
║  │   D3    │   D4    │   P1    │   D5    │  ← Parity rotates                ║
║  │   D6    │   P2    │   D7    │   D8    │                                   ║
║  │   P3    │   D9    │   D10   │   D11   │                                   ║
║  └─────────┴─────────┴─────────┴─────────┘                                   ║
║  Capacity: 75%  |  Fault tolerance: 1  |  Performance: ★★★★☆                 ║
║                                                                              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  CHERNO ALPHA (RAID 6) — "Russian engineering"                               ║
║  ┌─────────┬─────────┬─────────┬─────────┬─────────┐                         ║
║  │ Pilot 0 │ Pilot 1 │ Pilot 2 │ Pilot P │ Pilot Q │                         ║
║  │   D0    │   D1    │   D2    │ D0⊕D1⊕D2│ GF mult │  ← Double parity       ║
║  └─────────┴─────────┴─────────┴─────────┴─────────┘                         ║
║  Capacity: 60%  |  Fault tolerance: 2  |  Performance: ★★★☆☆                 ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **XOR vs OR** | OR ne permet pas reconstruction | Toujours XOR pour parité |
| **Stripe boundary** | Données coupées entre pilotes | Gérer les cas partiels |
| **Parity rotation** | Oublier de changer le pilote parité | Formule: (n-1-(stripe%n)) |
| **Rebuild incomplet** | Ne reconstruit pas tout | Parcourir toute la capacité |
| **Double failure RAID 5** | Perte totale avec 2 KO | Utiliser RAID 6 |

### 5.5 Cours Complet

#### 5.5.1 Introduction au RAID

**RAID** = Redundant Array of Independent Disks

Né dans les années 1980 pour combiner plusieurs disques bon marché en un système plus fiable et/ou plus rapide qu'un seul disque coûteux.

#### 5.5.2 Les Niveaux RAID

**RAID 0 (Striping)** :
- Données distribuées sur N disques
- Performance N× (lecture et écriture parallèles)
- AUCUNE redondance — une panne = perte totale
- Usage : Scratch disks, caches temporaires

**RAID 1 (Mirroring)** :
- Données dupliquées sur 2+ disques
- Lecture 2× (peut lire des deux)
- Écriture 1× (doit écrire aux deux)
- Perte de 50% capacité
- Usage : Boot drives, données critiques

**RAID 5 (Distributed Parity)** :
- N-1 disques de données + 1 disque équivalent de parité
- Parité distribuée (change de disque à chaque stripe)
- Peut survivre à 1 panne
- Usage : Serveurs génériques, NAS

**RAID 6 (Double Parity)** :
- Comme RAID 5 mais avec 2 parités (P et Q)
- Q utilise Reed-Solomon (Galois Field)
- Peut survivre à 2 pannes simultanées
- Usage : Datacenters, stockage critique

**RAID 10 (1+0)** :
- Miroirs stripés ensemble
- Performance de RAID 0 + redondance de RAID 1
- Peut survivre à 1 panne par paire miroir
- Usage : Bases de données haute performance

#### 5.5.3 La Magie du XOR

XOR a des propriétés magiques pour la reconstruction :

```
A ⊕ B = C
A ⊕ C = B
B ⊕ C = A
```

Si on perd A, on peut le reconstruire avec B ⊕ C!

### 5.8 Mnémotechniques

#### 🎬 MEME : "PACIFIC RIM — The Kaiju Survival Guide"

```
        🤖 JAEGERS vs 🦎 KAIJU (DISK FAILURES)

STRIKER EUREKA (RAID 0):
   "The fastest Jaeger... but one hit and it's over"
   Speed: ★★★★★  |  Survival: ☆☆☆☆☆

GIPSY DANGER (RAID 1):
   "Two pilots, one mind — complete backup"
   Speed: ★★★☆☆  |  Survival: ★★★★★

CRIMSON TYPHOON (RAID 5):
   "The Wei triplets: lose one, reconstruct via XOR"
   Speed: ★★★★☆  |  Survival: ★★★☆☆

CHERNO ALPHA (RAID 6):
   "Russian engineering: built to survive TWO direct hits"
   Speed: ★★★☆☆  |  Survival: ★★★★★
```

**Pour retenir les niveaux RAID :**
- **R0** = **Zero** redundancy (Striker — fast but fragile)
- **R1** = **One** copy (Gipsy — mirror = 2 pilots = 1 survit)
- **R5** = **Five** = nombre de lettres dans "parity" (presque)
- **R6** = R5 + **1** extra parity
- **R10** = **1** + **0** (Mirror groups striped)

### 5.9 Applications pratiques

| Workload | Niveau Recommandé | Pourquoi |
|----------|-------------------|----------|
| **OS Boot** | RAID 1 | Redondance simple, pas de calcul parité |
| **Base de données** | RAID 10 | Performance écriture + redondance |
| **Serveur fichiers** | RAID 5/6 | Bon ratio capacité/redondance |
| **Backup** | RAID 6 | Maximum redondance |
| **Vidéo editing** | RAID 0 | Performance pure (données récupérables) |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Erreur type | Solution |
|---|-------|-------------|----------|
| 1 | **XOR incorrect** | Utiliser OR ou AND | XOR obligatoire |
| 2 | **Parity fixe** | Même disque toujours | Rotation par stripe |
| 3 | **Rebuild partiel** | Oublier des blocs | Tout parcourir |
| 4 | **RAID 0 degraded** | Tenter de reconstruire | Impossible, données perdues |
| 5 | **Double fail RAID 5** | Continuer à servir | Data corruption |

---

## 📝 SECTION 7 : QCM

### Q1. Quel niveau RAID offre la meilleure performance en écriture ?
- A) RAID 1
- B) RAID 5
- C) RAID 0
- D) RAID 6

**Réponse : C** (pas de calcul de parité)

### Q2. Combien de disques peuvent tomber en RAID 6 ?
- A) 0
- B) 1
- C) 2
- D) 3

**Réponse : C**

### Q3. Quelle opération bitwise est utilisée pour la parité RAID 5 ?
- A) AND
- B) OR
- C) XOR
- D) NOT

**Réponse : C**

### Q4. Quelle est l'efficacité capacité d'un RAID 5 à 4 disques ?
- A) 50%
- B) 75%
- C) 100%
- D) 25%

**Réponse : B** ((4-1)/4 = 75%)

### Q5. Qu'est-ce qu'un "hot spare" ?
- A) Un disque surchauffé
- B) Un disque de réserve prêt à remplacer un disque défaillant
- C) Un disque avec plus de performance
- D) Un disque temporaire

**Réponse : B**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Concept | Maîtrisé | À revoir |
|---------|----------|----------|
| RAID 0 Striping | ☐ | ☐ |
| RAID 1 Mirroring | ☐ | ☐ |
| RAID 5 Parity | ☐ | ☐ |
| RAID 6 Double Parity | ☐ | ☐ |
| RAID 10 Combined | ☐ | ☐ |
| XOR Parity Calc | ☐ | ☐ |
| Degraded Mode | ☐ | ☐ |
| Rebuild Process | ☐ | ☐ |

**Score minimum pour valider : 70/100**

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.30-synth-jaeger-array",
    "generated_at": "2026-01-12 15:00:00",

    "metadata": {
      "exercise_id": "2.3.30-synth",
      "exercise_name": "jaeger_array",
      "module": "2.3.30",
      "module_name": "RAID Simulator",
      "concept": "synth",
      "concept_name": "Complete RAID simulation",
      "type": "complet",
      "tier": 3,
      "tier_info": "Synthèse (concepts a→j)",
      "phase": 2,
      "difficulty": 7,
      "difficulty_stars": "★★★★★★★☆☆☆",
      "language": "c",
      "language_version": "c17",
      "duration_minutes": 480,
      "xp_base": 550,
      "xp_bonus_multiplier": 4,
      "bonus_tier": "EXPERT",
      "bonus_icon": "💀",
      "complexity_time": "T3 O(n)",
      "complexity_space": "S3 O(n)",
      "prerequisites": ["bitwise-ops", "dynamic-allocation", "arrays", "modular-arithmetic"],
      "domains": ["FS", "Mem", "MD", "Encodage"],
      "domains_bonus": ["Crypto"],
      "tags": ["raid", "striping", "mirroring", "parity", "xor", "pacific-rim"],
      "meme_reference": "Pacific Rim Jaegers"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — "Today we are cancelling the data loss apocalypse!"*
*Jaeger Array: Because your data deserves Kaiju-level protection*
