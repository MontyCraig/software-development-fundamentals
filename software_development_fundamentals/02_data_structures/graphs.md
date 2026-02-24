# Graphs

## Overview

A graph is a non-linear data structure consisting of a set of **vertices** (also
called nodes) and a set of **edges** (connections between vertices). Graphs are the
most general-purpose data structure for representing relationships, and they model
an enormous variety of real-world systems: social networks, road maps, computer
networks, dependency chains, web links, and molecular structures.

Unlike trees, graphs can contain **cycles** (paths that return to the starting
vertex) and edges can be **directed** (one-way) or **undirected** (two-way). Edges
may also carry **weights** representing costs, distances, or capacities. A graph with
no cycles is called **acyclic**; a directed acyclic graph (DAG) is particularly
important for scheduling, dependency resolution, and topological sorting.

Two fundamental representations dominate: the **adjacency matrix** (a 2D array where
entry [i][j] indicates an edge from vertex i to vertex j) and the **adjacency list**
(each vertex stores a list of its neighbors). The choice between them depends on
graph density, the operations needed, and memory constraints. Two fundamental
traversal algorithms -- **Breadth-First Search (BFS)** and **Depth-First Search
(DFS)** -- form the basis for nearly all graph algorithms.

## Key Concepts

### Graph Terminology

- **Vertex (Node)**: A fundamental unit of the graph.
- **Edge**: A connection between two vertices.
- **Adjacent**: Two vertices connected by an edge.
- **Degree**: Number of edges incident to a vertex. For directed graphs: in-degree
  (incoming edges) and out-degree (outgoing edges).
- **Path**: A sequence of vertices where each adjacent pair is connected by an edge.
- **Cycle**: A path that starts and ends at the same vertex.
- **Connected**: An undirected graph where every vertex is reachable from every other.
- **Strongly connected**: A directed graph where every vertex is reachable from every
  other following edge directions.
- **Component**: A maximal connected subgraph.

### Types of Graphs

- **Undirected**: Edges have no direction (A-B means both A to B and B to A).
- **Directed (Digraph)**: Edges have direction (A->B does not imply B->A).
- **Weighted**: Edges carry numerical values (cost, distance, capacity).
- **Unweighted**: All edges are equal (or implicitly weight 1).
- **Cyclic**: Contains at least one cycle.
- **Acyclic**: Contains no cycles. A directed acyclic graph is called a **DAG**.
- **Dense**: Number of edges is close to V^2 (V = number of vertices).
- **Sparse**: Number of edges is close to V.
- **Complete**: Every pair of vertices is connected by an edge.
- **Bipartite**: Vertices can be divided into two sets with edges only between sets.

### Graph Density

```
Undirected: max edges = V * (V - 1) / 2
Directed:   max edges = V * (V - 1)

Dense graph:  E is close to V^2
Sparse graph: E is close to V
```

## Visual Representation

### Undirected Graph

```
    A --- B
    |   / |
    |  /  |
    | /   |
    C --- D --- E

Vertices: {A, B, C, D, E}
Edges: {A-B, A-C, B-C, B-D, C-D, D-E}
```

### Directed Graph (Digraph)

```
    A ---> B
    |    / |
    |   /  |
    v  v   v
    C ---> D ---> E

Vertices: {A, B, C, D, E}
Edges: {A->B, A->C, B->C, B->D, C->D, D->E}
```

### Weighted Graph

```
    A ---5--- B
    |       / |
    2     3   4
    |   /     |
    C ---1--- D ---6--- E

Edges with weights:
  A-B: 5, A-C: 2, B-C: 3, B-D: 4, C-D: 1, D-E: 6
```

### Adjacency Matrix (Undirected)

```
      A  B  C  D  E
  A [ 0  1  1  0  0 ]
  B [ 1  0  1  1  0 ]
  C [ 1  1  0  1  0 ]
  D [ 0  1  1  0  1 ]
  E [ 0  0  0  1  0 ]

1 = edge exists, 0 = no edge
Symmetric for undirected graphs.
Space: O(V^2)
```

### Adjacency Matrix (Weighted)

```
      A    B    C    D    E
  A [ 0    5    2   INF  INF ]
  B [ 5    0    3    4   INF ]
  C [ 2    3    0    1   INF ]
  D [ INF  4    1    0    6  ]
  E [ INF INF  INF   6    0  ]

INF = no direct edge
```

