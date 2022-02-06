# 4. Scalable Distributed Topologies

## 4.1. Graphs

G(V,E): set of Vertices and a set of Edges that connect pairs of vertices. For instance, a path G(V={a,b,c,d}, E={(a,b),(b,c),(c,d)} can be made into a ring by adding one edge connecting its ends E = E ∪ {(d,a)}.
- Graphs can be directed or undirected (edges uni or bi-directional);
- Having an edge(x,y), vertices x and y are adjacent (or neighbours);
- Path is a sequence of vertices v1,v2,...,vi,vi+1, with edges connecting them (vi,vi+1) ∈ E.

Definitions:
- Walk: Edges and vertices can be repeated;
- Trail: Only vertices can be repeated;
- Path: No repeated vertices or edges.
- Connected Component: maximal connected subgraph of G.
- Degree of vi: number of adjacent vertices to vi. In directed graphs there is in-degree and out-degree.
- Distance d(vi,vj): length of the shortest path connecting those nodes.
- Eccentricity of vi: ecc(vi) = max({d(vi,vj)|vj ∈ V}).
- Diameter: D = max({ecc(vi)|vi ∈ V}).
- Radius. R = min({ecc(vi)|vi ∈ V}).
- Center: {vi|ecc(vi) == R}.
- Periphery: {vi|ecc(vi) == D}.

Topologies:
- Simple Graph: undirected graph with no loops and no more than 1 edge between 2 different vertices.
- Weighted Graph is obtained by assigning a weight to each edge.
- Complete Graph: each pair of vertices has an edge connecting them. In a sub-graph we might find a clique with that property.
- Connected Graph: there is a path between any two nodes.
- Star: a "central" vertice and many leaf nodes connected to center.
- Tree: connected graph with no cycles.
- Planar Graph: vertices and edges can be drawn in a plane and no 2 edges intersect (ex: rings and trees).
- Random Geometric: vertices are dropped randomly uniformly into a unit square and adding edges to connect any 2 points within a given euclidean distance.
- Random Erdos-Renyi: G(n,p) model, n nodes are connected randomly. Each edge is included with independent probability p.
- Watts-Strogatz model (later on small world constructions).
- Barabasi-Albert model: preferential attachment: the more connected a node is, the more likely it is to receive new links. Degree Distribution follows a power law.

In a network context, graphs with cycles allow multi-path rounting. Can be robust but data handling can become more complex. Distributed algorithms construct trees to avoid cycles, while others try to work under multi-path.

## 4.2. Spanning Trees

- Directed Graph: strongly connected if for every pair of vertices u and v there is a path from u to v and a path from v to u.
- Distance from u to v is the length of the shortest path from u to v.
- Directed spanning tree with root node i is breadth first provided that each node at distance d from i in the graph appears at depth d in the tree.
- Every strongly connected graph has a breadth-first directed spanning tree.

### 4.2.1. Synchronous SyncBFS

Synchronous SyncBFS Algorithm: processes communicate over directed edges. Unique UIDs are available, but network diameter and size is unknown.
- Initial state:
  - parant = nil
  - marked = False (True in root node i0);
- SyncBFS Algorithm:
  - process i0 sends a search message in round 1;
  - unmarked processes receiving a search message from x do marked = True and set parent = x, in the next round search messages are sent from these processes.
- Complexity:
  - Time: at most diam rounds (depending on i0 eccentricity);
  - Message: |E|. Messages are sent across all edges E.
- Child Pointers:
  - If parents need to know their offspring, processes must reply to search messages with either parent or nonparent;
  - This is only easy if graph is undirected, but is achievable in general strongly connected graphs.
- Termination: making i0 know that the tree is constructed:
  - All processes respond with parent or nonparent;
  - Parent terminates when all children terminate;
  - Responses are collected from leaves to tree root.

Applications of Breadth First Spanning Trees:
- Aggregation (Global Computation): Input values in each process can be aggregated towards a sync node. Each value only contributes once, many functions can be used: Sums, Averages, Max, Voting.
- Leader Election: Largest UID wins. All process become root of their own tree and aggregate a Max(UID). Each decide by comparing their own UID with Max(UID).
- Broadcast: Message payload m can be attached to SyncBFS construction (m|E|message load) or broadcasted once tree is formed (m|V|message load).
- Computing Diameter: Each process constructs a SyncBFS. Then determines maxdist, longest tree path. Afterwards, all processes use their trees to aggregate max(maxdist) from all roots/nodes. Complexity: Time O(diam) and messages O(diam x |E|).

### 4.2.2. AsynchSpaningTree

Asynchronous network model, avoiding (for now) useful abstractions like logical time and global snapshots.

The lack of helping tools is compensated by a "generous" system model:
- Faults: no faults;
- Channels: Reliable FIFO send/receive channels:
  - Automaton:
    - Signature:
      - Input: send(m)i,j, m ∈ M
      - Output: receive(m)i,j, m ∈ M
    - States:
      - queue = {}
    - Transitions:
      - send(m)i,j -> queue := queue + {m};
      - receive(m)i,j -> queue.pophead() (precondition: queue.head = m).
  - Allowed trace behaviour: let β be a sequence of actions, and cause() a function mapping each receive event e ∈ β |receive to a preceding send event s ∈ β |send such that:
    - ∀receive(x) ∈ β |receive : cause(receive(x)) = send(y) ⇒ x = y. Messages don't come out of the blue.
    - cause() is surjective. For every send there is a mapped receive. Messages are not lost.
    - cause() is injective. For every receive there is a distinct send. Messages are not duplicated.
    - receive <β receive' => cause(receive) <β cause(receive'). Order is preserved.
