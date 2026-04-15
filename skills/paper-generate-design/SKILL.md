---
name: paper-generate-design
description: "Build or update full-page screens, views, or multi-section layouts in Paper from code or a description. Triggers: 'write to Paper', 'create in Paper from code', 'push page to Paper', 'take this app/page and build it in Paper', 'create a screen', 'build a landing page in Paper', 'update the Paper screen to match code'. Reads source code to understand the screen structure, then builds it section-by-section using write_html with inline CSS and flexbox."
disable-model-invocation: false
---

# Build / Update Screens in Paper

Use this skill to create or update full-page screens in Paper by building HTML/CSS representations of application pages, views, or layouts. Paper uses `write_html` with inline CSS and flexbox as its design language — there are no reusable component instances or variable bindings.

The key insight: read the source code (or description) to understand the screen's structure and design tokens, then reproduce it faithfully in Paper using the project's actual CSS values.

---

## Skill Boundaries

- Use this skill when the deliverable is a **Paper screen** (new or updated) built from code or a description.
- If the user wants to generate **code from a Paper design**, switch to [paper-implement-design](../paper-implement-design/SKILL.md).
- If the user wants to build a **design system library / token documentation**, switch to [paper-generate-library](../paper-generate-library/SKILL.md).

---

## Prerequisites

Before any Paper tool call in a session:

1. **Load the Paper guide**: `get_guide({ topic: "paper-mcp-instructions" })` — mandatory, once per session.
2. **Inspect the canvas**: `get_basic_info` — understand existing artboards, dimensions, fonts in use.
3. **Check fonts**: `get_font_family_info` — before any typographic work. Prefer fonts already in the file unless user specifies otherwise.

Additional requirements:

- Paper MCP server must be connected.
- User should provide source code or a description of the screen to build/update.

---

## Required Workflow

**Follow these steps in order. Do not skip steps.**

### Step 1: Understand the Screen

Before touching Paper, understand what you're building:

1. If building from code, read the relevant source files to understand the page structure, sections, and which components are used.
2. Identify the major sections of the screen (e.g., Header, Hero, Content Panels, Pricing Grid, FAQ Accordion, Footer).
3. For each section, note the UI patterns involved (buttons, inputs, cards, navigation, accordions, etc.).
4. **Extract design tokens from the codebase** — colors, typography, spacing, shadows, radii. Look in:
   - CSS custom properties / variables
   - Tailwind config
   - Design token JSON files
   - CSS-in-JS theme objects
5. Note any images the screen uses. Paper's `write_html` supports images via `<img>` tags with standard `src` attributes.

### Step 2: Create the Artboard

Create the artboard for the screen using `create_artboard`. Convention:

- **Width**: 1440px for desktop, 390px for mobile, 768px for tablet (or match the project's breakpoints).
- **Height**: Will grow as content is added — start with a reasonable estimate.
- **Name**: Descriptive name matching the screen (e.g., "Homepage", "Pricing Page", "Settings / Account").

### Step 3: Build Each Section

**This is the most important step.** Build one section at a time, each in its own `write_html` call targeting the artboard. This incremental approach prevents clipping issues and lets you verify as you go.

For each section:

1. Write HTML with **inline CSS** using flexbox for layout.
2. Use **actual design token values** from the codebase — CSS custom properties (`var(--color-primary)`), hex values, rem/px values matching the project's token system.
3. Use **`layer-name` attributes** on structural elements to keep the Paper node tree organized:
   ```html
   <div layer-name="Section/Header" style="display: flex; flex-direction: column; padding: 24px 80px; background: var(--color-bg-primary);">
     <div layer-name="Nav" style="display: flex; align-items: center; justify-content: space-between;">
       <span style="font-family: Inter; font-size: 18px; font-weight: 700; color: #111827;">Logo</span>
       <div style="display: flex; gap: 24px;">
         <span style="font-family: Inter; font-size: 14px; color: #6B7280;">Features</span>
         <span style="font-family: Inter; font-size: 14px; color: #6B7280;">Pricing</span>
         <span style="font-family: Inter; font-size: 14px; color: #6B7280;">About</span>
       </div>
     </div>
   </div>
   ```

4. After each section, call `get_screenshot` to verify. Look for:
   - **Clipped content** — if content overflows the artboard, use `update_styles` to set the overflowing dimension to `fit-content`.
   - **Cropped text** — line heights or frame sizing cutting off descenders/ascenders.
   - **Overlapping elements** — incorrect sizing or missing flex properties.
   - **Wrong colors/fonts** — compare against the source code's token values.

#### HTML/CSS Rules for Paper

- **Inline styles only** — no `<style>` blocks or external stylesheets.
- **Flexbox for layout** — `display: flex` is the primary layout tool.
- **No `display: grid`** — use nested flexbox instead.
- **No margins** — use `gap` on flex containers and `padding` on containers.
- **No `display: inline`** — use `display: flex` or `display: block`.
- **No HTML tables** — use flexbox rows/columns.
- **Use `<pre>` for code blocks** — preserves whitespace.
- **px for font-size, em for letter-spacing, px for line-height** — Paper convention.
- **All CSS color formats supported** — hex, rgb, rgba, hsl, hsla, oklch, oklab.

#### Repeated Patterns

For repeated elements (list items, cards in a grid, nav links), prefer `duplicate_nodes` + `update_styles` + `set_text_content` over rewriting HTML from scratch. This is faster and reduces errors:

1. Write the first instance via `write_html`.
2. Use `duplicate_nodes` to clone it.
3. Use `set_text_content` and `update_styles` to differentiate each copy.

#### Aligned Rows (Lists, Tables, Nav)

Use fixed-width slots for icons and trailing actions (`flex-shrink: 0`). Don't rely on `gap` alone to align columns across rows:

```html
<div layer-name="Row" style="display: flex; align-items: center; gap: 12px; padding: 12px 16px;">
  <div style="width: 24px; height: 24px; flex-shrink: 0; background: #E5E7EB; border-radius: 4px;"></div>
  <span style="flex: 1; font-family: Inter; font-size: 14px; color: #111827;">Row label</span>
  <span style="width: 80px; flex-shrink: 0; text-align: right; font-family: Inter; font-size: 14px; color: #6B7280;">Value</span>
</div>
```

#### Images

Paper's `write_html` supports `<img>` tags. Consult the Paper guide (`get_guide`) for the supported image source formats and protocols.

For placeholder images when the actual asset isn't available, use a colored rectangle with a label:

```html
<div style="width: 100%; height: 200px; background: #E5E7EB; display: flex; align-items: center; justify-content: center;">
  <span style="font-family: Inter; font-size: 12px; color: #9CA3AF;">Hero Image (1440×400)</span>
</div>
```

### Step 4: Validate the Full Screen

After all sections are built:

1. Call `get_screenshot` on the full artboard to see the complete screen.
2. Compare against the source code or description.
3. Fix issues with targeted `update_styles`, `set_text_content`, or additional `write_html` calls — don't rebuild the entire screen.
4. If the artboard height needs adjusting, use `update_styles` to resize it.

### Step 5: Finish

Call `finish_working_on_nodes` to signal completion and remove working indicators.

---

## Updating an Existing Screen

When updating rather than creating from scratch:

1. Use `get_tree_summary` to inspect the existing screen structure.
2. Use `get_screenshot` to see its current state.
3. Identify which sections need updating and which can stay.
4. For sections that need changes:
   - Use `get_children` to find the target nodes by name.
   - Use `update_styles` for style changes.
   - Use `set_text_content` for text changes.
   - Use `delete_nodes` to remove deprecated sections.
   - Use `write_html` with `insert-children` mode to add new sections.
5. Validate with `get_screenshot` after each modification.

---

## Error Recovery

1. **STOP** on error — do not retry immediately.
2. **Read the error message carefully** to understand what went wrong.
3. If the error is unclear, call `get_tree_summary` or `get_screenshot` to inspect the current state.
4. **Fix the issue** based on the error.
5. **Retry** — because this skill works incrementally (one section per call), errors are naturally scoped. Previous sections remain intact.

---

## Anti-Patterns

- **Writing the entire screen in a single `write_html` call** — too large, likely clips or produces malformed output.
- **Using `display: grid` or margins** — not supported in Paper's HTML parser.
- **Hardcoding values when the codebase has design tokens** — use the project's actual CSS variables or token values.
- **Skipping `get_screenshot` verification** — content may overflow invisibly.
- **Not using `layer-name` attributes** — leaves the Paper node tree disorganized.
- **Skipping `get_guide` / `get_font_family_info` at session start** — causes font loading failures and missed Paper conventions.
- **Rebuilding everything to fix a small issue** — use `update_styles` or `set_text_content` for targeted fixes.
- **Not calling `finish_working_on_nodes` when done** — leaves Paper in an editing state.

---

## Best Practices

- **Read the source code carefully before building.** Understand the exact structure, spacing, colors, and typography before writing any HTML.
- **Extract design tokens from the codebase.** Use the project's actual token values — don't guess colors or spacing.
- **Work section by section.** Never build more than one major section per `write_html` call.
- **Validate visually after each section.** Use `get_screenshot` to catch issues early.
- **Use `layer-name` on every structural element.** Keeps the node tree navigable and makes updates easier.
- **Match existing conventions.** If the Paper file already has screens, match their naming, sizing, and layout patterns.
- **Use `duplicate_nodes` for repetition.** Cloning + modifying is faster and more reliable than writing similar HTML multiple times.
- **Show both visual and code values.** When documenting design tokens alongside the screen, show both the rendered appearance and the CSS variable name.
