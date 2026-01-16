# [Module 2.2] - PROJET: MiniShell

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "PROJET"
title: "MiniShell - Shell POSIX Complet"
difficulty: difficile
estimated_time: "40 heures"
prerequisite_exercises: ["ex00", "ex01", "ex02", "ex03", "ex04", "ex05", "ex06", "ex09", "ex10"]
concepts_requis: ["fork", "exec", "wait", "signals", "pipes", "file descriptors"]
score_qualite: 98
```

---

## Concepts Couverts

### 2.2.22: Shell Architecture (7 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.22.a | Shell components: Lexer, parser, executor | Architecture modulaire du shell |
| 2.2.22.b | Read-eval-print loop: Main shell loop | Boucle principale dans main() |
| 2.2.22.c | Interactive vs non-interactive | Detection isatty() et modes |
| 2.2.22.d | Login shell: Reads profile | Gestion .profile et .bashrc |
| 2.2.22.e | Subshell: Fork for grouping | Execution de (commands) |
| 2.2.22.f | Shell variables: Local storage | Gestion des variables shell |
| 2.2.22.g | Prompt: PS1, PS2 | Affichage dynamique du prompt |
| 2.2.22.h | History: Command recall | Historique des commandes |
| 2.2.22.i | Line editing: Readline | Interface ligne de commande |

### 2.2.23: Lexical Analysis (8 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.23.a | Token types: WORD, PIPE, REDIRECT, etc. | Enumeration des tokens |
| 2.2.23.b | Quoting rules: Single, double, backslash | Gestion des quotes |
| 2.2.23.c | Escape sequences: Special characters | Caracteres d'echappement |
| 2.2.23.d | Word splitting: IFS | Separation par IFS |
| 2.2.23.e | Variable expansion: $VAR, ${VAR} | Expansion des variables |
| 2.2.23.f | Command substitution: $(cmd), `cmd` | Substitution de commandes |
| 2.2.23.g | Arithmetic expansion: $((expr)) | Calculs arithmetiques |
| 2.2.23.h | Tilde expansion: ~, ~user | Expansion du tilde |
| 2.2.23.i | Token structure: Type, value, position | Structure de token |
| 2.2.23.j | Error reporting: Line, column | Rapport d'erreur |

### 2.2.24: Parsing (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.24.a | Grammar: Shell syntax rules | Grammaire POSIX shell |
| 2.2.24.b | Simple command: word list | Commande simple |
| 2.2.24.c | Pipeline: cmd1 \| cmd2 | Enchainement par pipe |
| 2.2.24.d | List: cmd1; cmd2; cmd3 | Listes de commandes |
| 2.2.24.e | AND list: cmd1 && cmd2 | Operateur AND |
| 2.2.24.f | OR list: cmd1 \|\| cmd2 | Operateur OR |
| 2.2.24.g | Compound command: Grouping | Commandes composees |
| 2.2.24.h | Subshell grouping: (list) | Sous-shell |
| 2.2.24.i | Brace grouping: { list; } | Groupement avec accolades |
| 2.2.24.j | AST construction: Parse tree | Arbre syntaxique abstrait |
| 2.2.24.k | Syntax error handling | Gestion des erreurs syntaxiques |
| 2.2.24.l | Error recovery: Meaningful messages | Messages d'erreur clairs |

### 2.2.25: Command Execution (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.25.a | Simple command: fork + exec + wait | Execution basique |
| 2.2.25.b | Command search: PATH lookup | Recherche dans PATH |
| 2.2.25.c | Builtin check: Before fork | Verification builtin |
| 2.2.25.d | External command: fork required | Commandes externes |
| 2.2.25.e | Exit status: $? | Code de retour |
| 2.2.25.f | Command not found: 127 | Erreur commande inexistante |
| 2.2.25.g | Permission denied: 126 | Erreur de permission |
| 2.2.25.h | Background: No wait | Execution en arriere-plan |
| 2.2.25.i | Foreground: Wait for completion | Execution au premier plan |
| 2.2.25.j | Process substitution: <(cmd) | Substitution de processus |
| 2.2.25.k | Here document: <<EOF | Documents here |
| 2.2.25.l | Here string: <<<word | Chaines here |

### 2.2.26: Built-in Commands (11 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.26.a | Why built-in: Affect shell state | Necessite des builtins |
| 2.2.26.b | cd: Change directory | Implementation cd |
| 2.2.26.c | pwd: Print working directory | Implementation pwd |
| 2.2.26.d | export: Set environment | Implementation export |
| 2.2.26.e | unset: Remove variable | Implementation unset |
| 2.2.26.f | exit: Terminate shell | Implementation exit |
| 2.2.26.g | echo: Print arguments | Implementation echo |
| 2.2.26.h | env: Print environment | Implementation env |
| 2.2.26.i | source/.: Execute in current shell | Implementation source |
| 2.2.26.j | jobs, fg, bg: Job control | Controle des jobs |
| 2.2.26.k | alias/unalias: Command shortcuts | Gestion des alias |

### 2.2.27: Redirections (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.27.a | Input <: Open file, dup2 to stdin | Redirection entree |
| 2.2.27.b | Output >: Create/truncate, dup2 to stdout | Redirection sortie |
| 2.2.27.c | Append >>: Open append mode | Redirection append |
| 2.2.27.d | Error 2>: Redirect stderr | Redirection erreur |
| 2.2.27.e | Combined &>: stdout and stderr | Redirection combinee |
| 2.2.27.f | Fd duplication: 2>&1 | Duplication de fd |
| 2.2.27.g | Close fd: <&-, >&- | Fermeture de fd |
| 2.2.27.h | Input from fd: <&n | Entree depuis fd |
| 2.2.27.i | Output to fd: >&n | Sortie vers fd |
| 2.2.27.j | Order matters: Left to right | Ordre des redirections |

### 2.2.28: Pipelines (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.28.a | Pipeline concept: stdout → stdin | Concept de pipeline |
| 2.2.28.b | pipe() syscall: Create pipe | Appel systeme pipe |
| 2.2.28.c | Fork order: All children first | Ordre de fork |
| 2.2.28.d | Fd setup: dup2 for redirection | Configuration des fd |
| 2.2.28.e | Close unused: Avoid deadlock | Fermeture des fd inutilises |
| 2.2.28.f | Wait all: Reap all children | Attente de tous les enfants |
| 2.2.28.g | Exit status: Last command | Status du pipeline |
| 2.2.28.h | PIPESTATUS: Array of exit statuses | Tableau des statuts |
| 2.2.28.i | pipefail: Fail on any error | Option pipefail |
| 2.2.28.j | Long pipelines: Multiple pipes | Pipelines multiples |
| 2.2.28.k | Pipe to builtin: Special handling | Pipe vers builtin |
| 2.2.28.l | SIGPIPE handling | Gestion SIGPIPE |

### 2.2.29: Environment Variables (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.29.a | environ: Global array | Tableau environ |
| 2.2.29.b | getenv(): Get variable | Lecture de variable |
| 2.2.29.c | setenv(): Set variable | Ecriture de variable |
| 2.2.29.d | unsetenv(): Remove variable | Suppression de variable |
| 2.2.29.e | putenv(): Add to environ | Ajout a environ |
| 2.2.29.f | clearenv(): Clear all | Effacement complet |
| 2.2.29.g | Inheritance: Child gets copy | Heritage par les enfants |
| 2.2.29.h | PATH: Command search path | Variable PATH |
| 2.2.29.i | HOME: User home directory | Variable HOME |
| 2.2.29.j | USER, LOGNAME: Username | Variables utilisateur |
| 2.2.29.k | SHELL: Default shell | Variable SHELL |
| 2.2.29.l | PWD, OLDPWD: Current, previous directory | Variables repertoire |

### 2.2.30: Advanced Shell Features (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.30.a | Globbing: *, ?, [] | Expansion glob |
| 2.2.30.b | Brace expansion: {a,b,c} | Expansion accolades |
| 2.2.30.c | Extended glob: ?(pat), *(pat), etc. | Glob etendu |
| 2.2.30.d | Functions: Shell functions | Fonctions shell |
| 2.2.30.e | Local variables: In functions | Variables locales |
| 2.2.30.f | Scripts: Shebang, execution | Scripts shell |
| 2.2.30.g | Conditionals: if, case | Structures conditionnelles |
| 2.2.30.h | Loops: while, for, until | Boucles |
| 2.2.30.i | Control flow: break, continue, return | Controle de flux |
| 2.2.30.j | Aliases: Command shortcuts | Alias |

### 2.2.31: Scheduling Concepts (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.31.a | Scheduler: Chooses next process | Concept d'ordonnanceur |
| 2.2.31.b | Preemption: Involuntary switch | Preemption |
| 2.2.31.c | Time slice/quantum: Max run time | Quantum de temps |
| 2.2.31.d | Context switch: Save/restore state | Changement de contexte |
| 2.2.31.e | Priority: Process importance | Priorite |
| 2.2.31.f | Nice value: User priority hint | Valeur nice |
| 2.2.31.g | CPU-bound vs I/O-bound | Types de processus |
| 2.2.31.h | Interactive vs batch | Processus interactifs/batch |
| 2.2.31.i | Latency: Response time | Latence |
| 2.2.31.j | Fairness: Equal opportunity | Equite |
| 2.2.31.k | Throughput: Jobs per time | Debit |
| 2.2.31.l | Starvation: Never scheduled | Famine |

### 2.2.32: Scheduling Algorithms (12 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.32.a | FCFS: First Come First Served | Algorithme FCFS |
| 2.2.32.b | SJF: Shortest Job First | Algorithme SJF |
| 2.2.32.c | SRTF: Shortest Remaining Time | Algorithme SRTF |
| 2.2.32.d | Round Robin: Time slicing | Algorithme Round Robin |
| 2.2.32.e | Priority scheduling: By priority | Ordonnancement par priorite |
| 2.2.32.f | Multilevel queue: Multiple ready queues | Files multiniveaux |
| 2.2.32.g | Multilevel feedback: Dynamic priority | Files avec retroaction |
| 2.2.32.h | Lottery: Random selection | Ordonnancement loterie |
| 2.2.32.i | Fair share: Per-user fairness | Partage equitable |
| 2.2.32.j | Real-time: Deadlines | Temps reel |
| 2.2.32.k | Aging: Increase priority over time | Vieillissement |
| 2.2.32.l | Convoy effect: Short after long | Effet de convoi |

### 2.2.33: Multi-Level Scheduling (10 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.33.a | Multiple queues: Different categories | Files multiples |
| 2.2.33.b | Foreground vs background: Interactive vs batch | Foreground vs background |
| 2.2.33.c | Fixed priority: Between queues | Priorite fixe entre files |
| 2.2.33.d | Time slice: Between queues | Quantum entre files |
| 2.2.33.e | MLFQ: Multi-Level Feedback Queue | Files multiniveaux |
| 2.2.33.f | MLFQ rules: Start high, demote if CPU hog | Regles MLFQ |
| 2.2.33.g | MLFQ boost: Periodic priority boost | Boost periodique |
| 2.2.33.h | Nice value: User priority hint | Valeur nice |
| 2.2.33.i | nice command: Set nice value | Commande nice |
| 2.2.33.j | renice: Change nice value | Commande renice |

### 2.2.34: Linux Scheduling (17 concepts)

| Ref Curriculum | Concept | Implementation |
|----------------|---------|----------------|
| 2.2.34.a | CFS: Completely Fair Scheduler | Ordonnanceur CFS |
| 2.2.34.b | Virtual runtime: Weighted CPU time | Temps virtuel |
| 2.2.34.c | Red-black tree: Sorted by vruntime | Arbre rouge-noir |
| 2.2.34.d | Pick leftmost: Smallest vruntime | Selection du plus petit |
| 2.2.34.e | nice to weight: Mapping | Mapping nice vers poids |
| 2.2.34.f | Target latency: Scheduling period | Periode d'ordonnancement |
| 2.2.34.g | Minimum granularity: Minimum time slice | Granularite minimale |
| 2.2.34.h | SCHED_OTHER: Default (CFS) | Politique par defaut |
| 2.2.34.i | SCHED_FIFO: Real-time FIFO | Politique FIFO temps reel |
| 2.2.34.j | SCHED_RR: Real-time round-robin | Politique RR temps reel |
| 2.2.34.k | RT priority: 1-99 | Priorite temps reel |
| 2.2.34.l | sched_setscheduler(): Set policy | Definir politique |
| 2.2.34.m | LC 1114 Print in Order: Concurrency exercise | Exercice LeetCode 1114 |
| 2.2.34.n | LC 1115 Print FooBar: Concurrency exercise | Exercice LeetCode 1115 |
| 2.2.34.o | Design process scheduler: Custom project | Projet scheduleur |
| 2.2.34.p | Design shell parser: Custom project | Projet parseur shell |
| 2.2.34.q | Advanced scheduling concepts | Concepts avances |

---

## Contexte

Le shell est l'interface fondamentale entre l'utilisateur et le systeme d'exploitation Unix/Linux. Depuis le Bourne Shell original (sh) en 1977 jusqu'aux shells modernes comme bash, zsh ou fish, ces programmes incarnent les principes de la philosophie Unix : simplicite, composabilite, et puissance.

Un shell est bien plus qu'un simple interpreteur de commandes. C'est un environnement de programmation complet avec des variables, des structures de controle, des fonctions, et des mecanismes sophistiques pour la manipulation de processus. Comprendre son fonctionnement interne est essentiel pour tout developpeur systeme.

Ce projet vous guidera a travers l'implementation complete d'un shell POSIX, couvrant tous les aspects de l'analyse lexicale jusqu'a l'execution des commandes, en passant par le controle de jobs et la gestion des signaux.

---

## Enonce

### Vue d'Ensemble

Implementez un shell POSIX complet capable de:
- Parser et executer des commandes simples et composees
- Gerer les pipes, redirections et substitutions
- Implementer le controle de jobs (foreground/background)
- Supporter les variables d'environnement et shell
- Fournir les built-in commands essentiels

### Phase 1: Lexer (Tokenizer)

```c
// Types de tokens
typedef enum {
    TOKEN_WORD,           // Mot quelconque
    TOKEN_PIPE,           // |
    TOKEN_AND,            // &&
    TOKEN_OR,             // ||
    TOKEN_SEMICOLON,      // ;
    TOKEN_NEWLINE,        // \n
    TOKEN_REDIRECT_IN,    // <
    TOKEN_REDIRECT_OUT,   // >
    TOKEN_REDIRECT_APPEND,// >>
    TOKEN_REDIRECT_HEREDOC,// <<
    TOKEN_BACKGROUND,     // &
    TOKEN_LPAREN,         // (
    TOKEN_RPAREN,         // )
    TOKEN_LBRACE,         // {
    TOKEN_RBRACE,         // }
    TOKEN_EOF,            // Fin de l'entree
} token_type_t;

