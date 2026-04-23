# Graph Traversal Cheatsheet

## 1. Reading the Input — What Did They Give You?

First thing you check. The input format tells you what data structure you're working with.

### Adjacency List — Given Directly

Input is a list of lists. Each index = a node. Inner list = that node's neighbors.

```
rooms = [[1], [2], [3], []]
# rooms[0] = [1] means node 0 connects to node 1
# This IS an adjacency list. Use it directly.
# To find neighbors of node k: rooms[k]
```

**Problem hints:** "rooms[i] = list of keys", "graph[i] = list of neighbors"

### Adjacency Matrix — Given Directly

Input is an n × n 2D array. `matrix[i][j] = 1` means node i connects to node j.

```
isConnected = [[1,1,0], [1,1,0], [0,0,1]]
# Rows AND columns represent the SAME thing (cities).
# isConnected[0][1] = 1 means city 0 connects to city 1.
# Diagonal is always 1 (city connects to itself). Ignore it.
# ALWAYS SQUARE — because rows and columns are the same set of nodes.
```

**Problem hints:** "isConnected[i][j]", square matrix, n × n

**To find neighbors of node k:** Loop `for j in range(n)` and check `matrix[k][j] == 1`.

**How is this different from a grid?** Matrix rows and columns represent the SAME set of things (both are cities). Grid rows and columns represent DIFFERENT things (row = latitude, col = longitude). That's why matrix is always square and grid doesn't have to be.

### Grid — Given Directly (Implicit Graph)

2D grid where each cell is a node. Neighbors = up/down/left/right (4 directions).

```
grid = [[1,1,0], [0,1,0], [1,0,1]]
# Each cell (r, c) is a node.
# Neighbors: (r-1,c), (r+1,c), (r,c-1), (r,c+1)
# Rows and columns CAN be different sizes.
# Must check bounds before accessing neighbors.
```

**Problem hints:** "grid of 0s and 1s", "islands", "maze", "matrix" (but not square/symmetric)

### Edge List — YOU Build the Adjacency List

Input is a list of pairs. Each pair = an edge. NOT an adjacency list — you must convert it.

```
connections = [[0,1], [1,3], [2,3], [4,0], [4,5]]
# Each [a, b] = one edge between a and b.
# This is NOT ready to use. You must build an adjacency list from it.
# READ THE PROBLEM to know if edges are directed or undirected.
```

**Problem hints:** "connections", "edges", "prerequisites", "roads"

**Building adjacency list from edge list:**

```
# Undirected — both directions are real
graph = {i: [] for i in range(n)}
for a, b in connections:
    graph[a].append(b)
    graph[b].append(a)

# Directed — only one direction
graph = {i: [] for i in range(n)}
for a, b in connections:
    graph[a].append(b)  # only a → b
```

**When are "both directions real" vs "only one direction real"?**

| Situation | Both directions real? | Example |
| --- | --- | --- |
| Division: a/b = 2 means b/a = 0.5 | YES — both are math facts | LC 399 |
| Friendship: a knows b means b knows a | YES — symmetric relationship |  |
| One-way road: a → b only | NO — reverse is artificial | LC 1466 |
| Prerequisite: must take a before b | NO — reverse doesn't make sense | Course Schedule |

If only one direction is real but you need to traverse both ways, you add the reverse edge BUT tag it to remember it's artificial (see "Directed Graph Tricks" section below).

---

## 2. Adjacency List vs Matrix — When To Use Which

**If the problem gives you one, use it. Don't convert.**

If you're building from an edge list, use adjacency list. Always. Here's why:

| Factor | Adjacency List | Adjacency Matrix |
| --- | --- | --- |
| Default choice | YES | Only if problem gives it |
| Memory | O(V + E) | O(V²) — explodes for large n |
| Max practical n | 10^5 easily | ~500 before memory dies |
| Finding neighbors | Fast — just iterate the list | Must scan entire row of n entries |
| "Is X connected to Y?" | Slower — scan the list | O(1) — just matrix[X][Y] |



---

## 3. BFS — The Precise Mechanical Steps

This is the exact sequence of operations. Not hand-wavy. Exactly what happens.

### Setup

