# Exercice 2.3.0-a : samus_scan

**Module :**
2.3.0 — File System Inspector

**Concept :**
a — Analyse complète des métadonnées fichiers via stat()

**Difficulté :**
★★★★☆☆☆☆☆☆ (4/10)

**Type :**
complet

**Tiers :**
1 — Concept isolé

**Langage :**
C (C17)

**Prérequis :**
- Pointeurs et structures en C
- Manipulation de chaînes
- Bases des syscalls Unix

**Domaines :**
FS, Encodage

**Durée estimée :**
240 min

**XP Base :**
150

**Complexité :**
T1 O(1) × S1 O(1)

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex00/
├── scan_visor.h        # Header avec structures et prototypes
├── scan_visor.c        # Implémentation principale (stat)
├── entity_types.c      # Détection des types de fichiers
├── access_protocols.c  # Formatage des permissions
├── temporal_markers.c  # Formatage des timestamps
├── Makefile
```

**Fonctions autorisées :**
```c
malloc, free, calloc, realloc     // Allocation mémoire
stat, lstat, fstat                 // Analyse des fichiers
open, close, read, write, unlink   // Opérations fichiers
opendir, readdir, closedir         // Navigation répertoires
readlink, getcwd, realpath         // Résolution chemins
strlen, strcpy, strncpy, strcmp    // Manipulation chaînes
snprintf, printf, fprintf          // Affichage
localtime, strftime, time          // Timestamps
getpwuid, getgrgid                 // Résolution noms
strerror, errno                    // Gestion erreurs
```

**Fonctions interdites :**
```c
access      // Utiliser stat() à la place
system      // Pas d'appels shell
exec*       // Pas de fork/exec
```

### 1.2 Consigne

#### 🎮 Version Culture Pop : METROID PRIME — SCAN VISOR

**Dans l'univers de Metroid Prime, la chasseuse de primes Samus Aran possède une armure équipée de plusieurs visières. La plus importante est le SCAN VISOR — une technologie Chozo qui permet d'analyser n'importe quel objet, créature ou mécanisme pour en extraire toutes les informations.**

Quand Samus scanne un ennemi, elle obtient :
- Son **type** (Bioform, Mécanisme, Artefact, etc.)
- Ses **caractéristiques** (taille, résistances)
- Ses **faiblesses** (points d'accès)
- Son **historique** (quand il a été modifié)

**Tu es ingénieur chez Retro Studios. Ta mission : implémenter le système de scan pour le prochain Metroid.**

Dans le filesystem Unix, chaque fichier est comme une entité à scanner :
- **L'inode** = La signature neurale unique de l'entité
- **Le type** = Sa classification (Bioform/Directory, Mechanism/Device, etc.)
- **Les permissions** = Ses protocoles d'accès (qui peut interagir)
- **Les timestamps** = Son historique temporel
- **Les blocs** = Son allocation d'énergie

**Ta mission :**

Écrire une bibliothèque `scan_visor` qui implémente le système de scan de Samus, permettant d'analyser n'importe quel fichier Unix et d'en extraire TOUTES ses métadonnées.

**Entrée :**
- `target_path` : Le chemin vers l'entité à scanner (fichier/répertoire)
- `scan_mode` : Mode de scan (`SCAN_FOLLOW` ou `SCAN_SURFACE` pour les symlinks)

**Sortie :**
- Structure `scan_data_t` contenant TOUTES les métadonnées de l'inode
- `NULL` si le scan échoue (avec code d'erreur approprié)

**Contraintes :**
- Supporter les 7 types d'entités Unix (regular, directory, symlink, block, char, fifo, socket)
- Gérer les chemins absolus ET relatifs
- Résoudre les noms d'utilisateurs/groupes
- Formater les permissions en notation symbolique (`rwxr-xr-x`)
- Afficher les timestamps en ISO 8601
- Aucune fuite mémoire (Valgrind clean)

#### 📚 Version Académique : Inspecteur de Système de Fichiers

**Contexte :**

Dans les systèmes Unix, chaque fichier possède des métadonnées stockées dans une structure appelée **inode** (index node). L'inode contient toutes les informations sur le fichier SAUF son nom (stocké dans le répertoire parent) et son contenu (stocké dans les blocs de données).

**Objectif :**

Implémenter un inspecteur de système de fichiers qui utilise le syscall `stat()` pour récupérer et afficher toutes les métadonnées d'un fichier :
- Numéro d'inode (identifiant unique)
- Type de fichier (7 types possibles)
- Permissions (mode bits)
- Propriétaire (UID/GID)
- Taille en bytes
- Timestamps (atime, mtime, ctime)
- Nombre de liens (hard links)
- Allocation en blocs

### 1.3 Prototype

```c
#ifndef SCAN_VISOR_H
#define SCAN_VISOR_H

#include <sys/types.h>
#include <sys/stat.h>
#include <stdint.h>
#include <stddef.h>
#include <time.h>

/* ═══════════════════════════════════════════════════════════════════════════
 * CLASSIFICATION DES ENTITÉS (Types de fichiers Unix)
 * Dans Metroid, chaque entité scannée a une classification
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef enum {
    ENTITY_UNKNOWN    = 0,   /* Entité non identifiée */
    ENTITY_DATAFORM   = 1,   /* Fichier régulier (données) */
    ENTITY_HIVE       = 2,   /* Répertoire (contient d'autres entités) */
    ENTITY_WORMHOLE   = 3,   /* Lien symbolique (portail vers autre entité) */
    ENTITY_MECHANISM  = 4,   /* Device bloc (machinerie lourde) */
    ENTITY_INTERFACE  = 5,   /* Device caractère (terminal d'interface) */
    ENTITY_CONDUIT    = 6,   /* FIFO/Pipe (conduit de données) */
    ENTITY_NEXUS      = 7    /* Socket (point de connexion réseau) */
} entity_class_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * MODE DE SCAN
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef enum {
    SCAN_FOLLOW   = 0,   /* Suivre les wormholes (symlinks) - stat() */
    SCAN_SURFACE  = 1    /* Scan de surface uniquement - lstat() */
} scan_mode_t;

typedef enum {
    PATH_ABSOLUTE = 0,   /* Coordonnées galactiques (depuis racine) */
    PATH_RELATIVE = 1    /* Coordonnées locales (depuis position actuelle) */
} path_type_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * DONNÉES DE SCAN - Toutes les métadonnées de l'inode
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef struct {
    /* === Identification === */
    char           *target_path;     /* Chemin fourni pour le scan */
    char           *resolved_path;   /* Coordonnées absolues résolues */
    path_type_t     path_type;       /* Type de coordonnées */

    /* === Signature Neurale (Inode Number) === */
    ino_t           neural_sig;      /* ID unique dans le filesystem */
    dev_t           sector;          /* Secteur (device) contenant l'entité */

    /* === Classification === */
    entity_class_t  entity_class;    /* Type d'entité */

    /* === Protocoles d'Accès (Permissions) === */
    mode_t          access_mode;     /* Mode bits bruts */

    /* === Propriétaire === */
    uid_t           owner_id;        /* ID du propriétaire */
    gid_t           group_id;        /* ID du groupe */
    char           *owner_name;      /* Nom résolu du propriétaire */
    char           *group_name;      /* Nom résolu du groupe */

    /* === Dimensions === */
    off_t           data_size;       /* Taille en bytes */

    /* === Marqueurs Temporels === */
    time_t          last_access;     /* Dernier accès (atime) */
    time_t          last_modify;     /* Dernière modification contenu (mtime) */
    time_t          last_change;     /* Dernier changement inode (ctime) */

    /* === Liens Symbiotiques === */
    nlink_t         symbiotic_count; /* Nombre de hard links */

    /* === Allocation Énergie === */
    blksize_t       block_size;      /* Taille de bloc préférée */
    blkcnt_t        blocks_alloc;    /* Blocs 512-byte alloués */

    /* === Données Spéciales === */
    char           *wormhole_dest;   /* Destination du symlink (si applicable) */
    dev_t           device_id;       /* Major/minor pour devices */
} scan_data_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * CODES D'ERREUR DE SCAN
 * ═══════════════════════════════════════════════════════════════════════════ */
