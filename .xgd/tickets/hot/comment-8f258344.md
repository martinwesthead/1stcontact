---
uid: comment-8f258344
id: COMMENT-18
type: comment
title: Comment on request REQ-25
created_by: xgd
created_at: '2026-07-02T18:51:43.354285+00:00'
updated_at: '2026-07-02T19:31:35.170976+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-40136a30
  kind: chat_transcript
---

<!-- xgd-turn id="4cacd8b6-7c67-4928-a41f-40ff596b1c21-user" -->

<!-- xgd-chat role="user" ts="2026-07-02T18:51:35.362Z" -->
#### You
Pleasse free code this ticket

<!-- xgd-turn id="4cacd8b6-7c67-4928-a41f-40ff596b1c21-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-02T18:57:32.424Z" -->
#### Claude
I'll start by reading the current ticket to understand the full scope before touching anything.

Let me explore the codebase to understand the framework structure, the header module, and how the REQ-14 overlay and REQ-15 layer mechanisms work.

This is a TypeScript/Astro framework. Let me examine the header, hero, and background modules and the overlay/layer mechanisms.

Now let me look at the REQ-14 overlay mechanism (background), the REQ-15 layer module, and the gigabytealchemy site.

Let me look at the layer module, the page render pipeline, and the gigabytealchemy home page structure.

Now let me understand the render pipeline — how modules get wrapped with backgrounds/layers.

Let me find the render orchestration that stitches modules together (where wrapWithBackground/wrapWithLayer get applied).

Now let me look at the schema/validator, the store loader, and existing tests to understand conventions.

Let me look at existing test conventions (req14, req15, generate) and the naming rules.

I now have a complete picture. Let me look at the framework's public `index.ts` to wire new exports.

I have everything I need. Here's my design decision for the open question in the ticket, then I'll implement.

**Design call (compositing path, expressed as a header variant):**
- Add an `overlay` **variant** to the `header` module → renders transparent (no surface fill, no border).
- Handle the compositing in the **render pipeline**: an `overlay` header is not emitted as its own band; it's floated absolutely over the *immediately-following* module's band, so the two share one continuous background image band. This reuses the hero's `bg-image` (the blessed image path) / REQ-14 background as the shared image; the transparent header lets it show through continuously. No raw CSS — a new static structural CSS block (`OVERLAY_BAND_CSS`) plus the variant.

This keeps it type-general (any `header@overlay` + following band) rather than a gigabyte hack. Let me implement.

The header has evolved (REQ-24 in flight). Let me read the current full header component.

The header/meta and component carry in-flight REQ-24 changes. Let me check the working tree state carefully before I add my changes — I must not sweep unrelated changes into my commit.

There's uncommitted REQ-24 work in the tree (header wordmark font/treatment) overlapping the exact files I need to edit. Let me understand REQ-24's status before proceeding, since this affects how I can commit cleanly.

I've found a blocker I need to surface before writing any code. Let me confirm one more detail about the gigabyte site tracking.

I need to stop before writing code and surface a blocker — this is about your in-flight work, not something I should decide unilaterally.

## Blocker: uncommitted REQ-24 work overlaps REQ-25's files

The working tree has substantial **uncommitted REQ-24** work ("Display-font slot + Cinzel wordmark", status `draft`) sitting in exactly the files REQ-25 needs to touch:

| File | REQ-24 (uncommitted) | REQ-25 needs to edit? |
|---|---|---|
| `packages/framework/src/modules/header/meta.ts` | `logoFont`/`logoTreatment` dials | **Yes** — add `overlay` variant |
| `packages/framework/src/modules/header/index.astro` | wordmark font/treatment CSS (+40 lines) | **Yes** — variant class + overlay CSS |
| `packages/framework/src/modules/dials.ts` | `LOGO_FONT_DIAL`, `LOGO_TREATMENT_DIAL` | No |
| `packages/framework/src/tokens/css.ts` | @font-face emission | No |
| `packages/site-schema/src/{schema,types}.ts` | display-font schema | No |
| `storage/sites/gigabytealchemy/` (all **untracked**) | `logoFont: display`, `logoTreatment: gold` in `home.json` + `cinzel.woff2` | **Yes** — flip header to `overlay` |

