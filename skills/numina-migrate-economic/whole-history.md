# e-conomic → Numina: the whole history through the API

This goes with [Move your whole history](../numina-migrate-books): follow
that skill, and use this page for where the data comes from.

If your AI already has e-conomic API access, you need no export files. Read
from the REST API (`restapi.e-conomic.com`), 1,000 rows per page, and write
the same tables:

| Read | Becomes |
| --- | --- |
| `/accounting-years` | `fiscal_years` in the manifest |
| `/accounts` | `accounts.csv`, with headings and sum accounts |
| `/vat-accounts` | `vat_codes.csv` |
| `/customers`, `/suppliers` | `contacts.csv` |
| `/accounting-years/{year}/entries` | `transactions.csv` and `postings.csv` |
| `/invoices/booked` | sales invoices in `vouchers.csv` |

Look at a page of each before you write the script; do not assume field
names. What to know:

- An entry is one posting line. Lines with the same `voucherNumber` in the
  same accounting year are one transaction. A voucher number is only
  unique within its year, so make `transaction_ref` from year and number.
- Entries **without** a `voucherNumber` are the year's opening balances:
  they become that year's `primo`, and the year before is closed.
- `amount` is in the entry's currency and `amountInBaseCurrency` in DKK.
  Keep both.
- A trial balance you compute from the same entries proves nothing. Ask
  the user for e-conomic's own Saldobalance report at each year end as a
  file, or tell them the figures are unchecked.
- Save what you fetched to files and run the script on those, so a rerun
  does not fetch everything again.
