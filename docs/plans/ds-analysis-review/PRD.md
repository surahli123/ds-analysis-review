# Product Requirements Document: DS Analysis Review Agent

**Version:** 1.0  
**Date:** February 14, 2026  
**Status:** Draft

---

## 1. Problem Statement

Ineffective DS analysis communication costs organizations real money. When an analysis is misinterpreted, engineering builds the wrong feature. When a measurement report buries the answer, a PM makes a decision without the data they asked for. When limitations aren't stated clearly, downstream teams build on assumptions the author didn't intend. When an exec loses confidence in the DS org's ability to communicate clearly, they stop asking for data-driven input — and the org loses its seat at the table.

The gap between producing a technically sound analysis and delivering one that drives correct decisions is wide. Failure modes are numerous: rigorous methodology buried under poor narrative structure, compelling storytelling built on weak statistical foundations, recommendations that don't match the audience's decision-making style, measurements that lack the context needed to act, and findings where the bounds of interpretation aren't clear enough for cross-functional consumers.

Peer review — the traditional safety net — is scarce. Colleagues are busy, feedback is inconsistent, and most reviewers naturally skew toward either the analytical or communication dimension, rarely both. The result is that DS analyses routinely lose impact — or worse, cause wrong decisions — not because the work is bad, but because no one caught the gaps before the work reached its audience.

There is no structured, always-available way for data scientists to review their work across both analytical rigor and communication effectiveness, tailored to who will read it.

## 2. Product Vision

**Short-term:** An agent that reviews DS analyses across two dimensions — analysis quality and communication effectiveness — using specialized subagents, delivering actionable feedback with concrete examples.

**Long-term:** The agent becomes part of the DS analysis workflow cycle. The end goal is to turn any DS analysis — from raw draft to polished deliverable — into an influential knowledge piece that connects the author's ideas with target audiences effectively and generates measurable impact.

## 3. Target Users

The primary users are **individual data scientists** and **DS team leads**. These users need structured, on-demand review of their analytical work before it reaches stakeholders.

The distribution channel varies by user technical comfort: users familiar with CLI tools use the Claude Code plugin for maximum flexibility and local customization; users who prefer a web interface access the same review capability via Agent Studio. The core experience and review quality is identical across both channels.

## 4. Review Modes

The agent supports two modes, serving different workflow moments.

### 4.1 Full Mode

**When:** The user wants a comprehensive review of their analysis. Output thoroughness is the priority. Latency target is within 5 minutes, even for long documents.

**Output includes:**

1. **Verdict line:** Overall score (0-100) mapped to a qualitative assessment — Good to Go (80-100) / Minor Fix (60-79) / Major Rework (0-59)
2. **Top 3 priority fixes** with concrete examples showing how to address each one
3. **Encouragement** — what the user did well (2-3 specific positive findings)
4. **Lens-by-lens evaluation** from both subagents, with per-lens qualitative severity ratings and detailed findings

### 4.2 Quick Mode

**When:** The user needs a fast polish or last-minute confirmation before submitting to their audience or going into a review meeting. They need to know if there are critical errors that would cause the analysis to fail. Latency is critical — target is within 3 minutes, even with long document input.

**Output includes:**

1. **Verdict line:** Same scoring as Full mode
2. **Top 3 priority fixes** with concrete examples
3. **Encouragement** — what the user did well

Quick mode skips the lens-by-lens deep dive.

## 5. Audience Personas

Users explicitly select the target audience persona when invoking the agent. If no persona is specified, the agent defaults to **mixed audience** mode.

### 5.1 Three Personas

**Business Executive** — Includes C-suite, VPs, and senior Product Managers. Inductive thinker. Starts with "so what?" Expects the conclusion and recommendation (or measurement answer) upfront, supported by evidence. Low tolerance for methodological detail. Needs clear business impact framing and decision context. Senior PMs in this persona care about user impact, tradeoffs, and roadmap implications — they need to translate findings into product decisions.

**Technical Lead** — Includes Engineering partners and stakeholders across all levels, from ICs to Eng VPs. This persona applies to all Eng partners because they want the number details — especially in domains sensitive to metrics (e.g., Search, Ads, Marketplace). Deductive thinker. Starts with evidence and reasoning. Expects the logical chain from data to conclusion. Wants to see methodology is sound and metrics are correctly defined before trusting recommendations. Also needs spec implications: what does this analysis mean for what we build, what edge cases should engineering watch for, and what are the data pipeline requirements?

**Peer Data Scientist** — Includes DS managers reviewing their team's work and peer DS reviewers. Expects full rigor. Wants to evaluate methodology, assumptions, and limitations. Comfortable with statistical detail. Looks for completeness and intellectual honesty. Checks whether the analysis would hold up under scrutiny from someone with equivalent or deeper domain expertise.

