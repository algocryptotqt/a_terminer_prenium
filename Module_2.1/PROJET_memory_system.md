# [Module 2.1] - PROJET: Complete Memory System

## Metadonnees

```yaml
module: "2.1 - Memory Management"
exercise: "PROJET"
title: "Complete Memory System - Allocator, GC, and Profiler"
difficulty: difficile
estimated_time: "50 heures"
prerequisite_exercises: ["ex01", "ex02", "ex03", "ex04", "ex05"]
concepts_requis: ["virtual memory", "page tables", "allocation basics"]
score_qualite: 98
```

---

## Concepts Couverts

### 2.1.1: Memory Hierarchy & Architecture (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.1.a | Memory hierarchy: Registers → L1 → L2 → L3 → RAM → Disk | Vue d'ensemble hierarchie |
| 2.1.1.b | Locality: Temporal and spatial | Principes de localite |
| 2.1.1.c | Cache lines: 64 bytes typical | Lignes de cache |
| 2.1.1.d | Cache organization: Direct, set-associative, fully | Organisation cache |
| 2.1.1.e | Cache hits/misses: Performance impact | Impact performance |
| 2.1.1.f | Write policies: Write-through, write-back | Politiques d'ecriture |
| 2.1.1.g | Cache coherence: MESI protocol | Coherence cache |
| 2.1.1.h | Memory bandwidth: Bytes per second | Bande passante |
| 2.1.1.i | Latency: Cycles per access level | Latence par niveau |
| 2.1.1.j | Prefetching: Hardware and software | Prefetch |
| 2.1.1.k | TLB: Translation Lookaside Buffer | Buffer de traduction |
| 2.1.1.l | TLB structure: Entries, associativity | Structure TLB |

### 2.1.2: Virtual Memory Concepts (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.2.a | Virtual vs physical: Address spaces | Espaces d'adresses |
| 2.1.2.b | Address translation: Virtual to physical | Traduction d'adresses |
| 2.1.2.c | Page: Fixed-size memory unit | Unite memoire |
| 2.1.2.d | Page size: 4KB default, huge pages | Tailles de pages |
| 2.1.2.e | Page table: Mapping structure | Table de pages |
| 2.1.2.f | Page frame: Physical page | Frame physique |
| 2.1.2.g | PTE: Page table entry contents | Entree de table |
| 2.1.2.h | Present bit: In memory or not | Bit de presence |
| 2.1.2.i | Protection bits: Read, write, execute | Bits de protection |
| 2.1.2.j | Dirty bit: Page modified | Bit modifie |
| 2.1.2.k | Accessed bit: Page referenced | Bit d'acces |
| 2.1.2.l | Page walk: Hardware traversal | Parcours de table |

### 2.1.4: Page Fault Handling (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.4.a | Page fault types: Minor, major, invalid | Types de fautes |
| 2.1.4.b | Minor fault: Page in memory, not mapped | Faute mineure |
| 2.1.4.c | Major fault: Page not in memory | Faute majeure |
| 2.1.4.d | Invalid fault: Access violation | Faute invalide |
| 2.1.4.e | Fault handler: Kernel routine | Gestionnaire de faute |
| 2.1.4.f | Demand paging: Load on access | Pagination a la demande |
| 2.1.4.g | Copy-on-write: Share until write | Copie a l'ecriture |
| 2.1.4.h | Memory-mapped files: mmap | Fichiers mappes |
| 2.1.4.i | Anonymous mapping: No file backing | Mapping anonyme |
| 2.1.4.j | Shared mapping: Multiple processes | Mapping partage |
| 2.1.4.k | Swap: Fault loads from swap | Swap |
| 2.1.4.l | OOM killer: Out of memory handling | Tueur OOM |

### 2.1.6: Memory Allocation - Free Lists (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.6.a | Heap management: malloc/free implementation | Gestion du heap |
| 2.1.6.b | Free list: Linked list of free blocks | Liste des blocs libres |
| 2.1.6.c | Block header: Size, free flag, next pointer | En-tete de bloc |
| 2.1.6.d | First-fit: First block that fits | Premier bloc adequat |
| 2.1.6.e | First-fit analysis: Fast, causes fragmentation | Analyse first-fit |
| 2.1.6.f | Best-fit: Smallest block that fits | Meilleur bloc |
| 2.1.6.g | Best-fit analysis: Less waste, slower | Analyse best-fit |
| 2.1.6.h | Worst-fit: Largest block | Plus grand bloc |
| 2.1.6.i | Next-fit: Continue from last | Prochain bloc |
| 2.1.6.j | Segregated lists: Lists per size class | Listes segregees |
| 2.1.6.k | Size classes: 8, 16, 32, 64, 128... | Classes de tailles |
| 2.1.6.l | Quick lists: Fast allocation for common sizes | Listes rapides |

