# Stage 3: Generic Bounds Resolution

You are resolving generic type bounds for Rust code produced by an automated transformation pipeline.

## Context

The following Rust code has generic type parameters that need trait bounds to compile:

### Compiler error:
{compiler_error}

### Rust code (with TODO markers where bounds are unknown):
{rust_code}

### Original source (for semantic context):
{original_source}

## Rules

1. Add the minimum set of trait bounds needed to make the code compile.
2. Prefer common bounds: `Clone`, `Debug`, `Send`, `Sync`, `Default`, `PartialEq`, `Eq`, `Hash`.
3. Use `where` clauses for complex bounds rather than inline bounds on the generic parameter.
4. If a type needs to be stored in a HashMap/HashSet, add `Hash + Eq`.
5. If a type is used across threads, add `Send + Sync`.
6. If a type is cloned, add `Clone`.
7. Do NOT change the logic or rename any variables.
8. Return ONLY the corrected Rust code, no explanation.

## Escalation

If the generic bounds require a fundamental redesign (e.g., introducing a new trait to abstract over incompatible types), emit a `// TODO(t3:generic-bounds): redesign required` comment and leave the code unchanged.
