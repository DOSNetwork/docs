## Introduction
DOS NETWORK is a layer 2 chain-agnostic **D**ecentralized **O**racle **S**ervice network, providing real-time data feed oracle and computation oracle solution to mainstream blockchains.

DOS NETWORK brings real world data, event and computation power to blockchain smart contract, which will boost blockchain usability with new business models, use cases and more network effect.

Our vision is to offer oracle service to all mainstream public chains. But currently, we are focusing on supporting Ethereum first, since Ethereum has the largest technical community and most abundant applications.

We just released alpha version private testnet of data feed oracle on Rinkeby, which is a great development and testing network for Ethereum. So you can try to create your own contract based on our sdk right now in a testing network environment.

Following the Ethereum part of developers guild is a great way to understand how to use our sdk to fetch external data to smart contract.

## Features
- **Byzantine fault tolerant decentralized protocol:** A trustless solution without single point of failure or downtime, with node runners come and go on the fly.

- **Near real-time and verifiable result:** Driven by the secure randomness engine built upon Verifiable Random Function (VRF) and advanced cryptographic primitives, the data request or computation is performed and consented off-chain but verifiable on-chain in a system contract. The result will be returned in real-time.

- **Scalable at a low cost:** Thanks to the group sharding and off-chain consensus, the total data query throughput and computation capacity increases as more nodes joins DOS network.

- **Compatible to heterogeneous blockchains:** With core protocols operating in off-chain clients, it’s easy to support multiple public chains by adding off-chain adaptors and port on-chain governance contracts at a low overhead.

- **Crypto economy:** With designed token economic models and cryptographic primitives, it’s highly incentive for node runners to join the network early to earn the “mining” rewards and rapidly grow the network capacity. The network is resistant to sybil attack, with malicious nodes to be detected and forfeited of their stake and honest nodes always to be rewarded.


## Use Cases
- **Decentralized Derivatives:** Derivatives are financial contracts between two or more parties whose values are based on the underlying assets. Derivatives allow people to put different viewpoints (long or short) on the underlying assets and in essence promote the financial stability. DOS Network could take a significant role in decentralized derivatives by providing price feeds, settlement values and contract expirations to determine gain or loss for participating parties.

- **Stablecoins:** tablecoins are cryptocurrencies with stable fiat value, reducing volatility and making them more appealing as a store of value and medium of exchange in many ways, so they are honored as the holy grail in digital currency. When referring to stablecoins we’re refering to decentralized and algorithmically controlled cryptocurrency, such as collateral backed stablecoins like bitUSD and Dai, and Seigniorage Shares based stablecoins like Basecoin and kUSD. All stablecoins need the help of oracle system like DOS Network to get external data about exchange rate between stablecoins and the asset they’re pegged to. 

- **Decentralized Lending Platform:** Decentralized peer-to-peer lending platforms like SALT Lending and ETHLend allow anonymous users to pledge crypto assets on blockchain in exchange for fiat or crypto loans. DOS Network could be applied to introduce market rates during loan creation and to monitor the ratio of crypto collaterals to the loaned amounts, triggering liquidation events if loan terms are met. Furthermore, ETHLend also uses oracles to import borrowers’ social media data and other identity info to differentiate interest rates between different borrowers.

- **Decentralized Insurance:** Etherisc is building a platform for decentralized insurance applications like flight delay insurance and crop insurance by bringing in efficiency and transparency with lower operational costs. Users purchase insurance policy and pay the premium in ether and they’ll get corresponding payout in ether back according to the policy in case of agreed-on conditions are met. By introducing external data and events into smart contracts, DOS Network helps those decentralized insurance products in policy underwriting and payout decisions in case of claims as well as schedule future checks by the time of policy expiration to achieve automatic payout.

- **Decentralized Casino:** Decentralized casinos like Edgeless, DAO.Casino and FunFair benefit a lot from blockchains in terms of transparency, near-instant secure fund transfer, and provably fair with 0% house edge comparing to traditional online casinos with 1%~15% house edge. Unpredictable and verifiable random number generation is the core of any casino game, but random number generation in a pure deterministic environment is difficult or even impossible in principle. DOS Network is publishing a provably secure, unstoppable, unbiased and verifiable random entropy source on-chain available for those Dapps to use.

- **Decentralized Prediction Markets:** Decentralized prediction markets like Augur and Gnosis utilize wisdom of the crowds to predict real world outcomes such as presidential election and sports betting result. DOS Network could be used for fast and secure resolution in case of the voted results being challenged by users.

- **Decentralized Computation Markets and Execution Scalability:** Bypassing the block gas limit and expensive on-chain computation cost, DOS Network connects redundant third party computing power with business computation intensive tasks such as machine learning model training, 3D rendering, scientific computing like DNA sequencing. In our long-term roadmap, zkSNARK based computation oracle would also offer privacy to computing tasks as private input is supported. Also for the current blockchain scalability debate, it would bring unlimited execution scalability to supported chains.
