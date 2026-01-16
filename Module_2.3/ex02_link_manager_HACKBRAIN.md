# Exercice 2.3.4-a : multiverse_link_manager

**Module :**
2.3.4 — Hard Links & Symbolic Links

**Concept :**
a-l — Gestion complète des liens Unix (hard links, symlinks, dangling, loops)

**Difficulté :**
★★★★★☆☆☆☆☆ (5/10)

**Type :**
code

**Tiers :**
3 — Synthèse (tous concepts 2.3.4.a → 2.3.4.l)

**Langage :**
C (C17)

**Prérequis :**
- 2.3.1 (stat/lstat, inodes)
- 2.3.3 (opendir/readdir/closedir)
- Manipulation de chemins
- Gestion d'erreurs errno

**Domaines :**
FS, Mem, Struct

**Durée estimée :**
300 min (5h)

**XP Base :**
500

**Complexité :**
T3 O(n) × S2 O(SYMLOOP_MAX)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex02/
├── multiverse_links.h      # Header avec structures et prototypes
├── multiverse_links.c      # Implémentation principale
├── paradox_detector.c      # Détection de boucles et dangling
└── Makefile
```

**Fonctions autorisées :**
```c
// Syscalls liens
link, symlink, unlink, readlink

// Syscalls fichiers
stat, lstat, fstat, open, close, read, write

// Répertoires
opendir, readdir, closedir

// Mémoire
malloc, free, calloc, realloc

// Chaînes
strlen, strcpy, strncpy, strcmp, strcat, strdup, snprintf

// Chemins
realpath, dirname, basename

// Erreurs
strerror, perror, errno

// Affichage
printf, fprintf
```

**Fonctions interdites :**
```c
access      // Utiliser stat() à la place
system      // Pas de shell
```

### 1.2 Consigne

#### 🎬 CONTEXTE FUN — Spider-Man: Into the Spider-Verse

**"Anyone can wear the mask."** — Miles Morales

Dans le **Spider-Verse**, le Kingpin a créé un **collisionneur de particules** capable d'ouvrir des portails vers d'autres dimensions. Le problème ? Certains portails mènent vers des **dimensions effondrées** (dangling), d'autres créent des **boucles paradoxales** où l'on revient à son point de départ (symlink loops).

Tu es **Miles Morales**, et ton araignée-sens te permet de **tracer les connexions** entre les dimensions. Tu dois créer un système pour :
- **Ancrer** des Spider-People à la même identité (hard links = même ADN araignée)
- **Ouvrir des portails** vers d'autres dimensions (symlinks = chemins vers ailleurs)
- **Détecter les dimensions mortes** (dangling symlinks)
- **Repérer les paradoxes temporels** (boucles de symlinks)

**Le collisionneur a une limite de stabilité** : après **40 traversées** (SYMLOOP_MAX), il explose. Tu dois détecter les boucles AVANT d'atteindre cette limite !

---

#### 1.2.2 Énoncé Académique

**Ta mission :**

Implémenter un **gestionnaire de liens Unix** complet capable de :

1. **Créer des hard links** avec vérification des restrictions (même filesystem, pas de répertoires)
2. **Créer des liens symboliques** avec support des chemins relatifs/absolus
3. **Analyser les liens** pour déterminer leur type et leurs propriétés
4. **Détecter les liens symboliques morts** (dangling symlinks)
5. **Détecter les boucles de symlinks** avec limite SYMLOOP_MAX
6. **Supprimer des liens** en toute sécurité avec affichage du compteur

**Entrée :**
- Chemins de fichiers/liens à manipuler
- Options de configuration (récursif, verbose, dry-run)

**Sortie :**
- Codes d'erreur appropriés (succès, erreurs spécifiques)
- Structures d'information sur les liens
- Listes de liens problématiques

**Contraintes :**
- Hard links : même filesystem uniquement, pas sur répertoires
- Symlinks : peuvent traverser filesystems, peuvent pointer vers répertoires
- Limite de profondeur : SYMLOOP_MAX (40) pour détection de boucles
- `readlink()` ne termine PAS par `\0` — ajouter manuellement
- Toujours utiliser `lstat()` pour examiner un lien sans le suivre

### 1.3 Prototype

```c
#ifndef MULTIVERSE_LINKS_H
#define MULTIVERSE_LINKS_H

#include <sys/types.h>
#include <sys/stat.h>
#include <stdint.h>
#include <stddef.h>
#include <limits.h>

/*============================================================================
 * SPIDER-VERSE CONSTANTS
 *============================================================================*/

/* Limite de stabilité du collisionneur (2.3.4.l) */
#ifndef COLLIDER_STABILITY_LIMIT
#define COLLIDER_STABILITY_LIMIT 40  /* = SYMLOOP_MAX */
#endif

/*============================================================================
 * TYPES — Classification des Variants
 *============================================================================*/

/* Type de connexion dimensionnelle */
typedef enum {
    VARIANT_UNKNOWN    = 0,  /* Type inconnu */
    VARIANT_ORIGINAL   = 1,  /* Fichier normal (link_count == 1) */
    VARIANT_ANCHORED   = 2,  /* Hard link (link_count > 1) - 2.3.4.a */
    VARIANT_PORTAL     = 3   /* Lien symbolique - 2.3.4.f */
} variant_type_t;

/* État de la dimension cible */
typedef enum {
    DIMENSION_STABLE    = 0,  /* La cible existe */
    DIMENSION_COLLAPSED = 1,  /* Dangling symlink - 2.3.4.k */
    DIMENSION_PARADOX   = 2,  /* Boucle détectée - 2.3.4.l */
    DIMENSION_ERROR     = 3   /* Erreur d'accès */
} dimension_status_t;

/* Codes d'erreur du multiverse */
typedef enum {
    MULTIVERSE_OK              =  0,
    MULTIVERSE_NOT_FOUND       = -1,  /* ENOENT */
    MULTIVERSE_NO_ACCESS       = -2,  /* EACCES */
    MULTIVERSE_NO_MEMORY       = -3,  /* ENOMEM */
    MULTIVERSE_CROSS_DIMENSION = -4,  /* EXDEV - Hard link cross-fs (2.3.4.e) */
    MULTIVERSE_IS_NEXUS        = -5,  /* Hard link sur répertoire (2.3.4.e) */
    MULTIVERSE_ALREADY_EXISTS  = -6,  /* EEXIST */
    MULTIVERSE_PARADOX         = -7,  /* Boucle détectée (2.3.4.l) */
    MULTIVERSE_PATH_TOO_LONG   = -8,  /* ENAMETOOLONG */
    MULTIVERSE_INVALID_PARAM   = -9,  /* Paramètre invalide */
    MULTIVERSE_IO_ERROR        = -10  /* Erreur I/O générale */
} multiverse_error_t;

/*============================================================================
 * STRUCTURES — Données du Spider-Verse
 *============================================================================*/

/* Information sur un variant (fichier/lien) */
typedef struct {
    char               *path;            /* Chemin du variant */
    variant_type_t      type;            /* ORIGINAL, ANCHORED, PORTAL */
    ino_t               spider_dna;      /* Numéro d'inode - 2.3.4.a */
    nlink_t             variant_count;   /* Compteur de liens - 2.3.4.c */
    dev_t               universe_id;     /* Device ID (pour vérif même fs) */

    /* Spécifique aux portails (symlinks) */
    char               *portal_target;   /* Cible du portail - 2.3.4.j */
    char               *resolved_path;   /* Chemin résolu (absolu) */
    dimension_status_t  dimension_status;/* STABLE, COLLAPSED, PARADOX */
    int                 traversal_depth; /* Profondeur de résolution */
} variant_info_t;

/* Noeud de liste chaînée pour résultats */
typedef struct variant_node {
    variant_info_t         *info;
    struct variant_node    *next;
} variant_node_t;

/* Liste de variants */
typedef struct {
    variant_node_t  *head;
    variant_node_t  *tail;
    size_t           count;
} variant_list_t;

/* Options de scan pour find_collapsed */
typedef struct {
    int     recursive;       /* Parcours récursif */
    int     follow_mounts;   /* Suivre les points de montage */
    size_t  max_depth;       /* Profondeur max (0 = illimitée) */
} scan_options_t;

/* Options de suppression sécurisée */
typedef struct {
    int     dry_run;         /* Simulation sans suppression */
    int     warn_last_link;  /* Avertir si dernier lien */
    int     verbose;         /* Afficher les détails */
} unlink_options_t;

/*============================================================================
 * API — Création de Connexions
 *============================================================================*/

/**
 * spider_anchor - Crée un hard link (même ADN araignée)
 *
 * Comme Miles et Peter Parker partagent le même pouvoir araignée,
 * deux hard links partagent le même inode (2.3.4.a).
 *
 * @param existing  Chemin du variant existant (source)
 * @param new_anchor Chemin du nouveau hard link à créer
 * @return MULTIVERSE_OK en cas de succès, code d'erreur sinon
 *
 * Utilise link() (2.3.4.b). Le compteur de variants augmente (2.3.4.c).
 * RESTRICTIONS (2.3.4.e): même univers (filesystem), pas de nexus (répertoire)
 */
multiverse_error_t spider_anchor(const char *existing, const char *new_anchor);

/**
 * open_portal - Crée un lien symbolique (portail dimensionnel)
 *
 * Comme les portails du collisionneur, un symlink est un fichier
 * contenant le CHEMIN vers une autre dimension (2.3.4.f).
 *
 * @param destination Dimension cible (chemin stocké dans le portail)
 * @param portal_path  Emplacement du portail à créer
 * @return MULTIVERSE_OK en cas de succès, code d'erreur sinon
 *
 * Utilise symlink() (2.3.4.g). La cible peut ne pas exister (dimension instable).
 */
