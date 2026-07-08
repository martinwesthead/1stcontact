---
uid: comment-c3ab579b
id: COMMENT-51
type: comment
title: Comment on request REQ-39
created_by: xgd
created_at: '2026-07-06T17:25:18.557864+00:00'
updated_at: '2026-07-06T18:08:28.327408+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-c7f5b23a
  kind: chat_transcript
---

<!-- xgd-turn id="a49f7f7b-75e7-4195-890f-f74607f334e3-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T17:25:15.327887+00:00" -->
#### You
Please take a look at this ticket and tell me if you have enough info to free code it

<!-- xgd-turn id="a49f7f7b-75e7-4195-890f-f74607f334e3-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T17:26:52.524614+00:00" -->
#### Claude
I'll assess this. Let me read the current ticket state and check whether the dependencies it names actually exist in the codebase.

Let me verify the two named dependencies actually exist and understand the seam I'd build on.

The dependencies are real. Let me look at the CLI command structure, especially `shot`, `render`, `serve`, and `values-diff`, to understand the seam I'd delegate to.

Everything the ticket names exists. Let me read the key seam files to understand how a "one-module page" render + serve + Playwright drive would compose.

Now the critical question — does the `BrowserDriver` seam expose what the safety checks need (console errors, `evaluate` for scrollWidth/container geometry)? Let me check the driver interface and how fidelity reads computed styles.

Let me confirm how rendering works and whether a "one-module page" can be synthesized (fixtures → single-module site), plus check the fidelity seam that already reads computed styles.

I've read the ticket and traced every dependency it names into the actual code. Here's my assessment.

## Yes — I have enough to free-code this, with two small design decisions to confirm

### Everything the ticket depends on is real and in place

| Dependency | Where it lives | Status |
|---|---|---|
| `1c shot` render→serve→loopback→drive seam (REQ-13) | `tools/generate/src/cli/{shot,serve}.ts` | ✅ |
| Catalog-driven renderer (REQ-9) | `render/render.ts` — `getModule` + Astro container | ✅ |
| `BrowserDriver` seam with in-page `query(script)` | `cli/capture/{types,playwright-driver}.ts` | ✅ |
| Playwright + `chromiumAvailable()` skip-guard | already a dep of `tools/generate` | ✅ |

Critically, **`cli/fidelity.ts` (`values-diff`) is a near-exact template** for the harness: it does render → `startServe` → `driver.navigate` → `driver.query(EXTRACT_SCRIPT)` → assert, and skips cleanly when Chromium is absent. The harness is the same shape with safety-check assertions instead of a value-diff. That de-risks the whole thing.

### Two decisions the ticket leaves open (neither is a blocker)

