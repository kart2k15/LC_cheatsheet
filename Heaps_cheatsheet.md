# Heap / Priority Queue — Mental Model Cheatsheet

---

## 1. What Is a Heap

A heap is a list where `heap[0]` is always the smallest element (min-heap). That's the entire user-facing contract. You push stuff in, pop stuff out, and the smallest is always at index 0.

**It is NOT sorted.** You cannot compare `heap[3]` vs `heap[7]`. The only guarantee:

```
heap[0] <= every other element in the heap
```

Everything else is internal. No parent/child access, no tree-walking API. The heap is a black box.

**Why not a sorted list?**

| Operation | Sorted list | Heap |
| --- | --- | --- |
| Build from existing array | O(n log n) via sorted() | **O(n) via heapify()** |
| Insert one element | O(n) — shift elements | **O(log n)** |
| Pop min | O(1) from front / O(n) to shift | **O(log n)** |
| Peek min | O(1) | **O(1) — just heap[0]** |
| Get all sorted | O(1) — already sorted | O(n log n) — repeated heappop |

Heap wins when you're **repeatedly inserting AND popping**. Sorted list wins if you sort once and never touch it again.

---

## 2. Python's `heapq` — The 4 Operations You'll Use

```python
import heapq

# 1. BUILD from existing list — O(n), mutates in place
heapq.heapify(arr)

# 2. PUSH — O(log n)
heapq.heappush(heap, x)

# 3. POP min — O(log n), returns heap[0] and removes it
smallest = heapq.heappop(heap)

# 4. PEEK min — O(1), no function, just index 0
top = heap[0]
```

**heapify vs n pushes:**
If you have the full array upfront → always `heapify` (O(n)). Pushing one at a time is O(n log n). Only use `heappush` in a loop when elements arrive incrementally.

---

## 3. Max-Heap in Python

Python ONLY has min-heap. Two workarounds:

**Workaround 1 — Negate values (works everywhere):**

```python
heapq.heappush(heap, -x)          # push: negate going in
largest = -heapq.heappop(heap)    # pop: negate coming out
largest = -heap[0]                # peek: negate
```

**Workaround 2 — Python 3.14+ native max-heap:**

```python
heapq.heapify_max(list)
heapq.heappush_max(heap, item)
heapq.heappop_max(heap)
```

LC may not have 3.14 yet. **For interviews: use negate.** Works everywhere.

---

## 4. Tuple Comparison — Free Tiebreaking

Python compares tuples left to right. Heaps containing tuples sort by first element, then second on tie.

```python
(2, 3) < (2, 5)   # cost tied → compare index → 3 < 5 → True
(2, 3) < (3, 1)   # cost 2 < 3 → True, index never checked
```

**Use case:** problem says "break ties by smallest index" → store `(cost, idx)`. Tiebreaker is automatic.

**Trap:** `(idx, cost)` sorts by index first — wrong. **Order in the tuple matters.**

---

## 5. Pattern 1 — Top K / Kth Element

### Trigger Words

"kth largest", "kth smallest", "top k", "k closest", "maximum score with k picks"

### Core Idea

Kth largest = smallest of the k largest elements → `heap[0]` of a min-heap bounded at size k.

Think of it as a **VIP club with exactly k seats.** Every new element pushes in. When the club overflows k, evict the weakest (heap root). At the end, `heap[0]` = kth largest.

**Flip for kth smallest:** max-heap of size k, negate values.

### Skeleton

```
heap = []
for item in stream:
    heappush(heap, item)
    if len(heap) > k:          # overflow → evict weakest
        heappop(heap)          # > k not >= k! At capacity is the goal.
result = heap[0]               # smallest of top k = kth largest
```

**Why `> k` not `>= k`:** push grows heap by 1. At size k = goal state. At k+1 = overflowed, pop. `>= k` would pop at capacity and lose an element you want.

### Extension — Sort + Top-K

When score = (sum of k values from array1) × (min of corresponding array2 values):

**"Fix the minimum by sorting array2 descending, maximize the sum with a top-k heap on array1, multiply at each step."**

```
pairs = sorted(zip(arr1, arr2), key=by arr2, reverse=True)
heap = [], running_sum = 0, best = 0
for val1, val2 in pairs:
    heappush(heap, val1)
    running_sum += val1
    if len(heap) > k:
        running_sum -= heappop(heap)
        best = max(best, running_sum * val2)
    elif len(heap) == k:
        best = max(best, running_sum * val2)
return best
```

Current val2 is always the forced minimum of everything seen (sorted descending). Heap keeps the best sum. Score = best sum × forced min.

---

## 6. Pattern 2 — Implicit Counter + Heap of Exceptions

### Trigger Words

