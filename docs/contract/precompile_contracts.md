---
title: Precompile Contracts
hide_title: true
sidebar_position: 2
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Precompile Contracts

On top of a set of opcodes, EVM also offers advanced functionalities through precompiled contracts. These are special contracts bundled with EVM at fixed addresses that can be called with a determined gas. The gas is purely the contractual cost, neither the cost of the call itself nor the instructions to put the parameters in memory.

Precompiled contract addresses start from 1 and increment for each contract. New hardforks may introduce new precompiled contracts. Similar to the regular contracts, new contracts are called from opcodes with instructions, such as [CALL](https://www.evm.codes/#F1). 

For all precompiled contracts, inputs that are shorter than expected are assumed to be virtually padded with zeros at the end, whereas any surplus bytes at the end of inputs that are longer than expected are ignored.

The address precompile contract is described by the last 2 bytes:

| Byte range [0..18] | Byte range [18..20] |
| --- | --- |
| 0000000000000000 | addr |

### EcRecover

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000001 | 3000 | hash, v, r, s | publicAddress |

Elliptic curve digital signature algorithm (ECDSA) public key recovery function.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### SHA2-256

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000002 | 60 | data | hash |

Hash function used in Bitcoin.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### RIPEMD-160

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000003 | 600 | data | hash |

Hash function

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### Identity

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000004 | 15 | data | data |

It returns the input.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### Modexp

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000005 | 200 | Bsize, Esize, Msize, B, E, M | value |

Arbitrary-precision exponentiation under modulo.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### EcAdd

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000006 | 150 | x1, x2, y1, y2 | x, y |

Point addition (ADD) on the elliptic curve `alt_bn128`.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### EcMul

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000007 | 6000 | x1, x2, s | x, y |

Scalar multiplication (MUL) on the elliptic curve `alt_bn128`.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### EcPairing

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000008 | 45000 | x1, y1, x2, y2, …, xk, yk | success |

Bilinear function on groups on the elliptic curve `alt_bn128`.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### Blake2f

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000009 | 0 | rounds, h, m, t, f | h |

Compression function F used in the BLAKE2 cryptographic hashing algorithm.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### Metadata

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000101 | 0 | rounds, h, m, t, f | Metadata |

Compression function F used in the BLAKE2 cryptographic hashing algorithm.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### GetHeader

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000102 | 0 | rounds, h, m, t, f | HeaderInfo |

Compression function F used in the BLAKE2 cryptographic hashing algorithm.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### GetCell

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000103 | 0 | rounds, h, m, t, f | cellInfo |

Compression function F used in the BLAKE2 cryptographic hashing algorithm.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### CallCkbVm

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000104 | 300 | rounds, h, m, t, f | data |

Compression function F used in the BLAKE2 cryptographic hashing algorithm.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>

### VerifyInCkbVm

| ADDRESS | MINIMUM GAS | INPUT | OUTPUT |
| --- | --- | --- | --- |
| 0x00000000000000000105 | 600 | rounds, h, m, t, f | result |

Compression function F used in the BLAKE2 cryptographic hashing algorithm.

<details><summary>Click here to view ABI</summary>

#### Inputs

#### Output

</details>
