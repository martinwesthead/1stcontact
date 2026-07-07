---
uid: comment-aa67131d
id: COMMENT-38
type: comment
title: Comment on request REQ-36
created_by: xgd
created_at: '2026-07-03T18:15:49.359311+00:00'
updated_at: '2026-07-03T18:15:49.359311+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-d05379d0
  kind: comment
---

First draft rendered + served (config-only, no framework code changed).

Built sandbox site 'joyfulculinary' — theme (gold #f8bb1b / pink #cc3366 palette, Oswald/Karla/Lato webfonts) + 9-band home.json transcribed verbatim from the capture: header(overlay,logo img) · hero(bg-image HERO, fold, scrim) · Holistic Approach(text-block) · Who Uses(text-block, inverse, checklist) · Our Offerings(services-grid 3-col) · veggie quote(hero bg-image) · Testimonials(services-grid 2-col) · How it works(services-grid 3-col, 6 cells) · footer(accent/gold). Served at http://localhost:8790/ . Screenshot: storage/sandbox/joyfulculinary/draft-shot.png.

Known fidelity gaps for pass 2 (mostly framework generalizations -> free-coding):
1. GOLD SECTION HEADINGS — every section heading in the original is gold #f8bb1b; text-block/services-grid headings inherit surface colour. Needs a headingTreatment dial generalized onto text-block + services-grid (mirror hero's). #1 brand signal.
2. DARK OFFERINGS/PROCESS BANDS — original renders these as gold-heading + white text COLUMNS on a black band (no white card panels). services-grid always draws white cards; on an inverse band card text is white-on-white (invisible). Shipped draft puts these two grids on LIGHT bands to be legible. Needs a bare/transparent card surface (or card-text fix) so grids work on dark bands.
3. GREEN TESTIMONIAL PANEL — original is one solid sage-green panel; draft approximates with neutral-cool cards (render light-grey, tint too weak). Needs a green surface role.
4. Hero heading uppercased via CSS text-transform in original; draft mixed-case (literal captured text kept per DOC-19).

Next: run '1c values-diff joyfulculinary --ref storage/references/joyfulculinarycreations.com/index --sandbox' to enumerate deltas mechanically, then file the generalizations (generalize-before-adding; candidate home REQ-32).