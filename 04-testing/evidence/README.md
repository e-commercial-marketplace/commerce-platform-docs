# Test Evidence

This directory stores manually captured evidence for the frontend automated test suite.

No screenshot is committed yet. The files referenced by the plan below do **not** currently exist.

## Required evidence

| Evidence | Suggested filename | Captured from |
|---|---|---|
| Jest PASS summary | `jest-pass-<commit>.png` | `npm test` output in `commerce-platform-frontend-mobile` |
| Coverage summary | `jest-coverage-<commit>.png` | `npm run test:coverage` output |
| Typecheck pass | `typecheck-pass-<commit>.png` | `npm run typecheck` output |
| Optional app behavior | `app-cart-empty-<commit>.png` | Running app, empty cart state |

## Rules

- Label every captured item with the frontend commit hash and the capture date.
- Only commit real captures. Never commit a placeholder as if it were a real result.
- Evidence is point-in-time and tied to the frontend revision in
  [`../README.md`](../README.md) section 6.
- If the frontend revision changes, previous evidence remains valid only for the commit it recorded.
