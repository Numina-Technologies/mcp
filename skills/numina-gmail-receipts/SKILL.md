---
name: numina-gmail-receipts
description: Find receipts and supplier invoices in the user's Gmail and upload them to Numina as source documents. Use when the user asks to collect receipts, bilag or invoices from their inbox for bookkeeping.
---

# Gmail receipts → Numina

Needs **Gmail** and the **Numina MCP** ([setup](../../README.md#connect)).

## 1. Company and period

Call `get_company_info`, confirm the company with the user and note its
`slug`. Default period: last calendar month.

## 2. Find receipts

Search Gmail in the period:

```
has:attachment (filename:pdf OR filename:png OR filename:jpg)
  (kvittering OR faktura OR receipt OR invoice OR ordrebekræftelse OR "order confirmation")
```

Also look for receipts in the mail body from billing senders
(`from:(billing OR invoice OR noreply) receipt`). Skip newsletters, shipping
notices, quotes and reminders for invoices you already have.

## 3. Skip what Numina has

`search_documents` with the supplier, amount and date. Skip matches.

## 4. Upload

**If you can run shell commands** (Claude Code, Codex, code execution):

1. Save the attachment to a file.
2. `create_upload_link` with `file_name` and `content_type`
   (`application/pdf`, `image/png` or `image/jpeg`).
3. Run the returned `curl` within 10 minutes. You get an `attachment_id`;
   `duplicate: true` means Numina already had it.
4. Read the file yourself and call `set_document_ocr` with the vendor, date,
   total, VAT and invoice number. That makes it show in Numina and
   matchable to the bank.
5. If a draft for this purchase already exists without a document
   (`search_ledger_entries` with `status: "draft"`), attach it with
   `update_draft` and `attachment_id`.

**Otherwise** forward the mail to `<slug>@bilag.numina.app`. Numina reads
it and matches it to the bank. If you can only create drafts in Gmail,
create one forward draft per receipt and ask the user to send them.

## 5. Report back

A short table: date, supplier, amount, status (uploaded / already in Numina /
skipped and why). List anything you weren't sure was a receipt.
