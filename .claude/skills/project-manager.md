---
name: project-manager
version: 1.0.0
description: |
  Detailed briefing format, reporting structure, and task management rules
  for Claude's PM role. CLAUDE.md defines the role and default behavior.
  This skill provides the detailed specs for how to present briefings,
  prioritize tasks, and format reports. Read when producing any daily
  summary, task list, or EOD recap.
allowed-tools:
  - Read
  - mcp__claude_ai_HubSpot__search_crm_objects
  - mcp__claude_ai_HubSpot__get_crm_objects
  - mcp__claude_ai_Gmail__gmail_search_messages
  - mcp__claude_ai_Google_Calendar__gcal_list_events
  - mcp__claude_ai_Slack__slack_read_channel
  - mcp__claude_ai_Slack__slack_search_public_and_private
---

# Project Manager

Detailed specs for Claude's daily PM role. CLAUDE.md says "you are the PM." This skill says "here's exactly how."

---

## Morning Briefing Format

Present in this order:

### 1. Yesterday's Activity

Query HubSpot and Gmail for the previous business day (midnight to midnight PT).

| Metric | Source |
|---|---|
| Emails sent | Gmail: count of sent messages from miguel@trycentral.com |
| Calls made | HubSpot: call engagements logged |
| Deals created | HubSpot: deals with createdate = yesterday |
| Deals moved | HubSpot: deals with stage change yesterday |
| Sequences enrolled | HubSpot: enrollment activities |
| Email drafts created | Gmail: drafts created yesterday |

Present as a compact summary:
```
YESTERDAY (Apr 3, PT)
Emails sent: 12 | Calls: 3 | Deals created: 2 | Stage moves: 4
```

### 2. Pipeline Snapshot

Query HubSpot for current state across all active stages.

| Item | How to find |
|---|---|
| New leads awaiting first call | Stage = New, call_attempted = false |
| Trials expiring within 3 days | Stage = Trial, trial_end_date within 3 days |
| Stale deals (5+ days no activity) | Any active stage, no HubSpot activity in 5+ days |
| At-risk customers | Stage = Active, interest_level = Cold |
| Demos scheduled today | Stage = Demo, demo booked for today (Google Calendar) |
| Onboarding check-ins due | Stage = Onboarding, check-in milestone approaching |

Present as a status board:
```
PIPELINE
New: 63 (12 awaiting first call)
Demo: 19 (2 today)
Trial: 19 (5 expiring within 3 days)
Onboarding: 2
Active: 40 (3 at risk)
Stale deals: 8
```

### 3. Today's Task List

Prioritized per CLAUDE.md rules. Include specific deal names and contact info.

```
TODAY'S PRIORITIES
1. EXPIRING TRIALS (call first)
   - Phoenique Nicole — trial ends Apr 4, $79 plan
   - Simon Fletcher — trial ends Apr 4, $79 plan

2. DEMOS TODAY
   - Tiffany (Trinity Lab) — 11am PT, Getting Started with Central

3. NEW LEADS — CALL ASAP
   - Suncoast AI Consulting — signed up Apr 3, no call yet
   - Amanda Zenquis — signed up Apr 2, no call yet

4. STALE DEALS
   - Leonard Duru — 8 days no activity, New stage
   ...
```

---

## Task Priority Rules (detailed)

CLAUDE.md defines the order. This skill adds specifics:

1. **Expiring trials** — list by days remaining, closest first. Include plan tier and amount.
2. **Demos today** — list by time, include prospect brief link if available.
3. **New leads awaiting first call** — list by signup date, oldest first (longest wait = highest risk).
4. **Stale deals** — list by days since last activity, most stale first.
5. **At-risk accounts** — list by interest_level = Cold in Active stage.
6. **Onboarding check-ins** — list by days in Onboarding, approaching milestone dates.
7. **Growth opportunity outreach** — Active 90+ days, interest = Hot or Warm.

---

## EOD Recap Format

When Miguel asks for a recap or ends the session:

```
END OF DAY — Apr 4, PT

COMPLETED
- Called 5 new leads (3 no answer, 2 connected)
- Sent 8 follow-up emails
- Moved 2 deals: Noah Tovar New→Trial, Paul Artis New→Trial
- Demo with Tiffany (Trinity Lab) — went well, scheduling trial

ROLLED OVER
- 3 new leads still need first call (Ricky Jones, Jeffrey Holmboe, Carol Sinclair)
- Stale deals not addressed: 5

NEEDS ATTENTION TOMORROW
- Phoenique Nicole trial expires tomorrow
- Tiffany follow-up email needed
```

---

## Task Checklist Management

- Maintain a running checklist throughout the session
- Mark tasks complete as they're done
- If Miguel is about to end the session, surface incomplete tasks
- Don't create tasks for things that are already done
- Group tasks by priority level, not by type

---

## Reporting Cadence

- **Morning briefing:** Start of every session (or first session of the day)
- **Mid-day status:** If Miguel asks "what's left?" or starts a new session mid-day
- **EOD recap:** When Miguel asks or signals end of session
- **On-demand:** Any time Miguel asks "what's for today?" or similar

---

## Data Sources

| Data | Source | Tool |
|---|---|---|
| Deal pipeline state | HubSpot | search_crm_objects |
| Deal details | HubSpot | get_crm_objects |
| Emails sent/received | Gmail | gmail_search_messages |
| Calendar events | Google Calendar | gcal_list_events |
| Slack activity | Slack | slack_read_channel, slack_search |
| Call engagements | HubSpot | search_crm_objects (engagements) |

**Time context:**
- All dates and times in Pacific Time
- "Yesterday" = previous business day (skip weekends)
- Miguel is in Philippine Time — adjust when referencing his local time
