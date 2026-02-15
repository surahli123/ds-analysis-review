# Changelog

All notable changes to the DS Analysis Review Agent.

## [Unreleased]

### v0.4.0 — Real-World Testing & Calibration Review (2026-02-15)

**Status:** In progress. 3 real-world tests completed. All 3 calibration perspectives collected (Engineer, PM, DS Lead). Awaiting owner synthesis and decision before implementation.

#### Added
- `dev/test-fixtures/real/` — 12 real-world fixtures collected (gitignored):
  meta-llm-bug-reports, rossmann-sales-prediction, vanguard-ab-test,
  capstone-customer-churn, credit-card-churn-segmentation, tips-regression-analysis,
  kaggle-house-prices-eda, kaggle-titanic-solutions, kaggle-ibm-churn-lowquality,
  meta-posttreatment-variables, meta-llm-product-analytics, meta-asymmetric-experiments
- `dev/test-results/2026-02-15-meta-llm-bug-reports-review.md` — full review output (18/100)
- `dev/test-results/2026-02-15-vanguard-ab-test-full-review.md` — full review output (16/100)
- `dev/test-results/2026-02-15-calibration-notes.md` — root cause analysis across 3 tests
- `dev/test-results/2026-02-15-principal-ai-engineer-assessment.md` — scoring math diagnosis + diminishing returns proposal
- `dev/reviews/2026-02-15-pm-lead-calibration-review.md` — 6 critiques, 4 proposed fixes
- `dev/test-results/2026-02-15-ds-lead-assessment.md` — finding-by-finding audit, independent scoring, 6 proposed fixes

#### Tested
- 3/12 real-world fixtures tested (Meta blog 18/100, Rossmann 29/100, Vanguard 16/100)
- All three scored far below expert intuition (expected range: 40-65)
- Pipeline mechanics confirmed working: subagent dispatch, dimension separation, deduction table adherence, floor rules all correct
- Findings are often genuinely insightful — the scoring is the problem, not the analysis quality

#### Identified (Calibration Issues — from 3 independent assessments)
1. Scoring math saturates — additive deductions with no diminishing returns compress at ~15 findings
2. CRITICAL over-assigned — structural issues (missing TL;DR, no story arc) treated same as analytical validity threats
3. No strength credits — "What You Did Well" has zero impact on score, causing differentiation failure (Vanguard 16 vs Meta 18 when Vanguard is clearly better analytically)
4. Cross-cutting issues double-count through different lenses
5. Output too long and leads with negatives
6. Genre-inappropriate findings (DS Lead: 6 of 15 Meta findings are wrong for a blog post)
7. Too many findings generated (DS Lead: 15 findings per doc overwhelms scoring and user)
8. 3 bugs in review output (severity/deduction mismatches, routing table violation)

#### Three Assessment Summary

| Perspective | Primary Fix | Strength Cap | CRITICAL Approach | Unique Insight |
|---|---|---|---|---|
| Principal AI Engineer | Diminishing returns formula | Not proposed | TL;DR + story arc → MAJOR | Correlated findings treated as independent |
| PM Lead | Diminishing returns + strength credits | +15/dimension | + limitations → MAJOR | Tool has a communication problem itself |
| DS Lead | Strength credits (primary) + diminishing returns (safety net) | +25/dimension | + recommendations → MAJOR | Finding quality matters, not just scoring math; cap at 8 total findings |

#### UX Decisions (Owner Approved)
- Compress per-lens sections by default (1-2 sentences, full detail only in Top 3). No `--detail` flag — just make default output shorter.
- Add emoji indicators to lens dashboard (SOUND -> ✅, MINOR -> ⚠️, MAJOR/CRITICAL -> ❌)
- Rewrite examples in Top 3: use `> **✏️ Suggested rewrite:**` blockquote format. One format, no alternatives listed.

#### Synthesized — A3 Fix Plan + Calibration Loop Workflow (2026-02-15)

- `dev/test-results/2026-02-15-calibration-fix-plan.md` — A3 Problem Analysis synthesizing all 3 role reviews:
  - 4 consensus root causes (no strength credits, deduction stacking, CRITICAL over-assignment, teardown output)
  - 5 disagreements surfaced with tradeoffs (credit cap, Meta target, finding cap, reclassification scope, CRITICAL gradation)
  - 6 fixes in 2 phases with exact file paths and specific changes
  - Projected score impact tables for each phase
