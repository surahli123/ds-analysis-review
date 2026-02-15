# Development Backlog

Last updated: 2026-02-15

## Current Sprint: v0.4 — Calibration & Testing (IN PROGRESS)

### Done
- [x] First real-world test: meta-llm-bug-reports (scored 18/100 — too harsh)
- [x] Second real-world test: rossmann-sales-prediction (scored 29/100 — too harsh)
- [x] Third real-world test: vanguard-ab-test (scored 16/100 — LOWER than Meta's 18 despite being a better analysis)
- [x] Calibration notes written: `dev/test-results/2026-02-15-calibration-notes.md`
- [x] Principal AI Engineer assessment: `dev/test-results/2026-02-15-principal-ai-engineer-assessment.md`
  - Root cause: deduction stacking (correlated findings treated as independent)
  - Proposed: diminishing returns formula + tighten CRITICAL definitions
- [x] PM Lead review completed: `dev/reviews/2026-02-15-pm-lead-calibration-review.md`
  - 6 critiques: broken scoring math, CRITICAL over-assignment, no strength credits,
    cross-cutting double-counting, teardown-style output, no genre awareness
  - 4 proposed fixes: diminishing returns, reclassify structural CRITICALs, strength
    credits, output restructure
- [x] DS Lead assessment completed: `dev/test-results/2026-02-15-ds-lead-assessment.md`
  - Finding-by-finding audit of Meta review: 5 agree, 4 partial, 6 disagree (genre-inappropriate)
  - Identified 3 bugs in review output (severity/deduction mismatches, cross-cutting double-count)
  - Independent scores: Meta 52/100, Vanguard 45/100 (both Minor Fix)
  - Key new insights: finding quality matters (not just scoring math), cap total findings at 8,
    strength credits need +25 cap (not +15), genre affects which findings are generated
- [x] UX decisions approved: compressed per-lens detail, emoji lens dashboard, highlighted rewrites
- [x] 12 real-world fixtures collected in `dev/test-fixtures/real/`
- [x] **All 3 calibration perspectives collected** — ready for owner synthesis and decision
- [x] **A3 fix plan synthesized:** `dev/test-results/2026-02-15-calibration-fix-plan.md`
  - Merged all 3 reviews into consensus/disagreement analysis
  - 6 fixes in 2 phases (3 P0 + 3 P1) with exact file paths and changes
  - 5 owner decisions identified (strength cap, Meta target, finding cap, reclassification, DR)
- [x] **Calibration loop workflow planned:** `docs/plans/2026-02-15-calibration-loop-workflow.md`
  - 8-task repeatable loop: fix → test → notes → compare → 3 role reviews → synthesize → decide
  - File naming convention with round numbers (rN)
  - Acceptance criteria defined (score ranges, differentiation gaps, CRITICAL counts)
  - Estimated 1-3 rounds to converge

### To Do — Owner Decision Needed (5 Decisions Before Round 1)
Owner must synthesize the 3 assessments (Engineer, PM, DS Lead) and decide on the
calibration fix approach before implementation begins. Key decision points:

1. **Strength credit cap:** PM says +15/dimension. DS Lead says +25/dimension. Which?
2. **CRITICAL gradation:** Calibration notes propose CRITICAL-ABSENT vs CRITICAL-INCOMPLETE
   split. DS Lead says just reclassify structural CRITICALs to MAJOR (simpler). Which approach?
3. **Finding volume cap:** DS Lead proposes 8 total findings max. Others don't propose this.
   Add it or not?
4. **Diminishing returns:** Engineer and PM both propose it. DS Lead says it's a safety net,
   not the primary fix. Still include it?
5. **Meta target score:** Calibration notes say 20-35. DS Lead says 45-55. What's the target?

### To Do — Implementation (After Owner Decision)
- [ ] **P0: Implement Strength Credits** (all 3 assessments agree this is needed)
  - Credit table and cap TBD by owner decision
  - Update lead agent synthesis (Step 9) + subagent output format (add STRENGTH LOG)
- [ ] **P0: Reclassify CRITICALs** (all 3 assessments agree)
  - Missing TL;DR: CRITICAL (-15) → MAJOR (-10)
  - No story arc: CRITICAL (-12) → MAJOR (-8)
  - Limitations absent: CRITICAL (-12) → MAJOR (-10)
  - DS Lead adds: "No recommendations with owners" should never be CRITICAL
  - Update SKILL.md Sections 1 and 2
- [ ] **P0: Diminishing Returns** (Engineer + PM propose; DS Lead agrees as safety net)
  - 0-30: 100%, 31-50: 50%, 51+: 25%
  - Update lead agent synthesis (Step 9)
- [ ] **P1: Reduce Finding Volume** (DS Lead proposal — pending owner decision)
  - Cap at 8 total findings max (top 5 by severity + up to 3 MINOR)
  - Subsume derivative findings under parent root cause
- [ ] **P1: Output Restructure** (PM + DS Lead agree)
  - Move "What You Did Well" before findings
  - Compress per-lens detail to 1-2 sentences
  - Add emoji indicators to lens dashboard
  - Add one-sentence review summary after score (DS Lead addition)
  - Use blockquote format for suggested rewrites
  - Update lead agent Step 10 output templates
- [ ] **P1: Fix 3 Bugs Found in Audit** (DS Lead finding)
  - Meta C7 Actionability: severity says CRITICAL but deduction is -8 (MAJOR). Fix consistency.
  - Meta C5 Audience Fit: severity says MAJOR but deduction is -12 (CRITICAL). Fix consistency.
  - Meta A3 + C5: "no limitations" flagged by both subagents. Routing table says comms owns it.
- [ ] Create ADR-003 for calibration approach once owner confirms
- [ ] Rerun 3 real-world tests to validate calibration
  - Target ranges TBD by owner (see decision point #5)
  - Acceptance test: clear tier separation with 15-25 point gap
- [ ] Rerun synthetic fixtures to verify floor rules / dimension separation still work
- [ ] Run 2-3 more untested real-world fixtures to confirm consistency
- [ ] Verify cross-run consistency (same doc 3x, scores within +/- 10)

### Deferred
- Genre/format auto-detection — DS Lead recommends for v0.5 (affects finding generation, not just scoring)
- CRITICAL-ABSENT vs CRITICAL-INCOMPLETE gradation — try reclassification first, add gradation if needed
- Per-dimension deduction cap at -70 — diminishing returns makes this redundant
- Cross-cutting deduction deduplication — diminishing returns partially addresses
- Cluster-based scoring — Engineer recommends for v1.5

## Previous Sprint: v0.3 — Implementation (COMPLETE)

### Done
- [x] Architecture approved by product owner
- [x] Implementation plan v1 created (writing-plans skill)
- [x] IC8 review of implementation plan — 3 Critical + 5 Major issues identified
- [x] Agent prompts written (complete production prose, not outlines):
  - `plugin/agents/analysis-reviewer.md` (146 lines, 4 lenses, 17 checklist items)
  - `plugin/agents/communication-reviewer.md` (108 lines, 4 lenses, 18 checklist items)
  - `plugin/agents/ds-review-lead.md` (178 lines, 10-step pipeline)
- [x] Implementation plan revised to Rev 2 — all IC8 issues addressed
- [x] Parallel Opus code review of architecture + implementation plan (v0.2.2)
  - Architecture: 1C / 6M / 4m → all resolved
  - Impl plan: 1C / 3M / 4m → all resolved
- [x] Doc consistency fixes — 9 files, 28 edits, verified clean
- [x] Implementation plan revised to Rev 3 — skill-creator approach + .gitignore + enhanced frontmatter

### Done (Implementation)
- [x] Task 0: Validate auto_activate runtime behavior in subagent contexts
  - Finding: auto_activate does NOT work in subagents. ADR-002 created.
  - Fix: lead agent instructs subagents to read SKILL.md from disk.
- [x] Task 1: Write SKILL.md (209 lines, 8 sections, all rubrics and deduction tables)
- [x] Task 2: Validate agent prompts (instruction counts: analysis ~47, comm ~52, lead ~105)
- [x] Task 3: Update review.md command definition with --mode, --audience, --workflow flags
- [x] Task 4: Update plugin.json to v0.3.0
- [x] Task 5: Create 8 synthetic test fixtures (all in dev/test-fixtures/synthetic/)
- [x] Task 6: Integration smoke test — 59/59 checks pass across 9 test runs
- [x] Task 7: Session wrap-up (this update)

## Previous Sprint: v0.2 — Architecture Design (COMPLETE)

- [x] PRD and reference docs imported to `dev/specs/`
- [x] Architecture design session completed
- [x] 13 architecture decisions resolved
- [x] Architecture design document written (Rev 4)
- [x] PM lead review completed
- [x] CHANGELOG.md created

## Backlog: v1.0 and Beyond

### v0.5: Genre Awareness (if needed after v0.4 calibration)
- [ ] Evaluate whether recalibrated scores are reasonable across genres without format detection
- [ ] If still off: design `--format` parameter or auto-detection (ADR needed)

### v1.0: Ship
- [ ] All agents passing manual eval rubric
- [ ] Output format verified across all modes
- [ ] Plugin.json updated to v1.0
- [ ] Handoff documentation complete

### v1.5: Auto-Eval & Iteration
- [ ] Build LLM-as-Judge auto-eval pipeline (Section 15.3)
- [ ] Add review quality feedback loop ("Was this helpful?")
- [ ] Calibrate score weighting by workflow context
- [ ] Add workflow context inference (if usage data justifies)
- [ ] Add predictive session freshness detection (if reviews frequently fail)
- [ ] Support non-Confluence/non-md input formats

### v2.0: Extended Features
- [ ] Multi-page Confluence analysis review
- [ ] Iterative review with diff mode (compare to previous review)
- [ ] Agent Studio deployment for non-technical users

## Notes
- Architecture doc: `docs/plans/2026-02-14-architecture-design.md` (Rev 4)
- Implementation plan: `docs/plans/2026-02-14-implementation-plan.md` (Rev 3)
- Implementation order: Tasks 0-5 parallel → Task 6 smoke test → Task 7 wrap-up
