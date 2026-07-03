---
uid: comment-fb712e1a
id: COMMENT-34
type: comment
title: Comment on request REQ-33
created_by: xgd
created_at: '2026-07-03T16:50:58.807427+00:00'
updated_at: '2026-07-03T16:50:58.807427+00:00'
completed_at: null
last_field_updated: created_at
status: null
fields:
  subject_uid: request-f3342338
  kind: note
---

## Cherry-pick scope (framework fidelity fixes)

Rather than run the full 86-delta re-import loop, porting the clean, universal framework-code corrections surfaced by the sandbox values-diff. These improve fidelity for every imported site, not just gigabytealchemy (per CLAUDE.md: generalize, no new modules):

1. markdown.ts — disable smartypants (createMarkdownProcessor({ smartypants: false })). Authored straight quotes/dashes were being silently curled, so text no longer matched the reference -> ~11 body [missing] deltas. Verbatim rendering is correct for a faithful-repro engine.
2. contact-form submit button — add 'font: inherit'; the button fell back to UA Arial 13px instead of the site font/size ([fontFamily] Arial->ui-sans-serif + size deltas).
3. services-grid checklist tick — render the check glyph as a real leading text run instead of a ::before pseudo-element, so it exists in the DOM/a11y tree and is seen by values-diff (6 [missing] check deltas).

Deliberately excluded (per-site tuning, belongs in site-def/theme via the REQ-33 loop, NOT framework defaults): card-title/badge font sizes+weights, subhead muted-vs-black colour, hero title gradient/anchor, line-heights, footer copyright ordering + 2025-vs-2026 build year.

Framework-code changes -> full free-coding ceremony (UAT test_UAT_FC_REQ-33_*, [FREE-CODED], version bump).