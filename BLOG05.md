# TON Hello World: Step by step guide for writing your first smart contract in FunC (part 2) - Testing and Debugging

*by Tal Kol*

---

This is part 2 of my step by step guide for writing your first smart contract for TON blockchain. If you haven't read [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func), please do so first. I'm assuming that you have a working environment with a simple counter contract in FunC that you were able to build and deploy.

## Why should you test?

Testing is a big part of smart contract development. Smart contracts often deal with money and we don't want any of our users losing money because the smart contract had a bug. This is why it's normally expected from smart contract devleopers to share a test suite with the FunC implementation.

A thorough test suite is a good signal to your users that you've taken your role as a contract developer seriously. I would personally be very hesitant to deposit a substantial amount of money in any contract that has no tests. Since *code is law*, any bug in the contract code is also part of the agreement, so a user wouldn't really have anyone to blame for money lost, but themselves.

Personally, I don't view testing as an afterthought taking place only when your code is completed. If done correctly, tests can be a powerful aid to the development process itself from the beginning, that will allow you to write better code faster.

## Oh so many ways to test

*Warning - this section is a bit more advanced than beginner, feel free to skip it if you trust my judgement of how to test.*

Because testing is such as big deal in smart contract development, there's a surprising amount of tools and infrastructure in the TON ecosystem devoted to this topic. Before jumping in to the methodology that I believe in, I want to give a quick overview of the plethora of testing tools available out there:

1. **Deploying your contract to testnet** - Testnet is a live alternative instance of the entire TON blockchain where TON coin isn't the real deal and is [free to get](https://t.me/testgiver_ton_bot). This instance is obviously not as secure as mainnet, but offers an interesting staging environment where you can play. There are actually several different testnets on TON at the time of writing, "[testnet](https://testnet.tonscan.org/)" and "[sandbox](https://test.tonwhales.com/explorer)".

