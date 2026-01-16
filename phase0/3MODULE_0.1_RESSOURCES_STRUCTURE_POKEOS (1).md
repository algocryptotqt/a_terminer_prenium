# 📚 MODULE 0.1 - LINUX & COMMAND LINE
## Ressources Externes + Exercices | Structure POKÉOS

**210 heures | 3000 XP | Couverture: 100%**

---

# 📋 TABLE DE CORRESPONDANCE POKÉOS ↔ RESSOURCES

| Section POKÉOS | Heures | Ressources Principales |
|----------------|--------|------------------------|
| Partie 1: Premiers Pas Terminal | 30h | Linux Survival, LabEx, Fireship |
| Partie 2: Permissions & Users | 25h | LabEx, KodeKloud, Bandit |
| Partie 3: Éditeurs (nano/vim) | 25h | vimtutor, Vim Adventures, LearnLinuxTV |
| Partie 4: Flux & Text Processing | 30h | HackerRank, Exercism AWK, cmdchallenge |
| Partie 5: Processus & Jobs | 25h | LabEx, devconnected, freeCodeCamp |
| Partie 6: Réseau & Cron | 25h | Bandit, Crontab.guru, SadServers |
| Partie 7: Bash Scripting | 50h | Exercism (93 ex), learnshell.org |
| Projet Final | 15h | SadServers, KillerCoda |

---

# PARTIE 1 : PREMIERS PAS DANS LE TERMINAL (30h, 450 XP)

## 0.1.1 : Le Shell

### Vidéos
| Ressource | Durée | URL |
|-----------|-------|-----|
| **Fireship** - Linux in 100 Seconds | 2.5 min | youtube.com (search "Fireship Linux 100 seconds") |
| **freeCodeCamp** - Introduction to Linux (Ch.1-3) | 1h | youtube.com/watch?v=sWbUDq4S6Y8 |
| **NetworkChuck** - you NEED to learn Linux NOW | 15 min | youtube.com/@NetworkChuck |

### Plateformes Interactives
| Plateforme | Type | URL |
|------------|------|-----|
| **Linux Survival** ⭐⭐⭐⭐⭐ | Jeu navigateur | linuxsurvival.com |
| **Terminus** ⭐⭐⭐⭐ | Text adventure | web.mit.edu/mprat/Public/web/Terminus/Web/main.html |
| **Bashcrawl** ⭐⭐⭐⭐ | Dungeon crawler | gitlab.com/slackermedia/bashcrawl |

### Exercices
```bash
# Exercice 1: Historique et raccourcis
history              # Afficher historique
!!                   # Répéter dernière commande
!grep                # Dernière commande commençant par grep
Ctrl+R               # Recherche inverse
Ctrl+L               # Clear screen
Ctrl+C               # Interrompre
Ctrl+D               # EOF/Logout

# Exercice 2: Complétion
cd /etc/sys[TAB]     # Complétion automatique
ls /u[TAB][TAB]      # Afficher possibilités
```

---

## 0.1.2 : Navigation dans le Système de Fichiers

### Ressources
| Type | Ressource | Contenu |
|------|-----------|---------|
| Jeu | **Linux Survival** Module 1-2 | pwd, ls, cd |
| Lab | **LabEx** "Linux Basic Commands" | 30+ exercices |
| Lab | **KillerCoda** Linux scenarios | Ubuntu éphémère |
| Cours | **Software Carpentry** Lesson 2 | Navigation complète |

### Exercices
```bash
# Exercice 1: Navigation de base
pwd                          # Où suis-je?
cd /var/log                  # Chemin absolu
cd ../..                     # Remonter 2 niveaux
cd ~                         # Retour home
cd -                         # Répertoire précédent

# Exercice 2: Listing avancé
ls -la                       # Tout + détails
ls -lh                       # Tailles lisibles
ls -lt                       # Tri par date
ls -R                        # Récursif
tree -L 2                    # Arborescence 2 niveaux

# Exercice 3: Chemins
echo $PWD                    # Répertoire courant
echo $HOME                   # Home directory
echo $OLDPWD                 # Répertoire précédent
```

---

## 0.1.3 : Hiérarchie du Système de Fichiers Linux (FHS)

### Vidéos ⭐ ESSENTIELLES
| Ressource | Durée | URL |
|-----------|-------|-----|
| **Fireship** - Linux Directories Explained in 100 Seconds | 2.5 min | youtube.com (1.1M vues) |
| **NetworkChuck** - Linux File System Explained | 12 min | youtube.com/@NetworkChuck |
| **freeCodeCamp** - Ch.10 File Operations | 30 min | Dans cours 6h |

### Quiz Interactifs
| Ressource | Type | URL |
|-----------|------|-----|
| **Quizlet** FHS Flashcards | Flashcards | quizlet.com/306814831 |
| **Sanfoundry** Linux MCQ | 1000+ QCM | sanfoundry.com/linux-filesystem-hierarchy-questions-answers |
| **Quiz-Maker** FHS Quiz | Quiz interactif | quiz-maker.com |

### Tableau de Référence Complet
```
/               Racine du système (root)
├── bin         Binaires essentiels (ls, cp, cat, grep)
├── sbin        Binaires système (fdisk, mount, reboot)
├── boot        Fichiers démarrage (vmlinuz, grub)
├── dev         Périphériques (/dev/sda, /dev/null)
├── etc         Configuration système
│   ├── passwd      Liste utilisateurs
│   ├── shadow      Mots de passe hashés
│   ├── group       Groupes
│   ├── sudoers     Configuration sudo
│   ├── ssh/        Configuration SSH
│   ├── cron.d/     Cron système
│   └── fstab       Points de montage
├── home        Répertoires utilisateurs (/home/alice)
├── lib         Bibliothèques partagées
├── media       Montage médias amovibles (USB, CD)
├── mnt         Montage temporaire manuel
├── opt         Logiciels optionnels tiers
├── proc        Processus (virtuel)
│   ├── cpuinfo     Infos CPU
│   ├── meminfo     Infos mémoire
│   └── [PID]/      Infos par processus
├── root        Home de root
├── run         Données runtime
├── srv         Données services (web, ftp)
├── sys         Infos système/hardware (virtuel)
├── tmp         Fichiers temporaires (effacés au reboot)
├── usr         Programmes utilisateur
│   ├── bin         Binaires utilisateur
│   ├── lib         Bibliothèques
│   ├── local/      Installés manuellement
│   └── share/      Données partagées
└── var         Données variables
    ├── log/        Logs système
    ├── cache/      Cache applications
    ├── lib/        Données applications
    └── tmp/        Temporaires persistants
```

### Quiz FHS - 30 Questions
```
Q1: Où sont les logs système?
A: /var/log

Q2: Où est la config SSH?
A: /etc/ssh/sshd_config

Q3: Où sont les binaires essentiels?
A: /bin ou /usr/bin

Q4: Fichier liste des utilisateurs?
A: /etc/passwd

Q5: Fichier mots de passe hashés?
A: /etc/shadow

Q6: Home de l'utilisateur "bob"?
A: /home/bob

Q7: Home du root?
A: /root

Q8: Fichier kernel Linux?
A: /boot/vmlinuz-*

Q9: Infos CPU?
A: /proc/cpuinfo

Q10: Infos mémoire?
A: /proc/meminfo

Q11: Périphérique premier disque?
A: /dev/sda

Q12: Périphérique "poubelle"?
A: /dev/null

Q13: Configuration DNS?
A: /etc/resolv.conf

Q14: Configuration réseau (netplan)?
A: /etc/netplan/

Q15: Fichier montages au boot?
A: /etc/fstab

Q16: Logs Apache?
A: /var/log/apache2/ ou /var/log/httpd/

Q17: Logs SSH connexions?
A: /var/log/auth.log

Q18: Point montage USB?
A: /media/username/

Q19: Binaires installés manuellement?
A: /usr/local/bin

Q20: Configuration cron système?
A: /etc/crontab ou /etc/cron.d/

Q21: Données MySQL?
A: /var/lib/mysql

Q22: Cache apt?
A: /var/cache/apt

Q23: Fichiers temp effacés au reboot?
A: /tmp

Q24: Bibliothèques 64-bit?
A: /lib64 ou /usr/lib64

Q25: Config hostname?
A: /etc/hostname

Q26: Config hosts locaux?
A: /etc/hosts

Q27: Configuration sudo?
A: /etc/sudoers

Q28: Shells disponibles?
A: /etc/shells

Q29: Timezone système?
A: /etc/timezone

Q30: Processus PID 1?
A: /proc/1/ (systemd ou init)
```

---

## 0.1.4 : Manipulation de Fichiers et Dossiers

### Ressources
| Type | Ressource | Exercices |
|------|-----------|-----------|
| Jeu | **Linux Survival** Module 3 | touch, mkdir, cp, mv, rm |
| Lab | **LabEx** "File Operations" | 20+ labs |
| Cours | **Software Carpentry** Lesson 3 | Complet |

