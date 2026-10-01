---
name: numina-stripe
description: Book Stripe payouts, fees, refunds and VAT in Numina as draft journal entries. Use when the user asks to book, sync or reconcile their Stripe revenue.
---

# Stripe → Numina

You need two connections ([setup](../../README.md#connect)): **Stripe** (read balance transactions and payouts)
and the **Numina MCP with write access** (create draft journal entries).

## 1. Confirm the company and period

- Call `get_company_info` and check that `scopes` includes write access.
  If it is read-only, stop and tell the user to reconnect with write access.
- Agree on a period. Work payout by payout: each Stripe payout becomes one
  draft entry that matches the amount landing in the bank.

## 2. Load the chart of accounts once

`list_accounts` and `list_vat_codes`. Identify:

- Stripe clearing account (ask if there is none)
- Sales account(s) and output VAT
- Payment fees account
- Bank account the payouts land in

## 3. For each payout

1. Skip it if `search_ledger_entries` already finds the payout ID or the
   same date and amount on the bank account.
2. List the balance transactions in the payout (charges, refunds, fees,
   adjustments).
3. Split sales by VAT treatment using the customer's country and whether
   the customer is a business with a VAT number:
   - Denmark → 25 % Danish VAT
   - EU business with VAT number → reverse charge, no Danish VAT
   - EU consumer → OSS if the user is registered, otherwise Danish VAT; ask
   - Outside the EU → no VAT
4. Create a **draft** journal entry:
   - credit sales (net) and output VAT per group
   - debit refunds against the same accounts
   - debit fees (Stripe processing fees are usually VAT-exempt; follow the VAT code the company already uses for them)
   - debit the Stripe clearing account with the payout amount
5. Check that the draft balances and that the payout amount equals what hit
   the bank (`search_ledger_entries` on the bank account). Flag mismatches.

## 4. Report back

Per payout: amount, sales, VAT, fees, refunds, and whether it matched the
bank. Remind the user that drafts must be approved in Numina.
