# Plan: Improve analyze-coverage Skill

**Current Score**: 7/10
**Target Score**: 8.5/10

## Issues to Fix

### 1. Implement --report-only Flag (Critical)
Add Step 1.5 to handle `--report-only`:
- Skip test execution (Step 2)
- Locate existing coverage reports by tool:
  | Tool | Report Path |
  |------|-------------|
  | Jest | `coverage/coverage-summary.json` |
  | pytest-cov | `htmlcov/index.html` or `.coverage` |
  | Go | `coverage.out` |
  | Rust/tarpaulin | `tarpaulin-report.html` |
  | JaCoCo | `target/site/jacoco/jacoco.xml` |

### 2. Make Step 3 Actionable (High)
Replace vague "Extract key metrics" with specific parsing guidance:
- Jest: Parse `coverage/coverage-summary.json` for `total.lines.pct`
- pytest-cov: Parse stdout table format
- Go: Run `go tool cover -func=coverage.out`
- JaCoCo: Parse XML for `<counter type="LINE">`

### 3. Add Error Handling Section (High)
Cover scenarios:
- Tests fail during coverage (analyze partial report)
- Coverage tool not installed (provide install commands)
- No report generated
- Timeout on large projects

### 4. Fix Language-Specific Output Template (Medium)
Replace TypeScript-specific "Suggested Tests" example with language-agnostic placeholder.

### 5. Reconcile Threshold Values (Low)
Update Step 4 to reference the threshold section:
- Critical paths: < 90%
- Business logic: < 80%
- Utilities: < 70%

## Files to Modify
- `.claude/skills/analyze-coverage/SKILL.md`

## Estimated Changes
- Add ~40 lines for --report-only implementation
- Add ~50 lines for parsing guidance
- Add ~40 lines for error handling
- Modify ~10 lines for template fixes
