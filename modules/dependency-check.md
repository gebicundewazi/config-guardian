---
module: dependency-check
description: "Skill dependency integrity check"
inputs: [skill_list]
outputs: [dependency_issues]
parallel: true
---

## Execution Logic

### Dependency Check

Verify skill dependencies:

- Check if referenced skills exist
- Verify chain routing points to valid skills
- Check for circular dependencies
- Validate skill frontmatter completeness

## Severity Mapping

| Level | Criteria |
|-------|----------|
| Critical | Core skill missing, breaks chain |
| High | Circular dependency detected |
| Medium | Incomplete frontmatter (missing required fields) |
| Low | Optional skill missing or unused |

## Output Format

| Skill | Issue | Severity | Fix |
|-------|-------|----------|-----|

## Integration

- Report section: DEPENDENCY CHECK
- Output: raw issue-row compatible lines (one per dependency issue)
- Consumed by: main-dispatcher -> report-section.md template

## Error Handling

- Missing skills → record as Critical
- Invalid routing → record as High
