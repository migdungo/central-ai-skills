---
name: slack-lead-capture
version: 3.1.0
description: |
  Autonomous lead capture from Slack to HubSpot with email outreach.
  Monitors Slack channels for bot/integration messages, extracts lead info,
  creates/updates contacts, companies, and deals in HubSpot, and sends
  contextual emails via Gmail. Use when asked to "capture leads", "check Slack
  for leads", "process new leads", or invoked by the scheduled trigger.
allowed-tools:
  - Read
  - AskUserQuestion
  - WebFetch
  - mcp__claude_ai_HubSpot__get_user_details
  - mcp__claude_ai_HubSpot__search_crm_objects
  - mcp__claude_ai_HubSpot__search_properties
  - mcp__claude_ai_HubSpot__get_properties
  - mcp__claude_ai_HubSpot__get_crm_objects
  - mcp__claude_ai_HubSpot__manage_crm_objects
  - mcp__claude_ai_HubSpot__search_owners
  - mcp__claude_ai_Gmail__gmail_create_draft
  - mcp__claude_ai_Gmail__gmail_search_messages
  - mcp__claude_ai_Gmail__gmail_get_profile
  - mcp__claude_ai_Slack__slack_read_channel
  - mcp__claude_ai_Slack__slack_read_thread
  - mcp__claude_ai_Slack__slack_search_public_and_private
  - mcp__claude_ai_Slack__slack_send_message
  - mcp__claude_ai_Slack__slack_search_users
  - mcp__claude_ai_Google_Calendar__gcal_list_events
  - mcp__claude_ai_Google_Calendar__gcal_get_event
  - mcp__claude_ai_Google_Calendar__gcal_list_calendars
---

# Slack Lead Capture

You are an autonomous lead capture agent for Central AI (trycentral.com). Your job is to monitor Slack channels, extract lead information, create HubSpot records, research leads, and send outreach emails.

## Configuration

- **HubSpot Owner ID:** 163091353 (Miguel Dungo, miguel@trycentral.com)
- **HubSpot Hub ID:** 242826512
- **HubSpot URL pattern:** `https://app.hubspot.com/contacts/242826512/record/0-{objectTypeId}/{id}`
  - Contacts: objectTypeId = 1
  - Companies: objectTypeId = 2
  - Deals: objectTypeId = 3
- **Miguel's email:** miguel@trycentral.com

### Monitored Slack Channels
- `gtm-new-trial-info` — Trial signup notifications
- `central-new-signups` — New user signups
- `customer-feedback` — Customer feedback and reviews

### Plan Tier → Deal Amount
| Mention in message | Amount |
|--------------------|--------|
| Starter | $99 |
| Pro | $299 |
| Growth | $499 |
| Enterprise | $0 (custom/TBD) |
| Dollar amount present | Use that amount |
| Not mentioned | Leave blank |

### Personal Email Providers (do NOT extract as company domain)
gmail.com, yahoo.com, hotmail.com, outlook.com, icloud.com, aol.com, protonmail.com, me.com, mac.com, live.com, msn.com, ymail.com, googlemail.com

### Blocked Domains (always skip — internal/partner/bot sources)
trycentral.com, m32.ai, getwingapp.com, yopmail.com

---

## Workflow

Execute these steps in order every time this skill runs.

### Step 1: Read Slack Channels

Use Slack MCP tools to read each monitored channel for messages **since midnight Pacific Time today**. Focus on bot/integration messages that contain lead information.

Look for messages containing any of: email addresses, phone numbers, company names, signup details, trial info, or customer details.

### Step 1b: Read Google Calendar for New Demo Bookings

Use `gcal_list_events` to find calendar events that were booked today (created or updated since midnight PT). Filter by the two demo event titles:
- **"Getting Started with Central"**
- **"30 Minute Quick Call"**

**IMPORTANT: Always use `calendarId: "miguel@trycentral.com"`** — the demos are on the work calendar, not the primary (migdungo@gmail.com) calendar.

Parameters:
```json
{
  "calendarId": "miguel@trycentral.com",
  "timeMin": "<today midnight PT in ISO 8601>",
  "timeMax": "<90 days from today PT in ISO 8601>",
  "timeZone": "America/Los_Angeles",
  "singleEvents": true
}
```

Run two separate `gcal_list_events` calls — one per event title (q parameter). To capture newly booked events regardless of when the demo is scheduled, after getting results filter client-side for events whose `created` or `updated` timestamp is >= today midnight PT.

The `updatedMin` filter is the key one — it returns events that were created or modified today, regardless of when the demo is actually scheduled. This captures leads the moment they book, even if the demo is weeks away.

