# adr-generator

Create and manage Architecture Decision Records (ADRs). ADRs document important architectural decisions, their context, and consequences for future reference.

## Usage

```
/adr-generator <command> [options]
```

Commands:
- `new <title>` - Create a new ADR
- `list` - List all existing ADRs
- `supersede <id> <new-id>` - Mark an ADR as superseded
- `link <id1> <id2> <relation>` - Link related ADRs
- `status <id> <status>` - Update ADR status

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for architecture decision records. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: architecture, decision, adr, design
> 3. **Existing ADRs** - Find docs/adr/, docs/decisions/, or similar directories
> 4. **Tech Stack** - Identify technologies used for context
>
> Return: ADR directory location, existing ADR format/template, numbering scheme, tech stack

From the Explore results, extract:
- ADR storage location
- Existing numbering format (0001, 001, 1, etc.)
- Template format if established
- Related architecture documentation

---

## Commands

### New ADR

Create a new Architecture Decision Record.

**Usage:**
```
/adr-generator new "Use PostgreSQL for primary database"
```

**Steps:**
1. Determine next ADR number
2. Generate filename: `NNNN-title-slug.md`
3. Collect context through questions
4. Generate ADR content
5. Save to ADR directory

**Directory Detection:**
```
Default locations (in order):
1. docs/adr/
2. docs/decisions/
3. docs/architecture/decisions/
4. adr/
5. decisions/

If none exist, create docs/adr/
```

**Template:**
```markdown
# ADR-NNNN: [Title]

## Status

[Proposed | Accepted | Deprecated | Superseded]

Date: YYYY-MM-DD

## Context

[Describe the issue motivating this decision. What is the problem
we're trying to solve? What constraints exist? What forces are at play?]

## Decision

[Describe the decision and the approach taken. What is the change
that we're proposing and/or doing?]

## Consequences

### Positive

- [Benefit 1]
- [Benefit 2]

### Negative

- [Drawback 1]
- [Drawback 2]

### Neutral

- [Side effect 1]

## Alternatives Considered

### [Alternative 1 Name]

[Brief description]

**Pros:**
- [Pro 1]

**Cons:**
- [Con 1]

**Why not chosen:** [Reason]

### [Alternative 2 Name]

[Brief description]

**Pros:**
- [Pro 1]

**Cons:**
- [Con 1]

**Why not chosen:** [Reason]

## References

- [Link to relevant documentation]
- [Link to discussion/RFC]
- [Link to related ADRs]
```

**Interactive Collection:**
```
Creating ADR: "Use PostgreSQL for primary database"

Question: "What is the context for this decision?"
[User provides context about data requirements, team experience, etc.]

Question: "What alternatives were considered?"
Options:
  - MongoDB
  - MySQL
  - SQLite
  - Other (specify)

Question: "What are the main consequences?"
[User provides positive and negative outcomes]
```

---

### List ADRs

Display all existing ADRs with their status.

**Output:**
```
## Architecture Decision Records

**Location**: docs/adr/

| # | Title | Status | Date |
|---|-------|--------|------|
| 0001 | Use PostgreSQL for primary database | Accepted | 2024-01-15 |
| 0002 | Adopt microservices architecture | Accepted | 2024-01-20 |
| 0003 | Use REST over GraphQL | Accepted | 2024-02-01 |
| 0004 | Switch to Redis for caching | Proposed | 2024-02-10 |
| 0005 | Use MongoDB for logging | Superseded by 0008 | 2024-02-15 |

### By Status

**Accepted** (3): 0001, 0002, 0003
**Proposed** (1): 0004
**Superseded** (1): 0005

### By Category

**Database**: 0001, 0005, 0008
**Architecture**: 0002
**API Design**: 0003
**Infrastructure**: 0004
```

---

### Supersede ADR

Mark an existing ADR as superseded by a new one.

**Usage:**
```
/adr-generator supersede 0005 0008
```

**Steps:**
1. Update old ADR status to "Superseded by [new-id]"
2. Add link to new ADR
3. Update new ADR to reference the superseded one

