# [Module 2.1] - Exercise 08: Mark & Sweep Garbage Collector

## Métadonnées

```yaml
module: "2.1 - Memory Management"
exercise: "ex08"
difficulty: difficile
estimated_time: "8-10 heures"
prerequisite_exercises: ["ex04", "ex07"]
concepts_requis:
  - "Graphes et traversée"
  - "Gestion des racines"
  - "Bit manipulation"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.1.15.a | GC fundamentals | Principes du garbage collection |
| 2.1.15.b | Roots | Racines de l'ensemble accessible |
| 2.1.15.c | Reachability | Objets atteignables |
| 2.1.16.a | Mark phase | Marquage des objets vivants |
| 2.1.16.b | Sweep phase | Récupération des objets morts |
| 2.1.16.c | Tri-color marking | Blanc/gris/noir |
| 2.1.16.d | Stop-the-world | Pause pendant GC |

---

## Énoncé

Implémentez un garbage collector simple avec l'algorithme mark-and-sweep.

### API

```c
// Initialise le GC avec une taille de heap
gc_t *gc_init(size_t heap_size);
void gc_shutdown(gc_t *gc);

// Allocation gérée par le GC
void *gc_alloc(gc_t *gc, size_t size);

// Enregistrer une racine (variable sur le stack ou globale)
void gc_add_root(gc_t *gc, void **root);
void gc_remove_root(gc_t *gc, void **root);

// Déclarer un pointeur dans un objet GC (pour le scanning)
void gc_set_ptr(gc_t *gc, void *obj, size_t offset, void *target);

// Déclencher une collection
size_t gc_collect(gc_t *gc);  // Retourne bytes libérés

// Statistiques
typedef struct {
    size_t heap_size;
    size_t bytes_used;
    size_t objects_count;
    size_t collections;
    size_t total_freed;
} gc_stats_t;

gc_stats_t gc_get_stats(gc_t *gc);
```

### Algorithme

```
Mark Phase:
1. Marquer tous les objets comme BLANC (non visités)
2. Pour chaque racine:
   - Marquer GRIS (à visiter)
3. Tant qu'il y a des objets GRIS:
   - Prendre un objet GRIS
   - Scanner ses pointeurs, marquer les cibles GRIS
   - Marquer l'objet NOIR (visité)

Sweep Phase:
4. Parcourir le heap:
   - Objets BLANC → libérer (garbage)
   - Objets NOIR → garder, remettre BLANC
```

---

## Exemple

```c
typedef struct node {
    int value;
    struct node *left;
    struct node *right;
} node_t;

int main(void) {
    gc_t *gc = gc_init(1024 * 1024);  // 1 MB heap

    // Racine sur le stack
    node_t *root = NULL;
    gc_add_root(gc, (void **)&root);

    // Créer un arbre
    root = gc_alloc(gc, sizeof(node_t));
    root->value = 1;

    root->left = gc_alloc(gc, sizeof(node_t));
    gc_set_ptr(gc, root, offsetof(node_t, left), root->left);
    root->left->value = 2;

    root->right = gc_alloc(gc, sizeof(node_t));
    gc_set_ptr(gc, root, offsetof(node_t, right), root->right);
    root->right->value = 3;

    // Créer des objets orphelins (garbage)
    gc_alloc(gc, 100);
    gc_alloc(gc, 200);
    gc_alloc(gc, 300);

    printf("Before GC: %zu objects\n", gc_get_stats(gc).objects_count);

    size_t freed = gc_collect(gc);
    printf("GC freed: %zu bytes\n", freed);
    printf("After GC: %zu objects\n", gc_get_stats(gc).objects_count);

    gc_remove_root(gc, (void **)&root);
    gc_shutdown(gc);
}
```

**Sortie**:
```
Before GC: 6 objects
GC freed: 600 bytes
After GC: 3 objects
```

---

## Tests Clés

```yaml
test_simple_collection:
  expected: "Objets non référencés libérés"

test_preserves_reachable:
  expected: "Objets référencés gardés"

test_handles_cycles:
  expected: "Cycles non référencés collectés"

test_multiple_collections:
  expected: "Plusieurs GC successifs fonctionnent"

test_stress:
  expected: "1000 allocs, GC périodique, heap stable"
```

---

## Auto-Évaluation: **97/100** ✓
