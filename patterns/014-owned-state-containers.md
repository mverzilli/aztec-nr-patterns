---
number: 14
name: Owned State Containers
confidence: 2
contains: []
contained_by: [2, 13]
tags: [ownership, state, notes, per-user]
---

Private state variables hold notes that belong to specific owners, but the storage system assigns a single base slot per field. Without a way to scope private state by owner, every user's notes would share the same storage slot, making retrieval inefficient and semantics unclear.

## Context

The `Owned<V, Context>` wrapper provides per-owner instances of private state variables. Declaring `balances: Owned<PrivateSet<TokenNote, Context>, Context>` in the storage struct creates a single base slot; calling `self.storage.balances.at(owner_address)` derives an owner-specific slot and returns a `PrivateSet` scoped to that owner.

Under the hood, `Owned` uses the same hash-based slot derivation as `Map` (see **Storage Slot Derivation**), but it is semantically distinct: `Owned` is specifically for private state variables that require an owner context. This distinction matters because private note operations need an owner address for note hash computation, nullifier derivation, and encrypted delivery.

The `at(owner)` method does not merely derive a slot -- it also configures the returned state variable with the owner's address, which flows into note creation and destruction operations.

## Therefore

Use `Owned<V>` for any private state that is logically partitioned by owner. The most common patterns are `Owned<PrivateSet<Note>>` for collections (like token balances) and `Owned<PrivateMutable<Note>>` for single-value state (like a user's nonce). Access the owner-specific instance with `.at(owner_address)` before performing reads or writes.

## Consequences

The ownership model maps naturally to the UTXO approach: each owner's notes are logically grouped, and the framework knows which address to use for nullifier computation and delivery. The trade-off is that `Owned` is an additional wrapper type that developers must use for private-but-per-owner state, but attempting to use a bare `PrivateSet` in storage will produce a compile error, guiding developers toward the correct pattern.
