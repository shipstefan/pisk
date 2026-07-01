---
name: pisk:growth
description: Conduct a founding interview and write the docs/context/ business-strategy files that every agent session reads.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:growth` — Growth Context Interview

You are encoding the founder's business context into `docs/context/` so every future agent session starts with full strategic awareness. This is a conversational interview — ask questions as plain messages and wait for typed replies.

**Target files** (eight total, all written to `docs/context/` in the founder's project):
- `business.md` — product, value prop, customer segments, pricing
- `team.md` — founders, roles, decision authority, bandwidth
- `goals-and-targets.md` — north-star metric, growth targets, timeframe
- `bottlenecks.md` — binding constraints and current mitigation
- `vision-and-strategy.md` — long-term vision, operating principles
- `acquisition-strategy.md` — channel hypotheses, prioritisation
- `okrs.md` — current-cycle objectives and key results
- `README.md` — index with sprint frame and cross-links

---

## Step 1 — Check for existing files

Before the interview, check whether any of the eight target files already exist in `docs/context/`.

**If none exist:** proceed directly to the interview.

**If one or more exist:** tell the founder which files already exist and ask:

> "I found existing context files: [list]. How should I proceed?
>
> - **Overwrite** — replace all existing files with fresh interview answers
> - **Skip existing** — only write files that don't exist yet
> - **Abort** — stop; I'll run this manually"

Wait for their choice before continuing.

---

## Step 2 — Founding interview

Conduct seven domain rounds in order. Each round: ask 2–3 focused questions as a single message, wait for the reply, ask one follow-up if the answer is thin (under ~100 words), then move on.

Front-load the three core domains (1–3). Mark domains 4, 5, 6 as skippable by saying "you can type 'skip' to move on."

---

### Round 1 — Business identity

Ask:
> "Let's start with the basics.
>
> 1. What does your product do, in one sentence?
> 2. Who are your current customers — are they individuals, teams, or companies? What's the segment?
> 3. How do you charge — pricing model and current price point(s)?"

After reply: if the value prop is vague (no clear 'who' and 'what outcome'), follow up with: "What specific problem does it solve, and for whom exactly?"

---

### Round 2 — Team

Ask:
> "Tell me about the team.
>
> 1. Who are the founders and what does each person own?
> 2. Who has final say on product decisions? On growth decisions?
> 3. Roughly how much time per week can you each put into growth work right now?"

---

### Round 3 — Goals and targets

Ask:
> "What are you trying to achieve?
>
> 1. What's your north-star metric — the single number that tells you the business is working?
> 2. Where is that metric today, and where do you want it in 90 days?
> 3. Is there a funding milestone or revenue target attached to that 90-day goal?"

---

### Round 4 — Bottlenecks *(skippable)*

Ask:
> "What's slowing you down most right now? (Type 'skip' to move on.)
>
> 1. What's the single biggest constraint on growth — is it traffic, conversion, retention, or something else?
> 2. What are you doing about it today, even if it's not working?"

---

### Round 5 — Vision and strategy *(skippable)*

Ask:
> "Zoom out for a moment. (Type 'skip' to move on.)
>
> 1. What does the business look like in 3 years if things go well?
> 2. What are the 2–3 principles you use to make hard decisions about what to build or who to sell to?"

---

### Round 6 — Acquisition strategy *(skippable)*

Ask:
> "How do you plan to grow? (Type 'skip' to move on.)
>
> 1. Which acquisition channels are you betting on — content, paid ads, outbound, partnerships, PLG?
> 2. Which channel is your highest-confidence bet and why?
> 3. Which channels have you ruled out?"

---

### Round 7 — OKRs

Ask:
> "What are your OKRs for this quarter?
>
> List your objectives (what you're trying to achieve) and the key results (measurable outcomes) for each. If you don't have formal OKRs, describe the top 2–3 things you're optimising for this cycle."

---

## Step 3 — Write context files

After all rounds are complete (skipped domains get placeholder files), write all eight files to `docs/context/`. Create the directory if it doesn't exist.

Synthesize the founder's answers — don't transcribe them verbatim. Write in third-person declarative prose as if briefing an agent ("The company sells X to Y at $Z/mo").

Each file must include relative markdown links to related files at the bottom under `## Related`.

---

### `business.md`

```markdown
# Business

## Product
[One-sentence description from Round 1]

## Value proposition
[What problem it solves, for whom, and what outcome it delivers]

## Customer segments
[Segments from Round 1 — individual/team/company, description]

## Pricing
[Model and price points from Round 1]

## Related
- [Goals and targets](goals-and-targets.md)
- [Acquisition strategy](acquisition-strategy.md)
- [ICP and offer](icp.md) *(if pisk:funnel has been run)*
```

