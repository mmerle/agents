---
globs:
  - "**/*.css"
  - "**/*.scss"
  - "**/*.module.css"
  - "**/*.tsx"
  - "**/*.jsx"
---

# Style

> CSS modules as 's', no inline styles

---

## DO

### CSS Module Import Convention

```tsx
import s from './component.module.css'
function Component() { return <div className={s.wrapper}>...</div> }
```

### CSS Modules for Complex Components

```css
.button { /* Base */ }
.button[data-variant='primary'] { /* Variant */ }
```

### Theming with CSS Custom Properties

```css
:root { --color-primary: #0066cc; }
[data-theme='dark'] { --color-primary: #66b3ff; }
```

---

## DON'T

```tsx
// WRONG: Inline styles
<div style={{ padding: '20px' }}>
// OK: Dynamic values only
<div style={{ '--progress': `${percent}%` } as CSSProperties}>
```

```css
/* WRONG: Arbitrary z-index */
.element { z-index: 9999; }
/* CORRECT: Scale (10=dropdown, 20=sticky, 30=modal, 40=toast) */
.modal { z-index: 30; }

/* WRONG: Animate layout */
.animate { transition: width 0.3s; }
/* CORRECT: Compositor-only */
.animate { transition: transform 0.3s, opacity 0.3s; }

/* WRONG: will-change always on */
.element { will-change: transform; }
```

```tsx
// WRONG: Global styles in components
import '@/styles/globals.css'  // Only in layout.tsx
```

---

## Z-Index Scale

| Layer | Value |
|-------|-------|
| Dropdown | 10 |
| Sticky | 20 |
| Modal | 30 |
| Toast | 40 |

## Tools

- **CSS Modules**
- **Biome**