### Exercices
```bash
# Exercice 1: Création
touch fichier.txt                    # Créer fichier vide
mkdir dossier                        # Créer dossier
mkdir -p projet/{src,docs,tests}     # Structure complète

# Exercice 2: Copie
cp fichier.txt copie.txt             # Copier fichier
cp -r dossier/ backup/               # Copier dossier récursif
cp -i fichier.txt dest/              # Demander confirmation

# Exercice 3: Déplacement/Renommage
mv fichier.txt nouveau.txt           # Renommer
mv fichier.txt /autre/chemin/        # Déplacer
mv -i source dest                    # Avec confirmation

# Exercice 4: Suppression
rm fichier.txt                       # Supprimer fichier
rm -r dossier/                       # Supprimer dossier
rm -rf dossier/                      # Forcer sans confirmation
rmdir dossier_vide/                  # Supprimer si vide

# Exercice 5: Liens
ln fichier.txt lien_dur              # Lien dur
ln -s fichier.txt lien_symbolique    # Lien symbolique
ls -l lien_symbolique                # Voir la cible
```

---

## 0.1.5 : Lecture de Fichiers

### Exercices
```bash
# Exercice 1: Affichage complet
cat fichier.txt                      # Tout afficher
cat -n fichier.txt                   # Avec numéros ligne

# Exercice 2: Pagination
less fichier.txt                     # Paginer (q pour quitter)
more fichier.txt                     # Paginer ancien

# Exercice 3: Début/Fin
head fichier.txt                     # 10 premières lignes
head -n 20 fichier.txt               # 20 premières
tail fichier.txt                     # 10 dernières
tail -n 5 fichier.txt                # 5 dernières
tail -f /var/log/syslog              # Suivre en temps réel

# Exercice 4: Comptage
wc fichier.txt                       # lignes, mots, bytes
wc -l fichier.txt                    # Lignes seulement
wc -w fichier.txt                    # Mots seulement
```

---

## 0.1.6 : Recherche

### Ressources
| Type | Ressource | Contenu |
|------|-----------|---------|
| Wargame | **Bandit** Niveaux 0-5 | find, file |
| Lab | **LabEx** "Finding Things" | find avancé |
| Cours | **Software Carpentry** Lesson 7 | Finding Things |

### Exercices
```bash
# Exercice 1: find basique
find /home -name "*.txt"             # Par nom
find . -type f                       # Fichiers seulement
find . -type d                       # Dossiers seulement
find /var -size +10M                 # Plus de 10MB

# Exercice 2: find avancé
find . -mtime -7                     # Modifiés derniers 7 jours
find . -user alice                   # Propriétaire alice
find . -perm 644                     # Permissions exactes
find . -name "*.log" -delete         # Trouver ET supprimer

# Exercice 3: find + exec
find . -name "*.sh" -exec chmod +x {} \;
find . -type f -exec grep -l "error" {} \;

# Exercice 4: Autres outils
locate fichier.txt                   # Recherche rapide (updatedb)
which python3                        # Chemin exécutable
whereis bash                         # Binaire + man + source
type ls                              # Type de commande
```

---

### Mini-projet 0.1.A : Exploration du Système

**Objectif**: Explorer le système de fichiers Linux

```bash
# Tâche a: Explorer /
cd /
ls -la
tree -L 1

# Tâche b: Trouver configs
find /etc -name "*.conf" 2>/dev/null | head -20
find /etc -type f -name "*.conf" | wc -l

# Tâche c: Créer structure projet
mkdir -p ~/projet_exploration/{src,docs,tests,logs}
touch ~/projet_exploration/README.md
tree ~/projet_exploration

# Tâche d: Manipuler fichiers
cp /etc/passwd ~/projet_exploration/docs/
head -5 ~/projet_exploration/docs/passwd
mv ~/projet_exploration/docs/passwd ~/projet_exploration/docs/users.txt

# Tâche e: Documenter
cat > ~/projet_exploration/README.md << 'EOF'
# Exploration Linux

## Ce que j'ai appris:
- / est la racine
- /etc contient les configs
- /var/log contient les logs
- /home contient les users
EOF
```

---

# PARTIE 2 : PERMISSIONS ET UTILISATEURS (25h, 350 XP)

## 0.1.7 : Utilisateurs et Groupes

### Ressources
| Type | Ressource | Contenu |
|------|-----------|---------|
| Lab | **LabEx** "User Management" | Complet |
| Lab | **KodeKloud** "Security" module | Users, groups |
| Cours | **Linux Foundation edX** Ch.17 | User Environment |

### Exercices
```bash
# Exercice 1: Infos utilisateur
whoami                               # Utilisateur actuel
id                                   # UID, GID, groupes
groups                               # Groupes de l'utilisateur
who                                  # Utilisateurs connectés
w                                    # Activité utilisateurs

# Exercice 2: Fichiers système
cat /etc/passwd                      # Liste utilisateurs
cat /etc/group                       # Liste groupes
cat /etc/shadow                      # Mots de passe (root)

# Exercice 3: Gestion utilisateurs (root)
sudo useradd -m alice                # Créer avec home
sudo passwd alice                    # Définir mot de passe
sudo usermod -aG sudo alice          # Ajouter au groupe sudo
sudo userdel -r alice                # Supprimer avec home

# Exercice 4: Gestion groupes
sudo groupadd developers             # Créer groupe
sudo usermod -aG developers bob      # Ajouter au groupe
sudo gpasswd -d bob developers       # Retirer du groupe
sudo groupdel developers             # Supprimer groupe
```

---

## 0.1.8 : Permissions de Fichiers

### Ressources
| Type | Ressource | Contenu |
|------|-----------|---------|
| Wargame | **Bandit** Niveaux 4-6 | Permissions |
| Lab | **LabEx** "File Permissions" | chmod, chown |
| Lab | **TryHackMe** Linux Fundamentals Part 2 | Permissions |

### Tableau de Référence
```
Permission  Valeur  Fichier           Dossier
---------   ------  -------           -------
r (read)      4     Lire contenu      Lister contenu
w (write)     2     Modifier          Créer/supprimer
x (execute)   1     Exécuter          Entrer (cd)

Exemples:
rwxrwxrwx = 777 (tous les droits)
rwxr-xr-x = 755 (standard exécutable)
rw-r--r-- = 644 (standard fichier)
rw------- = 600 (privé)
rwx------ = 700 (dossier privé)
```

### Exercices
```bash
# Exercice 1: Voir permissions
ls -l fichier.txt                    # -rw-r--r--
stat fichier.txt                     # Détails complets

# Exercice 2: chmod numérique
chmod 755 script.sh                  # rwxr-xr-x
chmod 644 fichier.txt                # rw-r--r--
chmod 600 secret.txt                 # rw-------
chmod 700 dossier/                   # rwx------

# Exercice 3: chmod symbolique
chmod +x script.sh                   # Ajouter exécution
chmod -w fichier.txt                 # Retirer écriture
chmod u+x,g+r fichier.txt            # User +x, Group +r
chmod a+r fichier.txt                # All +read

# Exercice 4: chown/chgrp
sudo chown alice fichier.txt         # Changer propriétaire
sudo chown alice:developers fichier.txt  # Owner + group
sudo chgrp developers fichier.txt    # Groupe seulement
sudo chown -R alice:alice dossier/   # Récursif
```

---

## 0.1.9 : Permissions Spéciales (SUID, SGID, Sticky)

### Tableau de Référence
```
Permission   Valeur   Sur Fichier              Sur Dossier
----------   ------   ----------               -----------
SUID         4000     Exécute comme owner      -
SGID         2000     Exécute comme groupe     Nouveaux fichiers héritent groupe
Sticky       1000     -                        Seul owner peut supprimer

Exemples système:
/usr/bin/passwd     -rwsr-xr-x (SUID) - permet aux users de changer leur mdp
/tmp                drwxrwxrwt (Sticky) - tous peuvent créer, seul owner supprime
```

### Exercices
```bash
# Exercice 1: Voir permissions spéciales
ls -l /usr/bin/passwd                # 's' dans user = SUID
ls -l /usr/bin/wall                  # 's' dans group = SGID  
ls -ld /tmp                          # 't' à la fin = Sticky

# Exercice 2: Trouver fichiers SUID
find / -perm -4000 2>/dev/null       # Tous SUID
find / -perm -2000 2>/dev/null       # Tous SGID

# Exercice 3: Appliquer (root)
chmod 4755 script.sh                 # SUID + rwxr-xr-x
chmod 2755 dossier/                  # SGID
chmod 1777 shared/                   # Sticky bit
chmod u+s script.sh                  # SUID symbolique
chmod g+s dossier/                   # SGID symbolique
chmod +t dossier/                    # Sticky symbolique
```

---

## 0.1.10 : sudo et sudoers

### Ressources
| Type | Ressource | Contenu |
|------|-----------|---------|
| Lab | **LabEx** "Configure Sudo Privileges" | Complet |
| Lab | **KodeKloud** Security module | sudo config |

