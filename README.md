# @refactory/core

Shared utilities for Refactory Codemod packages. Contains type mapping functions, common transform helpers, and cross-language utilities used by both `@refactory/python-to-rust` and `@refactory/typescript-to-rust`.

## What's Shared

| Utility | Used By | Purpose |
|---------|---------|---------|
| `mapPrimitiveType()` | Both packages | `int→i64`, `str→String`, `number→f64`, etc. |
| `mapCollectionType()` | Both packages | `list[T]→Vec<T>`, `Array<T>→Vec<T>`, etc. |
| `toSnakeCase()` | Both packages | Identifier case conversion for Rust |
| `emitRustStruct()` | Both packages | Consistent struct emission with derives |
| `emitRustFunction()` | Both packages | Consistent function signature emission |
| `translateResultType()` | Both packages | `Result[T,E]→Result<T,E>` (Python), `Result<T,E>→Result<T,E>` (TS) |

## Shared Transform Rules

Several Tier 1 rules are structurally identical across Python and TypeScript profiles (string method mappings, numeric type mappings, Result-pattern transforms). These are implemented once in `@refactory/core` and imported by both packages.

## License

Apache-2.0
