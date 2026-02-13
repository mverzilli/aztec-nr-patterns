---
number: 27
name: Generated Calling Interfaces
confidence: 2
contains: []
contained_by: [4, 12, 26]
tags: [codegen, cross-contract, type-safety, calls]
---

Cross-contract calls require encoding function selectors and arguments correctly. Manual encoding is fragile -- a mistyped selector or wrong argument order produces silent failures or invalid proofs.

## Context

The `#[aztec]` macro generates a type-safe calling interface for every contract. For a contract named `Token` with an external function `transfer(to: AztecAddress, amount: u128)`, the macro produces:

```noir
impl Token {
    pub fn at(address: AztecAddress) -> Self { ... }
    pub fn transfer(self, to: AztecAddress, amount: u128) -> PrivateCallInterface { ... }
}
```

Callers invoke this as `self.call(Token::at(address).transfer(recipient, amount))`, which constructs a type-checked call request. The compiler verifies argument types, and the framework handles selector computation, argument serialization, and return value deserialization.

Three kinds of interfaces are generated:
- **External call stubs** -- for calling other contracts' functions
- **Self-call stubs** (`self.enqueue_self.*`, `self.call_self.*`) -- for a contract calling its own functions
- **Static call stubs** -- for read-only view calls

The distinction between `.call()` and `.view()` on the `self` object determines whether the call is state-modifying or read-only, adding another layer of compile-time safety.

## Therefore

Always use the generated interface to make cross-contract calls. Import the target contract's module and use `ContractName::at(address).function(args)` to construct calls. Never manually compute function selectors or serialize arguments.

## Consequences

The generated interfaces make cross-contract calls as type-safe as local function calls. Refactoring a function signature in the target contract will produce compile errors in all callers, preventing silent breakage. The cost is that contracts must be compiled together (or their interfaces made available) for the type checking to work, creating a compile-time dependency graph.
