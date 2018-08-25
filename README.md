# Lightning Charge

[![build status](https://api.travis-ci.org/ElementsProject/lightning-charge.svg)](https://travis-ci.org/ElementsProject/lightning-charge)
[![npm release](https://img.shields.io/npm/v/lightning-charge.svg)](https://www.npmjs.com/package/lightning-charge)
[![MIT license](https://img.shields.io/github/license/elementsproject/lightning-charge.svg)](https://github.com/ElementsProject/lightning-charge/blob/master/LICENSE)
[![Pull Requests Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)
[![IRC](https://img.shields.io/badge/chat-on%20freenode-brightgreen.svg)](https://webchat.freenode.net/?channels=lightning-charge)

A drop-in solution for calling the [suredbits api](https://suredbits.com), built on top of [c-lightning](https://github.com/ElementsProject/lightning).

:zap: radically low fees :zap: nano payments :zap: instant confirmations :zap:

# Getting dependencies

You need to install [Node.js](https://nodejs.org/en/) (v7.6 or newer) and [npm](https://www.npmjs.com/get-npm).

### Ubuntu 18.04

```bash
$ sudo apt update
$ sudo apt install nodejs npm
```

### Ubuntu 16.04

```bash
$ curl -sL https://deb.nodesource.com/setup_8.x -o - | sudo bash
$ sudo apt install nodejs
```

### macOS

This guide assumes you have [Xcode](https://developer.apple.com/xcode/) and [Homebrew](https://brew.sh/) installed.

```bash
$ brew install node # installs npm as well
```

---

## `bitcoind` installation and setup instructions

We need [`bitcoind`](https://github.com/bitcoin/bitcoin), for downloading and interacting with the Bitcoin blockchain.

### Ubuntu

```bash
$ sudo add-apt-repository ppa:bitcoin/bitcoin
$ sudo apt update
$ sudo apt install bitcoind
```

### macOS

```
$ brew install bitcoin
```

Downloading and verifying the Bitcoin blockchain is going to take a while, so you should start that right away.

First, edit your `bitcoind` configuration file, found in `~/.bitcoin/bitcoin.conf`. It should look like this, at the bare minimum:

```
rpcuser=REPLACEME
rpcpassword=REPLACEME
```

For more information on configuring bitcoind please see [this](https://en.bitcoin.it/wiki/Running_Bitcoin) page

Start `bitcoind`:

```bash
$ bitcoind -testnet -daemon
```

`bitcoind` will now sync the blockhain. This will take a while, typically several hours. You can inspect the log to ensure that progress is made:

```bash
$ tail -f $HOME/.bitcoin/testnet3/debug.log
```

---

## `c-lightning` installation instructions

`lightningd` is the piece of software you use to interact with the Lightning Network. It is installed by downloading and compiling the `c-lightning` software project. Note that we currently need `c-lightning` version 0.6. First, get dependencies:

### Ubuntu

```bash
$ sudo apt-get update
$ sudo apt-get install -y autoconf automake build-essential git libtool libgmp-dev libsqlite3-dev python python3 net-tools zlib1g-dev
```

### macOS

```bash
$ brew install autoconf automake libtool python3 gmp gnu-sed
```

Then, download the source code and compile:

```bash
$ cd
$ git clone https://github.com/ElementsProject/lightning.git
$ cd lightning
$ git fetch && git checkout v0.6
$ ./configure
$ make
# Optionally, if you want to have c-lightning in your path
$ sudo make install
```

# Running `sb-api`

Assuming `bitcoind` has finished syncing the blockchain, we can now start `lightningd`. To check the status of `bitcoind`:

```bash
$ bitcoin-cli -testnet getblockchaininfo
{
  ...
  "blocks": [block height]
  ...
  "verificationprogress": [percentage of sync complete]
  ...
}
```

Starting `lightningd`:

```bash
$ ./lightingd/lightningd --daemon
$ ./cli/lightning-cli getinfo # if this succeeds, lightningd is working
```

`lightningd` also needs some time to sync the blockchain, although this is going to be faster than `bitcoind`. To check the progress of `lightningd`:

```bash
$ ./cli/lightning-cli getinfo
{
  ...
  "blockheight": [block height]
  ...
}
```

Funds sent to your Lightning node will not show up until `lightningd` has finished syncing.

To get some free testnet coins, generate a new address and paste it in at the [TestNet Bitcoin Sandbox](https://testnet.manu.backend.hamburg/faucet):

```bash
$ ./cli/lightning-cli newaddr p2sh-segwit
{
  "address": "[should be a Bitcoin address here]"
}
```

You will need to wait 3 confirmations for your transaction to confirm. You can monitor your transaction on [this](https://testnet.smartbit.com.au/) block explorer.

```bash
#connect to suredbits testnet lightning network node
$ ./cli/lightning-cli connect 0338f57e4e20abf4d5c86b71b59e995ce4378e373b021a7b6f41dabb42d3aad069@ln.test.suredbits.com

#open a channel with suredbits and fund that channel with 100,000 satoshis, note you need 1 confirmation on your testnet tx
$ ./cli/lightning-cli fundchannel 0338f57e4e20abf4d5c86b71b59e995ce4378e373b021a7b6f41dabb42d3aad069 100000
```

You will need to wait 3 confirmations for your transaction to confirm. You can monitor channel funding transaction on [this](https://testnet.smartbit.com.au/) block explorer.

## Starting the API

Note, that the channel needs to be in the state `CHANNELD_NORMAL` before you can start the client, you can check this with the following command

```bash
$ ./cli/lightning-cli listpeers
{
  "peers": [
    {
      "id": "0338f57e4e20abf4d5c86b71b59e995ce4378e373b021a7b6f41dabb42d3aad069",
      "connected": true,
      "netaddr": [
        "54.218.43.96:9735"
      ],
      "alias": "eclair",
      "color": "49daaa",
      "channels": [
        {
          "state": "CHANNELD_NORMAL",
	  ...
        }
      ]
    }
  ]
}
```

Asuming that the channel is in state `CHANNELD_NORMAL`, we should be able to start your webserver and query our API!

First let's clone the `suredbits/lightning-charge` project

```bash
$ cd
$ git clone https://github.com/SuredBits/lightning-charge.git
$ cd lightning-charge
$ git fetch && git checkout sb_api
```

Now that you have cloned and moved into the `lightning-charge` directory, we can install dependencies and start the API!

```bash
$ npm install
$ npm start
```

You should expect output that looks something like this after 5 seconds:

```bash
> lightning-charge@0.4.1 start /home/chris/dev/lightning-charge
> bin/start.sh

API Request:  {"uuid":"3daa1d60-9430-11e8-a087-d736f35db536","channel":"info"}

LN Invoice : {"uuid":"3daa1d60-9430-11e8-a087-d736f35db536","invoice":"lntb10n1pd476tzpp5ejk020ds9cs7ulu80ecahnzwl6ngkym8kyl7a7n8h9ycguk8rzxsdqqxqrrssg09q7ya3vnec33z8rreynjkuse5frzws7rh505f2ucktv89tj0fykjvwpd68nczyejysrmtv7m5tv6d65has8we3tmyfjly0p2m02hsq4ygsxe"}


API response: {"version":"8","lastRosterDownload":"20180730T185645.071Z","seasonType":"Regular","seasonYear":2017,"week":"NflWeek17"}

LN Invoice Payment: {"id":161,"payment_hash":"ccacf53db02e21ee7f877e71dbcc4efea68b1367b13feefa67b9498472c7188d","destination":"0338f57e4e20abf4d5c86b71b59e995ce4378e373b021a7b6f41dabb42d3aad069","msatoshi":1000,"msatoshi_sent":1002,"timestamp":1532979555,"created_at":1532979555,"status":"complete","payment_preimage":"cd5b2a1fcab30e5f267b99e76bff5392217bf4dbcfa148c80bf77c5cd43ea331","getroute_tries":1,"sendpay_tries":1,"route":[{"id":"0338f57e4e20abf4d5c86b71b59e995ce4378e373b021a7b6f41dabb42d3aad069","channel":"1356054:2387:0","msatoshi":1002,"delay":9}],"failures":[]}
```

You can see more API calls you can make in our suredbits module [here](https://github.com/SuredBits/lightning-charge/blob/sb_api/src/sb_websocket_client.js) or by looking at our [API docs on suredbits.com](https://suredbits.com/api)

## License

MIT
