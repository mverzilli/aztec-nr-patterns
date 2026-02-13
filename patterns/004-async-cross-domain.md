---
number: 4
name: Asynchronous Cross-Domain Messaging
confidence: 2
contains: [27, 31]
contained_by: []
tags: [execution, async, enqueue, cross-domain]
---

Private functions execute on the user's device against historical state, while public functions execute on the block proposer against current state. A private function cannot synchronously call a public function because it would need access to state that does not yet exist from its perspective.

## Context

The **enqueue pattern** bridges the private-to-public gap. When a private function needs to trigger public execution, it does not call the public function directly. Instead, it enqueues a request that the block proposer will execute later in the same block, after all private execution has completed.

This is a one-way, fire-and-forget operation from the private side. The private function cannot read the return value of the enqueued public call. It can, however, pass arguments that were computed privately, enabling patterns where private logic determines *what* public action to take without revealing *why*.

The reverse direction (public-to-private) is not possible within a single transaction. Public functions can only call other public functions synchronously.

## Therefore

Use `self.enqueue_self.function_name(args)` to schedule public execution from private context. Design protocols so that private functions compute and validate data, then hand off to public functions for state changes that require current-state access. Accept that the private caller cannot observe the result of the enqueued call within the same transaction.

## Consequences

Protocols must be designed to tolerate the asynchronous boundary. A private function cannot assert that a subsequent public call succeeded. This shapes contract design: private logic should be self-contained where possible, delegating to public functions only for operations that genuinely need current state. The **Partial Notes** pattern addresses cases where the final value depends on public state.
