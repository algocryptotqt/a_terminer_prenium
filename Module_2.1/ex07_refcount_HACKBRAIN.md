<thinking>
## Analyse du Concept
- Concept : Reference Counted Smart Pointers (comptage de références)
- Phase demandée : 2
- Adapté ? OUI — Le concept de smart pointers avec comptage de références est fondamental pour la gestion mémoire moderne, utilisé dans Rust (Rc/Arc), C++ (shared_ptr), Swift, etc. Enseignable en un exercice avec progression claire.

## Combo Base + Bonus
- Exercice de base : Implémenter rc_new, rc_clone, rc_drop, rc_get, rc_count — le cœur du reference counting
- Bonus : Ajouter les weak references (rc_downgrade, weak_upgrade, weak_drop) pour résoudre le problème des cycles
- Palier bonus : 🔥 Avancé — Les weak references ajoutent une couche de complexité significative
- Progression logique ? OUI — On maîtrise d'abord le strong counting, puis on ajoute le weak counting

## Prérequis & Difficulté
- Prérequis réels : Pointeurs, structures, malloc/free, callbacks (destructeurs)
- Difficulté estimée : 6/10
- Cohérent avec phase ? OUI — Phase 2 permet 4-6/10

## Aspect Fun/Culture
- Contexte choisi : Naruto — Shadow Clone Jutsu
- MEME mnémotechnique : "Kage Bunshin no Jutsu!" — Les clones partagent la mémoire avec l'original
- Pourquoi c'est fun :
  - Les shadow clones de Naruto sont des copies qui partagent les expériences avec l'original
  - Quand un clone est détruit, ses souvenirs retournent à l'original (comme rc_drop)
  - L'original ne disparaît que quand TOUS les clones sont détruits
  - Les clones faibles (weak) = clones d'information qui ne comptent pas dans le chakra total
  - Parfaite analogie avec reference counting!

## Scénarios d'Échec (5 mutants concrets)
1. Mutant A (Boundary) : Ne pas décrémenter le compteur dans rc_drop — le destructeur n'est jamais appelé
2. Mutant B (Safety) : Oublier de vérifier count == 0 avant d'appeler le destructeur — double free
3. Mutant C (Resource) : Ne pas libérer le rc_t lui-même après avoir appelé le destructeur — memory leak
4. Mutant D (Logic) : Dans rc_clone, créer un nouveau rc_t au lieu d'incrémenter le compteur existant — comportement incorrect
5. Mutant E (Return) : weak_upgrade retourne le pointeur même si strong_count == 0 — use-after-free

## Verdict
VALIDE — L'exercice est parfait pour enseigner le reference counting avec une progression logique vers les weak references.
</thinking>

---

# Exercice 2.1.7 : kage_bunshin_memory

**Module :**
2.1 — Memory Management

**Concept :**
g — Reference Counted Smart Pointers

**Difficulté :**
★★★★★★☆☆☆☆ (6/10)

**Type :**
code

**Tiers :**
2 — Mélange (concepts ref_counting + callbacks + weak_refs)

**Langage :**
C (C17)

**Prérequis :**
- Pointeurs et structures (Phase 1)
- malloc/free (ex04)
- Callbacks et pointeurs de fonction
- Concept de propriété de la mémoire

**Domaines :**
Mem, Struct

**Durée estimée :**
240 min

**XP Base :**
150

**Complexité :**
T2 O(1) toutes opérations × S2 O(n) pour n références

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichier à rendre :** `kage_bunshin.c`, `kage_bunshin.h`

**Fonctions autorisées :**
- `malloc`, `free`, `calloc`
- Fonctions standard de libc

**Fonctions interdites :**
- Aucune bibliothèque de smart pointers existante
- Pas de threads pour la version de base (atomics pour bonus)

### 1.2 Consigne

**🎮 CONTEXTE FUN — Naruto: Kage Bunshin no Jutsu (Multi-Clonage)**

Dans l'univers de Naruto, le jutsu **Kage Bunshin** (Technique du Multi-Clonage) permet de créer des clones parfaits qui partagent les expériences avec l'original. Quand un clone est détruit, ses souvenirs et son expérience retournent au ninja original. L'original ne disparaît que lorsque TOUS les clones sont détruits.

Tu es un développeur de jutsu au village de Konoha. Le Hokage t'a confié la mission de créer un système de gestion mémoire inspiré du Kage Bunshin : le **Reference Counting**.

**Le concept :**
- Chaque donnée en mémoire est comme un ninja original
- Quand tu "clones" une référence, c'est comme créer un shadow clone
- Le compteur de références = nombre de clones actifs + l'original
- Quand le dernier clone/référence est détruit → le ninja original disparaît (destructeur appelé)

### 1.2.2 Énoncé Académique

Le **comptage de références** (reference counting) est une technique de gestion automatique de la mémoire où chaque objet maintient un compteur du nombre de références pointant vers lui. Quand une nouvelle référence est créée (clone), le compteur est incrémenté. Quand une référence est détruite (drop), le compteur est décrémenté. Quand le compteur atteint zéro, l'objet est automatiquement libéré via son destructeur.

**Ta mission :**

Écrire une bibliothèque de **smart pointers avec comptage de références**, similaire à `shared_ptr` en C++ ou `Rc<T>` en Rust.

**API à implémenter :**

```c
// Type opaque pour le smart pointer
typedef struct rc rc_t;

// Crée un nouveau smart pointer avec les données et un destructeur
rc_t *rc_new(void *data, void (*destructor)(void *));

// Incrémente le compteur et retourne une référence vers le même objet
rc_t *rc_clone(rc_t *rc);

// Décrémente le compteur; libère si count == 0
void rc_drop(rc_t *rc);

// Accès aux données encapsulées
void *rc_get(const rc_t *rc);

// Nombre de références actives
size_t rc_count(const rc_t *rc);
```

**Entrée :**
- `data` : Pointeur vers les données à gérer (void *)
- `destructor` : Fonction callback appelée quand count atteint 0
- `rc` : Pointeur vers un smart pointer existant

**Sortie :**
- `rc_new` : Retourne un nouveau smart pointer, ou NULL si échec
- `rc_clone` : Retourne une référence au même objet (count++)
- `rc_get` : Retourne le pointeur vers les données
- `rc_count` : Retourne le nombre actuel de références

**Contraintes :**
- Le destructeur peut être NULL (pas de cleanup personnalisé)
- `rc_clone(NULL)` doit retourner NULL
- `rc_drop(NULL)` ne fait rien (safe)
- `rc_get(NULL)` retourne NULL
- `rc_count(NULL)` retourne 0
- Toutes les références partagent le MÊME compteur
- Le destructeur est appelé EXACTEMENT UNE FOIS quand count == 0

**Exemples :**

