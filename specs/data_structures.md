# Data Structures

This document provides a complete description of the data structures used.
Structures are serialized/deserialized from first row to last row, in [internal byte order](#internal-byte-order).

## Block

| bytes | name | type | description |
|-|-|-|-|
| 108 | header | [block header](#block-headers) | The block header, which contains Merkle roots and other important information. |
| 8 | numDeposits | uint64 | The number of deposit transactions (transactions that come directly from deposits on the main chain, which have zero inputs). |
| variable | deposits | [transaction](#transactions) | List of deposit transactions. |
| 8 | numTransactions | uint64 | The number of transactions. |
| variable | transactions | [transaction](#transactions) | List of transactions. |

## Block Headers

| bytes | name | type | description |
|-|-|-|-|
| 4 | version | uint32 | Block version number, may be used for different consensus rules. Current version is `1`. |
| 32 | prev | [digest](#hashing) | Hash of previous block's header. |
| 32 | depositsRoot | [digest](#hashing) | Merkle root of deposit transactions. |
| 32 | transactionsRoot | [digest](#hashing) | Merkle root of transactions. |
| 8 | height | uint64 | Block height. Genesis block is at height `0`. |

## Transactions

| bytes | name | type | description |
|-|-|-|-|
| variable | txData | [transaction data](#transaction-data) | Transaction data (data minus witnesses). |
| 4 | numWitnesses | uint32 | Number of witnesses (digital signatures). |
| variable | witnesses | [witness](#witnesses) | List of witnesses |

## Transaction Data

| bytes | name | type | description |
|-|-|-|-|
| 4 | version | uint32 | Transaction version number, may be used for different consensus rules. Current version is `1`. |
| 4 | numInputs | uint32 | Number of inputs to transaction. |
| variable | inputs | [input](#inputs) | List of inputs. |
| 4 | numOutputs | uint32 | Number of outputs from transaction. |
| variable | outputs | [output](#outputs) | List of outputs. |
| 8 | heightMin | uint64 | Minimum block height which can include this transaction. Set to `0x0000000000000000` to disable. |
| 8 | heightMax | uint64 | Maximum block height which can include this transaction. Set to `0xFFFFFFFFFFFFFFFF` to disable. |
| 8 | recentBlockHeight | uint64 | Height of a recent block. Optional, set to `0x0000000000000000` to disable. |
| 32 | recentBlockHash | [digest](#hashing) | Optional. If recentBlockHeight is non-zero, hash of block at recentBlockHeight. May be used for consensus rules to exclude transactions that commit to a different chain. |

## Inputs

| bytes | name | type | description |
|-|-|-|-|
| 36 | outpoint | [outpoint](#outpoints) | Outpoint (transaction + output index) to spend. |
| 4 | witnessIndex | uint32 | Index of witness for this input. |

## Outputs

| bytes | name | type | description |
|-|-|-|-|
| 4 | index | uint32 | Ouput index. |
| 20 | recipient | [address](#addresses) | Recipient of this output. |
| 32 | amount | uint256 | Number of coins. |
| 1 or 21 | color | [color](#colors) | If the coin represents a colored coin (in this case, a token). |

## Witnesses

Each witness signs the [hash](#hashing) of the [transaction data](#transaction-data).
Transaction data therefore only needs to be hashed once, eliminating the quadratic hashing problem in Bitcoin.
Each input indicates which witness it refers to, allowing for space savings if more than one input consumed by a transaction belong to the same address.

## Outpoints

| bytes | name | type | description |
|-|-|-|-|
| 32 | txid | [digest](#hashing) | Originating transaction. |
| 4 | index | uint32 | Output index. |

## Addresses

Addresses are the last 20 bytes of the [hash](#hashing) of the public key.

## Colors

Coin coloring can be represented in two unique ways.
If uncolored:

| bytes | name | type | description |
|-|-|-|-|
| 1 | isColored | bool | `0x00` |

If colored:

| bytes | name | type | description |
|-|-|-|-|
| 1 | isColored | bool | `0x01` |
| 20 | id | [address](#addresses) | Address of main chain token contract. |

## Internal Byte Order

Big-endian byte order is used for any serialization/deserialization.

## Digital Signature Scheme

The chosen curve is [secp256k1](https://en.bitcoin.it/wiki/Secp256k1) (identically to Bitcoin and Ethereum).
Deterministic signatures ([RFC-6979](https://tools.ietf.org/rfc/rfc6979.txt)) are used.
An extra byte appended to the end of signatures indicates parity, allowing recovery (identically to Ethereum).

## Hashing

All cryptographic hashing is done with [Keccak-256](https://keccak.team/keccak.html) **not** SHA3-256 ([FIPS 202](https://keccak.team/specifications.html#FIPS_202)).
At a later date, support for different or multiple hashing functions may be considered.
Digests are 256 bits (32 bytes) in length.
