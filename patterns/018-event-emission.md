---
number: 18
name: Event Emission Protocol
confidence: 2
contains: []
contained_by: [1]
tags: [events, logging, privacy, encryption]
---

Contracts need to emit observable side-effects for off-chain indexing, but in a privacy-preserving system, events must support both encrypted (private) and transparent (public) emission modes.

## Context

Events in aztec-nr are defined with the `#[event]` macro, which generates an `EventInterface` implementation including a unique selector (computed from the event name and parameter types). The macro also derives `Serialize` if not already present.

Event emission follows a dual-mode pattern matching the execution model:

**In private functions:** `self.emit(event)` returns an event message that must be explicitly delivered -- mirroring the **Explicit Delivery** pattern for notes. The developer chooses between encrypted delivery (only specific parties can read it) and offchain logs.

**In public functions:** `self.emit(event)` emits the event as a public, unencrypted log visible to everyone. No delivery decision is needed because public logs are inherently transparent.

The `#[event]` macro detects selector collisions at compile time, ensuring that each event type within a contract has a unique identifier for off-chain decoding.

## Therefore

Define events with `#[event]` and emit them through `self.emit()`. In private functions, explicitly deliver the event message to its intended recipients. In public functions, emission is immediate and transparent. Use events for data that off-chain systems need to observe -- note delivery handles private state synchronization separately.

## Consequences

The dual-mode emission model ensures that event privacy matches function privacy. Developers do not accidentally emit private data publicly or vice versa, because the context type determines which emission mode is available. The trade-off is that private events require recipients to actively listen and decrypt, making them less universally observable than public events.
