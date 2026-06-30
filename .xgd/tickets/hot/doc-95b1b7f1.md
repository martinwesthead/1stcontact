---
uid: doc-95b1b7f1
id: DOC-12
type: doc
title: Site Storage, Versioning & Rendering Model
created_by: xgd
created_at: '2026-06-30T20:21:05.234795+00:00'
updated_at: '2026-06-30T20:35:01.348247+00:00'
completed_at: null
last_field_updated: body
status: null
fields:
  doc_kind: architecture
---

# Site Storage, Versioning & Rendering Model

## 1. Purpose

Defines how a site is **stored**, **versioned**, and **rendered** — for the current file-backed system and the eventual Cloudflare (D1 + R2) system. Companion to [[DOC-7]] (framework) and [[DOC-5]] (platform).

This model **supersedes** the earlier framing in which a site was a single flat `site.json` and a revision captured only the definition JSON.

## 2. Core principles

1. **Source is versioned; renders are derived.** The versioned source of truth is the site *definition + assets + metadata*. Rendered HTML/CSS/JS is a derived build artifact, regenerated from source — never versioned.
2. **A version captures everything.** A revision is an immutable snapshot of the *entire* site state: page definitions, assets, **and** metadata — not just the definition.
3. **Forward-only history.** Revisions are immutable once published. The live site is **always the latest revision** (no separate "head"/`published_revision_id` pointer — it is derivable). To change a published site you return it to draft, edit, and publish a *new* revision. Nothing is rewritten or deleted; "rollback" = `checkout <old>` + `publish`.
4. **Private draft vs public published — separate locations.** Draft output is author-only; published output is public. They never share a location, so unpublished work can never be served.
5. **Git is not the versioning mechanism.** Git version-controls the repository; site versioning is the explicit revisions model below, which is identical in files and in D1. Git history does not migrate to Cloudflare.
6. **Server-side rendering only (for now).** HTML is rendered on a machine (build/CLI) and shipped fully-formed to the browser. There is **no client-side/in-browser renderer** until interaction latency proves one is needed.

## 3. On-disk layout (current — file-backed)

```
sites/<slug>/                  REAL sites — TRACKED IN GIT (source of truth, VERSIONED)
  draft/                       mutable working copy (hand-edited)
    site.json                  metadata: slug, accountId, displayName, theme, nav
    pages/  home.json about.json   one file per PAGE (module instances inline)
    assets/ hero.jpg logo.svg      media bytes
  revisions/
    0001/  0002/  ...          locked, COMPLETE snapshots (site.json + pages/ + assets/)
  history.json                 revision log (no head pointer; live = latest)

sandbox/<slug>/                THROWAWAY test sites — GITIGNORED (identical internal shape to sites/)

dist/<root>/<slug>/            RENDERED output — GITIGNORED, regenerated (root = sites | sandbox)
  draft/                       private preview   <- 1c render
  published/                   public site       <- 1c publish
```

**Granularity:** one file per page; module instances are inline within their page file. (Per-module files rejected as too granular to hand-author; one-giant-`site.json` rejected for scalability.)

### 3.1 Directory roots & git tracking

Three roots, two of them gitignored:

| Root | Purpose | Git |
|---|---|---|
| `sites/` | Real sites we will build and deploy to Cloudflare | **Tracked** |
| `sandbox/` | Throwaway space for experimenting with site creation (identical internal shape to `sites/`) | **Gitignored** |
| `dist/` | All rendered output, namespaced `dist/<root>/<slug>/` | **Gitignored**, never committed, always regenerable |

`.gitignore` carries `/sandbox/` and `/dist/`. The `1c` CLI operates on `sites/` by default; `--sandbox` targets `sandbox/`. Rendered output is **never** committed in either root — it is always reproducible from source.

## 4. history.json

Append-only log; one entry per publish. No `head` field (live = highest id).

