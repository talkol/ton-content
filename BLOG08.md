# DAOs, multi-sigs and practical decentralized governance for dapps

*by Tal Kol*

---

One of the core principles of [web3](https://defi.org/ton/) is the concept of decentralized governance. A web3 protocol or a dapp should be governed and owned by its community. But what does that mean exactly? What is a protocol DAO? How should a dapp manage its treasury? What is a practical way to conduct decentralized governance?

## A new web3 project is born

The purpose of this post is to help guide TON dapp developers on the best practice way to manage the new project they're building. You may be building a DEX like [Uniswap](https://uniswap.org/), a lending platform like [Aave](https://aave.com/), a stablecoin like [Dai](https://makerdao.com/en/) or a web3 game like [Axie Infinity](https://axieinfinity.com). These dapps are the bread and butter of blockchain and are the primary use-case for which the TON Blockchain was created.

A dapp is a service running on top of the blockchain, whose backend is implemented using a set of smart contracts. Unlike traditional web services like [Uber](https://uber.com/), dapps are designed to be decentralized. This means that they should not be controlled by a centralized entity. Instead, they should be controlled by their community of users.

Decentralized entities are very difficult to steer. When you don't have a single person in charge, how can you cause a decentralized group to work together? The answer is incentives. Dapps normally involve an economy that incentivizes its members and revolves around some token. Ownership of this token symbolizes ownership of the dapp.

TON itself is no different. TON was designed as a decentralized protocol that revolves around [TON Coin](https://www.coingecko.com/en/coins/toncoin). The TON Foundation is the representative body of this organization, but it should ideally hold executive priviliges only. Core decisions, like changing the protocol tokenomics for example, should be taken by the TON community, its validators and its token holders.

So you have decided to found a new web3 project and build it over TON. You would normally raise some funding for R&D and [create a jetton](https://minter.ton.org) or NFT to tokenize ownership of the new project. These steps are relatively straightforward, but where do we go from here?

## Open DAOs vs. closed DAOs

A DAO (Decentralized Autonomous Organization) is defined as a collectively-owned, blockchain-governed organization working towards a shared mission. Unlike regular corporations, DAOs are fully democratized and are normally flat without internal hierarchies. Members of the DAO would normally vote on decisions and all activity is transparent and fully public.

The above definition is quite broad and I think it's important to distinguish between two very different types of DAOs - open DAOs and closed DAOs.

If you're building a new web3 project on TON, the project that you're building is most likely an *open DAO*. The characteristics of an open DAO are:

* Anyone can participate and become a member
* Membership involves acquisition of a token (jetton or NFT)
* Participation rights are fully transferable and can be sold to others

Participants in the new web3 project will hold the project token and become members of the project DAO. This DAO is the governing body that should make material decisions regarding the project's future. This is, for example, how [Uniswap](https://uniswap.org/) operates. The Uniswap protocol is an open DAO that is governed using the [UNI](https://www.coingecko.com/en/coins/uniswap) token and its holders. In a similar way, TON itself is an open DAO that is governed by the holders of [TON Coin](https://www.coingecko.com/en/coins/toncoin).

So what is a *closed DAO*? Suppose that a group of 5 friends decide to create a new organization together for some purpose. They may want to invest money together as a group, or provide some service as a group to generate revenue, or maybe even start a charity for some noble cause. While it is considered a DAO by definition, its closed nature has the following characteristics:

* Participation is not open and new candidates for membership may be denied
* Membership requires approval by a majority of existing members
* Participation rights are normally not transferable

## DAO vs. multi-sig

Although *closed DAOs* are technically DAOs, in the context of building dapps on TON, I prefer to use the term "DAO" only for *open DAOs*.

I prefer to refer to *closed DAOs* as "multi-sigs".

The term multi-sig comes from wallet management. A normal crypto wallet has a single signer with a single private key. If you know the private key, you effectively own the wallet and can use it to sign whatever transaction you desire. A multi-sig is crypto wallet that has a group of multiple different owners, each having their own private key. In order to sign a transaction, you need to collect multiple signatures. For example, a multi-sig of 5 owners will normally issue a transaciton when 3 of 5 signatures are collected.

If you look closely, you will see that a multi-sig has the same characteristics as a closed DAO:

* Participation is not open and new candidates for membership may be denied - match! strangers are naturally unwelcome when managing a mutli-sig wallet
* Membership requires approval by a majority of existing members - match! adding a new signer to a multi-sig requires approval of existing signers
* Participation rights are normally not transferable - match! once you know one of the private keys, you can't really "un-know" it so you cannot surrender your ownership to somebody else

The separation of terms is important because many ecosystem tools are only suitable for closed DAOs, yet by labeling themselves as a general tool for DAOs, they may confuse you that they're appropriate for the new token-based dapp you're building. Take [this tool](https://www.xdao.app) for example. If you were building Uniswap, you could not use XDAO to govern your open DAO. In my terminology, XDAO is a tool for multi-sigs.

