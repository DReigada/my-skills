> Part of the [paper-generate-library skill](../SKILL.md).

# Documentation Creation Reference

HTML/CSS patterns for creating design token documentation in Paper. Each section provides a complete template for `write_html`. Build one section per `write_html` call — never attempt an entire page in a single call.

---

## 1. Cover Artboard

The cover artboard is a branded title card. Create it via `create_artboard` at **1440 × 900**.

### HTML Template

```html
<div layer-name="Cover" style="
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 16px;
  background: #0f0f11;
">
  <div style="
    font-size: 64px;
    font-weight: 700;
    color: #ffffff;
    text-align: center;
  ">{System Name}</div>
  <div style="
    font-size: 20px;
    font-weight: 400;
    color: rgba(255,255,255,0.7);
    text-align: center;
    max-width: 600px;
  ">{Tagline — one sentence describing the design system}</div>
  <div style="
    font-size: 13px;
    font-weight: 500;
    color: rgba(255,255,255,0.45);
    text-align: center;
  ">{Version or date}</div>
</div>
```

Customize the background color to match the project's brand primary. If the project has a CSS variable for the brand color, use it: `background: var(--color-brand-primary)`.

---

## 2. Foundations Layout

Foundation artboards are **1440px wide** with content hugging vertically. Sections stack with **64–80px gaps**. Use a consistent outer structure:

### Outer Container

```html
<div layer-name="Foundations" style="
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 80px;
  padding: 80px;
  background: #ffffff;
">
  <!-- Sections go here as separate write_html calls -->
</div>
```

Write this container first via `write_html`, then insert each section as children using subsequent `write_html` calls with `mode: "insert-children"` targeting this container.

### Section Structure Pattern

Every documentation section follows this structure:

```html
<div layer-name="Section/{Name}" style="
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
">
  <div style="font-size: 32px; font-weight: 700; color: #121212;">{Section Title}</div>
  <div style="font-size: 14px; color: #666666; max-width: 720px;">
    {Brief description of this token category}
  </div>
  <!-- Content: swatches, specimens, bars, cards -->
</div>
```

---

## 3. Color Swatches

Color documentation shows the full palette (primitives) and semantic token assignments.

### Single Swatch Card

```html
<div layer-name="Swatch/{token-name}" style="
  display: flex;
  flex-direction: column;
  gap: 6px;
  width: 88px;
">
  <div style="
    width: 88px;
    height: 88px;
    border-radius: 8px;
    background: {value or var(--token-name)};
  "></div>
  <div style="font-size: 11px; font-weight: 500; color: #333333;">
    {leaf name, e.g. "primary"}
  </div>
  <div style="font-size: 9px; color: #999999;">
    {full path, e.g. "color/bg/primary"}
  </div>
  <div style="font-size: 9px; color: #999999; font-family: monospace;">
    {code ref, e.g. "var(--color-bg-primary)"}
  </div>
</div>
```

### Color Section (primitives + semantic)

```html
<div layer-name="Section/Colors" style="
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
">
  <div style="font-size: 32px; font-weight: 700; color: #121212;">Colors</div>
  <div style="font-size: 14px; color: #666666; max-width: 720px;">
    Primitive color palette and semantic color tokens.
    Semantic tokens reference primitives — always use semantic tokens in components.
  </div>

  <!-- Primitives subsection -->
  <div style="font-size: 13px; font-weight: 700; color: #8c8c8c;">Primitives</div>
  <div layer-name="Primitives/Row" style="
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
  ">
    <!-- Insert swatch cards here -->
  </div>

  <!-- Semantic subsection -->
  <div style="font-size: 13px; font-weight: 700; color: #8c8c8c;">Semantic</div>
  <div layer-name="Semantic/Row" style="
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
  ">
    <!-- Insert swatch cards here -->
  </div>
</div>
```

### Primitive Color Family Row

For color scales (blue/50 through blue/900), group as a row:

```html
<div layer-name="Family/{color}" style="
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
">
  <div style="font-size: 12px; font-weight: 600; color: #555555;">{Color Family}</div>
  <div style="display: flex; gap: 8px; flex-wrap: wrap;">
    <!-- One swatch per step: 50, 100, 200, ..., 900 -->
  </div>
</div>
```

### Dark Mode Side-by-Side

When documenting light/dark semantic tokens, show both modes:

