

## Quick logging

- **One line:** type `4.50 coffee`, `12 uber #transport` or `30 lunch yesterday` in the box at the top and press Enter. The tag is guessed from what you logged before (on your device); type `#tag` to pick one.
- **One tap:** your most frequent spends from the last 45 days show as chips under the box.
- **Apple Pay (iPhone):** in Shortcuts → Automation → **Transaction**, pick your cards, then add *Open URL*:
  `https://waliiidals.github.io/Everything-Budgeted/?amount=[Amount]&merchant=[Merchant]&ref=[Current Date]`
  Every tap-to-pay purchase opens the app and logs itself. The `ref` stops the same purchase being logged twice. Run *Merchant* and *Current Date* (formatted ISO 8601) through a *URL Encode* action first so spaces don't break the link.
- **Other automations:** `?q=4.50 coffee` logs a quick-log line; `?quick=1` opens the app with the quick-log box focused (also the "Quick log" home-screen shortcut).

## Duplicate detection

Before any spend is saved (quick log, chip, checklist tick, analytics form, automation) it is compared with spends on the same day with the same amount. It is flagged when enough extra evidence lines up: logged within minutes of each other, the same or similar note/merchant, the same tag, or coming from two different routes (e.g. Apple Pay automation and by hand). You get an "Already logged?" prompt with *Don't add* / *Add anyway*, so two genuine coffees are never blocked. A repeated automation `ref` is skipped automatically.

## Bank statement import

Tap **import bank statement** under the quick-log box and pick the file your bank exports (.xls, .xlsx, .csv or an HTML-table export). The file is read in the browser and never uploaded; the [SheetJS](https://sheetjs.com) reader is downloaded from cdn.sheetjs.com the first time you import.

- It finds the Date / Description / Debit (or Amount) table, keeps only purchases and pulls out the date, the place (e.g. `UTap KNET POS @STARBUCKS NUZHA COOP` becomes "Starbucks Nuzha Coop") and the amount.
- Transfers, salary and other income, refunds, cash withdrawals and totals are skipped and counted in the summary.
- Card rows that carry their own purchase date (`VISA POS,28/9/26,…`) use that date instead of the posting date.
- A review list shows every purchase with a suggested tag (learned from your history, then from the merchant name) before anything is saved.
- Re-importing the same statement, or one that overlaps, only adds rows that are new. Rows that match something you already logged by hand (same amount, same or next day, plus the usual duplicate evidence) are shown unticked with the match next to them.
