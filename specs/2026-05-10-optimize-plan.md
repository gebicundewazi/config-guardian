# Config Guardian Optimization — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure config-guardian SKILL.md from monolithic 278-line file into mode-segmented design with shared preamble, flexible retrospective template, and fixed stale references.

**Architecture:** Single-file rewrite of `./SKILL.md`. Three modes (Retrospective default, Audit, Preflight) share a one-time preamble that reads all config files into a cache. All stale `rules/1-skill-auto-triggers.md` references replaced with `rules/rule-trigger-semantic.md`. `ministry_*.md` replaced with `reference_*.md`.

**Tech Stack:** Markdown, bash commands for environment checks.

---

### Task 1: Backup current SKILL.md

**Files:**
- Create: `./SKILL.md.bak.20260510`

- [ ] **Step 1: Create backup**

```bash
cp ./SKILL.md ./SKILL.md.bak.20260510
```

- [ ] **Step 2: Verify backup matches original**

```bash
diff ./SKILL.md ./SKILL.md.bak.20260510
```

Expected: no output (files identical).

---

### Task 2: Write new SKILL.md — frontmatter + Parameters + Shared Preamble

**Files:**
- Modify: `./SKILL.md` (complete rewrite, this task writes the top section)

- [ ] **Step 1: Write frontmatter, parameter parsing, and shared preamble**

Write the entire file with content up through the preamble section:

```markdown
---
name: config-guardian
description: "Config audit + task retrospective + environment preflight. Defaults to retrospective mode. Use `audit` for full config audit, `preflight` for environment checks."
---

# Config Guardian

Three-mode config and environment management for Claude Code. **Default mode: Task Retrospective.**

## Parameters

| Arg | Mode | Description |
|-----|------|-------------|
| *(none)* | **Retrospective** (default) | Task retrospective with flexible template |
| `retrospective` | **Retrospective** | Explicit retrospective |
| `audit` / `--audit` | **Audit** | Full config ecosystem audit (Steps A1-A5) |
| `preflight` | **Preflight** | Environment readiness check (on-demand categories) |
| `--skip-env` | Modifier (audit) | Skip environment checks in audit mode |
| `--auto-fix` | Modifier (audit) | Auto-apply safe fixes after audit |

## Shared Preamble

All modes share a one-time file read. Results are cached and reused across mode steps.

### Read Once

```
~/.claude/CLAUDE.md              → rule index, routing table, red lines
~/.claude/rules/rule-*.md        → trigger maps, priority declarations per rule
~/.claude/projects/*/memory/     → MEMORY.md index + individual memory files
~/.claude/skills/*/SKILL.md      → skill list, frontmatter metadata
```

### Cache Structure

After reading, the following is available to all modes without re-reading files:

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
| `smart_quotes` | Files containing Unicode curly quotes (U+2018/2019/201C/201D) |

### Error Handling

- Missing file → record as stale ref, continue
- Read error → record failure, mode decides if fatal
- Cache lives for duration of single invocation only
```

- [ ] **Step 2: Verify file was written**

```bash
wc -l ./SKILL.md
```

Expected: ~60 lines.

---

### Task 3: Append Mode B — Task Retrospective (Default)

**Files:**
- Modify: `./SKILL.md` (append Mode B section)

- [ ] **Step 1: Append Mode B content**

Append the following to SKILL.md:

```markdown
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
## Task Retrospective — {date}

**Result**: {success / failure / partial} — {one-sentence summary}
**Duration**: {actual duration} / {expected duration}
```

**Extended fields (output only when content exists — never output empty headers or "N/A"):**

| Field | Condition to appear |
|-------|--------------------|
| **Blocker** | A blocking error stopped progress |
| **Surprise** | Unexpected behavior or result was encountered |
| **Decision** | A non-obvious technical choice was made |
| **Lesson** | Reusable experience or pitfall was identified |

**Minimal output example (Light, no issues):**