### 2.1.7: Memory Allocation - Advanced Structures (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.7.a | Buddy system: Power-of-2 allocation | Systeme buddy |
| 2.1.7.b | Buddy split: Divide into two buddies | Division buddy |
| 2.1.7.c | Buddy coalesce: Merge adjacent buddies | Fusion buddy |
| 2.1.7.d | Buddy address: XOR with size | Adresse buddy |
| 2.1.7.e | Buddy bitmap: Track allocations | Bitmap buddy |
| 2.1.7.f | Slab allocator: Object caching | Allocateur slab |
| 2.1.7.g | Slab structure: Pages divided into objects | Structure slab |
| 2.1.7.h | Slab cache: Per object type | Cache slab |
| 2.1.7.i | Slab states: Full, partial, empty | Etats slab |
| 2.1.7.j | Slab coloring: Cache line alignment | Coloration slab |
| 2.1.7.k | SLUB: Linux simplified slab | SLUB Linux |
| 2.1.7.l | SLOB: Simple list of blocks (embedded) | SLOB embarque |

### 2.1.8: Fragmentation (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.8.a | Internal fragmentation: Waste within allocated block | Fragmentation interne |
| 2.1.8.b | External fragmentation: Waste between blocks | Fragmentation externe |
| 2.1.8.c | Memory utilization: Used vs total | Utilisation memoire |
| 2.1.8.d | Compaction: Move blocks together | Compaction |
| 2.1.8.e | Compaction cost: Copy overhead | Cout de compaction |
| 2.1.8.f | Best-fit fragmentation: Small fragments | Fragmentation best-fit |
| 2.1.8.g | First-fit fragmentation: Variable | Fragmentation first-fit |
| 2.1.8.h | Coalescing: Merge adjacent free blocks | Coalescence |
| 2.1.8.i | Boundary tags: Enable bidirectional coalescing | Tags de frontiere |
| 2.1.8.j | Footer: Size at block end | Pied de bloc |
| 2.1.8.k | Arena allocators: Bulk free | Allocateurs arena |
| 2.1.8.l | Fragmentation metrics: Measurement | Metriques |

### 2.1.9: Implementing malloc/free (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.9.a | sbrk(): Extend heap | Extension du heap |
| 2.1.9.b | mmap(): Large allocations | Allocations larges |
| 2.1.9.c | Threshold: sbrk vs mmap | Seuil de decision |
| 2.1.9.d | Block splitting: Divide for smaller request | Division de blocs |
| 2.1.9.e | Minimum block size: Header + alignment | Taille minimale |
| 2.1.9.f | Alignment: 8 or 16 bytes | Alignement |
| 2.1.9.g | Coalescing policy: Immediate vs deferred | Politique de fusion |
| 2.1.9.h | realloc implementation: Resize or copy | Implementation realloc |
| 2.1.9.i | In-place realloc: Extend if possible | Realloc en place |
| 2.1.9.j | memalign implementation: Aligned allocation | Implementation memalign |
| 2.1.9.k | calloc implementation: Allocate + zero | Implementation calloc |
| 2.1.9.l | Free list optimization | Optimisation free list |

### 2.1.10: Coalescing Strategies (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.10.a | Immediate coalescing: On every free | Coalescence immediate |
| 2.1.10.b | Deferred coalescing: Batch during malloc | Coalescence differee |
| 2.1.10.c | Boundary tags: Size at both ends | Tags aux frontieres |
| 2.1.10.d | Previous block: Find via footer | Bloc precedent |
| 2.1.10.e | Next block: Find via header | Bloc suivant |
| 2.1.10.f | Four cases: prev/next free combinations | Quatre cas |
| 2.1.10.g | Performance tradeoffs: Immediate vs deferred | Compromis performance |
| 2.1.10.h | Segregated fit: Per-class free lists | Listes par classe |
| 2.1.10.i | Explicit free list: Double-linked | Liste explicite |
| 2.1.10.j | LIFO ordering: Last freed first | Ordre LIFO |
| 2.1.10.k | Address ordering: Sorted by address | Ordre par adresse |
| 2.1.10.l | Best fit with coalescing | Best-fit avec fusion |

