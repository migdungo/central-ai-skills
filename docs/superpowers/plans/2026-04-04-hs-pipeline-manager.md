# hs-pipeline-manager Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create the `hs-pipeline-manager` skill file — the single source of truth for all HubSpot pipeline structure, properties, and rules.

**Architecture:** One skill file that every agent reads before touching HubSpot. Contains pipeline stages, property definitions, transition rules, hygiene guardrails, and tag computations. No workflow logic — just the rulebook.

**Tech Stack:** Markdown skill file (.claude/skills/), HubSpot MCP tools

---

## Task 1: Create HubSpot custom deal properties

Before writing the skill file, the 12 custom properties must exist in HubSpot. This task creates them via the HubSpot MCP.

**Files:**
- None (HubSpot configuration only)

- [ ] **Step 1: Create `signup_date` property**

```
manage_crm_objects: create property
objectType: deals
name: signup_date
label: Signup Date
type: date
groupName: dealinformation
```

- [ ] **Step 2: Create `lead_source` property**

```
manage_crm_objects: create property
objectType: deals
name: lead_source
label: Lead Source
type: enumeration
options: slack-trial, slack-signup, slack-feedback, google-calendar, inbound-email
groupName: dealinformation
```

- [ ] **Step 3: Create `interest_level` property**

```
manage_crm_objects: create property
objectType: deals
name: interest_level
label: Interest Level
type: enumeration
options: hot, warm, cold
groupName: dealinformation
```

- [ ] **Step 4: Create `call_attempted` property**

```
manage_crm_objects: create property
objectType: deals
name: call_attempted
label: Call Attempted
type: bool
groupName: dealinformation
```

- [ ] **Step 5: Create `demo_status` property**

```
manage_crm_objects: create property
objectType: deals
name: demo_status
label: Demo Status
type: enumeration
options: none, booked, completed, missed
groupName: dealinformation
```

- [ ] **Step 6: Create `activation_status` property**

```
manage_crm_objects: create property
objectType: deals
name: activation_status
label: Activation Status
type: enumeration
options: not_started, partial, activated
groupName: dealinformation
```

- [ ] **Step 7: Create `next_action_due` property**

```
manage_crm_objects: create property
objectType: deals
name: next_action_due
label: Next Action Due
type: date
groupName: dealinformation
```

- [ ] **Step 8: Create `trial_end_date` property**

```
manage_crm_objects: create property
objectType: deals
name: trial_end_date
label: Trial End Date
type: date
groupName: dealinformation
```

- [ ] **Step 9: Create `product` property**

```
manage_crm_objects: create property
objectType: deals
name: product
label: Product
type: enumeration (multi-select)
options: AI Receptionist, AI Chat, AI Sales, EA
groupName: dealinformation
```

- [ ] **Step 10: Create `plan_tier` property**

```
manage_crm_objects: create property
objectType: deals
name: plan_tier
label: Plan Tier
type: enumeration
options: Free, Starter, Growth, Scale, Enterprise
groupName: dealinformation
```

- [ ] **Step 11: Create `original_plan` property**

```
manage_crm_objects: create property
objectType: deals
name: original_plan
label: Original Plan
type: enumeration
options: Free, Starter, Growth, Scale, Enterprise
groupName: dealinformation
```

- [ ] **Step 12: Create `mrr` property**

```
manage_crm_objects: create property
objectType: deals
name: mrr
label: MRR
type: number
groupName: dealinformation
```

- [ ] **Step 13: Verify all 12 properties exist**

Query HubSpot to confirm all properties were created successfully.

---

## Task 2: Restructure HubSpot pipeline

Modify the existing Sales pipeline to match the 7-stage unified pipeline. This must be done carefully to avoid losing existing deal data.

**Files:**
- None (HubSpot configuration only)

- [ ] **Step 1: Document current pipeline state**

Query HubSpot for all current pipeline stages and their IDs in both Sales (`default`) and Accounts (`2123072201`) pipelines. Record the results.

- [ ] **Step 2: Add new stages to Sales pipeline**

