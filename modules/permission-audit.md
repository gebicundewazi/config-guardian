---
module: permission-audit
description: "Permission allowlist audit"
inputs: []
outputs: [permission_issues]
parallel: false
---

## Execution Logic

### Permission Check

Read recent transcript logs (if available):

- Scan recent transcripts for frequently denied Bash/MCP tool calls
- Check current allowlist coverage in project `.claude/settings.json`
- Output suggested allowlist entries for user review

## Output Format

| Tool | DenyCount | SuggestedRule | Coverage |
|------|-----------|---------------|----------|

## Severity Mapping

| Level | Criteria |
|-------|----------|
| Critical | Sensitive tool (e.g., `rm`, `git push`) denied with no allowlist rule |
| High | Frequently denied tool (>5 denials) with no allowlist coverage |
| Medium | Tool denied occasionally (1-4 denials), allowlist may need expansion |
| Low | Tool denied once, likely one-off usage |

## Integration

- Report section: PERMISSION_ALLOWLIST
- Output: raw issue-row compatible lines (one per permission issue)
- Consumed by: main-dispatcher -> report-section.md template

## Error Handling

- No transcript logs → report as INFO, skip audit
- Missing settings.json → report as WARNING
