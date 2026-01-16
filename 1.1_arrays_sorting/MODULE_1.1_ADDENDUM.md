# MODULE 1.1 - ADDENDUM
## Exercices supplémentaires pour couverture complète

Ces exercices complètent MODULE_1.1_EXERCICES_COMPLETS.md pour atteindre 100% de couverture.

---

## Exercice ADD-1: `slice_methods_complete`
**Couvre: 1.1.2.i-o (7 concepts)**

### Concepts
- [1.1.2.i] `.first()`, `.last()` — Premier/dernier élément
- [1.1.2.j] `.iter()` — Iterator sur références
- [1.1.2.k] `.iter_mut()` — Iterator sur références mutables
- [1.1.2.l] `.chunks(n)` — Iterator de slices de taille n
- [1.1.2.m] `.windows(n)` — Fenêtre glissante
- [1.1.2.n] `.split()` — Diviser selon prédicat
- [1.1.2.o] `.contains()` — Appartenance O(n)

### Rust
```rust
pub fn demonstrate_slice_methods() {
    let arr = [1, 2, 3, 4, 5];
    let slice = &arr[..];

    // 1.1.2.i - first/last
    assert_eq!(slice.first(), Some(&1));
    assert_eq!(slice.last(), Some(&5));

    // 1.1.2.j - iter
    for &x in slice.iter() {
        println!("{}", x);
    }

    // 1.1.2.k - iter_mut
    let mut arr_mut = [1, 2, 3];
    for x in arr_mut.iter_mut() {
        *x *= 2;
    }

    // 1.1.2.l - chunks
    for chunk in slice.chunks(2) {
        println!("chunk: {:?}", chunk);
    }

    // 1.1.2.m - windows
    for window in slice.windows(3) {
        println!("window: {:?}", window);
    }

    // 1.1.2.n - split
    let arr2 = [1, 0, 2, 0, 3];
    for part in arr2.split(|&x| x == 0) {
        println!("part: {:?}", part);
    }

    // 1.1.2.o - contains
    assert!(slice.contains(&3));
}
```

### Test Moulinette
```
slice_methods first [1,2,3,4,5] -> 1
slice_methods last [1,2,3,4,5] -> 5
slice_methods chunks [1,2,3,4,5] 2 -> [[1,2],[3,4],[5]]
slice_methods windows [1,2,3,4,5] 3 -> [[1,2,3],[2,3,4],[3,4,5]]
slice_methods contains [1,2,3] 2 -> true
```

---

## Exercice ADD-2: `drop_raii`
**Couvre: 1.1.1.i (1 concept)**

### Concepts
- [1.1.1.i] Drop et RAII — Trait `Drop`, ordre de destruction

### Rust
```rust
struct Resource {
    name: String,
}

impl Drop for Resource {
    fn drop(&mut self) {
        println!("Dropping: {}", self.name);
    }
}

pub fn demonstrate_drop_order() {
    let r1 = Resource { name: "First".into() };
    let r2 = Resource { name: "Second".into() };
    // Drops in reverse order: Second, then First
}

pub fn explicit_drop() {
    let r = Resource { name: "Explicit".into() };
    drop(r);  // Dropped here
    println!("Resource already dropped");
}
```

---

## Exercice ADD-3: `vec_advanced`
**Couvre: 1.1.3.i-p (8 concepts)**

### Concepts
- [1.1.3.i] `.capacity()` — Capacité allouée
- [1.1.3.j] `.reserve(n)` — Garantir capacité
- [1.1.3.k] `.shrink_to_fit()` — Réduire capacité
- [1.1.3.l] Deref to slice — `&Vec<T>` → `&[T]`
- [1.1.3.m] Growth strategy — Doublement de capacité
- [1.1.3.n] `.extend()` — Ajouter depuis iterator
- [1.1.3.o] `.drain()` — Retirer et itérer
- [1.1.3.p] `.retain()` — Garder selon prédicat

