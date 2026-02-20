---
name: audit
description: Audit game or application systems for modularity, scalability, and reusability. Evaluates systems against SOLID-like principles (single responsibility, loose coupling, high cohesion, interface-driven design, network-readiness), identifies problem areas, proposes refactoring plans with priorities, recommends architecture patterns, and generates a phased refactor roadmap. Use when the user wants to review their codebase architecture, plan a refactoring effort, or assess system modularity for multiplayer or complex projects.
---

# Audit

Audit systems for modularity, scalability, and reusability. Produces actionable refactoring plans with prioritized recommendations and a phased roadmap.

## Usage

```
/audit
/audit [system-name-or-directory]
```

Analyzes the full project by default, or a specific system/directory if provided.

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent (via Task tool with subagent_type="Explore") to discover project context:

**Explore Prompt:**
> Discover project context for a systems architecture audit. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: architecture, systems, modules, networking, multiplayer, ECS, components
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, *.csproj, *.sln, etc.
> 4. **Architecture Clues** - Look for patterns: dependency injection configs, service locators, event buses, ECS frameworks, module boundaries
>
> Return: Project language, engine/framework, existing architecture patterns, coding conventions

From the Explore results, extract:
- Project type, language, and engine/framework (e.g., Unity, Unreal, Godot, custom)
- Existing architecture patterns in use
- Any architecture guidelines from CLAUDE.md

### Step 2: Identify and Read Systems

**If specific systems/directories provided:**
- Read the source files in those systems directly using the Read tool

**If no specific target:**
- Scan the project structure to identify distinct systems (e.g., inventory, combat, networking, UI, player controller, AI, audio, save/load)
- Use Glob and Grep to find system boundaries:
  - Look for directories named `Systems/`, `Modules/`, `Services/`, `Managers/`
  - Look for classes/files with suffixes like `System`, `Manager`, `Service`, `Controller`, `Handler`
- Present the discovered systems and ask the user which to audit, or audit all

**CRITICAL: Read the actual source code.** For each system identified:
1. Use Glob to list all source files in the system directory
2. Use Read to read each source file — do NOT skip this step or guess from file names alone
3. Use Grep to trace cross-system dependencies (imports, references to other systems)
4. Use Grep to find hardcoded values, singleton patterns, static references, god classes

The audit MUST be based on reading real code, not just documentation or file names.

### Step 3: Audit Each System (Code-Level)

Read and analyze the actual implementation of each system. For each source file:
- Identify classes, their responsibilities, and line counts
- Trace dependency chains (what does this system import/reference?)
- Check for interface usage vs concrete type dependencies
- Look for state mutation patterns and ownership
- Measure function lengths and nesting depth

Evaluate against these principles:

| Principle | Question | Red Flags |
|-----------|----------|-----------|
| **Single Responsibility** | Does it do one thing well? | Mixed concerns (UI + logic), multiple unrelated features in one class |
| **Loose Coupling** | Does it avoid hard dependencies? | Direct references to concrete classes, no dependency injection, static singletons everywhere |
| **High Cohesion** | Is related logic grouped together? | Scattered related logic across files, utility grab-bags, god classes |
| **Interface-Driven** | Are contracts defined by interfaces? | No interfaces/abstract classes, concrete types passed everywhere |
| **Network-Ready** | Can it handle client/server separation? | State mutations without authority checks, no serialization strategy, tight coupling to local-only assumptions |

For each principle, assign a rating:
- **Pass** - Meets the principle well
- **Warn** - Minor issues, but functional
- **Fail** - Significant violations that will cause scaling pain

### Step 4: Identify Problem Areas

For every problem found, cite the specific file, class, and line number (e.g., `src/Combat/CombatManager.cs:142`). Include short code snippets showing the violation.

Flag systems likely to cause pain at scale:

**Critical Problems:**
- God classes (classes > 500 lines handling multiple concerns)
- Tight coupling chains (System A depends on B depends on C depends on A)
- Mixed UI + game logic in the same class
- Hardcoded values that should be data-driven
- State management without clear ownership

**Moderate Problems:**
- Missing interfaces between systems
- No event/message system for cross-system communication
- Monolithic update loops
- Inconsistent patterns across similar systems

**Minor Problems:**
- Naming inconsistencies
- Missing documentation on public APIs
- Minor code duplication across systems

### Step 5: Recommend Architecture Pattern

Based on the project's needs, recommend one or more architecture approaches:

| Pattern | Best For | Trade-offs |
|---------|----------|------------|
| **ECS (Entity-Component-System)** | Data-heavy games, many entities, performance-critical | Steep learning curve, less intuitive for simple systems |
| **Event-Driven / Message Bus** | Decoupled systems, UI updates, multiplayer sync | Harder to debug, event ordering issues |
| **Service Locator** | Moderate decoupling, easy to adopt incrementally | Can become a god container, hides dependencies |
| **Dependency Injection** | Testability, clear dependency graphs | Setup overhead, framework dependency |
| **MVC / MVVM** | UI-heavy applications, data binding | Overhead for simple systems |
| **Command Pattern** | Undo/redo, replay, network command sync | More classes, indirection |

Explain why the recommended pattern fits this specific project, considering:
- Project size and team size
- Multiplayer requirements
- Performance constraints
- Existing codebase (migration cost)

### Step 6: Propose Refactoring Plan

For each problem area identified in Step 4, suggest a concrete refactoring strategy:

```markdown
### [Problem Area Name]
**Priority**: High / Medium / Low
**Impact**: [What breaks or gets harder if not addressed]
**Strategy**: [Concrete steps to refactor]
**Estimated Scope**: [Number of files/classes affected]
**Risk**: [What could go wrong during refactoring]
```

Priority criteria:
- **High** - Blocks new feature development, causes bugs, prevents multiplayer support
- **Medium** - Increases development time, makes testing harder, causes code duplication
- **Low** - Cosmetic, improves readability, nice-to-have consistency

### Step 7: Generate Modular System Checklist

Provide a reusable checklist for building new systems:

```markdown
## New System Checklist

### Design
- [ ] System has a single, clearly defined responsibility
- [ ] Public API defined through interface/abstract class
- [ ] Dependencies injected, not created internally
- [ ] No direct references to concrete implementations of other systems
- [ ] Configuration is data-driven (not hardcoded)

### Communication
- [ ] Cross-system communication uses events/messages, not direct calls
- [ ] System does not reach into another system's internal state
- [ ] Clear ownership of shared data (who reads, who writes)

### Network Readiness
- [ ] State changes go through an authority check (server-authoritative)
- [ ] State is serializable for network sync
- [ ] Client-side prediction separated from server state
- [ ] No local-only assumptions in core logic

### Testability
- [ ] System can be instantiated in isolation for unit testing
- [ ] Dependencies are mockable
- [ ] No static/global state dependencies

### Maintenance
- [ ] Public API documented
- [ ] System follows project naming conventions
- [ ] Error handling is consistent with project patterns
```

### Step 8: Outline Refactor Roadmap

Create a phased plan that keeps the project playable throughout:

```markdown
## Refactor Roadmap

### Phase 1: Foundation (Low Risk)
**Goal**: Set up infrastructure without breaking existing code
- [ ] Introduce interfaces for existing systems
- [ ] Add event bus / message system
- [ ] Create dependency injection setup
- [ ] Add integration tests for current behavior

### Phase 2: Decouple (Medium Risk)
**Goal**: Break tight coupling between systems
- [ ] Migrate system-to-system calls to events
- [ ] Extract mixed concerns into separate classes
- [ ] Replace singletons with injected services
- [ ] Update tests for new structure

### Phase 3: Optimize (Higher Risk)
**Goal**: Restructure for scalability
- [ ] Apply chosen architecture pattern fully
- [ ] Add network abstraction layer
- [ ] Data-drive hardcoded values
- [ ] Performance profiling and optimization
```

Each phase should:
- Be independently deployable (game stays playable)
- Have clear completion criteria
- Include rollback strategy if something breaks

### Step 9: Generate Report

Output the full audit report combining all steps:

```markdown
## System Architecture Audit Report

**Project**: [name]
**Systems Audited**: [count]
**Date**: [date]

---

## Audit Summary

| System | SRP | Coupling | Cohesion | Interfaces | Network-Ready | Overall |
|--------|-----|----------|----------|------------|---------------|---------|
| [name] | Pass/Warn/Fail | ... | ... | ... | ... | Grade |

## Problem Areas
[From Step 4]

## Recommended Architecture
[From Step 5]

## Refactoring Plan
[From Step 6]

## New System Checklist
[From Step 7]

## Refactor Roadmap
[From Step 8]

## Next Steps
1. [Immediate action]
2. [Short-term action]
3. [Long-term action]
```

### Step 10 (Final): Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

---

## Interaction Guidelines

- Ask clarifying questions before diving in if the project structure is unclear
- When auditing, read actual code rather than guessing — use Grep and Read tools
- Tailor recommendations to the project's engine/framework idioms
- Be pragmatic: recommend incremental improvements over "rewrite everything"
- Consider the team's familiarity with patterns when recommending architecture
