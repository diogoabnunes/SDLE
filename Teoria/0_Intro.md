# Intro

Distributed System: System with 2 or more processes:
1. executing on different computers;
2. communicating via messages with a no negligible delay (with respect to computation).

Large Scale System: System with:
- Thousands processes likely in different data centers;
- Geographically distributed high latency communication;
- Heavy load up to millions of request per second;
- Several administrative domains: Gmail, Netflix, Facebook, Amazon, Microsoft, Google, Bitcoin.

Synchronous model: expected in small scale and cost dominated (ex: cars subsystems).
- Processing delays have a known bound;
- Message delivery delays have a known bound;
- Rate of drift of local clocks has a known bound.

Asynchronous model: adapted to large scale and can spread costs (ex: Internet).
- Processing delays are unbounded or unknown;
- Message delivery delays are unbounded or unknown;
- Rate of drift of local clocks is unbounded or unknown.

Consistency Model: in an ideal world there would be only 1 consistency model: when an update is made all observers would see that update. Building reliable distributed systems at a worldwide scale demands trade-offs between consistency and availability.

Syllabus:
1. Communication and Processing for Large Scale:
   - Message Oriented Middleware:
     - Message Queues;
     - Publish-Subscribe.
   - Processing;
   - Quorum Systems: Linearizability (strong consistency).
2. Large Scale Systems:
   - Scalable Distributed Topologies: Graphs, Gossip, Broadcast;
   - System Design for Large Scale: P2P, Cloud and planetary systems;
   - Physical and Logical Time;
   - High Availability under Eventual Consistency: local first and convergence.
3. Large Scale Byzantine Algorithms:
   - Byzantine Generals;
   - Byzantine Quorums;
   - Distributed Ledgers: Blockchain.