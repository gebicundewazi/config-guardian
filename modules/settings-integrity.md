---
module: settings-integrity
description: "Settings & config integrity check"
inputs: []
outputs: [settings_issues]
parallel: false
---

## Execution Logic

### Settings File Check

Read fresh from `~/.claude/settings.json` and `~/.claude/settings.local.json`:

- Structure integrity of both settings files
- Hook configs referencing archived/deleted skills (check against `skill_list` and archived list)
- Permission rules with dead references (allow for non-existent tools)
- Env var conflicts between settings.json and settings.local.json

## Output Format

| Issue | Location | Severity | Fix |
|-------|----------|----------|-----|

## Severity Mapping

| Level | Criteria |
|-------|----------|
| Critical | JSON parse error preventing settings load |
| High | Hook referencing deleted/archived skill, dead permission reference |
| Medium | Env var conflict between settings files |
| Low | Redundant or duplicate entries |

## Integration

- Report section: SETTINGS ISSUES
- Output: raw issue-row compatible lines (one per issue)
- Consumed by: main-dispatcher -> report-section.md template

## Error Handling

- Missing settings files → report as issue, continue
- JSON parse error → record as Critical issue
