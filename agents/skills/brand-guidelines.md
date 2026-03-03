---
name: brand-guidelines
description: Applies brand guidelines with design tokens, colors, fonts, and visual tone for professional consistency.
---
## Brand Guidelines Skill

**CRITICAL - Respect the Architecture**: You **MUST** read `PROJECT_CONTEXT.md` to understand how brand assets and CSS variables are managed in the current architecture (e.g., global stylesheets, theme providers, Tailwind config, design tokens, etc.).

---

## Core Brand Elements

### Colors
| Token | Purpose | Default | Custom |
|-------|---------|---------|--------|
| `--primary` | Main brand color | `#007bff` | `{primary-color}` |
| `--primary-hover` | Hover state | `#0056b3` | `{primary-hover}` |
| `--accent` | Highlight, CTA | `#ff4500` | `{accent-color}` |
| `--neutral-light` | Light backgrounds | `#f8f9fa` | `{neutral-light}` |
| `--neutral-dark` | Dark backgrounds | `#121212` | `{neutral-dark}` |
| `--text-primary` | Main text | `#1a1a1a` | `{text-primary}` |
| `--text-secondary` | Muted text | `#6c757d` | `{text-secondary}` |

### Semantic Colors
| Token | Purpose | Default |
|-------|---------|---------|
| `--success` | Positive actions | `#28a745` |
| `--warning` | Caution | `#ffc107` |
| `--danger` | Errors, destructive | `#dc3545` |
| `--info` | Information | `#17a2b8` |

---

## Typography

### Font Stack
```css
:root {
  --font-display: '{font-display}', system-ui, sans-serif;
  --font-body: '{font-body}', system-ui, sans-serif;
  --font-mono: '{font-mono}', 'Fira Code', monospace;
}
```

### Type Scale
```css
:root {
  --text-xs: 0.75rem;    /* 12px */
  --text-sm: 0.875rem;   /* 14px */
  --text-base: 1rem;     /* 16px */
  --text-lg: 1.125rem;   /* 18px */
  --text-xl: 1.25rem;    /* 20px */
  --text-2xl: 1.5rem;    /* 24px */
  --text-3xl: 1.875rem;  /* 30px */
  --text-4xl: 2.25rem;   /* 36px */
  --text-5xl: 3rem;      /* 48px */
}
```

### Font Weights
```css
:root {
  --font-light: 300;
  --font-normal: 400;
  --font-medium: 500;
  --font-semibold: 600;
  --font-bold: 700;
}
```

---

## Visual Tone & Style

### Description
{visual-tone-description}
> Example: "Modern and minimalist with subtle asymmetry, smooth gradients, and full dark mode support"

### Key Characteristics
- **Mood**: <sophisticated|playful|professional|bold>
- **Density**: <spacious|compact|balanced>
- **Edges**: <sharp|rounded|mixed>
- **Shadows**: <flat|subtle|prominent>

---

## Design Tokens (Full Set)

### Spacing
```css
:root {
  --space-0: 0;
  --space-1: 0.25rem;  /* 4px */
  --space-2: 0.5rem;   /* 8px */
  --space-3: 0.75rem;  /* 12px */
  --space-4: 1rem;     /* 16px */
  --space-5: 1.25rem;  /* 20px */
  --space-6: 1.5rem;   /* 24px */
  --space-8: 2rem;     /* 32px */
  --space-10: 2.5rem;  /* 40px */
  --space-12: 3rem;    /* 48px */
  --space-16: 4rem;    /* 64px */
  --space-20: 5rem;    /* 80px */
  --space-24: 6rem;    /* 96px */
}
```

### Border Radius
```css
:root {
  --radius-none: 0;
  --radius-sm: 0.125rem;  /* 2px */
  --radius-md: 0.375rem;  /* 6px */
  --radius-lg: 0.5rem;    /* 8px */
  --radius-xl: 0.75rem;   /* 12px */
  --radius-2xl: 1rem;     /* 16px */
  --radius-full: 9999px;
}
```

### Shadows
```css
:root {
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
}
```

### Transitions
```css
:root {
  --transition-fast: 150ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-normal: 300ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-slow: 500ms cubic-bezier(0.4, 0, 0.2, 1);
}
```

### Z-Index Scale
```css
:root {
  --z-dropdown: 1000;
  --z-sticky: 1020;
  --z-fixed: 1030;
  --z-modal-backdrop: 1040;
  --z-modal: 1050;
  --z-popover: 1060;
  --z-tooltip: 1070;
}
```

---

## Mandatory Rules

1. **Always use CSS variables** - Never hard-code colors, fonts, spacing, or sizes
2. **Theme consistency** - Maintain full consistency across light & dark themes
3. **Token-based design** - Use defined tokens, don't create ad-hoc values
4. **Semantic naming** - Use purpose-based names (`--color-danger` not `--color-red`)
5. **Follow architecture** - Respect global rules in `PROJECT_CONTEXT.md`

---

## Theme Implementation

### Light Theme
```css
[data-theme="light"] {
  --bg-primary: var(--neutral-light);
  --bg-secondary: #ffffff;
  --text-primary: #1a1a1a;
  --text-secondary: #6c757d;
}
```

### Dark Theme
```css
[data-theme="dark"] {
  --bg-primary: var(--neutral-dark);
  --bg-secondary: #1e1e1e;
  --text-primary: #f5f5f5;
  --text-secondary: #a0a0a0;
}
```

---

## Usage Examples

### Button Component
```css
.btn-primary {
  background-color: var(--primary);
  color: white;
  padding: var(--space-2) var(--space-4);
  border-radius: var(--radius-md);
  font-family: var(--font-body);
  font-weight: var(--font-medium);
  transition: background-color var(--transition-fast);
}

.btn-primary:hover {
  background-color: var(--primary-hover);
}
```

### Card Component
```css
.card {
  background-color: var(--bg-secondary);
  border-radius: var(--radius-lg);
  padding: var(--space-6);
  box-shadow: var(--shadow-md);
}
```

---

## Guiding Principle

Make the interface **memorable** and 100% aligned with the brand identity.
Use this skill in combination with `frontend-design` skill and the global project rules.

---

## Customization Template

Fill in these values for your project:

```yaml
# Brand Configuration
colors:
  primary: "{primary-color}"
  primary-hover: "{primary-hover}"
  accent: "{accent-color}"
  neutral-light: "{neutral-light}"
  neutral-dark: "{neutral-dark}"

typography:
  display: "{font-display}"
  body: "{font-body}"
  mono: "{font-mono}"

visual-tone: "{visual-tone-description}"
```
