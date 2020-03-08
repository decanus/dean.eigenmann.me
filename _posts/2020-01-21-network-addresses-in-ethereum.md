---
layout:        post
title:         Network Addresses in Ethereum
date:          2020-01-21 14:30:00
summary:       An overview of multiaddrs, ENR & enodes.
categories:    blog
tags:          ethereum
---

After a discussion with Felix Lange, I decided it would make sense to write this blog post about the various types of network addresses you may encounter in the ethereum ecosystem and discuss the differences between them. I myself had some misconceptions about them and therefore thought clarification could be useful.

## [Multiaddr](https://multiformats.io/multiaddr)

Let's get started with the oldest, or at least the oldest documented as per commit timestamps, multiaddr. Multiaddrs are part of Protocol Labs' multiformats project. Multiformats is essentially various standards for self-describing values. You may have heard about them as they are used in libp2p and IPFS, other Protocol Labs projects.

Multiaddrs can be represented in two ways: one is a binary representation used in storage or when transmitting, and the second is a human-readable format for use when printing to the user.

```
/ip4/127.0.0.1/udp/1234
```

The above shows a multiaddr in its human-readable representation. The multiaddr is a recursive format which represents addresses in key-value pairs. The binary representation is the same -- there is a byte array representing the key and one representing the value. The key can be mapped from human-readable to the code using the [protocol table](https://github.com/multiformats/multiaddr/blob/master/protocols.csv).

## [enode](https://github.com/ethereum/wiki/wiki/enode-url-format)

Next up is enode which is not really a network address format but a url format instead. We will cover it anyway as it is the predecessor to ENRs. An enode url looks as follows:

```
enode://6f8a80d6ad92a0@10.3.58.6:30303?discport=30301
```

The `enode` scheme is used to denote the URL, it is followed by a hexadecimally encoded node ID. Next the host is represented after an `@` symbol. It is important to note that enode does not allow for the host to be a DNS domain name, it must be an IP address. After the host the TCP port is listed, in our case `30303`. If the UDP and TCP port differ, the UDP port can be specified by adding the `discport` parameter at the end.


## [ENR](https://eips.ethereum.org/EIPS/eip-778)

Finally we have the ENR (Ethereum Node Record). These are interesting because they use features from both of the previous types (multiaddrs and enode URLs) making them really versatile. The main motivation for ENRs was to allow for more information to be relayed, hence the introduction of the node record. The node records are self certified, and nodes prove authorship using signatures. The records are represented as an [RLP list](https://github.com/ethereum/wiki/wiki/rlp) -- I won't go into detail here, but RLP is a serialization format used by Ethereum.

The elements of this record are a signature, sequence number, and a required field indicating the identity scheme used to create and validate signatures. Finally, the remainder of the record contains arbitrary key / value pairs that can contain things like connection information. The EIP defines keys with predefined meanings such as `ip`, which is the IPv4 address of a node represented as 4 bytes.

The signature is used to verify the record by ensuring that the passed public key was the one used to create the signature.

The sequence number can be used to resolve conflicts, if there are 2 different records authored by the same identity, the one with larger sequence number must be used.

It is important to note that the RLP encoded version of this node record is not allowed to exceed 300 bytes.

This format is future proof as new keys can be added even when not all clients know how to inteperet these, as well as the fact that new identity schemes can be added to validate signatures.

## ETH 2.0

Now, let's look at ETH 2.0. Up until ETH 2.0 there was never any usage of multiaddrs in Ethereum, but now it becomes very important. Why is that the case you may ask? Well, ETH 2.0 uses libp2p which in turn uses multiaddrs to identify nodes.

So how can we handle this? Well there are 2 ways that were highlighted in the [p2p specs](https://github.com/ethereum/eth2.0-specs/blob/065b4ef856aeb7f84f1bed5c4a2cd4d6ac1edc87/specs/phase0/p2p-interface.md#what-is-the-difference-between-an-enr-and-a-multiaddr-and-why-are-we-using-enrs) of ETH2.0:
 1. The multiaddr can be derived from the ENR.
 2. The multiaddr can be included in the ENR due to the ability to add arbitrary keys.

Hopefully this gives a brief and helpful overview of what these different network addresses are, how they work and what they can be used for.
