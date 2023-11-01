---
title: Hardfork
hide_title: true
sidebar_position: 7
---

# Hardfork

Axon empowers developers to initiate hardforks. The currently available hardfork, Andromeda, takes its name from one of the 88 constellations. The upcoming hardforks on Axon will follow [this order of the constellations](https://en.wikipedia.org/wiki/IAU_designated_constellations#List). 

When Andromeda is enabled, validators are allowed to modify the `max_contract_limit` through [the system contract `0xffffffffffffffffffffffffffffffffffffff01`](https://docs.axonweb3.io/contract/system_contacts/#metadata). If the hardfork is not activated, only the `max_contract_limit` is immutable, while other configs remain adjustable. Axon provides a [hardfork test CI](https://github.com/axonweb3/axon/blob/f9974e62924693494476560316db9f70bc650b80/.github/workflows/hardfork_test.yml) and [a test project](https://github.com/axonweb3/axon-hardfork-test) for your reference.

## Usage Instructions

### 1. Build Axon from source code
Start to build Axon from the source code using the following commands:

   ```bash
   cd $your_workspace
   git clone https://github.com/axonweb3/axon.git
   cd axon
   cargo build
   ```

### 2. Start multiple Axon nodes

To run multiple Axon nodes, use the [`reset.sh`](https://github.com/axonweb3/axon-hardfork-test/blob/5c9c172cc1ed1dff544f7e092f7052c314030c1d/reset.sh) script to clear data and start axon nodes. You can also use it for the first-time startup.

   ```bash
   git clone https://github.com/axonweb3/axon-hardfork-test.git
   cd axon-hardfork-test
   bash reset.sh $your_workspace/axon
   ```

The default configuration to enable hardfork is [`hardforks = []`](https://github.com/axonweb3/axon/blob/f9974e62924693494476560316db9f70bc650b80/devtools/chain/specs/multi_nodes/chain-spec.toml#L10). The `reset.sh` script changes `hardforks = []` to `hardforks = ["None"]` to disable hardfork. Later, you can enable hardfork using `hardfork -c` followed by specifying the `hardfork-start-number`.
You can expect the output to resemble the following:

   ```bash
   No process found listening on port 8001
   No process found listening on port 8002
   No process found listening on port 8003
   No process found listening on port 8004
   hardforks = ["None"]
   node_1 height: 6 // `height: 6` indicates that the nodes are producing blocks without any issues. 
   node_2 height: 6
   node_3 height: 6
   node_4 height: 6
   {
     "jsonrpc": "2.0",
     "result": {}, // When querying with `axon_getHardforkInfo` and receiving a response of `"result": {}`, it implies that the hardfork is currently disabled.
     "id": 1
   }
   {
     "jsonrpc": "2.0",
     "result": {},
     "id": 2
   }
   {
     "jsonrpc": "2.0",
     "result": {},
     "id": 3
   }
   {
     "jsonrpc": "2.0",
     "result": {},
     "id": 4
   }
   ```

### 3. Enable hardfork

Use the [`hardfork.sh`](https://github.com/axonweb3/axon-hardfork-test/blob/5c9c172cc1ed1dff544f7e092f7052c314030c1d/hardfork.sh) script to activate the hardfork, [after 30 blocks by default](https://github.com/axonweb3/axon-hardfork-test/blob/5c9c172cc1ed1dff544f7e092f7052c314030c1d/hardfork.sh#L18).
The `hardfork.sh` first halts all nodes, then executes `hardfork -c` to specify the `hardfork-start-number` to restart the nodes. After restarting, you can check the status of `Andromeda` with `axon_getHardforkInfo`, where the expected status value is `determined`.

  ```bash
  bash hardfork.sh $your_workspace/axon	
  ```

You should see an output similar to this following:

  ```bash
  axon_path: /Users/sunchengzhu/tmp/axon
  hardfork-start-number: 694
  Killing processes on port 8001: 9285
  Killing processes on port 8002: 9286
  Killing processes on port 8003: 9287
  Killing processes on port 8004: 9288
  node_1 height: 670
  node_2 height: 670
  node_3 height: 670
  node_4 height: 670
  {
    "jsonrpc": "2.0",
    "result": {
      "Andromeda": "determined"
    },
    "id": 1
  }
  {
    "jsonrpc": "2.0",
    "result": {
      "Andromeda": "determined"
    },
    "id": 2
  }
  {
    "jsonrpc": "2.0",
    "result": {
      "Andromeda": "determined"
    },
    "id": 3
  }
  {
    "jsonrpc": "2.0",
    "result": {
      "Andromeda": "determined"
    },
    "id": 4
  }
  ```

### 4. Wait and verify

You can verify if `Andromeda`'s status becomes `enabled` when the nodes reach the specified height, using the following test case:

   ```bash
   npm install
   npx hardhat test --grep "check hardfork info after hardfork"
   ```

### 5. Deploy a contract larger than the default `max_contract_limit`

Execute the following test to confirm that the deployment transaction [returns an error message containing `CreateContractLimit`](https://github.com/axonweb3/axon-hardfork-test/blob/5c9c172cc1ed1dff544f7e092f7052c314030c1d/test/checkMetadata.ts#L18-L25).

   ```bash
   npx hardhat test --grep "deploy a big contract larger than max_contract_limit"
   ```

### 6. Increase `max_contract_limit`

Use the system contract `0xffffffffffffffffffffffffffffffffffffff01` to increases the `max_contract_limit` with the following test:

   ```bash
   npx hardhat test --grep "update max_contract_limit"
   ```

### 7. Test the new `max_contract_limit`

Deploy the previous contract again, and anticipate a successful deployment with the updated `max_contract_limit`.

   ```bash
   npx hardhat test --grep "deploy a big contract smaller than max_contract_limit"
   ```
