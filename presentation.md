# Graph Data Structures

**Computer Science Fundamentals Series**

Adjacency matrix | Adjacency list | CSR format | DAGs | Property graphs | Graph libraries

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [Graph Terminology](#slide-02--graph-terminology)
2. [Directed & Undirected Graphs](#slide-03--directed--undirected-graphs)
3. [Weighted Graphs](#slide-04--weighted-graphs)
4. [Adjacency Matrix](#slide-05--adjacency-matrix)
5. [Adjacency List](#slide-06--adjacency-list)
6. [Matrix vs List -- Trade-offs](#slide-07--matrix-vs-list--trade-offs)
7. [Edge List & Incidence Matrix](#slide-08--edge-list--incidence-matrix)
8. [Compressed Sparse Row (CSR)](#slide-09--compressed-sparse-row-csr)
9. [Implicit Graphs](#slide-10--implicit-graphs)
10. [Multigraphs & Hypergraphs](#slide-11--multigraphs--hypergraphs)
11. [Bipartite Graphs](#slide-12--bipartite-graphs)
12. [DAGs -- Directed Acyclic Graphs](#slide-13--dags--directed-acyclic-graphs)
13. [Topological Sort](#slide-14--topological-sort)
14. [Graph Storage in Databases](#slide-15--graph-storage-in-databases)
15. [Property Graphs & RDF](#slide-16--property-graphs--rdf)
16. [Graph Libraries & Frameworks](#slide-17--graph-libraries--frameworks)
17. [Choosing the Right Representation](#slide-18--choosing-the-right-representation)
18. [Applications of Graphs](#slide-19--applications-of-graphs)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- Graph Terminology

### Core vocabulary

A graph `G = (V, E)` consists of a set of **vertices** (nodes) and a set of **edges** (links) connecting them.

| Term | Definition |
|------|-----------|
| **Vertex (node)** | A fundamental unit -- represents an entity |
| **Edge (link)** | A connection between two vertices |
| **Degree** | Number of edges incident to a vertex; in-degree and out-degree for directed graphs |
| **Path** | A sequence of vertices where each adjacent pair is connected by an edge |
| **Cycle** | A path that starts and ends at the same vertex |
| **Connected** | Every vertex is reachable from every other vertex (undirected) |
| **Component** | A maximal connected subgraph |

> Graphs are the most general-purpose data structure in computer science. Trees, linked lists, and even arrays can be modelled as special cases of graphs.

---

## Slide 03 -- Directed & Undirected Graphs

### Undirected graphs

Edges have no direction. If `(u, v)` exists, then `(v, u)` is implied. Degree of a vertex counts all incident edges.

- Friendships on a social network -- symmetric by definition
- Road segments that allow traffic in both directions
- Molecular bonds in chemistry

### Directed graphs (digraphs)

Each edge has a source and a target. `(u, v)` does not imply `(v, u)`. Each vertex has an **in-degree** and an **out-degree**.

- Twitter follows -- Alice follows Bob does not mean Bob follows Alice
- Web hyperlinks -- page A links to page B
- Dependencies -- module A depends on module B

> Most real-world graphs are directed. Even "undirected" relationships are often stored as two directed edges for implementation simplicity.

---

## Slide 04 -- Weighted Graphs

### Adding numeric labels to edges

A **weighted graph** assigns a value (weight, cost, capacity) to each edge. Weights can represent distance, latency, bandwidth, probability, or any domain-specific metric.

```
   A --5-- B
   |       |
   2       3
   |       |
   C --7-- D --1-- E
```

### Where weights matter

- **Shortest path** -- Dijkstra, Bellman-Ford, A* all require edge weights
- **Minimum spanning tree** -- Kruskal, Prim select lowest-weight edges
- **Network flow** -- edge weights represent capacities
- **Negative weights** -- Bellman-Ford handles them; Dijkstra does not

> An unweighted graph is a weighted graph where every edge has weight 1. The distinction matters for algorithm selection and storage format.

---

## Slide 05 -- Adjacency Matrix

### 2D array representation

For a graph with `n` vertices, allocate an `n x n` matrix `M`. `M[i][j] = 1` if edge `(i, j)` exists, `0` otherwise. For weighted graphs, store the weight instead of 1.

```
Vertices: A=0, B=1, C=2, D=3

    A  B  C  D
A [ 0  1  1  0 ]
B [ 1  0  0  1 ]
C [ 1  0  0  1 ]
D [ 0  1  1  0 ]
```

### Characteristics

| Property | Value |
|----------|-------|
| **Space** | `O(V^2)` -- regardless of edge count |
| **Edge lookup** | `O(1)` -- direct array access |
| **Add edge** | `O(1)` |
| **Iterate neighbours** | `O(V)` -- must scan entire row |
| **Add vertex** | `O(V^2)` -- reallocate matrix |

> Best for dense graphs where `|E|` approaches `|V|^2`. Wastes memory on sparse graphs -- a 10,000-node social graph with average degree 50 uses 100M entries but only 500K are non-zero.

---

## Slide 06 -- Adjacency List

### Array of neighbour lists

Each vertex stores a list (array, linked list, or hash set) of its neighbours. The most common representation in practice.

```
A -> [B, C]
B -> [A, D]
C -> [A, D]
D -> [B, C]
```

### Characteristics

| Property | Value |
|----------|-------|
| **Space** | `O(V + E)` -- proportional to actual graph size |
| **Edge lookup** | `O(degree)` -- scan neighbour list (or `O(1)` with hash set) |
| **Add edge** | `O(1)` -- append to list |
| **Iterate neighbours** | `O(degree)` -- direct traversal |
| **Add vertex** | `O(1)` -- append new list |

> Default choice for most graph algorithms. BFS and DFS run in `O(V + E)` with adjacency lists -- optimal. With an adjacency matrix they degrade to `O(V^2)`.

---

## Slide 07 -- Matrix vs List -- Trade-offs

### Decision matrix

| Criterion | Adjacency Matrix | Adjacency List |
|-----------|-----------------|---------------|
| Space complexity | `O(V^2)` | `O(V + E)` |
| Edge existence check | `O(1)` | `O(degree)` |
| Iterate all neighbours | `O(V)` | `O(degree)` |
| Add/remove edge | `O(1)` | `O(1)` add / `O(degree)` remove |
| Dense graph performance | Excellent | Wasteful pointer overhead |
| Sparse graph performance | Wasteful memory | Excellent |
| Cache locality | Good (contiguous) | Poor (pointer chasing) |
| Matrix operations | Natural (multiply, transpose) | Requires conversion |

### When to prefer the matrix

- Dense graphs (`|E| > |V|^2 / 4`)
- Frequent edge-existence queries
- Matrix algebra on graphs (spectral methods, PageRank power iteration)
- Small graphs (under ~1000 vertices)

### When to prefer the list

- Sparse graphs (most real-world graphs)
- Graph traversal algorithms (BFS, DFS, Dijkstra)
- Dynamic graphs with frequent vertex additions
- Memory-constrained environments

---

## Slide 08 -- Edge List & Incidence Matrix

### Edge list

Store edges as a flat list of `(source, target, weight)` tuples. Simplest possible representation.

```
[(A,B,5), (A,C,2), (B,D,3), (C,D,7), (D,E,1)]
```

- **Space:** `O(E)`
- **Edge lookup:** `O(E)` -- linear scan
- **Best for:** Kruskal's algorithm (sort edges by weight), input parsing, serialisation

### Incidence matrix

An `|V| x |E|` matrix. Column `j` has `+1` at source vertex and `-1` at target vertex (directed), or `1` at both endpoints (undirected).

```
        e1  e2  e3  e4
   A [   1   1   0   0 ]
   B [   1   0   1   0 ]
   C [   0   1   0   1 ]
   D [   0   0   1   1 ]
```

- **Space:** `O(V * E)` -- usually larger than both matrix and list
- **Use case:** theoretical analysis, network flow formulations, electrical circuit modelling

> Edge lists are underrated for batch processing. Many distributed graph frameworks (Spark GraphX) use edge-list partitioning internally.

---

## Slide 09 -- Compressed Sparse Row (CSR)

### Cache-friendly sparse storage

CSR stores a sparse adjacency matrix using three flat arrays, eliminating pointer overhead and improving cache locality.

```
Graph:  0 -> [1,3]   1 -> [2]   2 -> [3]   3 -> []

row_ptr:  [0, 2, 3, 4, 4]    -- start index in col_idx for each vertex
col_idx:  [1, 3, 2, 3]       -- concatenated neighbour lists
values:   [1, 1, 1, 1]       -- edge weights (optional)
```

### Structure

| Array | Size | Purpose |
|-------|------|---------|
| `row_ptr` | `V + 1` | Index into `col_idx` where each vertex's neighbours begin |
| `col_idx` | `E` | Concatenated sorted neighbour indices |
| `values` | `E` | Edge weights (omit for unweighted) |

### Characteristics

- **Space:** `O(V + E)` with minimal overhead -- no pointers, no linked-list nodes
- **Neighbour iteration:** `col_idx[row_ptr[v] .. row_ptr[v+1]]` -- contiguous memory scan
- **Edge lookup:** binary search within row segment -- `O(log degree)`
- **Weakness:** static structure -- inserting edges requires full rebuild

> CSR is the standard format for high-performance graph libraries (Boost Graph, SuiteSparse, cuGraph) and GPU graph processing. Its cousin CSC (Compressed Sparse Column) enables efficient in-neighbour access.

---

## Slide 10 -- Implicit Graphs

### Graphs without explicit storage

An implicit graph generates its vertices and edges on demand via a function rather than storing them in memory. The graph exists logically but is never fully materialised.

### Examples

- **Grid / lattice** -- vertex `(r, c)` connects to `(r+/-1, c)` and `(r, c+/-1)`. An `n x n` grid has `n^2` vertices but you never allocate the adjacency structure.
- **State-space search** -- puzzle states (Rubik's cube, 15-puzzle) are vertices; legal moves are edges. The graph has billions of nodes but BFS explores only a fraction.
- **Game trees** -- chess positions are vertices; legal moves are edges. Fully materialising the tree is impossible (`~10^47` positions).
- **Procedural generation** -- infinite mazes, terrain graphs generated from a seed function.

### Neighbour function pattern

```python
def neighbours(state):
    for move in legal_moves(state):
        yield apply(move, state)
```

> If your graph has more than `~10^8` vertices, explicit storage is impractical. Implicit representation is the only option -- combined with BFS, DFS, or A*.

---

## Slide 11 -- Multigraphs & Hypergraphs

### Multigraphs

Allow **multiple edges** (parallel edges) between the same pair of vertices, and optionally **self-loops**.

- Flight routes -- multiple airlines between the same cities
- Communication networks -- multiple links between routers
- Representation: adjacency list with edge IDs, or edge list with duplicates

```
A ==[flight_101]==> B
A ==[flight_202]==> B
A ==[flight_303]==> B
```

### Hypergraphs

A **hyperedge** connects an arbitrary number of vertices (not just two). Generalises the concept of an edge.

- Database schemas -- a table (hyperedge) relates multiple columns (vertices)
- Co-authorship networks -- a paper (hyperedge) connects all its authors
- VLSI design -- a net (hyperedge) connects multiple pins

> Standard graph algorithms do not apply directly to hypergraphs. Common approach: expand to a bipartite graph where one partition is vertices and the other is hyperedges.

---

## Slide 12 -- Bipartite Graphs

### Structure

A graph whose vertices can be divided into two disjoint sets `U` and `V` such that every edge connects a vertex in `U` to a vertex in `V`. No edge connects two vertices within the same set.

```
  U          V
  o ------- o
  o ------- o
  o ------- o
```

### Detection algorithm

A graph is bipartite if and only if it contains **no odd-length cycle**. Detection via BFS 2-colouring:

1. Pick any unvisited vertex, colour it **red**
2. Colour all neighbours **blue**
3. Colour their uncoloured neighbours **red**
4. If a neighbour already has the same colour -- **not bipartite**
5. Repeat for disconnected components

Time complexity: `O(V + E)`

### Applications

- **Job matching** -- workers (U) to tasks (V); Hungarian algorithm, Hopcroft-Karp
- **Recommendation engines** -- users (U) to items (V)
- **Course scheduling** -- time slots (U) to courses (V)
- **Hypergraph expansion** -- every hypergraph has a natural bipartite representation

---

## Slide 13 -- DAGs -- Directed Acyclic Graphs

### Definition and properties

A **directed acyclic graph** is a directed graph with no cycles. Every DAG has at least one **topological ordering** -- a linear sequence where every edge goes from earlier to later.

### Key properties

- At least one vertex with in-degree 0 (source) and one with out-degree 0 (sink)
- Longest path is finite and computable in `O(V + E)`
- Number of paths between two vertices can be exponential
- Transitive closure and transitive reduction are well-defined

### Why DAGs matter

| Application | Vertices | Edges |
|-------------|----------|-------|
| Build systems (Make, Bazel) | Tasks / targets | Dependencies |
| Package managers (npm, pip) | Packages | Version dependencies |
| Data pipelines (Airflow) | Processing stages | Data flow |
| Git commit history | Commits | Parent pointers |
| Spreadsheet formulas | Cells | Cell references |
| Neural network layers | Layers / ops | Data flow |

> Cycle detection: run DFS and check for **back edges** (edge to a vertex currently on the recursion stack). If found, the graph has a cycle and is not a DAG.

---

## Slide 14 -- Topological Sort

### Linearising a DAG

Topological sort produces an ordering of vertices such that for every directed edge `(u, v)`, vertex `u` appears before `v`. Only possible on DAGs.

### Kahn's algorithm (BFS-based)

```
1. Compute in-degree for every vertex
2. Enqueue all vertices with in-degree 0
3. While queue is not empty:
   a. Dequeue vertex u, append to result
   b. For each neighbour v of u:
      - Decrement in-degree of v
      - If in-degree of v becomes 0, enqueue v
4. If result length != |V|, graph has a cycle
```

Time: `O(V + E)` -- Space: `O(V)`

### DFS-based approach

```
1. Run DFS from each unvisited vertex
2. On finishing a vertex (all descendants explored),
   push it onto a stack
3. Pop the stack to get topological order
```

> Multiple valid orderings may exist. Kahn's algorithm can be adapted to find **all** topological orderings or to detect the unique ordering (when the queue always has exactly one element).

---

## Slide 15 -- Graph Storage in Databases

### Native graph databases

Store vertices and edges as first-class objects with index-free adjacency -- each node physically points to its neighbours. Traversal does not require index lookups.

- **Neo4j** -- property graph model, Cypher query language
- **Amazon Neptune** -- supports both property graph (Gremlin) and RDF (SPARQL)
- **TigerGraph** -- massively parallel, GSQL language

### Relational graph storage

Graphs can be stored in an RDBMS using vertex and edge tables:

```sql
CREATE TABLE vertices (
  id    INTEGER PRIMARY KEY,
  label VARCHAR(64),
  props JSONB
);

CREATE TABLE edges (
  src    INTEGER REFERENCES vertices(id),
  dst    INTEGER REFERENCES vertices(id),
  label  VARCHAR(64),
  weight FLOAT,
  PRIMARY KEY (src, dst, label)
);
```

### Trade-offs

| Feature | Native graph DB | Relational |
|---------|----------------|-----------|
| Multi-hop traversal | Fast (pointer chasing) | Slow (recursive JOINs) |
| Ad-hoc analytics | Limited | Excellent (SQL) |
| ACID transactions | Varies | Mature |
| Ecosystem / tooling | Smaller | Enormous |

---

## Slide 16 -- Property Graphs & RDF

### Property graph model

Vertices and edges both carry **labels** and arbitrary **key-value properties**. The dominant model in industry (Neo4j, TigerGraph, Amazon Neptune).

```
(:Person {name:"Alice", age:30})
  -[:FOLLOWS {since:2023}]->
(:Person {name:"Bob", age:28})
```

Query language: **Cypher** (Neo4j), **Gremlin** (Apache TinkerPop), **GQL** (ISO standard, 2024).

### RDF (Resource Description Framework)

Data as **triples**: `(subject, predicate, object)`. Every entity is a URI. The W3C standard for the Semantic Web and knowledge graphs.

```
<:Alice>  <:follows>  <:Bob> .
<:Alice>  <:age>      "30"^^xsd:integer .
<:Bob>    <:worksAt>  <:Acme> .
```

Query language: **SPARQL**

### Comparison

| Aspect | Property Graph | RDF |
|--------|---------------|-----|
| Schema | Optional labels + constraints | Ontologies (OWL, RDFS) |
| Query | Cypher / Gremlin / GQL | SPARQL |
| Strength | Application development | Data integration, linked data |
| Ecosystem | Neo4j, TigerGraph, Memgraph | Wikidata, DBpedia, knowledge graphs |

---

## Slide 17 -- Graph Libraries & Frameworks

### Python

- **NetworkX** -- pure Python, rich algorithm library, excellent for prototyping. Not performant on large graphs (>100K edges).
- **igraph** -- C core with Python bindings. Much faster than NetworkX for large graphs.
- **graph-tool** -- C++ core, Boost Graph under the hood. Best performance in the Python ecosystem.

### Java / JVM

- **JGraphT** -- pure Java, extensive algorithm collection, well-documented.
- **Apache TinkerPop / Gremlin** -- graph traversal framework, vendor-neutral API for graph databases.
- **Neo4j Java Driver** -- native access to Neo4j from JVM languages.

### C++

- **Boost Graph Library (BGL)** -- generic, header-only, STL-style. Industry standard for high-performance graph algorithms.
- **LEMON** -- lightweight, efficient graph library with LP solver integration.

### Distributed / Large-scale

- **Apache Spark GraphX** -- distributed graph processing on Spark RDDs.
- **Pregel / Apache Giraph** -- vertex-centric BSP model for billion-edge graphs.
- **NVIDIA cuGraph** -- GPU-accelerated graph analytics (CSR-based).

---

## Slide 18 -- Choosing the Right Representation

| Scenario | Recommended representation | Reason |
|----------|--------------------------|--------|
| Sparse, dynamic graph (social network) | **Adjacency list** (hash map) | `O(V + E)` space, fast traversal, easy to add/remove |
| Dense graph or matrix algebra needed | **Adjacency matrix** | `O(1)` edge lookup, natural for spectral methods |
| Static analysis, HPC, GPU | **CSR / CSC** | Cache-friendly, minimal memory, vectorisable |
| Edge-centric algorithms (Kruskal) | **Edge list** | Sort by weight, union-find |
| Enormous state space (AI search) | **Implicit graph** | Generate on demand, never materialise |
| Persistent storage with queries | **Property graph DB** | Index-free adjacency, Cypher/Gremlin |
| Linked data / knowledge graph | **RDF triple store** | Standards-based, federated queries |
| Multiple edge types between same nodes | **Multigraph adjacency list** | Edge IDs distinguish parallel edges |

> Start with an adjacency list. Move to CSR when profiling shows memory or cache bottlenecks. Move to a graph database when you need persistence and multi-hop queries at scale.

---

## Slide 19 -- Applications of Graphs

### Social networks

Vertices are users; edges are follows, friendships, or interactions. Graph algorithms power friend suggestions (common neighbours), influence scoring (PageRank), and community detection (Louvain).

### Road maps and navigation

Intersections are vertices; road segments are weighted edges (distance, travel time). Dijkstra, A*, and contraction hierarchies enable real-time routing in Google Maps, Waze, and OSRM.

### Dependency graphs

Packages, modules, or build targets as vertices; import/require relationships as edges. Topological sort determines build order; cycle detection prevents circular dependencies.

### Knowledge graphs

Entities and relationships modelled as a graph -- Google Knowledge Graph, Wikidata, enterprise knowledge management. Power semantic search, question answering, and recommendation systems.

### Other domains

- **Compiler optimisation** -- control flow graphs, data flow graphs, SSA form
- **Bioinformatics** -- protein interaction networks, metabolic pathways, genome assembly (de Bruijn graphs)
- **Fraud detection** -- transaction graphs reveal suspicious patterns via subgraph matching
- **Circuit design** -- components are vertices, wires are edges; placement and routing as graph problems

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- A graph `G = (V, E)` is the most flexible data structure -- trees, lists, and matrices are all special cases
- Adjacency list is the default choice for most applications -- `O(V + E)` space and optimal traversal
- Adjacency matrix excels at dense graphs and matrix algebra but wastes memory on sparse graphs
- CSR format is essential for high-performance and GPU graph processing -- cache-friendly with minimal overhead
- DAGs underpin build systems, data pipelines, version control, and scheduling
- Property graph databases (Neo4j) and RDF triple stores serve different graph persistence needs
- Choose representation based on graph density, mutation frequency, and algorithm requirements

### Recommended reading

| Source | Description |
|--------|------------|
| **Cormen et al.** | *Introduction to Algorithms* (CLRS) -- chapters 22-26 cover graph algorithms comprehensively |
| **Skiena** | *The Algorithm Design Manual* -- excellent practical graph problem taxonomy |
| **Needham & Hodler** | *Graph Algorithms* (O'Reilly) -- practical guide using Neo4j and Apache Spark |
| **NetworkX Docs** | [networkx.org](https://networkx.org) -- Python graph library with tutorial and algorithm reference |
| **Boost Graph Docs** | [boost.org/libs/graph](https://www.boost.org/doc/libs/release/libs/graph/doc/) -- C++ generic graph library |
| **Neo4j GraphAcademy** | [graphacademy.neo4j.com](https://graphacademy.neo4j.com) -- free courses on graph databases and Cypher |
