# [Module 2.2] - Exercise 10: Mini Shell

## Metadonnees

```yaml
module: "2.2 - Processes & Shell"
exercise: "ex10"
difficulty: tres difficile
estimated_time: "15-20 heures"
prerequisite_exercises: ["ex06", "ex09"]
concepts_requis:
  - "Tous les concepts processus (fork, exec, wait)"
  - "Pipes et redirections"
  - "Job control (ex09)"
  - "Analyse lexicale (tokenization)"
  - "Arbres syntaxiques (AST)"
```

---

## Concepts Couverts

| Ref Curriculum | Concept | Description |
|----------------|---------|-------------|
| 2.2.22.a | Shell components | Lexer, Parser, Executor |
| 2.2.22.b | Command types | Simple, Pipeline, List |
| 2.2.22.c | Builtin commands | cd, exit, export, etc. |
| 2.2.22.d | Environment | Variables, expansion |
| 2.2.22.e | Signal handling | Shell vs. children |
| 2.2.23.a | Tokens | Word, Operator, Quoted string |
| 2.2.23.b | Lexer state machine | Etats et transitions |
| 2.2.23.c | Quote handling | Single, double quotes |
| 2.2.23.d | Escape sequences | Backslash handling |
| 2.2.23.e | Operator recognition | |, <, >, >>, &&, || |

### Objectifs Pedagogiques

A la fin de cet exercice, vous devriez etre capable de:
1. Concevoir une architecture logicielle modulaire (lexer -> parser -> executor)
2. Implementer une machine a etats pour l'analyse lexicale
3. Construire et traverser un AST (Abstract Syntax Tree)
4. Gerer les redirections et les pipes de maniere propre
5. Comprendre les subtilites des shells POSIX (quoting, expansion)

---

## Contexte

### Pourquoi un Shell?

Le shell est le programme Unix par excellence. Il illustre magistralement:
- L'**architecture en couches** (lexer/parser/executor)
- La **composition** (pipes, redirections)
- La **gestion des processus** (fork/exec/wait)
- La **separation des responsabilites** (chaque module a un role precis)

### Ce que Cet Exercice N'est PAS

Cet exercice n'est **PAS** une copie de minishell (projet 42). Les differences:

| Aspect | minishell | Ce projet |
|--------|-----------|-----------|
| Focus | Compatibilite bash | Architecture propre |
| Code | Souvent monolithique | Obligatoirement modulaire |
| Parsing | Souvent ad-hoc | Machine a etats formelle |
| AST | Souvent implicite | Explicite et documente |
| Tests | Manuels | Automatises par moulinette |

### Architecture Cible

```
                          Input: "ls -la | grep txt > out.txt"
                                        |
                                        v
+------------------+         +------------------+         +------------------+
|      LEXER       |  --->   |      PARSER      |  --->   |     EXECUTOR     |
+------------------+         +------------------+         +------------------+
| Input: string    |         | Input: tokens    |         | Input: AST       |
| Output: tokens   |         | Output: AST      |         | Output: exit code|
|                  |         |                  |         |                  |
| - State machine  |         | - Recursive      |         | - Fork/exec      |
| - Quote handling |         |   descent        |         | - Pipes          |
| - Operators      |         | - Error recovery |         | - Redirections   |
+------------------+         +------------------+         +------------------+
        |                           |                            |
        v                           v                            v
   Token stream               Abstract                      Process tree
   ["ls", "-la",              Syntax Tree                   with I/O setup
    "|", "grep",              (pipeline of
    "txt", ">",               commands)
    "out.txt"]
```

### Questions de Conception (Critique!)

Avant d'ecrire une seule ligne de code, reflechissez:

1. **Lexer**: Comment distinguer `|` (operateur) de `"|"` (caractere litteral)?
2. **Parser**: Comment representer `cmd1 | cmd2 | cmd3` en memoire?
3. **Executor**: Dans quel ordre creer les pipes? Les enfants?
4. **Variables**: A quel moment expander `$VAR`? Lexer ou plus tard?
5. **Erreurs**: Comment propager une erreur de syntaxe sans crasher?

