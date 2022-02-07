# 11. Pratical Byzantine Fault Tolerance

## 11.1. Impossibility of consensus with a faulty process

Consensus problem: Each process starts witn an input value from a set V and must decide a value from V.

Safety properties:
- Agreement: no two correct processes decide differently;
- Validity: the decided value depends on the input values of the processes.

Liveness: every execution of a protocol decides a value.

Theorem: in an asynchronous system in which at least one process may crash, there is no deterministic consensus protocol that is both live and safe:
- even if the network is reliable (doesn't loose messages);
- this was proven and is often referred to as FLP's impossibility result.

## 11.2. System Model (Asynchronous system) and Problem Definition

Network may:
- fail to deliver messages;
- delay messages;
- duplicate messages;
- deliver messages out of order.

Nodes/processes may fail arbitrarily (Byzantine failure model) but independently.

Processes use cryptographic techniiques to:
- prevent spoofing and replays;
- detect corrupted messages.

Service properties:
- State machine replication: replicated service with a state and some operations;
- Safety: the replicated service satisfies linearizability;
- Liveness: cannot be assured in an asynchronous system;
- Resiliency: tolerates f faulty replicas with n = 3f + 1 replicas.

## 11.3. Protocol Overview

View: numbered system configuration.
- Replicas move through a succession of views;
- Each view has a leader: p = v mod n
  - v: view number
  - n = 3f + 1 (number of replicas)
- View changes occur upon suspicion of the current leader (which may be caused by network instability).

Algorithm (SMR):
1. A client sends a request to execute a service operation to the leader;
2. The leader atomically broadcasts the request to all replicas;
3. Replicas execute the request and send the reply to the client;
4. The client waits for replies with the same result from f+1 replicas.

## 11.4. Atomic Broadcast

Quorums and Certificates:
- PBFT uses quorums to implement atomic multicast;
- These quorums satisfy 2 properties:
  - Intersection: any 2 quorums have at least a correct replica in common (as they intersect in f+1 replicas);
  - Availability: there is always a quorum with no faulty replicas.
- Messages are sent to replicas;
- Replicas collect quorum certificates:
  - Quorum certificate: set with 1 message for each element in quorum, ensuring that relevant information has been stored;
  - Weak certificate: set with at least f+1 messages from different replicas.

Replicas:
- state of each replica comprises:
  - service state;
  - message log (containing messages the replica has accepted);
  - view id.
- When the leader receives a client request, starts a three-phase protocol to atomically multicast the request to the replicas:
  - pre-prepare;
  - prepare (together with pre-prepare ensure total order of requests in a view);
  - commit (together with prepare ensure total order of requests across views).

## 11.5. View Change

Purpose: ensure liveness upon failure of the leader (while ensuring safety).

## 11.6. Final Remarks

Issues:
- Fairness: implementation guarantees that clients get replies to their requests even when there are other clients accessin the service;
- Speeding up cryptographic operations: around the year 2000, computing a 1024-bit RSA signature was about 3 orders of magnitude slower than computing an MD5 message digest.

Byzantine Quorums vs. PBFT
- Compared with state machine replication, Byzantine Quorums appear to require fewer messages, by a large margin;
- But the protocols we have seen assume that:
  - either clients can be trusted;
  - or the information stored is self-verifiable.
- To handle other cases, in Phalanx use consensus-objects, which appear to require about the same number of messages as Byzantine SMR;
- there quorum protocols support only read/write operations.