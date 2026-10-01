# Numina MCP

Connect Claude, ChatGPT or any MCP client to your [Numina](https://numina.app)
bookkeeping. Ask about your numbers, find documents, and let your own AI do
bookkeeping work with ready-made skills.

```
https://api.numina.app/mcp
```

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
Create an API key in Numina (Integrationer → API forbindelser → Opret API-nøgle), then see
[`examples/clients.md`](./examples/clients.md).

You can revoke access any time in Numina under Integrationer → API forbindelser.

## Skills

Step-by-step instructions your AI follows to do a bookkeeping job end to end.

| Skill | What your AI does |
| --- | --- |
| [Gmail receipts](./skills/numina-gmail-receipts) | Finds receipts in Gmail and sends them to Numina |
| [Google Sheets](./skills/numina-google-sheets) | Puts your trial balance and key figures in a sheet |
| [Pleo](./skills/numina-pleo) | Books Pleo expenses with receipts |
| [Stripe](./skills/numina-stripe) | Books Stripe payouts, fees and VAT |

**Install**
- Claude: upload the skill folder under Settings → Capabilities → Skills,
  or copy it to `~/.claude/skills/` for Claude Code.
- ChatGPT and others: paste the `SKILL.md` into a project's instructions
  or the chat.

Then ask, for example: *"Run the Gmail receipts skill for last month."*

Rather not run it yourself? Numina's full service does all of this for you.

## Tools

| Tool | What it returns |
| --- | --- |
| `get_company_info` | Company, accounting years, granted scopes |
| `list_accounts` | Chart of accounts |
| `get_account_balances` | Account activity per period |
| `get_trial_balance` | Closing balances on a date |
| `search_ledger_entries` | Ledger entries by text, account, amount or date |
| `get_transaction` | One transaction with its evidence |
| `list_invoices`, `get_invoice` | Sales invoices |
| `search_contacts`, `get_contact` | Customers and suppliers |
| `list_vat_codes` | VAT codes |
| `search_documents`, `get_document` | Receipts and other source documents |
| `get_bank_balances` | Last-synced bank balances |

## Examples

- [`examples/prompts.md`](./examples/prompts.md): questions to try
- [`examples/clients.md`](./examples/clients.md): config for clients without OAuth

## Contributing

Found a mistake or built a skill worth sharing? Open an issue or a pull request.

## License

[MIT](./LICENSE). "Numina" and the Numina logo are trademarks of Numina Technologies ApS and are not covered by the license.