```
## Task Retrospective — 2026-05-10

**Result**: Success — fixed login page token expiry redirect
**Duration**: 3min / 5min expected
```

**Full output example (Deep, with issues):**

```
## Task Retrospective — 2026-05-10

**Result**: Success — generated 6-panel Nature publication figure
**Duration**: 25min / 15min expected

**Blocker**: svglite 1.2.0 produced garbled Chinese labels under WSL2; downgraded to 1.1.0
**Decision**: Used patchwork over cowplot for multi-panel assembly — patchwork handles ggplot2 3.5+ theme inheritance more reliably
**Lesson**: Validate R package versions in container before upgrading in WSL2
```

### B3. Memory Write Decision — Three-Tier Gate

Only run when depth is Standard or Deep. Light skips entirely.

**Gate 1 — Trigger (must pass at least one):**

| # | Condition |
|---|-----------|
| 1 | Same issue appeared in 2 or more retrospectives |
| 2 | User explicitly corrected your approach |
| 3 | Non-obvious tool/API behavior discovered |
| 4 | Reproducible pitfall with root cause identified |

If no condition met → stop, do not write memory.

**Gate 2 — Auto-Routing (after passing Gate 1):**

| Content type | Target file | Memory type |
|-------------|-------------|-------------|
| User correction / preference | `feedback_{topic}.md` | feedback |
| Tool/API pitfall / version compat | `reference_{topic}.md` | reference |
| Active project state change | `project_{topic}.md` | project |
| User role/knowledge profile | `user_profile.md` | user |

**Gate 3 — Dedup (before write):**

1. Read target file (if exists)
2. Equivalent record already present → skip, do not write
3. Partial/outdated record present → update existing
4. No existing record → create new file

**Output indicator:** If memory was written, append to retrospective report:

```
memory → {target_file}
```

If not written, output nothing extra.
```

- [ ] **Step 2: Verify line count**

```bash
wc -l ./SKILL.md
```

Expected: ~150 lines.

---

### Task 4: Append Mode A — Full Audit

**Files:**
- Modify: `./SKILL.md` (append Mode A section)

- [ ] **Step 1: Append Mode A content**

Append the following to SKILL.md:

```markdown
---

## Mode A: Full Audit

Triggered by `audit` or `--audit`. All file I/O already done in preamble — steps consume `preamble_cache`.

### A1. Rule Conflict Analysis

Consumes: `rule_files`, `red_lines`

- Cross-compare every rule pair for contradictions (e.g. "always do X" vs "never do X")
- Check if any non-red-line rule overrides a red-line semantic
- Flag rules with ambiguous or conflicting priority
- Output: CONFLICTS table (Location, Rule A, Rule B, Severity, Fix)

### A2. Memory Health

Consumes: `memory_index`, `memory_orphans`, `memory_unindexed`

- Flag memory files with mtime > 30 days ago
- Validate frontmatter: `name`, `description`, `type` fields present and match content
- Report orphan entries (in MEMORY.md but file missing)
- Report unindexed files (on disk but not in MEMORY.md)
- Output: STALE REFERENCES entries for memory issues

### A3. Environment Availability

No cache consumption — fresh command execution.

Commands run in parallel:

```bash
Rscript --version &
Rscript -e "for(pkg in c('ggplot2','survival','survminer','pROC','epiR','lme4','svglite')) { if(requireNamespace(pkg, quietly=TRUE)) cat(pkg, ': OK\n') else cat(pkg, ': MISSING\n') }" &
python --version &
python -c "import pandas; print('pandas:', pandas.__version__)" &
python -c "import matplotlib; print('matplotlib:', matplotlib.__version__)" &
python -c "import openpyxl; print('openpyxl:', openpyxl.__version__)" &
python -c "import docx; print('python-docx:', docx.__version__)" &
git --version &
git config user.name &
git config user.email &
wait
```

