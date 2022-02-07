# 7. High Availability under Eventual Consistency

Speed of communication in the 19th century:
- Francis Galton Isochronic Map: within 10 days, 10-20, 20-30, 30-40, +40 days.

Speed of communication in the 21st century:
- 1 day is now about 10ms;
- Time delay between Mars and Earth: 12min.

## 7.1. Latency Magnitudes

- λ, up to 50ms (local region DC);
- Λ, between 100ms and 300ms (inter-continental).

No inter-DC replication: client writes observe λ latency.

Planet-wide geo-replication: replication techniques vs. client side write latency ranges:
- Consensus/Paxos: [Λ,2Λ] (with no divergence);
- Primary-Backup: [λ,Λ] (asynchronous/lazy);
- Multi-Master: λ (allowing divergence).

Eventually Consistent (CACM 2009, Werner Vogels):
- In an ideal world there would be  only 1 consistent model: when an update is made all observers would see that update;
- Building reliable distributed systems at a worldwide scale demands trade-offs between consistency and availability.

CAP theorem (PODC 2000, Eric Brewer):
- of 3 properties of shared-data systems - data consistency, system availability, and tolerance to network partition - only 2 can be achieved at any given time.

Eventual Consistency: special case of weak consistency. After an update, if no new updates are made to the object, eventually all reads will return the same value, that reflects the last update (ex: DNS). This can later be reformulated to avoid quiescence, by adapting a session guarantee.

Session Guarantees:
- Read your Writes: read operations reflect previous writes;
- Monotonic Reads: sucessive reads reflect a non-decreasing set of writes;
- Writes follow Reads: writes are propagated after reads on which they depend (Writes made during the session are ordered after any Writes whose effects were seen by previous Reads in the session);
- Monotonic Writes: writes are propagated after writes that logically precede them (write is only incorporated into a server's database copy if the copy includes all previous session writes).

From sequential to concurrent executions:
- Consensus provides illusion of a single replica;
- This also preserves (slow) sequential behaviour;
- We have an ordered set (O,<). O = {o,p,q} and o < p < q.
- EC Multi-master (or active-active) can expose concurrency;
- Partially ordered set (O ,≺). o ≺ p ≺ q ≺ r and o ≺ s ≺ r
- Some ops in O are concurrent: p || s and q || s.

## 7.2. Conflict-Free Replicated Data Types (CRDTs)

- Convergence after concurrent updates. Favor AP under CAP;
- Examples include counters, sets, mv-registers, maps, graphs;
- Operation based CRDTs. Operation effects must commute:
  - in some data types all operations are commutative;
  - for more complex examples operations need to generate "special" commutative effects.
- State based CRDTs are rooted on join semi-lattices:
  - join concurrent sets;

Now convergence can related to known updates, with no need to stop updates: upds(a) ⊆ upds(b) => a <= b.

This is slightly weaker than the previous definition and implies it: upds(a) = upds(b) => a = b.

Design of CRDTs:
- partially ordered log (polog) pf operations implements any CRDT;
- replicas keep increasing local views of an evolving distributed polog;
- any query, at replica i, can  be expressed from local polog Oi;
  - ex: Counter at i is |{inc | inc ∈ Oi}| - |{dec | dec ∈ Oi}|
- efficient representations that follow some general rules.

Principle of permutation equivalence: if operations in sequence can commute, preserving a given result, then under concurrency they should preserve the same result:
- lets track total number of incs and decs done at each replica;
- separate positive and negative counts are kept per replica;
- join does point-wise maximums among entries (semilattice);
- at any time, counter value is sum of incs minus sum of decs.

Registers: ordered set of write operations.

Last Writer Wins: policy approach to evolve state without string coordination. Popularized in the Cassandra system, uses timestamps do discard older writes and attain convergence.

Sequential Semantics Sets: {e |add(e) ∈ Oi ∧ (dont exist)rmv(e) ∈ Oi · add(e) < rmv(e)}.
- Problem: Concurrently adding and removing the same element;
- Possible solution: Add-Wins: {e |add(e) ∈ Oi ∧ (dont exist)rmv(e) ∈ Oi · add(e) ≺ rmv(e)}.

Conclusions:
- Concurrent executions are needed to deal with latency;
- Behaviour changes when moving from sequential to concurrent.

Road to accomodate transition:
- Permutation equivalence;
- Preserving sequential semantics;
- Concurrent executions lead to richer outcomes.

CRDTs provide sound guidelines and encode policies.