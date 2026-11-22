---
name: numina-stripe
description: Draft Stripe payouts in Numina against the bank line, split into sales, VAT, refunds and fees. Use when the user asks to book, sync or reconcile their Stripe revenue.
---

# Stripe → Numina

Needs **Stripe** (payouts and balance transactions) and the **Numina MCP**
with write access ([setup](../../README.md#connect)). One payout = one draft
on the bank line it landed on. The user approves and books the drafts in
Numina.

## 1. Company and period

`get_company_info`: confirm the company. Check that `create_draft` is among
your Numina tools: Numina only lists the tools the connection may use. If
it's missing, the connection is read-only; ask the user to reconnect Numina
with write access.

## 2. Accounts, once

`list_accounts` and `list_vat_codes`. Identify the sales account(s), the
Danish output VAT code, the code for EU sales without Danish VAT and the
fees account. Ask when unsure.

## 3. Each payout

1. **Find the bank line.** `search_bank_transactions` with `text: "stripe"`,
   `amount_min` = `amount_max` = the payout amount (positive, money in) and
   `state: "unreconciled"`, and `date_from` = the payout's `arrival_date`,
   `date_to` = five days later. Each result has the bank's
   `ledger_account_number`. Skip lines with `agent_processing: true`;
   Numina is already on them. No line → the payout hasn't arrived yet; skip
   it.

   Use a line only when it's the single match in the payout's currency.
   Payouts of the same amount are common with subscriptions, and a draft
   on the wrong line still balances. With several matches, take the one
   dated on the `arrival_date`; if that doesn't leave exactly one, list the
   candidates and ask the user.
2. **List what's in the payout** from its Stripe balance transactions. Use
   their settlement amounts (`amount`, `fee`): they're already in the payout
   currency.

   Stripe can only list this for automatic payouts. If the payout's
   `automatic` is false, don't draft it: report it without a breakdown and
   ask the user which transactions it covers.
3. **Group the charges:**
   - **Denmark** → Danish output VAT code
   - **EU business with a VAT number** → EU sale, no Danish VAT
   - **EU consumer** → OSS if the company is registered for it, otherwise
     Danish VAT; ask the user
   - **outside the EU** → no VAT
4. **`create_draft`** with `bank_transaction_id`, `date` = the payout date,
   and lines that sum to 0:
   - bank account (`ledger_account_number`): the payout, **positive**
   - sales per group: **negative**, with that group's VAT code from step
     3. A group on Danish VAT takes the gross amount and the Danish output
     VAT code. EU business sales take the EU sale code, and sales outside
     the EU take no VAT code: never put Danish VAT on those.
   - refunds: **positive**, on the same accounts and codes as the sale
   - fees: **positive** on the fees account. Stripe's fees are usually
     VAT-exempt; follow the code the company already uses for them.

   Payout in another currency (e.g. a EUR bank account)? Set `currency`;
   Numina converts at the day's rate.
5. If `create_draft` refuses because the lines don't balance, recheck the
   grouping. Never plug the difference.

`reasoning` example: "Stripe payout po_123: 42 charges, 1 refund, fees."

## 4. Report back

Per payout: amount, sales per VAT group, refunds, fees and
a link to the draft. List the payouts you left out: not arrived, several
possible bank lines, or manual. Remind the user to approve the drafts in
Numina.
