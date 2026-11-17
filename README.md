# Group_2_Graph-Theory_Network_Flow

## Group Members

| No. | Name | NRP |
|-----|------|-----|
| 1 | Ahmad Farras Favian Al Efasi | 5025251005 |
| 2 | Daniel Pedrosaputra | 5025251171 |

## Algorithm Introduction

This homework covers the **Network Flow** topic of the lecture. The task is to implement two maximum flow algorithms other than Ford-Fulkerson, and to test them on the lecture example and on other cases with a dynamic network size.

A network is a directed graph with one source (S), one sink (T) and a capacity `c(u, v)` on every edge. The goal is to find the largest possible flow from S to T, without exceeding any capacity and without losing flow at the intermediate nodes (flow conservation). Both algorithms below work on the **residual network** (remaining capacity of every edge plus a reverse edge that allows flow to be cancelled), the same idea as in the lecture slides.

### Dinic's Algorithm

Dinic's algorithm improves the "find an augmenting path, push flow, repeat" idea by working in **phases**. In every phase it builds a *level graph* with BFS (the level of a node is its distance from the source in the residual network) and then pushes a *blocking flow*: flow that only moves from level `k` to level `k+1`, repeated until no such path is left. Because the distance from S to T increases after every phase, there are at most V phases, which gives a running time of **O(V²E)**.

The algorithm relies on the following ideas:

- **Residual network**: edges with remaining capacity, plus reverse edges holding the flow already sent.
- **Level graph**: the residual network restricted to edges that go from one BFS level to the next.
- **Blocking flow**: a flow in the level graph in which every S-T path contains at least one saturated edge.
- **Termination**: when BFS can no longer reach T, no augmenting path exists, so the flow is maximum.

### Push-Relabel Algorithm (FIFO version)

Push-Relabel takes a different approach: it never looks for whole S-T paths. Instead it first **floods** the network by saturating every edge leaving the source, which creates *excess* (more flow entering a node than leaving it). It then repeatedly moves excess locally, one edge at a time, guided by a *height* label on every node, until only the source and the sink hold excess. The FIFO version used here processes active nodes in a queue and runs in **O(V³)**.

The algorithm relies on the following ideas:

- **Preflow**: a flow where nodes may receive more than they send. The surplus at a node is its excess.
- **Height (label)**: starts at 0 for every node except the source, which starts at V. Flow may only be pushed *downhill*.
- **Push**: if `u` has excess and an edge `u -> v` with residual capacity where `height[u] = height[v] + 1`, send `min(excess, residual)` along it.
- **Relabel**: if `u` has excess but no admissible edge, raise `height[u]` to `1 + min(height of neighbours reachable by a residual edge)`.
- **Termination**: when no node other than S and T has excess. Excess that cannot reach T is sent back to S (nodes get height above V), so the preflow becomes a valid maximum flow.

Both algorithms also report the **minimum cut** (nodes reachable from S in the final residual network), so the result can be checked with the **Max-Flow Min-Cut Theorem** from the lecture: the maximum flow value equals the minimum cut value.

## Steps

### Dinic's Algorithm

1. Build the residual network (every edge gets a reverse edge with capacity 0).
2. Run BFS from the source on edges with residual capacity > 0 and give every node a level.
3. If the sink was not reached, stop. The current flow is the maximum flow.
4. Run DFS from the source, moving only from level `k` to level `k+1` along edges with residual capacity > 0. Each time the sink is reached, push the bottleneck amount along the path: decrease the residual capacity of the forward edges and increase it on the reverse edges.
5. Keep a pointer per node so that dead-end edges are not tried again in the same phase. Repeat step 4 until no more flow can be pushed (a blocking flow).
6. Add the flow of the phase to the total and go back to step 2.

### Push-Relabel Algorithm

1. Set `height[source] = V` and all other heights to 0, all excess to 0.
2. Saturate every edge leaving the source. The neighbours of the source now have excess and become *active* (put them in a FIFO queue).
3. Take the first active node `u` from the queue and *discharge* it: while `u` has excess, look at its edges one by one.
4. If an edge `u -> v` has residual capacity and `height[u] = height[v] + 1`, **push** `min(excess[u], residual)`; add `v` to the queue if it becomes active (it is not the source or sink).
5. If all edges of `u` were tried and `u` still has excess, **relabel** it: `height[u] = 1 + min(height[v])` over residual edges, and start again from its first edge.
6. Repeat from step 3 until the queue is empty. The excess at the sink is the maximum flow value.

## Implementations

Both programs are implemented in Python. The network size is not fixed: the programs read the number of nodes and edges from the input, and node names can be any word (for example `S`, `A`, `v1`).

