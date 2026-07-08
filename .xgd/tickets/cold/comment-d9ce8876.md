---
uid: comment-d9ce8876
id: COMMENT-52
type: comment
title: Comment on chat CHAT-6
created_by: xgd
created_at: '2026-07-06T17:32:16.722263+00:00'
updated_at: '2026-07-06T18:40:53.524029+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  subject_uid: chat-5534f383
  kind: chat_transcript
---

<!-- xgd-turn id="eb13cfaf-bf5d-4021-be90-61062607f5a7-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T17:32:12.983313+00:00" -->
#### You
This is the first XGD project that has attempted to use pnpm configuration. It is proving a constant source of pain. I want you use this chat to help debug what is going wrong.

The key constraint is that I do not want to allow xgd to install packages on its own. I want the user to do the install explicitly. We can talk about whether that constraint makes sense but that is what we are designing around today.

The second constraint is that installed packages do not get added to git

So, when we make temporary branches: reconcile, resync, develop, regression the idea is to create symlinks from the branch to working where the real install exists.

Wrinkle that I do not understand properly: for testing it may be required that content created on the branch is "installed" if this is the case, it is very important that it is installed ON THE BRANCH and not following the symlink into working.

Can you review the current implementation and read back to me how it corresponds to the design constraints that I have outlined please?

<!-- xgd-turn id="eb13cfaf-bf5d-4021-be90-61062607f5a7-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T17:38:12.228934+00:00" -->
#### Claude
I'll investigate the current implementation. This is an analysis task, so let me dig into the code that handles worktree creation and package/node_modules linking across the temporary branches.

While that runs, let me look at the worktree infrastructure and any docs/config about this design directly.

I've launched two parallel investigations. Let me wait for them to report back.

This is key — pnpm uses relative symlinks from each package's `node_modules` back to the root `node_modules/.pnpm/` store. Let me wait for the agents to report the actual worktree-linking implementation before I analyze how well it fits the constraints.

Important detail already: the root `node_modules/*` symlinks use **absolute** paths into `/Users/martin/Projects/1stcontact/node_modules/.pnpm/...`, while the package-level symlinks (`packages/framework/node_modules/*`) use **relative** paths (`../../../node_modules/.pnpm/...`). That distinction matters a lot for a symlink-to-working strategy. Let me await the implementation findings.

Let me inspect how an existing live branch worktree currently has its `node_modules` set up — this shows me what the implementation actually produces.

The current implementation symlinks only the **root** `node_modules` to working; package-level `node_modules` (e.g. `packages/framework/node_modules`) don't exist on the branch. Let me verify whether pnpm's package-level dependency resolution actually survives that, since that's likely the source of pain.

Confirmed the likely root cause empirically. Now I need the implementation code to describe what it's *designed* to do versus what it actually produces. Both agents are still running — let me wait for their findings before synthesizing.

I have strong empirical findings already:

- **pnpm layout here**: root `node_modules/.pnpm/` is the real store; root `node_modules/*` has **absolute** symlinks but only for the *root* workspace's tooling deps (astro, vitest, typescript, wrangler…). Each package's own deps (e.g. `zod` for `site-schema`) live only under `packages/<pkg>/node_modules/*` as **relative** symlinks (`../../../node_modules/.pnpm/…`).
- **What the branch worktrees actually have**: only `<branch>/node_modules` symlinked → working's root `node_modules`. Package-level `node_modules` dirs are **absent** on the branch.
- **Consequence**: on a branch, `packages/site-schema` cannot resolve `zod` — it's not hoisted to root and the package-level `node_modules` doesn't exist. That's a concrete break, very likely "the constant source of pain."

Let me hold for the two agents to return the actual linking code.

No `node_modules`/`pnpm` references in `xgd_source` at all — so the linking is generic/config-driven, not hardcoded. Let me find the generic worktree-linking mechanism and its config.

xgd_source contains no symlink logic at all. The symlink was created by something else. Let me search the whole repo — `bin/`, hooks, and the live `.xgd/` config.

The first agent reports worktree creation lives in `xgd_source` but found no linking code — consistent with my greps. Yet the branch worktree *has* a `node_modules → working` symlink. I need to find where that comes from. Let me locate the worktree-creation code and any post-create setup/hook.

xgd_source in this repo has no worktree code at all — the real `xgd` must be pip-installed elsewhere. Let me locate the installed package.

The real XGD implementation lives in `/Users/martin/Projects/xgendev-main/` — my agents searched the 1stcontact repo, which doesn't contain the worktree/symlink code. Let me redirect. First confirm the source location and find the linking logic there.

