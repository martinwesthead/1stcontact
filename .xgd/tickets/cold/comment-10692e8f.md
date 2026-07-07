---
uid: comment-10692e8f
id: COMMENT-35
type: comment
title: Comment on request REQ-33
created_by: xgd
created_at: '2026-07-03T17:33:29.748225+00:00'
updated_at: '2026-07-03T17:33:29.748225+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-f3342338
  kind: note
---

## Batch 2 — fidelity generalizations + full site-def convergence

User asked to (1) make the framework generalizations needed for the remaining visible deltas, then (2) apply all site-def changes for the best copy. All framework additions are dials/roles on EXISTING modules (no new modules).

Framework (code — ceremony):
- Two optional warm palette roles accentLight / accentDeep (schema + defaults + TREATMENT_ROLE_DIAL) so multi-hue warm brand gradients + gold highlight text are expressible as roles, not raw colour. Serves the wordmark gradient AND the hero gold paragraph.
- hero subheadColor dial (a treatment-role) — colours the subhead/body block (gold paragraph).
- contact-form submitTreatment dial (primary|neutral) — neutral = dark button, near-white label.

Site-def (config — exempt):
- footer surface: inverse (was regressed to default → tan).
- header wordmark logoTreatment: gradient + logoGradient(to-right, accentLight->accentDeep); palette accentLight=#f5e6a3, accentDeep=#ff6b35.
- hero subheadColor: accent-light.
- both contact-forms submitTreatment: neutral.
- header spacingTop: xl nudge (title lower).

Consciously deferred to next checkpoint (low visual / not faithfully encodable): subhead +1-step sizes & heading 48->36 (viewport-dependent clamp), wordmark letter-spacing/weight micro-deltas, heading<->subhead gap (not manifest-encoded), footer copyright '© Holder Year' ordering + 2025-vs-2026 build year.