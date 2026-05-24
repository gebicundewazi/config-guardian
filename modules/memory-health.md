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

| File | Issue | Status | Fix |
|------|-------|--------|-----|

## Error Handling

- Missing MEMORY.md → report as issue, continue
- Invalid frontmatter → record as issue, suggest fix