```
from collections import deque

visited = set()        # tracks which nodes we've already fully processed
q = deque([start])     # queue (FIFO) — start must be in here to kick off BFS
```

**Why `start` must be in the queue:** The while loop processes what's in the queue and adds children. No start node = nothing to process = nothing to add = loop never runs.

### The Loop — One Node Per Iteration

```
while q:
    node = q.popleft()                    # STEP 1: pop ONE node from FRONT of queue

    if node not in visited:               # STEP 2: have I been here before?
        visited.add(node)                 #   NO → stamp it visited

        for neighbor in get_neighbors(node):    # STEP 3: scan ALL its neighbors
            if neighbor not in visited:         #   for each neighbor: already visited?
                q.append(neighbor)              #     NO → add to queue for future processing
                                                #     YES → skip, don't add

    # if node WAS already visited → skip everything, do nothing, next iteration
```

### What Exactly Happens Per Iteration — In Plain English

1. **Pop exactly ONE node** from the front of the queue.
2. **Check if it's already in visited.**
   * If YES → skip this node entirely. Go to next iteration. Do nothing.
   * If NO → continue to step 3.
3. **Add it to the visited set.** It's now stamped. Won't be processed again.
4. **Scan every neighbor of this node.** For each neighbor:
   * Is the neighbor already in visited? → skip it, don't add to queue.
   * Is the neighbor NOT in visited? → add it to the queue.
5. **This node is done.** Move to next iteration → pop the next node from the queue.

### CRITICAL: Steps 3 and 4 Are ONE Combined Step

You don't stamp a node visited, leave, and come back later to scan its neighbors. While you're "at" this node, you do BOTH: stamp it AND scan all its neighbors AND add unvisited ones to the queue. All in one iteration.

Why? Because you're already there. Walking away and coming back gives the same result but wastes time. There's no reason to visit a node twice.

### What Is the NEXT Node That Gets Popped?

The next node popped is whatever the queue hands you next. This is NOT necessarily a neighbor of the node you just processed.

**Why?** The queue accumulates neighbors from MANY past iterations. When you pop, you get the OLDEST item in the queue — which might be a neighbor of something you popped 5 iterations ago.

Example:

```
    0
   / \
  1   2
 /
3

BFS from 0:

Iteration 1: Pop 0. Add neighbors 1, 2.     Queue: [1, 2]
Iteration 2: Pop 1. Add neighbor 3.          Queue: [2, 3]
Iteration 3: Pop 2. No unvisited neighbors.  Queue: [3]
             ↑ 2 is a neighbor of 0 (from iteration 1), NOT of 1
Iteration 4: Pop 3. No unvisited neighbors.  Queue: []
```

In iteration 3, we popped 2. It was a neighbor of 0, added back in iteration 1. It was sitting in the queue waiting its turn while we processed 1 first. **BFS pops the oldest waiting node, not the freshest.**

The honest rule: every node in the queue got there because it was a neighbor of SOMETHING popped earlier. But not necessarily the immediately previous one.

### Any Counting/Logic Goes INSIDE the Visited Check

If you need to count flips, track distances, accumulate products — that logic goes inside `if node not in visited` and inside `if neighbor not in visited`. NEVER outside.

```
# WRONG — counts a flip for an already-visited neighbor
for neighbor in graph[node]:
    if (node, neighbor) in og_edges:
        flips += 1              # might count edges to already-visited nodes!
    if neighbor not in visited:
        q.append(neighbor)

# RIGHT — only counts for unvisited neighbors
for neighbor in graph[node]:
    if neighbor not in visited:
        if (node, neighbor) in og_edges:
            flips += 1          # only counts real forward-progress edges
        q.append(neighbor)
```

---

## 4. DFS — Same Steps, Different Container

DFS is the SAME 3 steps as BFS. Only difference: swap queue for stack.

### Iterative DFS (Stack)

```
visited = set()
stack = [start]

while stack:
    node = stack.pop()                    # pop from END (not front) — LIFO

    if node not in visited:
        visited.add(node)

        for neighbor in get_neighbors(node):
            if neighbor not in visited:
                stack.append(neighbor)
```

**`pop()` takes from the END → LIFO → goes deep.**
**`popleft()` takes from the FRONT → FIFO → goes wide.**