multiverse_error_t open_portal(const char *destination, const char *portal_path);

/*============================================================================
 * API — Analyse des Connexions
 *============================================================================*/

/**
 * spider_sense - Analyse un variant et retourne ses propriétés
 *
 * L'araignée-sens de Miles lui permet de "voir" les connexions.
 *
 * @param path   Chemin du variant à analyser
 * @param follow Si non-zero, traverse le portail (stat - 2.3.4.h)
 *               Si zero, examine le portail lui-même (lstat - 2.3.4.i)
 * @return Structure allouée avec les infos, NULL si erreur
 *
 * Pour les portails, lit la cible avec readlink() (2.3.4.j).
 * Doit être libérée avec variant_info_free().
 */
variant_info_t *spider_sense(const char *path, int follow);

/**
 * variant_info_free - Libère une structure variant_info_t
 */
void variant_info_free(variant_info_t *info);

/**
 * same_spider_dna - Vérifie si deux chemins ont le même inode
 *
 * @return 1 si même inode (hard links), 0 sinon, -1 si erreur
 */
int same_spider_dna(const char *path1, const char *path2);

/*============================================================================
 * API — Détection de Problèmes
 *============================================================================*/

/**
 * is_dimension_collapsed - Vérifie si un portail mène à une dimension morte
 *
 * Un "dangling symlink" (2.3.4.k) est comme un portail vers une
 * dimension qui s'est effondrée. lstat() réussit, mais stat() échoue.
 *
 * @param path Chemin du portail à vérifier
 * @return 1 si collapsed (dangling), 0 si stable, -1 si pas un portail
 */
int is_dimension_collapsed(const char *path);

/**
 * find_collapsed_dimensions - Trouve tous les portails morts dans un répertoire
 *
 * Scanne un univers (répertoire) pour trouver les portails instables.
 *
 * @param universe_path Chemin du répertoire à scanner
 * @param options       Options de scan (récursif, profondeur)
 * @return Liste des portails collapsed, NULL si aucun ou erreur
 */
variant_list_t *find_collapsed_dimensions(const char *universe_path,
                                          const scan_options_t *options);

/**
 * detect_paradox - Détecte si un chemin contient une boucle de symlinks
 *
 * Comme le collisionneur qui explose après trop de traversées,
 * le kernel impose SYMLOOP_MAX (2.3.4.l).
 *
 * @param path          Chemin à vérifier
 * @param paradox_chain Buffer pour stocker le chemin de la boucle (peut être NULL)
 * @param chain_size    Taille du buffer
 * @return 1 si paradoxe détecté, 0 sinon, -1 si erreur
 */
int detect_paradox(const char *path, char *paradox_chain, size_t chain_size);

/**
 * resolve_dimension - Résout un chemin en suivant tous les portails
 *
 * @param path       Chemin à résoudre
 * @param resolved   Buffer pour le chemin résolu
 * @param size       Taille du buffer
 * @param max_depth  Profondeur max (0 = COLLIDER_STABILITY_LIMIT)
 * @return 0 si succès, -1 si paradoxe ou erreur
 */
int resolve_dimension(const char *path, char *resolved,
                      size_t size, int max_depth);

/*============================================================================
 * API — Suppression Sécurisée
 *============================================================================*/

/**
 * close_portal_safely - Supprime un lien avec vérification du compteur
 *
 * Comme fermer proprement un portail dimensionnel. Affiche le compteur
 * de variants (2.3.4.c) avant suppression. unlink() décrémente (2.3.4.d).
 *
 * @param path    Chemin du lien à supprimer
 * @param options Options (dry_run, verbose, warn_last_link)
 * @return MULTIVERSE_OK si succès, code d'erreur sinon
 */
multiverse_error_t close_portal_safely(const char *path,
                                       const unlink_options_t *options);

/*============================================================================
 * UTILITAIRES
 *============================================================================*/

void variant_list_free(variant_list_t *list);
const char *multiverse_strerror(multiverse_error_t error);
const char *variant_type_string(variant_type_t type);
const char *dimension_status_string(dimension_status_t status);
void print_variant_info(const variant_info_t *info);

#endif /* MULTIVERSE_LINKS_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Hard Links : L'ADN Partagé des Spider-People

Dans le Spider-Verse, tous les Spider-People partagent le **même ADN araignée**. Ils sont différentes personnes (Miles, Peter, Gwen) mais ont la **même essence**.

C'est exactement ce qu'est un **hard link** : deux noms de fichiers différents qui pointent vers le **même inode**. Modifier le contenu via l'un modifie le contenu pour l'autre — c'est le même fichier !

```
Miles Morales ──┐
                ├──→ [Spider DNA] ←── Même inode !
Peter Parker ───┘
```

### 2.2 Symbolic Links : Les Portails Dimensionnels

Le **collisionneur de Kingpin** ouvre des portails vers d'autres dimensions. Un portail n'EST PAS la destination — c'est juste une PORTE avec une adresse écrite dessus.

Un **symlink** est pareil : c'est un fichier dont le CONTENU est un chemin. Quand tu le traverses (`stat()`), le système lit ce chemin et y va automatiquement.

```
/home/miles/shortcut.txt ──→ Fichier contenant: "/data/spider-verse/database.txt"
         │
         └──→ Quand tu ouvres, tu atterris sur database.txt
```

### 2.3 Dangling Symlinks : Les Dimensions Effondrées

Que se passe-t-il si un portail pointe vers une dimension qui **n'existe plus** ? Dans le film, ça causerait une catastrophe. En informatique, on appelle ça un **dangling symlink**.

