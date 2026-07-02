---
uid: request-3ad342ca
id: REQ-22
type: request
title: Consolidate site-data trees under storage/
created_by: xgd
created_at: '2026-07-02T16:36:13.154059+00:00'
updated_at: '2026-07-02T16:36:13.679243+00:00'
completed_at: null
last_field_updated: body
status: draft
fields:
  story_points: 2
  priority: medium
  auto_merge_back: true
  needs_review: false
---

## Scope

Consolidate the four site-data trees under a single top-level **`storage/`** directory to declutter the repo root (clean code / data split):

```
storage/
  sites/        (git-tracked — authored sites)
  sandbox/      (gitignored — scratch)
  dist/         (gitignored — rendered output)
  references/   (gitignored — captured external sites)
```

Purely a **path/layout refactor** — no behaviour change. Touches the centralized path builders, `.gitignore`, tests, and the layout in the docs. Done now, before the flagship sites and `references/` captures accumulate content that would have to be moved later.

## Deliverables

- `tools/generate/src/store/paths.ts` — `siteDir` and `distDir` resolve under `storage/`.
- `tools/generate/src/cli/capture/bundle.ts` — capture bundles write under `storage/references/`.
- `.gitignore` — ignore `/storage/{sandbox,dist,references}/`; `storage/sites/` stays tracked.
- `git mv sites storage/sites` (and `dist` → `storage/dist`) to preserve history / move existing content.
- Update path assertions in existing UATs (REQ-9/11/12/13 tests).
- Update the layout in [[DOC-12]], [[DOC-13]], [[DOC-15]].

## UATs (`test_UAT_FC_REQ-22_*`)
- `test_UAT_FC_REQ-22_new_and_render_use_storage` — `1c new` creates under `storage/sites/<slug>/`, `1c render` outputs under `storage/dist/sites/<slug>/`.
- `test_UAT_FC_REQ-22_capture_writes_under_storage` — a capture bundle lands under `storage/references/<host>/`.
- `test_UAT_FC_REQ-22_gitignore_tracks_sites_ignores_rest` — `storage/sites/**` tracked; `storage/{sandbox,dist,references}/**` ignored.

## Out of scope
Any behaviour change; D1 paths (later).
