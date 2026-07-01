# Pirate Skills (`pisk`)

## Install

```bash
claude plugin marketplace add shipstefan/pisk
claude plugin install pisk
```

Restart Claude Code after install. Skills are available as `/pisk:<name>` in any project.

> Plugin reference: https://code.claude.com/docs/en/plugins-reference.md

Marketing and growth skills for founders building agentic SaaS. Each skill turns a growth playbook into a `/pisk:<name>` slash command.

The collection works as a system: **scaffold** the product, **encode** business knowledge into the repo as `docs/context/` files, then run **daily operating loops** that read that context.

## Skills

### Foundation
Set up the product so the rest of the collection has something to run against.

- **`/pisk:launch`** — Scaffold the SaaS stack (Next.js, Clerk, Neon, Stripe, Resend, PostHog) through a guided, validated install. Run once, first.

### Context
Interview the founder and write `docs/context/` files that every agent session reads. Re-runnable to update individual domains.

- **`/pisk:growth`** — Business, team, goals, bottlenecks, vision, acquisition, OKRs → `docs/context/`.
- **`/pisk:funnel`** — Ideal customer profile and offer → `docs/context/icp.md`, `offer.md`.
- **`/pisk:content`** — Story, themes, voice, cadence → `docs/context/content/`.
- **`/pisk:analytics`** — Consent, behavior tooling, ad signals, database attribution → `docs/context/analytics.md`.

### Operations
Read the context files and run daily, often on a schedule.

- **`/pisk:post`** — Draft the day's content piece from the content context, at your set autonomy level (needs `pisk:content`).
- **`/pisk:reconcile`** — Daily whole-funnel reconciliation report, database as source of truth (needs `pisk:analytics`).
- **`/pisk:meta`** — Reconcile Meta Ads clicks vs. real conversions; optimize within autonomy limits (needs `pisk:analytics`, `pisk:funnel`, Meta Ads MCP).
- **`/pisk:linkedin`** — Same for LinkedIn, with Lead Gen Forms and B2B segment analysis (needs `pisk:analytics`, `pisk:funnel`, LinkedIn API).
- **`/pisk:ab`** — Set up and review PostHog A/B tests with a pragmatic low-traffic decision rule (needs `pisk:analytics`, PostHog).

## How it fits together

```
/pisk:launch
      │  (live stack: auth, db, payments, analytics)
      ▼
context skills  ──►  docs/context/*.md          (pisk:growth, funnel, content, analytics)
      │
      ▼
operating loops ──►  read docs/context/, run daily
   pisk:post · pisk:reconcile · pisk:meta · pisk:linkedin · pisk:ab
```

## Recommended order

1. `/pisk:launch`
2. `/pisk:growth`, then `/pisk:funnel`, `/pisk:content`, `/pisk:analytics`
3. Whichever operating loops match your channels — schedule them with `/loop`.

## Status

Skills are planned in OpenSpec under `openspec/changes/` and implemented into `pisk/skills/<name>/SKILL.md`. See `CLAUDE.md` for the development workflow.
