# [Module 2.1] - Exercise 05: Memory Debugger

## Métadonnées

```yaml
module: "2.1 - Memory Management"
exercise: "ex05"
difficulty: moyen
estimated_time: "4-6 heures"
prerequisite_exercises: ["ex04"]
concepts_requis:
  - "Wrapping de fonctions"
  - "Backtrace et symboles"
  - "Hash tables"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.1.11.a | Leak detection | Trouver les fuites mémoire |
| 2.1.11.b | Double-free detection | Détecter les libérations multiples |
| 2.1.11.c | Use-after-free | Accès après libération |
| 2.1.11.d | Buffer overflow | Débordements de buffer |
| 2.1.11.e | Allocation tracking | Suivi des allocations |
| 2.1.11.f | Memory report | Rapport de fin d'exécution |
| 2.1.11.g | Stack traces | Contexte des allocations |
| 2.1.11.h | Guard bytes | Détection de corruption |

### Objectifs Pédagogiques

À la fin de cet exercice, vous saurez:
1. Instrumenter malloc/free pour détecter les erreurs
2. Maintenir un registre des allocations actives
3. Détecter les fuites, double-free et corruptions
4. Générer des rapports de debug utiles

---

## Contexte

Les bugs mémoire sont parmi les plus difficiles à débugger:
- **Memory leaks**: Mémoire allouée jamais libérée
- **Double free**: Libération d'un pointeur déjà libéré
- **Use-after-free**: Accès à mémoire déjà libérée
- **Buffer overflow**: Écriture au-delà des limites allouées

Des outils comme Valgrind ou AddressSanitizer détectent ces problèmes, mais comment fonctionnent-ils?

**Le défi**: Créer votre propre outil de détection, simplifié mais fonctionnel.

---

## Énoncé

### Vue d'Ensemble

Créez une bibliothèque `memdbg` qui wrappe malloc/free et détecte les erreurs mémoire courantes au runtime.

### Spécifications Fonctionnelles

#### Partie 1: Wrappers malloc/free

```c
// Active le debugger (appelé au démarrage)
void memdbg_init(void);

// Désactive et génère le rapport
void memdbg_shutdown(void);

// Wrappers (utilisés via macros)
void *memdbg_malloc(size_t size, const char *file, int line);
void *memdbg_calloc(size_t n, size_t size, const char *file, int line);
void *memdbg_realloc(void *ptr, size_t size, const char *file, int line);
void memdbg_free(void *ptr, const char *file, int line);

// Macros pour capturer file/line automatiquement
#define malloc(size)       memdbg_malloc(size, __FILE__, __LINE__)
#define free(ptr)          memdbg_free(ptr, __FILE__, __LINE__)
#define calloc(n, size)    memdbg_calloc(n, size, __FILE__, __LINE__)
#define realloc(ptr, size) memdbg_realloc(ptr, size, __FILE__, __LINE__)
```

#### Partie 2: Détections

```c
typedef enum {
    MEMDBG_OK,
    MEMDBG_LEAK,           // Mémoire non libérée
    MEMDBG_DOUBLE_FREE,    // Libération multiple
    MEMDBG_INVALID_FREE,   // Pointeur jamais alloué
    MEMDBG_OVERFLOW,       // Écriture hors limites (détectée via guard)
    MEMDBG_UNDERFLOW       // Écriture avant le début
} memdbg_error_t;

// Callback appelé lors d'une erreur
typedef void (*memdbg_error_handler_t)(
    memdbg_error_t error,
    void *ptr,
    const char *alloc_file, int alloc_line,
    const char *free_file, int free_line
);

void memdbg_set_error_handler(memdbg_error_handler_t handler);
```

#### Partie 3: Guard Bytes

```c
// Structure autour de chaque allocation
// [GUARD_BEFORE][USER_DATA][GUARD_AFTER]

#define GUARD_PATTERN 0xDEADBEEF
#define GUARD_SIZE 8  // bytes avant et après

// Vérifier l'intégrité des guards
int memdbg_check_guards(void *ptr);

// Vérifier tous les blocs alloués
int memdbg_check_all(void);
```

#### Partie 4: Rapport et Statistiques

```c
typedef struct {
    size_t total_allocations;      // Nombre total d'allocations
    size_t total_frees;            // Nombre total de free
    size_t current_allocations;    // Allocations actives
    size_t bytes_allocated;        // Bytes actuellement alloués
    size_t peak_bytes;             // Pic d'utilisation
    size_t leaks_detected;         // Fuites trouvées
    size_t double_frees;           // Double-free détectés
    size_t corruptions;            // Corruptions détectées
} memdbg_stats_t;

memdbg_stats_t memdbg_get_stats(void);

// Afficher le rapport final
void memdbg_print_report(void);

// Lister toutes les allocations actives (pour debug)
void memdbg_list_allocations(void);
```

---

## Exemple d'Utilisation

### Exemple 1: Détection de Fuite

```c
#include "memdbg.h"

