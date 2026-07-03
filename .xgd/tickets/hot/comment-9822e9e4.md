---
uid: comment-9822e9e4
id: COMMENT-20
type: comment
title: Comment on request REQ-27
created_by: xgd
created_at: '2026-07-02T22:32:59.723385+00:00'
updated_at: '2026-07-02T22:38:00.451400+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-d06126ef
  kind: chat_transcript
---

<!-- xgd-turn id="e24ce96b-1021-4784-b9a6-4504c766fefe-user" -->

<!-- xgd-chat role="user" ts="2026-07-02T22:32:53.176Z" -->
#### You
Please free code this ticket

<!-- xgd-turn id="e24ce96b-1021-4784-b9a6-4504c766fefe-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-02T22:38:00.422Z" -->
#### Claude
I'll start by reading the existing ticket and investigating the relevant code before making any changes.

Let me examine the key files for surface and background handling.

Now let me see how the `surface` dial is applied in module CSS. Let me look at the hero module and how surface classes work.

The issue is clear now. The module's `surface-*` class sets an opaque `background` on the module's own `<section>`, which paints over the background layer that `wrapWithBackground` places behind it. Let me understand the render pipeline to find the right fix location.

Let me find where `wrapWithBackground` is called and how surface + background interact in the pipeline.

Now I fully understand the bug and the render pipeline. Let me look at the existing tests for background rendering to understand the test patterns.

Let me check how tests are run and confirm the vitest setup.

Let me run the existing background test to confirm the setup works before changing anything.

The setup works and my analysis is complete. Let me explain the root cause and fix before implementing.

**Root cause:** `wrapWithBackground` stacks a background layer (z-index 0) behind the module's content (z-index 2). But each module's `surface-*` class sets an **opaque `background`** on the module's own `<section>` (e.g. `.hero.surface-inverse { background: var(--color-surface-inverse); color: var(--color-bg) }`). That opaque fill paints over the background layer, so the REQ-14 background becomes inert. The `surface` dial conflates two things — a **fill** and a **text-color contract** — and only the text-color contract should survive when a background is present.

**Fix (structural precedence rule, no per-module edits, no `!important`):** add one rule to `SECTION_CSS` that neutralizes the wrapped band's own `background-color`/`background-image` while leaving `color` (the contrast contract) intact. The selector `.fc-bg-section > .fc-bg-section__content > *` has specificity (0,2,0), tying `.hero.surface-inverse`, and since `SECTION_CSS` is concatenated *after* module CSS in `theme.css`, it wins the cascade. Modules with no background wrapper are untouched — surface fills paint normally.

Let me implement it.

Now let me write the UAT test. Let me add it to a new FC-named test file following the established patterns.

The assertion is over-eager — `\bcolor:` matches inside `background-color:`. Let me fix the regex to match only a standalone `color:` property.

All passing. Now let me run the broader test suite as a regression scope, plus a typecheck on the framework.

Full suite green (133 passed). Let me typecheck the framework package to be safe, then bump the version.

Typecheck clean. Now let me bump the version and commit.

Version bumped `package.json` to 0.0.19. Now let me stage only the fix files (excluding the unrelated screenshot artifact and ticket internals) and commit.

Committed. The remaining untracked items (ticket search index, an unrelated screenshot) are not part of this fix. Now let me move the ticket to `free_coded` via the gate command and record story points.

Now let me update the ticket body — preserving the existing content and adding the implementation and test plan sections.

Let me verify the final ticket state.

Done. REQ-27 is free-coded and parked at the stable resting state.

## Summary

**The bug:** When a section had both a REQ-14 `background` and a `surface` dial, the surface fill painted over the background, making it inert. Root cause: `wrapWithBackground` stacks the background layer behind the module's content, but each module's `surface-*` class sets an **opaque `background`** on the module's own `<section>`, which covers the background layer. The `surface` dial conflates a *fill* with a *text-color contract* — only the contract should survive when a background is present.

**The fix** (`packages/framework/src/modules/background.ts`, one rule added to `SECTION_CSS`):
```css
.fc-bg-section > .fc-bg-section__content > * {
  background-color: transparent;
  background-image: none;
}
```
When a background wrapper is present, the wrapped band's own fill is suppressed (background paints) while `color` is left intact (surface still supplies the contrast contract). The two-class selector ties `.<module>.surface-*` at specificity (0,2,0); `SECTION_CSS` is emitted after module CSS, so it wins the tie deterministically. No `!important`, no per-module edits, no raw CSS in site definitions. Surface-only bands (no background wrapper) are untouched.

**Documented precedence rule:** *background paints; surface contracts.*

**Evidence:** 3 UATs in `tests/req27-background-surface-compose.test.ts` (including an end-to-end `1c render` of the gigabytealchemy hero case — image background + `surface: inverse` coexisting). Full suite green: **133 tests passed**, REQ-14 UATs unchanged; framework typecheck clean.

**Ticket state:** REQ-27 → `status=free_coded`, `commits=[b2cef07…]`, `version=0.0.19`, `story_points=3`. Body updated with root cause, fix, and test plan (original scope/ACs preserved).

This is parked at `free_coded` for you to validate. When you're satisfied, promote it to `ready_to_reconcile` to let the dispatcher fold it into the capability matrix — that step is yours to trigger.

One note: the fix addresses the `color` text-contract from `surface`, but a `surface` value also implies a *fill tone* that's now intentionally dropped when a background is set. That's the correct precedence per the AC (background paints), but if you ever want a surface tone to act as a *tint over* a background image, that would be a distinct composition (closer to the overlay layer) — out of scope here, worth a separate ticket if it comes up.

<!-- xgd-chat-end -->