`network.py` contains the parts shared by both algorithms. The `parse_network()` function reads the input while ignoring comments, checks that the capacities are non-negative integers, merges parallel edges and builds the residual graph (`FlowGraph`), where edge `2k` is a real edge and edge `2k+1` is its reverse. The `min_cut()` function finds the source side of a minimum cut with a BFS on the final residual graph, and `print_result()` prints the flow on every edge, the minimum cut and the maximum flow.

In `dinic.py`, `build_levels()` is the BFS that assigns levels, `send_flow()` is the DFS that pushes flow only to the next level and uses the pointer array to skip dead ends, and `dinic()` repeats the phases until the sink is unreachable. The program prints the level graph of every phase and every blocking flow path.

In `push_relabel.py`, `push_relabel()` first saturates the source edges, then keeps a FIFO queue of active nodes. Each node is discharged with the push and relabel rules described above. The program prints every push and relabel operation.

The implementations are verified in `test_maxflow.py` against the lecture example (maximum flow 6), the bipartite matching example from the lecture (2), a larger textbook network (23), and 300 random networks compared with an independent BFS-based reference. For every result the tests also check that the flow respects capacities and flow conservation, and that the minimum cut value equals the maximum flow.

## How to Run & Prerequisites

Before running the programs, make sure the following requirements are met:

- **Python 3**: Installed on your computer.
- **Code Editor (Optional)**: Visual Studio Code, PyCharm, or another editor for viewing and modifying the source code.
- **Project Files**: The repository cloned or extracted, with all files kept in the same directory:

```
├── network.py
├── dinic.py
├── push_relabel.py
├── test_maxflow.py
├── sample_input.txt
├── sample_input_2.txt
└── sample_input_3.txt
```

- **Third-Party Libraries**: None required, as the programs use standard Python libraries only.

## Running the Program

1. **Get the project files.** Clone the repository or extract the project files into a directory on your computer.
2. **Open the project directory.** Open the project folder using a terminal or command prompt.
3. **Run the program.** Execute one of the following commands to read the network from a file:

```
python dinic.py sample_input.txt
python push_relabel.py sample_input.txt
```

If the commands above do not work, try `python3` instead of `python`.

4. **Or type the network in the terminal.** Run the program without a file name, then enter the network. Finish the input with `Ctrl + D` (Linux/macOS) or `Ctrl + Z` then Enter (Windows):

```
python dinic.py
```

5. **Quiet mode.** Add `-q` to hide the step-by-step log and only print the final result:

```
python dinic.py sample_input.txt -q
```

6. **View the result.** The program prints the steps of the algorithm, the flow on every edge, the minimum cut and the maximum flow.

## Example Input

The first line contains the **source** and the **sink**. Every following line is one directed edge in the form `u v capacity`. Text after `#` is ignored. The following example is the network from the lecture (slide 4):

```
S T
S A 5
S B 2
A C 2
A B 2
B D 4
D C 1
C T 3
D T 5
```

## Running the Tests

The project includes `test_maxflow.py`. To execute the tests, run the following command from the project directory:

```
python -m unittest -v test_maxflow.py
```

The tests check the lecture example (6), the bipartite matching example (2), a larger network (23), a network with no path from source to sink, parallel and antiparallel edges, a single edge, a zero-capacity edge, invalid input, and 300 random networks whose results are compared with a reference implementation. Both algorithms are run on every case, and all 9 tests pass.

## Sample Run Result

### Example 1: Lecture Network (Dinic's Algorithm)

Command:

```
python dinic.py sample_input.txt
```

Output:

```
Dinic's Algorithm: 6 nodes, source = S, sink = T

Phase 1: level graph  S=0, A=1, B=1, C=2, D=2, T=3
  blocking flow path S -> A -> C -> T   (+2)
  blocking flow path S -> B -> D -> T   (+2)
  phase 1 pushed 4, total flow = 4

Phase 2: level graph  S=0, A=1, B=2, C=4, D=3, T=4
  blocking flow path S -> A -> B -> D -> T   (+2)
  phase 2 pushed 2, total flow = 6

BFS cannot reach T anymore -> no augmenting path left, flow is maximum.

Flow on every edge (flow/capacity)
----------------------------------
S -> A : 4/5
S -> B : 2/2
A -> C : 2/2
A -> B : 2/2
B -> D : 4/4
D -> C : 0/1
C -> T : 2/3
D -> T : 4/5

Minimum cut (Max-Flow Min-Cut Theorem)
--------------------------------------
P  = {S, A}
P' = {B, C, D, T}
Cut edges: S->B (2), A->C (2), A->B (2)
Cut value: 6

Maximum flow from S to T: 6
```

### Example 2: Lecture Network (Push-Relabel Algorithm)