```html
<div layer-name="Token/{name}" style="
  display: flex;
  gap: 12px;
  align-items: center;
">
  <div style="width: 48px; height: 48px; border-radius: 6px; background: {light-value};"></div>
  <div style="width: 48px; height: 48px; border-radius: 6px; background: {dark-value};"></div>
  <div style="display: flex; flex-direction: column; gap: 2px;">
    <div style="font-size: 12px; font-weight: 500; color: #333;">{token name}</div>
    <div style="font-size: 10px; color: #999; font-family: monospace;">{code reference}</div>
    <div style="font-size: 10px; color: #999;">Light: {light-value} · Dark: {dark-value}</div>
  </div>
</div>
```

---

## 4. Type Specimens

Typography specimens show each text style at actual size with specs.

### Single Specimen Row

```html
<div layer-name="Type/{style-name}" style="
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding: 16px 0;
  border-bottom: 1px solid #e5e5e5;
  width: 100%;
">
  <div style="font-size: 11px; font-weight: 500; color: #8c8c8c;">
    {Style name, e.g. "Heading / Large"}
  </div>
  <div style="
    font-family: {family};
    font-size: {size}px;
    font-weight: {weight};
    line-height: {line-height}px;
    color: #121212;
  ">The quick brown fox jumps over the lazy dog</div>
  <div style="font-size: 11px; color: #aaaaaa; font-family: monospace;">
    {family} {weight} · {size}px / {line-height}px · {code reference}
  </div>
</div>
```

### Typography Section

```html
<div layer-name="Section/Typography" style="
  display: flex;
  flex-direction: column;
  gap: 0;
  width: 100%;
">
  <div style="font-size: 32px; font-weight: 700; color: #121212; margin-bottom: 24px;">
    Typography
  </div>
  <!-- Insert specimen rows here — they stack with their own padding and borders -->
</div>
```

---

## 5. Spacing Scale Bars

Each spacing token rendered as a colored bar whose width matches the value.

### Single Spacing Bar

```html
<div layer-name="Spacing/{name}" style="
  display: flex;
  align-items: center;
  gap: 16px;
  width: 100%;
">
  <div style="
    width: {value}px;
    min-width: {value}px;
    height: 16px;
    border-radius: 3px;
    background: #387afa;
  "></div>
  <div style="font-size: 12px; color: #555555; white-space: nowrap;">
    {name} · {value}px · <span style="font-family: monospace; color: #999;">{code ref}</span>
  </div>
</div>
```

### Spacing Section

```html
<div layer-name="Section/Spacing" style="
  display: flex;
  flex-direction: column;
  gap: 12px;
  width: 100%;
">
  <div style="font-size: 32px; font-weight: 700; color: #121212;">Spacing</div>
  <div style="font-size: 14px; color: #666666; max-width: 720px;">
    Spacing scale used for padding, margin, and gap values throughout the system.
  </div>
  <!-- Insert spacing bars here, smallest to largest -->
</div>
```

---

## 6. Shadow / Elevation Cards

White cards with progressively stronger box-shadows.

### Single Shadow Card

```html
<div layer-name="Shadow/{name}" style="
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 120px;
  height: 120px;
  border-radius: 8px;
  background: #ffffff;
  box-shadow: {shadow-value};
">
  <div style="font-size: 12px; font-weight: 500; color: #333333;">
    {name, e.g. "Medium"}
  </div>
  <div style="font-size: 10px; color: #8c8c8c; text-align: center; font-family: monospace;">
    {shadow-value shorthand}
  </div>
</div>
```

### Elevation Section

```html
<div layer-name="Section/Elevation" style="
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
">
  <div style="font-size: 32px; font-weight: 700; color: #121212;">Elevation</div>
  <div style="
    display: flex;
    gap: 32px;
    padding: 40px 24px;
    background: #f7f7f7;
    border-radius: 8px;
  ">
    <!-- Insert shadow cards here, lightest to strongest -->
  </div>
</div>
```

---

## 7. Border Radius Demo

Shapes showing each radius token value.

### Single Radius Card

