# Block Building in Fuel

This section focuses on block building in Fuel and the role that the Block Builder plays in it.

The Fuel Block Builder is a component in the Fuel rollups, which is responsible for:

- Processing messages from L1 → L2
- Processing transactions in the mempool 
- Building blocks and submitting them to the Layer 1
- Providing soft-finality on the Layer 2

## L1 → L2 processing

Fuel rollups have a messaging system (from L1 → L2 and vice versa), which we will discuss further in the next section on bridging. In addition to relaying bridge messages, this system allows transactions to be sent directly from L1, which is used for forced transaction inclusion.

The Fuel Block Builder uses a relay to receive messages and transactions from L1 → L2, we will discuss both of these cases individually now.

### L1 Messages

Block Builder receives relayed messages from Layer 1 emitted as L1 events. The message is then picked up as part of the block-building process; each message sent from the L1 has the following format:

| name      | type      | description                                                  |
|-----------|-----------|--------------------------------------------------------------|
| sender    | bytes[32] | The identity of the sender of the message on the L1          |
| recipient | bytes[32] | The recipient of the message on the Fuel Blockchain          |
| nonce     | bytes[32] | Unique identifier of the message assigned by the L1 contract |
| amount    | uint64    | The amount of the base asset transfer                        |
| data      | byte[]    | Arbitrary message data                                       |


The block builder creates an output of type OutputMessage, and after this output is created, the processing of this message can be considered done.

Then, applications can leverage these OutputMessage(s) as they see fit. One example is the deposit process, where the bridge contract mints new ETH on the L2 after receiving specific messages that prove deposits on the L1 (we will discuss this further in the next section).

### L1 Transactions and Forced Inclusion

Fuel provides forced inclusion of transactions. If a user feels that the layer-2 block builder is trying to censor them, they can emit their serialized transaction from the L1 as an event, forcing the L2 Block Builder to include it in the block building. This process, called “Forced Inclusion,” guarantees user censorship resistance.

The Fuel transactions sent from L1 are emitted as events via the L1 and have the following format:

| name                   | type      | description                                                        |
|------------------------|-----------|--------------------------------------------------------------------|
| nonce                  | bytes[32] | Unique identifier of the transaction assigned by the L1 contract   |
| max_gas                | uint64    | The maximum amount of gas allowed to use on Fuel Blockchain        |
| serialized_transaction | byte[]    | The serialized transaction bytes following canonical serialization |


Forced inclusion allows processing of all transaction types except Mint, which can only be created by the Fuel Block Builder. This exception in no way restricts security guarantees for users' censorship resistance.

### Guarantees around L1 processing

One natural question that arises as part of this discussion is how does the L2 guarantee that it will always process a message or a transaction sent from the L1?

This is done by appending the da_height, i.e., the L1 block up to which the current block processes messages and transactions. A commitment for all the events and transactions is stored as part of the block header, using a Merkle tree and its root.

All events from L1 -> L2 (both inbox messages and forced transactions), are ordered by their block number and the index in that block. Following this order allows us to find a deterministic way of creating this Merkle tree.

We create this Merkle tree and store the root in the event_inbox_root field as part of the block header.

Later, Fuel blocks can be challenged, where it can be proven that a particular message or transaction was not included or processed, and hence, the corresponding block builder can be slashed.

## Processing Local Transactions

Apart from processing messages and transactions from L1 -> L2, the Block Builder is responsible for processing transactions sent to it locally. Users can send transactions to the Block Builder locally, collected in its Mempool, and then processed and sent to Layer 1.

This allows clever batching and compression techniques (gzip or zstd) to give users cheaper transactions than if they did them by submitting them directly to Layer 1.

Another advantage of sending transactions directly to the Block Builder is getting faster soft finality on the L2. For a transaction sent via L1 to be processed, first, the L1 block has to be finalized.


## Block Building and Proposing

The Fuel Block Builder is required to bundle transactions into blocks and propose them to Layer 1 as part of processing transactions. Committed blocks on Fuel have a  “Challenge Window,” which, when complete, allows the block and its corresponding state to be considered to have achieved “L1 finality.” 

The Fuel Block Builder currently sends the block hash and block height as new updates to the on-chain message portal, along with blobs containing transactions and other data,  to provide DA for that specific block.

## Transaction Ordering and MEV

The current Fuel Block Builder decides the priority of a transaction by taking by sorting by tip/max_gas, which means unlike many other L2s, the network isn’t FIFO; this also means that in Fuel, the Priority of your transaction inclusion is:

- directly proportional to the tip you provide as part of the transaction
- Inversely proportional to the max_gas you permit for your transaction

## Soft Finality

The Block Builder also plays a big role in providing soft finality for L2 transactions. As a participant on the L2, you can decide at what Finality you are fine making business decisions.

The Block Builder processing your transaction provides a soft finality that it has ordered and processed. It can be considered confirmed until the Block Builder fails to finalize this on the L1.

## Appendix

### Full Nodes

The fuel-core software also allows you to run a Full Node. A full node collects the latest updates on Layer 2 from the peers and broadcasts incoming transactions to the network.

Full nodes cannot build blocks; instead, they receive them as updates via p2p and re-execute them locally to maintain the correct state with complete verification.

By running the Full node, you can, as a user, be given the ability to keep verifying the L2 yourself and, hence, also be able to send fraud proofs. You also get your own GraphQL endpoint, which you can use to broadcast your transactions to the network.

All Fuel GraphQL providers run Full Nodes themselves to provide you with the latest Fuel state and allow you to broadcast transactions.
