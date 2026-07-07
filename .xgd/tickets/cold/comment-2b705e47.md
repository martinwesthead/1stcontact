---
uid: comment-2b705e47
id: COMMENT-39
type: comment
title: Comment on request REQ-36
created_by: xgd
created_at: '2026-07-03T18:25:45.998240+00:00'
updated_at: '2026-07-03T18:25:45.998240+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-d05379d0
  kind: comment
---

CONFIG CEILING REACHED — values-diff enumeration.

Ran '1c values-diff joyfulculinary --ref storage/references/joyfulculinarycreations.com/index --sandbox'. Config-only draft: 51 matched, 187 delta(s). Applied the only genuine config wins (verbatim casing 'Visit our FAQ page'; added omitted 'So fabulous' testimonial) — count moved 188->187, empirically confirming config is at its ceiling.

Delta breakdown (187): fontSizePx 46 · color 36 · fontWeight 36 · lineHeightPx 30 · fontFamily 18 · missing 12 · paddingLeftPx 6 · letterSpacingPx 2 · contentAnchor 1 · text 1.

CODE-TIER (~163, need framework generalizations -> free-coding, candidate home REQ-32):
- color 33/36: 7 = gold section headings (#f8bb1b) -> headingTreatment dial on text-block+services-grid (mirror hero's); 26 = white body text on the dark Offerings/Process bands -> those grids are on LIGHT bands in the draft because services-grid draws white cards (white-on-white on inverse). Needs a bare/transparent card surface so grids read on a dark band. These two also close the biggest VISUAL gaps.
- fontSizePx 46 + fontWeight 36 + lineHeightPx 30 = 112: modules use a fixed type scale + heading weight; original uses specific px (44/32/28/19/18/17) and light Oswald weights (200/300). Needs per-element type control — the largest, most architectural bucket.
- fontFamily 18: original mixes 4 faces per-element (Raleway labels, Lato hero subhead, Oswald footer nav, Karla process titles); theme has 3 family slots. Needs a 4th slot + per-module font assignment.

CONFIG-TIER remainder (~24, mostly non-fidelity artifacts):
- missing 12: my intentional merges (hero heading split into 2 nodes; Dan H./Parent-CFO merged into card title; quote modeled as hero heading not body) + punctuation normalization (curly quotes, zero-width ​) + 2 genuine omissions (Get Started, Offerings 'Learn More'). Content is present; these are DOM-segmentation mismatches, not losses.
- color 3: muted grey #54595f vs #333 (near-identical). paddingLeft 6 / letterSpacing 2 / contentAnchor 1: cosmetic micro-deltas.

Verdict: structurally + content complete via config; exact-value fidelity (colour of headings, dark-band drama, px sizes, weights, 4th font) is gated on framework code. Green testimonial panel also still code (neutral-cool tint too light).