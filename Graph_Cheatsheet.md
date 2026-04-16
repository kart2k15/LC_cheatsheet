# Graph Traversal Cheatsheet

## Core Mental Model

**Graph = nodes + edges.** Node = a "thing" (city, room, person, grid cell). Edge = a relationship between two nodes (connection, road, key).

**Traversal = visit every reachable node starting from somewhere.** Two flavors: BFS (wide) or DFS (deep). For most reachability/counting problems, **both give the same answer.** Pick one and be consistent.

---

## Reading the Input — Figure Out What You Got

This is the first thing you check. The input format tells you what to do.

### 1. Adjacency List Given Directly

Input looks like a dict or list of lists where each entry is the neighbors of that node.

```python
rooms = [[1], [2], [3], []]
# rooms[0] = [1] means room 0 has key to room 1
# this IS an adjacency list. Use directly.
```

- **Problem hint:** "rooms[i] = list of keys", "graph[i] = list of neighbors"
- **What to do:** Use it as-is. `rooms[node]` gives you neighbors.

### 2. Adjacency Matrix Given Directly

Input is an n × n 2D array where `matrix[i][j] = 1` means i and j are connected.

```python
isConnected = [[1,1,0], [1,1,0], [0,0,1]]
# rows AND columns represent the same thing (cities)
# isConnected[i][j] == 1 means city i connects to city j
```

- **Problem hint:** "isConnected[i][j]", "adjacency matrix", square matrix
- **What to do:** Use it as-is. To find neighbors of `city`, loop `for j in range(n)` and check `matrix[city][j] == 1`.
- **Always square:** rows and columns represent the same thing.

### 3. Grid Given Directly (Implicit Graph)

2D grid where each cell is a node and neighbors are up/down/left/right (or 8 directions).

```python
grid = [[1,1,0], [0,1,0], [1,0,1]]
# each cell (r, c) is a node
# neighbors are (r-1, c), (r+1, c), (r, c-1), (r, c+1)
```

- **Problem hint:** "grid of 0s and 1s", "islands", "maze"
- **What to do:** Use `(row, col)` as node coordinates. Check 4 directions for neighbors. Boundary checks needed.
- **Rows and columns can be DIFFERENT sizes** (unlike adjacency matrix).

### 4. Edge List Given — YOU Build the Adjacency List

Input is a list of pairs `[[a, b], ...]`. Each pair is an edge between a and b.

```python
connections = [[0,1], [1,3], [2,3], [4,0], [4,5]]
# NOT an adjacency list. Just a list of edges.
# Edges may be directed or undirected — READ THE PROBLEM.
```

- **Problem hint:** "connections", "edges", "prerequisites", "roads"
- **What to do:** Convert to adjacency list before traversing.

**Building an adjacency list from an edge list:**

```python
from collections import defaultdict

graph = defaultdict(list)
for a, b in connections:
    graph[a].append(b)       # undirected: add both directions
    graph[b].append(a)       # directed: only add one direction
```

---

## Adjacency List vs Matrix — When To Use Which

If the problem gives you one, use it. If you're building from scratch:

| Factor | Adjacency List | Adjacency Matrix |
|---|---|---|
| Default choice | Yes | Only if problem gives it |
| Memory | O(V + E) | O(V²) |
| Works for n up to | 10^5 easily | ~500 before memory explodes |
| Finding neighbors | Fast, just iterate the list | Must scan entire row |
| Check "is X connected to Y?" | Slower, scan the list | O(1), just matrix[X][Y] |

**For LC:** Use adjacency list 95% of the time. Matrix only when the problem hands you one.

---

## BFS vs DFS — When To Use Which

### BFS (queue, FIFO)

- **Use when:** shortest path, level-by-level traversal, "minimum steps to reach X"
- **Data structure:** `deque` with `popleft()`
- **Visits nodes by distance:** all nodes at distance 1 first, then distance 2, then 3

### DFS (stack or recursion)

- **Use when:** cycle detection, topological sort, path finding, connected components
- **Data structure:** stack with `pop()` OR recursion
- **Visits nodes by depth:** goes all the way down one branch before backtracking

### For reachability / counting components problems

Both work equally well. Same answer, same O(V + E) complexity. Pick one, be consistent.

