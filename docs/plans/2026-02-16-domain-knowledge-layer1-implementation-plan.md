# Domain Knowledge Skill (Layer 1) — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a standalone Domain Knowledge Skill for Search that any agent can consume. Pre-built digest files containing curated domain expertise, accessible via simple file reads.

**Architecture:** YAML index defines what docs to track. SKILL.md defines the digest format contract. Digest files are the product — pre-built, versioned markdown files with structured sections, authority tags, and audience tags. Consumer agents just read the files.

**Tech Stack:** Markdown, YAML. No scripts, no external dependencies.

**Scope:** Layer 1 only. Layer 2 (reviewer subagent) and Layer 3 (lead integration) come after validation.

**Design docs:**
- `docs/plans/2026-02-16-domain-knowledge-mvp-design.md` — MVP scope decisions (A1-A5)
- `docs/plans/2026-02-15-domain-knowledge-subagent-design-v3.md` — Full v3 spec

---

## Files to Create

| File | Purpose |
|---|---|
| `plugin/config/domain-index.yaml` | Curated domain-to-pages mapping (2 sub-domains + cross-domain) |
| `plugin/skills/domain-knowledge/SKILL.md` | Skill contract: digest format, consumption guide, staleness rules, refresh workflow |
| `plugin/digests/search-ranking.md` | Primary digest — Search Ranking/Relevance foundational + workstream |
| `plugin/digests/query-understanding.md` | QU pipeline evaluation standards |
| `plugin/digests/search-cross-domain.md` | Cross-cutting Search evaluation knowledge |

**Deferred domains:** search-infra (least useful for DS review), search-experience (future).

---

## Task 1: Create directory structure and domain-index.yaml

**Files:**
- Create: `plugin/config/domain-index.yaml`
- Create: `plugin/digests/` (directory)

**Step 1: Create directories**

```bash
mkdir -p plugin/config plugin/digests
```

**Step 2: Create domain-index.yaml**

Manually curated index. No auto-discovery (v1). Placeholder page IDs (user replaces
with real Confluence IDs). Audience tags per MVP decision A2. Token budgets per A2/A5.

```yaml
# Domain Knowledge Index — Search MVP
# All docs manually curated. Auto-discovery deferred to v1.
# Page IDs are placeholders — replace with real Confluence page IDs.

search-ranking:
  pages:
    - id: "SR-FOUND-001"
      title: "Search Ranking Evaluation Standards"
      tier: foundational
      audience: all
    - id: "SR-FOUND-002"
      title: "Position Bias Correction Methods"
      tier: foundational
      audience: all
    - id: "SR-FOUND-003"
      title: "Click Model & Training Data Guidelines"
      tier: foundational
      audience: ds
    - id: "SR-FOUND-004"
      title: "Interleaving Experiment Design"
      tier: foundational
      audience: all
    - id: "SR-WS-001"
      title: "Q1 2026 Ranking Evaluation Playbook"
      tier: workstream
      audience: all
  refresh:
    foundational-cadence: monthly
    workstream-cadence: weekly
    digest-path: "plugin/digests/search-ranking.md"
    token-budget: 8000
    retain-versions: 4

query-understanding:
  pages:
    - id: "QU-FOUND-001"
      title: "QU Pipeline Evaluation Standards"
      tier: foundational
      audience: all
    - id: "QU-FOUND-002"
      title: "Query Rewriting Impact Measurement"
      tier: foundational
      audience: ds
    - id: "QU-FOUND-003"
      title: "Intent Classification Taxonomy"
      tier: foundational
      audience: all
    - id: "QU-WS-001"
      title: "Query Rewriting v3 Design Decisions"
      tier: workstream
      audience: all
  refresh:
    foundational-cadence: monthly
    workstream-cadence: weekly
    digest-path: "plugin/digests/query-understanding.md"
    token-budget: 8000
    retain-versions: 4

# search-infra: deferred (least useful for DS review MVP)
# search-experience: deferred (future domain)

# Cross-domain: auto-included when any listed sub-domain is requested.
# Gets its own additive budget (1,500 tokens) per MVP decision A5.
search-cross-domain:
  pages:
    - id: "SX-FOUND-001"
      title: "End-to-End Search Evaluation Guide"
      tier: foundational
      audience: all
    - id: "SX-FOUND-002"
      title: "Cross-Component Metric Attribution"
      tier: foundational
      audience: ds
  applies-to: ["search-ranking", "query-understanding"]
  refresh:
    foundational-cadence: monthly
    digest-path: "plugin/digests/search-cross-domain.md"
    token-budget: 1500
    retain-versions: 4
```

