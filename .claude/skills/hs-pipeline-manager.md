---
name: hs-pipeline-manager
version: 1.0.0
description: |
  Single source of truth for HubSpot pipeline structure. Read this skill before
  creating, updating, or querying any HubSpot deal, contact, or company record.
  Contains pipeline stages, property definitions, transition rules, hygiene
  guardrails, tag computations, and association rules.
allowed-tools:
  - Read
---

# HubSpot Pipeline Manager

Every agent reads this before touching HubSpot. One pipeline, one deal per customer.

## HubSpot Configuration

- **Owner ID:** 163091353 (Miguel Dungo, miguel@trycentral.com)
- **Hub ID:** 242826512
- **URL pattern:** `https://app.hubspot.com/contacts/242826512/record/0-{objectTypeId}/{id}`
  - Contacts: objectTypeId = 1
  - Companies: objectTypeId = 2
  - Deals: objectTypeId = 3

---

## Pipeline Stages

Pipeline ID: `default`

| Stage | ID | Probability | Description |
|---|---|---|---|
| New | `appointmentscheduled` | 10% | Lead entered pipeline. No qualification yet. Can come from Slack, calendar, email, or direct signup. |
| Demo | `1743510241` | 40% | Demo or discovery call in progress. Check `demo_status` property for state. |
| Trial | `qualifiedtobuy` | 60% | Free 10-day trial active. Product is live. Check `trial_end_date` and `activation_status`. |
| Onboarding | `3453902565` | 80% | Paying customer getting set up. 14-day window to reach activation. |
| Active | `3453959920` | 100% | Fully active, paying, engaged customer. Check `interest_level` for health. |
| Lost | `3454079706` | 0% | Prospect declined or went silent 14+ days. Terminal. |
| Churned | `3364536008` | 0% | Customer cancelled or suspended. Terminal. |

**Non-linear entry:** Deals can skip stages. A trial signup enters at Trial, not New.

**One deal per customer.** Multiple products tracked via `product_type` multi-select.

---

## Deal Properties (12 custom)

| # | Property | Type | Options | Purpose |
|---|---|---|---|---|
| 1 | `signup_date` | Date | — | When they first signed up |
| 2 | `lead_source` | Dropdown | Slack-Trial, Slack-Signup, Slack-Feedback, Google Calendar, Inbound Email, Other | Where they came from |
| 3 | `interest_level` | Dropdown | Hot, Warm, Cold | Engagement level. Replaces Qualified and At Risk stages. |
| 4 | `call_attempted` | Checkbox | true/false | Has a call been made |
| 5 | `demo_status` | Dropdown | None, Booked, Completed, Missed | Demo state. Replaces Demo Done and Missed Demo stages. |
| 6 | `activation_status` | Dropdown | Not Started, Partial, Activated | Product setup progress |
| 7 | `next_action_date` | Date | — | When the next touch should happen |
| 8 | `trial_end_date` | Date | — | Trial expiry (signup_date + 10 days) |
| 9 | `product_type` | Multi-select | Voice, Chat, Caller, CRM, EA | Which Central products they use (Voice=AI Receptionist, Chat=AI Chat, Caller=AI Sales, CRM=CRM, EA=Executive Assistant) |
| 10 | `plan_tier` | Dropdown | Free, Starter, Growth, Scale, Enterprise | Current plan |
| 11 | `original_plan` | Dropdown | Free, Starter, Growth, Scale, Enterprise | Starting plan (track upgrades) |
| 12 | `mrr` | Number | — | Current monthly recurring revenue |

### Minimum Viable Record

Every deal must have at minimum:
- `signup_date` set
- `lead_source` set
- Contact associated
- Deal name = "{firstname} {lastname}"

---

## Stage Transition Rules

### New > Demo
- Trigger: Demo booked (Google Calendar event detected or manual)
- Properties to set: `demo_status` = booked, `next_action_date` = demo date

### New > Trial
- Trigger: Trial signup detected (Slack channel or direct)
- Properties to set: `trial_end_date` = signup_date + 10 days, `activation_status` = not_started
- Note: Non-linear entry. Lead skipped Demo.

### Demo > Trial
- Trigger: Trial started after demo
- Properties to set: `demo_status` = completed (if not already), `trial_end_date` = now + 10 days, `activation_status` = not_started

### Demo > Lost
- Trigger: Prospect declined or ghosted. `interest_level` = cold AND `next_action_date` overdue by 14+ days.

### Trial > Onboarding
- Trigger: Customer converts to paid plan
- Properties to set: `plan_tier`, `original_plan` (same as plan_tier at first conversion), `mrr`, `product`

### Trial > Lost
- Trigger: Trial expired without conversion. `trial_end_date` passed AND no payment.

### Onboarding > Active
- Trigger: `activation_status` = activated AND 14+ days since entering Onboarding
- Properties to set: `next_action_date` = 30 days from now (first monthly check-in)

