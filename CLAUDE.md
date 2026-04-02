# Central AI — Product Expert Knowledge Base

> Source: guide.trycentral.com (crawled 2026-03-26, 30+ pages)
> Use this file to answer any questions about Central AI's products, features, integrations, pricing, and setup.

---

## Product Overview

Central builds AI employees to help run businesses, stay organized, and manage life. The platform consists of:

**3 AI Agents (paid):**
- **Central Front Desk** — AI phone reception and omnichannel chat (calls, texts, WhatsApp, social)
- **Central Sales** — AI CRM with sales automation (voice, text, LinkedIn, email outreach)
- **Central Executive Assistant** — Email, calendar, tasks, meeting recording, agentic assistant

**4 Free Platform Apps:**
- **Scheduler** — Fast appointment booking for individuals and teams; round-robin assignment
- **CRM** — Free customer relationship management with bidirectional integrations and AI features (premium tiers available)
- **Docs** — Document editor with knowledge base integration
- **Ask Central** — Agentic assistant for multi-step automation across connected apps

**Key Links:**
- App: https://app.trycentral.com
- Book a demo: https://meet.trycentral.com/miguel
- Live chat support: https://trycentral.com
- Email support: miguel@trycentral.com

---

## AI Front Desk — Voice Receptionist

### What It Does
24/7 AI phone receptionist that handles inbound calls, books appointments, qualifies leads, routes calls, and responds in multiple languages.

### Setup (Quickstart)
1. Create account and select receptionist name, voice, and greeting message
2. Add knowledge (websites, documents, text, FAQs) via the "Knowledge" section in the left nav
3. Reserve a local or toll-free phone number (multiple countries supported)
4. Select a plan — all plans include a **10-day free trial**
5. Test by calling the assigned number (shown top-right of dashboard)
6. Iterate: update knowledge or modify the system prompt via Advanced Settings

### Knowledge Sources
- Website URLs (auto-scanned)
- Uploaded documents/files
- Manual text snippets
- FAQ lists

### Customization & Prompt Engineering
- Location: Personalization > Manage > Business Description/Prompt
- Customize AI behavior via prompt engineering to handle specific scenarios
- Example: Ask "Is this a new or existing case?" and route new cases to the user while existing cases get general support
- AI can escalate calls, book meetings, and query knowledge bases
- Cannot perform actions beyond its integrated capabilities (e.g., updating HubSpot requires the HubSpot integration)

### Call Routing & Escalation
- Access: Right sidebar on dashboard > "Escalation" > "Manage"
- Options:
  - "Never transfer to a human"
  - Route to selected contacts
- Default contact: Signup phone number, 9am–5pm Mon–Fri (customizable)
- Add multiple contacts for different departments, partners, or team members with separate availability windows
- Use cases: law firms (multiple partners), SaaS companies (sales + support teams)

### Warm Transfer
- **Enabled:** AI calls the recipient first, provides a summary of the caller, and requests confirmation before connecting — recipient can decline
- **Disabled (default):** Immediate connection or voicemail
- Toggle: Workflows > Warm Transfer > Toggle on (takes effect immediately)

### Language Support
- Multilingual — AI Receptionist can handle calls in multiple languages
- Configuration available in dashboard settings

### Notification Settings
- Configurable per user in the dashboard
- Escalation and call routing settings in right sidebar

### Mobile App
- Access AI Receptionist management on mobile
- Supports automatic meeting booking, call routing, escalation management
- Full availability/schedule management from mobile

### Lead Intake & Qualification Flows
Four core components:
1. **Lead Capture** — Collect name, phone, email (keep short to prevent caller drop-off)
2. **Lead Qualification** — AI asks strategic questions (budget, timeline, decision-maker status)
3. **Lead Actions** — Execute predetermined responses (schedule calls or conclude) based on qualification results
4. **Trigger Mode** — Start on every call, when AI detects genuine interest, or "Let AI decide"

Define "Good Answer Examples" to help the AI distinguish qualified from unqualified prospects.

### Sync Leads to CRM
- Native connections: HubSpot, Pipedrive, Zoho CRM
- Setup: Leads section > "Sync" button > Choose CRM > Complete in-app wizard
- Also supports Zapier for any other tool: https://zapier.com/apps/central-ai/integrations

### Phone Forwarding (Connect Existing Number)
Forward your existing number to the AI Receptionist number:

**U.S. Mobile Carriers:**
| Carrier | Enable | Disable |
|---|---|---|
| Verizon / AT&T | `*72 [number]` | `*73` |
| T-Mobile | `**21*[number]#` | `##21#` |
| US Cellular / Boost | `*72 [number]` | `*720` |
| Cricket Wireless | `*72 [number]` | `*730` |
| Google Fi | Use Fi app settings (no star codes) | — |
| Xfinity Mobile | Use Settings menu only | — |

