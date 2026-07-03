---
uid: request-b56dddb3
id: REQ-30
type: request
title: services-grid stacked (full-width) card variant
created_by: xgd
created_at: '2026-07-02T21:52:54.641692+00:00'
updated_at: '2026-07-02T22:42:14.720342+00:00'
completed_at: null
last_field_updated: body
status: draft
fields:
  priority: low
  story_points: 2
  auto_merge_back: true
  needs_review: false
---

## Scope — module capability

`services-grid` offers only multi-column layouts (`three-col`/`two-col`). There is no stacked/full-width variant where each card spans the band on its own row.

Surfaced by the **gigabytealchemy.ai** import (REQ-20): the reference "Building" section stacks its cards full-width, one per row; ours renders them side-by-side because no stacked variant exists.

## Acceptance criteria

- `services-grid` exposes a stacked / single-column (full-width card) layout variant selectable per instance.
- No raw CSS in the site definition.
- gigabytealchemy "Building" cards render stacked full-width, matching the captured layout.

## Related

Card treatments (accent border / badge / checklist) are REQ-26; this ticket is purely the stacked *layout* variant.


## Consolidated into REQ-28 (2026-07-02)

Per operator steer ("use fewer reqs"), the services-grid `stacked` variant was implemented under the REQ-28 umbrella (small module dials for the gigabytealchemy import). Archived — see REQ-28.
