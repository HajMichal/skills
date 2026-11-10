# Branch Protection Rules

## Protected Branch: `main`

`main` is the only long-lived branch. There is no `dev`.

| Rule | Setting |
|------|---------|
| Require PR before merging | ✅ Enabled |
| Required approvals | 1 minimum |
| Dismiss stale reviews on new push | ✅ Enabled |
| Require status checks to pass | ✅ Enabled |
| Require branch to be up to date | ✅ Enabled |
| Include administrators | ✅ Enabled |
| Allow force pushes | ❌ Disabled |
| Allow deletions | ❌ Disabled |

## Required Status Checks

- `build` — TypeScript compilation
- `lint` — ESLint
- `test` — Unit tests
- `test:e2e` — E2E tests (API)

## Common Mistakes

```bash
# ❌ Never do this
git push origin main
git push --force origin main

# ✅ Always create a branch and PR, named after the ticket
git checkout -b PROJ-42-add-order-endpoint
# ... make changes ...
git push origin PROJ-42-add-order-endpoint
# then open PR to main
```