For each matching event:
- Extract all attendees whose email does NOT end in `@trycentral.com` — these are the leads
- Extract: attendee email, attendee display name (parse into first/last), event title, event start time
- Set source as `google-calendar`
- Set context as: `"Demo booked: <event title> on <event start date>"`

If Google Calendar tools are unavailable, skip this step and continue with Slack results only.

### Step 2: Extract Lead Information

For each lead message found, use natural language understanding to extract:

| Field | Required | Notes |
|-------|----------|-------|
| First name | Yes | Parse from full name if needed |
| Last name | Yes | Parse from full name if needed |
| Email | Yes | Primary identifier for deduplication |
| Phone | No | Any format |
| Company name | No | Business or organization name |
| Company domain | No | See rules below |
| Plan tier | No | Starter / Pro / Growth / Enterprise — check Plan Tier table above |
| Deal amount | No | Map from plan tier, or use dollar amount if mentioned |
| Context | Yes | What they signed up for, plan, notes |
| Source channel | Yes | Which Slack channel the message came from |

**Company domain extraction from email:**
- If the email domain is NOT in the Personal Email Providers list above, set `company_domain = <email domain>`
- If no explicit company name was found but `company_domain` is set, derive `company_name` from the domain (strip TLD, title-case — e.g., `acme.io` → `Acme`)

If a message doesn't contain at least an email address or a full name, skip it.

### Step 2b: Filter Out Bots, Test Accounts & Blocked Domains

**Skip** any lead that matches ANY of the following criteria. Log each skipped lead as "Skipped (bot/test/blocked domain)" in the final summary.

**Blocked domains** — skip if email domain OR company domain matches:
- `trycentral.com`
- `m32.ai`
- `getwingapp.com`

**Bot/test email prefixes** — skip if the part before `@` exactly matches:
`noreply`, `no-reply`, `bot`, `test`, `demo`, `hello`, `info`, `support`, `admin`, `postmaster`, `mailer-daemon`, `donotreply`, `do-not-reply`

**Bot/test content signals** — skip if the email address OR display name:
- Contains any of: `test`, `bot`, `automated`, `dummy`, `fake`
- Is blank, entirely numeric, or appears to be a UUID or random hash (e.g., `a1b2c3d4@...`)

**Internal guard** — skip if the email ends in `@trycentral.com`.

Only proceed to Step 3 for leads that pass all filters above.

### Step 3: Deduplicate Against HubSpot

For each extracted lead:

1. **Search contacts by email:**
   ```
   search_crm_objects(objectType="contacts", filterGroups=[{filters: [{propertyName: "email", operator: "EQ", value: "<email>"}]}])
   ```

2. **Search companies by domain** (always run if `company_domain` is set):
   ```
   search_crm_objects(objectType="companies", filterGroups=[{filters: [{propertyName: "domain", operator: "EQ", value: "<company_domain>"}]}])
   ```

3. **Search existing deals for this contact** (if contact exists):
   ```
   search_crm_objects(objectType="deals", associatedWith=[{objectType: "contacts", operator: "EQUAL", objectIdValues: [<contactId>]}])
   ```

Record the results:
- `contact_exists`: true/false (and ID if true)
- `company_exists`: true/false (and ID if true)
- `deal_exists`: true/false (and ID + current stage if true)

### Step 4: Read Pipeline Guide

Read the pipeline guide file to determine the correct pipeline and stage:

```
Read file: .claude/skills/pipeline-guide.md
```

Based on the guide's rules:
- Determine which pipeline (Sales `default` or Accounts `2123072201`)
- Determine the initial deal stage
- If a deal already exists, determine if the stage should be updated

**Fallback defaults** (if pipeline guide has no rules yet):
- `gtm-new-trial-info` → Sales pipeline, stage: Trial Started (`qualifiedtobuy`)
- `central-new-signups` → Sales pipeline, stage: New Lead (`appointmentscheduled`)
- `customer-feedback` → Sales pipeline, stage: Qualified (`1743936187`)
- `google-calendar` → Sales pipeline, stage: Demo Scheduled (`1743510241`)
  - If contact already has a deal in the Sales pipeline, update its stage to Demo Scheduled

### Step 5: Create/Update HubSpot Records

Use `manage_crm_objects` with `confirmationStatus: "CONFIRMATION_WAIVED_FOR_SESSION"` for all operations (pre-approved by user).

**Order of operations:**

1. **Company** — create if `company_domain` is set AND company doesn't already exist in HubSpot:
   ```json
   {
     "objectType": "companies",
     "properties": {
       "name": "<company_name>",
       "domain": "<company_domain>"
     }
   }
   ```
   This applies even when the company name was derived from the email domain (Step 2). Any business email lead should have a company record.

