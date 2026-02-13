---
number: 30
name: Packable Serialization
confidence: 2
contains: []
contained_by: []
tags: [serialization, storage, fields, traits]
---

Blockchain state is ultimately stored as arrays of finite field elements, but contract developers work with structured types (structs, integers, addresses). The system needs a uniform way to convert between structured types and field arrays for storage and hashing.

## Context

The `Packable` trait defines how types are serialized to and deserialized from fixed-length field arrays:

```noir
trait Packable<let N: u32> {
    fn pack(self) -> [Field; N];
    fn unpack(fields: [Field; N]) -> Self;
}
```

Every type stored in state variables must implement `Packable`. This includes note types (packed for hashing), public state values (packed for storage tree insertion), and function arguments (packed for cross-contract call serialization).

The trait is implemented for primitive types (`Field`, `u128`, `AztecAddress`, `bool`) and can be derived for structs. The packed length `N` is a compile-time constant, enabling the type system to verify that storage slots are correctly sized and that serialization round-trips are length-preserving.

## Therefore

Implement `Packable` for any custom type that needs to be stored, hashed, or passed across contract boundaries. For structs, this typically means concatenating the packed representations of each field. The compile-time length parameter ensures that packing/unpacking is always symmetric -- a value packed into `N` fields will always unpack from exactly `N` fields.

## Consequences

`Packable` provides a single, consistent serialization mechanism across the entire framework. Storage, hashing, cross-contract calls, and event emission all use the same serialization path. The trade-off is that all storable types must express themselves as fixed-length field arrays, which can be awkward for variable-length data. In practice, the framework provides utilities for common cases (compressed strings, bounded arrays).
