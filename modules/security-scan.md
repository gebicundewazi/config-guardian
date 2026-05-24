---
module: security-scan
description: "Security scan for sensitive information"
inputs: []
outputs: [security_issues]
parallel: true
---

## Execution Logic

### Security Scan

Scan configuration files for sensitive information:

- API keys, tokens, secrets in plain text
- Hardcoded credentials
- Private keys or certificates
- Environment variables with sensitive values

## Severity Mapping

| Level | Criteria |
|-------|----------|
| Critical | Plaintext API key or token in tracked file |
| High | Hardcoded credential or private key |
| Medium | Environment variable with sensitive name |
| Low | False positive pattern or informational |

## Output Format

| File | Issue | Severity | Fix |
|------|-------|----------|-----|

## Integration

- Report section: SECURITY SCAN
- Output: raw issue-row compatible lines (one per security issue)
- Consumed by: main-dispatcher -> report-section.md template

## Error Handling

- File read errors → record as Medium, continue
- False positives → mark as Low, not Critical