- `docs/plans/2026-02-15-calibration-loop-workflow.md` — Repeatable 8-task calibration loop:
  - Fix → Test 3 fixtures → Write test notes → Write calibration notes → Compare to prior round → 3 role reviews (parallel) → A3 synthesis → Owner decision
  - File-based state with `rN` round numbering (session-boundary safe)
  - Quantitative acceptance criteria (score ranges, differentiation gaps, CRITICAL counts)
  - Round checklist template, session management, estimated 1-3 rounds to converge

#### Owner Decisions (5/5 Resolved)

| # | Decision | Choice |
|---|---|---|
| 1 | Strength credit cap | +25 per dimension |
| 2 | Meta target score | 42-50 (Minor Fix) |
| 3 | Finding volume cap | Cap at 10, defer to Phase 2 |
| 4 | Severity escalation bug | Fix — prevent escalation beyond deduction table |
| 5 | Diminishing returns | Yes, include in Phase 1 |

#### Next Step
Execute fix plan (Phase 1: strength credits, CRITICAL reclassification, diminishing returns, 2 bug fixes) → then start calibration loop Round 1 to validate fixes against 3 test fixtures.

### v0.3.0 — Implementation Complete (2026-02-15)

**Status:** Complete. All agents, rubrics, command, and test fixtures implemented. Smoke tests pass 59/59.

#### Added
- `plugin/skills/ds-review-framework/SKILL.md` (209 lines, 8 sections)
  - Severity definitions (CRITICAL/MAJOR/MINOR)
  - Deduction table (8 analysis rows + 18 communication rows)
  - Floor rules with verdict bands
  - Audience persona definitions (exec/tech/DS/mixed)
  - Dimension boundary routing table (10 gray-zone scenarios)
  - Workflow context calibration (proactive/reactive/general)
  - Common anti-patterns (10 entries)
  - Confluence structure guide
- `plugin/commands/review.md` — full command definition with `--mode`, `--audience`, `--workflow` flags
- `dev/test-fixtures/synthetic/` — 8 synthetic test fixtures:
  - 01-no-tldr.md, 02-causal-without-method.md, 03-good-analysis-bad-comms.md,
    04-contradicts-data.md, 05-data-dump.md, 06-wrong-audience.md,
    07-partial-input-bullets.md, 08-unstructured-text.md
- `dev/test-results/2026-02-14-smoke-test.md` — full smoke test results (59/59 pass)
- `dev/decisions/ADR-002-auto-activate-in-subagents.md` — auto_activate not visible in subagent contexts
- `.gitignore` — excludes real test fixtures with sensitive data

#### Changed
- `plugin/agents/ds-review-lead.md` — Step 7 dispatch payload updated to instruct subagents to read SKILL.md from disk (auto_activate fallback)
- `plugin/.claude-plugin/plugin.json` — version bumped from 0.1.0 to 0.3.0

#### Validated
- Agent prompt instruction counts: analysis ~47, communication ~52, lead ~105 (all under 120 red flag)
- All cross-references between agents and SKILL.md verified
- Dimension separation confirmed (fixture 03: analysis 100, communication 37, no cross-contamination)
- Floor rules fire correctly (1 CRITICAL → Minor Fix, 2+ CRITICAL → Major Rework)
- All output formats verified (Full, Quick, Draft Feedback)
- Special paths work (partial input detection, unstructured document handling)

### v0.2.2 — Doc Consistency Review (2026-02-14)

**Status:** Complete. All active docs aligned with architecture.

#### Fixed
- Architecture design doc: 5 edits — tier thresholds (Section 2.1 matched to 3.2.4), Quick mode dashboard labels (3-level), lens rating shorthand (4-level), degradation level reference (Level 1-2), section cross-reference (15.3)
- PRD: 4 edits — narrowed v1 input formats to Confluence + markdown only, Quick mode latency (3 min), scoring description (equal-weight average), In Scope list alignment
- CLAUDE.md: removed auto-detection reference (explicit invocation only)
- Repo structure doc: 5 edits — removed auto-detection (2 locations), updated lens names (structure & TL;DR, conciseness & prioritization), updated input format in diagram, updated orchestrator description
- Plugin README: 4 edits — updated lens names, input formats, removed auto-detection section, updated orchestrator description
- review.md command: 2 edits — updated input format references, updated description frontmatter
- communication-reviewer.md: 1 edit — updated frontmatter description to current lens names
- v1-review-plugin spec: 3 edits — removed auto-detection, updated lens names, updated testing plan

