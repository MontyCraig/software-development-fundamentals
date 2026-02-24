# Graph Algorithms

## Overview

Graphs are one of the most versatile data structures in computer science, modeling
relationships between entities in networks, maps, social connections, dependencies,
and countless other domains. A graph consists of vertices (nodes) connected by edges,
which may be directed or undirected, weighted or unweighted. Graph algorithms solve
fundamental problems like traversal, shortest paths, connectivity, and spanning trees.

Two foundational traversal algorithms -- Breadth-First Search (BFS) and Depth-First
Search (DFS) -- form the basis for nearly all graph algorithms. BFS explores vertices
layer by layer from a source, while DFS explores as deeply as possible before
backtracking. These traversals enable shortest-path computation, cycle detection,
topological ordering, and connected component identification.

This chapter covers traversal algorithms, shortest-path algorithms (Dijkstra and
Bellman-Ford), minimum spanning tree algorithms (Kruskal and Prim), topological sort,
and cycle detection. Each algorithm is presented with full pseudocode, step-by-step
traces on example graphs, and complexity analysis.

## Key Concepts

### Graph Representation

```
Adjacency List (preferred for sparse graphs):
  0: [1, 2]
  1: [0, 3]
  2: [0, 3]
  3: [1, 2]

Adjacency Matrix (preferred for dense graphs):
       0  1  2  3
  0: [ 0  1  1  0 ]
  1: [ 1  0  0  1 ]
  2: [ 1  0  0  1 ]
  3: [ 0  1  1  0 ]
```

| Representation   | Space    | Check Edge | List Neighbors | Add Edge |
|------------------|----------|------------|----------------|----------|
| Adjacency List   | O(V + E) | O(degree)  | O(degree)      | O(1)     |
| Adjacency Matrix | O(V^2)   | O(1)       | O(V)           | O(1)     |

### Graph Terminology

- **Directed/Undirected:** Edges have direction (one-way) or not (two-way)
- **Weighted/Unweighted:** Edges have associated costs or all edges cost 1
- **Cycle:** A path that starts and ends at the same vertex
- **Connected:** Every vertex is reachable from every other vertex
- **DAG:** Directed Acyclic Graph -- a directed graph with no cycles

## Algorithms and Pseudocode

### 1. Breadth-First Search (BFS)

Explore vertices layer by layer using a queue. Finds shortest paths in unweighted
graphs.

```
FUNCTION bfs(graph: AdjacencyList, source: Vertex) -> (distances: Map, parents: Map):
    SET visited = NEW Set
    SET distances = NEW Map
    SET parents = NEW Map
    SET queue = NEW Queue

    ADD source TO visited
    SET distances[source] = 0
    SET parents[source] = NONE
    ENQUEUE source INTO queue

    WHILE queue IS NOT EMPTY DO
        SET current = DEQUEUE(queue)
        FOR EACH neighbor IN graph[current] DO
            IF neighbor NOT IN visited THEN
                ADD neighbor TO visited
                SET distances[neighbor] = distances[current] + 1
                SET parents[neighbor] = current
                ENQUEUE neighbor INTO queue
            END IF
        END FOR
    END WHILE

    RETURN (distances, parents)
END FUNCTION
```

**Step-by-step trace on example graph:**

```
Graph:
    0 --- 1 --- 3
    |     |     |
    2 --- 4     5
          |
          6

Adjacency list:
  0: [1, 2]    1: [0, 3, 4]    2: [0, 4]
  3: [1, 5]    4: [1, 2, 6]    5: [3]      6: [4]

BFS from vertex 0:

Step  Queue         Visit  Distance  Parent
---   -----         -----  --------  ------
0     [0]           0      0         None
1     [1, 2]        1      1         0
2     [2, 3, 4]     2      1         0
3     [3, 4]        3      2         1
4     [4, 6]        4      2         1
5     [6, 5]        --     --        --      (3 already visited)
6     [5]           6      3         4
7     []            5      3         3

Layer 0: {0}
Layer 1: {1, 2}        (distance 1 from source)
Layer 2: {3, 4}        (distance 2 from source)
Layer 3: {5, 6}        (distance 3 from source)
```

### 2. Depth-First Search (DFS)

Explore as deep as possible along each branch before backtracking. Uses a stack
(explicit or via recursion).

