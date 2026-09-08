# Advanced Maze Generation & Solving

A Python project for generating random mazes and solving them using several classical search algorithms.

The project implements both **maze generation** and **maze solving**, allowing different generation strategies and search strategies to be compared through visualizations and basic performance measurements.

## Overview

The project has two main parts:

1. **Maze Generation**
   - Depth-First Search (DFS)
   - Prim's Algorithm
   - Hunt and Kill
   - Sidewinder
   - Origin Shift

2. **Maze Solving**
   - Breadth-First Search (BFS)
   - Depth-First Search (DFS)
   - Greedy Best-First Search
   - A*

Generated mazes can be animated frame-by-frame, saved as static images, and then solved while displaying both the explored cells and the final path.

## Maze Representation

The maze is represented as a 2D NumPy array.

The logical maze consists of cells separated by walls. For a maze with `n` logical cells per dimension, the underlying array has a size of:

```text
(2n + 1) × (2n + 1)
```

The representation uses:

- `1` for walls
- `-1` for open paths/cells
- `0` for unvisited logical cells during initial maze construction

Logical maze cells are positioned at odd row and column indices:

```text
(1, 1), (1, 3), (1, 5), ...
(3, 1), (3, 3), (3, 5), ...
```

Moving from one logical cell to another therefore normally requires moving two positions in the underlying array. The position between them represents the wall that must be opened.

## Maze Generation

The common entry point for maze generation is:

```python
generate_maze(n, start_point=(1, 1), method='dfs')
```

The function creates the appropriate grid size and selects the requested generation algorithm.

Available methods are:

```text
dfs
prim
hunt_and_kill
sidewinder
origin_shift
```

Each generation algorithm returns a **list of maze frames** rather than only the final maze. This makes it possible to visualize the construction process.

### Depth-First Search

The DFS generator uses a stack and a visited set.

Starting from the selected cell, it:

1. Selects the current cell.
2. Randomizes the four possible directions.
3. Looks two cells away for an unvisited cell.
4. Opens the wall between the cells.
5. Adds the new cell to the stack.
6. Continues until the stack is empty.

This produces the characteristic long, winding passages associated with depth-first maze generation.

```python
dfs_maze_generation(maze, start)
```

### Prim's Algorithm

The Prim generator maintains a collection of candidate walls.

It:

1. Starts from the selected cell.
2. Adds its surrounding walls to the candidate list.
3. Randomly selects a candidate wall.
4. If the cell on the other side has not been visited, the wall is removed.
5. The new cell is added to the visited set.
6. Its surrounding candidate walls are added.

```python
prim_maze_generation(maze, start)
```

### Hunt and Kill

The Hunt and Kill algorithm alternates between two behaviors:

- **Kill:** Continue moving randomly to unvisited neighboring cells.
- **Hunt:** When the current path cannot continue, scan the maze for an unvisited cell adjacent to an already visited cell.

The new cell is connected to the visited region and the random walk resumes.

```python
hunt_and_kill_maze_generation(maze, start)
```

### Sidewinder

The Sidewinder algorithm processes the maze row by row.

It maintains a `current_run` of cells and randomly decides whether to:

- Continue the run horizontally, or
- Close the run by connecting one randomly selected cell upward.

This produces a different visual structure from DFS and Prim, with strong horizontal characteristics.

```python
sidewinder_maze_generation(maze)
```

### Origin Shift

The project also implements the Origin Shift maze generation algorithm, based on the approach attributed in the code to CaptainLuma.

The algorithm first creates an initial connected structure and maintains a `parent` relationship between cells. It then performs a configurable number of random shifts, changing cell connections while maintaining the maze structure.

```python
origin_shift_maze_generation(maze, steps=100)
```

The `generate_maze()` wrapper currently invokes this implementation with `100` steps.

## Maze Visualization

The project stores intermediate maze states in `frames`.

The function:

```python
real_time_visualization(frames, save_dir=None)
```

displays these frames sequentially, creating an animation of the maze-generation process.

Each frame can optionally be saved as an image:

```text
maze_frame_0000.png
maze_frame_0001.png
maze_frame_0002.png
...
```

A final maze can also be saved directly using:

```python
save_static_maze_image(maze_grid, filename)
```

This is used to create final images such as:

```text
maze_dfs.png
maze_prim.png
maze_hunt_kill.png
maze_sidewinder.png
maze_origin_shift.png
```

## Maze Solving

After generating a maze, the project converts it into a `MazeModel`.

```python
maze_model = MazeModel(final_maze_grid, start, goal)
```

The model stores:

- The maze grid
- Maze dimensions
- Start position
- Goal position

It also validates that both the start and goal are open cells.

### Movement Model

The solver operates on the logical maze cells rather than every individual pixel in the grid.

From a position, the possible moves are:

```text
U — Up
D — Down
L — Left
R — Right
```

A move is allowed only when:

1. The destination logical cell is inside the maze.
2. The destination cell is open.
3. The wall between the current cell and destination is open.

This logic is implemented by:

```python
MazeModel.neighbors()
```

## Search Result

Every solving algorithm returns a `SearchResult`.

The result records more than just the path:

```text
algorithm
found
path
visited_order
edges
nodes_expanded
max_frontier
elapsed_ms
completeness
optimality
time_complexity
space_complexity
```

This allows the algorithms to be compared in terms of both their solution and their search behavior.

### Path Reconstruction

The solvers maintain a `parent` dictionary that records where each discovered cell came from.

Once the goal is reached, the path is reconstructed backwards from the goal:

```python
MazeSolver.reconstruct_path(parent, goal)
```

The resulting path is reversed so that it runs from the start to the goal.

## BFS

Breadth-First Search uses a FIFO queue:

```python
deque
```

It explores the maze level by level.

Because every movement has the same cost, BFS finds a shortest path when a solution exists.

Implementation:

```python
bfs_solve(model)
```

Properties recorded by the implementation:

- Complete
- Optimal for uniform step costs
- Time: `O(V + E)`
- Space: `O(V)`

## DFS

Depth-First Search uses a LIFO stack.

It follows one branch deeply before backtracking and exploring another branch.

Implementation:

```python
dfs_solve(model)
```

DFS is complete for the finite graph used here because the implementation keeps a visited set, but it does not guarantee the shortest path.

Properties recorded by the implementation:

- Complete for the finite graph with a visited set
- Not optimal
- Time: `O(V + E)`
- Space: `O(V)`

The neighbor order is reversed before being added to the stack to make the expansion order deterministic for the demonstration.

## Greedy Best-First Search

Greedy Best-First Search chooses the next cell according to its estimated distance to the goal.

The project supports two heuristic functions:

### Manhattan Distance

```python
abs(a.row - b.row) + abs(a.col - b.col)
```

### Euclidean Distance

```python
((a.row - b.row)**2 + (a.col - b.col)**2)**0.5
```

The solver is implemented as:

```python
greedy_best_first_solve(model, heuristic_name)
```

The priority queue always expands the cell with the smallest heuristic value:

```text
h(n)
```

Greedy Best-First Search is generally more goal-directed than uninformed search, but the implementation does not guarantee an optimal path.

Recorded properties:

- Complete for the finite graph with a visited set
- Not optimal
- Time: `O(E log V)`
- Space: `O(V)`

## A*

A* combines the cost already spent with an estimate of the remaining cost.

The priority is:

```text
f(n) = g(n) + h(n)
```

where:

- `g(n)` is the cost from the start to the current cell.
- `h(n)` is the heuristic estimate from the current cell to the goal.

Implementation:

```python
a_star_solve(model, heuristic_name)
```

The project currently demonstrates A* using the Manhattan heuristic.

The implementation maintains `g_cost` values and uses a priority queue to select the cell with the lowest `f(n)`.

With the admissible/consistent heuristic used by the implementation, A* is recorded as optimal.

Properties:

- Complete for finite graphs with positive costs
- Optimal with an admissible/consistent heuristic
- Time: `O(E log V)`
- Space: `O(V)`

