# Section 2: Data Structures

## Overview

Data structures are the organizational schemes that determine how data is stored,
accessed, and modified in a program. Choosing the right data structure is often the
single most impactful decision in software design -- it dictates the performance
characteristics of every operation your program performs.

This section covers the essential data structures from linear collections through
hierarchical and graph-based organizations. Each topic includes pseudocode implementations,
ASCII diagrams showing memory layout and structural relationships, and complexity analysis
for all major operations.

All material is language-agnostic. The pseudocode and concepts translate directly to any
programming environment.

---

## Table of Contents

### 1. [Arrays](arrays.md)
The most fundamental data structure: contiguous, fixed-size collections with O(1) random
access. Covers static and dynamic arrays, multidimensional arrays, memory layout, common
operations (insert, delete, search), and amortized analysis of dynamic resizing.

### 2. [Linked Lists](linked_lists.md)
Pointer-based linear collections where each node references the next. Covers singly linked
lists, doubly linked lists, circular variants, and sentinel nodes. Includes pseudocode for
insertion, deletion, reversal, and cycle detection, with node-and-pointer ASCII diagrams.

### 3. [Stacks](stacks.md)
Last-In, First-Out (LIFO) collections fundamental to expression evaluation, function call
management, and backtracking algorithms. Covers array-based and linked-list-based
implementations, common applications (parenthesis matching, undo systems), and the call
stack concept.

### 4. [Queues](queues.md)
First-In, First-Out (FIFO) collections used in scheduling, buffering, and breadth-first
traversal. Covers linear queues, circular queues, double-ended queues (deques), and
priority queues. Includes implementation strategies and application examples.

### 5. [Hash Tables](hash_tables.md)
Associative data structures providing near-O(1) average-case lookups. Covers hash
functions, collision resolution (chaining and open addressing), load factors, resizing
policies, and performance analysis. Includes ASCII diagrams of bucket layouts and
probing sequences.

### 6. [Trees](trees.md)
Hierarchical structures with a root node and parent-child relationships. Covers binary
trees, binary search trees (BST), AVL trees, and tree traversals (in-order, pre-order,
post-order, level-order). Includes ASCII tree diagrams and pseudocode for balancing
operations.

### 7. [Heaps](heaps.md)
Specialized tree-based structures satisfying the heap property, used for priority queues
and efficient sorting. Covers min-heaps, max-heaps, array-based representation, heapify
operations, and heap sort. Includes step-by-step ASCII diagrams of insertion and extraction.

### 8. [Graphs](graphs.md)
The most general data structure: vertices connected by edges, modeling networks, maps,
dependencies, and relationships. Covers representations (adjacency matrix, adjacency list),
directed and undirected graphs, weighted graphs, and basic traversals (BFS, DFS).

---

## Recommended Reading Order

The topics are arranged from simple to complex. Follow the numbered order for the most
natural progression:

1. **Arrays** -- The baseline; nearly every other structure builds on or contrasts with arrays.
2. **Linked Lists** -- Introduces pointer-based thinking, essential for trees and graphs.
3. **Stacks** -- A constrained linear structure; reinforces linked-list and array concepts.
4. **Queues** -- The FIFO counterpart to stacks; prepares you for BFS traversal later.
5. **Hash Tables** -- Introduces hashing; a leap in abstraction from sequential structures.
6. **Trees** -- The first hierarchical structure; builds on node/pointer concepts.
7. **Heaps** -- A specialized tree; easier to understand after general trees.
8. **Graphs** -- The most general structure; synthesizes traversal and representation ideas.

## Prerequisites

- Solid understanding of [Section 1: Core Programming](../01_core_programming/), especially
  variables, control flow, and functions.
- Familiarity with basic complexity notation (Big-O) is helpful but not required; it is
  introduced formally in [Section 3: Algorithms](../03_algorithms/).

## What You Will Learn

After completing this section, you will be able to:
- Select the appropriate data structure for a given problem based on operation requirements
- Implement each structure from scratch using pseudocode
- Analyze the time and space complexity of data structure operations
- Visualize memory layout and pointer relationships
- Recognize real-world applications of each data structure
