# TON Hello World: Step by step guide for writing your first smart contract in FunC (part 2) - Testing and Debugging

*by Tal Kol*

---

This is part 2 of my step by step guide for writing your first smart contract for TON blockchain. If you haven't read [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func), please do so first. I'm assuming that you have a working environment with a simple counter contract in FunC that you were able to build and deploy.

## Why should you test?

Testing is a big part of smart contract development. Smart contracts often deal with money and we don't want any of our users losing money because the smart contract had a bug. This is why it's normally expected from smart contract devleopers to share a test suite with their FunC implementation.

A thorough test suite is a good signal to your users that you've taken your role as a contract developer seriously. I would personally be very hesitant to deposit a substantial amount of money in any contract that has no tests. Since *code is law*, any bug in the contract code is also part of the agreement, so a user wouldn't really have anyone to blame for money lost, but themselves.

Personally, I don't view testing as an afterthought taking place only when your code is complete. If done correctly, tests can be a powerful aid to the development process itself from the beginning, that will allow you to write better code faster.

## Oh so many ways to test

*Warning - this section is a bit more advanced than beginner, feel free to skip it if you trust my judgement of how to test.*

Because testing is such as big deal in smart contract development, there's a surprising amount of tools and infrastructure in the TON ecosystem devoted to this topic. Before jumping in to the methodology that I believe in, I want to give a quick overview of the plethora of testing tools that are available out there:

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
* [mocha](https://mochajs.org/) for running our tests (if you're a jest fan, I think mocha is better for backend and jest for frontend)
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
import { Cell, Address, InternalMessage, CommonMessageInfo, CellMessage } from "ton";
import { SmartContract } from "ton-contract-executor";
const zeroAddress = Address.parseRaw("0:0000000000000000000000000000000000000000000000000000000000000000");

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

Now that the boilerplate is behind us, we can finally focus on writing some test logic. Ideally, we want to test through every execution path of our contract to make sure it's working. Let's start with something simple, our getter. Quick reminder, in [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func) step 4 we implemented a getter in FunC that looked like this:

```
int counter() method_id {        ;; getter declaration - returns int as result
  var (counter) = load_data();   ;; call our read utility function to load value
  return counter;
}
```

As you recall, our test initializes our contract with a data cell via `initData(17)` - so the initial counter value is 17. Therefore, we expect the getter to return `17` after initialization. Translated to TypeScript:

```ts
  it("should get counter value", async () => {
    const call = await contract.invokeGetMethod("counter", []);
    expect(call.result[0]).to.be.bignumber.equal(new BN(17));
  });
```

Replace the `it("should run the first test", ...` from before with this new test and run the tests with `npm test`.

Notice a few interesting things in our test. First, the FunC type returned from our getter is `int`. This TVM number type is [257 bit long](https://ton.org/docs/#/func/types?id=atomic-types) (256 signed) so it supports huge virtually unbounded numbers. The native JavaScript `number` type is limited to [64 bit](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) so it cannot necessarily hold the result. We use a [big number](https://github.com/indutny/bn.js/) library to work around this limitation.

Second, our getter does not take any input arguments, but if it did, we can easily pass them in `invokeGetMethod()`. You can see an [example](https://github.com/Naltox/ton-contract-executor/blob/8b352d0cf96553e9ded19a102a890e17c973d017/src/smartContract/SmartContract.spec.ts#L60) of this in ton-contract-executor's own test suite.

## Step 4: Testing a Message

While getters are read-only operations that don't change contract state, messages are used to modify state through user transactions. Reminder, we've implemented the following message handler in [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func) step 4:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {  ;; well known function signature
  int op = in_msg_body~load_uint(32);                                     ;; parse the operation type encoded in the beginning of msg body
  var (counter) = load_data();                                            ;; call our read utility function to load values from storage
  if (op == 1) {                                                          ;; handle op #1 = increment
    save_data(counter + 1);                                               ;; call our write utility function to persist values to storage
  }
}
```

Let's write a test that sends a message with op #1 = *increment*. To do that, let's first encode our message as a cell. As you recall from [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func), the format of [internal messages](https://ton.org/docs/#/howto/smart-contract-guidelines?id=internal-messages) on TON normally begins with a 32 bit unsigned integer that defines the op (`1` in our case) and then a 64 bit unsigned integer that defines the query_id (which we don't really need here, so we pass `0`). If our message had any other custom arguments, they would come next.

```ts
import { Cell, beginCell } from "ton";

function encodeIncrementMessage(): Cell {
  const op = 1;
  return beginCell().storeUint(op, 32).storeUint(0, 64).endCell();
}
```


Let's send the message and see that our contract indeed increments the counter value as expected:

```ts
  it("should increment the counter value", async () => {
    const message = encodeIncrementMessage();
    const send = await contract.sendInternalMessage(
      new InternalMessage({
        from: Address.parse('EQD4FPq-PRDieyQKkizFTRtSDyucUIqrj0v_zXJmqaDp6_0t'), // address of the sender of the message
        to: zeroAddress, // ignored, this is assumed to be our contract instance
        value: 0, // are we sending any TON coins with this message
        bounce: true, // do we allow this message to bounce back on error
        body: new CommonMessageInfo({ 
          body: new CellMessage(message)
        })
      })
    );
    expect(send.type).to.equal("success");

    const call = await contract.invokeGetMethod("counter", []);
    expect(call.result[0]).to.be.bignumber.equal(new BN(18));
  });
```

Notice that we already know from the previous test that the counter is indeed initialized to `17`, so if our message was successful, we can use the getter to get the counter value and make sure it has been incremented to `18`.

Also notice that we only send internal messages to contracts. TON supports [external messages](https://ton.org/docs/#/smart-contracts/messages) as well, but as a dapp developer you're never expected to use them. External messages are used for very specific contracts, mainly wallet contracts, that you would normally never have to write by yourself. You can safely ignore them.

## Step 5: Debugging by dumping variables

Testing is fun as long as everything works as expected. But what happens when something doesn't work and you're not sure where the problem is? The most convenient method I found to debug your FunC code is to add debug prints in strategic places. This is very similar to using `console.log(variable)` in JavaScript to [print](https://developer.mozilla.org/en-US/docs/Web/API/Console/log) the value of a variable.

The TVM has a special function for [dumping variables](https://ton.org/docs/#/func/builtins?id=dump-variable) in debug. Run `~dump variable_name;` in your FunC code to use it.

For example, let's say we're trying to send some TON coin to our contract on a message. We can do this by passing a non-zero `value` in TypeScript under `new InternalMessage()` above. In FunC, this value should arrive under the `msg_value` argument of `recv_internal()`. Let's print this incoming value in FunC to make sure that it indeed works as expected. I added the debug print as the first line of our `recv_internal()` message handler from before:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {  ;; well known function signature
  ~dump msg_value;                                                        ;; this debug print line was just added
  int op = in_msg_body~load_uint(32);                                     ;; parse the operation type encoded in the beginning of msg body
  var (counter) = load_data();                                            ;; call our read utility function to load values from storage
  if (op == 1) {                                                          ;; handle op #1 = increment
    save_data(counter + 1);                                               ;; call our write utility function to persist values to storage
  }
}
```

Since we changed our FunC code, we'll have to rebuild the contract to see the effect and generate a new `counter.cell`.

We're going to have to make another small change. By default, ton-contract-executor runs the TVM without debug mode active since it's faster. To enable debug mode TVM, we're going to have to change the initialization of our contract in TypeScript in `beforeEach`:

```ts
  beforeEach(async () => {
    const initCodeCell = Cell.fromBoc(fs.readFileSync("counter.cell"))[0]; // same like before
    const initDataCell = initData(17); // same like before
    contract = await SmartContract.fromCell(initCodeCell, initDataCell, {
      debug: true // this new addition will enable debug mode
    });
  });
```

Debug output will be printed to terminal when you run your tests.

## Step 6: Testing in production (without testnet)

Steps 2-5 above are all part of approach (4) - where I promised to spend 90% of our testing time. These tests are very fast to run (there's nothing faster than an in-process instance of a bare-bones TVM) and are very CI-friendly. They are also free and don't require you to spend any TON coin. These tests should give you the majority of confidence that your code is actually working. If there are testing scenarios that you desire but ton-contract-executor does not support smoothly like multi contract, please [open issues](https://github.com/Naltox/ton-contract-executor/issues) and contribute. I'm confident that as a community, this is where we should invest most of our efforts.

What about the remaining 10%? All of our tests so far worked inside a lab. Before we're launching our contract, we should run some tests in the wild! This is what approach (5) is all about.

From a technical perspective, this is actually the simplest approach of all. You don't need to do anything special. Get some TON coin and deploy your contract to mainnet! The process was covered in detail in [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func). Then, interact with your contract manually just like your users will. This may require you to implement a dapp client or frontend, something we'll only discuss in the next parts of this series.

If you don't have a dapp client, you can also interact with your contract programatically. We've covered this in [part 1](https://society.ton.org/ton-hello-world-step-by-step-guide-for-writing-your-first-smart-contract-in-func) at step 7.

If this step is so easy, why am I devoting so much time to discuss it? Because, from my experience, most dapp developers are reluctant to do so. Instead of testing on mainnet, they prefer to work on testnet. In my eyes, this is a waste of time. Let me attempt to refute any reasons to use testnet one last time:

* *"testnet is as easy to work with as mainnet"* - False. Testnet requires special wallets and special explorers. There are also multiple instances of testnet like TonHub's sandbox. This mess is going to cost you time to sort out. I've seen too many developers deploying their contract to testnet and then trying to access it on sandbox without understanding why they don't see anything deployed.

* *"mainnet is more expensive since it costs real TON coin to use"* - False. Deploying your contract to mainnet costs around 10 cents. Your time costs more. Let's say an hour of your time is only worth the minimum wage in the US (a little over $7), if working on mainnet saves you an hour, you can deploy your contract 70 times without feeling guilty that you're wasting money.

* *"testnet is a good simulation of mainnet"* - False. Nobody really cares about testnet since it's not a production network. Are you certain that validators on testnet are running the latest node versions? Are all config parameters like gas costs identical to mainnet? Are all contracts by other teams that you may be relying on deployed to testnet?

* *"I don't want to pollute mainnet with abandoned test contracts"* - Don't worry about it. Users won't care since the chance of them reaching your unadvertised contract address by accident is zero. Validators won't care since you paid them for this service, they enjoy the traction. Also, TON has an auto-cleanup mechanism baked in, your contract will eventually run out of gas of rent and will be destroyed automatically.

## Summary

This is the end of part 2 in this blog series. We learned how to test and debug our FunC contract effectively. The next parts will focus on enriching our contract with more features and implementing additional aspects of our dapp such as its web app client.

If you want to incorporate the methodologies explained above in your project, you are welcome to use a template Github repo that contains all the boilerplate discussed:

https://github.com/ton-defi-org/tonstarter-contracts

Happy coding!

---

Tal is a founder of Orbs Network (https://orbs.com). He's a passionate blockchain developer, open source advocate and a contributor to the TON ecosystem. He is also one of the main developers for TONcoin Fund (https://www.toncoin.fund). For Tal's work on TON, follow on GitHub (https://github.com/ton-defi-org). For Tal's personal work, follow on GitHub (https://github.com/talkol) and Twitter (https://twitter.com/koltal).
