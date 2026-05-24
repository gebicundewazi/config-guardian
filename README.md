# Config Guardian v2

Modular config and environment management for Claude Code.

## Architecture

- **Main Dispatcher**: SKILL.md (~100 lines)
- **Modules**: Independent check modules in modules/
- **Templates**: Reusable report templates in templates/
- **Data**: Format definitions in data/

## Usage

- **Retrospective**: `config-guardian` (default)
- **Full Audit**: `config-guardian audit`
- **Quick Audit**: `config-guardian audit --quick`
- **Auto-Fix**: `config-guardian audit --auto-fix`

## Modules

| Module | Description |
|--------|-------------|
| rule-conflict | Rule conflict analysis |
| memory-health | Memory health check |
| env-check | Environment availability |
| skill-overlap | Skill overlap scan |
| settings-integrity | Settings integrity |
| project-init | Project initialization health |
| permission-audit | Permission audit |
| security-scan | Security scan |
| dependency-check | Dependency check |
| performance-metrics | Performance metrics |
| encoding-check | Encoding issue detection |
| auto-fix | Auto-fix logic |

## Development

To add a new check module:

1. Create module file in modules/
2. Follow interface specification (see rule-conflict.md)
3. Add to parallel/sequential groups in SKILL.md
4. Update format-definitions.toml if needed
