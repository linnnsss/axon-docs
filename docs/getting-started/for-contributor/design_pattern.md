---
title: Design Pattern
hide_title: true
sidebar_position: 1
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Design Pattern
Developing large-scale projects poses challenges including complexity, maintainability, scalability, and high performance. Complexity arises from multiple interdependent submodules and components, making the overall architecture and logic intricate and difficult to develop, debug, and test. Maintaining such projects becomes critical as they expand, requiring ongoing maintenance, upgrades, and improvements. Proper decoupling of modules is essential to prevent changes in one module from affecting others, which can increase maintenance costs.

In the following sections, we will explore how Axon leverages the Adapter design pattern to decouple its submodules and overcome the challenges of complexity and maintainability.

## Architecture

<img src={useBaseUrl("img/for-contributors/Architect design_pattern_fig.1.png")}/> 

As illustrated above, Axon consists primarily of the following modules: Web3 RPC (Ethereum-compatible), Network, Sync, Consensus; [MemPool](https://docs.axonweb3.io/getting-started/for-contributor/mempool) (aka., memory pool, or transaction pool), [Executor](https://docs.axonweb3.io/getting-started/for-contributor/executor) (EVM), [Interoperability](https://docs.axonweb3.io/getting-started/for-contributor/interoperability), and [Storage](https://docs.axonweb3.io/getting-started/for-contributor/storage) (KV database).

These modules have interdependencies and interactions. For example, the MemPool module relies on the Sync module to synchronize transactions and blocks from other nodes. It also uses the Network module to broadcast the received transactions across the network.

The challenges due to high complexity and maintainability issue are encountered during the development, testing, and maintenance stages of Axon.

## Decoupling Between Modules
To ease development, testing, and maintenance, it is vital to decouple modules. After evaluating several common design patterns, such as the [Facade Pattern](https://en.wikipedia.org/wiki/Facade_pattern) and the [Bridge Pattern](https://en.wikipedia.org/wiki/Bridge_pattern), we found that leveraging the adapter pattern can effectively achieve module decoupling.

The core idea involves abstracting the required functionality of each module from external modules into an Adapter trait. Each module internally implements a concrete Adapter (adapter). When performing its tasks, a module relies solely on the adapter’s interface to interact with other modules to obtain necessary assistance.

### Mempool Class Diagram
Let's use the MemPool module as an example.

<img src={useBaseUrl("img/for-contributors/MemPool module Design pattern fig.2.png")}/> 

### MemPoolAdapter
This is the abstract trait for the MemPool adapter, which declares all the external interfaces required by the MemPool module. The specific implementation is as follows:
```Rust
pub trait MemPoolAdapter: Send + Sync {
    async fn pull_txs(
        &self,
        ctx: Context,
        height: Option<u64>,
        tx_hashes: Vec<Hash>,
    ) -> ProtocolResult<Vec<SignedTransaction>>;

    async fn broadcast_tx(
        &self,
        ctx: Context,
        origin: Option<usize>,
        tx: SignedTransaction,
    ) -> ProtocolResult<()>;

    async fn check_authorization(
        &self,
        ctx: Context,
        tx: &SignedTransaction,
    ) -> ProtocolResult<U256>;

    async fn check_transaction(&self, ctx: Context, tx: &SignedTransaction) -> ProtocolResult<()>;

    ...
}
```
### DefaultMemPoolAdapter
DefaultMemPoolAdapter is the actual implementation of the MempoolAdapter in the Axon code, as shown below:
```Rust
pub struct DefaultMemPoolAdapter<C, N, S, DB, I> {
    network:  N,
    storage:  Arc<S>,
    trie_db:  Arc<DB>,
    metadata: Arc<MetadataHandle>,

    addr_nonce:  DashMap<H160, (U256, U256)>,
    gas_limit:   AtomicU64,
    max_tx_size: AtomicUsize,
    chain_id:    u64,

    stx_tx: UnboundedSender<(Option<usize>, SignedTransaction)>,
    err_rx: Mutex<UnboundedReceiver<ProtocolError>>,

    pin_c: PhantomData<C>,
    pin_i: PhantomData<I>,
}

impl<C, N, S, DB, I> MemPoolAdapter for DefaultMemPoolAdapter<C, N, S, DB, I>
where
    C: Crypto + Send + Sync + 'static,
    N: Rpc + PeerTrust + Gossip + Clone + Unpin + 'static,
    S: Storage + 'static,
    DB: trie::DB + 'static,
    I: Interoperation + 'static,
{
    #[trace_span(kind = "mempool.adapter", logs = "{txs_len: tx_hashes.len()}")]
    async fn pull_txs(
        &self,
        ctx: Context,
        height: Option<u64>,
        tx_hashes: Vec<Hash>,
    ) -> ProtocolResult<Vec<SignedTransaction>> {
    ...
    }

    async fn broadcast_tx(
        &self,
        _ctx: Context,
        origin: Option<usize>,
        stx: SignedTransaction,
    ) -> ProtocolResult<()> {
    ...
    }

    async fn check_authorization(
        &self,
        ctx: Context,
        tx: &SignedTransaction,
    ) -> ProtocolResult<U256> {
        if is_call_system_script(tx.transaction.unsigned.action()) {
            return self.check_system_script_tx_authorization(ctx, tx).await;
        }

        let addr = &tx.sender;
        if let Some(res) = self.addr_nonce.get(addr) {
            if tx.transaction.unsigned.nonce() < &res.value().0 {
                return Err(MemPoolError::InvalidNonce {
                    current:  res.value().0.as_u64(),
                    tx_nonce: tx.transaction.unsigned.nonce().as_u64(),
                }
                .into());
            } else if res.value().1 < tx.transaction.unsigned.may_cost() {
                return Err(MemPoolError::ExceedBalance {
                    tx_hash:         tx.transaction.hash,
                    account_balance: res.value().1,
                    tx_gas_limit:    *tx.transaction.unsigned.gas_limit(),
                }
                .into());
            } else {
                return Ok(tx.transaction.unsigned.nonce() - res.value().0);
            }
        }

        let backend = AxonExecutorAdapter::from_root(
            **CURRENT_STATE_ROOT.load(),
            Arc::clone(&self.trie_db),
            Arc::clone(&self.storage),
            Default::default(),
        )?;

        let account = backend.basic(*addr);
        self.addr_nonce
            .insert(*addr, (account.nonce, account.balance));

        if &account.nonce > tx.transaction.unsigned.nonce() {
            return Err(MemPoolError::InvalidNonce {
                current:  account.nonce.as_u64(),
                tx_nonce: tx.transaction.unsigned.nonce().as_u64(),
            }
            .into());
        }

        if account.balance < tx.transaction.unsigned.may_cost() {
            return Err(MemPoolError::ExceedBalance {
                tx_hash:         tx.transaction.hash,
                account_balance: account.balance,
                tx_gas_limit:    *tx.transaction.unsigned.gas_limit(),
            }
            .into());
        }

        Ok(tx.transaction.unsigned.nonce() - account.nonce)
    }
    ...
```
You can see that some interface implementations are relatively complex, such as `check_authorization`.

### Storage, Network, and Others
These external modules serve as dependencies for the MemPool. For instance, MemPool relies on the storage module to verify if certain transactions exist. Note that both Network and Storage are traits, which are external abstractions of these modules.

## Advantages of Adapter Pattern
### Abstraction
Abstraction provides the advantage of encapsulating and hiding implementation details. Referring to the previous illustration, for the Network trait, regardless of changes in interface implementations (e.g., broadcast), it requires no modifications to the DefaultMemPoolAdapter, as long as the abstract Network trait (i.e., function parameters and return values) remains unchanged.

Furthermore, by using the Adapter pattern, even if traits like Network undergo changes (which is unlikely), the DefaultMemPoolAdapter can hide these changes from the MemPoolImpl (MemPool implementation), minimizing the impact of external module modifications.

### Improved Collaboration
The class diagram suggests that, except for DefaultMemPoolAdapter, the remaining components of MemPool module are independent and can be developed and tested individually.

### Simplified Testing
MemPoolAdapter also facilitates testing. As shown below, the HashMemPoolAdapter implemented tested is much simpler than the actual implementation of HashMemPoolAdapter, which is DefaultMemPoolAdapter.
```Rust
pub struct HashMemPoolAdapter {
    network_txs: DashMap<Hash, SignedTransaction>,
}

impl HashMemPoolAdapter {
    fn new() -> HashMemPoolAdapter {
        HashMemPoolAdapter {
            network_txs: DashMap::new(),
        }
    }
}

#[async_trait]
impl MemPoolAdapter for HashMemPoolAdapter {
    async fn pull_txs(
        &self,
        _ctx: Context,
        _height: Option<u64>,
        tx_hashes: Vec<Hash>,
    ) -> ProtocolResult<Vec<SignedTransaction>> {
        let mut vec = Vec::with_capacity(tx_hashes.len());
        for hash in tx_hashes {
            if let Some(tx) = self.network_txs.get(&hash) {
                vec.push(tx.clone());
            }
        }
        Ok(vec)
    }

    async fn broadcast_tx(
        &self,
        _ctx: Context,
        _origin: Option<usize>,
        tx: SignedTransaction,
    ) -> ProtocolResult<()> {
        self.network_txs.insert(tx.transaction.hash, tx);
        Ok(())
    }

    async fn check_authorization(
        &self,
        _ctx: Context,
        _tx: &SignedTransaction,
    ) -> ProtocolResult<U256> {
        Ok(U256::zero())
    }
...
```
The adapter pattern effectively encapsulates external modules like Storage and Network. Comparing it with DefaultMemPoolAdapter, we can see that the implementation of HashMemPoolAdapter is straightforward. For instance, in transaction pool module testing, there may be no specific requirements for the `check_authorization` interface. In such cases, we can simply return Ok(U256::zero()), significantly simplifying the testing code.

## Implementation of MemPool Module
As explained earlier, the MemPool module abstracts other modules (such as Network and Storage) as traits. Similarly, MemPool module also provides services to other modules and requires its own abstraction. Specifically, it exposes the MemPool abstraction to interact with others. Here is the code for reference:

```Rust
pub trait MemPool: Send + Sync {
    async fn insert(&self, ctx: Context, tx: SignedTransaction) -> ProtocolResult<()>;

    async fn package(
        &self,
        ctx: Context,
        cycles_limit: U256,
        tx_num_limit: u64,
    ) -> ProtocolResult<PackedTxHashes>;

    async fn flush(
        &self,
        ctx: Context,
        tx_hashes: &[Hash],
        current_number: BlockNumber,
    ) -> ProtocolResult<()>;

    async fn get_full_txs(
        &self,
        ctx: Context,
        height: Option<u64>,
        tx_hashes: &[Hash],
    ) -> ProtocolResult<Vec<SignedTransaction>>;

    async fn ensure_order_txs(
        &self,
        ctx: Context,
        height: Option<u64>,
        order_tx_hashes: &[Hash],
    ) -> ProtocolResult<()>;

    async fn get_tx_count_by_address(&self, ctx: Context, address: H160) -> ProtocolResult<usize>;

    fn get_tx_from_mem(&self, ctx: Context, tx_hash: &Hash) -> Option<SignedTransaction>;
    fn set_args(&self, context: Context, state_root: MerkleRoot, gas_limit: u64, max_tx_size: u64);
}
```
Through the adaptation of the classic adapter design pattern, Axon effectively reduces the coupling between its sub-modules. This approach resolves challenges regarding interdependencies, maintenance complexity, and testing difficulties encountered in the development process.
