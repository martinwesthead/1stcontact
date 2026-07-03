---
uid: comment-e0980976
id: COMMENT-15
type: comment
title: Comment on request REQ-16
created_by: xgd
created_at: '2026-07-02T16:36:44.002775+00:00'
updated_at: '2026-07-02T16:51:08.792887+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-727a9613
  kind: chat_transcript
---

<!-- xgd-turn id="f3a151ce-1271-4561-9994-acaf7b380176-user" -->

<!-- xgd-chat role="user" ts="2026-07-02T16:36:37.226Z" -->
#### You
Please free code this ticket

<!-- xgd-turn id="f3a151ce-1271-4561-9994-acaf7b380176-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-02T16:51:08.767Z" -->
#### Claude
I'll start by reading the current ticket to understand the full scope before touching any code.

This is the 1stcontact TS/pnpm monorepo (not the xgd Python codebase). Let me explore the framework and schema packages to understand existing patterns before designing the motion primitive.

Let me look at the key existing patterns — the schema types, framework module types/styles, and the recent REQ-15 layer work as a reference.

Now let me look at the schema.ts (where Motion schema goes), the layer schema/rendering as the closest reference, and how styles/dials render CSS.

Now let me look at the layer rendering (REQ-15, closest reference) and how the framework generates CSS from structured props, plus the REQ-15 test for the test pattern.

Let me see how the wrappers are wired into rendering, where CSS is aggregated, and the index exports.

This is the key integration point. Let me read the render.ts generator and the styles.ts aggregator.

Let me understand how client-side JS (islands) are delivered, since the ticket requires a scroll-trigger island. Let me check the contact-form enhance pattern and how scripts reach the page.

Let me check the test patterns — the contact-form client test (JSDOM), the vitest config, and the existing REQ-15 layer test — so my UATs match house style.

I have a thorough understanding of the codebase patterns now. Let me check the two barrel files where I'll wire exports, plus the site-schema index.

I now have a complete picture. Let me lay out my plan and confirm the design decisions before implementing.

The ticket scope is clear and the body is populated, so I'll proceed. Here's my implementation approach:

**Design decisions:**
- **`Motion` schema** — a `.strict()` object: `type` (`fade`/`slide`/`scale`/`stagger`), `trigger` (`load`/`scroll`/`hover`), optional `duration` (ms int), `easing` (named enum — structured, not raw CSS), `delay` (ms int). Attached to module instances **and** layer children (ticket: "modules/children").
- **Rendering** — a framework `motion.ts` with static `MOTION_CSS` (keyframes + transitions + `prefers-reduced-motion` block), `wrapWithMotion()` emitting framework-computed classes + CSS custom props (never raw instance CSS), and a self-contained `MOTION_SCRIPT` IntersectionObserver island injected only when a page has scroll-triggered motion.
- **Wiring** — motion wraps outermost (whole section reveals) in `render.ts`; layer children get their own motion in `layer.ts`.

Let me set up tracking and start implementing.

Clean slate. Let me implement. First the schema — add motion schemas and attach the field.

Now the derived types:

Now the core framework `motion.ts`:

Now apply motion to layer children. To avoid the slide/scale `transform` colliding with the child's positioning `transform: rotate(...)`, I'll wrap the child's **inner** content, not the positioned element itself.

Now wire the barrel exports (framework):

Now wire into the render pipeline. Import, wrap modules with motion, add `MOTION_CSS` to the stylesheet, and inject the island when a page has scroll motion.

Now inject the island into the page body and add `MOTION_CSS` to the stylesheet:

Now the UAT test file. Let me write the four UATs covering the schema, rendering, scroll island (executing the real shipped script in JSDOM), reduced-motion, and params-drive-output end-to-end.

Now let me run the new test file plus the regression scope (schema, layer, generate tests):

The jsdom environment breaks esbuild (used by the Astro container in the CLI render). The house pattern (contact-form) isolates DOM tests in a separate jsdom file importing the leaf module directly. Let me split: a node-env file for schema/render/CSS/CLI, and a jsdom file for the island — importing `motion.ts` directly to avoid pulling Astro.

Now remove the island describe block from the node file:

Now the jsdom island file, importing `motion.ts` directly (no Astro/esbuild):

Two fixes needed. First, the island degrade check: `vi.stubGlobal('IntersectionObserver', undefined)` leaves the key `in window` (true), so the script enters the observer branch and throws. A `typeof` check is more robust. Let me also investigate the `cmdNew` scaffold path.

REQ-15 uses the identical call and passes. Let me look at `cmdNew` and run my node file alone to see the real failure.

The schema/CSS UATs pass; only the CLI render ones fail — yet REQ-15 uses the identical pattern. Let me check `ctxOf`'s default root and trace where `cmdNew` actually writes.

The store uses `storage/<root>/`. But REQ-15 reads `sites/...` and passes — so `siteDir`/`draftDir` must differ from `listSlugs`. Let me read paths.ts.

I misread — the current path includes `storage/`. Let me fix both page paths in my node test.

Now fix the island's environment-detection to use `typeof` so the `undefined` stub degrades correctly:

The CLI loads via the **built** `dist/` of `@1stcontact/site-schema`, which predates my schema change. I need to rebuild the packages. Let me check build scripts and rebuild.

The CLI resolves packages via their built `dist/`. My schema/framework changes are in `src/` — I need to rebuild. Let me check for a root build and rebuild the affected packages.

Framework resolves from `src/` directly; only site-schema needed a build, which succeeded. Rerun:

One occurrence is the wrapper's attribute; the second is inside the injected `<script>` (the selector string `[data-fc-motion-scroll]`). I'll assert the `<script>` tag ships exactly once instead:

All 6 UATs pass. Now the regression scope — run the full suite plus typecheck to ensure I didn't break adjacent behavior (schema, layer, generate, site-schema):

Full suite green (102 tests). Now typecheck the changed packages plus the naming guardrail:

Both clean. Let me typecheck `tools/generate` (I changed render.ts) and run lint if configured.

All checks green. Now let me follow the free-coding lifecycle: bump version, review the diff, and commit. First, bump and check what changed (including the rebuilt `dist/`).

The working tree contains far more than my changes — a `sites/ → storage/sites/` migration plus edits to `commands.ts`, `paths.ts`, `bundle.ts`, `.gitignore`, and several test files that I never touched. Notably, `req15-layer.test.ts` *changed under me mid-session* (it read `sites/...` early, `storage/sites/...` later). This looks like a concurrent process (dispatcher resync / another workflow) mutating `xgd-working`. I must not sweep those into my commit. Let me investigate before committing.

Active `xgd_source.core.dispatcher.runner` processes are running, `.xgd/tmp/resync_state.json` exists, and `req15-layer.test.ts` was rewritten mid-session — a concurrent process (the `sites/`→`storage/` migration arriving on `xgd-working`) is mutating the tree. I must commit **only** REQ-16 files, not the staged migration. Let me confirm exactly which changes are mine vs. foreign, and the index state.

Confirmed my scope precisely:
- **Mine** (10 files): the two site-schema `src` files, four framework `src` files, `render.ts`, `package.json` (version bump only), two test files. `dist/` is gitignored (rebuilt by CI/reconcile).
- **Foreign** (concurrent `storage/` migration — must exclude): `.gitignore`, all `sites/→storage/` renames, `commands.ts`, `paths.ts`, `bundle.ts`, and edits to `generate/req11/req14/req15/shot` test files.

The migration's renames are already **staged** in the index, so a plain `git commit` would sweep them in. I'll commit only my paths with a pathspec (`--only` semantics), leaving the staged migration untouched.

zsh doesn't word-split unquoted `$PATHS`, and `-m` landed after `--` (treated as pathspec). Let me stage my new files, write the message to a file, and commit with the message before the pathspec.

Committed 10 files only. Let me verify the commit is clean (no foreign files) and capture the SHA:

The commit is clean — exactly my 10 files, no foreign migration content, and the version-bump gate check passes. SHA: `60b2a71215d99b5a2daaeef119b1ab0186bed4ca`.

Now move the ticket to `free_coded` via the gate command, then set story points:

Now update the ticket body to reflect the implementation as built, preserving the original scope content and adding the design decisions and final UAT set:

Let me verify the final ticket state and confirm the foreign migration changes remain untouched in the working tree:

Ticket is `free_coded` with my SHA and 5 points. HEAD is now the ticketing API's auto-commit — let me confirm it only touched ticket files and did **not** sweep in the staged migration renames:

Done. REQ-16 is free-coded and the ticket is in a clean resting state.

## Summary

**Implemented** the structured motion primitive (entrance / scroll-reveal / hover) end-to-end:

- **Schema** (`site-schema`) — `motionSchema` (`.strict()`): `type` (fade/slide/scale/stagger), `trigger` (load/scroll/hover), ms `duration`/`delay`, and a **named-enum** `easing` (raw `cubic-bezier(...)` is rejected — holds the structured-only line). Attached to module instances **and** layer children; derived types exported.
- **Framework** (`motion.ts`) — `MOTION_CSS` (keyframes, hover transitions, stagger nth-child cascade, `prefers-reduced-motion` that forces scroll content visible), a self-contained `MOTION_SCRIPT` IntersectionObserver island, and `wrapWithMotion`/`motionClasses`/`motionVars` emitting framework-computed `--fc-motion-*` props only.
- **Wiring** — motion wraps outermost in `render.ts` (over REQ-14 bg + REQ-15 layer); `MOTION_CSS` folded into `theme.css`; island injected once per page only when scroll motion is present. Layer-child motion wraps the child's *inner* content to avoid clobbering its positioning `transform`.

**Tests** — 5 UATs across two files (node-env for schema/CSS/CLI-render; jsdom for the shipped island against a stubbed IntersectionObserver). All 5 pass; **full suite 102/102 green**; schema/framework/generate typecheck clean.

**Free-coding lifecycle complete:**
- Commit `60b2a71` — `[FREE-CODED]`, 10 files only, version bumped 0.0.12 → 0.0.13.
- Ticket REQ-16 → `status=free_coded`, `commits=[60b2a71…]`, `story_points=5`. Body updated to the as-built design. Parked here — promotion to `ready_to_reconcile` is your call.

## ⚠️ One thing you should know

Your working tree contains a **large, unrelated `sites/ → storage/` migration** that is *not* part of REQ-16 — 16 staged file renames plus edits to `.gitignore`, `commands.ts`, `paths.ts`, `capture/bundle.ts`, and several test files (`generate`, `req11/14/15`, `shot`, plus a new `req22-storage.test.ts`). It appeared **mid-session** (`req15-layer.test.ts` rewrote itself from `sites/…` to `storage/sites/…` under me), and there are active `xgd_source.core.dispatcher.runner` processes plus a `resync_state.json` — so a workflow/resync brought it onto `xgd-working` while I worked.

I deliberately **excluded all of it** from my commit (used a scoped pathspec commit so the staged renames were never swept in) — my commit is exactly the 10 REQ-16 files. But that migration is now sitting uncommitted/partly-staged in your tree and belongs to a different intent. You'll want to decide how it gets committed/reconciled separately; it's untouched and intact.

<!-- xgd-chat-end -->