# DS Productivity Agents

A suite of Claude Code agents for data science workflows in Search Relevance. Built on shared domain knowledge for Query Understanding, Search Ranking, and Search Infrastructure.

## Agents

### 📊 DS Analysis Review (`/ds-review`)

Reviews completed DS analyses across three dimensions:
- **Analysis:** Methodology, logic, completeness, metrics (domain-agnostic)
- **Communication:** Narrative, audience fit, visualization, actionability (domain-agnostic)
- **Domain Knowledge:** Domain-specific techniques, benchmarks, pitfalls (v0.5+)

**Usage:**
```bash
/ds-review path/to/analysis.md
/ds-review --domain search-ranking path/to/analysis.md  # v0.5+
```

**Status:** ✅ Shipped (v0.4.1)

---

### 🔍 SQL Review (`/sql-review`)

Reviews SQL queries for syntax correctness and domain-specific patterns.

**Checks:**
- SQL syntax errors
- Search-specific SQL anti-patterns
- Performance issues
- Alignment with domain best practices

**Usage:**
```bash
/sql-review --domain search path/to/query.sql
```

**Status:** 🚧 Planned for Q2 2026

---

### 📈 Search Metric Analysis (`/metric-analysis`)

Analyzes search experiment metrics and generates insights.

**Workflow:**
1. Load metrics from CSV or database
2. Fetch Search domain context
3. Analyze metrics, detect anomalies
4. Generate analysis document
5. Review analysis via DS Review agent
6. Return analysis + review feedback

**Usage:**
```bash
/metric-analysis experiments/ranking-v2.csv
```

**Status:** 🚧 Planned for Q2 2026

---

## Shared Infrastructure

### Domain Knowledge Skills

**search-domain-knowledge** (v0.5+)
- Search Relevance expertise: Query Understanding, Search Ranking, Search Infrastructure
- Used by: DS review, SQL review, metric analysis agents
- Auto-refreshed weekly (workstream) and monthly (foundational)

**Future domains:**
- `causal-domain-knowledge` - Causal Inference
- `nlp-domain-knowledge` - NLP

### Review Framework

**ds-review-framework**
- Analysis and Communication rubrics
- Scoring models, deduction tables
- Used by: DS review agent

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Shared Skills (Infrastructure Layer)              │
│  ├── ds-review-framework                           │
│  └── search-domain-knowledge                       │
└─────────────────────────────────────────────────────┘
         ↑              ↑              ↑
         │              │              │
    ┌────┴───┐     ┌────┴───┐     ┌────┴────────┐
    │ DS     │     │ SQL    │     │ Metric      │
    │ Review │     │ Review │     │ Analysis    │
    └────────┘     └────────┘     └─────────────┘
                                        │
                                        ├─ calls ─→ DS Review
                                        └─ for quality gate
```

**Key relationships:**
- All agents share `search-domain-knowledge` for consistent domain expertise
- Metric Analysis agent calls DS Review agent as a quality gate for its output
- DS Review is domain-agnostic; domain expertise comes from pluggable skills

---

## Project Structure

```
ds-productivity-agents/
├── shared/skills/          # Reusable infrastructure
├── agents/                 # Agent implementations
│   ├── ds-review/
│   ├── sql-review/
│   └── search-metric-analysis/
├── .claude/commands/       # Command entry points
├── dev/                    # Development artifacts
│   ├── backlog.md
│   ├── sessions/
│   ├── test-results/
│   └── decisions/
└── docs/                   # Design docs, plans
```

---

## Current Status

**Shipped:**
- v0.4.1: DS Analysis Review Agent (2-dimension scoring)

**In Progress:**
- v0.5: Domain Knowledge dimension (3rd review dimension)
  - Search domain knowledge skill
  - Domain expert reviewer subagent
  - 50/25/25 weighted scoring

**Planned (Q2 2026):**
- SQL Review Agent
- Search Metric Analysis Agent

See `dev/backlog.md` for detailed roadmap.

---

## Development

### Session Start Protocol
1. Read `dev/backlog.md` for current priorities
2. Read latest `dev/sessions/*.md` for context
3. Check `dev/decisions/` for architectural decisions

### Session End Protocol
1. Update `dev/backlog.md`
2. Create `dev/sessions/YYYY-MM-DD-description.md`
3. Update `CHANGELOG.md` if anything shipped
4. Create `dev/decisions/ADR-*.md` if design choice was made

---

## Contributing

This is a personal project for learning vibe coding and building DS productivity tools.

## License

MIT
