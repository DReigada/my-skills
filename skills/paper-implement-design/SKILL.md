---
name: paper-implement-design
description: "Translates Paper designs into production-ready application code with 1:1 visual fidelity. Use when implementing UI code from Paper designs, when user mentions 'implement design', 'generate code', 'implement component', 'export to code', or asks to build components matching Paper specs. Uses get_jsx, get_computed_styles, and get_screenshot to extract design context and produce production-ready code."
disable-model-invocation: false
---

# Implement Design from Paper

## Overview

This skill provides a structured workflow for translating Paper designs into production-ready code with pixel-perfect accuracy. It uses Paper's export tools (`get_jsx`, `get_computed_styles`, `get_fill_image`) to extract design context and produces code that matches the project's framework and conventions.

---

## Skill Boundaries

- Use this skill when the deliverable is **code in the user's repository**.
- If the user asks to create/edit screens inside Paper itself, switch to [paper-generate-design](../paper-generate-design/SKILL.md).
- If the user asks to build a **design system library / token documentation** in Paper, switch to [paper-generate-library](../paper-generate-library/SKILL.md).
- If the user asks to author reusable agent rules (`CLAUDE.md`/`AGENTS.md`), switch to [paper-create-design-system-rules](../paper-create-design-system-rules/SKILL.md).

---

## Prerequisites

Before any Paper tool call in a session:

1. **Load the Paper guide**: `get_guide({ topic: "paper-mcp-instructions" })` — mandatory, once per session.

Additional requirements:

