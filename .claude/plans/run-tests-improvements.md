# Plan: Improve run-tests Skill

**Current Score**: 6.2/10
**Target Score**: 8.5/10

## Issues to Fix

### 1. Add Error Handling Section (Critical)
Add comprehensive error handling for:
- Test framework not installed
- No tests found
- Test timeout (>5 minutes)
- Compilation/build failures
- Missing dependencies (node_modules, venv, etc.)

### 2. Add Missing Ruby Support (Medium)
Add Ruby/RSpec test commands to match CLAUDE.md documentation:
```bash
bundle exec rspec
bundle exec rspec spec/models/user_spec.rb
bundle exec rspec spec/models/user_spec.rb:42
```

### 3. Clarify Detection Precedence (Medium)
Add explicit priority order when multiple project files exist:
1. Explicit test command in CLAUDE.md (highest)
2. package.json scripts.test (Node.js)
3. pyproject.toml [tool.pytest] (Python)
4. go.mod (Go)
5. Cargo.toml (Rust)
6. Makefile with test target (fallback)

### 4. Add Exit Code Handling (Medium)
Document how to interpret exit codes:
- `0` = All tests passed
- `1` = Test failures occurred
- Other non-zero = Infrastructure/configuration error

### 5. Add Timeout Handling (Low)
Add guidance for handling tests that run too long.

### 6. Define Project Root (Low)
Clarify how to determine project root:
1. Directory containing project file
2. Git repository root
3. Current working directory

## Files to Modify
- `.claude/skills/run-tests/SKILL.md`

## Estimated Changes
- Add ~80 lines for error handling
- Add ~15 lines for Ruby support
- Add ~20 lines for detection precedence
- Add ~15 lines for exit codes