Command:

```
python push_relabel.py sample_input.txt
```

Output:

```
Push-Relabel Algorithm: 6 nodes, source = S, sink = T

Initial: height[S] = 6, saturate all edges leaving S
  push 5 from S to A
  push 2 from S to B
  relabel A: height 0 -> 1
  push 2 from A to C
  push 2 from A to B
  relabel A: height 1 -> 7
  push 1 from A to S
  relabel B: height 0 -> 1
  push 4 from B to D
  relabel C: height 0 -> 1
  push 2 from C to T
  relabel D: height 0 -> 1
  push 4 from D to T

No active node left. pushes = 8, relabels = 5

Flow on every edge (flow/capacity)
----------------------------------
S -> A : 4/5
S -> B : 2/2
A -> C : 2/2
A -> B : 2/2
B -> D : 4/4
D -> C : 0/1
C -> T : 2/3
D -> T : 4/5

Minimum cut (Max-Flow Min-Cut Theorem)
--------------------------------------
P  = {S, A}
P' = {B, C, D, T}
Cut edges: S->B (2), A->C (2), A->B (2)
Cut value: 6

Maximum flow from S to T: 6
```

Node A receives 5 but can only send 4 onward (A->C 2 + A->B 2), so it is relabeled above the source (height 7) and returns the 1 unit of excess to S.

### Example 3: Maximum Bipartite Matching as Max Flow (Dinic's Algorithm)

The bipartite graph from the lecture (V1 = {a, b}, V2 = {c, d}) is converted into a network by adding a source `s` and a sink `t`, with capacity 1 on every edge.

Command:

```
python dinic.py sample_input_2.txt
```

Input (`sample_input_2.txt`):

```
s t
s a 1
s b 1
a c 1
a d 1
b c 1
c t 1
d t 1
```

Output:

```
Dinic's Algorithm: 6 nodes, source = s, sink = t

Phase 1: level graph  s=0, a=1, b=1, c=2, d=2, t=3
  blocking flow path s -> a -> c -> t   (+1)
  phase 1 pushed 1, total flow = 1

Phase 2: level graph  s=0, a=3, b=1, c=2, d=4, t=5
  blocking flow path s -> b -> c -> a -> d -> t   (+1)
  phase 2 pushed 1, total flow = 2

BFS cannot reach t anymore -> no augmenting path left, flow is maximum.

Flow on every edge (flow/capacity)
----------------------------------
s -> a : 1/1
s -> b : 1/1
a -> c : 0/1
a -> d : 1/1
b -> c : 1/1
c -> t : 1/1
d -> t : 1/1

Minimum cut (Max-Flow Min-Cut Theorem)
--------------------------------------
P  = {s}
P' = {a, b, c, d, t}
Cut edges: s->a (1), s->b (1)
Cut value: 2

Maximum flow from s to t: 2
```

The edges with flow 1 between V1 and V2 give the maximum matching **a-d, b-c**, the same result as the lecture. In phase 2 the path `s -> b -> c -> a -> d -> t` goes through the reverse edge `c -> a`, which cancels the first choice `a -> c`.

### Example 4: Larger Network (Both Algorithms)

Command:

```
python dinic.py sample_input_3.txt
python push_relabel.py sample_input_3.txt -q
```

Input (`sample_input_3.txt`):

```
s t
s v1 16
s v2 13
v1 v3 12
v2 v1 4
v2 v4 14
v3 v2 9
v3 t 20
v4 v3 7
v4 t 4
```

Output (Dinic):

```
Dinic's Algorithm: 6 nodes, source = s, sink = t

Phase 1: level graph  s=0, v1=1, v2=1, v3=2, v4=2, t=3
  blocking flow path s -> v1 -> v3 -> t   (+12)
  blocking flow path s -> v2 -> v4 -> t   (+4)
  phase 1 pushed 16, total flow = 16

Phase 2: level graph  s=0, v1=1, v2=1, v3=3, v4=2, t=4
  blocking flow path s -> v2 -> v4 -> v3 -> t   (+7)
  phase 2 pushed 7, total flow = 23

BFS cannot reach t anymore -> no augmenting path left, flow is maximum.

Minimum cut (Max-Flow Min-Cut Theorem)
--------------------------------------
P  = {s, v1, v2, v4}
P' = {v3, t}
Cut edges: v1->v3 (12), v4->v3 (7), v4->t (4)
Cut value: 23

Maximum flow from s to t: 23
```

Push-Relabel returns the same maximum flow (**23**) and the same minimum cut value (**23**).

These examples show that both algorithms reach the same maximum flow as the Max-Flow Min-Cut Theorem predicts. The lecture network gives 6 with a minimum cut of 6, matching the slides.
