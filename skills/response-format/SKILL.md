---
name: response-format
description: The standard structure every task/feature completion response must follow
---

# Response Format — Quick Reference

The global rule for how the agent reports back **after implementing a task or feature**.
Its purpose is consistency, readability, and easy reuse in PRs and tickets. This rule
supersedes the "No summaries" line in `code-quality` for completion responses.

## When it applies

Apply the template whenever a response **completes work that changed code or config** —
a feature, fix, refactor, migration, or config change.

## The template

End the completing response with these four sections, in this order:

```markdown
### Summary

One or two sentences: what changed and why.

### Verification

- Tests: <command + result, or "not run — reason">
- Lint/typecheck: <result, or "not run — reason">

### Notes

- Caveats, risks, follow-ups, or "None"
- Context docs: <updated files, or "N/A — reason">
```

## Formatting rules

- Use these exact `###` headings, in this order. Omit a section only if the exception rules below apply.
- `Summary` names what changed and why, referencing files as `path:line` where useful — not a narration of steps.
- `Verification` states what was run and the real result. If something wasn't run, say so and why — never imply a check passed that didn't.
- `Notes` is for anything the reviewer needs: risks, deferred work, or a ticket ref (e.g. `PROJ-350`). Write `None` rather than dropping the section.
- The `Context docs` line says whether the project's docs (product docs, ADRs, feature docs, or a
  `CLAUDE.md`) needed changes for this work. If a business rule, capability status, or
  hard-to-reverse decision changed, update those docs before reporting.
- Keep it tight — this is a report, not a transcript.

## Exceptions (do NOT apply the template)

- Read-only work: explanations, code reading, exploration, "how does X work".
- Clarifying questions, proposals, or plans presented for approval (before any code is written).
- Trivial single-line answers or a single command's output where the template adds noise.
- Any response where **no code or config was changed**.

## Examples

### Example 1 — feature completion (template applies)

> ### Summary
>
> Added an OS-based redirect to the universal download link so Android users land on Google Play and iOS users on the App Store. New `GET /download` route in `download.controller.ts` reads the `User-Agent`; `download.service.ts` resolves it to a store URL with a web fallback; e2e cases cover iOS, Android, and unknown UA.
>
> ### Verification
>
> - Tests: `npx jest download` — 6 passed
> - Lint/typecheck: `npm run lint` clean
>
> ### Notes
>
> - Fallback URL is hardcoded to the marketing site; move to config if it changes. PROJ-346.
> - Context docs: N/A — no business rule or capability status changed

### Example 2 — exploration (exception, template omitted)

> The geofence trip start is triggered in `location.service.ts:142` when a
> `LocationEvent` crosses a region boundary — no template needed here, this is a
> read-only explanation.
