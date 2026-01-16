# Exercice 1.9.2-a : there_is_no_best_algorithm

**Module :**
1.9.2 — Capstone: Algorithms Review

**Concept :**
a — Comprehensive algorithms synthesis (Sorting, Searching, Graphs, DP, Greedy, Complexity)

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
complet

**Tiers :**
3 — Synthèse (révision complète algorithmes Phase 1)

**Langage :**
Rust Edition 2024

**Prérequis :**
- Algorithmes de tri (merge, quick, heap sort)
- Binary search et variantes
- Graph algorithms (BFS, DFS, Dijkstra)
- Programmation dynamique de base
- Algorithmes gloutons

**Domaines :**
Algo, Struct, MD

**Durée estimée :**
90 min

**XP Base :**
200

**Complexité :**
T5 O(n log n) à O(n²) × S3 O(n)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**

| Catégorie | Fichiers |
|-----------|----------|
| Sorting | `sorting.rs` (5 algorithmes) |
| Searching | `searching.rs` (binary search + variantes) |
| Graphs | `graphs.rs` (BFS, DFS, Dijkstra, Bellman-Ford) |
| Analysis | `analysis.rs` (complexity analyzer) |

**Fonctions autorisées :**
- Rust : `std::collections::*`, `std::cmp::*`

**Fonctions interdites :**
- `.sort()`, `.sort_unstable()` (tu dois les implémenter!)

---

### 1.2 Consigne

#### 🎬 Section Culture : "There Is No Best Algorithm"

**🕶️ THE MATRIX — "Do not try to find the best algorithm. That's impossible. Instead, only try to realize the truth... there is no best algorithm."**

Tu connais la scène du gosse qui plie la cuillère ? Il dit à Neo : "Il n'y a pas de cuillère." La cuillère n'existe que dans l'esprit.

En algorithmique, c'est pareil. **Il n'y a pas d'algorithme "meilleur"** dans l'absolu. Il n'y a que :
- Le bon algorithme **pour le bon problème**
- Le bon algorithme **pour les bonnes contraintes**
- Le bon algorithme **pour le bon contexte**

Exemples :

| Problème | Input | Meilleur Algo | Pourquoi |
|----------|-------|---------------|----------|
| Trier un array | n = 10 | **Insertion Sort** | O(n²) mais rapide sur petit n |
| Trier un array | n = 10⁶ | **Quick Sort** | O(n log n) moyen, cache-friendly |
| Trier un array | n = 10⁶, déjà presque trié | **Tim Sort** | O(n) dans le meilleur cas |
| Trier des entiers | n = 10⁶, range [0, 1000] | **Counting Sort** | O(n + k), bat O(n log n) |

**La vérité ?** Il n'y a pas d'algorithme universel. **Tu dois choisir.**

Comme Neo qui apprend à voir la Matrix pour ce qu'elle est (du code), tu vas apprendre à voir les algorithmes pour ce qu'ils sont : des **outils** avec des **trade-offs**.

*"What are you trying to tell me? That I can dodge O(n²)?"*
*"No, Neo. I'm trying to tell you that when you're ready... you won't have to. You'll choose O(n log n) or O(n) according to your constraints."*

---

#### 🎓 Section Académique : Énoncé Formel

**Ta mission :**

Implémenter et comparer **11 catégories d'algorithmes** fondamentaux :

**1. Sorting Showdown (5 algorithmes)**
```rust
pub fn merge_sort<T: Ord + Clone>(arr: &mut [T]);
pub fn quick_sort<T: Ord>(arr: &mut [T]);
pub fn heap_sort<T: Ord>(arr: &mut [T]);
pub fn insertion_sort<T: Ord>(arr: &mut [T]);
pub fn counting_sort(arr: &mut [u32], max_val: u32);
```

**2. Binary Search Variants (3 variantes)**
```rust
pub fn binary_search<T: Ord>(arr: &[T], target: &T) -> Result<usize, usize>;
pub fn lower_bound<T: Ord>(arr: &[T], target: &T) -> usize;  // Premier >=
pub fn upper_bound<T: Ord>(arr: &[T], target: &T) -> usize;  // Premier >
```

**3. Graph Traversals (2 parcours)**
```rust
pub fn bfs(graph: &Graph, start: usize) -> Vec<usize>;  // Ordre BFS
pub fn dfs(graph: &Graph, start: usize) -> Vec<usize>;  // Ordre DFS
```

