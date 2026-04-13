# Tree Recursion — Mental Model Cheatsheet

---

## The Universal Skeleton

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

## Binary Tree (BT) vs Binary Search Tree (BST)

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

## The 4 Traversals

Four ways to visit every node in a tree. Three are DFS (go deep). One is BFS (go wide).

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
- Passing info DOWNWARD from parent to child
- Decision tree inference: start at root, evaluate split condition, go left or right
- Copying/serializing a tree (save root first, then children)

**LC pattern:** Problem gives you info at the top that you carry downward.
Example: "Count Good Nodes" — carry max_so_far down, evaluate BEFORE recursing.

```python
def goodNodes(node, max_so_far):
    if node is None:
        return 0

    # EVAL — pre-order: check myself BEFORE going to kids
    count = 1 if node.val >= max_so_far else 0
    new_max = max(max_so_far, node.val)

    # RECURSE
    count += goodNodes(node.left, new_max)
    count += goodNodes(node.right, new_max)

    return count
```

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

**LC pattern:** Problem involves BST + needs sorted order.

```python
def in_order(node):
    if node is None:
        return []

    left_result = in_order(node.left)
    my_val = [node.val]
    right_result = in_order(node.right)

    return left_result + my_val + right_result

# On BST:     20
#            /  \
#          10    30
# Returns: [10, 20, 30] ← sorted
```

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
  This is how stack-based calculators work: push 10, push 5, see minus,
  pop both, push 5, push 3, see multiply, pop both, 15.
- Computing anything that needs children's answers first
- Deleting a tree: delete children before parent (can't delete parent first, you'd lose references to children)
- Calculating directory sizes: file sizes bubble up from leaves

**LC pattern:** You NEED your children's answers before you can compute yours.

```python
def maxDepth(node):
    if node is None:
        return 0

    left_depth = maxDepth(node.left)
    right_depth = maxDepth(node.right)

    # EVAL + COMBINE — post-order: I need kids' answers first
    return 1 + max(left_depth, right_depth)
```

**Why post-order here:** You can't know your depth until both children tell you theirs.
You're computing from the bottom up — leaves say "0", parents add 1.
This is like prefix sum but on a tree: values accumulate upward from leaves to root.

---

### Level-order (BFS): Go wide, level by level

```
Read: *, -, 3, 10, 5
```

NOT DFS. Uses a queue, not recursion.

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
- Shortest path in tree (fewest edges)
- Level-by-level processing (right side view, zigzag, level sums)
- "Nearest" anything in a tree

---

## The Decision — Which Traversal Do I Need?

Don't think about traversal names. Ask ONE question:

**"Do I need info from my PARENT or from my CHILDREN?"**

| I need...                           | Eval goes...        | Traversal   |
|-------------------------------------|---------------------|-------------|
| Info from parent to pass DOWN       | BEFORE recurse      | Pre-order   |
| Sorted output from BST             | BETWEEN recurse     | In-order    |
| Children's answers to compute mine  | AFTER recurse       | Post-order  |
| Level-by-level processing           | Use BFS, not DFS    | Level-order |

---

## Prefix Sum Analogy

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

---

## AI/ML Relevance

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