### Active > Churned
- Trigger: Confirmed cancellation via customer-feedback Slack channel
- Autonomous: Yes, agent can move to Churned on confirmed cancellation

### Any stage > Lost
- Trigger: Prospect/customer declined, ghosted 14+ days, or explicitly said no
- Note: For Active customers, use Churned instead of Lost

---

## Deal Tags (auto-generated)

Tags are computed from properties. Never set manually.

| Tag | Condition | Stages |
|---|---|---|
| Call ASAP | `call_attempted` = false AND `signup_date` > 1 day ago | New |
| Going Cold | `interest_level` = cold AND `next_action_date` overdue | New, Trial, Active |
| Missed Demo | `demo_status` = missed | Demo |
| Awaiting Follow-up | `demo_status` = completed AND no follow-up note within 24h | Demo |
| Demo Today | `demo_status` = booked AND demo date = today | Demo |
| Trial Expiring | `trial_end_date` within 3 days | Trial |
| Needs Activation | `activation_status` = not_started AND 3+ days in stage | Trial, Onboarding |
| Ready for Active | `activation_status` = activated AND 14+ days in Onboarding | Onboarding |
| At Risk | `interest_level` = cold | Active |
| Growth Opportunity | `signup_date` > 90 days ago AND `interest_level` = hot or warm | Active |
| Stale | No HubSpot activity in 5+ days | All active stages |

---

## Hygiene Guardrails

### Duplicate Prevention
- Before creating any contact, search by email first
- Before creating any company, search by domain first
- Before creating any deal, check if the contact already has an active deal
- One deal per customer. If contact already has a deal, update it. Do not create a second one.

### Blocked Domains (never create as leads)
trycentral.com, getwingapp.com, m32.ai, yopmail.com

### Bot/Test Filters
Skip any lead where:
- Email prefix matches: noreply, no-reply, bot, test, demo, hello, info, support, admin, postmaster, mailer-daemon, donotreply, do-not-reply
- Email or name contains: test, bot, automated, dummy, fake
- Email is blank, entirely numeric, or appears to be a UUID/hash

### Personal Email Providers (do NOT extract as company domain)
gmail.com, yahoo.com, hotmail.com, outlook.com, icloud.com, aol.com, protonmail.com, me.com, mac.com, live.com, msn.com, ymail.com, googlemail.com

### Stale Deal Threshold
5+ days with no logged HubSpot activity (note, task, email, call).

### Association Rule
All contacts under a company auto-associate to the company's deal. When a new contact is created under an existing company, add them to the existing deal.

---

## Channel-to-Stage Mapping

| Source | Default Stage | Default Properties |
|---|---|---|
| Slack: `gtm-new-trial-info` | Trial (`qualifiedtobuy`) | lead_source = slack-trial, trial_end_date = signup + 10 days, activation_status = not_started |
| Slack: `central-new-signups` | New (`appointmentscheduled`) | lead_source = slack-signup |
| Slack: `customer-feedback` | Varies (see below) | lead_source = slack-feedback |
| Google Calendar (demo booked) | Demo (`1743510241`) | lead_source = google-calendar, demo_status = booked |
| Gmail (unknown sender) | New (`appointmentscheduled`) | lead_source = inbound-email |

### customer-feedback routing
- If contact has active deal in Active → update existing deal, check sentiment
- If contact has active deal in any other stage → update existing deal
- If no contact exists → create at New stage
- Positive feedback → no stage change
- Negative feedback → set `interest_level` = cold
- Cancellation confirmed → move to Churned

---

## Sequence Definitions

| Stage | Sequence | Cadence | Stop Conditions |
|---|---|---|---|
| New | Welcome / nurture | 3 emails over 7 days | Reply, meeting booked, stage change |
| Demo (missed) | Reschedule | 2 emails over 5 days | Reply, demo rebooked |
| Trial | Onboarding nudge | Day 3, Day 5, Day 8 | Reply, activation, stage change |
| Active | Monthly check-in | 1 email per 30 days | Reply |

Sequence email copy comes from the `writing-voice` skill, not this skill.

## Note Requirements

Every stage change must be logged with a HubSpot note containing:
- What changed (e.g., "Moved from New to Demo")
- Why (e.g., "Demo booked via Google Calendar for April 10")
- Written in Miguel's voice (agents read `writing-voice` skill for tone)

Notes written by agents must NOT look like bot logs. They should read as if Miguel wrote them.

---

## Key Definitions

- **ICP** — 1-25 employees, phone-dependent SMB, cannot afford a full-time receptionist, loses revenue from missed calls. Priority verticals: real estate, law firms, home services, beauty and wellness, healthcare, financial services.
- **Trial** — 10-day free trial. Product live on prospect's number. Tracked via `trial_end_date`.
- **Activation** — Calendar connected AND first real call received through Central AI. Tracked via `activation_status`.
- **At Risk** — Not a stage. An Active customer with `interest_level` = cold.
- **Growth Opportunity** — Not a stage. Active 90+ days with `interest_level` = hot or warm.
