# Central AI — Skills, Agents & Architecture Design

> Date: 2026-04-04
> Author: Miguel Dungo + Claude
> Status: Approved — ready for implementation

---

## Overview

Rebuild Central AI's Claude Code setup from an orchestrator-first architecture. CLAUDE.md routes work to skills and agents. Skills own rules and knowledge. Agents execute workflows. No logic lives in CLAUDE.md.

---

## Architecture Summary

```
CLAUDE.md (orchestrator — routing only)
  |
  +-- Skills (knowledge + rules, loaded on demand)
  |     1. hs-pipeline-manager
  |     2. writing-voice
  |     3. channel-playbook
  |     4. central-ai-product-expert (exists)
  |     5. project-manager
  |     6. onboarding-playbook
  |
  +-- Agents (autonomous actors)
  |     1. leads-specialist (exists as lead-capture, to revise)
  |     2. customer-success-specialist (exists as cs-monitor, to revise)
  |
  +-- Trigger (exists, to revise)
  |     - leads-specialist → customer-success-specialist on schedule
  |
  +-- Docs (reference material)
        - marketing-resources-library (placeholder)
        - central_ai_pipeline_rules.md (retiring — absorbed into hs-pipeline-manager)
```

---

## Pipeline Redesign — Single Pipeline

The current two-pipeline setup (Sales + Accounts) is being consolidated into a single pipeline that tracks the full customer journey. One deal per customer, full history in one place.

### Unified Pipeline — 7 Stages

```
New → Demo → Trial → Onboarding → Active → Churned
Side: Lost
```

**Key behaviors:**
- Leads can skip stages. A signup that immediately starts a trial goes straight to Trial.
- One deal per customer. Multiple products tracked via multi-select property.
- All contacts under a company auto-associate to the company's deal.

**What stages replaced:**
- Qualified → replaced by `interest_level` property
- Demo Done / Missed Demo → replaced by `demo_status` property
- Trial Ended → replaced by `trial_end_date` property (signup + 10 days)
- Proposal Sent → replaced by HubSpot notes
- Closed Won → removed, deals go straight to Onboarding
- At Risk → replaced by `interest_level` = cold on Active deals
- New Signups → merged into New
- Growth Opportunity → replaced by deal tag (Active 90+ days)

### Deal Properties (12 custom)

Properties carry the context that stages can't. They enable filtering, tagging, and automation.

| # | Property | Type | Purpose | Set By |
|---|---|---|---|---|
| 1 | `signup_date` | Date | When they first signed up | Agent (auto) |
| 2 | `lead_source` | Dropdown | Where they came from (slack-trial, slack-signup, slack-feedback, google-calendar, inbound-email) | Agent (auto) |
| 3 | `interest_level` | Dropdown: hot / warm / cold | Engagement level. Replaces Qualified and At Risk stages. | Agent (auto) + Miguel (manual override) |
| 4 | `call_attempted` | Checkbox | Has a call been made? | Miguel (manual) or agent (if call task completed) |
| 5 | `demo_status` | Dropdown: none / booked / completed / missed | Demo state. Replaces Demo Done and Missed Demo stages. | Agent (auto from calendar) |
| 6 | `activation_status` | Dropdown: not_started / partial / activated | Product setup progress (calendar connected, first call received) | Miguel (manual for now) |
| 7 | `next_action_due` | Date | When the next touch should happen | Agent (auto based on cadence rules) |
| 8 | `trial_end_date` | Date | Trial expiry (signup + 10 days). Replaces Trial Ended stage. | Agent (auto) |
| 9 | `product` | Multi-select: AI Receptionist / AI Chat / AI Sales / EA | Which Central products they use | Agent (auto from signup data) + Miguel (manual) |
| 10 | `plan_tier` | Dropdown: Free / Starter / Growth / Scale / Enterprise | Current plan | Agent (auto) + Miguel (manual) |
| 11 | `original_plan` | Dropdown: Free / Starter / Growth / Scale / Enterprise | Starting plan — tracks upgrades over time | Agent (auto at first conversion) |
| 12 | `mrr` | Number | Current monthly recurring revenue | Agent (auto from plan tier) + Miguel (manual) |

### Properties by Stage

**New:**
- `lead_source`, `interest_level`, `call_attempted`, `next_action_due`
- Tags: "Call ASAP" (call_attempted = no + signup > 1 day), "Going Cold" (interest_level = cold + next_action_due overdue)

