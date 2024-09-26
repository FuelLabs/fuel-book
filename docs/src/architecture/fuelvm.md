# The FuelVM

The FuelVM lies at the core of the whole Fuel stack; it was created by taking into account  years of learning from other virtual machine designs, like the EVM, Solana’s SVM and more.

The FuelVM enables developers to move away from stateful application designs often enabled by smart contracts by providing more feature-rich state-minimized facilities such as native assets, ephemeral scripting, and ephemeral spending conditions. By providing alternative methods for developers to build more state-minimized applications, we can achieve the aims of better full node sustainability, keeping the architecture decentralized and accessible to all, following in the footsteps of Ethereum’s core values.

In the following sections, we will discuss the key features of the FuelVM in detail.

## UTXO Model and Parallelization

Fuel’s parallelized transaction execution model is a cornerstone of its efficiency and scalability. Parallelization dramatically improves throughput and reduces latency compared to traditional sequential processing methods. It allows tasks to be broken down into smaller sub-tasks that can be executed simultaneously on multiple processing units.
Parallelization is built upon a foundation of Access Lists and the UTXO (Unspent Transaction Output) model, which works in tandem to enable concurrent processing of non-conflicting transactions.
In other words, Fuel leverages the UTXO model for performing transactions on Fuel, anything be it from a token transfer, to a smart contract call, all are done via the transactions that are modelled via UTXOs. 

This means that addresses on Fuel own unspent coins, which they can spend to conduct transactions via the FuelVM.

Using the UTXO model helps achieve transaction parallelization. At runtime, the user must provide the inputs and outputs they want to produce as part of their transaction. This means that transactions that don’t overlap can be processed in parallel, allowing Fuel to scale horizontally with the number of cores per machine.

## Register based design

The FuelVM is a register-based virtual machine, unlike the EVM (and many others), which is a stack-based virtual machine.

Register-based virtual machines have been shown to be more performant than Stack-based virtual machines.

The FuelVM has 64 registers, each 8 bytes, of which 16 are reserved and are 6 bits addressable.

| value | register |         name        |                                    description                                   |
|:-----:|:--------:|:-------------------:|:--------------------------------------------------------------------------------:|
| 0x00  | $zero    | zero                | Contains zero (0), for convenience.                                              |
| 0x01  | $one     | one                 | Contains one (1), for convenience.                                               |
| 0x02  | $of      | overflow            | Contains overflow/underflow of addition, subtraction, and multiplication.        |
| 0x03  | $pc      | program counter     | The program counter. Memory address of the current instruction.                  |
| 0x04  | $ssp     | stack start pointer | Memory address of bottom of current writable stack area.                         |
| 0x05  | $sp      | stack pointer       | Memory address on top of current writable stack area (points to free memory).    |
| 0x06  | $fp      | frame pointer       | Memory address of beginning of current call frame.                               |
| 0x07  | $hp      | heap pointer        | Memory address below the current bottom of the heap (points to used/OOB memory). |
| 0x08  | $err     | error               | Error codes for particular operations.                                           |
| 0x09  | $ggas    | global gas          | Remaining gas globally.                                                          |
| 0x0A  | $cgas    | context gas         | Remaining gas in the context.                                                    |
| 0x0B  | $bal     | balance             | Received balance for this context.                                               |
| 0x0C  | $is      | instructions start  | Pointer to the start of the currently-executing code.                            |
| 0x0D  | $ret     | return value        | Return value or pointer.                                                         |
| 0x0E  | $retl    | return length       | Return value length in bytes.                                                    |
| 0x0F  | $flag    | flags               | Flags register.                                                                  |

## The FuelVM Instruction Set

The FuelVM instructions are 4 bytes wide and have the following structure:
- Opcode: 8 bits
- Register Identifier: 6 bits
- Immediate value: 12, 18, or 24 bits, depending on the operation.

The FuelVM instruction set has been documented in detail here: https://docs.fuel.network/docs/specs/fuel-vm/instruction-set.

## Memory

FuelVM has byte-indexed memory, which can be configured with the VM_MAX_RAM parameter. Hence, each instance of FuelVM can decide how much memory it wants to allocate for the VM.

