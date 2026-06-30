---
uid: request-8c8a49e6
id: REQ-1
type: request
title: 'Monorepo: scaffold workspace + two-Worker Cloudflare deploy pipeline'
created_by: xgd
created_at: '2026-06-30T16:27:14.241274+00:00'
updated_at: '2026-06-30T16:28:28.195224+00:00'
completed_at: null
last_field_updated: story_points
status: draft
fields:
  auto_merge_back: true
  needs_review: false
  priority: medium
  story_points: 3
---

## Scope

Stand up the empty bones of the 1stcontact.io platform: a pnpm monorepo with two Cloudflare Worker apps, scaffolding for all future packages, and a GitHub Actions workflow that auto-deploys both Workers on every push to `xgd-stable`.

Design discussion: see CHAT-10 (Project Build and Deploy), CHAT-7 (Framework), CHAT-9 (Builder).

## Why free-coded

Scaffolding/infrastructure work — no algorithmic design needed, just the right files in the right places. Small, cohesive, single intent.

## What shipped

### Repo shape

```
first-contact/
├── apps/
│   ├── public-site/       Worker for 1stcontact.io (placeholder response)
│   └── control-app/       Worker for app.1stcontact.io (placeholder response)
├── packages/
│   ├── framework/         (package.json + README placeholder)
│   ├── site-schema/       (package.json + README placeholder)
│   ├── builder-ui/        (package.json + README placeholder)
│   └── ui-kit/            (package.json + README placeholder)
├── sites/
│   └── first-contact/     (README placeholder)
├── tools/
│   └── generate/          (package.json + README placeholder)
├── db/migrations/         (empty, with .gitkeep)
├── tests/                 (vitest UAT suite)
├── .github/workflows/
│   ├── ci.yml             PR checks: install + build + test + dry-run deploys
│   └── deploy.yml         push to xgd-stable: deploy both Workers (production env)
├── pnpm-workspace.yaml
├── package.json           Root workspace + scripts
├── tsconfig.base.json     Shared TS config
└── vitest.config.ts       Test runner config
```

### Worker placeholders

- `apps/public-site/src/index.ts`: returns `"Hello from 1stcontact.io"` (text/plain, 200)
- `apps/control-app/src/index.ts`: returns `"Hello from app.1stcontact.io"` (text/plain, 200)
- Each app has its own `wrangler.toml`:
  - `public-site` production env: `1stcontact.io/*` on the 1stcontact.io zone
  - `control-app` production env: `app.1stcontact.io/*`
  - Both: `compatibility_date = "2025-07-01"` (latest supported by wrangler 3.114.17 runtime; bump when wrangler v4 lands)
  - Both: `compatibility_flags = ["nodejs_compat"]`
  - Both: `workers_dev = true` (default `.workers.dev` URL available)
  - No D1/R2/KV bindings yet (separate tickets)

### Root scripts (package.json)

- `pnpm dev` — runs `wrangler dev` for both Workers in parallel via concurrently (ports 8787 / 8788)
- `pnpm dev:public` / `pnpm dev:control` — individual
- `pnpm -r build` — workspace-wide build (Workers tsc --noEmit; placeholder packages echo)
- `pnpm test` — vitest run across all UATs
- `pnpm deploy:public` / `pnpm deploy:control` — manual deploy escape hatch
- `pnpm dryrun:public` / `pnpm dryrun:control` — used by CI workflow

Node 20+ and pnpm 9+ required (engines field). pnpm provisioned via corepack from `packageManager` field. `pnpm-lock.yaml` committed for `--frozen-lockfile` reproducibility.

### GitHub Actions

**`.github/workflows/deploy.yml`**: triggered on `push` to `xgd-stable` plus manual dispatch. Steps:
1. Checkout, pnpm/action-setup@v4 (v9), setup-node@v4 (Node 20), pnpm install --frozen-lockfile
2. `pnpm -r build`
3. `pnpm --filter @1stcontact/public-site exec wrangler deploy --env production`
4. `pnpm --filter @1stcontact/control-app exec wrangler deploy --env production`

Uses `concurrency: deploy-${{ github.ref }}` to serialize.

Secrets required (set manually before first deploy works):
- `CLOUDFLARE_API_TOKEN` — scopes: Account → Workers Scripts:Edit, Account → D1:Edit, Account → Workers R2 Storage:Edit, Zone → Workers Routes:Edit on the 1stcontact.io zone
- `CLOUDFLARE_ACCOUNT_ID`

**`.github/workflows/ci.yml`**: triggered on PRs against main / xgd-working / xgd-stable, plus manual dispatch. Steps: checkout, install, `pnpm -r build`, `pnpm test`, `pnpm dryrun:public`, `pnpm dryrun:control`.

## Explicitly NOT in this ticket

- D1 database creation, schema, or migrations
- R2 buckets or KV namespaces
- Framework package code (section library, renderer, theme tokens)
- Builder SPA implementation
- Real FC homepage content / static generator
- Preview deploys for PRs
- Wrangler v4 upgrade (v3.114.17 shipped; v4 changes the `unstable_dev` testing API)

## Test approach (UATs)

UAT runner: vitest 2 (devDep at root).

- `test_UAT_FC_REQ-1_public_site_returns_placeholder` — uses `unstable_dev` from wrangler to boot `apps/public-site`, fetches `/`, asserts 200 + body `Hello from 1stcontact.io` + `text/plain` content-type.
- `test_UAT_FC_REQ-1_control_app_returns_placeholder` — same for `apps/control-app` (body `Hello from app.1stcontact.io`).
- `test_UAT_FC_REQ-1_deploy_workflow_lints` — parses `.github/workflows/deploy.yml` with the `yaml` lib and asserts: (1) triggers on push to `xgd-stable`, (2) has a concurrency group, (3) exposes both Cloudflare secrets to the deploy job env, (4) invokes `wrangler deploy` for both `@1stcontact/public-site` and `@1stcontact/control-app`.
- `test_UAT_FC_REQ-1_ci_workflow_lints` — parses `.github/workflows/ci.yml` and asserts: (1) triggers on `pull_request`, (2) runs `dryrun:public` and `dryrun:control`, (3) runs `pnpm test`.

## Verification

- `pnpm test` — 4 test files, 9 tests, all passing in ~700ms
- `pnpm -r build` — clean across all 7 buildable packages
- `pnpm dryrun:public` and `pnpm dryrun:control` — both succeed, bundles 21.5 KiB / 5.1 KiB gzip

## Follow-up tickets (not in scope)

- Add `CLOUDFLARE_API_TOKEN` + `CLOUDFLARE_ACCOUNT_ID` to GitHub repo secrets (operator task, no code)
- D1 schema v0 + bindings on both Workers
- Framework Phase 0: section library + renderer (CHAT-7)
- Static generator (`tools/generate`) reading `sites/first-contact/` definition
- Builder SPA (CHAT-9)