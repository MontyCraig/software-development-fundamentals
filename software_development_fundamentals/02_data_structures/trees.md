# Trees

## Overview

A tree is a hierarchical, non-linear data structure consisting of **nodes** connected
by **edges**. One distinguished node is the **root**, and every other node has exactly
one parent. Nodes with no children are called **leaves**. Trees model hierarchical
relationships -- file systems, organizational charts, parse trees, and decision
processes are all naturally tree-shaped.

A **binary tree** restricts each node to at most two children (left and right). A
**binary search tree (BST)** imposes an ordering invariant: for every node, all keys
in the left subtree are smaller and all keys in the right subtree are larger. This
enables O(log n) search, insertion, and deletion in the average case, but degrades
to O(n) if the tree becomes unbalanced (e.g., inserting sorted data into a plain BST).

**Self-balancing trees** solve this problem. **AVL trees** maintain strict balance
(height difference of at most 1 between subtrees) using rotations. **Red-Black trees**
use a coloring scheme and rotations to guarantee O(log n) worst-case operations with
fewer rotations than AVL. **B-trees** generalize to multi-way branching, optimized
for disk-based storage where minimizing tree height reduces expensive I/O operations.

## Key Concepts

### Tree Terminology

- **Root**: The topmost node (no parent).
- **Parent / Child**: A node directly above / below another.
- **Sibling**: Nodes sharing the same parent.
- **Leaf**: A node with no children.
- **Depth**: Number of edges from the root to a node.
- **Height**: Number of edges on the longest path from a node to a leaf.
- **Height of tree**: Height of the root node.
- **Subtree**: A node and all its descendants.
- **Degree**: Number of children a node has.

### Binary Search Tree Property

For every node N:
- All keys in N.left are less than N.key.
- All keys in N.right are greater than N.key.

### AVL Balance Factor

For every node: `balanceFactor = height(left) - height(right)`. Must be -1, 0, or 1.

### Red-Black Tree Rules

1. Every node is red or black.
2. The root is black.
3. Every leaf (NULL sentinel) is black.
4. Red nodes cannot have red children (no two consecutive reds).
5. Every path from a node to its descendant leaves has the same number of black nodes.

## Visual Representation

### Binary Tree

```
             10
            /  \
           5    15
          / \   / \
         3   7 12  20
        /     \      \
       1       8     25

Height = 3, Nodes = 9, Leaves = {1, 8, 12, 25}
```

### BST Insertion Sequence: 10, 5, 15, 3, 7, 12, 20

```
Insert 10:     Insert 5:      Insert 15:     Insert 3,7,12,20:
   10             10             10                10
                 /              /  \              /    \
                5              5    15           5      15
                                               / \    /  \
                                              3   7  12   20
```

### AVL Rotations

```
RIGHT ROTATION (when left-heavy):

        30                20
       /                 /  \
      20        -->    10    30
     /
    10

LEFT ROTATION (when right-heavy):

    10                    20
      \                  /  \
       20       -->    10    30
         \
          30

LEFT-RIGHT ROTATION (left child is right-heavy):

      30              30               20
     /               /                /  \
    10      -->    20        -->    10    30
      \           /
       20        10

RIGHT-LEFT ROTATION (right child is left-heavy):

    10              10                 20
      \               \              /  \
       30     -->      20    -->   10    30
      /                  \
     20                   30
```

### Red-Black Tree

```
         (B)10
        /      \
     (R)5     (R)15
     / \       / \
  (B)3 (B)7 (B)12 (B)20

B = Black, R = Red
Black height from root to any leaf = 2
```

### B-Tree (Order 3)

```
             [10 | 20]
            /    |    \
     [3|5|7]  [12|15]  [22|25|30]

Each node can hold 1-2 keys (order 3 = max 3 children).
All leaves are at the same depth.
```

## Operations and Pseudocode

### BST Search

```
FUNCTION search(node: TreeNode, key: KeyType) -> TreeNode:
    IF node == NULL THEN
        RETURN NULL
    END IF
    IF key == node.key THEN
        RETURN node
    ELSE IF key < node.key THEN
        RETURN CALL search(node.left, key)
    ELSE
        RETURN CALL search(node.right, key)
    END IF
END FUNCTION
```

### BST Insert

```
FUNCTION insert(node: TreeNode, key: KeyType, value: ValueType) -> TreeNode:
    IF node == NULL THEN
        RETURN NEW TreeNode(key, value)
    END IF
    IF key < node.key THEN
        SET node.left = CALL insert(node.left, key, value)
    ELSE IF key > node.key THEN
        SET node.right = CALL insert(node.right, key, value)
    ELSE
        SET node.value = value     // key exists, update value
    END IF
    RETURN node
END FUNCTION
```

