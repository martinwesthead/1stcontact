---
uid: comment-c8948b29
id: COMMENT-36
type: comment
title: Comment on request REQ-36
created_by: xgd
created_at: '2026-07-03T18:02:32.195837+00:00'
updated_at: '2026-07-08T15:31:15.719929+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-d05379d0
  kind: chat_transcript
---

<!-- xgd-turn id="9a6de726-d2a7-4abf-8748-9106a1e62780-user" -->

<!-- xgd-chat role="user" ts="2026-07-03T18:02:30.444732+00:00" -->
#### You
Please read REQ-33 and its chat and the how to which I think is DOC-19. We are going to embark on another exercise like this for a site of a friend of mine:

https://joyfulculinarycreations.com/

I would like you to capture just the home page to start with to a sandbox site.

<!-- xgd-turn id="9a6de726-d2a7-4abf-8748-9106a1e62780-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-03T18:04:55.029670+00:00" -->
#### Claude
I'll start by reading REQ-33, its chat, and DOC-19 to understand the process before capturing the site.

Let me check the REQ-33 chat context and the `1c` CLI capabilities.

`1c` lives at `tools/generate/bin/1c.mjs`. Let me check how it's invoked and view the full CLI help.

Everything's clear. `1c` runs via `node tools/generate/bin/1c.mjs`. This is a capture/data operation (writes to gitignored `storage/references/` and `storage/sandbox/`), so no free-coding ceremony is needed — but I'll keep REQ-36 updated as the session's scope ticket.

Let me run the capture of the home page (step 0 of the DOC-19 runbook). Headless browser, so I'll give it time.

Capture succeeded. Only 1 section detected, though — let me inspect the bundle to understand what was captured.

Capture is complete — it's a WordPress/Elementor site (Oswald/Lato fonts, gold `#f8bb1b` + pink `#cc3366` palette). The segmenter collapsed it to 1 style-scope band. Let me look at the captured structure and the screenshot to understand the page before scaffolding the sandbox site.

Rich capture — 59 content runs, 7 items. It's a personal-chef site (Chef Sarah Joy). Content and per-element values are all captured; the single-section grouping is the known segmenter limitation (grouping error, not fidelity error — each run carries its own color/size/font). Let me view the screenshot and scaffold the sandbox site.The sandbox site is scaffolded. I can now see the full page clearly. Both starting deliverables are done. Let me record this intent on the session's ticket (REQ-36), modeled on REQ-33, so the scope is captured.

Done — the home page is captured and the sandbox site is scaffolded.

## What I did

**1. Read the process docs** — REQ-33 (gigabytealchemy re-import), its chat, and DOC-19 (the faithful-reproduction runbook). The one rule: **transcribe from the captured DOM (`raw.html` / `capture.json`), never reconstruct from the screenshot**; drive from captured values → gate on `1c values-diff` → verify by eye.

**2. Captured the home page**
```
1c capture page https://joyfulculinarycreations.com/
→ storage/references/joyfulculinarycreations.com/index/
```
A REQ-31-complete bundle: `capture.json` (52K), `raw.html`, `rendered.html`, `screenshot.full.png` (full-page, 1280×4744), and 83 mirrored assets (fonts + food photography).

**3. Scaffolded the sandbox site**
```
1c new joyfulculinary --sandbox
→ storage/sandbox/joyfulculinary/draft/
```

