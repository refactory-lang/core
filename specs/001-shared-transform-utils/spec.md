# Feature Specification: Shared Transform Utilities

**Feature Branch**: `001-shared-transform-utils`
**Created**: 2026-03-13
**Status**: Draft

## Overview

The core shared transform utilities package provides common functions consumed by both the python-to-rust and typescript-to-rust transpilation pipelines. It centralises type mapping, case conversion, and Rust code emission so that each pipeline operates on a single, canonical set of helpers rather than maintaining its own divergent copies.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Pipeline developer imports shared utils (Priority: P1)

As a transpilation pipeline developer, I want to import `mapPrimitiveType`, `mapCollectionType`, `toSnakeCase`, `toPascalCase`, `emitRustStruct`, `emitRustFunction`, and `translateResultType` from a single `@refactory/core` package so that I do not duplicate mapping logic in my pipeline.

**Why this priority**: Without shared utilities every pipeline must reimplement type mapping and emission, leading to drift and bugs.

**Independent Test**: Import each function in an isolated test file and verify it is callable with expected arity.

**Acceptance Scenarios**:

```
Scenario 1: Primitive type mapping
  Given the source type string "str" from a Python AST node
  When I call mapPrimitiveType("str")
  Then the return value is "String"

Scenario 2: Collection type mapping
  Given the source type "list[int]" from a Python AST node
  When I call mapCollectionType("list", ["int"])
  Then the return value is "Vec<i64>"

Scenario 3: Case conversion — snake_case
  Given the identifier "getUserName"
  When I call toSnakeCase("getUserName")
  Then the return value is "get_user_name"

Scenario 4: Case conversion — PascalCase
  Given the identifier "user_profile"
  When I call toPascalCase("user_profile")
  Then the return value is "UserProfile"
```

---

### User Story 2 - New pipeline reuses core (Priority: P1)

As a developer building a new language-to-Rust pipeline (e.g., Go-to-Rust), I want to depend on `@refactory/core` and immediately gain access to Rust emission helpers so that I only need to write the language-specific parser and AST adapter.

**Why this priority**: Reusability is the primary architectural motivation for extracting these utilities.

**Independent Test**: Create a minimal mock pipeline that calls `emitRustStruct` and `emitRustFunction` and confirm valid Rust source is produced.

**Acceptance Scenarios**:

```
Scenario 1: Emit a Rust struct
  Given a struct descriptor { name: "UserProfile", fields: [{ name: "email", type: "String" }, { name: "age", type: "i64" }] }
  When I call emitRustStruct(descriptor)
  Then the output contains "pub struct UserProfile" and both fields with correct types

Scenario 2: Emit a Rust function
  Given a function descriptor { name: "calculate_total", params: [{ name: "items", type: "Vec<Item>" }], returnType: "f64" }
  When I call emitRustFunction(descriptor)
  Then the output contains "pub fn calculate_total(items: Vec<Item>) -> f64"

Scenario 3: Translate Result type
  Given a Python return annotation "Result[int, ValueError]"
  When I call translateResultType("int", "ValueError")
  Then the return value is "Result<i64, ValueError>"
```

---

### Edge Cases

- **Unknown primitive type**: `mapPrimitiveType("complex128")` must return an explicit error or fallback type (e.g., `/* unsupported: complex128 */`) rather than silently emitting invalid Rust.
- **Nested collections**: `mapCollectionType("dict", ["str", "list[int]"])` must recursively resolve to `HashMap<String, Vec<i64>>`.
- **Empty identifier**: `toSnakeCase("")` and `toPascalCase("")` must return an empty string without throwing.
- **Already-correct casing**: `toSnakeCase("already_snake")` must return the input unchanged; `toPascalCase("AlreadyPascal")` likewise.
- **Reserved Rust keywords**: If an identifier maps to a Rust keyword (e.g., `type`), the emission helpers must produce `r#type`.
- **Generic Result with no error type**: `translateResultType("int", null)` must default the error position to a project-standard error type or return a clear error.

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST map Python/TypeScript primitive types to Rust equivalents: `str`/`string` to `String`, `int`/`number` to `i64`, `float`/`number` to `f64`, `bool`/`boolean` to `bool`, `None`/`void`/`undefined` to `()`.
- **FR-002**: System MUST map collection types: `list`/`Array` to `Vec<T>`, `dict`/`Record`/`Map` to `HashMap<K, V>`, `set`/`Set` to `HashSet<T>`, `tuple`/`[T, U]` to `(T, U)`.
- **FR-003**: System MUST map `Optional[T]`/`T | null`/`T | undefined` to `Option<T>`.
- **FR-004**: System MUST map `Result[T, E]` to `Result<T, E>` via `translateResultType`.
- **FR-005**: `toSnakeCase` MUST correctly handle camelCase, PascalCase, SCREAMING_SNAKE, and kebab-case inputs.
- **FR-006**: `toPascalCase` MUST correctly handle snake_case, camelCase, kebab-case, and SCREAMING_SNAKE inputs.
- **FR-007**: `emitRustStruct` MUST produce a syntactically valid `pub struct` with `pub` fields and derive macros configurable via options.
- **FR-008**: `emitRustFunction` MUST produce a syntactically valid `pub fn` with typed parameters and return type.
- **FR-009**: All functions MUST be importable as named exports from the package entry point.
- **FR-010**: System MUST escape Rust reserved keywords using raw identifier syntax (`r#`).

### Key Entities

| Entity | Description |
|---|---|
| `PrimitiveTypeMap` | Lookup table from source language primitives to Rust primitives |
| `CollectionTypeMap` | Lookup table from source language collections to Rust generic collections |
| `StructDescriptor` | Object describing a Rust struct: name, fields (name + type), derive macros |
| `FunctionDescriptor` | Object describing a Rust function: name, params, return type, body placeholder |
| `CaseConverter` | Stateless utility for identifier case transformations |

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: The python-to-rust pipeline imports and uses core utils for all type mapping — zero local type-mapping logic remains in that repo.
- **SC-002**: The typescript-to-rust pipeline imports and uses core utils for all type mapping — zero local type-mapping logic remains in that repo.
- **SC-003**: Unit tests cover every primitive mapping (at least 8 source types), every collection mapping (at least 4), and both case converters with at least 5 input variants each.
- **SC-004**: `emitRustStruct` and `emitRustFunction` outputs pass `rustfmt --check` (valid syntax).
- **SC-005**: No duplicated type-mapping or case-conversion logic exists across the refactory-lang monorepo.
- **SC-006**: Edge-case inputs (unknown types, empty strings, reserved keywords) produce deterministic, documented behaviour — no unhandled exceptions.
