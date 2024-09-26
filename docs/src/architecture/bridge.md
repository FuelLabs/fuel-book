# Fuel and Ethereum

Fuel Ignition uses Ethereum as a Layer 1. We chose Ethereum as the L1 for Fuel for both Settlement and Data availability of the L2, because we think Fuel shares many of Ethereum’s values like:

- Building for the long-term sustainably
- Building with an emphasis on security
- Gocus on consumer hardware and making it accessible for ordinary people to participate in the protocol

Ethereum is one of the most decentralized L2s. It has been around for a long time and has been working on a rollup-centric roadmap for years. This makes it the best choice for building a rollup.

## Inheriting Ethereum’s Security

Fuel Ignition, Fuel’s flagship rollup, inherits Ethereum’s security. The natural question from the previous statement is, what do we mean by Inheriting Ethereum’s security?

Fuel uses Ethereum as the layer to keep users’ funds and propose its latest blocks and corresponding state updates. We deploy smart contracts that continuously get updates of Fuel Layer 2. 

Then, we have fraud-proving performed directly on the Ethereum L1 to prove that something about the posted blocks or related state updates is wrong. We also allow permissionless messaging and transaction inclusion via the L1 to ensure the user doesn’t experience any censorship resistance.

This gives the user guarantees that as long as Ethereum is secure and the honest majority assumption for it is held:

- No fraud blocks or state updates can be sent
- Their funds are always safe on the Layer 1
- They can never be stopped from withdrawing them or being able to send any transaction to the L2 (forced inclusion)

Now, we will discuss each of the properties we described. Ethereum as a Layer 1 helps provide Fuel in detail.

## Messaging

Fuel allows for messaging between L1 → L2 and L2 → L1, which means you can send any arbitrary message from Layer 1 to Layer 2 and vice versa. The protocol guarantees that if a message is included in the L1, it has to be processed on the L2 and vice versa. Let’s discuss both of these cases individually.

### L1 → L2 Messaging

The Fuel Message Portal facilitates message processing from L1 -> L2. Its method, sendMessage, accepts the L2 recipient (a Fuel Address) and the corresponding message to be sent. After a successful call to this method, a MessageSent event is emitted on Layer 1.

As discussed in the section on block building, part of processing the Fuel blocks requires committing to some L1 block height, up to which the block builder processes messages and transactions, this forces the Block builder to include all messages from the L1 (as in case of failure, the builder can be slashed).

As part of processing the message blocks from the L1, the block builder looks at the event and mints an OutputMessage transaction to the particular Fuel address with the specific data.

### L2 → L1 Messaging

Fuel also allows messages from the L2 -> L1 to be sent using MessageOut receipts. Every Fuel block includes a receipt root, the root of all receipts that were part of the block. This allows anyone to make a call to the relayMessage function of the Fuel Message Portal; a Merkle proof of inclusion is required to perform for the message you are trying to process along with that, it checks whether the block for which the message is being processed has been finalized or not (i.e., it outside of the challenge window).

Processing the message on the L1 coming from the L2 is done by calling the specific L1 address to which the message is sent to with some desired payload.

## ETH Deposits and Withdrawals

A core part of using the Fuel rollup is depositing ETH from the L1 to Fuel and withdrawing it from the L2. We will discuss both of these scenarios individually.

### ETH Deposits

The user can call the depositEth function on the L1 to create a deposit. The method is payable, and and emits a messageSent event with an empty payload, this makes the sequencer recoginize that this a deposit made on the L1 and it mints a new eth coin corresponding to the value of the deposit for the user.

### ETH Withdrawals

Withdrawals on the L2 are made by burning the tokens on the L2 via the L2 gateway. Then, the gateway emits a MessageOut receipt, which is part of the block header, allowing the relay of this message to Layer 1.

The Layer 1 Message Portal contract has a relayMessage function (read L2->L1 messaging for details); which allows for processing L2 messages aimed for L1, in the case of withdrawals, we send a message with the amount corresponding to the value the user has burned on the L2, and hence the Message Portal contract provides the L1 recipient with their funds for withdrawal.

Note: A withdrawal comes requires the “Challenge Window” to be cleared before being processed, and hence the user has to wait till the “Challenge Window” (although there are fast finality gadgets which can bring this down.)

## State Updates

Fuel uses Ethereum to submit new state updates. This is done via the State Contract on Layer 1, where the blocks are committed by sending the block hash and block height. The contract also records the timestamp as part of the commitment for a particular block.

These state updates and the data posted as Blobs on Ethereum allow for challenging any state updates sent to the L1.

## Challenge Window

The challenge window is the time it takes for a block and related state posted on the L1 to be considered finalized. Finalization means any withdrawal or message part of this block can be processed on the L1. For now, the challenge window for Fuel is seven days.

Techniques like hybrid proving and other fast finality gadgets can reduce the duration of the challenge window; we are actively researching these areas and would encourage you to read Nick Dodson’s post on faster finality gadgets for optimistic rollups.

## Hybrid Proving

Fuel believes in a philosophy of zk-pragmatism; rather than playing bisection games on-chain like other rollups (which increase the complexity of the proving system)  or sending zk proofs for every bath like zk rollups (which increase the cost per transaction), Fuel makes a hybrid approach for its proving system.

The system runs in an optimistic setting. If someone in the system believes that a fraud state has been sent, they create a zk-proof off-chain of the claim and prove fraud in a single interaction with the L1. This reduces the proving system's complexity and limits the challenge window.

Hybrid proving is being developed, and prototyping is done with RISC-V-based zkVMs like SP-1 and RISC-0. You can read more about the proving system here.

## Appendix

### Alt-DAs and L1s

We have launched our flagship rollup with Eth as our L1 for settlement and data availability, but Fuel believes in creating a neutral and modular tech stack. The Fuel tech stack can be extended to launch on alt L1s like Bitcoin and Solana and with alt DAs like Celestia and Avail. If someone wants, they can even use the Fuel stack to launch their L1. 

We will keep progressing our tech stack to be adaptable in multiple scenarios, resilient, and feasible on consumer-grade hardware.

### Blobs

EIP 4844 introduced Blobs as a cheaper way to get Data Availability for Ethereum rollups. Fuel block builders also use blobs, although this is a work in progress.

Fuel blocks are batched together in a bundle, compressed via popular techniques (gzip or zstd), and posted as blobs. Because blobs are fixed in size, uploading has to be done via a series of transactions.

Blobs and their exact implementation are still being finalized and will be live soon, but the above text summarizes the general approach for now.
