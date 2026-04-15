> Part of the [paper-generate-library skill](../SKILL.md).

# Discovery Phase Reference

Phase 0 of every design library build: analyze the codebase for tokens, inspect the Paper canvas for existing conventions, build the plan, and resolve conflicts before any write operations.

---

## 1. Codebase Analysis — Finding Token Sources

### Search Priority Order

Look for token sources in this order. Stop as soon as you find a definitive source; multiple formats can coexist:

1. Design token files: `*.tokens.json`, `tokens/*.json`, `src/tokens/**`
2. CSS variable files: `variables.css`, `tokens.css`, `theme.css`, `global.css`, `globals.css`
3. Tailwind config: `tailwind.config.js`, `tailwind.config.ts`, or CSS `@theme` blocks (Tailwind v4)
4. CSS-in-JS theme objects: `theme.ts`, `createTheme`, `ThemeProvider`
5. Platform-specific: iOS Asset catalogs (`.xcassets`), Android `themes.xml`, `colors.xml`

### CSS Custom Properties (Most Common for Web)

**What to search for:**

```
:root { ... }
@theme { ... }          ← Tailwind v4
--color-*, --spacing-*, --radius-*, --shadow-*, --font-*
```

**Pattern:** `/--[\w-]+:\s*[^;]+/g`

**Common file locations:** `src/styles/tokens.css`, `src/styles/variables.css`, `src/app/globals.css`

**Extraction:**

| CSS Property | Token Name | Category | Value |
|---|---|---|---|
| `--color-bg-primary: #fff` | `color/bg/primary` | Color | `#ffffff` |
| `--color-text-secondary: #757575` | `color/text/secondary` | Color | `#757575` |
| `--spacing-sm: 8px` | `spacing/sm` | Spacing | `8px` |
| `--radius-md: 8px` | `radius/md` | Radius | `8px` |

**Naming rule:** Replace hyphens with slashes at category boundaries. Keep hyphens within the final segment: `--color-bg-primary` → `color/bg/primary`, but `--color-bg-primary-hover` → `color/bg/primary-hover`.

### Tailwind Configuration

**In `tailwind.config.js` / `tailwind.config.ts`:**

```javascript
// theme.extend.colors → color tokens
{ primary: { DEFAULT: '#3366FF', light: '#6699FF', dark: '#0033CC' } }
// → color/primary/default, color/primary/light, color/primary/dark

// theme.extend.spacing → spacing tokens
{ 'xs': '4px', 'sm': '8px', 'md': '16px' }
// → spacing/xs = 4px, spacing/sm = 8px, spacing/md = 16px

// theme.extend.borderRadius → radius tokens
{ 'sm': '4px', 'md': '8px', 'lg': '16px' }
// → radius/sm = 4px, radius/md = 8px, radius/lg = 16px
```

**In Tailwind v4 (CSS-based config):**

```css
@theme inline {
  --color-primary: #3366FF;
  --spacing-sm: 8px;
}
```

Extract like CSS custom properties — the `@theme` block is the token source.

Tailwind utility class names (`bg-blue-500`, `p-4`) are not tokens — extract values from the config, not class names.

### Design Token Community Group (DTCG) Format

**Pattern:** `*.tokens.json` or `tokens/*.json`. Find source files, not generated outputs.

```json
{
  "color": {
    "bg": {
      "primary": { "$type": "color", "$value": "#ffffff" }
    }
  },
  "spacing": {
    "sm": { "$type": "dimension", "$value": "8px" }
  }
}
```

Nested keys map to slash-separated names: `color.bg.primary` → `color/bg/primary`.

### CSS-in-JS / Theme Objects

**Search for:** `createTheme`, `ThemeProvider`, `theme = {}`, styled-components, Emotion, Stitches, vanilla-extract

```typescript
// theme.colors.bg.primary → color/bg/primary
// theme.spacing.sm        → spacing/sm
// Multiple theme objects (lightTheme, darkTheme) → document both modes
```

### Detecting Dark Mode

| Platform | Signal |
|---|---|
| Web (CSS) | `@media (prefers-color-scheme: dark)`, `.dark { }`, `[data-theme="dark"]` |
| Web (Tailwind) | `darkMode: 'class'` or `darkMode: 'media'` in config |
| Web (JS) | Separate `darkTheme` object alongside `lightTheme` |
| iOS | Dual-appearance asset catalog, `traitCollection.userInterfaceStyle` |
| Android | `themes.xml` with `Theme.*.Night`, `values-night/` folder |

