---
module: project-init
description: "Project initialization health check"
inputs: []
outputs: [project_issues]
parallel: false
---

## Execution Logic

### Project CLAUDE.md Check

Read fresh from `$PWD/CLAUDE.md`:

- Whether current project has a CLAUDE.md
- If yes: verify structure — rule index, routing table, red lines sections present
- Compare against latest template, flag missing recommended sections
- If no: report "project not initialized", suggest running init via config-guardian

## Output Format

| Severity | Location | Issue | Fix |
|----------|----------|-------|-----|

## Severity Mapping

| Level | Criteria |
|-------|----------|
| Critical | CLAUDE.md exists but is unreadable or corrupt |
| High | Missing mandatory sections (rule index, red lines) |
| Medium | Missing recommended sections (routing table, workflows) |
| Low | Minor template drift or outdated version string |

## Integration

- Report section: PROJECT_INIT
- Output: raw issue-row compatible lines (one per issue)
- Consumed by: main-dispatcher -> report-section.md template

## Error Handling

- No CLAUDE.md → report as INFO, suggest initialization
- Invalid structure → report as WARNING with specific fixes
