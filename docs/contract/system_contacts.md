---
title: System Contracts
hide_title: true
sidebar_position: 1
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# System Contracts
System contracts are built-in contracts designed to facilitate inter-chain communications.

:::tip

Notice that only **validator** can call system contracts.

:::

## Native Token
The Native Token is the contract for Axon Token (AT), applying balance changes to the account.

### Address
```
0xffffffffffffffffffffffffffffffffffffff00
```

## Metadata
The Metadata Contract records the metadata of the chain, such as validator identities and epoch. Applications built on Axon can interact with the Metadata Contract through the precompiled contract `Metadata`.

### Address

```
0xFfFfFFFfFFfFFFFfFFFFffFfFFfffFFFfffffF
```

### ABI

<details><summary>Click here to view</summary>

```json
[
  {
    "inputs": [
      {
        "components": [
          {
            "components": [
              {
                "internalType": "uint64",
                "name": "start",
                "type": "uint64"
              },
              {
                "internalType": "uint64",
                "name": "end",
                "type": "uint64"
              }
            ],
            "internalType": "struct MetadataManager.MetadataVersion",
            "name": "version",
            "type": "tuple"
          },
          {
            "internalType": "uint64",
            "name": "epoch",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "gas_limit",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "gas_price",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "interval",
            "type": "uint64"
          },
          {
            "components": [
              {
                "internalType": "bytes",
                "name": "bls_pub_key",
                "type": "bytes"
              },
              {
                "internalType": "bytes",
                "name": "pub_key",
                "type": "bytes"
              },
              {
                "internalType": "address",
                "name": "address_",
                "type": "address"
              },
              {
                "internalType": "uint32",
                "name": "propose_weight",
                "type": "uint32"
              },
              {
                "internalType": "uint32",
                "name": "vote_weight",
                "type": "uint32"
              }
            ],
            "internalType": "struct MetadataManager.ValidatorExtend[]",
            "name": "verifier_list",
            "type": "tuple[]"
          },
          {
            "internalType": "uint64",
            "name": "propose_ratio",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "prevote_ratio",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "precommit_ratio",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "brake_ratio",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "tx_num_limit",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "max_tx_size",
            "type": "uint64"
          }
        ],
        "internalType": "struct MetadataManager.Metadata",
        "name": "metadata",
        "type": "tuple"
      }
    ],
    "name": "appendMetadata",
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [],
    "name": "construct",
    "outputs": [],
    "stateMutability": "nonpayable",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "uint64",
        "name": "epoch",
        "type": "uint64"
      }
    ],
    "name": "getMetadata",
    "outputs": [
      {
        "components": [
          {
            "components": [
              {
                "internalType": "uint64",
                "name": "start",
                "type": "uint64"
              },
              {
                "internalType": "uint64",
                "name": "end",
                "type": "uint64"
              }
            ],
            "internalType": "struct MetadataManager.MetadataVersion",
            "name": "version",
            "type": "tuple"
          },
          {
            "internalType": "uint64",
            "name": "epoch",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "gas_limit",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "gas_price",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "interval",
            "type": "uint64"
          },
          {
            "components": [
              {
                "internalType": "bytes",
                "name": "bls_pub_key",
                "type": "bytes"
              },
              {
                "internalType": "bytes",
                "name": "pub_key",
                "type": "bytes"
              },
              {
                "internalType": "address",
                "name": "address_",
                "type": "address"
              },
              {
                "internalType": "uint32",
                "name": "propose_weight",
                "type": "uint32"
              },
              {
                "internalType": "uint32",
                "name": "vote_weight",
                "type": "uint32"
              }
            ],
            "internalType": "struct MetadataManager.ValidatorExtend[]",
            "name": "verifier_list",
            "type": "tuple[]"
          },
          {
            "internalType": "uint64",
            "name": "propose_ratio",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "prevote_ratio",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "precommit_ratio",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "brake_ratio",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "tx_num_limit",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "max_tx_size",
            "type": "uint64"
          }
        ],
        "internalType": "struct MetadataManager.Metadata",
        "name": "",
        "type": "tuple"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  
  {
    "inputs": [
      {
        "internalType": "address",
        "name": "relayer",
        "type": "address"
      }
    ],
    "name": "isVerifier",
    "outputs": [
      {
        "internalType": "bool",
        "name": "",
        "type": "bool"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [],
    "name": "verifierList",
    "outputs": [
      {
        "internalType": "address[]",
        "name": "",
        "type": "address[]"
      },
      {
        "internalType": "uint256",
        "name": "",
        "type": "uint256"
      }
    ],
    "stateMutability": "view",
    "type": "function"
  }
]
```

</details>

## CKB Light Client
The CKB Light Client Contract is the light client on Axon for CKB Layer 1. It receives new headers from CKB Layer 1 through Forcerelay, an essential part of Axon that enables the interaction between Axon-based chains, Ethereum, and Cosmos-SDK chains via IBC protocol. Upon receiving new headers, the CKB Light Client Contract automatically updates header info, enabling applications to access this data via the precompiled contract `GetHeader`.

### Address

```
0x10000000000000000003
```

### ABI

<details><summary>Click here to view</summary>