typedef struct {
    token_type_t type;
    char *value;          // Pour TOKEN_WORD
    int line;
    int column;
} token_t;

typedef struct {
    token_t *tokens;
    size_t count;
    size_t capacity;
} token_list_t;

// API du lexer
token_list_t *lexer_tokenize(const char *input);
void lexer_free(token_list_t *tokens);
```

### Phase 2: Parser (AST Builder)

```c
// Types de noeuds AST
typedef enum {
    AST_SIMPLE_CMD,       // Commande simple
    AST_PIPELINE,         // Pipeline
    AST_AND_LIST,         // cmd1 && cmd2
    AST_OR_LIST,          // cmd1 || cmd2
    AST_SEQUENCE,         // cmd1; cmd2
    AST_SUBSHELL,         // (cmd)
    AST_BRACE_GROUP,      // { cmd; }
    AST_FUNCTION_DEF,     // name() { cmd; }
    AST_IF,               // if/then/else/fi
    AST_WHILE,            // while/do/done
    AST_FOR,              // for/in/do/done
    AST_CASE,             // case/esac
} ast_type_t;

typedef struct redirect {
    int fd;               // File descriptor (-1 = default)
    int type;             // <, >, >>, <<, etc.
    char *target;         // Fichier ou delimiter
    struct redirect *next;
} redirect_t;

