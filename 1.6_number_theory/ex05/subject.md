# Exercise 05: Game Theory & Combinatorial Games

## Concepts Covered
- **1.6.10.d-l** Nim, Sprague-Grundy theorem, impartial games
- **1.6.11.d-k** Combinatorial game theory, nimber arithmetic

## Objective

Implement game-theoretic algorithms for solving impartial games.

## Requirements

### Rust Implementation

```rust
pub mod nim {
    /// Basic Nim: XOR of pile sizes
    pub fn nim_winning(piles: &[u32]) -> bool;

    /// Optimal move in Nim (returns pile index and stones to remove)
    pub fn nim_best_move(piles: &[u32]) -> Option<(usize, u32)>;

    /// Nim with restriction: can remove at most k stones
    pub fn nim_restricted(piles: &[u32], k: u32) -> bool;

    /// Nim with move set: can remove any element in allowed set
    pub fn nim_custom_moves(piles: &[u32], moves: &[u32]) -> bool;

    /// Staircase Nim
    pub fn staircase_nim(piles: &[u32]) -> bool;

    /// Lasker's Nim (can split pile into two non-empty piles)
    pub fn laskers_nim(piles: &[u32]) -> bool;
}

pub mod sprague_grundy {
    /// Compute Sprague-Grundy value for a game state
    pub trait Game {
        type State;
        fn moves(&self, state: &Self::State) -> Vec<Self::State>;
        fn is_terminal(&self, state: &Self::State) -> bool;
    }

    /// Compute SG value with memoization
    pub fn sg_value<G: Game>(game: &G, state: &G::State) -> u32
    where
        G::State: std::hash::Hash + Eq + Clone;

    /// Precompute SG values for positions 0..n
    pub fn sg_values(max_n: usize, moves: &[usize]) -> Vec<u32>;

    /// XOR of SG values for independent games
    pub fn combined_sg(sg_values: &[u32]) -> u32;

    /// Is position a winning position?
    pub fn is_winning(sg: u32) -> bool;

    /// Find period in SG sequence (if exists)
    pub fn sg_period(values: &[u32]) -> Option<(usize, usize)>;  // (start, period)
}

pub mod classic_games {
    /// Subtraction game: remove 1..k stones from pile
    pub fn subtraction_game(n: u32, k: u32) -> bool;

    /// Euclid's game: subtract smaller from larger
    pub fn euclids_game(a: u32, b: u32) -> bool;

    /// Wythoff's game: remove from one pile or equal from both
    pub fn wythoffs_game(a: u32, b: u32) -> bool;

    /// Fibonacci Nim: remove 1 to 2*(last removed) stones
    pub fn fibonacci_nim(n: u32) -> bool;

    /// Chomp on m×n chocolate bar
    pub fn chomp(m: usize, n: usize) -> bool;

    /// Nim multiplication (nimber arithmetic)
    pub fn nim_multiply(a: u32, b: u32) -> u32;

    /// Nim addition (XOR)
    pub fn nim_add(a: u32, b: u32) -> u32;
}

pub mod graph_games {
    /// Game on DAG: first player to reach terminal loses
    pub fn dag_game(adj: &[Vec<usize>], start: usize) -> bool;

    /// SG values for all nodes in DAG
    pub fn dag_sg_values(adj: &[Vec<usize>]) -> Vec<u32>;

    /// Poset game
    pub fn poset_game(adj: &[Vec<usize>]) -> bool;

    /// Green hackenbush (on tree)
    pub fn green_hackenbush_tree(adj: &[Vec<usize>], root: usize) -> u32;

    /// Blue-Red hackenbush (surreal numbers)
    pub fn hackenbush_value(game_tree: &GameTree) -> f64;

    pub struct GameTree {
        pub left_moves: Vec<GameTree>,
        pub right_moves: Vec<GameTree>,
    }
}

pub mod partizan {
    /// Surreal number representation
    #[derive(Clone, Debug)]
    pub struct Surreal {
        left: Vec<Box<Surreal>>,
        right: Vec<Box<Surreal>>,
    }

    impl Surreal {
        pub fn zero() -> Self;
        pub fn one() -> Self;
        pub fn neg_one() -> Self;
        pub fn star() -> Self;  // * = {0 | 0}

        pub fn negate(&self) -> Self;
        pub fn add(&self, other: &Self) -> Self;
        pub fn compare(&self, other: &Self) -> std::cmp::Ordering;

        /// Convert to number if it's a number
        pub fn to_number(&self) -> Option<f64>;
    }

    /// Conway's game value
    pub fn game_value(left: &[f64], right: &[f64]) -> f64;

    /// Determine game outcome class
    pub enum Outcome {
        LeftWins,     // G > 0
        RightWins,    // G < 0
        SecondWins,   // G = 0
        FirstWins,    // G || 0 (fuzzy with 0)
    }

    pub fn game_outcome(game: &Surreal) -> Outcome;
}

pub mod applications {
    /// Optimal play for stone game (pick from ends)
    pub fn stone_game(stones: &[i32]) -> i32;

    /// Flip game: can first player win?
    pub fn flip_game(s: &str) -> bool;

    /// Cat and mouse on graph
    pub fn cat_mouse(adj: &[Vec<usize>], mouse: usize, cat: usize, hole: usize) -> i32;

    /// Number game: optimal subtraction
    pub fn number_game(n: u32, moves: &[u32]) -> bool;

    /// Divisor game: subtract proper divisor
    pub fn divisor_game(n: u32) -> bool;

    /// Jump game variations
    pub fn jump_game_win(positions: &[i32]) -> bool;
}
```

