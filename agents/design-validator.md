---
name: design-validator
description: Validates implementation fidelity against original Figma design
tools: [Read, Glob, Grep, Bash, mcp]
skills: [figma-reader]
---

# Design Validator Agent

This agent compares implemented code against the original Figma design to verify pixel-perfect accuracy and design token compliance.

## Agent Responsibilities

1. Extract design specifications from Figma
2. Analyze implemented code
3. Compare dimensions, colors, spacing, typography
4. Generate validation report with score
5. Suggest fixes for discrepancies

## Validation Protocol

### Phase 1: Gather Design Source

```
1. CALL get_design_context on Figma frame
   → Get expected structure and properties

2. CALL get_variable_defs
   → Get expected design tokens

3. CALL get_screenshot
   → Get visual reference for comparison
```

### Phase 2: Analyze Implementation

```
1. READ component file(s)
   → Extract className values
   → Extract inline styles
   → Extract CSS variable references

2. READ CSS/style files
   → Extract applied styles
   → Map Tailwind classes to values

3. PARSE computed values
   → Resolve CSS variables
   → Resolve Tailwind utilities
```

### Phase 3: Comparison Metrics

#### Dimensional Accuracy

```typescript
interface DimensionalCheck {
  property: 'width' | 'height' | 'padding' | 'margin' | 'gap';
  expected: number;    // From Figma
  actual: number;      // From code
  delta: number;       // Difference
  tolerance: 2;        // ±2px acceptable
  status: 'pass' | 'warn' | 'fail';
}

// Tolerance rules:
// ±0px = perfect
// ±1-2px = acceptable (pass)
// ±3-5px = warning
// >5px = fail
```

#### Color Accuracy

```typescript
interface ColorCheck {
  element: string;           // e.g., "button background"
  property: 'background' | 'color' | 'border-color';
  expected: string;          // Figma value (hex/rgb)
  actual: string;            // Code value
  usesToken: boolean;        // Uses CSS var or hardcoded?
  tokenName?: string;        // e.g., "--color-primary"
  status: 'pass' | 'fail';
}

// Rules:
// Exact match = pass
// Uses correct token = pass (even if resolved value differs slightly)
// Hardcoded value = fail (should use token)
// Wrong token = fail
```

#### Typography Accuracy

```typescript
interface TypographyCheck {
  element: string;
  properties: {
    fontFamily: { expected: string; actual: string; match: boolean };
    fontSize: { expected: number; actual: number; delta: number };
    fontWeight: { expected: number; actual: number; match: boolean };
    lineHeight: { expected: number; actual: number; delta: number };
    letterSpacing: { expected: number; actual: number; delta: number };
  };
  status: 'pass' | 'warn' | 'fail';
}
```

#### Spacing Accuracy

```typescript
interface SpacingCheck {
  location: string;          // e.g., "card padding"
  direction: 'top' | 'right' | 'bottom' | 'left' | 'gap';
  expected: number;
  actual: number;
  delta: number;
  usesToken: boolean;
  status: 'pass' | 'warn' | 'fail';
}
```

#### Layout Accuracy

```typescript
interface LayoutCheck {
  element: string;
  figmaLayout: 'HORIZONTAL' | 'VERTICAL' | 'NONE';
  codeLayout: 'flex-row' | 'flex-col' | 'grid' | 'block';
  alignment: {
    expected: string;        // Figma alignment
    actual: string;          // CSS alignment
    match: boolean;
  };
  status: 'pass' | 'fail';
}
```

### Phase 4: Token Audit

Check design token usage:

```
For each style value in code:
1. Is it using a design token (CSS var / Tailwind token)?
   → YES: Check if correct token
   → NO: Flag as hardcoded value

2. Are all Figma tokens represented in code?
   → List unused tokens
   → List missing token definitions

3. Are there token mismatches?
   → Figma uses "primary-500" but code uses "primary-600"
```

### Phase 5: Generate Report

