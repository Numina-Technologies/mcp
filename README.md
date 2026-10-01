# Numina MCP

Connect Claude, ChatGPT or any MCP client to your [Numina](https://numina.app)
bookkeeping. Ask about your numbers, and let your own AI do the bookkeeping
work with ready-made skills.

```
https://api.numina.app/mcp
```

Your AI can read your books and write **drafts**. It can never book anything:
a person approves every draft in Numina.

## Connect

**Claude (claude.ai / Desktop)**
Settings → Connectors → Add custom connector → paste the URL → Connect.
Log in to Numina, pick the company and approve.

**Claude Code**

```bash
claude mcp add --transport http numina https://api.numina.app/mcp
```

Then run `/mcp` and choose **Authenticate**.

**ChatGPT**
Settings → Apps & Connectors → Advanced → turn on Developer mode, then
create a connector with the URL above and sign in to Numina.

**Other clients (Cursor, VS Code, …)**
Use an API key, see [`examples/clients.md`](./examples/clients.md).

Connected before October 2026? Disconnect and connect again to give your AI
write access. You can revoke access any time in Numina under
Integrationer → API forbindelser.

## Skills

Step-by-step instructions your AI follows to do a bookkeeping job end to end.

| Skill | What your AI does |
| --- | --- |
| [Gmail receipts](./skills/numina-gmail-receipts) | Finds receipts in Gmail and uploads them to Numina |
| [Pleo](./skills/numina-pleo) | Drafts Pleo expenses with their receipts |
| [Stripe](./skills/numina-stripe) | Drafts Stripe payouts against the bank, with sales, VAT and fees |
| [Google Sheets](./skills/numina-google-sheets) | Puts your trial balance and key figures in a sheet |

**Install**
- Claude: upload the skill folder under Settings → Capabilities → Skills,
  or copy it to `~/.claude/skills/` for Claude Code.
- ChatGPT and others: paste the `SKILL.md` into a project's instructions
  or the chat.

Then ask, for example: *"Run the Gmail receipts skill for last month."*

Rather not run it yourself? Numina's full service does all of this for you.

## Tools

**Read**

| Tool | What it returns |
| --- | --- |
| `get_company_info` | Company, accounting years, granted scopes |
| `list_accounts`, `list_vat_codes` | Chart of accounts and VAT codes |
| `get_account_balances` | Net movement per account per period |
| `get_trial_balance` | Balances on a date |
| `search_ledger_entries`, `get_transaction` | Booked and draft transactions |
| `search_bank_transactions`, `get_bank_transaction` | Bank lines and whether they're reconciled |
| `get_bank_balances` | Last-synced bank balances |
| `list_invoices`, `get_invoice` | Sales invoices |
| `search_contacts`, `get_contact` | Customers and suppliers |
| `search_documents`, `get_document` | Receipts and other source documents |

**Write** (drafts only, nothing is booked)

| Tool | What it does |
| --- | --- |
| `create_draft`, `update_draft`, `delete_draft` | Balanced draft transactions, optionally for a bank line |
| `reconcile_bank_transaction`, `unlink_bank_transaction` | Match a bank line to a draft, or undo it |
| `propose_booked_change` | Suggest a correction to a booked transaction |
| `create_contact`, `update_contact` | Suppliers and customers |
| `create_upload_link`, `set_document_ocr` | Upload a receipt and record what it says |
| `resolve_review` | Close a review once it's fixed |

Lines are signed (positive = debit, negative = credit) and must balance to 0.
Danish VAT codes take the gross amount, reverse-charge codes the net amount.
Every write takes a short `reasoning`, shown on the transaction's history.

## Examples

- [`examples/prompts.md`](./examples/prompts.md): things to ask
- [`examples/clients.md`](./examples/clients.md): setup with an API key

## Contributing

Found a mistake or built a skill worth sharing? Open an issue or a pull request.

## License

[MIT](./LICENSE). "Numina" and the Numina logo are trademarks of Numina Technologies ApS and are not covered by the license.
