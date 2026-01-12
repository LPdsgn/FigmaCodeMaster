---
name: validate-design
description: Validates implemented code against original Figma design for pixel-perfect accuracy
args:
  - name: component-path
    description: Path to the component file to validate
    required: false
  - name: figma-url
    description: Figma URL of the original design (optional if component was created via /design-to-code)
    required: false
allowed_tools: [Read, Glob, Grep, Bash, mcp, AskUserQuestion]
---

# /validate-design

Compares implemented code against the original Figma design to verify pixel-perfect accuracy and design token compliance.

## Usage

```bash
# Validate a specific component
/validate-design components/ui/HeroSection.tsx

# Validate with explicit Figma reference
/validate-design components/ui/Card.tsx https://figma.com/design/ABC123?node-id=1-234

# Full validation with visual comparison
/validate-design components/ui/Button.tsx --visual

# Validate with auto-fix suggestions
/validate-design components/ui/Button.tsx --suggest-fixes

# Validate all components from a Figma file
/validate-design --figma-file https://figma.com/file/ABC123
```

## Workflow

### Step 1: Validate MCP Connection

```
IF MCP not connected:
  -> Prompt user: "Figma MCP not connected. Run /mcp to authenticate."
  -> STOP
```

### Step 2: Gather Design Source

```
IF figma-url provided:
  -> Use provided URL

ELSE IF component has Code Connect mapping:
  -> CALL get_code_connect_map
  -> Find linked Figma node

ELSE:
  -> Search by component name in connected Figma file
  -> Ask user to confirm or provide URL
```

### Step 3: Extract Design Specifications

```
1. CALL get_design_context(nodeId)
   -> Get expected structure and properties
   -> Extract dimensions, colors, spacing

2. CALL get_variable_defs(nodeId)
   -> Get design tokens used
   -> Map to expected values

3. CALL get_screenshot(nodeId)
   -> Get visual reference
```

### Step 4: Analyze Implementation

```
1. READ component file
   -> Parse className values
   -> Extract Tailwind classes
   -> Find inline styles
   -> Identify CSS variable usage

2. READ related CSS/style files
   -> Map classes to computed values
   -> Resolve CSS variables

3. BUILD comparison model
   -> Expected vs Actual for each property
```

### Step 5: Run Comparison Checks

#### Dimensional Check
```
For each dimension (width, height, padding, margin, gap):
  - Expected: value from Figma
  - Actual: value from code
  - Delta: difference
  - Status: pass (<=2px) | warn (3-5px) | fail (>5px)
```

#### Color Check
```
For each color property:
  - Expected: Figma color value
  - Actual: code color value
  - Uses token: yes/no
  - Correct token: yes/no
  - Status: pass | fail
```

#### Typography Check
```
For each text element:
  - font-family: match?
  - font-size: delta
  - font-weight: match?
  - line-height: delta
  - letter-spacing: delta
```

#### Spacing Check
```
For each spacing value:
  - Location: which element/property
  - Expected vs Actual
  - Uses token: yes/no
```

#### Token Audit
```
- Count tokens used correctly
- Find hardcoded values
- Identify missing token mappings
```

### Step 6: Generate Report

