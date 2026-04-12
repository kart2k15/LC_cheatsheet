# Tree Recursion Skeleton Cheatsheet
 
## The Universal Template
 
Every binary tree problem uses this same skeleton:
 
```python
def solve(node):
    if not node:
        return BASE_CASE
    
    left_result = solve(node.left)
    right_result = solve(node.right)
    
    return COMBINE(left_result, right_result)
```
 
You only change **BASE_CASE** and **COMBINE**. The structure never changes.
 
---
 
## How Each Problem Maps to the Skeleton
 
### Max Depth
- **BASE_CASE:** `return 0` (None has no depth)
- **COMBINE:** `return 1 + max(left_result, right_result)` (I'm 1 deeper than my tallest child)
 
```python
def maxDepth(node):
    if not node:
        return 0
    left_depth = maxDepth(node.left)
    right_depth = maxDepth(node.right)
    return 1 + max(left_depth, right_depth)
```
 
---
 
### Min Depth
- **BASE_CASE:** `return 0`
- **COMBINE:** tricky — if one side is None, take the other side (don't take min with 0)
 
```python
def minDepth(node):
    if not node:
        return 0
    left = minDepth(node.left)
    right = minDepth(node.right)
    if left == 0:
        return 1 + right
    if right == 0:
        return 1 + left
    return 1 + min(left, right)
```
 
---
 
### Is Balanced (height difference <= 1 at every node)
- **BASE_CASE:** `return 0`
- **COMBINE:** if either child returns -1 OR height diff > 1, return -1 (unbalanced). Else return height.
 
```python
def isBalanced(root):
    return getHeight(root) != -1
 
def getHeight(node):
    if not node:
        return 0
    left = getHeight(node.left)
    right = getHeight(node.right)
    if left == -1 or right == -1:
        return -1
    if abs(left - right) > 1:
        return -1
    return 1 + max(left, right)
```
 
---
 
### Diameter (longest path between any two nodes)
- **BASE_CASE:** `return 0`
- **COMBINE:** path through me = left + right. Update global max. Return 1 + max(left, right) as my height.
 
```python
def diameterOfBinaryTree(root):
    max_diameter = [0]
    
    def getHeight(node):
        if not node:
            return 0
        left = getHeight(node.left)
        right = getHeight(node.right)
        max_diameter[0] = max(max_diameter[0], left + right)
        return 1 + max(left, right)
    
    getHeight(root)
    return max_diameter[0]
```
 
---
 
### Same Tree
- **BASE_CASE:** both None → `return True`. One None → `return False`.
- **COMBINE:** my values match AND left subtrees match AND right subtrees match.
 
```python
def isSameTree(p, q):
    if not p and not q:
        return True
    if not p or not q:
        return False
    if p.val != q.val:
        return False
    return isSameTree(p.left, q.left) and isSameTree(p.right, q.right)
```
 
---
 
### Invert Tree (mirror it)
- **BASE_CASE:** `return None`
- **COMBINE:** swap my children, return myself.
 
```python
def invertTree(node):
    if not node:
        return None
    left = invertTree(node.left)
    right = invertTree(node.right)
    node.left = right
    node.right = left
    return node
```
 
---
 
### Path Sum (does any root-to-leaf path sum to target?)
- **BASE_CASE:** None → `return False`
- **COMBINE:** am I a leaf AND remaining target == my val? Else check left or right.
 
```python
def hasPathSum(node, target):
    if not node:
        return False
    if not node.left and not node.right:
        return node.val == target
    return (hasPathSum(node.left, target - node.val) or 
            hasPathSum(node.right, target - node.val))
```
 
---
 
### Lowest Common Ancestor (LCA)
- **BASE_CASE:** None → `return None`. Found p or q → `return node`.
- **COMBINE:** both sides returned something → I'm the LCA. One side returned → pass it up.
 
```python
def lowestCommonAncestor(root, p, q):
    if not root:
        return None
    if root == p or root == q:
        return root
    left = lowestCommonAncestor(root.left, p, q)
    right = lowestCommonAncestor(root.right, p, q)
    if left and right:
        return root
    if left:
        return left
    return right
```
 
---
 
### Count Nodes
- **BASE_CASE:** `return 0`
- **COMBINE:** `return 1 + left + right`
 
```python
def countNodes(node):
    if not node:
        return 0
    left = countNodes(node.left)
    right = countNodes(node.right)
    return 1 + left + right
```
 
---
 
### Max Path Sum (path can start/end anywhere)
- **BASE_CASE:** `return 0`
- **COMBINE:** path through me = me + left + right (update global max). Return me + max(left, right) as best one-sided path.
 
```python
def maxPathSum(root):
    max_sum = [float('-inf')]
    
    def solve(node):
        if not node:
            return 0
        left = max(0, solve(node.left))
        right = max(0, solve(node.right))
        max_sum[0] = max(max_sum[0], node.val + left + right)
        return node.val + max(left, right)
    
    solve(root)
    return max_sum[0]
```
 
---
 
### Validate BST
- **Different skeleton** — pass constraints DOWN instead of bubbling results UP.
 
```python
def isValidBST(node, lo=float('-inf'), hi=float('inf')):
    if not node:
        return True
    if node.val <= lo or node.val >= hi:
        return False
    return (isValidBST(node.left, lo, node.val) and 
            isValidBST(node.right, node.val, hi))
```
 
---
 
## General Tree (N children) Version
 
Same skeleton, loop instead of left/right:
 
```python
def solve(node):
    if not node:
        return BASE_CASE
    
    results = []
    for child in node.children:
        results.append(solve(child))
    
    return COMBINE(results)
```
 
### Max Depth of N-ary Tree
 
```python
def maxDepth(node):
    if not node:
        return 0
    max_child = 0
    for child in node.children:
        child_depth = maxDepth(child)
        if child_depth > max_child:
            max_child = child_depth
    return 1 + max_child
```
 
---
 
## The Pattern in Plain English
 
1. **Am I None?** → return the base case (0, True, False, None — depends on problem)
2. **Ask my left child** → store its answer
3. **Ask my right child** → store its answer
4. **Combine their answers with mine** → return result up to my parent
 
Every node does the same thing. Nobody knows about the full tree. Each node only knows itself and what its children told it. That's recursion.
