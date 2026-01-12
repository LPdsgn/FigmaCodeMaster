---
name: framework-adapter
description: Adapts Figma design implementation to the target framework detected in the codebase
---

# Framework Adapter Skill

This skill detects the framework used in the target codebase and adapts the generated code to match its conventions, component libraries, and styling patterns.

## Framework Detection

### Detection Priority

1. **package.json** - Primary source for framework detection
2. **Config files** - Framework-specific configuration (next.config.js, nuxt.config.ts, etc.)
3. **File structure** - Directory patterns (app/, pages/, src/routes/)
4. **Existing components** - Code patterns in existing files

### Detection Matrix

```
package.json dependencies → Framework Stack
├── "next"              → Next.js (App Router if app/ exists, Pages Router otherwise)
├── "react" (no next)   → React (Vite, CRA, or custom)
├── "vue" + "nuxt"      → Nuxt 3
├── "vue" (no nuxt)     → Vue 3 (Vite)
├── "svelte" + "@sveltejs/kit" → SvelteKit
├── "svelte" (no kit)   → Svelte (Vite)
├── "astro"             → Astro
├── "solid-js"          → SolidJS
├── "angular"           → Angular
└── none of above       → Vanilla HTML + CSS
```

## Framework-Specific Configurations

### Next.js (App Router)

**Detection**: `next` in dependencies + `app/` directory exists

**Component Structure**:
```typescript
// components/ui/button.tsx
'use client'; // Only if interactive

import { cn } from '@/lib/utils';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'default' | 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
}

export function Button({ variant = 'default', size = 'md', className, ...props }: ButtonProps) {
  return (
    <button
      className={cn(
        'inline-flex items-center justify-center rounded-md font-medium transition-colors',
        variants[variant],
        sizes[size],
        className
      )}
      {...props}
    />
  );
}
```

**UI Library Priority**: shadcn/ui > Radix UI > Headless UI > Custom

**Styling**: Tailwind CSS with CSS variables for design tokens

**File Organization**:
```
app/
├── (routes)/
├── components/
│   ├── ui/           # Base components (shadcn-style)
│   └── blocks/       # Composite components
├── lib/
│   └── utils.ts      # cn() helper
└── styles/
    └── globals.css   # CSS variables + Tailwind
```

---

### React (Vite/CRA)

**Detection**: `react` in dependencies, no `next`

**Component Structure**:
```typescript
// src/components/ui/Button.tsx
import { forwardRef } from 'react';
import { clsx } from 'clsx';

interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'ghost';
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', className, ...props }, ref) => {
    return (
      <button
        ref={ref}
        className={clsx('btn', `btn-${variant}`, className)}
        {...props}
      />
    );
  }
);
```

**UI Library Priority**: Radix UI > Headless UI > Custom

**Styling**: Tailwind CSS or CSS Modules (detect from config)

---

### Vue 3 / Nuxt 3

**Detection**: `vue` in dependencies

**Component Structure**:
```vue
<!-- components/ui/Button.vue -->
<script setup lang="ts">
interface Props {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
}

withDefaults(defineProps<Props>(), {
  variant: 'primary',
  size: 'md',
});
</script>

<template>
  <button
    :class="[
      'btn',
      `btn-${variant}`,
      `btn-${size}`,
    ]"
  >
    <slot />
  </button>
</template>

<style scoped>
.btn {
  @apply inline-flex items-center justify-center rounded-md font-medium transition-colors;
}
</style>
```

**UI Library Priority**: Nuxt UI > Headless UI Vue > PrimeVue > Custom

**Styling**: Tailwind CSS with UnoCSS as alternative

---

### SvelteKit / Svelte

**Detection**: `svelte` in dependencies

**Component Structure**:
```svelte
<!-- src/lib/components/ui/Button.svelte -->
<script lang="ts">
  export let variant: 'primary' | 'secondary' | 'ghost' = 'primary';
  export let size: 'sm' | 'md' | 'lg' = 'md';
</script>

<button
  class="btn btn-{variant} btn-{size}"
  on:click
  {...$$restProps}
>
  <slot />
</button>

<style>
  .btn {
    @apply inline-flex items-center justify-center rounded-md font-medium transition-colors;
  }
</style>
```