"infinite set", "stream with undo/restore", "add back", anything where the data structure is too big to store explicitly but follows a predictable rule.

### Core Idea

Split state into three pieces:

1. **Counter** — represents the predictable part implicitly ("everything from counter onward exists").
2. **Exception heap** — small bag of items that deviate from the rule. All strictly less than counter.
3. **Sidecar hashset** — mirrors the heap for O(1) "is this already in the heap?" checks, since heaps don't support fast membership lookup.

### Skeleton

```
counter = initial_value
exception_heap = []
in_heap = set()                    # mirrors heap for O(1) membership

pop_smallest:
    if exception_heap non-empty:   # exceptions < counter → they always win
        answer = heappop(exception_heap)
        in_heap.remove(answer)     # keep mirror in sync
    else:
        answer = counter
        counter += 1

add_back(num):
    if counter <= num:             # in counter region → already exists → no-op
        return
    elif num in in_heap:           # in heap → already exists → no-op
        return
    else:
        heappush(exception_heap, num)
        in_heap.add(num)           # keep mirror in sync
```

### Why the Hashset

Heaps allow duplicates. Without the hashset, repeated add_back calls push the same value multiple times → pop returns it multiple times → set semantics broken. Hashset deduplicates in O(1).

**Rule:** every `heappush` pairs with `set.add()`. Every `heappop` pairs with `set.remove()`. Always in lockstep.

---

## 7. Pattern 3 — Two Heaps / Two Windows

### Trigger Words

"first k and last k", "two ends", "pick from either end", "hire workers", "cheapest from left or right window"

### Core Idea

Two separate min-heaps, one for each end of the array. A middle pool sits between them (deque or two pointers). Each round pop the cheaper of the two heap roots. Refill the depleted heap from its side of the middle.

### Skeleton (Deque — Cleaner)

```
r_start = n - window_size
if r_start < window_size:
    r_start = window_size                  # overlap guard

l_heap = arr[:window_size]
r_heap = arr[r_start:]
mid    = deque(arr[window_size : n - window_size])
heapify(l_heap)
heapify(r_heap)

for each round:
    if l_heap non-empty AND (r_heap empty OR l_heap[0] <= r_heap[0]):
        pop from l_heap
        if mid non-empty:
            heappush(l_heap, mid.popleft())       # refill from left of middle
    else:
        pop from r_heap
        if mid non-empty:
            heappush(r_heap, mid.pop())            # refill from right of middle
```

### Skeleton (Two Pointers — Supports Tuples / Index Tracking)

```
l_heap = [(arr[i], i) for i in range(window_size)]
r_heap = [(arr[i], i) for i in range(r_start, n)]
heapify(l_heap), heapify(r_heap)

l = window_size                     # next refill index for left (moves →)
r = n - 1 - window_size            # next refill index for right (moves ←)

for each round:
    if r_heap empty OR (l_heap non-empty AND l_heap[0] <= r_heap[0]):
        pop from l_heap
        if l <= r:
            heappush(l_heap, (arr[l], l))
            l += 1
    else:
        pop from r_heap
        if l <= r:
            heappush(r_heap, (arr[r], r))
            r -= 1
```

### Deque vs Pointers

| | Deque | Two Pointers |
| --- | --- | --- |
| Mental model | Slice once, pop from ends | Explicit index tracking |
| Tiebreaking | Implicit (insertion order + `<=`) | Explicit via (cost, idx) tuples |
| Space | O(n) for deque | O(1) extra |
| Use when | Raw values, no index needed | Need to track which element was picked |

### Key Guards

**Overlap guard:** if `window_size * 2 >= n`, windows overlap. Clamp `r_start = max(window_size, n - window_size)`. If fully overlapping, `r_heap` is empty — the `r_heap empty` check handles it.

**Refill guard:** `l <= r` not `l < r`. When `l == r`, one unused middle element remains — still valid. `l < r` would skip it. `l > r` = nothing left.

**Tiebreaker:** `<=` picks left on tie. Left covers smaller indices by construction → correct for "break tie by smallest index."

---

## 8. Pattern Decision Table

| Problem shape | Pattern | Heap type |
| --- | --- | --- |
| "Kth largest / smallest" | Top-K (§5) | Min-heap of size k |
| "Top k frequent / closest" | Top-K (§5) | Min-heap of size k |
| "Max score = sum × min from two arrays" | Sort + Top-K (§5) | Min-heap + sort |
| "Infinite set with add-back / restore" | Counter + Exceptions (§6) | Min-heap + hashset |
| "Pick cheapest from two ends" | Two Heaps (§7) | Two min-heaps + deque/pointers |
| "Median from stream" | Two Heaps (not in LC 75) | Max-heap (small half) + min-heap (large half) |

---

## 9. Complexity Cheat Sheet

### By Operation