**Landlines:** Dial `*72`, wait for confirmation tone, enter receptionist number + `#`, hang up.

**VoIP/Cloud Platforms:** Google Voice, OpenPhone, Dialpad, RingCentral — use admin dashboard forwarding settings.

**Test:** Call your original number from another phone; the AI Receptionist should answer.

---

## AI Chat Agent (Front Desk Chat)

### What It Does
Omnichannel AI chat agent that handles messaging across website, SMS, and WhatsApp — same AI brain as the voice receptionist.

### Website Widget Setup
Supported platforms: HTML, React/Next.js/CRA/Vite, WordPress, Shopify, Wix, Squarespace, Ghost, Webflow, Framer

**Core pattern:**
1. Get custom HTML snippet from Integrations dashboard
2. Paste into site header/footer code area
3. Save and publish
4. Verify chat bubble appears after hard refresh (CTRL+F5 / Command+Shift+R)

**Platform-specific notes:**
- **React:** Import `ChatWidget` component, place in root component
- **WordPress:** Use "Insert Headers & Footers" plugin (avoids editing theme directly)
- **Traditional CMS:** Paste into code injection / theme editor

### SMS Integration
Setup path: Chats > Integrations > SMS

1. Select a toll-free phone number (local numbers cannot be used for SMS)
2. Connect chat agent (new or existing)
3. Click "Start Compliance"
4. Enter business information for carrier registration
5. Complete messaging use case details:
   - Monthly messaging volume estimate
   - Opt-in type (verbal, web form, paper form, text, QR code)
   - Opt-in policy image URLs (must be publicly accessible without login)
   - Use case category, description, and sample messages
6. Submit for review — approval takes **3–5 business days** (via Twilio)

### WhatsApp Integration
Setup path: Chats > Integrations > WhatsApp

1. Click "Continue with Facebook" (enable popups if blocked)
2. Log in with Facebook
3. Enter business details (portfolio, name, website, country)
4. Choose new or existing WhatsApp Business account
5. Review and approve permissions
6. Select phone number (Toll-Free or Local — Toll-Free numbers can be reused for SMS)
7. Click "Register Number"
8. Connect chat agent (new or existing)
9. Bot is active and ready to test

---

## Meeting Booking & Scheduling

### Supported Schedulers
- Google Calendar
- Cal.com
- Calendly *(note: callers may wait up to 3 minutes for confirmation)*
- Zoho Calendar
- Clio Calendar
- **Central Scheduler** (recommended — faster and free)

### Setup
1. Access Scheduler feature from dashboard
2. Click "Click to add a scheduler"
3. Tap "+ Add"
4. Connect via API key or login redirect (varies by platform)

For schedulers not listed: contact support@trycentral.com

---

## AI Sales

### AI Outreach & Sequences
AI-powered personalized outreach campaigns across LinkedIn, email, SMS, and calls.

**AI Sequence Generator (9-step process):**
1. Central Sales > Outreach > Sequences > "Generate with AI"
2. Input company name, language, website URL
3. Select leads (existing CRM contacts or new ones with AND/OR filter logic)
4. Enable auto-enrollment — Central automatically adds future contacts matching filters
5. AI analyzes lead personas and suggests pain points
6. AI generates value propositions addressing those pain points
7. Select channels: email, phone calls, SMS, LinkedIn (some require integration)
8. Review and customize campaign
9. Launch

### Find Leads
- Email discovery tool
- TCPA-compliant
- Part of the Central Sales suite

### Central CRM
- Free tier with core CRM features
- Premium tiers with AI features and deeper integrations
- Bidirectional sync with HubSpot, Zoho CRM, and Pipedrive

---

## AI Executive Assistant

### Core Features
- Email management
- Calendar management
- Task management
- Meeting recording and notes
- Ask Central agentic assistant
- Central Scheduler integration

### Create Tasks from Emails (Manual)
1. Central EA via product switcher
2. Select "Your Day" from main menu
3. Create tasks via drag-and-drop (drag emails to calendar dates) or email actions menu
4. Fill in task details (notes, attachments, priority, scheduling)
- View original email at any time via "View" button
- Modify or reschedule tasks from calendar view

### Automate Task Creation from Emails
Setup: Central EA > Settings > Automations > Enable "Allow AI to automatically create tasks from emails"