**UI Library Priority**: Skeleton UI > Melt UI > Custom

---

### Astro

**Detection**: `astro` in dependencies

**Component Structure**:
```astro
---
// src/components/ui/Button.astro
interface Props {
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  href?: string;
}

const { variant = 'primary', size = 'md', href, ...attrs } = Astro.props;
const Element = href ? 'a' : 'button';
---

<Element
  href={href}
  class:list={['btn', `btn-${variant}`, `btn-${size}`]}
  {...attrs}
>
  <slot />
</Element>

<style>
  .btn {
    @apply inline-flex items-center justify-center rounded-md font-medium transition-colors;
  }
</style>
```

**Note**: Can integrate React/Vue/Svelte components via Astro islands

---

### Vanilla HTML + CSS

**Detection**: No framework detected

**Structure**:
```html
<!-- components/button.html -->
<button class="btn btn-primary" data-size="md">
  Click me
</button>
```

```css
/* styles/components/button.css */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: var(--radius-md);
  font-weight: 500;
  transition: all 0.2s;
}

.btn-primary {
  background: var(--color-primary);
  color: var(--color-primary-foreground);
}
```

**Enhancement**: Alpine.js for interactivity if detected

---

## Styling System Detection

### Tailwind CSS
**Detect**: `tailwindcss` in devDependencies. 
- **Version 4**: Package version >= 4.0.0 OR presence of CSS with `@import "tailwindcss";` or `@theme` block.
- **Version 3**: Package version < 4.0.0 AND presence of `tailwind.config.js` or `tailwind.config.ts`.

**Token Integration (Version 3)**:
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: 'var(--color-primary)',
          foreground: 'var(--color-primary-foreground)',
        },
      },
      spacing: {
        // Map Figma spacing tokens
      },
      borderRadius: {
        // Map Figma radius tokens
      },
    },
  },
};
```

**Token Integration (Version 4)**:
```css
/* app/globals.css or styles/main.css */
@import "tailwindcss";

@theme {
  --color-primary: #3b82f6;
  --color-primary-foreground: #ffffff;
  
  /* Extend spacing */
  --spacing-18: 4.5rem; /* 72px */
  
  /* Extend radius */
  --radius-sm: 4px;
}
```

### CSS Modules
**Detect**: `.module.css` files exist or CSS modules config in build tool

### CSS-in-JS
**Detect**: `styled-components`, `@emotion/styled`, or `stitches` in dependencies

### Plain CSS
**Detect**: No styling library, use CSS custom properties

---

## Component Library Detection

Check for existing UI libraries to reuse:

| Library | Detection | Action |
|---------|-----------|--------|
| shadcn/ui | `components.json` OR (`components/ui/` + `@radix-ui/*`) | Extend existing components |
| Radix UI | `@radix-ui/*` | Use primitives directly |
| Headless UI | `@headlessui/*` | Use for accessible patterns |
| Chakra UI | `@chakra-ui/react` | Use Chakra components |
| MUI | `@mui/material` | Use MUI components |
| Ant Design | `antd` | Use Ant components |
| Nuxt UI | `@nuxt/ui` | Use Nuxt UI components |
| DaisyUI | `daisyui` | Use DaisyUI classes |

---

## Adaptation Algorithm

```
1. DETECT framework from package.json
2. DETECT styling system (Tailwind, CSS Modules, etc.)
3. DETECT existing UI library
4. SCAN existing components for patterns:
   - Naming convention (PascalCase, kebab-case)
   - File structure (flat, nested)
   - Props pattern (destructured, interface)
   - Export style (named, default)
5. GENERATE code matching detected patterns
6. INTEGRATE with existing design tokens if present
```

## Output Consistency Rules

1. **Match existing naming**: If project uses `Button.tsx`, don't create `button.tsx`
2. **Match existing structure**: If components are in `src/components/`, use same path
3. **Match existing patterns**: Copy prop typing style from existing components
4. **Reuse utilities**: If `cn()` or `clsx()` exists, use it
5. **Respect config**: Use existing Tailwind/PostCSS configuration