| Operation | Time | Notes |
| --- | --- | --- |
| heapify(list) | O(n) | Bottom-up. ALWAYS use over n pushes if you have full array. |
| heappush(heap, x) | O(log n) | n = current heap size |
| heappop(heap) | O(log n) | Returns heap[0] and removes it |
| heap[0] (peek) | O(1) | No function, just index |
| n separate heappush calls | O(n log n) | Worse than heapify. Avoid. |

### By Pattern

| Pattern | Time | Space |
| --- | --- | --- |
| Top-K | O(n log k) | O(k) |
| Sort + Top-K | O(n log n) | O(n) sorted pairs + O(k) heap |
| Counter + Exceptions | O(log n) per op | O(n) heap + hashset |
| Two Heaps / Two Windows | O(n + k log n) | O(n) deque or O(candidates) heaps |

### Top-K vs Alternatives

| Approach | Time | Space | When to use |
| --- | --- | --- | --- |
| Sort then pick | O(n log n) | O(log n) | Trivial but slower when k << n |
| Min-heap size k (Top-K) | O(n log k) | O(k) | **Interview default.** Streams, bounded memory. |
| Heapify-all + pop k | O(n + k log n) | O(n) | Faster time when k small. Can't stream. |
| Counting sort | O(n + m), m=value range | O(m) | Only when values are bounded small integers |
| Quickselect | O(n) avg, O(n²) worst | O(n) | Fastest avg. Hard to code. Not expected. |

---

## 10. Common Traps

1. **`> k` not `>= k` for overflow check.** Heap at size k = goal. Pop at k loses an element. Pop at k+1 is the fix.
2. **Forgetting hashset mirror with heap.** Heaps allow duplicates. Every heappush pairs with `set.add()`, every heappop pairs with `set.remove()`. Always in lockstep.
3. **Reassigning the running scalar inside the loop.** Same frozen-prefix trap as BFS (Graph Cheatsheet §7-prefix). `running_sum -= heappop()` is fine. Don't mutate a scalar that downstream computations share.
4. **Overlap guard on two-window problems.** Clamp `r_start = max(window_size, n - window_size)` to prevent double-counting.
5. **`l <= r` not `l < r` for refill guard.** `l == r` = one unused element remains. `l < r` would skip it.
6. **heapify vs n pushes.** heapify O(n). n pushes O(n log n). Always heapify if you have the full array.
7. **Python only has min-heap.** Negate on push AND pop for max-heap. Forgetting to negate on pop = garbage.
8. **`heap[0]` is the ONLY thing you can trust.** Heap is not sorted. Arbitrary positions are meaningless.
9. **Tuple ordering matters.** `(cost, idx)` sorts cost-first. `(idx, cost)` sorts index-first. Wrong order = wrong results.
10. **`<=` for left-preference tiebreaking.** In two-heap problems, `<=` picks left on tie → left covers smaller indices → correct. `<` gives right priority on ties.

---

## 11. Appendix: PROD-Grade API Design for Void Mutators (The "Uber Probe")

Asked at Uber during a hit counter design round: *"how would you write this in PROD — what would you return, how do you check for failures?"* Three patterns on the same `add_back` method:

### Pattern A — None + Raise (Internal Code Default)

```python
def add_back(self, num):
    if not isinstance(num, int) or num < 1:
        raise ValueError(f"must be positive int, got val={num}, val_type={type(num).__name__}")
    if counter <= num or num in in_heap:
        return None       # valid no-op
    heappush(exception_heap, num)
    in_heap.add(num)
    return None
```

**When:** internal helpers. Loud on bad input, silent on valid no-change.

### Pattern B — Bool ("Did This Mutate?")

```python
def add_back(self, num) -> bool:
    if counter <= num or num in in_heap:
        return False
    heappush(exception_heap, num)
    in_heap.add(num)
    return True
```

**When:** caller cares about idempotency — deduplication, retries, metrics.

### Pattern C — Structured Result (API Boundaries)

```python
@dataclass
class AddBackResult:
    added: bool
    reason: str    # "added" | "already_in_set" | "in_counter_region"
```

**When:** service boundaries — gRPC, REST, observability/dashboards.

### What the Uber Probe Was Really Asking

1. **Success/failure contract** — what does the caller see?
2. **Idempotency** — duplicates handled? Signaled?
3. **Validation** — bad input: crash, swallow, or report?
4. **Observability** — logs, metrics, drop reasons.
5. **Concurrency** — thread-safety implications.

### Error Message Hygiene

```python
# Cleanest — strips <class '...'> wrapper
f"val={num}, val_type={type(num).__name__}"    # → val=5, val_type=str

# Terser — repr() quotes strings, shows None
f"val={num!r}"                                  # → val='5'
```
