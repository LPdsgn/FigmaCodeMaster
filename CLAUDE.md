# FigmaCodeMaster - Plugin Rules

This plugin converts Figma designs to code with pixel-perfect fidelity while respecting the existing codebase architecture.

## MCP Server Requirement

This plugin requires the Figma MCP Remote Server. Before using any command, ensure the MCP server is connected:

1. Run `/mcp` in Claude Code to check connection status
2. If not connected, authorize via Figma OAuth when prompted
3. The server URL is: `https://mcp.figma.com/mcp`

## Core Principles

### 1. Design Fidelity First
- ALWAYS extract design context using `get_design_context` before generating code
- Use `get_screenshot` to maintain visual accuracy for complex layouts
- Never approximate values - use exact measurements from Figma

### 2. Codebase Architecture Respect
- ALWAYS analyze the existing codebase before generating code
- Detect framework from `package.json` (React, Vue, Svelte, vanilla)
- Reuse existing components when similarity > 80%
- Follow naming conventions already established in the project
- Use existing design tokens/CSS variables when available

### 3. Design Tokens Priority
- Extract variables using `get_variable_defs` for colors, spacing, typography
- Map Figma variables to CSS custom properties, Tailwind config (v3), or CSS theme (v4)
- Never hardcode values that exist as design tokens
- Report conflicts between Figma tokens and existing codebase tokens

### 4. Component Strategy
- **New**: Create following existing architecture patterns
- **Match (>80%)**: Propose merge with existing component
- **Different**: Create variant with descriptive name (e.g., `button-hero.tsx`)
- **Conflict**: STOP and ask user for decision

## Figma MCP Tools Usage

### Primary Tools

| Tool | When to Use |
|------|-------------|
| `get_design_context` | First step - get structured representation of Figma selection |
| `get_variable_defs` | Extract design tokens (colors, spacing, typography) |
| `get_screenshot` | Visual reference for complex layouts |
| `get_code_connect_map` | Check existing component mappings |
| `get_metadata` | Get layer structure for large designs |

### Best Practices

1. **Be explicit about tools**: If output seems wrong, explicitly request the tool
   - "Get the variable names and values" triggers `get_variable_defs`
   - "Generate code for this frame" triggers `get_design_context`

2. **Handle large designs**: Use `get_metadata` first, then fetch specific nodes

3. **Asset handling**: If MCP returns localhost URLs for images/SVGs, use them directly

## Framework Detection & Mapping

```
package.json detection:
├── next/react    → App Router + shadcn/ui + Tailwind + Radix
├── vue/nuxt      → Nuxt UI + Headless UI + Tailwind
├── svelte/kit    → Skeleton UI + Tailwind
├── astro         → Astro components + Tailwind
└── no framework  → Vanilla HTML + Tailwind + Alpine.js
```

## Output Standards

### File Organization
```
components/
├── ui/           # Base components (Button, Input, Card)
├── blocks/       # Composite components (Hero, Navbar, Footer)
└── [feature]/    # Feature-specific components
```

### Code Quality
- Use TypeScript when project uses TypeScript
- Include proper props typing
- Follow existing ESLint/Prettier configuration
- Add accessibility attributes (ARIA labels, roles)

## Validation Rules

When validating implementations:
- Compare dimensions with ±2px tolerance
- Verify all design tokens are correctly mapped
- Check responsive behavior matches Figma constraints
- Ensure color values match exactly (no approximations)

## What NOT to Do

- Never commit changes automatically
- Never modify files outside the component scope without asking
- Never ignore existing design system in favor of Figma values
- Never skip the codebase analysis step
- Never hardcode colors/spacing when tokens exist
