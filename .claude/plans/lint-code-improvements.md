# Plan: Improve lint-code Skill

**Current Score**: 7.5/10
**Target Score**: 9/10

## Issues to Fix

### 1. Add Error Handling Section (Critical)
Add handling for:
- Linter not installed (with install commands)
- Invalid configuration file
- Permission errors
- Timeout on large codebases
- Exit code interpretation

### 2. Add "No Linter Found" Handling (High)
When no linter configuration exists:
1. Check for common config files
2. Suggest appropriate linter based on project type
3. Report: "No linter configured. Consider adding [suggested linter]."

### 3. Add Linter Priority Rules (Medium)
Define selection order when multiple linters exist:

**JavaScript/TypeScript:**
1. package.json scripts.lint → use that
2. Biome (if biome.json exists)
3. ESLint (if .eslintrc* exists)
4. Prettier (formatting only)

**Python:**
1. pyproject.toml configured tool
2. Ruff (preferred)
3. Flake8 + Black (legacy)

### 4. Add Missing Language Support (Medium)
Add Java and C#/.NET linting:
- Java: Checkstyle, SpotBugs
- C#: dotnet format, StyleCop

### 5. Improve Usage Examples (Low)
Add explicit syntax examples:
```
/lint-code                    # Lint entire project
/lint-code src/utils.ts       # Lint specific file
/lint-code --fix              # Auto-fix issues
/lint-code src/ --fix         # Lint directory with auto-fix
```

## Files to Modify
- `.claude/skills/lint-code/SKILL.md`

## Estimated Changes
- Add ~60 lines for error handling
- Add ~30 lines for "no linter" handling
- Add ~40 lines for linter priority rules
- Add ~30 lines for Java/C# support
