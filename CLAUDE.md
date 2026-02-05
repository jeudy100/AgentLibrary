# Agent Library

A reusable library of DevOps agents and skills for Claude Code. Drop the `.claude/` folder into any project to add autonomous task capabilities.

## Quick Start

1. Copy the `.claude/` folder to your project root
2. Agents will appear in `/agents` command
3. Skills will appear in `/skills` command

## Agents

Agents are autonomous workers that handle complex, multi-step tasks. They run in isolated contexts and can use multiple skills.

| Agent | Purpose |
|-------|---------|
| `test-runner` | Runs tests, analyzes failures, suggests fixes |
| `pr-reviewer` | Reviews code changes, checks quality and security |
| `ci-cd-helper` | Debugs CI/CD pipeline issues |
| `release-manager` | Manages releases, generates changelogs |
| `code-simplifier` | Analyzes code complexity, suggests simplifications |
| `code-architect` | Analyzes architecture, detects anti-patterns, evaluates design |
| `build-validator` | Validates builds, checks dependencies, runs quality gates |
| `refiner` | Discovers and analyzes agent/skill definitions by name |
| `context-generator` | Explores codebase and generates modular context documentation |

### Using Agents

Agents are invoked via the Task tool with the agent name:
- "Run the test-runner agent to check all tests"
- "Use pr-reviewer to review my changes"
- "Use code-simplifier to simplify my recent changes"
- "Use code-architect to review the architecture"
- "Use build-validator to check if my code is ready to merge"
- "Use context-generator to document this codebase"

## Skills

Skills are inline capabilities that can be invoked directly or used by agents.

### Development Skills

| Skill | Purpose |
|-------|---------|
| `/run-tests` | Execute tests based on project type |
| `/analyze-coverage` | Analyze test coverage reports |
| `/lint-code` | Run linters based on project config |
| `/simplify-code` | Analyze code and suggest simplifications |
| `/refine` | Analyze definition quality by name, suggest improvements |
| `/skill-creator` | Create new skills with proper structure and best practices |
| `/agent-creator` | Create new agents with proper structure and best practices |

### Git/Release Skills

| Skill | Purpose |
|-------|---------|
| `/commit` | Create well-formatted commits |
| `/commit-push` | Commit and push to remote |
| `/review-changes` | Review staged/unstaged git changes |
| `/create-pr` | Create well-formatted pull requests |
| `/generate-release-notes` | Generate release notes from commits |

## Context System

The library uses the built-in **Explore** agent for context discovery.

### How Context Discovery Works

When skills and agents need context, they use the **Explore** agent with a topic-specific prompt. The Explore agent:

1. **Reads the root CLAUDE.md** (always) - base project instructions
2. **Searches for relevant CLAUDE.md files** - based on topic keywords
3. **Auto-detects project type** - from package.json, pyproject.toml, go.mod, etc.

### Context Sources

#### 1. CLAUDE.md Files (Instructions)

Place CLAUDE.md files anywhere in your project to provide instructions:

```
project/
  CLAUDE.md                    # Root - always read
  src/
    CLAUDE.md                  # Source-specific instructions
    features/
      auth/
        CLAUDE.md              # Auth module instructions
```

CLAUDE.md files can contain:
- Coding conventions and patterns
- Testing requirements
- Security guidelines
- Any project-specific instructions

**IMPORTANT**: Instructions in CLAUDE.md files are mandatory and must be followed.

### Example CLAUDE.md Structure

```markdown
# Project Instructions

## Testing
- Run `npm test` for unit tests
- Run `npm run test:e2e` for integration tests
- All new features require tests

## Code Style
- Use TypeScript strict mode
- Follow functional programming patterns
- Maximum function length: 30 lines

## Security
- Never commit secrets
- Validate all user input
- Use parameterized queries for database access
```

## Language Support

All components are language-agnostic. They detect project type from:
- `package.json` (Node.js/JavaScript/TypeScript)
- `pyproject.toml`, `setup.py`, `requirements.txt` (Python)
- `go.mod` (Go)
- `Cargo.toml` (Rust)
- `Makefile` (Make-based projects)
- `pom.xml`, `build.gradle` (Java)
- `.csproj`, `*.sln` (C#/.NET)
- `Gemfile` (Ruby)

## Directory Structure

```
.claude/
  agents/
    test-runner.md
    pr-reviewer.md
    ci-cd-helper.md
    release-manager.md
    code-simplifier.md
    code-architect.md
    build-validator.md
    refiner.md
    context-generator.md
  skills/
    run-tests/SKILL.md
    lint-code/SKILL.md
    analyze-coverage/SKILL.md
    simplify-code/SKILL.md
    refine/SKILL.md
    skill-creator/SKILL.md
    agent-creator/SKILL.md
    commit/SKILL.md
    commit-push/SKILL.md
    review-changes/SKILL.md
    create-pr/SKILL.md
    generate-release-notes/SKILL.md
```

## File Storage Rules

**Plans and temporary files must NOT be saved in the project directory.**

When creating plans, improvement documents, or any temporary working files:
- Save to the user's `.claude` folder: `~/.claude/plans/` or `~/.claude/temp/`
- On Windows: `C:\Users\<username>\.claude\plans\`
- Never create a `.claude/plans/` folder in the project
- The project's `.claude/` folder should only contain agents and skills

This keeps the project clean and prevents temporary files from being committed.

## Design Principles

1. **Language-agnostic**: Adapts to any project type
2. **Self-contained**: Just copy `.claude/` folder
3. **Composable**: Agents use skills, skills work standalone
4. **Context-aware**: Discovers context from CLAUDE.md files via Explore agent
5. **Built-in integration**: Leverages Claude Code's native Explore agent for context discovery