Memory has a stack and a heap model. The stack starts from the left, after the initialized VM data and call frame in case of a call context, while the heap starts at the byte at index VM_MAX_RAM.

Each byte allocation on the Stack increases the stack index by 1, and each byte allocation on the heap decreases its writable index by 1. Hence, the stack grows upwards, and the heap grows downwards.

The stack and the heap have the following essential registers associated with them:
- $ssp ( 0x05 ): Memory address of bottom of the current writable stack area.
- $sp ( 0x06 ): Memory address on top of current writable stack area (points to free memory).
- $hp ( 0x07 ): Memory address below the current bottom of the heap (points to used/OOB memory).

The FuelVM has ownership checks to ensure that contexts have a defined sense of ownership over particular regions in the memory and can only access memory from the region they own. We will elaborate more on this topic in later sections.

## Predicates, Scripts and Contracts

To be able to understand further concepts for Fuel, we need to understand:

- Predicates
- Scripts
- Contracts

Let’s dive deeper.

### Predicates

Predicates are stateless programs that define spending conditions for native assets. Native assets can be sent to predicates, and then, to determine whether these assets can be spent as part of a transaction or not, the FuelVM can execute their bytecode and check the return value for a boolean. If the returned value is true, the asset can be spent; if the returned value is false, then the transaction is invalid!

People can program various spending conditions; for example, spend only if three out of five have signed over a transaction or if a transaction has specific desired inputs and outputs { popularly referred to as intents }.

Predicates are stateless, meaning they don’t have persistent storage attached. At the same time, they cannot call other smart contracts.

### Scripts

Scripts are the entrypoint for Fuel transactions that dictate what happens in a transaction; like predicates, they lack the ability to have persistent storage. However, they can call contract Inputs, which are part of the Fuel transaction, which can have persistent storage of their own.

This gives Fuel the ability to support interesting features like Multi-calls, the conditional calling of contracts, and many more natively.

### Contracts

Fuel provides support for smart contracts in its UTXO model. Contracts are stateful and can be called other contracts. In Fuel Smart contracts are representative by InputContract type, you can refer more to the section on InputContract to know further. 

The first call to a contract in a transaction has to be made via a script, and then the contract can call other contracts.

Contracts have persistent storage, a key-value pair of 32-byte and 32-byte values. Various data structures are considered to determine the best way to commit to contract storage.

## Contexts

A context is a way to isolate the execution of various execution environments for predicate estimation and verification, scripts, and contracts. Each context has its memory ownership, which we will expand on later.

There are four types of contexts in Fuel:

- Predicate Estimation
- Predicate Verification
- Script Execution
- Calls

The first three are called External contexts, as the $fp is zero, while Calls are called Internal contexts and will have a non-zero value for $fp.

### Predicate Estimation

Fuel transactions provide predicateGasUsed  per predicate used in the transaction; this is so that during verification of a predicate, if predicateGasUsed is less than the total gas consumed during verification, then the transaction can be reverted.

The user performs Predictive Estimation either locally or by calling a remote full node, which executes the predicate in the FuelVM and returns the total gas consumed for the same.

Predicate estimation context cannot do persistent storage or make calls to Contracts.

### Predicate Verification

All predicate parts of the transaction are verified to return true before executing the transaction script. So, the FuelVM is used in the Predicate Verification context when verifying the predicates of a transaction.

Predicate verification context cannot do persistent storage or make calls to contracts.

### Script Execution

After verifying all predicates, the transaction script is executed; the script execution context cannot do persistent storage but can make calls to contract.

### Calls

Call contexts are for executing contracts; they are flexible, can store data persistently, and can make contract calls.

Call context can be created by either:

1. Script calling a smart contract
2. Contract calling a contract input

Each call creates a “Call Frame”, which is pushed to the Stack. A call frame holds metadata on the stack, aiding the execution of the call context in the FuelVM. A call context cannot mutate the state of the caller and only access its own stack and heap.