Why this matters for free-coding: a `free_coded` commit must be **one-intent-clean** (the reconciler assumes per-ticket cohesion, and `move-to-free-coded` records specific SHAs). But git commits whole-file changes — my REQ-25 hunks in `header/meta.ts` and `header/index.astro` would land in the same commit as REQ-24's uncommitted hunks, and the gigabyte `home.json` I'd edit already carries REQ-24 changes. I can't cleanly separate them at commit time. The whole `storage/sites/gigabytealchemy/` tree is also untracked (likely REQ-20 import work) — committing it is its own decision.

## How do you want to proceed?

1. **(Recommended)** Land or stash REQ-24 first — free-code/commit REQ-24 under its own ticket, or `git stash` it — giving me a clean tree. Then I free-code REQ-25 cleanly on top.
2. **Co-bundle intentionally** — you tell me REQ-24 + REQ-25 (+ the gigabyte site) should ship together; I proceed knowing the commit spans both. This muddies per-ticket attribution, so I'd only do it on your explicit say-so.
3. **I write the REQ-25 code, you own the commit split** — I make the edits; you use `git add -p` to stage only REQ-25 hunks. Workable for the framework files, but `home.json` and the untracked site can't be split cleanly.

My design for the actual work is settled and ready (`overlay` header variant + pipeline compositing over the following band, reusing the hero image / REQ-14 overlay as the shared band — no raw CSS). I just don't want to entangle your REQ-24 work. Which option?

<!-- xgd-turn id="7c6a7ffc-7dc1-4d5c-9ea1-53ff28cc4357-user" -->

<!-- xgd-chat role="user" ts="2026-07-02T19:24:51.767Z" -->
#### You
REQ-24 has finished

<!-- xgd-turn id="7c6a7ffc-7dc1-4d5c-9ea1-53ff28cc4357-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-02T19:31:35.143Z" -->
#### Claude
REQ-24 is done. Let me check whether its changes are now committed so I have a clean base.

REQ-24's framework changes are committed — the tree is clean except the untracked `storage/sites/gigabytealchemy/` (the REQ-20 import, still untracked) and `.xgd/tickets/search/`. Let me verify my base is clean and confirm the committed header meta.

Base is clean. REQ-24's framework changes are committed; the gigabyte site remains untracked REQ-20 import work. 

**Commit-hygiene decision:** My REQ-25 free-coded commit will contain only the framework capability + tests + version bump. I'll edit the gigabyte `home.json` to demonstrate/verify AC3, but I will **not** stage the untracked `storage/sites/gigabytealchemy/` tree into REQ-25 (that's REQ-20's to own) — I'll stage only my framework/test files.

Let me start the free-coding process. First, record the design decision in the ticket (the "note the call" item), preserving the existing body. Let me check the ticket update interface.

I can `--append-body` to preserve the existing content. Let me record the design decision.

Now let me implement. First, the `overlay` header variant — meta + component:

Now the header component — it was changed by REQ-24, let me read its current committed state.

Now the overlay variant CSS — placed after the surface rules so its transparent background wins the specificity tie with `surface-*`:

Now create the compositing module — `overlay.ts` with `OVERLAY_BAND_CSS` and `composeOverlayHeader`:

Now wire the exports through `modules/index.ts` and the framework `index.ts`:

Now the render pipeline. Update `renderModules` to composite an `overlay` header over the following band, and fold `OVERLAY_BAND_CSS` into `theme.css`:

Now fold `OVERLAY_BAND_CSS` into the per-site stylesheet:

Framework changes are done. Now let me check the `cmdNew` scaffold so my end-to-end UAT matches the default site shape.

Let me confirm `getModuleCss` aggregates the header component `<style>` (so I can assert the overlay CSS there).

Now the UAT test file:

Let me clean up that first e2e test — the validate line and unused anchor are sloppy. I'll mirror req14's `loadSite` pattern.

Now let me run the REQ-25 tests plus the adjacent framework/generate suites to catch regressions:

All 5 REQ-25 UATs pass. Now the regression check on the modules/render/generate suites:

Framework + tests are green. Now let me verify AC3 against the real gigabyte site. First flip its header to `overlay` (working-tree only — this stays with REQ-20's untracked site, not staged into REQ-25).

Now render the real gigabyte site and verify header + hero form one continuous image band:

AC3 verified — gigabyte renders header wordmark composited over the AlchemistLab hero image as one continuous `fc-overlay-band`, header transparent, exactly one band. Let me typecheck the framework + generate packages to be safe before committing.

Typecheck clean. Now the version bump (required before the free-coding gate). Let me read the bump script's interface.

Patch bump it is. Let me bump and capture the new version:

Version bumped to 0.0.17. Now stage **only** the REQ-25 framework/test files + version bump — explicitly excluding the untracked gigabyte site and `.xgd/tickets/search/`:

Staged set is exactly my 8 REQ-25 files; the untracked gigabyte site and `.xgd/tickets/search/` are excluded. Now commit:

Committed as `94cd1ec`. Let me verify the version-bump gate check passes for this commit, then update the ticket:

Gate check passes. Now set the ticket to `free_coded` with the commit SHA:

The direct status update is gated — there's a dedicated `move-to-free-coded` command that enforces validation. Let me use it.

Done. REQ-25 is complete and `free_coded`.

## Summary

**Header-over-hero compositing** — a `header` with variant `overlay` now shares one continuous image band with the following section instead of stacking as a separate band above it.

**Design call (recorded in the ticket):** compositing path in the render pipeline, expressed as a `header` `overlay` **variant** — not a REQ-15 layer concern (that composites children *within* one module; this needs two sibling instances to share a band) and not a per-site hack (type-general: any `header@overlay` + following band).

**What changed (commit `94cd1ec`, `[FREE-CODED]`, 8 files):**
- `header/meta.ts` + `index.astro` — `overlay` variant; renders transparent (no fill, no border) with a `variant-*` class, declared after `surface-*` so a `surface` dial still picks the text colour.
- `modules/overlay.ts` (new) — `OVERLAY_BAND_CSS` (static structural) + `composeOverlayHeader`; exported through `modules/index.ts` and framework `index.ts`.
- `render.ts` — `renderModules` holds an overlay header and floats it over the next band; `OVERLAY_BAND_CSS` folded into per-site `theme.css`.
- `tests/req25-header-overlay.test.ts` — 5 UATs (`test_UAT_FC_REQ-25_*`), all passing; 62 adjacent tests still green; both packages typecheck clean.
- version bump 0.0.16 → 0.0.17.

**AC verification:**
- AC1/AC2 — covered by UATs (structured variant, no raw/instance CSS; reuses the band's REQ-14 image/overlay).
- AC3 — rendered the real **gigabytealchemy** site: wordmark "GIGABYTE ALCHEMY" composites over the AlchemistLab hero image inside exactly one `fc-overlay-band`, header transparent — one continuous dark image band matching the reference.

**One thing left for you / REQ-20:** the AC3 verification required flipping the gigabyte header to `variant: overlay` in `storage/sites/gigabytealchemy/draft/pages/home.json`. That site tree is **untracked REQ-20 import work**, so — per the commit-hygiene call I flagged earlier — I left the edit in the working tree and did **not** stage it into REQ-25. It'll be committed when REQ-20 lands the site. The framework capability it depends on is now in `main`-bound history via REQ-25.

<!-- xgd-chat-end -->