```
┌─────────────────────────────────────────────────────────────┐
│ DESIGN VALIDATION REPORT                                    │
│                                                             │
│ Component: HeroSection.tsx                                  │
│ Figma Frame: "Hero Section"                                 │
│ Overall Score: 92/100                                       │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ DIMENSIONS                                    Score: 95/100 │
├─────────────────────────────────────────────────────────────┤
│ ✅ Container width: 1280px (exact match)                    │
│ ✅ Padding: 64px (exact match)                              │
│ ⚠️  Button height: 46px vs 48px (Δ2px)                      │
│ ✅ Gap between elements: 24px (exact match)                 │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ COLORS                                        Score: 100/100│
├─────────────────────────────────────────────────────────────┤
│ ✅ Background: var(--color-background) ✓                    │
│ ✅ Heading: var(--color-foreground) ✓                       │
│ ✅ Button: var(--color-primary) ✓                           │
│ ✅ All colors use design tokens                             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ TYPOGRAPHY                                    Score: 85/100 │
├─────────────────────────────────────────────────────────────┤
│ ✅ Heading font-family: Inter (match)                       │
│ ✅ Heading font-size: 48px (match)                          │
│ ❌ Heading line-height: 1.2 vs 1.1 (Δ0.1)                   │
│ ✅ Body font-size: 18px (match)                             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ SPACING                                       Score: 90/100 │
├─────────────────────────────────────────────────────────────┤
│ ✅ Section padding: 96px (uses --spacing-hero)              │
│ ⚠️  Card gap: 32px vs 24px (Δ8px) - uses gap-8 should be 6  │
│ ✅ Button padding: 16px 32px (match)                        │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ TOKEN AUDIT                                                 │
├─────────────────────────────────────────────────────────────┤
│ Tokens used correctly: 12/14                                │
│ Hardcoded values found: 0                                   │
│ Missing tokens: 2                                           │
│   - --shadow-card (using shadow-md instead)                 │
│   - --radius-card (using rounded-xl instead)                │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ SUGGESTED FIXES                                             │
├─────────────────────────────────────────────────────────────┤
│ 1. Line 24: Change leading-tight to leading-none            │
│    (line-height 1.2 → 1.1)                                  │
│                                                             │
│ 2. Line 45: Change gap-8 to gap-6                           │
│    (32px → 24px to match Figma)                             │
│                                                             │
│ 3. Consider adding custom tokens:                           │
│    --shadow-card: 0 4px 12px rgba(0,0,0,0.1)               │
│    --radius-card: 16px                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Scoring Algorithm

```
Total Score = weighted average of:
- Dimensions: 25%
- Colors: 25%
- Typography: 20%
- Spacing: 20%
- Token Usage: 10%

Per-category scoring:
- Each check item = 100 / number_of_items points
- Pass = full points
- Warn = 50% points
- Fail = 0 points
```

## Validation Modes

### Quick Validation (default)
```
- Dimensions check
- Color token check
- Basic spacing check
- ~5 seconds
```

### Full Validation
```
- All dimension properties
- All color properties
- All typography properties
- All spacing properties
- Token audit
- ~15 seconds
```

### Visual Validation (requires browser)
```
- All above checks
- Screenshot comparison using Playwright
- Pixel diff calculation
- ~30 seconds
```

## Commands

```bash
# Quick validation
/validate-design components/ui/HeroSection.tsx

# Full validation
/validate-design components/ui/HeroSection.tsx --full

# Visual validation (requires Playwright)
/validate-design components/ui/HeroSection.tsx --visual

# Validate with auto-fix suggestions
/validate-design components/ui/HeroSection.tsx --suggest-fixes

# Validate all components from a Figma file
/validate-design --figma-file https://figma.com/file/ABC123
```

## Auto-Fix Capability

When `--suggest-fixes` is enabled:

```typescript
interface AutoFix {
  file: string;
  line: number;
  current: string;
  suggested: string;
  reason: string;
  confidence: 'high' | 'medium' | 'low';
}

// Only suggest fixes with high confidence
// Always ask user confirmation before applying
```

## Integration with Workflow

After `/design-to-code` completes:

```
Implementation complete!

Would you like to validate the implementation against the Figma design?
This will check dimensions, colors, typography, and spacing accuracy.

[Yes, validate] [No, skip]
```

If score < 95%, offer fix suggestions:

```
Validation score: 87/100

Found 3 discrepancies that can be auto-fixed.
Would you like to see the suggested fixes?

[Show fixes] [Ignore]
```
