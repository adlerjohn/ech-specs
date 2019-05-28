# Consensus

This document provides a complete description of all consensus rules, for both block validity and fork choice.

## Constants

TODO

## Variables

### `block`

Denotes the current block.
Nested member variables are found in the [data structures](./data_structures.md) document.

### `head`

Denotes the block at the current head of the chain.

### `state`

Denotes the UTXO set.
TODO define helper functions and state better.

## Helper functions

### `hash(T)`

Compute the Keccak-256 hash of the binary input `T`.

### `id(block)`

Return `hash(block.header)`.

### `len(T[])`

Return length of list.

### `merkle(T[])`

Compute the Merkle root of the list of items `T[]`.
If there are an odd number of items on the level, the last one is duplicated.

### `to_outpoints(transaction)`

Return list of tuples of outpoints and recipients, one for each output in the transaction.

### `ecrecover(sig, msg)`

Return address corresponding to pubkey of signature `sig` of message `msg`.

## Block Validity

### Previous Link

Verify that `block.prev == id(head)`.

### Height

Verify that `block.height == head.height + 1`.

### Deposits Root

Verify that `block.header.depositsRoot == merkle(block.deposits)`.

### Transactions Root

Verify that `block.header.transactionsRoot == merkle(block.transactions)`.

### Validate Deposits

Verify that `block.numDeposits == len(block.deposits)`.

For each `deposit` in `block.deposits`:
1. Verify that `len(deposit.txData.inputs) == 0`.
1. For each `(outpoint, recipient)` in `to_outpoints(deposit)`:
     1. Execute `state.insert(hash(outpoint), recipient)`.

TODO verify that deposit comes from main chain, and recipients match up

### Validate Transactions

Verify that `block.numTransactions == len(block.transactions)`.

For each `tx` in `block.transactions`:
1. For each `input` in `tx.txData.inputs`:
     1. Compute `h = hash(tx.txData)`.
     1. Compute `witness = tx.witnesses[input.witnessIndex]`.
     1. Verify that `state[input] == ecrecover(witness, h)` (only spend unspent owned coins).
1. For each `(outpoint, recipient)` in `to_outpoints(tx)`:
     1. Execute `state.insert(hash(outpoint), recipient)` (prevents double spends).

Verify that, for each color, sum of amounts in inputs `<=` sum of amounts in outputs.

Verify that transactions are lexicographically ordered in ascending order of transaction id `hash(tx.txData)`.

TODO additional rules for ordering witnesses, inputs, and outputs

## Fork Choice Rule

TODO
