---
name: pisk:linkedin
description: Reconcile LinkedIn Ads delivery (clicks + Lead Gen Forms) against real database activations, analyse segment quality, append a journal entry, and act within autonomy level.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:linkedin` — LinkedIn Ads Reconciliation Loop

You are running the LinkedIn Ads reconciliation loop. This is B2B territory: four signals instead of three, segment quality matters as much as creative, and CPL is expected to be 3–5× higher than Meta. Read context and prior journal. Query sources. Analyse four-signal funnel and segment table. Act within autonomy level. Append journal entry.

---

## Step 1 — Check LinkedIn Campaign Manager API

Verify LinkedIn Campaign Manager API access is available (via MCP or configured credentials).

**Not available:** output:
```
LinkedIn Campaign Manager API is not connected. To set up:

1. Obtain a LinkedIn Marketing Developer Platform app (ads_read + ads_management permissions)
   or install a LinkedIn Ads MCP server in Claude Code settings.
2. Authenticate with your LinkedIn account that has Campaign Manager access.
3. Note your Ad Account ID — you'll need it on first run.
4. Re-run /pisk:linkedin
```
Then stop.

---

## Step 2 — Load config

**`docs/linkedin-ads/config.md` does not exist (first run):**

Ask:
> "First run. Two quick questions:
>
> 1. What's the naming prefix for your LinkedIn campaigns for this project? (e.g. "PS |" or "LI_"). Only campaigns starting with this prefix will be touched.
> 2. What autonomy level?
>    - **A1** — suggest only, no LinkedIn actions
>    - **A2** — pause/resume campaigns, update targeting exclusions (no budget changes)
>    - **A3** — as A2, plus budget adjustments (always confirm before executing)
>    - **A4** — as A3, plus draft new sponsored content or audience segments (start paused)"

After reply:
- Query LinkedIn Campaign Manager for campaigns matching the prefix.
- Show the list: "Do all these campaigns belong to this project? Confirm or adjust the prefix."
- Once confirmed, create `docs/linkedin-ads/config.md`:

```markdown
# LinkedIn Ads Config

**Project prefix:** [prefix]
**Autonomy level:** [A1/A2/A3/A4]
**Configured:** [date]
```

**`docs/linkedin-ads/config.md` exists:** read it. Load prefix and autonomy level. Proceed.

---

## Step 3 — Load prior journal

Read `docs/linkedin-ads/journal.md` if it exists. Load last 3 entries. Extract standing open problems and last decisions.

If no journal: note "First run — no prior journal."

---

## Step 4 — Load context files

Read:
- `docs/context/analytics.md` — data source config, north-star conversion event, LGF-to-DB matching field (Pillar 4)
- `docs/context/icp.md` — target audience profile, used for segment quality interpretation (if exists)
- `docs/context/offer.md` — offer type, used for CPL context benchmarking (if exists)

If `docs/context/analytics.md` is missing: output "Run `/pisk:analytics` first." and stop.

---

## Step 5 — Query data sources (scoped to project prefix)

**Safety:** every LinkedIn query and action MUST be scoped to campaigns matching the project prefix. Verify name before touching anything.

### Source 1 — LinkedIn Campaign Manager (per campaign, then per ad)

For each campaign matching prefix, retrieve:
- Spend (today / most recent complete day)
- Impressions, clicks, CTR, CPL
- Lead Gen Form submissions (if LGF is configured on the campaign)
- Segment breakdown (where available): conversion rate by job title, seniority, company size, or industry

**Note on segment suppression:** LinkedIn suppresses segment data for audiences below its privacy threshold. If breakdown is unavailable, note "Segment data suppressed (audience below LinkedIn threshold)" and proceed.

### Source 2 — Product analytics (from `analytics.md` Pillar 2)

Using the tool and event names from `analytics.md`, retrieve per-campaign product signup counts, filtering by `utm_source=linkedin` or `li_fat_id` presence. If unavailable: mark "unavailable — [reason]" and continue.

### Source 3 — First-party database (source of truth)

Using the north-star conversion event and attribution fields from `analytics.md` Pillar 4, retrieve per-campaign activation counts.

**LGF matching (if LGFs are in use):**
Query the database for records where the email matches Lead Gen Form submissions. Report aggregate totals only:
- Total LGF submissions for the period: N
- Matched to DB records: N
- Unmatched: N

**IMPORTANT: Never log individual email addresses in the journal.** Aggregate counts only.

If database unavailable: mark "unavailable — reconciliation incomplete without source of truth."

---

## Step 6 — Analyse

### Four-signal funnel (per campaign)

Compute drop-off at each transition:
1. **Click-to-LGF rate** = LGF submissions / clicks (or "N/A — no LGF")
2. **LGF-to-signup rate** = product signups / LGF submissions
3. **Signup-to-activation rate** = DB activations / product signups

Flag any drop-off above 60% as a budget leak. Label whether it's addressable pre-platform (ad/LGF problem) or post-signup (product/onboarding problem).

### Fake winner detection

**Click-based:** CTR above campaign average AND zero DB activations → "Fake winner (clicks)"

**LGF-based:** LGF completions above campaign average AND zero DB activations → "Fake winner (LGF)" — this often means the form attracts non-ICP leads. Cross-reference with ICP from `docs/context/icp.md`.

### Segment quality table

When segment data is available, rank segments by DB activation rate:

| Segment | Clicks | LGF | Activations | Activation rate |
|---|---|---|---|---|
| [segment] | N | N | N | X% |

Flag segments with activation rate more than 50% below average as candidates for exclusion. Cross-reference against `docs/context/icp.md` to confirm misalignment with ICP.

### CPL context note

Include in journal header. If `docs/context/offer.md` is available, use the pricing tier to estimate expected CPL range. Otherwise use generic benchmark:

> "LinkedIn B2B CPL benchmark (SaaS trial): €50–€200. Lead Gen Form CPL often 20–40% lower than landing page CPL for same audience."

---

## Step 7 — Determine action and act (by autonomy level)

Single highest-leverage action: pause fake winner, shift budget to top-performing segment, add targeting exclusion for worst segment.

### A1 — Suggest only
State recommendation. No LinkedIn actions.

### A2 — Settings updates
Execute without approval:
- Pause campaigns flagged as fake winners
- Resume paused campaigns that now show activations
- Update targeting exclusions to add underperforming segments

### A3 — Budget management
Before any budget change:
```
Budget change proposal:
- [Campaign name]: current budget $X → proposed $Y
  Reason: [one line]

