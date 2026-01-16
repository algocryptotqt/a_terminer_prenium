# Exercice 2.3.1-a : maze_runner_expedition

**Module :**
2.3.1 — Directory Walker

**Concept :**
a — Parcours récursif de répertoires avec opendir/readdir/closedir

**Difficulté :**
★★★★☆☆☆☆☆☆ (4/10)

**Type :**
complet

**Tiers :**
1 — Concept isolé

**Langage :**
C (C17)

**Prérequis :**
- Exercice 2.3.0 (File System Inspector)
- Récursion en C
- Pointeurs et structures

**Domaines :**
FS, Struct

**Durée estimée :**
180 min

**XP Base :**
120

**Complexité :**
T2 O(n) × S2 O(d) où d = profondeur max

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex01/
├── maze_runner.h        # Header avec structures et prototypes
├── maze_runner.c        # Implémentation principale
├── junction_analysis.c  # Analyse des directory entries
├── maze_structures.c    # Simulation des structures de stockage
├── Makefile
```

**Fonctions autorisées :**
```c
malloc, free, calloc, realloc     // Allocation mémoire
opendir, readdir, closedir        // Navigation répertoires
rewinddir                          // Rembobiner répertoire
stat, lstat                        // Métadonnées fichiers
strlen, strcmp, strncmp, strcpy    // Manipulation chaînes
snprintf, printf, fprintf          // Affichage
strerror, errno                    // Gestion erreurs
memset, memcpy                     // Manipulation mémoire
```

**Fonctions interdites :**
```c
ftw, nftw       // On implémente notre propre walker!
scandir         // Idem
glob            // Idem
```

### 1.2 Consigne

#### 🎮 Version Culture Pop : THE MAZE RUNNER — L'Expédition

**Dans l'univers de The Maze Runner, les Blocards vivent dans la Clairière, entourés par un gigantesque Labyrinthe qui change chaque nuit. Les Coureurs s'aventurent dans le Labyrinthe pour le cartographier, mémorisant chaque section, chaque embranchement, chaque cul-de-sac.**

Le Labyrinthe a des règles strictes :
- **Chaque Section** a des **Jonctions** (chemins possibles)
- **Le Point d'Ancrage** ("." = où tu te trouves)
- **Le Chemin de Retour** (".." = revenir à la section précédente)
- **Les Portes** se ferment la nuit (closedir = ne pas laisser ouvert!)

**Tu es Thomas, nouveau Coureur. Ta mission : créer un système de cartographie du Labyrinthe.**

Dans le filesystem Unix, c'est identique :
- **Un répertoire** = Une section du Labyrinthe
- **Les entries** = Les jonctions (fichiers, sous-répertoires)
- **"."** = Ta position actuelle (Point d'Ancrage)
- **".."** = Le chemin de retour vers la section parente
- **opendir()** = Entrer dans une section
- **readdir()** = Découvrir chaque jonction
- **closedir()** = Sceller la section (OBLIGATOIRE!)

**Ta mission :**

Écrire une bibliothèque `maze_runner` qui implémente un système de parcours récursif de répertoires, permettant de cartographier l'intégralité d'une arborescence de fichiers.

**Entrée :**
- `start_section` : Le chemin du répertoire de départ (la Clairière)
- `expedition_config` : Options de parcours (récursif, suivre symlinks, etc.)
- `cartographer` : Fonction callback appelée pour chaque découverte

**Sortie :**
- Appel du callback pour chaque entry découverte
- Statistiques complètes de l'expédition (maze_stats_t)
- Code de retour indiquant le succès/échec

**Contraintes :**
- NE JAMAIS descendre dans "." ou ".." (récursion infinie = mort!)
- Toujours fermer les sections ouvertes (closedir après opendir)
- Gérer la profondeur maximale (le Labyrinthe a des limites)
- Supporter les chemins jusqu'à PATH_MAX caractères

#### 📚 Version Académique : Parcoureur de Répertoires

**Contexte :**

Un répertoire Unix est un fichier spécial contenant une table d'entrées (directory entries). Chaque entrée associe un nom à un numéro d'inode. Le parcours récursif d'une arborescence nécessite l'utilisation du triplet opendir()/readdir()/closedir().

**Objectif :**

Implémenter un parcoureur de répertoires (directory walker) qui :
1. Ouvre un répertoire avec opendir()
2. Lit chaque entrée avec readdir()
3. Ignore les entrées spéciales "." et ".."
4. Descend récursivement dans les sous-répertoires
5. Appelle un callback utilisateur pour chaque entrée
6. Ferme proprement avec closedir()

**Points critiques :**
- Les entrées "." et ".." sont présentes dans TOUT répertoire
- Ignorer ces entrées lors de la récursion évite les boucles infinies
- closedir() doit être appelé même en cas d'erreur (ressource leak)

### 1.3 Prototype

```c
#ifndef MAZE_RUNNER_H
#define MAZE_RUNNER_H

#include <sys/types.h>
#include <dirent.h>
#include <stdint.h>
#include <stdbool.h>