```
+-------------------------------------------------------------+
| DESIGN VALIDATION REPORT                                    |
|                                                             |
| Component: HeroSection.tsx                                  |
| Figma Frame: "Hero Section"                                 |
| Overall Score: 92/100                                       |
|                                                             |
+-------------------------------------------------------------+
| DIMENSIONS                                    Score: 95/100 |
+-------------------------------------------------------------+
| [PASS] Container width: 1280px (exact match)                |
| [PASS] Padding: 64px (exact match)                          |
| [WARN] Button height: 46px vs 48px (delta: 2px)             |
| [PASS] Gap between elements: 24px (exact match)             |
|                                                             |
+-------------------------------------------------------------+
| COLORS                                       Score: 100/100 |
+-------------------------------------------------------------+
| [PASS] Background: var(--color-background)                  |
| [PASS] Heading: var(--color-foreground)                     |
| [PASS] Button: var(--color-primary)                         |
| [PASS] All colors use design tokens                         |
|                                                             |
+-------------------------------------------------------------+
| TYPOGRAPHY                                    Score: 85/100 |
+-------------------------------------------------------------+
| [PASS] Heading font-family: Inter (match)                   |
| [PASS] Heading font-size: 48px (match)                      |
| [FAIL] Heading line-height: 1.2 vs 1.1 (delta: 0.1)         |
| [PASS] Body font-size: 18px (match)                         |
|                                                             |
+-------------------------------------------------------------+
| SPACING                                       Score: 90/100 |
+-------------------------------------------------------------+
| [PASS] Section padding: 96px (uses --spacing-hero)          |
| [WARN] Card gap: 32px vs 24px (delta: 8px)                  |
| [PASS] Button padding: 16px 32px (match)                    |
|                                                             |
+-------------------------------------------------------------+
| TOKEN AUDIT                                                 |
+-------------------------------------------------------------+
| Tokens used correctly: 12/14                                |
| Hardcoded values found: 0                                   |
| Missing tokens: 2                                           |
|   - --shadow-card (using shadow-md instead)                 |
|   - --radius-card (using rounded-xl instead)                |
|                                                             |
+-------------------------------------------------------------+
| SUGGESTED FIXES                                             |
+-------------------------------------------------------------+
| 1. Line 24: Change leading-tight to leading-none            |
|    (line-height 1.2 -> 1.1)                                 |
|                                                             |
| 2. Line 45: Change gap-8 to gap-6                           |
|    (32px -> 24px to match Figma)                            |
|                                                             |
| 3. Consider adding custom tokens:                           |
|    --shadow-card: 0 4px 12px rgba(0,0,0,0.1)                |
|    --radius-card: 16px                                      |
+-------------------------------------------------------------+
```

## Options

```bash
# Quick validation (dimensions + colors only)
/validate-design <path>

# Full validation (all checks)
/validate-design <path> --full

# Visual comparison (requires browser/Playwright)
/validate-design <path> --visual

# Generate fix suggestions
/validate-design <path> --suggest-fixes

# Apply fixes automatically (with confirmation)
/validate-design <path> --auto-fix

# Output report to file
/validate-design <path> --output validation-report.md

# Set custom tolerance (default: 2px)
/validate-design <path> --tolerance 4
```

## Validation Modes

### Quick Mode (default)
- Dimensions check
- Color token verification
- Basic spacing check
- Fast execution

### Full Mode (`--full`)
- All dimension properties
- All color properties
- All typography properties
- All spacing properties
- Complete token audit

### Visual Mode (`--visual`)
- All above checks
- Screenshot comparison
- Pixel diff calculation
- Requires Playwright/browser

## Scoring

```
Overall Score = weighted average:
- Dimensions: 25%
- Colors: 25%
- Typography: 20%
- Spacing: 20%
- Token Usage: 10%

Per-check scoring:
- PASS = full points
- WARN = 50% points
- FAIL = 0 points
```

## Error Handling

| Error | Resolution |
|-------|------------|
| MCP not connected | Prompt to run /mcp |
| Component not found | Ask for correct path |
| Figma frame not found | Ask for Figma URL |
| No Code Connect mapping | Ask user to provide Figma URL |

## Integration with /design-to-code

After running `/design-to-code`, you'll be prompted:

```
Implementation complete!

Would you like to validate the implementation?
[Yes, validate] [No, skip]
```

If validation score < 95%:

```
Validation score: 87/100

Found 3 discrepancies that can be fixed.
Would you like to apply the suggested fixes?
[Apply fixes] [Show details] [Ignore]
```

## Examples

### Basic Validation
```bash
/validate-design components/ui/Button.tsx
```

### Full Validation with Fixes
```bash
/validate-design components/blocks/HeroSection.tsx --full --suggest-fixes
```

### Validate Against Specific Figma Frame
```bash
/validate-design components/ui/Card.tsx https://figma.com/design/ABC?node-id=1-234
```
