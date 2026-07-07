---
uid: doc-e8a65bcc
id: DOC-19
type: doc
title: 'How-To: Faithful Founder-Site Reproduction (successor runbook)'
created_by: xgd
created_at: '2026-07-03T01:39:12.471124+00:00'
updated_at: '2026-07-03T21:57:46.977631+00:00'
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

1. **`storage/references/<site>/<page>/raw.html`** — the rendered DOM. Inline `style="..."` and utility classes carry the *exact* values (colours as hex, gradients with direction+stops, `text-7xl`, `border-l-4 border-emerald-400`). **This is the ground truth.** For an art-directed montage this includes the **mirrored CSS asset** (`assets/*.css`): per-photo `position`/`rotate`/`width`/`border-radius`/`mask-image` live there, *not* in `capture.json` (see the perceptual gate below).
2. **`capture.json`** — structured, and as of [[REQ-31]] it now records the per-element **value manifest** (resolved colour, font-size/weight, line-height, letter-spacing, text-fill **gradient** with direction+stops, **left-bar** treatment, padding, plus section-level **scrim** and **content vertical anchor**). Tailwind/`var()` are already resolved to concrete values. **Caveat:** a bundle captured *before* REQ-31 (anything older than 2026-07-02) lacks these fields — **re-capture it** (`1c capture page <url>`) before trusting the diff, or it will silently not flag gradient/scrim/anchor. When in doubt, verify against `raw.html`.
3. **`screenshot.full.png`** — for **judgment only** (and as the reference image for the perceptual gate `1c diff`). Vision is good at structure/composition and *bad* at the exact things listed below. Never read a value off the screenshot.

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
- **What it does NOT catch:** the entire **art-directed layer** — freely-positioned photos, their rotation/scale/position, shape (circle/rounded), masks, borders, shadows — and fine-grained *relative* position of one element within a section. A captured montage records its text runs and section anchor, but its image children come through as `items: []`, so a photo collage can pass values-diff "clean" while every photo is visibly wrong. **This is exactly what the perceptual gate `1c diff` is for** (below; the SSIM-backstop idea once parked in [[REQ-31]]/[[DOC-17]], now landed as [[REQ-38]]). Run both.

## The perceptual gate — `1c diff` ([[REQ-38]], landed)

The value-diff reads *computed styles*; `1c diff` reads *pixels*. It is the sibling gate for everything the manifest is structurally blind to — above all the **art-directed layer** (a montage of positioned, rotated, masked photos). It renders + shoots the reproduction (same seam as `1c shot`), diffs it against `screenshot.full.png`, and emits a **severity-ranked `regions.json`** of automatically-derived regions of interest, each with a zoomed **ref / ours / diff crop triptych**.

```
1c diff <slug> --ref storage/references/<site>/<page> [--source draft|published] [--sandbox] [--out <dir>]
1c diff --ref <bundleDir|refPng> --actual <png> [--out <dir>]   # offline re-diff, no Chromium
1c crop <image> --box x,y,w,h [--out <png>]                     # magnify any existing PNG (ref, ours, or a diff)
```

- Read the **overall mean diff / 255**, the **`% pixels > threshold`**, and the **horizontal-band profile** (localizes *where vertically* the deltas are). Then open the top-ranked region triptychs and fix worst-first. This is a loop: fix the top region → `1c render` → `1c diff` → repeat; each fix should darken its region. Drive the number down — the faelan pass went **20.5 → 2.7 / 255** this way.
- Trust the **block-averaged** heatmap over the raw pixel heatmap for "real delta vs 1px noise": block-averaging de-noises sub-pixel registration jitter.
- `regions.json` is derived **mechanically** (block-average → threshold → connected-components → score = sum of block-diffs → rank), never hand-authored — so it catches both "large & faint" and "small & intense" and pre-crops them for the eyes.

### Match the viewport, or everything lies