**4. Shortest Paths (3 algorithmes)**
```rust
pub fn dijkstra(graph: &Graph, start: usize) -> Vec<u64>;  // Weights positifs
pub fn bellman_ford(graph: &Graph, start: usize) -> Result<Vec<i64>, NegativeCycle>;
pub fn floyd_warshall(graph: &Graph) -> Vec<Vec<i64>>;  // All-pairs
```

**5. MST (2 algorithmes)**
```rust
pub fn kruskal(graph: &Graph) -> u64;  // Utilise DSU
pub fn prim(graph: &Graph, start: usize) -> u64;  // Utilise Heap
```

**6. DP Patterns Recognition**
```rust
pub fn identify_dp_pattern(problem: &str) -> DPPattern;
// Knapsack, LIS, LCS, Edit Distance, etc.
```

**7. Greedy Proofs**
```rust
pub fn prove_greedy_optimal(algorithm: GreedyAlgo) -> Proof;
```

**8-11. Analysis, Selection, Speed (voir bonus)**

**Sortie :**
- Tous les algorithmes implémentés correctement
- Benchmark comparatif montrant les trade-offs
- Sélection automatique du meilleur algo selon contraintes

**Contraintes :**
- Implémenter from scratch (pas de `.sort()`)
- Complexité respectée pour chaque algo
- Tests sur edge cases (empty, single element, sorted, reverse sorted)

**Exemples :**

| Algorithme | Input | Output | Complexité |
|------------|-------|--------|------------|
| `merge_sort([3,1,2])` | `[3,1,2]` | `[1,2,3]` | O(n log n) |
| `binary_search([1,3,5,7], &5)` | Array trié | `Ok(2)` | O(log n) |
| `dijkstra(graph, 0)` | Graph | `[0, 1, 3, 4]` | O(E log V) |

---

### 1.3 Prototype

```rust
// Sorting
pub fn merge_sort<T: Ord + Clone>(arr: &mut [T]);
pub fn quick_sort<T: Ord>(arr: &mut [T]);
pub fn heap_sort<T: Ord>(arr: &mut [T]);
pub fn insertion_sort<T: Ord>(arr: &mut [T]);
pub fn counting_sort(arr: &mut [u32], max_val: u32);

// Searching
pub fn binary_search<T: Ord>(arr: &[T], target: &T) -> Result<usize, usize>;
pub fn lower_bound<T: Ord>(arr: &[T], target: &T) -> usize;
pub fn upper_bound<T: Ord>(arr: &[T], target: &T) -> usize;

// Graphs
pub struct Graph {
    pub adj: Vec<Vec<(usize, u64)>>,  // (neighbor, weight)
}

pub fn bfs(graph: &Graph, start: usize) -> Vec<usize>;
pub fn dfs(graph: &Graph, start: usize) -> Vec<usize>;
pub fn dijkstra(graph: &Graph, start: usize) -> Vec<u64>;
pub fn bellman_ford(graph: &Graph, start: usize) -> Result<Vec<i64>, NegativeCycle>;
pub fn kruskal(graph: &Graph) -> u64;
pub fn prim(graph: &Graph, start: usize) -> u64;
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Anecdote Historique

**Le Bug du QuickSort de Java (2006) — Le Pire Cas Provoqué**

En 2006, des chercheurs ont découvert qu'on pouvait **forcer le QuickSort de Java dans son pire cas** O(n²) en construisant un input adversarial.

La stratégie de pivot de Java était prévisible (median-of-3). En construisant un array spécifique, on pouvait forcer QuickSort à toujours choisir le pire pivot.

**Résultat :** Un array de 100,000 éléments prenait **30 secondes** à trier au lieu de 0.01s.

**Fix de Java (JDK 7) :** Passer à **Dual-Pivot QuickSort** avec randomisation, impossible à "casser".

**Leçon :** Aucun algorithme n'est parfait. QuickSort est O(n log n) **en moyenne**, mais O(n²) **au pire**. Il faut connaître les limites.

---

### 2.2 Fun Fact

**Pourquoi Python utilise TimSort (2002) ?**

Tim Peters (développeur Python) a créé **TimSort** — un hybride de Merge Sort et Insertion Sort — spécifiquement pour Python.

**Pourquoi ?** Parce que **la plupart des données réelles sont partiellement triées**.

Exemples :
- Logs de serveur → souvent triés par timestamp
- Données de capteurs → tendances monotones
- Résultats de DB → déjà triés par index

TimSort détecte les **runs** (séquences déjà triées) et les fusionne intelligemment.

**Performance :**
- Pire cas : O(n log n) (comme Merge Sort)
- Meilleur cas : **O(n)** (si déjà trié)
- Cas moyen : Plus rapide que QuickSort sur données réelles

Aujourd'hui utilisé par : **Python, Java, Android, Swift**

---

## SECTION 2.5 : DANS LA VRAIE VIE

### Backend Engineer chez Netflix

**Cas d'usage : Dijkstra pour Content Delivery**

Netflix utilise Dijkstra pour router les vidéos du CDN (Content Delivery Network) le plus proche vers l'utilisateur.

```rust
struct CDN {
    servers: Vec<Server>,
    latencies: Graph,  // Latence entre serveurs
}