```jsonc
{
  "site": "acme-bakery",
  "revisions": [
    {
      "id": "0001",
      "publishedAt": "2026-06-30T14:02:11Z",
      "publishedBy": "operator",
      "parent": null,
      "message": "Initial publish",
      "changes": [ { "path": "site.json", "op": "added" }, { "path": "pages/home.json", "op": "added" } ]
    },
    {
      "id": "0002",
      "publishedAt": "2026-06-30T16:20:05Z",
      "publishedBy": "operator",
      "parent": "0001",
      "basedOn": "0001",
      "message": "New hero copy, swapped logo",
      "changes": [ { "path": "pages/home.json", "op": "modified" }, { "path": "assets/logo.svg", "op": "added" } ]
    }
  ]
}
```

`changes` is computed by diffing the draft against the previous revision over the whole snapshot (definition + assets + metadata). `basedOn` is recorded when the draft was `checkout`'d from a non-latest revision.

## 5. Lifecycle & CLI (`1c`)

- **Author** edits files under `draft/` by hand.
- `1c render <slug>` — render `draft/` -> `dist/<slug>/draft/` (**private** preview).
- `1c publish <slug> [-m "msg"]` — snapshot `draft/` into the next locked revision, diff vs previous, append to `history.json`, **and render the new latest revision -> `dist/<slug>/published/` (public)**. Publish *always* renders.
- `1c checkout <slug> [<revId>]` — copy a revision's contents into `draft/` (default: latest).
- `1c revisions <slug>` — print the history log.
- **Rollback** is not a distinct command: `1c checkout <old>` then `1c publish`.

## 6. Rendering

- `render` is a **pure function** of `(source, framework)` -> static HTML/CSS/JS. Deterministic; reproducible.
- Two outputs, two audiences, two locations:

| | Draft preview | Published site |
|---|---|---|
| Rendered from | `draft/` | latest revision |
| Audience | author only (private) | public |
| Triggered by | `1c render` | `1c publish` (always) |
| Lives in | `dist/<slug>/draft/` | `dist/<slug>/published/` |

- Server-side only (see principle 6). The builder's future preview, when built, is server-rendered HTML shown in an iframe — not a client-side renderer.

## 7. Cloudflare mapping (eventual)

| Concept | File (now) | Cloudflare (eventual) |
|---|---|---|
| draft source | `sites/<slug>/draft/` | D1 draft + R2 draft assets |
| revision (snapshot) | `revisions/NNNN/` | R2 site snapshot + D1 revision metadata |
| history log | `history.json` | D1 `revisions` table |
| asset bytes | `.../assets/` | R2 (versioned with the revision) |
| published render | `dist/<slug>/published/` | Workers Static Assets / R2 (public URL) |
| draft preview | `dist/<slug>/draft/` | authenticated preview surface (control-app) |
| "live = latest" | highest revision | derivable; no `published_revision_id` pointer needed |

## 8. Versioning storage strategy

- **MVP: full snapshots** — a revision copies the whole working set. Simple, fully inspectable on disk.
- **Later (not now): content-addressed dedup** — blobs by hash + a manifest — when media scale demands it. The "revision = immutable snapshot" abstraction is unchanged by this optimization, so it can be introduced without a model change.

## 9. Known deferrals / caveats

- **Render fidelity for old revisions:** re-rendering uses the *current* framework, so an old revision can render differently than at original publish if the framework changed. MVP relies on the repo-pinned framework. Future options: pin a framework version per revision, or store rendered output.
- **Multi-tenant published-output serving** (per-site assets vs shared Worker + R2 by slug) — open; decided in a later REQ ([[DOC-7]] §11.3).
- **AI builder and client-side preview** — explicitly out of scope for now.

## 10. Reconciliations required in existing tickets/docs

- **REQ-7 (D1 schema):** drop `published_revision_id` (forward-only); revisions must snapshot *everything* (assets + metadata), not the definition only; adopt per-page granularity (aligns with [[DOC-5]]'s Pages/Sections entities).
- **REQ-4 (framework):** the render path need **not** be browser-ESM (no client-side preview); that constraint is lifted.
- **[[DOC-7]] §2.2/§2.4/§11** and **[[DOC-5]]** reconciled to this document.