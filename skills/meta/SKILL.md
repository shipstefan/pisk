---
name: pisk:meta
description: Reconcile Meta Ads delivery against real database conversions, detect fake winners, append a journal entry, and act within the configured autonomy level.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:meta` — Meta Ads Reconciliation Loop

You are running the Meta Ads reconciliation loop. Read context and prior journal. Query three sources. Detect fake winners and budget leaks. Act within autonomy level. Append journal entry.

---

## Step 1 — Check Meta Ads MCP

Verify the Meta Ads MCP is connected and available.

**MCP not available:** output:
```
Meta Ads MCP is not connected. To set up:

1. Open Claude Code settings → MCP → Add server
2. Add the Meta Ads MCP (search "meta-ads" in the MCP marketplace)
3. Authenticate with your Meta Business login
4. Request read/write permissions — DECLINE financial-tier access
5. Re-run /pisk:meta

Note: decline financial-tier access to limit the skill's maximum blast radius.
```
Then stop.

---

## Step 2 — Load config

**`docs/ads/config.md` does not exist (first run):**

Ask:
> "First run. Two quick questions:
>
> 1. What's the naming prefix for your Meta campaigns for this project? (e.g. "BM |" or "PS_"). The skill will only touch campaigns whose names start with this prefix.
> 2. What autonomy level do you want?
>    - **A1** — suggest only, no Meta actions
>    - **A2** — pause/resume ads and update ad settings (no budget changes)
>    - **A3** — as A2, plus budget adjustments (always confirm before executing)
>    - **A4** — as A3, plus draft and submit new creatives (start paused)"

After they reply:
- List all Meta campaigns matching the prefix: query Meta Ads MCP for campaigns with names starting with the given prefix.
- Show the list and ask: "Do all these campaigns belong to this project? Confirm or adjust the prefix."
- Once confirmed, create `docs/ads/config.md`:

```markdown
# Meta Ads Config

**Project prefix:** [prefix]
**Autonomy level:** [A1/A2/A3/A4]
**Configured:** [date]
```

**`docs/ads/config.md` exists:** read it. Load `prefix` and `autonomy_level`. Proceed without prompting.

---

## Step 3 — Load prior journal

Read `docs/ads/journal.md` if it exists. Load the last 3 entries. Extract:
- Standing open problems (unresolved across runs)
- Last decisions made
- Last spend levels

If the journal does not exist: note "First run — no prior journal." in this run's entry.

---

## Step 4 — Load context files

Read:
- `docs/context/analytics.md` — data source config, north-star conversion event, attribution fields
- `docs/context/icp.md` — ICP (if exists; used for creative review context at A4)
- `docs/context/offer.md` — offer (if exists; used for creative review context at A4)

If `docs/context/analytics.md` is missing, output: "Run `/pisk:analytics` first." and stop.

---

## Step 5 — Query data sources (scoped to project prefix)

**Safety:** every Meta query MUST be scoped to campaigns matching the project prefix. Never touch campaigns outside the prefix scope.

### Source 1 — Meta Ads MCP
For each ad within prefix-matching campaigns, retrieve:
- Spend (today / most recent complete day)
- Impressions
- Clicks
- CTR
- CPC
- Meta-reported conversions (platform attribution window)

### Source 2 — Product analytics
Using the tool and event names from `analytics.md` Pillar 2, retrieve per-ad conversion funnel counts (using click ID or UTM tracking). If unavailable: mark "unavailable — [reason]" and continue.

### Source 3 — First-party database (source of truth)
Using the north-star conversion event and attribution fields from `analytics.md` Pillar 4, query the database for per-ad activation counts (join on click ID or UTM source). If unavailable: mark "unavailable — note: reconciliation incomplete without source of truth" and continue.

---

## Step 6 — Analyse

### Fake winner detection
For each ad:
- Calculate campaign average CTR.
- Flag any ad with CTR > campaign average AND zero database conversions as **"Fake winner"**.
- Note: absence of database conversions may also reflect attribution lag — use last 7 days of database data, not just today.

If no fake winners: note "No fake winners detected."

### Budget leak identification
Calculate abandonment rate at each funnel stage:
- Click → product analytics event (pre-signup): if > 40%, label "ad/landing-page problem"
- Analytics event → database conversion (post-signup): if > 40%, label "product/onboarding problem"

Identify the single worst stage. This is the budget leak.

### Three-source reconciliation summary
State:
- Database conversions: [N] (source of truth)
- Analytics conversions: [N] (expected undercount — explain why)
- Meta conversions: [N] (expected undercount — explain why)

Expected undercount causes: read from `analytics.md` Pillar 1 and 3. Name the specific causes relevant to this stack.

---

## Step 7 — Determine action and act (by autonomy level)

Identify the single highest-leverage action (pause fake winner, adjust budget on top performer, create new creative for winning audience).

### A1 — Suggest only
State the recommendation in plain text. Take no Meta actions.

### A2 — Settings updates
Execute without additional approval:
- Pause ads flagged as fake winners
- Resume paused ads that now show conversions
- Update ad-level targeting or scheduling settings (not budgets, not creative)

State actions taken.

### A3 — Budget management
Before any budget change, output:
```
Budget change proposal:
- [Ad/Campaign name]: current budget $X → proposed $Y
  Reason: [one line]

