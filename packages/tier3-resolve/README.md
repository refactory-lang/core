# Tier 3: Rust→Rust Resolve

Language-agnostic Tier 3 resolution for `todo!("t3:*")` stub markers emitted by Tier 1/2 transforms.

Tier 3 operates exclusively on Rust code — it is **not** a Python→Rust or TypeScript→Rust transform. Both `python-to-rust` and `typescript-to-rust` pipelines use this shared T3 pass.

## Prompt Templates

- `prompts/trait-inference.md` — Python Protocol → Rust trait definition
- `prompts/lifetime-resolution.md` — Resolve lifetime annotations from compiler errors
- `prompts/generic-bounds.md` — Complex where clauses from inferred bounds
- `prompts/async-patterns.md` — Tokio-specific patterns (select!, Pin, cancellation)

## Workflow Integration

Reference this package from any translation pipeline's `workflow.yaml`:

```yaml
- id: count-stubs
  name: Count Tier 3 stubs
  type: shell
  depends_on: [tier2-modules]
  config:
    command: grep -rc 'todo!("t3:' build/rust/src/ | awk -F: '{s+=$2}END{print s}'

- id: tier3-resolve
  name: Tier 3 Rust→Rust idiomaticity pass
  type: ai
  depends_on: [count-stubs]
  skip_if: stub_count == 0
  config:
    model: claude-sonnet-4-20250514
    prompts: node_modules/@refactory/core/packages/tier3-resolve/prompts/
    input: build/rust/src/
    scope: stub_markers_only
    context:
      compiler_errors: true
      original_source: true
    output_schema: tier3-structured-output.json
```

## T3 Rule Types

Per v0.3 spec, these are the 5 irreducible residual construct types:

1. **Lifetime annotation inference** — ownership scope with no Python/TS representation
2. **Trait bound completion** — `T: Clone + Send` etc. from usage analysis
3. **Complex where clauses** — generic bounds from inferred constraints
4. **Async patterns** — tokio `select!`, `Pin`, cancellation semantics
5. **Generic bounds** — complex where clauses from inferred bounds
