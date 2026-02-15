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
