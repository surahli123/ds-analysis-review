# Agent Development Conventions

> **Note:** This repository was restructured from a single-plugin model to a multi-agent repository.
> The conventions below apply to all agents in the repository, not just the ds-review plugin.

## File Organization

### Agents
- Agent-specific files go in `agents/[agent-name]/`
- Each agent directory contains:
  - Agent prompt files (e.g., `ds-review-lead.md`, `analysis-reviewer.md`)
  - Agent-specific documentation (README.md)
- Filename should match agent name or role

### Commands
- Project commands go in `.claude/commands/`
- One command per file
- Use frontmatter for configuration:
  ```yaml
  ---
  description: What this command does
  argument-hint: [arg1] [--flag]
  model: opus
  ---
  ```

### Skills
- Shared skills go in `shared/skills/[skill-name]/`
- Skill definition in `SKILL.md` within that directory
- Use frontmatter for metadata:
  ```yaml
  ---
  name: skill-name
  description: What this skill provides
  auto_activate: true/false
  ---
  ```

### Plugin Structure (ds-review)
The ds-review plugin still maintains `plugin/` for distribution:
- `plugin/commands/` — plugin command files
- `plugin/.claude-plugin/` — plugin metadata

## Agent Prompts

### Structure
1. **Role**: What is this agent's job?
2. **Context**: What does it need to know?
3. **Process**: Step-by-step instructions
4. **Output Format**: What should the output look like?
5. **Examples**: Reference examples if helpful

### Best Practices
- Be specific about what the agent should do
- Include examples of good output
- Define clear output format (markdown templates work well)
- Specify when to use subagents vs handle directly
- Include error handling (what to do when input is unclear)

## Skill Design

### Shared Knowledge Pattern
- Skills should contain **reference content**, not instructions
- Agents reference the skill for rubrics, criteria, examples
- Skills should be auto-activatable when relevant

### What Goes in a Skill
- Evaluation rubrics
- Scoring guidelines
- Persona definitions
- Domain-specific knowledge
- Example outputs

### What Doesn't Go in a Skill
- Agent-specific instructions (belongs in agent prompt)
- Orchestration logic (belongs in lead agent)
- User-facing text (belongs in command file)

## Testing

Before considering an agent "done":
1. Test with at least 3 different inputs
2. Verify output format is consistent
3. Check that it handles unclear/incomplete input gracefully
4. Validate that shared skills are being used correctly

## Documentation

Every agent/command/skill should have:
- Clear description in frontmatter
- Comments explaining non-obvious logic
- Examples of expected input/output
- Notes about edge cases

## Version Control

- Keep `agents/` and `shared/` clean (only shipping code)
- Use `dev/` for work-in-progress, notes, scratch files
- Update CHANGELOG.md when shipping features
- Create ADRs for architectural decisions
