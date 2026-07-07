---
uid: request-2e01ed95
id: REQ-41
type: request
title: 'Conformance harness: responsive dimension (viewport axis + mobile checks)'
created_by: xgd
created_at: '2026-07-03T23:18:02.847530+00:00'
updated_at: '2026-07-03T23:18:02.847530+00:00'
completed_at: null
last_field_updated: created_at
status: draft
fields:
  priority: medium
  auto_merge_back: true
  needs_review: false
---

## Goal

Add the **responsive dimension** to the conformance harness ([[REQ-39]] core): assert every module stays safe, legible, and operable across the viewport range, and works on mobile with no per-viewport authoring. Architecture: [[DOC-20]] (AC-M4).

## Why

"Works on mobile with no change" is a core module-contract promise. Responsive is not a separate suite — it is the **viewport axis** over the fast safety checks, plus a few mobile-specific checks.

## Scope / behaviour

`assertModuleConforms(slug, fixtures, { dimension: 'responsive' })`:
- Run the [[REQ-39]] safety checks at each of a viewport set {320, 375, 768, 1024, 1280, 1440} — no horizontal overflow at any width ≥320 is the primary invariant.
- **Mobile-specific:** tap targets ≥44px in the mobile band; a font-size floor; images scale within the viewport; no expected content hidden/clipped.
- Stays `tier: 'fast'` (Chromium) — this axis runs in the tree every cycle; it is cheap (a viewport loop, one engine).

## Dependencies
[[REQ-39]] (harness core + safety checks to loop over viewports).

## UATs (`test_UAT_FC_<TICKET-ID>_*`)
- `_no_overflow_across_viewports` — a well-formed module has no horizontal overflow at any width ≥320.
- `_mobile_overflow_fixture_flagged` — a module that overflows only below 480px is flagged red.
- `_small_tap_target_flagged` — a mobile interactive element under 44px is flagged red.
- `_font_floor_enforced` — text below the legibility floor on mobile is flagged red.
- `_responsive_clean_passes` — a properly responsive module passes at every viewport.

## Out of scope
Cross-browser (separate engines) — this dimension is Chromium-only; cross-engine is [[REQ-42]]-style work.

## Notes
Framework/test-infra code → full free-coding ceremony. Adds responsive negative fixtures to the [[REQ-39]] self-test set.