If dark mode exists → document both light and dark token values side by side in the color documentation.

### Shadow/Elevation Extraction

```css
/* Look for: box-shadow, --shadow-* */
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
--shadow-md: 0 4px 6px -1px rgba(0,0,0,0.10);
--shadow-lg: 0 10px 15px -3px rgba(0,0,0,0.10);
```

Capture the full `box-shadow` shorthand value for each elevation level. These will be applied directly via CSS in the shadow card documentation.

### Typography Extraction

| Code token | What to capture |
|---|---|
| `font-family` | Family name (e.g., "Inter") |
| `font-size` | Size in px |
| `line-height` | Height in px or unitless multiplier |
| `font-weight` | Weight (400, 600, 700, etc.) |
| `letter-spacing` | Spacing in em or px |

Look for composite type scale definitions (e.g., `--font-heading-lg: 600 32px/40px "Inter"`) and individual property tokens.

### Component Extraction

For each component found in code, capture:

1. **Name** — the component identifier
2. **Key props** — union-type props that define visual variants (size, variant, type)
3. **States** — interaction/visual states (default, hover, disabled, active, loading)
4. **Description** — what the component does and when to use it

```typescript
// Example:
interface ButtonProps {
  size: 'sm' | 'md' | 'lg';
  variant: 'primary' | 'secondary' | 'ghost';
  disabled?: boolean;
  loading?: boolean;
  label: string;
  icon?: ReactNode;
}
// → Document: Button, 3 sizes × 3 variants, states: default/hover/disabled/loading
```

---

## 2. Paper Canvas Inspection

Call `get_basic_info` at the start of every build. Capture:

- **Existing artboards** — names, dimensions, organization pattern
- **Fonts in use** — which font families are already loaded
- **Canvas conventions** — naming style, dimensions, any existing documentation

If the canvas is empty, use the defaults from [naming-conventions.md](naming-conventions.md). If content exists, match its conventions.

---

## 3. Building the Plan

After codebase analysis and canvas inspection, produce inventories and present to the user.

### Token Inventory Table

| Code Token | CSS Variable | Raw Value | Category | Dark Value |
|---|---|---|---|---|
| `theme.colors.blue[500]` | `--color-blue-500` | `#3B82F6` | Color (primitive) | — |
| `theme.colors.bg.primary` | `--color-bg-primary` | `#ffffff` | Color (semantic) | `#111827` |
| `theme.spacing.sm` | `--spacing-sm` | `8px` | Spacing | — |
| `theme.radii.md` | `--radius-md` | `8px` | Radius | — |
| `theme.shadows.md` | `--shadow-md` | `0 4px 6px rgba(0,0,0,0.1)` | Shadow | — |

### Component Inventory Table

| Component | Key Props | States | Notes |
|---|---|---|---|
| `Button` | size (sm/md/lg), variant (primary/secondary/ghost) | default, hover, disabled, loading | Core CTA component |
| `Avatar` | size (sm/md/lg), type (image/initials/icon) | — | User identity display |

### User-Facing Checkpoint Message

Present before any writes. Never begin Phase 1 without explicit user approval.

```
Here's what I found and what I plan to build:

CODEBASE ANALYSIS
  Colors: {N} primitives ({families}), {M} semantic tokens ({light/dark if applicable})
  Spacing: {N} tokens ({range})
  Typography: {N} type styles
  Shadows: {N} elevation levels
  Border Radii: {N} tokens
  Components: {list}

EXISTING CANVAS
  Artboards: {list or "empty canvas"}
  Fonts: {list}

PLAN
  Artboards to create: {list with dimensions}
  Documentation sections: {list}
  Components to document: {list}

WHAT I WON'T DOCUMENT (and why)
  - {item}: not a visual design token (e.g., z-index, animation timing)
  - {item}: already documented on existing artboard

Shall I proceed?
```

---

## 4. Conflict Resolution

When the codebase has multiple token sources or the canvas already has documentation that disagrees with code values, **always ask the user**. Never silently pick one.

### Code Wins (default)

- Raw token values — code is the live production value
- Token naming — CSS variable names are the developer contract
- Dark mode values — code defines the actual mode behavior

### Canvas Wins (default)

- Artboard organization — if a well-organized structure exists, extend it
- Naming style — if designers already use specific naming, match it

### When Neither Is Clear

Propose a resolution and ask:
> "Code has `--color-primary: #3366FF` but the existing canvas documentation shows `#5B7FFF` for the primary color. Which is current?"