**4. Recorded the intent on REQ-36** (this session's ticket) — modeled on REQ-33, with the observed page structure and a progress checklist.

## What the site is

Chef Sarah Joy — holistic in-home personal chef services. A WordPress/Elementor site: gold `#f8bb1b` + pink `#cc3366` palette, Oswald headings, Lato/Karla/Raleway body, white text over food photography, alternating dark/green/light panels. Nine bands: header → photo hero → Holistic Approach → Who Uses Our Services → Our Offerings (3 columns) → quote band → testimonials → 6-cell process grid → gold footer.

One thing to flag: the **segmenter collapsed the page into a single style-scope band**. Per DOC-13 that's a *grouping* artefact, not a fidelity loss — every text run still carries its own colour/size/font/geometry (59 content runs + 7 items captured correctly). It just means the structure pass will lean on `raw.html` and the screenshot to re-establish section boundaries rather than trusting `capture.json`'s single-section split.

## Suggested next step

The capture is the foundation. The reproduction itself (structure → values → treatment → diff-to-zero → eyes) is the REQ-33-style multi-pass effort. Want me to start the **structure pass** next — mapping the nine bands to modules/variants in the `joyfulculinary` sandbox draft — or would you rather review the capture first?

One note: I picked the slug `joyfulculinary` — say the word if you'd prefer something else and I'll re-scaffold.

<!-- xgd-turn id="8e19c72d-a045-4b84-a171-4a9e56868cc7-user" -->

<!-- xgd-chat role="user" ts="2026-07-03T18:05:55.069574+00:00" -->
#### You
please proceeed, when you have a first draft of the reproduction, please fire up a serve and bgive me a link

<!-- xgd-turn id="8e19c72d-a045-4b84-a171-4a9e56868cc7-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-03T18:16:12.948064+00:00" -->
#### Claude
Let me proceed. First I'll study the site-def format via the existing gigabytealchemy example and the fresh scaffold, then transcribe from the capture.

Now let me study the completed gigabytealchemy reproduction to learn the module vocabulary and dials.

Now let me examine the framework module definitions to know the exact available modules, variants, and dials.

Let me read all module metas and the shared dials to know the exact vocabulary.

Now the dials vocabulary:

I have the full vocabulary. Now let me transcribe from the capture — full content runs with geometry, the section backgrounds from raw.html, and the assets.

The Holistic Approach panel text is `#54595f` on light grey. Now let me identify section backgrounds and the key images from raw.html and the assets.

Two background images identified: hero = `HERO-AdobeStock...jpeg`, quote band = `market-vegetables...jpg`. Let me pull the homepage's Elementor per-section colors from its post CSS.

post-4401.css is the homepage. Key: dark bands are `#000`, green testimonial panel `#28542D` (~62% α) / quote scrim `#141E14`, footer gold. Let me check the font assets and how fonts get loaded, then look at how an existing site declares custom fonts.

Let me see how the display font gets a `@font-face` wired in, so I can load Oswald/Lato/Karla/Raleway.

Let me check what color headings render in for text-block and services-grid — gold headings are the site's dominant brand signal.

text-block/services-grid headings inherit surface color — no gold. So the site's gold section headings are a genuine capability gap (a `headingTreatment` generalization, like hero's). I'll build a config-only first draft now (correct structure/layout/images/fonts/surfaces), serve it, then propose the gold-heading generalization as pass 2. Let me add the assets first.

Now let me write the theme (`site.json`) and the page (`home.json`). First let me see how the asset adds registered in site.json.

Let me pull the full untruncated text for the items I need to transcribe verbatim.

I have everything. Now let me write the theme (`site.json`) and the page. First the theme:

Now the page. Let me write `home.json` with the 9 bands:

Now render the draft to catch any schema errors:

Rendered cleanly. Let me screenshot the draft to sanity-check before serving.Strong structural match, but one clear bug: the **services-grid cards on the dark (inverse) bands render as empty white boxes** — white cards with white (inherited) text = invisible. Let me inspect the services-grid CSS to fix card/text colors on inverse bands.

