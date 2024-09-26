# SNAP Fast Finality Gadget

One of the most well-known limitations of rollups is the long periods of time needed to send messages from the layer-2 back up to the layer-1 chain. Notoriously, users of optimistic rollups like Arbitrum and Optimism must wait around a week to withdraw assets from the network. While ZK-rollups do offer reduced settlement times, the economics of proving mean that proofs (and therefore withdrawals) can still take up to 24 hours to process.

Fuel has proposed a theoretical new mechanism for substantially reducing the finality time of optimistic rollups. While the full details are outlined in a post on ETHResearch, the mechanism can be understood at a high level as using a bonded committee of operators to attest to both the validity of the chain, as well as the state of Ethereum not being censored.
