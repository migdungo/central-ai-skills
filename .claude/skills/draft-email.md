# Draft Email Skill

Draft a personalized outreach email using Miguel's email style and templates.

## How to invoke

```
/draft-email
```

Then provide the recipient details, or include them inline:

```
/draft-email Name: Hector Lopez, Company: Lopez Law Firm, Context: just started a trial
```

## What to ask for (if not provided)

1. **Name** — recipient's first name (and last name / company if known)
2. **Context** — what triggered this email? Examples:
   - Just started a trial
   - New signup (business email / looks like a real lead)
   - Qualified prospect (ICP fit, reached out directly)
   - Positive feedback
   - Negative feedback / complaint
   - Feature request
   - Custom (describe the situation)
3. **Any personalization** — anything notable about them or their business (optional)

## Instructions

1. Read `.claude/skills/email-templates.md` for tone, voice, and templates.
2. Select the appropriate template based on the context provided.
3. Personalize: use first name in subject and/or opener, weave in any business context if provided.
4. Output the draft in this format:

---

**To:** [email if known, otherwise leave blank]
**Subject:** [subject line]

[body]

---

5. After showing the draft, ask: "Want me to save this as a Gmail draft, or does it look good to send manually?"
6. If user confirms → use Gmail MCP (`gmail_create_draft`) to save it as a draft in miguel@trycentral.com.
   - Use `text/plain` content type
   - Do not HTML-encode the body
   - From: miguel@trycentral.com