### Exercices
```bash
# Exercice 1: Utilisation sudo
sudo commande                        # Exécuter comme root
sudo -i                              # Shell root interactif
sudo su -                            # Devenir root
sudo -u postgres psql                # Exécuter comme autre user
sudo -l                              # Lister permissions

# Exercice 2: Éditer sudoers (TOUJOURS avec visudo!)
sudo visudo                          # Éditer en sécurité

# Syntaxe sudoers:
# user    HOST=(RUNAS)  COMMANDS
alice     ALL=(ALL:ALL) ALL              # Tous droits
bob       ALL=(ALL)     /usr/bin/apt     # apt seulement
%developers ALL=(ALL)   /usr/bin/docker  # Groupe developers
alice     ALL=(ALL)     NOPASSWD: ALL    # Sans mot de passe

# Exercice 3: sudoers.d
sudo cat /etc/sudoers.d/README
# Créer fichier dans /etc/sudoers.d/ pour configs additionnelles
```

---

# PARTIE 3 : ÉDITION DE TEXTE (25h, 350 XP)

## 0.1.11 : nano (Débutant)

### Vidéos
| Ressource | Durée | URL |
|-----------|-------|-----|
| **LearnLinuxTV** - nano Crash Course | 15 min | youtube.com/@LearnLinuxTV |
| **freeCodeCamp** - Ch.11 Text Editors | 30 min | Dans cours 6h |

### Ressources Interactives
| Type | Ressource | URL |
|------|-----------|-----|
| Lab | **LabEx** "Linux Simple Text Editing" | labex.io |
| Terminal | **Webminal** tutorials | webminal.org |

### Tableau Raccourcis nano
```
Raccourci    Action
---------    ------
Ctrl+O       Sauvegarder (Write Out)
Ctrl+X       Quitter
Ctrl+K       Couper ligne
Ctrl+U       Coller
Ctrl+W       Rechercher
Ctrl+\       Rechercher/Remplacer
Ctrl+G       Aide
Ctrl+_       Aller à ligne
Alt+A        Début sélection
Alt+6        Copier sélection
Alt+U        Annuler
Alt+E        Refaire
Ctrl+C       Position curseur
```

### Exercices nano
```bash
# Exercice 1: Création et édition
nano script.sh
# Écrire:
#!/bin/bash
echo "Hello World"
# Ctrl+O, Enter pour sauvegarder
# Ctrl+X pour quitter

# Exercice 2: Recherche
nano /var/log/syslog
# Ctrl+W → taper "error" → Enter
# Ctrl+W → Enter (répéter recherche)

# Exercice 3: Copier/Coller lignes
nano fichier.txt
# Alt+A (début sélection)
# ↓↓↓ (sélectionner lignes)
# Alt+6 (copier)
# ↓↓↓ (déplacer)
# Ctrl+U (coller)

# Exercice 4: Configuration ~/.nanorc
nano ~/.nanorc
# Ajouter:
set linenumbers
set autoindent
set tabsize 4
set mouse
include "/usr/share/nano/*.nanorc"
```

### Quiz nano
```
Q1: Sauvegarder? → Ctrl+O
Q2: Quitter? → Ctrl+X
Q3: Rechercher? → Ctrl+W
Q4: Couper ligne? → Ctrl+K
Q5: Coller? → Ctrl+U
Q6: Aller ligne 50? → Ctrl+_ puis 50
Q7: Annuler? → Alt+U
Q8: Aide? → Ctrl+G
```

---

## 0.1.12 : vim (Essentiel)

### Ressources ⭐ ESSENTIELLES
| Type | Ressource | Durée/Contenu | URL |
|------|-----------|---------------|-----|
| Tutoriel | **vimtutor** | 30 min | Commande `vimtutor` |
| Interactif | **OpenVim** | 1h | openvim.com |
| Jeu | **Vim Adventures** ⭐⭐⭐⭐⭐ | 3h+ | vim-adventures.com |
| Drills | **ShortcutFoo** Vim | Illimité | shortcutfoo.com |
| Cours | **MIT Missing Semester** Editors | 1h | missing.csail.mit.edu |

### Modes vim
```
Mode       Touche    Description
----       ------    -----------
Normal     Esc       Commandes (défaut)
Insert     i,a,o     Écrire du texte
Visual     v,V,Ctrl+v Sélectionner
Command    :         Commandes Ex
```

### Commandes Essentielles
```
# Navigation
h,j,k,l     Gauche, Bas, Haut, Droite
w,b         Mot suivant/précédent
0,$         Début/Fin ligne
gg,G        Début/Fin fichier
:50         Aller ligne 50

# Édition
i           Insert avant curseur
a           Insert après curseur
o           Nouvelle ligne dessous
O           Nouvelle ligne dessus
x           Supprimer caractère
dd          Supprimer ligne
yy          Copier ligne
p           Coller après
P           Coller avant
u           Annuler
Ctrl+r      Refaire

# Recherche
/pattern    Chercher avant
?pattern    Chercher arrière
n,N         Suivant/Précédent
:%s/old/new/g  Remplacer tout

# Fichiers
:w          Sauvegarder
:q          Quitter
:wq ou :x   Sauver et quitter
:q!         Quitter sans sauver
```

### Exercices vim
```bash
# Exercice 1: vimtutor (OBLIGATOIRE)
vimtutor

# Exercice 2: Navigation
vim fichier.txt
# gg (début), G (fin), 50G (ligne 50)
# w (mot suivant), b (mot précédent)
# 0 (début ligne), $ (fin ligne)

# Exercice 3: Édition
# dd (supprimer ligne), 5dd (5 lignes)
# yy (copier ligne), 3yy (3 lignes)
# p (coller après), P (coller avant)
# u (annuler), Ctrl+r (refaire)

# Exercice 4: Recherche/Remplacement
# /error (chercher "error")
# n (suivant), N (précédent)
# :%s/foo/bar/g (remplacer tous foo par bar)
# :%s/foo/bar/gc (avec confirmation)

# Exercice 5: Sélection visuelle
# v (mode visual caractère)
# V (mode visual ligne)
# Ctrl+v (mode visual bloc)
# y (copier sélection), d (supprimer)
```

---

## 0.1.13 : Configuration vim

### .vimrc Minimal
```vim
" ~/.vimrc - Configuration vim minimale

" Affichage
set number              " Numéros de ligne
set relativenumber      " Numéros relatifs
syntax on               " Coloration syntaxique
set cursorline          " Surligner ligne courante

" Indentation
set tabstop=4           " Tab = 4 espaces
set shiftwidth=4        " Indentation = 4
set expandtab           " Tab → espaces
set autoindent          " Auto-indentation
set smartindent         " Indentation intelligente

" Recherche
set hlsearch            " Surligner résultats
set incsearch           " Recherche incrémentale
set ignorecase          " Ignorer casse
set smartcase           " Sauf si majuscule

" Divers
set mouse=a             " Support souris
set clipboard=unnamed   " Presse-papier système
set encoding=utf-8      " Encodage UTF-8
```

---

### Mini-projet 0.1.B : Maîtrise de vim

```bash
# Tâche a: Compléter vimtutor
vimtutor
# Temps: ~30 minutes

# Tâche b: Configurer .vimrc
vim ~/.vimrc
# Ajouter la config minimale ci-dessus

# Tâche c: Éditer 5 fichiers sans souris
vim fichier1.txt fichier2.txt fichier3.txt
# :n (fichier suivant), :N (précédent)
# :e fichier4.txt (ouvrir autre fichier)

# Tâche d: Rechercher/Remplacer dans gros fichier
vim /var/log/syslog
# :%s/error/ERROR/g
# :g/pattern/d (supprimer lignes avec pattern)

# Tâche e: Macros
# qa (enregistrer macro 'a')
# ... actions ...
# q (arrêter enregistrement)
# @a (rejouer macro)
# 10@a (rejouer 10 fois)
```

---

# PARTIE 4 : FLUX ET REDIRECTIONS (30h, 450 XP)

## 0.1.14-16 : Flux Standards, Redirections, Pipes

### Exercices
```bash
# Exercice 1: Flux standards
# stdin (0): Entrée
# stdout (1): Sortie
# stderr (2): Erreurs

# Exercice 2: Redirections
ls > fichier.txt                     # stdout → fichier (écrase)
ls >> fichier.txt                    # stdout → fichier (ajoute)
cat < fichier.txt                    # fichier → stdin
ls /inexistant 2> erreurs.txt        # stderr → fichier
ls /home /inexistant 2>&1 > all.txt  # stderr → stdout
ls &> tout.txt                       # stdout + stderr

# Exercice 3: Pipes
ls -l | head -5                      # Premiers résultats
cat fichier.txt | wc -l              # Compter lignes
ps aux | grep nginx                  # Filtrer processus
cat access.log | sort | uniq -c      # Compter occurrences

# Exercice 4: tee (dupliquer flux)
ls -l | tee listing.txt              # Affiche ET sauvegarde
ls -l | tee -a listing.txt           # Ajouter au fichier
```

---

## 0.1.17 : grep

### Ressources
| Type | Ressource | Exercices |
|------|-----------|-----------|
| TUI | **grepexercises** | 30+ |
| Online | **HackerRank** Text Processing | 15 |
| Online | **cmdchallenge** | 20 |

```bash
pip install grepexercises
grepexercises  # Lance l'app interactive
```

