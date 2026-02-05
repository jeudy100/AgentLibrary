# create-pr

Create well-formatted pull requests with proper titles, descriptions, and metadata.

## Usage

```
/create-pr
```

Creates a PR from the current branch to the default branch. You can also specify:
- `--draft` to create as a draft PR
- `--base <branch>` to target a different branch
- `--title <title>` to specify the PR title
- `--reviewers <users>` to request reviewers

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for creating pull requests. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: pull request, PR, branch, commit, conventional
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: PR conventions, description format, branch naming rules

From the Explore results, extract:
- Project type and tools
- PR title conventions
- PR description format
- Branch naming conventions
- Any PR-specific instructions from CLAUDE.md

Use project conventions to inform PR title style and description format.

### Step 2: Gather Git Context

```bash
# Current branch
git branch --show-current

# Default branch
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'

# Commits to include
git log origin/main..HEAD --oneline

# Changed files
git diff origin/main..HEAD --name-only

# Full diff
git diff origin/main..HEAD --stat
```

### Step 3: Analyze Changes

Read through the commits and changes to understand:
- What problem is being solved?
- What approach was taken?
- Are there any breaking changes?
- What testing was done?

### Step 4: Generate PR Title

Follow conventional commit style:
- `feat: Add user authentication`
- `fix: Resolve race condition in queue`
- `docs: Update API documentation`
- `refactor: Simplify payment logic`
- `test: Add integration tests for auth`
- `chore: Update dependencies`

Guidelines:
- Keep under 72 characters
- Use imperative mood ("Add" not "Added")
- Be specific but concise
- Don't end with period

### Step 5: Generate PR Description

Use this template:

```markdown
## Summary

[1-3 sentences describing what this PR does and why]

## Changes

- [Key change 1]
- [Key change 2]
- [Key change 3]

## Testing

- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing performed

### Test Instructions

[Steps to test the changes]

## Screenshots (if UI changes)

[Add screenshots here]

## Breaking Changes

[List any breaking changes, or "None"]

## Related Issues

Closes #[issue number]

---
Generated with Claude Code
```

### Step 6: Create the PR

```bash
# Basic PR
gh pr create \
  --title "feat: Add user authentication" \
  --body "$(cat <<'EOF'
## Summary

Added user authentication using JWT tokens.

## Changes

- Add login endpoint
- Add JWT token generation
- Add auth middleware

## Testing

- [x] Unit tests added
- [x] Manual testing performed

Closes #123
EOF
)"

# Draft PR
gh pr create --draft --title "..." --body "..."

# With reviewers
gh pr create --reviewer user1,user2 --title "..." --body "..."

# With labels
gh pr create --label "feature,needs-review" --title "..." --body "..."
```

### Step 7: Post-Creation

After creating:
```bash
# Get PR URL
gh pr view --web

# Add reviewers
gh pr edit --add-reviewer user1,user2

# Add labels
gh pr edit --add-label "needs-review"
```

## Output Format

```
## Pull Request Created

**Title**: [title]
**Branch**: [source] -> [target]
**URL**: [PR URL]

## PR Description

[Generated description]

## Next Steps

- [ ] Request reviews from: [suggested reviewers]
- [ ] Add labels: [suggested labels]
- [ ] Link issues: [related issues]
```

---

## When Detection Fails

### No Remote Configured

If `git remote` returns empty or origin doesn't exist:

```
Question: "No git remote found. What should I do?"
Options:
  - Add remote (provide URL)
  - Generate PR description only (copy/paste manually)
  - Cancel
```

### Default Branch Unknown

If unable to detect the default branch:

```
Question: "Can't detect default branch. Target which branch?"
Options:
  - main
  - master
  - develop
  - Custom branch name
```

### gh CLI Not Available

If `gh` command is not found:

```
Question: "GitHub CLI (gh) not found. How should I proceed?"
Options:
  - Output PR content only (create manually via web)
  - Provide installation instructions
  - Cancel
```

Installation instructions if requested:
```bash
# macOS
brew install gh

# Windows
winget install --id GitHub.cli

# Linux
# See https://github.com/cli/cli/blob/trunk/docs/install_linux.md
```

### Not Authenticated with gh

If `gh auth status` fails:

```
Question: "GitHub CLI is not authenticated. How should I proceed?"
Options:
  - Authenticate now (run gh auth login)
  - Output PR content only (create manually)
  - Cancel
```

### No Commits to Include

If current branch has no commits ahead of base:

```
Question: "No new commits found on this branch. What should I do?"
Options:
  - Create PR anyway (for discussion)
  - Compare against different branch
  - Cancel
```

### Branch Not Pushed

If current branch doesn't exist on remote:

```
Question: "Branch '[name]' hasn't been pushed to remote. Push now?"
Options:
  - Push and create PR
  - Use /commit-push to commit and push first
  - Cancel
```

**Tip:** Use `/commit-push` to commit and push changes in one step before creating a PR.

---

## Best Practices

### Good PR Titles
```
feat: Add password reset functionality
fix: Prevent duplicate form submissions
refactor: Extract payment processing logic
docs: Add API authentication guide

# Bad examples
Fixed stuff
WIP
Update index.js
Changes
```

### Good PR Descriptions
- Explain WHY, not just WHAT
- Link to related issues
- Include testing instructions
- Note any deployment considerations
- Mention breaking changes prominently

### PR Size Guidelines
- **Small**: < 200 lines (ideal)
- **Medium**: 200-400 lines (acceptable)
- **Large**: > 400 lines (consider splitting)

## Templates

Check for `.github/PULL_REQUEST_TEMPLATE.md` and follow the project's template if it exists.

## Related Skills

- `/commit` - Create well-formatted commits
- `/commit-push` - Commit and push in one step
- `/review-changes` - Review changes before committing
