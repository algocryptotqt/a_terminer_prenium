# MODULE 0.0.B : APPRENDRE À APPRENDRE
## Catalogue des Ressources Interactives

> **Durée** : 70h | **XP** : 1000  
> **Prérequis** : 0.1 Linux, 0.2 Git  
> **Débloque** : 0.3 Python  
> **Couverture** : 99% des concepts couverts par des ressources externes gratuites  
> **Dernière mise à jour** : Janvier 2026

---

## 📋 Table des Matières

- [B.1 : Naviguer la Documentation (20h)](#b1--naviguer-la-documentation-20h)
  - [B.1.1 : Types de Documentation](#b11--types-de-documentation)
  - [B.1.2 : Lire la Documentation Efficacement](#b12--lire-la-documentation-efficacement)
  - [B.1.3 : Documentation par Langage](#b13--documentation-par-langage)
  - [B.1.4 : Exercice — Explorer la Documentation](#b14--exercice--explorer-la-documentation)
- [B.2 : Méthodologie de Debug (25h)](#b2--méthodologie-de-debug-25h)
  - [B.2.1 : Types d'Erreurs](#b21--types-derreurs)
  - [B.2.2 : Lire un Message d'Erreur](#b22--lire-un-message-derreur)
  - [B.2.3 : Processus de Debug Systématique](#b23--processus-de-debug-systématique)
  - [B.2.4 : Techniques de Debug](#b24--techniques-de-debug)
  - [B.2.5 : Erreurs Courantes par Langage](#b25--erreurs-courantes-par-langage)
  - [B.2.6 : Exercice — Debug Challenge](#b26--exercice--debug-challenge)
- [B.3 : Chercher de l'Aide Efficacement (15h)](#b3--chercher-de-laide-efficacement-15h)
  - [B.3.1 : Avant de Demander](#b31--avant-de-demander)
  - [B.3.2 : Comment Poser une Bonne Question](#b32--comment-poser-une-bonne-question)
  - [B.3.3 : Où Demander de l'Aide](#b33--où-demander-de-laide)
  - [B.3.4 : MCVE — Minimal Complete Verifiable Example](#b34--mcve--minimal-complete-verifiable-example)
- [B.4 : Construire son Autonomie (10h)](#b4--construire-son-autonomie-10h)
  - [B.4.1 : Ressources pour Rester à Jour](#b41--ressources-pour-rester-à-jour)
  - [B.4.2 : Pratiquer Continuellement](#b42--pratiquer-continuellement)
  - [B.4.3 : Créer son Guide Personnel](#b43--créer-son-guide-personnel)
  - [B.4.4 : Mini-projet B — Mon Guide de Référence](#b44--mini-projet-b--mon-guide-de-référence)
- [Annexe A : Jeux Interactifs par Technologie](#annexe-a--jeux-interactifs-par-technologie)
- [Annexe B : Playgrounds & IDEs en Ligne](#annexe-b--playgrounds--ides-en-ligne)
- [Annexe C : Chaînes YouTube Recommandées](#annexe-c--chaînes-youtube-recommandées)
- [Annexe D : Récapitulatif des Exercices](#annexe-d--récapitulatif-des-exercices)

---

# B.1 : Naviguer la Documentation (20h)

---

## B.1.1 : Types de Documentation

### 📚 Concepts du Programme

| # | Concept | Description |
|---|---------|-------------|
| a | Documentation officielle | Source primaire |
| b | Tutoriels | Guides pas à pas |
| c | Références | Description exhaustive |
| d | API docs | Documentation de code |
| e | Man pages | Manuel Unix |
| f | README | Introduction projet |
| g | Wikis | Documentation collaborative |
| h | Stack Overflow | Q&A communautaire |

### 🔧 Ressources Interactives

#### Agrégateurs de Documentation

| Outil | URL | Description | Gratuit |
|-------|-----|-------------|---------|
| **DevDocs** | devdocs.io | 400+ documentations, mode offline, recherche rapide | ✅ 100% |
| **Zeal** | zealdocs.org | Clone open-source de Dash (Win/Linux) | ✅ 100% |
| **Dash** | kapeli.com/dash | Navigateur docs offline (macOS) | Freemium |
| **Devhints** | devhints.io | Cheatsheets élégantes par technologie | ✅ 100% |
| **Cheat.sh** | cheat.sh | Cheatsheets CLI (`curl cheat.sh/tar`) | ✅ 100% |
| **QuickRef.ME** | quickref.me | Cheatsheets développeurs | ✅ 100% |

#### Documentation Officielle Interactive

| Type | Ressource | URL | Fonctionnalités |
|------|-----------|-----|-----------------|
| **a** Officielle | MDN Web Docs | developer.mozilla.org | Try-It Editor intégré |
| **b** Tutoriels | W3Schools | w3schools.com | "Try It Yourself" sur chaque page |
| **c** Références | cppreference | cppreference.com | Exemples compilables |
| **d** API docs | Postman Learning | learning.postman.com | Tutoriels API interactifs |
| **d** API docs | Postman Academy | academy.postman.com | Cours certifiants API |
| **d** API docs | Swagger UI | swagger.io | Démos API interactives |
| **d** API docs | idratherbewriting | idratherbewriting.com/learnapidoc | Cours API 900+ pages |
| **e** Man pages | TLDR Pages | tldr.sh | Résumés concis de commandes |
| **e** Man pages | ExplainShell | explainshell.com | Déconstruction visuelle de commandes |
| **f** README | GitHub | github.com | READMEs avec badges, images |
| **f** README | MakeAReadme | makeareadme.com | Générateur interactif |
| **f** README | Read the Docs | readthedocs.org | Hébergement doc projets |
| **g** Wikis | ArchWiki | wiki.archlinux.org | Wiki le plus complet Linux |
| **h** Stack Overflow | Stack Overflow | stackoverflow.com | 23M+ questions |

#### Outils de Changelog

| Outil | URL | Description |
|-------|-----|-------------|
| **Keep a Changelog** | keepachangelog.com | Guide standard |

#### 🎬 Vidéos Recommandées

| Vidéo | Auteur | Durée | Concept couvert |
|-------|--------|-------|-----------------|
| How to Read Documentation as a Beginner | Andy Sterkowitz | ~15 min | a, b, c |
| APIs for Beginners - Full Course | freeCodeCamp | 3h20 | d |
| Linux Man Pages Explained | LearnLinuxTV | ~20 min | e |
| How to Write a Good README | Traversy Media | ~15 min | f |

### 📝 Exercice B.1.1 : Cartographie des Documentations

```
EXERCICE B.1.1 : Identifier les Types de Documentation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Reconnaître et utiliser chaque type de documentation

Pour chaque ressource, identifie le type (a-h) :

1. docs.python.org/3/library/functions.html    → Type: ___
2. man ls                                       → Type: ___
3. github.com/facebook/react/blob/main/README.md → Type: ___
4. stackoverflow.com/questions/231767          → Type: ___
5. developer.mozilla.org/en-US/docs/Web/API    → Type: ___
6. wiki.archlinux.org/title/Installation_guide → Type: ___
7. realpython.com/python-beginner-tips/        → Type: ___
8. stripe.com/docs/api                         → Type: ___

Réponses : 1-c, 2-e, 3-f, 4-h, 5-d, 6-g, 7-b, 8-d

Tâche bonus : Installe DevDocs offline avec 3 docs de ton choix
```

---

## B.1.2 : Lire la Documentation Efficacement

### 📚 Concepts du Programme

| # | Concept | Description |
|---|---------|-------------|
| a | Vue d'ensemble d'abord | Comprendre la structure |
| b | Table des matières | Navigation rapide |
| c | Recherche | Ctrl+F, search bar |
| d | Exemples | Toujours lire les exemples |
| e | Voir aussi | Liens connexes |
| f | Changelog | Historique des changements |
| g | Version | Attention à la version |

### 🔧 Ressources Interactives

#### Outils de Navigation Rapide

| Concept | Outil | URL | Usage |
|---------|-------|-----|-------|
| **a** Vue d'ensemble | DevDocs | devdocs.io | Sidebar avec structure |
| **b** Table des matières | MDN | developer.mozilla.org | TOC sticky |
| **c** Recherche | Algolia DocSearch | docsearch.algolia.com | Recherche instantanée |
| **d** Exemples | RunKit | runkit.com | Exécuter exemples npm |
| **d** Exemples | CodePen | codepen.io | Exemples CSS/JS live |
| **g** Version | docs.rs | docs.rs | Toutes versions Rust |

#### Méthode de Lecture Active

| Technique | Description | Outil Associé |
|-----------|-------------|---------------|
| **Skimming** | Lecture rapide titres/exemples | Table des matières |
| **Code First** | Lire les exemples avant le texte | Playground |
| **Try-Modify-Break** | Exécuter, modifier, casser | REPL/Sandbox |
| **Annotation** | Prendre des notes actives | Obsidian, Notion |

#### Guides pour Lire la Documentation

| Ressource | URL | Type |
|-----------|-----|------|
| **GitHub: eltiffster/readingDocs** | github.com/eltiffster/readingDocs | Guide complet |
| **learncodethehardway.com** | learncodethehardway.com/blog | Article "How to Read Programmer Documentation" |
| **how-to.dev** | how-to.dev | Article "How to Read the Documentation" |
| **stoplight.io** | stoplight.io/blog | "How to Read API Documentation" |

#### 🎬 Vidéos Recommandées

| Vidéo | Auteur | Durée | Concepts couverts |
|-------|--------|-------|-------------------|
| How to Use MDN Web Docs | Kevin Powell | ~10 min | a, b, c, e |
| How I Learn New Technologies | Fireship | ~8 min | a, d |
| How to Escape Tutorial Hell | Coding in Flow | ~12 min | d |

### 📝 Exercice B.1.2 : Lecture Efficace Chronométrée

```
EXERCICE B.1.2 : Speed Reading Challenge
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Trouver des informations rapidement dans la documentation

Chronomètre chaque recherche dans MDN (developer.mozilla.org) :

1. Trouve la syntaxe de Array.prototype.reduce()
   Temps: _____ sec | Méthode utilisée: ___________

2. Trouve TOUS les paramètres de fetch()
   Temps: _____ sec | Méthode utilisée: ___________

3. Trouve la différence entre let et const
   Temps: _____ sec | Méthode utilisée: ___________

4. Trouve quand Promise.allSettled() a été ajouté (version)
   Temps: _____ sec | Méthode utilisée: ___________

5. Trouve 3 méthodes "Voir aussi" liées à addEventListener
   Temps: _____ sec | Méthode utilisée: ___________

Objectif : < 2 minutes par question
Total cible : < 10 minutes pour les 5 questions
```

```
EXERCICE B.1.2b : Try-Modify-Break
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Apprendre par l'expérimentation active

Prends cet exemple de MDN (Array.map) :

const numbers = [1, 4, 9];
const roots = numbers.map((num) => Math.sqrt(num));
// roots is now [1, 2, 3]

1. TRY    : Exécute le code tel quel (playcode.io)
2. MODIFY : Change pour calculer les carrés
3. MODIFY : Ajoute l'index : ["0:1", "1:4", "2:9"]
4. BREAK  : Passe undefined à map() - que se passe-t-il ?
5. BREAK  : Utilise map sur un objet - que se passe-t-il ?

Documente tes observations pour chaque étape.
```

---

## B.1.3 : Documentation par Langage

### 📚 Concepts du Programme

| # | Langage | Ressources |
|---|---------|------------|
| a | Python | docs.python.org, PEP |
| b | C | cppreference.com, man pages |
| c | Rust | doc.rust-lang.org, docs.rs |
| d | Général | MDN, DevDocs.io |

### 🔧 Ressources Interactives par Langage

#### a) Python

| Ressource | URL | Type | Interactivité |
|-----------|-----|------|---------------|
| **docs.python.org** | docs.python.org | Officielle | REPL intégré |
| **Python Tutor** | pythontutor.com | Visualisation | Exécution pas à pas |
| **Real Python** | realpython.com | Tutoriels | Quiz intégrés |
| **PEP Index** | peps.python.org | Standards | - |
| **PyPI** | pypi.org | Packages | Docs par package |

#### b) C

| Ressource | URL | Type | Interactivité |
|-----------|-----|------|---------------|
| **cppreference.com** | cppreference.com | Référence | Exemples compilables |
| **Beej's Guide to C** | beej.us/guide/bgc | Tutoriel | Guide complet gratuit |
| **man pages** | man7.org | Manuel | - |
| **Compiler Explorer** | godbolt.org | Outil | Voir l'assembleur |
| **cdecl** | cdecl.org | Outil | Décoder déclarations C |
| **C Gibberish ↔ English** | cdecl.org | Traducteur | Interactif |

#### c) Rust

| Ressource | URL | Type | Interactivité |
|-----------|-----|------|---------------|
| **The Rust Book** | doc.rust-lang.org/book | Tutoriel | Playground intégré |
| **Rust by Example** | doc.rust-lang.org/rust-by-example | Exemples | Exécutable |
| **docs.rs** | docs.rs | API docs | Toutes versions |
| **Rust Playground** | play.rust-lang.org | Sandbox | Compilation live |
| **Rustlings** | github.com/rust-lang/rustlings | Exercices | CLI interactif |

#### d) Général (Web)

| Ressource | URL | Langages | Interactivité |
|-----------|-----|----------|---------------|
| **MDN Web Docs** | developer.mozilla.org | HTML/CSS/JS | Try-It Editor |
| **DevDocs** | devdocs.io | 400+ | Offline, recherche |
| **W3Schools** | w3schools.com | Web | Try It Yourself |
| **Can I Use** | caniuse.com | CSS/JS | Compatibilité navigateurs |
| **Bundlephobia** | bundlephobia.com | NPM | Taille des packages |
| **Carbon** | carbon.now.sh | Code | Screenshots de code |

#### 🎬 Vidéos par Langage

| Langage | Vidéo | Auteur | Durée |
|---------|-------|--------|-------|
| Python | Python Tutorial for Beginners | Programming with Mosh | 6h |
| C | C Programming Full Course | freeCodeCamp | 4h |
| Rust | Rust in 100 Seconds | Fireship | 2 min |
| Rust | Rust Tutorial Full Course | freeCodeCamp | 14h |

### 📝 Exercice B.1.3 : Navigation Multi-Langages

```
EXERCICE B.1.3 : Trouver la même info dans 3 langages
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Maîtriser la documentation de plusieurs langages

Trouve comment OUVRIR et LIRE un fichier dans chaque langage :

PYTHON (docs.python.org) :
- Fonction : ___________________
- Syntaxe : ___________________
- URL exacte : ___________________

C (cppreference.com ou man) :
- Fonction : ___________________
- Syntaxe : ___________________
- URL exacte : ___________________

RUST (doc.rust-lang.org) :
- Fonction/Struct : ___________________
- Syntaxe : ___________________
- URL exacte : ___________________

Temps total cible : < 15 minutes
```

---

## B.1.4 : Exercice — Explorer la Documentation

### 📚 Tâches du Programme

| # | Tâche | Objectif |
|---|-------|----------|
| a | Trouver `print` Python | Naviguer docs.python.org |
| b | Trouver `printf` C | Lire man page |
| c | Trouver `println!` Rust | Naviguer doc.rust-lang.org |
| d | Comparer les trois | Comprendre les différences |

### 📝 Exercice B.1.4 : Explorer print/printf/println!

```
EXERCICE B.1.4 : Exploration Comparative
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Comparer la même fonctionnalité dans 3 langages

TÂCHE A : print() en Python
━━━━━━━━━━━━━━━━━━━━━━━━━━━
URL trouvée : ___________________________________________
Signature complète : ___________________________________________
Paramètres optionnels (liste) : ___________________________________________
Exemple copié de la doc :
```python
# Colle ici un exemple de la doc
```

TÂCHE B : printf() en C
━━━━━━━━━━━━━━━━━━━━━━━
Commande man utilisée : ___________________________________________
Section du man : ___________________________________________
Format specifiers (5 exemples) : ___________________________________________
Valeur de retour : ___________________________________________

TÂCHE C : println!() en Rust
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
URL trouvée : ___________________________________________
Pourquoi c'est une macro (!) : ___________________________________________
Placeholder pour variable : ___________________________________________
Placeholder pour debug : ___________________________________________

TÂCHE D : Tableau Comparatif
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
| Aspect | Python print() | C printf() | Rust println!() |
|--------|----------------|------------|-----------------|
| Type | fonction | fonction | _________ |
| Newline auto | _________ | _________ | _________ |
| Format string | _________ | _________ | _________ |
| Type-safe | _________ | _________ | _________ |
| Retourne | _________ | _________ | _________ |
```

---

# B.2 : Méthodologie de Debug (25h)

---

## B.2.1 : Types d'Erreurs

### 📚 Concepts du Programme

| # | Type | Description |
|---|------|-------------|
| a | Erreur de syntaxe | Code mal écrit |
| b | Erreur de compilation | Ne compile pas |
| c | Erreur d'exécution | Crash pendant exécution |
| d | Erreur logique | Fonctionne mais mal |
| e | Erreur de performance | Trop lent |

### 🔧 Ressources Interactives

#### Outils de Détection par Type

| Type | Outil | URL | Usage |
|------|-------|-----|-------|
| **a** Syntaxe | ESLint | eslint.org | Linter JavaScript |
| **a** Syntaxe | Pylint | pylint.org | Linter Python |
| **b** Compilation | Compiler Explorer | godbolt.org | Voir erreurs compilo |
| **c** Exécution | Python Tutor | pythontutor.com | Visualiser crash |
| **c** Exécution | RustViz | github.com/rustviz/rustviz | Visualisation ownership Rust |
| **d** Logique | Debugger VS Code | - | Breakpoints |
| **e** Performance | Chrome DevTools | - | Profiler |

#### Jeux pour Identifier les Erreurs

| Jeu | URL | Description | Gratuit |
|-----|-----|-------------|---------|
| **What It Prints** | whatitprints.com | Deviner l'output d'un code | ✅ |
| **Gidget** | helpgidget.org | Jeu de debugging pour débutants | ✅ |
| **Python Tutor** | pythontutor.com | Visualiser l'exécution pas à pas | ✅ |
| **Tynker** | tynker.com | Jeu debug pour enfants/débutants | Freemium |
| **LearnCS Debugging** | learncs.online/debug/java | Exercices debugging Java | ✅ |
| **LaunchCode** | education.launchcode.org | Exercices debugging JS guidés | ✅ |
| **CodeChef Learn** | codechef.com/learn | Exercices debugging Python | ✅ |
| **Kidlo Coding** | kidlocoding.com/debug | Debugging pour enfants | ✅ |

#### 🎬 Vidéos Recommandées

| Vidéo | Auteur | Concepts couverts |
|-------|--------|-------------------|
| **Debug School 1: Incrementalism** | Toby Ho | Stratégie petits changements |
| Common Python Errors | Corey Schafer | a, c |
| Understanding Compiler Errors | The Cherno | b |

### 📝 Exercice B.2.1 : Classification des Erreurs

```
EXERCICE B.2.1 : Error Taxonomy
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Classifier 15 erreurs par type (a-e)

Classifie chaque situation :

1.  print("Hello)                              → ___
2.  x = 10 / 0                                 → ___
3.  for i in range(10): (boucle infinie)       → ___
4.  #include <fichier_inexistant.h>            → ___
5.  int *p; *p = 5; (pointeur non initialisé)  → ___
6.  Fonction retourne 4 au lieu de 5           → ___
7.  def foo( (parenthèse non fermée)           → ___
8.  Algorithme O(n³) sur 1M éléments           → ___
9.  liste[100] quand len(liste) == 5           → ___
10. Variable utilisée avant déclaration        → ___
11. Typo dans nom de variable                  → ___
12. Mauvais opérateur (+ au lieu de *)         → ___
13. Type mismatch en compilation               → ___
14. Memory leak après 2h d'exécution           → ___
15. Race condition multi-thread                → ___

Réponses :
1-a, 2-c, 3-d, 4-b, 5-c, 6-d, 7-a, 8-e,
9-c, 10-a/b, 11-a, 12-d, 13-b, 14-e, 15-d/e
```

---

## B.2.2 : Lire un Message d'Erreur

### 📚 Concepts du Programme

| # | Concept | Description |
|---|---------|-------------|
| a | Type d'erreur | Syntaxe, type, etc. |
| b | Localisation | Fichier:ligne:colonne |
| c | Message | Description du problème |
| d | Suggestion | Correction proposée |
| e | Stack trace | Chemin d'exécution |
| f | Lire de bas en haut | Pour stack traces |

### 🎬 SÉRIE VIDÉO ESSENTIELLE : Debug School (Toby Ho)

| Épisode | Titre | Durée | Concepts couverts |
|---------|-------|-------|-------------------|
| **2** | **Reading Error Messages 1** | ~10 min | a, b, c, d |
| **3** | **Reading Error Messages 2** | 12:32 | **e, f - Stack Traces** |

> 🔗 Série complète : tobyho.com/video-series/Debug-School.html

### 🔬 Anatomie d'un Message d'Erreur

```
┌─────────────────────────────────────────────────────────────────┐
│ Traceback (most recent call last):        ← (e) STACK TRACE    │
│   File "app.py", line 15, in <module>     ← (b) LOCALISATION   │
│     result = process(data)                ← CODE CONCERNÉ      │
│   File "utils.py", line 8, in process     ← APPEL PRÉCÉDENT    │
│     return data.split(',')                ← ORIGINE RÉELLE     │
│ AttributeError: 'NoneType' has no 'split' ← (a) TYPE + (c) MSG │
│                                                                 │
│ Hint: Did you mean to check if data is None? ← (d) SUGGESTION  │
└─────────────────────────────────────────────────────────────────┘

RÈGLE D'OR (f) : Lire la DERNIÈRE ligne en premier, puis REMONTER
```

### 🔧 Ressources Interactives

| Outil | URL | Usage |
|-------|-----|-------|
| **Python Tutor** | pythontutor.com | Visualiser le stack |
| **Sentry** | sentry.io | Erreurs en production |
| **ExplainShell** | explainshell.com | Erreurs shell |

### 📚 Guides Stack Traces

| Ressource | URL | Langage |
|-----------|-----|---------|
| **Twilio Blog** | twilio.com/blog | Java stack traces |
| **Samebug** | samebug.io | "Stack Trace 101" |
| **Microsoft Learn Video** | learn.microsoft.com | Node.js stack traces |
| **Brown University CS PDF** | cs.brown.edu | Python stack traces |

### 📝 Exercice B.2.2 : Dissection d'Erreurs

```
EXERCICE B.2.2a : Error Dissector
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Identifier chaque composant d'un message d'erreur

Analyse cette erreur Python :

Traceback (most recent call last):
  File "main.py", line 23, in <module>
    result = calculate_average(grades)
  File "main.py", line 10, in calculate_average
    return sum(numbers) / len(numbers)
ZeroDivisionError: division by zero

(a) Type d'erreur      : _______________________
(b) Fichier            : _______________________
(b) Ligne origine      : _______________________
(c) Message            : _______________________
(d) Suggestion         : _______________________ (aucune ici)
(e) Stack trace        : _______________________ lignes
(f) Première ligne à lire : _______________________
    Cause probable     : _______________________
    Solution           : _______________________
```

```
EXERCICE B.2.2b : Stack Trace Challenge (5 erreurs en 10 min)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Pour chaque erreur, identifie : Type | Fichier:Ligne | Cause

ERREUR 1:
TypeError: Cannot read properties of undefined (reading 'map')
    at renderList (App.jsx:42:18)
    at processData (utils.js:15:10)
→ Type: ___________ | Localisation: ___________ | Cause: ___________

ERREUR 2:
KeyError: 'username'
    at get_user (auth.py:23)
    at login (views.py:45)
→ Type: ___________ | Localisation: ___________ | Cause: ___________

ERREUR 3:
RecursionError: maximum recursion depth exceeded
    at factorial (math.py:8)
    at factorial (math.py:8) [repeated 996 times]
→ Type: ___________ | Localisation: ___________ | Cause: ___________

ERREUR 4:
ENOENT: no such file or directory, open 'config.json'
    at Object.openSync (fs.js:443:3)
    at loadConfig (app.js:12:15)
→ Type: ___________ | Localisation: ___________ | Cause: ___________

ERREUR 5:
error[E0382]: borrow of moved value: `data`
  --> src/main.rs:15:20
   |
14 |     let result = process(data);
   |                          ---- value moved here
15 |     println!("{}", data);
   |                    ^^^^ value borrowed here after move
→ Type: ___________ | Localisation: ___________ | Cause: ___________
```

---

## B.2.3 : Processus de Debug Systématique

### 📚 Concepts du Programme

| # | Étape | Description |
|---|-------|-------------|
| a | Reproduire | Confirmer le problème |
| b | Isoler | Trouver le code minimal |
| c | Hypothèses | Lister causes possibles |
| d | Tester | Vérifier chaque hypothèse |
| e | Corriger | Appliquer la solution |
| f | Vérifier | Confirmer la correction |
| g | Documenter | Noter pour le futur |

### 🎬 VIDÉOS Debug School (Toby Ho)

| Épisode | Titre | Concept couvert |
|---------|-------|-----------------|
| **1** | Incrementalism | Prévention |
| **4** | **Scientific Method** | c, d |
| **5** | **Test Case Simplification** | a, b |
| **6** | **The Bug Sandwich** | b (Binary search) |

### 🔧 Ressources Interactives

| Outil | URL | Étapes couvertes |
|-------|-----|------------------|
| **mysteries.wizardzines.com** | mysteries.wizardzines.com | a, b, c, d (Julia Evans puzzles) |
| **The Debugging Book** | debuggingbook.org | Tous |
| **Compiler Explorer** | godbolt.org | b, d |
| **Wizard Zines Debugging Guide** | wizardzines.com/zines/debugging-guide | Tous |

### 📝 Exercice B.2.3 : Debug Scientifique

```
EXERCICE B.2.3 : Méthode Scientifique Appliquée
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Suivre le processus systématique pour résoudre un bug

Code bugué :

def get_average_grade(students):
    total = 0
    for student in students:
        total += student['grade']
    return total / len(students)

# Test
students = [
    {'name': 'Alice', 'grade': 85},
    {'name': 'Bob', 'grade': 90},
    {'name': 'Charlie'}  # Oups !
]
print(get_average_grade(students))

JOURNAL DE DEBUG :

(a) REPRODUIRE
    - Erreur obtenue : _______________________
    - Reproductible : □ Oui □ Non
    - Conditions : _______________________

(b) ISOLER
    - Ligne problématique : _______________________
    - Code minimal pour reproduire :
    _______________________

(c) HYPOTHÈSES (liste 3 causes possibles)
    1. _______________________
    2. _______________________
    3. _______________________

(d) TESTER (pour chaque hypothèse)
    Test 1 : _______________________ → Résultat : _______
    Test 2 : _______________________ → Résultat : _______
    Test 3 : _______________________ → Résultat : _______

(e) CORRIGER
    Solution choisie : _______________________
    Code corrigé :
    _______________________

(f) VÉRIFIER
    - Test avec données originales : □ Pass □ Fail
    - Test avec liste vide : □ Pass □ Fail
    - Test avec tous les grades présents : □ Pass □ Fail

(g) DOCUMENTER
    Leçon apprise : _______________________
```

---

## B.2.4 : Techniques de Debug

### 📚 Concepts du Programme

| # | Technique | Usage |
|---|-----------|-------|
| a | Print debugging | Afficher des valeurs |
| b | Debugger | Points d'arrêt, pas à pas |
| c | Rubber duck | Expliquer à un canard |
| d | Binary search | Diviser le code en deux |
| e | Diff | Comparer versions |
| f | Rollback | Revenir à une version OK |
| g | Fresh eyes | Faire une pause |

### 🔧 Ressources par Technique

| Technique | Outil | URL/Commande | Tutoriel |
|-----------|-------|--------------|----------|
| **a** Print | console.log / print | - | - |
| **b** Debugger | VS Code Debugger | F5 | James Q Quick (YouTube) |
| **b** Debugger | Chrome DevTools | F12 → Sources | freeCodeCamp 2h30 |
| **b** Debugger | pdb (Python) | `breakpoint()` | Real Python |
| **c** Rubber Duck | rubberduckdebugging.com | rubberduckdebugging.com | Technique expliquée |
| **d** Binary | git bisect | `git bisect start` | Learn Git Branching |
| **e** Diff | VS Code | Ctrl+Shift+G | - |
| **e** Diff | diff / vimdiff | `diff -u old new` | - |
| **f** Rollback | git checkout | `git checkout HEAD~1` | - |

### 🎬 Vidéos Recommandées

| Vidéo | Auteur | Durée | Techniques |
|-------|--------|-------|------------|
| Chrome DevTools Full Course | freeCodeCamp | 2h30 | a, b |
| VS Code Debugger Tutorial | James Q Quick | ~20 min | b |
| Debugging Tips | Traversy Media | ~30 min | a, b, c |
| **Debug School 5: Test Case Simplification** | Toby Ho | ~10 min | d |
| **Debug School 6: The Bug Sandwich** | Toby Ho | ~10 min | d |
| Git Bisect Tutorial | The Modern Coder | ~15 min | d, f |

### 📝 Exercice B.2.4 : Maîtriser les Techniques

```
EXERCICE B.2.4a : Print Debugging Stratégique
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Utiliser les prints efficacement (pas au hasard)

Ajoute des prints STRATÉGIQUES à ce code (5 emplacements clés) :

function processOrder(order) {
  // Print 1: _______________
  const items = order.items;
  let total = 0;
  
  // Print 2: _______________
  for (let i = 0; i < items.length; i++) {
    // Print 3: _______________
    const price = items[i].price * items[i].quantity;
    total += price;
  }
  // Print 4: _______________
  
  const tax = total * 0.2;
  // Print 5: _______________
  return total + tax;
}

Écris les 5 console.log avec les bonnes infos à afficher.
```

```
EXERCICE B.2.4b : Debugger Challenge
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Maîtriser le debugger VS Code

1. Crée fibonacci.py avec une fonction récursive
2. Place un breakpoint ligne 1 de la fonction
3. Lance le debugger (F5)
4. Utilise :
   □ Step Over (F10) - 3 fois
   □ Step Into (F11) - entrer dans l'appel récursif
   □ Step Out (Shift+F11) - sortir
   □ Continue (F5) - jusqu'au prochain breakpoint

5. Dans le panneau VARIABLES, note :
   - Valeur de n à chaque appel : _______________
   - Profondeur max du Call Stack : _______________

6. Screenshot du Call Stack avec 5+ appels : □ Fait
```

```
EXERCICE B.2.4c : Binary Search Debug (Bug Sandwich)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Localiser un bug par dichotomie

Tu as un fichier de 64 lignes avec un bug.

Étape 1 : Commente lignes 33-64
  Bug présent ? □ Oui → bug dans 1-32
               □ Non → bug dans 33-64
  
Étape 2 : (selon résultat) Commente la moitié
  Bug présent ? □ Oui → _______________
               □ Non → _______________

Étape 3 : Continue jusqu'à trouver LA ligne

Nombre d'étapes pour 64 lignes : log2(64) = ___
Nombre d'étapes pour 1000 lignes : log2(1000) ≈ ___
```

---

## B.2.5 : Erreurs Courantes par Langage

### 📚 Concepts du Programme

| # | Langage | Erreurs Typiques |
|---|---------|------------------|
| a | Python | IndentationError, NameError, TypeError |
| b | C | Segfault, buffer overflow, memory leak |
| c | Rust | E0382 (moved), E0502 (borrow), lifetimes |

### 🔧 Ressources par Langage

#### a) Python - Erreurs Courantes

| Erreur | Cause | Solution | Ressource |
|--------|-------|----------|-----------|
| `IndentationError` | Mélange tabs/espaces | Configurer éditeur | Real Python |
| `NameError` | Variable non définie | Vérifier scope/typo | Python Tutor |
| `TypeError` | Mauvais type | Vérifier types | mypy (linter) |
| `KeyError` | Clé dict absente | `.get()` ou `in` | - |
| `IndexError` | Index hors limites | Vérifier len() | - |
| `AttributeError` | Attribut inexistant | Vérifier type objet | - |

#### b) C - Erreurs Courantes

| Erreur | Cause | Solution | Ressource |
|--------|-------|----------|-----------|
| `Segmentation fault` | Accès mémoire invalide | Valgrind, GDB | Valgrind |
| `Buffer overflow` | Écriture hors buffer | Vérifier tailles | AddressSanitizer |
| `Memory leak` | free() oublié | Valgrind | Valgrind |
| `Undefined behavior` | Code non défini | -Wall -Wextra | Compiler Explorer |
| `Double free` | free() 2 fois | Mettre à NULL | GDB |

**Outils de détection C :**
| Outil | URL | Usage |
|-------|-----|-------|
| **Valgrind** | valgrind.org | Détection memory leaks |
| **AddressSanitizer** | (intégré gcc/clang) | `-fsanitize=address` |
| **GDB** | gnu.org/software/gdb | Debugger C/C++ |

#### c) Rust - Erreurs Courantes

| Code | Erreur | Cause | Solution |
|------|--------|-------|----------|
| E0382 | "borrow of moved value" | Ownership transféré | Clone ou référence |
| E0502 | "cannot borrow as mutable" | Borrow rules | Revoir les borrows |
| E0106 | "missing lifetime specifier" | Lifetime ambigu | Ajouter `'a` |
| E0308 | "mismatched types" | Types incompatibles | Convertir |

### 🎬 Vidéos par Langage

| Langage | Vidéo | Auteur |
|---------|-------|--------|
| Python | 10 Common Python Mistakes | Corey Schafer |
| C | Common C Programming Errors | Jacob Sorber |
| C | Understanding Segfaults | Low Level Learning |
| Rust | Common Rust Mistakes | Let's Get Rusty |
| Rust | Rust Borrow Checker Explained | No Boilerplate |

### 📝 Exercice B.2.5 : Erreurs par Langage

```
EXERCICE B.2.5 : Diagnostic Multi-Langages
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Reconnaître et corriger les erreurs typiques

PYTHON - Corrige ces 3 erreurs :

# Erreur 1
def greet(name):
print("Hello " + name)  # ← Erreur : _______________
                         # ← Solution : _______________

# Erreur 2
user = {'name': 'Alice'}
print(user['age'])       # ← Erreur : _______________
                         # ← Solution : _______________

# Erreur 3  
numbers = [1, 2, 3]
print(numbers[5])        # ← Erreur : _______________
                         # ← Solution : _______________

C - Identifie le problème :

// Erreur 4
char buffer[10];
strcpy(buffer, "This is a very long string");
// ← Erreur : _______________
// ← Danger : _______________

// Erreur 5
int *ptr = malloc(sizeof(int));
*ptr = 42;
// (programme continue sans free)
// ← Erreur : _______________

RUST - Explique l'erreur du compilateur :

// Erreur 6
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    println!("{}", s1);  // ← Erreur E0382
}
// Explication : _______________
// Solution : _______________
```

---

## B.2.6 : Exercice — Debug Challenge

### 📚 Tâches du Programme

| # | Tâche | Objectif |
|---|-------|----------|
| a | Code bugué fourni | Trouver et corriger |
| b | Documenter le processus | Écrire les étapes |
| c | Expliquer la solution | Comprendre le pourquoi |

### 📝 Exercice B.2.6 : Debug Challenge Complet

```
EXERCICE B.2.6 : Debug Challenge
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Appliquer tout ce que tu as appris

CODE BUGUÉ (Python) - 5 bugs cachés :

def process_students(students):
    """Calcule la moyenne et trouve le meilleur étudiant."""
    
    # Bug 1 quelque part ici
    total = 0
    best_student = None
    best_grade = 0
    
    for i in range(len(students) + 1):  # Bug 2
        student = students[i]
        grade = student['grade']
        
        total += grade
        
        if grade > best_grade:  # Bug 3 (logique)
            best_grade = grade
            best_student = student['name']
    
    average = total / len(students)
    
    # Bug 4
    return {
        'average': average,
        'best': best_student,
        'total_students': len(students)
    }

# Test
data = [
    {'name': 'Alice', 'grade': 95},
    {'name': 'Bob', 'grade': 87},
    {'name': 'Charlie', 'grade': 92},
    {'name': 'Diana', 'grade': 95},  # Bug 5 : même note que Alice
]

result = process_students(data)
print(f"Moyenne: {result['average']}")
print(f"Meilleur: {result['best']}")

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TÂCHE A : Trouver et corriger les 5 bugs

Bug 1 : _______________________________________________________
  Ligne : ___  | Correction : ________________________________

Bug 2 : _______________________________________________________
  Ligne : ___  | Correction : ________________________________

Bug 3 : _______________________________________________________
  Ligne : ___  | Correction : ________________________________

Bug 4 : _______________________________________________________
  Ligne : ___  | Correction : ________________________________

Bug 5 : _______________________________________________________
  Ligne : ___  | Correction : ________________________________

TÂCHE B : Documenter le processus

Méthode utilisée pour Bug 1 : ________________________________
Méthode utilisée pour Bug 2 : ________________________________
Méthode utilisée pour Bug 3 : ________________________________
Temps total de debug : _______ minutes

TÂCHE C : Expliquer la solution

Pourquoi Bug 2 causait un crash ?
_______________________________________________________________

Pourquoi Bug 3 est une erreur LOGIQUE et pas une erreur de syntaxe ?
_______________________________________________________________

Comment aurais-tu pu PRÉVENIR Bug 5 dès la conception ?
_______________________________________________________________
```

---

# B.3 : Chercher de l'Aide Efficacement (15h)

---

## B.3.1 : Avant de Demander

### 📚 Concepts du Programme

| # | Concept | Description |
|---|---------|-------------|
| a | Lire la documentation | Toujours en premier |
| b | Chercher l'erreur | Google le message exact |
| c | Stack Overflow | Chercher avant de poster |
| d | GitHub Issues | Peut-être déjà reporté |
| e | 30 minutes minimum | Effort personnel |

### ⏱️ La Règle des 30 Minutes

```
CHECKLIST AVANT DE DEMANDER (30 min minimum)

□ [5 min]  (a) Lu la DOCUMENTATION officielle ?
□ [10 min] (b) Googlé l'erreur EXACTE (avec guillemets) ?
□ [5 min]  (c) Cherché sur Stack Overflow ?
□ [5 min]  (d) Vérifié GitHub Issues du projet ?
□ [5 min]  (e) Essayé de reproduire le problème isolément ?

✅ Si TOUT coché et toujours bloqué → Tu peux demander !
```

### 🔍 Google Like a Pro

| Opérateur | Exemple | Effet |
|-----------|---------|-------|
| `"exact"` | `"TypeError: Cannot read"` | Recherche exacte |
| `site:` | `site:stackoverflow.com python` | Limiter au site |
| `-` | `python tutorial -video` | Exclure un terme |
| `filetype:` | `filetype:pdf javascript` | Type de fichier |
| `OR` | `react OR vue tutorial` | Alternatives |
| `before:` | `python before:2023` | Avant une date |
| `after:` | `react hooks after:2022` | Après une date |

### 📚 Guides pour Googler les Erreurs

| Ressource | URL | Description |
|-----------|-----|-------------|
| **dev.to/swyx** | dev.to/swyx | "How To Google Your Errors" (guide complet) |
| **makeuseof.com** | makeuseof.com | "21 Tips to Master Googling as a Developer" |
| **gustavwengel.dk** | gustavwengel.dk | Série "Googling Error Messages" |
| **codeproject.com** | codeproject.com | "How to Use Google" pour devs |

### 🎬 VIDÉO ESSENTIELLE

| Vidéo | Auteur | Durée | Contenu |
|-------|--------|-------|---------|
| **How to Google Like a Pro** | freeCodeCamp (Seth Goldin) | **1 heure** | Tous les opérateurs, astuces avancées |

### 📝 Exercice B.3.1 : Recherche Avant Question

```
EXERCICE B.3.1a : Pre-Question Checklist
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Prouver que tu as cherché AVANT de demander

Pour ton PROCHAIN bug réel, remplis ce formulaire :

┌─────────────────────────────────────────────────────────────────┐
│ CHECKLIST PRÉ-QUESTION                           Date: _______ │
├─────────────────────────────────────────────────────────────────┤
│ Message d'erreur complet (copié) :                              │
│ ________________________________________________________________│
│ ________________________________________________________________│
│                                                                 │
│ (a) Documentation consultée :                                   │
│ URL 1: ________________________________________________________│
│ URL 2: ________________________________________________________│
│                                                                 │
│ (b) Recherches Google effectuées :                              │
│ Requête 1: ____________________________________________________│
│ Requête 2: ____________________________________________________│
│ Requête 3: ____________________________________________________│
│                                                                 │
│ (c) Stack Overflow - questions similaires trouvées :            │
│ URL 1: ________________________________________________________│
│ Pourquoi pas applicable : _____________________________________│
│                                                                 │
│ (d) GitHub Issues vérifiées : □ Oui □ Non □ N/A                 │
│                                                                 │
│ (e) Temps total passé : _______ minutes                         │
│                                                                 │
│ Prêt à demander ? □ Oui (30+ min) □ Non (continue à chercher)   │
└─────────────────────────────────────────────────────────────────┘
```

```
EXERCICE B.3.1b : Google-Fu Challenge
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Maîtriser les opérateurs de recherche

Écris la requête Google optimale pour :

1. Trouver comment inverser une liste en Python (Stack Overflow only)
   Requête : ________________________________________________

2. Trouver un PDF du livre "Clean Code"
   Requête : ________________________________________________

3. Tutoriel React publié après 2023, PAS de vidéo
   Requête : ________________________________________________

4. Erreur exacte "ECONNREFUSED" pour Node.js
   Requête : ________________________________________________

5. Documentation officielle de useState (React)
   Requête : ________________________________________________
```

---

## B.3.2 : Comment Poser une Bonne Question

### 📚 Concepts du Programme

| # | Concept | Description |
|---|---------|-------------|
| a | Titre clair | Résume le problème |
| b | Contexte | Ce que vous faites |
| c | Code minimal | MCVE |
| d | Erreur complète | Copier-coller |
| e | Ce que vous avez essayé | Montrer l'effort |
| f | Environnement | OS, versions |

### 📋 Template de Bonne Question

```markdown
## [Langage] Titre clair et spécifique en une ligne

### Ce que j'essaie de faire (b - Contexte)
Je développe [description] et j'essaie de [objectif].

### Ce qui se passe (d - Erreur complète)
Quand j'exécute le code, j'obtiens cette erreur :
```
[Copier-coller l'erreur COMPLÈTE ici]
```

### Code pour reproduire (c - MCVE)
```python
# Code MINIMAL qui reproduit le problème
```

### Ce que j'ai essayé (e - Efforts)
1. J'ai essayé [solution 1] → [résultat]
2. J'ai consulté [documentation] → [ce que j'ai compris]
3. J'ai trouvé [SO question] mais [pourquoi pas applicable]

### Environnement (f)
- OS : 
- Langage/Version : 
- Dépendances :
```

### 🎬 Vidéos Recommandées

| Vidéo | Auteur | Contenu |
|-------|--------|---------|
| How to Use Stack Overflow | codeSTACKr | a, c, e |
| Asking Good Questions | Codecademy | Tous |

### 📚 Guides Stack Overflow

| Ressource | URL | Description |
|-----------|-----|-------------|
| **Stack Overflow Ask Wizard** | stackoverflow.com | Guidage interactif étape par étape |
| **Stack Overflow Staging Ground** | stackoverflow.com | Review avant publication |
| **dataschool.io** | dataschool.io | "How to Write a Great SO Question" |
| **stackoverflow.blog** | stackoverflow.blog | "Getting started: guide for students" |
| **ruddra.com** | ruddra.com | "How to Ask Questions in Stack Overflow" |

### 📝 Exercice B.3.2 : Rédiger une Question

```
EXERCICE B.3.2 : Question Makeover
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Transformer une mauvaise question en bonne question

MAUVAISE QUESTION :
"Mon code marche pas aidez moi svp !!! 😭😭😭"

RÉÉCRIS selon le template :

(a) Titre :
_______________________________________________________________

(b) Contexte :
_______________________________________________________________
_______________________________________________________________

(c) Code minimal :
_______________________________________________________________
_______________________________________________________________
_______________________________________________________________

(d) Erreur complète :
_______________________________________________________________
_______________________________________________________________

(e) Ce que j'ai essayé :
1. ____________________________________________________________
2. ____________________________________________________________

(f) Environnement :
- OS : _________________
- Langage : _________________
- Version : _________________
```

---

## B.3.3 : Où Demander de l'Aide

### 📚 Concepts du Programme

| # | Ressource | Usage |
|---|-----------|-------|
| a | Stack Overflow | Questions techniques |
| b | Discord/Slack | Temps réel |
| c | Reddit | Discussion |
| d | Forums officiels | Spécifique au langage |
| e | GitHub Discussions | Spécifique au projet |

### 🌐 Ressources Détaillées

#### a) Stack Overflow

| Aspect | Détail |
|--------|--------|
| URL | stackoverflow.com |
| Type | Q&A formel |
| Temps réponse | Minutes à heures |
| Pour | Questions techniques précises |
| Éviter | Questions d'opinion, trop larges |

#### b) Discord/Slack (Temps réel)

| Communauté | URL | Spécialité |
|------------|-----|------------|
| Reactiflux | discord.gg/reactiflux | React/JS |
| Python Discord | discord.gg/python | Python |
| Rust Community | discord.gg/rust-lang | Rust |
| The Coding Den | discord.gg/code | Général |
| CS Career Hub | discord.gg/cscareers | Carrière |

#### c) Reddit

| Subreddit | Membres | Pour |
|-----------|---------|------|
| r/learnprogramming | 4M+ | Débutants |
| r/programming | 5M+ | News, discussion |
| r/webdev | 2M+ | Web développement |
| r/Python | 1M+ | Python |
| r/rust | 300k+ | Rust |

#### d) Forums Officiels

| Langage | Forum |
|---------|-------|
| Python | discuss.python.org |
| Rust | users.rust-lang.org |
| Go | forum.golangbridge.org |
| JavaScript | dev.to (communauté) |

#### e) GitHub Discussions

| Usage | Exemple |
|-------|---------|
| Question sur un projet | github.com/facebook/react/discussions |
| Feature request | Discussions > Ideas |
| Bug confirmé | Issues (pas Discussions) |

### 📝 Exercice B.3.3 : Choisir la Bonne Plateforme

```
EXERCICE B.3.3 : Platform Picker
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Choisir la plateforme appropriée

Où poserais-tu ces questions ? (a-e)

1. "Comment utiliser useEffect avec un tableau vide ?"
   → Plateforme : ___ | Raison : ___________________________

2. "J'ai trouvé un bug dans la lib Axios"
   → Plateforme : ___ | Raison : ___________________________

3. "Quel framework JS apprendre en 2026 ?"
   → Plateforme : ___ | Raison : ___________________________

4. "Quelqu'un peut m'aider en live sur mon code ?"
   → Plateforme : ___ | Raison : ___________________________

5. "Est-ce que Python 3.12 supporte X feature ?"
   → Plateforme : ___ | Raison : ___________________________

6. "Mon code crash avec cette stack trace [...]"
   → Plateforme : ___ | Raison : ___________________________
```

---

## B.3.4 : MCVE — Minimal Complete Verifiable Example

### 📚 Concepts du Programme

| # | Caractéristique | Description |
|---|-----------------|-------------|
| a | Minimal | Le moins de code possible |
| b | Complete | On peut le copier et exécuter |
| c | Verifiable | Reproduit le problème |
| d | Example | Code, pas description |

### 🎬 VIDÉO Debug School

| Épisode | Titre | Concept MCVE |
|---------|-------|--------------|
| **5** | **Test Case Simplification** | Comment réduire le code au minimum |

### 🔧 Méthode de Création MCVE

```
ÉTAPES POUR CRÉER UN MCVE :

1. COPIER le code bugué dans un NOUVEAU fichier
2. SUPPRIMER tout ce qui n'est PAS lié au bug :
   - Imports non utilisés
   - Fonctions non appelées
   - Variables non liées au problème
   - Commentaires
3. REMPLACER les données réelles par des données fake simples
4. VÉRIFIER que le bug est toujours présent
5. RÉPÉTER jusqu'à ne plus pouvoir supprimer de code

CRITÈRES DE VALIDATION :
□ (a) Minimal : Chaque ligne est NÉCESSAIRE pour reproduire
□ (b) Complete : Copier-coller → exécuter sans modification
□ (c) Verifiable : Le bug apparaît à chaque exécution
□ (d) Example : C'est du CODE, pas une description
```

### 📝 Exercice B.3.4 : Créer un MCVE

```
EXERCICE B.3.4 : Reduce This Bug
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Réduire un code bugué au strict minimum

CODE ORIGINAL (40+ lignes) :

import json
import os
from datetime import datetime
from typing import List, Dict

class UserManager:
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.users: List[Dict] = []
        self.load_users()
    
    def load_users(self):
        if os.path.exists(self.db_path):
            with open(self.db_path, 'r') as f:
                self.users = json.load(f)
    
    def save_users(self):
        with open(self.db_path, 'w') as f:
            json.dump(self.users, f, indent=2)
    
    def add_user(self, name: str, email: str):
        user = {
            'id': len(self.users) + 1,
            'name': name,
            'email': email,
            'created': datetime.now(),  # ← BUG ICI
            'active': True
        }
        self.users.append(user)
        self.save_users()
    
    def get_user(self, user_id: int):
        for user in self.users:
            if user['id'] == user_id:
                return user
        return None

# Utilisation
manager = UserManager('users.json')
manager.add_user('Alice', 'alice@example.com')
print("User added!")

ERREUR : TypeError: Object of type datetime is not JSON serializable

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TÂCHE : Réduis ce code au MCVE (cible : < 10 lignes)

TON MCVE :
_______________________________________________________________
_______________________________________________________________
_______________________________________________________________
_______________________________________________________________
_______________________________________________________________
_______________________________________________________________
_______________________________________________________________
_______________________________________________________________

VALIDATION :
□ (a) Minimal : Combien de lignes ? ___ (cible: <10)
□ (b) Complete : Peut s'exécuter tel quel ? □ Oui □ Non
□ (c) Verifiable : Même erreur ? □ Oui □ Non
□ (d) Example : C'est du code ? □ Oui □ Non
```

---

# B.4 : Construire son Autonomie (10h)

---

## B.4.1 : Ressources pour Rester à Jour

### 📚 Concepts du Programme

| # | Ressource | Type |
|---|-----------|------|
| a | Newsletters | Hebdomadaires |
| b | Blogs techniques | Articles de fond |
| c | YouTube | Tutoriels, conférences |
| d | Podcasts | Écoute passive |
| e | Twitter/X | Actualités rapides |
| f | Conférences | PyCon, RustConf, etc. |

### 🔧 Ressources Recommandées

#### a) Newsletters

| Newsletter | URL | Fréquence | Langue |
|------------|-----|-----------|--------|
| **TLDR** | tldr.tech | Quotidien | EN |
| **JavaScript Weekly** | javascriptweekly.com | Hebdo | EN |
| **Python Weekly** | pythonweekly.com | Hebdo | EN |
| **Rust Weekly** | this-week-in-rust.org | Hebdo | EN |
| **Hacker Newsletter** | hackernewsletter.com | Hebdo | EN |
| **Dev.to Weekly** | dev.to/newsletter | Hebdo | EN |
| **The Overflow** | stackoverflow.blog/newsletter | Hebdo | EN |
| **Bytes** | bytes.dev | Hebdo | EN |
| **Morning Brew Tech** | morningbrew.com | Quotidien | EN |

#### b) Blogs Techniques

| Blog | URL | Spécialité |
|------|-----|------------|
| **Dev.to** | dev.to | Communauté |
| **Hacker News** | news.ycombinator.com | Tech générale |
| **HackerNoon** | hackernoon.com | Articles tech |
| **Lobsters** | lobste.rs | News curated |
| **Daily.dev** | daily.dev | Agrégateur personnalisé |
| **Hashnode** | hashnode.dev | Blogs développeurs |
| **CSS-Tricks** | css-tricks.com | CSS/Frontend |
| **Real Python** | realpython.com | Python |
| **LogRocket Blog** | blog.logrocket.com | Frontend |
| **Julia Evans Blog** | jvns.ca | Zines, debugging, Linux |

#### c) YouTube - Voir [Annexe C](#annexe-c--chaînes-youtube-recommandées)

#### d) Podcasts

| Podcast | Langue | Durée moyenne | Spécialité |
|---------|--------|---------------|------------|
| **Syntax** | EN | 30-60 min | Web dev |
| **CodeNewbie** | EN | 45 min | Débutants |
| **Talk Python to Me** | EN | 60 min | Python |
| **The Changelog** | EN | 60 min | Open source |
| **Software Engineering Daily** | EN | 60 min | Interviews techniques |
| **Developer Tea** | EN | 10-30 min | Introspection dev |
| **Artisan Développeur** | FR | 45 min | Dev général |
| **Ifttd** | FR | 30 min | Tech FR |

#### e) Twitter/X - Comptes à suivre

| Compte | Spécialité |
|--------|------------|
| @addyosmani | Web performance |
| @dan_abramov | React |
| @ThePrimeagen | Rust, performance |
| @firlogsop | Fireship |
| @swaborni | Python |

#### f) Conférences

| Conférence | Langage | Vidéos gratuites |
|------------|---------|------------------|
| **PyCon** | Python | youtube.com/pycon |
| **JSConf** | JavaScript | youtube.com/jsconf |
| **RustConf** | Rust | youtube.com/rustconf |
| **FOSDEM** | Open source | fosdem.org |
| **Devoxx** | Java/JVM | youtube.com/daborni |

### 📝 Exercice B.4.1 : Information Diet

```
EXERCICE B.4.1 : Créer ta Routine d'Apprentissage
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Définir un régime informationnel équilibré

QUOTIDIEN (15-20 min max)
□ Newsletter : _______________________
□ Subreddit/Site : _______________________

HEBDOMADAIRE (1-2h)
□ Podcast (pendant trajet/sport) : _______________________
□ Vidéo YouTube (1-2) : _______________________
□ Article de blog (1-2) : _______________________

MENSUEL (4-8h)
□ Nouveau framework/outil à explorer : _______________________
□ Side project à avancer : _______________________
□ Conférence à regarder : _______________________

S'ABONNER MAINTENANT :
□ Newsletter 1 : _______________________
□ Newsletter 2 : _______________________
□ Chaîne YouTube : _______________________
□ Podcast : _______________________
```

---

## B.4.2 : Pratiquer Continuellement

### 📚 Concepts du Programme

| # | Ressource | Type |
|---|-----------|------|
| a | LeetCode | Algorithmes |
| b | Exercism | Par langage |
| c | Advent of Code | Puzzles annuels |
| d | Codewars | Katas |
| e | Projets personnels | Le meilleur |
| f | Open source | Contribuer |

### 🔧 Plateformes de Practice

| Plateforme | URL | Spécialité | Gratuit |
|------------|-----|------------|---------|
| **a) LeetCode** | leetcode.com | Algorithmes, FAANG | Freemium |
| **b) Exercism** | exercism.org | 65 langages, mentorat | ✅ 100% |
| **c) Advent of Code** | adventofcode.com | Puzzles décembre | ✅ 100% |
| **d) Codewars** | codewars.com | Katas, gamification | ✅ |
| **HackerRank** | hackerrank.com | Certifications | Freemium |
| **Project Euler** | projecteuler.net | Maths + code | ✅ 100% |
| **Coding Game** | codingame.com | Jeux, bots | ✅ |
| **CheckiO** | checkio.org | Python/JS island | ✅ |
| **CodeChef** | codechef.com | Contests, practice | ✅ |

### 🏆 Competitive Programming (Avancé)

| Plateforme | URL | Spécialité | Niveau |
|------------|-----|------------|--------|
| **Codeforces** | codeforces.com | Contests, ratings | Intermédiaire+ |
| **AtCoder** | atcoder.jp | Contests japonais | Intermédiaire+ |
| **TopCoder** | topcoder.com | Challenges, prizes | Avancé |
| **Kattis** | kattis.com | Problèmes académiques | Tous niveaux |
| **CP Book** | cpbook.net | Livre + exercices | Tous niveaux |

### 🎓 Plateformes d'Apprentissage Complètes

| Plateforme | URL | Spécialité | Gratuit |
|------------|-----|------------|---------|
| **freeCodeCamp** | freecodecamp.org | Web dev, certifications | ✅ 100% |
| **The Odin Project** | theodinproject.com | Full-stack curriculum | ✅ 100% |
| **Khan Academy** | khanacademy.org | CS, maths, vidéos | ✅ 100% |
| **Codecademy** | codecademy.com | Multi-langages, interactif | Freemium |
| **Scrimba** | scrimba.com | Screencasts interactifs | Freemium |
| **Brilliant** | brilliant.org | Maths, CS, puzzles | Freemium |
| **Boot.dev** | boot.dev | Backend, gamifié | Freemium |
| **CodeHS** | codehs.com | K-12, curriculum | Freemium |
| **LabEx** | labex.io | Labs hands-on | Freemium |
| **4Geeks** | 4geeks.com | Exercices interactifs | ✅ |

### 📚 Tutoriels Interactifs par Langage

| Tutoriel | URL | Langage | Gratuit |
|----------|-----|---------|---------|
| **LearnPython.org** | learnpython.org | Python | ✅ |
| **LearnShell.org** | learnshell.org | Bash/Shell | ✅ |
| **LearnSQL.com** | learnsql.com | SQL | Freemium |
| **Runestone Academy** | runestone.academy | Python, textbooks | ✅ |
| **W3Schools** | w3schools.com | Web (HTML/CSS/JS/SQL) | ✅ |
| **Invent with Python** | inventwithpython.com | Python, jeux | ✅ gratuit |
| **Real Python** | realpython.com | Python | Freemium |

### 👨‍🏫 Mentorat

| Plateforme | URL | Description |
|------------|-----|-------------|
| **Exercism** | exercism.org | Mentorat gratuit par humains |
| **Codementor** | codementor.io | Sessions 1-on-1 payantes |

### 🌱 Contribution Open Source (Premiers Pas)

| Ressource | URL | Description |
|-----------|-----|-------------|
| **First Contributions** | firstcontributions.github.io | Tutoriel interactif première contribution |
| **first-contributions repo** | github.com/firstcontributions | Repo de pratique |
| **opensource.guide** | opensource.guide/how-to-contribute | Guide officiel GitHub |
| **firsttimersonly.com** | firsttimersonly.com | Issues "first-timers-only" |
| **goodfirstissue.dev** | goodfirstissue.dev | Agrégateur par langage |
| **up-for-grabs.net** | up-for-grabs.net | Projets accueillant débutants |
| **contribute.cncf.io** | contribute.cncf.io | Mentorat CNCF cloud-native |
| **CodeTriage** | codetriage.com | Issues à trier, gamifié |

### 🎮 Jeux de Programmation (par technologie)

Voir [Annexe A](#annexe-a--jeux-interactifs-par-technologie) pour la liste complète.

### 📝 Exercice B.4.2 : Plan de Practice

```
EXERCICE B.4.2 : 30-Day Coding Challenge
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Coder minimum 30 minutes par jour pendant 30 jours

CHOIX DE PLATEFORME :
□ LeetCode (algorithmes)
□ Exercism (langage spécifique : _____________)
□ Codewars (katas variés)
□ Autre : _____________

SUIVI QUOTIDIEN :

Semaine 1 : □ □ □ □ □ □ □   Exercices : ___/7
Semaine 2 : □ □ □ □ □ □ □   Exercices : ___/7
Semaine 3 : □ □ □ □ □ □ □   Exercices : ___/7
Semaine 4 : □ □ □ □ □ □ □   Exercices : ___/7
Bonus     : □ □              Exercices : ___/2

STATS FINALES :
- Total exercices complétés : _____
- Streak le plus long : _____ jours
- Concept le plus difficile : _______________________
- Concept maîtrisé : _______________________
```

---

## B.4.3 : Créer son Guide Personnel

### 📚 Concepts du Programme

| # | Section | Contenu |
|---|---------|---------|
| a | Cheat sheet | Syntaxe rapide |
| b | Erreurs courantes | Tes erreurs + solutions |
| c | Ressources | Bookmarks organisés |
| d | Snippets | Code réutilisable |
| e | Notes | Ce que tu apprends |

### 🔧 Outils Recommandés

| Outil | URL | Points forts | Gratuit |
|-------|-----|--------------|---------|
| **Obsidian** | obsidian.md | Liens bidirectionnels, offline | ✅ |
| **Notion** | notion.so | Bases de données, templates | Freemium |
| **Logseq** | logseq.com | Open source, outliner | ✅ |
| **Trilium Next** | github.com/TriliumNext | Self-hosted, hiérarchique | ✅ |
| **GitHub Gists** | gist.github.com | Snippets versionnés | ✅ |
| **Devhints** | devhints.io | Cheatsheets prêtes | ✅ |
| **Raindrop.io** | raindrop.io | Bookmarks organisés, tags | Freemium |
| **Pocket** | getpocket.com | Save for later | Freemium |

### 📖 Méthodes Second Brain

| Méthode | Description | Ressource |
|---------|-------------|-----------|
| **PARA** | Projects, Areas, Resources, Archives | Tiago Forte |
| **Zettelkasten** | Notes atomiques interconnectées | Niklas Luhmann |
| **LYT** | Linking Your Thinking, Maps of Content | Nick Milo |
| **CODE** | Capture, Organize, Distill, Express | Building a Second Brain (livre) |

### 📚 Guides et Templates

| Ressource | URL | Type |
|-----------|-----|------|
| **Notion Templates** | notion.com/templates/category/second-brain | Templates officiels |
| **mattgiaro.com** | mattgiaro.com/second-brain-obsidian | Guide Obsidian + templates |
| **dev.to** | dev.to/envoy_/obsidian-an-ide-for-your-brain | Guide développeur |

### 📝 Exercice B.4.3 : Structure de Guide Personnel

```
EXERCICE B.4.3 : Créer la Structure de ton Guide
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Objectif : Mettre en place une base de connaissances personnelle

STRUCTURE À CRÉER (Obsidian ou Notion) :

📁 Dev-Notes/
├── 📁 (a) Cheatsheets/
│   ├── Python-Basics.md
│   ├── Git-Commands.md
│   ├── Regex.md
│   └── [Ajoute 2 de plus] : ___________.md, ___________.md
│
├── 📁 (b) Erreurs/
│   ├── Template-Erreur.md
│   └── [Tes 3 premières erreurs documentées]
│
├── 📁 (c) Ressources/
│   ├── Documentation.md (20 liens)
│   ├── YouTube.md (10 chaînes)
│   └── Communautés.md (5 Discord/Reddit)
│
├── 📁 (d) Snippets/
│   ├── Python/
│   ├── JavaScript/
│   └── Shell/
│
└── 📁 (e) Notes/
    ├── 2026-01-06-Ce-que-j-ai-appris.md
    └── [Journal d'apprentissage]

TÂCHES :
□ Créer la structure de dossiers
□ Ajouter 1 cheatsheet
□ Documenter 1 erreur résolue
□ Ajouter 20 bookmarks dans Ressources
□ Sauvegarder 3 snippets utiles
```

---

## B.4.4 : Mini-projet B — Mon Guide de Référence

### 📚 Livrables du Programme

| # | Livrable | Contenu |
|---|----------|---------|
| a | Créer structure | Dossiers organisés |
| b | Cheat sheet vide | Template à remplir |
| c | Premiers bookmarks | 20 liens essentiels |
| d | Template erreur | Format pour documenter |
| e | Premier snippet | Code utile |

### 📝 Mini-Projet B : Guide de Référence Personnel

```
MINI-PROJET B : Mon Guide de Référence
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Durée estimée : 2-3 heures
Outil : Obsidian, Notion, ou dossier simple

═══════════════════════════════════════════════════════════════════

LIVRABLE A : Structure de Dossiers
───────────────────────────────────
□ Créer la hiérarchie complète (voir B.4.3)
□ Screenshot de la structure : [ ]

═══════════════════════════════════════════════════════════════════

LIVRABLE B : Template de Cheat Sheet
────────────────────────────────────
Créer un fichier Template-Cheatsheet.md :

```markdown
# [Langage/Outil] Cheat Sheet

## Syntaxe de Base
| Concept | Syntaxe | Exemple |
|---------|---------|---------|
| ... | ... | ... |

## Fonctions Courantes
- ...

## Pièges à Éviter
- ...

## Ressources
- Doc officielle : [lien]
```

□ Template créé : [ ]

═══════════════════════════════════════════════════════════════════

LIVRABLE C : 20 Bookmarks Essentiels
────────────────────────────────────
Organise 20 liens dans Ressources.md :

DOCUMENTATION (5) :
1. _______________________________
2. _______________________________
3. _______________________________
4. _______________________________
5. _______________________________

TUTORIELS (5) :
6. _______________________________
7. _______________________________
8. _______________________________
9. _______________________________
10. ______________________________

OUTILS (5) :
11. ______________________________
12. ______________________________
13. ______________________________
14. ______________________________
15. ______________________________

COMMUNAUTÉS (5) :
16. ______________________________
17. ______________________________
18. ______________________________
19. ______________________________
20. ______________________________

□ 20 bookmarks ajoutés : [ ]

═══════════════════════════════════════════════════════════════════

LIVRABLE D : Template d'Erreur
──────────────────────────────
Créer un fichier Template-Erreur.md :

```markdown
# [Type]: [Message court]

## Erreur Complète
```
[Copier l'erreur ici]
```

## Contexte
- Langage : 
- Ce que je faisais :

## Cause
[Explication de la cause]

## Solution
```code
[Code corrigé]
```

## Leçon Apprise
[Ce que j'ai retenu]

## Tags
#erreur #[langage] #[type]
```

□ Template créé : [ ]
□ Première erreur documentée : [ ]

═══════════════════════════════════════════════════════════════════

LIVRABLE E : Premier Snippet
────────────────────────────
Créer un snippet utile que tu réutilises souvent :

Fichier : Snippets/[langage]/[nom].md

```markdown
# [Nom du Snippet]

## Usage
[Quand utiliser ce code]

## Code
```[langage]
[Ton code ici]
```

## Exemple
```[langage]
[Exemple d'utilisation]
```
```

□ Premier snippet créé : [ ]

═══════════════════════════════════════════════════════════════════

VALIDATION FINALE :
□ (a) Structure créée
□ (b) Template cheatsheet prêt
□ (c) 20 bookmarks organisés
□ (d) Template erreur + 1 erreur documentée
□ (e) 1 snippet sauvegardé

Date de complétion : _______________
```

---

# Annexe A : Jeux Interactifs par Technologie

## 🎨 CSS (10 jeux)

| Jeu | URL | Concept | Niveaux | Gratuit |
|-----|-----|---------|---------|---------|
| **Flexbox Froggy** | flexboxfroggy.com | Flexbox | 24 | ✅ |
| **Grid Garden** | cssgridgarden.com | CSS Grid | 28 | ✅ |
| **CSS Diner** | flukeout.github.io | Sélecteurs | 32 | ✅ |
| **Flexbox Defense** | flexboxdefense.com | Tower defense | 12 | ✅ |
| **Flexbox Zombies** | mastery.games/flexboxzombies | Story-driven | 12 ch. | Freemium |
| **Flexbox Adventure** | codingfantasy.com/games/flexboxadventure | RPG | 24 | ✅ |
| **Grid Attack** | codingfantasy.com/games/css-grid-attack | Combat | 80 | ✅ |
| **CSS Battle** | cssbattle.dev | Réplication | Illimité | ✅ |
| **CSS Speedrun** | css-speedrun.netlify.app | Sélecteurs chrono | 10 | ✅ |
| **Guess CSS** | guess-css.app | Quiz | Infini | ✅ |
| **Codepip** | codepip.com | Plateforme jeux CSS | Multi | Freemium |

## 💛 JavaScript (8 jeux)

| Jeu | URL | Concept | Gratuit |
|-----|-----|---------|---------|
| **Elevator Saga** | play.elevatorsaga.com | Algorithmes | ✅ |
| **Untrusted** | alexnisnevich.github.io/untrusted | Roguelike JS | ✅ |
| **WarriorJS** | warriorjs.com | Dungeon crawler | ✅ |
| **JSRobot** | lab.reaal.me/jsrobot | Puzzles | ✅ |
| **CheckiO** | checkio.org | Python/JS island | ✅ |
| **CodeCombat** | codecombat.com | RPG éducatif | Freemium |
| **Screeps** | screeps.com | MMO RTS | Payant |
| **Robocode** | robocode.sourceforge.io | Tank battle AI | ✅ |
| **Yare** | yare.io | RTS | ✅ |

## 🗃️ SQL (10 ressources)

| Ressource | URL | Type | Gratuit |
|-----------|-----|------|---------|
| **SQL Murder Mystery** | mystery.knightlab.com | Jeu détective | ✅ |
| **SQL Island** | sql-island.informatik.uni-kl.de | Jeu survie | ✅ |
| **SQL Police Department** | sqlpd.com | Crime solving | Freemium |
| **SQL Squid Game** | datalemur.com/sql-squid-game | 9 niveaux | ✅ |
| **Lost at SQL** | lost-at-sql.therobinlord.com | Sous-marin | ✅ |
| **Select Star SQL** | selectstarsql.com | Tutoriel | ✅ |
| **SQLBolt** | sqlbolt.com | 18 leçons | ✅ |
| **SQLZoo** | sqlzoo.net | Wiki | ✅ |
| **DataLemur** | datalemur.com | Interview prep | Freemium |
| **PGExercises** | pgexercises.com | PostgreSQL | ✅ |
| **SQL Practice** | sql-practice.com | Variés | ✅ |
| **Schemaverse** | schemaverse.com | Space game SQL | ✅ |
| **Oracle Dev Gym** | devgym.oracle.com | Certification | ✅ |

## 🌿 Git (6 ressources)

| Outil | URL | Type | Gratuit |
|-------|-----|------|---------|
| **Learn Git Branching** | learngitbranching.js.org | Visualisation | ✅ |
| **Oh My Git!** | ohmygit.org | Card game | ✅ |
| **GitHub Skills** | skills.github.com | Projets guidés | ✅ |
| **Git Exercises** | gitexercises.fracz.com | Exercices CLI | ✅ |
| **Git Immersion** | gitimmersion.com | 50+ labs | ✅ |
| **GitMastery** | gitmastery.me | Gamifié | ✅ |
| **Codecademy Learn Git** | codecademy.com | Cours interactif | Freemium |

## 🔤 Regex (6 ressources)

| Outil | URL | Type | Gratuit |
|-------|-----|------|---------|
| **RegexOne** | regexone.com | 15 leçons | ✅ |
| **RegexLearn** | regexlearn.com | Cours interactif | ✅ |
| **Regex101** | regex101.com | Testing/debug | ✅ |
| **RegExr** | regexr.com | Éditeur | ✅ |
| **Regex Crossword** | regexcrossword.com | Puzzles | ✅ |
| **RexEgg** | rexegg.com | Tutoriel avancé | ✅ |
| **iHateRegex** | ihateregex.io | Patterns prêts | ✅ |

## ⌨️ Vim (6 ressources)

| Outil | URL | Type | Gratuit |
|-------|-----|------|---------|
| **Vim Adventures** | vim-adventures.com | Zelda-like | Freemium |
| **OpenVim** | openvim.com | Tutoriel | ✅ |
| **PacVim** | github.com/jmoon018/PacVim | PacMan CLI | ✅ |
| **Vim Genius** | vimgenius.com | Flashcards | ✅ |
| **VimHero** | vim-hero.com | Challenges | ✅ |
| **vimtutor** | (intégré) | Officiel | ✅ |
| **Vim Golf** | vimgolf.com | Défis optimisation | ✅ |

## 🐧 Linux/Shell (6 ressources)

| Jeu | URL | Description | Gratuit |
|-----|-----|-------------|---------|
| **OverTheWire Bandit** | overthewire.org/wargames/bandit | 34 niveaux | ✅ |
| **OverTheWire Natas** | overthewire.org/wargames/natas | Web security | ✅ |
| **Terminus** | web.mit.edu/mprat/Public/web/Terminus/Web/main.html | Text adventure | ✅ |
| **Bashcrawl** | gitlab.com/slackermedia/bashcrawl | Dungeon bash | ✅ |
| **Command Challenge** | cmdchallenge.com | Défis CLI | ✅ |
| **Linux Survival** | linuxsurvival.com | Tutoriel | ✅ |
| **Linux Journey** | linuxjourney.com | Cours complet | ✅ |
| **SadServers** | sadservers.com | Debug serveurs | ✅ |

## ⌨️ Typing (Code)

| Outil | URL | Spécialité | Gratuit |
|-------|-----|------------|---------|
| **Typing.io** | typing.io | Code réel | ✅ |
| **SpeedCoder** | speedcoder.net | Par langage | ✅ |
| **Keybr** | keybr.com | Adaptatif | ✅ |
| **Monkeytype** | monkeytype.com | Minimaliste | ✅ |
| **10FastFingers** | 10fastfingers.com | Test rapide | ✅ |
| **TypingClub** | typingclub.com | Cours complet | ✅ |
| **TypeLit** | typelit.io | Taper des livres | ✅ |
| **TypeQuicker** | typequicker.com | Code + prose | ✅ |
| **Ratatype** | ratatype.com | Certificat | ✅ |

## 📱 Apps Mobiles

| App | URL | Plateforme | Spécialité | Gratuit |
|-----|-----|------------|------------|---------|
| **SoloLearn** | sololearn.com | iOS/Android | Multi-langages, social | Freemium |
| **Mimo** | getmimo.com | iOS/Android | Projets, bite-sized | Freemium |
| **Enki** | enki.com | iOS/Android | Workouts quotidiens | Freemium |
| **Grasshopper** | grasshopper.app | iOS/Android/Web | JavaScript (Google) | ✅ 100% |

---

# Annexe B : Playgrounds & IDEs en Ligne

## 🖥️ Full-Stack IDEs

| Plateforme | URL | Langages | Gratuit |
|------------|-----|----------|---------|
| **Replit** | replit.com | 50+ | Freemium |
| **CodeSandbox** | codesandbox.io | JS/TS/React | Freemium |
| **StackBlitz** | stackblitz.com | Node.js browser | Freemium |
| **Glitch** | glitch.com | Full-stack | ✅ |
| **CodePen** | codepen.io | HTML/CSS/JS | Freemium |
| **JSFiddle** | jsfiddle.net | HTML/CSS/JS | ✅ |
| **JSBin** | jsbin.com | HTML/CSS/JS, partage | ✅ |
| **PlayCode** | playcode.io | JS/TS/React | Freemium |
| **Codedamn** | codedamn.com/playgrounds | Multi | Freemium |

## 🔬 Playgrounds Spécialisés

| Plateforme | URL | Spécialité |
|------------|-----|------------|
| **Python Tutor** | pythontutor.com | Visualisation mémoire |
| **Compiler Explorer** | godbolt.org | Voir assembleur |
| **TypeScript Playground** | typescriptlang.org/play | TS officiel |
| **Rust Playground** | play.rust-lang.org | Rust officiel |
| **Go Playground** | go.dev/play | Go officiel |
| **Kotlin Playground** | play.kotlinlang.org | Kotlin officiel |
| **Swift Playground** | online.swiftplayground.run | Swift |

## 📊 Visualisation Algorithmes

| Outil | URL | Contenu |
|-------|-----|---------|
| **VisuAlgo** | visualgo.net | 24 modules, quiz |
| **Algorithm Visualizer** | algorithm-visualizer.org | Open-source |
| **Sorting.at** | sorting.at | Comparaison tris |
| **PathFinding.js** | qiao.github.io/PathFinding.js | Pathfinding |
| **Red Blob Games** | redblobgames.com | A*, grids, hexagons |
| **USF Visualization** | cs.usfca.edu/~galles/visualization | Data structures animées |

---

# Annexe C : Chaînes YouTube Recommandées

## 🌍 Anglophones

| Chaîne | Subs | Spécialité | Contenu type |
|--------|------|------------|--------------|
| **freeCodeCamp** | 10M+ | Cours complets | 3-17h |
| **Traversy Media** | 2.3M | Web dev | Projets |
| **Programming with Mosh** | 4.7M | Clean code | Cours structurés |
| **Fireship** | 3M+ | Rapide | "100 seconds" |
| **The Coding Train** | 1.8M | Créatif | p5.js, fun |
| **CS Dojo** | 2M | Algorithmes | Ex-Google |
| **Web Dev Simplified** | 1.5M | JS simplifié | Concepts |
| **Net Ninja** | 1.3M | Frameworks | Séries |
| **Corey Schafer** | 1.2M | Python | Tutoriels |
| **Tech With Tim** | 1.4M | Python | Projets |
| **Computerphile** | 2.4M | CS académique | Concepts |
| **3Blue1Brown** | 6M+ | Maths | Visualisations |
| **Hussein Nasser** | 500k+ | Backend | Architecture |
| **ThePrimeagen** | 500k+ | Performance | Rust, Vim |
| **Low Level Learning** | 400k+ | Bas niveau | C, Assembly |
| **Ben Awad** | 500k+ | Full-stack | React, GraphQL |
| **Jack Herrington** | 200k+ | React avancé | Patterns |

## 🇫🇷 Francophones

| Chaîne | Spécialité |
|--------|------------|
| **Grafikart** | Web dev complet |
| **Le Designer du Web** | HTML/CSS/JS |
| **Cocadmin** | Linux, sysadmin |
| **Pierre Giraud** | Python, SQL, PHP |
| **FormationVidéo** | Multi-langages |
| **Youssef Marzouk** | Algorithmes |
| **Cookie connecté** | Vulgarisation |
| **Underscore_** | Actualités tech |

## 🎬 Vidéos Spécifiques Essentielles

### Debugging (Module B.2)

| Vidéo | Auteur | Durée |
|-------|--------|-------|
| **Debug School (Série complète)** | **Toby Ho** | 6 épisodes |
| Chrome DevTools Full Course | freeCodeCamp | 2h30 |
| VS Code Debugger Tutorial | James Q Quick | 20 min |

### Documentation (Module B.1)

| Vidéo | Auteur | Durée |
|-------|--------|-------|
| How to Read Documentation | Andy Sterkowitz | 15 min |
| How to Use MDN | Kevin Powell | 10 min |
| APIs for Beginners | freeCodeCamp | 3h20 |

### Recherche (Module B.3)

| Vidéo | Auteur | Durée |
|-------|--------|-------|
| **How to Google Like a Pro** | **freeCodeCamp** | **1h** |
| How to Use Stack Overflow | codeSTACKr | 15 min |

### Apprentissage (Module B.4)

| Vidéo | Auteur | Durée |
|-------|--------|-------|
| How I Learn New Technologies | Fireship | 8 min |
| How to Escape Tutorial Hell | Coding in Flow | 12 min |

---

# Annexe D : Récapitulatif des Exercices

## 📊 Liste Complète (22 exercices)

| Code | Nom | Section | Durée |
|------|-----|---------|-------|
| B.1.1 | Cartographie des Documentations | B.1.1 | 15 min |
| B.1.2a | Speed Reading Challenge | B.1.2 | 15 min |
| B.1.2b | Try-Modify-Break | B.1.2 | 20 min |
| B.1.3 | Navigation Multi-Langages | B.1.3 | 15 min |
| B.1.4 | Explorer print/printf/println! | B.1.4 | 30 min |
| B.2.1 | Error Taxonomy | B.2.1 | 15 min |
| B.2.2a | Error Dissector | B.2.2 | 15 min |
| B.2.2b | Stack Trace Challenge | B.2.2 | 10 min |
| B.2.3 | Debug Scientifique | B.2.3 | 30 min |
| B.2.4a | Print Debugging Stratégique | B.2.4 | 15 min |
| B.2.4b | Debugger Challenge | B.2.4 | 25 min |
| B.2.4c | Binary Search Debug | B.2.4 | 20 min |
| B.2.5 | Diagnostic Multi-Langages | B.2.5 | 20 min |
| B.2.6 | Debug Challenge Complet | B.2.6 | 45 min |
| B.3.1a | Pre-Question Checklist | B.3.1 | 15 min |
| B.3.1b | Google-Fu Challenge | B.3.1 | 15 min |
| B.3.2 | Question Makeover | B.3.2 | 20 min |
| B.3.3 | Platform Picker | B.3.3 | 10 min |
| B.3.4 | Reduce This Bug (MCVE) | B.3.4 | 30 min |
| B.4.1 | Information Diet | B.4.1 | 20 min |
| B.4.2 | 30-Day Coding Challenge | B.4.2 | 30 jours |
| B.4.3 | Structure Guide Personnel | B.4.3 | 30 min |
| B.4.4 | **Mini-Projet B** | B.4.4 | 2-3h |

## 📈 Statistiques

| Métrique | Valeur |
|----------|--------|
| Total exercices | 22 |
| Temps estimé (hors 30-day) | ~7h |
| Mini-projet | 1 (2-3h) |
| Challenge long | 1 (30 jours) |

---

# 📊 Statistiques du Catalogue

| Catégorie | Nombre |
|-----------|--------|
| Jeux interactifs | 70+ |
| IDEs/Playgrounds | 28+ |
| Chaînes YouTube | 35+ |
| Vidéos spécifiques | 25+ |
| Exercices POKÉOS | 22 |
| Plateformes d'apprentissage | 20+ |
| Tutoriels interactifs | 12+ |
| Newsletters | 12+ |
| Podcasts | 10+ |
| Communautés | 18+ |
| Outils documentation | 30+ |
| Apps mobiles | 4 |
| Outils typing | 10+ |
| Guides Google-fu | 5+ |
| Guides Stack Overflow | 5+ |
| Ressources Open Source | 8+ |
| Méthodes Second Brain | 4 |
| Competitive Programming | 4 |
| Outils développeur | 10+ |

**Couverture des 96 concepts Module B** : **99%**

**Seule lacune** : Pas de jeu gamifié de "chasse au trésor documentation" (couvert par exercices manuels)

---

*Document généré pour POKÉOS - Janvier 2026*  
*Structure : 18 sous-sections conformes au programme original*  
*Licence : Usage éducatif*
