---
uid: doc-e8a65bcc
id: DOC-19
type: doc
title: 'How-To: Faithful Founder-Site Reproduction (successor runbook)'
created_by: xgd
created_at: '2026-07-03T01:39:12.471124+00:00'
updated_at: '2026-07-03T16:06:27.940904+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  doc_kind: architecture
---

# How-To: Faithful Founder-Site Reproduction

## Read me first (successor)

You are about to reproduce a captured site (e.g. gigabytealchemy.ai, faelan.com) "indistinguishably to the eye" using our own modules ([[REQ-20]] / [[REQ-21]]). This is a **procedural runbook** (doc_kind architecture for lack of a "howto" kind). The design *craft* (what reads expensive) lives in [[DOC-17]]; the taste/prompt rubric in [[DOC-16]]. This doc is *how you actually do the transcription without missing things*.

## The one rule

**Transcribe from the captured DOM. Do NOT reconstruct from memory + screenshot.**

Every fidelity miss in the gigabytealchemy pass came from building the page from a hand-authored config and *spot-checking* against the screenshot, instead of reading values out of the capture. The values were all right there.

## Sources of truth (ranked)

1. **`storage/references/<site>/<page>/raw.html`** — the rendered DOM. Inline `style="..."` and utility classes carry the *exact* values (colours as hex, gradients with direction+stops, `text-7xl`, `border-l-4 border-emerald-400`). **This is the ground truth.**
2. **`capture.json`** — structured, and as of [[REQ-31]] it now records the per-element **value manifest** (resolved colour, font-size/weight, line-height, letter-spacing, text-fill **gradient** with direction+stops, **left-bar** treatment, padding, plus section-level **scrim** and **content vertical anchor**). Tailwind/`var()` are already resolved to concrete values. **Caveat:** a bundle captured *before* REQ-31 (anything older than 2026-07-02) lacks these fields — **re-capture it** (`1c capture page <url>`) before trusting the diff, or it will silently not flag gradient/scrim/anchor. When in doubt, verify against `raw.html`.
3. **`screenshot.full.png`** — for **judgment only**. Vision is good at structure/composition and *bad* at the exact things listed below. Never read a value off the screenshot.

## What the screenshot hides (so you know to go to the DOM)

- **Near-neighbour colours** — `#F5E6A3` (subhead) vs `#FBBA72` (heading) both read "gold." Gold-vs-white is visible; gold-vs-gold is not.
- **Exact sizes** — `text-7xl` (72px) vs `text-4xl` (48px) both read "big-ish."
- **Gradient direction** — a 90deg (horizontal, multi-hue) vs 180deg (vertical) sweep is subtle.
- **Left-bars / callouts** — a `border-l-4 border-emerald-400 pl-6` reads as "some emphasis," easily mis-modelled as bold text.
- **Opacity / scrims** — `bg-slate-950/30` over an image is invisible unless you check.
- **Vertical anchor** — content pinned low (`pt-80` / flex `justify-end`) vs centred reads as "roughly in the middle."
- **Casing** — small-caps rendering (Cinzel) makes `Gigabyte Alchemy` look like `GIGABYTE ALCHEMY`; the literal text node differs even when the styling matches.

**All of the above are now mechanically flagged by the values-diff (below).** The screenshot is the *last* check, not the safety net.

## The mechanical gate — `1c values-diff` ([[REQ-31]], landed)

This is the anti-recurrence mechanism. It renders our draft, reads *its* computed styles through the same headless seam the eyes loop uses, and diffs field-by-field against the capture manifest — **before** any human judgement.

```
1c values-diff <slug> --ref storage/references/<site>/<page> [--source draft|published] [--sandbox] [--json]
```

- It emits a **severity-ranked delta list** `(element/section, property, expected, actual)` and **exits non-zero while any delta remains**. Clear every delta before judging by eye.
- Properties it compares, most-severe first: `missing` (element absent), `text` (verbatim content incl. **casing**), `color`, `gradient` (angle+stops), `overlay` (**hero scrim** colour+opacity), `borderLeft` (**callout bar**), `fontSizePx`, `contentAnchor` (**vertical anchor**, section-level), `fontWeight`, `fontFamily`, `lineHeightPx`, `paddingLeftPx`, `letterSpacingPx`.
- Text elements pair by case-folded, whitespace-collapsed text; section-level values (scrim, anchor) pair by ordinal index (the hero is `§0`).
- `--actual <manifest.json>` re-diffs an offline manifest without a browser (CI without Chromium).
- **What it does NOT catch:** fine-grained *relative* position of one element within a section (we record section-level anchor, not per-run geometry), and anything not modelled as a text run or section treatment (spacing rhythm, an extra/missing purely-visual element). Those remain eyes-only — see the SSIM-backstop idea parked in [[REQ-31]]/[[DOC-17]].

## Procedure (layered — each layer independently verifiable)

