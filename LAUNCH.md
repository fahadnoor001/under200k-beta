# Launch checklist — Under 200k (Gumroad + IG story)

**Channel:** Gumroad marketplace listing, primary. Single IG story for show-and-tell, not a growth channel.
**Decision gate:** 14 days post-listing. ≥25 sales = write Part II + ship v2. 10–24 = hold, re-angle copy. <10 = post-mortem, salvage as memoir blog.

## Pre-list (PC, ~60 min)

- [ ] **Render OG image to PNG.** Open `assets/og-image.svg` in Chrome → right-click → "Save as" → `assets/og-image.png` (Chrome rasterizes at 1200×630 automatically). Verify the file loads in `index.html` OG preview tester (metatags.io).
- [ ] **Gumroad account.** Sign up at gumroad.com → verify email → set payout (Stripe). Skip the seller quiz or breeze through it.
- [ ] **Lock product name.** Current: **Under 200k**. Alternates in `~/.claude/plans/gtm-playbook-outline.md` appendix.
- [ ] **Lock price.** Launch $29, regular $39 (Gumroad's "limited-time offer" switch handles the auto-flip).
- [ ] **Hero asset.** Use `assets/og-image.png` as the listing cover. Gumroad crops to ~1280×720 — the SVG already has bottom breathing room so the crop is safe.
- [ ] **Pitch copy.** Lift from landing page: headline + subhead + the 4 promises. Keep it skimmable — Gumroad shoppers read 30 seconds max.
- [ ] **Preview file.** DEFER — Chapter 1 doesn't exist yet. Ship listing without preview first pass; add Chapter 1 sample PDF as soon as it's drafted.
- [ ] **Refund policy.** Gumroad default 30-day no-questions. Keep it.

## Push landing page to GitHub

- [ ] `cd C:\Users\fahad\Documents\AI\under200k`
- [ ] `git init && git add . && git commit -m "Initial: Under 200k landing + OG + animated statusline"`
- [ ] `gitleaks detect` (should be clean)
- [ ] `gh repo create fahadnoor001/under200k-beta --public --source=. --push`
- [ ] Visit `https://fahadnoor001.github.io/under200k-beta/` after ~90s → confirm page renders, OG image loads (check view-source), animated statusline ticks

## Link the two

- [ ] In Gumroad listing description, add the landing page URL as the "Learn more" link
- [ ] In landing page, update the signup form or add a subtle "Buy on Gumroad" button after the listing goes live (swap `REPLACE_ME` buttondown action to `https://<yourhandle>.gumroad.com/l/<slug>`)

## IG story (15 seconds, one post)

**Script:**
1. 0–4s — screen recording of the landing page, focused on the animated statusline ticking 🟢 → 🟡 → 🔴
2. 4–10s — text overlay: "I kept hitting the 200k wall. So I built the system that stopped it."
3. 10–15s — link sticker to `<yourhandle>.gumroad.com/l/under200k`, final frame holds on the headline

**Record it on PC:**
- ShareX: Hotkey → "Screen recording (GIF)" or MP4 mode → draw region around browser → record 15s → auto-saves to `Documents\ShareX\Screenshots\`
- ScreenToGif: even lighter, 1.5MB install, native recorder + trimmer in one window

## Track

- [ ] Day 3: listing views + add-to-cart rate (Gumroad dashboard)
- [ ] Day 7: sales count + refund count + any IG story reactions/DMs
- [ ] Day 14: **decision gate** — see top of file

## Explicitly cut (preserved so the decision is legible later)

- No HN "Show HN" post
- No Twitter/X thread
- No r/ClaudeAI post
- No newsletter outreach (Latent.Space, Ben's Bites, TLDR AI)
- No podcast guest slots

Rationale: marketplace shoppers convert in buying mode. Social channels convert in browsing mode, and browsing-mode traffic on a page with no preview PDF is waste. Revisit any of these if 14-day gate hits ≥25 sales — the extended update pack cohort offers cleaner social fuel than the base product.

## Guardrails

- Never `fahadnoor.ai` for any URL (per `feedback_domain_constraint.md`)
- Never open-source the playbook assets — screenshots only
- Never commit `.env`, Gumroad API keys, Buttondown admin tokens
- If Gumroad rejects the listing for any reason, do NOT reflex-list on Lemon Squeezy — read the rejection, fix, re-submit. Dual-listing adds upkeep with no measured lift.
