---
number: 17
name: Explicit Delivery
confidence: 2
contains: []
contained_by: [2, 5]
tags: [notes, delivery, encryption, privacy, intentionality]
---

A created note is useless to its owner until the owner's PXE discovers it. If the framework silently handled delivery, developers might not realize that a missing or misconfigured delivery means the recipient can never access their state.

## Context

When `create_note` is called, it returns a `NoteMessage` rather than silently delivering the note. The developer must explicitly choose how to deliver:

- `.deliver()` -- encrypt the note for its owner and emit it as a constrained encrypted log
- `.deliver_unconstrained()` -- deliver via an unconstrained path (lower cost, weaker guarantees)
- Do nothing -- the note exists on-chain (its hash is in the tree) but no encrypted payload is broadcast

This pattern is modeled after Rust's `#[must_use]` philosophy: the return type forces the developer to make a conscious decision about delivery. Ignoring the `NoteMessage` is possible but requires explicit acknowledgment.

The same pattern appears in event emission: `self.emit(event)` returns a value that must be explicitly delivered, ensuring developers choose between private (encrypted) and public delivery.

## Therefore

Always handle the `NoteMessage` returned by `create_note`. For notes that must reach a specific owner, call `.deliver()` with the appropriate encryption parameters. For notes created for the caller's own use, the PXE can discover them through the oracle notification, but delivery to other parties still requires explicit action.

## Consequences

The explicit delivery pattern prevents a class of bugs where notes are created but never received by their intended owners. It adds one line of code per note creation but makes the delivery guarantee visible in the source code. Code reviewers can verify delivery correctness by looking for `.deliver()` calls adjacent to `create_note` calls.
