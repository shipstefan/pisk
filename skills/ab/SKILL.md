---
name: pisk:ab
description: Set up and review PostHog A/B tests with a pragmatic low-traffic decision rule. Two modes — /pisk:ab setup and /pisk:ab review — sharing a persistent experiment journal.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:ab` — A/B Test Setup and Review

Two modes, one journal. At early-stage traffic, don't wait for p<0.05 — ship when you see consistent multi-day uplift.

**Invoke as:**
- `/pisk:ab setup` — set up a new experiment
- `/pisk:ab setup "hypothesis"` — set up with hypothesis pre-supplied
- `/pisk:ab review` — review all running experiments and decide: ship / continue / kill

If no subcommand: ask "Set up a new test or review running tests?"

---

## Startup — always run first

### Load analytics context

Check for `docs/context/analytics.md`.

**Present:** read it. Extract:
- PostHog project key and host URL (Pillar 2)
- Top conversion event names (Pillar 2)
- Consent gate tool and analytics consent category (Pillar 1)

**Absent:** ask the founder inline:
> "No analytics context found. To proceed:
> 1. What's your PostHog project API key?
> 2. What event name fires on your primary conversion (e.g. `trial_signup`)?
>
> Run `/pisk:analytics` after this session to persist this config."

### Load decision threshold

Check for `docs/experiments/config.md`. If present, read `uplift_threshold`. If absent: default to **10%**.

Note to founder: "Decision threshold: [X]%. To customise, add `uplift_threshold: X` to `docs/experiments/config.md`."

### Check PostHog access

Verify PostHog MCP or API is reachable. If not: output setup instructions and stop.

---

# SETUP MODE

## Step S1 — Hypothesis

If provided inline (e.g. `/pisk:ab setup "showing interactive demo increases signups"`): use it directly.

If not: ask:
> "One sentence: what's your hypothesis? (e.g. 'Adding a demo video to the landing page increases trial signups')"

### Surface prior "Next hypothesis"

Check `docs/experiments/journal.md` for the most recent concluded experiment entry (status: "Shipped" or "Killed"). If one exists, surface its "Next hypothesis" line:
> "Last experiment suggested: '[next hypothesis]'. Want to use that, or a different one?"

---

## Step S2 — Experiment parameters

Ask in a single message:
> "Got it. A few more details:
>
> 1. Describe the treatment variant (what changes vs. control)?
> 2. Split ratio — what % sees treatment? (Default: 50%)
> 3. Primary conversion event — which PostHog event are we measuring? [suggest from analytics.md]
> 4. Any secondary events to track? (Optional)
> 5. How do we identify internal accounts to exclude? (Email domain, user property, or IP list — e.g. `@yourcompany.com`)"

---

## Step S3 — Create PostHog feature flag

Create the feature flag in PostHog:
- **Name:** `exp-[kebab-slug-of-hypothesis]-[YYYY-MM]` (e.g. `exp-demo-video-trial-signup-2025-01`)
- **Variants:** `control` (weight = 100 − split%) and `treatment` (weight = split%)
- **Status:** active

Confirm with founder: "Feature flag `[name]` created in PostHog. Proceeding to implementation."

---

## Step S4 — Implementation code (diff + confirm)

Write the server-side variant resolution code. Show a diff of all proposed changes before writing anything:

```
Proposed changes:
1. [file path]: add variant resolution function
2. [file path]: wrap [component/section] in variant check

Confirm to write these changes? (yes/no)
```

Wait for "yes". Then write.

The implementation must include:
- **Consent check:** if user has not granted analytics consent (per Pillar 1 from `analytics.md`), skip experiment assignment entirely — assign neither variant.
- **Server-side resolution:** call PostHog `getFeatureFlagPayload` (or equivalent) on the server, not client, to avoid flicker.
- **First-party cookie bucketing:** store the assigned variant in a first-party cookie (e.g. `__exp_[flag_name]`) so assignment is stable across page loads and subdomain transitions.
- **Internal account filter:** check the identifier from Step S2 (email domain / user property) before assignment. If internal: skip experiment, serve control.
- **Variant render:** show `control` or `treatment` experience based on resolved flag value.

---

## Step S5 — Event tracking validation

After writing code, instruct:
> "Before we go live:
>
> 1. Open your app in a test session (use a non-internal account).
> 2. Complete the action that triggers `[primary_event]`.
> 3. Check PostHog Live Events — confirm `[primary_event]` appears with the experiment flag property `[flag_name]: treatment` or `[flag_name]: control`.
>
> Confirm when you've verified the event fires correctly."

Wait for confirmation. Do not declare the experiment live until tracking is validated.

---

## Step S6 — Write setup journal entry

Append to `docs/experiments/journal.md` (create if absent):

```markdown
---
## [experiment name]

