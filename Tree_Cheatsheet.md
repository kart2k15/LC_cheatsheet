# Tree Recursion — Mental Model Cheatsheet

---

## 1. The Universal Skeleton

Every binary tree problem uses this:

```python
def solve(node):
    # 1. BASE CASE — node doesn't exist
    if node is None:
        return something

    # 2. EVAL (optional — WHERE you put this = which traversal)

    # 3. RECURSE LEFT — PAUSES here until left subtree fully finishes
    left_result = solve(node.left)

    # 4. RECURSE RIGHT — PAUSES here until right subtree fully finishes
    right_result = solve(node.right)

    # 5. COMBINE + RETURN — goes BACK UP to whoever called me
    return something
```

**Key insight:** Every recursive call is a PAUSE. The function stops at that line,
waits for the entire subtree to finish, gets the return value, THEN moves to the next line.

---

## 2. The Helper Method Pattern

In LC, the main function signature is fixed. You can't add parameters. So you set up state in the main function and call a helper that does the actual recursion.

```python
class Solution:
    def mainFunction(self, root):
        # SETUP — initialize any state the recursion needs
        self.count = 0              # or self.max_length = 0, etc.
        self.some_map = {}

        # CALL HELPER
        self.dfs_whatever(root, ...)

        # RETURN — result is in self.count (or whatever you set up)
        return self.count

    def dfs_whatever(self, node, ...):
        # actual recursion goes here
```

**When you need a helper:** When the recursion needs extra parameters (like `curr_sum`, `max_so_far`, `target`) or accumulates into an outside variable (`self.count`, `self.max_length`).

**When you DON'T need a helper:** When the main function signature already works as the recursion. Example: `maxDepth(root)` — no extra params needed, answer bubbles up through return values, so the main function IS the recursion.

---

## 3. Base Case Return Values — The Rule

What you return from `if node is None` depends on your traversal pattern:

| Pattern | Base case returns | Why |
|---|---|---|
| Pre-order (bubbling down) | `None` or nothing (`return`) | Work goes into outside variable like `self.count`. Nothing to return upward. |
| Post-order (bubbling up) | Concrete value: `0`, `[]`, `{'key': -1}`, etc. | Parent needs a real value to compute with. `None` would crash `1 + max(None, None)`. |

**The concrete value must make the math work:**
- maxDepth: base returns `0` → leaf gets `1 + max(0, 0) = 1` ✓
- leafSimilar: base returns `[]` → `[] + [4] = [4]` ✓ (`None + [4]` would crash)
- zigzag: base returns `{'went_left': -1, 'went_right': -1}` → leaf gets `-1 + 1 = 0` ✓

**Trap:** Getting the base case value wrong is a common source of off-by-one errors. Always trace a leaf node to verify: `leaf gets base_value + 1 = correct answer for a single node?`

---

## 4. Two Base Cases — When One Isn't Enough

Some problems have two different "stop" conditions:

```python
def get_leaves(node):
    # BASE CASE 1 — node doesn't exist (parent had no child here)
    if node is None:
        return []

    # BASE CASE 2 — node IS a leaf (has no children)
    if node.left is None and node.right is None:
        return [node.val]

    # otherwise keep recursing...
    return get_leaves(node.left) + get_leaves(node.right)
```

**When you need two base cases:** When "node doesn't exist" and "node has no children" require different behavior. Base case 1 is "nothing here." Base case 2 is "something here, but it's a leaf — handle it specially."

**When one base case is enough:** Most problems. `if node is None: return 0` handles both "doesn't exist" and implicitly makes leaves work through the math (`1 + max(0, 0) = 1`).

---

## 5. Bubbling Up vs Bubbling Down

This is the most important concept for deciding your approach.

### Bubbling DOWN (Pre-order)

Parent passes info to children. Info flows ROOT → LEAVES.

```python
def dfs(node, info_from_parent):
    if node is None:
        return
    # USE info_from_parent to do work
    # PASS updated info down to children
    dfs(node.left, updated_info)
    dfs(node.right, updated_info)
```

**What bubbles down:** Current max, current path sum, which direction to go, accumulated product — anything the child needs from its ancestors to do its job.

**Results go into:** Outside variable (`self.count`, `self.max_length`). Nothing returns upward.

**Examples:** Count Good Nodes (carry `max_so_far` down), Path Sum III (carry `curr_sum` down).

### Bubbling UP (Post-order)

Children pass answers to parent. Info flows LEAVES → ROOT.

```python
def dfs(node):
    if node is None:
        return concrete_value
    left = dfs(node.left)
    right = dfs(node.right)
    # COMBINE children's answers
    return something(left, right)
```

**What bubbles up:** Depth, subtree size, list of leaves, dict of zigzag lengths, a found target node — anything the parent needs from its children to compute its own answer.

**Results go into:** The return value itself. No outside variable needed (though some problems use both — return a value AND update `self.max_length`).

**Examples:** maxDepth (return depth), leafSimilar (return list), LCA (return found node or None), zigzag (return dict).

### How to Decide

Ask ONE question: **"Do I need info from my PARENT or from my CHILDREN?"**