- AsynchSpanningTree:
  - Signature:
    - Input: receive("search")i,j
    - Output: send("search")i,j
  - Transitions:
    - send("search")i,j
      - Preconditon: sendto(j) = yes
      - Effect: sendto(j) = no
    - receive("search")j,i
      - Effect: if i != i0 and parent = null then
        - parent := j
        - for all k ∈ nbrs \ {j} do
          - sendto(k) = yes
  - Channel automaton: consumes send("search")f,t, and produces receive("search")f,t in a reliable FIFO order.
- Invariant: A tree is gradually formed: in any reachable state, the edges defined by all parent variables form a spanning tree of a subgraph of G, containing i0; moreover, if there is a message in any channel C(i,j) then i is in this spanning tree.
- Invariant: all contacts are searched: in any reachable state, if i = i0 or parent != null, and if j ∈ nbrs \ {i0}, then either parent  j (!= null or Ci,j contains a search message or sendto(j), is yes.
- Theorem: AsynchSpanningTree algorithm constructs a spanning tree in the undirected graph.

### 4.2.3. AsynchSpanningTree vs SynchBFS

While AsynchSpanningTree looks like an asynchronous translation of SynchBFS, the former does not necessarily produce a breadh first spanning tree.

Faster longer paths will win over a slower direct path when setting up parent.

One can howsever show that a spanning tree is constructed.

In asynchronous systems although time is unbounded it is practical to assume a upper bound on time taken to execute a process effect, time l, and time taken to deliver a message in channel, time d:
- Messages complexity: O(|E|);
- Time complexity: O(diam(l+d)).

Although a tree with height h, such that h > diam, can occur it only occurs if it does nottake more timethan a tree with h = diam. Faster long paths must be faster.

Applications of AsynchSpanningTree:
- Child Pointers and Broadcast: if nodes report parent or nonparent one can build a tree that broadcasts. Time complexity of this tree still O(diam(l+d))? No, because a fast path is not always fast. Complexity is O(h(l+d)), at most O(n(l+d)), where n = |V|.
- Broadcast with Acks: it's possible to build a AsynchBcastAck algorithm that collects acknowledgements as the tree is constructed. Upon incoming broadcast messages each node Acks if they already know the broadcast and Acks to parentonce when all neighbours Ack to them.
- Leader Election with AsynchBcastAck: includes termination and if all nodes initiate it and report their UIDs it can be used for Leader Election with unknown diameter and number of nodes.

## 4.3. Epidemic Broadcast Trees (Plumtree Protocol)

- Gossip broadcast:
  - (+) highly scalable and resilient;
  - (-) excessive message overhead.
- Tree-based broadcast:
  - (+) small message complexity;
  - (-) fragile in the presence of failures.

Can we get the best of both worlds?

Gossip strategies:
- Eager push: nodes immediately forward new messages;
- Pull: nodes periodically query for new messages;
- Lazy push: nodes push new message ids and accept pulls; separation among payload and metadata.

Gossiping into tree:
- Nodes sample random peers into a eagerPush set;
- Neighbours should be stable and TCP can be used;
- Links are kept reciprocal (towards undirected graph);
- First message reception puts origin in eagerPush;
- Further duplicate receptions moves source to lazyPush;
- Eager push of payload and lazy push of metadata.

Repairing a broken tree: if tree breaks, and graph stays connected, nodes get metadata but not payloads. This is detected by timer expiration and the metadata source in lazyPush is moved to eagerPush. Potential redundancy (due to cycles) is cleared later by the standard algorithm.

Small Worlds:
- Milgram experiment "6 degrees of separation";
- Path lengths were calculated on sequences of letter forwardings;
- In theory, letters would be send from random senders and progressively forwarded to random recipients;
- At each point the letter was forwarded to an address and recipient more likel yto know the final destination recipient;
- An averate path length close to 6 hops was found in the results;
- "Why sould there exist short cains of acquaintances linking together arbitrary pairs of strangers?"

Random graphs and clustering:
- Consider a graph where a given number of edges is created uniformely at random among the graph vertices. See Erdos-Rényi model;
- The resulting random graph is known to depict a low diameter and thus ould support small paths (O(log n));
- Are random graphs a good model for people acquaintances? Not so, since people's graphs have more clustering. If A is friend to B and C, then it is likely that B and C are also friends;
- Watts and Strogatz proposed a model that mixes short range and long range contacts;
- Nodes estabilish k local contacts using some distance metric among vertices (say in a ring or lattice) and then a few long range contacts uniformly at random;
- Resulting in low diameter and high clustering.

Routing in Small Worlds:
- If we flood a Watts and Strogatz graph we will stumble on a short route between to arbitrary points. A global observer couuld also pinpoint a O(log N) path;
- But can we pick a path with local knowledge and a distance metric? Not so easy: going to the nest nearest point dest not home in the target, we can keep jumping and only achieve O(sqrt(N)) paths. These paths lack locallity.
- Kleinberg solver this issue by choosing a probability function that can restore locality to long links (Check in R: n = 10; s = exp(log(n)*(runif(1000) -1)); hist(s,100).).
- Long range contacts can be tuned to become more clustered in the vicinity. The target is to have uniformity across all distance scales, a property found in DHT desings like Chord, and locally find O(log^2 N) routes.