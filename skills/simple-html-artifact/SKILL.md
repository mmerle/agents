---
name: simple-html-artifact
description: Creates self-contained browser-based visual explainers as HTML. Use when the user explicitly asks to "explain this visually", "make an HTML explainer", "show this in the browser", or create an interactive explanation, diagram, timeline, comparison, report, brief, dashboard, or reference that terminal Markdown cannot present clearly.
---

# Simple HTML Artifact

Create a static, single-page HTML artifact when spatial layout, visual relationships, or lightweight interaction would explain the subject more clearly than terminal Markdown. Do not use this skill for an ordinary explanation that headings, lists, tables, or code blocks can communicate adequately.

Default to one self-contained, browser-openable HTML file. Use semantic HTML, embedded CSS, inline SVG when it conveys information, and only the minimum JavaScript needed for filtering, sorting, copying, disclosure, or keyboard-safe interaction. Do not add dependencies, frameworks, build tooling, CDN assets, or external fonts.

## Workflow

1. Decide whether the explanation genuinely needs a browser artifact. If not, answer in terminal Markdown.
2. Inspect any existing artifact before editing it. Preserve its copy, information architecture, density, and visual language unless those are the defect.
3. Write the artifact to `.artifacts/simple-html-artifact/<descriptive-name>.html` at the active workspace root. Create the directories if needed. Reuse the same descriptive file for revisions rather than creating numbered copies.
4. Open the completed file with the platform-appropriate command. On macOS, run `open ".artifacts/simple-html-artifact/<descriptive-name>.html"` from the workspace root.
5. If browser inspection tooling is available, inspect the rendered page, accessibility tree, responsive states, interactions, and console. Otherwise, state that visual verification was not available.

Do not overwrite an unrelated existing artifact. If the correct workspace root or destination is ambiguous, ask one short question before writing.

## Goal

Optimize for comprehension, orientation, retrieval, and trust, not conversion. The first viewport must show real information, not only introductory copy.

Prioritize:

- A reading path from summary to evidence to reference.
- Claims paired with definitions, caveats, dates, assumptions, confidence, or sources when those affect interpretation.
- Semantic HTML, readable typography, mobile fit, keyboard order, print readability, and contrast.
- One subject-specific retrieval device such as a decision path, annotated scale, timeline rail, field guide, map, checklist, compact diagram, or message template.

Avoid:

- Generic AI SaaS styling: gradient heroes, glass cards, icon grids, abstract glows, repeated rounded cards, and vague calls to action.
- Document-template styling: beige page washes, low-contrast gray text, boxed navigation chips, heavy dividers, and bordered panels around every section.
- Subject-cliche palettes. Use motif color as an accent unless the user requests a theme.
- Fake completeness: invented metrics, arbitrary percentages, placeholder data, decorative charts, or visuals implying false precision.

## Brief

Before styling, establish:

```text
Subject:
Audience/context:
Reader job:
Posture:
Primary surface:
Type direction:
Color direction:
Deviation reason, if any:
```

Carry forward explicit user preferences. Choose a posture that constrains the design, such as editorial guide, field guide, research brief, technical reference, museum label, workshop handout, or operating checklist. Without a posture, the result tends toward generic application or policy-document styling.

Use external research only when the subject is current, culturally specific, or visually ambiguous. Extract information-design logic from relevant references rather than copying marketing pages. Link every source used in the artifact and distinguish sourced facts from assumptions or interpretation.

## Information Architecture

Start with the reader's job:

- **Orient:** purpose, audience, scope, and main question.
- **Decide:** three to five key claims, rules, risks, rankings, or recommendations.
- **Inspect:** examples, excerpts, visuals, rows, scales, notes, or tables.
- **Retrieve:** anchors, labels, section names, and compact references.

Default order: title and purpose, useful metadata, concrete key points, one primary surface, supporting sections organized by reader need, then caveats, method, assumptions, and sources near the claims they affect or at the end.

For one-page artifacts, compose one explanation rather than reusable application sections. Add sticky navigation, rails, or repeated section chrome only when page length requires navigation.

Use this digestibility ladder:

- First screen: title, one useful rule or takeaway, and the beginning of the primary surface.
- Middle: the working reference, with labels, rows, checklists, or diagrams carrying most of the explanation.
- End: examples, caveats, source notes, templates, or print material.
- Give each section one job: decide, compare, explain, retrieve, or copy.

