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

### `hash(T)`

Compute the Keccak-256 hash of the binary input `T`.

### `len(T[])`

Return length of list.

### `merkle(T[])`

Compute the Merkle root of the list of items `T[]`.
If there are an odd number of items on the level, the last one is duplicated.

### `to_outpoints(transaction)`

Return list of tuples of (outpoints and tuples of (recipients and amounts), one for each output in the transaction.

### `ecrecover(sig, msg)`

Return address corresponding to pubkey of signature `sig` of message `msg`.

### `sizeof(transaction)`

Return the size in bytes of the transaction `transaction`.

## Block Validity

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
1. TODO verify that deposit comes from main chain, and recipients match up.

Verify that deposits are lexicographically ordered in ascending order of deposit id: `hash(deposit.txData)`.

### Validate Transactions

Verify that `block.numTransactions == len(block.transactions)`.

Initialize `block_inputs` as an empty list.

For each `tx` in `block.transactions[1:]`:
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

Verify that transactions other than the first are lexicographically ordered in ascending order of transaction id: `hash(tx.txData)`.

For `block.transactions[0]` (coinbase transaction):
1. Verify that `len(block.transactions[0].txData.inputs) == 0`.
1. Verify that sum of outputs in `block.transactions[0]` `==` sum of transaction fees for `block.transactions[1:]`. Fees collected for each `tx` in `block.transactions[1:]` are `block.header.feePerByte * sizeof(tx)` if `tx.txData.maxFeePerByte != 0`, `0` otherwise.

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
