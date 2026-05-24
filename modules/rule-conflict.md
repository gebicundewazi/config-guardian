---
module: rule-conflict
description: "Rule conflict analysis"
inputs: [rule_files, red_lines]
outputs: [conflicts]
parallel: true
---

## Execution Logic

### Layer 1 — String-level (always run)

- Cross-compare every rule pair for literal contradictions
- Check if any non-red-line rule overrides a red-line semantic
- Flag rules with ambiguous or conflicting priority
- Output: CONFLICTS table (Location, Rule A, Rule B, Severity, Fix)

### Layer 2 — LLM semantic (skip if --quick)

Load `rule_files` content into context, then analyze:

1. **Semantic contradictions** — Two rules substantively negate each other even with different wording
2. **Priority inversion** — A lower-tier rule effectively overrides a higher-tier one through side effects
3. **Dead rules** — A rule is fully covered by a later rule and can never trigger independently

## Output Format

| Location | Rule A | Rule B | Severity | Fix |
|----------|--------|--------|----------|-----|

## Error Handling

- Missing rule files → record as stale ref, continue
- Parse errors → record failure, add to issues list
