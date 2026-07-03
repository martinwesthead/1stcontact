---
uid: request-d05379d0
id: REQ-36
type: request
title: Faithful reproduction of joyfulculinarycreations.com (personal-chef site)
created_by: xgd
created_at: '2026-07-03T18:00:22.857118+00:00'
updated_at: '2026-07-03T18:04:37.548942+00:00'
completed_at: null
last_field_updated: priority
status: draft
fields:
  auto_merge_back: true
  needs_review: false
  priority: medium
---

## Goal

Reproduce the home page of **joyfulculinarycreations.com** (Chef Sarah Joy — holistic
in-home personal chef services) "indistinguishably to the eye" using our own modules,
following the runbook [[DOC-19]] and the diff-driven loop ([[REQ-31]]) — the same
exercise as the gigabytealchemy re-import ([[REQ-33]]). Reproduce **form** (layout,
type, colour, treatments), not brand/content ownership.

Starting scope: **home page only**, built in the gitignored sandbox tree.

## Source site (observed)

WordPress/Elementor site. Palette: gold `#f8bb1b`, pink accent `#cc3366`, dark
green panels, near-black section backgrounds, white text over food photography.
Fonts: Oswald (headings), Lato / Karla / Raleway (body/labels).

Page structure (top → bottom), from the capture + full-page screenshot:
1. **Header** — logo left; nav: Meet the Chef · Our Services · Sample Menus · FAQ · Get in Touch.
2. **Hero** — food-photo background, white heading "Dreaming of healthier meals / on your dinner table?", Lato subhead "Holistic In-Home Personal Chef Services for the busy family", gold "Learn More" button.
3. **The Holistic Approach** — light-grey panel, gold heading, prose.
4. **Who Uses Our Services** — dark band, gold heading, gold-tick checklist (5 items).
5. **Our Offerings** — dark band, gold headings, 3 service columns (Personal Chef Services, Postpartum Culinary Services, Cooking Classes) + "Learn More".
6. **Quote band** — food-photo background, white pull-quote, "Chef Sarah Joy / Owner".
7. **What People Are Saying** — green panel, testimonial carousel.
8. **Process grid** — dark band, gold icons, 6 cells (Visit Our FAQ Page, Questionnaire, Free Consultation, Home Visit, Menu Planning, Meal Prep) + "Get Started".
9. **Footer** — gold band, "Follow Us For Our Latest Updates", social icons, nav links.

## Progress

- [x] **Step 0 — Capture** `1c capture page https://joyfulculinarycreations.com/` →
  `storage/references/joyfulculinarycreations.com/index/` (REQ-31-complete bundle:
  capture.json + raw.html + rendered.html + screenshot.full.png + 83 assets).
  Note: segmenter grouped the page into 1 style-scope band; per-run values
  (colour/size/font/geometry) are captured, so this is a grouping artefact only
  (per [[DOC-13]]), not a fidelity loss.
- [x] **Sandbox scaffold** `1c new joyfulculinary --sandbox` →
  `storage/sandbox/joyfulculinary/draft/`.
- [ ] Structure pass — map captured sections to modules/variants.
- [ ] Values pass — transcribe colour/size/weight/gradient/etc. from raw.html.
- [ ] Treatment pass — scrims, panels, callouts, content anchor.
- [ ] Diff gate — `1c values-diff joyfulculinary --ref storage/references/joyfulculinarycreations.com/index --sandbox` to zero deltas.
- [ ] Eyes — `1c shot` + composition check.

## Notes

- Site-def / config / theme edits are exempt from free-coding ceremony; only
  framework *code* changes (a capability gap that can't be closed by config) need
  a scope ticket + UAT + `[FREE-CODED]` + version bump. Generalize before adding
  a module (CLAUDE.md); file any genuine gap against [[REQ-32]].
- `storage/references/` and `storage/sandbox/` are gitignored.