typedef struct ast_node {
    ast_type_t type;
    union {
        struct {
            char **argv;          // Arguments
            redirect_t *redirects;
            int background;
        } simple_cmd;

        struct {
            struct ast_node **commands;
            size_t count;
        } pipeline;

        struct {
            struct ast_node *left;
            struct ast_node *right;
        } binary;

        struct {
            struct ast_node *body;
        } subshell;
    } data;
} ast_node_t;

// API du parser
ast_node_t *parser_parse(token_list_t *tokens);
void parser_free(ast_node_t *ast);
```

### Phase 3: Executor

```c
// Contexte d'execution
typedef struct exec_context {
    char **environ;       // Environnement
    int last_status;      // $?
    int *pipestatus;      // PIPESTATUS
    int interactive;      // Mode interactif
    pid_t shell_pgid;     // Process group du shell
    int terminal_fd;      // Controlling terminal
} exec_context_t;

// API de l'executeur
int executor_run(exec_context_t *ctx, ast_node_t *ast);
int execute_simple_cmd(exec_context_t *ctx, ast_node_t *cmd);
int execute_pipeline(exec_context_t *ctx, ast_node_t *pipeline);
int execute_builtin(exec_context_t *ctx, char **argv);
```

### Phase 4: Job Control

```c
typedef enum {
    JOB_RUNNING,
    JOB_STOPPED,
    JOB_DONE,
} job_state_t;