### BST Delete

```
FUNCTION delete(node: TreeNode, key: KeyType) -> TreeNode:
    IF node == NULL THEN
        RETURN NULL
    END IF
    IF key < node.key THEN
        SET node.left = CALL delete(node.left, key)
    ELSE IF key > node.key THEN
        SET node.right = CALL delete(node.right, key)
    ELSE
        // Node found -- three cases
        IF node.left == NULL THEN
            RETURN node.right
        ELSE IF node.right == NULL THEN
            RETURN node.left
        ELSE
            // Two children: replace with in-order successor
            SET successor = CALL findMin(node.right)
            SET node.key = successor.key
            SET node.value = successor.value
            SET node.right = CALL delete(node.right, successor.key)
        END IF
    END IF
    RETURN node
END FUNCTION

FUNCTION findMin(node: TreeNode) -> TreeNode:
    WHILE node.left != NULL DO
        SET node = node.left
    END WHILE
    RETURN node
END FUNCTION
```

### Tree Traversals

```
FUNCTION inOrder(node: TreeNode) -> Void:
    IF node == NULL THEN RETURN END IF
    CALL inOrder(node.left)
    VISIT node
    CALL inOrder(node.right)
END FUNCTION

FUNCTION preOrder(node: TreeNode) -> Void:
    IF node == NULL THEN RETURN END IF
    VISIT node
    CALL preOrder(node.left)
    CALL preOrder(node.right)
END FUNCTION

FUNCTION postOrder(node: TreeNode) -> Void:
    IF node == NULL THEN RETURN END IF
    CALL postOrder(node.left)
    CALL postOrder(node.right)
    VISIT node
END FUNCTION

FUNCTION levelOrder(root: TreeNode) -> Void:
    IF root == NULL THEN RETURN END IF
    SET queue = EMPTY Queue
    CALL enqueue(queue, root)
    WHILE NOT isEmpty(queue) DO
        SET node = CALL dequeue(queue)
        VISIT node
        IF node.left != NULL THEN
            CALL enqueue(queue, node.left)
        END IF
        IF node.right != NULL THEN
            CALL enqueue(queue, node.right)
        END IF
    END WHILE
END FUNCTION
```

### Traversal Results for the Example Tree

```
Tree:        10
            /  \
           5    15
          / \   / \
         3   7 12  20

In-Order:    3, 5, 7, 10, 12, 15, 20   (sorted order for BST)
Pre-Order:   10, 5, 3, 7, 15, 12, 20   (root first)
Post-Order:  3, 7, 5, 12, 20, 15, 10   (root last)
Level-Order: 10, 5, 15, 3, 7, 12, 20   (breadth-first)
```

### AVL Insertion with Rebalancing

```
FUNCTION avlInsert(node: AVLNode, key: KeyType) -> AVLNode:
    // Standard BST insert
    IF node == NULL THEN
        RETURN NEW AVLNode(key)
    END IF
    IF key < node.key THEN
        SET node.left = CALL avlInsert(node.left, key)
    ELSE IF key > node.key THEN
        SET node.right = CALL avlInsert(node.right, key)
    ELSE
        RETURN node    // duplicate key
    END IF

    // Update height
    SET node.height = 1 + MAX(height(node.left), height(node.right))

    // Check balance factor
    SET balance = height(node.left) - height(node.right)

    // Left-Left case
    IF balance > 1 AND key < node.left.key THEN
        RETURN CALL rightRotate(node)
    END IF
    // Right-Right case
    IF balance < -1 AND key > node.right.key THEN
        RETURN CALL leftRotate(node)
    END IF
    // Left-Right case
    IF balance > 1 AND key > node.left.key THEN
        SET node.left = CALL leftRotate(node.left)
        RETURN CALL rightRotate(node)
    END IF
    // Right-Left case
    IF balance < -1 AND key < node.right.key THEN
        SET node.right = CALL rightRotate(node.right)
        RETURN CALL leftRotate(node)
    END IF

    RETURN node
END FUNCTION

FUNCTION rightRotate(y: AVLNode) -> AVLNode:
    SET x = y.left
    SET T2 = x.right
    SET x.right = y
    SET y.left = T2
    SET y.height = 1 + MAX(height(y.left), height(y.right))
    SET x.height = 1 + MAX(height(x.left), height(x.right))
    RETURN x
END FUNCTION

FUNCTION leftRotate(x: AVLNode) -> AVLNode:
    SET y = x.right
    SET T2 = y.left
    SET y.left = x
    SET x.right = T2
    SET x.height = 1 + MAX(height(x.left), height(x.right))
    SET y.height = 1 + MAX(height(y.left), height(y.right))
    RETURN y
END FUNCTION
```

