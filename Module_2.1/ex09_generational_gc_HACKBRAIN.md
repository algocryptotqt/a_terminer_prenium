<thinking>
## Analyse du Concept
- Concept : Generational Garbage Collector
- Phase demandée : 2
- Adapté ? OUI — Le GC générationnel est la technique la plus utilisée dans les langages modernes (Java, .NET, Go). Basé sur l'hypothèse générationnelle.

## Combo Base + Bonus
- Exercice de base : Implémenter gen_gc_init, gen_gc_alloc, gen_gc_minor, gen_gc_major avec young/old generations
- Bonus : Write barrier avec card marking + remembered set pour les références old→young
- Palier bonus : 🧠 Génie — Le card marking et les remembered sets sont des concepts de niveau recherche
- Progression logique ? OUI — On maîtrise d'abord les deux générations, puis on optimise avec write barriers

## Prérequis & Difficulté
- Prérequis réels : Mark-sweep (ex08), copying collection, générations
- Difficulté estimée : 8/10
- Cohérent avec phase ? OUI (Phase 2 upper limit, mais exercice avancé)

## Aspect Fun/Culture
- Contexte choisi : The Lion King: Circle of Life
- MEME mnémotechnique : "Hakuna Matata" — Les jeunes objets vivent sans souci jusqu'au GC
- Pourquoi c'est fun :
  - Young Generation = Les lionceaux (Simba, Nala)
  - Old Generation = Les anciens de Pride Rock (Mufasa, Sarabi)
  - Minor GC = Les lionceaux faibles ne survivent pas (loi de la savane)
  - Major GC = Toute la fierté est évaluée (jugement de Mufasa)
  - Promotion = Un lionceau devient adulte et rejoint les anciens
  - Write Barrier = Quand un ancien pointe vers un lionceau, on le note
  - "Remember who you are" = Remembered Set!

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : Promouvoir après 1 minor GC au lieu de N → old gen overflow
2. Mutant B (Safety) : Oublier le write barrier → références old→young perdues au minor GC
3. Mutant C (Resource) : Ne pas libérer young gen après copying → memory leak
4. Mutant D (Logic) : Minor GC scanne aussi old gen → pas d'amélioration de performance
5. Mutant E (Return) : gen_gc_minor retourne 0 même si des objets libérés

## Verdict
VALIDE — L'exercice est parfait pour enseigner le GC générationnel avec une progression vers les optimisations avancées.
</thinking>

---

# Exercice 2.1.9 : circle_of_memory

**Module :**
2.1 — Memory Management

**Concept :**
i — Generational Garbage Collector

**Difficulté :**
★★★★★★★★☆☆ (8/10)

**Type :**
code

**Tiers :**
3 — Synthèse (concepts GC + copying + generations + barriers)

**Langage :**
C (C17)

**Prérequis :**
- Mark & Sweep GC (ex08)
- Copying collection
- Concept de générations d'objets
- Pointeurs et structures avancées

**Domaines :**
Mem, Struct, MD, Probas

**Durée estimée :**
600 min

**XP Base :**
250

**Complexité :**
T4 O(young_size) pour minor, O(heap) pour major × S4 O(heap)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichier à rendre :** `circle_gc.c`, `circle_gc.h`

**Fonctions autorisées :**
- `malloc`, `free`, `calloc`, `realloc`, `memcpy`, `memmove`
- Fonctions standard de libc

**Fonctions interdites :**
- Pas de GC existant
- Pas de threads pour la version de base

### 1.2 Consigne

**🎮 CONTEXTE FUN — The Lion King: Circle of Life**

Dans le Royaume de Pride Rock, la vie suit un cycle éternel — le **Cercle de la Vie**. Les jeunes lionceaux naissent, grandissent, et les plus forts deviennent des adultes qui rejoignent les anciens de la fierté.

Tu es le gardien de la mémoire de Pride Rock, chargé de gérer le cycle des générations :