**Step 3: Validate YAML syntax**

```bash
python3 -c "import yaml; yaml.safe_load(open('plugin/config/domain-index.yaml'))"
```

**Step 4: Commit**

```bash
git add plugin/config/domain-index.yaml
git commit -m "feat(domain-knowledge): add curated domain index for Search MVP"
```

---

## Task 2: Create domain-knowledge SKILL.md

**Files:**
- Create: `plugin/skills/domain-knowledge/SKILL.md`

The contract definition. Tells any consuming agent: what digest files are, how to read
them, what the format means, how to check staleness, how to trigger refresh.

**Step 1: Create SKILL.md**

```markdown
---
name: domain-knowledge
description: >
  Standalone domain knowledge service for Search. Provides pre-built digest files
  containing curated domain expertise that any agent can consume. Owns: YAML index,
  digest format contract, refresh workflow, importance scoring, token budgeting.
  Currently scoped to Search (Ranking, Query Understanding, Infrastructure).
auto_activate: true
---

# Domain Knowledge Skill

Standalone service providing domain expertise as pre-built digest files. Any agent
can consume these files — DS review agents, code review agents, planning agents,
onboarding tools. The skill owns the knowledge lifecycle; consumers just read files.

---

## 1. Digest File Contract

Each domain has one digest file at the path specified in `plugin/config/domain-index.yaml`.
Digest files are markdown with structured headers. The format is the API.

### Required Headers

Every digest file starts with a metadata block:

    # {domain-name} domain digest
    # Version: {ISO-8601 timestamp}
    # Previous: {ISO-8601 timestamp of prior version}
    # Token budget: {number}
    # Audience tags: all, ds, eng

### Section Structure

Sections follow this pattern, always in this order:

    ## Foundational Knowledge [authority: authoritative] [audience: {tag}] (monthly refresh)
    ### {Topic heading}
    - {Knowledge item}

    ## Workstream Standards [authority: authoritative] [audience: {tag}] (weekly refresh)
    ### {Topic heading}
    - [{date}] {Decision or standard}

    ## Workstream Learnings [authority: advisory] [audience: {tag}] (weekly refresh)
    ### {Topic heading}
    - [{date}] {Learning or finding}

    ## Conflicts
    - [{date}] CONFLICT: {description}
      → Workstream takes precedence. Flagged for foundational review.

### Authority Levels

- **authoritative** — Foundational knowledge and workstream standards. Contradicting
  this content in an analysis is a finding. Consuming agents apply full deductions.
- **advisory** — Workstream learnings (recent experiments, post-mortems). Content here
  suggests improvements but should not trigger hard deductions. Consuming agents cap
  findings sourced from advisory content at ADVISORY severity (-2).

### Audience Tags

Each section is tagged: `all`, `ds`, or `eng`.
- `all` — relevant to any consumer
- `ds` — primarily relevant to data science methodology and analysis
- `eng` — primarily relevant to engineering implementation and infrastructure

Consumer agents filter sections by their audience need.

---

## 2. How to Consume Digests

### Single Domain

1. Read `plugin/config/domain-index.yaml` to find the digest path.
2. Read the digest file using the Read tool.
3. Check the `# Version:` header. If older than 14 days, emit staleness warning.
4. Filter sections by audience tag if your use case is audience-specific.
5. Include the content in your prompt/context as reference material.

### Multi-Domain

1. Read digest files for each requested domain.
2. Read the cross-domain digest if any requested domain appears in its `applies-to` list.
3. Concatenate. Cross-domain content included once regardless of how many domains reference it.
4. If total content exceeds context constraints, prioritize:
   foundational > workstream standards > workstream learnings.

