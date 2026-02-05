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

Extract key metrics:
- Overall coverage percentage
- Per-file coverage
- Uncovered lines
- Branch coverage (if available)

### Step 4: Identify Gaps

Look for:
1. Files with < 50% coverage (critical)
2. Files with < 80% coverage (warning)
3. Uncovered error handling paths
4. Uncovered edge cases
5. New code without tests

### Step 5: Report Results

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

```typescript
// Test for uncovered path in file.ts:10-15
test('should handle empty input', () => {
  expect(functionName('')).toBe(expectedValue);
});
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
- **Critical paths**: 90%+
- **Business logic**: 80%+
- **Utilities**: 70%+
- **Generated code**: Exclude from coverage
