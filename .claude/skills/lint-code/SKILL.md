# lint-code

Run code linters based on project configuration. Uses centralized detection to determine and run the appropriate linter.

## Usage

```
/lint-code
```

Lints the entire project by default. You can also specify:
- A file path to lint a specific file
- `--fix` to auto-fix issues where possible

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for linting code. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: lint, eslint, prettier, ruff, format, style, conventions
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: Project type, linter(s), lint command, code style instructions

From the Explore results, extract:
- Project type and package manager
- Linter(s) configured
- Lint command
- Any code style instructions from CLAUDE.md

### Step 2: Run Linter

Using the detected linter, run the appropriate command:

**JavaScript/TypeScript:**
```bash
# ESLint
npx eslint .
npx eslint . --fix  # with auto-fix

# Prettier (check)
npx prettier --check .
npx prettier --write .  # with auto-fix

# Biome
npx biome check .
npx biome check --apply .  # with auto-fix
```

**Python:**
```bash
# Ruff (fast, recommended)
ruff check .
ruff check . --fix  # with auto-fix
ruff format .  # formatting

# Flake8
flake8 .

# Black (formatting)
black --check .
black .  # with auto-fix
```

**Go:**
```bash
# golangci-lint
golangci-lint run

# go fmt
go fmt ./...

# go vet
go vet ./...
```

**Rust:**
```bash
# Clippy
cargo clippy

# rustfmt
cargo fmt --check
cargo fmt  # with auto-fix
```

**Ruby:**
```bash
# RuboCop
bundle exec rubocop
bundle exec rubocop -a  # with auto-fix
```

### Step 3: Parse Results

Extract from linter output:
- Total issues found
- Errors vs warnings
- File locations
- Rule violations

### Step 4: Report Results

Output in this format:

```
## Lint Results

**Linter**: [detected linter]
**Command**: [command executed]
**Status**: PASSED / [X] issues found

## Summary

- Errors: X
- Warnings: Y
- Fixable: Z

## Issues

### Errors

| File | Line | Rule | Message |
|------|------|------|---------|
| src/index.ts | 10 | no-unused-vars | 'x' is defined but never used |

### Warnings

| File | Line | Rule | Message |
|------|------|------|---------|
| src/utils.ts | 25 | prefer-const | Use 'const' instead of 'let' |

## Auto-fixable Issues

Run `[command] --fix` to automatically fix [Z] issues.

## Manual Fixes Required

### [File:Line] - [Rule]
**Issue**: [description]
**Fix**: [suggested fix]
```

## Common Linter Configurations

### ESLint + Prettier (Node.js)
```json
// .eslintrc.json
{
  "extends": ["eslint:recommended", "prettier"],
  "env": { "node": true, "es2022": true }
}
```

### Ruff (Python)
```toml
# pyproject.toml
[tool.ruff]
select = ["E", "F", "I", "N", "W"]
ignore = ["E501"]
```

### golangci-lint (Go)
```yaml
# .golangci.yml
linters:
  enable:
    - gofmt
    - govet
    - errcheck
    - staticcheck
```

## Tips

- Run linters in CI to catch issues early
- Use `--fix` carefully; review changes before committing
- Configure your editor to lint on save
- Add pre-commit hooks for automatic linting