- `lstat()` réussit (le portail existe)
- `stat()` échoue avec `ENOENT` (la cible n'existe pas)

### 2.4 Boucles de Symlinks : Les Paradoxes Temporels

Si le portail A mène au portail B, qui mène au portail C, qui mène... au portail A ? **Boucle infinie !**

Le kernel se protège avec `SYMLOOP_MAX` (typiquement 40). Après 40 traversées, il abandonne avec `ELOOP`.

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation des Liens |
|--------|----------------------|
| **SysAdmin** | Gestion des versions avec symlinks (`/usr/bin/python` → `python3.11`) |
| **DevOps** | Déploiement blue-green avec symlinks (`current` → `release-v2.1`) |
| **Package Manager** | Hard links pour déduplication (Nix, pnpm) |
| **Backup Engineer** | Hard links pour snapshots incrémentaux (rsync --link-dest) |
| **Security Analyst** | Détection de symlink attacks (race conditions) |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
multiverse_links.c  paradox_detector.c  multiverse_links.h  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -std=c17 -c multiverse_links.c
gcc -Wall -Wextra -Werror -std=c17 -c paradox_detector.c
ar rcs libmultiverse.a multiverse_links.o paradox_detector.o

$ gcc -Wall -Wextra -Werror main.c -L. -lmultiverse -o test

$ ./test
=== Test 1: Hard Link (Spider Anchor) ===
Creating anchor: spider_anchor("/tmp/peter.txt", "/tmp/miles.txt")
Result: MULTIVERSE_OK
Peter's DNA (inode): 12345678
Miles's DNA (inode): 12345678
Same Spider DNA? YES!
Variant count: 2

=== Test 2: Symbolic Link (Portal) ===
Opening portal: open_portal("/etc/passwd", "/tmp/secret_portal")
Result: MULTIVERSE_OK
Portal target: /etc/passwd
Dimension status: STABLE

=== Test 3: Dangling Symlink (Collapsed Dimension) ===
Opening portal: open_portal("/nonexistent/dimension", "/tmp/broken_portal")
Result: MULTIVERSE_OK
is_dimension_collapsed: YES (target doesn't exist)

=== Test 4: Symlink Loop (Paradox) ===
Creating paradox: A → B → C → A
detect_paradox("/tmp/loop_a"): PARADOX DETECTED!
Loop chain: /tmp/loop_a -> /tmp/loop_c -> /tmp/loop_b -> /tmp/loop_a

=== Test 5: Safe Unlink ===
close_portal_safely("/tmp/miles.txt", verbose=1)
Variant count: 2 -> 1 (data preserved, other anchor exists)
Result: MULTIVERSE_OK

All tests passed!
```

---

## ⚡ SECTION 3.1 : BONUS STANDARD (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★★☆☆☆ (7/10)

**Récompense :**
XP ×2

**Time Complexity attendue :**
O(n × d) où n = fichiers, d = profondeur

**Space Complexity attendue :**
O(SYMLOOP_MAX) pour détection de boucles

### 3.1.1 Consigne Bonus

**🎬 Le Collisionneur Amélioré de Kingpin**

Kingpin veut améliorer son collisionneur. Il te demande d'ajouter :

1. **Détection optimisée des boucles** : Au lieu de simplement compter jusqu'à 40, maintenir un **ensemble de chemins visités** pour identifier le POINT EXACT de la boucle.

2. **Statistiques du multiverse** : Compter les types de liens dans un répertoire (combien de hard links, symlinks, dangling, etc.)

3. **Réparation automatique** : Fonction qui supprime tous les dangling symlinks d'un répertoire.

**Prototype Bonus :**

```c
/* Détection de boucle avec chemin exact */
typedef struct {
    char    **visited_paths;   /* Chemins visités */
    int       count;           /* Nombre de chemins */
    int       loop_start;      /* Index où la boucle commence */
    char     *loop_entry;      /* Premier chemin de la boucle */
} paradox_trace_t;

int detect_paradox_detailed(const char *path, paradox_trace_t *trace);
void paradox_trace_free(paradox_trace_t *trace);

/* Statistiques */
typedef struct {
    size_t  total_files;
    size_t  regular_files;
    size_t  hard_links;        /* Fichiers avec nlink > 1 */
    size_t  symlinks;
    size_t  dangling_symlinks;
    size_t  directories;
} multiverse_stats_t;

multiverse_error_t scan_multiverse_stats(const char *path,
                                         const scan_options_t *options,
                                         multiverse_stats_t *stats);

/* Réparation */
size_t repair_collapsed_dimensions(const char *path,
                                   const scan_options_t *options,
                                   int dry_run);
```

### 3.1.2 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Détection boucle | Compteur simple | Ensemble de chemins visités |
| Information | Type de boucle | Chemin exact de la boucle |
| Scan | Liste des dangling | Statistiques complètes |
| Actions | Détection seule | Réparation automatique |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette (25 tests)

| # | Test | Entrée | Sortie Attendue | Concept |
|---|------|--------|-----------------|---------|
| 01 | Hard link création | `spider_anchor("/tmp/a.txt", "/tmp/b.txt")` | `MULTIVERSE_OK`, nlink=2 | 2.3.4.b,c |
| 02 | Même inode | Après test 01 | `same_spider_dna() == 1` | 2.3.4.a |
| 03 | Hard link cross-device | `/etc/passwd` → `/tmp/x` | `MULTIVERSE_CROSS_DIMENSION` | 2.3.4.e |
| 04 | Hard link sur répertoire | `/tmp` → `/tmp/x` | `MULTIVERSE_IS_NEXUS` | 2.3.4.e |
| 05 | Symlink création | `open_portal("/etc/passwd", "/tmp/p")` | `MULTIVERSE_OK` | 2.3.4.g |
| 06 | Symlink contient chemin | Après test 05 | `readlink == "/etc/passwd"` | 2.3.4.f |
| 07 | stat() suit symlink | `spider_sense("/tmp/p", 1)` | inode de /etc/passwd | 2.3.4.h |
| 08 | lstat() ne suit pas | `spider_sense("/tmp/p", 0)` | inode du symlink | 2.3.4.i |
| 09 | readlink() cible | Après test 05 | `portal_target == "/etc/passwd"` | 2.3.4.j |
| 10 | Dangling symlink | `open_portal("/x", "/tmp/d")` | `dimension_status == COLLAPSED` | 2.3.4.k |
| 11 | is_dimension_collapsed | Après test 10 | `== 1` | 2.3.4.k |
| 12 | Valid symlink not dangling | Symlink vers fichier existant | `is_dimension_collapsed == 0` | 2.3.4.k |
| 13 | Boucle simple A↔B | `A→B, B→A` | `detect_paradox == 1` | 2.3.4.l |
| 14 | Boucle triple A→B→C→A | 3 symlinks circulaires | `detect_paradox == 1` | 2.3.4.l |
| 15 | Chaîne longue sans boucle | 30 symlinks en chaîne | `detect_paradox == 0` | 2.3.4.l |
| 16 | Chaîne > SYMLOOP_MAX | 50 symlinks | Erreur ou boucle détectée | 2.3.4.l |
| 17 | unlink décrémente | Supprimer 1 des 3 hard links | nlink: 3 → 2 | 2.3.4.d |
| 18 | NULL param hardlink | `spider_anchor(NULL, "/tmp/x")` | `MULTIVERSE_INVALID_PARAM` | Robustesse |
| 19 | NULL param symlink | `open_portal(NULL, "/tmp/x")` | `MULTIVERSE_INVALID_PARAM` | Robustesse |
| 20 | Chemin vide | `spider_anchor("", "/tmp/x")` | `MULTIVERSE_INVALID_PARAM` | Robustesse |
| 21 | Destination existe | Créer lien où fichier existe | `MULTIVERSE_ALREADY_EXISTS` | Erreur |
| 22 | Source inexistante | Hard link vers fichier absent | `MULTIVERSE_NOT_FOUND` | Erreur |
| 23 | find_collapsed_dimensions | Répertoire avec 3 dangling | Liste de 3 éléments | 2.3.4.k |
| 24 | Scan récursif | Répertoire avec sous-dossiers | Trouve tous les dangling | Récursion |
| 25 | Mémoire (Valgrind) | Cycle complet create/analyze/free | 0 leaks | Sécurité |

### 4.2 main.c de test

```c
#include "multiverse_links.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

static int tests_passed = 0;
static int tests_failed = 0;

#define TEST(name, condition) do { \
    if (condition) { \
        printf("[OK] %s\n", name); \
        tests_passed++; \
    } else { \
        printf("[FAIL] %s\n", name); \
        tests_failed++; \
    } \
} while(0)

static void setup_test_file(const char *path, const char *content)
{
    FILE *f = fopen(path, "w");
    if (f) {
        fprintf(f, "%s", content);
        fclose(f);
    }
}

static void cleanup(const char *path)
{
    unlink(path);
}

int main(void)
{
    printf("=== MULTIVERSE LINKS TEST SUITE ===\n\n");

    /* Setup */
    setup_test_file("/tmp/mv_original.txt", "Spider-Verse Data\n");

    /* Test 1: Hard link creation (2.3.4.b) */
    multiverse_error_t err = spider_anchor("/tmp/mv_original.txt",
                                           "/tmp/mv_anchor.txt");
    TEST("Hard link creation", err == MULTIVERSE_OK);

    /* Test 2: Same inode (2.3.4.a) */
    TEST("Same spider DNA", same_spider_dna("/tmp/mv_original.txt",
                                            "/tmp/mv_anchor.txt") == 1);

    /* Test 3: Link count incremented (2.3.4.c) */
    variant_info_t *info = spider_sense("/tmp/mv_original.txt", 0);
    TEST("Variant count == 2", info && info->variant_count == 2);
    variant_info_free(info);

    /* Test 4: Hard link on directory (2.3.4.e) */
    err = spider_anchor("/tmp", "/tmp/mv_dir_link");
    TEST("Hard link on directory blocked", err == MULTIVERSE_IS_NEXUS);

    /* Test 5: Symlink creation (2.3.4.g) */
    err = open_portal("/etc/passwd", "/tmp/mv_portal");
    TEST("Portal creation", err == MULTIVERSE_OK);

    /* Test 6: Symlink is path container (2.3.4.f) */
    info = spider_sense("/tmp/mv_portal", 0);
    TEST("Portal contains target path",
         info && info->portal_target &&
         strcmp(info->portal_target, "/etc/passwd") == 0);
    variant_info_free(info);

    /* Test 7: stat follows symlink (2.3.4.h) */
    info = spider_sense("/tmp/mv_portal", 1);  /* follow = 1 */
    TEST("Following portal gets target inode",
         info && info->type == VARIANT_ORIGINAL);
    variant_info_free(info);

    /* Test 8: lstat doesn't follow (2.3.4.i) */
    info = spider_sense("/tmp/mv_portal", 0);  /* follow = 0 */
    TEST("Not following shows portal type",
         info && info->type == VARIANT_PORTAL);
    variant_info_free(info);

    /* Test 9: Dangling symlink (2.3.4.k) */
    err = open_portal("/nonexistent/dimension", "/tmp/mv_collapsed");
    TEST("Collapsed dimension creation", err == MULTIVERSE_OK);
    TEST("Collapsed dimension detected",
         is_dimension_collapsed("/tmp/mv_collapsed") == 1);

    /* Test 10: Symlink loop detection (2.3.4.l) */
    symlink("/tmp/mv_loop_b", "/tmp/mv_loop_a");
    symlink("/tmp/mv_loop_a", "/tmp/mv_loop_b");
    char loop_path[PATH_MAX];
    TEST("Paradox detected", detect_paradox("/tmp/mv_loop_a",
                                            loop_path, sizeof(loop_path)) == 1);

    /* Test 11: Link count decrement (2.3.4.d) */
    info = spider_sense("/tmp/mv_original.txt", 0);
    nlink_t before = info ? info->variant_count : 0;
    variant_info_free(info);

    unlink_options_t opts = { .dry_run = 0, .verbose = 0, .warn_last_link = 0 };
    close_portal_safely("/tmp/mv_anchor.txt", &opts);

    info = spider_sense("/tmp/mv_original.txt", 0);
    nlink_t after = info ? info->variant_count : 0;
    variant_info_free(info);
    TEST("Variant count decremented", after == before - 1);

    /* Test 12: NULL parameters */
    TEST("NULL param returns error",
         spider_anchor(NULL, "/tmp/x") == MULTIVERSE_INVALID_PARAM);
    TEST("NULL param for portal",
         open_portal(NULL, "/tmp/x") == MULTIVERSE_INVALID_PARAM);

    /* Cleanup */
    cleanup("/tmp/mv_original.txt");
    cleanup("/tmp/mv_portal");
    cleanup("/tmp/mv_collapsed");
    cleanup("/tmp/mv_loop_a");
    cleanup("/tmp/mv_loop_b");

    printf("\n=== RESULTS: %d passed, %d failed ===\n",
           tests_passed, tests_failed);
    return tests_failed > 0 ? 1 : 0;
}
```

### 4.3 Solution de référence

```c
/* multiverse_links.c - Solution de référence */
#include "multiverse_links.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <dirent.h>
#include <libgen.h>

/*============================================================================
 * CRÉATION DE CONNEXIONS
 *============================================================================*/

multiverse_error_t spider_anchor(const char *existing, const char *new_anchor)
{
    struct stat sb_src;
    struct stat sb_dst_parent;
    char *dst_copy;
    char *parent_dir;

    /* Validation des paramètres */
    if (existing == NULL || new_anchor == NULL)
        return (MULTIVERSE_INVALID_PARAM);
    if (existing[0] == '\0' || new_anchor[0] == '\0')
        return (MULTIVERSE_INVALID_PARAM);

    /* Vérifier que la source existe */
    if (stat(existing, &sb_src) == -1)
    {
        if (errno == ENOENT)
            return (MULTIVERSE_NOT_FOUND);
        return (MULTIVERSE_IO_ERROR);
    }

    /* Restriction 2.3.4.e : pas de hard link sur répertoire */
    if (S_ISDIR(sb_src.st_mode))
        return (MULTIVERSE_IS_NEXUS);

    /* Vérifier même filesystem (2.3.4.e) */
    dst_copy = strdup(new_anchor);
    if (dst_copy == NULL)
        return (MULTIVERSE_NO_MEMORY);
    parent_dir = dirname(dst_copy);

    if (stat(parent_dir, &sb_dst_parent) == -1)
    {
        free(dst_copy);
        return (MULTIVERSE_NOT_FOUND);
    }
    free(dst_copy);

    if (sb_src.st_dev != sb_dst_parent.st_dev)
        return (MULTIVERSE_CROSS_DIMENSION);

    /* Créer le hard link (2.3.4.b) */
    if (link(existing, new_anchor) == -1)
    {
        if (errno == EEXIST)
            return (MULTIVERSE_ALREADY_EXISTS);
        if (errno == EACCES || errno == EPERM)
            return (MULTIVERSE_NO_ACCESS);
        if (errno == EXDEV)
            return (MULTIVERSE_CROSS_DIMENSION);
        return (MULTIVERSE_IO_ERROR);
    }

    return (MULTIVERSE_OK);
}

multiverse_error_t open_portal(const char *destination, const char *portal_path)
{
    /* Validation des paramètres */
    if (destination == NULL || portal_path == NULL)
        return (MULTIVERSE_INVALID_PARAM);
    if (destination[0] == '\0' || portal_path[0] == '\0')
        return (MULTIVERSE_INVALID_PARAM);

    /* Créer le symlink (2.3.4.g) */
    if (symlink(destination, portal_path) == -1)
    {
        if (errno == EEXIST)
            return (MULTIVERSE_ALREADY_EXISTS);
        if (errno == EACCES || errno == EPERM)
            return (MULTIVERSE_NO_ACCESS);
        if (errno == ENAMETOOLONG)
            return (MULTIVERSE_PATH_TOO_LONG);
        return (MULTIVERSE_IO_ERROR);
    }

    return (MULTIVERSE_OK);
}

/*============================================================================
 * ANALYSE DES CONNEXIONS
 *============================================================================*/

variant_info_t *spider_sense(const char *path, int follow)
{
    variant_info_t *info;
    struct stat sb;
    char target_buf[PATH_MAX];
    ssize_t len;

    if (path == NULL)
        return (NULL);

    info = calloc(1, sizeof(variant_info_t));
    if (info == NULL)
        return (NULL);

    /* Utiliser lstat pour ne pas suivre (2.3.4.i) ou stat pour suivre (2.3.4.h) */
    if (follow)
    {
        if (stat(path, &sb) == -1)
        {
            free(info);
            return (NULL);
        }
    }
    else
    {
        if (lstat(path, &sb) == -1)
        {
            free(info);
            return (NULL);
        }
    }

    info->path = strdup(path);
    info->spider_dna = sb.st_ino;
    info->variant_count = sb.st_nlink;
    info->universe_id = sb.st_dev;

    /* Déterminer le type */
    if (S_ISLNK(sb.st_mode))
    {
        info->type = VARIANT_PORTAL;

        /* Lire la cible du symlink (2.3.4.j) */
        len = readlink(path, target_buf, sizeof(target_buf) - 1);
        if (len > 0)
        {
            target_buf[len] = '\0';  /* CRUCIAL: readlink ne termine pas ! */
            info->portal_target = strdup(target_buf);
        }

        /* Vérifier l'état de la dimension cible */
        struct stat target_sb;
        if (stat(path, &target_sb) == -1)
        {
            if (errno == ENOENT)
                info->dimension_status = DIMENSION_COLLAPSED;  /* 2.3.4.k */
            else if (errno == ELOOP)
                info->dimension_status = DIMENSION_PARADOX;    /* 2.3.4.l */
            else
                info->dimension_status = DIMENSION_ERROR;
        }
        else
        {
            info->dimension_status = DIMENSION_STABLE;
        }
    }
    else if (sb.st_nlink > 1)
    {
        info->type = VARIANT_ANCHORED;  /* Hard link (2.3.4.a) */
    }
    else
    {
        info->type = VARIANT_ORIGINAL;
    }

    return (info);
}

void variant_info_free(variant_info_t *info)
{
    if (info == NULL)
        return;
    free(info->path);
    free(info->portal_target);
    free(info->resolved_path);
    free(info);
}

int same_spider_dna(const char *path1, const char *path2)
{
    struct stat sb1;
    struct stat sb2;

    if (path1 == NULL || path2 == NULL)
        return (-1);

    if (stat(path1, &sb1) == -1 || stat(path2, &sb2) == -1)
        return (-1);

    /* Même inode ET même device = même fichier (2.3.4.a) */
    return (sb1.st_ino == sb2.st_ino && sb1.st_dev == sb2.st_dev);
}

/*============================================================================
 * DÉTECTION DE PROBLÈMES
 *============================================================================*/

int is_dimension_collapsed(const char *path)
{
    struct stat sb_link;
    struct stat sb_target;

    if (path == NULL)
        return (-1);

    /* Vérifier que c'est un symlink (2.3.4.i) */
    if (lstat(path, &sb_link) == -1)
        return (-1);
    if (!S_ISLNK(sb_link.st_mode))
        return (-1);  /* Pas un symlink */

    /* Essayer de suivre le symlink (2.3.4.h) */
    if (stat(path, &sb_target) == -1)
    {
        if (errno == ENOENT)
            return (1);  /* Dangling! (2.3.4.k) */
    }

    return (0);  /* Cible existe */
}

int detect_paradox(const char *path, char *paradox_chain, size_t chain_size)
{
    char current[PATH_MAX];
    char target[PATH_MAX];
    char *visited[COLLIDER_STABILITY_LIMIT];
    int depth;
    ssize_t len;
    struct stat sb;
    int found_loop;
    int i;

    if (path == NULL)
        return (-1);

    strncpy(current, path, sizeof(current) - 1);
    current[sizeof(current) - 1] = '\0';
    depth = 0;
    found_loop = 0;

    /* Initialiser tableau de chemins visités */
    for (i = 0; i < COLLIDER_STABILITY_LIMIT; i++)
        visited[i] = NULL;

    while (depth < COLLIDER_STABILITY_LIMIT)
    {
        /* Vérifier si c'est un symlink */
        if (lstat(current, &sb) == -1)
            break;
        if (!S_ISLNK(sb.st_mode))
            break;  /* Fin de chaîne, pas une boucle */

        /* Lire la cible (2.3.4.j) */
        len = readlink(current, target, sizeof(target) - 1);
        if (len == -1)
            break;
        target[len] = '\0';

        /* Résoudre le chemin relatif si nécessaire */
        if (target[0] != '/')
        {
            char *dir_copy = strdup(current);
            char *dir = dirname(dir_copy);
            char resolved[PATH_MAX];
            snprintf(resolved, sizeof(resolved), "%s/%s", dir, target);
            free(dir_copy);
            strncpy(target, resolved, sizeof(target) - 1);
        }

        /* Vérifier si déjà visité (détection de boucle) */
        for (i = 0; i < depth; i++)
        {
            if (visited[i] && strcmp(visited[i], target) == 0)
            {
                found_loop = 1;
                break;
            }
        }

        if (found_loop)
            break;

        /* Ajouter au tableau des visités */
        visited[depth] = strdup(current);
        depth++;

        /* Avancer au prochain */
        strncpy(current, target, sizeof(current) - 1);
    }

    /* Construire la chaîne de paradoxe si demandé */
    if (paradox_chain && chain_size > 0 && (found_loop || depth >= COLLIDER_STABILITY_LIMIT))
    {
        paradox_chain[0] = '\0';
        for (i = 0; i < depth && i < COLLIDER_STABILITY_LIMIT; i++)
        {
            if (visited[i])
            {
                if (i > 0)
                    strncat(paradox_chain, " -> ", chain_size - strlen(paradox_chain) - 1);
                strncat(paradox_chain, visited[i], chain_size - strlen(paradox_chain) - 1);
            }
        }
    }

    /* Libérer mémoire */
    for (i = 0; i < COLLIDER_STABILITY_LIMIT; i++)
        free(visited[i]);

    return (found_loop || depth >= COLLIDER_STABILITY_LIMIT);
}

/*============================================================================
 * SUPPRESSION SÉCURISÉE
 *============================================================================*/

multiverse_error_t close_portal_safely(const char *path, const unlink_options_t *options)
{
    struct stat sb;
    int is_verbose;
    int warn_last;

    if (path == NULL)
        return (MULTIVERSE_INVALID_PARAM);

    is_verbose = options ? options->verbose : 0;
    warn_last = options ? options->warn_last_link : 0;

    /* Obtenir infos avant suppression */
    if (lstat(path, &sb) == -1)
        return (MULTIVERSE_NOT_FOUND);

    if (is_verbose)
        printf("Variant count: %lu", (unsigned long)sb.st_nlink);

    if (warn_last && sb.st_nlink == 1 && S_ISREG(sb.st_mode))
        printf(" WARNING: Last link! Data will be lost.");

    /* Mode simulation */
    if (options && options->dry_run)
    {
        if (is_verbose)
            printf(" [DRY RUN - not deleted]\n");
        return (MULTIVERSE_OK);
    }

    /* Suppression (2.3.4.d) */
    if (unlink(path) == -1)
    {
        if (errno == EACCES || errno == EPERM)
            return (MULTIVERSE_NO_ACCESS);
        return (MULTIVERSE_IO_ERROR);
    }

    if (is_verbose)
        printf(" -> %lu (data %s)\n",
               (unsigned long)(sb.st_nlink - 1),
               sb.st_nlink > 1 ? "preserved" : "freed");

    return (MULTIVERSE_OK);
}

/*============================================================================
 * UTILITAIRES
 *============================================================================*/

const char *multiverse_strerror(multiverse_error_t error)
{
    switch (error)
    {
        case MULTIVERSE_OK:              return "Success";
        case MULTIVERSE_NOT_FOUND:       return "Not found (dimension doesn't exist)";
        case MULTIVERSE_NO_ACCESS:       return "Access denied";
        case MULTIVERSE_NO_MEMORY:       return "Out of memory";
        case MULTIVERSE_CROSS_DIMENSION: return "Cannot anchor across dimensions (EXDEV)";
        case MULTIVERSE_IS_NEXUS:        return "Cannot anchor to nexus (directory)";
        case MULTIVERSE_ALREADY_EXISTS:  return "Portal already exists";
        case MULTIVERSE_PARADOX:         return "Paradox detected (symlink loop)";
        case MULTIVERSE_PATH_TOO_LONG:   return "Path too long";
        case MULTIVERSE_INVALID_PARAM:   return "Invalid parameter";
        case MULTIVERSE_IO_ERROR:        return "I/O error";
        default:                         return "Unknown error";
    }
}

const char *variant_type_string(variant_type_t type)
{
    switch (type)
    {
        case VARIANT_ORIGINAL:  return "Original (Regular)";
        case VARIANT_ANCHORED:  return "Anchored (Hard Link)";
        case VARIANT_PORTAL:    return "Portal (Symbolic Link)";
        default:                return "Unknown";
    }
}

const char *dimension_status_string(dimension_status_t status)
{
    switch (status)
    {
        case DIMENSION_STABLE:    return "Stable (Exists)";
        case DIMENSION_COLLAPSED: return "Collapsed (Dangling)";
        case DIMENSION_PARADOX:   return "Paradox (Loop)";
        case DIMENSION_ERROR:     return "Error";
        default:                  return "Unknown";
    }
}
```

### 4.4 Solutions alternatives acceptées

```c
/* Alternative 1: Détection de boucle avec realpath() */
int detect_paradox_alt(const char *path, char *paradox_chain, size_t chain_size)
{
    char resolved[PATH_MAX];

    if (path == NULL)
        return (-1);

    /* realpath() suit les symlinks et détecte ELOOP */
    if (realpath(path, resolved) == NULL)
    {
        if (errno == ELOOP)
        {
            if (paradox_chain)
                strncpy(paradox_chain, path, chain_size);
            return (1);  /* Boucle détectée */
        }
        return (-1);
    }
    return (0);  /* Pas de boucle */
}

/* Alternative 2: Utilisation de hash table pour chemins visités */
/* Plus efficace O(1) lookup vs O(n) */
```

### 4.5 Solutions refusées (avec explications)

```c
/* REFUSÉ 1: readlink sans terminaison \0 */
int bad_read_target(const char *path, char *target, size_t size)
{
    /* BUG CRITIQUE: readlink ne met pas de \0 ! */
    readlink(path, target, size);  /* Buffer non terminé! */
    return (0);
}
/* Pourquoi: Buffer overflow potentiel, comportement indéfini */

/* REFUSÉ 2: Pas de limite sur détection de boucle */
int infinite_detect_paradox(const char *path)
{
    char current[PATH_MAX];
    char target[PATH_MAX];

    strcpy(current, path);
    /* BOUCLE INFINIE si symlinks circulaires! */
    while (1)
    {
        if (lstat(current, &sb) == -1) break;
        if (!S_ISLNK(sb.st_mode)) return (0);
        readlink(current, target, sizeof(target));
        strcpy(current, target);
    }
    return (0);
}
/* Pourquoi: Sans limite SYMLOOP_MAX, boucle infinie */

/* REFUSÉ 3: stat() au lieu de lstat() pour détecter symlinks */
int bad_is_symlink(const char *path)
{
    struct stat sb;
    stat(path, &sb);  /* SUIT le symlink! */
    return S_ISLNK(sb.st_mode);  /* Toujours faux! */
}
/* Pourquoi: stat() suit les symlinks, utiliser lstat() */

/* REFUSÉ 4: Hard link sans vérifier même filesystem */
int bad_hardlink(const char *src, const char *dst)
{
    /* Pas de vérification st_dev! */
    return link(src, dst);  /* Échoue avec EXDEV sans explication */
}
/* Pourquoi: Doit vérifier st_dev pour donner erreur claire */
```

### 4.10 Solutions Mutantes (6 mutants)

```c
/* ============================================================
 * MUTANT A (Safety) : Pas de vérification NULL
 * ============================================================ */
multiverse_error_t mutant_a_spider_anchor(const char *existing, const char *new_anchor)
{
    struct stat sb_src;
    /* BUG: Pas de vérification NULL! */
    if (stat(existing, &sb_src) == -1)  /* CRASH si existing == NULL */
        return (MULTIVERSE_NOT_FOUND);
    /* ... reste du code ... */
    return link(existing, new_anchor) == 0 ? MULTIVERSE_OK : MULTIVERSE_IO_ERROR;
}
/* Comportement: Segfault si existing ou new_anchor est NULL
 * Misconception: "stat() gère NULL" - FAUX, déréférencement avant appel */

/* ============================================================
 * MUTANT B (Resource) : readlink sans terminaison \0
 * ============================================================ */
variant_info_t *mutant_b_spider_sense(const char *path, int follow)
{
    variant_info_t *info = calloc(1, sizeof(variant_info_t));
    struct stat sb;
    char target_buf[PATH_MAX];

    lstat(path, &sb);
    if (S_ISLNK(sb.st_mode))
    {
        /* BUG CRITIQUE: readlink ne termine pas par \0 ! */
        ssize_t len = readlink(path, target_buf, sizeof(target_buf));
        /* Oubli de: target_buf[len] = '\0'; */
        info->portal_target = strdup(target_buf);  /* Buffer overflow! */
    }
    return (info);
}
/* Comportement: Buffer contient garbage après le chemin lu
 * Misconception: "readlink fonctionne comme read() sur chaîne" - FAUX */

/* ============================================================
 * MUTANT C (Logic) : stat() au lieu de lstat()
 * ============================================================ */
int mutant_c_is_dimension_collapsed(const char *path)
{
    struct stat sb;
    /* BUG: stat() suit le symlink! */
    if (stat(path, &sb) == -1)  /* DEVRAIT être lstat() */
    {
        if (errno == ENOENT)
            return (1);
    }
    /* Pour un symlink existant vers cible existante:
     * stat() retourne info de la CIBLE, pas du lien
     * On ne peut jamais détecter que c'est un symlink! */
    return S_ISLNK(sb.st_mode) ? 1 : 0;  /* Toujours 0! */
}
/* Comportement: Ne détecte jamais un symlink car stat() suit
 * Misconception: "stat() et lstat() sont interchangeables" - FAUX pour symlinks */

/* ============================================================
 * MUTANT D (Boundary) : Pas de limite SYMLOOP_MAX
 * ============================================================ */
int mutant_d_detect_paradox(const char *path, char *chain, size_t size)
{
    char current[PATH_MAX];
    char target[PATH_MAX];
    struct stat sb;

    strncpy(current, path, sizeof(current));
    /* BUG: Boucle sans limite! */
    while (1)  /* Devrait être: while (depth < COLLIDER_STABILITY_LIMIT) */
    {
        if (lstat(current, &sb) == -1)
            return (0);
        if (!S_ISLNK(sb.st_mode))
            return (0);
        ssize_t len = readlink(current, target, sizeof(target) - 1);
        target[len] = '\0';
        strncpy(current, target, sizeof(current));
        /* Pas de compteur de profondeur! */
    }
    return (0);  /* Jamais atteint si boucle de symlinks */
}
/* Comportement: Boucle infinie sur symlinks circulaires
 * Misconception: "Le kernel protège toujours" - FAUX, notre code doit aussi limiter */

/* ============================================================
 * MUTANT E (Logic) : Vérification cross-device inversée
 * ============================================================ */
multiverse_error_t mutant_e_spider_anchor(const char *existing, const char *new_anchor)
{
    struct stat sb_src, sb_dst_parent;

    if (existing == NULL || new_anchor == NULL)
        return (MULTIVERSE_INVALID_PARAM);

    stat(existing, &sb_src);

    char *dst_copy = strdup(new_anchor);
    char *parent = dirname(dst_copy);
    stat(parent, &sb_dst_parent);
    free(dst_copy);

    /* BUG: Condition inversée! */
    if (sb_src.st_dev == sb_dst_parent.st_dev)  /* DEVRAIT être != */
        return (MULTIVERSE_CROSS_DIMENSION);  /* Erreur sur même device! */

    return link(existing, new_anchor) == 0 ? MULTIVERSE_OK : MULTIVERSE_IO_ERROR;
}
/* Comportement: Refuse les hard links sur même FS, accepte cross-FS (impossible)
 * Misconception: Confusion sur la logique de comparaison st_dev */

/* ============================================================
 * MUTANT F (Return) : Mauvais code d'erreur pour répertoire
 * ============================================================ */
multiverse_error_t mutant_f_spider_anchor(const char *existing, const char *new_anchor)
{
    struct stat sb_src;

    if (stat(existing, &sb_src) == -1)
        return (MULTIVERSE_NOT_FOUND);

    /* BUG: Mauvais code d'erreur */
    if (S_ISDIR(sb_src.st_mode))
        return (MULTIVERSE_CROSS_DIMENSION);  /* Devrait être MULTIVERSE_IS_NEXUS */

    return link(existing, new_anchor) == 0 ? MULTIVERSE_OK : MULTIVERSE_IO_ERROR;
}
/* Comportement: Code d'erreur incorrect pour hard link sur répertoire
 * Misconception: Les erreurs EXDEV et EISDIR ont le même sens - FAUX */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

| Concept | Description | Référence |
|---------|-------------|-----------|
| **Hard Link = Même Inode** | Deux noms, un seul fichier | 2.3.4.a |
| **link()** | Crée un nouveau nom pour un inode existant | 2.3.4.b |
| **Compteur de liens** | Incrémenté par link(), décrémenté par unlink() | 2.3.4.c, 2.3.4.d |
| **Restrictions hard links** | Même FS, pas de répertoires | 2.3.4.e |
| **Symlink = Fichier chemin** | Contient le chemin vers la cible | 2.3.4.f |
| **symlink()** | Crée un fichier contenant un chemin | 2.3.4.g |
| **stat() suit** | Traverse automatiquement les symlinks | 2.3.4.h |
| **lstat() ne suit pas** | Examine le symlink lui-même | 2.3.4.i |
| **readlink()** | Lit le contenu (chemin) d'un symlink | 2.3.4.j |
| **Dangling symlink** | Cible n'existe pas | 2.3.4.k |
| **Boucles de symlinks** | Détectées via SYMLOOP_MAX | 2.3.4.l |

### 5.2 LDA — Traduction Littérale

```
FONCTION spider_anchor QUI RETOURNE UNE ERREUR ET PREND EN PARAMÈTRES
    existing QUI EST UN POINTEUR VERS UNE CHAÎNE CONSTANTE ET
    new_anchor QUI EST UN POINTEUR VERS UNE CHAÎNE CONSTANTE
DÉBUT FONCTION
    DÉCLARER sb_src COMME STRUCTURE stat
    DÉCLARER sb_dst_parent COMME STRUCTURE stat

    SI existing EST ÉGAL À NUL OU new_anchor EST ÉGAL À NUL ALORS
        RETOURNER ERREUR PARAMÈTRE INVALIDE
    FIN SI

    SI APPELER stat SUR existing VERS sb_src ÉCHOUE ALORS
        RETOURNER ERREUR NON TROUVÉ
    FIN SI

    SI sb_src EST UN RÉPERTOIRE ALORS
        RETOURNER ERREUR EST UN NEXUS (répertoire interdit)
    FIN SI

    DÉCLARER parent_dir COMME LE RÉPERTOIRE PARENT DE new_anchor
    SI APPELER stat SUR parent_dir VERS sb_dst_parent ÉCHOUE ALORS
        RETOURNER ERREUR NON TROUVÉ
    FIN SI

    SI LE DEVICE DE sb_src EST DIFFÉRENT DU DEVICE DE sb_dst_parent ALORS
        RETOURNER ERREUR CROSS DIMENSION (filesystems différents)
    FIN SI

    SI APPELER link AVEC existing ET new_anchor ÉCHOUE ALORS
        RETOURNER ERREUR I/O
    FIN SI

    RETOURNER SUCCÈS
FIN FONCTION
```

### 5.2.2.1 Logic Flow (Structured English)

```
ALGORITHME : Création de Hard Link (spider_anchor)
---
1. VALIDER les paramètres (non-NULL, non-vides)

2. OBTENIR les infos du fichier source avec stat()
   |-- SI échec : RETOURNER "Source non trouvée"

3. VÉRIFIER les restrictions (2.3.4.e) :
   |
   |-- SI source est un répertoire :
   |     RETOURNER "Hard link sur répertoire interdit"
   |
   |-- OBTENIR le répertoire parent de destination
   |-- SI source.st_dev != parent.st_dev :
   |     RETOURNER "Cross-filesystem interdit"

4. CRÉER le hard link avec link()
   |-- SI échec : mapper errno vers code d'erreur

5. RETOURNER succès
```

### 5.2.3.1 Logique de Garde (Fail Fast)

```
FONCTION : detect_paradox (path, chain, size)
---
INIT depth = 0
INIT visited = tableau[COLLIDER_STABILITY_LIMIT]

BOUCLE TANT QUE depth < COLLIDER_STABILITY_LIMIT :
    |
    |-- VÉRIFIER lstat(current) :
    |     SI échec → SORTIR (fin de chaîne)
    |
    |-- VÉRIFIER si symlink :
    |     SI non → RETOURNER 0 (pas de boucle)
    |
    |-- LIRE cible avec readlink() :
    |     IMPORTANT: Ajouter '\0' manuellement!
    |
    |-- RÉSOUDRE chemin relatif si nécessaire
    |
    |-- CHERCHER dans visited[] :
    |     SI trouvé → RETOURNER 1 (BOUCLE!)
    |
    |-- AJOUTER current à visited[depth]
    |-- depth++
    |-- current = cible

SI depth >= LIMIT :
    RETOURNER 1 (probablement boucle)

RETOURNER 0 (pas de boucle)
```

### 5.3 Visualisation ASCII

#### Hard Links vs Symbolic Links

```
                    HARD LINKS (Même ADN Spider)
                    ============================

    /home/miles/peter.txt ──┐
                            ├──→ [ INODE 12345 ] ──→ [ DATA BLOCKS ]
    /home/miles/miles.txt ──┘           │
                                   link_count = 2

    Même inode, même données, noms différents.
    Supprimer un nom → link_count--
    Données libérées quand link_count = 0


                    SYMBOLIC LINKS (Portails)
                    =========================

    /home/miles/portal.txt ──→ [ INODE 99999 ]
                                     │
                              contenu: "/data/target.txt"
                                     │
                                     ▼
                              [ INODE 11111 ] ──→ [ DATA BLOCKS ]
                              /data/target.txt

    Le symlink a son PROPRE inode (99999).
    Son contenu est le CHEMIN vers la cible.
```

#### Dangling Symlink (Dimension Effondrée)

```
    /tmp/broken_portal ──→ [ INODE 88888 ]
                                │
                         contenu: "/nonexistent/file"
                                │
                                ▼
                              ??? RIEN ???

    lstat("/tmp/broken_portal") → SUCCÈS (le portail existe)
    stat("/tmp/broken_portal")  → ÉCHEC ENOENT (la cible n'existe pas)
```

#### Boucle de Symlinks (Paradoxe)

```
    ┌─────────────────────────────────────────────┐
    │                                             │
    ▼                                             │
/tmp/loop_a ──→ contient: "/tmp/loop_c"           │
                     │                            │
                     ▼                            │
            /tmp/loop_c ──→ contient: "/tmp/loop_b"
                                 │                │
                                 ▼                │
                        /tmp/loop_b ──→ contient: "/tmp/loop_a"
                                             │
                                             └────┘

    Traversée: A → C → B → A → C → B → A → ...
    Après SYMLOOP_MAX (40) traversées → ELOOP
```

### 5.4 Les pièges en détail

#### Piège 1 : readlink() ne termine pas par \0

```c
/* DANGER! */
char buf[PATH_MAX];
readlink("/tmp/link", buf, sizeof(buf));
printf("%s\n", buf);  /* COMPORTEMENT INDÉFINI! */

/* CORRECT */
char buf[PATH_MAX];
ssize_t len = readlink("/tmp/link", buf, sizeof(buf) - 1);
if (len > 0) {
    buf[len] = '\0';  /* CRUCIAL! */
    printf("%s\n", buf);
}
```

#### Piège 2 : stat() vs lstat() pour détecter symlinks

```c
/* FAUX: stat() suit le symlink, ne détecte jamais S_ISLNK */
struct stat sb;
stat("/tmp/symlink", &sb);
if (S_ISLNK(sb.st_mode))  /* TOUJOURS FAUX! */
    printf("C'est un symlink\n");

/* CORRECT: lstat() examine le lien lui-même */
lstat("/tmp/symlink", &sb);
if (S_ISLNK(sb.st_mode))  /* Correct! */
    printf("C'est un symlink\n");
```

#### Piège 3 : Hard link cross-filesystem

```c
/* ÉCHOUE avec EXDEV */
link("/home/user/file.txt", "/tmp/link.txt");  /* /home et /tmp peuvent être différents FS */

/* VÉRIFICATION PRÉALABLE */
struct stat sb_src, sb_dst_parent;
stat("/home/user/file.txt", &sb_src);
stat("/tmp", &sb_dst_parent);
if (sb_src.st_dev != sb_dst_parent.st_dev) {
    fprintf(stderr, "Cannot create hard link across filesystems\n");
    return -1;
}
```

### 5.5 Cours Complet

#### 5.5.1 Hard Links : L'Architecture

Un **hard link** n'est pas une "copie" ni un "raccourci". C'est un **nom supplémentaire** pour un fichier existant.

```
AVANT link():                    APRÈS link("a.txt", "b.txt"):

Répertoire:                     Répertoire:
┌─────────────────┐             ┌─────────────────┐
│ "a.txt" → 12345 │             │ "a.txt" → 12345 │
└─────────────────┘             │ "b.txt" → 12345 │  ← Nouveau nom!
                                └─────────────────┘

Inode 12345:                    Inode 12345:
┌────────────────┐              ┌────────────────┐
│ nlink = 1      │              │ nlink = 2      │  ← Incrémenté!
│ data → blocks  │              │ data → blocks  │
└────────────────┘              └────────────────┘
```

**Pourquoi pas de hard links sur répertoires ?**
Pour éviter les **cycles** dans l'arborescence. Si on pouvait faire `link("/home", "/home/user/home_link")`, on créerait une boucle infinie.

#### 5.5.2 Symbolic Links : L'Indirection

Un **symlink** est un fichier spécial dont le **contenu** est un chemin.

```c
/* Création */
symlink("/etc/passwd", "/tmp/passwd_link");

/* Ce qui est créé */
Fichier: /tmp/passwd_link
Type: S_IFLNK (symlink)
Contenu brut: "/etc/passwd" (la chaîne de caractères)
Taille: 11 octets (strlen("/etc/passwd"))
```

**Avantages sur hard links :**
- Peut traverser les filesystems
- Peut pointer vers des répertoires
- Peut pointer vers des cibles qui n'existent pas (encore)

**Inconvénients :**
- Dangling symlinks possibles
- Boucles possibles
- Overhead (un inode supplémentaire)

#### 5.5.3 Le Compteur de Liens (nlink)

```
Création fichier:   nlink = 1
link():            nlink++
unlink():          nlink--
Quand nlink = 0:   Données et inode libérés
```

**Important :** Pour les répertoires, `nlink` compte différemment :
- Le répertoire lui-même : 1
- Son entrée "." : +1
- Chaque sous-répertoire (via "..") : +1

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ readlink(path, buf, sizeof(buf));  /* Sans terminaison */       │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ ssize_t len = readlink(path, buf, sizeof(buf) - 1);             │
│ if (len > 0)                                                    │
│     buf[len] = '\0';                                            │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ readlink() ne termine PAS la chaîne par \0. Il retourne le      │
│ nombre de bytes lus. Sans terminaison manuelle, le buffer       │
│ contient des données garbage après le chemin.                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ while (is_symlink(path)) {                                      │
│     path = readlink_target(path);  /* Boucle infinie! */        │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ int depth = 0;                                                  │
│ while (is_symlink(path) && depth < SYMLOOP_MAX) {               │
│     path = readlink_target(path);                               │
│     depth++;                                                    │
│ }                                                               │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│ Sans limite, une boucle de symlinks (A→B→A) cause une           │
│ boucle infinie. SYMLOOP_MAX (40) est la limite standard.        │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

**Trace : spider_anchor("/tmp/peter.txt", "/tmp/miles.txt")**

```
┌───────┬────────────────────────────────────────┬──────────────────┬─────────────────────────┐
│ Étape │ Instruction                            │ Valeur           │ Explication             │
├───────┼────────────────────────────────────────┼──────────────────┼─────────────────────────┤
│   1   │ Vérifier existing != NULL              │ VRAI             │ Paramètre valide        │
├───────┼────────────────────────────────────────┼──────────────────┼─────────────────────────┤
│   2   │ stat("/tmp/peter.txt", &sb_src)        │ OK, inode=12345  │ Fichier existe          │
├───────┼────────────────────────────────────────┼──────────────────┼─────────────────────────┤
│   3   │ S_ISDIR(sb_src.st_mode) ?              │ FAUX             │ C'est un fichier        │
├───────┼────────────────────────────────────────┼──────────────────┼─────────────────────────┤
│   4   │ dirname("/tmp/miles.txt")              │ "/tmp"           │ Répertoire parent       │
├───────┼────────────────────────────────────────┼──────────────────┼─────────────────────────┤
│   5   │ stat("/tmp", &sb_dst_parent)           │ OK, dev=0x801    │ Même filesystem         │
├───────┼────────────────────────────────────────┼──────────────────┼─────────────────────────┤
│   6   │ sb_src.st_dev == sb_dst_parent.st_dev? │ VRAI (0x801)     │ Cross-fs OK             │
├───────┼────────────────────────────────────────┼──────────────────┼─────────────────────────┤
│   7   │ link(existing, new_anchor)             │ OK               │ Hard link créé!         │
├───────┼────────────────────────────────────────┼──────────────────┼─────────────────────────┤
│   8   │ RETOURNER MULTIVERSE_OK                │ 0                │ Succès                  │
└───────┴────────────────────────────────────────┴──────────────────┴─────────────────────────┘

État après exécution:
- /tmp/peter.txt : inode=12345, nlink=2
- /tmp/miles.txt : inode=12345, nlink=2  ← Même inode!
```

### 5.8 Mnémotechniques

#### 🕷️ MEME : "Anyone can wear the mask" — Hard Links

Dans Spider-Verse, Miles découvre que N'IMPORTE QUI peut être Spider-Man.
Plusieurs personnes (noms) peuvent avoir le même pouvoir (inode).

```c
/* Miles et Peter = Même Spider-DNA */
link("/home/peter.txt", "/home/miles.txt");
/* Maintenant les deux noms pointent vers le même inode */
/* Comme Miles et Peter partagent le pouvoir araignée */
```

#### 🌀 MEME : "I've been falling for 30 minutes!" — Symlink Loops

Dans Thor: Ragnarok, Loki tombe en boucle. C'est exactement ce qui arrive
quand tu traverses des symlinks circulaires sans limite.

```c
/* A → B → C → A = BOUCLE! */
if (depth >= SYMLOOP_MAX)
    return ELOOP;  /* "I've been following for 40 symlinks!" */
```

#### 💀 MEME : "He's dead, Jim" — Dangling Symlinks

Dans Star Trek, quand quelqu'un est mort, Bones dit "He's dead, Jim".
Un dangling symlink pointe vers une cible qui est "morte" (n'existe plus).

```c
if (stat(symlink_path, &sb) == -1 && errno == ENOENT)
    printf("He's dead, Jim. The target doesn't exist.\n");
```

#### 📖 MEME : "Le nom n'est pas dans l'inode"

Rappel crucial : l'inode ne contient PAS le nom du fichier.
Le nom est dans l'entrée de répertoire, pas dans l'inode.

C'est pourquoi plusieurs noms (hard links) peuvent pointer vers le même inode.

### 5.9 Applications pratiques

| Application | Utilisation des liens |
|-------------|----------------------|
| **Gestion de versions** | `/usr/bin/python` → `python3.11` (symlink) |
| **Déploiement** | `/var/www/current` → `/var/www/releases/v2.1` |
| **Deduplication** | Hard links pour fichiers identiques (économie d'espace) |
| **Snapshots** | Hard links pour backups incrémentaux |
| **Build systems** | Symlinks pour dépendances locales |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Conséquence | Solution |
|---|-------|-------------|----------|
| 1 | readlink() sans \0 | Buffer overflow, garbage | `buf[len] = '\0'` |
| 2 | stat() sur symlink | Ne détecte pas le symlink | Utiliser lstat() |
| 3 | Boucle sans limite | Boucle infinie | Compteur SYMLOOP_MAX |
| 4 | Hard link cross-fs | EXDEV sans explication | Vérifier st_dev |
| 5 | Hard link sur répertoire | Refusé silencieusement | Vérifier S_ISDIR |
| 6 | Oublier de libérer | Fuite mémoire | free() sur toutes les allocs |

---

## 📝 SECTION 7 : QCM (10 questions)

**Q1.** Qu'est-ce qu'un hard link ?
- A) Une copie du fichier
- B) Un raccourci Windows
- C) Un nouveau nom pour le même inode
- D) Un fichier contenant un chemin
- E) Un lien vers un autre filesystem
- F) Une redirection réseau
- G) Un alias shell
- H) Un pointeur vers le répertoire parent
- I) Un fichier vide
- J) Un type de compression