```
FUNCTION dfs(graph: AdjacencyList, source: Vertex):
    SET visited = NEW Set
    dfs_visit(graph, source, visited)
END FUNCTION

FUNCTION dfs_visit(graph: AdjacencyList, vertex: Vertex, visited: Set):
    ADD vertex TO visited
    PROCESS(vertex)                         // e.g., record discovery time
    FOR EACH neighbor IN graph[vertex] DO
        IF neighbor NOT IN visited THEN
            dfs_visit(graph, neighbor, visited)
        END IF
    END FOR
END FUNCTION
```

**Iterative DFS using explicit stack:**

```
FUNCTION dfs_iterative(graph: AdjacencyList, source: Vertex):
    SET visited = NEW Set
    SET stack = NEW Stack
    PUSH source ONTO stack

    WHILE stack IS NOT EMPTY DO
        SET current = POP(stack)
        IF current NOT IN visited THEN
            ADD current TO visited
            PROCESS(current)
            FOR EACH neighbor IN graph[current] DO
                IF neighbor NOT IN visited THEN
                    PUSH neighbor ONTO stack
                END IF
            END FOR
        END IF
    END WHILE
END FUNCTION
```

**Step-by-step DFS trace (same graph as BFS):**

```
DFS from vertex 0 (recursive, processing neighbors in order):

Call Stack          Visit  Action
---------           -----  ------
dfs_visit(0)        0      Visit 0, explore neighbor 1
  dfs_visit(1)      1      Visit 1, explore neighbor 3
    dfs_visit(3)    3      Visit 3, explore neighbor 5
      dfs_visit(5)  5      Visit 5, no unvisited neighbors, return
    back to 3               No more unvisited neighbors, return
  back to 1                 Explore neighbor 4
    dfs_visit(4)    4      Visit 4, neighbor 1 visited, explore 2
      dfs_visit(2)  2      Visit 2, neighbor 0 visited, explore 4 visited, return
    back to 4               Explore neighbor 6
      dfs_visit(6)  6      Visit 6, neighbor 4 visited, return
    back to 4               Return
  back to 1                 Return
back to 0                   Neighbor 2 already visited, done

DFS order: 0, 1, 3, 5, 4, 2, 6
```

### BFS vs DFS Visualization

```
Same graph, different traversal orders:

Graph:        0
             / \
            1   2
           /|    \
          3 4 --- 2
          |  \
          5   6

BFS (level by level):           DFS (deep first):
Layer 0: [0]                    Path: 0 -> 1 -> 3 -> 5
Layer 1: [1, 2]                       backtrack to 3
Layer 2: [3, 4]                       backtrack to 1
Layer 3: [5, 6]                       1 -> 4 -> 2
                                      backtrack to 4
Visit: 0,1,2,3,4,5,6                 4 -> 6
                                Visit: 0,1,3,5,4,2,6
```

### 3. Dijkstra's Shortest Path Algorithm

Find the shortest path from a source to all other vertices in a graph with
non-negative edge weights.

```
FUNCTION dijkstra(graph: WeightedAdjList, source: Vertex) -> (dist: Map, parent: Map):
    SET dist = NEW Map, all values = INFINITY
    SET parent = NEW Map, all values = NONE
    SET dist[source] = 0
    SET pq = NEW MinPriorityQueue

    INSERT (0, source) INTO pq

    WHILE pq IS NOT EMPTY DO
        SET (d, u) = EXTRACT_MIN(pq)
        IF d > dist[u] THEN
            CONTINUE                        // stale entry, skip
        END IF
        FOR EACH (v, weight) IN graph[u] DO
            SET new_dist = dist[u] + weight
            IF new_dist < dist[v] THEN
                SET dist[v] = new_dist
                SET parent[v] = u
                INSERT (new_dist, v) INTO pq
            END IF
        END FOR
    END WHILE

    RETURN (dist, parent)
END FUNCTION
```

**Step-by-step trace:**

```
Graph (directed, weighted):
    A --2--> B --3--> D
    |        |        ^
    4        1        |
    |        v        2
    +------> C --5--> E --1--> D

Adjacency: A:[(B,2),(C,4)]  B:[(C,1),(D,3)]  C:[(E,5)]  D:[]  E:[(D,1)]

Dijkstra from A:

Step  Extract    dist[A] dist[B] dist[C] dist[D] dist[E]  Action
----  -------    ------- ------- ------- ------- -------  ------
Init  -          0       INF     INF     INF     INF      pq={(0,A)}
1     (0,A)      0       2       4       INF     INF      Relax B(2), C(4)
2     (2,B)      0       2       3       5       INF      Relax C(3), D(5)
3     (3,C)      0       2       3       5       8        Relax E(8)
4     (5,D)      0       2       3       5       8        D has no outgoing
5     (8,E)      0       2       3       5       8        Relax D? 8+1=9>5, no

Final distances: A:0, B:2, C:3, D:5, E:8

Shortest path to D: A -> B -> D (cost 5)
Shortest path to E: A -> B -> C -> E (cost 8)
```

