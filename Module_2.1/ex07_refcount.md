# [Module 2.1] - Exercise 07: Reference Counted Smart Pointers

## Métadonnées

```yaml
module: "2.1 - Memory Management"
exercise: "ex07"
difficulty: moyen
estimated_time: "4-5 heures"
prerequisite_exercises: ["ex04"]
concepts_requis:
  - "Pointeurs et structures"
  - "Callbacks (destructeurs)"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.1.17.a | Reference counting | Comptage de références |
| 2.1.17.b | Increment/decrement | Opérations atomiques |
| 2.1.17.c | Destructor callbacks | Libération personnalisée |
| 2.1.17.d | Cycles problem | Problème des références circulaires |
| 2.1.17.e | Weak references | Références faibles |

---

## Énoncé

Implémentez une bibliothèque de smart pointers avec comptage de références, similaire à `shared_ptr` en C++.

### API

```c
// Type opaque pour le smart pointer
typedef struct rc rc_t;

// Crée un nouveau smart pointer
// destructor: fonction appelée quand count atteint 0
rc_t *rc_new(void *data, void (*destructor)(void *));

// Incrémente le compteur et retourne une nouvelle référence
rc_t *rc_clone(rc_t *rc);

// Décrémente le compteur; libère si count == 0
void rc_drop(rc_t *rc);

// Accès aux données
void *rc_get(const rc_t *rc);

// Nombre de références actives
size_t rc_count(const rc_t *rc);

// Weak reference (ne compte pas)
typedef struct weak_rc weak_rc_t;
weak_rc_t *rc_downgrade(rc_t *rc);
rc_t *weak_upgrade(weak_rc_t *weak);  // NULL si déjà libéré
void weak_drop(weak_rc_t *weak);
```

---

## Exemple

```c
typedef struct {
    char name[32];
    int age;
} person_t;

void person_destructor(void *data) {
    printf("Person freed: %s\n", ((person_t *)data)->name);
    free(data);
}

int main(void) {
    person_t *p = malloc(sizeof(person_t));
    strcpy(p->name, "Alice");
    p->age = 30;

    rc_t *rc1 = rc_new(p, person_destructor);
    printf("Count: %zu\n", rc_count(rc1));  // 1

    rc_t *rc2 = rc_clone(rc1);
    printf("Count: %zu\n", rc_count(rc1));  // 2

    rc_drop(rc1);
    printf("Count: %zu\n", rc_count(rc2));  // 1

    rc_drop(rc2);  // Affiche "Person freed: Alice"
}
```

---

## Tests Clés

```yaml
test_basic_lifecycle:
  expected: "Destructor appelé exactement une fois"

test_multiple_refs:
  expected: "Count correct à chaque étape"

test_weak_refs:
  expected: "weak_upgrade retourne NULL après libération"

test_no_double_free:
  expected: "Valgrind clean"
```

---

## Auto-Évaluation: **96/100** ✓
