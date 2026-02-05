# Plan: Improve commit Skill

**Current Score**: 7.5/10
**Target Score**: 8.5/10

## Issues to Fix

### 1. Clarify Explore Agent Usage (High)
Replace vague invocation with explicit guidance:
```markdown
Use the **Explore** agent (via Task tool with agent "explore") to discover project context. If Explore is unavailable or returns no results, proceed with default conventional commit conventions.
```

### 2. Add Diff Reading for Message Generation (High)
Add to Step 2:
```bash
# Get actual diff content for understanding changes
git diff --cached
```

Add to Step 4:
> To generate an accurate commit message:
> 1. Read the full `git diff --cached` output
> 2. Understand what changed (not just which files)
> 3. Determine the primary purpose of the changes

### 3. Document Breaking Changes (Medium)
Add to conventional commit types:
```markdown
**Breaking Changes:**
- Use `!` after type/scope: `feat!: remove deprecated API`
- Or add footer: `BREAKING CHANGE: <description>`
```

### 4. Add Flag Combination Rules (Medium)
Document how flags interact:
- `--all --message`: Stage all + use provided message
- `--amend --all`: Stage all + amend previous commit
- `--amend --message`: Amend with new message
- `--no-verify`: Combines with any flag

### 5. Clarify Interactive Decision Protocol (Low)
Replace Question blocks with explicit guidance on how to present options to users.

### 6. Handle Co-Author Line Conditionally (Low)
- Include when Claude generates/modifies the message
- Omit when user provides complete message via `--message`

### 7. Add Missing Error Scenarios (Low)
- Repository not found (not a git repo)
- Git not configured (missing user.email)

## Files to Modify
- `.claude/skills/commit/SKILL.md`

## Estimated Changes
- Add ~15 lines for Explore clarification
- Add ~20 lines for diff reading guidance
- Add ~15 lines for breaking changes
- Add ~20 lines for flag combinations
- Add ~15 lines for error scenarios
