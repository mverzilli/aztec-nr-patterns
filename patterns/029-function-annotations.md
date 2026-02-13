---
number: 29
name: Function Annotation Convention
confidence: 2
contains: []
contained_by: [11, 26]
tags: [macros, annotations, functions, convention]
---

A contract contains functions with fundamentally different execution semantics: private, public, utility, internal, initializer, view-only. Without a clear classification system, it is ambiguous which functions can be called externally and in which execution context.

## Context

Aztec-nr uses a layered annotation system where every function must declare its nature:

**Domain annotations** (exactly one required):
- `#[external("private")]` -- externally callable, runs in private context
- `#[external("public")]` -- externally callable, runs in public context
- `#[external("utility")]` -- externally callable, runs unconstrained
- `#[internal]` -- callable only from within the contract
- `#[test]` -- test function, not part of the contract

**Modifier annotations** (optional, combinable):
- `#[initializer]` -- marks a constructor function
- `#[view]` -- marks a read-only function
- `#[only_self]` -- restricts to calls from the contract itself
- `#[noinitcheck]` -- skips initialization verification
- `#[allow_phase_change]` -- permits non-revertible to revertible transition (account contracts only)

The `#[aztec]` macro validates that every non-test function has exactly one domain annotation. Functions without annotations produce a compile error, preventing accidentally unclassified functions from appearing in the contract.

## Therefore

Annotate every function with its domain and any applicable modifiers. Read the annotations as a declaration of intent: `#[external("private"), initializer]` means "this is a private function that serves as a constructor." The annotations drive code generation, ABI export, and context injection, so they are not merely documentation -- they are load-bearing.

## Consequences

The annotation system makes function classification explicit and machine-readable. New developers can understand a contract's interface by scanning annotations alone. The compile-time validation ensures no function escapes classification, preventing a category of "accidentally exposed" bugs.
