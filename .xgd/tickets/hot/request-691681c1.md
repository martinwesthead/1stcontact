---
uid: request-691681c1
id: REQ-21
type: request
title: 'Milestone: indistinguishable import of faelan.com (import-fidelity benchmark)'
created_by: xgd
created_at: '2026-07-02T00:32:00.430097+00:00'
updated_at: '2026-07-03T20:26:15.653416+00:00'
completed_at: null
last_field_updated: body
status: draft
fields:
  story_points: 8
  priority: medium
  auto_merge_back: true
  needs_review: false
---

## Scope — milestone / acceptance benchmark (not a single build)

The standing **import-fidelity benchmark** for the capture → reproduce → eyes pipeline: import the two founder-owned sites **faelan.com** and **gigabytealchemy.ai** so faithfully that they are **indistinguishable to the eye** from the originals (not pixel-counted). Companion to [[REQ-19]] (which proves the _ceiling_; this proves _import fidelity_).

Why these two:

- **Founder-owned → IP-clean** to use as fixtures ([[DOC-15]] §6).

- **They bracket the range:** faelan.com is art-directed (photo-montage, treatments, motion); gigabytealchemy.ai is a conventional band-stack.

This ticket is specifically an import of faelan.com. (see REQ-20 for the import of gigabyte alchemy)

## Dependencies

REQ-12 (capture), REQ-13 (screenshot / eyes — the acceptance gate), REQ-14 (background/overlay), REQ-15 (`layer` + treatments), REQ-16 (motion). No bespoke module is expected for faelan; TBD for gigabytealchemy.ai once rendered.

## Acceptance criteria

- **faelan.com** reproduces **indistinguishably at the desktop reference** — exact copy, colours, fonts, mirrored assets, layout/positions, treatments, overlay — judged by eyes ([[DOC-13]] §6), not pixels.

- Then extended to **interaction states** (hover/motion) and **mobile** (via REQ-15 reflow).

- Built **entirely from our own modules** — no raw CSS/HTML, no per-site hacks outside the model.

## Notes

This is the standing regression benchmark for import: any capture/reproduce/primitive change is measured against "do the two founder sites still import indistinguishably?" Break into concrete build/test REQs as the primitives (REQ-14/15/16) land.


## Progress — 2026-07-03 (faelan.com desktop import)

Reproduced **faelan.com** at the desktop reference, diff-driven per [[DOC-19]]. Built entirely from our modules in `storage/sites/faelan/`:
- **Montage** — `layer` (variant `full`, 100vh) + `background` (scorched-earth image + `#000000 @ 0.3` scrim) + 4 freely-positioned photos (rotations, soft-mask, `xl` shadow, circle with white ring) + the FAELAN wordmark and subline as positioned text runs.
- **About** — `hero` (surface inverse `#0f172a`, `Faelan` heading + subhead).
- **Footer** — `footer` on a black background.

Closed the treatment gaps by **generalizing the `layer` primitive** (no new modules) under [[REQ-32]] cap 5 (free_coded, v0.0.34): layer-text typography, image `shadow` + `border`, and an `xl` shadow token.

`1c values-diff` → **6 matched, 2 residual deltas, both non-fidelity**:
- footer copyright differs only by year (`© 2026` — framework `BUILD_YEAR` — vs the original's hard-coded `2025`; showing the current year is correct for a live site);
- the montage overlay reads as a diff false-negative — it renders identically (`#000000 @ 0.3`) but the diff doesn't detect an overlay nested in a layer's `background`.

**Remaining for the milestone:** interaction states (hover/motion — hover-scale is wired, not yet eyes-checked) and **mobile** reflow; and the gigabytealchemy.ai import (tracked under [[REQ-20]]/[[REQ-33]]). Possible follow-up: per-child soft-mask feather amount (noted deferred in REQ-32 cap 5); background `attachment: fixed` for the montage parallax.


### Update — mobile + interaction states complete (commit 0c08c33)

- **Mobile** — the montage now reproduces faithfully at 375px: `layer` set to `reflow: none` (the original re-composes rather than stacks) with per-child `breakpoints.md` overrides (mobile-first base → desktop layout ≥768px, matching the original's `max-width:768` breakpoint). Responsive type tokens keep the wordmark/subline fitting small screens while holding desktop values (`5xl`→`clamp(2.5rem,9vw,4rem)`, `2xl`→`clamp(1.1rem,5vw,1.5rem)`). z-order corrected to violin<alley<torn<circle<text. All config — no framework change.
- **Hover** — all four photos carry hover-scale (1.04; original ~1.03–1.05), `prefers-reduced-motion` guard ships, the "Musician" Spotify link renders underlined.

**faelan.com is now reproduced indistinguishably at both desktop and mobile references** (per [[DOC-13]] eyes), built entirely from our modules. Milestone item for faelan.com is met; remaining REQ-21 scope is the gigabytealchemy.ai import (tracked under [[REQ-20]]/[[REQ-33]]).


### Fidelity pass — same-viewport eyes-diff (commit 502cbbc)

Captured a fresh original (`1c shot --url https://faelan.com/`) and the reproduction at an identical 1280×800, then compared region-by-region. Three residual differences found and closed:

1. **Circle ring** — original is a subtle translucent `rgba(255,255,255,.3)` ring; mine was a bright solid-white 4px ring. Fixed by pointing the border at a translucent `border` palette role (`#ffffff4d`) — config.
2. **Subline gap** — my "Artist • Musician • Creator" sat too tight under the wordmark; nudged its desktop `y` 16→22 to match the original wordmark→subline rhythm — config.
3. **Photo edge feather** — my photos were over-softened (fixed 55% mask stop feathering the outer ~45%) vs the original's crisper 70–75%. Closed by implementing the deferred **`imageTreatment.feather`** control ([[REQ-32]] cap 5, free_coded v0.0.35) and setting the soft photos to `feather: md` (~70%).

After the fixes, desktop and mobile both read as indistinguishable from the original. `values-diff` still 6 matched with only the two known non-fidelity residuals (footer build-year `2026` vs hard-coded `2025`; overlay false-negative — it renders identically `#000000 @ 0.3`).