impl CDN {
    fn best_server_for_user(&self, user_location: usize) -> usize {
        let distances = dijkstra(&self.latencies, user_location);
        distances.iter()
            .enumerate()
            .min_by_key(|(_, &dist)| dist)
            .map(|(idx, _)| idx)
            .unwrap()
    }
}
```

**Complexité :** O(E log V) avec binary heap, O(E + V log V) avec Fibonacci heap

**Résultat :** Vidéo commence en <1s au lieu de 5-10s

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ cargo test
   Compiling there_is_no_best_algorithm v0.1.0
     Running unittests src/lib.rs

running 25 tests
test sorting::test_merge_sort ... ok
test sorting::test_quick_sort ... ok
test sorting::test_heap_sort ... ok
test sorting::test_insertion_sort_small ... ok
test sorting::test_counting_sort ... ok
test searching::test_binary_search ... ok
test searching::test_lower_bound ... ok
test searching::test_upper_bound ... ok
test graphs::test_bfs ... ok
test graphs::test_dfs ... ok
test graphs::test_dijkstra ... ok
test graphs::test_bellman_ford ... ok
test graphs::test_negative_cycle ... ok
test graphs::test_kruskal ... ok
test graphs::test_prim ... ok
test analysis::test_complexity ... ok
test integration::test_algorithm_selection ... ok

test result: ok. 25 passed; 0 failed

$ cargo bench
Benchmark merge_sort/10k    time: [1.234 ms]
Benchmark quick_sort/10k    time: [892.3 µs]  ← Plus rapide !
Benchmark heap_sort/10k     time: [1.456 ms]
```

---

## 🔥 SECTION 3.1 : BONUS AVANCÉ

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×3

**Domaines Bonus :**
`Algo, MD, Probas`

### 3.1.1 Consigne Bonus

**🕶️ BONUS : "The Algorithm Architect"**

Implémenter un **système expert** qui :

1. **Analyse un problème** (description textuelle)
2. **Identifie le pattern** (Knapsack, Shortest Path, MST, etc.)
3. **Sélectionne le meilleur algorithme** selon les contraintes
4. **Génère le code** (template de solution)

**Exemple :**
```rust
let problem = Problem {
    description: "Find shortest path in weighted graph with negative edges",
    constraints: Constraints {
        n: 1000,
        m: 5000,
        has_negative_weights: true,
        needs_all_pairs: false,
    }
};

let solution = select_algorithm(&problem);
assert_eq!(solution.algorithm, Algorithm::BellmanFord);
assert_eq!(solution.complexity, "O(VE)");
assert_eq!(solution.reason, "Negative weights → can't use Dijkstra");
```

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| Test | Algorithme | Input | Expected | Points |
|------|------------|-------|----------|--------|
| T01 | merge_sort | `[3,1,2]` | `[1,2,3]` | 10 |
| T02 | quick_sort | `[5,2,8,1]` | `[1,2,5,8]` | 10 |
| T03 | binary_search | Sorted array | `Ok(index)` | 10 |
| T04 | dijkstra | Graph with 5 nodes | Correct distances | 15 |
| T05 | bellman_ford | Graph with negative edge | Correct distances | 15 |
| T06 | kruskal | Complete graph K5 | Correct MST weight | 10 |

