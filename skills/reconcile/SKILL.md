---
name: pisk:reconcile
description: Daily whole-funnel reconciliation report — query ad platforms, analytics, and the database, then surface one action. Database is source of truth.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:reconcile` — Daily Funnel Reconciliation

You are producing a daily reconciliation report across all configured data sources. Read the analytics context. Query each source. Surface discrepancies. Recommend one action.

Output is a chat message. No files written.

---

## Step 1 — Check analytics context

Check whether `docs/context/analytics.md` exists.

**Missing:** output exactly:
```
Run /pisk:analytics first to set up your analytics context.
```
Then stop.

**Present:** read the file. Note its last-modified date for the report header. Load:
- Which ad platforms are configured (Pillar 3)
- Which behavior analytics tool and top conversion events (Pillar 2)
- Database attribution fields and north-star conversion event (Pillar 4)
- Consent tool and gating rules (Pillar 1)

---

## Step 2 — Retrieve data

Query each configured source for today's numbers (or the most recent complete day if today's data is not yet available). Target metrics: **spend, impressions, clicks, conversions**.

Use whatever data access is available in the project: MCP tools, API calls via bash, SQL queries through a database MCP, or manual data input from the founder.

**For each configured source:**

### Ad platforms (from Pillar 3)
For each platform with a pixel or server-side signal configured, retrieve:
- Total spend (today)
- Impressions (today)
- Clicks (today)
- Reported conversions (today, using the platform's reported attribution window)

Use the platform's API (via MCP or authenticated CLI) if available. If unavailable, mark the source as "unavailable" and note which tool or credential is needed.

### Behavior analytics tool (from Pillar 2)
Retrieve:
- Sessions or pageviews (today)
- Each of the top 5 conversion events and their counts (today)

Use the configured tool's MCP or API. If unavailable, mark as "unavailable."

### First-party database (from Pillar 4)
Query the database for today's north-star conversion event count using the exact field names from Pillar 4. This is the source of truth.

Example pattern (adapt to actual schema):
```sql
SELECT COUNT(*) as activations
FROM users
WHERE created_at >= CURRENT_DATE
  AND [north_star_event_field] IS NOT NULL;
```

If database is unavailable, note it explicitly — the report cannot be authoritative without the source-of-truth row.

---

## Step 3 — Produce the report

Output the following report as a single chat message.

---

```
## Daily Reconciliation — [DATE]

Analytics context: docs/context/analytics.md (last updated: [LAST_MODIFIED_DATE])
If your analytics stack has changed since this date, re-run /pisk:analytics.

---

### Funnel table

| Source | Spend | Impressions | Clicks | Conversions |
|---|---|---|---|---|
| [DB name] (source of truth) | — | — | — | [N] |
| [Behavior tool] (expected undercount) | — | — | [N] | [N] |
| [Platform 1] (expected undercount) | $[N] | [N] | [N] | [N] |
| [Platform 2] (expected undercount) | $[N] | [N] | [N] | [N] |

*(Mark any unavailable source with "unavailable — [reason]")*

---

### Reconciliation

**Source of truth:** The database shows [N] [north-star event]s today.
**Ad platform total:** [N] reported conversions across all platforms.
**Behavior tool:** [N] [top event] events recorded.

Expected gap causes (relevant to this stack):
- [Cause 1 — e.g. "Ad blockers bypass [behavior tool] even with proxy; estimated X% of traffic affected"]
- [Cause 2 — e.g. "Consent gate blocks [behavior tool] for ~Y% of EU visitors (analytics category)"]
- [Cause 3 if applicable — e.g. "Meta attribution window (7-day click) counts conversions from prior days"]

Consent coverage note: If the consent pillar shows analytics tracking is gated, note the estimated % of visitors who granted consent, so funnel drop-offs can be contextualized.

---

### Worst funnel stage

[Identify the stage with the worst conversion rate:]
- Click-to-trial: [X]%
- Trial-to-activation: [X]%
- [Other stages as applicable]

Worst: [stage name] at [X]%

---

### Recommended action

[Exactly one action. Name the specific funnel stage, the current rate, and what to do.]

Example: "Click-to-trial rate is 1.2% vs. 3% target — review landing-page copy against docs/context/icp.md and tighten the headline."
```

---

## General rules

- Never write files. Report is chat-only.
- Database is always source of truth. Never treat platform conversion counts as authoritative.
- Mark unavailable sources rather than halting. The report is useful even with partial data.
- One recommended action only. Pick the worst funnel stage; state the action plainly.
- Do not make spend decisions in this report — that's `pisk:meta` and `pisk:linkedin`.
- Discrepancy section must name at least two undercount causes specific to the founder's configured stack (read from `analytics.md`), not generic disclaimers.
- If consent gates analytics tracking, include a consent coverage estimate — a flat event count without this context is misleading.
