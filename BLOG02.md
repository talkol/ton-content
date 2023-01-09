# TON Hello World: Step by step guide for writing your first smart contract in FunC (part 1)

*by Tal Kol*

---

A smart contract is simply a computer program running on TON blockchain - or more exactly its [TVM](https://ton-blockchain.github.io/docs/tvm.pdf) (TON Virtual Machine). The contract is made of code (compiled TVM instructions) and data (persistent state) that are stored in some address on the TON blockchain.

In the world of blockchain, *code is law*, meaning that instead of lawyers and papers, computer instructions define in absolute terms the rules of interaction between the different users of the contract. Before enagaging with any smart contract as a user, you're expected to review its code and thus understand its terms of agreement. Accordingly, we'll make an effort to make our contract as easy to read as possible, so its users could understand what they're getting into.

## Dapps - decentralized applications

Smart contracts are a key part of *decentralized apps* - a special type of application invented in the blockchain era, that does not depend on any single entity to run it. Unlike the app Uber, for example, which depends on the company Uber Inc to run it - a *decentralized Uber* would allow riders and drivers to interact (order, pay for and fulfill rides) without any intermediary. Dapps are also unstoppable - if we don't depend on anyone specific to run them, nobody can take them down.

Dapps on TON blockchain are usually made of 3 main projects:
* Smart contracts in the [FunC](https://ton.org/docs/develop/func/overview) programming language that are deployed on-chain - these act as the "backend server" of the app, with a "database" for persistent storage
* Web frontend for interacting with the dapp from a web browser - this acts as the "frontend" or "client"
* Telegram bot for interacting with the dapp from inside Telegram messenger - as TON is the blockchain of choice of the popular Telegram messenger, in-app interaction through chat is almost mandatory

Throughout this series of blog posts, we will build a full dapp together and see detailed implementations of all 3 projects.

## Step 1: Defining our first smart contract

So what are we going to build? Our smart contract will be quite simple:

Its first feature is to hold a *counter*. The counter will start at some number, and allow users to send *increment* transactions to the contract, which will in turn increase the counter value by 1. The contract will also have a getter function that will allow any user to query the current value of the counter.

Its second feature is to hold some money (TON coins). Any user would be able to send some TON in a *deposit* transaction. This TON will be kept in the contract's balance. Only a special user, named the contract owner, will be allowed to send a *withdraw* transaction and specify the amount of TON they want to take out.

Naturally, because we set a special owner, we want to be have the option to change it. Only the current owner will be allowed to send a special *transfer ownership* transaction, that will in turn pass the ownership to someone else. This will be feature number three.

## Step 2: Setting up your local machine

Before we can start writing code, we need to install certain developer tools on our computer. This part can be a bit annoying to setup, so let's do it together right now.

For convenience, our development environment will rely on several clever scripts for testing, compiling and deploying our code. The most convenient language for these scripts is JavaScript, executed by an engine called Nodejs. The installation instructions are [here](https://nodejs.org/). We will need a fairly recent version of node like `v16` or `v17`. You can verify your nodejs version by running `node -v` in terminal.

Last but not least, you will need a decent IDE with FunC and TypeScript support. I recommend [Visual Studio Code](https://code.visualstudio.com/) - it's free and open source. Also install the [FunC Plugin](https://marketplace.visualstudio.com/items?itemName=tonwhales.func-vscode) to add syntax highlighting for the FunC language.

## Step 3: Setting up our project

Let's create a folder for our project and initialize it. From the command line, in your folder run the following:
1. `npm init es6 --yes`, which will initialize a project ready to work with npm packages for us.
1. `npm install func-js`, which installs [func-js](https://github.com/ton-community/func-js),  Wasm cross-platform compiler that can be invoked either programmatically or via a CLI.

> Previously, we would have to deal with compiling or installing pre-compiled binaries of func and fift,
> but func-js, which uses Wasm, relieves us from this need.

Now our environment
## Step 4: Structuring our smart contract

Much like everything else in life, smart contracts in FunC are divided into 3 sections. These sections are: *storage*, *messages* and *getters*.

The **storage** section deals with our contract's persistent data. Our contract will have to store data between calls from different users, for example the value of our *counter* variable. To write this data to state storage, we will need a write/encode function and to read this data back from state storage, we will need a read/decode function.

The **messages** section deals with messages sent to our contract. The main form of interaction with contracts on TON blockchain is by sending them messages. We mentioned before that our contract will need to support a variety of actions like *increment*, *deposit*, *withdraw* and *transfer ownership*. All of these operations are performed by users as transactions. These operations are not read-only because they change something in the contract's persistent state.

The **getters** section deals with read-only interactions that don't change state. For example, we would want to allow users to query the value of our *counter*, so we can implement a getter for that. We've also mentioned that the contract has a special *owner*, so what about a getter to query that. Since our contract can hold money (TON coins), another useful getter could be to query the current balance.

## Step 5: Counter contract in FunC

We're about to write our first lines in FunC! Our first task would be to implement the *counter* feature of our contract.

The FunC programming language is very similar to the [C language](https://en.wikipedia.org/wiki/C_(programming_language)). It has strict types, which is a good idea, since compilation errors will help us spot contract mistakes early on. The language was designed specifically for TON blockchain, so you will not find a lot of documentation beyond the [official FunC docs](https://ton.org/docs/develop/func/overview).

### Storage

Let's start with the first section, *storage*, and implement two utility functions (which we will use later) for reading and writing variables to the contract's persistent state - `load_data()` and `save_data()`. The primary variable will be the counter value. We must persist this value to storage because we need to remember it between calls. The appropriate type for our counter variable is `int`. Notice [in the docs](https://ton.org/docs/develop/func/types#atomic-types) that the `int` TVM runtime type is always 257 bit long (256 bit signed) so it can hold huge huge numbers - I'm pretty sure the universe has less than 2^256 atoms in it, so you'll never have a number so large that you can't fit in it. Storing the full 257 bits in blockchain storage is somewhat wasteful because the contract pays rent proportionally to the total amount of data it keeps. To optimize costs, let's keep in persistent storage just the lowest 64 bits - capping our counter's maximum value at 2^64 which should be enough:

```
(int) load_data() inline {                 ;; read function declaration - returns int as result
  var ds = get_data().begin_parse();       ;; load the storage cell and start parsing as a slice
  return (ds~load_uint(64));               ;; read a 64 bit unsigned int from the slice and return it
}

() save_data(int counter) impure inline {  ;; write function declaration - takes an int as arg
  set_data(begin_cell()                    ;; store the storage cell and create it with a builder 
    .store_uint(counter, 64)               ;; write a 64 bit unsigned int to the builder
    .end_cell());                          ;; convert the builder to a cell
}
```

The standard library functions `get_data()` and `set_data()` are documented [here](https://ton.org/docs/develop/func/stdlib#persistent-storage-save-and-load) and load/store the storage cell. We will cover [*cells*](https://ton.org/docs/develop/func/types#atomic-types) in detail in future posts of this series. Cells are read from using the [*slice*](https://ton.org/docs/develop/func/types#atomic-types) type (an array of bits) and written to using the [*builder*](https://ton.org/docs/develop/func/types#atomic-types) type. The various methods that you see are all taken from the [standard library](https://ton.org/docs/develop/func/stdlib). Also notice two interesting function modifiers that appear in the declarations - [*inline*](https://ton.org/docs/develop/func/functions#inline-specifier) and [*impure*](https://ton.org/docs/develop/func/functions#impure-specifier).

### Messages

Let's continue to the next section, *messages*, and implement the main message handler of our contract - `recv_internal()`. This is the primary entry point of our contract. It runs whenever a message is sent as a transaction to the contract by another contract or by a user's wallet contract:

```
() recv_internal(int msg_value, cell in_msg, slice in_msg_body) impure {  ;; well known function signature
  int op = in_msg_body~load_uint(32);                                     ;; parse the operation type encoded in the beginning of msg body
  var (counter) = load_data();                                            ;; call our read utility function to load values from storage
  if (op == 1) {                                                          ;; handle op #1 = increment
    save_data(counter + 1);                                               ;; call our write utility function to persist values to storage
  }
}
```

As you can see, the first thing to do when parsing a new incoming message is to read its operation type. By convention, [internal messages](https://ton.org/docs/develop/smart-contracts/guidelines/internal-messages) are encoded with a 32 bit unsigned int in the beginning that acts as operation type (op for short). We are free to assign any serial numbers we want to our different ops. In this case, we've assigned the number `1` to the *increment* action, which is handled by writing back to persistent state the current value counter plus 1.

### Getters

Our last section, as you recall, is *getters*. Let's implement a simple getter that will allow users to query the counter value:

```
int counter() method_id {        ;; getter declaration - returns int as result
  var (counter) = load_data();   ;; call our read utility function to load value
  return counter;
}
```

We can choose what input arguments the getter takes as input and what output it returns as result. Also notice the function modifier appearing in the declaration - [*method_id*](https://ton.org/docs/develop/func/functions#method_id). It is customary to place `method_id` on all getters.

That's it. We completed our 3 sections and the first version of our contract is ready. To get the complete code, simply concat the 3 snippets above to a single file named `counter.fc` and save it. This will be the FunC (`.fc` file extension) source file of our contract. The resulting source file should look something like [this](https://gist.github.com/talkol/fc354dd9560ecca9a6b112e5e39cd9ed).

## Step 6: Building the counter contract

Right now, the contract is just FunC source code. To get it to run on-chain, we need to convert it to TVM [bytecode](https://ton.org/docs/learn/tvm-instructions/instructions). 

> In TON, we don't compile FunC directly to bytecode, but instead go through another programming language called [Fift](https://ton-blockchain.github.io/docs/fiftbase.pdf). Just like FunC, Fift is another language that was designed specifically for TON blockchain. It's a low level language that is very close to TVM opcodes. For us regular mortals, Fift is not very useful, so unless you're planning on some extra advanced things, I believe you can safely ignore it for now.

The `func-js` wraps all the functionality needed in order to compile our contract to bytecode.
Let's try to run it: 

```
npx func-js counter.fc --boc counter.cell
```

You'll notice that we immediately get a bunch of compilation errors on some function definitions missing like `set_data` and `begin_cell`. It's good practice to see what compilation errors look like. Indeed, our code relies on these standard library functions, but where are they defined? The foundation publishes these in the main TON repo in the file [stdlib.fc](https://github.com/newton-blockchain/ton/blob/master/crypto/smartcont/stdlib.fc). Since I've seen multiple versions of this file running around, it's good practice to download it and include it as part of your project.

Let's go and save it to our folder:

```
curl https://raw.githubusercontent.com/newton-blockchain/ton/master/crypto/smartcont/stdlib.fc > stdlib.fc
``` 

The `func-js` compiler supports taking multiple files as arguments. Note that order matters in this case, so `stdlib.fc` needs to be processed before `counter.fc`, which relies on it:

```
npx func-js stdlib.fc counter.fc --boc counter.cell
```

The build should now succeed, with the output of this command being a new file - `counter.cell`. This is a binary file that finally contains the TVM bytecode in cell format that is ready to be deployed on-chain. This will actually be the only file we need for deployment moving forward (we won't need the FunC source file).

Don't worry if the build process feels a little complex and cumbersome, at the end of the post I'll give you a wonderful build script that does all of the above automatically.

## Step 7: Deploying our contract on-chain

Now that our contract has been compiled to bytecode, we can finally see it in action running on-chain. The act of uploading the bytecode to the blockchain is called *deployment*. The deployment result would be an address where the contract resides. This address will allow us to communicate with this specific contract instance later on and send it transactions.

There are two variations of the TON blockchain we can deploy to - *mainnet* and *testnet*. Mainnet is the real thing, where we would have to pay real TON coin in order to deploy and staked validators would execute our contract and guarantee a very high level of security - our contract would be able to do dangerous things like holding large amounts of money without worrying too much. Testnet is a testing playground where the TON coin isn't real and is available for free. Naturally, testnet doesn't offer any real security so we would just use it to practice and see that our code is behaving as expected.

Personally, I almost never deploy to testnet. There are far better ways to gain confidence that my code is working as expected. The primary of which is writing a dedicated *test suite*. I will cover this in detail in the next post of this series. For now, let's assume the code is working perfectly and no further debugging is required.

If we're confident in our code, there's no reason not to deploy to mainnet. The process isn't very expensive and will cost you much less than 0.5 TON. If you're unable to get any real TON coin, do the next part on testnet and request some free testnet TON coin by contacting https://t.me/testgiver_ton_bot.

### Init arguments

The new address of our deployed contract in TON depends on only two things - the deployed bytecode and the initial contract storage. You can say that the address is some derivation of the hash of both. If two different developers were to deploy the exact same code with the exact same initialization data, they would collide.

The bytecode part is easy, we have that ready as a cell in the file `counter.cell` that we compiled in step 6. Now what about the initial contract storage? As you recall, the format of our persistent storage data was decided when we implemented the function `save_data()` of our contract FunC source. Our storage layout was very simple - just one unsigned int of 64 bit holding the counter value. Therefore, to initialize our contract, we would need to generate a data cell holding some arbitrary initial uint64 value - let's say - the number `17`.

I found it easiest to perform the deployment in JavaScript (TypeScript actually). Let's create this data cell in code and use a very convenient TypeScript library called [ton](https://www.npmjs.com/package/ton) for the task.

Let's install it:
```
npm install ton
```

```ts
import { beginCell } from "ton-core";

function initData() {
  const initialCounterValue = 17;
  return beginCell().storeUint(initialCounterValue, 64).endCell();
}
```

Notice how the API of this TypeScript library mimics the standard library API in FunC.

Now, that our two cells (init code and init data) are ready, we can calculate the future address of our deployed smart contract. Let's use the same TypeScript library:

```ts
import fs from "fs";
import { contractAddress, Cell } from "ton-core";

const initDataCell = initData(); // the function we've implemented just now
const initCodeCell = Cell.fromBoc(fs.readFileSync("counter.cell"))[0]; // compilation output from step 6

const newContractAddress = contractAddress(0, {code: initCodeCell, data: initDataCell});
```

### Deployment script

The actual deployment involves sending a special message that will deploy our contract. The deployment is going to cost gas and should be done through a wallet. I'm assuming that you have some familiarity with TON wallets and how they're derived from 24 word secret mnemonics. If not, read [this](https://ton.org/docs/develop/dapps/asset-processing/). Let's assume that you already have a TON wallet (holding at least 0.5 TON coin for gas) and that its secret mnemonic is `mad nation chief flavor ...`

```ts
import { mnemonicToWalletKey } from "ton-crypto";
import { WalletContractV3R2 } from "ton";

const mnemonic = "mad nation chief flavor ..."; // your 24 secret words
const key = await mnemonicToWalletKey(mnemonic.split(" "));

const wallet = WalletContractV3R2.create({
  publicKey: key.publicKey,
  workchain: 0,
});
```

To deploy, we will need RPC access to the TON blockchain in order to be able to interact with it over HTTP. 
TON Access is a library that will provide us with access to unthrottled API endpoints for achieving this. 
Let's install it:

```
npm install @orbs-network/ton-access
```

Now that we also have our wallet ready, we can finally send the deploy message:

```ts
import { TonClient } from "ton";
import { internal, SendMode } from "ton-core";
import { getHttpEndpoint } from "@orbs-network/ton-access"

const endpoint = await getHttpEndpoint({
  network: "mainnet" // or "testnet", according to your choice
});

const client = new TonClient({ endpoint });

async function deploy() {
  const contract = client.open(wallet);
  const seqno = await contract.getSeqno(); // get the next seqno of our wallet

  const transfer = contract.createTransfer({
    seqno,
    messages: [
      internal({
        to: newContractAddress.toString(),
        value: '0.02',
        init: { data: initDataCell, code: initCodeCell },
        bounce: false,
      }),
    ],
    secretKey: key.secretKey,
    sendMode: SendMode.PAY_GAS_SEPARATLY + SendMode.IGNORE_ERRORS,
  });

  await client.sendExternalMessage(wallet, transfer);
}

```

I'm aware that the deploy script seems partial and you probably haven't been running it yourself. Don't worry about it, since there's no reason you should be writing it yourself from scratch. I wrote a general purpose deploy script that you can use and I'll share it at the end of this post. My goal in this step was mostly to explain the process.

## Step 7: Interacting with the deployed contract

After the contract is deployed, it's always nice to examine it on a block explorer. Take the value of the variable `newContractAddress` from the TypeScript script above and provide it to https://tonscan.org (or the [testnet](https://testnet.tonscan.org) version). Review the displayed page. If *Contract Type* is "unknown contract" and *State* is "active", your contract was deployed successfully.

### Call a getter

The first interaction we'll do is call a getter. In our case we just have one getter - the `counter()` method. Calling a getter is free and does not cost gas. The reason is that this call is read-only, so it does not require consensus by the validators and is not stored in a block on-chain for all eternity. The natural place to make this call is from TypeScript:

```ts
async function callGetter() {
  const call = await client.callGetMethod(newContractAddress, "counter"); // newContractAddress from deploy
  console.log(`Counter value is ${call.stack.readBigNumber().toString()}`);
}
```

### Send a message

If getters are read-only, to perform a write and change contract state in storage, we must send a message. In our case we handled messages in `recv_internal()` and assigned op #1 = *increment*. The messages can be sent from any TON wallet, not necessarily the deployer wallet. Sending messages costs gas and requires payment in TON coin. The reason is that this call is not read-only, so it requires waiting for consensus by the validators and is stored as a transaction in a block on-chain for all eternity. Let's see the code in TypeScript:

```ts
async function sendMessage() {
  const messageBody = beginCell().storeUint(1, 32).storeUint(0, 64).endCell(); // op with value 1 (increment)
  
  const contract = client.open(wallet);
  const seqno = await contract.getSeqno(); // get the next seqno of our wallet
  
  const transfer = contract.createTransfer({
    seqno,
    messages: [
      internal({
        to: newContractAddress.toString(),
        value: '0.02',
        bounce: false,
        body: messageBody
      }),
    ],
    secretKey: key.secretKey,
    sendMode: SendMode.PAY_GAS_SEPARATLY + SendMode.IGNORE_ERRORS,
  });

  await client.sendExternalMessage(wallet, transfer);
}
```

Notice that the message will take a few seconds to be processed by validators and will only change contract state after it has been processed. The normal wait time is a block or two, since validators need to produce a new block that contains our sent transaction. The op that was sent above is #1 = *increment*, which means that after processing, the counter value will increase by 1.

## Summary

This is the end of part 1 of this blog series. We created a simple new FunC contract with the counter functionality. We then built and deployed it on-chain and finally interacted with it by calling a getter and sending a message.

Part 2 of the series will continue with the same FunC contract, expand its functionality to include the remaining features and focus on methodologies for *contract testing*. Testing is how we gain confidence that our contract is working as expected before deploying it. Testing is particularly important with smart contracts since they're often used to manage money, so any bugs you miss will literally cost you (or your users).

As promised earlier, I want to share a Github repo that contains all the boilerplate discussed above as a template for your new FunC projects. This template includes both a general purpose build script and a general purpose deploy script, allowing you to focus your efforts just on writing your FunC code, and let the environment take care of the rest:

https://github.com/ton-defi-org/tonstarter-contracts

The template already contains a test suite that will be discussed in detail in part 2. For now, please be patient and ignore it.

Happy coding!

---

Tal is a founder of Orbs Network (https://orbs.com). He's a passionate blockchain developer, open source advocate and a contributor to the TON ecosystem. He is also one of the main developers for TONcoin Fund (https://www.toncoin.fund). For Tal's work on TON, follow on GitHub (https://github.com/ton-defi-org). For Tal's personal work, follow on GitHub (https://github.com/talkol) and Twitter (https://twitter.com/koltal).
