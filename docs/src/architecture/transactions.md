# Transactions on Fuel

Fuel uses the UTXO model for transactions on its blockchain. The model is popularly used in the Bitcoin protocol and has various advantages, including parallel transaction execution.

In Fuel, addresses can own native assets and then spend these coins with transactions.

In Fuel, transactions fall under five categories determined by the operation they are performing in the blockchain:

1. Script
2. Create
3. Mint
4. Upgrade
5. Upload

The Fuel follows the UTXO model for its transactions, because of which it has some constructs that we will discuss before we dive deeper into the various types of transactions:

- Inputs
- Script
- Outputs
- Witnesses

We will discuss these components of a Fuel transaction in detail and then move towards discussing these individual transaction types.

## Inputs

Inputs are state elements that users access in the transaction; there are three types of Inputs in Fuel:

1. Coins
2. Contracts
3. Messages

### Coins

Coins are units for some asset that a user can spend as part of the transaction. Fuel supports native assets, which means that, unlike chains which just support one base asset (like ETH for Ethereum), creating assets is native to Fuel and enshrined in the protocol (see in the native assets section in the appendix for this section).

Users can own various denominations of some asset in different numbers of Coins; for example, a Fuel address A can have a balance of some asset 100, with four coins of 25 denominations each, and some address B can have a balance of 100 for the same asset, but three coins of denomination 10, 40, 50.

An Input Coin has the following parameters attached to it:

| name        | type      | description                                   |
|-------------|-----------|-----------------------------------------------|
| txID        |  byte[32] | Hash of transaction.                          |
| outputIndex | uint16    | Index of transaction output.                  |
| owner       | byte[32]  | Owning address or predicate root.             |
| amount      | uint64    | Amount of coins.                              |
| asset_id    | byte[32]  | Asset ID of the coins.                        |
| txPointer   | TXPointer | Points to the TX whose output is being spent. |

| name                | type   | description                                         |
|---------------------|--------|-----------------------------------------------------|
| witnessIndex        | uint16 | Index of witness that authorizes spending the coin. |
| predicateGasUsed    | uint64 | Gas used by predicate.                              |
| predicateLength     | uint64 | Length of predicate, in instructions.               |
| predicateDataLength | uint64 | Length of predicate input data, in bytes.           |
| predicate           | byte[] | Predicate bytecode.                                 |
| predicateData       | byte[] | Predicate input data (parameters).                  |

The transaction invalidity rules for this input type can be seen here.

### Contracts

One question frequently arises with the UTXO model is how to achieve smart contracts beyond ephemeral scripts;

One of the issues with Bitcoin is lack of complex smart contract feature, expanding a little bit more on this, the core issue arrives because of the following:

- Bitcoin script is not turning complete, which means you cannot do things like loops inside bitcoin
- Bitcoin doesn’t support persistent storage for bitcoin script being run in the transaction, which limits what u can do with Bitcoin

There is a common misconception that these issues in Bitcoin are there because of the UTXO model, but rather it is a conscious decision Bitcoin made and we at Fuel support full turning complete smart contracts, with persistent storage.

Fuel solves this problem by making stateful smart contracts an input for Fuel transactions!

Contracts have persistent storage attached to them and can own native assets. Every time users want to consume them, they use them as input for the transaction. Then, they can call various external functions attached to them via the ephemeral script attached to the transaction.

| name        | type      | description                                                             |
|-------------|-----------|-------------------------------------------------------------------------|
| txID        |  byte[32] | Hash of transaction.                                                    |
| outputIndex | uint16    | Index of transaction output.                                            |
| balanceRoot | byte[32]  | Root of amount of coins owned by contract before transaction execution. |
| stateRoot   | byte[32]  | State root of contract before transaction execution.                    |
| txPointer   | TXPointer | Points to the TX whose output is being spent.                           |
| contractID  | byte[32]  | Contract ID.                                                            |

When signing over them, txID, outputIndex, balanceRoot, stateRoot, and txPointer are initialized to zero values, which the block builder later fills in. This helps avoid concurrency issues with Contracts as previously seen in the Cardano model.

Contracts can be called during the script's execution, leading to changes in various fields, like the balanceRoot in case of a change in coins owned by Contract, or if some state change happens, the stateRoot will change. Etc.

So, suppose you want to use an AMM contract on Fuel. In that case, you will include it as an input to your transaction, call the external methods in the ephemeral script, and then emit the contract as an output, which can then be consumed for the next transaction for this particular contract.

The transaction invalidity rules for this input type can be seen here.

### Messages

The Block Builder creates messages created as part of sending messages from the L1 to the L2. Messages make deposits to the Fuel rollups from the L1 possible, and we will discuss them in better detail later in the Fuel & Ethereum section.

