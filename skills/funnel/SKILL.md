---
name: pisk:funnel
description: Interview the founder about their ICP and offer, then write docs/context/icp.md and docs/context/offer.md for downstream funnel agents.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:funnel` — Funnel Context Interview

You are encoding the founder's ideal customer profile and offer into two canonical files so every future agent session — landing pages, ad copy, lead magnets, email sequences — starts with that context pre-loaded.

**Target files** (written to `docs/context/` in the founder's project):
- `icp.md` — ideal customer persona, primary pain, transformation sought
- `offer.md` — product description, pricing tiers, positioning vs. alternatives

---

## Step 1 — Check for existing files

Check whether `docs/context/icp.md` or `docs/context/offer.md` already exist.

**Neither exists (first run):** proceed directly to the full two-domain interview.

**One or both exist (re-run):** ask:
> "I found existing funnel context. Which domain has changed?
>
> - **ICP** — your ideal customer has shifted
> - **Offer** — pricing or positioning has changed
> - **Both** — interview me on both again
> - **Neither** — just show me the current files"

Interview only the selected domain(s). Skip unchanged ones.

---

## Step 2 — Funnel interview

Two rounds. Ask each round as a single message. Wait for reply. Follow-up once if the answer is under ~80 words or lacks specificity.

---

### Round 1 — ICP *(skippable: type 'skip')*

Ask:
> "Let's define your ideal customer. (Type 'skip' to move on.)
>
> 1. Who is your best customer right now — what's their job title, company type, or life situation?
> 2. What's the specific frustration or problem that brings them to your product? What were they struggling with before they found you?
> 3. What's the transformation they're looking for — what does life look like after the problem is solved?"

Follow-up if thin: "Can you give me a specific example — describe the last customer who got real value from the product?"

---

### Round 2 — Offer *(skippable: type 'skip')*

Ask:
> "Now let's define your offer. (Type 'skip' to move on.)
>
> 1. In one sentence: what does your product do and for whom?
> 2. What are your pricing tiers and what does each include? (If you have a free plan, include it.)
> 3. How do you position yourself against the alternatives your customers consider — what do you win on?"

Follow-up if positioning is vague: "When a prospect picks you over [alternative], what's the reason they give?"

---

## Step 3 — Write context files

After both rounds (or whichever were answered), write the files to `docs/context/`. Create the directory if it doesn't exist.

Synthesize — don't transcribe. Write in third-person declarative prose as if briefing a conversion copywriter.

---

### `icp.md`

```markdown
# Ideal Customer Profile

## Persona
[Who the ideal customer is — job title, company type, or life context from Round 1]

## Primary pain
[Specific frustration they experience before finding the product]

## Transformation sought
[The outcome they want — what life looks like after the problem is solved]

## Example customer
[Concrete example if the founder gave one; otherwise omit this section]

## Related
- [Offer](offer.md)
- [Business](business.md) *(if pisk:growth has been run)*
- [Acquisition strategy](acquisition-strategy.md) *(if pisk:growth has been run)*
```

If skipped, write:
```markdown
# Ideal Customer Profile

*Not yet captured. Re-run `/pisk:funnel` to fill in.*

## Related
- [Offer](offer.md)
```

---

### `offer.md`

```markdown
# Offer

## What it does
[One-sentence product description for whom from Round 2]

## Pricing tiers

| Tier | Price | What's included |
|---|---|---|
| [tier name] | [price] | [inclusions] |
| [tier name] | [price] | [inclusions] |

*(Add rows as needed)*

## Positioning
[How the product wins vs. alternatives — what founders say when asked why a customer chose them]

## Related
- [ICP](icp.md)
- [Business](business.md) *(if pisk:growth has been run)*
```

If skipped, write:
```markdown
# Offer

*Not yet captured. Re-run `/pisk:funnel` to fill in.*

## Related
- [ICP](icp.md)
```

---

## Step 4 — Success summary

After writing files, output:

```
## Funnel context written ✓

Created in docs/context/:
- icp.md — [persona summary in one line]
- offer.md — [pricing model in one line]

Your funnel context is ready. Agents now read icp.md and offer.md automatically
when building landing pages, lead magnets, and email sequences.

Tip: pricing changes frequently. Re-run /pisk:funnel when you update tiers.

Next: /pisk:content to encode your content voice and cadence.
```

---

## General rules

- Write files to the **founder's project repo**, not this skills repo.
- ICP is optimized for conversion copy — focus on emotional pain and transformation, not demographic segments (that's `business.md`).
- Offer is optimized for sales context — pricing tiers and differentiation, not product roadmap.
- Skipped domains get placeholder files — never leave a file missing entirely.
- On re-run, only overwrite the files for interviewed domains; leave unchanged files as-is.
