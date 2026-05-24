---
name: config-guardian-v2
description: "Config audit + task retrospective. Modular architecture. Defaults to retrospective mode. Use `audit` for full config audit."
---

# Config Guardian v2

Modular config and environment management for Claude Code. **Default mode: Task Retrospective.**

## Parameters

| Arg | Mode | Description |
|-----|------|-------------|
| *(none)* | **Retrospective** (default) | Task retrospective with flexible template |
| `retrospective` | **Retrospective** | Explicit retrospective |
| `audit` / `--audit` | **Audit** | Full config ecosystem audit |
| `--skip-env` | Modifier (audit) | Skip environment checks |
| `--auto-fix` | Modifier (audit) | Auto-apply safe fixes after audit |
| `--quick` | Modifier (audit) | Skip LLM semantic layers and env checks |

## Shared Preamble

All modes share a one-time file read. Results cached for session duration.

### Read Once

```
~/.claude/CLAUDE.md              -> rule index, routing table, red lines
~/.claude/rules/rule-*.md        -> trigger maps, priority declarations per rule
~/.claude/projects/*/memory/     -> MEMORY.md index + individual memory files
~/.claude/skills/*/SKILL.md      -> skill list, frontmatter metadata
```

### Cache Structure

| Cache key | Content |
|-----------|---------|
| `red_lines` | List of red-line rules extracted from CLAUDE.md |
| `rule_files` | `{filename: {content, mtime}}` for each rule file |
| `trigger_map` | `{skill_name: [trigger_words]}` from rule-trigger-semantic.md |
| `memory_index` | `{file.md: {type, description, mtime}}` from MEMORY.md |
| `memory_orphans` | Entries in MEMORY.md referencing non-existent files |
| `memory_unindexed` | Files on disk not listed in MEMORY.md |
| `skill_list` | `{skill_name: {description, mtime}}` from skills/*/SKILL.md |
| `stale_refs` | Lines referencing files/rules that don't exist |
| `smart_quotes` | Files containing Unicode curly quotes |

### Cache Invalidation

- Check mtime for each file on access
- Refresh if mtime changed since last read
- Support incremental updates for changed files only

### Error Handling

- Missing file → record as stale ref, continue
- Read error → record failure, mode decides if fatal
- Cache lives for session duration. Repeated audits reuse cached reads.

---

## Mode B: Task Retrospective (Default)

Triggered when no args, or `retrospective` is passed explicitly.

### B1. Depth Classification

| Level | Criteria | Behavior |
|-------|----------|----------|
| **Light** | 1-2 steps, no errors | Core fields only; skip memory write |
| **Standard** | 3-5 steps | Core + relevant extended fields; run gate check |
| **Deep** | 5+ steps, debugging, unexpected errors | Full output; run gate check |

### B2. Flexible Template

**Core fields (always output):**

```
## Task Retrospective -- {date}

**Result**: {success / failure / partial} -- {one-sentence summary}
**Duration**: {actual duration} / {expected duration}
```

**Extended fields (output only when content exists):**

| Field | Condition to appear |
|-------|--------------------|
| **Blocker** | A blocking error stopped progress |
| **Surprise** | Unexpected behavior or result was encountered |
| **Decision** | A non-obvious technical choice was made |
| **Lesson** | Reusable experience or pitfall was identified |

**Minimal output example (Light, no issues):**

```
## Task Retrospective — 2026-05-24

**Result**: Success — fixed login page token expiry redirect
**Duration**: 3min / 5min expected
```

**Full output example (Deep, with issues):**

```
## Task Retrospective — 2026-05-24

**Result**: Success — generated 6-panel Nature publication figure
**Duration**: 25min / 15min expected

**Blocker**: svglite 1.2.0 produced garbled Chinese labels under WSL2; downgraded to 1.1.0
**Decision**: Used patchwork over cowplot for multi-panel assembly — patchwork handles ggplot2 3.5+ theme inheritance more reliably
**Lesson**: Validate R package versions in container before upgrading in WSL2
```

### B3. Memory Write Decision -- Three-Tier Gate

Only run when depth is Standard or Deep.

**Gate 1 -- Trigger (must pass at least one):**

| # | Condition |
|---|-----------|
| 1 | Same issue appeared in 2+ retrospectives |
| 2 | User explicitly corrected your approach |
| 3 | Non-obvious tool/API behavior discovered |
| 4 | Reproducible pitfall with root cause identified |

**Gate 2 -- Auto-Routing:**

| Content type | Target file |
|-------------|-------------|
| User correction / preference | `feedback_{topic}.md` |
| Tool/API pitfall / version compat | `reference_{topic}.md` |
| Active project state change | `project_{topic}.md` |
| User role/knowledge profile | `user_profile.md` |

**Gate 3 -- Dedup:**

1. Read target file (if exists)
2. Equivalent -> skip
3. Partial/outdated -> update
4. No record -> create

---

## Mode A: Full Audit

Triggered by `audit` or `--audit`.

### Module Execution Order

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

### Aggregated Report Template

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

### Auto-Fix Coordination

When `--auto-fix` is set, after audit completes:

1. Pass aggregated issues to `modules/auto-fix.md`
2. Tier 1 fixes applied automatically
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

### Quick Mode (`--quick`)

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

---

## Chain Marker

On completion, output `[[SYS_SKILL_FINISH]]\ntype: end`.

## Constraints

- DO NOT apply Tier 2/Tier 3 changes without user approval. Tier 1 auto-fix is always safe.
- Each finding must include: location, severity, issue description, proposed fix
- Environment checks use actual command execution, not assumptions
- If a check command fails (e.g., R not installed), report as issue, don't error out
- Report saved to `~/.claude/memory/audit_report_{date}.md`
- `--quick` saves to `~/.claude/memory/audit_report_{date}_quick.md`
- Preamble cache valid for session duration. For repeated audits within same session, reuse cached reads.
