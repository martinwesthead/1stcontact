---
uid: comment-5f36eec9
id: COMMENT-32
type: comment
title: Comment on request REQ-33
created_by: xgd
created_at: '2026-07-03T16:35:59.189487+00:00'
updated_at: '2026-07-03T16:35:59.189487+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-f3342338
  kind: note
---

Progress (fresh diff-driven import, DOC-19):

- Re-captured gigabytealchemy.ai -> storage/references/gigabytealchemy.ai/index (REQ-31-complete: per-element gradient/borderLeft/fontSize + section anchorRatio present). AC1 met.
- Fresh site built at sites/gigabytealchemy, transcribed from raw.html/capture.json.
- values-diff loop: 86 deltas -> 77 after the config/site-def pass.

Config/site-def wins (REQ-33 scope, done):
- hero headingTreatment gold->accent (solid #fbba72, removed wrong gradient) + size md->sm (36px) + scrim medium.
- header logoSize lg->xl (wordmark 72px).
- footer surface inverse->default (links #000000).
- removed **bold** from two card intro lines (weight 700->400).

Residual 77 deltas are ALL framework-level. Grouped by root cause; each is a REQ-32-style generalization needing free-coding:

1. markdown smartypants curls straight apostrophes -> 9 missing paragraphs. Fix: createMarkdownProcessor({smartypants:false}).
2. services-grid checklist tick is a ::before pseudo-element at pad-left 24 -> 6 missing tick runs + 6 paddingLeft(24->0). Render tick as inline text, no indent.
3. services-grid card-title size 20->24, weight 600->700.
4. services-grid subhead colour muted->#000000, size 16->18.
5. services-grid badge label colour ->#000000, size 12->14, weight 600->500.
6. services-grid card-body intro-line size ->18; "More to come" caption ->14.
7. hero subhead/body over image: gold #f5e6a3 (currently cream), sizes 24/18 (currently 20/16), subhead weight 300. Needs a hero subhead treatment.
8. contact-form buttons: colour #fff, size 16, weight 500, font-family ui-sans-serif (currently Arial), Send-message pad-left 32; plus missing "Protected by Cloudflare Turnstile." caption.
9. contact-form subhead colour ->#000000, size ->18.
10. footer copyright omits year -> "(c) Gigabyte Alchemy 2025" missing (renders "(c) 2026 ...").
11. header wordmark exact gradient 90deg [#f5e6a3,#f5e6a3,#ff8c42,#ff6b35] - palette-role-constrained (needs gold/orange roles or raw stops), plus letter-spacing -1.8, weight 600.
12. line-height (16 runs) + hero heading letter-spacing -0.9 + section 0/3 contentAnchor - fine controls not exposed.

Items 1, 2, 7, 11 have cross-site / design impact. Holding for operator direction before opening framework free-coding tickets.