### Exercices grep
```bash
# Exercice 1: Basique
grep "error" fichier.log             # Lignes avec "error"
grep -i "error" fichier.log          # Ignorer casse
grep -v "debug" fichier.log          # Lignes SANS "debug"
grep -n "error" fichier.log          # Avec numéros ligne

# Exercice 2: Récursif
grep -r "TODO" ./src/                # Dans tous fichiers
grep -l "function" *.js              # Noms fichiers seulement
grep -c "error" *.log                # Compter par fichier

# Exercice 3: Contexte
grep -B 2 "error" fichier.log        # 2 lignes avant
grep -A 2 "error" fichier.log        # 2 lignes après
grep -C 2 "error" fichier.log        # 2 lignes avant ET après

# Exercice 4: Regex
grep -E "^[0-9]+" fichier.txt        # Commence par chiffres
grep -E "error|warning" fichier.log  # OR
grep -E "[0-9]{3}-[0-9]{4}" phones.txt  # Pattern téléphone
grep -o "http[s]*://[^ ]*" fichier.txt  # Extraire URLs
```

---

## 0.1.18 : sed

### Ressources
| Type | Ressource | Exercices |
|------|-----------|-----------|
| TUI | **sedexercises** | 30+ |
| Online | **cmdchallenge** | 10 |

```bash
pip install sedexercises
sedexercises  # Lance l'app interactive
```

### Exercices sed
```bash
# Exercice 1: Remplacement
sed 's/ancien/nouveau/' fichier.txt           # Premier de chaque ligne
sed 's/ancien/nouveau/g' fichier.txt          # Tous
sed 's/ancien/nouveau/gi' fichier.txt         # Tous, ignore casse
sed -i 's/ancien/nouveau/g' fichier.txt       # Modifier en place

# Exercice 2: Suppression
sed '/^#/d' config.txt                        # Supprimer commentaires
sed '/^$/d' fichier.txt                       # Supprimer lignes vides
sed '1,10d' fichier.txt                       # Supprimer lignes 1-10

# Exercice 3: Affichage sélectif
sed -n '5p' fichier.txt                       # Ligne 5 seulement
sed -n '10,20p' fichier.txt                   # Lignes 10-20
sed -n '/pattern/p' fichier.txt               # Lignes avec pattern

# Exercice 4: Insertion
sed '5i\Nouvelle ligne' fichier.txt           # Insérer avant ligne 5
sed '5a\Nouvelle ligne' fichier.txt           # Insérer après ligne 5
sed 's/$/;/' fichier.txt                      # Ajouter ; en fin de ligne
```

---

## 0.1.19 : awk

### Ressources
| Type | Ressource | Exercices |
|------|-----------|-----------|
| Track | **Exercism AWK** ⭐⭐⭐⭐⭐ | 87 exercices |
| TUI | **awkexercises** | 30+ |
| Online | **HackerRank** | 10 |

```bash
pip install awkexercises
awkexercises  # Lance l'app interactive
```

### Exercices awk
```bash
# Exercice 1: Colonnes
awk '{print $1}' fichier.txt                  # Première colonne
awk '{print $1, $3}' fichier.txt              # Colonnes 1 et 3
awk '{print $NF}' fichier.txt                 # Dernière colonne
awk '{print NR, $0}' fichier.txt              # Numéro + ligne

# Exercice 2: Délimiteur
awk -F: '{print $1}' /etc/passwd              # Séparateur :
awk -F',' '{print $2}' data.csv               # Séparateur ,

# Exercice 3: Conditions
awk '$3 > 100' fichier.txt                    # Si col3 > 100
awk '/error/' fichier.log                     # Lignes avec "error"
awk 'NR > 1' fichier.csv                      # Sauter header
awk 'NR >= 10 && NR <= 20' fichier.txt        # Lignes 10-20

# Exercice 4: Calculs
awk '{sum += $1} END {print sum}' nombres.txt # Somme
awk '{sum += $1} END {print sum/NR}' nb.txt   # Moyenne
awk '{count[$1]++} END {for (k in count) print k, count[k]}' log.txt

# Exercice 5: Formatage
awk '{printf "%-20s %10d\n", $1, $2}' data.txt
```

---

## 0.1.20 : Autres Outils de Texte

### Exercices
```bash
# sort
sort fichier.txt                     # Tri alphabétique
sort -n fichier.txt                  # Tri numérique
sort -r fichier.txt                  # Tri inverse
sort -k2 fichier.txt                 # Trier par colonne 2
sort -t: -k3 -n /etc/passwd          # Par UID

# uniq (après sort!)
sort fichier.txt | uniq              # Dédupliquer
sort fichier.txt | uniq -c           # Compter occurrences
sort fichier.txt | uniq -d           # Seulement doublons

# cut
cut -d: -f1 /etc/passwd              # Colonne 1, délim :
cut -c1-10 fichier.txt               # Caractères 1-10
cut -f2,4 fichier.tsv                # Colonnes 2 et 4 (tab)

# tr
echo "hello" | tr 'a-z' 'A-Z'        # Majuscules
echo "hello" | tr -d 'aeiou'         # Supprimer voyelles
cat fichier.txt | tr -s ' '          # Réduire espaces multiples
cat fichier.txt | tr '\n' ' '        # Newlines → espaces

# paste
paste fichier1.txt fichier2.txt      # Joindre côte à côte
paste -d',' f1.txt f2.txt            # Avec délimiteur

# diff
diff fichier1.txt fichier2.txt       # Différences
diff -u fichier1.txt fichier2.txt    # Format unifié
diff -y fichier1.txt fichier2.txt    # Côte à côte
```

---

### Mini-projet 0.1.C : Analyse de Logs

```bash
# Tâche a: Télécharger logs (ou utiliser /var/log/syslog)
wget https://example.com/access.log
# ou
cp /var/log/syslog ~/analyse/

# Tâche b: Compter erreurs
grep -c "error" access.log
grep -i "error\|fail" access.log | wc -l

# Tâche c: Top 10 IPs
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Tâche d: Erreurs par heure
grep "error" access.log | awk '{print $4}' | cut -d: -f2 | sort | uniq -c

# Tâche e: Rapport complet
{
    echo "=== RAPPORT D'ANALYSE ==="
    echo "Date: $(date)"
    echo ""
    echo "Total lignes: $(wc -l < access.log)"
    echo "Erreurs: $(grep -c error access.log)"
    echo ""
    echo "Top 5 IPs:"
    awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -5
} > rapport.txt
```

---

# PARTIE 5 : PROCESSUS ET JOBS (25h, 350 XP)

## 0.1.21 : Gestion des Processus

### Ressources
| Type | Ressource | Contenu |
|------|-----------|---------|
| Exercices | **devconnected** | 30 exercices processus |
| Lab | **LabEx** Process Management | 10+ labs |
| Vidéo | **LearnLinuxTV** ps/top/htop | 3 vidéos |
| Vidéo | **freeCodeCamp** Ch.9 | Processes |

### Exercices
```bash
# Exercice 1: ps
ps                                   # Processus du terminal
ps aux                               # Tous les processus
ps aux | grep nginx                  # Filtrer
ps -ef                               # Format standard
ps auxf                              # Arborescence
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head  # Custom

# Exercice 2: top/htop
top                                  # Moniteur temps réel
# Dans top:
# P = trier par CPU
# M = trier par mémoire
# k = kill processus
# q = quitter

htop                                 # Version améliorée
# F6 = trier, F9 = kill, F10 = quitter

# Exercice 3: kill
kill PID                             # SIGTERM (15)
kill -9 PID                          # SIGKILL (forcer)
kill -l                              # Liste signaux
pkill nginx                          # Par nom
killall nginx                        # Tous du même nom
pgrep -a nginx                       # Trouver PIDs

# Exercice 4: Infos processus
pgrep -a ssh                         # PIDs + command
pidof nginx                          # PIDs d'un programme
cat /proc/PID/cmdline                # Ligne de commande
cat /proc/PID/status                 # Statut détaillé
```

---

## 0.1.22 : Jobs et Background

### Exercices
```bash
# Exercice 1: Background
sleep 100 &                          # Lancer en background
[1] 12345                            # [job_number] PID

# Exercice 2: Suspend + Resume
sleep 100                            # Ctrl+Z pour suspendre
[1]+  Stopped     sleep 100
bg                                   # Reprendre en background
fg                                   # Reprendre en foreground
fg %1                                # Job spécifique

# Exercice 3: Liste jobs
jobs                                 # Tous les jobs
jobs -l                              # Avec PIDs

# Exercice 4: nohup et disown
nohup ./script.sh &                  # Ignore hangup
disown %1                            # Détacher job du terminal
nohup ./script.sh > output.log 2>&1 &  # Avec redirection
```

---

## 0.1.23 : Signaux

