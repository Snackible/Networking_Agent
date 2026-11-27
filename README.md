# Networking Opportunity Agent

An AI agent that scans the web for places **Aditya Sanghavi** (Founder & CEO, Snackible) can
show up for networking and outreach — podcasts, YouTube shows, summits, panels, magazines,
awards, and founder lists — and produces a ranked report **with links**.

## How to use it

Give this repo to Claude (Claude Code / cloud coworker) and say:

> **"Run the agent."**

Claude will then, automatically:
1. **Ask how often to schedule** (daily / weekly / weekdays / monthly / just once).
2. **Set up the schedule** at that cadence.
3. **Run a scan now** and save a dated report to `reports/`.

See [`CLAUDE.md`](CLAUDE.md) for the exact protocol Claude follows.

## What you get

A report like `reports/2026-11-23-opportunities.md`, grouped into:
- 🔥 **Act now** — open, time-sensitive (deadlines, open nominations)
- 🎙️ **Podcasts** — evergreen guest targets
- 🏆 **Awards / lists** — open nominations
- 📌 **Recurring venues** — flagship events to pitch early for the next edition

## How it avoids repeats

Every opportunity ever reported is recorded in `seen.json`. Each run reads it first and **skips
anything already reported**, so you only see what's new. Recurring venues (like the same annual
summit) won't clutter every report.

## Files
- `CLAUDE.md` — the run protocol for Claude
- `agent-prompt.md` — the scan routine (editable)
- `watchlist.md` — recurring venues + who we're tracking (editable)
- `seen.json` — dedup memory
- `reports/` — saved reports
- `PLAN.md` — design/background

## Notes & guardrails
- Scheduled runs happen **while the Claude app is open**; a missed run fires on next launch.
- The agent **only reports** — it never applies, pitches, posts, or creates accounts.
- Optional weekly email delivery to a configured address can be enabled (ask Claude to set it up).
