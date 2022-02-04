# 1. Message Oriented Middleware

## 1.1. Message-based communication

Dystributed system: collection of distinct processes which are spatially separated and which communicate with another by exchanging message.

Message: Atomic bit string; its format and its meaning are specified by a communications protocol.

The transport of a message from its source to its destination is performed by a computer network. The network can be abstracted as a communication channel.

Internet Protocols:
- Application: Specific communication services;
- Transport: communication between 2 (or more) processes;
- Network: communication between 2 computers not directly connected with each other;
- Interface: communication between 2 computers directly connected.

Properties of the communication channel provided to an application depend on the transport protocol used:
| Property          | UDP       | TCP       |
| -                 | -         | -         |
| Abstraction       | Message   | Stream    |
| Connection-based  | N         | Y         |
| Reliability (loss & duplication) | N | Y  |
| Order             | N         | Y         |
| Flow Control      | N         | Y         |
| No. of recipients | 1 to n    | 1         |

TCP Reliability:
- Message Loss
  - Can we assume all data sent through a TCP connection will be delivered to the remote end?
  - Bad things can happen: networking hadrware misconfiguration or failures, unplugged or damaged cables;
  - TCP guarantees that the application will be notified if the local end is unable to communicate with the remote end. TCP cannot guarantee that there is no data loss.
  - It's up to the application to deal with this:
    - A web browser may just report the problem to the user;
    - If the application does not interface with the user, try to connect again;
    - But TCP does not re-transmit data that was lost in other connections.
- Message Duplication
  - Why not always re-transmit message that might have not been delivered?
  - The re-transmited message may have been delivered before the connection was closed:
    - TCP is not able to filter data duplicated by the application;
    - May be an issue (ex.: if the duplicated data is a request for a non-idempotent operation, suchas a purchase order);
    - In this case the application may need to synchronize with the remote end to learn if there was some data loss in either direction.

Client Stub:
- Request:
    1. Assembles message: parameter marshalling;
    2. Sends message, via write()/sendto() to server;
    3. Blocks waiting for response, via read()/recvfrom(); not in the case of asynchronous RPC.
- Response:
    1. Receives responses;
    2. Extracts the results (unmarshalling);
    3. Returns to client (assuming synchronous RPC).

Server Stub:
- Request:
    1. Receives message with request via read()/recvfrom();
    2. Parses message to determine arguments (unmarshalling);
    3. Calls function.
- Response:
    1. Assembles message with the return value of the function;
    2. Sends message, via write()/sendto();
    3. Blocks waiting for a new request.

RPC:
- Synchronous: client blocks until it receives the response.
- Asynchronous: client receives a confirmation of the accepted request and wait for the return results.
- very useful communications paradigm: programming distributed applications with (non-asynchronous) RPC would be almost as simples as programming non-distributes applications, if it were not for failures.
- Limitations: not always the best approach; great for REQ-REP communication patterns but may be better alternatives.

Asynchronous Communication:
- Problem: the communicating parties may not always be simultaneously available (ex: a submission server may not be available when you want to submit your work);
- Solution: Asynchronous Communication: the communication parties need not be active simultaneously.

## 1.2. Asynchronous Communication (Message Oriented Middleware)

- Asynchronous message-based communication:
  - Sender and receiver need not synchronize with one another to exchange messages;
  - Communication service (middleware) stores the messages as long as needed to deliver them.
- Service close to that of the (snail) mail service;
- guarantees may vary: order, reliability;
- some MOM provides also an abstraction similar to discussion fora/news groups:
  - publishers may send messages;
  - subscribers may receive messages.

Basic Patterns:
- Point-to-point: queue model:
  - Several senders can put messages in a queue;
  - Several receivers can get messages from a queue;
  - Each message is delivered to at most 1 process (receiver) (helps improving scalability).
- Publish-subscriber: "discussion forum"; instead of queues we talk about topics:
  - Several publishers can put messages in a topic;
  - Several subscribers can get messages from a topic;
- A message is delivered to more than one process (subscribers).

Difference with respect to UDP:
- UDP communication also supports:
  - Unicast communication: any process maay send to a single destination (IP, port) pair;
  - Multicast communication: any process may send to an IP multicast group.
- Key properties:
  - Asynchrony: senders/publishers need not synchronize with receivers/subscribers;
  - Anonymity: senders/publishers need not know receivers/subscribers: queues and topics do not use transport-level address, but rather high-level naming.

Asynchronous Communication Applications: appropriate for applications when the sender and receiver are loosely coupled (ex.: enterprise app integration, workflow apps, microservices, message-based communication between people (email, SMS; instant messaging)).

Workflow Applications: related to business processes:
- Can be decomposed on a set of activies whose execution depends on:
  - Other activities on that process;
  - External events, which may be generated by other processes.
