---
name: extract-tokens
description: Extracts design tokens from Figma and generates token files for the codebase
args:
  - name: figma-url
    description: URL to Figma file or frame containing design tokens
    required: true
  - name: type
    description: Token types to extract (colors, spacing, typography, radii, shadows, all)
    required: false
    default: all
  - name: format
    description: Output format (css, tailwind, json, scss)
    required: false
allowed_tools: [Read, Write, Edit, Glob, mcp, AskUserQuestion]
---

# /extract-tokens

Extracts design tokens (colors, spacing, typography, shadows, radii) from Figma and generates appropriate token files for your codebase.

## Usage

```bash
# Extract all tokens from a Figma file
/extract-tokens https://figma.com/file/ABC123

# Extract only color tokens
/extract-tokens https://figma.com/file/ABC123 --type colors

# Extract multiple token types
/extract-tokens https://figma.com/file/ABC123 --type colors,spacing,typography

# Extract and output as specific format
/extract-tokens https://figma.com/file/ABC123 --format css
/extract-tokens https://figma.com/file/ABC123 --format tailwind
/extract-tokens https://figma.com/file/ABC123 --format json
```

## Workflow

### Step 1: Validate MCP Connection

```
IF MCP not connected:
  -> Prompt user: "Figma MCP not connected. Run /mcp to authenticate."
  -> STOP
```

### Step 2: Extract Tokens from Figma

```
CALL get_variable_defs(figma-url)
  -> Extract all variable collections
  -> Parse token types:
     - Colors (primitives + semantic)
     - Spacing
     - Typography (font families, sizes, weights, line heights)
     - Border radius
     - Shadows
     - Other effects
```

### Step 3: Analyze Existing Token System

```
1. DETECT existing format in codebase:
   - CSS Variables: styles/tokens.css, globals.css, :root declarations
   - Tailwind: tailwind.config.js/ts theme extensions
   - Tailwind v4: @theme declarations in CSS
   - SCSS: _variables.scss, _tokens.scss
   - JSON: tokens.json, design-tokens.json

2. READ existing tokens:
   - Map current token names
   - Identify naming convention used
   - Find potential conflicts
```

### Step 4: Generate Token Files

Based on detected or specified format:

#### CSS Variables Output
```css
/* tokens/colors.css */
:root {
  /* Primitive Colors */
  --color-blue-50: #eff6ff;
  --color-blue-500: #3b82f6;

  /* Semantic Colors */
  --color-primary: var(--color-blue-500);
  --color-background: #ffffff;
  --color-foreground: #0f172a;
}

/* Dark mode */
.dark {
  --color-background: #0f172a;
  --color-foreground: #f8fafc;
}
```

#### Tailwind Config Output
```javascript
// tailwind.config.js additions
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: 'var(--color-primary)',
        background: 'var(--color-background)',
        foreground: 'var(--color-foreground)',
      },
      spacing: {
        '18': '4.5rem',
      },
      borderRadius: {
        'card': 'var(--radius-card)',
      },
    },
  },
};
```

#### JSON Output (Style Dictionary format)
```json
{
  "color": {
    "primary": {
      "value": "#3b82f6",
      "type": "color"
    }
  },
  "spacing": {
    "md": {
      "value": "16px",
      "type": "spacing"
    }
  }
}
```

### Step 5: Handle Conflicts

```
IF token already exists with different value:
  -> Show conflict to user
  -> Ask for resolution:
     1. Update to Figma value
     2. Keep existing value
     3. Create new token with suffix
```

### Step 6: Write Files

```
1. Create/update token files
2. Update imports in globals.css (if CSS)
3. Update tailwind.config.js (if Tailwind)
4. Generate summary report
```

## Options

```bash
# Preview tokens without writing (dry run)
/extract-tokens <url> --dry-run

# Force overwrite existing tokens without asking
/extract-tokens <url> --force

# Specify output directory
/extract-tokens <url> --output styles/tokens

# Include dark mode tokens
/extract-tokens <url> --include-modes

# Only show new tokens (exclude existing)
/extract-tokens <url> --new-only
```

## Token Types

| Type | Description | Example Output |
|------|-------------|----------------|
| `colors` | Color variables (primitive + semantic) | `--color-primary: #3b82f6` |
| `spacing` | Spacing/sizing values | `--spacing-md: 16px` |
| `typography` | Font families, sizes, weights | `--font-sans: 'Inter'` |
| `radii` | Border radius values | `--radius-md: 8px` |
| `shadows` | Box shadow definitions | `--shadow-md: 0 4px 6px...` |
| `all` | All of the above | Full token set |

## Output Summary

```
+-------------------------------------------------------------+
| TOKEN EXTRACTION COMPLETE                                   |
|                                                             |
| Source: Figma "Design System" file                          |
| Format: CSS Variables + Tailwind                            |
|                                                             |
| Extracted:                                                  |
| - Colors: 42 tokens (12 primitive, 30 semantic)             |
| - Spacing: 12 tokens                                        |
| - Typography: 18 tokens                                     |
| - Radii: 5 tokens                                           |
| - Shadows: 4 tokens                                         |
|                                                             |
| Files created:                                              |
| - styles/tokens/colors.css                                  |
| - styles/tokens/spacing.css                                 |
| - styles/tokens/typography.css                              |
| - styles/tokens/effects.css                                 |
|                                                             |
| Files updated:                                              |
| - tailwind.config.js                                        |
| - app/globals.css (added imports)                           |
|                                                             |
| Conflicts resolved: 2                                       |
| - --color-primary: updated to Figma value                   |
| - --spacing-lg: kept existing (user choice)                 |
+-------------------------------------------------------------+
```

## Examples

### Extract All Tokens
```bash
/extract-tokens https://figma.com/file/ABC123
```

### Colors Only with Preview
```bash
/extract-tokens https://figma.com/file/ABC123 --type colors --dry-run
```

### Full Design System with Dark Mode
```bash
/extract-tokens https://figma.com/file/ABC123 --include-modes
```

## Error Handling

| Error | Resolution |
|-------|------------|
| MCP not connected | Prompt to run /mcp |
| No variables found | Check if Figma file has Variables defined |
| Invalid URL | Ask for correct Figma file URL |
| Write error | Check file permissions |

## Notes

- This command only extracts tokens, not components
- For full design-to-code conversion, use `/design-to-code`
- Tokens are extracted from Figma Variables (not just styles)
- Dark mode tokens require Variables with multiple modes in Figma
