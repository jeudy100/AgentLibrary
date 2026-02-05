# Plan: Improve review-changes Skill

**Current Score**: 7.5/10
**Target Score**: 8.5/10

## Issues to Fix

### 1. Add Explicit Explore Invocation (High)
Replace vague "Use the Explore agent" with specific Task tool invocation:
```markdown
Use the **Explore** agent via the Task tool:

**Invocation:**
Task tool with agent "explore":
"Discover project context for reviewing code changes..."
```

### 2. Add Decision Logic for Git Commands (High)
Add clear criteria for which command to use:
| User Request | Command |
|--------------|---------|
| Default (no flags) | `git diff HEAD` |
| `--staged` | `git diff --cached` |
| `--commit <sha>` | `git show <sha>` |
| `--branch <name>` | `git diff <name>...HEAD` |

### 3. Add Large Diff Handling (Medium)
For diffs exceeding 500 lines:
1. Summarize with `git diff --stat HEAD`
2. Prioritize security-sensitive files
3. Sample review for very large (>2000 lines)
4. Inform user and suggest smaller commits

### 4. Fix Function Length Check (Medium)
Change from:
> "Large functions (> 50 lines)"

To:
> "If a diff hunk adds >30 lines to a single function, read the full file to check total function length"

### 5. Add Binary/Special File Handling (Low)
- Skip: Binary files, lock files, generated files
- Check carefully: .env files, config files, CI/CD files

### 6. Consolidate Duplicate Content (Low)
Merge "Common Patterns to Flag" examples into Step 4 checklists.

## Files to Modify
- `.claude/skills/review-changes/SKILL.md`

## Estimated Changes
- Add ~20 lines for Explore invocation
- Add ~15 lines for decision logic
- Add ~30 lines for large diff handling
- Add ~20 lines for special file handling