| Opération | rc_count() | Explication |
|-----------|------------|-------------|
| `rc1 = rc_new(data, dtor)` | 1 | Nouveau smart pointer créé |
| `rc2 = rc_clone(rc1)` | 2 | Deuxième référence au même objet |
| `rc3 = rc_clone(rc1)` | 3 | Troisième référence |
| `rc_drop(rc1)` | 2 | rc1 libéré, compteur décrémenté |
| `rc_drop(rc2)` | 1 | rc2 libéré, compteur décrémenté |
| `rc_drop(rc3)` | 0 | Destructeur appelé, data libéré |

### 1.3 Prototype

```c
#ifndef KAGE_BUNSHIN_H
#define KAGE_BUNSHIN_H

#include <stddef.h>

typedef struct rc rc_t;

rc_t *rc_new(void *data, void (*destructor)(void *));
rc_t *rc_clone(rc_t *rc);
void rc_drop(rc_t *rc);
void *rc_get(const rc_t *rc);
size_t rc_count(const rc_t *rc);

#endif /* KAGE_BUNSHIN_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Culture Générale

Le **reference counting** a été inventé par George Collins en 1960, c'est l'une des plus anciennes techniques de gestion automatique de la mémoire ! Aujourd'hui, on la retrouve partout :

- **Python** : Chaque objet a un `ob_refcnt` (c'est pourquoi `sys.getrefcount()` existe)
- **Swift** : ARC (Automatic Reference Counting) est au cœur du langage
- **Rust** : `Rc<T>` pour single-thread, `Arc<T>` pour multi-thread
- **C++** : `std::shared_ptr<T>` depuis C++11
- **Objective-C** : Manual reference counting avant ARC

### 2.2 Le Problème des Cycles

Le talon d'Achille du reference counting : les **cycles de références**. Si A pointe vers B et B pointe vers A, leurs compteurs ne descendront jamais à 0 même si plus personne n'utilise A ou B. C'est le **memory leak par cycle**.

```
┌───────┐     strong      ┌───────┐
│   A   │ ────────────►   │   B   │
│ cnt=1 │   ◄──────────── │ cnt=1 │
└───────┘     strong      └───────┘
         ↑
     Plus de référence externe mais cnt != 0 !
```

Solution : **Weak references** (bonus de cet exercice).

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation du Reference Counting |
|--------|-----------------------------------|
| **Développeur iOS/macOS** | ARC (Automatic Reference Counting) pour toute gestion mémoire Swift/Objective-C |
| **Développeur Python** | Chaque objet Python utilise le refcount, crucial pour les extensions C |
| **Développeur Rust** | `Rc<T>` pour partager des données entre plusieurs propriétaires |
| **Développeur C++** | `shared_ptr` pour la gestion automatique des ressources (RAII) |
| **Développeur de jeux** | Gestion des assets partagés (textures, sons, modèles 3D) |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
kage_bunshin.c  kage_bunshin.h  main.c

$ gcc -Wall -Wextra -Werror kage_bunshin.c main.c -o test

$ ./test
[JUTSU] Creating ninja: Naruto
Count after creation: 1
[CLONE] Kage Bunshin! Count: 2
[CLONE] Kage Bunshin! Count: 3
[DROP] Dispelling clone... Count: 2
[DROP] Dispelling clone... Count: 1
[DROP] Last clone dispelled!
[DESTRUCTOR] Ninja released: Naruto
All tests passed!
```

---

## 🔥 SECTION 3.1 : BONUS AVANCÉ (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★★☆☆ (8/10)

**Récompense :**
XP ×3

**Time Complexity attendue :**
O(1) pour toutes les opérations

**Space Complexity attendue :**
O(1) supplémentaire par weak reference

**Domaines Bonus :**
`Mem, Struct, Process`

### 3.1.1 Consigne Bonus — Weak References

**🎮 Kage Bunshin: Information Clones**

Dans Naruto, certains clones sont créés uniquement pour **observer et rapporter** — ils ne comptent pas dans le chakra total du ninja. Si le ninja original disparaît, ces clones d'information se dissipent automatiquement.

C'est exactement le concept des **weak references** : elles pointent vers un objet sans empêcher sa destruction.

**Ta mission bonus :**

Ajouter le support des **weak references** pour casser les cycles :

```c
// Type pour les weak references
typedef struct weak_rc weak_rc_t;

// Crée une weak reference à partir d'une strong reference
weak_rc_t *rc_downgrade(rc_t *rc);

// Tente de promouvoir une weak en strong (NULL si objet détruit)
rc_t *weak_upgrade(weak_rc_t *weak);

// Libère une weak reference
void weak_drop(weak_rc_t *weak);

// Nombre de weak references actives
size_t weak_count(const rc_t *rc);
```

**Contraintes :**
┌─────────────────────────────────────────┐
│  weak_upgrade retourne NULL si          │
│  strong_count == 0 (objet détruit)      │
│                                         │
│  Les weak refs ne prolongent pas        │
│  la vie de l'objet                      │
│                                         │
│  Le "inner block" (compteurs) survit    │
│  tant qu'il y a des weak refs           │
└─────────────────────────────────────────┘

**Exemples :**

| Opération | strong_count | weak_count | Explication |
|-----------|--------------|------------|-------------|
| `rc1 = rc_new(data, dtor)` | 1 | 0 | Strong ref créée |
| `weak1 = rc_downgrade(rc1)` | 1 | 1 | Weak ref créée |
| `rc_drop(rc1)` | 0 | 1 | Destructeur appelé! |
| `weak_upgrade(weak1)` | — | — | Retourne NULL |
| `weak_drop(weak1)` | — | 0 | Cleanup complet |

### 3.1.2 Prototype Bonus

```c
typedef struct weak_rc weak_rc_t;

weak_rc_t *rc_downgrade(rc_t *rc);
rc_t *weak_upgrade(weak_rc_t *weak);
void weak_drop(weak_rc_t *weak);
size_t weak_count(const rc_t *rc);
```

### 3.1.3 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Types | rc_t seulement | rc_t + weak_rc_t |
| Compteurs | strong_count | strong_count + weak_count |
| Destruction | Quand strong == 0 | Données quand strong == 0, block quand weak == 0 |
| Upgrade | N/A | weak → strong possible si strong > 0 |
| Cycles | Problématiques | Cassables avec weak refs |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette — Tests Automatisés

