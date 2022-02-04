# 3. Replication and Consistency Models

Data Replication: replicate data at many nodes.
- Performance: local reads;
- Reliability: no data-loss unless data is lost in all replicas;
- Availability: data available unless all replicas fail or become unreachable;
- Scalability: balance load across nodes for reads.

Upon an update:
- Push data to all replicas;
- Challenge: ensure data consistency.

Conflicts: updating at different replicas may lead to different results -> inconsistent data.

Strong Consistency: all replicas execute updates in the same order.
- Deterministic updates: same initial state leads to same result;
- actually, total order is not enough: it must be sensible.

Strong Consistency Models:

- Sequential Consistency (Lamport 79): an execution is sequential consistent if it is identical to a sequential execution of all the operations in that execution, such that all operations executed by any thread appear in the order in which they were executed by the corresponding thread.
  - Observation: model provided by a multi-threaded system on a uniprocessor;
  - Counter-example: following operations on 2 replicas of variables x and y:
    - Replica N: initial values; operation; final values;
    - Replica 1: (2,3); x = y + 2; (5,5);
    - Replica 2: (2,3); y = x + 3; (5,5);
    - If the 2 operations are executed sequentially, the final result cannot be (5,5).
  - Sequential Consistent Replication Protocol
    - Data type: array of 4 elements:
      - read(a): read value of array's element/index a;
      - write(a, v): write value v to array's element/index a;
      - snapshot(): read all values stored in the array;
    - Protocol
      - Read: reads from one replica;
      - Write: writes to all replicas in same order. Writes have no reply: return after sending the write request messages to all replicas;
      - Snapshot: reads from one replica.
  - Sequential consistency is not composable: consider 2 sub-arrays, each of 2 elements. Assume the same algorithm to replicate each of the sub-arrays, and thus ensure sequential consistency on each array. The combined execution may not be sequential consistent.

- One-copy Serializability: the execution of a set of transactions is one-copy serializable if its outcome is similar to the execution of those transactions in a single copy:
  - Observation 1: seerializability used to be the most common consistency model used in transaction-based systems: DB systems nowadays provide weaker consistency models to achieve higher performance;
  - Observation 2: This is essentially the sequential consistency model, when the operations executed by all processors are transactions: the isolation property ensures that the outcome of the concurrent execution of a set of transactions is equal to some sequential execution of those transactions;
  - Observation 3: Linearizability (sort of) where as:
    - Serializability was proposed for databases, where there is a need to preserve complex application-specific invariants;
    - Sequential consistency was proposed for multiprocessing, where programmers are expected to reason about concurrency.

- Linearizability (Herlihy&Wing90): an execution is linearizable if it is sequential consistent and if op1 occurs before op2, according to one omniscient observer, then op1 appears before op2:
  - Assumption: operations have start time and finish time, measured on some global clock accessible to the omniscient observer;
    - op1 occurs before op2, if op1's finish time is smaller thanthat op2's start time;
    - if op1 and op2 overlap in time, their relative order may be any.
  - Replication Protocol for Linearizability:
    - Data type: array of 4 elements:
      - read(a): read value of array's element/index a;
      - write(a, v): write value v to array's element/index a;
      - snapshot(): read all values stored in the array;
    - Protocol
      - Read: reads from one replica;
      - Write: writes to all replicas in same order. Waif for ack from all replicas before returning;
      - Snapshot: reads from one replica.
      - Guaranteeing linearizability usually requires more synchronization.