### Tableau de Référence
```
Signal    Num   Action par défaut       Usage
------    ---   -----------------       -----
SIGHUP    1     Terminer               Hangup (terminal fermé)
SIGINT    2     Terminer               Ctrl+C
SIGQUIT   3     Core dump              Ctrl+\
SIGKILL   9     Terminer (forcé)       Ne peut pas être ignoré
SIGTERM   15    Terminer               Arrêt propre (défaut)
SIGSTOP   19    Suspendre              Ne peut pas être ignoré
SIGTSTP   20    Suspendre              Ctrl+Z
SIGCONT   18    Reprendre              Après STOP/TSTP
```

### Exercices Signaux
```bash
# Exercice 1: Envoyer signaux
kill -TERM PID                       # Arrêt propre
kill -KILL PID                       # Forcer
kill -STOP PID                       # Suspendre
kill -CONT PID                       # Reprendre

# Exercice 2: trap dans script
#!/bin/bash
cleanup() {
    echo "Nettoyage..."
    rm -f /tmp/tempfile
    exit
}
trap cleanup SIGINT SIGTERM

echo "PID: $$"
while true; do
    echo "Running..."
    sleep 1
done
```

---

## 0.1.23b : Surveillance Système

### Exercices
```bash
# Exercice 1: Mémoire
free -h                              # Mémoire lisible
free -m                              # En MB
cat /proc/meminfo                    # Détails

# Exercice 2: Disque
df -h                                # Espace disque
df -i                                # Inodes
du -sh dossier/                      # Taille dossier
du -h --max-depth=1 /home            # Par sous-dossier

# Exercice 3: Système
uptime                               # Temps + charge
uname -a                             # Infos kernel
hostnamectl                          # Infos système
lscpu                                # Infos CPU
lsmem                                # Infos mémoire

# Exercice 4: Performance
vmstat 1 5                           # Stats VM (5 samples)
iostat 1 5                           # Stats I/O
mpstat 1 5                           # Stats CPU
```

---

# PARTIE 6 : RÉSEAU BASIQUE (25h, 350 XP)

## 0.1.24 : Commandes Réseau

### Ressources
| Type | Ressource | Contenu |
|------|-----------|---------|
| Wargame | **Bandit** 11-20 | SSH, réseau |
| Lab | **LabEx** Networking | 15+ labs |
| Lab | **SadServers** | Troubleshooting réseau |

### Exercices
```bash
# Exercice 1: Configuration IP
ip addr                              # Adresses IP
ip addr show eth0                    # Interface spécifique
ip route                             # Table routage
ip link                              # Interfaces

# Exercice 2: Connectivité
ping -c 4 google.com                 # Test ICMP
traceroute google.com                # Chemin réseau
mtr google.com                       # Traceroute amélioré

# Exercice 3: DNS
host google.com                      # DNS simple
dig google.com                       # DNS détaillé
dig +short google.com                # IP seulement
nslookup google.com                  # DNS lookup
cat /etc/resolv.conf                 # Config DNS

# Exercice 4: Connexions
ss -tuln                             # Ports ouverts
ss -tulnp                            # Avec processus
netstat -tuln                        # Ancien équivalent
lsof -i :80                          # Qui utilise port 80
```

---

## 0.1.25 : curl et wget

### Exercices
```bash
# Exercice 1: curl GET
curl https://api.example.com         # GET simple
curl -o fichier.html https://...     # Sauvegarder
curl -O https://.../fichier.zip      # Nom original
curl -I https://example.com          # Headers seulement
curl -v https://example.com          # Verbose

# Exercice 2: curl POST
curl -X POST https://api.example.com/data
curl -d "key=value" https://...      # Form data
curl -H "Content-Type: application/json" \
     -d '{"key":"value"}' https://...

# Exercice 3: wget
wget https://example.com/file.zip    # Télécharger
wget -c https://...                  # Continue download
wget -r -l 2 https://example.com     # Récursif, 2 niveaux
wget -b https://...                  # Background
```

---

## 0.1.26 : SSH

### Ressources
| Type | Ressource | Contenu |
|------|-----------|---------|
| Wargame | **Bandit** | SSH intensif |
| Lab | **LabEx** "SSH Configuration" | Complet |

### Exercices
```bash
# Exercice 1: Connexion
ssh user@host                        # Basique
ssh -p 2222 user@host                # Port custom
ssh -i ~/.ssh/key.pem user@host      # Avec clé

# Exercice 2: Clés SSH
ssh-keygen -t ed25519                # Générer clé
ssh-copy-id user@host                # Copier clé publique
cat ~/.ssh/authorized_keys           # Clés autorisées

# Exercice 3: Config SSH
cat > ~/.ssh/config << 'EOF'
Host serveur1
    HostName 192.168.1.100
    User admin
    Port 2222
    IdentityFile ~/.ssh/serveur1_key
EOF
ssh serveur1                         # Utilise config

# Exercice 4: Transfert fichiers
scp fichier.txt user@host:/chemin/   # Copier vers serveur
scp user@host:/chemin/fichier.txt .  # Copier depuis serveur
scp -r dossier/ user@host:/chemin/   # Récursif
rsync -avz dossier/ user@host:/dest/ # Sync optimisé

# Exercice 5: Tunnels
ssh -L 8080:localhost:80 user@host   # Local forward
ssh -R 9090:localhost:22 user@host   # Remote forward
ssh -D 1080 user@host                # SOCKS proxy
```

---

## 0.1.24b : Tâches Planifiées (cron)

### Ressources ⭐ ESSENTIELLES
| Type | Ressource | URL |
|------|-----------|-----|
| Outil | **Crontab.guru** ⭐⭐⭐⭐⭐ | crontab.guru |
| Exemples | Crontab.guru Examples | crontab.guru/examples.html |

### Syntaxe Cron
```
# ┌───────────── minute (0-59)
# │ ┌───────────── heure (0-23)
# │ │ ┌───────────── jour du mois (1-31)
# │ │ │ ┌───────────── mois (1-12)
# │ │ │ │ ┌───────────── jour de la semaine (0-7, 0=7=dimanche)
# │ │ │ │ │
# * * * * * commande

# Exemples:
* * * * *       # Chaque minute
0 * * * *       # Chaque heure
0 0 * * *       # Chaque jour à minuit
0 0 * * 0       # Chaque dimanche
0 9 * * 1-5     # 9h, lundi-vendredi
*/5 * * * *     # Toutes les 5 minutes
0 */2 * * *     # Toutes les 2 heures
```

### Exercices cron
```bash
# Exercice 1: Gestion crontab
crontab -e                           # Éditer
crontab -l                           # Lister
crontab -r                           # Supprimer tout

# Exercice 2: Exemples pratiques
# Backup quotidien à 2h
0 2 * * * /home/user/backup.sh >> /var/log/backup.log 2>&1

# Nettoyage /tmp chaque dimanche
0 3 * * 0 find /tmp -type f -mtime +7 -delete

# Health check toutes les 5 min
*/5 * * * * /usr/local/bin/healthcheck.sh

# Exercice 3: Raccourcis
@reboot     /home/user/startup.sh    # Au démarrage
@daily      /home/user/daily.sh      # Chaque jour
@hourly     /home/user/hourly.sh     # Chaque heure
@weekly     /home/user/weekly.sh     # Chaque semaine
@monthly    /home/user/monthly.sh    # Chaque mois
```

---

# PARTIE 7 : SCRIPTS BASH (50h, 700 XP)

## Ressources Principales

| Type | Ressource | Exercices | URL |
|------|-----------|-----------|-----|
| Track | **Exercism Bash** ⭐⭐⭐⭐⭐ | 93 exercices | exercism.org/tracks/bash |
| Interactif | **learnshell.org** | 40+ | learnshell.org |
| Challenges | **HackerRank Shell** | 65+ | hackerrank.com/domains/shell |
| Cours | **MIT Missing Semester** | Exercices | missing.csail.mit.edu |

---

## 0.1.27 : Variables

```bash
# Exercice 1: Variables simples
nom="Alice"                          # PAS d'espace autour de =
age=25
echo "Je suis $nom, j'ai $age ans"
echo "Je suis ${nom}, j'ai ${age} ans"  # Explicite

# Exercice 2: Variables spéciales
echo "Script: $0"                    # Nom du script
echo "Arg 1: $1"                     # Premier argument
echo "Tous args: $@"                 # Tous les arguments
echo "Nombre args: $#"               # Nombre d'arguments
echo "PID: $$"                       # PID du script
echo "Exit code: $?"                 # Code retour dernière cmd

# Exercice 3: Lecture
read -p "Votre nom: " username
echo "Bonjour $username"
read -s -p "Mot de passe: " password  # -s = silencieux

# Exercice 4: Export
export PATH="$PATH:/opt/bin"         # Variable d'environnement
export MY_VAR="value"
readonly CONSTANTE="immuable"        # Constante

# Exercice 5: Substitution
date_actuelle=$(date +%Y-%m-%d)
fichiers=$(ls | wc -l)
echo "Date: $date_actuelle, Fichiers: $fichiers"
```

---

## 0.1.28 : Conditions

