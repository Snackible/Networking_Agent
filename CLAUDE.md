# CLAUDE.md — Networking Opportunity Agent (run protocol)

This repo is a **networking-opportunity scanner** for Snackible's CEO, **Aditya Sanghavi**.
It finds places he can appear (podcasts, YouTube, summits, panels, magazines, awards,
founder lists) and writes a ranked report with links.

**If you are Claude and you've just been given this repo and asked to "run the agent",
follow the protocol below. Do not skip the scheduling question.**

---

## ▶ When asked to "run the agent" — do this, in order

### 1. Ask how often to schedule + where to email (ALWAYS ask first, on first run)
Before running anything, ask the user two things:

**(a) "How often should I schedule this?"**
- **Daily** → cron `0 9 * * *`
- **Weekly** (default) → cron `0 9 * * 1` (Mondays)
- **Every weekday** → cron `0 9 * * 1-5`
- **Biweekly / Monthly** → e.g. monthly `0 9 1 * *`
- **Just once now** (no schedule) → skip to step 3, don't create a task

**(b) "Where should I email each report — and do you want email at all?"**
Ask for the recipient email address (this repo does not store one). Use whatever the user gives
as `<RECIPIENT_EMAIL>` in the scan prompt. If they decline email, remove the email step from the
task prompt and deliver reports as files only.

Cron is in the user's **local** time. Confirm both choices back to them.

### 2. Create / update the scheduled task
Use the scheduled-task tools (`mcp__scheduled-tasks__create_scheduled_task`, or
`update_scheduled_task` if a task named `snackible-networking-agent` already exists — check
with `list_scheduled_tasks`). Use the cron from step 1 and the **scan prompt in
[`agent-prompt.md`](agent-prompt.md)** as the task prompt (it is the single source of truth for
the scan routine) — substituting the `<RECIPIENT_EMAIL>` the user gave in step 1(b). Tell the user: tasks run only while the Claude app is open; a due
run fires on next launch.

### 3. Run a scan now
Execute the routine in [`agent-prompt.md`](agent-prompt.md) immediately for a first/fresh report,
so the user sees output without waiting for the schedule.

---

## 📂 Store reports locally + never repeat (core rules)

- **Store locally:** every report is written to `reports/YYYY-MM-DD-opportunities.md` in this repo.
  Reports are kept (committed) so there's a durable history.
- **Never repeat:** `seen.json` holds a dedup key for every opportunity ever reported
  (`lowercase(venue + "|" + title)`). Before writing a report, **read `seen.json`, skip anything
  already in it**, and after writing, **append the new keys**. This is what stops the same podcast
  or annual summit being re-reported every run.
- **Watchlist:** `watchlist.md` lists recurring venues to re-check each run; the agent appends
  strong new venues it discovers. The user may hand-edit it — respect it.

---

## 🗂 Repo map
| File | Purpose |
|---|---|
| `CLAUDE.md` | This protocol (you are here) |
| `agent-prompt.md` | The exact scan routine — single source of truth for a run |
| `watchlist.md` | Recurring venues + subject profile (editable) |
| `seen.json` | Dedup memory — do not hand-edit unless resetting |
| `reports/` | Dated reports (kept locally) |
| `PLAN.md` | Design doc / background |

## 🔒 Guardrails
- **Report only.** Never submit applications, pitch, post publicly, or create accounts.
- The only message the agent may send is the weekly report email to **the configured recipient**
  (a single self-address kept in the scheduled-task config, not in this repo; see
  `agent-prompt.md` step 10), and only if the user has set up email delivery.
- Every reported item must carry a working link; drop anything unlinkable.
