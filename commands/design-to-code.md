---
name: design-to-code
description: Converts Figma designs to production-ready code respecting codebase architecture
args:
  - name: figma-url
    description: URL to Figma frame, component, or file
    required: false
  - name: frame-name
    description: Name of the frame to convert (if URL points to a file)
    required: false
allowed_tools: [Read, Write, Edit, Glob, Grep, Bash, mcp, Task, AskUserQuestion]
---

# /design-to-code

Converts Figma designs into production-ready code that integrates seamlessly with your existing codebase architecture.

## Usage

```bash
# Convert a specific frame by URL
/design-to-code https://figma.com/design/ABC123?node-id=1-234

# Convert a frame by name from a Figma file
/design-to-code https://figma.com/file/ABC123 "Hero Section"

# Use the currently connected Figma selection
/design-to-code "Hero Section"
```

## Workflow

### Step 1: Validate MCP Connection

Check if Figma MCP is connected and authenticated.

```
IF MCP not connected:
  -> Prompt user: "Figma MCP not connected. Run /mcp to authenticate."
  -> STOP
```

### Step 2: Parse Input

```
IF figma-url provided:
  -> Extract file ID and node ID from URL
  -> IF node-id missing AND frame-name provided:
    -> Use get_metadata to find frame by name
    -> Extract node ID

IF only frame-name provided:
  -> Assume MCP has active file context
  -> Search for frame by name
```

### Step 3: Extract Design Data

Delegate to figma-reader skill:

```
1. CALL get_design_context(nodeId)
   -> Get structured design representation
   -> Receive suggested React + Tailwind code

2. CALL get_variable_defs(nodeId)
   -> Extract design tokens used in this frame
   -> Colors, spacing, typography, effects

3. CALL get_screenshot(nodeId) [if complex layout]
   -> Visual reference for accuracy
```

### Step 4: Analyze Codebase

Delegate to framework-adapter skill:

```
1. READ package.json
   -> Detect framework: next | react | vue | svelte | astro | vanilla

2. SCAN existing components
   -> Find component directory structure
   -> Identify naming conventions
   -> Detect existing UI library (shadcn, radix, headless, etc.)

3. READ style configuration
   -> tailwind.config.js/ts
   -> CSS/SCSS structure
   -> Existing design tokens
```

### Step 5: Component Strategy

Check if similar components exist:

```
FOR each component in design:
  1. Search codebase for similar components
     -> By name similarity
     -> By structure similarity
     -> Via get_code_connect_map

  2. Calculate similarity score (0-100%)

  3. DECIDE action:
     |-- No match -> Create new component
     |-- >80% match -> Propose merge/extend existing
     |-- <80% match -> Create variant with new name
     |-- Token conflict -> STOP and ask user
```

### Step 6: Generate Code

Delegate to design-implementer agent:

```
1. Generate component code
   -> Match framework syntax
   -> Use existing patterns
   -> Include TypeScript types (if TS project)
   -> Add accessibility attributes

2. Generate/update design tokens
   -> Add new CSS variables
   -> Update Tailwind config
   -> Maintain token hierarchy

3. Handle assets
   -> Download images from MCP
   -> Save SVG icons
   -> Update imports
```

### Step 7: Write Files

Create or update files:

```
1. Component file(s)
   -> Place in correct directory
   -> Follow naming convention

2. Token files (if needed)
   -> CSS variables
   -> Tailwind config updates

3. Index/export files (if needed)
   -> Update barrel exports
```

### Step 8: Summary & Next Steps

```
+-------------------------------------------------------------+
| IMPLEMENTATION COMPLETE                                     |
|                                                             |
| Design: "Hero Section" from Figma                           |
| Framework: Next.js (App Router) + shadcn/ui                 |
|                                                             |
| Created:                                                    |
| - components/blocks/HeroSection.tsx                         |
| - components/ui/gradient-button.tsx                         |
|                                                             |
| Updated:                                                    |
| - tailwind.config.ts (added spacing-hero: 120px)            |
| - app/globals.css (added --color-accent)                    |
|                                                             |
| Reused:                                                     |
| - Button (existing shadcn component)                        |
| - Container (existing layout component)                     |
|                                                             |
| Would you like to validate the implementation?              |
| [Yes, run /validate-design] [No, I'll review manually]      |
+-------------------------------------------------------------+
```

## Decision Points

The command will pause and ask for user input in these scenarios:

### 1. Component Merge Decision
```
Found similar component: Button.tsx (87% match)

Options:
1. Extend existing Button with new variants from Figma
2. Create new ButtonHero.tsx component
3. Skip - use existing Button as-is

[1] [2] [3]
```

### 2. Token Conflict
```
Design token conflict detected:

--color-primary
  Existing: #2563eb
  Figma: #3b82f6

Options:
1. Update to Figma value
2. Keep existing value
3. Create --color-primary-new for Figma value

[1] [2] [3]
```

### 3. Multiple Frames Detected
```
Multiple frames found matching "Hero":
1. Hero Section (Desktop)
2. Hero Section (Mobile)
3. Hero Banner

Which frame(s) to convert?
[1] [2] [3] [All]
```

## Options

```bash
# Skip validation prompt at the end
/design-to-code <url> --skip-validation

# Auto-create new components (don't merge with existing)
/design-to-code <url> --no-merge

# Specify target directory
/design-to-code <url> --output components/landing

# Include only specific token types
/design-to-code <url> --tokens colors,spacing

# Dry run - show what would be created without writing files
/design-to-code <url> --dry-run
```

## Error Handling

| Error | Resolution |
|-------|------------|
| MCP not connected | Prompt to run /mcp and authenticate |
| Invalid Figma URL | Ask for valid URL with node-id |
| Frame not found | Show available frames, ask to select |
| No framework detected | Use vanilla HTML + Tailwind |
| Write permission denied | Check file permissions |
| Token file locked | Warn and skip token updates |

## Examples

### Basic Usage
```bash
# Convert Hero section from Figma
/design-to-code https://figma.com/design/ABC123?node-id=1-234
```

### With Frame Name
```bash
# Find and convert "Pricing Card" frame
/design-to-code https://figma.com/file/ABC123 "Pricing Card"
```

### Multiple Components
```bash
# Convert entire page with multiple components
/design-to-code https://figma.com/design/ABC123?node-id=1-500 --output app/landing
```

## Integration Notes

- This command never commits changes automatically
- Use `/commit` separately after reviewing generated code
- For token-only extraction, use `/extract-tokens` instead
- For validation only, use `/validate-design` after implementation