## Solved Maze Visualization

The function:

```python
visualize_solved_maze(...)
```

shows three important parts of the search:

- **Visited cells** — cells explored by the algorithm
- **Final path** — the route reconstructed from the parent relationships
- **Start and goal** — the endpoints of the search

The function can also save the resulting visualization to an image.

Example output files include:

```text
solved_maze_bfs.png
solved_maze_dfs.png
solved_maze_greedy_manhattan.png
solved_maze_astar_manhattan.png
```

This makes it possible to visually compare not only the final paths but also how much of the maze each algorithm explored.

## Example Usage

A maze can first be generated with DFS:

```python
n = 15

frames = generate_maze(
    n=n,
    start_point=(1, 1),
    method='dfs'
)

final_maze = frames[-1]
```

The start and goal can then be defined:

```python
start = Position(1, 1)
goal = Position(n * 2 - 1, n * 2 - 1)
```

Create the maze model:

```python
model = MazeModel(
    final_maze,
    start,
    goal
)
```

Then solve it using one of the implemented algorithms:

```python
bfs_result = bfs_solve(model)

dfs_result = dfs_solve(model)

greedy_result = greedy_best_first_solve(
    model,
    heuristic_name="Manhattan"
)

astar_result = a_star_solve(
    model,
    heuristic_name="Manhattan"
)
```

The resulting `SearchResult` objects can be used to inspect the solution:

```python
print(astar_result.path)
print(astar_result.nodes_expanded)
print(astar_result.elapsed_ms)
```

and visualize it:

```python
visualize_solved_maze(
    final_maze,
    astar_result,
    start,
    goal
)
```

## Algorithms at a Glance

| Category | Algorithm | Main Strategy | Optimal Path |
|---|---|---|---|
| Generation | DFS | Randomized depth-first traversal | — |
| Generation | Prim | Randomized frontier/wall expansion | — |
| Generation | Hunt and Kill | Random walk + hunt for new regions | — |
| Generation | Sidewinder | Horizontal runs with upward connections | — |
| Generation | Origin Shift | Randomized connection shifting | — |
| Solving | BFS | FIFO breadth-first exploration | Yes |
| Solving | DFS | LIFO depth-first exploration | No |
| Solving | Greedy Best-First | Lowest heuristic `h(n)` | No |
| Solving | A* | Lowest `g(n) + h(n)` | Yes* |

`*` A* optimality depends on the heuristic satisfying the required conditions.

## Project Structure

The implementation can be conceptually divided into the following components:

```text
Maze Generation
├── dfs_maze_generation()
├── prim_maze_generation()
├── hunt_and_kill_maze_generation()
├── sidewinder_maze_generation()
├── origin_shift_maze_generation()
└── generate_maze()

Visualization
├── real_time_visualization()
└── save_static_maze_image()

Maze Model
├── Position
├── SearchResult
├── MazeModel
└── MazeSolver

Maze Solving
├── bfs_solve()
├── dfs_solve()
├── greedy_best_first_solve()
└── a_star_solve()

Solved Maze Visualization
└── visualize_solved_maze()
```

## Dependencies

The project uses standard Python libraries together with NumPy and Matplotlib.

Main dependencies include:

- Python
- NumPy
- Matplotlib

Python standard-library modules used by the implementation include:

- `random`
- `time`
- `os`
- `collections.deque`
- `heapq`
- `typing`

## Purpose

The project is primarily a practical implementation for experimenting with **maze generation, graph search, heuristic search, and algorithm visualization**.

Rather than implementing only a single maze generator or solver, it provides several alternatives so their behavior can be observed directly.

The generated frames show how different algorithms construct mazes, while the solving visualizations show how BFS, DFS, Greedy Best-First Search, and A* explore the same maze differently.

This makes the project useful for studying the relationship between:

- Randomized maze generation
- Graph representation
- Uninformed search
- Heuristic search
- Path optimality
- Search-space exploration
- Frontier size
- Runtime
- Algorithmic complexity