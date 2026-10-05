---
name: numina-pleo
description: Draft Pleo card expenses and out-of-pocket expenses in Numina with their receipts. Use when the user asks to book, sync or reconcile Pleo expenses.
---

# Pleo → Numina

Needs **Pleo** (expenses and receipts) and the **Numina MCP** with write
access ([setup](../../README.md#connect)). You create drafts; the user
approves and books them in Numina.

## 1. Company and period

`get_company_info`: confirm the company and check `scopes` includes
`write:draft`. If not, ask the user to reconnect Numina. Default period:
expenses settled last calendar month.

## 2. Accounts, once

`list_accounts` and `list_vat_codes`. Find:

- the **Pleo account** (often named "Pleo"). Ask if there is none.
- for out-of-pocket expenses (udlæg), the **employee payable account**.

Top-ups from the bank to Pleo are transfers, not expenses. Leave them out.

## 3. Each expense

1. **Already in Numina?** `search_ledger_entries` with `status: "both"`,
   `account_number` = the Pleo account, `date_from` and `date_to` = the
   expense date, and `signed_amount_min` = `signed_amount_max` = minus the
   DKK amount (the Pleo line is a credit). A hit means skip.
2. **Supplier.** `search_contacts` by name. If there's no match,
   `create_contact` with `roles: ["supplier"]` and only details you actually
   know.
3. **Account and VAT.** See how this supplier was booked before
   (`search_ledger_entries` with `text_search` = the supplier) and reuse that
   account and VAT code. Otherwise choose from the Pleo category:
   - Danish receipt → the company's deductible purchase VAT code
   - foreign service (e.g. SaaS) → the reverse-charge code for EU or
     non-EU services
   - restaurant → the company's code for restaurant/representation
     (25 % of the VAT is deductible); ask if there isn't one
   - no valid receipt → no VAT code

   If you're unsure of the account, leave `account_number` out. The line
   stays unfinished instead of wrong.
4. **`create_draft`** with `date`, `text` (supplier and Pleo note),
   `contact_id`, and two lines:
   - expense account, `vat_code`, **positive** amount: gross with a Danish
     VAT code, net with a reverse-charge code
   - Pleo account (or employee payable for udlæg), the same amount,
     **negative**, no VAT code

   Paid in another currency? Set `currency` to it, use amounts in that
   currency, and put Pleo's DKK amount on each line as `amount_dkk` (with
   the expense line's sign) so the draft matches what Pleo charged.
5. **Receipt.** If you can run shell commands: `create_upload_link` with
   the draft's `transaction_id`, upload the receipt with the returned `curl`,
   then `set_document_ocr` with what it says. If you can't, or there's no
   receipt: `update_draft` with `review_note: "Mangler bilag"`.

Give every write a one-line `reasoning`, e.g. "Pleo: Adobe subscription,
same account as earlier Adobe charges."

## 4. Report back

Totals per account, and the drafts that need a look: missing receipt, unsure
VAT, unfinished account. Remind the user to approve the drafts in Numina.
