---
name: lint-code
description: Run code linters based on project configuration. Uses centralized detection to determine and run the appropriate linter.
---

# lint-code

Run code linters based on project configuration. Uses centralized detection to determine and run the appropriate linter.

## Usage

```
/lint-code                    # Lint entire project
/lint-code src/utils.ts       # Lint specific file
/lint-code --fix              # Auto-fix issues
/lint-code src/ --fix         # Lint directory with auto-fix
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

**Java:**
```bash
# Checkstyle (via Maven)
mvn checkstyle:check

# Checkstyle (via Gradle)
./gradlew checkstyleMain

# SpotBugs
mvn spotbugs:check
./gradlew spotbugsMain
```

**C#/.NET:**
```bash
# dotnet format
dotnet format --verify-no-changes
dotnet format  # with auto-fix

# With analyzers
dotnet build /p:TreatWarningsAsErrors=true
```

### Step 3: Parse Results

Extract from linter output:
- Total issues found
- Errors vs warnings
- File locations
- Rule violations

### Step 4: Report Results

Output in this format (see below).

### Step 5 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

### Report Format

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

---

## Linter Priority

When multiple linters are configured, use this order:

**JavaScript/TypeScript:**
1. `package.json` scripts.lint → use that command
2. Biome (if `biome.json` exists)
3. ESLint (if `.eslintrc*` exists)
4. Prettier (formatting only, if `.prettierrc*` exists)

**Python:**
1. `pyproject.toml` configured tool → use that
2. Ruff (preferred - fast, modern)
3. Flake8 + Black (legacy setup)

**If multiple linters configured:**
Run in order: static analysis first, then formatting.
Example: ESLint → Prettier, or Ruff check → Ruff format

---

## Error Handling

### No Linter Configured

If no linter configuration is found:

```
No linter configured for this project.

Question: "Would you like to set up a linter?"
Options:
  - Show recommended linter for [detected language]
  - Continue without linting
  - Cancel
```

**Recommendations by language:**
- Node.js → ESLint + Prettier
- Python → Ruff
- Go → golangci-lint
- Rust → Clippy (built-in)
- Ruby → RuboCop
- Java → Checkstyle
- C#/.NET → dotnet format

### Linter Not Installed

If the linter command fails with "command not found":

```
Linter '[name]' is not installed.

Question: "How should I proceed?"
Options:
  - Show installation instructions
  - Try a different linter
  - Cancel
```

**Installation commands:**
- ESLint: `npm install -D eslint`
- Prettier: `npm install -D prettier`
- Ruff: `pip install ruff`
- golangci-lint: `go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest`
- RuboCop: `gem install rubocop`

### Invalid Configuration

If linter fails due to config errors:

```
## Configuration Error

Linter '[name]' failed due to invalid configuration.

Error: [error message]
Config file: [path to config]

Fix the configuration, then run `/lint-code` again.
```

### Exit Code Handling

| Exit Code | Meaning |
|-----------|---------|
| `0` | No issues found (PASSED) |
| `1` | Lint issues found (report them) |
| `2+` | Configuration or runtime error |

### Timeout on Large Codebase

If linting takes longer than 60 seconds:

```
Warning: Linting is taking a long time on this codebase.

Question: "How should I proceed?"
Options:
  - Continue waiting
  - Lint specific directory instead
  - Cancel
```

---

## Tips

- Run linters in CI to catch issues early
- Use `--fix` carefully; review changes before committing
- Configure your editor to lint on save
- Add pre-commit hooks for automatic linting
