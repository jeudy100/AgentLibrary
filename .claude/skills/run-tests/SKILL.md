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

### Step 3: Parse Results

After running tests, parse the output to identify:
- Total tests run
- Passed tests
- Failed tests
- Skipped tests
- Error messages for failures

### Step 4: Report Results

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

## Examples

```
/run-tests
-> Runs all tests in project

/run-tests src/utils.test.ts
-> Runs tests in specific file

/run-tests --coverage
-> Runs tests with coverage (if supported)
```
