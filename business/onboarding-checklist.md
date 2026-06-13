# Client Onboarding Checklist

This checklist covers everything from "prospect says yes" to "work has begun." Complete every step in order. Do not start development until all pre-work items are checked off.

---

## Stage 1 — Verbal Agreement

> Triggered when a prospect says they want to move forward.

- [ ] Confirm which pricing tier applies (Starter / Business / Pro / Enterprise)
- [ ] Identify any add-ons
- [ ] Confirm there are no show-stopping scope items that need a custom quote
- [ ] Send a follow-up email within 24 hours summarizing what was agreed verbally and next steps

**Email template:**

> Hi [Name],
>
> Great talking with you. To confirm what we discussed: you're interested in the [Tier] package at $[setup fee] setup + $[monthly]/mo. Next step is for me to send you the MSA and Statement of Work for review. Once both are signed and the upfront payment is received, we'll schedule a kickoff call and get started.
>
> I'll have documents to you within [2 business days]. Let me know if you have any questions.
>
> — Phillip

---

## Stage 2 — Documents

- [ ] Fill in MSA template (`business/msa-template.md`) with client legal name, address, state/county
- [ ] Fill in SOW template (`business/sow-template.md`) with project name, deliverables, timeline, fees
- [ ] Double-check that the SOW explicitly lists out-of-scope items discussed
- [ ] Send MSA + SOW to client for review (email with PDF attachments or e-signature link)
- [ ] Log send date — follow up in 3 business days if no response

---

## Stage 3 — Signatures & Payment

- [ ] Receive signed MSA from client
- [ ] Receive signed SOW from client
- [ ] File both signed documents in `clients/[client-slug]/contracts/`
- [ ] Create Stripe invoice for upfront payment (50% of setup fee)
- [ ] Send Stripe invoice — include SOW reference in the invoice description
- [ ] Confirm payment received and cleared (do not start work on pending)
- [ ] Send payment confirmation email and estimated kickoff date

---

## Stage 4 — Kickoff Setup

- [ ] Create client folder: `clients/[client-slug]/`
- [ ] Create client record: `clients/[client-slug]/[client-slug].md` (copy from `clients/gym-client.md` as template)
- [ ] Create GitHub repo(s) under `Semper-Gratus-Inc` org if new project
- [ ] Add client to AWS resources if needed (separate account or prefix)
- [ ] Collect all client-supplied materials (logo, copy, credentials) — log what's missing
- [ ] Schedule kickoff call — confirm attendees, agenda, and dial-in details

---

## Stage 5 — Kickoff Call

Agenda (30–45 min):

1. Introductions and project overview
2. Walk through SOW deliverables — confirm client understands what's included
3. Confirm primary point of contact and how decisions get made
4. Review timeline and key dates
5. Discuss communication expectations (email-first, 1 business day response, no unscheduled calls)
6. Collect any outstanding materials

- [ ] Kickoff call completed
- [ ] Notes sent to client within 24 hours summarizing decisions and action items
- [ ] Outstanding materials logged with follow-up date

---

## Stage 6 — Development Begins

- [ ] All client materials received (or formally deferred with agreed timeline)
- [ ] GitHub repo set up with correct branch structure per coding standards
- [ ] Development environment configured and tested
- [ ] First status update sent to client (weekly cadence begins)

---

## Ongoing — Monthly Billing

- [ ] Stripe recurring invoice configured for 1st of month (begins month project goes live)
- [ ] Confirm billing email is correct in Stripe
- [ ] Note in client record: retainer start date, monthly amount, billing day

---

## Offboarding (when engagement ends)

- [ ] Final invoice sent and collected
- [ ] All code, assets, and credentials transferred to client
- [ ] Client confirms receipt in writing
- [ ] Stripe subscription cancelled
- [ ] Request testimonial / referral (optional but ask every time)
- [ ] Archive client folder, close GitHub issues
