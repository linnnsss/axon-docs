---
title: Executor
hide_title: true
sidebar_position: 6
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Executor

Executor is a module under the core module which executes block transactions on the Axon chain. The Executor contains the implementation of [Precompiled Contracts](https://docs.axonweb3.io/category/contract-info) and [System Contracts](https://docs.axonweb3.io/contract/system_contacts). It also stores the metadata, CKB headers, [Image Cells](https://docs.axonweb3.io/getting-started/for-contributor/system_contract/image_cell) etc. The Executor can execute transactions for axon system contracts or any EVM compatible contracts on Axon. The application binary interface (ABI) of Metadata, CKB Light Client and Image Cells is defined in Solidity inside another module called “Builtin Contracts”. For each system contract except Native Token, there is a corresponding precompiled contract. The overall architecture is shown below.

<img src={useBaseUrl("img/for-contributors/Executor overall architecture.png")}/>

## Adapter

Executor has an adapter which is mainly used for API calls to the underlying executor system.  The trait of the ExecutorAdapter is defined as follows:

```rust
pub trait ExecutorAdapter: ApplyBackend + Backend {
    fn set_origin(&mut self, origin: H160);  // set the origin of the transaction to be sender.
    fn set_gas_price(&mut self, gas_price: U256);
    fn get_logs(&mut self) -> Vec<Log>;
    fn commit(&mut self) -> MerkleRoot;
    fn get(&self, key: &[u8]) -> Option<Bytes>;  // get data from the key-value db.
    fn get_ctx(&self) -> ExecutorContext;
    fn get_account(&self, address: &H160) -> Account;
    fn save_account(&mut self, address: &H160, account: &Account);
}
```

We could see that the ExecutorAdapter is an EVM compatible backend. The Executor supports Legacy/Eip2930/Eip1559, three versions of transactions.

## Transaction Execution

To execute a transaction, a base gas is required plus the calculated gas for each operation. For the precompiled contracts, the gas fee is fixed for each contract.

For `Create` transaction, base gas is  53,000. For `Call`, base gas is 21,000.

The entire execution process is as follows:

1. For each transaction inside a block
    1. In the beginning, the prepaid gas `gas_limit * gas_price` is deducted from the account who sends the transaction.
    2. Given different actions, the executor either creates a contract or calls a specific address.
    3. Any remaining gas is calculated and added back to the sender’s account.
    4. If the transaction is the `Create`, the address is returned in the `TxResp.code_address`.
2. If the block is not the genesis block, fees are allocated to the validators based on their weights.
3. Update the global state root.

```rust
pub struct TxResp {
    pub exit_reason:  ExitReason,
    pub ret:          Vec<u8>,
    pub gas_used:     u64,
    pub remain_gas:   u64,
    pub fee_cost:     U256,
    pub logs:         Vec<Log>,
    pub code_address: Option<Hash>,
    pub removed:      bool,
}

pub struct ExecResp {
    pub state_root:   MerkleRoot,
    pub receipt_root: MerkleRoot,
    pub gas_used:     u64,
    pub tx_resp:      Vec<TxResp>,
}
```

## Data Storage

The storage of core data, including Axon metadata, headers, CKB cells, is implemented through [ckb-rocksdb](https://github.com/nervosnetwork/rust-rocksdb), which is basically a key-value store.

For Metadata, the format is `{epoch: metadata}`, and the epoch can be determined by the block number. To achieve that, another storage is used to store epoch ranges as a vector, `[0, end_block_1, end_block_2, ...]`, and the epoch is implied by the index of the range where the current block number is. The end block is determined by the metadata.

For example, the `version` field in the metadata records the start and end block number that the metadata is in effect. Suppose currently there are three metadata shown in the following figure. In such case, the epoch storage stores `[0, 100, 200, 300]`. Now we have a transaction at block number 150. We know it is in the range `[100, 200]`, which is the second range in the list. As a result, the metadata taken effect for the transaction is the second one.

<img src={useBaseUrl("img/for-contributors/Executor metadata and blocks.png")}/>

For CKB Light Client and Image Cell, they share one database since the former stores headers and the latter stores cells of CKB layer 1 information.

The key to retrieve a header is `block_hash`, to retrieve a call, `(out_point.tx_hash, out_point.index)`.

The structure of the `Header` is:

```rust
pub struct Header {
        pub version:           u32,
        pub compact_target:    u32,
        pub timestamp:         u64,
        pub number:            u64,
        pub epoch:             u64,
        pub parent_hash:       [u8; 32],
        pub transactions_root: [u8; 32],
        pub proposals_hash:    [u8; 32],
        pub uncles_hash:       [u8; 32],
        pub dao:               [u8; 32],
        pub nonce:             u128,
        pub block_hash:        [u8; 32],
    }
```

For the `CellInfo`, it is in the format of:

```rust
pub struct CellInfo {
    pub cell_output:     Bytes, // packed::CellOutput
    pub cell_data:       Bytes,
    pub created_number:  u64,   // block number that the cell is created
    pub consumed_number: Option<u64>,  // block number that the cell is consumed
}
```
