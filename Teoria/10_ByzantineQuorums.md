# 10. Byzantine Quorums

## 10.1. Consensus with Byzantine Failures

A Byzantine process may derivate arbitrarily from its specifications:
- A Byzantine process may send contradictory messages;
- But other processes may not know which processes are Byantine.

Byzantine Generals Problem (BGP): reliable broadcast: can be used as a building block of a solution to the distributed consensus problem.
- Agreement: All non-faulty processes deliver the same message;
- Validity: if the broadcaster is non-faulty, then all non-faulty processes deliver the message sent by the broadcaster.

Impossibility: there is no solution to the BGP in a system with 3 processes, if one of them may be Byzantine, if messages are not signed: the general result is that faulty processes must be fewer than a third of all processes.

Communication without signed messages: the LHS receiver gets the same messages in both scenarios: it should deliver the same message in both scenarios but no message satisfies the desired properties in both cases.

Communication with signed messages: byzantine receiver cannot properly sign a modified message: of course, the protocol must prevent replay attacks.

## 10.2. Quorum Consensus with Byzantine Replicas

Quorum Consensus:
- Each (replicated) operation (ex: read/write) requires a quorum  (set of replicas);
- The fundamental property of these quorums is that: if the result of one operation depends on the result of another, then their quorums must overlap (ex: have common replicas);
- A simple way to define quorums is to consider all replicas as peers (quorums determined by their size (number of replicas in the quorum); equivalent to assign 1 vote to each replica).
- More generally: Set of servers, U
  - A quorum system Q ⊆ 2^U is a non-empty set of subsets of U, every pair of which intersect.
  - Each set q ∈ Q is a quorum.

Byzantine Quorums: Model:
- Processes: some servers in U may exhibit byzantine failures;
- Network: any 2 processes (clients or servers) can communicate over an authenticated and reliable point-to-point channel; asynchronous communication.

Problem Definition:
- Clients perform read and write operations on a variable x replicated at all servers (in U);
- Each server stores a replica (copy) of x and a timestamp p;
- Timestamps are assigned by clients when the client writes the replica (each client, c, chooses its timestamps from a set of timestamps, Tc that does not overlap the sets of all other clients).

Safety: any read that is not concurrent with writes returns the last value written in some serialization of the preceding writes:
- Operations are assumed to have begin and end events that determine a partial order;
- Essentially, this means linearizability of read/write operations.

(Access Protocol)

## 10.3. Byzantine Masking Quorums

- Q1: latest write quorum
- B: set of byzantine nodes
- Q2: read quorum
- (Q1 ∩Q2) \ B: servers with up-to-date values
- Q2 \(Q1 ∪B): servers with stale values
- Q2 ∩ B: arbitrary values

Size-based Byzantine Masking Quorums:

Masking Quorum System, Q for a fail-prone system B if:
- M-consistency
  - ∀ Q1,Q2 ∈ Qiu, ∀B1,B2 ∈ Beta : (Q1 ∩ Q2) \ B1 ~⊆ B2
  - ensures that a client always obtains an up-to-date-value
  - assuming all servers are peers:
    - f: bound on faulty servers;
    - we need at least f+1 up-to-date non-faulty servers;
    - thus every pair of quorums must intersect in at least 2f+1 servers;
    - 2q - n >= 2f + 1;
- M-availability
  - ∀B ∈ Beta : ∃Q ∈ Qiu: B ∩ Q = ∅
  - required for liveness
  - assuming all servers are peers:
    - f: upper-bound on byzantine servers;
    - q: size of a quorum;
    - n: number of servers;
    - then n - f >= q;
    - Combining with the inequality derived from M-consistency: q = 3f + 1.

Non-byzantine Read/Write Quorums Based on Size:
- w >= f + 1 ensures that writes survive failures;
- w + r > n ensures that reads see most recent write;
- n - f >= r ensures read availability;
- n - f >= w ensures write availability.
- Increasing n only worsens performance (as it requires larger quorums), increasing fault tolerance, as f can increase.

## 10.4. Dissemination Quorum Systems

Application: repositories of self-verifying information (ex: apps in which clients can verify the validity of the information, where clients can detect the tampering of servers, ...):
- Repositories of public keys: they are signed by CAs;
- Blockchains: next classes.

Dissemination Quorum System, Q for a fail-prone system B if:
- D-consistency
  - ∀Q1,Q2 ∈ Qiu, ∀B ∈ Beta : (Q1 ∩ Q2) ~⊆ B
- D-availability
  - ∀B ∈ Beta : ∃Q ∈ Qiu: B ∩ Q = ∅

Safety:
1. Any read that is not concurrent with writes returns the last value written in some serialization of the preceding writes;
2. A read that is concurrent with one or more writes returns:
   1. either the value written by the last preceding write;
   2. or any of the values being written in a concurrent write.

(...)

## 10.5. Critical Evaluation

- Compared with state machine replication, Byzantine Quorums appear to require fewer messages, by a large margin;
- But the protocols we have seen assume that:
  - Either clients can be trusted;
  - Or the information stored is self-verifiable.
- To handle other cases, in Phalanx Malkhi and Reiter use consensus-objects, which appear to require about the same number of messages as Byzantine SMR;
- Furthermore, these protocols support only read/write operations.
  - Although, we can build more complex operations on top of read/write operations, the number of messages will increase;
  - Also, although read/write operations are atomic, and performed in the same order, consistency problems may arise when we build more complex operations on top of read/write (in Phalanx Malkhi and Reiter use mutual exclusion objects).