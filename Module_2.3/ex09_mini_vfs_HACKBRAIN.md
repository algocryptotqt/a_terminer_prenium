# Exercice 2.3.18-synth : monsters_inc_vfs

**Module :**
2.3.18-19 — Virtual File System + Mount/Unmount

**Concept :**
synth — Synthèse VFS complète (21 concepts)

**Difficulté :**
★★★★★★★★☆☆ (8/10)

**Type :**
code

**Tiers :**
3 — Synthèse (concepts 2.3.18.a-j + 2.3.19.a-k)

**Langage :**
C (C17)

**Prérequis :**
- Pointeurs de fonctions
- Structures de données (hash tables, listes chaînées)
- Gestion mémoire dynamique
- Concepts filesystem de base

**Domaines :**
FS, Struct, Mem

**Durée estimée :**
720 min (12h)

**XP Base :**
800

**Complexité :**
T3 O(n) pour lookup × S3 O(n) pour caches

---

## 📐 SECTION 1 : PROTOTYPE & CONSIGNE

### 1.1 Obligations

**Fichiers à rendre :**
```
ex09/
├── monsters_inc.h
├── door_station.c      (VFS core)
├── scare_floor.c       (Mount operations)
├── scream_cache.c      (Caches)
├── door_walk.c         (Path lookup)
├── laugh_floor.c       (Exemple FS)
└── Makefile
```

**Fonctions autorisées :**
`malloc`, `free`, `memset`, `memcpy`, `strcmp`, `strncpy`, `strlen`, `strdup`, `time`

**Fonctions interdites :**
`mount`, `umount`, `open`, `close`, `read`, `write` (syscalls réels)

---

### 1.2 Consigne

**🎬 CONTEXTE : MONSTERS, INC. — "We Scare Because We Care"**

Bienvenue chez **Monstres & Cie** ! Tu es le nouvel ingénieur de la **Door Station** (station des portes), le système révolutionnaire qui connecte Monstropolis aux chambres d'enfants du monde entier.

Chaque **porte** sur le **Scare Floor** (plateau de travail) est un **point de montage** vers un monde différent. La **Door Vault** (chambre forte des portes) contient des millions de portes, chacune menant à une chambre unique.

Ton travail : implémenter le système qui gère ces connexions !

**L'analogie parfaite :**

| Monsters, Inc. | VFS Linux |
|----------------|-----------|
| Door Station | VFS Layer |
| Porte (Door) | Mount Point |
| Chambre d'enfant | Filesystem monté |
| Scare Floor | Mount Table |
| Door Vault | FS Registry |
| Monstre (Sulley) | Processus |
| Canister (bidon) | Fichier ouvert |
| CDA (Child Detection Agency) | Sécurité/Permissions |
| Door Shredder | Unmount |
| Boo | User Process |

---

### 1.2.2 Consigne Académique

Implémenter une couche **Virtual File System (VFS)** complète qui :

1. **Abstrait** plusieurs types de filesystems derrière une interface unifiée
2. Gère les **objets VFS** : superblock, inode, dentry, file
3. Implémente les **structures d'opérations** (function pointers)
4. Maintient des **caches** (dentry cache, inode cache)
5. Résout les **chemins** (path lookup / namei)
6. Gère le **montage/démontage** avec flags et namespaces

---

### 1.3 Prototypes