What it does:
- AI scans email and meeting content
- Identifies action items (e.g., "Can you have the proposal ready by Friday?")
- Auto-generates tasks with title, due date, and link to original email
- Keeps all work organized and prevents missed commitments

### Ask Central Assistant
- Multi-step agentic automation
- Works across all connected apps
- Available as a standalone free app and integrated into the EA product

---

## CRM Integrations

### HubSpot
**Purpose:** Bidirectional sync of contacts and companies between Central and HubSpot.

**Setup (9 steps):**
1. Central Sales via product switcher
2. Navigate to Integrations
3. Select HubSpot
4. Authorize connection
5. Log in through popup to grant permissions
6. Map: Central Contacts ↔ HubSpot Contacts; Central Companies ↔ HubSpot Companies
7. Use AI field mapping (automatically matches common fields)
8. Complete mapping
9. Finalize — sync begins automatically

**Result:** Any contact/company added or updated in either platform syncs to the other automatically.

### Zoho CRM
**Purpose:** Automatic sync of contacts and companies between Central and Zoho.

**Setup (9 steps):**
1. Central Sales via product switcher
2. Integrations menu
3. Choose Zoho CRM
4. Select "Connect App"
5. Authenticate with Zoho credentials (popup)
6. Approve permissions > "Continue to the App"
7. Click "Complete Setup and Continue"
8. Map: Central Contacts ↔ Zoho Contacts; Central Companies ↔ Zoho Accounts
9. Use AI mapping > "Save and Continue Sync"

**Result:** New or modified records sync automatically between platforms.

### Pipedrive
**Purpose:** Automatic sync between Central and Pipedrive.

**Setup (9 steps):**
1. Central Sales via product switcher
2. Integrations menu
3. Select Pipedrive
4. Click "Connect App"
5. Log in via popup
6. Review and approve permissions
7. Complete setup
8. Map: Central Contacts ↔ Pipedrive Contacts; Central Organizations ↔ Pipedrive Organizations
9. Save and continue sync

**Result:** All subsequent additions or updates sync automatically without manual intervention.

---

## Zapier Integration

**Purpose:** Connect Central AI to 8,000+ apps.

**Setup (10 steps):**
1. Accept Zapier invitation via provided link
2. Create new Zap via "Create" menu
3. Click "Trigger" in the new Zap
4. Search for "Central Reception" in Zapier directory
5. Select trigger event (e.g., "Call Completed") and authorize
6. Click "Authorize" and select workspace
7. Test trigger to verify connection
8. Configure action(s) via "Action" tab
9. Connect desired CRM or communication channels
10. Turn on Zap

**Directory:** https://zapier.com/apps/central-ai/integrations

---

## Pricing

All paid plans include a **10-day free trial**, no setup fees, cancel anytime. Free apps (Scheduler, CRM basic, Docs, Ask Central) are always free.

**Pricing page:** https://trycentral.com/pricing

### AI Receptionist (Voice)
| Plan | Price | Calls/mo | Key Features |
|---|---|---|---|
| Standard | $79/mo | 100 | 24/7 call answering, CRM sync |
| Growth | $149/mo | 200 | AI lead qualification, custom routing |
| Scale | $299/mo | 400 | Advanced analytics, Zapier integrations |
| Enterprise | Custom | Unlimited | Multi-location, dedicated account manager, compliance-grade data handling |

### Chat Agent
| Plan | Price | Chats/mo | Key Features |
|---|---|---|---|
| Starter | $39/mo | 300 | Customizable chatbot, scheduler sync |
| Business | $99/mo | 1,000 | Multi-platform (WhatsApp, Instagram, SMS) |
| Growth | $299/mo | 5,000 | Shopify integration, omnichannel support |
| Professional | $499/mo | 10,000 | Dedicated onboarding specialist, priority support |

### AI Outbound Caller (Sales)
| Plan | Price | Calls/mo | Key Features |
|---|---|---|---|
| Free | $0 | — | Customizable CRM, lead scoring, basic integrations |
| Starter | $199/mo | 1,100 | 20 sequences |
| Growth | $599/mo | 3,500 | Multichannel outreach (SMS, email) |
| Enterprise | $999/mo | 5,000 | Advanced analytics & reporting |

---

## Key URLs Reference

| Resource | URL |
|---|---|
| Main site | https://trycentral.com |
| App login | https://app.trycentral.com |
| Book a demo | https://trycentral.com/demo |
| Sign up (Front Desk) | https://app.trycentral.com/auth/signup?product=receptionist |
| Documentation | https://guide.trycentral.com |
| Zapier integrations | https://zapier.com/apps/central-ai/integrations |
| Email support | support@trycentral.com |