### Token Budgets

- Per domain: 8,000 tokens (set in domain-index.yaml)
- Cross-domain: 1,500 tokens (additive, not shared with domain budgets)
- Multi-domain scales linearly: 2 domains = 16,000 + 1,500 cross-domain

---

## 3. Staleness Thresholds

| Condition | Warning |
|---|---|
| Digest version > 14 days old | "Digest may be stale. Consider refreshing." |
| Digest version > 30 days old | "Digest significantly outdated. Refresh recommended." |
| Digest file missing | "No digest found for {domain}. Run --refresh-domain to generate." |
| Digest file empty | "Digest for {domain} is empty. Check domain-index.yaml." |

Always proceed with available content. A stale digest is better than no digest.

---

## 4. Refresh Workflow (Manual — v0.5)

Refresh is manually triggered. No auto-discovery, no scheduled cadence.

### Trigger

`--refresh-domain {domain}` (e.g., `--refresh-domain search-ranking`).

### Process

1. Read domain-index.yaml for the target domain.
2. For each page: fetch content (Confluence MCP or user-provided), score importance.
3. Allocate token budget proportionally by importance score.
4. Generate digest with tier sections, authority tags, audience tags.
5. Version: copy current → `{domain}.{timestamp}.md`, write new → `{domain}.md`.
6. Anomaly check: if empty or >50% token drop, warn and keep previous.

### Rollback

`--rollback-domain {domain}` reverts to previous version.

---

## 5. Importance Scoring

LLM scores each document during refresh on two dimensions:

    knowledge_density (0.0-1.0): How much extractable domain knowledge?
    review_impact (0.0-1.0): If unknown, would analyses have domain errors?

    final_importance = (0.6 * review_impact) + (0.4 * knowledge_density)
    token_budget_per_doc = (final_importance / sum_all) * domain_budget

No Confluence tiebreaker or team roster in v0.5.

---

## 6. Knowledge Tier Precedence

When foundational and workstream knowledge conflict:
1. Workstream takes precedence — team's current understanding overrides defaults.
2. Conflict is flagged in the Conflicts section.
3. Next foundational refresh reviews the conflict.
```

**Step 2: Validate structure**

Check: frontmatter (name, description, auto_activate), 6 numbered sections present.

**Step 3: Commit**

```bash
git add plugin/skills/domain-knowledge/SKILL.md
git commit -m "feat(domain-knowledge): add skill contract definition"
```

---

## Task 3: Create search-ranking digest

**Files:**
- Create: `plugin/digests/search-ranking.md`

Primary digest. Largest sub-domain. Template for others. Foundational content is real
Search domain knowledge written in reviewer-oriented format (what to look for, what's
wrong). Workstream sections are placeholders for user to fill with team-specific content.

**Step 1: Create search-ranking.md**

```markdown
# search-ranking domain digest
# Version: 2026-02-16T00:00:00Z
# Previous: none
# Token budget: 8000
# Audience tags: all, ds, eng

## Foundational Knowledge [authority: authoritative] [audience: all] (monthly refresh)

### Evaluation Metrics — When to Use What
- **NDCG@10** is the primary metric for ranking quality with graded relevance judgments.
  NDCG@5 for mobile-first evaluation. Competitive systems score 0.40-0.55 on standard
  benchmarks (MS MARCO, TREC). An analysis citing NDCG > 0.80 on a standard benchmark
  likely uses a non-standard setup — flag for verification.
- **MRR (Mean Reciprocal Rank)** is preferred for single-answer retrieval tasks
  (navigational queries, QA, known-item search). If an analysis evaluates a single-answer
  task using NDCG instead of MRR, flag: NDCG rewards ordering across all positions, but
  for single-answer tasks only the first relevant result matters.
- **ERR (Expected Reciprocal Rank)** models cascade user behavior. Preferred over NDCG
  when modeling realistic user satisfaction on web search.
- **MAP (Mean Average Precision)** is for binary relevance. Standard for precision-oriented
  tasks (legal search, patent search) but less common for graded relevance.
