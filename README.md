# my-skills

Private repo for agent skills for [Paper](https://paper.design) via the Paper MCP server.

## Skills

| Skill | What it does |
|-------|-------------|
| **paper-generate-library** | Build a design system library in Paper from a codebase. Extracts design tokens (colors, typography, spacing, shadows, radii) and produces organized HTML/CSS documentation artboards. |
| **paper-generate-design** | Build or update full-page screens in Paper from code or a description. Reads source code to understand structure, then builds section-by-section using `write_html`. |
| **paper-implement-design** | Translate Paper designs into production-ready code. Uses `get_jsx`, `get_computed_styles`, and `get_screenshot` to extract design context and produce code matching project conventions. |
| **paper-create-design-system-rules** | Generate project-level AI agent rules (CLAUDE.md, AGENTS.md, .cursor/rules) that guide consistent Paper-to-code workflows. |

## Install

```bash
npx skills add dreigada/my-skills
```

## Requirements

- [Paper MCP server](https://paper.design) connected to your editor