- Output: ENVIRONMENT ISSUES table (Component, Status, Details)
- Skip entirely if `--skip-env` modifier is set

### A4. Skill Overlap Scan

Consumes: `skill_list`, `trigger_map` (from `rules/rule-trigger-semantic.md`)

- Compare trigger word sets across all skill pairs
- Flag pairs with overlapping trigger words
- Flag skills not used in recent sessions (inferred from memory/audit logs)
- Output: SKILL OVERLAP table (Skills, Overlap, Recommendation)

### A5. Aggregated Report

Consumes: all A1-A4 outputs + `stale_refs`, `smart_quotes` from preamble

Generate structured report:

```markdown
# Config Audit Report — {date}

## CONFLICTS ({count})
| Location | Rule A | Rule B | Severity | Fix |
|----------|--------|--------|----------|-----|

## REDUNDANCIES ({count})
| Content | Location 1 | Location 2 | Recommendation |
|---------|-----------|-----------|----------------|

## STALE REFERENCES ({count})
| Reference | Location | Status | Fix |
|-----------|----------|--------|-----|

## ENVIRONMENT ISSUES ({count})
> Omitted in --skip-env mode.
| Component | Status | Details |
|-----------|--------|---------|

## SKILL OVERLAP ({count})
| Skills | Overlap | Recommendation |
|--------|---------|----------------|

## ENCODING ISSUES ({count})
| File | Issue | Fix |
|------|-------|-----|

## SUMMARY
- Total issues: {n}
- High severity: {n}
- Recommended actions: {n}
```

Report saved to `~/.claude/memory/audit_report_{date}.md` (overwrites previous).
```

- [ ] **Step 2: Verify line count**

```bash
wc -l ./SKILL.md
```

Expected: ~250 lines.

---

### Task 5: Append Mode C + Auto-Fix + Chain Marker + Constraints

**Files:**
- Modify: `./SKILL.md` (append final sections)

- [ ] **Step 1: Append final sections**

Append the following to SKILL.md:

```markdown
---

## Mode C: Environment Preflight

Triggered by `preflight`. On-demand environment readiness check before multi-step operations.

### On-Demand Category Selection

Auto-detected from task context, or explicitly via `--for` flag:

| Task context | Categories run |
|-------------|----------------|
| R visualization / statistics | R + Filesystem + Format support |
| Python data processing | Python + Filesystem |
| Document generation (.docx) | python-docx + Filesystem |
| Literature search | API connectivity + Filesystem |
| Multi-step compound operation | All categories |

Usage: `config-guardian preflight` (auto-detect) or `config-guardian preflight --for R` (explicit).

### C1. Tool Availability

Commands run in parallel:

```bash
Rscript --version &
Rscript -e "library(ggplot2)" &
python --version &
python -c "import pandas" &
git --version &
wait
```

### C2. Format Support

- python-docx: PNG supported, EMF not supported
- R: SVG via svglite, TIFF via tiff()
- Check if target format is available for current task

### C3. API Connectivity

Only run when task context involves literature/search:

```bash
curl -s "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&term=test&retmax=1" | head -1
curl -s "https://api.crossref.org/works?query=test&rows=1" | head -1
```

### C4. File System

- Output directory exists and is writable
- Sufficient disk space (`df -h .`)
- No conflicting files at output location

### Output Format

Checklist with pass/fail per item, suggested fixes for failures, overall go/no-go recommendation. Critical failures → STOP and report.

---

## Auto-Fix Mode

When `--auto-fix` is passed with `audit`, run after A5 completes.

### Safe to Auto-Fix

