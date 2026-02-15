# v1 Review Plugin Specification

**Status**: Draft
**Target**: v1.0 release

## Scope

### Must Have (v1.0)
- [ ] Working ds-review-lead orchestrator
  - Invoked via `/ds-review:review` or natural language (explicit only, no auto-detection in v1)
  - Asks user preferences (dimensions, audience)
  - Routes to appropriate subagent(s)
  - Synthesizes outputs into unified review
  - Adds top-level assessment

- [ ] Working analysis-reviewer subagent
  - Reviews methodology rigor
  - Reviews logical reasoning
  - Reviews completeness of insights
  - Reviews metric selection
  - Provides structured feedback with strengths, improvements, recommendations

- [ ] Working communication-reviewer subagent
  - Reviews structure & TL;DR
  - Reviews audience fit (exec, tech lead, peer DS)
  - Reviews conciseness & prioritization
  - Reviews actionability of recommendations
  - Provides structured feedback with strengths, improvements, recommendations

- [ ] Complete ds-review-framework SKILL
  - Rubrics for analysis dimension
  - Rubrics for communication dimension
  - Audience personas (exec, tech lead, peer DS)
  - Scoring guidelines
  - Example reviews

- [ ] Documentation
  - Plugin README with installation and usage
  - Example reviews (at least 3 sample analyses with reviews)
  - Troubleshooting guide

### Nice to Have (v1.x)
- [ ] Support for different analysis formats (slides, docs, transcripts)
- [ ] Configurable rubrics (user can customize evaluation criteria)
- [ ] Export review as markdown/PDF
- [ ] Comparison mode (review multiple versions of same analysis)

### Out of Scope (v2+)
- Interactive editing mode (agent suggests edits inline)
- Integration with presentation tools
- Automated metric validation (checking statistical calculations)
- Team collaboration features

## Success Criteria

v1.0 is ready when:
1. All three agents work end-to-end on a sample DS analysis
2. Feedback is actionable and specific (not generic)
3. Different audiences get different feedback
4. Review is reproducible (same input → similar output)
5. Plugin can be installed and used by other Claude Code users

## Testing Plan

- Test with at least 5 different DS analyses covering:
  - Different methodologies (A/B test, observational, ML model eval)
  - Different audiences (exec summary, tech deep-dive, peer review)
  - Different quality levels (strong, medium, weak)

- Validate that:
  - Explicit invocation works (command + natural language)
  - User can choose dimensions
  - Feedback is dimension-specific
  - Top-level assessment is helpful
  - Output is well-structured

## Deployment

1. **Claude Code plugin**: Install in `~/.claude/plugins/ds-review/`
2. **Agent Studio**: Copy agent prompts to Agent Studio for web distribution (no code required)

## Timeline

- v0.1: ✅ Scaffolding complete
- v0.2: Design framework and agents (next sprint)
- v0.3: Test and refine
- v0.4: Add examples and documentation
- v1.0: Public release
