---
name: documenter
description: Explore the current project and its documentation to update stale docs and create new documentation where it is lacking. Covers README files, inline code docs, CLAUDE.md files, API docs, and module-level documentation. Use when the user wants to audit, refresh, or fill gaps in project documentation.
argument-hint: "[scope] [--dry-run]"
---

# documenter

Explore the current project, audit existing documentation for staleness and gaps, update what's outdated, and create new documentation where it's lacking.

## Usage

```
/documenter                     # Full project documentation audit and update
/documenter src/               # Scope to a specific directory
/documenter --dry-run          # Report gaps without making changes
/documenter README             # Focus on README files only
/documenter inline             # Focus on inline code documentation only
/documenter modules            # Focus on module/directory-level docs
```

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for documentation audit. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **README files** - Search for `**/README.md`, `**/README.txt`, `**/README`
> 3. **Existing documentation** - Search for `docs/**/*.md`, `**/CLAUDE.md`, `**/*.rst`, `**/CHANGELOG.md`, `**/CONTRIBUTING.md`
> 4. **Project manifests** - Find package.json, pyproject.toml, go.mod, Cargo.toml, pom.xml, build.gradle, *.csproj, Gemfile
> 5. **Source structure** - Map top-level source directories and their contents
> 6. **Config files** - Find linter, test, build, and CI config files
>
> Return: Project name, languages, existing docs inventory, source directory map, scripts/commands available

From the Explore results, extract:
- Project name and description
- Language(s) and frameworks
- Existing documentation files (with paths)
- Source directory structure
- Available scripts/commands
- Documentation conventions already in use

### Step 2: Inventory Existing Documentation

Build a documentation inventory by scanning all discovered docs:

**For each documentation file found:**

| Field | What to Record |
|-------|----------------|
| Path | File location |
| Type | README, CLAUDE.md, API doc, module doc, changelog, etc. |
| Last modified | Git last-modified date if available |
| Staleness signals | References to removed files, outdated versions, dead links, missing exports |
| Coverage | What code/modules/features it documents |

**Staleness Detection:**

Check each doc for:
- References to files/directories that no longer exist
- Version numbers that don't match current manifests
- Commands or scripts that no longer exist in package.json/Makefile
- References to deprecated APIs, renamed exports, or removed features
- Tables of contents that don't match actual sections
- Broken relative links to other docs or source files

### Step 3: Identify Documentation Gaps

Compare the documentation inventory against the actual codebase:

**Gap Detection Checklist:**

| Area | Check | Gap If Missing |
|------|-------|----------------|
| Root README | Exists with project description, setup, usage | No root README or missing key sections |
| Module READMEs | Each top-level source directory has docs | Undocumented modules |
| Public API | Exported functions/classes have docstrings/JSDoc | Undocumented public interfaces |
| Configuration | Config files are explained somewhere | Unexplained config |
| Scripts | Available commands are documented | Undocumented scripts |
| Architecture | High-level structure is documented | No architecture overview |
| Setup/Install | Getting started instructions exist | No onboarding docs |
| Environment | Required env vars are documented | Undocumented env requirements |

**Priority Classification:**

| Priority | Criteria |
|----------|----------|
| Critical | No root README, no setup instructions, undocumented public API |
| High | Stale docs with incorrect information, missing module docs for complex modules |
| Medium | Missing inline docs for public functions, incomplete README sections |
| Low | Missing docs for simple utilities, minor staleness |

### Step 4: Present Audit Report

If `--dry-run` flag is set, present the report and stop here.

Otherwise, present the report and ask the user what to proceed with:

```
Question: "Documentation audit complete. What should I update?"
Options:
  - Fix all issues (critical → low)
  - Fix critical and high priority only
  - Let me select specific items
  - Just show the report (dry run)
```

### Step 5: Update Stale Documentation

For each stale document (in priority order):

1. **Read the current file** in full
2. **Identify specific stale sections** — don't rewrite what's still accurate
3. **Read the source code** that the doc references to get current state
4. **Apply targeted edits** using the Edit tool — preserve tone, structure, and custom content
5. **Verify links** — ensure all relative links still resolve