#### Changed
- Implementation plan updated to Rev 3:
  - Added skill-creator hybrid approach guidance to Task 1 (single file, optimized frontmatter, token-efficient format, cross-reference stability)
  - Enhanced SKILL.md frontmatter description for better triggering
  - Added .gitignore step for `dev/test-fixtures/real/`
  - Added revision history entry

#### Process
- Parallel code review of architecture design + implementation plan (2 Opus subagents)
  - Architecture review: 1 CRITICAL, 6 MAJOR, 4 MINOR issues found
  - Implementation plan review: 1 CRITICAL, 3 MAJOR, 4 MINOR issues found
- All issues resolved across 9 files, 28 total edits
- Verification grep checks: all stale references cleared from active docs

### v0.2.1 — Implementation Planning (2026-02-14)

**Status:** Complete. Implementation plan Rev 3 ready. Agent prompts written. Execution pending in fresh session.

#### Added
- Implementation plan (`docs/plans/2026-02-14-implementation-plan.md`, Rev 1→3)
  - 8 tasks: auto_activate validation, SKILL.md, prompt validation, command, plugin.json, 8 fixtures, smoke test, wrap-up
  - Self-contained plan with full SKILL.md content embedded
  - IC8 Engineering Lead review conducted — 3 Critical + 5 Major issues identified and fixed
- Agent prompts written (complete production prose):
  - `plugin/agents/analysis-reviewer.md` (146 lines, 4 lenses, 17 checklist items, 8 rules)
  - `plugin/agents/communication-reviewer.md` (107 lines, 4 lenses, 18 checklist items, 10 rules)
  - `plugin/agents/ds-review-lead.md` (178 lines, 10-step pipeline, 4 output templates, 9 rules)

### v0.2 — Architecture Design (2026-02-14)

**Status:** Complete. Architecture approved.

#### Added
- Architecture design document (`docs/plans/2026-02-14-architecture-design.md`, Rev 4)
  - 13 consolidated architecture decisions
  - System data flow with 3-agent parallel dispatch
  - Token budget model calibrated to 200K context window
  - 3-level graceful degradation (Normal → Quick downgrade → Defer)
  - Tiered pre-processing for long documents (Tier 1/2/3)
  - Confluence integration via Atlassian MCP
  - Unstructured document handling with keyword/positional heuristics
  - Quick mode vs Full mode comparison
  - Agent prompt design (Pedro Sant'Anna's pattern)
  - Deduction-based scoring with floor rules
  - Output format templates (Full, Quick, Degraded, Draft)
  - Partial input handling with qualitative draft feedback
  - SKILL.md structure outline (8 sections)
  - Cost estimation per review
  - Testing & evaluation strategy (manual eval + auto-eval pipeline design)

- PRD and reference docs moved to `dev/specs/`
  - `PRD-DS-Analysis-Review-Agent.md`
  - `architecture-session-handoff.md`
  - `full-prd-session-record.md`

- Plugin conventions rule (`.claude/rules/plugin-conventions.md`)

#### Key Architecture Decisions (Rev 4)
1. Hybrid skill file approach (core in prompts, detail in SKILL.md)
2. Tiered pre-processing (< 2K pass-through, 2-5K section map, 5K+ extraction)
3. Quick mode: skip plan-first, default mixed audience, < 3 min target
4. Draft feedback: qualitative only, no numeric score
5. Equal weight scoring (50/50 analysis/communication)
6. Workflow context: default "general", flag-based (no inference)
7. Session handling: reactive failure handling, no predictive detection
8. Graceful degradation: 3 levels (cut from 5)
9. Confluence + local markdown only for v1

#### Decisions Made
- ADR 001: Two-dimension split (`dev/decisions/001-two-dimension-split.md`)

---

## [0.1.0] — 2026-02-14

### Added
- Initial repository structure and scaffolding
- Placeholder files for all agents, commands, and skills
- CLAUDE.md with project context and pickup instructions
- Dev tracking setup (backlog, sessions, decisions)
- Repository structure spec (`ds-analysis-review-agent-structure.md`)