---

### `team.md`

```markdown
# Team

## Founders
[Names, roles, and ownership areas from Round 2]

## Decision authority
[Who decides what — product vs. growth vs. technical]

## Growth bandwidth
[Hours/week each founder can put into growth work]

## Related
- [Goals and targets](goals-and-targets.md)
- [OKRs](okrs.md)
```

---

### `goals-and-targets.md`

```markdown
# Goals and targets

## North-star metric
[Metric name and what it measures]

## Current value
[Today's reading]

## 90-day target
[Target value and date]

## Milestone context
[Funding, revenue, or other milestone if mentioned]

## Related
- [OKRs](okrs.md)
- [Bottlenecks](bottlenecks.md)
- [Business](business.md)
```

---

### `bottlenecks.md`

If skipped, write:
```markdown
# Bottlenecks

*Not yet captured. Re-run `/pisk:growth` to add this section.*

## Related
- [Goals and targets](goals-and-targets.md)
- [Vision and strategy](vision-and-strategy.md)
```

Otherwise:
```markdown
# Bottlenecks

## Primary constraint
[Binding constraint from Round 4 — traffic / conversion / retention / other]

## Current mitigation
[What's being tried today]

## Related
- [Goals and targets](goals-and-targets.md)
- [Acquisition strategy](acquisition-strategy.md)
```

---

### `vision-and-strategy.md`

If skipped, write placeholder. Otherwise:
```markdown
# Vision and strategy

## 3-year vision
[What the business looks like if things go well]

## Operating principles
[2–3 decision-making rules from Round 5]

## Related
- [Business](business.md)
- [Acquisition strategy](acquisition-strategy.md)
```

---

### `acquisition-strategy.md`

If skipped, write placeholder. Otherwise:
```markdown
# Acquisition strategy

## Channels in play
[List of channels being pursued from Round 6]

## Highest-confidence bet
[Channel and rationale]

## Ruled out
[Channels explicitly de-prioritised and why]

## Related
- [Business](business.md)
- [Goals and targets](goals-and-targets.md)
- [ICP and offer](icp.md) *(if pisk:funnel has been run)*
```

---

### `okrs.md`

```markdown
# OKRs — [current quarter, e.g. Q3 2025]

## Objective 1: [title]
- KR1: [measurable result]
- KR2: [measurable result]

## Objective 2: [title]
- KR1: [measurable result]

*(Add more objectives as needed)*

## Related
- [Goals and targets](goals-and-targets.md)
- [Bottlenecks](bottlenecks.md)
```

---

### `README.md`

```markdown
# docs/context/

Business and growth context for AI agents. Updated by the `pisk:*` context skills.

## Sprint frame

**North-star metric:** [metric] — currently [value], target [target] by [date]
**Primary bottleneck:** [bottleneck or "not yet captured"]
**Top OKR this cycle:** [objective 1 title]

## Files

| File | Contents |
|---|---|
| [business.md](business.md) | Product, value prop, segments, pricing |
| [team.md](team.md) | Founders, roles, decision authority, bandwidth |
| [goals-and-targets.md](goals-and-targets.md) | North-star metric and 90-day targets |
| [bottlenecks.md](bottlenecks.md) | Binding constraints and mitigations |
| [vision-and-strategy.md](vision-and-strategy.md) | 3-year vision and operating principles |
| [acquisition-strategy.md](acquisition-strategy.md) | Channel bets and prioritisation |
| [okrs.md](okrs.md) | Current-cycle objectives and key results |

## Related skills

- `/pisk:funnel` — adds `icp.md` and `offer.md`
- `/pisk:content` — adds `content/` subfolder
- `/pisk:analytics` — adds `analytics.md`
```

---

## Step 4 — Success summary

After writing all files, output:

```
## Growth context written ✓

Created in docs/context/:
- business.md
- team.md
- goals-and-targets.md
- bottlenecks.md  [or: placeholder — run /pisk:growth to fill in]
- vision-and-strategy.md  [or: placeholder]
- acquisition-strategy.md  [or: placeholder]
- okrs.md
- README.md

Your agents now have growth context. In any new session, reference
docs/context/ in your prompt and they'll start with full strategic awareness.

Next: run /pisk:funnel to add your ICP and offer, then /pisk:content and
/pisk:analytics to complete the context layer.
```

---

## General rules

- Write files to the **founder's project repo**, not this skills repo.
- Ask one domain per message — don't combine rounds.
- Follow-up once if an answer is thin; don't interrogate.
- Write files only after all rounds are complete (or skipped).
- Skipped domains get placeholder files — never leave a file missing entirely.
- Do not add formatting or sections not in the templates above.
