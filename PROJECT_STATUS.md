# verified-bases — PROJECT_STATUS

> **ARCHIVED 2026-07-04 — removed from active fleet.**
>
> The storefront thesis is parked: paid checkout never landed (Cloudflare/Dodo
> provisioning was the blocker). The source retains a TinyGPT starter listing,
> but current preview, acquisition and delivery are unqualified. The repo
> is kept as history; no further active development. Reopen trigger: a concrete
> buyer/creator demand signal for verified starter "Bases."

Last reviewed: 2026-09-07. The implementation notes below are historical;
the current qualification limits and resumption criteria are in the README.

## Why/What

Verified Software Bases is a **Phase 1 personal storefront** for verified software Bases — curated packages where buyers preview, pay, own, remix, or get launch help. Every listing is Sarthak-built today. `/collab` captures inbound creator interest without marketplace UX commitment. Platform infra and first catalogue entry are shipped; **paid checkout awaits Cloudflare/Dodo provisioning**.

Bet: even when code generation is cheap, people still pay for judgment, verification, packaging, ownership, and a path to launch.

## Dependencies

| Layer | Choice |
|-------|--------|
| Frontend | Astro 5 + React 19 islands + Tailwind v4 (`web/`) |
| CSS | Lightning CSS transformer + minifier via Vite plugin |
| Backend | Go 1.22 → TinyGo WASM on Cloudflare Workers (`api/`) via `syumai/workers` |
| Data | D1 (orders, intents, submissions, webhook log) · KV (rate limits) · R2 (deliverables) |
| Payments | Dodo Payments REST + Standard Webhooks signing |
| Email | Resend transactional |
| Bot protection | Cloudflare Turnstile |
| Deploy | Cloudflare Pages (`verified-bases-web`) + Worker (`verified-bases-api`) |
| CI | `.github/workflows/ci.yml` — web type-check/build; api vet + TinyGo build |

**Local dev:**
- Frontend: `cd web && npm install && npm run dev` → http://localhost:4321 (degrades gracefully without `/api`)
- Backend: `cd api && cp .dev.vars.example .dev.vars && make migrate-local && make dev` → http://localhost:8787

**Key checks:** `cd web && npm run build` · `cd api && go vet && make build`

**Living spec:** `docs/PRD-full.md` (35-section product spec — positioning, tiers, trust model)

```
bases.sarthakagrawal.dev
        │
        ▼
Cloudflare Pages (verified-bases-web)
  static Astro (web/dist)
  /api/* → Pages Function proxy
        │ service binding "API"
        ▼
Cloudflare Worker (verified-bases-api) — Go/TinyGo
  D1 verified-bases-db
  KV RATELIMIT
  R2 verified-bases-delivery
  secrets: DODO_*, RESEND_*, TURNSTILE_*, DELIVERY_SECRET, DODO_PRODUCT_* per tier
```

**Checkout flow:** TierModal → Dodo checkout session OR intent capture fallback (`503 intent_captured` when product ID missing) → webhook updates D1 `orders` → Resend email → optional R2 signed download via `DELIVERY_SECRET`.

**Price authority:** display prices in `web/src/data/bases.ts`; authoritative cents in `api/handlers/prices.go` — must stay in sync.

| Concern | Detail |
|---------|--------|
| Price sync | Edit `bases.ts` + `prices.go` together on any tier price change |
| Validation metric | Will people pay for verified outcomes vs prompting from scratch? Track via D1 `orders` and `intents` |
| CI | Push triggers web build + api vet/build — keep green before deploy |
| Secrets | Never commit `.dev.vars`; production secrets via `wrangler secret put` only |
| Fulfillment | Phase 1 manual for Launch Help and remix; automate only after paid demand proven |

## Timeline

| Phase | Milestone |
|-------|-----------|
| Platform scaffold | Astro 5 storefront, Go/TinyGo Worker API, D1 schema, Dodo/Resend/Turnstile integrations, CI |
| Trust & legal | `/trust`, `/collab`, about/privacy/terms/refund pages, buyer-ownership license model |
| First Base (2026-06-20) | TinyGPT Specialist Starter catalogue entry with 5 tiers and backend price mirror |
| Pre-launch ops | Cloudflare resource provisioning, Dodo product registration, deploy + webhook wiring pending |
| Phase 2 (deferred) | Marketplace UX, creator dashboard, automated fulfillment — gated on `/collab` demand signal |

