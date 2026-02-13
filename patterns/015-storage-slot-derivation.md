---
number: 15
name: Storage Slot Derivation
confidence: 2
contains: []
contained_by: [13]
tags: [storage, hashing, slots, map]
---

A mapping from keys to values needs a way to assign unique storage slots to each key-value pair. Using sequential numbering would require knowing all keys in advance, which is impossible for open-ended mappings like user balances.

## Context

The `Map<K, V>` state variable uses hash-based slot derivation: for a map with base slot `s` and key `k`, the derived slot is `poseidon2_hash([s, k])`. This produces collision-free slot assignments without coordination, because Poseidon2 is a cryptographic hash function.

Maps can nest: `Map<A, Map<B, PublicMutable<T>>>` derives slots as `poseidon2_hash([poseidon2_hash([base, a]), b])`. This enables multi-dimensional lookups (e.g., allowances keyed by `(owner, spender)`) with the same collision-resistance guarantees.

The `Owned<V>` wrapper uses the same derivation internally when computing per-owner slots. The key type must implement `ToField` (convertible to a single field element), which is satisfied by `AztecAddress`, `Field`, and integer types.

## Therefore

Use `Map<K, V>` for any key-indexed public or private state. Nest maps for multi-key lookups. Trust the hash-based derivation to produce unique slots without manual coordination. The base slot is auto-assigned by the `#[storage]` macro; all per-key slots are deterministically derived from it.

## Consequences

Slot derivation is invisible to the developer in normal usage. The main design consequence is that map entries cannot be enumerated -- there is no way to iterate over all keys, because the slots are hash-derived and sparse. This matches Solidity's mapping semantics and is a fundamental trade-off of hash-based slot allocation.
