---
name: channel-playbook
version: 1.0.0
description: |
  Defines when to use which communication channel and the cadence for each
  pipeline stage. Call-first priority. Read this skill before deciding whether
  to call, email, or SMS a lead or customer. Also defines demo prep flow.
allowed-tools:
  - Read
  - WebFetch
---

# Channel Playbook

Call first. Every new lead gets a call attempt before any email. Call is always the highest-priority action.

## Core Rules

- **Call first.** Always.
- **Same-day follow-up.** After a missed call, send email the same day. Don't wait.
- **SMS is situational.** Not in any automated flow. Miguel decides when to text.
- **Sequence stops on:** reply, meeting booked, or stage change.

---

## Channel Routing by Stage

### New

| Priority | Action | Timing |
|---|---|---|
| 1 | Call | Immediately |
| 2 | Email (personalized welcome) | Same day if no answer |
| 3 | Enroll in sequence | If no reply after email |

### Demo

| Priority | Action | Timing |
|---|---|---|
| 1 | Confirmation email + full prospect brief | When demo is booked |
| 2 | Pre-demo prep notes | Day before demo (or same day if booked same day) |
| — | *If demo missed:* | |
| 1 | Call to reschedule | Same day |
| 2 | Email with booking link | Same day if no answer |

### Trial

| Priority | Action | Timing |
|---|---|---|
| 1 | Welcome email + schedule setup call | Day 0 |
| 2 | Activation check call | Day 3 |
| 3 | Check-in if not activated | Day 5 |
| 4 | Trial expiry warning + conversion nudge | Day 8 |
| 5 | Call (highest-leverage moment) | Day 10 (trial ends) |
| 6 | Email with conversion pitch | Same day if no answer |

### Onboarding

| Priority | Action | Timing |
|---|---|---|
| 1 | Welcome email + schedule setup call | Day 0 |
| 2 | Setup call | Day 1-3 |
| 3 | Activation check call | Day 3 |
| 4 | Check-in if not activated | Day 5 |
| 5 | Mid-onboarding check-in | Day 7 |
| 6 | Move to Active if activated | Day 14 |

### Active

| Priority | Action | Timing |
|---|---|---|
| 1 | Monthly check-in email | Every 30 days |
| — | *If interest_level = Cold (At Risk):* | |
| 1 | Call first | Immediately |
| 2 | Empathetic email | Same day if no answer |

### Churned

| Priority | Action | Timing |
|---|---|---|
| 1 | Call for feedback | When cancellation confirmed |
| 2 | Email asking what happened | Same day if no answer |

### Lost

| Priority | Action | Timing |
|---|---|---|
| 1 | Add to nurture sequence | When moved to Lost |

---

## Demo Prep Flow

**Trigger:** Demo appears on calendar (Google Calendar event detected or manual booking).

**Agent produces a full prospect brief containing:**

1. **Company overview** — WebFetch their website. What they do, industry, who they serve.
2. **HubSpot data** — Lead source, deal history, notes, previous interactions, any emails exchanged.
3. **Pain points** — Likely pain points based on their industry vertical:
   - Home services → missed calls = missed jobs
   - Law firms → intake calls, new vs. existing case routing
   - Healthcare → appointment booking, after-hours calls
   - Retail → order inquiries, store hours, customer support
   - Professional services → lead capture, scheduling
4. **Suggested talking points** — How Central solves their specific pain points.
5. **Contact context** — Name, role (if known), company size (if known), how they found Central.

**Delivery:** Brief delivered day before the demo. Same day if booked same day.

**Post-demo flow:**
- Follow-up email within 24 hours (reference something specific from the demo)
- If no reply in 48 hours, call
- Proposal within 3 days if qualified

---

## Cadence Rules

| Scenario | Wait Time |
|---|---|
| After missed call → email | Same day |
| After email → follow-up call | 48 hours |
| After demo → follow-up email | Within 24 hours |
| After demo → proposal | Within 3 days |
| Between sequence emails | 3 days minimum |
| Monthly check-in (Active) | Every 30 days |
| After proposal sent → follow-up | 3 days |