**Changes to ADR-0005:**
```markdown
## Status

~~Accepted~~ Superseded by [ADR-0008](0008-use-elasticsearch-for-logging.md)

Date: 2024-02-15
Superseded: 2024-03-01
```

**Changes to ADR-0008:**
```markdown
## Status

Accepted

Date: 2024-03-01
Supersedes: [ADR-0005](0005-use-mongodb-for-logging.md)
```

---

### Link ADRs

Create relationships between related ADRs.

**Usage:**
```
/adr-generator link 0002 0003 "constrains"
```

**Relation Types:**
- `supersedes` - This ADR replaces another
- `superseded-by` - This ADR is replaced by another
- `amends` - This ADR modifies another
- `constrains` - This ADR limits options for another
- `enables` - This ADR makes another possible
- `relates-to` - General relationship

**Output:**
```
## Linked ADRs

ADR-0002 (Microservices architecture) → constrains → ADR-0003 (REST API)

Updated both ADRs with cross-references.
```

---

### Update Status

Change the status of an ADR.

**Usage:**
```
/adr-generator status 0004 accepted
```

**Valid Statuses:**
- `proposed` - Under discussion
- `accepted` - Decision made and approved
- `deprecated` - No longer recommended
- `superseded` - Replaced by another ADR

**Output:**
```
## Status Updated

ADR-0004: Use Redis for caching
- Previous: Proposed
- New: Accepted
- Updated: 2024-02-15

Added acceptance date and updated status section.
```

---

## ADR Best Practices

### When to Create an ADR

Create an ADR when:
- Choosing between significant alternatives
- Making decisions that are hard to reverse
- Decisions that affect the whole team/project
- Introducing new technology or patterns
- Changing established approaches

### What Makes a Good ADR

| Element | Good | Avoid |
|---------|------|-------|
| **Title** | "Use PostgreSQL for user data" | "Database decision" |
| **Context** | Specific problem, constraints | Vague motivations |
| **Decision** | Clear, actionable statement | Implementation details |
| **Consequences** | Honest pros AND cons | Only positive outcomes |
| **Alternatives** | Serious contenders | Strawman options |

### Naming Convention

```
NNNN-short-title-slug.md

Examples:
0001-use-postgresql-for-primary-database.md
0002-adopt-microservices-architecture.md
0003-use-rest-over-graphql.md
```

---

## Output Format

### New ADR Created

```
## ADR Created

**Number**: 0006
**Title**: Use JWT for API Authentication
**File**: docs/adr/0006-use-jwt-for-api-authentication.md
**Status**: Proposed

### Summary

Decided to use JSON Web Tokens (JWT) for API authentication
instead of session-based auth to support stateless microservices.

### Next Steps

1. Review with team
2. Update status to Accepted when approved
3. Implement according to decision

[View full ADR](docs/adr/0006-use-jwt-for-api-authentication.md)
```

---

## Final Step: Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

## Error Handling

### No ADR Directory

```
Question: "No ADR directory found. Where should I create ADRs?"
Options:
  - Create docs/adr/ (recommended)
  - Create docs/decisions/
  - Specify custom location
  - Cancel
```

### Duplicate Title

```
Question: "An ADR with similar title exists (0003). What should I do?"
Options:
  - Create new ADR anyway (different focus)
  - Amend existing ADR-0003
  - Supersede ADR-0003
  - Cancel
```

### Invalid Status Transition

```
Warning: Cannot change status from 'Superseded' to 'Accepted'.

Superseded ADRs should not be reactivated. Instead:
- Create a new ADR if revisiting the decision
- The new ADR can reference the superseded one
```

### Missing Alternatives

```
Question: "No alternatives provided. A good ADR should document what was considered. What alternatives were evaluated?"
Options:
  - Add alternatives now
  - Skip alternatives (not recommended)
  - Cancel
```

## Important Notes

- ADRs are immutable once accepted - create new ones to change decisions
- Document the "why" more than the "how"
- Include consequences even if they seem obvious
- Link related ADRs to build a decision graph
- Review ADRs periodically - supersede outdated ones
- ADRs are for significant decisions, not every choice
- Keep ADRs concise - 1-2 pages maximum
- Date all status changes
- Consider ADR reviews as part of architecture review process