**Update Rules:**
- Preserve the document's existing structure and style
- Only change sections with verified staleness
- Keep `<!-- CUSTOM -->` sections untouched
- Update version numbers, file references, command examples
- Fix broken links — update path or remove if target is gone
- Add `<!-- Updated: YYYY-MM-DD -->` comment at the bottom of updated sections

### Step 6: Create Missing Documentation

For each gap (in priority order):

**Root README (if missing):**

```markdown
# [Project Name]

[Description from package.json/pyproject.toml/README or inferred from code]

## Getting Started

### Prerequisites

- [detected runtime] [version]
- [detected package manager]

### Installation

```bash
[install command]
```

### Usage

```bash
[start/run command]
```

## Development

| Command | Purpose |
|---------|---------|
| [from scripts] | [purpose] |

## Project Structure

| Directory | Purpose |
|-----------|---------|
| [detected dirs] | [inferred purpose] |

## License

[from LICENSE file if exists]
```

**Module/Directory README (if missing and module is non-trivial):**

```markdown
# [Module Name]

[Purpose inferred from code analysis]

## Overview

[Brief description of what this module does]

## Key Files

| File | Purpose |
|------|---------|
| [main files] | [description from exports/classes] |
```

**Inline Documentation (if missing on public interfaces):**

- Add JSDoc/docstrings to exported functions and classes
- Focus on: purpose, parameters, return values, thrown errors
- Match the existing documentation style in the project
- Only document public/exported interfaces, not internals

### Step 7: Verify Changes

After making changes:

1. **Check all modified files** compile/parse correctly
2. **Verify relative links** in updated docs still resolve
3. **Run any doc linters** if configured (e.g., markdownlint)

### Step 8: Report Results

Present the final report (see Output Format below).

### Step 9 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

## Output Format

```markdown
## Documentation Audit & Update Report

**Project**: [project name]
**Scope**: [full project / specific path]
**Date**: [date]

---

### Inventory

| File | Type | Status |
|------|------|--------|
| [path] | [type] | [Current / Stale / Created / Updated] |

### Changes Made

#### Updated Files

| File | Changes |
|------|---------|
| [path] | [brief description of what was updated] |

#### Created Files

| File | Reason |
|------|--------|
| [path] | [what gap it fills] |

#### Inline Docs Added

| File | Items Documented |
|------|-----------------|
| [path] | [function/class names] |

### Remaining Gaps

| Area | Priority | Description |
|------|----------|-------------|
| [area] | [priority] | [what's still missing] |

### Summary

- **Files audited**: [count]
- **Stale docs updated**: [count]
- **New docs created**: [count]
- **Inline docs added**: [count]
- **Remaining gaps**: [count]

---

**Next Steps:**
- [actionable recommendations]
```

## Error Handling

### No Documentation Found

```
Question: "No existing documentation found in this project. Should I create foundational docs?"
Options:
  - Create root README + module docs
  - Create root README only
  - Show what I would create (dry run)
  - Cancel
```

### Large Codebase

```
Question: "This codebase has [N] source directories. Should I scope the audit?"
Options:
  - Audit everything (may take a while)
  - Focus on top-level modules only
  - Let me specify directories
  - Cancel
```

### Conflicting Documentation

```
Question: "Found conflicting information between [doc1] and [doc2]. Which is authoritative?"
Options:
  - [doc1] is correct, update [doc2]
  - [doc2] is correct, update [doc1]
  - Check the source code for truth
  - Skip this conflict
```

### Ambiguous Code Purpose

```
Question: "Cannot determine the purpose of [module/file] from code alone. How should I document it?"
Options:
  - Skip it for now
  - Add a TODO placeholder
  - Let me describe it
  - Cancel
```

## Important Notes

- Always read source code before writing documentation about it — never guess
- Preserve existing documentation style and tone
- Don't over-document simple/obvious code
- Focus on "why" over "what" in documentation
- Respect `<!-- CUSTOM -->` sections in existing docs
- Inline docs should match project conventions (JSDoc vs TSDoc vs docstrings)
- Don't create docs for test files, build artifacts, or generated code
- When in doubt about accuracy, add a `<!-- TODO: verify -->` comment