- Info from parent → carry it DOWN → pre-order
- Info from children → wait for them, compute from their answers → post-order
- Both (rare) → carry info down AND return info up (Path Sum III does this)

---

## 6. Binary Tree (BT) vs Binary Search Tree (BST)

**Binary Tree (BT):** Any tree where each node has at most 2 children.
No rules about values. Could be anything anywhere.

```
      5
     / \
    9    2
   /
  7
```

**Binary Search Tree (BST):** A binary tree WITH a rule:
left child < parent < right child. Always. At every node.

```
      20
     /  \
    10    30
   / \   / \
  5  15 25  35
```

**Why it matters:**
- BT problems — you must visit every node. No shortcuts.
- BST problems — you can skip entire subtrees because you know where values are.
  Looking for 25? It's > 20 so go right. It's < 30 so go left. Found it. Never touched left subtree.
- In-order traversal on BST = sorted output. Always. This is the main BST trick on LC.

---

## 7. The 4 Traversals

Four ways to visit every node. Three are DFS (go deep). One is BFS (go wide).

```
        *
       / \
      -    3
     / \
    10   5
```

### Pre-order (DFS): ME first, then left, then right

```
Read: *, -, 10, 5, 3
```

"I do my work BEFORE asking my children."

The eval step goes BEFORE recursive calls:

```python
def solve(node):
    if node is None:
        return

    DO_WORK_HERE          # ← pre-order: eval BEFORE children
    solve(node.left)
    solve(node.right)
```

**Use cases:**
- Prefix notation (Polish notation): `* - 10 5 3` — operator before operands
- Passing info DOWNWARD from parent to child (bubbling down)
- Decision tree inference: start at root, evaluate split condition, go left or right
- Copying/serializing a tree (save root first, then children)

**LC pattern:** Problem gives you info at the top that you carry downward.
Example: "Count Good Nodes" — carry max_so_far down, evaluate BEFORE recursing.

---

### In-order (DFS): Left first, then ME, then right

```
Read: 10, -, 5, *, 3
```

"I let my left child go first, then do my work, then let my right child go."

The eval step goes BETWEEN recursive calls:

```python
def solve(node):
    if node is None:
        return

    solve(node.left)
    DO_WORK_HERE          # ← in-order: eval BETWEEN children
    solve(node.right)
```

**Use cases:**
- Infix notation: `(10 - 5) * 3` — operator between operands — how humans write math
- BST → sorted output: in-order on BST ALWAYS gives sorted order
- Kth smallest element in BST — in-order, count until you hit K

**Why BST + in-order = sorted:**
Left child is always smaller → gets read first.
Then me.
Right child is always bigger → gets read last.
Smaller, me, bigger. That's sorted order. At every level.

---

### Post-order (DFS): Left first, then right, then ME last

```
Read: 10, 5, -, 3, *
```

"I let both children finish first, then do my work with their answers."

The eval step goes AFTER recursive calls:

```python
def solve(node):
    if node is None:
        return something

    left_result = solve(node.left)
    right_result = solve(node.right)
    DO_WORK_HERE          # ← post-order: eval AFTER children
    return combine(left_result, right_result)
```

**Use cases:**
- Postfix notation (Reverse Polish): `10 5 - 3 *` — operator after operands.
  This is how stack-based calculators work.
- Computing anything that needs children's answers first (bubbling up)
- Deleting a tree: delete children before parent
- Calculating directory sizes: file sizes bubble up from leaves

**LC pattern:** You NEED your children's answers before you can compute yours.

---

### Level-order (BFS): Go wide, level by level

```
Read: *, -, 3, 10, 5
```

NOT DFS. Uses a queue, not recursion. See Graph Cheatsheet for BFS mechanics.

```python
from collections import deque

def level_order(root):
    if root is None:
        return []

    queue = deque([root])
    result = []

    while queue:
        level_size = len(queue)
        for _ in range(level_size):
            node = queue.popleft()
            result.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

    return result
```

**Use cases:**
- Level-by-level processing (right side view, zigzag, level sums)
- "Nearest" anything in a tree
- Shortest path in tree (fewest edges)

---

## 8. The Decision — Which Traversal Do I Need?

Don't think about traversal names. Ask ONE question:

**"Do I need info from my PARENT or from my CHILDREN?"**

| I need...                           | Direction   | Eval goes...        | Traversal   |
|-------------------------------------|-------------|---------------------|-------------|
| Info from parent to pass DOWN       | Bubble DOWN | BEFORE recurse      | Pre-order   |
| Sorted output from BST             | —           | BETWEEN recurse     | In-order    |
| Children's answers to compute mine  | Bubble UP   | AFTER recurse       | Post-order  |
| Level-by-level processing           | —           | Use BFS, not DFS    | Level-order |

---

## 9. Cleanup / Backtracking — When and Why

Some problems use a shared data structure (like a frequency map) during traversal. When you finish exploring a node's subtrees and are about to go BACK UP, you must UNDO your changes to that shared structure.

### When You Need Cleanup

When a shared map/set should only contain info about ANCESTORS on the current path from root to the current node. Siblings and cousins are NOT ancestors.

