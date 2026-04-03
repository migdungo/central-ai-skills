---
name: onboarding-playbook
version: 1.0.0
description: |
  Post-signup onboarding flow for both trial users and paying customers.
  Defines activation milestones, check-in timelines, and risk signals.
  Read this skill when a deal enters Trial or Onboarding stage.
allowed-tools:
  - Read
---

# Onboarding Playbook

Get new users set up and using the product. Triggers at both Trial and Onboarding stages. Both get the same attention.

## Activation Definition

A customer is **activated** when:
1. Calendar connected to Central AI
2. First real call received through Central AI

Track via `activation_status` property: Not Started / Partial / Activated

- **Not Started** — neither milestone reached
- **Partial** — one milestone reached (e.g., calendar connected but no calls yet)
- **Activated** — both milestones complete

---

## Trial Track (10-day window)

| Day | Action | Check |
|---|---|---|
| 0 | Welcome email + schedule setup call | — |
| 1-3 | Setup call (connect calendar, configure voice, set up call routing) | — |
| 3 | Activation check | Calendar connected? First call received? |
| 5 | Check-in if not activated | activation_status still Not Started? |
| 8 | Trial expiry warning + conversion nudge | trial_end_date within 2 days |
| 10 | Trial ends | Channel-playbook takes over (call then email for conversion) |

**Key moments:**
- Day 3 is the first risk checkpoint. If activation_status = Not Started, intervene.
- Day 8 is the conversion nudge window. Highest urgency.
- Day 10 is the deadline. Call-first per channel-playbook.

---

## Paid Track (14-day onboarding)

| Day | Action | Check |
|---|---|---|
| 0 | Welcome email + schedule setup call | — |
| 1-3 | Setup call (connect calendar, configure voice, set up call routing) | — |
| 3 | Activation check | Calendar connected? First call received? |
| 5 | Check-in if not activated | activation_status still Not Started? |
| 7 | Mid-onboarding check-in | How are things going? Any issues? |
| 14 | Move to Active if activated | activation_status = Activated AND 14+ days |

**Key moments:**
- Day 3 is the first risk checkpoint. Same as trial.
- Day 7 is a relationship-building touch. Not a sales push.
- Day 14 is graduation. Only move to Active if activated.

---

## Risk Signals During Onboarding

| Signal | When to flag | Action |
|---|---|---|
| No calendar connected | Day 3+ | Call to assist with setup |
| No calls received | Day 5 (trial) / Day 7 (paid) | Check if product is configured correctly |
| No login activity | Day 3+ | Email asking if they need help |
| No response to setup call scheduling | Day 2+ | Try email, then SMS (Miguel's discretion) |
| activation_status still Not Started at day 7 | Day 7 | Escalate — this customer is at high risk of churn |

---

## Setup Call Agenda

When Miguel gets on a setup call, cover these:

1. Connect Google Calendar (or supported scheduler)
2. Configure AI voice — name, greeting, tone
3. Set up call routing — who gets escalated calls, availability windows
4. Add knowledge sources — website URL, FAQs, business info
5. Test — make a test call to the Central number
6. Confirm activation — calendar connected, first call received

---

## Stage Transitions

### Trial → Onboarding
- Trigger: Customer converts to paid plan
- Set: `plan_tier`, `original_plan`, `mrr`, `product_type`
- Note: "Converted from trial to {plan_tier}. Setting up onboarding."

### Onboarding → Active
- Trigger: `activation_status` = Activated AND 14+ days in Onboarding
- Set: `next_action_date` = 30 days from now (first monthly check-in)
- Note: "Onboarding complete. Activated on day {N}. Moving to Active."

### Trial → Lost
- Trigger: Trial expired without conversion (`trial_end_date` passed, no payment)
- Note: "Trial expired. {Reason if known}."

### Onboarding → Active (early)
- If customer is activated before day 14, wait until day 14 to move. The 14-day window is for stabilization even if setup is complete.
