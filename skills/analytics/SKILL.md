---
name: pisk:analytics
description: Interview the founder about their measurement stack, then write docs/context/analytics.md — the keystone context file read by every operating loop.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:analytics` — Analytics Context Interview

You are encoding the founder's measurement stack into `docs/context/analytics.md` — a single file that captures all four analytics pillars (consent, behavior, ad signals, database attribution) so every operating loop — `pisk:reconcile`, `pisk:meta`, `pisk:linkedin`, `pisk:ab` — starts with full context pre-loaded.

**Target file:** `docs/context/analytics.md` in the founder's project.

---

## Step 1 — Check for existing file

Check whether `docs/context/analytics.md` already exists.

**Does not exist (first run):** proceed directly to the full four-pillar interview.

**Exists (re-run):** ask:
> "I found an existing analytics context file. Which pillar has changed?
>
> - **Consent** — your consent tool or gate logic has changed
> - **Behavior** — you've changed analytics tools or added/removed conversion events
> - **Ad platforms** — you've added a new platform, changed pixels, or updated CAPI config
> - **Database** — attribution schema or north-star event has changed
> - **All** — re-interview everything
>
> (Name one or more)"

Interview only the selected pillar(s). Merge updated sections back into the existing file.

---

## Step 2 — Analytics interview

Four pillar rounds. One round per message. Wait for reply. One follow-up if answer is thin.

Record the exact tool names the founder gives — never substitute assumed defaults.

---

### Pillar 1 — Consent *(skippable: type 'skip')*

Ask:
> "Let's capture your consent setup. (Type 'skip' to move on.)
>
> 1. Which consent management tool are you using? (e.g. iubenda, Cookiebot, Cookie Script, or custom)
> 2. Which tracking categories require user consent before firing — marketing pixels, analytics, functional cookies?
> 3. How do consent decisions get communicated to other tools — browser events, callbacks, a dataLayer push?"

Follow-up if vague: "What happens to PostHog / your analytics tool when a user declines marketing consent — is it blocked entirely, or does it fire without personal data?"

---

### Pillar 2 — Behavior measurement *(skippable: type 'skip')*

Ask:
> "Now your behavior analytics setup. (Type 'skip' to move on.)
>
> 1. Which tool do you use for product analytics? (e.g. PostHog, Amplitude, Mixpanel, GA4) Is it self-hosted or cloud? Are requests proxied through your own domain?
> 2. What are your top 5 conversion events — the events that tell you the product is working? Please give the exact event names as they're tracked.
> 3. Do you have session replays enabled? If so, which consent gate controls them?"

---

### Pillar 3 — Ad platform signals *(skippable: type 'skip')*

Ask:
> "Let's capture your ad tracking setup. (Type 'skip' to move on.)
>
> 1. Which ad platforms are you running — Meta, Google, TikTok, LinkedIn, other? Which have a tracking pixel installed?
> 2. For each platform: do you have a server-side signal configured (Meta Conversions API, Google Enhanced Conversions, TikTok Events API)? Or is it pixel-only?
> 3. How do click IDs (fbclid, gclid, ttclid, li_fat_id) get captured and carried through to the signup flow? Are they stored in the database?"

---

### Pillar 4 — First-party database attribution *(skippable: type 'skip')*

Ask:
> "Finally, your first-party attribution. (Type 'skip' to move on.)
>
> 1. Which database fields store attribution data — UTM parameters, click IDs, referrer? What are the exact field names?
> 2. What is your north-star conversion event — the single database action that confirms a real paying or activated user? (e.g. `subscription_created`, `activation_completed`)
> 3. Is attribution stored at the user record level, the event level, or both?"

---

## Step 3 — Write `docs/context/analytics.md`

After all pillars are answered (or skipped), write (or update) `docs/context/analytics.md`. Create `docs/context/` if it doesn't exist.

Synthesize — use the founder's exact tool names. Write in third-person declarative prose as if briefing a reconciliation agent.

```markdown
# Analytics Context

Last updated: [today's date]

## Pillar 1 — Consent

**Tool:** [name from Pillar 1, or "Not configured — skip this pillar"]
**Consent categories gated:**
- Marketing: [yes/no — which tools are blocked]
- Analytics: [yes/no — which tools are blocked]
- Functional: [yes/no]

**How decisions broadcast:** [browser event / callback / dataLayer push — exact mechanism]

---

## Pillar 2 — Behavior Measurement

**Tool:** [name from Pillar 2]
**Deployment:** [cloud / self-hosted / proxied — domain if proxied]

**Top conversion events:**
| Event name | What it means |
|---|---|
| [event_name] | [plain description] |
| [event_name] | [plain description] |
| [event_name] | [plain description] |
| [event_name] | [plain description] |
| [event_name] | [plain description] |

**Session replays:** [enabled/disabled — consent gate if enabled]

---

## Pillar 3 — Ad Platform Signals

| Platform | Pixel | Server-side signal | Click ID field |
|---|---|---|---|
| [Meta] | [yes/no] | [CAPI / not configured] | [fbclid] |
| [Google] | [yes/no] | [Enhanced Conversions / not configured] | [gclid] |
| [TikTok] | [yes/no] | [Events API / not configured] | [ttclid] |

**Click ID capture:** [how click IDs flow from landing page → signup → database]

---

## Pillar 4 — First-Party Database Attribution

**Attribution fields:**
| Field | Contents |
|---|---|
| [field_name] | UTM source |
| [field_name] | UTM medium |
| [field_name] | UTM campaign |
| [field_name] | Click ID (fbclid / gclid / etc.) |
| [field_name] | Referrer URL |

**Attribution level:** [user record / event / both]

**North-star conversion event:** `[event_name]` — [plain description of what it represents]
```

For any skipped pillar, replace the section body with:
```
*Not yet configured. Re-run `/pisk:analytics` and select this pillar to fill in.*
```

---

## Step 4 — Success summary

After writing the file:

```
## Analytics context written ✓

File: docs/context/analytics.md

Pillars configured:
✓ Consent — [tool name]
✓ Behavior — [tool name], [N] conversion events
✓ Ad platforms — [platform list]
✓ Database — north-star: [event name]
[or ✗ [pillar] — not yet configured]

Your analytics context is ready. Run /pisk:reconcile to generate a
daily reconciliation report across all four sources.

Tip: re-run /pisk:analytics after adding a new ad platform, changing
attribution fields, or updating conversion event names.
```

---

## General rules

- Write to the **founder's project repo**, not this skills repo.
- Record tool names exactly as the founder states them — never assume PostHog, iubenda, or any default.
- `analytics.md` is the source of truth for all operating loops. Be precise; vague entries propagate to every downstream skill.
- Skipped pillars get placeholder sections — never omit a section header entirely.
- On re-run, merge updated pillar sections into the existing file. Leave unchanged sections as-is.
- The north-star conversion event (Pillar 4) is the single most critical field — if the founder is vague, ask a clarifying follow-up before writing.
