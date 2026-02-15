# Plugin Development Conventions

## File Organization

### Agents
- One agent per file in `plugin/agents/`
- Filename should match agent name (e.g., `ds-review-lead.md` for agent named `ds-review-lead`)
- Use frontmatter for metadata:
  ```yaml
  ---
  name: agent-name
  description: Brief description
  ---
  ```

### Commands
- One command per file in `plugin/commands/`
- Use frontmatter to specify which agent handles the command:
  ```yaml
  ---
  name: command-name
  description: What this command does
  agent: agent-that-handles-it
  ---
  ```

### Skills
- One skill per directory in `plugin/skills/`
- Skill definition in `SKILL.md` within that directory
- Use frontmatter for metadata:
  ```yaml
  ---
  name: skill-name
  description: What this skill provides
  auto_activate: true/false
  ---
  ```

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

- Keep plugin/ clean (only shipping code)
- Use dev/ for work-in-progress, notes, scratch files
- Update CHANGELOG.md when shipping features
- Create ADRs for architectural decisions
