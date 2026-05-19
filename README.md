# BoxBazar

> AI receptionist + F-commerce operations SaaS for Bangladeshi Facebook/Instagram/WhatsApp sellers.

BoxBazar replaces the constant DM grind. A seller connects her Facebook Page; an LLM-powered receptionist talks to customers, identifies products, collects orders, and pushes structured drafts to an admin panel where the seller approves and dispatches with one click via integrated couriers.

**Status:** active development. Web app first, Flutter mobile to follow. Bengali / Banglish first-class.

---

## What it does today

- **AI receptionist** on Messenger DMs — replies, matches products, negotiates basic terms, drafts orders.
- **Multimodal** — reads customer-sent product photos, matches against the seller's catalog.
- **Bangla-aware** — infers `bhai` / `apu` from the customer's name, mirrors the seller's voice from starred conversations.
- **Order approval queue** — every AI-drafted order lands as `pending_approval`; seller confirms with one click.
- **Courier integration** — Steadfast, Pathao, RedX (book consignment, label PDF, status webhooks).
- **Two-factor authentication** — admin-gated platform settings, email-delivered 6-character codes.
- **Owner / admin panel** — `/login/admin-p` for the platform owner; impersonate any user for support without seeing private data.

## Tech stack

| Layer | Choice |
|---|---|
| Monorepo | pnpm workspaces + Turborepo |
| Web | Next.js 14 (App Router), Tailwind, Zustand, React Query |
| API | Fastify 4, JWT, BullMQ + Redis, Prisma 5 |
| Database | PostgreSQL |
| AI | Google Gemini 2.5 Flash (multimodal, JSON output, ~$0.0003/message) |
| Messenger | Meta Graph API v21 (webhook verify, Send API, long-lived Page tokens) |
| Email | nodemailer + SMTP |
| Couriers | Steadfast, Pathao, RedX REST APIs |

## Repo layout

```
apps/
  api/         Fastify API + BullMQ workers + Prisma client wiring
  web/         Next.js dashboard
packages/
  ai-engine/   Two-stage AI receptionist (intent → response + order extraction)
  meta-sdk/    Messenger webhook + Send API + attachment fetcher
  courier-sdk/ Steadfast / Pathao / RedX clients
  shared/      Shared validators (BD phone, address) used by API + engine
  db/          Prisma schema + generated client
```

## Quick start

### Prerequisites

- Node.js 20+, pnpm 9
- PostgreSQL 14+ running on `localhost:5432`
- Redis 7+ running on `localhost:6379`
- A Google Gemini API key (free tier works for dev)
- (Optional) A Meta App, an ngrok / Cloudflare Tunnel URL, an SMTP account

### Setup

```bash
pnpm install
pnpm --filter @fcommerce/db generate
pnpm --filter @fcommerce/db exec prisma migrate deploy
pnpm dev
```

### Required env (`apps/api/.env`)

```
DATABASE_URL=postgresql://user:pass@localhost:5432/boxbazar
REDIS_URL=redis://localhost:6379
JWT_SECRET=<32+ random chars>
ENCRYPTION_KEY=<64 hex chars — used for AES-GCM secret-at-rest>
PORT=3001
NODE_ENV=development
```

Everything else (Meta App ID + Secret, Gemini key, SMTP, courier tokens) is configured **from the UI** at `/platform-setup` after the first user registers — no `.env` edits required for ongoing operations.

### Open the app

- **Web** → `http://localhost:3000`
- **API** → `http://localhost:3001`

The first user to register is auto-promoted to admin. From there:

1. `/settings` → Enable 2FA (a 6-character code lands in your inbox).
2. `/platform-setup` → save Meta App ID + Secret + Verify Token + Gemini key.
3. `/settings` → connect a Facebook Page (long-lived token exchange runs automatically).
4. `/products` → add at least 5 active products.
5. `/settings` → toggle AI Receptionist **on**.
6. DM the page from a Meta App admin/dev account → the AI replies.

## Architecture highlights

- **Two-stage AI pipeline.** Stage 1 is a cheap intent classifier; Stage 2 generates the actual reply + extracts order entities. Image messages skip Stage 1 entirely (saves a call).
- **Few-shot learning per shop.** Sellers star their best conversations; those are injected into the system prompt as "examples from this shop." A curated 32-conversation library ships as the baseline for new sellers.
- **Name-aware salutation.** The AI infers `bhai` / `apu` from typical Bangladeshi name patterns; stays neutral on ambiguous or foreign names; mirrors customer self-references.
- **Budget guardrail.** Per-customer rate limit (30 inbound msgs / hour) keeps a single chatty customer from blowing the per-seller LLM budget. Long-lived Facebook page tokens prevent the "reconnect every 60 minutes" trap.
- **Admin impersonation.** The platform owner can open any user's `/settings` to configure on their behalf without seeing inbox / orders / private data. Every impersonation is audit-logged.

## Cost target

~$0.0003 — $0.0006 per AI-handled customer message on Gemini 2.5 Flash. At 500-1000 messages / seller / month, that's under $1 / seller / month in AI cost, leaving comfortable margin under a $2-3 / seller budget.

## Roadmap (high level)

- ✅ AI receptionist + Messenger integration
- ✅ Multimodal image understanding
- ✅ Admin role + email MFA + owner impersonation
- ✅ Long-lived Page Access Token exchange
- 🔜 Pacing + message chunking (deliberate typing delays, multi-message replies)
- 🔜 Voice note transcription
- 🔜 Customer profile injection (past orders, preferred language)
- 🔜 Subscription billing (bKash + SSLCommerz)
- 🔜 Flutter mobile app (v1.1)

## License

Private / Proprietary. © 2026 BoxBazar.
