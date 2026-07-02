---
uid: comment-3fbd90c6
id: COMMENT-13
type: comment
title: Comment on request REQ-15
created_by: xgd
created_at: '2026-07-02T15:48:06.069981+00:00'
updated_at: '2026-07-02T16:03:01.781623+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-284b9873
  kind: chat_transcript
---

<!-- xgd-turn id="c325914a-a65d-459b-8440-c1695ea1b348-user" -->

<!-- xgd-chat role="user" ts="2026-07-02T15:47:55.603Z" -->
#### You
Please free code a solution to this

<!-- xgd-turn id="c325914a-a65d-459b-8440-c1695ea1b348-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-02T16:03:01.750Z" -->
#### Claude
I'll start by reading the ticket to understand the scope and what needs to be solved.

The ticket is well-scoped. This is a substantial feature (5 story points) that builds on REQ-3 (schema), REQ-4/5 (framework), and REQ-14 (background/overlay). Before I write anything, I need to understand the current state of the codebase — what the schema, framework, and validator look like today, and what REQ-14 already delivered.

Let me explore.

I'll explore the key packages in parallel to understand the current schema, framework, and validator.

Now the validator, the REQ-14 background module, and the framework module structure.

Now the registry, framework validate, a sample module, and how rendering happens.

Now the generate tool (where rendering + background wrapping integrate) and the test structure.

Now the styles/CSS aggregation and existing tests, especially REQ-14's tests as the closest analog.

This gives me a thorough understanding. Let me check the existing site fixtures and the structured-edit test to ensure a `.strict()` validator change won't break existing shapes, and see the text-block module (simplest child-rendering reference).

All module instances use exactly `{id, type, version, variant, dials, content}` (+ optional `background`/`seoMeta`) — so a `.strict()` instance schema is safe and gives me the `no_raw_css` rejection. Let me check the text-block module (text child rendering reference), the markdown helper, and confirm the version-bump script works.

I now have a complete picture. Let me lay out my design decision and confirm the ticket body before coding.

**Design decision** (mirrors the proven REQ-14 `background` pattern):
- **`layer` is an optional structured field on a module instance** (exactly like `background`), carrying ordered positioned children. This gives z-compositing over *any* module for free (the `zcompositing` UAT).
- **A registered `layer` module** serves as the standalone art-directed host section (a bare positioning context) — honoring "a `layer` module".
- All positioning is **numeric structured data** (`x/y/z/width/rotate` + per-breakpoint overrides) emitted as framework-computed CSS custom properties — never instance-supplied CSS. Making the module-instance schema `.strict()` gives the `no_raw_css` rejection (verified: every existing instance uses only the known keys).

Let me record this in the ticket body (preserving existing content) and set up my task list.

Now let me implement. Starting with the schema — adding the Position, Treatment, LayerChild, and Layer types, and making the module-instance schema strict.

Now export the new types:

Schema done. Now the framework layer renderer — the core of the feature.

Now the standalone `layer` host module (meta + component):

Now wire it into the registry, module index, and framework index:

Now the framework top-level index:

Now integrate into the render pipeline — apply `wrapWithLayer` over the background-wrapped module HTML, and fold `LAYER_CSS` into `theme.css`:

Now the UAT test file. Let me write the six UATs against real entry points (framework `wrapWithLayer`/`renderLayer`, schema `validateSite`, and the `1c` CLI), mirroring REQ-14's structure.

Now let me run the new tests plus the regression scope (schema, framework modules, background, generate) to confirm nothing broke.

I invoked vitest wrong. Let me use the project's runner.

Both failures trace to the layer not validating. Let me see the actual errors.

The `z.record` with an enum key is exhaustive in Zod v4 (demands all four breakpoints). I'll switch `breakpoints` to an explicit optional-keyed object.

Now let me investigate the zcompositing render failure. Let me check whether `cmdRender` reloads/validates the site independently, and whether the scaffolded site survives the new strict schema.

Let me re-run now that the breakpoints schema is fixed.

5 of 6 pass. The layer isn't reaching the render. Let me reproduce the CLI flow manually and inspect.

tsx can't compile `.astro`. Let me check the scaffold template directly — the strict schema may be rejecting a scaffolded module (making `loaded.ok` false, so render falls back or the layer edit is lost).

Let me instrument the test to print what actually rendered.

`loaded.ok` is true but the hero's `layer` is **undefined after load** — the field is being stripped. `loadSite` imports `validateSite` from the package `@1stcontact/site-schema`, which likely resolves to a stale built `dist/` (without `layer`). Let me check.