### 4. Bellman-Ford Algorithm

Find shortest paths from a source, handling negative edge weights. Detects negative
weight cycles.

```
FUNCTION bellman_ford(vertices: List, edges: List of (u,v,w), source: Vertex) -> (dist, parent, has_neg_cycle):
    SET dist = NEW Map, all values = INFINITY
    SET parent = NEW Map, all values = NONE
    SET dist[source] = 0

    // Relax all edges V-1 times
    FOR iteration = 1 TO LENGTH(vertices) - 1 DO
        FOR EACH (u, v, w) IN edges DO
            IF dist[u] != INFINITY AND dist[u] + w < dist[v] THEN
                SET dist[v] = dist[u] + w
                SET parent[v] = u
            END IF
        END FOR
    END FOR

    // Check for negative weight cycles (V-th iteration)
    FOR EACH (u, v, w) IN edges DO
        IF dist[u] != INFINITY AND dist[u] + w < dist[v] THEN
            RETURN (dist, parent, true)     // negative cycle exists
        END IF
    END FOR

    RETURN (dist, parent, false)
END FUNCTION
```

**Step-by-step trace:**

```
Graph: A->B(1), B->C(3), A->C(10), C->D(2), B->D(-5)
Vertices: [A, B, C, D], Source: A

Iteration 1:
  Edge A->B(1):  dist[A]+1=1 < INF  -> dist[B]=1
  Edge B->C(3):  dist[B]+3=4 < INF  -> dist[C]=4
  Edge A->C(10): dist[A]+10=10 > 4  -> no change
  Edge C->D(2):  dist[C]+2=6 < INF  -> dist[D]=6
  Edge B->D(-5): dist[B]+(-5)=-4 < 6 -> dist[D]=-4

Iteration 2:
  All edges: no further improvements

Iteration 3:
  All edges: no further improvements

Negative cycle check: no improvements on V-th iteration -> no negative cycle

Final: A:0, B:1, C:4, D:-4
```

### 5. Kruskal's Minimum Spanning Tree

Build an MST by adding edges in order of increasing weight, skipping edges that
would create a cycle (using union-find).

```
FUNCTION kruskal(vertices: List, edges: List of (u,v,weight)) -> List of Edges:
    SORT edges BY weight ASCENDING
    SET mst = EMPTY LIST
    SET uf = NEW UnionFind(vertices)

    FOR EACH (u, v, weight) IN edges DO
        IF uf.FIND(u) != uf.FIND(v) THEN   // different components
            ADD (u, v, weight) TO mst
            uf.UNION(u, v)
            IF LENGTH(mst) = LENGTH(vertices) - 1 THEN
                BREAK                       // MST complete
            END IF
        END IF
    END FOR

    RETURN mst
END FUNCTION
```

**Step-by-step trace:**

```
Graph:
    A ---1--- B
    |  \      |
    4   3     2
    |    \    |
    C ---5--- D ---6--- E
    |                   |
    +--------7----------+

Edges sorted: (A,B,1), (B,D,2), (A,D,3), (A,C,4), (C,D,5), (D,E,6), (C,E,7)

Step  Edge     Weight  Action           Components
----  ----     ------  ------           ----------
1     (A,B)    1       ADD (diff comp)  {A,B}, {C}, {D}, {E}
2     (B,D)    2       ADD (diff comp)  {A,B,D}, {C}, {E}
3     (A,D)    3       SKIP (same comp) {A,B,D}, {C}, {E}
4     (A,C)    4       ADD (diff comp)  {A,B,C,D}, {E}
5     (C,D)    5       SKIP (same comp) {A,B,C,D}, {E}
6     (D,E)    6       ADD (diff comp)  {A,B,C,D,E}
DONE: 4 edges for 5 vertices.

MST edges: (A,B,1), (B,D,2), (A,C,4), (D,E,6)
Total weight: 1 + 2 + 4 + 6 = 13

MST visualization:
    A ---1--- B
    |         
    4         2
    |          \
    C          D ---6--- E
```

### 6. Prim's Minimum Spanning Tree

Grow the MST from a starting vertex by always adding the cheapest edge connecting
the tree to a non-tree vertex.