```bash
# Exercice 1: if/else
if [ $age -ge 18 ]; then
    echo "Majeur"
else
    echo "Mineur"
fi

# Exercice 2: Tests fichiers
if [ -f "$fichier" ]; then
    echo "Fichier existe"
elif [ -d "$fichier" ]; then
    echo "C'est un dossier"
elif [ -e "$fichier" ]; then
    echo "Existe (autre type)"
else
    echo "N'existe pas"
fi

# Exercice 3: Tests strings
if [ -z "$var" ]; then               # Vide
    echo "Variable vide"
fi
if [ -n "$var" ]; then               # Non-vide
    echo "Variable non-vide"
fi
if [ "$str1" = "$str2" ]; then       # Égaux
    echo "Égaux"
fi

# Exercice 4: Tests numériques
# -eq (equal), -ne (not equal)
# -lt (less than), -le (less or equal)
# -gt (greater than), -ge (greater or equal)
if [ $a -lt $b ]; then
    echo "$a < $b"
fi

# Exercice 5: Opérateurs logiques
if [ $age -ge 18 ] && [ $age -lt 65 ]; then
    echo "Adulte actif"
fi
if [ "$pays" = "FR" ] || [ "$pays" = "BE" ]; then
    echo "Francophone"
fi

# Exercice 6: case
case $choix in
    1|one)   echo "Option 1";;
    2|two)   echo "Option 2";;
    quit|q)  echo "Au revoir"; exit;;
    *)       echo "Choix invalide";;
esac
```

---

## 0.1.29 : Boucles

```bash
# Exercice 1: for sur liste
for fruit in pomme orange banane; do
    echo "J'aime $fruit"
done

# Exercice 2: for sur fichiers
for file in *.txt; do
    echo "Traitement: $file"
done

# Exercice 3: for C-style
for ((i=0; i<10; i++)); do
    echo $i
done

# Exercice 4: for avec séquence
for i in {1..10}; do
    echo $i
done
for i in $(seq 1 2 10); do           # 1, 3, 5, 7, 9
    echo $i
done

# Exercice 5: while
count=0
while [ $count -lt 5 ]; do
    echo $count
    ((count++))
done

# Exercice 6: Lecture fichier
while IFS= read -r line; do
    echo "Ligne: $line"
done < fichier.txt

# Exercice 7: until
until [ $count -eq 10 ]; do
    ((count++))
done

# Exercice 8: break/continue
for i in {1..10}; do
    [ $i -eq 5 ] && continue         # Sauter 5
    [ $i -eq 8 ] && break            # Arrêter à 8
    echo $i
done
```

---

## 0.1.30 : Fonctions

```bash
# Exercice 1: Fonction simple
dire_bonjour() {
    echo "Bonjour $1!"
}
dire_bonjour "Alice"

# Exercice 2: Return value
est_pair() {
    if [ $(($1 % 2)) -eq 0 ]; then
        return 0                      # True
    else
        return 1                      # False
    fi
}
if est_pair 4; then
    echo "4 est pair"
fi

# Exercice 3: Variables locales
ma_fonction() {
    local var_locale="local"
    global_var="global"
    echo "Local: $var_locale"
}
ma_fonction
echo "Global: $global_var"
# echo "Local hors fonction: $var_locale"  # Vide!

# Exercice 4: Retourner string (via echo)
obtenir_date() {
    echo $(date +%Y-%m-%d)
}
resultat=$(obtenir_date)
echo "Date: $resultat"
```

---

## 0.1.31 : Commandes Avancées

```bash
# Exercice 1: Arithmétique
result=$((5 + 3))
((count++))
((total = a + b * c))

# Exercice 2: xargs
find . -name "*.log" | xargs rm
cat urls.txt | xargs -I {} curl {}
echo "1 2 3" | xargs -n 1 echo

# Exercice 3: getopts
while getopts "vf:o:" opt; do
    case $opt in
        v) verbose=1;;
        f) file="$OPTARG";;
        o) output="$OPTARG";;
        ?) echo "Usage: $0 [-v] [-f file] [-o output]"; exit 1;;
    esac
done

# Exercice 4: trap
cleanup() {
    rm -f /tmp/tempfile.$$
    echo "Nettoyage effectué"
}
trap cleanup EXIT SIGINT SIGTERM
```

---

## 0.1.31b : Tableaux Bash

```bash
# Exercice 1: Tableau indexé
arr=(pomme orange banane)
echo ${arr[0]}                       # pomme
echo ${arr[@]}                       # Tous
echo ${#arr[@]}                      # Longueur: 3
arr+=(cerise)                        # Ajouter

# Exercice 2: Itération
for fruit in "${arr[@]}"; do
    echo $fruit
done

# Exercice 3: Tableau associatif
declare -A ages
ages[alice]=25
ages[bob]=30
echo ${ages[alice]}                  # 25
echo ${!ages[@]}                     # Clés
echo ${ages[@]}                      # Valeurs
```

---

## 0.1.32 : Gestion d'Erreurs

```bash
#!/bin/bash
# Exercice 1: Options strictes
set -e           # Exit on error
set -u           # Undefined variable = error
set -o pipefail  # Pipe failure
# Ou en une ligne:
set -euo pipefail

# Exercice 2: || et &&
mkdir dossier || echo "Échec création"
cd dossier && echo "Dans dossier"
command || { echo "Erreur"; exit 1; }

# Exercice 3: trap ERR
handle_error() {
    echo "Erreur ligne $1"
    exit 1
}
trap 'handle_error $LINENO' ERR

# Exercice 4: Code retour
if ! command; then
    echo "Command failed with exit code $?"
fi
```

---

### Mini-projet 0.1.D : Script d'Administration

```bash
#!/bin/bash
# admin_toolkit.sh - Script d'administration complet

set -euo pipefail

# === FONCTIONS ===

backup_home() {
    local backup_dir="/backup/$(date +%Y%m%d)"
    mkdir -p "$backup_dir"
    tar -czf "$backup_dir/home.tar.gz" /home
    echo "Backup créé: $backup_dir/home.tar.gz"
}

create_user() {
    local username=$1
    if id "$username" &>/dev/null; then
        echo "Utilisateur $username existe déjà"
        return 1
    fi
    sudo useradd -m "$username"
    echo "Utilisateur $username créé"
}

rotate_logs() {
    local log_dir="/var/log/myapp"
    find "$log_dir" -name "*.log" -mtime +30 -exec gzip {} \;
    find "$log_dir" -name "*.gz" -mtime +90 -delete
    echo "Rotation des logs effectuée"
}

health_check() {
    echo "=== HEALTH CHECK ==="
    echo "Uptime: $(uptime -p)"
    echo "Load: $(cat /proc/loadavg | cut -d' ' -f1-3)"
    echo "Memory: $(free -h | awk '/Mem:/ {print $3 "/" $2}')"
    echo "Disk: $(df -h / | awk 'NR==2 {print $5 " used"}')"
}

# === MENU ===

show_menu() {
    echo ""
    echo "=== Admin Toolkit ==="
    echo "1) Backup /home"
    echo "2) Créer utilisateur"
    echo "3) Rotation logs"
    echo "4) Health check"
    echo "5) Quitter"
    echo ""
}

# === MAIN ===

while true; do
    show_menu
    read -p "Choix: " choice
    
    case $choice in
        1) backup_home;;
        2) read -p "Nom utilisateur: " user; create_user "$user";;
        3) rotate_logs;;
        4) health_check;;
        5) echo "Au revoir!"; exit 0;;
        *) echo "Choix invalide";;
    esac
done
```

---

# PROJET FINAL : SERVER SETUP AUTOMATISÉ (15h, 1500 XP)

## Ressources Recommandées
| Type | Ressource | Usage |
|------|-----------|-------|
| Lab | **SadServers** | Troubleshooting |
| Lab | **KillerCoda** | Ubuntu sandbox |
| Lab | **LabEx** | Labs guidés |

## Structure du Projet
```
server-setup/
├── setup.sh           # Script principal
├── packages.txt       # Liste des paquets
├── users.csv          # Utilisateurs à créer
├── config/
│   ├── sshd_config    # Config SSH
│   ├── ufw.rules      # Règles firewall
│   └── cron.jobs      # Tâches cron
├── logs/              # Logs installation
└── README.md          # Documentation
```

## Script Principal (Exemple)
```bash
#!/bin/bash
# setup.sh - Server Setup Automatisé
set -euo pipefail

LOG_FILE="logs/setup_$(date +%Y%m%d_%H%M%S).log"
exec 1> >(tee -a "$LOG_FILE") 2>&1

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"; }

# Installation paquets
log "Installation des paquets..."
while read -r package; do
    apt install -y "$package"
done < packages.txt

# Création utilisateurs
log "Création des utilisateurs..."
while IFS=',' read -r username groups shell; do
    useradd -m -s "$shell" -G "$groups" "$username"
done < users.csv

# Configuration SSH
log "Configuration SSH..."
cp config/sshd_config /etc/ssh/sshd_config
systemctl restart sshd

# Firewall
log "Configuration firewall..."
ufw --force reset
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw --force enable

# Cron jobs
log "Installation cron jobs..."
crontab config/cron.jobs

# Vérification finale
log "Health check..."
./health_check.sh

log "Setup terminé avec succès!"
```

---

# 🌐 RESSOURCES COMPLÉMENTAIRES (CATALOGUE COMPLET)

