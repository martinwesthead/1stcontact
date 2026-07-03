---
uid: request-1c6e3630
id: REQ-29
type: request
title: Header content alignment dial (centered wordmark)
created_by: xgd
created_at: '2026-07-02T21:52:46.354173+00:00'
updated_at: '2026-07-02T22:42:14.051787+00:00'
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

`header` has no alignment dial: the wordmark/nav render left-aligned with no structured way to center (or right-align) the content.

Surfaced by the **gigabytealchemy.ai** import (REQ-20): the reference header centers the "GIGABYTE ALCHEMY" wordmark over the hero band; ours is left-aligned.

## Acceptance criteria

- `header` exposes a structured `align` dial (`left` | `center` | `right` or equivalent) controlling wordmark/nav placement.
- No raw CSS in the site definition.
- gigabytealchemy wordmark renders centered, matching the captured layout.

## Related

Composes with REQ-25 (header-over-hero shared image band) and REQ-24 (Cinzel gold wordmark) to complete the reference header.


## Consolidated into REQ-28 (2026-07-02)

Per operator steer ("use fewer reqs"), the header `align` dial was implemented under the REQ-28 umbrella (small module dials for the gigabytealchemy import). Archived — see REQ-28.
