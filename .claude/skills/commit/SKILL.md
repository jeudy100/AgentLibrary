# commit

Create well-formatted commits following conventional commit standards.

## Usage

```
/commit
```

Options:
- `--all` or `-a` - Stage all changes before committing
- `--amend` - Amend the previous commit (use with caution)
- `--no-verify` - Skip pre-commit hooks
- `--message <msg>` or `-m <msg>` - Use provided message instead of generating one

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for creating commits. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: commit, conventional, git, branch, message
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: Commit conventions, message format, branch naming rules

From the Explore results, extract:
- Project type and tools
- Commit message conventions
- Any commit-specific instructions from CLAUDE.md

### Step 2: Gather Git State

```bash
# Current branch
git branch --show-current

# Check if in detached HEAD state
git symbolic-ref -q HEAD || echo "DETACHED HEAD"

# Staged changes
git diff --cached --name-status

# Unstaged changes
git diff --name-status

# Untracked files (not recursive to avoid huge output)
git ls-files --others --exclude-standard

# Recent commits for context
git log --oneline -5
```

### Step 3: Stage Changes

If `--all` flag is used:
```bash
git add -A
```

Otherwise, if there are no staged changes but there are unstaged/untracked files:
```
Question: "No changes are staged. What would you like to commit?"
Options:
  - Stage all changes (git add -A)
  - Stage specific files (list files)
  - Cancel
```

If staging specific files, use:
```bash
git add <file1> <file2> ...
```

### Step 4: Generate Commit Message

If `--message` flag is provided, use the provided message.

Otherwise, analyze staged changes and generate a conventional commit message:

**Conventional Commit Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no code change |
| `refactor` | Code restructure, no feature/fix |
| `perf` | Performance improvement |
| `test` | Add/update tests |
| `chore` | Maintenance, dependencies |
| `build` | Build system changes |
| `ci` | CI configuration changes |

**Guidelines:**
- Subject line: max 50 characters, imperative mood, no period
- Body: wrap at 72 characters, explain what and why
- Scope: optional, describes the area (e.g., `auth`, `api`, `ui`)

**Examples:**
```
feat(auth): add password reset functionality

Add forgot password flow with email verification.
Users can now reset their password via email link.

Closes #123
```

```
fix: prevent duplicate form submissions

Add debounce to submit button to prevent
accidental double-clicks from creating
duplicate records.
```

### Step 5: Create Commit

```bash
# Standard commit
git commit -m "$(cat <<'EOF'
<type>(<scope>): <subject>

<body>

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"

# Amend commit (if --amend flag)
git commit --amend -m "..."

# Skip hooks (if --no-verify flag)
git commit --no-verify -m "..."
```

### Step 6: Verify Commit

```bash
# Show the commit that was just created
git log -1 --stat

# Verify working directory state
git status
```

## Output Format

```
## Commit Created

**Branch**: [branch name]
**Commit**: [short hash] [commit message first line]

## Files Changed

- [file1] (added/modified/deleted)
- [file2] (added/modified/deleted)
...

## Commit Details

[Full commit message]

## Next Steps

- Run tests to verify changes: [test command based on project]
- Push to remote: `git push` or use `/commit-push`
- Create PR: use `/create-pr`
```

---

## Error Handling

### No Changes to Commit

If `git status` shows nothing to commit:

```
No changes to commit. Working directory is clean.

To make changes:
1. Edit files in the project
2. Run `/commit` again
```

### Nothing Staged

If there are changes but nothing is staged:

```
Question: "No changes are staged for commit. What would you like to do?"
Options:
  - Stage all changes
  - Show changed files and select
  - Cancel
```

### Pre-commit Hook Failure

If commit fails due to hooks:

```
## Pre-commit Hook Failed

[Hook output]

Options:
1. Fix the issues and run `/commit` again
2. Use `--no-verify` to skip hooks (not recommended)
```

### Detached HEAD State

If in detached HEAD:

```
Warning: You are in detached HEAD state.

Question: "How would you like to proceed?"
Options:
  - Create a new branch from here
  - Return to a branch
  - Cancel
```

### Merge Conflict

If there are unresolved conflicts:

```
Cannot commit: Unresolved merge conflicts exist.

Conflicted files:
- [file1]
- [file2]

Resolve conflicts first, then run `/commit` again.
```

---

## Best Practices

### Good Commit Messages
```
feat(api): add user authentication endpoint
fix: resolve race condition in queue processing
docs: update API documentation for v2
refactor(db): extract connection pooling logic
```

### Bad Commit Messages
```
Fixed stuff
WIP
Update
Changes
asdf
```

### Commit Scope Guidelines
- Keep commits focused on a single change
- Separate refactoring from feature changes
- Don't mix formatting changes with logic changes

## Related Skills

- `/commit-push` - Commit and push in one step
- `/create-pr` - Create a pull request after pushing
- `/review-changes` - Review changes before committing
