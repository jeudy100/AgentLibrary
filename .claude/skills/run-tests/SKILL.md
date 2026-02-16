---
name: run-tests
description: Execute tests based on project type. Uses centralized detection to determine the test framework and runs appropriate commands.
argument-hint: "[file-or-pattern]"
---

# run-tests

Execute tests based on project type. Uses centralized detection to determine the test framework and runs appropriate commands.

## Usage

```
/run-tests
```

Runs all tests by default. You can also specify:
- A file path to run tests in a specific file
- A test name to run a specific test (if supported)

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for running tests. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: test, testing, jest, pytest, vitest, mocha, coverage
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: Project type, package manager, test framework, test command, testing instructions

From the Explore results, extract:
- Project type and package manager
- Test framework
- Test command
- Any testing-specific instructions from CLAUDE.md

### Step 2: Run Tests

Execute the test command from detection results:

**Node.js:**
```bash
# Using detected package manager
npm test
# or with specific file
npm test -- path/to/test.js
```

**Python:**
```bash
pytest
# or with specific file
pytest path/to/test.py
# or with specific test
pytest path/to/test.py::test_name
```

**Go:**
```bash
go test ./...
# or specific package
go test ./pkg/...
# with verbose
go test -v ./...
```

**Rust:**
```bash
cargo test
# specific test
cargo test test_name
```

**Make:**
```bash
make test
```

**Java Maven:**
```bash
mvn test
# or specific test
mvn test -Dtest=TestClass
```

**Java Gradle:**
```bash
./gradlew test
# or specific test
./gradlew test --tests TestClass
```

**.NET:**
```bash
dotnet test
# or specific test
dotnet test --filter "FullyQualifiedName~TestName"
```

**Ruby:**
```bash
bundle exec rspec
# or specific file
bundle exec rspec spec/models/user_spec.rb
# or specific line
bundle exec rspec spec/models/user_spec.rb:42
```

### Step 3: Parse Results

After running tests, parse the output to identify:
- Total tests run
- Passed tests
- Failed tests
- Skipped tests
- Error messages for failures

### Step 4: Report Results

Output in this format (see below).

### Step 5 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

### Report Format

Output in this format:

```
## Test Results

**Framework**: [detected framework]
**Command**: [command executed]
**Status**: PASSED / FAILED

**Summary**:
- Total: X
- Passed: Y
- Failed: Z
- Skipped: W

[If failures, show details]
```

---

## Detection Precedence

When multiple project files exist, use this order:
1. **CLAUDE.md instructions** - Explicit test command (highest priority)
2. **package.json** - `scripts.test` field (Node.js)
3. **pyproject.toml** - `[tool.pytest]` section (Python)
4. **go.mod** - Presence indicates Go project
5. **Cargo.toml** - Presence indicates Rust project
6. **Gemfile** - Presence indicates Ruby project
7. **pom.xml / build.gradle** - Java project
8. **Makefile** - `test` target (fallback)

If detection fails, ask the user:
```
Question: "Could not detect test framework. What command runs your tests?"
Options:
  - Provide test command
  - Cancel
```

---

## Exit Code Handling

Interpret test runner exit codes:
| Exit Code | Meaning |
|-----------|---------|
| `0` | All tests passed |
| `1` | Test failures occurred |
| Other non-zero | Infrastructure/configuration error |

**Note:** Don't rely solely on exit codes - always parse output for accurate counts, as some frameworks exit 0 with skipped tests.

---

## Error Handling

### Test Framework Not Installed

If the test command fails with "command not found":

```
Question: "Test framework not installed. What should I do?"
Options:
  - Show installation instructions
  - Try a different test command
  - Cancel
```

**Installation commands by framework:**
- Jest/Node: `npm install --save-dev jest`
- pytest: `pip install pytest`
- Go: Built-in, check `go` installation
- Rust: Built-in with `cargo`
- RSpec: `bundle install` or `gem install rspec`

### No Tests Found

If tests run but find no test files:

```
No tests found. Checked:
- [list of paths searched based on framework conventions]

Question: "No tests found. What should I do?"
Options:
  - Specify test file/directory
  - Check different location
  - Cancel
```

### Tests Timeout

If tests run longer than 5 minutes without completing:

```
Warning: Tests have been running for over 5 minutes.

Question: "Tests are taking a long time. How should I proceed?"
Options:
  - Continue waiting
  - Stop tests and show partial results
  - Run with --bail/fail-fast flag
  - Cancel
```

### Missing Dependencies

If tests fail due to missing dependencies:

```
Tests failed due to missing dependencies.

Error: [error message]

Question: "Dependencies may not be installed. What should I do?"
Options:
  - Run install command (npm install, pip install, etc.)
  - Continue anyway
  - Cancel
```

### Build/Compilation Failure

If tests fail before running due to build errors:

```
## Build Failed

Cannot run tests - compilation errors occurred.

[Build error output]

Fix the build errors first, then run `/run-tests` again.
```

---

## Project Root Detection

Project root is determined by (in order):
1. Directory containing the detected project file (package.json, pyproject.toml, etc.)
2. Git repository root (`git rev-parse --show-toplevel`)
3. Current working directory

---

## Examples

```
/run-tests
-> Runs all tests in project

/run-tests src/utils.test.ts
-> Runs tests in specific file

/run-tests --coverage
-> Runs tests with coverage (if supported)
```
