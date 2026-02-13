---
number: 2
name: UTXO-Based Private State
confidence: 2
contains: [14, 16, 17]
contained_by: []
tags: [privacy, state, notes, utxo]
---

An account model for private state (like Solidity's storage slots) would require a global view of all state to verify transitions, destroying privacy. The system needs a way to represent private state that can be created, read, and destroyed without revealing its contents or existence to other users.

## Context

In Aztec, private state is represented as **notes** following the UTXO (Unspent Transaction Output) model. Each note is a discrete data object that lives in a global note hash tree. Only its hash is publicly visible; the note's contents are known only to parties who have received its encrypted payload.

A note is created by inserting its hash into the tree. A note is destroyed by publishing a **nullifier** -- a deterministic value derived from the note hash and a secret key. Because the nullifier reveals nothing about which note it destroys (without the secret), observers cannot link creation and destruction events.

This model means private state is not a single mutable slot but a **collection of notes**. A token balance, for instance, is the sum of all unspent notes owned by an address. Updating a balance means destroying old notes and creating new ones with the desired values.

## Therefore

Model all private state as collections of notes. Each note type defines how its hash and nullifier are computed (see **Note Lifecycle**). State variables like `PrivateSet`, `PrivateMutable`, and `PrivateImmutable` provide ergonomic abstractions over the underlying note operations, but the UTXO model is always present beneath. Developers choose which note type to use based on their ownership and mutability requirements.

## Consequences

The UTXO model provides strong privacy guarantees but requires a different mental model than account-based storage. Balances are inherently fragmented across multiple notes. Reading state requires discovering which notes exist (see **Privacy-Preserving State Discovery**). The framework provides the **Owned State Containers** pattern to make per-owner state management ergonomic.
