---
uid: request-284b9873
id: REQ-15
type: request
title: 'Framework: `layer` module + z-compositing (free-positioned structured layout)'
created_by: xgd
created_at: '2026-07-02T00:19:58.473214+00:00'
updated_at: '2026-07-02T00:19:58.974241+00:00'
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

The first concrete "grow the language" primitive ([[DOC-7]] §6, [[DOC-14]]): a **`layer` module** — a stack of freely-positioned children (images, text blocks, other modules) — plus **z-compositing** so a layer can sit over another module. All positioning is **structured data**, never raw CSS. Unlocks art-directed heroes (e.g. faelan.com's photo montage) while staying inside the model.

## Dependencies
REQ-3 (schema — adds structured layout types), REQ-4/5 (framework), REQ-14 (background/overlay). 

## Deliverables
- **Schema:** `Position` (`x`/`y`/`z`, size, per-breakpoint overrides), `Layer` (ordered positioned children), image **treatments** (`shape: circle`, `edge: soft-mask | torn-asset`), and a `reflow` fallback (stack on narrow viewports). Validated as structured data.
- **Framework rendering:** scoped, class/custom-property CSS generated from the structured props (never instance-supplied CSS); responsive via per-breakpoint values + reflow; z-compositing over a sibling/parent module.
- **Validator:** accepts structured positions/treatments; rejects raw CSS/HTML.

## UATs (`test_UAT_FC_REQ-15_*`)
- `test_UAT_FC_REQ-15_layer_positions_children` — children render at their structured x/y/z.
- `test_UAT_FC_REQ-15_text_over_image` — text renders over a layered image + overlay.
- `test_UAT_FC_REQ-15_treatments` — circle / soft-mask / torn-asset treatments render.
- `test_UAT_FC_REQ-15_per_breakpoint_reflow` — a narrow viewport reflows to the stack fallback.
- `test_UAT_FC_REQ-15_zcompositing` — a layer composites over another module.
- `test_UAT_FC_REQ-15_no_raw_css` — raw CSS/style props on an instance are rejected.

## Out of scope
Motion (separate REQ); generative/canvas visuals (later).
