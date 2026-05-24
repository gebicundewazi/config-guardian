---
module: memory-health
description: "Memory health check"
inputs: [memory_index, memory_orphans, memory_unindexed]
outputs: [memory_issues]
parallel: true
---

## Execution Logic

### Local Memory Check

- Flag memory files with mtime > 30 days ago
- Validate frontmatter: `name`, `description`, `type` fields present and match content
- Report orphan entries (in MEMORY.md but file missing)
- Report unindexed files (on disk but not in MEMORY.md)

## Output Format

| Reference | Location | Status | Fix |
|-----------|----------|--------|-----|

## Status Mapping

| Value | Criteria |
|-------|----------|
| stale | mtime > 30 days ago |
| orphan | Entry in MEMORY.md but file missing on disk |
| unindexed | File on disk but not in MEMORY.md |
| invalid-frontmatter | Missing or mismatched `name`, `description`, or `type` fields |

## Integration

- Report section: STALE REFERENCES
- Output: raw issue-row compatible lines (one per memory issue)
- Consumed by: main-dispatcher -> report-section.md template

## Error Handling

- Missing MEMORY.md → report as issue, continue
- Invalid frontmatter → record as issue, suggest fix
