---
uid: request-342de4f4
id: REQ-32
type: request
title: Framework fidelity primitives surfaced by import (gradient direction, callout
  left-bar, hero scrim/anchor, cool-neutral role)
created_by: xgd
created_at: '2026-07-03T01:37:59.346134+00:00'
updated_at: '2026-07-03T02:20:17.131651+00:00'
completed_at: null
last_field_updated: status
status: free_coded
fields:
  priority: medium
  story_points: 5
  auto_merge_back: true
  needs_review: false
  commits:
  - b54f9f6c0ab4ef1667ee56c2baed9cf95fff75b1
  version: 0.0.24
---

## Scope

Generic, reusable **framework capabilities** the import pipeline needs but cannot yet express. Surfaced *by* — not scoped *to* — the gigabytealchemy repro; each applies to any site. Follows the framing of [[REQ-24]]/[[REQ-25]]/[[REQ-26]]/[[REQ-28]] (capabilities driven by the import, not named after it).

**Explicitly out of scope: site-specific values.** Exact colours/sizes (e.g. gigabytealchemy's wordmark size, hero body-text hex, footer text colour) are *config* under the site milestone [[REQ-20]], and are flagged mechanically by [[REQ-31]]'s values-diff. This ticket is only the missing **expressiveness**.

## Capabilities

1. **Heading / wordmark gradient treatment — arbitrary direction + multi-stop hues.** The current `gold` treatment is a fixed *vertical two-tone*; a gradient wordmark/heading with a horizontal (or any-angle) multi-hue sweep can't be expressed. Generalise the treatment to carry direction + stops as structured, token-backed values. *(Driver: gigabytealchemy wordmark, gold→orange, 90deg.)*

2. **`text-block` callout / left-bar treatment.** A semantic accent **left-bar + indent + optional italic** (blockquote-style emphasis) as a structured treatment, not markdown bold. Any site with pull-quote / callout emphasis needs it. *(Driver: gigabytealchemy emerald + amber callouts.)*

3. **Hero overlay-scrim + content vertical anchor.** A legibility **scrim** over the background image (opacity tint) plus a **content anchor** (top / center / bottom). Common to any text-over-image hero. *(Driver: gigabytealchemy slate scrim, content anchored low.)*

4. **Cool-neutral palette role.** The palette carries a single, often-warm `muted`; a site that mixes a **cool neutral (slate)** for panels/borders can't express it. Add a cool-neutral role (or let panel/border tints select a neutral independent of `muted`). *(Driver: gigabytealchemy slate exploring panel vs our warm-neutral derivation.)*

## Notes

- Bundled per the "fewer, generic tickets" preference; each is a small primitive. Split only if a single one grows.
- Reusable across all sites and the future builder; gates [[REQ-20]] / [[REQ-21]] fidelity.
- The runbook [[DOC-19]] points here for "known still-missing capabilities."


## Implementation note

Implement each capability by **generalizing the existing module**, not by adding a new one (project rule — CLAUDE.md "Generalize Modules Before Adding New Ones"): (1) heading gradient → generalize the hero/header `gold` treatment to carry direction+stops; (2) callout → a `text-block` treatment; (3) scrim/anchor → hero dials; (4) cool-neutral → a palette role. No new modules expected.