## Terminaux en Ligne Supplémentaires

| Plateforme | URL | Spécificité | Gratuit |
|------------|-----|-------------|---------|
| **sandbox.bio** | sandbox.bio | Debian 12, Carpentries intégré | ✅ 100% |
| **LinuxZoo** | linuxzoo.net | ROOT access, CentOS/Kali, VNC | ✅ 100% |
| **Replit** | replit.com/languages/bash | IDE cloud + shell, collaboration | ✅ |
| **JSLinux** | bellard.org/jslinux | Émulateur JS complet | ✅ 100% |
| **Copy.sh** | copy.sh/v86 | Multi-OS (Arch, FreeDOS) rapide | ✅ 100% |
| **OnlineGDB** | onlinegdb.com/online_bash_shell | IDE + Bash | ✅ 100% |
| **JDoodle** | jdoodle.com/test-bash-shell-script-online | Multi-terminal | ✅ 100% |
| **Paiza.io** | paiza.io/en/languages/bash | Bash + collaboration | ✅ 100% |
| **TutorialsPoint** | tutorialspoint.com/execute_bash_online.php | Simple bash | ✅ 100% |
| **OneCompiler** | onecompiler.com/bash | Illimité | ✅ 100% |

**Playground sandbox.bio**: https://sandbox.bio/playgrounds/terminal

---

## Exercices Supplémentaires

### LeetCode - Shell ⭐⭐⭐
```
URL: leetcode.com/problemset/shell/
Gratuit: ✅ 100%
Exercices: 4 problèmes Bash
```

| Problème | Difficulté |
|----------|------------|
| Word Frequency | Medium |
| Valid Phone Numbers | Easy |
| Transpose File | Medium |
| Tenth Line | Easy |

### Linux Journey ⭐⭐⭐⭐
```
URL: linuxjourney.com (maintenant hébergé sur LabEx)
Gratuit: ✅ 100%
Type: Cours progressif avec quiz
```

| Section | Contenu |
|---------|---------|
| Grasshopper | Command Line, Text-Fu |
| Journeyman | Devices, Filesystem |
| Networking Nomad | DNS, Routing |

---

## Sites de Référence & Documentation

### GeeksforGeeks ⭐⭐⭐⭐
```
URL: geeksforgeeks.org/linux-unix/
Type: Tutoriels + exercices
```

Articles clés:
- Linux File Hierarchy Structure
- Linux Process Management Command Cheat Sheet
- sudo Command in Linux with Examples

### DigitalOcean ⭐⭐⭐⭐⭐
```
URL: digitalocean.com/community/tutorials
Type: Tutoriels professionnels détaillés
```

Tutoriels clés:
- How To Use ps, kill, and nice to Manage Processes in Linux
- Linux Command Line Primer
- An Introduction to Linux Basics

### Linuxize ⭐⭐⭐⭐
```
URL: linuxize.com
Type: Guides pratiques
```

### Tecmint ⭐⭐⭐⭐
```
URL: tecmint.com
Type: Tutoriels Linux/sysadmin
```

### PhoenixNAP ⭐⭐⭐⭐
```
URL: phoenixnap.com/kb
Type: Knowledge base IT
```

Articles clés:
- Linux Network Commands
- Linux sudo Command

### Red Hat ⭐⭐⭐⭐⭐
```
URL: redhat.com/sysadmin
Type: Articles professionnels
```

Articles clés:
- jobs, bg, fg Commands
- ping, traceroute, netstat

---

## Outils Spécialisés Supplémentaires

### ExplainShell ⭐⭐⭐⭐⭐
```
URL: explainshell.com
Type: Décomposition visuelle de commandes
Usage: Coller une commande → explication détaillée de chaque argument
```

Exemple:
```bash
# Coller cette commande sur explainshell.com:
find /home -type f -name "*.log" -mtime +7 -exec rm {} \;
# → Explication de chaque argument avec références man pages
```

### Regex Tools

| Outil | URL | Type | Gratuit |
|-------|-----|------|---------|
| **RegexOne** | regexone.com | 15 leçons + 8 problèmes | ✅ 100% |
| **Regex Crossword** | regexcrossword.com | Puzzle game regex | ✅ 100% |
| **Regexle** | regexle.com | Wordle-style regex quotidien | ✅ 100% |
| **regex101** | regex101.com | Testeur + explications temps réel | ✅ 100% |

---

## Cours en Ligne Supplémentaires

### Codecademy ⭐⭐⭐⭐
```
URL: codecademy.com/learn/learn-the-command-line
Gratuit: Partiel (exercices de base gratuits)
```

| Cours | Durée | Contenu |
|-------|-------|---------|
| Learn the Command Line | 8h | Navigation, fichiers, redirection |
| Intro to the Command Line | 2h | Bases absolues |
| Learn Bash Scripting | 4h | Scripts, automatisation |

### DataCamp ⭐⭐⭐⭐
```
URL: datacamp.com/courses/introduction-to-shell
Gratuit: 1er chapitre gratuit
```

| Cours | Durée | Contenu |
|-------|-------|---------|
| Introduction to Shell | 4h | Commandes Unix, pipes, filtres |
| Data Processing in Shell | 4h | curl, wget, traitement données |
| Introduction to Bash Scripting | 4h | Variables, boucles, fonctions |

### Udacity ⭐⭐⭐
```
URL: udacity.com/course/linux-command-line-basics--ud595
Gratuit: ✅ 100%
```

| Cours | Durée | Contenu |
|-------|-------|---------|
| Linux Command Line Basics | 1 semaine | Terminal, shell, filesystem |
| Shell Workshop | 2h | Introduction rapide |

### Coursera ⭐⭐⭐⭐
```
URL: coursera.org/courses?query=linux
Gratuit: Audit gratuit / Certificat payant
```

| Cours | Provider | Contenu |
|-------|----------|---------|
| Linux for Beginners with Hands-on Labs | KodeKloud | Labs browser-based |
| Hands-on Introduction to Linux Commands | IBM | ETL, scripts, cron |
| Linux Fundamentals | LearnQuest | LFCA prep |
| Tools of the Trade: Linux and SQL | Google | DevOps focus |

---

## Chaînes YouTube Supplémentaires

### DistroTube ⭐⭐⭐⭐
```
URL: youtube.com/@DistroTube
Abonnés: 400K+
Spécialité: CLI avancé, Vim, tiling window managers
```

| Série | Contenu |
|-------|---------|
| Vim tutorials | Série complète Vim avancé |
| Linux CLI | Commandes avancées |
| Bash scripting | Scripts pratiques |

### The Linux Experiment ⭐⭐⭐⭐
```
URL: youtube.com/@TheLinuxExperiment
Abonnés: 300K+
Spécialité: Linux desktop, news, tutoriels
```

### Chris Titus Tech ⭐⭐⭐⭐
```
URL: youtube.com/@ChrisTitusTech
Abonnés: 1M+
Spécialité: Tweaks, scripts, optimisations Linux
```

---

## Chaînes YouTube Avancées/Spécialisées

### Corey Schafer ⭐⭐⭐⭐
```
URL: youtube.com/@coreyms
Spécialité: Python, Linux, développement
```
- Linux Process Management tutorials
- Terminal productivity

### The Primeagen ⭐⭐⭐⭐⭐
```
URL: youtube.com/@ThePrimeagen
Spécialité: Vim avancé, productivité CLI
```
- Vim As Your Editor (série avancée)
- Terminal workflow

### ThoughtBot ⭐⭐⭐⭐
```
URL: youtube.com/@thoughtbot
Spécialité: Développement professionnel
```
- Mastering Vim (professionnel)
- tmux tutorials

### Gary Explains ⭐⭐⭐⭐
```
URL: youtube.com/@GaryExplains
Spécialité: Linux internals, text processing
```
- Text Processing in Linux
- Linux Internals

### Luke Smith ⭐⭐⭐⭐
```
URL: youtube.com/@LukeSmithxyz
Spécialité: sed, awk, scripts minimalistes
```
- sed tutorials
- awk tutorials

### David Bombal ⭐⭐⭐⭐⭐
```
URL: youtube.com/@davidbombal
Spécialité: Networking, Linux pour hackers
```
- Linux Networking (playlist)
- Network troubleshooting

### The Cyber Mentor ⭐⭐⭐⭐
```
URL: youtube.com/@TCMSecurityAcademy
Spécialité: Linux for ethical hackers
```
- Linux for Ethical Hackers (cours complet)

### LiveOverflow ⭐⭐⭐⭐⭐
```
URL: youtube.com/@LiveOverflow
Spécialité: Linux security, CTF
```
- Linux Security
- Binary exploitation basics

---

## Source des Exercices TUI (grep/sed/awk)

### learnbyexample ⭐⭐⭐⭐⭐
```
URL: learnbyexample.github.io/interactive-grep-sed-awk-exercises
Auteur: Sundeep Agarwal
Gratuit: ✅ 100%
```

Installation:
```bash
pip install grepexercises sedexercises awkexercises

# Lancement
grepexercises   # 30+ exercices grep interactifs
sedexercises    # 30+ exercices sed interactifs
awkexercises    # 30+ exercices awk interactifs
```