**Demo:**
- `demo_status`, `next_action_due`, `call_attempted`
- Tags: "Missed Demo" (demo_status = missed), "Awaiting Follow-up" (demo_status = completed + no follow-up), "Demo Today" (demo booked today)

**Trial:**
- `trial_end_date`, `activation_status`, `interest_level`, `next_action_due`, `product`, `plan_tier`
- Tags: "Trial Expiring" (trial_end_date within 3 days), "Needs Activation" (activation_status = not_started + 3 days in), "Going Cold" (interest_level = cold)

**Onboarding:**
- `activation_status`, `signup_date`, `next_action_due`, `product`, `plan_tier`, `original_plan`, `mrr`
- Tags: "Needs Activation" (activation_status = not_started + 3 days), "Ready for Active" (activation_status = activated + 14 days)

**Active:**
- `interest_level`, `signup_date`, `next_action_due`, `product`, `plan_tier`, `mrr`
- Tags: "At Risk" (interest_level = cold), "Growth Opportunity" (signup > 90 days + interest = hot/warm), "Stale" (no activity 5+ days)

**Churned / Lost:** Terminal stages — properties are historical record.

### Association Rule

All contacts that belong to a company are automatically associated to that company's deal. When a new contact is created under an existing company, the agent adds them to the existing deal.

### Pipeline vs. Properties vs. Tags

- **Stages** = where someone is in the journey (position)
- **Properties** = what's happening and why (context + data for filtering/automation)
- **Tags** = visual shortcut computed from properties (never manually set)

Stages move forward. Properties update at any time. Tags surface what needs attention.

---

## Skill Specs

### 1. hs-pipeline-manager

**Purpose:** Single source of truth for all HubSpot pipeline structure. Every agent that reads or writes to HubSpot reads this skill first.

**Scope:**

- **HubSpot config** — Owner ID (163091353), Hub ID (242826512), URL patterns for contacts/companies/deals
- **Pipeline definition** — single unified pipeline with all stages and IDs
- **Stage transition rules** — entry/exit conditions for every stage move, including non-linear entry (e.g., signup → Trial Started directly)
- **Deal properties** — 7 custom properties (signup_date, lead_source, interest_level, call_attempted, demo_status, activation_status, next_action_due)
- **Deal tags** — auto-generated tag rules computed from property values
- **Hygiene guardrails** — stale deal thresholds (5+ days no activity), duplicate prevention, blocked domains (trycentral.com, getwingapp.com, m32.ai), bot/test email filters
- **Channel-to-stage mapping** — Slack channel / lead source → stage
- **Sequence definitions** — which sequence maps to which stage, cadence, stop conditions (reply, meeting booked)
- **Note requirements** — what must be logged at each stage change, required fields
- **Workflow triggers** — what actions fire at each transition (create task, enroll sequence, draft email, log note)

**Does NOT cover:** email copy/style, channel selection logic, product knowledge, agent execution workflows.

**Absorbs:** `pipeline-guide.md`, `central_ai_pipeline_rules.md`, pipeline/stage IDs from `slack-lead-capture.md` and `customer-success-agent.md`

---

### 2. writing-voice

**Purpose:** Miguel's writing style and tone. Any agent or skill that produces text a human will read loads this first.

**Core intent:** Always offer help. Every email should make the person feel supported, not sold to.

**Scope:**

