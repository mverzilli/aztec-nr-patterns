---
number: 1
name: Dual Execution Model
confidence: 2
contains: [11, 12, 18]
contained_by: []
tags: [architecture, execution, privacy, public]
---

A blockchain that executes everything publicly leaks all user data, but a blockchain that executes everything privately cannot support shared mutable state. The system needs both modes, but combining them introduces fundamental asymmetries in timing, visibility, and guarantees.

## Context

Aztec separates execution into two domains that operate under radically different rules. **Private functions** run on the user's device, producing zero-knowledge proofs that attest to correct execution without revealing inputs. **Public functions** run on the block proposer's machine against the current blockchain state, visible to everyone.

This split is not merely an optimization choice; it is an architectural invariant that pervades every design decision in aztec-nr. Private execution is asynchronous: by the time a private transaction is included in a block, the state it read may have changed. Public execution is synchronous: it sees and modifies current state within the same block.

The two domains communicate through a one-way bridge: private functions can **enqueue** public calls (see **Asynchronous Cross-Domain Messaging**), but public functions cannot call private ones. This asymmetry exists because private execution produces proofs against historical state, while public execution requires current state.

## Therefore

Accept the dual-execution model as the foundational invariant of the system. Design every abstraction so that the domain boundary is explicit and enforced at compile time. Private and public functions receive different context types (`PrivateContext` vs `PublicContext`), and the type system prevents operations that cross the boundary incorrectly. State variables declare which contexts they support, making it a compile error to read public mutable state from a private function.

## Consequences

Developers must think in terms of two execution environments with different capabilities and timing guarantees. This is the primary source of complexity in aztec-nr, but also the source of its privacy guarantees. The framework minimizes accidental misuse through the **Context-Aware Type System**, making the boundary feel natural rather than burdensome.
