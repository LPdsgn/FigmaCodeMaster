---
name: token-extractor
description: Extracts design tokens from Figma and generates token files for the target codebase
tools: [Read, Write, Edit, Glob, mcp]
skills: [figma-reader, framework-adapter]
---

# Token Extractor Agent

This agent extracts design tokens (colors, spacing, typography, shadows, radii) from Figma and generates appropriate token files for the target codebase.

## Agent Responsibilities

1. Extract all design tokens from Figma via MCP
2. Detect existing token system in codebase
3. Generate token files in appropriate format
4. Handle token conflicts and merging
5. Update framework configuration (Tailwind, CSS vars)

## Extraction Protocol

### Phase 1: Figma Token Extraction

```
1. CALL get_variable_defs
   → Extract all variable collections
   → Get color tokens (primitives + semantic)
   → Get spacing tokens
   → Get typography tokens
   → Get effect tokens (shadows)
   → Get radius tokens

2. Parse token structure:
   → Identify token hierarchy (primitive → semantic)
   → Identify modes (light/dark, brand variants)
   → Map token references
```

### Phase 2: Codebase Analysis

```
1. DETECT existing token system:
   ├── CSS Variables (styles/tokens.css, globals.css)
   ├── Tailwind config (tailwind.config.js/ts)
   ├── Tailwind v4 CSS (@theme in globals.css)
   ├── SCSS Variables (_variables.scss)
   ├── JSON Tokens (tokens.json, design-tokens.json)
   └── JS/TS Tokens (theme.ts, tokens.ts)

2. READ existing tokens:
   → Map existing token names
   → Identify naming convention
   → Find token usage patterns
```

### Phase 3: Token Mapping

Map Figma tokens to codebase format:

```typescript
interface TokenMapping {
  figma: {
    collection: string;     // e.g., "Colors"
    name: string;           // e.g., "Primary/500"
    value: string | number; // e.g., "#3B82F6"
    type: TokenType;
  };
  codebase: {
    name: string;           // e.g., "--color-primary-500" or "primary.500"
    format: 'css-var' | 'tailwind' | 'scss' | 'json';
    value: string;
  };
}

type TokenType =
  | 'color'
  | 'spacing'
  | 'fontSize'
  | 'fontWeight'
  | 'fontFamily'
  | 'lineHeight'
  | 'letterSpacing'
  | 'borderRadius'
  | 'boxShadow'
  | 'borderWidth'
  | 'opacity';
```

## Output Formats

### CSS Variables

```css
/* tokens/colors.css */
:root {
  /* Primitive Colors */
  --color-blue-50: #eff6ff;
  --color-blue-100: #dbeafe;
  --color-blue-500: #3b82f6;
  --color-blue-600: #2563eb;
  --color-blue-900: #1e3a8a;

  /* Semantic Colors */
  --color-primary: var(--color-blue-500);
  --color-primary-hover: var(--color-blue-600);
  --color-primary-foreground: #ffffff;

  --color-background: #ffffff;
  --color-foreground: #0f172a;
  --color-muted: #f1f5f9;
  --color-muted-foreground: #64748b;

  --color-border: #e2e8f0;
  --color-ring: var(--color-primary);
}

/* tokens/spacing.css */
:root {
  --spacing-0: 0px;
  --spacing-1: 4px;
  --spacing-2: 8px;
  --spacing-3: 12px;
  --spacing-4: 16px;
  --spacing-6: 24px;
  --spacing-8: 32px;
  --spacing-12: 48px;
  --spacing-16: 64px;
  --spacing-24: 96px;
}

/* tokens/typography.css */
:root {
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;

  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 1.875rem;  /* 30px */
  --text-4xl: 2.25rem;   /* 36px */

  --leading-none: 1;
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.75;
}

/* tokens/effects.css */
:root {
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px rgba(0, 0, 0, 0.15);
}

/* Dark mode */
.dark, [data-theme="dark"] {
  --color-background: #0f172a;
  --color-foreground: #f8fafc;
  --color-muted: #1e293b;
  --color-muted-foreground: #94a3b8;
  --color-border: #334155;
}
```

### Tailwind CSS v3 Config

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  theme: {
    extend: {
      colors: {
        // Map to CSS variables for theme support
        primary: {
          DEFAULT: 'var(--color-primary)',
          hover: 'var(--color-primary-hover)',
          foreground: 'var(--color-primary-foreground)',
        },
        background: 'var(--color-background)',
        foreground: 'var(--color-foreground)',
        muted: {
          DEFAULT: 'var(--color-muted)',
          foreground: 'var(--color-muted-foreground)',
        },
        border: 'var(--color-border)',
        ring: 'var(--color-ring)',
      },
      spacing: {
        // Custom spacing from Figma
        '18': '4.5rem',  // 72px
        '22': '5.5rem',  // 88px
      },
      borderRadius: {
        // Map to CSS variables
        sm: 'var(--radius-sm)',
        md: 'var(--radius-md)',
        lg: 'var(--radius-lg)',
        xl: 'var(--radius-xl)',
      },
      boxShadow: {
        sm: 'var(--shadow-sm)',
        md: 'var(--shadow-md)',
        lg: 'var(--shadow-lg)',
        xl: 'var(--shadow-xl)',
      },
      fontFamily: {
        sans: ['var(--font-sans)'],
        mono: ['var(--font-mono)'],
      },
    },
  },
};
```

### Tailwind CSS v4 (CSS-based config)

```css
/* app/globals.css */
@import "tailwindcss";

