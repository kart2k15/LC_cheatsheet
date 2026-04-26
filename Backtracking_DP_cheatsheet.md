# Backtracking + DP Cheatsheet v2

Written for 3 AM with zero context.

DP sections added after we solve DP problems.

---

## §1 — Mental Model

**Backtracking = DFS on an invisible decision tree.**

There is no TreeNode. The recursion CREATES the tree. Every function call is a node. Every choice is a branch. Every complete path is a leaf.

You already know how to DFS a tree. Backtracking is the same thing — the tree just doesn't exist as a data structure.

**It's pre-order / bubble DOWN.** You carry `path` (choices made so far) downward to children. Nothing combines upward. Same as carrying `target_sum` down in path sum problems.

**It's variable-depth nested for loops.** Each recursive call IS one for loop. 3 levels deep = 3 nested for loops. 10 levels deep = 10. Recursion handles the depth you can't hardcode.

---

## §2 — The 4 Components

Every backtracking problem has these 4 things. The problem only changes WHAT goes inside each one. The structure never changes.

| Component | What it does | Tree equivalent |
|---|---|---|
| **Base case** | Path is complete → save it | Reached a leaf node |
| **Choices** | What options exist at this level | Children of current node |
| **Constraints** | Skip options that can't lead to valid solutions | Pruning — don't visit dead branches |
| **Backtrack step** | Undo current choice before trying next one | Return from left child before visiting right child |

---

## §3 — The Two Skeletons

Same template, two shapes. The problem tells you which one to use.

### Form 1: For-loop (multiple choices per level)

Use when each level has multiple options to pick from (letters, numbers from a pool, etc).

```
def backtrack(params):
    if base_case_condition:
        results.append(copy_of_solution)
        return

    for choice in choices:
        if violates_constraints:
            continue

        make_choice
        backtrack(updated_params)
        undo_choice                  # backtracking step
```

### Form 2: Include/Skip (binary choice per item)

Use when you walk through items one by one and ask "take it or leave it?" for each.

```
def backtrack(params):
    if base_case_condition:
        results.append(copy_of_solution)
        return

    if violates_constraints:
        return

    # Branch 1: include current item
    make_choice
    backtrack(updated_params_with_choice)
    undo_choice

    # Branch 2: skip current item
    backtrack(updated_params_without_choice)
```

### When to use which

| Signal | Form |
|---|---|
| "Pick a letter from these options" | For-loop — multiple choices per level |
| "Include this item or skip it" | Include/Skip — binary decision per item |
| Each level has DIFFERENT options | For-loop |
| Each level has the SAME question (yes/no) for the next item | Include/Skip |

Both produce decision trees. For-loop makes an n-ary tree. Include/skip makes a binary tree. Same 4 components either way: base case, choices, constraints, undo.

**Pieces explained (same for both forms):**

- `params` — whatever state the current level needs (which item you're on, what's been chosen so far, running total, etc.).
- `base_case_condition` — path is complete → save a COPY and stop. Reached a leaf of the invisible tree.
- `violates_constraints` — should I prune? For-loop: `continue` to skip one option. Include/skip: `return` to cut the whole branch.
- `make_choice` → `backtrack(...)` → `undo_choice` — choose, explore, unchoose. Every make pairs with an undo.
- `results` — outer function owns it. Helper mutates it in place. No return needed from helper.

---

## §4 — Why Pop

The for loop runs multiple iterations. Each iteration appends ONE option. Without pop, the next iteration appends ON TOP of the previous iteration's choice.

```
WITHOUT pop:
  path = [A]
  iteration 1: append X → path = [A, X] → recurse → come back
  iteration 2: append Y → path = [A, X, Y]  ← WRONG. wanted [A, Y]

WITH pop:
  path = [A]
  iteration 1: append X → path = [A, X] → recurse → come back → pop → path = [A]
  iteration 2: append Y → path = [A, Y]  ← CORRECT
```

Pop resets path to what it was BEFORE the current iteration touched it. That's all it does.

---

## §5 — Alternative: No Pop Needed

If you create a NEW object instead of mutating, there's nothing to undo:

```python
for option in get_choices(index):
    backtrack(index + 1, path + [option])   # new list, path untouched
```

No pop, no deepcopy (each path is already its own list). Tradeoff: more memory (new list every call). For LC, doesn't matter.

**Rule:** pick one per problem. Don't mix append/pop with `path + [option]` in the same solution.

---

## §6 — Mapping to Tree Cheatsheet

| Tree DFS (you know this) | Backtracking (same thing) |
|---|---|
| Tree exists as TreeNode objects | Tree is invisible — recursion creates it |
| `if node == None: return` | `if index == len(items): save and return` |
| Visit left child, then right child | For loop: try option 1, then option 2, then option 3... |
| Carry value DOWN (pre-order) | Carry `path` DOWN via append |
| Return value UP (post-order) | NOT used in backtracking |
| Don't mutate shared state | DO mutate `path` — hence pop to undo |

---

## §7 — When to Use Backtracking

| Problem says | Use |
|---|---|
| "Return ALL combinations / permutations / subsets" | Backtracking |
| "Generate ALL valid X" | Backtracking |
| "Find minimum / maximum / optimal" | DP or greedy — NOT backtracking |
| "Count the number of ways" | Usually DP — backtracking works but slower |

Backtracking = exponential time (2ⁿ, kⁿ, n!). That's expected when generating ALL solutions. If the problem only wants one answer or a count, there's usually a faster approach.

---

## §8 — Common Traps

**Trap 1: Forgetting deepcopy.** `res.append(path)` saves a reference. Path keeps mutating. All entries in res end up as `[]`. Use `path[:]` (flat list) or `copy.deepcopy(path)`.

**Trap 2: Forgetting pop.** Every `append` needs a matching `pop`. No exceptions. If you can't track it, use the no-pop approach (§5).

**Trap 3: Mixing approaches.** Don't `append/pop` in one branch and `path + [x]` in another.

**Trap 4: Thinking it's post-order.** It's pre-order / bubble down. Pop between siblings is cleanup, not post-order computation.

**Trap 5: Duplicate results.** If problem wants unique combinations, the for loop needs a `start` parameter so each level only picks options ≥ previous pick. Without this → `[A,B]` AND `[B,A]` both appear.

**Trap 6: Not pruning.** If you KNOW a branch is dead (sum exceeded, too many items), `return None` or `continue` immediately. Pruning = same correctness, less wasted time.

**Trap 7: Using backtracking for optimization.** "Find minimum cost" → DP. "Find all paths" → backtracking. Wrong tool = TLE.

---

## §9 — DP Sections (Coming After DP Problems)

**Preview:** DP = backtracking + caching. If the invisible decision tree has overlapping subproblems (same state reachable from multiple paths), DP caches the result instead of re-exploring. Details after we solve the DP section.
