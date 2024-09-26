# Security on Fuel

This section discusses the current security of Fuel rollups, and hence also for Fuel Ignition.

## Fuel Security Council

Fuel currently has a security council that operates various multi-sigs to upgrade the various parts of the stack.

The security council exists because rollups are currently on training wheels, and we need to be highly cautious before making them fully permissionless. It is usually challenging to recover if something is wrong in the network.

Fuel aims to build a stack for type-2 security guarantees and is highly prioritized in our roadmap.

## Block Building

Block building and sequencing are centralized; decentralizing block building requires more work and thought. Block building rights give the builder access to extract MEV from the system, which can impact user transactions and their experience on the network.

We are working on decentralization for block builders and sequencers, which will happen in phases.

Fuel will start with a decentralized sequencer set, i.e., a shared sequencer that block builders can use for all Fuel rollups. Then, it will be followed by decentralized block building.

## Security Attack Vectors

In this section, we list down various attack vectors for the current system and, if applicable, also discuss a path forward to tightening security around them.

### Bridge Contract Bugs

Fuel has a bridge that allows for messaging between L1 and L2; this messaging system creates the base for building a deposit and withdrawal system along with abilities like forced transaction inclusion and calling contracts L2 on L1.

If a bug in the contract implementation on the L1 or L2 compromises the roll-up system, which can include relaying fake messages and transactions from the L1 and L2. A compromise of the mult-sig can also lead to potential malicious upgrades of these contracts.

Fuel follows the rigorous audits of its smart contracts with best-in-class security auditors in the space and also participates in bug bounties to keep the possibility of this very low. These issues become more concerning in stage 2 settings, as the stage 1 setting does allow for reverting many possible issues regarding bridge contracts.

### Layer 2 Client Bugs

The Fuel execution client could have some bugs, which could be exploited to allow behavior not specified.

This can lead to someone being able to create a malicious state and potentially not being challenged by a fraud-proof in case Layer 2 only has one execution client implementation, which can lead to a bug in ZK proving games based on top of it.

Fuel attempts to strengthen security around this by inviting various teams to collaborate on the stack and aiming for multiple implementations, followed by rigorous testing and security audits by the best security organizations in the industry.

### Sway Compiler Bugs

The Sway language is a dominant language built on the Fuel VM. A bug in the Sway compiler could allow malicious bytecode to be part of a particular predicate, smart contract, or script, which the implementation didn’t desire. A similar issue was seen in the ETH ecosystem with Vyper, which you can follow here.

Fuel aims to avoid this by having some of the best talent working on its compiler, followed by rigorous testing and audits by leading security organizations in the industry, in future we also aim to have multiple implementations of the compiler as well, which could help discover a bug in the other implementations.

### Application Level Bugs

Application implementations often have bugs because they avoid some required checks or are built on top of libraries with an underlying bug.

Fuel aims to avoid these by creating best-in-class support libraries in Sway, which are well-audited and tested and, hence, safe to build on. It also promotes the usage of secure patterns through developer evangelism.

### Multisig Compromisation

If compromised, the security council's multi-sig can lead to severe issues, such as malicious upgrades or behavior in various parts of the stack.

Fuel aims to solve this by having a security council with a very high reputation and a lot of social capital attached to it. At the same time, continuous protocol upgrades minimize the need for the security council and always accelerate towards allowing for a stage 2 rollup stack.

### Fraud Proving Bugs

A bug in the fraud-proving implementation can cause challenges and slash for block builders who did build correct blocks or could allow someone to fail to prove a faulty block. This can result in good builders being slashed or wrong states being finalized.

Fuel aims to solve this by initially correcting any such issues with the help of the security council while aiming for multiple implementations of the fraud-proving client or, if possible, multiple-proving system. The implementation is done with best security practices in mind and with regular audits of these systems.