---

## The Universal BFS Skeleton

Every BFS problem has the same shape. Memorize this.

```python
from collections import deque

def bfs(start):
    visited = set()
    q = deque()
    q.append(start)

    while q:
        node = q.popleft()               # POP

        if node not in visited:
            visited.add(node)            # MARK

            for neighbor in get_neighbors(node):  # FIND + ADD
                if neighbor not in visited:
                    q.append(neighbor)
```

**Three steps per iteration: POP → MARK → FIND NEIGHBORS AND ADD.**

**Important:** MARK and FIND NEIGHBORS happen in the SAME step. You don't mark a node, leave, and come back later to find its neighbors. While you're "there" (popped from queue), you do both: stamp it visited AND look at its neighbors. Next iteration pops a neighbor and does the same thing to it.

---

## The Universal DFS Skeleton

Same three operations as BFS, just swap the queue for a stack (or use recursion).

### DFS — Iterative (Stack)

```python
def dfs(start):
    visited = set()
    stack = [start]

    while stack:
        node = stack.pop()               # POP (from end, not front)

        if node not in visited:
            visited.add(node)            # MARK

            for neighbor in get_neighbors(node):  # FIND + ADD
                if neighbor not in visited:
                    stack.append(neighbor)
```

**Only difference from BFS: `stack.pop()` instead of `q.popleft()`.** That's it.
- `pop()` takes from the END → LIFO → goes deep
- `popleft()` takes from the FRONT → FIFO → goes wide

### DFS — Recursive

```python
def dfs(node, visited):
    if node in visited:
        return
    visited.add(node)                    # MARK

    for neighbor in get_neighbors(node): # FIND + RECURSE
        dfs(neighbor, visited)
```

- The call stack IS the stack. No explicit data structure.
- Cleaner code for tree-shaped problems.
- Warning: Python recursion limit is 1000 by default. For huge graphs use iterative.

### BFS vs DFS — Side by Side

| | BFS | DFS (iterative) | DFS (recursive) |
|---|---|---|---|
| Container | `deque` | `list` (stack) | call stack |
| Add | `append` | `append` | recurse |
| Remove | `popleft` | `pop` | return |
| Behavior | Wide (by distance) | Deep (one branch at a time) | Deep |
| Use for | Shortest path, level order | Components, cycles, topological | Trees, simple components |

The only thing that changes per problem is `get_neighbors(node)`:

| Problem Type | `get_neighbors(node)` |
|---|---|
| Adjacency list given | `graph[node]` |
| Adjacency matrix given | `[j for j in range(n) if matrix[node][j] == 1]` |
| Grid | Check 4 directions: up, down, left, right (with bounds) |
| Edge list (after building graph) | `graph[node]` |

---

## Single Source vs Multi Source

### Single Source

Start BFS from ONE node. Explore everything reachable.

**Examples:**
- Keys and Rooms — start from room 0
- Shortest path from A to B

**Pattern:**

```python
visited = set()
q = deque([start])
while q:
    # ... standard BFS ...
```

### Multi Source (Connected Components)

Loop through ALL nodes. Start a new BFS from every unvisited one. Count how many separate BFS "explosions" happened.

**Examples:**
- Number of Provinces — count disjoint groups
- Number of Islands — count disjoint 1-clusters in grid

**Pattern:**

```python
visited = set()
components = 0

for node in all_nodes:
    if node not in visited:
        components += 1                  # new component found
        q = deque([node])
        while q:
            # ... standard BFS, marks everything in this component ...

return components
```

**The trick:** Each new BFS you start = one more connected component. Count the outer-loop entries where you actually kicked off a BFS.

---

## Key Variants

### Reachability — "Can I visit all nodes?"

Run single-source BFS from start. Compare `len(visited)` to total nodes.

```python
return len(visited) == n
```

Example: Keys and Rooms.

### Count Components — "How many separate groups?"

Multi-source BFS pattern. Increment counter each time outer loop finds an unvisited node.

Example: Number of Provinces, Number of Islands.

### Shortest Path (Unweighted)

BFS with level tracking. First time you pop the target, return the distance.

