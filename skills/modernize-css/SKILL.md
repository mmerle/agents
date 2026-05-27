---
name: modernize-css
description: Modernize CSS, style lint CSS/SCSS, and clean up generated styles. Use when writing, editing, reviewing, refactoring, or linting .css/.scss files to prefer CSS 4+ syntax, native nesting, custom properties, logical properties, data attributes, low specificity selectors, and modern responsive patterns.
---

# Modernize CSS

Use this skill when writing, editing, reviewing, or refactoring CSS-like styles. Its main job is to clean up messy generated CSS and move it toward a modern, CSS-first style.

Treat `.scss` files as CSS files that happen to pass through Sass. Do not add Sass-only abstractions unless the surrounding legacy file already depends on them or the user explicitly asks for Sass.

## Operating Mode

When this skill is active:

- Prefer making the CSS better directly, not only describing what should change.
- Preserve the existing visual intent unless the user asks for a redesign.
- Modernize the smallest useful surface area instead of rewriting unrelated styles.
- Keep selectors and tokens understandable for future edits.
- Remove generated-code clutter, duplicate declarations, outdated patterns, and avoidable specificity.
- If reviewing instead of editing, report concrete issues with file and selector references.

## Core Style

Write modern CSS first.

Prefer:

- CSS custom properties over Sass variables.
- Native CSS nesting over repeated flat selectors.
- `:where()` for low-specificity scope, variants, and defensive selector grouping.
- Attribute selectors for state and variants, such as `[data-size="lg"]`, `[data-color="secondary"]`, `[data-state="open"]`, and `[data-visible="true"]`.
- Logical properties such as `padding-inline`, `margin-block`, `inset-inline`, `border-start-start-radius`, `inline-size`, and `block-size`.
- Modern media query range syntax such as `@media (width > 768px)` and `@media (480px <= width < 960px)`.
- Container queries when component layout depends on container size rather than viewport size.
- CSS functions such as `clamp()`, `min()`, `max()`, `color-mix()`, `calc()`, `round()`, and `light-dark()` when supported by the project.
- Modern color spaces such as `oklch`, `oklab`, and `color-mix(in oklab, ...)` when they improve color behavior.
- `text-wrap: balance` for short headings and hero copy.
- `text-wrap: pretty` for body copy where it improves ragging.
- `aspect-ratio`, `place-items`, `place-content`, `subgrid`, `container-type`, and `backdrop-filter` when they simplify layout or effects.

Avoid by default:

- Sass variables, mixins, maps, loops, `@extend`, interpolation, and Sass-only functions.
- Deep specificity chains that are hard to override.
- Styling state with extra modifier classes when a data attribute would be clearer.
- Utility sprawl when a component, section, or local token would be clearer.
- Hardcoded repeated values that should be custom properties.
- Physical directional properties like `margin-left`, `padding-right`, `top`, `right`, `bottom`, and `left` when logical properties work.
- Legacy media queries like `@media screen and (min-width: 768px)` when range syntax is enough.
- `transition: all` unless the affected properties are intentionally broad and low risk.
- `!important` unless the user explicitly asks or a third-party override leaves no practical alternative.

## Rewrite Priorities

When cleaning generated CSS, prioritize changes in this order:

1. Preserve behavior and responsive layout.
2. Collapse repeated values into custom properties.
3. Replace avoidable Sass syntax with native CSS.
4. Lower specificity with `:where()` and simpler selector ownership.
5. Convert physical properties to logical properties where directionality is not intentionally physical.
6. Replace old breakpoint syntax with range media queries.
7. Move variant and state styling to data attributes where markup supports it or the user is already editing markup.
8. Remove dead declarations, duplicate rules, and generated-code noise.

Do not introduce a design token system that does not exist. Use local custom properties near the component or page scope unless the project already has global tokens.

## Structure

Organize styles in this order when practical:

1. File or section comment.
2. Scope or root selector with custom property defaults.
3. Base element or layout selectors.
4. Reusable components.
5. Sections.
6. Utilities.
7. Page, route, or template overrides.

Inside a rule, prefer this rough order:

1. Custom properties.
2. Positioning and display.
3. Box model and sizing.
4. Typography.
5. Color, background, border, shadow, and visual effects.
6. Interaction, transitions, and animation.
7. Nested elements, variants, states, and media or container queries.

