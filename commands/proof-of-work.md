---
description: Prove the current diff is review-ready with project checks and visual browser evidence
agent: build
---

# Proof of Work

Verify the current working-tree changes. Use `$ARGUMENTS` as the target URL, route, or additional verification scope when provided.

## 1. Establish Scope

- Read the applicable `AGENTS.md` and project documentation.
- Inspect `git status` and the relevant diff to understand what changed.
- If there is no diff or other concrete output to verify, report that there is nothing to prove and stop.

## 2. Run Mechanical Checks

Discover the project's own validation commands from its instructions, package scripts, task runner, and CI configuration. Do not install tools or invent commands.

Run every applicable hard gate, including when available:

1. Type checking
2. Tests
3. Linting or static analysis
4. Build validation when the changed behavior depends on a successful build

Use the project's package manager and documented command order. Prefer focused checks for the changed scope, but run the full relevant gate when practical. Continue through independent gates after a failure so the report is complete.

## 3. Verify Visual Changes

Visual verification is required when the diff affects pages, components, layout, styling, responsive behavior, or user interaction.

When Chrome DevTools MCP tools and a reachable application are available:

1. Resolve the target from `$ARGUMENTS`, project instructions, or the changed route. Ask one concise question if the target cannot be determined.
2. Navigate to the affected state.
3. Inspect the accessibility tree and rendered content.
4. Capture a screenshot.
5. Exercise the interaction changed by the task.
6. Check relevant browser console errors.
7. For page or layout work, check representative desktop, tablet, and mobile viewports.

Verify the behavior requested by the task, not merely that the page loads. Check for clipping, overflow, broken hierarchy, missing content, incorrect states, and interaction regressions.

If Chrome DevTools tools are unavailable or the application cannot be reached, do not substitute an unsupported claim. Record visual proof as not run and explain why.

## 4. Report Evidence

Return a concise report with these sections:

```text
Mechanical
- <command>: pass | fail | not run - <reason>

Visual
- Required: yes | no
- Target: <URL or state>
- Viewports/interactions checked: <evidence>
- Result: pass | fail | not run - <reason>

Verdict
- review-ready
- review-ready (mechanical only)
- not review-ready
- insufficient proof
```

Use `review-ready` only when every applicable mechanical gate passes and any required visual verification passes. Use `review-ready (mechanical only)` when mechanical gates pass but required visual verification cannot run. Use `not review-ready` when a gate or visual check fails. Use `insufficient proof` when no meaningful validation is available.

Never claim a command, browser interaction, accessibility inspection, viewport check, or screenshot that did not run.
