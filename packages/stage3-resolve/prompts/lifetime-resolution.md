# Lifetime Resolution — Stage 3 AI Prompt Template

## Purpose

This prompt is used by the Stage 3 AI workflow step (`stage3-ai-fallback` in `workflow.yaml`)
to resolve Rust lifetime annotations for patterns that Stage 1 and Stage 2 deterministic
transforms could not handle.

Lifetimes arise when the translated Rust code contains references (`&T`, `&mut T`) whose
scope cannot be inferred by `rustc` automatically. Common sources:

- Python functions that accept mutable objects and return references into them
- Struct fields that borrow from an outer scope
- Iterator chains that yield references to owned data
- Closure captures that outlive the closure's enclosing scope

---

## Prompt Template

```
You are a Rust lifetime annotation expert assisting an automated Python-to-Rust
transformation pipeline.

The following Rust code was produced by deterministic AST transforms (Stage 1 and Stage 2).
It fails to compile because the borrow checker cannot infer lifetimes for one or more
references.

### Compiler error:
{compiler_error}

### Rust code (with TODO markers where lifetimes are unknown):
{rust_code}

### Original Python source (for semantic context):
{python_source}

### Instructions:

1. Add the minimum set of lifetime annotations needed to make the code compile.
2. Prefer named lifetimes (`'a`, `'b`) over anonymous lifetimes (`'_`) for clarity.
3. If a function borrows from multiple sources with different lifetimes, name each
   lifetime separately.
4. If a struct borrows data, add lifetime parameters to both the struct definition
   and all `impl` blocks.
5. Do NOT change the logic or rename any variables.
6. Return ONLY the corrected Rust code, no explanation.
```

---

## Context Variables

| Variable | Source | Description |
|---|---|---|
| `{compiler_error}` | `cargo build` stderr | The full rustc error message |
| `{rust_code}` | `build/rust/src/*.rs` | The Rust file with lifetime issues |
| `{python_source}` | `src/*.py` | The original Python source for semantic context |

---

## Common Patterns

### Borrowing from a parameter

```python
# Python (input)
def first_word(s: str) -> str:
    return s.split()[0]
```

```rust
// Stage 1 output (missing lifetime)
pub fn first_word(s: &str) -> &str {
    s.split_whitespace().next().unwrap_or("")
}
```

```rust
// After lifetime resolution (no explicit annotation needed — elision applies)
pub fn first_word(s: &str) -> &str {
    s.split_whitespace().next().unwrap_or("")
}
```

### Struct holding a reference

```python
# Python (input)
@dataclass(frozen=True)
class Parser:
    source: str
    pos: int
```

```rust
// Stage 1 output (if source is a borrow, not owned)
pub struct Parser<'a> {
    pub source: &'a str,
    pub pos: i64,
}

impl<'a> Parser<'a> {
    pub fn new(source: &'a str) -> Self {
        Parser { source, pos: 0 }
    }
}
```

---

## Escalation

If lifetime resolution requires changing the fundamental ownership model
(e.g., replacing a reference with `Arc<T>` or `Rc<T>` for shared ownership),
emit a `// TODO(lifetime): ownership model change required` comment and leave
the code unchanged. These cases require human review.
