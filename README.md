# com-etzhayyim-app-saikin

saikin (細菌) — bacteria horizontal transfer layer. Propagates knowledge
laterally across disconnected actor clusters via probe, transfer, colony
formation, and lysis. It is the Cloudflare Worker serving
`saikin.etzhayyim.com` and `s41k1n01.etzhayyim.com`.

## Frontend migrated to ClojureScript (2026-09-07)

The `svelte/` directory (SvelteKit) is gone. The frontend is now
ClojureScript — reagent + re-frame + `jp-go-dds` (デジタル庁デザインシステム) —
at [`cljs/`](cljs). This was a **frontend-only** migration; the backend
Worker/XRPC logic was moved, not rewritten:

| Then | Now |
|---|---|
| `svelte/src/routes/+page.svelte` (the status page) | [`cljs/src/saikin/app.cljk`](cljs/src/saikin/app.cljk) — same fields, faithfully ported |
| `svelte/src/routes/xrpc/[...path]/+server.ts` (the file that actually deployed, per `wrangler.jsonc`'s old `main`) | [`src/xrpc-dispatcher.ts`](src/xrpc-dispatcher.ts) — moved byte-for-byte, only a provenance header comment added |
| `wrangler.jsonc` `main: svelte/.svelte-kit/cloudflare/_worker.js` | `main` dropped entirely |
| `wrangler.jsonc` `assets.directory: ./svelte/.svelte-kit/cloudflare/client` | `assets.directory: ./cljs/public` |

Neither `src/app.ts` nor the moved `src/xrpc-dispatcher.ts` calls
`env.ASSETS.fetch`, so `main` was dropped rather than repointed at either —
putting either Worker in front of the static assets with no
`env.ASSETS.fetch` call would mean nothing serves the frontend. Both backend
files are orphaned source, not currently wired to any deploy target. See
`wrangler.jsonc`'s header comment and `src/xrpc-dispatcher.ts`'s header
comment for the full reasoning.

**This is unverified**: `wrangler deploy` / `wrangler dev` were not run
against this change.

Three fields in `cljs/src/saikin/app.cljk`'s `default-db` were also
corrected, not merely ported, against what the old Svelte constant held —
see that namespace's docstring for detail:

- `:app/route-count` / `:app/routes` now report the two route patterns
  `wrangler.jsonc` actually declares, not the stale `0` / `[]` the Svelte
  constant carried.
- `:app/xrpc?` is now `false`, even though the Svelte constant carried
  `true`, because `main` no longer deploys the XRPC handler (it is
  preserved, unwired, at `src/xrpc-dispatcher.ts`).
- `:app/relative-path` now names `cljs/src/saikin/app.cljk`, not the
  deleted Svelte source path.

## Build and test

```bash
cd cljs
npm install
npm run build   # amu compile --target wasm32-browser app -> public/js/app.js
npm test        # amu compile --target wasm32-browser test && node out/tests.js
```
