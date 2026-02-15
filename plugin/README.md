# DS Analysis Review Plugin

Claude Code plugin that reviews data science analyses across two dimensions:
- **Analysis**: methodology, logic, completeness, metrics
- **Communication**: structure & TL;DR, audience fit, conciseness & prioritization, actionability

## Installation

1. Copy this `plugin/` directory to your Claude Code plugins location:
   ```bash
   cp -r plugin/ ~/.claude/plugins/ds-review/
   ```

2. Restart Claude Code or reload plugins

## Usage

### Explicit invocation
```
/ds-review:review
```

Then provide your DS analysis (Confluence page URL or local markdown file).

> **Note:** v1 supports Confluence pages and local markdown files only. Explicit invocation required — no auto-detection.

## How It Works

1. **ds-review-lead** (orchestrator) parses your invocation:
   - Mode: full or quick (default: full)
   - Target audience: exec, tech, ds, or mixed (default: mixed)
   - Workflow: proactive, reactive, or general (default: general)

2. Routes to appropriate **reviewer subagents**:
   - **analysis-reviewer**: Reviews methodology, logic, completeness, metrics
   - **communication-reviewer**: Reviews structure & TL;DR, audience fit, conciseness & prioritization, actionability

3. Synthesizes feedback into a unified review with:
   - Dimension-specific strengths and improvements
   - Top-level assessment (overall quality, biggest strengths, highest-priority improvements)
   - "Ready to share?" recommendation

## Development

See the root repository for development docs, backlog, and session logs.