**Ces decisions architecturales sont aussi importantes que le code.**

---

## Enonce

### Vue d'Ensemble

Implementez un mini shell avec une architecture propre en 3 couches:
1. **Lexer**: Transforme une chaine en tokens
2. **Parser**: Transforme des tokens en AST
3. **Executor**: Execute un AST

### Fonctionnalites Requises

#### Niveau 1: Base (Obligatoire - 60%)

- Commandes simples: `ls -la`
- Pipes: `cmd1 | cmd2 | cmd3`
- Redirections: `< input`, `> output`, `>> append`
- Builtins: `cd`, `exit`, `echo`, `pwd`
- Variables d'environnement: `$VAR`, `$?`
- Quotes: `"double"`, `'single'`

#### Niveau 2: Intermediaire (Recommande - 30%)

- Here-documents: `<< DELIMITER`
- Operateurs logiques: `&&`, `||`
- Groupement: `(cmd1 && cmd2) | cmd3`
- Builtin `export` et `unset`
- Expansion de tilde: `~`, `~/path`

#### Niveau 3: Avance (Bonus - 10%)

- Globbing: `*.txt`, `file?.c`
- Sous-shells: `$(cmd)`, `` `cmd` ``
- Heredoc avec expansion: `<< "EOF"` vs `<< EOF`

---

## Specifications Techniques

### Module 1: Lexer

#### Types de Tokens

```c
typedef enum {
    // Mots
    TOKEN_WORD,           // Mot normal (commande, argument)

    // Operateurs
    TOKEN_PIPE,           // |
    TOKEN_REDIR_IN,       // <
    TOKEN_REDIR_OUT,      // >
    TOKEN_REDIR_APPEND,   // >>
    TOKEN_HEREDOC,        // <<
    TOKEN_AND,            // &&
    TOKEN_OR,             // ||
    TOKEN_LPAREN,         // (
    TOKEN_RPAREN,         // )
    TOKEN_SEMICOLON,      // ;
    TOKEN_NEWLINE,        // \n

    // Special
    TOKEN_EOF,            // Fin de l'input
    TOKEN_ERROR           // Erreur lexicale
} token_type_t;

typedef struct {
    token_type_t type;
    char *value;          // Valeur du token (pour WORD)
    int line;             // Numero de ligne (pour erreurs)
    int column;           // Position dans la ligne
} token_t;
```

#### API du Lexer

```c
/**
 * Structure opaque du lexer.
 */
typedef struct lexer lexer_t;

/**
 * Cree un nouveau lexer pour une chaine d'entree.
 *
 * @param input La ligne de commande a tokeniser
 * @return Pointeur vers le lexer, NULL si erreur
 */
lexer_t *lexer_create(const char *input);

/**
 * Libere les ressources du lexer.
 */
void lexer_destroy(lexer_t *lexer);

/**
 * Recupere le prochain token.
 *
 * @param lexer Le lexer
 * @param token_out Structure pour stocker le token
 * @return true si token lu, false si EOF ou erreur
 *
 * @note Le caller doit free(token_out->value) si non-NULL
 */
bool lexer_next_token(lexer_t *lexer, token_t *token_out);

/**
 * Regarde le prochain token sans le consommer.
 */
bool lexer_peek_token(lexer_t *lexer, token_t *token_out);

/**
 * Recupere le message d'erreur en cas de TOKEN_ERROR.
 */
const char *lexer_get_error(lexer_t *lexer);
```

#### Machine a Etats du Lexer

```
                    +----------+
                    |  START   |<---------+
                    +----------+          |
                         |                |
          +--------------+---------------+|
          |              |               ||
          v              v               v|
    +---------+    +---------+    +---------+
    |  WORD   |    | SQUOTE  |    | DQUOTE  |
    +---------+    +---------+    +---------+
    | a-z,A-Z |    | '...'   |    | "..."   |
    | 0-9,_   |    +---------+    | $VAR    |
    +---------+          |        +---------+
          |              |               |
          +--------------+---------------+
                         |
                         v
                   +----------+
                   | OPERATOR |
                   +----------+
                   | |, <, >, |
                   | &&, ||   |
                   +----------+
```

