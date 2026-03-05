# codex-generator

A Claude Code skill that generates an **Obsidian-format code knowledge base** from any GitNexus-indexed project.

Produces multi-layered, interlinked documentation with module overviews, file docs, function/class descriptions, and call relationships — all formatted for Obsidian's graph view and Dataview queries.

## What it generates

```
{project}-codex/
├── INDEX.md                   # Project overview + Module Map of Content
├── {module-a}/
│   ├── _overview.md           # Module responsibilities, key flows, file list
│   ├── {sourceFile}.md        # Per-file: classes, functions, call relationships
│   └── ...
├── {module-b}/
│   └── ...
└── ...
```

All files use:
- **Obsidian wikilinks** `[[file#heading]]` for navigable cross-references
- **YAML frontmatter** compatible with Dataview
- **Callouts** for warnings, design notes, and tips
- Bilingual naming: English code names + Chinese annotations (configurable)

## Prerequisites

- [GitNexus](https://gitnexus.dev) MCP server running and connected to Claude Code
- Target project indexed: `npx gitnexus analyze` in the project directory

## Installation

```bash
# Install globally (available in all Claude Code projects)
mkdir -p ~/.claude/skills/codex-generator
curl -o ~/.claude/skills/codex-generator/SKILL.md \
  https://raw.githubusercontent.com/AlexHercules/codex-generator-skill/main/codex-generator/SKILL.md
```

Or clone the repo:

```bash
git clone https://github.com/AlexHercules/codex-generator-skill \
  ~/.claude/skills/codex-generator
```

## Usage

Once installed, trigger the skill in Claude Code by describing what you want:

```
Generate a code knowledge base for this project
```
```
Create an Obsidian codex for the auth module
```
```
Build code docs I can browse in Obsidian
```

The skill runs in **two phases**:

1. **Skeleton** — reads GitNexus clusters and processes, generates module structure, presents it for your review
2. **Fill** — after your confirmation, generates all file-level docs with call graphs and code snippets

## Example output

`auth/_overview.md`:
```markdown
---
type: module
module: auth
source: src/auth/
tags: [认证, middleware]
generated: 2026-03-05
---

# auth（认证模块）

## 职责
Handles JWT-based stateless authentication...

## 文件清单
| 文件 | 职责 |
|------|------|
| [[authService]] | 令牌签发与校验 |
| [[authMiddleware]] | Express 中间件 |
```

## Dependencies

| Dependency | Role |
|------------|------|
| [GitNexus MCP](https://gitnexus.dev) | Code symbol index, cluster detection, call graph |

## License

MIT
