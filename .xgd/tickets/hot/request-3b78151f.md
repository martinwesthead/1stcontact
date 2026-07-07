---
uid: request-3b78151f
id: REQ-44
type: request
title: 'Tooling hygiene: pnpm install after lockfile change; fail loud on out-of-sync
  node_modules'
created_by: xgd
created_at: '2026-07-03T23:31:56.269585+00:00'
updated_at: '2026-07-03T23:31:56.269585+00:00'
completed_at: null
last_field_updated: created_at
status: draft
fields:
  auto_merge_back: true
  needs_review: false
  priority: medium
---

## Problem

`node_modules` can drift from the committed `pnpm-lock.yaml`, silently breaking the `1c` browser tooling. Observed this session: the REQ-38 `1c diff` work added `sharp` to `tools/generate/package.json` + the lockfile (commit `b76cf7f`), but the operator's working tree was never re-installed. `node_modules` lagged the lockfile, a reconcile pruned `playwright` (a *declared* dep), and `1c shot` / `1c diff` / `1c capture` all failed with `Cannot find module 'playwright'` (Vite logged "lockfile has changed"). A manual `pnpm install` fixed it and changed **no** tracked file — confirming the manifests were correct all along; only the on-disk install was stale.

Root cause: **declaring a dependency (package.json + lockfile) does not materialize it** — that needs `pnpm install`. When a dep lands via a workflow commit but no install follows, `node_modules` silently drifts.

## Ask

- Any XGD step / reconcile that edits `package.json` or `pnpm-lock.yaml` should run (or require) `pnpm install` before the next render/test/shot.
- A stale/out-of-sync `node_modules` (missing a declared dep, or lockfile-newer-than-install) should **fail loudly** at the start of `1c` browser commands, with a clear "run `pnpm install`" message — not silently prune or crash mid-render.
- Consider a cheap preflight in the `1c` CLI: verify the browser deps resolve before launching Vite/Playwright.

## Evidence
Surfaced during the faelan reproduction ([[REQ-21]]); related tooling: [[REQ-38]] (`1c diff`, added `sharp`), the generate CLI.