That's literally the only code difference between BFS and DFS.

### DFS Behavior vs BFS Behavior

**BFS (queue):** Processes nodes level by level. All distance-1 first, then distance-2, etc. Often pops "siblings" from earlier iterations before diving deeper.

**DFS (stack):** Immediately dives into the freshest neighbor. Goes as deep as possible down one branch, then backtracks to leftover siblings from earlier.

### Recursive DFS

```
def dfs(node, visited):
    if node in visited:
        return
    visited.add(node)

    for neighbor in get_neighbors(node):
        dfs(neighbor, visited)
```

The call stack IS the stack. Cleaner code for tree-shaped problems. Warning: Python recursion limit is 1000. For graphs with 10K+ nodes, use iterative.

### BFS vs DFS — Side by Side

|  | BFS | DFS (iterative) | DFS (recursive) |
| --- | --- | --- | --- |
| Container | `deque` | `list` (stack) | call stack |
| Add | `append` | `append` | recurse |
| Remove | `popleft` | `pop` | return |
| Behavior | Wide — by distance | Deep — one branch at a time | Deep |
| Use for | Shortest path, level order | Components, cycles, topological | Trees, simple components |

### For reachability / counting components

Both BFS and DFS give the same answer. Same visited set at the end. Pick one, be consistent.

---

## 5. Level-Order BFS — The Variant

When the problem cares about LEVELS (level number, rightmost at each level, sum per level), the structure changes slightly.

**Plain BFS** has one while loop — one node per iteration.
**Level-order BFS** has an outer while loop AND an inner for loop.

```
q = deque([root])

while q:                              # OUTER: one iteration = one FULL LEVEL
    level_length = len(q)             # snapshot: how many nodes at THIS level?

    for i in range(level_length):     # INNER: pop exactly that many
        node = q.popleft()
        # ... process node ...
        # ... add children to queue ...
```

**Why `len(q)` works:** When you enter the outer loop, the queue holds ONLY the current level's nodes. You haven't added next level's children yet. So `len(q)` at that moment = exact level size.

**Outer loop:** One iteration = one full level processed.
**Inner loop:** One iteration = one node popped.

Still only one node popped per inner iteration. But you process an entire level's worth of nodes per outer iteration.

**Examples:** LC 199 (right side view — take last node per level), LC 1161 (max level sum — sum nodes per level).

---

## 6. Single Source vs Multi Source

### The Fundamental

**One BFS from node X gives you the path/distance from X to EVERY other node in the graph.** Not just one target. All of them. BFS radiates outward from one starting point.

So:

* **Same source, multiple targets?** One BFS. Start from source, find all targets in one run.
* **Different sources?** Separate BFS per source. Starting from `a` gives you `a`'s paths. Starting from `b` gives completely different paths. Can't combine them — each needs its own visited set and its own starting state.

Example: LC 399 (Evaluate Division) has queries `a/c`, `b/a`, `x/x` — three different starting points → three separate BFS runs. No shortcut.

### Single Source

Start BFS from ONE node. Explore everything reachable from that node.

**Examples:**

* Keys and Rooms — start from room 0, can we reach all rooms?
* Shortest path from A to B
* LC 1466 — BFS from city 0, count wrong-direction roads

**Pattern:**

```
visited = set()
q = deque([start])
while q:
    # ... standard BFS ...
```

### Multi Source (Connected Components)

Loop through ALL nodes. Start a new BFS from every unvisited one. Each new BFS = one more connected component.

**Examples:**

* Number of Provinces — count disjoint groups of cities
* Number of Islands — count disjoint 1-clusters in a grid

**Pattern:**

```
visited = set()
components = 0

for node in all_nodes:
    if node not in visited:
        components += 1                  # found a new component
        q = deque([node])                # fresh BFS for this component
        while q:
            # ... standard BFS, marks everything in this component ...

return components
```

**The counting trick:** Each time the outer for-loop finds an unvisited node and kicks off a new BFS = one more connected component. The inner BFS marks everything in that component as visited so the outer loop skips them.

---

## 7. Weighted BFS — Carrying a Value Along the Path

Standard BFS just tracks "visited or not." Weighted BFS carries a number along the path — distance, product, cost, etc.

