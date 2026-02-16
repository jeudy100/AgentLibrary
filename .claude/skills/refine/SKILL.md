---
name: refine
description: Analyze an agent or skill definition by name and suggest improvements for clarity, structure, and error handling.
---

# refine

Analyze an agent or skill definition by name and suggest improvements for clarity, structure, and error handling.

## Usage

```
/refine <name>
/refine test-runner
/refine commit
/refine --path <file-path>
```

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent (via Task tool with subagent_type="Explore") to discover project context:

**Explore Prompt:**
> Discover project context for analyzing definitions. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Definition Locations** - Search for `.claude/agents/*.md` and `.claude/skills/*/SKILL.md`
> 3. **Style Guides** - Search `**/CLAUDE.md` for keywords: agent, skill, definition, template, convention
>
> Return: Definition locations, naming conventions, style guidelines

From the Explore results, extract:
- Where agent definitions are stored
- Where skill definitions are stored
- Any naming conventions or templates to follow
- Style guidelines from CLAUDE.md files

### Step 2: Locate the Definition

Given the name provided by the user:

**If `--path` flag provided:**
- Use the exact path specified
- Verify the file exists

**Otherwise, search by name:**

1. **Exact name match:**
   - `**/{name}.md`
   - `**/{name}/SKILL.md`

2. **Partial match:**
   - `**/*{name}*.md`

3. **Content search:**
   - Grep for `# {name}` (case-insensitive)

**Common locations to check:**
- `.claude/agents/{name}.md`
- `.claude/skills/{name}/SKILL.md`
- `agents/{name}.md`
- `skills/{name}/SKILL.md`
- `docs/agents/{name}.md`
- `commands/{name}.md`

### Step 3: Determine Definition Type

Based on file content:

| Pattern Found | Type |
|---------------|------|
| `## Tools Available` | Agent definition |
| `## Usage` with `/` prefix | Skill definition |
| `## Instructions` with steps only | Check other markers |
| Both patterns | Hybrid (analyze as both) |

### Step 4: Check Structural Completeness

Verify required sections are present:

**Agent Checklist:**
- [ ] Header with clear purpose
- [ ] Tools Available section
- [ ] Skills to Use section (recommended)
- [ ] Instructions with numbered steps
- [ ] Step 0: Context Discovery
- [ ] Output Format section
- [ ] Error Handling with Questions/Options
- [ ] Important Notes (recommended)

**Skill Checklist:**
- [ ] Header with clear purpose
- [ ] Usage section with examples
- [ ] Instructions with numbered steps
- [ ] Step 1: Context Discovery
- [ ] Output Format section
- [ ] Error Handling with Questions/Options
- [ ] Related Skills (recommended)

### Step 5: Assess Quality

Evaluate each dimension:

| Criterion | Check For |
|-----------|-----------|
| Clarity | Unambiguous instructions, specific actions |
| Conciseness | No redundant content, appropriate length |
| Step Structure | Numbered steps, clear sequence, logical flow |
| Error Resilience | Failure scenarios have user questions |
| User Fallbacks | Options provided (not open-ended questions) |
| Examples | Before/after or usage examples where helpful |

**Rating Definitions:**

| Rating | Criteria |
|--------|----------|
| High | Fully meets expectations, no issues |
| Medium | Mostly meets expectations, minor issues |
| Low | Significant gaps or ambiguities |

**Overall Quality Mapping:**
- **High**: All criteria rated High or Medium, no Critical issues
- **Medium**: At least one Low rating or one Critical issue
- **Needs Improvement**: Multiple Low ratings or multiple Critical issues

### Step 6: Generate Suggestions

See below for how to present suggestions.

### Step 7 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

### Suggestion Details

For each issue found:

1. **Identify** the problem with specific location
2. **Show current** content (or note "Missing")
3. **Provide suggested** improvement with exact text
4. **Explain why** the change helps

**Priority levels:**
- **Critical**: Missing required sections, no error handling
- **Warning**: Incomplete sections, vague instructions
- **Suggestion**: Style improvements, additional examples

## Output Format

```markdown
## Definition Analysis: [name]

**File**: [discovered path]
**Type**: [Agent / Skill]
**Overall Quality**: [High / Medium / Needs Improvement]

---

## Structural Completeness

| Section | Status | Notes |
|---------|--------|-------|
| Header + Purpose | [✓/✗] | [notes] |
| Usage (skills) | [✓/✗/N/A] | [notes] |
| Tools Available (agents) | [✓/✗/N/A] | [notes] |
| Instructions | [✓/Partial/✗] | [notes] |
| Context Discovery Step | [✓/✗] | [notes] |
| Output Format | [✓/✗] | [notes] |
| Error Handling | [✓/Partial/✗] | [notes] |
| Examples | [✓/Partial/✗] | [notes] |

---

## Quality Assessment

| Criterion | Rating | Details |
|-----------|--------|---------|
| Clarity | [High/Med/Low] | [assessment] |
| Conciseness | [High/Med/Low] | [assessment] |
| Step Structure | [High/Med/Low] | [assessment] |
| Error Resilience | [High/Med/Low] | [assessment] |
| User Query Fallbacks | [High/Med/Low] | [assessment] |

---

## Improvements

### [Priority: Critical/Warning/Suggestion]: [Issue Title]

**Current:**
```markdown
[existing content or "Missing"]
```

**Suggested:**
```markdown
[improved content]
```

**Why**: [Explanation of improvement]

---

## Summary

**Strengths:**
- [positive observations]

**Areas to Improve:**
- [key improvements needed]

**User Fallback Check:**
- [✓/✗] Has fallback questions when definition not found
- [✓/✗] Has fallback questions when ambiguous input
- [✓/✗] Has fallback questions when errors occur
- [✓/✗] Offers clear options to user (not open-ended)
```

## Example Analysis (Abbreviated)

For `/refine commit`:

```markdown
## Definition Analysis: commit

**File**: .claude/skills/commit/SKILL.md
**Type**: Skill
**Overall Quality**: High

## Quality Assessment

| Criterion | Rating | Details |
|-----------|--------|---------|
| Clarity | High | Clear conventional commit format with examples |
| Error Resilience | High | 5 error scenarios with user options |

## Improvements

### [Suggestion]: Add git config check

**Current:** Missing

**Suggested:**
Add to Step 2: `git config user.email` to verify git is configured.

**Why**: Prevents cryptic errors if user hasn't configured git.
```

---

## Error Handling

### Definition Not Found

```
Question: "Could not find a definition named '[name]'. How should I proceed?"
Options:
  - Search with different keywords
  - Provide the file path directly
  - List all available definitions
  - Cancel
```

### Multiple Matches Found

```
Question: "Found multiple definitions matching '[name]'. Which one should I analyze?"
Options:
  - [path1] - [brief description]
  - [path2] - [brief description]
  - Analyze all matches
  - Cancel
```

### Cannot Determine Type

```
Question: "Cannot determine if this is an agent or skill definition. What is it?"
Options:
  - Agent definition
  - Skill definition
  - Analyze as both
  - Cancel
```

### Empty or Minimal File

```
Question: "The definition '[name]' appears empty or incomplete. What would you like to do?"
Options:
  - Generate a template for [Agent/Skill]
  - Show what's missing
  - Cancel
```

### File Access Error

```
Question: "Cannot read the file at '[path]'. The file may be inaccessible or corrupted."
Options:
  - Try a different path
  - Check file permissions
  - Cancel
```

## Related Skills

- `/simplify-code` - Simplify complex code
- `/review-changes` - Review code changes
- `/lint-code` - Run linters on code