Keep headers one-column by default: eyebrow, heading, one-sentence purpose, then optional compact metadata in the same flow. Do not place a decorative summary card, metric strip, or side panel beside the heading without a real paired relationship.

## Surface Choice

Choose the primary surface by reader action rather than defaulting to cards, tables, dashboards, or two columns:

- Compare identical attributes: table or matrix.
- Choose among options: decision tree, ranked strips, "choose this if" guide, or scorecard.
- Learn a taxonomy: field-guide entries, specimen cards, annotated map, or glossary.
- Understand sequence: timeline, process diagram, checklist, or flow.
- See relationships: axis map, cluster diagram, annotated scale, or small multiples.
- Remember rules: rule cards, do/avoid pairs, recipe, or cheat sheet.
- Monitor state: compact dashboard with definitions.

Use tables only when rows share attributes and column scanning beats prose. Use two columns only for a paired relationship such as overview/detail, map/list, before/after, controls/results, image/annotation, or claim/evidence. Use inline SVG only when it explains, locates, compares, encodes scale, or gives subject-specific identity. Every SVG needs an accessible name or equivalent nearby text.

After the primary surface, choose at most two different supporting retrieval shapes. Do not turn every section into another table, card grid, or equal-width column set.

## Visual Defaults

Use embedded, hand-written CSS with custom properties for repeated design tokens. Keep the stylesheet specific to the artifact and avoid generic utility-class systems.

Keep the layout as narrow as the content permits. Prose should generally stay around 65 to 80 characters per line; wider comparison or diagram surfaces may extend beyond it. Use stable dimensions for fixed grids, charts, and controls while allowing horizontal overflow on small screens when compression would destroy meaning.

Use type, alignment, whitespace, tint, and composition before borders. Leave some modules unboxed. If most sections need lines, the hierarchy is too flat.

Default to a white background. Use tinted, dark, or textured backgrounds only when justified by user preference, source palette, subject motif, print requirements, or hierarchy. Use dark text, muted text, a light separator, one accent scale, and at most one supporting tint. Every added hue needs an informational job.

Use system font stacks intentionally and keep at most four type roles: display, section heading, body, and caption or label. Do not rely on external font loading. No more than one typographic layer should use letterspacing or all-caps labels.

## Content Discipline

Set length by the reader's job. Summaries should contain three to five concrete claims, table cells should stay compact, diagram labels should explain position or relationship, and prose should keep one idea per paragraph.

If a section is too long, convert prose into labels, rows, scales, captions, or do/avoid pairs. If it is too thin, add a real example, comparison, caveat, definition, or decision rule. Do not add generic overview paragraphs merely to make the page feel complete.

For quick references, organize around the reader's next decision rather than exhaustive coverage. Prefer four to six major sections and one or two desktop screens. If the artifact grows longer, cut weak examples, definitions, and source prose before adding navigation.

Clearly mark assumptions, dates, freshness, scope, confidence, sources, and placeholder data when they affect interpretation. Do not invent metrics, ratings, examples, or benchmarks to fill a layout. Charts and SVGs need units, labels or a legend, and a takeaway caption.

## Interaction

Prefer native HTML behavior such as links, buttons, `details`, forms, and in-page anchors. Add JavaScript only when native HTML cannot satisfy the reader's task cleanly.

When JavaScript is necessary:

- Keep it inline and dependency-free.
- Preserve functionality without JavaScript where practical.
- Support keyboard operation and visible focus.
- Respect `prefers-reduced-motion`.
- Avoid storing or transmitting user data unless explicitly requested.

## Final Check

Before finishing:

1. Remove fake data, arbitrary percentages, placeholders, lorem ipsum, and conversion copy unless requested.
2. Remove repeated section shells, equal-weight panels, decorative SVGs, excessive comments, and JavaScript that native HTML can replace.
3. Improve the two or three weakest aspects, usually hierarchy, verbosity, surface choice, or visual sameness.
4. Check semantics, keyboard and focus behavior, labels, contrast, touch targets, mobile overflow, long text, print readability, and reduced motion.
5. Confirm all CSS, SVG, and JavaScript are embedded and that no network request is required to render the artifact.
6. Confirm sources are linked and assumptions are visibly labeled.
7. Open the file in the browser and report its path along with what was or was not visually verified.