```html
<div layer-name="Radius/{name}" style="
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  width: 96px;
">
  <div style="
    width: 72px;
    height: 72px;
    border-radius: {value}px;
    background: rgba(56, 122, 250, 0.15);
    border: 1.5px solid #387afa;
  "></div>
  <div style="font-size: 11px; font-weight: 500; color: #333333;">
    {name, e.g. "md"}
  </div>
  <div style="font-size: 10px; color: #8c8c8c;">
    {value}px
  </div>
</div>
```

### Radius Section

```html
<div layer-name="Section/Radius" style="
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
">
  <div style="font-size: 32px; font-weight: 700; color: #121212;">Border Radius</div>
  <div style="
    display: flex;
    gap: 24px;
    padding: 24px;
    background: #f7f7f7;
    border-radius: 8px;
  ">
    <!-- Insert radius cards here, smallest to largest (or none to full) -->
  </div>
</div>
```

---

## 8. Component Documentation

Component docs show anatomy, props, and visual state examples.

### Component Page Structure

```html
<div layer-name="Component/{Name}" style="
  display: flex;
  flex-direction: column;
  gap: 40px;
  padding: 60px 80px;
  width: 100%;
  background: #ffffff;
">
  <!-- Header -->
  <div style="display: flex; flex-direction: column; gap: 12px;">
    <div style="font-size: 36px; font-weight: 700; color: #121212;">{Component Name}</div>
    <div style="font-size: 16px; color: #555555; max-width: 720px;">
      {Description — what it does and when to use it}
    </div>
  </div>

  <!-- Props table -->
  <div layer-name="Props" style="display: flex; flex-direction: column; gap: 16px;">
    <div style="font-size: 20px; font-weight: 700; color: #121212;">Properties</div>
    <!-- Table rows -->
  </div>

  <!-- Visual examples -->
  <div layer-name="Examples" style="display: flex; flex-direction: column; gap: 16px;">
    <div style="font-size: 20px; font-weight: 700; color: #121212;">Examples</div>
    <!-- State/variant examples -->
  </div>

  <!-- Usage notes -->
  <div layer-name="Usage" style="display: flex; flex-direction: column; gap: 8px;">
    <div style="font-size: 20px; font-weight: 700; color: #121212;">Usage</div>
    <!-- Bullet points -->
  </div>
</div>
```

### Props Table Row

```html
<div style="
  display: flex;
  gap: 0;
  padding: 10px 0;
  border-bottom: 1px solid #eeeeee;
  font-size: 13px;
">
  <div style="width: 140px; font-weight: 600; color: #333333; flex-shrink: 0;">{prop name}</div>
  <div style="width: 160px; color: #8c8c8c; font-family: monospace; flex-shrink: 0;">{type}</div>
  <div style="width: 100px; color: #8c8c8c; flex-shrink: 0;">{default}</div>
  <div style="flex: 1; color: #555555;">{description}</div>
</div>
```

### Visual State Example Row

Show variants or states side by side with labels:

```html
<div style="display: flex; gap: 24px; align-items: flex-end;">
  <div style="display: flex; flex-direction: column; align-items: center; gap: 8px;">
    <!-- Render the actual component visually here using HTML/CSS -->
    <div style="font-size: 10px; color: #999999;">{Label, e.g. "Default"}</div>
  </div>
  <!-- Repeat for each state/variant -->
</div>
```

---

## 9. Critical Rules

1. **One `write_html` per visual group** — write the section container first, then insert children. Never put an entire foundations page in one call.
2. **Verify with `get_screenshot`** after each section write. If content clips, use `update_styles` to set the overflowing dimension to `fit-content`.
3. **Use actual CSS custom properties** (`var(--token-name)`) in inline styles when the project defines them — this makes token values live in the documentation.
4. **Always show both visual + code reference** — every swatch, bar, and card must display the token's code name (CSS variable or JS path) alongside its visual representation.
5. **Use `layer-name` attributes** on all structural elements — this keeps the Paper node tree navigable: `<div layer-name="Section/Colors">`.
6. **Use `duplicate_nodes` for repetitive patterns** — create one swatch card, then duplicate and modify via `update_styles` + `set_text_content` for the rest. Faster and more consistent than writing each from scratch.
7. **Prefer fonts already in the file** — check `get_font_family_info` and match. Only introduce new fonts if the project's type scale requires them.
8. **Fixed-width slots for aligned rows** — when building table-like layouts (props tables, spacing bars with labels), use `flex-shrink: 0` and fixed widths on leading columns so content aligns across rows.
