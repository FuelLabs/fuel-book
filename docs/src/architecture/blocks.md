# Fuel Blocks

Blocks in Fuel are built by entities called Block Builders. Fuel blocks are made by processing transactions and messages. Transactions can be sent either directly to the builder or via layer 1, while messages are sent from layer 1. We expand further on messages and transactions sent from the L1 in the Fuel and Ethereum section of the book.

## Block Header

A Fuel block header at the top consists of 3 fields:

- Application Header
- Consensus Header
- Block Header Metadata

```c++
pub struct BlockHeaderV1 {

    pub application: ApplicationHeader<GeneratedApplicationFields>,

    pub consensus: ConsensusHeader<GeneratedConsensusFields>,

    metadata: Option<BlockHeaderMetadata>,
}
```

We will discuss each of these top-level fields in detail in the following discussion.

### Application Header

The application header records essential information regarding the operation of the Fuel rollup.

The application header at the high level consists of 4 essential components:

```c++
pub struct ApplicationHeader<Generated> {

    pub da_height: DaBlockHeight,
 
    pub consensus_parameters_version: ConsensusParametersVersion,

    pub state_transition_bytecode_version: StateTransitionBytecodeVersion,

    pub generated: Generated,
}
```

#### da_height

The da_height field records the latest block L1 block until the messages sent from the L1->L2 have been processed; this is helpful later on in fraud proving to establish that a particular message was sent from the L1 to the L2 rollup but wasn’t processed as part of the block that included the messages up to the block of which it was part.

#### consensus_parameters_version

The fuel rollup has a set of upgradeable consensus parameters, which can be upgraded via Transactions of type Upgrade. For each upgrade to these consensus parameters, a new version for consensus_paramters_version has to be assigned, which helps us keep track of which set of consensus parameters we are using while building a particular block.

#### state_transition_bytecode_version

The Fuel rollups keep the WASM compiled bytecode of their state transition function as part of the chain; this facilitates forkless upgrades for the Fuel rollups.

The new state transition function is uploaded via the Upload transactions, while the upgrade is done via the Upgrade transactions. Each upgrade updates the state_transition_bytecode_version, and this version helps keep track of which state transition function is being used to process transactions for a given block.

#### generated

The section contains various rollup-specific fields around execution for a specific block. The Fuel flagship rollup has the following fields for generated:

```c++
pub struct GeneratedApplicationFields {
    /// Number of transactions in this block.
    pub transactions_count: u16,
    /// Number of message receipts in this block.
    pub message_receipt_count: u32,
    /// Merkle root of transactions.
    pub transactions_root: Bytes32,
    /// Merkle root of message receipts in this block.
    pub message_outbox_root: Bytes32,
    /// Root hash of all imported events from L1
    pub event_inbox_root: Bytes32,
}
```

### Consensus Header

The consensus header is another top-level field for the Block Header for Fuel rollups, it is configurable and for the flagship Fuel rollup only keeps track of the hash of the Application Header.

```c++
pub struct GeneratedConsensusFields {
    /// Hash of the application header.
    pub application_hash: Bytes32,
}
```

### Block Header Metadata

The Block Header Metadata is used to track metadata. The current flagship Fuel rollup contains one field that keeps track of block ID, which is the block header's hash.

```c++
pub struct BlockHeaderMetadata {
    /// Hash of the header.
    id: BlockId,
}
```

## Coinbase Transaction

Fuel blocks contain a Coinbase transaction; block producers use Coinbase transactions to collect fees for building blocks. The Coinbase transaction is a Mint transaction, where the mintAmount cannot exceed the fees processed from all transactions in the block. The protocol also requires the Coinbase transaction to always be the last transaction in the block.