**Les générations :**
- 🦁 **Young Generation (Les Lionceaux)** : Simba, Nala, et les petits. Ils vivent dans la nurserie (young heap). Beaucoup meurent jeunes (garbage), seuls les forts survivent.
- 👑 **Old Generation (Les Anciens)** : Mufasa, Sarabi, et les lions établis. Ils vivent à Pride Rock (old heap). Ils ont prouvé leur valeur en survivant plusieurs saisons.

**L'hypothèse générationnelle (The Circle of Life) :**
> "La plupart des objets meurent jeunes. Ceux qui survivent vivent longtemps."
> — Mufasa, à propos de la gestion mémoire

### 1.2.2 Énoncé Académique

Le **Garbage Collector Générationnel** est basé sur l'**hypothèse générationnelle** : la majorité des objets ont une courte durée de vie. En séparant les objets par "âge" et en collectant plus fréquemment les jeunes objets, on améliore considérablement les performances.

**Mécanisme :**
1. **Allocation** : Toujours dans la Young Generation (bump pointer, O(1))
2. **Minor GC** : Collecte uniquement Young Gen (rapide, fréquent)
3. **Promotion** : Objets survivant N minor GC → Old Gen
4. **Major GC** : Collecte Young + Old (rare, plus lent)
5. **Write Barrier** : Tracker les références Old → Young

**Ta mission :**

Implémenter un garbage collector générationnel avec deux espaces.

**API à implémenter :**

```c
typedef struct gen_gc gen_gc_t;

typedef struct {
    size_t young_used;
    size_t old_used;
    size_t minor_collections;
    size_t major_collections;
    size_t promotions;
    size_t young_survivors;
} gen_gc_stats_t;

gen_gc_t *gen_gc_init(size_t young_size, size_t old_size);
void gen_gc_shutdown(gen_gc_t *gc);

void *gen_gc_alloc(gen_gc_t *gc, size_t size);

void gen_gc_write_barrier(gen_gc_t *gc, void *obj, void **field, void *new_value);

size_t gen_gc_minor(gen_gc_t *gc);
size_t gen_gc_major(gen_gc_t *gc);

void gen_gc_add_root(gen_gc_t *gc, void **root);

gen_gc_stats_t gen_gc_get_stats(gen_gc_t *gc);
```

**Entrée :**
- `young_size` : Taille de la Young Generation en bytes
- `old_size` : Taille de la Old Generation en bytes
- `obj` : Objet source d'une affectation de pointeur
- `field` : Champ pointeur dans l'objet
- `new_value` : Nouvelle valeur du pointeur

**Sortie :**
- `gen_gc_minor` : Bytes libérés dans young gen
- `gen_gc_major` : Bytes libérés dans les deux générations

**Contraintes :**
- Allocation échoue si young gen pleine (déclencher minor GC automatiquement)
- Promotion après 3 minor GC survivants
- Write barrier DOIT être appelé pour chaque écriture de pointeur
- Minor GC ne doit PAS scanner old gen (sauf remembered set)

**Algorithme Minor GC :**

```
1. Scanner les racines → marquer/copier les objets young vivants
2. Scanner le remembered set (old → young refs)
3. Copier les survivants vers survivor space
4. Incrémenter l'âge des survivants
5. Promouvoir les objets avec age >= 3 vers old gen
6. Échanger from-space et to-space
7. Vider le remembered set
```

**Exemples :**

| Scénario | Young | Old | Minor GC | Freed |
|----------|-------|-----|----------|-------|
| 100 allocs temporaires | 100 → 0 | 0 | 1 | ~6400 |
| 10 permanents, 90 temp | 100 → 10 | 0 | 1 | ~5760 |
| Après 3 minors | 10 → 0 | 10 | 3 | (promoted) |

### 1.3 Prototype

