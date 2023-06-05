---
title: Configuration
hide_title: true
sidebar_position: 1
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Configuration
## Basics
### accounts
Account is a list that includes the addresses and amount of tokens initially allocated. The value of each node has must be consistent.

| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Vec<InitialAccount\> | - | - |  False  |

  Example:
```toml
[[accounts]]
address = "0xa0ee7a142d267c1f36714e4a8f75612f20a79720"
balance = "04ee2d6d415b85acef8100000000"
```

### crosschain_contract_address
The address of the cross-chain contract. The value each nodes has must be consistent. (They are not in use until the official launch of the cross-chain functionality.)
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| H160 | - | - |  False  |
  
### data_path
The path for data storage.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| String | - | - |  False  |
  
### privkey
The private key for the HTTP listening port node, used for handshake and message signing. Privkey can be directly written in the configuration file or read from an environment variable. For instance, privkey = "A" refers to the privkey fetches its value from Environment Variable A.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Hex | - | - |  False  |
  
### wckb_contract_address
The address of the  WCKB (Wrapped CKB) ERC20 contract that represents the cross-chain assets transferred from CKB (currently not in use). The value each node has must be consistent.

| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| H160 | - | - |  False  |

## RPC
### maxconn
The maximum number of TCP connections.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Unit | 25000 | - |  False  |
  
### max_payload_size
The maximum payload size for RPC requests, primarily for limiting the `send_rawTransaction` interface. Recommended value is 1MB.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 1MB | Byte |  False  |
  
### http_listening_address
The HTTP listening port.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| SocketAddr | - | - |  False  |
  
### ws_listening_address
The WebSocket listening port.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| SocketAddr | - | - |  False  |
  
### client_version
The client version.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| String | "0.1.0" | - |  True  |
  
### gas_cap
The maximum gas limit allowed for RPC.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 25000000 | - |  True  |
  

## Network
### bootstraps
An array containing the list of bootstrap nodes.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Vec<String\> | - | - |  False  |
  
### listening_address
The listening address.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| MultiAddr | - | - |  False  |
   

### max_connected_peers
The maximum number of connectable peers.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 40 | - |  True  |
  
### ping_interval
The interval of the timed ping-pong messages of the protocol.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 15 | s |  True  |
  
### rpc_timeout
The timeout duration of RPC requests.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 10 | s |  True  |
  
### selfcheck_interval
The self-check interval of the maximum number of inbound P2P network connections.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 35 | s |  True  |
  

## Mempool
### pool_size
The size of the transaction pool.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | - | - |  False  |
  
### timeout_gap
The number of blocks after which a transaction becomes invalid.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | - | - |  False  |
  
### broadcast_txs_interval
The interval of broadcasting transactions.

| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 200 | ms |  True  |
  
### broadcast_txs_size
The number of transactions broadcasted at once.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 200 | - |  True  |

## Executor
### triedb_cache_size
The size of the trie database cache. Recommended value is 500 or higher. Larger value consumes more memory.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | - | - | False  |

## Consensus
### sync_txs_chunk_size
The maximum number of transactions in a chunk when synchronizing transactions.

| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 500 | - |  True  |
  
## Logger
### console_show_file_and_line
Whether to display file names and line numbers when output in console.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Bool | False | - |  True  |
  
### file_size_limit
The maximum size of log files. 1GB by default.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 1024\*1024\*1024 | Byte |  True  |
  
### filter
The level of log output.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| String | Info | - |  True  |
  
### log_path
The path of log files.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| String | data_path/logs | - |  True  |
  
### log_to_console
Whether to output logs to console.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Bool | True | - |  True  |
  
### log_to_file
Whether to output logs to files.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Bool | True | - |  True  |
  
### metrics
Whether to enable metrics.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Bool | True | - |  True  |
  
### modules_level
The log output level defined by module.  

| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| HashMap<String, String\> | - | - |  True  |

For example:

```toml
modules_level = { "overlord::state::process" = "debug", core_consensus = "error" }
```

## Rocksdb
### max_open_files
The maximum number of open files.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | - | - |  False  |
  
### cache_size
The cache size for each column family. Larger value consumes more memory. Recommended to be 50 or higher.
 
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | 100 | - |  True  |
  
### options_file
The path to the RocksDB configuration file. It is recommended to use the provided configuration file.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| String | - | - |  True  |

## Jaeger (Optional)
### service_name (Optional)
The name of service.

| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| String | - | - |  True  |
  
### tracing_address (Optional)
The address to send tracing span.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| SocketAddr | - | - |  True  |
  
### tracing_batch_size (Optional)
The size of tracing batch.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| Uint | - | - |  True  |
  
## Prometheus (Optional)
### listening_address (Optional)
The Prometheus listening port.
  
| Value Type| Default Value| Unit | Optional |
| --------- | ------------ | ---- | -------- |
| SocketAddr | - | - |  True  |
