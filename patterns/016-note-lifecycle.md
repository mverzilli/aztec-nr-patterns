---
number: 16
name: Note Lifecycle
confidence: 2
contains: []
contained_by: [2, 5]
tags: [notes, lifecycle, create, destroy, nullify]
---

Notes are the atoms of private state, but they have a non-trivial lifecycle: creation, retrieval, and destruction each involve different cryptographic operations and different interactions with the kernel and PXE.

## Context

The note lifecycle has three phases:

**Creation** (`create_note`):
1. Generate random `randomness` value for the note
2. Compute the note hash from packed fields, owner, slot, and randomness
3. Notify the PXE via oracle so it can track the note locally
4. Push the note hash to the kernel (which will silo it and commit to the note hash tree)
5. Return a `NoteMessage` for delivery (see **Explicit Delivery**)

**Retrieval** (`get_note` / `get_notes`):
1. Query the PXE oracle for notes matching the storage slot and owner
2. Oracle returns note data from its local database
3. Verify each note's existence via tree membership proof (the kernel checks this)
4. Return `ConfirmedNote` objects containing both the note and its hash

**Destruction** (`destroy_note`):
1. Retrieve the `ConfirmedNote` to be destroyed
2. Compute the nullifier from the note hash and the owner's nullifier secret key
3. Push the nullifier to the kernel (which will silo it and commit to the nullifier tree)
4. The note is now spent -- future retrieval attempts will exclude it

The `#[note]` macro generates the `NoteHash` implementation that computes hashes and nullifiers, and the `NoteProperties` implementation that enables filtering during retrieval.

## Therefore

Use `create_note`, `get_note`/`get_notes`, and `destroy_note` (or the state variable methods that wrap them) for all private state operations. Do not attempt to manually insert note hashes or nullifiers. The framework ensures that the cryptographic operations are performed correctly and that the kernel receives properly formatted data.

## Consequences

The lifecycle abstraction hides significant cryptographic complexity. Developers work with note structs rather than hashes and proofs. The cost is that "updating" private state always means destroying old notes and creating new ones -- there is no in-place mutation. This is inherent to the UTXO model and shapes how developers think about state transitions.
