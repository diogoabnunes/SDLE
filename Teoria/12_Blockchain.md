# 12. Blockchain

## 12.1. Bitcoin

Problem: how to make direct online payments withoug going through a trusted 3rd party?
- Bitcoin is more like a set of accounts rather than a big bucket of digital coins or digital bank-notes;
- Bitcoin maintains a record (Blockchain) of all transactions ever performed in a distributed fashion (rather than maintaining the balances of all accounts).
- A bitcoin transaction corresponds to a payment (to a transfer of "money" from one account to other accounts).

Assumptions:
1. There is a (large) peer-to-peer network of nodes with some computing resources.
2. There is a set of accounts each of which has a pair of private and public keys.

## 12.2. Bitcoin Blockchain

Blockchain: sequence of blocks.

Block: contains a set of transactions:
- set of events;
- each block contains a header with metadata;
- 1st block in the chain is the genesis block;
- blocks are appended to the blockchain head;
- maximum size of a block is 1 MByte.

Network: 
- Bitcoin's blockchain is maintained by a peer-to-peer network;
- Peers maintain random connections to other nodes/peers;
- Peers maintain a copy of the entire blockchain.

Consensus: needed to agree on the blocks and on their order.
- Conventional Byzantine Algorithms: difficult to know how many nodes there are and it's fairly easy to create multiple identities (Sybil attacks);
- Solution: Nakamoto protocol based on proof-to-work.

## 12.3. Bitcoin Proof-of-Work (PoW)

- Idea: solve a cryptographic puzzle that takes a random but large time.
- Rationale: SHA-256 is a non-invertible function, thus this puzzle must be solved by brute force.
- Target: can be tuned so as to adjust the difficulty of solving the puzzle.

Block Broadcasting:
- Upon solving the PoW, a node broadcasts the new block;
- Upon receiving a new block, a node checks its validity (its PoW (computing the hash of its header) and all transactions in the block) and if it is valid, node stops working on the PoW and adds the new block at the head of the blockchain, forwarding to the new block;
- When a node receives a new block, its chain may be missing some of its ancestors.

Block Broadcasting with Anti-Entropy:
- Upon validation of a new block;
- Upon receiving an inv message;
- Upon receiving a getdata message;
- Each block is inserted into the network.  

Block Propagation Delay:
- Block validation can add a significant delay;
- Block validation is repeated at every hop;
- Block propagation delay has a long tail distribution.

## 12.4. Bitcoin Forks

Fork: occurs when 2 or more nodes add a different block at the head of an otherwise identical blockchain at more or less the same time.

Resulution: based on the expected amount of work (usually length) of competing blockchains.

No finality: there is no 100% guarantee that a block will persist.

Eventual consistency: with high probability (assuming that the hash-power of an adversary is limited).

Fork Analysis:
- Depend on 2 factors:
  - expected time to generate the PoW;
  - the block propagation delay.
- Selfish mining strategies: node that finds a PoW for a block may withhold pushing that block to the network until it learns of a competing block;
- Can be used by an adversary to replace blocks;
- Network partitions can also lead to forks.

## 12.5. Bitcoin Scalability and Energy Consumption

Scalability issues: broadcasting, PoW computationally intensive, block cannot be larger than 1MB long, storage of the whole blockchain kepy by all (full-)nodes.

Transaction Rate Bound:
- Statistics: maximum 30-day average number of transactions confirmed per second by Bitcoin over the last 5 year, was 4.3;
- Theoretical limit of less than 8 transactions per second;
- Visa processes, on averate, 1700 transactions per second;
- Bitcoin parameter tuning cannot make for this difference of more than 3 orders of magniture (assuming capacity of 10K).

Energy Consumption:
- Extremely low energy-efficiency: the PoW is tuned so as to ensure a constant block-rate;
- Climate change: the impact is difficult to assess.

## 12.6. Proof-of-Stake (PoS)

- PoS: alternative to PoW;
- Idea: run a lottery to decide which user adds the next block to the chain. Users that "buy" more tickets have a higher chance to win;
- Coinage (coin + age): product of the amount of coins by the time that amount is held;
- Lottery: is run by requiring the hash of the block header to be below a given target;
- Clock synchronization: needed to validate blocks;
- Ties: broken using the block's coinage;
- Coinage consumption: occurs when 

## 12.7. Permissioned Blockchains

Features:
- allows to store unforgeable data in a persistend and transparent way;
- can be used to implement smart contracts.

PBFT vs PoW:
| Feature | PoW | PBFT |
| - | - | - |
| Node ids | open | known a priori |
| Node scalability | ~1000's | unknown |
| Latency | confirmation score | close to network's |
| Power consumption | high | low |
| Synchrony | for block validation | for liveness |
| Correctness proofs | kind of | yes |