2. **Debug and test with toncli** - [toncli](https://github.com/disintar/toncli) is a command-line tool written in Python that runs on your machine and supports [debug](https://github.com/disintar/toncli/blob/master/docs/advanced/transaction_debug.md) and [unit tests](https://github.com/disintar/toncli/blob/master/docs/advanced/func_tests_new.md) for FunC contracts where the tests are also written in FunC ([example](https://github.com/BorysMinaiev/func-contest-1-tests-playground/blob/main/task-1/tests/test.fc)).

3. **Local blockchain with MyLocalTon** - [MyLocalTon](https://github.com/neodiX42/MyLocalTon) is a Java-based desktop executable that runs a personal local instance of TON blockchain on your machine that you can deploy contracts to and interact with.

4. **Bare-bones TVM with ton-contract-executor** - [ton-contract-executor](https://github.com/Naltox/ton-contract-executor) is bare-bones version of just the [TVM](https://ton-blockchain.github.io/docs/tvm.pdf) running on [WebAssembly](https://webassembly.org/) with a thin JavaScript wrapper that allows test interactions from TypeScript.

5. **Deploying beta contracts to mainnet** - This form of "testing in production" simply deploys alternative beta versions of your contracts to mainnet and uses real (not free) TON coin to play with them in a real environment. If you found a bug, you simply deploy new fixed beta versions and waste a little more money.

So which method should you choose? You definitely don't need all of them.

My team started building smart contracts on Ethereum in 2017, we've witnessed the evolution of the art of smart contract development almost from its infancy. While I'm well aware of [fundamental differences](https://society.ton.org/six-unique-aspects-of-ton-blockchain-that-will-surprise-solidity-developers) between TON and the EVM, testing between the two platforms is not fundamentally different. All of the above approaches appeared on Ethereum at one point or another. And all of them practically disappeared - except two - the last two.

1. Testnets were once popular on Ethereum (funny names like Ropsten, Rinkeby and Goerli) but turned out to be the worst possible tradeoff between convenience and realism - they're slow and often more difficult to work with than mainnet (many wallets simply don't work) and useless for integration tests with other contracts (eg. your contract interacts with somebody else's token) because nobody bothers to maintain up-to-date versions of their projects on testnet.

2. Testing Solidity with Solidity proved to be too cumbersome as smart contract languages are inherently limited and restrictive by design and efficient testing seems to flourish on freeform languages like JavaScript. Trying to code a complex expectation in Solidity or simulate a difficult scenario is just too painful.

3. Local desktop versions of the entire blockchain, like [Ganache UI](https://trufflesuite.com/ganache/), proved to be too slow for unit tests and ineffective for integration tests (for the same reason as testnets). They also don't play nicely with [CI](https://docs.github.com/en/actions/automating-builds-and-tests/about-continuous-integration). People often confuse [ganache-cli](https://github.com/trufflesuite/ganache) with a local blockchain, but it is actually a bare-bones EVM implemented in JavaScript.

4. Bare-bones EVM turned out to be the holy grail. Most of the testing on Ethereum today takes place on [Hardhat](https://hardhat.org/) and Hardhat is a thin wrapper around [EthereumJs](https://github.com/ethereumjs/ethereumjs-monorepo) which is an EVM implementation in JavaScript. This approach turned out to be the most convenient (ultra-fast CI-friendly unit tests) as well as realistic where it matters (live lazy-loaded [forks](https://hardhat.org/hardhat-network/docs/guides/forking-other-networks) of mainnets for integration tests).

5. Testing in production is useful for the last mile. Ethereum has less than [5 million](https://www.fool.com/the-ascent/cryptocurrency/articles/more-people-own-ethereum-than-ever-before-heres-why/) active users yet over [40 million](https://cryptopotato.com/over-44-million-contracts-deployed-to-ethereum-since-genesis-research/) deployed contracts. The vast majority of all deployed contracts on Ethereum mainnet are beta versions that developers deployed for a few tests and then abandoned. Don't feel bad about polluting mainnet with garbage, nobody cares.

I saw a few people in the TON ecosystem complaining that [ton-contract-executor](https://github.com/Naltox/ton-contract-executor) is missing some important features like multi-contract tests. These features are conceptually very easy to add (they're all in the thin JavaScript wrapper, no need to touch the WebAssembly part). With some community love we can easily bring this amazing tool to be feature complete. Another killer feature of ton-contract-executor is its ability to load code and data cells [from mainnet](https://github.com/Naltox/ton-contract-executor/blob/8b352d0cf96553e9ded19a102a890e17c973d017/README.md?plain=1#L81), which will allow us to lazily "fork" mainnet in TON just like Hardhat/Ganache do.

After carefully considering all available approaches, I hope I convinced you why we're going to spend 90% of our time testing with approach (4) and 10% of our time testing with approach (5). We're going to conveniently forget about the other approaches and avoid using them at all.

## Step 1: Setting up a TypeScript environment

Our secret weapon for testing is going to be JavaScript. Learning how to interact with TON from JavaScript is a good investment since web apps (your dapp client) run in web browsers and are developed almost exclusively in JavaScript. This means that you're going to have to learn this at some point or another. The efficiency of our approach lies by relying on JavaScript for *just about everything* that FunC can't do - from building to deploying and now testing.

I now realize that I did mention TypeScript in [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func) yet neglected to go over the exact setup. Let's fix this now. Make sure that Node.js is properly installed by running `node -v` (I'm using `v17.3.0` but other recent version above '16' should be fine).

Create `package.json` with the following content:

```json
{
  "name": "",
  "description": "",
  "version": "0.0.0",
  "scripts": {
    "test": "mocha --exit test/**/*.spec.ts"
  },
  "devDependencies": {
    "@types/bn.js": "^5.1.0",
    "@types/chai": "^4.3.0",
    "@types/mocha": "^9.0.0",
    "chai": "^4.3.4",
    "chai-bn": "^0.3.1",
    "mocha": "^9.1.3",
    "ton": "^9.6.3",
    "ton-contract-executor": "^0.4.8",
    "ton-crypto": "^3.1.0",
    "ts-node": "^10.4.0",
    "typescript": "^4.5.4"
  },
  "mocha": {
    "require": [
      "chai",
      "ts-node/register"
    ],
    "timeout": 20000
  },
  "engines": {
    "node": ">=16.0.0"
  }
}
```

The main libraries we're relying on are:
* [typescript](https://www.typescriptlang.org/) and [ts-node](https://www.npmjs.com/package/ts-node) for adding types to JavaScript so we can maintain our sanity
* [mocha](https://mochajs.org/) for running our tests
* [chai](https://www.npmjs.com/package/chai) for writing [expectations](https://www.chaijs.com/api/bdd/) and [chai-bn](https://www.npmjs.com/package/chai-bn) for adding support for arbitrarily large [big numbers](https://github.com/indutny/bn.js/)
* [ton](https://www.npmjs.com/package/ton) and [ton-crypto](https://www.npmjs.com/package/ton) for basic interactions with TON blockchain primitives
* [ton-contract-executor](https://www.npmjs.com/package/ton-contract-executor) for running the TVM (that executes our contract code) right inside Node.js

Also create `tsconfig.json` with the [following content](https://github.com/ton-defi-org/tonstarter-contracts/blob/main/tsconfig.json). Now run `npm install` in terminal.

Alternatively, if you want to save time, you can simply copy my working project skeleton from [https://github.com/ton-defi-org/tonstarter-contracts](https://github.com/ton-defi-org/tonstarter-contracts).

## Step 2: Loading our contract in a test

Quick reminder, in [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func) we compiled our smart contract code in step 5 and generated the file `counter.cell` - which contains the TVM bytecode for our contract in cell format (code cell). In step 6, before deploying our contract, we initialized its persistent storage by implementing a simple JavaScript function to generate its initial data cell:

```ts
import { beginCell } from "ton";

function initData(initialCounterValue: number) {
  return beginCell().storeUint(initialCounterValue, 64).endCell();
}
```

In order to load our contract into [ton-contract-executor](https://github.com/Naltox/ton-contract-executor), all we have to do is provide it with both the initial code cell and the initial data cell:

```ts
import chai, { expect } from "chai";
import chaiBN from "chai-bn";
import BN from "bn.js";
chai.use(chaiBN(BN));

import * as fs from "fs";
import { Cell } from "ton";
import { SmartContract } from "ton-contract-executor";

describe("Counter tests", () => {
  let contract: SmartContract;

  beforeEach(async () => {
    const initCodeCell = Cell.fromBoc(fs.readFileSync("counter.cell"))[0]; // compilation output from part 1 step 5
    const initDataCell = initData(17); // the function we've implemented just now
    contract = await SmartContract.fromCell(initCodeCell, initDataCell);
  });

  it("should run the first test", async () => {
    // currently empty, will place a test body here soon
  });
});
```

Place this code inside `test/counter.spec.ts` as this is where we instructed mocha to look for tests in `package.json`. Make sure `counter.cell` is found in your project root.

As you can see, we load our contract in the `beforeEach()` clause, meaning our contract will be reloaded before every single test. This best practice will guarantee that our different tests will not pollute each other, even if they change the contract state.

To execute our (now empty) test suite, run in terminal `npm test` from the project root. This should be the result:

```
  Counter tests
    ✔ should run the first test (245ms)

  1 passing (1s)
```

What happens when the tests run? A Node.js process is executed which runs our JavaScript code. The code uses ton-contract-executor to load into the JavaScript runtime a live instance of the [TVM](https://ton-blockchain.github.io/docs/tvm.pdf) - the component from TON's codebase charged with executing smart contracts - and instructs the TVM to load our contract. Since all of this happens in-process, our tests will be blazingly fast and start instantaneously.

## Step 3: Testing a Getter