```c
/* ═══════════════════════════════════════════════════════════════════
 *  MONSTERS, INC. VFS - "Put That Thing Back Where It Came From"
 * ═══════════════════════════════════════════════════════════════════ */

#ifndef MONSTERS_INC_H
#define MONSTERS_INC_H

#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>
#include <time.h>
#include <sys/types.h>

/* Forward declarations - Les personnages */
struct door_station;      /* VFS superblock */
struct monster_card;      /* VFS inode */
struct room_entry;        /* VFS dentry */
struct scream_canister;   /* VFS file */

/* ───────────────────────────────────────────────────────────────────
 * 2.3.18.d: station_ops - Operations niveau Door Station (super_ops)
 * Comme les ingénieurs qui maintiennent la station
 * ─────────────────────────────────────────────────────────────────── */
typedef struct {
    struct monster_card* (*hire_monster)(struct door_station *station);
    void (*fire_monster)(struct monster_card *card);
    int (*update_monster)(struct monster_card *card);
    int (*sync_station)(struct door_station *station);
    int (*station_stats)(struct door_station *station, void *buf);
} station_ops_t;

/* ───────────────────────────────────────────────────────────────────
 * 2.3.18.e: monster_ops - Operations sur Monster Cards (inode_ops)
 * Ce que chaque monstre peut faire
 * ─────────────────────────────────────────────────────────────────── */
typedef struct {
    struct room_entry* (*find_room)(struct monster_card *dir, const char *name);
    int (*create_room)(struct monster_card *dir, const char *name, mode_t mode);
    int (*create_closet)(struct monster_card *dir, const char *name, mode_t mode);
    int (*destroy_room)(struct monster_card *dir, const char *name);
    int (*destroy_closet)(struct monster_card *dir, const char *name);
    int (*link_room)(struct room_entry *old, struct monster_card *dir, const char *name);
    int (*secret_passage)(struct monster_card *dir, const char *name, const char *target);
    int (*relocate_room)(struct monster_card *old_dir, const char *old_name,
                         struct monster_card *new_dir, const char *new_name);
} monster_ops_t;

/* ───────────────────────────────────────────────────────────────────
 * 2.3.18.f: canister_ops - Operations sur Canisters (file_ops)
 * Comment on remplit les bidons de cris/rires
 * ─────────────────────────────────────────────────────────────────── */
typedef struct {
    int (*open_canister)(struct monster_card *card, struct scream_canister *can);
    int (*close_canister)(struct monster_card *card, struct scream_canister *can);
    ssize_t (*collect_scream)(struct scream_canister *can, char *buf,
                              size_t count, off_t *offset);
    ssize_t (*store_laugh)(struct scream_canister *can, const char *buf,
                           size_t count, off_t *offset);
    off_t (*rewind_canister)(struct scream_canister *can, off_t offset, int whence);
    int (*list_contents)(struct scream_canister *can, void *dirent,
                         int (*filldir)(void*, const char*, int, off_t, ino_t, unsigned));
    int (*flush_canister)(struct scream_canister *can);
} canister_ops_t;

/* ───────────────────────────────────────────────────────────────────
 * 2.3.18.b: VFS Objects - Les structures de Monstropolis
 * ─────────────────────────────────────────────────────────────────── */

/* Door Station = Superblock (le QG) */
typedef struct door_station {
    uint32_t station_id;
    uint32_t door_size;
    struct monster_card *root_monster;
    station_ops_t *s_ops;
    void *station_data;
    char station_type[32];
} door_station_t;

/* Monster Card = Inode (identité de chaque monstre/fichier) */
typedef struct monster_card {
    ino_t card_id;
    mode_t clearance;           /* Niveau d'habilitation */
    uid_t owner_id;
    gid_t team_id;
    size_t scare_quota;         /* Taille */
    time_t last_scare;          /* atime */
    time_t last_update;         /* mtime */
    time_t creation_date;       /* ctime */
    uint32_t partner_count;     /* nlink */
    door_station_t *station;
    monster_ops_t *m_ops;
    canister_ops_t *c_ops;
    void *monster_data;
} monster_card_t;

/* Room Entry = Dentry (entrée dans le répertoire des chambres) */
typedef struct room_entry {
    char child_name[256];
    monster_card_t *card;
    struct room_entry *parent_room;
    struct room_entry *sub_rooms;
    struct room_entry *next_room;
    uint32_t visitors;          /* ref_count */
} room_entry_t;

/* Scream Canister = File (fichier ouvert) */
typedef struct scream_canister {
    room_entry_t *room;
    monster_card_t *card;
    off_t fill_level;           /* offset */
    int access_flags;
    canister_ops_t *c_ops;
} scream_canister_t;

/* ───────────────────────────────────────────────────────────────────
 * 2.3.18.g-h: Caches - Les fichiers de Roz
 * "I'm watching you, Wazowski. Always watching."
 * ─────────────────────────────────────────────────────────────────── */

/* Scream Cache = Dentry Cache (Roz's paperwork) */
typedef struct {
    room_entry_t **filing_cabinets;
    uint32_t cabinet_count;
    uint32_t total_entries;
} scream_cache_t;

/* Gallery Cache = Inode Cache (Wall of Fame) */
typedef struct {
    monster_card_t **wall_of_fame;
    uint32_t frame_count;
    uint32_t total_monsters;
} gallery_cache_t;

/* ───────────────────────────────────────────────────────────────────
 * 2.3.19: Scare Floor - Mount structures
 * Le plateau où les portes sont actives
 * ─────────────────────────────────────────────────────────────────── */
typedef struct active_door {
    door_station_t *station;
    room_entry_t *door_location;
    uint32_t door_flags;
    struct active_door *next_door;
    char source_world[256];
} active_door_t;

/* Door Flags (2.3.19.d) */
#define DOOR_READONLY   0x01    /* Chambre en quarantaine */
#define DOOR_NOEXEC     0x02    /* Pas de jeux autorisés */
#define DOOR_NOSUID     0x04    /* Pas de déguisement */
#define DOOR_BIND       0x08    /* Porte secrète de Randall */
#define DOOR_LAZY       0x10    /* Fermeture différée */

/* ═══════════════════════════════════════════════════════════════════
 *  API MONSTERS, INC. VFS
 * ═══════════════════════════════════════════════════════════════════ */

/* Lifecycle - Ouverture/Fermeture de l'usine */
int monsters_inc_open(void);                    /* 2.3.18.a */
void monsters_inc_close(void);

/* Door Vault Registration (2.3.18.c) - Enregistrer types de portes */
int door_vault_register(const char *door_type,
                        station_ops_t *s_ops,
                        monster_ops_t *m_ops,
                        canister_ops_t *c_ops);
int door_vault_unregister(const char *door_type);

/* Scare Floor Operations (2.3.19) - Gestion des portes actives */
int scare_floor_attach(const char *source, const char *target,
                       const char *door_type, uint32_t flags);     /* a,b,c,d */
int scare_floor_detach(const char *target);                        /* g */
int scare_floor_detach_lazy(const char *target);                   /* i */
int scare_floor_bind(const char *source, const char *target);      /* j */
int scare_floor_status(char *buf, size_t size);                    /* e */
int scare_floor_parse_config(const char *path);                    /* f */
bool scare_floor_is_busy(const char *target);                      /* h */

/* Door Walking (2.3.18.i) - Trouver une chambre */
room_entry_t *door_walk(const char *path);
monster_card_t *door_walk_to_monster(const char *path);

/* Cross-Door Navigation (2.3.18.j) */
active_door_t *door_get_active(room_entry_t *room);
bool door_is_portal(room_entry_t *room);

/* File Operations via VFS - Sulley au travail */
int sulley_open(const char *path, int flags, scream_canister_t **can);
int sulley_close(scream_canister_t *can);
ssize_t sulley_read(scream_canister_t *can, void *buf, size_t count);
ssize_t sulley_write(scream_canister_t *can, const void *buf, size_t count);
int sulley_mkdir(const char *path, mode_t mode);
int sulley_unlink(const char *path);

/* Cache Operations (2.3.18.g-h) - Roz's filing system */
void roz_init_screams(scream_cache_t *cache, uint32_t size);
room_entry_t *roz_find_room(scream_cache_t *cache, room_entry_t *parent,
                            const char *name);
void roz_file_room(scream_cache_t *cache, room_entry_t *room);
void roz_shred_room(scream_cache_t *cache, room_entry_t *room);

void gallery_init(gallery_cache_t *cache, uint32_t size);
monster_card_t *gallery_find(gallery_cache_t *cache, ino_t card_id);
void gallery_add(gallery_cache_t *cache, monster_card_t *card);

/* Door Namespaces (2.3.19.k) - Différentes vues par équipe */
int boo_create_world(void);
int boo_enter_world(int world_id);

#endif /* MONSTERS_INC_H */
```

