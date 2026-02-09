# analyze-coverage

Analyze test coverage reports and identify untested code paths. Uses centralized detection to determine the coverage tool.

## Usage

```
/analyze-coverage
```

Generates and analyzes coverage by default. You can also specify:
- `--report-only` to analyze an existing coverage report without regenerating

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for analyzing test coverage. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: coverage, threshold, uncovered, test coverage
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: Project type, coverage tool, coverage command, coverage thresholds

From the Explore results, extract:
- Project type and package manager
- Coverage tool
- Coverage command
- Any coverage thresholds or requirements from CLAUDE.md

### Step 1.5: Handle --report-only Flag

If `--report-only` flag is specified, skip Step 2 and locate the existing coverage report:

**Common Report Locations:**
| Tool | Report Path |
|------|-------------|
| Jest | `coverage/coverage-summary.json` or `coverage/lcov.info` |
| c8 | `coverage/lcov.info` |
| Vitest | `coverage/coverage-summary.json` |
| pytest-cov | `htmlcov/index.html` or `.coverage` |
| Go | `coverage.out` |
| Rust/tarpaulin | `tarpaulin-report.html` or `cobertura.xml` |
| JaCoCo | `target/site/jacoco/jacoco.xml` |
| .NET | `TestResults/*/coverage.cobertura.xml` |

If no report is found:
```
Question: "No existing coverage report found. What should I do?"
Options:
  - Generate a new coverage report
  - Specify report path
  - Cancel
```

### Step 2: Generate Coverage Report

Using the detected coverage tool, run the appropriate command:

**Node.js (with Jest):**
```bash
npm test -- --coverage
```

**Node.js (with c8):**
```bash
npx c8 npm test
```

**Node.js (with Vitest):**
```bash
npx vitest run --coverage
```

**Python:**
```bash
pytest --cov=. --cov-report=term-missing
# or
coverage run -m pytest && coverage report -m
```

**Go:**
```bash
go test -coverprofile=coverage.out ./...
go tool cover -func=coverage.out
```

**Rust:**
```bash
cargo tarpaulin --out Html
```

**Java Maven:**
```bash
mvn test jacoco:report
```

**Java Gradle:**
```bash
./gradlew test jacocoTestReport
```

**.NET:**
```bash
dotnet test --collect:"XPlat Code Coverage"
```

### Step 3: Parse Coverage Report

Locate and parse the coverage output based on the tool:

**Jest (coverage-summary.json):**
```bash
cat coverage/coverage-summary.json
```
Parse JSON for `total.lines.pct`, `total.branches.pct`, and per-file data under each file key.

**pytest-cov (terminal output):**
Parse stdout from Step 2. Look for lines like:
```
Name                      Stmts   Miss  Cover   Missing
-------------------------------------------------------
src/module.py               100     20    80%   15-20, 45
```

**Go (coverage.out):**
```bash
go tool cover -func=coverage.out
```
Parse the last line for total percentage, and per-function lines for file details.

**JaCoCo (jacoco.xml):**
Parse XML file at `target/site/jacoco/jacoco.xml` for `<counter>` elements with `type="LINE"` and `type="BRANCH"`.

**.NET (cobertura.xml):**
Parse XML for `<coverage line-rate="X">` attribute and `<class>` elements for per-file data.

**Extract key metrics:**
- Overall coverage percentage
- Per-file coverage
- Uncovered lines
- Branch coverage (if available)

### Step 4: Identify Gaps

Apply thresholds based on code type (see Coverage Thresholds section):
- **Critical paths** (auth, payments, core logic): Flag if < 90%
- **Business logic**: Flag if < 80%
- **Utilities/helpers**: Flag if < 70%
- **Generated code**: Skip analysis

**Always flag:**
1. Files with 0% coverage (completely untested)
2. Files with < 50% coverage (critical)
3. Files with < 80% coverage (warning)
4. Uncovered error handling paths (catch blocks, error returns)
5. Uncovered edge cases (null checks, boundary conditions)
6. New code without tests

### Step 5: Report Results

Output in this format (see below).

### Step 6 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

### Report Format

Output in this format:

```
## Coverage Report

**Overall Coverage**: XX%
**Branch Coverage**: XX% (if available)

## Summary by Status

### Critical (< 50%)
| File | Coverage | Uncovered Lines |
|------|----------|-----------------|
| path/to/file.ts | 35% | 10-15, 22-30 |

### Needs Attention (50-80%)
| File | Coverage | Uncovered Lines |
|------|----------|-----------------|
| path/to/file.ts | 65% | 45-50 |

### Good Coverage (> 80%)
| File | Coverage |
|------|----------|
| path/to/file.ts | 95% |

## Recommendations

1. **[file.ts]**: Add tests for error handling in `functionName()`
2. **[file.ts]**: Edge case not covered: empty input handling

## Suggested Tests

[Generate language-appropriate test examples based on detected project type]

**Example format:**
```[detected-language]
// Test for uncovered path in [file]:[lines]
[appropriate test syntax for the project's test framework]
```
```

## Common Coverage Tools Setup

### Node.js with Jest
```json
// package.json
{
  "jest": {
    "collectCoverage": true,
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80
      }
    }
  }
}
```

### Python with pytest-cov
```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = "--cov=src --cov-report=term-missing"

[tool.coverage.run]
branch = true
```

### Go
```bash
# Generate HTML report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html
```

## Coverage Thresholds

Recommended minimums:
- **Critical paths**: 90%+ (auth, payments, core logic)
- **Business logic**: 80%+
- **Utilities**: 70%+
- **Generated code**: Exclude from coverage

---

## Error Handling

### Tests Fail During Coverage

If tests fail while generating coverage:

```
Warning: X tests failed during coverage generation.

Coverage data may be incomplete. Showing partial results.

Question: "Tests failed. How should I proceed?"
Options:
  - Analyze partial coverage anyway
  - Fix tests first
  - Cancel
```

### Coverage Tool Not Installed

If the coverage tool is not found:

```
Question: "Coverage tool '[name]' is not installed. What should I do?"
Options:
  - Show installation instructions
  - Try different coverage tool
  - Cancel
```

**Installation commands:**
- Jest: Coverage built-in (`--coverage` flag)
- c8: `npm install -D c8`
- pytest-cov: `pip install pytest-cov`
- tarpaulin: `cargo install cargo-tarpaulin`
- JaCoCo: Add Maven/Gradle plugin

### No Report Generated

If coverage runs but produces no output:

```
Question: "Coverage ran but no report was generated. What should I do?"
Options:
  - Check if tests exist
  - Specify report output path
  - Run /run-tests first
  - Cancel
```

### Timeout on Large Projects

For projects with slow test suites:

```
Warning: Coverage generation is taking a long time.

Question: "How should I proceed?"
Options:
  - Continue waiting
  - Run coverage on subset of files
  - Use faster coverage tool (e.g., c8 over nyc)
  - Cancel
```
