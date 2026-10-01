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

`get_company_info`: confirm the company and check `scopes` includes
`write:draft`. If not, ask the user to reconnect Numina.

## 2. Accounts, once

`list_accounts` and `list_vat_codes`. Identify the sales account(s), the
Danish output VAT code, the code for EU sales without Danish VAT and the
fees account. Ask when unsure.

## 3. Each payout

1. **Find the bank line.** `search_bank_transactions` with `text: "stripe"`,
   `amount_min` = `amount_max` = the payout amount (positive, money in) and
   `state: "unreconciled"`. Each result has the bank's
   `ledger_account_number`. Skip lines with `agent_processing: true`;
   Numina is already on them. No line → the payout hasn't arrived yet; skip
   it.
2. **List what's in the payout** from its Stripe balance transactions. Use
   their settlement amounts (`amount`, `fee`): they're already in the payout
   currency.
3. **Group the charges:**
   - **Denmark** → Danish output VAT code
   - **EU business with a VAT number** → EU sale, no Danish VAT
   - **EU consumer** → OSS if the company is registered for it, otherwise
     Danish VAT; ask the user
   - **outside the EU** → no VAT
4. **`create_draft`** with `bank_transaction_id`, `date` = the payout date,
   and lines that sum to 0:
   - bank account (`ledger_account_number`): the payout, **positive**
   - sales per group: **negative**, gross with a Danish VAT code
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
a link to the draft. Remind the user to approve the drafts in Numina.
