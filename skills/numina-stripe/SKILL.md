---
name: numina-stripe
description: Draft Stripe payouts in Numina against the bank line, split into sales, VAT, refunds and fees. Use when the user asks to book, sync or reconcile their Stripe revenue.
---

# Stripe → Numina

Needs **Stripe** (payouts and balance transactions) and the **Numina MCP**
with write access ([setup](../../README.md#connect)). One payout = one draft
on the bank line it landed on. The user books the drafts in Numina.

## 1. Company and period

`get_company_info`: confirm the company and check `scopes` includes
`write:draft`. If not, ask the user to reconnect Numina.

## 2. Accounts

`list_accounts` and `list_vat_codes` once. Identify sales account(s), the
output VAT code, an EU-sale code without Danish VAT, and the fees account.
Ask when unsure.

## 3. Each payout

1. **Find the bank line.** `search_bank_transactions` with `text: "stripe"`,
   the payout amount and `state: "unreconciled"`. Skip lines with
   `agent_processing: true`; Numina is already on them. No line yet → the
   payout hasn't arrived; skip it.
2. **Bank account.** `get_bank_transaction` gives the bank's ledger account.
3. **Split the payout** from its Stripe balance transactions:
   - Denmark → Danish output VAT code
   - EU business with VAT number → EU sale, no Danish VAT
   - EU consumer → OSS if the user is registered, otherwise Danish VAT; ask
   - outside the EU → no VAT
4. **`create_draft`** with `bank_transaction_id` and lines that sum to 0:
   - bank account: the payout, **positive**
   - sales per VAT group: **negative**, gross with Danish VAT codes
   - refunds: **positive** on the same sales accounts and codes
   - fees: **positive** on the fees account (Stripe fees are usually
     VAT-exempt; follow the code the company already uses)

   Payout in EUR → `currency: "EUR"` and the bank's DKK amount as
   `amount_dkk` on the bank line.
5. If `create_draft` refuses because the lines don't balance, recheck the
   split rather than plugging the difference.

`reasoning` example: "Stripe payout po_123: 42 charges, 1 refund, fees."

## 4. Report back

Per payout: amount, sales, VAT, refunds, fees, and the draft link. Remind
the user to approve the drafts in Numina.
