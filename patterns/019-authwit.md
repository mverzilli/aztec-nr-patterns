---
number: 19
name: Authorization Witnesses
confidence: 2
contains: []
contained_by: []
tags: [authorization, authentication, accounts, delegation]
---

In a privacy-preserving system, the traditional `msg.sender` check for authorization is insufficient. A user might want to authorize a specific action without being the direct caller, or delegate authority to a contract acting on their behalf.

## Context

The **AuthWit** (Authorization Witness) pattern enables delegated authorization. Instead of checking "is the caller the owner?", a contract can check "did the owner authorize this specific action?" by verifying a signature over the action's hash.

The pattern works in two steps:

1. The application contract computes an `inner_hash` representing the specific action (e.g., "transfer 100 tokens from Alice to Bob via contract X").
2. It calls `check_authwit(outer_hash, inner_hash)` which routes to the authorizing account's `is_valid_impl` function.

Account contracts implement `is_valid_impl` with their own signature verification logic (ECDSA, Schnorr, multisig, etc.). This makes authorization scheme-agnostic: the token contract does not need to know *how* the owner authorized the action, only *that* they did.

In private, AuthWit checks happen via private function calls to the account contract. In public, they can use on-chain storage of pre-approved action hashes. Both paths converge on the same `check_authwit` interface.

## Therefore

Use `check_authwit` when a function operates on behalf of another user (transfers, approvals, delegated actions). The authorizing user creates a witness off-chain (signing the action hash) or on-chain (storing approval). The application contract verifies the witness without knowing the authorization mechanism. This decouples application logic from account implementation.

## Consequences

AuthWit enables a rich ecosystem of account types (hardware wallets, multisigs, social recovery) that all work with the same application contracts. The cost is an additional verification step per delegated action, but this is inherent to any authorization system. The pattern also enables "approve and call" flows without the two-transaction pattern required by ERC-20 approve/transferFrom in Ethereum.