**Réponse : C** — Un hard link est un nouveau nom (entrée de répertoire) pointant vers le même inode (2.3.4.a).

---

**Q2.** Quelle restriction s'applique aux hard links ? (2.3.4.e)
- A) Doivent avoir le même propriétaire
- B) Doivent être sur le même filesystem
- C) Doivent avoir les mêmes permissions
- D) Doivent être dans le même répertoire
- E) Doivent avoir la même extension
- F) Peuvent traverser les filesystems
- G) Peuvent pointer vers des répertoires
- H) Doivent être créés par root
- I) Doivent avoir moins de 255 caractères
- J) Doivent être des fichiers texte

**Réponse : B** — Hard links ne peuvent pas traverser les frontières de filesystem (2.3.4.e).

---

**Q3.** Que fait `readlink()` ? (2.3.4.j)
- A) Suit un symlink et retourne la cible finale
- B) Lit le contenu du fichier cible
- C) Lit le chemin stocké dans le symlink
- D) Crée un nouveau symlink
- E) Supprime un symlink
- F) Vérifie si un fichier est un symlink
- G) Retourne les permissions du symlink
- H) Compte le nombre de symlinks
- I) Résout un chemin relatif
- J) Termine la chaîne par \0

**Réponse : C** — readlink() lit le CONTENU d'un symlink (le chemin stocké), sans le suivre (2.3.4.j).

