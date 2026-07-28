---
description: Research and produce a concise technical design document for review
agent: plan
---

# Design Doc

Produce a technical design for `$ARGUMENTS`.

Work read-only. Return the design in chat; do not create or edit a file unless explicitly requested.

## Workflow

1. Read applicable project instructions and inspect the code paths, architecture, and conventions relevant to the proposal.
2. Identify the problem, current behavior, constraints, and unresolved requirements.
3. Ask only the targeted questions that are necessary to avoid designing against a false assumption.
4. Compare realistic alternatives before selecting an approach.
5. Produce a phased implementation and verification plan grounded in the existing codebase.

## Format

```text
# <Project or Feature> Design Doc

## Problem Context
Current behavior, pain points, affected users or systems, and relevant constraints.

## Proposed Solution
High-level approach, major behavior changes, and why it fits the existing system.

## Goals and Non-Goals
### Goals
- ...

### Non-Goals
- ...

## Design
Request and data flow, component boundaries, state ownership, interfaces, and important decisions. Include a concise diagram when it improves clarity.

## Alternatives Considered
| Alternative | Advantages | Drawbacks | Decision |
| --- | --- | --- | --- |

## Risks and Failure Modes
- Risk, impact, and mitigation.

## Verification Strategy
Tests, static checks, runtime verification, observability, and rollout evidence appropriate to the change.

## Implementation Plan
### Phase 1: ...
- ...

## Open Questions
- [ ] ...
```

Keep the document concise and concrete. Do not invent metrics, requirements, APIs, or codebase facts. Cite relevant files and symbols. Put unresolved decisions in Open Questions rather than silently choosing assumptions.
