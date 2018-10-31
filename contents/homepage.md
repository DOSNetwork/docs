## Introduction
DOS NETWORK is a layer 2 chain-agnostic **D**ecentralized **O**racle **S**ervice network, providing real-time data feed oracle and computation oracle solution to mainstream blockchains.

DOS NETWORK brings real world data, event and computation power to blockchain smart contract, which will boost blockchain usability with new business models, use cases and more network effect.

DOS NETWORK is chain-agnostic, meaning that it could serve all existing smart contract platforms; it is decentralized, meaning that it has no single point of failure, no central trust in a single company or special hardware, the trust lives in math and code; it is horizontally scalable, meaning that with more nodes running DOS client software the whole network offers more capability and computation power to supported blockchains; it is designed with cryptoeconomic models, meaning that the protocol is resistant to sybil attacks and the network effect is expanded with provable credibility.

![](../_media/architecture.png)

DOS NETWORK is consist of two part: on-chain contracts part and off-chain p2p network part:
* **on-chain contracts part:** A set of DOS system contracts deployed on supported blockchains, mainly including functionalities such as request handling and response/computation result verification, node registration and staking, stats monitoring, payment processing, etc. On-chain system contracts also provide a universal interface to all user contracts across supported chains.


* **off-chain p2p network part:** A client software implementing the core protocol run by third party users aiming for economic rewards, constituting a distributed network. Client software includes several important modules: event monitoring and chain adaptor module, distributed randomness engine module, off-chain group consensus module, and request processing/computation task processing module depending on the type of oracle service the user node provides.

Our vision is to offer oracle service to all mainstream public chains. But currently, we are focusing on supporting Ethereum first, since Ethereum has the largest technical community and most abundant applications.

We just released **alpha version private testnet of data feed oracle on Rinkeby**, which is a great development and testing network for Ethereum. So you can try to create your own contract based on our sdk right now in a testing network environment.

Following **the Ethereum part of developers guild** is a great way to understand how to use our sdk to fetch external data from smart contract.
