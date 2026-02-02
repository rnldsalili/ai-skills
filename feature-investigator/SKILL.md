---
name: feature-investigator
description: Investigate and map features by exploring a codebase. Use when asked to find where a feature lives, how it works end-to-end, what modules and services are involved, or to produce an investigation report and key code locations before implementation or debugging.
---

# Feature Investigation

## Objectives

- Produce an investigation report and a code location list.
- Stay read-only unless explicitly asked to change code.

## Workflow

1. Restate the feature in one sentence and list the key user-visible behaviors.
2. Identify likely entry points (UI routes, API endpoints, background jobs, CLIs, webhooks).
3. Search for entry points, feature names, and related domain terms.
4. Trace the flow from entry point to core logic to data access.
5. Identify dependencies, integrations, feature flags, configs, and ownership.
6. Check tests and docs for coverage and expected behavior.
7. Record unknowns and verification gaps.

## Tooling guidance

- Use file name search for likely modules (routes, handlers, controllers, services, stores, reducers, models).
- Use content search for endpoint paths, feature flags, analytics events, and domain terms.
- Read only the minimal set of files needed to trace the flow.
- Use git history only to identify owners or recent changes if needed.

## Investigation report format

### Summary

- Feature scope and current behavior in 2-4 sentences.

### Flow

- Entry point(s) -> core logic -> data/storage -> side effects.
- Call out async jobs, events, or queues if present.

### Key modules

- List the most important modules and their roles.

### Risks and gaps

- Unclear logic, missing tests, brittle dependencies, or unknowns.

### Open questions

- Questions that block a full understanding or safe change.

## Code location list format

- `path/to/file.ext` - role in the feature and why it matters.

## Heuristics

- For web apps, start at routing, UI entry components, and API clients.
- For APIs, start at routers/controllers, then services, then data layer.
- For event-driven systems, start at the consumer/handler and trace published events.
- For monorepos, map the boundary: UI, API, shared packages, and infra.
- For feature flags, search config and flag names to find conditional paths.

## Guardrails

- Avoid speculation; label assumptions.
- Do not dump large file contents; summarize key findings.
- Ask only one targeted question if blocked, after completing all possible analysis.
