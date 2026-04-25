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

Asked at Uber during a hit counter design round: *"how would you write this in PROD — what would you return, how do you check for failures?"* The interviewer was NOT asking about the algorithm. They were probing whether you think about API contracts — what happens when things go right, go wrong, or go weird.

All three patterns below are shown on the **same `add_back` method** so you can see how the same logic looks under each style.

---

### Pattern A — None + Raise (Internal Code Default)

**What it means:** the method returns nothing on success. On garbage input (wrong type, negative number, etc.) it CRASHES with an exception. The caller's job is to not pass garbage — if they do, it's a bug and should blow up loudly.

On valid no-ops (add_back on a number already in the set), it returns silently. That's not an error — the caller asked a valid question and the answer is "nothing changed."

```python
def add_back(self, num):
    # VALIDATION — bad input = bug = crash loud so the dev notices immediately.
    # isinstance check catches someone passing a string, float, None, etc.
    if not isinstance(num, int) or num < 1:
        raise ValueError(f"must be positive int, got val={num}, val_type={type(num).__name__}")
    # VALID NO-OP — num already in the set, nothing to do. Not an error.
    if counter <= num or num in in_heap:
        return None
    # ACTUAL MUTATION — num wasn't in the set, add it.
    heappush(exception_heap, num)
    in_heap.add(num)
    return None
```

**When to use:** internal code, helper functions, anything where the caller is YOUR code and bad input means YOU have a bug. Python standard library follows this pattern — `list.append()` returns None, `set.add()` returns None.

**What the caller looks like:**

```python
set_obj.add_back(num)   # fire and forget — trusts it worked, crashes if input was bad
```

---

### Pattern B — Bool ("Did This Mutate State?")

**What it means:** same as A, but the method returns `True` if it actually changed something, `False` if it was a no-op. The caller can now REACT to the outcome.

**What "deduplication" means concretely:** imagine a message queue that delivers the same message twice (network retry, consumer restart, etc.). Your handler calls `add_back(5)` twice. First call returns `True` — 5 was added. Second call returns `False` — 5 was already there, duplicate detected. Now you can:
- Count duplicates for monitoring ("10% of our add_back calls are dupes — our upstream is retrying too aggressively")
- Skip downstream work ("5 was already processed, don't recompute the report")
- Log it for debugging ("duplicate add_back for num=5 at 2:34 AM, source=queue_consumer_3")

```python
def add_back(self, num) -> bool:
    if not isinstance(num, int) or num < 1:
        raise ValueError(f"must be positive int, got val={num}, val_type={type(num).__name__}")
    # Returns False on no-op so caller knows "nothing changed"
    if counter <= num or num in in_heap:
        return False
    heappush(exception_heap, num)
    in_heap.add(num)
    return True
```

**When to use:** any time the caller cares about whether the operation actually did something. Common in:
- Deduplication systems (message queues, event processing)
- Retry logic ("did this retry actually accomplish anything or was it redundant?")
- Metrics/dashboards ("what % of add_back calls are redundant?")
- Idempotency checks ("calling this 5 times should have the same effect as calling it once")

**What "idempotency" means concretely:** calling `add_back(5)` once or 100 times has the same final result — 5 is in the set exactly once. The operation is safe to repeat. The bool return tells you WHETHER this particular call was the one that actually did it, without changing the outcome.

**What the caller looks like:**

```python
was_added = set_obj.add_back(num)
if was_added:
    log.info(f"Added {num} to set")
    downstream_service.notify(num)
else:
    metrics.increment("add_back.duplicate")
```

---

### Pattern C — Structured Result (API Boundaries / Observability)

**What it means:** instead of True/False, return an object with DETAILS about what happened and why. The caller doesn't just know "it didn't work" — they know "it didn't work because num was already in the counter region" vs "it didn't work because num was already in the heap."