### 5.2 Mixed Audience (Default)

When the analysis targets multiple audiences (e.g., an exec summary plus technical appendix), the agent evaluates effectiveness across all personas simultaneously. Rather than producing three separate reviews, it synthesizes and surfaces the **top 3 fixes that maximize overall cross-audience effectiveness**.

### 5.3 Structural Principle

The deductive vs. inductive framing is the key structural principle behind persona adaptation. The communication reviewer evaluates whether the analysis's argument flow matches the audience's thinking style — not just whether the language is simplified or detailed, but whether the narrative *leads* with what that audience needs to hear first.

## 6. DS Workflow Context

DS analyses serve different purposes depending on workflow context. The agent should infer the workflow context from the analysis content, or the user can specify it explicitly. Critically: **TL;DR and Actionability are always required and always heavily scored, regardless of workflow context.** What changes is the *form* they take, never the *weight*.

The logic from an org leader's perspective: if a DS sends a measurement report to a PM and the PM has to read 8 pages to find the answer to the question they asked, that report has failed. If a DS sends experiment results and the PM has the number but doesn't know what it means for their decision, that report has also failed. The workflow context tells the agent what good TL;DR and good actionability *look like* in each case — it never tells the agent to relax the standard.

### 6.1 Proactive / Advisory Mode

DS identifies an opportunity, conducts analysis, and recommends action. DS is driving the narrative and owns the conclusion. Examples: "We should invest in reducing churn in segment X because..." or "Here's why we should change our pricing model."

- **TL;DR** leads with insight + recommendation + business impact.
- **Actionability** means specific recommendations with named owners, next steps, and timeline where appropriate.
- **Structure** follows a persuasive arc.

### 6.2 Reactive / Support Mode

Eng/PM has already decided what to build or is evaluating options. DS plays a neutral role providing measurements, experiment results, metric definitions, impact sizing, or validation. Examples: "Here are the A/B test results for the new checkout flow" or "Here's the impact sizing for the three options the PM is considering."

- **TL;DR** leads with the direct answer to the question asked. A measurement TL;DR looks like "The new checkout flow increased conversion by 3.2% (p < 0.01) with no significant impact on average order value." This is still heavily scored — a measurement report that buries the answer is just as broken as a proactive analysis that buries the recommendation.
- **Actionability** means the stakeholder can confidently act on the measurement. Are confidence intervals provided? Is practical significance contextualized alongside statistical significance? Are the bounds of what the data can and cannot support made explicit so the PM doesn't over-interpret a directional finding? DS may or may not include a recommendation — but the measurement itself must be actionable. A number without decision context is not actionable, it's trivia, regardless of who asked for it.
- **Structure** follows an answer-first arc: answer → evidence → limitations → implications.

### 6.3 Collaborative / Embedded Mode (v2)

A third workflow context exists and is growing: DS is embedded in a product squad and the analysis is part of an ongoing conversation, not a standalone deliverable. Examples: a DS contributing a section to a broader product review doc that PM is assembling, or sharing experiment results mid-sprint as part of a team decision thread. In this mode, the "analysis" may be a partial document — a 2-page section inside someone else's 15-page doc. The agent would need to handle partial documents and understand that DS didn't author the whole thing. This is deferred to v2 but the agent should be aware that collaboration contexts exist.

### 6.4 Why This Matters

In many DS orgs, 60-70% of work is reactive/support. The agent must recognize the difference so it evaluates the right *form* of TL;DR and actionability for each context. But the standard never drops — a reactive analysis with a weak TL;DR or unclear actionability is scored just as harshly as a proactive one.

## 7. Supported Input Formats

### v1

- Local files: Markdown (.md)
- Confluence documents

> **Note (updated post-architecture review):** v1 scope narrowed to Confluence + local markdown only. Google Docs, Word (.docx), and PowerPoint (.pptx) deferred to v1.5 — they require additional MCP integrations and format-specific parsing not yet designed.

### v2

- Video transcripts
- Loom video recordings
- Jupyter notebook exports (HTML or PDF)

## 8. Agent Architecture

### 8.1 Three-Agent System

| Agent | Role | Scope |
|---|---|---|
| **ds-review-lead** | Orchestrator | Receives input; collects user preferences (mode, audience persona); shows review plan for user confirmation; dispatches subagents; synthesizes outputs; produces final verdict and top-level assessment |
| **analysis-reviewer** | Subagent | Reviews the analysis dimension: methodology & assumptions, logic & traceability, completeness & source fidelity, metrics |
| **communication-reviewer** | Subagent | Reviews the communication dimension: structure & TL;DR, audience fit, conciseness & prioritization (incl. visualization and professional polish), actionability |

### 8.2 Workflow