Add these stages to the `default` pipeline (they don't exist yet):
- Onboarding
- Active
- Churned

Record the new stage IDs returned by HubSpot.

- [ ] **Step 3: Migrate Accounts pipeline deals**

For each deal in the Accounts pipeline (`2123072201`):
- Onboarding deals → move to new Onboarding stage in `default` pipeline
- Healthy deals → move to new Active stage in `default` pipeline
- At Risk deals → move to new Active stage in `default` pipeline, set `interest_level` = cold
- Growth Opportunity deals → move to new Active stage, note Growth Opportunity status
- Churned deals → move to new Churned stage in `default` pipeline

**IMPORTANT:** Log every move. Do not delete the Accounts pipeline until all deals are migrated and verified.

- [ ] **Step 4: Consolidate Sales pipeline stages**

Move deals from removed stages to their new equivalents:
- New Signups → New Lead (which becomes "New")
- Qualified → New (set `interest_level` = hot)
- Demo Done → Demo Scheduled (set `demo_status` = completed)
- Missed Demo → Demo Scheduled (set `demo_status` = missed)
- Trial Ended → Trial Started (compute `trial_end_date` from deal age)
- Proposal Sent → Trial Started (log proposal in notes)
- Closed Won → Onboarding

- [ ] **Step 5: Rename remaining stages**

- New Lead → New
- Demo Scheduled → Demo
- Trial Started → Trial
- Closed Lost → Lost

- [ ] **Step 6: Remove old stages**

After confirming all deals have been migrated, remove:
- New Signups, Qualified, Demo Done, Missed Demo, Trial Ended, Proposal Sent, Closed Won

- [ ] **Step 7: Archive Accounts pipeline**

After confirming all Accounts deals are in the unified pipeline, archive `2123072201`.

- [ ] **Step 8: Verify final pipeline state**

Query HubSpot and confirm:
- 7 stages exist (New, Demo, Trial, Onboarding, Active, Churned, Lost)
- All deals accounted for
- No orphaned deals in the old Accounts pipeline

Record all final stage IDs.

---

## Task 3: Write the hs-pipeline-manager skill file

Using the stage IDs from Task 2, write the skill file.

**Files:**
- Create: `.claude/skills/hs-pipeline-manager.md`

- [ ] **Step 1: Write skill frontmatter and overview**

```markdown
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

This skill defines all HubSpot pipeline structure for Central AI. Every agent reads this before touching HubSpot.

## HubSpot Configuration

- **Owner ID:** 163091353 (Miguel Dungo, miguel@trycentral.com)
- **Hub ID:** 242826512
- **URL pattern:** `https://app.hubspot.com/contacts/242826512/record/0-{objectTypeId}/{id}`
  - Contacts: objectTypeId = 1
  - Companies: objectTypeId = 2
  - Deals: objectTypeId = 3
```

- [ ] **Step 2: Write pipeline stages section**

Replace `<ID>` placeholders with actual IDs from Task 2 Step 8.

```markdown
## Pipeline Stages

Pipeline ID: `default`

| Stage | ID | Description | Probability |
|---|---|---|---|
| New | `<ID>` | Lead entered pipeline. No qualification yet. Can come from Slack, calendar, email, or direct signup. | 10% |
| Demo | `<ID>` | Demo or discovery call in progress. Check `demo_status` property for state (booked/completed/missed). | 40% |
| Trial | `<ID>` | Free 10-day trial active. Product is live. Check `trial_end_date` and `activation_status`. | 60% |
| Onboarding | `<ID>` | Paying customer getting set up. 14-day window to reach activation. | 80% |
| Active | `<ID>` | Fully active, paying, engaged customer. Check `interest_level` for health. | 100% |
| Churned | `<ID>` | Cancelled or suspended. Terminal state. | 0% |
| Lost | `<ID>` | Prospect declined or went silent. Terminal state. | 0% |

**Non-linear entry:** Deals can skip stages. A signup that immediately starts a trial enters at Trial, not New.
```

- [ ] **Step 3: Write deal properties section**

```markdown
## Deal Properties (12 custom)

| # | Property | Type | Values | Purpose |
|---|---|---|---|---|
| 1 | `signup_date` | Date | — | When they first signed up |
| 2 | `lead_source` | Dropdown | slack-trial, slack-signup, slack-feedback, google-calendar, inbound-email | Where they came from |
| 3 | `interest_level` | Dropdown | hot, warm, cold | Engagement level. Replaces Qualified and At Risk stages. |
| 4 | `call_attempted` | Checkbox | true/false | Has a call been made |
| 5 | `demo_status` | Dropdown | none, booked, completed, missed | Demo state. Replaces Demo Done and Missed Demo stages. |
| 6 | `activation_status` | Dropdown | not_started, partial, activated | Product setup progress |
| 7 | `next_action_due` | Date | — | When the next touch should happen |
| 8 | `trial_end_date` | Date | — | Trial expiry (signup_date + 10 days) |
| 9 | `product` | Multi-select | AI Receptionist, AI Chat, AI Sales, EA | Which Central products they use |
| 10 | `plan_tier` | Dropdown | Free, Starter, Growth, Scale, Enterprise | Current plan |
| 11 | `original_plan` | Dropdown | Free, Starter, Growth, Scale, Enterprise | Starting plan (track upgrades) |
| 12 | `mrr` | Number | — | Current monthly recurring revenue |

### Plan Tier → MRR Mapping

| Plan | Default MRR |
|---|---|
| Free | $0 |
| Starter | $79-$199 (depends on product) |
| Growth | $149-$599 (depends on product) |
| Scale | $299 |
| Enterprise | Custom — set manually |

### Minimum Viable Record

Every deal must have at minimum:
- `signup_date` set
- `lead_source` set
- Contact associated
- Deal name = "{firstname} {lastname}"
```

- [ ] **Step 4: Write stage transition rules**

```markdown
## Stage Transition Rules

### New → Demo
- **Trigger:** Demo booked (Google Calendar event detected or manual)
- **Properties to set:** `demo_status` = booked, `next_action_due` = demo date
- **Note:** Deal can skip Demo entirely if lead goes straight to trial

### New → Trial
- **Trigger:** Trial signup detected (Slack channel or direct)
- **Properties to set:** `trial_end_date` = signup_date + 10 days, `activation_status` = not_started
- **Note:** Non-linear entry. Lead skipped Demo.

### Demo → Trial
- **Trigger:** Trial started after demo
- **Properties to set:** `demo_status` = completed (if not already), `trial_end_date` = now + 10 days, `activation_status` = not_started

### Demo → Lost
- **Trigger:** Prospect declined or ghosted. `interest_level` = cold AND `next_action_due` overdue by 14+ days.
- **Properties to set:** Log loss reason in notes

### Trial → Onboarding
- **Trigger:** Customer converts to paid plan
- **Properties to set:** `plan_tier`, `original_plan` (same as plan_tier at first conversion), `mrr`, `product`

### Trial → Lost
- **Trigger:** Trial expired without conversion. `trial_end_date` passed AND no payment.
- **Properties to set:** Log loss reason in notes

### Onboarding → Active
- **Trigger:** `activation_status` = activated AND 14+ days since entering Onboarding
- **Properties to set:** `next_action_due` = 30 days from now (first monthly check-in)

### Active → Churned
- **Trigger:** Confirmed cancellation via customer-feedback Slack channel
- **Autonomous:** Yes — agent can move to Churned on confirmed cancellation
- **Properties to set:** Log churn reason in notes

### Any stage → Lost
- **Trigger:** Prospect/customer declined, ghosted 14+ days, or explicitly said no
- **Note:** For Active customers, use Churned instead of Lost
```

- [ ] **Step 5: Write deal tags section**

```markdown
## Deal Tags (auto-generated)

Tags are computed from properties. Never set manually.

| Tag | Condition | Stages |
|---|---|---|
| Call ASAP | `call_attempted` = false AND `signup_date` > 1 day ago | New |
| Going Cold | `interest_level` = cold AND `next_action_due` overdue | New, Trial, Active |
| Missed Demo | `demo_status` = missed | Demo |
| Awaiting Follow-up | `demo_status` = completed AND no follow-up note within 24h | Demo |
| Demo Today | `demo_status` = booked AND demo date = today | Demo |
| Trial Expiring | `trial_end_date` within 3 days | Trial |
| Needs Activation | `activation_status` = not_started AND 3+ days in stage | Trial, Onboarding |
| Ready for Active | `activation_status` = activated AND 14+ days in Onboarding | Onboarding |
| At Risk | `interest_level` = cold | Active |
| Growth Opportunity | `signup_date` > 90 days ago AND `interest_level` = hot or warm | Active |
| Stale | No HubSpot activity in 5+ days | All active stages |
```

- [ ] **Step 6: Write hygiene guardrails section**

```markdown
## Hygiene Guardrails

### Duplicate Prevention
- Before creating any contact, search by email first
- Before creating any company, search by domain first
- Before creating any deal, check if the contact already has an active deal
- One deal per customer. If a contact already has a deal, update it — do not create a second one.

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
All contacts under a company are automatically associated to that company's deal. When a new contact is created under an existing company, add them to the existing deal.
```

- [ ] **Step 7: Write channel-to-stage mapping section**

```markdown
## Channel-to-Stage Mapping

| Source | Default Stage | Default Properties |
|---|---|---|
| Slack: `gtm-new-trial-info` | Trial | `lead_source` = slack-trial, `trial_end_date` = signup + 10 days, `activation_status` = not_started |
| Slack: `central-new-signups` | New | `lead_source` = slack-signup |
| Slack: `customer-feedback` | Varies — see below | `lead_source` = slack-feedback |
| Google Calendar (demo booked) | Demo | `lead_source` = google-calendar, `demo_status` = booked |
| Gmail (unknown sender) | New | `lead_source` = inbound-email |

### customer-feedback routing
- If contact has active deal in Active stage → update existing deal, check sentiment
- If contact has active deal in any other stage → update existing deal
- If no contact exists → create at New stage
- Positive feedback → no stage change
- Negative feedback → set `interest_level` = cold
- Cancellation confirmed → move to Churned
```

- [ ] **Step 8: Write sequence and note rules section**

```markdown
## Sequence Definitions

| Stage | Sequence | Cadence | Stop Conditions |
|---|---|---|---|
| New | Welcome / nurture | 3 emails over 7 days | Reply, meeting booked, stage change |
| Demo (missed) | Reschedule | 2 emails over 5 days | Reply, demo rebooked |
| Trial | Onboarding nudge | Day 3, Day 5, Day 8 | Reply, activation, stage change |
| Active | Monthly check-in | 1 email per 30 days | Reply |

*Sequence details (email copy) come from the `writing-voice` skill, not this skill.*

## Note Requirements

Every stage change must be logged with a HubSpot note containing:
- What changed (e.g., "Moved from New to Demo")
- Why (e.g., "Demo booked via Google Calendar for April 10")
- Written in Miguel's voice (agents read `writing-voice` skill for tone)

Notes written by agents must NOT look like bot logs. They should read as if Miguel wrote them.
```

- [ ] **Step 9: Verify skill file is complete**

Read through the entire skill file and confirm:
- All stage IDs are filled in (no `<ID>` placeholders remain)
- All 12 properties documented
- Transition rules cover every possible stage move
- Tags, guardrails, and mappings are complete

- [ ] **Step 10: Commit**

```bash
git add .claude/skills/hs-pipeline-manager.md
git commit -m "feat: add hs-pipeline-manager skill — unified pipeline with 7 stages and 12 properties"
```

---

## Task 4: Update architecture spec and CLAUDE.md

Update references to reflect the new pipeline structure.

**Files:**
- Modify: `CLAUDE.md`
- Modify: `docs/superpowers/specs/2026-04-04-central-ai-architecture-design.md`

- [ ] **Step 1: Update CLAUDE.md registry**

Change `hs-pipeline-manager` status from "To create" to "Active".

- [ ] **Step 2: Update architecture spec**

Fill in actual stage IDs in the Pipeline Redesign section (replacing any remaining placeholder text).

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md docs/superpowers/specs/2026-04-04-central-ai-architecture-design.md
git commit -m "docs: update CLAUDE.md and architecture spec with hs-pipeline-manager stage IDs"
```

---

## Task 5: Clean up retired pipeline files

Remove files that are now absorbed into hs-pipeline-manager.

**Files:**
- Delete: `.claude/skills/pipeline-guide.md`
- Delete: `central_ai_pipeline_rules.md`

- [ ] **Step 1: Delete pipeline-guide.md**

```bash
rm .claude/skills/pipeline-guide.md
```

- [ ] **Step 2: Delete central_ai_pipeline_rules.md**

```bash
rm central_ai_pipeline_rules.md
```

- [ ] **Step 3: Commit**

```bash
git add -A
git commit -m "chore: remove retired pipeline files — absorbed into hs-pipeline-manager skill"
```
