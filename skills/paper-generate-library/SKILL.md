---
name: paper-generate-library
description: "Build a professional design library in Paper from a codebase. Use when the user wants to document design tokens (colors, typography, spacing, shadows, radii), create foundation pages, or build component documentation artboards. Analyzes code to extract tokens and produces organized HTML/CSS artboards in Paper."
disable-model-invocation: false
---

# Design System Library Builder — Paper MCP Skill

Build professional design libraries in Paper that document a codebase's design tokens and component patterns. This skill analyzes code to extract tokens, then produces organized HTML/CSS artboards covering colors, typography, spacing, shadows, radii, and component anatomy.

**Paper creates designs via `write_html` (inline CSS, flexbox layout).** It has no variables system, no reusable components, no variant matrices. The goal is high-quality visual documentation of what exists in code — not a live design system with bindings.

---

## 1. Prerequisites

Before any Paper tool call in a session:

1. **Load the Paper guide**: `get_guide({ topic: "paper-mcp-instructions" })` — mandatory, once per session
2. **Inspect the canvas**: `get_basic_info` — understand existing artboards, dimensions, fonts in use
3. **Check fonts**: `get_font_family_info` — before any typographic work. Prefer fonts already in the file unless user specifies otherwise

---

## 2. Mandatory Workflow

Every design library build follows this phase order.

```
Phase 0: DISCOVERY (no Paper writes yet)
  0a. Analyze codebase → extract tokens, components, naming conventions
  0b. Call get_basic_info → understand existing artboards and conventions
  0c. Lock v1 scope → agree on exact token set + component list before any creation
  0d. Resolve conflicts (multiple token sources, code vs existing design) → ask user
  ✋ CHECKPOINT: present full plan, await explicit approval

Phase 1: ARTBOARD SETUP + COVER
  1a. Create cover artboard via create_artboard (1440×900)
  1b. Write cover HTML via write_html (system name, tagline, version)
  1c. Create foundation artboards (one per major section)
  ✋ CHECKPOINT: show get_screenshot of cover, await approval

Phase 2: FOUNDATIONS DOCUMENTATION
  For each token category (in this order):
    2a. Colors — primitive palette + semantic tokens
    2b. Typography — type specimens at actual sizes
    2c. Spacing — scale bars showing each value
    2d. Shadows — elevation cards with progressive depth
    2e. Border Radius — demo shapes at each radius value
  After each artboard: get_screenshot to verify, fix clipping if needed
  ✋ CHECKPOINT: show screenshots of all foundations, await approval

Phase 3: COMPONENT DOCUMENTATION (optional — one per component)
  For each component:
    3a. Create artboard
    3b. Write HTML: title, description, props table, visual state examples
    3c. get_screenshot to verify
  ✋ CHECKPOINT per component: show screenshot, await approval
```

**If user rejects at any checkpoint**: fix before moving on. Never build on rejected work.

---

## 3. Critical Rules

1. **Load `get_guide` first** — every session, before any other Paper tool call.
2. **One `write_html` call per visual group** — build incrementally (one section at a time). Never try to write an entire documentation page in a single call.
3. **Verify with `get_screenshot`** after every meaningful write. If content clips a frame, use `update_styles` to set the overflowing dimension to `fit-content`.
4. **Call `finish_working_on_nodes`** when done creating or editing a set of nodes.
5. **Use `layer-name` attributes** on HTML elements to keep the Paper tree organized: `<div layer-name="Section/Colors">`.
6. **Prefer `duplicate_nodes` + `update_styles` + `set_text_content`** for repeated patterns (e.g., multiple swatch cards). Faster than rewriting HTML from scratch.
7. **Use px for font sizes, em for letter-spacing, px for line-height** — Paper convention.
8. **Show token names AND code references** — every swatch/bar/card must display both the human-readable name and the CSS variable (or JS path) so designers and developers can both use the documentation.
9. **Use actual CSS custom properties** where the project defines them. Write `background: var(--color-primary)` in `write_html` styles when the project uses CSS custom properties — this makes documentation values live.
10. **Never skip user checkpoints** — design decisions require human judgment.

---

## 4. Token Architecture

Even though Paper doesn't create live tokens, understanding token architecture helps produce well-organized documentation.

### Primitive / Semantic Split

Most design systems have two layers:

- **Primitives** — raw values with no semantic meaning: `blue/500 = #3B82F6`, `gray/100 = #F3F4F6`
- **Semantic** — role-based aliases: `color/bg/primary → blue/50 (light) / gray/900 (dark)`

Document both layers separately. Primitives show the full palette; semantic tokens show how those colors are used in context.

### Complexity → Artboard Count

| Token count | Approach |
|-------------|----------|
| < 50 tokens | Single "Foundations" artboard with all sections |
| 50–200 tokens | One artboard per category (Colors, Typography, Spacing, etc.) |
| 200+ tokens | Split large categories further (Primitive Colors, Semantic Colors, etc.) |

### Dark Mode

If the codebase has light/dark modes, document token values side-by-side: show the light value and dark value for each semantic token. Use a split-column layout or adjacent swatch pairs.

---

## 5. Paper Tool Quick Reference

| When | Tool | Purpose |
|------|------|---------|
| Session start | `get_guide({ topic: "paper-mcp-instructions" })` | Load Paper instructions |
| Session start | `get_basic_info` | Understand canvas state, existing artboards, fonts |
| Before typography | `get_font_family_info` | Discover available fonts |
| Create new page | `create_artboard` | Create artboard for a section |
| Build documentation | `write_html` | Write HTML+CSS into artboard |
| Edit existing | `update_styles` / `set_text_content` | Modify without full rewrite |
| Repeat patterns | `duplicate_nodes` + `update_styles` + `set_text_content` | Clone and modify layouts |
| Verify quality | `get_screenshot` | Visual QA check |
| Inspect structure | `get_tree_summary` / `get_children` | Debug node hierarchy |
| Rename nodes | `rename_nodes` | Fix artboard/layer names |
| Done editing | `finish_working_on_nodes` | Signal completion |

---

## 6. Anti-Patterns

- **Starting to write HTML before scope is locked with user** — always complete Phase 0 first
- **Writing entire documentation in a single `write_html` call** — too large, likely clips or produces malformed output
- **Hardcoding token values without showing the CSS variable name** — developers need the code reference, not just the visual
- **Skipping `get_screenshot` verification** — content may overflow the artboard invisibly
- **Not calling `finish_working_on_nodes` when done** — leaves Paper in an editing state
- **Skipping `get_guide` / `get_font_family_info` at session start** — causes font loading failures and missed Paper conventions
- **Using fonts not available in the file** — always verify with `get_font_family_info` first
- **Building component docs before foundations are complete** — foundations establish the visual language that component docs reference

---

## 7. Reference Docs

Load on demand — each reference is authoritative for its scope:

| Doc | Phase | Load when |
|-----|-------|-----------|
| [discovery-phase.md](references/discovery-phase.md) | 0 | Starting any build — codebase analysis + canvas inspection |
| [documentation-creation.md](references/documentation-creation.md) | 1–3 | Creating any artboard content with HTML/CSS |
| [naming-conventions.md](references/naming-conventions.md) | Any | Naming artboards, sections, tokens |
