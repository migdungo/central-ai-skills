# Email Templates & Style Guide

This file defines how the slack-lead-capture skill composes outreach emails to leads. The skill reads this file before drafting or sending any email.

---

## Tone & Voice

- Personal, direct, and concise. Written as Miguel, founder of Central AI.
- No corporate jargon. Short sentences. Conversational.
- Always use the lead's first name.
- Sign off as "Miguel"
- Max ~100 words per email body.
- Never use emojis in subject lines.
- Never use em-dashes (`—`). Use a comma, period, or rewrite the sentence instead.
- Do not include links unless the template specifies one. The scheduling link (https://meet.trycentral.com/miguel/quick-call) is included in templates where a CTA is appropriate — use exactly that URL, no shortening or paraphrasing.

---

## Template: Trial Started (gtm-new-trial-info)

**Trigger:** Lead activated a free 10-day trial.

**Subject:** Your Central trial is live

**Message:**
> Your 10-day trial is live. Glad you're here.
>
> Quick question before you dive in — how are you planning to use Central for your business?
>
> If it's easier to chat, grab a time here: https://meet.trycentral.com/miguel/quick-call
>
> Miguel

---

## Template: New Signup (central-new-signups)

**Trigger:** New signup entered the pipeline. Qualification unknown — do not send an outreach email yet. Instead, only create the HubSpot record (contact + deal at New Lead). Skip email drafting for this channel unless the signup data clearly indicates a real, qualified lead (has a business name, business email, or phone number). If it looks like a real lead, use the Qualified template below.

**Skip email if:** The email is a free consumer domain (gmail, yahoo, hotmail, etc.) with no company name — likely a bot or casual signup.

---

## Sequence: New Lead (Deal enters New Lead stage)

Three-email sequence over 7 days. Goal: welcome and book a call. Stop sequence on reply or meeting booked.

### Day 0 — Welcome

**Subject:** Welcome to Central, {firstname}

**Message:**
> Welcome — glad you signed up.
>
> I'd love to show you around and make sure you get the most out of it. Want to grab a quick call?
>
> https://meet.trycentral.com/miguel/quick-call
>
> Miguel

---

### Day 3 — Value nudge

**Subject:** One thing Central can handle for you

**Message:**
> Most of our customers come in trying to solve one thing — missed calls, chasing leads, inbox overload.
>
> What's the biggest thing eating your time right now?
>
> Happy to show you how Central handles it: https://meet.trycentral.com/miguel/quick-call
>
> Miguel

---

### Day 7 — Last touch

**Subject:** Still worth a chat?

**Message:**
> Didn't want to let this slip. Still happy to do a quick call if the timing is right.
>
> https://meet.trycentral.com/miguel/quick-call — or just reply and let me know where you're at.
>
> Miguel

---

## Template: Qualified Prospect

**Trigger:** Lead confirmed as ICP-fit (used for real signups from central-new-signups that look like businesses).

**Subject:** Quick question, {firstname}

**Message:**
> Running a business means a lot of moving parts. Central helps SMBs manage them without adding to their team.
>
> Would you want to see how it works on a quick demo, or would you rather try it yourself with a free 10-day trial?
>
> https://meet.trycentral.com/miguel/quick-call
>
> Miguel

---

## Template: Customer Feedback Response (customer-feedback)

**Trigger:** Existing customer left feedback (positive, negative, or feature request).

**Subject:** Re: your feedback

**For positive feedback:**
> Thanks so much for sharing that — really means a lot to hear.
>
> Is there anything we can do to make Central even better for you?
>
> Miguel

**For negative feedback / complaints:**
> Thanks for letting me know — I'm sorry to hear that.
>
> Would you be open to jumping on a quick call so I can understand what happened and make it right?
>
> https://meet.trycentral.com/miguel/quick-call
>
> Miguel

**For feature requests:**
> Thanks for the suggestion — noted and appreciated.
>
> I'll follow up if we move on it. Anything else on your mind?
>
> Miguel

---

## General Guidelines

- Always draft, never send directly (until auto-send is enabled)
- Use `text/plain` content type. Write plain text only — no HTML tags. Separate paragraphs with a blank line (`\n\n`). For links, write the URL directly (e.g. https://meet.trycentral.com/miguel/quick-call).
- From: miguel@trycentral.com — Gmail MCP must be connected to this account, not migdungo@gmail.com
- Do not include unsubscribe links or legal footers — these are personal 1:1 emails
- Personalize with the lead's first name in the greeting where templates use {firstname}
