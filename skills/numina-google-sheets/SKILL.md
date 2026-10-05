---
name: numina-google-sheets
description: Pull the trial balance and key figures from Numina into a Google Sheet. Use when the user wants their accounting numbers, saldobalance or a monthly overview in Google Sheets.
---

# Numina → Google Sheets

Needs **Google Sheets** and the **Numina MCP**
([setup](../../README.md#connect)). This skill only reads from Numina.

## 1. Company and period

- `get_company_info`: confirm the company and pick an accounting year from
  `accounting_years` (default: the current one, up to today).
- Ask whether to create a new sheet or update an existing one. When
  updating, only touch the tabs this skill creates.

## 2. Fetch the figures

- `list_accounts`: numbers, names and account types.
- `get_trial_balance` with `date` = the period end: balances on that date.
- `get_account_balances` with `from`/`to` = the period, `granularity:
  "month"` and `statement: "profit_loss"`: movement per account per month.
- Last year, if that accounting year exists: the same `get_account_balances`
  call for the same months, and `get_trial_balance` one year before the
  period end.

Sign convention: negative = income/credit, positive = expense/debit. Flip
income to positive in the sheet, and say so in a note.

## 3. Write the sheet

| Tab | Content |
| --- | --- |
| `Saldobalance` | Account number, name, type, balance at the period end |
| `Måneder` | One row per P&L account, one column per month, a total column |
| `Nøgletal` | Revenue, gross profit, operating result, result before tax, and bank balance (the bank accounts in the trial balance); this year and last year side by side |

Write numbers as numbers, not text, with two decimals. Put the company name,
period and "Hentet fra Numina <date>" in the first row of each tab. Build
`Nøgletal` with formulas referencing `Måneder`, so the user can see how each
figure is made, and group accounts by their type from `list_accounts`.

## 4. Report back

A link to the sheet and two lines: the result for the period, and how it
compares with the same period last year.
