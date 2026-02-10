# Agent-First Workflow

For every user request, always look for a matching agent before doing the work manually.

1. **Scan project-level agents** - Check `.claude/agents/` for an agent whose purpose matches the task.
2. **Scan user-level agents** - If no project-level match, check `~/.claude/agents/` and `~/.claude/skills/`.
3. **Initiate the agent** - If a suitable agent exists, launch it rather than performing the task inline.
4. **Fall back to manual** - Only handle the task directly if no relevant agent or skill exists at either level.

Project-level definitions always take precedence over user-level ones with the same name.
