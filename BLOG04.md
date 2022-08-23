# How to shard your TON smart contract and why - studying the anatomy of TON's Jettons

*by Tal Kol*

---

TON's ultimate mission is to bring blockchain to mass adoption. How many people in the world have actually used blockchain? Ethereum's statistics mention [several millions](https://www.fool.com/the-ascent/cryptocurrency/articles/more-people-own-ethereum-than-ever-before-heres-why/), so taking a figure of 50 million global users for blockchain until today is probably being generous. How do we get this number to 1 billion?

The current version of Ethereum processes [about 1 million](https://ycharts.com/indicators/ethereum_transactions_per_day) transactions per day at peak capacity. Back in 2016, Telegram, a messaging app with mass adoption nearing the numbers we're aiming for, was delivering [15 billion](https://telegram.org/blog/15-billion) messages per day. Increasing scalability of systems x10000 is not normally achieved by mere protocol improvements, this feat requires a radical change in approach.

## The concept of sharding

*Sharding* is a mature concept that originates in [database design](https://en.wikipedia.org/wiki/Shard_(database_architecture)) which involves splitting and distributing one logical data set across multiple databases that share nothing and can be deployed across multiple servers. Simply put, sharding allows *horizontal scalability* - splitting data to different independent parts that can be processed in parallel. This is a key concept in the world's transition from data to [big data](https://en.wikipedia.org/wiki/Big_data). When data sets are becoming too big to be handled by traditional means, there's no other way to scale other than breaking them down to smaller parts.

TON is not the first to apply sharding to blockchain. Ethereum 2.0 supports a fixed number of [64 shards](https://consensys.net/blog/blockchain-explained/the-ethereum-2-0-beacon-chain-is-here-now-what/). TON's approach is radical not because the number of shards is larger, but because of two unique conceptual changes:

* The number of shards is not fixed - TON supports adding more and more shards as needed with an upper bound of 2^60 (per workchain). This number is practically limitless, it would be enough for every person in the world to be given 100 million shards and still have spare.

* The number of shards is elastic - TON supports automatic splitting of shardchains in two when the load is high and merging them back together when the load is low. This is the only possible way to deal with changing scaling requirements that are impossible to predict in advance.

You can read more about these novel ideas in the [TON whitepaper](https://ton-blockchain.github.io/docs/ton.pdf).

Trying to change the world in a fundamental way rarely comes without a price. To make use of this radical approach, the developers of smart contracts on TON must design their contracts in a different way. If you have prior smart contract experience, for example in Solidity, this architecture will feel alien. I recommend to read first a previous post of mine, [Six unique aspects of TON Blockchain that will surprise Solidity developers](https://society.ton.org/six-unique-aspects-of-ton-blockchain-that-will-surprise-solidity-developers), to ease the transition.

## Sharding TON smart contracts

The basic atomic unit in TON blockchain is a smart contract instance. A smart contract instance has an address, code and data cells (persistent state). We refer to this unit as atomic because a smart contract always has atomic synchronous access to all of its persistent state.

Communication between smart contract instances on TON is not atomic nor synchronous. Think of smart contracts on TON like [microservices](https://en.wikipedia.org/wiki/Microservices). Every microservice has atomic synchronous access to its local data only. Communication between two microservices involves sending asynchronous messages over the network.

As every system architect knows, larger systems require the world to shift from [monolithic architectures](https://medium.com/koderlabs/introduction-to-monolithic-architecture-and-microservices-architecture-b211a5955c63) to microservices. This distributed approach takes some effort to adopt, but unlocks some desirable benefits. The modern systems paradigm relies on an orchestrator like [Kubernetes](https://kubernetes.io/) to take a group of containerized microservices and launch new instances automatically on demand (autoscale) as well as partition them across machines efficiently.

I like this analogy because this is exactly what TON does. As load on a specific shardchain increases, it will be split in two. Smart contracts instances are atomic, they're never broken in half. This means that some smart contract instances that once lived on the same shardchain may one day find themselves living on different ones!

**In other words, TON's TVM is applying the concept of distributed microservices to Ethereum EVM's monolith.**

## Designing smart contracts for sharding

A common question by young systems architects is "how big should my microservices be?" - or in other words, "when is a microservice too monolithic and should be broken down to two different ones?"

There is no one answer to this question, it's an art. The idea is to assist Kubernetes in doing its thing. The smaller microservicea are, the easier it is for Kubernetes to optimize the system by creating new instances and moving them around on demand. But, the smaller they are, the harder it is for the developer to implement complex flows as more actions become asynchronous.

I've discovered that the same reasoning works for TON contract sharding. The idea is to let TON auto-sharding do its thing. Split state data to multiple smart contract instances so that when load increases, they could be broken down to smaller parts and moved to different shardchains efficiently. But, if you shard too aggressively, you will have to deal with too much complexity due to more asynchronicity.

## Practical example - TON's Jetton contract

This post has been quite theoretical so far. I want to take a sharp turn to practicalities. Let's analyze a real world example to understand this architecture. The example we'll use is TON's [Jetton](https://github.com/ton-blockchain/TIPs/issues/74) smart contract. A Jetton is a smart contract implementing a fungible token (very similar to TON coin itself). This is TON's version of Ethereum's popular [ERC20 token](https://ethereum.org/en/developers/docs/standards/tokens/erc-20/) standard.

Implementing a token is pretty simple. We will need one basic action - *transfer* - which allows an owner to transfer some token amount to a different owner. We will also need a *mint* action - the ability to add new tokens to circulation, and its opposite *burn* - which removes tokens from circulation. What about persistent state? We would need to store the balances of all users. On Ethereum, this would normally require a map where the key is a user's address and the value is the balance amount.

As the architects of this smart contract on TON, we will need now to decide whether this smart contract needs to be broken down to multiple smaller instances to support auto-sharding effectively. What would happen if our Jetton has 1 billion users? Will our architecture hold up in this case?

## Distributing Jetton to multiple smart contracts

Let's try to apply the reasoning outlined above to find the "correct" amount of sharding for Jetton. I realize that this is a little too theoretical. Luckily, there's a very practical test that I've found to work quite well:

**If you ever find yourself designing a smart contract with an unbounded data structure, there's a good chance you're supposed to break this contract to multiple instances.**

Unbounded data structures are arrays or maps that can grow indefinitely. Under Ethereum, our smart contract would require a map that holds all user balances. This map can grow indefinitely because the number of holders of our token is unbounded. New accounts can be created practically indefinitely and because the numerical precision is so high, it is possible to transfer miniscule amounts of the token to all of these accounts.

Let's apply our practical rule. If we were to hold all balances in a single smart contract on TON, we would have an unbounded data structure. This means we have an excellent candidate for sharding!

So how do we shard? This is quite straightforward. If we don't want all balances to be in a single smart contract instance, what if we split every balance to be in its own dedicated smart contract instance?

## The Jetton architecture

Let's assume that our Jetton instance is for a token called [Shiba-Inu](https://coinmarketcap.com/currencies/shiba-inu/) or `SHIB` for short. We have two users who are holding some SHIB - Alison and Becky. We already said that the balalnce of each user is held in its own contract instance, which means we have 2 instances (the "children"). It turns out that we also want another instance to hold global shared information about SHIB (the "parent").

This brings us to the following architecture:

<img src="https://i.imgur.com/S2lFhsY.png" width=513 />

I promised to be practical. Let's start reading the actual Jetton code! The TON core team has an official implementation of the Jetton standard which you can find [here](https://github.com/ton-blockchain/token-contract/tree/main/ft). Open it up so you can get familiar with the code.

You can see in the code two main FunC smart contracts:

* [`jetton-minter.fc`](https://github.com/ton-blockchain/token-contract/blob/main/ft/jetton-minter.fc) - This is the parent, which holds global shared information about the token, such as its name and symbol. There's just a single instance of the parent. I'm not entirely sure why the core team chose the name `jetton-minter`, I would have preferred the name `jetton-parent`. It is true that this contract is in charge of minting, but even if minting is disabled, you still need it, which is somewhat confusing.

* [`jetton-wallet.fc`](https://github.com/ton-blockchain/token-contract/blob/main/ft/jetton-wallet.fc) - This is the child, which holds the token balance for a single user. There are multiple instances of this contract, one for each user address. The core team chose the name `jetton-wallet` for this contract, I would have preferred the name `jetton-child`.

If our token is held by 1,000,000 different users, there will be exactly 1,000,001 contract instances deployed. This is where the magic of auto-sharding happens. By default, all contract instances will be found on a single shardchain. But, if these users start issuing a large number of transactions and this single shardchain is under high load, TON will automatically split it into smaller shardchains. Theoretically, the system can keep splitting and splitting until every contract instance is found on a dedicated shard. That's the secret that enables TON to scale to billions of users.

## The different user stories for Jetton

Now, that we understand the basic architecture, let's see what happens on different user stories. For example, let's explore what happens when one user transfers tokens to another.

Under TON, the entities participating are always smart contracts instances. Which contracts will play a part?

<img src="https://i.imgur.com/HXnp3Bl.png" width=1035 />

You've already met the first three. The source code for these is found in the Jetton [repo](https://github.com/ton-blockchain/token-contract/blob/main/ft). What about the three contracts on the right? Our user stories will involve three different users. Alison and Becky are holders of SHIB. The user Admin is the creator that deployed SHIB. The Admin has a special role because that's the only user that can mint new SHIB into circulation (that's how new SHIB tokens are born). This is a trusted role and would normally be revoked (changed to the zero address) once the token starts trade.

Users on TON are also represented by smart contracts. These are wallet smart contracts that are normally deployed for users by wallet apps such as [TonKeeper](https://tonkeeper.com/). If you're not familiar with how wallet contracts work on TON, please read my previous post [How TON wallets work and how to access them from JavaScript](https://society.ton.org/how-ton-wallets-work-and-how-to-access-them-from-javascript). Alison, Becky and Admin hold their TON coin balance in these wallets. These wallets are not specifically related to the Jetton code. Here is an [example implementation](https://github.com/ton-blockchain/ton/blob/master/crypto/smartcont/wallet3-code.fc) for such a wallet contract from the core TON repo.

### User story 1: Alison has SHIB and sends some to Becky

Our user stories will always start with one of our users (Alison in this case) that decides to perform some action with our Jetton. In this case, Alison has decided to send some tokens to Becky. Alison will open her wallet app of choice (TonKeeper for example) and approve the action. Once this happens, the wallet app will send a signed transaction to Alison's wallet contract.

The transaction contains a [message](https://ton.org/docs/#/smart-contracts/messages) for some destination contract. Messages are how smart contracts communicate on TON. Messages are encoded as a [bag of cells](https://ton.org/docs/#/overviews/Cells), which is in essence a packed binary format. One of the key fields of the message is a 32 bit integer called `op` which describes the operation type of this message.

1. In our example, since Alison wants to send some tokens, she sends a message with op type `transfer` to the smart contract instance holding her SHIB balance. This message is encoded on the transaction she sends her wallet contract. Once her wallet contract verifies the signature on the transaction [[code]](https://github.com/ton-blockchain/ton/blob/9640a2794a1cc88a9534991c0709f64f95c1c4e0/crypto/smartcont/wallet3-code.fc#L17), it will forward Alison's message to the destination she requested [[code]](https://github.com/ton-blockchain/ton/blob/9640a2794a1cc88a9534991c0709f64f95c1c4e0/crypto/smartcont/wallet3-code.fc#L22).

2. Once the `transfer` message reaches its destination [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L222), the contract holding Alison's SHIB balance, this contract will process the message and alter its persistent state (reduce Alison's SHIB balance by the sent amount [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L50)). If the contract needs to contact other contracts, it may send additional messages. In our case, the contract will send a message with op type `internal transfer` to the contract holding Becky's SHIB balance [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L90).

3. Once the `internal transfer` message reaches its destination [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L227), this contract will now process the message and alter its persistent state (increase Becky's SHIB balance by the sent amount [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L108)). This contract will normally send one last message with op type `excesses` back to refund any remaining gas back to Alison's wallet contract and let it know the transfer is complete [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L151).

This is the flow of messages:

<img src="https://i.imgur.com/QhNLCof.png" width=890 />

Messages on TON are asynchronous. We don't know exactly when they will be handled. There's a chance all messages will be handled on a single block and there's a chance each message will be handled on a different block. This means the the transfer may take some time to process and even if the first transaction has confirmed successfully, the transfer may still fail.

### User story 2: Alison has SHIB and sends some to Becky and notifies Becky about it

What if the SHIB recipient Becky is more than just a person, it's an online store contract that should do something when being paid. For example, change a DNS record to point to a new owner. It would be nice if we could trigger this smart contract with a dedicated message.

Fortunately, the `transfer` message supports this behavior. It allows the original sender to specify some notification payload that will be forwarded to the recipient SHIB wallet owner.

4. The flow in this case is nearly identical, except in the last step, before sending the message with the op type `excesses`, the contract holding Becky's SHIB balance will send a message of op type `transfer notification` to the owner of Becky's SHIB wallet - Becky's wallet contract [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L140). This story would make more sense if you rename "Becky" to an online store like "DNS-Superstore". In that case, the contract "DNS-Superstore" will receive this notification because it is the owner of the SHIB wallet for "DNS-Superstore". This contract would implement the behavior for changing the DNS record ownership in the handler of this message.

This is the flow of messages:

<img src="https://i.imgur.com/uhRXPJy.png" width=908 />

How would you know which other features the message `transfer` supports? Messages are normally encoded in a language called [TL-B](https://ton.org/docs/#/overviews/TL-B). The creator of the contract should normally publish the TL-B specification for all messages their contract handles. Here is the relevant TL-B spec [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L33):

```
transfer query_id:uint64 amount:(VarUInteger 16) destination:MsgAddress
           response_destination:MsgAddress custom_payload:(Maybe ^Cell)
           forward_ton_amount:(VarUInteger 16) forward_payload:(Either Cell ^Cell)
           = InternalMsgBody;
```

* `amount` is the number of SHIB tokens to transfer
* `destination` is Becky's wallet contract address
* `response_destination` is the address for the recipient of `excesses` (normally Alison's wallet contract)
* `forward_payload` is the notification payload for the "DNS-Superstore" use-case

### User story 3: Alison has some SHIB and burns it

In this user story, Alison decides to burn some of the SHIB she has. Burning SHIB will remove it from circulation and reduce an interesting metric of a token called [total supply](https://academy.binance.com/en/glossary/total-supply). Users care about the total supply of the token because that helps to calculate the token's [market cap](https://academy.binance.com/en/glossary/market-capitalization).

Where is the total supply stored? As you've probably guessed it, since this persistent state data is globally shared, it would make sense to store it under our parent `jetton-minter`.

1. To initiate the burn, Alison sends a message with op type `burn` to the smart contract instance holding her SHIB balance. This message is encoded on the transaction she sends her wallet contract like before, which forwards it to its destination after verifying the signature.

2. Once the `burn` message reaches its destination [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L232), the contract holding Alison's SHIB balance, this contract will process the message and alter its persistent state (reduce Alison's SHIB balance by the burned amount [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L166)). The contract will then send a message with op type `burn notification` to the parent minter contract [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L186).

3. Once the `burn notification` message reaches its destination [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-minter.fc#L69), this contract will now process the message and alter its persistent state (reduce the total supply by the burned amount [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-minter.fc#L75)). This contract will normally send one last message with op type `excesses` back to refund any remaining gas back to Alison's wallet contract and let it know the burn is complete [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-minter.fc#L85).

This is the flow of messages:

<img src="https://i.imgur.com/EFychzC.png" width=921 />

The parent minter contract allows users to query the total supply of the token using a Getter method [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-minter.fc#L106).

### User story 4: Admin mints SHIB for Becky

When all contracts are initially deployed, the total SHIB supply is zero and nobody has any tokens. How are tokens created? The action of creating new tokens is called minting and it can only be performed by a special admin role - our Admin user. The Admin user may also transfer the admin privilege to any other address. Normally, before a token starts trading, the admin will be transferred to the zero address to guarantee that nobody can mint new tokens and inflate the total supply.

1. To initiate a mint, Admin sends a message with op type `mint` to the parent `jetton-minter`. This message is encoded on the transaction he sends his wallet contract, which forwards it to its destination after verifying the signature.

2. Once the `mint` message reaches its destination [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-minter.fc#L56), the parent minter contract, this contract will process the message and verify the message indeed orignated from the admin [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-minter.fc#L57). Then, the contract will alter its persistent state (increase the total supply by the minted amount [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-minter.fc#L65)). The contract will send a message with op type `internal transfer` to the contract holding Becky's SHIB balance [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-minter.fc#L36).

3. Once the `internal transfer` message reaches its destination [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L227), this contract will now process the message and alter its persistent state (increase Becky's SHIB balance by the minted amount [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L108)). This contract will normally send one last message with op type `excesses` back to refund any remaining gas back to Admin's wallet contract and let it know the mint is complete [[code]](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L151).

This is the flow of messages:

<img src="https://i.imgur.com/PnvUPMY.png" width=908 />

The last step of this flow is almost identical to the transfer flow. Similar to the transfer flow, it is also possible to notify the SHIB recipient with a dedicated message so they can handle the payment - remember our "DNS-Superstore" example? I won't add another full user story for this case, but here is the flow of messages just in case:

<img src="https://i.imgur.com/qt81hKC.png" width=936 />

## Who deploys the children contracts?

Let's recall the SHIB contract architecture - one deployed instance of the `jetton-minter` parent and one deployed instance per SHIB holder of `jetton-wallet` contract:

<img src="https://i.imgur.com/S2lFhsY.png" width=513 />

The parent minter contract is naturally deployed by SHIB's creator, probably the Admin user.  But what about the children contracts, who deploys them? The system is efficient and a child contract is only deployed when its owner receives SHIB for the first time. But this sounds a bit tricky, because the owner is not necessarily aware that somebody had sent them SHIB..

If you recall the transfer user story above, receiving SHIB is triggered by the `internal transfer` message. If the recipient child contract of this message has not been deployed, the sender of this message will have to deploy the child! You can see this happening in the code [here](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L67). The `state_init` section of the message is actually responsible for the deployment. You can see it calculated [here](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L55) from the initial code cell of the child (the compiled TVM bytecode for this contract implementation) and its initial data cell.

Since the sender of the `internal transfer` message is never sure if the recipient is deployed or not, it *always* includes the deployment part. TON is clever enough to ignore the deployment element if the contract is already deployed.

## Authenticating the messages between parent and children

In the user stories above, we've seen that a full flow is distributed over multiple messages. The message `internal transfer`, for example, causes its recipient to increase their SHIB balance (as you recall, it was sent in the end of the transfer flow). What would happen if an attacker tries to forge this message and send it to the contract holding their own SHIB balance? If we're not careful, such forgery would result in the attacker's ability to generate themselves new tokens from thin air!

To secure the contract against this forgery, we would need to authenticate that these critical messages that change balances indeed originate from a valid sender. You can see the validation code [here](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L111) - the contract will only handle this message if it was sent by the minter parent (labeled `jetton_master_address` for some reason) or by one of the legal children.

This brings a very interesting question. How do you know if some address is a legal child? Wait a minute, when we wanted before to send a message to some child, how did we know their contract address in the first place?

The addresses for smart contracts on TON are derived from the initial code cell of the contract (the compiled TVM bytecode of its implementation) and the initial data cell of the contract (its initial persistent state on construction). If we know these two values, we can calculate the address of a contract even before it was deployed.

The Jetton code contains a utility function that calculates a child's address - meaning the address of the contract holding Alison's SHIB balance - based on Alison's address. You can see this function [here](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-utils.fc#L27). As you can see, it indeed depends on the initial code cell of the child and its initial data cell.

It's a little tricky to understand why this mechanism is secure. What prevents an attacker from somehow deploying their malicious contract in the address of one of the legal children? To land on a legal address, the malicious contract would have to have the initial code cell of the official children - this has already limited the attacker's ability to add malicious code to this implementation. In addition, the initial data cell guarantees that the child will only obey the correct minter parent since the initial data cell contains its address.

## Handling partial changes when things go wrong

As you recall, the transfer flow is distributed over multiple asynchronous messages. The first message reduces the sender's SHIB balance and the second message increases the recipient's SHIB balance. This makes sense in the happy flow when everything goes well, but what happens if the second message somehow fails?

Most smart contract machines, like Ethereum's EVM, process transactions in a fully atomic and synchronous manner - so if one of the later stages fails, the entire transaction reverts and all state changes caused by this transaction are reverted as well. This mechanism is indeed very easy to reason about. Unfortunately, since messages in TON are neither atomic or synchronous, we would not get this automatic revert out of the box.

So what do we do? We will need to handle the revert flow ourselves. This makes smart contract development on TON a little more difficult.

When the handling of a message on TON fails due to any thrown exception, if this message's `bounce` flag is set, the system would automatically send the failed message back to the sender with the `bounced` flag set. You can read the spec for this *message bouncing* mechanism [here](https://ton.org/docs/#/howto/smart-contract-guidelines?id=paying-for-processing-queries-and-sending-responses).

Let's return to the example above - failure of the second message in the transfer flow. This message fails after the SHIB sender has reduced their SHIB balance by the sent amount. To keep the system consistent, we would somehow need to undo this reduction on failure. How would that work? Assuming the second message was sent with the `bounce` flag set, we can undo the reduction in the sender when a bounced second message is received. You can see the official Jetton code handling bounced messages [here](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L210) and undoing the reduction [here](https://github.com/ton-blockchain/token-contract/blob/2c13d3ef61ca4288293ad65bf0cfeaed83879b93/ft/jetton-wallet.fc#L198).

Do this carefully! When designing a complex message flow on TON, pull out a whiteboard and draw the different message flow diagrams like I did in this post. My favorite tools for this job is the wonderful and open source [Excalidraw](https://excalidraw.com/). Then, start simulating a potential failure and message bounce on every step and make sure your code handles the undo correctly.

Happy coding!

---

Tal is a founder of Orbs Network (https://orbs.com). He's a passionate blockchain developer, open source advocate and a contributor to the TON ecosystem. He is also one of the main developers for TONcoin Fund (https://www.toncoin.fund). For Tal's work on TON, follow on GitHub (https://github.com/ton-defi-org). For Tal's personal work, follow on GitHub (https://github.com/talkol) and Twitter (https://twitter.com/koltal).
