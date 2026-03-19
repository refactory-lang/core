# @refactory/config

Shared TypeScript, oxlint, and oxfmt configuration for all refactory-lang repositories.

## Usage

Install as a dev dependency from GitHub:

```bash
pnpm add -D @refactory/config@github:refactory-lang/core#001-shared-transform-utils
```

### tsconfig.json

```json
{
  "extends": "@refactory/config/tsconfig.base.json",
  "include": ["src/**/*.ts"]
}
```

### oxlint

```bash
oxlint -c node_modules/@refactory/config/oxlintrc.json .
```

Or symlink/copy to repo root:

```bash
ln -s node_modules/@refactory/config/oxlintrc.json .oxlintrc.json
```

### oxfmt

```bash
oxfmt --config node_modules/@refactory/config/oxfmt.json .
```

## What's included

| File | Purpose |
|------|---------|
| `tsconfig.base.json` | Strict ESNext + NodeNext base config |
| `oxlintrc.json` | Lint rules (no-eval, eqeqeq, prefer-const, no-throw-literal) |
| `oxfmt.json` | Format config (2-space indent, 100 width, single quotes, trailing commas) |
