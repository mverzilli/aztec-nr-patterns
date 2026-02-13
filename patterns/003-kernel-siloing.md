---
number: 3
name: Kernel Siloing
confidence: 2
contains: []
contained_by: []
tags: [security, isolation, hashing, kernel]
---

If two contracts independently create notes with the same hash, or if a malicious contract can produce nullifiers that collide with another contract's notes, the integrity of private state collapses.

## Context

Aztec's kernel circuit automatically **silos** note hashes, nullifiers, and storage keys by mixing in the originating contract's address. Before a note hash is committed to the global tree, the kernel computes `hash(contract_address, note_hash)`. Similarly, nullifiers are siloed before entering the nullifier tree.

This siloing happens outside the application circuit -- contracts themselves work with "inner" (unsiloed) hashes, while the kernel produces the "siloed" versions that appear on-chain. The `ConfirmedNote` type carries both the inner hash (for the contract's own logic) and the siloed hash (for tree membership proofs).

## Therefore

Rely on kernel siloing as an implicit security boundary rather than implementing contract-level isolation manually. Contract code computes note hashes and nullifiers from its own data; the kernel ensures these cannot collide with or be confused for another contract's values. When verifying note existence, use the siloed hash for tree proofs and the inner hash for application logic.

## Consequences

Developers do not need to include contract addresses in their note hash computations -- the kernel does this automatically. This simplifies note definitions and prevents a class of cross-contract attacks. However, it means that a note's on-chain hash differs from its in-contract hash, which surfaces when working with `ConfirmedNote` objects that expose both representations.
