# Six unique aspects of TON Blockchain that will surprise Solidity developers

*by Tal Kol*

---

TON is a very modern blockchain that brings some radical new ideas to smart contract development. It was designed quite some time after Ethereum was launched and had the luxury to learn what works well in the EVM model and what can be improved.

If you have some prior smart contract experience, you're probably familiar with Ethereum's Solidiy language and its EVM. When learning TON development you should be aware of certain design differences that make things on TON behave quite differently from what you expect. The purpose of this post is to highlight some of these differences and give you some general ideas as to why they came to be.

#### *Moving from data to big data*

The main thing to understand about TON is that it was designed to bring blockchain to the hands of every human on earth. This means massive scale - billions of users sending billions of transactions per day.

Think of this as the shift from *data* to *big data*. When you need to store the menu of a restaurant, an SQL database is a great choice - it can run powerful and flexible queries because all the data is readily available. When you need to store the Facebook posts of every human on earth, an SQL database is probably not the route to take. These amounts of "big data" must be sharded aggressively - limiting the flexibility of the queries you can run. Different tradeoffs for different purposes.

---

Here are six unique aspects of TON blockchain that will probably surprise most Solidity developers:

## 1. Your smart contract needs to pay rent and charge its users

The blockchain as an immutable and eternal store of data, is a great concept on paper, but as we'll soon see, is quite impractical to scale. The fee model of Ethereum is inspired from that of a *bank*. You want to send some money, you pay the bank a transaction fee. Who is responsible for paying the fee? The user who initiated the transfer.

What about storing data on the blockchain - for example deploying the bytecode for a new smart contract? The Ethereum model dictates that the person sending the deploy transaction will pay the fee. But this fee is paid only once, yet if data on the chain is eternal, miners will have to keep paying infrastructure costs to retain this data for years to come. These fee economics don't add up, and if you try to scale them to billions of users, they will eventually collapse.

#### *Moving from a bank to an instant messenger*

The fee model on TON is quite different. Instead of emulating a bank account, TON is inspired from web apps like *instant messengers*. Who is paying the cost to send a message on Facebook Messenger? It's definitely not the person initiating the transfer. The app developer, Facebook Inc (or Meta Inc, I don't follow, I use Telegram myself), is actually covering the costs, and it's up to Facebook Inc to recover these costs somehow and fund itself.