**The only change:** Queue stores `(node, running_value)` instead of just `node`. When adding a neighbor, combine the current value with the edge weight.

```
q = deque([(start, initial_value)])  # start with initial value (1.0 for product, 0 for distance)
visited = set()

while q:
    node, value = q.popleft()

    if node not in visited:
        visited.add(node)

        if node == target:
            return value              # found it — value IS the answer

        for neighbor, weight in graph[node]:
            if neighbor not in visited:
                q.append((neighbor, value * weight))    # for products (LC 399)
                # or: q.append((neighbor, value + weight))  # for distances
                # or: q.append((neighbor, value + 1))       # for unweighted shortest path
```

**Examples:**

* LC 399 (Evaluate Division): `value = product`, `combine = multiply`, `initial = 1.0`
* LC 1926 (Nearest Exit from Maze): `value = steps`, `combine = add 1`, `initial = 0`
* Shortest path in unweighted graph: `value = distance`, `combine = add 1`, `initial = 0`

### 7a. Visited vs Container — The One-Line Rule

**Visited always stores node identity only — at most node + direction tag. NEVER weights/costs/steps.** Those live in the container with the node.

**Container choice is driven by what "shortest" means in the problem:**

| Problem asks for... | Container | Notes |
| --- | --- | --- |
| Min edges / min steps (unweighted) | `deque` (BFS) | First pop of any node = fewest edges to it. LC 1926, all grid BFS. |
| Any valid path's value | `deque` (BFS) | Problem accepts any path → first found wins. LC 399. |
| Min-weight path (variable edge costs) | `heapq` (Dijkstra) | Pop in cost order, not insertion order. Not in LC 75. |
| Min-weight path with negative edges | no container — `dist[]` array + relaxation (Bellman-Ford) | Not in LC 75. |

The moment the problem moves from "min edges" to "min weighted cost with variable edge weights," you swap the container from queue to heap. Visited stays the same — node identity only.

**Why this works for BFS:** every edge costs 1, so the first time BFS pops a node it arrived via fewest edges. No later path can beat it → no reason to reprocess → visited = node is sufficient.

**Why it breaks without heap for weighted:** with variable edge weights, a later path can be cheaper even if it has more edges. Queue order (FIFO) doesn't reflect cost order. Heap fixes this by popping cheapest-so-far first.

---

## 8. Directed Graph Tricks

### When Both Directions Are Real — Just Add Both

`a / b = 2.0` means `a → b (weight 2.0)` AND `b → a (weight 0.5)`. Both are true math facts. Nothing artificial. Just add both.

```
graph[a].append((b, w))
graph[b].append((a, 1.0 / w))
```

### When Only One Direction Is Real — Two Approaches

Problem says road goes `a → b` only. But you need to traverse both ways for BFS to explore the full graph. Two ways to handle this:

**Approach 1: Undirected adj list + separate set of original edges**

Build the adjacency list ignoring direction (both ways). Store the original directions separately. During BFS, check the set to know which way was original.

```
# remember original directions
og_edges = set()
for a, b in connections:
    og_edges.add((a, b))

# build UNDIRECTED adjacency list (for traversal only, direction doesn't matter here)
graph = {i: [] for i in range(n)}
for a, b in connections:
    graph[a].append(b)
    graph[b].append(a)

# during BFS, check: am I walking the original direction?
for neighbor in graph[city]:
    if neighbor not in visited:
        visited.add(neighbor)
        # (city, neighbor) in og_edges means original was city → neighbor
        # BFS walks outward from 0, so this road points AWAY from 0
        # that's the wrong direction → count it
        if (city, neighbor) in og_edges:
            wrong_direction += 1
        q.append(neighbor)
```

**Plain English of the check:** "I'm walking from `city` to `neighbor` during BFS from 0. Was the original road also `city → neighbor`? If yes, the road points outward (away from 0) — wrong direction, count it. If no, the original was `neighbor → city` — road points toward 0, correct, don't count."

**Approach 2: Cost-tagged tuples**

Add both directions to the adjacency list, but tag each with a cost: 1 = original direction (wrong), 0 = reverse direction (correct).