| Issue | Fix |
|-------|-----|
| Broken `memory: xxx.md` reference in CLAUDE.md | Remove dead reference line |
| Memory file exists but missing from MEMORY.md index | Add entry with inferred type and description |
| MEMORY.md entry references non-existent file | Remove orphan entry |
| Frontmatter `name`/`description` mismatch with content | Update frontmatter to match content |
| `rules/rule-trigger-semantic.md` trigger mapping drift from CLAUDE.md routing table | Sync from CLAUDE.md (CLAUDE.md is source of truth) |
| Stale version numbers / date stamps in rule files | Update to current date |
| CLAUDE.md routing table has skill with no matching trigger in `rules/rule-trigger-semantic.md` | Add trigger entry |
| `rules/rule-trigger-semantic.md` has trigger with no matching routing entry in CLAUDE.md | Add routing entry or remove orphan trigger |
| New memory file on disk not indexed in MEMORY.md | Add entry to MEMORY.md |

### NOT Auto-Fixed (Human Review Required)

| Issue | Reason |
|-------|--------|
| Contradicting rules | Requires judgment on which rule wins |
| Priority conflict (red line vs general) | Requires user decision |
| Skill overlap recommendations | Requires user preference |
| Context budget exceeded | Auto-trimming may break critical rules |
| Environment issues (missing packages/tools) | Requires install decisions |

### Post-Fix Verification

After applying fixes, re-run audit (A1-A5) against same `preamble_cache`. Output:

```
# Auto-Fix Report — {date}

## FIXED ({count})
| Issue | Location | Fix Applied |
|-------|----------|-------------|

## REMAINING ({count}) — needs human review
| Issue | Location | Severity | Recommendation |
|-------|----------|----------|----------------|
```

---

## Chain Marker

On completion, output `[[SYS_SKILL_FINISH]]\ntype: end`. Report final results to user.

## Constraints

- DO NOT apply any changes without user approval (except in `--auto-fix` mode, and only safe changes)
- Each finding must include: location, issue description, proposed fix
- Environment checks use actual command execution, not assumptions
- If a check command fails (e.g., R not installed), report as issue, don't error out
- Report saved to `~/.claude/memory/audit_report_{date}.md` (audit mode only)
```

- [ ] **Step 2: Verify final file**

```bash
wc -l ./SKILL.md
```

Expected: ~350 lines.

---

### Task 6: Verify against spec

**Files:**
- Read: `./SKILL.md`
- Read: `./specs/2026-05-10-optimize-design.md`

- [ ] **Step 1: Check all stale references are fixed**

```bash
grep -n "1-skill-auto-triggers\|ministry_\*" ./SKILL.md
```

Expected: no output (no stale references remain).

- [ ] **Step 2: Check all new references use correct naming**

```bash
grep -n "rule-trigger-semantic\|reference_\*\.md\|feedback_\*\.md" ./SKILL.md
```

Expected: matches found confirming correct references are in place.

- [ ] **Step 3: Verify mode structure completeness**

```bash
grep "^## Mode" ./SKILL.md
```

Expected output:
```
## Mode B: Task Retrospective (Default)
## Mode A: Full Audit
## Mode C: Environment Preflight
```

- [ ] **Step 4: Verify no placeholders or TODOs**

```bash
grep -in "TBD\|TODO\|FIXME\|PLACEHOLDER" ./SKILL.md
```

Expected: no output.

- [ ] **Step 5: Verify frontmatter is valid**

```bash
head -5 ./SKILL.md
```

Expected: valid YAML frontmatter with `name` and `description` fields.

- [ ] **Step 6: Commit**

```bash
git add ./SKILL.md ./specs/2026-05-10-optimize-design.md ./specs/2026-05-10-optimize-plan.md
git commit -m "refactor(config-guardian): restructure into 3-mode design with shared preamble

- Default mode: Task Retrospective with flexible template + 3-tier memory gate
- Audit mode: 5-step audit with I/O in shared preamble, parallel env checks
- Preflight mode: on-demand category selection, parallel command execution
- Fix all stale references: rules/1-skill-auto-triggers.md → rules/rule-trigger-semantic.md
- Fix stale references: ministry_*.md → reference_*.md"
```
```

- [ ] **Step 7: Verify commit**

```bash
git log --oneline -1 && git status
```