**NOTE:** An Input Message can only be consumed as an Input as part of a transaction, are then destroyed from the UTXO set.

A fuel InputMessage consists of the following parameters:

|         name        |   type   |                       description                       |
|:-------------------:|:--------:|:-------------------------------------------------------:|
| sender              | byte[32] | The address of the message sender.                      |
| recipient           | byte[32] | The address or predicate root of the message recipient. |
| amount              | uint64   | Amount of base asset coins sent with message.           |
| nonce               | byte[32] | The message nonce.                                      |
| witnessIndex        | uint16   | Index of witness that authorizes spending the coin.     |
| predicateGasUsed    | uint64   | Gas used by predicate execution.                        |
| dataLength          | uint64   | Length of message data, in bytes.                       |
| predicateLength     | uint64   | Length of predicate, in instructions.                   |
| predicateDataLength | uint64   | Length of predicate input data, in bytes.               |
| data                | byte[]   | The message data.                                       |
| predicate           | byte[]   | Predicate bytecode.                                     |
| predicateData       | byte[]   | Predicate input data (parameters).                      |

The transaction invalidity rules for this input type can be seen here.

## Scripts

Fuel scripts are ephemeral scripts that express the various actions taken during a transaction; a Script can call the contracts provided as inputs or perform any other arbitrary computation!

Multi-call in Fuel is achieved via scripts, because users can provide five contracts as inputs and call external methods on five of them together in the same transaction or call five external methods of a single contract output in the same transaction.

One thing to remember, as mentioned in the FuelVM section, is that when the FuelVM is in the Script Context, there is no ability for scripts to have their own persistent storage.

## Outputs

Fuel transactions have Outputs, which define the creation of new UTXOs post-transaction; these Outputs can then be inputs for the next set of transactions.

There are five types of possible Output types in a Fuel transaction:

1. Coin
2. Contract
3. Change
4. Variable
5. ContractCreated

One things to note is that we have 3 Outputs which deal with the Coins, and the table below summarizes the core differences between them (we will expand more in further sections):

|         | OutputCoin | OutputChange      | OutputVariable         |
|---------|------------|-------------------|------------------------|
| Amount  | Static     | Automatically set | Set by script/contract |
| AssetID | Static     | Static            | Set by script/contract |
| To      | Static     | Static            | Set by script/contract |

**NOTE:** A Coin Output (Coin, Change,Variable) has an amount zero that leads to the pruning of the output from the UTXO set, which means coin outputs of amount 0 are not part of the UTXO set.

Let’s go through them one by one.

### OutputCoin

Output Coins are new coins sent to a Fuel Address, which can then be spent as Input Coins in further transactions.

|   name   |   type   |              description             | OutputVariable         |
|:--------:|:--------:|:------------------------------------:|------------------------|
| to       | byte[32] | Receiving address or predicate root. | Set by script/contract |
| amount   | uint64   | Amount of coins to send.             | Set by script/contract |
| asset_id | byte[32] | Asset ID of coins.                   | Set by script/contract |

The transaction invalidity rules for this output type can be seen here.

### OutputContract

OutputContracts are new contract outputs that can then be used as InputContracts for a specific contract ID in the next transaction that uses this contract as an Input. They contain the newly updated index, balanceRoot, and stateRoot of the contract after being processed as part of the transaction.

**Note:** Every InputContract that is part of the transaction, always needs to have a corresponding Output Contract.

|     name    |   type   |                               description                              | OutputVariable         |
|:-----------:|:--------:|:----------------------------------------------------------------------:|------------------------|
| inputIndex  | uint16   | Index of input contract.                                               | Set by script/contract |
| balanceRoot | byte[32] | Root of amount of coins owned by contract after transaction execution. | Set by script/contract |
| stateRoot   | byte[32] | State root of contract after transaction execution.                    | Set by script/contract |

The transaction invalidity rules for this output type can be seen here.

### OutputChange

An OutputChange present as one of our outputs for a particular assetId allows us to recover any unused balance from the total balance of inputs provided as part of the transaction for that assetId.

For example, an OutputChange can collect any ETH not spent as gas, or any USDC not swapped as part of a DEX transaction.

**Note:** There can only be one OutputChange per asset_id in a transaction.

|   name   |   type   |              description             | OutputVariable         |
|:--------:|:--------:|:------------------------------------:|------------------------|
| to       | byte[32] | Receiving address or predicate root. | Set by script/contract |
| amount   | uint64   | Amount of coins to send.             | Set by script/contract |
| asset_id | byte[32] | Asset ID of coins.                   | Set by script/contract |

