# Architecture Overview

We started with the idea of making software that is _modular_.

This is something that is present in almost all parts of the JVMBuilder Edge. Below, you will find a brief overview of the built architecture and its layering.

### Juvidoe Edge Layering

!\[Juvidoe Edge Architecture]

### Libp2p

It all starts at the base networking layer, which utilizes **libp2p**. We decided to go with this technology because it fits into the designing philosophies of Juvidoe Edge. Libp2p is:

* Modular
* Extensible
* Fast

Most importantly, it provides a great foundation for more advanced features, which we'll cover later on.

### Synchronization & Consensus

The separation of the synchronization and consensus protocols allows for modularity and implementation of **custom** sync and consensus mechanisms - depending on how the client is being run.

Juvidoe Edge is designed to offer off-the-shelf pluggable consensus algorithms.

The current list of supported consensus algorithms:

* IBFT PoS

### Blockchain[​](https://polygon-edge-v063.evmbuilder.com/docs/architecture/overview#blockchain) <a href="#blockchain" id="blockchain"></a>

The Blockchain layer is the central layer that coordinates everything in the Juvidoe Edge system. It is covered in depth in the corresponding _Modules_ section.

### State[​](https://polygon-edge-v063.evmbuilder.com/docs/architecture/overview#state) <a href="#state" id="state"></a>

The State inner layer contains state transition logic. It deals with how the state changes when a new block is included. It is covered in depth in the corresponding _Modules_ section.

### JSON RPC

The JSON RPC layer is an API layer that dApp developers use to interact with the blockchain. It is covered in depth in the corresponding _Modules_ section.

### TxPool

The TxPool layer represents the transaction pool, and it is closely linked with other modules in the system, as transactions can be added from multiple entry points.

### GRPC

The GRPC layer is vital for operator interactions. Through it, node operators can easily interact with the client, providing an enjoyable UX.
