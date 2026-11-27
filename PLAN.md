# Networking Opportunity Agent — Plan

A scheduled Claude agent that scans the web on a recurring basis for places the
Snackible CEO could appear (podcasts, YouTube shows, summits, panels, magazines,
awards, founder lists) and delivers a report of ranked opportunities **with links**.

---

## 1. Who we're tracking

**Subject:** Aditya Sanghavi — Founder & CEO, Snackible (Lightsaber Food Ventures Pvt. Ltd.)
**Company:** Snackible — Mumbai-based healthy-snacking / D2C FMCG brand, founded May 2015.
Retails via Amazon, Swiggy Instamart, Blinkit, Zepto + ~5,000 offline stores.
**LinkedIn:** https://www.linkedin.com/in/aditya-sanghavi-487b3123 (canonical profile — monitoring anchor)

**Why this matters for targeting:** his public footprint clusters in a few predictable
lanes, which is exactly what makes an automated scan effective. Known/representative
appearances:

- **Podcasts:** "Beyond The Dream" Ep 06 — *The Real Work Behind D2C* (by Brego Business).
- **Press/magazines:** Entrepreneur India, YourStory, Inc42, BW Disrupt, Verve, SiliconIndia.
- **Category:** Indian D2C / CPG / FMCG / healthy-food / startup-founder ecosystem.

> The scan should confirm and expand this footprint over time, not assume it's complete.

---

## 2. What counts as an "opportunity" (the taxonomy)

The agent classifies every hit into one of these buckets so the report is scannable:

| Bucket | Examples | Signal that it's actionable |
|---|---|---|
| 🎙️ **Podcasts** | D2C / CPG / founder / food-business shows accepting guests | "pitch a guest", guest form, host is actively booking |
| 📹 **YouTube / video** | Founder-interview channels, business shows | Open call, series still publishing |
| 🏛️ **Summits & conferences** | D2C/FMCG/retail/food-tech events in India | Call for speakers open, agenda not finalized |
| 🪑 **Panels / fireside** | Industry meetups, demo days, trade shows | Speaker application link live |
| 📰 **Magazines / editorial** | Startup & food trade press, columns, Q&As | Accepting contributions / founder features |
| 🏆 **Awards & lists** | "30 under 30", D2C/FMCG awards, founder lists | Nominations open, deadline in future |
| 🤝 **Networking / community** | Founder communities, accelerator events, trade expos | Registration or application open |

Each item also gets: **date/deadline**, **location (virtual / city)**, **fit score**, and
**the exact apply/pitch link**.

---

## 3. Sources the agent scans

Primary engine is **WebSearch** (and `web-fetch` to read promising pages). A rotating
set of query templates keeps coverage broad without being noisy:

**A. Named-event / call-for-speakers queries**
- `India D2C summit 2026 call for speakers`
- `FMCG / food-tech conference India 2026 speaker application`
- `healthy snacking / packaged food expo India speakers`

**B. Podcast-guest queries**
- `D2C founder podcast India accepting guests`
- `CPG / food business podcast guest pitch form`

**C. Awards & lists**
- `D2C founder awards India 2026 nominations open`
- `Entrepreneur / Inc42 / YourStory founder list nominations`

**D. Subject-monitoring queries** (catch new mentions + find lookalike venues)
- `"Aditya Sanghavi" Snackible` / `Snackible founder interview`
- Used both to log new coverage *and* to reverse-engineer which outlets cover this niche.
- **LinkedIn anchor:** his profile (`/in/aditya-sanghavi-487b3123`) — watch for new posts/activity
  that signal events he's attending or being invited to. Note: LinkedIn blocks anonymous
  scraping (returns HTTP 429), so this is best monitored via Google site-search
  (`site:linkedin.com "Aditya Sanghavi" Snackible <event keyword>`) or an authenticated tool,
  not direct fetch.

**E. Recurring venues watchlist** (a maintained list the agent re-checks each run)
- Entrepreneur India, YourStory, Inc42, Startup CPG-style shows, Brego Business,
  TiE / Nasscom / IVCA events, major food & retail expos. This list grows as the agent
  discovers good-fit venues.

---

## 4. How the agent runs (architecture)

