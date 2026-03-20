# codex-generator

A Claude Code skill that generates an **Obsidian-format code knowledge base** from any project — no external indexing tools required.

Uses Claude Code's native tools (Glob, Grep, Read, Agent) to analyze codebase structure, detect modules, trace call relationships, and produce multi-layered, interlinked documentation formatted for Obsidian's graph view and Dataview queries.

## What it generates

```
{project}-codex/
├── INDEX.md                   # Project overview + Module Map of Content
├── README.md                  # Original project README (copied as-is)
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
- Bilingual naming: English code names + Chinese annotations

## Prerequisites

- Claude Code with access to the target project directory
- No external tools or indexing required

## Installation

```bash
# Install globally (available in all Claude Code projects)
mkdir -p ~/.claude/skills/codex-generator
cp SKILL.md ~/.claude/skills/codex-generator/SKILL.md
```

## Usage

Once installed, trigger the skill in Claude Code:

```
Generate a code knowledge base for this project
```
```
Create an Obsidian codex for the auth module
```
```
Build code docs I can browse in Obsidian
```

The skill runs in **three phases**:

1. **Scan** — analyzes project structure using Glob/Grep/Read, detects modules, builds dependency graph
2. **Skeleton** — generates module structure and INDEX.md, presents for your review
3. **Fill** — after confirmation, generates all file-level docs with call graphs and code snippets

## Supported Languages

TypeScript/JavaScript, Python, Go, Rust, Java/Kotlin, and any other language via generic directory-based analysis.

## License

MIT
