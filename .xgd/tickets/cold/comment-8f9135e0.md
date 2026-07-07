---
uid: comment-8f9135e0
id: COMMENT-41
type: comment
title: Comment on request REQ-37
created_by: xgd
created_at: '2026-07-03T18:29:56.655206+00:00'
updated_at: '2026-07-03T19:20:14.164108+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: request-8ccd3a3e
  kind: chat_transcript
---

<!-- xgd-turn id="c15fac04-e4db-4273-a865-b96e14f9b14c-user" -->

<!-- xgd-chat role="user" ts="2026-07-03T18:29:54.884025+00:00" -->
#### You
The current execution of 1c commands is cumbersome for me and you. Can you create a mechanism to make it easy - one approach would be a short shell script in ./bin 1c that took the arguments and ran the node command - open to alternative designs that mean I can just run 1c commands ...

<!-- xgd-turn id="c15fac04-e4db-4273-a865-b96e14f9b14c-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-03T18:31:40.014533+00:00" -->
#### Claude
I'll investigate how `1c` is currently invoked so I can design the right launcher.

Let me look at how `1c` is currently invoked and the tools/generate entrypoint.

Now let me check how it's currently invoked and whether a `bin/1c` already exists or is on PATH.

Current invocation is `node tools/generate/bin/1c.mjs <args>` — works but verbose (and no `node_modules/.bin/1c` symlink exists). A `bin/1c` launcher is the right fix.

Note on process: a developer-convenience launcher in `bin/` that isn't shipped with any package is explicitly **exempt** from the free-coding ticket flow (FREE-CODING.md: "Disposable shell scripts in `bin/` that aren't shipped → no ticket required"). It's pure plumbing — no `xgd_source/`/`tests/` files, no capability-matrix impact. So I'll just create it. REQ-37 can stay blank or be repurposed for something else.

Key design constraint from the mjs itself: it roots Vite at the repo but resolves **data/dist paths from the caller's CWD**. So the wrapper must preserve CWD (no `cd`) while resolving the repo root from its own location — letting you run `1c` from anywhere.

Done. Created `bin/1c` — a thin launcher that dispatches to `tools/generate/bin/1c.mjs`.

**Verified:**
- From repo root: `./bin/1c list` → lists your sites (`1stcontact`, `gigabytealchemy`, `harbor-cafe`).
- From a subdirectory: CWD is preserved (running in `storage/` correctly resolved `sites/` relative to that CWD → "(no sites)"), confirming it behaves identically to invoking node directly.

**How to use it:**
- From the repo root: `./bin/1c render <slug>`, `./bin/1c publish <slug> -m "..."`, etc.
- To type bare `1c` from anywhere, add to PATH or alias:
  ```bash
  export PATH="/Users/martin/Projects/1stcontact/bin:$PATH"
  # or
  alias 1c="/Users/martin/Projects/1stcontact/bin/1c"
  ```
  Run it from the repo root, since the CLI resolves `sites/`, `sandbox/`, and `dist/` relative to your current directory.

**Why a shell wrapper over the alternatives:**
- **vs. `node_modules/.bin/1c`** (pnpm bin symlink) — that symlink isn't being hoisted, and it'd break `1c`'s CWD-relative path resolution when invoked from arbitrary directories. The wrapper is deterministic.
- **vs. a root `package.json` script** (`pnpm 1c ...`) — pnpm mangles arg passing and adds noise.

