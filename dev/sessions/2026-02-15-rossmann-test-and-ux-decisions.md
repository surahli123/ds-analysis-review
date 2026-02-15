# Session: Rossmann Test #2 + UX Decisions

**Date:** 2026-02-15
**Duration:** ~1 session
**Focus:** Ran real-world test #2 (Rossmann), captured UX feedback

## What Happened

1. **Ran full 10-step pipeline on Rossmann Sales Prediction fixture**
   - Parameters: full / mixed / proactive / Tier 3 (7,452 words)
   - Built structured extraction for Tier 3 processing
   - Dispatched both subagents in parallel via Task tool
   - Analysis score: 39/100 (6 findings, 1 CRITICAL)
   - Communication score: 19/100 (9 findings, 1 CRITICAL)
   - Final: 29/100 — Major Rework (2 CRITICAL triggered floor rule)

2. **Captured initial calibration notes for Test #2**
   - Scoring too harsh (29/100, expected 45-60)
   - But marginally better than Meta (29 vs 18) — directional differentiation works
   - Confirmed deduction stacking as primary calibration problem

3. **Got owner UX feedback and made 3 decisions:**
   - Lens dashboard emoji indicators: APPROVED (SOUND->check, MINOR->warning, MAJOR/CRITICAL->x)
   - Top 3 rewrite format: DECIDED — blockquote with `> **pencil Suggested rewrite:**` prefix.
     Owner feedback: don't list 3 options with tradeoffs, pick highest-confidence one.
     The reader is already swamped with information.
   - Output compression: DECIDED — compress per-lens sections to 1-2 sentences by default.
     Owner feedback: don't add `--detail compact` flag, just make default output shorter.

## Key Decisions

- **No new CLI flags for output verbosity.** Compress by default, not via flags.
- **One rewrite format, not three options.** Blockquote with emoji label.
- **Owner's principle:** When the user is processing a lot of information, don't add
  decision overhead. Pick the best option and ship it.

## Modified Files

- `dev/test-results/2026-02-15-calibration-notes.md` — added Test #2 results + UX decisions
  (later restructured by owner with Test #3 Vanguard results and reframed around
  differentiation failure)

## Where We Left Off

Calibration notes have been rewritten by the owner to center on the differentiation
failure (Vanguard 16 < Meta 18, which is backwards). The proposed solution is now:

1. **P1: Positive Credit System + CRITICAL Gradation** (drives differentiation)
   - Positive credits: +5 to +8 per good practice, cap +25/dimension
   - CRITICAL split: CRITICAL-ABSENT (full -20, floor rule) vs CRITICAL-INCOMPLETE (-12, no floor rule)
2. **P2: Per-Dimension Deduction Cap** (-70 safety net)
3. **P3: UX Sprint** (emoji dashboard, blockquote rewrites, compressed output)

An ADR-003 is pending for the calibration approach once owner confirms.

## Pickup for Next Session

1. Read `dev/test-results/2026-02-15-calibration-notes.md` — fully rewritten, this is the source of truth
2. Read `dev/backlog.md` for current priorities
3. Owner needs to decide on the proposed solution before implementation begins
4. Next work: implement calibration fixes in SKILL.md + lead agent, then rerun all 3 tests