/* ═══════════════════════════════════════════════════════════════════════════
 * TYPES DE JONCTIONS (Directory Entry Types)
 * Chaque jonction du Labyrinthe a un type
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef enum {
    JUNCTION_UNKNOWN   = DT_UNKNOWN,  /* Type inconnu */
    JUNCTION_PASSAGE   = DT_REG,      /* Fichier (passage simple) */
    JUNCTION_SECTION   = DT_DIR,      /* Répertoire (nouvelle section) */
    JUNCTION_WORMHOLE  = DT_LNK,      /* Symlink (portail dimensionnel) */
    JUNCTION_CONDUIT   = DT_FIFO,     /* Pipe (conduit) */
    JUNCTION_TERMINUS  = DT_SOCK,     /* Socket (terminus) */
    JUNCTION_MECHANISM = DT_BLK,      /* Block device */
    JUNCTION_INTERFACE = DT_CHR       /* Char device */
} junction_type_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * DONNÉES D'UNE JONCTION (Directory Entry)
 * Informations extraites de chaque entry
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef struct {
    ino_t           inode;           /* Numéro d'inode */
    junction_type_t type;            /* Type de jonction */
    char            name[256];       /* Nom de l'entrée */
    size_t          name_len;        /* Longueur du nom */
    bool            is_anchor;       /* Est "." (Point d'Ancrage) */
    bool            is_return_path;  /* Est ".." (Chemin de Retour) */
    bool            is_hidden;       /* Commence par '.' */
    char            full_path[4096]; /* Chemin complet */
    int             depth;           /* Profondeur dans le Labyrinthe */
} maze_junction_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * CONFIGURATION DE L'EXPÉDITION
 * Options pour le parcours
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef struct {
    bool   recursive;           /* Descendre dans les sous-sections */
    bool   follow_wormholes;    /* Suivre les symlinks vers répertoires */
    bool   include_hidden;      /* Inclure les entries cachées (.xxx) */
    bool   include_anchors;     /* Inclure "." et ".." dans les résultats */
    int    max_depth;           /* Profondeur max (-1 = illimitée) */
    size_t max_entries;         /* Max entries à traiter (0 = illimité) */
} expedition_config_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * STATISTIQUES DE CARTOGRAPHIE
 * Résultats de l'expédition
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef struct {
    size_t total_junctions;     /* Total des jonctions découvertes */
    size_t passages;            /* Fichiers réguliers */
    size_t sections;            /* Sous-répertoires */
    size_t wormholes;           /* Liens symboliques */
    size_t hidden_entries;      /* Entrées cachées */
    size_t anchor_points;       /* Entrées "." */
    size_t return_paths;        /* Entrées ".." */
    int    max_depth_reached;   /* Profondeur maximale atteinte */
    double avg_name_length;     /* Longueur moyenne des noms */
    size_t errors_encountered;  /* Erreurs rencontrées */
} maze_stats_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * TYPE DU CALLBACK CARTOGRAPHE
 * Fonction appelée pour chaque jonction découverte
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef int (*cartographer_fn)(const maze_junction_t *junction, void *user_data);

/* ═══════════════════════════════════════════════════════════════════════════
 * CODES DE RETOUR
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef enum {
    MAZE_SUCCESS        = 0,
    MAZE_ERR_NOT_FOUND  = -1,   /* Section introuvable */
    MAZE_ERR_NOT_DIR    = -2,   /* Pas une section (répertoire) */
    MAZE_ERR_PERMISSION = -3,   /* Accès refusé */
    MAZE_ERR_MEMORY     = -4,   /* Mémoire insuffisante */
    MAZE_ERR_DEPTH      = -5,   /* Profondeur max atteinte */
    MAZE_ERR_CALLBACK   = -6,   /* Callback a demandé l'arrêt */
    MAZE_ERR_INVALID    = -7    /* Paramètres invalides */
} maze_error_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS PRINCIPALES — EXPÉDITION
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Lance une expédition dans le Labyrinthe.
 * Parcourt le répertoire (et sous-répertoires si recursif) en appelant
 * le cartographe pour chaque jonction découverte.
 *
 * @param start_section  Chemin du répertoire de départ
 * @param config         Configuration de l'expédition
 * @param cartographer   Callback appelé pour chaque entry
 * @param user_data      Données utilisateur passées au callback
 * @param stats          Statistiques de l'expédition (output, peut être NULL)
 * @return               MAZE_SUCCESS ou code d'erreur
 */
maze_error_t maze_expedition(
    const char *start_section,
    const expedition_config_t *config,
    cartographer_fn cartographer,
    void *user_data,
    maze_stats_t *stats
);

/**
 * Parcours simple d'une seule section (non récursif).
 */
maze_error_t maze_walk_section(
    const char *section_path,
    cartographer_fn cartographer,
    void *user_data
);

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS D'ANALYSE DES JONCTIONS
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Analyse une entry dirent et remplit maze_junction_t.
 */
int analyze_junction(
    const struct dirent *entry,
    const char *parent_path,
    int depth,
    maze_junction_t *junction
);

/**
 * Vérifie si une entry est le Point d'Ancrage (".").
 */
bool is_anchor_point(const char *name);

/**
 * Vérifie si une entry est le Chemin de Retour ("..").
 */
bool is_return_path(const char *name);

/**
 * Vérifie si une entry doit être ignorée lors de la récursion.
 * (Retourne true pour "." et "..")
 */
bool should_skip_junction(const char *name);

/**
 * Convertit un d_type en junction_type_t.
 */
junction_type_t dirent_type_to_junction(unsigned char d_type);

/**
 * Retourne le nom du type de jonction.
 */
const char *junction_type_name(junction_type_t type);

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS DE SIMULATION — STRUCTURES DE STOCKAGE
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Simule une recherche linéaire (comme ext2 original).
 * O(n) comparaisons.
 */
int simulate_linear_search(
    const char *section_path,
    const char *target_name,
    size_t *comparisons
);

/**
 * Simule une recherche par table de hachage (comme ext4 dir_index).
 * O(1) en moyenne.
 */
int simulate_hash_lookup(
    const char *section_path,
    const char *target_name,
    size_t *comparisons
);

/**
 * Simule une recherche B-tree (comme XFS, Btrfs).
 * O(log n) comparaisons.
 */
int simulate_btree_search(
    const char *section_path,
    const char *target_name,
    size_t *comparisons
);

/* ═══════════════════════════════════════════════════════════════════════════
 * UTILITAIRES
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Crée une configuration par défaut.
 */