### Python Implementation

```python
from typing import List, Optional, Tuple

def nim_winning(piles: List[int]) -> bool: ...
def nim_best_move(piles: List[int]) -> Optional[Tuple[int, int]]: ...
def nim_restricted(piles: List[int], k: int) -> bool: ...

def sg_values(max_n: int, moves: List[int]) -> List[int]: ...
def combined_sg(sg_values: List[int]) -> int: ...

def subtraction_game(n: int, k: int) -> bool: ...
def euclids_game(a: int, b: int) -> bool: ...
def wythoffs_game(a: int, b: int) -> bool: ...
def fibonacci_nim(n: int) -> bool: ...

def dag_game(adj: List[List[int]], start: int) -> bool: ...
def dag_sg_values(adj: List[List[int]]) -> List[int]: ...

def stone_game(stones: List[int]) -> int: ...
def divisor_game(n: int) -> bool: ...
```

## Test Cases

```rust
#[test]
fn test_nim() {
    assert!(!nim_winning(&[1, 2, 3]));  // XOR = 0, losing
    assert!(nim_winning(&[1, 2, 4]));   // XOR != 0, winning
    assert!(!nim_winning(&[3, 3]));
    assert!(nim_winning(&[1, 1, 1]));
}

#[test]
fn test_nim_move() {
    let piles = vec![3, 4, 5];  // XOR = 2
    let (pile, remove) = nim_best_move(&piles).unwrap();
    let mut new_piles = piles.clone();
    new_piles[pile] -= remove;
    // After move, XOR should be 0
    assert_eq!(new_piles.iter().fold(0, |a, &b| a ^ b), 0);
}

#[test]
fn test_nim_restricted() {
    // Can remove 1 to 3 stones
    assert!(nim_restricted(&[7], 3));   // SG = 7 mod 4 = 3
    assert!(!nim_restricted(&[4], 3));  // SG = 0
}

#[test]
fn test_sg_values() {
    // Subtraction game with moves {1, 3, 4}
    let sg = sg_values(10, &[1, 3, 4]);
    // SG sequence: 0,1,2,3,0,1,2,3,0,1,...
    assert_eq!(sg[0], 0);
    assert_eq!(sg[1], 1);
    assert_eq!(sg[4], 0);
}

#[test]
fn test_euclids_game() {
    assert!(euclids_game(5, 3));   // First player wins
    assert!(!euclids_game(6, 3));  // Second player wins
    assert!(euclids_game(10, 7));
}

#[test]
fn test_wythoffs_game() {
    // Losing positions are (⌊kφ⌋, ⌊kφ²⌋) for k = 1,2,3,...
    assert!(!wythoffs_game(1, 2));   // First cold position
    assert!(!wythoffs_game(3, 5));   // Second cold position
    assert!(wythoffs_game(2, 3));    // Winning
    assert!(wythoffs_game(1, 1));    // Winning
}

#[test]
fn test_fibonacci_nim() {
    // Losing positions are Fibonacci numbers
    assert!(!fibonacci_nim(1));
    assert!(!fibonacci_nim(2));
    assert!(!fibonacci_nim(3));
    assert!(fibonacci_nim(4));
    assert!(!fibonacci_nim(5));
    assert!(fibonacci_nim(6));
}

#[test]
fn test_dag_game() {
    // Simple DAG: 0 -> 1 -> 2 (terminal)
    let adj = vec![vec![1], vec![2], vec![]];
    assert!(dag_game(&adj, 0));   // 0 is winning
    assert!(!dag_game(&adj, 1));  // 1 is losing (only move to terminal)
}

#[test]
fn test_dag_sg() {
    let adj = vec![vec![1, 2], vec![3], vec![3], vec![]];
    let sg = dag_sg_values(&adj);
    assert_eq!(sg[3], 0);  // Terminal
    assert_eq!(sg[1], 1);  // mex{0} = 1
    assert_eq!(sg[2], 1);  // mex{0} = 1
    assert_eq!(sg[0], 0);  // mex{1, 1} = mex{1} = 0
}

#[test]
fn test_stone_game() {
    // Alice and Bob pick from ends, Alice first
    assert!(stone_game(&[5, 3, 4, 5]) > 0);  // Alice can win
    assert_eq!(stone_game(&[1, 2]), 1);       // Alice takes 2
}

#[test]
fn test_divisor_game() {
    assert!(!divisor_game(1));   // No moves
    assert!(divisor_game(2));    // Remove 1, opponent gets 1
    assert!(!divisor_game(3));
    assert!(divisor_game(4));
    // Pattern: even numbers win
}

#[test]
fn test_nim_multiply() {
    // Nimber multiplication
    assert_eq!(nim_multiply(0, 5), 0);
    assert_eq!(nim_multiply(1, 5), 5);
    assert_eq!(nim_multiply(2, 2), 3);
    assert_eq!(nim_multiply(2, 3), 1);
}

#[test]
fn test_staircase_nim() {
    // Only odd-indexed piles matter
    assert!(staircase_nim(&[1, 2, 3, 4, 5]));
    assert!(!staircase_nim(&[2, 2]));
}

#[test]
fn test_chomp() {
    // First player wins except for 1×1
    assert!(chomp(2, 3));
    assert!(chomp(3, 3));
    assert!(!chomp(1, 1));
}
```

## Grading

| Criterion | Points |
|-----------|--------|
| Nim and variants | 20 |
| Sprague-Grundy values | 20 |
| Classic games | 20 |
| Graph games | 15 |
| Nimber arithmetic | 10 |
| Applications | 10 |
| Edge cases | 5 |
| **Total** | **100** |