Found it: `xgd_source/quality/worktree_setup.py` is the key file. Let me redirect the still-running agent and read the core file myself.

I'll just read the key files directly rather than redirect the agent.

Now the framework plugin that does the actual linking:

The base defines the hook; the JS plugin overrides it. But the builtin fallback only lists rust/swift/python — no JS plugin. Let me find the JS/node plugin.

No JS/pnpm plugin exists in the plugin registry — only python/rust/swift/playwright. So the `setup_worktree_dependencies` hook is effectively a **no-op** for this pnpm project. Let me check `cherry_pick.py` (reconcile's path) and the discoverer, plus whether a JS plugin is registered via entry points.

There's a separately-installed `VitestPlugin` (`ai.gendevlabs.javascript_vitest_open`). That's where the pnpm logic lives. Let me find and read it.

The second agent found the same code. Now let me read the actual VitestPlugin implementation of the symlink logic in full — this is the crux, especially the workspace-member handling and the install/isolation wrinkle.

Now I have the full picture. Let me confirm two final things: what `workspace_member_dirs` reads, and whether anything ever runs `pnpm install` on a branch (the isolation wrinkle).

I now have the complete picture across all three repos. Let me give you the readback.

## How the implementation actually works

The design is **not** in the 1stcontact repo — it lives in the XGD engine (`/Users/martin/Projects/xgendev-main/xgd_source`) plus a separately-installed quality plugin (`/Users/martin/Projects/xgd-plugins/.../javascript_vitest_open/plugin.py`, an editable install, so that file *is* what runs).

The chain, fired after **every** `git worktree add` (reconcile, resync, develop, regression):

1. **`worktree_setup.py::setup_worktree_dependencies(new_worktree)`** — resolves `xgd-working` as the canonical source, then calls every detected plugin's `setup_worktree_dependencies(new, source)` hook. It explicitly **never installs** — only links.
2. **`framework.py` base hook** — default no-op. The docstring states the contract: plugins "make runtime dependencies available in the new worktree **without reinstalling**."
3. **`VitestPlugin.setup_worktree_dependencies` (plugin.py:659)** — the real work:
   - Symlinks `<branch>/node_modules → <working>/node_modules` (root).
   - Then, for **each workspace member** (`packages/*`, `apps/*`, `tools/*` from `pnpm-workspace.yaml`), symlinks `<branch>/packages/foo/node_modules → <working>/packages/foo/node_modules` (this is the BUG-14 fix — see below).
   - Rewrites `.gitignore` so the symlink is actually ignored (BUG-13).
   - **Raises** if `<working>/node_modules` is missing, with "Run 'pnpm install' in xgd-working first."
4. **`_repair_source_node_module_symlinks` (worktree_setup.py:97)** — a cleanup pass over `xgd-working/node_modules` that runs on every setup, converting hoisted pnpm symlinks back to absolute paths.

## Mapping to your four constraints

**1. "XGD must not install packages on its own."** ✅ **Honored.** No code path runs `pnpm install`. The plugin only symlinks; lint/build use `npx --no-install`; `check_dependencies()` is read-only. When something's missing it raises with an instruction for *you* to install. This constraint is cleanly met.

**2. "Installed packages don't go in git."** ✅ **Honored**, and defensively so. `node_modules` is gitignored, and `_ensure_symlink_ignored` handles the subtle trap that `node_modules/` (trailing slash) matches directories but *not* symlinks — so it rewrites the pattern to a bare `node_modules` so git ignores the symlink too (BUG-13).

**3. "Temp branches symlink to working where the real install exists."** ✅ **This is exactly the design** — and it's more correct than what I first saw on disk. Your pnpm layout is the reason the member-level links matter:
   - Root `node_modules/*` holds **absolute** symlinks, but only for the *root* workspace's tooling (astro, vitest, typescript, wrangler…).
   - Each package's own deps (e.g. `zod` for `site-schema`) exist **only** under `packages/<pkg>/node_modules/*` as **relative** links (`../../../node_modules/.pnpm/…`), never hoisted to root.
   - So symlinking root alone is insufficient — `packages/site-schema` couldn't resolve `zod`. The BUG-14 member-loop fixes precisely this.

**Empirical caveat worth flagging:** the *live* worktrees on disk (`reconcile-BUNDLE-1`, dated Jul 3 16:31) have **only** the root symlink and no member-level ones — I verified `zod` is unresolvable there. The plugin file carrying the member-loop fix is dated Jul 4 10:27, i.e. **newer than those worktrees**. So the breakage you've been hitting is real and reproducible on the old worktrees; the fix landed but those stale worktrees predate it. A freshly-cut reconcile should now get member links. That timing gap is very likely a chunk of "the constant source of pain."

**4. The wrinkle — "if content on the branch must be installed, install ON THE BRANCH, not through the symlink into working."** ❌ **This is the real gap. The current design does not do this, and structurally cannot without change.**

Because both root and member `node_modules` are symlinks *pointing into working*, any `pnpm install` executed from a branch worktree writes **through** the symlink into `xgd-working/node_modules`. The engine authors clearly hit this: `_repair_source_node_module_symlinks`' own docstring describes pnpm-in-a-worktree writing hoisted symlinks back into xgd-working with paths like `…/reconcile-BUNDLE-5/node_modules/.pnpm/…` that go stale when the bundle worktree is deleted. So today's answer to the wrinkle is **"don't install on the branch; if you do, we scrub the damage afterward"** — repair-after-corruption, not isolation.

This means the two scenarios are handled asymmetrically:
- **Branch needs the *same* deps as working** → fine, sharing works.
- **Branch introduces a *new/changed* dep** (new package.json entry cherry-picked in) → there is **no mechanism** to materialize it on the branch. Nothing installs it, and if something did, it would leak into working. This is the REQ-38/REQ-44 failure class (dep declared, `node_modules` never materialized → "Cannot find module").

## The core tension to resolve

Your constraints #3 (share via symlink) and the wrinkle (isolate branch installs) are in direct conflict under a pure-symlink model:

- A symlink *is* working's tree — you can't both share it and write to it in isolation.
- To honor the wrinkle you'd need, for a branch that changes dependencies, either: (a) a **copy** (not symlink) of the relevant `node_modules` so a branch-local install stays local, or (b) a pnpm `node-linker`/store split where the branch gets its own virtual store overlaying the shared content-addressed store, or (c) detect "deps changed vs working" and only then break the symlink for the affected member and install there.

Reconcile already hints at wanting (c): `cherry_pick.py` re-runs `setup_worktree_dependencies` *after* the bundle's commits land, specifically so plugin detection "sees the branch's real post-cherry-pick tree instead of leaving a downstream `pnpm install` to silently produce an incomplete node_modules" — but it still only *symlinks*; it never installs, so a genuinely-new dep still isn't materialized.

---

Two questions to steer where we go next:

1. **Does a temp branch ever legitimately introduce a new dependency** that isn't already installed in `xgd-working`? If your discipline is "always `pnpm install` in working *before* promoting anything," then the wrinkle is largely theoretical and the priority is just making the symlink-sharing bulletproof (+ a loud preflight when a branch's `package.json`/lockfile differs from working's installed state — that's REQ-44). If branches *do* introduce new deps that must be tested before working has them, we need real per-branch isolation and the symlink model has to change for that case.

