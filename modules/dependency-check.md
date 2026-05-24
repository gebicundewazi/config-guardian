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

## Output Format

| Skill | Issue | Severity | Fix |
|-------|-------|----------|-----|

## Error Handling

- Missing skills -> record as issue
- Invalid routing -> record as WARNING
