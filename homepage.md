## Introduction
> DOS Network is a layer 2 chain-agnostic **D**ecentralized **O**racle **S**ervice network, providing real-time data feed oracle and computation oracle solution to mainstream blockchains.

<center>![overview](_media/overview.png)</center>

Compared with other centralized or decentralized oracle solutions, DOS Network has many unique features making it the best choice for smart contract and Dapp developers:
- Byzantine fault tolerant and decentralized protocol - there's no single point of failure or downtime to break your Dapps. It's also an open membership network with crypto economy that tolerates malicious nodes and sybil attacks.
- Not reputation based, but driven by secure, unmanipulatable and verifiable randomness - Reputation based system is prone to Matthew Effect, collusion, and targeted attacks that leads to many security issues.
- Near real-time and verifiable response at a low cost - Built upon Verifiable Random Function and advanced cryptographic signatures, the response along with the proof of the response are delivered back on-chain together within only one transaction.
- Compatible to multiple heterogeneous blockchains - Use it once, apply it everywhere. After at least two blockchains are supported, cross-chain contract call will also available.
- Verifiable computation oracle based on zkSNARK.


## Overview
The vision is to offer standard and complete set of oracle services to all mainstream blockchains. Right now it's being built on Ethereum first, however, we're actively exploring and integrating with other blockchains like Thunder, Zilliqa, Neo, EOS, etc.

In the high level DOS Network consist of two parts: on-chain smart contract part and off-chain P2P network (client) part:
* **On-chain smart contract**: A set of DOS system contracts are deployed on supported blockchains, including functionalities such as request handling and response/computation result verification, node registration and staking, group public key registration, stats monitoring, payment processing, etc. On-chain system contracts expose a universal interface to all user contracts across supported chains.
* **Off-chain P2P network**: A P2P network consists of client software implementing the core protocol running by node operators. Client includes several important modules: event monitoring and chain adaptor module, VRF module, off-chain group consensus module, response parsing module, verifiable computation module, etc.


## Recent update
We've just released **alpha version** of the data feed oracle on Ethereum rinkeby testnet, which currently is a private network consisting of a group of 21 nodes with alpha functionalities including VRF, threshold group signature, on-chain group signature verification, json and xml/html response parsing, data query and secure random number generation and so on.

Please check [developers guide](contents/blockchains/ethereum) and examples to start using our on-chain SDK to fetch external data and request secure random numbers within your smart contracts, and stay tuned for more updates and future releases.

Also join our [telegram channel](https://t.me/dosnetwork_en) to communicate with the team and community members directly!