**Voice rules (derived from Miguel's actual sent emails):**
- Personal, direct, concise. 3-4 sentences max for outreach. Get to the point immediately.
- Sign as "Miguel" in email body. Signature block: "Best, Migs / Support @ Central AI / +1 (310) 894-8956"
- Always personalize — reference the person's business, industry, or situation. No generic batch emails.
- Always offer help, never push a sale. Frame CTAs as support ("Would you like a quick demo?" not "Sign up now").
- Binary CTAs always — "Would you like X, or would you prefer Y?" Forces an easy pick-one reply.
- One question per email. Never stack two asks.
- Use "lmk" naturally. Casual but professional.
- Scheduling link inline when appropriate: https://meet.trycentral.com/miguel/quick-call
- Phone number in signature: +1 (310) 894-8956

**Two writing modes:**
1. **Personalized first-touch** — research the business, reference it specifically, binary CTA.
   Example: "Kingdom Recovery KC does really important work, and I imagine you get a lot of calls from people reaching out for help. Would you like a quick demo to see how Central can handle those calls for you, or would you prefer to explore the trial first?"
2. **Empathetic response** — when customer is upset or cancelling, shift to formal and empathetic. Take ownership. Ask for feedback. Don't get defensive.
   Example: "I am very sorry to hear about your experience. I apologize for the lack of communication and the frustration this has caused. I would greatly appreciate your feedback on where we fell short."

**Subject line rules:**
- First-touch sales: "Hey {firstname}!" — casual, first name, exclamation mark
- Check-ins: conversational, lowercase ok — "Still worth a chat, {firstname}?", "Still here if you need anything"
- CS/support: 3-7 words, no exclamation points — "quick check-in", "everything okay on your end?"

**Anti-AI patterns (condensed from humanizer, tailored to Miguel's voice):**

| Never write | Write instead |
|---|---|
| "I hope this email finds you well" | Skip it entirely — open with the point |
| "I wanted to reach out..." | Just say what you're reaching out about |
| "I just wanted to check in..." | "Checking in on..." or open with the check-in directly |
| "Please don't hesitate to reach out" | "Reply here if you need anything" or "lmk" |
| "Looking forward to hearing from you" | Drop it or use a specific CTA |
| "seamless", "streamline", "leverage", "crucial" | Plain words |
| Em dashes (—) | Comma, period, or rewrite |
| Emojis | Never |
| Bullet lists in outreach emails | Write in sentences |
| Bold text mid-sentence | Don't format outreach emails |
| "The Central AI Team" | Sign as Miguel/Migs |
| Generic check-in with no personalization | Always reference their business or situation |

**Self-audit step:** After drafting, ask "what makes this obviously AI-generated?" Fix remaining tells. Then ask "does this sound like it's offering help?" If not, rewrite.

**HubSpot note style:** Plain, factual, no marketing language, no AI tells.

**General guidelines:**
- Plain text only — no HTML tags
- From: miguel@trycentral.com
- Always draft, never send directly
- No unsubscribe links or legal footers — these are personal 1:1 emails
- Max ~100 words per email body

**Email templates by stage:** Templates define structure and CTA per stage (trial started, new lead, qualified, demo follow-up, feedback response, at-risk, churn recovery). Each template follows the voice rules above. Templates are starting points — every email must be personalized to the recipient.

**Absorbs:** `email-templates.md`, `email-cs.md`, `humanizer.md`, `draft-email.md`

---

### 3. channel-playbook

**Purpose:** Defines when to use which communication channel and the cadence for each pipeline stage. The "when to do what" skill.

**Core rule:** Call first. Every new lead gets a call attempt before any email. Call is always the highest-priority action.

**Cadence rule:** After a missed call, send email same day. Don't wait.

**Channel routing by stage:**

| Stage | Action 1 (immediate) | Action 2 (if no answer/reply) | Action 3 (if still no response) |
|---|---|---|---|
| New Lead | Call | Email same day (personalized welcome) | Enroll in sequence |
| Qualified | Call | Email same day (demo/trial offer) | Enroll in sequence |
| Demo Scheduled | Confirmation email + full prospect brief | Pre-demo prep notes day before | -- |
| Demo Done | Follow-up email within 24h | Call if no reply in 48h | Proposal within 3 days |
| Missed Demo | Call to reschedule | Email same day with new booking link | -- |
| Trial Started | Welcome email + schedule setup call | Day 3: activation check call | Day 5: check-in if not activated |
| Trial Ended | Call (highest-leverage moment) | Email same day with conversion pitch | -- |
| Proposal Sent | Wait 3 days | Follow-up call | Follow-up email same day |
| Closed Won | Onboarding playbook takes over | -- | -- |
| Active | Monthly check-in email | -- | -- |
| At Risk | Call first | Empathetic email same day | -- |
| Churned | Call for feedback | Email asking what happened | -- |

**Demo prep flow:**
- Trigger: demo appears on calendar (booked via Google Calendar or manually)
- Agent produces a full prospect brief:
  - Company website scan (what they do, industry, who they serve)
  - HubSpot data (lead source, deal history, notes, previous interactions)
  - Pain points likely based on industry vertical
  - Suggested talking points for the demo
- Brief delivered day before the demo (or same day if booked same day)

**SMS:** Situational only. Not in any automated flow. Miguel decides when to text.

**Sequence stop conditions:** Stop on reply, meeting booked, or stage change.

**Does NOT cover:** email copy (that's writing-voice), pipeline structure (that's hs-pipeline-manager).

---

### 4. central-ai-product-expert (exists)

**Purpose:** Product knowledge, pricing, features, competitive positioning.

**Status:** Active at `~/.claude/skills/central-ai-product-expert/SKILL.md`. May need update to reflect current pricing and features.

**Action:** Review and update if needed after core skills are built.

---

### 5. project-manager

**Purpose:** Detailed reference for briefing format, reporting structure, and task management rules. CLAUDE.md defines the PM role and default behavior (orient → brief → route → track). This skill provides the detailed specs that the `daily-pm` agent reads at runtime.

**Relationship to CLAUDE.md:** CLAUDE.md says "you are the PM, here's how to behave." This skill says "here's the detailed format and rules." No duplication — CLAUDE.md has the summary, this skill has the specifics.

**Scope:**

- **Morning briefing format (detailed):**
  - Yesterday section: total emails sent (from Gmail), calls made (from HubSpot engagements), deals created, deals moved between stages, sequences enrolled
  - Pipeline section: new leads awaiting action, trials expiring within 3 days, stale deals (5+ days), at-risk accounts, demos scheduled today
  - Today's task list: prioritized per the rules in CLAUDE.md, with specific deal names and contact info for each task
- **Task priority rules:** Defined in CLAUDE.md (expiring trials → demos → new leads → stale → at-risk → onboarding → growth). This skill adds detail:
  - Expiring trials: list by days remaining, closest first
  - Demos: list by time today, include prospect brief link
  - New leads: list by signup date, oldest first (longest wait = highest risk of losing them)
  - Stale deals: list by days since last activity, most stale first
- **EOD recap format:**
  - What got done (tasks completed, emails sent, calls made, deals moved)
  - What rolled over (incomplete tasks, why)
  - Any blockers or items needing Miguel's attention tomorrow
- **Reporting cadence:** Morning briefing + EOD recap on weekdays
- **Task checklist management:** Maintained throughout the session. Mark complete as tasks are done. Surface incomplete tasks if Miguel is about to end the session.

**Does NOT cover:** pipeline rules (hs-pipeline-manager), email drafting (writing-voice), product knowledge (product-expert).

---

### 6. onboarding-playbook

**Purpose:** Get new users set up and using the product. Triggers at both Trial Started and Closed Won — both get the same onboarding attention.

**Scope:**

**Activation definition:** Calendar connected AND first real call received through Central AI.

**Activation tracking:** Uses the `activation_status` deal property (not_started / partial / activated) defined in hs-pipeline-manager. Miguel updates manually for now; automate later when Central AI dashboard data can feed HubSpot.

**Trial track (10-day window):**
- Day 0: Welcome email + schedule setup call
- Day 1-3: Setup call (connect calendar, configure voice, set up call routing)
- Day 3: Activation check (calendar connected? first call received?)
- Day 5: Check-in if not activated
- Day 8: Trial expiry warning + conversion nudge
- Day 10: Trial ends — channel-playbook takes over (call → email for conversion)

**Paid track (14-day onboarding):**
- Day 0: Welcome email + schedule setup call
- Day 1-3: Setup call (connect calendar, configure voice, set up call routing)
- Day 3: Activation check (calendar connected? first call received?)
- Day 5: Check-in if not activated
- Day 7: Mid-onboarding check-in
- Day 14: Move to Active if activated

**Risk signals during onboarding:**
- No calendar connected by day 3
- No calls received by day 5 (trial) or day 7 (paid)
- No login activity
- No response to setup call scheduling

**Does NOT cover:** email copy (writing-voice), pipeline stage IDs (hs-pipeline-manager), channel selection (channel-playbook).

---

## Agent Specs

### 1. leads-specialist (revise existing lead-capture)

**Purpose:** Autonomous lead capture from all sources into HubSpot. Capture only — no outreach.

**Lead sources:**

| Source | How | What to look for |
|---|---|---|
| Slack: `gtm-new-trial-info` | Read channel messages | Trial signup notifications |
| Slack: `central-new-signups` | Read channel messages | New user signups |
| Slack: `customer-feedback` | Read channel messages | Feedback from trial/paying users |
| Google Calendar | Check for new bookings | Demo events |
| Gmail inbox | Scan for unknown senders | Inbound product inquiries |

**What it does:**
1. Extract lead info (name, email, phone, company, product/plan)
2. Filter out bots, test accounts, blocked domains
3. Deduplicate against HubSpot
4. Create/update HubSpot records (contact, company, deal) with correct stage and properties
5. Research prospect's website for context (stored on deal)
6. Log a note on the deal in Miguel's voice (reading `writing-voice`)
7. Report summary

**What it does NOT do:** Email drafting, sequence enrollment, call tasks, any outreach.

**Consumes:** `hs-pipeline-manager`, `writing-voice`

---

### 2. customer-success-specialist (revise existing cs-monitor)

**Primary mission:** Keep HubSpot property fields accurate and up-to-date across deals, contacts, and companies. All actions serve this goal.

**What it does:**

| Area | Actions |
|---|---|
| Property management | Ensure all 7 deal properties are set and current. Update as events occur. Keep contact and company records complete. |
| Outreach | Enroll in HubSpot sequences directly, draft emails in HubSpot, create call tasks |
| Pipeline hygiene | Detect stale deals, flag duplicates, enforce stage transition rules |
| Stage moves | Autonomous — including Churned when confirmed via customer-feedback channel |
| Onboarding | Monitor activation milestones per onboarding-playbook |
| At-risk detection | Flag danger signals, draft re-engagement |
| Demo prep | Full prospect brief (website, HubSpot data, pain points, talking points) |
| Gmail monitoring | Scan inbound from known contacts, classify intent, draft replies |
| Activity logging | Notes on every action, tasks for items needing Miguel's review |

**Autonomy rules:**

| Action | Behavior |
|---|---|
| Stage transitions (including Churned on confirmed cancellation) | Autonomous |
| Property updates | Autonomous |
| HubSpot sequence enrollment | Autonomous |
| HubSpot email drafts | Autonomous |
| HubSpot notes/tasks | Autonomous |
| Gmail email drafts | Draft only, never send |
| Ambiguous churn signals | Human required — create task |
| Discounts | Human required — flag to Miguel |

**Phases:** Pipeline scan → Gmail monitoring → Summary

**Consumes:** `hs-pipeline-manager`, `writing-voice`, `channel-playbook`, `onboarding-playbook`

---

### Daily PM (not an agent — Claude directly)

Claude acts as Miguel's secretary in every live session, guided by CLAUDE.md + `project-manager` skill. No separate agent needed.

- Runs briefing at session start
- Tracks tasks throughout the day
- Provides EOD recap on request
- Queries HubSpot, Gmail, and Calendar read-only for reporting

---

## Trigger Spec (revise existing)

**Current:** `trig_01HSBycEwKUXPXZW6ffaAQPa` — disabled, runs lead-capture + cs-monitor hourly on weekdays

**Revised settings:**

| Setting | Value |
|---|---|
| ID | `trig_01HSBycEwKUXPXZW6ffaAQPa` |
| Name | Leads Specialist + Customer Success Specialist |
| Status | Enabled |
| Schedule | `0 16-23 * * 1-5` (hourly 9am-4pm PT, keep current) |
| Model | Sonnet 4.6 (execution model) |
| Skill source | `.claude/skills/` via `github.com/migdungo/central-ai-skills` |
| Persist session | false |

**MCP connections (add Google Calendar):**

| MCP | Connector UUID | Status |
|---|---|---|
| Slack | `16b87fde-6176-4c27-ac58-80072de26ca2` | Connected |
| HubSpot | `124a1e36-675d-4805-ba1a-077a7c5e4ea0` | Connected |
| Gmail | `8e627a0a-9734-4777-891b-9e210fb71c41` | Connected |
| Google Calendar | TBD — need connector UUID | To add |

**Tool prefix mapping (keep in prompt):**
The trigger environment uses different MCP tool prefixes than the local Claude Code environment. The prompt includes a mapping block:
- `mcp__claude_ai_HubSpot__*` → `mcp__HubSpot__*`
- `mcp__claude_ai_Gmail__*` → `mcp__Gmail__*`
- `mcp__claude_ai_Slack__*` → `mcp__Slack__*`
- `mcp__claude_ai_Google_Calendar__*` → `mcp__Google_Calendar__*`

**Allowed tools (add Calendar):**
```
Read, WebFetch,
mcp__HubSpot__search_crm_objects, mcp__HubSpot__get_crm_objects, mcp__HubSpot__manage_crm_objects,
mcp__Gmail__gmail_create_draft, mcp__Gmail__gmail_search_messages,
mcp__Slack__slack_read_channel, mcp__Slack__slack_send_message, mcp__Slack__slack_search_users,
mcp__Google_Calendar__gcal_list_events, mcp__Google_Calendar__gcal_get_event
```

**Prompt (revised):**
```
You are the Central AI operations agent for trycentral.com.

Tool name mapping for this environment:
- HubSpot: mcp__HubSpot__
- Gmail: mcp__Gmail__
- Slack: mcp__Slack__
- Google Calendar: mcp__Google_Calendar__

Execute in order:
1. Read .claude/skills/leads-specialist.md — run the full lead capture workflow with lookback since midnight Pacific Time today
2. Read .claude/skills/customer-success-specialist.md — run the full CS scan
3. Output the combined summary from both agents
```

**Execution order:** leads-specialist (capture) → customer-success-specialist (outreach + hygiene). Always sequential — Agent 2 depends on Agent 1's records being in HubSpot first.

---

## Files to Retire

| File | Absorbed Into |
|---|---|
| `central-ai-project-instructions.md` | Already deleted — covered by product-expert skill |
| `claude_cowork_instructions.md` | Already deleted — covered by CLAUDE.md + skills |
| `central_ai_pipeline_rules.md` | hs-pipeline-manager skill (delete after skill is built) |
| `.claude/skills/pipeline-guide.md` | hs-pipeline-manager skill |
| `.claude/skills/email-templates.md` | writing-voice skill |
| `.claude/skills/email-cs.md` | writing-voice skill |
| `.claude/skills/humanizer.md` | writing-voice skill |
| `.claude/skills/draft-email.md` | writing-voice skill |

---

## Build Order

1. `hs-pipeline-manager` — foundation, everything reads from it
2. `writing-voice` — second foundation, all text output reads from it
3. `channel-playbook` — defines action routing
4. `onboarding-playbook` — post-close flow
5. `project-manager` — briefing format and reporting rules
6. Revise `leads-specialist` agent — reference new skills, capture only
7. Revise `customer-success-specialist` agent — reference new skills, owns all outreach
8. Revise trigger — update paths, add Calendar MCP, re-enable
9. Update CLAUDE.md registry — point to all new skills/agents
10. Update `central-ai-product-expert` — review and refresh if needed
11. Delete retired files

---

## HubSpot Prerequisites

**Pipeline restructuring (modify existing Sales pipeline `default`):**
- Rename/consolidate stages: New Lead → New, Demo Scheduled → Demo, Trial Started → Trial
- Add new stages: Onboarding, Active, Churned
- Rename Closed Lost → Lost
- Remove stages: New Signups, Qualified, Demo Done, Missed Demo, Trial Ended, Proposal Sent, Closed Won
- Migrate existing Accounts pipeline deals into unified pipeline at equivalent stage (Onboarding/Active/Churned)
- Archive old Accounts pipeline (`2123072201`)

**Custom deal properties to create (12 total):**
1. `signup_date` (date)
2. `lead_source` (dropdown: slack-trial, slack-signup, slack-feedback, google-calendar, inbound-email)
3. `interest_level` (dropdown: hot, warm, cold)
4. `call_attempted` (checkbox)
5. `demo_status` (dropdown: none, booked, completed, missed)
6. `activation_status` (dropdown: not_started, partial, activated)
7. `next_action_due` (date)
8. `trial_end_date` (date)
9. `product` (multi-select: AI Receptionist, AI Chat, AI Sales, EA)
10. `plan_tier` (dropdown: Free, Starter, Growth, Scale, Enterprise)
11. `original_plan` (dropdown: Free, Starter, Growth, Scale, Enterprise)
12. `mrr` (number)

---

## Design Principles

- **Zero logic in CLAUDE.md** — routing pointers only
- **Skills own rules, agents execute them** — no duplicated logic
- **Single source of truth** — pipeline IDs in one place, writing style in one place
- **Anti-drift** — skills own playbooks; if no skill exists, it's not a defined workflow
- **Model rules** — Opus 4.6 for planning/conversations, Sonnet 4.6 for execution
- **Time** — Business in Pacific Time, Miguel in Philippine Time
- **Sales-first** — new business and calls always take priority over support tasks