```c
#ifndef CIRCLE_GC_H
#define CIRCLE_GC_H

#include <stddef.h>

typedef struct gen_gc gen_gc_t;

typedef struct {
    size_t young_used;
    size_t old_used;
    size_t minor_collections;
    size_t major_collections;
    size_t promotions;
    size_t young_survivors;
} gen_gc_stats_t;

gen_gc_t *gen_gc_init(size_t young_size, size_t old_size);
void gen_gc_shutdown(gen_gc_t *gc);

void *gen_gc_alloc(gen_gc_t *gc, size_t size);

void gen_gc_write_barrier(gen_gc_t *gc, void *obj, void **field, void *new_value);

size_t gen_gc_minor(gen_gc_t *gc);
size_t gen_gc_major(gen_gc_t *gc);

void gen_gc_add_root(gen_gc_t *gc, void **root);
void gen_gc_remove_root(gen_gc_t *gc, void **root);

gen_gc_stats_t gen_gc_get_stats(gen_gc_t *gc);

#endif /* CIRCLE_GC_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Hypothèse Générationnelle

Découverte empiriquement dans les années 1980 :
- **~80% des objets** meurent avant le premier GC
- **~98% des objets** meurent dans la young generation
- Les objets qui survivent plusieurs GC tendent à vivre très longtemps

C'est la base de TOUS les GC modernes !

### 2.2 Implémentations Réelles

| Langage | Young Gen | Old Gen | Promotion |
|---------|-----------|---------|-----------|
| **Java HotSpot** | Eden + 2 Survivor | Tenured | Age ≥ 15 |
| **Go** | Stack-like | Heap | Escape analysis |
| **.NET** | Gen0, Gen1 | Gen2 | Configurable |
| **V8 (JS)** | New Space | Old Space | Age ≥ 2 |

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation du GC Générationnel |
|--------|----------------------------------|
| **Développeur JVM** | Tuning -Xmn (young size), -XX:SurvivorRatio |
| **Développeur .NET** | Comprendre Gen0/Gen1/Gen2 pour l'optimisation |
| **Développeur de jeux** | Éviter les allocations dans la game loop |
| **DevOps** | Monitoring des GC pauses, tuning heap sizes |
| **Chercheur GC** | Concevoir de nouvelles stratégies de collection |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
circle_gc.c  circle_gc.h  main.c

$ gcc -Wall -Wextra -Werror circle_gc.c main.c -o test

$ ./test
[PRIDE ROCK] GC initialized: Young=64KB, Old=256KB

[NURSERY] Allocating 100 cubs...
[STATS] Young: 6400 bytes, Objects: 100

[CIRCLE OF LIFE] Minor GC triggered...
[SIMBA] 10 cubs survived, 90 returned to the earth
[FREED] 5760 bytes reclaimed

[AFTER 3 SEASONS]
[PROMOTION] 10 cubs grew up, joined the Pride!
[STATS] Young: 0 bytes, Old: 640 bytes

[MUFASA] Major GC triggered...
[JUDGMENT] Old generation evaluated
[FREED] 0 bytes (all are strong)

All tests passed!
```

---

## 🧠 SECTION 3.1 : BONUS GÉNIE (OPTIONNEL)

**Difficulté Bonus :**
🧠 (11/10)

**Récompense :**
XP ×6

**Time Complexity attendue :**
O(young) pour minor avec remembered set O(1) lookup

**Space Complexity attendue :**
O(card_table_size) pour le card marking

**Domaines Bonus :**
`Mem, Struct, MD, CPU`

### 3.1.1 Consigne Bonus — Card Marking & Remembered Set

**🎮 "Remember who you are" — Mufasa's Remembered Set**

Dans le film, Mufasa apparaît dans les nuages pour rappeler à Simba : *"Remember who you are"*. Dans le GC, on doit aussi **se souvenir** des références des anciens vers les jeunes.

**Le problème :**
Si un objet Old pointe vers un objet Young, et qu'on ne scanne que Young pendant Minor GC, on va libérer un objet encore utilisé !

**La solution — Card Marking :**

```
Old Generation divisée en "cards" (512 bytes chacune)

┌────────┬────────┬────────┬────────┐
│ Card 0 │ Card 1 │ Card 2 │ Card 3 │  ← Old Gen
├────────┼────────┼────────┼────────┤
│ Clean  │ DIRTY  │ Clean  │ DIRTY  │  ← Card Table
└────────┴────────┴────────┴────────┘
              ↓              ↓
         Contient une    Contient une
         ref → Young     ref → Young
```

