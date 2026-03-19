# Stage 3: Trait Inference Prompt

You are translating Python Protocol classes to Rust trait definitions. The Python code
uses structural typing (Protocol from typing) which maps to Rust traits.

## Context

The following Python Protocol class needs to be translated to a Rust trait:

```python
{PYTHON_SOURCE}
```

The surrounding Rust code that will use this trait:

```rust
{RUST_CONTEXT}
```

## Rules

1. Each Protocol method becomes a trait method
2. `self` parameter becomes `&self` (or `&mut self` if the method modifies state — check carefully)
3. All type annotations are already in the Python-as-Rust profile format — translate using:
   - `str` → `&str` for params, `String` for return types
   - `int` → `i64`, `float` → `f64`, `bool` → `bool`
   - `Result[T, E]` → `Result<T, E>`
   - `list[T]` → `Vec<T>`, `dict[K, V]` → `HashMap<K, V>`
4. Add appropriate `where` clauses if generic bounds are needed
5. Add `#[derive(Debug)]` if the trait is used in error messages
6. If the Protocol has associated types, translate them to `type` declarations in the trait

## Output

Return ONLY the Rust trait definition. No explanation, no markdown fences.
