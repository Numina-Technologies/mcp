---
name: numina-pleo
description: Draft Pleo card expenses and reimbursements in Numina with their receipts. Use when the user asks to book, sync or reconcile Pleo expenses.
---

# Pleo → Numina

Needs **Pleo** (expenses and receipts) and the **Numina MCP** with write
access ([setup](../../README.md#connect)). You create drafts; the user
books them in Numina.

## 1. Company and period

`get_company_info`: confirm the company and check `scopes` includes
`write:draft`. If not, ask the user to reconnect Numina. Default period:
expenses settled last calendar month.

## 2. Accounts

`list_accounts` and `list_vat_codes` once. Find the Pleo account (often named
"Pleo"); ask if there is none. Out-of-pocket expenses (udlæg) go to the
employee's payable account instead.

## 3. Each expense

1. **Duplicate?** `search_ledger_entries` (`status: "both"`) on the Pleo
   account for the same date and amount. Skip matches.
2. **Supplier.** `search_contacts`; if missing, `create_contact` with
   `roles: ["supplier"]` and only the details you actually know.
3. **Account and VAT.** Look at how this supplier was booked before
   (`search_ledger_entries` with its name) and reuse that. Otherwise pick
   from the Pleo category. Danish receipt → the deductible VAT code; foreign
   SaaS → reverse charge; restaurant → the representation code. Unsure of
   the account → leave `account_number` out so the line stays unfinished.
4. **`create_draft`**: date, text = supplier + Pleo note, `contact_id`, and
   two lines:
   - expense: account, `vat_code`, **positive** amount (gross for Danish
     VAT codes, net for reverse charge)
   - Pleo account: the same amount, **negative**, no VAT code

   Foreign currency: set `currency` and `amount_dkk` to what Pleo charged in
   DKK.
5. **Receipt.** `create_upload_link` with the draft's `transaction_id`,
   upload it, then `set_document_ocr`. No receipt → `update_draft` with
   `review_note: "Mangler bilag"`.

Give every write a one-line `reasoning`, e.g. "Pleo: Adobe subscription,
same account as previous Adobe charges."

## 4. Report back

Totals per account, and a list of drafts that need a look: missing receipt,
unsure VAT, unfinished account. Remind the user to approve the drafts in
Numina.
