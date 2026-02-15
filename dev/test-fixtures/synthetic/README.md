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
