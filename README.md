# Telecom brief kit

**A daily brief on whatever you need to stay current on — researched, written,
and delivered to your phone every morning while you sleep.**

You tell it what topics matter to you. It finds the sources, reads them daily,
skips anything it already told you, and sends you a short brief with a link.

Built for telecom, but the topics are yours. Fiber buildouts and BEAD funding,
copper retirement, competitor activity, supply chain and manufacturing,
standards work, certifications — whatever you actually need to know about.

**[See what a brief looks like →](example-brief.md)**

---

## Getting started

### 1. Make your own copy

Click **Use this template** → **Create a new repository** at the top of this
page. (No button? Click **Fork** instead.) Private is fine — everything works
with a private repository.

### 2. Open it in Claude and run `/setup`

Go to **[claude.ai/code](https://claude.ai/code)**, start a session on your new
repository, and type:

```
/setup
```

Nothing to install. It runs in your browser.

### 3. Answer a few questions

It asks what you do, what's already on your radar, where you're headed, and
**what topics you want tracked**. Answer in plain language — you do not need to
know any websites:

> *"AT&T and Verizon fiber buildouts, BEAD funding news, copper retirement,
> supply chain and manufacturing — Corning, Prysmian — competitor activity,
> FCC policy."*

Then it goes and **finds the sources for you**, checks each one is live and
readable, writes your beat files, and hands you a short list of things to click
— maybe ten minutes of setup, all in a browser.

That's it. Next morning, your phone buzzes.

---

## What you need

| | |
|---|---|
| **Claude Pro or Max** | The daily run is an ordinary Claude Code session |
| **GitHub account** | Free. Private repository is fine |
| **Notifications** | Free — the routine's own push and/or email. **Pushover** (~$5 one-time) is an optional upgrade |
| **Cloudflare** | Free, optional — only if you want a website |

**Running cost: nothing.** No API key anywhere in this project. The research
runs on your existing Claude subscription; everything else is free tier. One
thing to check during setup: turn *off* usage credits at
claude.ai/settings/usage, so hitting your plan limit skips a run rather than
billing you.

### Where the brief ends up

Every brief is committed to your repository as markdown, and GitHub renders it
perfectly well on a phone. That alone is a complete working system.

If you want a proper website — cleaner reading, an archive of past briefs — the
repository includes everything for that, and it's about five extra minutes on
Cloudflare's free tier. Setup will ask.

---

## How it works

Two to five research agents each cover a "beat." They run in parallel every
morning, each reading a ledger of what's already been reported so nothing
repeats. Findings get merged, de-duplicated across beats, ranked, and written up
as short paragraphs.

```
6:00am  Scheduled run starts on its own (your devices can be off)
          |
          +-- beat 1 ....... your topics
          +-- beat 2 ....... your topics
          +-- beat 3 ....... your topics
          +-- beat 4 ....... your topics
          |
          v
        merge, drop cross-beat duplicates, rank
          |
          +--> briefs/YYYY-MM-DD.md ... markdown archive
          +--> covered.json ........... so nothing repeats
          +--> site/ .................. optional website
          |
          v
        notification --> push or email, with a link
```

## The beats in this repository are an example

`.claude/agents/` ships with four beats tuned for outside plant fiber design
and QC — industry news, market forces, professional development, and quality
practice. They are there so you can see what a well-specified beat looks like —
**`/setup` replaces them with yours.**

If you'd rather do it by hand, they're plain markdown. Copy one, rewrite the
scope and sources, then add it to the spawn list and section order in
`.claude/commands/brief.md`.

## Design decisions worth keeping

These came out of running it and finding the failure modes.

**Open the page or drop the item.** An agent must actually fetch a page before
reporting it. If the fetch fails — blocked, paywalled, gone — the item is
dropped and reported, never written up from search results. Without this you get
plausible-looking items assembled from search snippets, with wrong dates and
dead sources. This is the most important rule here.

**"Nothing new" is a valid answer.** A beat with nothing worth reporting says so
rather than padding. A brief you trust to be empty on a quiet day is worth more
than one that always finds five things.

**Agents can't write files.** They research and report; the pipeline does the
writing. Read-only agents can't corrupt your archive or your ledger.

**Range across sources.** If most items come from one outlet, the agent searched
one site instead of working a beat. There's an explicit rule against it.

**The ledger is the whole trick.** `covered.json` records every reported item,
and every agent reads it before searching. It's what stops the brief becoming
the same five stories every week.

**Standard library only.** `render.py` and `notify.py` use nothing but Python's
standard library. This runs in a fresh container every morning, and a pipeline
that can't break because a dependency moved is worth more than any convenience a
library would add.

## Files

| Path | What it is |
|---|---|
| `.claude/commands/setup.md` | The `/setup` interview — start here |
| `.claude/agents/*.md` | Your beats (example ones ship in the box) |
| `.claude/commands/brief.md` | The pipeline the daily run follows |
| `render.py` | Turns `briefs/*.md` into the website |
| `notify.py` | Sends the optional Pushover notification |
| `covered.json` | Dedupe ledger: `{url, headline, date}` |
| `briefs/` | Your archive, one markdown file per day |
| `SETUP.md` | Manual setup, if you'd rather not use `/setup` |

## A note on expectations

The plumbing works and is well-tested. **The beats are where the quality lives**,
and they'll need tuning for your field specifically.

Expect the first week to involve reading each brief critically and saying which
items were useless. That's not a defect — the editorial judgment in those files
*is* the product, and it can't be got right in the abstract. Open a Claude
session on your repository, say which beat produced the weak item and why, and
have it adjust.
