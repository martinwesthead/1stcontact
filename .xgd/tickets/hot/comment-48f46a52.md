---
uid: comment-48f46a52
id: COMMENT-29
type: comment
title: Comment on request REQ-32
created_by: xgd
created_at: '2026-07-03T02:10:22.279497+00:00'
updated_at: '2026-07-03T02:10:22.279497+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-342de4f4
  kind: note
---

## Implementation design (free-coding — decided)

Following the services-grid precedent (semantic palette-role enums → token-backed scoped CSS; structured treatments in `content`; computed inline style only where values are genuinely free, per background.ts). Interpretation choices flagged for visibility:

1. **Gradient treatment.** New optional `content.logoGradient` (header) / `content.headingGradient` (hero), selected by a new `gradient` value on the existing `logoTreatment`/`headingTreatment` dials. Shape: `{ direction: enum, stops: [{ role: <palette-role enum> }] }`. Stops reference **palette roles only** (fully token-backed — no raw hex, no raw CSS); an off-palette hue is a site's palette addition (REQ-20 config), keeping this ticket pure expressiveness. `direction` is an 8-way enum (`to-right`, `to-br`, …) rather than a raw angle — stays enumerated/validated like `motionEasing`; covers the practical space (interpretation of "arbitrary direction"). Framework computes `linear-gradient(...)` + `background-clip:text` via a shared `gradient.ts` helper. Existing `gold` preset untouched.

2. **Callout / left-bar.** GFM-alert markdown syntax (`> [!accent] …`, `> [!secondary italic] …`) transformed in the shared `renderMarkdown` → `blockquote.fc-callout.fc-callout--<role>` (+ `--italic`). Available in ANY markdown body (text-block, hero subhead, grid), not just text-block. Styling via a shared `CALLOUT_CSS` constant folded into the stylesheet. Role is a closed set.

3. **Hero scrim + anchor.** `scrim` dial (`none`/`light`/`medium`/`strong` → opacity of a dark neutral token) + `contentAnchor` dial (`top`/`center`/`bottom`). Enumerated; generic to any text-over-image hero.

4. **Cool-neutral role.** New optional `neutralCool` palette token (defaulted, like `secondary`), exposed as a selectable role for gradient stops, callouts, and services-grid card accent/surface.

UATs (`test_UAT_FC_REQ-32_*`) assert each capability on **synthetic values** (accent→secondary, etc.), never gigabyte's hexes.