```
FUNCTION prim(graph: WeightedAdjList, start: Vertex) -> List of Edges:
    SET in_mst = NEW Set
    SET mst = EMPTY LIST
    SET pq = NEW MinPriorityQueue

    ADD start TO in_mst
    FOR EACH (neighbor, weight) IN graph[start] DO
        INSERT (weight, start, neighbor) INTO pq
    END FOR

    WHILE pq IS NOT EMPTY AND SIZE(in_mst) < NUM_VERTICES DO
        SET (weight, u, v) = EXTRACT_MIN(pq)
        IF v IN in_mst THEN
            CONTINUE
        END IF
        ADD v TO in_mst
        ADD (u, v, weight) TO mst
        FOR EACH (neighbor, w) IN graph[v] DO
            IF neighbor NOT IN in_mst THEN
                INSERT (w, v, neighbor) INTO pq
            END IF
        END FOR
    END WHILE

    RETURN mst
END FUNCTION
```

### 7. Topological Sort (DFS-based)

Order vertices of a DAG so that for every directed edge u->v, u appears before v.

```
FUNCTION topological_sort(graph: DirectedAdjList, vertices: List) -> List:
    SET visited = NEW Set
    SET result = NEW Stack

    FOR EACH vertex IN vertices DO
        IF vertex NOT IN visited THEN
            topo_dfs(graph, vertex, visited, result)
        END IF
    END FOR

    RETURN REVERSE(result)
END FUNCTION

FUNCTION topo_dfs(graph: AdjList, vertex: Vertex, visited: Set, result: Stack):
    ADD vertex TO visited
    FOR EACH neighbor IN graph[vertex] DO
        IF neighbor NOT IN visited THEN
            topo_dfs(graph, neighbor, visited, result)
        END IF
    END FOR
    PUSH vertex ONTO result                 // add after all descendants
END FUNCTION
```

**Step-by-step trace:**

```
DAG (course prerequisites):
  A -> B -> D
  A -> C -> D -> E
       C -> E

Adjacency: A:[B,C]  B:[D]  C:[D,E]  D:[E]  E:[]

DFS from A:
  Visit A -> Visit B -> Visit D -> Visit E
  E has no unvisited neighbors -> push E
  D done -> push D
  B done -> push B
  Back to A -> Visit C
  C -> D (visited), E (visited) -> push C
  A done -> push A

Stack (bottom to top): A, C, B, D, E
Topological order: A, C, B, D, E  (or A, B, C, D, E -- both valid)

Verification: Every edge goes left to right:
  A->B (ok), A->C (ok), B->D (ok), C->D (ok), C->E (ok), D->E (ok)
```

### 8. Cycle Detection

**In undirected graphs (using DFS):**

```
FUNCTION has_cycle_undirected(graph: AdjList, vertices: List) -> Boolean:
    SET visited = NEW Set
    FOR EACH vertex IN vertices DO
        IF vertex NOT IN visited THEN
            IF dfs_cycle(graph, vertex, NONE, visited) THEN
                RETURN true
            END IF
        END IF
    END FOR
    RETURN false
END FUNCTION

FUNCTION dfs_cycle(graph: AdjList, vertex: Vertex, parent: Vertex, visited: Set) -> Boolean:
    ADD vertex TO visited
    FOR EACH neighbor IN graph[vertex] DO
        IF neighbor NOT IN visited THEN
            IF dfs_cycle(graph, neighbor, vertex, visited) THEN
                RETURN true
            END IF
        ELSE IF neighbor != parent THEN
            RETURN true                     // back edge found = cycle
        END IF
    END FOR
    RETURN false
END FUNCTION
```

**In directed graphs (using coloring):**

```
FUNCTION has_cycle_directed(graph: DirectedAdjList, vertices: List) -> Boolean:
    SET color = NEW Map (all WHITE)

    FOR EACH vertex IN vertices DO
        IF color[vertex] = WHITE THEN
            IF dfs_cycle_directed(graph, vertex, color) THEN
                RETURN true
            END IF
        END IF
    END FOR
    RETURN false
END FUNCTION

FUNCTION dfs_cycle_directed(graph: AdjList, vertex: Vertex, color: Map) -> Boolean:
    SET color[vertex] = GRAY                // currently being explored
    FOR EACH neighbor IN graph[vertex] DO
        IF color[neighbor] = GRAY THEN
            RETURN true                     // back edge = cycle
        END IF
        IF color[neighbor] = WHITE THEN
            IF dfs_cycle_directed(graph, neighbor, color) THEN
                RETURN true
            END IF
        END IF
    END FOR
    SET color[vertex] = BLACK               // fully explored
    RETURN false
END FUNCTION
```

## Complexity Analysis

