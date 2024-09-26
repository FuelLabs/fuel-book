# Building on Fuel: An Overview

Building on Fuel empowers developers to create high-performance, scalable decentralized applications with cutting-edge tools and infrastructure. Fuel’s architecture prioritizes speed, security, and developer productivity. This section outlines the core components of the Fuel ecosystem. We will explore each component in further detail in Part 2.

## The FuelVM (Fuel Virtual Machine)

The FuelVM incorporates years of blockchain design to bring the Ethereum community a reliable machine architecture built for longevity. It drives the Fuel Network and delivers exceptional performance by processing transactions in parallel. Unlike most blockchain virtual machines like the Ethereum Virtual Machine (EVM), which execute transactions serially, FuelVM handles concurrent processing, dramatically increasing throughput.

The FuelVM draws on a variety of architectures, including RISC-V, ARM ISAs, Bitcoin scripts, and the Ethereum Virtual Machine, to create a low-level execution environment optimized for blockchain use cases. By offering state-minimized facilities like native assets, ephemeral scripting, and spending conditions, it reduces the load on full nodes, improving network sustainability. Developers can avoid the inefficiencies of traditional state-heavy designs and build applications that deliver high performance while keeping the network decentralized and accessible.

As of May 2024, the FuelVM can achieve asset transfer benchmarks of 12,000 transactions per second (TPS) per core on high-end CPUs, offering unparalleled speed for modern blockchain applications.

## The Fuel Transaction Architecture

Fuel’s transaction architecture brings together lessons from Bitcoin, Ethereum, Cosmos, and Solana to create a highly parallel and efficient transaction model. By using a UTXO (Unspent Transaction Output) model, Fuel enables parallel execution both within and across blocks, allowing developers to process transactions quickly without overloading the network.

Fuel transactions are flexible enough to handle everything from simple asset transfers to complex multi-party, multi-asset interactions and batch smart-contract calls. Developers can build sophisticated applications using advanced conditional logic with predicates, reducing the need for state-heavy smart contracts. By minimizing reliance on state, developers can ensure that applications perform efficiently without overburdening network resources.

Fuel’s transaction model also solves concurrency issues seen in other UTXO-based blockchains. This maintains a familiar developer experience for those coming from Ethereum while benefiting from the performance advantages of UTXO-based execution.

## Fuel Ignition (Rollup)

Fuel Ignition will be the first Fuel V2 rollup to go live on Ethereum Mainnet. It aims to surpass traditional EVM rollups by delivering a vastly improved execution design. Initially starting as a more trustful Layer-2, Ignition’s ultimate goal is to evolve into a fully Ethereum-secured rollup with fraud proving, decentralized sequencing, and secure upgrades via a delayed multi-signature process.

Ignition’s focus on leveraging Ethereum’s security ensures that developers can build high-performance applications while benefiting from the strong security guarantees Ethereum offers. As Ignition develops, it will incorporate decentralized sequencing and Ethereum-based data availability (DA), further enhancing its trustless, scalable design.

## The Fuel Network

Fuel operates as a network of interconnected rollups, designed to offer seamless interaction between different blockchains and rollups. Fuel’s architecture allows projects to decentralize block production through its proposed design for a shared sequencing and builder network. This will provide developers with a fair and efficient way to manage transaction inclusion and interoperation between Fuel rollups.

## Developer Tooling

The Fuel project realized early on the importance of thoughtful and considerate developer tooling. We believe developer time is one of the most important assets we have in our community and it should be optimized for building high value code. In order to make developers maximally productive and build the best possible future proof applications, we built our own suite of tools. These tools make it easy to build, test, and deploy decentralized applications, allowing developers to focus on innovation.

**Sway:** Sway is a domain specific language (DSL) for modern blockchain programming which has familiar syntax, grammar and design ideology to Rust while incorporating blockchain specific functionality such as smart contract interface concepts. Sway is inherently designed to save developers time by providing a single programming language for constructing all critical blockchain application components such as: predicates, scripts, smart contracts, libraries, testing, deployment scripting, indexing and more.

Why not Rust or Solidity? Rust is primarily designed to be a systems language which is heavily bonded to the LLVM toolchain and is not designed for the special considerations of blockchain development. Solidity, while being a powerful language for developing on the Ethereum Virtual Machine, has many known short-comings as a language. Sway aims to incorporate the best of both of these languages to bring developers a both familiar and powerful programming language for blockchain development.

Other tools include:

- **Forc (Fuel Orchestrator):** This command-line toolchain serves as the backbone of Fuel development. It supports everything from compiling Sway smart contracts to managing dependencies and deploying applications. Forc simplifies the entire development process, ensuring that developers can build robust dApps with ease.

- **Fuel Rust SDK:** The Rust SDK allows developers to interact with Fuel’s blockchain using the Rust programming language. It offers a seamless experience for creating system-level applications and managing interactions with the Fuel Network.

- **Fuel Wallet SDK:** The Fuel Wallet SDK provides developers with the tools to create secure, user-friendly wallets that natively interact with the Fuel ecosystem. It ensures developers can easily build wallets that integrate into decentralized applications.

- **Fuel Typescript SDK:** The Typescript SDK allows developers to integrate Fuel into web applications. It simplifies interaction with the Fuel blockchain, making it easy for frontend developers to build decentralized applications that interact with Fuel’s infrastructure.