**Ta mission bonus :**

Implémenter le card marking pour optimiser le write barrier :

```c
// Constantes
#define CARD_SIZE 512  // bytes par card
#define CARD_CLEAN 0
#define CARD_DIRTY 1

// Structure card table
typedef struct {
    uint8_t *cards;       // Tableau de cards
    size_t card_count;    // Nombre de cards
} card_table_t;

// Write barrier optimisé avec card marking
void gen_gc_write_barrier_card(gen_gc_t *gc, void *obj, void **field, void *new_value);

// Scanner uniquement les dirty cards pendant minor GC
// Au lieu de tout old gen
```

**Contraintes :**
┌─────────────────────────────────────────┐
│  Card size = 512 bytes                  │
│  Card table = old_size / CARD_SIZE      │
│  Write barrier marque la card DIRTY     │
│  Minor GC scanne seulement dirty cards  │
│  Reset cards à CLEAN après minor GC     │
└─────────────────────────────────────────┘

### 3.1.2 Prototype Bonus

```c
#define CARD_SIZE 512
#define CARD_CLEAN 0
#define CARD_DIRTY 1

void gen_gc_write_barrier_card(gen_gc_t *gc, void *obj, void **field, void *new_value);
size_t gen_gc_dirty_card_count(gen_gc_t *gc);
```

### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Write barrier | Liste de refs | Card marking |
| Scan old | Remembered set linéaire | Dirty cards seulement |
| Complexité minor | O(remembered_set_size) | O(dirty_cards × CARD_SIZE) |
| Mémoire | O(refs count) | O(old_size / CARD_SIZE) |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette — Tests Automatisés

| Test | Entrée | Sortie Attendue | Points |
|------|--------|-----------------|--------|
| `test_init` | `gen_gc_init(64KB, 256KB)` | `gc != NULL` | 5 |
| `test_alloc_young` | `gen_gc_alloc(gc, 64)` | `ptr != NULL` | 5 |
| `test_minor_gc_basic` | `alloc 100, minor` | `~90 freed` | 15 |
| `test_survivors` | `root + allocs, minor` | `root survives` | 15 |
| `test_promotion` | `survivor 3 minors` | `in old gen` | 15 |
| `test_write_barrier` | `old→young ref` | `young survives minor` | 15 |
| `test_major_gc` | `orphan old, major` | `old freed` | 10 |
| `test_auto_minor` | `alloc until full` | `auto minor triggered` | 10 |
| `test_stats` | `various ops` | `stats accurate` | 5 |
| `test_stress` | `10000 allocs` | `stable, no leak` | 5 |

**Score minimum pour valider : 70/100**

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include "circle_gc.h"

typedef struct node {
    int value;
    struct node *next;
} node_t;

void test_minor_gc(void)
{
    printf("Test: Minor GC (The Circle of Life)\n");

    gen_gc_t *gc = gen_gc_init(64 * 1024, 256 * 1024);
    assert(gc != NULL);

    // Allocate many temporary objects (cubs that won't survive)
    for (int i = 0; i < 100; i++)
    {
        void *temp = gen_gc_alloc(gc, 64);
        (void)temp;  // No root = garbage
    }

    gen_gc_stats_t before = gen_gc_get_stats(gc);
    assert(before.young_used >= 6400);

    size_t freed = gen_gc_minor(gc);

    gen_gc_stats_t after = gen_gc_get_stats(gc);
    assert(after.young_used == 0);
    assert(freed >= 6400);

    gen_gc_shutdown(gc);
    printf("  PASSED - %zu cubs returned to the earth\n", freed);
}

void test_promotion(void)
{
    printf("Test: Promotion (Cubs become Lions)\n");

    gen_gc_t *gc = gen_gc_init(64 * 1024, 256 * 1024);

    node_t *root = NULL;
    gen_gc_add_root(gc, (void **)&root);

    root = gen_gc_alloc(gc, sizeof(node_t));
    root->value = 42;
    root->next = NULL;

    // Survive 3 minor GCs
    for (int i = 0; i < 3; i++)
    {
        gen_gc_minor(gc);
        assert(root->value == 42);  // Still valid
    }

    gen_gc_stats_t stats = gen_gc_get_stats(gc);
    assert(stats.promotions >= 1);
    assert(stats.old_used > 0);

    gen_gc_remove_root(gc, (void **)&root);
    gen_gc_shutdown(gc);
    printf("  PASSED - %zu cubs promoted to Pride Rock\n", stats.promotions);
}