### 2.1.11: Debug Features (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.11.a | Canaries: Magic values at boundaries | Valeurs canari |
| 2.1.11.b | Red zones: Extra space for overflow detection | Zones rouges |
| 2.1.11.c | Poison values: Fill freed memory | Valeurs poison |
| 2.1.11.d | Double-free detection: Check freed flag | Detection double-free |
| 2.1.11.e | Use-after-free: Detect via poison | Detection use-after-free |
| 2.1.11.f | Buffer overflow: Detect via canary | Detection buffer overflow |
| 2.1.11.g | Heap metadata: Track allocations | Metadonnees heap |
| 2.1.11.h | Allocation backtrace: Store call stack | Backtrace |
| 2.1.11.i | Leak detection: Unfreed at exit | Detection de fuites |
| 2.1.11.j | Statistics: Count, sizes, patterns | Statistiques |
| 2.1.11.k | Debug vs release: Conditional compilation | Mode debug |
| 2.1.11.l | MALLOC_CHECK_: glibc debug | Variable debug glibc |

### 2.1.12: Thread-Safe Allocation (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.12.a | Global lock: Simple but slow | Verrou global |
| 2.1.12.b | Contention: Threads waiting | Contention |
| 2.1.12.c | Per-thread caches: Thread-local storage | Caches par thread |
| 2.1.12.d | TLS arenas: Thread-local arenas | Arenas TLS |
| 2.1.12.e | Central heap: Shared pool | Heap central |
| 2.1.12.f | Transfer batches: Bulk movements | Transferts batch |
| 2.1.12.g | False sharing: Cache line ping-pong | Faux partage |
| 2.1.12.h | Padding: Avoid false sharing | Padding |
| 2.1.12.i | Lock-free freelists: CAS-based | Listes lock-free |
| 2.1.12.j | Atomic operations: Compare-and-swap | Operations atomiques |
| 2.1.12.k | Memory barriers: Ordering guarantees | Barrieres memoire |
| 2.1.12.l | Scalability testing | Tests scalabilite |

### 2.1.13: Production Allocators (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.13.a | glibc malloc: ptmalloc2 | Allocateur glibc |
| 2.1.13.b | ptmalloc arenas: Per-CPU pools | Arenas ptmalloc |
| 2.1.13.c | tcmalloc: Google's allocator | Allocateur Google |
| 2.1.13.d | jemalloc: Facebook's allocator | Allocateur Facebook |
| 2.1.13.e | mimalloc: Microsoft allocator | Allocateur Microsoft |
| 2.1.13.f | Allocator comparison: Performance | Comparaison |
| 2.1.13.g | Choosing allocator: Use case | Choix allocateur |
| 2.1.13.h | Custom malloc: Override symbols | Malloc personnalise |
| 2.1.13.i | LD_PRELOAD: Override allocator | Surcharge LD_PRELOAD |
| 2.1.13.j | Allocator hooks | Hooks allocateur |
| 2.1.13.k | Memory pressure: Low memory handling | Pression memoire |
| 2.1.13.l | Allocator benchmarks | Benchmarks |

### 2.1.14: Specialized Allocators (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.14.a | Memory pools: Fixed-size objects | Pools memoire |
| 2.1.14.b | Pool implementation: Free list in objects | Implementation pool |
| 2.1.14.c | Object pools: Reuse initialized objects | Pools d'objets |
| 2.1.14.d | Arena allocators: Linear allocation | Allocateurs arena |
| 2.1.14.e | Arena bump pointer: Simple increment | Pointeur bump |
| 2.1.14.f | Arena reset: Free all at once | Reset arena |
| 2.1.14.g | Stack allocators: LIFO allocation | Allocateurs pile |
| 2.1.14.h | Ring buffers: Circular allocation | Buffers circulaires |
| 2.1.14.i | Custom allocators: Interface design | Interface custom |
| 2.1.14.j | Allocator traits: Generic interface | Traits allocateur |
| 2.1.14.k | Composable allocators | Allocateurs composables |
| 2.1.14.l | Region-based memory management | Gestion par regions |

### 2.1.15: Garbage Collection Fundamentals (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.15.a | GC motivation: Automatic memory management | Motivation GC |
| 2.1.15.b | Reachability: Object can be accessed | Accessibilite |
| 2.1.15.c | Root set: Global, stack, registers | Ensemble racine |
| 2.1.15.d | Live object: Reachable from roots | Objet vivant |
| 2.1.15.e | Garbage: Unreachable objects | Ordures |
| 2.1.15.f | Collector vs mutator: GC vs application | Collecteur vs mutateur |
| 2.1.15.g | Stop-the-world: Pause application | Arret du monde |
| 2.1.15.h | GC metrics: Pause time, throughput, footprint | Metriques GC |
| 2.1.15.i | Tracing vs reference counting | Tracage vs comptage |
| 2.1.15.j | GC overhead | Surcharge GC |
| 2.1.15.k | GC triggers | Declencheurs GC |
| 2.1.15.l | GC safety points | Points surs |