1. **Console/page-error capture is a genuine seam gap.** The `BrowserDriver` interface exposes `navigate / screenshot / query / responses / content` — but **not** console errors, `pageerror`, or unhandled rejections. `responses()` covers "no failed requests" (status codes), but "no console/page errors" needs either extending the driver seam to record those events during `navigate()`, or the harness holding its own Playwright page. I lean toward **extending the driver seam** (keeps the CF-swap abstraction intact, and it's a natural addition) — but that touches shared capture code, so I want your nod.

2. **How to synthesize the "one-module page."** `cmdRender` loads a site from disk. A fixture is `{label, props:{variant,dials,content}}` for one module type (`slug`). Cleanest options: (a) scaffold a throwaway one-module site under `storage/sandbox/` and reuse `cmdRender`+`startServe` verbatim, or (b) add an in-memory single-module render entry. I lean **(a)** — maximal reuse of the proven seam, zero new render path. The `_isolation_renders_single_module` UAT then asserts exactly one module band is served over loopback.

Both are the kind of choice I'd normally just make and note in the ticket body — flagging them because they're the only non-mechanical calls in the build.

### Process note
This is framework/test-infra **code**, so it's the full free-coding ceremony: this ticket (REQ-39) is the scope ticket, the 6 named `test_UAT_FC_REQ-39_*` UATs, `[FREE-CODED]` commit, version bump, `move-to-free-coded`. The ticket body is already well-scoped — I wouldn't rewrite it, just keep it current.

Want me to proceed with decisions **1 = extend the driver seam** and **2 = sandbox one-module site**? If you'd rather I go the other way on either, say so and I'll start there.

<!-- xgd-turn id="a7eb94a0-c50e-429e-9be6-a4d453a121a2-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T17:39:03.280599+00:00" -->
#### You
1 sounds fine. 2. I don't have strong feelings - choose the one that makes debugging easiest when it fails I guess?

Its critical that the testing does not polute data on the site. The harness must create its own test_sandbox and clean it up after running. Ok?

<!-- xgd-turn id="a7eb94a0-c50e-429e-9be6-a4d453a121a2-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T18:02:10.177841+00:00" -->
#### Claude
Good — that all makes sense, and yes, agreed on the no-pollution requirement. Here's how I'll settle decision 2 given "easiest to debug on failure":

**Decision 2 → on-disk one-module site in a dedicated, isolated temp sandbox.** This is the most debuggable option because failures leave real artifacts you can open: the generated `site.json`/page JSON, the rendered HTML, and a live loopback URL. Concretely:
- The harness `mkdtemp`s its **own** root under the OS temp dir (never `storage/` and never the real `storage/sandbox/`, so zero chance of touching site data), scaffolds a one-module site there, and drives `cmdRender` + `startServe` against that root via the `cwd` seam.
- **On success:** clean up the temp dir in a `finally`.
- **On failure:** *preserve* the temp dir and log its path + the served HTML path, so you can reopen exactly what the assertion saw. (Cleanup-on-pass, keep-on-fail — best of both.)

That satisfies your isolation requirement structurally: it's physically impossible for it to write into a real site.

Let me record these decisions in the ticket before I write code. First I'll re-read the body and check the update interface so I preserve what's there.

`--append-body` preserves the existing body. Let me record the session decisions.

Decisions recorded on REQ-39. Here's the build plan I'll follow:

**1. Extend the `BrowserDriver` seam** (`cli/capture/types.ts` + `playwright-driver.ts`) — capture console errors / `pageerror` / unhandled rejections during `navigate()`, exposed via a new accessor. This is the only shared-code touch; existing consumers (`fidelity`, `shot`, `capture`) are unaffected.

**2. Harness core** — a new `assertModuleConforms(slug, fixtures, opts)` module that:
- `mkdtemp`s its own store root, scaffolds a one-module site, renders + serves via existing seams, drives Chromium.
- Runs the safety checks (overflow, console/page errors, failed requests, collapsed container, clipped text) at default viewports (desktop + one mobile).
- Honors `except` (declared AC opt-outs), skips clean when Chromium is absent.
- Cleans up temp root on pass, preserves + logs on fail.

**3. Negative-fixture self-tests** — the 6 `test_UAT_FC_REQ-39_*` UATs, including deliberately-broken fixtures the harness *must* flag red (the proof-of-discrimination that's the gating deliverable per DOC-20).

Then: `[FREE-CODED]` commit + version bump, `status=free_coded` with `fields.commits`.

Let me start by reading DOC-20 (the architecture the ticket points to for the `assertModuleConforms` interface) and the `fields.commits` bump interface, so the harness matches the specified contract rather than one I invent.

I now have the exact interface contract from DOC-20 and confirmed the version-bump script is fully implemented. One architectural note before I build: DOC-20 says "e.g. `framework/testing/conformance`", but the render/serve/driver seams all live in `tools/generate/src` (and `tools/generate` depends on `packages/framework`, not the reverse). Putting the harness in `packages/framework` would invert that dependency. So the harness belongs in **`tools/generate/src/conformance/`**, alongside the seams it composes. I'll note that in the ticket.

Let me gather the remaining shapes I need — the module/site schema, `loadSite`, `cmdRender` signature, and store paths — then build.

Let me read the exact `moduleInstance`/`page`/`site` schema shapes and an existing minimal site.json to model the scaffold on.

I've hit the one genuinely new design point worth surfacing before I write shared code. The negative fixtures (`_console_error_fixture_flagged`, `_collapsed_container_flagged`, etc.) are *deliberately-broken modules* that DOC-20 requires to render **through the same catalog renderer** — but a broken module isn't in the shipping registry, and a console-error can't be provoked through content props on a real static module. So the harness needs the renderer to resolve modules through an **injectable resolver** (default = the framework registry; overridden with a tiny test-only registry of broken `.astro` modules for the negative path). That's a small generalization of `renderSite` (an optional param, default unchanged), not a new module — consistent with the project's "generalize before adding" rule.

Let me read the full `render.ts` to design that seam minimally, then I'll record all three decisions and build.

Now I understand the full render path. The broken negative modules need to render through this same path, which confirms the injectable-resolver seam. Two more things I must verify before writing shared code: how `.astro` components are transformed in tests (the broken modules will be `.astro`), and the existing UAT/driver-fake test patterns so my tests match house style.

Tests live in the root `tests/` dir with Astro's transform wired in. Let me look at the existing `shot.test.ts` (fake-driver pattern) and how a render test scaffolds a temp site, so my harness + UATs match house patterns and reuse helpers.

I have all the patterns. Let me grab the last few pieces — `cmdNew` (scaffold shape), `defaultTokens` (for a valid minimal theme), and `text-block`'s meta/content contract (my clean fixture) plus `ModuleDefinition` type — then I'll build.

I have the complete picture. Let me find every `BrowserDriver` implementer, since extending the seam (adding `diagnostics()`) will require updating existing fakes.

I now have every shape I need. Let me set up task tracking and start building. First, let me peek at `text-block`'s root markup (my clean fixture) and the section wrapper, so the isolation assertion targets a real selector.

I have everything needed. Let me set up task tracking and build. I'll start with the shared-seam changes (driver + renderer), then the harness, fixtures, and UATs.

I also need to record the third design decision (injectable resolver seam) on the ticket, then start editing. Let me do both.

Now let me build. First, the driver seam extension:

Now the Playwright driver implementation:

Now update the two existing driver fakes for the new `diagnostics()` method, and export the type. Let me check the capture index exports and read the fake spots.

`capture/index.ts` does `export * from './types'`, so `PageDiagnostics` is already exported. Now update the three fakes:

Now the two fakes in `shot.test.ts` (the `FakeDriver` class and the wrapper object). I've already read this file:

Task 1 done. Now the renderer resolver seam:

Now let me apply the resolver seam. Four edits to `render.ts`: import the type, add `RenderSiteOptions`, thread `resolveModule` into `renderModules`/`renderPage`, and update `renderSite`.

Now `renderPage` and `renderSite`. Let me read the current lines:

Now I need to export `ModuleResolver`/`RenderSiteOptions` from the tools/generate index. Let me check and update it:

Task 2 done. Now the conformance harness core. Let me write the four files:

Now the harness. Let me verify the store exports I'll import:

Now I'll write the harness:

There's a `ModuleResolver` naming collision — it's exported from both `./render` and `./conformance` via `index.ts`. Let me make conformance reuse the renderer's type rather than redefine it.

Task 3 core done. Now the negative broken-module fixtures. These are test-only `.astro` modules plus a resolver. I'll put them under `tests/fixtures/conformance/`:

Now the three broken `.astro` fixture modules (test infrastructure — inline styles/handlers, since the container render drops scoped `<style>`/`<script>`):

Now let me check whether `cmdNew` validates the slug format (my broken-module slugs must pass it), and confirm `createPlaywrightDriver` is exported:

Slugs aren't format-validated. Now the UAT test file:

Now let me typecheck and run the new tests. First, does the project have a typecheck script? Let me build the generate package types and run the REQ-39 test:

Clean typecheck. Now run the REQ-39 UATs:

All 6 pass, and since the 5 browser-driven ones ran (not skipped), a real Chromium genuinely flagged the broken fixtures and passed the clean one. Now let me run the full suite to confirm the seam changes didn't break anything:

All 274 tests pass — the seam changes are backward-compatible. Let me check for leftover temp dirs, then run lint/format and the version bump:

No leftover temp dirs. Let me find how quality/lint is run in this project:

Task 5 done. Now the free-coding ceremony. Let me run quality (lint + tests + coverage), then version bump:

Quality passes (SUCCESS, coverage well above threshold). Now the version bump, then a precise commit (excluding the unrelated gigabytealchemy storage edits):

The gigabytealchemy storage edits remain unstaged (correctly excluded). Now commit:

Commit `0a3e029` created. Now set the ticket to `free_coded` with the commit sha. Let me use the gate command that also verifies the version bump:

The gate verified the version bump and moved REQ-39 to `free_coded`. Let me confirm the final ticket state:

Done. REQ-39 is implemented, tested, and free-coded.

## What I built

The **conformance harness core** (DOC-20) — the discriminator every future module leaf delegates to, proven correct *first* via negative fixtures.

**Shared-seam extensions** (additive, backward-compatible):
- `BrowserDriver.navigate(url, viewport?)` + `diagnostics()` — captures console errors, uncaught page errors, and failed requests during load (the signals a screenshot can't see).
- `renderSite(..., { resolveModule, extraCss })` — an injectable catalog so deliberately-broken fixtures render through the **same** renderer without polluting the shipping registry.

**The harness** (`tools/generate/src/conformance/`):
- `assertModuleConforms(slug, fixtures, opts)` — renders each fixture as a one-module page → serves over loopback → drives Chromium → throws on any non-excepted violation.
- Fast-tier safety checks at desktop + mobile: horizontal overflow, collapsed-to-0 containers, clipped text, console/page errors, failed requests.
- The `except` exemption mechanism, and `serveOneModulePage` for isolation testing.
- **Isolation:** every fixture renders/serves under its own `mkdtemp` root — structurally impossible to touch `storage/` — removed on pass, **preserved + logged on failure** for debugging, exactly as we agreed.

**Proof of discrimination** — 6 `test_UAT_FC_REQ-39_*` UATs, all passing against a **real Chromium**: broken modules (overflow, page-error, collapsed) are flagged red; a clean real `text-block` passes with no false positive; `except` suppresses a declared AC; isolation mounts exactly one module over loopback.

## Verification
- REQ-39 UATs: **6/6 pass** (real browser confirmed — not skipped).
- Full suite: **274/274 pass** — seam changes broke nothing.
- `xgd quality run` (scoped): **SUCCESS**, coverage 96.7% stmts.

## Ceremony complete
- Commit `0a3e029` with `[FREE-CODED]`; version bumped 0.0.42 → 0.0.43.
- REQ-39 → `status=free_coded`, `fields.commits=[0a3e029…]`, `version=0.0.43` (gate verified the bump).
- Three design decisions (driver seam, sandbox lifecycle, resolver seam + harness location) recorded in the ticket body.

Two notes: the unrelated `gigabytealchemy` storage edits that were already dirty at session start I left untouched — they're outside REQ-39's scope. And the follow-on components DOC-20 slices out (responsive axis, cross-browser `full` tier, the template-and-stamp) remain their own REQs, as the ticket's out-of-scope section specifies.

<!-- xgd-turn id="b8bbc254-a4a3-4706-9db8-c2b485cab181-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T18:08:05.314510+00:00" -->
#### You
did you create any tests that use the harness?

<!-- xgd-turn id="b8bbc254-a4a3-4706-9db8-c2b485cab181-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T18:08:28.300245+00:00" -->
#### Claude
Yes — but only the harness's **own self-tests**, not any per-module leaf. Being precise about the distinction:

**Tests that use the harness** — all in `tests/req39-conformance.test.ts` (the 6 UATs):
- 4 call `assertModuleConforms(...)` against the injected broken/clean catalogs (overflow, page-error, collapsed, except).
- 1 (`clean_module_passes`) runs a **real catalog module** — `text-block` — through `assertModuleConforms` with the default framework resolver. So there *is* a test exercising the harness end-to-end against a shipping module.
- 1 (`isolation_renders_single_module`) uses `serveOneModulePage`.

**What I did NOT create:** a *thin module leaf* — i.e., wiring an actual module's story/tests to delegate, the three-line `test('UAT_FC_services-grid_safety', () => assertModuleConforms('services-grid', fixtures, {...}))` shape from DOC-20. No shipping module (hero, services-grid, footer, …) now delegates to the harness.

That omission is deliberate and correct for REQ-39: DOC-20's build-slicing puts "the **module-contract template + stamp** into module stories" as component **#4**, and this ticket's *Out of scope* section explicitly excludes "the module-contract template/stamp — each its own component REQ." REQ-39's job was to build and **prove** the discriminator; stamping it onto real modules is the next ticket.

So: the harness is tested (including against one real module via the clean fixture), but nothing in the shipping module suite delegates to it yet. If you'd like, I can file the follow-on ticket for the template-and-stamp work, or — if you want a stronger proof now — add a leaf that runs, say, `services-grid` and `hero` through `assertModuleConforms` against schema-derived fixtures. Want either?

<!-- xgd-chat-end -->