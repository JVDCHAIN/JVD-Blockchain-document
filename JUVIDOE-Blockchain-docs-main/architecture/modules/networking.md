# Networking

### Overview

A node has to communicate with other nodes on the network, in order to exchange useful information.\
To accomplish this task, the Juvidoe Edge leverages the battle-tested **libp2p** framework.

The choice to go with **libp2p** is primarily focused on:

* **Speed** - libp2p has a significant performance improvement over devp2p (used in GETH and other clients)
* **Extensibility** - it serves as a great foundation for other features of the system
* **Modularity** - libp2p is modular by nature, just like the Juvidoe Edge. This gives greater flexibility, especially when parts of the Juvidoe Edge need to be swappable

### GRPC

On top of **libp2p**, the Juvidoe Edge uses the **GRPC** protocol.\
Technically, the Juvidoe Edge uses several GRPC protocols, which will be covered later on.

The GRPC layer helps abstract all the request/reply protocols and simplifies the streaming protocols needed for the Juvidoe Edge to function.

GRPC relies on **Protocol Buffers** to define _services_ and _message structures_.\
The services and structures are defined in _.proto_ files, which are compiled and are language-agnostic.

Earlier, we mentioned that the Juvidoe Edge leverages several GRPC protocols.\
This was done to boost the overall UX for the node operator, something which often lags with clients like GETH and Parity.

The node operator has a better overview of what is going on with the system by calling the GRPC service, instead of sifting through logs to find the information they're looking for.

#### GRPC for Node Operators

The following section might seem familiar because it was briefly covered in the CLI Commands section.

The GRPC service that is intended to be used by **node operators** is defined like so:

minimal/proto/system.proto

```
service System {
    // GetInfo returns info about the client
    rpc GetStatus(google.protobuf.Empty) returns (ServerStatus);

    // PeersAdd adds a new peer
    rpc PeersAdd(PeersAddRequest) returns (google.protobuf.Empty);

    // PeersList returns the list of peers
    rpc PeersList(google.protobuf.Empty) returns (PeersListResponse);

    // PeersInfo returns the info of a peer
    rpc PeersStatus(PeersStatusRequest) returns (Peer);

    // Subscribe subscribes to blockchain events
    rpc Subscribe(google.protobuf.Empty) returns (stream BlockchainEvent);
}
```

{% hint style="success" %}
**TIP**

The CLI commands actually call the implementations of these service methods.

These methods are implemented in _**minimal/system\_service.go**_.
{% endhint %}

#### GRPC for Other Nodes

The Juvidoe Edge also implements several service methods that are used by other nodes on the network.\
The mentioned service is described in the **Protocol** section.

### 📜 Resources

* [**Protocol Buffers**](https://developers.google.com/protocol-buffers)
* [**libp2p**](https://libp2p.io/)
* [**gRPC**](https://grpc.io/)
