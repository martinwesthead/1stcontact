---
uid: request-691681c1
id: REQ-21
type: request
title: 'Milestone: indistinguishable import of faelan.com + gigabytealchemy.ai (import-fidelity
  benchmark)'
created_by: xgd
created_at: '2026-07-02T00:32:00.430097+00:00'
updated_at: '2026-07-02T00:32:00.430097+00:00'
completed_at: null
last_field_updated: created_at
status: draft
fields:
  story_points: 8
  priority: medium
  auto_merge_back: true
  needs_review: false
---

## Scope — milestone / acceptance benchmark (not a single build)

The standing **import-fidelity benchmark** for the capture → reproduce → eyes pipeline: import the two founder-owned sites **faelan.com** and **gigabytealchemy.ai** so faithfully that they are **indistinguishable to the eye** from the originals (not pixel-counted). Companion to [[REQ-19]] (which proves the *ceiling*; this proves *import fidelity*).

Why these two:
- **Founder-owned → IP-clean** to use as fixtures ([[DOC-15]] §6).
- **They bracket the range:** faelan.com is art-directed (photo-montage, treatments, motion); gigabytealchemy.ai is a conventional band-stack.

## Dependencies
REQ-12 (capture), REQ-13 (screenshot / eyes — the acceptance gate), REQ-14 (background/overlay), REQ-15 (`layer` + treatments), REQ-16 (motion). No bespoke module is expected for faelan; TBD for gigabytealchemy.ai once rendered.

## Acceptance criteria
- **faelan.com** and **gigabytealchemy.ai** each reproduce **indistinguishably at the desktop reference** — exact copy, colours, fonts, mirrored assets, layout/positions, treatments, overlay — judged by eyes ([[DOC-13]] §6), not pixels.
- Then extended to **interaction states** (hover/motion) and **mobile** (via REQ-15 reflow).
- Built **entirely from our own modules** — no raw CSS/HTML, no per-site hacks outside the model.

## Notes
This is the standing regression benchmark for import: any capture/reproduce/primitive change is measured against "do the two founder sites still import indistinguishably?" Break into concrete build/test REQs as the primitives (REQ-14/15/16) land.
