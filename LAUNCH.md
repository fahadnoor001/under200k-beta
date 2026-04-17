# Launch checklist — Under 200k (Gumroad audience test)

**Goal this phase:** gauge audience. Not a product launch — a hype + signal test before committing to write Part I.
**Channel:** Gumroad free listing (email capture) + IG story (traffic driver). Running in parallel.
**Decision gate:** 72h after both surfaces are live. ≥10 page visits + ≥3 emails/Gumroad follows = write Part I. <3 = re-angle headline before writing anything.

## The Gumroad move (free listing, ~15 min)

Listing is a **pre-launch email capture**, not a sale. $0, no file delivered, buyer gets a "you'll be emailed when it ships" confirmation. This gives Gumroad something to index in Discover and gives you a second capture surface beyond mailto.

### Steps (gumroad.com, logged in as you)

1. **+ New product** → type: **Digital product** → **"I'll deliver it later"** (pre-order toggle, or just leave file field blank and set price to $0).
2. **Name:** `Under 200k — a field guide to heavy Claude Code sessions`
3. **Price:** `$0` (free). Optional: enable "Pay what you want" with `$0` minimum, `$5` suggested — lets supporters tip without gating capture.
4. **Cover image:** upload `assets/og-image.png` (see rasterization step below).
5. **Short description (skim copy — first 240 chars show in Discover):**
   ```
   Stop hitting the 200k-token wall. A field guide to running heavy Claude Code sessions without losing context — or your sanity. Pre-launch: sign up free, get one email on the day the playbook ships. No newsletter, no drip.
   ```
6. **Full description (Markdown — paste as-is):**
   ```
   ## What this is

   A field guide for Claude Code power users who've hit the 200k-token wall and watched autocompact swallow their work.

   Not a newsletter. Not a course. One PDF, shipped once, when it's real.

   ## What's coming

   **I. A context audit you can run in under two minutes.** Walk every component that eats tokens — messages, MCP connectors, skills, agents, hooks, system prompt — and learn which two typically hide the most bloat.

   **II. Where MCP sprawl actually lives.** If your local settings.json looks clean but you're still loading 60k+ tokens before typing, the culprit is browser-side. The playbook names the exact setting and where to click.

   **III. A response protocol enforced by a Stop-hook.** Five mandatory sections per response — including a "harshest critic" block. The playbook gives the template, the hook script, and the reason each section exists.

   **IV. Verify the binary, not the docs.** A sixty-second grep pattern on any installed CLI. The playbook walks a real example: finding `context_window.used_percentage` in `cli.js` — a field the public docs don't mention.

   ## Honest status

   - The statusline itself — shipped, reading real `context_window` fields from the bundled CLI.
   - The response-protocol Stop-hook — running on every session on my personal setup.
   - The audit methodology — used on my own work, not yet written down.
   - The written playbook — in draft, not yet a deliverable PDF.

   Deep-dive: [fahadnoor001.github.io/under200k-beta](https://fahadnoor001.github.io/under200k-beta/)

   ## What you get by signing up

   One email, on the day there's something real. That's it.
   ```
7. **Tags:** `claude`, `claude-code`, `ai`, `developer-tools`, `productivity`, `llm`
8. **URL slug:** `under200k` → your public URL becomes `https://<yourhandle>.gumroad.com/l/under200k`
9. **Publish.**

### Rasterize og-image.svg → og-image.png (2 min)

Gumroad won't accept SVG for cover. Fastest path:
- Open `assets/og-image.svg` in Chrome.
- Right-click → **Save image as** — Chrome will offer PNG for inline SVG with explicit width/height. If it only offers SVG, use a converter:
  - cloudconvert.com/svg-to-png → upload → **Custom resolution: 1200 × 630** → convert → download → save as `assets/og-image.png`
- Drop the PNG into the repo, commit, push. The landing page `og:image` meta tag already points to it.

## Landing page: swap mailto → Gumroad URL

Once the Gumroad listing is live, send me the URL (format: `https://<yourhandle>.gumroad.com/l/under200k`) and I'll swap both CTA buttons in `index.html` in one commit. The mailto stays as a fallback for people who don't want to touch Gumroad.

## IG story (15 seconds, one post) — parallel surface

Script, record, post within the same 72h window so the signal attribution is clean (either surface could be the source, both traffic paths funnel to the same decision gate).

**Script:**
1. 0–4s — phone screen recording of `fahadnoor001.github.io/under200k-beta` hero, focused on the animated statusline ticking 🟢 → 🟡 → 🔴
2. 4–10s — text overlay: *"Stopped hitting the 200k wall."*
3. 10–15s — link sticker to landing page (swap to Gumroad URL once live). Final frame holds on the headline.

**Record it:** phone-native screen recorder (iOS Control Center / Android quick settings). Portrait orientation. Send clip to me for trim/polish before posting.

## The 3 DMs (warm outreach, not cold)

Three accounts you already follow who post about Claude Code, context limits, agent workflows. One-liner template:

> Built a statusline that shows real context usage — working on a playbook around it. Curious if the wall hits you too: [link]

Space 30 min apart, personal IG, not a copy-paste.

## Track (72h window)

- **GitHub traffic graph:** https://github.com/fahadnoor001/under200k-beta/graphs/traffic (visits + unique)
- **Gmail:** search `subject:"Notify me — Under 200k"` for mailto captures
- **Gumroad dashboard:** followers count on the product
- **IG story insights:** link sticker taps, story views

### Decision gate

| Signal over 72h | Action |
|---|---|
| ≥10 visits AND ≥3 captures (email OR Gumroad follow) | Green light — write Part I |
| 3–10 visits OR 1–2 captures | Yellow — re-angle headline, re-post IG story once |
| <3 visits AND 0 captures | Red — salvage as memoir blog, no Part I |

## Explicitly cut (preserved so the decision is legible later)

- No HN "Show HN" post
- No Twitter/X thread
- No r/ClaudeAI post
- No newsletter outreach
- No podcast guest slots
- No paid ads

Rationale: warm DM + IG story + marketplace = three cheap signal surfaces that don't burn your one shot at cold broadcast. Cold channels get saved for when there's a real deliverable to point at.

## Guardrails

- Never `fahadnoor.ai` for any URL (per `feedback_domain_constraint.md`)
- Never open-source playbook assets — screenshots only
- Never commit `.env`, Gumroad API keys, or OAuth tokens
- If Gumroad rejects the free listing for any reason, do NOT reflex-list on Lemon Squeezy — read the rejection, fix, re-submit
