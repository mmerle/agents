---
name: code-conventions
description: Write code following my conventions — commenting, file organization, component architecture, CSS patterns, TypeScript config, API patterns, and tooling. Use when writing or reviewing code in any project.
---

# Code Conventions

This skill teaches agents how to write code the way I do.

## Quick Reference

These are the conventions agents must follow immediately. Load reference files for deeper context.

### Commenting

```typescript
// NOTE: Explanation of WHY, not WHAT.
// TODO: Planned work with owner attribution.
// REQUEST(entity) MM-DD-YY: Brief of request.
```

No self-documenting comments. No commenting what code does — only why.

See [references/commenting.md](references/commenting.md) for full patterns.

### Naming

Treat names and paths as search terms. Exported symbols use specific, domain-bearing names and canonical vocabulary. Keep private local names proportional rather than mechanically verbose.

See [references/file-organization.md](references/file-organization.md).

### Composition

```
entrypoint -> feature -> module -> utility
```

Code is organized by ownership. Each unit owns one concern, exposes one clear entrypoint, and keeps its internals private.

State belongs as close as possible to where it is used.

See [references/composition.md](references/composition.md).

### CSS

No Tailwind, no CSS-in-JS, no SCSS in new projects.

See [references/css.md](references/css.md) for spacing, typography, themes, and precision values.

## Reference Files

Load these for deeper context on specific areas:

| Reference                                               | When to load                                             |
| ------------------------------------------------------- | -------------------------------------------------------- |
| [commenting.md](references/commenting.md)               | Writing or reviewing comments                            |
| [composition.md](references/composition.md)             | Composing components, modules, packages, and boundaries  |
| [file-organization.md](references/file-organization.md) | Creating files, setting up imports, path aliases         |
| [css-conventions.md](references/css.md)                 | Writing styles, working with themes, spacing, typography |
