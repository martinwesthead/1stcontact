---
uid: comment-fe6aba7e
id: COMMENT-11
type: comment
title: Comment on request REQ-13
created_by: xgd
created_at: '2026-07-02T00:08:03.493524+00:00'
updated_at: '2026-07-02T01:03:50.900530+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-8a601134
  kind: chat_transcript
---

<!-- xgd-turn id="c12a4542-6d1e-4623-a2d1-b3413549dc86-user" -->

<!-- xgd-chat role="user" ts="2026-07-02T00:07:55.606Z" -->
#### You
Are you ready to code this?

<!-- xgd-turn id="c12a4542-6d1e-4623-a2d1-b3413549dc86-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-02T00:09:01.868Z" -->
#### Claude
I'll do the legwork first — read the current ticket and check the codebase state for the dependencies REQ-13 relies on.

Now let me verify the two dependencies REQ-13 needs actually exist: the `BrowserDriver` seam and `1c render`/`1c serve`.

Both dependencies exist. Let me read the seam, the serve helper, and the driver to understand the exact interfaces I'd build on.

Let me check the args parser and the capture command shape, plus how viewport presets might already be expressed.

Yes — I'm ready. I've read the ticket and confirmed both dependencies are in place, so this is a thin command over existing pieces exactly as the ticket says.

**What exists (verified):**
- `BrowserDriver` seam (`tools/generate/src/cli/capture/types.ts`) with `navigate()` / `screenshot(viewport?)` / `close()`, plus `createPlaywrightDriver` and `chromiumAvailable()` for clean test-skip.
- `cmdRender` (`1c render`) and `startServe` (`1c serve`) returning `{ server, url, rootDir }` against `dist/<root>/<slug>/<channel>/`.
- Args parser already treats `--url` / `--viewport` / `--out` / `--source` as valued flags — no parser change needed.

**Implementation plan (single free-coded intent):**
1. New `tools/generate/src/cli/shot.ts` → `cmdShot(...)`:
   - **slug mode:** `cmdRender(slug, {source})` → `startServe` → `driver.navigate(localhost url)` → `screenshot` → write PNG → **close the server** (the missing-asset fix: screenshot the *served* page so `/assets/` resolve over localhost).
   - **`--url` mode:** navigate + screenshot the URL directly.
   - Viewport presets from DOC-8 §3.4: `mobile 375`, `tablet 768`, `desktop 1280×800` (default desktop). Deterministic width is the asserted dimension; `fullPage` height is content-driven.
   - Inject `driverFactory` (defaults to Playwright) for testability, mirroring `cmdCapturePage`.
2. Wire into `cli/index.ts`: `case 'shot'`, USAGE lines, re-export.
3. UATs in `tests/shot.test.ts` (real Chromium behind `chromiumAvailable()`, ephemeral loopback fixture — same pattern as `capture.test.ts`):
   - `test_UAT_FC_REQ-13_shot_draft_assets_load`
   - `test_UAT_FC_REQ-13_shot_url`
   - `test_UAT_FC_REQ-13_deterministic_viewport`

**Two small calls I'll make unless you object:**
- **Default `--out`:** slug mode → `dist/<root>/<slug>/shot-<source>-<viewport>.png`; `--url` mode → `./shot.png`. `--out` overrides both.
- **Per-section crops:** the ticket marks these *optional* ("when a capture/segmentation is available"). I'll **defer** them from this pass to keep the intent tight — the full-page shot is the primitive the AI-eyes loop needs now. Say the word if you want crops included.

Ticket's already populated and at `draft`; I'll leave it there, code + UATs, then commit `[FREE-CODED]` and move it to `free_coded`.

Give me the go and I'll start — or tell me to fold crops in first.

<!-- xgd-turn id="9fe2cd81-a236-4018-8b02-988bde6e0a14-user" -->

<!-- xgd-chat role="user" ts="2026-07-02T00:54:00.461Z" -->
#### You
go ahead