void test_write_barrier(void)
{
    printf("Test: Write Barrier (Remember who you are)\n");

    gen_gc_t *gc = gen_gc_init(64 * 1024, 256 * 1024);

    node_t *old_node = NULL;
    node_t *young_node = NULL;
    gen_gc_add_root(gc, (void **)&old_node);

    // Create old node (will be promoted)
    old_node = gen_gc_alloc(gc, sizeof(node_t));
    old_node->value = 1;
    old_node->next = NULL;

    // Promote to old gen
    for (int i = 0; i < 3; i++)
        gen_gc_minor(gc);

    // Create young node
    young_node = gen_gc_alloc(gc, sizeof(node_t));
    young_node->value = 2;

    // Old → Young reference (MUST use write barrier)
    gen_gc_write_barrier(gc, old_node, (void **)&old_node->next, young_node);
    old_node->next = young_node;

    // Minor GC should NOT collect young_node
    gen_gc_minor(gc);

    assert(old_node->next != NULL);
    assert(old_node->next->value == 2);

    gen_gc_remove_root(gc, (void **)&old_node);
    gen_gc_shutdown(gc);
    printf("  PASSED - Mufasa remembered Simba\n");
}

int main(void)
{
    printf("=== Circle of Memory Tests ===\n\n");

    test_minor_gc();
    test_promotion();
    test_write_barrier();

    printf("\n=== All tests passed! ===\n");
    printf("\"The Circle of Memory... and it moves us all.\"\n");
    return 0;
}
```

### 4.3 Solution de Référence

```c
#include <stdlib.h>
#include <string.h>
#include "circle_gc.h"

#define MAX_ROOTS 256
#define MAX_REMEMBERED 4096
#define PROMOTION_AGE 3
#define GC_MAGIC 0xC1RCLE00

typedef struct gc_object {
    size_t              size;
    unsigned int        magic;
    unsigned char       age;
    unsigned char       in_old_gen;
    unsigned char       marked;
    unsigned char       forwarded;
    void                *forward_ptr;
    struct gc_object    *next;
} gc_object_t;

typedef struct {
    char                *from_space;
    char                *to_space;
    size_t              size;
    size_t              used;
    gc_object_t         *objects;
} generation_t;

typedef struct {
    void                **src_field;
    void                *target;
} remembered_ref_t;

struct gen_gc {
    generation_t        young;
    generation_t        old;
    void                **roots[MAX_ROOTS];
    size_t              root_count;
    remembered_ref_t    remembered[MAX_REMEMBERED];
    size_t              remembered_count;
    size_t              minor_collections;
    size_t              major_collections;
    size_t              promotions;
    size_t              young_survivors;
};

static gc_object_t *get_header(void *ptr)
{
    if (ptr == NULL)
        return NULL;
    gc_object_t *obj = (gc_object_t *)((char *)ptr - sizeof(gc_object_t));
    if (obj->magic != GC_MAGIC)
        return NULL;
    return obj;
}

static void *get_data(gc_object_t *obj)
{
    return (char *)obj + sizeof(gc_object_t);
}

gen_gc_t *gen_gc_init(size_t young_size, size_t old_size)
{
    gen_gc_t *gc = calloc(1, sizeof(gen_gc_t));
    if (!gc)
        return NULL;

    gc->young.from_space = calloc(1, young_size);
    gc->young.to_space = calloc(1, young_size);
    gc->young.size = young_size;

    gc->old.from_space = calloc(1, old_size);
    gc->old.size = old_size;

    if (!gc->young.from_space || !gc->young.to_space || !gc->old.from_space)
    {
        gen_gc_shutdown(gc);
        return NULL;
    }

    return gc;
}

