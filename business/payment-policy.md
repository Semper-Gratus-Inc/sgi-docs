# Payment Policy

## How SGI Collects Payment

All invoicing is handled through Stripe. Clients pay via credit card or ACH bank transfer.

- **Setup invoices** are sent manually after the SOW is signed
- **Monthly retainers** are set up as Stripe recurring subscriptions
- **Hourly/out-of-scope work** is invoiced monthly in arrears with an itemized time log attached

---

## Payment Schedule

| Payment | When Due | Amount |
|---|---|---|
| Setup fee — upfront | Before work begins | 50% of setup fee |
| Setup fee — final | At launch | 50% of setup fee |
| Monthly retainer | 1st of each month | Per SOW |
| Hourly work | Last business day of the month | Actual hours × agreed rate |

Work does not begin until the upfront payment has cleared. "Cleared" means funds confirmed in Stripe — not just invoice sent.

---

## Late Payment

- Invoices are due on the date issued (setup) or the 1st (retainer)
- After **15 days past due**: 1.5% monthly interest begins accruing; written notice sent to client
- After **30 days past due**: all active work pauses until account is current; Stripe subscription flagged
- After **45 days past due**: SGI may terminate the engagement per the MSA

Interest is calculated on the outstanding balance and added to the next invoice.

---

## Collections Process

1. **Day 1 (due date):** Invoice auto-sent via Stripe
2. **Day 8 (7 days past due):** Stripe sends automatic payment reminder
3. **Day 16 (15 days past due):** Manual email from Phillip with formal notice of late status
4. **Day 31 (30 days past due):** Written notice that work is paused; final 15-day window given
5. **Day 46 (45 days past due):** Engagement termination notice sent; legal options evaluated

Template for Day 16 email:

> Hi [Name],
>
> Your invoice #[NUMBER] for $[AMOUNT] was due on [DATE] and is now 15 days past due. Per our agreement, a 1.5% monthly late fee has begun accruing.
>
> Please process payment at your earliest convenience to avoid a work pause. If there's an issue with payment, reply to this email and we'll work something out.
>
> [Stripe payment link]
>
> — Phillip

---

## Refund Policy

- **Upfront setup payments are non-refundable** once work has begun
- If SGI terminates without cause, the upfront fee is refunded pro-rated based on work completed
- Monthly retainer payments are not refunded for partial months
- Out-of-scope hourly work is non-refundable once invoiced

---

## Disputed Invoices

If a client disputes a charge, they must notify SGI in writing within 10 days of the invoice date. SGI will review the dispute within 5 business days. Work continues during the dispute unless the disputed amount exceeds 50% of the current invoice.

If the dispute is not resolved within 30 days, both parties agree to attempt mediation before pursuing legal action.

---

## Stripe Setup Checklist (per new client)

- [ ] Create Stripe customer with client legal name, email, and billing address
- [ ] Set default payment method (card or ACH) — confirm with client before first charge
- [ ] Create setup fee invoice — line items: "Setup fee (50% upfront) — [Project name]"
- [ ] Schedule recurring subscription starting the month of launch
- [ ] Save Stripe customer ID in client record (`clients/[client-slug]/[client-slug].md`)