0. **Capture (or re-capture) the reference.** `1c capture page https://<site>/` → writes `storage/references/<host>/<page>/` (capture.json + raw.html + rendered.html + screenshot.full.png + mirrored assets). Re-capture if the existing bundle predates REQ-31.
1. **Read the capture.** Open `raw.html` and `capture.json`. Skim the section structure.
2. **Structure pass.** Map each captured `<section>` to a module + variant. Reconcile section count first.
3. **Values pass — per element, transcribe:** colour (resolved hex), font-size, weight, line-height, letter-spacing, gradient (**note direction + stops**), border/left-bar, padding/indent, opacity/scrim, and the section's vertical anchor. Translate Tailwind tokens to concrete values (`text-7xl`→4.5rem; `emerald-400`→#34d399; `slate-400`→#94a3b8) — or grep the mirrored CSS asset. Set them in the module config / theme palette. **Copy literal text verbatim, including casing** (see below).
4. **Treatment pass.** Gradients, callout bars, scrims/overlays, badges (soft vs solid), panel fills, content anchor.
5. **Diff gate.** `1c render <slug> --source draft`, then `1c values-diff <slug> --ref <bundle>`. **Fix every flagged delta and re-run until it exits clean.** This is where the six value deltas, the casing slip, and the scrim/anchor would each have been caught.
6. **Render → screenshot → eyes.** `1c shot <slug> --source draft --out <file>`. Use vision only to **judge** what the manifest can't encode (does the gradient read intentional? composition right?), never to read a value.

## Capabilities already in the framework (don't reinvent / don't assume absence)

- **hero**: `height: fold` (fills viewport), `headingTreatment: gold|accent`, `align`, `size`, **`scrim`** (overlay darkening) and **`contentAnchor: top|center|bottom`** ([[REQ-32]]); subhead renders **markdown** (multi-paragraph).
- **header**: `logoFont: display`, `logoTreatment: gold`, `logoSize: sm|md|lg|xl`, `align`; `overlay` variant composites the header over the next band (shared image), via the render pipeline.
- **footer**: `layout: center|spread` (justified copyright/links).
- **contact-form**: `width: full|half` — consecutive `half` bands auto-group into one `fc-row` (side-by-side forms).
- **services-grid**: card `accent` (incl. `secondary`), `badge` (soft pills), `surface: default|muted` (panel card), `checklist` (ticks follow badge variant), `stacked` variant.
- **text-block**: **callout / left-bar** treatment ([[REQ-32]]).
- **palette**: 10 roles incl. optional `secondary` (a second functional accent) and a **cool-neutral (slate)** role ([[REQ-32]]).
- **wordmark**: multi-hue **horizontal gradient** direction ([[REQ-32]]).
- **background/overlay** ([[REQ-14]]), **layer** free-positioning ([[REQ-15]]), **motion** ([[REQ-16]]).

The fidelity primitives surfaced by the gigabytealchemy import — gradient direction, callout left-bar, hero scrim + content anchor, cool-neutral role, display-scale logo — **have landed** ([[REQ-32]] / [[REQ-28]]). Check [[REQ-32]] before assuming a treatment is impossible; almost every gap was closed by *generalizing* an existing module, not adding one.

## Process reminders

- **Free-coding**: framework *code* changes need a scope ticket + `test_UAT_FC_<ID>_*` UATs + `[FREE-CODED]` commit + `bin/project/xgd_version_bump` + `move-to-free-coded`. **Site-def / config / theme edits are exempt** — most of a faithful repro is config.
- **Drive from captured values, gate on the diff, verify by eye — in that order.** That inversion is the whole lesson. Companion: [[DOC-17]] §D (import/reproduction lessons), [[DOC-13]] (capture/eyes), [[REQ-20]], [[REQ-31]].

## Module authoring: generalize before adding

When a repro needs a capability the framework lacks, **generalize an existing module (dial / variant / treatment / content field) before creating a new one** — a new module is the last resort. Cross-cutting layout/composition → a shared primitive (e.g. `fc-row`), not a bespoke module. This is a project rule (see CLAUDE.md "Generalize Modules Before Adding New Ones"); the gigabytealchemy pass closed almost every gap this way with zero new modules.

## Transcribe literal text verbatim — including casing

Copy text content exactly as captured; do **not** normalize it (uppercase, title-case, trim punctuation). Casing can carry design: gigabytealchemy's wordmark is the mixed-case string `Gigabyte Alchemy` rendered in **Cinzel**, whose lowercase glyphs are small capitals — so the real capitals `G`/`A` read larger and the rest as small caps. Uppercasing it to `GIGABYTE ALCHEMY` flattens both the content *and* the typographic intent. This was the third repeat of the transcribe-don't-reconstruct failure — the first two were value-level (gold subhead rendered white, wordmark size/gradient); this one is content-level. The values-diff now flags it as a `text` delta, but the rule stands: **the literal text node is a captured value too.**
