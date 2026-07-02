---
uid: request-f5aad951
id: REQ-17
type: request
title: 'Bespoke-module lifecycle: draft-module rendering + publish-gate-on-hardening'
created_by: xgd
created_at: '2026-07-02T00:20:05.021809+00:00'
updated_at: '2026-07-02T00:20:05.568226+00:00'
completed_at: null
last_field_updated: body
status: draft
fields:
  story_points: 5
  priority: medium
  auto_merge_back: true
  needs_review: false
---

## Scope

The [[DOC-14]] plumbing that makes Tier-B (bespoke modules) real: `1c render` resolves **site-local / draft** modules (not just the shared library), and **publishing gates on module hardening** — a site cannot publish while it depends on an unhardened bespoke module.

## Dependencies
REQ-9 (`1c` CLI / render), [[DOC-14]], [[DOC-12]] (publish gate).

## Deliverables
- **Module resolution:** `1c render` loads a site-local module dir in addition to `packages/framework`; a draft module renders in the **draft** channel.
- **Hardened marker:** a module carries a `hardened` state (set by the XGD harden pass).
- **Publish gate:** `1c publish` refuses when any depended-on module is unhardened, with a clear, structured error.
- **Capability spec:** a gap emits a structured capability request (target screenshot + description) — the [[DOC-13]] gap-log entry.

## UATs (`test_UAT_FC_REQ-17_*`)
- `test_UAT_FC_REQ-17_render_resolves_site_local_module` — a draft renders using a site-local module.
- `test_UAT_FC_REQ-17_publish_blocked_unhardened` — publish is blocked (clear error) with an unhardened dependency.
- `test_UAT_FC_REQ-17_publish_allowed_after_harden` — publish succeeds once the module is marked hardened.
- `test_UAT_FC_REQ-17_capability_spec_emitted` — a gap produces a capability spec.

## Out of scope
The XGD harden pass itself (its own workflow); the in-session drafting UX (builder).