Confirm? (yes/no)
```
Wait for "yes". If "no": note declined in journal. Also execute A2 actions.

### A4 — Creative / audience creation
Also execute A3 (with confirmation for budgets). May additionally:
- Draft new sponsored content targeting highest-performing segment
- Create new matched audience or exclusion list

All new objects start **paused**. Output:
```
New [object type] drafted: [name] — status: PAUSED
Review and activate in LinkedIn Campaign Manager when ready.
```

**Safety rails (all levels):**
- ONLY touch campaigns matching project prefix. Verify before every action.
- No bulk operations. One campaign at a time with explicit handling.
- New creations always start paused.
- Budget changes always require explicit confirmation.

---

## Step 8 — Append journal entry

Append to `docs/linkedin-ads/journal.md` (create if absent):

```markdown
---
## [ISO datetime] — Run [N]

### CPL Context
[CPL benchmark note — one line]

### Spend pacing
| Campaign | Spend | Impressions | Clicks | CTR | CPL | LGF submissions |
|---|---|---|---|---|---|---|
| [name] | $[N] | [N] | [N] | [N]% | $[N] | [N] |

### Four-signal funnel
| Campaign | Click→LGF | LGF→Signup | Signup→Activation | DB activations |
|---|---|---|---|---|
| [name] | [N]% | [N]% | [N]% | [N] |

### LGF matching (aggregate)
Total LGF submissions: [N] | Matched to DB: [N] | Unmatched: [N]

### Fake winners
[Flags or "None detected"]

### Segment quality
[Table if available, or "Suppressed — audience below LinkedIn threshold"]

### Decisions made this run
[Actions taken or "None — A1 mode"]

### Open problems (carry-forward)
[Unresolved issues from this and prior runs]

### Prior journal context applied
[Standing problems from last 3 entries that informed this analysis]
---
```

**If journal exceeds 50 entries:** add to chat summary: "Journal has >50 entries — archive older entries to `docs/linkedin-ads/journal-archive.md`."

---

## Step 9 — Chat summary

```
## LinkedIn Ads Loop — [DATE]

Campaigns scoped: [prefix] — [N] campaigns

Four-signal funnel:
  Click → LGF: [avg rate]%
  LGF → Signup: [avg rate]%
  Signup → Activation: [avg rate]%

Fake winners: [N flagged / none]
Worst segment: [segment name] at [N]% activation rate
Action taken: [action or "A1 — none"]

Journal entry appended to docs/linkedin-ads/journal.md
```

---

## General rules

- Prefix scoping is non-negotiable. Verify every LinkedIn object name before touching it.
- Database is source of truth. Never treat LinkedIn conversion counts as authoritative.
- LGF email matching: aggregate counts only, no individual PII in journal.
- Budget changes require explicit founder confirmation at every autonomy level.
- New creations start paused — always.
- No bulk operations. One campaign at a time.
- Segment data suppression is normal for small audiences — note it and continue.
- LinkedIn CPL is 3–5× Meta CPL by design. Do not flag high CPL as a problem without context.
