---
uid: comment-d2af3cca
id: COMMENT-55
type: comment
title: Comment on request REQ-47
created_by: xgd
created_at: '2026-07-06T22:16:59.575484+00:00'
updated_at: '2026-07-06T23:03:10.576737+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-65fa5199
  kind: chat_transcript
---

<!-- xgd-turn id="f499426a-991b-4c19-a33e-cb4cb6cebcf1-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T22:16:56.408290+00:00" -->
#### You
Pplease review this ticket - are you ready to implement it. Note that REQ-46 and REQ-20 are making changes in the workspace so any changes need to be contained until they are finished

<!-- xgd-turn id="f499426a-991b-4c19-a33e-cb4cb6cebcf1-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T22:20:09.275482+00:00" -->
#### Claude
I'll review REQ-47 for implementation readiness. Since this is an assessment (not yet coding), let me investigate the current state of the fidelity tooling the ticket builds on. Let me do the legwork in parallel.

I have the ticket and the concurrency picture. Let me note the workspace situation while the code-mapping agent runs:

- **REQ-20** (gigabytealchemy import) has modified: the module `.astro` files + `storage/sites/gigabytealchemy/*`
- **REQ-46** (renderer hardening) added `packages/framework/src/modules/safety.ts` and is touching module rendering
- **REQ-47** (this ticket) is the **fidelity-diff tooling** — the `1c` capture / values-diff / perceptual-diff pipeline

These are different areas, so containment looks natural. I'll confirm exact file boundaries once I have the code map. Let me wait for the exploration to finish.

`tools/generate` is the `1c` CLI. The capture code lives in `tools/generate/src/cli/capture`. One flag: `tools/generate/src/conformance/harness.ts` is **already modified in the working tree** (uncommitted, by REQ-20/REQ-46) — and the ticket references "the harness" that captures section geometry. That's a potential overlap I'll need to pin down. Let me wait for the full code map.

The picture is clarifying: `conformance/harness.ts` is the **module-conformance** harness (REQ-46-adjacent security work), *not* the capture harness — separate concern. REQ-47's actual target files are all under `tools/generate/src/cli/capture/` (`extract.ts`, `sections.ts`, `types.ts`, `values-diff.ts`) plus `cli/perceptual.ts` and `cli/fidelity.ts`. None of those show as modified in the working tree, so no live overlap with REQ-20/REQ-46. Let me wait for the code-mapping agent to confirm the internals before I give the verdict.

I have a strong architectural picture now. The tooling is TypeScript under `tools/generate/src/cli/` (the `1c` CLI): `fidelity.ts` = `values-diff`, `perceptual.ts` = `diff`, capture/extract under `capture/`. Let me wait for the mapping agent to finish detailing the manifest schema and pairing logic before I write the verdict.

I have nearly the full picture. Let me wait for the code-mapping agent to return the internals of the values-diff pairing/scoring and the extract manifest schema — those determine how much of Part A/B is greenfield vs. extension.

I have enough to see the shape. The extract script already captures section `box` geometry and a `role` per content item, but per the ticket, content items (headings/body/inputs) carry `role` without `box` — which is exactly what Part A extends. Let me wait for the mapping agent's synthesis to confirm the pairing/scoring internals, then I'll give you the verdict.

Confirmed the core contract directly: `RawRun` (text elements) carries styling fields but **no `box` geometry, no shape (border-radius/box-shadow), no a11y `nameSource`** — while `RawBand` (sections) has `box`. That's exactly the gap Part A describes, so the ticket's read of the current state is accurate. Let me wait for the mapping agent to return the values-diff pairing/scoring internals so I can gauge Part B precisely.

I now have everything I need — I verified the ticket's claims about current state directly against the code. Here's my readiness assessment.

## REQ-47 — implementation readiness: **Yes, with one scoping decision and a containment plan**

### The ticket is accurate and well-specified
I verified its description of the current state against the code rather than taking it on faith:

- **Capture** (`tools/generate/src/cli/capture/extract.ts`): confirmed `RawBand` (section) carries `box: {x,y,width,height}`, but `RawRun` (headings/body/inputs/buttons) carries styling only — **no geometry, no shape, no a11y `nameSource`**. That's exactly the gap Part A fills.
- **values-diff** (`capture/values-diff.ts`): confirmed it joins text runs by normalized-text and sections by ordinal, and ranks by a flat **per-property numeric weight** (e.g. `borderLeft: 80`). Part B's kind→tier table is a refactor/extension of this existing `severity` mechanism, not greenfield. Unmatched-element handling already exists (`matched`/`unmatched` counts), so the "over-emit as `presence`" stance has a foothold.
- **perceptual diff** (`cli/perceptual.ts`): confirmed block-averaged heatmap → connected-components → scored regions. Part C extends this; it's explicitly optional in the ticket.

Tooling is TypeScript; tests are `tests/reqNN-*.test.ts` with `test_UAT_FC_REQ-N_*` naming (predecessors: `req31-values-diff.test.ts`, `req38-perceptual-diff.test.ts`). So REQ-47 tests → `tests/req47-fidelity-structural.test.ts`.

### Containment vs REQ-20 / REQ-46 — clean, with one nuance
- **REQ-47 touches**: `capture/{extract,sections,types,values-diff}.ts`, `cli/{perceptual,fidelity}.ts`, and a new test file. **None are modified in the working tree.**
- **REQ-20 touches**: module `.astro` files + `storage/sites/gigabytealchemy/*`. **REQ-46 touches**: `modules/safety.ts` + `conformance/harness.ts`.
- The one thing to not trip on: the modified `tools/generate/src/conformance/harness.ts` is the **module-conformance** harness (REQ-46 security work), *not* the capture harness the ticket refers to. Different file, outside REQ-47's path. **No file-level overlap.**

The real coupling is at the **acceptance layer, not the code layer**: REQ-47's headline acceptance ("the three gigabytealchemy misses appear as CRITICAL deltas") uses REQ-20's reproduction as its fixture — and REQ-20 is still reshaping that site's modules/JSON. So the reproduction's rendered output is in flux.

**My plan to stay contained**: build and unit-test Parts A+B against **synthetic fixtures** (the `tests/fixtures/capture/*.html` pattern already in use), which are independent of REQ-20. Defer only the end-to-end "three REQ-20 misses" validation until REQ-20's repro stabilises. That way REQ-47's tool code lands and is fully tested without racing REQ-20.

### One scoping decision for you
The ticket is 8 points and splits into A (richer capture), B (severity-tier ranking — nearly free once A lands), and C (structure-aware image diffs — the ticket itself marks "lower priority, partly redundant, build last and selectively"). I recommend **A + B this cycle, C deferred** (a later slice or its own follow-on within this ticket). A+B alone satisfies the three CRITICAL acceptance items — all three are derivable from A's projection without any image work.

### Two design choices I'd lock before coding
1. **a11y `nameSource` extraction**: I'd derive it **in-page** within the existing `EXTRACT_SCRIPT` (check `placeholder` attr / associated `<label>` / `aria-label`) rather than pulling Playwright's accessibility snapshot — keeps it to the single existing extraction pass. Agree?
2. **Text-free element pairing** (inputs/images/dividers) by `role + order-within-section` — the ticket accepts this as the main false-positive source under its over-emit stance. Fine as specified.

**So: I'm ready.** Before I write any code I'll (1) get your go on **A+B-now / C-deferred**, (2) confirm the in-page `nameSource` approach, then (3) start the free-coding flow properly — the ticket's already at `draft` with a strong body (I won't rewrite it), I'll populate the test-approach section, code against synthetic fixtures, `[FREE-CODED]` commit + version bump + `move-to-free-coded`. Want me to proceed on that basis?

The code-mapping agent came back and corroborates my direct findings — plus one factual correction worth surfacing before we lock scope.

**Correction to the ticket's premise:** the ticket says the perceptual-diff "ranks regions by `intensity × area`." It actually ranks by **sum-of-block-averages** in each region (`score = Σ block-averages`, then `sort descending`). That model already partly balances "large-faint ≈ small-intense" — it is *not* literally area-weighted. The ticket's practical point still holds (pixel-heat ≠ eye-importance; a 100%-wrong small form still can't outrank a large mildly-off band, and structural/containment/arrangement facts aren't represented at all). But the stated mechanism is slightly off. That doesn't change the plan — Part B's severity-tier ranking lives in **values-diff** (the A→A structural diff), which is where the three CRITICAL acceptance items are derived; the perceptual scorer is Part C (deferred). I'd suggest a one-line ticket-body tweak to say "ranks by summed block-intensity, blind to structure" rather than "intensity × area" — your call, since the body is yours.