---

**Q4.** Quelle est la particularité de `readlink()` ?
- A) Il retourne toujours un chemin absolu
- B) Il ajoute automatiquement '\0' à la fin
- C) Il NE termine PAS la chaîne par '\0'
- D) Il suit automatiquement les symlinks
- E) Il ne fonctionne que sur les répertoires
- F) Il modifie le symlink
- G) Il requiert des permissions root
- H) Il crée une copie du fichier
- I) Il est bloquant
- J) Il libère automatiquement la mémoire

**Réponse : C** — readlink() ne termine PAS la chaîne par \0, il faut l'ajouter manuellement (2.3.4.j).

---

**Q5.** Quelle est la différence entre `stat()` et `lstat()` pour un symlink ?
- A) stat() est plus rapide
- B) lstat() nécessite root
- C) stat() suit le symlink, lstat() examine le lien lui-même
- D) lstat() suit le symlink, stat() examine le lien lui-même
- E) Aucune différence pour les symlinks
- F) stat() retourne plus d'informations
- G) lstat() ne fonctionne pas sur les symlinks
- H) stat() modifie le fichier
- I) lstat() est déprécié
- J) stat() crée une copie

**Réponse : C** — stat() suit le symlink et retourne info de la cible (2.3.4.h), lstat() examine le lien lui-même (2.3.4.i).