`screenshot.full.png` was shot at the **capture's** viewport (faelan: 1280×**1195** full-page; the montage `100vh` band is only ~800 tall because the next section starts at `y=800` per `capture.json`). Shoot your reproduction at the **same** viewport (`1c shot --viewport desktop` = 1280×800, full-page) — otherwise a `100vh` band rescales and *every* percentage-positioned element diffs vertically, a false cascade that hides the real deltas. When you diff against the *live* site instead of the capture, shoot both sides at one viewport.

### Read the diff — real gap vs render noise

- A whole element lit as an **embossed edge-map** = a small *position / rotation / scale* offset (or a wrong transform pivot), **not** a colour delta.
- Two *separated* ghosts of the same thing = a gross position offset.
- A faint all-over speckle over photographic areas = **JPEG recompression** of a mirrored asset (or two render engines' AA) — near the floor, ignore it.
- Turn "looks off" into a **number before touching config**: measure computed geometry on both sides (`getComputedStyle().{left,top,width,height}`, or `boundingBox()`) — the pixels tell you *where*, the measurements tell you *what* and *how much*.

### Watch-list — geometry/treatment deltas the value-diff can't see (all real, all from the faelan pass)

- **Corner rounding — check this first when a photo's whole outline glows.** The original collage photos combine a small crisp `border-radius` (6–8px) *with* a soft-mask; a mask-*only* repro has soft *faded* corners instead of crisp rounded ones, and the difference lights up the entire photo **perimeter** in the diff. Fix: add `shape: rounded` alongside `edge: soft-mask` — they compose (border-radius on the `<img>` + the feather).
- **Rotation pivot.** A layer child rotates about its **centre** (the CSS default the source relies on); a `transform-origin: top left` swings every rotated photo away from its intended spot despite matching `top`/`left`. Whole-montage emboss → suspect the pivot.
- **Circle vs ellipse.** `shape: circle` needs a *square* box; if a motion wrapper or a percentage height collapses the image's height, `object-fit` yields an **ellipse**. Crop the portrait (`1c crop`) and check width == height; the fix is `aspect-ratio: 1` + not depending on a percentage height.
- **Mask feather geometry.** A farthest-corner `ellipse at center` feathers far more than the source's box-sized `ellipse 92% 92%` — **halos ring the photos**. Match the ellipse size *and* the opaque stop.
- **Text vertical rhythm.** A positioned wordmark/label sits high unless its line-height (`leading`) is set; a markdown link underline hugs the letters without `text-underline-offset`; a `shadow: glow` vs `soft` changes the halo. These show as thin **doubled outlines** on the text.
- **Dynamic content.** `© {year}` renders the *build* year (`© 2026`) vs the source's hardcoded `© 2025` — a permanent single-run delta you can knowingly accept.

**Run both gates:** `values-diff` for the band-stack (text, colour, section treatments); `1c diff` for the art-directed layer. A montage that passes `values-diff` has barely been checked.

## Procedure (layered — each layer independently verifiable)

0. **Capture (or re-capture) the reference.** `1c capture page https://<site>/` → writes `storage/references/<host>/<page>/` (capture.json + raw.html + rendered.html + screenshot.full.png + mirrored assets). Re-capture if the existing bundle predates REQ-31.
1. **Read the capture.** Open `raw.html` and `capture.json`. Skim the section structure. For an art-directed page, also open the mirrored CSS asset — per-photo geometry/treatments live there.
2. **Structure pass.** Map each captured `<section>` to a module + variant. Reconcile section count first.
3. **Values pass — per element, transcribe:** colour (resolved hex), font-size, weight, line-height, letter-spacing, gradient (**note direction + stops**), border/left-bar, padding/indent, opacity/scrim, and the section's vertical anchor. Translate Tailwind tokens to concrete values (`text-7xl`→4.5rem; `emerald-400`→#34d399; `slate-400`→#94a3b8) — or grep the mirrored CSS asset. Set them in the module config / theme palette. **Copy literal text verbatim, including casing** (see below).
4. **Treatment pass.** Gradients, callout bars, scrims/overlays, badges (soft vs solid), panel fills, content anchor. For an art-directed layer: per-photo position/rotation/width, `shape` (circle/**rounded** — corners!), `edge`/mask feather, shadow, border; wordmark `leading`/`tracking`/`shadow`.
5. **Value diff gate.** `1c render <slug> --source draft`, then `1c values-diff <slug> --ref <bundle>`. **Fix every flagged delta and re-run until it exits clean.** This is where the six value deltas, the casing slip, and the scrim/anchor would each have been caught.
6. **Perceptual diff gate.** `1c diff <slug> --ref <bundle>` — mandatory for any art-directed layer the value-diff is blind to. Drive the mean diff down worst-first: open the top-ranked region triptychs, fix, `1c render`, re-`diff`. Use the watch-list above (corner rounding, rotation pivot, circle/ellipse, mask feather, text rhythm). Skip only for a pure band-stack with no positioned imagery.
7. **Render → screenshot → eyes.** `1c shot <slug> --source draft --out <file>`. Use vision only to **judge** what neither gate encodes (does the gradient read intentional? composition right?), never to read a value.

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
- **layer art-direction ([[REQ-32]] cap 5)** — image children: `shape: circle|rounded` (`circle` is square via `aspect-ratio`), `edge: soft-mask` with per-child `feather`, token-backed `shadow` (incl. `xl`) and `border` (width + palette-role); text children: `typography` (`size`/`weight`/`color`/`font`/`tracking`/`leading`/`align`/`shadow: soft|glow`); rotation is about the child centre. **Known gap:** percentage `top` on layer children can shift ~1 line in Safari/Firefox vs Chromium (min-height-derived band height) — deferred, tracked in [[REQ-32]].

The fidelity primitives surfaced by the gigabytealchemy import — gradient direction, callout left-bar, hero scrim + content anchor, cool-neutral role, display-scale logo — **have landed** ([[REQ-32]] / [[REQ-28]]). Check [[REQ-32]] before assuming a treatment is impossible; almost every gap was closed by *generalizing* an existing module, not adding one.

## Process reminders

- **Free-coding**: framework *code* changes need a scope ticket + `test_UAT_FC_<ID>_*` UATs + `[FREE-CODED]` commit + `bin/project/xgd_version_bump` + `move-to-free-coded`. **Site-def / config / theme edits are exempt** — most of a faithful repro is config (incl. per-photo layer geometry/treatments).
- **Drive from captured values, gate on the value-diff *and* the perceptual diff, verify by eye — in that order.** That inversion is the whole lesson. Companion: [[DOC-17]] §D (import/reproduction lessons), [[DOC-13]] (capture/eyes), [[REQ-20]], [[REQ-31]], [[REQ-38]].

## Module authoring: generalize before adding

When a repro needs a capability the framework lacks, **generalize an existing module (dial / variant / treatment / content field) before creating a new one** — a new module is the last resort. Cross-cutting layout/composition → a shared primitive (e.g. `fc-row`), not a bespoke module. This is a project rule (see CLAUDE.md "Generalize Modules Before Adding New Ones"); the gigabytealchemy pass closed almost every gap this way with zero new modules. The faelan art-direction pass ([[REQ-21]]) did the same — every montage treatment (corners, rotation pivot, circle, feather, shadow, border, text leading/shadow) closed by generalizing the `layer` primitive ([[REQ-32]] cap 5), zero new modules.

## Transcribe literal text verbatim — including casing

Copy text content exactly as captured; do **not** normalize it (uppercase, title-case, trim punctuation). Casing can carry design: gigabytealchemy's wordmark is the mixed-case string `Gigabyte Alchemy` rendered in **Cinzel**, whose lowercase glyphs are small capitals — so the real capitals `G`/`A` read larger and the rest as small caps. Uppercasing it to `GIGABYTE ALCHEMY` flattens both the content *and* the typographic intent. This was the third repeat of the transcribe-don't-reconstruct failure — the first two were value-level (gold subhead rendered white, wordmark size/gradient); this one is content-level. The values-diff now flags it as a `text` delta, but the rule stands: **the literal text node is a captured value too.**