**On process:** no scope ticket — a non-shipped dev launcher in `bin/` is explicitly exempt from the free-coding flow. REQ-37 (this session's blank ticket) is untouched and free for whatever you actually want to capture in it.

Two follow-ups if you want them:
1. **Commit it?** I haven't committed — say the word and I'll commit `bin/1c` to `xgd-working` (no version bump / `[FREE-CODED]` tag needed given the exemption).
2. **Suppress the startup noise** (`WebSocket server error: Port 24678 in use`, `Missing pages directory`)? That comes from the Vite SSR server inside the mjs. I left it visible so real errors aren't masked, but I can quiet those two specific lines in the wrapper if they bother you.

<!-- xgd-turn id="77041857-a37c-457b-be7b-98d944cc8e84-user" -->

<!-- xgd-chat role="user" ts="2026-07-03T19:15:03.716569+00:00" -->
#### You
please commit and update the ticket - it would be good to fix the ERROR

<!-- xgd-turn id="77041857-a37c-457b-be7b-98d944cc8e84-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-03T19:20:14.135467+00:00" -->
#### Claude
Let me read the ticket first (to preserve its body) and investigate the ERROR before touching code.

`1c serve` (a long-running preview) holds the HMR port 24678, so every other `1c` invocation collides. `hmr: false` in the mjs isn't actually suppressing the HMR WebSocket. Let me confirm why by dumping the resolved config.

`server.hmr` resolves to `false`, yet the WebSocket server is still created and binds the fixed port 24678 — so `hmr: false` isn't enough in this Vite/Astro version. Let me find what forces the ws open and test candidate fixes.

Found the root cause. In **Vite 8**, `createWebSocketServer` only returns a no-op stub when `server.ws === false` — the old `hmr: false` flag no longer suppresses the WebSocket server (node.js:16585). So the CLI keeps binding the fixed HMR port 24678, colliding with any running `1c serve`. The fix is `server.ws: false`. Let me verify it empirically before editing.

Confirmed: `ws: false` suppresses the error and `ssrLoadModule` still works (the `1c serve` on 24678 is still running, no collision). Now the full free-coding flow. Let me read the ticket and existing test conventions.

Blank ticket — nothing to preserve. `xgd` is the ticketing CLI. Now let me look at existing test conventions and the version-bump gate.

Now I have everything. Let me apply the fix to the launcher, add the UAT, then handle the ticket/version/commit.

**1. Fix the ERROR** — disable the WebSocket in the launcher's Vite server:

**2. Add the UAT** — reproduces the collision by occupying 24678, asserts the launcher stays quiet:

Now let me verify the fix end-to-end (with 24678 still held by the running `1c serve`) and run the new UAT:

ERROR is gone (only the benign "Missing pages directory" WARN remains), and the UAT passes. Now the free-coding bookkeeping: version bump, commit, ticket. Let me check the ticket subcommands and bump the version.

Now let me set the ticket's title and behavior description (body):

Now commit the code (launcher + fix + UAT + version bump) with `[FREE-CODED]`:

Now move the ticket to `free_coded` (validates the version bump + records the commit):

All done. Ticket **REQ-37** is now `free_coded` with the commit and version recorded.

## Summary

**Committed** (`cf37056`, `[FREE-CODED]`, version `0.0.32 → 0.0.33`):

1. **`bin/1c`** — launcher wrapper. Resolves the repo root from its own location (works from any directory) and preserves the caller's CWD so `sites/`/`dist/` paths resolve correctly. Run `./bin/1c <cmd>`, or add `bin/` to PATH / alias it for bare `1c`.

2. **Fixed the ERROR** in `tools/generate/bin/1c.mjs` — root cause: under **Vite 8**, `createWebSocketServer` only returns a no-op stub when `server.ws === false`; the old `hmr: false` no longer suppresses it (verified in Vite's `node.js:16585`). So the CLI kept binding the fixed HMR port **24678**, colliding with a running `1c serve`. Adding `server.ws: false` returns the no-op stub — `ssrLoadModule` is unaffected, no more `Port 24678 is already in use`.

3. **UAT** `tests/req37-launcher.test.ts` — spawns the real launcher with 24678 occupied and asserts exit 0 + no port error. Passes.

**Verified:** with a `1c serve` still holding 24678, `./bin/1c list` now runs clean (only the benign `Missing pages directory` WARN remains — that's Astro noting there's no `src/pages`, unrelated to the error you flagged; leave it or I can silence it separately if you like).

One note: the commit deliberately excluded the unrelated modified `CLAUDE.md` and the untracked `.xgd/tickets/search/` and `storage/sites/faelan/` — those are outside this ticket's scope.

<!-- xgd-chat-end -->