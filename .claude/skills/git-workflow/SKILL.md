# git-workflow

Advanced git operations for complex version control tasks. Handles cherry-pick, rebase, squash, bisect, stash, and branch management.

## Usage

```
/git-workflow <operation> [options]
```

Operations:
- `cherry-pick` - Apply specific commits to current branch
- `rebase` - Rebase current branch onto another
- `squash` - Squash multiple commits into one
- `bisect` - Find commit that introduced a bug
- `stash` - Manage stashed changes
- `cleanup` - Clean up merged/stale branches

## Instructions

### Step 0: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for git workflow. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: git, branch, commit, merge, rebase
> 3. **Git Configuration** - Check for .gitconfig, branch protection rules
>
> Return: Git workflow preferences, branch naming conventions, protected branches, commit conventions

From the Explore results, extract:
- Branch naming conventions
- Protected branches (main, master, develop)
- Rebase vs merge preference
- Commit message conventions

---

## Operations

### Cherry-Pick

Apply specific commits from one branch to another.

**Usage:**
```
/git-workflow cherry-pick <commit-hash> [--no-commit]
```

**Steps:**
1. Verify current branch and status
2. Show commit details for confirmation
3. Execute cherry-pick
4. Handle conflicts if any

**Commands:**
```bash
# Show commit to be cherry-picked
git show <commit-hash> --stat

# Cherry-pick single commit
git cherry-pick <commit-hash>

# Cherry-pick without committing (stage only)
git cherry-pick <commit-hash> --no-commit

# Cherry-pick range of commits
git cherry-pick <start-hash>^..<end-hash>

# If conflicts occur
git status
# After resolving:
git cherry-pick --continue
# Or abort:
git cherry-pick --abort
```

---

### Rebase

Rebase current branch onto another branch.

**Usage:**
```
/git-workflow rebase <target-branch> [--onto <newbase>]
```

**Steps:**
1. Check for uncommitted changes (stash if needed)
2. Show commits that will be rebased
3. Execute rebase
4. Handle conflicts step by step

**Commands:**
```bash
# Show what will be rebased
git log --oneline HEAD ^<target-branch>

# Standard rebase
git rebase <target-branch>

# Rebase onto specific commit
git rebase --onto <newbase> <upstream> <branch>

# If conflicts occur
git status
# After resolving each:
git add <resolved-files>
git rebase --continue
# Or abort:
git rebase --abort
```

**Conflict Resolution Output:**
```
## Rebase Conflict

**Rebasing**: commit abc123 "commit message"
**Conflict in**: [file paths]

### Conflict Details

[Show conflict markers and context]

### Options

1. Keep our changes (current branch)
2. Keep their changes (incoming)
3. Manual merge (show both)
```

---

### Squash

Combine multiple commits into a single commit.

**Usage:**
```
/git-workflow squash <number-of-commits> [-m "message"]
```

**Steps:**
1. Show commits to be squashed
2. Confirm with user
3. Perform soft reset and recommit
4. Use provided message or combine messages

**Commands:**
```bash
# Show commits to squash
git log --oneline -n <number>

# Squash last N commits (soft reset method - safer than interactive rebase)
git reset --soft HEAD~<number>
git commit -m "combined commit message"

# Alternative: Interactive rebase
git rebase -i HEAD~<number>
# Then mark commits as 'squash' or 's'
```

**Output:**
```
## Squash Preview

**Commits to squash** (oldest first):

1. abc1234 - First commit message
2. def5678 - Second commit message
3. ghi9012 - Third commit message

**Combined into**:
> [generated or provided commit message]

Proceed with squash? [Yes/No]
```

---

### Bisect

Find the commit that introduced a bug using binary search.

**Usage:**
```
/git-workflow bisect --good <commit> --bad <commit> [--test <command>]
```

**Steps:**
1. Start bisect session
2. Mark known good and bad commits
3. Test each midpoint (manually or with command)
4. Report the culprit commit

**Commands:**
```bash
# Start bisect
git bisect start

# Mark bad commit (usually HEAD)
git bisect bad <commit>

# Mark last known good commit
git bisect good <commit>

# After testing each commit:
git bisect good  # if commit is good
git bisect bad   # if commit is bad

# Automated bisect with test command
git bisect run <test-command>

# Show current bisect status
git bisect log

# End bisect and return to original branch
git bisect reset
```

