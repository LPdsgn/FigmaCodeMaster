# FigmaCodeMaster

A Claude Code plugin that converts Figma designs to production-ready code with pixel-perfect fidelity and full respect for your existing codebase architecture.

## Features

- **Design-to-Code Conversion**: Convert Figma frames to React, Vue, Svelte, or vanilla HTML
- **Multi-Framework Support**: Automatic detection of Next.js, Nuxt, SvelteKit, Astro, and more
- **Design Token Extraction**: Extract colors, spacing, typography, shadows as CSS variables or Tailwind config
- **Codebase-Aware**: Respects existing component patterns, naming conventions, and design systems
- **Validation**: Compare implemented code against Figma for pixel-perfect accuracy
- **Smart Merging**: Intelligently merge with existing components or create variants

## Prerequisites

- [Claude Code](https://claude.com/claude-code) CLI installed
- Figma account with access to the designs you want to convert

## Installation & Configuration (Permanent Installation)

Follow these steps to install the plugin and configure it for use across all your projects.

### 1. Install the Plugin

Install the plugin globally using Claude Code's plugin manager. You can install it from a local directory or a Git repository.

**From a Local Directory (if distributed via shared drive/repo):**
```bash
# Replace with the actual path to the plugin folder
/plugin install /path/to/FigmaCodeMaster
```

**Verify Installation:**
Run `/plugin list` to confirm `figma-code-master` is active.

### 2. Configure Figma MCP Server (Required)

This plugin relies on the Figma Model Context Protocol (MCP) server. Each team member must configure this once on their machine.

**Step A: Add the Server**
Run the following command in your terminal to register the Figma MCP server with Claude Code:

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

**Step B: Authenticate**
1. Open Claude Code (`claude`).
2. Type `/mcp` and press Enter.
3. Locate **figma** in the list.
4. Select **Authenticate** (or ensure status is Connected).
5. Follow the browser prompt to authorize Figma access.

**Step C: Verify Connection**
Run `/doctor` or check `/mcp` again. You should see a green indicator next to "figma".

### 3. Usage in Projects

Once installed and configured, the plugin is available globally. You can use it in any project directory.

**Example Workflow:**
1. Navigate to your project folder: `cd my-nextjs-app`
2. Start Claude: `claude`
3. Convert a design: `/figma-code-master:design-to-code https://figma.com/design/...`

> **Note**: If you have a local version of the plugin in the current directory (e.g. for development), you can override the global installation by running `claude --plugin-dir .`

## Quick Start

### 1. Connect to Figma MCP

Run `/mcp` in Claude Code and authorize access to Figma when prompted.

### 2. Convert a Design

```bash
# Convert a Figma frame to code
/design-to-code https://figma.com/design/ABC123?node-id=1-234

# Or specify a frame by name
/design-to-code https://figma.com/file/ABC123 "Hero Section"
```

### 3. Extract Design Tokens

```bash
# Extract all design tokens from a Figma file
/extract-tokens https://figma.com/file/ABC123

# Extract only colors
/extract-tokens https://figma.com/file/ABC123 --type colors
```

### 4. Validate Implementation

```bash
# Validate a component against its Figma source
/validate-design components/ui/HeroSection.tsx
```

## Commands

| Command | Description |
|---------|-------------|
| `/design-to-code` | Convert Figma designs to code |
| `/extract-tokens` | Extract design tokens from Figma |
| `/validate-design` | Validate implementation vs Figma |

## Supported Frameworks

The plugin auto-detects your framework from `package.json`:

| Framework | UI Library | Styling |
|-----------|------------|---------|
| Next.js | shadcn/ui, Radix | Tailwind CSS |
| React | Radix, Headless UI | Tailwind CSS, CSS Modules |
| Vue/Nuxt | Nuxt UI, Headless UI | Tailwind CSS |
| Svelte/Kit | Skeleton UI, Melt UI | Tailwind CSS |
| Astro | Any (islands) | Tailwind CSS |
| Vanilla | Alpine.js (optional) | Tailwind CSS, Plain CSS |

## Workflow

```
                    +------------------+
                    | /design-to-code  |
                    +--------+---------+
                             |
              +--------------+--------------+
              |                             |
     +--------v--------+          +---------v--------+
     | Analyze Figma   |          | Analyze Codebase |
     | (via MCP)       |          | (framework, etc) |
     +--------+--------+          +---------+--------+
              |                             |
              +--------------+--------------+
                             |
                    +--------v--------+
                    | Component       |
                    | Strategy        |
                    +--------+--------+
                             |
         +-------------------+-------------------+
         |                   |                   |
+--------v------+   +--------v------+   +--------v------+
| New Component |   | Merge with    |   | Create        |
| (no match)    |   | Existing      |   | Variant       |
+-------+-------+   +-------+-------+   +-------+-------+
         |                   |                   |
         +-------------------+-------------------+
                             |
                    +--------v--------+
                    | Generate Code   |
                    | + Tokens        |
                    +--------+--------+
                             |
                    +--------v--------+
                    | Offer           |
                    | Validation      |
                    +-----------------+
```

## Configuration

### MCP Server

The plugin uses Figma's official MCP Remote Server:

```json
// .mcp.json (auto-generated)
{
  "mcpServers": {
    "figma": {
      "url": "https://mcp.figma.com/mcp",
      "type": "http"
    }
  }
}
```

### Plugin Settings

Settings are configured via `CLAUDE.md` in the plugin directory. Key settings:

- Design token naming conventions
- Component output directories
- Validation tolerance (default: 2px)
- Framework preferences

## Figma MCP Tools Used

| Tool | Purpose |
|------|---------|
| `get_design_context` | Extract structured design representation |
| `get_variable_defs` | Extract design tokens (variables) |
| `get_screenshot` | Visual reference for complex layouts |
| `get_metadata` | Layer structure for large designs |
| `get_code_connect_map` | Check existing component mappings |

## Best Practices

### In Figma

1. **Use Variables**: Define colors, spacing, and typography as Figma Variables for clean token extraction
2. **Name Frames**: Give frames descriptive names that map to component names
3. **Use Auto Layout**: Auto Layout converts cleanly to Flexbox/Grid
4. **Component Variants**: Define variants in Figma for automatic variant props

### In Your Codebase

1. **Consistent Structure**: Keep components in a predictable location (`components/ui/`, `src/components/`)
2. **Use Design Tokens**: Define CSS variables or Tailwind config for colors/spacing
3. **Type Components**: Use TypeScript for better prop inference

## Validation Scoring

When running `/validate-design`, components are scored on:

| Category | Weight | Tolerance |
|----------|--------|-----------|
| Dimensions | 25% | ±2px pass, ±5px warn |
| Colors | 25% | Exact match required |
| Typography | 20% | Font-size ±1px |
| Spacing | 20% | ±2px pass |
| Token Usage | 10% | Must use tokens |

## Troubleshooting

### MCP Connection Issues

```bash
# Check MCP status
/mcp

# Reconnect if needed
# Follow OAuth prompts
```

### Frame Not Found

Ensure the Figma URL includes the node-id parameter:
```
https://figma.com/design/FILE_ID?node-id=NODE_ID
```

### Token Conflicts

When Figma tokens conflict with existing codebase tokens, you'll be prompted to:
1. Update to Figma value
2. Keep existing value
3. Create a new token

## Contributing

Contributions are welcome. Please open an issue first to discuss proposed changes.

## License

MIT License - see [LICENSE](LICENSE) for details.

---

Built with Claude Code by [Boosha AI](https://boosha.ai)
