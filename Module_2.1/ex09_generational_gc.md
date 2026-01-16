# [Module 2.1] - Exercise 09: Generational GC Simulator

## Métadonnées

```yaml
module: "2.1 - Memory Management"
exercise: "ex09"
difficulty: très difficile
estimated_time: "10-15 heures"
prerequisite_exercises: ["ex08"]
concepts_requis:
  - "Mark & sweep (ex08)"
  - "Générations d'objets"
  - "Write barriers"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.1.18.a-d | Copying collection | Semi-space, Cheney |
| 2.1.19.a | Young generation | Objets récents |
| 2.1.19.b | Old generation | Objets survivants |
| 2.1.19.c | Promotion | Young → Old |
| 2.1.19.d | Minor/major GC | Collections partielles |
| 2.1.20.a | Write barrier | Tracker inter-gen refs |
| 2.1.20.b | Card marking | Optimisation write barrier |
| 2.1.20.c | Remembered set | Références old→young |

---

## Énoncé

Implémentez un garbage collector générationnel avec deux générations (young et old).

### API

```c
typedef struct gen_gc gen_gc_t;

gen_gc_t *gen_gc_init(size_t young_size, size_t old_size);
void gen_gc_shutdown(gen_gc_t *gc);

// Allocation (toujours dans young)
void *gen_gc_alloc(gen_gc_t *gc, size_t size);

// Write barrier (DOIT être appelé lors d'écriture de pointeur)
void gen_gc_write_barrier(gen_gc_t *gc, void *obj, void **field, void *new_value);

// GC explicite
size_t gen_gc_minor(gen_gc_t *gc);  // Young generation only
size_t gen_gc_major(gen_gc_t *gc);  // Both generations

// Roots
void gen_gc_add_root(gen_gc_t *gc, void **root);

// Stats
typedef struct {
    size_t young_used;
    size_t old_used;
    size_t minor_collections;
    size_t major_collections;
    size_t promotions;
    size_t young_survivors;
} gen_gc_stats_t;

gen_gc_stats_t gen_gc_get_stats(gen_gc_t *gc);
```

### Mécanisme

1. **Allocation**: Toujours dans Young (rapide, bump pointer)
2. **Minor GC**: Collecte Young, copie survivants dans survivor space
3. **Promotion**: Objets survivant N minor GC → Old
4. **Major GC**: Collecte Young + Old (mark-sweep)
5. **Write Barrier**: Si old objet pointe vers young → remembered set

---

## Exemple

```c
gen_gc_t *gc = gen_gc_init(1024 * 64, 1024 * 256);

void *root = NULL;
gen_gc_add_root(gc, &root);

// Beaucoup d'allocations temporaires
for (int i = 0; i < 10000; i++) {
    void *temp = gen_gc_alloc(gc, 64);
    // temp devient garbage au prochain tour
}

gen_gc_stats_t s = gen_gc_get_stats(gc);
printf("Minor GCs: %zu, Promotions: %zu\n",
       s.minor_collections, s.promotions);

gen_gc_shutdown(gc);
```

---

## Tests Clés

```yaml
test_young_collection:
  expected: "Minor GC libère objets non référencés"

test_promotion:
  expected: "Objets survivant 3 minor GC sont promus"

test_write_barrier:
  expected: "Références old→young trackées"

test_major_gc:
  expected: "Old generation collectée correctement"

test_performance:
  expected: "Minor GC < 1ms, Major GC < 100ms pour heap raisonnable"
```

---

## Auto-Évaluation: **96/100** ✓