```
User invokes via command or natural language with input + preferences
                    │
                    ▼
           ds-review-lead
           ├── 1. Parse input and invocation intent
           ├── 2. Collect/confirm preferences (mode, audience persona)
           ├── 3. Show review plan → user confirms
           ├── 4. Dispatch subagent(s) with input + preferences
           ├── 5. Receive subagent outputs (per-lens findings, severities, subagent scores)
           ├── 6. Synthesize into unified review
           ├── 7. Compute final score from subagent scores + cross-cutting assessment
           └── 8. Produce final output (format depends on mode)
                    │
                    ▼
           Review delivered to user
```

### 8.3 Plan-First Step

Before dispatching subagents, the lead agent shows the user a brief review plan. For example: "This is a 12-page churn analysis. I'll focus the analysis-reviewer on the regression methodology and metric selection, and the communication-reviewer on exec-level narrative flow. Proceed?"

This serves two purposes: it catches misalignment early (wrong audience, wrong focus area) before a full review cycle runs, and it sets user expectations on what depth to expect.

### 8.4 Invocation

Explicit only — no auto-detection. Two methods supported:

- **Slash command:** `/ds-review:review` with optional flags for mode and persona
- **Natural language:** e.g., "full review on churn_analysis.md targeting business exec" or "quick review on this doc"

The lead agent parses the intent and fills in missing preferences by asking the user.

### 8.5 Parallel Subagent Execution

The analysis-reviewer and communication-reviewer are independent — neither needs the other's output. They should run in parallel, not sequentially. This is critical for meeting latency targets, especially Quick mode's 1-2 minute goal.

Pedro's guide confirms 3 parallel agents as the sweet spot and notes that independent agents should always run simultaneously. Our two subagents fit this pattern cleanly. The lead agent dispatches both at once and synthesizes when both return.

Note: each parallel agent consumes its own context window. For very long documents, this interacts with the long document chunking risk (see Section 15) — the architecture session must address whether long documents should be pre-processed by the lead agent before dispatch.

### 8.6 Multi-Model Strategy (v1.5 Optimization)

Not all agents need the same model. Pedro's guide assigns different models per agent: Opus for tasks requiring deep reasoning across large documents, Sonnet for bounded checklist-style work. For our agents, the communication-reviewer (evaluating narrative flow, audience fit, argument structure) may benefit from a more capable model than the analysis-reviewer (more checklist-driven methodology checks). This is a v1.5 optimization — v1 uses the same model for all agents, then we evaluate whether differentiation improves quality or speed.

## 9. Agent Prompt Design Principles

