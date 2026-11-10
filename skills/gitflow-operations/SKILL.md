---
name: gitflow-operations
description: Branch naming, commit format, PR and review rules for trunk-based development
---

# Trunk-Based Development — Quick Reference

`main` is the single trunk and the only protected branch. It is the source of truth for both
test and production — there is no `dev`, `release/*`, or `hotfix/*` branch. Every change, including
urgent production fixes, follows the same flow.

## Branch Flow

| From   | To     | Naming              |
| ------ | ------ | ------------------- |
| `main` | `main` | `PROJ-123-short-name` |

- **Never commit directly** to `main`
- Keep branches small and short-lived — one focused change, merged as soon as review + CI pass
- Don't let branches accumulate multiple unrelated changes; open another branch instead
- Delete branch after merge

## Commit Format

```
[PROJ-123] type(scope): description
```

Ticket id comes first, in square brackets, before the conventional-commit type.

`PROJ` is the project's issue-tracker key. Take the ticket id from the current branch name (`PROJ-123-short-name`), falling back to recent
commits on the branch. Only ask the user when neither has one — never invent it.

| Type       | Use                                  |
| ---------- | ------------------------------------ |
| `feat`     | New feature                          |
| `fix`      | Bug fix                              |
| `refactor` | Code restructure, no behavior change |
| `test`     | Adding/fixing tests                  |
| `docs`     | Documentation only                   |
| `chore`    | Maintenance, deps, config            |
| `style`    | Formatting only                      |

Examples:

```
[PROJ-42] feat(auth): add JWT refresh flow
[PROJ-58] fix(orders): correct total for discounted items
[PROJ-61] chore(deps): bump prisma to 7.4
```

## PR Descriptions

Fill in `.github/pull_request_template.md` if the repo has one (Ticket / Summary / How to test / Checklist). Never add a
"Generated with Claude Code" line. Short — a wall of text doesn't get read. Reviewers need what
changed, not how.

- **Summary**: 1–2 sentences on the feature, then terse one-line bullets of changes grouped by app. No file-by-file narration, no design essays — link the ADR/doc instead.
- **How to test**: numbered steps with expected results.
- **Checklist**: one line per check with the real result.
- **Follow-ups**: one line, only if something is deliberately left out.
- **Exception — instructions others must follow** (e.g. how a partner calls the API, manual deploy or onboarding steps): keep these complete and copy-pasteable. Be detailed there; be brief everywhere else.

## Code Review

1. All changes via Pull Request — no direct commits to `main`
2. Minimum 1 approval required; CI checks must pass
3. Review is intentionally lightweight: the main value is a second person confirming _what_
   is being introduced to `main`, plus a sanity check for obvious bugs — not an exhaustive
   line-by-line audit
4. Small diffs are what make this fast. Reviews must not become a blocking bottleneck —
   if a PR is small and does what it says, approve and move on
5. Delete branch after merge

## Urgent Production Fixes

No separate hotfix process. Branch from `main`, keep the fix minimal, PR back into `main` —
same flow, same naming, same review bar as any other change. `main` is always assumed
releasable, so there's nothing a distinct branch type would add.

## Deploys

- Test and prod both deploy from `main`
- Deploy workflows and schedules are project-specific — check the repo's CI config and README
- App version bumps go through a normal small PR to `main`, like any other change

## Resources

| Topic                   | File                                                     | When to Use             |
| ----------------------- | -------------------------------------------------------- | ----------------------- |
| Branch protection rules | [`branch-protection.md`](resources/branch-protection.md) | Repo settings reference |
