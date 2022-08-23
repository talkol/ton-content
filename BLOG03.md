# How TON wallets work and how to access them from JavaScript

*by Tal Kol*

---

The TON blockchain is [based](https://ton-blockchain.github.io/docs/ton.pdf) on the TON coin (previously labeled TonCoin). This cryptocurrency is used to pay for executing transactions (gas), much like ETH on the Ethereum blockchain. If you're participating in the TON ecosystem, most likely that you're already holding some TON and probably already have a wallet.

In this blog post, we will create a new TON wallet using one of the native apps and then try to access it programatically from JavaScript. This can be useful for example if you're planning on writing a bot that receives and sends TON. We'll also understand how wallets work and dive into their internals.

## Creating a wallet using an app

The simplest way to create a TON wallet is visit https://ton.org/wallets and choose one of the wallet apps from the list. This page explains the difference between custodial and non-custodial wallets. With a non-custodial wallet, you own the wallet and hold its private key by yourself. With a custodial wallet, you trust somebody else to do this for you.

The point of blockchain is being in control of your own funds, so we'll naturally choose a non-custodial option. They're all pretty similar, let's choose [TonKeeper](https://tonkeeper.com). Go ahead and install the TonKeeper app on your phone, run it and choose "Set up wallet". Let's create a new wallet. After a few seconds, your wallet is created and TonKeeper displays your recovery phrase - the secret 24 words that give access to your wallet funds.

## The 24 word recovery phrase

The recovery phrase is the key to accessing your wallet. Lose this phrase and you'll lose access to your funds. Give this phrase to somebody and they'll be able to take your funds. Keep this secret and backed up in a safe place.

Why 24 words? The OG crypto wallets, like Bitcoin in its early days, did not use word phrases, they used a bunch of random looking letters to specify your key. This didn't work so well because of typos. People would make a mistake with a single letter and not be able to access their funds. The idea behind words was to eliminate these mistakes and make the key easier to write down. These phrases are also called "mnemonics" because they act as [mnemonic](https://en.wikipedia.org/wiki/Mnemonic) devices that make remembering them easier for humans.

For the sake of this post, I'm going to write the secret mnemonic here. Never do this! I only do this for educational purposes and I will make sure the wallet doesn't have any TON in it that can be stolen:

```
rail sound peasant garment bounce trigger true abuse arctic gravity ribbon ocean absurd okay blue remove neck cash reflect sleep hen portion gossip arrow
```

Using the mnemonic, it is possible to calculate the public and private keys of the wallet. A public-private key pair is a fundamental part of [public key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography) which allows the rightful owner alone (holder of the secret private key) to sign messages that correspond to the public key (that can be shared with everybody else).

## The wallet address

If you click on the top left in the TonKeeper app you will see your wallet address. This should be it:

```
EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ
```

This wallet address isn't secret. You can share it with anyone you want and they won't be able to touch your funds. If you want anyone to send you some TON, you will need to give them this address. You should be aware though of some privacy matters. Wallet addresses in TON and most blockchains are [pseudo-anonymous](https://en.wikipedia.org/wiki/Pseudonymization), this means that they don't reveal your identity in the real world. If you tell somebody your address and they know you in the real world, they can now make the connection.

The blockchain ledger is public, so anyone can look up the balance and transactions of every address. This means that anyone who knows you and your address will be able to see what you do with your funds. Not a big deal usually.

## Let's look in a block explorer

A block explorer is a tool that allows you to query data from the chain. We can query information using an address. There are many [explorers](https://ton.app/explorers) to choose from, I personally like https://tonwhales.com/explorer because it shows some useful information.

Let's go ahead and input the address and see [what it shows](https://tonwhales.com/explorer/address/EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ):


<img src="https://i.imgur.com/QCjpdZb.png" width=600 />

## Wallets in TON are smart contracts

The explorer shows a few interesting things. Our wallet is actually a contract! and it has not been initialized yet. Smart contracts is a concept invented by the Ethereum blockchain which allows developers to easily create their own decentralized apps that run on the blockchain. You can use a smart contract for example to implement your own token, which is similar in principle to TON or Bitcoin, in just a few lines of code.

It seems that our wallet is such a decentralized app. The idea that all wallets are smart contracts is one of the differences between TON and Ethereum. If you want to learn about more differences, check out [this post](https://society.ton.org/six-unique-aspects-of-ton-blockchain-that-will-surprise-solidity-developers). Smart contracts have code that is deployed on-chain and is executed by the [TVM](https://ton-blockchain.github.io/docs/tvm.pdf). The act of deployment means that the compiled contract code is uploaded to the blockchain by its creator. Has our wallet been deployed yet? If so, who paid the gas for the deployment fee?

You've probably guessed it. Our contract is labeled as "uninitialized" so it hasn't been actually deployed yet. Hmm.. if it's not deployed, how can we know its address?

## Addresses of TON smart contracts

Our wallet address is the smart contract address. Smart contracts in TON have addresses even before they are deployed. A smart contract address on TON is derived from two parameters:

* **The smart contract initial code** - When you compile a smart contract, you get bytecode (actually bitcode since it's not necessarily full bytes on TON) that the TVM can execute. The address is based in part on hash of this code.

* **The smart contract initial data** - When you deploy a smart contract, you need to define its initial data cell (persistent state storage). The address is based in part on hash of this data.

So how did TonKeeper wallet know the address of the smart contract before deploying it? Simple, TonKeeper knows the code it's going to deploy and the initial data cell it's going to deploy. So it can show you the address you will get even before deploying.

What happens if a smart contract is upgraded and its code changes? Does it change its address in this case? No. As you see above, the address only depends on the *initial* code (and data). If either of those change in the future, this will not change the contract address. Please also be aware that contract code can only be upgraded if the contract explicitly supports this functionality.

## Let's send some TON to our wallet

As you can see in the block explorer, the TON balance of our wallet is currently zero. We will need to fund our wallet by asking somebody to transfer some TON coins to our address. But wait... isn't this dangerous? How can we transfer some coins to the smart contract before it is deployed?!

It turns out that this isn't a problem on TON. The TON blockchain maintains a list of accounts by address and stores the TON coin balance per address. Since our smart contract has an address, it can have a balance, even before it has been deployed. Let's send 1 TON to our address and see what happens in the [block explorer](https://tonwhales.com/explorer/address/EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ):

<img src="https://i.imgur.com/GQDQ7F8.png" width=600 />

As you can see, the balance of the smart contract is now 1 TON. And the contract remains "uninitialized", meaning it still hasn't been deployed.

## Deploying your wallet smart contract

So when is your wallet smart contract being deployed? This would normally happen when you execute your first transaction - normally an outgoing transfer. This transaction is going to cost gas, so your balance cannot be zero to make it. TonKeeper is going to deploy our smart contract automatically when we issue the first transfer. Let's send 0.5 TON somewhere through TonKeeper and refresh the [block explorer](https://tonwhales.com/explorer/address/EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ):

<img src="https://i.imgur.com/6yff093.png" width=600 />

We can see that TonKeeper indeed deployed our contract! It is no longer uninitialized and shows [Wallet V4](https://tonwhales.com/explorer/address/EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ/code) instead. We can see that we've also paid some gas for the deployment and transfer fees. After sending 0.5 TON we have 0.4865 TON remaining, meaning we paid a total of 0.0135 TON in fees, not too bad.

Another interesting thing is that we didn't lose access to the 1 TON previously sent. In order to gain access to these funds, the contract had to be deployed first. We can see that TonKeeper has estimated its address correctly.

We can also see that TonKeeper deployed our smart contract to the "Basic Workchain". TON supports more than one chains. The basic workchain (chain index 0) is used for most day to day user activity. The other important chain is the Masterchain (chain index -1) which is used mostly for validator activity. You will often see validator wallets deployed to the masterchain because the TON election contracts are found there.

## Multiple versions of wallet smart contracts

[Wallet V4](https://tonwhales.com/explorer/address/EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ/code) refers to the version of our smart contract code. If our wallet smart contract was deployed with "V4" as its code, this means somewhere must exist "V1", "V2" and "V3". This is indeed correct. Over time, the TON core team has published multiple versions of the wallet contract code. We can see that TonKeeper has chosen to rely on V4.

Let's look at this well known wallet address of [TON Foundation](https://tonwhales.com/explorer/address/EQCD39VS5jcptHL8vMjEXrzGaRcCVYto7HUn4bpAOg8xqB2N). As you can see, it uses [Wallet V3 (r2)](https://tonwhales.com/explorer/address/EQCD39VS5jcptHL8vMjEXrzGaRcCVYto7HUn4bpAOg8xqB2N/code) for its code. It was probably deployed before V4 was released:

<img src="https://i.imgur.com/OZUdPd2.png" width=600 />

If you want to see the FunC source code of the various official wallet smart contracts, you can take a look [in the official TON repo](https://github.com/ton-blockchain/ton/tree/master/crypto/smartcont). Look for the file [wallet3-code.fc](https://github.com/ton-blockchain/ton/blob/master/crypto/smartcont/wallet3-code.fc) for example to see how V3 was implemented.

## Different wallet addresses for different secret mnemonics

We said earlier that a smart contract address is derived from the combination of the code that was initially deployed and the initial data cell. What is the data cell for wallet V3 that we've examined earlier? This is the relevant [source code line](https://github.com/ton-blockchain/ton/blob/master/crypto/smartcont/wallet3-code.fc#L13):

```
var (stored_seqno, stored_subwallet, public_key) = (ds~load_uint(32), ds~load_uint(32), ds~load_uint(256));
```

This is not very important to remember, but still a little interesting to investigate. The first 32 bit hold the sequence number (seqno), this is similar to the nonce on Ethereum and holds the sequence number of the last transaction executed (used to prevent [replay attacks](https://en.wikipedia.org/wiki/Replay_attack)). The next 32 bit hold the subwallet ID, again not very interesting, most standard wallets will always use the hardcoded value `698983191` here. The last 256 bit hold the public key - this is the interesting bit.

When a wallet is deployed, during its data cell initialization, the deployer sets the public key of the user controlling this wallet. The wallet smart contract will only respect transactions that are signed by the private key corresponding to this public key. Since every user has their own unique 24 word mnemonic, which corresponds to their own unique public-private key pair, every user will receive a unique wallet address.

## Multiple wallet addresses for the same secret mnemonic

Let's look at this now from a different angle. We said that a smart contract address is derived from the combination of the initial code and the initial data. What if the same user with the same secret mnemonic separately deploys both the wallet V3 code and the wallet V4 code? This could happen for example if this user tries two native wallet apps, that each rely on a different version of the code.

You guessed it, this user will have two different wallets deployed, each with its own unique address. This means that your secret mnemonic can correspond to multiple wallet addresses! The next time you try to access your wallet using your secret mnemonic and you see a different address than you expect and a balance of zero, don't be alarmed. Nobody stole your money, you are probably just looking at the wrong wallet version.

Another interesting bit to keep in mind is that TON addresses may have multiple representations - meaning the same address can be represented in multiple ways. Let's take our address `EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ` for example and input it to [ton.org/address](https://ton.org/address/). You can see that this same address has a representation that is bounceable or not, the topic of bouncing messages is out of scope but briefly explained [here](https://ton.org/docs/#/howto/smart-contract-guidelines?id=using-non-bounceable-messages).

## Let's run some JavaScript

Time to access our wallet programatically through code. I'm going to use JavaScript, it's the easiet language in my eyes for these purposes. If you prefer a different language like Python or Golang, search around, you'll easily find libraries in these languages as well.

There are two popular JavaScript libraries in the TON ecosystem that you can use. They do pretty much the same things but they're written by different people so their API vary a litte:

1. **TonWeb** - https://github.com/toncenter/tonweb - install via `npm install tonweb`

2. **npm ton** - https://github.com/tonwhales/ton - install via `npm install ton`

You only need to choose one, but I'm going to show you how to use both. You can choose the one that you like better, it doesn't really matter. We're also going to rely on another library for converting our 24 word mnemonic to a public-private key pair.

## 1a. Reading the wallet balance using TonWeb

Make sure you have [Node.js](https://nodejs.org/) installed on your machine, you can verify node is working by running in terminal `node -v` (my version is v17.3.0 but most recent versions will work). Create a new directory somewhere and run the following:

```
npm install tonweb
npm install tonweb-mnemonic
```

Now create a new file `balance.js` with this content:

```js
const tonMnemonic = require("tonweb-mnemonic");
const TonWeb = require("tonweb");

async function main() {
  // mnemonic to key pair
  const mnemonic = "rail sound peasant garment bounce trigger true abuse arctic gravity ribbon ocean absurd okay blue remove neck cash reflect sleep hen portion gossip arrow";
  const mnemonicArray = mnemonic.split(" ");
  const keyPair = await tonMnemonic.mnemonicToKeyPair(mnemonicArray);
  console.log("public key:", Buffer.from(keyPair.publicKey).toString('hex'));
  
  // list available wallet versions
  const tonweb = new TonWeb(new TonWeb.HttpProvider("https://toncenter.com/api/v2/jsonRPC"));
  console.log("wallet versions:", Object.keys(tonweb.wallet.all).toString());
  
  // instance of wallet V4 r2 (from the list printed above)
  const WalletClass = tonweb.wallet.all["v4R2"];
  const wallet = new WalletClass(tonweb.provider, { publicKey: keyPair.publicKey });
  const address = await wallet.getAddress();
  console.log("address:", address.toString(true, true, true));
  const seqno = await wallet.methods.seqno().call();
  console.log("seqno:", seqno);
  await sleep(1500); // avoid throttling by toncenter.com
  const balance = await tonweb.getBalance(address);
  console.log("balance:", TonWeb.utils.fromNano(balance));
}

main();

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

Execute the script by running in terminal:

```
node balance.js
```

The result should be something like:

```
public key: 49f50bb94c5fb463534a9d0df0d8e39bcb93109589daf65197e9151c3777402f
wallet versions: simpleR1,simpleR2,simpleR3,v2R1,v2R2,v3R1,v3R2,v4R1,v4R2
address: EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ
seqno: 1
balance: 0.486493996
```

As you can see, we needed to make sure we instantiate the correct wallet code version (`v4R2` in this case) or else we would reach a different address. This part may take some trial and error. If you don't know your wallet version and you can't check in the block explorer, you can simply iterate over all types and check which address holds a nonzero TON balance.

Another peculiar thing you may have noticed is that I included some `sleep` commands before accessing toncenter.com [RPC provider](https://toncenter.com/api/v2/). An RPC provider is an HTTP service that allows clients to make queries to the blockchain. It is possible to run this provider yourself on your own server, but in most cases you will find a third-party like toncenter.com to do this for you. Without an API key (that you can register for free via [@tonapibot](https://t.me/tonapibot)) toncenter.com throttles requests to 1 per second, so these `sleep` command limit the request rate. If you have an API key, you can remove the delays.

## 1b. Sending a transfer transaction using TonWeb

The previous action was read-only and should generally be possible even if you don't have the private key of the wallet. Now, we're going to transfer some TON from the wallet. Since this is a priviliged action, the private key is required.

Create a new file `transfer.js` with this content:

```js
const tonMnemonic = require("tonweb-mnemonic");
const TonWeb = require("tonweb");

async function main() {
  // mnemonic to key pair
  const mnemonic = "rail sound peasant garment bounce trigger true abuse arctic gravity ribbon ocean absurd okay blue remove neck cash reflect sleep hen portion gossip arrow";
  const mnemonicArray = mnemonic.split(" ");
  const keyPair = await tonMnemonic.mnemonicToKeyPair(mnemonicArray);
  console.log("secret key:", Buffer.from(keyPair.secretKey).toString('hex'));
  
  // list available wallet versions
  const tonweb = new TonWeb(new TonWeb.HttpProvider("https://toncenter.com/api/v2/jsonRPC"));
  console.log("wallet versions:", Object.keys(tonweb.wallet.all).toString());
  
  // instance of wallet V4 r2 (from the list printed above)
  const WalletClass = tonweb.wallet.all["v4R2"];
  const wallet = new WalletClass(tonweb.provider, { publicKey: keyPair.publicKey });
  const seqno = await wallet.methods.seqno().call() || 0;
  console.log("seqno:", seqno);
  await sleep(1500); // avoid throttling by toncenter.com
  await wallet.methods.transfer({
    secretKey: keyPair.secretKey,
    toAddress: "EQDrjaLahLkMB-hMCmkzOyBuHJ139ZUYmPHu6RRBKnbdLIYI",
    amount: TonWeb.utils.toNano("0.02"), // 0.02 TON
    seqno: seqno,
    payload: "Hello", // optional comment
    sendMode: 3,
  }).send();
  
  // wait until confirmed
  let currentSeqno = seqno;
  while (currentSeqno == seqno) {
    console.log("waiting for transaction to confirm...");
    await sleep(1500); // avoid throttling by toncenter.com
    currentSeqno = await wallet.methods.seqno().call() || 0;
  }
  const address = await wallet.getAddress();
  await sleep(1500); // avoid throttling by toncenter.com
  const balance = await tonweb.getBalance(address);
  console.log("balance:", TonWeb.utils.fromNano(balance));
}

main();

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

Execute the script by running in terminal:

```
node transfer.js
```

The result should be something like:

```
secret key: 95de3fdf1748f0e0e9b19b1065e5b6ac1783362bc38bd9c26413fc69441e605f49f50...
wallet versions: simpleR1,simpleR2,simpleR3,v2R1,v2R2,v3R1,v3R2,v4R1,v4R2
seqno: 1
waiting for transaction to confirm...
waiting for transaction to confirm...
waiting for transaction to confirm...
waiting for transaction to confirm...
waiting for transaction to confirm...
waiting for transaction to confirm...
waiting for transaction to confirm...
balance: 0.454690417
```

## 2a. Reading the wallet balance using npm ton

We're going to do the same exact task, but this time with a different set of libraries. You are free to choose which library you like better. 

Create a new directory somewhere and run the following:

```
npm install ton
npm install ton-crypto
npm install axios-request-throttle
```

The last one is required because adding `sleep` commands to deal with toncenter.com throttling without an API key will not work in this case. The library `ton` makes multiple consecutive requests under some of its API, so throttling requests must be done in a lower level using the `axios-request-throttle` library. If you have an API key, you can remove this part.

Now create a new file `balance.js` with this content:

```js
// this part is required since toncenter jsonRPC limits to 1 req/sec without an API key
const axios = require("ton/node_modules/axios");
const axiosThrottle = require("axios-request-throttle");
axiosThrottle.use(axios, { requestsPerSecond: 0.5 });

const { mnemonicToPrivateKey } = require("ton-crypto");
const { TonClient, Wallet, AllWalletContractTypes, fromNano } = require("ton");

async function main() {
  // mnemonic to key pair
  const mnemonic = "rail sound peasant garment bounce trigger true abuse arctic gravity ribbon ocean absurd okay blue remove neck cash reflect sleep hen portion gossip arrow";
  const mnemonicArray = mnemonic.split(" ");
  const keyPair = await mnemonicToPrivateKey(mnemonicArray);
  console.log("public key:", Buffer.from(keyPair.publicKey).toString('hex'));
  
  // list available wallet versions
  const client = new TonClient({ endpoint: "https://toncenter.com/api/v2/jsonRPC" });
  console.log("wallet versions:", AllWalletContractTypes.toString());

  // instance of wallet V4 (from the list printed above)
  const wallet = Wallet.openByType(client, 0, keyPair.secretKey, "org.ton.wallets.v4");
  const address = wallet.address;
  console.log("address:", address.toFriendly(true, true, true));
  const seqno = await wallet.getSeqNo();
  console.log("seqno:", seqno);
  const balance = await client.getBalance(address);
  console.log("balance:", fromNano(balance));
}

main();
```

Execute the script by running in terminal:

```
node balance.js
```

The result should be something like:

```
public key: 49f50bb94c5fb463534a9d0df0d8e39bcb93109589daf65197e9151c3777402f
wallet versions: org.ton.wallets.simple.r2,org.ton.wallets.simple.r3,org.ton.wallets.v2,org.ton.wallets.v2.r2,org.ton.wallets.v4,org.ton.wallets.v3.r2,org.ton.wallets.v3
address: EQAC824gsw8OZLoMV6_nr4nkxaEQFlbzoiHHOWIYY81eM5rQ
seqno: 3
balance: 0.434690417
```

## 2b. Sending a transfer transaction using npm ton

The previous action was read-only and should generally be possible even if you don't have the private key of the wallet. Now, we're going to transfer some TON from the wallet. Since this is a priviliged action, the private key is required.

Create a new file `transfer.js` with this content:

```js
// this part is required since toncenter jsonRPC limits to 1 req/sec without an API key
const axios = require("ton/node_modules/axios");
const axiosThrottle = require("axios-request-throttle");
axiosThrottle.use(axios, { requestsPerSecond: 0.5 });

const { mnemonicToPrivateKey } = require("ton-crypto");
const { TonClient, Wallet, AllWalletContractTypes, toNano, fromNano, Address } = require("ton");

async function main() {
  // mnemonic to key pair
  const mnemonic = "rail sound peasant garment bounce trigger true abuse arctic gravity ribbon ocean absurd okay blue remove neck cash reflect sleep hen portion gossip arrow";
  const mnemonicArray = mnemonic.split(" ");
  const keyPair = await mnemonicToPrivateKey(mnemonicArray);
  console.log("secret key:", Buffer.from(keyPair.secretKey).toString('hex'));
  
  // list available wallet versions
  const client = new TonClient({ endpoint: "https://toncenter.com/api/v2/jsonRPC" });
  console.log("wallet versions:", AllWalletContractTypes.toString());

  // instance of wallet V4 (from the list printed above)
  const wallet = Wallet.openByType(client, 0, keyPair.secretKey, "org.ton.wallets.v4");
  const seqno = await wallet.getSeqNo();
  console.log("seqno:", seqno);
  await wallet.transfer({
    secretKey: keyPair.secretKey,
    to: Address.parseFriendly("EQDrjaLahLkMB-hMCmkzOyBuHJ139ZUYmPHu6RRBKnbdLIYI").address,
    value: toNano(0.02), // 0.02 TON
    seqno: seqno,
    bounce: false,
    payload: "Hello", // optional message
  });
  
  // wait until confirmed
  let currentSeqno = seqno;
  while (currentSeqno == seqno) {
    console.log("waiting for transaction to confirm...");
    await sleep(1500);
    currentSeqno = await wallet.getSeqNo();
  }
  const address = wallet.address;
  const balance = await client.getBalance(address);
  console.log("balance:", fromNano(balance));
}

main();

function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
```

Execute the script by running in terminal:

```
node transfer.js
```

The result should be something like:

```
secret key: 95de3fdf1748f0e0e9b19b1065e5b6ac1783362bc38bd9c26413fc69441e605f49f50...
wallet versions: org.ton.wallets.simple.r2,org.ton.wallets.simple.r3,org.ton.wallets.v2,org.ton.wallets.v2.r2,org.ton.wallets.v4,org.ton.wallets.v3.r2,org.ton.wallets.v3
seqno: 3
waiting for transaction to confirm...
waiting for transaction to confirm...
waiting for transaction to confirm...
waiting for transaction to confirm...
waiting for transaction to confirm...
balance: 0.402895251
```

Happy coding!

---

Tal is a founder of Orbs Network (https://orbs.com). He's a passionate blockchain developer, open source advocate and a contributor to the TON ecosystem. He is also one of the main developers for TONcoin Fund (https://www.toncoin.fund). For Tal's work on TON, follow on GitHub (https://github.com/ton-defi-org). For Tal's personal work, follow on GitHub (https://github.com/talkol) and Twitter (https://twitter.com/koltal).