```python
q = deque([(start, 0)])               # (node, distance)
while q:
    node, dist = q.popleft()
    if node == target:
        return dist
    # ... standard add neighbors with dist + 1 ...
```

### Directed Graph with "Flip" Cost (LC 1466 Style)

When edges are directed but you need to explore the whole tree, add BOTH directions to adjacency list and tag each with a cost:

```python
graph = defaultdict(list)
for a, b in connections:
    graph[a].append((b, 1))   # original direction, cost 1
    graph[b].append((a, 0))   # reverse direction, cost 0
```

Then DFS/BFS from root and sum all costs crossed.

---

## Problem-Type Pattern Recognition

| Keywords in Problem | Likely Pattern |
|---|---|
| "Starting from X, can we reach Y?" | Single-source BFS/DFS, check if Y ∈ visited |
| "Starting from X, visit all?" | Single-source BFS, `len(visited) == n` |
| "How many groups/islands/provinces?" | Multi-source BFS/DFS, count outer entries |
| "Shortest path in unweighted graph" | BFS with distance tracking |
| "Shortest path in weighted graph" | Dijkstra (out of scope for LC 75) |
| "Minimum cost to reorient/flip" | Build bidirectional graph, tag costs, traverse |
| "Cycle detection" | DFS with "in current path" tracking |
| "Topological order" | DFS with post-order, or BFS with in-degree |

---

## Common Traps

1. **Adjacency matrix for large n → OOM.** Only use if problem gives it.
2. **Forgetting to check visited before adding to queue** → infinite loop on cyclic graphs.
3. **Grid problems: forgetting bounds check** → index out of range.
4. **Directed vs undirected confusion** → in edge list, undirected means add BOTH `graph[a].append(b)` AND `graph[b].append(a)`. Directed means only one.
5. **Using `set()` when nodes are integers 0 to n-1** → `[False] * n` boolean array is marginally faster but set is cleaner. Either works.
6. **Multi-source pattern: forgetting to reset the inner BFS queue** — each new unvisited node needs its own fresh BFS.

---

## The 3-Step Summary (Memorize This)

**`visited` is a set of nodes you've already stamped. `container` is your queue (BFS) or stack (DFS).**

### Plain BFS / DFS (the default — used 90% of the time)

One while-loop iteration = one node popped and processed.

1. **Pop** exactly one node from the container.
2. **If not already in visited:**
   - Add to visited set.
   - Scan its neighbors. Put unvisited ones into the container.
   - (If already visited → skip entirely.)
3. **Next iteration:** pop one more node. Container decides which — BFS pops oldest, DFS pops newest.

**What's the relationship between the popped node and the previous iteration's node?**

Every node in the container got there because it was a neighbor of something popped earlier. But NOT necessarily the IMMEDIATELY previous one.
- **BFS:** often pops a "sibling" first — a neighbor of something popped several iterations ago — because the container preserves order. You don't dive deep; you sweep across.
- **DFS (iterative stack):** usually pops a freshly-added neighbor of the previous iteration's node (LIFO). But when it hits a dead end, it backtracks to leftover siblings added earlier.

**So the honest rule:** the next popped node is connected to SOME earlier popped node, not always the immediately previous one.

### Level-Order BFS (the variant — "right side view", "max level sum", etc.)

When the problem cares about LEVELS (level number, rightmost at level, sum per level), the structure changes slightly:

```python
while q:
    level_length = len(q)           # how many nodes AT this level
    for i in range(level_length):   # pop exactly that many
        node = q.popleft()
        # ... mark + add neighbors ...
```

- **Outer while loop:** one iteration = one full level processed.
- **Inner for loop:** one iteration = one node popped.
- Still only one node popped per inner iteration. But you process an entire level's worth of nodes per outer iteration.

The `len(q)` trick works because when you enter the outer loop, the container holds exactly the current level's nodes (you haven't added the next level's neighbors yet).

### What Stays the Same

In both plain and level-order variants: **mark + find neighbors + add is always one combined step.** You don't leave a node and come back later to scan its neighbors. You do everything while you're there, then move on.

Same pattern for rooms, provinces, islands, shortest path, level views, cycle detection, everything. BFS vs DFS = which end of the container you pop from. Plain vs level-order = whether you track "how many per level" with an inner loop.
