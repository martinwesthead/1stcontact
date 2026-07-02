---
uid: request-22aa8ea3
id: REQ-19
type: request
title: 'Milestone: 1stcontact.io at sycamore.so-level (ceiling proof)'
created_by: xgd
created_at: '2026-07-02T00:20:11.447007+00:00'
updated_at: '2026-07-02T00:20:11.955037+00:00'
completed_at: null
last_field_updated: body
status: draft
fields:
  story_points: 8
  priority: medium
  auto_merge_back: true
  needs_review: false
---

## Scope — milestone / acceptance goal (not a single build)

The corollary of [[DOC-15]] §4 and the public demonstration of [[DOC-4]]: **rebuild 1stcontact.io to sycamore.so-level using only our own modules**, proving the language + eyes reach the expressive ceiling. This is a **milestone**, gated on the flashy primitives — not a discrete feature.

## Dependencies
`layer` module, motion primitive (and later a generative-visual primitive), REQ-14 (background). The framework catalog must be rich enough first.

## Acceptance criteria
- The 1st Contact marketing site is **indistinguishable in polish** from a sycamore-class site (motion, layered composition, art direction) — judged by eyes, not pixels.
- Built **entirely from our own modules** (Tier A + any hardened Tier-B modules) — no raw CSS/HTML, no per-site hacks outside the model.
- Passes our own quality gates and renders through the standard `1c` pipeline.

## Notes
Serves as the **ceiling-proof driver** for the module roadmap, counterbalancing the coverage histogram (which alone would never prioritize wow-factor primitives — [[DOC-15]] §4). Break down into concrete build REQs once the underlying primitives exist.