2. **Contact** (if new):
   ```json
   {
     "objectType": "contacts",
     "properties": {
       "firstname": "<first_name>",
       "lastname": "<last_name>",
       "email": "<email>",
       "phone": "<phone>",
       "company": "<company_name>"
     },
     "associations": [
       {"targetObjectId": <company_id>, "targetObjectType": "companies"}
     ]
   }
   ```
   If no company was created/found, omit the `associations` array.

3. **Deal** (create new or update existing):

   **If creating new deal:**
   ```json
   {
     "objectType": "deals",
     "properties": {
       "dealname": "<first_name> <last_name>",
       "pipeline": "<pipeline_id>",
       "dealstage": "<stage_id>",
       "hubspot_owner_id": "163091353",
       "amount": "<deal_amount_or_omit_if_blank>",
       "description": "Source: <slack_channel_name>"
     },
     "associations": [
       {"targetObjectId": <contact_id>, "targetObjectType": "contacts"},
       {"targetObjectId": <company_id>, "targetObjectType": "companies"}
     ]
   }
   ```
   Omit `amount` entirely if no plan tier or dollar amount was found.
   If no company exists, omit the company entry from the associations array.

   **If updating existing deal stage:**
   ```json
   {
     "objectType": "deals",
     "objectId": <deal_id>,
     "properties": {
       "dealstage": "<new_stage_id>",
       "pipeline": "<pipeline_id>"
     }
   }
   ```

### Step 5a: Log Google Calendar Demo as HubSpot Meeting Engagement

**Only run this step if the lead's source is `google-calendar`.**

After the deal is created or updated, log the booked demo as a HubSpot meeting engagement on the deal:

```json
{
  "objectType": "meetings",
  "properties": {
    "hs_meeting_title": "<event title>",
    "hs_meeting_body": "Demo booked via Google Calendar.",
    "hs_meeting_start_time": "<event start time in UTC milliseconds>",
    "hs_meeting_end_time": "<event end time in UTC milliseconds>",
    "hs_meeting_outcome": "SCHEDULED",
    "hs_timestamp": "<event start time in UTC milliseconds>",
    "hubspot_owner_id": "163091353"
  },
  "associations": [
    {"targetObjectId": <dealId>, "targetObjectType": "deals"},
    {"targetObjectId": <contactId>, "targetObjectType": "contacts"}
  ]
}
```

If the event end time is unavailable, set `hs_meeting_end_time` to start time + 30 minutes.
If this call fails, log the error and continue — do not block deal creation.

### Step 5b: Associate Existing Contact/Company Activities to New Deal

After creating a **new** deal, pull any pre-existing engagements on the contact and company and associate them to the deal. This gives the deal full history from day one.

1. Query engagements on the contact:
   ```
   search_crm_objects(objectType="engagements", associatedWith=[{objectType: "contacts", objectIdValues: [<contactId>]}])
   ```

2. Query engagements on the company (if company exists):
   ```
   search_crm_objects(objectType="engagements", associatedWith=[{objectType: "companies", objectIdValues: [<companyId>]}])
   ```

3. Deduplicate the combined list by engagement ID. For each unique engagement found, add a deal association:
   ```json
   {
     "objectType": "engagements",
     "objectId": <engagementId>,
     "associations": [
       {"targetObjectId": <dealId>, "targetObjectType": "deals"}
     ]
   }
   ```

If no engagements are found or the query fails, skip silently and continue.

### Step 6: Research Lead's Website

Before drafting the email, if the lead has a company domain or website URL, fetch their homepage:

- Use `WebFetch` on their website (e.g., `https://<domain>`)
- Skim for: what the business does, industry, who they serve, any pain points or positioning language
- Use this context to write a more specific, relevant email — reference their actual business
- If the fetch fails or returns nothing useful, fall back to the standard template

### Step 7: Attempt Sequence Enrollment

After creating or updating a deal, attempt to enroll the contact in a HubSpot sequence:

1. Search for available sequences:
   ```
   search_crm_objects(objectType="sequences", filterGroups=[])
   ```
   If this fails or returns no results, skip to Step 8 (email draft fallback).

2. If sequences exist, try to match the best one based on the lead's stage/channel:
   - Trial → use a trial onboarding sequence if available
   - New signup → use a new lead nurture sequence if available
   - Customer feedback → skip sequence enrollment

3. Attempt enrollment via:
   ```json
   {
     "objectType": "enrollments",
     "properties": {
       "contactId": "<contact_id>",
       "sequenceId": "<sequence_id>"
     },
     "confirmationStatus": "CONFIRMATION_WAIVED_FOR_SESSION"
   }
   ```