Accordingly, in TON, the dapp itself needs to pay for its own resource costs. Every smart contract holds a TON token balance and uses this balance to pay rent. If the smart contract runs out of money, it will eventually be deleted (don't worry, everything is recoverable). Notice that paying for chain storage doesn't happen once, the rent payment is continuous. If you only hold data for a short period of time, you will pay significantly less. These fee economics are more in line with the costs of miners and thus easier to scale.

Very similar to Facebook Inc, the contract developer in TON has a lot of freedom to choose how to fund its operation. The developer can fund the contract with TON tokens out of pocket and subsidize its users; or it can charge gas from users for different actions and keep this gas in its balance for future rent payments.

## 2. Calls between smart contracts are asynchronous and not atomic

One of the great enablers of a powerful DeFi ecosystem on Ethereum is the seamless composability of contracts. In a single transaction, you can take some WBTC, supply it as collateral with a contract by [Compound](https://compound.finance/), and use it to borrow USDC, trade this USDC with a contract by [Uniswap](https://uniswap.org/) for more WBTC - and thus leverage your WBTC position. The whole process is even atomic - if any of these steps fails, even the last, the whole transaction rolls back like it never happened.

When your smart contract calls a method of a different smart contract, the call will be processed immediately in the same transaction. Ethereum in this regard is very similar to running your entire backend on a single server. Every part of your backend will be able to synchronously access every other part - an approach that is very easy to reason about. It has but one caveat, it can only grow as long as it fits in one place.

#### *Moving from a single server to a cluster of microservices*

If you imagine Ethereum as a monolith on a single server, TON is more similar to a cluster of microservices. Think that every smart contract may be running on a different machine. If two smart contracts want to call each other, just like two microservices communicating, they can send a message over the network. This message takes some time to travel, so communication is suddenly asynchronous! This means that when your smart contract calls a method of a different smart contract, the call will be processed after the transaction terminates, on some different future block.

This is much more difficult to reason about. What happens if conditions change from when the message was sent and until it was received? For example, the calling contract balance had one value, but by the time the second contract processes the call, the balance has changed. Maintaining consistency is more difficult and bugs can creep up. What about atomicity? What happens if you chain three calls and only the last one fails? If you need to roll back all the changes, you will have to do so manually.

## 3. Your smart contract cannot run getter methods on other contracts

This is actually a different face of the previous item on the list. On Ethereum, where calls between contracts are synchronous, reading data from a different smart contract is straightforward. Let's say that my contract has a balance of USDC. Since USDC is also a contract by itself, in order to know its own balance, my contract will have to call the `getBalance` method of the USDC contract.

Remember the monolith running on a single server? One of the great benefits of this approach is that every service can read  directly the state memory of every other service.

#### *Moving from a single server to a cluster of microservices*

When we have separate microservices running on different machines, reading state memory across services is suddently impossible. Smart contracts on TON can only communicate by sending asynchronous messages. If you need to query data from another contract and you require the answer immediately, you're out of luck.

It actually gets even stranger. If you see getter methods on a TON smart contract, such as `getBalance` - these methods are not accessible from other smart contracts. Getter methods can only be called by off-chain clients, similar to how your Ethereum wallet can use a full node like Infura to query any smart contract state.

## 4. Smart contract code is not immutable and can easily be modified

The original inspiration for dapps on Ethereum is legal documents drafted by *lawyers* - hence the name "smart contracts". The developer programs the terms of the legal contract as code, and code, as you know, is law. When two parties in the real world sign a contract, the contract is immutable. If any of the parties wants to change the terms of the contract, they would draft a new contract instead.

True to this approach, the code of smart contracts on Ethereum was designed to be immutable and never be modified. Over the years, the developer community has learned to overcome this limitation and generated some cumbersome patterns that rely on tricks like a proxy contract that points to a different contract in order to implement upgrades.

#### *Moving from a lawyers to software engineers*

Unlike lawyers, *software engineers* are taught that every piece of code has a bug in it. And even if mistakes would never happen, requirements still change over time and code often must be upgraded and modified.

Under TON, the pretence that contracts should be immutable is dropped completely. Smart contracts are free to modify their own code just like writing to any other state variable. If a contract writes to the code variable, it is mutable, and if it doesn't, it is immutable. This isn't a big change in practice, it just makes the cumbersome proxy pattern redundant.

## 5. You should not have unbounded data structures in your contract state

This is a tricky one that takes a while to understand, but will explain why some smart contracts on TON are architected the way they are.

Unbounded data structures are state variables in smart contracts that can grow indefinitely. Consider the ERC20 contract implementing the USDC token. This contract needs to maintain a map of balances per user address. The number of different USDC holders can grow indefinitely since USDC can be minted in large amounts and broken down to tiny pieces. In other words, the number of keys in the map can be as large as you want.

What happens if an attacker tries to spam the contract by adding more and more entries? Would they be able to cause some DoS attack and prevent other honest users from using this contract? Ethereum solves this problem quite elegantly for smart contract developers. The Ethereum fee model dictates that the user who writes new state data pays the fee for this data. This means our attacker will have to pay a high cost for their spam. In addition, the gas cost for writes to a map on Ethereum is constant and does not depend on how much data this map contains, which means other users will not suffer from the spam. The bottom line is that spamming maps on Ethereum is not economical and the protection is provided by the system.

#### *Moving from unbounded maps to unbounded contracts*

Unfortunately for TON smart contract developers, the system does not protect against spamming unbounded data structures in contract state. The TON gas fee model dictates that writes are not constant in cost, the cost is normally proportional to how much data exists in the data structure. This behavior stems from TON's reliance on the "Bag of Cells" architecture - contract state is divided to 1023 bit chunks called "cells" that the developer is required to maintain. Maps are implemented as a tree of cells, and writing to a leaf in the tree requires writing new hashes along its entire height. If an attacker were to spam keys in the map, some user balances would be pushed so low in the tree that updating them would be pass the gas limit.

The best practice in TON is therefore to avoid unbounded data structures in state. This will protect the contract from crafty DoS vulerabilities. This topic probably deserves its own independent blog post, but the solution in short is to rely on contract sharding. If we have a potentially infinite number of user balances in our USDC contract, we should break the single contract to multiple child contracts - each child holding the balance for a single user.

This should explain why [NFT](https://github.com/ton-blockchain/token-contract/tree/main/nft) collection contracts on TON place every item in its own separate contract (the number of items can be unbounded); and why [fungible token](https://github.com/ton-blockchain/token-contract/tree/main/ft) contracts on TON place every user's balance in its own separate contract.

We normally provide a bit of the reasoning as to why TON is designed this way. The Ethereum gas fee model where map writes are fixed and independent of map size is over simplified. In reality, as maps grow in size, miners require more effort to change their contents. This extra effort is negligible as long as the maps are small, but when maps can grow to billions of entries, this is no longer the case.

## 6. Wallets are contracts and one public key can have mutliple wallets deployed

On Ethereum, a user's wallet is synonymous to their address, and an address is derived directly from a public key (and its corresponding private key). This is a 1:1 relationship, there's one address per public key and one public key per address. As long as the user knows their private key, they could never lose their wallet.

In addition, a user on Ethereum doesn't have to do anything special to have a wallet. The Ethereum address is the wallet. The address can hold the native currency ETH, the address can hold ERC20 tokens and NFTs and the address can send and sign transactions to other smart contracts directly.

#### *Moving from address to contract*

On TON, wallets are not implied, they are independent smart contracts that must be deployed like any other smart contract. When a new user wants to start using TON blockchain, their first step would be to deploy a wallet on-chain. The address of this wallet is derived from the wallet contract code and various init parameters like the user's public key.

This means that a user can have more than one wallet deployed, each with its own address. The wallets can differ on their code (different official code versions are published by the foundation from time to time) or on their init parameters (one of these parameters is normally a sequence number). This also means that a user that knows their private key must still make a conscious effort to remember their wallet address (or the init parameters used in its construction).

Sending a transaction to some dapp on TON involves signing a message using the user's private key. Unlike Ethereum, this transaction is not sent to the dapp smart contract, but to the user's wallet contract, which will in turn forward the message to the dapp smart contract.

This design approach opens up a new dimension of flexibility on TON. New wallet contracts can be invented by the community over time, for example consider a wallet without a nonce (transaction sequence number) that allows multiple transactions to be sent in parallel from different clients without prior synchronization. In addition, special wallets like multi-sig wallets (that also on Ethereum require a special smart contract to be deployed), behave just like their regular counterparts.

---

Tal is a founder of Orbs Network (https://orbs.com). He's a passionate blockchain developer, open source advocate and a contributor to the TON ecosystem. He is also one of the main developers for TONcoin Fund (https://www.toncoin.fund). For Tal's work on TON, follow on GitHub (https://github.com/ton-defi-org). For Tal's personal work, follow on GitHub (https://github.com/talkol) and Twitter (https://twitter.com/koltal).
