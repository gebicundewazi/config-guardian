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
| `--incremental` | Modifier (audit) | Only scan changed files |

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

### Quick Mode (`--quick`)

| Step | Normal | `--quick` |
|------|--------|-----------|
| String-level checks | Always | Always |
| LLM semantic layers | Run | Skip |
| Environment checks | Run | Skip (or `--skip-env`) |
| Report format | Full | Compact single-table |

Report saved to `~/.claude/memory/audit_report_{date}_quick.md`

---

## Chain Marker

On completion, output `[[SYS_SKILL_FINISH]]\ntype: end`.

## Constraints

- DO NOT apply Tier 2/Tier 3 changes without user approval
- Each finding must include: location, severity, issue description, proposed fix
- Environment checks use actual command execution, not assumptions
- Report saved to `~/.claude/memory/audit_report_{date}.md`
- Preamble cache valid for session duration
