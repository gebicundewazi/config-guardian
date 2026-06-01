# Config Guardian

> Claude Code 配置生态审计与任务复盘工具
>
> Config ecosystem audit and task retrospective for Claude Code

[English](#english) | [中文](#中文)

---

## English

### Introduction

Config Guardian is a modular Claude Code skill for managing your configuration ecosystem. It helps you:

- **Audit config health** — check consistency across CLAUDE.md, MEMORY.md, Skills, and Rules
- **Retrospect on tasks** — structured review of execution: goals, decisions, outputs, lessons
- **Auto-fix issues** — safely resolve common configuration problems with one command

### Two Modes

| Mode | Command | Purpose |
|------|---------|---------|
| **Retrospective** (default) | `config-guardian` | Task retrospective: goals → decisions → outputs → lessons |
| **Audit** | `config-guardian audit` | Full configuration ecosystem health check |

#### Audit Options

```bash
config-guardian audit            # Full audit (env checks + LLM semantic layers)
config-guardian audit --quick    # Quick audit (skip env checks and LLM layers)
config-guardian audit --auto-fix # Audit then auto-fix safe issues
config-guardian audit --skip-env # Skip environment checks
```

### Audit Modules

| Module | What It Checks |
|--------|---------------|
| `rule-conflict` | Rule conflict analysis |
| `memory-health` | MEMORY.md health (orphaned indexes, unindexed files) |
| `env-check` | Tool environment availability |
| `skill-overlap` | Skill trigger word overlap |
| `settings-integrity` | settings.json integrity |
| `project-init` | Project initialization health |
| `permission-audit` | Permission configuration audit |
| `security-scan` | Security scan (hardcoded keys, sensitive files) |
| `dependency-check` | Dependency verification |
| `performance-metrics` | Performance metrics |
| `auto-fix` | Auto-fix logic |

### Architecture

```
config-guardian/
├── SKILL.md              # Main dispatcher (~100 lines)
├── modes/
│   ├── retrospective.md  # Retrospective mode
│   └── audit.md          # Audit mode
├── modules/              # 11 independent check modules
├── templates/            # Report templates
├── data/                 # Format definitions
└── scripts/              # Helper scripts
```

### Installation

```bash
git clone https://github.com/gebicundewazi/config-guardian.git ~/.claude/skills/config-guardian
```

### Example Output

**Retrospective mode:**

```markdown
## Task Retrospective
- Goal: Fix 3 configuration issues
- Decision: Prioritize security fixes, defer performance optimization
- Output: settings.json updated, 2 rule files corrected
- Lesson: Rule naming needs a more unified convention
```

**Audit mode:**

```
✅ PASS  rule-conflict    — No rule conflicts
⚠️ WARN  memory-health    — 3 orphaned indexes
✅ PASS  skill-overlap    — No trigger word overlap
🔴 FAIL  security-scan    — 1 hardcoded token found
🔧 FIXED auto-fix         — Safe issues auto-fixed
```

### Contributing

To add a new check module:

1. Create a module file in `modules/`
2. Follow the interface spec (see `rule-conflict.md`)
3. Add it to parallel/sequential groups in `SKILL.md`
4. Update `data/format-definitions.toml` if needed

---

## 中文

### 简介

Config Guardian 是一个模块化的 Claude Code 配置管理 Skill，帮助你：

- **审计配置健康**：检查 CLAUDE.md、MEMORY.md、Skills、Rules 之间的一致性
- **任务复盘**：结构化回顾任务执行过程，记录决策和产出
- **自动修复**：安全的配置问题可一键修复

### 两种模式

| 模式 | 命令 | 用途 |
|------|------|------|
| **任务复盘**（默认） | `config-guardian` | 复盘任务执行：目标→决策→产出→教训 |
| **配置审计** | `config-guardian audit` | 全量检查配置生态系统健康状态 |

#### 审计模式参数

```bash
config-guardian audit            # 全量审计（含环境检查 + LLM 语义层）
config-guardian audit --quick    # 快速审计（跳过环境检查和 LLM 层）
config-guardian audit --auto-fix # 审计后自动修复安全项
config-guardian audit --skip-env # 跳过环境检查
```

### 审计模块

| 模块 | 检查内容 |
|------|---------|
| `rule-conflict` | 规则冲突分析 |
| `memory-health` | MEMORY.md 健康检查（孤立索引、未索引文件） |
| `env-check` | 工具环境可用性 |
| `skill-overlap` | Skill 触发词重叠扫描 |
| `settings-integrity` | settings.json 完整性 |
| `project-init` | 项目初始化健康度 |
| `permission-audit` | 权限配置审计 |
| `security-scan` | 安全扫描（硬编码密钥、敏感文件） |
| `dependency-check` | 依赖检查 |
| `performance-metrics` | 性能指标 |
| `auto-fix` | 自动修复逻辑 |

### 架构

```
config-guardian/
├── SKILL.md              # 主调度器（~100行）
├── modes/
│   ├── retrospective.md  # 复盘模式
│   └── audit.md          # 审计模式
├── modules/              # 11个独立检查模块
├── templates/            # 报告模板
├── data/                 # 格式定义
└── scripts/              # 辅助脚本
```

### 安装

```bash
git clone https://github.com/gebicundewazi/config-guardian.git ~/.claude/skills/config-guardian
```

### 典型输出

**复盘模式：**

```markdown
## 任务复盘
- 目标：修复 3 个配置问题
- 决策：优先修复安全项，推迟性能优化
- 产出：settings.json 更新、2 个 rule 文件修正
- 教训：规则命名需要更统一的约定
```

**审计模式：**

```
✅ PASS  rule-conflict    — 无规则冲突
⚠️ WARN  memory-health    — 3 个孤立索引
✅ PASS  skill-overlap    — 触发词无重叠
🔴 FAIL  security-scan    — 发现 1 处硬编码 token
🔧 FIXED auto-fix         — 已自动修复安全项
```

### 开发

添加新检查模块：

1. 在 `modules/` 目录创建模块文件
2. 遵循接口规范（参考 `rule-conflict.md`）
3. 在 `SKILL.md` 中添加到并行/串行执行组
4. 按需更新 `data/format-definitions.toml`

---

## License / 许可

MIT
