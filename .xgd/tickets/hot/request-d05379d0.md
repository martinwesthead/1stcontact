---
uid: request-d05379d0
id: REQ-36
type: request
title: Faithful reproduction of joyfulculinarycreations.com (personal-chef site)
created_by: xgd
created_at: '2026-07-03T18:00:22.857118+00:00'
updated_at: '2026-07-08T18:37:32.339865+00:00'
completed_at: null
last_field_updated: body
status: draft
fields:
  auto_merge_back: true
  needs_review: false
  priority: medium
  commits:
  - 9e9fda0
---

## Goal

Reproduce the home page of **joyfulculinarycreations.com** (Chef Sarah Joy — holistic
in-home personal chef services) "indistinguishably to the eye" using our own modules,
following the runbook [[DOC-19]] and the diff-driven loop ([[REQ-31]]) — the same
exercise as the gigabytealchemy re-import ([[REQ-33]]). Reproduce **form** (layout,
type, colour, treatments), not brand/content ownership.

This ticket is the **first worked example of [[DOC-21]]** (Reproduction-Driven Framework
Growth Loop). Per operator decision, the framework changes this reproduction forces are
free-coded **in this ticket** rather than spun out to separate REQs. This ticket states
the *requirement*; capability-matrix artifacts (capabilities / stories / ACs / UAT
coverage) are derived later by reconciliation — this session does not author them.

Starting scope: **home page only**, built in the gitignored sandbox tree.

## Source site (observed)

WordPress/Elementor site. Palette: gold `#f8bb1b`, pink accent `#cc3366`, dark
green panels, medium-grey (`#7a7a7a`) section bands, white text over food photography.
Fonts: Oswald (headings), Lato / Karla / Raleway (body/labels). Verified band colours:
grey bands `#7A7A7A`; testimonial `#28542D9E`; quote scrim `#141E14BA` over
market-vegetables photo; hero HERO-AdobeStock image at 0.49 over black; footer gold.

Page structure (top → bottom), from the capture + full-page screenshot:
1. **Header** — logo left; nav: Meet the Chef · Our Services · Sample Menus · FAQ · Get in Touch.
2. **Hero** — food-photo background, white UPPERCASE heading "Dreaming of healthier meals / on your dinner table?", Lato subhead, divider rule, gold "Learn More" button.
3. **The Holistic Approach** — light-grey panel, gold heading, prose.
4. **Who Uses Our Services** — grey band, gold heading, gold-tick checklist (5 items).
5. **Our Offerings** — grey band, gold headings, affirmations column + 3 service columns + "Learn More".
6. **Quote band** — food-photo background, white pull-quote, "Chef Sarah Joy / Owner".
7. **What People Are Saying** — green panel, testimonial carousel.
8. **Process grid** — grey band, gold icons, 6 cells + "Get Started".
9. **Footer** — gold band, "Follow Us For Our Latest Updates", social icons, nav links.

## Framework changes required (free-coded in this ticket)

Gaps that site config/theme cannot close (confirmed by the perceptual `1c diff` overlays).
Each is a **generalization of an existing module** (generalize before adding a module —
CLAUDE.md / [[DOC-21]] §5), free-coded here with `test_UAT_FC_REQ-36_*` UATs. Site-def /
config / theme edits remain exempt from free-coding; framework *code* changes take the
full ceremony (code + UAT + `[FREE-CODED]` + version bump).

1. **Bare card surface on services-grid** — a grid-wide `cardSurface: bare` dial strips
   the card fill/border/radius/padding so cards read as plain text columns on a dark band;
   chrome (badge / accent border) suppressed; orthogonal to `variant`. **DONE** — commit
   9e9fda0, v0.0.66. joyfulculinary diff mean 64.57 → 46.68.
2. **Gold heading treatment on text-block + services-grid** — generalize hero's
   `headingTreatment: gold` so section headings and card titles render in the accent gold.
3. **Uppercase heading treatment** — a `headingCase: upper` dial (hero + text-block +
   services-grid) that renders uppercase while leaving the DOM text node literal (so the
   values-diff text check stays clean).
4. **Role-driven panel surface** — let a band take a theme-defined panel colour via a
   palette *role* (not a colour-named `surface: green`), for the green testimonial band.

## Progress

- [x] **Step 0 — Capture** → `storage/references/joyfulculinarycreations.com/index/`
  (REQ-31 bundle). Segmenter grouped into 1 style-scope band (grouping artefact per
  [[DOC-13]]); per-element value manifest intact.
- [x] **Sandbox scaffold** → `storage/sandbox/joyfulculinary/draft/`.
- [x] **Pass 2 — structure + values + treatment** built config-only (theme + 9 bands),
  verbatim text, all 4 fonts loaded, exact band colours.
- [x] **Both gates run** (DOC-21 steps 2–3): `values-diff` 54 matched / 142 deltas;
  `1c diff` perceptual **mean 64.57 / 255** at config-exhaustion → framework changes needed.
- [x] **Bare cards** → implemented + UATs + re-diff (mean 46.68). commit 9e9fda0.
- [ ] **Gold headings** → implement + UATs + re-diff.
- [ ] **Uppercase headings + hero polish** → implement + UATs + re-diff.
- [ ] **Role-driven panel surface** → implement + UATs + re-diff.
- [ ] **Good-Enough gate** (DOC-21 §4) at 3 viewports.

## Notes

- Site-def / config / theme edits are exempt from free-coding; framework *code* changes
  are free-coded (code + UAT + `[FREE-CODED]` + version bump). Generalize before adding a
  module (CLAUDE.md / [[DOC-21]] §5).
- `storage/references/` and `storage/sandbox/` are gitignored.
