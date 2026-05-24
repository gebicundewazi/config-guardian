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

| Issue | Status | Recommendation |
|-------|--------|----------------|

## Error Handling

- No CLAUDE.md → report as INFO, suggest initialization
- Invalid structure → report as WARNING with specific fixes
