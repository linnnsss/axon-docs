---
title: Interoperability
hide_title: true
sidebar_position: 8
---

import useBaseUrl from "@docusaurus/useBaseUrl";

:::tip

Before proceeding, we recommend reading the following pieces to fully grasp this article, as they provide prerequisite knowledge:

- [Axon Fundamentals](https://docs.axonweb3.io/fundamentals)
- [Image Cell System Contract](https://docs.axonweb3.io/getting-started/for-contributor/icsc)

:::

This article introduces the design of Axon’s interoperable transaction module. 
Let’s start with a brief recap on interoperability. Interoperability, in the realm of blockchain, refers to the ability of blockchains to communicate with one another, thereby enabling a blockchain to access data from and write data to other blockchains. However, a blockchain is akin to a computer devoid of an internet connection. It lacks the innate capabilities to communicate with other blockchains or [external API](https://blog.chain.link/understanding-how-data-and-apis-power-next-generation-economies/), commonly known as the [oracle problem](https://encyclopedia.pub/entry/2959). Interoperability provides a powerful solution to this issue, enabling seamless integration of various blockchain systems, making it possible to create complex, decentralized applications that leverage the benefits of multiple blockchains.

Axon is an app-chain[^1] framework that natively supports interoperability. It is compatible with the Inter-Blockchain Communication (IBC) protocol, thereby enabling itself to communicate with any other IBC compatible chains. Axon also involves CKB-VM, a virtual machine used in CKB that is based on [RISC-V](https://riscv.org/) instruction set. Combined with the [Image Cell System Contract (ICSC)](https://docs.axonweb3.io/getting-started/for-contributor/icsc), Axon can execute any scripts deployed on CKB. This significantly enhances the interoperability of Axon and makes abstraction possible under the account model and EVM.

## Interoperable Transaction Outlined
As it is known, Ethereum uses the Elliptic Curve Digital Signature (ECDSA) with secp256k1 curve to sign transactions. However, transactions on Axon are expected to be verified by a broader range of curves or even more intricate logic. This challenge is addressed by the built-in CKB-VM module included in the interoperable transaction verifier.

<img src={useBaseUrl("img/for-contributors/interoperability axon.png")}/>

The image above illustrates the interplay among the components involved in the interoperable transaction. If you find it difficult to understand them all at once, don’t worry. The following discussion will only focus on Axon.

In brief, all of the verification processes (arrows in green color), except the hardcoded secp256k1, can be compiled to RISC-V binary and deployed in CKB, the Layer 1, then the code cell will be synchronized to ICSC, and CKB-VM module can load the binary from ICSC and verify the transaction. This process might appear laborious, however, the advantages shown below are worth the effort:

1. Binaries deployed in CKB can be upgraded easily without breaking the compatibility of Axon. It only needs to change the reference to the script deployed cell, which is included in transaction in ICSC when a logic upgrade occurs.
2. The *max_cycles* argument of CKB-VM provides a robust guarantee against dead loops. According to the mechanism of CKB-VM, each instruction execution consumes some [cycles](https://docs.nervos.org/docs/basics/glossary#cycles) and if the sum of used cycles reaches the set cycle limit, the program will return directly.
3. The verification can reference the state in CKB Layer 1. CKB cells can not only store script binaries, but record information as well. Axon can read the data in CKB smoothly if it considers ICSC as an oracle.

## Signature Verification
Axon uses a redefined signature component based on Ethereum to ensure compatibility. This component consists of two numbers (uint256): `r` and `s`, as well as a recovery identifier variable (uint8) called ECDSA. Additionally, the signature verified through CKB-VM is referred to as the Interoperability Signature (IOPSA).

### Signature Structure

The `r` and `s` fields employ the `Bytes` type to enhance the information containment capabilities of Axon transactions. ECDSA and IOPSA are distinguished by the signature length. Specifically, the ECDSA signature length is 65 bytes; the IOPSA signature length must be greater than 65.

IOPSA can be used in two ways: verify in CKB-VM by mock transactions and call CKB-VM directly.

Below is a comparison of Axon’s signature structure with Ethereum:

<table>
<tr>
<th> Axon Signature Structure </th>
<th> Ethereum Signature Structure </th>
</tr>
<tr>
<td>

```rust
pub struct SignatureComponent {
		pub r: Bytes,
		pub s: Bytes,
		pub v: u8,
}
```

</td>
<td>

```rust
pub struct SignatureComponent {
		pub r: U256,
		pub s: U256,
		pub v: u8,
}
```

</td>
</tr>
</table>

### Verify by Calling CKB-VM
To verify a variety of curves or algorithms, a simple and efficient method is calling CKB-VM. For the security of the asset, this approach only allows the verification of other digital signatures encoded in a whitelist, such as [EdDSA](https://en.wikipedia.org/wiki/EdDSA), [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)#Signing_messages), [BLS](https://en.wikipedia.org/wiki/BLS_digital_signature). In this mode, the r field in signature component includes the following information:
```rust
pub struct CellDepWithPubKey {
		pub cell_dep: CellDep,
		pub pub_key: Bytes,
}

pub struct CellDep {
		pub out_point: OutPoint,
		pub dep_type: u8,
}

pub strcut OutPoint {
		pub tx_hash: H256,
		pub index: u32,
}
```
`CellDep` references to a live cell, while the `s` field represents the signature. Interoperable transaction verifier finds the cell where the script is deployed via `cell_dep` and loads the cell data as binary. It then executes the binary by calling the CKB-VM [API](Interoperable transaction verifier finds the cell where the script is deployed via cell_dep and loads), taking the public key and signature as input arguments. The order of arguments is fixed, with the public key being the first, followed by the signature. The verification of sender address is similar to Ethereum, and the sender should be the last 20 bytes of the public key [keccak256](https://github.com/ethereum/eth-hash) hash.

### Verify by Mock Transaction
Verify by mock transaction is more complex compared to calling CKB-VM. Axon first decodes the signature component, constructs a mock [CKB transaction](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0002-ckb/0002-ckb.md#44-transaction), and then verifies the transaction in CKB transaction verifier. Unlike calling CKB-VM verification, this method can leverage CKB-VM [syscalls](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0009-vm-syscalls/0009-vm-syscalls.md) to enhance script processing capabilities. By accessing information stored in cells through syscalls, the verification scripts can access more information in CKB, thereby greatly enhancing their processing capabilities. So far, Axon interoperability verifier can read all of the data stored in CKB and establish a bridge between the abstract accounts[^2] in CKB and Axon. 

Mock transaction verification offers two modes depending on the type of cells taken as transaction input:

- Reality input mode: Use the existing cells in CKB as the input of the mock transaction to verify.
- Dummy input mode: User input an inexistent cell that will be taken as the input of the mock transaction to verify.

#### Reality Input Mode Data Structure
In reality mode, the `r` field in signature component includes the following information:
```rust
pub struct CKBTxMockByRef {
    pub cell_deps:             Vec<CellDep>,
    pub header_deps:           Vec<H256>,
    pub out_points:            Vec<OutPoint>,
    pub out_point_addr_source: AddressSource,
}

pub struct AddressSource {
    pub type_: u8,
    pub index: u32,
}
```
- `AddressSource`: The source used to calculate the sender address.
    - `type_`: An `uint8` that indicates which hash will be used to compute the sender. `0` represents lock script hash; `1` represents type script hash.
    - `index`: indicates that the sender is computed from the input at the specified index[^3].
- `HeaderDeps`: A list of references to the CKB block header. `HeaderDeps` is similar to `CellDeps`, since they both provide some read-only data in a transaction.
- `OutPoint`: A reference of the input cell.

The s field in signature component includes the following information:
```rust
pub struct Witnesses {
    pub inner: Vec<Witness>,
}

pub struct Witness {
    pub input_type:  Option<Bytes>,
    pub output_type: Option<Bytes>,
    pub lock:        Option<Bytes>,
}
```

The `Witnesses` are the information provided by the transaction creator to ensure the successful execution of the corresponding lock script. For example, signatures might be included to ensure that a signature verification lock script passes.

#### Dummy Input Mode Data Structure

In this mode, the `r` field in the signature component includes the following information:
```rust
pub struct CKBTxMockByRefAndOneInput {
    pub cell_deps:             Vec<CellDep>,
    pub header_deps:           Vec<H256>,
    pub input_cell_with_data:  CellWithData,
    pub out_point_addr_source: AddressSource,
}

pub struct CellWithData {
    pub type_script: Option<Script>,
    pub lock_script: Script,
    pub data:        Bytes,
}
```
The `CellWithData` object is the input dummy cell. It includes all information required by a cell except for `capacity`. Capacity is the size of the cell, which Axon will compute according to the input. In this mode, the `index` field in `AddressSource` must be `0`.

The `s` field in the signature component is the same as the reality input mode, which restricts the witness to only include one item.

#### Output Cell

Each CKB transaction must have at least one output cell. The output cell in an interoperable transaction has the following structure:
```json
{
    "capacity": "Compute by Axon", 
    "lock": {
        "code_hash": "0x0000000000000000000000000000000000000000000000000000000000000000", 
        "args": "0x", 
        "hash_type": "data"
    }, 
    "type": {
        "code_hash": "ckb_blake2b_256(AlwaysSuccessScript)", 
        "args": "Axon transaction signature hash", 
        "hash_type": "data1"
    }
}
```

Because of the `lock` script in output cells is not executed, all fields are `0`. The `type` script is an always-success script, and the script argument is the Axon transaction signature hash which is specified in [EIP-155](https://eips.ethereum.org/EIPS/eip-155). The `capacity` is calculated by the following rules:

- In reality input mode, capacity is the sum of the input’s capacity minus one.
- In dummy input mode, capacity is `100`.

#### Mock CKB Transaction Structure

After decoding the `r` and `s` field in signature component, Axon builds a mock CKB transaction with the following structure:
```json
{
    "version": "0x0", 
    "cell_deps": "Decode from signature r field", 
    "header_deps": "Decode from signature r field", 
    "inputs": "Decode from signature r field", 
    "outputs": [
        "Mock by Axon", 
		],
    "outputs_data": [
        "0x", 
    ], 
    "witnesses": "Decode from signature s field"
}
```
### Signature Decode

The first byte of the `r` field is used as a flag to determine the verification mode. 

- `0`: call CKB-VM verify mode
- `1`: reality input mode of verification by mock transaction
- `2`: dummy input mode of verification by mock transaction

Any other flag values are considered invalid. The following code demonstrates the different modes of verification used by Axon:

```rust
match signature.r[0] {
		0u8 => {
				let sig_r = rlp::decode::<CellDepWithPubKey>(&signature.r[1..])?;
				...
		}
		1u8 => {
				let sig_r = rlp::decode::<CKBTxMockByRef>(&signature.r[1..])?;
				...
		}
		2u8 => {
				let sig_r = rlp::decode::<CKBTxMockByRefAndOneInput>(& signature.r[1..])?;
				...
		}
		_ => return Err(InvalidFlag);
}
```

After the verification mode is determined, the type of `s` field can be confirmed. If it is calling CKB-VM verification mode, take the `s` field as signature directly. Otherwise, the type of `s` should be decoded as a `Witnesses` object.

```rust
let witnesses = rlp::decode::<Witnesses>(&signature.s)?;
```
Therefore, the encode of interoperability transaction signature component is the reverse of the above process. The encode functions are provided in [Axon SDK](https://github.com/axonweb3/axon-sdk-js).

## Interoperability in EVM

To expand the interoperability of EVM, Axon provides some precompile contracts to call CKB-VM. The process that calls CKB-VM in EVM is similar to that in interoperability transaction, they both depend on the same implementation.

### Precompile Contracts

🚧 WIP. WE WILL UPDATE SOON! 🚧




## Footnotes
[^1]: Axon contains two types of contracts: general contracts and system contracts. The main difference is that system contracts are written in Rust only. Compared with general contracts, system contracts can invoke more system resources, such as storage. Besides, system contracts are not necessarily stored in EVM MPT, since they have their own storage space.
[^2]: Abstract accounts are blockchain user accounts implemented as smart contracts that allow for a high degree of customization. This is because they can contain diverse types of logic and implement different flows. By shifting user authentication from the network to the smart contracts, abstract accounts empower wallet designers to determine how to authenticate their users.
[^3]: CKB input usually contains multiple cells, whose serial numbers are represented via index. For example, index is 0 stands for the first cell; 1, the second.