2. **Do you want me to verify the BUG-14 member-link fix on a fresh worktree** (create a throwaway reconcile/resync, confirm member `node_modules` appear and `zod` resolves on-branch), so we know whether current pain is "old worktrees pre-fix" vs "fix incomplete"?

I'd lean toward answering (2) empirically first — it tells us whether you're chasing an already-fixed bug or a live one. Want me to run that check?

<!-- xgd-turn id="56b4e202-6ab0-4533-a690-d9918d6458b7-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T17:58:23.870630+00:00" -->
#### You
ok I'm still trying to understand the current state:

```
((.venv-working) ) martin@nyx 1stcontact % tree node_modules
node_modules
├── @cloudflare
│   └── workers-types -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/@cloudflare+workers-types@4.20260630.1/node_modules/@cloudflare/workers-types
├── @vitest
│   └── coverage-v8 -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/@vitest+coverage-v8@4.1.9_vitest@4.1.9/node_modules/@vitest/coverage-v8
├── astro -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/astro@7.0.4_@astrojs+markdown-remark@7.2.0_@emnapi+core@1.11.1_@emnapi+runtime@1.11.1_@_e6c453bca761cf77889f3c2de0fd9183/node_modules/astro
├── concurrently -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/concurrently@9.2.3/node_modules/concurrently
├── jsdom -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/jsdom@25.0.1/node_modules/jsdom
├── typescript -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/typescript@6.0.3/node_modules/typescript
├── vitest -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/vitest@4.1.9_@types+node@26.0.1_@vitest+coverage-v8@4.1.9_jsdom@25.0.1_vite@8.1.0_@type_53557701d3ab3a888e1526d0323656ae/node_modules/vitest
├── wrangler -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/wrangler@4.106.0_@cloudflare+workers-types@4.20260630.1/node_modules/wrangler
└── yaml -> /Users/martin/Projects/1stcontact/node_modules/.pnpm/yaml@2.9.0/node_modules/yaml

12 directories, 0 files
((.venv-working) ) martin@nyx 1stcontact % tree .pnpm
.pnpm  [error opening dir]

0 directories, 0 files

```

