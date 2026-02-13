---
number: 12
name: The Self Object
confidence: 2
contains: [27]
contained_by: [1]
tags: [interface, abstraction, contract, ergonomics]
---

Contract functions need access to storage, the execution context, the contract's own address, and the ability to call other contracts or emit events. Passing all of these as separate parameters would be verbose and error-prone.

## Context

Every contract function in aztec-nr receives a `self` parameter of type `ContractSelf<Context, Storage, ...>`. This object is the developer's primary interface to the framework, bundling together:

- `self.storage` -- the contract's state variables, initialized from the `#[storage]` struct
- `self.context` -- the execution context (`PrivateContext`, `PublicContext`, or `UtilityContext`)
- `self.address` -- the contract's own address
- `self.msg_sender()` -- the caller's address
- `self.call(...)` / `self.view(...)` -- cross-contract calls
- `self.enqueue_self.*` -- enqueue public calls from private context
- `self.call_self.*` / `self.internal.*` -- self-calls and internal function calls
- `self.emit(event)` -- event emission

The `#[aztec]` macro generates the `ContractSelf` type and injects it into every function's signature. Developers never construct it manually.

## Therefore

Use `self` as the single entry point for all framework interactions. Access storage through `self.storage`, make cross-contract calls through `self.call()`, and emit events through `self.emit()`. The object's available methods change based on the context type, naturally guiding developers toward valid operations.

## Consequences

The `self` object provides a discoverable API surface -- developers can explore available operations through IDE auto-completion. It also serves as the attachment point for **Generated Calling Interfaces**, where the macro synthesizes type-safe methods for calling the contract's own functions and other contracts' functions.
