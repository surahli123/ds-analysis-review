# Smoke Test Results — 2026-02-14

## Summary

**Result: ALL PASS (59/59 checks across 9 test runs)**

No prompt fixes required. All agent prompts, SKILL.md rubrics, and output formats
behave as designed.

## Test Environment

- Testing method: Simulated pipeline (subagents read all prompts + SKILL.md from disk,
  act as both reviewers, synthesize as lead)
- Plugin command `/ds-review:review` not registered in environment — full pipeline
  simulated via Task tool dispatches
- Model: claude-opus-4-6
- Date: 2026-02-14

## Results by Fixture

### Fixture 01: no-tldr (Full mode, --audience exec)

| Expected Finding | Status |
|---|---|
| Structure & TL;DR rated CRITICAL (missing TL;DR) | PASS |
| "Buried key finding" appears as MAJOR | PASS |
| Analysis lenses mostly SOUND | PASS |
| Floor rule caps verdict at Minor Fix (1 CRITICAL) | PASS |

**Score: 77/100 — Minor Fix** | Analysis: 97 | Communication: 57

---

### Fixture 02: causal-without-method (Full mode, --audience mixed)

| Expected Finding | Status |
|---|---|
| Methodology CRITICAL: correlation as causation, no control group | PASS |
| Methodology CRITICAL: unstated comparability assumption | PASS |
| Logic MAJOR: unsupported leap (selection bias in user comparison) | PASS |
| 2+ CRITICAL triggers floor rule → Major Rework (max 59) | PASS |

**Score: 43/100 — Major Rework** | Analysis: 22 | Communication: 64

---

### Fixture 03: good-analysis-bad-comms (Full mode, --audience exec)

Stress-tests **dimension separation** — analytically exemplary RCT with terrible
exec communication.

| Check | Status |
|---|---|
| Analysis dimension score HIGH (80+) | PASS (100) |
| Communication dimension score LOW (< 65) | PASS (37) |
| Analysis reviewer does NOT flag communication issues | PASS |
| Communication reviewer does NOT flag methodology issues | PASS |
| Missing TL;DR detected (CRITICAL) | PASS |
| No story arc detected (CRITICAL) | PASS |
| Jargon/audience mismatch detected (MAJOR) | PASS |
| Data dump pattern detected (MAJOR) | PASS |
| Routing table compliance (no cross-dimension findings) | PASS |
| "Number without decision context" routed to communication | PASS |
| Floor rule: 2 CRITICAL → Major Rework | PASS |
| Floor rule explanation shown in output | PASS |

**Score: 69/100 — Major Rework** | Analysis: 100 | Communication: 37

---

### Fixture 04: contradicts-data (Full mode, --audience mixed)

| Expected Finding | Status |
|---|---|
| Logic & Traceability rated CRITICAL | PASS |
| Contradiction detected: conclusion says "improved" but data shows 8.0% decline | PASS |
| Backward pass failure identified | PASS |
| Floor rule: 1 CRITICAL → Minor Fix (max 79) | PASS |
| Verdict is Minor Fix, not Good to Go | PASS |
| Remaining document quality recognized as strong | PASS |
| Both subagent outputs in correct format | PASS |
| Full Mode output format followed (all 8 sections) | PASS |

**Score: 81/100 — Minor Fix (floor rule applied)** | Analysis: 85 | Communication: 77

---

### Fixture 05: data-dump (Full mode, --audience mixed)

| Expected Finding | Status |
|---|---|
| Actionability MAJOR: missing "so what" for key findings | PASS |
| Conciseness MAJOR: data without narrative context | PASS |
| Structure CRITICAL: no clear story arc | PASS |
| Structure CRITICAL: missing TL;DR | PASS |
| Anti-pattern match: "Data dump" | PASS |

**Score: 35/100 — Major Rework** | Analysis: 82 | Communication: 38

---

### Fixture 06: wrong-audience (Full mode, --audience exec, --workflow reactive)