```
┌─ Scheduled trigger (weekly) ─────────────────────────────┐
│                                                          │
│  1. Load watchlist + query templates                     │
│  2. Run WebSearch across all query buckets               │
│  3. web-fetch promising pages to confirm details         │
│     (dates, deadlines, apply links, guest forms)         │
│  4. Normalize → {title, bucket, venue, date, deadline,   │
│     location, link, apply_link, fit_score, why}          │
│  5. Deduplicate vs. previous runs (seen-list)            │
│  6. Score & rank by fit + urgency (deadline proximity)   │
│  7. Write report + append to running log                 │
│  8. Deliver (file / email / message)                     │
└──────────────────────────────────────────────────────────┘
```

**Implementation in this environment:** use the **`schedule`** skill /
`mcp__scheduled-tasks__create_scheduled_task` to register a recurring cloud agent whose
prompt is the scan-and-report routine above. State (the "seen" list + venue watchlist)
is kept in simple files in this project folder so each run dedupes against the last.

**Fit score (0–100)** — simple weighted heuristic:
- Audience match (D2C / FMCG / food / founder) — 40
- Reach / prestige of venue — 20
- Actionability (open application vs. invite-only) — 25
- Urgency (deadline soon but not passed) — 15

---

## 5. Report format (what gets delivered)

```
# Networking Opportunities — Week of <date>

## 🔥 Act now (deadline within 14 days)
- [Event/Show name] — bucket · location · DEADLINE <date>
  Why it fits: <one line> · Fit: 86/100
  → Apply/pitch: <direct link>

## 🟢 New this week
...grouped by bucket (Podcasts / Summits / Awards / ...)

## 📌 Still open (carried over)
...

## 🗂️ Logged for reference (lower fit)
...

Scanned N sources · M new · K dropped as duplicates
```

Delivery options (pick one to start): write the report as a dated Markdown file in this
folder, **or** email it to you, **or** post to Slack. Email/Slack need a one-time connector
auth; the file option works immediately.

---

## 6. Cadence

- **Default: weekly** (e.g. Monday 9:00 AM) — enough to catch call-for-speaker windows
  without flooding you.
- Optional **monthly deep sweep** with broader, extended-mode searches for smaller/regional
  events.
- Deadlines inside items are what create urgency, not the run frequency.

---

## 7. Setup steps (to build it)

1. **Confirm the subject profile** — lock Aditya Sanghavi + Snackible keywords, and add any
   personal angles you want emphasized (e.g. healthy snacking, D2C ops, bootstrapped growth).
2. **Seed the watchlist file** — the recurring venues in §3E.
3. **Write the agent prompt** — the §4 routine, with the §2 taxonomy and §5 output format baked in.
4. **Choose delivery** — file (instant) vs. email/Slack (needs auth).
5. **Register the schedule** — weekly cloud task via the `schedule` skill.
6. **Run once manually** — review the first report, tune queries and fit weights.
7. **Let it run** — review weekly; prune/add venues as the watchlist learns.

---

## 8. Guardrails & notes

- **Links are mandatory.** Every item carries a real, fetched URL; the agent should drop
  anything it can't link.
- **Verify before it acts.** The agent only *reports* opportunities — it does not submit
  applications, send pitches, or create accounts. You review and act.
- **Dedupe is essential** — without the seen-list, the same evergreen podcast shows up every week.
- **Freshness** — prefer results from the last ~60 days for events; awards/lists are checked
  against their nomination deadlines.
- **LinkedIn:** canonical profile confirmed (`/in/aditya-sanghavi-487b3123`). Direct fetching is
  rate-limited/blocked, so monitor via Google site-search or an authenticated connector.
- **Still useful to confirm:** any personal site, X/Instagram handles, or a company press/PR
  contact to broaden subject-monitoring.

---

### Sources (initial research)
- [Entrepreneur India — Aditya Sanghavi profile](https://india.entrepreneur.com/en-in/entrepreneurs/curiosity-into-creativity-translator/474300)
- [YourStory — Snackible, "new-age Haldirams"](https://yourstory.com/2016/02/snackible)
- [YourStory — Snackible pivot to FMCG](https://ts.yourstory.com/2017/07/snackible-startup-healthy-snacks-business-model)
- [BW Disrupt — Snackible founder feature](https://bwdisrupt.com/article/snackible-a-solution-for-all-snackaholics-to-experience-healthier-snacking-100443)
- [Beyond The Dream podcast — Ep 06 ft. Aditya Sanghavi / Snackible](https://www.buzzsprout.com/2449840/episodes/17356637-beyond-the-dream-ep-06-the-real-work-behind-d2c-aditya-sanghavi-ft-snackible)
- [CB Insights — Lightsaber Food Ventures (Snackible) people](https://www.cbinsights.com/company/lightsaber-food-ventures/people)
