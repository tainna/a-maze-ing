*This project has been created as part of the 42 curriculum by \<login\>.*

# A-Maze-ing

## Description

A-Maze-ing is a Python maze generator that creates 2D mazes from a configuration file and displays them in the terminal. Mazes can be **perfect** (exactly one path between entry and exit) or **imperfect** (with loops). Every maze contains a hidden **"42"** pattern formed by fully closed cells.

The generation logic is packaged as a standalone, reusable Python library (`mazegen`) that can be installed via `pip`.

---

## Instructions

### Requirements

- Python 3.10+
- pip / venv

### Installation

```bash
python3 -m venv .venv
source .venv/bin/activate
make install
```

### Running

```bash
make run
# or
python3 a_maze_ing.py config.txt
```

### Linting

```bash
make lint          # flake8 + mypy (standard)
make lint-strict   # flake8 + mypy --strict
```

### Building the mazegen package

```bash
make build
# output: dist/mazegen-1.0.0-py3-none-any.whl
```

---

## Configuration file format

One `KEY=VALUE` pair per line. Lines starting with `#` are comments.

| Key | Type | Description | Example |
|---|---|---|---|
| `WIDTH` | int | Number of columns | `WIDTH=20` |
| `HEIGHT` | int | Number of rows | `HEIGHT=15` |
| `ENTRY` | x,y | Entry coordinates | `ENTRY=0,0` |
| `EXIT` | x,y | Exit coordinates | `EXIT=19,14` |
| `OUTPUT_FILE` | str | Output file path | `OUTPUT_FILE=maze.txt` |
| `PERFECT` | bool | Perfect maze? | `PERFECT=True` |
| `SEED` | int (optional) | RNG seed | `SEED=42` |
| `ALGORITHM` | str (optional) | Generation algorithm | `ALGORITHM=prims` |

Valid algorithms: `recursive_backtracker`, `prims`, `kruskals`

---

## Maze generation algorithm

### Primary: Recursive Backtracker (iterative DFS)

The default algorithm. It performs a depth-first search starting from the entry cell, randomly carving passages into unvisited neighbors. When it hits a dead end, it backtracks until a cell with unvisited neighbors is found.

**Why this algorithm?** It produces mazes with long, winding corridors, few dead ends, and a visually interesting texture. It is straightforward to implement iteratively (avoiding Python recursion limits), and naturally produces perfect mazes (spanning trees).

### Bonus: Randomized Prim's

Instead of DFS, it grows the maze from a frontier list, picking edges at random. Produces mazes with more branching and shorter corridors — a different feel from the backtracker.

### Bonus: Randomized Kruskal's

Uses a union-find structure to merge disjoint sets. Shuffles all internal walls and opens them if they connect two different components. Produces highly uniform, unbiased mazes.

---

## Reusable module (mazegen)

The `mazegen` package can be installed independently and used in any Python project.

### Install

```bash
pip install mazegen-1.0.0-py3-none-any.whl
```

### Basic usage

```python
from mazegen.generator import MazeGenerator

# Create and generate a 20x15 maze with a fixed seed
gen = MazeGenerator(width=20, height=15, seed=42)
gen.generate()

# Access the grid: grid[y][x] = wall bitmask (int 0-15)
# Bit 0=North, Bit 1=East, Bit 2=South, Bit 3=West
for row in gen.grid:
    print([hex(cell) for cell in row])

# Access the solution path (list of 'N','E','S','W' strings)
print("Solution:", "".join(gen.solution))
```

### Custom parameters

```python
gen = MazeGenerator(
    width=30,
    height=20,
    entry=(0, 0),
    exit_=(29, 19),
    perfect=True,
    seed=1337,
    algorithm="prims",   # or "kruskals", "recursive_backtracker"
)
gen.generate()
```

### Animation (bonus)

```python
def on_step(grid):
    # called after each wall removal during generation
    pass

gen = MazeGenerator(width=20, height=15, step_callback=on_step)
gen.generate()
```

---

## Resources

- [Maze generation algorithms — Wikipedia](https://en.wikipedia.org/wiki/Maze_generation_algorithm)
- [Buckblog: Maze Generation (Jamis Buck)](http://weblog.jamisbuck.org/2011/2/7/maze-generation-algorithm-recap)
- [Think Labyrinth — Walter D. Pullen](http://www.astrolog.org/labyrnth/algrithm.htm)
- Python docs: [random](https://docs.python.org/3/library/random.html), [collections.deque](https://docs.python.org/3/library/collections.html#collections.deque)
- [flake8](https://flake8.pycqa.org/), [mypy](https://mypy.readthedocs.io/)

### AI usage

Claude (Anthropic) was used to: scaffold the initial project architecture and file skeleton, suggest the multi-algorithm strategy, review docstring and type hint conventions, and help reason through the union-find implementation for Kruskal's. All logic was reviewed, understood, and adapted by the project author before submission.

---

## Team and project management

### Roles

- \<login\>: sole developer — architecture, maze generation, rendering, packaging.

### Planning

- Week 1: Config parser, MazeGenerator skeleton, recursive backtracker.
- Week 2: Output writer, BFS solver, terminal renderer, "42" pattern.
- Week 3: Prim's and Kruskal's bonus, animation, mazegen packaging, README.

### What worked well

- Separating the reusable `mazegen` module from the app-specific I/O from the start avoided painful refactoring later.
- Using a bitmask per cell kept the grid representation compact and made wall operations straightforward.

### What could be improved

- The "42" pattern placement could be more flexible (auto-scaling font).
- The corridor-width constraint (`_enforce_borders` TODO) needs a proper post-processing pass.

### Tools used

- VS Code, Python 3.12, venv
- flake8, mypy, pytest
- Claude (Anthropic) — see AI usage above
