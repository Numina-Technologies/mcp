---
name: numina-gmail-receipts
description: Find receipts and supplier invoices in the user's Gmail and get them into Numina as source documents. Use when the user asks to collect receipts, bilag or invoices from their inbox for bookkeeping.
---

# Gmail receipts → Numina

You need two connections ([setup](../../README.md#connect)): **Gmail** (search and read mail) and the
**Numina MCP**.

## 1. Confirm the company and period

- Call `get_company_info`. Tell the user which company you are working on and
  note its `slug`.
- Agree on a period. Default: last calendar month.

## 2. Find candidate mails

Search Gmail within the period. Start broad, then read each hit:

```
has:attachment (filename:pdf OR filename:png OR filename:jpg)
  (kvittering OR faktura OR receipt OR invoice OR ordrebekræftelse OR "order confirmation")
```

Also search for mails *without* attachments from known billing senders
(e.g. `from:(billing OR invoice OR noreply)` with "receipt"/"kvittering"),
since many SaaS receipts are in the body or behind a link.

Skip newsletters, shipping notices, quotes (tilbud) and reminders (rykker)
for invoices you already found.

## 3. Check what Numina already has

For each candidate, call `search_documents` with the supplier name, amount or
invoice number. Skip anything already there.

## 4. Send the new ones to Numina

Every company has an inbox for source documents:
`<slug>@bilag.numina.app`. Numina reads the attachment and matches it to
the bank transaction.

- Preferred: if the Numina MCP offers a document upload tool, upload the
  attachment directly.
- Otherwise forward the mail (with its attachment) to
  `<slug>@bilag.numina.app`.
- If your Gmail connection can only create drafts, create one forward draft
  per receipt and tell the user to send them.

A receipt that only exists in the mail body: save the body as a PDF if you
can, otherwise forward the mail as is.

## 5. Report back

A short table: date, supplier, amount, and status (sent / already in Numina /
skipped + why). Mention anything that looked like a receipt but that you were
unsure of, and let the user decide.