```
graph = {i: [] for i in range(n)}
for a, b in connections:
    graph[a].append((b, 1))  # original a → b: if we walk this way, count it
    graph[b].append((a, 0))  # reverse: if we walk this way, don't count
```

Both approaches work. Pick whichever clicks.

---

## 9. Pattern Recognition — Which Problem Type Am I Looking At?

| Keywords in Problem | Likely Pattern |
| --- | --- |
| "Starting from X, can we reach Y/all?" | Single-source BFS/DFS |
| "How many groups/islands/provinces?" | Multi-source BFS/DFS, count components |
| "Shortest path in unweighted graph" | BFS with distance tracking |
| "Shortest path in weighted graph" | Dijkstra (not in LC 75) |
| "Minimum flips/reorders" | Build bidirectional graph, tag costs, traverse |
| "Evaluate division / find ratio" | Weighted BFS, multiply along path |
| "Cycle detection" | DFS with "in current path" tracking |
| "Topological order / prerequisites" | DFS post-order or BFS with in-degree |
| "Level-by-level / rightmost / level sum" | Level-order BFS with inner for loop |
| "Nearest exit / min steps in maze" | BFS on grid, (r,c,steps) tuple, visited = (r,c) only |



---

## 10. Common Traps

1. **Neighbor logic outside the visited check.** Any counting, flipping, cost tracking must be INSIDE `if neighbor not in visited`. Otherwise you count edges to already-processed nodes.
2. **Forgetting to add neighbors to the queue.** You scan neighbors AND add them. If you scan but don't append, BFS never goes deeper than the start node.
3. **Directed adj list when you need to traverse both ways.** If the graph is directed but you need to reach every node from one starting point, you must add reverse edges (with tags) or build undirected + separate direction set.
4. **Adjacency matrix for large n.** n = 50,000 → matrix is 2.5 billion cells. Use adjacency list.
5. **Grid problems: forgetting bounds check.** `(r-1, c)` when r=0 → index -1 → Python silently wraps to LAST row, gives wrong answer without crashing. Bounds-check BEFORE indexing.
6. **Using the same visited set across multiple independent BFS runs.** Each query with a different source needs a FRESH visited set. Old visited set blocks paths that the new query needs.
7. **Level-order BFS: forgetting `len(q)` snapshot.** Must capture `level_length = len(q)` BEFORE the inner for loop. If you check `len(q)` during the loop, it changes as you add children.
8. **Putting the carried value in visited.** `visited.add((node, value))` breaks BFS — same node reached via different paths gets reprocessed, exponential revisits in graphs with cycles. Visited = node ONLY. See §7a.
9. **Reassigning the carried scalar inside the neighbor for-loop.** `value = value * weight` before pushing mutates the prefix that ALL remaining neighbors need. Second neighbor inherits first neighbor's multiplication. Always compute inline: `q.append((neighbor, value * weight))`.
10. **Mutating the grid instead of using a visited set.** Marking cells as walls destroys input. Use `visited` set — cleaner, no side effects.

---

## 11. The 3-Step Summary

**`visited` is a set. `container` is your queue (BFS) or stack (DFS).**

Each iteration processes exactly ONE node:

1. **Pop** exactly one node from the container.
2. **If not already in visited:**
   * Add it to the visited set.
   * Scan ALL its neighbors. For each neighbor:
     + Already in visited? → skip, do NOT add to container.
     + Not in visited? → add to container. Any counting/cost logic goes here too.
   * (If already visited → skip everything. Do nothing. Next iteration.)
3. **Next iteration:** pop the next node from the container. The container decides which node — BFS pops the oldest (front), DFS pops the newest (end). This node might be a neighbor of the previous node or a neighbor of something from many iterations ago.

**Repeat until the container is empty.**

Steps 2a (add to visited) and 2b (scan neighbors + add to container) are ONE combined step. You never leave a node and come back later to scan its neighbors. You do everything while you're there.

Same skeleton for every graph problem. The only things that change per problem:

* How you find neighbors (adj list lookup, matrix row scan, grid 4-directions)
* What you carry in the container (just the node, or node + distance, or node + product)
* What you count (components, flips, products, distances)

**And one invariant no matter what:** visited holds NODES only. The carried value lives in the queue tuple. Never conflate them.