|          bytes          |     type    |   value  |                                  description                                  |
|:-----------------------:|:-----------:|:--------:|:-----------------------------------------------------------------------------:|
| Unwritable area begins. |             |          |                                                                               |
| 32                      | byte[32]    | to       | Contract ID for this call.                                                    |
| 32                      | byte[32]    | asset_id | asset ID of forwarded coins.                                                  |
| 8*64                    | byte[8][64] | regs     | Saved registers from previous context.                                        |
| 8                       | uint64      | codesize | Code size in bytes, padded to the next word boundary.                         |
| 8                       | byte[8]     | param1   | First parameter.                                                              |
| 8                       | byte[8]     | param2   | Second parameter.                                                             |
| 1*                      | byte[]      | code     | Zero-padded to 8-byte alignment, but individual instructions are not aligned. |
|  Unwritable area ends.  |             |          |                                                                               |
| *                       |             |          | Call frame's stack.                                                           |

After a call context has successfully ended, its call frame is popped from the stack. However, heap-allocated space; it persists in memory.

A call context returns its value with the help of the $ret and $retl registers. Large return values can be written to the heap and later read from the caller contexts.

## Memory Policies

After understanding the various contexts under which the FuelVM can be executed, it is time to discuss the policies around reading and writing for contexts.

### Read Policies for Context

A context can read any data from the stack in the range from the byte at index 0 (i.e., from the start of the memory ) to the highest ever $sp and between the current $hp and VM_MAX_RAM (i.e., until the end of the memory ) of the previous context that created the current context.

Any attempt to read from the region between the highest ever $sp during the context execution and the current $hp will return into an error.

Now, what do we mean by the highest ever $sp?

Since the stack can be grown and shrunk in size, it is possible that during the execution of some context, the $sp went until, for example, index 1000, but then elements were popped out of the stack, and now the current $sp is 900. In this scenario, the highest ever $sp during the execution of this call context is 1000, and hence, the memory region until 1000 is readable for the stack!

### Write Policies for Context.

A given context can write to any region between its $ssp and current $hp; hence, that region of memory can be allocated and used for writing data. 

Before writing to this memory region, the bytes have to be allocated first. In the case of a stack, this is done using CFE and CFEI opcodes, while in the case of the heap, it is done via an ALOC opcode.

Note: Remember that once a context completes, all values on the stack (i.e., the call frame and all values allocated on the stack during execution ) are wiped down. Still, heap allocation stays, and the next context can only write data below the $hp of the existing context.

## VM Initialization & Configuration

### Configuration

The VM can be configured by setting the following parameters:

|          name         |  type  | value |                   note                  |
|:---------------------:|:------:|:-----:|:---------------------------------------:|
| CONTRACT_MAX_SIZE     | uint64 |       | Maximum contract size, in bytes.        |
| VM_MAX_RAM            | uint64 | 2**26 | 64 MiB.                                 |
| MESSAGE_MAX_DATA_SIZE | uint64 |       | Maximum size of message data, in bytes. |


### VM Initialization

This section describes what happens during the VM initialization for each VM run.
To initialize the VM, the following is pushed on the stack sequentially:

1. Transaction hash (byte[32], word-aligned), computed as defined here.
2. Base asset ID (byte[32], word-aligned)
3. MAX_INPUTS pairs of (asset_id: byte[32], balance: uint64), of:
    1. For predicate estimation and predicate verification, zeroes.
    2. For script execution, the free balance for each asset ID seen in the transaction's inputs is ordered in ascending order. If there are fewer than MAX_INPUTS asset IDs, the pair has a zero value.
4. Transaction length, in bytes (uint64, word-aligned).
5. The transaction, serialized.

Then, the following registers are initialized (without explicit initialization, all registers are initialized to zero):
1. $ssp = 32 + 32 + MAX_INPUTS*(32+8) + size(tx)): the writable stack area starts immediately after the serialized transaction in memory (see above).
2. $sp = $ssp: writable stack area is empty to start.
3. $hp = VM_MAX_RAM: the heap area begins at the top and is empty to start.

## Further Readings

- Nick Dodson’s tweet on what makes FuelVM unique: https://x.com/IAmNickDodson/status/1542516357886988288
- Blockchain Capital’s blog on FuelVM and Sway: https://medium.com/blockchain-capital-blog/exploring-the-fuelvm-86cf9ccdc159
- UTXO Mode by River.com: https://river.com/learn/bitcoins-utxo-model/