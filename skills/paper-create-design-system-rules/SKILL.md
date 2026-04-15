---
name: paper-create-design-system-rules
description: "Generates custom design system rules for the user's codebase tailored to Paper-to-code workflows. Use when user says 'create design system rules', 'generate rules for my project', 'set up design rules', 'customize design system guidelines', or wants to establish project-specific conventions for implementing Paper designs as code."
disable-model-invocation: false
---

# Create Design System Rules for Paper

## Overview

This skill helps you generate custom design system rules tailored to your project's specific needs. These rules guide AI coding agents to produce consistent, high-quality code when implementing Paper designs, ensuring that your team's conventions, component patterns, and architectural decisions are followed automatically.

### Supported Rule Files

| Agent | Rule File |
|-------|-----------|
| Claude Code | `CLAUDE.md` |
| Codex CLI | `AGENTS.md` |
| Cursor | `.cursor/rules/paper-design-system.mdc` |

## What Are Design System Rules?

Design system rules are project-level instructions that encode the "unwritten knowledge" of your codebase — the kind of expertise that experienced developers know and would pass on to new team members:

- Which layout primitives and components to use
- Where component files should be located
- How components should be named and structured
- What should never be hardcoded
- How to handle design tokens and styling
- Project-specific architectural patterns

Once defined, these rules dramatically reduce repetitive prompting and ensure consistent output across all Paper implementation tasks.

---

## Skill Boundaries

- Use this skill when the deliverable is a **rules file** (CLAUDE.md, AGENTS.md, or .cursor/rules/).
- If the user wants to **implement code from a Paper design**, switch to [paper-implement-design](../paper-implement-design/SKILL.md).
- If the user wants to **build screens in Paper**, switch to [paper-generate-design](../paper-generate-design/SKILL.md).
- If the user wants to build a **design system library** in Paper, switch to [paper-generate-library](../paper-generate-library/SKILL.md).

---

## Prerequisites

Before any Paper tool call in a session:

1. **Load the Paper guide**: `get_guide({ topic: "paper-mcp-instructions" })` — mandatory, once per session.

Additional requirements:

- Paper MCP server must be connected and accessible.
- Access to the project codebase for analysis.
- Understanding of your team's component conventions (or willingness to establish them).

---

## When to Use This Skill

Use this skill when:

- Starting a new project that will use Paper designs
- Onboarding an AI coding agent to an existing project with established patterns
- Standardizing Paper-to-code workflows across your team
- Updating or refining existing design system conventions
- Users explicitly request: "create design system rules", "set up Paper guidelines", "customize rules for my project"

---

## Required Workflow

**Follow these steps in order. Do not skip steps.**

### Step 1: Analyze the Codebase

Analyze the project to understand existing patterns:

**Component Organization:**

- Where are UI components located? (e.g., `src/components/`, `app/ui/`, `lib/components/`)
- Is there a dedicated design system directory?
- How are components organized? (by feature, by type, flat structure)

**Styling Approach:**

- What CSS framework or approach is used? (Tailwind, CSS Modules, styled-components, etc.)
- Where are design tokens defined? (CSS variables, theme files, config files)
- Are there existing color, typography, or spacing tokens?

**Component Patterns:**

- What naming conventions are used? (PascalCase, kebab-case, prefixes)
- How are component props typically structured?
- Are there common composition patterns?

**Architecture Decisions:**

- How is state management handled?
- What routing system is used?
- Are there specific import patterns or path aliases?

### Step 2: Inspect the Paper File (Optional)

If the user has an existing Paper file with designs:

1. Call `get_basic_info` to understand the file structure, artboards, and conventions.
2. Call `get_tree_summary` on key artboards to understand the design structure.
3. Note how designs are organized — this informs how rules should map Paper elements to code.

### Step 3: Generate Project-Specific Rules

Based on your codebase analysis, create a comprehensive set of rules covering these categories:

#### General Component Rules

```markdown
- IMPORTANT: Always use components from `[YOUR_PATH]` when possible
- Place new UI components in `[COMPONENT_DIRECTORY]`
- Follow `[NAMING_CONVENTION]` for component names
- Components must export as `[EXPORT_PATTERN]`
```

#### Styling Rules

```markdown
- Use `[CSS_FRAMEWORK/APPROACH]` for styling
- Design tokens are defined in `[TOKEN_LOCATION]`
- IMPORTANT: Never hardcode colors - always use tokens from `[TOKEN_FILE]`
- Spacing values must use the `[SPACING_SYSTEM]` scale
- Typography follows the scale defined in `[TYPOGRAPHY_LOCATION]`
```

#### Paper MCP Integration Rules

