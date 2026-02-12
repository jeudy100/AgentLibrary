# Documentation Sync on Code Changes

Whenever code files are **created, updated, or deleted** during a session, check for existing documentation that may need corresponding updates.

## When to Check

- **Create**: New file, module, component, endpoint, agent, or skill added
- **Update**: Existing code modified (renamed exports, changed signatures, altered behavior)
- **Delete**: File, module, component, endpoint, agent, or skill removed

## What to Check

1. **Root CLAUDE.md** — tables, directory trees, quick-start guides, feature lists
2. **Colocated CLAUDE.md files** — instructions in the same or parent directories
3. **README files** — any README.md at project root or in affected directories
4. **API docs** — OpenAPI specs, JSDoc, docstrings referenced by the change
5. **Config references** — if the change affects entries in package.json scripts, Makefile targets, CI configs, or similar

## How to Apply

- After completing code changes, scan the files listed above for stale references.
- If documentation references the changed code (by name, path, or behavior), update it to stay consistent.
- If a new feature or component has no documentation entry where peers are documented, add one.
- Do **not** create new documentation files unprompted — only update existing ones.