This order is guidance, not a reason to churn large files unnecessarily.

## Selectors

- Use `:where([data-page-id="..."])` for page-scoped styles.
- Use `:where([data-page-template="..."])` for reusable template-scoped styles.
- Use `&:where([data-*="..."])` for component variants to keep specificity low.
- Use child nesting for component internals when the relationship is structural.
- Keep nesting shallow. If a selector needs four or more levels, consider a new class or data attribute.
- Prefer state attributes like `[data-state="open"]`, `[aria-expanded="true"]`, `[aria-current="page"]`, and `[open]` over invented state classes when they match the markup semantics.
- Do not replace framework-required classes or third-party selectors unless the user is also changing the markup/API that depends on them.

Good pattern:

```css
:where([data-page-id="example"]) {
    --example-card-bg: var(--color-bg-elevated-primary, white);
    --example-card-padding: clamp(1rem, 3vw, 2rem);

    .example-card {
        display: grid;
        gap: 1rem;
        padding: var(--example-card-padding);
        border: 1px solid var(--border-light, color-mix(in oklab, currentColor 14%, transparent));
        border-radius: var(--radius-md, 0.75rem);
        background: var(--example-card-bg);

        &:where([data-featured="true"]) {
            --example-card-bg: color-mix(in oklab, var(--color-blue, #2563eb) 8%, white);
        }

        @media (width > 768px) {
            --example-card-padding: 2rem;
        }
    }
}
```

## Custom Properties

- Define tokens close to the scope that owns them.
- Use custom properties for repeated values, variants, responsive values, and theming seams.
- Prefix local tokens by feature or component, such as `--mkt-*`, `--form-*`, `--card-*`, or `--example-*`.
- Use fallbacks when a token may be consumed outside its owning scope.
- Prefer changing custom property values in variants and breakpoints instead of duplicating whole declaration blocks.
- Do not turn one-off values into tokens unless it improves clarity or future variation.

## Responsive CSS

- Start with the mobile/default layout, then layer wider breakpoints with range media queries.
- Prefer fluid sizing with `clamp()` where it reduces breakpoint churn.
- Prefer container queries when the component can appear in multiple layout contexts.
- Keep responsive overrides near the base rule they modify.
- Change custom properties at breakpoints when multiple declarations depend on the same responsive value.
- Avoid desktop-first overrides unless the existing file is clearly desktop-first and a local rewrite would be risky.

## Motion And Interaction

- Wrap non-essential transitions and animation in `@media (prefers-reduced-motion: no-preference)`.
- Keep transition lists explicit, such as `transition: opacity 180ms ease, transform 180ms ease`.
- Preserve visible focus states.
- Do not remove `outline` without adding an accessible replacement.
- Prefer `:focus-visible` for keyboard focus styling.
- Respect semantic state selectors like `[disabled]`, `[aria-disabled="true"]`, `[aria-expanded]`, and `[aria-selected]`.

## Compatibility

Modernize aggressively, but honor project constraints.

- If the project has a browser support policy, follow it.
- If the project has PostCSS, Lightning CSS, Sass, or build tooling that transforms modern CSS, account for what it supports.
- If unsupported CSS would ship raw to production, avoid risky features or note the compatibility tradeoff.
- Do not add polyfills, build tools, or lint packages unless the user explicitly asks.

## Review Checklist

When reviewing generated CSS or SCSS, check for:

- CSS-first code instead of Sass-heavy code.
- Low-specificity page, template, component, and variant scoping with `:where()` where useful.
- Custom properties for repeated values, variants, theming seams, and responsive tokens.
- Logical properties instead of physical directional properties.
- Modern media query range syntax.
- Container queries where viewport queries are the wrong abstraction.
- Reasonable nesting depth and clear selector ownership.
- Explicit transitions and reduced-motion handling.
- Accessible focus states.
- No unnecessary Sass-only features.
- No avoidable `!important`.
- No accidental regressions to existing shared styles.
- Responsive behavior on mobile and desktop.

## Verification

After editing bundled styles, run the relevant style check or asset build when available. Prefer the narrowest relevant command, such as a CSS lint, stylelint, Vite build, framework build, or `npm run build`.

If no verification command is obvious, inspect package scripts and report that no targeted style/build command was found.
