# File Organization

## Naming Conventions

| Type                 | Convention                          | Examples                                                        |
| -------------------- | ----------------------------------- | --------------------------------------------------------------- |
| **Component files**  | PascalCase                          | `Button.tsx`, `ActionBar.tsx`, `DropdownMenu.element.tsx`       |
| **Component styles** | PascalCase, matching component      | `Button.module.css`, `ActionBar.module.css`, `DropdownMenu.css` |
| **Utility files**    | camelCase with a domain concept     | `dateFormatting.ts`, `billingConstants.ts`, `userQueries.ts`    |
| **Directories**      | lowercase, singular when a category | `components/`, `common/`, `modules/`, `runtime/`                |
| **Pages**            | lowercase or kebab-case             | `pages/examples/accordion.tsx`                                  |
| **CSS classes**      | kebab-case with `_` modifier        | `.root`, `.action-button`, `.left-panel`, `.left-panel_inner`   |
| **CSS variables**    | kebab-case with prefix              | `--theme-background`, `--color-gray-60`                         |
| **TypeScript types** | PascalCase with a domain concept    | `UserProfileResult`, `NotificationDeliveryConfig`               |
| **Functions**        | camelCase with a specific operation | `getUserProfile()`, `formatInvoiceDate()`, `isShoppingCartEmpty()` |
| **Constants**        | UPPER_SNAKE_CASE or camelCase       | `BILLING_API_URL`, `notificationRetryLimit`                     |
| **Vendored modules** | lowercase kebab-case directory      | `modules/object-assign/index.ts`                                |

## Naming for Search

Names and paths must make code easy to locate through text search.

- Prefer specific, domain-bearing names for exported symbols.
- Qualify generic operations such as `create`, `get`, `process`, `validate`, and `format` with the concept they operate on.
- Avoid exported names such as `Data`, `Result`, `Config`, `Manager`, `Helper`, and `Utils` without a domain qualifier.
- Use one canonical spelling for each concept across symbols, types, filenames, tests, and imports.
- Do not alias imported domain symbols unless resolving a real collision.
- Before introducing an exported symbol, search the repository for existing terminology and unrelated collisions.
- Keep private local names concise when their meaning is clear from nearby context.

## Test Files

Name tests after the source they cover:

```text
stripeClient.ts
stripeClient.test.ts
```

Keep tests beside their source unless the project has an established mirrored test directory.

## Path Aliases

All projects define path aliases in `tsconfig.json` to avoid relative imports across top-level directories:

```json
{
  "compilerOptions": {
    "paths": {
      "~/*": ["./src/*"]
      "@lib/*": ["./vendor/lib/*"]
    }
  }
}
```

**Rule**: Never use `../../` to cross top-level directory boundaries. Use path aliases instead.

```typescript
// GOOD
import Button from "~/components/Button";
import { formatInvoiceDate } from "~/common/dateFormatting";

// BAD
import Button from "../../components/Button";
import { formatInvoiceDate } from "../../../common/dateFormatting";
```

Relative imports are fine within the same directory (e.g., `./Button.module.css`).

## Vendored Modules

The `modules/` or `packages/` directory contains self-contained, low-dependency code that would otherwise be an npm package:

```
modules/
  cookies/index.ts     # Cookie parsing
  cors/index.ts        # CORS headers
```

**Rule**: If a package does one thing and is small, vendor it in `modules/` instead of adding it to `package.json`. This keeps the dependency tree minimal and the code auditable.
