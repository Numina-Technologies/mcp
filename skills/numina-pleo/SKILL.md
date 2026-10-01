---
name: numina-pleo
description: Book Pleo card expenses and reimbursements in Numina as draft journal entries with the receipt attached. Use when the user asks to book, sync or reconcile Pleo expenses.
---

# Pleo → Numina

You need two connections ([setup](../../README.md#connect)): **Pleo** (read expenses and receipts) and the
**Numina MCP with write access** (create draft journal entries and attach
documents).

## 1. Confirm the company and period

- Call `get_company_info` and check that `scopes` includes write access.
  If it is read-only, stop and tell the user to reconnect with write access.
- Agree on a period. Default: expenses settled last calendar month.

## 2. Load the chart of accounts once

- `list_accounts` and `list_vat_codes`. Find the Pleo clearing/bank account
  (often named "Pleo"). If there is none, ask the user which account Pleo
  card spend runs through.

## 3. For each Pleo expense

1. Skip it if `search_ledger_entries` already finds an entry with the same
   Pleo reference, or the same date, amount and supplier.
2. Pick the expense account from the Pleo category and the supplier. Look at
   how the same supplier was booked before (`search_ledger_entries`) and
   reuse that account and VAT code.
3. VAT: Danish suppliers with a valid receipt → deductible VAT per the VAT
   code. Foreign SaaS → reverse charge (EU/non-EU services). Meals and
   entertainment → 25 % of the VAT. When unsure, pick no deduction and flag
   it.
4. Create a **draft** journal entry: expense account (net), VAT account (if
   any), credit the Pleo account (gross). Date = transaction date. Text =
   supplier + Pleo note.
5. Attach the Pleo receipt to the draft. No receipt → still create the
   draft and flag "missing receipt".

Out-of-pocket expenses (udlæg) credit the employee's payable account, not
the Pleo account.

## 4. Report back

Totals per account, a list of flagged items (missing receipt, unsure VAT,
unsure account), and remind the user that drafts must be approved in Numina
before they are booked.