Confirmed: the CLI path (`loadSite`, `renderSite`) imports the **built** `@1stcontact/site-schema` and `@1stcontact/framework` packages, which are stale (built July 1, before my changes). The unit tests pass because they import from `src/`. I need to rebuild the packages.

`@1stcontact/framework` exports `src/` directly (picked up live), but `@1stcontact/site-schema` exports `dist/`. Let me rebuild site-schema.

The dist now contains the layer schema. Let me remove the debug block and re-run.

All 6 UATs pass. Now the regression scope — the strict schema change is the riskiest edit, so I'll run the full suite to catch any breakage in existing tests (schema, background, generate, control-app, req11 structured-edit).

One expected failure: an existing REQ-4 test hardcodes `registry.size === 6`; I added the `layer` module → 7. Updating this count is the correct reconciliation of my change. Let me look and fix.

Now let me re-run the full suite and typecheck the changed packages.

All 92 tests pass. Now typecheck the changed packages to ensure the TS is sound.

Both clean. Let me typecheck `tools/generate` (I modified render.ts) and check the lint setup.

No lint step configured; all typechecks clean. Now — the `site-schema` package ships `dist/` and I rebuilt it. Let me check whether `dist/` is tracked in git (it must be committed, or a fresh reconcile checkout would run stale schema and my integration tests would fail).

`dist/` is gitignored, yet integration tests (req14, generate) use the built `@1stcontact/site-schema`. There must be a build step before tests on a fresh checkout. Let me check CI.

CI runs `pnpm -r build` before `pnpm test`, so gitignored `dist/` is fine (rebuilt each time). Let me run the CI-equivalent (`pnpm -r build` then `pnpm test`) to validate exactly as CI would.

Full build passes — including `control-app` and `public-site` typechecking against the strict schema. Now the CI test step:

Everything is green — 92 tests pass, full build clean, typechecks clean. Now the free-coding commit sequence. First, bump the version.

Version bumped 0.0.11 → 0.0.12. Let me confirm what the bump touched and stage only my code + the version file (not the unrelated `.xgd/tickets/search/` ticket index).

Let me stage my files explicitly (excluding the unrelated `.xgd/tickets/search/` ticket-index artifact) and commit with `[FREE-CODED]`.

Commit landed. Now the gating move to `free_coded` and record story points.

REQ-15 is at `free_coded` with the commit recorded. Done.

## Summary

I free-coded **REQ-15 — the `layer` module + z-compositing** following the full free-coding lifecycle. The ticket is at `status=free_coded`, commit `80a7b73`, version `0.0.12`, story points 5.

**What it does:** adds a structured *layer* — an ordered stack of freely-positioned children (images, text runs) composited over a module's markup — enabling art-directed heroes while never leaving the structured-data model (no raw CSS).

**Design** (mirrors the proven REQ-14 `background` pattern for one clean integration point):
- **Schema** (`site-schema`): `Position` (numeric `x/y/z` + `width/height/rotate` + per-breakpoint overrides), image `treatment` (`shape: circle|rounded`, `edge: soft-mask|torn-asset`), and `Layer` (children + `reflow` + optional overlay). `layer` is an optional field on the module instance. The instance/layer/position schemas are `.strict()`, so a raw `style`/`css` prop is a path-pointed validation error.
- **Framework**: `LAYER_CSS` (static positioning/reflow/treatment rules) + `renderLayer`/`wrapWithLayer`, which emit **framework-computed CSS custom properties** (`--fc-x`, `--fc-z`, …) — never instance-supplied CSS. A registered `layer` host module provides standalone art-directed sections.
- **Render pipeline**: `wrapWithLayer` composites over the (optionally background-wrapped) module; `LAYER_CSS` folded into `theme.css` alongside `SECTION_CSS`.

**Evidence:** six UATs (`test_UAT_FC_REQ-15_*`) covering positions, text-over-image, treatments, per-breakpoint reflow, z-compositing over another module, and raw-CSS rejection — all green. Full suite **92/92 pass**, `pnpm -r build` clean (including the `control-app`/`public-site` apps against the new strict schema), all typechecks clean.

**Two reconciliations along the way:** the `breakpoints` map needed an explicit optional-keyed object (Zod v4 treats `z.record(enum, …)` as exhaustive), and I updated the REQ-4 registry-size guard (6 → 7) for the new module.

Note: `"other modules" as layer children` was left out of the child union (image/text cover all six UATs) — flagged in the ticket body as a deferral reconciliation can extend. The ticket is parked at `free_coded`; promotion to `ready_to_reconcile` is your call whenever you've validated it.

<!-- xgd-chat-end -->