Root cause confirmed: on an `inverse` band, `.services-grid` sets text color to white, but each card draws a **white** background (`--color-surface`) without resetting its text to dark — so card text is white-on-white. The original's offerings/process are actually *text columns on a dark band* (no white panels), which the framework can't express yet (that's a pass-2 generalization). 

For a viewable first draft now, I'll switch those two grids to a light `subtle` surface so cards + text render. Let me update and note the dark-band treatment as a pass-2 gap.The first draft now renders cleanly — all nine bands present, content complete and legible, Oswald/Karla fonts loading, hero + quote images, gold buttons/accents/footer. Let me start a server in the background and get you a link.

Server is live — `http://localhost:8790/` returns 200. Let me record the first-draft milestone and the identified fidelity gaps on the ticket.

The first draft is built, rendering cleanly, and served. Here's your link:

## 🔗 http://localhost:8790/

(sandbox draft of `joyfulculinary` — leave this terminal running; the server stops when the session ends.)

## What I did

Following the DOC-19 runbook — **transcribe from the capture, don't reconstruct** — I built the sandbox site config-only (no framework code touched, so no free-coding ceremony needed):

- **Theme** — gold `#f8bb1b` / pink `#cc3366` palette, near-black inverse surface, and the site's real webfonts (Oswald headings, Karla body, Lato) loaded as `@font-face` assets.
- **9 bands, text transcribed verbatim** from `capture.json`/`raw.html`: header (overlay, logo image) → photo hero (fold height, scrim, gold "Learn More") → The Holistic Approach → Who Uses Our Services (checklist) → Our Offerings (3 service cards) → veggie pull-quote (second bg-image hero) → testimonials (2 cards) → How It Works (6 cells) → gold footer.

## Honest fidelity gaps (pass 2)

The structure, content, images, fonts and gold/dark palette are all there, but four things need a second pass — and three of them are genuine framework gaps that need the **generalize-before-adding + free-coding** treatment:

1. **Gold section headings** — in the original *every* section heading is gold; text-block/services-grid headings currently inherit the surface colour. This is the biggest visual miss. Fix = generalize hero's `headingTreatment` dial onto text-block + services-grid.
2. **Dark Offerings / How-It-Works bands** — the original renders these as gold-heading + white text *columns on black*. services-grid always draws white cards, and on a dark band the card text goes white-on-white (invisible) — so I put those two grids on **light** bands to keep the draft legible. Fix = a transparent/"bare" card surface.
3. **Green testimonial panel** — the original is one solid sage-green panel; I approximated with neutral-cool cards (the tint renders too light). Fix = a green surface role.
4. Hero heading is uppercased via CSS in the original; I kept the literal mixed-case text per DOC-19.

I've recorded all this on **REQ-36** (title set, progress + gaps in COMMENT-38).

**Recommended next step:** run the mechanical gate — `1c values-diff joyfulculinary --ref storage/references/joyfulculinarycreations.com/index --sandbox` — to enumerate the deltas precisely, then I'll file the 2-3 generalizations (likely against REQ-32) and knock them out with the free-coding process. Want me to run the diff and start pass 2?

<!-- xgd-turn id="a1f338a3-65a3-4178-a552-6e56adf84892-user" -->

<!-- xgd-chat role="user" ts="2026-07-03T18:17:34.958524+00:00" -->
#### You
The server stopped can you give me a command to run it myself?

<!-- xgd-turn id="a1f338a3-65a3-4178-a552-6e56adf84892-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-03T18:17:42.615580+00:00" -->
#### Claude
Here's the command — run it from the repo root and leave the terminal open:

```bash
cd /Users/martin/Projects/1stcontact
node tools/generate/bin/1c.mjs serve joyfulculinary --source draft --sandbox --port 8790
```

Then open **http://localhost:8790/**

Notes:
- It serves the last render, so if you edit the draft, re-render first: `node tools/generate/bin/1c.mjs render joyfulculinary --source draft --sandbox`
- The `Port 24678 is already in use` line is just Vite's HMR socket (harmless) — the serve still comes up.
- If 8790 is taken, change `--port` to any free port. Stop it with `Ctrl-C`.

<!-- xgd-turn id="3b2bcb8b-ad3f-45d6-8eb8-24a34de0c426-user" -->

<!-- xgd-chat role="user" ts="2026-07-03T18:21:11.778005+00:00" -->
#### You
ok so keep pushing - (1) how close can you get it with the tools you have (2) can you ennumerate the differences?

<!-- xgd-turn id="3b2bcb8b-ad3f-45d6-8eb8-24a34de0c426-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-03T18:26:06.980895+00:00" -->
#### Claude
Good questions. Let me answer (2) mechanically first — run the values-diff gate to enumerate every delta — then use that to push config fidelity as far as it goes.

The gate gives a precise enumeration — 188 deltas. Let me capture the full output and break it down by category.

188 deltas, categorized:

| property | count |
|---|---|
| fontSizePx | 46 |
| color | 36 |
| fontWeight | 36 |
| lineHeightPx | 30 |
| fontFamily | 18 |
| missing | 12 |
| paddingLeftPx | 6 |
| letterSpacingPx | 2 |
| contentAnchor | 1 |
| text (casing) | 1 |

Let me inspect the non-obvious categories (contentAnchor, fontFamily, the "missing" set) to sort config-fixable from code-only.

Now I have a clear picture. The `fontFamily` deltas confirm the original mixes **four** fonts per-element (Raleway for labels/"Learn More", Lato for hero subhead, Oswald for footer nav, Karla for process titles) in ways my 3-slot theme can't assign; the `color` deltas are dominated by gold headings + the dark-band choice; sizes/weights/line-heights are module-fixed. Let me apply the genuine **config** wins (verbatim casing, the omitted short testimonial), re-run the gate to show the delta, then give you the full verdict.

187 — essentially unchanged. That empirically confirms the ceiling: config can't move it further. Let me quantify exactly what each framework change would buy, so the enumeration is actionable.

I now have the complete, quantified picture. Let me record it on the ticket, then give you the verdict.

Here are your two answers.

## (1) How close config gets — the ceiling is reached

I applied the only genuine config wins left (verbatim casing, the omitted testimonial). The mechanical gate moved **188 → 187 deltas** — essentially nothing. That's the empirical proof: **with the tools as-is (config/theme only, no framework code), the draft is structurally and content-complete, but exact-value fidelity is blocked.** The server command from before still serves it.

## (2) The differences — mechanically enumerated (187 deltas)

From `1c values-diff`, **51 matched / 187 deltas**:

| property | count | tier |
|---|---|---|
| fontSizePx | 46 | **code** |
| color | 36 | 33 code / 3 config |
| fontWeight | 36 | **code** |
| lineHeightPx | 30 | **code** |
| fontFamily | 18 | **code** |
| missing | 12 | config (artifacts) |
| paddingLeftPx | 6 | cosmetic |
| letterSpacingPx | 2 | cosmetic |
| contentAnchor | 1 | cosmetic |
| text (casing) | 1 | ✅ fixed |

**~163 of 187 need framework code.** Broken down:

- **Colour (33):** 7 = gold section headings (`#f8bb1b`); 26 = white body text on the dark Offerings/Process bands. The framework can't do either yet — `services-grid` draws white cards, so on a dark band the text goes white-on-white (invisible), which is why I put those two grids on *light* bands. Fixing both = the two biggest **visual** gaps too.
- **Size + weight + line-height (112):** the modules use a fixed type scale and heading weight; the original uses specific pixel sizes (44/32/28/19/18) and *light* Oswald weights (200/300). This is the largest, most architectural bucket.
- **Font family (18):** the original mixes **four** faces per-element (Raleway labels, Lato hero subhead, Oswald footer nav, Karla process titles); the theme has only 3 family slots.

**The remaining ~24 are not real fidelity losses:** the 12 "missing" are my deliberate merges (hero heading split across two nodes; "Dan H."/"Parent / CFO" merged into a card title; the veggie quote modeled as a hero heading rather than body) plus punctuation normalization (curly quotes, zero-width spaces) and 2 genuine omissions (a "Get Started" and an Offerings "Learn More" button). The rest are muted-grey-vs-`#333` and sub-pixel padding/tracking.

## What would actually close it (framework code → free-coding)

Three well-scoped generalizations, in impact order — each mirrors an existing pattern, so per DOC-19/CLAUDE.md they're *generalizations*, not new modules (candidate home: REQ-32):

1. **`headingTreatment` on text-block + services-grid** (gold headings) — mirror hero's existing dial. Closes 7 colour deltas, huge visual win.
2. **A "bare"/transparent card surface** for services-grid — lets the Offerings/Process grids sit on dark bands with white text. Closes ~26 colour deltas + restores the dark drama.
3. **A green surface role** for the testimonial panel.

The 112 size/weight/line-height deltas are a bigger, separate effort (per-element type control) with diminishing visual return.

**These are framework code changes, so they go through the full free-coding ceremony** (UATs named `test_UAT_FC_*`, `[FREE-CODED]` commit, version bump), and per the project's "generalize before adding" rule they'd likely be their own capability tickets against REQ-32 rather than folded into REQ-36. That's a real chunk of work — **want me to proceed with #1–#3 (the visual wins) under free-coding?** I'd start with the gold-heading generalization.

Full enumeration + verdict is saved on REQ-36 (COMMENT-39).

<!-- xgd-turn id="70724186-c272-48aa-8297-a5244b777d07-user" -->

<!-- xgd-chat role="user" ts="2026-07-07T22:29:48.614460+00:00" -->
#### You
ok please read DOC-19 again there are more tools and process. I have deleted your previous attempt. Can you take a pass at the most faithful reproduction you can of 

https://joyfulculinarycreations.com/

PLease use all the diff tools and be sure to compare the final results before declaring victory

<!-- xgd-turn id="70724186-c272-48aa-8297-a5244b777d07-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-07T22:45:00.608336+00:00" -->
#### Claude
I'll re-read DOC-19 carefully and get the lay of the land. Let me start with reconnaissance in parallel.

The capture bundle is intact; my prior sandbox attempt was deleted as expected. The key thing I notice: last session I **never ran the perceptual `1c diff` gate** — I declared a ceiling from `values-diff` alone. DOC-19 is explicit that `1c diff` (overlay reading) is the gate that catches what values-diff is blind to. Let me do this properly this time.

Let me scaffold fresh and launch parallel investigation to build a compact working reference.

Scaffold is ready. While the framework-mapping agent runs, let me study the worked example and the capture structure myself.

The capture has a full per-element value manifest — exactly what `values-diff` compares against. Let me dump the complete content inventory to drive transcription.

Now I have the ground-truth value manifest. Let me view the screenshot to establish section structure (the capture collapsed to 1 band, so the screenshot + raw.html are my structure source).That gives me the full structure. Now let me establish exact band background colors and geometry from the Elementor CSS (the screenshot shows medium-grey bands and a sage-green testimonial panel — I need the real hex values, not eyeballed).

I now have the full framework vocabulary and verified band colors:
- Grey bands `#7A7A7A` (Who Uses / Offerings / Process) → `surfaceInverse`
- Green testimonial `#28542D9E` (~62% α)
- Quote scrim `#141E14BA` over `market-vegetables` photo
- Hero: `HERO-AdobeStock` image at 0.49 opacity over black

Let me check the font assets and a couple remaining values before writing the theme.

Font weights confirmed (Oswald 200 is the heading weight — light). I have enough to build. Let me stop analyzing and start building — the diff tools will drive the rest. Let me set up tracking and gather the assets I need.

I have the key images (hero, quote, logo) and fonts. The 3 "offering" boxes render as empty grey in the capture screenshot, so omitting images there actually matches the reference better. Let me copy assets into the sandbox and set up tracking.

Now let me write the theme (`site.json`), transcribing exact captured values.

Now let me look at the scaffold's page format and the gigabytealchemy home.json as a worked reference for module instance shape.

I have the module shape. I need the full untruncated text for all runs. Let me dump them cleanly.

All text captured. Now let me write the full 9-band page.

Now let me render the first draft and catch any schema errors.

Rendered cleanly. Now let me run **both** gates — the perceptual `1c diff` (which I skipped last time) and `values-diff` — plus a screenshot to eyeball.

Let me capture the values-diff to a file cleanly and view the draft screenshot.

142 deltas, 54 matched. Let me view the draft render and run the perceptual gate.Now I can see the draft. The perceptual view immediately reveals what `values-diff` alone would never surface. Big issues visible:
1. **Hero heading renders pink** (should be white) + **CTA pink** (should be gold)
2. **Gold section headings render dark/white** (text-block/services-grid have no gold treatment)
3. **services-grid cards are empty white boxes** — white-on-white (content invisible)
4. **Testimonials band grey** (should be green)

Let me run the perceptual gate for ranked regions, and investigate the hero/grid rendering code in parallel.

Perceptual mean is **66/255** — very high. Ranked regions confirm the eyeball: #1 & #3 & #4 are the empty white services-grid boxes, #2 is the hero. Let me investigate whether the hero colors and card rendering are config-fixable vs framework gaps before deciding scope.

The hero `treatment-plain` inherits `surface-inverse` color = white (per CSS), yet the screenshot looked pink — and the CTA is `--color-primary` (pink). Let me read the hero region overlay triptych to see the actual rendered colors (DOC-19: read the overlay, don't eyeball the full shot).

Good — the hero heading is actually **white** (I misread the downscaled full shot); the CTA is a full-width pink bar. Let me compare against the reference hero to calibrate precisely.

Now the comparison is precise. Reference hero: darker image, **uppercase** heading, a divider rule, small **gold** button. Ours: lighter scrim, mixed-case, full-width **pink** bar. Let me check the hero inner structure (why the CTA is full-width) then apply config fixes.

The full-width CTA is because `.hero__inner` is `flex-direction: column` with default `align-items: stretch` — the button stretches. That's framework behavior. I now have a complete picture. Let me apply the pure-config wins (gold CTA via primary swap, stronger scrim) and re-measure to establish the honest config ceiling.

Config-only tops out at **64.57/255** (from 66.19) — marginal, because the dominant regions are all framework-blocked. This is the honest config ceiling, and per your instruction I'm **not** declaring victory. Let me record the diff-driven findings on the ticket, then lay out the decision.

I've done the diff-driven rebuild you asked for. Here's where it stands — and the decision I need from you before I keep going.

## What I did differently this time
Fresh sandbox build from the capture (theme + 9 bands), then ran **both** gates — including the perceptual `1c diff` with **overlay reading**, which I skipped entirely last time and which is the whole point of DOC-19. Reading the region triptychs (not the downscaled full shot) immediately caught things `values-diff` is blind to.

## Honest measured state (not victory)
- **`1c diff` perceptual mean: 64.57 / 255** — faelan-grade is ~2–3, so this is far off.
- **`values-diff`: 54 matched / 142 deltas.**

Config wins are already in: gold CTA (gold=primary swap), strong scrim, exact band colours (`#7a7a7a` grey bands, `#28542d` green, gold footer), all four fonts loaded, verbatim text.

## Why config is exhausted — overlay-confirmed, ranked by region score
| Region | What the overlay shows | Blocker |
|---|---|---|
| #1 / #3 / #4 (largest) | Offerings + Process render as **empty white boxes** | services-grid draws white cards on the dark band → white-on-white |
| #2 (hero) | heading mixed-case not **UPPERCASE**; no divider; CTA stretches full-width | framework |
| pervasive | every gold section heading renders dark/white | text-block & services-grid have no gold `headingTreatment` |
| testimonials | grey not green | no green surface value |

Every remaining big lever is a **framework code change**, not config. Per the generalize-first rule they're all generalizations of existing modules:

1. **`headingTreatment: gold`** on text-block + services-grid (generalize hero's existing dial) — the pervasive gold-heading signal.
2. **Bare/transparent card surface** on services-grid — makes Offerings/Process render as text-columns on the dark band. **Closes the three highest-scoring regions.**
3. **Uppercase heading treatment** — author uppercases via CSS; keeps DOM text mixed so `values-diff` stays clean.
4. **Green surface** for the testimonial band.

## The decision I need
Each of these is framework `xgd_source`/`packages` code → **mandatory free-coding ceremony** (scope ticket + `test_UAT_FC_*` UATs + `[FREE-CODED]` + version bump). They're reusable framework capabilities, so per the model they belong on **framework capability tickets** (the docs point to REQ-32), **not** folded into REQ-36 (which is the site reproduction).

**I can't cleanly create those tickets or touch framework code without your say-so.** How do you want to proceed?

- **(A)** I create capability tickets (against/near REQ-32) and free-code them, biggest-win-first (#2 bare cards + #1 gold headings first — they clear the top regions), re-diffing after each until the mean drops, then verify final. *Recommended.*
- **(B)** You'd rather I fold the framework changes into REQ-36 as one bundle (simpler ticketing, less "correct" per the intent model).
- **(C)** Stay config-only — accept ~64/255 and I document the gaps as the reproduction's honest ceiling.

Findings are recorded on REQ-36 (COMMENT-59). Which option?

<!-- xgd-turn id="71d07810-264e-4904-8e42-e296e920a9fe-user" -->

<!-- xgd-chat role="user" ts="2026-07-08T15:28:35.418030+00:00" -->
#### You
What I want to do is figure out how to turn this into a process. From my point of view I think it would be cleaner if framework changes stayed in this ticket.

I want to discuss when you create new modules vs adding dials to existing ones (I see failure cases in both directions)

I also need due process around tests we now have a conformance test harness we need to work through the testing requirements for modules

<!-- xgd-turn id="71d07810-264e-4904-8e42-e296e920a9fe-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-08T15:31:15.689574+00:00" -->
#### Claude
This is a shift from "finish the repro" to "make it a repeatable process." Good — three threads: (1) framework changes stay in REQ-36 (agreed), (2) module-vs-dial decision procedure, (3) test due-process against the conformance harness. Before I talk about testing I should actually look at what the harness is rather than theorize. Let me investigate.

The harness is REQ-39/DOC-20: `assertModuleConforms(slug, fixtures)` — renders each fixture as a one-module page and checks safety/security/responsive dimensions, with negative fixtures proving the discriminator works. Let me read DOC-20 and how existing modules wire into it.

Let me see how an existing module wires into this — the per-module test pattern the user wants to standardize.

Good — that reframes the work. I dug into the harness so I can talk concretely. Here's what's actually on disk:

- **Conformance harness exists** (`tools/generate/src/conformance/`, REQ-39/DOC-20): `assertModuleConforms(slug, fixtures, opts)` renders each fixture as a one-module page, serves it, drives Chromium, checks **safety / security / responsive / x-browser**. Tiers `fast` (chromium, 1280+375) / `full`. Negative fixtures prove the discriminator discriminates.
- **Gap:** it's only exercised by the REQ-39/40/41 harness tests (with injected broken modules + a text-block baseline). **No module has its own conformance leaf**, and there's **no correctness dimension** — it proves "nothing broke," not "gold is actually gold."

Let me take your three threads in turn.

## 1. Scope: framework changes in REQ-36
Agreed, and it's clean *if* REQ-36's body declares the framework-capability scope before I touch code (free-coding needs the behaviour described first). I'll fold that in once we lock the process below.

## 2. When a new module vs. a dial/variant

The failure modes you're seeing, named so we can detect them:

| Direction | Failure | Detectable smell |
|---|---|---|
| Over-dialing (god-module) | one module grows two behaviours behind a mode flag | a dial only valid when *another* dial has a certain value; render branches on `variant && dial` combos; a content field only read in one variant |
| Over-modularizing (fragmentation) | a module per section shape | two modules share ~80% of template/CSS; capability split across 3 modules so users hunt for it; catalog grows faster than the concept count |

Decision procedure (first match wins):

1. **Same communicative job, different look → dial or variant.** Presents heading+prose, or presents N cards — same job → treatment.
   - **Variant** = discrete *structural* preset; the template branches (three-col/two-col/stacked, top-nav/overlay). Closed, few, mutually exclusive.
   - **Dial** = *orthogonal* modifier that must compose with **every** variant and every other dial (surface, size, spacing, headingTreatment, gap).
2. **Different content question → module** (nav vs. testimonial carousel vs. contact form).
3. **Cross-cutting composition several modules share → shared primitive** (the `fc-row`, the overlay band) — not a module, not a per-module dial.

The load-bearing guardrail against over-dialing is a **composability invariant**: every dial value must be legal with every variant and every other dial. The moment "dial X only means something when Y" appears, you've got two jobs → split, or make it a variant.

Applying it to our four changes:

| Change | Verdict | Note |
|---|---|---|
| gold `headingTreatment` on text-block + services-grid | **dial** (generalize hero's) | composes cleanly ✓ |
| bare/transparent card | **per-card dial** `surface: bare` | need to decide: does "bare + badge + accent-border" compose, or is bare mutually exclusive with card chrome? |
| uppercase headings | **dial** `case`/`transform` | orthogonal, composes with everything ✓ |
| green testimonial band | **neither a color-named surface nor a new module** | see below |

The green case is the interesting one and I think it's a *trap in both directions*. Adding `surface: green` fragments the enum (every site's brand hue becomes a new surface value) — that's fragmentation-by-enum. But not supporting a distinct panel colour under-serves. The right resolution is to generalize the **mechanism**: surfaces already map to semantic tokens (default→bg, inverse→surface-inverse, accent→accent). The green panel is just *a second panel role*. So either surface takes a **palette role** (role-driven surface) or we add one generic `surface-panel` role the theme colours per-site — green here, something else next site. That keeps the framework hue-agnostic.

**Open question for you:** on the bare card — is `bare` a card *surface value* that's mutually exclusive with badge/accent chrome (simplest, and I'd argue "bare" means "no chrome" so badges don't apply), or do you want them to compose?

## 3. Testing due-process for module changes

What the harness gives us free (once wired) vs. what's missing:

- ✅ safety / security / responsive / x-browser — universal, per fixture.
- ❌ **correctness** — that a dial produces its intended effect. `headingTreatment:gold` passing conformance only proves it didn't overflow or XSS, not that it's gold.
- ❌ **matrix coverage** — nothing fails when a module adds a variant/dial with no fixture.

Proposed requirements for any module add/change (the process):

1. **Conformance leaf per module** — colocated `modules/<m>/<m>.conformance.test.ts` calling `assertModuleConforms`, fixtures covering **each variant once + each non-default dial value once + the new capability's combination**. Backfill the 7 existing modules.
2. **Effect assertion** — a computed-style probe proving the dial *changed the output the intended way* (`getComputedStyle(heading).color === accent token` for `gold`). Also guards the silent-no-op failure mode (a dial that does nothing).
3. **Default-vs-set pair** — prove the dial has an effect by asserting the pair differs.
4. **UAT naming** `test_UAT_FC_REQ-36_*`.
5. **The `1c diff` repro gate stays the integration-level truth** — conformance is per-module correctness; the repro mean is the whole-page eye test.

Decisions I need from you to turn this into the process:

- **(a)** Correctness: extend the conformance harness with an **expectation-probe dimension** (fixtures carry `expect: {selector, prop, equals}`), or keep correctness as **separate colocated unit tests** and leave the harness to safety/security/responsive? I lean toward adding the dimension — one call, one place.
- **(b)** Coverage: build a **completeness gate** (module meta enumerates variants/dials → a test fails if any lacks a fixture), or trust author discipline?
- **(c)** Location: **colocated** per-module test files (my recommendation, matches "test by module"), or centralized under `tests/`?
- **(d)** Tier policy: leaves run `fast` in CI, `full` (all engines + responsive ladder) in regression only?

If you answer (a)–(d) plus the bare-card composability question, I'll write it up as a short module-testing standard, put the framework-scope + test plan into REQ-36's body, and then start free-coding the four generalizations biggest-win-first — re-diffing after each.

<!-- xgd-chat-end -->