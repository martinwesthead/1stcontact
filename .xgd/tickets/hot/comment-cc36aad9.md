---
uid: comment-cc36aad9
id: COMMENT-28
type: comment
title: Comment on request REQ-20
created_by: xgd
created_at: '2026-07-03T02:03:55.923189+00:00'
updated_at: '2026-07-03T02:03:55.923189+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-97357769
  kind: note
---

Two more config values (milestone scope; capability is [[REQ-32]] #3, flagged by [[REQ-31]]'s diff):
- Hero scrim: capture is a translucent darkening overlay over the image — 'bg-slate-950/30' = slate-950 (#020617) at 30% opacity. Our hero currently renders no scrim (image at full brightness).
- Hero text vertical position: capture anchors content LOW — 'pt-80' (20rem/320px top padding) inside a min-h-screen section, so text sits lower-middle. Ours currently centres content vertically (height:fold -> justify-content:center). Set anchor to bottom/low once the hero content-anchor dial (REQ-32 #3) exists.