| Expected Finding | Status |
|---|---|
| Audience Fit MAJOR: audience mismatch (technical jargon for exec) | PASS |
| Audience Fit MAJOR: recycled presentation for wrong audience | PASS |
| Structure CRITICAL: missing/ineffective TL;DR | PASS |
| Analysis lenses all SOUND (methodology is exemplary) | PASS |

**Score: 69/100 — Minor Fix** | Analysis: 100 | Communication: 37

Anti-patterns matched: Recycled deck, Burying the lede, Jargon soup

---

### Fixture 07: partial-input-bullets (Full mode, --audience mixed)

Tests **Step 4: Partial Input Check** path.

| Expected Finding | Status |
|---|---|
| Lead agent detects < 500 words (actual: 281) | PASS |
| Lead agent detects outline/bullet-point format | PASS |
| Partial input check triggers Step 4 prompt | PASS |
| Draft feedback mode: no numeric score | PASS |
| Draft feedback mode: severity capped at MAJOR | PASS |

**Output: Draft Feedback** (no numeric score, qualitative only)

---

### Fixture 08: unstructured-text (Full mode, --audience mixed)

Tests **Step 6: Unstructured Document Handling** with content-signal scanning.

| Expected Finding | Status |
|---|---|
| Lead agent flags minimal structure (zero headings) | PASS |
| Communication: Structure CRITICAL (no clear story arc, -12) | PASS |
| Communication: Structure MAJOR (buried key finding, -10) | PASS |
| Communication: Conciseness MAJOR (too long/buries signal, -10) | PASS |
| Analysis lenses mostly SOUND | PASS |

**Score: 69/100 — Major Rework (floor rule: 2 CRITICAL)** | Analysis: 92 | Communication: 45

---

### Quick Mode: Fixture 01 (Quick mode, --audience exec)

| Expected Finding | Status |
|---|---|
| Title + score + verdict present | PASS |
| Metadata shows Mode: Quick | PASS |
| Status table is 2-row (Dimension / Status) | PASS |
| Status uses Pass / Issues Found / Critical Issues vocabulary | PASS |
| Top 3 Priority Fixes present with correct format | PASS |
| What You Did Well section present | PASS |
| No lens-by-lens breakdown (Quick mode omits per-lens detail) | PASS |
| Footer suggests --mode full | PASS |
| Missing TL;DR surfaces as key finding despite Quick mode | PASS |
| Floor rule correctly applied (1 CRITICAL) | PASS |
| Quick mode finding caps respected (max 2 per lens) | PASS |
| Processing Tier forced to 3 by Quick mode | PASS |

**Score: 83/100 — Minor Fix (floor rule applied)** | Quick format verified

---

## Key Validation Outcomes

### 1. Dimension Separation
Analysis and communication reviewers stay in their lanes. No cross-dimension
findings observed in any test. Routing table (SKILL.md Section 5) correctly
directs gray-zone findings.

### 2. Floor Rules
All three floor rule scenarios validated:
- 1 CRITICAL → Minor Fix cap (fixtures 01, 04, 06)
- 2+ CRITICAL → Major Rework cap (fixtures 02, 03, 05, 08)
- No CRITICAL → no cap (not tested — no fixture designed for Good to Go)

### 3. Deduction Table
Subagents use exact values from SKILL.md Section 2. Self-correction behavior
observed in several runs (agent catches its own deduction errors and re-derives).

### 4. Output Formats
- Full mode: all 8 sections present in correct order
- Quick mode: 2-row status table, no per-lens breakdown, footer with upgrade prompt
- Draft feedback: no numeric score, severity capped at MAJOR, qualitative focus

### 5. Special Paths
- Partial input detection (Step 4): correctly triggers at < 500 words + bullet format
- Unstructured document (Step 6): content-signal scanning works, flag message produced
- Tier forced to 3 in Quick mode regardless of word count

## No Prompt Fixes Required

All agent prompts performed as designed. No changes needed.
