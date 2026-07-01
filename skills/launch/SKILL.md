---
name: pisk:launch
description: Guide founders through the Pirate Launch Kit installation — Next.js, Clerk, Neon, Stripe, Resend, PostHog — and hand off to the growth skill collection.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:launch` — Pirate Launch Kit Setup

You are guiding a founder through installing and configuring the Pirate Launch Kit so their SaaS project is ready for the rest of the Pirate Skills collection.

## On invocation

**Step 1 — Resume check.** Read `.launch-kit-status.yaml` if it exists.

- If the file exists and all four phases are marked complete → jump to [Full completion](#full-completion).
- If the file exists with some phases complete → show the founder which phases are done, confirm the next incomplete phase, and resume from there. Skip the calibration question.
- If the file does not exist → this is a first run. Proceed to calibration.

## Calibration (first run only)

Ask exactly one question before doing anything else:

> "Before we start, how comfortable are you with terminal setup and web-service dashboards?
>
> 1 — I'll need every command explained step by step
> 2 — I need explanations but can follow along
> 3 — Show me what to do and I'll handle it
> 4 — I'm comfortable, just list the steps
> 5 — Just tell me what's needed; I'll figure it out
>
> Reply with a number."

Store the response as `$LEVEL` for the rest of this session.

**$LEVEL 1–2 (beginner) style:** Full step-by-step. Explain what each command does. Offer a session break between phases ("Want to stop here and continue in a new session?").

**$LEVEL 3 (intermediate) style:** Numbered steps, brief explanations for non-obvious parts.

**$LEVEL 4–5 (advanced) style:** Compact checklist. Minimal prose.

---

## Phase 1 — Foundation

*Goal: Node.js ≥ 20 installed, git repo initialised with secret-blocking hooks, Vercel CLI plugin installed.*

### Node.js runtime

Ask the founder to run:
```bash
node --version
```

Required: **v20 or higher**. If below v20 or missing, direct them to [nodejs.org](https://nodejs.org) → "LTS" download. Beginners: walk through the installer. Advanced: `nvm install 20 && nvm use 20`.

### Git with secret-blocking hooks

If no `.git` directory exists, initialise:
```bash
git init
```

Install secret-blocking pre-commit hooks to prevent `.env` files from being committed:
```bash
npx husky init
echo 'npx --no -- commitlint --edit $1' > .husky/commit-msg
echo '#!/bin/sh\nif git diff --cached --name-only | grep -E "^\\.env" ; then\n  echo "ERROR: Attempting to commit .env file. Remove it and add to .gitignore."\n  exit 1\nfi' > .husky/pre-commit
chmod +x .husky/pre-commit
```

Verify `.env*` patterns are in `.gitignore`. Add if missing:
```
.env
.env.local
.env.*.local
```

### Vercel plugin

```bash
npm install -g vercel
vercel --version
```

Confirm the CLI responds with a version number.

### Phase 1 complete

When Node.js, git hooks, and Vercel CLI are confirmed:
- Update `.launch-kit-status.yaml`:
```yaml
phases:
  "1":
    done: true
    completedAt: "<ISO timestamp>"
