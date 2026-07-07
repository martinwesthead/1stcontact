---
uid: comment-54ac266f
id: COMMENT-27
type: comment
title: Comment on request REQ-20
created_by: xgd
created_at: '2026-07-03T01:55:37.704122+00:00'
updated_at: '2026-07-03T01:55:37.704122+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-97357769
  kind: note
---

Fidelity config values (milestone scope here, NOT [[REQ-32]] — which is now generic framework primitives only). These are gigabytealchemy-specific values to set in the site def, and are flagged automatically by [[REQ-31]]'s values-diff:
- Wordmark size: capture is md:text-7xl (72px); currently logoSize=lg. Set to xl (or add a 7xl step if xl underscales).
- Hero subhead + body colour: capture inline #F5E6A3 (pale gold); currently white. Set hero body/subhead colour.
- Hero heading colour: capture #FBBA72 (≈ our gold treatment — verify exact).
- Footer text colour: capture text-slate-400 on bg-slate-900; currently cream on #0f1b2e.
These land as config under this milestone once the framework primitives in REQ-32 (gradient direction, callout bars, scrim, cool-neutral role) exist for the remaining treatment-level items.