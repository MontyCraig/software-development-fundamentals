# Graphs

## Overview
A collection of nodes (vertices) connected by edges, representing relationships between objects.

## Types and Representations

### 1. Graph Types
- Directed Graphs
- Undirected Graphs
- Weighted Graphs
- Cyclic vs Acyclic Graphs
- Connected vs Disconnected Graphs

### 2. Graph Representations
- Adjacency Matrix
  - Space complexity: O(V²)
  - Quick edge weight lookups
  - Better for dense graphs
- Adjacency List
  - Space complexity: O(V + E)
  - Better for sparse graphs
  - Efficient iteration over neighbors

## Core Operations and Algorithms

### 1. Basic Operations
- Add vertex: O(1)
- Remove vertex: O(V)
- Add edge: O(1)
- Remove edge: O(E)
- Check if edge exists: O(1) for matrix, O(V) for list

### 2. Graph Traversal
- Breadth-First Search (BFS)
  - Level-by-level exploration
  - Queue-based implementation
  - Time complexity: O(V + E)
- Depth-First Search (DFS)
  - Recursive exploration
  - Stack-based implementation
  - Time complexity: O(V + E)

### 3. Shortest Path Algorithms
- Dijkstra's Algorithm
- Bellman-Ford Algorithm
- Floyd-Warshall Algorithm

## Implementation Files
- `graph.py`: Base graph implementation
- `directed_graph.py`: Directed graph implementation
- `weighted_graph.py`: Weighted graph implementation
- `graph_algorithms.py`: Common graph algorithms
- `graph_utils.py`: Utility functions

## Testing
- `test_graph.py`
- `test_directed_graph.py`
- `test_weighted_graph.py`
- `test_graph_algorithms.py`

## Applications
1. Social Networks
2. Road Networks/GPS
3. Network Routing
4. Dependency Resolution 