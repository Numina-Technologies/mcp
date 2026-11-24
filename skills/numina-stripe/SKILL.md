---
name: numina-stripe
description: Draft Stripe payouts in Numina against the bank line, split into sales, VAT, refunds and fees. Use when the user asks to book, sync or reconcile their Stripe revenue.
---

# Stripe → Numina

Needs **Stripe** (payouts and balance transactions) and the **Numina MCP**
with write access ([setup](../../README.md#connect)). One payout = one draft
on the bank line it landed on. The user approves and books the drafts in
Numina.

If the company invoices its customers in Numina and Stripe only collects the
payment, those charges are not sales: step 3 finds them and books them
against the invoice instead.

## 1. Company and period

`get_company_info`: confirm the company. Check that `create_draft` is among
your Numina tools: Numina only lists the tools the connection may use. If
it's missing, the connection is read-only; ask the user to reconnect Numina
with write access.

## 2. Accounts, once

`list_accounts` and `list_vat_codes`. Identify the sales account(s), the
Danish output VAT code, the code for EU sales without Danish VAT and the
fees account. Also find the debtor account (what customers owe, often
"Debitorer") if the company has invoices in Numina. Ask when unsure.

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
   currency. Note each one's `type` and `source` (the charge, refund or
   other object it comes from).

   Stripe can only list this for automatic payouts. If the payout's
   `automatic` is false, don't draft it: report it without a breakdown and
   ask the user which transactions it covers.
3. **Is a charge already invoiced in Numina?** Then the invoice has already
   booked the revenue and the output VAT, and booking the charge as sales
   books them twice. Call `list_invoices` with `type: "invoice"`,
   `from_date` two months before the oldest charge in the payout and
   `include_line_items: false`, and ignore drafts and cancelled invoices.
   Don't use `open_only`: it hides an invoice whose payment already sits
   unmatched in the bank. No invoices → nothing is invoiced; go to step 4.

   Match each charge to an invoice by:
   - its invoice number, when the charge's description or metadata, or its
     Stripe invoice, carries one (`list_invoices` takes `invoice_number`)
   - otherwise the same customer (`search_contacts` by name) and the same
     total in the same currency

   Use a match only when exactly one invoice fits. If the invoice is
   already `paid`, is in another currency than the payout, or fits more
   than one charge, don't book the charge as sales: give it a **negative**
   line for its gross amount with `account_number` left out (its fee stays
   with the fees), flag it as in step 5 and list it in the report.
4. **Group the other charges.** Balance transactions carry no customer
   country or VAT number. Expand each one's `source` to the charge and read:
   - **country:** the charge's `billing_details.address.country`, or the
     customer's address. For a charge on a Stripe invoice, that invoice's
     `customer_address`. Not `payment_method_details.card.country`: that is
     the card's country, not the customer's.
   - **VAT number:** a tax ID of type `eu_vat` on the customer, or in the
     Stripe invoice's `customer_tax_ids`.

   If you can't read a charge's country, don't guess: list those charges
   and ask the user once for all of them. Then group:
   - **Denmark** → Danish output VAT code
   - **EU business with a VAT number** → EU sale, no Danish VAT
   - **EU consumer** → OSS if the company is registered for it, otherwise
     Danish VAT; ask the user
   - **outside the EU** → no VAT
5. **Anything that isn't a charge, refund or fee.** A dispute or chargeback
   (`type` `adjustment`), `tax_fee`, a reserve, a transfer and the like
   don't fit a sales group. Give each its own line with `account_number`
   left out, so it stays unfinished instead of wrong: the text is its
   `type`, Stripe ID and description, and the amount is its `net`
   (`amount` minus `fee`) with the opposite sign. Never fold it into a
   sales group or the fees. After `create_draft`, call `update_draft` with
   a `review_note` that says which lines need an account.
6. **`create_draft`** with `bank_transaction_id`, `date` = the payout date,
   and lines that sum to 0:
   - bank account (`ledger_account_number`): the payout, **positive**
   - invoiced charges (step 3): the debtor account, **negative**, the
     charge's gross amount, no VAT code (the invoice booked the VAT), the
     invoice number in the text. If the payout pays exactly one invoice,
     also pass its `invoice_id`.
   - sales per group: **negative**, with that group's VAT code from step
     4. A group on Danish VAT takes the gross amount and the Danish output
     VAT code. EU business sales take the EU sale code, and sales outside
     the EU take no VAT code: never put Danish VAT on those.
   - refunds: **positive**, on the same accounts and codes as the sale. A
     refund of an invoiced charge goes on the debtor account.
   - fees: **positive** on the fees account. Stripe's fees are usually
     VAT-exempt; follow the code the company already uses for them.
   - anything from step 5

   Payout in another currency (e.g. a EUR bank account)? Set `currency`;
   Numina converts at the day's rate.
7. If `create_draft` refuses because the lines don't balance, recheck the
   grouping. Never plug the difference.

`reasoning` example: "Stripe payout po_123: 42 charges, 1 refund, fees."

## 4. Report back

Per payout: amount, sales per VAT group, refunds, fees and
a link to the draft. List the charges booked against invoices (with the
invoice numbers), the lines left without an account, and the charges whose
country you couldn't read. List the payouts you left out: not arrived,
several possible bank lines, or manual. Remind the user to approve the
drafts in Numina.
