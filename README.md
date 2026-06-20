# Lab 1 — Drone Pathfinding with Search Algorithms
**Student:** Manyok Gai  
**Course Topic:** Uninformed & Informed Search | Problem Formulation  

---

## Overview

This lab explores how a drone can navigate a grid map from a start position to a goal using classical AI search algorithms. The work is split into two parts:

- **Part A** — Uninformed (blind) search: BFS, DFS, DLS, IDS  
- **Part B** — Informed (heuristic) search: Greedy Best-First, A\*, Weighted A\*, UCS, IDA\* (bonus)

Both parts use Object-Oriented Python and share the same `Problem` / `Node` / `SearchAlgorithm` design pattern so every algorithm plugs into the same interface without changing the grid or the problem definition.

---

## Repository Contents

| File | Description |
|------|-------------|
| `Manyok_Gai_Lab_1A.ipynb` | Part A notebook — uninformed search algorithms |
| `Manyok_Gai_Lab_1B.ipynb` | Part B notebook — informed search algorithms |
| `Reflection_Question_1A.txt` | Written reflections for Part A |
| `Reflection_Question_1B.txt` | Written reflections for Part B |
| `AI Use Declaration Form.pdf` | AI use declaration for Part A |
| `Lab1BAI Use Declaration Form.pdf` | AI use declaration for Part B |

---

## Part A — Uninformed Search

**Notebook:** `Manyok_Gai_Lab_1A.ipynb`

### Problem Formulation

The drone operates on a 10×10 grid where:
- `0` = free cell (passable)
- `1` = obstacle (blocked)
- **State** — a `(row, col)` tuple representing the drone's position
- **Actions** — UP, DOWN, LEFT, RIGHT (only into free, in-bounds cells)
- **Result** — the new `(row, col)` after applying an action
- **Action cost** — 1 per move (uniform cost grid)

### Algorithms Implemented

| Algorithm | Strategy | Complete | Optimal | Memory |
|-----------|----------|----------|---------|--------|
| **BFS** | FIFO queue — level by level | Yes | Yes (unit cost) | O(b^d) |
| **DFS** | LIFO stack — deepest first | Yes (with cycle check) | No | O(b·d) |
| **DLS** | DFS with a fixed depth limit | Only if goal ≤ limit | No | O(b·limit) |
| **IDS** | Repeating DLS with increasing limits | Yes | Yes (unit cost) | O(b·d) |

### Key Design Choices

- A shared abstract `SearchAlgorithm` class provides the `expand()` method used by all algorithms — keeps the logic clean and reusable.
- BFS and DFS use a global `reached` set to avoid revisiting cells.
- DLS uses path-cycle checking instead of a global set, so the same cell can be reached by different routes.
- IDS calls DLS repeatedly with `limit = 0, 1, 2, ...` until a solution is found, combining BFS's completeness with DFS's low memory usage.

---

## Part B — Informed Search

**Notebook:** `Manyok_Gai_Lab_1B.ipynb`

### What Changes from Part A

Part B introduces **terrain costs** — cells have weights greater than 1 (e.g., high-turbulence zones), so the cheapest path is no longer just the shortest one in number of steps. This makes heuristics essential for efficient search.

### Heuristic Functions

Two admissible heuristics are used to estimate remaining cost to the goal:

- **Manhattan distance** — `|row_goal - row| + |col_goal - col|`  
  Works on the relaxed problem where obstacles don't exist and every move costs 1. Dominates Euclidean, so it prunes more nodes.

- **Euclidean distance** — straight-line distance to the goal.  
  Also admissible when terrain costs ≥ 1, but less informed than Manhattan on grid problems.

### Algorithms Implemented

| Algorithm | Uses g(n) | Uses h(n) | Optimal | Notes |
|-----------|-----------|-----------|---------|-------|
| **Greedy Best-First** | No | Yes | No | Fast but ignores accumulated cost; can find expensive paths |
| **A\*** | Yes | Yes | Yes (admissible h) | Balances cost so far and estimated cost to go |
| **Weighted A\*** | Yes | Yes × W | Bounded (≤ W × optimal) | Faster than A\* at the cost of a controllable quality bound |
| **UCS** | Yes | No | Yes | Like BFS but with a priority queue; expands cheapest node first |
| **IDA\*** *(bonus)* | Yes | Yes | Yes | A\* quality with O(d) memory; re-expands nodes across f-cost iterations |

### Why the `reached` Structure Changed

In Part A, `reached` was a set — BFS always arrives at a state optimally first on unit-cost grids.  
In Part B, `reached` is a **dictionary** mapping each state to the best path cost seen so far. With non-uniform terrain, the first path to a state may not be the cheapest, so the algorithm must allow re-discovery when a cheaper route is found.

### Key Observations

- Greedy expanded far fewer nodes than A\* on the turbulence map but produced a path **3× more expensive** — fewer expansions does not mean a better result.
- A\* with Manhattan expanded ~18 nodes vs ~52 for UCS on the same map, showing how much a good heuristic prunes the search.
- Weighted A\* (W = 2) expanded roughly half as many nodes as standard A\* while still finding the optimal path on the test maps — a useful trade-off when compute time matters.

---

## How to Run

1. Open either notebook in **Jupyter Notebook** or **JupyterLab**.
2. Run all cells from top to bottom (`Kernel → Restart & Run All`).
3. Each section is self-contained with inline visualisations showing the grid, obstacles, and the path found by each algorithm.

### Dependencies

```
python >= 3.9
numpy
pandas
matplotlib
jupyter
```

Install with:

```bash
pip install numpy pandas matplotlib jupyter
```

---

## Reflection Highlights

**Part A**
- BFS guarantees the shortest path on unit-cost grids because it explores nodes level by level and returns the first solution found.
- IDS combines the completeness and optimality of BFS with the low memory footprint of DFS — the cost of re-expanding shallow nodes is small relative to the savings.

**Part B**
- An admissible heuristic never overestimates the true cost, which is what guarantees A\*'s optimality. Manhattan stays admissible as long as terrain costs are ≥ 1.
- For real drone navigation, useful extensions would include real-time wind data, dynamic no-fly zones, battery depletion rates per terrain type, and replanning strategies for moving targets.

---

*Submitted as part of the AI course lab series.*
