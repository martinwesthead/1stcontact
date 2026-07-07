---
uid: comment-6d8ba4e9
id: COMMENT-23
type: comment
title: Comment on request REQ-20
created_by: xgd
created_at: '2026-07-03T01:09:41.458857+00:00'
updated_at: '2026-07-03T01:09:41.458857+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-97357769
  kind: note
---

## Progress update (2026-07-02, eyes pass #3 — card colors + Exploring panel)

Operator reframed as a fidelity-reproduction exercise (not design preference) and asked to close the 'What We're Exploring' gap and match card colors, using the captured reference values (raw.html Tailwind classes).

Captured reference: Sanctum = amber/gold border + soft-emerald 'In development' badge + emerald ticks; XGD = blue border + soft-blue 'Coming soon' badge + blue ticks; Exploring = slate panel (from-slate-100/200) + slate border, a sibling card inside the Building grid.

Framework limits found + closed:
1. Palette had no second functional accent. Added optional 'secondary' palette role (10th role; defaults-filled so 9-role themes still validate). Site sets it to #3b82f6 (blue).
2. services-grid: CARD_ACCENT + BADGE_VARIANT extended with 'secondary'; badges restyled soft (light tint + role-coloured label, via color-mix) to match captured pills; checklist ticks now follow the badge variant colour (status-<variant> hook) instead of hardwired green.
3. Added card 'surface' field (default|muted) — a filled neutral panel (the Exploring card).
4. 'What We're Exploring' moved from a standalone text-block into the Building services-grid as a third card (accent muted, surface muted) — closes the structural gap.

Residual (noted, minor): the Exploring panel derives its fill from the neutral 'muted' role, so it reads as a light neutral-grey panel rather than the reference's slightly-cool Tailwind slate; border is muted-grey vs slate-400. Palette-faithful and reads as a distinct panel; a dedicated cool-neutral role would close the last hue delta if wanted.