Drawn from our analysis of Pedro Sant'Anna's multi-agent academic workflow ([website](https://psantanna.com/claude-code-my-workflow/), [repo](https://github.com/pedrohcgs/claude-code-my-workflow)), and the [full workflow guide](https://psantanna.com/claude-code-my-workflow/workflow-guide.html), adapted for the DS analysis review context.

### 9.1 Lens-Based Review Structure

Each subagent reviews through explicit, named lenses with checklists. This prevents the agent from doing a vague holistic pass and ensures every sub-dimension gets attention. The lens design for the analysis-reviewer draws directly from Pedro Sant'Anna's 5-lens domain-reviewer framework ([full guide](https://psantanna.com/claude-code-my-workflow/workflow-guide.html)), adapted for DS analysis.

**analysis-reviewer lenses:**

- **Methodology & Assumptions** — Is the analytical approach appropriate for the question? Are all assumptions explicitly stated and sufficient? Would weakening an assumption change the conclusion? Are limitations acknowledged? This lens includes an explicit assumption stress test: for every claim or finding, check whether the necessary conditions are stated and whether they hold.
- **Logic & Traceability** — Two-pass check. Forward pass: does each reasoning step follow from the previous? Are there logical gaps or unsupported leaps? Backward pass (adapted from Pedro's "Backward Logic Check"): starting from the conclusion/recommendation, can you trace it back to a finding, to a methodology, to data? This catches the common DS failure mode where the conclusion doesn't actually follow from what was presented, even if individual steps are locally valid.
- **Completeness & Source Fidelity** — Are key questions addressed? Are there obvious missing analyses? And critically: do referenced numbers, benchmarks, prior findings, and external sources actually say what the analysis claims they say? Source fidelity is not a deal-breaker but it catches misrepresented benchmarks and incorrect prior citations. (Adapted from Pedro's "Citation Fidelity" lens.)
- **Metrics** — Are the right metrics selected for the question? Are baselines and benchmarks provided for context? Are metrics defined clearly enough that the audience can interpret them?

**communication-reviewer lenses:**

The communication-reviewer evaluates how effectively a DS analysis communicates its findings to its intended audience. The core principle: a DS analysis doesn't just need to "present well" — it needs to be correctly interpreted and confidently acted on by everyone who reads it, including people who weren't in the room when it was created. The failure mode isn't just "the reader didn't get the point." It's: ambiguous limitations lead to downstream teams building on wrong assumptions, unclear scope leads to engineering shipping the wrong thing, buried findings lead to decisions made without the most important data.

The restructuring into 4 lenses is driven by research from Matt Abrahams' communication frameworks (Stanford GSB), DS-specific writing guidance (DoorDash Analytics team practices, Nolan & Stoudt's "Communicating with Data" from Oxford, PMC backward paper writing guide), and cross-functional handoff failure patterns. The research identified 6 core communication concerns, two of which — Conciseness & Prioritization and Professional Polish — were missing from the original lens design and consistently appeared as top failure modes.

**Why 4 lenses instead of 6:** The ~150 instruction constraint (Section 9.8) means the communication-reviewer prompt can't afford 6 separate lens checklists without risking silent instruction-dropping. By grouping related concerns, each lens stays focused enough for reliable evaluation while covering all 6 axes. The grouping logic: Structure & TL;DR are inseparable (TL;DR *is* the structural opening); Visualization naturally falls under Prioritization (choosing what to show is a prioritization decision); Professional Polish is a surface-quality check that pairs with Conciseness as an overall "information presentation" lens.

- **Structure & TL;DR** — The TL;DR (or executive summary) is the single most important element of a DS analysis and is heavily weighted in scoring. Following the DoorDash Analytics team's practice: every analysis should open with a clear, concise summary that frames key insights in the context of business impact. If writing a TL;DR is hard, the analysis isn't understood well enough to communicate — and that ambiguity will propagate to every downstream consumer. Beyond TL;DR: Is there a clear story arc? Does the structure follow "What → So What → Now What" or "Problem → Solution → Benefit"? Is the argument flow deductive or inductive, and does it match the audience's thinking style (see audience personas, Section 6.1)? Do section headings and subheadings serve as actionable signposts rather than generic labels (e.g., "Churn dropped 15% after intervention" not "Results")?
- **Audience Fit** — A single analysis often reaches multiple consumers: the exec who needs to decide whether to fund the next phase, the Eng partner who needs to know what to build, the DE who needs to understand data pipeline requirements, the peer DS who will extend the work. Is the level of detail calibrated for the primary audience? Is jargon appropriate? Does the framing match what the audience cares about? Is this the same deck being recycled for a different audience without adaptation (a common anti-pattern)? Critically: are limitations and scope boundaries stated clearly enough that a downstream consumer who only reads the TL;DR and recommendations won't build on an assumption the author didn't intend? This is the cross-functional handoff check — the lens evaluates whether the analysis will be correctly interpreted by people who weren't in the room when it was created.
- **Conciseness & Prioritization** — Does every section, chart, and table earn its place? Is the analysis the right length for its purpose, or does it bury signal in noise? Are there unnecessary sections that could be cut or moved to an appendix? Data presented without narrative overwhelms even data-savvy audiences. Does every visualization support the narrative (clear labels, right chart type, no chartjunk)? Do charts stand alone without requiring the reader to hunt for context? Are long-tail details grouped or removed rather than cluttering the main presentation? **Professional polish** is evaluated here as a sub-check: formatting consistency, spelling, grammar, visual hierarchy — because sloppy presentation signals sloppy analysis and adds cognitive burden on the reader.
- **Actionability** — Always heavily scored regardless of workflow context (see Section 6). The core question is: after reading this analysis, does the audience know what to do next? In **proactive mode**: Are recommendations specific, prioritized, and feasible? Are next steps named with owners and timeline? In **reactive mode**: Is the measurement clear enough that the stakeholder can confidently make their decision? Are confidence intervals provided? Is practical significance contextualized alongside statistical significance? Are the bounds of what the data can and cannot support made explicit? A number without decision context is not actionable — it's trivia, regardless of who asked for it. Following DoorDash's principle: "any insight which is not actionable is trivia." This applies equally to proactive recommendations and reactive measurements. In both modes: are findings scoped to prevent over-interpretation? Could a downstream consumer mistake a directional finding for a prescriptive conclusion?

### 9.1.1 Rubric Location: Skill File with Fallback

The full rubric for each lens (checklists, severity definitions, gray-zone routing table, common anti-patterns) lives in the shared skill file (`ds-review-framework/SKILL.md`). This is the preferred approach: rubrics are maintained once and shared across all agents.

**Fallback:** If the CLI agent or platform does not support auto-loaded skill files, the rubric detail should be embedded directly into each subagent's `.md` prompt file. The tradeoff is that rubrics are duplicated across agents, making maintenance harder. But it works on any system that can read agent markdown files. When porting to Agent Studio (v2.0), rubrics will need to be embedded in the prompt regardless since skill file auto-loading is not available there.

The architecture session should determine which approach to use based on the target CLI agent's capabilities.

### 9.2 Scoring at Both Levels: Subagent and Orchestrator

Each subagent produces its own per-lens severity ratings AND an overall subagent-level score (0-100). This ensures graceful degradation — if one subagent fails or errors out, the other subagent's score and findings still provide meaningful output to the user.

The lead agent then synthesizes subagent scores into the final unified score, applying cross-cutting adjustments for issues that span both dimensions. v1 uses an equal-weight average: `final_score = (analysis_score + communication_score) / 2`. This avoids undefined behavior from uncalibrated weighting. Weighted synthesis (e.g., adjusting weights by workflow context) is deferred to v1.5 once calibration data from real reviews is available.

This two-level scoring approach is inspired by Pedro Sant'Anna's system, where individual agents report qualitative severities and the orchestrator applies numeric quality gates on top. We extend this by having subagents also produce their own numeric scores for resilience.

### 9.3 Structured Output Templates

Each subagent produces findings in a consistent format per issue: location in document, severity, what's there now, what's wrong, and a suggested fix with example. The lead agent consumes these to produce the unified output.

### 9.4 Positive Findings Are Required

Both subagents must identify 2-3 things the analysis does well. This is not optional. It ensures the final output balances critique with encouragement, which is critical for user trust and adoption.

### 9.5 Review Fatigue Awareness

If the agent consistently gives the same feedback across multiple reviews (e.g., every review says "your TL;DR needs work"), the user stops reading. The lead agent should vary its framing when surfacing recurring feedback patterns — for example, escalating from specific fix examples to pointing the user toward a resource or practice for that skill. This is a v1.5 refinement, but the agent prompt should be designed with this awareness from v1: feedback should feel fresh and targeted to the specific analysis, not templated.

### 9.6 Self-Verification Rule

Before flagging an issue, the agent must verify its own critique is correct. False positives erode trust quickly. This is an explicit instruction in each subagent prompt: "Before reporting an error, confirm your assessment is accurate."

### 9.7 Report-Only Constraint

Subagents produce reports. They do not edit or rewrite the user's analysis. The lead agent synthesizes and prioritizes. The user decides what to act on. (This is consistent with the "advisory, not gate" design — see Section 12.)

### 9.8 Prompt Length Constraint (~150 Instructions)

Pedro's guide documents a critical finding: Claude reliably follows about 100-150 custom instructions per session. Beyond that, rules get silently ignored. This directly constrains our agent prompt design:

- Each subagent's `.md` prompt should be concise — role, lenses, checklist, output format, rules. Keep it under ~120 lines.
- Detailed rubric content (anti-patterns, examples, gray-zone routing) should live in the shared skill file, not in agent prompts, to avoid bloating individual prompts.
- If rubrics must be embedded in agent prompts (see fallback in 8.1.1), prioritize the highest-impact criteria and move secondary detail to comments or supplementary files.

This constraint should be validated during the architecture session with the target CLI agent.

## 10. Scoring System

### 10.1 Numeric Scale

0-100, with three threshold bands:

| Score | Verdict | Meaning |
|---|---|---|
| 80-100 | **Good to Go** | Analysis is ready to share. Minor polish optional. |
| 60-79 | **Minor Fix** | Solid foundation but specific issues should be addressed before sharing. |
| 0-59 | **Major Rework** | Significant gaps in analysis or communication that would undermine effectiveness. |

Inspired by the quality gate pattern from Pedro Sant'Anna's academic workflow (80/90/95 thresholds). Our thresholds are adapted to the DS analysis context where the primary question is "is this ready for its audience?"

### 10.2 How Scoring Works

Scoring uses a **deduction-based approach**: start at 100, subtract points based on issue severity. This makes scores auditable — users can see exactly why they scored an 82 vs a 95.

Adapted from Pedro Sant'Anna's quality scoring system ([full guide](https://psantanna.com/claude-code-my-workflow/workflow-guide.html)), where deductions are calibrated per issue type (e.g., equation overflow = -20, broken citation = -15, notation inconsistency = -3).

**Floor rules:** Deduction arithmetic alone can produce misleading scores. An analysis with 3 MINOR issues (-3, -3, -5 = -11) scores 89 (Good to Go), while an analysis with 1 CRITICAL issue (-15) scores 85 (also Good to Go). But an analysis with a flawed methodology or missing TL;DR should never be "Good to Go" regardless of arithmetic. Therefore:

- **Any CRITICAL finding automatically caps the verdict at Minor Fix (max 79)**, regardless of the computed score. An analysis with a CRITICAL issue is not ready to share.
- **Two or more CRITICAL findings automatically caps the verdict at Major Rework (max 59).** Multiple fundamental problems mean the analysis needs significant rework.
- The deduction arithmetic still runs normally — floor rules only override the verdict band, not the numeric score. This means the user sees both the computed score and the floor-adjusted verdict, maintaining auditability.

For our DS analysis context, the deduction table will be calibrated per lens. Example starting framework:

**Analysis-reviewer deductions:**

| Issue Type | Lens | Severity | Deduction | Example |
|---|---|---|---|---|
| Unstated critical assumption | Methodology & Assumptions | CRITICAL | -20 | Causal claim without stating comparability assumption |
| Flawed statistical methodology | Methodology & Assumptions | CRITICAL | -20 | Using correlation to claim causation without acknowledgment |
| Conclusion doesn't trace to evidence | Logic & Traceability | CRITICAL | -15 | Backward check fails — recommendation has no supporting finding |
| Unsupported logical leap | Logic & Traceability | MAJOR | -10 | Key finding not backed by presented evidence |
| Misrepresented source/benchmark | Completeness & Source Fidelity | MAJOR | -8 | "Industry average is X" but source says Y |
| Missing obvious analysis | Completeness & Source Fidelity | MAJOR | -8 | Obvious follow-up question unaddressed |
| Missing baseline/benchmark | Metrics | MAJOR | -10 | Metric reported without comparison point |
| Unacknowledged sampling or selection bias | Methodology & Assumptions | MAJOR | -10 | Analysis draws conclusions from non-representative sample, compares non-comparable groups, or uses convenience sample without acknowledging limitations. Not penalized if the analysis explicitly states the sampling limitation and explains why it's acceptable for the question at hand |

**Communication-reviewer deductions:**

Deductions are calibrated for real-world impact: a communication failure doesn't just mean the reader is confused — it means downstream consumers build on wrong assumptions, engineering ships the wrong thing, and teams waste cycles correcting course.

| Issue Type | Lens | Severity | Deduction | Example |
|---|---|---|---|---|
| Missing or ineffective TL;DR | Structure & TL;DR | CRITICAL | -15 | Proactive: opens with methodology instead of key insight + business impact. Reactive: opens with background instead of directly answering the question asked |
| No clear story arc or structure | Structure & TL;DR | CRITICAL | -12 | Reader can't follow the argument from setup to conclusion |
| Buried key finding | Structure & TL;DR | MAJOR | -10 | "So what" appears on page 8 instead of upfront |
| Generic/non-actionable section headings | Structure & TL;DR | MINOR | -3 | Heading says "Results" instead of "Churn dropped 15% after intervention" |
| Audience mismatch | Audience Fit | MAJOR | -10 | Technical jargon in exec-targeted deck, or insufficient rigor for peer DS |
| Recycled presentation for wrong audience | Audience Fit | MAJOR | -8 | Same deck sent to exec and peer DS without adaptation |
| Limitations/scope not clear for downstream consumers | Audience Fit | CRITICAL | -12 | ML Eng builds production model on exploratory finding because caveats were buried; DE pipelines data at wrong granularity because scope boundaries were vague |
| Analysis too long / buries signal in noise | Conciseness & Prioritization | MAJOR | -10 | 30-page deck that should be 10; appendix material in main body |
| Data presented without narrative context | Conciseness & Prioritization | MAJOR | -8 | Metrics table dumped without explanation of what matters |
| Unclear or misleading visualization | Conciseness & Prioritization | MAJOR | -8 | Chart missing axis labels, wrong chart type for the data, or chartjunk |
| Unnecessary chart or table | Conciseness & Prioritization | MINOR | -3 | Chart showing long-tail data that adds no insight |
| Sloppy formatting / inconsistent polish | Conciseness & Prioritization | MINOR | -5 | Mixed fonts, broken indentation, spelling errors — signals low diligence and adds cognitive burden on the reader |
| Vague recommendation or answer | Actionability | MAJOR | -8 | Proactive: "We should look into this further" without specifics. Reactive: "The results look positive" without quantification or decision context |
| Recommendation scoped to wrong decision level | Actionability | MAJOR | -8 | VP-level funding decision framed as team-level action item, or directional finding framed as prescriptive recommendation |
| No named owner or next step | Actionability | MINOR | -5 | Proactive: recommendation says what but not who or when. Reactive: measurement provided but no indication of what decision it informs |
| Missing "so what" for key finding | Actionability | MAJOR | -8 | Finding or measurement presented as trivia without business implication — applies in both modes |
| Measurement not interpretable by requester | Actionability | MAJOR | -8 | A/B test result without confidence interval or practical significance context |
| Over-interpretation boundary unclear | Actionability | MAJOR | -8 | Result presented without stating what the data can and cannot support, risking stakeholder over-interpretation |

Each subagent applies deductions within its lenses and produces a subagent-level score. The lead agent synthesizes into the final score. The deduction table is a v1 starting point; calibration based on real usage is a v1.5 deliverable.

## 11. Shared Skill: Review Framework

Located at `plugin/skills/ds-review-framework/SKILL.md`. This file is auto-loaded and shared across all three agents. It contains:

- **Rubrics** for each review lens with explicit evaluation criteria (4 analysis lenses: Methodology & Assumptions, Logic & Traceability, Completeness & Source Fidelity, Metrics; 4 communication lenses: Structure & TL;DR, Audience Fit, Conciseness & Prioritization, Actionability)
- **Dimension boundary definitions** — a routing table that specifies which subagent owns gray-zone feedback (e.g., "metric without context" → analysis-reviewer; "metric presented without baseline for audience" → communication-reviewer)
- **Audience persona definitions** with the deductive/inductive structural principle and expectations per persona
- **Common anti-patterns** with examples (e.g., "burying the lede," "missing or ineffective TL;DR," "conclusion without supporting evidence," "recommendation without owner or timeline," "unstated assumption," "recycled deck for wrong audience," "30-page analysis that should be 10," "data dump without narrative")
- **Severity level definitions** (what qualifies as CRITICAL vs MAJOR vs MINOR)
- **Deduction table** for scoring (see Section 10.2)

**Fallback:** If the target platform does not support auto-loaded skill files, all rubric content is embedded directly in each subagent's `.md` prompt file. See Section 9.1.1 for details.

## 12. Scope — v1 Boundaries

### In Scope

- Review of DS analyses provided as local files (md) and Confluence documents
- Two-dimension review (analysis + communication) via specialized subagents
- Two modes: Full (lens-by-lens + summary) and Quick (summary only)
- Audience-aware feedback with three personas + mixed audience default
- Plan-first step before review dispatch
- Unified output with verdict, top 3 fixes with examples, and encouragement
- Rubrics and scoring output
- Explicit invocation via `/ds-review:review` command or natural language prompts

### Out of Scope (v1)

- Auto-detection of DS analysis input
- Stage-awareness inference (user explicitly states stage and focus if relevant)
- Code review (Python/R/SQL in notebooks)
- Data quality assessment
- Automated rewrite or redraft of the analysis
- Review of non-DS content
- Iterative adversarial review loops
- Video or Loom input
- MEMORY.md feedback loop (deferred to v1.5 as optional feature)

## 13. Distribution Strategy

### Phase 1: Claude Code Plugin (v1.0)

Full three-agent architecture with command invocation. Installed as a plugin in Claude Code. Target: DS practitioners comfortable with CLI tools who want maximum customization.

### Phase 2: Agent Studio (v2.0)

Same core review capability deployed via Agent Studio for web-based access. Target: broader DS audience who prefer a web interface. Agent prompts are designed from v1 to be copy-pasteable as standalone agents for Agent Studio deployment.

Note: The subagent orchestration pattern may need adaptation for Agent Studio. Prompts should be modular enough to work as standalone agents if the orchestrator pattern doesn't translate directly.

## 14. Success Criteria

The following dimensions define success. Concrete targets (e.g., adoption percentages, revision round reductions, false positive rate thresholds) are being scoped and will be finalized in the v2 PRD or a dedicated growth-oriented document.

### 14.1 Efficiency (Directly Measurable)

- Fewer rounds of revision before analyses reach stakeholders
- Time saved per analysis review cycle compared to waiting for peer feedback

### 14.2 Discourse Quality (North Star — Qualitative)

- Stakeholder feedback shifts from clarification questions ("what does this mean?") to substantive discussion ("what if we tried a different approach?")
- The quality of downstream conversation improves because the analysis is clearer, more rigorous, and better targeted

This is the hardest to measure but the most meaningful signal. Validated through user interviews and surveys.

### 14.3 Adoption (Proxy for Value)

- Number of data scientists in the organization using the tool
- Frequency of use per user (one-time vs recurring) — retention is the real signal, not initial adoption
- Mode distribution (Full vs Quick) as signal for workflow integration

### 14.4 Trust (Negative Signal)

- False positive rate: if a significant share of flagged issues are not real problems, trust collapses and adoption dies. Monitoring false positive rate is a first-order concern from v1 launch.

## 15. Risks

| Risk | Impact | Mitigation |
|---|---|---|
| **Long document processing errors** | When reviewing lengthy analyses (10+ pages), the agent may produce erroneous or incomplete output. Early v0 testing with another AI coding agent confirmed this — long documents led to errors in review results, likely related to how the document is chunked and processed within context limits. This is the highest-priority technical risk. | Must be addressed in architecture design session. Investigate chunking strategies, context window management, and whether the lead agent should pre-process/summarize long inputs before dispatching to subagents. Test with real long-form DS analyses during development. |
| **Input format variance** | Analyses come in wildly different formats, lengths, and levels of completeness. Lead agent parsing must be robust. | Start with well-defined formats (md, docs, slides). Plan-first step lets user correct misinterpretation. |
| **Subagent dimension overlap** | Analysis and communication dimensions bleed into each other, causing duplicate or conflicting feedback. | Explicit routing table in shared skill file defines ownership for gray-zone issues. |
| **Over-criticism** | Every review feels negative, discouraging adoption. | Positive findings are mandatory in output. Encouragement section is a design constraint, not optional. |
| **False positives** | Agent flags issues that aren't real problems, eroding trust. | Self-verification rule in subagent prompts. Scoring calibration in v1.5. |
| **Latency targets** | Quick mode (1-2 min) and Full mode (5 min) may be hard to hit with long inputs and subagent dispatch. Quick mode particularly challenging: the plan-first step requires a user round trip, which may need to be skipped or streamlined for Quick mode. | Currently being tested. Quick mode latency design to be addressed in architecture session — may require skipping plan-first step or using a lightweight confirmation. |
| **Agent Studio portability** | Three-agent orchestration may not translate cleanly to Agent Studio. | Design prompts to be modular and standalone-capable from v1. |

## 16. Roadmap

| Version | Focus | Key Deliverables |
|---|---|---|
| **v1.0** | Full plugin launch | Three-agent system (lead + 2 subagents), shared skill with rubrics, both modes (Full/Quick), scoring output, `/ds-review:review` command, plugin manifest |
| **v1.5** | Calibration & local power features | Rubric and scoring recalibration based on real usage, multi-model strategy evaluation (different models per subagent for cost/quality optimization), optional MEMORY.md feedback loop for local users (Claude Code only — power user customization, not core feature) |
| **v2.0** | Web deployment | Agent Studio deployment for non-CLI users, standalone subagent versions adapted for web |
| **v2.5+** | Advanced capabilities | Adversarial critic-fixer loop for iterative review, lens-by-lens sequential pass within subagents, code reviewer subagent, domain knowledge subagent, video/Loom input support |

## 17. Open Questions

| # | Question | Impact | Status |
|---|---|---|---|
| 1 | Detailed rubric content per lens — what specific criteria score high vs low for each of the 8 lenses? | Directly affects review quality. | To be designed in architecture session. Starting framework drawn from Pedro Sant'Anna's domain-reviewer lens structure. |
| 2 | How should the agent handle partial or minimal input (e.g., just a few bullet points with no context)? | Determines minimum viable input. | Open |
| 3 | Quick mode architecture — how to reliably hit 1-2 min latency while maintaining quality? | Core UX promise of Quick mode. | To be discussed in dedicated architecture session. |
| 4 | Scoring deduction calibration — are the initial deduction values per issue type correctly weighted? | Determines verdict accuracy. | Starting deduction table in Section 10.2. Calibration in v1.5. |
| 5 | Long document chunking strategy — how should the agent handle documents that exceed context limits? | Highest-priority technical risk. Directly affects output quality. | Must be addressed in architecture session. |

## 18. Reference: Transferable Patterns

This project draws structural inspiration from Pedro Sant'Anna's multi-agent academic workflow for reviewing lecture slides ([website](https://psantanna.com/claude-code-my-workflow/), [repo](https://github.com/pedrohcgs/claude-code-my-workflow)).

### Adopted Patterns

- **Progressive agent specialization:** Narrow, focused agents outperform generalist reviewers. Each agent has a clearly scoped domain.
- **Lens-based review with checklists:** Each agent reviews through named lenses with explicit criteria, ensuring no dimension is skipped.
- **Qualitative severity per issue, deduction-based scoring at both subagent and orchestrator level:** Subagents assess severity per issue, apply deductions, and produce their own score. Orchestrator synthesizes into final score. Two-level scoring ensures graceful degradation if one subagent fails.
- **Structured output templates:** Consistent format per finding (location, severity, problem, suggested fix).
- **Mandatory positive findings:** Every review includes what the work does well.
- **Self-verification rule:** Agents check their own critique before reporting.
- **Shared knowledge files:** Rubrics, conventions, and standards in shared skill files rather than duplicated across agents.
- **Session logging and dev process:** Build continuity across development sessions.

### Deferred Patterns (Future Versions)

- **Adversarial critic-fixer loop:** Two agents checking each other's work across multiple rounds. Pedro's most powerful pattern, deferred to v2.5+ for our project.
- **Single-lens agents:** Pedro uses 10 agents with very narrow scope (one per review dimension). We start with 2 broader subagents, with the option to split further if review quality demands it.
- **Quality gates that block actions:** Pedro's scores gate commits and PRs. Our tool is advisory — it recommends but doesn't enforce.

---

*This is a living document. Update as decisions are made and scope evolves. Detailed agent prompt design and rubric content will be developed in the architecture session.*