| Algorithm       | Time             | Space    | Notes                        |
|----------------|------------------|----------|------------------------------|
| BFS            | O(V + E)         | O(V)     | Queue-based traversal        |
| DFS            | O(V + E)         | O(V)     | Stack/recursion-based        |
| Dijkstra       | O((V+E) log V)   | O(V)     | With binary heap; non-neg w  |
| Bellman-Ford   | O(V * E)         | O(V)     | Handles negative weights     |
| Kruskal MST    | O(E log E)       | O(V)     | Sort edges + union-find      |
| Prim MST       | O((V+E) log V)   | O(V)     | With binary heap             |
| Topological Sort| O(V + E)        | O(V)     | DAGs only                    |
| Cycle Detection| O(V + E)         | O(V)     | DFS-based coloring           |

### Dijkstra vs Bellman-Ford

```
+------------------+-------------------+-------------------+
| Property         | Dijkstra          | Bellman-Ford      |
+------------------+-------------------+-------------------+
| Time complexity  | O((V+E) log V)    | O(V * E)          |
| Negative weights | Cannot handle     | Can handle        |
| Negative cycles  | Undefined         | Can detect        |
| Implementation   | Priority queue    | Simple relaxation |
| Best for         | Non-negative, fast| Negative weights  |
+------------------+-------------------+-------------------+
```

## When to Use Each Algorithm

- **BFS:** Shortest paths in unweighted graphs, level-order traversal, finding
  connected components, testing bipartiteness.

- **DFS:** Cycle detection, topological sorting, finding connected components,
  maze solving, path existence queries.

- **Dijkstra:** Shortest paths in weighted graphs with non-negative edges. The
  standard choice for map routing and network paths.

- **Bellman-Ford:** Shortest paths when negative edge weights exist, or when
  negative cycle detection is needed.

- **Kruskal MST:** Minimum spanning tree when edges are naturally available as a
  list (edge-centric). Works well with sparse graphs.

- **Prim MST:** Minimum spanning tree when starting from a specific vertex
  (vertex-centric). Works well with dense graphs.

- **Topological Sort:** Dependency resolution, build systems, course scheduling,
  any ordering problem on a DAG.

## Real-World Applications

- **GPS navigation:** Dijkstra or A* for finding shortest driving routes.
- **Social networks:** BFS for friend-of-friend suggestions (degrees of separation).
- **Build systems:** Topological sort for determining compilation order.
- **Network design:** MST algorithms for laying cables at minimum cost.
- **Web crawling:** BFS to explore websites level by level from a starting page.
- **Deadlock detection:** Cycle detection in resource allocation graphs.
- **Flight routing:** Bellman-Ford for networks with discount (negative) connections.

## Common Pitfalls and Best Practices

1. **Using Dijkstra with negative weights.** Dijkstra assumes non-negative weights.
   Negative edges produce incorrect results. Use Bellman-Ford instead.

2. **Forgetting to mark vertices as visited.** Both BFS and DFS require tracking
   visited vertices. Missing this causes infinite loops in cyclic graphs.

3. **Confusing directed and undirected cycle detection.** The parent-check method
   only works for undirected graphs. Directed graphs need the coloring approach.

4. **Topological sort on cyclic graphs.** Topological ordering is only defined for
   DAGs. Always check for cycles before attempting topological sort.

5. **Wrong graph representation.** Use adjacency lists for sparse graphs and
   adjacency matrices for dense graphs. The wrong choice wastes time or space.

6. **Not handling disconnected graphs.** BFS and DFS from a single source only reach
   connected vertices. Loop over all vertices to handle disconnected components.

## Practice Problems

1. Implement BFS to find the shortest path between two vertices in an unweighted
   graph. Reconstruct the path using the parent map.

2. Use DFS to determine if an undirected graph is bipartite (two-colorable).

3. Run Dijkstra on a graph with 6 vertices and trace the priority queue state at
   each step. Reconstruct the shortest path tree.

4. Given a directed graph, implement topological sort using Kahn's algorithm
   (BFS-based with in-degree tracking) instead of DFS.

5. Implement Kruskal's algorithm with a union-find data structure. Test it on a
   graph with 7 vertices and 10 edges.

6. Detect if a directed graph contains a cycle using the three-color DFS method.
   If a cycle exists, reconstruct and output the cycle path.

7. Compare the performance of Dijkstra and Bellman-Ford on a graph with 1000
   vertices and 5000 edges (all non-negative weights). Measure the number of
   edge relaxations performed by each.

8. Implement BFS on a grid (2D matrix) to find the shortest path from the top-left
   corner to the bottom-right corner, where some cells are blocked.
