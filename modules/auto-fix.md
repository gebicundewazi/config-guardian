---
module: auto-fix
description: "Auto-fix logic for identified issues"
inputs: [all_issues]
outputs: [fix_results]
parallel: false
---

## Execution Logic

### Three-Tier Fix System

#### Tier 1 — Full Auto (applied immediately, no confirmation)

| Issue | Fix |
|-------|-----|
| Broken `memory: xxx.md` reference in CLAUDE.md | Remove dead reference line |
| Memory file exists but missing from MEMORY.md index | Add entry with inferred type and description |
| MEMORY.md entry references non-existent file | Remove orphan entry |
| Frontmatter `name`/`description` mismatch with content | Update frontmatter to match content |
| Stale version numbers / date stamps in rule files | Update to current date |
| New memory file on disk not indexed in MEMORY.md | Add entry to MEMORY.md |
| Smart quotes (curly quotes) in config files | Replace with ASCII straight quotes |
| BOM / invisible control characters | Remove |
| Stale audit reports (>30 days) in `~/.claude/memory/` | Delete |
| Dead chain routing references (skill deleted, routing remains) | Remove dead routing entry |
| Inconsistent date formats | Standardize to YYYY-MM-DD |
| Trigger mapping drift (CLAUDE.md vs triggers.toml mismatch) | Sync from triggers.toml |
| Expired cache files (>30 days) | Delete |
| Incomplete frontmatter | Add missing fields |
| Non-standard file names | Rename to follow file-naming.md |

#### Tier 2 — Recommend + Confirm (listed, user approves once)

| Issue | Fix |
|-------|-----|
| Medium-severity semantic contradictions in rules | Present resolution options, apply selected |
| Dead rules (fully covered by later rules) | List candidates, delete confirmed ones |
| Skills with 0 invocations + no recent triggers | List candidates, disable confirmed ones |

#### Tier 3 — Report Only (human judgment required)

| Issue | Reason |
|-------|--------|
| High/Critical rule conflicts + priority inversions | Requires judgment on which rule wins |
| Skill functional duplication (merge vs keep) | Requires user preference |
| Missing packages/tools (environment) | Requires install decisions |

## Output Format

### Auto-Fixed ({count})
| Issue | Location | Fix Applied |
|-------|----------|-------------|

### Pending Confirmation ({count})
| Issue | Location | Recommendation |
|-------|----------|----------------|

### Needs Review ({count})
| Issue | Location | Severity | Recommendation |
|-------|----------|----------|----------------|

## Error Handling

- Fix fails → record as issue, continue
- User rejects → move to Needs Review