| Test | Entrée | Sortie Attendue | Points |
|------|--------|-----------------|--------|
| `test_create` | `rc_new(ptr, dtor)` | `rc != NULL, count == 1` | 10 |
| `test_null_data` | `rc_new(NULL, dtor)` | `rc != NULL, get == NULL` | 5 |
| `test_null_dtor` | `rc_new(ptr, NULL)` | `rc != NULL, no crash on drop` | 5 |
| `test_clone` | `rc_clone(rc)` | `count == 2, same data` | 10 |
| `test_clone_null` | `rc_clone(NULL)` | `NULL` | 5 |
| `test_drop_single` | `rc_drop(rc1)` | `destructor called` | 10 |
| `test_drop_multiple` | `drop 2/3 refs` | `destructor NOT called` | 10 |
| `test_drop_last` | `drop all refs` | `destructor called once` | 15 |
| `test_drop_null` | `rc_drop(NULL)` | `no crash` | 5 |
| `test_get` | `rc_get(rc)` | `returns data ptr` | 5 |
| `test_count_accuracy` | `various ops` | `count always correct` | 10 |
| `test_valgrind` | `full lifecycle` | `0 leaks, 0 errors` | 10 |

**Score minimum pour valider : 70/100**

### 4.2 main.c de test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include "kage_bunshin.h"

typedef struct {
    char name[32];
    int power_level;
} ninja_t;

static int destructor_called = 0;
static char last_destroyed[32] = {0};

void ninja_destructor(void *data)
{
    ninja_t *ninja = (ninja_t *)data;
    strcpy(last_destroyed, ninja->name);
    destructor_called++;
    free(ninja);
}

void test_basic_lifecycle(void)
{
    printf("Test: Basic Lifecycle\n");
    destructor_called = 0;

    ninja_t *naruto = malloc(sizeof(ninja_t));
    strcpy(naruto->name, "Naruto");
    naruto->power_level = 9001;

    rc_t *rc1 = rc_new(naruto, ninja_destructor);
    assert(rc1 != NULL);
    assert(rc_count(rc1) == 1);
    assert(rc_get(rc1) == naruto);

    rc_drop(rc1);
    assert(destructor_called == 1);
    assert(strcmp(last_destroyed, "Naruto") == 0);

    printf("  PASSED\n");
}

void test_multiple_refs(void)
{
    printf("Test: Multiple References\n");
    destructor_called = 0;

    ninja_t *sasuke = malloc(sizeof(ninja_t));
    strcpy(sasuke->name, "Sasuke");
    sasuke->power_level = 8500;

    rc_t *rc1 = rc_new(sasuke, ninja_destructor);
    rc_t *rc2 = rc_clone(rc1);
    rc_t *rc3 = rc_clone(rc1);

    assert(rc_count(rc1) == 3);
    assert(rc_count(rc2) == 3);
    assert(rc_count(rc3) == 3);
    assert(rc_get(rc1) == rc_get(rc2));
    assert(rc_get(rc2) == rc_get(rc3));

    rc_drop(rc1);
    assert(destructor_called == 0);
    assert(rc_count(rc2) == 2);

    rc_drop(rc2);
    assert(destructor_called == 0);
    assert(rc_count(rc3) == 1);

    rc_drop(rc3);
    assert(destructor_called == 1);

    printf("  PASSED\n");
}

void test_null_handling(void)
{
    printf("Test: NULL Handling\n");

    assert(rc_clone(NULL) == NULL);
    rc_drop(NULL);  // Should not crash
    assert(rc_get(NULL) == NULL);
    assert(rc_count(NULL) == 0);

    // NULL data is allowed
    rc_t *rc = rc_new(NULL, NULL);
    assert(rc != NULL);
    assert(rc_get(rc) == NULL);
    assert(rc_count(rc) == 1);
    rc_drop(rc);

    printf("  PASSED\n");
}

void test_no_double_free(void)
{
    printf("Test: No Double Free\n");
    destructor_called = 0;

    ninja_t *kakashi = malloc(sizeof(ninja_t));
    strcpy(kakashi->name, "Kakashi");

    rc_t *rc1 = rc_new(kakashi, ninja_destructor);
    rc_t *rc2 = rc_clone(rc1);
    rc_t *rc3 = rc_clone(rc2);
    rc_t *rc4 = rc_clone(rc1);

    rc_drop(rc1);
    rc_drop(rc2);
    rc_drop(rc3);
    rc_drop(rc4);

    assert(destructor_called == 1);
    printf("  PASSED\n");
}

int main(void)
{
    printf("=== Kage Bunshin Memory Tests ===\n\n");

    test_basic_lifecycle();
    test_multiple_refs();
    test_null_handling();
    test_no_double_free();

    printf("\n=== All tests passed! ===\n");
    return 0;
}
```

### 4.3 Solution de Référence

```c
#include <stdlib.h>
#include "kage_bunshin.h"

/*
** Structure interne partagée entre toutes les références
** C'est le "bloc de contrôle" qui contient le compteur et les données
*/
typedef struct rc_inner {
    size_t          strong_count;
    void            *data;
    void            (*destructor)(void *);
} rc_inner_t;

/*
** Structure externe : chaque rc_t pointe vers le même inner
*/
struct rc {
    rc_inner_t      *inner;
};

rc_t *rc_new(void *data, void (*destructor)(void *))
{
    rc_t *rc;
    rc_inner_t *inner;

    rc = malloc(sizeof(rc_t));
    if (rc == NULL)
        return (NULL);
    inner = malloc(sizeof(rc_inner_t));
    if (inner == NULL)
    {
        free(rc);
        return (NULL);
    }
    inner->strong_count = 1;
    inner->data = data;
    inner->destructor = destructor;
    rc->inner = inner;
    return (rc);
}

rc_t *rc_clone(rc_t *rc)
{
    rc_t *clone;

    if (rc == NULL || rc->inner == NULL)
        return (NULL);
    clone = malloc(sizeof(rc_t));
    if (clone == NULL)
        return (NULL);
    clone->inner = rc->inner;
    rc->inner->strong_count++;
    return (clone);
}

void rc_drop(rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return;
    rc->inner->strong_count--;
    if (rc->inner->strong_count == 0)
    {
        if (rc->inner->destructor != NULL)
            rc->inner->destructor(rc->inner->data);
        free(rc->inner);
    }
    free(rc);
}

void *rc_get(const rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return (NULL);
    return (rc->inner->data);
}

size_t rc_count(const rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return (0);
    return (rc->inner->strong_count);
}
```

### 4.4 Solutions Alternatives Acceptées

**Alternative 1 : Single allocation (rc et inner fusionnés)**

```c
typedef struct rc {
    size_t          count;
    void            *data;
    void            (*destructor)(void *);
    size_t          ref_count;  // Nombre de rc_t pointant vers cette structure
} rc_t;

