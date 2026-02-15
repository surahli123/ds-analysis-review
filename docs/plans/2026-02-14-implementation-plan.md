# DS Analysis Review Agent — Implementation Plan (Rev 3)

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Complete the Claude Code plugin by writing the shared skill file, validating agent prompts, updating the command and plugin config, and building synthetic test fixtures.

**Architecture:** 3-agent system — lead orchestrator dispatches two reviewer subagents in parallel. Shared SKILL.md provides rubrics and scoring tables. Agent prompts follow Pedro Sant'Anna's pattern: frontmatter, role, lenses with checklists, output format, rules.

**Tech Stack:** Claude Code plugin (markdown prompt files). No traditional code — all deliverables are `.md` files.

**What's Already Done:**
- Architecture design approved (Rev 4): `docs/plans/2026-02-14-architecture-design.md`
- Agent prompts written (complete production prose, not outlines):
  - `plugin/agents/analysis-reviewer.md` (146 lines)
  - `plugin/agents/communication-reviewer.md` (107 lines)
  - `plugin/agents/ds-review-lead.md` (178 lines)

**What This Plan Covers:**
- Task 0: Validate auto_activate runtime behavior
- Task 1: Write SKILL.md (shared knowledge file)
- Task 2: Validate agent prompts (instruction count, cross-reference, format)
- Task 3: Update review.md command definition
- Task 4: Update plugin.json
- Task 5: Create 8 synthetic test fixtures
- Task 6: Integration smoke test
- Task 7: Session wrap-up

---

## Task 0: Validate auto_activate Runtime Behavior

**Why this matters:** All three agent prompts reference `SKILL.md Section 2`, `Section 4`, `Section 5`, etc. The lead agent dispatches subagents via the Task tool. If `auto_activate: true` in SKILL.md does NOT make the skill available inside subagent contexts, the whole system breaks silently — subagents produce generic output because they can't find the deduction tables.

**Step 1: Create a minimal test skill**

Create a temporary test file:

```markdown
# File: dev/scratch/test-auto-activate.md

Test: Does a subagent dispatched via Task tool have access to auto_activate skills?

1. Create a simple Task tool dispatch with prompt: "List any skills you can see that mention 'ds-review-framework'. If you see a skill with that name, print its first 3 lines. If you don't see any skills, say NO_SKILLS_VISIBLE."
2. Run it.
3. Check result.
```

**Step 2: Run the test**

Dispatch a Task tool subagent with the test prompt above.

**Step 3: Record result**

Create `dev/decisions/ADR-002-auto-activate-in-subagents.md`:

```markdown
# ADR-002: auto_activate Skill Availability in Subagents

**Date:** 2026-02-14
**Status:** [Accepted/Rejected based on test]

## Context
Subagents are dispatched via Task tool. SKILL.md has `auto_activate: true`.

## Test Result
[Record whether subagent could see the skill]

## Decision
- If YES: Subagents reference SKILL.md by section number as designed.
- If NO: Lead agent embeds SKILL.md Sections 1-7 directly in the dispatch payload (fallback per architecture Decision 9). Update ds-review-lead.md Step 7 accordingly.
```

**Step 4: If fallback needed, update lead agent dispatch**

Only if the test shows subagents CANNOT see auto_activate skills: update `plugin/agents/ds-review-lead.md` Step 7 to embed SKILL.md content in the dispatch payload. The lead agent already has a fallback instruction for this case (line 105-106 in current file).

**Step 5: Commit**

```bash
git add dev/decisions/ADR-002-auto-activate-in-subagents.md
git commit -m "docs: ADR-002 — test auto_activate skill visibility in subagents"
```

---

## Task 1: Write SKILL.md (Shared Knowledge File)

The foundation everything else depends on. Both subagents and the lead agent reference this file for severity definitions, deduction tables, routing rules, and personas.

### Skill-Creator Approach (Hybrid)

This SKILL.md follows Claude Code's skill-creator best practices while using the architecture's content:

1. **Single file** — content is ~230 lines, well under the 500-line skill limit. No need to split into `references/` files.
2. **Optimized frontmatter** — `description` field should be comprehensive enough to serve as the triggering mechanism. Include both what the skill does and when agents should reference it.
3. **Token-efficient format** — use tables over prose where possible. Remove explanations Claude already knows (e.g., don't explain statistics generically). Every line should earn its token cost.
4. **No progressive disclosure split** — the file is small enough that splitting adds complexity without benefit. All 8 sections stay in one file.
5. **Cross-reference stability** — agents reference sections by number (e.g., "SKILL.md Section 2"). Do NOT renumber or restructure without updating all three agent prompts.

### Directory Setup

Create the skill directory first (it does not exist yet):
```bash
mkdir -p plugin/skills/ds-review-framework
```

**Files:**
- Create: `plugin/skills/ds-review-framework/SKILL.md`

**Step 1: Write SKILL.md with all 8 sections**

Write the skill file with this complete content:

```markdown
---
name: ds-review-framework
description: >
  Shared rubrics, scoring tables, audience personas, and routing rules for the DS analysis
  review agent system. Auto-loaded by ds-review-lead, analysis-reviewer, and communication-reviewer
  agents. Contains: (1) severity definitions with verdict impact, (2) deduction tables for 8 review
  lenses across analysis and communication dimensions, (3) floor rules that cap verdicts on CRITICAL
  findings, (4) audience persona definitions (exec, tech, DS, mixed) with thinking styles,
  (5) dimension boundary routing table for gray-zone finding ownership, (6) workflow context
  definitions (proactive, reactive, general), (7) common anti-patterns with catching lenses,
  (8) Confluence structural element guide for TL;DR detection.
auto_activate: true
---

# DS Review Framework

Shared knowledge for ds-review-lead, analysis-reviewer, and communication-reviewer agents.
This file is auto-loaded. Agents reference specific sections — do not restructure without
updating all three agent prompts.

---

## 1. Severity Definitions

| Severity | Definition | Impact on Verdict |
|---|---|---|
| CRITICAL | A fundamental flaw that would cause the analysis to fail its purpose — wrong conclusions reached, key audience misled, or decisions made on faulty foundation. The analysis is NOT ready to share. | Floor rule: caps verdict at Minor Fix (max 79). 2+ CRITICAL caps at Major Rework (max 59). |
| MAJOR | A significant gap that weakens the analysis but doesn't invalidate it. The reader can still reach roughly correct conclusions, but the analysis loses impact, credibility, or actionability. | Standard deduction applies (-8 to -12). Multiple MAJORs compound. |
| MINOR | A polish issue that doesn't affect the core message or analytical validity. Nice to fix, not blocking. | Small deduction (-3 to -5). |

---

## 2. Deduction Table

### Analysis Dimension

| Issue Type | Lens | Severity | Deduction | Example |
|---|---|---|---|---|
| Unstated critical assumption | Methodology & Assumptions | CRITICAL | -20 | Causal claim without stating comparability assumption |
| Flawed statistical methodology | Methodology & Assumptions | CRITICAL | -20 | Using correlation to claim causation without acknowledgment |
| Unacknowledged sampling/selection bias | Methodology & Assumptions | MAJOR | -10 | Conclusions from non-representative sample without acknowledging limitation |
| Conclusion doesn't trace to evidence | Logic & Traceability | CRITICAL | -15 | Backward check fails — recommendation has no supporting finding |
| Unsupported logical leap | Logic & Traceability | MAJOR | -10 | Key finding not backed by presented evidence |
| Misrepresented source/benchmark | Completeness & Source Fidelity | MAJOR | -8 | "Industry average is X" but source says Y |
| Missing obvious analysis | Completeness & Source Fidelity | MAJOR | -8 | Obvious follow-up question unaddressed |
| Missing baseline/benchmark | Metrics | MAJOR | -10 | Metric reported without comparison point |

### Communication Dimension

| Issue Type | Lens | Severity | Deduction | Example |
|---|---|---|---|---|
| Missing or ineffective TL;DR | Structure & TL;DR | CRITICAL | -15 | Proactive: opens with methodology instead of key insight + business impact. Reactive: opens with background instead of answering the question asked |
| No clear story arc or structure | Structure & TL;DR | CRITICAL | -12 | Reader can't follow the argument from setup to conclusion |
| Buried key finding | Structure & TL;DR | MAJOR | -10 | "So what" appears on page 8 instead of upfront |
| Generic/non-actionable headings | Structure & TL;DR | MINOR | -3 | Heading says "Results" instead of "Churn dropped 15% after intervention" |
| Audience mismatch | Audience Fit | MAJOR | -10 | Technical jargon in exec-targeted deck, or insufficient rigor for peer DS |
| Recycled presentation for wrong audience | Audience Fit | MAJOR | -8 | Same deck sent to exec and peer DS without adaptation |
| Limitations/scope unclear for downstream | Audience Fit | CRITICAL | -12 | ML Eng builds production model on exploratory finding because caveats were buried |
| Too long / buries signal in noise | Conciseness & Prioritization | MAJOR | -10 | 30-page deck that should be 10; appendix material in main body |
| Data without narrative context | Conciseness & Prioritization | MAJOR | -8 | Metrics table dumped without explanation of what matters |
| Unclear or misleading visualization | Conciseness & Prioritization | MAJOR | -8 | Chart missing axis labels, wrong chart type, or chartjunk |
| Unnecessary chart or table | Conciseness & Prioritization | MINOR | -3 | Chart showing long-tail data that adds no insight |
| Sloppy formatting / inconsistent polish | Conciseness & Prioritization | MINOR | -5 | Mixed fonts, broken indentation, spelling errors |
| Vague recommendation or answer | Actionability | MAJOR | -8 | "We should look into this further" without specifics |
| Recommendation scoped to wrong decision level | Actionability | MAJOR | -8 | VP-level funding decision framed as team-level action item |
| No named owner or next step | Actionability | MINOR | -5 | Recommendation says what but not who or when |
| Missing "so what" for key finding | Actionability | MAJOR | -8 | Finding presented as trivia without business implication |
| Measurement not interpretable by requester | Actionability | MAJOR | -8 | A/B test result without confidence interval or practical significance |
| Over-interpretation boundary unclear | Actionability | MAJOR | -8 | Result presented without stating what the data can and cannot support |

---

## 3. Floor Rules

1. **Any CRITICAL finding** in either dimension → verdict capped at **Minor Fix (max 79)**, regardless of computed score.
2. **Two or more CRITICAL findings** across either dimension → verdict capped at **Major Rework (max 59)**.
3. Floor rules override the **verdict band only**, not the numeric score. User sees both: `Score: 85 → Verdict: Minor Fix (floor rule applied)`.
4. Rationale: An analysis with a flawed methodology or missing TL;DR should never be "Good to Go" regardless of arithmetic.

**Verdict Bands:**

| Score | Verdict | Meaning |
|---|---|---|
| 80-100 | Good to Go | Analysis ready to share. Minor polish optional. |
| 60-79 | Minor Fix | Solid foundation, specific issues to address before sharing. |
| 0-59 | Major Rework | Significant gaps that would undermine effectiveness. |

---

## 4. Audience Persona Definitions

### Business Executive
- **Thinking style:** Inductive — "so what?" first, evidence second
- **Expects:** Impact framing, business context, clear recommendation, minimal technical detail
- **Red flags:** Jargon without explanation, methodology-heavy opening, no bottom line upfront
- **TL;DR pattern:** Key insight + business impact + recommended action

### Technical Lead
- **Thinking style:** Deductive — evidence first, conclusion follows
- **Expects:** Sufficient technical detail to evaluate feasibility, clear scope boundaries, implementation implications
- **Red flags:** Hand-wavy methodology, missing edge cases, no limitations section
- **TL;DR pattern:** Direct answer + key evidence + technical caveats

### Peer Data Scientist
- **Thinking style:** Full rigor — wants to evaluate methodology end-to-end
- **Expects:** Statistical detail, reproducibility info, assumption enumeration, alternative approaches considered
- **Red flags:** Causal claims without methodology justification, missing confidence intervals, no sensitivity analysis
- **TL;DR pattern:** Key finding + methodology summary + open questions

### Mixed Audience (Default)
- **Thinking style:** Cross-audience synthesis
- **Expects:** Layered structure — summary accessible to all, detail available for those who need it
- **Red flags:** Content that only works for one audience, missing progressive disclosure
- **TL;DR pattern:** Key insight framed for broadest audience + signposts to detail sections

### Structural Principle
- **Inductive framing** (exec): conclusion → supporting evidence → detail
- **Deductive framing** (tech/DS): evidence → analysis → conclusion
- **Mixed framing**: conclusion upfront, evidence structure below, detail in appendix

---

## 5. Dimension Boundary Routing Table

Defines which subagent owns gray-zone feedback. Prevents duplicate or conflicting findings.

| Gray-Zone Scenario | Owner | Lens | Rationale |
|---|---|---|---|
| Metric without context | analysis-reviewer | Metrics | Missing baseline is an analytical gap |
| Metric without audience-appropriate baseline | communication-reviewer | Audience Fit | The baseline exists but isn't calibrated for the reader |
| Missing limitation | communication-reviewer | Audience Fit | Impact is on downstream consumer interpretation |
| Flawed methodology | analysis-reviewer | Methodology & Assumptions | Even if it affects communication, the root cause is analytical |
| Conclusion without evidence | analysis-reviewer | Logic & Traceability | Unless it's purely a framing/placement issue |
| Buried key finding | communication-reviewer | Structure & TL;DR | The finding exists but isn't positioned for impact |
| Causal claim without comparability | analysis-reviewer | Methodology & Assumptions | Methodological flaw regardless of how it's presented |
| Number without decision context | communication-reviewer | Actionability | The analysis produced the number; comms failed to contextualize it |
| Statistical vs practical significance confusion | analysis-reviewer | Metrics | If the analysis itself conflates them |
| Statistical vs practical significance not communicated | communication-reviewer | Actionability | If the analysis distinguishes them but the presentation doesn't |

**Rule:** When in doubt, assign to the subagent whose lens checklist most directly addresses the issue. If a finding genuinely spans both dimensions, the analysis-reviewer owns the root cause and the communication-reviewer owns the presentation impact — but only if both are independently actionable. Never report the same issue from both subagents.

---

## 6. Workflow Context

### Proactive / Advisory Mode
- **Trigger:** User passes `--workflow proactive` (or analysis is self-initiated)
- **TL;DR expectation:** Leads with key insight + business impact + recommended action
- **Actionability expectation:** Specific, prioritized recommendations with named owners and next steps
- **Example:** "We should invest in feature X because our analysis shows Y. Here's the implementation plan."

### Reactive / Support Mode
- **Trigger:** User passes `--workflow reactive` (or analysis answers a specific question)
- **TL;DR expectation:** Leads with direct answer to the question asked
- **Actionability expectation:** Measurement clear enough for stakeholder to act, confidence intervals provided, practical significance contextualized
- **Example:** "The A/B test shows a 3.2% lift in conversion (95% CI: 1.1%-5.3%, p=0.003). This translates to ~$2.1M annual revenue."

### General Mode (v1 Default)
- **Trigger:** No `--workflow` flag specified (default)
- **TL;DR expectation:** Key insight or answer is clear and upfront
- **Actionability expectation:** Findings have "so what?" context, recommendations are specific enough to act on
- **Key principle:** TL;DR and Actionability are always heavily evaluated regardless of workflow mode

---

## 7. Common Anti-Patterns

| Anti-Pattern | Catching Lens | Description |
|---|---|---|
| Burying the lede | Structure & TL;DR | Key insight appears deep in the document instead of upfront. Reader must wade through context to find the point. |
| Recycled deck | Audience Fit | Same presentation reused for a different audience without adapting detail level, framing, or emphasis. |
| Data dump | Conciseness & Prioritization | Tables, charts, and numbers presented without narrative thread. Reader is left to draw their own conclusions. |
| Unsupported conclusion | Logic & Traceability | Final recommendation or conclusion that doesn't trace back to evidence presented in the analysis. |
| Causal claim without comparability | Methodology & Assumptions | "X caused Y" without establishing a valid comparison group or acknowledging confounders. |
| Number without decision context | Actionability | "Churn is 5.2%" — but is that good or bad? What was it before? What's the benchmark? What should the reader do? |
| Over-interpretation boundary unclear | Actionability | Results presented without stating what the data can and cannot support, risking stakeholder over-extrapolation. |
| Missing TL;DR | Structure & TL;DR | No executive summary, no upfront conclusion. Reader doesn't know the point until the end (if then). |
| Jargon soup | Audience Fit | Technical terminology used without regard for audience. NDCG, p-values, and confidence intervals thrown at a business exec. |
| Zombie appendix | Conciseness & Prioritization | Appendix material (exploratory charts, raw data tables, code snippets) in the main body instead of a clearly separated appendix. |

---

## 8. Confluence Structure Guide

### Where TL;DR Typically Lives
1. Info/Note/Success panel in top 20% of page
2. Bold/emphasized block before first H2 heading
3. Section explicitly named "TL;DR", "Executive Summary", "Summary", "Key Findings"
4. First paragraph (if it reads like a summary)
5. Content-signal scan (keyword detection as fallback)
6. ABSENT — becomes a communication finding

### Structural Elements

| Element | Signal | Agent Usage |
|---|---|---|
| Info/Note/Warning panel | Key callout — likely TL;DR, caveat, or key finding | TL;DR detection checks panels in top 20% |
| Expand macro | Appendix-like content, supplementary detail | Deprioritize in extraction — note existence, don't include |
| Status macro | Draft / In Review / Final | Drafts get lighter polish penalties |
| Table of Contents macro | Page has intentional structure | Positive signal for Structure lens |
| Page labels | May indicate target audience or workflow stage | Informational context, not used for inference |
| Inline comments | Existing reviewer feedback | Note that peer review is in progress |
| Child pages | Analysis spans multiple pages | Note but don't review (v1: single page only) |

### Label Conventions
- Labels like "exec-review", "ds-team", "eng-handoff" provide audience signals
- Labels like "draft", "in-review", "final" provide status signals
- These are informational — the agent does not infer audience from labels in v1
```

**Step 2: Verify SKILL.md structure**

Read back the file. Confirm:
- [ ] All 8 sections present and numbered (1-8)
- [ ] Frontmatter has `name: ds-review-framework` and `auto_activate: true`
- [ ] Analysis deduction table has 8 rows matching values above
- [ ] Communication deduction table has 18 rows matching values above
- [ ] Floor rules: 1 CRITICAL → max 79, 2+ CRITICAL → max 59
- [ ] All 4 audience personas defined with thinking style, expects, red flags, TL;DR pattern
- [ ] Routing table has all 10 gray-zone scenarios
- [ ] Workflow context has 3 modes (proactive, reactive, general)
- [ ] Anti-patterns table has 10 entries
- [ ] Confluence guide has TL;DR detection order (6 steps) and structural elements table

**Step 3: Commit**

```bash
git add plugin/skills/ds-review-framework/SKILL.md
git commit -m "feat: write ds-review-framework SKILL.md with all 8 sections

Shared knowledge file for DS analysis review agents.
Contains severity definitions, deduction tables, floor rules,
audience personas, dimension boundary routing table, workflow
context definitions, common anti-patterns, and Confluence
structure guide."
```

---

## Task 2: Validate Agent Prompts

Agent prompts are already written to production files. This task validates they meet quality standards before proceeding.

**Files to validate (read-only, fix only if issues found):**
- `plugin/agents/analysis-reviewer.md` (146 lines)
- `plugin/agents/communication-reviewer.md` (107 lines)
- `plugin/agents/ds-review-lead.md` (178 lines)

**Step 1: Instruction count validation**

For each agent prompt, count imperative instructions (sentences that tell the agent to DO something). The ~150 instruction attention limit means we need to stay well under that.

Count methodology — for each file, count lines that are:
- Imperative sentences ("Evaluate...", "Check...", "Report...", "Do not...")
- Checklist items (bullet points starting with "-" that describe what to check)
- Rules (numbered items in Rules section)

Expected counts:
- analysis-reviewer: ~40-50 instructions (6+4+3+4 checklist items + 8 rules + scattered imperatives)
- communication-reviewer: ~45-55 instructions (4+5+4+5 checklist items + 10 rules + scattered imperatives)
- ds-review-lead: ~60-80 instructions (10 process steps with sub-instructions + 9 rules)

**Red flag:** If any prompt exceeds ~120 imperative instructions, it needs trimming. Prioritize cutting from lower-severity rules and edge case handling.

**Step 2: Cross-reference SKILL.md sections**

Verify each agent prompt references the correct SKILL.md sections:
- [ ] analysis-reviewer references: Section 2 (deduction table), Section 5 (routing table)
- [ ] communication-reviewer references: Section 2, Section 4 (personas), Section 5, Section 6 (workflow)
- [ ] ds-review-lead references: Section 3 (floor rules)

**Step 3: Format verification**

For each agent prompt, confirm:
- [ ] Frontmatter has `name` and `description` fields
- [ ] Output format section defines the exact structure (PER-LENS RATINGS, FINDINGS, POSITIVE FINDINGS, DEDUCTION LOG, SUBAGENT SCORE)
- [ ] Output format is identical between analysis-reviewer and communication-reviewer
- [ ] Lead agent has all 4 output templates (Full, Quick, Draft Feedback, Degraded)
- [ ] Lead agent dispatch payload (Step 7) includes: mode, audience, workflow, tier, content
- [ ] Lead agent handles: both success, one failure, both failure, malformed output

**Step 4: Fix any issues found**

If instruction count is too high, trim lowest-priority rules. If cross-references are wrong, fix them. If format is inconsistent, align.

**Step 5: Commit (only if changes were made)**

```bash
git add plugin/agents/
git commit -m "fix: address prompt validation findings

[describe what was fixed]"
```

---

## Task 3: Update review.md Command Definition

**Files:**
- Replace: `plugin/commands/review.md`

**Step 1: Write the updated command definition**

Replace the placeholder with this content:

```markdown
---
name: review
description: Review a DS analysis across methodology, logic, narrative, and actionability
agent: ds-review-lead
---

# DS Analysis Review

Review your DS analysis across two dimensions:

1. **Analysis**: methodology, logic, completeness, metrics
2. **Communication**: structure, audience fit, conciseness, actionability

## Usage

/ds-review:review <source> [options]

### Source (required)
- Confluence URL: `https://confluence.company.com/wiki/spaces/DS/pages/12345`
- Local file: `path/to/analysis.md`

### Options
- `--mode full|quick` — Full review with lens-by-lens breakdown (default) or quick summary
- `--audience exec|tech|ds|mixed` — Target audience persona (default: mixed)
- `--workflow proactive|reactive` — Analysis workflow context (default: general)

### Examples

/ds-review:review churn_analysis.md
/ds-review:review churn_analysis.md --mode quick
/ds-review:review https://confluence.company.com/wiki/spaces/DS/pages/12345 --audience exec
/ds-review:review impact_measurement.md --mode full --audience tech --workflow reactive

### Natural Language

You can also invoke naturally:
- "Review my churn analysis targeting business executives"
- "Quick review on impact_measurement.md"
- "Full review on this Confluence page: [URL]"
```

**Step 2: Verify**

- [ ] Frontmatter `agent: ds-review-lead`
- [ ] All 3 flags documented with valid values and defaults
- [ ] Examples cover: basic, quick mode, audience flag, all flags combined
- [ ] Natural language examples present

**Step 3: Commit**

```bash
git add plugin/commands/review.md
git commit -m "feat: update review command with flag syntax

Adds --mode, --audience, --workflow flags with defaults.
Includes usage examples for both command and natural language invocation."
```

---

## Task 4: Update plugin.json

**Files:**
- Modify: `plugin/.claude-plugin/plugin.json`

**Step 1: Update version to 0.3.0**

```json
{
  "name": "ds-review",
  "version": "0.3.0",
  "description": "Review DS analyses across analysis and communication dimensions",
  "author": "surahli",
  "agents": [
    "agents/ds-review-lead.md",
    "agents/analysis-reviewer.md",
    "agents/communication-reviewer.md"
  ],
  "commands": [
    "commands/review.md"
  ],
  "skills": [
    "skills/ds-review-framework"
  ]
}
```

**Step 2: Commit**

```bash
git add plugin/.claude-plugin/plugin.json
git commit -m "chore: bump plugin version to 0.3.0 for implementation milestone"
```

---

## Task 5: Create 8 Synthetic Test Fixtures

Create 8 synthetic DS analyses — each designed to trigger specific findings. This validates that the right lenses fire on the right problems.

**Files:**
- Create: `dev/test-fixtures/synthetic/01-no-tldr.md`
- Create: `dev/test-fixtures/synthetic/02-causal-without-method.md`
- Create: `dev/test-fixtures/synthetic/03-good-analysis-bad-comms.md`
- Create: `dev/test-fixtures/synthetic/04-contradicts-data.md`
- Create: `dev/test-fixtures/synthetic/05-data-dump.md`
- Create: `dev/test-fixtures/synthetic/06-wrong-audience.md`
- Create: `dev/test-fixtures/synthetic/07-partial-input-bullets.md`
- Create: `dev/test-fixtures/synthetic/08-unstructured-text.md`
- Create: `dev/test-fixtures/synthetic/README.md`
- Create: `dev/test-fixtures/real/README.md`
- Create: `dev/test-fixtures/expected/README.md`

### Fixture Specifications

**01-no-tldr.md** (~1,500 words)
- Topic: User churn analysis
- Defect: Analytically sound but NO TL;DR, opens with methodology, key finding buried deep
- Expected triggers: Structure & TL;DR CRITICAL (missing TL;DR, -15), MAJOR (buried key finding, -10). Analysis lenses mostly SOUND.
- Expected score: ~75 (analysis ~95, communication ~55) → Minor Fix (floor rule from 1 CRITICAL)

**02-causal-without-method.md** (~1,200 words)
- Topic: "Feature X caused a 12% increase in retention"
- Defect: Pre/post comparison without control group, no comparability assumption stated, confounders ignored
- Expected triggers: Methodology CRITICAL (flawed methodology, -20), CRITICAL (unstated assumption, -20), Logic MAJOR (unsupported leap, -10)
- Expected score: Very low → Major Rework, floor rule (2+ CRITICAL → max 59)

**03-good-analysis-bad-comms.md** (~2,500 words)
- Topic: Feature adoption experiment with proper A/B test
- Defect: Solid methodology, proper controls, correct metrics — but jargon soup, no story arc, buried conclusions, data dump format
- Expected triggers: Analysis lenses mostly SOUND. Communication: multiple MAJOR/CRITICAL findings.
- Purpose: Tests dimension separation — analysis score high, communication score low

**04-contradicts-data.md** (~1,000 words)
- Topic: Engagement metric analysis
- Defect: Results section shows metric X declined 8%, but conclusion says "Metric X improved"
- Expected triggers: Logic & Traceability CRITICAL (conclusion contradicts data, -15)
- Purpose: Tests backward pass check

**05-data-dump.md** (~2,000 words)
- Topic: Quarterly metrics dashboard review
- Defect: Mostly tables and numbers, minimal narrative, no recommendations, no "so what"
- Expected triggers: Actionability MAJOR (missing "so what", -8 each), Conciseness MAJOR (data without narrative, -8), Structure MAJOR (no story arc)

**06-wrong-audience.md** (~1,500 words)
- Topic: A/B test results with full statistical detail
- Defect: Written for peer DS audience (p-values, confidence intervals, detailed methodology) but intended for executive audience
- Expected triggers: Audience Fit MAJOR (audience mismatch, -10), MAJOR (recycled presentation, -8)
- Purpose: Tests audience-awareness. Run with `--audience exec`.

**07-partial-input-bullets.md** (~300 words)
- Topic: Early-stage search ranking improvement analysis
- Defect: Just bullet points and an outline — not a finished analysis
- Expected triggers: Lead agent Step 4 partial input check (< 500 words, outline format). Should prompt user to choose Full review vs Draft feedback.
- Purpose: Tests the partial input detection path

**08-unstructured-text.md** (~3,000 words)
- Topic: Customer segmentation analysis written as a narrative essay
- Defect: No headings, no sections, no markdown structure — just prose paragraphs. Content is analytically decent.
- Expected triggers: Lead agent Step 6 unstructured document handling (content-signal scanning instead of heading-based extraction). Communication reviewer should flag structure issues.
- Purpose: Tests the unstructured document heuristic path

### Steps

**Step 1-8: Write each fixture**

Each fixture should read like a realistic DS analysis (not obviously synthetic). Include plausible data, realistic company context, and natural language patterns. The flaws should be the kind a real data scientist would make, not cartoonish errors.

**Step 9: Write README files**

`dev/test-fixtures/synthetic/README.md`:
```markdown
# Synthetic Test Fixtures

8 intentionally flawed DS analyses for validating the review agent system.

| # | File | Primary Defect | Key Lenses Tested | Expected Verdict |
|---|---|---|---|---|
| 01 | no-tldr.md | Missing TL;DR, buried findings | Structure & TL;DR | Minor Fix |
| 02 | causal-without-method.md | Causal claim without methodology | Methodology & Assumptions | Major Rework |
| 03 | good-analysis-bad-comms.md | Sound analysis, terrible communication | Dimension separation | Split score |
| 04 | contradicts-data.md | Conclusion contradicts results | Logic & Traceability | Major Rework |
| 05 | data-dump.md | Numbers without narrative | Actionability, Conciseness | Minor Fix |
| 06 | wrong-audience.md | Technical content for exec audience | Audience Fit | Minor Fix |
| 07 | partial-input-bullets.md | Early draft / outline only | Partial input detection | Draft Feedback |
| 08 | unstructured-text.md | No headings or structure | Unstructured doc handling | Minor Fix |

## How to Use
Run: `/ds-review:review dev/test-fixtures/synthetic/01-no-tldr.md --mode full`
Compare output against expected triggers listed above.

## Adding Fixtures
Name pattern: `NN-descriptive-name.md`. Update this README with the fixture spec.
```

`dev/test-fixtures/real/README.md`:
```markdown
# Real Test Fixtures

Place real DS analyses here for testing. These files are NOT committed to git
(add *.md to .gitignore in this directory if needed) as they may contain
sensitive company data.

## How to Add
1. Export from Confluence or copy your .md file here
2. Note: target audience, workflow context, and any known issues
3. Run the review and compare against your expectations
```

`dev/test-fixtures/expected/README.md`:
```markdown
# Expected Findings

After running smoke tests, document expected findings here for regression testing.
Each file should be named to match its fixture: `01-no-tldr-expected.md`, etc.

This directory will be populated after the first round of smoke testing (Task 6).
```

**Step 10: Update .gitignore**

Add `dev/test-fixtures/real/*.md` to `.gitignore` (keep the README but exclude actual analysis files that may contain sensitive data):
```
# Real test fixtures may contain sensitive company data
dev/test-fixtures/real/*.md
!dev/test-fixtures/real/README.md
```

**Step 11: Commit**

```bash
git add dev/test-fixtures/ .gitignore
git commit -m "feat: create 8 synthetic test fixtures for agent validation

Each fixture targets specific lenses and agent behaviors:
- 01-06: Core finding detection (TL;DR, methodology, dimension separation,
  logic contradiction, data dump, audience mismatch)
- 07: Partial input detection path (< 500 words outline)
- 08: Unstructured document handling (no headings, prose only)
Includes READMEs for synthetic, real, and expected fixture directories."
```

---

## Task 6: Integration Smoke Test

Depends on: Tasks 0-5

Run each synthetic fixture through the review system. This is the first time the full pipeline runs end-to-end.

**Step 1: Test fixture 01 (no TL;DR) — Full mode**

Run: `/ds-review:review dev/test-fixtures/synthetic/01-no-tldr.md --mode full --audience exec`

Expected:
- [ ] Structure & TL;DR lens rates CRITICAL
- [ ] Missing TL;DR appears in top 3 fixes
- [ ] Analysis lenses rate mostly SOUND
- [ ] Score in Minor Fix range (floor rule from 1 CRITICAL finding)
- [ ] Output follows Full mode template (title, score, metadata, lens dashboard, top 3, positives, per-dimension breakdown)

**Step 2: Test fixture 02 (causal without method)**

Run: `/ds-review:review dev/test-fixtures/synthetic/02-causal-without-method.md --mode full`

Expected:
- [ ] Methodology & Assumptions rates CRITICAL
- [ ] 2+ CRITICAL findings → Major Rework verdict
- [ ] Flawed methodology in top 3 fixes
- [ ] Floor rule applied and explained: "Score: X → Verdict: Major Rework (floor rule: 2+ CRITICAL)"

**Step 3: Test fixture 03 (good analysis, bad comms) — Dimension separation**

Run: `/ds-review:review dev/test-fixtures/synthetic/03-good-analysis-bad-comms.md --mode full --audience exec`

Expected:
- [ ] Analysis dimension score high (80+)
- [ ] Communication dimension score low (< 65)
- [ ] No analysis findings about communication issues (routing table respected)
- [ ] No communication findings about methodology (routing table respected)

**Step 4: Test Quick mode**

Run: `/ds-review:review dev/test-fixtures/synthetic/01-no-tldr.md --mode quick`

Expected:
- [ ] Output follows Quick mode template (no lens-by-lens breakdown)
- [ ] Top 3 fixes still present
- [ ] Dimension dashboard (2-row table: Pass/Issues/Critical) instead of lens dashboard
- [ ] Footer suggests running --mode full for details

**Step 5: Test fixture 07 (partial input) — Partial input detection**

Run: `/ds-review:review dev/test-fixtures/synthetic/07-partial-input-bullets.md --mode full`

Expected:
- [ ] Lead agent detects < 500 words / outline format
- [ ] User is prompted to choose: Full review vs Draft feedback
- [ ] If Draft feedback chosen: output has no numeric score, severity capped at MAJOR

**Step 6: Test fixture 08 (unstructured text)**

Run: `/ds-review:review dev/test-fixtures/synthetic/08-unstructured-text.md --mode full`

Expected:
- [ ] Lead agent flags: "This document has minimal explicit structure"
- [ ] Content-signal scanning extracts meaningful content despite no headings
- [ ] Communication reviewer identifies structure issues

**Step 7: Test remaining fixtures (04, 05, 06)**

Run each and verify primary expected findings trigger:
- 04: Logic & Traceability CRITICAL (contradiction)
- 05: Actionability MAJOR (missing "so what"), Conciseness MAJOR (data dump)
- 06: Audience Fit MAJOR (with `--audience exec`)

**Step 8: Document test results**

Create `dev/test-results/2026-02-14-smoke-test.md` with:
- Pass/fail per fixture per expected finding
- Unexpected findings (false positives or false negatives)
- Score calibration notes (do scores feel right? Too harsh? Too lenient?)
- Prompt adjustment notes (what to tweak in agent prompts)
- Cross-cutting issues observed

**Step 9: Fix any prompt issues found during testing**

If findings don't fire as expected, adjust the relevant agent prompt. Common fixes:
- Tighten checklist wording if findings are too vague
- Adjust deduction values if scores feel miscalibrated
- Fix routing table references if dimension boundaries are crossed

**Step 10: Commit test results and any prompt adjustments**

```bash
git add dev/test-results/ plugin/agents/ plugin/skills/
git commit -m "test: smoke test results for 8 synthetic fixtures

[summarize pass/fail and any prompt adjustments]"
```

---

## Task 7: Session Wrap-Up

**Step 1: Update backlog**

Update `dev/backlog.md`:
- Mark v0.3 Implementation tasks as done
- Update current sprint to v0.4 (Calibration & Testing)
- Add any new items discovered during testing

**Step 2: Update CHANGELOG.md**

Add v0.3 entry:
```markdown
## [0.3.0] - 2026-02-14

### Added
- ds-review-framework SKILL.md with 8 sections (severity, deductions, floor rules, personas, routing, workflow, anti-patterns, Confluence guide)
- analysis-reviewer agent prompt (146 lines, 4 lenses, 17 checklist items)
- communication-reviewer agent prompt (107 lines, 4 lenses, 18 checklist items)
- ds-review-lead orchestrator prompt (178 lines, 10-step pipeline)
- review command with --mode, --audience, --workflow flags
- 8 synthetic test fixtures for agent validation
- ADR-002: auto_activate behavior in subagents
```

**Step 3: Create session log**

Create `dev/sessions/2026-02-14-implementation.md` with:
- What was built
- Decisions made (ADR-002 result, any prompt adjustments)
- Test results summary
- Pickup instructions for next session

**Step 4: Commit**

```bash
git add dev/backlog.md CHANGELOG.md dev/sessions/
git commit -m "docs: update backlog, changelog, and session log for v0.3"
```

---

## Implementation Order Summary

```
Task 0: Validate auto_activate ─────────────────────────────────────────┐
                                                                         │
Task 1: Write SKILL.md ─────────────────────────────────────────────────┤
                                                                         │
Task 2: Validate agent prompts (instruction count, cross-ref, format) ──┤
                                                                         │
Task 3: Update review.md ───────────────────────────────────────────────┤
                                                                         │
Task 4: Update plugin.json ─────────────────────────────────────────────┤
                                                                         │
Task 5: Create 8 synthetic fixtures ────────────────────────────────────┘
                                                                         │
                                                                         v
Task 6: Integration smoke test ──→ Task 7: Wrap-up
```

**Parallelization opportunities:**
- Tasks 0-5 are all independent of each other and can run in parallel
- Exception: If Task 0 reveals auto_activate doesn't work, Task 1 and Task 2 may need updates
- Task 6 depends on ALL of Tasks 0-5 (needs everything in place for end-to-end test)
- Task 7 depends on Task 6 (needs test results to document)

**Estimated work:**
- Tasks 0, 2, 3, 4: Lightweight validation/config (~5 min each)
- Task 1: Medium (SKILL.md content is provided above, just needs copying) (~10 min)
- Task 5: Heavy (8 realistic synthetic documents) (~30 min)
- Task 6: Heavy (8 end-to-end test runs + analysis) (~45 min)
- Task 7: Lightweight bookkeeping (~10 min)

---

## Revision History

| Rev | Date | Changes |
|---|---|---|
| 1 | 2026-02-14 | Initial plan (9 tasks, writing-plans skill) |
| 2 | 2026-02-14 | IC8 review fixes: agent prompts pre-written (C1), plan self-contained with full SKILL.md content (C2), dispatch payload in lead agent prompt (C3), Task 0 for auto_activate validation (M5), interleaved testing restructured to Task 6 (M1), 2 new fixtures: partial-input + unstructured (M3), instruction count validation in Task 2 (M4), score edge cases addressed in lead agent prompt (M2) |
| 3 | 2026-02-14 | Post code-review fixes: SKILL.md Task 1 updated with skill-creator hybrid approach (optimized frontmatter, token-efficient format, directory setup). Added .gitignore step for real test fixtures. Architecture doc inconsistencies fixed separately (tier thresholds, dashboard labels, degradation levels, section references). Note: architecture Section 2.1 data flow diagram tier thresholds (originally ~1.2K/4K) have been corrected to match Section 3.2.4 revised values (2K/5K). |
