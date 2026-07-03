---
uid: comment-57c30ff9
id: COMMENT-21
type: comment
title: Comment on request REQ-28
created_by: xgd
created_at: '2026-07-02T22:40:53.601201+00:00'
updated_at: '2026-07-02T22:42:18.409155+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-150b5ebf
  kind: chat_transcript
---

<!-- xgd-turn id="65b01a58-1dd3-4394-a727-0bc31f2561ce-user" -->

<!-- xgd-chat role="user" ts="2026-07-02T22:40:51.531382+00:00" -->
#### You
Please free code this ticket

<!-- xgd-turn id="65b01a58-1dd3-4394-a727-0bc31f2561ce-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-02T22:42:18.382386+00:00" -->
#### Claude
I'll start by investigating the current state — the ticket body, the modified files already in the working tree, and the test that exists. It looks like prior work may have been done in a previous session.

There's substantial uncommitted work spanning multiple modules. This is broader than REQ-28 alone (hero) — it also touches `header` (REQ-24 territory), `services-grid`, and `dials.ts`. Let me understand exactly what was done before deciding how to proceed.

The working tree mixes changes from **four different tickets** (REQ-24, REQ-28, REQ-29, REQ-30 per the code comments). This session is scoped only to **REQ-28** (hero `headingTreatment`). I need to see the test file and the site JSON, and check the sibling tickets' statuses before deciding what belongs in this commit.

I now have the full picture. Here's what a prior session left in the working tree, and it raises a structural decision I want your call on before I commit anything.

## What's in the working tree

A prior session implemented **three independent module capabilities**, each of which has its **own pre-existing ticket**:

| Change | Files | Own ticket |
|---|---|---|
| `hero` `headingTreatment` dial (plain/accent/gold) | `dials.ts`, `hero/meta.ts`, `hero/index.astro` | **REQ-28** (this session) |
| `header` `align` dial (left/center) | `header/meta.ts`, `header/index.astro` | **REQ-29** (exists, draft) |
| `services-grid` `stacked` variant | `services-grid/meta.ts`, `services-grid/index.astro` | **REQ-30** (exists, draft) |

Plus `tests/req28-module-dials.test.ts` (one file, UATs for all three — named `test_UAT_FC_REQ-28_*`) and edits to `gigabytealchemy/draft/pages/home.json`.

The prior session **folded all three tickets into REQ-28** — the test docstring literally says *"consolidated into one ticket (subsumes REQ-29, REQ-30)."*

## The problem with that

- This session is scoped to REQ-28 only.
- REQ-29 and REQ-30 are real, separate `draft` tickets. Committing all three under REQ-28 and marking only REQ-28 `free_coded` **orphans** REQ-29/REQ-30 — when reconcile later runs on REQ-28 it will mint ACs for all three behaviours under REQ-28, leaving REQ-29/REQ-30 as stale duplicate tickets.
- One-intent-per-ticket cuts both ways: three tickets shouldn't collapse into one commit any more than one ticket should splinter into three.

There's also **noise** in `home.json`: only 3 lines are meaningful (`align: center`, `headingTreatment: gold`, `variant: stacked`), but the file got re-serialized — every `—` became `\u2014` and every inline dial object was expanded multi-line. That churn is unrelated to any ticket and I'd strip it.

## My recommendation

**Split into three free-coded commits, one per ticket** (each is ~1–2 story points):
- REQ-28 ← hero heading treatment (+ its home.json line, + its UATs)
- REQ-29 ← header align (+ its line, + its UATs)
- REQ-30 ← services-grid stacked (+ its line, + its UATs)

I'd split the test file to match (`test_UAT_FC_REQ-29_*`, `test_UAT_FC_REQ-30_*`) and drop the JSON serialization noise. Slightly more work now, but each ticket reconciles cleanly to its own capability.

The alternative is to honour the prior session's consolidation (everything under REQ-28) and then **abandon REQ-29/REQ-30** as superseded — faster, but it discards two tickets you deliberately filed as separate intents.

Which do you want — **split across the three tickets** (my recommendation), or **consolidate under REQ-28** and abandon 29/30?

<!-- xgd-chat-end -->