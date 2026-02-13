---
number: 11
name: Context-Aware Type System
confidence: 2
contains: [13, 30]
contained_by: [1]
tags: [types, generics, safety, compile-time]
---

A developer might accidentally read public mutable state from a private function, or attempt to create notes in a public context. These errors are subtle and would produce cryptographically invalid proofs at runtime -- far too late to catch.

## Context

Nearly every abstraction in aztec-nr is parameterized by a `Context` generic type. State variables like `PublicMutable<T, Context>` and `PrivateSet<Note, Context>` implement different methods depending on whether `Context` is `PrivateContext`, `PublicContext`, or `UtilityContext`. A `PublicMutable` provides a `read()` method when the context is `PublicContext` but a `read_public()` (historical read) method when the context is `PrivateContext`.

This means the compiler itself enforces the dual-execution boundary. If you write code that tries to call `.read()` on a `PublicMutable` inside a private function, the code will not compile. The error message points directly at the invalid operation rather than failing at proof generation time.

The pattern extends beyond state variables. The `ContractSelf<Context, Storage>` type provides different methods (`.enqueue_self`, `.call_self`, `.internal`) depending on the context, ensuring that only valid cross-domain operations are expressible.

## Therefore

Rely on the `Context` generic parameter as the primary enforcement mechanism for execution-domain safety. When designing new abstractions, parameterize them with `Context` and implement domain-specific behavior as trait impls constrained to specific context types. This shifts domain-rule violations from runtime failures to compile errors.

## Consequences

The codebase carries `Context` as a generic parameter through most types, adding syntactic weight. This is a deliberate trade-off: the verbosity is the price for compile-time domain safety. Developers quickly internalize which operations are available in which context, guided by the compiler rather than documentation.
