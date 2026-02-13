---
number: 28
name: Initialization Guards
confidence: 2
contains: []
contained_by: []
tags: [initialization, constructor, nullifier, safety]
---

A contract that can be initialized multiple times is vulnerable to state reset attacks. But in a privacy-preserving system, simply checking a boolean flag in public storage would leak information about the contract's initialization status.

## Context

Contracts use an **initialization nullifier** to enforce single initialization. When a function marked with `#[initializer]` executes, it emits a nullifier derived from the contract's address. Any subsequent attempt to call an initializer will try to emit the same nullifier, which the kernel rejects because the nullifier already exists in the tree.

A contract can have multiple functions marked `#[initializer]` (e.g., one for private setup, one for public setup), but they are all mutually exclusive -- whichever runs first claims the initialization nullifier. Functions not marked as initializers will fail if called before initialization, because the framework checks for the initialization nullifier's existence.

The `#[noinitcheck]` annotation opts a function out of the initialization check, useful for functions that genuinely work without initialization (e.g., utility functions).

## Therefore

Mark at least one function with `#[initializer]` in every contract. Use `#[noinitcheck]` sparingly and only for functions that are provably safe to call before initialization. If a contract needs both private and public setup, have the private initializer enqueue the public setup function using the enqueue pattern.

## Consequences

The nullifier-based approach provides privacy-preserving initialization enforcement -- the nullifier's existence proves initialization happened, but its value does not reveal which initializer was called or with what arguments. The trade-off is that initialization is irreversible: there is no "re-initialize" operation, matching the immutability expectations of deployed contracts.
