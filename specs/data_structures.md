# Data Structures

## Block

| bytes | name | type | description |
|-|-|-|-|
| 108 | headers | [block header](#block-headers) | TODO |
| 8 | numDeposits  | uint64 | TODO |
| variable | deposits | [transaction](#transactions) | TODO |
| 8 | numTransactions | uint64 | TODO |
| variable | transactions | [transaction](#transactions) | TODO |

## Block Headers

| bytes | name | type | description |
|-|-|-|-|
| 4 | version | uint32 | TODO |
| 32 | prev  | [digest](#hashing) | TODO |
| 32 | depositsRoot | [digest](#hashing) | TODO |
| 32 | transactionsRoot | [digest](#hashing) | TODO |
| 8 | height | uint64 | TODO |

## Transactions

| bytes | name | type | description |
|-|-|-|-|
| 4 | version | uint32 | TODO |
| 4 | numInputs | uint32 | TODO |
| variable | inputs | [input](#inputs) | TODO |
| 4 | numOutputs | uint32  | TODO |
| variable | outputs | [output](#outputs) | TODO |
| 8 | heightMin | uint64 | TODO |
| 8 | heightMax | uint64 | TODO |
| 8 | recentBlockHeight | uint64 | TODO |
| 32 | recentBlockHash | [digest](#hashing) | TODO |

## Inputs

| bytes | name | type | description |
|-|-|-|-|
| 36 | outpoint | [outpoint](#outpoint) | TODO |
| 4 | witnessIndex | uint32 | TODO |


## Outputs

| bytes | name | type | description |
|-|-|-|-|
| 4 | index | uint32 | TODO |
| 20 | recipient | [address](#addresses) | TODO |
| 32 | amount | uint256 | TODO |
| 1 or 21 | color | [color](#colors) | TODO |

## Outpoints

| bytes | name | type | description |
|-|-|-|-|
| 32 | txid | [digest](#hashing) | TODO |
| 4 | index | uint32 | TODO |

## Addresses

Addresses are the last 20 bytes of the [hash](#hashing) of the public key.

## Colors

Coin coloring can be represented in two unique ways.
If uncolored:

| bytes | name | type | description |
|-|-|-|-|
| 1 | isColored | bool | TODO |

If colored:

| bytes | name | type | description |
|-|-|-|-|
| 1 | isColored | bool | TODO |
| 20 | id | [address](#addresses) | TODO |

## Internal Byte Order

Big-endian byte order is used for any serialization/deserialization.

## Digital Signature Scheme



## Hashing

All cryptographic hashing is done with [Keccak-256](https://keccak.team/keccak.html) **not** SHA3-256 ([FIPS 202](https://keccak.team/specifications.html#FIPS_202)).
At a later date, support for different or multiple hashing functions may be considered.
Digests are 256 bits (32 bytes) in length.