### 4.3 Solution de référence (extraits)

```rust
// Merge Sort
pub fn merge_sort<T: Ord + Clone>(arr: &mut [T]) {
    let n = arr.len();
    if n <= 1 { return; }

    let mid = n / 2;
    merge_sort(&mut arr[..mid]);
    merge_sort(&mut arr[mid..]);

    let mut temp = Vec::with_capacity(n);
    let (left, right) = arr.split_at(mid);

    let (mut i, mut j) = (0, 0);
    while i < left.len() && j < right.len() {
        if left[i] <= right[j] {
            temp.push(left[i].clone());
            i += 1;
        } else {
            temp.push(right[j].clone());
            j += 1;
        }
    }
    temp.extend_from_slice(&left[i..]);
    temp.extend_from_slice(&right[j..]);

    arr.clone_from_slice(&temp);
}

// Dijkstra
pub fn dijkstra(graph: &Graph, start: usize) -> Vec<u64> {
    let n = graph.adj.len();
    let mut dist = vec![u64::MAX; n];
    let mut heap = BinaryHeap::new();

    dist[start] = 0;
    heap.push(Reverse((0, start)));

    while let Some(Reverse((d, u))) = heap.pop() {
        if d > dist[u] { continue; }

        for &(v, w) in &graph.adj[u] {
            if dist[u] + w < dist[v] {
                dist[v] = dist[u] + w;
                heap.push(Reverse((dist[v], v)));
            }
        }
    }

    dist
}
```

### 4.10 Solutions Mutantes

```rust
// Mutant A (Boundary): Merge sort avec indices incorrects
pub fn mutant_merge_sort_boundary<T: Ord + Clone>(arr: &mut [T]) {
    let mid = arr.len() / 2 + 1;  // ❌ Off by one
    // ...
}

// Mutant B (Safety): Dijkstra sans check d'overflow
pub fn mutant_dijkstra_overflow(graph: &Graph, start: usize) -> Vec<u64> {
    // ❌ dist[u] + w peut overflow si u64::MAX
    let new_dist = dist[u] + w;  // Pas de checked_add
}

// Mutant C (Logic): Binary search avec condition inversée
pub fn mutant_binary_search_logic<T: Ord>(arr: &[T], target: &T) -> Option<usize> {
    let mid = (lo + hi) / 2;
    if arr[mid] > *target {  // ❌ Devrait être <
        lo = mid + 1;  // Logique inversée
    }
}
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

**Tableau comparatif des algorithmes de tri :**

| Algorithme | Meilleur | Moyen | Pire | Espace | Stable | Quand l'utiliser |
|------------|----------|-------|------|--------|--------|------------------|
| **Merge** | O(n log n) | O(n log n) | O(n log n) | O(n) | ✅ | Toujours safe, garanties |
| **Quick** | O(n log n) | O(n log n) | O(n²) | O(log n) | ❌ | Cache-friendly, moyen |
| **Heap** | O(n log n) | O(n log n) | O(n log n) | O(1) | ❌ | In-place, garanti |
| **Insertion** | O(n) | O(n²) | O(n²) | O(1) | ✅ | Petit n ou presque trié |
| **Counting** | O(n+k) | O(n+k) | O(n+k) | O(k) | ✅ | Range petit, entiers |

**Tableau comparatif des algos de graphes :**

| Algorithme | Complexité | Contraintes | Utilisation |
|------------|------------|-------------|-------------|
| **BFS** | O(V + E) | — | Shortest path unweighted |
| **DFS** | O(V + E) | — | Cycle detection, topological sort |
| **Dijkstra** | O(E log V) | Weights ≥ 0 | Shortest path weighted |
| **Bellman-Ford** | O(VE) | Détecte cycles négatifs | Shortest path avec weights négatifs |
| **Floyd-Warshall** | O(V³) | All-pairs | Distances entre toutes paires |
| **Kruskal** | O(E log E) | — | MST, utilise DSU |
| **Prim** | O(E log V) | — | MST, utilise Heap |

---

### 5.2 LDA (extrait Dijkstra)

```
FONCTION dijkstra QUI RETOURNE VECTEUR D'ENTIERS ET PREND PARAMÈTRES graph ET start
DÉBUT FONCTION
    DÉCLARER dist COMME VECTEUR D'ENTIERS INITIALISÉ À INFINI
    DÉCLARER heap COMME TAS BINAIRE MINIMUM

    AFFECTER 0 À dist[start]
    INSÉRER (0, start) DANS heap

    TANT QUE heap N'EST PAS VIDE FAIRE
        EXTRAIRE (d, u) DU MINIMUM DE heap

        SI d EST SUPÉRIEUR À dist[u] ALORS
            CONTINUER AU PROCHAIN TOUR
        FIN SI

        POUR CHAQUE VOISIN (v, w) DE u FAIRE
            SI dist[u] PLUS w EST INFÉRIEUR À dist[v] ALORS
                AFFECTER dist[u] PLUS w À dist[v]
                INSÉRER (dist[v], v) DANS heap
            FIN SI
        FIN POUR
    FIN TANT QUE

    RETOURNER dist
