---
name: pisk:post
description: Draft today's content piece from the docs/context/content/ canonical files, at the founder's configured autonomy level.
version: 0.1.0
license: MIT
compatibility: claude-code>=1.0
---

# `/pisk:post` — Daily Content Production

You are drafting today's content piece from the founder's locked canonical context files. Read the files first. Draft second. Present for review third.

---

## Step 1 — Check context files

Check that all four files exist in `docs/context/content/`:
- `story.md`
- `themes-and-topics.md`
- `voice-and-formats.md`
- `cadence.md`

**If any are missing:** output exactly:

```
Missing context files: [list missing files]

Run /pisk:content first to set up your content context, then re-run /pisk:post.
```

Then stop. Do not attempt to draft without the full context.

---

## Step 2 — Read all context files

Read all four files before doing anything else. Load:
- The audience persona and transformation from `story.md`
- Today's theme from `themes-and-topics.md`
- All voice rules and channel format specs from `voice-and-formats.md`
- The autonomy level from `cadence.md` (the `Autonomy:` field)

---

## Step 3 — Select today's theme

Get the current day of week (e.g. Monday).

Find the matching row in `themes-and-topics.md` and note:
- The theme name
- The topic types for that day

**Override:** If the user's invocation message specifies a theme, topic, or day (e.g. "/pisk:post use Tuesday's theme" or "/pisk:post — do a founder story"), use the specified theme instead.

If today's day is not in the theme map (e.g. weekend and the founder only publishes weekdays), use the most recent mapped day and note it in the draft header.

---

## Step 4 — Draft

Draft one content piece. Apply every rule in `voice-and-formats.md` as a hard constraint. Before presenting, internally self-check the draft against every voice rule. Revise internally until no rules are violated — then present.

**Draft header** (shown to founder, not part of the published content):
```
Theme: [theme name] ([day])
Channel: [channel name]
Format: [format name]
Autonomy: [L1/L2/L3/L4]
```

**Format rules:**
- Follow the format spec for today's channel from `voice-and-formats.md`.
- **Carousels:** max 4 slides, each as a clearly separated section with a slide heading.
- **Threads:** each tweet/post on its own line, numbered.
- **Long-form:** write to the stated length target.
- **Missing format spec:** default to single text post, note in header: `Format: text post (no format spec found for [channel] — update /pisk:content to fix)`.

**Voice rule conflicts:** If two voice rules pull in opposite directions, apply them in order of their listing in `voice-and-formats.md` and note the tension in the draft header so the founder can decide.

---

## Step 5 — Review cycle

The review cycle is determined by the `Autonomy:` level in `cadence.md`.

---

### L1 — Founder reviews every piece

Present the draft, then say:
> "Read this back aloud and note any words or lines that don't feel right. Tell me the corrections and I'll revise."

Wait for feedback. Revise once. Present the revised draft. Say: "Here's the revised version — ship it when ready."

---

### L2 — Quick edit before publishing

Present the draft, then say:
> "Approve to ship, or tell me one change and I'll revise."

Wait. If approved: "Ship it." If revision requested: revise once, present. Say: "Here's the revised version."

---

### L3 — Near-final, founder can ship as-is

Present the draft, then say:
> "This is ready to ship. Flag any concerns or copy it straight to [channel]."

---

### L4 — Agent output, founder reviews async

Present the draft. No review prompt. End with:
> "Done. Saved to review queue."

---

### Second revision (any level)

If the founder asks for a second revision after the first:
Revise and present. Then add exactly:
> "Two revision passes done — ship it or the moment passes."

Do not offer a third revision pass.

---

## Step 6 — Output

Default: the final draft appears in chat. No files written.

**Save on request:** If the founder asks to save the draft, write it to:
- The path they specify, or
- `docs/drafts/YYYY-MM-DD.md` (today's date) if no path given.

Include the draft header and the final content in the saved file.

---

## General rules

- Never write files unless the founder asks.
- Never modify the four canonical context files.
- One piece per run. No batching.
- Treat voice rules as hard constraints, not style preferences. Self-correct before presenting.
- No image generation.
- No platform publishing.
- If `cadence.md` has no `Autonomy:` field, default to L1 and note it in the draft header.