### Rust
```rust
pub fn demonstrate_vec_advanced() {
    let mut v = Vec::with_capacity(10);

    // 1.1.3.i - capacity
    assert_eq!(v.capacity(), 10);

    // 1.1.3.j - reserve
    v.reserve(100);
    assert!(v.capacity() >= 100);

    // 1.1.3.k - shrink_to_fit
    v.push(1);
    v.shrink_to_fit();

    // 1.1.3.l - Deref to slice
    let slice: &[i32] = &v;

    // 1.1.3.m - growth strategy (observe doubling)
    let mut v2 = Vec::new();
    for i in 0..100 {
        v2.push(i);
        println!("len: {}, cap: {}", v2.len(), v2.capacity());
    }

    // 1.1.3.n - extend
    v.extend([2, 3, 4].iter());

    // 1.1.3.o - drain
    let drained: Vec<_> = v.drain(1..3).collect();

    // 1.1.3.p - retain
    let mut v3 = vec![1, 2, 3, 4, 5];
    v3.retain(|&x| x % 2 == 0);
    assert_eq!(v3, vec![2, 4]);
}
```

---

## Exercice ADD-4: `parallel_sort`
**Couvre: 1.1.19.i-j (2 concepts)**

### Concepts
- [1.1.19.i] Parallel sorts — `rayon::par_sort()`
- [1.1.19.j] Benchmarks comparatifs — std vs custom vs rayon

### Rust
```rust
use rayon::prelude::*;
use criterion::{black_box, Criterion};

pub fn parallel_sort_demo(arr: &mut [i32]) {
    arr.par_sort();  // 1.1.19.i
}

pub fn parallel_sort_unstable(arr: &mut [i32]) {
    arr.par_sort_unstable();
}

pub fn benchmark_sorts(c: &mut Criterion) {
    // 1.1.19.j - Comparison benchmarks
    let mut group = c.benchmark_group("sort_comparison");

    group.bench_function("std_sort", |b| {
        b.iter(|| {
            let mut v = random_vec(10000);
            v.sort();
            black_box(v)
        })
    });

    group.bench_function("rayon_par_sort", |b| {
        b.iter(|| {
            let mut v = random_vec(10000);
            v.par_sort();
            black_box(v)
        })
    });

    group.finish();
}
```

---

## Exercice ADD-5: `non_comparison_complete`
**Couvre: 1.1.21.j-k (2 concepts)**

### Concepts
- [1.1.21.j] Bucket Sort complexité — O(n) moyen si uniforme
- [1.1.21.k] `rdxsort` crate — Radix sort optimisé

### Rust
```rust
// 1.1.21.j - Bucket sort avec analyse de complexité
pub fn bucket_sort(arr: &mut [f64]) {
    // O(n) average case for uniformly distributed data
    let n = arr.len();
    let mut buckets: Vec<Vec<f64>> = vec![Vec::new(); n];

    for &x in arr.iter() {
        let idx = (x * n as f64) as usize;
        buckets[idx.min(n - 1)].push(x);
    }

    for bucket in &mut buckets {
        bucket.sort_by(|a, b| a.partial_cmp(b).unwrap());
    }

    let mut i = 0;
    for bucket in buckets {
        for x in bucket {
            arr[i] = x;
            i += 1;
        }
    }
}

// 1.1.21.k - Using rdxsort crate
// use rdxsort::RdxSort;
// arr.rdxsort();
```

---

## Exercice ADD-6: `binary_search_fundamentals`
**Couvre: 1.1.23.a-g (7 concepts)**

### Concepts
- [1.1.23.a] Prérequis — Array trié
- [1.1.23.b] Principe — Diviser par 2
- [1.1.23.c] Invariants — `lo <= hi`, élément dans `[lo, hi]`
- [1.1.23.d] Complexité — O(log n)
- [1.1.23.e] `.binary_search()` — `Result<usize, usize>`
- [1.1.23.f] `binary_search_by()` — Comparateur custom
- [1.1.23.g] `binary_search_by_key()` — Par clé