FIN FONCTION
```

---

### 5.8 Mnémotechniques

#### 🕶️ MEME : "There is no spoon" — Il n'y a pas d'algorithme parfait

![Matrix spoon](https://i.imgflip.com/2/26hg2.jpg)

Comme le gosse qui dit "Il n'y a pas de cuillère", tu dois réaliser : **Il n'y a pas d'algorithme parfait**.

- QuickSort ? O(n²) au pire
- MergeSort ? O(n) espace
- HeapSort ? Pas stable

**La vérité :** Tous les algorithmes ont des trade-offs. Le "meilleur" dépend du contexte.

---

#### 🎯 MEME : "Choose your fighter" — Sélection d'algorithme

```
┌─────────────┬─────────────┬─────────────┐
│  QUICK      │   MERGE     │    HEAP     │
│  ⚡Fast      │   🛡️Safe    │   💾Inplace │
│  O(n log n) │  O(n log n) │  O(n log n) │
│  Unstable   │   Stable    │   Unstable  │
│  O(log n)   │   O(n) mem  │   O(1) mem  │
└─────────────┴─────────────┴─────────────┘
```

Comme dans Street Fighter où tu choisis ton combattant, tu choisis ton algorithme selon :
- **Speed** → QuickSort
- **Safety** (garanties) → MergeSort
- **Memory** (in-place) → HeapSort

---

## ⚠️ SECTION 6 : PIÈGES

1. **QuickSort au pire** — O(n²) si pivot toujours le pire
2. **Dijkstra avec weights négatifs** — Résultats incorrects
3. **Binary search sur array non trié** — Ne trouve pas l'élément
4. **Overflow dans dist[u] + w** — Utilise `checked_add()`
5. **Confondre BFS et Dijkstra** — BFS pour unweighted seulement

---

## 📝 SECTION 7 : QCM

**Question 1:** Quel algorithme choisir pour trier 10 millions d'entiers dans [0, 255] ?

A) QuickSort
B) MergeSort
C) Counting Sort
D) HeapSort

**Réponse:** C (Counting Sort en O(n + 256) bat O(n log n))

---

**Question 2:** Dijkstra fonctionne avec des poids négatifs ?

A) Oui, toujours
B) Non, jamais
C) Oui, si pas de cycle négatif
D) Seulement avec Fibonacci heap

**Réponse:** B (Dijkstra assume weights ≥ 0)

---

## 📊 SECTION 8 : RÉCAPITULATIF

**Concepts (11) :**

| # | Concept | Maîtrisé ? |
|---|---------|-----------|
| a | Sorting showdown | ☐ |
| b | Binary search variants | ☐ |
| c | Graph traversals | ☐ |
| d | Shortest paths | ☐ |
| e | MST algorithms | ☐ |
| f | DP patterns | ☐ |
| g | Greedy proofs | ☐ |
| h | Complexity analysis | ☐ |
| i | Algorithm selection | ☐ |
| j | Speed implementation | ☐ |
| k | Trade-offs understanding | ☐ |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "exercise_slug": "1.9.2-a-there-is-no-best-algorithm",
    "metadata": {
      "exercise_id": "1.9.2-a",
      "module": "1.9.2",
      "difficulty": 5,
      "xp_base": 200,
      "bonus_icon": "🔥",
      "meme_reference": "THE MATRIX - There is no spoon"
    }
  }
}
```

---

**FIN DE L'EXERCICE 1.9.2-a**
