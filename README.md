# Verified Software Bases

> Skip the blank prompt. Start from verified software.

A parked storefront prototype for packaged software **Bases**. Its retained
TinyGPT starter listing does not establish a currently verified preview,
purchasable package or completed delivery workflow.

Bet: even when code generation is cheap, people still pay for **judgment,
verification, packaging, ownership, and a path to launch**.

## Layout

```
verified-bases/
├── docs/
│   ├── PRD-full.md           — 35-section product spec (living)
│   └── docs/archive/2026-06-20-prd-distilled-phase1-shipped.md — archived short spec
├── web/                      — Astro 5 + React 19 + Tailwind v4 (Cloudflare Pages)
│   ├── src/
│   │   ├── data/bases.ts            — retained TinyGPT starter listing and schema
│   │   ├── pages/                   — /, /bases, /bases/[slug], /trust,
│   │   │                              /collab, /about, /privacy, /terms, /refund, /404
│   │   ├── components/astro/        — Nav, Footer, Head, BaseCard, TierCard, TierModal
│   │   ├── layouts/BaseLayout.astro
│   │   └── styles/global.css        — design tokens + primitives
│   ├── functions/api/[[path]].ts    — Pages Function proxy → Worker
│   ├── scripts/icons.mjs            — Favicon + OG raster generation
│   ├── astro.config.mjs · package.json · tsconfig.json · wrangler.jsonc
│   └── .env.example
├── api/                      — Go on Cloudflare Workers (TinyGo + syumai/workers)
│   ├── main.go                      — router
│   ├── handlers/                    — health, submit, intent, checkout, webhook,
│   │                                  Dodo client + Standard Webhooks verify,
│   │                                  Resend client, D1 store, KV rate-limit
│   ├── migrations/0001_init.sql     — D1 schema
│   ├── wrangler.jsonc · Makefile · go.mod · .dev.vars.example
│   └── README.md                    — TinyGo / wrangler specifics
├── .github/workflows/ci.yml  — type-check + build web; vet + TinyGo build api
├── DEPLOY.md                 — first-time deploy walkthrough
├── PROJECT_STATUS.md
└── README.md (this file)
```

## Stack

Follows fleet standards (matches `fleet/sarthakagrawal`):

**Frontend** — Astro 5 static · React 19 islands · Tailwind v4 via the Vite
plugin · Lightning CSS transformer + minifier · Geist + Geist Mono variable
fonts · Cloudflare Pages.

**Backend** — Go 1.22 compiled with TinyGo to WASM · `syumai/workers` for
the fetch ⇄ net/http bridge · Cloudflare D1 (managed SQLite) for orders,
intents, submissions, webhook log · Cloudflare KV for rate-limit counters.

**Payments** — Dodo Payments (REST). Standard Webhooks signing.

**Email** — Resend (transactional).

## Design language

Dark engineered aesthetic with an **amber accent (`#f0b54a`)** — a
"verified stamp" / curator's seal feel. Retune the entire site by editing
`--color-accent` in `src/styles/global.css`.

## Run locally

```bash
# Frontend (no backend dep — modal flows degrade gracefully without /api).
cd web && npm install && npm run dev
# → http://localhost:4321

# Backend (separate terminal). Requires TinyGo.
cd api
brew install tinygo                     # one time
cp .dev.vars.example .dev.vars && $EDITOR .dev.vars
make migrate-local
make dev
# → http://localhost:8787
```

## Deploy

See `DEPLOY.md` for the full first-time walkthrough.

```bash
cd api && make deploy
cd web && npm run build && npx wrangler pages deploy dist --project-name=verified-bases-web
```

## Status

**Inactive prototype.** The source implements a proposed manual curated store,
per `docs/PRD-full.md` §31. It contains a TinyGPT Specialist Starter listing,
prices and verification badges dated June 2026. Those historical labels are
not fresh qualification evidence. On 2026-09-08, `tinygpt.sarthakagrawal.dev`, the recorded Pages hostname
`verified-bases-web.pages.dev`, and target `bases.sarthakagrawal.dev` did not
resolve from the verification machine. No checkout or delivery was exercised.

Before resuming, establish one concrete buyer need and support economics, then
verify the exact deliverable, license, setup, preview and acquisition path.
Use an existing storefront for that first transaction if sufficient; expanding
marketplace infrastructure is not justified by the retained listing alone.
Payment provisioning and any publication remain separately approved actions.

### Deferred resumption requirements

These are historical requirements, not an active launch queue. No GitHub Issues
were open at the 2026-09-08 check.

1. Establish a concrete buyer need, one exact deliverable and support economics.
   Prefer an existing storefront for that first package; do not build a generic
   marketplace before demand is established.
2. Verify the package, license, setup instructions and a working preview against
   the actual deliverable. Historical verification badges are insufficient.
3. If this storefront is deliberately revived, reconcile retained provider
   resources and DNS, configure payment products/webhooks and transactional
   delivery, then prove a test purchase, order record, receipt, download and
   refund. Existing provisioning checkmarks do not prove current operation.
4. Publish only after the buyer path works. Keep creator dashboards, additional
   listings, automated fulfilment and recurring hosting deferred until demand
   justifies them.

The [historical status](docs/archive/2026-09-08-storefront-status.md) preserves
all former implementation notes and launch checklist items. The current
[project status](PROJECT_STATUS.md) points here as their single active home.

## License

Buyer-ownership model. See each Base listing for its license terms; see
`/terms` and `/refund` on the live site for marketplace-level terms.