---

**Q6.** Qu'est-ce qu'un "dangling symlink" ? (2.3.4.k)
- A) Un symlink vers un répertoire
- B) Un symlink vers un autre symlink
- C) Un symlink dont la cible n'existe pas
- D) Un symlink avec des permissions 000
- E) Un symlink vers /dev/null
- F) Un symlink circulaire
- G) Un symlink vers un fichier vide
- H) Un symlink créé par root
- I) Un symlink vers un autre filesystem
- J) Un symlink vers lui-même

**Réponse : C** — Un dangling symlink pointe vers une cible qui n'existe pas (2.3.4.k).

---

**Q7.** Qu'est-ce que SYMLOOP_MAX ? (2.3.4.l)
- A) Le nombre maximum de symlinks dans un répertoire
- B) La longueur maximum d'un chemin symlink
- C) Le nombre maximum de traversées de symlinks autorisées
- D) Le nombre maximum de hard links par inode
- E) La taille maximum d'un fichier symlink
- F) Le nombre de processus pouvant accéder à un symlink
- G) La profondeur maximum de répertoires
- H) Le temps maximum de création d'un symlink
- I) Le nombre de caractères dans un nom de symlink
- J) La limite de symlinks par utilisateur

**Réponse : C** — SYMLOOP_MAX (typiquement 40) est le nombre max de traversées de symlinks pour éviter les boucles infinies (2.3.4.l).

