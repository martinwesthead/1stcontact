---
uid: comment-a788edf8
id: COMMENT-40
type: comment
title: Comment on request REQ-33
created_by: xgd
created_at: '2026-07-03T18:27:30.099655+00:00'
updated_at: '2026-07-03T18:27:30.099655+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-f3342338
  kind: note
---

## Batch 4 — emphatic lines are callouts, not bold

Snapshot review (with the user) confirmed the two emphatic lines render as REQ-32 left-bar callouts, NOT bold paragraphs — the capture's borderLeft:null is bad extraction (same class as the footer/header colour fallbacks). We had mis-transcribed them as **bold** (strong->700).

- Site-def: mission-note -> '> [!primary] ...' (green bar); alchemy last paragraph -> '> [!accent italic] ...' (gold bar, italic). Adds size:lg to mission-note (16->18).
- Framework: callout text renders at medium (font-weight-medium / 500) — the reference emphasises callouts at 500, and a callout is the right single place to define that weight (site-defs cannot set raw weight by design). Applies to all callouts.

Rationale on 'why weight is hard': the framework forbids raw per-element font-weight (structured/token-backed, DOC-7); weight is a consequence of the semantic construct. Markdown only reaches 400 (plain) and 700 (**bold**) — no 'medium' primitive — so an arbitrary captured 500 has no home except inside a treatment. Modelling these as callouts puts the weight in one framework place.