**What "structured result" means concretely:** a dataclass (or dict, or protobuf, or JSON response) that bundles the outcome + the reason into one return value.

```python
from dataclasses import dataclass

@dataclass
class AddBackResult:
    added: bool       # did the set actually change?
    reason: str       # WHY — "added", "already_in_set", "in_counter_region"

def add_back(self, num) -> AddBackResult:
    if counter <= num:
        return AddBackResult(False, "in_counter_region")
    if num in in_heap:
        return AddBackResult(False, "already_in_set")
    heappush(exception_heap, num)
    in_heap.add(num)
    return AddBackResult(True, "added")
```

**When to use:** service-to-service boundaries where you need observability. Common in:
- REST APIs (the response body tells the frontend exactly what happened)
- gRPC handlers (structured response → client reacts differently per reason)
- Dashboards/monitoring ("35% of failures are 'already_in_set', 65% are 'in_counter_region'" → tells you something about caller behavior)
- Debugging in production ("why did add_back(7) fail at 3 AM? Check the reason field in the logs")

**Overkill for:** internal helpers, simple scripts, anything where a bool is enough.

**What the caller looks like:**

```python
result = set_obj.add_back(num)
if result.added:
    log.info(f"Added {num}")
else:
    log.warn(f"add_back({num}) no-op: {result.reason}")
    metrics.increment(f"add_back.noop.{result.reason}")
    # Now your Grafana dashboard shows:
    # - add_back.noop.already_in_set: 200/hour
    # - add_back.noop.in_counter_region: 50/hour
    # and you can reason about WHY no-ops are happening
```

---

### Summary Table

| Pattern | Returns | Caller sees | Use when |
| --- | --- | --- | --- |
| A (None + raise) | None on success, crash on bad input | "It worked" or program crashes | Internal code, simple helpers |
| B (Bool) | True if mutated, False if no-op | "Did anything change?" | Dedup, retries, metrics, idempotency |
| C (Structured) | Object with outcome + reason | "What happened and why?" | API boundaries, observability, dashboards |

**Default for interviews:** start with A. Mention B when the interviewer asks about retries/duplicates. Mention C when they ask about observability/monitoring. Each escalation shows you think about progressively more PROD-level concerns.

---

### The 5 Things the Uber Interviewer Was Actually Probing

1. **Success/failure contract** — what does the caller see on success? On failure? On weird edge cases? Is "nothing happened" distinguishable from "it worked"?

2. **Idempotency** — if network hiccups cause the same request to arrive twice, does the system corrupt? Does the caller know the second call was redundant? Can they safely retry without side effects?

3. **Validation** — someone passes `add_back("foo")` or `add_back(-5)`. Do you crash (good — bug caught early), silently swallow (bad — bug hides), or return an error (good for APIs)?

4. **Observability** — when something goes wrong at 3 AM in production, can you figure out what happened from logs/metrics? Does the return value give you enough signal to debug? Can your dashboard show failure breakdowns by reason?

5. **Concurrency** — if two threads call `add_back(5)` simultaneously, do you get 5 in the set once or twice? Do you need a lock? Is the hashset + heap combo thread-safe? (It's not — Python's GIL helps but doesn't guarantee atomicity across multiple data structure mutations.)

---

### Error Message Hygiene

Always include both value AND type in error messages — you need to tell apart `5` (int) from `"5"` (string) from `5.0` (float) when debugging at 3 AM.

```python
# Cleanest — type(num).__name__ strips the <class '...'> wrapper
f"val={num}, val_type={type(num).__name__}"
# Output: val=5, val_type=str

# Terser — Python's repr() automatically quotes strings, shows None as None, etc.
f"val={num!r}"
# Output: val='5'
```

`type(num).__name__` gives: `str`, `int`, `float`, `NoneType`
`str(type(num))` gives: `<class 'str'>` — noisier, same info, more typing.
`type(num)` inside f-string gives: same as `str(type(num))` — f-strings call str() automatically.
