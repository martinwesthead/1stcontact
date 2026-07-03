---
uid: request-342de4f4
id: REQ-32
type: request
title: Framework fidelity primitives surfaced by import (gradient direction, callout
  left-bar, hero scrim/anchor, cool-neutral role)
created_by: xgd
created_at: '2026-07-03T01:37:59.346134+00:00'
updated_at: '2026-07-03T20:25:36.058146+00:00'
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
  - 25f1c91afbba502e294a1481334667c59d3196a5
  - 502cbbc56a0db4a465764a8f362e08b57ba15572
  version: 0.0.35
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


## Capability 5 — Layer art-direction treatments (surfaced by the faelan.com import, [[REQ-21]])

The `layer` primitive (REQ-15) can position/rotate/mask freely-placed children, but an art-directed photo-montage (faelan.com) needs three more **generic, token-backed** treatments before it reads as intended. All are generalizations of the existing `layer` child — no new module (CLAUDE.md rule). Site-specific values (exact shadow, which role, which size) remain config under [[REQ-21]]; this ticket is only the expressiveness.

1. **Layer text-child typography.** A positioned `text` child renders unstyled markdown (inherits body size), so a wordmark/label can't be sized. Add a structured, token-backed `typography` field to the layer text child — `size` (font-scale step), `weight`, `color` (palette role), `font` (heading/body/display), `tracking` (closed enum → em), `align`, and a legibility `shadow` (bool). Framework-computed custom properties only; no raw CSS. *(Driver: faelan "FAELAN" wordmark 64px/900 + subline over imagery.)*

2. **Layer image-child `shadow`.** Montage photos need to lift off the background. Add a `shadow` treatment on the image child = a step bound to the theme shadow tokens. Introduces an `xl` shadow token (optional; defaulted) for a lifted drop+glow — the exact value is per-site config. *(Driver: faelan photos `0 15px 50px …, 0 0 30px …`.)*

3. **Layer image-child `border`.** A framed/ringed photo needs a token-backed border — `{ width: none|thin|medium|thick, color: <palette-role> }` → `border: <px> solid var(--color-<role>)`. *(Driver: faelan circular portrait's light ring.)*

Deferred (note, not in scope here): per-child soft-mask feather amount — the fixed radial stop is close enough at the desktop reference; revisit if eyes flag it.

### Acceptance
- Layer text children can carry structured typography; image children can carry `shadow` + `border`; all token-backed, `.strict()` preserved, no raw CSS reaches the page.
- `test_UAT_FC_REQ-32_*` cover: text typography → font-size/weight/color/tracking/shadow custom props; image `shadow` → `var(--shadow-*)`; image `border` → `<px> solid var(--color-<role>)`.
- faelan montage renders the wordmark at display scale and the photos lifted/ringed (eyes vs the captured reference).


### Capability 5 follow-up — soft-mask `feather` (now in scope, was deferred)

Same-viewport eyes-diff against the faelan reference (REQ-21) showed the layer photos over-softened: the fixed soft-mask stop (`#000 55%`) feathers the outer ~45% of each image, vs the original's crisper 70–75% stop. Bringing the deferred per-child feather control into scope:

- `imageTreatment.feather: sm | md | lg` (soft-mask only) → the radial black-stop, emitted as a framework-computed `--fc-feather` custom property the mask reads (`sm`=82% crisp, `md`=70%, `lg`=55% = the prior default). Default unchanged when absent, so existing soft-mask behaviour is untouched.

Adds a `test_UAT_FC_REQ-32_*` for the feather custom property + default fallback.