<!-- codemod-skill-discovery:begin -->
## Codemod Skill Discovery
This section is managed by `codemod` CLI.

- Core skill: `.agents/skills/codemod/SKILL.md`
- Package skills: `.agents/skills/<package-skill>/SKILL.md`
- List installed Codemod skills: `npx codemod agent list --harness antigravity --format json`

<!-- codemod-skill-discovery:end -->

## Project: @refactory/core

Shared utilities for Refactory Codemod packages. Part of the [refactory-lang](https://github.com/refactory-lang) organization. Contains type mapping functions, common transform helpers, and cross-language utilities used by both `@refactory/python-to-rust` and `@refactory/typescript-to-rust`. Also contains the Stage 3 Resolve package with LLM prompt templates for Rust-to-Rust resolution.

### Architecture

- **Shared utilities** (`src/`): Type mappers (`mapPrimitiveType`, `mapCollectionType`), case converters (`toSnakeCase`), emit helpers (`emitRustStruct`, `emitRustFunction`), Result type translators
- **Stage 3 Resolve** (`packages/stage3-resolve/`): LLM-assisted Rust→Rust resolve package
  - `prompts/`: Prompt templates for lifetime resolution, generic bounds, trait inference, async patterns
  - `src/`: Resolve logic implementation
- **Specs** (`specs/`): Implementation specifications
- **Tests** (`tests/`): Test suite

### Running

```bash
# Validate the codemod package
npx codemod workflow validate -w codemod.yaml

# Run tests
npx vitest run
```

### Key Files

| File | Purpose |
|------|---------|
| `codemod.yaml` | Codemod package definition |
| `src/` | Shared transform utilities (type mappers, emit helpers, case converters) |
| `packages/stage3-resolve/prompts/lifetime-resolution.md` | LLM prompt for Rust lifetime inference |
| `packages/stage3-resolve/prompts/generic-bounds.md` | LLM prompt for generic bound inference |
| `packages/stage3-resolve/prompts/trait-inference.md` | LLM prompt for trait inference |
| `packages/stage3-resolve/prompts/async-patterns.md` | LLM prompt for async pattern resolution |

### Shared Utilities

| Utility | Used By | Purpose |
|---------|---------|---------|
| `mapPrimitiveType()` | Both packages | `int`/`number` → Rust numeric types |
| `mapCollectionType()` | Both packages | `list[T]`/`Array<T>` → `Vec<T>`, etc. |
| `toSnakeCase()` | Both packages | Identifier case conversion |
| `emitRustStruct()` | Both packages | Consistent struct emission with derives |
| `emitRustFunction()` | Both packages | Consistent function signature emission |
| `translateResultType()` | Both packages | Result type translation to Rust |
