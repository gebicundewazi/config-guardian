---
module: encoding-check
description: "Encoding issue detection"
inputs: []
outputs: [encoding_issues]
parallel: true
---

## Execution Logic

### Encoding Scan

Scan configuration files for encoding issues:

- Smart quotes (curly quotes) in config files
- BOM (Byte Order Mark) characters
- Invisible control characters
- Non-UTF-8 encoding

## Output Format

| File | Issue | Severity | Fix |
|------|-------|----------|-----|

## Severity Mapping

| Level | Criteria |
|-------|----------|
| High | BOM or control characters breaking parsing |
| Medium | Smart quotes in config files |
| Low | Minor encoding inconsistencies |

## Integration

- Report section: ENCODING ISSUES
- Output: raw issue-row compatible lines (one per encoding issue)
- Consumed by: main-dispatcher -> report-section.md template

## Error Handling

- File read errors -> record as Medium, continue
- Binary files -> skip, continue
