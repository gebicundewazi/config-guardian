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

## Status Mapping

| Value | Criteria |
|-------|----------|
| OK | Metric within acceptable range |
| WARNING | Metric approaching threshold |
| CRITICAL | Metric exceeds threshold |
| UNKNOWN | Metric unavailable or invalid |

## Output Format

| Metric | Value | Status | Recommendation |
|--------|-------|--------|----------------|

## Integration

- Report section: PERFORMANCE METRICS
- Output: raw issue-row compatible lines (one per performance issue)
- Consumed by: main-dispatcher -> report-section.md template

## Error Handling

- Missing metrics → skip, continue
- Invalid data → record as Low