**Hypothesis:** [hypothesis]
**Variant:** [treatment description]
**Split:** [N]% treatment / [100-N]% control
**Primary event:** [event name]
**Secondary events:** [list or "none"]
**Internal filter:** [identifier]
**Started:** [date]
**Status:** Running
---
```

Output:
```
Experiment live ✓

Flag: [flag_name] in PostHog
Watching: [primary_event]
Journal: docs/experiments/journal.md

Run /pisk:ab review in a few days to check the numbers.
```

---

# REVIEW MODE

## Step R1 — Query PostHog

Query PostHog for all feature flags matching the `exp-*` naming convention.

For each matching flag, retrieve:
- Per-variant per-day conversion counts for the primary event (from experiment start date to today)
- Per-variant exposure counts (users bucketed into each variant per day)

### Orphan detection

For each `exp-*` flag found in PostHog, check for a matching active entry in `docs/experiments/journal.md`.

If a flag has no matching active journal entry: flag as orphan:
> "Found orphan flag: `[flag_name]` — no active journal entry. Clean it up? (yes/no)"

If yes: delete the PostHog flag. If no: skip.

---

## Step R2 — Apply decision rule per experiment

For each running experiment (status: "Running" in journal):

### Calculate per-day uplift

For each day the experiment has been running:
```
daily_uplift = (treatment_rate - control_rate) / control_rate × 100%
```
where `rate = conversions / exposures` per variant per day.

### Apply ship rule

**Ship:** treatment shows uplift ≥ [threshold]% on **at least 2 consecutive days** → recommend "Ship treatment".

**Low-traffic caveat:** if total primary conversions across both variants < 100, append:
> "⚠️ Fewer than 100 conversions total — uplift signal is noisy. Consider waiting for more data before shipping."

**Kill:** consistent uplift below 0% (or flat) for >7 days → recommend "Kill treatment".

**Continue:** uplift is positive but inconsistent, or below threshold → recommend "Continue running". State: "Ship trigger: [threshold]% uplift on the primary event for 2+ consecutive days."

---

## Step R3 — Review output

For each experiment, output:

```
### [experiment name]
Hypothesis: [hypothesis]
Running since: [start date] ([N] days)

| Day | Control rate | Treatment rate | Uplift |
|---|---|---|---|
| [date] | [N]% | [N]% | [+/-N]% |
...

Recommendation: [Ship / Continue / Kill]
Reason: [one line]
[Low-traffic caveat if applicable]
```

---

## Step R4 — Execute conclusion (ship or kill)

If recommendation is Ship or Kill, ask: "Confirm [ship/kill]? (yes/no)"

If "no": note in journal as "Recommended [ship/kill], founder deferred. Continue running." Skip conclusion actions.

### Ship conclusion (founder confirms)

1. Roll PostHog flag to 100% treatment (update split to 100% treatment, 0% control).
2. Show diff: remove losing variant code (control path) and the feature flag check (replace with direct treatment render). Require confirmation before writing.
3. Apply changes.
4. Delete the PostHog feature flag (flag is now unnecessary; treatment is 100%).
5. Append conclusion journal entry.

### Kill conclusion (founder confirms)

1. Roll PostHog flag to 100% control.
2. Show diff: remove treatment variant code and feature flag check. Require confirmation.
3. Apply changes.
4. Delete the PostHog feature flag.
5. Append conclusion journal entry.

---

## Step R5 — Append conclusion journal entry

```markdown
---
## [experiment name] — CONCLUDED

**Final uplift:** [N]% over [N] days
**Decision:** [Shipped / Killed]
**Rationale:** [one line]
**Concluded:** [date]
**Status:** [Shipped / Killed]

**Next hypothesis:** [suggest a follow-on test based on what was learned — e.g. "Since demo video lifted trial signups, test adding demo CTA earlier in the funnel"]
---
```

---

## General rules

- Show code diffs and require explicit confirmation before any codebase changes.
- Consent check is mandatory in all variant resolution code. Never assign an experiment to a user without analytics consent.
- Pragmatic rule: 10% consistent uplift over 2+ days = ship. Don't wait for statistical significance.
- Internal accounts must be excluded from experiment assignment.
- Orphan flags must be cleaned up — dead flags accumulate and create maintenance debt.
- PostHog flag naming convention `exp-[slug]-[YYYY-MM]` is mandatory. Deviation breaks orphan detection.
- One treatment variant only (control + one treatment). No multivariate.
- Journal is the single source of truth for experiment state — always read before acting.