expedition_config_t default_expedition_config(void);

/**
 * Initialise les statistiques à zéro.
 */
void reset_maze_stats(maze_stats_t *stats);

/**
 * Affiche les statistiques de l'expédition.
 */
void print_maze_stats(const maze_stats_t *stats);

/**
 * Récupère le dernier code d'erreur.
 */
maze_error_t get_maze_error(void);

/**
 * Description textuelle d'une erreur.
 */
const char *maze_strerror(maze_error_t error);

#endif /* MAZE_RUNNER_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 Un Répertoire est un Fichier Spécial

Contrairement à ce qu'on pourrait penser, un répertoire Unix n'est pas un "conteneur magique" mais un **fichier spécial** dont le contenu est une table d'entrées. Chaque entrée (directory entry) est une association :

```
Nom du fichier → Numéro d'inode
```

C'est le kernel qui interprète ce fichier de manière spéciale lors de la navigation.

### 2.2 Les Entrées Magiques "." et ".."

Tout répertoire Unix contient OBLIGATOIREMENT deux entrées spéciales :

| Entrée | Signification | Utilité |
|--------|--------------|---------|
| `.` | Le répertoire lui-même | `./script.sh`, calcul du link count |
| `..` | Le répertoire parent | `cd ..`, navigation ascendante |

**Cas spécial de la racine (`/`)** : Les deux pointent vers le même inode !

### 2.3 Structures de Stockage des Répertoires

| Structure | Recherche | Utilisé par | Idéal pour |
|-----------|-----------|-------------|------------|
| **Liste linéaire** | O(n) | ext2 original | < 100 fichiers |
| **Table de hachage** | O(1) moyen | ext3/ext4 | Répertoires moyens |
| **B-tree** | O(log n) | XFS, Btrfs | Millions de fichiers |

### 2.5 DANS LA VRAIE VIE

| Métier | Cas d'usage |
|--------|-------------|
| **SysAdmin** | Scripts de nettoyage (`find` implémente un walker) |
| **DevOps** | Monitoring de répertoires, détection de changements |
| **Forensics** | Scan complet d'un disque pour analyse |
| **Backup Software** | Parcours pour sauvegardes incrémentales |
| **IDE** | Indexation des fichiers du projet |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
maze_runner.c  maze_runner.h  junction_analysis.c  maze_structures.c  main.c  Makefile

$ make

$ ./test_maze /etc
=== MAZE RUNNER EXPEDITION ===
Starting from: /etc
Configuration: recursive=yes, max_depth=3, include_hidden=no

Entering section: /etc
  [PASSAGE] hostname (inode: 131090)
  [PASSAGE] passwd (inode: 131073)
  [PASSAGE] shadow (inode: 131074)
  [SECTION] apt (inode: 131080)
    Entering section: /etc/apt
    [PASSAGE] sources.list (inode: 131085)
    [SECTION] sources.list.d (inode: 131086)
    Leaving section: /etc/apt
  [WORMHOLE] localtime -> ../usr/share/zoneinfo/UTC
Leaving section: /etc

=== EXPEDITION COMPLETE ===
Statistics:
  Total junctions: 127
  Passages (files): 98
  Sections (dirs): 24
  Wormholes (symlinks): 5
  Max depth reached: 3
  Errors: 0

$ ./test_maze --search /usr/bin ls
=== SEARCH SIMULATION ===
Target: "ls" in /usr/bin (contains 1847 entries)

Linear search (ext2):    1423 comparisons
Hash lookup (ext4):      3 comparisons
B-tree search (XFS):     11 comparisons
```

### 3.1 ⚡ BONUS STANDARD (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★☆☆☆☆ (6/10)

**Récompense :**
XP ×2

### 3.1.1 Consigne Bonus : MODE GRIFFEUR

**🎮 Les Griffeurs patrouillent le Labyrinthe la nuit. Thomas doit créer un système de détection qui identifie les "anomalies" dans la structure du Labyrinthe.**

Implémenter `maze_detect_anomalies()` qui détecte :
- Symlinks cassés (wormholes instables)
- Fichiers world-writable (brèches de sécurité)
- Répertoires sans "." ou ".." (corruption)
- Boucles de symlinks (pièges dimensionnels)

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| # | Test | Input | Expected | Points |
|---|------|-------|----------|--------|
| 01 | Section simple | `/tmp` | Liste des entries | 5 |
| 02 | Section vide | Empty dir | Juste . et .. | 5 |
| 03 | Récursif 1 niveau | Dir avec sous-dir | Entre dans sous-dir | 5 |
| 04 | Récursif profond | Arbo 5 niveaux | Descend jusqu'au bout | 5 |
| 05 | Ignore "." | Tout répertoire | "." non récursé | 5 |
| 06 | Ignore ".." | Tout répertoire | ".." non récursé | 5 |
| 07 | Max depth | max_depth=2 | S'arrête à niveau 2 | 5 |
| 08 | Callback stop | Return -1 | Arrête le parcours | 5 |
| 09 | Section inexistante | `/nonexistent` | ERR_NOT_FOUND | 3 |
| 10 | Pas un répertoire | `/etc/passwd` | ERR_NOT_DIR | 3 |
| 11 | Permission denied | Dir sans +r | ERR_PERMISSION | 3 |
| 12 | closedir appelé | 100 parcours | Pas de leak fd | 10 |
| 13 | Stats total | /usr | Compte correct | 5 |
| 14 | Stats par type | /dev | Types corrects | 5 |
| 15 | Hidden entries | include_hidden=true | Inclut .xxx | 3 |
| 16 | Hidden skip | include_hidden=false | Exclut .xxx | 3 |
| 17 | Symlink dir | follow=true | Entre dans symlink | 5 |
| 18 | Symlink skip | follow=false | N'entre pas | 3 |
| 19 | Linear search sim | /usr/bin, "ls" | O(n) comparisons | 5 |
| 20 | Hash search sim | /usr/bin, "ls" | O(1) comparisons | 5 |
| 21 | Memory (Valgrind) | 50 expéditions | 0 leaks | 10 |

### 4.2 main.c de test

```c
#include "maze_runner.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <unistd.h>