- Paper MCP server must be connected and accessible.
- User must either:
  - Select the target node(s) in Paper (use `get_selection` to read what's selected), OR
  - Provide a node ID directly.
- Project should have an established design system or component library (preferred).

---

## Required Workflow

**Follow these steps in order. Do not skip steps.**

### Step 1: Identify the Target Node

Determine which Paper node to implement:

**Option A: User selects in Paper (preferred)**

Call `get_selection` to get the currently selected node(s). This returns node IDs, names, component types, sizes, and artboard membership.

**Option B: User provides a node ID**

Use the node ID directly in subsequent tool calls.

If the selection or ID is ambiguous, use `get_tree_summary` to explore the node hierarchy and confirm the correct target.

### Step 2: Fetch Design Context

Gather the full design context using multiple Paper tools:

1. **Structure**: `get_tree_summary` — understand the node hierarchy, child components, and layout structure.
2. **JSX export**: `get_jsx` — get a JSX representation of the design. Choose the format that best matches the project:
   - `tailwind` (default) — JSX with Tailwind CSS classes.
   - `inline-styles` — JSX with inline style objects.
3. **Computed styles**: `get_computed_styles` — get exact CSS property values for specific nodes. Use this for precise values that the JSX export may approximate.
4. **Node details**: `get_node_info` — for specific nodes where you need detailed properties (size, visibility, parent, text content).

**Important**: `get_jsx` output is a **design representation**, not final production code. It captures the visual design faithfully but must be adapted to the project's conventions, components, and patterns.

### Step 3: Capture Visual Reference

Call `get_screenshot` on the target node for a visual reference.

This screenshot serves as the source of truth for visual validation throughout implementation. For complex layouts, also take screenshots of individual sections.

### Step 4: Extract Assets

For any nodes containing images:

- Call `get_fill_image` to extract image data. This returns base64-encoded image data and may include the original image URL in metadata.
- Save extracted images to the project's assets directory using appropriate naming conventions.
- For SVG icons, check if the project already has equivalent icons before adding new ones.

**Important**: Do NOT import new icon packages. Use assets extracted from Paper or existing project assets.

### Step 5: Translate to Project Conventions

Translate the Paper export into the project's framework, styles, and conventions.

**Key principles:**

- **Treat `get_jsx` output as a starting point**, not final code. It represents the design, not the production implementation.
- **Reuse existing components** — check if the project already has buttons, inputs, cards, typography components, etc. Use those instead of creating new primitives.
- **Map design values to project tokens** — replace raw hex colors with the project's CSS variables, Tailwind theme values, or design token references. Replace pixel values with the project's spacing scale.
- **Respect project patterns** — use the project's routing, state management, data-fetching, and styling conventions.
- **Use `get_computed_styles` for exact values** — don't read colors or sizes from screenshots. Always use computed styles for precise measurements.

### Step 6: Achieve 1:1 Visual Parity

Strive for pixel-perfect visual parity with the Paper design.

**Guidelines:**

- Prioritize matching the Paper design exactly.
- Avoid hardcoded values — use design tokens from the project where available.
- When project tokens differ from Paper values, prefer project tokens for consistency but adjust spacing/sizing minimally to maintain visual fidelity.
- Follow WCAG requirements for accessibility.

### Step 7: Validate Against Paper

Before marking complete, validate the final UI against the Paper screenshot from Step 3.

**Validation checklist:**

- [ ] Layout matches (spacing, alignment, sizing)
- [ ] Typography matches (font, size, weight, line height)
- [ ] Colors match exactly
- [ ] Interactive states work as designed (hover, active, disabled)
- [ ] Responsive behavior is appropriate
- [ ] Assets render correctly
- [ ] Accessibility standards met

---

## Implementation Rules

### Component Organization

- Place UI components in the project's designated component directory.
- Follow the project's component naming conventions.
- Avoid inline styles unless truly necessary for dynamic values.

### Design System Integration

- ALWAYS check for existing components in the project's design system before creating new ones.
- Map Paper design values to project design tokens.
- When a matching component exists, extend it rather than creating a new one.

### Code Quality

- Avoid hardcoded values — extract to constants or design tokens.
- Keep components composable and reusable.
- Add TypeScript types for component props when the project uses TypeScript.

---

## Examples

### Example 1: Implementing a Button Component

User selects a button in Paper and says: "Implement this as a component"

**Actions:**

1. `get_selection` → get the button node ID
2. `get_jsx(nodeId, format="tailwind")` → get JSX representation
3. `get_computed_styles(nodeId)` → get exact padding, border-radius, colors
4. `get_screenshot(nodeId)` → visual reference
5. Check if the project already has a button component
6. If yes, add a new variant; if no, create using project conventions
7. Map colors to project tokens (e.g., `bg-primary-500`, `hover:bg-primary-600`)
8. Validate against screenshot

### Example 2: Building a Dashboard Layout

User selects a full dashboard screen and says: "Implement this page"

**Actions:**

1. `get_selection` → get the dashboard artboard ID
2. `get_tree_summary(nodeId)` → understand section structure (header, sidebar, content, cards)
3. `get_jsx(nodeId, format="tailwind")` → get full JSX export
4. `get_screenshot(nodeId)` → full page reference
5. `get_fill_image` for any image nodes → extract assets
6. Break into sections, implement each using existing project components
7. Use `get_computed_styles` for precise spacing between sections
8. Validate responsive behavior

---

## Best Practices

### Always Start with Context

Never implement based on assumptions. Always fetch `get_jsx`, `get_computed_styles`, and `get_screenshot` first.

### Use `get_computed_styles` for Exact Values

Screenshots are for visual reference only. Always use `get_computed_styles` for precise colors, sizes, spacing, and typography values.

### Incremental Validation

Validate frequently during implementation, not just at the end. This catches issues early.

### Document Deviations

If you must deviate from the Paper design (e.g., for accessibility or technical constraints), document why in code comments.

### Reuse Over Recreation

Always check for existing components before creating new ones. Consistency across the codebase is more important than exact Paper replication.

### Design System First

When in doubt, prefer the project's design system patterns over literal Paper translation.

---

## Common Issues and Solutions

### Issue: `get_jsx` output doesn't match project patterns

**Cause:** `get_jsx` produces generic JSX that doesn't know about your project's component library.
**Solution:** Use `get_jsx` as a structural reference, then rewrite using project components. Use `get_computed_styles` for exact values.

### Issue: Design doesn't match after implementation

**Cause:** Visual discrepancies between implemented code and Paper design.
**Solution:** Compare side-by-side with the screenshot from Step 3. Use `get_computed_styles` on specific nodes to get exact values.

### Issue: Colors or spacing don't match project tokens

**Cause:** Paper design uses raw values that don't map 1:1 to project tokens.
**Solution:** Find the closest project token. If no close match exists, document the deviation and use the exact Paper value.

### Issue: Complex nested layouts

**Cause:** Deeply nested Paper layouts produce verbose `get_jsx` output.
**Solution:** Use `get_tree_summary` to understand the hierarchy first. Implement section-by-section, fetching `get_jsx` for individual sections rather than the entire layout.