### Why Cleanup Exists

Without cleanup, when you go from the left subtree to the right subtree, the left subtree's data is still in the map. Nodes on the right branch would see left branch data and think those nodes are their ancestors — creating fake paths/results that don't exist.

### Where Cleanup Goes

AFTER both children are done. NOT between them, NOT before them.

```python
def dfs(node, ...):
    if node is None:
        return

    # ADD my data to the shared structure
    shared_map[my_data] += 1

    # RECURSE — both children need to see my data in the map
    dfs(node.left, ...)       # I AM an ancestor of my left child ✓
    dfs(node.right, ...)      # I AM an ancestor of my right child ✓

    # CLEANUP — both children done, I'm going back up now
    # Remove my data so nodes on OTHER branches don't see it
    shared_map[my_data] -= 1
```

**Why AFTER both children:** I AM an ancestor of BOTH my children. Both need to see me in the map. If I erase myself between left and right, my right child can't see me. If I erase before recursing, neither child can see me.

**When you DON'T need cleanup:** When there's no shared mutable state. maxDepth passes info through return values — nothing to undo. goodNodes passes `max_so_far` as a parameter — each call gets its own copy, no shared state.

**Rule of thumb:** If you're modifying `self.some_map` or any dict/set that persists across recursive calls, you probably need cleanup. If you're just passing parameters and returning values, you don't.

**Example problem:** Path Sum III — prefix sum map should only contain running totals of ancestors on the current root-to-node path. Cleanup removes current node's running total when backtracking.

---

## 10. Iterative Alternatives to Recursion

Not every tree problem needs recursion. Two important alternatives:

### Parent Pointers (Iterative DFS with Post-Processing)

When recursion is confusing for a particular problem, you can:
1. Walk the tree iteratively (DFS with stack), save every node's parent in a dict.
2. Use the parent dict to trace paths upward.

```python
# Build parent map
stack = [root]
parent = {root: None}
while stack:
    node = stack.pop()
    if node.left:
        parent[node.left] = node
        stack.append(node.left)
    if node.right:
        parent[node.right] = node
        stack.append(node.right)

# Now you can walk from ANY node up to root using parent[node]
```

**When to use:** When you need to trace ancestry (LCA), find paths between arbitrary nodes, or when the recursive solution's early-return logic is too confusing.

**Example:** LCA — trace p's ancestors into a set, walk q upward until you hit someone in that set. First match = LCA (because you're walking from bottom up, first match is the LOWEST common ancestor).

### Level-Order BFS (Queue)

When the problem is about levels, not depth. See Section 7 in the traversals above and the Graph Cheatsheet for full BFS mechanics.

---

## 11. Prefix Sum Analogy

**Prefix sum on array:** Running total left to right. Each position stores sum of everything before it.

**Post-order on tree:** Running total bottom to top. Each node stores result of everything below it.

```
Array prefix sum:    [1, 2, 3, 4] → [1, 3, 6, 10]
                      each element = me + everything before me

Tree post-order:        3          depth: 3
                       / \                / \
                      5    1             2    1
                     /                  /
                    9                  1
                    each node = 1 + max of everything below me
```

Both accumulate values in one direction.
Prefix sum accumulates left → right.
Post-order accumulates bottom → up (leaves → root).

**Prefix sum ON a tree (e.g., Path Sum III):** Same idea as array prefix sum (running total + frequency map) but walking top-to-bottom paths instead of left-to-right. Requires cleanup/backtracking (Section 9) because tree branches diverge — array paths don't.

---

## 12. AI/ML Relevance

**Decision Trees / Random Forest / XGBoost:**
Inference = pre-order traversal. Start at root, evaluate split condition,
go left or right, repeat until leaf = prediction.

**NLP Parse Trees:**
Post-order evaluation. Words (leaves) combine into phrases,
phrases combine into clauses, clauses combine into sentence meaning.
Bottom-up, just like maxDepth.

**Expression Trees (compilers):**
In-order = infix (human readable math).
Pre-order = prefix (LISP programming language).
Post-order = postfix (stack machines, bytecode).

**File Systems:**
Post-order to calculate directory sizes.
Pre-order to list directory structure (ls -R).

---

## 13. Quick Reference — What Changes Per Problem

The skeleton is always the same. Only these things change:

| What | Options | How to decide |
|---|---|---|
| Base case return | `None`, `0`, `[]`, `{}`, `-1` | What does parent need to compute correctly? Trace a leaf to verify. |
| Where eval goes | Before/between/after recursion | Do I need parent's info (pre) or children's answers (post)? |
| What you carry down | `max_so_far`, `curr_sum`, `target`, `direction` | What does the child need from its ancestors? |
| What you return up | `int`, `list`, `dict`, `TreeNode`, `None` | What does the parent need from this subtree? |
| Outside variable | `self.count`, `self.max_length`, `self.result` | Only for pre-order patterns where you accumulate without returning |
| Cleanup needed? | Yes if shared mutable state, No if just params + returns | Am I modifying a map/set that persists across calls? |
| Helper method? | Yes if you need extra params or outside variable | Does LC's function signature have everything I need? |