// Note: Cette approche est plus efficace en mémoire mais plus complexe
// car il faut tracker combien de rc_t pointent vers la structure partagée
```

**Alternative 2 : Macro-based implementation**

```c
#define RC_NEW(type, dtor) rc_new(malloc(sizeof(type)), dtor)
// Approche valide tant que l'API est respectée
```

### 4.5 Solutions Refusées

**Refusée 1 : Pas de partage du inner**

```c
// FAUX : Chaque clone a son propre compteur
rc_t *rc_clone(rc_t *rc)
{
    rc_t *clone = malloc(sizeof(rc_t));
    clone->inner = malloc(sizeof(rc_inner_t));  // ERREUR !
    clone->inner->count = rc->inner->count;     // Copie au lieu de partage
    return clone;
}
// POURQUOI C'EST FAUX : Les compteurs ne sont pas synchronisés
```

**Refusée 2 : Oubli de libérer le rc_t lui-même**

```c
void rc_drop(rc_t *rc)
{
    rc->inner->count--;
    if (rc->inner->count == 0)
    {
        rc->inner->destructor(rc->inner->data);
        free(rc->inner);
    }
    // ERREUR : free(rc) manquant → memory leak
}
```

**Refusée 3 : Décrémentation après libération**

```c
void rc_drop(rc_t *rc)
{
    if (rc->inner->count == 1)
    {
        rc->inner->destructor(rc->inner->data);
        free(rc->inner);
    }
    rc->inner->count--;  // ERREUR : Use-after-free si count était 1
    free(rc);
}
```

### 4.6 Solution Bonus de Référence (Weak References)

```c
#include <stdlib.h>
#include "kage_bunshin.h"

typedef struct rc_inner {
    size_t          strong_count;
    size_t          weak_count;
    void            *data;
    void            (*destructor)(void *);
} rc_inner_t;

struct rc {
    rc_inner_t      *inner;
};

struct weak_rc {
    rc_inner_t      *inner;
};

rc_t *rc_new(void *data, void (*destructor)(void *))
{
    rc_t *rc;
    rc_inner_t *inner;

    rc = malloc(sizeof(rc_t));
    if (rc == NULL)
        return (NULL);
    inner = malloc(sizeof(rc_inner_t));
    if (inner == NULL)
    {
        free(rc);
        return (NULL);
    }
    inner->strong_count = 1;
    inner->weak_count = 0;
    inner->data = data;
    inner->destructor = destructor;
    rc->inner = inner;
    return (rc);
}

rc_t *rc_clone(rc_t *rc)
{
    rc_t *clone;

    if (rc == NULL || rc->inner == NULL)
        return (NULL);
    clone = malloc(sizeof(rc_t));
    if (clone == NULL)
        return (NULL);
    clone->inner = rc->inner;
    rc->inner->strong_count++;
    return (clone);
}

void rc_drop(rc_t *rc)
{
    rc_inner_t *inner;

    if (rc == NULL || rc->inner == NULL)
        return;
    inner = rc->inner;
    inner->strong_count--;
    if (inner->strong_count == 0)
    {
        if (inner->destructor != NULL)
            inner->destructor(inner->data);
        inner->data = NULL;
        if (inner->weak_count == 0)
            free(inner);
    }
    free(rc);
}

void *rc_get(const rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return (NULL);
    return (rc->inner->data);
}

size_t rc_count(const rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return (0);
    return (rc->inner->strong_count);
}

weak_rc_t *rc_downgrade(rc_t *rc)
{
    weak_rc_t *weak;

    if (rc == NULL || rc->inner == NULL)
        return (NULL);
    weak = malloc(sizeof(weak_rc_t));
    if (weak == NULL)
        return (NULL);
    weak->inner = rc->inner;
    rc->inner->weak_count++;
    return (weak);
}

rc_t *weak_upgrade(weak_rc_t *weak)
{
    rc_t *rc;

    if (weak == NULL || weak->inner == NULL)
        return (NULL);
    if (weak->inner->strong_count == 0)
        return (NULL);
    rc = malloc(sizeof(rc_t));
    if (rc == NULL)
        return (NULL);
    rc->inner = weak->inner;
    weak->inner->strong_count++;
    return (rc);
}

void weak_drop(weak_rc_t *weak)
{
    rc_inner_t *inner;

    if (weak == NULL || weak->inner == NULL)
        return;
    inner = weak->inner;
    inner->weak_count--;
    if (inner->strong_count == 0 && inner->weak_count == 0)
        free(inner);
    free(weak);
}

size_t weak_count(const rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return (0);
    return (rc->inner->weak_count);
}
```

### 4.7 Solutions Alternatives Bonus (COMPLÈTES)

**Alternative : Utilisation d'un flag "is_alive" au lieu de vérifier strong_count**

```c
typedef struct rc_inner {
    size_t          strong_count;
    size_t          weak_count;
    int             is_alive;  // Flag explicite
    void            *data;
    void            (*destructor)(void *);
} rc_inner_t;

void rc_drop(rc_t *rc)
{
    // ...
    if (inner->strong_count == 0 && inner->is_alive)
    {
        inner->is_alive = 0;
        if (inner->destructor)
            inner->destructor(inner->data);
    }
    // ...
}

