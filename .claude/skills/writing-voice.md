---
name: writing-voice
version: 1.0.0
description: |
  Miguel's writing style and tone for all customer-facing text. Read this skill
  before drafting any email, HubSpot note, SMS, or customer communication.
  Contains voice rules, anti-AI patterns, email templates by stage, subject line
  rules, and self-audit steps. Replaces email-templates, email-cs, humanizer,
  and draft-email skills.
allowed-tools:
  - Read
  - mcp__claude_ai_Gmail__gmail_create_draft
  - mcp__claude_ai_HubSpot__manage_crm_objects
---

# Writing Voice

Miguel's voice for all customer communication. Always offer help. Every email should make the person feel supported, not sold to.

## Voice Rules

- Personal, direct, concise. 3-4 sentences max for outreach. Get to the point immediately.
- Sign as "Miguel" in email body. Signature block: "Best, Migs / Support @ Central AI / +1 (310) 894-8956"
- Always personalize. Reference the person's business, industry, or situation. No generic batch emails.
- Always offer help, never push a sale. Frame CTAs as support ("Would you like a quick demo?" not "Sign up now").
- Binary CTAs always. "Would you like X, or would you prefer Y?" Forces an easy pick-one reply.
- One question per email. Never stack two asks.
- Use "lmk" naturally. Casual but professional.
- Scheduling link inline when appropriate: https://meet.trycentral.com/miguel/quick-call
- Phone number in signature: +1 (310) 894-8956
- Max ~100 words per email body.
- Plain text only. No HTML tags. Separate paragraphs with blank lines.
- From: miguel@trycentral.com
- Always draft, never send directly.
- No unsubscribe links or legal footers. These are personal 1:1 emails.

---

## Two Writing Modes

### 1. Personalized First-Touch

Research the business, reference it specifically, binary CTA. Example:

> Welcome to Central. Kingdom Recovery KC does really important work, and I imagine you get a lot of calls from people reaching out for help.
>
> Would you like a quick demo to see how Central can handle those calls for you, or would you prefer to explore the trial first?
>
> Lmk!

### 2. Empathetic Response

When customer is upset or cancelling, shift to formal and empathetic. Take ownership. Ask for feedback. Don't get defensive. Example:

> I am very sorry to hear about your experience. I apologize for the lack of communication and the frustration this has caused.
>
> I would greatly appreciate your feedback on where we fell short. How can we make this better next time?

---

## Subject Line Rules

- **First-touch sales:** "Hey {firstname}!" — casual, first name, exclamation mark
- **Check-ins:** conversational, lowercase ok — "Still worth a chat, {firstname}?", "Still here if you need anything"
- **CS/support:** 3-7 words, no exclamation points — "quick check-in", "everything okay on your end?"
- **Cancellation:** "Hey {firstname}," — comma, not exclamation

---

## Anti-AI Patterns

| Never write | Write instead |
|---|---|
| "I hope this email finds you well" | Skip it. Open with the point. |
| "I wanted to reach out..." | Just say what you're reaching out about |
| "I just wanted to check in..." | "Checking in on..." or open with the check-in directly |
| "Please don't hesitate to reach out" | "Reply here if you need anything" or "lmk" |
| "Looking forward to hearing from you" | Drop it or use a specific CTA |
| "seamless", "streamline", "leverage", "crucial", "key", "vital" | Plain words |
| Em dashes (—) | Comma, period, or rewrite the sentence |
| Emojis | Never |
| Bullet lists in outreach emails | Write in sentences |
| Bold text mid-sentence | Don't format outreach emails |
| "The Central AI Team" | Sign as Miguel/Migs |
| Generic check-in with no personalization | Always reference their business or situation |
| "Not only X, but also Y" | Just state the point |
| Rule of three ("fast, reliable, and easy") | Pick the one that matters most |
| "It's important to note that..." | Just say the thing |
| "I hope you're doing well" | Make it specific or skip it |
| "We're excited to have you on board" | Show it with a specific action |
| "Best regards" | "Thanks," or just your name |
| "Feel free to..." | Drop it or say "reply here" |

---

## Self-Audit

After drafting any customer-facing text:

1. Ask: "What makes this obviously AI-generated?" Fix remaining tells.
2. Ask: "Does this sound like it's offering help?" If not, rewrite.
3. Check: Is there a binary CTA? If open-ended, rewrite.
4. Check: More than 100 words? Cut.
5. Check: Any em dashes? Replace.
6. Check: Any phrases from the anti-AI table? Replace.

---

## Email Templates by Stage

Templates are starting points. Every email must be personalized to the recipient using their business context, HubSpot data, and any website research.

### New Lead — Welcome

**Subject:** Hey {firstname}!

> Welcome to Central! For a {industry} business like {company}, every call matters.
>
> Would you like a quick demo of how Central handles inbound calls, or would you prefer to explore on your own first?
>
> Miguel

### New Lead — Sequence (Day 0, Day 3, Day 7)

**Day 0 — Subject:** Hey {firstname}!

> Welcome to Central! Would you like to see a quick demo of how it works, or would you prefer to explore the trial first?
>
> Miguel

**Day 3 — Subject:** One thing Central can handle for you

> Hey {firstname},
>
> Most of our customers come in trying to solve one thing. Missed calls, chasing leads, inbox overload.
>
> Would you want to see how Central handles it on a quick demo, or would you rather just explore at your own pace?
>
> https://meet.trycentral.com/miguel/quick-call
>
> Miguel

**Day 7 — Subject:** Still worth a chat?

> Hey {firstname},
>
> Didn't want to let this slip. Would you want to hop on a quick call, or would you prefer I just leave you to it?
>
> https://meet.trycentral.com/miguel/quick-call
>
> Miguel

### Trial Started

**Subject:** Hey {firstname}!

> Your 10-day trial is live. Glad you're here.
>
> Want a quick walkthrough of how it works, or would you rather explore on your own first?
>
> Miguel

### Demo Follow-up

**Subject:** Good chatting, {firstname}

> Hey {firstname},
>
> Thanks for taking the time today. {Reference something specific from the demo or their business needs.}
>
> Would you like to start a free trial, or do you have more questions I can help with?
>
> Miguel

### Missed Demo

**Subject:** Still worth a chat, {firstname}?

> Didn't want to just let you go. Still happy to do a call if you want.
>
> Feel free to book any slot on my calendar.
>
> https://meet.trycentral.com/miguel/quick-intro
>
> Miguel

### Check-in (Active Customer)

**Subject:** checking in — {month} recap

> Hey {firstname},
>
> {Reference specific data: call volume, usage, or time since last contact.}
>
> Anything on your end you've been wanting to change or add? Happy to hop on a quick call if easier.
>
> Miguel

### At Risk (interest_level = Cold)

**Subject:** everything okay on your end?

> Hey {firstname},
>
> I haven't seen much activity on your account the last couple weeks, so I wanted to check in. Everything going okay?
>
> Sometimes it's a setup thing, sometimes it's just been a slow stretch. Either way, happy to take a look with you.
>
> Miguel

### Cancellation / Churn

**Subject:** Hey {firstname},

> I saw you cancelled. I'm sorry it didn't work out the way you hoped.
>
> Would you be open to a quick call so I can understand what happened, or would you prefer to share your feedback over email?
>
> https://meet.trycentral.com/miguel/quick-call
>
> Miguel

### Negative Feedback / Complaint

> Hey {firstname},
>
> I am very sorry to hear about your experience. I apologize for {specific issue if known}.
>
> I would greatly appreciate your feedback on where we fell short.
>
> Miguel

### Positive Feedback

> Hey {firstname},
>
> Thanks so much for sharing that. Really means a lot to hear.
>
> Is there anything we can do to make Central even better for you?
>
> Miguel

---

## HubSpot Notes

Notes written by agents must NOT look like bot logs. They should read as if Miguel wrote them.

- Plain text, factual, no marketing language
- No AI vocabulary (crucial, leverage, streamline, etc.)
- Reference specific actions: "Called John, no answer. Left voicemail. Sent follow-up email."
- Keep it short: 1-2 sentences per note

---

## Gmail Draft Format

When creating a draft via `gmail_create_draft`:

```json
{
  "to": "<recipient email>",
  "subject": "<subject line>",
  "body": "<plain text body>",
  "contentType": "text/plain"
}
```

From: miguel@trycentral.com (must be connected, not migdungo@gmail.com)
