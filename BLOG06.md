# What is blockchain? What is a smart contract? What is gas?

*by Tal Kol*

---

This is an introductory post aimed at giving some context as to what we're doing here. What is special about blockchain and why are we using it? This post will also define some basic terms like *smart contract* and *gas*. If you're feeling lost, this is a good place to start.

## The old Internet (Web2)

The term [Web 2.0](https://en.wikipedia.org/wiki/Web_2.0) is used to describe the traditional Internet as you know it. This Internet is operated primarily by large corporations like [Google](https://abc.xyz/investor/). The formal reason for the existance of corporations is to [generate profit for their shareholders](https://corpgov.law.harvard.edu/2020/08/05/on-the-purpose-and-objective-of-the-corporation/). This means that the global good or the benefits of their users are a byproduct and not the end goal.

Let's take a Web2 service like [Gmail](https://gmail.com). You, as a user of this service, are inherently not equal to the creator of the service, Google. We call this property ***centralization***. Centralized services don't provide users with true ownership. If Google decides that you've breached its terms of service, for example, it is allowed to take your email access away from you. Centralized services are also ***permissioned***, meaning that to use the service and send an email you must ask for permission. If Google decides that your email is spam, it is not required to send it.

Centralization is based on ***trust***. Users allow Google to maintain a position of power because they trust Google with this power.

## The new Internet (Web3)

Many of us view the Internet as a common good. We see it as a tool that turns the world into a global village. A tools that allows users to communicate and form communities. As such, we would like to see a shift of power from corporations to users. Web3 is the implementation of this ideal, what we would like to see as the next stage of evolution of the Internet.

Under this ideal, you, as a user of a Web3 service, must be inherently equal to the creator of the service. We call this property ***decentralization***. Decentralized services provide users with true ownership. This is not only true for data, but also for assets. Decentralized assets like your Bitcoins or your TON coins are yours, and nobody can take them away from you. Decentralized services are also ***permissionless***, meaning that to transfer your TON coin to somebody else requires nobody's permission but your own. Nobody can stop this from happening or censor you.

Decentralization allows systems to be ***trustless***. Since there are no positions of authority, authority cannot be abused to hurt users.

## The blockchain

The ideal of Web3 sounds great on paper, but is it practical? As developers, how can we implement services where we are inherently equal to our users? Implementing a service normally requires the developer to write a *backend*. This backend runs on some server. Who owns the server? the developer. The developer can change the server without asking or even take it down. This relationship is inherently not equal. Backend servers are ***centralized***.

Blockchain technology was invented to solve this problem and allow developers to create backends that are ***decentralized***. Who runs this backend? The users do. Since the relationship is equal, any user that wishes to participate in running this backend is allowed to do so. Blockchain runs as a collaboration between its users.

Collaboration is governed by ***consensus***. For an execution result to hold true, multiple users, a majority to be exact, must all vote to confirm the result. This makes blockchains very inefficient since every calculation must be executed by many users. This also makes blockchains expensive to run compared to a traditional centralized server.

## The token

We mentioned that execution results require voting. How does it work? Is it - one user, one vote - like in democratic elections? It turns out that this doesn't work on the Internet due to something known as the [Sybil attack](https://en.wikipedia.org/wiki/Sybil_attack). It is very easy to create fake users on the Internet. Since Web3 is decentralized, we can't have a centralized source of authority that decides who's fake and who's real.

A popular decentralized solution to this problem is to revolve voting rights around a ***token***. If you own 10 tokens, you have 10 votes. Tokens cannot be faked, it is very easy to tell between a real token and a fake one. The TON blockchain revolves around the [TON coin](https://coinmarketcap.com/currencies/toncoin/). The Ethereum blockchain revolves around [Ether](https://coinmarketcap.com/currencies/ethereum/). This means that every blockchain is also an economy. The token acts as an incentivization tool to make sure that the decentralized community is all pulling in the same direction.

## Network validators

All blockchains are networks because they are operated by a group of users. Users that do the heavy lifting of operating the network and actively participate in the consensus process are called ***validators***. The voting weight of every validator is proportional to the amount of tokens they hold. To keep validators honest, they are normally required to put their tokens **at stake**. If the consensus deems that a validator is dishonest, their tokens will be taken away as punishment. This governance process is called ***proof-of-stake***.

Being a network validator is usually hard work. You need to run the ***blockchain node*** code on a server that you own and stake it with a lot of tokens. Smaller users that want to participate but don't have enough tokens to warrant going to all this effort can often delegate their tokens to one of the validators. These participants are called ***nominators***.

## Gas fees

We said earlier that blockchains are economies. The equipment for network validators is not free, so they must be paid for their efforts. Payment naturally takes place with the token of the blockchain. On the TON blockchain, users pay fees using the TON coin. TON network validators earn TON coin for performing the validation process and executing all the apps that are running on the blockchain.

When a user is performing some action on the blockchain, they must send a ***transaction***. The transaction includes a fee payment called ***gas***. The analogy comes from cars. Just like a car needs gas to execute its purpose, so does a blockchain transaction. Users must ***sign*** transactions using their blockchain ***wallets***. This signature guarantees that only the owner of the wallet can authorize the payment of gas and sending the transaction.

## Dapps

We said earlier that the purpose of blockchains is to run decentralized backends. A simpler name for these services that run on the blockchain network is *apps* - decentralized apps to be exact or ***dapps*** for short. Developers create dapps and have network validators execute them. Users interact with dapps by sending them transactions. The developer of a dapp is equal to the dapp's users. The developer should have no special priviliges since the app is decentralized.

Let's reiterate over the last point with an example. Let's take a Web2 service like Google Search. The developer of the service, Google, ranks search results for the benefit of users. Google enjoys this position of power and is allowed to promote its own products in search results. For example, when searching for "storage", Google can promote the result "Google Drive" over a competitor like "Dropbox". In a Web3 version of Google Search, the developer of the service will not be allowed to promote their own products in search.

## Smart contracts

Every Web2 service like Google Search has terms of service. If a user feels that they were wronged under these terms, they can sue and ask a judge to rule on the dispute. Web3 is decentralized and cannot have centralized sources of authority like judges. On blockchain, code is law. The code of the dapp is the only agreement between its users. Unlike traditional legal agreements, this agreement is not open to interpretation. Code always executes in the same way.

Blockchain replaces legal contracts with code. The code of the dapp is therefore called a ***smart contract***. Before users decide to participate in a dapp, by sending it transactions for example, they are expected to [review](https://verifier.ton.org) the dapp's source code to understand its terms. Just like you wouldn't sign a lease contract for your apartment without reading the contract, you should not sign a transaction without reviewing the smart contract. Since not all users are tech savvy enough to do this, communities often rely on each other for this purpose.

After the developer finished writing the smart contract, the act of publishing this contract to the blockchain is called ***deployment***. The contract code is deployed on to the chain where everybody can find it by its ***contract address***.

## Blocks and explorers

We said earlier that network validators must vote on the execution result of every transaction. To streamline this process, groups of transactions are batched together in ***blocks***. Every block of transactions gets its own ***block number*** and undergoes the consensus process where a majority of network validators is required to approve it. When you order all the blocks one after the other you get a chain of blocks - this is the source of the word ***blockchain***.

After sending a transaction, a user must wait until this transaction is included in a block. A new block on TON blockchain is created every 5 seconds on average. Users can inspect transactions, check if they succeeded or not and see which block they were added to by using a tool called a ***block explorer***, or an [explorer](https://tonscan.org) for short.

## So what is blockchain good for?

In this post we mostly covered what blockchain is and defined a lot of the terminology involved. It sounds like an awful lot of trouble to achieve abstract benefits like *decentralization* and *trustlessness*. Can we give a more practical example where blockchain can improve your life?

Back in 2018 I wrote a well received 2-part post series about this very topic. Part 1 is ["How a Blockchain Can Help You on a Deserted Island"](https://talkol.medium.com/why-decentralized-consensus-blockchain-is-good-for-business-5ff263468210) and part 2 is ["How to Run a Blockchain on a Deserted Island with Pen and Paper"](https://talkol.medium.com/how-to-run-a-blockchain-on-a-deserted-island-with-pen-and-paper-899949ec555b). If you like the old TV series [Lost](https://www.imdb.com/title/tt0411008/), give it a read, I'm sure that you'll enjoy it.

Happy coding!

---

*Tal is a founder of [Orbs Network](https://orbs.com). He's a passionate blockchain developer, open source advocate and a contributor to the TON ecosystem. He is also one of the main developers for [TONcoin Fund](https://www.toncoin.fund). Follow Tal on [GitHub](https://github.com/talkol) and [Twitter](https://twitter.com/koltal). If you found any mistakes in this post, please let Tal know on [Telegram](https://t.me/talkol).*
