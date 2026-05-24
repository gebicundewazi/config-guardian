---
module: skill-overlap
description: "Skill overlap scan"
inputs: [skill_list, trigger_map]
outputs: [skill_overlaps]
parallel: true
---

## Execution Logic

### Layer 1 — String-level (always run)

- Compare trigger word sets across all skill pairs
- Flag pairs with overlapping trigger words
- Output: string-level overlap table

### Layer 2 — LLM semantic (skip if --quick)

Load skill descriptions + trigger words into context, then analyze:

1. **Functional duplication** — Two skills do the same thing even with different trigger words
2. **Trigger ambiguity** — Same phrase could match multiple skills with no clear disambiguation
3. **Chain breakage** — Skill A's chain routing points to Skill B, but B is deleted or renamed

## Output Format

| Skills | Overlap | Recommendation |
|--------|---------|----------------|

## Error Handling

- Missing skill files → skip, continue
- Invalid trigger map → record as issue