typedef enum {
    SCAN_SUCCESS       = 0,
    SCAN_ERR_NOT_FOUND = -1,   /* Entité introuvable */
    SCAN_ERR_DENIED    = -2,   /* Accès refusé par protocoles */
    SCAN_ERR_MEMORY    = -3,   /* Mémoire insuffisante */
    SCAN_ERR_PATH_LONG = -4,   /* Coordonnées trop longues */
    SCAN_ERR_INVALID   = -5,   /* Paramètres invalides */
    SCAN_ERR_IO        = -6,   /* Erreur I/O */
    SCAN_ERR_LOOP      = -7    /* Boucle de wormholes détectée */
} scan_error_t;

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS PRINCIPALES - SCAN VISOR
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Active le Scan Visor sur une cible.
 * Récupère TOUTES les métadonnées de l'inode via stat()/lstat().
 *
 * @param target_path Coordonnées de la cible (chemin fichier)
 * @param mode        Mode de scan (SCAN_FOLLOW ou SCAN_SURFACE)
 * @return            Données de scan allouées, ou NULL si échec
 */
scan_data_t *samus_scan(const char *target_path, scan_mode_t mode);

/**
 * Libère les données de scan.
 */
void scan_data_free(scan_data_t *data);

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS DE NAVIGATION
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Détermine le type de coordonnées (absolues ou relatives).
 */
path_type_t get_path_type(const char *path);

/**
 * Résout des coordonnées relatives en absolues.
 */
char *resolve_coordinates(const char *path, char *resolved, size_t size);

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS DE CLASSIFICATION
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Classifie une entité depuis son mode.
 */
entity_class_t classify_entity(mode_t mode);

/**
 * Retourne le nom de la classification.
 */
const char *entity_class_name(entity_class_t class);

/**
 * Retourne le caractère de classification (pour affichage ls).
 */
char entity_class_char(entity_class_t class);

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS DE FORMATAGE - PROTOCOLES D'ACCÈS
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Convertit les protocoles d'accès en notation symbolique.
 * Format: "rwxr-xr-x" (9 caractères + bits spéciaux)
 */
char *format_access_protocols(mode_t mode, char *buf, size_t size);

/**
 * Convertit les protocoles en notation octale.
 * Format: "0755"
 */
char *format_access_octal(mode_t mode, char *buf, size_t size);

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS DE FORMATAGE - DIMENSIONS ET TEMPS
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Formate une taille en format humain (KB, MB, GB).
 */
char *format_data_size(off_t size, char *buf, size_t size);

/**
 * Formate un timestamp en ISO 8601.
 */
char *format_timestamp_iso(time_t ts, char *buf, size_t size);

/**
 * Formate un timestamp style ls.
 */
char *format_timestamp_ls(time_t ts, char *buf, size_t size);

/* ═══════════════════════════════════════════════════════════════════════════
 * FONCTIONS D'AFFICHAGE
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Affiche le rapport de scan complet (style commande stat).
 */
void display_scan_report(const scan_data_t *data);

/**
 * Affiche en format ls -li (inode + permissions + infos).
 */
void display_ls_format(const scan_data_t *data);

/**
 * Scanne et affiche le contenu d'un HIVE (répertoire).
 */
int scan_hive_contents(const char *path);

/* ═══════════════════════════════════════════════════════════════════════════
 * UTILITAIRES
 * ═══════════════════════════════════════════════════════════════════════════ */

/**
 * Récupère le dernier code d'erreur de scan.
 */
scan_error_t get_scan_error(void);

/**
 * Description textuelle d'une erreur de scan.
 */
const char *scan_strerror(scan_error_t error);

/**
 * Affiche une explication pédagogique de ce que contient un inode.
 */
void explain_neural_signature(void);

#endif /* SCAN_VISOR_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'Inode : Le Cerveau du Fichier

Dans Unix, chaque fichier a un **inode** (index node) qui stocke TOUTES ses métadonnées. C'est comme la fiche d'identité complète du fichier.

**Ce que l'inode CONTIENT :**
- Type de fichier (regular, directory, symlink, device, etc.)
- Permissions (qui peut lire/écrire/exécuter)
- Propriétaire (UID et GID)
- Taille en bytes
- Timestamps (atime, mtime, ctime)
- Nombre de liens (hard links)
- Pointeurs vers les blocs de données

**Ce que l'inode NE CONTIENT PAS :**
- Le **nom du fichier** (stocké dans le répertoire parent !)
- Le **contenu du fichier** (stocké dans les blocs de données)

C'est pourquoi plusieurs noms (hard links) peuvent pointer vers le même inode !

### 2.2 Les 7 Types de Fichiers Unix

| Type | Char | Macro C | Description |
|------|------|---------|-------------|
| Regular | `-` | S_ISREG | Fichier ordinaire (texte, binaire) |
| Directory | `d` | S_ISDIR | Répertoire (conteneur) |
| Symbolic Link | `l` | S_ISLNK | Lien symbolique (raccourci) |
| Block Device | `b` | S_ISBLK | Device bloc (disque dur) |
| Character Device | `c` | S_ISCHR | Device caractère (terminal) |
| FIFO | `p` | S_ISFIFO | Pipe nommé (IPC) |
| Socket | `s` | S_ISSOCK | Socket Unix (réseau local) |

### 2.5 DANS LA VRAIE VIE

**Qui utilise ces concepts ?**

| Métier | Cas d'usage |
|--------|-------------|
| **SysAdmin** | Diagnostic avec `ls -li`, `stat`, `find -inum` pour retrouver des fichiers par inode |
| **DevOps** | Scripts de monitoring qui vérifient les permissions et timestamps des fichiers de config |
| **Forensics** | Analyse de timestamps pour retracer les activités sur un système compromis |
| **Développeur Backend** | Vérification des permissions avant d'accéder aux fichiers utilisateurs |
| **DBA** | Monitoring de la taille et des blocs alloués pour les fichiers de base de données |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
scan_visor.c  scan_visor.h  entity_types.c  access_protocols.c  temporal_markers.c  main.c  Makefile

$ make

$ ./test_scan /etc/passwd
=== SCAN VISOR ACTIVATED ===
Target: /etc/passwd

--- Neural Signature (Inode) ---
Signature:   131073
Sector:      0x820 (major: 8, minor: 32)

--- Entity Classification ---
Class:       DATAFORM (Regular file)

--- Access Protocols ---
Mode:        -rw-r--r-- (0644)
             Owner: rw- (read, write)
             Group: r-- (read)
             Other: r-- (read)

--- Ownership ---
Owner ID:    0 (root)
Group ID:    0 (root)

--- Dimensions ---
Size:        2847 bytes (2.8 KB)

--- Temporal Markers ---
Last Access: 2025-01-04T10:30:45
Last Modify: 2024-12-15T09:22:11
Last Change: 2024-12-15T09:22:11

--- Symbiotic Links ---
Link Count:  1

--- Energy Allocation ---
Block Size:  4096 bytes
Blocks:      8 (512-byte units)

=== SCAN COMPLETE ===

$ ./test_scan --ls /etc/passwd
131073 -rw-r--r-- 1 root root 2847 Jan  4 10:30 /etc/passwd

$ ./test_scan /dev/null
=== SCAN VISOR ACTIVATED ===
Target: /dev/null

--- Entity Classification ---
Class:       INTERFACE (Character device)
Device:      1:3 (major:minor)
...
```

### 3.1 ⚡ BONUS STANDARD (OPTIONNEL)

**Difficulté Bonus :**
★★★★★★☆☆☆☆ (6/10)

**Récompense :**
XP ×2

**Time Complexity attendue :**
O(n) pour scanner un répertoire

**Space Complexity attendue :**
O(1) auxiliaire

### 3.1.1 Consigne Bonus : SCAN VISOR AMÉLIORÉ

**🎮 Samus découvre une zone infestée de Metroids. Elle doit scanner TOUS les éléments d'une zone (répertoire) et détecter les anomalies.**

Implémenter `scan_hive_recursive()` qui :
- Scanne récursivement un répertoire et ses sous-répertoires
- Affiche les statistiques globales (nombre par type, taille totale)
- Détecte les fichiers avec des permissions dangereuses (world-writable)
- Identifie les symlinks cassés (wormholes instables)

**Prototype Bonus :**

```c
typedef struct {
    int total_entities;
    int by_class[8];        /* Compteur par classe d'entité */
    off_t total_size;       /* Taille totale */
    int broken_wormholes;   /* Symlinks cassés */
    int dangerous_access;   /* Fichiers world-writable */
} hive_stats_t;