```markdown
## Paper MCP Integration Rules

These rules define how to translate Paper designs into code for this project and must be followed for every Paper-driven change.

### Required Flow (do not skip)

1. Use get_selection to identify the target node, or accept a node ID from the user
2. Run get_jsx to get a JSX representation of the design (use "tailwind" or "inline-styles" format)
3. Run get_computed_styles for exact CSS values on key nodes — do NOT read sizes or colors from screenshots alone
4. Run get_screenshot for a visual reference of the design being implemented
5. If the design contains images, use get_fill_image to extract them
6. Only after you have design context and screenshot, start implementation
7. Translate the output into this project's conventions, styles, and framework
8. Validate against Paper screenshot for 1:1 look and behavior before marking complete

### Implementation Rules

- Treat the Paper get_jsx output as a representation of design, not as final code style
- Replace generic styling with `[YOUR_STYLING_APPROACH]` when applicable
- Reuse existing components from `[COMPONENT_PATH]` instead of duplicating functionality
- Use the project's color system, typography scale, and spacing tokens consistently
- Respect existing routing, state management, and data-fetch patterns
- Use get_computed_styles for exact values — never guess from screenshots
- Strive for 1:1 visual parity with the Paper design
```

#### Asset Handling Rules

```markdown
## Asset Handling

- Use get_fill_image from the Paper MCP server to extract images from designs
- IMPORTANT: DO NOT import/add new icon packages - extract assets from Paper
- IMPORTANT: DO NOT use placeholder images when actual assets can be extracted
- Store downloaded assets in `[ASSET_DIRECTORY]`
```

#### Project-Specific Conventions

```markdown
## Project-Specific Conventions

- [Add any unique architectural patterns]
- [Add any special import requirements]
- [Add any testing requirements]
- [Add any accessibility standards]
- [Add any performance considerations]
```

### Step 4: Save Rules to the Appropriate Rule File

Detect which AI coding agent the user is working with and save the generated rules to the corresponding file:

| Agent | Rule File | Notes |
|-------|-----------|-------|
| Claude Code | `CLAUDE.md` in project root | Markdown format. Can also use `.claude/rules/paper-design-system.md` for modular organization. |
| Codex CLI | `AGENTS.md` in project root | Markdown format. Append as a new section if file already exists. 32 KiB combined size limit. |
| Cursor | `.cursor/rules/paper-design-system.mdc` | Markdown with YAML frontmatter (`description`, `globs`, `alwaysApply`). |

If unsure which agent the user is working with, check for existing rule files in the project or ask the user.

For Cursor, wrap the rules with YAML frontmatter:

```markdown
---
description: Rules for implementing Paper designs using the Paper MCP server. Covers component organization, styling conventions, design tokens, asset handling, and the required Paper-to-code workflow.
globs: "src/components/**"
alwaysApply: false
---

[Generated rules here]
```

Customize the `globs` pattern to match the directories where Paper-derived code will live in the project (e.g., `"src/**/*.tsx"` or `["src/components/**", "src/pages/**"]`).

After saving, the rules will be automatically loaded by the agent and applied to all Paper implementation tasks.

### Step 5: Validate and Iterate

After creating rules:

1. Test with a simple Paper design implementation.
2. Verify the agent follows the rules correctly.
3. Refine any rules that aren't working as expected.
4. Share with team members for feedback.
5. Update rules as the project evolves.

---

## Rule Categories and Examples

### Essential Rules (Always Include)

**Component Discovery:**

```markdown
- UI components are located in `src/components/ui/`
- Feature components are in `src/components/features/`
- Layout primitives are in `src/components/layout/`
```

**Design Token Usage:**

```markdown
- Colors are defined as CSS variables in `src/styles/tokens.css`
- Never hardcode hex colors - use `var(--color-*)` tokens
- Spacing uses the 4px base scale: `--space-1` (4px), `--space-2` (8px), etc.
```

**Styling Approach:**

```markdown
- Use Tailwind utility classes for styling
- Custom styles go in component-level CSS modules
- Theme customization is in `tailwind.config.js`
```

### Recommended Rules (Highly Valuable)

**Component Patterns:**

```markdown
- All components must accept a `className` prop for composition
- Variant props should use union types: `variant: 'primary' | 'secondary'`
- Icon components should accept `size` and `color` props
```

**Import Conventions:**

```markdown
- Use path aliases: `@/components`, `@/styles`, `@/utils`
- Group imports: React, third-party, internal, types
- No relative imports beyond parent directory
```

### Optional Rules (Project-Specific)

**Accessibility:**

```markdown
- All interactive elements must have aria-labels
- Color contrast must meet WCAG AA standards
- Keyboard navigation required for all interactions
```

**Performance:**

```markdown
- Lazy load images with the `Image` component from `@/components/Image`
- Use React.memo for components that receive complex props
- Icons should be SVG components, not icon fonts
```

---

## Examples

