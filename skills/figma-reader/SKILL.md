---
name: figma-reader
description: Extracts Figma design data via MCP Remote Server with intelligent tool selection
---

# Figma Reader Skill

This skill provides structured access to Figma designs through the official Figma MCP Remote Server.

## Prerequisites

- Figma MCP Remote Server must be connected (`https://mcp.figma.com/mcp`)
- User must be authenticated via Figma OAuth
- Valid Figma URL or selection context required

## Available MCP Tools

### get_design_context
**Purpose**: Get structured React + Tailwind representation of a Figma selection

**When to use**:
- Converting a frame/component to code
- Understanding the structure of a design
- Getting implementation-ready output

**Parameters**:
- `url` or `nodeId`: Figma frame/component URL or node ID
- `framework` (optional): Target framework (react, vue, html)

**Example prompt**: "Generate code for this Figma frame: [URL]"

### get_variable_defs
**Purpose**: Extract design tokens (variables and styles)

**When to use**:
- Extracting color palettes
- Getting spacing/sizing values
- Retrieving typography definitions
- Building design token files

**Returns**: Variable names, values, types (color, spacing, typography, etc.)

**Example prompt**: "Get the variable names and values used in this frame"

### get_screenshot
**Purpose**: Capture visual reference of a Figma selection

**When to use**:
- Complex layouts that need visual verification
- Maintaining pixel-perfect accuracy
- Generating visual documentation

**Returns**: Screenshot image for visual comparison

**Example prompt**: "Take a screenshot of this frame for reference"

### get_metadata
**Purpose**: Get sparse XML with layer structure

**When to use**:
- Large designs that exceed context limits
- Understanding component hierarchy
- Identifying specific nodes to fetch

**Returns**: Layer IDs, names, types, positions, sizes

**Example prompt**: "Get the layer structure of this design"

### get_code_connect_map
**Purpose**: Map Figma nodes to existing codebase components

**When to use**:
- Checking if components already exist in code
- Understanding existing mappings
- Avoiding duplicate implementations

**Returns**: Node ID to component path mappings

### add_code_connect_map
**Purpose**: Create mappings between Figma elements and code components

**When to use**:
- After implementing a new component
- Updating component relationships
- Maintaining design-code sync

## Extraction Workflow

### Step 1: Validate Connection
```
Check MCP connection status via /mcp
If not connected, prompt user to authenticate
```

### Step 2: Parse Figma URL
```
Extract from URL:
- File ID: /file/{fileId}/...
- Node ID: ?node-id={nodeId} or /design/{fileId}?node-id={nodeId}
```

### Step 3: Choose Tool Based on Intent

| User Intent | Primary Tool | Secondary Tool |
|-------------|--------------|----------------|
| "Convert to code" | get_design_context | get_screenshot |
| "Extract tokens/variables" | get_variable_defs | - |
| "What's in this design?" | get_metadata | get_screenshot |
| "Check component mapping" | get_code_connect_map | - |
| "Visual reference" | get_screenshot | - |

### Step 4: Handle Large Designs
```
If design too large:
1. Use get_metadata to get structure
2. Identify key nodes/frames
3. Fetch individual nodes with get_design_context
4. Combine results
```

## Output Format

### Design Context Output
```typescript
interface DesignContext {
  name: string;
  type: 'FRAME' | 'COMPONENT' | 'INSTANCE';
  layout: {
    mode: 'HORIZONTAL' | 'VERTICAL' | 'NONE';
    gap: number;
    padding: { top: number; right: number; bottom: number; left: number };
  };
  dimensions: { width: number; height: number };
  styles: {
    background: string;
    borderRadius: number;
    boxShadow?: string;
  };
  children: DesignContext[];
  suggestedCode: string; // React + Tailwind suggestion
}
```

### Variable Definitions Output
```typescript
interface VariableDefinitions {
  colors: { name: string; value: string; mode?: string }[];
  spacing: { name: string; value: number; unit: string }[];
  typography: {
    name: string;
    fontFamily: string;
    fontSize: number;
    fontWeight: number;
    lineHeight: number;
  }[];
  radii: { name: string; value: number }[];
  shadows: { name: string; value: string }[];
}
```

## Error Handling

| Error | Cause | Resolution |
|-------|-------|------------|
| "MCP not connected" | Server not authenticated | Run `/mcp` and authorize |
| "Node not found" | Invalid node ID or URL | Verify Figma URL is correct |
| "Access denied" | No permission to file | Request file access from owner |
| "Rate limited" | Too many requests | Wait and retry |

## Best Practices

1. **Be explicit**: Clearly state which tool/data you need
2. **Batch requests**: Combine related extractions when possible
3. **Cache results**: Store extracted data for validation phase
4. **Verify visually**: Use get_screenshot for complex layouts
5. **Check mappings**: Always check get_code_connect_map before creating new components
