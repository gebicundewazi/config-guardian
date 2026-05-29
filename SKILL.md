---
name: config-guardian
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

## Mode Details

| Mode | File |
|------|------|
| `retrospective` (default) | [modes/retrospective.md](modes/retrospective.md) |
| `audit` | [modes/audit.md](modes/audit.md) |

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

## Anti-Patterns (不要做什么)

| # | 不要 | 替代做法 |
|---|------|---------|
| 1 | 自动应用 Tier 2/Tier 3 修复 | 列出修复建议，等用户确认 |
| 2 | 运行 `git config` 修改命令 | 永远不修改 git config |
| 3 | 运行 `rm -rf`、`reset --hard`、`force push` | 报告为安全问题，不执行 |
| 4 | 跳过 hooks（`--no-verify`） | 报告 hook 失败原因 |
| 5 | 假设环境状态 | 用实际命令验证（`which R`、`pip show`） |
| 6 | 修改 `~/.claude/` 之外的文件 | 只读取，不写入外部路径 |
| 7 | 在审计过程中创建新 skill | 只报告重叠/冲突，不自动创建 |
| 8 | 自动修复安全扫描发现的问题 | 只报告，修复由用户决定 |
