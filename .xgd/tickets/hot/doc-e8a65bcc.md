---
uid: doc-e8a65bcc
id: DOC-19
type: doc
title: 'How-To: Faithful Founder-Site Reproduction (successor runbook)'
created_by: xgd
created_at: '2026-07-03T01:39:12.471124+00:00'
updated_at: '2026-07-03T02:18:57.980438+00:00'
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

1. **`storage/references/<site>/<page>/raw.html`** — the rendered DOM. Inline `style="..."` and utility classes carry the *exact* values (colours as hex, gradients with direction+stops, `text-7xl`, `border-l-4 border-emerald-400`). **This is the source of truth.**
2. **`capture.json`** — structured, but **partial** — today it dropped the wordmark gradient, the `7xl` size, and the callout bars. Use it, but never assume it is complete; verify against `raw.html`. (Closing this is [[REQ-31]].)
3. **`screenshot.full.png`** — for **judgment only**. Vision is good at structure/composition and *bad* at the exact things listed below. Never read a value off the screenshot.

## What the screenshot hides (so you know to go to the DOM)

- **Near-neighbour colours** — `#F5E6A3` (subhead) vs `#FBBA72` (heading) both read "gold." Gold-vs-white is visible; gold-vs-gold is not.
- **Exact sizes** — `text-7xl` (72px) vs `text-4xl` (48px) both read "big-ish."
- **Gradient direction** — a 90deg (horizontal, multi-hue) vs 180deg (vertical) sweep is subtle.
- **Left-bars / callouts** — a `border-l-4 border-emerald-400 pl-6` reads as "some emphasis," easily mis-modelled as bold text.
- **Opacity / scrims** — `bg-slate-950/30` over an image is invisible unless you check.

## Procedure (layered — each layer independently verifiable)

0. **Read the capture.** Open `raw.html` and `capture.json` for the page. Skim the section structure.
1. **Structure pass.** Map each captured `<section>` to a module + variant. Reconcile section count first.
2. **Values pass — per element, transcribe:** colour (resolved hex), font-size, weight, line-height, letter-spacing, gradient (**note direction + stops**), border/left-bar, padding/indent, opacity/scrim. Translate Tailwind tokens to concrete values (`text-7xl`->4.5rem; `emerald-400`->#34d399; `slate-400`->#94a3b8) — or grep the mirrored CSS asset. Set them in the module config / theme palette.
3. **Treatment pass.** Gradients, callout bars, scrims/overlays, badges (soft vs solid), panel fills.
4. **Render -> screenshot -> eyes.** `1c render <site> --source draft` then `1c shot ... --out ...`. Use vision to **judge** (does the gradient read intentional? composition right?), not to read values. When [[REQ-31]] lands, run the values-diff here and clear every flagged delta before judging by eye.

## Capabilities already in the framework (don't reinvent / don't assume absence)

- **hero**: `height: fold` (fills viewport), `headingTreatment: gold|accent`, `align`, `size`; subhead renders **markdown** (multi-paragraph).
- **header**: `logoFont: display`, `logoTreatment: gold`, `logoSize: sm|md|lg|xl`, `align`; `overlay` variant composites the header over the next band (shared image), via the render pipeline.
- **footer**: `layout: center|spread` (justified copyright/links).
- **contact-form**: `width: full|half` — consecutive `half` bands auto-group into one `fc-row` (side-by-side forms).
- **services-grid**: card `accent` (incl. `secondary`), `badge` (soft pills), `surface: default|muted` (panel card), `checklist` (ticks follow badge variant), `stacked` variant.
- **palette**: 10 roles incl. optional `secondary` (a second functional accent).
- **background/overlay** ([[REQ-14]]), **layer** free-positioning ([[REQ-15]]), **motion** ([[REQ-16]]).

## Known still-missing capabilities (see [[REQ-32]])

Wordmark **multi-hue horizontal gradient**; hero **scrim + content vertical anchor**; text-block **callout/left-bar treatment**; **cool-neutral (slate) palette role**; a `logoSize` step at display 7xl. Check [[REQ-32]] before assuming a treatment is impossible.

## Process reminders

- **Free-coding**: framework *code* changes need a scope ticket + `test_UAT_FC_<ID>_*` UATs + `[FREE-CODED]` commit + `bin/project/xgd_version_bump` + `move-to-free-coded`. **Site-def / config / theme edits are exempt** — most of a faithful repro is config.
- **Drive from captured values, verify by eye.** That inversion is the whole lesson. Companion: [[DOC-17]] SS D (import/reproduction lessons), [[REQ-20]], [[REQ-31]].


## Module authoring: generalize before adding

When a repro needs a capability the framework lacks, **generalize an existing module (dial / variant / treatment / content field) before creating a new one** — a new module is the last resort. Cross-cutting layout/composition → a shared primitive (e.g. `fc-row`), not a bespoke module. This is a project rule (see CLAUDE.md "Generalize Modules Before Adding New Ones"); the gigabytealchemy pass closed almost every gap this way with zero new modules.


## Transcribe literal text verbatim — including casing

Copy text content exactly as captured; do **not** normalize it (uppercase, title-case, trim punctuation). Casing can carry design: gigabytealchemy's wordmark is the mixed-case string `Gigabyte Alchemy` rendered in **Cinzel**, whose lowercase glyphs are small capitals — so the real capitals `G`/`A` read larger and the rest as small caps. Uppercasing it to `GIGABYTE ALCHEMY` flattens both the content *and* the typographic intent. This was the third repeat of the transcribe-don't-reconstruct failure — the first two were value-level (gold subhead rendered white, wordmark size/gradient); this one is content-level. Rule of thumb: **the literal text node is a captured value too.**
