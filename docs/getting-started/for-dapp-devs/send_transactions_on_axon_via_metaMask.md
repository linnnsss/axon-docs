---
title: Send Transactions On Axon Via MetaMask
hide_title: true
sidebar_position: 2
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Send Transactions On Axon Via MetaMask

To proceed with this guide, you must have MetaMask installed. Visit [Metamask](https://metamask.io/) and you will be automatically guided to the relevant store to download the extension or app based on the device and browser you’re using.

This guide provides instructions for sending transactions on Axon via MetaMask after setting up the Axon local node.

## 1 Set Up an Axon Node

### 1.1 Local Setup

This document assumes that you have followed [the Axon quick start](./quick_start.md) to run an Axon node.

Once the node has been successfully set up, you will notice that the block height is increasing, for instance: 

```log
Overlord: state goto new height 2171.
```

### 1.2 Add Axon to MetaMask's Local Network

#### Open Setting

<img alt="open settings" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/2.1_open_settings.png")}  width="50%"/>

#### Choose Networks

<img alt="choose networks" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/2.2_choose_networks.png")}  width="50%"/>

#### Add Network

<img alt="Add network" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/2.3_Add_network.png")}  width="50%"/>

#### Config Axon Network Manually

<img alt="Config Axon Network Manually" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/2.4_Config_Axon_Network_Manually.png")}  width="80%"/>

On the <b>Networks</b> page, make sure that the <b>New RPC URL</b> and <b>Chain ID</b> are configured according to the following information. Copy and paste the text from the boxes below:

**Network name**: axon-devnet

**New RPC URL**: http://localhost:8000

**Chain ID**: 0x41786f6e
> This is the hexadecimal of ASCII string "Axon"

**Currency symbol**: axon


If you know Axon well enough, you can modify the <b>RPC URL</b> and <b>Chain ID</b>. They are in [`devtools/chain/config.toml`](https://github.com/axonweb3/axon/blob/88c9a91354187f7935d4a17d1e0bbc9ef517519f/devtools/chain/config.toml#L7-L9) and [`devtools/chain/specs/single_node/chain-spec.toml`](https://github.com/axonweb3/axon/blob/88c9a91354187f7935d4a17d1e0bbc9ef517519f/devtools/chain/specs/single_node/chain-spec.toml#L8-L9).


#### Save Axon Network

Once you have filled out all the items above, click <b>Save</b> and you will be notified that the Axon network has been added.

<img alt="Untitled" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/Untitled.png")}  width="80%"/>

## 2 Send a Transaction

### 2.1 Add an Account

Add your Axon Genesis account to the local network. MetaMask supports importing accounts via both private keys and keystore files.

In Axon’s repository, the Genesis accounts and their associated funds are configured in [`devtools/chain/specs/single_node/chain-spec.toml`](https://github.com/axonweb3/axon/blob/6fe5777e0b4a9b994dc84a56a00005745fd05085/devtools/chain/specs/single_node/chain-spec.toml#L18-L56). Select an account and then use its corresponding private key.

For example, the private key `0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80`, which corresponds to the address `0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266`, is derived from the mnemonic phrase `test test test test test test test test test test test junk` for the first account. This mnemonic phrase is consistent with [the one used for the initialization of the Hardhat Network](https://hardhat.org/hardhat-network/docs/reference#initial-state).

The account and the balance will be displayed once the private key is added. This Genesis account holds 100,000,000,000,000 AXON, as configured in [`devtools/chain/specs/single_node/chain-spec.toml`](https://github.com/axonweb3/axon/blob/6fe5777e0b4a9b994dc84a56a00005745fd05085/devtools/chain/specs/single_node/chain-spec.toml#L20).

<img alt="Untitled 1" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/Untitled 1.png")}  width="50%"/>

### 2.2 Send a Transaction

Click <b>Send</b> on the balance page and let’s transfer some tokens to another account. Here we are about to transfer 100 AXON to `0xdc796dfc1bb45f21d17be267877c3388d766937b`.

<img alt="Untitled 2" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/Untitled 2.png")}  width="50%"/>

Click <b>Next</b>.

<img alt="Untitled 3" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/Untitled 3.png")}  width="50%"/>

Click <b>Confirm</b>.

<img alt="Untitled 4" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/Untitled 4.png")}  width="80%"/>

You'll see that the transaction is in <b>Pending</b>. It takes a few seconds for the status to change, then you'll know that the transaction has been successful and the balance is 100 AXON less.

<img alt="Untitled 5" src={useBaseUrl("img/for-dapp-devs/send-transactions-on-axon-via-metamask/Untitled 5.png")}  width="80%"/>