Everything else confirms the assessment:
- **Extension surface is clean**: per-element `box`/shape/a11y are additive optional fields on `ContentRun` — backward-compatible with existing capture bundles.
- **Zero a11y-tree coupling today** (role is inferred from tag name), so in-page `nameSource` derivation slots in without new browser plumbing — reinforces my design recommendation #1.
- **Tests can be synthetic** (Vitest, fixture HTML/PNG pairs, mostly no live browser) — which is exactly what lets REQ-47 stay contained from REQ-20's in-flight reproduction.

So the map is complete and the verdict stands: **ready to implement.** I'm holding on your two answers before writing code:

1. **Scope**: A + B this cycle, **C deferred**? (A+B satisfies all three CRITICAL acceptance items.)
2. **`nameSource`**: derive in-page inside the existing extract script (placeholder attr / associated `<label>` / `aria-label`)?

Give me the go on those and I'll start the free-coding flow (populate the test-approach section of the existing body, code against synthetic fixtures, `[FREE-CODED]` + version bump + `move-to-free-coded`), keeping strictly to the `capture/` + `perceptual`/`fidelity` files so nothing collides with REQ-20/REQ-46.

<!-- xgd-turn id="a1d631c9-17f6-459e-a108-fea88f4b39d3-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T22:36:24.835268+00:00" -->
#### You
All good - update the ticket and then implement A and B now

<!-- xgd-turn id="a1d631c9-17f6-459e-a108-fea88f4b39d3-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T23:03:10.539519+00:00" -->
#### Claude
Starting now. Let me set up tracking, then update the ticket (preserving its body), then read the code I'll extend.

Let me read the rest of the pipeline: `sections.ts`, `types.ts`, and `values-diff.ts`.

Now the values-diff — the biggest file and the core of Part B.

I have the full pipeline mapped. Before I design the delta-shape changes, I must see what the existing REQ-31/35/38 tests assert so I don't break them (that would be drift). Let me read them and set up task tracking.

Now I understand the constraints. The existing REQ-31/35 tests pin these orderings I must preserve: `text > color`, `overlay > contentAnchor`, `color > letterSpacingPx`, and property names (`missing`, `text`, `color`, etc.). Key insight: **`contentAnchor` must stay ranked below `overlay`** — so the new CRITICAL `position` kind must be a *new per-element field*, not a re-tiering of `contentAnchor`. Let me read `fidelity.ts` (the report printer / command) to see how deltas are consumed and printed.

`fidelity.ts` only reads `property/text/role/expected/actual` and relies on pre-sorted `deltas` — so I can change the internal sort freely and enhance the printer to lead with the tier. 

My design is locked. Let me record the plan on the ticket first (per instruction), then implement. Checking the `append` interface:

Ticket updated. Now let me verify no existing test does strict object-equality on the capture shapes (new fields would break a `toEqual`/snapshot), then I'll implement.

Clean — no strict-equality on capture shapes. Implementing now. Starting with `extract.ts` (types + EXTRACT_SCRIPT).

Now add `RawField` after `RawRun`, and `fields` to `RawBand`:

Now the EXTRACT_SCRIPT. First, add the in-page helpers (shape, a11y role/name) after `roleOf`:

Extend `runsUnder` to capture per-element geometry, and add a `fieldsUnder` collector:

Now wire `fields` into the band loop and assign arrangement over the merged element list:

Now `types.ts` — add the REQ-47 fields to `ContentRun`, a `Field` type, and `Section.fields`:

Now `sections.ts` — flow geometry through `toContentRun` and build the `fields` array:

Now the core: `values-diff.ts`. Before I rewrite the diff, I must see exactly how the existing REQ-31/35 tests build manifests (synthetic `el()`/`mani()` vs real captures) and what they assert on counts — new geometry deltas could break them. Let me read the test helpers and the full-capture diff tests.

