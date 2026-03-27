# Pipeline & Deal Stage Guide

This file defines how the slack-lead-capture skill routes leads to HubSpot pipelines and manages deal stage transitions. The skill reads this file before creating or updating any deal.

---

## Pipeline Routing Rules

- `gtm-new-trial-info` → **Sales pipeline**, stage: **Trial Started**
  - These are prospects who have activated a free trial. Product is live on their number.
- `central-new-signups` → **Sales pipeline**, stage: **New Lead**
  - New inbound signups. No qualification yet. Skill should note these may include bots/fake accounts — flag if the name/email looks suspicious.
- `customer-feedback` → **Accounts pipeline** (if existing customer) or **Sales pipeline / Qualified** (if prospect)
  - If the contact already exists in HubSpot with an active Accounts deal → update that deal's stage based on sentiment (see rules below).
  - If the contact does not exist or only has a Sales deal → create/update in Sales pipeline at Qualified stage.

---

## Sales Pipeline Stages

Pipeline ID: `default`

| Stage | ID | Description |
|-------|-----|-------------|
| New Lead | `appointmentscheduled` | Initial entry — inbound, outbound, or marketing. No qualification yet. |
| Qualified | `1743936187` | ICP-fit confirmed — right vertical, decision maker, inbound call volume, clear pain point. |
| Demo Scheduled | `1743510241` | Discovery/demo call booked. |
| Demo Cancelled | `3374807784` | Prospect no-showed. Requires re-engagement. |
| Trial Started | `qualifiedtobuy` | Free trial active. Product live on their number. |
| Trial Ended | `3371584247` | Trial expired. Highest-leverage conversion window. |
| Proposal Sent | `3371149048` | Specific plan recommendation formally presented. |
| Closed Won | `3364536007` | Paid plan committed. Hand off to Accounts pipeline at Onboarding. |
| Closed Lost | `3364536008` | Declined or silent for 14+ days. Move to nurture. |

## Accounts Pipeline Stages

Pipeline ID: `2123072201`

| Stage | ID | Description | Transition Rule |
|-------|-----|-------------|-----------------|
| Onboarding | `3364559579` | New paying customer getting set up. | Moves to Healthy after 14 days. |
| Healthy | `3364559580` | Active, paying, engaging regularly. | CS reviews at 90 days for Growth Opportunity. |
| At Risk | `3364559581` | Danger signals — missed payment, no logins, dropped usage. | CS moves manually when signals appear. |
| Growth Opportunity | `3364559582` | Strong cross-sell candidate. | CS confirms after 90-day Healthy flag. |
| Churned | `3364559583` | Cancelled or suspended. | CS moves manually after cancellation confirmed. |

---

## Channel-to-Stage Mapping

| Slack Channel | Pipeline | Stage | Notes |
|---------------|----------|-------|-------|
| `gtm-new-trial-info` | Sales (`default`) | Trial Started (`qualifiedtobuy`) | Trial is live |
| `central-new-signups` | Sales (`default`) | New Lead (`appointmentscheduled`) | Flag suspicious signups |
| `customer-feedback` | Accounts (`2123072201`) | Based on sentiment (see below) | Only if existing customer; else Sales / Qualified |

### customer-feedback Sentiment Rules

- **Positive feedback** → if in Accounts pipeline, no stage change (already Healthy or better)
- **Negative feedback / complaints** → move to or flag as **At Risk** (`3364559581`)
- **Feature requests / suggestions** → no stage change, note in deal description
- **Cancellation signals** → move to **Churned** (`3364559583`) if confirmed, else **At Risk**