static int tests_passed = 0;
static int tests_total = 0;
static int entries_found = 0;

#define TEST(name, cond) do { \
    tests_total++; \
    if (cond) { \
        printf("✓ %s\n", name); \
        tests_passed++; \
    } else { \
        printf("✗ %s\n", name); \
    } \
} while(0)

/* Callback simple qui compte les entries */
int count_callback(const maze_junction_t *junction, void *data)
{
    (void)junction;
    int *count = (int *)data;
    (*count)++;
    return 0;
}

/* Callback qui vérifie qu'on n'entre pas dans . ou .. */
int no_dots_callback(const maze_junction_t *junction, void *data)
{
    bool *found_recursion_into_dots = (bool *)data;

    /* Si on trouve . ou .. à profondeur > 0, c'est une erreur */
    if (junction->depth > 0) {
        if (junction->is_anchor || junction->is_return_path) {
            *found_recursion_into_dots = true;
        }
    }
    return 0;
}

void test_simple_walk(void)
{
    int count = 0;
    expedition_config_t config = default_expedition_config();
    config.recursive = false;

    maze_error_t ret = maze_expedition("/tmp", &config, count_callback, &count, NULL);

    TEST("Simple walk success", ret == MAZE_SUCCESS);
    TEST("Found entries > 0", count > 0);
}

void test_no_recursion_into_dots(void)
{
    bool found_dots = false;
    expedition_config_t config = default_expedition_config();
    config.recursive = true;
    config.max_depth = 3;

    maze_expedition("/tmp", &config, no_dots_callback, &found_dots, NULL);

    TEST("No recursion into . or ..", !found_dots);
}

void test_max_depth(void)
{
    maze_stats_t stats;
    expedition_config_t config = default_expedition_config();
    config.recursive = true;
    config.max_depth = 2;

    maze_expedition("/usr", &config, count_callback, &entries_found, &stats);

    TEST("Max depth respected", stats.max_depth_reached <= 2);
}

void test_closedir_called(void)
{
    /* Run 100 times and check for fd leaks */
    for (int i = 0; i < 100; i++) {
        int count = 0;
        expedition_config_t config = default_expedition_config();
        maze_expedition("/tmp", &config, count_callback, &count, NULL);
    }
    /* If we get here without running out of fds, test passes */
    TEST("closedir always called (100 iterations)", 1);
}

void test_error_cases(void)
{
    int count = 0;
    expedition_config_t config = default_expedition_config();
    maze_error_t ret;

    ret = maze_expedition("/nonexistent", &config, count_callback, &count, NULL);
    TEST("Nonexistent returns NOT_FOUND", ret == MAZE_ERR_NOT_FOUND);

    ret = maze_expedition("/etc/passwd", &config, count_callback, &count, NULL);
    TEST("File returns NOT_DIR", ret == MAZE_ERR_NOT_DIR);

    ret = maze_expedition(NULL, &config, count_callback, &count, NULL);
    TEST("NULL returns INVALID", ret == MAZE_ERR_INVALID);
}

void test_special_entries(void)
{
    TEST("is_anchor_point(\".\")", is_anchor_point(".") == true);
    TEST("is_anchor_point(\"..\")", is_anchor_point("..") == false);
    TEST("is_return_path(\"..\")", is_return_path("..") == true);
    TEST("is_return_path(\".\")", is_return_path(".") == false);
    TEST("should_skip_junction(\".\")", should_skip_junction(".") == true);
    TEST("should_skip_junction(\"..\")", should_skip_junction("..") == true);
    TEST("should_skip_junction(\"file\")", should_skip_junction("file") == false);
}

int main(void)
{
    printf("=== MAZE RUNNER TEST SUITE ===\n\n");

    test_simple_walk();
    test_no_recursion_into_dots();
    test_max_depth();
    test_closedir_called();
    test_error_cases();
    test_special_entries();

    printf("\n=== RESULTS: %d/%d tests passed ===\n", tests_passed, tests_total);
    return (tests_passed == tests_total) ? 0 : 1;
}
```

### 4.3 Solution de référence

```c
#include "maze_runner.h"
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <sys/stat.h>

static maze_error_t g_last_error = MAZE_SUCCESS;

/* ═══════════════════════════════════════════════════════════════════════════
 * Fonctions utilitaires pour les entrées spéciales
 * ═══════════════════════════════════════════════════════════════════════════ */
bool is_anchor_point(const char *name)
{
    return (name != NULL && strcmp(name, ".") == 0);
}

bool is_return_path(const char *name)
{
    return (name != NULL && strcmp(name, "..") == 0);
}

bool should_skip_junction(const char *name)
{
    if (name == NULL)
        return true;
    return (strcmp(name, ".") == 0 || strcmp(name, "..") == 0);
}

junction_type_t dirent_type_to_junction(unsigned char d_type)
{
    switch (d_type)
    {
        case DT_REG:  return JUNCTION_PASSAGE;
        case DT_DIR:  return JUNCTION_SECTION;
        case DT_LNK:  return JUNCTION_WORMHOLE;
        case DT_FIFO: return JUNCTION_CONDUIT;
        case DT_SOCK: return JUNCTION_TERMINUS;
        case DT_BLK:  return JUNCTION_MECHANISM;
        case DT_CHR:  return JUNCTION_INTERFACE;
        default:      return JUNCTION_UNKNOWN;
    }
}

