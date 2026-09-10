# Setup

One-time setup to get the brief arriving on your phone each morning and readable
on your PC. Everything here is done in a browser — no terminal, no local install.

> **Most people should not read this file.** Open your repository at
> [claude.ai/code](https://claude.ai/code) and run **`/setup`** instead — it
> interviews you, finds and verifies sources for your topics, writes your beat
> files, and gives you a short list of things to click. This document is the
> manual path, and a reference for anything `/setup` hands back to you.

Work through the steps in order; later steps need values from earlier ones.

**Time:** about 20 minutes. **Cost:** $0 — nothing here asks for a credit card.

---

## Step 1 — Make your own copy

On this repository's GitHub page, click **Use this template** → **Create a new
repository**.

Name it whatever you like. **Private is fine** — everything downstream works
with a private repository, and nothing here needs to be public.

If you don't see that button, click **Fork** instead. Either gets you your own
copy to edit.

**Success:** a repository under your own account with these files in it.

---

## Step 2 — Get your Pushover credentials

Skip this if you don't want phone notifications; everything else still works.

1. Sign in at **https://pushover.net**
2. Copy **Your User Key** from the dashboard — this is `PUSHOVER_USER`
3. Scroll down, click **Create an Application/API Token**, name it whatever you
   want, and create it
4. Copy the **API Token/Key** — this is `PUSHOVER_TOKEN`

You need both. Keep them somewhere temporary — they go into the environment in
Step 4, never into the repository.

---

## Step 3 — Deploy the site on Cloudflare

Do this before Step 4, because it produces the URL Step 4 needs.

The repository carries a `wrangler.jsonc` that serves `site/` as static assets.
There is no Worker script and none is needed — requests are handled by
Cloudflare's asset path, with no cold start and no CPU billing.

1. Sign in at **https://dash.cloudflare.com** (no card required)
2. **Compute (Workers & Pages)** → **Create** → **Import a repository**
3. Authorize Cloudflare's GitHub app and grant it access to your repository
4. On the **Set up your application** screen:

   | Field | Value |
   |---|---|
   | Project name | whatever you want — it becomes part of your URL |
   | Build command | *leave empty* |
   | Deploy command | `npx wrangler deploy` (already filled in — keep it) |
   | Builds for non-production branches | uncheck |

   The empty build command is correct, not an oversight. The pipeline commits
   finished HTML, so there is nothing to compile.

5. Click **Deploy**

**Success:** a live `*.workers.dev` URL showing *"No briefs yet."* That is the
right answer — no brief has run.

**Copy that URL.** It is `SITE_BASE_URL` in the next step.

### Optional — put the site behind a login

The site is public by default: anyone with the URL can read it. To lock it down,
turn on **Protect with Cloudflare Access** — it's on the deploy screen, free for
up to 50 users, and gives you a one-time email code when you open the site. You
can also enable it later from the project's settings.

---

## Step 4 — Create a cloud environment

This controls what the daily run can reach on the network, and holds your
Pushover keys.

**Create a new environment — do not edit your Default one.** Default uses
*Trusted* access, which allows package registries and dev domains that your other
work probably needs. Narrowing it would break that.

1. Go to **https://claude.ai/code**
2. Click the **cloud icon** above the message box → **Add cloud environment**
3. **Name:** anything, e.g. `daily-brief`
4. **Network access:** select **Custom** — the *Allowed domains* box only appears
   once you do
5. Paste the domain list below into **Allowed domains**

```
lightwaveonline.com
*.lightwaveonline.com
fierce-network.com
*.fierce-network.com
telecompetitor.com
*.telecompetitor.com
bbcmag.com
*.bbcmag.com
isemag.com
*.isemag.com
fiberbroadband.org
*.fiberbroadband.org
fcc.gov
*.fcc.gov
ntia.gov
*.ntia.gov
tiaonline.org
*.tiaonline.org
itu.int
*.itu.int
ieee.org
*.ieee.org
bicsi.org
*.bicsi.org
thefoa.org
*.thefoa.org
pmi.org
*.pmi.org
asq.org
*.asq.org
iso.org
*.iso.org
nist.gov
*.nist.gov
ansi.org
*.ansi.org
astm.org
*.astm.org
esri.com
*.esri.com
qgis.org
*.qgis.org
autodesk.com
*.autodesk.com
bls.gov
*.bls.gov
lightreading.com
*.lightreading.com
broadbandbreakfast.com
*.broadbandbreakfast.com
benton.org
*.benton.org
statescoop.com
*.statescoop.com
ntca.org
*.ntca.org
ustelecom.org
*.ustelecom.org
nspe.org
*.nspe.org
roberthalf.com
*.roberthalf.com
qualitydigest.com
*.qualitydigest.com
elsmar.com
*.elsmar.com
construction-institute.org
*.construction-institute.org
iaf.nu
*.iaf.nu
api.pushover.net
```

   Leave **"Also include default list of common package managers"** unchecked —
   this project has no dependencies.

   `api.pushover.net` is what lets the notification out. Without it the brief is
   still written and published, but your phone stays quiet.

   **This list matches the OSP/fiber beats shipped in this repository.** If you
   rewrite the beats for a different field, replace these with your own sources.

> **Or just use Full.** The agents drop any item they can't actually open, so
> unreachable sources are discarded rather than guessed at — the allowlist isn't
> doing quality control. What it does is limit where the beats can look, which
> particularly hurts a beat meant to follow good writing wherever it lives.
> Setting **Network access: Full** removes that ceiling and the maintenance. The
> isolation that matters is unchanged either way: the container is ephemeral and
> has no path to your own machine.
>
> Be aware that some sites refuse automated fetches regardless of your network
> setting — several major industry sites return 403 to any bot. Those items get
> dropped and reported. That is the rule working, not a misconfiguration.

6. In **Environment variables**, add these three lines with your own values:

```
PUSHOVER_TOKEN=your_application_api_token
PUSHOVER_USER=your_user_key
SITE_BASE_URL=https://your-project.your-subdomain.workers.dev
```

No quotes, no spaces around `=`, no trailing slash on the URL.

7. **Create environment**

> **On the keys.** Environment variables are visible to anyone using this
> environment — on a personal account, that's you. Pushover keys are low risk:
> worst case someone sends notifications to your phone. If one leaks, regenerate
> it on the Pushover dashboard and update it here.

---

## Step 5 — Create the routine

1. Go to **https://claude.ai/code/routines** → **New routine**

   Use the full form, not the "What do you want automated?" box — that drafts a
   paraphrase, and the exact wording below matters.

2. **Name:** anything, e.g. `Daily brief`
3. **Prompt:** paste exactly:

```
Run the daily research brief for this repository.

Read .claude/commands/brief.md and follow it exactly, start to finish. Every
step, in order, including rendering the site, committing, pushing, and sending
the Pushover notification.

Work on the main branch and push directly to it. Do not open a pull request.

Do not skip the push or the notification. If either one fails, say so plainly
in your final message instead of reporting success.
```

4. **Repositories:** your repository from Step 1
5. **Environment:** the one from Step 4 — **not** Default
6. **Connectors:** remove all of them. They are attached by default, and a
   routine runs unattended with no permission prompts, so anything left attached
   is usable without asking. This one needs none.
7. **Trigger:** Schedule → Daily → pick a time
8. **Create**

After creating it, check the **Runs with** row on the detail page and confirm it
lists only your repository and environment. If a connector is still there, click
**Edit** and remove it.

### Pick a time earlier than you want it

The routine **starts** at the scheduled time; the run takes 6–12 minutes and
there's a deliberate few-minute stagger on top. **Schedule it about 15 minutes
before you want the notification.** The offset is consistent per routine, so
after a few mornings you can tighten it.

While you're there: **Push notifications** is Claude's own "routine finished"
alert, separate from the Pushover message carrying the brief. Leaving it on means
two notifications every morning.

---

## Step 6 — Confirm you can't be billed

Go to **https://claude.ai/settings/usage** and check that **usage credits are
off**.

That is the only path by which this could charge you. Off means a run is rejected
if you hit your plan limit. On means it continues on metered overage.

---

## Step 7 — Test it now

Don't wait until the scheduled time to discover a misconfiguration. Click **Run
now** on the routine and watch it.

- [ ] The beats run and return items (or an honest `nothing new`)
- [ ] `briefs/YYYY-MM-DD.md` is written
- [ ] `render.py` succeeds
- [ ] `git push` succeeds — **most likely to fail**
- [ ] Cloudflare shows a new deployment within a minute or two
- [ ] Your phone gets a notification
- [ ] Tapping it opens today's brief

**Push fails:** check whether `main` is a protected branch.

**No notification:** confirm `api.pushover.net` is in the allowlist and both
Pushover values are set. The run log shows the exact error.

**Site doesn't update:** confirm the Cloudflare project's production branch is
`main` and its build output directory is `site`.

---

## After setup

Nothing, from you. Each morning a cloud session starts on its own, researches,
writes, publishes, pushes, and notifies. You read it.

| You want to | Do this |
|---|---|
| Change the time | Routine detail page → edit the schedule |
| Pause it | Routine detail page → toggle in **Repeats** |
| Change what a beat covers | Edit the file in `.claude/agents/` and commit |
| Add or drop a source | Edit the agent file, and update the allowlist in Step 4 |
| Stop notifications | Remove `PUSHOVER_TOKEN` from the environment |
| See why a run did nothing | Routine detail page → open the run and read the log |

A green run status only means the session started and exited without an
infrastructure error. It does not mean the brief was any good. Open the run and
read it when something looks off.

## Pruning covered.json

`covered.json` is a flat array of `{url, headline, date}`, where `date` is the day
the item **went into a brief**, not the article's publication date — which is
what makes pruning by age straightforward.

It grows by up to 18 entries per run. Every agent reads the whole file before
searching, so once it's long it costs real context each run. Prune past roughly
300–500 entries.

```bash
jq 'length' covered.json                    # check size
jq '.[-400:]' covered.json > t && mv t covered.json   # keep the most recent 400
```

Pruning has one consequence: a pruned URL becomes eligible again, so an old item
can resurface. For news beats that's harmless — old news won't clear the bar
anyway. It matters more for a beat with no recency requirement, where a good
article from years ago is a legitimate item forever.

The files in `briefs/` are the real archive — committed and never pruned — so
nothing is lost by trimming the ledger. Keep it a valid JSON array; if it's
corrupt the pipeline stops rather than recreating it, because recreating would
erase the history and make the next brief repeat everything.
