# Central AI — Sales and Account Pipeline Rules

> This document defines the Sales and Accounts pipelines for Central AI. Use these definitions as the source of truth when assisting with deals, accounts, outreach sequences, and pipeline decisions.

---

## Sales Pipeline

Tracks every prospect from first contact to closed deal. The goal of this pipeline is to convert leads — inbound or outbound — into paying Central AI customers. A deal exits this pipeline only when it is Closed Won (handed off to the Accounts pipeline) or Closed Lost (moved to nurture).

| # | Stage | Probability | Definition |
|---|---|---|---|
| 1 | New Lead | 10% | A prospect has been identified or has entered the pipeline via inbound (trial sign-up, demo request), outbound, or marketing (paid ads, organic search, content, affiliates, app stores). No qualification has happened yet. |
| 2 | Qualified | 20% | Prospect has been confirmed as ICP-fit — not a bot, right vertical, decision maker identified, has inbound call volume, and a clear pain point around missed calls. |
| 3 | Demo Scheduled | 40% | A discovery/demo call has been booked. Prospect has shown enough interest to commit time. |
| 4 | Trial Started | 60% | Prospect has signed up for the free trial. Product is live on their number. |
| 5 | Trial Ended | 70% | The free trial has expired. Prospect is actively at the decision point — highest-leverage conversion window. |
| 6 | Proposal Sent | 80% | A specific plan recommendation has been formally presented to the prospect based on their needs. |
| 7 | Closed Won | 100% | Prospect has committed to a paid plan. Payment processed. Handed off to Accounts pipeline at Onboarding. |
| 8 | Closed Lost | 0% | Prospect has declined or gone silent for 14+ days. Loss reason documented. Added to nurture sequence. |

---

## Accounts Pipeline

Tracks every paying customer from their first day through their entire lifecycle with Central AI. The goal of this pipeline is to retain customers, ensure they are getting value, and identify opportunities to grow the account. A record enters this pipeline automatically when a Sales deal is Closed Won and only exits when the account is Churned.

| # | Stage | Probability | Definition | Transition Rule |
|---|---|---|---|---|
| 1 | Onboarding | 90% | New paying customer getting set up. Triggered automatically from Closed Won. | Moves to Healthy after **14 days** |
| 2 | Healthy | 100% | Account is fully active, paying, and engaging with the product regularly. Baseline state for all active customers. | Flagged for Customer Success review at **90 days** for potential Growth Opportunity |
| 3 | At Risk | 50% | Account showing danger signals — missed payment, no logins, dropped usage. Requires immediate CS action. | CS manually moves from Healthy when signals appear |
| 4 | Growth Opportunity | 90% | A Healthy account identified as a strong candidate for cross-sell. | CS confirms and moves manually after 90-day flag |
| 5 | Churned | 0% | Account has cancelled or been suspended. Loss reason documented. Added to win-back sequence. | Moved manually by CS after cancellation confirmed |

---

## Pipeline Handoff Rule

When a Sales deal reaches **Closed Won**, a new record is automatically created in the Accounts pipeline at **Onboarding**. The following information must transfer at handoff:

- Plan signed up for (Standard, Growth, Scale, Enterprise)
- Lead source (inbound, outbound, or marketing)
- Whether the account was activated during the trial (calendar connected, first real call received)
- Assigned AE name for CS context

---

## Key Definitions

- **ICP (Ideal Customer Profile)** — The type of business that is the best fit for Central AI. Characteristics: 1–25 employees, phone-dependent SMB, cannot afford a full-time receptionist, loses revenue from missed calls. Priority verticals: real estate, law firms, home services, beauty and wellness, healthcare, financial services.
- **Trial** — A 10-day free trial where the product is live on the prospect's number. Trials are tracked in the Sales pipeline, not the Accounts pipeline. An account only moves to the Accounts pipeline upon conversion to a paid plan.
- **Activation** — The milestone where a trial or new paying account has added their website URL, connected their calendar, and received at least one real call through Central.
- **At Risk** — Covers both payment issues (past due) and engagement issues (no logins, dropped call volume). Either signal qualifies an account for this stage.
- **Growth Opportunity** — Healthy accounts that have been active for 90+ days and are candidates for cross-sell into Central's free built-in products: Scheduler, CRM, Builder, or Docs.

---

## Outreach Sequences

Outreach sequences are tied to pipeline stages. Each stage in both the Sales and Accounts pipelines has a corresponding sequence. Sequences use a mix of automated and human-led touches across email, LinkedIn, phone, and SMS.

| Sequence | Pipeline | Trigger |
|---|---|---|
| New Lead | Sales | Lead enters pipeline |
| Qualified | Sales | Deal moves to Qualified |
| Demo Scheduled | Sales | Demo booked |
| Trial Started | Sales | Trial activated |
| Trial Ended | Sales | Trial expires |
| Proposal Sent | Sales | Proposal delivered |
| Closed Lost | Sales | Deal marked lost |
| Onboarding | Accounts | Closed Won triggers Accounts pipeline |
| At Risk | Accounts | CS flags account |
| Growth Opportunity | Accounts | 90-day Healthy flag |
| Churned | Accounts | Account cancelled |

*Sequence details to be defined separately per stage.*

---

*Last updated: March 2026 | Central AI — Sales & Onboarding | Confidential*
