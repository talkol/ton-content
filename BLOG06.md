# What is blockchain? What is a smart contract? What is gas?

*by Tal Kol*

---

This is an introductory post aimed at giving some context as to what we're doing here. What is special about blockchain and why are we using it? This post will also define some basic terms like *smart contract* and *gas*. If you're feeling lost, this is a good place to start.

## The old Internet (Web2)

The term [Web 2.0](https://en.wikipedia.org/wiki/Web_2.0) is used to describe the traditional Internet as you know it. This Internet is operated primarily by large corporations like [Google](https://abc.xyz/investor/). The formal reason for the existance of corporations is to [generate profit for their shareholders](https://corpgov.law.harvard.edu/2020/08/05/on-the-purpose-and-objective-of-the-corporation/). This means that the global good or the benefits of their users are a byproduct and not the end goal.

Let's take a Web2 service like [Gmail](https://gmail.com). You, as a user of this service, are inherently not equal to the creator of the service, Google. We call this property ***centralization***. Centralized services don't provide users with true ownership. If Google decides that you've breached its terms of service, for example, it is allowed to take your email access away from you. Centralized services are also ***permissioned***, meaning that to use the service and send an email you must ask for permission. If Google decides that your email is spam, it is not required to send it.

## The new Internet (Web3)

Many of us view the Internet as a common good. We see it as a tool that turns the world into a global village. A tools that allows users to communicate and form communities. As such, we would like to see a shift of power from corporations to users. Web3 is the implementation of this ideal, what we would like to see as the next stage of evolution of the Internet.

Under this ideal, you, as a user of a Web3 service, must be inherently equal to the creator of the service. We call this property ***decentralization***. Decentralized services provide users with true ownership. This is not only true for data, but also for assets. Decentralized assets like your Bitcoins or your TON coins are yours, and nobody can take them away from you. Decentralized services are also ***permissionless***, meaning that to transfer your TON coin to somebody else requires nobody's permission but your own. Nobody can stop this from happening or censor you.

## The blockchain

The ideal of Web3 sounds great on paper, but is it practical? As developers, how can we implement services where we are inherently equal to our users? Implementing a service normally requires the developer to write a *backend*. This backend runs on some server. Who owns the server? the developer. The developer can change the server without asking or even take it down. This relationship is inherently not equal. Backend servers are ***centralized***.

Blockchain technology was invented to solve this problem and allow developers to create backends that are ***decentralized***. Who runs this backend? The users do. Since the relationship is equal, any user that wishes to participate in running this backend is allowed to do so. Blockchain runs as a collaboration between its users.

Collaboration is governed by ***consensus***. For an execution result to hold true, multiple users, a majority to be exact, must all vote to confirm the result. This makes blockchains very inefficient since every calculation must be executed by many users. This also makes blockchains expensive to run compared to a traditional centralized server.

## The token

We mentioned that execution results require voting. How does it work? Is it one user one vote like in democratic elections? It turns out that this doesn't work on the Internet due to something known as the [Sybil attack](https://en.wikipedia.org/wiki/Sybil_attack). It is very easy to create fake users on the Internet. Since Web3 is decentralized, we can't have a centralized source of authority that decides who's fake and who's real.

A good decentralized solution is to revolve voting rights around a ***token***. If you own 10 tokens, you have 10 votes. The TON blockchain revolves around [TON coin](https://coinmarketcap.com/currencies/toncoin/). The Ethereum blockchain revolves around [Ether](https://coinmarketcap.com/currencies/ethereum/).