Key constraints confirmed: synthetic `el()`/`run()` set no `box`, and `writeRefBundle` omits `fields` — so `flattenCapture` must guard `section.fields ?? []` and geometry comparisons must skip when `box` is absent. Let me read the rest (section-level + capture tests) to be sure no real-render diff asserts exact counts.

All existing tests use direct manifests or field-free fixtures — my guards (`fields ?? []`, skip-when-no-`box`) keep them green, and my tiering preserves `overlay>contentAnchor` / `color>letterSpacing`. Now I'll rewrite `values-diff.ts`. First, imports + `ValueElement`:

Now the delta model — extend `DeltaProperty`, add `DeltaKind`/`SeverityTier`, and enrich `ValueDelta`:

Now the projection functions — copy geometry into `contentRunToElement`/`rawRunToElement` and add `fieldToElement`:

Now replace the flat `SEVERITY` map with the kind→tier tables and a composite-severity helper:

Add the new tolerances to `DiffOptions`:

Now add label helpers next to the existing ones:

Now let me check whether any test diffs a *real* reproduction (where new geometry deltas could fire) and asserts exact delta sets — REQ-33/REQ-20/capture tests:

No existing test diffs a real render — only synthetic manifests (REQ-31/35) and inspection-only capture tests. My geometry deltas are safe. Now the `diffManifests` rework. First, the new tolerance vars:

Now the queue building, `record`/`push` (with kind+magnitude), field pairing, and geometry comparison:

Add the geometry comparison to the end of the text-run loop:

Let me thread magnitude into the color and font-size deltas for within-kind tiebreak:

Now update `fidelity.ts` `formatReport` to lead with the severity tier (the ticket's "lead with the severity-ranked list" note), and refine the sort comment:

Now let me verify the barrel exports so my tests can import the new types (`Field`, `DeltaKind`, `SeverityTier`, `fieldToElement`, etc.):

The barrel re-exports `./types` (so `Field`, `Arrangement`, `NameSource` are covered) but I should export the new `values-diff` symbols and `RawField`. Let me add them:

Let me verify the toolchain is present (so I don't trigger an install) and find the typecheck/test commands:

`tsc` and `vitest` are present locally. Let me typecheck the tooling directly with the local binary (no install):

Typecheck clean. Now run the existing REQ-31/35 tests to confirm no regression:

All 35 existing tests green, typecheck clean. Now the REQ-47 UATs. First a real-render fixture:

Now the test file. Let me check the `serveDir` helper pattern REQ-31 uses for the real-render suite:

Now I'll write the REQ-47 test file:

Let me remove the leftover placeholder cruft in the real-Chromium suite:

Now run the new REQ-47 suite plus the existing ones together:

47 pass. Critical question: did the **real-Chromium** tests actually execute (validating the in-page extraction of box/nameSource/arrangement), or silently skip? Let me confirm Chromium is available and my extract code runs in a real browser:

The probe path was wrong (ran from `/tmp`). Simpler: run REQ-31's real-Chromium capture suite verbosely — those share `EXTRACT_SCRIPT` with my changes, so if they execute (not skip), my in-page code is exercised on a real browser:

The real-Chromium tests ran in 0ms — they **no-op'd** (Chromium unavailable in this sandbox), so my in-page `EXTRACT_SCRIPT` additions are not yet validated at runtime. The highest risk there is template-string escaping. Let me at least syntax-validate the compiled script, then decide on deeper validation:

vitest only includes `tests/**`. Let me validate via a temp test there, and while at it, attempt a **jsdom** end-to-end of `EXTRACT_SCRIPT` (jsdom is a devDependency) so the novel in-page logic gets real execution without a browser:

jsdom-as-vitest-environment breaks esbuild. Let me instead construct JSDOM **manually** in the default node environment and `eval` the script inside its window — that avoids the global-environment conflict:

The JSDOM end-to-end validates the novel in-page logic (a11yRole, `nameSource='placeholder'`, geometry-derived `arrangement='row'`, per-element box) deterministically — no browser needed. Let me fold these into the main test file as proper UATs and drop the probe file:

14/14 pass, including real execution of the in-page script under JSDOM. Now run the broader capture-adjacent suites to confirm no collateral breakage, and re-typecheck:

One capture.test.ts failure — a "Capture failed after N attempts" (browser capture). Let me check whether it's browser-gated and whether it's pre-existing (Chromium-unavailable) vs. caused by my change:

Found it — the test uses a `FakeDriver` returning a synthetic `signals` whose `RawBand` has no `fields`, and my `sections.ts` does `b.fields.map(...)` unguarded → throws. Let me guard it (matching the `?? []` pattern I already used in `flattenSignals`):

Fixed. Now the full suite to confirm nothing else broke:

All 339 pass. Now lint, then the free-coding commit flow. Let me check lint and the version-bump interface:

No eslint installed. Let me check what the quality gate actually runs (prettier? tsc?) so I format correctly before committing:

No lint/formatter installed — quality is typecheck + tests + coverage (all green). My edits match the file style (no semicolons, single quotes, 2-space). Now the free-coding commit flow. First the required version bump, checking what it touches:

Version at 0.0.48. Now let me stage **only** my REQ-47 files (leaving REQ-20/REQ-46's in-flight changes untouched) and verify the staged set before committing:

Exactly my REQ-47 files staged; only REQ-20's two site JSON files remain unstaged (untouched). Note: the framework module changes I saw earlier are now committed — concurrent REQ-46/REQ-20 headless runs are active, which confirms the containment matters. My files never overlap theirs. Committing:

Committed `df5732b`. Now the free-coding gate transition — let me check the `move-to-free-coded` interface:

Let me verify the ticket state and re-run the affected suites (concurrent commits may have landed since):

Done. REQ-47 Parts A + B are implemented, tested, and moved to `free_coded`.

## What landed

**Part A — richer rendered projection** (`capture/extract.ts`, `sections.ts`, `types.ts`):
- Every rendered element now carries `box{x,y,w,h}`, `borderRadiusPx`, `boxShadow`, `a11yRole`, and geometry-derived `arrangement` (`row`/`stack`) — geometry descended from sections to individual headings/paragraphs/buttons.
- New `Field` type: text-free elements (inputs, dividers) captured with box/shape/a11y and **`nameSource`** (`placeholder`=inside vs `label`/`aria`=outside), paired on `a11yRole + document order`, failing safe to a `presence` delta when unmatched.
- `EXTRACT_SCRIPT` derives a11y role/name-source + arrangement **in-page**, as agreed.

**Part B — severity, not area** (`values-diff.ts`, `fidelity.ts`):
- Every delta tagged `kind` → fixed `tier` table (CRITICAL presence/containment/arrangement/position/text · HIGH size/fontSize/fontFamily · MEDIUM shape/borderLeft/gradient/fontWeight · LOW colour/overlay/anchor/…). Sort = `(tier, kind-within-tier, magnitude)`; pixel area is never an input.
- `contentAnchor` demoted to LOW (superseded by the new per-element CRITICAL `position` kind), which preserved the REQ-31/35 pinned orderings.

**The three REQ-20 misses now surface as CRITICAL text deltas** — hero block `Δy≈195px [position]`, email field `[containment]` placeholder-vs-label, subscribe button `[arrangement]` beside-vs-below — verified by UAT to outrank a large tonal delta.

## Validation
- **14 new UATs** (`test_UAT_FC_REQ-47_*`), incl. the in-page `EXTRACT_SCRIPT` run **end-to-end under jsdom** (real execution of the novel logic, since Chromium isn't in this sandbox) plus real-Chromium capture tests for CI.
- Full suite: **339/339 green**; typecheck clean. Caught and fixed one guard (`b.fields ?? []`) that the FakeDriver capture test surfaced.

## Containment
Touched only the fidelity tooling (`capture/` + `fidelity.ts`). Staged exactly those files + the version bump; left REQ-20's `gigabytealchemy` site JSON untouched. Notably, REQ-46/REQ-20 **headless runs committed their module work mid-session** — so the isolation mattered; there was zero file overlap.

Process: version bumped `0.0.47→0.0.48`, committed `df5732b` `[FREE-CODED]`, ticket at `free_coded` with the SHA recorded, body updated with the scope/decisions.

**Deferred:** Part C (structure-aware image diffs) per the ticket. Also worth a later doc pass — the ticket notes DOC-19 should point here ("why pixel area is not importance"); I didn't touch it since it's a separate doc task. Want me to pick up C, or open that DOC-19 update next?

<!-- xgd-chat-end -->