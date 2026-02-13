---
number: 26
name: Macro-Driven Contract Definition
confidence: 2
contains: [27, 30]
contained_by: []
tags: [macros, codegen, boilerplate, aztec-macro]
---

A privacy-preserving smart contract requires substantial boilerplate: note hash and nullifier computation dispatch, state synchronization entry points, message decryption logic, and ABI exports. Writing this by hand for every contract would be error-prone and tedious.

## Context

The `#[aztec]` macro, applied to a contract module, is the central code generation engine. It transforms a developer-authored module into a fully functional contract by:

1. **Validating** that all functions carry appropriate annotations (`#[external]`, `#[internal]`, `#[test]`)
2. **Generating calling interfaces** -- a `ContractName` struct with `.at(address)` factory and type-safe calling methods for every external function (see **Generated Calling Interfaces**)
3. **Synthesizing utility functions**:
   - `_compute_note_hash_and_nullifier()` -- dispatches to the correct note type's hash/nullifier computation
   - `sync_state()` -- discovers notes from encrypted logs and verifies against the note hash tree
   - `process_message()` -- decrypts encrypted note messages and stores them in the PXE
4. **Injecting context** -- wrapping each function body with the appropriate context initialization and validation checks
5. **Exporting ABI** -- generating the metadata that external tools need to interact with the contract

The macro operates at compile time using Noir's `comptime` system, producing code that is indistinguishable from hand-written implementations.

## Therefore

Apply `#[aztec]` to every contract module. Write functions with their semantic annotations (`#[external("private")]`, `#[initializer]`, etc.) and let the macro handle the mechanical parts. Do not manually implement the generated utility functions -- the macro produces them from the contract's note type declarations and function signatures.

## Consequences

The macro approach trades transparency for productivity. Developers cannot see the generated code directly (though they can inspect it via compiler tooling). This can make debugging harder when the generated code misbehaves. However, the macro eliminates entire categories of bugs (mismatched note type IDs, missing ABI exports, incorrect dispatch tables) by generating these mechanically.
