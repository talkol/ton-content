# Creating a stable home for TON libraries

Blockchain development on TON is more than just smart contracts in FunC and Tact. TON developers rely on a plethora of libraries for building anything from dapp frontends to bots and rely on diverse dev tools for compiling and testing their smart contracts.

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

Shared libraries should always be published under a permissive open source license like [MIT](https://opensource.org/license/mit). This provides a strong guarantee to the library userbase that this library will never be abused. A permissive open source license eliminates lock-in. If the maintainer of the library fails its userbase, somebody will [fork](https://docs.github.com/en/get-started/quickstart/fork-a-repo) the library and create a better version.

The open source model keeps libraries honest.

## Surrendering shared libraries to shared ownership

I'm a firm believer that once a library or a tool become a popular shared dependency of many projects, it should transition away from single ownership. It is never in the best interest of the community to keep relying on something that poses too much risk. Single ownership is too much risk. One rogue actor should not be able to impact a large part of our ecosystem.

When [Jetton Minter](https://github.com/ton-blockchain/minter) became popular, Orbs surrendered ownership over it to the community. The repo moved from Orbs to [ton-blockchain](https://github.com/ton-blockchain) and the tool moved to be hosted on a [TON Foundation domain](https://minter.ton.org).

Even if we hadn't done these things, since the tool was [MIT](https://github.com/ton-blockchain/minter/blob/main/LICENSE), if these steps would have improved the tool for the community, somebody else would have forked and done them for us.

## The challenge with NPM

When you publish a TypeScript/JavaScript library for import by other developers, you normally publish it to [NPM](https://npmjs.com). The problem with NPM is that it doesn't follow the open source no lock-in model. NPM packages have a well-known name that people import. This name, similar to a domain name, cannot be forked.

NPM names are given for free on a first-come first-serve basis. This means that they're very prone to squatting. Names that we could have used often turn out to be already registered even if they're still unused. This also poses a security issue. Is the NPM library [ton-crypto](https://www.npmjs.com/package/ton-crypto) official or not? Can you use it safely with your secret mnemonic?

## A stable home for shared TON libraries

In the effort to make the TON ecosystem safer and more stable for developers, we have been looking for a shared home where popular shared libraries in the TON ecosystem can be transitioned to shared ownership.

This place must not be owned by a single company or a single person. It must operate under a code of conduct where a group of well-known contributors trusted by the TON community can make decisions together for the benefit of the ecosystem. Unresolved disputes should escalate to the TON Foundation.

This home is for peripheral libraries and tools. Many of those are not maintained by the core team, so this home should be able to give write access to external contributors. Nevertheless, representatives from the core team will have admin privileges.

* https://github.com/ton-org

  We are creating a new GitHub organization labeled after the primary `ton.org` domain.

  We've chosen a destination outside the [ton-blockchain](https://github.com/ton-blockchain) GitHub organization, since this was reserved for the core node infrastructure and protocol and is primarily maintained by the core team and not external contributors from the community.
  
  We've chosen a destination different from the [ton-community](https://github.com/ton-community) GitHub organization so it can focus on a smaller and more official list of core libraries whereas community is open for almost anything.
  
* https://www.npmjs.com/org/ton

  We were finally successful in acquiring the `@ton` NPM organization!
  
  We are now on par with other prominent blockchain ecosystems like [@solana](https://www.npmjs.com/org/solana) that list all of the official core libraries under one namespace.
  
  Using an NPM organization gives us full control of all names under this namespace. This resolves many of the challenges we had with NPM. Squatting is not possible under this namespace since all sub-names are ours and users will be able to identify safe official libraries with ease.
  
## A first example

One of the most popular TypeScript libraries in the TON ecosystem is `ton-core`. This is not only a shared dependency for many projects, but other shared libraries often depend on it too.

The library was originally contributed by [TonWhales](https://tonwhales.com) and maintained exclusively by the Whales team - credit given where credit is due. We want the library to transition to shared ownership where multiple contributors from multiple entities will be able to work on it together.

* The shared ownership fork of the library will be hosted on https://github.com/ton-org/ton-core

* The shared ownership fork of the library will be published on NPM as `@ton/core`

In the spirit of decentralization, I call teams to support this transition of popular shared libraries to shared ownership and move the original repositories so we can avoid unnecessary forks. We've been thinking about this transition for a long time with support of the TON Foundation. The only motivation is improving the ecosystem as a whole.

Happy coding!
