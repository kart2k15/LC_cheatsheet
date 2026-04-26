# Binary Search Cheatsheet v1

Written for 3 AM with zero context. Every edge case, every "why."

---

## §1 — Core Mechanics

Binary search throws away half the search space every iteration. Works when you can prove "answer is in this half" based on a single midpoint check.

**Does NOT require a sorted array.** Requires a **monotonic guarantee** — some property that lets you eliminate half. Sorted array is the most common source of that guarantee, but not the only one (see §4).

### Skeleton (your template — used everywhere)

```python
l = LOW_BOUND
r = HIGH_BOUND
res = DEFAULT_ANSWER

while l <= r:
    m = (l + r) // 2

    if CONDITION(m):
        res = m           # save candidate
        r = m - 1         # try to find better (go left)
        # OR l = m + 1    # try to find better (go right) — depends on min vs max
    else:
        l = m + 1         # OR r = m - 1
        
return res
```

**Why `l <= r` and not `l < r`?** With `l < r` you need `r = m` (not `r = m - 1`) to avoid infinite loops when `m == l`. The `l <= r` + `r = m - 1` template avoids that headache entirely. You save candidates in `res` instead of relying on convergence to a single point. Consistent, no special cases.

**Why `m = (l + r) // 2` and not `(l + r) / 2`?** Integer division. Python handles big ints natively so no overflow worry (unlike Java/C++ where you'd use `l + (r - l) // 2`).

---

## §2 — Template 1: Exact Match

**When:** searching a sorted array (or API) for a specific target value.

**Pattern:** find target → return immediately. No need for `res`.

```python
l = 0
r = len(arr) - 1

while l <= r:
    m = (l + r) // 2
    if arr[m] == TARGET:
        return m
    elif arr[m] < TARGET:
        l = m + 1
    else:
        r = m - 1

# if we get here, target doesn't exist
return -1
```

**Key points:**
- Three branches: equal (found it), less (go right), greater (go left)
- Returns INSIDE the loop — no `res` variable needed
- If loop ends without returning, target isn't there

**Example:** LC 374 Guess Number Higher or Lower — API returns -1/0/1 instead of array comparison. Same template, just `guess(m)` instead of `arr[m] vs target`.

**Optimization:** cache the API/comparison result. `g = guess(m)` then branch on `g`. Don't call `guess(m)` twice — matters when the check is expensive (network call, DB lookup).

---

## §3 — Template 2: Leftmost Valid (Lower Bound)

**When:** searching for the FIRST index/value where a condition becomes true. The condition is monotonic — once it's true, it stays true for everything to the right.

**Pattern:** condition true → save candidate, go left (maybe there's an earlier one). Condition false → go right.

```python
l = LOW_BOUND
r = HIGH_BOUND
res = DEFAULT  # worst case answer if nothing valid

while l <= r:
    m = (l + r) // 2
    if CONDITION(m):
        res = m        # m works! save it, but maybe something smaller works too
        r = m - 1      # search left for potentially earlier/smaller valid answer
    else:
        l = m + 1      # m doesn't work, need to go higher

return res
```

**Why this works:** the answer space looks like `[✗ ✗ ✗ ✗ ✓ ✓ ✓ ✓]`. You're finding the first ✓. Every time you find a ✓, there might be an earlier one to the left. Every time you find a ✗, the answer must be to the right. Loop converges, `res` holds the leftmost ✓.

### Example A: LC 2300 — Successful Pairs of Spells and Potions

Searching a **sorted array** for the first potion where `potion >= target`.

```python
# target = math.ceil(success / spell)
# search sorted potions for first index where potions[m] >= target
l = 0
r = len(potions) - 1
res = None  # None means no valid potion found

while l <= r:
    m = (l + r) // 2
    if potions[m] >= target:
        res = m
        r = m - 1
    else:
        l = m + 1

if res == None:
    return 0
else:
    return len(potions) - res  # count from res to end, NOT len(potions[res:])
```

**Trap:** `len(potions) - res` is O(1). `len(potions[res:])` creates a new list = O(m). See §9 trap #3.

**Math trick:** `spell * potion >= success` rearranges to `potion >= success / spell`. Search the sorted potions for `ceil(success / spell)` directly. No need to multiply spell × every potion (that's O(m) per spell, defeats the purpose of binary search). See §7 for ceiling division.

### Example B: LC 875 — Koko Eating Bananas

Searching an **answer space** (not an array) for the minimum speed where Koko finishes in time. See §5.

---

## §4 — Template 3: Condition Chase (No Sorted Array)

**When:** the array isn't sorted, but a monotonic GUARANTEE lets you prove "answer exists in this half" based on one midpoint check.

**Pattern:** same save-and-search skeleton, but the condition is structural, not value-based.

### Example: LC 162 — Find Peak Element

Array is unsorted, but `nums[-1] = nums[n] = -∞` (outside bounds = negative infinity) and no adjacent duplicates. This guarantees at least one peak exists.

At any midpoint, check if you're going uphill or downhill to the right:
- **Downhill** (`nums[m] > nums[m+1]`): right side is smaller, so `m` could be a peak. Save it, search left.
- **Uphill** (`nums[m] < nums[m+1]`): `m` is NOT a peak (right neighbor bigger). Peak MUST exist to the right — it has to come back down to -∞ eventually.

```python
l = 0
r = len(nums) - 1
res = 0

while l <= r:
    m = (l + r) // 2
    if m + 1 > len(nums) - 1 or nums[m] > nums[m + 1]:
        # downhill to the right, OR m is last element (right neighbor is -∞)
        # m COULD be a peak — right side confirmed smaller
        # search left to check if left side is also smaller
        res = m
        r = m - 1
    else:
        # uphill to the right — m is NOT a peak
        # peak MUST exist to the right (has to come back down to -∞)
        l = m + 1

return res
```

**Why `m + 1 > len(nums) - 1` goes in the SAME branch as downhill:** if `m` is the last index, right neighbor is -∞ by problem definition. That's smaller than `nums[m]`, so it's effectively downhill. Same logic, same branch.

**Why you can't return the first `res` you find:** `nums[m] > nums[m+1]` only confirms the RIGHT neighbor is smaller. The LEFT neighbor might still be bigger (you're on a downslope, not at a peak). The loop must converge to confirm both sides. See §9 trap #5.

**What's the "monotonic guarantee" here?** Not value ordering — structural. The array starts at -∞ and ends at -∞. If you're going uphill, it MUST come back down. That's the guarantee that lets you throw away half.

---

## §5 — Binary Search on Answer Space

**Big idea:** you're not searching an array. You're searching a range of possible answers — [min_possible, max_possible]. At each midpoint, you check "does this answer work?" and eliminate half.

**When it applies:** the answer is an integer in a known range, and valid answers form a contiguous block (all values above some threshold work, or all below). That monotonic property is what makes it binary searchable.

### Skeleton

```python
l = MIN_POSSIBLE_ANSWER
r = MAX_POSSIBLE_ANSWER
res = DEFAULT  # usually r for "find minimum", l for "find maximum"

while l <= r:
    m = (l + r) // 2
    if is_feasible(m):
        res = m
        r = m - 1    # finding MINIMUM valid → go left
        # l = m + 1  # finding MAXIMUM valid → go right
    else:
        l = m + 1    # or r = m - 1 depending on direction

return res
```

### Finding MINIMUM vs MAXIMUM

| Goal | `is_feasible` true → | `is_feasible` false → | Init `res` |
|---|---|---|---|
| Minimum valid answer | save, go LEFT (`r = m - 1`) | go RIGHT (`l = m + 1`) | `r` (worst case) |
| Maximum valid answer | save, go RIGHT (`l = m + 1`) | go LEFT (`r = m - 1`) | `l` (worst case) |

### Example: LC 875 — Koko Eating Bananas

Answer space = eating speed. Range = [1, max(piles)].

- `is_feasible(speed)` = "can Koko finish all piles in ≤ h hours at this speed?"
- For each pile: `hours = pile // speed + (1 if pile % speed != 0 else 0)`
- Sum all hours. If total ≤ h → feasible.
- Finding MINIMUM speed → save, go left.

```python
l_speed = 1
r_speed = max(piles)
koko_min_speed = r_speed  # worst case = max speed

while l_speed <= r_speed:
    m_speed = (l_speed + r_speed) // 2
    total_hrs = 0
    for pile in piles:
        hrs_for_p = pile // m_speed
        if pile % m_speed != 0:
            hrs_for_p = hrs_for_p + 1
        total_hrs = total_hrs + hrs_for_p

    if total_hrs <= h:
        koko_min_speed = m_speed  # works! try slower
        r_speed = m_speed - 1
    else:
        l_speed = m_speed + 1  # too slow, go faster

return koko_min_speed
```

### Other problems that use this exact pattern

| Problem | Answer space | is_feasible check |
|---|---|---|
| LC 875 Koko Eating Bananas | speed [1, max(piles)] | total hours ≤ h |
| LC 1011 Ship Packages in D Days | capacity [max(weights), sum(weights)] | can ship in ≤ D days |
| LC 410 Split Array Largest Sum | max subarray sum [max(nums), sum(nums)] | can split into ≤ k subarrays |
| LC 1482 Min Days to Make Bouquets | days [1, max(bloomDay)] | can make m bouquets |

All the same skeleton. The hard part is recognizing the problem IS binary search on answer space, then defining the range and the feasibility check.

---

## §6 — Merge Sort (Recursion Fundamental)

Not a binary search problem, but uses the same divide-and-conquer structure. Included here because it came up while solving LC 2300 and it's a post-order recursion fundamental.

**What we're doing:** split array in half until single elements (already sorted), then merge sorted halves back together. The merging is where sorting happens — splitting just sets up the recursion.

**This is post-order / bubble-up:** can't merge until both children are sorted. Same pattern as tree problems where you need children's answers before computing yours (e.g., maxDepth).

```python
def merge_sort(vals):
    # BASE CASE: 0 or 1 elements → already sorted
    if len(vals) <= 1:
        return vals

    m = len(vals) // 2
    l_sort = merge_sort(vals[:m])    # PAUSE — wait for left to finish
    r_sort = merge_sort(vals[m:])    # PAUSE — wait for right to finish
    return merge(l_sort, r_sort)     # POST-ORDER: combine after both children done

def merge(l_arr, r_arr):
    res = []
    l = r = 0
    while l < len(l_arr) and r < len(r_arr):
        if l_arr[l] <= r_arr[r]:     # <= for stable sort (left wins ties)
            res.append(l_arr[l])
            l = l + 1
        else:
            res.append(r_arr[r])
            r = r + 1
    # one side exhausted — dump the rest of the other
    # slicing beyond array length returns [] (no IndexError)
    res.extend(l_arr[l:])
    res.extend(r_arr[r:])
    return res
```

**Tree cheatsheet mapping:**

| Tree concept | Merge sort equivalent |
|---|---|
| Base case return | `len(vals) <= 1` → return as-is (leaf = single element) |
| Recurse left, PAUSE | `l_sort = merge_sort(vals[:m])` |
| Recurse right, PAUSE | `r_sort = merge_sort(vals[m:])` |
| Post-order eval | `merge(l_sort, r_sort)` — AFTER both children |
| Bubble UP | Sorted subarrays return upward through return values |

**Complexity:**

| | Time | Space |
|---|---|---|
| Merge sort | O(n log n) always — no best/worst variance | O(n log n) with slicing, O(n) in-place |
| Python `list.sort()` | O(n log n) Timsort, C-implemented | O(n) |

**Rule:** understand merge sort for fundamentals. Use `list.sort()` or `sorted()` in actual LC submissions — Python's recursive merge sort with slicing TLEs on large inputs due to constant factors.

---

## §7 — Ceiling Division

Comes up constantly in binary search feasibility checks (Koko, Ship Packages, etc.).

**The problem:** `pile=11, speed=4`. How many hours? `11/4 = 2.75`. Need 3 hours (can't eat 0.75 of an hour's worth).

**Three ways to compute:**

| Method | Code | Pros | Cons |
|---|---|---|---|
| math.ceil | `math.ceil(pile / speed)` | Readable | Float division — precision risk with very large numbers; needs `import math` |
| Integer trick | `(pile + speed - 1) // speed` | No floats, no imports | Less readable at 3 AM |
| Explicit check | `pile // speed + (1 if pile % speed != 0 else 0)` | Crystal clear what it does | Verbose |

**Why the integer trick works:** adding `speed - 1` before floor dividing "rounds up" any remainder. If pile is exactly divisible, `pile + speed - 1` doesn't push into the next multiple. If there's any remainder at all, it pushes over.

Concrete: `(11 + 4 - 1) // 4 = 14 // 4 = 3 ✓`. `(8 + 4 - 1) // 4 = 11 // 4 = 2 ✓`.

**Pick whichever you can read at 3 AM.** Explicit check (method 3) is the most self-documenting for interviews.

---

## §8 — Pattern Decision Table

**"Is this a binary search problem?"** Ask:

| Trigger | Pattern | Template |
|---|---|---|
| Sorted array + find exact value | Exact match | §2 |
| Sorted array + find first/last position where condition holds | Leftmost/rightmost valid | §3 |
| "Find minimum X such that Y" or "find maximum X such that Y" | Binary search on answer space | §5 |
| Unsorted but you can prove "answer in this half" from one check | Condition chase | §4 |
| Problem says O(log n) required | Almost certainly binary search | Check which template fits |
| Array not sorted, no monotonic guarantee | NOT binary search | Try sorting first, or different pattern |

**"Which template?"** Ask:

| Question | If yes → |
|---|---|
| Am I searching for a specific known value? | Template 1 (exact match) §2 |
| Am I searching for the first index where condition flips? | Template 2 (leftmost valid) §3 |
| Am I searching a range of possible answers, not an array? | Template 2 applied to answer space §5 |
| Am I chasing a structural guarantee (peak, valley, rotation point)? | Template 3 (condition chase) §4 |

---

## §9 — Complexity Cheat Sheet

| Scenario | Time | Space |
|---|---|---|
| Single binary search on array of size n | O(log n) | O(1) |
| n binary searches on array of size m (LC 2300) | O(n log m) | O(1) per search |
| Sort + n binary searches (LC 2300 total) | O(m log m + n log m) = O((n+m) log m) | O(m) for sort |
| Binary search on answer space, O(n) feasibility check (LC 875) | O(n log R) where R = answer range | O(1) |
| Merge sort | O(n log n) | O(n log n) with slicing, O(n) in-place |

---

## §10 — Common Traps

**Trap 1: Slicing for count.** `len(arr[i:])` creates a new list just to measure it = O(n). Use `len(arr) - i` = O(1). This caused TLE on LC 2300.

**Trap 2: Multiplying before searching.** If you need `spell * potion >= target`, don't multiply spell × every potion to build a products array (O(m) per spell). Rearrange: `potion >= target / spell` and search the original sorted potions. Saves O(m) per spell.

**Trap 3: Float precision on ceiling.** `math.ceil(10**18 / 3)` might give wrong answer due to float precision loss. Integer trick `(a + b - 1) // b` avoids this. For LC constraints (values ≤ 10^9, success ≤ 10^10) `math.ceil` is fine, but know the alternative.

**Trap 4: Boundary check on `m+1` or `m-1`.** If your condition checks `arr[m+1]` (like LC 162 peak), guard against out-of-bounds: `m + 1 > len(arr) - 1`. Put the guard FIRST with `or` so Python short-circuits. Same for `m-1 < 0`.

**Trap 5: Returning first candidate in condition chase.** `nums[m] > nums[m+1]` in LC 162 only confirms the right neighbor. Left neighbor might be bigger. The loop must converge — don't return `res` early from inside the loop.

**Trap 6: Wrong branch for boundary in LC 162.** When `m` is the last index (right neighbor = -∞), that's effectively DOWNHILL. Goes in the same branch as `nums[m] > nums[m+1]`, not in `else`. Getting this wrong skips valid peaks at the array's end.

**Trap 7: `l, r = 0` doesn't unpack.** Python can't unpack a single int into two variables. Use `l, r = 0, 0` or `l = r = 0`.

**Trap 8: Answer space bounds.** For "find minimum speed/capacity/etc.":
- Lower bound = smallest value that could theoretically work (usually 1)
- Upper bound = value that DEFINITELY works (e.g., max(piles) for Koko — every pile takes 1 hour)
- Getting bounds wrong means either missing the answer or wasting iterations

**Trap 9: Caching expensive checks.** LC 374's `guess(m)` or any API call — store the result: `g = guess(m)`, then branch on `g`. Don't call it twice per iteration.

**Trap 10: Python merge sort TLEs.** Recursive merge sort with list slicing has massive constant factors in Python (O(n) copies per level, function call overhead per recursion). Use `list.sort()` for submissions. Write merge sort to learn the fundamental, not to pass.