- **Offline vs. online metric gap:** Offline metric lifts often do not translate 1:1 to
  online lifts. A statistically significant offline improvement is necessary but not
  sufficient for a launch recommendation. If the analysis recommends a ranking change
  based solely on offline metrics without online validation, flag.

### Position Bias
- Click data is biased by position: higher-ranked items get more clicks regardless of
  quality. Any analysis using raw click data without addressing position bias has a
  critical methodological gap.
- **IPW (Inverse Propensity Weighting):** Standard correction. Well-established but
  high variance on tail queries.
- **Doubly-robust estimation:** Combines propensity model with direct model for reduced
  variance. Preferred over IPW alone when both models are available.
- **Randomized data collection:** Gold standard for judgment quality but temporarily
  degrades user experience.

### Selection Bias in Logged Data
- Click-through training data reflects the current ranker's policy. Models trained on
  logged data may not generalize to different ranking policies.
- If the analysis trains on logged click data without acknowledging logging policy bias,
  flag: standard practice is counterfactual correction or acknowledging the limitation.

### Experiment Methodology
- **Interleaving experiments** (Team Draft, Balanced) are preferred for ranking changes.
  More sensitive than A/B — detect smaller quality differences with fewer queries.
- **A/B testing** is appropriate for downstream business metrics (revenue, engagement)
  that interleaving cannot capture. Both approaches are complementary.
- **Minimum experiment duration:** 1-2 weeks to capture weekday/weekend patterns, novelty
  effects, and user adaptation. <7 days of A/B data is insufficient for launch decisions.

## Foundational Knowledge [authority: authoritative] [audience: ds] (monthly refresh)

### Click Models
- **PBM (Position-Based Model):** Click = examination × attractiveness. Standard baseline.
- **Cascade Model:** User scans top-to-bottom. Appropriate for navigational queries.
- **DBN (Dynamic Bayesian Network):** Models satisfaction and abandonment. Better for
  informational queries with multiple relevant results.
- Verify the click model matches the query type distribution. Cascade model for broad
  informational queries is a technique mismatch.

### Learning to Rank
- **Pointwise:** Predict per-document relevance. Simple, appropriate for prototyping.
- **Pairwise** (RankNet, LambdaRank): Predict relative ordering. Standard for production.
- **Listwise** (LambdaMART, ListNet): Optimize entire ranked list. Generally best performance.
- If a pointwise approach is used for production without justifying why pairwise/listwise
  was not used, flag: domain standard is pairwise or listwise for production ranking.

## Foundational Knowledge [authority: authoritative] [audience: eng] (monthly refresh)

### Serving & Latency
- Ranking model inference must fit within serving latency budget. Typical p99 targets:
  L1 retrieval < 50ms, L2 ranking < 100ms, full stack < 200ms.
- If a ranking model change is proposed without discussing latency impact, note this gap.

## Workstream Standards [authority: authoritative] [audience: all] (weekly refresh)

### Active Standards
- [PLACEHOLDER] Replace with team's current evaluation standards and project decisions.
- Example: [2026-02-12] Third-party connector: use federated ranking with per-source
  quality scores.
- Example: [2026-02-08] Position bias v2: switching from IPW to doubly-robust.

## Workstream Learnings [authority: advisory] [audience: ds] (weekly refresh)

### Recent Experiment Learnings
- [PLACEHOLDER] Replace with recent experiment outcomes. These generate ADVISORY (-2) only.
- Example: [2026-02-10] Embedding retrieval A/B: p99 latency regressed 40ms at 10% traffic.

### Recent Post-Mortems
- [PLACEHOLDER] Replace with recent incident learnings.
- Example: [2026-01-28] CTR optimization incident: optimizing CTR alone led to clickbait.

