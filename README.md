# Under 200k — validation landing page

A single-page landing site to validate demand for a Claude Code token-discipline playbook before writing it.

**Status:** pre-launch. Kill/greenlight gate at 7 days post-HN launch. See [LAUNCH.md](LAUNCH.md).

## What this is

- One `index.html`, one `styles.css`, zero JavaScript, zero dependencies
- Deployed via GitHub Pages on push to `main`
- Email capture via Buttondown (swappable — form action is a single URL)

## What this is NOT

- Not the product. The product is a closed-source ebook, written only if this page clears 100 emails in 7 days.
- Not a newsletter. One launch email, nothing else.
- Not open-source infrastructure. Screenshots and results are shown; copy-paste assets are withheld intentionally.

## Local preview

Open `index.html` in any browser. No server needed.

## Deploy

Push to `main`. The workflow at `.github/workflows/pages.yml` handles the rest — it auto-enables Pages via the REST API on first push.

## Structure

```
under200k/
├── .github/workflows/pages.yml   # GH Pages deploy
├── index.html                     # The whole landing page
├── styles.css                     # Editorial palette (adapted from fahad-memoir)
├── LAUNCH.md                      # Fahad-only launch checklist
├── README.md                      # This file
└── .gitignore
```

## Design language

Adapted from `fahad-memoir`:
- Ivory `#F5F1EB` / warm charcoal `#1A1A1A`
- Playfair Display (display) + Inter (body) + JetBrains Mono (statusline accent)
- No `backdrop-filter: blur`. Ever.
- Traffic-light accents (green/yellow/red) for the statusline demo only