The transaction invalidity rules for this output type can be seen here.

### OutputVariable

OutputVariable acts as a placeholder for coins created in the execution of scripts and contracts, since they can create a coin of arbitrary amount and to an arbitrary user, This is useful in scenarios where the exact output amount and owner cannot be determined beforehand.

**Note:** This means every transaction that makes uses of mint internally will need an OuputVariable for that particular assetID.

This can be very helpful in various scenarios. One example is where the contract transfers its output coin to a user only if the user provides a correct value; in that scenario, we can put a variable output, which can or cannot have some value attached to it at the end!

Variable Outputs can be used via the TRO opcode.

The transaction invalidity rules for this output type can be seen here.

|   name   |   type   |              description             |
|:--------:|:--------:|:------------------------------------:|
| to       | byte[32] | Receiving address or predicate root. |
| amount   | uint64   | Amount of coins to send.             |
| asset_id | byte[32] | Asset ID of coins.                   |

### OutputContractCreated

The OutputContractCreated output indicates that a new contract was created as part of the transaction. The parameters include the contractID and the initial state root for this contract.

|    name    |   type   |           description           |
|:----------:|:--------:|:-------------------------------:|
| contractID | byte[32] | Contract ID.                    |
| stateRoot  | byte[32] | Initial state root of contract. |
| asset_id   | byte[32] | Asset ID of coins.              |

The transaction invalidity rules for this output type can be seen here.

## Witness

The witness is a parameter attached to transactions. Witnesses are filled in by the block builders and are not part of the transaction ID. A Witness is usually used to provide digital signatures for verification purposes, for example, the signature to prove the spending of Coin or anything else.

Witnesses are not part of the transaction id, which allows for someone to able to sign over a transaction and provide it as part of the transaction.

**Note:** The protocol doesn’t require witnesses to be used only for providing signatures; they can be used to fill any data and can be leveraged for various interesting use cases, like State Rehydration.

Each witness contains a byte array data along with the field dataLength helping know the length of this data.

|    name    |   type   |            description            |
|:----------:|:--------:|:---------------------------------:|
| dataLength | uint64   | Length of witness data, in bytes. |
| data       | byte[]   | Witness data.                     |
| asset_id   | byte[32] | Asset ID of coins.                |

Multiple witnesses can be provided as part of the transaction, and the inputs can indicate which witness block builders, contracts, scripts or predicates can look at to verify the validity of being able to spend the input by providing the index at which their witness lives.

## TransactionScript

Script transactions are transactions that, as the name suggests, have Inputs, Outputs, and a Script that dictates what happens as part of the transaction.

Note: Scripts are optional in transactions of type TransactionScript; for example, a simple token transfer can work only on inputs and outputs, with no requirement for a script. So, scripts are mainly leveraged when you want to do other things as part of your transaction beyond simply transferring or burning assets.

The transaction's script can compute arbitrary amounts and call other contracts. A famous example of script transactions is using an AMM or doing a token transfer.

|       name       |    type    |               description               |
|:----------------:|:----------:|:---------------------------------------:|
| scriptGasLimit   | uint64     | Gas limits the script execution.        |
| receiptsRoot     | byte[32]   | Merkle root of receipts.                |
| scriptLength     | uint64     | Script length, in instructions.         |
| scriptDataLength | uint64     | Length of script input data, in bytes.  |
| policyTypes      | uint32     | Bitfield of used policy types.          |
| inputsCount      | uint16     | Number of inputs.                       |
| outputsCount     | uint16     | Number of outputs.                      |
| witnessesCount   | uint16     | Number of witnesses.                    |
| script           | byte[]     | Script to execute.                      |
| scriptData       | byte[]     | Script input data (parameters).         |
| policies         | Policy []  | List of policies, sorted by PolicyType. |
| inputs           | Input []   | List of inputs.                         |
| outputs          | Output []  | List of outputs.                        |
| witnesses        | Witness [] | List of witnesses.                      |

**Note:** It is important to note that a script transaction cannot create a contract and hence cannot have output of type ContractCreated. Other transaction invalidity rules can be viewed here.

## TransactionCreate

TransactionCreate is used to create new Contracts; the parameters allow for contracts with initialized storage slots.

The contract ID of smart contracts on Fuel is calculated deterministically, and the calculation mechanism is referred to here.

