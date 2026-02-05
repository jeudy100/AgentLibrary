# review-changes

Review staged and unstaged git changes, checking for common issues and providing feedback.

## Usage

```
/review-changes
```

Reviews all uncommitted changes by default. You can also specify:
- `--staged` to review only staged changes
- `--commit <sha>` to review a specific commit
- `--branch <name>` to review changes vs a branch

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for reviewing code changes. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: review, quality, security, standards, PR, code review
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: Project type, review guidelines, security requirements

From the Explore results, extract:
- Project type and tools
- Code review guidelines and standards
- Security requirements
- Any review-specific instructions from CLAUDE.md

Use the conventions from CLAUDE.md to inform what patterns to look for during review.

### Step 2: Gather Changes

```bash
# Get staged changes
git diff --cached

# Get unstaged changes
git diff

# Get all uncommitted changes
git diff HEAD

# Changes vs main branch
git diff main...HEAD

# Specific commit
git show <sha>
```

### Step 3: Get Changed Files List

```bash
# List changed files with status
git diff --name-status HEAD

# Just file names
git diff --name-only HEAD
```

### Step 4: Analyze Changes

For each changed file, check for:

**Code Quality Issues**
- [ ] Debugging code left in (console.log, print, debugger)
- [ ] Commented-out code
- [ ] TODO/FIXME without ticket reference
- [ ] Large functions (> 50 lines)
- [ ] Deep nesting (> 4 levels)
- [ ] Magic numbers without constants
- [ ] Duplicate code

**Security Issues**
- [ ] Hardcoded credentials or API keys
- [ ] SQL injection vulnerabilities
- [ ] XSS vulnerabilities
- [ ] Unsafe regex (ReDoS)
- [ ] Path traversal risks
- [ ] Insecure random number generation

**Common Mistakes**
- [ ] Missing error handling
- [ ] Unclosed resources (files, connections)
- [ ] Race conditions
- [ ] Memory leaks
- [ ] Off-by-one errors
- [ ] Null/undefined checks

**Best Practices**
- [ ] Follows project conventions
- [ ] Meaningful variable names
- [ ] Functions do one thing
- [ ] Tests included for new code
- [ ] Documentation for public APIs

### Step 5: Check Specific Patterns

```bash
# Find console.log/print statements in changes
git diff HEAD | grep -E "console\.(log|warn|error)|print\("

# Find potential secrets
git diff HEAD | grep -iE "(password|secret|api_key|token)\s*="

# Find TODO/FIXME
git diff HEAD | grep -E "(TODO|FIXME|HACK|XXX)"
```

### Step 6: Report Findings

Output in this format:

```
## Change Review

**Files Changed**: X
**Insertions**: +Y
**Deletions**: -Z

## Summary

[Brief overview of what the changes do]

## Issues Found

### Critical

#### [Issue Title]
**File**: path/to/file.ts:42
**Issue**: [Description]
**Code**:
```diff
- problematic line
+ suggested fix
```

### Warnings

#### [Issue Title]
**File**: path/to/file.ts:15
**Issue**: [Description]
**Suggestion**: [How to improve]

### Suggestions

- [Minor improvement 1]
- [Minor improvement 2]

## What's Good

- [Positive observation 1]
- [Positive observation 2]

## Checklist Before Commit

- [ ] Tests pass
- [ ] No debug code
- [ ] Error handling complete
- [ ] Documentation updated (if needed)
```

---

## When Detection Fails

### Not a Git Repository

If git commands fail because this is not a git repository:

```
Question: "This doesn't appear to be a git repository. What should I review?"
Options:
  - Specific files (provide file paths)
  - Initialize git first
  - Cancel review
```

### No Changes Detected

If `git diff` returns empty:

```
Question: "No uncommitted changes found. What should I review?"
Options:
  - Last commit (git show HEAD)
  - Specific commit (provide SHA)
  - Compare branches (provide branch names)
  - Cancel review
```

### Branch Not Found

If the specified branch doesn't exist:

```
Question: "Branch '[name]' not found. Compare against which branch?"
Options:
  - main
  - master
  - develop
  - Custom branch name
```

### Detached HEAD State

If in detached HEAD state:

```
Question: "Repository is in detached HEAD state. What should I review?"
Options:
  - Changes in current commit
  - Compare to specific branch
  - Cancel review
```

---

## Common Patterns to Flag

### Debugging Code
```javascript
// Flag these
console.log('debug', variable)
debugger
print(f"DEBUG: {value}")
```

### Potential Secrets
```javascript
// Flag these
const API_KEY = "sk-abc123..."
password = "hardcoded"
```

### Error Handling
```javascript
// Flag this (swallowed error)
try {
  doSomething()
} catch (e) {}

// Better
try {
  doSomething()
} catch (e) {
  logger.error('Failed to do something', e)
  throw e
}
```

## Tips

- Focus on meaningful issues, not style (linters handle that)
- Consider the context of changes
- Be constructive, offer solutions
- Acknowledge good patterns