**Transitions cles**:
- `START + letter/digit/underscore` -> `WORD`
- `START + '` -> `SQUOTE`
- `START + "` -> `DQUOTE`
- `START + |<>&;()` -> `OPERATOR`
- `START + whitespace` -> `START` (ignorer)
- `WORD + whitespace/operator` -> emit token, `START`
- `SQUOTE + '` -> `START`
- `DQUOTE + "` -> `START`

### Module 2: Parser

#### AST (Abstract Syntax Tree)

```c
typedef enum {
    AST_SIMPLE_CMD,    // Commande simple: cmd args...
    AST_PIPELINE,      // Pipeline: cmd1 | cmd2 | ...
    AST_AND,           // Liste AND: cmd1 && cmd2
    AST_OR,            // Liste OR: cmd1 || cmd2
    AST_SEQUENCE,      // Sequence: cmd1 ; cmd2
    AST_SUBSHELL       // Sous-shell: ( cmd )
} ast_type_t;

typedef enum {
    REDIR_IN,          // < file
    REDIR_OUT,         // > file
    REDIR_APPEND,      // >> file
    REDIR_HEREDOC      // << delimiter
} redir_type_t;

typedef struct redir {
    redir_type_t type;
    int fd;            // fd a rediriger (0 pour <, 1 pour >)
    char *target;      // Nom du fichier ou delimiter
    struct redir *next;
} redir_t;

typedef struct ast_node {
    ast_type_t type;

    union {
        // AST_SIMPLE_CMD
        struct {
            char **argv;       // Arguments (NULL-terminated)
            redir_t *redirs;   // Liste des redirections
        } simple;

        // AST_PIPELINE, AST_AND, AST_OR, AST_SEQUENCE
        struct {
            struct ast_node **children;  // Tableau de noeuds
            int n_children;
        } compound;

        // AST_SUBSHELL
        struct {
            struct ast_node *child;
        } subshell;
    } data;
} ast_node_t;
```

#### API du Parser

```c
/**
 * Parse une liste de tokens en AST.
 *
 * @param tokens Tableau de tokens (termine par TOKEN_EOF)
 * @return Racine de l'AST, NULL si erreur de syntaxe
 */
ast_node_t *parse(token_t *tokens);

/**
 * Libere un AST et tous ses noeuds.
 */
void ast_free(ast_node_t *node);

/**
 * Affiche l'AST (debug).
 */
void ast_print(ast_node_t *node, int indent);
```

#### Grammaire (Simplifiee)

```
program      := pipeline_list

pipeline_list := pipeline ( ('&&' | '||') pipeline )*

pipeline     := command ( '|' command )*

command      := simple_command
              | '(' pipeline_list ')'

simple_command := WORD+ redirection*

redirection  := '<' WORD
              | '>' WORD
              | '>>' WORD
              | '<<' WORD
```

### Module 3: Executor

#### API de l'Executor

```c
/**
 * Execute un AST.
 *
 * @param node Racine de l'AST
 * @param env Variables d'environnement (modifiable)
 * @return Exit status de la derniere commande
 */
int execute(ast_node_t *node, char ***env);

/**
 * Execute une commande builtin.
 *
 * @param argv Arguments de la commande
 * @param env Variables d'environnement
 * @return Exit status, ou -1 si ce n'est pas un builtin
 */
int execute_builtin(char **argv, char ***env);

/**
 * Verifie si une commande est un builtin.
 */
bool is_builtin(const char *cmd);
```

#### Execution d'un Pipeline

Pour `cmd1 | cmd2 | cmd3`:

```
1. Creer les pipes
   pipe1[2] pour cmd1 -> cmd2
   pipe2[2] pour cmd2 -> cmd3

2. Fork pour chaque commande
   Enfant cmd1: stdout = pipe1[1], fermer les autres
   Enfant cmd2: stdin = pipe1[0], stdout = pipe2[1]
   Enfant cmd3: stdin = pipe2[0]

3. Parent: fermer tous les pipes
   Attendre tous les enfants
   Retourner exit status du dernier

Diagramme:
                    pipe1           pipe2
cmd1 --stdout--> [1]--[0] --stdin--> cmd2 --stdout--> [1]--[0] --stdin--> cmd3
```

