---
uid: comment-ecff3567
id: COMMENT-30
type: comment
title: Comment on request REQ-20
created_by: xgd
created_at: '2026-07-03T02:18:57.321849+00:00'
updated_at: '2026-07-03T02:18:57.321849+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-97357769
  kind: note
---

Wordmark casing (config; fixed in draft, verified visually):
- Capture literal text is mixed-case 'Gigabyte Alchemy' with font-family Cinzel and NO text-transform. Cinzel's lowercase glyphs are small capitals, so the true capitals G and A render full-height and the rest as smaller caps — the 'larger G/A' effect.
- I had set logo='GIGABYTE ALCHEMY' (uppercased), which makes every letter a full capital and flattens the effect. Fixed: logo='Gigabyte Alchemy'. No CSS/capability needed — the effect is inherent to Cinzel + verbatim casing.
- Third instance of the same root cause (transcribe-don't-reconstruct): normalized the literal text instead of copying it verbatim. Casing carried a typographic design.