---

**Q8.** Que se passe-t-il quand on supprime un fichier avec plusieurs hard links ?
- A) Tous les hard links sont supprimés
- B) Les données sont immédiatement effacées
- C) Le compteur de liens est décrémenté, données préservées si nlink > 0
- D) Une erreur est retournée
- E) Le fichier est déplacé vers la corbeille
- F) Les autres hard links deviennent des symlinks
- G) Le système demande confirmation
- H) Seul root peut supprimer
- I) Le fichier est marqué comme "supprimé"
- J) Rien ne se passe

**Réponse : C** — unlink() décrémente le compteur (2.3.4.d). Les données sont libérées seulement quand nlink atteint 0.

---

**Q9.** Comment détecter si un fichier a des hard links ?
- A) Vérifier si le fichier est un symlink
- B) Utiliser la commande `ls -l`
- C) Vérifier si st_nlink > 1 avec stat()
- D) Comparer les tailles de fichiers
- E) Utiliser readlink()
- F) Vérifier les permissions
- G) Analyser le nom du fichier
- H) Utiliser access()
- I) Vérifier le propriétaire
- J) Impossible à détecter

**Réponse : C** — st_nlink > 1 indique que l'inode a plusieurs noms (hard links) (2.3.4.c).

---

**Q10.** Pourquoi ne peut-on pas créer de hard link vers un répertoire ?
- A) Les répertoires sont trop gros
- B) Pour éviter les boucles dans l'arborescence
- C) Les répertoires n'ont pas d'inode
- D) C'est autorisé avec sudo
- E) Limitation historique sans raison
- F) Les permissions l'interdisent
- G) Le filesystem ne le supporte pas
- H) Ça créerait des fichiers cachés
- I) Les répertoires sont read-only
- J) C'est une limitation Windows