### 2.1.16: Mark and Sweep (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.16.a | Mark phase: Traverse from roots | Phase de marquage |
| 2.1.16.b | Mark bit: Flag on each object | Bit de marquage |
| 2.1.16.c | Recursive marking: Follow pointers | Marquage recursif |
| 2.1.16.d | Mark stack: Avoid stack overflow | Pile de marquage |
| 2.1.16.e | Sweep phase: Collect unmarked | Phase de balayage |
| 2.1.16.f | Free list: After sweep | Liste libre apres |
| 2.1.16.g | Lazy sweep: Sweep on demand | Balayage paresseux |
| 2.1.16.h | Bitmap marking: External bits | Marquage bitmap |
| 2.1.16.i | Implementation: Complete in C | Implementation C |
| 2.1.16.j | Mark-sweep variants | Variantes |
| 2.1.16.k | Tri-color marking | Marquage tri-couleur |
| 2.1.16.l | Mark-compact | Marquage-compaction |

### 2.1.17: Reference Counting (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.17.a | Ref count: Per-object counter | Compteur par objet |
| 2.1.17.b | Increment: On assignment | Increment |
| 2.1.17.c | Decrement: On overwrite/scope exit | Decrement |
| 2.1.17.d | Zero check: Free when zero | Test zero |
| 2.1.17.e | Cycle problem: A→B→A | Probleme de cycle |
| 2.1.17.f | Cycle detection: Backup tracing | Detection de cycles |
| 2.1.17.g | Weak references: Don't count | References faibles |
| 2.1.17.h | Deferred ref counting: Batch updates | Comptage differe |
| 2.1.17.i | Atomic refcount: Thread-safe | Compteur atomique |
| 2.1.17.j | Swift ARC: Automatic Reference Counting | ARC Swift |
| 2.1.17.k | Python refcounting | Comptage Python |
| 2.1.17.l | Rust ownership: Compile-time | Ownership Rust |

### 2.1.18: Copying Collection (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.18.a | Semi-space: Two equal halves | Semi-espace |
| 2.1.18.b | From-space: Current allocation area | Espace source |
| 2.1.18.c | To-space: Copy destination | Espace destination |
| 2.1.18.d | Copying: Move live objects | Copie |
| 2.1.18.e | Forwarding pointer: Track moved objects | Pointeur de redirection |
| 2.1.18.f | Cheney's algorithm: Breadth-first copy | Algorithme Cheney |
| 2.1.18.g | Compaction effect: No fragmentation | Effet compaction |
| 2.1.18.h | Pointer update: Fix all references | Mise a jour pointeurs |
| 2.1.18.i | Flip: Swap from and to | Echange |
| 2.1.18.j | Downside: 50% memory waste | Inconvenient |
| 2.1.18.k | Copying vs mark-sweep | Comparaison |
| 2.1.18.l | Incremental copying | Copie incrementale |

### 2.1.19: Generational GC (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.19.a | Generational hypothesis: Most objects die young | Hypothese generationnelle |
| 2.1.19.b | Young generation: New objects | Generation jeune |
| 2.1.19.c | Old generation: Survived objects | Generation ancienne |
| 2.1.19.d | Minor GC: Collect young only | GC mineur |
| 2.1.19.e | Major GC: Collect all | GC majeur |
| 2.1.19.f | Promotion: Young to old | Promotion |
| 2.1.19.g | Remembered set: Old→young pointers | Ensemble rappele |
| 2.1.19.h | Write barrier: Track cross-gen pointers | Barriere d'ecriture |
| 2.1.19.i | Nursery: Very young objects | Nursery |
| 2.1.19.j | Card marking: Coarse-grained tracking | Marquage par carte |
| 2.1.19.k | Tenuring threshold | Seuil de tenure |
| 2.1.19.l | Multi-generation | Multi-generations |

### 2.1.20: Concurrent & Incremental GC (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.20.a | STW problem: Long pauses | Probleme STW |
| 2.1.20.b | Incremental: Small steps | Incremental |
| 2.1.20.c | Concurrent: GC with mutator | Concurrent |
| 2.1.20.d | Tri-color: White, grey, black | Tri-couleur |
| 2.1.20.e | Write barrier: Track mutations | Barriere ecriture |
| 2.1.20.f | Read barrier: Track reads | Barriere lecture |
| 2.1.20.g | SATB: Snapshot at the beginning | Snapshot initial |
| 2.1.20.h | G1 GC: Garbage First | G1 GC |
| 2.1.20.i | ZGC: Colored pointers | ZGC |
| 2.1.20.j | Concurrent mark: Mark with mutator | Marquage concurrent |
| 2.1.20.k | Shenandoah: Red Hat low-pause | Shenandoah |
| 2.1.20.l | Pause time goals | Objectifs de pause |

