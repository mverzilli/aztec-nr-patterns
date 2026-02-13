---
number: 13
name: State Variable Hierarchy
confidence: 2
contains: [14, 15]
contained_by: [11]
tags: [storage, state, traits, hierarchy]
---

The dual-execution model demands different storage semantics for public and private state, yet contracts need a uniform way to declare and organize their storage. Without a shared abstraction, each state variable type would require its own initialization and slot-assignment logic.

## Context

All state variables implement the `StateVariable<N, Context>` trait, where `N` is the number of storage slots consumed. This trait provides a `storage` method that accepts a context and a base slot, returning an initialized instance. The `#[storage]` macro walks the storage struct's fields, assigns sequential slot numbers starting from 1, and calls `storage()` on each.

The hierarchy splits into two families:

**Public state variables** store data directly in the public state tree:
- `PublicMutable<T>` -- read/write current state
- `PublicImmutable<T>` -- write-once, readable from private (historical)
- `DelayedPublicMutable<T>` -- scheduled updates, readable from private

**Private state variables** manage collections of notes in the note hash tree:
- `PrivateMutable<Note>` -- single note, replace-to-update
- `PrivateSet<Note>` -- multiple notes, insert/remove
- `PrivateImmutable<Note>` -- single note, write-once

**Composite types** add structure on top:
- `Map<K, V>` -- key-value mappings with derived slots (see **Storage Slot Derivation**)
- `Owned<V>` -- per-owner instances of private state (see **Owned State Containers**)

## Therefore

Declare all contract state in a `#[storage]` struct using the provided state variable types. Choose the variable type based on two axes: public vs. private, and mutability semantics. The framework handles slot assignment, initialization, and context-appropriate method availability automatically.

## Consequences

Developers do not manually manage storage slots or initialization. The trait hierarchy ensures that new state variable types can be added to the framework without changing the storage macro. The cost is that developers must learn the type menu, but the names are descriptive and the compiler prevents misuse.