## Conflicts
(No conflicts in initial digest)
```

**Step 2: Verify format matches SKILL.md contract**

Check: metadata headers, all section types present, authority/audience tags on every section.

**Step 3: Commit**

```bash
git add plugin/digests/search-ranking.md
git commit -m "feat(domain-knowledge): add search-ranking digest"
```

---

## Task 4: Create remaining digests

**Files:**
- Create: `plugin/digests/query-understanding.md`
- Create: `plugin/digests/search-cross-domain.md`

Same format as search-ranking. Smaller content — these have fewer foundational items
for MVP. User expands over time. search-infra deferred.

**Step 1: Create query-understanding.md**

Foundational content (audience: all):
- Query classification taxonomies (informational/navigational/transactional)
- Intent detection evaluation: precision and recall at intent level
- Query segmentation: compound query splitting evaluation

Foundational content (audience: ds):
- Query rewriting: must measure impact on downstream ranking metrics, not just rewrite
  quality in isolation. If analysis evaluates rewrites by edit distance or BLEU only, flag.
- Spell correction: precision > recall — false corrections damage user trust more than
  missed corrections. Evaluate with precision-weighted metrics.

Workstream: placeholders (same pattern as search-ranking).

**Step 2: Create search-cross-domain.md**

Foundational content (audience: all):
- End-to-end search evaluation: full-stack metrics vs. component metrics. An analysis
  that only measures component metrics (e.g., QU accuracy) without connecting to
  end-to-end user outcomes has a completeness gap.
- Cross-component experiments: when a QU change is launched, ranking metrics may shift.
  Analysis should account for cross-component effects.

Foundational content (audience: ds):
- Cross-component metric attribution: how to isolate QU vs. ranking vs. infra effects.
  Standard approach: counterfactual evaluation or sequential A/B testing.
- Full-stack experiment design: control for upstream/downstream dependencies.

Workstream: none (cross-domain is foundational-only in MVP).

**Step 3: Validate all digests match contract**

For each digest, verify:
- Metadata headers present (Version, Previous, Token budget, Audience tags)
- Section headers follow the contract pattern
- Authority and audience tags on every section

**Step 4: Commit**

```bash
git add plugin/digests/
git commit -m "feat(domain-knowledge): add QU and cross-domain digests"
```

---

## Task 5: Final validation and cleanup

**Step 1: Verify complete file set**

```bash
ls -la plugin/config/domain-index.yaml
ls -la plugin/skills/domain-knowledge/SKILL.md
ls -la plugin/digests/
```

Expected: 1 YAML file, 1 SKILL.md, 3 digest files (ranking, QU, cross-domain).

**Step 2: Verify YAML index paths match actual digest file paths**

Cross-check each `digest-path` in domain-index.yaml against files in `plugin/digests/`.

**Step 3: Verify cross-domain applies-to references valid domains**

Check that `search-cross-domain.applies-to` lists domains that exist in the index.

**Step 4: Read each digest and verify it follows the SKILL.md contract**

Quick manual review: headers, sections, tags.

**Step 5: Update backlog and create session log**

Per session end protocol:
- Update `dev/backlog.md` with Layer 1 completion status
- Create `dev/sessions/2026-02-16-domain-knowledge-layer1.md`
- Note: Layer 2 and Layer 3 are next (future session)

---

## Verification Checklist

| Check | How |
|---|---|
| YAML parses cleanly | `python3 -c "import yaml; yaml.safe_load(open(...))"` |
| All 3 digests exist | `ls plugin/digests/` → 3 .md files |
| Digest format matches contract | Manual: headers, sections, authority/audience tags |
| Cross-domain references valid | Index `applies-to` lists match domain names in index |
| Digest paths in index match files | Each `digest-path` points to an existing file |
| Foundational content is accurate | User review: domain knowledge is correct and useful |
| Workstream placeholders are clear | Placeholder format is obvious and easy to fill in |

---

## What Comes Next (Future Sessions)

| Phase | What | Depends on |
|---|---|---|
| Layer 2 | Domain Expert Reviewer subagent (3 lenses, deduction tables) | Layer 1 validated |
| Layer 2 | Update ds-review-framework SKILL.md (ADVISORY severity, domain routing) | Layer 1 validated |
| Layer 3 | Lead agent integration (--domain flag, 3rd subagent, 50/25/25 scoring) | Layer 2 complete |
| Layer 3 | Update review.md command with new flags | Layer 2 complete |
| Calibration | Run fixtures with 3-dimension scoring, compare to 2-dimension baselines | Layer 3 complete |
