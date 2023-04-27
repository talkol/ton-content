# Creating a stable home for TON libraries

Blockchain development on TON is more than just smart contracts in FunC and Tact. TON developers rely on a plethora of libraries for building anything from dapp frontends to bots and rely on dev tools for compiling and testing their smart contracts.

These libraries and tools are implemented in a variety of languages from TypeScript/JavaScript to Python. Each language usually comes with its own library distrubtion channel like [NPM](https://npmjs.com).

## Who maintains these libraries?

TON is a decentralized ecosystem and as such, many important contributions originate from outside the core team. This has many advantages like exponential growth and a fast pace of innovation, but it does come with a few challenges.

Libraries often originate from one of the businesses active in the TON ecosystem. Let's take [Jetton Minter](https://github.com/ton-blockchain/minter) for example - this tool was originally created by my project, [Orbs](https://orbs.com). The project that invents a library normally maintains and owns it. Even if somebody in the community wants to propose a [PR](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request), it's up to the owning project to decide whether to accept it or not.

But what happens when one of these libraries becomes popular, and other projects start relying on it. This turns the library into some sort of shared infrastructure. Who should maintain it in this case?

## The problem with single ownership of shared libraries

In the first days of [Jetton Minter](https://github.com/ton-blockchain/minter), the tool was hosted on an Orbs-owned repo and served from an Orbs-owned [domain](https://jetton.live). The tool isn't really a core part of Orbs business model. Orbs does't earn any money from it. It was really created for the benefit of the community.

If so, why not leave it under single ownership in the hands of Orbs? We did create it, didn't we?

Well.. this raises a few quesions:

* Orbs is developing other products that are core to its business model. Obviously they'll always be prioritized over something that isn't. What happens if we become too busy to maintain this tool properly?

* What happens if we decide to use the popularity of this tool to push some of our business agendas? For example by introducing a Jetton listing fee?

* What happens if we fail to recognize a security vulnerability in this tool since we focus on other things? This will create a domino effect of vulnerabilities in all of the other projects that depend on it.

## The beauty of the open source model

The open source model gives simple answers to the above questions. The tool is only popular as long as its users make it popular. Many projects will depend on this tool only as long as they're happy to do so.

Shared libraries should always be published under a permissive open source license like [MIT](https://opensource.org/license/mit). This provides a strong guarantee to the library userbase that this library will never be abused. A permissive open source license eliminates lock in. If the maintainer of the library fails its userbase, somebody will [fork](https://docs.github.com/en/get-started/quickstart/fork-a-repo) the library and create a better version.

The open source model keeps libraries honest.

## Surrendering shared libraries to shared ownership

I'm a firm believer that once a library or a tool become a popular shared dependency of many projects, it should transition away from single ownership. It is never in the best interest of the community to keep relying on something that poses too much risk. Single ownership is too much risk. One rogue actor should not be able to impact a large part of our ecosystem.

When [Jetton Minter](https://github.com/ton-blockchain/minter) became popular, Orbs surrendered ownership over it to the community. The repo moved from Orbs to [ton-blockchain](https://github.com/ton-blockchain) and the tool moved to be hosted on a [TON Foundation](https://ton.org) domain.

