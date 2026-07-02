---
uid: request-727a9613
id: REQ-16
type: request
title: 'Framework: structured motion primitive (entrance / scroll-reveal / hover)'
created_by: xgd
created_at: '2026-07-02T00:20:01.871249+00:00'
updated_at: '2026-07-02T00:20:02.398370+00:00'
completed_at: null
last_field_updated: body
status: draft
fields:
  story_points: 5
  priority: medium
  auto_merge_back: true
  needs_review: false
---

## Scope

The highest-leverage language-growth primitive (surfaced by the sycamore.so analysis): a **structured motion system** — entrance animations, scroll-triggered reveals, hover states, stagger — expressed as **structured params** (which / trigger / duration / easing / delay), rendered server-side as CSS + a minimal Astro island for scroll triggers. No raw CSS ([[DOC-7]] §6).

## Dependencies
REQ-4 (framework), REQ-3 (schema — motion fields).

## Deliverables
- **Schema:** a `Motion` type attachable to modules/children — `type` (fade / slide / scale / stagger), `trigger` (load / scroll / hover), `duration`, `easing`, `delay`.
- **Framework rendering:** CSS keyframes/transitions from the params; an island for scroll-triggered reveals; `prefers-reduced-motion` respected.

## UATs (`test_UAT_FC_REQ-16_*`)
- `test_UAT_FC_REQ-16_entrance_and_hover_render` — load/hover motions render from params.
- `test_UAT_FC_REQ-16_scroll_reveal_hydrates` — scroll-triggered reveal works via the island.
- `test_UAT_FC_REQ-16_reduced_motion` — `prefers-reduced-motion` disables/animates minimally.
- `test_UAT_FC_REQ-16_params_drive_output` — duration/easing/delay flow into the output.

## Out of scope
Generative/canvas animation (later); physics.