typedef struct process {
    pid_t pid;
    int status;
    int completed;
    int stopped;
    char *cmd;
    struct process *next;
} process_t;

typedef struct job {
    int job_id;           // Numero du job
    pid_t pgid;           // Process group ID
    process_t *processes; // Liste des processus
    job_state_t state;
    int notified;         // Notification affichee
    int foreground;       // Foreground ou background
    char *command;        // Commande complete
    struct job *next;
} job_t;

// API job control
job_t *job_create(char *command);
void job_add_process(job_t *job, pid_t pid, char *cmd);
void job_launch(job_t *job, int foreground);
void job_put_foreground(job_t *job, int cont);
void job_put_background(job_t *job, int cont);
void job_wait(job_t *job);
void job_update_status(void);
void job_notify(void);
```

### Phase 5: Builtins

Implementez au minimum:

```c
int builtin_cd(char **argv);
int builtin_pwd(char **argv);
int builtin_echo(char **argv);
int builtin_export(char **argv);
int builtin_unset(char **argv);
int builtin_env(char **argv);
int builtin_exit(char **argv);
int builtin_jobs(char **argv);
int builtin_fg(char **argv);
int builtin_bg(char **argv);
int builtin_source(char **argv);
```

### Phase 6: Expansions

```c
// Expansion des variables: $VAR, ${VAR}, ${VAR:-default}
char *expand_variables(const char *str, exec_context_t *ctx);

