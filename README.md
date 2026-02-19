# Agent Library

A collection of reusable agents and skills for Claude Code and VSCode Copilot. Provides autonomous workers for complex tasks like testing, code review, CI/CD debugging, refactoring, and more.

## Quick Start

Symlink the folders to your project:

```bash
ln -s ~/AgentLibrary/.claude/agents ~/yourproject/.claude/agents
ln -s ~/AgentLibrary/.claude/skills ~/yourproject/.claude/skills
ln -s ~/AgentLibrary/.claude/rules ~/yourproject/.claude/rules
```

Or for a single folder:

```bash
ln -s ~/AgentLibrary/.claude ~/yourproject/.claude
```

Then in Claude Code or VSCode Copilot:

- Agents appear in the `/agents` menu
- Skills appear in the `/skills` menu
- Rules auto-load into context

## What's Included

**14 Agents** for autonomous multi-step work:

- `test-runner`, `pr-reviewer`, `code-architect`, `bug-fixer`, `feature-implementer`
- `release-manager`, `dependency-manager`, `project-setup`
- Plus specialized agents for CI/CD, code simplification, pattern evaluation, and more

**20+ Skills** for targeted capabilities:

- Development: test execution, linting, code analysis, test generation
- Git/Release: commit management, PR creation, release notes, advanced workflows
- Context: dependency management, environment validation, API documentation

## Usage

### Agents (Complex Multi-Step Tasks)

```text
"Use code-architect to review the architecture"
"Use bug-fixer to diagnose and fix [bug description]"
"Use feature-implementer to implement [feature description]"
```

### Skills (Quick Capabilities)

```text
/run-tests
/lint-code
/commit-push
/create-pr
```

## Context System

Add `CLAUDE.md` files to your project to provide instructions:

```text
yourproject/
  CLAUDE.md              # Root instructions (always read)
  src/
    CLAUDE.md            # Feature-specific instructions
```

Instructions are discovered and applied automatically based on task context.

## Documentation

See [CLAUDE.md](./CLAUDE.md) for detailed agent/skill documentation, capabilities, and context system details.

## Requirements

- Claude Code or VSCode Copilot with Claude model
- Git repository in your project
- Symlinks must point to the correct paths
