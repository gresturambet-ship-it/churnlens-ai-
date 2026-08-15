# ChurnLens AI — Backend

Production Express + TypeScript backend: Claude API proxy, JWT auth (email + Google + GitHub),
Supabase persistence, Stripe subscriptions, rate limiting, structured logging.

## ⚠️ Before you start

This code was written and bracket/syntax-checked in a sandbox **without internet access**,
so `npm install` and `tsc` could not actually be run here. Run them yourself before deploying:

```bash
npm install
npm run typecheck
npm run build
```

Fix anything that surfaces — dependency versions above are current as of writing but may have
moved on since.

## 1. Supabase setup

1. Create a project at supabase.com.
2. Open SQL Editor → paste the contents of `supabase/schema.sql` → Run.
3. Copy `Project URL`, `anon public` key, and `service_role` key into `.env`.

## 2. Google OAuth

1. console.cloud.google.com → APIs & Services → Credentials → Create OAuth client ID (Web).
2. Authorized redirect URI: `https://your-backend-domain.com/api/auth/google/callback`
3. Copy Client ID/Secret into `.env`.

## 3. GitHub OAuth

1. github.com/settings/developers → New OAuth App.
2. Callback URL: `https://your-backend-domain.com/api/auth/github/callback`
3. Copy Client ID/Secret into `.env`.

## 4. Stripe

1. dashboard.stripe.com → create two recurring Prices (Pro, Business).
2. Copy their price IDs into `.env` (`STRIPE_PRICE_PRO`, `STRIPE_PRICE_BUSINESS`).
3. Developers → Webhooks → add endpoint `https://your-backend-domain.com/api/billing/webhook`,
   subscribe to `checkout.session.completed` and `customer.subscription.deleted`.
4. Copy the webhook signing secret into `.env`.

## 5. Run locally

```bash
cp .env.example .env   # fill in all values
npm install
npm run dev             # http://localhost:8080
```

## 6. Deploy

Any Node host works (Railway, Render, Fly.io, a VPS). Vercel serverless functions need
adapting the Express app to their handler format — plain Railway/Render is simplest for
a stateful Express app like this one.

Set all `.env` values as environment variables in your host's dashboard — never commit `.env`.

## API surface

| Method | Path                          | Auth | Description |
|--------|-------------------------------|------|--------------|
| POST   | /api/auth/signup              | –    | Email+password signup |
| POST   | /api/auth/login               | –    | Email+password login |
| GET    | /api/auth/google               | –    | Start Google OAuth |
| GET    | /api/auth/github               | –    | Start GitHub OAuth |
| POST   | /api/analyze                  | JWT  | Run churn analysis, persist report |
| POST   | /api/chat                     | JWT  | Ask a follow-up question grounded in a saved report |
| GET    | /api/reports                  | JWT  | List saved reports |
| GET    | /api/reports/:id               | JWT  | Fetch one report |
| DELETE | /api/reports/:id               | JWT  | Delete a report |
| GET    | /api/reports/compare/summary   | JWT  | Compare current vs. prior period |
| POST   | /api/billing/checkout          | JWT  | Create a Stripe Checkout session |
| POST   | /api/billing/webhook           | –    | Stripe webhook (raw body, signature-verified) |
| GET    | /health                        | –    | Liveness check |

## What this does NOT include

- A frontend build — pair this with the ChurnLens AI artifact/frontend, pointing its
  API calls at this backend's `/api/*` routes instead of directly at Anthropic.
- Enterprise plan billing logic (seats, invoicing) — add a fourth Stripe Price and
  extend `PLAN_PRICE_MAP` in `src/routes/stripe.ts`.
- Email verification / password reset flows — straightforward to add with Supabase's
  own auth email templates if you switch to `supabase.auth` instead of the custom
  `users` table used here.