Confirm? (yes/no)
```
Wait for explicit "yes" before executing. If "no": note it as declined in the journal.

Also execute all A2 actions without approval.

### A4 — Creative creation
Also execute all A3 actions (with confirmation for budgets). In addition:
May draft and submit new ad creatives. All new creations MUST start in **paused** state. Output:
```
New creative drafted and submitted as [name] — status: PAUSED
Review and activate in Meta Ads Manager when ready.
```

**Safety rails (all levels):**
- ONLY touch objects matching project prefix. Verify name before every action.
- Never affect more than one campaign per action (no bulk operations). Handle each individually.
- New creations always start paused.
- Budget changes always require explicit confirmation (even at A4).

---

## Step 8 — Append journal entry

Append to `docs/ads/journal.md` (create the file if it doesn't exist):

```markdown
---
## [ISO datetime] — Run [N]

### Spend pacing
| Ad | Spend | Impressions | Clicks | CTR | CPC |
|---|---|---|---|---|---|
| [ad name] | $[N] | [N] | [N] | [N]% | $[N] |

### Conversion signals (database = source of truth)
| Ad | Meta conv. | Analytics conv. | DB activations |
|---|---|---|---|
| [ad name] | [N] | [N] | [N] |

### Fake winners
[flag or "None detected"]

### Budget leak
[worst funnel stage, rate, classification (ad problem / product problem)]

### Decisions made this run
[List actions taken, or "None — A1 mode"]

### Open problems (carry-forward)
[Unresolved issues from this and prior runs]

### Prior journal context applied
[Standing problems from last 3 entries that informed this analysis]
---
```

**If journal exceeds 50 entries:** add to the chat summary: "Journal has >50 entries — archive older entries to `docs/ads/journal-archive.md` to keep the file manageable."

---

## Step 9 — Chat summary

After appending the journal entry, output a brief chat summary:

```
## Meta Ads Loop — [DATE]

Campaigns scoped: [prefix] — [N] campaigns, [N] ads

Fake winners: [N flagged / none]
Budget leak: [stage] at [N]%
Action taken: [action or "A1 — none"]

Journal entry appended to docs/ads/journal.md
```

---

## General rules

- Prefix scoping is non-negotiable. Verify every Meta object name before touching it.
- Database is source of truth. Never treat Meta conversion counts as authoritative.
- Budget changes require explicit founder confirmation at every autonomy level.
- New creations start paused — always.
- No bulk operations. One campaign at a time.
- Single highest-leverage action per run. Don't enumerate every possible improvement.