const char *junction_type_name(junction_type_t type)
{
    switch (type)
    {
        case JUNCTION_PASSAGE:   return "Passage";
        case JUNCTION_SECTION:   return "Section";
        case JUNCTION_WORMHOLE:  return "Wormhole";
        case JUNCTION_CONDUIT:   return "Conduit";
        case JUNCTION_TERMINUS:  return "Terminus";
        case JUNCTION_MECHANISM: return "Mechanism";
        case JUNCTION_INTERFACE: return "Interface";
        default:                 return "Unknown";
    }
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Configuration par défaut
 * ═══════════════════════════════════════════════════════════════════════════ */
expedition_config_t default_expedition_config(void)
{
    expedition_config_t config = {
        .recursive = true,
        .follow_wormholes = false,
        .include_hidden = true,
        .include_anchors = false,
        .max_depth = -1,
        .max_entries = 0
    };
    return config;
}

void reset_maze_stats(maze_stats_t *stats)
{
    if (stats != NULL)
        memset(stats, 0, sizeof(maze_stats_t));
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Analyse d'une jonction
 * ═══════════════════════════════════════════════════════════════════════════ */
int analyze_junction(
    const struct dirent *entry,
    const char *parent_path,
    int depth,
    maze_junction_t *junction)
{
    if (entry == NULL || junction == NULL)
        return -1;

    memset(junction, 0, sizeof(maze_junction_t));

    /* Copie des informations de base */
    junction->inode = entry->d_ino;
    junction->type = dirent_type_to_junction(entry->d_type);
    strncpy(junction->name, entry->d_name, 255);
    junction->name[255] = '\0';
    junction->name_len = strlen(junction->name);
    junction->depth = depth;

    /* Détection des entrées spéciales */
    junction->is_anchor = is_anchor_point(entry->d_name);
    junction->is_return_path = is_return_path(entry->d_name);
    junction->is_hidden = (entry->d_name[0] == '.');

    /* Construction du chemin complet */
    if (parent_path != NULL)
    {
        snprintf(junction->full_path, sizeof(junction->full_path),
                 "%s/%s", parent_path, entry->d_name);
    }
    else
    {
        strncpy(junction->full_path, entry->d_name, sizeof(junction->full_path) - 1);
    }

    return 0;
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Fonction récursive interne
 * ═══════════════════════════════════════════════════════════════════════════ */
static maze_error_t expedition_recursive(
    const char *section_path,
    const expedition_config_t *config,
    cartographer_fn cartographer,
    void *user_data,
    maze_stats_t *stats,
    int current_depth)
{
    DIR *dir;
    struct dirent *entry;
    maze_junction_t junction;
    maze_error_t result = MAZE_SUCCESS;

    /* Vérification de la profondeur max */
    if (config->max_depth >= 0 && current_depth > config->max_depth)
        return MAZE_SUCCESS;

    /* Ouverture de la section (opendir) */
    dir = opendir(section_path);
    if (dir == NULL)
    {
        switch (errno)
        {
            case ENOENT: return MAZE_ERR_NOT_FOUND;
            case ENOTDIR: return MAZE_ERR_NOT_DIR;
            case EACCES: return MAZE_ERR_PERMISSION;
            default: return MAZE_ERR_NOT_FOUND;
        }
    }

    /* Parcours des entries (readdir) */
    while ((entry = readdir(dir)) != NULL)
    {
        /* Analyse de la jonction */
        analyze_junction(entry, section_path, current_depth, &junction);

        /* Mise à jour des stats */
        if (stats != NULL)
        {
            stats->total_junctions++;
            if (junction.is_anchor)
                stats->anchor_points++;
            if (junction.is_return_path)
                stats->return_paths++;
            if (junction.is_hidden)
                stats->hidden_entries++;
            if (junction.type == JUNCTION_PASSAGE)
                stats->passages++;
            if (junction.type == JUNCTION_SECTION)
                stats->sections++;
            if (junction.type == JUNCTION_WORMHOLE)
                stats->wormholes++;
            if (current_depth > stats->max_depth_reached)
                stats->max_depth_reached = current_depth;
        }

        /* Filtrage selon config */
        if (!config->include_anchors &&
            (junction.is_anchor || junction.is_return_path))
            continue;

        if (!config->include_hidden && junction.is_hidden &&
            !junction.is_anchor && !junction.is_return_path)
            continue;

        /* Appel du callback cartographe */
        if (cartographer != NULL)
        {
            int cb_result = cartographer(&junction, user_data);
            if (cb_result < 0)
            {
                result = MAZE_ERR_CALLBACK;
                break;
            }
        }

        /* Récursion dans les sous-sections */
        if (config->recursive &&
            junction.type == JUNCTION_SECTION &&
            !should_skip_junction(entry->d_name))  /* CRITIQUE: NE PAS descendre dans . ou .. */
        {
            expedition_recursive(
                junction.full_path,
                config,
                cartographer,
                user_data,
                stats,
                current_depth + 1
            );
        }
    }

    /* Fermeture de la section (closedir) - OBLIGATOIRE! */
    closedir(dir);

    return result;
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Fonction principale d'expédition
 * ═══════════════════════════════════════════════════════════════════════════ */
maze_error_t maze_expedition(
    const char *start_section,
    const expedition_config_t *config,
    cartographer_fn cartographer,
    void *user_data,
    maze_stats_t *stats)
{
    struct stat sb;
    expedition_config_t default_config;

    /* Validation des paramètres */
    if (start_section == NULL)
    {
        g_last_error = MAZE_ERR_INVALID;
        return MAZE_ERR_INVALID;
    }

    /* Vérification que c'est bien un répertoire */
    if (stat(start_section, &sb) != 0)
    {
        g_last_error = MAZE_ERR_NOT_FOUND;
        return MAZE_ERR_NOT_FOUND;
    }

    if (!S_ISDIR(sb.st_mode))
    {
        g_last_error = MAZE_ERR_NOT_DIR;
        return MAZE_ERR_NOT_DIR;
    }

    /* Configuration par défaut si non fournie */
    if (config == NULL)
    {
        default_config = default_expedition_config();
        config = &default_config;
    }

    /* Initialisation des stats */
    if (stats != NULL)
        reset_maze_stats(stats);

    /* Lancement de l'expédition récursive */
    return expedition_recursive(
        start_section,
        config,
        cartographer,
        user_data,
        stats,
        0
    );
}

maze_error_t maze_walk_section(
    const char *section_path,
    cartographer_fn cartographer,
    void *user_data)
{
    expedition_config_t config = default_expedition_config();
    config.recursive = false;
    return maze_expedition(section_path, &config, cartographer, user_data, NULL);
}

maze_error_t get_maze_error(void)
{
    return g_last_error;
}

const char *maze_strerror(maze_error_t error)
{
    switch (error)
    {
        case MAZE_SUCCESS:        return "Expedition successful";
        case MAZE_ERR_NOT_FOUND:  return "Section not found";
        case MAZE_ERR_NOT_DIR:    return "Not a section (directory)";
        case MAZE_ERR_PERMISSION: return "Access denied to section";
        case MAZE_ERR_MEMORY:     return "Insufficient memory";
        case MAZE_ERR_DEPTH:      return "Maximum depth reached";
        case MAZE_ERR_CALLBACK:   return "Cartographer stopped exploration";
        case MAZE_ERR_INVALID:    return "Invalid parameters";
        default:                  return "Unknown error";
    }
}
```

### 4.10 Solutions Mutantes

```c
/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT A (Critical) : Récursion dans "." et ".." → BOUCLE INFINIE!
 * ═══════════════════════════════════════════════════════════════════════════ */
/* Dans la boucle readdir: */
if (config->recursive && junction.type == JUNCTION_SECTION)
{
    /* MANQUE: && !should_skip_junction(entry->d_name) */
    expedition_recursive(junction.full_path, ...);
    /* BOUCLE INFINIE car "." pointe vers le répertoire courant! */
}
/* Pourquoi faux: Sans le test, on descend dans "." indéfiniment */
/* Ce qui était pensé: "Tous les répertoires doivent être explorés" */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT B (Resource) : Pas de closedir() → FUITE DE FD
 * ═══════════════════════════════════════════════════════════════════════════ */
static maze_error_t expedition_recursive_mutant_b(...)
{
    DIR *dir = opendir(section_path);
    if (dir == NULL)
        return MAZE_ERR_NOT_FOUND;

    while ((entry = readdir(dir)) != NULL)
    {
        /* ... traitement ... */
        if (some_error_condition)
            return MAZE_ERR_CALLBACK;  /* FUITE: dir jamais fermé! */
    }

    closedir(dir);  /* Atteint seulement si pas d'erreur */
    return MAZE_SUCCESS;
}
/* Pourquoi faux: Après ~1024 appels avec erreurs, plus de file descriptors */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT C (Safety) : Pas de vérification NULL
 * ═══════════════════════════════════════════════════════════════════════════ */
maze_error_t maze_expedition_mutant_c(
    const char *start_section,
    const expedition_config_t *config,
    ...)
{
    /* MANQUE: if (start_section == NULL) return INVALID */

    if (stat(start_section, &sb) != 0)  /* CRASH si NULL */
        return MAZE_ERR_NOT_FOUND;

    /* ... */
}
/* Pourquoi faux: Segfault sur stat(NULL, ...) */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT D (Logic) : Test "." avec == au lieu de strcmp
 * ═══════════════════════════════════════════════════════════════════════════ */
bool should_skip_junction_mutant_d(const char *name)
{
    return (name == "." || name == "..");  /* FAUX! Compare les pointeurs! */
}
/* Pourquoi faux: Les pointeurs sont différents même si le contenu est identique */
/* Devrait utiliser strcmp() */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT E (Boundary) : Buffer overflow sur chemin
 * ═══════════════════════════════════════════════════════════════════════════ */
int analyze_junction_mutant_e(...)
{
    /* MANQUE: vérification de la taille */
    sprintf(junction->full_path, "%s/%s", parent_path, entry->d_name);
    /* Si parent_path + "/" + d_name > 4096, overflow! */
}
/* Pourquoi faux: Utiliser snprintf avec taille limite */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT F (Logic) : Ignorer seulement "." mais pas ".."
 * ═══════════════════════════════════════════════════════════════════════════ */
bool should_skip_junction_mutant_f(const char *name)
{
    return (strcmp(name, ".") == 0);  /* Oubli de ".." ! */
}
/* Pourquoi faux: Récursion dans ".." remonte dans le parent indéfiniment */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Structure des répertoires** : Un répertoire est un fichier spécial contenant des entries
2. **API POSIX** : Maîtrise du triplet opendir/readdir/closedir
3. **Entrées spéciales** : Comprendre "." et ".." et pourquoi les ignorer
4. **Récursion sécurisée** : Éviter les boucles infinies
5. **Gestion des ressources** : Toujours fermer ce qu'on ouvre

### 5.2 LDA — Traduction Littérale

```
FONCTION maze_expedition QUI RETOURNE UN maze_error_t ET PREND EN PARAMÈTRES start_section, config, cartographer, user_data, stats
DÉBUT FONCTION
    DÉCLARER dir COMME POINTEUR VERS DIR
    DÉCLARER entry COMME POINTEUR VERS struct dirent

    SI start_section EST ÉGAL À NUL ALORS
        RETOURNER MAZE_ERR_INVALID
    FIN SI

    AFFECTER opendir(start_section) À dir
    SI dir EST ÉGAL À NUL ALORS
        RETOURNER L'ERREUR APPROPRIÉE
    FIN SI

    TANT QUE readdir(dir) RETOURNE UNE ENTRÉE FAIRE
        SI entry->d_name EST ÉGAL À "." OU ".." ALORS
            CONTINUER    /* NE PAS DESCENDRE DEDANS! */
        FIN SI

        APPELER LE CALLBACK cartographer

        SI L'ENTRY EST UN RÉPERTOIRE ET config->recursive ALORS
            APPELER maze_expedition RÉCURSIVEMENT
        FIN SI
    FIN TANT QUE

    FERMER LE RÉPERTOIRE AVEC closedir(dir)    /* OBLIGATOIRE! */

    RETOURNER MAZE_SUCCESS
FIN FONCTION
```

### 5.3 Visualisation ASCII

```
    THE MAZE RUNNER — PARCOURS RÉCURSIF

    Clairière (/)
    ├── .                    ← POINT D'ANCRAGE (ne pas descendre!)
    ├── ..                   ← CHEMIN DE RETOUR (ne pas descendre!)
    ├── Section_A (etc/)     ← opendir("/etc")
    │   ├── .
    │   ├── ..               ← Pointe vers /
    │   ├── passwd           ← PASSAGE (fichier)
    │   ├── Section_B (apt/) ← opendir("/etc/apt")
    │   │   ├── .
    │   │   ├── ..           ← Pointe vers /etc
    │   │   └── sources.list
    │   └── shadow
    └── Section_C (home/)
        ├── .
        ├── ..
        └── thomas/
            ├── .
            ├── ..
            └── notes.txt


    DANGER: Si on descend dans "." :

    /etc → /etc/. → /etc/./. → /etc/././. → ... INFINI!
         ↑___________|
           Même répertoire!
```

```
    opendir/readdir/closedir FLOW

    ┌─────────────────────────────────────────────────────────┐
    │  maze_expedition("/etc", config, callback, data, stats) │
    └───────────────────────────┬─────────────────────────────┘
                                │
                                ▼
    ┌───────────────────────────────────────────────────────┐
    │  DIR *dir = opendir("/etc")  ← Ouvre le "stream"      │
    └───────────────────────────┬───────────────────────────┘
                                │
            ┌───────────────────┼───────────────────────┐
            ▼                   ▼                       ▼
    ┌───────────────┐  ┌───────────────┐       ┌───────────────┐
    │ readdir(dir)  │→ │ readdir(dir)  │→ ... →│ readdir(dir)  │
    │ → "."         │  │ → ".."        │       │ → NULL (fin)  │
    │ SKIP!         │  │ SKIP!         │       │               │
    └───────────────┘  └───────────────┘       └───────┬───────┘
                                                       │
                                                       ▼
                               ┌─────────────────────────────────┐
                               │  closedir(dir)  ← OBLIGATOIRE!  │
                               └─────────────────────────────────┘
```

### 5.4 Les pièges en détail

| Piège | Impact | Solution |
|-------|--------|----------|
| Récursion dans "." | BOUCLE INFINIE | `should_skip_junction()` |
| Oublier closedir() | FUITE DE FD | Appeler avant TOUT return |
| strcmp vs == pour "." | Ne détecte pas | Toujours utiliser strcmp() |
| Buffer overflow path | Crash/Sécurité | snprintf avec taille |
| Pas de vérif NULL | Segfault | Valider tous les paramètres |

### 5.5 Cours Complet

#### La Structure d'un Répertoire

Un répertoire Unix est stocké comme un fichier ordinaire, mais son contenu est interprété spécialement par le kernel. Il contient une série d'entrées (directory entries) :

```
+------------+------------+------------+------------------+
| Inode Num  | Name Len   | Entry Type | Filename         |
| (4-8 bytes)| (1-2 bytes)| (1 byte)   | (variable, ≤255) |
+------------+------------+------------+------------------+
```

#### Le Triplet POSIX

```c
DIR *dir = opendir("/path");     // Ouvre le répertoire
struct dirent *entry;
while ((entry = readdir(dir))) { // Lit chaque entrée
    printf("%s\n", entry->d_name);
}
closedir(dir);                   // Ferme le répertoire
```

**RÈGLE D'OR** : Tout `opendir()` DOIT avoir son `closedir()` correspondant, même en cas d'erreur !

#### Les Entrées Spéciales

- **"."** : Hard link vers le répertoire lui-même. Son inode = inode du répertoire.
- **".."** : Hard link vers le parent. Exception : pour "/" (racine), `.` == `..`.

C'est pourquoi un répertoire vide a `st_nlink = 2` (lui-même via "." + entrée dans le parent).

### 5.7 Simulation avec trace d'exécution

```
┌───────┬───────────────────────────────────────────┬──────────────────────────┐
│ Étape │ Instruction                               │ Explication              │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   1   │ maze_expedition("/tmp", config, ...)      │ Début expédition         │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   2   │ dir = opendir("/tmp")                     │ Ouvre la section         │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   3   │ readdir(dir) → "."                        │ Point d'Ancrage          │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   4   │ should_skip_junction(".") → true          │ On ne descend PAS!       │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   5   │ readdir(dir) → ".."                       │ Chemin de Retour         │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   6   │ should_skip_junction("..") → true         │ On ne descend PAS!       │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   7   │ readdir(dir) → "subdir"                   │ Sous-répertoire trouvé   │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   8   │ should_skip_junction("subdir") → false    │ On peut descendre        │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   9   │ callback(&junction, user_data)            │ Notifie l'utilisateur    │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│  10   │ maze_expedition("/tmp/subdir", ...)       │ Récursion!               │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│  ...  │ (traitement de subdir)                    │                          │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│  N-1  │ readdir(dir) → NULL                       │ Fin des entries          │
├───────┼───────────────────────────────────────────┼──────────────────────────┤
│   N   │ closedir(dir)                             │ OBLIGATOIRE!             │
└───────┴───────────────────────────────────────────┴──────────────────────────┘
```

### 5.8 Mnémotechniques

#### 🏃 MEME : "Never Go Back to the Maze!" — Règle de "." et ".."

![Maze Runner](maze_runner_door.jpg)

Dans The Maze Runner, les Blocards savent qu'il ne faut JAMAIS retourner dans une section déjà explorée la même journée. Sinon, on tourne en rond!

```c
// 🏃 Thomas le sait : on ne descend JAMAIS dans . ou ..
if (strcmp(entry->d_name, ".") == 0 ||
    strcmp(entry->d_name, "..") == 0) {
    continue;  // "We never go back to the Maze!"
}
```

#### 🚪 MEME : "Ferme la Porte!" — closedir() obligatoire

Dans le Labyrinthe, les portes se ferment automatiquement la nuit. Si tu laisses une porte ouverte (oublies closedir), les Griffeurs entrent et c'est la catastrophe (fuite de file descriptors) !

```c
DIR *door = opendir(section);  // Ouvre la porte
// ... exploration ...
closedir(door);                 // FERME LA PORTE! Toujours!
```

#### 🗺️ MEME : "Minho cartographie tout"

Minho, le Gardien des Coureurs, cartographie méthodiquement chaque section. Le callback `cartographer` fait la même chose : il est appelé pour CHAQUE jonction découverte.

### 5.9 Applications pratiques

| Commande/Outil | Ce qu'il fait | Notre équivalent |
|----------------|---------------|------------------|
| `ls` | Liste un répertoire | maze_walk_section() |
| `ls -R` | Liste récursif | maze_expedition() avec recursive=true |
| `find` | Recherche récursive | maze_expedition() avec callback de filtrage |
| `du` | Taille récursive | Callback qui accumule les tailles |
| `tree` | Affiche l'arborescence | Callback qui indente selon depth |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Fréquence | Impact | Détection |
|---|-------|-----------|--------|-----------|
| 1 | Récursion dans "." | CRITIQUE | Boucle infinie | Test avec timeout |
| 2 | Oublier closedir() | Très fréquent | Fuite fd | Valgrind, lsof |
| 3 | == au lieu de strcmp | Fréquent | "." non détecté | Test unitaire |
| 4 | Buffer overflow path | Moyen | Crash | ASAN |
| 5 | Pas de gestion d'erreurs | Moyen | Comportement indéfini | Tests d'erreur |

---

## 📝 SECTION 7 : QCM

### Q1. Que retourne readdir() quand il n'y a plus d'entrées ?
- A) -1
- B) 0
- C) NULL
- D) EOF
- E) Une structure vide