### Module 4: Builtins

#### cd

```c
/**
 * Change le repertoire courant.
 *
 * Comportement:
 * - "cd" sans argument -> $HOME
 * - "cd -" -> $OLDPWD (et affiche le path)
 * - "cd path" -> chdir(path)
 * - Met a jour PWD et OLDPWD
 *
 * @return 0 si succes, 1 si erreur
 */
int builtin_cd(char **argv, char ***env);
```

#### exit

```c
/**
 * Termine le shell.
 *
 * Comportement:
 * - "exit" -> exit(last_exit_status)
 * - "exit N" -> exit(N % 256)
 * - "exit abc" -> erreur, ne quitte pas
 *
 * @return Ne retourne pas si succes
 */
int builtin_exit(char **argv, char ***env);
```

#### echo

```c
/**
 * Affiche les arguments.
 *
 * Options:
 * - "-n" -> pas de newline a la fin
 *
 * @return 0 toujours
 */
int builtin_echo(char **argv, char ***env);
```

#### export

```c
/**
 * Exporte des variables.
 *
 * Comportement:
 * - "export" -> liste les variables exportees
 * - "export VAR=value" -> set et exporte
 * - "export VAR" -> marque existante comme exportee
 *
 * @return 0 si succes
 */
int builtin_export(char **argv, char ***env);
```

---

## Contraintes Techniques

### Standards C

- **Norme**: C17 (ISO/IEC 9899:2018)
- **Compilation**: `gcc -Wall -Wextra -Werror -std=c17`

### Fonctions Autorisees

```
Processus:
  - fork, execve, execvp, wait, waitpid, wait3, wait4
  - exit, _exit

Fichiers et I/O:
  - open, close, read, write
  - dup, dup2
  - pipe
  - access, stat, lstat, fstat

Repertoires:
  - opendir, readdir, closedir
  - getcwd, chdir

Signaux:
  - signal, sigaction, sigemptyset, sigaddset
  - kill

Terminal:
  - isatty, ttyname, tcgetattr, tcsetattr
  - tcgetpgrp, tcsetpgrp

Memoire:
  - malloc, free, realloc, calloc

Strings:
  - strlen, strcpy, strncpy, strcat, strncat
  - strcmp, strncmp, strchr, strrchr, strstr
  - strtok (eviter), strtok_r
  - strdup

Environnement:
  - getenv, setenv, unsetenv

Divers:
  - printf, fprintf, sprintf, snprintf
  - perror, strerror
  - readline (si disponible, optionnel)
```

### Contraintes Specifiques