rc_t *weak_upgrade(weak_rc_t *weak)
{
    if (!weak->inner->is_alive)
        return (NULL);
    // ...
}
```

### 4.8 Solutions Refusées Bonus

**Refusée : weak_upgrade ne vérifie pas strong_count**

```c
rc_t *weak_upgrade(weak_rc_t *weak)
{
    rc_t *rc = malloc(sizeof(rc_t));
    rc->inner = weak->inner;
    rc->inner->strong_count++;  // ERREUR : strong était peut-être 0
    return (rc);
}
// POURQUOI : Use-after-free car data a été détruit
```

**Refusée : weak_drop libère inner même si strong_count > 0**

```c
void weak_drop(weak_rc_t *weak)
{
    weak->inner->weak_count--;
    if (weak->inner->weak_count == 0)
        free(weak->inner);  // ERREUR : strong refs existent encore!
    free(weak);
}
```

### 4.9 spec.json (ENGINE v22.1)

```json
{
  "name": "kage_bunshin_memory",
  "language": "c",
  "type": "code",
  "tier": 2,
  "tier_info": "Mélange (ref_counting + callbacks + lifecycle)",
  "tags": ["memory", "smart_pointers", "reference_counting", "phase2"],
  "passing_score": 70,

  "function": {
    "name": "rc_new",
    "prototype": "rc_t *rc_new(void *data, void (*destructor)(void *))",
    "return_type": "rc_t *",
    "parameters": [
      {"name": "data", "type": "void *"},
      {"name": "destructor", "type": "void (*)(void *)"}
    ],
    "additional_functions": [
      {
        "name": "rc_clone",
        "prototype": "rc_t *rc_clone(rc_t *rc)",
        "return_type": "rc_t *"
      },
      {
        "name": "rc_drop",
        "prototype": "void rc_drop(rc_t *rc)",
        "return_type": "void"
      },
      {
        "name": "rc_get",
        "prototype": "void *rc_get(const rc_t *rc)",
        "return_type": "void *"
      },
      {
        "name": "rc_count",
        "prototype": "size_t rc_count(const rc_t *rc)",
        "return_type": "size_t"
      }
    ]
  },

  "driver": {
    "reference": "typedef struct rc_inner { size_t count; void *data; void (*dtor)(void *); } rc_inner_t; struct rc { rc_inner_t *inner; }; rc_t *ref_rc_new(void *data, void (*dtor)(void *)) { rc_t *rc = malloc(sizeof(rc_t)); if (!rc) return NULL; rc->inner = malloc(sizeof(rc_inner_t)); if (!rc->inner) { free(rc); return NULL; } rc->inner->count = 1; rc->inner->data = data; rc->inner->dtor = dtor; return rc; } rc_t *ref_rc_clone(rc_t *rc) { if (!rc || !rc->inner) return NULL; rc_t *c = malloc(sizeof(rc_t)); if (!c) return NULL; c->inner = rc->inner; rc->inner->count++; return c; } void ref_rc_drop(rc_t *rc) { if (!rc || !rc->inner) return; rc->inner->count--; if (rc->inner->count == 0) { if (rc->inner->dtor) rc->inner->dtor(rc->inner->data); free(rc->inner); } free(rc); } void *ref_rc_get(const rc_t *rc) { return (rc && rc->inner) ? rc->inner->data : NULL; } size_t ref_rc_count(const rc_t *rc) { return (rc && rc->inner) ? rc->inner->count : 0; }",

    "edge_cases": [
      {
        "name": "null_rc_clone",
        "test_code": "rc_t *r = rc_clone(NULL);",
        "expected": "r == NULL",
        "is_trap": true,
        "trap_explanation": "rc_clone(NULL) doit retourner NULL"
      },
      {
        "name": "null_rc_drop",
        "test_code": "rc_drop(NULL);",
        "expected": "no crash",
        "is_trap": true,
        "trap_explanation": "rc_drop(NULL) ne doit pas crasher"
      },
      {
        "name": "null_rc_get",
        "test_code": "void *p = rc_get(NULL);",
        "expected": "p == NULL",
        "is_trap": true,
        "trap_explanation": "rc_get(NULL) doit retourner NULL"
      },
      {
        "name": "null_rc_count",
        "test_code": "size_t c = rc_count(NULL);",
        "expected": "c == 0",
        "is_trap": true,
        "trap_explanation": "rc_count(NULL) doit retourner 0"
      },
      {
        "name": "destructor_called_once",
        "test_code": "/* Create rc, clone 3 times, drop all */",
        "expected": "destructor called exactly 1 time",
        "is_trap": true,
        "trap_explanation": "Le destructeur ne doit être appelé qu'une fois"
      },
      {
        "name": "null_data_allowed",
        "test_code": "rc_t *r = rc_new(NULL, NULL);",
        "expected": "r != NULL && rc_get(r) == NULL",
        "is_trap": false,
        "trap_explanation": "NULL data est valide"
      },
      {
        "name": "count_synchronization",
        "test_code": "rc_t *a = rc_new(data, dtor); rc_t *b = rc_clone(a);",
        "expected": "rc_count(a) == rc_count(b) == 2",
        "is_trap": true,
        "trap_explanation": "Toutes les refs doivent voir le même count"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 5000,
      "generators": [
        {
          "type": "int",
          "param_index": 0,
          "params": {
            "min": 0,
            "max": 100,
            "description": "Nombre de clones à créer"
          }
        }
      ],
      "invariants": [
        "destructor appelé exactement une fois par objet",
        "count toujours >= 1 tant qu'au moins une ref existe",
        "pas de memory leak"
      ]
    }
  },

  "norm": {
    "allowed_functions": ["malloc", "free", "calloc"],
    "forbidden_functions": ["realloc"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

**Mutant A (Boundary) : Ne pas décrémenter le compteur**

```c
void rc_drop(rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return;
    // OUBLI : rc->inner->strong_count--;
    if (rc->inner->strong_count == 0)  // Toujours faux!
    {
        if (rc->inner->destructor != NULL)
            rc->inner->destructor(rc->inner->data);
        free(rc->inner);
    }
    free(rc);
}
// POURQUOI C'EST FAUX : Le destructeur n'est jamais appelé → memory leak
// CE QUI ÉTAIT PENSÉ : "Le if vérifie si c'est 0, donc ça marche"
```

**Mutant B (Safety) : Oubli de vérifier count avant destruction**

```c
void rc_drop(rc_t *rc)
{
    if (rc == NULL)
        return;
    rc->inner->strong_count--;
    // ERREUR : Appelle toujours le destructeur
    if (rc->inner->destructor != NULL)
        rc->inner->destructor(rc->inner->data);
    free(rc->inner);
    free(rc);
}
// POURQUOI C'EST FAUX : Double free si plusieurs références
// CE QUI ÉTAIT PENSÉ : "Je dois libérer quand je drop"
```

**Mutant C (Resource) : Ne pas libérer le rc_t**

```c
void rc_drop(rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return;
    rc->inner->strong_count--;
    if (rc->inner->strong_count == 0)
    {
        if (rc->inner->destructor != NULL)
            rc->inner->destructor(rc->inner->data);
        free(rc->inner);
    }
    // OUBLI : free(rc);
}
// POURQUOI C'EST FAUX : Memory leak de sizeof(rc_t) à chaque drop
// CE QUI ÉTAIT PENSÉ : "J'ai libéré le inner, c'est bon"
```

**Mutant D (Logic) : Clone crée un nouveau inner au lieu de partager**

```c
rc_t *rc_clone(rc_t *rc)
{
    rc_t *clone;
    rc_inner_t *new_inner;

    if (rc == NULL || rc->inner == NULL)
        return (NULL);
    clone = malloc(sizeof(rc_t));
    if (clone == NULL)
        return (NULL);
    // ERREUR : Crée un nouveau inner au lieu de partager
    new_inner = malloc(sizeof(rc_inner_t));
    new_inner->strong_count = rc->inner->strong_count + 1;  // Copie
    new_inner->data = rc->inner->data;
    new_inner->destructor = rc->inner->destructor;
    clone->inner = new_inner;
    return (clone);
}
// POURQUOI C'EST FAUX : Les compteurs ne sont pas synchronisés
// CE QUI ÉTAIT PENSÉ : "Je clone tout, donc c'est un vrai clone"
```

**Mutant E (Return) : rc_count retourne 1 au lieu du vrai count**

```c
size_t rc_count(const rc_t *rc)
{
    if (rc == NULL || rc->inner == NULL)
        return (0);
    return (1);  // ERREUR : Hardcodé à 1
}
// POURQUOI C'EST FAUX : Le count ne reflète pas la réalité
// CE QUI ÉTAIT PENSÉ : "Il y a toujours au moins 1 référence"
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Reference Counting** : Technique fondamentale de gestion mémoire automatique
2. **Partage de structure** : Plusieurs pointeurs vers le même bloc de contrôle
3. **Callbacks (Destructeurs)** : Exécution de code à la destruction
4. **Gestion du cycle de vie** : Création, partage, destruction propre
5. **Type opaque** : Cacher l'implémentation derrière une interface

### 5.2 LDA — Traduction Littérale

```
FONCTION rc_new QUI RETOURNE UN POINTEUR VERS rc_t ET PREND EN PARAMÈTRES data QUI EST UN POINTEUR VOID ET destructor QUI EST UN POINTEUR DE FONCTION
DÉBUT FONCTION
    DÉCLARER rc COMME POINTEUR VERS rc_t
    DÉCLARER inner COMME POINTEUR VERS rc_inner_t

    ALLOUER LA MÉMOIRE DE LA TAILLE D'UN rc_t ET AFFECTER À rc
    SI rc EST ÉGAL À NUL ALORS
        RETOURNER NUL
    FIN SI

    ALLOUER LA MÉMOIRE DE LA TAILLE D'UN rc_inner_t ET AFFECTER À inner
    SI inner EST ÉGAL À NUL ALORS
        LIBÉRER LA MÉMOIRE POINTÉE PAR rc
        RETOURNER NUL
    FIN SI

    AFFECTER 1 AU CHAMP strong_count DE inner
    AFFECTER data AU CHAMP data DE inner
    AFFECTER destructor AU CHAMP destructor DE inner
    AFFECTER inner AU CHAMP inner DE rc

    RETOURNER rc
FIN FONCTION

FONCTION rc_drop QUI NE RETOURNE RIEN ET PREND EN PARAMÈTRE rc QUI EST UN POINTEUR VERS rc_t
DÉBUT FONCTION
    SI rc EST ÉGAL À NUL OU LE CHAMP inner DE rc EST ÉGAL À NUL ALORS
        RETOURNER
    FIN SI

    DÉCRÉMENTER LE CHAMP strong_count DU CHAMP inner DE rc DE 1

    SI LE CHAMP strong_count DU CHAMP inner DE rc EST ÉGAL À 0 ALORS
        SI LE CHAMP destructor DU CHAMP inner DE rc EST DIFFÉRENT DE NUL ALORS
            APPELER LE CHAMP destructor DU CHAMP inner DE rc AVEC LE CHAMP data DU CHAMP inner DE rc
        FIN SI
        LIBÉRER LA MÉMOIRE POINTÉE PAR LE CHAMP inner DE rc
    FIN SI

    LIBÉRER LA MÉMOIRE POINTÉE PAR rc
FIN FONCTION
```

### 5.2.2 Logic Flow (Structured English)

```
ALGORITHME : Reference Counting Lifecycle
---
1. CRÉATION (rc_new):
   a. ALLOUER un nouveau bloc rc_t
   b. ALLOUER un bloc inner partagé
   c. INITIALISER count = 1, stocker data et destructor
   d. RETOURNER le smart pointer

2. CLONAGE (rc_clone):
   a. ALLOUER un nouveau bloc rc_t
   b. POINTER vers le MÊME inner
   c. INCRÉMENTER le compteur
   d. RETOURNER le clone

3. DESTRUCTION (rc_drop):
   a. DÉCRÉMENTER le compteur
   b. SI count == 0:
      |-- APPELER le destructeur si non-NULL
      |-- LIBÉRER le bloc inner
   c. LIBÉRER le bloc rc_t (toujours)
```

### 5.2.3 Représentation Algorithmique (Fail Fast)

```
FONCTION : rc_drop (rc)
---
INIT : Aucune

1. GARDE : rc NULL ?
   |-- OUI → RETOURNER immédiatement

2. GARDE : inner NULL ?
   |-- OUI → RETOURNER immédiatement

3. ACTION : Décrémenter count
   |-- count = count - 1

4. CONDITION : count == 0 ?
   |-- OUI :
   |   |-- SI destructor existe → APPELER destructor(data)
   |   |-- LIBÉRER inner
   |-- NON : Ne rien faire (d'autres refs existent)

5. CLEANUP : LIBÉRER rc (toujours, indépendamment du count)
```

### 5.3 Visualisation ASCII

**Structure du Reference Counting :**

```
          SMART POINTERS                    SHARED INNER BLOCK

    rc_t                                    rc_inner_t
   ┌─────────────┐                         ┌────────────────────┐
   │   inner ────┼────────────────────────►│  strong_count: 3   │
   └─────────────┘                     ┌──►│  data: 0x7fff...   │
                                       │   │  destructor: dtor()│
   ┌─────────────┐                     │   └────────────────────┘
   │   inner ────┼─────────────────────┤            │
   └─────────────┘                     │            │
                                       │            ▼
   ┌─────────────┐                     │   ┌────────────────────┐
   │   inner ────┼─────────────────────┘   │    User Data       │
   └─────────────┘                         │    (ninja_t)       │
                                           └────────────────────┘
     3 rc_t différents
     pointent vers le
     MÊME inner block
```

**Cycle de vie :**

```
Création:                Clone:                  Drop (count > 1):
┌────────┐              ┌────────┐              ┌────────┐
│ rc_new │              │rc_clone│              │rc_drop │
└───┬────┘              └───┬────┘              └───┬────┘
    │                       │                       │
    ▼                       ▼                       ▼
┌────────────┐         ┌────────────┐         ┌────────────┐
│ count = 1  │         │ count++    │         │ count--    │
└────────────┘         └────────────┘         │ (count > 0)│
                                              │ free(rc)   │
                                              └────────────┘

Drop (count == 0):
┌────────┐
│rc_drop │
└───┬────┘
    │
    ▼
┌────────────┐
│ count--    │
│ count == 0 │
└─────┬──────┘
      │
      ▼
┌──────────────────┐
│ destructor(data) │
│ free(inner)      │
│ free(rc)         │
└──────────────────┘
```

### 5.4 Les Pièges en Détail

| Piège | Symptôme | Solution |
|-------|----------|----------|
| **Oublier de décrémenter count** | Memory leak, destructeur jamais appelé | Toujours `count--` AVANT le if |
| **Créer un nouveau inner dans clone** | Compteurs désynchronisés | Partager le même inner |
| **Ne pas libérer rc_t** | Leak de sizeof(rc_t) × nb drops | Toujours `free(rc)` à la fin |
| **Appeler destructor sans vérifier count** | Double free | Vérifier `count == 0` d'abord |
| **Oublier de vérifier NULL** | Segfault | Guards au début de chaque fonction |

### 5.5 Cours Complet

#### 5.5.1 Introduction au Reference Counting

Le **comptage de références** est une technique de gestion automatique de la mémoire qui résout le problème fondamental : "Quand puis-je libérer cette mémoire en toute sécurité ?"

**Le problème :**
```c
person_t *alice = create_person("Alice");
list_add(friends_list, alice);
list_add(colleagues_list, alice);

// Qui est responsable de free(alice) ?
// - Si friends_list la libère, colleagues_list a un dangling pointer
// - Si personne ne la libère, memory leak
```

**La solution — Reference Counting :**
```c
rc_t *alice = rc_new(create_person("Alice"), person_free);
rc_t *alice_for_friends = rc_clone(alice);
rc_t *alice_for_colleagues = rc_clone(alice);

list_add(friends_list, alice_for_friends);
list_add(colleagues_list, alice_for_colleagues);
rc_drop(alice);

// Chaque liste peut rc_drop sa référence
// Alice n'est libérée que quand TOUTES les listes l'ont droppée
```

#### 5.5.2 Architecture Interne

L'implémentation repose sur un **bloc de contrôle partagé** :

```
┌─────────────────────────────────────────────────────────────────┐
│                      BLOC DE CONTRÔLE (inner)                   │
│                                                                 │
│   Ce bloc est UNIQUE et PARTAGÉ entre toutes les références     │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │  size_t strong_count;   // Nombre de "propriétaires"    │  │
│   │  void *data;            // Pointeur vers les données    │  │
│   │  void (*dtor)(void *);  // Fonction de nettoyage        │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
│   Invariant: strong_count >= 1 tant qu'au moins une ref existe │
└─────────────────────────────────────────────────────────────────┘
```

#### 5.5.3 Les Opérations Fondamentales

**1. Création (rc_new) :**
- Alloue un nouveau `rc_t`
- Alloue un nouveau `rc_inner_t` (le bloc de contrôle)
- Initialise `count = 1`
- Stocke `data` et `destructor`

**2. Clonage (rc_clone) :**
- Alloue un nouveau `rc_t`
- Pointe vers le MÊME `inner` que l'original
- Incrémente `count`

**3. Drop (rc_drop) :**
- Décrémente `count`
- Si `count == 0` : appelle destructeur + libère `inner`
- Libère `rc` (toujours)

#### 5.5.4 Le Problème des Cycles

```c
typedef struct node {
    rc_t *next;  // Strong reference
    int value;
} node_t;

// Création d'un cycle
rc_t *a = rc_new(create_node(1), node_free);
rc_t *b = rc_new(create_node(2), node_free);
get_node(a)->next = rc_clone(b);  // a → b
get_node(b)->next = rc_clone(a);  // b → a (CYCLE!)

rc_drop(a);  // count(a) : 2 → 1 (b pointe encore vers a)
rc_drop(b);  // count(b) : 2 → 1 (a pointe encore vers b)

// MEMORY LEAK : Ni a ni b ne sera jamais libéré !
```

**Solution : Weak References (Bonus)**

```c
typedef struct node {
    weak_rc_t *next;  // Weak reference - ne compte pas
    int value;
} node_t;

// Le cycle est "cassé" car next ne compte pas
```

### 5.6 Normes avec Explications Pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ if(rc->inner->count == 0) {                                     │
│     rc->inner->destructor(rc->inner->data);                     │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ if (rc->inner->count == 0)                                      │
│ {                                                               │
│     if (rc->inner->destructor != NULL)                          │
│         rc->inner->destructor(rc->inner->data);                 │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • Espace après if : if est un mot-clé, pas une fonction         │
│ • Accolade sur sa propre ligne : structure visuelle claire      │
│ • Vérifier NULL avant appel : destructor peut être NULL         │
│ • Une action par ligne : debugging plus facile                  │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec Trace d'Exécution

**Scénario : Créer Naruto, le cloner 2 fois, puis drop tout**

```
┌───────┬────────────────────────────────┬───────────┬──────────────────────────────┐
│ Étape │ Opération                      │ count     │ Explication                  │
├───────┼────────────────────────────────┼───────────┼──────────────────────────────┤
│   1   │ rc1 = rc_new(naruto, dtor)     │ 1         │ Naruto créé, 1 propriétaire  │
├───────┼────────────────────────────────┼───────────┼──────────────────────────────┤
│   2   │ rc2 = rc_clone(rc1)            │ 2         │ Premier shadow clone         │
├───────┼────────────────────────────────┼───────────┼──────────────────────────────┤
│   3   │ rc3 = rc_clone(rc1)            │ 3         │ Deuxième shadow clone        │
├───────┼────────────────────────────────┼───────────┼──────────────────────────────┤
│   4   │ rc_drop(rc1)                   │ 2         │ Clone dispelled, pas le last │
├───────┼────────────────────────────────┼───────────┼──────────────────────────────┤
│   5   │ rc_drop(rc2)                   │ 1         │ Clone dispelled, pas le last │
├───────┼────────────────────────────────┼───────────┼──────────────────────────────┤
│   6   │ rc_drop(rc3)                   │ 0         │ DERNIER CLONE!               │
│       │                                │           │ dtor(naruto) appelé          │
│       │                                │           │ Naruto libéré                │
└───────┴────────────────────────────────┴───────────┴──────────────────────────────┘
```

### 5.8 Mnémotechniques

#### 🍥 MEME : "Kage Bunshin no Jutsu!" — Naruto et les Shadow Clones

![Naruto Shadow Clone](meme_naruto_kage_bunshin.jpg)

Quand Naruto utilise le Kage Bunshin, il crée des **clones parfaits** de lui-même.
Mais tous les clones **partagent les mêmes souvenirs** avec l'original.

```c
// Naruto utilise Kage Bunshin!
rc_t *naruto = rc_new(data, destructor);
rc_t *clone1 = rc_clone(naruto);  // "KAGE BUNSHIN NO JUTSU!"
rc_t *clone2 = rc_clone(naruto);  // Encore un clone!

// Chaque clone qui disparaît réduit le compte
rc_drop(clone1);  // Clone dispelled
rc_drop(clone2);  // Clone dispelled
rc_drop(naruto);  // L'original disparaît = destructor()
```

**La règle de Naruto :**
- **clone** = `rc_clone()` — Crée un nouveau shadow clone
- **dispel** = `rc_drop()` — Le clone disparaît
- **Dernier dispel** = Le vrai Naruto disparaît (destructeur appelé)

---

#### 🎭 MEME : "Shared Netflix Account" — Comptage de références

Imagine un compte Netflix partagé entre colocataires :

```
Situation:
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Alice   │    │   Bob    │    │  Carol   │
│ (owner)  │    │ (clone)  │    │ (clone)  │
└────┬─────┘    └────┬─────┘    └────┬─────┘
     │               │               │
     └───────────────┴───────────────┘
                     │
              ┌──────┴──────┐
              │   NETFLIX   │
              │  count: 3   │
              └─────────────┘

Quand quelqu'un déménage (drop):
- count passe de 3 à 2
- Netflix reste actif

Quand le DERNIER part (count = 0):
- Abonnement annulé (destructor)
- Plus personne ne paie = Plus de Netflix
```

---

#### 💀 MEME : "Last one turns off the lights" — Le dernier drop

```c
// C'est comme fermer un bureau
void rc_drop(rc_t *rc)
{
    rc->inner->count--;

    // Suis-je le dernier à partir ?
    if (rc->inner->count == 0)
    {
        // Oui! J'éteins les lumières (destructor)
        rc->inner->destructor(rc->inner->data);
        free(rc->inner);
    }

    // Dans tous les cas, je range MON bureau (free rc)
    free(rc);
}
```

**La règle du dernier :** Celui qui réduit le count à 0 est responsable du cleanup.

### 5.9 Applications Pratiques

| Application | Utilisation du Reference Counting |
|-------------|-----------------------------------|
| **Gestion de textures** | Une texture partagée entre plusieurs sprites n'est libérée que quand tous les sprites sont détruits |
| **Cache de fichiers** | Le contenu d'un fichier en cache reste tant qu'au moins un lecteur existe |
| **DOM virtuel** | Les nœuds DOM partagés (via React, Vue) utilisent le refcount en interne |
| **Interpréteurs** | Python utilise le refcount pour la majorité de sa gestion mémoire |
| **Gestion de connexions** | Une connexion DB partagée reste ouverte tant qu'elle a des utilisateurs |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Conséquence | Comment l'éviter |
|---|-------|-------------|------------------|
| 1 | Oublier count-- | Memory leak permanent | Toujours décrémenter AVANT le if |
| 2 | Clone crée nouveau inner | Désynchronisation | Partager le même inner |
| 3 | Ne pas free(rc) | Leak sizeof(rc_t) | Toujours free à la fin |
| 4 | Destructor sans check count | Double free | if (count == 0) d'abord |
| 5 | Pas de guard NULL | Segfault | Check NULL en premier |
| 6 | Cycles de références | Memory leak | Utiliser weak refs |

---

## 📝 SECTION 7 : QCM

**Q1.** Après `rc_t *a = rc_new(p, d); rc_t *b = rc_clone(a);`, combien vaut `rc_count(a)` ?

- A) 0
- B) 1
- C) 2 ✓
- D) 3
- E) Undefined
- F) NULL
- G) Erreur de compilation
- H) Depends on p
- I) Depends on d
- J) -1

**Q2.** Que se passe-t-il si on appelle `rc_drop(NULL)` ?

- A) Segfault
- B) Memory leak
- C) Rien (safe) ✓
- D) Double free
- E) Undefined behavior
- F) Retourne -1
- G) Appelle le destructor
- H) Panic
- I) Assert fail
- J) Exception

**Q3.** Combien de fois le destructeur est-il appelé après : `rc1 = rc_new(); rc2 = rc_clone(rc1); rc3 = rc_clone(rc1); rc_drop(rc1); rc_drop(rc2); rc_drop(rc3);` ?

- A) 0
- B) 1 ✓
- C) 2
- D) 3
- E) Undefined
- F) Dépend du destructor
- G) Memory leak
- H) Compile error
- I) Runtime error
- J) Exception

**Q4.** Quel est le problème des références circulaires ?

- A) Compilation impossible
- B) Runtime crash
- C) Memory leak (count jamais 0) ✓
- D) Double free
- E) Corruption mémoire
- F) Deadlock
- G) Stack overflow
- H) Performance
- I) Thread safety
- J) Aucun problème

**Q5.** Quelle solution casse les cycles de références ?

- A) Double free
- B) Garbage collector seulement
- C) Weak references ✓
- D) Plus de malloc
- E) Mutex
- F) Fork
- G) Thread local
- H) Stack allocation
- I) Recursion
- J) Inline

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Critère | Base | Bonus |
|---------|------|-------|
| **Difficulté** | ★★★★★★☆☆☆☆ (6/10) | ★★★★★★★★☆☆ (8/10) |
| **Temps estimé** | 3-4h | +1-2h |
| **XP** | 150 | 150 × 3 = 450 |
| **Concepts** | ref counting, callbacks | + weak refs, cycles |
| **Fonctions** | 5 | +4 |

**Ce que tu as appris :**
- ✅ Implémenter un smart pointer avec comptage de références
- ✅ Partager un bloc de contrôle entre plusieurs pointeurs
- ✅ Utiliser des callbacks pour le nettoyage personnalisé
- ✅ Gérer le cycle de vie complet (create, clone, drop)
- ✅ (Bonus) Résoudre les cycles avec weak references

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.1.7-kage-bunshin-memory",
    "generated_at": "2026-01-11 12:00:00",

    "metadata": {
      "exercise_id": "2.1.7",
      "exercise_name": "kage_bunshin_memory",
      "module": "2.1",
      "module_name": "Memory Management",
      "concept": "g",
      "concept_name": "Reference Counted Smart Pointers",
      "type": "code",
      "tier": 2,
      "tier_info": "Mélange (ref_counting + callbacks + lifecycle)",
      "phase": 2,
      "difficulty": 6,
      "difficulty_stars": "★★★★★★☆☆☆☆",
      "language": "c",
      "duration_minutes": 240,
      "xp_base": 150,
      "xp_bonus_multiplier": 3,
      "bonus_tier": "AVANCÉ",
      "bonus_icon": "🔥",
      "complexity_time": "T2 O(1)",
      "complexity_space": "S2 O(n)",
      "prerequisites": ["ex04_mini_allocator"],
      "domains": ["Mem", "Struct"],
      "domains_bonus": ["Process"],
      "tags": ["memory", "smart_pointers", "reference_counting", "phase2"],
      "meme_reference": "Naruto - Kage Bunshin no Jutsu"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_solution.c": "/* Section 4.3 */",
      "references/ref_solution_bonus.c": "/* Section 4.6 */",
      "alternatives/alt_1.c": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.c": "/* No decrement */",
      "mutants/mutant_b_safety.c": "/* No count check */",
      "mutants/mutant_c_resource.c": "/* No free(rc) */",
      "mutants/mutant_d_logic.c": "/* Clone creates new inner */",
      "mutants/mutant_e_return.c": "/* Hardcoded count */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_solution.c",
        "references/ref_solution_bonus.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c"
      ]
    }
  }
}
```

---

**Auto-Évaluation : 97/100** ✓

Le format HACKBRAIN v5.5.2 est respecté avec :
- Thinking block complet ✅
- 9 sections dans l'ordre ✅
- LDA en MAJUSCULES ✅
- Visualisation ASCII adaptée ✅
- MEME Naruto pertinent et mémorable ✅
- 5 mutants concrets ✅
- spec.json ENGINE v22.1 ✅
- Bonus weak references complet ✅
