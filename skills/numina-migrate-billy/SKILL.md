---
name: numina-migrate-billy
description: Move from Billy to Numina by writing the opening balance (åbningsbalance) from Billy's trial balance as a draft in Numina. Use when the user is switching from Billy and wants their balances carried over.
---

# Billy → Numina: opening balance

Moves the **balances** from Billy to Numina, not the full history. You
read Billy's trial balance (saldobalance) on the last day before the
company starts in Numina, and write it as one opening-balance draft. The
user checks it and books it in Numina.

Needs the **Numina MCP** with write access ([setup](../../README.md#connect))
and Billy's trial balance, from one of these:

**Export it.** In Billy, open the *Saldobalance* report for the last day of
the previous accounting year and export it (Excel or CSV), then give the
file to your AI.

**Or read it from the API.** If your AI has Billy API access, it can read the
accounts and postings up to that date instead.

## 1. Company and start date

1. `get_company_info`: confirm the company and check `scopes` includes
   `write:draft`. If not, ask the user to reconnect Numina.
2. The opening date is the `start_date` of the **earliest** year in
   `accounting_years`. Confirm it with the user. The trial balance you need
   from Billy is the one for the day before.
3. Check there's no opening balance already: `search_ledger_entries` with
   `status: "both"`, `date_from` = `date_to` = the opening date. If you find
   one, stop and ask the user.

## 2. Read the trial balance

From Billy's trial balance, keep only the **balance-sheet** accounts
(assets, liabilities and equity) that have a balance. Leave out the profit
and loss accounts: their total is last year's result, which goes to equity
in step 4.

Check that the whole trial balance, all accounts, sums to 0. If it
doesn't, the export is incomplete; ask the user before going on.

## 3. Match the accounts

`list_accounts` gives Numina's chart of accounts with names and types.
Match each Billy account to a Numina account by number, name and type.
Danish charts often share numbers, but check the name every time.

Ask the user about any account you can't match with confidence. If they
don't know either, leave `account_number` out on that line: it stays
unfinished in the draft instead of wrong.

## 4. Write the draft

`create_draft` with:

- `date`: the opening date
- `text`: "Åbningsbalance fra Billy pr. <opening date>"
- one line per balance-sheet account, no VAT code, `amount` = the balance
  from Billy: **positive** for debit balances (assets), **negative** for
  credit balances (liabilities, equity)
- one last line on Numina's **Overført resultat** (retained earnings)
  account for last year's result, so the draft sums to 0
- `reasoning`: "Opening balance from Billy's trial balance on <date>."

The draft must balance. If `create_draft` refuses, recheck the signs and
the matching. Never plug a difference onto another account.

## 5. Report back

Show the user:

- a table of Billy account → Numina account → amount
- last year's result as carried to Overført resultat
- any lines left without an account

Then explain three things before they book it:

- **Bank:** the bank account's opening line can show as unreconciled in
  Numina's bank view if the bank feed covers the opening date. That's
  expected.
- **Customers and suppliers:** open receivables and payables come over as
  totals on the debtor and creditor accounts, not per customer or supplier.
  Unpaid invoices from Billy still need to be followed up there.
- **Booking:** nothing is booked until they approve the draft in Numina.