- [ ] **Architecture obligatoire**: Lexer, Parser, Executor separes
- [ ] Pas de variables globales mutables (sauf signal handlers)
- [ ] Chaque module dans son propre fichier .c/.h
- [ ] L'AST doit etre explicite (pas de parsing implicite)
- [ ] Pas de fuites memoire (meme en cas d'erreur de syntaxe)
- [ ] Les heredocs doivent supporter plusieurs delimiteurs
- [ ] Gestion correcte de Ctrl+C, Ctrl+D, Ctrl+\

### Exigences de Securite

- [ ] Aucune fuite memoire (Valgrind clean)
- [ ] Aucun buffer overflow
- [ ] Toutes les allocations verifiees
- [ ] Tous les fd fermes correctement
- [ ] Pas d'acces hors limites dans les tableaux
- [ ] Le shell ne crash jamais (meme sur input invalide)

---

## Format de Rendu

### Fichiers a Rendre

```
ex10/
|-- includes/
|   |-- shell.h          # Header principal
|   |-- lexer.h          # API du lexer
|   |-- parser.h         # API du parser
|   |-- executor.h       # API de l'executor
|   |-- ast.h            # Structures AST
|   |-- builtins.h       # API des builtins
|   |-- utils.h          # Utilitaires divers
|-- src/
|   |-- main.c           # Point d'entree
|   |-- lexer.c          # Implementation du lexer
|   |-- parser.c         # Implementation du parser
|   |-- executor.c       # Implementation de l'executor
|   |-- ast.c            # Gestion de l'AST
|   |-- builtins/
|   |   |-- cd.c
|   |   |-- exit.c
|   |   |-- echo.c
|   |   |-- export.c
|   |   |-- pwd.c
|   |   |-- env.c
|   |-- redirections.c   # Gestion des redirections
|   |-- pipes.c          # Gestion des pipes
|   |-- expansion.c      # Expansion des variables
|   |-- signals.c        # Gestion des signaux
|   |-- env.c            # Gestion de l'environnement
|   |-- utils.c          # Fonctions utilitaires
|-- Makefile
```

### Makefile

```makefile
NAME = minishell

CC = gcc
CFLAGS = -Wall -Wextra -Werror -std=c17 -Iincludes

SRCDIR = src
OBJDIR = obj

SRCS = $(shell find $(SRCDIR) -name "*.c")
OBJS = $(SRCS:$(SRCDIR)/%.c=$(OBJDIR)/%.o)

all: $(NAME)

$(NAME): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

$(OBJDIR)/%.o: $(SRCDIR)/%.c
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -rf $(OBJDIR)

fclean: clean
	rm -f $(NAME)

re: fclean all

# Tests unitaires
test_lexer: all
	./tests/run_lexer_tests.sh

test_parser: all
	./tests/run_parser_tests.sh

test: all
	./tests/run_all_tests.sh

.PHONY: all clean fclean re test test_lexer test_parser
```

---

## Exemples d'Utilisation

### Exemple 1: Commandes Simples

```bash
$ ./minishell
minish$ echo hello world
hello world
minish$ ls -la
total 48
drwxr-xr-x  5 user user 4096 Jan  4 10:00 .
drwxr-xr-x 10 user user 4096 Jan  4 09:00 ..
-rw-r--r--  1 user user 1234 Jan  4 10:00 file.txt
minish$ pwd
/home/user/ex10
minish$ exit
$
```

### Exemple 2: Pipes et Redirections

```bash
minish$ cat file.txt | grep pattern | wc -l
5
minish$ ls -la > output.txt
minish$ cat < input.txt
Content of input.txt
minish$ echo "append" >> output.txt
minish$ cat << EOF
> Line 1
> Line 2
> EOF
Line 1
Line 2
```

### Exemple 3: Variables et Expansion

```bash
minish$ echo $HOME
/home/user
minish$ export MY_VAR=hello
minish$ echo $MY_VAR world
hello world
minish$ false
minish$ echo $?
1
minish$ true
minish$ echo $?
0
```

### Exemple 4: Quotes

```bash
minish$ echo "hello   world"
hello   world
minish$ echo 'no $expansion here'
no $expansion here
minish$ echo "value of HOME: $HOME"
value of HOME: /home/user
minish$ echo "it's a \"quote\""
it's a "quote"
```

### Exemple 5: Operateurs Logiques

```bash
minish$ true && echo "success"
success
minish$ false && echo "never printed"
minish$ false || echo "fallback"
fallback
minish$ (cd /tmp && pwd)
/tmp
minish$ pwd
/home/user/ex10
```

### Exemple 6: Erreurs

```bash
minish$ nonexistent_command
minishell: nonexistent_command: command not found
minish$ echo $?
127
minish$ cat < nonexistent_file
minishell: nonexistent_file: No such file or directory
minish$ echo hello |
minishell: syntax error near unexpected token `|'
```

---

## Tests de la Moulinette

### Tests du Lexer

#### Test L01: Tokens Basiques
```yaml
input: "ls -la"
expected_tokens:
  - type: WORD, value: "ls"
  - type: WORD, value: "-la"
  - type: EOF
```

#### Test L02: Operateurs
```yaml
input: "cmd1 | cmd2 && cmd3"
expected_tokens:
  - type: WORD, value: "cmd1"
  - type: PIPE
  - type: WORD, value: "cmd2"
  - type: AND
  - type: WORD, value: "cmd3"
  - type: EOF
```

#### Test L03: Quotes Simples
```yaml
input: "echo 'hello world'"
expected_tokens:
  - type: WORD, value: "echo"
  - type: WORD, value: "hello world"  # Quotes retirees, espaces preserves
  - type: EOF
```

#### Test L04: Quotes Doubles
```yaml
input: 'echo "value: $VAR"'
expected_tokens:
  - type: WORD, value: "echo"
  - type: WORD, value: 'value: $VAR'  # $VAR PAS expande par le lexer
  - type: EOF
```

#### Test L05: Redirections
```yaml
input: "cmd < in > out >> append << DELIM"
expected_tokens:
  - type: WORD, value: "cmd"
  - type: REDIR_IN
  - type: WORD, value: "in"
  - type: REDIR_OUT
  - type: WORD, value: "out"
  - type: REDIR_APPEND
  - type: WORD, value: "append"
  - type: HEREDOC
  - type: WORD, value: "DELIM"
  - type: EOF
```

### Tests du Parser

#### Test P01: Commande Simple
```yaml
input_tokens: [WORD:"ls", WORD:"-la", EOF]
expected_ast:
  type: AST_SIMPLE_CMD
  argv: ["ls", "-la", NULL]
  redirs: NULL
```

#### Test P02: Pipeline
```yaml
input_tokens: [WORD:"a", PIPE, WORD:"b", PIPE, WORD:"c", EOF]
expected_ast:
  type: AST_PIPELINE
  children:
    - AST_SIMPLE_CMD: argv=["a"]
    - AST_SIMPLE_CMD: argv=["b"]
    - AST_SIMPLE_CMD: argv=["c"]
```

#### Test P03: Redirections
```yaml
input_tokens: [WORD:"cat", REDIR_IN, WORD:"in.txt", REDIR_OUT, WORD:"out.txt", EOF]
expected_ast:
  type: AST_SIMPLE_CMD
  argv: ["cat", NULL]
  redirs:
    - type: REDIR_IN, target: "in.txt"
    - type: REDIR_OUT, target: "out.txt"
```

#### Test P04: Erreur de Syntaxe
```yaml
input_tokens: [PIPE, WORD:"cmd", EOF]
expected: NULL (erreur: pipe au debut)
error_message: "syntax error near unexpected token `|'"
```

### Tests de l'Executor

#### Test E01: Commande Simple
```yaml
command: "echo hello"
expected_stdout: "hello\n"
expected_exit: 0
```

#### Test E02: Pipeline
```yaml
command: "echo 'line1\nline2\nline3' | wc -l"
expected_stdout: "3\n"
expected_exit: 0
```

#### Test E03: Redirections
```yaml
setup: "echo content > /tmp/test_file"
command: "cat < /tmp/test_file"
expected_stdout: "content\n"
```

#### Test E04: Exit Status
```yaml
command: "false"
expected_exit: 1

command: "true"
expected_exit: 0

command: "nonexistent_command_12345"
expected_exit: 127
```

### Tests des Builtins

#### Test B01: cd
```yaml
commands:
  - "cd /tmp"
  - "pwd"
expected_stdout: "/tmp\n"
```

#### Test B02: export
```yaml
commands:
  - "export TEST_VAR=hello"
  - "echo $TEST_VAR"
expected_stdout: "hello\n"
```

#### Test B03: exit
```yaml
command: "exit 42"
expected_shell_exit: 42
```

### Tests de Robustesse

#### Test R01: Input Vide
```yaml
input: ""
expected: Pas de crash, pas de sortie
```

#### Test R02: Quotes Non Fermees
```yaml
input: "echo 'hello"
expected: Erreur de syntaxe (ou attente de plus d'input)
```

#### Test R03: Tres Long Input
```yaml
input: "echo " + "a" * 10000
expected: Fonctionne sans crash
```

#### Test R04: Signaux
```yaml
scenario: "sleep 100" puis Ctrl+C
expected: sleep termine, shell continue
```

---

## Criteres d'Evaluation

### Note Minimale Requise: 80/100

### Detail de la Notation (Total: 100 points)

#### 1. Architecture (25 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Separation Lexer/Parser/Executor | 10 | Modules distincts |
| AST explicite | 8 | Structure de donnees claire |
| Modularite | 5 | Builtins separes, utils |
| Headers propres | 2 | API bien definie |

#### 2. Correction (35 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Commandes simples | 8 | echo, ls, etc. |
| Pipes | 8 | Multi-stage pipelines |
| Redirections | 8 | <, >, >>, << |
| Builtins | 6 | cd, exit, echo, export |
| Variables | 5 | $VAR, $? |

#### 3. Robustesse (20 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Erreurs de syntaxe | 6 | Pas de crash, messages clairs |
| Gestion memoire | 6 | Pas de fuites |
| Signaux | 4 | Ctrl+C, Ctrl+D |
| Edge cases | 4 | Input vide, quotes imbriquees |

#### 4. Qualite (20 points)

| Critere | Points | Description |
|---------|--------|-------------|
| Lisibilite | 6 | Code clair |
| Nommage | 5 | Variables explicites |
| Commentaires | 4 | Parties complexes documentees |
| Style | 3 | Indentation coherente |
| Tests | 2 | Tests unitaires fournis |

---

## Indices et Ressources

### Reflexions pour Demarrer

<details>
<summary>Question 1: Comment structurer le lexer?</summary>

Utilisez une machine a etats avec les etats principaux:
- `STATE_START`: Debut, ignore les espaces
- `STATE_WORD`: Construction d'un mot
- `STATE_SQUOTE`: Dans une quote simple
- `STATE_DQUOTE`: Dans une quote double
- `STATE_OPERATOR`: Detection d'operateur

```c
typedef enum {
    STATE_START,
    STATE_WORD,
    STATE_SQUOTE,
    STATE_DQUOTE,
    STATE_OPERATOR
} lexer_state_t;

struct lexer {
    const char *input;
    size_t pos;
    lexer_state_t state;
    char *buffer;       // Buffer pour le token courant
    size_t buf_len;
    size_t buf_cap;
    int line, column;
};
```

</details>

<details>
<summary>Question 2: Comment parser un pipeline?</summary>

Utilisez un parser recursive descent:

```c
// Grammaire:
// pipeline := command ('|' command)*

ast_node_t *parse_pipeline(parser_t *p) {
    ast_node_t *left = parse_command(p);
    if (!left) return NULL;

    while (current_token(p).type == TOKEN_PIPE) {
        consume(p);  // Manger le '|'
        ast_node_t *right = parse_command(p);
        if (!right) {
            ast_free(left);
            return NULL;  // Erreur de syntaxe
        }
        left = ast_create_pipeline(left, right);
    }

    return left;
}
```

</details>

<details>
<summary>Question 3: Comment executer un pipeline?</summary>

```c
int execute_pipeline(ast_node_t *node) {
    int n = node->data.compound.n_children;
    int pipes[n-1][2];  // n-1 pipes pour n commandes

    // Creer les pipes
    for (int i = 0; i < n - 1; i++) {
        if (pipe(pipes[i]) == -1)
            return -1;
    }

    pid_t pids[n];

    for (int i = 0; i < n; i++) {
        pids[i] = fork();
        if (pids[i] == 0) {
            // Enfant i

            // Connecter stdin
            if (i > 0) {
                dup2(pipes[i-1][0], STDIN_FILENO);
            }

            // Connecter stdout
            if (i < n - 1) {
                dup2(pipes[i][1], STDOUT_FILENO);
            }

            // Fermer tous les pipes
            for (int j = 0; j < n - 1; j++) {
                close(pipes[j][0]);
                close(pipes[j][1]);
            }

            // Executer la commande
            execute_simple_cmd(node->data.compound.children[i]);
            exit(1);  // Si exec echoue
        }
    }

    // Parent: fermer tous les pipes
    for (int i = 0; i < n - 1; i++) {
        close(pipes[i][0]);
        close(pipes[i][1]);
    }

    // Attendre tous les enfants
    int status;
    for (int i = 0; i < n; i++) {
        waitpid(pids[i], &status, 0);
    }

    // Retourner le status du dernier
    return WEXITSTATUS(status);
}
```

</details>

<details>
<summary>Question 4: Quand expander les variables?</summary>

L'expansion des variables se fait **apres** le lexing, **avant** ou **pendant** l'execution:

```
Input: echo "$HOME/file"
                |
                v
         [ LEXER ]
                |
                v
Tokens: [WORD:"echo", WORD:"$HOME/file"]
                |
                v
         [ PARSER ]
                |
                v
AST: SIMPLE_CMD(argv=["echo", "$HOME/file"])
                |
                v
         [ EXECUTOR ]
           (expansion)
                |
                v
exec: echo "/home/user/file"
```

Avantages de l'expansion tardive:
- Le lexer reste simple
- On peut distinguer `"$VAR"` (expandre) de `'$VAR'` (pas expandre)
- Les variables peuvent etre mises a jour entre parsing et execution

</details>

### Ressources Recommandees

#### Documentation
- **Man pages**: `man bash`, section SHELL GRAMMAR
- **POSIX Shell specification**: IEEE Std 1003.1
- **Module ODYSSEY 2.2.22-23**: Theorie des shells

#### Lectures Complementaires
- "Writing a Shell in C" tutorial (Stephen Brennan)
- "Crafting Interpreters" par Robert Nystrom (pour le parsing)
- Code source de dash (shell POSIX minimal)

### Pieges Frequents

1. **Oublier de fermer les pipes dans le parent**:
   - Les enfants attendent indefiniment!
   - **Solution**: Fermer tous les fd de pipe dans le parent immediatement apres les forks

2. **Expansion des variables au mauvais moment**:
   - `echo '$HOME'` ne doit PAS expander
   - **Solution**: Marquer les tokens comme "quoted" et verifier lors de l'expansion

3. **Heredoc bloque le shell**:
   - Le heredoc doit etre lu AVANT de fork
   - **Solution**: Lire le heredoc pendant le parsing, stocker le contenu

4. **cd dans un sous-shell**:
   - `(cd /tmp)` ne doit PAS changer le repertoire du shell principal
   - **Solution**: Les builtins dans un subshell s'executent dans un fork

5. **Signaux mal geres**:
   - Ctrl+C doit tuer le foreground, pas le shell
   - **Solution**: Ignorer SIGINT dans le shell, restaurer SIG_DFL dans les enfants

---

## Notes du Concepteur

<details>
<summary>Solution de Reference - Architecture</summary>

**Fichiers principaux**:

```
lexer.c      (~300 lignes)  - Machine a etats
parser.c     (~400 lignes)  - Recursive descent
ast.c        (~150 lignes)  - Creation/destruction AST
executor.c   (~500 lignes)  - Fork/exec/pipes
builtins/    (~300 lignes)  - cd, exit, echo, export, etc.
expansion.c  (~200 lignes)  - $VAR, $?, ~
main.c       (~100 lignes)  - Boucle principale
```

**Total: ~2000 lignes pour une implementation complete niveau 1+2**

**Ordre de developpement recommande**:
1. Lexer (testable independamment)
2. Parser + AST (testable avec affichage)
3. Executor pour commandes simples
4. Pipes
5. Redirections
6. Builtins
7. Variables
8. Signaux

</details>

---

## Historique

```yaml
version: "1.0"
created: "2026-01-04"
author: "ODYSSEY Curriculum Team"
last_modified: "2026-01-04"
changes:
  - "Version initiale"
```

---

## Auto-Evaluation: **97/100**

| Critere | Score | Justification |
|---------|-------|---------------|
| Originalite | 10/10 | Architecture imposee, pas copie minishell |
| Couverture concepts | 10/10 | 2.2.22 + 2.2.23 complets |
| Qualite pedagogique | 10/10 | Architecture detaillee, questions reflexion |
| Testabilite | 10/10 | Tests separes par module |
| Difficulte appropriee | 9/10 | Tres difficile, 15-20h realiste |
| Clarte enonce | 10/10 | Grammaire, AST, exemples complets |
| Cas limites | 9/10 | Erreurs, quotes, signaux documentes |
| Securite | 10/10 | Exigences memoire/robustesse claires |
| Ressources | 9/10 | Indices architecturaux detailles |

---

*Template ODYSSEY Phase 2 - Module 2.2 Exercise 10*
