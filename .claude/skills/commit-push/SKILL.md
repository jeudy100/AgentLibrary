# commit-push

Commit changes and push to remote in a single workflow.

## Usage

```
/commit-push
```

Options:
- `--all` or `-a` - Stage all changes before committing
- `--amend --force` - Amend previous commit and force push (use with extreme caution)
- `--no-verify` - Skip pre-commit and pre-push hooks
- `--message <msg>` or `-m <msg>` - Use provided message instead of generating one
- `--set-upstream` or `-u` - Set upstream branch (automatic for new branches)

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for committing and pushing changes. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: commit, push, remote, branch, conventional, git
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: Commit conventions, push rules, protected branches, CI triggers

From the Explore results, extract:
- Project type and tools
- Commit message conventions
- Protected branch rules
- Any push-specific instructions from CLAUDE.md

### Step 2: Perform Commit

Follow the `/commit` skill workflow:

1. Gather git state (staged, unstaged, untracked)
2. Stage changes (interactive or all with `--all`)
3. Generate commit message (conventional commit format)
4. Create commit

See `/commit` skill for detailed commit instructions.

### Step 3: Check Remote State

```bash
# Check if remote exists
git remote -v

# Get current branch
git branch --show-current

# Check if upstream is configured
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "NO_UPSTREAM"

# Check if branch exists on remote
git ls-remote --heads origin $(git branch --show-current)

# Check for divergent history
git fetch origin
git status -sb
```

### Step 3.5: Check Default Branch

Detect if on the default branch and prompt the user:

```bash
# Get current branch
CURRENT_BRANCH=$(git branch --show-current)

# Check if it's a default branch
if [ "$CURRENT_BRANCH" = "main" ] || [ "$CURRENT_BRANCH" = "master" ]; then
  echo "ON_DEFAULT_BRANCH"
fi
```

**If on default branch, ALWAYS ask:**

```
You are about to push to the default branch '[branch]'.

Question: "Would you like to push directly to [branch] or create a new branch?"
Options:
  - Push directly to [branch]
  - Create a new branch (will prompt for branch name)
  - Cancel
```

**If user chooses "Create a new branch":**
1. Prompt for branch name
2. Create the branch: `git checkout -b <new-branch-name>`
3. Continue with push workflow (Step 4)

### Step 4: Push to Remote

**New Branch (no upstream):**
```bash
git push -u origin $(git branch --show-current)
```

**Existing Branch (upstream configured):**
```bash
git push
```

**Force Push (only with --amend --force):**
```bash
# WARNING: This rewrites history on the remote
git push --force-with-lease
```

### Step 5: Verify Push

```bash
# Confirm push succeeded
git log origin/$(git branch --show-current) -1 --oneline

# Show remote tracking status
git status -sb
```

## Output Format

```
## Commit and Push Complete

**Branch**: [branch name]
**Commit**: [short hash] [commit message first line]
**Remote**: [remote URL]

## Files Changed

- [file1] (added/modified/deleted)
- [file2] (added/modified/deleted)
...

## Commit Details

[Full commit message]

## Push Status

✓ Pushed to origin/[branch]
Remote URL: [URL]

## Next Steps

- Create PR: use `/create-pr`
- View on GitHub: [URL to branch]
```

---

## Error Handling

### No Remote Configured

If `git remote` returns empty:

```
Question: "No git remote configured. What should I do?"
Options:
  - Add remote (provide URL)
  - Commit only (skip push)
  - Cancel
```

If adding remote:
```bash
git remote add origin <url>
```

### Protected Branch

For branches other than main/master (which are handled in Step 3.5), if pushing to other protected branches (develop, release/*, production):

```
Warning: You are about to push directly to '[branch]'.

Question: "This appears to be a protected branch. Continue?"
Options:
  - Push anyway (if permitted)
  - Create feature branch instead
  - Cancel
```

### Divergent History

If local and remote have diverged:

```
Warning: Your branch has diverged from the remote.

Local: [n] commits ahead
Remote: [m] commits ahead

Question: "How would you like to resolve this?"
Options:
  - Pull and rebase, then push
  - Pull and merge, then push
  - Force push (WARNING: overwrites remote)
  - Cancel
```

### Authentication Failure

If push fails due to auth:

```
## Authentication Failed

Unable to push to remote. This could be due to:
- Invalid credentials
- Expired token
- Missing SSH key

To fix:
1. Check your git credentials: `git config credential.helper`
2. For HTTPS: `gh auth login` or update stored credentials
3. For SSH: Verify your key is added to the agent: `ssh-add -l`
```

### Pre-push Hook Failure

If push fails due to hooks:

```
## Pre-push Hook Failed

[Hook output]

Options:
1. Fix the issues and run `/commit-push` again
2. Use `--no-verify` to skip hooks (not recommended)
```

### No Changes to Commit

Same as `/commit` skill - see error handling there.

---

## Safety Features

### Force Push Protection

Force push is only allowed with explicit `--amend --force` flags together:

```bash
# This is blocked:
/commit-push --force  # ERROR: --force requires --amend

# This is allowed (with warning):
/commit-push --amend --force
```

When force pushing:
```
⚠️  WARNING: Force push will overwrite remote history.

This can cause problems for collaborators who have pulled this branch.

Question: "Are you sure you want to force push?"
Options:
  - Yes, force push
  - Cancel
```

### Protected Branch Detection

Check for common protected branch names (excluding main/master which always prompt in Step 3.5):
- `develop`
- `release/*`
- `production`

Warn before pushing directly to these branches.

---

## Best Practices

### When to Use /commit-push
- Working on a feature branch alone
- Quick fixes that need immediate sharing
- After completing a logical unit of work

### When to Use /commit Only
- Making multiple related commits before pushing
- Working offline
- Want to squash/rebase commits before pushing

### Commit Frequency
- Commit often, push when ready
- Each commit should be a logical unit
- Don't push broken code

## Related Skills

- `/commit` - Create commits without pushing
- `/create-pr` - Create a pull request after pushing
- `/review-changes` - Review changes before committing