## Products

**Target domain:** `bases.sarthakagrawal.dev`

**Primary routes:** `/` · `/bases` · `/bases/[slug]` · `/trust` · `/collab` · `/about` · `/privacy` · `/terms` · `/refund` · `/404`

**API (Worker):** `/api/health` · `/api/submit` · `/api/intent` · `/api/checkout` · `/api/webhook/dodo` · `/api/download` (signed delivery)

| Surface | Role |
|---------|------|
| Storefront | Bases index and detail pages with tier selection |
| Trust | Verification model and buyer ownership explanation |
| Collab | Inbound creator interest form (supply-side signal) |
| Checkout | Dodo session or intent capture fallback |
| Delivery | Signed R2 download for paid tiers |

**First catalogue entry — TinyGPT Specialist Starter:**
- **Slug:** `tinygpt-specialist-starter` · **Category:** creator-tool · **Preview:** https://tinygpt.sarthakagrawal.dev
- **Tiers:** preview (free) · use ($29) · own ($99) · remix ($79) · launch ($199)

## Features (shipped)

### Platform — frontend
- Astro 5 storefront with dark engineered aesthetic and amber accent (`#f0b54a` in `global.css`).
- All marketing and legal pages: home, bases index, base detail `[slug]`, trust, collab, about, privacy, terms, refund, 404.
- React islands: TierModal → checkout or intent capture; TierCard; BaseCard; BaseAnatomy; CategoryIcon; EmptyStateArt.
- Nav, Footer, Head components with OG/favicon set (`scripts/icons.mjs` raster generation).
- Mobile-responsive layout; Geist + Geist Mono variable fonts.
- Pages Function proxy at `web/functions/api/[[path]].ts` → Worker service binding.
- `web/.env.example` with `PUBLIC_TURNSTILE_SITE_KEY`, `PUBLIC_SITE_ORIGIN`, analytics tokens.
- `web/wrangler.jsonc` Pages configuration; CI build on push.

### Platform — backend
- Go Worker router (`api/main.go`) with handlers: health, submit, intent, checkout, webhook, delivery, turnstile verify, middleware, rate limit.
- D1 schema `migrations/0001_init.sql` — orders, intents, submissions, webhook log tables.
- Dodo client + Standard Webhooks signature verification (`handlers/dodo.go`, `handlers/webhook.go`).
- Resend email client for transactional messages (`handlers/resend.go`).
- KV rate-limit counters (`handlers/middleware.go`).
- Turnstile server-side verification (`handlers/turnstile.go`).
- Checkout session creation (`handlers/checkout.go`); intent capture when tier not yet provisioned (`handlers/intent.go`).
- Signed R2 download path (`handlers/delivery.go`) with `DELIVERY_SECRET` HMAC.
- `api/.dev.vars.example` documenting all secrets including per-tier `DODO_PRODUCT_*` env var naming convention.
- `api/Makefile` — migrate-local, migrate-remote, dev, deploy targets.
- `DEPLOY.md` first-time deploy walkthrough (D1, KV, R2, Turnstile, Dodo, service binding, smoke checklist).
- `.github/workflows/ci.yml` — web + api green builds.

### First Base — TinyGPT Specialist Starter (2026-06-20)
- Catalogue entry in `web/src/data/bases.ts`:
  - **Slug:** `tinygpt-specialist-starter`
  - **Category:** creator-tool
  - **Preview:** https://tinygpt.sarthakagrawal.dev
  - **Badges:** live-preview-verified, source-build-verified, remix-ready, local-first, commercial-use-allowed, no-external-api
  - **does / doesNotDo / limitations / diy / alternatives** per PRD §14 listing requirements
  - **lastVerified:** 2026-06-20
- **Tiers (5):**
  | Tier | Key | Price | Includes |
  |------|-----|-------|----------|
  | Live preview | `preview` | Free | Browser playground, scope doc, eval methodology overview |
  | Use it | `use` | $29 | Packaged eval bundle, MLX export script, usage guide (no source) |
  | Own it | `own` | $99 | GitHub source handoff, commercial license, setup guide, model-card template |
  | Remix run | `remix` | $79 | One scoped customization pass (branding/niche adapter) |
  | Launch help | `launch` | $199 | Guided Mac setup, checkpoint wiring, first eval-gate smoke, launch checklist |
  - **startingPrice:** $29 (Use it tier)
