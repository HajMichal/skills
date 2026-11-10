---
name: code-quality
description: Clean code principles and process rules that govern how code changes are made in a TypeScript monorepo
---

# Code Quality — Quick Reference

## Clean Code Rules

- **Constants over magic numbers** — named constants, never raw values
- **Meaningful names** — reveal intent; avoid abbreviations; booleans prefix with `is`/`has`
- **Single responsibility** — one function, one job; if it needs a comment to explain what it does, split it
- **DRY** — extract repeated logic; single source of truth
- **Comments explain why**, not what — code must be self-documenting
- **No `console.log`** in committed code — use NestJS `Logger` in the API

## Code Style

- Single quotes, `printWidth: 100`, `trailingComma: 'es5'`, `tabWidth: 2`
- Run `pnpm exec prettier --write <changed files>` before committing — never root `pnpm format` (rewrites the whole repo)

## Process Rules (How to Apply Changes)

| Rule | Behavior |
|------|---------|
| Make changes file by file | Allow review between files |
| No inventions | Only change what was explicitly requested |
| Preserve existing code | Don't remove unrelated functionality |
| No unnecessary updates | Don't touch files with no actual changes needed |
| Verify first | Read the file before commenting on or modifying it |
| Standard completion report | After changing code, close with the `response-format` template — not a step-by-step narration |
| Provide real file links | Always link to actual source files, not docs |

## Refactoring Guidance

- Fix technical debt early — don't let it compound
- Leave code cleaner than you found it
- Refactor in a separate commit from feature work

## Resources

| Topic | File | When to Use |
|-------|------|------------|
| Naming examples | [`naming-examples.md`](resources/naming-examples.md) | Unsure how to name a variable/function |
| Code review checklist | [`review-checklist.md`](resources/review-checklist.md) | Pre-PR self-review |
| Encapsulation patterns | [`encapsulation.md`](resources/encapsulation.md) | Hiding implementation, clean interfaces |
