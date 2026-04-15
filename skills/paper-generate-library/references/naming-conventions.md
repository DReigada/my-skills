> Part of the [paper-generate-library skill](../SKILL.md).

# Naming Conventions Reference

Naming conventions for token documentation, artboard organization, and section structure in Paper design libraries. These conventions are tool-agnostic best practices drawn from production design systems (Material 3, Polaris, Simple DS).

---

## 1. Token Naming in Documentation

### Slash hierarchy

Tokens use slash-separated paths for grouping. This creates a clear visual hierarchy in documentation and maps directly to code token structures.

```
{category}/{subcategory}/{role}
```

Examples:

```
color/bg/primary
color/bg/secondary
color/text/primary
color/text/muted
color/border/default
color/border/focus
color/feedback/error
color/feedback/success
spacing/xs
spacing/sm
spacing/md
spacing/lg
spacing/xl
spacing/2xl
radius/none
radius/sm
radius/md
radius/lg
radius/full
```

### Primitive tokens

Primitives hold raw values. They use a flat `{family}/{step}` pattern:

```
blue/50   blue/100   blue/200   ...   blue/900
gray/50   gray/100   gray/200   ...   gray/900
```

Step numbers follow the codebase convention. If no convention exists, use `100–900` in increments of 100.

### Semantic tokens

Semantic tokens reference primitives by role. They use `{category}/{role}` or `{category}/{subcategory}/{role}`:

```
color/bg/primary         → blue/50 (light), gray/900 (dark)
color/bg/secondary       → gray/100 (light), gray/800 (dark)
color/text/primary       → gray/900 (light), white (dark)
spacing/sm               → 8px
radius/md                → 8px
```

### Casing

**Default:** lowercase with forward slashes: `color/bg/primary`, `spacing/2xl`.

**Match the codebase** when it uses a different convention (e.g., PascalCase, camelCase). The documentation token names should be recognizable to developers who work in that codebase.

**Always show the actual CSS variable or code path** alongside the documentation name:
- Token name: `color/bg/primary`
- Code reference: `var(--color-bg-primary)` or `theme.colors.bg.primary`

---

## 2. Artboard Naming

### Default pattern

Use plain, descriptive names. This is the most common pattern in professional design systems.

```
Cover
Foundations
Foundations/Colors
Foundations/Typography
Foundations/Spacing
Foundations/Elevation
Foundations/Radius
Components/Button
Components/Input
Components/Card
```

For small libraries (< 50 tokens), a single `Foundations` artboard is sufficient — no need to split by category.

### Artboard dimensions

| Artboard | Recommended size |
|----------|-----------------|
| Cover | 1440 × 900 |
| Foundations (single) | 1440 × auto (hug content) |
| Per-category foundations | 1440 × auto |
| Component docs | 1440 × auto |

Width of 1440px is the convention across major design system libraries. Height should accommodate content without clipping.

---

## 3. Section Naming Within Artboards

Each documentation section within an artboard uses a consistent heading hierarchy:

```
Section heading (32px, bold)         — "Colors", "Typography", "Spacing"
  Subsection label (13px, bold)      — "Primitives", "Semantic", "Scale"
    Item labels (10–12px, regular)   — token name, value, code reference
```

Section names should match the token categories used in the codebase. Common sections:

- **Colors** — primitive palette + semantic tokens
- **Typography** — type scale specimens
- **Spacing** — spacing scale bars
- **Elevation** / **Shadows** — shadow depth cards
- **Border Radius** — radius demo shapes

---

## 4. When to Match Existing vs Use Defaults

**Always inspect before naming anything.** Call `get_basic_info` to discover existing artboard names and conventions.

### Match existing conventions when:

- The canvas already has artboards with a consistent naming pattern
- The codebase uses specific token naming that designers already recognize
- The user explicitly references an existing convention

### Use defaults from this document when:

- Starting fresh on an empty canvas
- Existing conventions are inconsistent (mixed styles = no convention to match)
- The user asks for best-practice organization

### When code naming and documentation naming conflict:

Keep both. Show the human-readable documentation name for scannability AND the exact code reference for developer utility:

```
color/bg/primary                    ← documentation name (readable)
var(--color-bg-primary)             ← code reference (copy-pasteable)
```
