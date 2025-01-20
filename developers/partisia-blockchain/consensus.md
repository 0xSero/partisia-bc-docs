# Consensus

## Consensus <a href="#consensus" id="consensus"></a>

Consensus plays a central role in blockchains and is required for them to function correctly.

Below we motivate the need for consensus and provide an outline of how the consensus protocol of Partisia Blockchain works.

### Why a consensus mechanism is needed <a href="#why-a-consensus-mechanism-is-needed" id="why-a-consensus-mechanism-is-needed"></a>

As described in the [Introduction to the fundamentals](https://partisiablockchain.gitlab.io/documentation/pbc-fundamentals/introduction-to-the-fundamentals.html#what-is-a-blockchain), the core functionality of a blockchain is that it is a decentralized database. The fact that it is _decentralized_ simply means that no single node controls access to the database. That is, multiple nodes have a copy of the database (in fact any party that would like to have a copy of the database is free to obtain it) and all these copies must be _consistent_.

To gain some intuition for what this entails let us consider a simple example of a naive (inconsistent) implementation of a decentralized database. Consider a database that is supposed to contain non-overdraft bank account balances at two nodes

and , where the nodes simply execute transactions as soon as they receive them. Additionally, let us consider 3 parties, each with a bank account in the decentralized database: Alice, Bob, and Charlie. For our example, let the initial state of the database at both and

be:

| Account | Balance |
| ------- | ------- |
| Alice   | $50     |
| Bob     | $0      |
| Charlie | $0      |

Now, Alice would like to buy two services, each costing $50: one from Bob and one from Charlie. However, having only $50 available, she lacks the money to obtain both services. Nevertheless, the naive decentralized database would allow her to obtain both. Let us see how.

Alice could first buy a service from Bob by making a transaction

in which she sends the $50 to Bob. But she could choose to send the transaction only to , and convince Bob to check his account balance at

which would then show:

| Account | Balance |
| ------- | ------- |
| Alice   | $0      |
| Bob     | $50     |
| Charlie | $0      |

Swiftly after (before the transaction

reaches ), she could then buy the service from Charlie by making a transaction in which she sends $50 to Charlie, send the transaction to , and convince Charlie to check his balance at

which would then show:

| Account | Balance |
| ------- | ------- |
| Alice   | $0      |
| Bob     | $0      |
| Charlie | $50     |

As a result, the databases would end up in an inconsistent state and the non-overdraft policy of the system would have been violated, since the protocol allowed Alice to _double-spend_ her savings.

To prevent such a scenario it must be ensured that the decentralized database is consistent i.e., it must be ensured that _all transactions are executed in the same order_ at all nodes holding a copy of the database. This is exactly what a consensus mechanism ensures.

### What is a consensus mechanism <a href="#what-is-a-consensus-mechanism" id="what-is-a-consensus-mechanism"></a>

A consensus mechanism protocol ensures that all nodes participating in the protocol execute all transactions in the same order. That is, a consensus mechanism provides a _total order of input transactions_ agreed upon by all nodes participating in the protocol.

Here, we will not go into the technical definition of what it means that transactions are totally ordered. Interested readers can refer to the [technical description of the FastTrack protocol](https://drive.google.com/file/d/1nxAMs95F3Y6LhibOjHiDVRgAN2Z51iLn/view), where a detailed characterization of this is provided.

### FastTrack: The consensus mechanism of PBC <a href="#fasttrack-the-consensus-mechanism-of-pbc" id="fasttrack-the-consensus-mechanism-of-pbc"></a>

Partisia Blockchain uses a tailor-made consensus mechanism called _FastTrack_.

The FastTrack consensus protocol is an optimistic protocol with built-in failure recovery. When running optimistically, a single block producer (proposer) continuously produces new blocks whilst the rest of the committee verifies the validity of said blocks and signs off on this. Consensus is achieved using Proof-of-Justification (PoJ) and Proof-of-Finalization (PoF). Let

be the number of members in the current committee. A PoJ is then a list of at least

signatures confirming the validity of a block. This PoJ then becomes PoF for the previous block. When a PoJ exists for a block, we refer to it as being _justified_, and when a PoF exists for a block we refer to it as being _final_.

After a fixed number of blocks have been proposed by a single producer a new proposer is chosen by the rules of the protocol. As long as the proposer is honest and produces sufficiently many valid blocks that become justified (which also ensures that new blocks are consistently being finalized), the committee is satisfied with this flow. If the proposer fails to produce valid blocks, either by losing network connectivity or by malicious intent, the protocol enters what is called the _Shutdown state_. In this state the entire committee votes on which block is the newest. Once the committee has agreed on a block the next block proposer is chosen. To ensure that honest proposers are not shut down because of a calm day with not many transactions input to the chain, an empty "heartbeat" block is produced every 30 seconds if no transactions are received. This ensures that an honest proposer with good connectivity can produce sufficiently many blocks not to be shut down.

All Baker nodes run the ledger with the FastTrack plugin enabled and they communicate with the other Baker nodes for their specific shard. Since a separate consensus protocol runs for each available shard, cross-shard communication requires the propagation of the PoF from one shard to another.&#x20;

[A full technical description of the FastTrack protocol is available here](https://drive.google.com/file/d/1nxAMs95F3Y6LhibOjHiDVRgAN2Z51iLn/view).