### Adjacency List

```
A: [B, C]
B: [A, C, D]
C: [A, B, D]
D: [B, C, E]
E: [D]

For weighted graph:
A: [(B,5), (C,2)]
B: [(A,5), (C,3), (D,4)]
C: [(A,2), (B,3), (D,1)]
D: [(B,4), (C,1), (E,6)]
E: [(D,6)]

Space: O(V + E)
```

### Adjacency Matrix vs Adjacency List Comparison

```
                  Adjacency Matrix    Adjacency List
                  +---+---+---+---+   A -> [B] -> [C]
                  | 0 | 1 | 1 | 0 |   B -> [A] -> [C] -> [D]
                  | 1 | 0 | 1 | 1 |   C -> [A] -> [B] -> [D]
                  | 1 | 1 | 0 | 1 |   D -> [B] -> [C]
                  | 0 | 1 | 1 | 0 |
                  +---+---+---+---+

Space:            O(V^2)              O(V + E)
Edge lookup:      O(1)                O(degree)
All neighbors:    O(V)                O(degree)
Add edge:         O(1)                O(1)
Best for:         Dense graphs        Sparse graphs
```

## Operations and Pseudocode

### Graph Representation (Adjacency List)

```
STRUCTURE Graph:
    vertices: Set of Vertex
    adjacencyList: Map from Vertex to List of (Vertex, Weight)
    isDirected: Boolean
END STRUCTURE

FUNCTION addVertex(graph: Graph, v: Vertex) -> Void:
    IF v NOT IN graph.vertices THEN
        ADD v TO graph.vertices
        SET graph.adjacencyList[v] = EMPTY List
    END IF
END FUNCTION

FUNCTION addEdge(graph: Graph, u: Vertex, v: Vertex, weight: Number) -> Void:
    APPEND (v, weight) TO graph.adjacencyList[u]
    IF NOT graph.isDirected THEN
        APPEND (u, weight) TO graph.adjacencyList[v]
    END IF
END FUNCTION
```

### Breadth-First Search (BFS)

```
FUNCTION bfs(graph: Graph, start: Vertex) -> List:
    SET visited = EMPTY Set
    SET queue = EMPTY Queue
    SET result = EMPTY List

    ADD start TO visited
    CALL enqueue(queue, start)

    WHILE NOT isEmpty(queue) DO
        SET current = CALL dequeue(queue)
        APPEND current TO result

        FOR EACH (neighbor, weight) IN graph.adjacencyList[current] DO
            IF neighbor NOT IN visited THEN
                ADD neighbor TO visited
                CALL enqueue(queue, neighbor)
            END IF
        END FOR
    END WHILE

    RETURN result
END FUNCTION
```

### BFS Traversal Trace

```
Graph:  A --- B --- E
        |   / |
        |  /  |
        C --- D

BFS from A:

Step  Queue         Visited          Action
----  -----         -------          ------
  0   [A]           {A}              Start with A
  1   [B, C]        {A, B, C}        Dequeue A, enqueue neighbors B, C
  2   [C, D, E]     {A, B, C, D, E}  Dequeue B, enqueue neighbors D, E
  3   [D, E]        {A, B, C, D, E}  Dequeue C, D already visited
  4   [E]           {A, B, C, D, E}  Dequeue D, neighbors visited
  5   []            {A, B, C, D, E}  Dequeue E, done

BFS order: A, B, C, D, E
```

### Depth-First Search (DFS) - Iterative

```
FUNCTION dfsIterative(graph: Graph, start: Vertex) -> List:
    SET visited = EMPTY Set
    SET stack = EMPTY Stack
    SET result = EMPTY List

    CALL push(stack, start)

    WHILE NOT isEmpty(stack) DO
        SET current = CALL pop(stack)
        IF current NOT IN visited THEN
            ADD current TO visited
            APPEND current TO result

            FOR EACH (neighbor, weight) IN graph.adjacencyList[current] DO
                IF neighbor NOT IN visited THEN
                    CALL push(stack, neighbor)
                END IF
            END FOR
        END IF
    END WHILE

    RETURN result
END FUNCTION
```

### Depth-First Search (DFS) - Recursive

