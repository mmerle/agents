# CSS Conventions

## Native CSS Nesting

New projects use native CSS nesting instead of SCSS:

```css
.element {
  display: flex;
  flex-direction: column;
  padding-inline: 16px;

  /* states and pseudo selectors */
  &:hover {
    background: var(--theme-foreground);
  }

  &:focus-visible {
    box-shadow: 0 0 0 4px var(--theme-input-active);
  }

  &:not(.is-active) {
    opacity: 0;
  }

  /* media queries */
  @media (width > 640px) {
    flex-direction: row;
  }
}

.element_inner {
  padding-block: 32px;
}
```

## Class Names

Use `c-` only for explicit UI components, never as a general class prefix. Use `o-` for reusable layout patterns and `u-` for single-purpose utilities.

Use `_` for owned elements and `-` for simple modifiers.

Use `data-*` for named variants and JavaScript hooks. Use `is-*` for current state and `has-*` when an element's contents or context affect it.

```html
<article class="c-card">
  <h2 class="c-card_title">Account</h2>
</article>

<div class="o-grid"></div>

<span class="u-screen-reader-text">Loading</span>

<button class="c-button -with-icon" data-size="sm" data-variant="primary">
  <span class="c-button_icon" aria-hidden="true">→</span>
  Continue
</button>

<div class="c-dialog is-open" data-module-dialog>
  <button class="c-dialog_close" data-dialog="close">Close</button>
</div>

<form class="c-form has-errors"></form>
```

## Style Ownership

Set inherited properties on the nearest shared ancestor instead of repeating them on each descendant. Override only the exceptions.

Do not group unrelated selectors because their declarations happen to match. Use a shared class for a shared visual role, or a utility class for an intentionally reusable style.

```css
/* GOOD */
.account-summary {
  color: var(--color-text-secondary);
}

.account-summary_title {
  color: var(--color-text-primary);
}

/* BAD */
.account-summary_intro,
.account-summary_details,
.account-summary_billing,
.account-summary_card p,
.account-summary_status {
  color: var(--color-text-secondary);
}
```

## Media Queries

Keep responsive overrides with the rule they change. Nest media queries instead of collecting them at the end of the file.

```css
/* GOOD */
.profile-layout {
  display: grid;
  grid-template-columns: minmax(0, 1fr);

  @media (width > 640px) {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

.profile-sidebar {
  padding: var(--space-3);

  @media (width > 640px) {
    padding: var(--space-5);
  }
}

/* BAD */
.profile-layout {
  display: grid;
  grid-template-columns: minmax(0, 1fr);
}

.profile-sidebar {
  padding: var(--space-3);
}

@media (width > 640px) {
  .profile-layout {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .profile-sidebar {
    padding: var(--space-5);
  }
}
```

## Element Selectors

Give styled elements their own classes instead of targeting tags. Reserve element selectors for intentional global styles.

```css
/* GOOD */
.feature-card_title {
  font-size: var(--text-heading);
}

.feature-card_description {
  max-width: 60ch;
}

/* BAD */
.feature-card h3 {
  font-size: var(--text-heading);
}

.feature-card p {
  max-width: 60ch;
}
```
