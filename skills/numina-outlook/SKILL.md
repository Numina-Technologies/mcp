---
name: numina-outlook
description: Find receipts and supplier invoices in the user's Outlook (Microsoft 365 or Outlook.com) mailbox and get them into Numina as source documents. Use when the user asks to collect receipts, bilag or invoices from their Outlook inbox for bookkeeping.
---

# Outlook receipts → Numina

Needs an **Outlook** connector (Microsoft 365 or Outlook.com) and the
**Numina MCP** ([setup](../../README.md#connect)).

## 1. Company and period

Call `get_company_info`, confirm the company with the user and note its
`slug`. Default period: last calendar month.

## 2. Find receipts

Search the mailbox in the period. Outlook search takes KQL; set the dates in
your tool's date filter if it has one, otherwise in the query:

```
hasattachments:true received>=<first day> received<=<last day>
  (kvittering OR faktura OR receipt OR invoice OR ordrebekræftelse OR "order confirmation")
```

Keep PDF and image attachments (`.pdf`, `.png`, `.jpg`); ignore signature
logos and calendar invites. Also look for receipts in the mail body from
billing senders (`from:billing OR from:invoice OR from:noreply` with
`receipt`). Skip newsletters, shipping notices, quotes and payment reminders.
Search all folders, not just the inbox: receipts are often filed or archived.

## 3. Skip what Numina already has

For each receipt, call `search_documents` with `vendor`, `amount` (the total)
and `from_date`/`to_date` a few days around the receipt date. Vendor and
amount only rank the results; they don't filter. So check the top hits
yourself: the same vendor, total and date means Numina has it. Skip those.

## 4. Get it into Numina

Pick the first path that works for you.

**A. Forward it.** If you can send mail from Outlook, forward the receipt to
`<slug>@bilag.numina.app`. Numina reads it and matches it to the bank line
itself. If you can only create drafts, create one forward draft per receipt
and ask the user to send them.

**B. Upload it.** If you can run shell commands (Claude Code, Codex, code
execution):

1. Save the attachment to a file.
2. `create_upload_link` with `file_name` and `content_type`
   (`application/pdf`, `image/png` or `image/jpeg`).
3. Run the returned `curl` within 10 minutes. It returns an
   `attachment_id`. `duplicate: true` means Numina already had the file;
   skip to the next receipt.
4. Read the file and call `set_document_ocr` with `document_type`, a
   one-line `summary`, `vendor_name`, `date`, `total_amount`,
   `total_vat_amount`, `currency` and `invoice_number` when there is one.
5. Numina does not match uploads to the bank by itself. If a draft for this
   purchase exists without a document, attach it: find it with
   `search_ledger_entries` (`status: "draft"`, `text_search` = the vendor,
   dates around the receipt) and call `update_draft` with the
   `attachment_id`. Otherwise the receipt waits in Numina's document list
   for the user to match.

If neither path works, list the receipts for the user to forward by hand.
An Outlook rule that forwards mail from regular suppliers to
`<slug>@bilag.numina.app` saves doing this every month.

## 5. Report back

A short table: date, supplier, amount, status (forwarded / uploaded and
attached / uploaded, not matched / already in Numina / skipped and why).
List anything you weren't sure was a receipt.