**Réponse : B** — Hard links sur répertoires créeraient des cycles dans l'arborescence, rendant impossible le parcours (2.3.4.e).

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Élément | Valeur |
|---------|--------|
| **Exercice** | 2.3.4-a : multiverse_link_manager |
| **Thème** | Spider-Man: Into the Spider-Verse |
| **Difficulté** | ★★★★★☆☆☆☆☆ (5/10) |
| **Concepts** | 2.3.4.a → 2.3.4.l (12 concepts) |
| **Fonctions clés** | link(), symlink(), readlink(), stat(), lstat(), unlink() |
| **Pièges majeurs** | readlink sans \0, stat vs lstat, boucles infinies |
| **Tests** | 25 tests fonctionnels |
| **Mutants** | 6 solutions buggées |
| **XP Base** | 500 |
| **Bonus** | ⚡ Standard (×2) |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.4-a-multiverse-link-manager",
    "generated_at": "2026-01-11",

    "metadata": {
      "exercise_id": "2.3.4-a",
      "exercise_name": "multiverse_link_manager",
      "module": "2.3.4",
      "module_name": "Hard Links & Symbolic Links",
      "concept": "a-l",
      "concept_name": "Gestion complète des liens Unix",
      "type": "code",
      "tier": 3,
      "tier_info": "Synthèse (tous concepts 2.3.4)",
      "phase": 2,
      "difficulty": 5,
      "difficulty_stars": "★★★★★☆☆☆☆☆",
      "language": "c",
      "language_version": "C17",
      "duration_minutes": 300,
      "xp_base": 500,
      "xp_bonus_multiplier": 2,
      "bonus_tier": "STANDARD",
      "bonus_icon": "⚡",
      "complexity_time": "T3 O(n)",
      "complexity_space": "S2 O(SYMLOOP_MAX)",
      "prerequisites": ["2.3.1", "2.3.3"],
      "domains": ["FS", "Mem", "Struct"],
      "tags": ["hardlink", "symlink", "link", "unlink", "readlink", "dangling", "loop"],
      "theme": "Spider-Man: Into the Spider-Verse",
      "meme_references": ["Anyone can wear the mask", "I've been falling for 30 minutes"]
    },

    "spec": {
      "name": "multiverse_link_manager",
      "language": "c",
      "type": "code",
      "tier": 3,
      "passing_score": 80,

      "function": {
        "name": "spider_anchor",
        "prototype": "multiverse_error_t spider_anchor(const char *existing, const char *new_anchor)",
        "return_type": "multiverse_error_t",
        "parameters": [
          {"name": "existing", "type": "const char *"},
          {"name": "new_anchor", "type": "const char *"}
        ]
      },

      "driver": {
        "reference": "multiverse_error_t ref_spider_anchor(const char *existing, const char *new_anchor) { struct stat sb_src, sb_dst; if (existing == NULL || new_anchor == NULL) return MULTIVERSE_INVALID_PARAM; if (stat(existing, &sb_src) == -1) return MULTIVERSE_NOT_FOUND; if (S_ISDIR(sb_src.st_mode)) return MULTIVERSE_IS_NEXUS; char *d = strdup(new_anchor); char *p = dirname(d); if (stat(p, &sb_dst) == -1) { free(d); return MULTIVERSE_NOT_FOUND; } free(d); if (sb_src.st_dev != sb_dst.st_dev) return MULTIVERSE_CROSS_DIMENSION; if (link(existing, new_anchor) == -1) { if (errno == EEXIST) return MULTIVERSE_ALREADY_EXISTS; return MULTIVERSE_IO_ERROR; } return MULTIVERSE_OK; }",

        "edge_cases": [
          {"name": "null_existing", "args": [null, "/tmp/x"], "expected": -9, "is_trap": true, "trap_explanation": "existing est NULL"},
          {"name": "null_new", "args": ["/tmp/x", null], "expected": -9, "is_trap": true, "trap_explanation": "new_anchor est NULL"},
          {"name": "empty_existing", "args": ["", "/tmp/x"], "expected": -9, "is_trap": true, "trap_explanation": "Chemin vide"},
          {"name": "source_not_found", "args": ["/nonexistent", "/tmp/x"], "expected": -1, "is_trap": true, "trap_explanation": "Source n'existe pas"},
          {"name": "source_is_dir", "args": ["/tmp", "/tmp/link"], "expected": -5, "is_trap": true, "trap_explanation": "Hard link sur répertoire interdit (2.3.4.e)"},
          {"name": "valid_hardlink", "args": ["/tmp/test.txt", "/tmp/test_link.txt"], "expected": 0}
        ],

        "fuzzing": {
          "enabled": true,
          "iterations": 500,
          "generators": [
            {"type": "string", "param_index": 0, "params": {"min_len": 1, "max_len": 255, "charset": "alphanumeric"}},
            {"type": "string", "param_index": 1, "params": {"min_len": 1, "max_len": 255, "charset": "alphanumeric"}}
          ]
        }
      },

      "norm": {
        "allowed_functions": ["link", "symlink", "unlink", "readlink", "stat", "lstat", "fstat", "open", "close", "read", "write", "opendir", "readdir", "closedir", "malloc", "free", "calloc", "realloc", "strlen", "strcpy", "strncpy", "strcmp", "strcat", "strdup", "snprintf", "realpath", "dirname", "basename", "strerror", "perror", "printf", "fprintf"],
        "forbidden_functions": ["access", "system"],
        "check_security": true,
        "check_memory": true,
        "blocking": true
      }
    },

    "validation": {
      "expected_pass": ["references/ref_solution.c"],
      "expected_fail": ["mutants/mutant_a_safety.c", "mutants/mutant_b_resource.c", "mutants/mutant_c_logic.c", "mutants/mutant_d_boundary.c", "mutants/mutant_e_logic.c", "mutants/mutant_f_return.c"]
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — Module 2.3.4 : Hard Links & Symbolic Links*
*"Anyone can wear the mask." — Into the Spider-Verse*
*L'excellence pédagogique ne se négocie pas*

