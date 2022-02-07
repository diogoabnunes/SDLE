# 8. Quorum Consensus Replication

## 8.1. Quorums and Quorum Consensus Replication

Quorum Consensus Replication:
- clients communicate directly to the servers/replicas;
- each (replicated) operation (ex: read/write) requires a quorum (set of replicas).
- fundamental property of these quorums is that: if the result of one operation depends on the result of another, thentheir quorums must overlap (ex: have common replicas);
- a simple way to define quorums is to consider all replicas as peers: quorums determined by their size (number of replicas), equivalent to assign 1 vote to each replica.

Read/Write Quorums Must Overlap:
- Replicas provide only read and write operations;
- Because the output of a read operation depends on previous write operations, the read quorum must overlap the write quorum: NR + NW > N, where NR is size of read quorum, NW, is size of write quorum and the number of replicas.

Implementation:
- IMP:
  - Each object's replica has a version number; 
  - A write operation depends on previous write operations (via the version) and therefore write quorums must overlap: NW + NW > N.
- Read:
    1. Poll a read quorum, to find out the current version (a server replies with the current version);
    2. Read the object value from an up-to-date replica (if the size of the object is small, it can be read as the read quorum is assembled).
- Write:
    1. Poll a write quorum, to find out the current version (a server replies with the current version);
    2. Write the new value with the new version to a write quorum (we assume that writes modify the entire object, not parts of it).

Naive Implementation with Faults:
- N = 3; NR = 2; NW = 2;
- First/left client attempts to write, but because of a partition it updates only one replica (A);
- Second/right client, in different partition, attempts to write and it succeeds;
- Variable has different values for the same version;
- The partition heals and each client does a read;
- Each client gets a value differente from the one it wrote: protocol does not ensure read-your-writes.

## 8.2. Ensuring Consistency with Transactions

- Gifford assumes the use of transacions, which use two-phase commit or some variant:
  - The write (or read) of each replica is an operation of a distributed transaction (we can view the sequence of operations in a replica on behalf of a distributed transaction as a sub-transaction on that replica);
  - If the write is not accepted by at least a write quorum, the transaction aborts.
- Transactions also prevent consistencies in the case of concurrent writes:
  - Transactions ensure isolation, by using concurrency control;
  - Lets assume the use of locks.

## 8.3. Playing with Quorums

Protocol read-one/write-all: by choosing NR and NW appropriately we can trade off performance and availability of the different operations.
- By assigning each replica its own number of votes, which may be different from one, weighted-voting provides extra flexibility.

Quorum Consensus Fault Tolerance:
- Tolerates unavailability of replicas caused by both process (replicas) failures and communication failures, including partitions.
- The availability analysis by Giffors relies on the probability of crashing of a replica/server.

## 8.4. Dynamo Quorums

Dynamo: replicated key-value storage sysyem developed at Amazon.

It uses quorums to provide high-availabiltiy:
- Whereas Gifford's quorums support a simple read/write memory abstraction, Dynamo supports an associative memory abstraction, essentially a put(key,value)/get(key) API;
- Rather than a simple version number, each replica of a (key,value) pair has a version vector.

Dynamo further enhances high-availability by using multi-version objects. Thus sacrificing strong consistency under certain failure scenarios.

Dynamo's Quorums:
- Each key is associated with a set of servers, the preference list;
- Each operation get/put has a coordinator, which is one of the first N servers (main replicas) in the preference list: the coordinator is the process that executes the actions tipically executed by the client in Gifford's quorums(as well as the actions required from a replica);
- as in Gifford's quorums:
  - put(key,value,context) requires a quorum of W replicas:
    1. The coordinator generates the version vector for the new version and writes the new value locally;
    2. The coordinator sends the (key,value) and its version vector to the N first servers in the key's preference list.
  - get(key) requires a quorum of R replicas:
    1. The coordinator requests all versions of the (key,vakue) pair, including the respective version vectors, from the remaining first N servers in the preference list;
    2. On receiving the response from at least R-1 replicas, it returns all the (key,value) pairs whose version-vector are maximal.
  - R + W > N;

Without failures: Dynamo provides strong consistency.

In case of failures: the coordinator may not be able to get a quorum from the N first replicas in the preference list.

To ensure availability: the coordinator will try to get a sloppy quorum by enlisting the backup replicas in the preference list.

At the cost of consistency: sloppy quorums do not ensure that every quorum of a get() overlaps every quorum of a put().

Sloppy quorums are intended as a solution to temporary failures: to handle failures with a longer duration, Dynamo uses a anti-entropy approach for replica synchronization.