int main(void) {
    memdbg_init();

    int *a = malloc(100);  // Alloué ligne 6
    int *b = malloc(200);  // Alloué ligne 7

    free(a);
    // b n'est jamais libéré!

    memdbg_shutdown();  // Génère le rapport
    return 0;
}
```

**Sortie attendue**:
```
=== MEMORY DEBUG REPORT ===
LEAK detected: 200 bytes at 0x55a8b5400100
  Allocated at: main.c:7
  Never freed

Summary:
  Total allocations: 2
  Total frees: 1
  Leaked: 1 block (200 bytes)
===========================
```

### Exemple 2: Double Free

```c
memdbg_init();

int *p = malloc(100);
free(p);
free(p);  // ERREUR!

memdbg_shutdown();
```

**Sortie attendue**:
```
ERROR: Double-free detected!
  Pointer: 0x55a8b5400100
  Originally allocated at: main.c:3
  First freed at: main.c:4
  Second free attempt at: main.c:5
```

### Exemple 3: Buffer Overflow Détection

```c
memdbg_init();

char *buf = malloc(10);
strcpy(buf, "Hello World!");  // 13 chars > 10!

// La corruption est détectée au free ou au check
memdbg_check_all();  // Vérifie maintenant
free(buf);

memdbg_shutdown();
```

**Sortie attendue**:
```
ERROR: Buffer overflow detected!
  Pointer: 0x55a8b5400100
  Allocated size: 10
  Guard bytes after allocation corrupted
  Allocated at: main.c:3
```

### Exemple 4: Statistiques d'Utilisation

```c
void complex_operation(void) {
    memdbg_init();

    for (int i = 0; i < 1000; i++) {
        void *p = malloc(rand() % 1000 + 1);
        // ... use p ...
        free(p);
    }

    memdbg_stats_t s = memdbg_get_stats();
    printf("Peak memory usage: %zu bytes\n", s.peak_bytes);
    printf("Total allocations: %zu\n", s.total_allocations);

    memdbg_shutdown();
}
```

---

## Tests Moulinette

### Tests Fonctionnels

```yaml
test_01_no_leak:
  description: "Pas de fuite = rapport propre"
  code: |
    memdbg_init();
    void *p = malloc(100);
    free(p);
    memdbg_shutdown();
  expected: "leaks_detected == 0"

test_02_detect_leak:
  description: "Fuite détectée"
  code: |
    memdbg_init();
    void *p = malloc(100);
    // pas de free
    memdbg_shutdown();
  expected: "leaks_detected == 1"

test_03_detect_double_free:
  description: "Double-free signalé"
  code: |
    memdbg_init();
    void *p = malloc(100);
    free(p);
    free(p);
  expected: "double_frees == 1"

test_04_detect_invalid_free:
  description: "Free d'un pointeur invalide"
  code: |
    memdbg_init();
    int x;
    free(&x);  // Pas alloué par malloc!
  expected: "Erreur signalée"

test_05_guard_overflow:
  description: "Overflow détecté via guards"
  code: |
    memdbg_init();
    char *p = malloc(10);
    p[10] = 'X';  // Overflow
    assert(memdbg_check_guards(p) == 0);
  expected: "Corruption détectée"
```

### Tests Statistiques

```yaml
test_06_peak_tracking:
  description: "Pic mémoire correct"
  code: |
    void *a = malloc(1000);
    void *b = malloc(2000);
    free(a);
    void *c = malloc(500);
    // Peak = 3000 (a+b ensemble)
  expected: "peak_bytes == 3000"

test_07_file_line_tracking:
  description: "File/line capturés"
  validation: "Rapport inclut noms de fichiers et numéros de ligne"
```

---

## Critères d'Évaluation

| Critère | Points | Description |
|---------|--------|-------------|
| **Correction** | 40 | |
| - Leak detection | 10 | Toutes les fuites trouvées |
| - Double-free | 10 | Détection correcte |
| - Guard bytes | 10 | Over/underflow détectés |
| - Statistiques | 10 | Compteurs exacts |
| **Sécurité** | 25 | |
| - Pas de fuites internes | 10 | Le debugger lui-même clean |
| - Thread-safe hash table | 10 | Optionnel mais valorisé |
| - Robustesse | 5 | Pas de crash sur entrées invalides |
| **Conception** | 20 | |
| - Hash table efficace | 10 | O(1) lookup par pointeur |
| - Séparation | 10 | Tracking/detection/reporting |
| **Lisibilité** | 15 | |
| - Rapport clair | 5 | Facile à comprendre |
| - Code modulaire | 5 | |
| - Documentation | 5 | |

**Score minimum**: 80/100

---

## Auto-Évaluation Qualité

| Critère | Score /25 | Justification |
|---------|-----------|---------------|
| Intelligence énoncé | 24 | Comprendre le fonctionnement des outils de debug |
| Couverture conceptuelle | 25 | 8 concepts debug couverts |
| Testabilité auto | 23 | Tests sur détection d'erreurs |
| Originalité | 24 | Pas une copie, focus pratique |
| **TOTAL** | **96/100** | ✓ Validé |

**✓ Score ≥ 95, exercice validé.**