<!-- xgd-turn id="9fe2cd81-a236-4018-8b02-988bde6e0a14-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-02T01:03:50.875Z" -->
#### Claude
I have the full picture. Writing the `shot` command now.

Now wire it into the CLI dispatcher.

Now add the USAGE lines and the `case 'shot'` dispatcher:

Now add the `parseViewport` helper near the other helpers:

Let me typecheck before writing tests:

Typecheck clean. Now let me study the test setup patterns for creating a site + rendering, and existing fixtures.

I need to understand how rendered HTML references `/assets/` so the assets-load UAT is faithful. Let me look at the render pipeline and asset handling.

Let me render a real site in a temp dir to see the actual emitted asset URL — most reliable.

Let me see how `capture.test.ts` guards real-Chromium tests and builds fixtures, to mirror the pattern exactly.

Let me inspect a fresh site's default structure and how the header logo becomes an `<img>` — that's the most reliable way to guarantee a rendered `/assets/` image.

I have everything I need. Writing the UATs:

The unused `createServer`/`Server` imports are dead weight — let me remove them rather than paper over with a re-export.

Now typecheck and run the tests:

All 3 UATs pass — including the real-Chromium fidelity test (Chromium was available, so it ran, not skipped). Let me run the full suite to confirm no regressions, then check the CLI dispatch path for coverage.

Full suite green. Now the quality gate (lint + coverage), which is the real gate:

Quality gate passed (0 errors/warnings, coverage 94% stmts). Now the free-coding bookkeeping. Let me check the version-bump requirement and current git/ticket state.

Version-bump script is implemented. Free-coding sequence: bump version, commit with `[FREE-CODED]`, then update ticket. Let me bump and stage:

Now the ticket bookkeeping. Let me get the full SHA and check the ticket CLI verbs for setting commits and moving to free_coded.

Let me append a brief implementation note (preserving the existing body), then run the gate:

Ticket moved to `free_coded`. Let me verify the final ticket state:

Done. `1c shot` is implemented, tested, and the free-coding cycle is complete.

## What shipped (REQ-13)

**`tools/generate/src/cli/shot.ts`** — the screenshot primitive, wired into the dispatcher:
- `1c shot <slug> [--source draft|published] [--viewport mobile|tablet|desktop] [--out <file>]` — renders the source, serves it over loopback, and screenshots the **served** localhost URL so `/assets/` resolve (the blank-screenshot fix). Server torn down in a `finally`.
- `1c shot --url <url> [...]` — screenshots any URL via the same seam.
- Deterministic viewport presets (mobile 375×667, tablet 768×1024, desktop 1280×800 default); full-page PNG. Reuses `cmdRender` + `startServe` + the `BrowserDriver` seam with an injectable `driverFactory`.

**`tests/shot.test.ts`** — 3 UATs, all passing:
- `test_UAT_FC_REQ-13_shot_draft_assets_load` — **real Chromium** against our own served draft; asserts every referenced `/assets/` image actually painted (`naturalWidth > 0`) and the PNG is valid + desktop-width. This is the faithful proof the missing-asset bug is fixed.
- `test_UAT_FC_REQ-13_shot_url` — fake-driver seam test: `--url` drives the driver at the exact URL and writes a PNG.
- `test_UAT_FC_REQ-13_deterministic_viewport` — each preset maps to fixed, stable dimensions across runs.

**Verification & bookkeeping:**
- Full suite: 82/82 green. Scoped quality gate: passed, 0 errors/warnings, 94% statements.
- Version bumped 0.0.9 → 0.0.10; committed `19fdbf3` with `[FREE-CODED]`.
- REQ-13 → `free_coded`, `commits=[19fdbf3…]`, body updated with implementation notes.

**Deferred (was marked optional in scope):** per-section crops — pending a capture/segmentation for a served slug. Say the word if you want them folded in next.

<!-- xgd-chat-end -->