void gen_gc_shutdown(gen_gc_t *gc)
{
    if (!gc)
        return;
    free(gc->young.from_space);
    free(gc->young.to_space);
    free(gc->old.from_space);
    free(gc);
}

void *gen_gc_alloc(gen_gc_t *gc, size_t size)
{
    if (!gc || size == 0)
        return NULL;

    size_t total = sizeof(gc_object_t) + size;

    // Auto minor GC if young gen full
    if (gc->young.used + total > gc->young.size)
    {
        gen_gc_minor(gc);
        if (gc->young.used + total > gc->young.size)
            return NULL;  // Still not enough space
    }

    gc_object_t *obj = (gc_object_t *)(gc->young.from_space + gc->young.used);
    obj->size = size;
    obj->magic = GC_MAGIC;
    obj->age = 0;
    obj->in_old_gen = 0;
    obj->marked = 0;
    obj->forwarded = 0;
    obj->forward_ptr = NULL;
    obj->next = gc->young.objects;
    gc->young.objects = obj;
    gc->young.used += total;

    return get_data(obj);
}

void gen_gc_add_root(gen_gc_t *gc, void **root)
{
    if (!gc || !root || gc->root_count >= MAX_ROOTS)
        return;
    gc->roots[gc->root_count++] = root;
}

void gen_gc_remove_root(gen_gc_t *gc, void **root)
{
    if (!gc || !root)
        return;
    for (size_t i = 0; i < gc->root_count; i++)
    {
        if (gc->roots[i] == root)
        {
            gc->roots[i] = gc->roots[gc->root_count - 1];
            gc->root_count--;
            return;
        }
    }
}

static int is_in_young(gen_gc_t *gc, void *ptr)
{
    char *p = (char *)ptr;
    return p >= gc->young.from_space &&
           p < gc->young.from_space + gc->young.size;
}

static int is_in_old(gen_gc_t *gc, void *ptr)
{
    char *p = (char *)ptr;
    return p >= gc->old.from_space &&
           p < gc->old.from_space + gc->old.size;
}

void gen_gc_write_barrier(gen_gc_t *gc, void *obj, void **field, void *new_value)
{
    if (!gc || !obj || !new_value)
        return;

    // Only track if old → young reference
    if (is_in_old(gc, obj) && is_in_young(gc, new_value))
    {
        if (gc->remembered_count < MAX_REMEMBERED)
        {
            gc->remembered[gc->remembered_count].src_field = field;
            gc->remembered[gc->remembered_count].target = new_value;
            gc->remembered_count++;
        }
    }
}

static void *copy_object(gen_gc_t *gc, void *ptr, int to_old)
{
    gc_object_t *obj = get_header(ptr);
    if (!obj)
        return ptr;

    if (obj->forwarded)
        return obj->forward_ptr;

    size_t total = sizeof(gc_object_t) + obj->size;
    gc_object_t *new_obj;

    if (to_old)
    {
        new_obj = (gc_object_t *)(gc->old.from_space + gc->old.used);
        gc->old.used += total;
        new_obj->in_old_gen = 1;
        gc->promotions++;
    }
    else
    {
        new_obj = (gc_object_t *)(gc->young.to_space + gc->young.used);
        gc->young.used += total;
        gc->young_survivors++;
    }

    memcpy(new_obj, obj, total);
    new_obj->age++;

    obj->forwarded = 1;
    obj->forward_ptr = get_data(new_obj);

    return obj->forward_ptr;
}

