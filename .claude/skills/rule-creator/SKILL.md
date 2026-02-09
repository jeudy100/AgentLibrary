---
name: rule-creator
description: Create project rules that auto-load into Claude's context. Use this skill to persist conventions, constraints, coding patterns, or project-specific requirements as `.md` files in `.claude/rules/`. Rules are auto-discovered by Claude Code.
---

# Rule Creator

Create rules that auto-load into Claude's context from `.claude/rules/`.

## Usage

```
/rule-creator <name> <content>
```

- `name` - Rule filename (without `.md` extension)
- `content` - Rule content in markdown

## Instructions

### Step 1: Validate Input

1. **Check name**: Must be non-empty, kebab-case (`my-rule-name`). If invalid, skip with warning.
2. **Check content**: Must be non-empty and contain specific, actionable guidance. If too vague or generic, skip with warning: "Rule content too vague. Only persist specific, actionable rules."

### Step 2: Ensure Directory Exists

Check if `.claude/rules/` exists. If not, create it.

### Step 3: Check for Duplicates

Check if `.claude/rules/<name>.md` already exists.

- **If exists**: Append numeric suffix. Try `<name>-2.md`, `<name>-3.md`, etc. until a unique filename is found.
- **If not**: Use `<name>.md`.

### Step 4: Write Rule File

Write the content to `.claude/rules/<filename>.md`.

**On write failure** (permissions, disk): Report error and continue. Never block the calling workflow.

### Step 5: Confirm

Report what was created:

```
Rule created: .claude/rules/<filename>.md
```

### Step 6 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

## Error Handling

- **Directory doesn't exist**: Create `.claude/rules/` automatically.
- **Duplicate filename**: Append numeric suffix (`-2`, `-3`, etc.).
- **Empty/invalid content**: Skip with warning, continue main workflow.
- **Write failure**: Report error, continue main workflow.
- **Content too vague**: Skip with warning. Only persist specific actionable rules.

## Important Notes

- Rules auto-load into Claude's context (no manual registration needed)
- Git is the undo mechanism - unwanted rules can be reverted
- Create without prompting - auto-create and let the user review via git
- Only create rules for specific, actionable patterns - not generic advice
- Keep rules concise (under 50 lines)
