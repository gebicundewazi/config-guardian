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

## Output Format

| File | Issue | Severity | Fix |
|------|-------|----------|-----|

## Error Handling

- File read errors -> skip, continue
- False positives -> mark as INFO, not Critical