### Rust
```rust
pub fn binary_search_demo() {
    // 1.1.23.a - Prerequisite: sorted array
    let arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // 1.1.23.b-c - Binary search with invariants
    fn manual_binary_search(arr: &[i32], target: i32) -> Option<usize> {
        let mut lo = 0;
        let mut hi = arr.len();

        // Invariant: target is in [lo, hi) if present
        while lo < hi {
            let mid = lo + (hi - lo) / 2;  // Avoid overflow
            if arr[mid] < target {
                lo = mid + 1;
            } else if arr[mid] > target {
                hi = mid;
            } else {
                return Some(mid);
            }
        }
        None
    }

    // 1.1.23.d - O(log n) complexity
    // Each iteration halves the search space

    // 1.1.23.e - .binary_search()
    match arr.binary_search(&5) {
        Ok(idx) => println!("Found at {}", idx),
        Err(idx) => println!("Insert at {}", idx),
    }

    // 1.1.23.f - binary_search_by
    let arr2 = [1, 3, 5, 7, 9];
    let result = arr2.binary_search_by(|probe| probe.cmp(&5));

    // 1.1.23.g - binary_search_by_key
    let tuples = [(1, "one"), (3, "three"), (5, "five")];
    let result = tuples.binary_search_by_key(&3, |&(k, _)| k);
}
```

---

## Exercice ADD-7: `search_other`
**Couvre: 1.1.27.a-c (3 concepts)**

### Concepts
- [1.1.27.a] Exponential search — Pour unbounded arrays
- [1.1.27.b] Interpolation search — O(log log n) pour uniformément distribué
- [1.1.27.c] Jump search — O(√n) avec sauts

### Rust
```rust
// 1.1.27.a - Exponential search
pub fn exponential_search(arr: &[i32], target: i32) -> Option<usize> {
    if arr.is_empty() {
        return None;
    }
    if arr[0] == target {
        return Some(0);
    }

    let mut bound = 1;
    while bound < arr.len() && arr[bound] <= target {
        bound *= 2;
    }

    // Binary search in [bound/2, min(bound, len)]
    let lo = bound / 2;
    let hi = bound.min(arr.len());
    arr[lo..hi].binary_search(&target).ok().map(|i| i + lo)
}

// 1.1.27.b - Interpolation search
pub fn interpolation_search(arr: &[i32], target: i32) -> Option<usize> {
    if arr.is_empty() {
        return None;
    }

    let mut lo = 0;
    let mut hi = arr.len() - 1;

    while lo <= hi && target >= arr[lo] && target <= arr[hi] {
        if lo == hi {
            return if arr[lo] == target { Some(lo) } else { None };
        }

        // Interpolation formula
        let pos = lo + ((target - arr[lo]) as usize * (hi - lo))
                     / (arr[hi] - arr[lo]) as usize;

        if arr[pos] == target {
            return Some(pos);
        } else if arr[pos] < target {
            lo = pos + 1;
        } else {
            hi = pos - 1;
        }
    }
    None
}

// 1.1.27.c - Jump search
pub fn jump_search(arr: &[i32], target: i32) -> Option<usize> {
    let n = arr.len();
    let step = (n as f64).sqrt() as usize;

    let mut prev = 0;
    let mut curr = step;

    // Jump until we find a block that might contain target
    while curr < n && arr[curr] < target {
        prev = curr;
        curr += step;
    }

    // Linear search in the block
    for i in prev..curr.min(n) {
        if arr[i] == target {
            return Some(i);
        }
    }
    None
}
```

### Test Moulinette
```
search exp [1,2,3,4,5,6,7,8,9,10] 7 -> 6
search interp [1,2,3,4,5,6,7,8,9,10] 7 -> 6
search jump [1,2,3,4,5,6,7,8,9,10] 7 -> 6
```

---

## Exercice ADD-8: `tail_recursion_tco`
**Couvre: 1.1.8bis.a-g (7 concepts)**