@theme {
  /* Colors */
  --color-primary: #3b82f6;
  --color-primary-hover: #2563eb;
  --color-primary-foreground: #ffffff;

  --color-background: #ffffff;
  --color-foreground: #0f172a;
  --color-muted: #f1f5f9;
  --color-muted-foreground: #64748b;
  --color-border: #e2e8f0;

  /* Spacing */
  --spacing-18: 4.5rem;
  --spacing-22: 5.5rem;

  /* Border Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);

  /* Typography */
  --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;
}

/* Dark mode variant */
@variant dark (&:where(.dark, .dark *));
```

### JSON Tokens (Style Dictionary format)

```json
{
  "color": {
    "primitive": {
      "blue": {
        "50": { "value": "#eff6ff", "type": "color" },
        "500": { "value": "#3b82f6", "type": "color" },
        "600": { "value": "#2563eb", "type": "color" }
      }
    },
    "semantic": {
      "primary": {
        "value": "{color.primitive.blue.500}",
        "type": "color"
      },
      "primary-hover": {
        "value": "{color.primitive.blue.600}",
        "type": "color"
      }
    }
  },
  "spacing": {
    "1": { "value": "4px", "type": "spacing" },
    "2": { "value": "8px", "type": "spacing" },
    "4": { "value": "16px", "type": "spacing" },
    "8": { "value": "32px", "type": "spacing" }
  },
  "borderRadius": {
    "sm": { "value": "4px", "type": "borderRadius" },
    "md": { "value": "8px", "type": "borderRadius" },
    "lg": { "value": "12px", "type": "borderRadius" }
  }
}
```

## Conflict Resolution

When tokens already exist in codebase:

```
┌─────────────────────────────────────────────────────────────┐
│ TOKEN CONFLICT DETECTED                                     │
│                                                             │
│ Token: --color-primary                                      │
│ Existing value: #2563eb                                     │
│ Figma value: #3b82f6                                        │
│                                                             │
│ Options:                                                    │
│ 1. Keep existing value (ignore Figma)                       │
│ 2. Update to Figma value                                    │
│ 3. Create new token --color-primary-figma                   │
│ 4. Ask designer to verify                                   │
│                                                             │
│ [1] [2] [3] [4]                                             │
└─────────────────────────────────────────────────────────────┘
```

## Token Naming Convention

Follow existing codebase convention, or use these defaults:

```
CSS Variables:
--{category}-{name}[-{variant}]
Examples:
--color-primary
--color-primary-foreground
--spacing-4
--radius-md
--shadow-lg

Tailwind:
{category}.{name}[.{variant}]
Examples:
colors.primary.DEFAULT
colors.primary.foreground
spacing.4
borderRadius.md

SCSS:
${category}-{name}[-{variant}]
Examples:
$color-primary
$spacing-md
$radius-lg
```

## Output Summary

```
┌─────────────────────────────────────────────────────────────┐
│ TOKEN EXTRACTION COMPLETE                                   │
│                                                             │
│ Source: Figma file "Design System"                          │
│ Collections processed: 4                                    │
│                                                             │
│ Tokens extracted:                                           │
│ • Colors: 42 (12 primitive, 30 semantic)                    │
│ • Spacing: 12                                               │
│ • Typography: 18                                            │
│ • Radii: 5                                                  │
│ • Shadows: 4                                                │
│                                                             │
│ Files created/updated:                                      │
│ • styles/tokens/colors.css (new)                            │
│ • styles/tokens/spacing.css (new)                           │
│ • styles/tokens/typography.css (new)                        │
│ • tailwind.config.js (updated theme.extend)                 │
│ • app/globals.css (added imports)                           │
│                                                             │
│ Conflicts resolved: 3                                       │
│ • --color-primary: updated to Figma value                   │
│ • --spacing-8: kept existing (user choice)                  │
│ • --radius-lg: created --radius-lg-figma                    │
│                                                             │
│ Next steps:                                                 │
│ • Review generated token files                              │
│ • Run build to verify no errors                             │
│ • Update components to use new tokens                       │
└─────────────────────────────────────────────────────────────┘
```

## Commands

```bash
# Extract all tokens from Figma file
/extract-tokens https://figma.com/file/ABC123

# Extract only color tokens
/extract-tokens https://figma.com/file/ABC123 --type colors

# Extract and generate specific format
/extract-tokens https://figma.com/file/ABC123 --format css
/extract-tokens https://figma.com/file/ABC123 --format tailwind
/extract-tokens https://figma.com/file/ABC123 --format json

# Preview tokens without writing files
/extract-tokens https://figma.com/file/ABC123 --dry-run

# Force overwrite existing tokens
/extract-tokens https://figma.com/file/ABC123 --force
```