```
FUNCTION dfsRecursive(graph: Graph, start: Vertex) -> List:
    SET visited = EMPTY Set
    SET result = EMPTY List
    CALL dfsHelper(graph, start, visited, result)
    RETURN result
END FUNCTION

FUNCTION dfsHelper(graph: Graph, vertex: Vertex, visited: Set, result: List) -> Void:
    ADD vertex TO visited
    APPEND vertex TO result

    FOR EACH (neighbor, weight) IN graph.adjacencyList[vertex] DO
        IF neighbor NOT IN visited THEN
            CALL dfsHelper(graph, neighbor, visited, result)
        END IF
    END FOR
END FUNCTION
```

### DFS Traversal Trace

```
Graph:  A --- B --- E
        |   / |
        |  /  |
        C --- D

DFS from A (recursive, neighbors visited in alphabetical order):

Call Stack        Visited          Action
----------        -------          ------
dfs(A)            {A}              Visit A, recurse to B
  dfs(B)          {A,B}            Visit B, recurse to C
    dfs(C)        {A,B,C}          Visit C, recurse to D
      dfs(D)      {A,B,C,D}        Visit D, no unvisited neighbors
    back to C     {A,B,C,D}        C: no unvisited neighbors
  back to B       {A,B,C,D}        B: recurse to E
    dfs(E)        {A,B,C,D,E}      Visit E, no unvisited neighbors
  back to B       {A,B,C,D,E}      B: done
back to A         {A,B,C,D,E}      A: C already visited, done

DFS order: A, B, C, D, E
```

### Detect Cycle (Directed Graph)

```
FUNCTION hasCycle(graph: Graph) -> Boolean:
    SET WHITE = 0    // unvisited
    SET GRAY = 1     // in current path
    SET BLACK = 2    // fully processed

    SET color = MAP each vertex to WHITE

    FOR EACH vertex IN graph.vertices DO
        IF color[vertex] == WHITE THEN
            IF CALL dfsDetectCycle(graph, vertex, color) THEN
                RETURN TRUE
            END IF
        END IF
    END FOR

    RETURN FALSE
END FUNCTION

FUNCTION dfsDetectCycle(graph: Graph, vertex: Vertex, color: Map) -> Boolean:
    SET color[vertex] = GRAY

    FOR EACH (neighbor, weight) IN graph.adjacencyList[vertex] DO
        IF color[neighbor] == GRAY THEN
            RETURN TRUE            // back edge = cycle
        END IF
        IF color[neighbor] == WHITE THEN
            IF CALL dfsDetectCycle(graph, neighbor, color) THEN
                RETURN TRUE
            END IF
        END IF
    END FOR

    SET color[vertex] = BLACK
    RETURN FALSE
END FUNCTION
```

### Topological Sort (DAG)

```
FUNCTION topologicalSort(graph: Graph) -> List:
    SET visited = EMPTY Set
    SET stack = EMPTY Stack

    FOR EACH vertex IN graph.vertices DO
        IF vertex NOT IN visited THEN
            CALL topoHelper(graph, vertex, visited, stack)
        END IF
    END FOR

    SET result = EMPTY List
    WHILE NOT isEmpty(stack) DO
        APPEND pop(stack) TO result
    END WHILE
    RETURN result
END FUNCTION

FUNCTION topoHelper(graph: Graph, vertex: Vertex, visited: Set, stack: Stack) -> Void:
    ADD vertex TO visited
    FOR EACH (neighbor, weight) IN graph.adjacencyList[vertex] DO
        IF neighbor NOT IN visited THEN
            CALL topoHelper(graph, neighbor, visited, stack)
        END IF
    END FOR
    CALL push(stack, vertex)
END FUNCTION
```

## Complexity Analysis

| Operation              | Adjacency Matrix | Adjacency List |
|------------------------|------------------|----------------|
| Space                  | O(V^2)           | O(V + E)       |
| Add vertex             | O(V^2)*          | O(1)           |
| Add edge               | O(1)             | O(1)           |
| Remove edge            | O(1)             | O(degree)      |
| Check edge exists      | O(1)             | O(degree)      |
| Find all neighbors     | O(V)             | O(degree)      |
| BFS / DFS              | O(V^2)           | O(V + E)       |
| Topological sort       | O(V^2)           | O(V + E)       |

\* Adding a vertex to a matrix requires resizing the 2D array.

