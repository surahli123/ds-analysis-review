# Architecture Design: DS Analysis Review Agent v1.0

**Date:** 2026-02-14
**Status:** Draft — Pending Review (Rev 4)
**Prerequisite:** PRD v1.0 (`dev/specs/PRD-DS-Analysis-Review-Agent.md`)

---

## 1. Consolidated Architecture Decisions

| # | Decision | Resolution | Rationale |
|---|---|---|---|
| 1 | Skill file vs embedded rubrics | **Hybrid** — core criteria in agent prompts, detailed rubrics/examples/anti-patterns in SKILL.md | Agent prompts stay self-sufficient for Agent Studio portability. SKILL.md avoids duplication and centralizes calibration content. |
| 2 | Long document handling | **Tiered pre-processing** — tier determined by computed word count (Confluence API does not expose reading time directly) | Tiering avoids over-processing short docs while protecting quality on long ones. |
| 3 | Quick mode architecture | **Skip plan-first entirely** — default to mixed audience, both subagents run with tighter finding caps. Latency target < 3 min. | Plan-first step requires a user round trip. Mixed audience default avoids unreliable inference. |
| 4 | Partial/minimal input | **Ask user to clarify intent** — "Early draft or full review?" Both subagents run in either path. **Draft feedback is qualitative only (no numeric score).** | Partial input is ambiguous. Numeric draft scores invite invalid comparison with full review scores. |
| 5 | v1 input formats | **Confluence + local markdown only** | Most DS analyses live on Confluence. Local md covers testing and edge cases. Other formats deferred to v1.5. |
| 6 | Invocation | **Explicit only** — `/ds-review:review` command or natural language | No auto-detection in v1. Reduces false activation risk. |
| 7 | Agent prompt templates | **Adapted from Pedro Sant'Anna's domain-reviewer structure** | Proven pattern: frontmatter, role, lenses with checklists, output format, rules. |
| 8 | Audience in Quick mode | **Default to mixed audience** — no inference. User specifies via `--audience` flag if needed. | Domain-specific jargon is unreliable for inference (e.g., Search execs know NDCG). Saves tokens, avoids wrong assumptions. |
| 9 | Skill file fallback | **Embed critical rubric content in agent prompts** if CLI does not support auto-loaded skills | Ensures agents work standalone on any system. See Section 7.4. |
| 10 | Score weighting | **Equal weight (50/50)** — analysis and communication dimensions weighted equally | Avoids undefined behavior from uncalibrated weighting. Adjust in v1.5 with calibration data. |
| 11 | Workflow context | **Default to "general"** — user specifies via `--workflow` flag if needed. No inference in v1. | Content-based inference is unreliable. Flag-based is explicit and costs nothing. |
| 12 | Session context handling | **Reactive, not predictive** — handle failure when it happens instead of estimating remaining context | Predictive heuristics (turn counting, self-test) are unreliable and cause over-degradation. |
| 13 | Graceful degradation | **3 levels** — Normal, Auto-downgrade to Quick, Defer to new session | Levels 2-3 from the original 5-level design (single-subagent, lead-only) produced too-degraded output to be useful. |

---

## 2. System Data Flow

### 2.1 End-to-End Flow

```
User invokes:
  /ds-review:review [confluence-url | file.md] [--mode full|quick] [--audience exec|tech|ds|mixed] [--workflow proactive|reactive]

  or natural language:
  "full review on churn_analysis.md targeting business exec"
  "quick review on https://confluence.company.com/wiki/spaces/DS/pages/12345"

                    |
                    v
           +-------------------------------------------+
           |           ds-review-lead                   |
           |                                            |
           |  1. FETCH INPUT                            |
           |     Confluence? -> confluence_get_page      |
           |       -> page content + labels + status     |
           |     Local md? -> Read tool                  |
           |                                            |
           |  2. COMPUTE WORD COUNT                      |
           |     Count words from fetched content        |
           |     Derive reading time (word_count / 230)  |
           |     Use word count for tier selection        |
           |                                            |
           |  3. DETERMINE PROCESSING TIER              |
           |     (from computed word count)              |
           |     < 2,000 words   -> Tier 1 (pass-through)|
           |     2,000-5,000     -> Tier 2 (section map) |
           |     5,000+ words    -> Tier 3 (extraction)  |
           |                                            |
           |  4. PARTIAL INPUT CHECK                    |
           |     If minimal content detected ->          |
           |     ask user: "Early draft or full          |
           |     review?" -> adjust depth accordingly    |
           |                                            |
           |  5. MODE BRANCH                            |
           |     Full -> show plan, wait for confirm     |
           |     Quick -> skip plan, auto-proceed        |
           |                                            |
           |  6. PRE-PROCESS (Tier 2-3 only)            |
           |     Tier 2: build section map               |
           |     Tier 3: build structured extraction     |
           |     Both: detect TL;DR location,            |
           |           identify Confluence macros        |
           |                                            |
           |  7. DISPATCH (parallel via Task tool)       |
           |     -> analysis-reviewer                    |
           |     -> communication-reviewer               |
           |     Payload per subagent:                   |
           |       - content (full or extraction)        |
           |       - section map (if Tier 2)             |
           |       - preferences (mode, audience,        |
           |         workflow: flag or default general)   |
           +------+----------------+--------------------+
                  |                |
         +--------+                +--------+
         v                                  v
+------------------+          +------------------------+
| analysis-reviewer|          | communication-reviewer  |
|                  |          |                          |
| 4 lenses:        |          | 4 lenses:               |
| - Methodology &  |          | - Structure & TL;DR     |
|   Assumptions    |          | - Audience Fit           |
| - Logic &        |          | - Conciseness &          |
|   Traceability   |          |   Prioritization         |
| - Completeness & |          | - Actionability          |
|   Source Fidelity|          |                          |
| - Metrics        |          |                          |
|                  |          |                          |
| Output:          |          | Output:                  |
| - Findings list  |          | - Findings list          |
|   (max 3/lens)   |          |   (max 3/lens)           |
| - Per-lens rating|          | - Per-lens rating        |
| - Positives(2-3) |          | - Positives (2-3)        |
| - Subagent score |          | - Subagent score         |
|   (0-100)        |          |   (0-100)                |
+--------+---------+          +------------+-------------+
         |                                 |
         +-----------+-----------+---------+
                     v
           +-------------------------------+
           |    ds-review-lead (cont.)     |
           |                               |
           |  8. HANDLE SUBAGENT RESULTS   |
           |     - Both returned? Merge.   |
           |     - One failed? Use other + |
           |       warn user.              |
           |     - Both failed? Defer to   |
           |       new session (Level 2).  |
           |                               |
           |  9. SYNTHESIZE                |
           |     - Merge findings          |
           |     - Apply floor rules       |
           |     - Compute final score     |
           |     - Select top 3 priority   |
           |       fixes (BOTH modes)      |
           |     - Select encouragement    |
           |     - Build lens dashboard    |
           |                               |
           | 10. PRODUCE OUTPUT            |
           |     Full: verdict + dashboard |
           |       + top 3 + encouragement |
           |       + lens-by-lens breakdown|
           |     Quick: verdict + dashboard|
           |       + top 3 + encouragement |
           +-------------------------------+
```

### 2.2 Subagent Dispatch Payload

Each subagent receives a single prompt containing:

```
REVIEW REQUEST
Mode: [full | quick]
Audience: [exec | tech | ds | mixed]
Workflow Context: [proactive | reactive | general (default)]
Processing Tier: [1 | 2 | 3]

CONTENT:
[Tier 1: full document text]
[Tier 2: full document text + SECTION MAP below]
[Tier 3: STRUCTURED EXTRACTION below]

SECTION MAP (Tier 2 only):
[heading hierarchy with line references, macro notes]

STRUCTURED EXTRACTION (Tier 3 only):
[see Section 4.4 for template]
```

