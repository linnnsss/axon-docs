---
title: Deploy and Interact with a Solidity Contract
hide_title: true
sidebar_position: 3
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Deploy and Interact with a Solidity Contract

## Deploy a Solidity contract
Given Axon’s full compatibility with the EVM, contract deployment on Axon closely resembles the process on Ethereum. You can follow the [Quick Start](https://hardhat.org/hardhat-runner/docs/getting-started#quick-start) for guidance. The only distinction is the network, Axon. To make the adjustment, you need to edit the [hardhat.config.ts](https://hardhat.org/hardhat-runner/docs/config) file as follows, which is done on the internal Axon’s testnet.

```javascript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";

// Axon genesis account configrued on the local / test network.
const AXON_PRIVATE_KEY = "0x37aa0f893d05914a4def0460c0a984d3611546cfb26924d7a7ca6e0db9950a2d";

const config: HardhatUserConfig = {
  solidity: "0.8.19",
  networks: {
    axon: {
      chainId: 2022,
      url: "The axon net URL"
      accounts: [AXON_PRIVATE_KEY],
    },
  },
};

export default config;
```

Then deploy the contract:
```shell
$ npx hardhat run scripts/deploy.ts --network axon

Lock with 0.001ETH and unlock timestamp 1692058859 deployed to 0x7CcECF6cc5E022F7D582deF5d5b53fD179f9A368
```
You can follow [this](https://docs.axonweb3.io/getting-started/for-dapp-devs/send_transactions_on_axon_via_metaMask/#11-local-setup) instruction to set up an Axon node locally, and replace the `url` in the `hardhat.config.ts` file with `http://127.0.0.1:8000`.

## Interact With the Deployed Contract
Interacting with Axon contracts is the same as with Ethereum contracts. You can refer to [Connecting to Existing Contracts](https://docs.ethers.org/v4/api-contract.html#connecting-to-existing-contracts) for more details.

Below is an example:

```javascript
import { ethers } from "ethers";

// Copy from 'solidity-contract/artifacts/contracts/Lock.sol/Lock.json'
import lockContract from "./Lock.json";

// Address of the deployed contract
const contractAddress = "0x7CcECF6cc5E022F7D582deF5d5b53fD179f9A368";

async function main() {
  // Connect to the network
  let provider = new ethers.JsonRpcProvider(AXON_NET_URL);

  // A Signer from a private key
  const signer = new ethers.Wallet(AXON_PRIVATE_KEY, provider);

  // Create a new instance of the Contract with a Signer, which allows
  // update methods
  const contract = new ethers.Contract(
    contractAddress,
    lockContract.abi,
    signer,
  );

  console.log(
    "before calling the contract: ",
    await provider.getBalance(contract.target),
  );

  // Call a Contract's non-constant method
  let tx = await contract.withdraw();
  console.log("tx hash: ", tx.hash);

  // The operation is NOT complete yet; we must wait until it is mined
  await tx.wait();

  console.log(
    "after calling the contract: ",
    await provider.getBalance(contract.target),
  );

  // Call Contract's read-only constant methods
  console.log("unlock time: ", await contract.unlockTime());
  console.log("contract owner: ", await contract.owner());
}

main();
```
