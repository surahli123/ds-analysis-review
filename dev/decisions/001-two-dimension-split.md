# ADR 001: Split Review into Two Dimensions (Analysis vs Communication)

**Date**: 2026-02-14
**Status**: Accepted
**Decider**: surahli

## Context

When designing a DS analysis review agent, we need to decide how to structure the review process. Two main options:

1. **Monolithic reviewer**: Single agent that reviews everything at once
2. **Dimension-based reviewers**: Separate agents for different aspects of the analysis

## Decision

We chose **dimension-based reviewers** with two specialized subagents:
- **analysis-reviewer**: Reviews methodology, logic, completeness, metrics
- **communication-reviewer**: Reviews narrative, audience fit, visualization, actionability

Both are orchestrated by a lead agent that routes, synthesizes, and adds top-level assessment.

## Rationale

### Why two dimensions?

Analysis and communication are **fundamentally different skills**:
- **Analysis dimension**: Requires deep DS expertise — understanding statistical rigor, metric selection, logical soundness. This is the "is it correct?" layer.
- **Communication dimension**: Requires audience awareness — what does an exec need vs a tech lead? Is the story clear? Are recommendations actionable? This is the "is it effective?" layer.

Splitting them allows:
1. **Focused prompts**: Each reviewer can specialize without trying to do everything
2. **User choice**: Users can request just analysis feedback, just communication feedback, or both
3. **Clearer feedback**: Dimension-specific strengths and improvements are easier to act on
4. **Parallel development**: We can build and test each reviewer independently

### Why not more dimensions?

We considered splitting further (e.g., methodology, visualization, narrative as separate dimensions), but:
- Two dimensions maps well to the actual review process (technical correctness vs presentation)
- More dimensions = more orchestration complexity
- Analysis naturally groups methodology + logic + completeness + metrics
- Communication naturally groups narrative + audience + viz + actionability

### Alternative considered: Monolithic reviewer

**Pros**:
- Simpler architecture (one agent instead of three)
- No synthesis step needed
- Less coordination overhead

**Cons**:
- Harder to give focused feedback (everything mixed together)
- Less flexible (can't review just one dimension)
- Prompt gets very long and complex
- Harder to test and iterate on specific aspects

## Consequences

### Positive
- Clear separation of concerns
- Users can choose which dimension(s) to review
- Each reviewer prompt can be optimized independently
- Easier to add new dimensions later if needed
- Framework (rubrics + personas) is shared, avoiding duplication

### Negative
- More complexity in orchestration (lead agent must route and synthesize)
- Potential for redundant feedback across dimensions
- Need to maintain consistency in feedback format across reviewers

### Mitigation
- Lead agent handles all orchestration complexity
- Shared framework (SKILL) ensures consistent rubrics
- Standardized output format for both reviewers makes synthesis easier

## Notes

This is a **structural decision**, not a content decision. The actual rubrics, personas, and evaluation criteria will be designed in the review framework (next sprint).
