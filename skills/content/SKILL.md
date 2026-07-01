---
name: pisk:content
description: Interview the founder about their content strategy, then write the four docs/context/content/ files that pisk:post reads on every production run.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:content` — Content Context Interview

You are encoding the founder's content strategy into four canonical files so the `pisk:post` production skill can draft on-brand content every day without re-briefing.

**Target files** (written to `docs/context/content/` in the founder's project):
- `story.md` — audience persona, pain points, transformation narrative
- `themes-and-topics.md` — weekly theme map (day → theme → topic types)
- `voice-and-formats.md` — voice rules, channel specs, visual style guide
- `cadence.md` — publishing rhythm, autonomy level (L1–L4), review expectations

---

## Step 1 — Check for existing files

Check whether any of the four canonical files already exist in `docs/context/content/`.

**None exist (first run):** proceed to the full four-domain interview.

**One or more exist (re-run):** ask:
> "I found existing content context. Which domains have changed?
>
> - **Story** — your audience or transformation narrative has shifted
> - **Themes** — you want to remap your weekly themes
> - **Voice** — voice rules or channel specs need updating
> - **Cadence** — publishing rhythm or autonomy level has changed
> - **All** — re-interview everything
>
> (You can name multiple, e.g. 'voice and cadence')"

Interview only the selected domains. Leave unchanged files as-is.

---

## Step 2 — Content interview

Four rounds. Ask each round as a single message. Wait for reply. One follow-up if answer is under ~80 words.

---

### Round 1 — Story *(skippable: type 'skip')*

Ask:
> "Let's define your content audience. (Type 'skip' to move on.)
>
> 1. Who do you write for — describe the person reading your content (job, situation, mindset)?
> 2. What's the core frustration or challenge that brings them to you?
> 3. What transformation are you helping them achieve through your content?"

Follow-up if thin: "Give me a specific example — describe one reader who got value from something you published."

---

### Round 2 — Themes and topics *(skippable: type 'skip')*

Ask:
> "Now let's map your weekly content themes. (Type 'skip' to move on.)
>
> 1. Do you publish every day or on specific days? Which days?
> 2. For each day you publish, what's the theme or type of content — for example 'Monday is tactical how-to, Wednesday is founder story, Friday is industry opinion'?
> 3. Are there topic areas you deliberately avoid?"

Follow-up if vague: "For [most important theme], what are 2–3 specific topic types you return to repeatedly?"

---

### Round 3 — Voice and formats *(skippable: type 'skip')*

Ask:
> "Let's capture your voice and format preferences. (Type 'skip' to move on.)
>
> 1. How would you describe your writing voice in 3–5 adjectives? (e.g. direct, irreverent, data-driven)
> 2. What does your voice explicitly avoid — what would make you say 'that doesn't sound like me'?
> 3. Which channels do you publish on (LinkedIn, X/Twitter, newsletter, blog)? For each, what's the preferred format — thread, short-form post, long-form essay, etc.?
> 4. Any visual style guidance — preferred image style, no-stock-photo rules, color palette?"

---

### Round 4 — Cadence *(skippable: type 'skip')*

Ask:
> "Finally, let's set your publishing cadence and how much autonomy you want to give the agent.
>
> 1. How many pieces do you aim to publish per week?
> 2. What's your review process — do you review every draft before it goes out, or are you comfortable with the agent publishing directly?
> 3. Choose your autonomy level:
>    - **L1** — you review and approve every piece before it's published
>    - **L2** — agent drafts; you do a quick edit before publishing
>    - **L3** — agent drafts and queues; you review weekly and can roll back
>    - **L4** — agent publishes autonomously; you review weekly
>
> Which level fits your workflow right now?"

---

## Step 3 — Write context files

After all rounds (or whichever were answered), write the files to `docs/context/content/`. Create the directory if it doesn't exist.

Synthesize — don't transcribe. Write in third-person declarative prose as if briefing a content agent.

---

### `story.md`

```markdown
# Content Story

## Audience persona
[Who the content is written for — job, situation, mindset from Round 1]

## Core frustration
[The challenge or pain that brings them to the founder's content]

## Transformation
[What the audience achieves through this content — the outcome they're working toward]

## Example reader
[Concrete example if given; omit section if not]

## Related
- [Themes and topics](themes-and-topics.md)
- [ICP](../icp.md) *(if pisk:funnel has been run)*
```

If skipped:
```markdown
# Content Story

*Not yet captured. Re-run `/pisk:content` to fill in.*
```

---

### `themes-and-topics.md`

```markdown
# Weekly Themes and Topics

## Publishing days
[Which days content is published]

## Theme map

| Day | Theme | Topic types |
|---|---|---|
| Monday | [theme] | [type 1, type 2] |
| Wednesday | [theme] | [type 1, type 2] |
| Friday | [theme] | [type 1, type 2] |

*(Add or remove rows to match publishing days)*

## Off-limits topics
[Topics or areas deliberately avoided; omit if none given]

## Related
- [Story](story.md)
- [Voice and formats](voice-and-formats.md)
- [Cadence](cadence.md)
```

If skipped: placeholder noting "Re-run `/pisk:content` to fill in."

---

### `voice-and-formats.md`

```markdown
# Voice and Formats

## Voice
[3–5 adjectives from Round 3, expanded into brief descriptions]

## Voice rules
[What the voice explicitly avoids — 'not X, not Y' rules from Round 3]

## Channel specs

### [Channel name, e.g. LinkedIn]
- Format: [thread / short-form post / long-form essay]
- Length target: [e.g. 150–300 words]
- [Any channel-specific rules]

### [Channel name, e.g. Newsletter]
- Format: [format]
- Length target: [target]
- [Rules]

*(Add a section per channel)*

## Visual style
[Image style preferences, no-stock-photo rules, color palette — omit section if nothing given]

## Related
- [Story](story.md)
- [Cadence](cadence.md)
```

If skipped: placeholder.

---

### `cadence.md`

```markdown
# Publishing Cadence

## Rhythm
[Number of pieces per week and which days from Round 4]

## Autonomy level
**[L1 / L2 / L3 / L4]** — [exact level description]

- L1 — founder reviews and approves every piece before publish
- L2 — agent drafts; founder edits before publishing
- L3 — agent drafts and queues; founder reviews weekly, can roll back
- L4 — agent publishes autonomously; founder reviews weekly

## Review expectations
[Founder's stated review process from Round 4]

## Related
- [Themes and topics](themes-and-topics.md)
- [Voice and formats](voice-and-formats.md)
```

If skipped: placeholder with "Autonomy: not set — default to L1 (founder reviews all)."

---

## Step 4 — Success summary

After writing files:

```
## Content context written ✓

Created in docs/context/content/:
- story.md — [audience in one line]
- themes-and-topics.md — [publishing days]
- voice-and-formats.md — [channels listed]
- cadence.md — Autonomy [L1/L2/L3/L4]

Your content context is ready. Run /pisk:post to draft your first piece.

Tip: if your voice drifts or you change channels, re-run /pisk:content and
select 'voice' to update just that file.
```

---

## General rules

- Write files to the **founder's project repo**, not this skills repo.
- `cadence.md` MUST record an explicit autonomy level (L1–L4). If the founder skips Round 4, default to L1 and note it.
- `voice-and-formats.md` voice rules should have at least 5 entries — if the founder gives fewer, ask one follow-up to expand.
- Skipped domains get placeholder files — never leave a file missing entirely.
- On re-run, only overwrite files for interviewed domains; leave unchanged files as-is.
- `pisk:post` reads these files verbatim — write them as briefs for another agent, not as notes for the founder.