- Communication among activities can benefit from MOM: the receiving/consuming activity may not exist at the time the message is sent, because the preconditions for its execution may not yet be satisfied.

### 1.2.2. Java Message Service (JMS)

- Is an API for MOM (not a service neither a protocol):
  - allows Java apps to access MOM in a portable way;
  - it provides a maximum common divisor of the functionality provided by well known MIM providers.
- Representative of MOM functionalities that may be useful for developing enterprise applications; can be integrated with Java Transaction Service and take advantage of transactions.

Architecture and Model:
- 2 types of destinations:
  - Queues (single-destination communication);
  - Topics (multi-destination communication).
- 2 fundamental components:
  - JMS Provider: MOM service implementation (it includes client-side libraries);
  - JMS Client: app that sends/receives messages to a destination via the JMS Provider.
- Specifies the API and its semantics, that a provider offers to a client;
- To use it, client must first set up a connection to the provider (can be built on top of TCP);
- Clients send/receive messages to/from destinations in the context of a session (created in the context of a connection).

JMS Messages:
- 3 parts:
  - Header: set of fields (11) necessary for identifying and rounting messages;
  - Properties: defined by the apps, are optional fields that logically belong to the header (metadata):
    - a property is a (key,value) pair;
  - Body: data to exchange; can be typed.
- JMS doesn't specify the messages format on the wire (specifies an API not a protocol).

JMS Queues:
- Match the Queue Model;
- Are long lived (created by an admin, not the clients; always available to receive messages, even if there are no active receivers) with exception of temporary queues;
- Communication Semantics:
  - send():
    - Blocking: to ensure reliability client may have to synchronize with JMS server;
    - Asynchronous: callback is executed after synchronizatioin with JMS server.
  - receive():
    - Non-blocking: with 0 valued timeout (and blocking also);
    - Asynchronous: callback is executed upon message reception.
  - Reliability: depends on the delivery mode (send()):
    - PERSISTENT: ensures once-and-only-once semantics, even in the crash of a JMS server; implementation not trivial because of partial failures);
    - NON_PERSISTENT: ensures at-most-once semantics.
  - Consumer Acknowledgment: behavior is set per session: used to ensure that a message is delivered to 1 consumer; 3 modes:
    - AUTO_ACKNOWLEDGE: the JMS session automatically acknowledges upon a successful return from either receive() or reception callback;
    - DUPS_OK_ACKNOWLEDGE: the JMS session lazily acknowledges the delivery of messages;
    - CLIENT_ACKNOWLEDGE: it is up to the client to acknowledge the delivery of messages.
  - The Session interface offers also recover(), which is useful upon recovery of the consumer:
    - issue: upon failure of the consumer some messages may have been lost / have not been acknowledged;
    - solution: recover() allows the server to learn about the consumer's crash:
      - Server will resend messages starting on the first non-acknowledged message and will mark these messages as Redelivered.

JMS Topics:
- Supports the PUB-SUB pattern;
- Are long lived, just like queues, (created by an admin, not the clients; always available to receive messages, even if there are no active receivers) with exception of temporary topics;
- Sending/receiving messages to/from a topic use the same API as that used for queues;
- Topic Subscription: receives (all) messages sent to the respective topic. May be:
  - Durable: once created it exists until deleted;
  - Non-Durable: exists only while there is an active consumer (but the topic continues to exist).
  - Unshared: can have only 1 active consumer at a time;
  - Shared: multiple active consumers.
- Reliability: depends on the message's delivery mode and on the durability of the subscription:

| | Non-durable | Durable |
| - | - | - |
| NON_PERSISTENT | at-most-once (missed if inactive) | at-most-once |
| PERSISTENT | once-and-only-once (missed if inactive) | once-and-only-once |

- - Durable subscriptions provide same guarantees as queues;
- Like queues, no duplication guarantees don't hold on session recovery (it is up to the client/app to filter duplicates);
- Asynchronism of subscribers and publishers and communication latency:
  - a message sent after a subscription may not be delivered;
  - a message sent before a subscription may be delivered.
- Consumption order: similar to queues: messages sent by a session to a topic are delivered in the sending order (with same delivery mode); may be affected by other JMS features (priorities, delivery time, message selectors).

JMS doesn't support:
- Fault-tolerance/load balancing (doesn't specify how clients implementing a critical service cooperate);
- Error notification (messages for reporting problems or system events to clients);
- JMS Provider administration;
- Security (doesn't offer an API to manage security attributes of exchanged messages);
- Interoperability (between JMS providers).

Architecture: larger scale systems may use message relays to route messages to their destinations (ex: applications/services run on different data centers).

Message brokers: convert the messages format used by 1 application to the format used by another (not part of communication service).