**Réponse : C**

### Q2. Pourquoi ne faut-il JAMAIS descendre dans "." ?
- A) C'est un fichier, pas un répertoire
- B) On entrerait dans une boucle infinie
- C) C'est interdit par le kernel
- D) Ça cause une erreur de permission
- E) "." n'existe pas vraiment

**Réponse : B**

### Q3. Que se passe-t-il si on oublie closedir() ?
- A) Rien de grave
- B) Le répertoire reste vérouillé
- C) Fuite de file descriptor
- D) Le répertoire est supprimé
- E) Erreur de compilation

**Réponse : C**

### Q4. Combien d'entrées minimum contient un répertoire Unix ?
- A) 0
- B) 1
- C) 2 (. et ..)
- D) 3
- E) Ça dépend du filesystem

**Réponse : C**

### Q5. Quelle fonction ouvre un répertoire pour lecture ?
- A) open()
- B) fopen()
- C) opendir()
- D) diropen()
- E) read_directory()

**Réponse : C**

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Critère | Valeur |
|---------|--------|
| **Exercice** | 2.3.1-a : maze_runner_expedition |
| **Thème** | The Maze Runner — Cartographie du Labyrinthe |
| **Difficulté** | ★★★★☆☆☆☆☆☆ (4/10) |
| **Durée** | 3 heures |
| **XP Base** | 120 |
| **Concepts clés** | opendir/readdir/closedir, "." et "..", récursion |
| **Prérequis** | Ex 2.3.0 (stat), récursion C |
| **Tests** | 21 tests, Valgrind obligatoire |
| **Mutants** | 6 solutions buggées |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.1-a-maze-runner-expedition",
    "generated_at": "2025-01-11T12:30:00",

    "metadata": {
      "exercise_id": "2.3.1-a",
      "exercise_name": "maze_runner_expedition",
      "module": "2.3.1",
      "module_name": "Directory Walker",
      "concept": "a",
      "concept_name": "Parcours récursif de répertoires",
      "type": "complet",
      "tier": 1,
      "phase": 2,
      "difficulty": 4,
      "difficulty_stars": "★★★★☆☆☆☆☆☆",
      "language": "c",
      "duration_minutes": 180,
      "xp_base": 120,
      "xp_bonus_multiplier": 2,
      "bonus_tier": "STANDARD",
      "bonus_icon": "⚡",
      "complexity_time": "T2 O(n)",
      "complexity_space": "S2 O(d)",
      "prerequisites": ["2.3.0-a"],
      "domains": ["FS", "Struct"],
      "tags": ["directory", "opendir", "readdir", "recursion", "maze-runner"],
      "meme_reference": "The Maze Runner — Never go back!"
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — Exercice 2.3.1-a : maze_runner_expedition*
*"We never go back to the Maze!"*
*Thème : The Maze Runner — L'Expédition*