There is no .pnpm?

<!-- xgd-turn id="56b4e202-6ab0-4533-a690-d9918d6458b7-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T17:59:00.160328+00:00" -->
#### Claude
Nothing's missing — this is a `tree` display artifact, not a real gap.

Two things are happening:

1. **`tree` hides dotfiles by default.** `.pnpm`, `.bin`, and `.modules.yaml` all start with a `.`, so `tree node_modules` silently omits them. That's why you see "12 directories, 0 files" and no `.pnpm` — it's there, just not shown. Add `-a`.

2. **`tree .pnpm` errored because there is no `.pnpm` at the repo root.** The store lives at `node_modules/.pnpm`, not `./.pnpm`. From the repo root `./.pnpm` genuinely doesn't exist, so "error opening dir" is correct.

Let me show you it's actually there:

So your working store is healthy and complete:

- **`node_modules/.pnpm/`** — 419 entries. This is the real content-addressed store; every actual package lives here.
- **`node_modules/<pkg>`** — the top-level symlinks you saw, pointing into `.pnpm`. Absolute paths, only for the root workspace's tooling deps.
- `.bin`, `.modules.yaml`, `.package-map.json` — pnpm bookkeeping, also dot-prefixed so `tree` hid them.

Nothing is missing. `tree` just doesn't show dot-entries without `-a`, and `.pnpm` is one level down inside `node_modules`, not at the repo root.

The mental model this confirms: **this one `node_modules` in `xgd-working` is the single real install** — the 419-package `.pnpm` store plus symlinks into it. Every temp-branch worktree is meant to be nothing but symlinks pointing back here (root `node_modules → working/node_modules`, and per BUG-14, each `packages/*/node_modules → working/packages/*/node_modules`). There is exactly one physical copy of the packages on disk; the branches borrow it.

That's also precisely why the wrinkle bites: since the branch's `node_modules` *is* working's store by reference, a `pnpm install` run from a branch writes back into this store — the very thing `_repair_source_node_module_symlinks` exists to clean up afterward.

Want me to now go verify (2) from before — cut a throwaway worktree and confirm whether the member-level `packages/*/node_modules` symlinks actually get created and resolve on-branch — so we know if your pain is old-worktree-pre-fix or a live bug?

<!-- xgd-turn id="6f526379-75f0-4415-8d50-303592af82b7-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T18:06:10.112248+00:00" -->
#### You
So it seems to me that a stronger design would be for node_modules to be a real directory on the branch node_modules/.pnpm would be an absolute symlink to Projects/1stconnect/node_modules/.pnpm  and the top level modules on the branches would also copies of the symlinks in working that way local coude could be installed in node_modules, right?

<!-- xgd-turn id="6f526379-75f0-4415-8d50-303592af82b7-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T18:08:44.985114+00:00" -->
#### Claude
That's a real improvement for the *read* path, and the key insight in it is genuinely elegant — but it doesn't fully deliver branch-isolated installs, because in pnpm the thing you'd need to isolate isn't the top-level links, it's the store. Let me check one load-bearing fact before I explain, because it changes the tradeoff.

Two facts from that matter for your question:

- **`nodeLinker: isolated`** — this is pnpm's strict layout, which is *why* the per-package `node_modules` exist and why sharing only the root is insufficient.
- **There's a global content store at `~/Library/pnpm/store/v11`**, and packages are brought into `.pnpm` by clone/reflink (link count 1, not found in-store by inode → APFS copy-on-write, not hardlink). This changes the economics, see below.

## Verdict on your proposed design

You're **half right** — and the half that's right contains a genuinely elegant trick, but it doesn't actually solve the wrinkle, because in pnpm you're isolating the wrong layer.

