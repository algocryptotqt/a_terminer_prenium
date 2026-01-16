# Module 0.2 : Git & Development Tools
## Ressources d'Apprentissage Interactif — Guide Complet

> **POKÉOS** — 140h, 2000 XP  
> **Prérequis** : Module 0.1 Linux (commandes terminal)  
> **Débloque** : Module 0.0.B Méthodologie

---

## 📋 Table des Matières

1. [Plateformes Interactives Globales](#plateformes-interactives-globales)
2. [Partie 1 : Fondamentaux Git (35h, 500 XP)](#partie-1--fondamentaux-git-35h-500-xp)
3. [Partie 2 : Branches et Merge (30h, 450 XP)](#partie-2--branches-et-merge-30h-450-xp)
4. [Partie 3 : Git Remote et GitHub (35h, 500 XP)](#partie-3--git-remote-et-github-35h-500-xp)
5. [Partie 4 : Git Avancé (20h, 300 XP)](#partie-4--git-avancé-20h-300-xp)
6. [Partie 5 : CI/CD avec GitHub Actions (20h, 250 XP)](#partie-5--cicd-avec-github-actions-20h-250-xp)
7. [Projet Final (15h, 1000 XP)](#projet-final-module-02--projet-collaboratif-15h-1000-xp)

---

## Plateformes Interactives Globales

Ces plateformes couvrent plusieurs sections du module. À utiliser tout au long de l'apprentissage.

### Plateformes avec Terminal Intégré ⭐⭐⭐

| Plateforme | Description | URL | Sections Couvertes |
|------------|-------------|-----|-------------------|
| **LabEx** | 79 labs Git + Playground Ubuntu | https://labex.io/courses/git-practice-labs | 0.2.1-0.2.23 |
| **LabEx Main** | Plateforme principale | https://labex.io/learn | Tous |
| **LabEx VMs & Playgrounds** | Machines virtuelles | https://labex.io/tutorials/linux-online-virtual-machines-and-playgrounds-593595 | Tous |
| **LabEx Learn Git** | Débutant complet | https://labex.io/courses/learn-git-from-scratch | 0.2.1-0.2.6 |
| **LabEx Playground** | Sandbox libre | https://labex.io/tutorials/git-online-git-playground-593602 | Tous |
| **LabEx Exercises** | Exercices variés | https://labex.io/exercises/git | 0.2.1-0.2.20 |
| **LabEx Tutorials** | Tous tutoriels | https://labex.io/tutorials/category/git | Tous |
| **LabEx Free Labs** | Labs gratuits | https://labex.io/free-labs/git | Tous |
| **LabEx CI/CD** | Actions labs | https://labex.io/learn/git | 0.2.21-0.2.23 |
| **Codecademy** | Cours interactifs + terminal | https://www.codecademy.com/learn/learn-git | 0.2.1-0.2.16 |
| **Killercoda** | Scénarios terminal (ex-Katacoda) | https://killercoda.com/git | 0.2.1-0.2.20 |
| **Killercoda Main** | Plateforme principale | https://killercoda.com/ | Tous |
| **Killercoda Examples** | Exemples scénarios | https://killercoda.com/examples | Documentation |
| **Killercoda Scenarios Repo** | Code source scénarios | https://github.com/killercoda/scenarios-git | 0.2.1-0.2.20 |
| **Killercoda Creators** | Créer ses scénarios | https://killercoda.com/creators | Documentation |
| **GitHub Skills** | Cours officiels sur vrais repos | https://skills.github.com | 0.2.12-0.2.23 |
| **GitHub Skills Main** | Page principale | https://skills.github.com/ | Tous |
| **Microsoft Learn** | Modules certifiés gratuits | https://learn.microsoft.com/training/github | 0.2.21-0.2.23 |
| **Microsoft Learn: Git Path** | Parcours Git complet | https://learn.microsoft.com/en-us/training/paths/intro-to-vc-git/ | 0.2.1-0.2.16 |
| **Microsoft Learn: Actions Path** | Parcours Actions complet | https://learn.microsoft.com/en-us/training/paths/github-actions/ | 0.2.21-0.2.23 |

### Outils de Visualisation ⭐⭐⭐

| Outil | Description | URL | Sections Couvertes |
|-------|-------------|-----|-------------------|
| **Learn Git Branching** | Simulation visuelle branches/merge/rebase | https://learngitbranching.js.org | 0.2.7-0.2.11, 0.2.17-0.2.18 |
| **Learn Git Branching (alt)** | URL alternative | https://pcottle.github.io/learnGitBranching/ | 0.2.7-0.2.11 |
| **Oh My Git!** | Jeu + terminal temps réel | https://ohmygit.org | 0.2.1-0.2.11 |
| **A Grip On Git** | Tutoriel visuel scroll-based | https://agripongit.vincenttunru.com | 0.2.1-0.2.5 |
| **Visualizing Git** | Graphe commits interactif | https://git-school.github.io/visualizing-git | 0.2.3-0.2.10 |

### Exercices Pratiques Locaux ⭐⭐⭐

| Ressource | Description | URL | Sections Couvertes |
|-----------|-------------|-----|-------------------|
| **Git Katas** | 40+ exercices à cloner | https://github.com/eficode-academy/git-katas | 0.2.1-0.2.20 |
| **Git Exercises (AGH)** | 23 exercices auto-validés | https://gitexercises.fracz.com | 0.2.1-0.2.11 |
| **W3Schools Git** | Tutorial + Quiz + Exercices | https://www.w3schools.com/git | 0.2.1-0.2.20 |
| **LabEx Git Practice Challenges** | Challenges avancés | https://github.com/labex-labs/git-practice-challenges | 0.2.1-0.2.23 |
| **GitLab Fundamentals** | Labs officiels GitLab | https://handbook.gitlab.com/handbook/customer-success/professional-services-engineering/education-services/gitbasicshandson/ | 0.2.1-0.2.16 |
| **Gojo (SaleMove)** ⭐ | Git Katas alternatifs | https://github.com/salemove/gojo | 0.2.7-0.2.18 |
| **GitHub Minesweeper** ⭐⭐ | Workflow Git avec bot | https://profy.dev/project/github-minesweeper | 0.2.12-0.2.15 |

### Plateformes Alternatives

| Ressource | Description | URL |
|-----------|-------------|-----|
| **Bitbucket Cloud Tutorial** | Git sur Bitbucket | https://www.atlassian.com/git/tutorials |
| **GitLab Git Tutorials** | Git sur GitLab | https://docs.gitlab.com/ee/tutorials/ |
| **The Turing Way** ⭐ | Guide interactif Git | https://book.the-turing-way.org/reproducible-research/vcs/vcs-git-interactive.html |

### Cours Vidéo Gratuits

| Cours | Durée | URL |
|-------|-------|-----|
| **Udemy: Git Expert 4 Hours** | 4h | https://www.udemy.com/course/git-expert-4-hours |
| **Udemy: Git Started with GitHub** | 30min | https://www.udemy.com/course/git-started-with-github |
| **Udemy: Advanced GIT Course** | 3h | https://www.udemy.com/course/advanced-git-course |
| **Udemy: Git & GitHub Bootcamp** | 17h | https://www.udemy.com/course/git-and-github-bootcamp/ |
| **Udemy: GitHub Ultimate** | 6h | https://www.udemy.com/course/github-ultimate/ |
| **Udemy: Complete Git GitHub Copilot** | 8h | https://www.udemy.com/course/github-git/ |
| **freeCodeCamp: Git & GitHub Crash Course** | 1h | https://www.freecodecamp.org/news/git-and-github-crash-course |
| **freeCodeCamp: Git for Professionals** | 1h | https://www.freecodecamp.org/news/git-for-professionals |
| **freeCodeCamp: Git & GitHub for Beginners** | 1h | https://www.freecodecamp.org/news/git-and-github-for-beginners/ |
| **freeCodeCamp: Complete Guide** | 2h | https://www.freecodecamp.org/news/guide-to-git-github-for-beginners-and-experienced-devs/ |
| **freeCodeCamp: Crash Course Beginners** | 1h | https://www.freecodecamp.org/news/git-and-github-crash-course-for-beginners/ |
| **freeCodeCamp: Learn Git Basics** | 1h | https://www.freecodecamp.org/news/learn-git-basics/ |
| **freeCodeCamp: Git Through Gamification** ⭐ | Article | https://www.freecodecamp.org/news/learn-git-through-gamification/ |
| **freeCodeCamp: GitHub Actions Guide** | Article | https://www.freecodecamp.org/news/learn-to-use-github-actions-step-by-step-guide/ |

---

## 🎮 Jeux Linux/Terminal (Bonus Prérequis)

Ces jeux enseignent les commandes terminal utiles pour Git :

| Jeu | Description | URL | Niveau |
|-----|-------------|-----|--------|
| **OverTheWire Bandit** ⭐⭐⭐ | Wargame SSH + Linux | https://overthewire.org/wargames/bandit/ | Débutant→Avancé |
| **Terminus (MIT)** ⭐⭐⭐ | Aventure textuelle Linux | https://web.mit.edu/mprat/Public/web/Terminus/Web/main.html | Débutant |
| **Bashcrawl** ⭐⭐ | Dungeon crawler en Bash | https://gitlab.com/slackermedia/bashcrawl | Débutant |
| **CLI Murder Mystery** ⭐⭐ | Enquête en ligne de commande | https://github.com/veltman/clmystery | Intermédiaire |
| **Command Challenge** ⭐⭐⭐ | Défis one-liner | https://cmdchallenge.com/ | Tous niveaux |
| **Linux Survival** ⭐⭐ | Tutoriel interactif | https://linuxsurvival.com/ | Débutant |
| **Vim Adventures** ⭐⭐⭐ | Apprendre Vim en jouant | https://vim-adventures.com/ | Débutant |
| **Command Line Heroes Bash** | Quiz speed | https://www.redhat.com/en/command-line-heroes/bash/index.html | Tous |

## 🎮 Jeux Git Spécifiques

| Jeu | Description | URL | Concepts |
|-----|-------------|-----|----------|
| **Learn Git Branching** ⭐⭐⭐ | Visualiseur + niveaux | https://learngitbranching.js.org/ | Branches, merge, rebase |
| **Oh My Git!** ⭐⭐⭐ | Jeu cartes + terminal | https://ohmygit.org/ | Tous concepts |
| **Git Adventure Game** ⭐⭐ | Aventure textuelle Bloomberg | https://github.com/bloomberg/git-adventure-game | Navigation Git |
| **GitMastery** ⭐⭐ | Plateforme gamifiée | https://www.gitmastery.me | Progression + rewards |
| **Devlands** (en dev) ⭐ | Monde 3D voxel Git | https://initialcommit.com/blog/im-making-a-git-game | Visualisation |
| **Git Exercises (AGH)** ⭐⭐ | Exercices interactifs | https://gitexercises.fracz.com/ | Pratique |

---

# Partie 1 : Fondamentaux Git (35h, 500 XP)

---

## 0.2.1 : Concepts de Base

| Concept | Description |
|---------|-------------|
| VCS | Version Control System |
| Repository | Dossier suivi par Git |
| Commit | Snapshot du code |
| Branch | Ligne de développement |
| Merge | Fusionner des branches |
| Remote | Dépôt distant |
| Clone | Copier un dépôt |
| Push/Pull | Envoyer/Recevoir |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **A Grip On Git** ⭐ | Tutoriel visuel scrollable | https://agripongit.vincenttunru.com |
| **Oh My Git!** ⭐ | Jeu + visualisation | https://ohmygit.org |
| **Codecademy: Learn Git Introduction** | Cours interactif 2h | https://www.codecademy.com/learn/learn-git-introduction |
| **LabEx: Git Basics** | Labs pratiques | https://labex.io/courses/learn-git-from-scratch |
| **W3Schools: Git Tutorial** | Tutorial + exercices | https://www.w3schools.com/git/git_intro.asp |
| **Git Official Book Ch.1** | Documentation | https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control |

### Git Katas Associés
```bash
cd git-katas/basic-commits
source setup.sh && cat README.md
```

---

## 0.2.2 : Configuration

| Commande | Usage |
|----------|-------|
| `git config --global user.name "Nom"` | Définir nom |
| `git config --global user.email "email"` | Définir email |
| `git config --global init.defaultBranch main` | Branche par défaut |
| `git config --global core.editor vim` | Éditeur |
| `git config --list` | Voir configuration |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **LabEx: Git Configuration Lab** | Lab pratique | https://labex.io/tutorials/git-git-config-management-387471 |
| **Codecademy: Set Up Git** | Exercice guidé | https://www.codecademy.com/learn/learn-git/modules/learn-git-git-workflow |
| **W3Schools: Git Getting Started** | Tutorial | https://www.w3schools.com/git/git_getstarted.asp |
| **Atlassian: Git Config** | Guide complet | https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-config |
| **GitHub Docs: Set up Git** | Officiel | https://docs.github.com/en/get-started/quickstart/set-up-git |

---

## 0.2.3 : Initialisation et Commits

| Commande | Usage |
|----------|-------|
| `git init` | Créer repo |
| `git clone URL` | Cloner repo |
| `git status` | État actuel |
| `git add fichier` | Staging |
| `git commit -m "message"` | Commit avec message |
| `git log` | Historique |
| `git log --oneline` | Historique compact |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Learn Git Branching: Main 1-2** ⭐ | Simulation | https://learngitbranching.js.org |
| **Git Katas: basic-commits** ⭐ | Exercice local | git-katas/basic-commits |
| **Oh My Git!: Levels 1-5** | Jeu | https://ohmygit.org |
| **Git Exercises: master** | Auto-validé | https://gitexercises.fracz.com |
| **LabEx: First Git Repository** | Lab | https://labex.io/tutorials/git-your-first-git-lab-92739 |
| **W3Schools: Git New Files** | Tutorial | https://www.w3schools.com/git/git_new_files.asp |

### Commandes à Pratiquer
```bash
# Créer un repo
mkdir mon-projet && cd mon-projet
git init

# Premier commit
echo "# Mon Projet" > README.md
git add README.md
git commit -m "Initial commit"

# Voir historique
git log --oneline --graph
```

---

## 0.2.4 : Staging Area

| Commande | Usage |
|----------|-------|
| `git add fichier` | Ajouter au staging |
| `git add .` | Tout ajouter |
| `git restore --staged fichier` | Retirer du staging |
| `git diff` | Voir changements non staged |
| `git diff --staged` | Voir changements staged |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Git Katas: basic-staging** ⭐ | Exercice local | git-katas/basic-staging |
| **A Grip On Git: Staging Section** | Visuel | https://agripongit.vincenttunru.com |
| **Learn Git Branching: Level 3** | Simulation | https://learngitbranching.js.org |
| **Codecademy: Git Workflow** | Interactif | https://www.codecademy.com/learn/learn-git/modules/learn-git-git-workflow |
| **Visualizing Git** | Graphe interactif | https://git-school.github.io/visualizing-git |
| **W3Schools: Git Staging** | Tutorial | https://www.w3schools.com/git/git_staging_environment.asp |

### Git Katas
```bash
cd git-katas/basic-staging
source setup.sh
# Suivre les instructions du README
```

---

## 0.2.5 : Annuler des Changements

| Commande | Usage |
|----------|-------|
| `git restore fichier` | Annuler modifications |
| `git reset --soft HEAD~1` | Annuler commit (garder changements) |
| `git reset --hard HEAD~1` | Annuler commit (perdre changements) |
| `git commit --amend` | Modifier dernier commit |
| `git revert HASH` | Créer commit inverse |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Learn Git Branching: Reversing Changes** ⭐ | Simulation | https://learngitbranching.js.org/?locale=en_US |
| **Git Katas: basic-revert** ⭐ | Exercice | git-katas/basic-revert |
| **Git Katas: reset** | Exercice | git-katas/reset |
| **Git Katas: amend** | Exercice | git-katas/amend |
| **Git Exercises: fix-old-typo** | Auto-validé | https://gitexercises.fracz.com |
| **Atlassian: Undoing Changes** | Guide | https://www.atlassian.com/git/tutorials/undoing-changes |
| **W3Schools: Git Undo** | Tutorial | https://www.w3schools.com/git/git_reset.asp |

### Exercice Pratique
```bash
# Créer des commits à annuler
echo "test1" > file.txt && git add . && git commit -m "commit 1"
echo "test2" >> file.txt && git add . && git commit -m "commit 2"

# Soft reset (garde les changements)
git reset --soft HEAD~1

# Amend (modifier message)
git commit --amend -m "nouveau message"

# Revert (créer commit inverse)
git revert HEAD
```

---

## 0.2.6 : .gitignore

| Pattern | Usage |
|---------|-------|
| `fichier.txt` | Fichier spécifique |
| `*.log` | Extension |
| `dossier/` | Dossier |
| `!important.log` | Exception (négation) |
| `**/logs` | Pattern récursif |
| `/build` | Seulement à la racine |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **globster.xyz** ⭐⭐⭐ | Visualiseur patterns temps réel | https://globster.xyz |
| **gitignore.io (Toptal)** ⭐⭐⭐ | Générateur 571+ templates | https://www.toptal.com/developers/gitignore |
| **GitIgnore.pro Validator** ⭐⭐ | Test patterns + fichiers | https://gitignore.pro/tools/validator |
| **GitIgnore.pro Generator** | + Audit + Merge + Diff | https://gitignore.pro |
| **gitignore-exercise** ⭐ | Auto-graded via Actions | https://github.com/201758/gitignore-exercise |
| **W3Schools gitignore** | Tutorial + exercices | https://www.w3schools.com/git/git_ignore.asp |
| **W3Docs gitignore + Quiz** | Tutorial + quiz | https://www.w3docs.com/learn-git/gitignore.html |
| **globitor CLI** ⭐ | CLI interactif patterns | https://github.com/ranyitz/globitor |
| **gitmatch Python** | Lib test patterns | https://gitmatch.readthedocs.io/ |

### Documentation

| Ressource | URL |
|-----------|-----|
| **Git Official Docs** | https://git-scm.com/docs/gitignore |
| **git check-ignore** | https://git-scm.com/docs/git-check-ignore |
| **Atlassian Guide** | https://www.atlassian.com/git/tutorials/saving-changes/gitignore |
| **GitHub gitignore repo** | https://github.com/github/gitignore |
| **Cheatsheet patterns** | https://gist.github.com/jstnlvns/ebaa046fae16543cc9efc7f24bcd0e31 |
| **GitIgnore.pro Advanced Guide** ⭐ | https://gitignore.pro/guides/advanced-gitignore-patterns |
| **GeeksforGeeks gitignore** | https://www.geeksforgeeks.org/git/git-ignore-and-gitignore/ |

### Patterns à Maîtriser
```gitignore
# Fichiers spécifiques
secret.txt
.env
.env.local

# Extensions
*.log
*.tmp
*.pyc
__pycache__/

# Dossiers
node_modules/
build/
dist/
.venv/

# Négation (exception)
*.log
!important.log

# Patterns récursifs
**/debug.log
logs/**/

# Racine seulement
/TODO.md
```

### Exercice : gitignore-exercise
```bash
# Cloner le template
gh repo create mon-gitignore-exercise --template 201758/gitignore-exercise
git clone <votre-repo>

# Objectifs :
# 1. Ignorer tous les fichiers commençant par 'z'
# 2. Ignorer .env
# 3. Ignorer artifacts/ à la racine

# Commit et push pour validation automatique
git add .gitignore && git commit -m "Add gitignore" && git push
```

---

## Mini-Projet 0.2.A : Historique Propre

| Tâche | Objectif | Ressource |
|-------|----------|-----------|
| Créer repo | `git init` | LabEx Playground |
| 10 commits | Messages clairs | Git Katas: basic-commits |
| .gitignore | Complet pour Python | gitignore.io |
| Utiliser amend | Corriger erreur | Git Katas: amend |
| Utiliser revert | Annuler proprement | Git Katas: basic-revert |

### Checklist Validation
- [ ] Repository initialisé
- [ ] 10+ commits avec messages descriptifs
- [ ] .gitignore Python complet (venv, __pycache__, .pyc, etc.)
- [ ] Au moins 1 `git commit --amend` utilisé
- [ ] Au moins 1 `git revert` utilisé
- [ ] `git log --oneline` montre historique propre

---

# Partie 2 : Branches et Merge (30h, 450 XP)

---

## 0.2.7 : Gestion des Branches

| Commande | Usage |
|----------|-------|
| `git branch` | Lister branches locales |
| `git branch -a` | Toutes les branches |
| `git branch feature` | Créer branche |
| `git checkout -b feature` | Créer et switch |
| `git switch main` | Changer de branche |
| `git branch -d feature` | Supprimer branche |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **Learn Git Branching: Main 3-4** ⭐⭐⭐ | Simulation visuelle | https://learngitbranching.js.org |
| **Git Katas: basic-branching** ⭐ | Exercice local | git-katas/basic-branching |
| **Oh My Git!: Branch levels** | Jeu | https://ohmygit.org |
| **Git Exercises: chase-branch** | Auto-validé | https://gitexercises.fracz.com |
| **Codecademy: Git Branching** | Cours 3h | https://www.codecademy.com/learn/learn-git-branching-and-collaboration |
| **Visualizing Git** | Graphe interactif | https://git-school.github.io/visualizing-git |
| **W3Schools: Git Branch** | Tutorial | https://www.w3schools.com/git/git_branch.asp |

### Learn Git Branching Levels
```
Niveaux recommandés :
- Main > Introduction Sequence > 3. Branching in Git
- Main > Introduction Sequence > 4. Merging in Git
- Main > Ramping Up > 1. Detach yo' HEAD
```

---

## 0.2.8 : Merge

| Commande | Usage |
|----------|-------|
| `git merge feature` | Merge simple (fast-forward possible) |
| `git merge --no-ff feature` | Merge avec commit de merge |
| `git merge --abort` | Annuler merge en cours |
| `git log --graph --oneline` | Visualiser historique |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Learn Git Branching: Main 4** ⭐⭐⭐ | Simulation merge | https://learngitbranching.js.org |
| **Git Katas: ff-merge** ⭐ | Fast-forward | git-katas/ff-merge |
| **Git Katas: 3-way-merge** ⭐ | 3-way merge | git-katas/3-way-merge |
| **Visualizing Git** | Voir graphe merge | https://git-school.github.io/visualizing-git |
| **Atlassian: Git Merge** | Guide complet | https://www.atlassian.com/git/tutorials/using-branches/git-merge |
| **W3Schools: Git Merge** | Tutorial | https://www.w3schools.com/git/git_branch_merge.asp |

### Exercice Pratique
```bash
# Créer branche feature
git checkout -b feature/login
echo "login code" > login.py
git add . && git commit -m "Add login"

# Retour main et merge
git checkout main
git merge --no-ff feature/login -m "Merge feature/login"

# Visualiser
git log --graph --oneline --all
```

---

## 0.2.9 : Résolution de Conflits

| Étape | Description |
|-------|-------------|
| Identifier | `git status` montre les conflits |
| Ouvrir fichier | Voir marqueurs `<<<<<<<`, `=======`, `>>>>>>>` |
| Résoudre | Éditer manuellement |
| Add | `git add fichier` |
| Commit | `git commit` |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **Git Katas: merge-conflict** ⭐⭐⭐ | Exercice pratique | git-katas/merge-conflict |
| **GitHub Skills: Resolve merge conflicts** ⭐⭐ | Cours officiel | https://github.com/skills/resolve-merge-conflicts |
| **Learn Git Branching: Merge Conflicts** | Explication | https://learngitbranching.js.org |
| **Git Exercises: merge-conflict** | Auto-validé | https://gitexercises.fracz.com |
| **Atlassian: Merge Conflicts** | Guide | https://www.atlassian.com/git/tutorials/using-branches/merge-conflicts |
| **W3Schools: Git Conflicts** | Tutorial | https://www.w3schools.com/git/git_branch_merge.asp |

### Simulation Conflit
```bash
# Setup conflit
git checkout -b branch-a
echo "Version A" > file.txt && git add . && git commit -m "A"

git checkout main
echo "Version Main" > file.txt && git add . && git commit -m "Main"

# Créer conflit
git merge branch-a
# CONFLICT!

# Résoudre
cat file.txt  # Voir marqueurs
# Éditer et choisir version
echo "Version fusionnée" > file.txt
git add file.txt
git commit -m "Resolve conflict"
```

---

## 0.2.10 : Rebase

| Commande | Usage |
|----------|-------|
| `git rebase main` | Rebaser sur main |
| `git rebase -i HEAD~3` | Rebase interactif |
| `git rebase --abort` | Annuler |
| `git rebase --continue` | Continuer après résolution |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **Learn Git Branching: Rebase** ⭐⭐⭐ | Simulation visuelle | https://learngitbranching.js.org |
| **Git Katas: rebase-branch** ⭐ | Exercice basique | git-katas/rebase-branch |
| **Git Katas: advanced-rebase-interactive** ⭐ | Rebase -i | git-katas/advanced-rebase-interactive |
| **Git Katas: squashing** | Squash commits | git-katas/squashing |
| **Git Rebase Tutorial** | Tutorial interactif | https://github.com/ianmiell/git-rebase-tutorial |
| **Atlassian: Git Rebase** | Guide complet | https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase |

### Learn Git Branching Levels
```
Niveaux recommandés :
- Main > Introduction Sequence > 4. Rebase Introduction
- Main > Ramping Up > 3. Relative Refs
- Main > Moving Work Around > 1. Cherry-pick Intro
- Main > Moving Work Around > 2. Interactive Rebase Intro
```

### Rebase Interactif
```bash
# Créer plusieurs commits
git checkout -b feature
echo "1" > f.txt && git add . && git commit -m "WIP 1"
echo "2" >> f.txt && git add . && git commit -m "WIP 2"
echo "3" >> f.txt && git add . && git commit -m "WIP 3"

# Squash en 1 commit
git rebase -i HEAD~3
# Changer "pick" en "squash" pour les 2 derniers
# Sauvegarder et éditer le message
```

---

## 0.2.11 : Stash

| Commande | Usage |
|----------|-------|
| `git stash` | Sauvegarder changements |
| `git stash list` | Lister stashes |
| `git stash pop` | Récupérer et supprimer |
| `git stash apply` | Récupérer seulement |
| `git stash drop` | Supprimer |
| `git stash show -p` | Voir contenu |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Git Katas: basic-stash** ⭐ | Exercice | git-katas/basic-stash |
| **Git Exercises: save-your-work** | Auto-validé | https://gitexercises.fracz.com |
| **Atlassian: Git Stash** | Guide | https://www.atlassian.com/git/tutorials/saving-changes/git-stash |
| **W3Schools: Git Stash** | Tutorial | https://www.w3schools.com/git/git_stash.asp |

### Exercice Pratique
```bash
# Faire des modifications
echo "work in progress" > wip.txt

# Stash pour changer de branche
git stash -m "WIP: feature login"

# Changer de branche
git checkout autre-branche
# ... faire quelque chose ...

# Revenir et récupérer
git checkout main
git stash pop
```

---

## Mini-Projet 0.2.B : Workflow Branches

| Tâche | Objectif | Ressource |
|-------|----------|-----------|
| Créer feature branch | `git checkout -b` | Learn Git Branching |
| Plusieurs commits | Sur la feature | Git Katas |
| Merge dans main | Sans fast-forward | Git Katas: 3-way-merge |
| Créer conflit | Et le résoudre | Git Katas: merge-conflict |
| Utiliser stash | Sauvegarder WIP | Git Katas: basic-stash |
| Rebase interactif | Squash commits | Git Katas: squashing |

### Checklist Validation
- [ ] Branche feature créée et switchée
- [ ] 3+ commits sur la feature
- [ ] Merge --no-ff réalisé
- [ ] Conflit créé et résolu manuellement
- [ ] Stash utilisé pour sauvegarder WIP
- [ ] Rebase -i avec squash effectué
- [ ] `git log --graph` montre historique correct

---

# Partie 3 : Git Remote et GitHub (35h, 500 XP)

---

## 0.2.12 : Remotes

| Commande | Usage |
|----------|-------|
| `git remote -v` | Voir remotes |
| `git remote add origin URL` | Ajouter remote |
| `git remote set-url origin URL` | Modifier URL |
| `git remote remove origin` | Supprimer remote |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Learn Git Branching: Remote** ⭐⭐⭐ | Simulation | https://learngitbranching.js.org/?locale=en_US |
| **GitHub Skills: Introduction** ⭐ | Cours officiel | https://github.com/skills/introduction-to-github |
| **Codecademy: Learn GitHub** | Cours 2h | https://www.codecademy.com/learn/learn-github-introduction |
| **LabEx: Git Remote Labs** | Labs | https://labex.io/tutorials/git |
| **W3Schools: Git Remote** | Tutorial | https://www.w3schools.com/git/git_remote_add.asp |

### Learn Git Branching - Remote
```
Niveaux recommandés :
- Remote > Push & Pull > 1. Clone Intro
- Remote > Push & Pull > 2. Remote Branches
- Remote > Push & Pull > 3. Git Fetch
```

---

## 0.2.13 : Push et Pull

| Commande | Usage |
|----------|-------|
| `git push origin main` | Push vers remote |
| `git push -u origin main` | Set upstream |
| `git push --force-with-lease` | Force push safe |
| `git pull` | Pull (fetch + merge) |
| `git pull --rebase` | Pull avec rebase |
| `git fetch` | Récupérer sans merge |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Learn Git Branching: Remote 4-7** ⭐⭐⭐ | Push/Pull simulation | https://learngitbranching.js.org |
| **Git Katas: fetch** | Exercice | git-katas/fetch |
| **Git Katas: pull** | Exercice | git-katas/pull |
| **GitHub Skills: Introduction** | Hands-on | https://github.com/skills/introduction-to-github |
| **W3Schools: Git Push/Pull** | Tutorial | https://www.w3schools.com/git/git_push_to_remote.asp |

### Learn Git Branching - Remote
```
Niveaux recommandés :
- Remote > Push & Pull > 4. Git Pull
- Remote > Push & Pull > 5. Faking Teamwork
- Remote > Push & Pull > 6. Git Push
- Remote > Push & Pull > 7. Diverged History
```

---

## 0.2.14 : Configuration SSH pour GitHub ⭐

| Étape | Commande |
|-------|----------|
| Générer clé | `ssh-keygen -t ed25519 -C "email@example.com"` |
| Ajouter à l'agent | `ssh-add ~/.ssh/id_ed25519` |
| Copier publique | `cat ~/.ssh/id_ed25519.pub` |
| Ajouter à GitHub | Settings > SSH Keys |
| Tester | `ssh -T git@github.com` |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **GitHub Docs Official** ⭐⭐⭐ | Guide officiel | https://docs.github.com/en/authentication/connecting-to-github-with-ssh |
| **The Odin Project** ⭐⭐ | Tutoriel guidé complet | https://www.theodinproject.com/lessons/foundations-setting-up-git |
| **freeCodeCamp Multi-Accounts** ⭐⭐ | SSH config multi-comptes | https://www.freecodecamp.org/news/manage-multiple-github-accounts-the-ssh-way |
| **LabEx Linux Playground** ⭐ | Terminal pour pratiquer | https://labex.io/tutorials/linux-online-linux-terminal-and-playground-372915 |
| **Webminal** | Terminal Linux gratuit | https://www.webminal.org |
| **HSF Training SSH** | Tutorial complet + crypto | https://hsf-training.github.io/hsf-training-ssh-webpage/aio/index.html |

### Labs Pratiques Hands-On

| Ressource | Type | URL |
|-----------|------|-----|
| **A Cloud Guru SSH Lab** ⭐⭐⭐ | Lab environnement réel | https://acloudguru.com/hands-on-labs/using-ssh-keys-for-secure-access |
| **Linux Academy SSH Lab** | Lab guidé | https://linuxacademy.com/hands-on-lab/2cbbd745-3440-421e-9d18-70b3355b5139/ |
| **IoT-LAB SSH Tutorial** | Tuto + validation | https://www.iot-lab.info/legacy/tutorials/ssh-access/index.html |
| **sandbox.bio Terminal** | Terminal browser | https://sandbox.bio/tutorials/terminal-basics |
| **ShellNGN** | SSH client web | https://shellngn.com/ |
| **OnWorks SSH** | Emulateur Linux | https://www.onworks.net/programs/ssh-online |
| **SysAdmin Simulator** ⭐⭐ | Jeu tickets sysadmin | https://github.com/rmcmillan34/sysadmin-sim |

### Guides Multi-Comptes SSH

| Ressource | URL |
|-----------|-----|
| **DEV.to Guide** | https://dev.to/mr_mornin_star/how-to-manage-multiple-github-accounts-on-your-machine-using-ssh |
| **GitHub Gist (oanhnn)** | https://gist.github.com/oanhnn/80a89405ab9023894df7 |
| **GitHub Gist (jexchan)** | https://gist.github.com/jexchan/2351996 |
| **Jeff Brown Tech** | https://jeffbrown.tech/multiple-github-accounts-ssh |
| **Andrew Stiefel** | https://andrewstiefel.com/working-multiple-github-accounts |
| **Coderwall SSH** | https://coderwall.com/p/7smjkq/multiple-ssh-keys-for-different-accounts-on-github-or-gitlab |
| **TheServerSide SSH** | https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/GitHub-SSH-Key-Setup-Config-Ubuntu-Linux |
| **Hatica SSH Guide** | https://www.hatica.io/blog/how-to-configure-github-ssh-keys/ |
| **RMauro SSH** | https://rmauro.dev/github-ssh-key-authentication-and-ssh-config/ |

### Workshops Universitaires

| Ressource | URL |
|-----------|-----|
| **NSRC SSH Exercises** ⭐⭐ | https://nsrc.org/workshops/ws-files/2011/sanog17/exercises/exercise-ssh-key.html |
| **Bristol Uni SSH** | https://cs-uob.github.io/COMSM0085/exercises/part1/posix1/ssh.html |
| **CS559 UWisc** | https://cs559.github.io/559Tutorials/tools-and-setup/git-ssh |
| **PacNOG SSH Exercises** | https://www.pacnog.org/pacnog3/day1/linux/ssh-exercises.html |
| **Adam The Automator GitLab SSH** | https://adamtheautomator.com/gitlab-ssh/ |

### Documentation Supplémentaire

| Ressource | URL |
|-----------|-----|
| **DigitalOcean SSH Essentials** | https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys |
| **DataCamp SSH Keys** | https://www.datacamp.com/tutorial/ssh-keys |
| **Atlassian SSH Tutorial** | https://www.atlassian.com/git/tutorials/git-ssh |
| **Happy Git with R** | https://happygitwithr.com/ssh-keys |
| **Kinsta SSH Guide** | https://kinsta.com/blog/generate-ssh-key/ |
| **Heidelberg SSH Tutorial** | https://zah.uni-heidelberg.de/it-guide/ssh-tutorial-linux |
| **InMotion SSH Tutorial** | https://www.inmotionhosting.com/support/server/ssh/ssh-tutorial-for-beginners/ |
| **GeeksforGeeks SSH GitHub** ⭐ | https://www.geeksforgeeks.org/git/using-github-with-ssh-secure-shell/ |
| **GeeksforGeeks Add SSH Key** | https://www.geeksforgeeks.org/git/how-to-add-ssh-key-to-your-github-account/ |

### Configuration Multi-Comptes
```bash
# ~/.ssh/config

# Compte personnel
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes

# Compte travail
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes
```

### Commandes Essentielles
```bash
# Générer clé Ed25519
ssh-keygen -t ed25519 -C "email@example.com"

# Démarrer agent et ajouter clé
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Tester connexion
ssh -T git@github.com

# Copier clé publique
cat ~/.ssh/id_ed25519.pub | pbcopy  # macOS
cat ~/.ssh/id_ed25519.pub | xclip   # Linux
```

---

## 0.2.15 : Fork et Pull Request

| Étape | Description |
|-------|-------------|
| Fork | Bouton Fork sur GitHub |
| Clone | `git clone votre-fork` |
| Upstream | `git remote add upstream URL-original` |
| Branch | `git checkout -b feature` |
| Commit | Faire changements |
| Push | `git push origin feature` |
| PR | Créer Pull Request sur GitHub |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **GitHub Skills: Introduction** ⭐⭐⭐ | Cours officiel | https://github.com/skills/introduction-to-github |
| **GitHub Skills: Review PRs** ⭐⭐ | Code review | https://github.com/skills/review-pull-requests |
| **First Contributions** ⭐⭐⭐ | Premier PR réel | https://github.com/firstcontributions/first-contributions |
| **Good First Issues** | Trouver issues | https://goodfirstissues.com |
| **Up For Grabs** | Projets débutants | https://up-for-grabs.net |
| **Codecademy: GitHub** | Cours | https://www.codecademy.com/learn/learn-github-introduction |

### Workflow Fork & PR
```bash
# 1. Fork sur GitHub (bouton)

# 2. Cloner votre fork
git clone git@github.com:VOTRE-USER/projet.git
cd projet

# 3. Ajouter upstream
git remote add upstream git@github.com:ORIGINAL/projet.git

# 4. Sync avec upstream
git fetch upstream
git checkout main
git merge upstream/main

# 5. Créer branche feature
git checkout -b fix/typo-readme

# 6. Faire changements et commit
git add . && git commit -m "Fix typo in README"

# 7. Push vers votre fork
git push origin fix/typo-readme

# 8. Créer PR sur GitHub
```

---

## 0.2.16 : GitHub Features

| Feature | Usage |
|---------|-------|
| Issues | Bug reports, feature requests |
| Pull Requests | Code review, merge |
| Actions | CI/CD |
| Projects | Kanban boards |
| Wiki | Documentation |
| Releases | Versions |
| Discussions | Forum communautaire |
| **Codespaces** | Environnement dev cloud |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **GitHub Skills: Communicate with Markdown** ⭐ | Cours | https://github.com/skills/communicate-using-markdown |
| **GitHub Skills: GitHub Pages** | Cours | https://github.com/skills/github-pages |
| **GitHub Skills: Code with Codespaces** ⭐ | Cours | https://github.com/skills/code-with-codespaces |
| **GitHub Docs: Features** | Documentation | https://docs.github.com/en/get-started |
| **GitHub Docs: Codespaces** | Documentation | https://docs.github.com/en/codespaces |
| **GitHub Learning Lab** (archivé) | Référence | https://lab.github.com |

---

## Mini-Projet 0.2.C : Contribution Open Source

| Tâche | Objectif | Ressource |
|-------|----------|-----------|
| Trouver projet | good first issue | https://goodfirstissues.com |
| Fork | Copier le repo | GitHub |
| Cloner et configurer | upstream | Terminal |
| Créer branche | feature/fix | Git |
| Faire changement | Code ou docs | Éditeur |
| Push et PR | Soumettre | GitHub |

### Checklist Validation
- [ ] Projet open source trouvé avec "good first issue"
- [ ] Fork créé sur votre compte
- [ ] Clone local avec upstream configuré
- [ ] Branche feature créée
- [ ] Changement effectué (typo, doc, ou code)
- [ ] PR créée avec description claire
- [ ] (Bonus) PR mergée !

### Ressources pour Trouver des Projets

| Ressource | URL |
|-----------|-----|
| **First Contributions** | https://github.com/firstcontributions/first-contributions |
| **Good First Issues** | https://goodfirstissues.com |
| **Up For Grabs** | https://up-for-grabs.net |
| **Awesome for Beginners** | https://github.com/MunGell/awesome-for-beginners |
| **CodeTriage** | https://www.codetriage.com |

---

# Partie 4 : Git Avancé (20h, 300 XP)

---

## 0.2.17 : Git Log Avancé

| Commande | Usage |
|----------|-------|
| `git log --grep="bug"` | Recherche dans messages |
| `git log --author="Alice"` | Par auteur |
| `git log --since="2024-01-01"` | Par date |
| `git blame fichier` | Qui a modifié chaque ligne |
| `git diff branch1..branch2` | Différences entre branches |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Git Katas: git-log** | Exercice | git-katas/git-log |
| **Git Katas: investigation** | Exercice | git-katas/investigation |
| **Atlassian: Git Log** | Guide | https://www.atlassian.com/git/tutorials/git-log |
| **Git Official: Viewing History** | Doc | https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History |

### Commandes Avancées
```bash
# Log graphique avec toutes branches
git log --oneline --graph --all --decorate

# Commits d'un auteur cette semaine
git log --author="Alice" --since="1 week ago"

# Rechercher dans les messages
git log --grep="fix" --oneline

# Voir qui a modifié une ligne
git blame -L 10,20 fichier.py

# Différences entre branches
git diff main..feature --stat
```

---

## 0.2.18 : Cherry-pick et Bisect

| Commande | Usage |
|----------|-------|
| `git cherry-pick HASH` | Appliquer un commit spécifique |
| `git bisect start` | Commencer recherche de bug |
| `git bisect bad` | Marquer commit mauvais |
| `git bisect good` | Marquer commit bon |
| `git bisect reset` | Terminer bisect |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **Learn Git Branching: Cherry-pick** ⭐⭐⭐ | Simulation | https://learngitbranching.js.org |
| **Git Katas: cherry-pick** ⭐ | Exercice | git-katas/cherry-pick |
| **Git Katas: bisect** ⭐ | Exercice | git-katas/bisect |
| **Atlassian: Git Cherry Pick** | Guide | https://www.atlassian.com/git/tutorials/cherry-pick |
| **Git Official: Debugging with Git** | Doc | https://git-scm.com/book/en/v2/Git-Tools-Debugging-with-Git |

### Learn Git Branching
```
Niveaux recommandés :
- Main > Moving Work Around > 1. Cherry-pick Intro
```

### Bisect Automatique
```bash
# Trouver le commit qui a cassé les tests
git bisect start
git bisect bad HEAD
git bisect good v1.0.0

# Automatique avec script
git bisect run pytest tests/

# Terminer
git bisect reset
```

---

## 0.2.19 : Tags

| Commande | Usage |
|----------|-------|
| `git tag v1.0.0` | Tag léger |
| `git tag -a v1.0.0 -m "Release 1.0"` | Tag annoté |
| `git push origin --tags` | Push tous les tags |
| `git push origin v1.0.0` | Push un tag |
| `git checkout v1.0.0` | Checkout tag |

### Ressources Interactives

| Ressource | Type | URL |
|-----------|------|-----|
| **Git Katas: tags** | Exercice | git-katas/tags |
| **Atlassian: Git Tag** | Guide | https://www.atlassian.com/git/tutorials/inspecting-a-repository/git-tag |
| **W3Schools: Git Tags** | Tutorial | https://www.w3schools.com/git/git_tags.asp |
| **Semantic Versioning** | Standard | https://semver.org |

### Workflow Release
```bash
# Créer tag annoté
git tag -a v1.0.0 -m "First stable release"

# Lister tags
git tag -l "v1.*"

# Push tags
git push origin --tags

# Créer release sur GitHub
gh release create v1.0.0 --title "v1.0.0" --notes "First release"
```

---

## 0.2.20 : Git Hooks ⭐

| Hook | Usage |
|------|-------|
| `pre-commit` | Avant commit (lint, tests) |
| `commit-msg` | Vérifier format message |
| `pre-push` | Avant push (tests complets) |
| `post-merge` | Après merge |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **pre-commit.com** ⭐⭐⭐ | Framework officiel | https://pre-commit.com |
| **pre-commit hooks collection** | Hooks prêts | https://github.com/pre-commit/pre-commit-hooks |
| **MIT 6.S194 Git Hooks Exercise** ⭐⭐ | Lab pratique | https://courses.csail.mit.edu/6.S194/13/lessons/03-git/adding-custom-hooks-to-git.html |
| **AKS DevSecOps Workshop** ⭐⭐ | Lab Azure complet | https://azure.github.io/AKS-DevSecOps-Workshop/modules/Module2/Lab-2c.html |
| **Turing School Frontend** | Tutorial pre-commit | https://frontend.turing.edu/lessons/module-3/advanced-git-workflow.html |
| **PDC Support pre-commit Lab** | Exercice guidé | https://pdc-support.github.io/software-engineering-intro/11-pre-commit-hook/ |

### Documentation & Guides

| Ressource | URL |
|-----------|-----|
| **Git Official Hooks** | https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks |
| **Atlassian Git Hooks** | https://www.atlassian.com/git/tutorials/git-hooks |
| **DigitalOcean Tutorial** | https://www.digitalocean.com/community/tutorials/how-to-use-git-hooks-to-automate-development-and-deployment-tasks |
| **githooks.com** | https://githooks.com |
| **Stefanie Molin Setup Guide** | https://stefaniemolin.com/articles/devx/pre-commit/setup-guide |
| **DEV.to pre-commit** | https://dev.to/jonasbn/til-use-pre-commit-hook-the-framework-1hoh |
| **DataCamp Git Hooks Guide** ⭐⭐ | https://www.datacamp.com/tutorial/git-hooks-complete-guide |
| **NHS RAP Git Hooks** | https://nhsdigital.github.io/rap-community-of-practice/training_resources/git/githooks/ |
| **GitKraken Hook Example** | https://help.gitkraken.com/gitkraken-client/githooksexample/ |
| **W3Schools Git Hooks** | https://www.w3schools.com/git/git_hooks.asp |
| **Graphite pre-commit Guide** ⭐ | https://graphite.com/guides/implementing-pre-commit-hooks-to-enforce-code-quality |
| **pre-commit Hooks List** | https://pre-commit.com/hooks.html |
| **DEV.to pre-commit Amazing** | https://dev.to/vaiolabs_io/amazing-pre-commit-and-how-to-use-it-5enb |
| **Elliot Jordan Intro** | https://www.elliotjordan.com/posts/pre-commit-01-intro/ |
| **Infra Bootstrap Tools** | https://xnok.github.io/infra-bootstrap-tools/docs/tools/pre-commit/ |

### Frameworks Hooks

| Outil | Langage | URL |
|-------|---------|-----|
| **pre-commit** ⭐⭐⭐ | Python (multi-lang) | https://pre-commit.com |
| **Husky** | Node.js | https://typicode.github.io/husky |
| **Lefthook** | Go | https://github.com/evilmartians/lefthook |

### Installation pre-commit
```bash
# Installer
pip install pre-commit

# Créer .pre-commit-config.yaml
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: check-merge-conflict
  
  - repo: https://github.com/psf/black
    rev: 24.1.0
    hooks:
      - id: black
        language_version: python3

  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8
EOF

# Installer hooks
pre-commit install

# Tester sur tous les fichiers
pre-commit run --all-files
```

### Hook Manuel (sans framework)
```bash
# .git/hooks/pre-commit
#!/bin/sh
echo "Running pre-commit checks..."

# Vérifier trailing whitespace
if git diff --cached --check; then
    echo "✓ No whitespace errors"
else
    echo "✗ Whitespace errors found"
    exit 1
fi

# Vérifier fichiers volumineux (>1MB)
for file in $(git diff --cached --name-only); do
    if [ -f "$file" ]; then
        size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file")
        if [ "$size" -gt 1000000 ]; then
            echo "✗ File too large: $file ($size bytes)"
            exit 1
        fi
    fi
done

echo "✓ All checks passed"
exit 0
```

```bash
# Rendre exécutable
chmod +x .git/hooks/pre-commit
```

### Hooks Disponibles
```
CLIENT-SIDE:
├── pre-commit       # Avant commit (lint, format, tests rapides)
├── prepare-commit-msg
├── commit-msg       # Valider format message (ex: JIRA-XXX)
├── post-commit
├── pre-push         # Avant push (tests complets)
├── pre-rebase
└── post-checkout

SERVER-SIDE:
├── pre-receive
├── update
└── post-receive
```

---

# Partie 5 : CI/CD avec GitHub Actions (20h, 250 XP)

---

## 0.2.21 : Introduction CI/CD

| Concept | Description |
|---------|-------------|
| CI | Continuous Integration |
| CD | Continuous Deployment |
| Pipeline | Séquence d'étapes automatisées |
| Workflow | Définition du pipeline (YAML) |
| Job | Groupe de steps |
| Step | Action individuelle |
| Runner | Machine d'exécution |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **GitHub Skills: Hello Actions** ⭐⭐⭐ | Cours officiel <2h | https://github.com/skills/hello-github-actions |
| **GitHub Actions Hero** ⭐⭐⭐ | Émulateur workflows | https://github-actions-hero.vercel.app |
| **Microsoft Learn: Intro Actions** ⭐⭐ | Module gratuit | https://learn.microsoft.com/training/modules/introduction-to-github-actions |
| **GitHub Docs: Understanding Actions** | Documentation | https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions |

---

## 0.2.22 : Syntaxe Workflow

| Élément | Description |
|---------|-------------|
| `name:` | Nom du workflow |
| `on:` | Déclencheurs (push, PR, schedule) |
| `jobs:` | Liste des jobs |
| `runs-on:` | Runner (ubuntu-latest, windows) |
| `steps:` | Étapes du job |
| `uses:` | Action externe |
| `run:` | Commande shell |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **GitHub Skills: Hello Actions** ⭐⭐⭐ | Hands-on | https://github.com/skills/hello-github-actions |
| **GitHub Actions Hero** ⭐⭐⭐ | Simulateur YAML | https://github-actions-hero.vercel.app |
| **GitHub Skills: CI** ⭐⭐ | Cours CI | https://github.com/skills/continuous-integration |
| **Microsoft Learn: Implement Actions** | Module | https://learn.microsoft.com/training/modules/implement-github-actions |
| **Microsoft Learn: Intro Actions** | Module | https://learn.microsoft.com/en-us/training/modules/introduction-to-github-actions/ |
| **Microsoft Learn: Automate Tasks** | Module | https://learn.microsoft.com/en-us/training/modules/github-actions-automate-tasks/ |
| **Microsoft Learn: CD with Actions** | Module | https://learn.microsoft.com/en-us/training/modules/github-actions-cd/ |
| **Microsoft Learn: CI with Actions** | Module | https://learn.microsoft.com/en-us/training/modules/learn-continuous-integration-github-actions/ |
| **Microsoft Learn: Custom Actions** | Module | https://learn.microsoft.com/en-us/training/modules/create-custom-github-actions/ |
| **KodeKloud GitHub Actions** ⭐⭐ | Cours + labs | https://kodekloud.com/courses/github-actions |
| **Pluralsight: Getting Started** | Cours | https://www.pluralsight.com/courses/github-actions-getting-started |
| **GitHub Actions Training** | Officiel | https://github.com/services/actions-training |

### Documentation

| Ressource | URL |
|-----------|-----|
| **GitHub Docs: Workflow Syntax** | https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions |
| **GitHub Actions Cheat Sheet** | https://github.github.io/actions-cheat-sheet/actions-cheat-sheet.pdf |
| **GitHub Actions Tutorials** | https://docs.github.com/en/actions/tutorials |

### Workflow de Base
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
        
      - name: Run tests
        run: pytest tests/ -v
```

---

## 0.2.23 : Matrix et Cache

| Feature | Usage |
|---------|-------|
| `strategy.matrix` | Tester plusieurs versions |
| `needs:` | Dépendances entre jobs |
| `actions/cache` | Cache des dépendances |
| `actions/upload-artifact` | Sauvegarder fichiers |
| `actions/download-artifact` | Récupérer fichiers |

### Ressources Interactives ⭐⭐⭐

| Ressource | Type | URL |
|-----------|------|-----|
| **GitHub Skills: Reusable Workflows** ⭐⭐ | Cours avancé | https://github.com/skills/reusable-workflows |
| **GitHub Skills: Publish Packages** | Docker + packages | https://github.com/skills/publish-packages |
| **Microsoft Learn: CI with Actions** | Module | https://learn.microsoft.com/training/modules/learn-continuous-integration-github-actions |
| **Microsoft Learn: Custom Actions** | Module | https://learn.microsoft.com/training/modules/create-custom-github-actions |
| **GitHub Actions Workshop** ⭐⭐ | Workshop complet | https://prasadhonrao.github.io/github-actions-workshop |

### Guides Avancés

| Ressource | Sujet | URL |
|-----------|-------|-----|
| **Codefresh Matrix Guide** | Matrix builds | https://codefresh.io/learn/github-actions/github-actions-matrix |
| **CICube Cache Guide** | Caching avancé | https://cicube.io/blog/github-actions-cache |
| **Blacksmith Artifacts** | Matrix + artifacts | https://www.blacksmith.sh/blog/matrix-builds-with-github-actions |
| **BrowserStack Local Test** | Guide complet | https://www.browserstack.com/guide/test-github-actions-locally |
| **Red Hat act-js Guide** | Testing avancé | https://www.redhat.com/en/blog/testing-github-actions-locally |
| **Codacy Actions Guide** | Best practices | https://blog.codacy.com/how-to-test-github-actions |
| **Actions Cheat Sheet** | PDF référence | https://github.github.io/actions-cheat-sheet/actions-cheat-sheet.pdf |

### Repos d'Entraînement

| Ressource | Type | URL |
|-----------|------|-----|
| **ActionsFundamentals** | Repo training | https://github.com/ps-actions-sandbox/ActionsFundamentals |
| **actions-sandbox** | Playground dev | https://github.com/paltom/github-actions-sandbox |
| **codespaces-actions-playground** | Sandbox archivé | https://github.com/github/codespaces-actions-playground |
| **hello-github-actions** | Template officiel | https://github.com/githubtraining/hello-github-actions |

### Outils Test Local

| Outil | Description | URL |
|-------|-------------|-----|
| **nektos/act** ⭐⭐⭐ | Run workflows localement | https://github.com/nektos/act |
| **debugger-action** | SSH into runner | https://github.com/csexton/debugger-action |

### Workflow avec Matrix et Cache
```yaml
name: CI Matrix

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: ['3.9', '3.10', '3.11', '3.12']
        exclude:
          - os: macos-latest
            python-version: '3.9'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run tests
        run: pytest tests/ --cov=src --cov-report=xml
      
      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage-${{ matrix.os }}-${{ matrix.python-version }}
          path: coverage.xml

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: echo "Deploying..."
```

---

## Mini-Projet 0.2.D : Pipeline CI Complet

| Tâche | Objectif | Ressource |
|-------|----------|-----------|
| Lint | Code style (flake8, black) | pre-commit |
| Tests | Avec coverage | pytest-cov |
| Build | Créer artifact | actions/upload-artifact |
| Matrix | Plusieurs versions Python | strategy.matrix |
| Cache | Dépendances pip | actions/cache |

### Checklist Validation
- [ ] Workflow `.github/workflows/ci.yml` créé
- [ ] Lint avec flake8 ou black
- [ ] Tests pytest avec coverage > 80%
- [ ] Matrix: au moins 2 versions Python
- [ ] Matrix: au moins 2 OS
- [ ] Cache pip configuré
- [ ] Artifact coverage uploadé
- [ ] Badge CI dans README

### Template Complet
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install flake8 black
      - run: flake8 src/ tests/
      - run: black --check src/ tests/

  test:
    needs: lint
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ['3.10', '3.11', '3.12']
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      
      - name: Cache pip
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
      
      - name: Install
        run: pip install -r requirements.txt
      
      - name: Test
        run: pytest tests/ --cov=src --cov-report=xml --cov-fail-under=80
      
      - name: Upload Coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage-${{ matrix.os }}-py${{ matrix.python-version }}
          path: coverage.xml

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install build
      - run: python -m build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
```

---

# Projet Final Module 0.2 : Projet Collaboratif (15h, 1000 XP)

---

## Livrables

| # | Livrable | Description |
|---|----------|-------------|
| a | Repository | Structure propre, .gitignore complet |
| b | Branches | main, develop, feature/* |
| c | CI/CD | Tests + lint automatiques |
| d | Protection | Branch rules sur main |
| e | Documentation | README.md, CONTRIBUTING.md |
| f | Issues | Templates issues/PR |
| g | Releases | Au moins 1 release avec tag |

## Ressources pour le Projet Final

| Ressource | Usage | URL |
|-----------|-------|-----|
| **gitignore.io** | Générer .gitignore | https://gitignore.io |
| **GitHub Skills: All courses** | Révision | https://skills.github.com |
| **Readme.so** | Générer README | https://readme.so |
| **Shields.io** | Badges | https://shields.io |
| **Conventional Commits** | Format commits | https://www.conventionalcommits.org |

## Structure Repository Recommandée

```
projet/
├── .github/
│   ├── workflows/
│   │   └── ci.yml
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── PULL_REQUEST_TEMPLATE.md
├── src/
│   └── __init__.py
├── tests/
│   └── test_main.py
├── .gitignore
├── .pre-commit-config.yaml
├── README.md
├── CONTRIBUTING.md
├── LICENSE
└── requirements.txt
```

## Checklist Validation Finale

### Repository
- [ ] Structure de dossiers propre
- [ ] .gitignore complet pour le langage
- [ ] LICENSE présent

### Branches
- [ ] main protégé (require PR, require reviews)
- [ ] develop branch existe
- [ ] Au moins 2 feature branches créées et mergées

### CI/CD
- [ ] Workflow CI fonctionnel
- [ ] Lint automatique
- [ ] Tests automatiques
- [ ] Badge CI dans README

### Documentation
- [ ] README.md complet (description, installation, usage, contribution)
- [ ] CONTRIBUTING.md avec workflow
- [ ] Issue templates configurés
- [ ] PR template configuré

### Releases
- [ ] Au moins 1 tag créé (v0.1.0 ou v1.0.0)
- [ ] Release créée sur GitHub
- [ ] Changelog ou release notes

### Collaboration
- [ ] Au moins 3 PRs créées et mergées
- [ ] Au moins 1 PR avec code review
- [ ] Commits suivent convention (feat:, fix:, docs:)

---

## 📊 Récapitulatif des Ressources par Section

### Quiz & Évaluation

| Ressource | Type | URL |
|-----------|------|-----|
| **W3Schools Git Quiz** | 25 questions | https://www.w3schools.com/git/git_quiz.asp |
| **W3Schools Git Exercises** | Exercices interactifs | https://www.w3schools.com/git/git_exercises.asp |
| **W3Schools Git Tutorial** | Tutorial complet | https://www.w3schools.com/git/ |
| **GeeksforGeeks Git Quiz** | Quiz avancé | https://www.geeksforgeeks.org/quizzes/git-quiz-day-3/ |

### Cheat Sheets & Références

| Ressource | URL |
|-----------|-----|
| **GitHub Education Cheat Sheet** | https://education.github.com/git-cheat-sheet-education.pdf |
| **Git Official Documentation** | https://git-scm.com/doc |
| **Pro Git Book (gratuit)** | https://git-scm.com/book/en/v2 |
| **GitHub Learning Lab (legacy)** | https://lab.github.com/ |

| Section | Ressource Principale | Ressource Secondaire |
|---------|---------------------|---------------------|
| 0.2.1-0.2.5 | Learn Git Branching | Git Katas, Oh My Git! |
| 0.2.6 | globster.xyz | gitignore.io |
| 0.2.7-0.2.9 | Learn Git Branching | Git Katas |
| 0.2.10 | Learn Git Branching | Git Katas: rebase |
| 0.2.11 | Git Katas: stash | Atlassian |
| 0.2.12-0.2.13 | Learn Git Branching Remote | GitHub Skills |
| 0.2.14 | GitHub Docs SSH | The Odin Project |
| 0.2.15-0.2.16 | GitHub Skills | First Contributions |
| 0.2.17-0.2.18 | Learn Git Branching | Git Katas |
| 0.2.19 | Git Katas: tags | Atlassian |
| 0.2.20 | pre-commit.com | MIT Lab, Atlassian |
| 0.2.21-0.2.23 | GitHub Skills Actions | Actions Hero, Microsoft Learn |

---

## ✅ Couverture Finale

| Partie | Sections | Couverture | Ressources |
|--------|----------|------------|------------|
| Partie 1 : Fondamentaux | 0.2.1-0.2.6 | ✅ 95% | Learn Git Branching, Git Katas, globster.xyz |
| Partie 2 : Branches/Merge | 0.2.7-0.2.11 | ✅ 95% | Learn Git Branching, Git Katas |
| Partie 3 : Remote/GitHub | 0.2.12-0.2.16 | ✅ 90% | GitHub Skills, The Odin Project |
| Partie 4 : Git Avancé | 0.2.17-0.2.20 | ✅ 85% | Git Katas, pre-commit.com |
| Partie 5 : CI/CD | 0.2.21-0.2.23 | ✅ 95% | GitHub Skills, Actions Hero |
| Mini-Projets | A, B, C, D | ✅ 90% | Combinaison ressources |
| Projet Final | Collaboratif | ✅ 85% | GitHub Skills, templates |

**Total : 100+ ressources interactives couvrant l'intégralité du module 0.2**

---

*Document généré pour POKÉOS — Module 0.2 Git & Development Tools*  
*140h, 2000 XP — Dernière mise à jour : Janvier 2026*
