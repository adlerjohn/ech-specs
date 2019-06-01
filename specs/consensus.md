# Consensus

This document provides a complete description of all consensus rules, for both block validity and fork choice.

## Constants

| name | value | description |
|-|-|-|
| `MAX_INPUTS` | `2**4` (= 16) | Maximum number of inputs in a transaction. |
| `MAX_OUTPUTS` | `2**4` (= 16) | Maximum number of outputs in a transaction. |

## Variables

### `block`

The current block that is being validated.
Nested member variables are found in the [data structures](./data_structures.md) document.

### `state`

The chain state (UTXO set).

TODO define helper functions and state better, especially with state commitments.

## Helper functions

### `ecrecover(sig, msg)`

Return address corresponding to pubkey of signature `sig` of message `msg`.

### `hash(T)`

Compute the Keccak-256 hash of the binary input `T`.

### `header_at(height)`

Return block header at height `height`.

### `len(T[])`

Return length of list.

### `merkle(T[])`

Compute the Merkle root of the list of items `T[]`.
If there are an odd number of items on the level, the last one is duplicated.

### `sizeof(transaction)`

Return the size in bytes of the transaction `transaction`.

### `to_outpoints(transaction)`

Return list of tuples of (outpoints and tuples of (recipients and amounts), one for each output in the transaction.

## Block Validity

This section defines block validity rules.
A block that does not pass a verification check listed in this section is invalid.
A block that passes all verification checks listen in this section is valid, but may not be included in the canonical chain, depending on application of the [fork choice rule](#fork-choice-rule).

### Inclusion on Parent Chain

Verify that `block.header` is included as a receipt in the form of a log from the [deposit contract](./deposit_contract.md) for the linked parent chain block.

### Previous Link

Verify that a block with ID `block.header.prev` exists (ID is the hash of the block header), and assign the previous block header to `prevHeader`.

Verify that `block.header.height == prevHeader.height + 1`.

### Deposits Root

Verify that `block.header.depositsRoot == merkle(block.deposits)`.

### Transactions Root

Verify that `block.header.transactionsRoot == merkle(block.transactions)`.

### Validate Deposits

Verify that `block.numDeposits == len(block.deposits)`.

For each `deposit` in `block.deposits`:
1. Verify that `len(deposit.txData.inputs) == 0`.
1. Verify that `len(deposit.txData.outputs) <= MAX_OUTPUTS`.
1. Verify that `deposit` is a deposit from the [deposit contract](./deposit_contract.md) for the linked parent chain block.

For each `deposit_receipt` from the linked parent chain block and all parent chain blocks since `block.header.prev`'s linked parent chain block:
1. Verify that `deposit_receipt` is in `block.deposits` (mandatory deposit processing).

Verify that deposits are lexicographically ordered in ascending order of deposit id: `hash(deposit.txData)`.

### Validate Transactions

Verify that `block.numTransactions == len(block.transactions)`.

Initialize `block_inputs` as an empty list.

For each `tx` in `block.transactions[1:]`:
1. Verify that `len(tx.txData.inputs) <= MAX_INPUTS`.
1. Verify that `len(tx.txData.outputs) <= MAX_OUTPUTS`.
1. For each `input` in `tx.txData.inputs`:
     1. Compute `h = hash(tx.txData)`.
     1. Compute `witness = tx.witnesses[input.witnessIndex]`.
     1. Verify that `state[input] == ecrecover(witness, h)` (only spend unspent owned coins).
     1. Verify that `input not in block_inputs`.
     1. Compute `block_inputs.append(input)` (no double spending within block).
1. Verify that there are no duplicate witnesses, that each subsequent input references a monotonically increasing witness index, and that all witnesses are referenced at least once.
1. Verify that `tx.txData.maxFeePerByte >= block.header.feePerByte` (transaction fee rate is high enough for inclusion) or `tx.txData.maxFeePerByte == 0` (no-free transactions are acceptable).
1. Verify that `tx.txData.outputs[0] >= tx.txData.maxFeePerByte * sizeof(tx)` (change output must have enough to pay for max fees).
1. Verify that, for each color, sum of amounts in inputs `==` sum of amounts in outputs (coins can't be created or destroyed, only transferred from one owner to another).
1. Verify that `tx.txData.heightMin <= block.header.height <= tx.txData.heightMax`.
1. If `tx.txData.recentBlockHeight != 0`:
    1. Verify that `tx.txData.recentBlockHeight < block.header.height`.
    1. Verify that `tx.txData.recentBlockHash == header_at(tx.txData.recentBlockHeight).height`.

Verify that transactions other than the first are lexicographically ordered in ascending order of transaction id: `hash(tx.txData)`.

For `coinbase = block.transactions[0]` (coinbase transaction):
1. Verify that `len(coinbase.txData.inputs) == 0`.
1. Verify that `len(coinbase.txData.outputs) == 1`.
1. Verify that `coinbase.txData.outputs[0].color == 0x00` (coinbase output is for native coin).
1. Verify that `coinbase.txData.outputs[0].amount ==` sum of transaction fees for `block.transactions[1:]`. Fees collected for each `tx` in `block.transactions[1:]` are `block.header.feePerByte * sizeof(tx)` if `tx.txData.maxFeePerByte != 0`, `0` otherwise.

TODO block reward maturity (tied to Ethereum blocks)

## State Transition

For each `deposit` in `block.deposits`:
1. For each `(outpoint, (recipient, amount))` in `to_outpoints(deposit)`:
    1. Execute `state.insert(hash(outpoint), (recipient, amount))`.

For each `tx` in `block.transactions`:
1. For each `input` in `tx.txData.inputs`:
    1. Execute `state.erase(input)`.
1. For each `(outpoint, (recipient, amount))` in `to_outpoints(tx)`:
    1. Execute `state.insert(hash(outpoint), (recipient, amount))`.

## Fork Choice Rule

TODO