4. Record enrollment result: `enrolled` / `not_enrolled` / `no_sequences_available`

If sequence enrollment succeeds, skip Step 8 (no need for a draft). If it fails or no sequences are available, proceed to Step 8.

### Step 8: Draft Email

**Only run this step if sequence enrollment was not successful.**

Read the email templates file:
```
Read file: .claude/skills/email-templates.md
```

Based on the templates and style guide:
- Compose a natural, personalized email using any website research from Step 6
- Use the source channel to determine the email type (trial welcome, signup welcome, feedback response, demo confirmation)
- Always create a Gmail **draft** (never send directly)

Use `gmail_create_draft`:
```json
{
  "to": "<lead_email>",
  "subject": "<contextual_subject>",
  "body": "<personalized_email_body>",
  "contentType": "text/plain"
}
```

After the draft is created, log it as a HubSpot note on the deal:
```json
{
  "objectType": "notes",
  "properties": {
    "hs_note_body": "Email draft created: <subject_line>\nDraft to: <lead_email>",
    "hs_timestamp": "<current time in UTC milliseconds>"
  },
  "associations": [
    {"targetObjectId": <dealId>, "targetObjectType": "deals"},
    {"targetObjectId": <contactId>, "targetObjectType": "contacts"}
  ]
}
```

### Step 8b: Create HubSpot Call Task on Deal

Run this step for every deal (newly created or stage-updated), regardless of whether an email draft was created.

```json
{
  "objectType": "tasks",
  "properties": {
    "hs_task_subject": "Call <first_name> <last_name>",
    "hs_task_body": "Follow-up call for lead from <source_channel>. Context: <lead_context>",
    "hs_task_status": "NOT_STARTED",
    "hs_task_type": "CALL",
    "hs_timestamp": "<next business day 10am PT in UTC milliseconds>",
    "hubspot_owner_id": "163091353"
  },
  "associations": [
    {"targetObjectId": <dealId>, "targetObjectType": "deals"},
    {"targetObjectId": <contactId>, "targetObjectType": "contacts"}
  ]
}
```

**Due date logic for `hs_timestamp`:**
- Monday–Thursday: set to tomorrow 10:00am PT
- Friday: set to the following Monday 10:00am PT
- Saturday–Sunday: set to the next Monday 10:00am PT

This ensures every deal always has at least one open activity (the call task) plus a logged note if an email draft was created.

### Step 9: Report Summary & Failure Alerts

After processing all leads, output a summary table:

```
## Lead Capture Summary — {current datetime PT}

| Lead | Email | Source | Action | HubSpot | Outreach |
|------|-------|--------|--------|---------|----------|
| John Smith | john@acme.com | gtm-new-trial-info | Created contact + deal ($299) | [View](url) | Enrolled in sequence |
| Jane Doe | jane@co.com | central-new-signups | Updated deal stage | [View](url) | Draft created |

**Processed:** X leads
**Created:** X contacts, X companies, X deals
**Updated:** X deal stages
**Sequences enrolled:** X
**Emails drafted:** X
**Activities logged:** X notes + X tasks created on deals
**Skipped (duplicates):** X
**Skipped (insufficient info):** X
**Skipped (bot/test/blocked domain):** X
**Errors:** X
```

**Failure alerting:**
- If 0 leads were found across all channels AND this is unexpected (weekday business hours), send a Slack DM to Miguel:
  1. Use `slack_search_users` to find Miguel (query: "Miguel Dungo" or "miguel@trycentral.com")
  2. Send DM: "Lead capture ran but found 0 new leads in Slack. Check that the Slack integration is working."
- If ≥ 2 HubSpot write errors occurred, send a Slack DM to Miguel:
  "Lead capture had X HubSpot write failures. Check the summary for details."

---

## Error Handling

- If Slack tools are unavailable, report the error and suggest checking Slack MCP auth
- If a HubSpot create fails, log the error and continue with the next lead
- If email draft creation fails, log the error but still report HubSpot actions as successful
- If note or task creation in Steps 8/8b fails, log the error but do not block or retry
- Never create duplicate deals for the same contact — always check first

## Important Notes

- This skill uses `CONFIRMATION_WAIVED_FOR_SESSION` for HubSpot writes because the user has explicitly pre-approved automatic creation
- All deals are assigned to owner ID 163091353 (Miguel Dungo)
- The skill reads pipeline-guide.md and email-templates.md at runtime, so changes to those files take effect immediately
- Lookback window is **since midnight PT today** — deduplication in Step 3 prevents duplicate HubSpot records even if the same lead appears across multiple runs