---

## 3. Token Management & Graceful Degradation

This is the most critical reliability section. LLM-powered agents fail in predictable ways — the architecture must handle every one.

### 3.1 Failure Modes

| # | Failure Mode | Root Cause | Impact |
|---|---|---|---|
| F1 | **Instruction degradation** | Long doc + detailed checklists exceed the ~150 instruction attention limit | Subagent silently drops rules, produces incomplete or off-target review |
| F2 | **Output truncation** | Subagent hits max output tokens before finishing report | Partial report — lower-severity findings lost |
| F3 | **Lost signal in noise** | Full document sent when only 20% is relevant per subagent | Subagent spends attention on irrelevant content, misses key issues |
| F4 | **Subagent failure** | One or both subagents error out, timeout, or return malformed output | No review or partial review delivered |
| F5 | **Lead agent context exhaustion** | User invokes review mid-session with accumulated context | Lead agent can't read doc + pre-process + synthesize in remaining context |
| F6 | **Confluence fetch failure** | MCP server unavailable, auth expired, rate limit hit | No content to review |

### 3.2 Token Budget Model (200K Context Window)

**Baseline assumption:** Each context window is 200K tokens max. Subagents get fresh 200K windows via the Task tool. The lead agent shares the user's session window (200K minus accumulated conversation).

**Estimation method:** `estimated_tokens = word_count * 1.3` (English text averages ~1.3 tokens per word including whitespace and punctuation).

#### 3.2.1 Why the Lead Agent Is the Bottleneck

Subagents are virtually unconstrained — each gets a fresh 200K window. Even a 50-page analysis (~15K words = ~20K tokens) uses only 10% of a subagent's input budget. The subagent constraint is **instruction-following quality** (the ~150 instruction limit), not raw tokens.

The lead agent is the bottleneck because the document appears **multiple times** in its context:

| Occurrence | What | Tokens (for a 5K-word doc) |
|---|---|---|
| 1. Read/fetch result | Document content returned by Read tool or Confluence MCP | ~6,500 |
| 2. Dispatch to subagent 1 | Task tool call prompt includes content/extraction | ~6,500 (Tier 1) or ~3,000 (Tier 3) |
| 3. Dispatch to subagent 2 | Task tool call prompt includes content/extraction | ~6,500 (Tier 1) or ~3,000 (Tier 3) |
| 4. Subagent 1 result | Findings, ratings, score returned | ~3,000 |
| 5. Subagent 2 result | Findings, ratings, score returned | ~3,000 |
| 6. Synthesis + output | Lead agent's final report | ~3,000 |

**Document multiplier in lead agent context:**
- Tier 1 (pass-through): document appears **3x** = 3 * doc_tokens
- Tier 3 (extraction): document appears **1x** + extraction appears **2x** = doc_tokens + 2 * ~3K

This is the core architectural insight: **Tier 3 extraction is not just about subagent quality — it's a context efficiency strategy for the lead agent.**

#### 3.2.2 Full Budget Breakdown (200K Window)

**Fixed overhead per review (regardless of doc size):**

| Component | Tokens | Notes |
|---|---|---|
| System prompt + CLAUDE.md + tool definitions | ~8,000-10,000 | Platform overhead, always present |
| Lead agent orchestration logic | ~3,000 | Pre-processing, tier decisions, plan step |
| Subagent dispatch overhead x2 (prompts, flags) | ~2,000 | Mode, audience, context instructions |
| Subagent results x2 (findings + scores) | ~6,000-8,000 | ~3-4K per subagent (12 findings + ratings + score) |
| Lead synthesis + final output | ~3,000-4,000 | Verdict, dashboard, top 3, encouragement |
| Safety margin | ~5,000 | Buffer for tool metadata, unexpected overhead |
| **Total fixed overhead** | **~27,000-37,000** | Call it **~35K** for safe planning |

**Variable cost (document-dependent):**

| Tier | Doc read | Dispatch payload x2 | Total variable | Total review cost |
|---|---|---|---|---|
| Tier 1 (1K words, ~1.3K tokens) | 1,300 | 2 * 1,300 = 2,600 | **3,900** | ~39K |
| Tier 1 (3K words, ~3.9K tokens) | 3,900 | 2 * 3,900 = 7,800 | **11,700** | ~47K |
| Tier 2 (5K words, ~6.5K tokens) | 6,500 | 2 * (6,500 + 1,000 map) = 15,000 | **21,500** | ~57K |
| Tier 3 (10K words, ~13K tokens) | 13,000 | 2 * 3,000 extraction = 6,000 | **19,000** | ~54K |
| Tier 3 (20K words, ~26K tokens) | 26,000 | 2 * 3,000 extraction = 6,000 | **32,000** | ~67K |
| Tier 3 (40K words, ~52K tokens) | 52,000 | 2 * 3,000 extraction = 6,000 | **58,000** | ~93K |

#### 3.2.3 Available Context by Session State

This table is **informational** — the system does not try to detect session state (see Section 3.6). It helps us understand the design constraints.

| Session State | Estimated Accumulated | Remaining Context | Max Review Cost | What fits |
|---|---|---|---|---|
| **Fresh session** | ~0K | ~200K | ~200K | Any document, any tier, any mode |
| **Light session** (~10 turns) | ~30-50K | ~150-170K | ~150K | Any realistic DS analysis |
| **Mid session** (~30 turns) | ~80-120K | ~80-120K | ~80K | Tier 3 up to ~35K-word doc, or Tier 1 up to ~15K-word doc |
| **Deep session** (~50+ turns) | ~140-170K | ~30-60K | ~30K | Tier 3 only. Reviews may fail — user should retry in fresh session. |
| **Near limit** | ~180K+ | < 20K | < 20K | Review will likely fail. Defer to new session. |

#### 3.2.4 Revised Tier Thresholds

Tiers are set based on two constraints: (1) lead agent context efficiency and (2) subagent instruction-following quality.

| Tier | Word Count | Doc Tokens | Lead Context Cost (3x) | Lead Context Cost (extraction) | Strategy |
|---|---|---|---|---|---|
| **1 (Short)** | < 2,000 words | < ~2,600 | ~7,800 (3x) | n/a | Pass-through. 3x multiplier is affordable. |
| **2 (Medium)** | 2,000-5,000 words | ~2,600-6,500 | ~8K-20K (3x + map) | n/a | Section map + full content. 3x still fits in most sessions. |
| **3 (Long)** | 5,000+ words | > ~6,500 | ~20K+ (3x) | ~6K (extraction) | Extraction. 3x multiplier becomes expensive; extraction caps dispatch cost at ~6K regardless of doc size. |

**Why the Tier 3 threshold is 5K words (not 4K):** At 5K words, the 3x multiplier costs ~20K tokens in the lead agent. Above this, extraction provides meaningful savings (dispatch cost drops from 2 * doc_tokens to 2 * ~3K = ~6K). Below this, the savings don't justify the extraction overhead.

**Quick mode override:** Quick mode always uses Tier 3 to minimize both latency AND lead agent context consumption.

#### 3.2.5 Subagent Budget (Fresh 200K Window)

Subagents are not context-constrained for any realistic document. Budget allocation:

