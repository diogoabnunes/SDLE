# 9 Quorum Consensus Replication ADT (Abstract Data Type)

## 9.1. Initial and Final Quorums

Herlihy proposed a generalization of quorum consensus to replicated abstract data types such as queues.

Quorum: for an operation is any set of replicas whose cooperation is sufficient to execute the operation.
- When executing an operation, a client:
  - reads from an initial quorum;
  - writes to a final quorum.
- Ex.: in the read operation, a client must read from some set of replicas, but its final quorum is empty;
- A quorum for an operation is any set of replicas that includes both an initial and a final quorum;
- Assuming that all replicas are considered equals, a quorum may be represented by a pair, (m,n), whose elements are the sizes of its initial (m) and its final (m) quorums.

Quorum intersection constraints are defined between the final quorum of one operation and the final quorum of another.

Example: Gifford's Read/Write Quorums
- Object (ex: file) read/write operations are subjecet to 2 constraints:
    1. Each final quorum for write must intersect each initial quorum for read;
    2. Each final quorum for write must intersect each initial quorum for write (to ensure that versions are updated properly).

Gifford's-based Replicated Queue:
- Enq: adds an item to the queue;
- Deq: removes least recent the item from the queue, raising an exception if the queue is empty.
    1. Read an initial read quorum to determine the current version of the queue;
    2. Read the state from an updated replica;
    3. If the queue is not empty, normal deq:
       1. Remove the item at the head of the queue;
       2. Write the new queue state to a final write quorum;
       3. Return the item removed in 3.1.
    4. If the queue is empty, abnormal deq, raise an exception.

## 9.2. (Herlihy's) Replicated ADTs

- Timestamps instead of version numbers:
  - Reduce quorum intersection constraints;
  - Reduce messages.
- Logs instead of (state) versions: these changes allow for more flexible replication quorums;
- Assumption: clients are able to generate timestamps that can be totally ordered: this order is consistent to that seen by an omniscient observer (ex: consistent with linearizability).

Read: similar to the version-based, except that a client uses the timestamp instead of the version to identify a replica that is up-to-date.

Write: there is no need to read the versions from an initial quorum:
- Timestamp generation guarantees total order consistent with the order seen by an omniscient observer;
- No need for initial message round;
- Client needs only write the new state to a final quorum (suitable only for whole state changes).

Replicated Event Logs vs Replicate State
- Event: State change, represented as a pair of:
  - Operation with respective arguments (Read() or Write(x));
  - Outcome a termination condition and returned results (Ok(x) or Ok());
  - Ex: [Read(), Ok(x)] and [Write(x), Ok()]
- Event log: sequence of log entries;
- Log entry: timestamped event: [op(args); term(results)]
  - Ex: [Enq(x); Ok()], [Deq(); Ok(x)]
  - Entries in a log are ordered by their timestamps.
- Idea: rather than replicate state, replicate event logs (event log subsumes the state).

## 9.3. Replicated Queue: an example of a replicated ADT

Herlihy's Replicated Queue:
- Deq implementation - Client (conceptual implementation):
    1. Reads the logs from an initial Deq quorum and creates a view (log obtained by mergint in timestamp order the entries of a set of logs, discarding duplicates);
    2. Reconstructs the queue state from the view, and finds the item to return;
    3. If the queue is not-empty:
       1. Records the Deq event, by appending a new entry to the view;
       2. Sending the modified view to a final Deq quorum of replicas (replicas merge this view with their local logs).
    4. Returns the response (the dequeued item or an exception) to Deq's caller.

Constraints:
1. Every initial Deq quorum must intersect every final Enq quorum (so that the reconstructed queue reflects all previous Enq events);
2. Every initial Deq quorum must intersect every final Deq quorum (so that the reconstructed queue reflects all previous Deq events);

Notes:
- The views for Enq operations need not include any prior events, because Enq returns no information about the queue's state: an initial Enq quorum may be empty;
- As before, an abnormal Deq has an empty final quorum.

Optimizations:
- Disadvantages: logs and messages grow indefinitely.
- Fixes:
  - Garbage collect logs take advantage of observation:
    - If an item has been dequeued, all items with earlier timestamps must have been dequeued;
    - However, we can't just remove all the entries with earlier timestamps;
    - But it is enough to keep the horizon timestamp (ex: timestamp of the most recently dequeued item).
  - Cache logs at clients.

## 9.4. Critical Evaluation

Issues with Replicated ADTs:
- Timestamps generated by clients and consistent with linearizability:
  - Herlihy's relies on transactions and hierarchical timestamps;
  - If replicated ADTs do not use transactions this is challenging.
- Logs must be garbage collected to bound the size of messages:
  - Garbage collecting log entries is ADT-dependent.