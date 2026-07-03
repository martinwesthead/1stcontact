---
uid: request-dd68c68a
id: REQ-31
type: request
title: 'Fidelity verification loop: capture computed per-element values + mechanical
  values-diff'
created_by: xgd
created_at: '2026-07-03T01:37:35.057755+00:00'
updated_at: '2026-07-03T15:26:30.258104+00:00'
completed_at: null
last_field_updated: status
status: free_coded
fields:
  priority: high
  story_points: 5
  auto_merge_back: true
  needs_review: false
  commits:
  - 6fdd57423dc650d4666a027508d0272685abcb4c
  - c7219d6510814c69a5ab03ed0b6833d522e38e6f
  version: 0.0.25
---

## Problem

Import reproduction (REQ-20) drifts at the **value level** — exact colours, font sizes, gradients, and treatments — and nothing catches it mechanically. In the gigabytealchemy pass, six value-level deltas (gold subhead rendered white, wordmark `text-7xl` rendered smaller, vertical vs horizontal wordmark gradient, green/amber callout bars rendered bold, footer `text-slate-400` vs cream) were all **explicit in the captured DOM** yet shipped wrong. They were missed because the repro was built screenshot-first and the capture was consulted only selectively.

Root cause is structural: two authored representations exist (the original's rendered DOM vs our module config) and **nothing reconciles them field-by-field**. Screenshots hide exactly this class of delta (near-neighbour colours like #F5E6A3 vs #FBBA72, 7xl vs 4xl, gradient *direction*, subtle left-bars, opacity/scrims), so the eyes gate can't be the safety net for it.

## Two parts

**1. Capture records computed per-element values (REQ-12 extension).**
Today `capture.json` recorded only 2 of the 6 flagged values; `raw.html` carried all six as inline styles / utility classes. The capture must emit a structured, per-section **expected-values manifest**: for each text/box element — colour (resolved hex), font-size, font-weight, line-height, letter-spacing, gradient (with direction + stops), border/left-bar treatment, padding/indent, opacity/scrim. Resolve Tailwind utilities and `var()` chains to concrete values at capture time (headless computed styles).

**2. Repro emits the same values and diffs them (REQ-13 extension).**
Render our reproduction, read *its* computed styles, and diff field-by-field against the manifest. Emit a ranked delta report (element, property, expected, actual) **before** human review. Vision is then reserved for what the manifest can't encode ("does the gradient read intentional," "is the composition right"), not for reading a hex.

## Acceptance criteria

- Given `storage/references/gigabytealchemy.ai/index/` (capture) and our rendered draft, the values-diff **flags all six known deltas** (subhead colour, wordmark size, wordmark gradient direction, the two callout left-bars, footer text colour).
- The manifest is structured data (not prose) and is the single artifact both the diff and a human can read.
- No false "match" when a near-neighbour colour differs (#F5E6A3 vs #FBBA72 must diff).

## Why this is the anti-recurrence fix

This converts the whole class of "font-size and colour you could just read off" from *missed-by-eye* to *mechanically-flagged*. It is the mechanism the successor runbook (new DOC How-To) depends on. Companion to [[DOC-13]] (capture/eyes), gates [[REQ-20]] / [[REQ-21]] fidelity.


---

## Implementation (as landed — commit 6fdd574, v0.0.23)

**Part 1 — capture records per-element values.** `ContentRun` (persisted in
`capture.json`) and the raw extraction gained optional per-element fields:
`lineHeightPx`, `letterSpacingPx`, `gradient` ({angleDeg, stops} — a text-fill
gradient normalized from `background-clip: text` background-image), `borderLeft`
({widthPx, color}), and `paddingLeftPx`. All read from computed styles in the
headless browser; Tailwind/`var()` already resolved. Fields are optional so
pre-REQ-31 bundles still parse.

**Part 2 — mechanical values-diff.** New module
`tools/generate/src/cli/capture/values-diff.ts`:
- `flattenCapture(capture)` / `flattenSignals(signals)` project either side to a
  flat **value manifest** — one element per verbatim text run, carrying its
  resolved styling. Verbatim text (captured exactly, DOC-13 §5) is the join key.
- `diffManifests(expected, actual)` aligns by text (FIFO queues so repeated
  texts pair in document order) and diffs each field, emitting a
  **severity-ranked** `ValueDelta[]` (missing/colour/gradient/border rank above
  weight/family above line-height/padding/letter-spacing). Only fields present on
  the expected (reference) side are compared. Colour compare is exact hex
  (case-insensitive) so near-neighbour golds diff; gradient direction compares
  angle within a 20° tolerance plus stop colours.

**Command** (`tools/generate/src/cli/fidelity.ts`):
`1c values-diff <slug> --ref <captureBundleDir> [--source draft|published]
[--out <file>] [--json]` — renders + serves the draft over loopback, reads *its*
computed styles through the same BrowserDriver seam the eyes loop uses, and diffs
against the capture. `--actual <manifest.json>` short-circuits the browser
(offline re-diff / CI without Chromium). Exits non-zero when any delta remains.

**Evidence** — `tests/req31-values-diff.test.ts` (12 UATs): the six known
gigabytealchemy deltas all flagged through the real command; faithful repro →
zero deltas; near-neighbour `#f5e6a3` vs `#fbba72` diffs; ranking; missing
elements; repeated-text alignment; gradient normalization; and real-Chromium
capture of gradient + left-bar out of the DOM (`tests/fixtures/capture/values.html`).

**Note / follow-on:** the existing `storage/references/gigabytealchemy.ai/index/`
capture predates the Part-1 fields, so a *live* diff there catches colour/size
deltas but not gradient/left-bar until the site is re-captured (`1c capture page`).
The diff degrades gracefully — it only compares fields the reference carries.


---

## Extension — verbatim text/casing delta (commit c7219d6, v0.0.25)

**Gap found and closed.** The diff aligned elements on a case-folded,
whitespace-collapsed join key (`norm()`), then compared only *styling* fields —
so a pairing that survived could still differ in **casing**, and the diff was
blind to it. This is exactly the third-instance gigabytealchemy miss: small-caps
"Gigabyte Alchemy" rendered as literal "GIGABYTE ALCHEMY" — a content delta both
screenshots and computed styles miss (fontFamily matches; only the text casing
is wrong). `DeltaProperty` had no member that could represent it.

**Fix.** Split the normal form into `collapse()` (trim + collapse internal
whitespace, **case-preserving**) and `norm()` (= `collapse()` lowercased, still
the join key). After pairing, compare `collapse(exp.text)` vs `collapse(act.text)`
case-sensitively and emit a new `text` delta (severity 95 — below `missing` 100,
above every styling field, since wrong content outranks wrong styling).
Whitespace-only formatting noise stays ignored because both sides collapse first.

**Evidence** — three new UATs in `tests/req31-values-diff.test.ts`:
casing delta flagged while the pair still matches (not "missing");
whitespace-only difference not flagged; `text` delta ranks above a colour delta.
Full suite green (189 tests).

**Still eyes-only (out of this text-run diff's scope, tracked on [[REQ-32]]):**
non-text treatments — the hero **scrim/overlay** and **content vertical anchor** —
are not text runs, so the manifest can't encode them and the diff can't flag them.