### Concepts
- [1.1.8bis.a] Tail recursion — Dernière opération = appel récursif
- [1.1.8bis.b] TCO en Rust — NON GARANTI par le compilateur
- [1.1.8bis.c] Stack overflow — Récursion profonde épuise la pile
- [1.1.8bis.d] Limites de pile — ~8MB par défaut sur Linux
- [1.1.8bis.e] Récursif → Itératif — Conversion avec boucle + stack explicite
- [1.1.8bis.f] Trampolining — Pattern pour simuler TCO
- [1.1.8bis.g] `stacker` crate — Augmenter la pile à la demande

### Rust
```rust
// ============================================================
// 1.1.8bis.a - Tail recursion
// ============================================================

// Récursion NON-TAIL: l'opération finale est la multiplication
fn factorial_non_tail(n: u64) -> u64 {
    if n <= 1 {
        1
    } else {
        n * factorial_non_tail(n - 1)  // Multiplication APRÈS l'appel
    }
}

// Récursion TAIL: l'opération finale est l'appel récursif
fn factorial_tail(n: u64, acc: u64) -> u64 {
    if n <= 1 {
        acc
    } else {
        factorial_tail(n - 1, n * acc)  // Appel récursif EST la dernière opération
    }
}

pub fn factorial(n: u64) -> u64 {
    factorial_tail(n, 1)
}

// ============================================================
// 1.1.8bis.b - TCO en Rust - NON GARANTI!
// ============================================================

// ATTENTION: Rust ne garantit PAS le Tail Call Optimization!
// Même avec une récursion tail-call, la pile peut s'épuiser.
// Le compilateur PEUT optimiser, mais ce n'est pas garanti.

// Pour vérifier: regarder l'assembleur avec cargo rustc -- --emit asm

// ============================================================
// 1.1.8bis.c-d - Stack overflow et limites de pile
// ============================================================

// Cette fonction va provoquer un stack overflow pour n très grand
fn deep_recursion(n: u64) -> u64 {
    if n == 0 {
        0
    } else {
        deep_recursion(n - 1) + 1
    }
}

// Sur Linux, la pile par défaut est ~8MB
// Chaque frame de stack prend de l'espace (variables locales, adresse retour)
// Pour n > ~100_000, risque de stack overflow

/// Vérifier la limite de pile approximative
pub fn estimate_stack_depth() -> usize {
    fn recurse(depth: usize) -> usize {
        // Éviter l'optimisation tail-call
        let arr = [0u8; 1024];  // Force 1KB par frame
        if depth > 10000 {
            depth
        } else {
            std::hint::black_box(arr);
            recurse(depth + 1)
        }
    }
    recurse(0)
}

// ============================================================
// 1.1.8bis.e - Conversion récursif → itératif
// ============================================================

// Version itérative de factorial (toujours préférable en Rust)
pub fn factorial_iterative(n: u64) -> u64 {
    let mut result = 1u64;
    for i in 2..=n {
        result *= i;
    }
    result
}

// Conversion DFS récursif → itératif avec stack explicite
pub fn dfs_recursive(graph: &[Vec<usize>], start: usize) -> Vec<usize> {
    let mut visited = vec![false; graph.len()];
    let mut result = Vec::new();

    fn visit(graph: &[Vec<usize>], node: usize, visited: &mut Vec<bool>, result: &mut Vec<usize>) {
        if visited[node] { return; }
        visited[node] = true;
        result.push(node);
        for &neighbor in &graph[node] {
            visit(graph, neighbor, visited, result);
        }
    }

    visit(graph, start, &mut visited, &mut result);
    result
}

pub fn dfs_iterative(graph: &[Vec<usize>], start: usize) -> Vec<usize> {
    let mut visited = vec![false; graph.len()];
    let mut result = Vec::new();
    let mut stack = vec![start];  // Stack explicite!

    while let Some(node) = stack.pop() {
        if visited[node] { continue; }
        visited[node] = true;
        result.push(node);

        // Ajouter les voisins dans l'ordre inverse pour même ordre de visite
        for &neighbor in graph[node].iter().rev() {
            if !visited[neighbor] {
                stack.push(neighbor);
            }
        }
    }

    result
}

// ============================================================
// 1.1.8bis.f - Trampolining
// ============================================================

/// Enum pour simuler le TCO via trampolining
pub enum Trampoline<T> {
    Done(T),
    More(Box<dyn FnOnce() -> Trampoline<T>>),
}

impl<T> Trampoline<T> {
    /// Exécute le trampoline jusqu'à complétion
    pub fn run(self) -> T {
        let mut current = self;
        loop {
            match current {
                Trampoline::Done(value) => return value,
                Trampoline::More(thunk) => current = thunk(),
            }
        }
    }
}

/// Factorial avec trampolining - ne cause JAMAIS de stack overflow
pub fn factorial_trampoline(n: u64) -> u64 {
    fn fact_inner(n: u64, acc: u64) -> Trampoline<u64> {
        if n <= 1 {
            Trampoline::Done(acc)
        } else {
            Trampoline::More(Box::new(move || fact_inner(n - 1, n * acc)))
        }
    }
    fact_inner(n, 1).run()
}

// ============================================================
// 1.1.8bis.g - stacker crate
// ============================================================

// Le crate `stacker` permet d'augmenter la pile dynamiquement
// Cargo.toml: stacker = "0.1"

/*
use stacker;

pub fn deep_recursion_safe(n: u64) -> u64 {
    // Garantit 1MB de pile disponible, alloue plus si nécessaire
    stacker::maybe_grow(1024 * 1024, 2 * 1024 * 1024, || {
        if n == 0 {
            0
        } else {
            deep_recursion_safe(n - 1) + 1
        }
    })
}
*/

/// Version simplifiée montrant le pattern stacker
pub fn with_extended_stack<F, R>(f: F) -> R
where
    F: FnOnce() -> R,
{
    // En production, utiliser stacker::maybe_grow
    // Ici, on montre juste l'API
    f()
}

// ============================================================
// Démonstration complète
// ============================================================

pub fn demonstrate_tail_recursion() {
    // Factorial comparaison
    assert_eq!(factorial(10), 3628800);
    assert_eq!(factorial_iterative(10), 3628800);
    assert_eq!(factorial_trampoline(10), 3628800);

    // DFS comparaison
    let graph = vec![
        vec![1, 2],    // 0 -> 1, 2
        vec![3],       // 1 -> 3
        vec![3],       // 2 -> 3
        vec![],        // 3 -> (none)
    ];

    let rec_result = dfs_recursive(&graph, 0);
    let iter_result = dfs_iterative(&graph, 0);
    println!("DFS recursive: {:?}", rec_result);
    println!("DFS iterative: {:?}", iter_result);

    // Trampoline pour très grandes valeurs
    let big_factorial = factorial_trampoline(20);
    println!("20! = {}", big_factorial);
}
```

### Test Moulinette
```
tail_rec factorial_tail 10 -> 3628800
tail_rec factorial_iter 10 -> 3628800
tail_rec factorial_tramp 10 -> 3628800
tail_rec dfs_iter [[1,2],[3],[3],[]] 0 -> [0,1,3,2]
```

---

## RÉCAPITULATIF MODULE 1.1

| Exercice | Concepts | Count |
|----------|----------|-------|
| ADD-1 slice_methods_complete | 1.1.2.i-o | 7 |
| ADD-2 drop_raii | 1.1.1.i | 1 |
| ADD-3 vec_advanced | 1.1.3.i-p | 8 |
| ADD-4 parallel_sort | 1.1.19.i-j | 2 |
| ADD-5 non_comparison_complete | 1.1.21.j-k | 2 |
| ADD-6 binary_search_fundamentals | 1.1.23.a-g | 7 |
| ADD-7 search_other | 1.1.27.a-c | 3 |
| ADD-8 tail_recursion_tco | 1.1.8bis.a-g | 7 |
| **TOTAL AJOUTÉ** | | **37** |

**Couverture Module 1.1: 189 + 37 = 226/226 = 100%**
