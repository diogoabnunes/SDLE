# 6. Physical and Logical Time

Everything is distributed: "Distributed systems once were the territory of computer sciente Ph.D.s and software architects tucked off in a corner somewhere. That's no longer the case."

Anomalies in Distributed Systems:
- A buzz, you have a new message, but message is not there yet;
- Remove the boss from a group, post to it and she sees posting;
- Read a value but cannot overwrite it;
- Assorted inconsistencies.

## 6.1. Time

Can't we use time(stamps) to fix it?

The problem with time is that "there is no now". There is no single universal reference frame.

- Time needs memory (since it is countable change);
- Time is local (rate of change slows with acceleration);
- Time synchronization is harder at distance;
- Time is a bad proxy for causality.

Clock Drift: drift between measured time and a reference time for the same measurement unit.
- Quartz clocks: from 10^-6 to 10^-8 seconds per second. 10^6 seconds per second means 1 second each 11.6 days;
- Atomic clocks: about 10^-13 seconds per second;

Second: defined in terms of transitions of Cesium-133. Coordinated Universal Time (UTC), inserts or removes seconds to sync with orbital time.

- External synchronization: states a precision with respect to an authoritative reference.
  - For a band D > 0 and UTC source S we have: |S(t) - Ci(t)| < D
- Internal synchronization: states a precision among 2 nodes (if 1 is authoritative it is also external).
  - For a band D > 0 we have |Cj(t) - Ci(t)| < D

An externally synchronized system at band D is necessarily also at 2D internal synchronization.

Monotonicity: some uses (ex: make) require time monotonicity (t' > t => C(t') > C(t)) on wall-clock adjusting. Correcting advance clocks can be obtained by reducing the time rate until aimed synchrony is reached.

## 6.2. Synchronization

### 6.2.1. Synchronous System

- Knowing the transmit time "trans" and receiving origin time "t", one could set to: t + trans;
- However "trans" can vary between tmin and tmax;
- Using t + tmin or t + tmax the uncertainty is u = tmax-tmin;
- But, using t + (tmin + tmax)/2 the uncertainty becomes u/2.

### 6.2.2. Asynchronous System

Cristian's Algorithm:
- Send a request "mr" that triggers responde "mt" carrying time t;
- Measure the rount-trip-time (RTT) of request and replay as "tr";
- set clock to t + tr/2, assuming a balanced round trip;
- Precision can be increased by repeating the protocol until a low "tr" occurs.

Berkeley Algorithm:
- coordinator measures RTT to the other nodes and sets target time to the average of times. New times for nodes to set are propagated as deltas to their local times, in order to be resilient to propagation delays.

## 6.3. Causality

A social interaction:
- Alice decides to have dinner
- She tells that to Bob and he agrees
- Meanwhile Chris was bored
- Bob tells Chris and he asks to join for dinner

Causality is a partial order relation:
- Causally: "Alice wants dinner" || "Chris is bored"
- Timeline: "Alice wants dinner" < "Chris is bored"

Causality is only potential influence:
- Past statements only potentially influence future ones:
  - Causally: "Bob is jogging" -> "Bob says: Yes, let's do it" => but they are probably unrelated.
  - Causally: "b = 5;" -> "if (a > 2) c = 2;" => but in fact not.

Causal Histories: 
- uniquely tag each event: for instance, node name and growing counter;
- collect memories as sets of unique events;
- set inclusion explains causality: {a1,b1} ⊂ {a1,a2,b1}
- you are in my past if I know your history;
- if we don't know each other's history, we are concurrent;
- if our histories are the same, we are the same;
- lots of redundancy that can be compressed -> vector clocks.

Vector Clocks:
- {a1,a2,b1,b2,b3,c1,c2,c3} -> {a→2,b→3,c→3}
- Finally a vector, assuming a fixed number of processes with totally ordered identifiers: [2,3,3];
- Graphically: same logic, but in graphics: concurrent, less than, difference, (point-wise maximum = join), ...;
- Node b, with [0,1,0] is receiving a message with [2,0,0]:
  - We need to combine the 2 vectors and update b entry.
- Vector clocks with dots: [2,0,0] becomes [1,0,0]a2
  - The causal past excludes the event itself.
  - Check [2,0,0] -> [2,2,0]:
    - Check [1,0,0]a2 -> [2,1,0]b2 if dot a2 index 2 <= 2

Registering relevant events:
- Not always important to track all events;
- Track only update events in data replicas;
- Applications in File-Systems, Databases, Version Control.

Dynamo: causally tracking of write/put operations.

Casual histories with only some relevant events:
- relevant events are marked with a • and get an unique tag/dot;
- other events get a ◦ and don't add to history.
- concurrent states {a1} || {b1} lead to a • marked merge M;
- causally dominated state {} -> {a1,b1,b2} is simply replaced.
- versions can be collected and merge deferred;
- casual histories are only merged upon version merging in a new •.

Version vectors: similar to vector clocks but only regists the differences.

## 6.4. Scaling Causality

One entry needed per source of concurrency.

Scaling at the edge (DVVs):
- Mediate interaction by DC proxies;
- Support failover and DC switching;
- One entry per proxy (not per client).

Dynamic concurrency degree (ITCs):
- Creation and retirement of active entities;
- Transparent tracking with minimal coordination;
- Causality tracing under concurrency across services.

Dynamo like, get/put interface:
- Conditional writes;
- Overwrite first value;
- Multi-Value ([1,0] || [0,1], one entry per client).

Dotted Version Vectors: scalable and accurate causality tracking for eventually consistent stores.

## 6.5. Dynamic Causality

Tracking causality requires exclusive access to identities. To avoid preconfiguring identities, id space can be split and joined.

Interval Tree Clocks (ITCs):
- A seed node controls the initial id space;
- Registering events can use any portion above the controlled id;
- Ids can be split from any available entity and be split again;
- Entities can register new events and become concurrent;
- Any 2 entries can merge together, eventually collecting the whole id space and simplifying the encoding of events.

Global Invariants on IDs: each active replica has to control a different (and corresponding) space of ids, so that those replicas can register events without any problems.

ITCs are used in concurrency tracing and debugging, and in distributed settings within the Pivot tracing system.

- Causality is important because time is limited;
- Causality is about memory of relevant events;
- Causal histories are very simple encodings of causality;
- VC, DVV, ITC do efficient encoding of causal histories;
- All mechanisms are only encoding of causal histories.

Graphical representations allow visualization of causality, this is useful in comparing existing mechanisms and to develop novel ones.