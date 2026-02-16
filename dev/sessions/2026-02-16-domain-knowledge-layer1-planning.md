# Session: Domain Knowledge Layer 1 — MVP Design & Implementation Planning

**Date:** 2026-02-16
**Duration:** ~2 hours (across web session + Claude Code)
**Branch:** main (planning only, no code changes)

## Goal

Complete the design and create an implementation plan for Layer 1 of the Domain Knowledge Skill — the standalone, reusable knowledge service that any agent can consume.

## What Happened

### Phase 1: MVP Design Review (Web Session)

PM-led design review of the v3 spec identified 5 implementation challenges for Layer 1. Resolved each with scoped-down decisions:

1. **A1 — API as file contract:** Dropped callable `get_domain_context()` function. Layer 1 is digest files + SKILL.md format contract. Consumers just use Read tool.
2. **A2 — Audience tags + token budget:** Increased from 5K → 8K tokens/domain. Added `[audience: all|ds|eng]` tags so different consumers filter relevant sections.
3. **A3 — Manual-only refresh:** No auto-discovery, no cron, no `refresh.sh`. Refresh triggered explicitly via `--refresh-domain` in Claude Code session.
4. **A4 — Drop team roster:** LLM content scoring (`review_impact × 0.6 + knowledge_density × 0.4`) is sufficient. No `team-roster.yaml` in v0.5.
5. **A5 — Additive cross-domain budget:** Cross-domain gets 1,500 tokens additive (not shared with domain budgets). Single domain = 8K + 1.5K = 9.5K total.

**Output:** `docs/plans/2026-02-16-domain-knowledge-mvp-design.md`

### Phase 2: Implementation Planning (Claude Code)

Created detailed 5-task implementation plan with full file contents:

| Task | Files | Content |
|---|---|---|
| 1 | `plugin/config/domain-index.yaml` | Curated index: 2 sub-domains (ranking, QU) + cross-domain. Placeholder page IDs. |
| 2 | `plugin/skills/domain-knowledge/SKILL.md` | 6-section contract: digest format, consumption guide, staleness, refresh, scoring, precedence |
| 3 | `plugin/digests/search-ranking.md` | Real foundational knowledge (metrics, position bias, experiments, click models, LTR, latency) + workstream placeholders |
| 4 | `plugin/digests/query-understanding.md`, `search-cross-domain.md` | QU evaluation standards + cross-component evaluation knowledge |
| 5 | Validation | YAML parsing, path cross-checks, contract compliance, backlog/session updates |

**Output:** `docs/plans/2026-02-16-domain-knowledge-layer1-implementation-plan.md`

### Phase 3: Session Wrap-Up (Claude Code)

- Copied MVP design doc from `DS-Analysis-Review-Agent` workspace to main repo
- Saved implementation plan to main repo
- Updated CHANGELOG with domain expert agent effort (design + planning summary)
- Updated backlog with Layer 1 planning status and next steps
- Created this session log

## Files Created / Modified

### Created
- `docs/plans/2026-02-16-domain-knowledge-mvp-design.md` — MVP design decisions
- `docs/plans/2026-02-16-domain-knowledge-layer1-implementation-plan.md` — 5-task implementation plan
- `dev/sessions/2026-02-16-domain-knowledge-layer1-planning.md` — this file

### Modified
- `CHANGELOG.md` — Added v0.5 domain expert agent section under [Unreleased]
- `dev/backlog.md` — Updated v0.5 section with planning completion + Layer 1 next steps

## Key Decisions

| Decision | Rationale |
|---|---|
| Layer 1 only in implementation plan | Validate digest format + content before building reviewer subagent |
| search-infra deferred | Least useful domain for DS review MVP |
| No scripts or dependencies | Markdown + YAML only. Keep it simple for v0.5. |
| Real domain knowledge in ranking digest | Foundational content is genuine Search expertise, not placeholder. Workstream sections are placeholders. |
| Plan lives in `docs/plans/`, not executed yet | User chose to plan only, not implement in this session |

## Next Session

1. **Execute Layer 1 plan** — Open `docs/plans/2026-02-16-domain-knowledge-layer1-implementation-plan.md` and implement all 5 tasks
2. Use `superpowers:executing-plans` skill for task-by-task execution
3. After Layer 1 validated: proceed to Layer 2 (Domain Expert Reviewer subagent)