### Example 1: React + Tailwind Project

User says: "Create design system rules for my React project"

**Actions:**

1. Analyze codebase structure
2. Inspect Paper file if available (`get_basic_info`, `get_tree_summary`)
3. Generate rules:

```markdown
# Paper MCP Integration Rules

## Component Organization

- UI components are in `src/components/ui/`
- Page components are in `src/app/`
- Use Tailwind for styling

## Paper Implementation Flow

1. Use get_selection or accept node ID from user
2. Run get_jsx for JSX representation
3. Run get_computed_styles for exact values
4. Run get_screenshot for visual reference
5. Map design colors to Tailwind colors defined in `tailwind.config.js`
6. Reuse components from `src/components/ui/` when possible
7. Validate against screenshot before completing

## Styling Rules

- IMPORTANT: Use Tailwind utility classes, not inline styles
- Colors are defined in `tailwind.config.js` theme.colors
- Spacing uses Tailwind's default scale
- Custom components go in `src/components/ui/`

## Asset Rules

- IMPORTANT: Use get_fill_image to extract images from Paper
- Store static assets in `public/assets/`
- DO NOT install new icon libraries
```

4. Save to the appropriate rule file for the user's agent
5. Test with a simple button implementation

### Example 2: Vue + Custom CSS Project

User says: "Set up Paper rules for my Vue app"

**Actions:**

1. Discover that project uses CSS modules and a custom design token system
2. Generate rules:

```markdown
# Paper MCP Integration Rules

## Component Organization

- Components are in `src/components/`
- Composables are in `src/composables/`
- Vue SFC structure: <script setup>, <template>, <style scoped>

## Design Tokens

- IMPORTANT: All colors defined in `src/styles/tokens.css` as CSS variables
- Use `var(--color-primary)`, `var(--color-secondary)`, etc.
- Spacing: `var(--space-xs)` through `var(--space-xl)`
- Typography: `var(--text-sm)` through `var(--text-2xl)`

## Paper Implementation Flow

1. Get node ID from get_selection or user
2. Run get_jsx and get_computed_styles for design context
3. Run get_screenshot for visual reference
4. Translate JSX output to Vue 3 Composition API
5. Map design colors to CSS variables in `src/styles/tokens.css`
6. Use CSS Modules for component styles
7. Check for existing components in `src/components/` before creating new ones

## Styling Rules

- Use CSS Modules (`.module.css` files)
- IMPORTANT: Reference design tokens, never hardcode values
- Scoped styles with CSS modules
```

3. Save to the appropriate rule file for the user's agent
4. Validate with a card component

---

## Best Practices

### Start Simple, Iterate

Don't try to capture every rule upfront. Start with the most important conventions and add rules as you encounter inconsistencies.

### Be Specific

Instead of: "Use the design system"
Write: "Always use Button components from `src/components/ui/Button.tsx` with variant prop ('primary' | 'secondary' | 'ghost')"

### Make Rules Actionable

Each rule should tell the agent exactly what to do, not just what to avoid.

Good: "Colors are defined in `src/theme/colors.ts` — import and use these constants"
Bad: "Don't hardcode colors"

### Use IMPORTANT for Critical Rules

Prefix rules that must never be violated with "IMPORTANT:" to ensure the agent prioritizes them.

### Document the Why

When rules seem arbitrary, explain the reasoning:

```markdown
- Place all data-fetching in server components (reduces client bundle size and improves performance)
- Use absolute imports with `@/` alias (makes refactoring easier and prevents broken relative paths)
```

### Review Periodically

Schedule periodic rule reviews (monthly or quarterly). Update rules when architectural decisions change. Version control your rule files.

---

## Common Issues and Solutions

### Issue: The agent isn't following the rules

**Cause:** Rules may be too vague or not properly loaded by the agent.
**Solution:**

- Make rules more specific and actionable
- Verify rules are saved in the correct configuration file
- Restart your agent or IDE to reload rules
- Add "IMPORTANT:" prefix to critical rules

### Issue: Rules conflict with each other

**Cause:** Contradictory or overlapping rules.
**Solution:**

- Review all rules for conflicts
- Establish a clear priority hierarchy
- Remove redundant rules
- Consolidate related rules into single, clear statements

### Issue: Too many rules increase latency

**Cause:** Excessive rules increase context size and processing time.
**Solution:**

- Focus on the 20% of rules that solve 80% of consistency issues
- Remove overly specific rules that rarely apply
- Combine related rules
- Use progressive disclosure (basic rules first, advanced rules in linked files)

### Issue: Rules become outdated as project evolves

**Cause:** Codebase changes but rules don't.
**Solution:**

- Schedule periodic rule reviews
- Update rules when architectural decisions change
- Version control your rule files
- Document rule changes in commit messages
