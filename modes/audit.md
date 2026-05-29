# Mode A: Full Audit

Triggered by `audit` or `--audit`.

## Module Execution Order

**Parallel Group 1** (independent checks):
- `modules/rule-conflict.md`
- `modules/memory-health.md`
- `modules/env-check.md` (skip if `--skip-env`)
- `modules/skill-overlap.md`
- `modules/security-scan.md`
- `modules/dependency-check.md`
- `modules/performance-metrics.md`

**Sequential Group 2** (depends on Group 1):
- `modules/settings-integrity.md`
- `modules/project-init.md`
- `modules/permission-audit.md`

**Final Step**:
- Aggregate results into report
- Apply auto-fix if `--auto-fix`
- Save report to `~/.claude/memory/audit_report_{date}.md`
- Add to MEMORY.md index

## Aggregated Report Template

```markdown
# Config Audit Report — {date}

## CONFLICTS ({count})
| Location | Rule A | Rule B | Severity | Fix |
|----------|--------|--------|----------|-----|

## STALE REFERENCES ({count})
| Reference | Location | Status | Fix |
|-----------|----------|--------|-----|

## ENVIRONMENT ISSUES ({count})
| Component | Status | Details |
|-----------|--------|---------|

## SKILL OVERLAP ({count})
| Skills | Overlap | Recommendation |
|--------|---------|----------------|

## SETTINGS & CONFIG ({count})
| Issue | Location | Severity | Fix |
|-------|----------|----------|-----|

## PROJECT INIT ({count})
| Issue | Status | Recommendation |
|-------|--------|----------------|

## PERMISSION ALLOWLIST ({count})
| Tool | DenyCount | SuggestedRule | Coverage |
|------|-----------|---------------|----------|

## SECURITY SCAN ({count})
| File | Issue | Severity | Fix |
|------|-------|----------|-----|

## DEPENDENCY CHECK ({count})
| Skill | Issue | Severity | Fix |
|-------|-------|----------|-----|

## PERFORMANCE METRICS ({count})
| Metric | Value | Status | Recommendation |
|--------|-------|--------|----------------|

## ENCODING ISSUES ({count})
| File | Issue | Fix |
|------|-------|-----|

## SUMMARY
- Total issues: {n}
- High severity: {n}
- Recommended actions: {n}
```

## Auto-Fix Coordination

🔴 **CHECKPOINT — Tier 2/3 fixes require user confirmation before proceeding.**

When `--auto-fix` is set, after audit completes:

1. Pass aggregated issues to `modules/auto-fix.md`
2. 🛑 **STOP — Tier 1 fixes applied automatically. Pause here for Tier 2/3.**
3. Tier 2 fixes listed for user confirmation
4. Tier 3 issues reported only
5. Re-run audit (A1-A8) to verify fixes
6. Output fix report:

```markdown
# Auto-Fix Report — {date}

## AUTO-FIXED ({count})
| Issue | Location | Fix Applied |
|-------|----------|-------------|

## PENDING CONFIRMATION ({count})
| Issue | Location | Recommendation |
|-------|----------|----------------|

## NEEDS REVIEW ({count})
| Issue | Location | Severity | Recommendation |
|-------|----------|----------|----------------|
```

## Quick Mode (`--quick`)

| Step | Normal | `--quick` |
|------|--------|-----------|
| A1 | String + LLM layers | String layer only |
| A2 | Local + cross-session full scan | Local memory orphan check only |
| A3 | Full env: R + Python + Git | Git version only |
| A4 | String + LLM + Frequency layers | String layer only |
| A5 | Full aggregated report | Compact single-table report |
| A6 | Settings/config integrity check | Structure check only |
| A7 | Project init + template diff | CLAUDE.md existence check |
| A8 | Transcript scan + allowlist audit | Allowlist existence check |
| A9 | Security scan | Skip |
| A10 | Dependency check | Skip |
| A11 | Performance metrics | Skip |

Report saved to `~/.claude/memory/audit_report_{date}_quick.md`
