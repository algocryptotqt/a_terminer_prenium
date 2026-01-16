# [Module 2.3] - Exercise 04: Permission Manager

## Metadonnees

```yaml
module: "2.3 - File Systems"
exercise: "ex04"
title: "Permission Manager"
difficulty: intermediaire
estimated_time: "6 heures"
prerequisite_exercises: ["ex00", "ex01", "ex03"]
concepts_requis: ["file permissions", "system calls", "user/group management", "bitwise operations"]
concepts_couverts: ["2.3.7 File Permissions"]
score_qualite: 97
```

---

## Concepts Couverts

Liste exhaustive des concepts abordes dans cet exercice avec references au curriculum:

### Section 2.3.7 - File Permissions

| Ref | Concept | Description |
|-----|---------|-------------|
| **2.3.7.a** | Permission bits: rwxrwxrwx | Les 9 bits de permission organises en triplets pour user, group et other |
| **2.3.7.b** | User/Group/Other: Three categories | Les trois categories d'utilisateurs auxquelles s'appliquent les permissions |
| **2.3.7.c** | Read: View contents | Permission de lecture permettant de voir le contenu d'un fichier ou lister un repertoire |
| **2.3.7.d** | Write: Modify contents | Permission d'ecriture permettant de modifier un fichier ou creer/supprimer des entrees dans un repertoire |
| **2.3.7.e** | Execute: Run file / traverse directory | Permission d'execution pour lancer un programme ou traverser un repertoire |
| **2.3.7.f** | Octal notation: 755, 644 | Representation octale des permissions (chaque chiffre = 3 bits) |
| **2.3.7.g** | chmod(): Change permissions | Syscall pour modifier les permissions d'un fichier via son chemin |
| **2.3.7.h** | fchmod(): Change by fd | Syscall pour modifier les permissions via un file descriptor ouvert |
| **2.3.7.i** | chown(): Change owner | Syscall pour changer le proprietaire (user et/ou group) d'un fichier |
| **2.3.7.j** | umask: Default permission mask | Masque de creation par defaut qui restreint les permissions des nouveaux fichiers |
| **2.3.7.k** | Setuid bit: Run as owner | Bit special permettant d'executer un programme avec les droits du proprietaire |
| **2.3.7.l** | Setgid bit: Run as group | Bit special permettant d'executer avec les droits du groupe ou heriter du groupe parent pour les repertoires |
| **2.3.7.m** | Sticky bit: Restrict deletion | Bit special restreignant la suppression aux seuls proprietaires dans un repertoire partage |

### Objectifs Pedagogiques

A la fin de cet exercice, vous devriez etre capable de:

1. Comprendre et manipuler les 9 bits de permission standard (rwxrwxrwx)
2. Distinguer et appliquer les permissions pour User, Group et Other
3. Maitriser la notation octale et symbolique des permissions
4. Utiliser chmod() et fchmod() pour modifier les permissions
5. Utiliser chown() pour changer le proprietaire d'un fichier
6. Comprendre et configurer le umask pour controler les permissions par defaut
7. Implementer et comprendre les bits speciaux (setuid, setgid, sticky)
8. Construire un outil complet d'analyse et de modification des permissions

---

## Contexte Theorique

### Architecture des Permissions Unix

Le systeme de permissions Unix est fonde sur une architecture elegante utilisant 12 bits pour controler l'acces aux fichiers:

```
    Bits speciaux     User        Group       Other
    +---+---+---+  +---+---+---+---+---+---+---+---+---+
    | S | G | T |  | r | w | x | r | w | x | r | w | x |
    +---+---+---+  +---+---+---+---+---+---+---+---+---+
      4   2   1      4   2   1   4   2   1   4   2   1

    S = Setuid (4000)    r = Read (4)
    G = Setgid (2000)    w = Write (2)
    T = Sticky (1000)    x = Execute (1)
```

**2.3.7.a - Permission bits: rwxrwxrwx**

Les 9 bits de permission de base sont organises en trois triplets identiques, chacun representant les droits d'une categorie d'utilisateurs. Chaque triplet contient:
- **r** (read, valeur 4): Permission de lecture
- **w** (write, valeur 2): Permission d'ecriture
- **x** (execute, valeur 1): Permission d'execution

```c
#include <sys/stat.h>

// Constantes pour les permission bits
// User (owner)
#define S_IRUSR  0400   // r--------
#define S_IWUSR  0200   // -w-------
#define S_IXUSR  0100   // --x------
#define S_IRWXU  0700   // rwx------

// Group
#define S_IRGRP  0040   // ---r-----
#define S_IWGRP  0020   // ----w----
#define S_IXGRP  0010   // -----x---
#define S_IRWXG  0070   // ---rwx---

// Other
#define S_IROTH  0004   // ------r--
#define S_IWOTH  0002   // -------w-
#define S_IXOTH  0001   // --------x
#define S_IRWXO  0007   // ------rwx
```

**2.3.7.b - User/Group/Other: Three categories**

Unix categorise les utilisateurs en trois groupes pour l'evaluation des permissions:

1. **User (Owner)**: Le proprietaire du fichier. Identifie par l'UID stocke dans l'inode.
2. **Group**: Les membres du groupe proprietaire. Identifie par le GID stocke dans l'inode.
3. **Other**: Tous les autres utilisateurs du systeme.

L'evaluation des permissions suit un ordre strict:
```c
// Pseudo-code de verification d'acces du noyau
int check_access(struct inode *inode, int requested_mode) {
    uid_t uid = getuid();
    gid_t gid = getgid();

    // 1. Si root, acces total (sauf execute sans aucun x)
    if (uid == 0) {
        if (requested_mode & X_OK) {
            return (inode->mode & (S_IXUSR | S_IXGRP | S_IXOTH)) ? 0 : -1;
        }
        return 0;
    }

    // 2. Si proprietaire, utiliser les bits User
    if (uid == inode->uid) {
        return check_bits(inode->mode >> 6, requested_mode);
    }

    // 3. Si membre du groupe, utiliser les bits Group
    if (gid == inode->gid || is_supplementary_group(gid, inode->gid)) {
        return check_bits(inode->mode >> 3, requested_mode);
    }

    // 4. Sinon, utiliser les bits Other
    return check_bits(inode->mode, requested_mode);
}
```

**2.3.7.c - Read: View contents**

La permission de lecture (`r`) a des significations differentes selon le type de fichier:

| Type | Signification de Read |
|------|----------------------|
| Fichier regulier | Lire le contenu du fichier (open avec O_RDONLY, read()) |
| Repertoire | Lister les noms des entrees (readdir(), ls) |
| Lien symbolique | Non applicable (les liens ont toujours rwxrwxrwx) |

```c
// Sans permission read sur un fichier
int fd = open("secret.txt", O_RDONLY);
// fd == -1, errno == EACCES

// Sans permission read sur un repertoire
DIR *d = opendir("/private/");
// d == NULL, errno == EACCES
```

**2.3.7.d - Write: Modify contents**

La permission d'ecriture (`w`) permet:

| Type | Signification de Write |
|------|------------------------|
| Fichier regulier | Modifier, tronquer ou ecraser le contenu |
| Repertoire | Creer, supprimer ou renommer des entrees |

```c
// Attention: pour supprimer un fichier, il faut w sur le REPERTOIRE, pas le fichier!
// Donc pour supprimer /home/user/file.txt:
// - Besoin de w+x sur /home/user/
// - Les permissions de file.txt n'importent pas pour la suppression

unlink("/home/user/file.txt");  // Verifie w sur /home/user/, pas sur file.txt
```

**2.3.7.e - Execute: Run file / traverse directory**

La permission d'execution (`x`) a une semantique differente:

| Type | Signification de Execute |
|------|--------------------------|
| Fichier regulier | Executer comme programme (exec*()) |
| Repertoire | Traverser le repertoire (acceder aux fichiers a l'interieur) |

```c
// Pour un repertoire, x permet de "traverser"
// Exemple: pour acceder a /home/user/docs/file.txt
// - Besoin de x sur /home
// - Besoin de x sur /home/user
// - Besoin de x sur /home/user/docs
// - Besoin de r (ou w selon operation) sur file.txt

// Cas interessant: un repertoire avec r mais sans x
// mkdir test_dir && chmod 400 test_dir
// ls test_dir        -> Peut lister les NOMS
// cat test_dir/file  -> ECHEC! Impossible de traverser
```

**2.3.7.f - Octal notation: 755, 644**

La notation octale encode les permissions en 3 ou 4 chiffres octaux:

```
Position:    [special] user group other
Valeur:      [4/2/1]   4/2/1 4/2/1  4/2/1

Exemples courants:
  755 = rwxr-xr-x  (executables, repertoires)
  644 = rw-r--r-- (fichiers normaux)
  700 = rwx------  (prive)
  777 = rwxrwxrwx  (acces total - dangereux!)

Avec bits speciaux:
 4755 = rwsr-xr-x  (setuid + 755)
 2755 = rwxr-sr-x  (setgid + 755)
 1777 = rwxrwxrwt  (sticky + 777, comme /tmp)
```

```c
// Conversion octale en C
mode_t mode = 0755;  // Attention: le prefixe 0 indique l'octal!

// Decomposition
int user  = (mode >> 6) & 07;  // 7 = rwx
int group = (mode >> 3) & 07;  // 5 = r-x
int other = mode & 07;         // 5 = r-x

// Bits speciaux
int special = (mode >> 9) & 07;
int setuid  = special & 04;    // 4000
int setgid  = special & 02;    // 2000
int sticky  = special & 01;    // 1000
```

**2.3.7.g - chmod(): Change permissions**

`chmod()` modifie les permissions d'un fichier via son chemin:

```c
#include <sys/stat.h>

int chmod(const char *pathname, mode_t mode);

// Exemples d'utilisation
chmod("script.sh", 0755);           // rwxr-xr-x
chmod("data.txt", S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);  // rw-r--r--

// Modifier relativement aux permissions existantes
struct stat st;
stat("file.txt", &st);
chmod("file.txt", st.st_mode | S_IXUSR);  // Ajouter x pour user

// Retour: 0 si succes, -1 si erreur
// Erreurs courantes:
//   ENOENT: fichier inexistant
//   EACCES: pas proprietaire (sauf root)
//   EPERM: tentative de modifier setuid/setgid sans droits
```

**2.3.7.h - fchmod(): Change by fd**

`fchmod()` modifie les permissions via un file descriptor ouvert:

```c
#include <sys/stat.h>

int fchmod(int fd, mode_t mode);

// Avantages par rapport a chmod():
// 1. Atomicite: pas de race condition TOCTOU
// 2. Performance: pas besoin de resoudre le chemin
// 3. Securite: opere sur le fichier deja ouvert

int fd = open("sensitive.txt", O_RDWR);
if (fd >= 0) {
    // Modifier les permissions du fichier qu'on a REELLEMENT ouvert
    fchmod(fd, 0600);  // rw-------

    // ... operations sur le fichier ...

    close(fd);
}
```

**2.3.7.i - chown(): Change owner**

`chown()` modifie le proprietaire et/ou le groupe d'un fichier:

```c
#include <unistd.h>

int chown(const char *pathname, uid_t owner, gid_t group);
int fchown(int fd, uid_t owner, gid_t group);
int lchown(const char *pathname, uid_t owner, gid_t group);  // N'suit pas les symlinks

// Utiliser -1 pour ne pas modifier un des champs
chown("file.txt", 1000, -1);   // Change seulement le proprietaire
chown("file.txt", -1, 100);    // Change seulement le groupe
chown("file.txt", 1000, 100);  // Change les deux

// ATTENTION: Seul root peut changer le proprietaire!
// Un utilisateur normal peut seulement:
// - Changer le groupe vers un groupe dont il est membre
// - Et seulement s'il est proprietaire du fichier

// Securite: chown() efface les bits setuid/setgid!
// Ceci empeche les attaques d'elevation de privileges
struct stat st;
stat("setuid_prog", &st);
printf("Mode avant: %o\n", st.st_mode);  // 4755
chown("setuid_prog", 1000, 1000);
stat("setuid_prog", &st);
printf("Mode apres: %o\n", st.st_mode);  // 0755 (setuid efface!)
```

**2.3.7.j - umask: Default permission mask**

Le `umask` est un masque qui restreint les permissions des fichiers nouvellement crees:

```c
#include <sys/stat.h>

mode_t umask(mode_t mask);  // Retourne l'ancien masque

// Fonctionnement:
// permissions_effectives = permissions_demandees & ~umask

// Exemple avec umask = 022 (defaut courant)
// Creation avec mode 0666: 0666 & ~022 = 0666 & 0755 = 0644
// Creation avec mode 0777: 0777 & ~022 = 0777 & 0755 = 0755

// Le umask est herite par les processus enfants

// Pour creer un fichier avec des permissions precises:
mode_t old_umask = umask(0);  // Desactiver temporairement
int fd = open("precise.txt", O_CREAT | O_WRONLY, 0600);
umask(old_umask);  // Restaurer

// Valeurs courantes de umask:
// 022 - Fichiers: rw-r--r--, Repertoires: rwxr-xr-x (defaut)
// 077 - Fichiers: rw-------, Repertoires: rwx------ (prive)
// 002 - Fichiers: rw-rw-r--, Repertoires: rwxrwxr-x (collaboratif)
```

**2.3.7.k - Setuid bit: Run as owner**

Le bit setuid (Set User ID, 4000) permet d'executer un programme avec les droits de son proprietaire:

```c
#include <sys/stat.h>

// Activer setuid
chmod("my_program", S_ISUID | S_IRWXU | S_IRGRP | S_IXGRP | S_IROTH | S_IXOTH);
// ou: chmod("my_program", 04755);

// Verification dans ls -l: 's' remplace 'x' pour user
// -rwsr-xr-x  (setuid actif avec x)
// -rwSr-xr-x  (setuid actif SANS x - generalement une erreur)

// Exemple classique: /usr/bin/passwd
// Proprietaire: root
// Permissions: -rwsr-xr-x
// Permet aux utilisateurs de modifier /etc/shadow (propriete de root)

// En code:
#include <unistd.h>

int main(void) {
    printf("Real UID: %d\n", getuid());       // UID de l'utilisateur
    printf("Effective UID: %d\n", geteuid()); // UID du proprietaire (si setuid)

    // Le programme tourne avec euid = proprietaire du fichier
    // Peut acceder aux ressources du proprietaire

    // Bonne pratique: abandonner les privileges des que possible
    setuid(getuid());  // Revenir a l'UID reel

    return 0;
}
```

**2.3.7.l - Setgid bit: Run as group**

Le bit setgid (Set Group ID, 2000) a deux comportements selon le type de fichier:

```c
// Sur un EXECUTABLE: execute avec le GID du groupe proprietaire
// -rwxr-sr-x  (setgid actif)

// Sur un REPERTOIRE: les nouveaux fichiers heritent du groupe du repertoire
// drwxrwsr-x  (setgid sur repertoire)

// Exemple: repertoire de projet partage
mkdir("project", 0775);
chown("project", -1, 1000);           // Groupe = developers (GID 1000)
chmod("project", S_ISGID | 0775);     // 2775

// Maintenant, tout fichier cree dans project/ appartiendra au groupe developers
// meme si l'utilisateur a un groupe primaire different

// Verification
struct stat st;
stat("project", &st);
if (st.st_mode & S_ISGID) {
    printf("Setgid est actif sur le repertoire\n");
}
```

**2.3.7.m - Sticky bit: Restrict deletion**

Le sticky bit (1000) restreint la suppression des fichiers dans un repertoire:

```c
// Le sticky bit sur un repertoire signifie:
// Seuls peuvent supprimer/renommer un fichier:
// - Le proprietaire du fichier
// - Le proprietaire du repertoire
// - root

// Exemple classique: /tmp
// drwxrwxrwt  (777 + sticky)
// Tout le monde peut creer des fichiers, mais chacun ne peut supprimer que les siens

// Activer le sticky bit
chmod("/shared", S_ISVTX | 0777);  // 1777

// Verification dans ls -l: 't' remplace 'x' pour other
// drwxrwxrwt  (sticky actif avec x)
// drwxrwxrwT  (sticky actif SANS x - rare)

// Test du comportement
mkdir("public", 0777);
chmod("public", 01777);  // Activer sticky

// User A cree un fichier
// $ touch public/file_a

// User B ne peut PAS le supprimer meme avec w sur le repertoire
// $ rm public/file_a
// rm: cannot remove 'public/file_a': Operation not permitted
```

---

## Enonce

### Vue d'Ensemble

Vous devez implementer un **gestionnaire de permissions Unix complet** (`libperm`) qui permet d'analyser, afficher, modifier et auditer les permissions des fichiers. Cette bibliotheque doit exposer de maniere pedagogique tous les mecanismes de permissions Unix incluant les bits speciaux.

### Specifications Fonctionnelles

#### Partie 1: Structures de Donnees

```c
// perm.h

#ifndef PERM_H
#define PERM_H

#include <sys/types.h>
#include <sys/stat.h>
#include <stdint.h>

// ============================================
// STRUCTURES DE DONNEES
// ============================================

// 2.3.7.a: Representation des 9 bits de permission rwxrwxrwx
typedef struct {
    uint8_t read    : 1;   // Bit r
    uint8_t write   : 1;   // Bit w
    uint8_t execute : 1;   // Bit x
} perm_rwx_t;

// 2.3.7.b: Les trois categories User/Group/Other
typedef struct {
    perm_rwx_t user;    // Permissions du proprietaire
    perm_rwx_t group;   // Permissions du groupe
    perm_rwx_t other;   // Permissions des autres
} perm_categories_t;

// 2.3.7.k,l,m: Bits speciaux
typedef struct {
    uint8_t setuid : 1;  // 2.3.7.k: Setuid bit
    uint8_t setgid : 1;  // 2.3.7.l: Setgid bit
    uint8_t sticky : 1;  // 2.3.7.m: Sticky bit
} perm_special_t;

// Structure complete des permissions
typedef struct {
    perm_categories_t perms;     // 2.3.7.a,b: rwxrwxrwx
    perm_special_t special;      // 2.3.7.k,l,m: setuid/setgid/sticky
    mode_t raw_mode;             // Mode brut du systeme
} perm_mode_t;

// 2.3.7.c,d,e: Description des types d'acces
typedef enum {
    PERM_ACCESS_READ    = 0x01,  // 2.3.7.c: View contents
    PERM_ACCESS_WRITE   = 0x02,  // 2.3.7.d: Modify contents
    PERM_ACCESS_EXECUTE = 0x04   // 2.3.7.e: Run/traverse
} perm_access_t;

// Informations completes sur un fichier
typedef struct {
    char path[4096];             // Chemin du fichier
    perm_mode_t mode;            // Permissions
    uid_t uid;                   // Proprietaire (user id)
    gid_t gid;                   // Groupe (group id)
    char owner_name[256];        // Nom du proprietaire
    char group_name[256];        // Nom du groupe
    mode_t file_type;            // Type de fichier (S_IFREG, S_IFDIR, etc.)
    int is_symlink;              // Est-ce un lien symbolique?
} perm_file_info_t;

// Resultat de verification d'acces
typedef struct {
    int allowed;                 // Acces autorise?
    perm_access_t requested;     // Type d'acces demande
    const char* category;        // "user", "group", ou "other"
    const char* reason;          // Explication detaillee
} perm_check_result_t;

// 2.3.7.j: Configuration du umask
typedef struct {
    mode_t current_mask;         // Masque actuel
    mode_t file_default;         // Permissions par defaut pour fichiers
    mode_t dir_default;          // Permissions par defaut pour repertoires
} perm_umask_info_t;

// Rapport d'audit de securite
typedef struct {
    int world_writable;          // Accessible en ecriture par tous
    int world_executable;        // Executable par tous
    int setuid_enabled;          // Setuid actif
    int setgid_enabled;          // Setgid actif
    int sticky_enabled;          // Sticky actif
    int owner_is_root;           // Proprietaire est root
    int insecure_permissions;    // Permissions potentiellement dangereuses
    char warnings[10][256];      // Messages d'avertissement
    int warning_count;           // Nombre d'avertissements
} perm_security_audit_t;

// ============================================
// PARTIE 1: Analyse des Permissions
// ============================================

// Obtient les informations de permission d'un fichier
// Couvre: 2.3.7.a (rwxrwxrwx), 2.3.7.b (User/Group/Other),
//         2.3.7.k (setuid), 2.3.7.l (setgid), 2.3.7.m (sticky)
int perm_get_info(const char* path, perm_file_info_t* info);

// Obtient les permissions depuis un file descriptor
int perm_get_info_fd(int fd, perm_file_info_t* info);

// Parse un mode_t en structure perm_mode_t
perm_mode_t perm_parse_mode(mode_t mode);

// ============================================
// PARTIE 2: Affichage des Permissions
// ============================================

// Convertit les permissions en notation symbolique "rwxr-xr-x"
// Couvre: 2.3.7.a (format rwxrwxrwx)
void perm_to_symbolic(const perm_mode_t* mode, char* buf, size_t buflen);

// Convertit les permissions en notation octale "755"
// Couvre: 2.3.7.f (Octal notation)
void perm_to_octal(const perm_mode_t* mode, char* buf, size_t buflen);

// Convertit les permissions en notation octale complete "4755"
void perm_to_octal_full(const perm_mode_t* mode, char* buf, size_t buflen);

// Affiche une representation detaillee des permissions
void perm_print_detailed(const perm_file_info_t* info);

// Genere une representation type "ls -l"
void perm_format_ls(const perm_file_info_t* info, char* buf, size_t buflen);

// ============================================
// PARTIE 3: Verification d'Acces
// ============================================

// Verifie si l'utilisateur courant a un certain acces
// Couvre: 2.3.7.c (Read), 2.3.7.d (Write), 2.3.7.e (Execute)
perm_check_result_t perm_check_access(const char* path, perm_access_t access);

// Verifie pour un utilisateur/groupe specifique
perm_check_result_t perm_check_access_for(const char* path,
                                          uid_t uid, gid_t gid,
                                          perm_access_t access);

// Determine quelle categorie s'applique a un utilisateur
// Couvre: 2.3.7.b (User/Group/Other classification)
const char* perm_get_category(const perm_file_info_t* info, uid_t uid, gid_t gid);

// ============================================
// PARTIE 4: Modification des Permissions
// ============================================

// Change les permissions via le chemin
// Couvre: 2.3.7.g (chmod())
int perm_chmod(const char* path, mode_t mode);

// Change les permissions via file descriptor
// Couvre: 2.3.7.h (fchmod())
int perm_fchmod(int fd, mode_t mode);

// Change les permissions en notation symbolique (comme "chmod u+x")
// Supporte: u/g/o/a, +/-/=, r/w/x/s/t
int perm_chmod_symbolic(const char* path, const char* mode_string);

// Change les permissions en notation octale
// Couvre: 2.3.7.f (Octal notation)
int perm_chmod_octal(const char* path, const char* octal_string);

// Ajoute des permissions (OR logique)
int perm_add(const char* path, mode_t perms_to_add);

// Retire des permissions (AND NOT logique)
int perm_remove(const char* path, mode_t perms_to_remove);

// ============================================
// PARTIE 5: Gestion du Proprietaire
// ============================================

// Change le proprietaire et/ou le groupe
// Couvre: 2.3.7.i (chown())
int perm_chown(const char* path, uid_t uid, gid_t gid);

// Change seulement le proprietaire
int perm_chown_user(const char* path, uid_t uid);

// Change seulement le groupe
int perm_chown_group(const char* path, gid_t gid);

// Change par nom (convertit automatiquement en uid/gid)
int perm_chown_by_name(const char* path,
                       const char* owner_name,
                       const char* group_name);

// Version pour file descriptor
int perm_fchown(int fd, uid_t uid, gid_t gid);

// ============================================
// PARTIE 6: Gestion du Umask
// ============================================

// Obtient les informations sur le umask courant
// Couvre: 2.3.7.j (umask)
int perm_get_umask_info(perm_umask_info_t* info);

// Change le umask et retourne l'ancien
mode_t perm_set_umask(mode_t new_mask);

// Calcule les permissions effectives apres application du umask
mode_t perm_apply_umask(mode_t requested, mode_t umask_val);

// Execute une fonction avec un umask temporaire
int perm_with_umask(mode_t temp_mask, int (*func)(void*), void* arg);

// ============================================
// PARTIE 7: Bits Speciaux
// ============================================

// Active le bit setuid
// Couvre: 2.3.7.k (Setuid bit)
int perm_set_setuid(const char* path, int enable);

// Active le bit setgid
// Couvre: 2.3.7.l (Setgid bit)
int perm_set_setgid(const char* path, int enable);

// Active le bit sticky
// Couvre: 2.3.7.m (Sticky bit)
int perm_set_sticky(const char* path, int enable);

// Verifie les bits speciaux
int perm_has_setuid(const char* path);
int perm_has_setgid(const char* path);
int perm_has_sticky(const char* path);

// ============================================
// PARTIE 8: Audit de Securite
// ============================================

// Effectue un audit de securite sur un fichier
int perm_security_audit(const char* path, perm_security_audit_t* audit);

// Recherche les fichiers avec des permissions dangereuses
typedef int (*perm_audit_callback_t)(const char* path,
                                     const perm_security_audit_t* audit,
                                     void* user_data);

int perm_audit_recursive(const char* root_path,
                         perm_audit_callback_t callback,
                         void* user_data);

// Detecte les fichiers setuid/setgid
int perm_find_setuid_files(const char* root_path,
                           void (*callback)(const char* path, void* data),
                           void* user_data);

// ============================================
// PARTIE 9: Operations par Lot
// ============================================

// Applique des permissions recursivement
typedef struct {
    mode_t file_mode;            // Mode pour les fichiers
    mode_t dir_mode;             // Mode pour les repertoires
    int apply_to_files;          // Appliquer aux fichiers?
    int apply_to_dirs;           // Appliquer aux repertoires?
    int follow_symlinks;         // Suivre les liens symboliques?
    int verbose;                 // Mode verbeux
} perm_recursive_opts_t;

int perm_chmod_recursive(const char* path, const perm_recursive_opts_t* opts);

// Copie les permissions d'un fichier vers un autre
int perm_copy_mode(const char* source, const char* dest);

// ============================================
// PARTIE 10: Utilitaires
// ============================================

// Convertit une chaine octale en mode_t
mode_t perm_octal_to_mode(const char* octal_str);

// Convertit une chaine symbolique en operations de mode
int perm_parse_symbolic(const char* symbolic,
                        mode_t* add_perms,
                        mode_t* remove_perms);

// Obtient le uid depuis un nom d'utilisateur
int perm_name_to_uid(const char* name, uid_t* uid);

// Obtient le gid depuis un nom de groupe
int perm_name_to_gid(const char* name, gid_t* gid);

// Obtient le nom depuis un uid
int perm_uid_to_name(uid_t uid, char* name, size_t namelen);

// Obtient le nom depuis un gid
int perm_gid_to_name(gid_t gid, char* name, size_t namelen);

// Message d'erreur pour le dernier echec
const char* perm_strerror(void);

#endif // PERM_H
```

#### Partie 2: Programme de Demonstration

```c
// perm_demo.c - Programme de demonstration

#include "perm.h"
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>
#include <fcntl.h>

void demonstrate_permission_bits(void) {
    printf("=== 2.3.7.a: Permission bits rwxrwxrwx ===\n\n");

    // Creer un fichier de test
    int fd = open("test_perms.txt", O_CREAT | O_WRONLY, 0644);
    close(fd);

    perm_file_info_t info;
    perm_get_info("test_perms.txt", &info);

    printf("Fichier: %s\n", info.path);
    printf("Mode brut: %04o\n", info.mode.raw_mode & 07777);

    char symbolic[16];
    perm_to_symbolic(&info.mode, symbolic, sizeof(symbolic));
    printf("Notation symbolique: %s\n", symbolic);

    char octal[8];
    perm_to_octal(&info.mode, octal, sizeof(octal));
    printf("Notation octale: %s\n", octal);

    printf("\nDecomposition des bits:\n");
    printf("  User  - r:%d w:%d x:%d\n",
           info.mode.perms.user.read,
           info.mode.perms.user.write,
           info.mode.perms.user.execute);
    printf("  Group - r:%d w:%d x:%d\n",
           info.mode.perms.group.read,
           info.mode.perms.group.write,
           info.mode.perms.group.execute);
    printf("  Other - r:%d w:%d x:%d\n",
           info.mode.perms.other.read,
           info.mode.perms.other.write,
           info.mode.perms.other.execute);
}

void demonstrate_categories(void) {
    printf("\n=== 2.3.7.b: User/Group/Other Categories ===\n\n");

    perm_file_info_t info;
    perm_get_info("test_perms.txt", &info);

    printf("Fichier: %s\n", info.path);
    printf("Proprietaire: %s (UID %d)\n", info.owner_name, info.uid);
    printf("Groupe: %s (GID %d)\n", info.group_name, info.gid);

    printf("\nCategorie pour l'utilisateur courant:\n");
    uid_t my_uid = getuid();
    gid_t my_gid = getgid();
    const char* category = perm_get_category(&info, my_uid, my_gid);
    printf("  UID %d, GID %d -> Categorie: %s\n", my_uid, my_gid, category);
}

void demonstrate_read_permission(void) {
    printf("\n=== 2.3.7.c: Read Permission (View contents) ===\n\n");

    // Creer un fichier lisible
    int fd = open("readable.txt", O_CREAT | O_WRONLY, 0644);
    write(fd, "Contenu lisible\n", 16);
    close(fd);

    // Creer un fichier non-lisible
    fd = open("unreadable.txt", O_CREAT | O_WRONLY, 0200);
    write(fd, "Contenu secret\n", 15);
    close(fd);

    printf("Test de permission de lecture:\n");

    perm_check_result_t result;

    result = perm_check_access("readable.txt", PERM_ACCESS_READ);
    printf("  readable.txt: %s (%s)\n",
           result.allowed ? "AUTORISE" : "REFUSE",
           result.reason);

    result = perm_check_access("unreadable.txt", PERM_ACCESS_READ);
    printf("  unreadable.txt: %s (%s)\n",
           result.allowed ? "AUTORISE" : "REFUSE",
           result.reason);

    // Nettoyer
    chmod("unreadable.txt", 0644);
}

void demonstrate_write_permission(void) {
    printf("\n=== 2.3.7.d: Write Permission (Modify contents) ===\n\n");

    // Creer un fichier en lecture seule
    int fd = open("readonly.txt", O_CREAT | O_WRONLY, 0444);
    write(fd, "Read-only\n", 10);
    close(fd);

    // Creer un fichier modifiable
    fd = open("writable.txt", O_CREAT | O_WRONLY, 0644);
    write(fd, "Writable\n", 9);
    close(fd);

    printf("Test de permission d'ecriture:\n");

    perm_check_result_t result;

    result = perm_check_access("writable.txt", PERM_ACCESS_WRITE);
    printf("  writable.txt: %s (%s)\n",
           result.allowed ? "AUTORISE" : "REFUSE",
           result.reason);

    result = perm_check_access("readonly.txt", PERM_ACCESS_WRITE);
    printf("  readonly.txt: %s (%s)\n",
           result.allowed ? "AUTORISE" : "REFUSE",
           result.reason);

    // Nettoyer
    chmod("readonly.txt", 0644);
}

void demonstrate_execute_permission(void) {
    printf("\n=== 2.3.7.e: Execute Permission (Run/Traverse) ===\n\n");

    // Creer un script executable
    int fd = open("script.sh", O_CREAT | O_WRONLY, 0755);
    write(fd, "#!/bin/sh\necho 'Hello'\n", 23);
    close(fd);

    // Creer un script non-executable
    fd = open("not_executable.sh", O_CREAT | O_WRONLY, 0644);
    write(fd, "#!/bin/sh\necho 'Hello'\n", 23);
    close(fd);

    printf("Test de permission d'execution:\n");

    perm_check_result_t result;

    result = perm_check_access("script.sh", PERM_ACCESS_EXECUTE);
    printf("  script.sh: %s (%s)\n",
           result.allowed ? "AUTORISE" : "REFUSE",
           result.reason);

    result = perm_check_access("not_executable.sh", PERM_ACCESS_EXECUTE);
    printf("  not_executable.sh: %s (%s)\n",
           result.allowed ? "AUTORISE" : "REFUSE",
           result.reason);

    // Test sur un repertoire
    mkdir("test_dir", 0755);
    mkdir("no_traverse_dir", 0644);

    printf("\nTest de traversee de repertoire:\n");

    result = perm_check_access("test_dir", PERM_ACCESS_EXECUTE);
    printf("  test_dir (755): %s (%s)\n",
           result.allowed ? "AUTORISE" : "REFUSE",
           result.reason);

    result = perm_check_access("no_traverse_dir", PERM_ACCESS_EXECUTE);
    printf("  no_traverse_dir (644): %s (%s)\n",
           result.allowed ? "AUTORISE" : "REFUSE",
           result.reason);

    // Nettoyer
    chmod("no_traverse_dir", 0755);
}

void demonstrate_octal_notation(void) {
    printf("\n=== 2.3.7.f: Octal Notation ===\n\n");

    printf("Exemples de notation octale:\n\n");

    struct {
        mode_t mode;
        const char* description;
    } examples[] = {
        {0755, "rwxr-xr-x - Executables, repertoires"},
        {0644, "rw-r--r-- - Fichiers normaux"},
        {0700, "rwx------ - Prive (user uniquement)"},
        {0777, "rwxrwxrwx - Acces total (dangereux!)"},
        {0600, "rw------- - Fichiers sensibles"},
        {04755, "rwsr-xr-x - Setuid + 755"},
        {02755, "rwxr-sr-x - Setgid + 755"},
        {01777, "rwxrwxrwt - Sticky + 777 (/tmp)"},
        {0, NULL}
    };

    for (int i = 0; examples[i].description; i++) {
        perm_mode_t mode = perm_parse_mode(examples[i].mode);
        char symbolic[16];
        perm_to_symbolic(&mode, symbolic, sizeof(symbolic));

        printf("  %04o = %s = %s\n",
               examples[i].mode, symbolic, examples[i].description);
    }

    // Test de conversion
    printf("\nConversion de notation:\n");
    int fd = open("octal_test.txt", O_CREAT | O_WRONLY, 0644);
    close(fd);

    printf("  Avant: ");
    perm_file_info_t info;
    perm_get_info("octal_test.txt", &info);
    char buf[8];
    perm_to_octal(&info.mode, buf, sizeof(buf));
    printf("%s\n", buf);

    // Changer avec notation octale
    perm_chmod_octal("octal_test.txt", "755");

    printf("  Apres chmod_octal('755'): ");
    perm_get_info("octal_test.txt", &info);
    perm_to_octal(&info.mode, buf, sizeof(buf));
    printf("%s\n", buf);
}

void demonstrate_chmod(void) {
    printf("\n=== 2.3.7.g: chmod() - Change permissions ===\n\n");

    int fd = open("chmod_test.txt", O_CREAT | O_WRONLY, 0644);
    close(fd);

    printf("Fichier: chmod_test.txt\n");

    perm_file_info_t info;
    char buf[16];

    // Etat initial
    perm_get_info("chmod_test.txt", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Initial: %s (0%o)\n", buf, info.mode.raw_mode & 07777);

    // Utiliser chmod avec mode_t
    perm_chmod("chmod_test.txt", 0755);
    perm_get_info("chmod_test.txt", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Apres perm_chmod(0755): %s\n", buf);

    // Utiliser notation symbolique
    perm_chmod_symbolic("chmod_test.txt", "u+x,g-w,o=r");
    perm_get_info("chmod_test.txt", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Apres 'u+x,g-w,o=r': %s\n", buf);

    // Ajouter des permissions
    perm_add("chmod_test.txt", S_IWGRP | S_IWOTH);
    perm_get_info("chmod_test.txt", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Apres ajout g+w,o+w: %s\n", buf);

    // Retirer des permissions
    perm_remove("chmod_test.txt", S_IWOTH | S_IXOTH);
    perm_get_info("chmod_test.txt", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Apres retrait o-wx: %s\n", buf);
}

void demonstrate_fchmod(void) {
    printf("\n=== 2.3.7.h: fchmod() - Change by fd ===\n\n");

    int fd = open("fchmod_test.txt", O_CREAT | O_RDWR, 0644);
    if (fd < 0) {
        perror("open");
        return;
    }

    printf("Fichier: fchmod_test.txt (fd=%d)\n", fd);

    perm_file_info_t info;
    char buf[16];

    perm_get_info_fd(fd, &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Initial: %s\n", buf);

    // Changer via fchmod
    perm_fchmod(fd, 0600);
    perm_get_info_fd(fd, &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Apres fchmod(0600): %s\n", buf);

    printf("\nAvantages de fchmod:\n");
    printf("  - Pas de race condition TOCTOU\n");
    printf("  - Opere sur le fichier reellement ouvert\n");
    printf("  - Plus performant (pas de resolution de chemin)\n");

    close(fd);
}

void demonstrate_chown(void) {
    printf("\n=== 2.3.7.i: chown() - Change owner ===\n\n");

    int fd = open("chown_test.txt", O_CREAT | O_WRONLY, 0644);
    close(fd);

    perm_file_info_t info;
    perm_get_info("chown_test.txt", &info);

    printf("Fichier: chown_test.txt\n");
    printf("  Proprietaire actuel: %s (UID %d)\n", info.owner_name, info.uid);
    printf("  Groupe actuel: %s (GID %d)\n", info.group_name, info.gid);

    printf("\nNote: chown() necessite les privileges root pour:\n");
    printf("  - Changer le proprietaire\n");
    printf("  - Changer vers un groupe dont on n'est pas membre\n");

    // Tenter de changer seulement le groupe (peut fonctionner sans root)
    gid_t my_gid = getgid();
    if (perm_chown_group("chown_test.txt", my_gid) == 0) {
        printf("\n  Groupe change vers GID %d\n", my_gid);
    }

    printf("\nAttention: chown() efface automatiquement les bits setuid/setgid!\n");
    printf("  C'est une mesure de securite contre l'elevation de privileges.\n");
}

void demonstrate_umask(void) {
    printf("\n=== 2.3.7.j: umask - Default permission mask ===\n\n");

    perm_umask_info_t umask_info;
    perm_get_umask_info(&umask_info);

    printf("Umask actuel: %04o\n", umask_info.current_mask);
    printf("Permissions par defaut:\n");
    printf("  Fichiers: %04o\n", umask_info.file_default);
    printf("  Repertoires: %04o\n", umask_info.dir_default);

    printf("\nFonctionnement du umask:\n");
    printf("  permissions_effectives = permissions_demandees & ~umask\n\n");

    // Demonstration
    mode_t old_umask = perm_set_umask(0022);
    printf("Avec umask = 022:\n");

    int fd = open("umask_demo1.txt", O_CREAT | O_WRONLY, 0666);
    close(fd);
    perm_file_info_t info;
    perm_get_info("umask_demo1.txt", &info);
    char buf[16];
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  open(..., 0666) -> %s (%04o)\n", buf, info.mode.raw_mode & 07777);

    // Avec umask plus restrictif
    perm_set_umask(0077);
    printf("\nAvec umask = 077:\n");

    fd = open("umask_demo2.txt", O_CREAT | O_WRONLY, 0666);
    close(fd);
    perm_get_info("umask_demo2.txt", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  open(..., 0666) -> %s (%04o)\n", buf, info.mode.raw_mode & 07777);

    // Restaurer
    perm_set_umask(old_umask);
    printf("\nUmask restaure a %04o\n", old_umask);
}

void demonstrate_setuid(void) {
    printf("\n=== 2.3.7.k: Setuid bit - Run as owner ===\n\n");

    // Creer un script "setuid" (les scripts setuid ne fonctionnent pas vraiment
    // sur la plupart des systemes modernes, mais on peut illustrer le concept)
    int fd = open("setuid_demo", O_CREAT | O_WRONLY, 0755);
    write(fd, "#!/bin/sh\nid\n", 13);
    close(fd);

    printf("Fichier: setuid_demo\n");

    perm_file_info_t info;
    char buf[16];

    perm_get_info("setuid_demo", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Avant: %s\n", buf);

    // Activer setuid
    perm_set_setuid("setuid_demo", 1);
    perm_get_info("setuid_demo", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Apres set_setuid(1): %s\n", buf);
    printf("  Mode octal: %04o\n", info.mode.raw_mode & 07777);

    printf("\nExplication:\n");
    printf("  - Le 's' dans la position user/x indique setuid actif\n");
    printf("  - Le programme s'execute avec l'EUID du proprietaire\n");
    printf("  - Exemple classique: /usr/bin/passwd (setuid root)\n");

    // Verifier
    printf("\n  perm_has_setuid(): %s\n",
           perm_has_setuid("setuid_demo") ? "OUI" : "NON");

    // Desactiver
    perm_set_setuid("setuid_demo", 0);
    perm_get_info("setuid_demo", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("\n  Apres set_setuid(0): %s\n", buf);
}

void demonstrate_setgid(void) {
    printf("\n=== 2.3.7.l: Setgid bit - Run as group ===\n\n");

    // Sur un fichier executable
    int fd = open("setgid_demo", O_CREAT | O_WRONLY, 0755);
    write(fd, "#!/bin/sh\nid\n", 13);
    close(fd);

    printf("1. Setgid sur un fichier executable:\n");

    perm_file_info_t info;
    char buf[16];

    perm_set_setgid("setgid_demo", 1);
    perm_get_info("setgid_demo", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Permissions: %s\n", buf);
    printf("  Le programme s'execute avec l'EGID du groupe proprietaire\n");

    // Sur un repertoire
    mkdir("setgid_dir", 0775);

    printf("\n2. Setgid sur un repertoire:\n");

    perm_set_setgid("setgid_dir", 1);
    perm_get_info("setgid_dir", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Permissions: %s\n", buf);
    printf("  Les nouveaux fichiers heritent du groupe du repertoire\n");
    printf("  Utile pour les repertoires de projet partages\n");

    printf("\n  perm_has_setgid(): %s\n",
           perm_has_setgid("setgid_dir") ? "OUI" : "NON");
}

void demonstrate_sticky(void) {
    printf("\n=== 2.3.7.m: Sticky bit - Restrict deletion ===\n\n");

    mkdir("sticky_dir", 0777);

    printf("Repertoire: sticky_dir\n");

    perm_file_info_t info;
    char buf[16];

    perm_get_info("sticky_dir", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Avant: %s\n", buf);

    perm_set_sticky("sticky_dir", 1);
    perm_get_info("sticky_dir", &info);
    perm_to_symbolic(&info.mode, buf, sizeof(buf));
    printf("  Apres set_sticky(1): %s\n", buf);
    printf("  Mode octal: %04o\n", info.mode.raw_mode & 07777);

    printf("\nExplication:\n");
    printf("  - Le 't' dans la position other/x indique sticky actif\n");
    printf("  - Seul le proprietaire d'un fichier peut le supprimer\n");
    printf("  - Meme si le repertoire est world-writable (777)\n");
    printf("  - Exemple classique: /tmp (drwxrwxrwt)\n");

    printf("\n  perm_has_sticky(): %s\n",
           perm_has_sticky("sticky_dir") ? "OUI" : "NON");
}

void demonstrate_security_audit(void) {
    printf("\n=== Audit de Securite ===\n\n");

    // Creer des fichiers avec permissions variees pour l'audit
    int fd = open("world_writable.txt", O_CREAT | O_WRONLY, 0666);
    close(fd);

    fd = open("setuid_file", O_CREAT | O_WRONLY, 04755);
    close(fd);

    printf("Audit de fichiers:\n\n");

    const char* files[] = {
        "world_writable.txt",
        "setuid_file",
        "test_perms.txt",
        NULL
    };

    for (int i = 0; files[i]; i++) {
        perm_security_audit_t audit;
        if (perm_security_audit(files[i], &audit) == 0) {
            printf("Fichier: %s\n", files[i]);

            if (audit.world_writable)
                printf("  [!] World-writable\n");
            if (audit.setuid_enabled)
                printf("  [!] Setuid actif\n");
            if (audit.setgid_enabled)
                printf("  [!] Setgid actif\n");
            if (audit.insecure_permissions)
                printf("  [!] Permissions potentiellement dangereuses\n");

            for (int j = 0; j < audit.warning_count; j++) {
                printf("  Warning: %s\n", audit.warnings[j]);
            }

            if (!audit.insecure_permissions && audit.warning_count == 0)
                printf("  [OK] Aucun probleme detecte\n");

            printf("\n");
        }
    }
}

void cleanup_demo_files(void) {
    // Nettoyer les fichiers de demonstration
    unlink("test_perms.txt");
    unlink("readable.txt");
    unlink("unreadable.txt");
    unlink("readonly.txt");
    unlink("writable.txt");
    unlink("script.sh");
    unlink("not_executable.sh");
    rmdir("test_dir");
    rmdir("no_traverse_dir");
    unlink("octal_test.txt");
    unlink("chmod_test.txt");
    unlink("fchmod_test.txt");
    unlink("chown_test.txt");
    unlink("umask_demo1.txt");
    unlink("umask_demo2.txt");
    unlink("setuid_demo");
    unlink("setgid_demo");
    rmdir("setgid_dir");
    rmdir("sticky_dir");
    unlink("world_writable.txt");
    unlink("setuid_file");
}

int main(void) {
    printf("============================================\n");
    printf("  DEMONSTRATION DU GESTIONNAIRE DE PERMISSIONS\n");
    printf("  Couvrant tous les concepts 2.3.7.a-m\n");
    printf("============================================\n");

    demonstrate_permission_bits();        // 2.3.7.a
    demonstrate_categories();             // 2.3.7.b
    demonstrate_read_permission();        // 2.3.7.c
    demonstrate_write_permission();       // 2.3.7.d
    demonstrate_execute_permission();     // 2.3.7.e
    demonstrate_octal_notation();         // 2.3.7.f
    demonstrate_chmod();                  // 2.3.7.g
    demonstrate_fchmod();                 // 2.3.7.h
    demonstrate_chown();                  // 2.3.7.i
    demonstrate_umask();                  // 2.3.7.j
    demonstrate_setuid();                 // 2.3.7.k
    demonstrate_setgid();                 // 2.3.7.l
    demonstrate_sticky();                 // 2.3.7.m
    demonstrate_security_audit();

    cleanup_demo_files();

    printf("\n=== Fin de la demonstration ===\n");
    return 0;
}
```

---

## Fonctions Autorisees

### Syscalls de permissions
- `chmod`, `fchmod`, `fchmodat`
- `chown`, `fchown`, `lchown`, `fchownat`
- `umask`
- `access`, `faccessat`
- `stat`, `fstat`, `lstat`, `fstatat`

### Gestion des utilisateurs/groupes
- `getuid`, `geteuid`, `getgid`, `getegid`
- `getpwnam`, `getpwuid` (password database)
- `getgrnam`, `getgrgid` (group database)
- `getgroups` (groupes supplementaires)

### Fichiers et repertoires
- `open`, `close`, `read`, `write`
- `opendir`, `readdir`, `closedir`
- `mkdir`, `rmdir`, `unlink`

### Gestion memoire
- `malloc`, `calloc`, `realloc`, `free`

### Manipulation de chaines
- `memset`, `memcpy`, `memmove`
- `strlen`, `strncpy`, `strcmp`, `strncmp`
- `snprintf`, `vsnprintf`
- `strtol`, `strtoul`

### Autres
- `perror`, `strerror`
- `errno`

### Fonctions interdites
- Aucune fonction de la famille exec*() dans la bibliotheque
- Pas de system(), popen()

---

## Contraintes

### Contraintes de Code

1. **Standard C17** - Compilation avec `-std=c17 -Wall -Wextra -Werror -pedantic`

2. **Pas de fuites de memoire** - Tout `malloc` doit avoir son `free` correspondant

3. **Gestion d'erreurs exhaustive** - Toute fonction systeme doit voir son retour verifie

4. **Securite** - Ne jamais suivre les liens symboliques de maniere non securisee

5. **Pas de variables globales mutables** - Constantes globales permises uniquement

6. **Documentation** - Chaque fonction doit mentionner les concepts 2.3.7.X couverts

### Contraintes de Design

1. **API coherente** - Toutes les fonctions suivent les memes conventions de retour

2. **Validation des entrees** - Les chemins NULL ou vides doivent etre rejetes proprement

3. **Messages d'erreur clairs** - `perm_strerror()` doit retourner des messages utiles

4. **Robustesse face aux race conditions** - Utiliser les variantes *at() quand possible

### Contraintes de Securite

1. **Pas d'elevation de privileges** - La bibliotheque ne doit pas permettre de contourner les permissions

2. **Audit des bits speciaux** - Avertir systematiquement sur les fichiers setuid/setgid

3. **Validation des chemins** - Verifier l'absence de composants ".." dangereux

---

## Tests Moulinette

La moulinette Rust 2024 executera les tests suivants:

### Test 1: Permission Bits (2.3.7.a)

```rust
#[test]
fn test_permission_bits_parsing() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            int fd = open("test_bits.txt", O_CREAT | O_WRONLY, 0754);
            close(fd);

            perm_file_info_t info;
            if (perm_get_info("test_bits.txt", &info) != 0) return 1;

            // 2.3.7.a: Verifier les 9 bits rwxrwxrwx
            // 754 = rwxr-xr--

            // User: rwx (7)
            if (!info.mode.perms.user.read) return 2;
            if (!info.mode.perms.user.write) return 3;
            if (!info.mode.perms.user.execute) return 4;

            // Group: r-x (5)
            if (!info.mode.perms.group.read) return 5;
            if (info.mode.perms.group.write) return 6;  // Pas de w
            if (!info.mode.perms.group.execute) return 7;

            // Other: r-- (4)
            if (!info.mode.perms.other.read) return 8;
            if (info.mode.perms.other.write) return 9;   // Pas de w
            if (info.mode.perms.other.execute) return 10; // Pas de x

            unlink("test_bits.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.a: Parsing des bits rwxrwxrwx");
}

#[test]
fn test_symbolic_notation() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <string.h>

        int main(void) {
            perm_mode_t mode = perm_parse_mode(0754);
            char buf[16];

            perm_to_symbolic(&mode, buf, sizeof(buf));

            // 2.3.7.a: Format rwxrwxrwx
            if (strcmp(buf, "rwxr-xr--") != 0) return 1;

            // Test avec bits speciaux
            mode = perm_parse_mode(04755);
            perm_to_symbolic(&mode, buf, sizeof(buf));
            if (strcmp(buf, "rwsr-xr-x") != 0) return 2;

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.a: Notation symbolique");
}
```

### Test 2: User/Group/Other Categories (2.3.7.b)

```rust
#[test]
fn test_categories() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <unistd.h>
        #include <fcntl.h>
        #include <string.h>

        int main(void) {
            int fd = open("cat_test.txt", O_CREAT | O_WRONLY, 0644);
            close(fd);

            perm_file_info_t info;
            perm_get_info("cat_test.txt", &info);

            // 2.3.7.b: Tester la classification User/Group/Other
            uid_t my_uid = getuid();
            gid_t my_gid = getgid();

            // L'utilisateur courant devrait etre "user" car il est proprietaire
            const char* cat = perm_get_category(&info, my_uid, my_gid);
            if (strcmp(cat, "user") != 0) return 1;

            // Un autre UID devrait etre "other" (sauf si meme groupe)
            cat = perm_get_category(&info, 99999, 99999);
            if (strcmp(cat, "other") != 0) return 2;

            unlink("cat_test.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.b: Classification User/Group/Other");
}
```

### Test 3: Read Permission (2.3.7.c)

```rust
#[test]
fn test_read_permission() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.c: Read permission - View contents

            // Fichier lisible
            int fd = open("readable.txt", O_CREAT | O_WRONLY, 0644);
            close(fd);

            perm_check_result_t result = perm_check_access("readable.txt",
                                                           PERM_ACCESS_READ);
            if (!result.allowed) return 1;

            // Fichier non-lisible (write-only)
            fd = open("writeonly.txt", O_CREAT | O_WRONLY, 0200);
            close(fd);

            result = perm_check_access("writeonly.txt", PERM_ACCESS_READ);
            if (result.allowed) return 2;  // Ne devrait PAS etre lisible

            // Nettoyer
            chmod("writeonly.txt", 0644);
            unlink("readable.txt");
            unlink("writeonly.txt");

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.c: Permission de lecture");
}
```

### Test 4: Write Permission (2.3.7.d)

```rust
#[test]
fn test_write_permission() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.d: Write permission - Modify contents

            // Fichier en lecture seule
            int fd = open("readonly.txt", O_CREAT | O_WRONLY, 0444);
            close(fd);

            perm_check_result_t result = perm_check_access("readonly.txt",
                                                           PERM_ACCESS_WRITE);
            if (result.allowed) return 1;  // Ne devrait PAS etre modifiable

            // Fichier modifiable
            fd = open("writable.txt", O_CREAT | O_WRONLY, 0644);
            close(fd);

            result = perm_check_access("writable.txt", PERM_ACCESS_WRITE);
            if (!result.allowed) return 2;

            // Nettoyer
            chmod("readonly.txt", 0644);
            unlink("readonly.txt");
            unlink("writable.txt");

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.d: Permission d'ecriture");
}
```

### Test 5: Execute Permission (2.3.7.e)

```rust
#[test]
fn test_execute_permission() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>
        #include <sys/stat.h>

        int main(void) {
            // 2.3.7.e: Execute permission - Run file / traverse directory

            // Script executable
            int fd = open("exec.sh", O_CREAT | O_WRONLY, 0755);
            close(fd);

            perm_check_result_t result = perm_check_access("exec.sh",
                                                           PERM_ACCESS_EXECUTE);
            if (!result.allowed) return 1;

            // Script non-executable
            fd = open("noexec.sh", O_CREAT | O_WRONLY, 0644);
            close(fd);

            result = perm_check_access("noexec.sh", PERM_ACCESS_EXECUTE);
            if (result.allowed) return 2;

            // Repertoire traversable
            mkdir("traverse_dir", 0755);
            result = perm_check_access("traverse_dir", PERM_ACCESS_EXECUTE);
            if (!result.allowed) return 3;

            // Repertoire non-traversable
            mkdir("no_traverse", 0644);
            result = perm_check_access("no_traverse", PERM_ACCESS_EXECUTE);
            if (result.allowed) return 4;

            // Nettoyer
            chmod("no_traverse", 0755);
            unlink("exec.sh");
            unlink("noexec.sh");
            rmdir("traverse_dir");
            rmdir("no_traverse");

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.e: Permission d'execution/traversee");
}
```

### Test 6: Octal Notation (2.3.7.f)

```rust
#[test]
fn test_octal_notation() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <string.h>
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.f: Octal notation: 755, 644

            // Test conversion mode_t -> octal string
            perm_mode_t mode = perm_parse_mode(0755);
            char buf[8];
            perm_to_octal(&mode, buf, sizeof(buf));
            if (strcmp(buf, "755") != 0) return 1;

            mode = perm_parse_mode(0644);
            perm_to_octal(&mode, buf, sizeof(buf));
            if (strcmp(buf, "644") != 0) return 2;

            // Test avec bits speciaux
            mode = perm_parse_mode(04755);
            perm_to_octal_full(&mode, buf, sizeof(buf));
            if (strcmp(buf, "4755") != 0) return 3;

            // Test conversion string -> mode_t
            if (perm_octal_to_mode("755") != 0755) return 4;
            if (perm_octal_to_mode("644") != 0644) return 5;
            if (perm_octal_to_mode("4755") != 04755) return 6;

            // Test chmod avec notation octale
            int fd = open("octal_test.txt", O_CREAT | O_WRONLY, 0600);
            close(fd);

            perm_chmod_octal("octal_test.txt", "755");

            perm_file_info_t info;
            perm_get_info("octal_test.txt", &info);
            if ((info.mode.raw_mode & 0777) != 0755) return 7;

            unlink("octal_test.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.f: Notation octale");
}
```

### Test 7: chmod() (2.3.7.g)

```rust
#[test]
fn test_chmod() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.g: chmod() - Change permissions

            int fd = open("chmod_test.txt", O_CREAT | O_WRONLY, 0644);
            close(fd);

            // Test chmod() avec mode_t
            if (perm_chmod("chmod_test.txt", 0755) != 0) return 1;

            perm_file_info_t info;
            perm_get_info("chmod_test.txt", &info);
            if ((info.mode.raw_mode & 0777) != 0755) return 2;

            // Test chmod symbolique
            if (perm_chmod_symbolic("chmod_test.txt", "u-x,g+w") != 0) return 3;

            perm_get_info("chmod_test.txt", &info);
            // 755 -> u-x -> 655 -> g+w -> 675
            if ((info.mode.raw_mode & 0777) != 0675) return 4;

            // Test perm_add
            perm_chmod("chmod_test.txt", 0600);
            perm_add("chmod_test.txt", S_IRGRP | S_IROTH);
            perm_get_info("chmod_test.txt", &info);
            if ((info.mode.raw_mode & 0777) != 0644) return 5;

            // Test perm_remove
            perm_remove("chmod_test.txt", S_IROTH);
            perm_get_info("chmod_test.txt", &info);
            if ((info.mode.raw_mode & 0777) != 0640) return 6;

            unlink("chmod_test.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.g: chmod()");
}
```

### Test 8: fchmod() (2.3.7.h)

```rust
#[test]
fn test_fchmod() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.h: fchmod() - Change by fd

            int fd = open("fchmod_test.txt", O_CREAT | O_RDWR, 0644);
            if (fd < 0) return 1;

            // Changer via fchmod
            if (perm_fchmod(fd, 0600) != 0) return 2;

            perm_file_info_t info;
            perm_get_info_fd(fd, &info);
            if ((info.mode.raw_mode & 0777) != 0600) return 3;

            // Encore un changement
            if (perm_fchmod(fd, 0755) != 0) return 4;
            perm_get_info_fd(fd, &info);
            if ((info.mode.raw_mode & 0777) != 0755) return 5;

            close(fd);
            unlink("fchmod_test.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.h: fchmod()");
}
```

### Test 9: chown() (2.3.7.i)

```rust
#[test]
fn test_chown() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.i: chown() - Change owner
            // Note: Les changements de proprietaire necessitent root
            // On teste principalement la verification des parametres

            int fd = open("chown_test.txt", O_CREAT | O_WRONLY, 0644);
            close(fd);

            perm_file_info_t info;
            perm_get_info("chown_test.txt", &info);
            uid_t original_uid = info.uid;
            gid_t original_gid = info.gid;

            // Tenter de changer le groupe vers notre propre groupe (devrait reussir)
            gid_t my_gid = getgid();
            if (perm_chown_group("chown_test.txt", my_gid) != 0) {
                // Peut echouer si deja un autre groupe
                // Ce n'est pas une erreur fatale
            }

            // Verifier que chown avec -1 ne change pas le champ
            if (perm_chown("chown_test.txt", (uid_t)-1, (gid_t)-1) != 0) return 1;

            perm_get_info("chown_test.txt", &info);
            // Le uid ne devrait pas avoir change (sauf si root)

            unlink("chown_test.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.i: chown()");
}
```

### Test 10: umask (2.3.7.j)

```rust
#[test]
fn test_umask() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.j: umask - Default permission mask

            // Sauvegarder l'umask original
            mode_t original = perm_set_umask(0022);

            // Verifier le fonctionnement de perm_get_umask_info
            perm_umask_info_t info;
            perm_get_umask_info(&info);
            if (info.current_mask != 0022) return 1;

            // Creer un fichier avec umask 022
            int fd = open("umask_test1.txt", O_CREAT | O_WRONLY, 0666);
            close(fd);

            perm_file_info_t finfo;
            perm_get_info("umask_test1.txt", &finfo);
            // 0666 & ~022 = 0644
            if ((finfo.mode.raw_mode & 0777) != 0644) return 2;

            // Changer le umask
            perm_set_umask(0077);

            fd = open("umask_test2.txt", O_CREAT | O_WRONLY, 0666);
            close(fd);

            perm_get_info("umask_test2.txt", &finfo);
            // 0666 & ~077 = 0600
            if ((finfo.mode.raw_mode & 0777) != 0600) return 3;

            // Tester perm_apply_umask
            mode_t result = perm_apply_umask(0777, 0022);
            if (result != 0755) return 4;

            result = perm_apply_umask(0666, 0077);
            if (result != 0600) return 5;

            // Restaurer
            perm_set_umask(original);

            unlink("umask_test1.txt");
            unlink("umask_test2.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.j: umask");
}
```

### Test 11: Setuid Bit (2.3.7.k)

```rust
#[test]
fn test_setuid_bit() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.k: Setuid bit - Run as owner

            int fd = open("setuid_test", O_CREAT | O_WRONLY, 0755);
            close(fd);

            // Verifier que setuid n'est pas actif initialement
            if (perm_has_setuid("setuid_test")) return 1;

            // Activer setuid
            if (perm_set_setuid("setuid_test", 1) != 0) return 2;

            // Verifier
            if (!perm_has_setuid("setuid_test")) return 3;

            perm_file_info_t info;
            perm_get_info("setuid_test", &info);
            if (!info.mode.special.setuid) return 4;
            if ((info.mode.raw_mode & S_ISUID) == 0) return 5;

            // Notation symbolique doit montrer 's'
            char buf[16];
            perm_to_symbolic(&info.mode, buf, sizeof(buf));
            if (buf[2] != 's') return 6;  // Position du x de user

            // Desactiver
            if (perm_set_setuid("setuid_test", 0) != 0) return 7;
            if (perm_has_setuid("setuid_test")) return 8;

            unlink("setuid_test");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.k: Setuid bit");
}
```

### Test 12: Setgid Bit (2.3.7.l)

```rust
#[test]
fn test_setgid_bit() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>
        #include <sys/stat.h>

        int main(void) {
            // 2.3.7.l: Setgid bit - Run as group

            // Test sur fichier
            int fd = open("setgid_file", O_CREAT | O_WRONLY, 0755);
            close(fd);

            if (perm_has_setgid("setgid_file")) return 1;

            if (perm_set_setgid("setgid_file", 1) != 0) return 2;
            if (!perm_has_setgid("setgid_file")) return 3;

            perm_file_info_t info;
            perm_get_info("setgid_file", &info);
            if (!info.mode.special.setgid) return 4;

            char buf[16];
            perm_to_symbolic(&info.mode, buf, sizeof(buf));
            if (buf[5] != 's') return 5;  // Position du x de group

            // Test sur repertoire
            mkdir("setgid_dir", 0755);

            if (perm_set_setgid("setgid_dir", 1) != 0) return 6;
            if (!perm_has_setgid("setgid_dir")) return 7;

            // Nettoyer
            perm_set_setgid("setgid_file", 0);
            unlink("setgid_file");
            chmod("setgid_dir", 0755);
            rmdir("setgid_dir");

            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.l: Setgid bit");
}
```

### Test 13: Sticky Bit (2.3.7.m)

```rust
#[test]
fn test_sticky_bit() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <sys/stat.h>
        #include <unistd.h>

        int main(void) {
            // 2.3.7.m: Sticky bit - Restrict deletion

            mkdir("sticky_dir", 0777);

            // Verifier que sticky n'est pas actif
            if (perm_has_sticky("sticky_dir")) return 1;

            // Activer
            if (perm_set_sticky("sticky_dir", 1) != 0) return 2;
            if (!perm_has_sticky("sticky_dir")) return 3;

            perm_file_info_t info;
            perm_get_info("sticky_dir", &info);
            if (!info.mode.special.sticky) return 4;
            if ((info.mode.raw_mode & S_ISVTX) == 0) return 5;

            // Notation symbolique doit montrer 't'
            char buf[16];
            perm_to_symbolic(&info.mode, buf, sizeof(buf));
            if (buf[8] != 't') return 6;  // Position du x de other

            // Desactiver
            if (perm_set_sticky("sticky_dir", 0) != 0) return 7;
            if (perm_has_sticky("sticky_dir")) return 8;

            rmdir("sticky_dir");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "2.3.7.m: Sticky bit");
}
```

### Test 14: Security Audit

```rust
#[test]
fn test_security_audit() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>

        int main(void) {
            // Test de l'audit de securite

            // Fichier world-writable
            int fd = open("insecure.txt", O_CREAT | O_WRONLY, 0666);
            close(fd);

            perm_security_audit_t audit;
            if (perm_security_audit("insecure.txt", &audit) != 0) return 1;

            if (!audit.world_writable) return 2;
            if (!audit.insecure_permissions) return 3;

            // Fichier setuid
            fd = open("setuid_audit", O_CREAT | O_WRONLY, 04755);
            close(fd);

            if (perm_security_audit("setuid_audit", &audit) != 0) return 4;
            if (!audit.setuid_enabled) return 5;

            // Fichier securise
            fd = open("secure.txt", O_CREAT | O_WRONLY, 0600);
            close(fd);

            if (perm_security_audit("secure.txt", &audit) != 0) return 6;
            if (audit.insecure_permissions) return 7;
            if (audit.world_writable) return 8;

            unlink("insecure.txt");
            unlink("setuid_audit");
            unlink("secure.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Audit de securite");
}
```

### Test 15: Integration Complete

```rust
#[test]
fn test_full_integration() {
    let result = run_c_test(r#"
        #include "perm.h"
        #include <fcntl.h>
        #include <unistd.h>
        #include <string.h>
        #include <sys/stat.h>

        int main(void) {
            // Test integre de tous les concepts 2.3.7.a-m

            // === Creer un fichier de test ===
            int fd = open("integration.txt", O_CREAT | O_WRONLY, 0644);
            close(fd);

            perm_file_info_t info;
            char symbolic[16], octal[8];

            // === 2.3.7.a: Verifier les bits rwxrwxrwx ===
            perm_get_info("integration.txt", &info);
            perm_to_symbolic(&info.mode, symbolic, sizeof(symbolic));
            if (strcmp(symbolic, "rw-r--r--") != 0) return 1;

            // === 2.3.7.b: Verifier la categorie ===
            const char* cat = perm_get_category(&info, getuid(), getgid());
            if (strcmp(cat, "user") != 0) return 2;

            // === 2.3.7.c: Tester Read ===
            perm_check_result_t check = perm_check_access("integration.txt",
                                                          PERM_ACCESS_READ);
            if (!check.allowed) return 3;

            // === 2.3.7.d: Tester Write ===
            check = perm_check_access("integration.txt", PERM_ACCESS_WRITE);
            if (!check.allowed) return 4;

            // === 2.3.7.e: Tester Execute (devrait echouer) ===
            check = perm_check_access("integration.txt", PERM_ACCESS_EXECUTE);
            if (check.allowed) return 5;

            // === 2.3.7.f: Notation octale ===
            perm_to_octal(&info.mode, octal, sizeof(octal));
            if (strcmp(octal, "644") != 0) return 6;

            // === 2.3.7.g: chmod() ===
            perm_chmod("integration.txt", 0755);
            perm_get_info("integration.txt", &info);
            if ((info.mode.raw_mode & 0777) != 0755) return 7;

            // === 2.3.7.h: fchmod() ===
            fd = open("integration.txt", O_RDWR);
            perm_fchmod(fd, 0700);
            perm_get_info_fd(fd, &info);
            if ((info.mode.raw_mode & 0777) != 0700) return 8;
            close(fd);

            // === 2.3.7.i: chown() (verification seulement) ===
            perm_get_info("integration.txt", &info);
            uid_t orig_uid = info.uid;
            // Pas de changement reel sans root

            // === 2.3.7.j: umask ===
            mode_t old_umask = perm_set_umask(0077);
            mode_t effective = perm_apply_umask(0666, 0077);
            if (effective != 0600) return 9;
            perm_set_umask(old_umask);

            // === 2.3.7.k: Setuid ===
            perm_set_setuid("integration.txt", 1);
            if (!perm_has_setuid("integration.txt")) return 10;
            perm_set_setuid("integration.txt", 0);

            // === 2.3.7.l: Setgid ===
            perm_set_setgid("integration.txt", 1);
            if (!perm_has_setgid("integration.txt")) return 11;
            perm_set_setgid("integration.txt", 0);

            // === 2.3.7.m: Sticky ===
            mkdir("test_sticky", 0777);
            perm_set_sticky("test_sticky", 1);
            if (!perm_has_sticky("test_sticky")) return 12;
            rmdir("test_sticky");

            // === Verification finale avec tous les bits speciaux ===
            perm_chmod("integration.txt", 04755);
            perm_get_info("integration.txt", &info);
            perm_to_symbolic(&info.mode, symbolic, sizeof(symbolic));
            if (strcmp(symbolic, "rwsr-xr-x") != 0) return 13;

            unlink("integration.txt");
            return 0;
        }
    "#);

    assert_eq!(result.exit_code, 0, "Integration complete 2.3.7.a-m");
}
```

---

## Bareme

### Distribution des Points (Total: 100 points)

#### Partie 1: Analyse des Permissions (25 points)

| Concept | Points | Description |
|---------|--------|-------------|
| 2.3.7.a | 6 | Parsing et representation des 9 bits rwxrwxrwx |
| 2.3.7.b | 5 | Classification User/Group/Other correcte |
| 2.3.7.c | 3 | Verification permission Read |
| 2.3.7.d | 3 | Verification permission Write |
| 2.3.7.e | 4 | Verification permission Execute (fichier et repertoire) |
| 2.3.7.f | 4 | Conversions notation octale bidirectionnelles |

#### Partie 2: Modification des Permissions (30 points)

| Concept | Points | Description |
|---------|--------|-------------|
| 2.3.7.g | 10 | Implementation chmod() avec notation symbolique et octale |
| 2.3.7.h | 5 | Implementation fchmod() |
| 2.3.7.i | 8 | Implementation chown() et variantes |
| 2.3.7.j | 7 | Gestion complete du umask |

#### Partie 3: Bits Speciaux (25 points)

| Concept | Points | Description |
|---------|--------|-------------|
| 2.3.7.k | 8 | Gestion du bit setuid |
| 2.3.7.l | 8 | Gestion du bit setgid |
| 2.3.7.m | 9 | Gestion du bit sticky |

#### Partie 4: Qualite et Robustesse (20 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Gestion des erreurs | 5 | Verification de tous les retours syscall |
| Audit de securite | 5 | Detection des permissions dangereuses |
| Pas de fuites memoire | 4 | Valgrind clean |
| Documentation | 3 | References aux concepts dans le code |
| Code propre | 3 | Style, organisation, nommage |

### Penalites

| Violation | Penalite |
|-----------|----------|
| Compilation echoue | -100 (note finale: 0) |
| Warning de compilation | -2 par warning |
| Fuite memoire | -10 |
| Segfault | -20 |
| Concept manquant (2.3.7.X non implemente) | -8 par concept |
| Fonction interdite | -10 par fonction |
| Race condition TOCTOU | -5 |

### Bonus (Maximum +10 points)

| Bonus | Points |
|-------|--------|
| Support des ACL POSIX (getfacl/setfacl) | +3 |
| Operations recursives optimisees | +2 |
| Detection des fichiers avec capabilities | +2 |
| Mode interactif avec prompts de confirmation | +3 |

---

## Fichiers a Rendre

```
ex04/
├── perm.h              # Header public de la bibliotheque
├── perm.c              # Implementation principale
├── perm_parse.c        # Parsing et conversions
├── perm_check.c        # Verification d'acces
├── perm_modify.c       # Modification des permissions
├── perm_special.c      # Bits speciaux (setuid/setgid/sticky)
├── perm_audit.c        # Audit de securite
├── perm_utils.c        # Utilitaires (uid/gid conversion)
├── Makefile            # Compilation de la bibliotheque
└── main.c              # Programme de demonstration
```

### Makefile Requis

```makefile
CC = gcc
CFLAGS = -std=c17 -Wall -Wextra -Werror -pedantic -g
LDFLAGS =

NAME = libperm.a

SRCS = perm.c perm_parse.c perm_check.c perm_modify.c \
       perm_special.c perm_audit.c perm_utils.c
OBJS = $(SRCS:.c=.o)

all: $(NAME)

$(NAME): $(OBJS)
	ar rcs $(NAME) $(OBJS)

%.o: %.c perm.h
	$(CC) $(CFLAGS) -c $< -o $@

demo: $(NAME) main.c
	$(CC) $(CFLAGS) main.c -L. -lperm $(LDFLAGS) -o perm_demo

test: $(NAME)
	$(CC) $(CFLAGS) -DTEST_MODE main.c -L. -lperm $(LDFLAGS) -o test_perm
	./test_perm

clean:
	rm -f $(OBJS) $(NAME) perm_demo test_perm

re: clean all

.PHONY: all clean re demo test
```

---

## Ressources Additionnelles

### Man Pages Essentielles
- `man 2 chmod` / `man 2 fchmod` - Modification des permissions
- `man 2 chown` / `man 2 fchown` / `man 2 lchown` - Changement de proprietaire
- `man 2 umask` - Masque de creation par defaut
- `man 2 stat` / `man 2 fstat` - Obtention des metadonnees
- `man 2 access` - Verification d'acces
- `man 3 getpwnam` / `man 3 getpwuid` - Base de donnees utilisateurs
- `man 3 getgrnam` / `man 3 getgrgid` - Base de donnees groupes

### Lecture Recommandee
- "Advanced Programming in the UNIX Environment" - Stevens & Rago, Chapitre 4
- "The Linux Programming Interface" - Kerrisk, Chapitres 15 et 17
- Documentation kernel: Documentation/filesystems/

### Representation Visuelle des Permissions

```
    Mode complet: 4 chiffres octaux
    +----+----+----+----+
    |spec|user|grp |oth |
    +----+----+----+----+

    Exemple: 4755

    4    = Special bits (setuid=4, setgid=2, sticky=1)
    7    = User (r=4 + w=2 + x=1 = 7)
    5    = Group (r=4 + x=1 = 5)
    5    = Other (r=4 + x=1 = 5)

    Representation ls -l:

    -rwsr-xr-x
    │││││││││└── other: execute
    ││││││││└─── other: no write
    │││││││└──── other: read
    ││││││└───── group: execute
    │││││└────── group: no write
    ││││└─────── group: read
    │││└──────── user: execute (s = setuid + x)
    ││└───────── user: write
    │└────────── user: read
    └─────────── type (- = regular file)
```

---

## Note Finale

Cet exercice couvre de maniere exhaustive les concepts 2.3.7.a-m du curriculum concernant les permissions Unix. L'implementation d'un gestionnaire de permissions complet permet de comprendre en profondeur:

1. **L'architecture des permissions** - Les 9 bits standard et les 3 bits speciaux
2. **Les categories d'utilisateurs** - User, Group, Other et leur evaluation
3. **Les types d'acces** - Read, Write, Execute et leurs significations selon le type de fichier
4. **Les syscalls de modification** - chmod(), fchmod(), chown() et leurs variantes
5. **Le masque de creation** - umask et son impact sur les nouveaux fichiers
6. **Les bits speciaux** - setuid, setgid et sticky et leurs implications de securite

La moulinette Rust verifiera systematiquement que chaque concept est correctement implemente et documente, avec une attention particuliere aux aspects de securite.

---

## Historique

```yaml
version: "1.0"
created: "2025-01-04"
author: "ODYSSEY Curriculum"
last_modified: "2025-01-04"
changes:
  - "Version initiale couvrant 2.3.7.a-m"
```

---

*ODYSSEY Phase 2 - Module 2.3 File Systems - Exercise 04*
