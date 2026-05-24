---
module: performance-metrics
description: "Performance metrics check"
inputs: []
outputs: [performance_issues]
parallel: true
---

## Execution Logic

### Performance Check

Collect performance metrics:

- Cache hit rate from preamble
- Execution time per module
- File read counts
- Memory usage estimates

## Output Format

| Metric | Value | Status | Recommendation |
|--------|-------|--------|----------------|

## Error Handling

- Missing metrics -> skip, continue
- Invalid data -> record as INFO
