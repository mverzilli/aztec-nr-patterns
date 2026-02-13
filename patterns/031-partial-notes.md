---
number: 31
name: Partial Notes
confidence: 1
contains: []
contained_by: [4]
tags: [notes, cross-domain, partial, advanced]
---

Some operations require a value that is only known after public execution, but the note must be created in private context. Neither domain alone has all the information needed to create the complete note.

## Context

The **partial note** pattern bridges the private-public gap for note creation. The flow is:

1. A private function creates an incomplete note -- a "partial note" -- containing all privately-known fields (e.g., the owner, the storage slot) but leaving publicly-determined fields unset.
2. The private function enqueues a public function call, passing the partial note's commitment.
3. The public function computes the missing values (e.g., an exchange rate, a fee amount) and "completes" the note by combining the partial commitment with the public values.
4. The completed note hash is committed to the note hash tree.

This allows protocols where the final state depends on both private inputs and current public state. For example, a token swap where the user privately specifies a maximum amount but the actual amount is determined by the current on-chain price.

## Therefore

Use partial notes when a note's contents depend on values that are only available in public execution. Structure the protocol so that private execution creates the partial commitment and enqueues the public completion. The public function must validate that the completion is consistent with the original partial note's constraints.

## Consequences

Partial notes are the most complex pattern in aztec-nr, combining asynchronous execution, cross-domain messaging, and note creation into a single flow. They are necessary for many DeFi primitives but should be used sparingly. The pattern requires careful security analysis to ensure that the public completion step cannot be exploited to create notes with unexpected values.