|         name         |          type          |                    description                    |
|:--------------------:|:----------------------:|:-------------------------------------------------:|
| bytecodeWitnessIndex | uint16                 | Witness index of contract bytecode to create.     |
| salt                 | byte[32]               | Salt.                                             |
| storageSlotsCount    | uint64                 | Number of storage slots to initialize.            |
| policyTypes          | uint32                 | Bitfield of used policy types.                    |
| inputsCount          | uint16                 | Number of inputs.                                 |
| outputsCount         | uint16                 | Number of outputs.                                |
| witnessesCount       | uint16                 | Number of witnesses.                              |
| storageSlots         | (byte[32], byte[32])[] | List of storage slots to initialize (key, value). |
| policies             | Policy []              | List of policies.                                 |
| inputs               | Input []               | List of inputs.                                   |
| outputs              | Output []              | List of outputs.                                  |
| witnesses            | Witness []             | List of witnesses.                                |

The transaction invalidity rules for this transaction type can be seen here.

## TransactionMint

The block producer uses this transaction to mint new assets and doesn’t require any signature. The transaction is currently used to create the fees for the block producer. The last transaction in the blocks is a coinbase transaction, allowing the block producer to collect fees for building the block.

|      name      |      type      |                                 description                                |
|:--------------:|:--------------:|:--------------------------------------------------------------------------:|
| txPointer      | TXPointer      | The location of the Mint transaction in the block.                         |
| inputContract  | InputContract  | The contract UTXO that assets are minted to.                               |
| outputContract | OutputContract | The contract UTXO that assets are being minted to.                         |
| mintAmount     | uint64         | The amount of funds minted.                                                |
| mintAssetId    | byte[32]       | The asset IDs corresponding to the minted amount.                          |
| gasPrice       | uint64         | The gas price to be used in calculating all fees for transactions on block |

The transaction invalidity rules for this transaction type can be seen here.

## TransactionUpgrade

The Fuel network has some consensus parameters that can be upgraded occasionally; the state transition function of the Fuel network is stored on-chain and hence can be upgraded by privileged addresses.

Hence, at TransactionUpgrade at any given instance, could be attempting to do any of the following:

- Trying to upgrade the consensus parameters
- Trying to upgrade the state transition function

|      name      |      type      |           description          |
|:--------------:|:--------------:|:------------------------------:|
| upgradePurpose | UpgradePurpose | The purpose of the upgrade.    |
| policyTypes    | uint32         | Bitfield of used policy types. |
| inputsCount    | uint16         | Number of inputs.              |
| outputsCount   | uint16         | Number of outputs.             |
| witnessesCount | uint16         | Number of witnesses.           |
| policies       | Policy []      | List of policies.              |
| inputs         | Input []       | List of inputs.                |
| outputs        | Output []      | List of outputs.               |
| witnesses      | Witness []     | List of witnesses.             |

The transaction invalidity rules for this transaction type can be seen here.

## TransactionUpload

Before an upgrade can be performed, the fuel state transition bytecode must be uploaded to the chain. This requires uploading the bytecode via multiple transactions. TransactionUpload allows us to split the bytecode into multiple subsections and upload each subsection sequentially over multiple transactions.

Once all subsections have been successfully uploaded, the transaction can be considered final, and then the new bytecode can be used.

|        name       |    type    |                                      description                                      |
|:-----------------:|:----------:|:-------------------------------------------------------------------------------------:|
| root              | byte[32]   | The root of the Merkle tree is created over the bytecode.                             |
| witnessIndex      | uint16     | The witness index of the subsection of the bytecode.                                  |
| subsectionIndex   | uint16     | The index of the subsection of the bytecode.                                          |
| subsectionsNumber | uint16     | The total number of subsections on which bytecode was divided.                        |
| proofSetCount     | uint16     | Number of Merkle nodes in the proof.                                                  |
| policyTypes       | uint32     | Bitfield of used policy types.                                                        |
| inputsCount       | uint16     | Number of inputs.                                                                     |
| outputsCount      | uint16     | Number of outputs.                                                                    |
| witnessesCount    | uint16     | Number of witnesses.                                                                  |
| proofSet          | byte[32][] | The proof set of Merkle nodes to verify the connection of the subsection to the root. |
| policies          | Policy []  | List of policies.                                                                     |
| inputs            | Input []   | List of inputs.                                                                       |
| outputs           | Output []  | List of outputs.                                                                      |
| witnesses         | Witness [] | List of witnesses.                                                                    |

The transaction invalidity rules for this transaction type can be seen here.

## Appendix

### Native Assets

In Fuel, apart from Eth { which is called the base Asset }, the functionality of minting and burning assets is enshrined in the protocol! The FuelVM provides op-codes for creating minting and burning assets, MINT and BURN, respectively.

All native assets can be spent using similar rules as the base asset, which allows Fuel developers and users to fully utilize the UTXO model and the resulting parallelization.

To explore Native assets further, it is recommended that you look at Sway Standards, which provides various standards ( like SRC-3, SRC-20, and many more ) related to native assets.