int scan_hive_recursive(const char *path, hive_stats_t *stats);
```

### 3.1.2 Ce qui change par rapport à l'exercice de base

| Aspect | Base | Bonus |
|--------|------|-------|
| Cible | 1 fichier | Répertoire entier |
| Récursion | Non | Oui |
| Statistiques | Non | Agrégation |
| Détection anomalies | Non | Oui |

---

## ✅❌ SECTION 4 : ZONE CORRECTION

### 4.1 Moulinette

| # | Test | Input | Expected | Points |
|---|------|-------|----------|--------|
| 01 | Fichier régulier | `/etc/passwd` | type=DATAFORM, size>0 | 5 |
| 02 | Répertoire | `/tmp` | type=HIVE, links>=2 | 5 |
| 03 | Symlink follow | Symlink vers fichier | type=DATAFORM | 5 |
| 04 | Symlink surface | Symlink vers fichier | type=WORMHOLE | 5 |
| 05 | Device char | `/dev/null` | type=INTERFACE | 5 |
| 06 | Device block | `/dev/sda` (si existe) | type=MECHANISM | 5 |
| 07 | Path absolu | `/etc/passwd` | path_type=ABSOLUTE | 3 |
| 08 | Path relatif | `./file` | path_type=RELATIVE | 3 |
| 09 | Résolution path | `../etc/passwd` | resolved=/etc/passwd | 5 |
| 10 | Permissions rwx | Mode 0755 | "rwxr-xr-x" | 5 |
| 11 | Permissions setuid | Mode 04755 | "rwsr-xr-x" | 3 |
| 12 | Permissions sticky | Mode 01755 | "rwxr-xr-t" | 3 |
| 13 | Owner name | /etc/passwd | owner="root" | 3 |
| 14 | Size correct | Fichier 100 bytes | size=100 | 3 |
| 15 | Timestamps valides | Fichier récent | atime,mtime,ctime > 0 | 5 |
| 16 | Hard links | 2 liens vers même inode | link_count=2 | 5 |
| 17 | Blocs alloués | Fichier 8KB | blocks >= 16 | 3 |
| 18 | NULL input | NULL | NULL, error=INVALID | 5 |
| 19 | Fichier inexistant | `/nonexistent` | NULL, error=NOT_FOUND | 5 |
| 20 | Symlink cassé follow | Dangling symlink | NULL, error=NOT_FOUND | 5 |
| 21 | Symlink cassé surface | Dangling symlink | type=WORMHOLE, dest set | 5 |
| 22 | Mémoire (Valgrind) | 100 scans | 0 leaks | 10 |
| 23 | ls -i format | `/etc/passwd` | Matches `ls -i` output | 3 |

### 4.2 main.c de test

```c
#include "scan_visor.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <unistd.h>

static int tests_passed = 0;
static int tests_total = 0;

#define TEST(name, cond) do { \
    tests_total++; \
    if (cond) { \
        printf("✓ %s\n", name); \
        tests_passed++; \
    } else { \
        printf("✗ %s\n", name); \
    } \
} while(0)

void test_regular_file(void)
{
    scan_data_t *data = samus_scan("/etc/passwd", SCAN_FOLLOW);
    TEST("Regular file scan", data != NULL);
    TEST("Type is DATAFORM", data && data->entity_class == ENTITY_DATAFORM);
    TEST("Size > 0", data && data->data_size > 0);
    TEST("Neural sig > 0", data && data->neural_sig > 0);
    TEST("Owner name resolved", data && data->owner_name != NULL);
    scan_data_free(data);
}

void test_directory(void)
{
    scan_data_t *data = samus_scan("/tmp", SCAN_FOLLOW);
    TEST("Directory scan", data != NULL);
    TEST("Type is HIVE", data && data->entity_class == ENTITY_HIVE);
    TEST("Link count >= 2", data && data->symbiotic_count >= 2);
    scan_data_free(data);
}

void test_symlink(void)
{
    /* Créer un symlink de test */
    system("ln -sf /etc/passwd /tmp/test_symlink_scan");

    scan_data_t *follow = samus_scan("/tmp/test_symlink_scan", SCAN_FOLLOW);
    scan_data_t *surface = samus_scan("/tmp/test_symlink_scan", SCAN_SURFACE);

    TEST("Symlink follow -> DATAFORM", follow && follow->entity_class == ENTITY_DATAFORM);
    TEST("Symlink surface -> WORMHOLE", surface && surface->entity_class == ENTITY_WORMHOLE);
    TEST("Wormhole dest set", surface && surface->wormhole_dest != NULL);

    scan_data_free(follow);
    scan_data_free(surface);
    unlink("/tmp/test_symlink_scan");
}

void test_path_types(void)
{
    TEST("Absolute path", get_path_type("/etc/passwd") == PATH_ABSOLUTE);
    TEST("Relative path ./", get_path_type("./file") == PATH_RELATIVE);
    TEST("Relative path ../", get_path_type("../file") == PATH_RELATIVE);
    TEST("Relative path plain", get_path_type("file") == PATH_RELATIVE);
}

void test_permissions(void)
{
    char buf[16];

    TEST("Perms 0755", strcmp(format_access_protocols(0100755, buf, 16), "rwxr-xr-x") == 0);
    TEST("Perms 0644", strcmp(format_access_protocols(0100644, buf, 16), "rw-r--r--") == 0);
    TEST("Perms setuid", strstr(format_access_protocols(0104755, buf, 16), "s") != NULL);
}

void test_errors(void)
{
    scan_data_t *data;

    data = samus_scan(NULL, SCAN_FOLLOW);
    TEST("NULL path returns NULL", data == NULL);
    TEST("Error is INVALID", get_scan_error() == SCAN_ERR_INVALID);

    data = samus_scan("/nonexistent/file", SCAN_FOLLOW);
    TEST("Nonexistent returns NULL", data == NULL);
    TEST("Error is NOT_FOUND", get_scan_error() == SCAN_ERR_NOT_FOUND);
}

void test_memory(void)
{
    /* Test pour Valgrind - pas de leaks */
    for (int i = 0; i < 100; i++) {
        scan_data_t *data = samus_scan("/etc/passwd", SCAN_FOLLOW);
        if (data) scan_data_free(data);
    }
    TEST("Memory test (100 iterations)", 1); /* Valgrind vérifiera */
}

int main(void)
{
    printf("=== SCAN VISOR TEST SUITE ===\n\n");

    test_regular_file();
    test_directory();
    test_symlink();
    test_path_types();
    test_permissions();
    test_errors();
    test_memory();

    printf("\n=== RESULTS: %d/%d tests passed ===\n", tests_passed, tests_total);
    return (tests_passed == tests_total) ? 0 : 1;
}
```

### 4.3 Solution de référence

```c
#include "scan_visor.h"
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <pwd.h>
#include <grp.h>
#include <limits.h>

static scan_error_t g_last_error = SCAN_SUCCESS;

/* ═══════════════════════════════════════════════════════════════════════════
 * Classification des entités
 * ═══════════════════════════════════════════════════════════════════════════ */
entity_class_t classify_entity(mode_t mode)
{
    if (S_ISREG(mode))  return ENTITY_DATAFORM;
    if (S_ISDIR(mode))  return ENTITY_HIVE;
    if (S_ISLNK(mode))  return ENTITY_WORMHOLE;
    if (S_ISBLK(mode))  return ENTITY_MECHANISM;
    if (S_ISCHR(mode))  return ENTITY_INTERFACE;
    if (S_ISFIFO(mode)) return ENTITY_CONDUIT;
    if (S_ISSOCK(mode)) return ENTITY_NEXUS;
    return ENTITY_UNKNOWN;
}

const char *entity_class_name(entity_class_t class)
{
    static const char *names[] = {
        "Unknown", "Dataform", "Hive", "Wormhole",
        "Mechanism", "Interface", "Conduit", "Nexus"
    };
    if (class < 0 || class > 7) return "Unknown";
    return names[class];
}

