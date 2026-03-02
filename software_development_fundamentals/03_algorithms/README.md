# Section 3: Algorithms

## Overview

Algorithms are step-by-step procedures for solving computational problems. While data
structures organize information, algorithms transform and query it. Together, they form
the core of computer science.

This section covers the major algorithmic paradigms and techniques: complexity analysis
to reason about efficiency, searching and sorting as foundational operations, recursion
and backtracking for exploring solution spaces, dynamic programming and greedy strategies
for optimization, and graph algorithms for network and relationship problems.

Every algorithm is presented with pseudocode implementations, ASCII trace diagrams showing
execution steps, and formal complexity analysis. The material is entirely language-agnostic.

---

## Table of Contents

### 1. [Complexity Analysis](complexity_analysis.md)
The mathematical framework for evaluating algorithm efficiency. Covers Big-O, Big-Omega,
and Big-Theta notation, best/average/worst-case analysis, amortized analysis, and space
complexity. Includes comparison charts and growth-rate diagrams for common complexity
classes.

### 2. [Searching Algorithms](searching_algorithms.md)
Techniques for locating elements within data structures. Covers linear search, binary
search, interpolation search, and jump search. Includes step-by-step trace diagrams,
pseudocode for each variant, and comparative complexity analysis.

### 3. [Comparison Sorting](comparison_sorting.md)
Sorting algorithms that order elements by pairwise comparison. Covers bubble sort,
selection sort, insertion sort, merge sort, quicksort, and heapsort. Includes trace
diagrams showing each algorithm's execution on sample data and analysis of stability,
adaptivity, and in-place properties.

### 4. [Non-Comparison Sorting](non_comparison_sorting.md)
Sorting algorithms that break the O(n log n) lower bound by exploiting structure in the
data. Covers counting sort, radix sort, and bucket sort. Explains when and why these
algorithms outperform comparison-based sorting, with pseudocode and worked examples.

### 5. [Recursion and Backtracking](recursion_and_backtracking.md)
Problem-solving through self-referential function calls and systematic exploration of
candidate solutions. Covers recursive thinking, base cases and recursive cases, the call
stack, memoization, and backtracking with pruning. Includes recursion tree diagrams and
classic problems (N-Queens, permutations, subset generation).

### 6. [Dynamic Programming](dynamic_programming.md)
An optimization technique for problems with overlapping subproblems and optimal
substructure. Covers top-down (memoization) and bottom-up (tabulation) approaches, state
definition, transition formulation, and space optimization. Includes worked examples with
filled DP tables.

### 7. [Greedy Algorithms](greedy_algorithms.md)
Optimization strategies that make locally optimal choices at each step. Covers the greedy
choice property, optimal substructure, proof techniques for correctness, and classic
problems (activity selection, Huffman coding, fractional knapsack, minimum spanning trees).
Includes comparison with dynamic programming.

### 8. [Graph Algorithms](graph_algorithms.md)
Algorithms operating on graph structures for traversal, shortest paths, spanning trees,
and topological ordering. Covers BFS, DFS, Dijkstra's algorithm, Bellman-Ford, Floyd-Warshall,
Kruskal's and Prim's MST algorithms, and topological sort. Includes detailed ASCII
execution traces.

### 9. [String Algorithms](string_algorithms.md)
Algorithms for processing and searching text. Covers pattern matching (KMP, Rabin-Karp),
trie operations, edit distance (Levenshtein), and suffix arrays. Includes pseudocode
implementations and ASCII trace diagrams for each algorithm.

### 10. [Bit Manipulation](bit_manipulation.md)
Techniques for operating directly on binary representations of data. Covers binary
representation, bitwise operators (AND, OR, XOR, NOT, shifts), common bit tricks, and
bit masking. Includes truth tables and worked examples.

---

## Recommended Reading Order

Start with complexity analysis, then proceed through the topics:

1. **Complexity Analysis** -- You need this framework to evaluate everything that follows.
2. **Searching Algorithms** -- The simplest algorithmic problems; introduces binary search thinking.
3. **Comparison Sorting** -- Fundamental algorithms; introduces divide-and-conquer.
4. **Non-Comparison Sorting** -- Builds on sorting knowledge; introduces counting techniques.
5. **Recursion and Backtracking** -- Deepens divide-and-conquer; introduces solution-space exploration.
6. **Dynamic Programming** -- Extends recursion with memoization and tabulation.
7. **Greedy Algorithms** -- A simpler alternative to DP when greediness can be proven correct.
8. **Graph Algorithms** -- Synthesizes traversal, shortest-path, and optimization techniques.
9. **String Algorithms** -- Efficient text processing and pattern matching techniques.
10. **Bit Manipulation** -- Low-level binary operations and common bit tricks.

## Prerequisites

- [Section 1: Core Programming](../01_core_programming/) -- especially functions and recursion.
- [Section 2: Data Structures](../02_data_structures/) -- especially arrays, trees, and graphs.

## What You Will Learn

After completing this section, you will be able to:
- Classify algorithms by their time and space complexity
- Implement fundamental searching and sorting algorithms from pseudocode
- Apply recursive thinking and backtracking to constraint problems
- Recognize when dynamic programming or greedy strategies apply
- Implement graph traversals and shortest-path algorithms
- Apply string algorithms for pattern matching and text processing
- Use bit manipulation techniques for efficient low-level operations
- Choose the right algorithmic approach for a given problem