- **Backend price mirror** in `api/handlers/prices.go`:
  - `tinygpt-specialist-starter:use` → 2900¢
  - `tinygpt-specialist-starter:own` → 9900¢
  - `tinygpt-specialist-starter:remix` → 7900¢
  - `tinygpt-specialist-starter:launch` → 19900¢
  - Preview tier intentionally absent (always free, no checkout row)
- Dodo product ID env vars documented: `DODO_PRODUCT_tinygpt_specialist_starter_{use,own,remix,launch}`

### Trust and positioning
- `/trust` page explaining verification model and buyer ownership.
- `/collab` inbound creator interest form (supply-side signal without Phase 2 marketplace UX).
- Buyer-ownership license model documented per listing and `/terms` + `/refund`.

## Todo / Planned / Deferred / Blocked

### Planned
1. **Register Dodo products** for TinyGPT Specialist Starter paid tiers; set `DODO_PRODUCT_*` Worker secrets and `DODO_API_KEY` / `DODO_WEBHOOK_SECRET` (operator action — requires Dodo account).
2. **Webhook registration** — add Dodo webhook URL pointing to `/api/webhook/dodo`; subscribe to payment/refund events (operator action).
3. **End-to-end smoke** — collab form → checkout test mode → webhook → order row in D1 → Resend email → refund path (requires Dodo + Resend secrets).
4. **R2 delivery** — upload Use/Own tier artifacts; set `DODO_DELIVERY_*` secrets for instant signed download in success email (else manual fulfillment runbook).
5. **DNS** — wire `bases.sarthakagrawal.dev` custom domain to Pages project (operator action).

### Pre-launch checklist
- [x] `npx wrangler login`
- [x] Create D1 → paste `database_id` into `api/wrangler.jsonc`
- [x] `make migrate-remote` (migrations 0001 + 0002 applied 2026-06-28)
- [x] Create KV namespace → paste id into `wrangler.jsonc`
- [x] Create R2 bucket `verified-bases-delivery`
- [x] Create Turnstile site → `TURNSTILE_SECRET` (Worker) + `PUBLIC_TURNSTILE_SITE_KEY` (Pages)
- [ ] `wrangler secret put DODO_API_KEY`
- [ ] `wrangler secret put DODO_WEBHOOK_SECRET`
- [ ] `wrangler secret put RESEND_API_KEY`
- [x] `wrangler secret put DELIVERY_SECRET` (`openssl rand -hex 32`)
- [ ] Register 4 Dodo products (use/own/remix/launch) → set `DODO_PRODUCT_tinygpt_specialist_starter_*` secrets
- [x] `make deploy` (api) + Pages deploy (web)
- [x] Wire service binding `API` on Pages project
- [ ] Register Dodo webhook URL
- [ ] Test checkout in Dodo test mode → verify D1 `orders` row + email
- [ ] Upload R2 deliverables + optional `DODO_DELIVERY_*` secrets

### Deferred
- Phase 2 marketplace: automated GitHub invite, creator dashboard, automated freshness checks, Cloudflare Rate Limiting binding swap (vs KV counters).
- Marketplace UX expansion until `/collab` validates supply-side demand.
- Additional catalogue Bases beyond TinyGPT Specialist Starter.
- Hosting tier recurring billing automation (Launch Help is one-time; hosting tier in schema reserved for later).
- Automated remix pipeline — Phase 1 remix is scoped manual pass.

### Blocked / Known gaps
- **No paid order has completed yet** — Dodo product registration + API keys are the ops-only blocker. Cloudflare provisioning (D1, KV, R2, Turnstile, Worker, Pages, service binding, migrations) is done.
- Tiers without `DODO_PRODUCT_*` secrets correctly fall back to intent capture — buyers see "Captured. We'll email within 24h" (by design for curated launch).
- R2 auto-delivery not configured — first paid orders may require manual fulfillment until `DODO_DELIVERY_*` and artifact uploads land.
- `DEPLOY.md` references target domain `bases.sarthakagrawal.dev` — DNS/custom domain wiring is operator step.
- Rate limiting uses KV counters; Phase 2 may swap to Cloudflare Rate Limiting binding.