```
- Add `.launch-kit-status.yaml` to `.gitignore` if not already present.
- **Beginners ($LEVEL 1–2):** ask "Want to continue to Phase 2 now (deployment), or pause and come back?"
- **Others:** proceed immediately to Phase 2.

---

## Phase 2 — Deployment Readiness

*Goal: Live Vercel URL accessible, OTP / sign-in gate visible.*

### Deploy to Vercel

```bash
vercel
```

Follow the prompts: link to a Vercel project (or create new), accept defaults. When the deployment URL is printed, ask the founder to open it in a browser.

**Validate:**
- URL loads without error.
- A sign-in or OTP screen is visible (the scaffold includes a basic auth gate by default).

If the page 404s or shows a build error, guide the founder through checking the Vercel dashboard → Deployments → latest build logs for the error.

### Phase 2 complete

When the founder confirms the URL loads and the auth gate is visible:
- Update `.launch-kit-status.yaml` → add phase `"2"` with `done: true` and `completedAt: "<ISO timestamp>"` under `phases`.
- **Beginners:** offer session break.
- **Others:** proceed to Phase 3.

---

## Phase 3 — Service Integration

*Goal: Clerk, Neon, Resend, Stripe, and PostHog each configured and validated in sequence. Do not advance until each service passes its validation gate.*

### Clerk (auth)

**Setup:**
1. Go to [clerk.com](https://clerk.com) → create application → choose "Email + OTP" as the sign-in method.
2. Copy the API keys from the Clerk dashboard.
3. Add to `.env.local`:
```
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
```
4. Redeploy: `vercel --prod`

**Validation gate:** Ask the founder to complete a sign-up flow on the live URL, then open the Clerk dashboard → Users and confirm the new user appears. Do not proceed until they confirm.

**Diagnostics if validation fails:**
- "Clerk key not found" in logs → check key names match Clerk dashboard exactly.
- Sign-up form not showing → confirm Clerk middleware is wrapping `app/layout.tsx`.
- User appears locally but not in dashboard → check `NEXT_PUBLIC_` prefix on publishable key.

**`.env.example` update:** Append these keys (with placeholder values):
```
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_your_key_here
CLERK_SECRET_KEY=sk_test_your_key_here
```
Tell the founder to also add the real values in Vercel → Project Settings → Environment Variables.

---

### Neon + Drizzle (database)

**Setup:**
1. Go to [neon.tech](https://neon.tech) → create project → copy the connection string.
2. Add to `.env.local`:
```
DATABASE_URL=postgresql://...
```
3. Push the schema:
```bash
npx drizzle-kit push
```

**Validation gate:** Ask the founder to run:
```bash
npx tsx -e "import { db } from './src/db'; db.execute('SELECT 1').then(r => console.log('DB OK', r)).catch(console.error)"
```
Confirm output is `DB OK`. Do not proceed until confirmed.

**Diagnostics if validation fails:**
- `SSL required` error → append `?sslmode=require` to the connection string.
- `relation does not exist` → schema push did not complete; re-run `npx drizzle-kit push`.
- Connection timeout → Neon free tier sleeps; open the Neon dashboard to wake the project and retry.

**`.env.example` update:**
```
DATABASE_URL=postgresql://user:password@host/dbname?sslmode=require
```

---

### Resend (email)

**Setup:**
1. Go to [resend.com](https://resend.com) → API Keys → create key.
2. Add to `.env.local`:
```
RESEND_API_KEY=re_...
```
3. Verify the sender domain in Resend (or use the Resend test domain for development).

**Validation gate:** Ask the founder to run the test send:
```bash
npx tsx -e "
import { Resend } from 'resend';
const r = new Resend(process.env.RESEND_API_KEY);
r.emails.send({ from: 'onboarding@resend.dev', to: 'YOUR_EMAIL', subject: 'pisk:launch test', text: 'Resend OK' }).then(console.log).catch(console.error);
"
```
Replace `YOUR_EMAIL` with the founder's email address. Confirm they receive the email. Do not proceed until confirmed.

**Diagnostics if validation fails:**
- `401 Unauthorized` → API key incorrect or not loaded; check `.env.local`.
- Email not received → check spam folder; for custom domains, verify DNS records in Resend dashboard.

**`.env.example` update:**
```
RESEND_API_KEY=re_your_key_here
```

---

### Stripe (payments)

**Setup:**
1. Go to [stripe.com/dashboard](https://dashboard.stripe.com) → Developers → API Keys → copy test keys.
2. Add to `.env.local`:
```
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```
3. For the webhook secret: Stripe CLI → `stripe listen --forward-to localhost:3000/api/webhooks/stripe` and copy the signing secret it prints.

**Validation gate:** Ask the founder to run:
```bash
npx tsx -e "
import Stripe from 'stripe';
const s = new Stripe(process.env.STRIPE_SECRET_KEY!);
s.paymentIntents.create({ amount: 999, currency: 'usd' }).then(pi => console.log('Stripe OK', pi.id)).catch(console.error);
"
```
Confirm output starts with `Stripe OK pi_`. Also confirm the payment intent appears in the Stripe dashboard under Payments → test mode. Do not proceed until both confirmed.

**Diagnostics if validation fails:**
- `No such API key` → using live key in test mode; ensure `pk_test_` / `sk_test_` prefixes.
- Webhook 400 errors → webhook secret mismatch; re-run `stripe listen` and copy the fresh secret.

**`.env.example` update:**
```
STRIPE_PUBLISHABLE_KEY=pk_test_your_key_here
STRIPE_SECRET_KEY=sk_test_your_key_here
STRIPE_WEBHOOK_SECRET=whsec_your_secret_here
```

---

### PostHog (analytics)

**Setup:**
1. Go to [posthog.com](https://posthog.com) → create project → copy the project API key and host URL.
2. Add to `.env.local`:
```
NEXT_PUBLIC_POSTHOG_KEY=phc_...
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com
```
3. The scaffold includes PostHog provider in `app/layout.tsx`; confirm it wraps the app.

**Validation gate:** Ask the founder to open the live URL in a browser, navigate to any page, then open PostHog → Live Events. Confirm a `$pageview` event from their browser appears within 30 seconds. Do not proceed until confirmed.

**Diagnostics if validation fails:**
- No events appearing → check browser console for CORS errors; confirm `NEXT_PUBLIC_POSTHOG_HOST` has no trailing slash.
- Events appear in development but not production → confirm env vars are set in Vercel environment settings, not just `.env.local`.
- `phc_` key not loading client-side → must use `NEXT_PUBLIC_` prefix for client-side keys.

**`.env.example` update:**
```
NEXT_PUBLIC_POSTHOG_KEY=phc_your_key_here
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com
```

---

### Phase 3 complete

When all five services are validated:
- Update `.launch-kit-status.yaml` → add phase `"3"` with `done: true` and `completedAt: "<ISO timestamp>"` under `phases`.
- **Beginners:** offer session break.
- **Others:** proceed to Phase 4.

---

## Phase 4 — Handoff

*Goal: Confirm everything is live and point the founder at the next skills.*

### Handoff summary

Output a summary:

```
## Your stack is live ✓