```json
[
  {
    "inputs": [
      {
        "internalType": "bytes32[]",
        "name": "blockHashs",
        "type": "bytes32[]"
      }
    ],
    "name": "rollback",
    "outputs": [],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "bool",
        "name": "allowRead",
        "type": "bool"
      }
    ],
    "name": "setState",
    "outputs": [],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "components": [
          {
            "internalType": "uint32",
            "name": "version",
            "type": "uint32"
          },
          {
            "internalType": "uint32",
            "name": "compactTarget",
            "type": "uint32"
          },
          {
            "internalType": "uint64",
            "name": "timestamp",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "number",
            "type": "uint64"
          },
          {
            "internalType": "uint64",
            "name": "epoch",
            "type": "uint64"
          },
          {
            "internalType": "bytes32",
            "name": "parentHash",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "transactionsRoot",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "proposalsHash",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "unclesHash",
            "type": "bytes32"
          },
          {
            "internalType": "bytes32",
            "name": "dao",
            "type": "bytes32"
          },
          {
            "internalType": "uint128",
            "name": "nonce",
            "type": "uint128"
          },
          {
            "internalType": "bytes32",
            "name": "blockHash",
            "type": "bytes32"
          }
        ],
        "internalType": "struct CkbType.Header[]",
        "name": "headers",
        "type": "tuple[]"
      }
    ],
    "name": "update",
    "outputs": [],
    "stateMutability": "view",
    "type": "function"
  }
]
```

</details>

## Image Cell
The Image Cell Contract stores the CKB Layer 1 cells in Axon, allowing applications on Axon to read CKB data through the precompiled contract `GetCell`. 

### Address

```
0x10000000000000000004
```

### ABI

<details><summary>Click here to view</summary>

```json
[
  {
    "inputs": [
      {
        "components": [
          {
            "components": [
              {
                "internalType": "bytes32",
                "name": "txHash",
                "type": "bytes32"
              },
              {
                "internalType": "uint32",
                "name": "index",
                "type": "uint32"
              }
            ],
            "internalType": "struct CkbType.OutPoint[]",
            "name": "txInputs",
            "type": "tuple[]"
          },
          {
            "components": [
              {
                "internalType": "bytes32",
                "name": "txHash",
                "type": "bytes32"
              },
              {
                "internalType": "uint32",
                "name": "index",
                "type": "uint32"
              }
            ],
            "internalType": "struct CkbType.OutPoint[]",
            "name": "txOutputs",
            "type": "tuple[]"
          }
        ],
        "internalType": "struct ImageCell.BlockRollBlack[]",
        "name": "blocks",
        "type": "tuple[]"
      }
    ],
    "name": "rollback",
    "outputs": [],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "internalType": "bool",
        "name": "allowRead",
        "type": "bool"
      }
    ],
    "name": "setState",
    "outputs": [],
    "stateMutability": "view",
    "type": "function"
  },
  {
    "inputs": [
      {
        "components": [
          {
            "internalType": "uint64",
            "name": "blockNumber",
            "type": "uint64"
          },
          {
            "components": [
              {
                "internalType": "bytes32",
                "name": "txHash",
                "type": "bytes32"
              },
              {
                "internalType": "uint32",
                "name": "index",
                "type": "uint32"
              }
            ],
            "internalType": "struct CkbType.OutPoint[]",
            "name": "txInputs",
            "type": "tuple[]"
          },
          {
            "components": [
              {
                "components": [
                  {
                    "internalType": "bytes32",
                    "name": "txHash",
                    "type": "bytes32"
                  },
                  {
                    "internalType": "uint32",
                    "name": "index",
                    "type": "uint32"
                  }
                ],
                "internalType": "struct CkbType.OutPoint",
                "name": "outPoint",
                "type": "tuple"
              },
              {
                "components": [
                  {
                    "internalType": "uint64",
                    "name": "capacity",
                    "type": "uint64"
                  },
                  {
                    "components": [
                      {
                        "internalType": "bytes32",
                        "name": "codeHash",
                        "type": "bytes32"
                      },
                      {
                        "internalType": "enum CkbType.ScriptHashType",
                        "name": "hashType",
                        "type": "uint8"
                      },
                      {
                        "internalType": "bytes",
                        "name": "args",
                        "type": "bytes"
                      }
                    ],
                    "internalType": "struct CkbType.Script",
                    "name": "lock",
                    "type": "tuple"
                  },
                  {
                    "components": [
                      {
                        "internalType": "bytes32",
                        "name": "codeHash",
                        "type": "bytes32"
                      },
                      {
                        "internalType": "enum CkbType.ScriptHashType",
                        "name": "hashType",
                        "type": "uint8"
                      },
                      {
                        "internalType": "bytes",
                        "name": "args",
                        "type": "bytes"
                      }
                    ],
                    "internalType": "struct CkbType.Script[]",
                    "name": "type_",
                    "type": "tuple[]"
                  }
                ],
                "internalType": "struct CkbType.CellOutput",
                "name": "output",
                "type": "tuple"
              },
              {
                "internalType": "bytes",
                "name": "data",
                "type": "bytes"
              }
            ],
            "internalType": "struct CkbType.CellInfo[]",
            "name": "txOutputs",
            "type": "tuple[]"
          }
        ],
        "internalType": "struct ImageCell.BlockUpdate[]",
        "name": "blocks",
        "type": "tuple[]"
      }
    ],
    "name": "update",
    "outputs": [],
    "stateMutability": "view",
    "type": "function"
  }
]
```

</details>
