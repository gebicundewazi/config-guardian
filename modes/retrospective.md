# Mode B: Task Retrospective (Default)

Triggered when no args, or `retrospective` is passed explicitly.

## B1. Depth Classification

| Level | Criteria | Behavior |
|-------|----------|----------|
| **Light** | 1-2 steps, no errors | Core fields only; skip memory write |
| **Standard** | 3-5 steps | Core + relevant extended fields; run gate check |
| **Deep** | 5+ steps, debugging, unexpected errors | Full output; run gate check |

## B2. Flexible Template

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

## B3. Memory Write Decision — Three-Tier Gate

Only run when depth is Standard or Deep.

🔴 **CHECKPOINT — Before writing to memory, verify all 3 gates pass.**

**Gate 1 — Trigger (must pass at least one):**

| # | Condition |
|---|-----------|
| 1 | Same issue appeared in 2+ retrospectives |
| 2 | User explicitly corrected your approach |
| 3 | Non-obvious tool/API behavior discovered |
| 4 | Reproducible pitfall with root cause identified |

**Gate 2 — Auto-Routing:**

| Content type | Target file |
|-------------|-------------|
| User correction / preference | `feedback_{topic}.md` |
| Tool/API pitfall / version compat | `reference_{topic}.md` |
| Active project state change | `project_{topic}.md` |
| User role/knowledge profile | `user_profile.md` |

**Gate 3 — Dedup:**

1. Read target file (if exists)
2. Equivalent -> skip
3. Partial/outdated -> update
4. No record -> create