Services confirmed:
- Clerk — auth and user management
- Neon + Drizzle — database and ORM
- Resend — transactional email
- Stripe — payments (test mode)
- PostHog — product analytics

Environment variables documented in .env.example.
All keys added to Vercel environment settings.

## Next steps

Run the context skills to encode your business knowledge into the repo:

1. /pisk:growth — business model, goals, acquisition, OKRs
2. /pisk:funnel — ideal customer profile and offer
3. /pisk:content — story, voice, content cadence
4. /pisk:analytics — measurement stack, conversion events, ad signals

Then the operating loops read that context and run daily:
- /pisk:post, /pisk:reconcile, /pisk:meta, /pisk:linkedin, /pisk:ab
```

Close with: **"Your stack is live. Run `/pisk:growth` to start encoding your business context."**

Update `.launch-kit-status.yaml` → add phase `"4"` with `done: true` and `completedAt: "<ISO timestamp>"` under `phases`.

---

## Full completion

When `.launch-kit-status.yaml` shows all four phases done, output:

```
## Pirate Launch Kit — already complete

Your stack was set up on <Phase 1 date>. All services confirmed:
Clerk · Neon · Resend · Stripe · PostHog

If you need to re-configure a service, delete the relevant phase entry from
.launch-kit-status.yaml and re-run /pisk:launch.

Ready to grow? Run /pisk:growth.
```

---

## `.launch-kit-status.yaml` format

```yaml
phases:
  "1":
    done: true
    completedAt: "2025-01-15T09:23:00Z"
  "2":
    done: true
    completedAt: "2025-01-15T09:45:00Z"
  "3":
    done: true
    completedAt: "2025-01-15T11:02:00Z"
  "4":
    done: true
    completedAt: "2025-01-15T11:05:00Z"
```

Always check `.gitignore` contains `.launch-kit-status.yaml` before writing it for the first time. Add the entry if missing.

---

## General rules

- Never skip a validation gate. Each service must pass its test before the next service begins.
- Always update `.launch-kit-status.yaml` immediately when a phase completes.
- Adjust verbosity to `$LEVEL` throughout. Beginners need every command explained; advanced founders just need the commands.
- If the founder is blocked at a validation gate, exhaust the diagnostics listed before suggesting they skip the service.
- Do not modify any files in this skills repo. All work happens in the founder's project repo.
