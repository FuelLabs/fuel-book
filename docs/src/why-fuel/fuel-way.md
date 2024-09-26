# The Fuel Way

Since Ethereum launched, blockchains have largely iterated in the same direction. 

Each subsequent chain has presented itself as faster, more scalable, more powerful and more usable. These chains have implemented new consensus mechanisms, new databases, and new ZK proving mechanisms.

Yet the overall system stays roughly the same: developers build smart contracts for their applications and assets (typically in Solidity or Rust). Users connect to a centralized server to read data from the chain, and interact with the chain by signing messages with a standard private-key and sending them back to the same centralized server.

Fuel takes a different approach. We believe the blockchain industry needs a new direction– one focused on decentralization, one built the Fuel way. 

## Decentralized… Sustainably Decentralized

Blockchains fundamentally consist of a network of distributed nodes, all validating new blocks and transactions. The ability for independent, distributed and unqualified actors to participate in this process is what gives blockchains their valuable properties of liveness, censorship resistance and verifiability.

Bitcoin continues to take the most principled stance on maintaining these properties. The low node requirements and low bandwidth usage mean that Bitcoin full nodes can be run on devices as light as Raspberry Pis, and in locations as remote as outer space.

However, subsequent blockchains have all made ongoing compromises. Most newer blockchains today (including most layer-2s) can only be run on high-powered servers with data-center connections. And some high throughput projects remove the key cryptographic primitives of verifiability, such as the merkelization of state elements.

Fuel aims to pull the blockchain space back from this creeping centralization, back towards the values of Bitcoin. The Fuel architecture allows for high performance, while still running on consumer hardware. And furthermore, Fuel always maintains the property of cryptographic verifiability, allowing users to check the state of the chain without trusting third parties.

## Blockchains are not Computers

To truly advance blockchain, we can’t only upgrade the underlying technology of chains. Sometimes, breaking changes are necessary. Fuel believes that realizing the potential of blockchain technology requires changing more than how the blockchains themselves are constructed, but also how the applications built on top of them are built.

Typical smart-contract platforms have treated blockchains like computers, and smart contracts as the programs that run upon these computers. Smart contracts contain the code for various types of read and write operations, and store data to the chain’s state as if it was a global Postgres database.

However, Fuel believes that blockchains themselves are not global computers, but rather “trust machines”. The job of a blockchain node is not to be a cloud server, but rather a device that can verify the current state of the chain, and all future state transitions.

Moving computation off blockchain full nodes and moving data outside of the blockchain’s state allows full node requirements to remain low and blockchains to scale without becoming centralized. Fuel aims to enable developers to build applications following this paradigm without unnecessary complexity.

## ZK Pragmatism

Zero-knowledge technology has captured the imagination of researchers and developers from across the blockchain industry. The promise of succinct verification for arbitrary computation has opened up a whole new range of possibilities for scaling blockchains, making them interoperable, and more. The thesis of building the future of ZK-powered blockchain tech has driven some of the most anticipated and well-funded projects in this space.

However, Fuel takes a more pragmatic perspective in regards to zero-knowledge technology. Of course, we’re as excited as anyone about these groundbreaking new primitives, and their potential within and beyond blockchains. Furthermore, ZK technology is already being integrated into the Fuel stack (such as in Fuel’s hybrid-proving model and with the service chain’s ZK-powered bridge).

Yet we believe that blockchains should be built to be secure, highly-performant, and interoperable with or without the use of ZK technology. Fuel pushed forward with launching the first optimistic rollup on Ethereum, at a time when the majority of Ethereum rollups were focused on building ZK rollups. To this day, we continue to believe that meeting the cost and performance standards demanded by the market can not be sustainably built using full ZK-verification.

The costs of generating these proofs and the time needed to generate them prevent fully ZK-proven chains from being both cost-effective and high-speed. The path towards things like sustainable proofs and “real-time proving” are typically dependent on ZK-specific hardware, which have a series of challenges before they can fully reach production.

Fuel’s approach is to build the best blockchain possible today, using ZK technology off-the-shelf whenever it provides value to the stack. The rise of generalized ZK-VMs like RISC Zero and Succinct’s SP-1 point to a future where ZK technology is commodified and easily available without the need for directly handling the needed cryptography.