---

## 💡 SECTION 2 : LE SAVIEZ-VOUS ?

### 2.1 L'histoire de VFS

Le **Virtual File System** a été introduit par Sun Microsystems dans SunOS 2.0 (1985) pour permettre à un même système d'accéder simultanément à des filesystems locaux (UFS) et distants (NFS).

Linux a adopté ce concept dès le début. Aujourd'hui, le VFS Linux supporte plus de 50 types de filesystems différents, tous accessibles via la même API (`open`, `read`, `write`...).

### 2.2 Les portes de Monsters, Inc.

Dans le film Pixar (2001), l'usine Monsters, Inc. utilise des **millions de portes** stockées dans une immense chambre forte. Chaque porte connecte Monstropolis à une chambre d'enfant spécifique.

Cette métaphore est PARFAITE pour le VFS :
- Les portes = mount points (connexions vers différents "mondes")
- La station = VFS layer (gère toutes les connexions)
- Les monstres = processus (traversent les portes pour accéder aux ressources)

### 2.3 Pourquoi Roz ?

Roz, la secrétaire terrifiante qui surveille toujours Mike Wazowski, représente les **caches**. Elle garde une trace de TOUT : qui a traversé quelle porte, quand, combien de fois...

> "I'm watching you, Wazowski. Always watching."