**Output:**
```
## Bisect Progress

**Range**: abc123 (good) → def456 (bad)
**Remaining**: ~7 commits to test
**Current**: ghi789

### Testing Commit

**Hash**: ghi789
**Author**: John Doe
**Date**: 2024-01-15
**Message**: Add new feature

**Changes**:
- src/feature.ts (+50, -10)
- tests/feature.test.ts (+20)

Is this commit good or bad?
```

---

### Stash

Manage stashed changes.

**Usage:**
```
/git-workflow stash [list|save|pop|apply|drop|show]
```

**Commands:**
```bash
# List all stashes
git stash list

# Save with message
git stash push -m "description of changes"

# Save specific files
git stash push -m "message" -- <file1> <file2>

# Apply most recent stash (keep in stash list)
git stash apply

# Apply and remove from stash list
git stash pop

# Apply specific stash
git stash apply stash@{n}

# Show stash contents
git stash show -p stash@{n}

# Drop specific stash
git stash drop stash@{n}

# Clear all stashes
git stash clear
```

**Output for list:**
```
## Stashed Changes

| # | Age | Branch | Message | Files |
|---|-----|--------|---------|-------|
| 0 | 2h  | main   | WIP: auth feature | 3 files |
| 1 | 1d  | feature| Debugging changes | 1 file |
| 2 | 3d  | main   | Temp fix | 2 files |

Commands:
- Apply: `git stash apply stash@{n}`
- Pop: `git stash pop stash@{n}`
- Show: `git stash show -p stash@{n}`
- Drop: `git stash drop stash@{n}`
```

---

### Cleanup

Clean up merged and stale branches.

**Usage:**
```
/git-workflow cleanup [--dry-run] [--remote]
```

**Steps:**
1. Fetch latest from remote
2. Identify merged branches
3. Identify stale branches (no commits in 30+ days)
4. Confirm deletion with user

**Commands:**
```bash
# Fetch and prune remote-tracking branches
git fetch --prune

# List merged branches (excluding protected)
git branch --merged main | grep -v "main\|master\|develop"

# List branches with last commit date
git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) %(committerdate:relative)'

# Delete local branch
git branch -d <branch-name>

# Delete remote branch
git push origin --delete <branch-name>
```

**Output:**
```
## Branch Cleanup

### Merged Branches (safe to delete)

| Branch | Merged Into | Last Commit |
|--------|-------------|-------------|
| feature/auth | main | 5 days ago |
| fix/typo | main | 2 weeks ago |

### Stale Branches (no activity 30+ days)

| Branch | Last Commit | Author |
|--------|-------------|--------|
| experiment/test | 45 days ago | John |
| wip/old-feature | 60 days ago | Jane |

### Protected Branches (will not delete)

- main
- develop

Delete merged branches? [Yes/No/Select]
```

---

## Error Handling

### Uncommitted Changes

```
Question: "You have uncommitted changes. What should I do?"
Options:
  - Stash changes and continue
  - Commit changes first
  - Discard changes (DANGEROUS)
  - Cancel
```

### Merge Conflicts

```
Question: "Merge conflicts occurred in [files]. How should I resolve?"
Options:
  - Show conflict details
  - Keep current branch changes (ours)
  - Keep incoming changes (theirs)
  - Abort operation
```

### Protected Branch

```
Question: "Cannot modify protected branch [branch]. What should I do?"
Options:
  - Create a new branch and apply changes there
  - Cancel
```

### Diverged Branches

```
Question: "Local and remote branches have diverged. How should I proceed?"
Options:
  - Rebase local onto remote
  - Merge remote into local
  - Force push local (DANGEROUS - may lose remote changes)
  - Cancel
```

### Bisect Test Ambiguous

```
Question: "Cannot determine if this commit is good or bad. What should I do?"
Options:
  - Skip this commit (git bisect skip)
  - Mark as good
  - Mark as bad
  - End bisect early
```

## Important Notes

- NEVER force push to protected branches (main, master, develop)
- Always confirm before destructive operations (reset, clean, force push)
- Stash or commit changes before rebase/checkout operations
- Use `--dry-run` when available to preview changes
- Keep bisect sessions short - reset if interrupted
- When in doubt, create a backup branch before complex operations
- Respect project's branch naming conventions
- Check with team before cleaning up shared remote branches
