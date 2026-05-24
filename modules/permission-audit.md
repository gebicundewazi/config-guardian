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

## Error Handling

- No transcript logs → report as INFO, skip audit
- Missing settings.json → report as WARNING
