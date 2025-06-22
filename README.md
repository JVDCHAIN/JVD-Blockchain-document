# Overview

### JUVIDOE Chain <a href="#credit-smart-chain" id="credit-smart-chain"></a>

JUVIDOE Chain is a modular and extensible framework for building Juvidoe-compatible blockchain networks, sidechains, and general scaling solutions.

Its primary use is to bootstrap a new blockchain network while providing full compatibility with Juvidoe smart contracts and transactions. It uses IBFT (Istanbul Byzantine Fault Tolerant) consensus mechanism, supported in one flavour as [PoS (proof of stake)](https://juvidoe.gitbook.io/juvidoe-blockchain-docs/consensus/proof-of-stake).

JUVIDOE Chain also supports communication with multiple blockchain networks, enabling transfers of both [JRC-20](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/) and [JRC-721](https://ethereum.org/en/developers/docs/standards/tokens/erc-721) tokens, by utilising the centralised bridge solution.

Industry standard wallets can be used to interact with JUVIDOE Chain through the [JSON-RPC](https://juvidoe.gitbook.io/juvidoe-blockchain-docs/get-started/json-rpc-commands) endpoints and node operators can perform various actions on the nodes through the gRPC protocol.

To find out more about JUVIDOE Chain, visit the [official website](https://github.com/JVDCHAIN).

[**GitHub repository**](https://github.com/JVDCHAIN/JVD-Blockchain)

{% hint style="danger" %}
**CAUTION**

This is a work in progress so architectural changes may happen in the future, so please contact the JUVIDOE team if you would like to use it in production.
{% endhint %}

To get started by running a`juvidoe-edge` network locally, please read: [Installation](https://juvidoe.gitbook.io/juvidoe-blockchain-docs/get-started/installation) and [Local Setup](https://juvidoe.gitbook.io/juvidoe-blockchain-docs/get-started/local-setup).