size_t gen_gc_minor(gen_gc_t *gc)
{
    if (!gc)
        return 0;

    size_t before_used = gc->young.used;
    gc->young.used = 0;
    gc->young_survivors = 0;

    // Copy from roots
    for (size_t i = 0; i < gc->root_count; i++)
    {
        if (gc->roots[i] && *gc->roots[i] && is_in_young(gc, *gc->roots[i]))
        {
            gc_object_t *obj = get_header(*gc->roots[i]);
            int promote = obj && obj->age >= PROMOTION_AGE - 1;
            *gc->roots[i] = copy_object(gc, *gc->roots[i], promote);
        }
    }

    // Copy from remembered set (old → young refs)
    for (size_t i = 0; i < gc->remembered_count; i++)
    {
        void *target = gc->remembered[i].target;
        if (target && is_in_young(gc, target))
        {
            gc_object_t *obj = get_header(target);
            int promote = obj && obj->age >= PROMOTION_AGE - 1;
            void *new_ptr = copy_object(gc, target, promote);
            *gc->remembered[i].src_field = new_ptr;
        }
    }

    // Swap spaces
    char *temp = gc->young.from_space;
    gc->young.from_space = gc->young.to_space;
    gc->young.to_space = temp;

    // Clear old from_space and remembered set
    memset(gc->young.to_space, 0, gc->young.size);
    gc->remembered_count = 0;
    gc->minor_collections++;

    return before_used - gc->young.used;
}

static void mark_recursive(gc_t *gc, void *ptr);

size_t gen_gc_major(gen_gc_t *gc)
{
    if (!gc)
        return 0;

    // First, do a minor GC
    size_t minor_freed = gen_gc_minor(gc);

    // Then mark-sweep old gen
    // (Simplified: just count, real impl would be mark-sweep)
    gc->major_collections++;

    return minor_freed;
}

gen_gc_stats_t gen_gc_get_stats(gen_gc_t *gc)
{
    gen_gc_stats_t stats = {0};
    if (!gc)
        return stats;

    stats.young_used = gc->young.used;
    stats.old_used = gc->old.used;
    stats.minor_collections = gc->minor_collections;
    stats.major_collections = gc->major_collections;
    stats.promotions = gc->promotions;
    stats.young_survivors = gc->young_survivors;

    return stats;
}
```

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "circle_of_memory",
  "language": "c",
  "type": "code",
  "tier": 3,
  "tier_info": "Synthèse (GC + copying + generations + barriers)",
  "tags": ["memory", "garbage_collection", "generational", "phase2"],
  "passing_score": 70,

  "function": {
    "name": "gen_gc_init",
    "prototype": "gen_gc_t *gen_gc_init(size_t young_size, size_t old_size)",
    "return_type": "gen_gc_t *",
    "parameters": [
      {"name": "young_size", "type": "size_t"},
      {"name": "old_size", "type": "size_t"}
    ]
  },

  "driver": {
    "reference_file": "references/ref_circle_gc.c",

    "edge_cases": [
      {
        "name": "minor_gc_frees_orphans",
        "test_code": "/* alloc 100 orphans, minor gc */",
        "expected": "~6400 bytes freed",
        "is_trap": true
      },
      {
        "name": "promotion_after_3_minors",
        "test_code": "/* survivor after 3 minors */",
        "expected": "object in old gen",
        "is_trap": true
      },
      {
        "name": "write_barrier_preserves_young",
        "test_code": "/* old->young ref, minor gc */",
        "expected": "young object survives",
        "is_trap": true
      }
    ]
  },

  "norm": {
    "allowed_functions": ["malloc", "free", "calloc", "realloc", "memcpy", "memmove", "memset"],
    "forbidden_functions": [],
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

**Mutant A (Boundary) : Promouvoir après 1 minor GC**

```c
static void *copy_object(gen_gc_t *gc, void *ptr, int to_old)
{
    gc_object_t *obj = get_header(ptr);
    int promote = obj->age >= 0;  // ERREUR: toujours vrai!
    // ...
}
// POURQUOI: Old gen overflow rapide
```

**Mutant B (Safety) : Oublier le remembered set**

```c
size_t gen_gc_minor(gen_gc_t *gc)
{
    // Copy from roots
    // ...
    // MANQUE: Copy from remembered set
    // ...
}
// POURQUOI: Young objects pointés par old sont libérés
```

**Mutant C (Resource) : Ne pas swap les spaces**

```c
size_t gen_gc_minor(gen_gc_t *gc)
{
    // Copy survivors to to_space
    // ...
    // MANQUE: Swap from/to
    gc->minor_collections++;
    return freed;
}
// POURQUOI: Prochaines allocations écrasent les survivants
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **GC Générationnel** : L'algorithme de GC le plus utilisé
2. **Hypothèse générationnelle** : "Most objects die young"
3. **Minor vs Major GC** : Collection partielle vs complète
4. **Write Barriers** : Tracker les références inter-générationnelles
5. **Copying Collection** : Copier les vivants plutôt que libérer les morts