### 2.1.21: Conservative GC (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.21.a | Problem: C has no type info at runtime | Probleme types |
| 2.1.21.b | Conservative: Treat potential pointers as pointers | Approche conservative |
| 2.1.21.c | False positives: Integer looks like pointer | Faux positifs |
| 2.1.21.d | Boehm GC: Popular conservative GC | GC Boehm |
| 2.1.21.e | Blacklisting: Avoid certain addresses | Liste noire |
| 2.1.21.f | Interior pointers: Point to middle of object | Pointeurs interieurs |
| 2.1.21.g | Limitations: No moving collection | Limitations |
| 2.1.21.h | Usage: Drop-in for C/C++ | Utilisation |
| 2.1.21.i | Precise vs conservative | Precis vs conservatif |
| 2.1.21.j | Type information recovery | Recuperation types |
| 2.1.21.k | Conservative roots only | Racines conservatives |
| 2.1.21.l | Semi-conservative | Semi-conservatif |

### 2.1.22: Memory Profiling (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.1.22.a | Valgrind memcheck: Leak detection | Valgrind memcheck |
| 2.1.22.b | memcheck errors: Invalid read/write | Erreurs memcheck |
| 2.1.22.c | Definitely lost: No pointer to start | Perdu definitivement |
| 2.1.22.d | Indirectly lost: Lost via other lost | Perdu indirectement |
| 2.1.22.e | Possibly lost: Interior pointer | Possiblement perdu |
| 2.1.22.f | Still reachable: Pointer exists at exit | Encore accessible |
| 2.1.22.g | Valgrind massif: Heap profiler | Massif Valgrind |
| 2.1.22.h | massif-visualizer: Graphical view | Visualiseur massif |
| 2.1.22.i | heaptrack: KDE profiler | Heaptrack KDE |
| 2.1.22.j | AddressSanitizer: Runtime detection | AddressSanitizer |
| 2.1.22.k | LeakSanitizer: Integrated with ASan | LeakSanitizer |
| 2.1.22.l | gperftools heap profiler: Google's tool | gperftools Google |
| 2.1.22.m | LC 146 LRU Cache: Medium | Exercice LeetCode LRU |
| 2.1.22.n | LC 460 LFU Cache: Hard | Exercice LeetCode LFU |
| 2.1.22.o | Design memory allocator: Custom | Projet custom |

---

## Contexte

Ce projet integre tous les concepts de gestion memoire du Module 2.1 dans un systeme complet. Vous implementerez un allocateur de memoire sophistique avec support du multithreading, un garbage collector conservatif pour C, et un profiler de memoire integre.

---

## Enonce

### Partie 1: Allocateur Multi-Niveau

Implementez un allocateur avec:
- Segregated free lists pour les petites allocations
- Buddy system pour les allocations moyennes
- mmap direct pour les grandes allocations
- Per-thread caches pour la performance

### Partie 2: Garbage Collector Conservatif

Implementez un GC de type Boehm:
- Scanning conservatif des racines
- Mark-and-sweep collection
- Support des pointeurs interieurs
- Integration transparente avec malloc/free

### Partie 3: Memory Profiler

Implementez un profiler avec:
- Detection de fuites memoire
- Detection de buffer overflows (canaries)
- Detection de use-after-free (poison values)
- Statistiques d'allocation detaillees

---

## API Principale

```c
// Allocateur
void *mem_alloc(size_t size);
void *mem_realloc(void *ptr, size_t size);
void mem_free(void *ptr);
void *mem_aligned_alloc(size_t alignment, size_t size);

// GC
void gc_init(void);
void *gc_alloc(size_t size);
void gc_collect(void);
void gc_shutdown(void);

// Profiler
void profiler_start(void);
void profiler_stop(void);
void profiler_report(FILE *out);
int profiler_check_leaks(void);
```

---

## Criteres d'Evaluation

| Critere | Points |
|---------|--------|
| Allocateur segregated lists | 20 |
| Allocateur buddy system | 15 |
| Thread safety | 15 |
| Garbage collector | 20 |
| Profiler et debug features | 20 |
| Documentation et tests | 10 |
| **Total** | **100** |
