---
name: numina-google-sheets
description: Pull the trial balance and key figures from Numina into a Google Sheet. Use when the user wants their accounting numbers, saldobalance or a monthly overview in Google Sheets.
---

# Numina → Google Sheets

Needs **Google Sheets** and the **Numina MCP**
([setup](../../README.md#connect)). This skill only reads from Numina.

## 1. Confirm the company and period

- Call `get_company_info`. Confirm the company and pick an accounting year
  from `accounting_years`.
- Ask whether the user wants a new sheet or an existing one updated. If
  updating, only touch tabs this skill created.

## 2. Fetch the figures

- `get_trial_balance` for the closing balances at the period end.
- `get_account_balances` (activity per period) for month-by-month
  movements on revenue and expense accounts.
- `list_accounts` to get account names, numbers and types.

## 3. Write the sheet

Create these tabs:

| Tab | Content |
| --- | --- |
| `Saldobalance` | Account no., name, type, closing balance |
| `Måneder` | One row per account, one column per month |
| `Nøgletal` | Revenue, gross profit, operating result, result before tax, bank balance |

Write numbers as numbers, not text, with two decimals. Put the company name,
period and "Hentet fra Numina <date>" in the first row of each tab.

Compute key figures with sheet formulas that reference the other tabs, so
the user can see how each number is built.

## 4. Report back

Link to the sheet and a two-line summary: result for the period and how it
compares to the same period last year, if that year is available.