| Component | Tokens | Notes |
|---|---|---|
| System overhead | ~5,000 | Tool definitions, platform overhead |
| Agent prompt | ~2,000 | ~120 lines of instructions |
| SKILL.md (if auto-loaded) | ~6,000 | Rubrics, deduction table, routing table, etc. |
| Document/extraction content | Variable | Full doc (Tier 1-2) or extraction (Tier 3) |
| Output reserve | ~8,000 | Max output tokens for findings report |
| **Available for document** | **~179,000** | = ~137K words. Never the binding constraint. |

**The binding constraint for subagents is instruction-following quality, not tokens.** Per Pedro's finding, Claude reliably follows ~100-150 instructions. For documents over ~30K tokens (~23K words), even though they fit in the window, review quality degrades because the model's attention is spread thin. This is why Tier 3 extraction exists — it keeps subagent input focused regardless of original doc length.

### 3.3 Graceful Degradation (3 Levels)

The lead agent computes `review_cost` (from Section 3.2.2). Instead of trying to predict remaining context (see Section 3.6 on why we don't), the system uses a simple 3-level model:

```
Level 0: NORMAL
  Condition: Review proceeds as requested.
  Action:    Appropriate tier, both subagents, full output.

Level 1: AUTO-DOWNGRADE TO QUICK MODE
  Condition: Review output is truncated or incomplete, OR the lead agent
             detects the document exceeds 20K words and the user requested Full mode.
  Action:    Force Tier 3 extraction + Quick mode (tighter finding caps).
             Both subagents still run.
  User sees: "Note: Switching to Quick mode for this document. Run
             /ds-review:review --mode full in a fresh session for the
             complete lens-by-lens breakdown."

Level 2: DEFER TO NEW SESSION
  Condition: Review fails entirely (subagent errors, output fully truncated,
             or document too large to process in current session).
  Action:    Do not attempt partial review. Tell user to retry.
  User sees: "This review could not be completed in the current session.
             Please start a new terminal session and run
             /ds-review:review again. Tip: for best results, invoke
             the review at the start of a fresh session."
```

**Design rationale — why 3 levels, not 5:**

The original design had 5 degradation levels (including single-subagent and lead-only lightweight review). These were cut because:
- **Single-subagent mode** produces a review that's missing an entire dimension — half the value proposition. Users are better served by retrying in a fresh session than getting a one-dimensional review.
- **Lead-only lightweight mode** reduces the review to 4 generic checks. At that quality level, the tool isn't providing meaningful value over the user's own judgment.
- Both modes required their own output templates, scoring adjustments, and edge case handling — significant implementation cost for rarely-triggered paths.

The simplified model handles the same scenarios: if the review works, great (Level 0). If it's degraded, auto-fix by switching to Quick mode (Level 1). If it fails, tell the user to retry cleanly (Level 2).

### 3.4 Subagent Failure Handling

| Scenario | Lead Agent Action |
|---|---|
| Both subagents return successfully | Normal synthesis (Section 8) |
| One subagent returns, one fails/times out | Use the successful subagent's output. Mark the missing dimension as "Not reviewed" in output. Warn user. Show partial score from available dimension only. |
| Both subagents fail | Fall back to Level 2 (defer to new session). Do not attempt a degraded lead-only review — the quality is too low to be useful. |
| Subagent returns malformed output | Attempt to parse what's usable. If score is missing, derive from findings. If findings are truncated, use what's available and note "partial review." |

### 3.5 Output Truncation Mitigation

Even when token budgets are managed, subagent output can still be truncated if the model generates verbose findings. Defense in depth:

1. **Priority-ordered output:** "Emit findings CRITICAL first, MAJOR second, MINOR last." Truncation loses the least important findings.
2. **Finding caps:** Max 3 per lens (Full) / 2 per lens (Quick). Bounds output length predictably.
3. **Compact format enforced:** Each finding is a structured block (~100 tokens), not prose. 12 findings = ~1,200 tokens — well within output limits.
4. **Score and positives emitted last:** These are short and formulaic. If they're lost, the lead agent can derive the score from findings and generate its own encouragement.

### 3.6 Session Context: Reactive, Not Predictive

**v1 does not attempt to estimate remaining context.** The earlier design explored heuristics (turn counting, self-test output) to predict context pressure, but these were cut because:
- Turn count is unreliable — one turn could be 500 tokens or 15K
- Self-test output adds latency for a vague signal
- The conservative default (assume mid-session) causes frequent over-degradation, giving users worse reviews than they could get

**v1 approach: handle failure when it happens.**
- If the review completes normally → Level 0 (user sees full output)
- If output is truncated or incomplete → Level 1 (auto-downgrade to Quick, inform user)
- If the review errors out entirely → Level 2 (defer, tell user to retry in fresh session)

**User guidance (in plugin docs and degraded output):** "For best results with long documents, invoke /ds-review:review at the start of a fresh terminal session."

**v1.5 consideration:** If real usage shows that reviews frequently fail mid-session, revisit predictive detection. But don't build speculation-based logic before you have data showing it's needed.

---

## 4. Processing Tiers: Long Document Handling

### 4.1 Tiered Strategy

Tiers are calibrated against the 200K context window (see Section 3.2.4 for full derivation):

| Tier | Trigger | Tokens (doc) | Lead Context Cost | Strategy | What subagents receive |
|---|---|---|---|---|---|
| **1 (Short)** | < 2,000 words | < ~2,600 | ~8K (3x multiplier) | Pass-through | Full document content |
| **2 (Medium)** | 2,000-5,000 words | ~2,600-6,500 | ~8K-20K (3x + map) | Section map + full content | Full document + section map as attention guide |
| **3 (Long)** | 5,000+ words | > ~6,500 | ~6K fixed (extraction) | Structured extraction | Extraction only (verbatim excerpts, not summaries) |

**Why 5K words is the Tier 3 threshold:** Above 5K words, the 3x multiplier in the lead agent costs >20K tokens. Extraction caps dispatch cost at ~6K regardless of doc length — a significant savings that also protects against mid/deep session context pressure.

**Quick mode override:** Quick mode always uses Tier 3 to minimize both latency AND lead agent context consumption. This is a dual benefit — faster subagent processing AND lower lead agent token cost.

### 4.2 Section Map Format (Tier 2)

The lead agent creates this during the plan-first pass:

```
SECTION MAP
Document: [title]
Total length: [word count] (~[X] min read)

STRUCTURE:
- [H1] Title of Analysis
  - [H2] Executive Summary / TL;DR (lines 1-15) <- TL;DR DETECTED
  - [H2] Background & Motivation (lines 16-45)
  - [H2] Methodology (lines 46-120)
    - [expand macro] "Detailed assumptions" (deprioritize)
    - Key claims at lines 62-68
  - [H2] Results (lines 121-180)
    - [table] Metric comparison at line 135
    - [chart/image] referenced at line 142
  - [H2] Recommendations (lines 181-210)
  - [H2] Appendix (expand macro - deprioritize)

KEY CLAIMS (with locations):
1. "[verbatim claim]" (line X)
2. "[verbatim claim]" (line Y)
```

### 4.3 Handling Unstructured Documents

Not all DS analyses have clear headings or logical structure. Some are running text, some have vague headings ("Section 1", "Part A"), and some bury conclusions in the middle of a narrative.

**When the lead agent cannot identify a clear heading structure, it falls back to content-signal scanning:**

**Keyword/phrase detection (in order of priority):**

| Signal Type | Trigger Phrases | What it identifies |
|---|---|---|
| Conclusion signals | "in conclusion", "we recommend", "key finding", "therefore", "the result shows", "takeaway", "bottom line", "based on this analysis" | Where the author's conclusion lives — may not be at the end |
| Methodology signals | "we used", "the approach", "our model", "regression", "sample size", "population", "cohort", "treatment group", "control" | Where methodology is described |
| TL;DR signals | "in summary", "the bottom line", "key takeaway", "executive summary", "overview" | Summary location, if any |
| Limitation signals | "caveat", "limitation", "however", "should be noted", "does not account for", "out of scope" | Where limitations are stated |
| Result signals | Numbers, percentages, p-values, confidence intervals, metric names | Where quantitative results cluster |

**Positional heuristics:**
- First 15% of document: likely contains framing, context, and possibly TL;DR
- Last 15% of document: likely contains conclusions and recommendations
- Paragraphs with high density of numbers/statistics: likely results sections

**Extraction output for unstructured docs:**
```
DOCUMENT EXTRACTION (UNSTRUCTURED)
Note: This document has minimal explicit structure.
Key information inferred from content signals.

TL;DR AS WRITTEN: [best candidate paragraph, or ABSENT]
Source signal: [keyword match / positional / ABSENT]

INFERRED SECTIONS:
- Framing/context: [paragraphs 1-3, lines 1-30]
- Methodology (inferred): [paragraph with methodology signals, lines X-Y]
- Results (inferred): [paragraphs with high number density, lines X-Y]
- Conclusions (inferred): [paragraph with conclusion signals, lines X-Y]
- Limitations: [ABSENT or location]

KEY CLAIMS: [same format as structured extraction]
...
```

The extraction explicitly flags that structure was inferred, so subagents calibrate their evaluation accordingly. A finding like "No clear section structure — reader must work to find the methodology" becomes a legitimate communication finding, not a parsing error.

### 4.4 Structured Extraction Format (Tier 3, Structured Docs)

Uses **verbatim excerpts**, not summaries — subagents need the author's actual words.

```
DOCUMENT EXTRACTION
Source: [Confluence URL or file path]
Space: [Confluence space key, if applicable]
Word count: [X] (~[Y] min read) -> Tier 3
Status: [Draft | In Review | Final]
Labels: [list of Confluence labels, if applicable]

TL;DR AS WRITTEN:
[verbatim content from panel/callout/first paragraph]
Location: [top panel | info macro | first paragraph | ABSENT]

STRUCTURE (headings + macros):
[heading hierarchy with line/section references]
[note which sections are in expand macros]

KEY CLAIMS (verbatim quotes with locations):
1. "[exact quote]" (Section: Results, paragraph 2)
2. "[exact quote]" (Section: Methodology, info panel)
3. "[exact quote]" (Section: Recommendations)
...

METHODOLOGY EXCERPT:
[verbatim copy of methodology/approach section]

DATA SOURCES & METRICS:
[verbatim copy of data description and metric definitions]

RESULTS EXCERPT:
[verbatim copy of key results - tables described, not reproduced]

RECOMMENDATIONS / CONCLUSIONS:
[verbatim copy]

LIMITATIONS STATED:
[verbatim if present, "ABSENT" if not]

AUDIENCE SIGNALS:
[from Confluence labels, space context, document framing — informational only, not used for inference]

WORKFLOW CONTEXT:
[as specified by user via --workflow flag, or "general" if not specified]
```

### 4.5 Output Discipline

See Section 3.5 for full output truncation mitigation. Summary:

- **Priority-ordered output:** CRITICAL first, MAJOR second, MINOR last.
- **Finding caps:** 3 per lens (Full) / 2 per lens (Quick). Lead agent always selects **top 3 priority fixes** for output in **both modes**.
- **Compact finding format** (~100 tokens per finding).

---

## 5. Confluence Integration

### 5.1 Scope

**v1:** Single Confluence page only. If the page has child pages, the lead agent notes their existence but reviews only the target page. Multi-page analysis review is a v2 feature.

### 5.2 MCP Tools Available

Based on the Atlassian MCP ecosystem (official Atlassian Remote MCP server + sooperset/mcp-atlassian):

| Tool | Purpose | Our Usage |
|---|---|---|
| `confluence_get_page` | Get page content | Primary content fetch — returns page body, labels, status |
| `confluence_search` | Search with CQL | Optional — find pages by label, space, or title |

**Authentication:** OAuth (Atlassian Remote MCP) or API token via environment variables (sooperset/mcp-atlassian). Both operate within the signed-in user's permissions.

**Rate limits (Atlassian Remote MCP):** 500-10,000 calls/hour depending on plan. Our usage is 1-2 calls per review invocation — well within limits.

**Key implementation note:** The Confluence API does **not** expose reading time as a metadata field. Reading time displayed in the Confluence UI is computed client-side from word count. Our lead agent computes word count from the fetched page content and derives estimated reading time as `word_count / 230` for display in the review output.

### 5.3 Confluence Structural Elements

The lead agent parses Confluence-specific structural elements from the page content (storage format contains XHTML with Confluence macros):

| Confluence Element | What it signals | How the agent uses it |
|---|---|---|
| Info/Note/Warning panel | Key callouts — likely TL;DR, caveats, key findings | TL;DR detection heuristic checks panels in top 20% of page |
| Expand macro | Appendix-like content, supplementary detail | Deprioritize in extraction — note existence, don't include in main content |
| Status macro | Draft / In Review / Final | Adjust polish scoring — drafts get lighter polish penalties |
| Table of Contents macro | Page has intentional structure | Positive signal for Structure & TL;DR lens |
| Page labels | May indicate target audience or workflow stage | Used in Full mode for audience context (not for inference) |
| Inline comments | Existing reviewer feedback | Context for the agent — notes that peer review is in progress |
| Child pages | Analysis spans multiple pages | Note but don't review in v1 |

### 5.4 TL;DR Detection Heuristic

Confluence pages have dynamic structure — TL;DR doesn't always live under a heading called "TL;DR." The lead agent checks these locations in order:

1. **Panel macro in top 20% of page** — info panel, success panel, or note panel. Most DS orgs put the summary in a colored panel at the top.
2. **Bold/emphasized block in first section** — before the first H2 heading.
3. **Section explicitly named** "TL;DR", "Executive Summary", "Summary", "Key Findings" or similar.
4. **First paragraph** — if it reads like a summary (contains conclusion language, business impact framing).
5. **Content-signal scan** — if none of the above, use keyword detection (see Section 4.3).
6. **ABSENT** — no TL;DR detected. This becomes a finding for the communication reviewer.

### 5.5 Confluence Fetch Failure Handling

| Failure | Lead Agent Action |
|---|---|
| MCP server unavailable | Inform user: "Cannot connect to Confluence. Check MCP server status. You can copy the page content to a local .md file and run the review on that instead." |
| Authentication expired | Inform user: "Confluence authentication expired. Please re-authenticate your Atlassian MCP connection." |
| Page not found / no permissions | Inform user: "Cannot access this page. Verify the URL and your permissions." |
| Rate limit hit | Inform user: "Confluence API rate limit reached. Try again in a few minutes." |

---

## 6. Quick Mode vs Full Mode

| Aspect | Full Mode | Quick Mode |
|---|---|---|
| Plan-first step | Show plan, wait for user confirmation | **Skip entirely** |
| Audience | Ask user if not specified in invocation | **Default to mixed audience** (no inference) |
| Processing tier | Appropriate tier based on word count | **Always Tier 3** (extraction) regardless of length |
| Finding caps per lens | 3 per lens (12 per subagent) | 2 per lens (8 per subagent) |
| Top fixes in output | **Top 3** | **Top 3** |
| Subagents dispatched | Both, parallel | Both, parallel |
| Lens dashboard | Per-lens rating (SOUND / MINOR ISSUES / MAJOR ISSUES / CRITICAL) | Per-dimension status only (Pass / Issues Found / Critical Issues) |
| Lens-by-lens detail | Full findings per lens | Omitted |
| Latency target | < 5 minutes | < 3 minutes |

### 6.1 Quick Mode Audience Handling

Quick mode **does not infer audience**. It defaults to mixed audience unless the user explicitly passes `--audience [exec|tech|ds]`.

**Rationale:** Domain-specific jargon is unreliable for audience inference. In specialized fields like Search, even senior executives understand terms like NDCG, MRR, and CTR. Attempting to infer audience from language style wastes tokens and produces wrong results. Mixed audience is the safe default — it evaluates cross-audience effectiveness, which is the most broadly useful feedback.

---

## 7. Agent Prompt Design

All agent prompts follow Pedro Sant'Anna's structural pattern: **frontmatter -> role -> task -> lenses -> output format -> rules**. Target: ~120 lines each. Core checklist criteria live in the prompt; detailed rubric examples and anti-patterns reference SKILL.md.

### 7.1 ds-review-lead.md (Orchestrator)

**Responsibilities:**
1. Parse input and invocation intent (command flags or natural language)
2. Fetch content (`confluence_get_page` for Confluence, Read tool for local files)
3. Compute word count, derive reading time, determine processing tier
4. (No predictive degradation — degradation is reactive, see Section 3.3)
6. Check for partial input -> ask user to clarify intent if minimal
7. Mode branch: Full (plan -> confirm -> dispatch) or Quick (dispatch immediately)
8. Pre-process if Tier 2-3 (section map or structured extraction)
9. Dispatch both subagents in parallel via Task tool
10. Handle subagent results (success, partial failure, full failure)
11. Synthesize: merge findings, apply floor rules, compute final score, build lens dashboard
12. Produce output in mode-appropriate format

**Token budget awareness (informational):**
- After fetching content, the lead agent computes word count and estimates total token cost (Section 3.2.2)
- This informs tier selection (Section 4.1) but does NOT trigger predictive degradation
- Degradation is reactive — triggered only if the review fails or truncates (Section 3.3)

**Confluence integration:**
- Fetch page via `confluence_get_page` (single call returns content + metadata)
- Parse storage format for structural elements (panels, expands, headings)
- Compute word count from content, derive reading time for display
- Run TL;DR detection heuristic
- Note child pages but don't review them
- Handle fetch failures gracefully (Section 5.5)

**Scoring synthesis logic:**
- Combine subagent scores with equal weight (50/50) — `final = (analysis + communication) / 2`
- Apply floor rules from SKILL.md (CRITICAL caps verdict)
- Add cross-cutting assessment for issues spanning both dimensions
- Build lens dashboard from subagent per-lens ratings
- User sees: computed numeric score + floor-adjusted verdict + lens dashboard

**Output templates (embedded in prompt):**
- Full mode: Verdict -> Lens Dashboard -> Top 3 Priority Fixes -> Encouragement -> Lens-by-Lens Breakdown
- Quick mode: Verdict -> Dimension Dashboard -> Top 3 Priority Fixes -> Encouragement

**Key rules:**
- Report only — never edit or rewrite the user's analysis
- Always include positive findings (non-negotiable)
- Top 3 fixes in BOTH Full and Quick mode
- Show processing tier and degradation status in output metadata
- If degraded: always explain why and how to get a full review

### 7.2 analysis-reviewer.md (Subagent)

**Role:** Reviews the analysis dimension — methodology, logic, completeness, metrics.

**Lens 1: Methodology & Assumptions**
Core checklist (in prompt):
- Analytical approach appropriate for the question?
- All assumptions explicitly stated and sufficient?
- Assumption stress test: would weakening any assumption change the conclusion?
- Limitations acknowledged?
- Sampling/selection bias addressed or acknowledged?
- Causal claims supported by appropriate methodology?

**Lens 2: Logic & Traceability**
Core checklist (in prompt):
- Forward pass: does each reasoning step follow from the previous?
- Backward pass: starting from conclusion, can you trace to finding -> method -> data?
- No unsupported logical leaps?
- No circular reasoning?

**Lens 3: Completeness & Source Fidelity**
Core checklist (in prompt):
- Key questions addressed? Obvious follow-up analyses missing?
- Referenced sources, numbers, benchmarks accurately represented?
- External citations say what the analysis claims they say?

**Lens 4: Metrics**
Core checklist (in prompt):
- Right metrics selected for the question being answered?
- Baselines and benchmarks provided for context?
- Metrics defined clearly enough for the target audience to interpret?
- Statistical vs practical significance distinguished (where relevant)?

**Output:** Per-lens rating + structured findings (compact format) + 2-3 positive findings + subagent score (0-100).

**Rules:** Self-verify before reporting. Max 3 findings/lens (2 in Quick). Stay in your dimension — consult routing table. Priority-ordered output (CRITICAL first). Emit per-lens rating (SOUND / MINOR ISSUES / MAJOR ISSUES / CRITICAL) before findings.

### 7.3 communication-reviewer.md (Subagent)

**Role:** Reviews the communication dimension — structure, audience fit, conciseness, actionability.

**Lens 1: Structure & TL;DR**
Core checklist (in prompt):
- TL;DR present and effective? (Proactive: leads with insight + impact. Reactive: leads with direct answer.)
- Clear story arc? (What -> So What -> Now What)
- Argument flow matches audience's thinking style? (Inductive for exec, deductive for tech)
- Section headings serve as actionable signposts? ("Churn dropped 15%" not "Results")

**Lens 2: Audience Fit**
Core checklist (in prompt):
- Detail level calibrated for target audience?
- Jargon appropriate for the reader?
- Limitations and scope boundaries clear enough for downstream consumers?
- Cross-functional handoff safe? (Would someone not in the room interpret this correctly?)
- Recycled presentation for wrong audience?

**Lens 3: Conciseness & Prioritization**
Core checklist (in prompt):
- Every section, chart, and table earns its place?
- Right length for its purpose? Appendix material in main body?
- Visualizations support the narrative? (Clear labels, right chart type, no chartjunk)
- Professional polish? (Formatting consistency, spelling, visual hierarchy)

**Lens 4: Actionability**
Core checklist (in prompt):
- Proactive: recommendations specific, prioritized, with named owners and next steps?
- Reactive: measurement clear enough for stakeholder to act? Confidence intervals provided?
- Practical significance contextualized alongside statistical significance?
- Over-interpretation boundaries explicit? (What the data can and cannot support)
- "So what?" present for every key finding?

**Output:** Same format as analysis-reviewer (per-lens rating + findings + positives + score).

**Rules:** Same pattern. Additionally: adapt evaluation to audience persona and workflow context (general/proactive/reactive as specified by user or defaulted) per SKILL.md definitions.

### 7.4 Skill File Fallback (No Auto-Loaded Skills)

If the target CLI does not support auto-loaded skill files, the following content from SKILL.md must be embedded directly into each subagent's prompt:

**Must embed (critical for review quality):**
- Severity definitions (CRITICAL / MAJOR / MINOR) — ~10 lines
- Deduction table for the subagent's own lenses — ~15-20 lines
- Dimension boundary routing rules — ~10 lines

**Can omit (referenced but not essential in prompt):**
- Common anti-patterns with examples (subagents can identify anti-patterns from checklists alone)
- Confluence structure guide (lead agent handles this, not subagents)
- Detailed persona definitions (summary in prompt is sufficient)

**Impact on prompt length:** Embedding adds ~35-40 lines per subagent, pushing prompts from ~120 to ~155-160 lines. This approaches the ~150 instruction attention limit. Mitigation: condense the embedded content to the most essential rules and use SKILL.md-level detail only when skill files are available.

**Lead agent:** Always needs full context regardless of skill file support, because it handles orchestration, scoring synthesis, and Confluence parsing. If skills are unavailable, embed severity definitions, floor rules, and output templates directly. The lead agent prompt grows to ~150-160 lines.

---

## 8. Scoring System

### 8.1 Subagent-Level Scoring

Each subagent independently:
1. Evaluates each lens and assigns a **per-lens rating**: SOUND / MINOR ISSUES / MAJOR ISSUES / CRITICAL
2. Applies deductions from the deduction table (SKILL.md) for each finding
3. Computes its own score: 100 - (sum of deductions)
4. Reports: per-lens ratings + finding list + score + positive findings

This ensures **graceful degradation** — if one subagent fails, the other's output still provides meaningful feedback.

### 8.2 Lead Agent Synthesis

The lead agent:
1. Receives both subagent scores, per-lens ratings, and findings
2. Builds the **lens dashboard** from subagent per-lens ratings
3. Applies **floor rules** (from SKILL.md):
   - Any CRITICAL finding across either dimension -> verdict capped at Minor Fix (max 79)
   - 2+ CRITICAL findings across either dimension -> verdict capped at Major Rework (max 59)
4. Computes final score: **equal weight (50/50)** — `final_score = (analysis_score + communication_score) / 2`
5. Checks for cross-cutting issues (e.g., a methodology flaw that also affects communication)
6. Selects **top 3 priority fixes** across both dimensions (by severity, then impact) — **both modes**
7. Selects 2-3 positive findings (at least 1 from each dimension if available)

**v1 simplification — equal weight scoring:** Both dimensions are weighted equally. This avoids undefined behavior from an uncalibrated weighting formula. In v1.5, once we have calibration data from real reviews, we may adjust weights based on workflow context (e.g., reactive analyses might weight communication slightly higher). For now, 50/50 is transparent and predictable.

### 8.3 Verdict Bands

| Score | Verdict | Meaning |
|---|---|---|
| 80-100 | **Good to Go** | Analysis ready to share. Minor polish optional. |
| 60-79 | **Minor Fix** | Solid foundation, specific issues to address before sharing. |
| 0-59 | **Major Rework** | Significant gaps that would undermine effectiveness. |

### 8.4 Auditability

The user sees both the computed numeric score AND the floor-adjusted verdict. If floor rules override, the output explains why:

> **Score: 85 -> Verdict: Minor Fix (floor rule applied)**
> One CRITICAL finding detected (missing TL;DR). CRITICAL findings cap the verdict at Minor Fix regardless of computed score. Address this before sharing.

---

## 9. Output Format Templates

### 9.1 Full Mode Output

```markdown
# DS Analysis Review: [Document Title]

**Score: [X]/100 -- [Good to Go | Minor Fix | Major Rework]**
[Floor rule note if applicable]

**Mode:** Full | **Audience:** [persona] | **Workflow:** [general (default) | proactive | reactive]
**Processing:** Tier [1|2|3] ([word count] words, ~[X] min read)

### Lens Dashboard

| Dimension | Lens | Rating |
|---|---|---|
| Analysis | Methodology & Assumptions | [SOUND / MINOR ISSUES / MAJOR ISSUES / CRITICAL] |
| Analysis | Logic & Traceability | [rating] |
| Analysis | Completeness & Source Fidelity | [rating] |
| Analysis | Metrics | [rating] |
| Communication | Structure & TL;DR | [rating] |
| Communication | Audience Fit | [rating] |
| Communication | Conciseness & Prioritization | [rating] |
| Communication | Actionability | [rating] |

---

## Top 3 Priority Fixes

### 1. [Finding title] ([CRITICAL/MAJOR])
**Where:** [location in document]
**Issue:** [2-3 sentences]
**Suggested fix:** [concrete example showing how to address it]

### 2. [Finding title] ([CRITICAL/MAJOR])
...

### 3. [Finding title] ([CRITICAL/MAJOR/MINOR])
...

---

## What You Did Well

1. **[Positive finding title]** -- [1-2 sentences on what's strong and why]
2. **[Positive finding title]** -- [1-2 sentences]
3. **[Positive finding title]** -- [1-2 sentences]

---

## Analysis Dimension (Score: [X]/100)

### Methodology & Assumptions -- [rating]
[Findings for this lens, or "No issues found"]

### Logic & Traceability -- [rating]
[Findings]

### Completeness & Source Fidelity -- [rating]
[Findings]

### Metrics -- [rating]
[Findings]

---

## Communication Dimension (Score: [X]/100)

### Structure & TL;DR -- [rating]
[Findings]

### Audience Fit -- [rating]
[Findings]

### Conciseness & Prioritization -- [rating]
[Findings]

### Actionability -- [rating]
[Findings]
```

### 9.2 Quick Mode Output

```markdown
# DS Analysis Review: [Document Title]

**Score: [X]/100 -- [Good to Go | Minor Fix | Major Rework]**
[Floor rule note if applicable]

**Mode:** Quick | **Audience:** mixed (default) | **Workflow:** [general (default) | proactive | reactive]

### Status

| Dimension | Status |
|---|---|
| Analysis | [Pass / Issues Found / Critical Issues] |
| Communication | [Pass / Issues Found / Critical Issues] |

---

## Top 3 Priority Fixes

### 1. [Finding title] ([CRITICAL/MAJOR])
**Where:** [location]
**Issue:** [2-3 sentences]
**Suggested fix:** [concrete example]

### 2. [Finding title] ([CRITICAL/MAJOR])
...

### 3. [Finding title] ([CRITICAL/MAJOR/MINOR])
...

---

## What You Did Well

1. **[Positive]** -- [1-2 sentences]
2. **[Positive]** -- [1-2 sentences]
3. **[Positive]** -- [1-2 sentences]

---

*Run `/ds-review:review --mode full` for per-lens ratings and detailed findings.*
```

### 9.3 Degraded Mode Output

When the system operates at degradation Level 1-2 (Section 3.3), the output includes a notice:

```markdown
# DS Analysis Review: [Document Title]

> **[!] Capacity Notice:** [Explanation of degradation and how to get full review]

[Reduced output appropriate to degradation level]
```

---

## 10. Partial Input Handling

When the lead agent detects minimal or incomplete input (e.g., bullet points, half-written draft, outline only), it asks the user to clarify intent before dispatching subagents:

> "This looks like an early draft -- I see [X sections, ~Y words, no methodology section]. Would you like:
> 1. **Full review** -- I'll review what's here and flag what's missing
> 2. **Draft feedback** -- I'll focus on direction and structure with lighter expectations"

**Both paths run both subagents.** The difference is depth and output format:

**"Full review" path:**
- Both subagents review whatever is provided at full rigor
- Missing sections become findings (e.g., "No methodology section detected -- cannot evaluate rigor")
- Score reflects incompleteness as MAJOR findings, not as an error state
- Normal output format (verdict + score + lens dashboard + findings)

**"Draft feedback" path:**
- Both subagents run, but with adjusted expectations:
  - analysis-reviewer: flags methodology gaps as "consider adding" rather than penalizing absence. Focuses on whether the analytical direction is sound given what's written.
  - communication-reviewer: focuses on structure, TL;DR direction, and whether the argument arc is forming. Lighter on polish penalties.
- **No numeric score.** Draft feedback produces qualitative output only:
  - A qualitative assessment of analytical direction and communication arc
  - Top 3 things to address before finalizing
  - Positive findings (what's working so far)
- Finding severity is capped at MAJOR (no CRITICAL for draft mode) — nothing in a draft is "ready to share" by definition, so CRITICAL verdicts are not meaningful

**Why no numeric score for drafts:** A "Draft Score: 72/100" invites comparison with full review scores, but the two aren't comparable (different expectations, different severity caps). Users who see 72 on a draft and then 68 on the finished version will think their analysis got *worse*. Qualitative feedback is more honest and more useful at the draft stage.

---

## 11. SKILL.md Structure (Shared Knowledge File)

Located at `plugin/skills/ds-review-framework/SKILL.md`. Auto-activates for the plugin. Contains reference material that agent prompts draw from but don't duplicate.

### 11.1 Contents Overview

```
Section 1: SEVERITY DEFINITIONS
  - CRITICAL, MAJOR, MINOR -- definitions and examples

Section 2: DEDUCTION TABLE
  - Analysis-reviewer deductions by lens (from PRD Section 10.2)
  - Communication-reviewer deductions by lens (from PRD Section 10.2)
  - Each entry: issue type, lens, severity, deduction points, example

Section 3: FLOOR RULES
  - 1 CRITICAL -> verdict capped at Minor Fix (max 79)
  - 2+ CRITICAL -> verdict capped at Major Rework (max 59)
  - Floor rules override verdict band, not numeric score (auditability)

Section 4: AUDIENCE PERSONA DEFINITIONS
  - Business Executive (inductive: "so what?" first)
  - Technical Lead (deductive: evidence-first)
  - Peer Data Scientist (full rigor)
  - Mixed Audience (default: cross-audience synthesis)
  - Structural principle: deductive vs inductive framing

Section 5: DIMENSION BOUNDARY ROUTING TABLE
  - Defines which subagent owns gray-zone feedback
  - Prevents duplicate or conflicting findings
  - Examples:
    "metric without context" -> analysis-reviewer (Metrics lens)
    "metric without audience-appropriate baseline" -> communication-reviewer (Audience Fit)
    "missing limitation" -> communication-reviewer (Audience Fit: downstream impact)
    "flawed methodology" -> analysis-reviewer (Methodology) even if it affects communication
    "conclusion without evidence" -> analysis-reviewer (Logic) unless it's a framing issue
    "buried key finding" -> communication-reviewer (Structure & TL;DR)

Section 6: WORKFLOW CONTEXT
  - Proactive/Advisory mode: what good TL;DR and actionability look like
  - Reactive/Support mode: what good TL;DR and actionability look like
  - General mode (v1 default): evaluation criteria that apply regardless of workflow
  - Key principle: TL;DR and Actionability always heavily scored in all modes
  - Note: v1 defaults to "general" — user specifies via --workflow flag if needed. No inference.

Section 7: COMMON ANTI-PATTERNS
  - With examples and which lens catches each
  - "Burying the lede" -> Structure & TL;DR
  - "Recycled deck" -> Audience Fit
  - "Data dump" -> Conciseness & Prioritization
  - "Unsupported conclusion" -> Logic & Traceability
  - "Causal claim without comparability" -> Methodology & Assumptions
  - "Number without decision context" -> Actionability
  - "Over-interpretation boundary unclear" -> Actionability
  - etc.

Section 8: CONFLUENCE STRUCTURE GUIDE
  - Where TL;DR typically lives (panels, callouts, first paragraph)
  - Expand macros = appendix (deprioritize)
  - Label conventions and audience signals
  - Status macros and scoring implications
```

---

## 12. Implementation Scope

### 12.1 Files to Create/Modify

| File | Action | Content |
|---|---|---|
| `plugin/agents/ds-review-lead.md` | Replace placeholder | Orchestrator prompt (~120 lines, ~150-160 if no skill support) |
| `plugin/agents/analysis-reviewer.md` | Replace placeholder | Analysis subagent prompt (~120 lines, ~155-160 if no skill support) |
| `plugin/agents/communication-reviewer.md` | Replace placeholder | Communication subagent prompt (~120 lines, ~155-160 if no skill support) |
| `plugin/skills/ds-review-framework/SKILL.md` | Replace placeholder | Shared rubrics, deduction table, routing table, personas, anti-patterns, Confluence guide |
| `plugin/commands/review.md` | Update | Command definition with flag syntax |

### 12.2 What This Design Does NOT Cover

- Actual prompt text (this design specifies structure and content; the implementation session writes the prompts)
- Confluence MCP setup and configuration (depends on user's Atlassian plan and MCP choice)
- Deduction table calibration (v1 starting values from PRD; calibration in v1.5)
- Multi-page Confluence analysis review (v2)
- Non-Confluence/non-md input formats (v1.5)
- Auto-eval judge agent implementation (design in Section 15.3; build in v1.5)

---

## 13. Open Items for Implementation

| # | Item | Notes | Priority |
|---|---|---|---|
| 1 | Confluence MCP server choice | Atlassian Remote MCP (OAuth) vs sooperset/mcp-atlassian (API token). Both expose `confluence_get_page`. Depends on user's setup. | v1 — decide before implementation |
| 2 | Deduction point calibration | PRD Section 10.2 provides starting values. Real calibration requires testing with actual DS analyses. | v1 starting values, calibrate in v1.5 |
| 3 | Extraction fidelity for Tier 3 | Structured extraction may miss nuances. Subagents can use the Read tool to check specific sections if needed (extraction includes location references). | v1 — test with real docs |
| 4 | Auto-eval pipeline implementation | Design described in Section 15.3. Build the judge agent and scoring rubric. | v1.5 |
| 5 | Review quality feedback loop | Simple "Was this review helpful?" at end of output. Collect signal for iteration. | v1.5 |
| 6 | Workflow context inference | Currently flag-based (--workflow). May add content-based inference if users frequently don't specify. | v1.5 — only if usage data justifies |
| 7 | Score weighting by workflow context | Currently 50/50. May adjust weights based on calibration data (e.g., reactive -> slightly higher communication weight). | v1.5 — requires calibration data |

**Resolved from earlier revisions:**
- ~~Session freshness detection~~ — cut for v1 (Section 3.6). Reactive failure handling instead.
- ~~Subagent score weighting~~ — set to equal weight 50/50 for v1 (Section 8.2).
- ~~Draft mode scoring calibration~~ — cut numeric draft scoring entirely (Section 10). Qualitative output only.

---

## 14. Cost Estimation & Operational Notes

### 14.1 Estimated Token Cost Per Review

Each review consumes tokens across the lead agent and two subagent calls. Rough estimates:

| Scenario | Lead Agent | Subagent 1 | Subagent 2 | Total Input Tokens | Total Output Tokens |
|---|---|---|---|---|---|
| Short doc (1K words), Full mode | ~40K | ~15K | ~15K | ~70K | ~12K |
| Medium doc (4K words), Full mode | ~50K | ~20K | ~20K | ~90K | ~12K |
| Long doc (10K words), Tier 3 | ~55K | ~15K | ~15K | ~85K | ~12K |
| Long doc (20K words), Tier 3 | ~65K | ~15K | ~15K | ~95K | ~12K |
| Quick mode (any length, Tier 3) | ~55K | ~15K | ~15K | ~85K | ~8K |

**Key insight:** Tier 3 extraction keeps subagent costs roughly constant regardless of original doc length. The lead agent's cost grows with doc length (it reads the full document), but dispatch costs stay flat.

**User implication:** At current API pricing, each review costs roughly equivalent to processing 80-100K tokens. Users running 5+ reviews per day should be aware of cumulative cost.

### 14.2 Iterative Use Pattern

**v1: Each review is fully independent.** If a user fixes issues from a first review and runs the agent again, the second review starts from scratch with no memory of the first. This is the simplest model and avoids stale-state bugs.

**What this means for users:**
- The score may go up, down, or stay the same across runs — the agent may notice different things each time
- There's no "diff" mode showing what improved since the last review
- For best results, address all top 3 fixes before re-running

**v1.5 consideration:** If users frequently re-review documents, consider a lightweight diff mode that compares the current document against a cached extraction from the previous review. This is a significant feature — scope it separately.

### 14.3 Review Quality Feedback (v1.5)

**v1 does not include a feedback mechanism.** The review output is final — there's no "Was this helpful?" prompt or thumbs up/down.

**Why this matters:** Without feedback, we can't distinguish between:
- Reviews that found real issues (true positives)
- Reviews that flagged non-issues (false positives — erode trust quickly)
- Issues the review missed (false negatives — invisible without user reporting)

**v1.5 plan:** Add a simple feedback prompt at the end of review output:
> "Was this review helpful? Reply with any feedback to help improve future reviews."

This is a lightweight signal collection mechanism. Combined with the auto-eval pipeline (Section 15), it provides both user-reported and automated quality metrics.

---

## 15. Testing & Evaluation Strategy

### 15.1 Why This Section Exists

The architecture is only as good as the reviews it produces. A beautiful 3-agent system that outputs inaccurate, unhelpful, or inconsistent reviews is worthless. Testing validates that the system meets its quality bar before shipping v1, and auto-eval enables iteration after shipping.

### 15.2 v1 Testing: Manual Evaluation

**Two types of test documents:**

#### Real Documents (User-Provided)

Provide 3-5 actual DS analyses from your Confluence space covering a range of quality and length:

| Test Doc | Purpose | What it validates |
|---|---|---|
| A strong analysis (~2K words) | Baseline — should score high (80+) | Are scores calibrated? Does "Good to Go" feel right? |
| A mediocre analysis (~4K words) | Mid-range — should identify real issues | Are findings accurate and actionable? |
| A problematic analysis (~3K words) | Should flag clear methodology/communication issues | Does the system catch what a human reviewer would catch? |
| A long analysis (8K+ words) | Tests Tier 3 extraction | Does the extraction preserve enough content for quality review? |
| A Confluence page with panels/macros | Tests Confluence integration | Are structural elements parsed correctly? TL;DR detected? |

**What to evaluate per test run (manual rubric):**

| Criterion | Question | Pass/Fail |
|---|---|---|
| Finding accuracy | Are all reported findings real issues in the document? | No false positives |
| Severity calibration | Are CRITICAL findings actually critical? Are MINORs actually minor? | Severities match human judgment |
| Coverage | Did the review miss any obvious issues a human reviewer would catch? | No major false negatives |
| Actionability | Could the author act on each finding without further clarification? | All findings have clear next steps |
| Top 3 relevance | Are the top 3 priority fixes actually the 3 most important things? | Top 3 match human prioritization |
| Positive findings | Do the positive findings ring true? (Not generic filler) | Positives are specific and accurate |
| Score believability | Does the numeric score feel right for this document? | Score matches intuitive assessment |
| Output format | Is the output well-structured and easy to scan? | Format matches templates |
| Cross-run consistency | Run the same doc 3x — do findings and scores converge? | Score within +/- 10 points, same top findings |

#### Synthetic Documents (Agent-Generated)

Create 6-8 synthetic analyses designed to trigger specific findings. These provide systematic coverage of each lens:

| Synthetic Doc | Designed Deficiency | Primary Lens Tested |
|---|---|---|
| Analysis with no TL;DR | Missing executive summary | Structure & TL;DR |
| Causal claims without proper methodology | "X caused Y" without controls | Methodology & Assumptions |
| Perfect analysis, terrible communication | Sound methods but buried lede, jargon soup | Dimension separation |
| Conclusion contradicts data | Results say X, conclusion says Y | Logic & Traceability |
| Data dump with no recommendations | Tables and charts with no "so what?" | Actionability |
| Copy-pasted exec deck repackaged for DS audience | Wrong detail level for audience | Audience Fit |
| Very short doc (~500 words, bullet points) | Partial input | Partial input handling |
| Unstructured running text (~3K words) | No headings or sections | Unstructured doc handling |

**Synthetic doc generation process:**
1. Claude generates a realistic but flawed DS analysis for each scenario
2. The author (user) reviews each synthetic doc to confirm the designed deficiency is present and realistic
3. The synthetic doc becomes a test fixture in `dev/test-fixtures/`
4. Expected findings are documented alongside each fixture for comparison

### 15.3 v1.5 Auto-Eval: LLM-as-Judge Pipeline

**Purpose:** Enable automated quality measurement so you can iterate on prompts without manually reviewing every test case.

**Architecture:**

```
Test fixture (source doc + expected findings)
        |
        v
  DS Review Agent (produces review output)
        |
        v
  Judge Agent (evaluates review quality)
        |
        v
  Quality scores per dimension
```

**Judge agent design:**
- Separate agent, not part of the review system
- Receives: source document + review output + expected findings (gold standard)
- Evaluates review on 5 dimensions (see rubric below)
- Outputs a quality score per dimension + overall quality score

**Judge rubric (5 dimensions, weighted):**

| Dimension | Weight | What it measures |
|---|---|---|
| Finding accuracy | 30% | Did the review correctly identify issues that exist in the gold standard? Penalize false positives and false negatives. |
| Severity calibration | 20% | Are severity levels (CRITICAL/MAJOR/MINOR) aligned with gold standard? |
| Coverage | 20% | What fraction of gold standard findings were caught? |
| Actionability | 15% | Are suggested fixes concrete and actionable? |
| Consistency | 15% | Across 3 runs on the same doc, how stable are findings and scores? |

**Building the gold standard:**
- Start from your v1 manual evaluation results
- For each test fixture (real + synthetic), document:
  - Expected findings with severity levels
  - Expected score range (+/- 10 points)
  - Expected verdict band
  - Key findings that must NOT be missed
- This gold standard becomes the calibration set for the judge agent

**How to use auto-eval for prompt iteration:**
1. Change a prompt (e.g., adjust a lens checklist or deduction weight)
2. Run all test fixtures through the review agent
3. Run all review outputs through the judge agent
4. Compare quality scores to baseline
5. If quality improved or held steady: keep the change. If it dropped: revert.

Think of it like an A/B test for prompt quality — the gold standard is your success metric, and the judge agent is your evaluation function.

### 15.4 Test Fixture Storage

```
dev/
  test-fixtures/
    real/
      README.md               # Instructions for contributing real test docs
      [real docs go here]      # Not committed to git (may contain sensitive data)
    synthetic/
      no-tldr-analysis.md
      causal-without-method.md
      good-analysis-bad-comms.md
      contradicts-data.md
      data-dump.md
      wrong-audience.md
      partial-input-bullets.md
      unstructured-text.md
    expected/
      no-tldr-analysis.expected.md     # Expected findings + severity + score range
      causal-without-method.expected.md
      ...
```

**Real test docs are not committed to git** — they may contain sensitive business data. The README explains how to add them locally.

---

## Appendix A: Reference Material

- **PRD:** `dev/specs/PRD-DS-Analysis-Review-Agent.md`
- **Architecture session handoff:** `dev/specs/architecture-session-handoff.md`
- **Pedro Sant'Anna's workflow:** [website](https://psantanna.com/claude-code-my-workflow/) | [repo](https://github.com/pedrohcgs/claude-code-my-workflow)
- **Repo structure spec:** `ds-analysis-review-agent-structure.md`
- **ADR 001 (two-dimension split):** `dev/decisions/001-two-dimension-split.md`
- **Atlassian Remote MCP:** https://www.atlassian.com/platform/remote-mcp-server
- **sooperset/mcp-atlassian:** https://github.com/sooperset/mcp-atlassian