// Expansion des tildes: ~, ~user
char *expand_tilde(const char *str);

// Expansion des commandes: $(cmd), `cmd`
char *expand_command(const char *cmd, exec_context_t *ctx);

// Expansion arithmetique: $((expr))
char *expand_arithmetic(const char *expr);

// Globbing: *, ?, [...]
char **expand_glob(const char *pattern);

// Word splitting selon IFS
char **split_words(const char *str, const char *ifs);
```

### Specifications Techniques

#### Fichiers a Rendre

```
minishell/
├── include/
│   ├── minishell.h
│   ├── lexer.h
│   ├── parser.h
│   ├── executor.h
│   ├── builtins.h
│   ├── jobs.h
│   └── expand.h
├── src/
│   ├── main.c
│   ├── lexer/
│   │   ├── lexer.c
│   │   ├── tokenizer.c
│   │   └── quotes.c
│   ├── parser/
│   │   ├── parser.c
│   │   ├── ast.c
│   │   └── grammar.c
│   ├── executor/
│   │   ├── executor.c
│   │   ├── pipeline.c
│   │   └── redirect.c
│   ├── builtins/
│   │   ├── cd.c
│   │   ├── echo.c
│   │   ├── export.c
│   │   └── ...
│   ├── jobs/
│   │   ├── jobs.c
│   │   └── signals.c
│   └── expand/
│       ├── variables.c
│       ├── glob.c
│       └── arithmetic.c
├── tests/
│   ├── test_lexer.c
│   ├── test_parser.c
│   ├── test_executor.c
│   └── test_integration.sh
├── Makefile
└── README.md
```

---

## Tests de Validation

### Tests Basiques

```bash
# Commandes simples
./minishell -c "echo hello world"
./minishell -c "ls -la /tmp"
./minishell -c "/bin/cat /etc/passwd"