## Complexity Analysis

| Operation    | BST (avg) | BST (worst) | AVL       | Red-Black | B-Tree      |
|--------------|-----------|-------------|-----------|-----------|-------------|
| Search       | O(log n)  | O(n)        | O(log n)  | O(log n)  | O(log n)    |
| Insert       | O(log n)  | O(n)        | O(log n)  | O(log n)  | O(log n)    |
| Delete       | O(log n)  | O(n)        | O(log n)  | O(log n)  | O(log n)    |
| In-order     | O(n)      | O(n)        | O(n)      | O(n)      | O(n)        |
| Space        | O(n)      | O(n)        | O(n)      | O(n)      | O(n)        |
| Height       | O(log n)  | O(n)        | ~1.44logn | ~2log n   | O(log_m n)  |
| Rotations/op | -         | -           | O(log n)  | O(1)*     | -           |

\* Red-Black trees perform at most 2 rotations per insert and 3 per delete.

## When to Use / When NOT to Use

### Use BSTs/Balanced Trees When:
- You need ordered data with fast search, insert, and delete.
- You need in-order traversal to produce sorted output.
- You need range queries (find all keys between A and B).
- Worst-case guarantees matter (use AVL or Red-Black).

### Use B-Trees When:
- Data is stored on disk (databases, file systems).
- Minimizing tree height (and thus I/O operations) is critical.
- Nodes can hold many keys, matching disk block sizes.

### Avoid Trees When:
- You only need key-value lookup without ordering (use a hash table).
- Data is small and a sorted array with binary search suffices.
- You need constant-time access by index (use an array).

## Real-World Applications

- **File systems**: Directory hierarchies are trees.
- **Databases**: B-trees and B+ trees power database indexes.
- **Compilers**: Abstract syntax trees (ASTs) represent parsed code.
- **Routing**: Trie-based routing tables for IP address lookup.
- **Autocomplete**: Tries store dictionaries for prefix search.
- **Priority queues**: Binary heaps are complete binary trees.
- **Decision making**: Decision trees in machine learning.
- **Version control**: Commit history forms directed acyclic graphs with tree-like merges.

## Common Pitfalls and Best Practices

1. **Unbalanced BST**: Inserting sorted data into a plain BST produces a linked list
   with O(n) operations. Always use a self-balancing variant for production code.
2. **Confusing traversal orders**: In-order gives sorted output for BSTs. Pre-order
   is useful for tree copying. Post-order is useful for deletion. Know the difference.
3. **Forgetting the two-child delete case**: BST deletion with two children requires
   finding the in-order successor (or predecessor), not just removing the node.
4. **Incorrect rotation implementation**: Off-by-one errors in AVL rotations corrupt
   the tree. Always test with all four rotation cases.
5. **Not maintaining height/color**: Forgetting to update height (AVL) or color
   (Red-Black) after operations violates invariants silently.

## Comparison with Related Structures

| Feature              | BST        | AVL        | Red-Black  | B-Tree     | Hash Table |
|----------------------|------------|------------|------------|------------|------------|
| Search (avg)         | O(log n)   | O(log n)   | O(log n)   | O(log n)   | O(1)       |
| Search (worst)       | O(n)       | O(log n)   | O(log n)   | O(log n)   | O(n)       |
| Ordered iteration    | Yes        | Yes        | Yes        | Yes        | No         |
| Range queries        | Yes        | Yes        | Yes        | Yes        | No         |
| Disk-optimized       | No         | No         | No         | Yes        | No         |
| Rotations per insert | 0          | O(log n)   | <=2        | 0          | N/A        |
| Implementation       | Simple     | Moderate   | Complex    | Complex    | Simple     |

## Practice Problems

1. **Validate BST**: Write a function that determines whether a given binary tree
   satisfies the BST property.

2. **Lowest Common Ancestor**: Find the lowest common ancestor of two nodes in a
   BST in O(log n) time.

3. **Serialize and Deserialize**: Convert a binary tree to a string representation
   and reconstruct it. Use pre-order traversal with NULL markers.

4. **Kth Smallest Element**: Find the kth smallest element in a BST. Augment nodes
   with subtree sizes for O(log n) performance.

5. **Build BST from Sorted Array**: Given a sorted array, construct a balanced BST
   with minimum height using divide-and-conquer.

6. **AVL Insertion Sequence**: Given a sequence of insertions into an initially empty
   AVL tree, draw the tree after each insertion, showing all rotations performed.
