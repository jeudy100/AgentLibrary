# generate-release-notes

Generate release notes from git commits, grouped by type and formatted for release.

## Usage

```
/generate-release-notes
```

Generates release notes since the last tag by default. You can also specify:
- `--from <tag>` to start from a specific tag
- `--to <ref>` to end at a specific ref (default: HEAD)
- `--version <version>` to specify the new version number

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for generating release notes. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: release, version, changelog, tag, deploy, publish
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: Version file location, changelog format, release conventions

From the Explore results, extract:
- Project type and package manager
- Version file location
- Release conventions
- Changelog format (if specified in CLAUDE.md)

### Step 1.5: Verify Git Repository

First, verify this is a git repository:

```bash
git rev-parse --is-inside-work-tree 2>/dev/null || echo "NOT_A_GIT_REPO"
```

If not a git repository:
```
Error: This directory is not a git repository.

Cannot generate release notes without git history.
```

### Step 2: Determine Version Range

```bash
# Get latest tag
git describe --tags --abbrev=0 2>/dev/null || echo "v0.0.0"

# List all tags
git tag --sort=-version:refname | head -5

# Commits since last tag
git log $(git describe --tags --abbrev=0)..HEAD --oneline
```

### Step 3: Parse Commits

```bash
# Get detailed commit info
git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%h|%s|%an|%ae"

# Get commit bodies for breaking changes
git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%h%n%s%n%b%n---"
```

### Step 4: Categorize by Type

Parse conventional commits into categories:

| Prefix | Category | Example |
|--------|----------|---------|
| `feat:` | Features | New functionality |
| `fix:` | Bug Fixes | Bug corrections |
| `docs:` | Documentation | Doc changes |
| `style:` | Styles | Formatting |
| `refactor:` | Refactoring | Code restructure |
| `perf:` | Performance | Speed improvements |
| `test:` | Tests | Test additions |
| `build:` | Build | Build system |
| `ci:` | CI | CI changes |
| `chore:` | Chores | Maintenance |
| `revert:` | Reverts | Reverted changes |
| `BREAKING CHANGE` | Breaking | Breaking changes |

### Step 5: Extract Breaking Changes

Look for:
- `BREAKING CHANGE:` in commit body
- `!` after type (e.g., `feat!:`)
- Commit messages containing: "remove", "rename", "change signature", "incompatible", "deprecate"
- Commits modifying public APIs (files in `api/`, `public/`, exported functions)

**Note:** When uncertain whether a change is breaking, err on the side of caution and include it in Breaking Changes with a note asking the user to verify.

### Step 6: Generate Notes

Use this format:

```markdown
# [Version] (YYYY-MM-DD)

## Highlights

[1-2 sentence summary of the most important changes]

**How to generate highlights:**
1. Identify the 1-3 most impactful changes (prioritize: breaking changes > features > critical fixes)
2. Write from the user's perspective - focus on benefits, not implementation
3. Example: "This release adds multi-factor authentication and fixes a critical bug that caused data loss during exports."

## Breaking Changes

- **[scope]**: [description] ([commit])

  Migration: [how to migrate]

## Features

- **[scope]**: [description] ([commit])
- [description] ([commit])

## Bug Fixes

- **[scope]**: [description] ([commit])
- [description] ([commit])

## Performance Improvements

- [description] ([commit])

## Documentation

- [description] ([commit])

## Other Changes

- [description] ([commit])

## Contributors

- @username
- @username

**Extracting contributors:**
```bash
git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%an|%ae" | sort -u
```

**Formatting:**
- If email contains `@users.noreply.github.com`, extract username from email prefix
- Otherwise, list as "Name" without @ prefix
- If GitHub usernames cannot be determined, omit the @ prefix

---

**Full Changelog**: [link to compare]

**Constructing the compare URL:**

Detect the remote URL:
```bash
git remote get-url origin
```

Then construct the appropriate URL:
- **GitHub**: `https://github.com/owner/repo/compare/v1.0.0...v2.0.0`
- **GitLab**: `https://gitlab.com/owner/repo/-/compare/v1.0.0...v2.0.0`
- **Bitbucket**: `https://bitbucket.org/owner/repo/branches/compare/v2.0.0..v1.0.0`

If the remote URL format is unrecognized, omit the Full Changelog link.
```

### Step 7: Output

Provide the release notes in markdown format, ready to:
1. Copy to CHANGELOG.md
2. Use in GitHub release
3. Send to stakeholders

## Output Format

```
## Release Notes Generated

**Version**: v[X.Y.Z]
**Previous**: v[A.B.C]
**Commits**: [N]
**Contributors**: [M]

## Suggested Version

Based on changes:
- Breaking changes: [Y/N]
- New features: [Y/N]
- Bug fixes only: [Y/N]

**Recommended bump**: [major/minor/patch]

---

# v[X.Y.Z] (YYYY-MM-DD)

[Generated release notes in markdown]
```

---

## When Detection Fails

### No Tags Exist

If `git describe --tags` fails (no tags in repository):

```
Question: "No git tags found. What version is this release?"
Options:
  - v1.0.0 (first major release)
  - v0.1.0 (initial development release)
  - Custom version
```

If user selects custom version, prompt for the version string.

### Can't Determine Version Bump

If commits don't follow conventional format or are ambiguous:

```
Question: "Can't automatically determine version type. What kind of release is this?"
Options:
  - Major (breaking changes, new major version)
  - Minor (new features, backward compatible)
  - Patch (bug fixes only)
```

### Non-Conventional Commits

If most commits don't follow conventional commit format:

```
Question: "Commits don't follow conventional format. How should I categorize changes?"
Options:
  - Analyze commit messages manually (best effort)
  - Group all as 'Changes'
  - Ask for each significant commit
```

### No Commits Since Last Tag

If there are no new commits since the last tag:

```
Question: "No new commits since last tag ([tag]). What should I do?"
Options:
  - Generate notes for last tag
  - Specify different tag range
  - Cancel
```

### Multiple Versioning Systems

If multiple version files exist with different versions:

```
Question: "Found multiple version files with different values. Which is authoritative?"
Options:
  - package.json (v1.2.3)
  - pyproject.toml (v1.2.4)
  - VERSION file (1.2.3)
  - Use git tags only
```

### Tag Format Unknown

If existing tags don't follow semver:

```
Question: "Existing tags don't follow semver format. What format should new tags use?"
Options:
  - Semver (v1.2.3)
  - Date-based (2024.01.15)
  - Custom format
```

---

## Example Output

```markdown
# v2.1.0 (2024-01-15)

## Highlights

This release adds user authentication and fixes several critical bugs in the payment system.

## Features

- **auth**: Add JWT-based authentication (a1b2c3d)
- **api**: Add rate limiting to public endpoints (e4f5g6h)

## Bug Fixes

- **payments**: Fix race condition in transaction processing (i7j8k9l)
- **ui**: Resolve modal closing issue on mobile (m0n1o2p)

## Documentation

- Add authentication guide (q3r4s5t)
- Update API reference (u6v7w8x)

## Contributors

- @alice
- @bob

**Full Changelog**: https://github.com/org/repo/compare/v2.0.0...v2.1.0
```

## Tips

- Group related commits together
- Write for end users, not developers
- Highlight user-facing changes
- Include migration instructions for breaking changes
- Link to relevant documentation or issues