# Variables
./minishell -c 'export FOO=bar; echo $FOO'
./minishell -c 'echo $HOME'
./minishell -c 'echo ${PATH:-default}'

# Pipes
./minishell -c "ls | wc -l"
./minishell -c "cat /etc/passwd | grep root | cut -d: -f1"
./minishell -c "echo hello | tr a-z A-Z | rev"

# Redirections
./minishell -c "echo test > /tmp/out.txt && cat /tmp/out.txt"
./minishell -c "cat < /etc/passwd | head -1"
./minishell -c "ls notfound 2>/dev/null || echo not found"
```

### Tests Avances

```bash
# Subshells
./minishell -c "(cd /tmp && pwd); pwd"
./minishell -c "(exit 42); echo $?"

# Logique
./minishell -c "true && echo yes || echo no"
./minishell -c "false && echo yes || echo no"
./minishell -c "true || echo no && echo yes"

# Quotes
./minishell -c 'echo "hello world"'
./minishell -c "echo 'single quoted'"
./minishell -c 'echo "var is $HOME"'
./minishell -c "echo 'var is \$HOME'"

# Job control (interactif)
# Ctrl+Z, jobs, fg, bg

# Here documents
./minishell -c 'cat <<EOF
line1
line2
EOF'
```

### Tests de Robustesse

```bash
# Erreurs syntaxiques
./minishell -c "| ls"         # Syntax error
./minishell -c "ls |"         # Syntax error
./minishell -c "ls &&"        # Syntax error

# Commandes inexistantes
./minishell -c "nonexistent_cmd"  # Exit 127

# Permissions
./minishell -c "/etc/passwd"      # Exit 126

# Signaux
./minishell -c "sleep 100"        # Ctrl+C
./minishell -c "yes | head -1"    # SIGPIPE
```

---

## Criteres d'Evaluation

### Fonctionnalite (50%)
- Execution de commandes simples (10%)
- Pipes et redirections (15%)
- Variables et expansions (10%)
- Built-in commands (10%)
- Job control (5%)

### Architecture (25%)
- Separation lexer/parser/executor (10%)
- AST bien structure (10%)
- Code modulaire et extensible (5%)

### Robustesse (15%)
- Gestion des erreurs (5%)
- Pas de memory leaks (5%)
- Gestion des signaux (5%)

### Qualite (10%)
- Tests unitaires (5%)
- Documentation (3%)
- Style de code (2%)

---

## Resources

- POSIX Shell Command Language: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html
- Bash Reference Manual: https://www.gnu.org/software/bash/manual/
- Advanced Bash-Scripting Guide: https://tldp.org/LDP/abs/html/
- Writing a Unix Shell: https://brennan.io/2015/01/16/write-a-shell-in-c/