char entity_class_char(entity_class_t class)
{
    static const char chars[] = "?-dlbcps";
    if (class < 0 || class > 7) return '?';
    return chars[class];
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Navigation et chemins
 * ═══════════════════════════════════════════════════════════════════════════ */
path_type_t get_path_type(const char *path)
{
    if (path == NULL || path[0] == '\0')
        return PATH_RELATIVE;
    return (path[0] == '/') ? PATH_ABSOLUTE : PATH_RELATIVE;
}

char *resolve_coordinates(const char *path, char *resolved, size_t size)
{
    if (path == NULL || resolved == NULL || size == 0)
        return NULL;

    char *result = realpath(path, NULL);
    if (result == NULL)
    {
        /* Si le fichier n'existe pas, construire le chemin manuellement */
        if (path[0] == '/')
        {
            strncpy(resolved, path, size - 1);
            resolved[size - 1] = '\0';
        }
        else
        {
            char cwd[PATH_MAX];
            if (getcwd(cwd, PATH_MAX) == NULL)
                return NULL;
            snprintf(resolved, size, "%s/%s", cwd, path);
        }
        return resolved;
    }

    strncpy(resolved, result, size - 1);
    resolved[size - 1] = '\0';
    free(result);
    return resolved;
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Formatage des permissions
 * ═══════════════════════════════════════════════════════════════════════════ */
char *format_access_protocols(mode_t mode, char *buf, size_t size)
{
    if (buf == NULL || size < 10)
        return NULL;

    /* User permissions */
    buf[0] = (mode & S_IRUSR) ? 'r' : '-';
    buf[1] = (mode & S_IWUSR) ? 'w' : '-';
    if (mode & S_ISUID)
        buf[2] = (mode & S_IXUSR) ? 's' : 'S';
    else
        buf[2] = (mode & S_IXUSR) ? 'x' : '-';

    /* Group permissions */
    buf[3] = (mode & S_IRGRP) ? 'r' : '-';
    buf[4] = (mode & S_IWGRP) ? 'w' : '-';
    if (mode & S_ISGID)
        buf[5] = (mode & S_IXGRP) ? 's' : 'S';
    else
        buf[5] = (mode & S_IXGRP) ? 'x' : '-';

    /* Other permissions */
    buf[6] = (mode & S_IROTH) ? 'r' : '-';
    buf[7] = (mode & S_IWOTH) ? 'w' : '-';
    if (mode & S_ISVTX)
        buf[8] = (mode & S_IXOTH) ? 't' : 'T';
    else
        buf[8] = (mode & S_IXOTH) ? 'x' : '-';

    buf[9] = '\0';
    return buf;
}

char *format_access_octal(mode_t mode, char *buf, size_t size)
{
    if (buf == NULL || size < 5)
        return NULL;
    snprintf(buf, size, "%04o", mode & 07777);
    return buf;
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Formatage taille et temps
 * ═══════════════════════════════════════════════════════════════════════════ */
char *format_data_size(off_t size, char *buf, size_t buf_size)
{
    if (buf == NULL || buf_size < 16)
        return NULL;

    const char *units[] = {"B", "KB", "MB", "GB", "TB"};
    double dsize = (double)size;
    int unit = 0;

    while (dsize >= 1024.0 && unit < 4)
    {
        dsize /= 1024.0;
        unit++;
    }

    if (unit == 0)
        snprintf(buf, buf_size, "%ld %s", (long)size, units[0]);
    else
        snprintf(buf, buf_size, "%.1f %s", dsize, units[unit]);

    return buf;
}

char *format_timestamp_iso(time_t ts, char *buf, size_t size)
{
    if (buf == NULL || size < 20)
        return NULL;

    struct tm *tm = localtime(&ts);
    if (tm == NULL)
        return NULL;

    strftime(buf, size, "%Y-%m-%dT%H:%M:%S", tm);
    return buf;
}

char *format_timestamp_ls(time_t ts, char *buf, size_t size)
{
    if (buf == NULL || size < 13)
        return NULL;

    struct tm *tm = localtime(&ts);
    if (tm == NULL)
        return NULL;

    time_t now = time(NULL);
    time_t six_months = 6 * 30 * 24 * 60 * 60;

    if (now - ts > six_months)
        strftime(buf, size, "%b %e  %Y", tm);
    else
        strftime(buf, size, "%b %e %H:%M", tm);

    return buf;
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Fonction principale de scan
 * ═══════════════════════════════════════════════════════════════════════════ */
scan_data_t *samus_scan(const char *target_path, scan_mode_t mode)
{
    struct stat sb;
    scan_data_t *data;
    int ret;

    /* Validation des paramètres */
    if (target_path == NULL || target_path[0] == '\0')
    {
        g_last_error = SCAN_ERR_INVALID;
        return NULL;
    }

    /* Appel stat() ou lstat() selon le mode */
    if (mode == SCAN_FOLLOW)
        ret = stat(target_path, &sb);
    else
        ret = lstat(target_path, &sb);

    if (ret != 0)
    {
        switch (errno)
        {
            case ENOENT:  g_last_error = SCAN_ERR_NOT_FOUND; break;
            case EACCES:  g_last_error = SCAN_ERR_DENIED; break;
            case ELOOP:   g_last_error = SCAN_ERR_LOOP; break;
            case ENAMETOOLONG: g_last_error = SCAN_ERR_PATH_LONG; break;
            default:      g_last_error = SCAN_ERR_IO; break;
        }
        return NULL;
    }

    /* Allocation de la structure */
    data = calloc(1, sizeof(scan_data_t));
    if (data == NULL)
    {
        g_last_error = SCAN_ERR_MEMORY;
        return NULL;
    }

    /* Copie du chemin */
    data->target_path = strdup(target_path);
    if (data->target_path == NULL)
    {
        free(data);
        g_last_error = SCAN_ERR_MEMORY;
        return NULL;
    }

    /* Résolution du chemin absolu */
    data->resolved_path = malloc(PATH_MAX);
    if (data->resolved_path == NULL)
    {
        free(data->target_path);
        free(data);
        g_last_error = SCAN_ERR_MEMORY;
        return NULL;
    }
    resolve_coordinates(target_path, data->resolved_path, PATH_MAX);

    /* Type de chemin */
    data->path_type = get_path_type(target_path);

    /* Données de l'inode */
    data->neural_sig = sb.st_ino;
    data->sector = sb.st_dev;
    data->entity_class = classify_entity(sb.st_mode);
    data->access_mode = sb.st_mode;
    data->owner_id = sb.st_uid;
    data->group_id = sb.st_gid;
    data->data_size = sb.st_size;
    data->last_access = sb.st_atime;
    data->last_modify = sb.st_mtime;
    data->last_change = sb.st_ctime;
    data->symbiotic_count = sb.st_nlink;
    data->block_size = sb.st_blksize;
    data->blocks_alloc = sb.st_blocks;
    data->device_id = sb.st_rdev;

    /* Résolution du nom propriétaire */
    struct passwd *pw = getpwuid(sb.st_uid);
    if (pw != NULL)
        data->owner_name = strdup(pw->pw_name);

    /* Résolution du nom groupe */
    struct group *gr = getgrgid(sb.st_gid);
    if (gr != NULL)
        data->group_name = strdup(gr->gr_name);

    /* Si c'est un symlink, lire la destination */
    if (data->entity_class == ENTITY_WORMHOLE)
    {
        char link_target[PATH_MAX];
        ssize_t len = readlink(target_path, link_target, PATH_MAX - 1);
        if (len > 0)
        {
            link_target[len] = '\0';
            data->wormhole_dest = strdup(link_target);
        }
    }

    g_last_error = SCAN_SUCCESS;
    return data;
}

void scan_data_free(scan_data_t *data)
{
    if (data == NULL)
        return;

    free(data->target_path);
    free(data->resolved_path);
    free(data->owner_name);
    free(data->group_name);
    free(data->wormhole_dest);
    free(data);
}

scan_error_t get_scan_error(void)
{
    return g_last_error;
}

const char *scan_strerror(scan_error_t error)
{
    switch (error)
    {
        case SCAN_SUCCESS:       return "Scan successful";
        case SCAN_ERR_NOT_FOUND: return "Entity not found";
        case SCAN_ERR_DENIED:    return "Access denied by protocols";
        case SCAN_ERR_MEMORY:    return "Insufficient memory";
        case SCAN_ERR_PATH_LONG: return "Coordinates too long";
        case SCAN_ERR_INVALID:   return "Invalid parameters";
        case SCAN_ERR_IO:        return "I/O error";
        case SCAN_ERR_LOOP:      return "Wormhole loop detected";
        default:                 return "Unknown error";
    }
}

/* ═══════════════════════════════════════════════════════════════════════════
 * Affichage
 * ═══════════════════════════════════════════════════════════════════════════ */
void display_scan_report(const scan_data_t *data)
{
    char buf[64];

    if (data == NULL)
        return;

    printf("=== SCAN VISOR ACTIVATED ===\n");
    printf("Target: %s\n\n", data->target_path);

    printf("--- Neural Signature (Inode) ---\n");
    printf("Signature:   %lu\n", (unsigned long)data->neural_sig);
    printf("Sector:      0x%lx\n\n", (unsigned long)data->sector);

    printf("--- Entity Classification ---\n");
    printf("Class:       %s (%c)\n\n",
           entity_class_name(data->entity_class),
           entity_class_char(data->entity_class));

    printf("--- Access Protocols ---\n");
    format_access_protocols(data->access_mode, buf, sizeof(buf));
    printf("Mode:        %c%s (", entity_class_char(data->entity_class), buf);
    format_access_octal(data->access_mode, buf, sizeof(buf));
    printf("%s)\n\n", buf);

    printf("--- Ownership ---\n");
    printf("Owner ID:    %d (%s)\n", data->owner_id,
           data->owner_name ? data->owner_name : "?");
    printf("Group ID:    %d (%s)\n\n", data->group_id,
           data->group_name ? data->group_name : "?");

    printf("--- Dimensions ---\n");
    format_data_size(data->data_size, buf, sizeof(buf));
    printf("Size:        %ld bytes (%s)\n\n", (long)data->data_size, buf);

    printf("--- Temporal Markers ---\n");
    format_timestamp_iso(data->last_access, buf, sizeof(buf));
    printf("Last Access: %s\n", buf);
    format_timestamp_iso(data->last_modify, buf, sizeof(buf));
    printf("Last Modify: %s\n", buf);
    format_timestamp_iso(data->last_change, buf, sizeof(buf));
    printf("Last Change: %s\n\n", buf);

    printf("--- Symbiotic Links ---\n");
    printf("Link Count:  %lu\n\n", (unsigned long)data->symbiotic_count);

    printf("--- Energy Allocation ---\n");
    printf("Block Size:  %ld bytes\n", (long)data->block_size);
    printf("Blocks:      %ld (512-byte units)\n\n", (long)data->blocks_alloc);

    if (data->wormhole_dest)
    {
        printf("--- Wormhole Destination ---\n");
        printf("Target:      %s\n\n", data->wormhole_dest);
    }

    printf("=== SCAN COMPLETE ===\n");
}

void display_ls_format(const scan_data_t *data)
{
    char perms[16];
    char time_buf[16];

    if (data == NULL)
        return;

    format_access_protocols(data->access_mode, perms, sizeof(perms));
    format_timestamp_ls(data->last_modify, time_buf, sizeof(time_buf));

    printf("%lu %c%s %lu %s %s %ld %s %s",
           (unsigned long)data->neural_sig,
           entity_class_char(data->entity_class),
           perms,
           (unsigned long)data->symbiotic_count,
           data->owner_name ? data->owner_name : "?",
           data->group_name ? data->group_name : "?",
           (long)data->data_size,
           time_buf,
           data->target_path);

    if (data->wormhole_dest)
        printf(" -> %s", data->wormhole_dest);

    printf("\n");
}
```

### 4.4 Solutions alternatives acceptées

```c
/* Alternative 1: Utilisation de fstat() après open() */
scan_data_t *samus_scan_alt(const char *path, scan_mode_t mode)
{
    int fd;
    int flags = O_RDONLY;

    if (mode == SCAN_SURFACE)
        flags |= O_NOFOLLOW;

    fd = open(path, flags);
    if (fd < 0)
        return NULL;

    struct stat sb;
    if (fstat(fd, &sb) != 0)
    {
        close(fd);
        return NULL;
    }
    close(fd);

    /* ... reste identique ... */
}

/* Alternative 2: Macros au lieu de switch pour classification */
entity_class_t classify_entity_alt(mode_t mode)
{
    return S_ISREG(mode) ? ENTITY_DATAFORM :
           S_ISDIR(mode) ? ENTITY_HIVE :
           S_ISLNK(mode) ? ENTITY_WORMHOLE :
           S_ISBLK(mode) ? ENTITY_MECHANISM :
           S_ISCHR(mode) ? ENTITY_INTERFACE :
           S_ISFIFO(mode) ? ENTITY_CONDUIT :
           S_ISSOCK(mode) ? ENTITY_NEXUS : ENTITY_UNKNOWN;
}
```

### 4.5 Solutions refusées

```c
/* REFUSÉ: Pas de vérification NULL */
scan_data_t *samus_scan_bad1(const char *path, scan_mode_t mode)
{
    struct stat sb;
    stat(path, &sb);  /* CRASH si path == NULL */
    /* ... */
}
/* Pourquoi refusé: Segfault garanti sur entrée NULL */

/* REFUSÉ: Fuite mémoire */
scan_data_t *samus_scan_bad2(const char *path, scan_mode_t mode)
{
    scan_data_t *data = malloc(sizeof(scan_data_t));
    data->target_path = strdup(path);

    struct stat sb;
    if (stat(path, &sb) != 0)
        return NULL;  /* FUITE: data et target_path jamais libérés */
    /* ... */
}

/* REFUSÉ: Buffer overflow potentiel */
char *format_perms_bad(mode_t mode, char *buf, size_t size)
{
    /* Pas de vérification de size >= 10 */
    buf[0] = 'r';
    buf[1] = 'w';
    /* ... crash si size < 10 */
}
```

### 4.6 Solution bonus de référence

```c
#include "scan_visor.h"
#include <dirent.h>
#include <string.h>
#include <limits.h>

int scan_hive_recursive(const char *path, hive_stats_t *stats)
{
    DIR *dir;
    struct dirent *entry;
    char full_path[PATH_MAX];

    if (path == NULL || stats == NULL)
        return -1;

    dir = opendir(path);
    if (dir == NULL)
        return -1;

    while ((entry = readdir(dir)) != NULL)
    {
        /* Skip . et .. */
        if (strcmp(entry->d_name, ".") == 0 ||
            strcmp(entry->d_name, "..") == 0)
            continue;

        snprintf(full_path, PATH_MAX, "%s/%s", path, entry->d_name);

        scan_data_t *data = samus_scan(full_path, SCAN_SURFACE);
        if (data == NULL)
        {
            /* Symlink cassé ? */
            if (get_scan_error() == SCAN_ERR_NOT_FOUND)
                stats->broken_wormholes++;
            continue;
        }

        stats->total_entities++;
        stats->by_class[data->entity_class]++;
        stats->total_size += data->data_size;

        /* Vérifier permissions dangereuses (world-writable) */
        if (data->access_mode & S_IWOTH)
            stats->dangerous_access++;

        /* Récursion si c'est un HIVE */
        if (data->entity_class == ENTITY_HIVE)
            scan_hive_recursive(full_path, stats);

        scan_data_free(data);
    }

    closedir(dir);
    return 0;
}
```

### 4.9 spec.json

```json
{
  "name": "samus_scan",
  "language": "c",
  "type": "code",
  "tier": 1,
  "tier_info": "Concept isolé - Inspection fichiers via stat()",
  "tags": ["filesystem", "stat", "inode", "permissions", "phase2"],
  "passing_score": 70,

  "function": {
    "name": "samus_scan",
    "prototype": "scan_data_t *samus_scan(const char *target_path, scan_mode_t mode)",
    "return_type": "scan_data_t *",
    "parameters": [
      {"name": "target_path", "type": "const char *"},
      {"name": "mode", "type": "scan_mode_t"}
    ]
  },

  "driver": {
    "reference": "scan_data_t *ref_samus_scan(const char *target_path, scan_mode_t mode) { struct stat sb; if (target_path == NULL || target_path[0] == '\\0') return NULL; int ret = (mode == 0) ? stat(target_path, &sb) : lstat(target_path, &sb); if (ret != 0) return NULL; scan_data_t *d = calloc(1, sizeof(scan_data_t)); if (!d) return NULL; d->target_path = strdup(target_path); d->neural_sig = sb.st_ino; d->entity_class = S_ISREG(sb.st_mode) ? 1 : S_ISDIR(sb.st_mode) ? 2 : S_ISLNK(sb.st_mode) ? 3 : 0; d->access_mode = sb.st_mode; d->owner_id = sb.st_uid; d->group_id = sb.st_gid; d->data_size = sb.st_size; d->symbiotic_count = sb.st_nlink; return d; }",

    "edge_cases": [
      {
        "name": "null_path",
        "args": [null, 0],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "path NULL doit retourner NULL"
      },
      {
        "name": "empty_path",
        "args": ["", 0],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "Chemin vide doit retourner NULL"
      },
      {
        "name": "nonexistent_file",
        "args": ["/nonexistent/file/path", 0],
        "expected": null,
        "is_trap": true,
        "trap_explanation": "Fichier inexistant doit retourner NULL"
      },
      {
        "name": "regular_file",
        "args": ["/etc/passwd", 0],
        "expected": "entity_class == ENTITY_DATAFORM"
      },
      {
        "name": "directory",
        "args": ["/tmp", 0],
        "expected": "entity_class == ENTITY_HIVE"
      },
      {
        "name": "symlink_follow",
        "args": ["/tmp/test_symlink", 0],
        "expected": "entity_class == type of target"
      },
      {
        "name": "symlink_surface",
        "args": ["/tmp/test_symlink", 1],
        "expected": "entity_class == ENTITY_WORMHOLE"
      }
    ],

    "fuzzing": {
      "enabled": true,
      "iterations": 500,
      "generators": [
        {
          "type": "string",
          "param_index": 0,
          "params": {
            "min_len": 0,
            "max_len": 256,
            "charset": "printable"
          }
        },
        {
          "type": "int",
          "param_index": 1,
          "params": {
            "min": 0,
            "max": 1
          }
        }
      ]
    }
  },

  "norm": {
    "allowed_functions": ["malloc", "free", "calloc", "realloc", "stat", "lstat", "fstat", "open", "close", "read", "write", "unlink", "opendir", "readdir", "closedir", "readlink", "getcwd", "realpath", "strlen", "strcpy", "strncpy", "strcmp", "strdup", "snprintf", "printf", "fprintf", "localtime", "strftime", "time", "getpwuid", "getgrgid", "strerror"],
    "forbidden_functions": ["access", "system", "exec", "fork"],
    "check_security": true,
    "check_memory": true,
    "blocking": true
  }
}
```

### 4.10 Solutions Mutantes

```c
/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT A (Boundary) : Pas de vérification taille buffer permissions
 * ═══════════════════════════════════════════════════════════════════════════ */
char *format_access_protocols_mutant_a(mode_t mode, char *buf, size_t size)
{
    /* MANQUE: if (size < 10) return NULL; */
    buf[0] = (mode & S_IRUSR) ? 'r' : '-';
    buf[1] = (mode & S_IWUSR) ? 'w' : '-';
    buf[2] = (mode & S_IXUSR) ? 'x' : '-';
    buf[3] = (mode & S_IRGRP) ? 'r' : '-';
    buf[4] = (mode & S_IWGRP) ? 'w' : '-';
    buf[5] = (mode & S_IXGRP) ? 'x' : '-';
    buf[6] = (mode & S_IROTH) ? 'r' : '-';
    buf[7] = (mode & S_IWOTH) ? 'w' : '-';
    buf[8] = (mode & S_IXOTH) ? 'x' : '-';
    buf[9] = '\0';
    return buf;
}
/* Pourquoi faux: Buffer overflow si size < 10 */
/* Ce qui était pensé: "Le buffer sera toujours assez grand" */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT B (Safety) : Pas de vérification NULL pour path
 * ═══════════════════════════════════════════════════════════════════════════ */
scan_data_t *samus_scan_mutant_b(const char *target_path, scan_mode_t mode)
{
    struct stat sb;

    /* MANQUE: if (target_path == NULL) return NULL; */

    int ret = (mode == SCAN_FOLLOW) ? stat(target_path, &sb) : lstat(target_path, &sb);
    if (ret != 0)
        return NULL;

    scan_data_t *data = calloc(1, sizeof(scan_data_t));
    data->target_path = strdup(target_path);  /* CRASH si target_path == NULL */
    /* ... */
    return data;
}
/* Pourquoi faux: Segfault si target_path == NULL */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT C (Resource) : Fuite mémoire en cas d'erreur
 * ═══════════════════════════════════════════════════════════════════════════ */
scan_data_t *samus_scan_mutant_c(const char *target_path, scan_mode_t mode)
{
    struct stat sb;

    if (target_path == NULL)
        return NULL;

    scan_data_t *data = calloc(1, sizeof(scan_data_t));
    if (data == NULL)
        return NULL;

    data->target_path = strdup(target_path);

    int ret = (mode == SCAN_FOLLOW) ? stat(target_path, &sb) : lstat(target_path, &sb);
    if (ret != 0)
        return NULL;  /* FUITE: data et target_path jamais libérés! */

    /* ... */
    return data;
}
/* Pourquoi faux: Fuite mémoire à chaque fichier inexistant */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT D (Logic) : Inversion stat/lstat
 * ═══════════════════════════════════════════════════════════════════════════ */
scan_data_t *samus_scan_mutant_d(const char *target_path, scan_mode_t mode)
{
    struct stat sb;

    if (target_path == NULL)
        return NULL;

    /* INVERSÉ: SCAN_FOLLOW devrait utiliser stat(), pas lstat() */
    int ret = (mode == SCAN_FOLLOW) ? lstat(target_path, &sb) : stat(target_path, &sb);

    /* ... */
}
/* Pourquoi faux: SCAN_FOLLOW ne suit plus les symlinks */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT E (Return) : Mauvaise classification des types
 * ═══════════════════════════════════════════════════════════════════════════ */
entity_class_t classify_entity_mutant_e(mode_t mode)
{
    if (S_ISDIR(mode))  return ENTITY_DATAFORM;  /* INVERSÉ! */
    if (S_ISREG(mode))  return ENTITY_HIVE;      /* INVERSÉ! */
    if (S_ISLNK(mode))  return ENTITY_WORMHOLE;
    /* ... */
    return ENTITY_UNKNOWN;
}
/* Pourquoi faux: Fichiers détectés comme répertoires et vice-versa */

/* ═══════════════════════════════════════════════════════════════════════════
 * MUTANT F (Edge) : readlink sans terminaison NULL
 * ═══════════════════════════════════════════════════════════════════════════ */
/* Dans samus_scan, partie symlink */
if (data->entity_class == ENTITY_WORMHOLE)
{
    char link_target[PATH_MAX];
    ssize_t len = readlink(target_path, link_target, PATH_MAX - 1);
    if (len > 0)
    {
        /* MANQUE: link_target[len] = '\0'; */
        data->wormhole_dest = strdup(link_target);  /* Données garbage après */
    }
}
/* Pourquoi faux: readlink ne termine pas la chaîne par '\0' */
```

---

## 🧠 SECTION 5 : COMPRENDRE

### 5.1 Ce que cet exercice enseigne

1. **Structure inode Unix** : Comprendre ce que contient (et ne contient PAS) un inode
2. **Syscall stat()** : Récupérer les métadonnées de fichiers
3. **Types de fichiers** : Les 7 types Unix et comment les détecter
4. **Permissions** : Lecture et formatage des mode bits
5. **Timestamps** : atime, mtime, ctime et leur signification
6. **Gestion mémoire** : Allocation et libération propres

### 5.2 LDA — Traduction Littérale

```
FONCTION samus_scan QUI RETOURNE UN POINTEUR VERS scan_data_t ET PREND EN PARAMÈTRES target_path QUI EST UN POINTEUR VERS CARACTÈRE CONSTANT ET mode QUI EST UN scan_mode_t
DÉBUT FONCTION
    DÉCLARER sb COMME STRUCTURE stat
    DÉCLARER data COMME POINTEUR VERS scan_data_t
    DÉCLARER ret COMME ENTIER

    SI target_path EST ÉGAL À NUL OU target_path[0] EST ÉGAL AU CARACTÈRE NUL ALORS
        AFFECTER SCAN_ERR_INVALID À g_last_error
        RETOURNER NUL
    FIN SI

    SI mode EST ÉGAL À SCAN_FOLLOW ALORS
        AFFECTER stat(target_path, &sb) À ret
    SINON
        AFFECTER lstat(target_path, &sb) À ret
    FIN SI

    SI ret EST DIFFÉRENT DE 0 ALORS
        RETOURNER NUL
    FIN SI

    AFFECTER ALLOUER LA MÉMOIRE DE LA TAILLE D'UN scan_data_t À data
    SI data EST ÉGAL À NUL ALORS
        RETOURNER NUL
    FIN SI

    AFFECTER strdup(target_path) À data->target_path
    AFFECTER sb.st_ino À data->neural_sig
    AFFECTER classify_entity(sb.st_mode) À data->entity_class
    AFFECTER sb.st_mode À data->access_mode
    AFFECTER sb.st_size À data->data_size
    AFFECTER sb.st_nlink À data->symbiotic_count

    RETOURNER data
FIN FONCTION
```

### 5.2.2 Logic Flow

```
ALGORITHME : Scanner une entité fichier
---
1. VALIDER les paramètres d'entrée
   - SI path est NULL ou vide → RETOURNER erreur

2. APPELER stat() ou lstat() selon le mode
   - SCAN_FOLLOW → stat() (suit les symlinks)
   - SCAN_SURFACE → lstat() (ne suit pas)

3. SI échec du stat() :
   - DÉTERMINER le type d'erreur (ENOENT, EACCES, etc.)
   - RETOURNER NULL avec code erreur

4. ALLOUER la structure de données
   - SI échec allocation → RETOURNER NULL

5. REMPLIR les champs depuis struct stat :
   a. Signature neurale (inode number)
   b. Classification (type de fichier)
   c. Protocoles d'accès (permissions)
   d. Propriétaire (UID/GID)
   e. Dimensions (taille)
   f. Marqueurs temporels (timestamps)
   g. Liens symbiotiques (nlink)
   h. Allocation énergie (blocks)

6. SI type == WORMHOLE (symlink) :
   - LIRE la destination avec readlink()

7. RETOURNER les données de scan
```

### 5.3 Visualisation ASCII

```
                    SCAN VISOR SYSTEM
    ┌──────────────────────────────────────────────────┐
    │                                                  │
    │   samus_scan("/etc/passwd", SCAN_FOLLOW)         │
    │        │                                         │
    │        ▼                                         │
    │   ┌─────────────┐                                │
    │   │   stat()    │ ◄── Syscall vers le kernel     │
    │   └──────┬──────┘                                │
    │          │                                       │
    │          ▼                                       │
    │   ┌─────────────────────────────────────┐        │
    │   │         INODE #131073               │        │
    │   ├─────────────────────────────────────┤        │
    │   │ Type:       Regular file (-) ◄──────┼── entity_class │
    │   │ Mode:       0644 (rw-r--r--)  ◄─────┼── access_mode  │
    │   │ Owner:      0 (root)          ◄─────┼── owner_id     │
    │   │ Group:      0 (root)          ◄─────┼── group_id     │
    │   │ Size:       2847 bytes        ◄─────┼── data_size    │
    │   │ atime:      2025-01-04        ◄─────┼── last_access  │
    │   │ mtime:      2024-12-15        ◄─────┼── last_modify  │
    │   │ ctime:      2024-12-15        ◄─────┼── last_change  │
    │   │ nlink:      1                 ◄─────┼── symbiotic_count │
    │   │ blocks:     8                 ◄─────┼── blocks_alloc │
    │   └─────────────────────────────────────┘        │
    │                                                  │
    │   ⚠️ NOTE: Le nom "passwd" n'est PAS ici!        │
    │      Il est dans le répertoire /etc/             │
    │                                                  │
    └──────────────────────────────────────────────────┘
```

```
    stat() vs lstat() - SCAN_FOLLOW vs SCAN_SURFACE

    SCAN_FOLLOW (stat):
    ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
    │  /tmp/link  │ ───► │  symlink    │ ───► │ /etc/passwd │
    └─────────────┘      └─────────────┘      └─────────────┘
                                                    ▲
                                                    │
                                              On scanne ÇA

    SCAN_SURFACE (lstat):
    ┌─────────────┐      ┌─────────────┐
    │  /tmp/link  │ ───► │  symlink    │ ◄── On scanne ÇA
    └─────────────┘      └─────────────┘
                              │
                              ▼
                         /etc/passwd (ignoré)
```

### 5.4 Les pièges en détail

| Piège | Description | Solution |
|-------|-------------|----------|
| **readlink() sans '\0'** | readlink() ne termine PAS la chaîne | Toujours ajouter `buf[len] = '\0'` |
| **Fuite sur erreur** | Oublier de libérer si stat() échoue après malloc | Libérer avant chaque return NULL |
| **stat vs lstat inversés** | Confondre les modes | FOLLOW=stat(), SURFACE=lstat() |
| **Buffer trop petit** | Pas de vérification taille pour permissions | Toujours vérifier size >= 10 |
| **Pas de "creation time"** | Unix n'a pas de timestamp de création! | Ne pas chercher à l'afficher |

### 5.5 Cours Complet

#### L'Architecture du Filesystem Unix

Dans un système de fichiers Unix, les données sont organisées en trois composants principaux :

1. **Le Superblock** : Métadonnées du filesystem lui-même (taille, nombre d'inodes, etc.)
2. **La Table des Inodes** : Tableau de toutes les structures inode
3. **Les Blocs de Données** : Contenu réel des fichiers

#### Structure de l'Inode (struct stat)

```c
struct stat {
    dev_t     st_dev;     /* ID du device contenant le fichier */
    ino_t     st_ino;     /* Numéro d'inode */
    mode_t    st_mode;    /* Type de fichier et permissions */
    nlink_t   st_nlink;   /* Nombre de hard links */
    uid_t     st_uid;     /* UID du propriétaire */
    gid_t     st_gid;     /* GID du groupe */
    dev_t     st_rdev;    /* ID device (si special file) */
    off_t     st_size;    /* Taille totale en bytes */
    blksize_t st_blksize; /* Taille de bloc pour I/O */
    blkcnt_t  st_blocks;  /* Nombre de blocs 512B alloués */
    time_t    st_atime;   /* Dernier accès */
    time_t    st_mtime;   /* Dernière modification */
    time_t    st_ctime;   /* Dernier changement d'état */
};
```

#### Les Mode Bits (st_mode)

```
  16 bits de st_mode:
  ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
  │ 15 │ 14 │ 13 │ 12 │ 11 │ 10 │  9 │  8 │  7 │  6 │  5 │  4 │  3 │  2 │  1 │  0 │
  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
  │         TYPE        │SUID│SGID│STKY│  USER   │  GROUP  │  OTHER  │
  └──────────────────────┴────┴────┴────┴─────────┴─────────┴─────────┘
                                        │ r│ w│ x│ r│ w│ x│ r│ w│ x│
```

### 5.6 Normes avec explications pédagogiques

```
┌─────────────────────────────────────────────────────────────────┐
│ ❌ HORS NORME                                                   │
├─────────────────────────────────────────────────────────────────┤
│ struct stat sb; stat(path, &sb); // Pas de vérification        │
├─────────────────────────────────────────────────────────────────┤
│ ✅ CONFORME                                                     │
├─────────────────────────────────────────────────────────────────┤
│ struct stat sb;                                                 │
│ if (stat(path, &sb) != 0)                                       │
│     return NULL;                                                │
├─────────────────────────────────────────────────────────────────┤
│ 📖 POURQUOI ?                                                   │
│                                                                 │
│ • stat() peut échouer (fichier inexistant, permissions)         │
│ • Ignorer le retour = comportement indéfini                     │
│ • Toujours vérifier les syscalls !                              │
└─────────────────────────────────────────────────────────────────┘
```

### 5.7 Simulation avec trace d'exécution

```
┌───────┬─────────────────────────────────────────┬─────────────────────┬─────────────────────┐
│ Étape │ Instruction                             │ Résultat            │ Explication         │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   1   │ samus_scan("/etc/passwd", SCAN_FOLLOW)  │ Appel fonction      │ Début du scan       │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   2   │ path == NULL ?                          │ FAUX                │ Path valide         │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   3   │ stat("/etc/passwd", &sb)                │ ret = 0             │ Succès syscall      │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   4   │ ret != 0 ?                              │ FAUX                │ Continue            │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   5   │ calloc(1, sizeof(scan_data_t))          │ data = 0x...        │ Allocation OK       │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   6   │ strdup("/etc/passwd")                   │ target_path set     │ Copie chemin        │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   7   │ data->neural_sig = sb.st_ino            │ 131073              │ Inode copié         │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   8   │ classify_entity(sb.st_mode)             │ ENTITY_DATAFORM     │ C'est un fichier    │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│   9   │ data->data_size = sb.st_size            │ 2847                │ Taille copiée       │
├───────┼─────────────────────────────────────────┼─────────────────────┼─────────────────────┤
│  10   │ return data                             │ Pointeur valide     │ Scan terminé !      │
└───────┴─────────────────────────────────────────┴─────────────────────┴─────────────────────┘
```

### 5.8 Mnémotechniques

#### 🎮 MEME : "Scanning..." — Le Scan Visor de Metroid

![Scan Visor](metroid_scan.jpg)

Dans Metroid Prime, quand Samus scanne un ennemi, elle obtient :
- **Classification** → Type de fichier (regular, directory...)
- **Points faibles** → Permissions (où peut-on accéder)
- **Historique** → Timestamps (quand a-t-il été vu/modifié)

```c
// 🎮 Comme Samus, on scanne AVANT d'agir !
scan_data_t *data = samus_scan(target, SCAN_FOLLOW);
if (data == NULL) {
    // "Scan failed. Unable to gather data."
    return;
}
// Maintenant on peut agir en toute sécurité
```

#### 🔮 MEME : "Le nom n'est pas dans l'inode"

Imagine un annuaire téléphonique :
- **Le répertoire** = L'annuaire (associe noms → numéros)
- **L'inode** = La fiche d'abonné (infos, mais PAS le nom)

C'est pourquoi deux noms (hard links) peuvent pointer vers le même inode !

#### ⏰ MEME : "atime, mtime, ctime — Les 3 Mousquetaires du temps"

- **atime** = "Access" → Athos (le premier à lire)
- **mtime** = "Modify" → Porthos (il modifie le contenu)
- **ctime** = "Change" → Aramis (il change les métadonnées)

Et d'Artagnan ? C'est le "creation time" qui **N'EXISTE PAS sous Unix** !

### 5.9 Applications pratiques

| Commande | Ce qu'elle utilise | Notre équivalent |
|----------|-------------------|------------------|
| `ls -l` | stat() + permissions | display_ls_format() |
| `ls -i` | st_ino | data->neural_sig |
| `stat file` | Toutes les infos | display_scan_report() |
| `file type` | st_mode + S_ISXXX | classify_entity() |
| `find -type f` | S_ISREG() | entity_class == ENTITY_DATAFORM |

---

## ⚠️ SECTION 6 : PIÈGES — RÉCAPITULATIF

| # | Piège | Fréquence | Impact | Détection |
|---|-------|-----------|--------|-----------|
| 1 | NULL sans vérification | Très fréquent | Crash | Test NULL input |
| 2 | Fuite mémoire sur erreur | Fréquent | Leak | Valgrind |
| 3 | readlink sans '\0' | Fréquent | Données garbage | Test symlink dest |
| 4 | stat/lstat inversés | Moyen | Mauvais résultats | Test symlink modes |
| 5 | Buffer overflow perms | Moyen | Crash/Corruption | ASAN |
| 6 | Oublier bits spéciaux | Rare | Permissions fausses | Test setuid/sticky |

---

## 📝 SECTION 7 : QCM

### Q1. Que retourne stat() en cas de succès ?
- A) 1
- B) 0
- C) Le numéro d'inode
- D) Un pointeur vers struct stat
- E) La taille du fichier

**Réponse : B**

### Q2. Quelle est la différence entre stat() et lstat() ?
- A) stat() est plus rapide
- B) lstat() ne fonctionne que sur Linux
- C) stat() suit les symlinks, lstat() non
- D) Aucune différence
- E) lstat() retourne plus d'informations

**Réponse : C**

### Q3. Qu'est-ce que l'inode NE contient PAS ?
- A) Les permissions
- B) Le nom du fichier
- C) La taille
- D) Les timestamps
- E) Le numéro d'inode

**Réponse : B**

### Q4. Quel caractère représente un répertoire dans ls -l ?
- A) -
- B) r
- C) d
- D) l
- E) f

**Réponse : C**

### Q5. Combien de types de fichiers existe-t-il sous Unix ?
- A) 3
- B) 5
- C) 7
- D) 10
- E) 12

**Réponse : C**

### Q6. Que signifie st_nlink ?
- A) Nombre de symlinks
- B) Nombre de hard links
- C) Nombre de blocs
- D) Nombre de bytes
- E) Numéro de ligne

**Réponse : B**

### Q7. Quelle macro teste si c'est un fichier régulier ?
- A) S_ISFILE()
- B) S_ISREG()
- C) S_ISNORMAL()
- D) IS_REGULAR()
- E) FILE_TEST()

**Réponse : B**

### Q8. Que retourne readlink() ?
- A) Une chaîne terminée par '\0'
- B) Le nombre de bytes lus (sans '\0')
- C) 0 en cas de succès
- D) Un pointeur vers la cible
- E) Le numéro d'inode de la cible

**Réponse : B**

### Q9. Unix a-t-il un timestamp de création de fichier ?
- A) Oui, c'est st_btime
- B) Oui, c'est st_ctime
- C) Oui, c'est st_crtime
- D) Non, ctime = "change time"
- E) Oui, mais seulement sur ext4

**Réponse : D**

### Q10. Quelle est la valeur minimale de st_nlink pour un répertoire ?
- A) 0
- B) 1
- C) 2
- D) 3
- E) Dépend du filesystem

**Réponse : C** (lui-même "." + entrée dans le parent)

---

## 📊 SECTION 8 : RÉCAPITULATIF

| Critère | Valeur |
|---------|--------|
| **Exercice** | 2.3.0-a : samus_scan |
| **Thème** | Metroid Prime - Scan Visor |
| **Difficulté** | ★★★★☆☆☆☆☆☆ (4/10) |
| **Durée** | 4 heures |
| **XP Base** | 150 |
| **XP Bonus** | ×2 (300 total) |
| **Concepts clés** | stat(), inode, permissions, timestamps |
| **Prérequis** | Pointeurs, structures, syscalls basiques |
| **Tests** | 23 tests, Valgrind obligatoire |
| **Mutants** | 6 solutions buggées à détecter |

---

## 📦 SECTION 9 : DEPLOYMENT PACK

```json
{
  "deploy": {
    "hackbrain_version": "5.5.2",
    "engine_version": "v22.1",
    "exercise_slug": "2.3.0-a-samus-scan",
    "generated_at": "2025-01-11T12:00:00",

    "metadata": {
      "exercise_id": "2.3.0-a",
      "exercise_name": "samus_scan",
      "module": "2.3.0",
      "module_name": "File System Inspector",
      "concept": "a",
      "concept_name": "Analyse via stat()",
      "type": "complet",
      "tier": 1,
      "tier_info": "Concept isolé",
      "phase": 2,
      "difficulty": 4,
      "difficulty_stars": "★★★★☆☆☆☆☆☆",
      "language": "c",
      "duration_minutes": 240,
      "xp_base": 150,
      "xp_bonus_multiplier": 2,
      "bonus_tier": "STANDARD",
      "bonus_icon": "⚡",
      "complexity_time": "T1 O(1)",
      "complexity_space": "S1 O(1)",
      "prerequisites": ["pointeurs", "structures", "syscalls"],
      "domains": ["FS", "Encodage"],
      "domains_bonus": [],
      "tags": ["filesystem", "stat", "inode", "permissions", "metroid"],
      "meme_reference": "Metroid Prime - Scan Visor"
    },

    "files": {
      "spec.json": "/* Section 4.9 */",
      "references/ref_samus_scan.c": "/* Section 4.3 */",
      "references/ref_bonus.c": "/* Section 4.6 */",
      "alternatives/alt_fstat.c": "/* Section 4.4 */",
      "mutants/mutant_a_boundary.c": "/* Buffer overflow */",
      "mutants/mutant_b_safety.c": "/* NULL crash */",
      "mutants/mutant_c_resource.c": "/* Memory leak */",
      "mutants/mutant_d_logic.c": "/* stat/lstat inversé */",
      "mutants/mutant_e_return.c": "/* Classification inversée */",
      "mutants/mutant_f_edge.c": "/* readlink sans NULL */",
      "tests/main.c": "/* Section 4.2 */"
    },

    "validation": {
      "expected_pass": [
        "references/ref_samus_scan.c",
        "references/ref_bonus.c",
        "alternatives/alt_fstat.c"
      ],
      "expected_fail": [
        "mutants/mutant_a_boundary.c",
        "mutants/mutant_b_safety.c",
        "mutants/mutant_c_resource.c",
        "mutants/mutant_d_logic.c",
        "mutants/mutant_e_return.c",
        "mutants/mutant_f_edge.c"
      ]
    }
  }
}
```

---

*HACKBRAIN v5.5.2 — Exercice 2.3.0-a : samus_scan*
*"Scanning... Data acquired."*
*Thème : Metroid Prime — Scan Visor*