Livres gratuits de l'auteur:
- CLI text processing with GNU grep and ripgrep
- CLI text processing with GNU sed
- CLI text processing with GNU awk
- CLI text processing with GNU Coreutils

---

## Outils CLI Essentiels

### ShellCheck ⭐⭐⭐⭐⭐
```
URL: shellcheck.net / github.com/koalaman/shellcheck
Type: Linter pour scripts Bash
```

```bash
# Installation
apt install shellcheck  # ou brew install shellcheck

# Usage
shellcheck mon_script.sh
```

### tldr ⭐⭐⭐⭐⭐
```
URL: tldr.sh / github.com/tldr-pages/tldr
Type: Man pages simplifiées
```

```bash
# Installation
npm install -g tldr  # ou pip install tldr

# Usage
tldr tar
tldr find
```

### cheat.sh ⭐⭐⭐⭐⭐
```
URL: cheat.sh
Type: Cheatsheets en ligne de commande
```

```bash
# Usage sans installation
curl cheat.sh/tar
curl cheat.sh/grep
curl cheat.sh/awk
```

---

## Tutoriels & Guides Supplémentaires

### W3Schools Bash ⭐⭐⭐
```
URL: w3schools.com/bash
Gratuit: ✅ 100%
Type: Tutoriel + Quiz "Try it yourself"
Exercices: w3schools.com/bash/bash_exercises.php
```

### It's FOSS ⭐⭐⭐⭐
```
URL: itsfoss.com
Type: Tutoriels Linux accessibles
```

Articles populaires:
- Nano Editor Guide
- Linux Commands for Beginners
- Bash Scripting Tutorial

### wooledge (Greg's Wiki) ⭐⭐⭐⭐⭐
```
URL: mywiki.wooledge.org
Type: Référence Bash ULTIME
```

Pages essentielles:
- BashGuide - Guide complet Bash
- BashFAQ - Questions fréquentes
- BashPitfalls - Erreurs courantes à éviter

---

## Vim Ressources Supplémentaires

### vim.rtorr ⭐⭐⭐⭐⭐
```
URL: vim.rtorr.com
Type: Vim Cheat Sheet interactive
```

### VimGenius ⭐⭐⭐⭐
```
URL: vimgenius.com
Type: Flashcards Vim spaced repetition
```

### Upcase (thoughtbot) ⭐⭐⭐⭐
```
URL: thoughtbot.com/upcase/vim
Type: Cours Vim professionnel
Gratuit: Partiellement
```

---

## Chaînes YouTube Tutoriels & Apps

### ProgrammingKnowledge ⭐⭐⭐
```
URL: youtube.com/@ProgrammingKnowledge
Spécialité: Tutoriels Linux détaillés
```

- nano tutorial Linux
- Shell Scripting Tutorial (28 vidéos)

### Traversy Media ⭐⭐⭐⭐
```
URL: youtube.com/@TraversyMedia
Spécialité: Crash courses
```

- Shell Scripting Crash Course (~30 min)

### Enki ⭐⭐⭐
```
URL: enki.com (app mobile)
Type: Microlearning quotidien
```

- Linux track disponible

---

## Podcasts

### Command Line Heroes ⭐⭐⭐⭐⭐
```
URL: redhat.com/en/command-line-heroes
Type: Podcast Red Hat sur l'histoire de la tech
Gratuit: ✅ 100%
```

Épisodes recommandés:
- The Origins of Linux
- Bash Shell
- DevOps history

---

# 📊 RÉCAPITULATIF COUVERTURE

| Partie | Heures | Couverture | Ressources Principales |
|--------|--------|------------|------------------------|
| 1. Terminal & Navigation | 30h | ✅ 100% | Linux Survival, Fireship FHS |
| 2. Permissions & Users | 25h | ✅ 100% | LabEx, Bandit |
| 3. nano & vim | 25h | ✅ 100% | vimtutor, Vim Adventures, LearnLinuxTV |
| 4. Flux & Text Processing | 30h | ✅ 100% | Exercism AWK, HackerRank, TUI exercises |
| 5. Processus & Jobs | 25h | ✅ 100% | devconnected, LabEx, freeCodeCamp |
| 6. Réseau & Cron | 25h | ✅ 100% | Bandit, Crontab.guru, SadServers |
| 7. Bash Scripting | 50h | ✅ 100% | Exercism (93 ex), learnshell.org |
| Projet Final | 15h | ✅ 100% | SadServers, KillerCoda |

**TOTAL: 225h | Couverture 100%**

---

# 🔗 LIENS RAPIDES PAR PARTIE

```
# PARTIE 1 - Terminal
linuxsurvival.com
youtube.com (Fireship "Linux Directories 100 Seconds")
labex.io/learn/linux

# PARTIE 2 - Permissions
overthewire.org/wargames/bandit
labex.io (User Management labs)

# PARTIE 3 - Éditeurs
vimtutor (commande)
openvim.com
vim-adventures.com
youtube.com/@LearnLinuxTV (nano)

# PARTIE 4 - Text Processing
exercism.org/tracks/awk (87 exercices)
pip install grepexercises sedexercises awkexercises
hackerrank.com/domains/shell
cmdchallenge.com

# PARTIE 5 - Processus
devconnected.com/30-linux-processes-exercises-for-sysadmins
labex.io (Process Management)

# PARTIE 6 - Réseau & Cron
crontab.guru
overthewire.org/wargames/bandit (SSH)
sadservers.com

# PARTIE 7 - Bash Scripting
exercism.org/tracks/bash (93 exercices)
learnshell.org
hackerrank.com/domains/shell
missing.csail.mit.edu
```

---

# 🔗 LIENS RAPIDES - RESSOURCES COMPLÉMENTAIRES

```
# TERMINAUX EN LIGNE
sandbox.bio/playgrounds/terminal
linuxzoo.net (ROOT access)
replit.com/languages/bash
bellard.org/jslinux
copy.sh/v86
onlinegdb.com/online_bash_shell
jdoodle.com/test-bash-shell-script-online

# EXERCICES SUPPLÉMENTAIRES
leetcode.com/problemset/shell (4 problèmes)
linuxjourney.com (hébergé sur LabEx)
w3schools.com/bash/bash_exercises.php

# OUTILS CLI ESSENTIELS
shellcheck.net (linter Bash)
tldr.sh (man pages simplifiées)
cheat.sh (curl cheat.sh/commande)

# OUTILS SPÉCIALISÉS
explainshell.com (décomposer commandes)
crontab.guru (éditeur cron)
regexone.com (15 leçons regex)
regexcrossword.com (puzzle game)
regex101.com (testeur regex)

# SITES DE RÉFÉRENCE
geeksforgeeks.org/linux-unix
digitalocean.com/community/tutorials
linuxize.com
tecmint.com
phoenixnap.com/kb
redhat.com/sysadmin
itsfoss.com
mywiki.wooledge.org (référence Bash ultime)

# VIM RESSOURCES
vim.rtorr.com (cheat sheet)
vimgenius.com (flashcards)
thoughtbot.com/upcase/vim

# COURS GRATUITS
codecademy.com/learn/learn-the-command-line
datacamp.com/courses/introduction-to-shell
udacity.com/course/linux-command-line-basics--ud595
coursera.org (Linux KodeKloud, IBM)

# YOUTUBE CHAÎNES PRINCIPALES
youtube.com/@Fireship
youtube.com/@LearnLinuxTV
youtube.com/@NetworkChuck
youtube.com/@DistroTube
youtube.com/@TheLinuxExperiment
youtube.com/@ChrisTitusTech

# YOUTUBE CHAÎNES AVANCÉES
youtube.com/@coreyms (Corey Schafer)
youtube.com/@ThePrimeagen (Vim avancé)
youtube.com/@thoughtbot (Vim pro)
youtube.com/@GaryExplains (Text processing)
youtube.com/@LukeSmithxyz (sed/awk)
youtube.com/@davidbombal (Networking)
youtube.com/@TCMSecurityAcademy (Cyber Mentor)
youtube.com/@LiveOverflow (Security)
youtube.com/@ProgrammingKnowledge (Tutorials)
youtube.com/@TraversyMedia (Crash courses)

# PODCASTS
redhat.com/en/command-line-heroes

# EXERCICES TUI
pip install grepexercises sedexercises awkexercises
learnbyexample.github.io/interactive-grep-sed-awk-exercises
```

---

# 📈 STATISTIQUES FINALES

| Métrique | Valeur |
|----------|--------|
| Plateformes terminaux | 16+ |
| Cours en ligne | 12+ |
| Chaînes YouTube | 17 |
| Sites de référence | 10+ |
| Outils CLI | 5+ |
| Exercices catalogués | 500+ |
| Outils spécialisés | 12+ |
| Podcasts | 1 |
| Couverture Module 0.1 | **100%** |

---

*Document généré le 5 janvier 2026*
*Structure: POKÉOS Module 0.1 | Couverture: 100% | Toutes ressources incluses*
*Dernière vérification: Session 8 - Vérification exhaustive complète*
