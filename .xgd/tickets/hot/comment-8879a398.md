---
uid: comment-8879a398
id: COMMENT-22
type: comment
title: Comment on request REQ-20
created_by: xgd
created_at: '2026-07-03T00:28:18.953075+00:00'
updated_at: '2026-07-03T00:28:18.953075+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-97357769
  kind: note
---

## Progress update (2026-07-02, eyes pass #2)

Operator eyes-review flagged four fidelity gaps beyond the band-stack body. Implementing under this milestone (structured dials/primitives — no raw CSS reaches site defs):

1. Hero fills to the fold — new hero 'height' dial (auto|fold); fold => min-height 100vh, content vertically centered (explicit original design intent).
2. Hero subhead renders markdown — was schema-typed markdown but rendered raw, collapsing paragraph breaks; now via renderMarkdown.
3. Header wordmark size — new header 'logoSize' dial (sm|md|lg|xl); reference wordmark is display-scale.
4. Footer justified layout — new footer 'layout' dial (center|spread); spread => copyright left, links right.
5. Side-by-side forms — new 'width' dial (full|half) + render-pipeline row grouping: consecutive half-width bands share one fc-row (get-in-touch subscribe|contact two-column).

Also: header align corrected center->left (config only) — the actual capture is left-aligned.