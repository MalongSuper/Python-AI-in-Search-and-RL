# 8-Puzzle Solver using A* Search Algorithm

An implementation of an **8-Puzzle Solver** written in Python using the **A* Search Algorithm**. The solver evaluates path choices using the total cost function $f(n) = g(n) + h(n)$ and provides a step-by-step animated visual breakdown of the solving process.

---

## Overview

The 8-puzzle is a classic sliding tile puzzle played on a $3 \times 3$ grid. It contains 8 numbered tiles (1 through 8) and a single empty space (represented as `0`). The goal is to rearrange the tiles from an initial configuration into a target goal configuration by sliding tiles horizontally or vertically into the blank space.

The state space contains $181,440$ solvable states with an average optimal path length of around 22 moves.

---

## Core Features

* **A* Search Algorithm:** Uses the evaluation function $f(n) = g(n) + h(n)$:


* $g(n)$: Exact path cost from the start state to node $n$.


* $h(n)$: Estimated heuristic cost from node $n$ to the goal.




* **Dual Heuristic Options:**
1. **Misplaced Tiles ($h_1$):** Counts the total number of non-blank tiles not in their goal positions.


2. **Manhattan Distance ($h_2$):** Calculates the sum of horizontal and vertical distances each tile must move to reach its goal position.




* **Lightweight Node Representation:** Represents state nodes as dictionary objects instead of heavy class instances for performance efficiency.


* **Real-time Animated Playback:** Renders live, frame-by-frame updates of the solution sequence with tile-by-tile distance breakdown tables.



---

## Project Structure & Architecture

The program is organized into functional procedural components:

| Section | Core Functions | Description |
| --- | --- | --- |
| **I. Node Representation**<br> | `create_node`, `get_fn`, `get_gn`, `get_hn`, `update_gn`, `update_hn`<br> | Dictionary structure managing state, parent pointers, path cost, and heuristic values.

 |
| **II. Heuristic Evaluation**<br> | `calculate_distance`<br> | Computes Misplaced Tile count or Manhattan Distance matrices.

 |
| **III. State Expansion**<br> | `expand_node`<br> | Generates valid successor states (Up, Down, Left, Right) and prevents redundant state re-exploration.

 |
| **IV. Search Helpers**<br> | `find_least_fn`, `print_puzzle_state`, `handle_goal_reached`<br> | Manages priority node selection, backtrack path reconstruction, and terminal output formatting.

 |
| **V. Solver Logic**<br> | `solve_8_puzzle`<br> | Main A* search loop managing the priority queue fringe and explored state sets.

 |
| **VI. Visualization Engine**<br> | `get_individual_heuristics`, `display_puzzle_frame`, `animate_solution`, `solve_and_animate_8_puzzle`<br> | Live playback visualization engine using Jupyter output manipulation.

 |

---

## Prerequisites & Installation

Ensure you have Python installed along with the following required dependencies:

```bash
pip install numpy ipython

```

---

## Usage

### Standard Solver

Run the standard solver loop to calculate and output the step-by-step move history to the terminal:

```python
from main import solve_8_puzzle

# Goal State (1-8 and 0 for empty tile)
goal_state = [1, 2, 3, 4, 5, 6, 7, 8, 0][cite: 1]

# Initial Board Configuration
initial_state = [8, 6, 4, 2, 1, 3, 5, 7, 0][cite: 1]

# Choose Heuristic: 1 = Misplaced Tiles, 2 = Manhattan Distance
heuristic_choice = 2[cite: 1]

solve_8_puzzle(initial_state, goal_state, heuristic_choice)[cite: 1]

```

### Animated Visual Playback

In Jupyter Notebooks or Google Colab environments, execute `solve_and_animate_8_puzzle` to render an animated playback sequence:

```python
solve_and_animate_8_puzzle(initial_state, goal_state, heuristic_choice=2)[cite: 1]

```

---

## Solvability Rules & Parity

> [!WARNING]
> **Avoid Unsolvable Random States**
> 
> Sliding tile puzzle configurations fall into two distinct mathematical parity groups. Only **50%** of all random tile permutations are solvable.
> 
> 

A given state can reach the standard goal state if and only if its number of **inversions** (pairs of tiles that are out of numerical order) is **even**. Generating unverified random states via methods like `random.sample()` risks introducing unsolvable boards, causing an exhaustive state-space search ($181,440$ nodes) that leads to infinite execution loops.