**The elegant part:** making `<branch>/node_modules/.pnpm` an absolute symlink to working's `.pnpm` means *every relative link resolves for free*. The member-level links in working are relative (`packages/framework/node_modules/astro → ../../../node_modules/.pnpm/…`). Reproduce those same relative links on the branch and they walk up to `<branch>/node_modules/.pnpm` → follow the symlink → land in the shared store. So one symlink at `.pnpm` makes the entire link-farm resolve, at root and in every member, without symlinking each member to working. That's cleaner than today's per-member-symlink approach.

**Why it doesn't deliver isolated installs, though:** in pnpm the *installed content lives in `.pnpm`* (the virtual store), and the top-level entries are just links into it. Your design shares `.pnpm` and isolates the links. But an install writes **content**, and content goes into `.pnpm`. So:

- Installing a package **already in working's store** → pnpm just adds a top-level link; that part is isolated. ✓ (This is the case your intuition is picturing.)
- Installing a **genuinely new** dependency (the actual reason you'd install on a branch) → pnpm must materialize it into `.pnpm`, which is a symlink to working. **It writes into working.** ✗ Same leak as today, just relocated.

So "local code could be installed in node_modules" is true only for the top-level linking layer, not for anything that adds store content — which is most real installs.

**And it's fragile:** pnpm *manages* `.pnpm` and `.modules.yaml`. A real `pnpm install` on the branch won't respect a hand-built hybrid — it'll rewrite the virtual store, and with `nodeLinker: isolated` it prunes anything not matching the lockfile (that's the REQ-44 "silent pruning" you already hit). Your `.pnpm`-symlink would get clobbered or written through on the first actual install.

## The fact that reframes the whole tradeoff

The plugin's premise is *"pnpm install is slow, so we symlink instead."* But you already have a **global content store**. A per-branch `pnpm install --offline` doesn't re-download or re-copy package bytes — it clones them from `~/Library/pnpm/store/v11`. The expensive part is resolution + link creation + clonefile ops (seconds on a monorepo), not gigabytes.

That means the genuinely stronger design isn't the hybrid — it's the **clean endpoint**: a branch that needs isolation gets its *own real `node_modules`*, populated by a real offline install from the shared global store. That's the only layout where "installed on the branch, not leaking into working" is actually true, and pnpm keeps it stable because it's pnpm-native. The hybrid tries to dodge the install but can't, because the store is where installs live.

## So the real design choice

The three requirements can't all hold for the *same* worktree at once:

| | No auto-install (#1) | Share, don't reinstall (#3) | Isolated branch install (wrinkle) |
|---|---|---|---|
| **Symlink-share (today)** | ✓ | ✓ | ✗ |
| **Your hybrid** | ✓ | ✓ (store) | ✗ (new deps leak) |
| **Per-branch real install** | ✗ (it installs) | partial (global store) | ✓ |

The resolution is **conditional, not one layout for everything**:

- **Branch doesn't change deps** (the common reconcile/regression case) → symlink-share is correct; keep it. It's the only thing that satisfies #1 for headless runs.
- **Branch's `package.json`/lockfile diverges from working's installed state** → *that's* when you need isolation. Detect the divergence and either (a) **fail loud** — "this branch changes deps; run `pnpm install` in working first" (the REQ-44 direction, cheapest), or (b) give that worktree a **real isolated install** from the global store.

The thing your instinct is really pointing at — "make node_modules real so branch installs stay local" — is right in spirit; it just has to be a *full* real node_modules for the diverging case, not a `.pnpm`-shared hybrid, because pnpm's isolation unit is the store.

Want me to make this concrete by measuring the actual cost of the "clean" path — time a `pnpm install --offline` into a throwaway worktree — so we know whether per-branch isolation is cheap enough to just do it whenever a branch's deps diverge? That's the one number that decides between "fail loud and make the user install in working" vs "isolate automatically."

<!-- xgd-turn id="1c280aa6-eff4-4c05-9f6e-b4d0ae943283-user" -->

<!-- xgd-chat role="user" ts="2026-07-06T18:36:52.057591+00:00" -->
#### You
Can we pause for a second and understand the use-cases for local installs? Any external package that is needed for any part of the project should be installed in the master repo in working. The only piece I am concerned about is testing the code on the branch - if modifications are made to the rendering engine, for example, does that need to be "installed" for it to be tested, or can it run from source in the repo filespace?

<!-- xgd-turn id="1c280aa6-eff4-4c05-9f6e-b4d0ae943283-assistant" -->

<!-- xgd-chat role="assistant" ts="2026-07-06T18:40:53.495438+00:00" -->
#### Claude
Excellent question to pause on — this is the actual crux, and it's answerable precisely. Let me check how workspace packages (like the rendering engine) are linked, because that determines whether edits run from source.

`framework` exports raw **`./src/index.ts`** with **no build step** — so it's consumed directly from TypeScript source. That's the key half of your answer. Now the critical half: where do those workspace links actually *point*?

The workspace link is **relative** (`../../../../packages/framework`) and resolves to source. That reframes your entire concern — let me verify the one thing that actually matters on a branch, because it's subtle.

The pre-fix worktree can't resolve it at all (the breakage you've been hitting). Let me prove what the *post-fix* member-symlink design does, since that's what determines whether branch edits are actually tested. I'll simulate the exact link structure the fix creates:

Confirmed empirically, and this **reframes your whole concern**. Let me verify the fix direction too (positive control):

Both directions proven. Now let me check how this repo's tests actually import the framework — that determines whether the bug bites in practice here:

That's the decisive detail. Let me check the one place it would still bite — the generate/render path importing by package name:

That completes the picture. Here's the direct answer to your question, plus the thing the investigation actually uncovered.

## Does the rendering engine need to be "installed" to be tested? — No.

Local/workspace code is **never installed**. pnpm wires workspace packages together with symlinks that point straight at the package's source directory, and `framework` specifically:
- exports **`./src/index.ts`** (raw TypeScript, not a built `dist/`), and
- has **no build step** (only `tsc --noEmit` for typechecking).

So an edit to `packages/framework/src/**` is live the instant you save it — no `pnpm install`, not even a build. Your rule holds cleanly: **external packages live in working; local code runs from source.** There is no legitimate "install on the branch" use-case for the thing you were worried about. `pnpm install` is only ever about external packages, which by your rule are always already in working.

## But the investigation found the *real* issue — and it's not about installing, it's about *which* source resolves.

This repo has **two** resolution paths, and they behave differently on a branch:

**1. The top-level `tests/` — safe.** Every test imports by **relative path**: `from '../packages/framework/src/modules/hero/index.astro'`. From a branch worktree that resolves to `<branch>/packages/framework/src` — the branch's own edited source. So a branch that validates the rendering engine via these vitest tests genuinely tests the branch's code. ✓

**2. The generate/render path — silently wrong.** `tools/generate` (and the apps) import the engine **by package name**: `from '@1stcontact/framework'` (a `workspace:*` dep). That resolves *through* `node_modules`. And I proved (by simulating the exact BUG-14 member-symlink layout) that because the branch borrows working's `node_modules` and the workspace links inside are relative, the branch's `@1stcontact/framework` resolves back into **working's** `packages/framework`, not the branch's:

```
branch consumer → @1stcontact/framework  →  /working/packages/framework   ← marker read "WORKING"
```

So if a branch edits the rendering engine and validation runs through `1c generate` / astro build / anything importing by package name, it **silently renders with working's engine, not the branch's**. Green results that prove nothing about the branch. That's worse than the "Cannot find module" crash you've been seeing — it's the silent version of the same root cause.

Your original "wrinkle" instinct — *"it must not follow the symlink into working"* — was **directionally correct but mislabeled**. It's not an install concern; it's a source-resolution concern.

## What this means for the design

The requirement is narrower and clearer than "isolate installs":

- **External store** (`.pnpm` + external top-level links): share from working. Identical, always-installed-in-working. ✓ Keep sharing.
- **Workspace links** (`@1stcontact/*`): must be **physically present in the branch's own `node_modules`** so their relative targets resolve to `<branch>/packages/*`. I verified the positive control — a branch-local relative workspace link resolves to the branch's source ("BRANCH" marker).

So the branch `node_modules` **cannot be a blanket symlink to working** — the workspace-link portion has to be branch-local. This is exactly the "real `node_modules` dir + shared `.pnpm`" shape you proposed earlier, but now with the *correct justification*: not install-isolation (never needed), but making the branch's own packages resolve to the branch's source.

Two things worth deciding:

1. **Confirm the exposure matters for your flows:** do reconcile/regression/develop ever validate by *rendering* (through `tools/generate`), or only by the relative-import vitest suite? If it's purely the vitest suite, you're accidentally safe today and this is lower-priority. If any branch validation renders, this is a real correctness hole.
2. Whether you want me to write this up as the design note for CHAT-6 (the workspace-link-locality requirement), so it's captured before we touch the plugin.

Want me to check what reconcile/regression actually run as their test/validation command — that tells us whether path #2 is exercised in practice?

<!-- xgd-chat-end -->