Les caches VFS (dentry cache, inode cache) font exactement ça : ils mémorisent les résolutions de chemins et les métadonnées pour éviter de redemander au filesystem.

---

### 2.5 DANS LA VRAIE VIE

| Métier | Utilisation VFS |
|--------|-----------------|
| **Kernel Developer** | Implémenter de nouveaux filesystems via l'interface VFS |
| **SysAdmin** | Monter/démonter des filesystems, gérer fstab |
| **Container Engineer** | Mount namespaces pour l'isolation (Docker, K8s) |
| **Storage Engineer** | Bind mounts, overlay filesystems |
| **Security Engineer** | Flags nosuid/noexec pour hardening |

---

## 🖥️ SECTION 3 : EXEMPLE D'UTILISATION

### 3.0 Session bash

```bash
$ ls
monsters_inc.h  door_station.c  scare_floor.c  scream_cache.c  door_walk.c  laugh_floor.c  main.c  Makefile

$ make
gcc -Wall -Wextra -Werror -c door_station.c -o door_station.o
gcc -Wall -Wextra -Werror -c scare_floor.c -o scare_floor.o
gcc -Wall -Wextra -Werror -c scream_cache.c -o scream_cache.o
gcc -Wall -Wextra -Werror -c door_walk.c -o door_walk.o
gcc -Wall -Wextra -Werror -c laugh_floor.c -o laugh_floor.o
gcc -Wall -Wextra -Werror -c main.c -o main.o
gcc -o monsters_test *.o

$ ./monsters_test
[MONSTERS INC] Factory opening...
[DOOR VAULT] Registering 'laughfs'... OK
[SCARE FLOOR] Attaching door to /mnt/boo... OK
[DOOR WALK] Resolving /mnt/boo/teddy.txt... Found!
[SULLEY] Opening canister... OK
[SULLEY] Storing laugh: "Kitty!"... 6 bytes
[SULLEY] Closing canister... OK
[SCARE FLOOR] Status:
  laughfs on /mnt/boo (NOEXEC)
[SCARE FLOOR] Detaching /mnt/boo... OK
[MONSTERS INC] Factory closing. We scare because we care!

All tests passed! Boo goes home safely.
```

---
