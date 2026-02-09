# Validate Context

Validates that context documentation is properly structured for autodiscovery by Claude.

## When to Use

- After running context-generator to verify output
- When context files aren't being discovered as expected
- Before committing context changes
- To audit existing context structure

## Instructions

### Step 1: Find Context Root

Locate the context root directory:

1. **Check root CLAUDE.md** for context location config
2. **Search for context patterns**:
   ```
   docs/CLAUDE.md
   docs/context/CLAUDE.md
   .context/CLAUDE.md
   context/CLAUDE.md
   ```
3. **Search for category folders** with CLAUDE.md files

If no context root found, report: "No context structure detected."

### Step 2: Validate Root Index

Check `[context-root]/CLAUDE.md`:

| Check | Pass | Fail |
|-------|------|------|
| File exists | Root CLAUDE.md found | Missing root index |
| Has category links | Links to `[category]/CLAUDE.md` | No category links |
| Links are valid | All linked files exist | Broken links found |
| Has project info | Project name, stack summary | Missing project overview |

### Step 3: Validate Category Folders

For each folder in context root:

| Check | Pass | Fail |
|-------|------|------|
| Has CLAUDE.md | Entry point exists | Missing CLAUDE.md |
| CLAUDE.md has content | Not empty, has structure | Empty or minimal |
| Additional files linked | Extra .md files are linked from CLAUDE.md | Orphaned files |
| Cross-links valid | Links to colocated files exist | Broken cross-links |

### Step 4: Validate Colocated Context

Search for colocated context files (`**/CONTEXT.md`, `**/*CONTEXT*.md`):

| Check | Pass | Fail |
|-------|------|------|
| Links back to central | Has link to `[context-root]/[category]/CLAUDE.md` | No back-link |
| Central links to it | Category CLAUDE.md references colocated file | Orphaned colocated file |
| Consistent naming | Follows UPPERCASE.md pattern | Inconsistent naming |

### Step 5: Check Autodiscovery Readiness

Verify Claude can discover context:

| Check | Pass | Fail |
|-------|------|------|
| CLAUDE.md naming | Entry points named `CLAUDE.md` | Non-standard names |
| Reasonable depth | Max 2 levels deep from root | Too deeply nested |
| File sizes | CLAUDE.md files < 150 lines | Oversized files |
| No circular links | Link graph is acyclic | Circular references |

### Step 6: Report Results

See Output Format below.

### Step 7 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

## Output Format

```markdown
## Context Validation Report

**Context Root**: [path]
**Status**: [VALID / ISSUES FOUND / NO CONTEXT]

---

## Structure

```
[context-root]/
  CLAUDE.md                 ✓
  testing/
    CLAUDE.md               ✓
    patterns.md             ✓ (linked)
  database/
    CLAUDE.md               ✓
    migrations.md           ⚠ (not linked from CLAUDE.md)
```

## Checks

| Category | Check | Status |
|----------|-------|--------|
| Root | Index exists | ✓ |
| Root | Category links valid | ✓ |
| testing | CLAUDE.md exists | ✓ |
| testing | Additional files linked | ✓ |
| database | CLAUDE.md exists | ✓ |
| database | Additional files linked | ⚠ |

## Issues Found

### Warnings
- `database/migrations.md` not linked from `database/CLAUDE.md`

### Errors
- [none]

## Colocated Context

| File | Back-link | Central-link |
|------|-----------|--------------|
| tests/CONTEXT.md | ✓ | ✓ |
| docker/CONTEXT.md | ✓ | ✗ missing |

## Autodiscovery Score

**Score**: 8/10

- ✓ All entry points named CLAUDE.md
- ✓ Reasonable nesting depth
- ✓ File sizes within limits
- ⚠ 1 orphaned file (database/migrations.md)
- ⚠ 1 missing central link (docker/CONTEXT.md)

---

## Recommendations

1. Add link to `migrations.md` in `database/CLAUDE.md`:
   ```markdown
   ## Additional Context
   - [migrations.md](./migrations.md) - Migration workflow
   ```

2. Add link to `docker/CONTEXT.md` in `container/CLAUDE.md`:
   ```markdown
   ## Colocated Context
   - [docker/CONTEXT.md](../../docker/CONTEXT.md) - Detailed Docker setup
   ```
```

## Validation Rules

### CLAUDE.md Entry Points

Every folder in the context structure must have a `CLAUDE.md`:
- It's the autodiscovery entry point
- Must be named exactly `CLAUDE.md` (case-sensitive on some systems)
- Should contain overview and links to additional files

### Link Integrity

All links must be valid:
- Relative links preferred (`./file.md`, `../other/file.md`)
- No broken links (target must exist)
- Bidirectional linking (central ↔ colocated)

### File Organization

Keep context discoverable:
- Max 2 levels: `[root]/[category]/CLAUDE.md`
- Additional files in same folder as their CLAUDE.md
- Colocated files at tool location, not nested deeper

### Size Limits

Keep files focused:
- CLAUDE.md: Target <100 lines, max 150
- Additional files: Target <150 lines, max 200
- If larger, split into more files

## Quick Fixes

The skill can suggest automatic fixes:

| Issue | Fix |
|-------|-----|
| Missing link to additional file | Add to "Additional Context" section |
| Missing back-link in colocated | Add overview link at top |
| Orphaned colocated file | Add to category CLAUDE.md |
| Oversized CLAUDE.md | Suggest split points |

## Error Handling

### No Context Found

```
Result: No context structure detected.

Searched:
- docs/CLAUDE.md
- docs/context/CLAUDE.md
- .context/CLAUDE.md
- **/CLAUDE.md (context patterns)

Recommendation: Run context-generator to create context structure.
```

### Partial Structure

```
Result: Partial context structure found.

Found:
- docs/testing/CLAUDE.md
- docs/database/CLAUDE.md

Missing:
- docs/CLAUDE.md (root index)

Recommendation: Create root index linking to category folders.
```