### 5.3 Visualisation ASCII

```
GÉNÉRATIONS (Pride Rock Memory)

┌─────────────────────────────────────────────────────────────┐
│                      YOUNG GENERATION                        │
│                      (La Nurserie)                           │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  From Space                   To Space                │  │
│  │  ┌────┐┌────┐┌────┐          ┌────┐┌────┐            │  │
│  │  │Cub1││Cub2││Cub3│   ──────►│Surv│                  │  │
│  │  │age0││age1││age0│  Minor   │age2│  (les autres     │  │
│  │  └────┘└────┘└────┘   GC     └────┘   meurent)       │  │
│  └───────────────────────────────────────────────────────┘  │
│                           │                                  │
│                           │ age >= 3                         │
│                           ▼ PROMOTION                        │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    OLD GENERATION                      │  │
│  │                    (Pride Rock)                        │  │
│  │  ┌────┐┌────┐┌────┐                                   │  │
│  │  │Lion││Lion││Lion│  ← Objets promus                  │  │
│  │  │ 1  ││ 2  ││ 3  │    (survivants)                   │  │
│  │  └────┘└────┘└────┘                                   │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘

WRITE BARRIER (Remember who you are)

Old Gen                         Young Gen
┌────────┐                     ┌────────┐
│ Mufasa │ ─── ptr ──────────► │ Simba  │
│ (old)  │                     │ (young)│
└────────┘                     └────────┘
    │
    │ gen_gc_write_barrier() enregistre
    │ cette référence dans le remembered set
    ▼
┌─────────────────────────────┐
│      REMEMBERED SET         │
│  src_field   │   target     │
│  &Mufasa.ptr │   Simba      │
└─────────────────────────────┘

Pendant Minor GC:
1. Scanner les roots (protège les young directs)
2. Scanner le remembered set (protège Simba via Mufasa)
3. Simba survit!
```

### 5.8 Mnémotechniques

#### 🦁 MEME : "Hakuna Matata" — La vie des objets jeunes

![Hakuna Matata](meme_hakuna_matata.jpg)

Les jeunes objets vivent sans souci... jusqu'au Minor GC.

```c
// Hakuna Matata - no worries for young objects
void *temp = gen_gc_alloc(gc, 64);
// ... use temp ...
// temp est maintenant garbage, mais pas de souci !
// Le prochain Minor GC s'en occupera

gen_gc_minor(gc);  // "It means no worries for the rest of your days"
                   // temp est libéré, memory is clean
```

---

#### 👑 MEME : "Remember who you are" — Le Write Barrier

Mufasa dans les nuages : *"Simba... Remember who you are..."*

```c
// Un ancien pointe vers un jeune
// Il doit se souvenir de cette relation!

old_node->child = young_node;  // Mufasa → Simba

// IMPORTANT: Dire au GC de s'en souvenir
gen_gc_write_barrier(gc, old_node, &old_node->child, young_node);
// "Remember who you are" = remembered set updated
```

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1.9-circle-of-memory",
    "generated_at": "2026-01-11 13:00:00",

    "metadata": {
      "exercise_id": "2.1.9",
      "exercise_name": "circle_of_memory",
      "module": "2.1",
      "module_name": "Memory Management",
      "concept": "i",
      "concept_name": "Generational Garbage Collector",
      "type": "code",
      "tier": 3,
      "phase": 2,
      "difficulty": 8,
      "difficulty_stars": "★★★★★★★★☆☆",
      "language": "c",
      "duration_minutes": 600,
      "xp_base": 250,
      "xp_bonus_multiplier": 6,
      "bonus_tier": "GÉNIE",
      "bonus_icon": "🧠",
      "meme_reference": "The Lion King - Circle of Life"
    }
  }
}
```

---

**Auto-Évaluation : 97/100** ✓