| Algorithm          | Time Complexity | Space Complexity | Notes                    |
|--------------------|-----------------|------------------|--------------------------|
| BFS                | O(V + E)        | O(V)             | Queue + visited set      |
| DFS                | O(V + E)        | O(V)             | Stack/recursion + visited|
| Topological sort   | O(V + E)        | O(V)             | DFS-based                |
| Cycle detection    | O(V + E)        | O(V)             | Three-color DFS          |
| Dijkstra           | O((V+E) log V)  | O(V)             | With binary heap         |
| Bellman-Ford       | O(V * E)        | O(V)             | Handles negative weights |
| Floyd-Warshall     | O(V^3)          | O(V^2)           | All-pairs shortest path  |

## When to Use / When NOT to Use

### Use Adjacency Matrix When:
- The graph is dense (E approaches V^2).
- You need O(1) edge existence checks.
- Memory is not a primary constraint.
- You perform matrix-based algorithms (Floyd-Warshall).

### Use Adjacency List When:
- The graph is sparse (E is much less than V^2).
- You need to iterate over neighbors efficiently.
- Memory is a concern.
- Most graph algorithms (BFS, DFS, Dijkstra) are list-friendly.

### Avoid Graphs When:
- The relationship is strictly hierarchical with no cycles (use a tree).
- You only need key-value lookups (use a hash table).
- Data is sequential (use an array or linked list).

## Real-World Applications

- **Social networks**: Users are vertices; friendships/follows are edges.
- **Navigation/Maps**: Intersections are vertices; roads are weighted edges.
- **Internet/Networks**: Routers are vertices; connections are edges.
- **Dependency resolution**: Build systems, package managers (DAGs).
- **Web crawling**: Pages are vertices; hyperlinks are directed edges.
- **Recommendation engines**: Bipartite graphs of users and products.
- **Circuit design**: Components are vertices; wires are edges.
- **Scheduling**: Tasks are vertices; dependencies are directed edges.

## Common Pitfalls and Best Practices

1. **Choosing the wrong representation**: Use adjacency lists for sparse graphs and
   adjacency matrices for dense graphs. The wrong choice wastes either time or space.
2. **Forgetting to mark visited nodes**: BFS and DFS without a visited set will loop
   infinitely on cyclic graphs.
3. **Confusing directed and undirected**: In an undirected graph, each edge must be
   stored in both directions in the adjacency list.
4. **Off-by-one in matrix indexing**: Ensure vertex labels map correctly to matrix
   indices, especially if vertices are not numbered 0 through V-1.
5. **Negative weight cycles**: Dijkstra does not handle negative weights. Use
   Bellman-Ford and check for negative cycles.
6. **Not handling disconnected components**: BFS/DFS from a single source only
   visits one connected component. Loop over all vertices for full coverage.

## Comparison with Related Structures

| Feature              | Graph        | Tree         | Linked List  | Hash Table   |
|----------------------|--------------|--------------|--------------|--------------|
| Cycles               | Allowed      | None         | None*        | N/A          |
| Direction            | Either       | Parent-child | One/two-way  | N/A          |
| Root node            | None         | Exactly one  | Head node    | N/A          |
| Edges per node       | Any          | 1 parent     | 1-2 links    | N/A          |
| Traversal            | BFS, DFS     | In/Pre/Post  | Sequential   | Iteration    |
| Use case             | Relationships| Hierarchy    | Sequence     | Lookup       |

\* Circular linked lists are an exception.

## Practice Problems

1. **Number of Islands**: Given a 2D grid of land (1) and water (0), count the
   number of islands. Use BFS or DFS to explore connected land cells.

2. **Clone Graph**: Given a reference to a node in a connected undirected graph,
   return a deep copy. Use BFS/DFS with a hash table to track cloned nodes.

3. **Course Schedule**: Given prerequisites as directed edges, determine if all
   courses can be completed (detect cycles in a directed graph).

4. **Shortest Path in Unweighted Graph**: Find the shortest path between two nodes
   using BFS. Return the path, not just the distance.

5. **Topological Sort of a DAG**: Given a DAG representing build dependencies,
   produce a valid build order using DFS-based topological sort.

6. **Bipartite Check**: Determine if a graph is bipartite using BFS with two-color
   assignment. If any edge connects same-color nodes, it is not bipartite.
